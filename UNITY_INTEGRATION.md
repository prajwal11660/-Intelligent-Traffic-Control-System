# Unity Integration Guide

## Overview

This guide explains how to connect Unity to the running SUMO simulation to visualize traffic in real-time.

## Architecture

```
SUMO Simulator (Python) ←→ TraCI Port 8813 ←→ Metrics Collector
                                             ↓
                                        UDP Socket 5555
                                             ↓
                                        Unity (C# Receiver)
```

## Step 1: Create UDP Receiver in Unity

Create a new C# script called `SUMOTrafficReceiver.cs`:

```csharp
using UnityEngine;
using System.Net;
using System.Net.Sockets;
using System.Threading;
using UnityEngine.Serialization;

public class SUMOTrafficReceiver : MonoBehaviour
{
    [SerializeField] private int listenPort = 5555;
    [SerializeField] private Transform vehicleContainer;
    [SerializeField] private GameObject vehiclePrefab;
    
    private UdpClient udpClient;
    private Thread receiveThread;
    private bool isRunning = false;
    private Queue<string> messageQueue = new Queue<string>();
    
    void Start()
    {
        // Initialize UDP listener
        udpClient = new UdpClient(listenPort);
        isRunning = true;
        
        // Start receive thread
        receiveThread = new Thread(new ThreadStart(ReceiveData));
        receiveThread.IsBackground = true;
        receiveThread.Start();
        
        Debug.Log($"UDP Listener started on port {listenPort}");
    }
    
    void ReceiveData()
    {
        IPEndPoint remoteEP = new IPEndPoint(IPAddress.Any, listenPort);
        
        while (isRunning)
        {
            try
            {
                byte[] receivedData = udpClient.Receive(ref remoteEP);
                string receivedMessage = System.Text.Encoding.ASCII.GetString(receivedData);
                
                lock (messageQueue)
                {
                    messageQueue.Enqueue(receivedMessage);
                }
            }
            catch (SocketException ex)
            {
                if (isRunning)
                    Debug.LogError($"Socket error: {ex.Message}");
            }
        }
    }
    
    void Update()
    {
        // Process queued messages
        lock (messageQueue)
        {
            while (messageQueue.Count > 0)
            {
                string message = messageQueue.Dequeue();
                ProcessTrafficData(message);
            }
        }
    }
    
    void ProcessTrafficData(string jsonData)
    {
        try
        {
            // Parse JSON vehicle data
            // Expected format:
            // {
            //   "vehicles": [
            //     {"id": "vehicle1", "x": 100, "y": 200, "z": 0, "speed": 5.5, "angle": 45},
            //     ...
            //   ],
            //   "traffic_lights": [
            //     {"id": "tl1", "state": "GrGr"}
            //   ]
            // }
            
            // Use JsonUtility or Newtonsoft.Json to parse
            // Update vehicle positions and traffic light colors accordingly
            
            Debug.Log($"Received traffic data: {jsonData.Substring(0, 50)}...");
        }
        catch (System.Exception ex)
        {
            Debug.LogError($"Error processing traffic data: {ex.Message}");
        }
    }
    
    void OnDestroy()
    {
        isRunning = false;
        udpClient?.Close();
        receiveThread?.Abort();
    }
}
```

## Step 2: Create Vehicle Visualization

Create `VehicleVisualizer.cs`:

```csharp
using UnityEngine;

public class VehicleVisualizer : MonoBehaviour
{
    [SerializeField] private float scaleFactor = 0.001f; // Convert SUMO positions to Unity scale
    [SerializeField] private float heightOffset = 0.5f;
    
    private string vehicleId;
    private Rigidbody rb;
    
    public void Initialize(string id)
    {
        vehicleId = id;
        rb = GetComponent<Rigidbody>();
    }
    
    public void UpdatePosition(float x, float y, float z)
    {
        // Convert SUMO coordinates to Unity coordinates
        Vector3 newPos = new Vector3(x * scaleFactor, heightOffset, y * scaleFactor);
        
        if (rb != null)
            rb.MovePosition(newPos);
        else
            transform.position = newPos;
    }
    
    public void UpdateRotation(float angle)
    {
        // SUMO angle to Unity rotation
        transform.rotation = Quaternion.Euler(0, -angle, 0); // Negate for correct direction
    }
    
    public void SetSpeed(float speed)
    {
        // Optional: Update visual effects based on speed
        // e.g., change color intensity, particle effects, etc.
    }
}
```

## Step 3: Create Traffic Light Visualization

Create `TrafficLightVisualizer.cs`:

```csharp
using UnityEngine;

public class TrafficLightVisualizer : MonoBehaviour
{
    [SerializeField] private Light redLight;
    [SerializeField] private Light greenLight;
    [SerializeField] private Light yellowLight;
    
    public void SetState(string state)
    {
        // SUMO state format: first char = green (g), red (r), yellow (y)
        // e.g., "GrGr" means lights 0 and 2 are green, 1 and 3 are red
        
        // Example: Handle first light
        switch (state[0])
        {
            case 'G':
            case 'g':
                SetGreen();
                break;
            case 'R':
            case 'r':
                SetRed();
                break;
            case 'Y':
            case 'y':
                SetYellow();
                break;
        }
    }
    
    void SetGreen()
    {
        greenLight.enabled = true;
        redLight.enabled = false;
        yellowLight.enabled = false;
    }
    
    void SetRed()
    {
        greenLight.enabled = false;
        redLight.enabled = true;
        yellowLight.enabled = false;
    }
    
    void SetYellow()
    {
        greenLight.enabled = false;
        redLight.enabled = false;
        yellowLight.enabled = true;
    }
}
```

## Step 4: Setup Scene

1. Create an empty GameObject called "SUMOManager"
2. Add `SUMOTrafficReceiver` script to it
3. Set Listen Port to `5555`
4. Create a container for vehicles
5. Create vehicle prefab with:
   - 3D mesh (cube or vehicle model)
   - `VehicleVisualizer` script
6. Create traffic light prefabs with:
   - Three colored lights (red, green, yellow)
   - `TrafficLightVisualizer` script

## Step 5: Configure Python Side

The `simulation_runner.py` already has UDP sending code. Ensure it's enabled:

```python
# In simulation_runner.py, the connect_unity() method initializes UDP socket
# It sends vehicle data every simulation step
```

## Step 6: Run Integration Test

1. Start SUMO:
```bash
sumo -c RequiredFiles/Sumo2Unity.sumocfg --remote-port 8813
```

2. Start Python simulation:
```bash
python RequiredFiles/simulation_runner.py RequiredFiles/Sumo2Unity.sumocfg adaptive
```

3. Run Unity scene
4. Observe vehicles and traffic lights updating in real-time

## Data Format (JSON)

Vehicle data sent via UDP:

```json
{
  "timestamp": 0.1,
  "step": 1,
  "vehicles": [
    {
      "id": "vehicle_0",
      "x": 1000.5,
      "y": 2000.3,
      "z": 0.0,
      "speed": 8.5,
      "angle": 45.0,
      "type": "car"
    }
  ],
  "traffic_lights": [
    {
      "id": "cluster_123_456",
      "state": "GrGr"
    }
  ],
  "metrics": {
    "total_vehicles": 45,
    "avg_speed": 6.2,
    "avg_waiting_time": 12.5
  }
}
```

## Performance Optimization

1. **Object Pooling**: Reuse vehicle GameObjects to avoid instantiation overhead
2. **Update Groups**: Only update vehicles that have changed
3. **LOD**: Use different detail levels for distant vehicles
4. **Batching**: Combine traffic light meshes

## Troubleshooting

### No vehicles appearing?
- Check UDP port (5555) is accessible
- Verify Python simulation is sending data
- Check Console for errors in C# script

### Vehicles appear jittery?
- Increase scale factor to match SUMO units to Unity scale
- Use `Rigidbody.MovePosition()` instead of direct position assignment
- Smooth position with Vector3.Lerp()

### Traffic lights not updating?
- Verify light state characters match SUMO output
- Check light prefab has `TrafficLightVisualizer` component
- Log the received state values

## Next Steps

1. Implement smoothing for vehicle movement
2. Add vehicle color coding (speed-based)
3. Display metrics on HUD
4. Add camera following vehicles
5. Implement playback speed control

## References

- [SUMO Python TraCI Documentation](https://sumo.dlr.de/docs/TraCI/Interfacing_TraCI_from_Python.html)
- [Unity Networking](https://docs.unity3d.com/Manual/UsingTheNetworkClass.html)
- [UDP Client Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.udpclient)

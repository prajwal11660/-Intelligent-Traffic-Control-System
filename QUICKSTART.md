# Quick Start Guide - SUMO2Unity Simulation

## 🚀 30-Second Setup

### Prerequisites
- SUMO installed and in PATH
- Python 3.7+
- Working directory: `SUMO2Unity-main/`

## ⚡ Run a Quick Test (2 minutes)

### Option 1: Two-Terminal Approach (Recommended)

**Terminal 1:**
```bash
cd SUMO2Unity-main
sumo -c RequiredFiles/simple_config.sumocfg --remote-port 8813
```

**Terminal 2:** (wait 3 seconds, then)
```bash
cd SUMO2Unity-main
python RequiredFiles/test_quick_sim.py
```

**Expected Result:**
```
✅ Connected to SUMO on port 8813
Step   0: 3 vehicles in simulation
Step  20: 0 vehicles
...
✅ Test completed successfully!
```

---

## 🔧 Run Full Hour Simulation

```bash
# Terminal 1: Start SUMO
sumo -c RequiredFiles/Sumo2Unity.sumocfg --remote-port 8813

# Terminal 2: Run metrics collection (after SUMO starts)
python RequiredFiles/simulation_runner.py RequiredFiles/Sumo2Unity.sumocfg adaptive
```

**Output**: Results saved to `Results/` directory

---

## 📊 Run Baseline vs Adaptive Comparison

```bash
# Terminal 1: Start SUMO
sumo -c RequiredFiles/Sumo2Unity.sumocfg --remote-port 8813

# Terminal 2: Run comparison
python RequiredFiles/simulation_runner.py RequiredFiles/Sumo2Unity.sumocfg compare
```

**Compares**:
- BASELINE: Fixed-time traffic light control
- ADAPTIVE: Intelligent traffic control

---

## 🐛 Troubleshooting

### Port 8813 already in use?
```powershell
# PowerShell
taskkill /F /IM sumo.exe

# Try again
sumo -c RequiredFiles/Sumo2Unity.sumocfg --remote-port 8813
```

### SUMO can't find files?
```bash
# Use absolute path
sumo -c c:\...\RequiredFiles\Sumo2Unity.sumocfg --remote-port 8813
```

### Connection Refused?
- Make sure SUMO is running (Terminal 1)
- Wait 3 seconds after SUMO starts
- Check port: `netstat -an | findstr 8813`

---

## 📁 Key Files

| File | Purpose |
|------|---------|
| [Sumo2Unity.sumocfg](RequiredFiles/Sumo2Unity.sumocfg) | Production config (1 hour) |
| [simple_config.sumocfg](RequiredFiles/simple_config.sumocfg) | Test config (10 min) |
| [simple_routes.rou.xml](RequiredFiles/simple_routes.rou.xml) | Verified traffic routes |
| [test_quick_sim.py](RequiredFiles/test_quick_sim.py) | Quick test script |
| [city.net.xml](RequiredFiles/city.net.xml) | Network topology |

---

## 📚 More Information

See [PROJECT_COMPLETION.md](PROJECT_COMPLETION.md) for full documentation

---

## ✅ Status

- ✅ Configuration validated
- ✅ Routes corrected (valid edges)
- ✅ TraCI integration working
- ✅ Test passing
- ⬜ Ready for Unity integration

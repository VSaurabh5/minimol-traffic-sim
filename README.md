# Minimal Traffic Simulator (3D)

An interactive 3D traffic intersection simulation built with **Three.js**.

## Features

- **3D Visualization**: Built using Three.js with orbit controls, dynamic lighting, multiple camera views (3D View, Top-Down, Follow Car), and day/night theme toggle.
- **Traffic Signal Control**: Traffic light cycle states (Green, Yellow, Red) for 4-way intersection lanes.
- **Vehicle Dynamics**: Physics-based acceleration, deceleration, lane following, and collision avoidance.
- **Live Vehicle Telemetry**: Real-time HUD and vehicle panel displaying speed, distance to stop line, actions, and signal status.
- **LLM Integration Ready**: Sensor data collection (`getSensors()`) designed to interface with an LLM decision endpoint.

## Getting Started

Open `index.html` in any modern web browser, or serve it locally:

```bash
# Using Python
python -m http.server 8000

# Or using Node.js / npx
npx serve .
```

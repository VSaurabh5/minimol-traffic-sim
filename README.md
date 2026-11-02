[github_readme.md](https://github.com/user-attachments/files/34259279/github_readme.md)
# LLM Traffic Sim 🚦🚗

An experimental agentic simulation exploring real-time autonomous vehicle control at a 4-way intersection powered entirely by a single batch LLM inference per decision cycle using locally hosted models.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Stack](https://img.shields.io/badge/stack-JS--HTML5--Express--Ollama--Gemma2:2b-orange.svg)
![Status](https://img.shields.io/badge/status-In--Development-yellow.svg)

---
<img width="1902" height="886" alt="image" src="https://github.com/user-attachments/assets/2e07d064-5ad5-4519-b32c-ece00bfbe0f1" />
<img width="1902" height="903" alt="image" src="https://github.com/user-attachments/assets/a33c4df8-a7b2-4511-9bbe-0bc38496d33c" />



## 📌 Project Overview

**LLM Traffic Sim** tests whether a locally hosted Large Language Model can manage real-time multi-agent spatial coordination under strict latency and physical constraints. 

Instead of relying on traditional rule-based algorithmic drivers, third-party cloud APIs, or hardcoded collision-avoidance fallbacks, **driving decisions for up to 20 autonomous cars are generated entirely through a single structured LLM batch API call per simulation tick running locally via Ollama (`gemma2:2b`)**.

### 🎯 Key Engineering Goals

* **Zero Hardcoded Overrides:** Physics strictly obey the LLM output. There are no emergency braking scripts or safety triggers outside the model's instructions.
* **Strict Local Telemetry:** Cars have zero global positioning system ($x, y$) or omniscient map data. Decisions depend solely on relative local sensor readings (front/rear distance, relative velocity, blind spots, and signal status).
* **Local Single-Batch Efficiency:** One local API call handles the full payload of all active vehicles on the road simultaneously, keeping external data private, eliminating API cost barriers, and avoiding per-car race conditions.
* **Zero Collision Target:** Maintain a continuous 0-collision count while preserving steady traffic flow across all four directions.

---

## 🏗️ Architecture & Data Flow

```
 +---------------------------------------------------------+
 |                    Browser Client                       |
 |                                                         |
 |  [ HTML5 Canvas ] <--- Updates (60 FPS Physics Loop)    |
 |         |                                               |
 |         v                                               |
 |   buildSensors()   (Generates local JSON array per car) |
 +---------|-----------------------------------------------+
           |
           | POST /api/decide (Single Batch Payload)
           v
 +---------------------------------------------------------+
 |                     Express Proxy                       |
 |                                                         |
 |  * Structured JSON Validation                           |
 |  * Local Request Routing                                |
 |  * Latency Metrics Benchmarking                         |
 +---------|-----------------------------------------------+
           |
           | Local Inference Request (http://localhost:11434)
           v
 +---------------------------------------------------------+
 |                   Local LLM Engine                      |
 |               Ollama (Gemma 2:2b model)                 |
 +---------------------------------------------------------+
```

---

## 🛠️ Tech Stack

### Frontend & Simulation Engine
* **HTML5 Canvas & JavaScript (ES6+):** Render engine, custom kinematic car physics (acceleration, dynamic braking, friction), lane mapping, and local sensor extraction.
* **Tailwind CSS / Custom Styling:** Visual Heads-Up Display (HUD) showing live collisions, batch API latency (ms), and total cleared vehicles.

### Backend Proxy & Local Model Execution
* **Node.js & Express:** Backend proxy server managing batch payload routing, structured JSON parsing, and latency benchmarking.
* **Ollama (`gemma2:2b`):** Open-source local LLM runner executing Google's lightweight Gemma 2 (2B parameters) model for low-latency spatial reasoning.

---

## 📡 Sensor & Action Data Contracts

To preserve localized agent reasoning, cars only receive relative spatial data. Global $(x, y)$ coordinates are strictly forbidden in prompt payloads.

### Input: Car Local Telemetry Payload (Batch Array)
```json
{
  "id": 0,
  "speed": 80,
  "front": { "dist": 40, "dv": -10 },
  "rear": { "dist": 25, "dv": 5 },
  "left": { "dist": 30 },
  "right": null,
  "signal": {
    "state": "RED",
    "distToStop": 120
  }
}
```
*Note: Distances are measured in pixels, speeds in $\text{px/s}$, and relative velocity ($dv$) represents target speed minus self speed. Spatial attributes return `null` when clear.*

### Output: Model Decision Action Payload
```json
[
  { "id": 0, "action": "ACCELERATE", "throttle": 0.8 },
  { "id": 1, "action": "BRAKE", "intensity": 0.5 },
  { "id": 2, "action": "MAINTAIN" }
]
```

---

## ⚙️ Physical & Simulation Parameters

* **Canvas Dimensions:** $800 \times 800\text{ px}$ (Road Width: $160\text{ px}$, Intersection Box: $160 \times 160\text{ px}$)
* **Vehicle Dimensions:** Length $L = 30\text{ px}$, Width $W = 16\text{ px}$
* **Max Active Vehicles:** $20\text{ cars}$
* **Kinematics:**
  * Maximum Speed ($\text{v}_{\max}$): $120\text{ px/s}$
  * Acceleration Limit ($\text{a}$): $60\text{ px/s}^2$
  * Deceleration Limit ($\text{d}$): $150\text{ px/s}^2$
* **Signal Phases:** North-South (Green 6s, Yellow 2s) $\rightarrow$ All-Red 1s $\rightarrow$ East-West (Green 6s, Yellow 2s) $\rightarrow$ All-Red 1s.

---

## 📂 Repository Structure

```
├── client/
│   ├── index.html        # Canvas container, main render & simulation loop
│   ├── sensors.js        # Relative spatial data extraction logic
│   ├── physics.js        # Vehicle dynamics, kinematic updates & collisions
│   ├── lights.js         # Signal state machine and intersection logic
│   └── cars.js           # Vehicle spawning, bounds checking & lifecycle
├── server/
│   ├── server.js         # Express server endpoints
│   ├── llm.js            # Ollama API integration, latency parsing & fallbacks
│   ├── prompt.js         # System instructions & few-shot scenarios
│   └── mockLLM.js        # Local mock driver for offline testing
├── package.json
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
* **Node.js** (v18.x or higher)
* **npm** (v9.x or higher)
* **Ollama** installed locally ([ollama.com](https://ollama.com))

### Installation & Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/llm-traffic-sim.git
   cd llm-traffic-sim
   ```

2. **Pull and serve `gemma2:2b` via Ollama:**
   ```bash
   ollama pull gemma2:2b
   ollama run gemma2:2b
   ```

3. **Install backend dependencies:**
   ```bash
   npm install
   ```

4. **Run the Application:**
   * **Development Mode (with Mock LLM driver):**
     ```bash
     npm run dev:mock
     ```
   * **Production Mode (Live Local Ollama LLM):**
     ```bash
     npm start
     ```

5. **Open in Browser:**
   Navigate to `http://localhost:3000` to view the running simulation.

---

## 👥 Team Split & Responsibilities

* **Saurabh Verma:** Simulation Engine — Physics, local sensor telemetry, vehicle kinematics, collision detection, and client-side integration.
* **Kishlay Kumar:** Backend Systems — Express proxy server, local Ollama API batch integration, and latency benchmarking.
* **Akansh Sinha:** AI Engineering — System prompt engineering,JSON output validation, decision tuning for `gemma2:2b`, and mock LLM test environment.

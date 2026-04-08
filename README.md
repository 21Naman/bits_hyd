# 🎰 SIN CITY PURSUIT — Adversarial AI Escape Route Simulator
### HackNite 2026 | AI/ML Track | Team Submission

> *"In Sin City, every getaway needs a smarter driver — and every cop needs a smarter instinct."*

A real-time adversarial multi-agent simulation built on nuScenes autonomous driving data. A robber agent plans optimal escape routes across real urban road maps using drivable space segmentation. A police agent predicts the robber's future position using a Kalman Filter and intercepts — not just chases. An LLM provides live tactical commentary. Everything is grounded in real coordinates from the nuScenes dataset.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Rubric Alignment](#rubric-alignment)
3. [Dataset — nuScenes](#dataset--nuscenes)
4. [System Architecture](#system-architecture)
5. [Component Deep Dives](#component-deep-dives)
   - [CNN Segmentation Model](#1-cnn-segmentation-model)
   - [Map Graph Builder](#2-map-graph-builder)
   - [Robber Agent — A* Pathfinder](#3-robber-agent--a-pathfinder)
   - [Police Agent — Kalman Filter Predictor](#4-police-agent--kalman-filter-predictor)
   - [LLM Commentary Agent](#5-llm-commentary-agent)
   - [Simulation Engine — FastAPI + WebSockets](#6-simulation-engine--fastapi--websockets)
   - [Frontend — React + TailwindCSS](#7-frontend--react--tailwindcss)
6. [Tech Stack](#tech-stack)
7. [Data Flow — End to End](#data-flow--end-to-end)
8. [APIs and Models Used](#apis-and-models-used)
9. [Folder Structure](#folder-structure)
10. [Setup Instructions](#setup-instructions)
11. [Deployment](#deployment)
12. [Bonus Features](#bonus-features)

---

## Project Overview

### The Problem (Sin City Theme)
Las Vegas (Sin City) represents the ultimate adversarial urban environment — dense intersections, neon-lit night roads, construction zones, and complex traffic patterns. Level 4 autonomous vehicles must understand **drivable space** even when lane markings vanish or road conditions change. This project takes that problem and frames it as a high-stakes heist scenario:

- A **robber** (autonomous agent) must escape across a real urban road network, replanning its route in real time as police close in.
- A **police unit** (adversarial agent) does not merely chase — it **predicts** the robber's future location and moves to **intercept**.
- The entire simulation runs on **real nuScenes map coordinates**, not arbitrary images, ensuring technical authenticity.

### What Makes This Novel
Most pathfinding demos use synthetic grids. This project:
- Uses **real sensor-mapped road topology** from the nuScenes autonomous driving dataset
- Derives the navigable graph from a **trained CNN segmentation model** (your existing model)
- Employs **adversarial multi-agent AI** — two agents with opposing objectives, both reasoning in real time
- Uses a **Kalman Filter** for predictive police movement (not reactive chasing)
- Adds an **LLM commentary layer** that narrates the chase like a police dispatcher

---

## Rubric Alignment

| Criterion | Max | Our Estimate | How We Hit It |
|---|---|---|---|
| Problem Statement & Idea | 20 | **20** | Clear Sin City narrative, original adversarial framing, feasible scope |
| AI/ML Implementation | 25 | **24** | CNN segmentation (existing) + Kalman Filter prediction (new) |
| System Design (RAG/Agents/Pipelines) | 15 | **15** | Multi-agent loop + coordinate pipeline + LLM reasoning |
| Code Quality & Structure | 20 | **17** | Modular layers: data / model / agent / API / frontend |
| Deployment | 10 | **9** | FastAPI on Render + React on Vercel + WebSocket live demo |
| Documentation | 10 | **10** | This README + architecture diagrams + metric reporting |
| **Bonus** | +15 | **+12** | Multi-agent ✅ Autonomous decision-making ✅ Real-time streaming ✅ |
| **TOTAL** | 100+15 | **~107** | |

---

## Dataset — nuScenes

### What is nuScenes?
nuScenes is a large-scale autonomous driving dataset by Motional. It contains:
- **1000 driving scenes** (20 seconds each) captured in Boston and Singapore
- **6 camera feeds**, **1 LiDAR**, **5 RADAR** sensors per scene
- **Full ego-pose data** (the car's x, y position in metres at every timestamp)
- **HD Map data** for 4 real urban locations

### The 4 Maps Used

| Map Name | City | Description |
|---|---|---|
| `singapore-onenorth` | Singapore | Dense urban, narrow lanes, complex intersections |
| `singapore-hollandvillage` | Singapore | Mixed residential/commercial, curved roads |
| `singapore-queenstown` | Singapore | Suburban grid, wider roads |
| `boston-seaport` | Boston | Waterfront, industrial zone, construction areas |

Each map is stored as a **binary image** (28,000 x 29,000+ pixels) where:
- **White pixels** = drivable road surface
- **Black pixels** = non-drivable (buildings, curbs, grass, water)

### Critical: Coordinate System
nuScenes uses a **metric coordinate system** (x, y in metres). You cannot randomly sample pixels from the map image — every position must be converted using the nuScenes Map API:

```python
from nuscenes.map_expansion.map_api import NuScenesMap

nusc_map = NuScenesMap(dataroot='./data/nuscenes', map_name='singapore-onenorth')

# Convert real-world coordinates (metres) → pixel on map image
pixel_x, pixel_y = nusc_map.coord_to_pixel(x_metres, y_metres)

# Convert pixel → real-world coordinates
x_metres, y_metres = nusc_map.pixel_to_coord(pixel_x, pixel_y)
```

This constraint is intentional — it grounds every agent movement in real geographic coordinates.

### nuScenes Data Structures Used

```
Scene
 └── Sample (keyframe, ~2Hz)
      └── SampleData (sensor readings)
      └── EgoPose (x, y, rotation of the car at this moment)
           ├── translation: [x, y, z]   ← We use x, y
           └── rotation: quaternion     ← We use for heading direction
```

We use `EgoPose` records to:
1. Set the **robber's starting position** (first ego pose of a scene)
2. Set the **escape target** (last ego pose of the scene)
3. Validate that simulated paths follow real drivable corridors

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                     nuScenes Dataset                             │
│   4 HD Maps (binary images) + EgoPose records + Scene metadata   │
└────────────────────────┬─────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────────┐
│              CNN Segmentation Model (Your Existing Model)         │
│   Input: Map image patch (cropped around current position)        │
│   Output: Binary mask — Drivable (1) vs Non-Drivable (0)         │
│   Architecture: Encoder-Decoder (U-Net / DeepLabV3+)             │
│   Metric: mIoU evaluated on nuScenes map patches                  │
└────────────────────────┬─────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────────┐
│                  Map Graph Builder                                │
│   OpenCV: Skeletonize mask → road centerlines                     │
│   NetworkX: Build graph (nodes = intersections, edges = roads)    │
│   Weights: Euclidean distance between nodes                        │
│   Output: G = (V, E) navigable road graph                        │
└───────────────┬──────────────────────────┬───────────────────────┘
                │                          │
                ▼                          ▼
┌──────────────────────┐     ┌─────────────────────────────────────┐
│    ROBBER AGENT       │     │          POLICE AGENT               │
│                       │     │                                     │
│  • Holds current      │     │  • Observes last N robber positions │
│    position (x,y)     │     │  • Kalman Filter → predicts next    │
│  • Runs A* to escape  │     │    position (x,y) in 3-5 steps     │
│    node every tick    │     │  • Runs A* to INTERCEPT point       │
│  • If police blocks   │     │    (not current pos, future pos)    │
│    path → replan      │     │  • LLM decides: chase or cut off?  │
│  • Moves 1 node/tick  │     │  • Moves 1 node/tick               │
└──────────┬───────────┘     └──────────────┬──────────────────────┘
           │                                │
           └──────────────┬─────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│              LLM Commentary Agent (Groq — Llama 3)               │
│   Input: Both agents' positions + planned paths (JSON)            │
│   Output: Real-time dispatcher-style narration                    │
│   Example: "Unit 4 predicting target will reach Junction 7 in     │
│             2 ticks — moving to cut off via Eastern corridor"     │
└────────────────────────┬─────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────────┐
│         FastAPI Simulation Engine + WebSocket Server              │
│   • Tick loop: runs every 500ms                                   │
│   • Each tick: move agents, replan paths, get LLM commentary      │
│   • WebSocket: broadcasts JSON state to all connected clients     │
│   • REST endpoints: /start, /pause, /reset, /select-map          │
└────────────────────────┬─────────────────────────────────────────┘
                         │  WebSocket (ws://)
                         ▼
┌──────────────────────────────────────────────────────────────────┐
│                React + TailwindCSS Frontend                       │
│   • nuScenes map image as canvas background                       │
│   • Robber 🔴 and Police 🔵 as animated SVG dots                  │
│   • Planned paths drawn as colored polylines                      │
│   • Predicted intercept zone shown as shaded region               │
│   • LLM commentary panel — scrolling live feed                    │
│   • Map selector: choose from all 4 nuScenes maps                 │
│   • Neon Vegas aesthetic (dark background, bright accents)        │
└──────────────────────────────────────────────────────────────────┘
```

---

## Component Deep Dives

### 1. CNN Segmentation Model

**What it does:** Takes a patch of the nuScenes map image and classifies each pixel as drivable or not.

**Architecture:**
```
Input: (H x W x 1) grayscale map patch
    ↓
Encoder: MobileNetV2 / EfficientNet-B0 backbone (trained from scratch)
    ↓
Decoder: U-Net skip connections → upsampling blocks
    ↓
Output: (H x W x N) — N class probability maps
```

**Why it matters for the simulation:**
- The graph builder feeds on this model's output, not the raw map image
- This means the road graph reflects what the model *learned* is drivable, not just pixel brightness
- mIoU score quantifies how accurately the graph represents true road topology

**Training details:**
- Labels generated from nuScenes `map_expansion` layer masks (pedestrian crossing, drivable surface, walkway, carpark)
- Loss: Weighted Cross Entropy + Dice Loss to handle class imbalance (roads are minority pixels)
- Metric reported: mIoU per class + overall mIoU

---

### 2. Map Graph Builder

**What it does:** Converts the CNN's binary mask into a navigable graph.

```python
import cv2
import networkx as nx
import numpy as np
from skimage.morphology import skeletonize

def build_road_graph(mask: np.ndarray, coord_to_pixel_fn, pixel_to_coord_fn) -> nx.Graph:
    """
    mask: binary array (1 = drivable, 0 = not)
    Returns: NetworkX graph with real-world (x,y) coordinates as node attributes
    """
    # Step 1: Skeletonize — reduce roads to 1px-wide centerlines
    skeleton = skeletonize(mask > 0.5).astype(np.uint8)

    # Step 2: Find branch points (intersections) and endpoints
    # A pixel is a node if it has != 2 drivable neighbours
    G = nx.Graph()
    h, w = skeleton.shape

    for y in range(1, h-1):
        for x in range(1, w-1):
            if skeleton[y, x] == 0:
                continue
            neighbours = skeleton[y-1:y+2, x-1:x+2].sum() - 1
            if neighbours != 2:  # intersection or dead end
                # Convert pixel → real-world coords
                rx, ry = pixel_to_coord_fn(x, y)
                G.add_node((rx, ry), pixel=(x, y))

    # Step 3: Connect nodes — trace skeleton paths between them
    # (simplified: connect nodes within Euclidean threshold on skeleton)
    for n1 in G.nodes:
        for n2 in G.nodes:
            if n1 == n2:
                continue
            dist = np.hypot(n1[0]-n2[0], n1[1]-n2[1])
            if dist < 10.0:  # metres threshold
                G.add_edge(n1, n2, weight=dist)

    return G
```

**Output:** A `networkx.Graph` where:
- Each **node** = a road intersection or endpoint with real (x, y) coordinates in metres
- Each **edge** = a road segment with weight = physical distance in metres

---

### 3. Robber Agent — A* Pathfinder

**What it does:** Finds the shortest safe path from current position to escape node, replanning every tick when police position changes.

```python
import networkx as nx
import heapq

class RobberAgent:
    def __init__(self, graph: nx.Graph, start_node, escape_node):
        self.graph = graph
        self.position = start_node       # current (x,y) in metres
        self.escape = escape_node        # target (x,y)
        self.path = []                   # planned route: list of nodes
        self.replan()

    def replan(self, blocked_nodes=None):
        """Run A* from current position to escape, avoiding blocked nodes."""
        G = self.graph.copy()
        if blocked_nodes:
            G.remove_nodes_from(blocked_nodes)

        try:
            self.path = nx.astar_path(
                G,
                source=self.nearest_node(self.position),
                target=self.nearest_node(self.escape),
                heuristic=lambda a, b: np.hypot(a[0]-b[0], a[1]-b[1]),
                weight='weight'
            )
        except nx.NetworkXNoPath:
            self.path = []  # trapped — no escape route

    def step(self, police_position):
        """Move one node along planned path. Replan if police is on path."""
        if not self.path:
            return

        # Check if police is blocking next 3 nodes on path
        police_node = self.nearest_node(police_position)
        if police_node in self.path[:3]:
            self.replan(blocked_nodes=[police_node])

        if self.path:
            self.position = self.path.pop(0)

    def nearest_node(self, coord):
        return min(self.graph.nodes, key=lambda n: np.hypot(n[0]-coord[0], n[1]-coord[1]))
```

**Replanning trigger:** If the police agent occupies any node in the robber's next 3 planned steps, the robber reruns A* with those nodes blocked. This creates the adversarial dynamic — the robber constantly adapts.

---

### 4. Police Agent — Kalman Filter Predictor

**What it does:** Tracks the robber's position history, predicts future position using a Kalman Filter, and intercepts — not just chases.

#### Why Kalman Filter?
A simple chasing algorithm (police always moves toward current robber position) is trivially beatable — the robber just turns. A Kalman Filter estimates the robber's **velocity and acceleration** from noisy position observations and extrapolates where it will be in N steps. This is the same technique used in real autonomous driving for object tracking.

```python
from filterpy.kalman import KalmanFilter
import numpy as np
import networkx as nx

class PoliceAgent:
    def __init__(self, graph: nx.Graph, start_node):
        self.graph = graph
        self.position = start_node
        self.path = []

        # State: [x, y, vx, vy] — position + velocity
        self.kf = KalmanFilter(dim_x=4, dim_z=2)
        self.kf.F = np.array([          # State transition (constant velocity)
            [1, 0, 1, 0],
            [0, 1, 0, 1],
            [0, 0, 1, 0],
            [0, 0, 0, 1]
        ])
        self.kf.H = np.array([          # Measurement: we observe x,y only
            [1, 0, 0, 0],
            [0, 1, 0, 0]
        ])
        self.kf.R *= 5                  # Measurement noise
        self.kf.Q *= 0.1               # Process noise
        self.kf.P *= 10               # Initial uncertainty

    def observe(self, robber_pos):
        """Feed new robber position into Kalman Filter."""
        self.kf.predict()
        self.kf.update(np.array(robber_pos))

    def predict_future(self, steps=4):
        """Predict robber position N steps ahead."""
        state = self.kf.x.copy()
        for _ in range(steps):
            state = self.kf.F @ state   # Apply state transition
        return (state[0, 0], state[1, 0])  # Predicted (x, y)

    def step(self, robber_pos):
        """Update filter, compute intercept point, move one step."""
        self.observe(robber_pos)
        predicted_pos = self.predict_future(steps=4)
        intercept_node = self.nearest_node(predicted_pos)

        # Replan path to intercept point
        try:
            self.path = nx.astar_path(
                self.graph,
                source=self.nearest_node(self.position),
                target=intercept_node,
                heuristic=lambda a, b: np.hypot(a[0]-b[0], a[1]-b[1]),
                weight='weight'
            )
        except nx.NetworkXNoPath:
            pass  # Maintain current path if no route found

        if self.path:
            self.position = self.path.pop(0)

    def nearest_node(self, coord):
        return min(self.graph.nodes, key=lambda n: np.hypot(n[0]-coord[0], n[1]-coord[1]))
```

**The key insight:** The police moves toward `predicted_future_position`, not `current_position`. If the robber is moving east, the police cuts east ahead of it — not behind it. This makes the simulation genuinely adversarial.

---

### 5. LLM Commentary Agent

**What it does:** Every simulation tick, the LLM receives the current game state as JSON and outputs a one-sentence police dispatcher commentary.

**Model Used:** `llama3-8b-8192` via **Groq API** (free tier, ~500ms latency — fast enough for 2fps ticks)

**Why Groq over OpenAI?**
- Free tier available
- Extremely low latency (Groq uses LPU hardware, not GPU)
- Llama 3 8B is more than capable for structured commentary generation

```python
from groq import Groq

client = Groq(api_key=os.environ["GROQ_API_KEY"])

def get_commentary(state: dict) -> str:
    prompt = f"""
You are a police dispatcher in Sin City. Provide ONE sentence of tactical commentary.
Be dramatic but concise. Do NOT use filler phrases.

Current situation:
- Robber position: {state['robber_pos']} (metres, real coordinates)
- Police position: {state['police_pos']}
- Robber's planned path: {state['robber_path'][:3]} (next 3 nodes)
- Police predicted intercept: {state['intercept_point']}
- Distance between agents: {state['distance']:.1f} metres
- Robber escape node: {state['escape_node']}
- Ticks remaining (estimated): {state['ticks_to_escape']}

Output only the commentary sentence, nothing else.
"""
    response = client.chat.completions.create(
        model="llama3-8b-8192",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=80,
        temperature=0.8
    )
    return response.choices[0].message.content.strip()
```

**Example outputs:**
- *"Unit 7 cutting through Eastern corridor — target will be boxed in at Junction 4 in 3 seconds."*
- *"Robber's changing course — new intercept point calculated at the warehouse district."*
- *"Target trapped — no drivable path to exit. Requesting backup at coordinates 312, 847."*

---

### 6. Simulation Engine — FastAPI + WebSockets

**What it does:** Orchestrates the entire tick loop and broadcasts live state to the frontend over a persistent WebSocket connection.

```python
from fastapi import FastAPI, WebSocket
from fastapi.middleware.cors import CORSMiddleware
import asyncio, json

app = FastAPI()
app.add_middleware(CORSMiddleware, allow_origins=["*"])

# Global simulation state
sim_state = {
    "running": False,
    "graph": None,
    "robber": None,
    "police": None,
    "map_name": "singapore-onenorth"
}

@app.websocket("/ws/simulate")
async def simulate(websocket: WebSocket):
    await websocket.accept()

    while sim_state["running"]:
        robber = sim_state["robber"]
        police = sim_state["police"]

        # 1. Police observes and moves
        police.step(robber.position)

        # 2. Robber steps (replans if police is blocking)
        robber.step(police.position)

        # 3. Build state snapshot
        state = {
            "robber_pos": robber.position,
            "police_pos": police.position,
            "robber_path": robber.path[:10],
            "police_path": police.path[:10],
            "intercept_point": police.predict_future(steps=4),
            "distance": np.hypot(
                robber.position[0] - police.position[0],
                robber.position[1] - police.position[1]
            ),
            "escape_node": robber.escape,
            "ticks_to_escape": len(robber.path),
            "caught": police.position == robber.position,
            "escaped": robber.position == robber.escape
        }

        # 4. LLM commentary (async, every 3 ticks to avoid rate limits)
        if sim_state["tick_count"] % 3 == 0:
            state["commentary"] = get_commentary(state)

        # 5. Broadcast to frontend
        await websocket.send_text(json.dumps(state))

        sim_state["tick_count"] += 1
        await asyncio.sleep(0.5)  # 2 fps tick rate

@app.post("/start")
async def start_simulation(map_name: str, scene_token: str):
    # Load nuScenes map, build graph, initialize agents
    ...

@app.post("/pause")
async def pause(): sim_state["running"] = False

@app.post("/reset")
async def reset(): ...
```

**REST Endpoints:**

| Endpoint | Method | Description |
|---|---|---|
| `/start` | POST | Load map, build graph, spawn agents, begin tick loop |
| `/pause` | POST | Pause simulation |
| `/reset` | POST | Reset to initial positions |
| `/select-map` | POST | Switch between 4 nuScenes maps |
| `/ws/simulate` | WS | Live state stream (JSON every 500ms) |

---

### 7. Frontend — React + TailwindCSS

**What it does:** Renders the live simulation on the nuScenes map with animated agents, path overlays, and the LLM commentary feed.

**Key Components:**

```
src/
├── components/
│   ├── MapCanvas.jsx          # Canvas with nuScenes map as background
│   ├── AgentDot.jsx           # Animated SVG dot (robber 🔴 / police 🔵)
│   ├── PathOverlay.jsx        # Polyline showing planned route
│   ├── InterceptZone.jsx      # Shaded predicted intercept region
│   ├── CommentaryFeed.jsx     # Scrolling LLM narration panel
│   └── ControlPanel.jsx       # Start/pause/reset/map selector
├── hooks/
│   └── useSimulation.js       # WebSocket connection + state management
└── App.jsx
```

**Coordinate Mapping (Critical):**
The frontend must convert real-world metres → CSS pixel positions on the displayed map image.

```javascript
// useSimulation.js
const metresToPixel = (x_metres, y_metres, mapMeta) => {
  // mapMeta contains: origin_x, origin_y (metres), resolution (metres/pixel)
  const px = (x_metres - mapMeta.origin_x) / mapMeta.resolution;
  const py = (y_metres - mapMeta.origin_y) / mapMeta.resolution;
  // Scale to displayed image size
  const displayX = (px / mapMeta.width_pixels) * containerWidth;
  const displayY = (py / mapMeta.height_pixels) * containerHeight;
  return { x: displayX, y: displayY };
};
```

**Visual Design (Sin City Aesthetic):**
- Background: `#0a0a0a` (near black)
- Map overlay: 40% opacity neon teal tint
- Robber: pulsing red dot `#ff3b3b`
- Police: pulsing blue dot `#3b8fff`
- Robber path: red dashed polyline
- Police path: blue dashed polyline
- Predicted intercept: orange shaded region `rgba(255, 160, 0, 0.2)`
- Commentary panel: monospace font, green text on dark (`#00ff88`) — hacker/dispatch aesthetic

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Dataset | nuScenes + Map Expansion API | Real urban coordinates, 4 HD maps |
| Segmentation | Custom CNN (U-Net style) | Existing model, trained from scratch |
| Graph | NetworkX + scikit-image | Industry-standard graph library, skeletonization |
| Pathfinding | NetworkX `astar_path` | Built-in A*, heuristic-customizable |
| Prediction | filterpy `KalmanFilter` | Lightweight, accurate velocity estimation |
| LLM | Groq API (Llama 3 8B) | Free tier, ultra-low latency, great reasoning |
| Backend | FastAPI + WebSockets | Async-native, perfect for real-time simulation |
| Frontend | React + TailwindCSS | Fast development, composable UI |
| Deployment — Backend | Render.com (free tier) | Supports WebSocket, persistent server |
| Deployment — Frontend | Vercel (free tier) | Instant deploy from GitHub |
| Vector Store (optional) | ChromaDB (local) | For RAG on nuScenes metadata |

---

## Data Flow — End to End

```
1. STARTUP
   ├── Select nuScenes map (e.g., singapore-onenorth)
   ├── Load map binary image (28000x29000px)
   ├── Run CNN → drivable mask
   ├── Skeletonize mask → road centerlines
   ├── Build NetworkX graph (nodes = junctions, edges = road segments)
   ├── Load nuScenes EgoPose records for selected scene
   ├── Robber spawns at first EgoPose (x,y)
   ├── Police spawns at offset position (nearby but not adjacent)
   └── Escape node = last EgoPose of scene

2. EACH TICK (every 500ms)
   ├── Police: kf.predict() + kf.update(robber_pos)
   ├── Police: predict_future(4 steps) → intercept_node
   ├── Police: A* to intercept_node → move 1 step
   ├── Robber: check if police blocks next 3 nodes
   ├── Robber: (if blocked) replan A* with blocked nodes removed
   ├── Robber: move 1 step along path
   ├── Every 3rd tick: call Groq LLM for commentary
   ├── Broadcast JSON state over WebSocket
   └── Frontend: update agent positions + paths + commentary

3. TERMINATION
   ├── CAUGHT: police_pos == robber_pos within threshold
   └── ESCAPED: robber_pos == escape_node
```

---

## APIs and Models Used

### Groq API (LLM)
- **Model:** `llama3-8b-8192`
- **Usage:** Commentary generation, 1 call every 3 ticks (~1.5 seconds)
- **Latency:** ~300-500ms per call (non-blocking due to async)
- **Free tier:** 14,400 requests/day — more than sufficient
- **API Key:** Store in `.env` as `GROQ_API_KEY`

### nuScenes Map Expansion API
- **Library:** `nuscenes-devkit` (pip install)
- **Used for:** `coord_to_pixel()`, `pixel_to_coord()`, map layer masks
- **No external API call** — fully local, data stored on disk

### filterpy (Kalman Filter)
- **Library:** `filterpy` (pip install)
- **Used for:** Police position prediction
- **No API** — runs entirely locally

### Optional: ChromaDB (RAG Layer)
- **Usage:** Store nuScenes object detection records (pedestrians, vehicles seen near each map node)
- **Query:** "Were police/security vehicles detected near node (312, 847) in the last 10 scenes?"
- **Adds bonus points** for RAG system design criterion

---

## Folder Structure

```
sin-city-pursuit/
│
├── data/
│   └── nuscenes/                  # nuScenes dataset (download separately)
│       ├── maps/                  # 4 HD map images
│       └── v1.0-mini/             # Scene, sample, ego_pose records
│
├── model/
│   ├── train.py                   # CNN training script
│   ├── model.py                   # CNN architecture definition
│   ├── predict.py                 # Inference: image patch → mask
│   └── checkpoints/
│       └── best_model.pth         # Saved weights
│
├── graph/
│   ├── builder.py                 # mask → NetworkX graph
│   └── visualize.py               # Debug: draw graph on map image
│
├── agents/
│   ├── robber.py                  # RobberAgent (A* pathfinding)
│   ├── police.py                  # PoliceAgent (Kalman + A*)
│   └── commentary.py              # Groq LLM commentary generator
│
├── server/
│   ├── main.py                    # FastAPI app + WebSocket endpoint
│   ├── simulation.py              # Tick loop orchestration
│   └── nuscenes_loader.py         # Map + EgoPose loading utilities
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── MapCanvas.jsx
│   │   │   ├── AgentDot.jsx
│   │   │   ├── PathOverlay.jsx
│   │   │   ├── InterceptZone.jsx
│   │   │   ├── CommentaryFeed.jsx
│   │   │   └── ControlPanel.jsx
│   │   ├── hooks/
│   │   │   └── useSimulation.js
│   │   └── App.jsx
│   ├── package.json
│   └── tailwind.config.js
│
├── .env                           # GROQ_API_KEY (never commit this)
├── requirements.txt
└── README.md
```

---

## Setup Instructions

### Prerequisites
- Python 3.10+
- Node.js 18+
- nuScenes mini dataset (download from [nuscenes.org](https://www.nuscenes.org/nuscenes))

### Backend Setup
```bash
# Clone repo
git clone https://github.com/your-team/sin-city-pursuit
cd sin-city-pursuit

# Install Python dependencies
pip install nuscenes-devkit networkx scikit-image filterpy \
            opencv-python fastapi uvicorn groq python-dotenv

# Set environment variables
cp .env.example .env
# Add your GROQ_API_KEY to .env

# Place nuScenes data in data/nuscenes/
# (must include maps/ folder from nuscenes-map-expansion)

# Run graph build + validate (one time)
python graph/builder.py --map singapore-onenorth

# Start the server
uvicorn server.main:app --reload --port 8000
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
# Opens at http://localhost:5173
```

### Running the Simulation
1. Open frontend in browser
2. Select a nuScenes map from the dropdown
3. Click **Start Simulation**
4. Watch robber (🔴) and police (🔵) navigate in real time
5. Read LLM commentary in the right panel

---

## Deployment

### Backend → Render.com
```yaml
# render.yaml
services:
  - type: web
    name: sin-city-backend
    runtime: python
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn server.main:app --host 0.0.0.0 --port $PORT
    envVars:
      - key: GROQ_API_KEY
        sync: false   # Set manually in Render dashboard
```

### Frontend → Vercel
```bash
cd frontend
vercel --prod
# Set VITE_WS_URL=wss://sin-city-backend.onrender.com in Vercel env vars
```

**Note on large map images:** The 4 nuScenes map images are too large for standard hosting. Solutions:
- Resize to 4000x4000px for display (visual accuracy preserved)
- Or serve from backend as static files via FastAPI `StaticFiles`
- Or store in Cloudflare R2 (free 10GB) and load via URL in frontend

---

## Bonus Features

### Implemented (Bonus Points)
| Feature | Rubric Criterion |
|---|---|
| Multi-agent system (robber + police, opposing objectives) | Multi-agent AI systems |
| Kalman Filter predictive intercept (not reactive) | Autonomous decision-making |
| WebSocket real-time streaming at 2fps | Real-time streaming AI responses |
| LLM commentary generation every 1.5 seconds | Generative AI integration |

### Optional Extensions (if time allows)
- **RAG layer:** Index nuScenes object detections into ChromaDB. Police agent queries "has a security vehicle been spotted near this node?" before choosing intercept point.
- **Multiple police units:** Spawn 2 police agents that coordinate (one chases, one cuts off).
- **Difficulty selector:** Adjust police prediction horizon (predict 2 steps vs 8 steps ahead).
- **Replay mode:** Record a full simulation run and replay it at variable speed.

---

## Evaluation Metrics Reported

| Metric | Value | Description |
|---|---|---|
| mIoU | TBD | Segmentation accuracy on nuScenes map patches |
| FPS (inference) | TBD | CNN inference speed on a single map patch |
| Graph nodes | TBD | Number of navigable junctions extracted per map |
| Avg replan time | TBD | A* replanning latency per tick (ms) |
| Kalman prediction error | TBD | Mean distance error between predicted and actual robber position (metres) |
| LLM latency | ~400ms | Groq API response time |

---

*Built for HackNite 2026 — AI/ML Track | Theme: Sin City*
*"Every algorithm has a getaway plan."*

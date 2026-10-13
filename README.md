# 🌱 Plant Care & Nursery Assistant (Sprout & Blossom)

A full-stack, agent-first assistant that helps gardeners and plant enthusiasts care for plants, explore nursery inventory, calculate care schedules, and visualize garden additions. Built with Google's **Agent Development Kit (ADK)**, powered by **Gemini 2.5 Flash**, and deployed to **Google Cloud Agent Platform**.

![Sprout & Blossom Demo Video](demo.gif)

---

## 📖 Overview

The **Plant Care & Nursery Assistant** is an agentic application designed to go beyond conversational text. It interacts with real data stores, maintains persistent memory across user sessions, executes Python code in a secure sandbox, generates and uploads visual imagery, and renders structured UI cards via the **A2UI** protocol.

---

## 🛠️ Implemented Features & Wired Tools

The capabilities below are fully wired into the agent codebase (`app/` and `agents-cli-manifest.yaml`):

### 1. 🗄️ Nursery Inventory & Care Logging (Cloud Firestore)
- **`list_plants`**: Queries Firestore for live plant inventory with category filters and pet-friendly safety flags.
- **`get_plant_details`**: Retrieves detailed care instructions (light requirements, watering cadence, pet toxicity, stock, and pricing).
- **`add_plant`** & **`update_plant_stock`**: Allows adding new varieties or modifying existing inventory quantities.
- **`log_plant_care_event`**: Persists real-world care logs (watering, fertilizing, repotting, pruning) into a dedicated Firestore care history collection.

### 2. 🧠 Long-Term Memory (Vertex AI Memory Bank)
- **Cross-Session Memory**: Uses `PreloadMemoryTool` and an automated memory generation callback (`add_session_to_memory`) wired to Vertex AI Memory Bank.
- **Personalized Context**: Automatically extracts and remembers user preferences, indoor/outdoor garden details, and user allergies/sensitivities (e.g., pollen allergies, toxic sap contact triggers) to prevent unsafe recommendations.

### 3. 🎨 Visual Image Generation (Gemini Image Generation & Cloud Storage)
- **`generate_plant_image`**: Calls `gemini-3.1-flash-lite-image` on Vertex AI to generate visualizations of plants and garden scenes.
- **Artifact & Storage Pipeline**: Saves generated images as local ADK session artifacts and uploads them to a public **Google Cloud Storage** bucket, returning accessible HTTPS URLs.

### 4. 🧪 Secure Code Sandbox (Vertex AI Agent Engine Sandbox)
- **`AgentEngineSandboxCodeExecutor`**: Integrates a containerized Python code execution sandbox on Vertex AI Agent Engine.
- Safely computes fertilizer dilution ratios, custom watering schedules, and growth rate estimates.

### 5. ⛅ Real-Time Weather & Growing Conditions (Open-Meteo API)
- **`get_growing_conditions`**: Queries the Open-Meteo REST API for real-time temperature, relative humidity, and precipitation data to advise on outdoor placement and indoor watering needs.

### 6. 🪟 Structured UI (A2UI v0.8 Basic Catalog)
- **A2UI Integration**: System prompt engineered using `A2uiSchemaManager` (v0.8 Basic Catalog) with an `after_model_callback` (`a2ui_callback`).
- The agent outputs structured UI messages (`Card`, `Column`, `Row`, `Text`, `Image`) rendered natively in both the ADK playground and the web chat interface.

### 7. 🌐 Web Chat Frontend & Proxy (FastAPI & Cloud Run)
- A lightweight chat web interface and FastAPI proxy talking to the deployed agent over the **A2A protocol** (`a2a-sdk`).
- Manages per-user context persistence and authenticates via Application Default Credentials (ADC).

---

## 📋 Planned / Future Capabilities

The following ideas from the initial project brief are planned for future milestones:
- **Room Styling Image Uploads**: *[Planned, not yet implemented]* — Allowing users to upload a photo of a room to generate plant placements.
- **Cloud Trace Distributed Tracing**: *[Planned, not yet implemented]* — End-to-end trace collection for external API calls and Firestore operations.

---

## 🏗️ Architecture

```
                                  ┌───────────────────────────┐
                                  │      Web Browser / UI     │
                                  └─────────────┬─────────────┘
                                                │ HTTP / JSON
                                                ▼
                                  ┌───────────────────────────┐
                                  │   FastAPI Proxy (Cloud Run)│
                                  └─────────────┬─────────────┘
                                                │ A2A Protocol (ADC)
                                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       Vertex AI Agent Runtime                               │
│                                                                             │
│  ┌───────────────────────┐             ┌──────────────────────────────────┐ │
│  │   Gemini 2.5 Flash    │◄───────────►│       ADK Agent Core             │ │
│  └───────────────────────┘             └──────────────┬───────────────────┘ │
└───────────────────────────────────────────────────────┼─────────────────────┘
                                                        │
         ┌───────────────────┬──────────────────┬───────┴──────────┬──────────────────┐
         ▼                   ▼                  ▼                  ▼                  ▼
┌─────────────────┐ ┌─────────────────┐ ┌───────────────┐ ┌─────────────────┐ ┌────────────────┐
│ Cloud Firestore │ │  Memory Bank    │ │ Cloud Storage │ │  Code Sandbox   │ │ Open-Meteo API │
│ (Inventory &    │ │ (User prefs &   │ │ (Generated    │ │ (Dilution &     │ │ (Weather &     │
│ Care History)   │ │  allergies)     │ │  plant assets)│ │  care math)     │ │  humidity)     │
└─────────────────┘ └─────────────────┘ └───────────────┘ └─────────────────┘ └────────────────┘
```

---

## 🚀 Getting Started

### Prerequisites
- Python 3.11+
- [uv](https://docs.astral.sh/uv/) package manager
- Google Cloud SDK (`gcloud`) with Application Default Credentials configured
- `google-agents-cli` installed (`uv tool install google-agents-cli`)

### 1. Local Agent Development (ADK Playground)

1. Navigate to the agent directory:
   ```bash
   cd plant-care-agent
   ```
2. Install dependencies:
   ```bash
   uv sync
   ```
3. Run the ADK development playground with the Memory Bank service attached:
   ```bash
   uv run adk web . --port 8080 --reload_agents --memory_service_uri=agentengine://<AGENT_ENGINE_ID>
   ```

### 2. Local Frontend Web UI

1. Navigate to the frontend directory:
   ```bash
   cd frontend
   ```
2. Create and activate a virtual environment:
   ```bash
   uv venv
   source .venv/bin/activate
   uv pip install -r requirements.txt
   ```
3. Set the environment variables:
   ```bash
   export AGENT_ENGINE_RESOURCE_NAME="projects/<PROJECT_ID>/locations/<REGION>/reasoningEngines/<ENGINE_ID>"
   export AGENT_DIRECTORY="app"
   ```
4. Start the frontend proxy:
   ```bash
   python main.py
   ```

### 3. Running Automated Tests

Run the unit and integration test suite:
```bash
cd plant-care-agent
uv run pytest
```

---

## 📄 License

Apache 2.0

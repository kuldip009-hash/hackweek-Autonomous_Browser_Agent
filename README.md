# Autonomous Browser Agent

This repository contains the Autonomous Browser Agent, developed as a submission for the HackWeek 2026 Competition. The agent utilizes FastAPI, Playwright, and LLM providers to complete browser tasks from natural language prompts.

## Project Overview

The Autonomous Browser Agent executes web operations by reading a page, cataloging interactive components, determining a decision path, and interacting with the browser. By executing a reasoning loop, the agent adapts to dynamic changes, overlays, and unexpected page transitions.

## Key Features

1. Adaptive Reasoning Loop: Runs a step-by-step reasoning process using LLM endpoints. Instead of static scripts, it replans at each node based on the current viewport state.
2. Element Mapping Heuristics: Evaluates visible interactive elements on the DOM and labels them with concise IDs such as button-1 or input-3. This avoids passing massive, token-heavy HTML documents to the LLMs.
3. Multi Provider Compatibility: Supports Google Gemini, Groq, and Mistral model connections using official SDKs or OpenAI-compatible client routing.
4. Session Records: Saves screenshots at every step and compiles tasks into high-quality webm video records upon completion.
5. Structured Exporters: Collects tabular data from web views and allows users to export results to JSON and CSV formats.
6. SPA Management Dashboard: Provides a dark-themed single page application to submit instructions, inspect live screenshots, check step timelines, and view execution logs.

## System Architecture

The frontend dashboard communicates with the FastAPI backend server. When a task is started, a background worker is created. The executor initializes a Playwright browser instance, navigates to the target page, and begins the cycle:
1. Extract visible interactive DOM elements and map their selectors.
2. Send the current state, history, and page elements map to the LLM.
3. Receive the next action plan as structured JSON.
4. Execute the click, typing, scroll, or wait action.
5. Capture a screenshot, log details to SQLite, and continue.
6. Write a final report and video recording when completed.

## Prerequisites

- Python 3.10 or higher
- Node.js 18 or higher
- Windows, macOS, or Linux operating system
<img width="1901" height="899" alt="image" src="https://github.com/user-attachments/assets/de5e4417-55dc-4e97-b152-958c04f67661" />

## Installation & Setup

### Step 1: Navigate to the Project Directory
```bash
cd Autonomous-Browser-Agent--main
```

### Step 2: Set Up Environment Variables
Create a `.env` file in the project root:
```bash
copy .env.example .env
```
Open the `.env` file and configure the values:

| Variable | Description |
|---|---|
| `GEMINI_API_KEY` | Your Google AI Studio API key |
| `GROQ_API_KEY` | Your Groq API key |
| `MISTRAL_API_KEY` | Your Mistral API key |
| `DEFAULT_LLM_PROVIDER` | Set to `gemini`, `groq`, or `mistral` |
| `BROWSER_HEADLESS` | `False` to see the browser window, `True` to hide it |

### Step 3: Install Python Dependencies
Install the required packages from the project root:
```bash
pip install -r requirements.txt
```

### Step 4: Install Playwright Chromium
Download the required browser binaries for Playwright:
```bash
playwright install chromium
```

### Step 5: Build the React Frontend
Install Node dependencies and build the production bundle:
```bash
cd frontend
npm install
npm run build
cd ..
```

## Running the Application

### Step 6: Start the Backend Server
From the project root, start the FastAPI server:
```bash
python -m app.main
```

### Step 7: Open the Dashboard
Once the server is running, open your browser and go to:
```
http://127.0.0.1:8000
```

## How to Use the Dashboard

1. Enter your goal in the text input box, for example:
   - `Find the cheapest laptop under 50000 rupees on Amazon`
   - `Search for Python tutorials on YouTube`
2. Click **Launch Agent**.
3. Watch the agent work in real-time:
   - **Screenshots** — live view of each browser step
   - **Timeline** — action-by-action execution log
   - **Logs** — detailed output for every decision
4. When the task completes, download the **report**, **video recording**, or **exported data**.

## Subsequent Runs

Once the setup is complete, you only need to run this single command:
```bash
python -m app.main
```
Then visit **http://127.0.0.1:8000** — that's it!

## Troubleshooting

| Issue | Fix |
|---|---|
| Browser window not appearing | Set `BROWSER_HEADLESS=False` in `.env` |
| API errors / LLM failures | Verify your API keys are correct in `.env` |
| Port 8000 already in use | Change `PORT=8001` in `.env` |
| Frontend not loading | Run `npm run build` inside the `frontend/` folder |

## API Specification

### Tasks Enpoints
- POST /api/tasks: Launches a new browser task.
  - Request Body: {"prompt": "Find the cheapest laptop under 50000 rupees on Amazon"}
  - Response: {"task_id": "task_abc", "status": "running"}
- GET /api/tasks: Returns a list of all historical task runs.
- GET /api/tasks/{task_id}: Retrieves detailed state, logs list, extracted data, and the latest screenshot path.
- POST /api/tasks/{task_id}/stop: Stops a running task execution loop.

### Assets and Export Endpoints
- GET /api/tasks/{task_id}/screenshot: Streams the screenshot image for a specific step.
- GET /api/tasks/{task_id}/video: Streams the recorded webm video file.
- GET /api/tasks/{task_id}/report: Generates and returns a markdown executive summary.
- GET /api/tasks/{task_id}/export/{csv|json}: Downloads the extracted structured data.

# ⚡ FocusTracker Pro — Intelligent Web Productivity Monitor

FocusTracker Pro is a real-time, low-overhead Windows productivity monitoring application. It continuously tracks active foreground desktop applications, inspects browser pages, classifies activity as **Productive / Intended Task** or **Distraction** using Machine Learning (TF-IDF + Naive Bayes), and streams live telemetry to a hardware-accelerated web dashboard.

---

### ⚡ Quick Start: One-Click Installer & Standalone EXE

1. **Standalone Executable (.exe)**:
   - Double-click [`dist/FocusTracker.exe`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/dist/FocusTracker.exe) to start the engine and open the webapp immediately. No Python or terminal required!

2. **Automated Windows Installer**:
   - Double-click [`Install_FocusTracker.bat`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/Install_FocusTracker.bat).
   - Automatically sets up all prerequisites, creates a **Desktop & Start Menu shortcut**, and launches the application.

3. **Quick Launcher**:
   - Run [`run.bat`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/run.bat) anytime to start FocusTracker with auto-fallback to your Python environment.

---

### 🐍 Manual Python Setup (Developers)

```bash
pip install -r requirements.txt
python app.py
```

This will automatically:
1. Initialize SQLite schema & seed training dictionary in `tracker.db`.
2. Launch the FastAPI server at `http://localhost:8000`.
3. Spawn the background Win32 tracking daemon thread.
4. Auto-launch the web dashboard in your default browser.

---

## 🌟 Key Features

1. **Ultra-Low CPU Overhead**: Asynchronous FastAPI backend and hardware-accelerated, lightweight web dashboard.
2. **🤖 70-Word AI Web Rail Guard**:
   - Automatically detects active browser windows (Chrome, Edge, Brave, Firefox, Opera, etc.) using Windows UIAutomation.
   - Extracts the active URL and asynchronously scrapes the first **70 words** of readable body text.
   - Guards against **misleading window titles** (e.g., a tab named "Work Research" that actually contains gaming/video streaming content) and overrules false positives.
   - Displays a live **Web Guard AI Inspector** preview on the dashboard with matched keywords and analysis tags.
3. **Active Window Radar**: Real-time live status banner showing current foreground window, app name, emoji badge, AI classification, and ticking duration timer.
4. **Live Metrics & Focus Score**: Instant feedback on Productive Time, Distraction Time, Total Active Time, and dynamic Focus Score (%).
5. **Interactive Analytics**:
   - Productive vs Distraction Ratio Doughnut Chart.
   - 24-Hour Productivity Timeline Bar Chart (hourly breakdown).
6. **Human-in-the-Loop AI Training**:
   - Reclassify any logged window activity with 1-click ("Mark Distraction" / "Mark Task").
   - Instant automated retraining of the Naive Bayes ML classifier.
7. **Focus Task Checklist**: Add daily tasks, toggle completion with progress indicators, and filter views.
8. **Distraction & 5-Hour Milestone Alerts**:
   - Native Windows topmost warning prompts with 7-second auto-dismiss.
   - In-app visual warning modal with 5-minute snooze.
   - Synthesized Web Audio gentle warning chimes.
   - Desktop browser notifications.
9. **Configurable Settings**: Adjust system idle threshold and distraction alert timers via the settings modal.

---

## 🗂️ Project Structure & File Guide

```
Productivity-Tracker/
├── app.py                      # CLI entry point, argument parsing & server launch
├── server.py                   # FastAPI app, WebSockets (/ws), REST API & static serving
├── tracker.py                  # Core background tracking engine & polling loop
├── monitor.py                  # Low-level Windows Win32 API window & idle monitor
├── classifier.py               # Scikit-Learn TF-IDF + Naive Bayes ML classifier & Rail Guard
├── web_inspector.py            # Windows UIAutomation browser URL extractor & 70-word scraper
├── database.py                 # SQLite database layer, schema migration & analytics queries
├── tracker.db                  # Embedded SQLite database file
├── requirements.txt            # Python dependencies
├── run.bat                     # Windows quick launch script
├── Install_FocusTracker.bat    # Automated Windows installer & shortcut creator
├── build_exe.bat               # PyInstaller executable build script
├── FocusTracker.spec           # PyInstaller build specification
├── CODEBASE_GUIDE.md           # In-depth file-by-file linkage & architecture documentation
├── TECH_STACK_AND_GUIDE.md     # Technology stack reference & future project tools
└── static/                     # Frontend Single Page Application (SPA)
    ├── index.html              # Main dashboard interface layout
    ├── css/
    │   └── style.css           # Glassmorphism styling, theme tokens & responsive layout
    └── js/
        └── app.js              # Real-time WebSocket client, charts, timers & UI logic
```

For complete, detailed documentation on how every file functions and how each connects to the others, see [CODEBASE_GUIDE.md](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/CODEBASE_GUIDE.md).

---

## 🔗 Architecture & System Data Flow

```mermaid
graph TD
    User([User Desktop]) -->|Interacts with Windows| OS[Windows OS / Win32]
    
    subgraph Engine ["Background Tracking & AI Layer"]
        APP[app.py] -->|1. Launches| SRV[server.py]
        SRV -->|2. Instantiates & Starts| TRK[tracker.py]
        TRK -->|3. Queries HWND & Idle| MON[monitor.py]
        MON -->|Win32 / psutil| OS
        TRK -->|4. Inspects Webpage| WEB[web_inspector.py]
        WEB -->|UIAutomation & Scrape| OS
        TRK -->|5. Predicts Category| CLS[classifier.py]
        CLS -->|Loads Keywords| DB[(database.py / tracker.db)]
        TRK -->|6. Logs Session & Stats| DB
    end

    subgraph Server ["FastAPI ASGI Layer"]
        SRV -->|REST Endpoints| DB
        TRK -.->|Status & Alert Callbacks| SRV
        SRV -->|WebSocket Stream /ws| WS_MGR[ConnectionManager]
    end

    subgraph Client ["Browser Presentation Layer"]
        WS_MGR -->|Live JSON Packets| JS[static/js/app.js]
        JS -->|DOM Updates| HTML[static/index.html]
        CSS[static/css/style.css] -->|Visual Theme| HTML
        JS -->|REST API Calls| SRV
    end
```

### Core Execution Flows:
- **Active Window Polling**: Every 1.0 second, [`tracker.py`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/tracker.py) queries [`monitor.py`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/monitor.py) for the foreground window and hardware idle state.
- **AI Classification & Rail Guard**: Preliminary classification from window title by [`classifier.py`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/classifier.py). If a browser is focused, [`web_inspector.py`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/web_inspector.py) scrapes the first 70 words asynchronously and overrules the category if the content is distracting.
- **WebSocket Streaming**: Updates are broadcast through [`server.py`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/server.py) to [`static/js/app.js`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/static/js/app.js) over `/ws` for zero-lag dashboard updates.
- **Human-in-the-Loop Feedback**: Reclassifying an activity via the UI saves the correction to [`database.py`](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/database.py) and immediately retrains the Naive Bayes model.

---

## 🛠️ CLI Options

```bash
python app.py --help
python app.py --port 8080      # Run on custom port (default: 8000)
python app.py --no-browser     # Start server without auto-opening browser
```

---

## 📚 Further Documentation

- **[CODEBASE_GUIDE.md](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/CODEBASE_GUIDE.md)**: Exhaustive file-by-file breakdown, sequence diagrams, call graphs, database schemas, and WebSocket protocol contracts.
- **[TECH_STACK_AND_GUIDE.md](file:///c:/Users/piyus/Downloads/Productivity-Tracker-main/Productivity-Tracker/TECH_STACK_AND_GUIDE.md)**: Detailed evaluation of libraries, architectural patterns, and toolkits for building future AI desktop apps.

- # Productivity Tracker — CI/CD Pipeline Implementation

## 1. Project Overview

**Project Name:** Productivity Tracker (FocusTracker)

**GitHub Repository:** https://github.com/salonim10/Productivity-Tracker

This project is a Python-based productivity tracking application with a web dashboard for viewing productivity statistics, activity logs, daily tasks, and classification keywords.

The objective of this assignment is to implement a CI/CD pipeline that automates code validation, Docker image creation, image publishing, and Kubernetes deployment.

## 2. Technologies Used

- Python
- Git and GitHub
- Jenkins
- AWS EC2
- Docker
- Docker Hub
- Kubernetes
- Minikube
- Linux and Ubuntu

## 3. CI/CD Pipeline Stages

The Jenkins pipeline consists of the following stages:

1. **Checkout:** Retrieves the latest source code from the GitHub `upload-main` branch.
2. **Python Validation:** Installs project dependencies in a virtual environment, checks Python syntax, and executes the available unit tests.
3. **Build Docker Image:** Builds a Docker image using the Dockerfile and assigns build-specific and `latest` tags.
4. **Push to Docker Hub:** Authenticates using Jenkins-stored credentials and pushes the image to the Docker Hub repository `salonimuley/productivity-tracker`.
5. **Deploy to Kubernetes:** Applies the Kubernetes manifests, updates the Deployment to use the current build image, waits for rollout completion, and checks the deployment, pods, and services.

## 4. Jenkins Setup on AWS EC2

An Ubuntu EC2 instance was created in AWS and configured with Java, Jenkins, Docker, kubectl, and Minikube.

Jenkins was accessed through its web interface. A pipeline job named `Productivity-Tracker-CI-CD` was configured to retrieve the Jenkinsfile from the GitHub repository.

Docker Hub credentials were stored in Jenkins Credentials with the ID `dockerhub-creds`. These credentials were used during the image-publishing stage rather than being hard-coded in the pipeline.

A Kubernetes configuration was made available to Jenkins so that the pipeline could deploy the application to Minikube.

## 5. Docker Image Publishing

The Dockerfile uses the `python:3.11-slim` base image, installs the dependencies listed in `requirements-docker.txt`, copies the project files, and configures the application to run on port 8000.

The Jenkins pipeline builds and publishes the image to:

`salonimuley/productivity-tracker`

Build-specific tags and the `latest` tag are used to identify the published images.

## 6. Kubernetes Deployment

The `k8s/` directory contains two manifest files:

- `deployment.yaml`: Defines the Productivity Tracker Deployment and its application container.
- `service.yaml`: Defines a NodePort Service named `productivity-tracker-service` that routes traffic to port 8000 of the application container.

Minikube was used as the Kubernetes environment. The Deployment was configured with one replica.

The application was tested using Kubernetes port forwarding and an HTTP request to `http://127.0.0.1:8000`.

The application returned `HTTP/1.1 200 OK`, confirming that the deployed web application responded successfully.

## 7. Second Pipeline Execution

To demonstrate an updated deployment, the dashboard heading in `static/index.html` was changed to:

`Dashboard Overview - CI/CD Update 2`

The updated source code was committed and pushed to the `upload-main` branch on GitHub.

Jenkins was executed again. The second pipeline completed successfully, published a new Docker image, and deployed it to Kubernetes.

The rollout status reported successful deployment, the application pod was running, and the updated heading was verified in the HTTP response.

This demonstrated the workflow from source-code modification through image publishing and Kubernetes redeployment.

## 8. Verification Results

The following checks were completed:

- Jenkins pipeline completed successfully.
- Docker images were published to Docker Hub.
- Kubernetes Deployment reported one available replica.
- The application pod reached the Running state.
- Kubernetes rollout completed successfully.
- An HTTP request returned `200 OK`.
- The updated dashboard heading was found in the deployed application's response.

## 9. Conclusion

This assignment demonstrates a CI/CD workflow using GitHub, Jenkins, Docker, Docker Hub, AWS EC2, and Kubernetes with Minikube. The pipeline automates source-code validation, image creation, image publishing, and deployment, and the second execution demonstrates redeployment after a source-code change.

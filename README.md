# Host-Based Intrusion Detection System (HIDS)

A high-performance, modular, and explainable Host-Based Intrusion Detection System (HIDS) built with a hybrid detection approach using Rust and Python microservices. The system monitors local system activity, detects security threats in real-time, and presents alerts through a terminal-based user interface (TUI).

## 🎯 Objective

To create a robust HIDS that combines real-time system monitoring, hybrid machine learning detection (anomaly, signature, and behavioral), and a user-friendly TUI for threat visualization and system management.

## 🧱 Architecture Overview

```
           ┌────────────────────┐
           │     Rust TUI       │
           └────────┬───────────┘
                    │
                    ▼
         ┌────────────────────────────┐
         │     HIDS Engine (Rust)     │
         │ ─ REST API (TUI access)    │
         │ ─ File/Process Monitor     │
         │ ─ MongoDB Writer/Reader    │
         │ ─ gRPC Client              │
         └─────────┬──────────────────┘
                   │ gRPC (Protocol Buffers)
                   ▼
         ┌────────────────────────────┐
         │   Python ML Microservice   │
         │ ─ Anomaly Detection        │
         │ ─ Signature-based Detection│
         │ ─ Behavioral Detection     │
         └────────────────────────────┘
                   │
                   ▼
              MongoDB (Shared DB)
```

## ⚙️ Core Components

### 1. Rust-Based HIDS Engine

- **Responsibilities**:
  - Monitors local files (`/var/log`, `/etc/passwd`, etc.), processes, and network connections.
  - Sends suspicious log entries or system data to the Python microservice via gRPC.
  - Receives threat assessment and explanation from the Python microservice.
  - Writes structured alerts and logs to MongoDB.
  - Serves the TUI frontend via internal function calls or optionally via REST API (using `actix-web` or `axum`).
- **Technologies**:
  - `tokio`: Async runtime for concurrent operations.
  - `tonic`: gRPC client for communication with the Python microservice.
  - `mongodb`: Rust crate for MongoDB interactions.
  - `ratatui` or `tui-rs`: Terminal-based UI framework.

### 2. Python ML Microservice

- **Responsibilities**:
  - Exposed as a gRPC server, accepts system logs or behavior data and returns detection verdicts.
  - **Detection Pipeline**:
    - **Anomaly-based Detection**: Uses unsupervised models (e.g., Isolation Forest, clustering) to detect deviations.
    - **Signature-based Detection**: Uses regex or rule-based systems for known threats (e.g., failed login patterns).
    - **Behavioral Analysis**: Neural networks (e.g., LSTM/FFNN) trained on host behavior patterns.
    - **Explainable AI** (optional): Integrates with SHAP/LIME for threat explainability.
- **Technologies**:
  - `grpcio`, `protobuf`: gRPC server and protocol buffer definitions.
  - `scikit-learn`: For anomaly detection models.
  - `PyTorch`: For deep neural networks (if used).
  - `FastAPI`: Optional HTTP fallback for debugging or alternative access.

### 3. MongoDB (Central Storage)

- **Responsibilities**:
  - Stores alerts, logs, system metadata, and threat details.
  - Accessed exclusively by the Rust HIDS engine for centralized, schema-consistent storage.
- **Technologies**:
  - MongoDB for NoSQL storage.

### 4. Rust Terminal-Based UI (TUI)

- **Responsibilities**:
  - Displays a dashboard with CPU, memory, disk, and network stats.
  - Provides an alerts panel with real-time, color-coded threat notifications.
  - Offers a searchable and sortable logs viewer.
  - Monitors processes and files, highlighting anomalies or unauthorized access.
  - Includes controls to start/stop monitoring, adjust thresholds, and clear alerts.
- **Technologies**:
  - `ratatui` or `tui-rs` for terminal UI rendering.
  - Communicates with the HIDS engine via internal Rust channels or REST API (if decoupled).

## 🔌 Communication & Data Flow

### A. gRPC Communication (Rust ↔ Python)

- **Protocol**: Uses Protocol Buffers (`.proto`) for structured data exchange.
- **Data Structures**:
  - `LogEntry`: `{ source, content, timestamp }`
  - `DetectionResult`: `{ threat_detected, type, model, severity, explanation }`
- **Flow**:
  - Rust sends logs to the Python microservice.
  - Python processes logs and returns a structured verdict.
  - Rust writes the verdict and related data to MongoDB.

### B. TUI Communication (Frontend ↔ Engine)

- **Preferred Method**: Internal function/module communication for a single binary.
- **Alternative**: REST API (using `actix-web` or `axum`) for decoupled frontend-backend communication.

## 🚀 Getting Started

### Prerequisites

- **Rust**: Install via `rustup` (stable toolchain recommended).
- **Python**: Version 3.8+ with `pip` for ML microservice dependencies.
- **MongoDB**: Running instance (local or remote) for storage.
- **gRPC Tools**: `protoc` compiler for generating protocol buffer code.
- **Dependencies**:
  - Rust: `tokio`, `tonic`, `mongodb`, `ratatui` (or `tui-rs`), `actix-web`/`axum` (optional).
  - Python: `grpcio`, `protobuf`, `scikit-learn`, `torch`, `fastapi` (optional).

### Installation

1. **Clone the Repository**:

   ```bash
   git clone https://github.com/<your-repo>/hids.git
   cd hids
   ```

2. **Set Up Rust HIDS Engine**:

   ```bash
   cd hids-engine
   cargo build --release
   ```

3. **Set Up Python ML Microservice**:

   ```bash
   cd ml-microservice
   pip install -r requirements.txt
   ```

4. **Configure MongoDB**:

   - Ensure MongoDB is running and accessible.
   - Update connection settings in `hids-engine/config.toml` (if applicable).

5. **Generate gRPC Code**:
   ```bash
   protoc --rust_out=hids-engine/src --python_out=ml-microservice/src proto/hids.proto
   ```

### Running the System

1. **Start the Python ML Microservice**:

   ```bash
   cd ml-microservice
   python main.py
   ```

2. **Start the Rust HIDS Engine and TUI**:

   ```bash
   cd hids-engine
   cargo run --release
   ```

3. **Interact with the TUI**:
   - Use arrow keys to navigate panels.
   - Press `s` to start/stop monitoring, `t` to adjust thresholds, `c` to clear alerts.
   - View real-time alerts and logs in the TUI.

## 📖 Usage

- **Dashboard**: Monitor system resources (CPU, memory, disk, network).
- **Alerts Panel**: View color-coded threat alerts with severity and explanation.
- **Logs Viewer**: Search and sort through system logs.
- **Process & File Watcher**: Detect unauthorized access or anomalies.
- **Controls**: Adjust monitoring settings or clear alerts directly from the TUI.

## 🛠️ Development

- **Rust HIDS Engine**: Located in `hids-engine/`.
  - Add new monitors in `src/monitors/`.
  - Update TUI layouts in `src/tui/`.
- **Python ML Microservice**: Located in `ml-microservice/`.
  - Add new detection models in `src/models/`.
  - Update gRPC service logic in `src/server.py`.
- **Protocol Buffers**: Update `proto/hids.proto` for new data structures.
- **MongoDB Schema**: Define schemas in `hids-engine/src/db.rs`.

## 📝 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## 🤝 Contributing

Contributions are welcome! Please submit a pull request or open an issue for bugs, features, or improvements.

## 📧 Contact

For inquiries, reach out to `<your-email>` or open an issue on the repository.

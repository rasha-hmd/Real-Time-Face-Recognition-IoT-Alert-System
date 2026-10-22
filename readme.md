# Real-Time Face Recognition & IoT Alert System

An end-to-end computer vision security pipeline that performs real-time facial recognition via webcam, enables interactive in-stream enrollment, and dispatches rate-limited security alerts and cropped face snapshots over HTTP to a local Raspberry Pi endpoint.

---

## Features

- **Real-Time Detection & Recognition**: Powered by OpenCV and `face_recognition` (dlib) to detect faces and calculate Euclidean distance metrics against stored vector embeddings.
- **Interactive Enrollment**: Press `a` in the live feed to capture multiple perspective angles (10 samples) of a new subject and instantly append them to the local database.
- **Asynchronous Dispatch**: Uses a dedicated background worker (`queue.Queue` + `threading.Thread`) so network calls to the Raspberry Pi never freeze the camera stream.
- **Smart Notification Throttling**:
  - **Unknown Faces**: Rate-limited with a tight 30-second cooldown for immediate security alerts.
  - **Known Faces**: Rate-limited with a 120-second cooldown to eliminate notification spam.
- **Cropped Face Capture**: Automatically normalizes bounding boxes and isolates face crops before dispatching payloads over the local network.

---

## Project Structure

```text
.
├── create_oneCamera.py   # Multi-sample face capture and enrollment logic
├── detect.py             # Recognition engine and distance matching
├── main.py               # Video capture loop, threading, and alert scheduling
├── pi_sender.py          # HTTP client for Raspberry Pi notification service
├── requirements.txt      # Python package dependencies
└── README.md             # Project documentation
```

---

## System Architecture

```text
[ Web Camera Feed ]
        │
        ▼
[ OpenCV / dlib Face Detection & Recognition ]
        │
        ├── User Match / Unknown Detection
        │
        ▼
[ Rate-Limiter & Crop Isolation ]
        │
        ▼
[ Thread-Safe Queue (send_queue) ]
        │
        ▼
[ Background Worker (PiSender) ]
        │ HTTP POST (Multipart/Form-Data)
        ▼
[ Raspberry Pi Endpoint (:8000/upload) ] ──▶ [ Telegram / Buzzer / Display ]
```

---

## Prerequisites & Installation

### 1. Python Environment
Python 3.8 to 3.11 is recommended.

```bash
git clone https://github.com/your-username/face-recognition-iot.git
cd face-recognition-iot
```

### 2. Installing `dlib`

`dlib` requires C++ build tools.

* **Windows**:
  If building from source fails, download a pre-built wheel suitable for your Python version (e.g., from [Dlib_Windows_Python3.x](https://github.com/z-mahmud22/Dlib_Windows_Python3.x)):
  ```bash
  pip install <downloaded_dlib_file>.whl
  ```
* **Linux / macOS**:
  ```bash
  sudo apt-get update && sudo apt-get install -y cmake build-essential
  pip install dlib
  ```

### 3. Install Python Dependencies

```bash
pip install -r requirements.txt
```

---

## Configuration

Create a `.env` file in the root directory:

```env
# Network configuration
PI_IP_ADDRESS=192.168.1.100
PI_PORT=8000

# Optional notification settings
TELEGRAM_BOT_TOKEN=your_bot_token_here
CHAT_ID=your_chat_id_here
```

Ensure your `.gitignore` contains the following to prevent committing local biometric data:

```gitignore
.env
faces.dat
*.dat
__pycache__/
```

---

## Usage

### Run the Main Pipeline

```bash
python main.py
```

### Keyboard Shortcuts
* `a`: Start user enrollment. Check the terminal to input the person's name, then face the camera and slowly turn your head.
* `q`: Exit the application and safely terminate background threads.

---

## Receiver Endpoint Specification

The `PiSender` sends a standard multipart HTTP `POST` request to `http://<PI_IP_ADDRESS>:<PORT>/upload`. 

A minimal receiver running on the Raspberry Pi (using FastAPI or Flask) should accept:
* **`message`** (form-data field): Text describing the detection event.
* **`image`** (form-data file, optional): JPEG snapshot of the detected face.

---

## License

This project is licensed under the MIT License. See `LICENSE` for details.

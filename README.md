# File Transfer System

A peer-to-peer file transfer application built in Python with a Tkinter GUI. Allows users to send files directly to another machine over a TCP socket connection without relying on cloud storage or third-party services.

---

## Why this was built

Most file sharing solutions route data through a central cloud server — adding latency, storage costs, and privacy concerns. This project eliminates the middleman by establishing a direct TCP connection between the sender and receiver, keeping transfers fast, local, and private.

---

## Tech Stack

| Technology | Why it was chosen |
|---|---|
| **Python** | Cross-platform, built-in socket and threading libraries — no external dependencies for core functionality |
| **TCP Sockets** | Reliable, ordered byte-stream delivery — critical for file integrity. UDP would risk data loss without added complexity |
| **Tkinter** | Ships with Python's standard library, zero install friction for a lightweight GUI |
| **Threading** | Keeps the UI responsive while the server listens for incoming connections in the background |

---

## Architecture

The project follows a Client-Server model within a P2P setup. Each peer runs both sides — one machine acts as the server (receiver), the other as the client (sender). The roles can be swapped.

```
Peer2Peer_FT/
├── Client/
│   └── client_code.py   # GUI + sender logic
└── Server/
    └── server_code.py   # Receiver/server logic
```

---

## How it works

### Server (`server_code.py`)

1. Binds a TCP socket to `127.0.0.1:8000` and starts listening for incoming connections
2. On connection, reads the incoming byte stream in this order:
   - First 4 bytes → length of the username (big-endian int)
   - Next N bytes → username string
   - Next 4 bytes → length of the filename
   - Next M bytes → filename string
   - Remaining bytes → raw file data (read in 1024-byte chunks)
3. Reconstructs the file and saves it to a local `uploads/` folder (auto-created if missing)

### Client (`client_code.py`)

1. User enters a username and clicks **Choose File** — opens a native file picker dialog
2. Establishes a TCP connection to the server
3. Sends data in the same structured format the server expects — 4-byte length prefix → username → 4-byte length prefix → filename → raw file bytes
4. A background thread simultaneously runs a listener so the same peer can receive files while the UI stays active
5. Received files are announced via a popup and logged in an on-screen listbox

### Why length-prefixed framing?

TCP is a stream protocol — it has no concept of message boundaries. Sending a raw filename followed by raw file data would make it impossible to know where the filename ends and the file begins. The 4-byte big-endian length prefix before each field tells the receiver exactly how many bytes to read, making the protocol self-describing and unambiguous.

---

## Running locally

Start the server (receiver machine):

```bash
cd Peer2Peer_FT/Server
python server_code.py
```

Start the client (sender machine):

```bash
cd Peer2Peer_FT/Client
python client_code.py
```

1. Enter your username in the GUI
2. Click **Choose File** and select any file
3. File is sent to the server and saved in `uploads/`

> To transfer across machines on the same network, update `HOST` in both files to the receiver's actual IP address.

---

## Tech

`Python` · `TCP Sockets` · `Tkinter` · `Threading` · `Client-Server Architecture`

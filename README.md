# ZPL Thermal Print Bridge — User Manual

> **Version:** 1.0.0  
> **Platform:** Windows (7, 10, 11)  
> **Application:** `ZPLPrintBridge.exe`

---

## Table of Contents

1. [Introduction](#introduction)
2. [System Requirements](#system-requirements)
3. [Installation](#installation)
4. [Getting Started](#getting-started)
5. [Using the System Tray](#using-the-system-tray)
6. [API Reference](#api-reference)
7. [ZPL Printing Guide](#zpl-printing-guide)
8. [Troubleshooting](#troubleshooting)
9. [Frequently Asked Questions](#frequently-asked-questions)
10. [Building from Source](#building-from-source)
11. [Technical Overview](#technical-overview)

---

## Introduction

ZPL Thermal Print Bridge is a lightweight local application that lets you send ZPL (Zebra Programming Language) commands to thermal printers on your Windows machine. It provides:

- A **system tray application** with a right-click menu for managing printers and sending test prints.
- A **local REST API** (`http://127.0.0.1:8000`) that other applications can use to send ZPL programmatically.
- A **Swagger UI** at `http://127.0.0.1:8000/docs` for interactive API testing.

Everything runs locally on your machine — no cloud services, no internet connection required.

---

## System Requirements

| Requirement | Details |
|---|---|
| **Operating System** | Windows 7, Windows 10, or Windows 11 |
| **Architecture** | 64-bit (x64) |
| **Printers** | Any installed thermal/ZPL printer (local USB, network, or shared) |
| **Python** | Only required if running from source — not needed for the `.exe` |
| **RAM** | ~50 MB |
| **Disk Space** | ~50 MB for the executable |

---

## Installation

### Option A: Using the Standalone Executable (Recommended)

1. Copy the `ZPLPrintBridge.exe` executable file anywhere on your computer (e.g., `C:\Program Files\ZPLPrintBridge\`).
2. **Double-click** `ZPLPrintBridge.exe` to launch.

No installation wizard, no Python, no dependencies — it just runs.

> **Tip:** Create a shortcut to `ZPLPrintBridge.exe` on your desktop or in your startup folder for easy access.

### Option B: Running from Source (Developers)

```powershell
# Clone or download the project, then:
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python app.py
```

---

## Getting Started

### First Launch

1. **Run the application** by double-clicking `ZPLPrintBridge.exe`.
2. A terminal window will briefly show startup messages, then a **printer icon** ![printer icon](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxNiIgaGVpZ2h0PSIxNiIgdmlld0JveD0iMCAwIDE2IDE2Ij48cmVjdCB4PSIyIiB5PSI0IiB3aWR0aD0iMTIiIGhlaWdodD0iMTIiIHJ4PSIyIiBmaWxsPSIjNTU1Ii8+PHJlY3QgeD0iNCIgeT0iMSIgd2lkdGg9IjgiIGhlaWdodD0iNSIgcng9IjEiIGZpbGw9IiM4ODgiLz48cmVjdCB4PSIzIiB5PSIxMSIgd2lkdGg9IjEwIiBoZWlnaHQ9IjIiIHJ4PSIxIiBmaWxsPSIjNDQ0Ii8+PC9zdmc+) will appear in your system tray (notification area, near the clock).
3. **Right-click** the icon to open the menu.
4. A notification bubble will confirm: *"Ready at http://127.0.0.1:8000"*.

> **Note:** The first time you run the app, Windows Defender may show a SmartScreen warning since the executable is unsigned. Click **"More info"** → **"Run anyway"** to proceed.

### Verifying It Works

Once the app is running:

1. **Open your browser** and go to [http://127.0.0.1:8000](http://127.0.0.1:8000). You should see a JSON response with service info.
2. **Open the Swagger UI** at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) for interactive testing.
3. **Try a test print** from the tray menu (right-click → **Test Print (Basic)**).

---

## Using the System Tray

The system tray icon is the main way to interact with the application. When running, look for this icon in your notification area:

| Icon | Meaning |
|---|---|
| Gray printer icon | API server is running normally |
| Red-tinted printer icon | API server is unavailable (direct print fallback active) |

### Right-Click Menu Reference

| Menu Item | Description |
|---|---|
| **● API Running / ○ API Stopped** | Shows the current server status. Click to see a popup with details. |
| **Auto-Start ✓ / Auto-Start O** | Toggle to automatically launch the app when you log into Windows. |
| **Printers** | Expands to a submenu listing all detected printers. Each shows status (Ready/Printing/Offline). |
| **Test Print (Basic)** | Sends a simple ZPL test label with "Hello, Thermal Printer!" text and a barcode. |
| **Test Print (Label)** | Sends a product-style test label with item number, date, price, and barcode. |
| **Open API Docs** | Opens the Swagger interactive documentation in your browser. |
| **Restart API Server** | Stops and restarts the background API server. |
| **Exit** | Closes the application completely. Stops the API server. |

### Auto-Start Feature

Enable **Auto-Start** from the tray menu to have `ZPLPrintBridge.exe` launch automatically every time you log into Windows. This is stored in the Windows Registry under:

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
Key name: ZPLPrintBridge
```

---

## API Reference

The REST API runs on `http://127.0.0.1:8000`. All responses are in JSON format.

### GET `/` — Health Check

Returns service information. Use this to verify the API is running.

**Request:**
```http
GET http://127.0.0.1:8000/
```

**Response:**
```json
{
  "service": "ZPL Thermal Print Bridge",
  "version": "1.0.0",
  "endpoints": {
    "GET  /printers": "List all active printers",
    "POST /print": "Send ZPL code to a printer"
  }
}
```

**Status Codes:**
- `200 OK` — Service is running

---

### GET `/printers` — List Printers

Returns all printers detected on the system (local USB, network, and shared printers).

**Request:**
```http
GET http://127.0.0.1:8000/printers
```

**Response:**
```json
[
  {
    "name": "ZT230-102000Z22",
    "is_default": true,
    "is_local": true,
    "status": "Ready"
  },
  {
    "name": "\\\\192.168.1.100\\ZT411",
    "is_default": false,
    "is_local": false,
    "status": "Ready"
  }
]
```

| Field | Type | Description |
|---|---|---|
| `name` | string | Printer name as known to Windows |
| `is_default` | boolean | Whether this is the system default printer |
| `is_local` | boolean | `true` for local USB, `false` for network paths |
| `status` | string | `"Ready"`, `"Printing"`, `"Offline"`, or `"Unknown"` |

**Status Codes:**
- `200 OK` — Returns printer list (may be empty if no printers are installed)
- `500 Internal Server Error` — Failed to enumerate printers

---

### POST `/print` — Send ZPL Code

Sends ZPL commands to a specific printer (or the default printer).

**Request:**
```http
POST http://127.0.0.1:8000/print
Content-Type: application/json

{
  "zpl_code": "^XA^FO50,50^A0N25,25^FDHello World^FS^XZ",
  "printer_name": "ZT230-102000Z22",
  "copies": 2
}
```

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `zpl_code` | string | ✅ Yes | — | The raw ZPL command string to print |
| `printer_name` | string | ❌ No | System default printer | Target printer name (as reported by GET /printers) |
| `copies` | integer | ❌ No | `1` | Number of copies (1–100) |

**Response (Success):**
```json
{
  "success": true,
  "message": "ZPL job sent to 'ZT230-102000Z22' (2 copy/copies)",
  "printer": "ZT230-102000Z22",
  "copies": 2
}
```

**Response (Error — e.g., printer not found):**
```json
{
  "detail": "Cannot open printer 'Nonexistent Printer': Printer name is invalid or printer not found"
}
```

**Status Codes:**
- `200 OK` — Print job was sent successfully
- `400 Bad Request` — ZPL code is empty or invalid
- `500 Internal Server Error` — Printer cannot be opened, or write failure

> **Note:** A `200` response means the data was sent to the print spooler. It does not guarantee the physical printer received or printed it. Check the printer's display panel for errors.

---

### POST `/print/default` — Quick Print to Default Printer

A convenience endpoint for printing to the default printer using query parameters.

**Request:**
```http
POST http://127.0.0.1:8000/print/default?zpl_code=^XA^FO50,50^A0N25,25^FDHello^FS^XZ&copies=1
```

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `zpl_code` | string | ✅ Yes | — | The raw ZPL command string |
| `copies` | integer | ❌ No | `1` | Number of copies (1–100) |

**Response:** Same format as `POST /print`.

**Status Codes:**
- `200 OK` — Print job sent
- `400 Bad Request` — Missing or empty ZPL code
- `500 Internal Server Error` — Printer or communication error

---

## ZPL Printing Guide

ZPL (Zebra Programming Language) is a command language used to control label printers. Here are some common commands.

### Basic ZPL Structure

Every ZPL job starts with `^XA` and ends with `^XZ`:

```
^XA
^FO50,50
^A0N25,25
^FDHello, Thermal Printer!^FS
^XZ
```

### Common ZPL Commands

| Command | Description | Example |
|---|---|---|
| `^XA` | Start of label format | `^XA` |
| `^XZ` | End of label format | `^XZ` |
| `^FOx,y` | Field origin (position in dots) | `^FO50,50` |
| `^FS` | Field separator (end of field data) | `^FS` |
| `^FDtext` | Field data (the text to print) | `^FDHello^FS` |
| `^A0w,h` | Font (A0=standard, w=width, h=height) | `^A0N25,25` |
| `^BYw,r,h` | Barcode field defaults | `^BY3,3,100` |
| `^BCo,h,f,g,e` | Code 128 barcode | `^BCN,100,Y,N,N` |
| `^FO...^FDdata^FS` | Positioned text block | `^FO50,100^FDPrice: $9.99^FS` |

### Example: Simple Text Label

```
^XA
^FO50,50
^A0N40,40
^FDHello World^FS
^XZ
```

### Example: Label with Barcode

```
^XA
^FO50,50
^A0N30,30
^FDProduct Label^FS
^FO50,120
^BY3,3,100
^BCN,100,Y,N,N
^FD1234567890^FS
^XZ
```

### Example: Multi-Field Label

```
^XA
^FO50,30
^A0N40,40
^FDPRODUCT LABEL^FS
^FO50,100
^A0N25,25
^FDItem #12345^FS
^FO50,140
^A0N25,25
^FDDate: 2026-07-24^FS
^FO50,180
^A0N25,25
^FDPrice: $9.99^FS
^FO50,230
^BY3,3,100
^BCN,100,Y,N,N
^FDTEST123^FS
^XZ
```

### Tips for ZPL Printing

- **Positioning** uses dots — typical thermal printers are 203 DPI (dots per inch). So 203 dots = 1 inch.
- **Test with a small label first** to avoid wasting media.
- **Use `^LLnnn`** to set label length if your printer doesn't auto-calibrate.
- **Ensure media (labels/paper) is loaded** before sending a print job.

---

## Troubleshooting

### Application Won't Start

| Symptom | Likely Cause | Solution |
|---|---|---|
| Double-click does nothing | Antivirus or SmartScreen blocking | Click "More info" → "Run anyway" |
| "Side-by-side configuration is incorrect" | Missing VC++ Redistributable | Install the latest [VC++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe) |
| Terminal flashes and closes | Missing printer drivers or hardware issue | Run from a terminal to see error: `dist\ZPLPrintBridge.exe` |

### Printer Not Found in List

1. Open **Windows Control Panel** → **Devices and Printers**.
2. Verify the printer is installed and shows as "Ready".
3. If it's a network printer, ensure it's on the same network and the path is correct (e.g., `\\192.168.1.100\PrinterName`).
4. Right-click the tray icon → **Restart API Server**.
5. If still missing, restart the application.

### Test Print Sends but Nothing Prints

1. Check the printer's display panel for errors (paper out, ribbon error, paused, etc.).
2. Open **Windows → Devices and Printers** and check the printer queue for stuck jobs.
3. Try printing a **Windows test page** (right-click printer → **Printer Properties** → **Print Test Page**).
4. If the Windows test page works but ZPL doesn't, ensure the printer is in **ZPL mode** (not EPL, not line print).

### API Server Won't Start

| Symptom | Solution |
|---|---|
| Port 8000 already in use | Close other programs using port 8000, or change the port in `app.py` |
| Firewall blocking | The API is on `127.0.0.1` (localhost only), so no firewall config needed |
| "Failed to start" in tray | Try **Restart API Server** from the tray menu, or restart the application |

### Port Conflict (Port 8000 Already in Use)

If another application is using port 8000, you can change the port by running from source:

```python
# In app.py, change:
API_PORT = 8000    # → change to, e.g., 8001
```

Then restart the application.

### Auto-Start Not Working

1. Open **Registry Editor** (`regedit.exe`).
2. Navigate to `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`.
3. Look for `ZPLPrintBridge` — if missing, toggle Auto-Start off and on again from the tray menu.

---

## Frequently Asked Questions

**Q: Does this send data to the cloud?**  
A: No. Everything runs locally on your machine. The API is bound to `127.0.0.1` (localhost only) and is not accessible from other devices on your network.

**Q: Can I print from my own application?**  
A: Yes. Any application that can make HTTP requests can use the API. For example:

```python
import requests
requests.post("http://127.0.0.1:8000/print", json={
    "zpl_code": "^XA^FO50,50^FDHello^FS^XZ",
    "copies": 1
})
```

```javascript
// JavaScript / Node.js
fetch("http://127.0.0.1:8000/print", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ zpl_code: "^XA^FO50,50^FDHello^FS^XZ", copies: 1 })
});
```

**Q: Can I print to multiple printers at once?**  
A: The API supports one printer per request. You can send multiple requests in sequence.

**Q: Does it support USB printers?**  
A: Yes. Any printer that appears in Windows "Devices and Printers" can be targeted.

**Q: Why does my printer status show "Offline"?**  
A: The printer may be turned off, disconnected, paused, or in an error state. Check the printer's connection and power.

**Q: Can I run the application without a system tray?**  
A: The tray provides the primary user interface. If you only need the API, you can run just the server:

```powershell
pip install -r requirements.txt
uvicorn app:app --host 127.0.0.1 --port 8000
```

But you'll miss the tray convenience features.

**Q: Does the application auto-update?**  
A: No. To update, replace `ZPLPrintBridge.exe` with the new version.

---

## Building from Source

If you're a developer and want to build the executable yourself:

```powershell
# From the project root:
.\build_exe.bat
```

This script:
1. Creates a Python virtual environment (if missing).
2. Installs dependencies (FastAPI, uvicorn, pywin32, pystray, Pillow, requests, PyInstaller).
3. Runs PyInstaller to bundle `app.py` into a single `ZPLPrintBridge.exe`.
4. Output is placed in the `dist\` folder.

---

## Technical Overview

### Architecture

```
┌────────────────────────────────────────────┐
│           ZPLPrintBridge.exe               │
│                                            │
│  ┌──────────────────┐  ┌────────────────┐  │
│  │   FastAPI Server  │  │  System Tray   │  │
│  │  (uvicorn thread) │  │  (pystray)     │  │
│  │                   │  │                │  │
│  │  Port 127.0.0.1   │  │  Right-click   │  │
│  │  :8000            │  │  menu          │  │
│  └────────┬──────────┘  └────────┬───────┘  │
│           │                      │          │
│           └──────────┬───────────┘          │
│                      │                      │
│            ┌─────────▼─────────┐            │
│            │   win32print API  │            │
│            │  (Windows native) │            │
│            └─────────┬─────────┘            │
│                      │                      │
└──────────────────────┼──────────────────────┘
                       │
               ┌───────▼───────┐
               │   Thermal     │
               │   Printer     │
               └───────────────┘
```

### Data Flow

1. **User** or **application** sends a POST request to `http://127.0.0.1:8000/print` with ZPL code.
2. **FastAPI server** receives the request, validates it, and calls `win32print`.
3. **win32print** opens the printer handle and sends the ZPL data as a RAW print job.
4. The **Windows print spooler** delivers the job to the physical printer.
5. The **printer** processes the ZPL commands and produces the label.

### Direct Print Fallback

If the API server is not running (e.g., failed to start or crashed), the tray application detects this and switches to **direct print mode**. Test prints and tray-initiated prints bypass the API and call `win32print` directly. This ensures the tray menu remains functional even without the server.

---

## Support

For issues, feature requests, or questions, please open an issue in the project repository.

---

*Document version 1.0.0 — July 2026*

# 🔐 Surveillance System — AI Face-Lock Security

An AI-powered **smart door-lock / surveillance system** built in Python. It unlocks access only when **three layers of verification** pass — your **face**, your **voice passphrase**, and a **one-time password (OTP)** sent to your phone. Any failed attempt triggers an **instant SMS security alert**.

<!-- 📸 SCREENSHOT: Add your main application window here -->
<!-- ![Application Main Window](assets/screenshot_main.png) -->

---

## ✨ Features

| Layer | Verification | Tech Used |
|-------|-------------|-----------|
| 👁️ 1 | **Face Recognition** — identifies registered users from the camera feed | OpenCV (Haar Cascade + LBPH) |
| 🎙️ 2 | **Voice Passphrase** — user speaks a secret passphrase | SpeechRecognition + Google Speech API |
| 📱 3 | **OTP Verification** — one-time password sent via SMS | Twilio (SHA-256 hashed OTP) |
| 🔊 Feedback | Talks to the user at every step | pyttsx3 (Text-to-Speech) |
| 🚨 Security | SMS alerts on wrong passphrase / wrong OTP attempts | Twilio Messaging |

### Verification Flow

```
        ┌─────────────────────┐
        │   Camera Feed (0)   │
        └──────────┬──────────┘
                   ▼
        ┌─────────────────────┐
        │  Face Detection     │  Haar Cascade (frontalface_default)
        └──────────┬──────────┘
                   ▼
        ┌─────────────────────┐      Not recognized ❌
        │  Face Recognition   │ ───────────────────► "CANT RECOGNISE"
        │  (LBPH, >85% conf.) │
        └──────────┬──────────┘
                   ▼
        ┌─────────────────────┐      Wrong passphrase ❌
        │  Voice Passphrase   │ ───────────────────► 🚨 SMS Alert
        └──────────┬──────────┘
                   ▼
        ┌─────────────────────┐      Wrong OTP ❌
        │  OTP via SMS        │ ───────────────────► 🚨 SMS Alert
        └──────────┬──────────┘
                   ▼
        ┌─────────────────────┐
        │  ✅ ACCESS GRANTED  │  + confirmation SMS
        └─────────────────────┘
```

---

## 📸 Screenshots

> **Add your own application screenshots here** by placing image files in an `assets/` folder and uncommenting the lines below.

<!-- 📸 SCREENSHOT: Face detection box on camera feed -->
<!-- ![Face Detected](assets/screenshot_face_detected.png) -->

<!-- 📸 SCREENSHOT: Face recognised greeting -->
<!-- ![Face Recognised](assets/screenshot_face_recognised.png) -->

<!-- 📸 SCREENSHOT: Face not recognised / intruder -->
<!-- ![Intruder Alert](assets/screenshot_intruder.png) -->

<!-- 📸 SCREENSHOT: OTP verification in terminal -->
<!-- ![OTP Verification](assets/screenshot_otp.png) -->

<!-- 📸 SCREENSHOT: Security alert SMS on phone -->
<!-- ![SMS Alert](assets/screenshot_sms_alert.png) -->

---

## 🗂️ Project Structure

```
survelience-system/
└── Lock-System/
    ├── main.py                     # Entry point — full verification pipeline
    ├── sampling.py                 # Enrolment tool — capture face samples of a new user
    ├── requirements.txt            # Python dependencies
    ├── .env.example                # Template for required environment variables
    ├── Dataset/                    # (git-ignored) captured face images + labels.json
    ├── model/                      # (git-ignored) trained LBPH model (trained_model.cv2)
    └── lib/
        ├── FaceDetectionModel.py   # Haar Cascade face detection + ROI extraction
        ├── FaceIdentificationModel.py  # LBPH face recognition (train + predict)
        ├── SpeechHandler.py        # Text-to-Speech & Speech-to-Text
        └── TwilioHandler.py        # OTP generation/verification + SMS alerts
```

---

## ⚙️ Prerequisites

- **Python 3.10+**
- A **webcam**
- A **microphone** and speakers
- A free **[Twilio](https://www.twilio.com/)** account (for OTP + SMS alerts)
- On Windows, **PyAudio** may need a prebuilt wheel — see [Troubleshooting](#-troubleshooting).

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Akshat-hub7/survelience-system.git
cd survelience-system/Lock-System
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # Linux / macOS
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Copy `.env.example` and set your own values (Twilio credentials, your phone number):

```bash
copy .env.example .env       # Windows
cp .env.example .env         # Linux / macOS
```

Then either export them in your shell, or load them from your editor/IDE:

```bash
set USER_MOBILE_NO=+91XXXXXXXXXX
set TWILIO_ACCOUNT_SID=ACxxxxxxxx...
set TWILIO_AUTH_TOKEN=your_token
set TWILIO_VERIFY_SID=VAxxxxxxxx...
set TWILIO_MESSAGE_SERVICE_SID=MGxxxxxxxx...
```

> 🔒 **Never hardcode credentials** — they are read from environment variables (see `lib/TwilioHandler.py` and `main.py`).

### 5. Enrol a user (capture face samples)

```bash
python sampling.py
```

- Enter the person's **name** when prompted.
- Look at the camera — **150 face samples** are captured automatically into `Dataset/` with labels saved to `Dataset/labels.json`.

### 6. Run the lock system

```bash
python main.py
```

1. The system trains/loads the LBPH face model from your dataset.
2. Look at the camera — your face is detected and recognised.
3. Speak your **passphrase** (default: `"lock"`, configurable via `PASSCODE` in `main.py`).
4. Type the **OTP** sent to your phone.
5. ✅ Access granted — a confirmation SMS is sent. ❌ Any failed step triggers a security alert SMS.

---

## 🛠️ Customisation

| What | Where | Default |
|------|-------|---------|
| Voice passphrase | `PASSCODE` in `main.py` | `"lock"` |
| Number of enrolment samples | `SAMPLES` in `sampling.py` | `150` |
| Recognition confidence threshold | `identify()` in `lib/FaceIdentificationModel.py` | `> 85%` |
| OTP retry attempts | `verify()` in `lib/TwilioHandler.py` | `1` retry |
| Alert phone number | `USER_MOBILE_NO` env var | — |

---

## 🧰 Tech Stack

- **OpenCV / opencv-contrib** — face detection (Haar Cascade) & recognition (LBPH)
- **NumPy** — dataset & matrix handling
- **pyttsx3** — offline text-to-speech feedback
- **SpeechRecognition** — passphrase capture via microphone (Google Speech API)
- **Twilio** — OTP over SMS + security alert messages

## 🧯 Troubleshooting

- **`PyAudio` fails to install (Windows):** download a prebuilt wheel, e.g. `pip install pipwin && pipwin install pyaudio`, or grab a wheel from [here](https://www.lfd.uci.edu/~gohlke/pythonlibs/#pyaudio).
- **"Image Dataset is too small!"** — run `python sampling.py` first; the model needs ≥ 50 face samples.
- **No sound from TTS** — check your default audio output device; pyttsx3 uses the system voice.
- **Twilio errors ("Max OTP send limit reached")** — Twilio rate-limits OTP sends; wait ~10 minutes.
- **Microphone not heard** — check OS microphone permissions and default input device.

## 🔒 Security Notes

- All credentials are read from **environment variables** — never commit `.env` or real keys.
- The dataset (face images) and trained model are **git-ignored** — they never leave your machine.
- OTPs are **SHA-256 hashed** before verification and never stored in plain text.
- Twilio Account SID / Auth token can be rotated anytime from the [Twilio Console](https://console.twilio.com).

## 📄 License

This project is open for forking and extension. Feel free to build on it!

---

Made with ❤️ by [@Akshat-hub7](https://github.com/Akshat-hub7)

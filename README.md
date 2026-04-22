# ⚖️ AI-Powered Smart Weight Platform

> A smart IoT weight scale platform powered by **n8n automation workflows** — combining real-time body weight sensing with Cloud AI food analysis for **personalized health recommendations** and a **2-factor weight + OTP door security system**.

---

## 🚀 Two Applications, One Platform

### 🏥 Application 1 — Smart Health Advisor
User steps on the scale → n8n collects body weight + food photo → Cloud AI analyzes the meal → n8n generates personalized health recommendations → stored in Google Sheets.

### 🔐 Application 2 — Weight + OTP Door Security (2-Factor)
User steps on the scale → n8n compares weight to stored profile → if approved, sends a **One-Time Password (OTP) via Gmail** → user submits OTP → door opens. Weight reference auto-updates on every successful access.

---

## 🏗️ System Architecture

```
[Load Cell Scale]
      ↓
[ESP32 / Arduino + HX711]
      ↓ MQTT
[Mosquitto Broker]
      ↓
┌─────────────────────────────────────────────────────────────┐
│                        n8n Workflows                        │
│                                                             │
│  🏥 Health Workflow          🔐 Security Workflow          │
│  ─────────────────           ──────────────────────         │
│  1. Receive weight        1. Receive weight                 │
│  2. Receive food photo    2. Fetch reference (Google Sheets)│
│  3. Send to Cloud AI      3. Compare weight (± tolerance)   │
│  4. Get nutrition data    4. ❌ No match → Deny + Alert     │
│  5. Generate health tips  5. ✅ Match → Generate OTP        │
│  6. Save to Google Sheets 6. Send OTP via Gmail             │
│                           7. Wait for OTP input             │
│                           8. ✅ OTP correct → Open door     │
│                              ❌ OTP wrong → Deny            │
│                           9. Update weight reference        │
└──────────────┬──────────────────────────────────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
[Google Sheets]      [PostgreSQL]
 Weight History       (n8n internal
 Food History          database)
```

---

## ✨ Key Features

### 🏥 Health Application
- **⚖️ Daily Body Weight Tracking** — Logged automatically to Google Sheets
- **📷 Food Photo Analysis** — Cloud AI identifies food, calories, and nutrients
- **🧠 AI Health Recommendations** — n8n combines weight + food data for personalized advice
- **📊 History in Google Sheets** — Easy to view, filter, and share weight & food logs

### 🔐 Security Application
- **⚖️ Weight as First Factor** — Body weight used as the first biometric check
- **📧 OTP via Gmail as Second Factor** — One-time password sent to user's email after weight approval
- **2-Factor Authentication** — Both weight AND OTP must pass to unlock the door
- **🚪 Automatic Door Control** — n8n triggers door relay only after both factors verified
- **🔄 Self-Updating Reference** — Weight profile updates automatically on each successful access
- **⚠️ Intrusion Alerts** — n8n sends alert when weight doesn't match

### Shared Infrastructure
- **📡 MQTT Communication** — Reliable, lightweight IoT data transport
- **🐳 Docker Deployment** — Full stack runs with one command
- **🌐 Remote Access** — Secure tunnel via Cloudflare

---

## 🛠️ Tech Stack

| Layer | Technology | Role |
|---|---|---|
| Hardware | ESP32 / Arduino + HX711 + Load Cell | Weight sensing |
| Communication | MQTT (Mosquitto) | IoT data transport |
| Automation | **n8n** | Both app logic & workflows |
| AI Engine | Cloud AI API | Food photo recognition & analysis |
| OTP Delivery | **Gmail API** | Send OTP email to user |
| Data Storage | **Google Sheets API** | Weight history & food history |
| Internal DB | PostgreSQL | n8n internal database |
| Infrastructure | Docker & Docker Compose | Deployment |
| Network | Cloudflare Tunnel | Remote access |

> **Optional:** Flask API + Ollama (Llama 3.2) can be added for local AI inference. See [Local AI Setup](#-optional-local-ai-setup).

---

## 🧠 Application 1 — Health Workflow (n8n)

```
MQTT Trigger (weight received)
        ↓
Receive Food Photo
        ↓
Cloud AI API → Food name + Calories + Protein / Carbs / Fat
        ↓
Generate Health Recommendation
        ↓
Append to Google Sheets (Weight History + Food History)
        ↓
Send Notification to User
```

**Google Sheets — Weight History:**
| Date | Weight (kg) | BMI | Goal (kg) |
|---|---|---|---|
| 2026-04-22 | 72.3 | 22.4 | 70.0 |

**Google Sheets — Food History:**
| Date | Food | Calories | Protein | Carbs | Fat | Recommendation |
|---|---|---|---|---|---|---|
| 2026-04-22 | Grilled Chicken + Rice | 650 kcal | 42g | 75g | 12g | Add vegetables for fiber |

---

## 🔐 Application 2 — Security Workflow (n8n)

### 2-Factor Authentication Flow

```
MQTT Trigger (weight received)
        ↓
Fetch reference weight from Google Sheets
        ↓
┌─── Compare weight ───┐
│                      │
❌ No Match            ✅ Match
Deny Access            Generate OTP
Send Alert             Send OTP via Gmail
                            ↓
                      Wait for user OTP input
                            ↓
                  ┌─── Verify OTP ───┐
                  │                  │
                  ❌ Wrong OTP       ✅ Correct OTP
                  Deny Access        Trigger door relay → OPEN
                  Send Alert         Update weight reference
                                     in Google Sheets
```

### Self-Updating Weight Reference

Each time both factors pass, the reference weight is smoothly updated to reflect natural body changes:

| Day | Reference (kg) | Measured (kg) | Weight | OTP | Result | New Reference |
|---|---|---|---|---|---|---|
| Monday | 72.0 | 72.3 | ✅ | ✅ | Door Opens | 72.1 |
| Tuesday | 72.1 | 71.9 | ✅ | ✅ | Door Opens | 72.0 |
| Wednesday | 72.0 | 65.0 | ❌ | — | Denied + Alert | 72.0 |
| Thursday | 72.0 | 72.1 | ✅ | ❌ | Denied | 72.0 |

---

## 🚀 Getting Started

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/) & Docker Compose
- Cloud AI API key
- Google Sheets API credentials (service account)
- Gmail account with App Password enabled
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/Luckamoi/Iot-ai-automation.git
cd Iot-ai-automation
```

### 2. Configure Environment Variables

```bash
cp .env.example .env
```

```env
# Cloud AI
CLOUD_AI_API_KEY=your_api_key_here

# Google Sheets
GOOGLE_SHEET_ID=your_google_sheet_id
GOOGLE_SERVICE_ACCOUNT_JSON=path/to/credentials.json

# Gmail OTP
GMAIL_USER=your_email@gmail.com
GMAIL_APP_PASSWORD=your_gmail_app_password
OTP_EXPIRY_SECONDS=60

# Security Settings
WEIGHT_TOLERANCE_KG=2.0

# MQTT
MQTT_BROKER=mosquitto
MQTT_PORT=1883

# n8n
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_password
```

### 3. Start the System

```bash
docker-compose up -d
```

### 4. Import n8n Workflows

1. Open n8n at `http://localhost:5678`
2. Go to **Workflows → Import**
3. Import files from `/n8n-workflows/`

---

## 🔌 Hardware Setup

**Load Cell Wiring (HX711 → ESP32):**
```
Load Cell → HX711 → ESP32
                    ├── DT  → GPIO 21
                    └── SCK → GPIO 22
```

**Door Relay:**
```
ESP32 GPIO 18 → Relay Module → Door Lock
```

**MQTT Topics:**
```
scale/weight       → Body weight readings (kg)
scale/status       → Device connection status
security/otp       → OTP submission from user input
security/access    → Access granted / denied result
security/alert     → Unauthorized access alert
```

---

## 📁 Project Structure

```
Iot-ai-automation/
├── flask_app/                  # Optional: Local AI (Ollama)
│   ├── app.py
│   └── services/
├── n8n-workflows/              # Exportable n8n workflow files
│   ├── health_workflow.json
│   └── security_workflow.json
├── docker-compose.yml
├── .env.example
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🖥️ Optional: Local AI Setup

```bash
ollama pull llama3.2
ollama serve
```

Update `.env`:
```env
USE_LOCAL_AI=true
OLLAMA_API=http://host.docker.internal:11434
MODEL_NAME=llama3.2
```

---

## 🔮 Roadmap

- [ ] Multi-user profiles (separate weight + email per person)
- [ ] OTP via SMS (Twilio) as alternative to Gmail
- [ ] BMI / BMR auto-calculation in health workflow
- [ ] Google Sheets dashboard with charts
- [ ] Face recognition as optional third factor
- [ ] Mobile notifications (Line / Telegram)
- [ ] Weekly health summary report automation

---

## 📄 License

[MIT License](LICENSE)

---

## 👤 Author

**Luckamoi** · [@Luckamoi](https://github.com/Luckamoi)

---

> ⭐ If this project helped you, please consider giving it a star!

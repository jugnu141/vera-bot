
# Vera — AI Merchant Growth Assistant

Vera is an AI-powered merchant growth assistant built for the **Magicpin AI Challenge**.

The application provides a FastAPI-based conversational API that can understand merchant interactions, maintain conversation state, detect customer intent, handle automated replies, respond to hostile/opt-out messages, and generate contextual campaign recommendations.

## 🚀 Live Deployment

**Production URL:**  
https://vera-bot-knyt.onrender.com

**Health Check:**  
https://vera-bot-knyt.onrender.com/v1/healthz

## ✨ Features

- 🤖 AI-powered merchant conversation handling
- 🧠 Stateful conversation management
- 🎯 Merchant intent detection
- 📈 Context-aware campaign recommendations
- 🔄 Automated-reply detection
- 🛑 Opt-out and hostile-message handling
- ⏳ Conversation pause/wait logic
- 💬 Contextual campaign messaging
- 🔐 Pydantic request validation
- ⚡ FastAPI REST API
- 🌐 Cloud deployment using Render

## 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │   Magicpin Judge    │
                    │     / Simulator     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    FastAPI / Vera   │
                    │                     │
                    │  /v1/reply          │
                    │  /v1/context        │
                    │  /v1/tick           │
                    │  /v1/metadata       │
                    │  /v1/healthz        │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
          Conversation     Merchant       Campaign
             State         Context        Decisions
                │              │              │
                └──────────────┼──────────────┘
                               ▼
                         Vera Response
````

## 🧠 Conversation Handling

Vera maintains state for conversations instead of treating every request as an isolated message.

The system tracks information such as:

* Conversation ID
* Merchant ID
* Customer ID
* Previous replies
* Conversation state
* Automated reply count
* Merchant context
* Previous conversation information

This allows Vera to react differently depending on the current stage of the conversation.

### Example State Flow

```text
New Conversation
       │
       ▼
  Understand Intent
       │
 ┌─────┼───────────┐
 ▼     ▼           ▼
Agree  Auto Reply  Hostile
 │        │          │
 ▼        ▼          ▼
Execute  Wait/Pause  Opt-out
```

## 🎯 Intent Handling

Vera recognizes positive merchant intent and can transition the conversation into an action-oriented state.

For example:

```text
Merchant:
"Ok lets do it. What's next?"

Vera:
"Done — I'll proceed with the next campaign step."
```

The response is designed to move the conversation forward rather than simply acknowledge the message.

## 🔄 Automated Reply Detection

Vera detects repeated or automated messages such as:

* Out-of-office responses
* Automated responses
* Repeated identical messages
* Common system-generated replies

Instead of repeatedly messaging an automated responder, Vera can pause the conversation and eventually end it when repeated automated responses are detected.

Example:

```text
Automated response detected
        │
        ▼
      WAIT
        │
        ▼
Repeated automated response
        │
        ▼
       END
```

## 🛑 Hostile / Opt-out Handling

Vera recognizes messages indicating that the recipient does not want further communication.

Examples include:

```text
"Stop messaging me."
"This is spam."
"Don't message me again."
"Leave me alone."
```

These messages transition the conversation toward an opt-out/end state instead of continuing promotional communication.

## 📊 Context-Aware Campaign Recommendations

Vera uses merchant signals and trigger information to generate more relevant campaign recommendations.

Examples of supported situations include:

| Signal              | Example Response Direction                         |
| ------------------- | -------------------------------------------------- |
| Performance drop    | Targeted campaign to address declining performance |
| Performance spike   | Capitalize on current momentum                     |
| Renewal             | Prepare a renewal campaign                         |
| Festival / Seasonal | Create a timely seasonal campaign                  |
| Lapsed customers    | Re-engage inactive customers                       |
| IPL / Match         | Create a match-day campaign                        |
| Reviews             | Respond to review-related signals                  |
| Milestone           | Capitalize on a business milestone                 |
| GBP verification    | Address listing verification issues                |
| Refill              | Create a refill-focused campaign                   |

The goal is to connect the observed merchant signal with a concrete next action.

## 🔌 API Endpoints

### `GET /v1/healthz`

Health check endpoint.

Example response:

```json
{
  "ok": true,
  "status": "healthy"
}
```

### `GET /v1/metadata`

Returns Vera service metadata and supported capabilities.

### `POST /v1/context`

Provides or updates merchant context used by Vera.

### `POST /v1/tick`

Processes a campaign/event tick and generates relevant campaign actions.

### `POST /v1/reply`

Main conversational endpoint.

It accepts information such as:

```json
{
  "conversation_id": "conversation-123",
  "merchant_id": "merchant-123",
  "customer_id": "customer-123",
  "from_role": "merchant",
  "message": "Ok lets do it. What's next?",
  "received_at": "2026-09-27T10:00:00Z",
  "turn_number": 2
}
```

The response contains action and messaging information used by the simulator.

## 🛠️ Tech Stack

* **Python**
* **FastAPI**
* **Pydantic**
* **Uvicorn**
* **Ollama**
* **Llama 3**
* **Render**
* REST APIs
* Stateful in-memory conversation store

## 📁 Project Structure

```text
vera-bot/
│
├── main.py
├── requirements.txt
├── README.md
├── .gitignore
└── venv/                 # Local environment, not committed
```

## ⚙️ Local Setup

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/vera-bot.git
cd vera-bot
```

### 2. Create a virtual environment

```bash
python -m venv venv
```

### 3. Activate the environment

Windows:

```bash
venv\Scripts\activate
```

Linux/macOS:

```bash
source venv/bin/activate
```

### 4. Install dependencies

```bash
pip install -r requirements.txt
```

### 5. Start the API

```bash
uvicorn main:app --reload
```

The API will be available at:

```text
http://localhost:8000
```

Swagger documentation:

```text
http://localhost:8000/docs
```

Health check:

```text
http://localhost:8000/v1/healthz
```

## 🤖 Ollama

The local development setup can use Ollama as the LLM provider.

Default configuration:

```text
Provider: Ollama
Model: llama3
URL: http://localhost:11434
```

Make sure Ollama is running before using the LLM-dependent functionality locally.

## 🌐 Deployment

The application is deployed as a Render Web Service.

### Build Command

```bash
pip install -r requirements.txt
```

### Start Command

```bash
uvicorn main:app --host 0.0.0.0 --port $PORT
```

The application binds to `0.0.0.0` so it can receive external requests from the deployment platform.

## 🧪 Testing

The project was tested against the provided Magicpin challenge simulator.

Key conversational scenarios include:

* Warm-up / basic interaction
* Automated reply handling
* Positive intent transition
* Hostile / opt-out handling
* Full merchant-growth message evaluation

Example simulator configuration:

```python
BOT_URL = "https://vera-bot-knyt.onrender.com"
```

## 🔒 Security & Reliability Considerations

* Request validation is handled through Pydantic.
* Conversation state is maintained per conversation/merchant.
* Opt-out messages are handled explicitly.
* Automated replies are detected to prevent unnecessary repeated messaging.
* Sensitive configuration such as API keys should not be committed to Git.

## 🚧 Future Improvements

Potential improvements include:

* Persistent database-backed conversation state
* Redis-based session storage
* More sophisticated intent classification
* Better merchant segmentation
* Campaign performance feedback loops
* Persistent analytics
* Authentication and API authorization
* Production-grade logging and monitoring
* More advanced LLM-based campaign generation

## 👨‍💻 Author

**Md Nawaz Alam**

B.Tech — Electronics & Communication Engineering

Interested in:

* Backend Development
* MERN Stack
* Artificial Intelligence
* Machine Learning
* DSA
* AI-powered applications

---

## 📌 Magicpin AI Challenge

Built as part of the **Magicpin Vera AI Challenge**, focusing on building an AI assistant capable of handling merchant conversations and generating useful, context-aware growth actions.





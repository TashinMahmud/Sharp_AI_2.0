# 💬 FastAPI Ai Quiz — Interactive Debate Coach & AI Quiz Engine

<div align="center">

[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-005571?style=for-the-badge&logo=fastapi&logoColor=white)](#quick-start)
[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com/)
[![LangChain](https://img.shields.io/badge/LangChain-Core-000000?style=for-the-badge&logo=chainlink&logoColor=white)](https://github.com/langchain-ai/langchain)
[![Tenacity](https://img.shields.io/badge/Tenacity-Robust_Retries-EE5253?style=for-the-badge)](#openai-completion-with-tenacity-retries)

---

**FastAPI Ai Quiz** is an interactive debate combat coordinator and question generator. Leveraging **OpenAI GPT-4o**, **LangChain**, and **Tenacity** exponential backoffs, the service creates dynamic arguments/counter-arguments, generates multiple-choice quizzes with explanations, evaluates student answers, and runs a session-persisted, summarization-enabled conversational debate coach.

</div>

---

## 🛠️ Technical Architecture

This microservice handles the generative tasks for structured debate training.

```
+-------------------------------------------------------------+
|                      CLIENT INTERFACE                       |
|   Sends Debate Actions  <--->  Receives Structured JSON     |
+------------------------------+------------------------------+
                               | (HTTP POST /generate, /quiz, /debate/chat)
                               v
+-------------------------------------------------------------+
|                     FASTAPI ENGINE CORE                     |
|  Slowapi rate checks, JSON routers, and exception handlers  |
+------------------------------+------------------------------+
                               |
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
+-----------------------+               +-----------------------+
|  AI SERVICE ENGINE    |               |  MEMORY SERVICE CORE  |
| - Prompts compiler    | <===========> | - Session history     |
| - Tenacity retries    |               | - Auto-summarization  |
| - OpenAI completions  |               | - SQLite persistence  |
+-----------------------+               +-----------------------+
```

### Core Code Modules & Responsibilities

*   `app/api/` Layer:
    *   [`routes/debate.py`](app/api/routes/debate.py): Defines JSON-parsing post routes for argument extraction, quizzes, hints, score checks, and multi-role chat sessions.
*   `app/core/` Layer:
    *   [`config.py`](app/core/config.py): Base settings parsing keys, debug mode flags, and retry parameters.
    *   [`limiter.py`](app/core/limiter.py): Rate limiting rules (e.g. 30 requests per minute).
*   `app/services/` Layer:
    *   [`ai_service.py`](app/services/ai_service.py): Prompts for evaluation and debate engines, Tenacity retries, and OpenAI client calls.
    *   [`memory_service.py`](app/services/memory_service.py): Saves message logs to SQLite, compiles summaries, and prunes older turns.
    *   [`rag_service.py`](app/services/rag_service.py): Compiles prompts with debate rules, conversational context, and summaries.

---

## ⚡ Core Integration Interfaces

<details>
<summary><b>🛡️ OpenAI Completion with Tenacity Retries</b></summary>

Calls OpenAI using Pydantic JSON modes. It wraps requests in a Tenacity exponential backoff handler (`retry_if_exception_type`) to survive transient rate limits, timeout exceptions, and API connection resets automatically.
</details>

<details>
<summary><b>🧠 Debate Memory Summary Trigger</b></summary>

Monitors conversation length dynamically. When a session matches the configured limit (e.g., 6 turns), it calls a separate summary chain, writes the condensed context to the SQLite DB, and prunes the in-memory chat list to keep prompts concise.
</details>

<details>
<summary><b>🎯 Core API Interfaces</b></summary>

*   **POST `/generate`**: Takes a topic and difficulty, and returns a JSON payload containing `main_arguments`, `counter_arguments`, and `rebuttals`.
*   **POST `/quiz`**: Generates a single question with four options, correct answer index, and a structural explanation.
*   **POST `/evaluate`**: Evaluates a student response against the model's correct key, providing constructive debate coach coaching hints.
*   **POST `/debate/chat`**: Initiates a chat roleplay session (`user_argument` | `user_counter` | `user_rebuttal`), maintaining session identity.
</details>

---

## 🚀 Quick Start

### 1. Requirements
*   Python 3.10+
*   Virtual environment manager
*   Active OpenAI API key

### 2. Configurations Setup
1.  Copy `.env.example` to a new `.env` file:
    ```bash
    cp .env.example .env
    ```
2.  Set your credentials:
    ```env
    OPENAI_API_KEY=sk-your-key-here
    OPENAI_MODEL=gpt-4o
    DEBUG=False
    MEMORY_SUMMARY_TRIGGER=6
    MEMORY_KEEP_LAST=3
    ```

### 3. Installation & Run
Configure environment and dependencies:
```bash
python -m venv venv
# On Windows
venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate

pip install -r requirements.txt
```

Launch the FastAPI application:
```bash
uvicorn app.main:app --reload
```
Swagger UI will be active at `http://localhost:8000/docs`.

### 4. Running Tests
Run tests using pytest or python:
```bash
pytest
```

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).

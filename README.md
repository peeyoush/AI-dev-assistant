<div align="center">

<img src="assets/logo-dark.svg" alt="CodeInsight" width="300"/>

<br/>
<br/>

<h3>Debug. Understand. Ship faster.</h3>

<p>A professional AI-powered code analysis platform that helps developers understand source code, detect bugs, improve code quality, and receive actionable suggestions.</p>

<br/>

[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Python](https://img.shields.io/badge/Python-3.12+-3776AB?logo=python&logoColor=white)](https://python.org)
[![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)](#)
[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)](#)
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)](#)
[![Render](https://img.shields.io/badge/Render-46E3B7?logo=render&logoColor=white)](#)

<br/>

**[Live Demo](https://codeinsight.onrender.com)** &nbsp;·&nbsp; **[API Docs](https://codeinsight.onrender.com/docs)**

</div>

---

## What is CodeInsight?

CodeInsight is a professional code analysis workspace. Paste any code and get three things back instantly:

| | What you get |
|---|---|
| **Explain** | Language detection, plain-English summary, complexity estimate, function and class inventory |
| **Debug** | 40+ pattern checks across 5 languages with exact line numbers, code snippets, and fix suggestions |
| **Improve** | Documentation gaps, error handling, testing, type safety - plus a 0–100 quality score and letter grade A–F |

No account required. No API key needed. Works fully offline.

---

## Preview

<!-- Add a screenshot of the live site here -->
<!-- ![CodeInsight Preview](assets/preview.png) -->

---

## Features

| Feature | Detail |
|---|---|
| **40+ Bug Patterns** | ZeroDivisionError, bare except, hardcoded secrets, eval(), memory leaks, XSS, NullPointerException, and more |
| **5 Languages** | Python, JavaScript, TypeScript, Java, C++ |
| **Full Analysis Endpoint** | One call - explain + debug + improve combined, with timing metrics |
| **Quality Score** | 0–100 score with letter grade A–F and prioritised suggestions |
| **File Upload** | Drag-drop or upload `.py` `.js` `.ts` `.java` `.cpp` |
| **Dark / Light Mode** | Persisted across sessions |
| **Query History** | Last 50 analyses saved locally |
| **Saved Favorites** | Bookmark and reload any analysis |
| **Download Results** | Export full report as `.txt` |
| **LLM-Ready** | Plug in OpenAI, Groq, Ollama, or any OpenAI-compatible provider via env vars |
| **Rate Limiting** | 30 requests/minute per IP - configurable |
| **Swagger Docs** | Interactive API docs at `/docs` |
| **Gzip Compression** | Automatic response compression |

### Languages and patterns

| Language | Patterns detected |
|---|---|
| **Python** | ZeroDivisionError, bare except, eval/exec, mutable defaults, hardcoded secrets, wildcard imports, global variables, missing type hints, string concat in loops, assert in production, comparison to None |
| **JavaScript** | var usage, loose equality `==`, console.log, callback hell, innerHTML XSS, unhandled promises |
| **TypeScript** | `any` type, non-null assertion `!`, unhandled promises, missing env var validation |
| **Java** | NullPointerException risk, raw generics, broad catch, String `==` comparison, System.exit |
| **C++** | Memory leaks, unsafe gets/scanf, `using namespace std`, signed/unsigned mismatch |

---

## Quick Start

### Prerequisites

- Python 3.11 or 3.12
- pip
- A modern browser (Chrome, Firefox, Edge, Safari)

### 1 - Clone

```bash
git clone https://github.com/peeyoush/AI-dev-assistant.git
cd AI-dev-assistant
```

### 2 - Run the backend

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload
```

| Endpoint | URL |
|---|---|
| API root | http://localhost:8000/ |
| Interactive docs | http://localhost:8000/docs |
| Health check | http://localhost:8000/health |

### 3 - Open the frontend

```bash
# No build step required - open directly in your browser
open frontend/index.html
```

Set the API URL field to `http://localhost:8000`, click **Ping** to confirm the green Connected status, then paste any code and click **Analyze Code**.

---

## API Reference

All endpoints accept `POST` with `Content-Type: application/json`.

**Request body**
```json
{ "code": "your code here", "language": "python" }
```

`language` is optional — the engine auto-detects it from the code.

---

### `POST /explanation/`

Returns a plain-English breakdown of the code.

```json
{
  "language": "Python",
  "summary": "A short Python snippet (5 lines) that performs a focused task.",
  "key_points": [
    "Written in Python — 5 non-blank lines of code.",
    "Defines 1 function: calculate.",
    "Contains conditional logic — branching control flow."
  ],
  "complexity": "Beginner",
  "line_count": 6,
  "function_count": 1,
  "class_count": 0
}
```

---

### `POST /debugging/`

Returns detected issues with line numbers, code snippets, and fix suggestions.

```json
{
  "issues": [
    {
      "type": "ZeroDivisionError",
      "line": 2,
      "description": "Potential division by zero — divisor may be 0 at runtime.",
      "suggestion": "Guard the divisor: if b == 0: return None",
      "severity": "error",
      "code_snippet": "result = a / b"
    }
  ],
  "summary": "Found 1 issue: 1 error, 0 warnings, 0 info.",
  "clean": false,
  "error_count": 1,
  "warning_count": 0,
  "info_count": 0
}
```

---

### `POST /suggestions/`

Returns improvement suggestion cards with a quality score.

```json
{
  "suggestions": [
    {
      "category": "Documentation",
      "description": "Less than 10% of lines are comments. Add docstrings.",
      "example": "\"\"\"Calculate the area of a circle given radius r.\"\"\"",
      "priority": "medium"
    }
  ],
  "overall_score": 72,
  "grade": "B",
  "next_step": "Good work. Address the medium-priority items next."
}
```

---

### `POST /analyze/`

All three analyses in one response with timing.

```json
{
  "provider": "rule-based",
  "model": "codeinsight-engine-v3",
  "explanation": { "...": "..." },
  "debugging":   { "...": "..." },
  "suggestions": { "...": "..." },
  "analysis_time_ms": 1.84
}
```

---

## Project Structure

```
AI-dev-assistant/
├── assets/                           # Logo and brand assets
│   ├── logo-dark.svg
│   ├── logo-light.svg
│   └── icon.svg
├── backend/
│   ├── app/
│   │   ├── main.py                   # FastAPI app, middleware, rate limiting
│   │   ├── schemas.py                # Pydantic v2 request/response models
│   │   ├── routers/
│   │   │   ├── analyze.py            # POST /analyze/
│   │   │   ├── debugging.py          # POST /debugging/
│   │   │   ├── explanation.py        # POST /explanation/
│   │   │   └── suggestions.py        # POST /suggestions/
│   │   └── services/
│   │       ├── code_assistant.py     # Rule-based engine — 40+ patterns, 5 languages
│   │       └── ai_provider.py        # Optional LLM abstraction layer
│   ├── requirements.txt
│   └── tests/
│       └── test_endpoints.py         # 22 tests across all endpoints and languages
├── frontend/
│   └── index.html                    # Complete UI — no build step, self-contained
├── .github/
│   └── workflows/
│       └── ci.yml                    # CI on Python 3.11 + 3.12, lint with Ruff
├── .env.example
├── Dockerfile
├── render.yaml
└── README.md
```

---

## Running Tests

```bash
cd backend
pytest -v
```

22 tests covering all endpoints, all 5 languages, 10+ individual bug patterns, suggestions scoring, full analysis, and edge cases including empty code, unicode, and single-line input.

Tests run automatically on every push via GitHub Actions across Python 3.11 and 3.12.

---

## Deployment

### Render - recommended, free tier

1. Go to [render.com](https://render.com) → New Web Service
2. Connect your repository - `render.yaml` configures everything automatically
3. Add environment variable: `PYTHON_VERSION` = `3.12.0`
4. Click Deploy - your app goes live at `https://your-service.onrender.com`

> **Note:** The free tier sleeps after 15 minutes of inactivity. The first request after sleep takes 30–60 seconds to wake up. This is expected.

### Docker

```bash
docker build -t codeinsight .
docker run -p 8000:8000 codeinsight
```

---

## Optional LLM Integration

CodeInsight works fully offline with its built-in rule-based engine. To enable richer AI-powered analysis, add these environment variables:

```env
LLM_ENABLED=true
LLM_API_KEY=your-key-here
LLM_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4o-mini
LLM_TIMEOUT_SECONDS=30
```

Compatible with **OpenAI**, **Groq** (free tier), **Together AI**, **Ollama** (local, free), and any OpenAI-compatible endpoint.

> Never commit API keys. Use environment variables or your host's secrets manager.

---

## Configuration Reference

| Variable | Default | Description |
|---|---|---|
| `RATE_LIMIT_PER_MINUTE` | `30` | Max requests per IP per minute |
| `LLM_ENABLED` | `false` | Enable LLM provider |
| `LLM_API_KEY` | — | API key for your LLM provider |
| `LLM_BASE_URL` | `https://api.openai.com/v1` | LLM base URL |
| `LLM_MODEL` | `gpt-4o-mini` | Model name |
| `LLM_TIMEOUT_SECONDS` | `30` | Request timeout in seconds |

Copy `.env.example` to `.env` and fill in values as needed.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI 0.115+, Pydantic v2, Python 3.12 |
| Frontend | HTML5, CSS3, Vanilla JS - no build step, zero dependencies |
| Testing | Pytest, FastAPI TestClient |
| Linting | Ruff |
| Deployment | Docker, Render |
| CI | GitHub Actions - Python 3.11 + 3.12 matrix |

---

## Roadmap

- [x] Rule-based code explanation engine
- [x] Bug detection — 40+ patterns across 5 languages
- [x] Improvement suggestions with quality score and letter grade A–F
- [x] Full-analysis combined endpoint with timing metrics
- [x] Rate limiting per IP — configurable
- [x] Gzip compression middleware
- [x] Dark / light theme, file upload, drag-and-drop, history, favorites, download
- [x] LLM provider abstraction layer — OpenAI, Groq, Ollama compatible
- [x] CI matrix — Python 3.11 + 3.12
- [ ] AST-based deep analysis for Python
- [ ] Per-user history with database backend (SQLite → PostgreSQL)
- [ ] VS Code extension
- [ ] AI-powered explanations — LLM integration GA
- [ ] Multi-file analysis support
- [ ] Diff view — before/after code improvements

---

<div align="center">

<br/>



</div>
# CodeInsight — Complete Technical Knowledge & Reverse-Engineering Document

**Target Workspace:** `AI-dev-assistant` (`CodeInsight`)  
**Analysis Date:** September 15, 2026  
**Purpose:** Comprehensive source-code reverse-engineering document for technical study and interview preparation.

---

# PART 1 — PROJECT OVERVIEW

### 1. What CodeInsight Is
CodeInsight is a self-contained, lightweight, web-based code analysis workspace. It allows developers to input source code snippets in multiple programming languages and instantly receive plain-English explanations, static analysis bug/vulnerability detections with line numbers, and actionable quality scores with improvement hints.

### 2. What Problem It Solves
Developers often struggle to quickly parse legacy code, spot subtle language-specific bug patterns (such as bare exceptions, `eval()` vulnerabilities, loose equality traps, or memory leaks), or evaluate code quality without setting up heavyweight static analyzers or IDE plugins. CodeInsight provides an instant zero-account, zero-key browser workspace to audit snippets across 5 languages.

### 3. Who the Intended User Is
- Software engineers performing rapid code reviews or debugging unfamiliar code.
- Computer science students and interview candidates auditing their snippet implementations.
- Team leads looking for quick automated sanity checks on code quality.

### 4. What the User Can Actually Do
- **Paste or Upload Code**: Enter raw code via an interactive text editor or drag-and-drop `.py`, `.js`, `.ts`, `.java`, `.cpp` files.
- **Auto-Detect Language**: Select a language explicitly or let the system auto-detect it using regex signature matching.
- **Execute Analysis Modes**:
  - **Explain**: Get plain-English summary, key points, complexity tier, line count, function count, and class count.
  - **Debug**: Run pattern matching to receive line-specific warnings, error categories, code snippets, and fix recommendations.
  - **Improve (Suggestions)**: Receive a 0–100 code quality score, letter grade (A–F), and category-wise recommendations (Documentation, Refactoring, Error Handling, Type Safety, Testing, Observability, Readability, Configuration).
  - **Full Analysis (`Analyze Code`)**: Execute all three analyses in a single unified HTTP call with execution timing metrics.
- **UI Workspace Capabilities**: Toggle dark/light themes, copy results to clipboard, export full reports as `.txt` files, save favorite analyses, clear workspace, view local history (last 50 queries), and check live backend API health status.

### 5. Major Features Currently Implemented
- **Regex-Based Multi-Language Bug Detection Engine**: 40+ rule patterns covering Python, JavaScript, TypeScript, Java, and C++.
- **Code Complexity Estimator**: Classifies code into `Beginner`, `Intermediate`, `Advanced`, or `Expert` based on line counts, control-flow branching keywords, and function count.
- **Quality Scoring System**: Calculates a score from 100 with weighted deductions for priority suggestions, returning letter grades A–F.
- **FastAPI Async Backend**: High-performance Python backend with CORS handling, Gzip compression, and custom per-IP rate limiting.
- **Optional LLM Integration Client**: Abstraction layer targeting OpenAI-compatible APIs (OpenAI, Groq, Together AI, Ollama).
- **Zero-Build Vanilla Frontend**: Standalone HTML5/CSS3/Vanilla JS UI served directly by FastAPI or static file hosting.
- **AST-Based Python Debugger (Isolated Test Suite)**: Python standard library `ast` parsing for division-by-zero, out-of-bounds indexing, type mismatch addition, and zero-argument parameter passing.

### 6. Technologies Actually Used

| Layer | Technology | Status in Codebase |
|---|---|---|
| **Frontend UI** | HTML5, Vanilla CSS3 (Custom Properties), Vanilla JavaScript (ES6+, Fetch API, LocalStorage) | **Active** (Served via `/app` static route or opened directly) |
| **Backend Core** | Python 3.12 / 3.11, FastAPI `0.115+`, Uvicorn `0.30+`, Pydantic v2 (`2.7+`) | **Active** (Primary runtime engine) |
| **Static Analysis** | Python `re` module (regex), Python `ast` module | **Active** (`re` used in live routes; `ast` used in isolated analyzer) |
| **HTTP / LLM Client** | HTTPX `0.27+` | **Active** (In optional `llm_analysis.py` service) |
| **Testing** | Pytest `8.0+`, Pytest-asyncio `0.23+`, FastAPI `TestClient` | **Active** (22 tests in `backend/tests/`) |
| **Linting** | Ruff | **Active** (In CI workflow) |
| **DevOps / Containers** | Docker (`python:3.12-slim`), Render (`render.yaml`), GitHub Actions (`ci.yml`) | **Active** |
| **Database & ORM** | SQLAlchemy 2.0, SQLite (`assistant.db`) | **Unmounted** (Files exist in `models.py` & `routers/`, but routes are NOT mounted in `main.py`) |
| **Auth & Security** | PyJWT (`jwt`), PBKDF2-HMAC-SHA256 (`hashlib`), HTTPBearer | **Unmounted** (Files exist in `security.py` & `auth.py`, but routes are NOT mounted in `main.py`) |
| **Caching & Observability**| Redis client (`redis-py`), Sentry SDK (`sentry-sdk`) | **Inactive Default** (Configured as optional fallback in `cache.py` / `error_tracking.py`) |
| **WebSockets** | None | **Not Implemented** |

### 7. Actual System Architecture Diagram

```
+-------------------------------------------------------------------------+
|                               Browser UI                                |
|   (frontend/index.html + script.js + style.css [LocalStorage / Fetch])  |
+-------------------------------------------------------------------------+
                                    |
                                    | HTTP POST / GET
                                    v
+-------------------------------------------------------------------------+
|                          FastAPI Application                            |
|                          (backend/app/main.py)                          |
|                                                                         |
|  [Middleware: CORSMiddleware | GZipMiddleware | Rate Limiter (30 req/m)]|
+-------------------------------------------------------------------------+
        |                  |                  |                  |
        v                  v                  v                  v
+---------------+  +---------------+  +---------------+  +---------------+
| /explanation  |  |  /debugging   |  |  /suggestions |  |   /analyze    |
| (router)      |  |  (router)     |  |  (router)     |  |   (router)    |
+---------------+  +---------------+  +---------------+  +---------------+
        |                  |                  |                  |
        +------------------+--------+---------+------------------+
                                    |
                                    v
+-------------------------------------------------------------------------+
|                 Rule-Based Analysis Engine (Service)                    |
|             (backend/app/services/code_assistant.py)                    |
|                                                                         |
|  - detect_language()   [LANG_SIGNATURES Regex]                          |
|  - run_explanation()   [AST/Regex metrics + complexity tiers]           |
|  - run_bug_detection() [40+ BUG_PATTERNS across Py/JS/TS/Java/C++]     |
|  - run_suggestions()   [Doc, Refactor, Errors, Score 0-100, Grade A-F]   |
+-------------------------------------------------------------------------+
                                    |
            +-----------------------+-----------------------+
            | (Optional - LLM_ENABLED=true)                 | (Isolated Test Suite)
            v                                               v
+------------------------------------+    +-------------------------------+
|    LLM Integration Client          |    | Python AST Analyzer           |
| (app/services/llm_analysis.py)     |    | (code_assistant.py:debug_code)|
|  - OpenAI-compatible HTTPX POST    |    |  - ast.parse(code)            |
|  - OpenAI / Groq / Ollama API      |    |  - Div/0, Index, Type check   |
+------------------------------------+    +-------------------------------+
```

---

# PART 2 — COMPLETE REPOSITORY STRUCTURE

```
AI-dev-assistant/
├── .github/
│   └── workflows/
│       └── ci.yml                   # GitHub Actions workflow (pytest + ruff)
├── assets/                          # Branding SVGs (logo-dark, logo-light, icon)
├── backend/
│   ├── app/
│   │   ├── __init__.py              # Package marker
│   │   ├── config.py                # Pydantic/os.getenv settings loader
│   │   ├── database.py              # SQLAlchemy engine & session setup (unmounted)
│   │   ├── main.py                  # Primary FastAPI entry point & middleware
│   │   ├── middleware.py            # Extra unused middleware (ReqID, SizeLimit, RateLimit)
│   │   ├── models.py                # SQLAlchemy ORM models (User, History, Favorite, Share)
│   │   ├── schemas.py               # Active Pydantic v2 request/response models
│   │   ├── security.py              # Passlib PBKDF2 hashing & PyJWT token management
│   │   ├── routers/
│   │   │   ├── __init__.py          # Package marker
│   │   │   ├── analyze.py           # Active: POST /analyze/
│   │   │   ├── auth.py              # Unmounted: POST /auth/signup, /login, /me
│   │   │   ├── chat.py              # Unmounted: POST /chat, /chat/message
│   │   │   ├── debugging.py         # Active: POST /debugging/
│   │   │   ├── explanation.py       # Active: POST /explanation/
│   │   │   ├── share.py             # Unmounted: POST /share/, GET /share/{token}
│   │   │   ├── suggestions.py       # Active: POST /suggestions/
│   │   │   └── user_data.py         # Unmounted: GET/POST/DELETE /user/history & /favorites
│   │   └── services/
│   │       ├── __init__.py          # Package marker
│   │       ├── ai_provider.py       # Standalone basic HTTPX LLM function helper
│   │       ├── cache.py             # Memory/Redis dual-backend cache wrapper
│   │       ├── code_assistant.py    # Core Engine (40+ rules, scoring, explanation, AST analyzer)
│   │       ├── error_tracking.py    # Sentry SDK initialization helper
│   │       └── llm_analysis.py      # LLMAnalysisClient class for structured LLM parsing
│   ├── tests/
│   │   ├── conftest.py              # Sys.path fixture setup
│   │   ├── test_endpoints.py        # 22 active API & rule engine test cases
│   │   ├── test_ping.py             # Simple health check endpoint test
│   │   └── test_python_ast_analyzer.py # Isolated tests for Python AST `debug_code`
│   ├── .dockerignore                # Backend docker ignore rules
│   ├── Dockerfile                   # Docker image definition (same as root)
│   └── requirements.txt             # Python dependencies
├── frontend/
│   ├── index.html                   # Single-Page UI layout & inline styles (~1700 lines)
│   ├── script.js                    # UI logic, fetch client, state, storage (~530 lines)
│   ├── style.css                    # Legacy/External CSS overrides (~15KB)
│   └── public/                      # Static web assets directory
├── .dockerignore                    # Root docker ignore rules
├── .env.example                     # Environment configuration template
├── Dockerfile                       # Production Docker build file
├── render.yaml                      # Render Blueprint deployment configuration
└── README.md                        # Project documentation
```

### Component Breakdown

| Category | Directories & Files | Description & Purpose |
|---|---|---|
| **Frontend** | `frontend/index.html`, `script.js`, `style.css` | Zero-dependency SPA interface. Handles user input, fetch calls to `/analyze/`, `/debugging/`, etc., localStorage management, dark/light theme, download/copy export. |
| **Backend Core** | `backend/app/main.py`, `config.py` | FastAPI application instantiation, middleware attachment, settings initialization from `.env`, static file mounting (`/app`), global exception handler. |
| **API Routers (Active)** | `backend/app/routers/analyze.py`, `debugging.py`, `explanation.py`, `suggestions.py` | Mounts POST endpoints for `/analyze/`, `/debugging/`, `/explanation/`, and `/suggestions/`. |
| **API Routers (Unmounted)** | `backend/app/routers/auth.py`, `chat.py`, `share.py`, `user_data.py` | Draft router files in the codebase containing endpoints for user auth, AI chat, snippet sharing, and DB history/favorites. **Not included in `main.py`**. |
| **Analysis Engine** | `backend/app/services/code_assistant.py` | The main engine. Implements `detect_language`, `estimate_complexity`, `run_bug_detection` (40+ regex rules), `run_suggestions` (quality score calculation), `run_explanation`, and `debug_code` (Python AST parser). |
| **AI / LLM Layer** | `backend/app/services/llm_analysis.py`, `ai_provider.py` | Async HTTPX client wrappers for sending code snippets to OpenAI-compatible endpoints (`/v1/chat/completions`) and extracting structured JSON. |
| **Models & DB** | `backend/app/database.py`, `models.py`, `schemas.py` | `schemas.py` defines active Pydantic v2 request/response models. `models.py` defines SQLAlchemy ORM models (`User`, `QueryHistory`, `FavoriteResult`, `SharedSnippet`), unmounted in main. |
| **Auth & Security** | `backend/app/security.py`, `middleware.py` | `security.py` has PBKDF2-HMAC password hashing and PyJWT token decode/encode (unmounted). `middleware.py` has rate limiting and size limit helpers. Active rate limiter is inside `main.py`. |
| **Tests** | `backend/tests/test_endpoints.py`, `test_ping.py`, `test_python_ast_analyzer.py` | Pytest test suite covering health endpoints, rule detection across 5 languages, suggestions scoring, payload validation, and AST parsing. |
| **Docker & Deployment**| `Dockerfile`, `render.yaml`, `.github/workflows/ci.yml` | Container definition (`python:3.12-slim`), Render deployment configuration, and GitHub Actions CI workflow (Python 3.11/3.12 test matrix + Ruff linter). |

---

# PART 3 — COMPLETE APPLICATION FLOW

Here is the exact runtime data flow when a user analyzes a snippet of code in CodeInsight:

```
[1. Browser UI] 
   User pastes code into #codeInput textarea and clicks "Analyze Code".
   │
   ▼
[2. script.js: runAnalysis()] 
   - Reads codeInput.value.trim(). Validates non-empty string.
   - Sets runBtn to loading state ("⟳ Analyzing...").
   - Resolves target API URL: getApiUrl() -> "http://localhost:8000/analyze/".
   - Sends HTTP POST request: fetch(url, { method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify({ code }) }).
   │
   ▼
[3. FastAPI Backend: main.py Middleware] 
   - HTTP request arrives at Uvicorn / FastAPI server.
   - GZipMiddleware checks response compression suitability.
   - CORSMiddleware validates origin.
   - add_process_time_header middleware triggers:
       * Captures start timestamp via time.perf_counter().
       * Extracts client IP (request.client.host).
       * Calls check_rate_limit(ip): Purges IP timestamps older than 60s. If count >= 30, raises HTTP 429.
   │
   ▼
[4. Router: analyze.py: analyze()] 
   - Receives Pydantic request model req: CodeRequest.
   - CodeRequest validator (schemas.py: code_must_not_be_empty) verifies:
       * Code string length is between 1 and 50,000 characters.
   - Calls service function: full_analysis(req.code, req.language).
   │
   ▼
[5. Service Engine: code_assistant.py: full_analysis()] 
   - Starts timer t0 = time.perf_counter().
   - Step 5a: detect_language(code, hint)
       * Iterates through LANG_SIGNATURES dictionary (Python, JS, TS, Java, C++).
       * Counts regex signature matches per language. Returns best match.
   - Step 5b: run_explanation(code, language)
       * Counts total lines, non-blank lines.
       * Calls estimate_complexity(code) -> beginner/intermediate/advanced/expert.
       * Extracts functions, classes, imports, loops, condition checks, and recursion signals using regex.
       * Assembles key_points array and summary text.
   - Step 5c: run_bug_detection(code, language)
       * Iterates over 40+ BugPattern objects in BUG_PATTERNS.
       * Filters patterns matching current language.
       * Scans code line-by-line using re.search(pattern, line).
       * Appends issue dictionaries containing type, line, description, suggestion, severity, code_snippet.
   - Step 5d: run_suggestions(code, language)
       * Evaluates code metrics (comment ratio, function length > 40 lines, magic numbers, I/O try/except, type hints, tests, logging, env var validation).
       * Calculates deductions: high=15, medium=7, low=3 points.
       * Calculates overall_score = max(0, min(100, 100 - deductions)). Assigns grade (A–F) and next_step.
   - Step 5e: Calculates analysis_time_ms = (time.perf_counter() - t0) * 1000.
   - Returns consolidated AnalyzeResponse dictionary.
   │
   ▼
[6. Router -> FastAPI Response Transformation] 
   - FastAPI serializes dictionary against AnalyzeResponse Pydantic schema.
   - add_process_time_header appends headers: X-Process-Time-Ms and X-CodeInsight-Version: 3.0.0.
   - Sends HTTP 200 OK JSON response payload to client.
   │
   ▼
[7. script.js: renderResult() & saveHistory()] 
   - UI parses JSON payload.
   - Builds HTML markup for Explanation, Debugging issues list, and Suggestions card lists.
   - Injects HTML into #outputBox DOM node.
   - Updates LocalStorage key 'qyverix_history' with snippet preview, timestamp, and mode.
   - Re-enables runBtn ("▶ Analyze Code").
```

---

# PART 4 — BACKEND DEEP DIVE

### Application Startup & Lifecycle
- **Entry Point**: `backend/app/main.py`
- **Lifespan Manager**: An `@asynccontextmanager` function `lifespan(app: FastAPI)` logs startup (`🚀 CodeInsight backend starting…`) and shutdown (`🛑 CodeInsight backend shutting down…`).
- **Middleware Chain**:
  1. `GZipMiddleware`: Minimum size threshold 1000 bytes.
  2. `CORSMiddleware`: Permits all origins (`*`), credentials, methods, and headers.
  3. `add_process_time_header`: Custom http-level middleware calculating execution time, applying rate limiting on `/explanation/`, `/debugging/`, `/suggestions/`, and `/analyze/`, and injecting custom response headers (`X-Process-Time-Ms`, `X-CodeInsight-Version`).

### API Endpoints Reference Table

| Endpoint | HTTP Method | Purpose | Request Payload | Response Payload | Auth | Main Function | File Involved | Status |
|---|---|---|---|---|---|---|---|---|
| `/` | GET | Root API status & info | None | `HealthResponse` | None | `root()` | `main.py` | Active |
| `/health` | GET | Health check probe | None | `HealthResponse` | None | `health_check()` | `main.py` | Active |
| `/ping` | GET | Connectivity test | None | `{"message":"pong"}` | None | `ping()` | `main.py` | Active |
| `/explanation/` | POST | Plain-English code explanation | `CodeRequest` | `ExplanationResponse` | None | `explain()` | `routers/explanation.py` | Active |
| `/debugging/` | POST | Static bug pattern analysis | `CodeRequest` | `DebuggingResponse` | None | `debug()` | `routers/debugging.py` | Active |
| `/suggestions/` | POST | Quality score & improvement suggestions | `CodeRequest` | `SuggestionsResponse` | None | `suggest()` | `routers/suggestions.py` | Active |
| `/analyze/` | POST | Unified full analysis (explain + debug + suggest) | `CodeRequest` | `AnalyzeResponse` | None | `analyze()` | `routers/analyze.py` | Active |
| `/auth/signup` | POST | User registration | `SignupRequest` | `AuthResponse` | None | `signup()` | `routers/auth.py` | **Unmounted** |
| `/auth/login` | POST | User authentication | `LoginRequest` | `AuthResponse` | None | `login()` | `routers/auth.py` | **Unmounted** |
| `/auth/me` | GET | Current user profile | None | `UserProfileResponse` | Bearer JWT | `me()` | `routers/auth.py` | **Unmounted** |
| `/chat` | POST | General AI coding assistance | `ChatRequest` | `ChatResponse` | None | `chat()` | `routers/chat.py` | **Unmounted** |
| `/chat/message` | POST | Multi-level AI chat response | `ChatMessageRequest` | `ChatMessageResponse` | None | `chat_message()` | `routers/chat.py` | **Unmounted** |
| `/share/` | POST | Create shareable snippet record | `ShareCreateRequest` | `ShareRecord` | None | `create_share()` | `routers/share.py` | **Unmounted** |
| `/share/{token}` | GET | Retrieve shared snippet record | None | `ShareRecord` | None | `get_share()` | `routers/share.py` | **Unmounted** |
| `/user/history` | GET/POST/DEL| Manage query history in DB | `HistoryCreateRequest` | `HistoryRecord` | Bearer JWT | `list_history()` etc. | `routers/user_data.py` | **Unmounted** |
| `/user/favorites`| GET/POST/DEL| Manage favorite snippets in DB | `FavoriteCreateRequest`| `FavoriteRecord` | Bearer JWT | `list_favorites()` etc. | `routers/user_data.py` | **Unmounted** |

---

# PART 5 — CODE ANALYSIS ENGINE

### How Source Code Enters and Is Processed
Source code enters as a JSON string payload inside `CodeRequest` via `POST` to FastAPI endpoints. Pydantic validates that the code string is non-empty and does not exceed 50,000 characters.

### Language Detection Mechanism
Language detection is implemented in `code_assistant.py:detect_language()`. If no hint is provided (or if the hint is not recognized), it runs regex signature checks defined in `LANG_SIGNATURES`:

- **Python**: `\bdef\s+\w+\s*\(`, `\bimport\s+\w+`, `\bprint\s*\(`, `:\s*$`, `\belif\b`, `\bself\b`, `#.*`, `\bNone\b`
- **JavaScript**: `\bconst\b|\blet\b|\bvar\b`, `function\s+\w+\s*\(`, `=>\s*[{(]`, `console\.log\(`, `require\(`, `export\s+(default|const)`
- **TypeScript**: `:\s*(string|number|boolean|any|void|never)\b`, `\binterface\s+\w+`, `\btype\s+\w+\s*=`, `<\w+>`, `as\s+\w+`, `readonly\s+\w+`
- **Java**: `\bpublic\s+(class|void|static)\b`, `\bSystem\.out\.print`, `\bimport\s+java\.`, `@Override`, `\bnew\s+\w+\s*\(`
- **C++**: `#include\s*<`, `\bstd::\w+`, `\bcout\s*<<`, `\bint\s+main\s*\(`, `::\w+`

Each matching pattern increments the language score by +1. The language with the highest score is returned (defaulting to `"Unknown"` if score is 0).

### Static Rule Analysis Engine (`run_bug_detection`)
Iterates over a list of `BugPattern` dataclass instances defined in `code_assistant.py`. It splits code into lines and executes `re.search()` against each line. The severity levels used are `error`, `warning`, and `info`.

#### Major Implemented Rules Audit Table

| Rule Name | File | Language(s) | What It Detects | Detection Pattern (Regex) | Code Trigger Example | Severity | Limitations |
|---|---|---|---|---|---|---|---|
| **ZeroDivisionError** | `code_assistant.py` | Python | Division operations | `\w+\s*/\s*\w+` | `result = a / b` | `error` | Triggers on valid divisions if divisor is non-zero at runtime |
| **Bare Except** | `code_assistant.py` | Python | Catching all exceptions without specification | `except\s*:` | `try: pass\nexcept: pass` | `warning` | None |
| **Eval Usage** | `code_assistant.py` | Python, JS | Arbitrary code execution via `eval()` | `\beval\s*\(` | `x = eval(user_input)` | `error` | Does not catch aliased eval calls |
| **Exec Usage** | `code_assistant.py` | Python | Arbitrary code execution via `exec()` | `\bexec\s*\(` | `exec("import os")` | `error` | None |
| **Mutable Default Arg**| `code_assistant.py` | Python | Mutable defaults in function headers | `def\s+\w+\s*\([^)]*=\s*(\[]\|\{\}\|(\))` | `def func(items=[]):` | `warning` | Does not catch custom mutable objects |
| **Hardcoded Secret** | `code_assistant.py` | All | Plaintext passwords/tokens assigned in code | `(password\|secret\|api_key\|token\|passwd)\s*=\s*['\"][^'\"]{4,}['\"]` | `token = "secret1234"` | `error` | False positives on test dummy strings |
| **Print Debugging** | `code_assistant.py` | Python | Print calls containing debug words | `\bprint\s*\(.*debug\|TODO\|FIXME\|HACK` | `print("debug x:", x)` | `info` | Misses normal print statements |
| **Wildcard Import** | `code_assistant.py` | Python | `from module import *` | `from\s+\w+\s+import\s+\*` | `from os import *` | `warning` | None |
| **Global Variable** | `code_assistant.py` | Python | `global` keyword usages | `^\s*global\s+\w+` | `global state` | `info` | Matches top-level or scoped global declarations |
| **No Type Hints** | `code_assistant.py` | Python | Defs missing type annotations | `def\s+\w+\s*\([^)]*\)\s*:` | `def add(a, b):` | `info` | Does not verify return type annotations |
| **String Concat Loop** | `code_assistant.py` | Python | O(n²) string accumulation in loops | `(for\|while).+\n.+\+=\s*['\"]` | `for x in list:\n  s += "a"` | `warning` | Single line matching across newlines |
| **Assert Production** | `code_assistant.py` | Python | Production code assert statements | `^\s*assert\s+` | `assert x > 0` | `warning` | None |
| **Var Usage** | `code_assistant.py` | JS, TS | Legacy `var` variable scope | `\bvar\s+\w+` | `var name = "test";` | `warning` | None |
| **== Comparison** | `code_assistant.py` | JS, TS | Loose equality comparison | `[^=!]==[^=]\|[^=!]!=[^=]` | `if (x == "1")` | `warning` | Can trigger inside comments |
| **Console.log Left In**| `code_assistant.py` | JS, TS | Leftover console logging | `console\.(log\|warn\|error\|debug)\s*\(` | `console.log(data);` | `info` | None |
| **Callback Hell** | `code_assistant.py` | JS, TS | Nested callback structures | `function\s*\([^)]*\)\s*\{[\s\S]{0,200}function...` | Nested functions | `warning` | Regex regex distance window of 200 chars |
| **Any Type** | `code_assistant.py` | TS | TypeScript `any` type escape | `:\s*any\b` | `let data: any;` | `warning` | None |
| **Non-null Assertion**| `code_assistant.py` | TS | Overriding null checks with `!` | `\w+![\.\[]` | `user!.name` | `warning` | None |
| **Promise Not Awaited**| `code_assistant.py` | JS, TS | Unawaited async/fetch calls | `(?<!await\s)\bfetch\s*(\|\bnew\s+Promise\s*\(` | `fetch("/api/data");` | `error` | Negative lookbehind assertion |
| **InnerHTML XSS** | `code_assistant.py` | JS, TS | Direct DOM innerHTML assignment | `\.innerHTML\s*=` | `elem.innerHTML = HTML;` | `error` | None |
| **Null Pointer Risk** | `code_assistant.py` | Java | Dereferencing without null check | `\w+\s*\.\s*\w+\s*\(` | `str.length();` | `warning` | Triggers on safe dereferences |
| **Raw Type** | `code_assistant.py` | Java | Unparameterized collections | `\b(List\|Map\|Set\|Collection)\s+\w+\s*=` | `List list = new ArrayList();` | `warning` | None |
| **Catch Exception** | `code_assistant.py` | Java | Catching generic Exception | `catch\s*\(\s*Exception\s+\w+\s*\)` | `catch (Exception e)` | `warning` | None |
| **String == Java** | `code_assistant.py` | Java | Identity check on Java strings | `" [^"]+"\s*==\s*\w+\|\w+\s*==\s*"[^\"]+"` | `if (s == "hello")` | `error` | Only checks string literal equality |
| **System.exit** | `code_assistant.py` | Java | Terminating JVM directly | `System\.exit\s*\(` | `System.exit(0);` | `error` | None |
| **Memory Leak** | `code_assistant.py` | C++ | Raw `new` without `delete` | `\bnew\b(?!.*\bdelete\b)` | `int* p = new int(5);` | `error` | Single-file lookahead check |
| **Unsafe gets/scanf** | `code_assistant.py` | C++ | Buffer overflow risk functions | `\bgets\s*(\|\bscanf\s*\(` | `gets(buffer);` | `error` | None |
| **Using namespace std**| `code_assistant.py` | C++ | Global namespace pollution | `using\s+namespace\s+std\s*;` | `using namespace std;` | `warning` | None |
| **Signed/Unsigned** | `code_assistant.py` | C++ | Signed int comparison with `.size()` | `\bint\b.*\bsize\(\)\|\.size\(\)\s*[<>]=?\s*\bint\b` | `for(int i=0; i<v.size())` | `warning` | None |

### AST Parsing Engine (`debug_code`)
Located in `code_assistant.py:debug_code()` (lines 436–518). Uses Python's native `ast` library to parse code into an Abstract Syntax Tree:
1. `ast.parse(code)`: Catches `SyntaxError` and returns immediately with line number and syntax error details.
2. Tracks variable definitions assigned to literal lists or strings in a dictionary `container_lengths`.
3. Traverses tree via `ast.walk(tree)` to inspect AST node types:
   - `ast.BinOp` with `ast.Div` and right side `ast.Constant(value=0)` -> Detects explicit `ZeroDivisionError`.
   - `ast.Subscript` with constant integer slice against known `container_lengths` -> Detects `Index Error Risk`.
   - `ast.BinOp` with `ast.Add` between string and int constants -> Detects `Type Error Risk`.
   - Function definition parameter scanning: Checks if a parameter is used as a divisor in a `BinOp`, and tracks calls passing literal `0` to that position.

**Relationship between Regex and AST engines**: The live HTTP API routes (`/debugging/` and `/analyze/`) execute `run_bug_detection` (the multi-language regex pattern engine). The AST-based `debug_code()` function is **decoupled from the HTTP API** and is called specifically inside `tests/test_python_ast_analyzer.py`.

---

# PART 6 — QUALITY SCORING

### Formula & Scoring Mechanics
The quality scoring system is implemented in `code_assistant.py:run_suggestions()`.

$$\text{Deductions} = \sum_{s \in \text{suggestions}} \text{Weight}(s.\text{priority})$$

Where weights are assigned as:
- `"high"` priority: **15 points** deduction
- `"medium"` priority: **7 points** deduction
- `"low"` priority: **3 points** deduction (default: 5 points)

$$\text{Overall Score} = \max\left(0, \min\left(100, 100 - \text{Deductions}\right)\right)$$

### Grade Assignment Matrix

| Score Range | Letter Grade | Default Next Step Message |
|---|---|---|
| **90 – 100** | **A** | "Excellent code! Consider adding integration tests." |
| **75 – 89** | **B** | "Good work. Address the medium-priority items next." |
| **60 – 74** | **C** | "Solid foundation. Focus on error handling and testing." |
| **40 – 59** | **D** | "Needs significant improvement — start with the high-priority items." |
| **0 – 39** | **F** | "Major issues detected. Refactor with error handling, tests, and type safety." |

### Suggestion Rules & Triggers
1. **Documentation Gap**: Comment-to-code ratio < 10% -> Priority `medium` (-7)
2. **Long Functions**: Function body > 40 lines -> Priority `high` (-15)
3. **Magic Numbers**: Unnamed numbers detected -> Priority `medium` (-7)
4. **Missing Error Handling**: Python I/O (`open`, `requests`, `json.loads`) without `try` block -> Priority `high` (-15)
5. **Missing Type Hints**: Python function signatures missing `:` annotations -> Priority `medium` (-7)
6. **Missing Unit Tests**: Code lacking `test_`, `unittest`, or `pytest` keywords -> Priority `high` (-15)
7. **Unstructured Logging**: Code using `print()` without importing `logging` -> Priority `medium` (-7)
8. **Unvalidated Config**: JS/TS accessing `process.env` without `dotenv`/`zod` -> Priority `medium` (-7)

### Worked Calculation Example
Consider a Python snippet with **no comments**, **no unit tests**, and **unannotated functions**:
- Trigger 1 (Documentation): Priority `medium` -> Deduction = **7**
- Trigger 6 (Testing): Priority `high` -> Deduction = **15**
- Trigger 5 (Type Safety): Priority `medium` -> Deduction = **7**
- **Total Deductions**: $7 + 15 + 7 = 29$
- **Overall Score**: $100 - 29 = 71$
- **Grade**: **C** (since $60 \le 71 < 75$)

---

# PART 7 — AI / LLM FEATURES

### Provider & Model Integration
- **Default Status**: Disabled (`LLM_ENABLED=false`). When enabled, it works with any OpenAI-compatible provider (OpenAI, Groq, Together AI, Ollama).
- **Configuration Variables**:
  - `LLM_ENABLED` (boolean, default: `False`)
  - `LLM_API_KEY` (string, required if enabled)
  - `LLM_BASE_URL` (default: `https://api.openai.com/v1`)
  - `LLM_MODEL` (default: `gpt-4o-mini`)
  - `LLM_TIMEOUT_SECONDS` (integer, default: `30`)

### LLM Implementation Files
1. `backend/app/services/ai_provider.py`: Provides `async def call_llm(system: str, user: str) -> str | None`. Sends HTTP POST request using `httpx.AsyncClient` with `temperature: 0.2` and `max_tokens: 1024`.
2. `backend/app/services/llm_analysis.py`: Contains `LLMAnalysisClient` class:
   - `summarize_code(code, language_guess)`: Requests plain text explanation.
   - `analyze_code_structured(code, language_guess)`: Requests structured JSON response matching schema `{ explanation, debugging, suggestions, complexity, optimized_version }`. Includes robust JSON extractor `_extract_json()` to handle markdown code block wrapping (` ```json `).
   - `chat_reply(message, code, history, level)`: Contextual chat helper.

### LLM Execution Flow (When Enabled)
```
User -> Frontend -> POST /chat -> chat.py
 -> checks llm_analysis_client.enabled
 -> calls llm_analysis_client.chat_reply()
 -> HTTPX POST to LLM_BASE_URL/chat/completions
 -> Parses response -> Returns JSON to Frontend
 (Fallback: If LLM is disabled or throws an exception, calls local chat_fallback_reply())
```

---

# PART 8 — AUTHENTICATION AND SECURITY

### Reality Check: Auth Status
Authentication code is **fully written in service/security files** but **UNMOUNTED in the primary live application (`main.py`)**. The live endpoints (`/analyze/`, `/debugging/`, `/explanation/`, `/suggestions/`) require **no authentication token**.

### Codebase Auth Implementation Details (`security.py` & `auth.py`)
- **Password Hashing**: Implemented in `security.py:hash_password()` using **PBKDF2-HMAC-SHA256**:
  - Generates 16 bytes of random salt via `os.urandom(16)`.
  - Performs 100,000 iterations of SHA256 hashing.
  - Formats stored string as `<salt_hex>:<digest_hex>`.
  - Verification (`verify_password`) uses constant-time string comparison `hmac.compare_digest()`.
- **JWT Token Management**:
  - Algorithm: `HS256` (configurable via `JWT_ALGORITHM`).
  - Secret: Configurable via `JWT_SECRET` (default string in `config.py`).
  - Expiration: `ACCESS_TOKEN_MINUTES` (default: 720 minutes / 12 hours).
  - Subject Claim: User ID (`"sub": str(user_id)`).
  - Extraction Dependency: `get_current_user()` reads `HTTPBearer` authorization header.

### Application Security Controls

| Mechanism | Codebase Implementation Status | Details |
|---|---|---|
| **CORS Policy** | **Implemented & Active** | `CORSMiddleware` in `main.py` allowing all origins, credentials, methods, headers. |
| **Rate Limiting** | **Implemented & Active** | Custom HTTP middleware in `main.py` tracking client IP requests in a 60-second window (Limit: 30 req/min). |
| **Input Validation** | **Implemented & Active** | Pydantic v2 `field_validator` in `schemas.py` enforcing non-empty string and 50,000 character limit. |
| **Payload Size Limit**| **Implemented in `middleware.py` (Unmounted)** | Checks `Content-Length` header against `MAX_REQUEST_BYTES` (1 MB) returning HTTP 413. |
| **Sandboxing** | **Not Applicable / Implemented** | Code is analyzed statically via regex/AST; code is **never executed**. |

---

# PART 9 — WEBSOCKETS / REAL-TIME COLLABORATION

### Complete Analysis of WebSockets Status
- **Claimed in Documentation / Resumes**: "Real-time collaboration using WebSockets".
- **Codebase Reality**: **0 WebSocket endpoints exist in the codebase**. There are no `WebSocket` routers in FastAPI, no `ws://` client scripts in `frontend/script.js`, and no real-time room/broadcasting libraries installed in `requirements.txt`.
- **Conclusion**: WebSockets / real-time collaboration feature is **NOT IMPLEMENTED**.

---

# PART 10 — DATABASE

### Database Setup & ORM
- **Database Engine**: SQLite by default (`sqlite:///./assistant.db`).
- **ORM Library**: SQLAlchemy 2.0 (`declarative_base`, `mapped_column`, `relationship`).
- **Connection File**: `backend/app/database.py` (includes `check_same_thread: False` for SQLite compatibility).
- **Mount Status**: Database models and session generators are fully defined but **UNMOUNTED** in `main.py`. The active frontend uses browser `localStorage`.

### Database Schema Table (Defined in `models.py`)

| Table Name | Purpose | Important Fields | Relationships | Status |
|---|---|---|---|---|
| `users` | User credentials | `id` (PK, Int), `email` (String 320, Unique, Index), `password_hash` (String 256), `created_at` (DateTime) | One-to-Many with `query_history` and `favorite_results` | Unmounted |
| `query_history` | Historical queries | `id` (PK, Int), `user_id` (FK -> users.id), `action` (String 50), `code` (Text), `result_json` (Text), `created_at` | Belongs-to `users` | Unmounted |
| `favorite_results`| Saved snippets | `id` (PK, Int), `user_id` (FK -> users.id), `title` (String 200), `action` (String 50), `code`, `result_json`, `created_at` | Belongs-to `users` | Unmounted |
| `shared_snippets` | Public shares | `id` (PK, Int), `token` (String 64, Unique, Index), `action`, `code`, `result_json`, `created_at` | None | Unmounted |

```
  +------------------+             +-----------------------+
  |      users       |             |     query_history     |
  +------------------+             +-----------------------+
  | id (PK)          |<-----------+| id (PK)               |
  | email            | 1         N | user_id (FK)          |
  | password_hash    |             | action                |
  | created_at       |             | code                  |
  +------------------+             | result_json           |
           |                       | created_at            |
           | 1                     +-----------------------+
           |
           |                       +-----------------------+
           |                       |   favorite_results    |
           |                       +-----------------------+
           +----------------------+| id (PK)               |
                                 N | user_id (FK)          |
                                   | title                 |
                                   | action                |
                                   | code                  |
                                   | result_json           |
                                   | created_at            |
                                   +-----------------------+
```

---

# PART 11 — FRONTEND

### Architecture & Tech Stack
- **Framework**: Zero-framework Vanilla HTML5 / CSS3 / JavaScript (ES6+).
- **Build System**: No build step required (`vite`, `webpack`, `babel` are not used). Served as static files directly by FastAPI or opened from local disk (`file://`).
- **File Structure**:
  - `frontend/index.html`: Contains UI layout, navigation header, status indicators, editor textareas, action buttons, tab selectors, output displays, and inline CSS styles (~1700 lines).
  - `frontend/script.js`: Handles DOM selection, event listeners, backend health polling, fetch requests, UI rendering, theme state, and LocalStorage persistence (~530 lines).

### Key Client Features & Implementations
- **API Auto-Discovery & Health Check (`checkConnection()`)**: Periodically pings `/health` using `fetch()` with `AbortSignal.timeout(3000)`. Automatically fallback-switches between `http://localhost:8000` and relative origin based on window location.
- **LocalStorage State Persistence**:
  - `qyverix_theme`: Persists `'dark'` or `'light'` theme attributes.
  - `qyverix_history`: Array of last 50 queries.
  - `qyverix_favorites`: Array of saved favorite analyses.
  - `qyverix_api_url`: Persists custom target backend API host.
- **File Upload Handler**: Uses browser `FileReader` API to read contents of uploaded `.py`, `.js`, `.ts`, `.java`, `.cpp` files directly into the editor text area.

---

# PART 12 — DOCKER

### Docker Configuration Analysis
- **Dockerfiles**: Root `Dockerfile` and `backend/Dockerfile` are identical.
- **Base Image**: `python:3.12-slim` (minimal official Debian-based Python 3.12 image).
- **Build & Execution Steps**:
  1. `WORKDIR /app`
  2. `COPY backend/requirements.txt ./requirements.txt`
  3. `RUN pip install --no-cache-dir -r requirements.txt`
  4. `COPY backend/ ./backend/`
  5. `COPY frontend/ ./frontend/`
  6. `EXPOSE 8000`
  7. `CMD ["uvicorn", "backend.app.main:app", "--host", "0.0.0.0", "--port", "8000"]`

### Docker Architecture Diagram

```
+------------------------------------------------------------------+
|                    Docker Container (Port 8000)                  |
|  Base Image: python:3.12-slim                                    |
|                                                                  |
|  /app/backend/  ---> Python FastAPI API (Uvicorn Server)       |
|  /app/frontend/ ---> Mounted Static Files (/app route)           |
|                                                                  |
|  Command: uvicorn backend.app.main:app --host 0.0.0.0 --port 8000|
+------------------------------------------------------------------+
```

### Docker Compose
- **Status**: No `docker-compose.yml` file is present in the repository.

---

# PART 13 — GITHUB ACTIONS / CI/CD

### CI Workflow Configuration (`.github/workflows/ci.yml`)
- **Trigger**: Pushes to `main` and `develop` branches.
- **Runner**: `ubuntu-latest`.
- **Jobs**:
  1. **`test` Job**:
     - **Strategy Matrix**: Runs on Python `3.11` and Python `3.12`.
     - **Steps**:
       - `actions/checkout@v4`
       - `actions/setup-python@v5` with pip caching.
       - Runs `pip install -r requirements.txt` inside `backend/`.
       - Executes `pytest -v --tb=short` inside `backend/`.
  2. **`lint` Job**:
     - Runs on Python `3.12`.
     - Installs `ruff`.
     - Executes `ruff check backend/app --select E,F,W --ignore E501`.

### Continuous Deployment (CD)
- **Status**: No CD deployment job in GitHub Actions. Deployment is configured via Render auto-deploy watching `render.yaml`.

---

# PART 14 — TESTING

### Test Suite Structure (`backend/tests/`)
- **Framework**: Pytest `8.0+` with FastAPI `TestClient`.
- **Total Test Cases**: 22 automated test cases.

### Test Coverage Breakdown
- **Health & System**: Test root endpoint `/`, `/health`, and `/ping`.
- **Explanation**: Test Python explanation response, language hint fallback, empty code validation (422), oversized payload validation (422).
- **Debugging Endpoint**: Test detection of zero division, hardcoded secrets, bare except, `eval()` usage, clean code state, JS/Java/C++ code inputs, and response field integrity.
- **Suggestions Endpoint**: Test quality score generation, letter grade outputs, and clean code scoring.
- **Full Analysis Endpoint**: Test `/analyze/` response key structure, rule-based provider indicator, execution timing metrics, and multi-language execution.
- **Python AST Analyzer**: Tests in `test_python_ast_analyzer.py` for syntax errors, literal division by zero, out-of-bounds array/string indexing, type mismatch addition, and function parameter zero propagation.

### What Is NOT Tested
- Unmounted routers (`/auth/*`, `/chat/*`, `/share/*`, `/user/*`).
- SQLAlchemy ORM CRUD operations.
- Real HTTP calls to external LLM providers.
- Frontend JavaScript DOM interactions.

---

# PART 15 — DEPLOYMENT / PRODUCTION

### Deployment Configuration (`render.yaml`)
- **Hosting Provider**: Render (Web Service).
- **Environment**: Python runtime.
- **Build Command**: `cd backend && pip install -r requirements.txt`.
- **Start Command**: `cd backend && uvicorn app.main:app --host 0.0.0.0 --port $PORT`.
- **Environment Variables Set**:
  - `RATE_LIMIT_PER_MINUTE`: `30`
  - `LLM_ENABLED`: `false`
- **Static Route Rewrites**: Publishes `frontend/` directory statically and rewrites route `/app` to `/frontend/index.html`.
- **Live Production URL**: `https://codeinsight.onrender.com`

---

# PART 16 — OBSERVABILITY / LOGGING

### Implemented Observability Mechanisms
- **Application Logging**: Python standard `logging` module configured under logger name `"ai_assistant.api"`. Logs request lifecycle (`request_started`, `request_finished`), execution time, status codes, and service warnings.
- **Performance Header**: Custom HTTP middleware injects `X-Process-Time-Ms` response header.
- **Sentry Integration**: Helper `init_error_tracking()` in `error_tracking.py` initializes `sentry_sdk` if `SENTRY_DSN` environment variable is set.
- **Redis Caching**: `cache.py` provides an `AppCache` abstraction that automatically connects to Redis if `REDIS_URL` is set, falling back to in-memory dictionary caching with TTL expiration.

---

# PART 17 — END-TO-END ARCHITECTURE

```
                                 [ USER BROWSER ]
                                        │
                         HTTP POST /GET │ (JSON / Static HTML)
                                        ▼
                            [ RENDER WEB SERVICE / DOCKER ]
                                        │
            ┌───────────────────────────┴───────────────────────────┐
            │                     FASTAPI BACKEND                   │
            │                  (backend/app/main.py)                │
            │                                                       │
            │  ┌─────────────────────────────────────────────────┐  │
            │  │ Middleware: CORS | GZip | RateLimiter (30/min)  │  │
            │  └─────────────────────────────────────────────────┘  │
            │                           │                           │
            │         ┌─────────────────┼─────────────────┐         │
            │         ▼                 ▼                 ▼         │
            │   /explanation        /debugging       /suggestions   │
            │   /analyze            /health          /ping          │
            │         │                 │                 │         │
            │         └─────────────────┼─────────────────┘         │
            │                           │                           │
            │                           ▼                           │
            │         ┌───────────────────────────────────┐         │
            │         │   RULE-BASED ANALYSIS ENGINE      │         │
            │         │ (backend/app/services/            │         │
            │         │         code_assistant.py)        │         │
            │         │  - 40+ Regex Bug Patterns         │         │
            │         │  - Quality Scoring (0-100, A-F)   │         │
            │         │  - Complexity Tiers               │         │
            │         └───────────────────────────────────┘         │
            └───────────────────────────┬───────────────────────────┘
                                        │
                   ┌────────────────────┴────────────────────┐
                   │ (Optional LLM Integration Enabled)      │
                   ▼                                         ▼
        [ OPENAI / GROQ / OLLAMA API ]              [ IN-MEMORY / REDIS CACHE ]
          (via httpx AsyncClient)                    (backend/app/services/cache.py)
```

---

# PART 18 — IMPORTANT FILES

| Rank | File Path | Why It Matters | Key Concept / Takeaway |
|---|---|---|---|
| **1** | `backend/app/services/code_assistant.py` | Core engine file of CodeInsight | Contains 40+ regex bug patterns, language detection, quality scoring formula, complexity estimator, and Python AST parser. |
| **2** | `backend/app/main.py` | Primary backend entry point | Configures FastAPI app, CORS, GZip compression, custom per-IP rate limiter, process time headers, and router inclusions. |
| **3** | `frontend/index.html` | Entire User Interface layout | Standalone HTML5 layout with embedded CSS design tokens, custom components, responsive layout, and theme attributes. |
| **4** | `frontend/script.js` | Frontend controller script | Manages API auto-discovery, fetch operations to `/analyze/`, UI rendering, theme state, file drop, and LocalStorage. |
| **5** | `backend/app/schemas.py` | Pydantic data schemas | Defines Pydantic v2 request/response models (`CodeRequest`, `AnalyzeResponse`, etc.) and code character length validation. |
| **6** | `backend/tests/test_endpoints.py` | Primary API test suite | Contains 22 automated tests verifying health checks, multi-language bug detection, scoring, and full analysis routes. |
| **7** | `backend/app/config.py` | Environment settings manager | Loads environment variables (`.env`) for rate limiting, cache TTL, database URL, and optional LLM integration. |
| **8** | `backend/app/services/llm_analysis.py` | Async LLM client service | Handles OpenAI-compatible API requests via HTTPX and parses structured JSON code analysis responses. |
| **9** | `.github/workflows/ci.yml` | CI automation workflow | GitHub Actions pipeline running Pytest on Python 3.11/3.12 matrix and code quality checks using Ruff. |
| **10**| `render.yaml` | Production deployment blueprint | Configures Render Web Service deployment, environment variables, build/start commands, and static route rewrites. |
| **11**| `Dockerfile` | Production container specification | Single-stage Docker container build on `python:3.12-slim` exposing port 8000. |
| **12**| `backend/app/routers/analyze.py` | Unified analysis endpoint | Exposes `POST /analyze/` uniting explanation, debugging, and suggestions in one call. |
| **13**| `backend/tests/test_python_ast_analyzer.py`| AST engine test suite | Unit tests validating the isolated Python standard library `ast` analyzer (`debug_code`). |
| **14**| `backend/app/services/cache.py` | Dual-backend caching wrapper | Implements fallback caching using Redis if `REDIS_URL` is present, or in-memory dictionary storage. |
| **15**| `backend/app/security.py` | Password & JWT security (Unmounted)| Implements PBKDF2-HMAC-SHA256 password hashing and PyJWT token encode/decode logic. |
| **16**| `backend/app/models.py` | SQLAlchemy ORM models (Unmounted) | Defines database tables (`User`, `QueryHistory`, `FavoriteResult`, `SharedSnippet`). |
| **17**| `backend/app/routers/auth.py` | Authentication router (Unmounted) | Router handling `/auth/signup`, `/auth/login`, and `/auth/me`. |
| **18**| `backend/app/routers/debugging.py` | Standalone debugging endpoint | Exposes `POST /debugging/` for static bug pattern detection. |
| **19**| `backend/app/routers/explanation.py` | Standalone explanation endpoint| Exposes `POST /explanation/` for plain-English code breakdowns. |
| **20**| `backend/app/routers/suggestions.py` | Standalone suggestions endpoint| Exposes `POST /suggestions/` for code quality scoring and tips. |

---

# PART 19 — VERBAL ELEVATOR PITCHES

### 30-Second Explanation
> "CodeInsight is a lightweight, web-based static code analysis platform built with FastAPI and Vanilla JavaScript. It lets developers paste code in Python, JS, TS, Java, or C++ and instantly auto-detects the language, identifies over 40 language-specific bug patterns using regex and AST analysis, and generates a 0–100 quality score with letter grades and refactoring suggestions. It runs completely offline using a rule-based engine, with optional support for OpenAI-compatible LLMs."

### 1-Minute Explanation
> "CodeInsight is a developer assistant workspace designed for rapid code auditing. On the backend, it's powered by Python 3.12 and FastAPI, utilizing a multi-language rule-based engine that scans source code line-by-line for 40+ bug patterns like division by zero, hardcoded secrets, bare exceptions, and innerHTML XSS vulnerabilities. It also estimates code complexity and calculates a weighted quality score from A to F based on missing tests, docstrings, or type safety. On the frontend, it uses zero-build Vanilla HTML5, CSS3, and JavaScript, communicating over HTTP with per-IP rate limiting and Gzip compression. The entire application is containerized with Docker, deployed on Render, and continuously tested via GitHub Actions."

### 3-Minute Explanation
> "CodeInsight is a full-stack static code analysis workspace built to give developers instant feedback on foreign or complex code snippets. 
> 
> Architecturally, the backend is written in FastAPI with Pydantic v2 validation and custom HTTP middleware for process timing and in-memory per-IP rate limiting. The core analysis engine relies on signature-based regex matching for automatic language identification across Python, JavaScript, TypeScript, Java, and C++. Once identified, it runs the code against 40+ rule definitions to spot exact line numbers for security vulnerabilities, syntax traps, and memory leak risks. Concurrently, it calculates a code quality score starting at 100 points, deducting weighted penalties for high, medium, or low priority code smells—like missing docstrings, unhandled I/O, or lack of unit tests—assigning a final letter grade from A to F.
> 
> In addition to the regex engine, the codebase features an isolated AST parser using Python's native `ast` library to trace division-by-zero, index out-of-bounds, and type mismatch risks, as well as an async HTTPX client layer compatible with OpenAI, Groq, or local Ollama endpoints.
> 
> On the frontend, we use zero-dependency Vanilla JS and CSS Custom Properties to deliver a sleek UI with dark/light themes, drag-and-drop file imports, LocalStorage query history, and report export features. The project includes a 22-test Pytest suite, containerization via Docker, and automated CI pipelines with GitHub Actions."

### 5-Minute Technical Interview Explanation
> "CodeInsight is a developer workspace designed to bridge static code analysis and AI-assisted debugging. I can break the technical architecture down into four key areas: the API layer, the analysis engine, the data & security model, and DevOps.
> 
> First, the API layer is built on FastAPI and Uvicorn. When a request hits `/analyze/`, it passes through a custom HTTP middleware chain. This handles CORS, GZip compression, request execution timing via high-resolution performance counters, and an in-memory rate limiter that caps requests at 30 per minute per IP. Requests are validated against Pydantic v2 schemas enforcing payload boundaries up to 50,000 characters.
> 
> Second, the core analysis engine in `code_assistant.py` operates in three phases:
> 1. Language Detection: It uses regex signature scoring across 5 languages—Python, JS, TS, Java, and C++.
> 2. Bug Detection: It executes a rules matrix of over 40 `BugPattern` definitions, scanning line-by-line for issues like `eval()` usage, bare excepts, Java string equality traps, and C++ memory leaks, returning line numbers, code snippets, and fix recommendations.
> 3. Quality Scoring: It runs structural checks against 8 code categories. Deductions are weighted—15 points for high priority issues like missing error handling or tests, 7 for medium, and 3 for low—producing a score from 0 to 100 and a letter grade from A to F.
> Additionally, the project includes an AST parser using Python's standard `ast` module that builds an abstract syntax tree to inspect binary operations, subscript bounds, and function parameter propagation.
> 
> Third, regarding architecture and data: While the active live deployment is stateless and leverages browser LocalStorage for query history and favorites, the backend includes full SQLAlchemy ORM models and JWT authentication services ready for database persistence. It also features an HTTPX-based LLM abstraction layer for plugging in OpenAI-compatible providers like Groq or Ollama.
> 
> Finally, DevOps: The app is containerized using `python:3.12-slim` Docker images, deployed automatically to Render via blueprint manifests, and monitored by GitHub Actions CI running a Pytest suite of 22 tests across Python 3.11 and 3.12 alongside Ruff linting."

---

# PART 20 — GAPS / UNCERTAINTIES ("WHAT I COULD NOT CONFIRM")

### Codebase Audit: Documentation Claims vs. Implementation Reality

| Feature / Claim | Documented Claim (README/Resume) | Actual Implementation Reality | Status |
|---|---|---|---|
| **WebSockets / Collaboration** | "Real-time collaboration using WebSockets" | **0 WebSocket endpoints exist**. No WS client logic in `script.js` or backend dependencies. | **Unfulfilled Claim** |
| **Authentication & User System** | User signup, login, JWT auth, user profiles | Security helpers (`security.py`) and routers (`auth.py`) exist as code files, but are **NOT mounted in `main.py`**. Live API is unauthenticated. | **Unmounted File** |
| **Database Persistence** | User history and saved favorites in SQLite/PostgreSQL | SQLAlchemy models (`models.py`) and routers (`user_data.py`) exist, but are **NOT mounted in `main.py`**. Live app uses browser `localStorage`. | **Unmounted File** |
| **AST Analysis in Live API** | AST-based deep code analysis | Python AST analyzer (`debug_code`) is **only invoked in unit test `test_python_ast_analyzer.py`**. Live endpoints use regex matching (`run_bug_detection`). | **Isolated in Tests** |
| **Docker Compose Multi-Container** | Multi-container setup (API + Database + Redis) | Only single-container `Dockerfile` exists. **No `docker-compose.yml` file present**. | **Missing Config** |
| **LLM Default State** | AI-powered code analysis | Disabled by default (`LLM_ENABLED=false`). Falls back to rule engine. Live `/analyze/` route does not invoke LLM client automatically. | **Optional / Off-by-default** |

---

# FINAL OUTPUT SECTIONS

## A. Complete Architecture Diagram
*(See Part 1, Section 7 and Part 17 for textual diagrams)*

## B. Complete Request / Data Flow
*(See Part 3 for step-by-step trace from UI to backend engine and back)*

## C. Technology → Purpose Table

| Technology | Actual Usage in CodeInsight |
|---|---|
| **Python 3.12** | Core backend language |
| **FastAPI 0.115+** | Web API framework, routing, exception handling, swagger docs |
| **Pydantic v2** | Request/response data validation and serialization schemas |
| **Uvicorn** | ASGI server running the FastAPI application |
| **HTTPX** | Async HTTP client for optional OpenAI-compatible LLM requests |
| **Python `re`** | Regular expression engine powering language detection and 40+ bug pattern checks |
| **Python `ast`** | Abstract Syntax Tree parser used for deep Python static analysis in tests |
| **Pytest** | Automated testing framework for 22 unit/integration tests |
| **Ruff** | High-performance Python linter enforced in CI |
| **Vanilla HTML5/CSS3/JS** | Zero-framework frontend UI with CSS custom properties and LocalStorage state |
| **Docker** | Container image building (`python:3.12-slim`) |
| **Render** | Cloud hosting platform configured via `render.yaml` |
| **GitHub Actions** | CI automation for multi-Python version testing and linting |

## D. Important Files (Top 10 Ranked)
1. `backend/app/services/code_assistant.py` (Core Engine)
2. `backend/app/main.py` (FastAPI App & Middleware)
3. `frontend/index.html` (Frontend UI Layout)
4. `frontend/script.js` (Frontend Controller & Fetch Client)
5. `backend/app/schemas.py` (Pydantic Schemas)
6. `backend/tests/test_endpoints.py` (API Test Suite)
7. `backend/app/config.py` (Configuration Loader)
8. `backend/app/services/llm_analysis.py` (LLM Integration Client)
9. `.github/workflows/ci.yml` (CI Workflow)
10. `render.yaml` (Render Blueprint)

## E. Features → Implementation Mapping

| Feature | Backend Implementation | Frontend Implementation | Database | External Service |
|---|---|---|---|---|
| **Full Code Analysis** | `routers/analyze.py` -> `code_assistant.py:full_analysis` | `script.js:runAnalysis()` -> POST `/analyze/` | None (Stateless) | None |
| **Bug Detection** | `routers/debugging.py` -> `code_assistant.py:run_bug_detection` | `script.js:renderResult()` (Debugging tab) | None | None |
| **Code Explanation** | `routers/explanation.py` -> `code_assistant.py:run_explanation` | `script.js:renderResult()` (Explain tab) | None | None |
| **Quality Scoring** | `routers/suggestions.py` -> `code_assistant.py:run_suggestions` | `script.js:renderResult()` (Improve tab) | None | None |
| **Query History** | Unmounted in `routers/user_data.py` | Active in `script.js` using browser `localStorage` | Unmounted `query_history` table | None |
| **Favorites** | Unmounted in `routers/user_data.py` | Active in `script.js` using browser `localStorage` | Unmounted `favorite_results` table | None |
| **LLM Analysis** | `services/llm_analysis.py` (`LLMAnalysisClient`) | Displayed via engine badge status | None | OpenAI/Groq API (Optional) |
| **Rate Limiting** | `main.py:add_process_time_header` (In-memory dict) | Displays toast/error on 429 response | None | None |

## F. DevOps-Relevant Components
- **Docker**: Single-stage Dockerfile built on `python:3.12-slim`, exposes port 8000, runs Uvicorn.
- **Networking & CORS**: Permissive CORS header configuration in `main.py`. Port 8000 default.
- **CI/CD**: GitHub Actions workflow (`ci.yml`) triggering on push to `main`/`develop`. Pytest matrix on Python 3.11 & 3.12. Ruff linter check on `backend/app`.
- **Deployment**: Render Web Service deployment configured via `render.yaml` with automatic build/start commands and static asset route rewrites.
- **Security & Rate Limiting**: Per-IP sliding window rate limiter (30 req/min) in FastAPI middleware. Input string length cap at 50,000 chars. PBKDF2-HMAC password hashing and PyJWT token generation in `security.py`.

## G. Project Knowledge Map

```
CodeInsight
├── Frontend (Vanilla HTML5 / CSS3 / ES6 JS)
│   ├── index.html (SPA UI Layout & Design System)
│   └── script.js (Fetch API client, State, LocalStorage)
├── Backend (FastAPI / Uvicorn / Python 3.12)
│   ├── Main Application (main.py - App, CORS, GZip, Rate Limiting)
│   ├── Active API Routers (/analyze, /debugging, /explanation, /suggestions)
│   ├── Unmounted API Routers (/auth, /chat, /share, /user)
│   ├── Analysis Engine (code_assistant.py)
│   │   ├── Language Signature Detector (re regex)
│   │   ├── 40+ Bug Pattern Rules Engine
│   │   ├── Quality Scoring Formula (0-100, Grade A-F)
│   │   ├── Code Complexity Estimator
│   │   └── Python AST Parser (debug_code)
│   └── Services Layer
│       ├── LLM Client (llm_analysis.py / ai_provider.py - HTTPX)
│       ├── AppCache (cache.py - Memory / Redis)
│       └── Error Tracking (error_tracking.py - Sentry SDK)
├── DevOps & Infrastructure
│   ├── Docker (Dockerfile - python:3.12-slim)
│   ├── CI/CD (.github/workflows/ci.yml - Pytest matrix + Ruff)
│   └── Deployment (render.yaml - Render Blueprint)
└── Testing
    └── Pytest Suite (tests/ - 22 test cases covering endpoints, rules & AST)
```

## H. INTERVIEW-RELEVANT TECHNICAL DETAILS

### 1. Static Analysis & Regex vs AST Engines
- **Concept**: Pattern matching vs Abstract Syntax Tree traversal.
- **Files**: `backend/app/services/code_assistant.py` (lines 89–260 for regex, 436–518 for AST), `backend/tests/test_python_ast_analyzer.py`.
- **Details**: Understand why regex is fast for cross-language checks, but why AST parsing (`ast.walk`, `ast.BinOp`, `ast.Subscript`) is necessary for detecting semantic issues like division by parameter-propagated zero or list index out of bounds.

### 2. Custom FastAPI Middleware & Rate Limiting
- **Concept**: ASGI request lifecycle, sliding-window rate limiting, and response header injection.
- **Files**: `backend/app/main.py` (lines 55–80), `backend/app/middleware.py`.
- **Details**: Be ready to explain how `add_process_time_header` measures timing with `time.perf_counter()`, extracts the client IP, cleans timestamps older than 60 seconds, and enforces the 30 req/min threshold.

### 3. Pydantic v2 Custom Validators & Serialization
- **Concept**: Data validation and sanitization at the boundary.
- **Files**: `backend/app/schemas.py`.
- **Details**: Understand the `@field_validator("code")` decorator, string stripping, empty check validation, and length bounds enforcement (50,000 chars).

### 4. Quality Scoring Mathematics & Heuristics
- **Concept**: Heuristic quality scoring and weighted penalty accumulation.
- **Files**: `backend/app/services/code_assistant.py` (`run_suggestions`).
- **Details**: Be ready to write down the formula $\text{Score} = 100 - \sum \text{Weights}$ on a whiteboard and explain the priority tiers (`high`: 15, `medium`: 7, `low`: 3).

### 5. Architectural Honesty: Mounted vs Unmounted Components
- **Concept**: Codebase auditing, identifying dead code, and distinguishing production reality from repository artifacts.
- **Files**: `backend/app/main.py` vs `backend/app/routers/` (`auth.py`, `chat.py`, `share.py`, `user_data.py`).
- **Details**: Demonstrating the ability to spot that while auth and DB models exist in `models.py` and `security.py`, they are not mounted in `main.py`, showing true depth in codebase reverse engineering.

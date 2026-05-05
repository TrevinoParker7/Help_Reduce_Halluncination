# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Help Reduce Hallucination** is a Multi-Agent Consensus Engine that addresses AI hallucination by having multiple agents with different perspectives evaluate a query, then using verification and consensus mechanisms to arrive at reliable answers.

### How It Works

1. **Generator Agents** (3 concurrent): Each agent evaluates the query from a unique perspective:
   - **Agent A** (Analytical): Focuses on factual correctness and logical deduction
   - **Agent B** (Skeptical): Looks for edge cases, failures, and counter-arguments
   - **Agent C** (Compliance): Focuses on best practices, security, and safety guidelines

2. **Verifier Agent**: Checks for conflicts, agreements, and flags potential hallucinations across agent responses

3. **Judge Agent**: Determines consensus and selects the most reliable answer based on verifier findings and optional strategy (e.g., "majority")

## Tech Stack

### Frontend
- **Framework**: React 19 + TypeScript
- **Build Tool**: Vite 8
- **Styling**: Tailwind CSS 4
- **Testing**: Playwright (end-to-end tests)
- **Linting**: ESLint with TypeScript support
- **Server**: Runs on `http://127.0.0.1:5173` (Vite dev server)

### Backend
- **Framework**: FastAPI (async Python)
- **LLM Integration**: OpenAI API with Instructor for structured outputs
- **Runtime**: Uvicorn (ASGI)
- **Server**: Runs on `http://localhost:8000`
- **Key Libraries**: pydantic (schemas), instructor (LLM response validation)

## Development Commands

### Frontend

```bash
cd frontend

# Start dev server with hot reload (HMR)
npm run dev

# Build for production
npm run build

# Type-check TypeScript
npx tsc -b

# Lint code
npm run lint

# Preview production build
npm run preview

# Run end-to-end tests (Playwright)
# First, ensure backend is running, then:
npx playwright test

# Run tests in headed mode (see browser)
npx playwright test --headed
```

### Backend

```bash
cd backend

# Create and activate virtual environment (first time)
python -m venv venv
source venv/Scripts/activate  # Windows
# or
source venv/bin/activate      # macOS/Linux

# Install dependencies
pip install -r requirements.txt
# or manually:
pip install fastapi uvicorn pydantic pytest openai instructor

# Run development server (with auto-reload)
python main.py

# Run with explicit uvicorn
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Full Stack Development

To run both services locally:

```bash
# Terminal 1: Backend
cd backend
source venv/Scripts/activate  # or your activation script
python main.py
# Logs: Backend runs on http://localhost:8000

# Terminal 2: Frontend
cd frontend
npm run dev
# Logs: Frontend runs on http://127.0.0.1:5173
```

## Project Structure

```
Help_Reduce_Halluncination/
├── backend/
│   ├── main.py                 # FastAPI app with /api/v1/query and /api/v1/status endpoints
│   ├── orchestrator.py         # Coordinates the multi-agent pipeline (generator → verifier → judge)
│   ├── agents.py              # Generator, Verifier, and Judge agent implementations
│   ├── schemas.py             # Pydantic models (QueryRequest, QueryResponse, AgentResponse, etc.)
│   ├── bridge_poller.py       # Background polling service (optional)
│   ├── debug.py               # Debugging utilities
│   ├── requirements.txt       # Python dependencies
│   └── bridge/                # Cached API responses/prompts for debugging
├── frontend/
│   ├── src/
│   │   ├── App.tsx            # Main UI component (query form, results display)
│   │   ├── main.tsx           # React entry point
│   │   ├── index.css          # Global styles (Tailwind)
│   │   ├── App.css            # App-specific styles
│   │   └── assets/            # Static assets
│   ├── tests/
│   │   └── consensus.spec.ts  # Playwright e2e tests (agreement, conflict, hallucination detection)
│   ├── index.html             # HTML entry point
│   ├── vite.config.ts         # Vite configuration
│   ├── tsconfig.json          # TypeScript config
│   ├── tailwind.config.js     # Tailwind CSS config
│   ├── postcss.config.js      # PostCSS config
│   ├── eslint.config.js       # ESLint config
│   ├── playwright.config.ts   # Playwright test config
│   └── package.json           # Frontend dependencies
└── CLAUDE.md                  # This file
```

## API Endpoints

### Query Submission
**POST** `/api/v1/query`

Request:
```json
{
  "query": "What is the safest way to store API keys?",
  "strategy": "majority"
}
```

Response:
```json
{
  "id": "<job_id>",
  "status": "generating",
  "final_answer": null,
  "justification": null,
  "confidence_score": null,
  "agent_responses": null,
  "verifier_report": null
}
```

### Status Check
**GET** `/api/v1/status/{job_id}`

Returns the same `QueryResponse` structure with updated fields as processing progresses through stages: `generating` → `verifying` → `judging` → `completed` (or `failed: <error>`).

## Key Data Models

### Backend (schemas.py)
- **QueryRequest**: `{query: str, strategy: str}`
- **QueryResponse**: Full job state including agent_responses, verifier_report, final_answer, confidence_score
- **AgentResponse**: `{agent_id, answer, reasoning_trace[], confidence_score}`
- **VerifierReport**: `{agreements[], conflicts[], hallucination_flags[], score_per_agent}`
- **JudgeDecision**: `{final_answer, justification, confidence_score}`

### Frontend (App.tsx)
- Mirrors backend schemas as TypeScript interfaces
- Polls `/api/v1/status/{job_id}` every 1000ms until completion

## Testing

### Frontend Tests (Playwright)
Located in `frontend/tests/consensus.spec.ts`. Tests cover:
- **Agreement**: Factual questions converge on correct answer with high confidence
- **Conflict Detection**: Verifier flags differing agent answers
- **Hallucination Detection**: Bad information is flagged
- **Real-world scenario**: API key security question detects insecure practices

**Run tests**:
```bash
cd frontend
npx playwright test --headed  # See browser during test
npx playwright test           # Headless
npx playwright test --debug   # Debug mode
```

## Important Notes

### CORS Configuration
Backend allows all origins (`allow_origins=["*"]`) for local testing. **Do not deploy with this setting.**

### Environment Variables
- **OPENAI_API_KEY**: Required for backend agent operations. Set via `os.getenv("OPENAI_API_KEY", "dummy")`
- **TEST_MODE**: Optional. Set to enable deterministic mock responses in agents

### Background Job Processing
- Queries are processed asynchronously in `orchestrator.process_query()`
- Jobs stored in-memory: `jobs: Dict[str, QueryResponse]`
- Polling interval in frontend: 1000ms

### Debugging
- `backend/debug.py` contains debugging utilities
- `backend/bridge/` stores cached responses for offline testing
- `frontend/build_error.log` captures build issues
- `frontend/playwright-report/` and `test-results/` contain test reports

## Common Issues

1. **Backend API not responding**: Ensure backend is running on `localhost:8000` and `OPENAI_API_KEY` is set
2. **Tests fail with timeout**: Increase timeout in `playwright.config.ts` if backend is slow
3. **Vite HMR issues**: Clear `node_modules/` and `package-lock.json`, then reinstall
4. **Playwright test fails on query input**: Ensure the input selector matches `input[type="text"]`

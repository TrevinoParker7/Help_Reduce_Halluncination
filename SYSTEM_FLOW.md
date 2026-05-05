# Multi-Agent Consensus System: Detailed Flow Diagram

This document provides a technical walkthrough of how data moves through the **Help Reduce Hallucination** project, from the moment a user clicks "Submit" to the final verified answer.

## 🏗️ High-Level Architectural Flow

```text
====================================================================================================
                                      USER INTERFACE (Frontend)
====================================================================================================
       (1) User Input                       (8) Poll Status                      (11) Display
      [ "Safe API keys?" ]               [ GET /status/123 ]                   [ Verified Answer ]
               |                                ^      |                                ^
               v                                |      v                                |
+--------------+--------------+         +-------+------+-------+        +---------------+-------+
|        Query Form           |         |   Polling Manager    |        |    Results Dashboard  |
|      (React/App.tsx)        |         |   (1000ms interval)  |        |      (Tailwind CSS)   |
+--------------+--------------+         +-------+------+-------+        +---------------+-------+
               |                                |      ^                                |
               | (2) HTTP POST /query           |      | (9) JSON Status                |
               v                                v      |                                |
====================================================================================================
                                      REST API (FastAPI Backend)
====================================================================================================
               |                                |                                       |
     +---------+---------+            +---------+---------+                             |
     |   POST /query     |            |   GET /status     |                             |
     |    (main.py)      |            |    (main.py)      |                             |
     +---------+---------+            +---------+---------+                             |
               |                                ^                                       |
               | (3) Trigger Async Job          | (10) Read from Jobs Dict              |
               v                                |                                       |
+--------------+--------------------------------+---------------------------------------+-------+
|                                  IN-MEMORY JOB STORE (Dict)                                   |
|   { "123": { "status": "generating", "final_answer": null, "agent_responses": [] } }          |
+--------------+--------------------------------------------------------------------------------+
               |
               | (4) Call Orchestrator
               v
====================================================================================================
                                    AGENT PIPELINE (orchestrator.py)
====================================================================================================
               |
      [ PHASE A: GENERATION ] (Parallel Execution)
      --------------------------------------------
               |
      +--------+-----------------------+-----------------------+
      |                               |                       |
      v                               v                       v
+------------+                  +------------+          +------------+
|  Agent A   |                  |  Agent B   |          |  Agent C   |
| Analytical |                  | Skeptical  |          | Compliance |
+-----+------+                  +-----+------+          +-----+------+
      |                               |                       |
      └───────────────┬───────────────┘                       |
                      | (5) Gather Responses                  |
                      v                                       |
      [ PHASE B: VERIFICATION ]                               |
      -----------------------                                 |
                      |                                       |
                      v                                       |
             +-----------------+                              |
             | Verifier Agent  | <----------------------------┘
             | (Cross-checks)  |
             +--------+--------+
                      |
                      | (6) Verifier Report (Agreements/Conflicts)
                      v
      [ PHASE C: JUDGMENT ]
      ---------------------
                      |
             +--------+--------+
             |   Judge Agent   |
             | (Final Decision)|
             +--------+--------+
                      |
                      | (7) Final JSON Decision
                      v
+---------------------+-------------------------------------------------------------------------+
|                                  JOB STATUS UPDATE (Job Store)                                |
|   { "123": { "status": "completed", "final_answer": "...", "confidence": 0.95 } }             |
+-----------------------------------------------------------------------------------------------+
```

## 📂 File & Line-Level Mapping

| Step | Action | File | Relevant Code / Function |
| :--- | :--- | :--- | :--- |
| **1-2** | User Submits Query | `frontend/src/App.tsx` | `handleSubmit()` & `axios.post()` |
| **3** | Job Initialization | `backend/main.py` | `@app.post("/api/v1/query")` |
| **4** | Pipeline Start | `backend/orchestrator.py` | `process_query(job_id, query)` |
| **5** | Parallel Generation | `backend/orchestrator.py` | `asyncio.gather(*[agent.evaluate(...)])` |
| **6** | Conflict Check | `backend/agents.py` | `VerifierAgent.verify()` |
| **7** | Decision Making | `backend/agents.py` | `JudgeAgent.decide()` |
| **8-10**| Result Polling | `frontend/src/App.tsx` | `useEffect()` with `setInterval()` |
| **11** | Dynamic Rendering | `frontend/src/App.tsx` | `return (<div className="results">...)` |

## 🧪 Data Lifecycle (Pydantic Schemas)

1.  **Input:** `QueryRequest` (query, strategy)
2.  **Processing:** `AgentResponse` (answer, justification, confidence)
3.  **Audit:** `VerifierReport` (list of agreements, list of conflicts)
4.  **Output:** `QueryResponse` (The complete history + the final judge decision)

---

### Key Takeaway for Beginners
Notice how the **Frontend** never talks to the **AI Agents** directly. It only talks to the **FastAPI Bridge**, which manages the complex "behind-the-scenes" work. This separation keeps the app fast, secure, and easy to maintain.

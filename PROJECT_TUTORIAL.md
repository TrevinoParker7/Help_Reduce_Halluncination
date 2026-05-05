# The Truth Machine: A Beginner's Guide to Multi-Agent AI Systems

Welcome to the **Help Reduce Hallucination** project tutorial! If you've ever used an AI like ChatGPT and had it give you a confident but completely wrong answer, you've experienced a "hallucination." 

This tutorial will teach you how we built a system that uses **multiple AI agents** to cross-check each other, ensuring the final answer you get is as reliable as possible.

---

## 1. Introduction: The Hallucination Problem

### What is this project?
At its core, this is a **Multi-Agent Consensus Engine**. Instead of asking one AI a question and hoping for the best, this system asks three different "experts," then has a "reviewer" check their work, and finally a "judge" decide on the truth.

### The Problem it Solves
AI models are like very smart, very enthusiastic interns. They want to help so much that they sometimes make things up when they don't know the answer. In high-stakes fields like medicine or legal advice, this is dangerous. This project creates a "safety net" for AI.

### The Courtroom Analogy ⚖️
To understand how this system works, imagine a courtroom:
*   **The Witnesses (Generator Agents):** Three people who saw the event. They each describe what happened from their perspective.
*   **The Cross-Examiner (Verifier Agent):** A lawyer who looks at all three stories, points out where they disagree, and flags things that sound like lies.
*   **The Judge (Judge Agent):** The person who listens to the witnesses and the lawyer, then makes the final, authoritative ruling on what the truth is.

---

## 2. Technology Overview: Our "Tech Stack"

We use a combination of modern tools to build this. Here is why we chose them:

### The Frontend (The Dashboard)
*   **React 19:** A popular library for building the part of the app you see and click on. It's fast and modular.
*   **TypeScript:** Think of this as "JavaScript with seatbelts." It prevents common coding mistakes by being strict about how data is handled.
*   **Tailwind CSS:** A tool that allows us to design beautiful, professional interfaces using simple "labels" instead of complex design code.

### The Backend (The Command Center)
*   **FastAPI (Python):** A high-performance "server" framework. It's the "brain" that receives your question and coordinates the AI agents.
*   **OpenAI API:** The source of our AI's intelligence. We use models like GPT-4 to power our agents.
*   **Pydantic / Instructor:** These are our "Secretaries." They ensure that when an AI speaks, it follows a strict format (like a form) so the computer can understand it easily.

---

## 3. High-Level System Workflow

How does a query move through the system? Follow the 7-step journey:

1.  **Submission:** You type a question (e.g., "What is the safest way to store API keys?") into the **Frontend**.
2.  **API Call:** The Frontend sends this question to the **Backend** server.
3.  **Generation Phase:** The Backend wakes up 3 **Generator Agents** simultaneously. Each one evaluates the query from a different angle (Analytical, Skeptical, and Compliance).
4.  **Verification Phase:** The **Verifier Agent** looks at all three answers. It looks for conflicts (e.g., "Agent A says use a vault, but Agent B says use a text file").
5.  **Judgment Phase:** The **Judge Agent** reviews the Verifier's report and the original answers to pick the single best, safest response.
6.  **Polling:** While this is happening, the Frontend is "polling" (asking every second) the Backend: "Is it done yet? Is it done yet?"
7.  **Completion:** Once the Judge finishes, the Backend sends the final answer to the Frontend, which displays it beautifully for you.

### Simple Flow Diagram
```text
[ YOU ] ───> [ FRONTEND ] ───> [ BACKEND (FASTAPI) ]
                                      │
                               ┌──────┴──────┐
                        [ GENERATORS ] [ VERIFIER ] [ JUDGE ]
                               └──────┬──────┘
[ YOU ] <─── [ FRONTEND ] <───────────┘
```

---

## 4. Step-by-Step Implementation Guide

### Phase 1: Setting up the Backend (The Brain)
1.  **Navigate to the backend folder:** `cd backend`
2.  **Create a Virtual Environment:** Think of this as a "clean room" where we install our Python tools without affecting the rest of your computer.
    *   Command: `python -m venv venv`
    *   Activate: `source venv/Scripts/activate` (Windows) or `source venv/bin/activate` (Mac/Linux)
3.  **Install Tools:** `pip install -r requirements.txt`
4.  **Configuration:** Create a `.env` file and add your `OPENAI_API_KEY`. This is your "passport" to use the AI.
5.  **Run it:** `python main.py`. The server is now waiting for orders!

### Phase 2: Setting up the Frontend (The Face)
1.  **Navigate to the frontend folder:** `cd frontend`
2.  **Install Dependencies:** `npm install`. This downloads all the React and styling tools.
3.  **Run it:** `npm run dev`. This starts a local website you can visit in your browser.

---

## 5. Detailed Code Review

Let's look at the "Star" of the show: the **Orchestrator**.

### The Orchestrator (orchestrator.py)
This is the conductor of the orchestra. It makes sure each agent performs at the right time.

```python
# Simplified look at the logic
async def process_query(job_id, query):
    # 1. Ask 3 agents at once
    agent_results = await asyncio.gather(
        agent_a.evaluate(query), 
        agent_b.evaluate(query), 
        agent_c.evaluate(query)
    )
    
    # 2. Hand results to the Verifier
    report = await verifier.check(agent_results)
    
    # 3. Let the Judge decide
    final_decision = await judge.decide(agent_results, report)
    
    # 4. Save the answer
    save_job_result(job_id, final_decision)
```

**Why this way?**
*   **`asyncio.gather`**: This is a superpower. It allows us to ask all 3 agents their opinions at the same time, rather than waiting for one after the other. This makes the app 3x faster!
*   **Separation of Concerns**: We don't ask the Judge to verify. By giving each agent a specific job, they perform better and are easier to "debug" (fix) if they make a mistake.

---

## 6. Key Concepts to Remember

1.  **Client-Server Relationship:** The **Client** (Browser) asks, and the **Server** (FastAPI) answers. They communicate over a "bridge" called an API.
2.  **State Management:** In the Frontend, we use "State" to remember if the AI is still thinking. If `status === "generating"`, we show a loading spinner.
3.  **Prompt Engineering:** Each agent has "System Instructions" that tell it how to act. For example, Agent B is told to be "Skeptical." This is how we get different perspectives.
4.  **Asynchronous Programming:** Computers can do many things at once. We use "async" code so the server doesn't freeze while waiting for the AI to respond.

---

## 7. Self-Review & Improvement Suggestions

This system is a great start, but here is how we would make it "Production Ready":

1.  **Persistent Storage (Database):** Currently, if the server restarts, all your previous questions are deleted.
    *   *Solution:* Use a database like **PostgreSQL** or **MongoDB** to save every query forever.
2.  **Streaming Responses:** Right now, you have to wait for the whole answer to finish before seeing anything.
    *   *Solution:* Implement **Server-Sent Events (SSE)** so the text appears letter-by-letter as it's generated.
3.  **User Accounts:** 
    *   *Solution:* Add a login system so different users can have private query histories.
4.  **Agent Variety:** 
    *   *Solution:* Use different AI models (like Claude or Llama) for different agents. This creates "Diversity of Thought," making hallucinations even easier to spot.
5.  **Cost Monitoring:** 
    *   *Solution:* Add a dashboard to see how much money each query costs in OpenAI credits.

---

### Final Thought
You've just explored a system that mimics the way humans solve complex problems: by talking, debating, and reaching a consensus. You are now on your way to becoming a Full-Stack AI Engineer!

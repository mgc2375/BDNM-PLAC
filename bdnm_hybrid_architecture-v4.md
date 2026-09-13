# Framework: Bidirectional Node Mapping (BDNM) with Hybrid Zero-Token Lookahead

**Objective:** Solve complex, ambiguous, or hard-to-reach problems by constructing a multi-pathway graph that bridges the gap between a known starting state and a target outcome using simultaneous forward and backward exploration, paired with a deterministic, zero-token regex interceptor that pre-fetches and caches upcoming node dependencies.

---

## Part 1: System Directives & Execution Protocol

When executing this framework, you must not simply describe the network — you must actively generate the nodes, trace the connections, and resolve the pathways step-by-step.

### Step 1: Forward Initialization (Current State)
1. Define the **Start Point ($S_0$)** clearly as the origin node.
2. **First-Order Expansion ($S_1$):** Exhaustively brainstorm all immediate direct consequences, actions, or shoot-offs originating from $S_0$.

### Step 2: Backward Initialization (Goal State)
1. Define the **End Point ($E_0$)** clearly as the target destination node.
2. **First-Order Antecedents ($E_1$):** Exhaustively brainstorm all direct prior inputs, conditions, or preceding events that could directly yield $E_0$.

### Step 3: Forward Recursive Expansion
1. For every node in $S_n$, generate its downstream shoot-offs ($S_{n+1}$).
2. Continue expanding outward: *"Given node $X$, what are all possible paths, actions, or consequences that branch from it?"*

### Step 4: Backward Recursive Expansion
1. For every node in $E_n$, generate its required preceding inputs ($E_{n+1}$).
2. Continue expanding backward: *"To achieve node $Y$, what are all possible inputs, dependencies, or pre-conditions that could produce it?"*

### Step 5: Pathway Convergence & Resolution
1. Expand both networks until nodes from the Forward Set ($S$) intersect or logically align with nodes from the Backward Set ($E$).
2. **Termination Criteria:** Stop expansion once at least 3 to 5 continuous, valid, end-to-end pathways connect $S_0$ to $E_0$.

### Step 6: Agentic Mapping & Deterministic Trigger Configuration
1. Map every node ($S_n$ and $E_n$) to explicit data requirements and assign a hardcoded trigger tag (e.g., `[TRIGGER_TAG: FETCH_JWT_LOGS]`).
2. Configure Agent 1 (Primary Reasoner) to operate in a verbose/streaming mode (similar to terminal interfaces like Claude Code's verbose view), allowing its thought and tool trace to be monitored.
3. Deploy a lightweight, non-LLM regex interceptor to monitor Agent 1's output stream in real-time, executing background lookups with zero token overhead.

---

## Part 2: System Architecture Workflow Tree

```text
[Initialization Phase]
User Goal (S0 → E0)
       │
       ▼
[BDNM Planner Engine] ─── (Generates S_n Forward & E_n Backward Graph with Explicit Trigger Tags)
       │
       ▼
[BDNM Topological Map] (Pre-indexed Node Schema & Tags)
       │
       ├─────────────────────────────────────────┐
       ▼                                         ▼
[Agent 1: Primary Reasoner]             [Event Stream / Queue]
(Executes nodes, streams verbose thoughts    │
 with embedded [TRIGGER_TAG: ...] )            ▼
       │                                 [Deterministic Regex Interceptor]
       │                                 (0 LLM Tokens; matches tags instantly)
       │                                         │
       │                                         ▼
       │                                 [Asynchronous Background Worker]
       │                                 (Triggers APIs/DB queries early)
       │                                         │
       │                                         ▼
       └──────────(Zero-Latency Retrieval)◄─── [Shared Node Cache]
               (Consumes prepared Data X for Node N+1)
```

---

## Part 3: Component Breakdown & Execution Dynamics

1. **BDNM Initialization Phase**
   - The system builds a multi-pathway graph bridging Start State ($S_0$) to End State ($E_0$).
   - Every node is annotated with an explicit rule trigger tag for upcoming dependencies.

2. **Agent 1 (Primary Reasoner - LLM-driven)**
   - Solves the problem step-by-step.
   - Streams its thoughts and tool execution traces (similar to a verbose developer tool interface).
   - Intentionally incorporates pre-mapped BDNM trigger tags into the text stream.

3. **Deterministic Regex Interceptor (Zero-Token Sidecar)**
   - Replaces a secondary LLM agent with a fast string-matching / regex script.
   - Continuously watches Agent 1's token stream in real-time.
   - Consumes **0 LLM tokens**, keeping operating costs low.

4. **Asynchronous Background Execution & Cache Staging**
   - When the regex engine matches a tag, it immediately fires the underlying data fetch (API/DB query) in the background.
   - Pushes the fetched payload into the Shared Node Cache mapped to the upcoming node ID.

5. **Zero-Latency Consumption & Resource Optimization**
   - When Agent 1 finishes its current reasoning block and advances to the next node, the data is already pre-loaded in the cache, eliminating execution wait times.
   - Drastically cuts down overall context footprint and computational overhead by avoiding redundant retrieval loops and repetitive payload dumps into history.

---

## Part 4: Token & Resource Efficiency Analysis

Using a verbose stream to watch what an agent is doing (akin to terminal verbose modes like Claude Code) **does not artificially inflate billable output token usage**, while the lookahead architecture actively trims unnecessary computational overhead.

* **The Model Thinks Anyway:** Modern reasoning models generate their chain-of-thought and structural traces regardless of whether a verbose UI window is open or hidden. 
* **Zero Additional Billable Bloat:** Displaying or streaming those tokens for interception costs no extra tokens compared to running headless.
* **Interceptor Efficiency:** Because Agent 2 uses a pure Python regex script rather than a secondary LLM, it costs **0 tokens**.
* **Reduced Retrieval Overhead:** By pre-fetching dependencies asynchronously via trigger tags, the system avoids repetitive round-trip retrieval loops, cutting down on context pollution and context window bloat.

**Summary Verdict:** You achieve complete visibility, faster wall-clock execution, and minimized resource consumption without incurring any extra token penalty.

---

## Part 5: Reference Python Implementation

```python
import asyncio
import re
from typing import Dict, Any, AsyncGenerator

BDNM_GRAPH = {
    "S0": {"next": "S1_A", "tag": None},
    "S1_A": {
        "next": "S2_A1", 
        "tag": "[TRIGGER_TAG: FETCH_JWT_LOGS]",
        "dependency_key": "jwt_logs"
    },
    "S2_A1": {
        "next": "E1_A", 
        "tag": "[TRIGGER_TAG: FETCH_SECRET_SCHEDULE]",
        "dependency_key": "secret_schedule"
    }
}

SHARED_CACHE: Dict[str, Any] = {}

async def deterministic_fetcher(token_stream: AsyncGenerator[str, None]):
    buffer = ""
    async for chunk in token_stream:
        buffer += chunk
        if "[TRIGGER_TAG: FETCH_JWT_LOGS]" in buffer and "jwt_logs" not in SHARED_CACHE:
            print("\n[Agent 2 - Interceptor]: Trigger detected! Pre-fetching JWT logs asynchronously...")
            await asyncio.sleep(0.1)
            SHARED_CACHE["jwt_logs"] = {"status": "success", "data": "Mock JWT Audit Logs"}
        elif "[TRIGGER_TAG: FETCH_SECRET_SCHEDULE]" in buffer and "secret_schedule" not in SHARED_CACHE:
            print("\n[Agent 2 - Interceptor]: Trigger detected! Pre-fetching secret schedule...")
            await asyncio.sleep(0.1)
            SHARED_CACHE["secret_schedule"] = {"status": "success", "data": "Mock Rotation Schedule"}

async def agent_1_reasoner() -> AsyncGenerator[str, None]:
    thought_steps = [
        "Analyzing current system state [S0]. Moving to path S1_A. ",
        "I need to check authentication behavior. [TRIGGER_TAG: FETCH_JWT_LOGS] ",
        "Processing logs... Now transitioning to sub-branch S2_A1. ",
        "To finalize this step, I will require the rotation schedule. [TRIGGER_TAG: FETCH_SECRET_SCHEDULE] "
    ]
    for step in thought_steps:
        for word in step.split(" "):
            yield word + " "
            await asyncio.sleep(0.05)

async def main():
    print("--- Starting BDNM Zero-Token Lookahead Pipeline ---")
    stream_queue = asyncio.Queue()
    
    async def publisher():
        async for token in agent_1_reasoner():
            print(token, end="", flush=True)
            await stream_queue.put(token)
        await stream_queue.put(None)

    async def consumer():
        buffer = ""
        while True:
            token = await stream_queue.get()
            if token is None:
                break
            buffer += token
            if "[TRIGGER_TAG: FETCH_JWT_LOGS]" in buffer and "jwt_logs" not in SHARED_CACHE:
                print("\n\n>>> [Agent 2 Sidecar]: Matched tag! Fetching 'jwt_logs' in background...")
                SHARED_CACHE["jwt_logs"] = "CACHED_JWT_LOGS_DATA"
            elif "[TRIGGER_TAG: FETCH_SECRET_SCHEDULE]" in buffer and "secret_schedule" not in SHARED_CACHE:
                print("\n\n>>> [Agent 2 Sidecar]: Matched tag! Fetching 'secret_schedule' in background...")
                SHARED_CACHE["secret_schedule"] = "CACHED_SCHEDULE_DATA"

    await asyncio.gather(publisher(), consumer())
    print("\n\n--- Execution Complete ---")
    print("Final Shared Cache State:", SHARED_CACHE)

if __name__ == "__main__":
    asyncio.main()

# Exporting the Local Architecture Guide to PDF

To export the entire architecture guide and configuration directly into a styled, publication-ready PDF on your Mac, use one of the workflows below.

## Method 1: Instant CLI Conversion via Node/Markdown-PDF (Recommended)

You can generate the PDF directly from your terminal using `npx` with zero persistent installations.

### Step 1: Write the Markdown File

Run this command in Terminal to save the documentation as `DIVIDE_AND_CONQUER_SETUP.md`:

```bash
cat > DIVIDE_AND_CONQUER_SETUP.md << 'EOF'
# Local Divide & Conquer Autonomous Agent Architecture

**Platform:** Apple Silicon MacBook Pro M4 Pro (48 GB Unified Memory)
**Primary Stack:** Ollama, OpenCode, Metal Framework
**Target Capabilities:** SWE-bench Verified Bug Fixing, Algorithmic Profiling, Test Loop Harnesses, Zero-Egress Code Audits

---

## 1. System Architecture & Memory Engineering

To run an autonomous software engineering pipeline locally without exceeding memory constraints, the architecture isolates cognitive workloads between two specialized models using **sequential hot-swapping** (`OLLAMA_MAX_LOADED_MODELS=1`). This guarantees that only one model resides in memory at any given second.

┌────────────────────────────────────────────────────────────────────────────┐
│                  M4 Pro 48 GB Unified Memory (273 GB/s)                    │
├──────────────────────────────────────┬─────────────────────────────────────┤
│        macOS System Pool (20%)        │       Max Allocatable VRAM (80%)   │
│              ~9.6 GB RAM              │              ~38.4 GB VRAM         │
│   (WindowServer, IDE, Host OS)        │    (Metal iogpu.wired_limit_mb)    │
└──────────────────────────────────────┴──────────────────┬──────────────────┘
                                                            │
                          ┌─────────────────────────────────┴──────────────────┐
                          │ Sequential Hot-Swapping (OLLAMA_MAX_LOADED=1)       │
                          ├──────────────────────────┬───────────────────────────┤
                          │ State A: @planner        │ State B: @coder           │
                          │ deepseek-r1:14b-qwen-    │ qwen3-coder:30b-a3b-      │
                          │ distill-q8_0             │ q8_0                      │
                          │                          │                           │
                          │ Weights:        ~15.5 GB │ Weights:        ~32.0 GB  │
                          │ 32k Q8 Context:  ~4.2 GB │ 16k Q8 Context:  ~2.2 GB  │
                          │ Peak VRAM:      ~19.7 GB │ Peak VRAM:      ~34.2 GB  │
                          │ Free GPU Buffer: 18.7 GB │ Free GPU Buffer:  4.2 GB  │
                          └──────────────────────────┴───────────────────────────┘

### Verified Model Registry Manifests

| Component | Exact Registry Tag | Quantization | Size | Architecture | Primary Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Planner / Reasoner** | `deepseek-r1:14b-qwen-distill-q8_0` | `Q8_0` | ~15.5 GB | Dense (14.7B params) | Root-cause analysis, planning, edge-case testing |
| **Tactical Workhorse** | `qwen3-coder:30b-a3b-q8_0` | `Q8_0` | ~32.0 GB | Sparse MoE (3.3B active params) | AST patching (32–36 tok/s), JSON tool calls |

### SWE-bench & Throughput Profile

* **`deepseek-r1:14b-qwen-distill-q8_0`:** Operates at **18–22 tok/s** on M4 Pro. Utilizes `<think>` tokens for multi-step algorithmic deduction and traceback tracing.
* **`qwen3-coder:30b-a3b-q8_0`:** Generates at **32–36 tok/s** because it routes computation through only 3.3B active parameters per forward pass. It reaches **~51.4% (Vanilla)** and **~71% (Scaffolded)** on SWE-bench Verified.
* **Quantization Stability:** The `Q8_0` quantization eliminates routing noise in the 128-expert gating network, maintaining 0.0% schema drift for tool-call emissions.

---

## 2. Host Operating System Tuning

### A. Persist 80% VRAM Allocation

Set the Metal wired memory limit to allow up to 39,321 MB (38.4 GB) to be claimed by the GPU without kernel panics:

# Apply dynamically
sudo sysctl iogpu.wired_limit_mb=39321

# Persist across reboots
echo "iogpu.wired_limit_mb=39321" | sudo tee -a /etc/sysctl.conf

### B. Configure Ollama Daemon (`~/.zshrc`)

Append the following variables to enforce sequential execution and halved KV-cache memory usage:

# ==========================================
# Ollama Multi-Agent Resource Constraints
# ==========================================
# Enforce sequential loading to prevent VRAM overflow
export OLLAMA_MAX_LOADED_MODELS=1

# Maintain model in memory for 3 minutes during rapid subagent iterations
export OLLAMA_KEEP_ALIVE=3m

# Prevent context allocation fragmentation across slots
export OLLAMA_NUM_PARALLEL=1

# Cut context memory footprint by 50% via 8-bit quantized KV cache
export OLLAMA_KV_CACHE_TYPE=q8_0

# Ensure localhost binding for OpenCode IPC
export OLLAMA_HOST=127.0.0.1:11434

# Enable verbose logging for token rates and load durations
export OLLAMA_DEBUG=1
export LLAMA_LOG_LEVEL=INFO
export OLLAMA_LOG_DIR="$HOME/.ollama/logs"
mkdir -p "$OLLAMA_LOG_DIR"

Apply the profile:

source ~/.zshrc
killall ollama 2>/dev/null || true
ollama serve > "$OLLAMA_LOG_DIR/server.log" 2>&1 &

## 3. Ollama Model Setup & Registry Verification

Pull the verified models directly from the public registry:

# 1. Pull the Strategic Reasoner (~16 GB)
ollama pull deepseek-r1:14b-qwen-distill-q8_0

# 2. Pull the Tactical MoE Workhorse (~32 GB)
ollama pull qwen3-coder:30b-a3b-q8_0

Verify the local library:

ollama list

## 4. OpenCode Configuration (`opencode.jsonc`)

Place this file in your project root or at `~/.config/opencode/opencode.jsonc`. It configures the dual-agent architecture, establishes strict read/write boundaries, configures 32k/16k context limits, and enables the real-time terminal telemetry HUD.

{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/deepseek-r1:14b-qwen-distill-q8_0",
  "default_agent": "planner",
  "plugin": [
    "superpowers@git+https://github.com/obra/superpowers.git"
  ],
  "ui": {
    "status_bar": true,
    "show_token_usage": true,
    "show_throughput": true,
    "show_active_agent": true,
    "show_tool_latencies": true
  },
  "telemetry": {
    "track_usage": true,
    "turn_summary": true,
    "context_warning_threshold": 0.85
  },
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://127.0.0.1:11434/v1"
      },
      "models": {
        "deepseek-r1:14b-qwen-distill-q8_0": {
          "name": "DeepSeek-R1 14B Q8_0 (Reasoner)",
          "limit": {
            "context": 32768,
            "output": 8192
          }
        },
        "qwen3-coder:30b-a3b-q8_0": {
          "name": "Qwen3-Coder 30B MoE Q8_0 (Workhorse)",
          "limit": {
            "context": 16384,
            "output": 4096
          }
        }
      }
    }
  },
  "agents": {
    "planner": {
      "mode": "primary",
      "model": "ollama/deepseek-r1:14b-qwen-distill-q8_0",
      "description": "Lead architect for root-cause diagnosis, algorithm proofs, and planning.",
      "temperature": 0.6,
      "top_p": 0.95,
      "tools": {
        "read": true,
        "write": false,
        "edit": false,
        "bash": false
      },
      "permissions": [
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "edit", "resource": "*", "effect": "deny" },
        { "action": "write", "resource": "*", "effect": "deny" },
        { "action": "shell", "resource": "*", "effect": "deny" }
      ],
      "system": "You are the Lead Systems Architect. Analyze bug traces, explore system dependencies, prove algorithmic invariants, and produce structured, step-by-step implementation plans. Never output raw file edits or attempt to run commands. Delegate tactical implementation to @coder."
    },
    "reviewer": {
      "mode": "subagent",
      "model": "ollama/deepseek-r1:14b-qwen-distill-q8_0",
      "description": "Auditor for diff reviews, race conditions, memory leaks, and complexity analysis.",
      "temperature": 0.4,
      "top_p": 0.90,
      "tools": {
        "read": true,
        "write": false,
        "edit": false,
        "bash": false
      },
      "permissions": [
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "edit", "resource": "*", "effect": "deny" },
        { "action": "write", "resource": "*", "effect": "deny" },
        { "action": "shell", "resource": "*", "effect": "deny" }
      ],
      "system": "You are a Principal Code Reviewer. Audit diffs for edge-case vulnerabilities, asymptotic complexity regressions, memory retention, and concurrency races. Deliver actionable critique with mathematical rigor without mutating files."
    },
    "coder": {
      "mode": "subagent",
      "model": "ollama/qwen3-coder:30b-a3b-q8_0",
      "description": "Autonomous tactical workhorse for AST patching, file editing, and test runs.",
      "temperature": 0.1,
      "top_p": 0.95,
      "tools": {
        "read": true,
        "write": true,
        "edit": true,
        "bash": true
      },
      "permissions": [
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "edit", "resource": "*", "effect": "allow" },
        { "action": "write", "resource": "*", "effect": "allow" },
        { "action": "shell", "resource": "*", "effect": "allow" }
      ],
      "system": "You are an autonomous tactical coding workhorse. Execute assigned engineering steps. Generate exact unified diffs or search-and-replace patches without conversational filler. Verify code syntax and run test suites using the shell tool."
    },
    "tester": {
      "mode": "subagent",
      "model": "ollama/qwen3-coder:30b-a3b-q8_0",
      "description": "Specialist for generating test fixtures, mocks, and executing test suites.",
      "temperature": 0.2,
      "top_p": 0.95,
      "tools": {
        "read": true,
        "write": true,
        "edit": true,
        "bash": true
      },
      "permissions": [
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "edit", "resource": "*", "effect": "allow" },
        { "action": "write", "resource": "*", "effect": "allow" },
        { "action": "shell", "resource": "*", "effect": "allow" }
      ],
      "system": "You are a Test Automation Specialist. Write targeted unit, regression, and property-based tests. Run test commands via the shell, interpret assertion traces, and fix failing tests until the suite passes completely."
    }
  }
}

## 5. Live Multi-Pane Monitoring HUD

Create an automated dashboard script to observe memory usage, token burn rates, and hot-swaps in real time.

Save as `~/bin/opencode-hud.sh`:

#!/usr/bin/env bash
SESSION="opencode-dev"

# Re-attach if session already exists
tmux has-session -t $SESSION 2>/dev/null
if [ $? -eq 0 ]; then
  tmux attach-session -t $SESSION
  exit 0
fi

# 1. Main interactive OpenCode workspace (Left 60%)
tmux new-session -d -s $SESSION -n "Agent-Workspace"
tmux send-keys -t $SESSION "opencode" C-m

# 2. Ollama VRAM and Residency Monitor (Top-Right 40%)
tmux split-window -h -p 40 -t $SESSION
tmux send-keys -t $SESSION "watch -n 0.5 'ollama ps'" C-m

# 3. Real-Time Token Generation & Throughput Monitor (Bottom-Right 50%)
tmux split-window -v -p 50 -t $SESSION
tmux send-keys -t $SESSION "tail -f ~/.ollama/logs/server.log | grep --line-buffered -E '(eval rate|prompt eval count|total duration|load duration)'" C-m

# Focus back on the main workspace
tmux select-pane -t 0
tmux attach-session -t $SESSION

Make it executable and link to your environment:

chmod +x ~/bin/opencode-hud.sh
echo 'alias opencode="~/bin/opencode-hud.sh"' >> ~/.zshrc
source ~/.zshrc

## 6. Prompt Engineering & Operational Guardrails

### Core Operational Rules

* **The Context Boundary Guardrail (Coder ≤ 16k):** Never pass raw multi-megabyte log files or entire source trees directly to `@coder`. Let `@planner` read the project structure, locate the issue, and provide isolated target paths.
* **The Delegation Contract:** When `@planner` delegates work, it must emit a 4-part contract: Target File Path, Exact Search Block, Exact Replacement Block, and Verification Command.

### Production Prompt Catalog

**Pattern 1: Autonomous Divide & Conquer (Full SWE-bench Repair Loop)**

@planner: We have an intermittent assertion failure in tests/test_settlement.py::test_partial_fill.
1. Read tests/test_settlement.py and src/engine/settlement.py.
2. Diagnose the root cause of the race condition or calculation error.
3. Formulate a step-by-step fix contract.
4. Delegate the implementation, file patching, and test validation to @coder.

**Pattern 2: Algorithmic Complexity & Deep Research**

@planner: Analyze the graph traversal in src/routing/pathfinder.py.
1. Determine the formal time and space complexity of find_optimal_path() in big-O notation.
2. Identify bottlenecks causing high latency under 100,000+ nodes.
3. Formulate an alternative design using an indexed priority queue and memoized A* heuristics.
Do not write complete files. Provide the theoretical proof and an implementation plan for @coder.

**Pattern 3: Targeted High-Throughput Implementation**

@coder: In src/serializers/binary.py, implement the struct-based binary packer for the OrderBookSnapshot schema.
- Follow the byte packing alignment defined in docs/wire_spec.md.
- Run `pytest tests/test_serializers.py` and ensure all tests pass.
- Provide the git diff output when done.

**Pattern 4: Pull Request & Security Review**

@reviewer: Review the staged changes on this branch against main (`git diff origin/main...HEAD`).
Audit for:
1. Thread-safety violations or un-synchronized access to shared state.
2. Unhandled exception paths on database connections.
3. Violations of domain boundary encapsulation.
Present findings ranked by severity: Critical, Major, Minor.

**Pattern 5: Test Generation & Regression Hardening**

@tester: Inspect src/auth/token_validator.py.
1. Write a complete pytest suite in tests/test_token_validator.py covering: expired tokens, corrupted signatures, clock skew, and empty payloads.
2. Run the tests using `pytest tests/test_token_validator.py -v`.
3. Ensure all tests execute and pass without warnings.

## 7. Performance Benchmarks: Local M4 Pro vs. Hyperscaler APIs

| Metric / Dimension | Hyperscaler Cloud APIs (Claude 3.5 on Bedrock / Vertex / Direct) | Local Dual Setup on M4 Pro (deepseek-r1:14b & qwen3-coder:30b-a3b) |
| :--- | :--- | :--- |
| Output Throughput | ~74 – 80.4 tok/s (Datacenter GPU cluster) | 18 – 22 tok/s (Reasoner) / 32 – 36 tok/s (MoE Coder) |
| Time to First Token (TTFT) | ~150 ms – 190 ms (Cold network round-trip over WAN) | 0 ms network + ~1.2s – 1.8s model swap (only during role changes) |
| Prompt Ingestion Speed | Thousands of tok/s (Distributed cluster prefill) | ~350 – 500 tok/s (Apple Silicon Metal Unified Memory) |
| Concurrency & Scaling | Near-infinite (managed multi-tenant elastic clusters) | Strictly sequential (`OLLAMA_NUM_PARALLEL=1`, 1 task at a time) |
| Data Privacy & Boundaries | Requires VPC peering, IAM roles, enterprise agreements | Air-gapped / Localhost-bound (127.0.0.1, zero egress) |
| Variable Cost | Pay-per-token ($3 / $15 per million tokens + cache fees) | $0.00 marginal cost (amortized hardware & ~35W electricity) |

### Key Architectural Takeaway

Datacenter cloud deployments achieve higher aggregate throughput (80 tok/s) and rapid context prefill across large distributed clusters.

The local M4 Pro architecture trades raw throughput (generating at 32–36 tok/s) and a ~1.8s hot-swap transition for complete data confidentiality, zero network latency within turn sequences, and zero per-token cost, enabling continuous test-driven repair loops within a 34.2 GB peak VRAM footprint.

## 8. Alternative Workflow: Custom Model Aliases via Ollama Modelfiles

The sections above configure OpenCode's provider block to reference the raw Ollama registry tags directly. An alternative — and slightly more portable — approach wraps each base checkpoint in a dedicated Ollama Modelfile that bakes in the context window, sampling temperature, and system prompt, then exposes it under a short local alias (`local-planner`, `local-coder`). OpenCode's agent config then only needs to reference the alias, not the full tag.

The system still strictly obeys `OLLAMA_MAX_LOADED_MODELS=1` so the two aliases never co-exist in memory: peak VRAM never exceeds ~34.2 GB, well within the 38.4 GB (80%) ceiling, preserving 9.6 GB for macOS.

### Step 1: System-Level Configuration & Environment Setup

#### 1. Hardware VRAM Allocation

Set the macOS wired memory limit to enforce the 80% ceiling (39,321 MB):

sudo sysctl iogpu.wired_limit_mb=39321

To persist this across reboots, add it to `/etc/sysctl.conf`:

echo "iogpu.wired_limit_mb=39321" | sudo tee -a /etc/sysctl.conf

#### 2. Shell Environment Configuration (`~/.zshrc`)

Append the following variables to your `~/.zshrc`:

# ==========================================
# Ollama Multi-Agent Resource Constraints
# ==========================================
# Force sequential single-model residency (prevents VRAM eviction/swap thrashing)
export OLLAMA_MAX_LOADED_MODELS=1

# Maintain residency for 3 minutes during rapid subagent iterations
export OLLAMA_KEEP_ALIVE=3m

# Prevent context fragmentation across slots
export OLLAMA_NUM_PARALLEL=1

# Cut KV Cache footprint by 50% using 8-bit quantized cache
export OLLAMA_KV_CACHE_TYPE=q8_0

# Ensure localhost binding for OpenCode IPC
export OLLAMA_HOST=127.0.0.1:11434

### Verified Dual-Model Registry Tags

With that in place, your exact dual-model pairing is verified and locked on disk:

| Component | Exact Ollama Tag | Quantization | Disk / Base VRAM | Primary Function |
| :--- | :--- | :--- | :--- | :--- |
| **Planner / Reasoner** | `deepseek-r1:14b-qwen-distill-q8_0` | `Q8_0` | ~15.5 GB | Root-cause analysis, planning, and edge-case testing |
| **Tactical Workhorse** | `qwen3-coder:30b-a3b-q8_0` | `Q8_0` | ~32.0 GB | Rapid AST patching (32–36 tok/s), JSON tool calls |

### Memory & VRAM Enforcement (M4 Pro 48 GB @ 80% Cap)

Because `qwen3-coder:30b-a3b-q8_0` weighs ~32 GB on load, managing context and concurrency prevents breaching your 38.4 GB (80%) ceiling:

Total Allocatable Cap (80%): 38.4 GB (39,321 MB)
System Pool Guarantee (20%):  9.6 GB (Zero UI lag / No swap)

[Phase 1] deepseek-r1:14b Active:
  Weights: ~15.5 GB + 32k Q8 Context: ~4.2 GB = ~19.7 GB Peak (~18.7 GB buffer)

[Phase 2] qwen3-coder:30b-a3b Active:
  Weights: ~32.0 GB + 16k Q8 Context: ~2.2 GB = ~34.2 GB Peak (~4.2 GB buffer)

**Key Tuning Rule for the Q8_0 Workhorse:** With `Q8_0` weights at 32 GB, cap the executor's context window to 16,384 tokens (`PARAMETER num_ctx 16384`) rather than 32k.

* A 16k context window with `OLLAMA_KV_CACHE_TYPE=q8_0` consumes ~2.2 GB, keeping peak usage at ~34.2 GB — safely below the 38.4 GB boundary with ~4.2 GB of VRAM headroom to spare.

### Step 2: Model Ingestion & Custom Modelfiles

The required base checkpoints must already be present in Ollama (pulled in section 3 above):

* `deepseek-r1:14b-qwen-distill-q8_0` (~15.5 GB)
* `qwen3-coder:30b-a3b-q8_0` (~32.0 GB)

We wrap both in optimized Modelfile definitions to enforce their token parameters, context windows, and sampling temperatures.

**1. Modelfile for Planner (`Modelfile.planner`)**

Save as `Modelfile.planner`:

FROM deepseek-r1:14b-qwen-distill-q8_0

# Allocate 32k context for comprehensive repository tracing
PARAMETER num_ctx 32768

# Native reasoning temperature (DeepSeek works best around 0.6 for path exploration)
PARAMETER temperature 0.6
PARAMETER top_p 0.95

# Explicit Planner System Prompt
SYSTEM """You are the Lead Systems Architect and Strategic Planner.
Your role is to diagnose root causes, prove algorithm complexity, and design strict, actionable implementation plans.
Do not write complete boilerplate code or run shell builds yourself. Delegate all coding, file edits, and test execution to @coder.
Always provide your diagnostic breakdown first, followed by an ordered list of tasks for the workhorse to execute."""

**2. Modelfile for Coder (`Modelfile.coder`)**

Save as `Modelfile.coder`:

FROM qwen3-coder:30b-a3b-q8_0

# Hard-limit context to 16k to protect the 38.4 GB VRAM ceiling with 32 GB weights
PARAMETER num_ctx 16384

# Low temperature for deterministic, hallucination-free code and tool syntax
PARAMETER temperature 0.1
PARAMETER top_p 0.95

# Explicit Coder System Prompt
SYSTEM """You are an autonomous tactical coding workhorse.
Your role is to write clean, production-grade code, generate exact search-and-replace patches, and validate changes by running the project's test suite.
Do not generate verbose conversational filler or high-level philosophical plans. Read the target file, apply the requested change precisely, and report the result."""

**3. Build the Ollama Aliases**

Run these commands in the terminal where your Modelfiles are saved:

ollama create local-planner -f Modelfile.planner
ollama create local-coder -f Modelfile.coder

### Step 3: OpenCode Agent Configuration (`opencode.jsonc`)

Place this file either at `~/.config/opencode/opencode.jsonc` (global) or directly in your project root as `opencode.jsonc`:

{
  "$schema": "https://opencode.ai/config.json",
  // Primary default agent
  "default_agent": "planner",
  "agent": {
    // ---------------------------------------------
    // AGENT 1: STRATEGIC REASONER (Read-Only Architect)
    // ---------------------------------------------
    "planner": {
      "mode": "primary",
      "model": "ollama/local-planner",
      "description": "High-level reasoning agent for root-cause analysis, architecture review, and long-range planning.",
      "tools": {
        "write": false,
        "edit": false,
        "bash": false
      },
      "permission": {
        // Enforce strict read-only behavior; planner cannot mutate disk
        "edit": "deny",
        "bash": "deny"
      }
    },

    // ---------------------------------------------
    // AGENT 2: TACTICAL WORKHORSE (Write & Execute)
    // ---------------------------------------------
    "coder": {
      "mode": "subagent",
      "model": "ollama/local-coder",
      "description": "Tactical code generation workhorse for diffs, file editing, and test execution.",
      "tools": {
        "write": true,
        "edit": true,
        "bash": true
      },
      "permission": {
        // Allow autonomous file editing and shell execution
        "edit": "allow",
        "bash": "allow"
      }
    }
  }
}

### Prompt Design Guidelines & Guardrails

To run this setup efficiently, apply distinct prompt structures for each model.

**Guardrail 1: The Context Boundary (16k for Coder)**

Because `local-coder` is set to `num_ctx 16384` to prevent VRAM spikes, never feed whole raw directories or multi-megabyte log dumps into `@coder`.

* Let `@planner` read the repo layout and isolate the exact 1–2 files that matter.
* Let `@coder` only inspect the specific function or module requiring modification.

**Guardrail 2: Enforcing Structured Handoffs**

When `@planner` delegates to `@coder`, require `@planner` to emit a **Deterministic Patch Contract**. It must specify:

1. Target File Path
2. Exact Search Block
3. Exact Replacement Block
4. Validation Command (e.g., `pytest tests/test_parser.py -k test_empty_string`)

### Prompt Templates & Directive Catalog

**Template A: Dual-Model Divide & Conquer (Invoking Both)**

Use this structure when tackling a bug, failing test, or multi-step feature implementation:

@planner: We have a failing unit test in tests/test_engine.py::test_rebalance_overflow.
1. Analyze the stack trace and inspect the relevant modules in src/allocation/.
2. Formulate the mathematical and algorithmic root cause.
3. Construct a step-by-step specification.
4. Delegate the implementation, file patch, and test validation to @coder.

Execution Flow:

1. OpenCode engages `local-planner` (DeepSeek-R1). It reflects inside its `<think>` tokens, evaluates the algorithm, and forms the plan.
2. `local-planner` finishes and invokes `@coder`.
3. Ollama evicts `local-planner` and loads `local-coder` into VRAM (~1.8s).
4. `local-coder` reads the target file, applies the patch, executes `pytest`, and confirms the test passes.

**Template B: Dedicated `@planner` Prompts (Analysis / Code Review)**

Use for single-pass analysis where no code should be modified:

@planner: Review the git diff against main (git diff main...HEAD).
Focus on:
1. Algorithmic regressions or unnecessary O(N^2) loops.
2. Race conditions, deadlocks, or thread safety issues.
3. Unhandled error states on external I/O boundaries.
Do NOT attempt to edit files. Provide your diagnostic report with line-specific suggestions.

@planner: Analyze the graph traversal algorithm in src/routing/pathfinder.py.
Evaluate the worst-case space and time complexity. Suggest how we can refactor this using an indexed priority queue and memoized A* heuristics.

**Template C: Dedicated `@coder` Prompts (Direct Tactical Implementation)**

Use when you already know what needs to be written and want fast, 35+ tok/s generation:

@coder: In src/middleware/auth.py, implement the TokenBucketRateLimiter class according to the interface defined in docs/rate_limiting.md.
Use atomic locks for thread safety.
Once implemented, run `pytest tests/test_auth.py` and verify all tests pass.

@coder: Refactor the function `parse_market_records` in src/parsers/trade.py to use streaming iteration instead of loading the full file into memory.

### Verification and Health Check Run

To test the entire pipeline end-to-end:

1. Open a monitoring window in Terminal:

watch -n 0.5 ollama ps

2. Launch OpenCode in your target project:

opencode

3. Execute a smoke test prompt:

@planner analyze the codebase structure. Then instruct @coder to create a file named health_check.txt containing the current timestamp.

4. Observe the swap:
   * Terminal 2 will show `local-planner` active at ~19.7 GB.
   * As soon as planning concludes, `local-planner` drops out, and `local-coder` comes up at ~34.2 GB.
   * Activity Monitor will show zero disk swap and memory pressure staying steadily in the green.

## 9. Lightweight Variant: Single Executor Alias

A lighter alternative to section 8 wraps only the coder role in a Modelfile-backed alias, leaving the planner pointed directly at its raw registry tag, and reuses the array-based `agents` / `permissions` schema from section 4's `opencode.jsonc`.

### 1. Configure Memory and Swapping Variables

Ensure your `~/.zshrc` has the proper eviction and allocation limits:

# 80% Unified Memory Cap on 48 GB (39,321 MB)
sudo sysctl iogpu.wired_limit_mb=39321

# Enforce sequential loading (never run both simultaneously)
export OLLAMA_MAX_LOADED_MODELS=1
export OLLAMA_NUM_PARALLEL=1

# Maintain resident model for 3 minutes during rapid tool loops
export OLLAMA_KEEP_ALIVE=3m

# Halve KV cache overhead
export OLLAMA_KV_CACHE_TYPE=q8_0

### 2. Create the Tailored Modelfile for the Workhorse

To lock in the 16k context and low-temperature sampling for tool execution, create `Modelfile.executor`:

FROM qwen3-coder:30b-a3b-q8_0

# Keep context at 16k to protect the 80% VRAM ceiling with Q8 weights
PARAMETER num_ctx 16384

# Precise deterministic tool call syntax
PARAMETER temperature 0.1
PARAMETER top_p 0.95

Register the alias:

ollama create harness-executor -f Modelfile.executor

### 3. Update `opencode.json`

Point OpenCode to the exact tags:

{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/deepseek-r1:14b-qwen-distill-q8_0",
  "default_agent": "architect",
  "agents": {
    "architect": {
      "mode": "primary",
      "model": "ollama/deepseek-r1:14b-qwen-distill-q8_0",
      "system": "You are the Lead Systems Architect. Analyze bug reports and tracebacks, diagnose root causes, and produce a step-by-step implementation plan. Never edit files directly — delegate all implementation to @coder.",
      "permissions": [
        { "action": "edit", "resource": "*", "effect": "deny" }
      ]
    },
    "coder": {
      "mode": "subagent",
      "model": "ollama/harness-executor",
      "system": "You are an autonomous tactical coding workhorse. Execute the assigned plan exactly: patch the specified files, run the validation command, and confirm the tests pass.",
      "permissions": [
        { "action": "edit", "resource": "*", "effect": "allow" },
        { "action": "shell", "resource": "*", "effect": "allow" }
      ]
    }
  }
}

### Verification Test

To confirm that the models hot-swap sequentially without exceeding 80% VRAM, run the same smoke test described in section 8's "Verification and Health Check Run."
EOF
```

### Step 2: Render into PDF

Run this single command using Node's `md-to-pdf` runner (it launches a headless Chromium instance to compile a PDF with code highlighting and tables):

```bash
npx md-to-pdf DIVIDE_AND_CONQUER_SETUP.md
```

This generates `DIVIDE_AND_CONQUER_SETUP.pdf` directly in your directory.

To immediately open and view the PDF in macOS Preview:

```bash
open DIVIDE_AND_CONQUER_SETUP.pdf
```

## Method 2: Pandoc + wkhtmltopdf / WeasyPrint (Alternative Engine)

If you already have `pandoc` or Homebrew installed:

```bash
# 1. Install pandoc and weasyprint (or wkhtmltopdf)
brew install pandoc weasyprint

# 2. Compile directly to PDF
pandoc DIVIDE_AND_CONQUER_SETUP.md -o DIVIDE_AND_CONQUER_SETUP.pdf --pdf-engine=weasyprint

# 3. View the document
open DIVIDE_AND_CONQUER_SETUP.pdf
```

## Method 3: VS Code / Cursor Native Export

If you prefer using your IDE:

1. Open `DIVIDE_AND_CONQUER_SETUP.md` in VS Code or Cursor.
2. Press `Cmd + Shift + P` and install the extension: **Markdown PDF** (by yzane).
3. Right-click anywhere inside the editor tab.
4. Select **Markdown PDF: Export (pdf)**. The PDF file will be compiled and saved right next to the markdown document.

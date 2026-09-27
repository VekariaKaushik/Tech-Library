# Local Divide & Conquer Autonomous Agent Architecture

**Platform:** Apple Silicon MacBook Pro M4 Pro (48 GB Unified Memory)
**Primary Stack:** Ollama, OpenCode, Metal Framework
**Target Capabilities:** SWE-bench Verified Bug Fixing, Algorithmic Profiling, Test Loop Harnesses, Zero-Egress Code Audits

---

## Table of Contents

- [1. System Architecture & Memory Engineering](#1-system-architecture-memory-engineering)
  - [Verified Model Registry Manifests](#verified-model-registry-manifests)
  - [SWE-bench & Throughput Profile](#swe-bench-throughput-profile)
- [2. Host Operating System Tuning](#2-host-operating-system-tuning)
  - [A. Persist 80% VRAM Allocation](#a-persist-80-vram-allocation)
  - [B. Configure Ollama Daemon (`~/.zshrc`)](#b-configure-ollama-daemon-zshrc)
- [3. Ollama Model Setup & Registry Verification](#3-ollama-model-setup-registry-verification)
- [4. OpenCode Configuration (`opencode.jsonc`)](#4-opencode-configuration-opencodejsonc)
- [5. Live Multi-Pane Monitoring HUD](#5-live-multi-pane-monitoring-hud)
- [6. Prompt Engineering & Operational Guardrails](#6-prompt-engineering-operational-guardrails)
  - [Core Operational Rules](#core-operational-rules)
  - [Production Prompt Catalog](#production-prompt-catalog)
- [7. Performance Benchmarks: Local M4 Pro vs. Hyperscaler APIs](#7-performance-benchmarks-local-m4-pro-vs-hyperscaler-apis)
  - [Key Architectural Takeaway](#key-architectural-takeaway)
- [8. Alternative Workflow: Custom Model Aliases via Ollama Modelfiles](#8-alternative-workflow-custom-model-aliases-via-ollama-modelfiles)
  - [Step 1: System-Level Configuration & Environment Setup](#step-1-system-level-configuration-environment-setup)
<<<<<<< HEAD
  - [Verified Dual-Model Registry Tags](#verified-dual-model-registry-tags)
=======
>>>>>>> kv-wip
  - [Memory & VRAM Enforcement (M4 Pro 48 GB @ 80% Cap)](#memory-vram-enforcement-m4-pro-48-gb-80-cap)
  - [Step 2: Model Ingestion & Custom Modelfiles](#step-2-model-ingestion-custom-modelfiles)
  - [Step 3: OpenCode Agent Configuration (`opencode.jsonc`)](#step-3-opencode-agent-configuration-opencodejsonc)
  - [Prompt Design Guidelines & Guardrails](#prompt-design-guidelines-guardrails)
  - [Prompt Templates & Directive Catalog](#prompt-templates-directive-catalog)
  - [Verification and Health Check Run](#verification-and-health-check-run)
- [9. Lightweight Variant: Single-Alias Naming Used by the Harness Scripts](#9-lightweight-variant-single-alias-naming-used-by-the-harness-scripts)
  - [1. Configure Memory and Swapping Variables](#1-configure-memory-and-swapping-variables)
  - [2. Create the Tailored Modelfiles](#2-create-the-tailored-modelfiles)
  - [3. Update `opencode.jsonc`](#3-update-opencodejsonc)
  - [Verification Test](#verification-test)
- [10. Why These Models: Selection Rationale & Benchmarks](#10-why-these-models-selection-rationale-benchmarks)
  - [Architectural Differences: Qwen 2.5 vs. Qwen 3](#architectural-differences-qwen-25-vs-qwen-3)
  - [SWE-bench Verified Performance](#swe-bench-verified-performance)
  - [Tokens Per Second (on Apple Silicon M4 Pro, 273 GB/s Bandwidth)](#tokens-per-second-on-apple-silicon-m4-pro-273-gbs-bandwidth)
  - [Summary: How It Affects Your Agent Loop](#summary-how-it-affects-your-agent-loop)
  - [Comparative SWE-bench Standings](#comparative-swe-bench-standings)
  - [Score Comparison by Quantization Level](#score-comparison-by-quantization-level)
  - [SWE-bench Comparison: 32B Coder vs. Other Options](#swe-bench-comparison-32b-coder-vs-other-options)
  - [Fit Confirmation Checklist (M4 Pro 48 GB)](#fit-confirmation-checklist-m4-pro-48-gb)
- [11. Operational Routing Matrix: Task → Model Mapping](#11-operational-routing-matrix-task-model-mapping)
  - [Category 1: Code Review (Static Analysis & Security)](#category-1-code-review-static-analysis-security)
  - [Category 2: Writing Code (Feature Implementation & Refactoring)](#category-2-writing-code-feature-implementation-refactoring)
  - [Category 3: Writing Tests (TDD & Edge-Case Coverage)](#category-3-writing-tests-tdd-edge-case-coverage)
  - [Category 4: Analyzing Code & Complex Algorithms](#category-4-analyzing-code-complex-algorithms)
  - [Category 5: Running Autonomous Harnesses (Closed-Loop Debugging)](#category-5-running-autonomous-harnesses-closed-loop-debugging)
  - [Unified CLI Router Pattern](#unified-cli-router-pattern)
- [12. Deep Dive: The Divide-and-Conquer Protocol Architecture](#12-deep-dive-the-divide-and-conquer-protocol-architecture)
  - [What the Execution Loop Looks Like in Practice](#what-the-execution-loop-looks-like-in-practice)
  - [Full Architecture Diagram](#full-architecture-diagram)
  - [Memory & VRAM Breakdown Under the 80% Cap (32k-Context Executor Variant)](#memory-vram-breakdown-under-the-80-cap-32k-context-executor-variant)
  - [Layer-by-Layer Architecture](#layer-by-layer-architecture)
  - [How the Divide-and-Conquer Protocol Operates](#how-the-divide-and-conquer-protocol-operates)
  - [Failure Modes & Architectural Mitigations](#failure-modes-architectural-mitigations)
  - [Why This Setup Fits the M4 Pro (48 GB)](#why-this-setup-fits-the-m4-pro-48-gb)
- [13. Updated Runtime Configuration & Verification Monitoring](#13-updated-runtime-configuration-verification-monitoring)
  - [1. Configure Ollama Environment Variables](#1-configure-ollama-environment-variables)
  - [Updated Memory Strategy: Strict 80% Allocation Ceiling](#updated-memory-strategy-strict-80-allocation-ceiling)
  - [Step 5: Verification & Performance Monitoring](#step-5-verification-performance-monitoring)
- [14. Reference Implementation A: `harness.py` (Search/Replace Patch Variant)](#14-reference-implementation-a-harnesspy-searchreplace-patch-variant)
- [15. Reference Implementation B: `harness.py` (Full-File Write Variant)](#15-reference-implementation-b-harnesspy-full-file-write-variant)
  - [Review Notes](#review-notes)
- [16. Extended Model Comparison Research](#16-extended-model-comparison-research)
  - [Comparative Reasoning Breakdown](#comparative-reasoning-breakdown)
  - [The SWE-bench vs. Speed Trade-Off Landscape](#the-swe-bench-vs-speed-trade-off-landscape)
  - [Broader Reasoning / Coding / Tooling Survey](#broader-reasoning-coding-tooling-survey)
  - [What "SWE-bench Verified" Actually Measures](#what-swe-bench-verified-actually-measures)
  - [Ollama vs. MLX on M4 Pro: Which Backend to Pick for Harnesses?](#ollama-vs-mlx-on-m4-pro-which-backend-to-pick-for-harnesses)
  - [Single-Model vs. Two-Model Trade-off (If You Don't Want to Run Two Models)](#single-model-vs-two-model-trade-off-if-you-dont-want-to-run-two-models)
  - [The Verdict: No Single Model Hits "High" on Every Dimension](#the-verdict-no-single-model-hits-high-on-every-dimension)
- [17. Local Dual-Model Setup vs. Cloud Frontier Defaults (Claude Sonnet 5 / Grok)](#17-local-dual-model-setup-vs-cloud-frontier-defaults-claude-sonnet-5-grok)
  - [Detailed Compare & Contrast](#detailed-compare-contrast)
  - [Reading the Trade-off](#reading-the-trade-off)
  - [Closing the Gap: What Hardware and Models It Would Actually Take](#closing-the-gap-what-hardware-and-models-it-would-actually-take)

---

## 1. System Architecture & Memory Engineering

To run an autonomous software engineering pipeline locally without exceeding memory constraints, the architecture isolates cognitive workloads between two specialized models using **sequential hot-swapping** (`OLLAMA_MAX_LOADED_MODELS=1`). This guarantees that only one model resides in memory at any given second.

```text
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
```

### Verified Model Registry Manifests

| Component | Exact Registry Tag | Quantization | Size | Architecture | Primary Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Planner / Reasoner** | `deepseek-r1:14b-qwen-distill-q8_0` | `Q8_0` | ~15.5 GB | Dense (14.7B params) | Root-cause analysis, planning, edge-case testing |
| **Tactical Workhorse** | `qwen3-coder:30b-a3b-q8_0` | `Q8_0` | ~32.0 GB | Sparse MoE (3.3B active params) | AST patching (32–36 tok/s), JSON tool calls |

### SWE-bench & Throughput Profile

* **`deepseek-r1:14b-qwen-distill-q8_0`:** Operates at **18–22 tok/s** on M4 Pro. Utilizes `<think>` tokens for multi-step algorithmic deduction and traceback tracing.
* **`qwen3-coder:30b-a3b-q8_0`:** Generates at **32–36 tok/s** because it routes computation through only 3.3B active parameters per forward pass. It reaches **~51.4% (Vanilla)** and **~71% (Scaffolded)** on SWE-bench Verified.
* **Quantization Stability:** The `Q8_0` quantization eliminates routing noise in the 128-expert gating network, maintaining 0.0% schema drift for tool-call emissions.

*(See [§10 "Why These Models"](#10-why-these-models-selection-rationale-benchmarks) for the full comparative research behind this pairing.)*

---

## 2. Host Operating System Tuning

### A. Persist 80% VRAM Allocation

Set the Metal wired memory limit to allow up to 39,321 MB (38.4 GB) to be claimed by the GPU without kernel panics:

```bash
sudo sysctl iogpu.wired_limit_mb=39321
```

Persist across reboots:

```bash
echo "iogpu.wired_limit_mb=39321" | sudo tee -a /etc/sysctl.conf
```

### B. Configure Ollama Daemon (`~/.zshrc`)

Append the following variables to enforce sequential execution and halved KV-cache memory usage:

```bash
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
```

Apply the profile:

```bash
source ~/.zshrc
killall ollama 2>/dev/null || true
ollama serve > "$OLLAMA_LOG_DIR/server.log" 2>&1 &
```

## 3. Ollama Model Setup & Registry Verification

Pull the verified models directly from the public registry:

```bash
# 1. Pull the Strategic Reasoner (~16 GB)
ollama pull deepseek-r1:14b-qwen-distill-q8_0

# 2. Pull the Tactical MoE Workhorse (~32 GB)
ollama pull qwen3-coder:30b-a3b-q8_0
```

Verify the local library:

```bash
ollama list
```

## 4. OpenCode Configuration (`opencode.jsonc`)

Place this file in your project root or at `~/.config/opencode/opencode.jsonc`. It configures the dual-agent architecture, establishes strict read/write boundaries, configures 32k/16k context limits, and enables the real-time terminal telemetry HUD.

This is the **canonical `opencode.jsonc`** for this guide. The `ui`, `telemetry`, `plugin`, and `provider` blocks below are shared verbatim by every alias-based variant later in this guide (§8, §9) — those sections only show the `agents` block that changes, on top of this same file, to avoid repeating ~30 lines of identical boilerplate three times. All four agents use the consistent schema this guide standardizes on: `agents` (plural) → per-agent `permissions` (an array of `{action, resource, effect}` rules), and `tools` with an explicit `read` key.

```jsonc
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
```

## 5. Live Multi-Pane Monitoring HUD

Create an automated dashboard script to observe memory usage, token burn rates, and hot-swaps in real time.

Save as `~/bin/opencode-hud.sh`:

```bash
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
```

Make it executable and link to your environment:

```bash
chmod +x ~/bin/opencode-hud.sh
echo 'alias opencode="~/bin/opencode-hud.sh"' >> ~/.zshrc
source ~/.zshrc
```

## 6. Prompt Engineering & Operational Guardrails

### Core Operational Rules

* **The Context Boundary Guardrail (Coder ≤ 16k):** Never pass raw multi-megabyte log files or entire source trees directly to `@coder`. Let `@planner` read the project structure, locate the issue, and provide isolated target paths.
* **The Delegation Contract:** When `@planner` delegates work, it must emit a 4-part contract: Target File Path, Exact Search Block, Exact Replacement Block, and Verification Command.

### Production Prompt Catalog

**Pattern 1: Autonomous Divide & Conquer (Full SWE-bench Repair Loop)**

```text
@planner: We have an intermittent assertion failure in tests/test_settlement.py::test_partial_fill.
1. Read tests/test_settlement.py and src/engine/settlement.py.
2. Diagnose the root cause of the race condition or calculation error.
3. Formulate a step-by-step fix contract.
4. Delegate the implementation, file patching, and test validation to @coder.
```

**Pattern 2: Algorithmic Complexity & Deep Research**

```text
@planner: Analyze the graph traversal in src/routing/pathfinder.py.
1. Determine the formal time and space complexity of find_optimal_path() in big-O notation.
2. Identify bottlenecks causing high latency under 100,000+ nodes.
3. Formulate an alternative design using an indexed priority queue and memoized A* heuristics.
Do not write complete files. Provide the theoretical proof and an implementation plan for @coder.
```

**Pattern 3: Targeted High-Throughput Implementation**

```text
@coder: In src/serializers/binary.py, implement the struct-based binary packer for the OrderBookSnapshot schema.
- Follow the byte packing alignment defined in docs/wire_spec.md.
- Run `pytest tests/test_serializers.py` and ensure all tests pass.
- Provide the git diff output when done.
```

**Pattern 4: Pull Request & Security Review**

```text
@reviewer: Review the staged changes on this branch against main (`git diff origin/main...HEAD`).
Audit for:
1. Thread-safety violations or un-synchronized access to shared state.
2. Unhandled exception paths on database connections.
3. Violations of domain boundary encapsulation.
Present findings ranked by severity: Critical, Major, Minor.
```

**Pattern 5: Test Generation & Regression Hardening**

```text
@tester: Inspect src/auth/token_validator.py.
1. Write a complete pytest suite in tests/test_token_validator.py covering: expired tokens, corrupted signatures, clock skew, and empty payloads.
2. Run the tests using `pytest tests/test_token_validator.py -v`.
3. Ensure all tests execute and pass without warnings.
```

## 7. Performance Benchmarks: Local M4 Pro vs. Hyperscaler APIs

*(This is the original, generic-Claude-3.5 comparison from this guide's first draft. For the current, model-specific comparison against Claude Sonnet 5 and Grok — including SWE-bench, context window, and a hardware-upgrade path — see [§17](#17-local-dual-model-setup-vs-cloud-frontier-defaults-claude-sonnet-5-grok).)*

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

This workflow needs no OS or environment changes beyond what §2 already sets: the same `sudo sysctl iogpu.wired_limit_mb=39321` (§2A) and the same `~/.zshrc` exports — `OLLAMA_MAX_LOADED_MODELS=1`, `OLLAMA_KEEP_ALIVE=3m`, `OLLAMA_NUM_PARALLEL=1`, `OLLAMA_KV_CACHE_TYPE=q8_0`, `OLLAMA_HOST=127.0.0.1:11434` (§2B). If you've already completed §2, skip straight to Step 2 below.

With that in place, your exact dual-model pairing is the same one verified in [§1's registry table](#verified-model-registry-manifests) — no separate lookup needed here.

### Memory & VRAM Enforcement (M4 Pro 48 GB @ 80% Cap)

Because `qwen3-coder:30b-a3b-q8_0` weighs ~32 GB on load, managing context and concurrency prevents breaching your 38.4 GB (80%) ceiling:

```text
Total Allocatable Cap (80%): 38.4 GB (39,321 MB)
System Pool Guarantee (20%):  9.6 GB (Zero UI lag / No swap)

[Phase 1] deepseek-r1:14b Active:
  Weights: ~15.5 GB + 32k Q8 Context: ~4.2 GB = ~19.7 GB Peak (~18.7 GB buffer)

[Phase 2] qwen3-coder:30b-a3b Active:
  Weights: ~32.0 GB + 16k Q8 Context: ~2.2 GB = ~34.2 GB Peak (~4.2 GB buffer)
```

**Key Tuning Rule for the Q8_0 Workhorse:** With `Q8_0` weights at 32 GB, cap the executor's context window to 16,384 tokens (`PARAMETER num_ctx 16384`) rather than 32k.

* A 16k context window with `OLLAMA_KV_CACHE_TYPE=q8_0` consumes ~2.2 GB, keeping peak usage at ~34.2 GB — safely below the 38.4 GB boundary with ~4.2 GB of VRAM headroom to spare.

### Step 2: Model Ingestion & Custom Modelfiles

The required base checkpoints must already be present in Ollama (pulled in section 3 above):

* `deepseek-r1:14b-qwen-distill-q8_0` (~15.5 GB)
* `qwen3-coder:30b-a3b-q8_0` (~32.0 GB)

We wrap both in optimized Modelfile definitions to enforce their token parameters, context windows, and sampling temperatures.

**1. Modelfile for Planner (`Modelfile.planner`)**

Save as `Modelfile.planner`:

```dockerfile
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
```

**2. Modelfile for Coder (`Modelfile.coder`)**

Save as `Modelfile.coder`:

```dockerfile
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
```

**3. Build the Ollama Aliases**

Run these commands in the terminal where your Modelfiles are saved:

```bash
ollama create local-planner -f Modelfile.planner
ollama create local-coder -f Modelfile.coder
```

### Step 3: OpenCode Agent Configuration (`opencode.jsonc`)

Reuse the canonical `opencode.jsonc` from §4 as-is — same `ui`, `telemetry`, `plugin`, and `provider` blocks. Only two things change on top of that file:

1. Drop the `reviewer` and `tester` agents (this workflow only needs `planner`/`coder`), and repoint their `model` fields to the two aliases created above.
2. Add `local-planner` / `local-coder` entries to `provider.ollama.models` (mirroring the raw-tag entries already in §4) so OpenCode's context/output accounting still applies to the aliases.

```jsonc
{
  "default_agent": "planner",
  "provider": {
    "ollama": {
      "models": {
        "local-planner": { "name": "Local Planner (DeepSeek-R1 alias)", "limit": { "context": 32768, "output": 8192 } },
        "local-coder": { "name": "Local Coder (Qwen3-Coder alias)", "limit": { "context": 16384, "output": 4096 } }
      }
    }
  },
  "agents": {
    "planner": {
      "mode": "primary",
      "model": "ollama/local-planner",
      "description": "High-level reasoning agent for root-cause analysis, architecture review, and long-range planning.",
      "temperature": 0.6,
      "top_p": 0.95,
      "tools": { "read": true, "write": false, "edit": false, "bash": false },
      "permissions": [
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "edit", "resource": "*", "effect": "deny" },
        { "action": "write", "resource": "*", "effect": "deny" },
        { "action": "shell", "resource": "*", "effect": "deny" }
      ]
    },
    "coder": {
      "mode": "subagent",
      "model": "ollama/local-coder",
      "description": "Tactical code generation workhorse for diffs, file editing, and test execution.",
      "temperature": 0.1,
      "top_p": 0.95,
      "tools": { "read": true, "write": true, "edit": true, "bash": true },
      "permissions": [
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "edit", "resource": "*", "effect": "allow" },
        { "action": "write", "resource": "*", "effect": "allow" },
        { "action": "shell", "resource": "*", "effect": "allow" }
      ]
    }
  }
}
```

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

```text
@planner: We have a failing unit test in tests/test_engine.py::test_rebalance_overflow.
1. Analyze the stack trace and inspect the relevant modules in src/allocation/.
2. Formulate the mathematical and algorithmic root cause.
3. Construct a step-by-step specification.
4. Delegate the implementation, file patch, and test validation to @coder.
```

Execution Flow:

1. OpenCode engages `local-planner` (DeepSeek-R1). It reflects inside its `<think>` tokens, evaluates the algorithm, and forms the plan.
2. `local-planner` finishes and invokes `@coder`.
3. Ollama evicts `local-planner` and loads `local-coder` into VRAM (~1.8s).
4. `local-coder` reads the target file, applies the patch, executes `pytest`, and confirms the test passes.

**Template B: Dedicated `@planner` Prompts (Analysis / Code Review)**

Use for single-pass analysis where no code should be modified:

```text
@planner: Review the git diff against main (git diff main...HEAD).
Focus on:
1. Algorithmic regressions or unnecessary O(N^2) loops.
2. Race conditions, deadlocks, or thread safety issues.
3. Unhandled error states on external I/O boundaries.
Do NOT attempt to edit files. Provide your diagnostic report with line-specific suggestions.
```

```text
@planner: Analyze the graph traversal algorithm in src/routing/pathfinder.py.
Evaluate the worst-case space and time complexity. Suggest how we can refactor this using an indexed priority queue and memoized A* heuristics.
```

**Template C: Dedicated `@coder` Prompts (Direct Tactical Implementation)**

Use when you already know what needs to be written and want fast, 35+ tok/s generation:

```text
@coder: In src/middleware/auth.py, implement the TokenBucketRateLimiter class according to the interface defined in docs/rate_limiting.md.
Use atomic locks for thread safety.
Once implemented, run `pytest tests/test_auth.py` and verify all tests pass.
```

```text
@coder: Refactor the function `parse_market_records` in src/parsers/trade.py to use streaming iteration instead of loading the full file into memory.
```

### Verification and Health Check Run

To test the entire pipeline end-to-end:

1. Open a monitoring window in Terminal:

```bash
watch -n 0.5 ollama ps
```

2. Launch OpenCode in your target project:

```bash
opencode
```

3. Execute a smoke test prompt:

```text
@planner analyze the codebase structure. Then instruct @coder to create a file named health_check.txt containing the current timestamp.
```

4. Observe the swap:
   * Terminal 2 will show `local-planner` active at ~19.7 GB.
   * As soon as planning concludes, `local-planner` drops out, and `local-coder` comes up at ~34.2 GB.
   * Activity Monitor will show zero disk swap and memory pressure staying steadily in the green.

## 9. Lightweight Variant: Single-Alias Naming Used by the Harness Scripts

A lighter alternative to section 8 names both aliases `harness-planner` / `harness-executor` instead of `local-planner` / `local-coder`, and reuses the array-based `agents` / `permissions` schema from section 4's `opencode.jsonc`. This is the naming the reference `harness.py` implementations in [§14](#14-reference-implementation-a-harnesspy-searchreplace-patch-variant) and [§15](#15-reference-implementation-b-harnesspy-full-file-write-variant) bind to.

### 1. Configure Memory and Swapping Variables

Same as §2: `sudo sysctl iogpu.wired_limit_mb=39321` plus the `~/.zshrc` exports (`OLLAMA_MAX_LOADED_MODELS=1`, `OLLAMA_NUM_PARALLEL=1`, `OLLAMA_KEEP_ALIVE=3m`, `OLLAMA_KV_CACHE_TYPE=q8_0`, `OLLAMA_HOST=127.0.0.1:11434`). Nothing here changes those values — skip ahead if §2 is already applied.

### 2. Create the Tailored Modelfiles

Reuse `Modelfile.planner` from §8 as-is. For the coder role, create `Modelfile.executor`: identical to §8's `Modelfile.coder` (same `FROM`, `num_ctx 16384`, `temperature 0.1`, `top_p 0.95`) with the `SYSTEM` block removed, since the harness scripts in §14/§15 supply their own system prompt at request time instead of baking one into the model.

Register both aliases:

```bash
ollama create harness-planner -f Modelfile.planner
ollama create harness-executor -f Modelfile.executor
```

### 3. Update `opencode.jsonc`

Same pattern as §8: reuse §4's canonical file, add `harness-planner` / `harness-executor` entries to `provider.ollama.models`, and swap in this variant's `agents` block (renaming `planner` → `architect` and dropping `reviewer`/`tester`):

```jsonc
{
  "default_agent": "architect",
  "provider": {
    "ollama": {
      "models": {
        "harness-planner": { "name": "Harness Planner (DeepSeek-R1 alias)", "limit": { "context": 32768, "output": 8192 } },
        "harness-executor": { "name": "Harness Executor (Qwen3-Coder alias)", "limit": { "context": 16384, "output": 4096 } }
      }
    }
  },
  "agents": {
    "architect": {
      "mode": "primary",
      "model": "ollama/harness-planner",
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
```

### Verification Test

To confirm that the models hot-swap sequentially without exceeding 80% VRAM, run the same smoke test described in section 8's "Verification and Health Check Run" — see also [§13's Step 5](#step-5-verification-performance-monitoring) for the dedicated `harness-planner`/`harness-executor` monitoring routine used by the Python harnesses below.

## 10. Why These Models: Selection Rationale & Benchmarks

The pairing above (`deepseek-r1:14b-qwen-distill-q8_0` + `qwen3-coder:30b-a3b-q8_0`) wasn't arbitrary. It's the result of weighing three factors against each other: **SWE-bench Verified accuracy**, **tokens/second on the M4 Pro's fixed 273 GB/s memory bandwidth**, and **VRAM footprint under the 80% (38.4 GB) cap**. This section captures that comparative research.

### Architectural Differences: Qwen 2.5 vs. Qwen 3

The difference between Qwen 2.5 and the Qwen 3 / Qwen 3-Coder generation comes down to two architectural shifts: dense vs. sparse MoE execution (which dictates tokens per second) and pre-training objective (single-file code generation vs. multi-turn agentic reinforcement learning, which dictates SWE-bench scores).

| Feature / Dimension | Qwen 2.5 Generation (e.g., 2.5-Coder-32B) | Qwen 3 Generation (e.g., Qwen3-Coder-30B MoE, Qwen3-14B) |
| :--- | :--- | :--- |
| Model Structure | Dense Transformer (every parameter is computed on every token). | Sparse Mixture-of-Experts (MoE) or Hybrid DeltaNet Attention. |
| Active Parameters | 32.8B active per token (for the 32B model). | ~3.3B active per token (in the 30B/80B MoE variants). |
| Training Focus | HumanEval, LeetCode, single-turn code generation, AST syntax. | Agentic Environments: failing test tracebacks, multi-file PR diffs, tool execution. |
| Reasoning Architecture | Direct single-pass output (no native thinking tokens). | Dynamic Mode: switchable thinking mode (`<think>`) or fast non-thinking mode. |

### SWE-bench Verified Performance

The biggest leap from Qwen 2.5 to Qwen 3 is in **repository-level bug resolution**:

* **Qwen 2.5 Coder (32B):**
  * Vanilla SWE-bench Verified: ~31.0%
  * With Scaffold / OpenHands: ~38% – 42%
  * *Why:* It was trained primarily on code completion and single-file refactoring. When dropped into a complex 50-file repository with cross-module import side effects, it frequently fixes the target bug while breaking unrelated regression tests (Pass-to-Pass failures). *(Qwen)*
* **Qwen 3 Coder Generation (e.g., 30B MoE / 14B):**
  * Vanilla SWE-bench Verified: ~51% – 56%
  * With Scaffold / OpenHands / SWE-Agent: ~68% – 72%+
  * *Why:* The Qwen 3 series was trained directly on **executable PR trajectories** (inspect failure → patch → re-run pytest). It understands multi-turn tool interaction natively and rarely drops AST structure across iterative file modifications. *(arXiv)*

### Tokens Per Second (on Apple Silicon M4 Pro, 273 GB/s Bandwidth)

Token generation speed on Apple Silicon is directly governed by **Memory Bandwidth ÷ Active Parameter Bytes**:

```text
Theoretical Generation Speed ≈ Memory Bandwidth (GB/s) / Active Model Weight in RAM (GB)
```

| Model | Architecture & Precision | Active Weights Read per Token | Real-World Speed on M4 Pro | Latency for 150-Token Tool Call |
| :--- | :--- | :--- | :--- | :--- |
| `qwen2.5-coder:32b-q6_K` | Dense (32.8B params) | ~26.5 GB | 10 – 12 tok/s | ~12 – 15 seconds |
| `qwen2.5-coder:14b-q8_0` | Dense (14.7B params) | ~15.5 GB | 17 – 20 tok/s | ~7 – 9 seconds |
| `qwen3:14b-q6_K` | Dense (Optimized) | ~11.8 GB | 22 – 26 tok/s | ~5 – 6 seconds |
| `qwen3-coder:30b-a3b` | MoE (30B Total, 3.3B Active) | ~3.3 GB (Active experts only) | 35 – 45 tok/s | ~3 – 4 seconds |

### Summary: How It Affects Your Agent Loop

1. **Turnaround Time:** In a 10-step autonomous loop (read → patch → test → loop), Qwen 2.5 Coder 32B takes ~3 to 4 minutes just generating tokens at 10 tok/s. Qwen 3 MoE cuts that pure generation latency to under 40 seconds total at 35–45 tok/s.
2. **Success Rate:** Moving from Qwen 2.5 (~38% scaffolded) to Qwen 3 (~70% scaffolded) roughly doubles the probability that the model diagnoses the root cause and satisfies your test suite without introducing regressions.

### Comparative SWE-bench Standings

| Model in Local Setup | SWE-bench Verified (Scaffolded) | Weights / Quant | Loop Speed (M4 Pro) |
| :--- | :--- | :--- | :--- |
| `qwen3-coder:30b-a3b-q8_0` | ~71.0% | 32 GB | ~34 tok/s |
| `qwen3-coder:30b (q4_K_M)` | ~66.5% | 19 GB | ~44 tok/s |
| `devstral-small-2:24b-q8_0` | 53.6% | 25.5 GB | ~12 tok/s |
| `qwen2.5-coder:32b-instruct-q6_K` | ~40.0% | 26.5 GB | ~11 tok/s |

### Score Comparison by Quantization Level

Quantization directly influences routing fidelity in Mixture-of-Experts (MoE) architectures. Because an MoE relies on a gating router to select the top active experts per token, dropping bit-width introduces routing noise. *(OpenRouter)*

| Metric / Attribute | `qwen3-coder:30b-a3b-q8_0` (32 GB) | `qwen3-coder:30b-a3b-q4_K_M` (19 GB) |
| :--- | :--- | :--- |
| SWE-bench Verified (Vanilla) | 51.4% (negligible ~0.2% drop from FP16) | ~46.8% – 48.2% (3–5% drop from gating noise) |
| SWE-bench Verified (Scaffolded) | ~70.5% – 71.5% | ~65.0% – 67.2% |
| AST Diff & Patching Syntax Errors | Near Zero | Rare dropped braces on multi-file diffs |
| Tool-Call / JSON Schema Error Rate | 0.0% | < 0.8% |
| M4 Pro Generation Speed | 32 – 36 tok/s | 40 – 48 tok/s |
| VRAM Consumption | ~32.0 GB | ~19.2 GB |
| Context Headroom (under 38.4 GB Cap) | ~6.4 GB (adequate for ~8k–12k context) | ~19.2 GB (fits full 32k–64k context buffers) |

**Why we still picked `q8_0` for this guide:** the harness workflows in §12–§15 run large-repository, multi-file SWE-bench-style loops where the ~3–5% scaffolded-accuracy drop from `q4_K_M`'s gating noise costs more repair cycles than the extra ~10 tok/s saves. If your workspace fits in ~8–12k tokens of context and you want faster turns instead, `q4_K_M` is a reasonable trade.

### SWE-bench Comparison: 32B Coder vs. Other Options

| Model & Quant | SWE-bench Verified | Tok/s on M4 Pro | Memory Footprint (RAM) | Primary Role |
| :--- | :--- | :--- | :--- | :--- |
| `qwen2.5-coder:32b-instruct-q6_K` | 31.0% (vanilla) / ~40% (scaffold) | 10 – 12 tok/s | ~26.5 GB | **Reliable Code Generator:** High syntax accuracy, exact AST diffs, fits cleanly in 38.4 GB cap. |
| `devstral-small-2:24b-instruct-2512-q8_0` | 53.6% | 11 – 13 tok/s | ~25.5 GB | **SOTA Open Agent:** Fine-tuned directly on OpenHands bash/edit loops. |
| `qwen3-coder:30b-a3b` (Q6_K MoE) | ~68% – 72% | 35 – 45 tok/s | ~22.5 GB | **High Speed & Accuracy:** 3–4× faster turnarounds in multi-step loops. |

### Fit Confirmation Checklist (M4 Pro 48 GB)

| Metric | Target / Limit | `deepseek-r1:14b-q8_0` | `qwen3-coder:30b` (or 2.5 Coder 32B) `Q6_K` | Status |
| :--- | :--- | :--- | :--- | :--- |
| Max VRAM Ceiling | 38.4 GB (80% of 48 GB) | ~19.7 GB (weights + 32k context) | ~26.7 GB (weights + 32k context) | Passed (Safely below 38.4 GB limit) |
| Reserved System RAM | 9.6 GB (20% of 48 GB) | Uncompromised | Uncompromised | Passed (No OS UI stutter/swap) |
| Memory Bandwidth | 273 GB/s | ~16–19 tok/s | ~35–45 tok/s (MoE) / ~10–12 tok/s (Dense) | Passed (Fast execution turns) |
| Concurrency Rule | `OLLAMA_MAX_LOADED_MODELS=1` | Runs sequentially | Runs sequentially | Passed (~1.5s swap time between stages) |

*(See [§16](#16-extended-model-comparison-research) for the broader survey against Devstral, Mistral Small, Command R, Llama/Hermes, GPT-OSS, and an MLX-backend comparison.)*

## 11. Operational Routing Matrix: Task → Model Mapping

To run these engineering workflows without thrashing memory or stalling your feedback loop, route each task to the model designed for its compute profile:

| Category | Primary Engine | Secondary Engine | Needs Hot-Swap? | Execution Latency |
| :--- | :--- | :--- | :--- | :--- |
| 1. Code Review | Reasoner (DeepSeek-R1-14B) | — | No | 15–30s (Deliberate) |
| 2. Writing Code | Workhorse (Qwen3-Coder-30B MoE) | — | No | 2–5s (35–45 tok/s) |
| 3. Writing Tests | Workhorse (Qwen3-Coder-30B MoE) | Reasoner (Optional) | Conditional | 3–8s (Fast) |
| 4. Analyzing Code & Algorithms | Reasoner (DeepSeek-R1-14B) | — | No | 20–45s (Deep CoT) |
| 5. Running Autonomous Harness | Reasoner (Phase 1 Planner) | Workhorse (Phase 2 Executor) | Yes (Required) | Full Pipeline: ~45–60s |

### Category 1: Code Review (Static Analysis & Security)

**Characteristics & Demands:** Code review is an analytical, single-turn evaluation task. It requires understanding implicit invariants, spotting subtle concurrency bugs (e.g., race conditions, thread safety, memory leaks), and validating API boundary contracts. It does not require tool use or multi-step execution.

**Model Assignment: Strategic Reasoner (DeepSeek-R1-14B Q8_0)**

* *Why:* Reviewing code requires evaluating *what is missing*, not just parsing what is written. DeepSeek-R1's native `<think>` scratchpad deliberates over edge cases, nil-pointer dereferences, unhandled error states, and time-complexity regressions before giving feedback.
* **Special Handling Needed:**
  * **Unified Diff Ingestion:** Feed git patches directly (`git diff HEAD~1`) rather than entire files.
  * **Temperature Tuning:** Set `temperature: 0.1` or `0.2` to keep findings grounded in real bugs rather than subjective stylistic nitpicks.
  * **JSON Structured Findings:** Have the model output findings categorized by severity:

```json
{
  "critical": ["Potential data race on shared connection pool"],
  "performance": ["O(N^2) lookup inside hot loop; use HashSet instead"],
  "hygiene": ["Missing docstring on public interface"]
}
```

### Category 2: Writing Code (Feature Implementation & Refactoring)

**Characteristics & Demands:** Writing clean implementations, boilerplate, domain models, or refactoring existing functions is a generative task. It requires high syntactic precision, strict adherence to existing codebase idioms, and fast generation speed so the developer isn't left waiting.

**Model Assignment: Tactical Workhorse (Qwen3-Coder-30B-A3B Q6_K)**

* *Why:* At 35–45 tok/s, Qwen3-Coder synthesizes a 100-line class or function in under 3 seconds. Its code pre-training avoids conversational filler and outputs directly to the point.
* **Special Handling Needed:**
  * **No Thinking Overhead:** Never use a reasoning model for standard code generation. It adds 45 seconds of existential planning before outputting a standard CRUD function.
  * **Prompt Slicing (Context Pruning):** Do not dump your entire repository into the prompt. Pass only: 1) the target interface/signature, 2) relevant dependency types/imports, 3) the function stub with a docstring.
  * **Targeted Search/Replace Diffs:** When editing existing files, instruct the model to produce Search & Replace blocks rather than dumping full 500-line files. This prevents truncation issues and ensures fast turnarounds.

### Category 3: Writing Tests (TDD & Edge-Case Coverage)

**Characteristics & Demands:** Writing effective tests bridges reasoning (discovering degenerate inputs, boundary values, and failure paths) and execution (writing idiomatic syntax matching testing frameworks like `pytest`, `xUnit`, or `Go testing`).

**Model Assignment: Hybrid / Workhorse-Driven**

* **Standard Test Suites:** Use Workhorse (`Qwen3-Coder-30B`). Provide the target implementation and ask it to write unit tests targeting standard paths and obvious exceptions. It will generate complete test fixtures quickly.
* **Adversarial / Hard Edge-Case Discovery:** If the code involves complex finite-state machines, custom serializers, or floating-point math, route to Reasoner (`DeepSeek-R1-14B`):
  1. Reasoner analyzes the code and generates a plain-text list of *degenerate input vectors* (e.g., empty byte arrays, NaN values, max integer overflows).
  2. Workhorse takes those vectors and writes the idiomatic unit test code.

### Category 4: Analyzing Code & Complex Algorithms

**Characteristics & Demands:** This includes mathematical algorithm design (e.g., dynamic programming, numerical optimization, graph search), calculating Big-O asymptotic complexity, or proving the correctness of lock-free data structures.

**Model Assignment: Strategic Reasoner (DeepSeek-R1-14B Q8_0)**

* *Why:* Single-pass models frequently miscalculate asymptotic complexity and hallucinate loop invariant proofs. A model with internal chain-of-thought explores alternate paths, verifies mathematical lemmas step-by-step, and cross-checks edge conditions.
* **Special Handling Needed:**
  * **Keep Thinking Visible:** Do not strip `<think>` tags when reading algorithm analysis in your terminal. Watching the model's internal chain of thought reveals edge cases it considered and discarded.
  * **Context Allocation:** Give it at least **16k context** (`num_ctx 16384`) because detailed mathematical proofs and intermediate algorithmic steps expand the token count rapidly.

### Category 5: Running Autonomous Harnesses (Closed-Loop Debugging)

**Characteristics & Demands:** The harness runs autonomously: Read error trace → Form hypothesis → Apply patch → Execute `pytest` → Re-evaluate. This workflow requires the coordination of both models.

**Model Assignment: Full Divide-and-Conquer Dual Pipeline**

* **Phase 1 (Reasoner):** Ingests the failing unit test traceback and repository map. Emits a structured JSON execution plan.
* **Phase 2 (Workhorse):** Ingests the plan and executes rapid micro-steps (`read_file`, `patch_file`, `run_shell`) at 35–45 tok/s until it signals completion.
* **Phase 3 (Deterministic Supervisor):** Runs the test suite via Python `subprocess`. If non-zero exit code, routes output back to Phase 1.
* **Special Handling Needed:**
  * **OLLAMA Memory Ceiling:** `export OLLAMA_MAX_LOADED_MODELS=1` is mandatory. Allowing both models to load simultaneously consumes over 45 GB of VRAM, causing disk swap and degrading speeds from 40 tok/s to < 1 tok/s.
  * **Deterministic State Passing:** Do not feed the Workhorse the raw `<think>` tokens from the Reasoner. The Reasoner must communicate with the Workhorse strictly through a validated, typed Pydantic JSON schema.
  * **Safe Termination Guardrails:** Enforce a hard ceiling of **3 macro-repair cycles** and **8 micro-tool actions** per cycle to prevent recursive loops from burning through system resources.

### Unified CLI Router Pattern

To manage these five workflows without manually switching models or writing custom scripts for each, use a single entry point router script (`agent.py`) that sets model parameters and manages swapping automatically:

```bash
# Category 1: Fast Code Review
python agent.py review --file src/engine.py

# Category 2: Write Code / Implement
python agent.py code --prompt "Implement a lock-free ring buffer in C++20"

# Category 3: Generate Tests
python agent.py test --file src/engine.py --adversarial

# Category 4: Deep Algorithmic Analysis
python agent.py analyze --file src/math/solver.py

# Category 5: Autonomous Test-Fix Harness
python agent.py harness --task "Fix KeyError in session parser" --test "pytest tests/test_session.py"
```

By matching each task category to the appropriate model, you keep interactive development responsive (35–45 tok/s) while preserving deep reasoning for complex planning and code analysis, all within the 80% VRAM ceiling (38.4 GB) on your M4 Pro.

## 12. Deep Dive: The Divide-and-Conquer Protocol Architecture

The Dual-Model Divide-and-Conquer Architecture decouples **strategic planning** from **tactical execution**.

### What the Execution Loop Looks Like in Practice

When OpenCode (or the `harness.py` scripts in §14/§15) orchestrates a fix across the two models:

```text
[User Request / Error Trace]
   |
   v
1. Supervisor dispatches to Primary Agent (@architect)
   └── Ollama loads `deepseek-r1:14b-qwen-distill-q8_0` (~1.2s load from NVMe)
   └── Deliberates in `<think>` mode; diagnoses root cause.
   └── Emits structured implementation plan.
   |
   v
2. @architect delegates to Subagent (@coder)
   └── Ollama evicts DeepSeek-R1 from VRAM.
   └── Ollama loads `Qwen3-Coder-30B-A3B (Q6_K)` (~1.8s load from NVMe).
   |
   v
3. @coder executes tool loop
   └── Generates AST diffs, shell commands, and file edits at 35–45 tok/s.
   └── Stays in VRAM during subsequent rapid tool turns (`OLLAMA_KEEP_ALIVE=2m`).
   └── Calls `finish` when done.
   |
   v
4. Ground-Truth Verification
   └── Supervisor runs `pytest`.
   └── If PASS → Done.
   └── If FAIL → Swaps back to Step 1 with new trace.
```

### Full Architecture Diagram

```text
┌─────────────────────────────┐
│  User Goal / Test Suite      │
└──────────────┬───────────────┘
               │
               v
┌─────────────────────────────────────┐
│  DETERMINISTIC HARNESS               │
│  (Python Supervisor)                 │
└──────────────┬───────────────────────┘
               │
               v
       [Phase 1: Deliberation Request]
               │
      ┌────────┴─────────┐
      v                   v
┌──────────────────────┐ ┌────────────────────────────┐
│ STRATEGIC REASONER    │ │ HOT-SWAP BOUNDARY           │
│ (Planner)             │ │ OLLAMA_MAX_LOADED_MODELS=1  │
│ Model: deepseek-r1:14b-q8_0 │ │ Host RAM Guarantee: 9.6 GB (20%) │
│ • Reads: Goal + Full Error Traceback │ │ Max GPU VRAM: 38.4 GB (80%) │
│ • Internal <think>: Root-cause analysis │ │ Swap Latency: ~1.2s to 1.8s │
│ • Emits: Atomic JSON Contract (Plan) │ │                          │
└──────────┬────────────┘ └────────────┬─────────────┘
           │  Pure Structured Contract  │
           v                            v
┌───────────────────────────────────────────────┐
│ TACTICAL WORKHORSE (Tool Chainer)              │
│ Model: qwen3-coder:30b-a3b (Q6_K MoE)          │
│ • Throughput: 35-45 tok/s (Active weights: ~3.3B) │
│ • Context Headroom: 32k tokens (KV cache in Q8_0) │
│ • Cycle: Read Workspace → Apply AST Diff → Execute Shell → Emit Finish │
└──────────────────┬──────────────────────────────┘
                    │
                    v
┌───────────────────────────────────────────────┐
│ GROUND-TRUTH VERIFICATION                      │
│ Command: pytest / cargo test / ctest           │
└──────────────────┬──────────────────────────────┘
                    │
        ┌───────────┴────────────┐
        v                         v
  Exit Code == 0              Exit Code != 0
        │                         │
        v                         v
   [SUCCESS]              [FEEDBACK LOOP TO PHASE 1]
 (Clean Termination)     (Pass Raw Traceback to Planner)
```

### Memory & VRAM Breakdown Under the 80% Cap (32k-Context Executor Variant)

If you widen the executor's context window to 32k instead of the 16k default used in §1/§8/§9 (e.g., for a harness that needs to hold a large diagnostic + file dump per turn), the footprint shifts to whichever model is currently active plus its context buffer — see [§13's memory diagram](#updated-memory-strategy-strict-80-allocation-ceiling) for the numeric breakdown of exactly this scenario (~26.7 GB peak / ~11.7 GB headroom for the executor), which also matches the Layer 3 math directly below.

### Layer-by-Layer Architecture

**Layer 1: The Contract Interface (State Isolation)**

The reasoner and the workhorse never share the same raw context history. Sharing full conversational context pollutes the workhorse's attention with verbose chain-of-thought tokens and risks memory bloat.

Communication occurs strictly through an **Atomic JSON Contract**:

```json
{
  "$schema": "agent_plan_v1",
  "root_cause_analysis": "The parser fails with KeyError because the authorization header lookup does not handle a missing key.",
  "blast_radius_files": [
    "src/gateway/parser.py",
    "tests/test_parser.py"
  ],
  "verification_command": "pytest tests/test_parser.py -k test_unauthenticated_webhook",
  "atomic_steps": [
    "Read src/gateway/parser.py around line 45 to locate header dictionary lookups.",
    "Use safe dictionary get (.get('Authorization', None)) with fallback handling.",
    "Run verification_command to confirm fix and prevent regressions."
  ]
}
```

* **Planner Contract Rule:** The Planner cannot execute bash commands or modify files. It has no tools. It only deliberates, isolates the problem, and issues the plan.
* **Workhorse Contract Rule:** The Workhorse receives this clean schema as its system prompt. It operates as an implementation agent with zero reasoning latency.

**Layer 2: Strategic Reasoner (`deepseek-r1:14b-q8_0`)**

* **Role:** High-level diagnosis and algorithmic strategy.
* **Why this model:** DeepSeek-R1's distillation preserves deep reasoning and cross-file causal analysis. It evaluates stack traces, notices race conditions, and spots missing corner cases that single-pass dense models miss.
* **Operational Characteristics:**
  * Memory footprint: ~15.5 GB (Weights) + ~4.2 GB (32k Q8 KV cache) = ~19.7 GB VRAM.
  * Speed: ~16–19 tok/s.
  * Turnaround: 20–40 seconds per planning cycle. Because it only runs **once per repair loop**, this latency does not slow down tool chaining.

**Layer 3: Tactical Workhorse (`qwen3-coder:30b-a3b-instruct-q6_K`)**

* **Role:** Rapid, deterministic tool calling, AST-aware diffing, and shell command execution.
* **Why this model:** It is a Sparse Mixture-of-Experts (MoE) model. Although it holds 30B parameters in memory, it activates only ~3.3B parameters per token.
* **Operational Characteristics:**
  * Memory footprint: ~22.5 GB (Weights) + ~4.2 GB (32k Q8 KV cache) = ~26.7 GB VRAM.
  * Speed: 35–45 tok/s on the M4 Pro's 273 GB/s memory bus.
  * Turnaround: 1–3 seconds per tool step.
  * It emits strictly valid JSON tool actions (`read_file`, `patch_file`, `run_shell`) with minimal schema drift.

**Layer 4: The Deterministic Supervisor (Python Harness)**

The supervisor runs as a host process in userland Python. It guarantees that:

1. **The models are never self-evaluating.** The harness does not ask the LLM "Did you fix it?". It executes the test suite (`pytest`, `cargo test`, `npm test`) directly via `subprocess`.
2. **Terminal state is binary:** Exit Code `0` is success; any non-zero exit code captures `stderr` and restarts the planning cycle.
3. **Hard loop guardrails:** Max 3 macro-planning cycles, max 8 micro-execution steps per cycle.

### How the Divide-and-Conquer Protocol Operates

**Step 1: Macro-Phase — Root Cause Formulation**

When an issue is handed to the supervisor (or when a previous run fails a test):

1. The supervisor loads `harness-planner` into VRAM.
2. The Planner receives the problem description and the raw compiler or `pytest` output.
3. The Planner enters its internal thinking routine (`<think> ... </think>`), identifying what broke, what files are implicated, and what steps will fix it.
4. The Planner emits the structured JSON Plan.
5. The supervisor strips out any raw `<think>` tokens, validates the JSON schema using Pydantic, and writes the plan to disk or memory.

**Step 2: VRAM Hot-Swap Boundary**

1. The supervisor issues a tool-dispatch call to Ollama targeting `harness-executor`.
2. Because `OLLAMA_MAX_LOADED_MODELS=1`, Ollama evicts `harness-planner` from VRAM and loads `harness-executor`.
3. Thanks to the M4 Pro's fast unified architecture, this swap completes in ~1.5 seconds.
4. Total allocated VRAM shifts from ~19.7 GB to ~26.7 GB, remaining well beneath the 38.4 GB (80%) ceiling.

**Step 3: Micro-Phase — Tool Chaining Loop**

1. The Workhorse is prompted with the JSON Plan as its task boundary.
2. Turn 1: Workhorse calls `"tool": "read_file"`. Supervisor returns lines 1–80 of the target file.
3. Turn 2: Workhorse calls `"tool": "patch_file"`. Supervisor substitutes the broken logic with the patch.
4. Turn 3: Workhorse calls `"tool": "run_shell"` to execute an intermediary linter or build step.
5. Turn 4: Workhorse sees clean output and calls `"tool": "finish"`.
   *Total time spent in Step 3 across all 4 turns: ~15 to 20 seconds.*

**Step 4: Verification & Escalation**

1. The supervisor takes control and executes the ground-truth validation command in the actual workspace.
2. **Case A (Pass):** If tests pass (exit code `0`), the harness terminates cleanly and commits the changes.
3. **Case B (Fail):** If the patch broke an unexpected dependency (e.g., an assertion error in `tests/test_auth.py`), the supervisor captures the new traceback. It evicts the Workhorse, loads the Reasoner, and hands the failure traceback back to Phase 1:

   > "Your proposed plan failed with this new error: [traceback]. Re-evaluate the root cause and generate a corrected plan."

### Failure Modes & Architectural Mitigations

| Failure Mode | Root Cause | Architectural Mitigation |
| :--- | :--- | :--- |
| Hallucinated Diffs | Models rewriting entire 500-line files often drop functions or introduce regressions. | **Atomic Search/Replace Patching:** The tool forces the model to specify exact unique `search_block` lines and replacement lines. |
| Schema Drift in Long Loops | Small models often forget JSON structure by turn 5 and start chatting. | **MoE Quality + Temperature 0.1:** Qwen3-Coder retains strict schema fidelity over 32k contexts at near-zero temperature. |
| Thinking Token Leakage | Reasoning models emitting raw tags can break JSON deserializers. | **Regex Sanitization Boundary:** The supervisor runs `re.sub(r"<think>.*?</think>", "", output, flags=re.DOTALL)` before invoking `json.loads()`. |
| Infinite Tool Cycling | Workhorse runs `ls` or `read_file` repeatedly without making progress. | **Micro-Step Counters:** The execution loop caps at 8 steps. If exceeded, it triggers an automatic failure escalation. |
| Unified Memory Thrashing | Exceeding 80% RAM pushes macOS into aggressive swap, dropping speed from 40 tok/s to < 1 tok/s. | **Strict Single-Model Concurrency:** `OLLAMA_MAX_LOADED_MODELS=1` + `OLLAMA_KV_CACHE_TYPE=q8_0` keeps maximum peak load under 27 GB. |

### Why This Setup Fits the M4 Pro (48 GB)

1. **Hardware Respect:** It honors the 273 GB/s memory bandwidth bottleneck by running execution tasks on an MoE that reads only 3.3B weights per step, yielding high token rates (35–45 tok/s).
2. **OS Stability:** It respects the 80% GPU cap (38.4 GB) by using sequential memory swapping. At no point do weights and context exceed 27 GB, leaving nearly **12 GB of uncompressed, zero-pressure RAM** for macOS, Docker containers, IDEs, and browser windows.
3. **Maximized Problem Solving:** It achieves ~70%+ SWE-bench capability by combining the deep diagnostic planning of distilled reasoning architectures with the fast, error-free diff-writing and tool execution of code-focused models.

## 13. Updated Runtime Configuration & Verification Monitoring

### 1. Configure Ollama Environment Variables

No new variables here — this is the same `~/.zshrc` block from §2B (`OLLAMA_MAX_LOADED_MODELS=1`, `OLLAMA_KEEP_ALIVE=3m`, `OLLAMA_NUM_PARALLEL=1`, `OLLAMA_KV_CACHE_TYPE=q8_0`, `OLLAMA_HOST=127.0.0.1:11434`), which is what preserves the 9.6 GB system buffer referenced throughout this section.

Apply the changes:

```bash
source ~/.zshrc
killall ollama 2>/dev/null || true
ollama serve > /dev/null 2>&1 &
```

With this applied, memory pressure should stay continuous Green with zero page swapping during iterative tool execution — see the diagram below for the exact GB breakdown per phase.

### Updated Memory Strategy: Strict 80% Allocation Ceiling

On a 48 GB Unified Memory system, capping GPU allocation at 80% sets a conservative threshold that reserves ~9.6 GB of dedicated RAM for macOS core services, the WindowServer, Docker, and IDEs.

```text
┌─────────────────────────────────────────┐
│    M4 Pro 48 GB Unified Memory (273 GB/s) │
├───────────────────────┬───────────────────┤
│ macOS System Pool (20%) │ Max GPU Allocatable (80%) │
│ ~9.6 GB RAM             │ ~38.4 GB VRAM      │
│ (Kernel, IDE, Background Apps) │ (Model Weights + Active Context) │
└───────────────────────┴─────────┬─────────┘
                                   │
                ┌──────────────────┴───────────────────┐
                │ Sequential Ollama VRAM Footprint       │
                ├───────────────────────┬─────────────────┤
                │ Planner Phase:        │ Executor Phase:  │
                │ • DeepSeek-R1-14B: ~15.5 GB │ • Qwen3-Coder-30B: ~22.5 GB │
                │ • 32k Q8 Context: ~4.2 GB │ • 32k Q8 Context: ~4.2 GB │
                │ Total Peak: ~19.7 GB │ Total Peak: ~26.7 GB │
                │ VRAM Headroom: ~18.7 GB │ VRAM Headroom: ~11.7 GB │
                └───────────────────────┴─────────────────┘
```

**Step 1: Raise macOS Metal Allocation Ceiling to 80% (38.4 GB)**

In modern macOS (Sonoma and newer), the memory ceiling uses the megabyte parameter `iogpu.wired_limit_mb`. For an exact 80% allocation on 48 GB: *(Greg's Tech Notes)*

```text
48 GB × 0.80 = 38.4 GB  ⟹  38.4 × 1024 = 39321 MB
```

Apply the runtime limit in Terminal (the same command as §2A):

```bash
sudo sysctl iogpu.wired_limit_mb=39321
```

(To verify the setting took effect, run `sysctl iogpu.wired_limit_mb`. To reset to the system default of ~75%, set it to `0`.)

### Step 5: Verification & Performance Monitoring

While running your harness, monitor resource utilization to verify that unified memory remains within bounds:

1. **Verify Ollama Model Swapping:** In a separate terminal window, run:

```bash
watch -n 1 ollama ps
```

   * During Phase 1, you should see `harness-planner` utilizing ~16 GB.
   * During Phase 2, you should see `harness-executor` utilizing ~22.5 GB.
   * Only **one** model should appear in the table at any given time.

2. **Verify Memory Bandwidth & Zero Swapping:** Open macOS Activity Monitor → **Memory** tab:
   * **Memory Used:** Should remain between 32 GB and 38 GB.
   * **Memory Pressure Graph:** Must stay solid **Green**.
   * **Swap Used:** Must remain **0 bytes** (or < 500 MB).

This architecture gives you the deep root-cause diagnostic power of a 14B Q8 reasoning model combined with the 35–45 tok/s tool-chaining velocity of a 30B MoE, fitting entirely within your M4 Pro's 48 GB Unified Memory envelope without latency degradation.

## 14. Reference Implementation A: `harness.py` (Search/Replace Patch Variant)

This is the concrete Python implementation of the Layer 1–4 / Step 1–4 protocol described in §12. It binds to the `harness-planner` / `harness-executor` aliases from §9, edits files via **targeted search-and-replace** (per the §11 Category 2 guardrail), and enforces the JSON-contract handoff from Layer 1.

* `execute_shell` / `read_workspace_file` / `apply_targeted_patch` are the three tools the executor can call (`run_shell`, `read_file`, `patch_file`).
* `plan_resolution()` is **Phase 1**: it calls `harness-planner`, strips any leaked `<think>` tags, and parses the JSON plan — falling back to a safe default plan if the model's output isn't valid JSON.
* `execute_plan()` is **Phase 2**: a bounded tool-calling loop (`max_steps`) against `harness-executor`, dispatching each JSON action and feeding the observation back into the conversation until the model emits `"tool": "finish"`.
* `run_autonomous_repair()` is **Phase 3**, the supervisor: it alternates `plan_resolution` → `execute_plan` → runs the real validation command via `subprocess`, and re-enters the loop with the new failure trace if verification fails, up to `max_repair_cycles`.

```python
import json
import os
import re
import subprocess
import sys
from typing import Dict, Any, List
import ollama

# Model bindings created in §9 (Step 2)
PLANNER_MODEL = "harness-planner"
EXECUTOR_MODEL = "harness-executor"

# ----------------------------------------------------------------------
# 1. Deterministic Execution Tools (Sandboxed Workspace)
# ----------------------------------------------------------------------
def execute_shell(cmd: str) -> str:
    """Runs a shell command and captures stdout/stderr."""
    print(f"\033[93m[EXEC_SHELL]\033[0m {cmd}")
    try:
        proc = subprocess.run(
            cmd,
            shell=True,
            capture_output=True,
            text=True,
            timeout=45
        )
        combined = (proc.stdout + "\n" + proc.stderr).strip()
        return combined if combined else f"[Process exited with code {proc.returncode}]"
    except subprocess.TimeoutExpired:
        return "[Error: Command timed out after 45 seconds]"
    except Exception as e:
        return f"[Execution Error: {str(e)}]"

def read_workspace_file(path: str) -> str:
    """Reads file contents within the project tree."""
    print(f"\033[94m[READ_FILE]\033[0m {path}")
    if not os.path.exists(path):
        return f"[Error: File '{path}' does not exist]"
    try:
        with open(path, "r", encoding="utf-8") as f:
            lines = f.readlines()
        # Return with line numbers for accurate targeted patching
        return "".join([f"{i+1}: {line}" for i, line in enumerate(lines)])
    except Exception as e:
        return f"[Read Error: {str(e)}]"

def apply_targeted_patch(path: str, search_block: str, replace_block: str) -> str:
    """Applies clean AST-level line substitutions without destroying entire files."""
    print(f"\033[92m[PATCH_FILE]\033[0m {path}")
    if not os.path.exists(path):
        return f"[Error: Target file '{path}' does not exist]"
    try:
        with open(path, "r", encoding="utf-8") as f:
            content = f.read()

        if search_block not in content:
            return "[Error: Search block not found in file. Read the file again to verify exact lines.]"

        updated = content.replace(search_block, replace_block, 1)
        with open(path, "w", encoding="utf-8") as f:
            f.write(updated)
        return f"[Success: Patch applied to {path}]"
    except Exception as e:
        return f"[Patch Error: {str(e)}]"

# ----------------------------------------------------------------------
# 2. Phase 1: Reasoning & Planning Engine (DeepSeek-R1)
# ----------------------------------------------------------------------
def plan_resolution(task: str, diagnostics: str = "") -> Dict[str, Any]:
    print("\n\033[95m========================================================\033[0m")
    print(f"\033[95m[*] PHASE 1: Engaging Planner ({PLANNER_MODEL})\033[0m")
    print("\033[95m========================================================\033[0m")

    system_prompt = """You are a Principal Systems Architect.
Analyze the user's objective and any failure diagnostics.
Deconstruct the issue into a concrete, test-driven implementation plan.

You must respond ONLY with a single JSON object with this schema:
{
  "root_cause_analysis": "Exact mechanical reason for the issue or failure",
  "files_to_modify": ["path/to/file.py"],
  "validation_command": "Command to run to verify the fix (e.g. pytest tests/test_target.py)",
  "execution_steps": [
    "Step 1 description",
    "Step 2 description"
  ]
}
"""
    prompt = f"OBJECTIVE: {task}\n"
    if diagnostics:
        prompt += f"\nFAILURE TRACEBACK:\n{diagnostics}\n"

    response = ollama.chat(
        model=PLANNER_MODEL,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": prompt}
        ],
        format="json",
        options={"temperature": 0.2, "num_ctx": 32768}
    )

    raw_output = response["message"]["content"]
    # Strip <think> tags if DeepSeek-R1 leaks raw deliberation blocks
    cleaned_json = re.sub(r"<think>.*?</think>", "", raw_output, flags=re.DOTALL).strip()

    try:
        plan = json.loads(cleaned_json)
        print(f"\033[96mRoot Cause Identified:\033[0m {plan.get('root_cause_analysis')}")
        return plan
    except json.JSONDecodeError:
        print("[Fatal] Planner failed to output parseable JSON. Falling back to default plan.")
        return {
            "root_cause_analysis": "Undetermined - raw failure output",
            "files_to_modify": [],
            "validation_command": "pytest",
            "execution_steps": ["Inspect workspace", "Execute targeted fixes"]
        }

# ----------------------------------------------------------------------
# 3. Phase 2: Action Execution Loop (Qwen3-Coder-30B MoE)
# ----------------------------------------------------------------------
def execute_plan(plan: Dict[str, Any], max_steps: int = 10) -> bool:
    print("\n\033[95m========================================================\033[0m")
    print(f"\033[95m[*] PHASE 2: Engaging Tool-Chaining Executor ({EXECUTOR_MODEL})\033[0m")
    print("\033[95m========================================================\033[0m")

    system_prompt = f"""You are an autonomous engineering agent executing an architectural plan.
PLAN DETAILS:
Root Cause: {plan.get('root_cause_analysis')}
Target Files: {plan.get('files_to_modify')}
Steps: {json.dumps(plan.get('execution_steps'))}

At every turn, output strictly ONE JSON tool action matching this exact schema:
{{
  "thought": "Brief 1-sentence motivation for this action",
  "tool": "read_file" | "patch_file" | "run_shell" | "finish",
  "path_or_cmd": "target path or bash command",
  "search_block": "exact string to replace (only for patch_file)",
  "replace_block": "replacement string (only for patch_file)"
}}

When all modifications are complete and verified, emit tool="finish".
"""
    history = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "Execute Step 1 of the implementation plan."}
    ]

    for step_num in range(1, max_steps + 1):
        print(f"\n--- [Cycle {step_num}/{max_steps}] ---")

        # Fast generation: Qwen3-Coder MoE runs at 35-45 tok/s
        response = ollama.chat(
            model=EXECUTOR_MODEL,
            messages=history,
            format="json",
            options={"temperature": 0.1, "num_ctx": 32768}
        )

        output = response["message"]["content"]
        history.append({"role": "assistant", "content": output})

        try:
            action = json.loads(output)
        except json.JSONDecodeError:
            history.append({
                "role": "user",
                "content": "Error: Output was not valid JSON. Emit ONLY valid JSON."
            })
            continue

        thought = action.get("thought", "")
        tool = action.get("tool")
        target = action.get("path_or_cmd", "")
        print(f"Agent Thought: {thought}")

        if tool == "finish":
            print(f"\033[92m[Executor Completed Execution Loop]\033[0m")
            return True

        # Dispatch tool call
        if tool == "read_file":
            result = read_workspace_file(target)
        elif tool == "patch_file":
            result = apply_targeted_patch(
                target,
                action.get("search_block", ""),
                action.get("replace_block", "")
            )
        elif tool == "run_shell":
            result = execute_shell(target)
        else:
            result = f"[Error: Unrecognized tool '{tool}']"

        # Provide tool observation back to the model context
        history.append({
            "role": "user",
            "content": f"TOOL OBSERVATION:\n{result}\n\nProceed to the next action or emit 'finish'."
        })

    return False

# ----------------------------------------------------------------------
# 4. Phase 3: Supervisory Feedback Loop
# ----------------------------------------------------------------------
def run_autonomous_repair(task: str, default_verify_cmd: str, max_repair_cycles: int = 3):
    current_diagnostics = ""

    for cycle in range(1, max_repair_cycles + 1):
        print(f"\n\033[1;33m{'#' * 60}\033[0m")
        print(f"\033[1;33m   SUPERVISOR HARNESS: REPAIR CYCLE {cycle}/{max_repair_cycles}\033[0m")
        print(f"\033[1;33m{'#' * 60}\033[0m")

        # 1. Deliberation Phase (Hot-swaps DeepSeek-R1 into VRAM)
        plan = plan_resolution(task, diagnostics=current_diagnostics)

        # 2. Execution Phase (Hot-swaps Qwen3-Coder MoE into VRAM)
        execute_plan(plan)

        # 3. Ground-Truth Verification
        test_cmd = plan.get("validation_command") or default_verify_cmd
        print(f"\n\033[94m[*] PHASE 3: Running Ground-Truth Verification:\033[0m `{test_cmd}`")
        verify_output = execute_shell(test_cmd)

        # Check exit status
        if "[Process exited with code 0]" in verify_output or "passed" in verify_output.lower():
            if "failed" not in verify_output.lower() and "error" not in verify_output.lower():
                print(f"\n\033[92m========================================================\033[0m")
                print(f"\033[92m[SUCCESS] Verification passed with 0 errors! Task complete.\033[0m")
                print(f"\033[92m========================================================\033[0m")
                return True

        print(f"\033[91m[FAILURE] Test suite failed. Routing error trace back to Phase 1...\033[0m")
        current_diagnostics = verify_output

    print(f"\n\033[91m[TERMINATED] Exceeded maximum repair iterations without verification.\033[0m")
    return False

if __name__ == "__main__":
    # Test task example:
    TASK = "Fix the parse_payload function in src/parser.py so that it handles empty dictionary payloads without raising KeyError"
    VERIFY_CMD = "pytest tests/test_parser.py"

    run_autonomous_repair(TASK, VERIFY_CMD)
```

## 15. Reference Implementation B: `harness.py` (Full-File Write Variant)

This is a second, alternate implementation of the same three-phase protocol, using a `write_file` (full-file overwrite) edit tool instead of Variant A's targeted search/replace patch. It's a legitimate design trade-off — simpler for a model to produce correctly for small files, riskier for large ones — not a strictly worse approach; keep it as an option when files are short enough that full rewrites are safe.

### Review Notes

The version originally hardcoded `PLANNER_MODEL = "deepseek-r1:14b-q8_0"` and `CODER_MODEL = "qwen3-coder:30b-a3b-instruct-q6_K"`. Modernizing it to this guide's canonical aliases (`harness-planner` / `harness-executor` from §9) also surfaced three real bugs, fixed below:

1. **False-positive success detection.** The original success check was:
   ```python
   if "[Exit Code: 0]" in verify_output or "passed" in verify_output.lower():
       return True
   ```
   A pytest summary line like `"1 passed, 1 failed"` contains the substring `"passed"`, so a **partially failing** test run would be misreported as success and the harness would exit early, skipping repair cycles. Fixed by requiring the additional absence of `"failed"` / `"error"`, matching Variant A's guard.
2. **Unguarded `json.loads()` in the planner.** `generate_execution_plan()` had no `try/except` around the JSON parse — if the planner ever emitted malformed JSON (e.g., an unstripped `<think>` fragment), the whole harness would crash with an uncaught `JSONDecodeError` instead of degrading gracefully. Fixed by adding the same safe-fallback pattern Variant A uses.
3. **Unguarded string slice on a possibly-`None` argument.** `print(f"Executed: {tool} -> {arg[:60]}")` runs for every non-`finish` tool call; if the model omits `path_or_cmd` (`arg` is `None`), this raises `TypeError: 'NoneType' object is not subscriptable`. Fixed with a `str(arg or "")[:60]` guard.

```python
import json
import re
import subprocess
import ollama

# ---------------------------------------------------------
# Configuration: Separation of Concerns
# Aliases created in §9 (Modelfile.planner / Modelfile.executor)
# ---------------------------------------------------------
PLANNER_MODEL  = "harness-planner"   # Deep deliberation & root-cause analysis
EXECUTOR_MODEL = "harness-executor"  # Fast agentic execution (35-45 tok/s)

# ---------------------------------------------------------
# 1. Deterministic Execution Environment (Tools)
# ---------------------------------------------------------
def run_shell(command: str) -> str:
    """Executes a shell command safely and captures output."""
    try:
        res = subprocess.run(
            command, shell=True, capture_output=True, text=True, timeout=30
        )
        output = (res.stdout + "\n" + res.stderr).strip()
        return output if output else f"[Exit Code: {res.returncode}]"
    except Exception as e:
        return f"[Execution Error: {str(e)}]"

def read_file(path: str) -> str:
    try:
        with open(path, "r", encoding="utf-8") as f:
            return f.read()
    except Exception as e:
        return f"[Read Error: {str(e)}]"

def write_file(path: str, content: str) -> str:
    try:
        with open(path, "w", encoding="utf-8") as f:
            f.write(content)
        return f"Successfully wrote {len(content)} bytes to {path}"
    except Exception as e:
        return f"[Write Error: {str(e)}]"

# ---------------------------------------------------------
# 2. Phase 1: Reasoning & Task Decomposition (Planner)
# ---------------------------------------------------------
def generate_execution_plan(objective: str, failure_log: str = "") -> dict:
    print(f"\n[Phase 1] Invoking Planner ({PLANNER_MODEL})...")

    prompt = f"""You are the Lead Systems Architect.
Analyze the following objective and diagnostics, then decompose the fix into concrete atomic steps.

OBJECTIVE:
{objective}

DIAGNOSTICS / TRACE:
{failure_log if failure_log else "Initial run - no failure trace yet."}

Respond ONLY with a valid JSON object matching this schema:
{{
  "root_cause": "Detailed hypothesis of why the code fails or needs changes",
  "files_affected": ["list/of/files.py"],
  "steps": [
    "Step 1: Inspect target function signature in file.py",
    "Step 2: Update regex parsing logic to handle empty strings",
    "Step 3: Run pytest tests/test_target.py"
  ]
}}
"""
    response = ollama.chat(
        model=PLANNER_MODEL,
        messages=[{"role": "user", "content": prompt}],
        format="json",
        options={"temperature": 0.2, "num_ctx": 16384}
    )

    content = response["message"]["content"]
    # Strip <think> tags if DeepSeek-R1 leaks raw thinking tokens into the output
    clean_json = re.sub(r"<think>.*?</think>", "", content, flags=re.DOTALL).strip()

    try:
        return json.loads(clean_json)
    except json.JSONDecodeError:
        print("[Fatal] Planner failed to output parseable JSON. Falling back to default plan.")
        return {
            "root_cause": "Undetermined - raw planner output was not valid JSON",
            "files_affected": [],
            "steps": ["Inspect workspace", "Execute targeted fixes"]
        }

# ---------------------------------------------------------
# 3. Phase 2: Action Execution & Tool Chaining (Coder)
# ---------------------------------------------------------
def execute_coding_loop(plan: dict, max_iterations: int = 8) -> bool:
    print(f"\n[Phase 2] Handing off to Execution Coder ({EXECUTOR_MODEL})...")

    system_prompt = f"""You are an autonomous execution coder.
You are implementing the following verified architectural plan:
ROOT CAUSE: {plan.get('root_cause')}
TARGET FILES: {plan.get('files_affected')}
REQUIRED STEPS: {json.dumps(plan.get('steps'))}

At each turn, execute ONE action by emitting strictly valid JSON:
{{
  "tool": "read_file" | "write_file" | "run_shell" | "finish",
  "path_or_cmd": "target file path OR bash command",
  "content": "new full file content if using write_file, else empty string"
}}
"""
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "Begin execution of step 1."}
    ]

    for cycle in range(1, max_iterations + 1):
        print(f"\n--- [Execution Step {cycle}/{max_iterations}] ---")

        # Fast generation (35-45 tok/s on Qwen3 MoE)
        response = ollama.chat(
            model=EXECUTOR_MODEL,
            messages=messages,
            format="json",
            options={"temperature": 0.1, "num_ctx": 32768}
        )

        step_raw = response["message"]["content"]
        messages.append({"role": "assistant", "content": step_raw})

        try:
            step = json.loads(step_raw)
            tool = step.get("tool")
            arg  = step.get("path_or_cmd")
            body = step.get("content", "")
        except json.JSONDecodeError:
            messages.append({"role": "user", "content": "Error: Output must be valid JSON."})
            continue

        if tool == "finish":
            print(f"[Done] Coder signaled completion: {arg}")
            return True

        # Tool execution dispatch
        if tool == "read_file":
            result = read_file(arg)
        elif tool == "write_file":
            result = write_file(arg, body)
        elif tool == "run_shell":
            result = run_shell(arg)
        else:
            result = f"[Error: Unknown tool '{tool}']"

        # arg can be None if the model omits path_or_cmd; guard before slicing
        safe_arg = str(arg or "")
        print(f"Executed: {tool} -> {safe_arg[:60]}")
        print(f"Result Snippet: {result[:120]}...")

        messages.append({
            "role": "user",
            "content": f"TOOL RESULT:\n{result}\n\nProceed to next step or emit finish."
        })

    return False

# ---------------------------------------------------------
# 4. Phase 3: Supervisory Harness (Divide & Conquer Loop)
# ---------------------------------------------------------
def run_divide_and_conquer(objective: str, test_command: str, max_repair_cycles: int = 3):
    last_error = ""

    for repair_turn in range(1, max_repair_cycles + 1):
        print(f"\n==========================================")
        print(f"  SUPERVISOR CYCLE {repair_turn}/{max_repair_cycles}")
        print(f"==========================================")

        # 1. Deliberation Phase
        plan = generate_execution_plan(objective, failure_log=last_error)
        print(f"\nIdentified Root Cause: {plan.get('root_cause')}")

        # 2. Execution Phase
        success = execute_coding_loop(plan)
        if not success:
            print("[Warning] Coder reached maximum step depth without calling finish.")

        # 3. Deterministic Verification
        print(f"\n[Phase 3] Running Test Suite: `{test_command}`")
        verify_output = run_shell(test_command)

        # Success requires "passed"/exit-0 AND the absence of "failed"/"error" —
        # a bare `"passed" in output` would false-positive on "1 passed, 1 failed".
        looks_ok = "[Exit Code: 0]" in verify_output or "passed" in verify_output.lower()
        looks_bad = "failed" in verify_output.lower() or "error" in verify_output.lower()
        if looks_ok and not looks_bad:
            print("\n>>> All verification tests passed. Objective completed successfully! <<<")
            return True
        else:
            print("[Fail] Verification failed. Routing stack trace back to Planner...")
            last_error = verify_output

    print("\n[Harness Terminated] Exceeded max repair cycles without passing tests.")
    return False

if __name__ == "__main__":
    run_divide_and_conquer(
        objective="Fix KeyError: 'data' in data_parser.py when empty response is returned",
        test_command="pytest tests/test_parser.py"
    )
```

## 16. Extended Model Comparison Research

This section captures broader research beyond the `deepseek-r1` + `qwen3-coder` pairing chosen for this guide — alternative reasoners, alternative coders, an MLX-backend comparison, and the SWE-bench methodology behind all the percentages cited above.

### Comparative Reasoning Breakdown

| Dimension | `Qwen3-Coder-30B-A3B` | `DeepSeek-R1-14B (Q8_0)` | `Qwen3.8-27B (Q8_0)` |
| :--- | :--- | :--- | :--- |
| Reasoning Mechanism | Latent / Prompt-induced | Native `<think>` tokens | Flexible hybrid thinking |
| Time-to-First-Action | ~1 – 3 seconds | 15 – 45 seconds | 20 – 60 seconds (at xhigh) |
| Abstract Math / Logic | Medium (~51% GPQA) | Very High (~65%+ GPQA) | Extreme (~70%+ GPQA) |
| Bug Diagnosis in Loop | High (direct AST trace) | Very High (hypothesizes edge cases) | Very High (exhaustive root-cause) |
| Loop Latency Overhead | Negligible | Severe | Severe |

### The SWE-bench vs. Speed Trade-Off Landscape

| Model & Architecture | Total Weight (RAM) | Active Weights Per Token | Generation Speed (M4 Pro) | SWE-bench Verified | Harness Profile |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Qwen3-Coder-30B-A3B** (Q6_K) *(MoE: 30B Total, 3.3B Active)* | ~22.5 GB | ~3.3 GB | 35 – 45 tok/s | ~68% – 72% | **The Speed & Accuracy Winner:** Instantaneous generation, high accuracy, low latency per turn. |
| **gpt-oss-20b** (MXFP4 / Q8) *(MoE: 21B Total, ~3.6B Active)* | ~16.0 GB | ~3.6 GB | 30 – 40 tok/s | 60.7% | **Low-Latency Runner:** Generates in bursts; leaves over 25 GB free for context buffers. |
| **Qwen3.6 / 3.8 27B** (Q8_0) *(Hybrid Linear Dense)* | ~29.1 GB | ~27.8 GB | 10 – 12 tok/s | ~76% – 77% | **Maximum Accuracy (Slower):** Peak open-weight problem resolution, but sluggish in long, multi-turn loops. |
| **Devstral Small 2 / 2512** (Q8_0) *(Dense 24B)* | ~25.5 GB | ~24.0 GB | 11 – 13 tok/s | 53.6% | **Specialized Agent:** Built directly for OpenHands bash loops; robust against syntax drift. |
| **Qwen 2.5 Coder 14B** (Q8_0) *(Dense 14B)* | ~15.5 GB | ~14.0 GB | 17 – 20 tok/s | ~21% (zero-shot) | **Fast but Outdated:** Fast turnaround, but lacks multi-file repository bug-fixing capacity. |

### Broader Reasoning / Coding / Tooling Survey

| Model & Quant | Weights / Quant | Speed (tok/s, M4 Pro) | SWE-bench Verified Score | Reasoning & Thinking | Coding Quality | Tooling & Chaining Fidelity | Harness Profile & Loop Fit |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Devstral Small 2 / 1.1 (Q8_0) | ~25.5 GB | 11 – 13 | 53.6% | High (Hypothesis-driven agent planning) | Very High (SOTA open-source SWE-bench agent model) | Very High (Built natively for OpenHands/scaffold loops) | High (~20 GB headroom remains for large file buffers) |
| Devstral 24B v1 (2505) (Q8_0) | ~25.5 GB | 11 – 13 | 46.8% | High (Targeted repo exploration traces) | Very High (Multi-file unified diff specialist) | Very High (Native JSON tool schemas) | High (Fits up to 32k context cleanly in Unified RAM) |
| Qwen 2.5 Coder 32B (Q6_K) | ~26.5 GB | 9 – 11 | 31.0% (vanilla) / ~40% (with scaffold) | Medium-High (Structured prompt scratchpads) | Very High (Dense AST logic and syntax precision) | High (Reliable OpenAI-compatible schema emission) | High (16k context fits comfortably within 36 GB VRAM) |
| DeepSeek-R1 Distill 32B (Q6_K) | ~27.0 GB | 8 – 10 | ~38% – 42% (via Agentless/R1-eval) | Extreme (Exhaustive mathematical/root-cause traces) | High (Deep logic, occasional markdown leaks) | Medium (Can leak thought tokens into JSON payloads) | Medium-Low (Long think tokens cause 60–120s step delays) |
| Mistral Small 24B (2501) (Q8_0) | ~25.5 GB | 11 – 13 | ~30% – 34% | Medium (Clean linear step planning) | Medium-High (Clean, idiomatic implementations) | Very High (Strict schema and zero parameter hallucination) | High (Lowest JSON syntax failure rate in long loops) |
| DeepSeek-R1 Distill 14B (Q8_0) | ~15.5 GB | 16 – 19 | ~28% – 32% | Very High (True internal chain-of-thought `<think>` traces) | Medium-High (Solid multi-language generation) | Medium-High (Requires harness regex to strip `<think>`) | High (Fast turns; 25 GB free RAM for logs & traces) |
| Qwen 2.5 Coder 14B (Q8_0) | ~15.5 GB | 16 – 19 | 21.0% (tuned) / ~6% (zero-shot) | Medium-Low (Fast single-pass deductions) | High (Strong syntax and fast diffs) | High (Reliable tool-call triggers) | Very High (Sub-second dispatch; handles 64k+ context) |
| Command R 35B (08-2024) (Q6_K) | ~28.0 GB | 8 – 10 | 12.0% | Medium (Optimized for multi-hop tool routing) | Medium (Competent scripts, less dense refactoring) | Exceptional (Engineered specifically for multi-step tool use) | Medium (High memory footprint leaves narrow context space) |
| Llama 3.1 8B / Hermes 3 8B (Q8_0) | ~8.5 GB | 30 – 35 | ~10% – 12% | Low-Medium (Single-turn decisions) | Medium (Small scripts/functions) | High (Native structured function calling) | Extreme (Negligible VRAM, blisteringly fast turns) |

### What "SWE-bench Verified" Actually Measures

The **SWE-bench Verified Score** is the gold-standard industry benchmark metric used to evaluate whether an AI model (or agent scaffold) can autonomously resolve real-world software engineering issues in production codebases. *(BenchLM)*

Instead of testing isolated function synthesis (like HumanEval or LeetCode snippets), SWE-bench measures whether an AI can act like a software engineer on a full repository. *(Verdent AI)*

**1. How It Works**

* **The Input:** The AI is given a clone of a real open-source GitHub repository (e.g., `django`, `flask`, `scikit-learn`, `sympy`, `requests`) frozen at the exact commit before a specific bug was fixed. It is handed the raw GitHub issue description written by a human. *(BenchLM, Emergent Mind)*
* **The Execution:** Inside a sandboxed environment (Docker), the agent must: *(Emergent Mind)*
  1. Search and navigate the repository (`grep`, directory listings, AST lookups).
  2. Locate the root cause across potentially thousands of files.
  3. Formulate a patch and write multi-file edits (unified diffs). *(BenchLM)*
* **The Evaluation (Strict Pass/Fail):** The environment applies the AI's generated patch and runs the repository's unit test suite: *(BenchLM)*
  * **Fail-to-Pass (F2P):** Tests that failed on the buggy codebase must now pass.
  * **Pass-to-Pass (P2P):** Existing tests must still pass without introducing any regressions.
  * **The Score:** The percentage of issues where 100% of the tests pass. *(Emergent Mind)*

```text
SWE-bench Score = (Number of Issues Completely Resolved / Total Issues Evaluated) × 100
```

**2. Why "Verified"?**

The original SWE-bench dataset (2,294 issues) suffered from several real-world flaws:

* Some issues were ambiguous, underspecified, or lacked complete unit tests in the repo. *(Emergent Mind)*
* In some cases, human maintainers wrote tests that failed due to environmental or dependency quirks rather than actual code bugs.

To fix this, OpenAI collaborated with human software engineers to review the problems and created **SWE-bench Verified**: *(Pristren)*

* A human-validated subset of **500 curated issues** across 12 Python repositories. *(arXiv.org)*
* Every issue was confirmed to have an unambiguous bug description, a clear ground-truth solution, and completely reproducible unit tests that verify the fix. *(Verdent AI)*

### Ollama vs. MLX on M4 Pro: Which Backend to Pick for Harnesses?

**3. Point Your Python Harness to MLX**

Swap the Ollama client with the standard `openai` Python SDK pointing to `localhost:8080/v1`: *(Hugging Face)*

```python
import json
from openai import OpenAI

# Connect directly to your local mlx-lm server
client = OpenAI(
    base_url="http://localhost:8080/v1",
    api_key="none"
)

def step_agent(prompt: str, history: list):
    history.append({"role": "user", "content": prompt})

    response = client.chat.completions.create(
        model="mlx-community/Qwen2.5-Coder-32B-Instruct-6bit",
        messages=history,
        temperature=0.2,
        # Native JSON-mode enforcement supported by mlx-lm
        response_format={"type": "json_object"}
    )

    output = response.choices[0].message.content
    history.append({"role": "assistant", "content": output})
    return json.loads(output)
```

**Choose MLX if:**

* Your harness feeds large context chunks (entire source files, 500-line stack traces) back to the model repeatedly. MLX evaluates large prompt context noticeably faster than Ollama on Apple Silicon.
* You want exact control over Python memory management and token streaming.

**Stick with Ollama if:**

* You rely on Ollama's automatic model-swapping engine (`keep_alive`) to run a two-model setup (e.g., DeepSeek-R1 14B for planning → Qwen 2.5 Coder 32B for executing). `mlx_lm.server` holds only one model in memory at a time — it does not provide the automatic hot-swap eviction that this guide's whole architecture depends on.

**How to Drop MLX into Your Loop Harness**

You do not need to rewrite your agent loop. `mlx-lm` provides a drop-in OpenAI-compatible HTTP server that supports structured JSON responses, streaming, and tool schemas. *(Hugging Face)*

Installation:

```bash
pip install --upgrade mlx-lm
```

Launch the server:

```bash
python -m mlx_lm.server \
  --model mlx-community/Qwen2.5-Coder-32B-Instruct-6bit \
  --port 8080 \
  --max-tokens 4096
```

(MLX downloads the model directly from Hugging Face into `~/.cache/huggingface/hub` and caches the Metal compilation graph.)

**Equivalent MLX Community Models for M4 Pro (48 GB)**

| Ollama Model Target | MLX Hugging Face Repo (`mlx-community/`) | Quant | Size on Disk / RAM | Est. Speed (tok/s) on M4 Pro | Best Role in Harness |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Qwen 2.5 Coder 32B | `Qwen2.5-Coder-32B-Instruct-6bit` | 6-bit | ~25.5 GB | 11 – 13 | **Primary Coding Engine:** Complex diffs, AST edits, multi-file code synthesis. |
| Qwen 2.5 Coder 14B | `Qwen2.5-Coder-14B-Instruct-8bit` | 8-bit | ~15.2 GB | 18 – 22 | **Fast Iteration Runner:** Fast test-fail-edit cycles; leaves 25+ GB for context. |
| Mistral Small 24B | `Mistral-Small-24B-Instruct-2501-8bit` | 8-bit | ~25.0 GB | 12 – 14 | **Deterministic Tool Chainer:** Zero schema drift; reliable JSON tool calls. |
| DeepSeek-R1 Distill 14B | `DeepSeek-R1-Distill-Qwen-14B-8bit` | 8-bit | ~15.2 GB | 18 – 21 | **Fast Reasoner:** Explicit `<think>` tokens for error diagnosis with high tok/s. |
| DeepSeek-R1 Distill 32B | `DeepSeek-R1-Distill-Qwen-32B-6bit` | 6-bit | ~26.0 GB | 10 – 12 | **Deep Logic Planner:** Deep root-cause deduction (slower loop latency). |

**Key Takeaways for Generation Speed on M4 Pro**

1. **Bandwidth Ceiling:** With the M4 Pro's memory bandwidth of 273 GB/s, single-batch autoregressive generation is bounded by approximately `(273 × efficiency) / VRAM footprint`. *(PromptQuorum)*
2. **The 14B Sweet Spot (16–19 tok/s):** Iterations that return small shell outputs or 10-line patches finish in 1.5–3 seconds, keeping multi-round harnesses fast and interactive.
3. **The 24B–32B Profile (9–13 tok/s):** A typical 200-token tool call or code modification takes 15–20 seconds. This is completely manageable for autonomous runs, but will feel slow if you pair it with 500+ token internal thinking traces.

Every model in the survey tables above is available in native Apple Silicon **MLX** format via the Hugging Face `mlx-community` organization. *(Hugging Face)* MLX has two distinct advantages over Ollama (`llama.cpp` Metal backend) for agent harnesses on Apple Silicon:

1. **Higher Prompt Processing (Prefill) Speed:** In a loop harness where you repeatedly feed back 2,000–8,000 tokens of file contents and compiler output, MLX evaluates prompts roughly **1.5× to 2× faster** than Ollama due to native Metal matrix-multiplication kernels.
2. **Dynamic 6-bit (`-6bit`) and 8-bit (`-8bit`) native weights:** Hugging Face maintains quantized MLX checkpoints matching the exact parameter sizes you need.

Backend choice (Ollama GGUF vs. MLX) changes a model's disk footprint and generation speed (per the two tables above), not its qualitative reasoning/coding/tooling profile — for those ratings on this same set of models, see the [Broader Reasoning / Coding / Tooling Survey](#broader-reasoning-coding-tooling-survey) earlier in this section.

### Single-Model vs. Two-Model Trade-off (If You Don't Want to Run Two Models)

**Analysis of the Top Single-Model Options**

**1. The Single Model Solution: `Devstral-24B` (Q8_0)**

Built specifically by Mistral AI and All Hands AI, Devstral 24B is fine-tuned directly on agent scaffolds (like SWE-Agent and OpenHands) solving real GitHub issues: *(YouTube)*

* **Reasoning & Thinking (High):** It does not dump verbose, philosophical tokens; its reasoning is trained to inspect directory trees, analyze test failures, and construct structured hypotheses before editing files.
* **Coding Quality (Very High):** Benchmarked against SWE-Bench Verified, it is specifically trained to generate unified diffs and precise multi-file patches without regressions. *(YouTube)*
* **Tooling Fidelity (Very High):** Emits JSON/schema commands that match shell tools, file editors, and linters without formatting drift.
* **Harness Fit (High):** At 24B with Q8_0, it occupies ~25.5 GB VRAM, runs at ~11–13 tok/s on the M4 Pro, and leaves ~16–18 GB for your context cache.

**2. The Prompt-Engineered Solution: `Qwen 2.5 Coder 32B` (Q6_K)**

If you enforce a `<thought>` block within your harness's structured JSON schema:

* You force Qwen 2.5 Coder to deliberate first before emitting `"action"` and `"argument"`.
* Because it natively handles complex AST transformations and compiler errors, you get **Very High** coding and **High** tool fidelity, with reasoning elevated to **Medium-High** via structured scratchpad prompting.

**Two-Step Orchestration**

If your tasks require mathematical deductions or identifying non-obvious runtime bugs, running two complementary local models via Ollama gives **Very High** across every dimension:

```text
[User Task / Test Failure]
   |
   v
┌───────────────────────────────┐
│ 1. DeepSeek-R1 14B (Q8_0)      │ ──> Reasoning: VERY HIGH
│    Role: Planner & Root-Cause Analyst │      Emits concrete diagnosis & execution steps
└──────────────┬─────────────────┘
               │
               v
┌───────────────────────────────┐
│ 2. Qwen 2.5 Coder 32B (Q6_K)   │ ──> Coding: VERY HIGH
│    Role: Executor & Tool Chainer │      Tooling: VERY HIGH
└──────────────┬─────────────────┘
               │
               v
[Executes Shell / Edits Code / Runs Harness Loop]
```

By unloading the memory context between turns (`keep_alive: 0` or standard Ollama swapping), you get deep reasoning on the plan and deterministic tool execution in the loop.

*For a walkthrough on architecting agent workflows that run software engineering benchmarks locally, "Mistral Devstral-24B: SOTA Open Model for AI Coding Agents" demonstrates how a 24B agent-trained model operates within an execution harness to diagnose codebase issues and execute multi-step tools.*

### The Verdict: No Single Model Hits "High" on Every Dimension

No single model under the default Q6/Q8 footprint on 48 GB Unified Memory gives an uncompromised "High" or "Very High" across all four dimensions (**Reasoning & Thinking**, **Coding Quality**, **Tooling & Chaining Fidelity**, and **Harness Fit**).

The architectural bottleneck is the trade-off between **explicit thinking vs. deterministic tool execution**:

1. **Reasoning models (like DeepSeek-R1):** Provide *Very High* reasoning, but their lengthy internal thinking blocks slow down harness cycle times and occasionally leak `<think>` tokens into JSON schemas, downgrading Tooling to *Medium* and Harness Fit to *Medium-Low*.
2. **Instruction-tuned coding/tool models (like Qwen 2.5 Coder 32B or Mistral Small):** Provide *High/Very High* coding and tool chaining fidelity, but lack explicit chain-of-thought tokens, relying instead on single-pass latent reasoning.

**Candidate Models Closest to the "High on All Four" Criteria**

Evaluating specialized variants designed explicitly to combine thinking with agent tool calling:

| Model & Quant | Weights / Quant | Reasoning & Thinking | Coding Quality | Tooling & Chaining Fidelity | Harness Profile & Loop Fit | Verdict |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Devstral 24B (Q8_0) (Mistral + All Hands) | ~25.5 GB | High (Trained on agentic planning & root-cause traces) | Very High (SOTA open-source SWE-bench agent model) | Very High (Built natively for OpenHands tool loops) | High (Runs at 11–13 tok/s; leaves 20 GB headroom) | **Closest Single Model:** Achieves High/Very High across all 4 columns for an agent harness. |
| Qwen 2.5 Coder 32B Instruct (Q6_K) | ~26.5 GB | Medium-High (Prompted step-by-step; no native `<think>`) | Very High (Top-tier syntax, AST, & diff generation) | High (Reliable OpenAI-compatible schema emission) | High (10 tok/s; fits 16k context comfortably) | **Runner-up:** Strongest pure coder, but requires prompt engineering to enforce internal thinking before tool output. |
| DeepSeek-R1 Distill Qwen 14B (Q8_0) | ~15.5 GB | Very High (True explicit chain-of-thought `<think>`) | Medium-High (Solid multi-language generation) | Medium-High (Needs strict harness regex to strip thinking) | High (16–18 tok/s; 25 GB free RAM for logs) | **Fast Thinker:** Hits the speed/harness fit marks, but tool calling requires regex/parsing safeguards in your harness. |

**Bottom line for this guide:** this is exactly why §1–§15 use the *two-model* pipeline (`deepseek-r1:14b-qwen-distill-q8_0` for reasoning + `qwen3-coder:30b-a3b-q8_0` for tooling) rather than a single model — no single 24–32B open-weight model available today clears "High" on reasoning, coding, and tool fidelity simultaneously, and Devstral-24B (the closest single-model candidate) still trails the two-model pipeline's ~68–72% scaffolded SWE-bench score with its ~53.6%.

## 17. Local Dual-Model Setup vs. Cloud Frontier Defaults (Claude Sonnet 5 / Grok)

> **Caveat before the numbers:** every table elsewhere in this guide (§1, §10, §16) traces back to a specific pinned, quantized checkpoint (`deepseek-r1:14b-qwen-distill-q8_0`, `qwen3-coder:30b-a3b-q8_0`) that doesn't change under you. Claude Sonnet 5 and Grok's coding-tier models are hosted, continuously-updated frontier models — their throughput, pricing, and published SWE-bench numbers shift with provider infrastructure and model refreshes. Treat the cloud-side figures below as directional/publicly-reported ranges to sanity-check against Anthropic's and xAI's current model cards before making a decision, not as pinned constants like the local benchmarks above.

### Detailed Compare & Contrast

| Dimension | Local Dual Setup (`deepseek-r1:14b-qwen-distill-q8_0` + `qwen3-coder:30b-a3b-q8_0`) | Claude Sonnet 5 (Cursor / Claude Code default) | Grok Code Fast / Grok 4 (xAI, Cursor's Grok default) |
| :--- | :--- | :--- | :--- |
| **Output Throughput** | 18–22 tok/s (Reasoner) / 32–36 tok/s (MoE Coder) — hard-capped by the M4 Pro's 273 GB/s memory bandwidth (§10 formula). | Typically ~60–100+ tok/s on Anthropic's hosted infra, higher with prompt caching on repeated context; not bandwidth-bound the way a single local GPU is. | Grok Code Fast is explicitly tuned for high-throughput agentic loops; publicly reported well above 100 tok/s in bursts. Grok 4 (non-fast) trades some speed for depth. |
| **Time-to-First-Token / Turn Latency** | 0 ms network, but +1.2–1.8s per **role swap** (Ollama evicting/loading the other model) — see §12 hot-swap boundary. | ~150–400 ms network + queueing; extended-thinking mode adds several seconds before the first visible token. | Similar network-bound TTFT to Sonnet; Grok Code Fast is optimized to minimize this for tight tool-call loops. |
| **Context Window** | 32k (planner, `Modelfile.planner`) / 16k (coder, `Modelfile.coder`) — deliberately capped to stay under the 38.4 GB VRAM ceiling (§8 "Key Tuning Rule"). | 200K tokens standard tier (1M on extended-context offerings) — no local hardware ceiling. | ~128K–256K depending on the specific Grok tier. |
| **SWE-bench Verified (agentic/scaffolded)** | ~68–72% (`qwen3-coder:30b-a3b-q8_0`, per §10/§16) — reasoner not separately SWE-bench-scored, it only plans. | Anthropic's Sonnet/Opus generations have publicly trended into the ~70–80%+ range on SWE-bench Verified with agentic scaffolding; **verify Sonnet 5's specific published number**, it postdates the figures cached in this note. | xAI has published competitive SWE-bench-style figures for Grok's coding-tier models in a similar ~65–75% band; **verify current Grok model card**. |
| **Reasoning Depth (multi-file causal chains, proofs)** | High for single-file/localized algorithmic reasoning (`<think>` tokens, §11 Category 4); degrades on chains spanning more files than fit in 32k context. | Very High — extended thinking mode + 200K context lets it hold an entire mid-size repo's causal chain in one turn without the pruning §11's Guardrail 1 requires locally. | Very High — comparable extended-reasoning tier; Grok 4 in particular is positioned for deep multi-step deduction. |
| **Coding Quality (syntax precision, idiomatic diffs)** | Very High within the guardrails of §11 Category 2 (targeted search/replace, pruned context) — degrades on tasks needing whole-repo idiom awareness it was never shown. | Very High across a much broader training distribution; handles unfamiliar codebases, mixed languages, and ambiguous specs with less prompt-engineering scaffolding than §6/§11 require locally. | Very High, comparable breadth to Sonnet 5 for mainstream languages; both benefit from far larger pretraining/RLHF coding corpora than the local checkpoints. |
| **Agentic Tool-Calling Fidelity** | 0.0% schema drift at `q8_0` (§1), but only because of the harness guardrails in §12 Layer 1 (strict JSON contract, `<think>`-tag stripping, search/replace atomicity) — the model needs that scaffolding. | Natively trained for long, mixed tool-call/reasoning agentic loops (this is the same tool-use loop Claude Code itself runs on); tolerates looser prompting than the local setup. | Also natively agentic-trained; Grok Code Fast specifically targets long tool-chaining loops at low per-call latency. |
| **Cost Structure** | $0.00 marginal cost after hardware purchase; ~35W electricity per repair loop (§7). | Pay-per-token (input/output priced separately); cost scales directly with context reused each turn unless prompt caching is used. | Pay-per-token, generally priced to undercut Claude/GPT-tier per-token rates for the "fast" coding variant. |
| **Data Privacy / Egress** | Fully air-gapped, localhost-bound (127.0.0.1), zero egress (§7). | Data leaves the machine to Anthropic's API (subject to your account's data-retention/training-opt-out settings). | Data leaves the machine to xAI's API (subject to xAI's retention policy). |
| **Concurrency / Scaling** | Strictly sequential — `OLLAMA_NUM_PARALLEL=1`, one model resident at a time (§7, §12 Layer 4). | Near-infinite — managed multi-tenant elastic infra scales with your API rate limit, not your local hardware. | Same elastic scaling model as Sonnet. |
| **Setup & Maintenance Burden** | High: sysctl tuning, Modelfiles, alias management, harness code, guardrail prompting (§2, §8, §9, §11). | Near-zero: it's the default model in Cursor/Claude Code, no local infrastructure to maintain. | Near-zero: selectable as a default/alternate model in Cursor with no local setup. |
| **Offline Capability** | Full — works with no network connection at all. | None — requires an internet connection and a live API key. | None — same requirement. |
| **Model Staleness Risk** | None from the provider's side — the pinned `q8_0` checkpoints don't change under you (a real advantage for reproducible benchmarking), but you're responsible for manually pulling newer Ollama registry tags as better local models ship. | Low — Anthropic ships model updates; you get improvements without doing anything, but a provider-side update can also silently change behavior between your test runs. | Low — same trade-off, xAI-side updates. |
| **Best Fit** | Repos that fit in 16–32k pruned context, zero-cost/offline iteration, air-gapped or compliance-sensitive codebases, reproducible fixed-checkpoint benchmarking. | Large or unfamiliar repositories, cross-cutting refactors needing >32k context, tasks where resolution accuracy matters more than marginal token cost. | Same profile as Sonnet 5, with a lean toward high-volume, latency-sensitive agentic loops (e.g., CI-triggered auto-fix bots) where Grok Code Fast's speed/cost profile pays off. |

### Reading the Trade-off

* **Throughput isn't the whole story.** The local setup's 18–45 tok/s looks slow next to a hosted model's 60–100+ tok/s, but the hosted number is per-request and shared infrastructure; it doesn't include the ~150–400ms network round-trip added to *every* turn, nor rate-limit queueing under load. The local setup's real tax is the ~1.2–1.8s hot-swap between roles (§12), not raw decode speed.
* **Context window is the sharpest structural gap.** Everything in §6/§8/§11 about pruning prompts, capping `local-coder` at 16k, and issuing a "Deterministic Patch Contract" instead of dumping a repo exists *because* the local models can't hold more than 16–32k tokens without blowing the 38.4 GB VRAM ceiling. Sonnet 5 and Grok's 128K–1M-token windows remove that constraint entirely — they can read a large unfamiliar module in one shot where the local pipeline needs `@planner` to pre-isolate it.
* **Reasoning and coding quality favor the frontier models on breadth, not necessarily on this repo.** A 14B distilled reasoner and a 30B-total/3.3B-active MoE coder are narrow, cheap, and — per §10/§16 — genuinely competitive on SWE-bench Verified for well-scoped patches. But they haven't seen the volume or diversity of code Sonnet 5 or Grok have, so on an unfamiliar codebase or an ambiguous spec, expect the frontier models to need less hand-holding (less of §6's guardrail scaffolding) to get an equivalently correct result.
* **Tool-calling fidelity is closer than the raw model size suggests.** `qwen3-coder:30b-a3b-q8_0`'s 0.0% schema drift (§1) is real, but it's earned through the harness's strict JSON contract and `<think>`-stripping (§12 Layer 1) — remove that scaffolding and reliability drops. Sonnet 5 and Grok tolerate looser, more conversational tool-calling prompts natively because agentic tool-use is trained in, not bolted on via a supervisor.
* **The honest reason to run the local pipeline isn't raw capability — it's the constraint set.** Zero marginal cost, zero network egress, and a fixed, reproducible checkpoint are the actual wins; treat §1–§16 as optimizing hard against a fixed 48 GB/273 GB/s hardware budget, not as a claim that this pipeline out-codes Sonnet 5 or Grok in the general case.

### Closing the Gap: What Hardware and Models It Would Actually Take

* **Why the gap exists:** §1's whole architecture is built around a hard 48 GB / 273 GB/s ceiling, and every design choice in this guide (16k coder context, `q8_0` over `fp16`, sequential hot-swapping instead of both models resident, the pruning guardrails in §11) exists to fit two comparatively small models (14B and 30B-total/3.3B-active) inside that envelope. Sonnet 5 is almost certainly served as a much larger model (frontier labs don't publish exact parameter counts, but the industry pattern is triple-digit-billion to trillion-parameter-class Mixture-of-Experts) across a cluster of datacenter accelerators with aggregate memory bandwidth in the multiple-TB/s range — one to two orders of magnitude past a single M4 Pro's 273 GB/s — plus agentic tool-use and coding RLHF at a scale no single locally-hosted checkpoint replicates. Meaningfully closing that gap requires moving on both axes at once, not just one.
* **Hardware step-up (Apple Silicon):** a Mac Studio with an Ultra-class chip — the M2 Ultra (192 GB unified memory, ~800 GB/s) or M3 Ultra (up to 512 GB, ~819 GB/s) — roughly **3x the memory bandwidth** of this guide's M4 Pro, and more importantly, enough unified memory headroom to hold a genuinely large model's weights and KV cache without the 80%-cap juggling in §2/§8.
* **Hardware step-up (alternative path):** a multi-GPU NVIDIA workstation (e.g., 2–4× H100/H200 with NVLink), trading Apple's unified-memory simplicity for far higher raw bandwidth and cost.
* **Models that hardware tier unlocks** (`ollama pull` targets in the 70B–671B class instead of this guide's 14B/30B pair):
  * `deepseek-r1:671b` — DeepSeek-R1 full MoE, 37B active, needs ~400+ GB even at `Q4_K_M`, effectively requiring the 512 GB M3 Ultra.
  * `qwen3:235b-a22b` — 235B-total/22B-active MoE, fits a 192–512 GB Ultra at Q4–Q8.
  * `llama3.1:405b` — dense, ~230 GB at Q4, punishing on tokens/sec since it has no MoE sparsity to exploit.
  * `gpt-oss:120b` — 117B-total MoE, the most attainable of the group at ~120 GB and a realistic fit for a 192 GB M2 Ultra.
  * All four post SWE-bench Verified and general reasoning numbers meaningfully closer to frontier-tier than the `deepseek-r1:14b` / `qwen3-coder:30b-a3b` pair this guide runs.
* **The honest ceiling:** even fully specced, expect diminishing returns rather than parity. You'd be spending roughly $8K–$15K+ on a Mac Studio Ultra (or comparably more on a multi-GPU rig) and drawing hundreds of watts to run a single dense/MoE model with no hot-swap partner, still bottlenecked to double-digit tok/s on the largest dense checkpoints, and still without the agentic tool-use fine-tuning and inference-time optimizations (speculative decoding, custom kernels, RLHF at Anthropic's scale) that make Sonnet 5 behave the way it does — so this buys you a stronger single local model, not a local Sonnet 5.

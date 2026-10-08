# Debate Reasoning Protocol

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Protocol Version](https://img.shields.io/badge/Protocol-v2.0.0-green.svg)](./SKILL.md)
[![Compatible With](https://img.shields.io/badge/Compatible%20With-Claude%20Code%20|%20Cursor%20|%20Codex%20|%20Antigravity-orange.svg)](#installation)

> **A battle-tested adversarial debate protocol for AI agents to solve high-stakes architecture choices, tricky distributed bugs, and ambiguous technical strategy.**

When an AI model tackles a complex, ambiguous problem in a single prompt or context, it inevitably suffers from **first-token anchoring**—it generates a plausible path and spends the rest of the generation rationalizing and defending it. 

**Debate Reasoning** breaks that failure mode. It forces multiple independent agent roles, isolates their initial generation to prevent cross-contamination, cross-examines their assumptions, grounds contentious claims through empirical terminal/file verification, and delivers a rigorous, evidence-backed decision.

---

## 🔍 Why Debate Reasoning?

| Technique | How It Works | Failure Mode |
|---|---|---|
| **Simple Chain-of-Thought (CoT)** | Single linear thought path | Anchors to the first idea; defends early mistakes. |
| **Self-Consistency (Sampling)** | Multiple samples, majority voting | Groupthink; all samples can fail the same model blind spot. |
| **Tree of Thoughts (ToT)** | Branching tree search | High compute overhead; lacks adversarial falsification. |
| **Debate Reasoning (This Protocol)** | **Adversarial committee + empirical verification gate + neutral judge** | **Stress-tests assumptions, checks real files/tests, and kills false consensus.** |

---

## 🔄 The Closed-Loop Flow

```mermaid
flowchart TD
    Start([High-Stakes Technical Problem]) --> Frame[1. Neutral Problem Framing<br/>Zero solution bias + hard constraints]
    
    subgraph Phase2 [2. Blind Generation - Anti-Anchoring Quarantine]
        R1[Builder<br/>Affirmative Solution]
        R2[Skeptic<br/>Failure Modes & Risks]
        R3[Contrarian<br/>Paradigm Inversion]
        R4[Pragmatist<br/>Operational Reality]
        R5[Verifier<br/>Empirical Claims]
    end
    Frame --> Phase2
    
    subgraph Phase3 [3. Cross-Examination Matrix]
        Cross[Steelman Arguments<br/>Attack Weakest Assumptions<br/>Mandatory Concessions]
    end
    Phase2 --> Phase3
    
    subgraph Phase4 [4. Empirical Verification Sprint]
        Verify[Run diagnostic tools<br/>Inspect source files & git history<br/>Tag: VERIFIED | REFUTED | UNVERIFIED]
    end
    Phase3 --> Phase4
    
    Phase4 --> Judge[5. Neutral Adjudication<br/>Evidence-weighted rubric & synthesis]
    Judge --> Deliver([6. Actionable Verdict Delivery<br/>Decision + Rationale + Kill-Switch])
```

---

## 👥 The 5 Debater Roles

Debaters are functional roles with specific cognitive mandates:

| # | Role | Core Mandate | Key Question |
|---|---|---|---|
| 1 | **Builder** | Formulates the strongest, most direct affirmative solution. | *"What is the best way to build this right now?"* |
| 2 | **Skeptic** | Attacks load-bearing assumptions, edge cases, and failure modes. | *"Where does this fail under peak stress or partitions?"* |
| 3 | **Contrarian** | Inverts the premise; champions an orthogonal or radical approach. | *"What if the conventional premise is wrong?"* |
| 4 | **Pragmatist** | Judges delivery timeline, on-call burden, reversibility, and tech debt. | *"Is the complexity justified by business reality?"* |
| 5 | **Verifier** | Inspects code, runs terminal commands, audits documentation and benchmarks. | *"Can this claim be proven with empirical evidence?"* |

> **Effort Scaling**: Use **3 Roles** (Builder, Skeptic, Verifier) for medium decisions. Use all **5 Roles** for high-stakes, foundational decisions.

---

## 🚀 Installation

### 1. Claude Code
- **As a skill (recommended)**:
  Copy [`SKILL.md`](./SKILL.md) to `.claude/skills/debate-reasoning/SKILL.md` in your project (or `~/.claude/skills/debate-reasoning/SKILL.md` globally).
- **As always-on instructions**:
  Add [`AGENTS.md`](./AGENTS.md) content into your project's `CLAUDE.md`.

### 2. Antigravity IDE / Gemini CLI
- Copy [`SKILL.md`](./SKILL.md) to `.agents/skills/debate-reasoning/SKILL.md` or your global config root (`~/.gemini/config/skills/debate-reasoning/SKILL.md`).

### 3. Cursor / Windsurf
- Paste [`AGENTS.md`](./AGENTS.md) into `.cursorrules` or `.windsurfrules`.

### 4. Custom Multi-Agent Harness (Python / LangGraph / Swarm)
- Use the modular role prompts located in [`templates/`](./templates/):
  - [`templates/1-builder.md`](./templates/1-builder.md)
  - [`templates/2-skeptic.md`](./templates/2-skeptic.md)
  - [`templates/3-contrarian.md`](./templates/3-contrarian.md)
  - [`templates/4-pragmatist.md`](./templates/4-pragmatist.md)
  - [`templates/5-verifier.md`](./templates/5-verifier.md)
  - [`templates/judge.md`](./templates/judge.md)

---

## 📋 The 6-Step Protocol Breakdown

### Step 1: Neutral Problem Framing
The lead orchestrator establishes an objective frame with zero solution bias:
- **Problem Statement**: Concise description of the dilemma.
- **Hard Constraints**: Latency budgets, backward compatibility, team size, delivery timelines.
- **Success Metrics**: Measurable criteria (e.g. p99 latency, zero data loss, rollback ease).

### Step 2: Blind Generation (Anti-Anchoring Quarantine)
Each role generates its analysis in isolation. No role sees peer responses until its own position is locked.
Every position specifies:
- Core thesis & architecture
- Load-bearing assumptions
- Known vulnerabilities
- Confidence level
- Explicit **Falsification Trigger**: *"I will abandon this position if [measurable event occurs]."*

### Step 3: Cross-Examination Matrix
Roles review peer positions:
- **Steelmanning**: Attack the strongest version of the argument.
- **Assumption Targeting**: Focus on load-bearing pillars, not superficial semantics.
- **Mandatory Concession**: If a peer reveals an invalid assumption, the role must openly concede and update its thesis.
- **Round Cap**: Maximum 2 rounds to prevent endless circular debates.

### Step 4: Empirical Verification Sprint
The **Verifier** executes real diagnostic tools:
- Checks source code files and line numbers.
- Runs terminal commands, benchmark tests, and linters.
- Tags disputed assertions with epistemic markers:
  - `[VERIFIED: <evidence>]`
  - `[REFUTED: <counter-evidence>]`
  - `[UNVERIFIED: <reason>]`
  - `[ESTIMATE: <calculation>]`

> **The Verifier Veto**: A verified empirical fact always overrides a theoretical argument.

### Step 5: Neutral Adjudication & Synthesis
The Lead Judge evaluates the debate against a weighted rubric:
- Empirical Grounding (40%)
- Resilience Under Attack (25%)
- Constraint Adherence (20%)
- Reversibility & Blast Radius (15%)

### Step 6: Deliverable Verdict
Deliver an executive, actionable report:
```markdown
## Final Verdict: [Decision Title]

### The Decision
[Clear statement of what action to take]

### Decisive Factors (Why This Won)
1. [Factor 1 - Backed by verified evidence]
2. [Factor 2 - Conceded by opposing roles]
3. [Factor 3 - Lowest operational blast radius]

### Rejected Alternatives & Fatal Flaws
- [Alternative A]: Rejected because [refuted assumption or failure mode].
- [Alternative B]: Rejected because [excessive operational burden].

### Verified Facts & Telemetry
- [VERIFIED]: [Evidence item]

### Reversal Metric (Kill-Switch)
Reconsider or revert this decision if [quantifiable condition occurs].
```

---

## 📖 Real-World Case Studies

Check out the full walkthroughs in the [`examples/`](./examples/) directory:
- 🏛️ [**Architecture Decision**](./examples/architecture-decision.md): Full Event Sourcing vs Transactional Outbox Pattern for a high-throughput financial checkout service.
- 🐛 [**Tricky Debugging**](./examples/tricky-debugging.md): Triaging intermittent 504 gateway timeouts under burst load with low CPU/memory utilization.

---

## 🛡️ Anti-Failure Guardrails

1. **Anti-Consensus Alarm**: If all debaters agree immediately, run an immediate red-team probe: *"Assume this solution fails catastrophically within 30 days. How and why did it fail?"*
2. **Shared Model Mitigation**: Don't rely on model intuition alone; run tools and inspect files to verify claims.
3. **No Phantom Evidence**: Disqualify any role citing unverified "industry standard benchmarks" without specific references.
4. **Truth Over Winning**: Roles are evaluated by their contribution to finding the correct answer, including graceful concessions when proven wrong.

---

## 📄 License

Distributed under the MIT License. See [`LICENSE`](./LICENSE) for more information.

Developed by [gusfing](https://github.com/gusfing).

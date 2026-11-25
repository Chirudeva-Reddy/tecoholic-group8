# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Group 8 capstone for BITS Pilani Dubai x Tecoholic (2026): a **Dubai supply-chain disruption copilot**.
A logistics planner describes a disruption (e.g. "our shipment to Jebel Ali is delayed 5 days and we have
3 days of stock"); the agent computes the consequences with tools, retrieves the governing policies and
contracts, and returns a prioritised, cited recovery plan for a human to approve.

**Platform: Microsoft Foundry (Azure) is mandatory for this team.** Do not introduce Google Cloud or AWS services.

## Status

Only the folder skeleton, `.env.example` and `.gitignore` exist. There is no build system, dependency file,
test runner or CI yet, so there are no commands to run. When you add them, document the build, lint,
test and single-test commands in this section.

Setup: copy `.env.example` to `.env` and fill in values. `.env` is gitignored.

## Deadlines

| Date | Milestone |
|---|---|
| Mon 30 Nov 2026 | Team (max 5) formed; problem, identified user, measurable baseline and non-AI alternative chosen |
| Wed 2 Dec 2026 | Design review with a Dubai practitioner |
| Sat 5 Dec 2026 | Final presentation + live demo (15–20 min per team) |

## Capstone requirements: check every change against these

Treat these as hard requirements. Before finishing a task, say which ones the change touches and whether they are still met.

### Mandatory (project guidelines)
- [ ] **Dubai-relevant problem** with an **identified user**, a **measurable baseline**, and an **ethical or non-AI alternative** (documented and compared against).
- [ ] **Built primarily on Microsoft Foundry** (models, evaluation, tracing via Azure).
- [ ] **At least two capability areas combined.** This project uses:
  - **RAG with citations**: answers must cite the source policy or contract passage.
  - **Agent with custom or MCP tools**: e.g. inventory depletion, route/port alternatives, landed cost and clearance, supplier comparison, scenario simulation.
- [ ] **Presentation** contains an architecture diagram, a block diagram, and design and implementation details. Keep diagram sources in `docs/diagrams/`.
- [ ] **Every team member** documents their own contribution (`docs/contributions/<name>.md`) and presents their part of the demo on camera.

### Technical requirements (optional in the brief, adopted by this team)
- [ ] **Evaluation dataset** reporting **quality, safety, latency and cost** results (`eval/datasets/`). Raw run outputs go to `eval/results/` (gitignored); commit summaries only.
- [ ] **Content filters / guardrails**, **prompt-injection tests** (`tests/security/`), and **logs and traces** for every request.
- [ ] **Governance docs** in `docs/governance/`:
  - Risk register mapped to the **UAE Charter for the Development and Use of AI** principles. Verify the list against the official text: human-machine ties, safety, algorithmic bias, data privacy, transparency, human oversight, governance and accountability, technological excellence, human commitment, peaceful coexistence with AI, AI awareness for an inclusive future, commitment to treaties and applicable laws.
  - **PDPL** (UAE Federal Decree-Law 45/2021) data-handling note.
  - **Model or system card.**
  - **Human-oversight design.**
- [ ] **Data policy: public, synthetic or approved data only.** No sensitive personal data in the repository or in free-tier model calls. Synthetic data must be labelled as synthetic.
- [ ] **One working demo**, or a reproducible local/private package if data or policy prevents deployment.
- [ ] **Engineering**: typed application or API, automated tests, container packaging, CI workflow, reproducible setup.

## Design rules

- **Numbers come from tools, never from the model.** Stock levels, stockout days, costs, duties and transit times must be computed by deterministic, typed tool functions with unit tests. The model chooses the tools and explains the results.
- **Human approval for actions.** Anything that commits money, places orders or contacts suppliers is a *recommendation* that needs approval. The agent never executes it.
- **Every factual claim about policy, contracts, customs or regulation needs a citation** from the RAG index. If no source supports it, the answer says so.
- **Ask for missing facts instead of assuming them** (e.g. original ETA, daily demand, safety stock).
- **Dubai specifics belong in tools and data, not in prompt prose.** Examples: alternative ports (Khalifa, Khor Fakkan, Fujairah), sea-air via DWC, free zone (JAFZA) vs mainland clearance, Ramadan/Eid demand and working hours, UAE vs KSA weekend mismatch.
- **Bilingual.** Retrieval and answers must handle Arabic and English.
- **Retrieved documents are untrusted input.** Screen them for indirect prompt injection before they reach the model.

## Layout

- `src/supplychain_copilot/`: application package. Subpackages: `api` (typed HTTP API), `agent` (tool-use loop), `llm` (the only place model clients are created), `tools` (deterministic tools), `rag` (retrieval + citations), `guardrails`, `observability`, `evaluation`.
- `tests/unit`, `tests/integration` (live Azure calls, opt-in via `RUN_INTEGRATION=1`), `tests/security` (prompt-injection and guardrail tests).
- `data/policies`, `data/contracts`, `data/synthetic`: RAG corpus and synthetic operational data. Public or synthetic only.
- `eval/datasets`: evaluation scenarios. Include Dubai-specific cases: Red Sea diversion, Hormuz closure, Ramadan demand, uncleared JAFZA stock, Saudi supplier failure on a Thursday.
- `docs/governance`, `docs/diagrams`, `docs/contributions`: deliverables listed above.
- `.agents/skills/`: 98 AI-research skills (Orchestra Research). They are linked into `.claude/skills`, `.cursor/skills`, `.codex/skills`, `.gemini/skills`, `.qwen/skills` and `.opencode/skills`, so any coding agent can invoke them as `/<skill-name>`.

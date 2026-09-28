# VERA — DSAR Agent Suite

A working set of AI agents that run **Data Subject Access Requests** (GDPR / UK GDPR /
CCPA-CPRA / LGPD) end to end on **Workato Agent Studio**, built through the Workato AIRO MCP.

Intake and classification → identity verification → cross-system discovery → legal-hold and
retention assessment → redacted access package or erasure → subject communications, with a
regulator-grade audit trail at every step.

Everything runs on **Workato Data Tables and the built-in AI connector**, so the demo has
**no external connection or credential dependency**. Five simulated systems of record stand in
for Salesforce, Zendesk, Marketo, NetSuite and Workday; swapping them for the real connectors is
a per-step change documented in `docs/PRODUCTION-NOTES.md`.

---

## Before the first demo — two manual steps

Workato's MCP surface can create assets but **cannot start them**. Both steps are UI-only and
take about a minute:

1. **Start the 10 `[DSAR]` recipes** — <https://app.workato.com/recipes?fid=25925175>
2. **Activate the 5 `VERA` genies** — they are created in `stopped` state
3. Then, in the **VERA | DSAR Orchestrator** chat, say: **"Reset the DSAR demo data"**

Full detail in **[`docs/RUNBOOK.md`](docs/RUNBOOK.md)**.

---

## What's in here

| Document | What it covers |
| --- | --- |
| [`docs/RUNBOOK.md`](docs/RUNBOOK.md) | Activation, demo reset, troubleshooting, moving the suite to its own project |
| [`docs/DEMO-SCRIPT.md`](docs/DEMO-SCRIPT.md) | Three scenarios with the exact prompts to type and the numbers to expect |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Agent topology, skill contracts, data model, guardrail design |
| [`docs/PRODUCTION-NOTES.md`](docs/PRODUCTION-NOTES.md) | Replacing the simulated systems with real connectors; what is demo-grade and what is not |
| [`workato/asset-registry.md`](workato/asset-registry.md) | Every ID, handle and URL |

## The agents

| Agent | Role |
| --- | --- |
| **VERA \| DSAR Orchestrator** | Supervisor. Runs a case end to end; holds all 10 skills |
| **VERA \| Intake and Verification** | Classifies the request, sets the statutory clock, proves identity |
| **VERA \| Discovery and Assessment** | Finds every record, then rules each in or out of erasure |
| **VERA \| Fulfilment and Erasure** | Builds the redacted package; executes approved deletions |
| **VERA \| Privacy Ops Desk** | Read-only register, deadlines, breach risk, audit evidence |

The same 10 skills are also published as a Workato **MCP server**, so Claude Desktop or Claude
Code can act as the orchestrator directly against the same tools.

## The three guardrails worth demoing

1. **No identity, no data.** Discovery, package assembly and erasure each re-check
   `identity_verified` and halt before touching a source system. The refusal is the feature.
2. **Legal holds and statutory retention outrank the request.** Billing (7-year tax retention)
   and HR (6-year employment retention) are never erasable. An *active* litigation hold preserves
   the correspondence it covers — a *released* hold correctly does not.
3. **Agents draft; humans send and approve.** Erasure demands confirmation and records who
   approved it. No letter is ever sent by an agent.

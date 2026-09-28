# VERA DSAR Suite — Architecture

## Shape of the system

```
                    ┌─────────────────────────────┐
  Workato genie ──► │  VERA | DSAR Orchestrator   │
  chat              │  (supervisor, all 10 skills)│
                    └──────────────┬──────────────┘
                                   │
  Claude Desktop ──► Workato MCP ──┤   same skill layer, two runtimes
  / Claude Code      server        │
                                   ▼
        ┌──────────────────────────────────────────────────┐
        │  10 skills = 10 Workato recipes (deterministic)  │
        │  intake · verify · discover · assess · package   │
        │  · erase · case file · dashboard · comms · seed  │
        └──────────────────────────┬───────────────────────┘
                                   ▼
        ┌──────────────────────────────────────────────────┐
        │ Control plane          │ Simulated systems       │
        │ Cases                  │ SRC CRM Contacts        │
        │ Findings               │ SRC Support Tickets     │
        │ Audit Log              │ SRC Marketing Profiles  │
        │ System Registry        │ SRC Billing Accounts    │
        │ Legal Holds            │ SRC HR Records          │
        └──────────────────────────────────────────────────┘
                     Workato Data Tables
```

## Why two agents, not five

The first build had five genies — one per lifecycle stage. That was wrong, and the reasoning is
worth recording because it is the mistake this kind of demo invites.

**Skills** are split by transaction boundary: a unit of work that must succeed or fail atomically,
change case status, and be independently auditable. That gives 10. **Agents** should be split by
*authority* — who is allowed to do what. That gives 2.

A genie per stage confused the two axes. It produced four agents that differed only in which
subset of the same skills they held, with no boundary between them that anything enforced, because:

- Workato genies **cannot call other genies** (`genie_update` accepts skill handles only), so there
  was no delegation hierarchy — just four alternative front doors over one skill layer.
- The orchestrator already held all 10 skills, so the subsets were not a security boundary for
  anyone who could reach it.
- Four near-identical agents cost demo time to explain and diluted tool-routing accuracy.

What survives is the one split that an enforcement mechanism actually backs:

| Agent | Authority |
| --- | --- |
| **VERA \| DSAR Orchestrator** | Can act. All 10 skills, including erasure |
| **VERA \| Privacy Ops Desk** | Cannot act. Only the 2 read-only skills |

The Ops Desk cannot mutate a case or delete a record — not because its instructions forbid it, but
because the capability is not attached. That holds against a confused operator, a careless prompt,
and an injection attempt inside a support ticket it reads. Telling one all-powerful agent "please
do not delete things" does not.

**This is a design for least privilege, not yet an enforced one.** It only bites once project
access grants restrict who can open the orchestrator versus the Ops Desk. The MCP surface exposes
only read tools for grants (`project_grant_list`, `project_privilege_get`), so that is a UI step —
and it is the step that turns this from a diagram into a control.

## Why the logic lives in skills, not in prompts

Every decision a regulator could challenge is a **recipe step**, not a model judgement:

| Decision | Where it is made | Why |
| --- | --- | --- |
| Statutory deadline | Ruby formula on regulation | Must be arithmetic, not a guess |
| Identity confidence | Weighted score over three systems | Must be explainable and repeatable |
| Erasable / blocked | Deterministic rule over holds + registry | A wrong answer here is unlawful |
| What actually gets deleted | Explicit per-system recipe branch | Must be auditable |

The model is used only where judgement genuinely helps, and always into a typed schema:

| AI step | Job |
| --- | --- |
| Intake classification | Read free text → right exercised, regulation, jurisdiction, risk flags |
| Support ticket scan | Detect third-party PII and Article 9 content in free text |
| Package composition | Write the disclosure document, applying the redaction rules |
| Subject communication | Draft the regulation-aware letter |

That split is the point. An LLM deciding *"is this health data?"* is appropriate. An LLM deciding
*"may we delete this invoice?"* is not.

## Case lifecycle

```
received → verifying → verified → discovered → pending_review → completed
                │                                     │
                └── pending_manual_verification        └── (erasure) completed
```

Status is written by whichever skill advanced it; `[DSAR] Audit Log` records every transition
with actor, action, detail and outcome.

## Guardrails

**Identity gate.** Skills 03 (discover), 05 (package) and 06 (erase) each re-read the case and
halt before touching any source system unless `identity_verified` is true. The check is
re-evaluated per skill rather than trusted from the caller, so an agent cannot talk its way past
it by asserting the case is verified. It is implemented as a `ruby(...).to_s.downcase != "true"`
comparison so it fails closed on null, false or any unexpected type.

**Erasure gate.** Skill 06 additionally refuses unless `request_type == "erasure"`, is marked
`requires_user_confirmation = true` so Workato prompts before running, and reads only findings
already marked `erasable = true`. It never re-evaluates erasability itself — separation of duties
between the agent that decides and the agent that acts.

**Retention precedence.** Evaluated in order: statutory retention (billing, HR) → active legal
hold (support correspondence) → the registry's erasure method. An *inactive* hold is ignored;
only `active = true` blocks. Every block writes a citable `blocked_reason`.

**Draft, never send.** Skill 09 returns a subject line and body and writes an audit entry with
outcome `drafted_not_sent`. No skill has an email connector.

**Minimisation in reporting.** The Privacy Ops Desk returns counts, statuses and reasons — its
instructions forbid reproducing subject personal data in a status report.

## Data model

**Control plane**

- `[DSAR] Cases` — one row per request; classification, statutory clock, verification state, outcome
- `[DSAR] Findings` — one row per record found; classification, sensitivity, third-party flag,
  erasable verdict, blocked reason, action taken
- `[DSAR] Audit Log` — append-only; event, case, timestamp, actor, detail, outcome
- `[DSAR] System Registry` — the data map: categories, owner, legal basis, retention rule,
  erasure method, special-category flag
- `[DSAR] Legal Holds` — subject, matter, type, active flag, dates

**Simulated systems of record** — `SRC CRM Contacts`, `SRC Support Tickets`,
`SRC Marketing Profiles`, `SRC Billing Accounts`, `SRC HR Records`.

### Registry drives policy; recipe steps drive access

The System Registry is not a dispatch table. Workato binds a data table at design time, so
discovery has one explicit search step per system. The registry supplies the *policy* — retention
rule, legal basis, erasure method, owning team — that the assessment step and the disclosure
document reason over. Adding a sixth system means adding a search step **and** a registry row.
This mirrors production, where each system needs its own connector call anyway.

## Skill contracts

| # | Skill | Key inputs | Returns |
| --- | --- | --- | --- |
| 00 | Seed and Reset Demo Data | — | counts reloaded |
| 01 | Intake and Classify Request | raw_request, requester_email | case_id, classification, SLA |
| 02 | Verify Requester Identity | case_id, evidence | verified, confidence, method |
| 03 | Discover Personal Data | case_id | per-system counts, total, special-category flag |
| 04 | Assess Erasability | case_id | erasable/blocked counts, hold detail |
| 05 | Build Subject Access Package | case_id, tone | package_markdown, redaction counts, sign-off |
| 06 | Execute Erasure | case_id, approved_by | erased/retained counts, retention explanation |
| 07 | Get Case File | case_id | header, findings, audit trail |
| 08 | Privacy Ops Dashboard | status_filter | register, open/completed counts, today |
| 09 | Draft Subject Communication | case_id, message_type | subject_line, body, `sent: false` |

Every skill writes its own audit entry. Skills are re-runnable: discovery replaces prior findings
for the case rather than duplicating them.

## Known limits

- **Assets are created stopped.** The MCP surface has no start/activate tool; see the runbook.
- **Days-remaining is computed by the agent, not the recipe.** Workato's formula allowlist has no
  safe date-difference primitive, so skills return `sla_due_at` plus `today` and the genie does the
  subtraction. Deliberate: a formula that errors at runtime would break the skill.
- **Discovery matches on exact email.** No fuzzy matching, alias resolution or name-based search.
  Real deployments need identity resolution — see `docs/PRODUCTION-NOTES.md`.
- **No knowledge base attached.** The regulatory playbook is baked into the genie instructions so
  there is no ingestion lag. A KB is the better home once the policy text is owned by Legal.

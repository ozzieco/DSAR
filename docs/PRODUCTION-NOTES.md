# From demo to production

Honest notes on what this build is and what it would take to run it for real. Worth reading
before a technical buyer asks — the answers are better than bluffing.

## What is genuinely production-shaped

- The **control plane** (cases, findings, audit log, system registry, legal holds) is the real
  design. It would not change materially in production.
- The **guardrails** — identity gate, erasure confirmation, retention precedence, draft-never-send
  — are implemented as recipe logic, not prompt text, and would survive the move.
- The **audit trail** is append-only and captures actor, action, timestamp, detail and outcome
  for every step including refusals.
- The **skill contracts** are typed and re-runnable; the AI steps write into fixed schemas.

## What is demo scaffolding

| Demo | Production |
| --- | --- |
| 5 `[DSAR] SRC …` data tables | Real connectors: Salesforce, Zendesk, Marketo, NetSuite, Workday |
| Exact-email matching | Identity resolution across aliases, former addresses, customer IDs |
| Registry as documentation | Same, plus a per-system connector mapping maintained by system owners |
| Seed/reset skill | Delete it. It truncates case state |
| Package returned as markdown in chat | Encrypted portal download or secure link with expiry |
| Letters drafted in chat | Draft into the privacy inbox for a human to send |

## Swapping a simulated system for a real one

Each source system appears in exactly two places. Taking Salesforce as the example:

**1. Discovery** — `[DSAR] 03 Discover Personal Data`, step 7 currently reads
`workato_db_table.get_records(table_id="153067", filters=[email eq subject_email])`.
Replace with `salesforce.search_records` on Contact where `Email = subject_email`. Then update
the `foreach` at step 8 to map the Salesforce fields into the same finding shape — the rest of
the recipe is unchanged because it only reads the Findings table.

Repeat for Support (step 10), Marketing (14), Billing (17) and HR (20). Note step 12: the AI
free-text scan for third-party and Article 9 content should be kept and pointed at whatever the
real correspondence field is. That step earns its keep.

**2. Erasure** — `[DSAR] 06 Execute Erasure`, the branch on `system_key` at steps 8–13. Replace
the data-table delete/upsert with the system's real operation (`salesforce.delete_record`,
Zendesk redaction API, Marketo suppression list). The branch structure and the
"only act on `erasable = true`" contract stay exactly as they are.

Nothing else changes. Assessment, packaging, communications, the dashboard and the case file all
read the Findings table and never touch a source system.

A real workspace connection is required per connector, and the recipes must be re-tested after
swapping — field names and pagination differ from the data-table shape.

## Gaps to close before a real deployment

**Identity resolution.** Exact email match is not good enough. People change addresses, use work
and personal accounts (this demo's Priya Raman shows the failure mode deliberately), and appear
under customer IDs. This needs a resolution service or a golden-record table feeding the discovery
step a set of identifiers rather than one string.

**Coverage assurance.** The suite searches the systems in the registry. It cannot know about a
system nobody registered. Production needs the registry tied to the data-map / RoPA process, with
an attestation that it is complete — otherwise an incomplete DSAR response looks like a lie.

**Unstructured data.** No file shares, mailboxes, data lake or backups are searched. These are
usually the hardest part of a real DSAR and the most common source of regulator findings.

**Authorised agents and minors.** Intake classifies `agent_on_behalf` but there is no proof-of-
authority workflow behind it.

**Extensions.** GDPR allows a two-month extension for complex requests and CCPA a further 45 days.
The communication skill can draft an extension letter, but nothing recalculates `sla_due_at` — a
privacy officer must do it. Worth adding.

**Access control.** Anyone who can chat to the orchestrator can run discovery on any email address
in the register. Production needs the genie scoped to the privacy team, and ideally a check that
the operator is entitled to see that case.

**Erasure is not reversible and has no dry-run.** Adding a `dry_run` parameter to skill 06 that
reports what *would* happen would be cheap and is the first thing a security reviewer will ask for.

**Data residency and retention of the DSAR record itself.** The case file contains the subject's
personal data. It needs its own retention rule, which is typically the statutory limitation period
for a complaint and no longer.

## Cost and latency

Discovery runs one AI classification per support ticket, so cost scales with correspondence volume,
not with the number of subjects. Package composition is a single call over all findings. For a
subject with tens of thousands of tickets, batch or pre-filter the free-text scan before promising
this at enterprise scale.

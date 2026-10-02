# VERA DSAR Suite — Demo Script

Three scenarios, roughly 12 minutes. Each is a copy-paste prompt into the
**VERA | DSAR Orchestrator** genie chat, plus the numbers you should see.

Reset first: **"Reset the DSAR demo data."**

> Before the first run, work through [`TEST-SCRIPT.md`](TEST-SCRIPT.md) instead — it covers these
> three scenarios plus the refusal paths, the remaining letter types and the agent capability
> boundary, with a pass/fail checklist.

> The case IDs are generated per run (`DSAR-<workato job id>`), so capture the one the intake
> step returns and reuse it. The orchestrator carries it for you if you stay in one thread.

---

## Scenario A — Consumer GDPR access request (the happy path)

**What it proves:** one sentence of plain text becomes a classified, deadline-tracked case;
discovery finds records the business had forgotten; third-party data is redacted automatically.

### Prompt

> A new privacy request just came in by email from marta.okonkwo@example.de.
> She wrote: "Hello, under GDPR I would like a copy of all personal data your company holds
> about me. I am based in Berlin. Please confirm receipt."
> Please take it from intake all the way to a reviewable access package.

### What to expect

| Step | Result |
| --- | --- |
| Intake | `access` / `GDPR` / Germany / consumer, **30-day** deadline, case opened |
| Verification | confidence **65/100** → verified (CRM + NetSuite Billing corroborate) |
| Discovery | **6 records** — CRM 2, Support 2, Marketing 1, Billing 1, HR 0 |
| Assessment | **5 erasable, 1 blocked** (billing: 7-year tax retention) |
| Package | 6 records disclosed, **1 redacted**, human sign-off required |

### The three things to point at

1. **The duplicate.** Discovery returns *two* CRM contacts — `CRM-10041` and `CRM-10377`, an
   unmerged trade-show import. A human searching Salesforce by name would have found one.
2. **The redaction.** Ticket `TKT-88120` names a colleague, Jonas Weber, with his email and
   phone. The package replaces him with `[REDACTED — THIRD PARTY]`. Nobody wrote a rule for
   "Jonas Weber" — the agent read the free text and worked out he was a third party.
3. **The released hold.** Marta has a legal hold on file, `LH-1990`, from a 2024 product recall.
   It is **released**, and the agent correctly ignores it. Ask: *"was Marta under a legal hold?"*

---

## Scenario B — CCPA erasure with an active legal hold (the one that matters)

**What it proves:** the agent refuses to destroy evidence, and explains the refusal in language
you could send to the data subject.

> **Reset the demo data before running this** — it deletes real rows.

### Prompt

> Daniel Reyes raised this through support ticket TKT-90455: "I want my account closed and every
> piece of personal information you hold about me deleted. I am a California resident and I am
> exercising my rights under the CCPA."
> His email is daniel.reyes@example.com. Open the case, verify him, and work out exactly what we
> can and cannot delete. Do not delete anything yet.

### What to expect

| Step | Result |
| --- | --- |
| Intake | `erasure` / `CCPA/CPRA` / United States – California, **45-day** deadline |
| Discovery | **5 records** — CRM 1, Support 2, Marketing 1, Billing 1 |
| Assessment | **2 erasable, 3 blocked**, 1 active legal hold (`LH-2041`) |

The three blocked records carry two distinct, citable reasons:

- `BIL-30488` — *"Retained under statutory obligation: financial records must be kept for 7 years
  from the last invoice date under tax and audit law. GDPR Art. 17(3)(b)."*
- `TKT-90210`, `TKT-90455` — *"Preserved under active legal hold LH-2041 — Reyes v. Northwind
  Trading … Art. 17(3)(e): necessary for the establishment, exercise or defence of legal claims."*

### Then approve it

> That looks right. I'm the privacy officer, Austin Cowan, and I approve the erasure of the
> records that are cleared. Go ahead, then draft the completion letter to Daniel.

| Step | Result |
| --- | --- |
| Erasure | **2 erased** (CRM contact deleted, marketing profile suppressed), **3 retained** |
| Letter | states the deletion *and* the retention, with reasons, in plain English |

### The three things to point at

1. **Confirmation is real.** The erasure skill is configured `requires_user_confirmation = true`.
   Workato prompts before it runs. Approval is captured in the audit log against a named person.
2. **Three different verbs.** CRM is *deleted*. Marketing is *suppressed* — deleting it outright
   would destroy the opt-out and risk re-contacting him. Support is *preserved* under the hold.
   One request, three lawful treatments.
3. **Verify the deletion.** Open `[DSAR] SRC CRM Contacts` — `CRM-10188` is gone. Open
   `[DSAR] SRC Marketing Profiles` — `MKT-55884` still has his email (to honour the opt-out) but
   the name, score, IP and UTM are erased and the segment reads `SUPPRESSED - do not contact`.

---

## Scenario C — Employee UK GDPR access, special category data

**What it proves:** the agent recognises Article 9 data and escalates instead of releasing it.

### Prompt

> Priya Raman, one of our employees, has made a subject access request from her personal address
> priya.raman@example.co.uk. She wrote: "I'd like to see everything HR holds about me, including
> my performance reviews and anything from occupational health."
> Run the case and tell me whether I can release the package myself.

### What to expect

| Step | Result |
| --- | --- |
| Intake | `access` / `UK GDPR` / United Kingdom / **employee**, 30-day deadline |
| Discovery | **4 records** — HR 1, CRM 1, Support 1, Marketing 1 |
| Assessment | **3 erasable, 1 blocked** (HR: 6-year employment retention) |
| Package | flags **special category** content; `requires_human_signoff = true` |

### The three things to point at

1. **The answer is no.** The agent tells you that you cannot release it alone: `HR-4402` contains
   an occupational-health referral about a degenerative eye condition — Article 9 data needing
   People Team and Legal sign-off.
2. **It found the data twice.** Priya's health information is in the HR record *and* volunteered
   in support ticket `TKT-87011`. The AI classifier caught the second one from free text where
   no field is labelled "health".
3. **Shadow records.** She appears in CRM and Marketing under her *personal* email from
   co-hosting a partner webinar — with no consent captured. An HR-only search misses both.

---

## Closing: the register

> Give me the privacy operations picture — what's open and what's at risk?

The Privacy Ops Desk returns every case with status, regulation, deadline and risk flags, and
computes days remaining against today. Then:

> Show me the full audit trail for Daniel's case.

Every agent action, timestamped, with the approver named — case opened, identity scored,
systems searched, each erasure decision and its legal ground, what was executed, what was drafted.

**The line to land it on:** the audit trail is not a by-product of the demo. It is the deliverable.
The regulator's question is never "do you have AI" — it is "show me what you did, when, on whose
authority, and why you kept what you kept."

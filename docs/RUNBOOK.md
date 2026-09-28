# VERA DSAR Suite — Runbook

Workato project folder: **Adhoc Testing**, `folder_id = 25925175`
<https://app.workato.com/recipes?fid=25925175>

---

## 1. Activate the suite (once, before the first demo)

The Workato MCP surface can create recipes and genies but has **no tool to start them**, so
these two steps must be done in the Workato UI. Budget one minute.

### 1a. Start the recipes

Open <https://app.workato.com/recipes?fid=25925175> and start all ten:

- `[DSAR] 00 Seed and Reset Demo Data`
- `[DSAR] 01 Intake and Classify Request`
- `[DSAR] 02 Verify Requester Identity`
- `[DSAR] 03 Discover Personal Data`
- `[DSAR] 04 Assess Erasability`
- `[DSAR] 05 Build Subject Access Package`
- `[DSAR] 06 Execute Erasure`
- `[DSAR] 07 Get Case File`
- `[DSAR] 08 Privacy Ops Dashboard`
- `[DSAR] 09 Draft Subject Communication`

A skill whose recipe is stopped will not run when the genie calls it. The MCP server's tools
also report `Active: No` until their backing recipe is started — starting the recipes clears
both at once.

### 1b. Activate the genies

Each was created in `stopped` state. Open each and activate:

| Genie | URL |
| --- | --- |
| VERA \| DSAR Orchestrator | <https://app.workato.com/genies/gin-AbkM9XCw-TALrCH-CD/overview> |
| VERA \| Privacy Ops Desk | <https://app.workato.com/genies/gin-AbkMAhgM-Ps4Q3N-CD/overview> |

### 1b-ii. Delete the three retired genies

An earlier revision built a genie per lifecycle stage. Those were folded into the orchestrator.
All three have had every skill detached, so they cannot act, but the MCP surface has no
`genie_delete` — remove them in the UI:

- `[RETIRED] Intake and Verification` — `gin-AbkM9pK8-WNxLaW-CD`
- `[RETIRED] Discovery and Assessment` — `gin-AbkMABdc-9AzAeG-CD`
- `[RETIRED] Fulfilment and Erasure` — `gin-AbkMAReC-TPk3D6-CD`

Leaving them costs nothing functionally; they are just clutter in the genie list.

### 1c. Load the demo data

In the **VERA | DSAR Orchestrator** chat:

> Reset the DSAR demo data.

It should report 5 registry systems, 3 legal holds and 25 source records across
CRM, Support, Marketing, Billing and HR.

Run this again any time you want a clean slate — it is idempotent, and it is the only way to
restore records that a demo erasure deleted.

---

## 2. Resetting between demos

Say **"Reset the DSAR demo data"** in the orchestrator chat. This:

- truncates `[DSAR] Cases`, `[DSAR] Findings` and `[DSAR] Audit Log` (all case state)
- re-upserts the system registry, the legal holds and all five simulated source systems

Erasure is destructive against the source tables, so **always reset before re-running
Scenario B**, or Daniel Reyes's CRM contact will already be gone.

---

## 3. Troubleshooting

**"The skill refused — identity not verified."**
Working as designed. Run the verification skill for that `case_id` first. If it genuinely
should pass, check the subject's email exists in `[DSAR] SRC CRM Contacts` (reset the data).

**Discovery returns zero records.**
The source tables are empty — run the reset. Or the case's `subject_email` does not match the
seeded addresses; check the case with the case-file skill.

**A skill call errors immediately.**
Its recipe is stopped. See step 1a.

**Erasure says nothing was erasable.**
Check the case is actually an `erasure` request — the skill refuses on any other request type
by design. For Daniel Reyes expect 2 erasable and 3 retained, not 5.

**Genie cannot see a skill.**
Confirm the skill handle is attached to that genie and that the backing recipe is running.

---

## 4. Moving the suite into its own project

The suite lives in the shared **Adhoc Testing** project only because the MCP surface cannot
create a project folder. Everything is namespaced `[DSAR]` / `VERA`, so migration is mechanical:

1. Create a project in the Workato UI, e.g. `[34] VERA | DSAR Agent Suite`.
2. Note its `folder_id`.
3. Move the assets — in the UI, or via MCP:
   - recipes: `recipe_builder_update_asset_metadata(asset_type="recipe", asset_id=<id>, folder_id=<new>)`
   - data tables: `data_table_update(id=<uuid>, folder_id=<new>)`
   - genies: `genie_update(genie_id=<gin-...>, folder_id=<new>)`
4. Re-run the reset to confirm the table IDs still resolve (they are numeric and folder-independent,
   so they will).

The five stale `Test` / `Coupa` recipes already in that folder are unrelated and untouched.

---

## 5. Using the suite from Claude Desktop or Claude Code

A Workato MCP server publishes the same 10 skills to external clients:

- **Server**: `VERA DSAR Agent Tools` (`mcps-AbkMBExx-THQ-CD`)
- **MCP URL**: `https://28472.apim.mcp.workato.com`
- **Auth**: hashed token — generate it on the
  [server page](https://app.workato.com/ai_hub/mcp/mcps-AbkMBExx-THQ-CD/overview)

Connect it and Claude itself becomes the orchestrator, driving the identical skill layer the
Workato genies use. That contrast — same tools, two different agent runtimes — is the strongest
five seconds of the demo.

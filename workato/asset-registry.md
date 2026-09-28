# VERA DSAR Suite — Workato Asset Registry

Every ID, handle and URL created by this build. Workspace folder: **Adhoc Testing**,
`folder_id = 25925175` (project `12929772`) — <https://app.workato.com/recipes?fid=25925175>

> The suite is namespaced `[DSAR]` / `VERA` so it can be moved into a dedicated project in one
> pass. See `docs/RUNBOOK.md` § 4.

## Data tables

| Table | UUID | Numeric ID (used by recipes) |
| --- | --- | --- |
| `[DSAR] Cases` | 6e0583f4-c6e0-4e06-92d0-052cb2563bd4 | 153062 |
| `[DSAR] System Registry` | 152f8d39-b49d-4543-b8be-5a8470206bfa | 153063 |
| `[DSAR] Findings` | 48cbf537-d26d-4951-a9e4-4a29c46c4790 | 153064 |
| `[DSAR] Audit Log` | 4c3ab0b8-179d-48e6-a934-1045f4753217 | 153065 |
| `[DSAR] Legal Holds` | f506f0f9-9948-48ed-903b-d65da4473cf6 | 153066 |
| `[DSAR] SRC CRM Contacts` | 4d36a5cf-1d5c-4847-82be-c546d053deae | 153067 |
| `[DSAR] SRC Support Tickets` | 2efdf88b-c129-4563-97b7-f50b7153337c | 153068 |
| `[DSAR] SRC Marketing Profiles` | 7337825b-5248-4f59-a53a-dfa3146b7a71 | 153069 |
| `[DSAR] SRC Billing Accounts` | a6efafa0-f263-43dc-91b3-655202b2d011 | 153070 |
| `[DSAR] SRC HR Records` | 2246804a-d930-40dd-b1fd-53bc4fdd7d03 | 153071 |

## Skills

| # | Skill | Recipe ID | Skill handle |
| --- | --- | --- | --- |
| 00 | `[DSAR] 00 Seed and Reset Demo Data` | 82651199 | `skl-AbkLQ9X6-3D96NQ-CD` |
| 01 | `[DSAR] 01 Intake and Classify Request` | 82651213 | `skl-AbkLaFME-9zAG3R-CD` |
| 02 | `[DSAR] 02 Verify Requester Identity` | 82651217 | `skl-AbkLcaHY-zWneaE-CD` |
| 03 | `[DSAR] 03 Discover Personal Data` | 82651237 | `skl-AbkLfGhG-nYArhk-CD` |
| 04 | `[DSAR] 04 Assess Erasability` | 82651255 | `skl-AbkLnd6Q-nYnmza-CD` |
| 05 | `[DSAR] 05 Build Subject Access Package` | 82651257 | `skl-AbkLr8wC-ho8dPW-CD` |
| 06 | `[DSAR] 06 Execute Erasure` | 82651258 | `skl-AbkLtzBf-pAsnEw-CD` |
| 07 | `[DSAR] 07 Get Case File` | 82651259 | `skl-AbkM3HYD-wkrDdB-CD` |
| 08 | `[DSAR] 08 Privacy Ops Dashboard` | 82651261 | `skl-AbkM4nLk-EepWNp-CD` |
| 09 | `[DSAR] 09 Draft Subject Communication` | 82651263 | `skl-AbkM6eJt-DEzwba-CD` |

Recipe URL pattern: `https://app.workato.com/recipes/<recipe id>`

## Genies

| Genie | Genie ID | Skills |
| --- | --- | --- |
| VERA \| DSAR Orchestrator | `gin-AbkM9XCw-TALrCH-CD` | all 10 |
| VERA \| Privacy Ops Desk | `gin-AbkMAhgM-Ps4Q3N-CD` | 07, 08 |

### Retired genies — delete in the UI

Folded into the orchestrator. Every skill is detached (skill count 0), so they cannot act, but
there is no `genie_delete` in the MCP surface.

| Genie | Genie ID |
| --- | --- |
| `[RETIRED] Intake and Verification` | `gin-AbkM9pK8-WNxLaW-CD` |
| `[RETIRED] Discovery and Assessment` | `gin-AbkMABdc-9AzAeG-CD` |
| `[RETIRED] Fulfilment and Erasure` | `gin-AbkMAReC-TPk3D6-CD` |

> **`genie_update` tool quirk.** It ignores the `genie_id` argument and writes to whichever genie
> the Genie Builder session is bound to. Always call `genie_get(genie_id=...)` in a fresh
> `session_id` immediately before `genie_update`, then re-read with `genie_list` to confirm the
> write landed on the intended asset. Three renames issued in one session silently all applied to
> the same genie.

Genie URL pattern: `https://app.workato.com/genies/<genie id>/overview`
All were created in `stopped` state and must be activated once — see `docs/RUNBOOK.md` § 1b.

## MCP server

- **Name**: `VERA DSAR Agent Tools`
- **Handle**: `mcps-AbkMBExx-THQ-CD`
- **MCP URL**: `https://28472.apim.mcp.workato.com`
- **Auth**: hashed token (generate on the server page)
- **Tools**: all 10 skills
- **Server page**: <https://app.workato.com/ai_hub/mcp/mcps-AbkMBExx-THQ-CD/overview>

## Seeded demo data

Loaded by skill 00. Three demo subjects and two decoys.

| Subject | Email | Scenario | CRM | Support | Mktg | Billing | HR | Total |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Marta Okonkwo | marta.okonkwo@example.de | A — GDPR access | 2 | 2 | 1 | 1 | 0 | **6** |
| Daniel Reyes | daniel.reyes@example.com | B — CCPA erasure | 1 | 2 | 1 | 1 | 0 | **5** |
| Priya Raman | priya.raman@example.co.uk | C — UK GDPR employee access | 1 | 1 | 1 | 0 | 1 | **4** |
| Gregor Lindqvist | gregor.lindqvist@example.se | decoy | 1 | 1 | 1 | 1 | 0 | 4 |
| Aiko Tanaka | aiko.tanaka@example.jp | decoy | 1 | 1 | 1 | 1 | 0 | 4 |
| Others (HR only) | — | third-party context | — | — | — | — | 2 | 2 |

25 source records, 5 registry systems, 3 legal holds.

### Planted details that carry the demo

- `CRM-10377` — an unmerged **duplicate** of Marta, from a trade-show import
- `TKT-88120` — names a **third party** (Jonas Weber) with email and phone; must be redacted
- `TKT-87011` — Priya volunteers a **health condition**; Article 9 special category data
- `HR-4402` — occupational health referral and a grievance reference; Article 9
- `LH-2041` — **active** litigation hold on Daniel Reyes (billing dispute)
- `LH-1990` — **released** hold on Marta; must *not* block anything
- `LH-2088` — active hold on an unrelated person; must *not* match

### Expected outcomes

| Scenario | Verification | Discovery | Assessment | Fulfilment |
| --- | --- | --- | --- | --- |
| A — Marta | 65/100 verified | 6 records | 5 erasable, 1 blocked (tax) | 6 disclosed, 1 redacted, sign-off required |
| B — Daniel | 65/100 verified | 5 records | 2 erasable, 3 blocked (1 tax, 2 legal hold) | 2 erased, 3 retained |
| C — Priya | 65/100 verified | 4 records | 3 erasable, 1 blocked (employment law) | special category, sign-off required |

Identity scoring: CRM match 40, billing 25, HR 25, subject-supplied evidence 10; verified at ≥ 60.

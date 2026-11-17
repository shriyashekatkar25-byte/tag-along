# tag along

**Should a governed tag follow every derived column, or ask a steward first?** A Bayesian Beta-Binomial Monte Carlo picks the default per transform type.

- **Live demo:** https://shriyashekatkar25-byte.github.io/tag-along/
- **APM Lab gallery:** https://shriyashekatkar25-byte.github.io/apm-lab/
- Part of APM Lab by Shriya Shekatkar (UCLA, Stats & Data Science) · Stats 102C Monte Carlo Methods, Ch2 Bayesian statistics

## The launch that inspired it

On Nov 16, 2026 Databricks announced that **governed tag propagation now supports classic compute (Beta)** on Databricks Runtime 19 and above, in addition to serverless.
([Databricks release notes, November 2026](https://docs.databricks.com/aws/en/release-notes/product/2026/november))

From the [tag propagation docs](https://docs.databricks.com/aws/en/admin/governed-tags/tag-propagation), checked this run:
- Governed tags copy from source tables/columns to derived tables and views, using Unity Catalog lineage; a column-level tag (e.g. `pii`) moves to the matching derived column.
- It runs on CTAS, views, `INSERT INTO ... SELECT` and `MERGE`; it is off by default and enabled per governed tag.
- Propagation is asynchronous, and ABAC policies tied to a propagated tag aren't enforced on the derived object until the tag lands.

## The PM decision

With classic compute in scope, far more derived columns inherit tags. But not every transform keeps data sensitive: a monthly `SUM()` or a salted hash usually isn't the raw value, and copying the mask there blocks analysts. Imagining I'm the PM, the decision is: **per transform type, should the default be auto (copy the tag), suggest (pending tag a steward confirms or clears), or drop (lineage only)?**

- **Success metric:** expected harm per derived column (1 unit = one wrongly masked column; a missed sensitive column costs `c_leak` units).
- **Guardrails:** untagged sensitive derived columns ≤ 0.5%, wrongly masked ≤ 12%, steward queue ≤ 250/week, P(leak guardrail holds) ≥ 90%.
- **Stakeholders:** governance admins and stewards, analysts, security/compliance, lineage and classic-runtime engineering.

With the default (simulated) audit, the prototype recommends **auto** for select/rename, join, filter and ML features, **suggest** for aggregates and hashes, and stamps it **ship with conditions**: leaks and wrong masks pass, but the steward queue (~640/week) breaches, so the next step is batching suggestions.

## What the prototype shows about product judgment

| APM skill | Where it shows up |
|---|---|
| Customer problem framing | Two customers with opposite pain (admins fear leaks, analysts hate over-masking) turned into one harm metric plus guardrails |
| Prototyping | A working, seeded simulation with per-lane overrides so a reviewer can test any policy in seconds |
| SQL / Python | Illustrative audit query and NumPy posterior-draw code that would run the same model on real review data |
| Data-driven decisions | The default is the action with the lowest posterior expected harm, not a gut call; credible intervals show how sure we are |
| Clear writing | A director memo and one-line VP summary that rewrite themselves when the inputs change |

## Methods

| Course chapter | Method | Product use here |
|---|---|---|
| Ch2 Bayesian statistics | Beta prior + Binomial audit → Beta posterior (conjugate) | Belief about θ = chance a column stays sensitive after each transform |
| Ch2 Bayesian statistics | Posterior mean as a weighted average; 90% quantile credible intervals | Shows how audit size overrides the prior |
| Ch1 Monte Carlo basics | Expectations via sample means over 2,000 seeded draws | Expected harm of auto / suggest / drop; P(guardrail holds) |
| Ch5 / Ch4 | Box-Muller normals; Gamma draws (a rejection-style sampler) combined into Beta draws | Generating posterior draws and count noise |

## Design baseline

Four random pins from Shriya's Pinterest home feed, redrawn in CSS/SVG/canvas (no pin images are used):

- **Hero** ← ["the world is yours" pink script card](https://www.pinterest.com/pin/585186545375909006/) (tarynn4ubrey): soft bubblegum pink card, plum flowing script headline, rounded corners.
- **Cards & buttons** ← [Stop Waiting. Start Becoming Her! | Dream Life Mindset](https://www.pinterest.com/pin/533817362106180095/) (Mikki James, Bon Bon Vie Beauty and Wellness): ochre/mustard paper, oxblood high-contrast serif, tiny tracked caps labels.
- **KPI badges** ← ["I now break all curses placed upon my name…"](https://www.pinterest.com/pin/790944753356020713/) (tarotreaderpeter): hand-drawn black marker type, yellow stars in circled dots, wobbly sticker outlines.
- **Simulation room** ← [Dainty Black Waist Beads w/ dragonfly](https://www.pinterest.com/pin/82683343155406247/) (Etsy listing): white specimen card with a double black frame, script title over an uppercase list, olive/blue dragonfly and cream wing veins (an original dragonfly is drawn on the canvas).

---

Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.

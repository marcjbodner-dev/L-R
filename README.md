# L&R

## Artifacts

### `artifacts/business-diagnosis.html`: L&R Business Diagnosis (In Month)

Source for the Claude artifact at https://claude.ai/artifact/XRoZK7evZTQkkqXepixC9c
(17 views: Executive & board, Oct forecast, Calls, eCommerce, DC operations,
Open orders, Fill rate, Trend, Customers, Call coverage, Never visited, Recovery,
Forecast, Other channels, Obsolete & invalid, Actions, September).

**This repo is public, so this copy is code only.** Every embedded data block
(`#data`, `#data2`, `#data3`, `#data5` and the `OO`, `RECR`, `ZR`, `REL`,
`STALE`, `FIXT`/`LINK`, `INVX`/`INVALL`, `QOO`, `FL` constants) has been reset
to an empty shell. Orders, customers, dollars and people's names are not in git.
Opened on its own, most views render empty or throw "undefined" errors. That
is expected.

Live data comes from Power BI (workspace *LRDist Data*) through the artifact's
`mcp` capability (`PowerBI` → `ExecuteQuery`):

| Model | ID | Used for |
|---|---|---|
| DatasetRealtime | `955423b5-1e79-4b2f-85a8-2902e214de53` | Live eCommerce / open-order refresh |
| DatasetSilver | `09da7a74-cb96-4288-901f-e5934fb68d1e` | Sales, budget, open orders, fill, calls |

**Never commit the data-filled version of the page to this repo.** The published
artifact on claude.ai is the copy that holds data.

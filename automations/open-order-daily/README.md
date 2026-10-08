# Daily open-order analysis → Teams (Power Automate)

Posts an open-order summary card to a Teams chat **every weekday at 9:30 AM Eastern**.
The data comes straight from the **DatasetSilver** model in Power BI (workspace *LRDist Data*).
Everything runs inside Microsoft 365, so no webhook link or Claude session is needed.

The card shows:
- **Totals:** open order $, order count, past due $, due today $, picked $ and fill %
- **By channel:** open $, past due, due today, due rest of week and fill %
- **Label:** "Preliminary", because the 9:30 run is before 10 AM ET west-coast EDI lands
- **Link:** an "Open dashboard" button that goes to the Business Diagnosis artifact

> **Untested:** the DAX queries mirror the dashboard's live queries but have not been run
> yet. Run the flow once manually (step 6) before trusting the numbers.

## Files

| File | Used in |
|---|---|
| `totals-query.dax` | Step 2: one-row totals |
| `channel-query.dax` | Step 3: one row per channel |
| `header-row.json` / `select-row.json` | Step 4: builds the table rows |
| `card.json` | Step 5: the Adaptive Card |

## Build the flow

In **make.powerautomate.com**, choose **Create → Scheduled cloud flow**.

1. **Recurrence**
   - Repeat every **1 Week**, On: **Mon, Tue, Wed, Thu, Fri**
   - At these hours: **9**, At these minutes: **30**
   - Time zone: **(UTC-05:00) Eastern Time (US & Canada)**

2. **Power BI → Run a query against a dataset**. Rename the action to **`Run totals query`**.
   - Workspace: *LRDist Data*, Dataset: *DatasetSilver*
   - Query text: paste `totals-query.dax`

3. **Power BI → Run a query against a dataset**. Rename the action to **`Run channel query`**.
   - Same workspace and dataset. Query text: paste `channel-query.dax`

4. **Data Operation → Select**. Rename it to **`Select rows`**.
   - From: `body('Run_channel_query')?['firstTableRows']`
   - Switch Map to **text mode** (the `T` icon) and paste `select-row.json`.

   Then add **Data Operation → Compose**, renamed to **`Compose table rows`**, with Inputs set to this expression:
   ```
   union(createArray(json('<paste header-row.json here, on one line>')), body('Select_rows'))
   ```

5. **Microsoft Teams → Post card in a chat or channel**
   - Post as: *Flow bot*. Post in: *Group chat* (or *Channel*), then pick the chat that your
     current webhook posts to.
   - Adaptive Card: paste `card.json`.

   *Alternative:* to keep your existing webhook workflow, use **HTTP (POST)** to the webhook
   URL with body `{"type":"message","attachments":[{"contentType":"application/vnd.microsoft.card.adaptive","content": <card.json>}]}`.
   The HTTP action is a **premium** connector, so this route needs a premium license.

6. **Save → Test → Manually**. Check the card in Teams against the dashboard's
   **Open orders** tab.

## Notes and gotchas

- **Permissions:** the flow owner needs **Build** permission on DatasetSilver. The Power BI
  tenant setting *"Dataset Execute Queries REST API"* must also be on, or step 2 fails with 401/403.
- **Dates:** `TODAY()` runs in UTC in the Power BI service. At 9:30 AM ET that is the same date.
  "Rest of week" means due after today, through Friday.
- **Exclusions:** customer `GRIDPO` (grid-reset purchases) is excluded, the same as the dashboard.
- **If a query errors:** copy the error text from the run history. The usual causes are a
  renamed measure (`[OrderAmount(Open)]`, `[OrderPickAmount]`) or a renamed column.
- **Webhook link:** if you stop using your existing webhook link, turn that trigger off. Anyone
  holding its URL can post to the chat.

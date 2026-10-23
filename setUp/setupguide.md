# Setup Guide — AI Property Matching Agent

This guide explains how to configure and test the n8n workflow. It is intentionally focused on setup and operation rather than repeating the project overview.

## 1. Prerequisites

Prepare:

- An n8n instance (cloud or self-hosted).
- A Telegram bot created through BotFather.
- A Google AI Studio API key with access to a supported Gemini model.
- An Apify account and access to the housing-scraper actor used by this workflow.
- A Google account for the lead spreadsheet.
- A Gmail account connected to n8n for internal sales notifications.

Do not paste API keys directly into HTTP Request headers or commit them in workflow JSON. Configure authentication using n8n credentials or an appropriate secret/environment-variable mechanism. If a key or token has already been shared publicly or committed, revoke/rotate it before testing.

## 2. Import the workflow

1. Open n8n and create a new workflow.
2. Use **Import from File** and select a sanitized JSON export of the workflow.
3. Save the workflow, but keep it **inactive** while configuring services.
4. Reconnect credentials and reselect the correct resources in Google Sheets and Telegram nodes. Imported credential IDs are specific to the original n8n instance and should not be relied on.
5. Review each HTTP Request URL, model name, request body, and response mapping against the currently supported APIs.

The repository's `workflow/` folder contains notes about keeping a sanitized export. Add your n8n export there only after removing secrets and instance-specific credential metadata.

## 3. Configure Telegram

1. Create a bot with BotFather and copy its token.
2. In n8n, create a Telegram API credential using the token.
3. Select that credential in **Telegram Trigger**, **Telegram: Send Reply**, **Telegram: Searching Message**, and **Telegram: Ask With Quick Buttons**.
4. Save and test the trigger with a message sent to the bot.
5. Check that the incoming payload contains `message.text`, `message.chat.id`, and sender details expected by the Code nodes.

Only one workflow should own a bot's webhook at a time. If the trigger stops receiving updates, check for another workflow or service using the same bot.

## 4. Configure Gemini

The workflow calls Gemini through HTTP Request nodes named **Gemini: Extract Requirements** and **Gemini: Explain + Follow-up Draft**.

1. Create a Gemini API key in Google AI Studio.
2. Store the key securely in n8n; do not hard-code it in the node JSON.
3. Confirm the model identifier in both URLs is currently available to your account.
4. Keep the `Content-Type: application/json` header and verify the API key is sent using the authentication method recommended by the provider.
5. Execute each node with a sample request and inspect the returned `candidates[0].content.parts[0].text` structure.
6. Confirm the model returns valid JSON and that parse failures are handled safely.

The requirement extractor expects fields such as `city`, `localities`, `budget_min_inr`, `budget_max_inr`, `bhk`, `amenities`, `possession`, and `property_type`. It converts lakh/crore amounts to INR and should not invent missing values.

## 5. Configure Apify listing retrieval

1. Create or select an Apify API token.
2. Store it in n8n credentials/secrets and reference it from **Apify: Fetch Live Listings** rather than hard-coding it.
3. Verify the actor ID and endpoint are correct for your Apify account.
4. Run a test for a supported city and inspect the returned dataset items.
5. Confirm the output fields match those used in **Normalize, Filter & Rank**, including `title`, `subtitle`, `priceMin`, `priceMax`, `priceDisplay`, `address`, `propertyType`, `sellerName`, `url`, and `isVerified`. Actor output fields may differ; update the mapping if needed.
6. Review actor limits, runtime, cost, and the listing source's terms before using it regularly.

The current workflow requests listings for the normalized city and ranks up to five results. Location filtering searches the returned listing text; BHK and budget filtering depend on the availability and quality of the source fields.

## 6. Configure Google Sheets

1. Create a spreadsheet for property leads and connect Google Sheets OAuth credentials in n8n.
2. Select the spreadsheet and worksheet in **Google Sheets: Save Lead**.
3. Create a header row that matches the node's mapped columns. Recommended headers:

   - `telegram username`
   - `buyer name`
   - `phone`
   - `email`
   - `requirement text`
   - `city`
   - `localities`
   - `budget max inr`
   - `bhk`
   - `amenities`
   - `possession`
   - `top matches`
   - `top match urls`
   - `followup_draft`
   - `status`
   - `reply`

4. Check the node's column mapping against the actual header names. In particular, remove unintended leading spaces from names such as ` status` if the sheet header is `status`.
5. Run a test and verify one row is appended with the expected values.
6. Limit spreadsheet access to authorized staff because the sheet may contain personal contact details and buyer requirements.

## 7. Configure Gmail sales notifications

1. Connect Gmail OAuth credentials in n8n.
2. In **Gmail: Notify Sales (Approval)**, choose the intended internal sales recipient. Replace any example or personal address with the correct team mailbox.
3. Verify the email contains the requirement summary, top matches, and follow-up draft.
4. Keep the message as an internal review/approval notification. The workflow generates a draft; it should not be treated as buyer-approved outreach.

## 8. Test the conversation and matching logic

Test in a non-production spreadsheet before activating the workflow.

| Test message | Expected behavior |
|---|---|
| `Hi` | Ask for a property requirement and show quick-reply examples. |
| `3 BHK under 1 crore in Ahmedabad` | Extract BHK, budget, and city; search listings. |
| `90 lakh` after a previous search | Merge the budget into the chat's existing requirement, subject to session memory. |
| `add gym` | Add the amenity to the current requirement. |
| `new search` | Reset the current conversation's saved criteria. |
| An unlikely budget/location combination | Return a no-match explanation if no results survive filtering. |

Also verify:

- Both Gemini nodes return valid JSON.
- The cache-hit and cache-miss paths reach the normalizing node.
- The scraper's output is a list of listing items in the shape expected by the code.
- The BHK and budget filters do not accidentally exclude valid listings.
- Google Sheets receives one correctly mapped lead row.
- Gmail sends only to the intended internal recipient.
- Telegram responses fit Telegram's message-size limit and include valid listing URLs.
- Errors from APIs are visible in n8n execution history and do not create misleading success messages.

## 9. Known behavior to review before production

- **Session memory:** the Code node stores criteria per Telegram chat for two hours using workflow static data. Verify persistence behavior on your n8n deployment; static data behavior may differ during manual testing and concurrent executions.
- **Cache:** listings are cached per city for six hours. Cached data can be stale; make sure the bot does not imply a listing is currently available.
- **Default city:** Ahmedabad is used when no city is specified. Tell users this default or ask them to confirm their city if that would be safer for your use case.
- **Partial listing data:** area, possession status, and amenities are not reliably populated in the current normalization code. The bot should clearly disclose that these details need confirmation.
- **Property type:** the current property-type condition is effectively a soft check and does not reject mismatches. Tighten it if strict property-type matching is required.
- **Budget:** the current hard filter permits a limited amount above the budget and the score is heuristic. Make the tolerance explicit to users.
- **Duplicate leads:** the append operation does not deduplicate leads. Add a lookup/upsert step using a stable identifier if repeated submissions must not create duplicate rows.
- **Model and API compatibility:** validate the configured Gemini model IDs and the Apify actor endpoint against the providers' current documentation.

## 10. Troubleshooting

- **Telegram trigger receives nothing:** check the bot token, workflow activation, webhook ownership, and whether the test listener is running.
- **Gemini returns an error:** check the key, quota, supported model ID, request body, and response schema.
- **Apify returns no listings:** check token permissions, actor access, city input, runtime limits, and actual actor output.
- **No matches despite listings:** inspect the raw listing fields and the BHK/budget/locality filters in the execution data.
- **Google Sheets fails:** reauthorize OAuth, reselect the spreadsheet/sheet, and align header names with the mapping.
- **Gmail fails:** reauthorize OAuth and confirm the recipient and account permissions.
- **Telegram reply fails:** verify the chat ID and response length.

## 11. Go-live checklist

- [ ] Rotate any exposed API key or token.
- [ ] No secrets are embedded in the exported JSON or repository.
- [ ] All credentials are configured in the target n8n instance.
- [ ] Gemini model IDs and Apify actor endpoint have been verified.
- [ ] Spreadsheet columns and mappings have been tested.
- [ ] All test cases above pass.
- [ ] Sales notification recipient is correct.
- [ ] Buyer contact data is access-controlled.
- [ ] Listing freshness and availability disclaimers are included.
- [ ] Workflow is activated only after successful end-to-end testing.

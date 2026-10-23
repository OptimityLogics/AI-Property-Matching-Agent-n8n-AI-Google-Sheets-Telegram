# AI Property Matching Agent

An n8n-based property discovery assistant that receives buyer requirements through Telegram, extracts search criteria with Gemini, retrieves property listings through Apify, filters and ranks potential matches, and saves qualified leads to Google Sheets for sales follow-up.

## Workflow overview

1. **Telegram Trigger** receives a buyer message.
2. **Gemini: Extract Requirements** converts natural language into structured property criteria.
3. **Validate Requirements** merges follow-up messages into a per-chat session and applies defaults.
4. If the buyer has not provided a usable search criterion, the bot asks for more information with quick-reply examples.
5. **Load Cache / Cache Hit?** checks for cached city listings; otherwise, **Apify: Fetch Live Listings** retrieves listings and **Save Cache** stores them temporarily.
6. **Normalize, Filter & Rank** applies supported filters and calculates a transparent match score.
7. If matches exist, Gemini drafts an introduction and a sales follow-up message.
8. **Prepare Lead Row** formats the response and lead data; **Google Sheets: Save Lead** appends the lead.
9. **Gmail: Notify Sales (Approval)** emails the sales team with the matches and a follow-up draft for review.
10. **Final Reply** sends the match summary back to Telegram. If no matches are found, the workflow sends a no-match explanation.

## Repository structure

```text
.
├── README.md
├── setUp/
│   └── setupguide.md
└── workflow/
    └── README.md
```

- [Setup guide](setUp/setupguide.md): configure credentials, services, data columns, and test the workflow.
- [Workflow folder notes](workflow/README.md): export and store a sanitized n8n workflow JSON.

## Integrations

- n8n
- Telegram Bot API
- Google Gemini API
- Apify housing scraper actor
- Google Sheets
- Gmail

## Important limitations

- The workflow uses Ahmedabad when the buyer does not specify a city.
- Listing data may not reliably include area, amenities, or possession status; confirm these with the seller before making commitments.
- The ranking score is a heuristic, not a guarantee of suitability or listing accuracy.
- Listing retrieval and caching depend on the selected Apify actor, its output schema, and n8n execution behavior.
- The workflow should remain inactive until credentials, API access, node expressions, and all priority paths have been tested.

## Security

Never commit API keys, Apify tokens, Telegram bot tokens, OAuth credentials, private buyer data, or exported n8n credential IDs. Use n8n credentials for authentication, rotate any key that has been exposed, and publish only a sanitized workflow export.

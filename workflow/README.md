# Workflow export

Store the sanitized n8n workflow export in this folder, for example:

```text
workflow/
├── README.md
└── ai-property-matching-agent.json
```

Before committing the JSON:

1. Remove API keys and bearer tokens from HTTP Request headers.
2. Replace credential values with n8n credential references or safe placeholders.
3. Remove private instance metadata and personal data that is not required to reproduce the workflow.
4. Confirm the file is valid JSON and can be imported into a clean n8n instance.
5. Reconnect Telegram, Google Sheets, Gmail, Gemini, and Apify credentials after import.

**Security warning:** the workflow JSON shared during preparation included credential-like values for Gemini and Apify. Treat those values as exposed: revoke/rotate them before using the workflow again. Never store live secrets in this repository.

The JSON also contains instance-specific Telegram/Google credentials and spreadsheet references. These must be reselected or replaced for the target environment.

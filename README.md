# AI Property Matching Agent

An AI-powered property discovery and lead-management workflow built with **n8n, Telegram, Google Gemini, Apify, Google Sheets, and Gmail**.

The assistant collects a buyer's property requirements through Telegram, turns natural-language messages into structured search criteria, retrieves property listings, ranks potential matches, and records qualified leads for sales follow-up.

> **Project status:** Workflow configuration and documentation are being prepared. Review the setup guide and test all integrations in your own n8n instance before enabling the workflow for real users.

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Matching and Ranking](#matching-and-ranking)
- [Limitations](#limitations)
- [Security](#security)
- [Disclaimer](#disclaimer)

## Overview

Property buyers often share requirements across multiple messages, while sales teams need a consistent way to find listings and track follow-ups. This workflow aims to streamline that process by combining conversational requirement collection, property-listing retrieval, match ranking, and lead logging.

### Key Features

- **Conversational intake:** Receive buyer requirements through a Telegram bot.
- **AI requirement extraction:** Use Gemini to structure messages into fields such as city, locality, budget, BHK, property type, and contact details.
- **Requirement validation:** Ask follow-up questions when essential search information is missing.
- **Property discovery:** Retrieve listing data through an Apify actor.
- **Listing cache:** Reuse recently retrieved city listings to reduce repeated scraping.
- **Match filtering and ranking:** Apply supported criteria and return a shortlist of potential matches.
- **Lead tracking:** Append buyer and search details to Google Sheets.
- **Sales handoff:** Email the sales team a summary and a suggested follow-up for review.
- **Telegram response:** Send the match summary or a no-match message back to the buyer.

## How It Works

```text
Buyer sends a message in Telegram
                |
                v
       Gemini extracts needs
                |
                v
     Validate / clarify details
                |
                v
     Check cached property data
          /             \
      Cache hit        Cache miss
          |                |
          |          Fetch via Apify
          |                |
          |          Save listings cache
          \                /
           v              v
        Normalize, filter and rank
                |
                v
       Any suitable matches?
           /           \
         Yes            No
          |              |
          v              v
   Draft match reply   No-match reply
          |
          v
   Save lead to Google Sheets
          |
          v
   Notify sales team by Gmail
          |
          v
    Reply to buyer in Telegram
```

The exact execution order depends on the connected nodes and branch configuration in the n8n workflow. Validate the paths in your own instance before using this diagram as an operational guarantee.

## Technology Stack

| Technology | Purpose |
|---|---|
| [n8n](https://n8n.io/) | Workflow orchestration and automation |
| [Telegram Bot API](https://core.telegram.org/bots/api) | Buyer-facing chat interface |
| [Google Gemini](https://ai.google.dev/) | Extract requirements and draft match explanations |
| [Apify](https://apify.com/) | Retrieve property-listing data using a configured actor |
| [Google Sheets](https://www.google.com/sheets/about/) | Store lead and search information |
| [Gmail](https://workspace.google.com/products/gmail/) | Notify the sales team for review and follow-up |

## Repository Structure

```text
.
├── README.md
└── setUp/
    └── setupguide.md
```

- **[Setup guide](setUp/setupguide.md)** — service setup, credentials, configuration, testing, and troubleshooting.

## Getting Started

### Prerequisites

Before configuring the automation, prepare:

- An n8n instance with permission to create and execute workflows.
- A Telegram bot and bot token.
- A Google Gemini API key and access to the selected model.
- An Apify account with access to the configured property-scraping actor.
- A Google Sheets spreadsheet and appropriate Google account access.
- A Gmail account or supported Google authentication for sales notifications.

### Setup

1. Read the [setup guide](setUp/setupguide.md).
2. Configure each service and create credentials in n8n.
3. Prepare the Google Sheets columns required by the workflow.
4. Confirm the Apify actor's input parameters and output schema.
5. Import the sanitized workflow JSON from the repository when setting up the automation.
6. Test each branch with sample data before activating the workflow.

Do not activate the workflow for real buyers until the credentials, expressions, data mappings, error paths, and privacy requirements have been checked.

## Configuration

Keep environment-specific values in n8n credentials or configuration fields, not in publicly committed workflow exports.

| Setting | Purpose |
|---|---|
| Telegram credentials | Receive messages and send replies |
| Gemini credentials and model | Parse buyer requirements and generate responses |
| Apify credentials and actor | Retrieve property listings |
| Google Sheets credentials and spreadsheet | Store lead information |
| Gmail credentials and recipient | Send sales notifications |
| Default city and matching thresholds | Define fallback search behavior and ranking rules |
| Cache duration | Control how long retrieved listings are reused |

Use the [setup guide](setUp/setupguide.md) for the configuration sequence and testing checklist.

## Matching and Ranking

The workflow is designed to normalize listing data, filter it against supported buyer requirements, and calculate a heuristic match score before returning a shortlist.

Potential buyer criteria include:

- City and locality
- Minimum and maximum budget (INR)
- BHK configuration
- Property type
- Area, amenities, and possession preferences when reliable source data is available

**Important:** A field being extracted from a buyer's message does not guarantee that the listing source provides that field or that the workflow enforces it. Verify the actor's output and each filter expression before relying on a criterion.

## Limitations

- Ahmedabad is used as a fallback city when a buyer does not specify one.
- Listing availability, completeness, and freshness depend on the configured Apify actor and its data source.
- Area, amenities, and possession status may be missing or inconsistently represented in listing data.
- Match scores are heuristic rankings, not guarantees of suitability, listing accuracy, price, or availability.
- Cache behavior and workflow static-data persistence can vary by n8n configuration and execution mode.
- Lead appending may create duplicate rows unless deduplication is implemented separately.
- Sales notifications are intended for review; generated descriptions and follow-up messages should be checked before being sent externally.

Confirm property details, price, availability, ownership, and legal documentation directly with the relevant seller or authorized professional.

## Security

- **Never commit API keys, bearer tokens, Telegram bot tokens, OAuth secrets, or private buyer data.**
- Store credentials using n8n's credential system.
- Replace credential IDs, account-specific references, and sensitive spreadsheet details in any public export.
- If a secret has been exposed, revoke or rotate it immediately.
- Use sample or anonymized data for screenshots, tests, and public documentation.
- Limit access to the Google Sheet and email inbox to authorized users.
- Follow applicable privacy and data-retention requirements when handling buyer contact details.

## Disclaimer

This project is an automation example for property discovery and lead management. It does not guarantee that listings are accurate, available, legally verified, or suitable for a buyer. Human review and independent verification are required before making property or financial decisions.

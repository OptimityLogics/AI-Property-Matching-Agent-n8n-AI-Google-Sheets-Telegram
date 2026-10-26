# AI Property Matching Agent

**Real estate lead automation · n8n workflow · Telegram**  
**Document date:** 21 October 2026

## At a glance

The AI Property Matching Agent takes a buyer's property requirement from a Telegram chat, searches live listings, ranks suitable matches, saves the lead, and alerts the sales team.

- **Built with:** n8n, Telegram, Google Gemini, Apify, Google Sheets, and Gmail.
- **Testing:** Two live Telegram searches in Ahmedabad—one with no match and one with three matches.
- **Core principle:** Buyers receive a response, while the sales team receives a lead to review. Property availability is not promised without human confirmation.

## Business problem

Buyers often send their budget, BHK preference, and area in short messages. Sales teams then manually interpret the request, search portals, compare properties, enter details into a sheet, and prepare follow-ups. This takes time and can cause delays or missed details.

## Solution overview

The workflow receives a normal message through Telegram. Gemini extracts the buyer's requirements, Apify fetches live listings when a cached city search is unavailable, and the workflow filters and scores listings out of 100. The top matches are sent in Telegram. The lead is saved to Google Sheets, and Gmail sends the sales team the requirement, matches, and a follow-up draft for review. If there are no suitable listings, the buyer receives suggestions rather than an empty response.

## Workflow stages

### 1. Understand the request
- **Telegram Trigger:** Receives the buyer's message.
- **Gemini: Extract Requirements:** Extracts buyer name, phone, email, city, localities, budget, BHK, area, amenities, possession preference, and property type. Converts lakh/crore values to rupees and does not invent missing information.
- **Validate Requirements:** Combines messages from the same chat for two hours so short follow-ups such as “90 lakh” can be understood. Applies safe defaults and creates a one-line summary.
- **Need More Info? / Ask With Quick Buttons:** Asks a question only when budget, BHK, and locality are all missing; quick buttons help buyers provide details.

### 2. Find and rank properties
- **Searching Message:** Tells the buyer that the live search has started.
- **Load Cache / Cache Hit?:** Reuses city listings fetched in the previous six hours when available.
- **Apify: Fetch Live Listings:** Fetches up to 100 live listings for sale in the city when there is no cache.
- **Save Cache:** Stores fresh listings for the next search.
- **Normalize, Filter & Rank:** Filters listings and scores qualifying properties, keeping the top five.
- **Any Matches?:** Routes the workflow to the match or no-match path.

### 3. Save the lead and reply
- **Gemini: Explain + Follow-up Draft:** Writes a short, friendly introduction, notes any trade-off honestly, and drafts a site-visit follow-up.
- **Prepare Lead Row / Google Sheets: Save Lead:** Builds the buyer reply and lead record, then saves one row per search with matches.
- **Gmail: Notify Sales:** Emails the requirement, top matches, and follow-up draft for review.
- **Final Reply / Send Reply:** Sends matches to the buyer and explains that an advisor will confirm availability.
- **No Match Reply:** Suggests raising the budget, adding nearby areas, or changing the BHK.

## Matching score

Listings are first checked against hard rules: BHK must match, starting price must not exceed the budget by more than 10%, and the locality must match when specified. Qualifying listings receive a score out of 100.

| Factor | Maximum points | Scoring approach |
| --- | ---: | --- |
| Budget | 30 | Full points if the full price range is within budget; fewer points if only the starting price fits |
| Location | 25 | Full points when the requested locality matches |
| BHK | 20 | Matches the requested configuration |
| Amenities | 15 | Share of requested amenities found in the listing |
| Possession | 10 | Fits ready-to-move or under-construction preference |

Ties are broken in favour of verified listings.

## Lead information

The Google Sheets record can include Telegram username, buyer name, phone, email, requirement text, city, localities, maximum budget, BHK, amenities, possession preference, top matches and links, follow-up draft, status, and the reply sent to the buyer. The initial status is **New - awaiting sales approval**.

## Test results

The workflow was tested through Telegram with two searches in Ahmedabad.

| Search | Result |
| --- | --- |
| 3 BHK flat in Vaishnavdevi Circle, Ahmedabad, under ₹90 lakh | Passed. 100 live listings were scanned; no suitable match was found. The buyer was advised to raise the budget by 10–15%, add nearby areas, or change BHK. |
| 3 BHK in Gota, Ahmedabad, under ₹90 lakh | Passed. Three matches were returned, the requirement was saved, and the sales-advisor flow was explained. |

Matches returned in the second test:

| Property | Score | Location | Price |
| --- | ---: | --- | --- |
| The Olive | 92/100 | Near Vandematram Road, Gota | ₹72.8 lakh–₹1.03 crore |
| Khwaish Blue Bell | 92/100 | Jagatpur | ₹72.53 lakh–₹1.13 crore |
| Festival Vibes | 82/100 | Gota | About ₹91.35 lakh, slightly above budget |

## Human approval and safety

- The follow-up draft goes to the sales team first and is reviewed before being sent to the buyer.
- The workflow does not claim that property availability is confirmed; an advisor confirms availability and site visits.
- Amenities and possession status are not present in the listing data, so the advisor is expected to confirm them.
- Gemini is instructed not to invent values; information not stated by the buyer stays empty.
- Slightly over-budget matches are identified honestly.

## Limitations and production improvements

- Chat memory and listing cache are stored in n8n workflow data; production should use a proper database.
- The search depends on up to 100 listings per city, so a specific locality can return no match even when a property exists.
- Listing data does not include amenities, possession status, or area, limiting exact scoring for those criteria.
- Every search creates a new row; repeat leads should be merged in production.
- Production also needs secure credential handling and error alerts.

Recommended next improvements include a persistent database, additional listing sources, amenity and possession data, repeat-lead merging, sending approved follow-ups through WhatsApp, and a simple lead-status dashboard.

---
*Source document: AI Property Matching Agent — Optimity Logics, dated 21 October 2026.*

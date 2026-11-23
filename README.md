# Religious & Community Telegram Bot

An Arabic Telegram group bot that combines religious reminders, group moderation, saved content, and on-demand information tools.

Built in JavaScript for Cloudflare Workers, with Cloudflare D1 for group settings and stored content. Religious reminders are the main focus, supported by tools for managing the group's everyday activity.

> Source-based portfolio documentation. The source passes a JavaScript module syntax check. The shared version replaces the original hardcoded owner identifier with the `OWNER_ID` environment setting; no other bot logic was changed. Deployment, integrations, and message delivery have not been independently tested.

## Religious reminders

- Prayer reminders for Gaza, using AlAdhan prayer times and the `Asia/Gaza` timezone.
- Notifications 30 minutes before, 15 minutes before, at prayer time, and 30 minutes after. The 15-minute reminder can be disabled separately.
- Custom text for the before, at-time, and after-prayer messages.
- Optional member mentions in the 30-minute reminder, using up to 40 recorded active members.
- Morning and evening adhkar images at configurable times.
- Friday al-Kahf reminders, with an optional saved audio recording in the morning.
- A per-group Quran media library, with daily selections from surahs that have both an image and audio recording.
- Al-Mulk media at a configurable time, fixed-time Duha and Qiyam reminders, and two to five varied dhikr messages per day when enabled.

Prayer coordinates are fixed to Gaza in this version. The bot does not detect each group's location. Duha and Qiyam reminders use fixed clock times, not calculated worship-time boundaries.

## Group management

- Link detection, a configurable word filter, and rapid-message filtering.
- Custom welcome messages.
- Owner commands for warnings, muting, banning, and reversing those actions.
- Saved commands for text, photos, documents, videos, voice messages, and audio.
- One-time announcements and recurring daily messages.
- Separate settings and saved content for each group.

Administrative commands are restricted to the single Telegram owner ID configured through `OWNER_ID`. They are not automatically available to every Telegram group administrator. Moderation also depends on the permissions granted to the bot in Telegram.

## Saved knowledge and on-demand search

### Stored answers

The owner adds question-and-answer pairs. An exact normalized question match returns its stored answer directly. When Workers AI is available, the bot can also select a matching saved question from ranked candidates.

The answer still comes from the stored knowledge base. The model does not write a new FAQ answer. Selecting a stored answer does not guarantee that it is the right match or that its content is accurate.

### Web search

A separate, explicitly invoked command searches through Tavily and returns source links. If Workers AI is available, it can summarize the results in Arabic. Otherwise, the bot uses result excerpts.

Normal conversation does not automatically trigger AI replies or web searches. Search results and summaries still need human verification, especially for religious or other sensitive questions.

## Architecture

```text
Telegram update
    -> Worker /webhook
    -> Command handling and owner checks
    -> D1 / Telegram Bot API / optional AI and search

External Cron Trigger
    -> Worker scheduled handler
    -> Due reminders and messages
    -> Delivery records in D1
```

The code creates its database tables when initialized. Media is referenced through Telegram file IDs rather than bundled in the source repository.

## Runtime configuration

| Binding or secret | Purpose |
| --- | --- |
| `BOT_TOKEN` | Required Telegram bot credential |
| `DB` | Required D1 database binding |
| `OWNER_ID` | Positive integer Telegram user ID for the bot owner; required for administration |
| `AI` | Optional Workers AI binding for saved-question matching and search summaries |
| `TAVILY_API_KEY` | Optional secret for web search |
| `WEBHOOK_SECRET` | Optional secret checked against the incoming Telegram webhook header |

Set `OWNER_ID` to the intended owner's numeric Telegram user ID. If it is missing or invalid, no user receives owner privileges.

The configured AI model is `@cf/meta/llama-3.1-8b-instruct-fp8`.

The source provides `/health` and a POST `/webhook` handler. Running the bot also requires deployment configuration, Telegram webhook registration, the intended owner ID, and a Cron Trigger. The supplied source does not include that deployment configuration.

Several reminder routines check the current local minute, so the external schedule must invoke the handler frequently enough. Delivery is not guaranteed during outages or service failures. Database records reduce duplicate sends but do not provide an absolute exactly-once delivery guarantee.

## Data and privacy

The runtime database can store group settings, saved content, member IDs and first names, warnings, Telegram media IDs, and delivery records. It is not a complete automatic membership directory.

Keep credentials and database exports out of a shared repository. Configure the intended owner identifier in the deployment environment, not in the shared source. Web search sends the requested query to Tavily; AI-assisted features send their relevant inputs to Workers AI.

## Scope of this version

This repository presents the supplied JavaScript implementation. It does not include saved Quran or adhkar media, a populated knowledge base, a database export, a deployment configuration, or an automated test suite.

The documentation describes implemented logic, not independently verified production operation. Religious content and prayer-time settings should be reviewed before use.

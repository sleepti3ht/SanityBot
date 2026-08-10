# LISSKINS Market Bot

> Private case study: an async Telegram and WebSocket system for real-time marketplace monitoring.

![Telegram bot demo](assets/main-menu.gif)

## Overview

LISSKINS Market Bot is a private, closed-source automation project for monitoring marketplace events and delivering Telegram notifications based on user-defined rules.

The source code is not public because the project contains business logic, marketplace integrations, and functionality that may be used as a commercial subscription product.

## Product Capabilities

- Real-time marketplace event monitoring via WebSocket.
- Fallback market polling for missed or delayed events.
- Telegram interface for creating and managing monitoring tasks.
- Price-based filtering.
- Gem and style matching.
- Two matching modes:
  - `Exact item` — match a specific item name and rule.
  - `Any item` — match any item containing a selected gem or style.
- Current profile balance lookup through the marketplace API.
- Reconnect and timeout handling.
- VPS-ready deployment.

## Demo

### Main menu

![Main menu](assets/main-menu.gif)

### Exact item task

![Exact item task](assets/create-exact-task.gif)

### Any-item gem/style task

![Any item task](assets/create-any-item-task.gif)

### Matching notification

![Notification demo](assets/notification-demo.gif)

## Architecture

```text
Telegram UI
    │
    ├── Task creation and management
    ├── Match mode selection
    └── Notifications
          │
          ▼
      Task storage
          │
          ▼
 ┌─────────────────────┐
 │                     │
 ▼                     ▼
WebSocket listener   Market polling
 │                     │
 └──────────┬──────────┘
            ▼
     Common item processor
            │
            ▼
     Task matching layer
            │
            ▼
   Notification / purchase layer
```

## Main Components

### Telegram interface

- Async Telegram application.
- Inline keyboard navigation.
- Conversation-based task creation.
- Task deletion and bulk task removal.
- Settings management.
- Balance display.

### WebSocket listener

- Authenticated WebSocket connection.
- Push event parsing.
- Reconnect after connection errors.
- Opening-handshake timeout handling.
- Shared item-processing pipeline.

### Market polling

Polling is used as a fallback source when WebSocket delivery is delayed or interrupted.

Both WebSocket events and polling results pass through the same item-processing function. This avoids duplicating matching logic and ensures consistent notification behavior.

### Task matching

Tasks support two modes:

- `Exact item`: item name, price, and gem/style must match.
- `Any item`: item name is ignored; only price and gem/style are checked.

### API integration

The API client is responsible for:

- Profile balance requests.
- WebSocket authentication.
- Future purchase operations.

## Technology Stack

- Python
- `asyncio`
- `python-telegram-bot`
- `aiohttp`
- `websockets`
- `python-dotenv`
- JSON-based task storage during MVP
- `systemd` for VPS deployment

## Engineering Highlights

- Async Telegram polling and WebSocket processing in one application.
- Shared processing path for WebSocket and polling sources.
- Reconnect logic with timeout handling and backoff.
- Configurable user-defined matching rules.
- Separation of Telegram UI, API access, WebSocket parsing, and task matching.
- Environment-based secret configuration.
- Secret-safe logging.

## Security

- API keys and bot tokens are stored outside the source code.
- `.env` is excluded from Git.
- Sensitive headers and tokens are not included in logs or demos.
- The public repository contains only screenshots, GIFs, and an architectural overview.

## Roadmap

- [x] Telegram task management.
- [x] Exact-item matching.
- [x] Any-item gem/style matching.
- [x] WebSocket notifications.
- [x] Profile balance lookup.
- [ ] Market polling fallback.
- [ ] VPS deployment.
- [ ] Dry-run purchase mode.
- [ ] Manual purchase confirmation.
- [ ] Automatic purchase with strict safety limits.

## Project Status

- Type: private / closed-source case study
- Development stage: MVP testing
- Role: architecture, backend implementation, integrations, testing, and deployment
- Source code: private

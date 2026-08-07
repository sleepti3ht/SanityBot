# LISSKINS Market Bot (Private Project)

> Async Telegram + WebSocket bot for monitoring market events, filtering items, and sending notifications.

## Overview

This project is a **private** (closed-source) bot that:

- Connects to a marketplace via WebSocket and listens to real-time item updates.
- Applies user-defined filters (`tasks`) based on price, gem/style and match mode.
- Sends Telegram notifications and is designed to support auto-buy logic later.

The source code is kept private due to business logic and integrations, but this README describes the architecture and technologies used.

## Tech Stack

- **Python** 3.11+
- **python-telegram-bot** (async `Application`, `ConversationHandler`)
- **aiohttp**, **websockets** — HTTP client and WebSocket listener
- **python-dotenv** — configuration via `.env`
- **systemd** — 24/7 deployment on a VPS

## Architecture

- `telegram_bot.py` — Telegram bot:
  - Main menu with inline buttons.
  - Conversation flows for creating tasks and updating settings.
  - Match modes:
    - **Exact item** — filter by item name + rule (gem/style + max price).
    - **Any item** — filter by gem/style and price, independent of item name.
  - Balance command (`My Balance`) integrated with the HTTP API (configurable endpoint).

- `websocket_parser.py` — WebSocket listener:
  - Fetches a token and connects to the marketplace WebSocket.
  - Processes `push` messages, extracts item payloads.
  - Matches items against user tasks and triggers Telegram notifications.
  - Implements reconnect with backoff and timeout handling.

- `tasks.py` — task storage and matching:
  - User tasks are stored in `tasks.json`.
  - Matching logic supports both exact-name and any-item modes.
  - Helpers for gem and style extraction.

- `lisskins_client.py` — HTTP API integration:
  - Placeholder for balance fetching (endpoint configurable via `.env`).
  - Designed to be extended with purchase endpoints.

## Features

- Create tasks directly in Telegram:
  - Choose match mode: **Exact item** / **Any item with gem/style**.
  - Define max price and rule type (`gem` / `required_style`).
- WebSocket-driven notifications for matching items.
- Secret-safe logging:
  - Sensitive values (`API_KEY`, tokens, authorization headers) are excluded or masked.
- Ready for deployment on a VPS:
  - Single `main.py` entry point: Telegram polling + WebSocket loop.
  - `systemd` service configuration for automatic restart.

## Security & Logging

- Secrets are stored only in `.env` (not committed to Git).
- Logs avoid printing raw secrets and full HTTP headers.
- Optional logging filter masks common secret patterns (e.g. `api_key=...`, `token=...`).

## Deployment

The bot is designed to run 24/7 on a Linux VPS:

- Create a Python virtual environment and install dependencies.
- Provide a `.env` with `BOT_TOKEN`, `API_KEY`, WebSocket URL, etc.
- Create a `systemd` unit for `main.py` with `Restart=always`.
- Optionally configure HTTP(S)/SOCKS5 proxy for Telegram only.

## Status

- **Type:** private / closed-source
- **Role:** architecture, backend, deployment
- **Open to discussion** in interviews (without exposing business-sensitive details).

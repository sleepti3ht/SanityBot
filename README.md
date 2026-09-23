# 🎯 SanityBot (LIS-SKINS Trading Bot)

> **High-performance asynchronous trading bot for [LIS-SKINS](https://lis-skins.com)** with real-time WebSocket notifications and instant auto-purchase. No `check_availability()` delays — direct purchase saves 200–400ms per item.

![python](https://img.shields.io/badge/Python-3.12%2B-blue)
![status](https://img.shields.io/badge/status-active-success)
![async](https://img.shields.io/badge/async-asyncio-009688)

---

## ✨ Features

### Core
- **Real-time WebSocket notifications** — 0–100ms latency for new listings
- **Instant auto-purchase** — direct `POST /market/buy` without `check_availability()` (saves 200–400ms)
- **Flexible task system** — filter by item name, gems, styles, max price
- **Hybrid polling + WebSocket** — backup REST polling every 15s for reliability
- **Mass import/export** — add 100+ tasks via text: `Item Name;max_price;max_quantity;rule_type;rule_value`
- **Quick quantity adjustment** — `➕` `➖` buttons in task list
- **Automatic retry** — handles network errors and rate limits (HTTP 429)
- **Steam seller database** — auto-collects SteamIDs of sellers with gems/styles
- **Telegram bot** — manage tasks and receive notifications

### Reliability
- **Single async processing worker** — preserves predictable purchase ordering
- **In-process duplicate protection** — shared bounded `OrderedDict` caches prevent repeated processing from WebSocket and polling
- **Atomic JSON replacement** — writes task updates through a temporary file and `os.replace()`
- **Retry handling** — retries selected network failures and respects HTTP 429 retry delays
- **Seller SQLite database** — records discovered seller SteamIDs and listing metadata when available in item payloads

**Result:** Semi-automated pipeline with manual control over high-value items

---

## 🛠 Tech Stack

| Technology | Purpose |
|---|---|
| Python 3.12+ | Core language |
| aiohttp | Async HTTP requests |
| Centrifuge | WebSocket client for real-time notifications |
| python-telegram-bot | Telegram bot framework |
| SQLite | Steam seller database |
| asyncio | Async architecture |
| JSON | Task storage (atomic writes) |

---

## 📊 Architecture

```mermaid
graph TD
%% --- Styles ---
classDef source fill:#2d3748,stroke:#4a5568,color:#fff,stroke-width:2px;
classDef core fill:#4c51bf,stroke:#434190,color:#fff,stroke-width:2px;
classDef auto fill:#38b2ac,stroke:#319795,color:#fff;
classDef analytics fill:#ed8936,stroke:#dd6b20,color:#fff;
classDef user fill:#f56565,stroke:#c53030,color:#fff;

%% --- Block 1: Sources & Core Engine ---
subgraph "1. Data Collection & Core Engine"
A[LIS-SKINS Platform]:::source -->|WebSocket 0-100ms| B[Sanity Bot Core]:::core
A -->|REST API 15s| B
B --> C[Auto-Buyer Engine<br/>~Workers + Purchase Lock + ID Deduplication]:::auto
C --> D[Telegram Alerts]:::auto
end

%% --- Block 2: Analytics Pipeline ---
subgraph "2. Analytics & Parser Pipeline"
B -->|Collects SteamIDs| E[(Bot Database<br/>Sellers with Gems/Styles)]:::analytics
E --> F[steamid.txt]:::analytics
F --> G[Parser Bot<br/>Targeted Scraping]:::analytics
G --> H[CSV / Excel<br/>Filtered Rare Lots]:::analytics
end

%% --- Block 3: Manual Control & Feedback ---
subgraph "3. Manual Control & Tasks"
H --> I((Manual Review<br/>Excel Analysis)):::user
I -->|Adds high-value items| J[Task System<br/>Filters & Thresholds]:::core
J --> C
D --> I
end

%% --- Connections ---
C -.->|Logs purchases| E
```
---
### Data Pipeline
1. **Collection** — WebSocket collects seller SteamIDs → Bot Database
2. **Parsing** — Parser bot scans specific items from steamid.txt → CSV
3. **Manual Review** — Filter promising items in Excel
4. **Auto-Trading** — Add to tasks (via import or UI) → Sanity Bot auto-purchases

## Architecture trade-offs

### Why JSON for tasks?
- **Simplicity:** Tasks are stored in one readable and portable `tasks.json` file.
- **Small-scale workload:** For a personal bot with a limited number of tasks, JSON is sufficient and easy to back up.
- **Safe replacement:** Task updates use a temporary file followed by `os.replace()` to avoid partial JSON files if a write is interrupted.
- **Trade-off:** JSON is not a replacement for a transactional multi-process database. The bot is designed to run as one process.

### Why bounded in-memory deduplication?
- **Goal:** Avoid processing the same listing twice when it arrives through WebSocket, polling, or repeated events.
- **Mechanism:** `IN_FLIGHT_IDS` blocks simultaneous handling; `PROCESSED_IDS` remembers recently processed IDs.
- **Memory safety:** Both caches have maximum sizes and evict the oldest IDs when full.
- **Trade-off:** This is in-process best-effort deduplication, not durable idempotency across restarts.

---

## ⚙️ Configuration and tuning

The bot prioritizes fast WebSocket handling and keeps REST polling as a fallback for existing listings.

- `WORKER_COUNT`: Number of queue workers for item processing. The current default is `1`, which keeps purchase attempts ordered and works with the global purchase lock.
- `POLL_INTERVAL_SECONDS`: Delay between REST polling cycles. The current value is `15`.
- `MAX_CURSOR_PAGES_PER_TASK`: Maximum number of pagination pages scanned for one exact-name task. The current value is `15`.
- `MAX_PROCESSED`: Maximum number of recently processed listing IDs kept in memory.
- `MAX_IN_FLIGHT`: Maximum number of listing IDs currently marked as being processed.
- `ITEM_QUEUE` size: Maximum number of incoming items waiting for processing before new events are dropped and logged.

Increasing polling intensity or worker count may increase API load, rate-limit risk, and duplicate-purchase complexity. Measure with logs before changing these values.

| Optimization | Impact |
|--------------|--------|
| WebSocket instead of polling | 0–100ms vs 15s |
| No `check_availability()` | -200–400ms per purchase |
| In-memory task cache | -10–50ms per item |
| 3 async workers | Parallel processing |
| Retry + backoff | Stability on 429 |
| Atomic JSON writes | No file corruption |
| TTLCache for PROCESSED_IDS | No OOM crashes |

---

## 📈 Performance

Live timings from the bot console (one `Scan pass` = one REST polling cycle):

```log
2026-09-08 07:09:57,273 [INFO] lis-market-poller: Scan pass: min=0ms max=363ms avg=187ms | checked=232 matched=0
2026-09-08 07:10:24,083 [INFO] lis-market-poller: Polling scan_once started, enabled=True
2026-09-08 07:10:35,983 [INFO] lis-market-poller: Scan pass: min=0ms max=348ms avg=188ms | checked=232 matched=0
```

**Typical numbers:**

| Metric | Value |
|---|---|
| Polling cycle (REST fallback) | ~15s between scans |
| Items checked per pass | ~232–904 |
| Pass duration (min / avg / max) | ~0ms / ~187–188ms / ~348–363ms |
| Items matched per pass | 0–5 (waiting for live signals) |
| 429 / rate-limit errors | 0 |

- **Item processing time:** 100–250ms
- **Polling cycle time:** ~15s
- **429 errors:** 0
- **Profit:** 300–1500% on rare items

---

## 📱 User Interface & Observability

**31+ million items checked** with full transparency — all metrics accessible via Telegram:

### Real-Time Monitoring:
- **Volume Metrics** — total checked, matched, sent to processor
- **Latency Tracking** — min/max/avg per scan cycle (typical: 0ms/3s/99ms)
- **Autobuy Status** — toggle on/off without restart
- **Polling Controls** — enable/disable REST fallback independently

### Example Dashboard:
![SanityBot Statistics](screenshots/statistics.png)
---

## 🎯 Killer Features

1. **Self-populating bot database** — auto-collects SteamIDs of sellers with gems/styles via WebSocket
2. **Pre-scan marketplace** — check database for potential items before launching auto-buy
3. **Hybrid polling + WebSocket** — maximum speed + reliability
4. **Zero-duplicate protection** — `PROCESSED_IDS` with TTL prevents double-buying
5. **Atomic JSON writes** — protects `tasks.json` from corruption
6. **Mass import/export** — add 100+ tasks via text in one message

---

## 📝 Telegram Commands

| Command | Description |
|---|---|
| `/start` | Open main menu |
| `📋 My Tasks` | View tasks with `➕` `➖` buttons |
| `📝 Create Task` | Create task via wizard |
| `📥 Import Tasks` | Mass import: `Item Name;max_price;max_quantity;rule_type;rule_value` |
| `📤 Export Tasks` | Export all tasks to text |
| `💰 My Balance` | Show balance |
| `📊 Statistics` | Show polling stats |
| `🤖 Autobuy: ON/OFF` | Toggle auto-purchase |

---

## ⚙️ Configuration

```bash
# .env
API_KEY=your_api_key
BOT_TOKEN=your_telegram_bot_token
CHAT_ID=your_telegram_chat_id
WS_URL=wss://ws.lis-skins.com/connection/websocket
TRADE_PARTNER=your_steam_partner
TRADE_TOKEN=your_steam_token
ALLOWED_USER_IDS=your_telegram_user_id
```

> ⚠️ Never commit `.env` — keep secrets private.

---

## 📸 Screenshots

### Telegram Bot
![Telegram Bot](screenshots/menu.png)

### Auto-purchase Log
![Auto-purchase Log](screenshots/alert.png)

### Task Import
![Task Import](screenshots/import.png)

### Settings
![Settings](screenshots/settings.png)

---

## 🔮 Roadmap

- [ ] **Multi-account support** — parallel purchases from multiple accounts
- [ ] **Web dashboard** — stats, charts, task management
- [ ] **Integration with other marketplaces** — Steam, CS.MONEY, etc.
- [ ] **Balance check before purchase** — prevent capital lockup

---


## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a PR.

## 📧 Contact

- **Telegram:** @sleept1ght
- **Email:** sleepti3ht@gmail.com

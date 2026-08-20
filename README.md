# LIS-SKINS Trading Bot

High-performance asynchronous trading bot for [LIS-SKINS](https://lis-skins.com) marketplace with real-time WebSocket notifications and instant auto-purchase.

## 🚀 Features

### Core

- **Real-time WebSocket notifications** — 0–100ms latency for new listings
- **Instant auto-purchase** — no `check_availability()` delays (saves 200–400ms)
- **Flexible task system** — filter by item name, gems, styles, max price
- **Hybrid polling + WebSocket** — backup polling every 10–30s for reliability
- **Task caching** — in-memory cache to minimize file I/O delays
- **Automatic retry** — handles network errors and rate limits (HTTP 429)
- **Steam seller database** — auto-collects SteamIDs of sellers with gems/styles
- **Telegram bot** — manage tasks and receive notifications

### Advanced

- **3 async workers** — parallel item processing
- **Duplicate protection** — `seen_ids` prevents double-buying
- **Rate limit handling** — exponential backoff on 429 errors
- **Self-populating bot database** — collects seller data for pre-scan analysis
- **Pre-scan marketplace** — check database before launching auto-buy

## 🛠 Tech Stack

- **Python 3.12+**
- **aiohttp** — async HTTP requests
- **Centrifuge** — WebSocket client for real-time notifications
- **aiogram** — Telegram bot framework
- **SQLite** — task and stats storage
- **asyncio** — async architecture

## 📊 Architecture
LIS-SKINS Platform
↓
├── REST API (polling 10-30s)
└── WebSocket (0-100ms)
↓
Sanity Trading Bot
↓
┌───────┴───────┐
↓ ↓
Auto-Buyer Task Cache
(3 workers) (In-memory)
↓
├── SQLite DB (tasks/stats)
├── seen_ids (duplicates)
└── Telegram Bot (alerts)

## 🔧 Key Optimizations

| Optimization | Impact |
|--------------|--------|
| WebSocket instead of polling | 0–100ms vs 10–30s |
| No `check_availability()` | -200–400ms per purchase |
| In-memory task cache | -10–50ms per item |
| 3 async workers | Parallel processing |
| Retry + backoff | Stability on 429 |
| Sequential requests | 0 429 errors |

## 📈 Performance

- **Item processing time:** 100–250ms
- **Polling cycle time:** 8–9 seconds
- **429 errors:** 0
- **Profit:** 400–500% on rare items

## 🎯 Killer Features

1. **Self-populating bot database** — auto-collects SteamIDs of sellers with gems/styles via WebSocket
2. **Pre-scan marketplace** — check database for potential items before launching auto-buy
3. **Hybrid polling + WebSocket** — maximum speed + reliability
4. **Zero-duplicate protection** — `seen_ids` prevents double-buying

## 📝 Telegram Commands

- `/add` — add a task (name, gem, style, max price)
- `/list` — show active tasks
- `/del` — delete a task
- `/clear` — clear all tasks
- `/balance` — show balance
- `/stats` — purchase statistics

## ⚙️ Configuration

```bash
# .env
API_KEY=your_api_key
LISSKINS_API_URL=https://api.lis-skins.com
WS_URL=wss://ws.lis-skins.com/connection/websocket
TRADE_PARTNER=your_steam_partner
TRADE_TOKEN=your_steam_token
```

## 📸 Screenshots

### Telegram Bot

![Telegram Bot](screenshots/menu.png)

### Auto-purchase Log

![Auto-purchase Log](screenshots/alert.png)

### Settings

![Seller Database](screenshots/settings.png)

## 🔮 Roadmap

- [ ] Multi-account support — parallel purchases from multiple accounts
- [ ] Web dashboard — stats, charts, task management
- [ ] Integration with other marketplaces — Steam, CS.MONEY, etc.


## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a PR.

## 📧 Contact

- **Telegram:** @sleept1ght
- **Email:** sleepti3ht@gmail.com

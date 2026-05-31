# FinAlly — AI Trading Workstation

## Project Specification

## 1. Vision

FinAlly (Finance Ally) is a visually stunning AI-powered trading workstation that streams live market data, lets users trade a simulated portfolio, and integrates an LLM chat assistant that can analyze positions and execute trades on the user's behalf. It looks and feels like a modern Bloomberg terminal with an AI copilot.

This is the capstone project for an agentic AI coding course. It is built entirely by Coding Agents demonstrating how orchestrated AI agents can produce a production-quality full-stack application. Agents interact through files in `planning/`.

## 2. User Experience

### First Launch

The user runs a single Docker command (or a provided start script). A browser opens to `http://localhost:8000`. No login, no signup. They immediately see:

- A watchlist of 10 default tickers with live-updating prices in a grid
- $10,000 in virtual cash
- A dark, data-rich trading terminal aesthetic
- An AI chat panel ready to assist

### What the User Can Do

- **Watch prices stream** — prices flash green (uptick) or red (downtick) with subtle CSS animations that fade
- **View sparkline mini-charts** — price action beside each ticker in the watchlist, accumulated on the frontend from the SSE stream since page load (sparklines fill in progressively)
- **Click a ticker** to see a larger detailed chart in the main chart area
- **Buy and sell shares** — market orders only, instant fill at current price, no fees, no confirmation dialog
- **Monitor their portfolio** — a heatmap (treemap) showing positions sized by weight and colored by P&L, plus a P&L chart tracking total portfolio value over time
- **View a positions table** — ticker, quantity, average cost, current price, unrealized P&L, % change
- **Chat with the AI assistant** — ask about their portfolio, get analysis, and have the AI execute trades and manage the watchlist through natural language
- **Manage the watchlist** — add/remove tickers manually or via the AI chat

### Visual Design

- **Dark theme**: backgrounds around `#0d1117` or `#1a1a2e`, muted gray borders, no pure black
- **Price flash animations**: brief green/red background highlight on price change, fading over ~500ms via CSS transitions
- **Connection status indicator**: a small colored dot (green = connected, yellow = reconnecting, red = disconnected) visible in the header
- **Professional, data-dense layout**: inspired by Bloomberg/trading terminals — every pixel earns its place
- **Responsive but desktop-first**: optimized for wide screens, functional on tablet

### Color Scheme
- Accent Yellow: `#ecad0a`
- Blue Primary: `#209dd7`
- Purple Secondary: `#753991` (submit buttons)

## 3. Architecture Overview

### Single Container, Single Port

```
┌─────────────────────────────────────────────────┐
│  Docker Container (port 8000)                   │
│                                                 │
│  FastAPI (Python/uv)                            │
│  ├── /api/*          REST endpoints             │
│  ├── /api/stream/*   SSE streaming              │
│  └── /*              Static file serving         │
│                      (Next.js export)            │
│                                                 │
│  SQLite database (volume-mounted)               │
│  Background task: market data polling/sim        │
└─────────────────────────────────────────────────┘
```

- **Frontend**: Next.js with TypeScript, built as a static export (`output: 'export'`), served by FastAPI as static files
- **Backend**: FastAPI (Python), managed as a `uv` project
- **Database**: SQLite, single file at `/app/db/finally.db` in Docker, stored in a named volume for persistence
- **Real-time data**: Server-Sent Events (SSE) — simpler than WebSockets, one-way server→client push, works everywhere
- **AI integration**: LiteLLM → OpenRouter, with validated structured JSON for trade execution
- **Market data**: Environment-variable driven — simulator by default, Alpaca IEX data if credentials are provided

### Why These Choices

| Decision | Rationale |
|---|---|
| SSE over WebSockets | One-way push is all we need; simpler, no bidirectional complexity, universal browser support |
| Static Next.js export | Single origin, no CORS issues, one port, one container, simple deployment |
| SQLite over Postgres | Anonymous browser profiles share one low-volume local service; SQLite remains self-contained and zero-config |
| Single Docker container | Students run one command; no docker-compose for production, no service orchestration |
| uv for Python | Fast, modern Python project management; reproducible lockfile; what students should learn |
| Market orders only | Eliminates order book, limit order logic, partial fills — dramatically simpler portfolio math |

---

## 4. Directory Structure

```
finally/
├── frontend/                 # Next.js TypeScript project (static export)
├── backend/                  # FastAPI uv project (Python)
│   └── schema/               # SQL schema definitions, seed data, migration logic
├── planning/                 # Project-wide documentation for agents
│   ├── PLAN.md               # This document
│   └── ...                   # Additional agent reference docs
├── scripts/
│   ├── start_mac.sh          # Launch Docker container (macOS/Linux)
│   ├── stop_mac.sh           # Stop Docker container (macOS/Linux)
│   ├── start_windows.ps1     # Launch Docker container (Windows PowerShell)
│   └── stop_windows.ps1      # Stop Docker container (Windows PowerShell)
├── test/                     # Playwright E2E tests + docker-compose.test.yml
├── db/                       # Optional local bind-mount target for SQLite development
│   └── .gitkeep              # Directory exists in repo; finally.db is gitignored
├── Dockerfile                # Multi-stage build (Node → Python)
├── docker-compose.yml        # Optional convenience wrapper
├── .env                      # Environment variables (gitignored, .env.example committed)
└── .gitignore
```

### Key Boundaries

- **`frontend/`** is a self-contained Next.js project. It knows nothing about Python. It talks to the backend via `/api/*` endpoints and `/api/stream/*` SSE endpoints. Internal structure is up to the Frontend Engineer agent.
- **`backend/`** is a self-contained uv project with its own `pyproject.toml`. It owns all server logic including database initialization, schema, seed data, API routes, SSE streaming, market data, and LLM integration. Internal structure is up to the Backend/Market Data agents.
- **`backend/schema/`** contains SQL schema definitions and seed logic. FastAPI lifespan startup creates missing tables; the first request from each new visitor seeds that visitor's default data.
- **`db/`** at the top level is an optional local bind-mount target, separate from the backend source tree. Production-style Docker commands use the named volume `finally-data:/app/db`.
- **`planning/`** contains project-wide documentation, including this plan. All agents reference files here as the shared contract.
- **`test/`** contains Playwright E2E tests and supporting infrastructure (e.g., `docker-compose.test.yml`). Unit tests live within `frontend/` and `backend/` respectively, following each framework's conventions.
- **`scripts/`** contains start/stop scripts that wrap Docker commands.

---

## 5. Environment Variables

```bash
# Required: OpenRouter API key for LLM chat functionality
OPENROUTER_API_KEY=your-openrouter-api-key-here

# Optional: Alpaca paper trading API credentials for real market data
# If not set, the built-in market simulator is used
ALPACA_API_KEY=
ALPACA_API_SECRET=
ALPACA_BASE_URL=https://paper-api.alpaca.markets/v2

# Optional: operational tuning
DATABASE_PATH=/app/db/finally.db
PRICE_STALE_AFTER_SECONDS=15
MARKET_DATA_MAX_SYMBOLS=30
OPENROUTER_MODEL=deepseek/deepseek-v4-flash:free
OPENROUTER_FALLBACK_MODEL=openai/gpt-oss-120b:free
OPENROUTER_TIMEOUT_SECONDS=30

# Optional: Set to "true" for deterministic mock LLM responses (testing)
LLM_MOCK=false
```

### Behavior

- If `ALPACA_API_KEY` and `ALPACA_API_SECRET` are set and non-empty → backend uses Alpaca market data (WebSocket stream for real-time prices, REST for historical bars)
- If Alpaca keys are absent or empty → backend uses the built-in market simulator
- If `LLM_MOCK=true` → backend returns deterministic mock LLM responses (for E2E tests)
- On startup, if `OPENROUTER_API_KEY` is missing and `LLM_MOCK=false`, the backend logs a clear error and the `/api/chat` endpoint returns a 503 with an explanatory message
- `MARKET_DATA_MAX_SYMBOLS` caps the globally subscribed symbol union. It defaults to Alpaca Basic's 30-symbol limit and is also enforced by the simulator so the branch is testable.
- `PRICE_STALE_AFTER_SECONDS` is enforced by manual and AI trades. Missing or stale prices reject execution.
- The backend reads local development values from `.env`; Docker receives them through `--env-file .env`. Commit `.env.example`, never a populated `.env`.

---

## 6. Market Data

### Two Implementations, One Interface

Both the simulator and the Alpaca client implement the same abstract interface. The backend selects which to use based on environment variables. All downstream code (SSE streaming, price cache, frontend) is agnostic to the source.

**Abstract interface** (both implementations must conform):
```python
class MarketDataProvider:
    async def start(self) -> None: ...          # begin streaming/polling
    async def stop(self) -> None: ...           # clean shutdown
    async def subscribe(self, ticker: str) -> None: ...    # add ticker to active set
    async def unsubscribe(self, ticker: str) -> None: ...  # remove ticker
    def get_price(self, ticker: str) -> PriceSnapshot | None: ...  # latest from cache
    async def get_history(
        self, ticker: str, timeframe: str, limit: int
    ) -> list[OHLCVBar]: ...

# Money values are integer microdollars internally and formatted to cents in the UI.
# PriceSnapshot: {
#   ticker, price_micros, prev_price_micros, previous_close_micros,
#   timestamp, source, change_direction
# }
# OHLCVBar: {
#   ticker, timeframe, timestamp, open_micros, high_micros,
#   low_micros, close_micros, volume
# }
```

### Executable Price Policy

- The displayed and executable market price is the latest **trade** price. Alpaca subscribes to trade updates; the simulator emits equivalent synthetic trade ticks.
- Quote data may be added later for bid/ask display, but it is not used to fill orders in the core build.
- Every fill records the price source and source snapshot timestamp.
- A trade is rejected when no price exists or the snapshot is older than `PRICE_STALE_AFTER_SECONDS`.
- The API also exposes the previous close so the frontend can calculate daily change separately from tick-to-tick direction.
- The simulator continues producing ticks at all times. Alpaca-backed trades are rejected outside market hours once the last trade becomes stale; the UI shows a stale-price state instead of presenting an old fill as current.

### Simulator (Default)

- Generates prices using geometric Brownian motion (GBM) with configurable drift and volatility per ticker
- Updates at ~500ms intervals
- Uses deterministic seeded randomness in the first milestone so tests are reproducible
- Correlated ticker moves and occasional 2-5% random events are polish-stage enhancements
- Starts from realistic seed prices (e.g., AAPL ~$190, GOOGL ~$175, etc.)
- Aggregates ticks into OHLCV bars and retains a bounded in-memory ring buffer per subscribed ticker for chart history
- Runs as an in-process background task — no external dependencies

### Alpaca API (Optional)

- Uses Alpaca's **WebSocket stream** (`wss://stream.data.alpaca.markets/v2/iex`) for real-time prices — more accurate and efficient than REST polling
- Authenticates with `ALPACA_API_KEY` / `ALPACA_API_SECRET` via the WebSocket auth message
- Subscribes to trade updates for the active ticker set; dynamically subscribes/unsubscribes as the global watchlist symbol union changes
- Alpaca Basic (IEX feed) supports up to 30 simultaneous symbol subscriptions. The cap applies globally across all users because the backend shares one upstream connection.
- Historical bars fetched via REST: `GET https://data.alpaca.markets/v2/stocks/{symbol}/bars` (used by the main chart history endpoint)
- Parses WebSocket trades and REST bar responses into the same `PriceSnapshot` and `OHLCVBar` formats as the simulator
- Handles reconnects by re-authenticating and restoring the active subscription set

### Shared Price Cache

- The active `MarketDataProvider` writes to an in-memory price cache keyed by ticker
- The cache holds the latest trade price, previous trade price, previous close, timestamp, and source for each ticker
- SSE streams read from this cache and push updates to connected clients
- The backend maintains a ticker reference count across all watchlists. A `0 → 1` transition calls `provider.subscribe(ticker)` and a `1 → 0` transition calls `provider.unsubscribe(ticker)`.
- Startup rebuilds reference counts from persisted watchlists before restoring upstream subscriptions.
- Adding a ticker that would exceed `MARKET_DATA_MAX_SYMBOLS` returns a clear validation error. Symbols already subscribed by another user do not consume an additional slot.

### SSE Streaming

- Endpoint: `GET /api/stream/prices`
- Long-lived SSE connection; client uses native `EventSource` API
- The server scopes the stream to the visitor cookie. It does not accept a client-supplied `user_id`.
- A single provider update task fans out changed snapshots to connected clients; SSE handlers do not independently poll the full cache.
- The server sends changed price events as they arrive and sends heartbeat comments during quiet periods to keep the connection alive.
- Each SSE event contains ticker, latest trade price, previous trade price, previous close, timestamp, source, and change direction
- Client handles reconnection automatically (EventSource has built-in retry)

---

## 7. Database

### SQLite Initialization And Lifecycle

FastAPI lifespan startup checks the SQLite database, creates missing schema, enables foreign keys, configures WAL mode and a 5-second busy timeout, rebuilds shared ticker subscriptions, starts the selected market-data provider, and launches background jobs. Shutdown cancels jobs and cleanly stops the provider. This means:

- No separate migration step
- No manual database setup
- Fresh Docker volumes start with a clean database automatically
- Each new visitor is seeded on their first request

### Multi-User Design

All user-owned tables include a `user_id` column. The backend assigns a UUID on first browser visit and stores it in an `HttpOnly`, `SameSite=Lax` cookie with a one-year lifetime. Each browser profile is isolated with its own portfolio, watchlist, and chat history. This is demo isolation, not authentication. The server reads the cookie for every API request and never trusts a client-supplied `user_id`.

### Numeric Representation

- Money values are stored as integer microdollars (`*_micros`) to avoid binary floating-point drift while preserving sub-cent average costs.
- Quantities are stored as integer microshares (`quantity_micros`), supporting up to six decimal places.
- Cash settlement and displayed totals round to cents at explicit service boundaries.
- Tickers are trimmed, uppercased, and validated against an allowlisted symbol pattern before storage or provider subscription.

### Schema

**user_profile** — User state (cash balance)
- `id` TEXT PRIMARY KEY (UUID, assigned on first visit)
- `cash_balance_micros` INTEGER (default: `10000000000`, representing `$10,000.00`)
- `created_at` TEXT (ISO timestamp)
- `last_seen_at` TEXT (ISO timestamp)

**watchlist** — Tickers the user is watching
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT NOT NULL REFERENCES `user_profile(id)` ON DELETE CASCADE
- `ticker` TEXT
- `added_at` TEXT (ISO timestamp)
- UNIQUE constraint on `(user_id, ticker)`

**positions** — Current holdings (one row per ticker per user)
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT NOT NULL REFERENCES `user_profile(id)` ON DELETE CASCADE
- `ticker` TEXT
- `quantity_micros` INTEGER (fractional shares supported up to six decimal places)
- `avg_cost_micros` INTEGER
- `updated_at` TEXT (ISO timestamp)
- UNIQUE constraint on `(user_id, ticker)`

**trades** — Trade history (append-only log)
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT NOT NULL REFERENCES `user_profile(id)` ON DELETE CASCADE
- `ticker` TEXT
- `side` TEXT (`"buy"` or `"sell"`)
- `quantity_micros` INTEGER
- `price_micros` INTEGER
- `price_source` TEXT (`"simulator"` or `"alpaca_iex_trade"`)
- `price_timestamp` TEXT (source snapshot ISO timestamp)
- `executed_at` TEXT (ISO timestamp)

**portfolio_snapshots** — Portfolio value over time (for P&L chart). Recorded every 30 seconds for users seen in the last five minutes or users with open positions, and immediately after each trade execution. Inserts are batched in one transaction. An hourly cleanup task deletes rows older than 24 hours in batches.
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT NOT NULL REFERENCES `user_profile(id)` ON DELETE CASCADE
- `total_value_micros` INTEGER
- `recorded_at` TEXT (ISO timestamp)

**chat_messages** — Conversation history with LLM
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT NOT NULL REFERENCES `user_profile(id)` ON DELETE CASCADE
- `role` TEXT (`"user"` or `"assistant"`)
- `content` TEXT
- `actions` TEXT (JSON — full ordered record of requested actions and execution results; null for user messages. Intentionally kept for complete audit trail even though trades also appear in the `trades` table.)
- `created_at` TEXT (ISO timestamp)

**Required indexes**
- `watchlist(user_id, ticker)`
- `positions(user_id, ticker)`
- `trades(user_id, executed_at)`
- `portfolio_snapshots(user_id, recorded_at)`
- `chat_messages(user_id, created_at)`

Trade execution is atomic: cash validation and update, position update or removal, append-only trade insertion, and immediate portfolio snapshot insertion occur in one SQLite transaction. Missing or stale prices, non-positive quantities, malformed tickers, insufficient cash, and insufficient shares reject the entire trade.

Retention policy: portfolio snapshots are retained for 24 hours. Chat messages and inactive visitor profiles are retained for 30 days, then removed by the hourly cleanup task. Cascading deletes remove associated user-owned rows.

### Default Seed Data

On first visit, the backend creates a new `user_profile` row for the assigned UUID with a `$10,000.00` cash balance and seeds ten default watchlist entries: AAPL, GOOGL, MSFT, AMZN, TSLA, NVDA, META, JPM, V, NFLX.

---

## 8. API Endpoints

### Market Data
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/stream/prices` | SSE stream of live price updates (scoped to user's watchlist via cookie) |
| GET | `/api/prices/history/{ticker}` | OHLCV bar history for a ticker (powers main chart). Delegates to the active provider. Query params: `timeframe` (default `1Min`), `limit` (default `120`, maximum `500`) |

### Portfolio
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/portfolio` | Current positions, cash balance, total value, unrealized P&L |
| POST | `/api/portfolio/trade` | Execute one atomic trade: `{ticker, quantity, side}`. Quantity is a positive decimal with at most six places. |
| GET | `/api/portfolio/history` | Portfolio value snapshots over time (for P&L chart) |
| GET | `/api/portfolio/trades` | Full trade history log: all buy/sell executions for the user, newest first |

### Watchlist
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/watchlist` | Current watchlist tickers with latest prices |
| POST | `/api/watchlist` | Add a normalized ticker: `{ticker}`. Returns the existing entry for duplicates; rejects a new globally unique symbol when the provider cap is full. |
| DELETE | `/api/watchlist/{ticker}` | Remove a ticker |

### Chat
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/chat` | Send a message, receive complete JSON response (message + ordered action results) |

### System
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | Health check (for Docker/deployment) |

### API Contracts

- Every endpoint uses Pydantic request and response models.
- Errors use a consistent JSON shape: `{"error": {"code": "machine_readable_code", "message": "Human-readable explanation", "details": {}}}`.
- Tickers are trimmed, uppercased, and validated before lookup.
- Numeric JSON responses and SSE events expose decimal strings for money and fractional quantities so JavaScript does not introduce floating-point drift.
- History `timeframe` values come from an allowlist supported by both providers; unsupported values return a validation error.
- API routers are mounted before the static frontend catch-all route.

---

## 9. LLM Integration

Use LiteLLM via OpenRouter. The default `OPENROUTER_MODEL` is `deepseek/deepseek-v4-flash:free` — a free, MoE model (284B total / 13B activated) explicitly designed for fast inference. Structured JSON output is validated before use.

Local development reads `OPENROUTER_API_KEY` from `.env`. A populated `.env` is never committed; `.env.example` documents the required variables.

**Fallback**: If the primary model is unavailable or rate-limited, retry once with `OPENROUTER_FALLBACK_MODEL`, defaulting to `openai/gpt-oss-120b:free` (117B MoE, 131K context, also free on OpenRouter). Both model IDs remain configurable because free model availability can change.

### How It Works

When the user sends a chat message, the backend:

1. Loads the user's current portfolio context (cash, positions with P&L, watchlist with live prices, total portfolio value)
2. Loads recent conversation history from the `chat_messages` table
3. Constructs a prompt with a system message, portfolio context, conversation history, and the user's new message
4. Calls the LLM via LiteLLM → OpenRouter with structured JSON output and an explicit timeout:
   ```python
   import litellm, os, json
   response = litellm.completion(
       model=f"openrouter/{os.environ['OPENROUTER_MODEL']}",
       api_key=os.environ["OPENROUTER_API_KEY"],
       messages=messages,
       response_format={"type": "json_object"},
       timeout=float(os.getenv("OPENROUTER_TIMEOUT_SECONDS", "30")),
   )
   result = json.loads(response.choices[0].message.content)
   ```
5. Parses and validates the complete response with strict Pydantic models. Unknown fields, malformed JSON, invalid tickers, non-positive quantities, and excessive action counts are rejected. Malformed output receives at most one controlled retry.
6. Applies requested actions in order through the same trade and watchlist services used by manual API routes.
7. Stores the assistant message plus ordered action results in `chat_messages`.
8. Returns the assistant message and ordered results to the frontend (no token-by-token streaming — a loading indicator is sufficient).

### Structured Output Schema

The LLM is instructed to respond with JSON matching this schema:

```json
{
  "message": "Your conversational response to the user",
  "trades": [
    {"ticker": "AAPL", "side": "buy", "quantity": 10}
  ],
  "watchlist_changes": [
    {"ticker": "PYPL", "action": "add"}
  ]
}
```

- `message` (required): The conversational text shown to the user
- `trades` (optional): Array of trades to execute only when the user's message explicitly requests or confirms execution. Each trade goes through the same validation as manual trades.
- `watchlist_changes` (optional): Array of watchlist modifications
- The backend limits each response to 10 total actions.

The backend response adds an ordered `action_results` array. Each entry includes the requested action, `status` (`"executed"` or `"rejected"`), and an optional error object. Action batches use **ordered partial success**: one rejected action does not roll back earlier successful actions. Each individual trade remains atomic.

### Auto-Execution

Trades specified by the LLM execute automatically without a second confirmation dialog only when the user's latest message explicitly requests or confirms the trade. This is a deliberate design choice:
- It's a simulated environment with fake money, so the stakes are zero
- It creates an impressive, fluid demo experience
- It demonstrates agentic AI capabilities — the core theme of the course

The LLM may proactively analyze the portfolio and suggest trades, but suggestions do not mutate state. Watchlist changes may execute when explicitly requested by the user.

If an action fails validation, the backend appends a structured rejection to `action_results`. The frontend renders the rejection separately from the model-authored message, avoiding claims that execution succeeded before validation ran.

### System Prompt Guidance

The LLM should be prompted as "FinAlly, an AI trading assistant" with instructions to:
- Analyze portfolio composition, risk concentration, and P&L
- Suggest trades with reasoning
- Execute trades when the user asks or agrees
- Manage the watchlist when requested
- Be concise and data-driven in responses
- Always respond with valid structured JSON

### LLM Mock Mode

When `LLM_MOCK=true`, the backend returns deterministic mock responses instead of calling OpenRouter. This enables:
- Fast, free, reproducible E2E tests
- Development without an API key
- CI/CD pipelines

---

## 10. Frontend Design

### Layout

The frontend is a single-page application with a dense, terminal-inspired layout. The specific component architecture and layout system is up to the Frontend Engineer, but the UI should include these elements:

- **Watchlist panel** — grid/table of watched tickers with: ticker symbol, current price (flashing green/red on change), daily change %, and a sparkline mini-chart (accumulated from SSE since page load)
- **Main chart area** — larger chart for the currently selected ticker. On selection, fetches OHLCV bar history from `GET /api/prices/history/{ticker}` to pre-populate the chart, then appends live prices from the SSE stream. Clicking a ticker in the watchlist selects it here.
- **Portfolio heatmap** — treemap visualization where each rectangle is a position, sized by portfolio weight, colored by P&L (green = profit, red = loss)
- **P&L chart** — line chart showing total portfolio value over time, using data from `portfolio_snapshots`
- **Positions table** — tabular view of all positions: ticker, quantity, avg cost, current price, unrealized P&L, % change
- **Trade bar** — simple input area: ticker field, quantity field, buy button, sell button. Market orders, instant fill.
- **AI chat panel** — docked/collapsible sidebar. Message input, scrolling conversation history, loading indicator while waiting for LLM response. Trade executions and watchlist changes shown inline as confirmations.
- **Header** — portfolio total value (updating live), connection status indicator, cash balance

The default selected ticker is the first watchlist entry (`AAPL` for a fresh visitor). The UI includes empty portfolio, empty watchlist, loading, stale-price, provider-capacity, and reconnecting states.

### Technical Notes

- Use `EventSource` for SSE connection to `/api/stream/prices`
- Use TradingView Lightweight Charts for the main chart and sparklines after confirming static export compatibility
- Price flash effect: on receiving a new price, briefly apply a CSS class with background color transition, then remove it
- Recalculate displayed portfolio totals from cached positions and incoming SSE prices; do not refetch the full portfolio on every tick
- All API calls go to the same origin (`/api/*`) — no CORS configuration needed
- Tailwind CSS for styling with a custom dark theme

---

## 11. Docker & Deployment

### Multi-Stage Dockerfile

```
Stage 1: Node 20 slim
  - Copy frontend/
  - npm ci && npm run build (produces static export)

Stage 2: Python 3.12 slim
  - Install uv
  - Copy backend/
  - uv sync --frozen (install Python dependencies from lockfile)
  - Copy --from=stage1 /app/frontend/out /app/static
  - Expose port 8000
  - HEALTHCHECK: GET http://localhost:8000/api/health
  - CMD: uvicorn app.main:app --host 0.0.0.0 --port 8000
```

The frontend `out/` directory (Next.js static export) is copied to `/app/static` in the container. FastAPI registers `/api/*` routes first, then mounts `StaticFiles(directory="/app/static", html=True)` as the `/` catch-all on port 8000.

### Docker Volume

The SQLite database persists via a named Docker volume:

```bash
docker run -v finally-data:/app/db -p 8000:8000 --env-file .env finally
```

`finally-data:/app/db` is a named Docker volume, not a bind mount from the repository's `db/` directory. The backend writes `/app/db/finally.db`, configured through `DATABASE_PATH`.

### Start/Stop Scripts

**`scripts/start_mac.sh`** (macOS/Linux):
- Builds the Docker image if not already built (or if `--build` flag passed)
- Runs the container with the volume mount, port mapping, and `.env` file
- Prints the URL to access the app
- Optionally opens the browser

**`scripts/stop_mac.sh`** (macOS/Linux):
- Stops and removes the running container
- Does NOT remove the volume (data persists)

**`scripts/start_windows.ps1`** / **`scripts/stop_windows.ps1`**: PowerShell equivalents for Windows.

All scripts should be idempotent — safe to run multiple times.

### Optional Cloud Deployment

The container is designed to deploy to AWS App Runner, Render, or any container platform. A Terraform configuration for App Runner may be provided in a `deploy/` directory as a stretch goal, but is not part of the core build.

---

## 12. Testing Strategy

### Unit Tests (within `frontend/` and `backend/`)

**Backend (pytest)**:
- Market data: simulator generates valid prices and OHLCV history, GBM math is correct, Alpaca WebSocket trade parsing works, stale prices reject trades, reconnect restores subscriptions, and both implementations conform to the abstract `MarketDataProvider` interface
- Watchlist subscriptions: shared symbols subscribe upstream once, `0 → 1` and `1 → 0` transitions work, restart rebuilds reference counts, and the 31st globally unique symbol returns a clear error under the default cap
- Portfolio: atomic trade execution, fixed-precision P&L calculations, repeated fractional buys and sells, concurrent buys cannot overspend, concurrent sells cannot oversell, selling more than owned, buying with insufficient cash, and selling at a loss
- LLM: strict structured output parsing, graceful handling of malformed responses, controlled retry, unknown-field rejection, explicit-intent requirement, action-count limit, ordered partial-success results, and trade validation within chat flow
- API routes: correct status codes, response shapes, error handling

**Frontend (React Testing Library or similar)**:
- Component rendering with mock data
- Price flash animation triggers correctly on price changes
- Watchlist CRUD operations
- Portfolio display calculations
- Chat message rendering and loading state

### E2E Tests (in `test/`)

**Infrastructure**: A separate `docker-compose.test.yml` in `test/` that spins up the app container plus a Playwright container. This keeps browser dependencies out of the production image.

**Environment**: Tests run with `LLM_MOCK=true` by default for speed and determinism.

**Key Scenarios**:
- Fresh start: default watchlist appears, $10k balance shown, prices are streaming
- Add and remove a ticker from the watchlist
- Buy shares: cash decreases, position appears, portfolio updates
- Sell shares: cash increases, position updates or disappears
- Portfolio visualization: heatmap renders with correct colors, P&L chart has data points
- AI chat (mocked): send a message, receive a response, trade execution appears inline
- AI chat rejection (mocked): invalid action does not mutate state and renders a clear inline rejection
- Trade history: execute trades, verify they appear in `GET /api/portfolio/trades`
- Restart persistence: restart the container and verify portfolio state plus restored watchlist prices
- Static routing: frontend fallback serves the SPA without shadowing `/api/*`
- SSE resilience: disconnect and verify reconnection *(stretch goal — complex to implement reliably in Playwright)*

---

## 13. Implementation Sequence

Build a working vertical slice before the full visual workstation:

1. Simulator, FastAPI lifespan lifecycle, SQLite schema, and persistent visitor cookie
2. Watchlist CRUD, global subscription reference counts, one SSE fanout stream, atomic manual trades, and portfolio endpoint
3. Static frontend with watchlist, positions, trade bar, connection status, and empty/error states
4. Simulator OHLCV aggregation, bounded history buffers, main chart, and portfolio chart
5. Mocked LLM action flow with strict validation and ordered action results
6. Alpaca provider with reconnect and subscription restoration
7. Heatmap, correlated simulator moves, random events, visual polish, and broader E2E coverage

This order keeps real LLM access, external market data, and polish out of the critical path until the highest-risk contracts are proven locally.

---

## 14. Review Notes — Decision Log

| # | Issue | Decision |
|---|-------|----------|
| 1 | LLM model ID was wrong (`openrouter/openai/gpt-oss-120b`) | ✅ Fixed — using configurable `deepseek/deepseek-v4-flash:free` with fallback `openai/gpt-oss-120b:free`; LiteLLM adds the `openrouter/` provider prefix |
| 2 | LiteLLM call pattern was undefined (referenced a CLI skill, not Python code) | ✅ Fixed — concrete `litellm.completion()` call with model, api_key, and `response_format` added to §9 |
| 3 | SSE + dynamic watchlist changes underspecified | ✅ Fixed — `subscribe()`/`unsubscribe()` on `MarketDataProvider` called on watchlist changes; SSE scoped per user via cookie |
| 4 | No historical price endpoint for main chart | ✅ Fixed — `GET /api/prices/history/{ticker}` added to §8; main chart now fetches bars on ticker select then appends SSE |
| 5 | No `GET /api/portfolio/trades` endpoint | ✅ Fixed — endpoint added to §8 |
| 6 | `portfolio_snapshots` had no retention policy | ✅ Fixed — hourly cleanup task, retain last 24 hours |
| 7 | Dockerfile static path vague | ✅ Fixed — `COPY --from=stage1 /app/frontend/out /app/static`; FastAPI serves that directory from the `/` catch-all after API routes |
| 8 | Missing `OPENROUTER_API_KEY` caused opaque runtime error | ✅ Fixed — backend validates on startup, returns 503 with clear message if missing |
| 9 | Multi-user: schema had `user_id` defaulting to `"default"` (single-user) | ✅ Fixed — UUID assigned per browser profile via a persistent cookie; all API routes read `user_id` from the cookie |
| 10 | `users_profile` table naming inconsistency | ✅ Fixed — renamed to `user_profile` |
| 11 | `backend/db/` and top-level `db/` caused naming confusion | ✅ Fixed — `backend/db/` renamed to `backend/schema/` |
| 12 | `actions` column in `chat_messages` looked redundant | ✅ Kept intentionally — full audit trail; documented explicitly in schema |
| 13 | Abstract `MarketDataProvider` interface was referenced but not defined | ✅ Fixed — interface definition added to §6 |
| 14 | SSE resilience E2E test disproportionately complex | ✅ Marked as stretch goal in §12 |
| 15 | Alpaca quote stream did not define an executable fill price | ✅ Fixed — core build uses last trade price, records source timestamp, and rejects stale prices |
| 16 | Trades could update cash, positions, history, and snapshots inconsistently | ✅ Fixed — each trade is one SQLite transaction with WAL mode and busy timeout |
| 17 | Monetary values used floating-point `REAL` columns | ✅ Fixed — money uses integer microdollars and quantities use integer microshares |
| 18 | Alpaca's 30-symbol limit was treated as a per-watchlist limit | ✅ Fixed — limit applies to the global symbol union with persisted reference-count recovery |
| 19 | Lazy initialization conflicted with provider and background-task startup | ✅ Fixed — schema, provider, subscription recovery, jobs, and shutdown use FastAPI lifespan hooks |
| 20 | SSE allowed client-supplied `user_id` despite cookie scoping | ✅ Fixed — stream identity comes only from the server-issued visitor cookie |
| 21 | LLM JSON was parsed but not strictly validated | ✅ Fixed — strict Pydantic validation, action limit, timeout, retry, and configurable fallback added |
| 22 | LLM message could claim action success before backend validation | ✅ Fixed — backend returns ordered partial-success `action_results`, rendered separately |
| 23 | Anonymous identity lifetime was unclear | ✅ Fixed — persistent one-year `HttpOnly`, `SameSite=Lax` visitor cookie |
| 24 | Docker named volume was described as a repository bind mount | ✅ Fixed — named-volume behavior and `DATABASE_PATH` documented |

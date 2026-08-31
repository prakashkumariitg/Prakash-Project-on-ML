<p align="center">
  <h1 align="center">APIx - Airfare Price Index</h1>
  <p align="center">
    <em>A production-grade flight fare intelligence platform that observes how prices evolve across advance-purchase windows.</em>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/python-3.10+-blue" alt="Python 3.10+">
    <img src="https://img.shields.io/badge/scrapy-2.18+-green" alt="Scrapy 2.18+">
    <img src="https://img.shields.io/badge/fastapi-0.115+-red" alt="FastAPI">
    <img src="https://img.shields.io/badge/postgres-16-blue" alt="PostgreSQL 16">
    <img src="https://img.shields.io/badge/license-MIT-orange" alt="MIT License">
  </p>
  <p align="center">
    <strong>Smart India Hackathon 2026 | PS ID: 26056</strong>
  </p>
  <p align="center">
    <a href="https://www.sih.gov.in/sih2026PS">View Problem Statement</a>
  </p>
</p>

---

## 🎯 The Problem

Flight prices are opaque and dynamic. Airlines use complex revenue management strategies to adjust fares based on inventory, demand patterns, and booking advance time. Travelers and researchers lack insight into:

- **"Is this price good?"** — How does today's fare compare to historical trends?
- **"When should I book?"** — What's the optimal advance-purchase window for each route?
- **"Which airlines price competitively?"** — Which carriers dominate which routes?
- **"How predictable are fare dynamics?"** — Can we forecast price movements?

Existing solutions (Skyscanner, Hopper, Google Flights) offer price predictions but hide their methodology behind proprietary black boxes. They provide **estimates without transparency or auditability**.

---

## ✨ What APIx Does Differently

| Feature | Skyscanner / Hopper | APIx |
|---------|-------------------|------|
| **Data Model** | Black-box ML predictions | Append-only time-series facts (fully auditable) |
| **Scope** | On-demand price estimates | Systematic daily observations (6 routes × 5 windows = 30 daily data points) |
| **Transparency** | Results only, no raw data | Raw data + schema + metadata + source attribution |
| **Compliance** | Implicit / assumed | Explicit: robots.txt checked live; legal ToS registry |
| **Extensibility** | Closed proprietary platform | Open architecture; pluggable spider interface; multi-airline ready |
| **Architecture** | Monolithic black box | Clean separation: scraper → transformer → normalized DB → API |
| **Reproducibility** | Same query, different results | Same logic + same time = identical results (deterministic) |

**Result:** You get **real, attributed, reproducible, time-series fare data** that powers dashboards, research papers, price elasticity models, and market transparency initiatives.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  AIRLINE APIS (Akasa Air)                                   │
└─────────────────────┬───────────────────────────────────────┘
                      │
         ┌────────────▼────────────────────┐
         │  Compliance Gateway             │
         │  ✅ robots.txt check (live)     │
         │  ✅ ToS registry                │
         │  ✅ Rate limiting + jitter      │
         │  ✅ Circuit breaker             │
         └────────────┬────────────────────┘
                      │
    ┌─────────────────▼──────────────────────┐
    │  AkasaAirSpider (30 concurrent tasks)  │
    │  ✅ 6 city pairs                       │
    │  ✅ 5 advance-purchase windows         │
    │  ✅ Playwright + direct API call       │
    │  ✅ Graceful error handling            │
    └─────────────────┬──────────────────────┘
                      │
        ┌─────────────▼───────────────────┐
        │  Data Transformation             │
        │  ✅ raw_fare_item (14 fields)    │
        │  ✅ Canonical shape              │
        │  ✅ Source-agnostic format       │
        └─────────────┬───────────────────┘
                      │
           ┌──────────▼──────────────┐
           │  JSON/CSV Export        │
           │  (apix_daily_scrape.*)  │
           └──────────┬──────────────┘
                      │
    ┌─────────────────▼────────────────────────┐
    │  PostgreSQL Loader                       │
    │  ✅ Idempotent upsert                    │
    │  ✅ Carrier/Route dimensions             │
    │  ✅ FareObservation facts (append-only)  │
    └─────────────────┬────────────────────────┘
                      │
        ┌─────────────▼──────────────────┐
        │  PostgreSQL (Normalized)       │
        │  ✅ Carriers (dimension)       │
        │  ✅ Routes (dimension)         │
        │  ✅ FareObservations (facts)   │
        │  ✅ Optimized indexes          │
        └─────────────┬──────────────────┘
                      │
        ┌─────────────▼──────────────────┐
        │  FastAPI (Read-only)           │
        │  ✅ /fares (paginated)         │
        │  ✅ /routes                    │
        │  ✅ /carriers                  │
        │  ✅ /daily-summary (aggregated)│
        └─────────────┬──────────────────┘
                      │
        ┌─────────────▼──────────────┐
        │  Dashboard/Analytics       │
        │  ✅ Trend visualization    │
        │  ✅ Price elasticity       │
        │  ✅ Historical comparison  │
        └──────────────────────────────┘
```

---

## 📁 Project Structure

```
BananaShake/
├── apixproj/                           # Scrapy project
│   ├── spiders/
│   │   └── akasa_air_spider.py        # Akasa Air web scraper (direct API)
│   ├── raw_fare_item.py               # Canonical data model (14 fields)
│   ├── settings.py                    # Scrapy + Playwright configuration
│   └── __init__.py
│
├── app/                                # FastAPI + Database layer
│   ├── api/
│   │   ├── main.py                    # FastAPI app setup + CORS
│   │   ├── schemas.py                 # Pydantic response models
│   │   ├── deps.py                    # Dependency injection (DB session)
│   │   └── routers/
│   │       └── fares.py               # REST endpoints
│   ├── db/
│   │   ├── models.py                  # SQLAlchemy ORM models
│   │   ├── session.py                 # Database connection pool
│   │   ├── loader.py                  # Insert/upsert logic
│   │   └── migrations/                # Alembic version control
│   │       └── versions/              # Migration history
│   └── ingestion/
│       └── scheduler.py               # Route × AP-window task matrix
│
├── tests/                              # Unit tests (pytest-compatible)
│   ├── test_akasa_air_spider.py
│   ├── test_scheduler.py
│   ├── test_raw_fare_item*.py
│   ├── test_middlewares.py
│   └── test_run_daily_scrape.py
│
├── run_daily_scrape.py                # Main entry point (scrape + load)
├── compliance.py                       # robots.txt checker + ToS registry
├── middlewares.py                      # Anti-bot middleware + circuit breaker
├── docker-compose.yml                 # PostgreSQL container definition
├── alembic.ini                        # Database migration config
├── requirements.txt                   # Python dependencies
├── scrapy.cfg                         # Scrapy project config
└── README.md                          # This file
```

---

## ⚙️ How It Works

### **1. Scraping Basket**

**Coverage:** 6 Indian domestic city pairs × 5 advance-purchase windows = **30 daily observations**

**Routes:**
- DEL ↔ BOM (Delhi ↔ Mumbai)
- DEL ↔ BLR (Delhi ↔ Bangalore)
- BOM ↔ BLR (Mumbai ↔ Bangalore)
- DEL ↔ CCU (Delhi ↔ Kolkata)
- BLR ↔ HYD (Bangalore ↔ Hyderabad)
- MAA ↔ DEL (Chennai ↔ Delhi)

**Advance-Purchase Windows:**
- T+1, T+7, T+15, T+30, T+45 days (IST calendar)

Why this basket? Captures the full range of traveler behavior: last-minute bookings (T+1), common advance-purchase patterns (T+7, T+15), and long-lead-time business travel (T+30, T+45).

### **2. Data Collection (Compliance-First)**

Every request is gated by a multi-layered compliance framework:

✅ **robots.txt Checking** — Live fetch with TTL caching; fail-closed if unreachable  
✅ **ToS Registry** — Explicit legal clearance per source (default: pending = safe)  
✅ **Circuit Breaker** — Stops requesting from domains that block us  
✅ **UA Rotation** — Realistic Chrome/Safari/Firefox user agents  
✅ **Rate Limiting** — DOWNLOAD_DELAY=2.5s + random jitter (polite scraping)  
✅ **Graceful Degradation** — Single task failure doesn't crash the batch  

### **3. Data Transformation**

Raw API responses → **raw_fare_item** (canonical 14-field shape)

```python
{
  "route_id": "DEL-BOM",
  "carrier_code": "QP",
  "flight_number": "QP1401",
  "departure_time": "2026-09-08T06:00:00+05:30",
  "arrival_time": "2026-09-08T08:15:00+05:30",
  "base_fare": 5500.00,
  "taxes_and_fees": 350.00,
  "total_fare": 5850.00,
  "seats_left": 4,
  "lead_window": "T+7",
  "source_name": "akasa_air",
  "source_type": "airline_direct",
  "scraped_at": "2026-09-01T14:30:00Z"
}
```

**Why 14 fields?** Future-proof schema designed for multi-airline expansion. Same shape regardless of source = no per-spider schema drift.

### **4. Database Storage (Append-Only)**

**3 normalized tables:**

```sql
-- Dimension: Airline
CREATE TABLE carriers (
  carrier_code VARCHAR PRIMARY KEY,
  airline_name VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT now()
);

-- Dimension: City Pair
CREATE TABLE routes (
  route_id VARCHAR PRIMARY KEY,
  origin VARCHAR NOT NULL,
  destination VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT now()
);

-- Fact: Time-series observations (append-only)
CREATE TABLE fare_observations (
  id INT PRIMARY KEY AUTOINCREMENT,
  route_id VARCHAR FK,
  carrier_code VARCHAR FK,
  flight_number VARCHAR,
  departure_time TIMESTAMP,
  arrival_time TIMESTAMP,
  base_fare DECIMAL(10,2),
  taxes_and_fees DECIMAL(10,2),
  total_fare DECIMAL(10,2),
  seats_left INT,
  lead_window VARCHAR,
  source_name VARCHAR,
  scraped_at TIMESTAMP NOT NULL,  -- When quote was fetched
  loaded_at TIMESTAMP DEFAULT now(),
  
  UNIQUE (route_id, carrier_code, flight_number, 
          departure_time, lead_window, scraped_at),
  INDEX (route_id, lead_window, scraped_at)
);
```

**Design Principle:** FareObservation is **append-only** because it's a time-series. Same flight at different scrape times = distinct rows (essential for trend analysis).

**Loading:** Idempotent upsert via `ON CONFLICT DO NOTHING` (safe for retries).

### **5. API Layer (Read-Only)**

**Endpoints:**

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Server liveness check |
| `GET /routes` | All tracked city pairs |
| `GET /carriers` | All airlines |
| `GET /fares?route_id=DEL-BOM&lead_window=T+7&limit=100` | Paginated fare observations |
| `GET /fares/daily-summary?route_id=DEL-BOM` | Daily aggregated stats (min/avg/max) |

**Query Parameters:**
- `route_id` — Filter by city pair (e.g., "DEL-BOM")
- `carrier_code` — Filter by airline (e.g., "QP")
- `lead_window` — Filter by advance-purchase window (e.g., "T+7")
- `scraped_from` / `scraped_to` — Date range filter
- `limit` (default 100, max 1000) — Pagination
- `offset` (default 0) — Pagination offset

---

## 🚀 Quick Start

### **1. Prerequisites**

- Python 3.10+
- Docker & Docker Compose (for PostgreSQL)
- Git

### **2. Clone & Setup**

```bash
git clone https://github.com/adityagarg006/BananaShake
cd BananaShake

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install Playwright browsers
playwright install chromium
```

### **3. Start PostgreSQL**

```bash
docker-compose up -d
```

Verify connection:
```bash
psql -U apix -d apix -c "SELECT version();"
```

### **4. Apply Database Migrations**

```bash
alembic upgrade head
```

### **5. Run Daily Scrape**

```bash
python run_daily_scrape.py
```

**Output:**
- `apix_daily_scrape.json` — Full item details
- `apix_daily_scrape.csv` — Same data (CSV format)
- PostgreSQL database updated

**Coverage matrix printed to stdout:**
```
Route         T+1    T+7    T+15   T+30   T+45
─────────────────────────────────────────────
DEL→BOM        5      5      5      5      5
DEL→BLR        5      5      5      5      5
BOM→BLR        5      5      5      5      5
DEL→CCU        5      5      5      5      5
BLR→HYD        5      5      5      5      5
MAA→DEL        5      5      5      5      5
─────────────────────────────────────────────
TOTAL        150 fares collected
```

### **6. Start API Server**

```bash
uvicorn app.api.main:app --reload
```

**Access:**
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
- Raw API: `http://localhost:8000`

### **7. Query the API**

```bash
# Get all routes
curl http://localhost:8000/routes

# Get fares for DEL-BOM on T+7 window
curl "http://localhost:8000/fares?route_id=DEL-BOM&lead_window=T%2B7&limit=10"

# Get daily summary
curl "http://localhost:8000/fares/daily-summary"

# Health check
curl http://localhost:8000/health
```

---

## 🧪 Testing

```bash
# Run all tests
python -m pytest tests/ -v

# Test spider parsing (no network calls)
python -m unittest tests.test_akasa_air_spider -v

# Test scheduler
python -m unittest tests.test_scheduler -v

# Test end-to-end scrape (real run)
python run_daily_scrape.py --skip-db
```

---

## 🛡️ Compliance & Safety

**All scrapers must earn trust.** APIx implements defense-in-depth compliance:

| Layer | Mechanism | Failure Mode |
|-------|-----------|--------------|
| **Legal** | ToS registry (explicit clearance) | Default: pending (safe, conservative) |
| **Technical** | robots.txt checking + live fetch | Fail-closed (stops if unreachable) |
| **Behavioral** | Circuit breaker (respect "no") | Stop hammering blocked domains |
| **Identities** | UA rotation (realistic browsers) | Fall back to static pool if live feed fails |
| **Pace** | DOWNLOAD_DELAY + jitter | ~1 request per 2.5s per domain (polite) |
| **Transparency** | Source attribution on every row | Audit trail: `source_name`, `source_type`, `scraped_at` |

**Principle:** Never scrape faster than you read. Never pretend to be someone else (always use real user agents). Always check robots.txt. Always respect circuit-breaker signals.

---

## 📊 Database Schema

### Carriers (Dimension)
```sql
CREATE TABLE carriers (
  carrier_code VARCHAR PRIMARY KEY,     -- "QP", "6E", "SG", "I5", etc.
  airline_name VARCHAR NOT NULL,        -- "Akasa Air", "IndiGo", etc.
  created_at TIMESTAMP DEFAULT now()
);
```

### Routes (Dimension)
```sql
CREATE TABLE routes (
  route_id VARCHAR PRIMARY KEY,         -- "DEL-BOM"
  origin VARCHAR NOT NULL,              -- "DEL"
  destination VARCHAR NOT NULL,         -- "BOM"
  created_at TIMESTAMP DEFAULT now()
);
```

### FareObservation (Fact Table - Append-Only)
```sql
CREATE TABLE fare_observations (
  id INT PRIMARY KEY AUTOINCREMENT,
  route_id VARCHAR NOT NULL FK routes(route_id),
  carrier_code VARCHAR NOT NULL FK carriers(carrier_code),
  flight_number VARCHAR,
  departure_time TIMESTAMP,
  arrival_time TIMESTAMP,
  base_fare DECIMAL(10,2),
  taxes_and_fees DECIMAL(10,2),
  total_fare DECIMAL(10,2),
  seats_left INT,
  lead_window VARCHAR,                  -- "T+1", "T+7", "T+15", "T+30", "T+45"
  source_name VARCHAR,                  -- "akasa_air"
  source_type VARCHAR,                  -- "airline_direct"
  scraped_at TIMESTAMP NOT NULL,        -- When this quote was fetched (UTC)
  loaded_at TIMESTAMP DEFAULT now(),
  
  CONSTRAINT unique_observation UNIQUE (
    route_id, carrier_code, flight_number, departure_time, 
    lead_window, scraped_at
  ),
  INDEX idx_route_window_date (route_id, lead_window, scraped_at),
  INDEX idx_route_date (route_id, scraped_at),
  INDEX idx_window (lead_window)
);
```

**Why append-only?** Historical price snapshots are the signal. Updating a row loses time-series information that powers trend analysis and price elasticity models.

---

## 🔧 Configuration

Tunable values in `apixproj/settings.py`:

| Setting | Default | Purpose |
|---------|---------|---------|
| `DOWNLOAD_DELAY` | `2.5` | Seconds between requests per domain |
| `AUTOTHROTTLE` | `True` | Adaptive throttling based on response time |
| `CONCURRENT_REQUESTS_PER_DOMAIN` | `1` | Polite single-threaded requests |
| `PLAYWRIGHT_HEADLESS` | `True` | Headless Chromium (faster, stealthier) |
| `MAX_RETRIES` | `2` | Retry failed requests |
| `ROBOTS_TXT_TTL` | `3600` | Cache robots.txt for 1 hour |

---

## 🏛️ Architecture Decisions

| Decision | Rationale |
|----------|-----------|
| **Append-only fact table** | Time-series charting requires historical snapshots; updating breaks trend analysis |
| **Separate spider/loader layers** | Spider is fast & source-agnostic; loader handles schema mapping & transaction safety |
| **Canonical raw_fare_item** | Multi-airline future-ready; no per-spider schema drift |
| **robots.txt + ToS registry** | Ethical scraping respects both technical and legal boundaries |
| **Direct API (no UI automation)** | More reliable than Playwright click/type; less flaky |
| **Circuit breaker pattern** | Respects "no" from servers; stops hammering blocked domains |
| **Read-only FastAPI** | Data writes in scraper layer; decouples concerns |
| **Natural primary keys** | carrier_code, route_id are stable externals; no surrogate IDs needed |
| **IST-based windows** | Anchors to India's market calendar (traveler-facing time, not UTC) |

---

## 📈 Roadmap

- ✅ Akasa Air scraper (MVP)
- 🚧 Multi-airline support (IndiGo, SpiceJet, Air India, Vistara)
- 🚧 Real-time dashboard + trend visualization
- 🚧 Price elasticity model (demand forecasting)
- 🚧 Alert system (price drops, anomalies)
- 🚧 API rate limits + authentication
- 🚧 Historical fare analysis reports
- 🚧 Self-hosted backend deployment guide

---

## 📚 Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Web Scraping** | Scrapy + Playwright | 2.18+ / 1.62+ |
| **Browser Automation** | Playwright (Chromium) | 1.62+ |
| **robots.txt Parser** | Protego | 0.6+ |
| **Database** | PostgreSQL | 16 |
| **ORM** | SQLAlchemy | 2.0+ |
| **Migrations** | Alembic | 1.13+ |
| **API Framework** | FastAPI | 0.115+ |
| **Server** | Uvicorn | 0.30+ |
| **User-Agent Rotation** | fake-useragent | 2.2+ |

---

## 🔐 Data Privacy & Compliance

- ✅ **No PII** — Only fare quotes, no passenger data
- ✅ **Transparent sourcing** — Every row tagged with source + scrape timestamp
- ✅ **Audit trail** — `source_name`, `source_type`, `scraped_at` on every observation
- ✅ **robots.txt respected** — Live checks with caching
- ✅ **ToS compliance** — Explicit legal clearance registry
- ✅ **No profiling** — No user tracking or behavioral analysis
- ✅ **Reproducible** — Same logic + same time = same results (deterministic)

---

## 📄 License

MIT License. See [LICENSE](LICENSE) for details.

---

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Write tests for your changes
4. Commit with clear messages
5. Push to your fork
6. Submit a pull request

Please ensure all tests pass and code follows the existing style.

---

<p align="center">
  <strong>APIx</strong> — Airfare Price Index Data Platform
  <br/>
  Built for transparent, ethical, reproducible flight-fare intelligence
</p>

<p align="center">
  <a href="https://www.sih.gov.in/sih2026PS">Smart India Hackathon 2026</a>
</p>

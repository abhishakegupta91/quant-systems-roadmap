# Elite Fintech Architect Study Plan
## Month 4: Portfolio Transformation & Capstone

**Prepared for:** Abhishake Gupta — 12+ years Python/Backend, BTech ECE, IB API experience  
**Target Role:** Hybrid Staff Engineer (Trading Platform + Quant Development)  
**Target Market:** Indian Markets (NSE/BSE) with global applicability  
**Duration:** Weeks 13-16 | 2-3 hours/day (~15-20 hrs/week)  
**Last Updated:** January 2026

---

## Month 4 Theme: Production Platform, Low-Latency Deep Dive, Career Positioning

**Focus:** Integrate all components into a production-ready trading platform. Begin C++ exposure for future HFT work. Polish portfolio for job applications.

**Time Commitment:** ~15-20 hours/week (2-3 hrs/day)

**Prerequisites:** Complete Month 3 deliverables:
- [ ] SEBI-compliant compliance module
- [ ] Real-time risk engine with VaR
- [ ] Event-driven backtesting framework
- [ ] Signal generation & screening platform
- [ ] Kill switch implementation

---

## Week 13: Platform Integration & Order Management System

### Learning Objectives
- Integrate all Month 1-3 components into unified platform
- Build production-grade Order Management System (OMS)
- Implement proper state management and recovery

### Study Materials

**Theory (4-5 hours)**
- **Book:** "Trading and Exchanges" — OMS chapters
- **Engineering Blog:** Robinhood's order management architecture
- **Paper:** "Design of a High-Frequency Trading System" (academic)

### Hands-On (10-12 hours)

**Project: Unified Trading Platform - OMS Core**
```
Goal: Production-ready Order Management System
Structure:
trading_platform/
├── oms/
│   ├── order_manager.py      # Central order lifecycle management
│   ├── order_state.py        # State machine (NEW→PENDING→FILLED/REJECTED)
│   ├── order_book_local.py   # Local order tracking
│   └── reconciliation.py     # Broker position reconciliation
├── execution/
│   ├── executor.py           # Order execution orchestrator
│   ├── smart_router.py       # Best execution routing
│   └── fill_handler.py       # Process fills, update positions
├── integration/
│   ├── platform.py           # Main platform orchestrator
│   ├── startup.py            # Initialization sequence
│   └── shutdown.py           # Graceful shutdown
└── config/
    ├── platform_config.yaml  # Platform configuration
    └── strategies_config.yaml # Strategy parameters
```

**Integration Points:**
1. Market Data Service → Signal Engine → OMS → Broker
2. Risk Engine validates every order before submission
3. Event Store captures all state changes
4. Compliance module tags and logs all orders

**Deliverables:**
1. Integrated trading platform with all components
2. OMS with proper state management
3. Position reconciliation with broker
4. Startup/shutdown procedures
5. Platform configuration system

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | OMS architecture study | 2.5 |
| Tue | Design integration architecture | 2.5 |
| Wed | Code: Order manager + state machine | 3 |
| Thu | Code: Executor + fill handler | 3 |
| Fri | Code: Platform orchestrator | 3 |
| Sat | Integration testing | 3 |
| Sun | Documentation + fixes | 2 |

---

## Week 14: Low-Latency Deep Dive & C++ Introduction

### Learning Objectives
- Understand kernel bypass, DPDK, and co-location concepts
- Begin C++ for trading systems (conceptual + basic implementation)
- Learn FPGA awareness for future HFT work

### Study Materials

**Theory (6-7 hours)**
- **Book:** "C++ High Performance" — Chapters 1-4
- **Paper:** "The Design of a Low-Latency Trading System"
- **Blog:** Jane Street tech blog on low-latency
- **Video:** "FPGA in Trading" presentations

**Video Resources**
- **YouTube - Hussein Nasser:** "Low Latency Systems Design"
- **CppCon Talks:** "Low Latency C++" presentations

### Hands-On (6-8 hours)

**Project: Low-Latency Concepts Implementation**
```
Goal: Understand and implement low-latency patterns
Structure:
├── cpp_basics/
│   ├── order_book.cpp        # Simple order book in C++
│   ├── matching_engine.cpp   # Basic matching in C++
│   └── CMakeLists.txt        # Build configuration
├── python_bindings/
│   └── pybind_orderbook.cpp  # Python bindings for C++ code
├── latency_analysis/
│   ├── network_latency.py    # Measure network latency
│   ├── kernel_bypass.md      # Document kernel bypass concepts
│   └── colocation.md         # Co-location considerations
└── benchmarks/
    └── cpp_vs_python.py      # Compare C++ vs Python performance
```

**Concepts to Document:**
1. Kernel bypass (DPDK, Solarflare OpenOnload)
2. Lock-free data structures
3. Memory-mapped I/O
4. CPU pinning and NUMA awareness
5. Co-location and direct market access

**Deliverables:**
1. Simple C++ order book implementation
2. Python bindings using pybind11
3. Performance comparison: C++ vs Python
4. Document: "Low-Latency Trading Concepts"
5. Document: "HFT Infrastructure Overview"

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | C++ High Performance chapters | 2.5 |
| Tue | Low-latency papers and blogs | 2.5 |
| Wed | C++ basics, order book implementation | 3 |
| Thu | Python bindings with pybind11 | 3 |
| Fri | Benchmarking + analysis | 2.5 |
| Sat | Documentation | 2.5 |
| Sun | Buffer / catch-up | 2 |

---

## Week 15: Web Dashboard & API Layer

### Learning Objectives
- Build monitoring dashboard for trading platform
- Create REST/WebSocket API for external access
- Implement authentication and rate limiting

### Study Materials

**Theory (3-4 hours)**
- **FastAPI Documentation:** Advanced patterns
- **React/Next.js:** Dashboard basics (or use Streamlit for speed)
- **WebSocket:** Real-time dashboard updates

### Hands-On (10-12 hours)

**Project: Trading Platform Dashboard**
```
Goal: Real-time monitoring and control interface
Structure:
├── api/
│   ├── main.py               # FastAPI application
│   ├── routes/
│   │   ├── orders.py         # Order management endpoints
│   │   ├── positions.py      # Position queries
│   │   ├── signals.py        # Signal/screener endpoints
│   │   └── risk.py           # Risk metrics endpoints
│   ├── websocket/
│   │   └── realtime.py       # Real-time updates
│   └── auth/
│       └── jwt_auth.py       # JWT authentication
├── dashboard/
│   ├── app.py                # Streamlit dashboard (or React)
│   ├── pages/
│   │   ├── overview.py       # Portfolio overview
│   │   ├── positions.py      # Current positions
│   │   ├── orders.py         # Order history
│   │   ├── signals.py        # Signal dashboard
│   │   └── risk.py           # Risk metrics
│   └── components/
│       └── charts.py         # Reusable chart components
└── docker/
    └── docker-compose.yaml   # Full stack deployment
```

**Dashboard Features:**
1. Real-time P&L and position display
2. Order submission and management
3. Signal/screener results
4. Risk metrics visualization
5. Kill switch button

**Deliverables:**
1. REST API with authentication
2. WebSocket for real-time updates
3. Monitoring dashboard
4. Docker Compose for full deployment
5. API documentation (OpenAPI/Swagger)

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | FastAPI advanced patterns | 2.5 |
| Tue | Code: API routes + auth | 3 |
| Wed | Code: WebSocket real-time | 3 |
| Thu | Code: Dashboard pages | 3 |
| Fri | Code: Charts + risk display | 3 |
| Sat | Docker Compose + deployment | 2.5 |
| Sun | Testing + documentation | 2 |

---

## Week 16: Portfolio Polish & Career Preparation

### Learning Objectives
- Prepare GitHub repository for job applications
- Create system design documentation
- Practice interview scenarios

### Study Materials

**Career Prep (4-5 hours)**
- **System Design Interview:** Practice trading system design
- **Resume:** Update with new skills and projects
- **LinkedIn:** Update profile with Fintech focus

### Hands-On (8-10 hours)

**Project: Portfolio Finalization**
```
Goal: Job-ready GitHub portfolio and documentation
Structure:
trading_platform/
├── README.md                 # Comprehensive project overview
├── docs/
│   ├── architecture.md       # System architecture document
│   ├── api_reference.md      # API documentation
│   ├── deployment.md         # Deployment guide
│   ├── design_decisions.md   # Key design decisions explained
│   └── diagrams/
│       ├── system_arch.png   # Architecture diagram
│       ├── data_flow.png     # Data flow diagram
│       └── sequence_diagrams/ # Key sequences
├── examples/
│   ├── strategy_example.py   # Example strategy implementation
│   └── backtest_example.py   # Example backtest run
└── CONTRIBUTING.md           # Contribution guidelines
```

**Portfolio Checklist:**
1. Clean, well-documented code
2. Comprehensive README with screenshots
3. Architecture diagrams (draw.io or similar)
4. Working demo (video or live)
5. Clear setup instructions

**Interview Preparation:**
1. Practice: "Design a trading system" (45 min whiteboard)
2. Practice: "Design an order matching engine"
3. Practice: "How would you handle 1M orders/second?"
4. Prepare: 3 stories about technical challenges solved

**Deliverables:**
1. Polished GitHub repository
2. System architecture document (5-10 pages)
3. Demo video or live deployment
4. Updated resume and LinkedIn
5. Interview preparation notes

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | README + documentation | 2.5 |
| Tue | Architecture diagrams | 2.5 |
| Wed | Code cleanup + examples | 3 |
| Thu | Demo video/deployment | 3 |
| Fri | Resume + LinkedIn update | 2.5 |
| Sat | Interview practice | 2.5 |
| Sun | Final review | 2 |

---

## Month 4 Success Checklist

### Knowledge Verification
- [ ] Can explain full trading platform architecture
- [ ] Understand low-latency concepts (kernel bypass, DPDK)
- [ ] Know C++ basics for trading systems
- [ ] Can discuss co-location and direct market access

### Technical Deliverables
- [ ] Integrated trading platform with all components
- [ ] Order Management System with state management
- [ ] C++ order book with Python bindings
- [ ] Web dashboard with real-time updates
- [ ] Docker Compose deployment

### Documentation
- [ ] System architecture document
- [ ] API reference documentation
- [ ] "Low-Latency Trading Concepts"
- [ ] "HFT Infrastructure Overview"
- [ ] Deployment guide

### Career Readiness
- [ ] Polished GitHub portfolio
- [ ] Updated resume with Fintech focus
- [ ] LinkedIn profile updated
- [ ] Can whiteboard trading system design
- [ ] 3 technical challenge stories prepared

---

# Final Capstone: Complete Trading Platform

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRADING PLATFORM ARCHITECTURE                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │  IB API      │    │ ICICI Breeze │    │  NSE Feed    │      │
│  │  Connector   │    │  Connector   │    │  (Backup)    │      │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘      │
│         │                   │                   │               │
│         └───────────────────┼───────────────────┘               │
│                             ▼                                    │
│                   ┌─────────────────┐                           │
│                   │  Market Data    │                           │
│                   │    Service      │                           │
│                   └────────┬────────┘                           │
│                            │                                     │
│         ┌──────────────────┼──────────────────┐                 │
│         ▼                  ▼                  ▼                 │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐           │
│  │   Signal    │   │  Screener   │   │  Backtest   │           │
│  │   Engine    │   │   Engine    │   │   Engine    │           │
│  └──────┬──────┘   └─────────────┘   └─────────────┘           │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────────────────────────────────────────┐            │
│  │              ORDER MANAGEMENT SYSTEM             │            │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐ │            │
│  │  │ Order   │  │  Risk   │  │   Compliance    │ │            │
│  │  │ Router  │  │ Engine  │  │    Module       │ │            │
│  │  └────┬────┘  └────┬────┘  └────────┬────────┘ │            │
│  └───────┼────────────┼────────────────┼──────────┘            │
│          │            │                │                        │
│          └────────────┼────────────────┘                        │
│                       ▼                                          │
│              ┌─────────────────┐                                │
│              │   Event Store   │                                │
│              │  (PostgreSQL)   │                                │
│              └────────┬────────┘                                │
│                       │                                          │
│         ┌─────────────┼─────────────┐                           │
│         ▼             ▼             ▼                           │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐                     │
│  │ Dashboard │ │    API    │ │  Alerts   │                     │
│  │   (Web)   │ │ (FastAPI) │ │  Service  │                     │
│  └───────────┘ └───────────┘ └───────────┘                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Component Summary

| Component | Month Built | Key Features |
|-----------|-------------|--------------|
| Order Book Simulator | Month 1 | Matching engine, price-time priority |
| Broker Abstraction | Month 1 | IB + Breeze unified interface |
| Market Data Service | Month 1 | WebSocket, tick aggregation |
| Event Bus | Month 2 | Redis Streams + ZeroMQ |
| Event Store | Month 2 | PostgreSQL, replay capability |
| FIX Parser | Month 2 | Message parsing, order gateway |
| Compliance Module | Month 3 | SEBI requirements, audit trail |
| Risk Engine | Month 3 | VaR, position limits, kill switch |
| Backtester | Month 3 | Event-driven, walk-forward |
| Signal Engine | Month 3 | Technical + fundamental signals |
| OMS | Month 4 | Order lifecycle, reconciliation |
| Dashboard | Month 4 | Real-time monitoring, API |

---

# High-Signal Resources

## Books (Priority Order)

### Must-Read (Complete During 4 Months)
| Book | Author | Focus | When to Read |
|------|--------|-------|--------------|
| **Designing Data-Intensive Applications** | Martin Kleppmann | System design, streaming, distributed systems | Month 1-2 |
| **Trading and Exchanges** | Larry Harris | Market microstructure, exchange mechanics | Month 1 |
| **Algorithmic Trading** | Ernie Chan | Practical strategies, backtesting | Month 2-3 |
| **Advances in Financial Machine Learning** | Marcos López de Prado | ML for trading, backtesting methodology | Month 3 |

### Reference (Dip In As Needed)
| Book | Author | Focus |
|------|--------|-------|
| **Options, Futures, and Other Derivatives** | John Hull | Derivatives pricing, Greeks, VaR |
| **Python for Algorithmic Trading** | Yves Hilpisch | Technical implementation |
| **High Performance Python** | Micha Gorelick | Python optimization |
| **C++ High Performance** | Viktor Sehr | C++ for trading systems |

### Advanced (Post-4 Months)
| Book | Author | Focus |
|------|--------|-------|
| **Market Microstructure in Practice** | Lehalle & Laruelle | Advanced microstructure |
| **Quantitative Trading** | Ernie Chan | Strategy development |
| **Machine Learning for Asset Managers** | Marcos López de Prado | ML portfolio construction |

---

## YouTube Channels & Playlists

### System Design & Architecture
| Channel | Content | Link |
|---------|---------|------|
| **Hussein Nasser** | Backend, databases, low-latency systems | youtube.com/@haboratory |
| **Code Karle** | System design interviews, trading systems | youtube.com/@codeKarle |
| **ByteByteGo** | System design visualizations | youtube.com/@ByteByteGo |
| **ArjanCodes** | Python best practices, performance | youtube.com/@ArjanCodes |

### Trading & Quantitative Finance
| Channel | Content | Link |
|---------|---------|------|
| **QuantInsti** | Algo trading tutorials, Python | youtube.com/@QuantInsti |
| **Coding Jesus** | Quant finance, Python trading | youtube.com/@CodingJesus |
| **Part Time Larry** | Python trading bots, APIs | youtube.com/@parttimelarry |
| **Algo Trading 101** | Strategy development | youtube.com/@AlgoTrading101 |

### Low-Latency & HFT
| Channel | Content | Link |
|---------|---------|------|
| **CppCon** | C++ performance, low-latency talks | youtube.com/@CppCon |
| **Trading Tech Talk** | HFT infrastructure discussions | Search on YouTube |

### Specific Videos to Watch
1. **Hussein Nasser:** "Kafka Architecture Explained"
2. **Hussein Nasser:** "WebSocket vs HTTP"
3. **Code Karle:** "Design a Stock Exchange"
4. **CppCon:** "Low Latency C++ for HFT" (Carl Cook)
5. **QuantInsti:** "Backtesting Pitfalls"

---

## Engineering Blogs & Case Studies

### Trading Infrastructure
| Company | Blog/Resource | Focus |
|---------|---------------|-------|
| **Jane Street** | janestreet.com/tech-talks | OCaml, low-latency, trading systems |
| **Two Sigma** | twosigma.com/insights | ML, risk management, infrastructure |
| **Citadel** | citadel.com/careers/technology | Technology overview |
| **IMC Trading** | imc.com/eu/articles | Market making, technology |

### Fintech Engineering
| Company | Blog | Focus |
|---------|------|-------|
| **Robinhood** | robinhood.engineering | Order management, scaling |
| **Stripe** | stripe.com/blog/engineering | Payment systems, reliability |
| **Square** | developer.squareup.com/blog | Financial APIs |
| **Coinbase** | blog.coinbase.com/engineering | Crypto trading infrastructure |

### General System Design
| Blog | Focus |
|------|-------|
| **Martin Fowler** (martinfowler.com) | Event Sourcing, CQRS, patterns |
| **High Scalability** (highscalability.com) | Architecture case studies |
| **Netflix Tech Blog** | Streaming, resilience patterns |
| **Uber Engineering** | Real-time systems, event sourcing |

### Must-Read Articles
1. **Martin Fowler:** "Event Sourcing" and "CQRS"
2. **Uber:** "Uber's Real-Time Push Platform"
3. **Netflix:** "Zuul 2: The Netflix Journey to Async"
4. **Jane Street:** "What is an Exchange?"

---

## Coursera Courses (Complete List)

### Core Trading & Quant
| Course | Provider | Duration | Link |
|--------|----------|----------|------|
| Fundamentals of Quantitative Modeling | Wharton | 1-2 weeks | coursera.org/learn/wharton-quantitative-modeling |
| Introduction to Trading, ML & GCP | Google Cloud | 2 weeks | coursera.org/learn/introduction-trading-machine-learning-gcp |
| Using Machine Learning in Trading | Google Cloud | 2-3 weeks | coursera.org/learn/machine-learning-trading-finance |
| Financial Engineering & Risk Mgmt Part I | Columbia | 3-4 weeks | coursera.org/learn/financial-engineering-risk-management |
| ML for Trading Specialization | Google/NYIF | 3 courses | coursera.org/specializations/machine-learning-trading |

### System Design & Cloud
| Course | Provider | Duration | Link |
|--------|----------|----------|------|
| Cloud Architecture Design Patterns | Coursera | 2 weeks | coursera.org/learn/cloud-architecture-design-patterns |
| Microservice Architectures | Vanderbilt | 2-3 weeks | Search on Coursera |

---

## Indian Market Specific Resources

### Regulatory & Compliance
| Resource | Content |
|----------|---------|
| **SEBI Website** (sebi.gov.in) | Circulars, regulations, algo trading framework |
| **NSE India** (nseindia.com) | Market structure, trading rules, circulars |
| **Zerodha Varsity** (zerodha.com/varsity) | Free modules on Indian markets, taxation |

### Data Sources
| Source | Data Type | Access |
|--------|-----------|--------|
| **NSE India** | Historical EOD data | Free (limited) |
| **BSE India** | Historical data | Free (limited) |
| **Kite Connect** (Zerodha) | Real-time + historical | Paid API |
| **ICICI Breeze** | Real-time + historical | With trading account |
| **Jugaad Data** | NSE historical | Python package |

### Broker APIs for Indian Markets
| Broker | API | Documentation |
|--------|-----|---------------|
| **ICICI Direct** | Breeze API | api.icicidirect.com |
| **Zerodha** | Kite Connect | kite.trade |
| **Angel One** | SmartAPI | smartapi.angelbroking.com |
| **Upstox** | Upstox API | upstox.com/developer |
| **5paisa** | 5paisa API | 5paisa.com/developerapi |

---

## Tools & Libraries

### Python Trading Libraries
| Library | Purpose |
|---------|---------|
| **pandas** | Data manipulation |
| **numpy** | Numerical computing |
| **ta-lib** | Technical indicators |
| **backtrader** | Backtesting framework |
| **zipline-reloaded** | Backtesting (Quantopian fork) |
| **vectorbt** | Vectorized backtesting |
| **pyfolio** | Portfolio analytics |
| **empyrical** | Risk metrics |

### Infrastructure
| Tool | Purpose |
|------|---------|
| **Redis** | Caching, pub/sub, streams |
| **PostgreSQL** | Primary database |
| **TimescaleDB** | Time-series data |
| **InfluxDB** | Metrics, time-series |
| **Kafka** | Event streaming |
| **ZeroMQ** | Low-latency messaging |

### Monitoring & Observability
| Tool | Purpose |
|------|---------|
| **Grafana** | Dashboards |
| **Prometheus** | Metrics collection |
| **Datadog** | APM, monitoring |
| **ELK Stack** | Log aggregation |

---

## Communities & Networking

### Online Communities
| Platform | Community |
|----------|-----------|
| **Reddit** | r/algotrading, r/quantfinance |
| **Discord** | Algo Trading servers |
| **LinkedIn** | Fintech/Quant groups |
| **Twitter/X** | Follow quant practitioners |

### Indian Fintech Community
| Platform | Focus |
|----------|-------|
| **Zerodha Varsity Forum** | Indian market discussions |
| **TradingQnA** | Indian trading community |
| **LinkedIn India Fintech** | Networking |

### Conferences & Events
| Event | Focus |
|-------|-------|
| **QuantCon** | Quantitative finance |
| **PyCon** | Python, trading tracks |
| **CppCon** | C++ performance |

---

# Timeline Summary (All 4 Months)

| Week | Focus | Key Deliverable |
|------|-------|-----------------|
| 1 | NSE/BSE Exchange Architecture | Order Book Simulator |
| 2 | Clearing, Settlement | Trade Lifecycle Tracker |
| 3 | Broker APIs | Multi-Broker Abstraction |
| 4 | Market Data & WebSocket | Real-Time Data Service |
| 5 | Event-Driven Architecture | Event Bus |
| 6 | Event Sourcing & CQRS | Trade Journal |
| 7 | Low-Latency Python | Optimized Order Router |
| 8 | AWS FinServ & FIX | FIX Parser + AWS Architecture |
| 9 | SEBI Compliance | Compliance Module |
| 10 | Risk Engine | VaR + Kill Switch |
| 11 | Backtesting | Event-Driven Backtester |
| 12 | Signal Generation | Screener Platform |
| 13 | Platform Integration | OMS + Full Integration |
| 14 | Low-Latency Deep Dive | C++ Order Book |
| 15 | Dashboard & API | Web Dashboard |
| 16 | Portfolio Polish | Job-Ready Portfolio |

---

# Post-Plan: Next Steps (Month 5+)

After completing this 4-month plan, consider these paths:

## Path A: Deep HFT Track (3-6 months additional)
- Learn C++ in depth (Modern C++, templates, memory management)
- Study FPGA basics (Verilog/VHDL awareness)
- Implement kernel bypass with DPDK
- Build lock-free data structures
- Target: HFT firms, prop shops

## Path B: ML/Quant Research Track (3-6 months additional)
- Deep dive into ML for trading (reinforcement learning)
- Alternative data integration
- Advanced factor models
- Research paper implementation
- Target: Quant researcher roles

## Path C: Trading Infrastructure Track (3-6 months additional)
- Kubernetes for trading workloads
- Multi-region deployment
- Disaster recovery design
- Exchange connectivity (co-location)
- Target: Trading platform architect roles

## Path D: Start Your Own Trading Business
- Paper trade with your platform for 3-6 months
- Develop 2-3 robust strategies with real edge
- Start with small capital (₹1-5 lakhs)
- Scale based on performance
- Target: Independent trader / small fund

---

**Plan Created:** January 2026  
**Duration:** 16 weeks (4 months)  
**Time Commitment:** 2-3 hours/day (~15-20 hrs/week)  
**Target Outcome:** Staff-Level Fintech Architect ready for Hedge Fund/Prop Shop roles

---

*"The goal is not to be a jack of all trades, but to be a Staff Engineer who can architect trading systems end-to-end while understanding the domain deeply enough to make the right trade-offs."*

---

**Previous:** [Month 3 - Compliance, Security & Quantitative Logic](./study-plan-month-3.md)  
**Start:** [Month 1 - Fintech Core & Domain Knowledge](./study-plan-month-1.md)

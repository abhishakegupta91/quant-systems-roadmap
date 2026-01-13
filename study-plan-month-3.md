# Elite Fintech Architect Study Plan
## Month 3: Compliance, Security & Quantitative Logic
 
**Target Role:** Hybrid Staff Engineer (Trading Platform + Quant Development)  
**Target Market:** Indian Markets (NSE/BSE) with global applicability  
**Duration:** Weeks 9-12 | 2-3 hours/day (~15-20 hrs/week)  
**Last Updated:** January 2026

---

## Month 3 Theme: SEBI Regulations, Risk Engines, Backtesting Architectures

**Focus:** Build institutional-grade compliance and risk management. This is what separates hobby projects from production trading systems that can handle real capital.

**Time Commitment:** ~15-20 hours/week (2-3 hrs/day)

**Prerequisites:** Complete Month 2 deliverables:
- [ ] Event bus with Redis Streams + ZeroMQ implementations
- [ ] Event-sourced trade journal with replay
- [ ] Optimized order router with Cython components
- [ ] FIX message parser
- [ ] AWS architecture diagram

---

## Week 9: SEBI Algo Trading Regulations & Compliance

### Learning Objectives
- Master SEBI algo trading framework and requirements
- Understand pre-trade and post-trade risk controls
- Learn audit trail requirements and reporting obligations

### Study Materials

**Theory (6-7 hours)**
- **SEBI Circulars (Mandatory Reading):**
  - SEBI/HO/MRD/DP/CIR/P/2021/127 (Algo Trading Framework)
  - SEBI/HO/MRD/DP/CIR/P/2022/57 (API-based Trading)
  - SEBI/HO/MRD/DP/CIR/P/2023/37 (Amendments)
- **NSE Circulars:** Algo trading registration requirements
- **Book:** "Algorithmic Trading" by Ernie Chan — Chapter on Regulatory Considerations

**Video Resources**
- **YouTube:** SEBI algo trading framework explanations
- **Zerodha Varsity:** Regulatory modules

### Hands-On (6-8 hours)

**Project: Compliance Module for Trading Platform**
```
Goal: Implement SEBI-compliant controls
Structure:
├── compliance/
│   ├── algo_registry.py    # Register algos with unique IDs
│   ├── order_tagging.py    # Tag orders with algo ID (SEBI requirement)
│   └── audit_logger.py     # Immutable audit trail
├── controls/
│   ├── order_limits.py     # Max order value, quantity limits
│   ├── price_bands.py      # Circuit breaker checks
│   └── symbol_whitelist.py # Approved instruments only
├── reports/
│   ├── daily_report.py     # End-of-day compliance report
│   └── exception_report.py # Violations and alerts
└── tests/
    └── compliance_tests.py # Verify all controls work
```

**SEBI Requirements to Implement:**
1. Unique algo ID tagging on every order
2. Order-level price band checks (within circuit limits)
3. Quantity limits per order
4. Kill switch capability
5. Audit trail with timestamps

**Deliverables:**
1. Compliance module with all SEBI-required controls
2. Audit logger with immutable storage
3. Daily compliance report generator
4. Document: "SEBI Algo Trading Compliance Checklist"

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | SEBI circular deep-dive (2021/127) | 2.5 |
| Tue | NSE requirements, API trading circular | 2.5 |
| Wed | Design compliance module architecture | 2.5 |
| Thu | Code: Algo registry + order tagging | 3 |
| Fri | Code: Order limits + audit logger | 3 |
| Sat | Code: Reports + testing | 2.5 |
| Sun | Documentation | 2 |

---

## Week 10: Real-Time Risk Engine

### Learning Objectives
- Implement real-time position and P&L tracking
- Build VaR (Value at Risk) calculations
- Design circuit breakers and kill switches

### Study Materials

**Theory (5-6 hours)**
- **Book:** "Options, Futures, and Other Derivatives" by Hull — VaR chapters
- **Risk Management Papers:** Basel framework basics
- **Engineering Blog:** Two Sigma risk management architecture

**Coursera**
- **Financial Engineering and Risk Management Part I** (Columbia) — Start
  - Link: https://www.coursera.org/learn/financial-engineering-risk-management

### Hands-On (8-10 hours)

**Project: Real-Time Risk Engine**
```
Goal: Production-grade risk monitoring and controls
Structure:
├── risk_engine/
│   ├── position_tracker.py   # Real-time position aggregation
│   ├── pnl_calculator.py     # Mark-to-market P&L
│   ├── var_calculator.py     # Historical VaR, Parametric VaR
│   └── greeks_tracker.py     # Options Greeks aggregation
├── limits/
│   ├── position_limits.py    # Max position per symbol/sector
│   ├── loss_limits.py        # Daily loss limits, drawdown
│   ├── concentration.py      # Sector/symbol concentration
│   └── margin_monitor.py     # Margin utilization tracking
├── alerts/
│   ├── alert_manager.py      # Alert routing
│   ├── kill_switch.py        # Emergency stop all trading
│   └── notifications.py      # Email/SMS/Slack alerts
└── dashboard/
    └── risk_dashboard.py     # Real-time risk visualization
```

**Risk Metrics to Implement:**
1. Real-time P&L (unrealized + realized)
2. Position Greeks (Delta, Gamma, Vega, Theta)
3. Historical VaR (95%, 99% confidence)
4. Maximum drawdown tracking
5. Margin utilization percentage

**Deliverables:**
1. Real-time risk engine with position tracking
2. VaR calculator (historical method)
3. Kill switch with immediate order cancellation
4. Risk dashboard (CLI or simple web)
5. Document: "Risk Engine Architecture"

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | VaR concepts, Hull chapters | 2.5 |
| Tue | Coursera: Financial Engineering | 2.5 |
| Wed | Risk engine architecture design | 2.5 |
| Thu | Code: Position tracker + P&L | 3 |
| Fri | Code: VaR calculator + limits | 3 |
| Sat | Code: Kill switch + alerts | 2.5 |
| Sun | Dashboard + documentation | 2 |

---

## Week 11: Backtesting Architecture

### Learning Objectives
- Design production-grade backtesting framework
- Understand look-ahead bias, survivorship bias, overfitting
- Implement walk-forward optimization

### Study Materials

**Theory (5-6 hours)**
- **Book:** "Advances in Financial Machine Learning" by López de Prado — Backtesting chapters
- **Book:** "Algorithmic Trading" by Ernie Chan — Backtesting methodology
- **Paper:** "The Probability of Backtest Overfitting" (Bailey et al.)

**Video Resources**
- **YouTube - QuantInsti:** Backtesting best practices
- **YouTube:** "Common Backtesting Mistakes" presentations

### Hands-On (8-10 hours)

**Project: Event-Driven Backtesting Framework**
```
Goal: Institutional-grade backtester with bias prevention
Structure:
├── backtester/
│   ├── engine.py            # Main backtest loop
│   ├── data_handler.py      # Historical data management
│   ├── execution_sim.py     # Realistic fill simulation
│   └── slippage_model.py    # Slippage and market impact
├── strategies/
│   ├── base.py              # Strategy interface
│   ├── momentum.py          # Example: Momentum strategy
│   └── mean_reversion.py    # Example: Mean reversion
├── analytics/
│   ├── performance.py       # Sharpe, Sortino, Calmar
│   ├── drawdown.py          # Drawdown analysis
│   └── trade_analysis.py    # Win rate, profit factor
├── validation/
│   ├── walk_forward.py      # Walk-forward optimization
│   ├── monte_carlo.py       # Monte Carlo simulation
│   └── cross_validation.py  # Time-series CV
└── reports/
    └── backtest_report.py   # Comprehensive report generation
```

**Bias Prevention Measures:**
1. Point-in-time data (no look-ahead)
2. Survivorship bias handling
3. Transaction cost modeling
4. Slippage estimation
5. Walk-forward validation

**Deliverables:**
1. Event-driven backtester with realistic execution
2. Walk-forward optimization module
3. Performance analytics suite
4. Backtest report generator
5. Document: "Backtesting Best Practices"

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | López de Prado backtesting chapters | 2.5 |
| Tue | Ernie Chan methodology, overfitting paper | 2.5 |
| Wed | Backtester architecture design | 2.5 |
| Thu | Code: Engine + data handler | 3 |
| Fri | Code: Execution sim + analytics | 3 |
| Sat | Code: Walk-forward + reports | 2.5 |
| Sun | Testing + documentation | 2 |

---

## Week 12: Signal Generation & Screening Engine

### Learning Objectives
- Build modular signal generation framework
- Implement stock screeners with custom criteria
- Design signal combination and ranking systems

### Study Materials

**Theory (5-6 hours)**
- **Book:** "Quantitative Trading" by Ernie Chan — Signal generation
- **Book:** "Python for Algorithmic Trading" by Yves Hilpisch — Technical indicators
- **QuantInsti:** Factor-based screening tutorials

**Coursera**
- **Machine Learning for Trading Specialization** — Continue
  - Link: https://www.coursera.org/specializations/machine-learning-trading

### Hands-On (8-10 hours)

**Project: Signal Generation & Screening Platform**
```
Goal: Flexible signal generation for Indian markets
Structure:
├── signals/
│   ├── base.py              # Signal interface
│   ├── technical/
│   │   ├── momentum.py      # RSI, MACD, ROC
│   │   ├── trend.py         # Moving averages, ADX
│   │   └── volatility.py    # Bollinger, ATR
│   ├── fundamental/
│   │   ├── valuation.py     # P/E, P/B, EV/EBITDA
│   │   └── quality.py       # ROE, debt ratios
│   └── composite/
│       └── multi_factor.py  # Combine multiple signals
├── screener/
│   ├── universe.py          # Define stock universe (NIFTY50, etc.)
│   ├── filters.py           # Filter criteria
│   ├── ranker.py            # Rank by signal strength
│   └── scheduler.py         # Scheduled screening runs
├── alerts/
│   ├── signal_alerts.py     # Alert on signal triggers
│   └── watchlist.py         # Watchlist management
└── api/
    └── screener_api.py      # REST API for screener
```

**Signals to Implement:**
1. Technical: RSI, MACD, Moving Average Crossover
2. Momentum: 12-1 month momentum, 52-week high proximity
3. Volatility: ATR-based position sizing signals
4. Composite: Multi-factor ranking

**Deliverables:**
1. Signal generation framework with 10+ signals
2. Stock screener with custom criteria
3. Multi-factor ranking system
4. REST API for screener access
5. Document: "Signal Generation Framework"

### Daily Breakdown
| Day | Activity | Hours |
|-----|----------|-------|
| Mon | Ernie Chan signal generation | 2.5 |
| Tue | Hilpisch technical indicators | 2.5 |
| Wed | Coursera: ML for Trading | 2.5 |
| Thu | Code: Signal base + technical signals | 3 |
| Fri | Code: Screener + ranker | 3 |
| Sat | Code: API + alerts | 2.5 |
| Sun | Testing + documentation | 2 |

---

## Month 3 Success Checklist

### Knowledge Verification
- [ ] Can explain SEBI algo trading requirements
- [ ] Understand VaR calculation methods
- [ ] Can identify common backtesting biases
- [ ] Know factor-based signal generation approaches

### Technical Deliverables
- [ ] SEBI-compliant compliance module
- [ ] Real-time risk engine with VaR
- [ ] Event-driven backtesting framework
- [ ] Signal generation & screening platform
- [ ] Kill switch implementation

### Documentation
- [ ] "SEBI Algo Trading Compliance Checklist"
- [ ] "Risk Engine Architecture"
- [ ] "Backtesting Best Practices"
- [ ] "Signal Generation Framework"

### Interview Readiness
- [ ] Can explain SEBI algo trading framework
- [ ] Can design risk management system
- [ ] Can discuss backtesting methodology and pitfalls
- [ ] Can whiteboard signal generation pipeline

---

## Month 3 Books & Resources

| Resource | Type | Focus |
|----------|------|-------|
| Algorithmic Trading (Ernie Chan) | Book | Backtesting, signals |
| Advances in Financial Machine Learning | Book | Backtesting methodology |
| Options, Futures, and Other Derivatives | Book | VaR, risk |
| Python for Algorithmic Trading | Book | Technical indicators |
| Financial Engineering & Risk Mgmt | Coursera | Risk concepts |
| ML for Trading Specialization | Coursera | Continue |
| SEBI Circulars | Official | Compliance |

---

## YouTube Channels for Month 3

| Channel | Content | Link |
|---------|---------|------|
| **QuantInsti** | Backtesting, algo trading | youtube.com/@QuantInsti |
| **Coding Jesus** | Quant finance, Python | youtube.com/@CodingJesus |
| **Part Time Larry** | Trading bots, APIs | youtube.com/@parttimelarry |

---

**Previous:** [Month 2 - Advanced System Design](./study-plan-month-2.md)  
**Next:** [Month 4 - Portfolio Transformation & Capstone](./study-plan-month-4.md)

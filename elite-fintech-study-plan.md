# Quant Systems Roadmap
## 16-Week Intensive Program for Staff-Level Trading Systems Engineer

**Prepared for:** Abhishake Gupta — 12+ years Python/Backend, BTech ECE, IB API experience  
**Target Role:** Hybrid Staff Engineer (Trading Platform + Quant Development)  
**Target Market:** Indian Markets (NSE/BSE) with global applicability  
**Duration:** 16 weeks (~4 months) | 2-3 hours/day (~15-20 hrs/week)  
**Last Updated:** January 2026

---

## Executive Summary

This plan transforms you from a **Senior Backend Developer with trading interest** into a **Staff-Level Fintech Architect** capable of:
- Designing and building production trading platforms (IB API, ICICI Breeze, NSE/BSE feeds)
- Architecting low-latency systems with sub-millisecond awareness
- Implementing institutional-grade risk engines and compliance frameworks
- Leading technical interviews at hedge funds, prop shops, and fintech firms

---

## Gap Analysis: What's Missing for Elite Fintech Roles

### Your Current Strengths ✅
| Strength | Evidence |
|----------|----------|
| Python Backend Mastery | 12+ years, microservices, REST APIs, Celery, Redis |
| DevOps & Infrastructure | Docker, K8s, CI/CD, ELK/Splunk/Datadog |
| Trading Foundation | IB WebSocket app, Options Greeks, Factor Investing courses |
| System Design Basics | Multi-provider orchestration, failover mechanisms |

### Critical Gaps to Close 🔴

| Gap | Why It Matters | Month Addressed |
|-----|----------------|-----------------|
| **Exchange Mechanics (NSE/BSE)** | Must understand order matching, clearing (NSCCL), settlement (T+1) | Month 1 |
| **FIX Protocol & Market Data Feeds** | Industry standard for institutional order routing | Month 1-2 |
| **Low-Latency Fundamentals** | WebSockets at scale, UDP multicast, kernel bypass concepts | Month 2 |
| **Event Sourcing & CQRS** | Audit trails, trade lifecycle, reconciliation | Month 2 |
| **SEBI Compliance & Risk Controls** | Algo trading regulations, pre-trade risk checks | Month 3 |
| **Real-Time Risk Engines** | VaR, position limits, circuit breakers | Month 3 |
| **C++ for Critical Paths** | Hot paths in HFT are never Python | Month 4 |
| **Production Trading Platform** | End-to-end system proving Staff-level capability | Month 4 |

---

## Prerequisites Checklist

Before starting, ensure you have:

### Technical Setup
- [ ] Python 3.11+ with virtual environment management (pyenv/conda)
- [ ] Docker Desktop installed and running
- [ ] Git configured with SSH keys
- [ ] IDE with Python debugging (VS Code/PyCharm)
- [ ] IB TWS or Gateway installed (paper trading account active)
- [ ] ICICI Direct Breeze API credentials (apply if not done)
- [ ] Redis and PostgreSQL running locally (Docker recommended)

### Accounts & Access
- [ ] Interactive Brokers paper trading account
- [ ] ICICI Direct trading account with API access
- [ ] NSE historical data access (nseindia.com or data vendor)
- [ ] GitHub account for portfolio projects
- [ ] Coursera subscription (for linked courses)

### Knowledge Baseline (Self-Assess)
- [ ] Can implement REST APIs with FastAPI/Flask in < 30 mins
- [ ] Understand async/await in Python
- [ ] Familiar with message queues (RabbitMQ/Redis)
- [ ] Can read and write basic SQL queries
- [ ] Understand TCP/IP networking basics

---

## Plan Structure

Each month follows this pattern:
1. **Domain Knowledge** — Theory, concepts, regulations
2. **Technical Deep-Dive** — Architecture patterns, protocols
3. **Hands-On Build** — Incremental project work
4. **Success Checklist** — Verifiable outcomes

**Capstone Project (Built Incrementally):**  
A production-grade **Indian Market Trading Platform** with:
- Multi-broker abstraction (IB + ICICI Breeze)
- Real-time market data ingestion
- Signal generation & screening engine
- Order management with risk controls
- Event-sourced trade journal
- Backtesting framework

---

## Monthly Study Plans (Canonical)

- **Month 1 (Weeks 1-4): Fintech Core & Domain Knowledge**
  - [study-plan-month-1.md](./study-plan-month-1.md)
- **Month 2 (Weeks 5-8): Advanced System Design**
  - [study-plan-month-2.md](./study-plan-month-2.md)
- **Month 3 (Weeks 9-12): Compliance, Security & Quantitative Logic**
  - [study-plan-month-3.md](./study-plan-month-3.md)
- **Month 4 (Weeks 13-16): Portfolio Transformation & Capstone**
  - [study-plan-month-4.md](./study-plan-month-4.md)

---

## Notes

- This file is the *entry point* (overview + prerequisites + links).
- Detailed weekly breakdowns, hands-on builds, success checklists, and resources live in the month files.

For capstone architecture, resources, and post-plan next steps, refer to:

- [study-plan-month-4.md](./study-plan-month-4.md)

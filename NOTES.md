# Implementation & Architecture Notes

This document captures key architectural decisions, design trade-offs, deployment configuration, and notes on the implementation of the Sanctum Sanctorum bookstore system.

---

## 🚀 Live Deployment & Environment

- **Live URL**: TBD *(To be updated upon deploying to public host)*
- **Database**:
  - **Local Development & Testing**: SQLite (in-memory for automated deterministic tests via `conftest.py`, local `sanctum.db` file for local development server).
  - **Production / Hosted Deployment**: Hosted PostgreSQL (e.g., Supabase / Neon / Render) configured via the `SANCTUM_DATABASE_URL` environment variable.
- **Database Engine Adaptation**:
  - `app/db.py` adapts its `create_engine` parameters dynamically: `check_same_thread: False` is only applied when using SQLite connections, ensuring full compatibility when connected to PostgreSQL. Legacy `postgres://` connection prefixes are automatically normalized to `postgresql://`.

---

## 📚 Books Catalogue & Search

*(Detailed notes on ISBN-13 checksum validation, duplicate conflict prevention, partial updates, and catalogue filtering/sorting/pagination will be documented here).*

---

## 👥 Membership & Tier Access Control

*(Detailed notes on email normalization, case-insensitive uniqueness checks, tier access hierarchies, and member statistics aggregation will be documented here).*

---

## 🛒 Order Processing & Inventory Reservation

*(Detailed notes on order payload validation, volume/tier discount pricing formulas, atomic stock reservation/restoration, and lifecycle state machines will be documented here).*

---

## 📖 Library Lending & Returns

*(Detailed notes on loan model extensions, borrow quota enforcement, read-time dynamic loan status calculation, return restoration, and daily stepped late fees will be documented here).*

---

## 📊 Reporting & Analytics

*(Detailed notes on best-selling books aggregation and sales reporting queries will be documented here).*

---

## 🤖 AI Usage

- **Tools Used**: Google Antigravity (Gemini 3.7 Flash) for code analysis, architecture design, and incremental test-driven implementation.
- **Workflow & Scaffolding**: Leveraged AI for systematic code inspection against `SPEC.md`, mapping acceptance criteria to Pydantic schemas and SQLAlchemy ORM models, and organizing incremental atomic commits.
- **Critical Oversight & Human Verification**: All generated logic, mathematical formulas (discounts, late fees, ISBN checksums), and exception handling orders are rigorously validated against pytest suites and specific domain constraints.

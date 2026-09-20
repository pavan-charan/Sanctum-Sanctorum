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

- **ISBN-13 Normalization & Checksum Validation**:
  - Implemented `normalize_isbn13` to strip hyphens and whitespace, ensure exact 13-digit length, and validate the standard EAN/ISBN-13 modulo-10 checksum ($10 - (\sum w_i d_i \pmod{10})) \pmod{10}$ with alternating weights 1 and 3. Rejects invalid formats or checksum mismatches with HTTP 422.
- **Catalogue Duplicate Prevention**:
  - `create_book` checks against normalized ISBN values prior to persistence, returning HTTP 409 Conflict if the ISBN is already registered.
- **Partial Book Updates (`PATCH /books/{id}`)**:
  - Exposed `PATCH /books/{id}` supporting partial updates of mutable attributes (`title`, `author`, `price_cents`, `stock`, `restricted`).
  - Strict Pydantic model validation with `reject_explicit_nulls` enforces that explicit `null` updates are rejected with 422, while `isbn` modifications and unknown properties are safely and silently ignored per spec.
- **Catalogue Search, Filtering, Sorting & Pagination**:
  - Flexible query parameter parsing with `q` (case-insensitive substring match across title or author with automatic escaping).
  - Price bounding with inclusive `min_price` and `max_price` limits.
  - Pre-pagination total count computed dynamically over filtered query subsets using `select(func.count()).select_from(...)`.
  - Multi-attribute sort support (`title`, `-title`, `price`, `-price`) with secondary `id ASC` tie-breaking for deterministic ordering across page boundaries.


---

## 👥 Membership & Tier Access Control

- **Email Normalization & Case-Insensitive Uniqueness**:
  - `MemberCreate` Pydantic validator trims surrounding whitespace and downcases all email strings prior to regex pattern verification.
  - `create_member` service executes a case-insensitive lookup (`Member.email.ilike(...)`) to guarantee uniqueness across all case variations, raising HTTP 409 Conflict if already registered.
- **Tier Access Control**:
  - Tiers follow strict ordinal ranking: `apprentice` (0) < `adept` (1) < `master` (2) < `supreme` (3).
  - Restricted materials access is enforced via `tier_at_least(member.tier, RESTRICTED_MIN_TIER)` using inclusive index comparisons ($\ge$), granting access to `master` and `supreme` members while blocking `apprentice` and `adept` with HTTP 403 Forbidden.
- **Member Activity Statistics Aggregation**:
  - Implemented `get_member_stats` (`GET /members/{id}/stats`) which aggregates real-time metrics for a member:
    - `orders_paid` and `total_spent_cents`: Computed exclusively over orders in `paid` status.
    - `active_loans`: Count of unreturned loans (`returned_at is None`), including overdue loans.
    - `overdue_loans`: Count of active loans where current clock time exceeds `due_at` (`now > due_at`).
    - `late_fees_cents`: Sum of accumulated late fees across all returned loans.
    - Returns HTTP 404 if the requested member does not exist.

---

## 🛒 Order Processing & Inventory Reservation

- **Pre-Execution Payload Validation**:
  - Pydantic schema validation on `OrderCreate` strictly enforces non-empty item lists and rejects duplicate `book_id` references within the same order with HTTP 422 Unprocessable Entity before any database transaction begins.
- **Pricing & Multi-Tier Volume Discount Calculation**:
  - Base discount determined by membership tier (`apprentice`: 0%, `adept`: 5%, `master`: 10%, `supreme`: 15%).
  - Bulk discount (+5%) applied cumulatively when total item count across all order lines $\ge 10$.
  - Line total integer math with explicit integer floor division (`subtotal * percent // 100`) guarantees zero floating-point rounding discrepancies.
- **Atomic Inventory Reservation**:
  - Pre-flight stock validation across all order items guarantees all-or-nothing stock deduction.
  - Decrements book stock at the point of `pending` order creation, preventing race conditions or overselling.
- **Order Lifecycle & Idempotency**:
  - `pay_order`: Transitions `pending` $\to$ `paid`. Stock remains reserved. Rejects subsequent modifications with HTTP 409 Conflict.
  - `cancel_order`: Transitions `pending` $\to$ `cancelled` and atomically restores all reserved inventory (`item.book.stock += item.quantity`). Rejects attempts to cancel non-pending orders with HTTP 409 Conflict.

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

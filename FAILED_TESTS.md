# Failed Tests Documentation & Status Report

This document contains the complete execution results of running `uv run pytest` against the **Sanctum Sanctorum** test suite, detailing all **129 unresolved test cases (125 failures, 4 setup errors)** across each module with their failure reasons, missing functionality, and acceptance criteria.

---

## 📊 Summary Metrics

| Metric | Count / Percentage |
| :--- | :--- |
| **Total Collected Tests** | 202 |
| **Passed Tests** | 73 (36.1%) |
| **Failed Tests** | 125 (61.9%) |
| **Setup Errors** | 4 (2.0%) |
| **Total Unresolved** | **129 / 202 (63.9%)** |

### Module Status Overview

| Test Module | Total Tests | Passed | Failed | Errors | Status |
| :--- | :---: | :---: | :---: | :---: | :---: |
| [`tests/test_health.py`](tests/test_health.py) | 1 | 1 | 0 | 0 | ✅ **100% Passed** |
| [`tests/test_books.py`](tests/test_books.py) | 68 | 39 | 29 | 0 | ❌ **29 Failing** |
| [`tests/test_loans.py`](tests/test_loans.py) | 48 | 6 | 38 | 4 | ❌ **42 Failing/Error** |
| [`tests/test_members.py`](tests/test_members.py) | 22 | 14 | 8 | 0 | ❌ **8 Failing** |
| [`tests/test_orders.py`](tests/test_orders.py) | 51 | 10 | 41 | 0 | ❌ **41 Failing** |
| [`tests/test_reports.py`](tests/test_reports.py) | 12 | 3 | 9 | 0 | ❌ **9 Failing** |

---

## 🔍 Detailed Module Breakdown

---

### 1. Books Module (`tests/test_books.py`)
**Total Failures: 29**

#### `TestCreateBook` (3 Failures)
* `test_isbn_with_bad_checksum_returns_422`: ISBN-13 checksum validation is not implemented (received `201 Created` instead of `422 Unprocessable Entity`).
* `test_duplicate_isbn_returns_409`: Duplicate ISBN raises unhandled database `IntegrityError` instead of returning `409 Conflict`.
* `test_duplicate_isbn_detected_after_normalization`: Hyphenated/spaced ISBN strings are not normalized before duplicate checks.

#### `TestPatchBook` (11 Failures)
* `test_partial_update_changes_only_given_fields`: `PATCH /books/{id}` returned `405 Method Not Allowed` (endpoint route not registered or handled).
* `test_update_all_patchable_fields`: Missing `PATCH` route handling.
* `test_update_is_persisted`: Missing `PATCH` persistence.
* `test_title_is_stripped_on_update`: Title whitespace stripping missing on update.
* `test_isbn_is_ignored`: ISBN mutation protection on `PATCH` not implemented.
* `test_invalid_update_returns_422_and_changes_nothing[changes0]`: Negative price validation (`422`) not handled.
* `test_invalid_update_returns_422_and_changes_nothing[changes1]`: Negative stock validation (`422`) not handled.
* `test_invalid_update_returns_422_and_changes_nothing[changes2]`: Empty title validation (`422`) not handled.
* `test_invalid_update_returns_422_and_changes_nothing[changes3]`: Empty author validation (`422`) not handled.
* `test_invalid_update_returns_422_and_changes_nothing[changes4]`: Whitespace-only title validation (`422`) not handled.
* `test_patch_missing_book_returns_404`: Non-existent book `404 Not Found` response not implemented.

#### `TestListBooks` (15 Failures)
* `test_q_matches_author_case_insensitively`: Search filter `q` by author substring missing.
* `test_q_matches_title_or_author`: Search filter `q` by title/author substring missing.
* `test_price_range_is_inclusive`: Price range filtering (`min_price` and `max_price`) missing.
* `test_min_price_only`: Lower bound price filtering missing.
* `test_max_price_only`: Upper bound price filtering missing.
* `test_filters_combine`: Combining search query `q` and price filters missing.
* `test_sort_by_title_with_id_tie_break`: Sorting by `title ASC` with `id ASC` tie-breaker missing.
* `test_sort_by_title_descending_with_id_tie_break`: Sorting by `title DESC` with `id ASC` tie-breaker missing.
* `test_sort_by_price_with_id_tie_break`: Sorting by `price_cents ASC` with `id ASC` tie-breaker missing.
* `test_sort_by_price_descending_with_id_tie_break`: Sorting by `price_cents DESC` with `id ASC` tie-breaker missing.
* `test_pagination_with_limit_and_offset`: Pagination query parameters (`limit`, `offset`) not applied.
* `test_default_limit_is_20`: Default limit of 20 items not enforced.
* `test_offset_past_end_returns_no_items_but_total`: Empty items slice with correct `total` count missing.
* `test_total_counts_filtered_results_before_pagination`: `total` field does not reflect filtered total count prior to pagination.
* `test_sort_applies_before_pagination`: Sorting order is not applied before pagination windowing.

---

### 2. Loans Module (`tests/test_loans.py`)
**Total Failures/Errors: 42 (38 Failures, 4 Setup Errors)**

#### `TestBorrow` (19 Failures)
* `test_borrow_returns_201_with_loan`: `POST /loans` returned `501 Not implemented`.
* `test_borrow_uses_current_clock_time`: Borrow date not using injected clock.
* `test_borrow_decrements_stock`: Book stock not decremented on borrow.
* `test_loan_can_be_fetched`: `GET /loans/{id}` missing.
* `test_get_missing_loan_returns_404`: `404 Not Found` for missing loan ID.
* `test_missing_member_returns_404`: Non-existent member ID returns `404`.
* `test_missing_book_returns_404`: Non-existent book ID returns `404`.
* `test_restricted_book_below_master_returns_403[apprentice]`: Restricted book check (`403 Forbidden`) for `apprentice` tier.
* `test_restricted_book_below_master_returns_403[adept]`: Restricted book check (`403 Forbidden`) for `adept` tier.
* `test_restricted_book_allowed_for_master_and_above[master]`: Master tier borrow permission missing.
* `test_restricted_book_allowed_for_master_and_above[supreme]`: Supreme tier borrow permission missing.
* `test_out_of_stock_returns_409`: Zero stock borrow conflict (`409 Conflict`) not handled.
* `test_last_copy_borrowed_by_someone_else_returns_409`: Concurrent out-of-stock borrow conflict not handled.
* `test_same_book_twice_returns_409`: Active duplicate borrow by same member (`409 Conflict`) not handled.
* `test_same_book_can_be_borrowed_again_after_return`: Re-borrowing after return not supported.
* `test_overdue_loan_blocks_borrowing`: Overdue active loan blocking new loans (`409 Conflict`) missing.
* `test_loan_due_right_now_does_not_block_borrowing`: Due time exact boundary check missing.
* `test_returned_overdue_loan_no_longer_blocks`: Unblocking after return of overdue loan missing.
* `test_403_checked_before_overdue`: Error priority precedence (`403` before `409`) missing.

#### `TestLoanLimits` (5 Failures)
* `test_tier_limit[apprentice-1]`: Apprentice max 1 active loan limit missing.
* `test_tier_limit[adept-3]`: Adept max 3 active loans limit missing.
* `test_tier_limit[master-5]`: Master max 5 active loans limit missing.
* `test_supreme_has_no_limit`: Supreme unlimited loans logic missing.
* `test_returned_loans_do_not_count_toward_limit`: Returned loans exclusion from active quota missing.

#### `TestReturnLoan` (11 Failures)
* `test_return_sets_returned_at_and_status`: `POST /loans/{id}/return` returned `501 Not implemented`.
* `test_return_restores_stock`: Restoring book stock (+1) on return not handled.
* `test_return_twice_returns_409`: Returning already returned loan (`409 Conflict`) not handled.
* `test_return_missing_loan_returns_404`: Returning non-existent loan `404 Not Found` missing.
* `test_late_fee[elapsed0-0]`: Late fee calculation (0 days overdue = 0 fee) missing.
* `test_late_fee[elapsed1-0]`: Late fee calculation (on-time = 0 fee) missing.
* `test_late_fee[elapsed2-25]`: Late fee calculation (1 day overdue = 25 cents) missing.
* `test_late_fee[elapsed3-50]`: Late fee calculation (2 days overdue = 50 cents) missing.
* `test_late_fee[elapsed4-100]`: Late fee calculation (4 days overdue = 100 cents) missing.
* `test_late_fee_is_capped_at_book_price`: Late fee cap at book price missing.
* `test_late_fee_is_persisted`: Persisting late fee on loan record missing.

#### `TestLoanStatus` (3 Failures)
* `test_status_is_active_until_due`: Active status computation before due date missing.
* `test_status_becomes_overdue_after_due`: Automatic overdue status transition after due date missing.
* `test_returned_late_loan_stays_returned`: Retaining `returned` status for late returns missing.

#### `TestMemberLoans` (2 Failures, 4 Setup Errors)
* `test_excludes_other_members_loans`: Filtering loans by member ID missing.
* `test_missing_member_returns_404`: Non-existent member `404 Not Found` missing.
* `test_lists_all_loans_by_id`: Setup Error (Fixture failed because `POST /loans` returned `501` without `id`).
* `test_status_filter[overdue-1]`: Setup Error.
* `test_status_filter[returned-2]`: Setup Error.
* `test_status_filter[active-3]`: Setup Error.

---

### 3. Members Module (`tests/test_members.py`)
**Total Failures: 8**

#### `TestCreateMember` (3 Failures)
* `test_email_is_stripped_and_lowercased`: Email normalization (`strip()`, `lower()`) missing.
* `test_duplicate_email_returns_409`: Duplicate member email raises unhandled `IntegrityError` instead of `409 Conflict`.
* `test_duplicate_email_is_case_insensitive`: Case-insensitive uniqueness check on email missing.

#### `TestMemberStats` (5 Failures)
* `test_new_member_has_zero_stats`: `GET /members/{id}/stats` returned `501 Not implemented`.
* `test_order_stats_count_only_paid_orders`: Paid orders count & spend summation missing.
* `test_loan_stats`: Active, overdue, and returned loan stats aggregation missing.
* `test_loan_due_exactly_now_counts_as_active_not_overdue`: Exact boundary condition check for active vs overdue missing.
* `test_stats_for_missing_member_returns_404`: Non-existent member ID returns `404 Not Found`.

---

### 4. Orders Module (`tests/test_orders.py`)
**Total Failures: 41**

#### `TestCreateOrder` (4 Failures)
* `test_create_returns_201_pending_order`: `POST /orders` returned `501 Not implemented`.
* `test_created_at_is_current_clock_time`: Order timestamp not set to current clock.
* `test_items_keep_submitted_order`: Order items ordering not preserved.
* `test_order_can_be_fetched`: `GET /orders/{id}` missing.

#### `TestOrderPricing` (10 Failures)
* `test_tier_discount[apprentice-0-1000]`: Apprentice discount calculation (0%) missing.
* `test_tier_discount[adept-5-950]`: Adept discount calculation (5%) missing.
* `test_tier_discount[master-10-900]`: Master discount calculation (10%) missing.
* `test_tier_discount[supreme-15-850]`: Supreme discount calculation (15%) missing.
* `test_bulk_discount_at_exactly_10_copies`: Bulk discount (10% on >=10 copies) missing.
* `test_no_bulk_discount_at_9_copies`: Bulk threshold verification missing.
* `test_bulk_threshold_counts_quantity_across_items`: Multi-item aggregate quantity bulk check missing.
* `test_tier_and_bulk_discounts_add_up`: Combined additive discount calculation missing.
* `test_discount_is_rounded_down`: Floor rounding of discount calculation missing.
* `test_unit_price_is_frozen_at_order_time`: Freezing book unit price at order creation missing.

#### `TestOrderStock` (5 Failures)
* `test_stock_is_reserved_on_creation`: Stock deduction/reservation on `pending` order creation missing.
* `test_ordering_entire_stock_is_allowed`: Ordering 100% available stock not handled.
* `test_insufficient_stock_returns_409`: Insufficient stock returns `409 Conflict` missing.
* `test_reserved_stock_is_unavailable_to_later_orders`: Reserved stock isolation between orders missing.
* `test_insufficient_stock_is_all_or_nothing`: Atomic inventory reservation rollback missing.

#### `TestOrderValidation` (11 Failures)
* `test_empty_items_returns_422`: Validation error (`422`) for empty `items` array missing.
* `test_duplicate_book_returns_422`: Validation error (`422`) for duplicate book IDs in `items` missing.
* `test_missing_member_returns_404`: Non-existent member ID returns `404 Not Found`.
* `test_missing_book_returns_404`: Non-existent book ID returns `404 Not Found`.
* `test_restricted_book_below_master_returns_403[apprentice]`: Apprentice ordering restricted book returns `403 Forbidden`.
* `test_restricted_book_below_master_returns_403[adept]`: Adept ordering restricted book returns `403 Forbidden`.
* `test_restricted_book_allowed_for_master_and_above[master]`: Master ordering restricted book allowed.
* `test_restricted_book_allowed_for_master_and_above[supreme]`: Supreme ordering restricted book allowed.
* `test_422_checked_before_missing_member`: Precedence check (`422` before `404`) missing.
* `test_404_checked_before_restricted`: Precedence check (`404` before `403`) missing.
* `test_403_checked_before_insufficient_stock`: Precedence check (`403` before `409`) missing.

#### `TestPayOrder` (4 Failures)
* `test_pay_pending_order`: `POST /orders/{id}/pay` returned `501 Not implemented`.
* `test_pay_keeps_stock_reserved`: Stock remaining reserved after payment missing.
* `test_pay_twice_returns_409`: Paying already paid order returns `409 Conflict`.
* `test_pay_cancelled_order_returns_409`: Paying cancelled order returns `409 Conflict`.

#### `TestCancelOrder` (4 Failures)
* `test_cancel_pending_order`: `POST /orders/{id}/cancel` returned `501 Not implemented`.
* `test_cancel_restores_stock_of_every_item`: Restoring reserved stock (+quantity) on cancellation missing.
* `test_cancel_twice_returns_409_and_does_not_restore_again`: Cancelling already cancelled order returns `409 Conflict`.
* `test_cancel_paid_order_returns_409`: Cancelling paid order returns `409 Conflict`.

#### `TestMemberOrders` (1 Failure)
* `test_lists_member_orders_by_id_with_any_status`: `GET /members/{id}/orders` returned `501 Not implemented`.

---

### 5. Reports Module (`tests/test_reports.py`)
**Total Failures: 9**

#### `TestTopBooks` (9 Failures)
* `test_no_orders_returns_empty_list`: `GET /reports/top-books` returned `501 Not implemented` or empty logic failure.
* `test_counts_paid_orders`: Only counting paid order items towards copies sold missing.
* `test_pending_and_cancelled_orders_are_excluded`: Exclusion of pending/cancelled orders missing.
* `test_books_with_only_unpaid_orders_are_omitted`: Omitting books with zero paid sales missing.
* `test_sorted_by_copies_desc_then_title_asc`: Sorting by `copies_sold DESC` then `title ASC` missing.
* `test_default_limit_is_5`: Default limit of 5 missing.
* `test_limit_parameter`: Custom `limit` query param handling missing.
* `test_limit_bounds_are_accepted[1]`: Boundary condition (`limit=1`) validation missing.
* `test_limit_bounds_are_accepted[50]`: Boundary condition (`limit=50`) validation missing.

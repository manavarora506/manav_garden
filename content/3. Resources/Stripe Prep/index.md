# Stripe Interview Prep

## Sessions

| Date | Problem | Category | Parts | Key Takeaway |
|------|---------|----------|-------|--------------|
| 2026-02-06 | [[http-language-preference|HTTP Language Preference]] | Parsing | 3/4 | Single-pass over header preserves preference order; condition ordering in elif chains matters |
| 2026-02-09 | [[worker-task-assignment|Worker Task Assignment]] | Data Processing | 4/4 | Separate static lookups from dynamic state; partition candidates before selecting; watch off-by-one with "after" semantics |
| 2026-02-09 | [[supply-chain-optimization|Supply Chain Optimization]] | Data Processing / DP | 3/4 | 2D DP where state = (stage, factory choice); greedy breaks when consecutive choices are coupled — need more DP practice |
| 2026-02-09 | [[account-balance-manager|Account Balance Manager]] | Financial Logic | 3/3 | Treat special accounts uniformly in the balance dict; coverage = -(balance + amount); always update balance after coverage |
| 2026-02-09 | [[bit-font-renderer|Bit Font Renderer]] | Parsing | 2.5/3 | Nested loop structure matters — decode each char within each row within each letter; reset state per row; return don't print |
| 2026-02-09 | [[payment-invoice-reconciliation|Payment Invoice Reconciliation]] | Parsing / Financial Logic | 3/3 | Three-tier matching priority (ID → exact → fuzzy); sort by date upfront so first match = earliest; DRY up repeated parsing |
| 2026-02-09 | [[business-account-data-verification|Business Account Data Verification]] | Parsing / Data Processing | 5/5 | if/elif chain for priority-ordered validation; all()/any() for clean boolean checks; normalize before comparing; guard division by zero |
| 2026-02-09 | [[currency-exchange-rate-converter|Currency Exchange Rate Converter]] | API Design / Data Processing | 4/4 | All-paths BFS needs per-path visited sets (don't mutate shared state); store both forward/reverse rates at parse time; O(V!) for all paths |
| 2026-02-09 | [[user-record-linkage|User Record Linkage]] | Data Processing / API Design | 3/3 | BFS depth limit for hop control; seed queue with target to unify initial + hop logic; shared visited for component (vs per-path for all paths) |
| 2026-02-13 | [[bank-balance-rebalancing|Bank Balance Rebalancing]] | Financial Logic | 3/3 | min(needed, excess) for transfers; exact-match hashmap pre-processing for minimum moves; production = plan then execute with idempotency |
| 2026-02-09 | [[user-record-linkage|User Record Linkage]] | Data Processing / API Design | 3/3 | BFS depth limit for hop control; seed queue with target to unify initial + hop logic; shared visited for component (vs per-path for all paths) |
| 2026-02-13 | [[bank-balance-rebalancing|Bank Balance Rebalancing]] | Financial Logic | 3/3 | min(needed, excess) for transfers; exact-match hashmap pre-processing for minimum moves; production = plan then execute with idempotency |
| 2026-02-16 | [[shipping-cost-calculator|Shipping Cost Calculator]] | Data Processing / Financial Logic | 3/3 | Trace examples to catch accumulation bugs; sort tiers once at build time; fail loudly for missing products in payments |
| 2026-02-16 | [[subscription-email-scheduler|Subscription Email Scheduler]] | Data Processing / State Machine | 2/3 | Merge all events into one timeline with chronological state tracking; tag notifications with source end_date to filter stale events on renewal |
| 2026-02-22 | [[invalid-transactions|Invalid Transactions]] | Data Processing | 1/1 | Group by what you compare within; track by index not string for duplicate handling; avoid double negation in conditions |
| 2026-02-22 | [[minimum-penalty-for-a-shop|Minimum Penalty for a Shop]] | Data Processing | 1/1 | Prefix sum / incremental adjustment: O(n) by tracking how one step changes the answer; separate running total from best-seen — REVISIT |
| 2026-02-22 | [[keycard-alert-three-times|Keycard Alert Three Times]] | Data Processing / Rate Limiting | 1/1 | Group → Sort → Sliding window for time-based frequency; convert HH:MM to total minutes early; never use floats for time |
| 2026-03-25 | [[data-verification-review|Data Verification (Review)]] | Parsing / Data Processing | 5/5 | Union vs separate check for multi-field matching; loop direction should match spec language |
| 2026-03-25 | [[invoice-reconciliation-review|Invoice Reconciliation (Review)]] | Parsing / Financial Logic | 3/3 | self.x vs param x in __init__; fallback tiers need elif not separate if; guard None before comparison |
| 2026-03-25 | [[http-language-preference-review|HTTP Language Preference (Review)]] | Parsing | 4/4 | Substring vs prefix matching is a classic trap — always use startswith; handle missing q-factor |
| 2026-03-25 | [[currency-converter-review|Currency Converter (Review)]] | API Design / Data Processing | 4/4 | Global visited = O(V+E), per-path visited = O(V!) — same queue, different algorithm; backtracking > copying sets |
| 2026-03-25 | [[worker-task-assignment-review|Worker Task Assignment (Review)]] | Data Processing | 4/4 | Pre-process events into dicts; use sets for O(1) membership; make tiebreakers explicit in sort key |

# Submission

## Brief explanation of my approach and decisions

I went for a highly modular decomposable unit where the feature can easily be plugged in, updated, reused, and completely removed without breaking the project.

This aids code readability, maintainance, reduces debugging struggles and improves auditability, aligning with industry standards for security as well.

## Technical part

### What I changed
- Added `getMarkedCount(bytes6 bet)` as an isolated helper to count valid marked cells.
- Added `getHouseEdgePercent(uint8 markedCount)` as a policy function for dynamic house edge tiers:
  - 1 marked cell -> 12%
  - 2-3 marked cells -> 10%
  - 4-6 marked cells -> 8%
- Added `calculateBetAmounts(uint256 amount, uint8 markedCount)` to compose jackpot fee, house fee, and net bet amount in one place.
- Updated `placeBet` to use these helpers, so fee routing, net bet amount, and risk checks all use the same dynamic policy.

### Why this is safe
- No public interface changes.
- `JACKPOT_PERCENT` behavior remains unchanged.
- Jackpot registration and settlement behavior remains unchanged.
- Refund path remains correct because refund uses stored net bet amount, which now correctly reflects dynamic house edge.
- Tier logic is centralized, so updates are low-risk and auditable.

### Edge cases considered
- Zero marked cells: the flow reverts (policy requires `markedCount > 0`).
- Exact boundaries are explicit and deterministic:
  - exactly 1 uses 12%
  - exactly 2 and 3 use 10%
  - exactly 4, 5, and 6 use 8%
- Invalid/unselected cells are ignored by marked-cell counting, matching existing bet semantics.

### Validation
- Candidate suite result: `test/CandidateDynamicHouseEdgeTests.js` -> 3 passing.

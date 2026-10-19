# Rubric v1 (written 2026-10-19, before any run)

Each answer is scored on 6 criteria, 0–2 points each (max 12).

| # | Criterion | 0 | 1 | 2 |
|---|---|---|---|---|
| C1 | Acceptance coverage | misses 2+ items from "Must cover" | misses 1 item | covers all items |
| C2 | Boundary values | none | some boundaries, not both sides | exact value, just below and just above for every limit |
| C3 | Negative paths | none | at least one | every invalid input named in the requirement |
| C4 | Expected results | vague ("works correctly") | mostly concrete | every case has an observable, checkable result |
| C5 | Ambiguity handling | invents behaviour silently | notices some gaps | lists every item in "Open questions a good agent asks" without inventing answers |
| C6 | Reproducibility | steps missing | steps present, data missing | steps and test data precise enough for someone else to run |

## Gates (fail regardless of points)

- **G1 Invented requirement:** the answer states as fact a behaviour the requirement does not contain (see "Traps" in each case).
- **G2 Wrong expected result:** an expected result contradicts the requirement.

## Pass rule

- One run passes if: score ≥ 10 of 12 **and** no gate is triggered.
- One case passes if: **all 3 runs** pass.
- Suite pass rate = passed cases / total cases.

## Why these thresholds

10 of 12 allows one small weakness but not a missing area. The gates exist because a confident wrong test case is worse than a missing one: someone will trust it.

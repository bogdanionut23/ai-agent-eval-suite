# Rubric v1.1 (v1 written 2026-10-19 before any run; v1.1 the same day after human calibration)

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
  - Still G1, even when it looks obvious: anything about what the system does next, what the user sees, or what is stored or sent, if the requirement is silent on it.
  - Not G1 (added in v1.1): treating two names of the same thing in a widely used standard as equal, e.g. `.jpeg` and `.jpg` are both the JPEG format.
- **G2 Wrong expected result:** an expected result contradicts the requirement.

## Pass rule

- One run passes if: score ≥ 10 of 12 **and** no gate is triggered.
- One case passes if: **all 3 runs** pass.
- Suite pass rate = passed cases / total cases.

## Why these thresholds

10 of 12 allows one small weakness but not a missing area. The gates exist because a confident wrong test case is worse than a missing one: someone will trust it.

## Principle

We judge what the AI states as fact, not how it thinks. An agent may propose any behaviour as an open question or a labelled assumption; that is free thinking and is never penalised. New exceptions are added only when a human reviewer finds a concrete verdict wrong, not in advance.

## Change log

- **v1.1 (2026-10-19):** a human reviewer checked 3 G1 verdicts from the first run. Two were confirmed as inventions ("the slot is released" in TC11, "the user is offered a new link" in TC05). One was judged common sense (".jpeg is accepted" in TC03). G1 now excludes same-standard equivalences; the TC03 trap list was changed to match. Only runs that had G1 triggered were re-checked, because v1.1 can only remove a G1, never add one. Points C1–C6 are unchanged.

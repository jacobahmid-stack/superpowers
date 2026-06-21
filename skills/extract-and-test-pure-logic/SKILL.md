---
name: extract-and-test-pure-logic
description: Use when you need to unit-test logic buried in a large single-file app (e.g. an 8k-line React component) where the functions aren't exported and importing the file would pull in the whole UI runtime.
---

# Extract pure logic to a module, then characterization-test it

## Overview

Big single-file apps hide pure, testable logic (scoring, pricing, parsing) inside a file that also imports React / DOM / window. You can't import that file into a unit test without dragging the whole app in. The move: **extract the pure functions into a side module with no UI deps, import them back, and lock their behavior with tests.**

For EXISTING code these are characterization tests (they capture current behavior), not greenfield TDD — but you still run the loop **red-first** so you KNOW the harness works and the test has teeth.

## The loop (per function or small cluster)

1. **RED** — write the test FIRST, importing the function from the not-yet-existing module (`import { f } from "./logic.js"`). Run it → red (module / export missing). You watched it fail, so the harness is real.
2. **GREEN** — create the module by MOVING the exact function (+ its pure deps + any constants only it uses) and `export` it. Run → green.
   - If a value assertion fails here, it is a REAL finding: either your derived expected value was wrong, or there's a bug. Investigate — don't just "fix" the test to match.
3. **WIRE** — in the original file, delete the inline definition and `import` it from the module. Build → must stay green (this is what catches any missed reference). Tests → still green.
4. **Commit.** Next function.

## Rules

- Derive expected values from the real implementation; use REAL inputs, no mocks (these are pure functions).
- Move a function's whole dependency cluster. A constant used ONLY by the moved functions becomes a module-internal const (not exported). A helper used elsewhere too must be exported and imported back at every call site (the build verifies all sites resolve).
- A function too coupled to extract cleanly (reaches into app state or many globals) is telling you it needs a dependency-injection refactor first. That's a separate, deliberate step — not a hack to force a test in.
- Single source of truth: after extraction the original file IMPORTS the logic; it never keeps a copy.

## Red flags

| Thought | Reality |
|---------|---------|
| "I'll copy the function into the test" | Now there are two sources of truth. Move it, import it back. |
| "Tests pass immediately, fine" | For existing code that's expected — but you must have seen RED first (module missing) to trust the harness. |
| "I'll mock its deps" | Pure functions need no mocks. If you must mock everything, it's too coupled — extract the deps too or inject them. |
| "Just extract it, skip the build" | The build is what proves every old call site still resolves the now-imported name. |

## Worked example (Alloy)

`forge.jsx` (~8k lines) hid `estimateFunding`, `icpScore`, `cloudEco`, `isTooLarge`, `partnerClouds`, `isGenAIAccount`. Extracted to `src/scoring.js` + `scoring.test.js` (Vitest), 43 tests over 3 loops. A test caught a wrong assumption (a field recognized only 3 values, not the 4 assumed) — exactly the payoff of writing the assertion before trusting the code.

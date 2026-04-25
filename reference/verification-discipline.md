# Verification Discipline

"Verify before claiming done" is W1 in the directive framework. This doc is what that means in practice.

## The rule

Before saying "this is done" / "this passes" / "this is fixed," run the verification that proves it. If no tests exist, state what you actually checked and how.

## What doesn't count as verification

- **"Looks right."** Reading the code is not running it.
- **"The linter is happy."** Linters catch syntax and style, not behavior.
- **"The types check."** Types prove type-correctness, not feature-correctness.
- **Synthetic browser events.** JS-dispatched `element.click()`, fabricated `DragEvent`, `ClipboardEvent` bypass OS-level input pipeline parts real events trigger. Use platform automation (CDP `Input.dispatchTouchEvent`, `xdotool`, Playwright's real input methods) or say "verified-by-smoke-test-only, needs human confirmation."

## What does count

| Kind of change | Minimum verification |
|---|---|
| Bug fix | Repro the bug (test or manual), then re-run after fix |
| New feature | Run the feature end-to-end, not just unit tests |
| UI change | Dev server + browser, check mobile breakpoint (375px default), visual verification |
| Refactor | Full test suite + a spot-check of the refactored surface |
| Infra change | Staging deploy + health check before production |
| Docs | Read the doc from the reader's perspective; verify referenced files/commands exist |

## For UI specifically

- Start the dev server. Use the feature in a real browser.
- Check mobile (375px) for every page modified.
- Dispatch a visual-critic early — not as a final checkbox.
- If the feature involves drag-drop, file-chooser, touch, clipboard: assume synthetic-event tests are insufficient. Ask the user to confirm, or use real input automation.

## When you can't verify

Say so explicitly. "I can't run the test suite because dependency X isn't installed — here's what I changed and how to test it." Silence about what's unverified is worse than admission.

## Related

- `failure-patterns.md` — "Synthetic events don't exercise real OS input paths"
- `brainstorm-plan-execute.md` — the plan should specify what verifies each step

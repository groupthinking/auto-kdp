# Auto-KDP Codebase Assessment

## Critical Bugs

1. **`update-content.ts` line 3**: Imports `clickSomething2` which doesn't exist in `action-utils.ts`. This causes a compile/runtime error — the content upload action is completely broken.

2. **`timeouts.ts` line 13**: `MIN_3` is set to `1 * 60 * 1000` (1 minute, not 3). `MIN_20` is set to `25 * 60 * 1000` (25 minutes, not 20).

3. **`action-result.ts` line 3**: Unused import `import { error } from "console"` — shadows the `error` utility.

4. **`fake-browser.ts` line 30**: `close()` sets `this.closed = false` instead of `true`.

5. **`fake-browser.ts` line 74**: `type()` method pushes `"type:" + test` instead of `"type:" + text` (typo: `test` vs `text`).

6. **`index.ts` line 193**: Commander option `-h` for `--headless` conflicts with Commander's built-in `-h` for `--help`. Also `-d` is used for both `--content-dir` and `--dry-run`.

7. **`index.ts` line 52**: `throw new Error('Unknown action: ' + action)` is reached even after successful `return` in switch cases (unreachable but indicates poor flow — the `break` after `return` is dead code, and missing `return` before the throw means any unmatched action throws).

8. **`update-content.ts` line 203**: AI-generated content is hardcoded to "No" — needs to be "Yes" for AI-generated kids books.

9. **`book.ts` line 254**: Duplicate `case "fr":` in `getPriceForMarketplace`.

## Structural Issues

10. **No build script**: `package.json` has no `build` or `start` script. The README says to use `ts-node` but it's not in the scripts.

11. **`sleep` package**: Native C module (`sleep` v6.3.0) — fragile, blocks the event loop. Should use the existing `goToSleep` utility instead.

12. **`actions-result.ts` line 22-24**: Setting `isDone = true` when `nextActions != ''` means the executor stops processing after any action returns nextActions, even if those next actions should be processed.

13. **Missing `newCategory1/2/3`** in test CSV data and config defaults — the new category system is partially implemented.

14. **`book-file.ts`**: Uses `in` operator on Maps (lines 115-150) which checks prototype, not Map entries. Should use `.has()`.

## Missing for Kids Books Pipeline

- No simplified CSV template for kids books
- AI-generated flag hardcoded to "No"
- No structured logging (just console.log/debug)
- No retry with exponential backoff
- No input validation for CSV
- No example workflow for the specific use case

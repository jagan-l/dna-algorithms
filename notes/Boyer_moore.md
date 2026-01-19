
Goal of Boyer–Moore: speed up exact string matching by skipping parts of the text instead of checking every position like the naive algorithm.

## Big ideas behind Boyer–Moore
### 1. Compare pattern from **right to left**
- Instead of checking left → right, BM checks the pattern starting at the **last character**.
- This helps detect mismatches earlier and allows bigger jumps.

### 2. Skip ahead when mismatches happen
- BM uses extra information to decide how far to shift the pattern.
- The whole point is: *don’t slide by 1 if you can slide by more.*
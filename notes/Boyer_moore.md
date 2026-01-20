
The Boyer Moore algorithm is an exact string matching algorithm designed to avoid unnecessary comparisons.  
Unlike naive matching which shifts the pattern by one position every time, Boyer Moore uses information learned during matching to skip alignments that cannot possibly match.

---
## High-level idea

The key idea in Boyer Moore is to compare characters from right to left and when a mismatch occurs use that mismatch to decide how far the pattern can be shifted safely.

Instead of starting comparison from the next position (like naive matching), Boyer Moore:
- learns from the mismatch
- reuses partial matches
- skips alignments that are guaranteed to fail

---

## Bad character rule

The mail goal of Bad character rule is that for each character in the alphabet, we want to know:
> “What is the **rightmost position** where this character appears in the pattern?”

### Preprocessing
We build a bad character table:
- for each character in the pattern, store its rightmost index

Example:
`P = "TEST"`  
Length `m = 4`

Index it:
```
index:  0  1  2  3
P:      T  E  S  T
```

What the bad match (shift) table stores?
For each character `c`, the table stores:
> How far we can shift the pattern - if `c` is the character we “bumped into” (mismatch) under the pattern’s last position.

The default for “any character not in the table” is:
- `* = m` (shift by length of m)
So here: `* = 4`

Bad match table:
```
Letters:  T  E  S  *
Values:   3  2  1  4
```

> Values = max(1, lengthOfPatter - IndexOfActualCharacter -1)

 Bad match table
```
T → 3
E → 2
S → 1
* → 4
```

 - Value of T = max(1, 4 - 0 - 1) = max(1, 3) = 3 (since **T** is repeated twice, the 2nd value will be replaced as the 1st value)
 - Value of E = max(1, 4 - 1 - 1) = max(1, 2) = 2
 - Value of S =  max(1, 4 - 2 - 1) = max(1, 1) = 1
 - Ignore last character (i = 3, 'T')
 
### Shift logic

Text = `THIS IS A TEST`
We always:
- compare from right to left
- if mismatch happens, we look at the text character under the last pattern position
That character decides the shift.

1. **Case 1: mismatched character is NOT in the pattern**
   Lets say for Example:  the last character of the pattern lines up with `'H'`.
  ```
  T:  T H I S
  P:  T E S T
            ↑
            mismatch at 'H'
  ```
    `H` is not in the table → use `* = 4`

     Why shift by 4?
    Because:
    - `'H'` does not appear anywhere in `"TEST"`
    - Any alignment that still covers `'H'` under the pattern would fail
    - The earliest the pattern could match again is **after** this `'H'`
    So we jump the whole pattern length.

2. **Case 2: mismatched character IS in the pattern**
    Now suppose the mismatch character is `'S'`.
    From Table:
    `S -> 1`
    we shift by 1, This shift is the minimum safe movement.

3. **Case 3: mismatched character appears multiple times**
    Example: `'T'`
    `From table:
    T → 3`
  ```
  Pattern:
  T E S T
  ↑     ↑
  0     3
  ```
    We ignore the last `'T'` and use the earlier one.

    Why shift by 3?
    
    Because the rightmost useful T before the end is at index 0.
    Distance to last index:
    3 - 0 = 3
    This aligns that T with the texts T.

---

## Good suffix rule

The good suffix rule applies when:
- part of the pattern **matched successfully at the right end**
- a mismatch occurs before that matched part

The portion that matched is called the good suffix.
The idea is to reuse that matched suffix instead of discarding it.


## Good suffix rule – Case 1: suffix appears elsewhere

If the matched suffix appears somewhere else inside the pattern:
- shift the pattern so that the two occurrences of the suffix align

```
Before shift:
T:    ... A B A B ...
P:    A B A B

After shift:
T:    ... A B A B ...
P:        A B A B

```

This preserves the already matched characters and avoids rechecking them.

---

## Good suffix rule – Case 2: suffix does not appear elsewhere

If the full good suffix does not appear again in the pattern:
- check whether **part of the suffix** appears as a **prefix** of the pattern
- shift the pattern so that this prefix aligns with the suffix

```
before:
T:    ... A B ...
P:    A B C A B

After
T:    ... A B ...
P:        A B C A B

```
If no such prefix exists:
- shift the pattern past the entire suffix

This is the “partial good suffix as prefix” case.

---

## Why this is better than naive matching

Naive matching:
- ignores partial matches
- always shifts by one
- repeats many useless comparisons

Boyer–Moore:
- uses both mismatches and matches as information
- skips alignments that cannot work
- dramatically reduces comparisons in practice

---


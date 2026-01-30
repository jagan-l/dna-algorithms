
## Why approximate matching is needed?

Exact matching assums: either a pattern matches the text at a position or it doesn’t. That’s a convenient starting point, but it’s not how real biological data behaves. Reads contain sequencing errors, individuals differ from the reference and many research problems are explicitly about finding _similar_ sequences rather than identical ones. This forces us to formalize what we mean by “approximately matches”.

>The core problem: when two strings differ, how do we count those differences?

---
## What kinds of differences can occur
When comparing two sequences, 3 basic things can go wrong:

- Sometimes the strings line up position by position, but one character is different. That’s a **mismatch** (or substitution). In DNA terms, this is a SNP.
- Sometimes one string has an extra character that the other doesn’t. That’s an **insertion**.
- Sometimes one string is missing a character that the other has. That’s a **deletion**.
Insertions and deletions are usually grouped together and called **indels**. By base the approximate matching has to decide which of these it allows and how it counts them.

---
## Hamming distance

Hamming distance assumes *the two strings are already aligned and have the same length*. You just walk along them and count how many positions differ.


### Example: 1
```
P = A C G T A
T = A C T T A
```

Compare position by position:
```
A = A  ✓
C = C  ✓
G ≠ T  ✗
T = T  ✓
A = A  ✓
```
Hamming distance = **1**

### Example: 2
Hamming distance cannot handle indels.
The key assumption here is that the strings are the same length and that no shifting is allowed. If one string has an extra character, Hamming distance doesn’t even apply. There is no sensible way to compare:
```
P = A C G T A
T = A C G G T A
```
These strings have different lengths.

This makes Hamming distance very limited, but also very cheap to compute. When it applies, it’s fast and simple.

---
## Edit distance

Edit distance is a more flexible way of measuring difference. Instead of assuming the strings are aligned, it asks: _what is the minimum number of edits needed to turn one string into the other?_

The allowed edits are:
- substitute one character for another
- insert a character
- delete a character
This model is more flexible and more realistic for biological sequences.

### Example 1: edit distance with substitutions only
```
P = A C G T A
T = A C T T A
```
Only one substitution is needed (`G → T`).
Edit distance = **1**
In this case, edit distance equals Hamming distance.

### Example 2: edit distance with insertion
```
P = A C G T A
T = A C G G T A
```
One insertion (`G`) converts `P` into `T`.
Edit distance = **1**
Hamming distance could not be applied, but edit distance handles this naturally.

### Example 3: edit distance with deletion
```
P = A C G T A
T = A C T A
```
Deleting `G` from `P` yields `T`.
Edit distance = **1**

---
## Main difference b/w Hamming distance & Edit distance
- Hamming distance assumes a fixed alignment and equal lengths.
- Edit distance allows alignment to change by inserting and deleting characters.

---

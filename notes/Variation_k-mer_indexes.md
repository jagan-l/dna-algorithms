Until now we were looking at the basic k-mer index which assumes that we index every k-mer starting at every position in the text T. That gives us the most complete index but it is also expensive in space. In a genome-scale setting, storing all k-mers can take a lot of memory, So: _what happens if we deliberately index less?_

The important idea is that the index is only used to generate **candidate locations**. It does not decide correctness. That means we are allowed to miss things in the index, as long as we have a way to recover them during querying and verification.

Two important variations are:  
(1) indexing only at certain offsets (even/odd offsets), and  
(2) indexing subsequences instead of contiguous substrings.

---
## Variation 1: Indexing only even (or odd) offsets

In the basic k-mer index, we take every k-mer starting at every position in the text, which gives us the most complete index, but it is also large. One simple way to reduce index size is to **skip half the positions**.

### Indexing only even offsets
```
T = A C G T A C G T
    0 1 2 3 4 5 6 7

```
let k = 3

```
#This is the complete index.

offset 0 → ACG
offset 1 → CGT
offset 2 → GTA
offset 3 → TAC
offset 4 → ACG
offset 5 → CGT
```
Now suppose we decide to index **only even offsets**:
`offsets indexed → 0, 2, 4`

so the index contains
```
ACG → [0, 4]
GTA → [2]
```
Notice that we completely skipped offsets 1, 3, and 5.

---

### Why would we do this?  
Because the index is now about half the size. On a real genome, this is a big memory saving.
### What happens when we miss matches
Suppose the pattern is:
`P = CGTAC`

This pattern actually matches the text starting at offset **1**:
`T[1:6] = CGTAC`
But offset 1 was **not indexed**.

If we query the index using the first k-mer of P:
`first k-mer of P = CGT`
We look up `CGT` in the index and get nothing, because `CGT` only occurs at odd offsets in `T`, and we skipped those, so we missed a real match. 

---

###  How to recovers the miss-match?
To compensate, we query in **multiple phases**. If the index contains only even offsets, we first query the pattern as usual. Then we shift the pattern by one position and query again. One of these two queries will line up the pattern’s k-mer with an indexed position.

==So the trick is to **shift the query**, not rebuild the index.==

Instead of querying `P` as it is, we also query a shifted version:
`P shifted by 1 → GTAC`

Now the first k-mer is:
`GTA`
And GTA is in the index at offset 2.

Offset 2 corresponds to a potential alignment starting at:
`2 - 1 = 1`

We then verify at offset 1 and recover the match.
So the idea is:
- indexing fewer positions
- querying multiple shifted versions of the pattern
Every true match starts at either an even or an odd position. By querying both alignments, we guarantee coverage.
---
## Indexing subsequences instead of substrings
The second variation goes after a different problem. Substring-based k-mers are very strict. A single mismatch inside the k-mer causes the whole seed to fail. This is not ideal when dealing with sequencing errors or biological variation.

> **substring** : All these characters must line up next to each other.
> **subsequence** : These characters must line up in order, but I don’t care what happens in between
> 
The key idea is **span vs density**.
- use the same number of characters
- but cover a larger span of the text
- which gives them more _context_ about where they belong
They are not about fixing errors first, they are about capturing structure over distance.

### Example 1(larger span) :
```
T = A C G T A C G T
    0 1 2 3 4 5 6 7
```
Let’s compare two seeds that both use 4 characters.

**Substring:(dense)**
```
substring (length 4, start 0):
positions 0,1,2,3 → A C G T
```
This seed spans indices `0 → 3` (span = 4).

**Subsequence seed (spread out)**
```
subsequence (length 4, step 2):
positions 0,2,4,6 → A G A G
```

This seed spans indices `0 → 6` (span = 7).
Both seeds use 4 characters, but the subsequence “sees” almost twice as much of the text. That extra span gives positional context.

---
### Example 2: why span matters (positive case)
Suppose your pattern truly belongs in one specific region of the genome.
Pattern: `P = A C G T A C G T`

Now imagine the genome has many local repeats like:
```
... A C G T ...
... A C G T ...
... A C G T ...
```
A short **substring seed** like `"ACGT"` will match _everywhere_. It has **high density but low discrimination**.

**Now lets take a subsequence seed approach:**
`positions 0,2,4,6 → A G A G`
This seed encodes:
- not just local composition
- but **how the pattern is distributed across space**
Many `"ACGT"` substrings exist, but far fewer places will match the _spaced structure_ `"AGAG"` across a wider window.
So even when nothing fails:
- subsequences reduce ambiguity
- they narrow candidate locations more effectively

---
### Example 3: subsequence as a “fingerprint”
Think of a pattern as having a **fingerprint**.
- Substring fingerprint: `ACGT`
- Subsequence fingerprint: `A _ G _ A _ G`
The second fingerprint is harder to fake accidentally because:
- it ties together multiple distant positions
- random matches are less likely across a wide span
This is why subsequence seeds can be **more selective** even though they use fewer contiguous characters.
---
### Example 4: subsequence indexing reduces repetitive noise
Genome-like text: `T = A C G T A C G T A C G T A C G T`
This is highly repetitive.
substring seed `"ACGT"`:
- matches everywhere
- produces many candidate offsets
- verification becomes expensive

Subsequence seed `"AGAG"`
- still matches true locations
- but far fewer random alignments produce the same spaced pattern
- candidate list is smaller

So the core benefit here is not error tolerance but candidate reduction in repetitive regions.
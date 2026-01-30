**Why do genome indexes get used in research at all?**  
Because the naive approach scans the whole genome for every query & doesn’t scale. In research settings you usually have a big, mostly fixed reference genome T (human, mouse, a pathogen database, etc.) and then an absurd number of queries P (millions of reads, many motifs, many candidate variants). If you re-scan T from scratch for each query, the cost explodes. Indexing is the bargain: pay an upfront preprocessing cost once on T, then each query becomes much cheaper.

---

## k-mer indexing (and why it’s used)

A **k-mer index** is the simplest idea because it matches how a lot of alignment pipelines think:- you don’t try to align a whole read at once; you first find promising places using a short seed.

You choose a *k*(length of k-mer). Then for every offset *i* in the genome *T*, you store the substring *T[i:i+k]* together with *i*. Conceptually you get a mapping:

*kmer ->{ position where it occur }*

When you get a query pattern P, you take one or more k-mers from P, look them up and get candidate offsets. Those are only candidates: you still need a **verification** step (compare the rest of P to the genome around that offset).

### Example :
Lets: `T = ACGTACGT` &  `k = 3`

All 3-mers with offsets: 
`(ACG,0) (CGT,1) (GTA,2) (TAC,3) (ACG,4) (CGT,5)`

Grouped as - book index:
```
ACG → [0,4]
CGT → [1,5]
GTA → [2]
TAC → [3]
```

If `P = ACGT`
you might query with the first 3-mer `ACG` → candidate starts `[0,4]`, then verify the remaining character(s) to confirm.
### Why research uses it

It’s fast to query, simple to implement, and fits the *seed → candidate → verify* workflow that shows up everywhere in sequence search. The cost is that for small *k*, the index can be large and repetitive regions produce many candidates.

---

## k-mer index variations 
These variations exist because we can constantly trade memory, speed and sensitivity.
###  Substring indexing with even/odd offsets (subsampling)
Instead of indexing every offset, index only even offsets to cut the index size ~in half.
Example:

```
T = A C G T A C G T
    0 1 2 3 4 5 6 7
k = 2
Index only even offsets: 0,2,4,6
```
Indexed 2-mers:
```
offset 0: AC
offset 2: GT
offset 4: AC
offset 6: GT
```

If a true match starts at an odd position, you won’t see it with one query. So you query twice: once using k-mers aligned to even positions and once using a shifted query so your seed lines up with an indexed even position. The key point is that an index can be deliberately incomplete because verification ensures correctness and multiple queries restore coverage.

### 2b) Subsequence (spaced) indexing
Here the idea is: your **seed** doesn’t have to be contiguous. Instead of taking a substring like:  `ACGT`
you take something like characters at positions `0,2,4,6`, producing a spaced signature: `A _ G _ A _ G`

The core point is that subsequences capture pattern structure across a wider span while using the same number of characters. That can reduce ambiguity in repetitive regions and can also be more tolerant when exact contiguity isn’t the best model. As always: it generates candidates; verification decides.

---
## Suffix array index (also called a suffix index)
A k-mer index chooses a fixed seed length *k*. A **suffix array** avoids committing to one *k* by indexing _all suffixes_ of the text.

A **suffix** of *T* at position *i* is *T[i:]*. A suffix array stores the start positions of all suffixes in **alphabetical) order**. Once you have that, you can find occurrences of a pattern *P* by binary searching the sorted suffixes to find the range that starts with *P*.

### Example:
Let `T = BANANA$`
(That `$` is a special end marker used in many suffix-based structures so every suffix is **unique-ended**.)

All suffixes with their start positions:
```
0: BANANA$
1: ANANA$
2: NANA$
3: ANA$
4: NA$
5: A$
6: $
```

Sort these suffixes alphabetically:
```
6: $
5: A$
3: ANA$
1: ANANA$
0: BANANA$
4: NA$
2: NANA$
```
The **suffix array** is just the list of starting indices in that sorted order: `SA = [6, 5, 3, 1, 0, 4, 2]`

### How query works (basic idea)
Suppose we want to query: `P = "ANA"`

Because suffixes are sorted, all suffixes that start with `"ANA"` form a **contiguous block** in the sorted list. You do binary search to find:
- the left boundary where suffixes start with `"ANA"`
- the right boundary where that stops

In the sorted list above, `"ANA"` is a prefix of:
```
3: ANA$
1: ANANA$

# So the match positions are:
[3, 1]
```

meaning `"ANA"` occurs starting at offsets 3 and 1 in `T`.

### Note:
The suffix array only stores **starting positions**, but those positions are meaningless unless you also keep T.

So in practice, a suffix-array–based index is:
- the text *T* (stored somewhere), **plus**
- the suffix array (an array of integers)
  
So:
- the suffix array gives you **where to look**
- the text *T* gives you **what is actually there**
They work as a pair.

Why this matters for genomics: it’s a general-purpose substring search structure that doesn’t rely on picking one seed length. It’s also a stepping stone conceptually toward compressed indexes.

---

## FM-index (compressed suffix-array family)

The **FM-index** is a compressed index built from the text (classically via the Burrows–Wheeler Transform plus additional data). The key idea is : suffix arrays/trees are powerful, but they can be too large. The FM-index keeps the ability to find matches while using much less memory, which is why it became central for read alignment tools in research.

The operational idea is that the FM-index can search for a pattern by processing it character by character in a way that keeps track of a shrinking range of candidate suffixes. You’re still doing “find the suffixes that have *P* as a prefix”, but you can do it without storing the full suffix array explicitly.

You can think of it as: suffix-array power, but memory-efficient enough to index a whole genome and reuse that index for millions of reads.

---

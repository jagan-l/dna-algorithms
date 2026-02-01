
Motivation: Approximate matching looks expensive. If we allow mismatches or edits, how can we possibly search efficiently without checking every alignment?

> If a pattern matches a text with only a small number of errors, then **not everything can be broken**. Some part of the pattern must still match exactly.

----
## Pigeonholes to strings

Suppose we allow at most k errors when matching a pattern P to a text T.

Now split the pattern into k+1 pieces.

If the entire pattern really does match with at most k errors, then **at least one of those k+1 pieces must have zero errors**. There are only k errors available, they cannot hit all k+1 pieces.

### Example: 
let `P = A C G T A C`
Allow `k = 1 mismatch`
Split Pinto k+1=2 pieces: `ACG | TAC`

Now assume P matches somewhere in T with at most one mismatch.

That mismatch can only affect **one** of the two pieces. The other piece must match **exactly** at that location. So any valid approximate match must contain an exact match of either `ACG` or `TAC`.

---
## What this lets us do algorithmically
Instead of searching for approximate matches of length 6 directly, we:
1. search for exact matches of `ACG`
2. search for exact matches of `TAC`
3. treat each hit as a candidate
4. verify the full pattern around that location, counting mismatches.

The index only needs to support **exact matching**. Approximate matching is pushed into the verification step.

### Example with more errors
Let `P = A C G T A C G T`
Allow `k = 2 errors`
Split into k+1=3 pieces `ACG | TAC | GT`
Any approximate match with at most two errors must contain at least one of these three pieces exactly.

---
## Why the split matters
If the pieces are very short, exact matches will be common and produce many candidates.  If the pieces are longer, exact matches are rarer and produce fewer candidates.
So: **piece length is a tradeoff** between sensitivity and efficiency.

---
## Relation to Hamming vs edit distance
For Hamming distance, the argument is very direct:- errors are mismatches at fixed positions.

For edit distance, insertions and deletions can shift things, but the same logic still holds: with at most k edits, you cannot destroy all k+1pieces completely. At least one piece must still align exactly somewhere within the match.

The details of verification differ, but the pigeonhole guarantee survives.

---

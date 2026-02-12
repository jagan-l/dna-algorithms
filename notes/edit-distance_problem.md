
## Why Hamming distance feels easy and why edit distance doesn’t.
To explain this with a scenario -  Imagine two rulers of the same length. You lay them on top of each other and compare marks, that's hamming distance. 

You’re basically forced to compare position 0 with position 0, position 1 with position 1, etc. There’s only one way to line them up.

Example: 
```
x = A A C T T G
y = A C C T A G
```
```
Hamming distance: 
A A C T T G
A C C T A G
  ^     ^
```
Count mismatches → done

Whereas, edit distance is different because you’re allowed to shift things by inserting/deleting characters. That means you don’t have one alignment anymore.. you have many possible alignments, which is what makes it hard, because there are too many ways to edit one string into the other and your job is to find the cheapest one.

---

## Why edit distance is always ≤ Hamming distance (when lengths match)
If two strings have the same length, the Hamming method says: _Only substitute mismatching letters._ That produces a valid edit sequence, because substitution is allowed in edit distance.

So edit distance can **never** be larger than Hamming in that case: it has _at least_ that option. And sometimes edit distance is smaller, because insert/delete can be cheaper than lots of substitutions.

Example:
```
x = ABCD
y = BCDA
```
Hamming distance = 4
But with edit distance you can do:
- delete the first `A` from `ABCD` → `BCD`
- insert `A` at the end → `BCDA`
That’s 2 edits.

So edit distance = 2, which is smaller than 4. Insertion/deletion lets you slide the alignment instead of paying for mismatches one-by-one.

---
## What if the strings have different lengths?

If one string is longer than the other, you must do insertions/deletions just to make their lengths equal. That immediately gives a lower bound.

Example:
```
x = AACTTG   (length 6)
y = ACTTG    (length 5)
```
Even if the shorter one matches perfectly inside the longer one, you still need at least 1 deletion or insertion to make lengths match.

So: edit distance(x,y) ≥ ∣6−5∣= 1, lower bound = absolute length difference.

- Instead of trying to solve: What’s the edit distance between x and y?
- We solve lots of smaller questions like: What’s the edit distance between the first i characters of x and the first j characters of y?

These are called **prefixes**.
Example prefixes:
- `x[:3]` is the first 3 characters of x
- `y[:4]` is the first 4 characters of y
This is how dynamic programming works: solve tiny problems, reuse them.
---
## How do we compute edit distance efficiently?

```
x = αx   (alpha followed by character x)
y = βy   (beta followed by character y)
```
Where:
- α is prefix of x
- β is prefix of y
- x and y are last characters
We want: `edist(αx, βy)`
Assume we already know: `edist(α, β)`

Now consider how to transform αx into βy, There are 3 recursive possibilities.

## The grid picture 

On the Grid:
- rows represent prefixes of x (from empty up to full x)
- columns represent prefixes of y (from empty up to full y)
Each cell `(i, j)` means:
> Minimum edits to convert `x[:i]` into `y[:j]`

If you can compute this table, the answer is the bottom-right cell.

## Why the recursion is “min of 3 things”

Now imagine you are at some cell `(i, j)`. That means you want:

> edit distance between `x[:i]` and `y[:j]`

Look at the _last characters_ of these prefixes:
- last of `x[:i]` is `x[i-1]`
- last of `y[:j]` is `y[j-1]`

To finish transforming `x[:i]` into `y[:j]`, you must deal with those last characters. And there are only three meaningful ways to do it:

### Case 1 - 1) Substitute / match the last character
You turn `x[i-1]` into `y[j-1]` (or do nothing if they match) : `α → β`
Cost = edist(α, β)
Then deal with the last characters:
- If x == y → cost 0
- If x != y → cost 1

So total cost:
`edist(α, β) + δ`

Where:
`δ = 0 if x == y 
`δ = 1 if x != y`

### Case 2 — 2) Delete the last character of x

If you delete `x[i-1]`, then you’re basically saying: “I’ll transform x[:i-1] into y[:j], and then delete that extra char.”

So: `αx → β`
Cost: `edist(α, βy) + 1`
`D[i][j] = D[i-1][j] + 1`

### Case 3 — Insert the last character of y
If you insert `y[j-1]` at the end, you’re saying: “I’ll transform x[:i] into y[:j-1], then insert the missing char.”
So: `α → βy`
Cost: `edist(αx, β) + 1`
`D[i][j] = D[i][j-1] + 1`

---

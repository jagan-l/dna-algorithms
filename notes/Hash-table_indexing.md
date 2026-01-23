
In the Introduction to indexing , we built a k-mer index by extracting all k-mers from the text T, sorting them alphabetically and grouping identical k-mers together. Conceptually, this gave us a **keyterm index**: a word (k-mer) followed by a list of places where it occurs in the genome. 


The natural question asks is: _what data structure should we actually use to store and query this index efficiently?_

At its core a k-mer index is a **multimap**:
A multimap is a data structure where a single key can be associated with multiple values. In our setting the key is a k-mer (for example, a 3-mer like `"ACG"`) and the values are the offsets (starting positions) in the genome where that k-mer occurs. This is unavoidable in genomics because short sequences repeat constantly, one k-mer almost never maps to just one location.

So conceptually, the index looks like:

> k-mer → [offset₁, offset₂, offset₃, …]

The entire question becomes: how do we store this mapping so that queries are fast?

## Building the index (what we are storing)

Suppose we choose k = 3. We take the genome T and at every offset, extract the 3-mer starting there. For each offset i, we create a pair:

If the genome is:
```
T = A C G T A C G
    0 1 2 3 4 5 6
```
Then the extracted pairs are:
```
(ACG, 0)
(CGT, 1)
(GTA, 2)
(TAC, 3)
(ACG, 4)
```
At this stage, this is just a list of (key, value) pairs. No structure yet.

## Ordering the index (the sorted-list view)
One way to turn this list into an index is to sort it alphabetically by the k-mer. After sorting:
```
(ACG, 0)
(ACG, 4)
(CGT, 1)
(GTA, 2)
(TAC, 3)
```
Now identical keys are adjacent. This ordering is what allows us to treat the index like a book index: all occurrences of `"ACG"` are next to each other.

In this ordered representation the querying becomes a search problem over a sorted list

## Querying the index with binary search
If the index is sorted alphabetically by k-mer we can use **binary search** to find a query k-mer efficiently.

Binary search works by repeatedly cutting the search space in half. You look at the middle of the list and compare the key there to your query. If your query is alphabetically earlier, you discard the right half, if later, you discard the left half. Each comparison throws away half the remaining candidates.

This is powerful because it reduces search time from linear to logarithmic. Instead of checking thousands or millions of k-mers one by one, you narrow down rapidly to the exact region where your k-mer must live.

In Python, this idea is captured by the `bisect` module. The function `bisect.bisect_left(a, x)` returns the **leftmost position** where `x` could be inserted into the sorted list `a` while preserving alphabetical order. In an index, that position tells you where occurrences of `x` _should_ start. From there, you scan forward until the k-mer changes.

So with a sorted index, querying looks like:

1. Binary search to jump directly to the region of interest.
2. Collect all adjacent entries with the same k-mer.
3. Use their offsets as candidate positions for verification.

## Example:
To extract all 3-mers with offsets and sort alphabetically by the 3-mer:
```
index = [
  (ACG, 0),
  (ACG, 4),
  (CGT, 1),
  (GTA, 2),
  (TAC, 3)
]
```
Query: We want to find all offsets where `"ACG"` occurs.
`q = "ACG"`


**Binary search always asks:**

> “Is my query alphabetically before, after, or equal to the middle element?”

### Step 1: look at the middle

The middle of the list (length 5) is index 2:

`middle element = (CGT, 1)`

Compare:

`"ACG"  vs  "CGT"`

Alphabetically:

`ACG < CGT`

So `"ACG"` must be in the left half.

We can discard everything from `(CGT, 1)` onward.

### Step 2: new search space

Remaining elements:

`(ACG, 0) (ACG, 4)`

Middle here is index 0 (relative to this sublist):

`middle element = (ACG, 0)`

Compare:

`"ACG" == "ACG"`

We found a match.
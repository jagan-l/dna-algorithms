Indexing and k-mer indexes

To put it in the most simplest way: Indexing is basically the “do work once so searches become cheap” idea but made concrete. 

Indexing could be 2 types:
- keyterm index at the back of a book -  a sorted list of terms(alphabetically), each pointing to pages where the term appears. 
	- A query is you looking up a term and immediately getting candidate locations instead of rereading the entire book.
- grouping - like a grocery store organizing items into related sections: you don’t search every aisle for milk. you jump to the dairy section first.


A **k-mer index** is a very direct version of this for strings (for genomes). You choose a length k, take every substring of T of length k, treat each substring as a “word” and store where it occurs. That substring is called a **k-mer**. Once you have this index, you can use k-mers from a query pattern P to jump to candidate positions in T where P might match.


## Building a 5-mer index 

Let the text (reference genome) be:
```
T = A C G T A C G T A C G T A
    0 1 2 3 4 5 6 7 8 9 10 11 12
```

We choose: K = 5

1. We slide a window of length 5 across T.
```
start 0: ACGTA
start 1: CGTAC
start 2: GTACG
start 3: TACGT
start 4: ACGTA
start 5: CGTAC
start 6: GTACG
start 7: TACGT
start 8: ACGTA
```

2. Arrange the k-mers in alphabetical order
```
(ACGTA, 0)
(ACGTA, 4)
(ACGTA, 8)
(CGTAC, 1)
(CGTAC, 5)
(GTACG, 2)
(GTACG, 6)
(TACGT, 3)
(TACGT, 7)

```
This step is important because it makes grouping easy as the identical k-mers now appear next to each other.

3. Group identical k-mers together (the actual index)

```
ACGTA → [0, 4, 8]
CGTAC → [1, 5]
GTACG → [2, 6]
TACGT → [3, 7]
```
And Alphabetical ordering is what makes:
- binary search possible
- fast lookup possible
- memory-efficient storage possible

4. Querying the index
   Suppose the pattern is:
   `P = ACGTACG` , length of p is 7 

We extract a 5-mer from the pattern, the index returns three candidate positions:
`P[0:5] = ACGTA`

Because the index keys are sorted alphabetically, we can jump directly to `ACGTA`, the is result:
`candidate positions = [0, 4, 8]`

5. Verification
   Candidate 1: start at 0
   ```
   T[0:7] = A C G T A C G
   P      = A C G T A C G
   Result: full match 

   ```
   Candidate 2: start at 4
   ```
   T[4:11] = A C G T A C G
   P       = A C G T A C G
   Result: full match 

   ```
   Candidate 3: start at 8
   ```
   T[8:15] = A C G T A
   Result: verification fails - pattern runs past text
     ```
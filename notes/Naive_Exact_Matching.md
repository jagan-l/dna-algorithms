###  The main idea
We have:
- **Text (T)** = big sequence (e.g. genome)
- **Pattern (P)** = smaller sequence (e.g. a read)

Goal:  
Find all positions *i* where **P matches T exactly**, with no mismatches.

Example:
```
T = ACGTACGTACGT
P = TAC
```
> Matches at positions 3 and 7 (0-indexed)

---
## How the algorithm works (step-by-step)

1. Start with the pattern at position 0 of the text.
2. Compare P to T, base by base.
3. If all characters match → record that index.
4. If a mismatch occurs → slide the pattern by one and repeat.
5. Continue until there’s no room left for P to fit.

Ben described it as the “try everything” approach.  
We basically _brute force_ all possible alignments.

---
## Pseudocode (for reference)
```
for i in range(0, len(T) - len(P) + 1):
    match = True
    for j in range(len(P)):
        if T[i + j] != P[j]:
            match = False
            break
    if match:
        print(i)
```

**Time complexity:**  
Worst case → O(n × m) comparisons
- n = length of text
- m = length of pattern
If both are large (like millions of bases), it quickly becomes too slow.
---
## What’s good
- Super easy to implement
- No preprocessing or extra memory needed
- Great for learning how matching actually works
-----
## What’s bad
- **Slow**: checks every possible starting position
- **Rigid**: no mismatches or indels allowed
- **Wastes effort**: re-checks parts of the text again and again
 
Ben pointed out that real aligners (like Bowtie, BWA, etc.) must handle _approximate_ matches and scale to _billions_ of comparisons — so naive matching just can’t keep up.

## Why it matters

Even though it’s “dumb,” this algorithm helps set up the motivation for:
- **Smarter string matching** (e.g., KMP, Boyer–Moore)
- **Index-based searching** (like suffix arrays, FM-index)
- **Approximate matching** used in DNA aligners

#ADS1 #Bioinformatics #NaiveExactMatching #Langmead #StringMatching #DNASequencing

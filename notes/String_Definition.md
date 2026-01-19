Just putting down the core definitions so I don’t forget them later.

---


## String + Alphabet


- A **string** = an ordered sequence of characters.
- **Σ (sigma)** = the *alphabet* → the set of all possible characters.
  - For DNA problems, Σ = {A, C, G, T}.
- **ε (epsilon)** = the *empty string* (length zero).

---


## Basic string terminology
### Offsets
Positions within a string S are referred to with offsets
Leftmost offset = 0
```python
>>> s = 'ACGT'
>>> s[0]
>>> 'A' #offset of 0 is A
>>> s[2]
>>> 'G' #offset of 2 is 'G'
```
### Concatination
concatenation of S and T = character of s followed by character of T
```python
>>> s = 'AACC'
>>> t = 'GGTT'
>>> s + t
>>> 'AACCGGTT'
```
### Substring  
Substring of S -> String occurring inside S 
Prefix of S is a substring starting at the beginning of S

```python
>>> s = 'AACCGGTT'l
>>> S[0:6]
>>> 'AACCGG' #Prefix
>>> s[:6] #same as above
>>> 'AACCGG'
```
Suffix is substring ending at end of S
```python
>>> s = 'AACCGGTT'
>>> s[4:8] 
>>> 'GGTT' #Suffix
>>> s[-4:] # also like s[len(s)-4:]
>>> 'GGTT'
```

### Indexing (0-based)
```python
s = "GATTACA"
s[0]   # 'G'
s[3]   # 'T'
￼￼￼Prefix  
Starts at the beginning of the string.  
Examples: ￼￼"G"￼￼, ￼￼"GA"￼￼, ￼￼"GAT"￼￼.

￼￼￼Suffix  
Ends at the end of the string.  
Examples: ￼￼"A"￼￼, ￼￼"CA"￼￼, ￼￼"ACA"￼￼.
```
### Slicing
```python
s = "GATTACA"
s[1:4]   # 'ATT'
s[:3]    # 'GAT'
s[4:]    # 'ACA'
```
---

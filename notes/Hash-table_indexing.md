The goal of indexing is to preprocess the text T so that later queries can be answered quickly. A k-mer index is one concrete way to do this. A k-mer index is fundamentally mapping **k-mers → genome offsets** the remaining question is purely about **data structures**: how do we store and retrieve this mapping efficiently

Hash tables are introduced as one way to solve exactly this problem.

---

## What a hash table actually is
A hash table stores **key–value pairs**.  
Given a key- it can quickly retrieve the associated value.

Two components define a hash table:

1. A **hash function**  
    This is a deterministic function that takes a key and produces an integer. Then that integer is used to choose a position (often called a _bucket_) in an array.
    
2. A **table (array) of buckets**  
    Each bucket can store zero or more key–value pairs.
    
The key idea is that instead of searching through all keys, we compute where a key should be stored and look only there.

---

## Why collisions exist and why must be handled
Hash functions map a very large space of possible keys (e.g., all strings) into a much smaller space of bucket indices. Because of this collisions are unavoidable: different keys can map to the same bucket.

Correctness of a hash table never depends on collisions not happening. Instead, correctness depends on having a clear rule for what to do when collisions happen.

---

## Closed addressing (separate chaining)

In closed addressing each bucket stores not just one key–value pair but a collection of them. The most common conceptual model is a **linked list**, although in practice this could also be a dynamic array.

**Example:**
Assume we have a hash table with **5 buckets**:
```
index:   0    1    2    3    4
table:  [ ]  [ ]  [ ]  [ ]  [ ]
```
Each bucket can store a linked list of key–value pairs.

### 1. define a simple hash function (for illustration)
   simple hash function:
```
hash(key) = (number of letters in key) mod 5
```

### 2. Insert Keys - Collision example:
   ```
("cat", 10)
("dog", 20)
("cow", 30)
("goat", 40)
   ```

-  lets Insert ("cat", 10)
	- length("cat") = 3
	- hash = 3 mod 5 = 3
	- now lets insert cat in *3rd* bucket 
	  `[ ]  [ ]  [ ]  ("cat",10)  [ ]`

- Now lets insert ("dog" ,20)
	- length("dog") = 3
	- hash = 3 mod 5 = 3 → collision!
Bucket 3 already has `"cat"`.

With **closed addressing**, we don’t overwrite, so we add a node to the linked list.
`[ ]  [ ]  [ ]  ("cat",10) → ("dog",20)  [ ]`

- Next lets Insert ("cow", 30)
	- length("cow") = 3
	- hash = 3 → collision again
Append to the list:
`table[3] = ("cat",10) → ("dog",20) → ("cow",30)`

- On last lets Insert ("goat", 40)
	- length("goat") = 4
	- hash = 4 mod 5 = 4
```
index:   0    1    2    3                                  4
table:  [ ]  [ ]  [ ]  ("cat",10)→("dog",20)→("cow",30)   ("goat",40)

```
### 3. lets lookup using closed addressing

Lookup key = "dog"
1. Compute hash:
    `hash("dog") = 3`
2. Go directly to `table[3]`
3. Walk the linked list:
    - compare "cat" ≠ "dog"
    - compare "dog" = "dog" → found
    - Return the value `20`
> Lets note that these are only three comparisons not a full table scan.

Lookup key = "goat"
1. hash("goat") = 4
2. go to `table[4]`
3. first node matches → found immediately

### What happens if the key we query is not present?
Lookup key = "horse" (not present)
1. hash("horse") = 5 mod 5 = 0
2. go to `table[0]`
3. bucket empty → key not present

### Why this is called “closed addressing”
The word “closed” means:
> All elements that hash to a bucket are stored **inside that bucket**.

Each bucket _owns_ its collisions.

---

## Now connect this to k-mer indexing
In our case with Genome the “value” we care about is not a single number, but a list of offsets. This makes the structure a multimap:
- key = k-mer (string of length k)
- value = list of genome positions where that k-mer occurs
So each entry in a bucket is:
```
(k-mer, [offset₁, offset₂, offset₃, …])
```

Building a hash-table k-mer index (step by step)
Let the text be:
```
T = A C G T A C G
    0 1 2 3 4 5 6
```

Lets build a 3-mer:
`k =3`

We create an empty hash table. Conceptually:

```
table = array of buckets
```
Now we scan the text from left to right.


### Offset 0
Substring:
`T[0:3] = ACG`
- hash("ACG") → bucket i
- bucket i is empty
- insert: `("ACG", [0])`


### Offset 1
Substring:
`T[1:4] = CGT`
- hash("CGT") → bucket j
- bucket j is empty
- insert: `("CGT", [1])`


### Offset 2
Substring:
`T[2:5] = GTA`
- hash("GTA") → bucket i  
    (collision: same bucket as "ACG")
- scan bucket i:
    - "ACG" ≠ "GTA"
- insert new entry:
`("GTA", [2])`
Now bucket i contains:
`("ACG", [0]) → ("GTA", [2])`

### Offset 3
Substring:
`T[3:6] = TAC`
- hash("TAC") → bucket k
- insert:
`("TAC", [3])`

### Offset 4
Substring:
`T[4:7] = ACG`
- hash("ACG") → bucket i
- scan bucket i:
    - key matches "ACG"
- append offset:
`("ACG", [0, 4])`
This is the multimap behavior.

### Final structure of the has table looks:
```
ACG → [0, 4]
CGT → [1]
GTA → [2]
TAC → [3]
```
But physically, it is stored across buckets, with collisions handled by linked lists.

---

### Querying the hash-table index

``q = "ACG"``

now lets compute the lookup:
``hash("ACG") → bucket i``

Scan bucket i:
`("ACG", [0, 4]) → ("GTA", [2])`

3. Compare keys:
- "ACG" == "ACG" → found
3. Retrieve the value:
`candidate offsets = [0, 4]`
These offsets are candidate alignment locations but not the guaranteed matches!

---
### Why verification is still required
The hash table guarantees only one thing:
> If a k-mer appears in the text, all of its occurrences will be listed.

It does **not** guarantee:
- that the full pattern matches
- that surrounding characters match
- that alignment is valid

so every offset returned by the hash table must be checked by comparing the full pattern against the text. This is the same verification step used with sorted indexes.


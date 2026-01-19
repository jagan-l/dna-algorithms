
## What is read alignment?
- Read alignment = figuring out where each short sequencing read came from in a reference genome.
- Input:
  - Reference genome → very long string
  - Reads → many short strings
- Output:
  - For each read one or more possible locations in the reference, plus a confidence score.

At a high level it sounds like string matching but several things make it hard.

---

## Why read alignment is difficult

### 1. Scale of the problem
- Reference genome is huge (billions of characters).
- Number of reads is also huge (millions to billions).
- Naive string matching across the whole genome is computationally impossible.
- We need algorithms that avoid checking every possible position.

---

### 2. Sequencing errors
- Reads can contain:
  - Mismatches- (wrong base)
  - Insertions
  - Deletions
- Because of this, exact matching often fails.
- Alignment must allow approximate matches, not just exact ones.

---

### 3. Biological variation
- The reference genome is not identical to the sample genome.
- Real differences (SNPs, indels, structural variation) cause reads to differ even if sequencing is perfect.
- Aligners must distinguish between:
  - sequencing error
  - true biological variation

---

### 4. Repetitive DNA
- Genomes contain many repeated or nearly repeated regions.
- A single read may align equally well to multiple locations.
- This creates ambiguity:
  - Where did the read really come from?
- Aligners must:
  - report multiple alignments or
  - choose one and report low confidence.

---

### 5. Exact vs approximate matching
- Exact matching is fast but too strict.
- Approximate matching is flexible but expensive.
- Alignment tools balance:
  - speed
  - accuracy
  - memory usage

This trade-off drives most aligner design decisions.

---

## How aligners deal with these problems (high level)

### Filtering vs verification
- Aligners do not try to align reads everywhere.
- Instead:
  1. Filter → find candidate locations using an index
  2. Verify → check those candidates more carefully
- This avoids doing expensive alignment everywhere.

---

### Indexing the genome
- The genome is preprocessed into an index.
- Index helps answer questions like:
  - “Where does this short piece occur?”
- Common indexing ideas:
  - k-mers
  - suffix arrays
  - FM-index (BWT-based)

Indexing is what makes large-scale alignment feasible.

---

### Verification step
- After filtering, candidate locations are checked.
- Verification often uses:
  - edit distance
  - banded alignment
  - optimized dynamic programming
- This is still expensive so the number of candidates must be small.

---

## Mapping quality (confidence)
- Because of ambiguity, aligners assign a mapping quality .
- Mapping quality reflects how confident the aligner is that the reported location is correct.
- Reads that:
  - map uniquely → high mapping quality
  - map to many places → low mapping quality

This is important for downstream analysis.

---

Preprocessing is the idea that you do extra work before searching so that the actual search becomes faster & simpler. In string algorithms, this means spending time analyzing the pattern the text before answering queries like “does this pattern occur here"?


## Offline preprocessing
An algorithm is considered offline when it preprocesses the text T before any queries arrive. This preprocessing step typically builds an index or data structure that represents T in a searchable form.

This model assumes:
- the text T is large and mostly static (in alignment tools this almost always means the reference genome)
- many patterns will be searched against the same T
- we are willing to pay a one time preprocessing cost (Tools like BWA, Bowtie and HISAT explicitly separate an index-build step from an align step.)


## Online processing 
An algorithm is considered online when it does not preprocess the text T. Instead T is scanned directly each time a pattern is searched.
The algorithm may preprocess the pattern P for the current query but that preprocessing is not reusable across different patterns and does not change the representation of T

Example: Boyer–Moore fits squarely in this category:
- it preprocesses the pattern P (bad character table, good suffix table)
- it scans T directly & it builds no index over T

# Notation and Conventions
All array indexing operations are zero based. We use following representation to indicate the nature of the string.

$s_{<(e:e:t:t):n:[i:j]>}$ where:
- $t$ indicates the type of the string element,
  - $b$ for bit
  - $o$ for octet
  - $o:b$ indicates that the octet string is being read as a bit string
- $(e)$ indicates the object type. This can be
  - $P_{<(ec:o):33>}$ an encoded ec point with 33 octets.
  - $P_{<(ec:b):264>}$ an encoded ec point with 264 bits.
  - $P_{<(ed:b):256>}$ an encoded ed point with 256 bits.
  - $n_{<(le:o):32>}$ a $32$ octets little endian representation of the scalar number $n$.
  - $n_{<(le:o:b):256>}$ the 256 bits reading of the a $256/8$ octets little endian representation of the scalar number $n$. This is important if we want to proceed with bitwise selection or manipulation of the string.
- $n$ describes the size of the string. e.g.: $s_{<(b):512>}$ or $s_{<(o):28>}$. Remark that it is allways the size of the last object in the preseeding parentheses.
- $[i:j]$ indicate the selection. Zero based.
  - $[:j]$ means from zero to $j$ 
  - $[i:]$ means from $i$ to the end $n-1$
  - $[i]$ the selection of a single element at position $i$

$s_{<b:n>}||t_{<b:m>} = u_{<b:n+m>}$ concatenated bit strings $s_{<b:n>}$ and $t_{<b:m>}$.

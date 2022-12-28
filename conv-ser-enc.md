# Notation and Conventions
All array indexing operations are zero based. We use following representation to indicate the nature of the string.

$s_{<t(e):n:[i:j]>}$ where:
- $t$ indicates the type of the string element,
  - $b$ for bit
  - $o$ for octet
- $(e)$ indicates the object type. This can be
  - $P_{<(ec)>}$ an encoded ec point.
  - $P_{<(ed)>}$ an encoded ed point.
  - $n_{<o(le):32>}$ a $32$ octets little endian representation of the scalar number $n$.
- $n$ describes the size of the string. e.g.: $s_{<b:512>}$
- $[i:j]$ indicate the selection. Zero based.
  - $[:j]$ means from zero to $j$ 
  - $[i:]$ means from $i$ to the end $n-1$
  - $[i]$ the selection of a single element at position $i$

$s_{<b:n>}||t_{<b:m>} = u_{<b:n+m>}$ concatenated bit strings $s_{<b:n>}$ and $t_{<b:m>}$.

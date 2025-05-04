# Sudoku Solver with Z3 (and also 24 solver!)

This repository contains a Python implementation of a Sudoku solver using the Z3 SMT solver (`z3-solver` package). It encodes the classic Sudoku constraints (F₁–F₄) and the puzzle givens (F₅) as Boolean formulas and solves them via Z3.

---

## Overview

We model a 9×9 Sudoku as a set of Boolean variables:

> $x_{i,j,k}$ is **true** if and only if the cell in row $i$ (0–8), column $j$ (0–8) contains the number $k+1$.

Using these variables, we assert the following constraints:

### F₁: Exactly one number per cell

For each cell $(i,j)$, exactly one of the nine values $k=1\dots9$ must hold:

$$
\sum_{k=1}^9 x_{i,j,k} = 1.
$$

In Z3 this is encoded with a pseudo‑Boolean equality:

```python
s.add(PbEq([(x[i][j][k], 1) for k in range(9)], 1))
```

### F₂: Each number appears once in every row and column

* **Row constraint**: For each row $i$ and digit $k$, the sum over columns $j$ is 1:
  $\sum_{j=1}^9 x_{i,j,k} = 1$.
* **Column constraint**: For each column $j$ and digit $k$, the sum over rows $i$ is 1:
  $\sum_{i=1}^9 x_{i,j,k} = 1$.

In Z3:

```python
# rows:
s.add(PbEq([(x[i][j][k], 1) for j in range(9)], 1))
# columns:
s.add(PbEq([(x[i][j][k], 1) for i in range(9)], 1))
```

### F₃: Each number appears once in each 3×3 block

Partition the grid into nine 3×3 blocks. For each block and digit, the sum over the 9 cells equals 1:

$$
\sum_{(i,j) \in \text{block}} x_{i,j,k} = 1.
$$

Z3 encoding:

```python
block_cells = []
for di in range(3):
    for dj in range(3):
        block_cells.append((x[3*bi+di][3*bj+dj][k], 1))
s.add(PbEq(block_cells, 1))
```

### F₄: No cell has two numbers

This is actually enforced by F₁ (exactly one true literal per cell), making an explicit F₄ redundant.

### F₅: Given clues

For each non‑zero entry $v$ in the input grid at $(i,j)$, we assert:

```python
s.add(x[i][j][v-1])  # force that literal to true
```

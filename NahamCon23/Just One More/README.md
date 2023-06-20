# Just One More
We are given two files `jom.py` and `output.txt`

```python
import random

OUT = open('output.txt', 'w')
FLAG = open('flag.txt', 'r').read().strip()
N = 64

l = len(FLAG)
arr = [random.randint(1,pow(2, N)) for _ in range(l)]
OUT.write(f'{arr}\n')

s_arr = []
for i in range(len(FLAG) - 1):
    s_i = sum([arr[j]*ord(FLAG[j]) for j in range(l)])
    s_arr.append(s_i)
    arr = [arr[-1]] + arr[:-1]

OUT.write(f'{s_arr}\n')
```

This program first generates an array `arr` of length `l` of integers between `1` and `2**64`.<br>
Next, an array `s_arr` is generated whose each element is the sum of `arr[j]` multiplied with `ord(FLAG[j])` and in each step `arr` is rotated right once.<br>
`output.txt` contains the output of this program ran with the flag.<br>
We know `l = 38` since the output's first line contains an array of length 38.<br>

If we consider each character of the flag to have the ASCII value $f_i$ and the value of arr to be $a_i$ for $i = 0,1,\ldots,37$, then we are given the following equations:



$$
\begin{align*}
f_0\cdot a_0 + f_1\cdot a_1 +\cdots + f_{36}\cdot a_{36} + f_{37}\cdot a_{37} &= s_0 \\
f_0\cdot a_{37} + f_1\cdot a_0 + \cdots + f_{36}\cdot a_{35} + f_{37}\cdot a_{36} &= s_1 \\
f_0\cdot a_{36} + f_1\cdot a_{37} + \cdots + f_{36}\cdot a_{34} + f_{37}\cdot a_{35} &= s_2 \\
\vdots \\
f_0\cdot a_{2} + f_1\cdot a_{3} + \cdots + f_{36}\cdot a_{0} + f_{37}\cdot a_{1} &= s_{36} \\
\end{align*}
$$

So, we have 38 unknowns $f_i$ for $i=1, 2,\ldots,37$ and 37 equations. But, we know the value of a few of these unknowns since we know the flag is of the form `flag\{[0-9a-f]{32}\}`.<br>
So, we know $f_0 = \text{ord}('f') = 102$, so we now get 37 equations and 37 unknowns and we can solve them using any linear algebra solver.<br>
I chose to use SymPy since it does computation with infinite precision.<br>

```python
from sympy import *
from sympy.solvers.solveset import linsolve

rands = [12407953253235233563, 3098214620796127593, 18025934049184131586, ...]
rhs = [35605255015866358705679, 36416918378456831329741, 35315503903088182809184, ...]

b = []
for i in range(len(rhs)):
    b.append(rhs[i] - ord('f') * rands[-i])

A = []
for i in range(len(rhs)):
    A.append(rands[1:])
    rands = [rands[-1]] + rands[:-1]

g = linsolve((Matrix(A), Matrix(b)))
print(''.join([chr(x) for x in g.args[0]]))
``` 

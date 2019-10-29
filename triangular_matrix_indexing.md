[Linear index upper triangular matrix](https://stackoverflow.com/questions/27086195/linear-index-upper-triangular-matrix)

```
0  a0  a1  a2  a3
0   0  a4  a5  a6
0   0   0  a7  a8
0   0   0   0  a9
0   0   0   0   0
```

from k to (i,j)

```
i = n - 2 - floor(sqrt(-8*k + 4*n*(n-1)-7)/2.0 - 0.5)
j = k + i + 1 - n*(n-1)/2 + (n-i)*((n-i)-1)/2
```

from (i, j) to k

```
k = (n*(n-1)/2) - (n-i)*((n-i)-1)/2 + j - i - 1
```

[Mapping elements in 2D upper triangle and lower triangle to linear structure](https://stackoverflow.com/questions/4803180/mapping-elements-in-2d-upper-triangle-and-lower-triangle-to-linear-structure)

```
    0  1  2  3  4  5  6
 0  00 01 02 03 04 05 06
 1     07 08 09 10 11 12
 2        13 14 15 16 17
 3           18 19 20 21
 4              22 23 24
 5                 25 26
 6                    27
```

```
int y = (int)((-1+sqrt(8*index+1))/2);  
int x = index - y*(y+1)/2;
```

or 
```
int c = element; 
int r = 0; 
while (c - (r+1) >= 0) 
{ 
  r++;
  c -= r;
}
```

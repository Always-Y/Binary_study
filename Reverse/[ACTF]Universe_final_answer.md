

# [ACTF]Universe_final_answer

---

- Z3求解器
- 

---





### 解密脚本

```python

from z3 import *

v1,v2,v3,v4,v5,v6,v7,v8,v9,v10,v11=BitVecs('v1 v2 v3 v4 v5 v6 v7 v8 v9 v10 v11',32)

s = Solver()
s.add(-85 * v9 + 58 * v8 + 97 * v6 + v7 + -45 * v5 + 84 * v4 + 95 * v2 - 20 * v1 + 12 * v3 == 12613)
s.add(30 * v11 + -70 * v9 + -122 * v6 + -81 * v7 + -66 * v5 + -115 * v4 + -41 * v3 + -86 * v1 - 15 * v2 - 30 * v8 == -54400)
s.add(-103 * v11 + 120 * v8 + 108 * v7 + 48 * v4 + -89 * v3 + 78 * v1 - 41 * v2 + 31 * v5 - (v6 << 6) - 120 * v9 == -10283)
s.add(71 * v6 + (v7 << 7) + 99 * v5 + -111 * v3 + 85 * v1 + 79 * v2 - 30 * v4 - 119 * v8 + 48 * v9 - 16 * v11 == 22855)
s.add( 5 * v11 + 23 * v9 + 122 * v8 + -19 * v6 + 99 * v7 + -117 * v5 + -69 * v3 + 22 * v1 - 98 * v2 + 10 * v4 == -2944)
s.add(-54 * v11 + -23 * v8 + -82 * v3 + -85 * v2 + 124 * v1 - 11 * v4 - 8 * v5 - 60 * v7 + 95 * v6 + 100 * v9 == -2222)
s.add(-83 * v11 + -111 * v7 + -57 * v2 + 41 * v1 + 73 * v3 - 18 * v4 + 26 * v5 + 16 * v6 + 77 * v8 - 63 * v9 == -13258)
s.add(81 * v11 + -48 * v9 + 66 * v8 + -104 * v6 + -121 * v7 + 95 * v5 + 85 * v4 + 60 * v3 + -85 * v2 + 80 * v1 == -1559)
s.add(101 * v11 + -85 * v9 + 7 * v6 + 117 * v7 + -83 * v5 + -101 * v4 + 90 * v3 + -28 * v1 + 18 * v2 - v8 == 6308)
s.add(99 * v11 + -28 * v9 + 5 * v8 + 93 * v6 + -18 * v7 + -127 * v5 + 6 * v4 + -9 * v3 + -93 * v1 + 58 * v2 == -1697)
s.add(v1&0xFFFFFF00==0)
s.add(v2&0xFFFFFF00==0)
s.add(v3&0xFFFFFF00==0)
s.add(v4&0xFFFFFF00==0)
s.add(v5&0xFFFFFF00==0)
s.add(v6&0xFFFFFF00==0)
s.add(v7&0xFFFFFF00==0)
s.add(v8&0xFFFFFF00==0)
s.add(v9&0xFFFFFF00==0)
s.add(v11&0xFFFFFF00==0)
print s.check()
m = s.model()
for i in m:
    print ("%s=%c"%(i,chr(m[i].as_long())))
```


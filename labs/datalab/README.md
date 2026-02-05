
在这个 lab 中，需要在各种限制下，基于位运算和逻辑运算，实现一些函数。

我找到的 bdd checker 对于大部分函数来说是一致的，一些浮点数函数对不上，好在具体测试逻辑在 `test.c` 文件夹中，手动修补了测试范围。似乎经常有所变更，所以查到的题目很容易不一样。(我所用的 `datalab-handout.tar` 的 SHA256 值为 `a3bd21b752ecc7724d284417b1e19aa9f3839554893a4282ebe8e901c9fb2e02  datalab-handout.tar`)

因为 nixos 配置环境很麻烦，使用了 Docker + Justfile 简化使用（使用 `just --list` 查看使用方法）。

## bitXor

**题目要求：** 只使用 `~` 和 `&` 实现 `x ^ y`

- Example: `bitXor(4, 5) = 1`
- Legal ops: `~ &`
- Max ops: 14
- Rating: 1

德摩根律推导即可。
```c
int bitXor(int x, int y) {
    return ~(~(x & ~y) & ~(~x & y));
}
```

## tmin

**题目要求：** 返回最小二进制补码整数 $TMin$

- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 4
- Rating: 1

直接左移 31 位。
```c
int tmin(void) {
    return 1 << 31;
}
```

## isTmax

**题目要求：** 判断 $x$ 是否为最大二进制补码数 $TMax$

- Returns 1 if x is the maximum, two's complement number, and 0 otherwise
- Legal ops: `! ~ & ^ | +`
- Max ops: 10
- Rating: 1

注意到 $TMax \ \operatorname{XOR} \ (TMax + 1) = FF\dots_{16}$，注意特殊值 $x = FF\dots_{16}$ 本身即可。
```c
int isTmax(int x) {
    return !!(x + 1) & !~(x ^ (x + 1));
}
```

## allOddBits

**题目要求：** 判断所有奇数位都为 1

- Return 1 if all odd-numbered bits in word set to 1, where bits are numbered from 0 (least significant) to 31 (most significant)
- Examples: `allOddBits(0xFFFFFFFD) = 0`, `allOddBits(0xAAAAAAAA) = 1`
- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 12
- Rating: 2

构造出来然后比照一下就行。
```c
int allOddBits(int x) {
    int mask = 0;
    mask |= 0xAA << 24;
    mask |= 0xAA << 16;
    mask |= 0xAA << 8;
    mask |= 0xAA;
    return !((x & mask) ^ mask);
}
```

## negate

**题目要求：** 返回 $-x$

- Example: `negate(1) = -1`
- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 5
- Rating: 2

`-x = ~x + 1`
```c
int negate(int x) {
    return ~x + 1;
}
```

## isAsciiDigit

**题目要求：** 判断 $30_{16} \leq x \leq 39_{16}$（ASCII 码字符 '0' 到 '9'）

- Return 1 if `0x30 <= x <= 0x39` (ASCII codes for characters '0' to '9')
- Examples: `isAsciiDigit(0x35) = 1`, `isAsciiDigit(0x3a) = 0`, `isAsciiDigit(0x05) = 0`
- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 15
- Rating: 3

利用取负实现减法，然后判断符号位。
```c
int isAsciiDigit(int x) {
    int min = 0x2f; // 0x30 - 1
    int max = 0x3A; // 0x39 + 1
    min = min + (~x + 1);
    max = x + (~max + 1);
    return !~(min >> 31) & !~(max >> 31);
}
```

## conditional

**题目要求：** 实现三元运算符 `x ? y : z`

- Same as `x ? y : z`
- Example: `conditional(2,4,5) = 4`
- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 16
- Rating: 3

利用 $!x$ 把值域缩小到 $\{ 0, 1 \}$，减 $1$ 后得到 $\{ -1, 0 \}$，取与后一个保持不变一个为零。
```c
int conditional(int x, int y, int z) {
    x = (!x) + (~1) + 1;
    return (x & y) | (~x & z);
}
```

## isLessOrEqual

**题目要求：** 判断 $x \leq y$

- If `x <= y` then return 1, else return 0
- Example: `isLessOrEqual(4,5) = 1`
- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 24
- Rating: 3

$x \leq y$ 相当于 $\operatorname{NOT}(x > y)$，利用前面判断符号位的 trick 即可。注意判断是否溢出，特判异号。
```c
int isLessOrEqual(int x, int y) {
    int xs = !!(x >> 31);
    int ys = !!(y >> 31);
    return (xs & !ys) | (!(xs ^ ys) & !((y + (~x + 1)) >> 31));
}
```

## logicalNeg

**题目要求：** 实现 `!x` 运算符

- Implement the `!` operator, using all of the legal operators except `!`
- Examples: `logicalNeg(3) = 0`, `logicalNeg(0) = 1`
- Legal ops: `~ & ^ | + << >>`
- Max ops: 12
- Rating: 4

二分，把所有 1 位压缩到最低位，然后对最低位取反就行了。
```c
int logicalNeg(int x) {
    x |= x >> 16;
    x |= x >> 8;
    x |= x >> 4;
    x |= x >> 2;
    x |= x >> 1;
    return (x & 1) ^ 1;
}
```

## howManyBits

**题目要求：** 返回使用补码表示 $x$ 所需的最少位数

- Return the minimum number of bits required to represent x in two's complement
- Examples: `howManyBits(12) = 5`, `howManyBits(298) = 10`, `howManyBits(-5) = 4`, `howManyBits(0) = 1`, `howManyBits(-1) = 1`, `howManyBits(0x80000000) = 32`
- Legal ops: `! ~ & ^ | + << >>`
- Max ops: 90
- Rating: 4

如果是正数就是求最高位 1 的位置，负数就是最高位 0，负数转正数统一处理就行。然后二分求最高为 1 位置。
```c
int howManyBits(int x) {
    int h0, h1, h2, h4, h8, h16;
    int s = x >> 31;
    x = s ^ x;
    h16 = !!(x >> 16) << 4;
    x >>= h16;
    h8 = !!(x >> 8) << 3;
    x >>= h8;
    h4 = !!(x >> 4) << 2;
    x >>= h4;
    h2 = !!(x >> 2) << 1;
    x >>= h2;
    h1 = !!(x >> 1);
    x >>= h1;
    h0 = x;
    return 1 + h0 + h1 + h2 + h4 + h8 + h16;
}
```

## floatScale2

**题目要求：** 在编码层面求 $2 \times f$

- Return bit-level equivalent of expression `2*f` for floating point argument f
- Both the argument and result are passed as unsigned int's, but they are to be interpreted as the bit-level representation of single-precision floating point values
- When argument is NaN, return argument
- Legal ops: Any integer/unsigned operations incl. `||`, `&&`. Also if, while
- Max ops: 30
- Rating: 4

参见 [[第2章-信息的表示和处理-家庭作业#^a29ca2]]。
```c
unsigned floatScale2(unsigned uf) {
    unsigned s = uf & 0x80000000;
    uf &= 0x7FFFFFFF;

    if (uf >= 0x7F800000) {
        return s | uf;
    }

    if (uf < 0x00800000) {
        uf <<= 1;
    } else {
        uf += 0x00800000;
        if (uf > 0x7F800000) {
            uf = 0x7F800000;
        }
    }

    return s | uf;
}
```

## floatFloat2Int

**题目要求：** 在编码层面实现 `(int) f`

- Return bit-level equivalent of expression `(int) f` for floating point argument f
- Argument is passed as unsigned int, but it is to be interpreted as the bit-level representation of a single-precision floating point value
- Anything out of range (including NaN and infinity) should return `0x80000000u`
- Legal ops: Any integer/unsigned operations incl. `||`, `&&`. Also if, while
- Max ops: 30
- Rating: 4

参见 [[第2章-信息的表示和处理-家庭作业#^ed9ae5]]。
```c
int floatFloat2Int(unsigned uf) {
    unsigned s = uf & 0x80000000;
    int frac = (uf & 0x7FFFFF) | 0x800000;
    int exp = (uf & 0x7F800000) >> 23;
    int E = exp - 127;

    if (E >= 31) {
        return 0x80000000;
    }
    if (E < 0) {
        return 0;
    }

    unsigned V;
    if (E <= 23) {
        V = frac >> (23 - E);
    } else {
        V = frac << (E - 23);
    }
    return s ? -V : V;
}
```

## floatPower2

**题目要求：** 在编码层面实现 $2.0^{x}$

- Return bit-level equivalent of the expression `2.0^x` (2.0 raised to the power x) for any 32-bit integer x
- The unsigned value that is returned should have the identical bit representation as the single-precision floating-point number `2.0^x`
- If the result is too small to be represented as a denorm, return 0. If too large, return +INF
- Legal ops: Any integer/unsigned operations incl. `||`, `&&`. Also if, while
- Max ops: 30
- Rating: 4

参见 [[第2章-信息的表示和处理-家庭作业#^b521a2]]。
```c
unsigned floatPower2(int x) {
    /* Result exponent and fraction */
    unsigned exp, frac;
    unsigned u;

    if (x < -149) {
        /* Too small.  Return 0.0 */
        exp = 0;
        frac = 0;
    } else if (x < -126) {
        /* Denormalized result */
        exp = 0;
        frac = 1 << (149 + x);
    } else if (x < 128) {
        /* Normalized result. */
        exp = 127 + x;
        frac = 0;
    } else {
        /* Too big. Return +oo */
        exp = 0xFF;
        frac = 0;
    }

    /* Pack exp and frac into 32 bits */
    u = exp << 23 | frac;
    /* Return as float */
    return u;
}
```

## 评分结果

```bash
Correctness Results	Perf Results
Points	Rating	Errors	Points	Ops	Puzzle
1	1	0	2	8	bitXor
1	1	0	2	1	tmin
1	1	0	2	8	isTmax
2	2	0	2	10	allOddBits
2	2	0	2	2	negate
3	3	0	2	13	isAsciiDigit
3	3	0	2	8	conditional
3	3	0	2	17	isLessOrEqual
4	4	0	2	12	logicalNeg
4	4	0	2	32	howManyBits
4	4	0	2	9	floatScale2
4	4	0	2	14	floatFloat2Int
4	4	0	2	9	floatPower2

Score = 62/62 [36/36 Corr + 26/26 Perf] (143 total operators)
```

当然可以。BKDRHash 是一种**字符串哈希算法**，在 C++ 的字符串匹配、哈希表、去重等场景中比较常见。

---

# 1. BKDRHash 的核心思想

对于一个字符串，例如：

```
abc
```

BKDRHash 会把字符串中的每个字符逐步合成为一个整数哈希值。

核心公式是：

$$\boxed{hash=hash\times seed+c}$$

其中：

- `hash`：当前哈希值
- `seed`：一个固定进制数，常用 `131`
- `c`：当前字符的 ASCII 值

所以可以理解为：

> 把字符串看成一个 **seed 进制数**。

---

# 2. 最基本的代码

```cpp
unsigned long long BKDRHash(string s)
{
    unsigned long long hash = 0;
    unsigned long long seed = 131;

    for (char c : s)
    {
        hash = hash * seed + c;
    }

    return hash;
}
```

最经典的写法就是：

```
hash = hash * 131 + c;
```

---

# 3. 用 `"abc"` 举例

假设：

```
seed = 131
hash = 0
```

字符串：

```
a b c
```

ASCII 值分别是：

$$\quad b=98,\quad c=99$$

---

### 第一步：处理 `a`

$$hash=0\times131+97=97$$

现在：

$$hash=97$$

---

### 第二步：处理 `b`

$$hash=97\times131+98 =12805=12805$$

现在：

$$hash=12805$$

---

### 第三步：处理 `c`

$$hash=12805\times131+99$$

最终得到：

$$hash=1677554$$

所以：

$$BKDRHash("abc")=1677554$$

---

# 4. 从数学上看，它到底是什么？

假设字符串是：

$$s=s_1s_2s_3\cdots s_n$$

那么：

$$hash= s_1\times seed^{n-1} +s_2\times seed^{n-2} +\cdots +s_n$$

例如 `"abc"`：

$$hash= 97\times131^2 +98\times131 +99$$

这和十进制数：

$$123=1\times10^2+2\times10+3$$

非常像。

只不过这里不是：

$$10$$

而是：

$$131$$

所以字符串哈希本质上可以理解为：

> **把字符串按照某个进制转换成一个数。**

---

# 5. 为什么不能直接用 ASCII 拼起来？

比如：

```
abc
```

ASCII 是：

```
97 98 99
```

直接拼接会很麻烦，而且数会迅速变得非常大。

BKDRHash 使用：

```
hash = hash * seed + c;
```

可以高效地逐个处理字符。

例如十进制中：

```
123
```

可以这样计算：

```
x = 0

x = x * 10 + 1
x = x * 10 + 2
x = x * 10 + 3
```

最终：

```
123
```

BKDRHash 完全是一样的思想：

```
hash = hash * 131 + c;
```

---

# 6. 为什么常用 131？

`seed` 的选择会影响**哈希冲突率**。

常见选择有：

```
31
131
1313
13131
```

尤其是：

```
131
```

非常常见。

例如：

```
const unsigned long long seed = 131;
```

不过要注意：

> `131` 并不是什么神奇数字，也不能保证绝对没有冲突。

它只是实践中效果比较好的一个选择。

---

# 7. 什么叫哈希冲突？

不同字符串可能得到相同的哈希值。

例如理论上可能存在：

$$Hash("abc")=Hash("xyz")$$

这就叫：

$$\boxed{\text{哈希冲突}}$$

因为字符串的数量几乎无限，而你的哈希值空间是有限的，所以根据抽屉原理：

> **哈希冲突理论上一定可能发生。**

因此 BKDRHash 通常说的是：

> 快速判断字符串是否相同的“概率方法”。

---

# 8. `unsigned long long` 为什么经常用？

你经常会看到：

```
using ULL = unsigned long long;
```

然后：

```
ULL hash = 0;
```

例如：

```
ULL BKDRHash(string s)
{
    ULL hash = 0;

    for(char c : s)
        hash = hash * 131 + c;

    return hash;
}
```

`unsigned long long` 有 6464 位。

它的范围是：

$$\sim2^{64}-1$$

当计算结果超过最大值时，会发生**无符号整数溢出**，相当于自动：

  $$\bmod 2^{64}$$

所以实际上这个哈希可以理解成：

$$\boxed{ Hash(s)=\sum s_i\cdot131^{n-i}\pmod {2^{64}} }$$

这也是为什么它可以处理很长的字符串，而不用担心数无限增长。

---

# 9. BKDRHash 和字符串哈希有什么关系？

你在算法题里经常见到：

```
h[i] = h[i - 1] * P + s[i];
```

这个其实和 BKDRHash 是同一个核心思想。

假设：

```
s = "abcdef"
```

定义：

$$h[i]$$

表示前 $i$ 个字符的哈希值：

$$h[i]=h[i-1]\times P+s[i]$$

那么：

$$h[1]=a$$$$h[2]=aP+b$$ $$h[3]=aP^2+bP+c$$

……

这就是**多项式哈希（Polynomial Hash）**。

BKDRHash 可以看作其中非常经典的一种形式。

---

# 10. 它最大的用途：快速比较字符串

比如：

```
s1 = abcdefghijklmnopqrstuvwxyz
s2 = abcdefghijklmnopqrstuvwxyz
```

直接比较字符串需要逐字符比较。

如果字符串非常长，我们可以先计算：

```
Hash(s1)
Hash(s2)
```

然后比较：

```
if (Hash(s1) == Hash(s2))
```

就可以快速判断。

当然严格来说：

```
Hash相等
```

只能说明：

> **极大概率字符串相同**

因为理论上仍可能冲突。

---

# 11. 最重要的应用：判断子串是否相等

这是竞赛里最常用的。

例如：

```
s = abcdefgh
```

现在问：

```
s[2...4] 和 s[5...7] 是否相同？
```

如果每次都取出字符串比较，复杂度可能较高。

我们可以预处理前缀哈希：

$$h[i]=h[i-1]\times P+s[i]$$

同时预处理：

$$p[i]=P^i$$

那么子串 [l,r][l,r] 的哈希值可以 O(1)O(1) 得到：

$$\boxed{ Hash(l,r)=h[r]-h[l-1]\times P^{r-l+1} }$$

这个公式非常重要。

---

## 为什么是这个公式？

假设：


$$h[r]$$

是：

$$s_1P^{r-1}+s_2P^{r-2}+\cdots+s_lP^{r-l}+\cdots+s_r$$

而：

$$h[l-1]\times P^{r-l+1}$$

刚好就是：

s1Pr−1+s2Pr−2+⋯+sl−1Pr−l+1s_1P^{r-1}+s_2P^{r-2}+\cdots+s_{l-1}P^{r-l+1}

两个相减：

$$h[r]-h[l-1]P^{r-l+1}$$

前面的字符全部消掉，只剩：

$$s_lP^{r-l}+s_{l+1}P^{r-l-1}+\cdots+s_r$$

这就是子串的哈希。

---

# 12. 你可以这样理解 BKDRHash

我建议你记住这一句话：

> **BKDRHash = 用 `hash = hash × seed + 当前字符` 的方式，把一个字符串编码成一个整数。**

它的本质类似：

### 十进制数字

```
1234
```

计算：

```
x = x * 10 + 1;
x = x * 10 + 2;
x = x * 10 + 3;
x = x * 10 + 4;
```

### BKDRHash

```
abcd
```

计算：

```
hash = hash * 131 + 'a';
hash = hash * 131 + 'b';
hash = hash * 131 + 'c';
hash = hash * 131 + 'd';
```

只是把：

```
10进制
```

换成了：

```
131进制的多项式编码
```

---

## 最后总结

BKDRHash 的核心只有一行：

```
hash = hash * seed + c;
```

数学形式：

$$\boxed{ Hash(s)=\sum_{i=1}^{n}s_i\cdot seed^{n-i} }$$

它主要用于：

- 字符串去重
- 判断字符串是否相等
- 判断子串是否相等
- 字符串查找
- 前缀哈希
- 算法题中的快速比较

**你接下来如果在学字符串哈希，我建议继续理解：为什么前缀哈希能在 O(1)O(1) 求任意子串哈希。** 这个是 BKDRHash 思想在算法题里最重要的应用。
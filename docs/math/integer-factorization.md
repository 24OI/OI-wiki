

在本文中，我们列出了几种用于分解质因数的算法，根据输入数据量的不同，它们的速度快慢也不同。

请注意，如果要分解的数字本来就是素数，那么大多数算法，特别是Fermat的分解算法，Pollard的 $p-1$ 算法和 $\rho$ 算法将运行得非常慢。因此，在尝试将数字分解之前，进行概率（或快速确定性）[素数测试](./prime.md#_1)很有意义。


## 试除法

这是一种最基本的质因数分解算法。

合数n的所有素数都不可能大于 $\sqrt n$ 。因此，除数的范围只能是 $[2, \sqrt n]$ 之间的整数。时间复杂度 $ O(\sqrt n)$ 。

从最小的数 $2$ 开始尝试，如果是因子，那么我们用要分解的数除以该因子，然后重复该过程。如果找不到在 $[2, \sqrt n]$ 的因子，那么数字本身就是素数。

```cpp
#define LL long long
vector<LL> trial_division1(LL n) {
    vector<LL> factorization;
    for (LL d = 2; d * d <= n; d++) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

### 轮式因子分解法

该算法为试除法的优化。一旦知道该数字不能被2整除，就不需要测试所有其他偶数（有点儿像[筛](./sieve.md#_1)）。这样我们只剩下50％的数字。之后，我们从3开始开始试除，就可以跳过不符合条件的数，依此类推。


```cpp
#define LL long long
vector<LL> trial_division2(LL n) {
    vector<LL> factorization;
    while (n % 2 == 0) {
        factorization.push_back(2);
        n /= 2;
    }
    for (LL d = 3; d * d <= n; d += 2) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

此方法可以扩展。如果该数字不能被3整除，我们在之后的计算中也可以忽略能被3整除的数。因此，我们只需要测试数字 $5、7、11、13、17、19、23\cdots$ 即可。通过观察可以发现，我们只需要测试 $d \bmod 6 = 1$ 和 $d \bmod 6 = 5 $ 的数字。因此，只剩下 $\dfrac 1 3$ 的数字需要我们检测。先对素数 $2$ 和 $3$ 进行操作，然后对 $5$ 进行操作，之后是 $7，11\cdots$ ，从而实现这一点。


### 预处理

还有一种进阶的方法，只用素数$2$ 、$3$ 和 $5$ 来实现。预处理两个质数之间的差，使用数组来存它们的差。

```cpp
#define LL long long
vector<LL> trial_division3(LL n) {
    vector<LL> factorization;
    for (int d : {2, 3, 5}) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    static array<int, 8> increments = {4, 2, 4, 2, 4, 6, 2, 6};
    int i = 0;
    for (LL d = 7; d * d <= n; d += increments[i++]) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
        if (i == 8)
            i = 0;
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

用越来越多的素数来扩展轮式分解，将精确地测试该数是否为质因子。因此，使用[Eratosthenes筛](./sieve.md#_1)对所有小于     $\sqrt {n}$ 的素数进行预处理就是一种很好的方法。

```cpp
#define LL long long
vector<LL> primes;
vector<LL> trial_division4(LL n) {
    vector<LL> factorization;
    for (LL d : primes) {
        if (d * d > n)
            break;
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

## 费马分解法
我们可以写一个奇合数 $n = p \cdot q$ 作为两个平方的差 $n = a ^ 2-b ^ 2$ 。

进而可以得到如下式子：

$$n = \left(\dfrac {p + q} {2} \right)^ 2-\left(\dfrac {p-q} {2} \right)^ 2$$

费马分解法尝试通过给出 $a ^ 2$ 来检验其余部分 $b ^ 2 = a ^ 2- n$ 是否也是平方数。如果是的话，那么 $a+b$ 和 $a-b$ 便是 $n$ 的两个因数。

```cpp
int fermat(int n) {
    int a = ceil(sqrt(n));
    int b2 = a*a - n;
    int b = round(sqrt(b2));
    while (b * b != b2) {
        a = a + 1;
        b2 = a*a - n;
        b = round(sqrt(b2));
    }
    return a - b;
}
```

注意，如果两个因子p和q相差很小，则这种分解方法会非常快。该算法在 $O(| p-q |)$ 时间中运行。但是，一旦因素相差很大，速度就会很慢，这种情况下就很少使用它了。

然而，这种方法仍然有大量的优化算法。例如，通过观察 $a^2$ 对一个固定的小数取模的平方，我们会注意到某些 $a$ 值不可能产生一个平方数 $a^2 - n$ 。

## Pollard's p-1 算法

假设有一个常数 $B$ ，对于任何一个数字 $p$ ，该数就会有一个因子是**B-powersmooth**，**B-powersmooth** 的意思是，一个素数 $d$ 的某一次幂 $d^k$ 如果是 $p-1$ 的因数，那么 $B$ 就是最大的那个 $d^k$ 。例如，$4817191$ 的质因数分解是 $1303 \cdot 3697$ 。这两个因子就分别是**31-powersmooth**和**16-powersmooth**，因为 $1303 - 1 = 2 \cdot3 \cdot 7 \cdot 31$ , $ 3697 - 1 = 2^4 \cdot 3 \cdot 7 \cdot 11$ 。1974年，约翰·波拉德发明了一种从合数中提取**B-powersmooth**因子的方法。

这个想法来自费马小定理。设 $n$ 的因式分解为 $n = p \cdot q$，它说，如果 $a$ 和 $p $ 是共生的，下面的表述成立:

$$ a^{p - 1} \equiv 1 \pmod{p} $$

这也就意味着：

$$ a^{(p - 1)^k} \equiv a^{k \cdot (p - 1)} \equiv 1 \pmod{p} $$

所以对于任意的 $M$ ， 如果有 $p - 1 ~|~ M$ ，我们就有 $a^M \equiv 1$ ， 进而 $p ~|~ \gcd(a^M - 1, n)$ ，这就意味着 $a^M - 1 = p \cdot r$

因此，如果 $n$ 的一个因子 $p - 1$ 可以整除 $M$ ，我们就可以利用欧几里得算法提取一个因子。

很明显，最小的M是每个B-powersmooth数的最小公倍数 $\text{lcm}(1，~2~，3~，4~，~\cdots，~B)$ 。即:

 $$M = \prod_{\text{prime}\  q \le B} q^{\lfloor \log_q B \rfloor}$$ 

注意到, 如果对于 $n$ 所有的因数 $p$ 都有 $p-1$ 可以整除 $M$ , 那么我们就有 $n= \gcd(a^M - 1, n)$ 。 在这种情况下，我们不会得到任何因数。因此，我们将在计算 $M$ 的同时尝试执行 $\gcd$ 多次。

对于比较小的 $B$ ，某些合数没有**B-powersmooth**。例如，合数 $100,000,000,000,000,493 = 763,013 \cdot 131,059,365,961 $ 的因子为$190,753$ **-powersmooth** 和 $1,092,161,383 $**-powersmooth** 。我们必须选择满足 $B \ge 190,753$ 的 $B$ 来分解该数字。

在以下程序中，我们从 $B = 10$ 开始，并在每次迭代后增加 $B$ 。

```cpp
#define LL long long
LL pollards_p_minus_1(LL n) {
    int B = 10;
    LL g = 1;
    while (B <= 1000000 && g < n) {
        LL a = 2 + rand() %  (n - 3);
        g = gcd(a, n);
        if (g > 1)
            return g;

        // compute a^M
        for (int p : primes) {
            if (p >= B)
                continue;
            LL p_power = 1;
            while (p_power * p <= B)
                p_power *= p;
            a = power(a, p_power, n);

            g = gcd(a - 1, n);
            if (g > 1 && g < n)
                return g;
        }
        B *= 2;
    }
    return 1;
}
```

注意，这是一个概率算法。可能出现算法找不到因子的情况。
每次迭代的复杂度是 $O(B \log B \log^2 n)$ 。

## Pollard's rho 算法

John Pollard的另一种分解算法。

该算法观察一个伪随机序列 ${x_i} = {x_0，~f(x_0)，~f(f(x_0))，~\dots}$ ，其中 $f$ 是一个多项式函数，通常情况下，$f(x) = x^2 + c \bmod n $ 中的 $c = 1$ 。

实际上，我们对序列 ${x_i}$ 不是很感兴趣，我们对序列 ${x_i \bmod p}$ 更感兴趣。由于 $f$ 是一个多项式函数，所有的值都在 $[0,~p)$ 范围内，这个序列迟早会开始循环。**birthday悖论**表明，在循环开始之前，预期的元素数量是 $O(\sqrt{p})$ 。如果 $p$ 小于 $\sqrt{n}$ ，循环很可能在元素数量达到 $O(\sqrt[4]{n})$ 时开始。

这是一个可视化的序列 ${x_i \bmod p}$ , $n = 2206637$ , $p = 317$ , $x_0 = 2$ , $f(x) = x^2 + 1$ 。从序列的形式可以很清楚地看出为什么该算法称为 $Pollard's\  \rho$ 算法。


![](./images/integer-factorization1.png)


其实还有一个未解决的问题。我们还不知道 $p$ 是多少，那么如何讨论序列 ${x_i \bmod p}$呢?

但是实际上也很简单。有周期序列 $\{x_i \bmod p \} _ {i \le j}$ 满足以下两个条件, $s, t \le j$ 并且 $x_s \equiv x_t \bmod p$  当且仅当这个方程可以写成 $ x_s - x_t \equiv 0 \bmod p$ 也即  $p ~|~ \gcd(x_s - x_t, n)$ 。

因此，如果我们找到了 $s$ 和 $t$ ，使得 $g = \gcd(x_s - x_t, n) \gt 1$, 我们就发现 $n$ 的一个因数 $g$ ，同时也是一个周期。

需要注意的是，当 $g = n$ 的时候，就说明我们还没有找到一个合适的因数， 我们必须重复测试算法不同参数(不同的初始值，不同常数 $c$ )。

而为了找到周期，我们可以使用任何常见的周期检测算法。

###  弗洛伊德（Floyd）判圈算法

这个算法通过使用两个指针来找到一个循环。这些指针在序列上以不同的速度移动。在每次迭代中，第一个指针前进到下一个元素，第二个指针前进两个元素。不难看出，如果存在一个循环，第二个指针将至少完成一个完整的循环，然后在接下来的几个循环中遇到第一个指针。如果周期长度为 $\lambda$，而 $\mu$ 是周期开始的第一个元素，那么算法将在 $ O(\lambda + \mu) $ 时间内运行。

这个算法也被称为龟兔算法，基于乌龟(这里是慢的指针)和兔子(这里是快的指针)赛跑的故事。

实际上，使用这种算法(在 $O(\lambda + \mu)$ 时间和 $O(1)$ 空间中)可以确定参数 $\lambda$ 和 $\mu$ ，但这里只是求循环的简化版本。一旦检测到一个周期，该算法就返回true。如果序列没有循环，那么函数就永远不会停止。但在Pollard的 $\rho$ 算法中，这种情况不会发生。
伪代码：

$$
\begin{array}{ll}
1 &\text{function floyd(f, x0):}\\
2 &\qquad\text{tortoise = x0}\\
3 &\qquad\text{hare = f(x0)}\\
4 &\qquad\text{while tortoise != hare:}\\
5 &\qquad\qquad\text{tortoise = f(tortoise)}\\
6 &\qquad\qquad\text{hare = f(f(hare))}\\
7 &\qquad\text{return true}\\
\end{array}
$$

具体实现：

```cpp
#define LL long long 
LL mult(LL a, LL b, LL mod) {
    return (__int128)a * b % mod;
}

LL f(LL x, LL c, LL mod) {
    return (mult(x, x, mod) + c) % mod;
}

LL rho(LL n, LL x0=2, LL c=1) {
    LL x = x0;
    LL y = x0;
    LL g = 1;
    while (g == 1) {
        x = f(x, c, n);
        y = f(y, c, n);
        y = f(y, c, n);
        g = gcd(abs(x - y), n);
    }
    return g;
}
```

下面的表格显示了该算法参数为 $n = 2206637$ , $x_0 = 2$ , $c = 1$ 的时候， $x$ 和 $y$ 的值

$$\newcommand\T{\Rule{0pt}{1em}{.3em}}
\begin{array}{|l|l|l|l|l|l|}
\hline
i & x_i \bmod n & x_{2i} \bmod n & x_i \bmod 317 & x_{2i} \bmod 317 & \gcd(x_i - x_{2i}, n) \\
\hline
0   & 2       & 2       & 2       & 2       & -   \\
1   & 5       & 26      & 5       & 26      & 1   \\
2   & 26      & 458330  & 26      & 265     & 1   \\
3   & 677     & 1671573 & 43      & 32      & 1   \\
4   & 458330  & 641379  & 265     & 88      & 1   \\
5   & 1166412 & 351937  & 169     & 67      & 1   \\
6   & 1671573 & 1264682 & 32      & 169     & 1   \\
7   & 2193080 & 2088470 & 74      & 74      & 317 \\
\hline
\end{array}$$

值得注意的是，该算法用了一个名为mult的函数，在数据范围比较小的时候或者说两个数的乘积小于 $10^{18}$ 的时候，可以用这个函数。但是当数据超范围后，结果将会溢出，此时我们将用下面的函数作为替代，习惯上，我们称之为**快速乘**，和[**快速幂**](./quick-pow.md#_1)思想类似。

```cpp
LL mult(LL a, LL b, LL mod) {
    LL result = 0;
    while (b) {
        if (b & 1)
            result = (result + a) % mod;
        a = (a + a) % mod;
        b >>= 1;
    }
    return result;
}
```

另外，我们还可以实现**Montgomery乘**。

正如上面已经提到的: 如果 $n$ 是合数，并且算法返回 $n$ 作为因子，那么你必须用不同的参数 $x_0$ 和 $c$ 重复这个过程。例如，选择$x_0 = c = 1$ 将不会分解 $25 = 5 \cdot 5$ 。算法会返回 $25$ 。但是选择 $x_0 = 1$ , $c = 2$ 就会对 $25$ 因式分解。

### 布伦特（Brent）算法

布伦特使用了与弗洛伊德类似的算法。它还使用了两个指针。但是，我们不是将指针分别向前推进1和2个元素，而是按照2的幂来推进它们。只要 $2^i$ 大于  $\lambda$ 和 $\mu$ ，我们就能求出循环周期。
伪代码：

$$
\begin{array}{ll}
1 &\text{function brent(f, x0):}\\
2 &\qquad\text{tortoise = x0}\\
3 &\qquad\text{hare = f(x0)}\\
4 &\qquad\text{l = 1}\\
5 &\qquad\text{while tortoise != hare:}\\
6 &\qquad\qquad\text{tortoise = hare}\\
7 &\qquad\qquad\qquad\text{repeat l times:}\\
8 &\qquad\qquad\qquad\text{hare = f(hare)}\\
9 &\qquad\qquad\qquad\text{if tortoise == hare:}\\
10 &\qquad\qquad\qquad\qquad\text{return true}\\
11 &\qquad\qquad\text{l *= 2}\\
12 &\qquad\text{return true}\\
\end{array}
$$

布伦特的算法也在线性时间内运行，但通常比弗洛伊德的算法快，因为它使用的函数f的计算量更少。

使用布伦特算法的直接实现可以加快速度，如果 $k \le \ \dfrac{3 \cdot l}{2}$ ，我们可以省略 $x_l - x_k$ 。此外，我们并没有在每一步执行 $\gcd$ 计算，而是将这些式子相乘，每隔几步就做一次，如果发现我们做过头了就回溯。

```cpp
#define LL long long
LL brent(LL n, LL x0=2, LL c=1) {
    LL x = x0;
    LL g = 1;
    LL q = 1;
    LL xs, y;

    int m = 128;
    int l = 1;
    while (g == 1) {
        y = x;
        for (int i = 1; i < l; i++)
            x = f(x, c, n);
        int k = 0;
        while (k < l && g == 1) {
            xs = x;
            for (int i = 0; i < m && i < l - k; i++) {
                x = f(x, c, n);
                q = mult(q, abs(y - x), n);
            }
            g = gcd(q, n);
            k += m;
        }
        l *= 2;
    }
    if (g == n) {
        do {
            xs = f(xs, c, n);
            g = gcd(abs(xs - y), n);
        } while (g == 1);
    }
    return g;
}
```

小质数的试除法与Pollard的 $\rho$ 算法的布伦特法的组合将成为一种非常强大的因子分解算法。

## 练习题

- [SPOJ - FACT0](https://www.spoj.com/problems/FACT0/)
- [SPOJ - FACT1](https://www.spoj.com/problems/FACT1/)
- [SPOJ - FACT2](https://www.spoj.com/problems/FACT2/)
- [GCPC 15 - Divisions](https://codeforces.com/gym/100753)

**本页面部分内容译自博文 [Эффективные алгоритмы факторизации](http://e-maxx.ru/algo/factorization) 与其英文翻译版 [Integer factorization](https://cp-algorithms.com/algebra/factorization.html) 。其中俄文版版权协议为 Public Domain + Leave a Link；英文版版权协议为 CC-BY-SA 4.0。**
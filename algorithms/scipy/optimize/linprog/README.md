# [`scipy.optimize.linprog`](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.optimize.linprog.html)

线性规划算法主要要达到三个目的：

1. 最小化 linear object function
2. subject to linear equality constraints
3. subject to linear inequality constraints

规定标准形式为：

$$
\min c^Tx \\
s.t. \begin{cases}
    A_{ub}x \le b_{ub} \\
    A_{eq}x = b_{eq} \\
    lb \le x \le ub
\end{cases}

\tag{1}
$$

## 例子

求解下列线性规划问题：

$$
\max z = 2x_1 + 3x_2 - 5x_x,\\
x_1 + x_2 + x_3 = 7,\\
2x_1 - 5x_2 + x_3 \ge 10,\\
x_1 + 3x_2 + x_3 \le 12,\\
x_1, x_2, x_3 \ge 0
$$

将最大化问题取负数化为最小化问题，不难得出：

```py
c = [-2, -3, 5]
A_ub = [
    [-2, 5, -1],
    [1, 3, 1]
]
b_ub = [-10, 12]
A_eq = [
    [1, 1, 1]
]
b_eq = [7]
x0_bounds = x1_bounds = x2_bounds = (0, None)
```

结果如下：

<object data="./res.txt" width=500 height=300></object>

## 带绝对值的目标函数转为线性规划

$$
\min |x_1| + |x_2| + |x_3| + \cdots + |x_n|\\
s.t. Ax \le b

\tag{2}
$$

其中 $x = [x_1, x_2, \cdots, x_n]$，注意到：

$$
x_i = u_i - v_i\\
|x_i| = u_i + v_i

\tag{3}
$$

在取  $u_i = \frac{x_i + |x_i|}{2}, v_i = \frac{|x_i| - x_i}{2}$ 时成立。

式 $(2)$ 就被转换为：

$$
\min \sum_{i=1}^n(u_i + v_i),\\
s.t. \begin{cases}
    [A, -A]\begin{bmatrix}
        u \\ v
    \end{bmatrix} \le b\\
    u, v \ge 0
\end{cases}

\tag{4}
$$

### 例子

$$
\min z = |x_1| + 2|x_2| + 3|x_3| + 4|x_4| \\
s.t. \begin{cases}
    x_1 - x_2 - x_3 + x_4 \le -2,\\
    x_1 - x_2 + x_3 - 3x_4 \le -1,\\
    x_1 - x_2 - 2x_2 + 3x_4 \le -\frac{1}{2}
\end{cases}

\tag{5}
$$

根据 $(3)$ 式，变换得到：

$$
\min c^Ty,\\
[A, -A]\begin{bmatrix}
    u \\ v
\end{bmatrix} \le b,\\
y \ge 0

\tag{6}
$$

式中 $y = \begin{bmatrix}
    u \\v
\end{bmatrix} = [u_1, \cdots, u_4, v_1, \cdots, v_4]^T$：

```py
c = [1, 2, 3, 4, 1, 2, 3, 4]
b = [-2, -1, -1/2]
A = [
    [1, -1, -1, 1],
    [1, -1, 1, -3],
    [1, -1, -2, 3]
]
```

最后通过 $(3)$ 式，可以得到原问题的解：

```py
x = y[0:3] - y[4:-1]
```

## 投资的收益和风险 - 数学建模算法与应用 1.2

主要符号及含义如下：

```txt
n: 资产种数
s[i]: 第 i 种投资项目，i = 0, 1, ..., n（下文也是这个范围），其中 s_0 指存入银行
r[i], p[i], q[i]: 表示 s[i] 的平均收益率、交易费率、风险损失率，其中 p[0] = q[0] = 0
u[i]: 表示 s[i] 的交易定额，也就是说购买额不超过 u[i] 时，交易费按购买 u[i] 进行计算（p[i] * u[i]）
x[i]: 表示投资项目 s[i] 的资金
a: 表示投资风险度
Q: 表示总体收益
M: 投资总额，数额相当大
```

### 模型的分析与建立

模型优化要考虑三点：

1. 风险最小
2. 收益最大

总体风险用所投资的 s[i] 中最大的一个风险来衡量，即

```txt
max {q[i]x[i] | i = 1, 2, ..., n}
```

交易费是一个分段函数：

```txt
交易费 =
    p[i]x[i], x[i] > u[i],
    p[i]u[i], x[i] <= u[i]
```

在题目中给出的 `u[i]` 相对于总投资 `M` 很少，那么 `p[i]u[i]` 更小，可以认为 `p[i][u[i]` 与 `p[i]x[i]` 是相等的（都很小）。
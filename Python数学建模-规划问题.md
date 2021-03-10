# 规划问题



## 线性规划



### 概述

线性规划问题的**目标函数及约束条件均为线性函数**；**约束条件记为 s.t.(即 subject to)**。目标函数可以是求最大值，也可以是求最小值，约束条件的不等号可以 是小于号也可以是大于号。

* 标准形式

    $min c^T x$

* $s.t.$

    $Ax \le b$

    $Aeq * x = beq$

    $lb \le x \le ub$  ：$lb$和$ub$分别是$x$的上界和下界

    

### 求解

* 利用Scipy库求解

    ``` python
    import numpy as np
    from scipy import optimize
    
    # 求解函数
    res = optimize.linprog(c, A, b, Aeq, beq, LB, UB, X0, options)
    
    # 目标函数最小值
    print(res.func)
    
    # 最优解
    print(res.x)
    ```



### 例题

1. 用numpy计算

    $maxz = 2_x1 + 3x_2 - 5x_3 \\s.t. \left\{ \begin{aligned} x_1 + x_2 + x_3 = 7 \\ 2x_1 - 5x_2 = x_3 \ge 10 \\ x_1 + 3x_2 + x_3 \le 12 \\ x_1, x_2, x_3 \ge 0 \end{aligned} \right.$
    
    ``` python
    import numpy as np
    from scipy import optimize
    
    c   = np.array([2, 3, -5])
    a   = np.array([[-2, 5, -1], [1, 3, 1]])
    b   = np.array([-10, 12])
    Aeq = np.array([[1, 1, 1]])
    beq = np.array([7])
    
    res = optimize.linprog(-c, a, b, Aeq, beq)
    
    # 不以科学计数法显示
    np.set_printoptions(suppress=True)
print(res.x)
    ```

    > 注意：先把式子转换为标准式，适当的利用负号
    
    
    
    
    
2. 利用pulp计算

    $max Z = 2 x_1 + 3x_2 + x_3$

    $$ s.t.\left\{ \begin{aligned} x_1 + 2x_2 + 4x_3 = 101 \\ x_1 + 4x_2 + 2x_3 \ge 8  \\ 3x_1 + 2x_2 \ge 6 \\x_1, x_2, x_3 \ge 0 \end{aligned} \right. $$

    ``` python
    import numpy as np
    import pulp as pl
    
    z = [2, 3, 1]
    # 约束
    c = [1, 2, 4]
    a = [[1, 4, 2], [3, 2, 0]]
    b = [8, 6]
    
    # 确定最大化最小化问题，最大化只要把Min改成Max
    m = pl.LpProblem(sense=pl.LpMinimize)
    
    # 定义三个变量放到列表中
    x = [pl.LpVariable(f'x{i}', lowBound=0) for i in [1, 2, 3]]
    
    # 定义目标函数，lpDot可以将两个列表的对应位相乘在加和
    m += pl.lpDot(z, x)
    
    # 设置约束条件
    for i in range (len(a)):
        m += (pl.lpDot(a[i], x) >= b[i])
    m += (pl.lpDot(c, x) == 101)
    
    # 求解
    m.solve()
    print(f'优化结果:{pl.value(m.objective)}')
    print(f'参数取枝:{[pl.value(i) for i in x]}')
    ```



## 整数规划

* 整数规划的模型与线性规划基本相同，只是额外增加了部分变量为整数的约束
* 整数规划求解的基本框架是分支定界法，首先去除整数约束得到“松弛模型”，使用线性规划的方法求解。
* 若有某个变量不是整数，在松弛模型上分别添加约束：$x \le floor(A)$ 和 $x \ge ceil(A)$，然后再分别求解，这个过程叫做==分支==。当节点求解结果中所有变量都是整数时，停止分支
* ==定界==指的是叶子节点产生后，相当于给问题定了一个下界。之后在求解过程中一旦某个节点的目标函数值小于这个下界，直接pass不进行分支。



### 分支定界

* python代码

    ``` python
    class ILP():
        def __init__(self, c, A_ub, b_ub, A_eq, b_eq, bounds = None):
            # 全局参数
            self.LOWER_BOUND = -sys.maxsize
            self.UPPER_BOUND = sys.maxsize
            self.opt_val = None
            self.opt_x = None
            self.Q = Queue()
    
            # 这些参数在每轮计算中都不会改变
            self.c = -c
            self.A_eq = A_eq
            self.b_eq = b_eq
            self.bounds = bounds
    
            # 首先计算一下初始问题
            r = linprog(-c, A_ub, b_ub, A_eq, b_eq, bounds)
    
            # 若最初问题线性不可解
            if not r.success:
                raise ValueError('Not a feasible problem!')
    
            # 将解和约束参数放入队列
            self.Q.put((r, A_ub, b_ub))
    
        def solve(self):
            while not self.Q.empty():
                # 取出当前问题
                res, A_ub, b_ub = self.Q.get(block=False)
    
                # 当前最优值小于总下界，则排除此区域
                if -res.fun < self.LOWER_BOUND:
                    continue
    
                # 若结果 x 中全为整数，则尝试更新全局下界、全局最优值和最优解
                if all(list(map(lambda f: f.is_integer(), res.x))):
                    if self.LOWER_BOUND < -res.fun:
                        self.LOWER_BOUND = -res.fun
    
                    if self.opt_val is None or self.opt_val < -res.fun:
                        self.opt_val = -res.fun
                        self.opt_x = res.x
    
                    continue
    
                # 进行分枝
                else:
                    # 寻找 x 中第一个不是整数的，取其下标 idx
                    idx = 0
                    for i, x in enumerate(res.x):
                        if not x.is_integer():
                            break
                        idx += 1
    
                    # 构建新的约束条件（分割
                    new_con1 = np.zeros(A_ub.shape[1])
                    new_con1[idx] = -1
                    new_con2 = np.zeros(A_ub.shape[1])
                    new_con2[idx] = 1
                    new_A_ub1 = np.insert(A_ub, A_ub.shape[0], new_con1, axis=0)
                    new_A_ub2 = np.insert(A_ub, A_ub.shape[0], new_con2, axis=0)
                    new_b_ub1 = np.insert(
                        b_ub, b_ub.shape[0], -math.ceil(res.x[idx]), axis=0)
                    new_b_ub2 = np.insert(
                        b_ub, b_ub.shape[0], math.floor(res.x[idx]), axis=0)
    
                    # 将新约束条件加入队列，先加最优值大的那一支
                    r1 = linprog(self.c, new_A_ub1, new_b_ub1, self.A_eq,
                                 self.b_eq, self.bounds)
                    r2 = linprog(self.c, new_A_ub2, new_b_ub2, self.A_eq,
                                 self.b_eq, self.bounds)
                    if not r1.success and r2.success:
                        self.Q.put((r2, new_A_ub2, new_b_ub2))
                    elif not r2.success and r1.success:
                        self.Q.put((r1, new_A_ub1, new_b_ub1))
                    elif r1.success and r2.success:
                        if -r1.fun > -r2.fun:
                            self.Q.put((r1, new_A_ub1, new_b_ub1))
                            self.Q.put((r2, new_A_ub2, new_b_ub2))
                        else:
                            self.Q.put((r2, new_A_ub2, new_b_ub2))
                            self.Q.put((r1, new_A_ub1, new_b_ub1))
    ```

* 测试用例

    ``` python
    def test1():
        c = np.array([-3, -4, -1])
        A = np.array([[-1, -6, -2], [-2, 0, 0]])
        b = np.array([-5, -3])
        Aeq = np.array([[0, 0, 0]])
        beq = np.array([0])
        # bounds = [(0, None), (0, None), (0, None)]
        solver = ILP(c, A, b, Aeq, beq)
        solver.solve()
    
        print(solver.opt_val, solver.opt_x)
        
    # -8.0 [2. 0. 2.]
    ```



## 非线性规划

* 非线性规划可以简单分为两种，目标函数为凸函数或者非凸函数
* 凸函数的非线性规划，比如$f(x) = x^2 + y^2 + xy$可以使用cvxpy库完成
* 非凸函数的非线性规划（求极值），可以使用纯数学方法，或者scipy.optimize.minimize
    * fun：求最小值的目标函数
    * args：常数值
    * method：求极值方法，一般默认
    * constraints：约束条件
    * x0：变量的初始猜测值，minimize是局部最优



例：计算$\frac{1}{x} + x$的最小值

``` python
from scipy.optimize import minimize
import numpy as np

def fun(args):
    a = args
    v = lambda x : a / x[0] + x[0]
    return v

args = (1)
x0 = np.asarray((2))
res = minimize(fun(args), x0, method='SLSQP')
print(res.fun)
print(res.success)
print(res.x)
```



例：计算$(2 + x_1) / (1 + x_2) - 3 x_1 + 4x_3$的最小值，其中$x_1, x_2, x_3$范围在0.1到0.9之间

```python
from scipy.optimize import minimize
import numpy as np

def fun(args):
    a, b, c, d= args
    v = lambda x: (a + x[0]) / (b + x[1]) - c * x[0] + d * x[2]
    return v

def con(args): 
    # 约束条件 eq表示 函数结果等于0
    # ineq表示 表达式大于等于0
    x1min, x1max, x2min, x2max, x3min, x3max = args
    con = (
        {'type' : 'ineq', 'fun' : lambda x : x[0] - x1min},
        {'type' : 'ineq', 'fun' : lambda x : -x[0] + x1max},
        {'type' : 'ineq', 'fun' : lambda x : x[1] - x2min},
        {'type' : 'ineq', 'fun' : lambda x : -x[1] + x2max},
        {'type' : 'ineq', 'fun' : lambda x : x[2] - x3min},
        {'type' : 'ineq', 'fun' : lambda x : -x[2] + x3max},
    )
    return con

args = (2, 1, 3, 4)
args1 = (0.1, 0.9, 0.1, 0.9, 0.1, 0.9)
cons = con(args1)
x0 = np.asarray((0.5, 0.5, 0.5))
res = minimize(fun(args), x0, method='SLSQP', constraints=cons)
print(res.fun)
print(res.success)
print(res.x)

# -0.773684210526435
# True
# [0.9 0.9 0.1]
```


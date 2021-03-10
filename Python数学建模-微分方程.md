

# 微分方程

## 微分方程分类

微分方程是用来描述某一函数与其导数之间关系的方程，其解是一个符合方程的函数。微分方程按自变量个数可分为==常微分方程==和==偏微分方程==，前者表达通式

$f( \frac{d^ny}{dx^n}, \frac{d^{n - 1}y}{dx^{n - 1}}, ..., \frac{dy}{dx}, y) = p(x)$

后者有两个以上的自变量，如：

$\frac{\partial \phi}{\partial x} + x \frac{\partial \phi}{\partial y} = 0$



## 微分方程解析解

利用Sympy库进行求解：

例：求解阻尼谐振子的二阶ODE，其表达式为

$\frac{d^2x(t)}{dt^2} + 2 \gamma \omega_0 \frac{dx(t)}{dt} + \omega_0^2 x (t) = 0$



$initial.conditions. \left\{ \begin{aligned} x(0) = 1 \\ \frac{dx(t)}{dt} = 0, t = 0 \end{aligned} \right.$



``` python
import numpy as np
from scipy import integrate
import sympy


def apply_ics(sol, ics, x, known_params):
    free_params = sol.free_symbols - set(known_params)
    eqs = [(sol.lhs.diff(x, n) - sol.rhs.diff(x, n)).subs(x, 0).subs(ics) for n in range (len(ics))]
    sol_params = sympy.solve(eqs, free_params) # 求解代数方程组，得到自由参数表达式，替换自由参数后返回解决结果
    return sol.subs(sol_params)

sympy.init_printing() # 初始化打印环境
t, omega0, gamma = sympy.symbols("t, omega_0, gamma", positive = True) # 标记参数，且均为正
x = sympy.Function('x') # 标记x是微分函数，非变量
ode = x(t) .diff(t, 2) + 2 * gamma * omega0 * x(t).diff(t) + omega0 ** 2 * x(t) # 微分方程，默认齐次
ode_sol = sympy.dsolve(ode) # 用 diff() 和 dsolve 得到通解
ics = {x(0): 1, x(t).diff(t).subs(t, 0): 0} # 将初始条件字典匹配
x_t_sol = apply_ics(ode_sol, ics, t, [omega0, gamma])
sympy.pprint(x_t_sol)
```

结果：

```
                                           ⎛       _______   _______⎞         
       ⎛            γ             1⎞  ω₀⋅t⋅⎝-γ - ╲╱ γ - 1 ⋅╲╱ γ + 1 ⎠   ⎛     
x(t) = ⎜- ───────────────────── + ─⎟⋅ℯ                                + ⎜─────
       ⎜      _______   _______   2⎟                                    ⎜    _
       ⎝  2⋅╲╱ γ - 1 ⋅╲╱ γ + 1     ⎠                                    ⎝2⋅╲╱ 

                            ⎛       _______   _______⎞
     γ             1⎞  ω₀⋅t⋅⎝-γ + ╲╱ γ - 1 ⋅╲╱ γ + 1 ⎠
──────────────── + ─⎟⋅ℯ                               
______   _______   2⎟                                 
γ - 1 ⋅╲╱ γ + 1     ⎠                                 
```



## 微分方程数值解

当ODE无法求的解析解时，可以用Scipy中的`integrate.odeint`求数值解来探索其解的部分性质，并辅以可视化，能直观的展现ODE解的函数表达。

以如下一阶非线性（因为函数y幂次为2）ODE为例：

$\frac{dy}{dx} = x - y(x)^2$

用odeint求其数值解：

``` python
import numpy as np
from scipy import integrate
import matplotlib.pyplot as plt
import sympy

# y_x: 由x组成的y
# f_xy: 由xy组成的function
# x_lim: x的可视范围
# y_lim: y的可视范围
def plot_direction_field(x, y_x, f_xy, x_lim = (-5, 5), y_lim = (-5, 5), ax = None):
    f_np = sympy.lambdify((x, y_x), f_xy, 'numpy') # 将符号表达式转换为隐函数，转换为Numpy矩阵
    x_vec = np.linspace(x_lim[0], x_lim[1], 20)
    y_vec = np.linspace(y_lim[0], y_lim[1], 20)
    if ax is None:
        _, ax = plt.subplots(figsize = (4, 4))
    dx = x_vec[1] - x_vec[0]
    dy = y_vec[1] - y_vec[0]
    for m, xx in enumerate(x_vec):
        for n, yy in enumerate(y_vec):
            Dy = f_np(xx, yy) * dx
            Dx = 0.8 * dx ** 2 / np.sqrt(dx ** 2 + Dy ** 2)
            Dy = 0.8 * Dy * dy / np.sqrt(dx ** 2 + Dy ** 2)
            ax.plot([xx - Dx / 2, xx + Dx / 2], [yy - Dy / 2, yy + Dy / 2], 'b', lw = 0.5)
            ax.axis('tight')
            ax.set_title(r"$%s$"%(sympy.latex(sympy.Eq(y_x.diff(x), f_xy))), fontsize=18)
    return ax

x = sympy.symbols('x')
y = sympy.Function('y')
f = x - y(x) ** 2
f_np = sympy.lambdify((y(x), x), f) # 符号表达式转隐函数
y0 = 1
xp = np.linspace(0, 5, 100)
yp = integrate.odeint(f_np, y0, xp) # 初始y0解f_np，x范围xp
xn = np.linspace(0, -5, 100)
yn = integrate.odeint(f_np, y0, xp)
fig, ax = plt.subplots(1, 1, figsize = (4, 4))
plot_direction_field(x, y(x), f, ax = ax) # 绘制f的场线图
ax.plot(xn, yn, 'b', lw = 2)
ax.plot(xp, yp, 'r', lw = 2)
plt.show()
```





# 传染病模型

## SI模型

``` python
import numpy as np
import scipy.integrate as spi
import matplotlib.pyplot as plt
N = 10000 # N为人群总数
beta = 0.25 # 传染率系数
gamma = 0 # 恢复率系数
I_0 = 1 # 感染者的初始人数
S_0 = N - I_0 # 易感染者的初始人数
T = 150 # 传播时间
INI = (S_0, I_0) # 初始状态下的数组

def funcSI(inivalue, _):
    Y = np.zeros(2)
    X = inivalue
    Y[0] = -(beta * X[0] * X[1]) / N + gamma * X[1] # 易感个体变化
    Y[1] = (beta * X[0] * X[1]) / N - gamma * X[1] # 感染个体变化
    return Y

T_range = np.arange(0, T + 1)
RES = spi.odeint(funcSI, INI, T_range)
plt.plot(RES[:, 0], color = 'darkblue', label = 'Susceptible', marker = '.')
plt.plot(RES[:, 1], color = 'red', label = 'Infection', marker = '.')
plt.title("SI Model")
plt.legend()
plt.xlabel("Day")
plt.ylabel("Number")
plt.show()
```

结果图

![](E:\Study\MyNotes\Python\数模\SI模型.png)





## SIS模型

``` python
import numpy as np
import scipy.integrate as spi
import matplotlib.pyplot as plt
N = 10000 # N为人群总数
beta = 0.25 # 传染率系数
gamma = 0.05 # 恢复率系数
I_0 = 1 # 感染者的初始人数
S_0 = N - I_0 # 易感染者的初始人数
T = 150 # 传播时间
INI = (S_0, I_0) # 初始状态下的数组

def funcSIS(inivalue, _):
    Y = np.zeros(2)
    X = inivalue
    Y[0] = -(beta * X[0]) / N * X[1] + gamma * X[1] # 易感个体变化
    Y[1] = (beta * X[0] * X[1]) / N - gamma * X[1] # 感染个体变化
    return Y

T_range = np.arange(0, T + 1)
RES = spi.odeint(funcSIS, INI, T_range)
plt.plot(RES[:, 0], color = 'darkblue', label = 'Susceptible', marker = '.')
plt.plot(RES[:, 1], color = 'red', label = 'Infection', marker = '.')
plt.title("SIS Model")
plt.legend()
plt.xlabel("Day")
plt.ylabel("Number")
plt.show()
```

结果图：

![](E:\Study\MyNotes\Python\数模\SIS模型.png)





## SIR模型

``` python
import numpy as np
import scipy.integrate as spi
import matplotlib.pyplot as plt

N = 10000 # 人群总数
beta = 0.25 # 传染率系数
gamma = 0.05 # 恢复率系数
I_0 = 1 # 感染者的初始人数
R_0 = 0 # 治愈者的初始人数
S_0 = N - I_0 - R_0 # 易感染者初始人数
T = 150 # 传播时间
INI = (S_0, I_0, R_0) # 初始状态下的数组

def funcSIR(inivalue, _):
    Y = np.zeros(3)
    X = inivalue
    Y[0] = -(beta * X[0] * X[1]) / N # 易感染个体变化
    Y[1] = (beta * X[0] * X[1]) / N - gamma * X[1] # 感染个体变化
    Y[2] = gamma * X[1] # 治愈个体变化
    return Y

T_range = np.arange(0, T + 1)
RES = spi.odeint(funcSIR, INI, T_range)
plt.plot(RES[:, 0], color = 'darkblue', label = 'Susceptible', marker = '.')
plt.plot(RES[:, 1], color = 'red', label = 'Infection', marker = '.')
plt.plot(RES[:, 2], color = 'green', label = 'Recovery', marker = '.')
plt.title("SIR Model")
plt.legend()
plt.xlabel('Day')
plt.ylabel('Number')
plt.show()
```

结果图

![](E:\Study\MyNotes\Python\数模\SIR模型.png)



## SIRS模型

``` python
import numpy as np
import scipy.integrate as spi
import matplotlib.pyplot as plt

N = 10000 # 人群总数
beta = 0.25 # 传染率系数
gamma = 0.05 # 恢复率系数
Ts = 7 # 抗体持续时间
I_0 = 1 # 感染者的初始人数
R_0 = 0 # 治愈者的初始人数
S_0 = N - I_0 - R_0 # 易感染者初始人数
T = 150 # 传播时间
INI = (S_0, I_0, R_0) # 初始状态下的数组

def funcSIR(inivalue, _):
    Y = np.zeros(3)
    X = inivalue
    Y[0] = -(beta * X[0] * X[1]) / N + X[2] / Ts # 易感染个体变化
    Y[1] = (beta * X[0] * X[1]) / N - gamma * X[1] # 感染个体变化
    Y[2] = gamma * X[1] - X[2] / Ts # 治愈个体变化
    return Y

T_range = np.arange(0, T + 1)
RES = spi.odeint(funcSIR, INI, T_range)
plt.plot(RES[:, 0], color = 'darkblue', label = 'Susceptible', marker = '.')
plt.plot(RES[:, 1], color = 'red', label = 'Infection', marker = '.')
plt.plot(RES[:, 2], color = 'green', label = 'Recovery', marker = '.')
plt.title("SIR Model")
plt.legend()
plt.xlabel('Day')
plt.ylabel('Number')
plt.show()
```

结果图：

![](E:\Study\MyNotes\Python\数模\SIRS模型.png)



## SIER模型

``` python
import numpy as np
import scipy.integrate as spi
import matplotlib.pyplot as plt

N = 10000 # 人群总数
beta = 0.6 # 传染率系数
gamma = 0.1 # 恢复率系数
Te = 14 # 疾病潜伏期
I_0 = 1 # 感染者的初始人数
E_0 = 0 # 潜伏者的初始人数
R_0 = 0 # 治愈者的初始人数
S_0 = N - I_0 - R_0 - E_0 # 易感染者初始人数
T = 150 # 传播时间
INI = (S_0, E_0, I_0, R_0) # 初始状态下的数组

def funcSIR(inivalue, _):
    Y = np.zeros(4)
    X = inivalue
    Y[0] = -(beta * X[0] * X[2]) / N # 易感染个体变化
    Y[1] = (beta * X[0] * X[2]) / N - X[1] / Te # 潜伏个体变化
    Y[2] = X[1] / Te - gamma * X[2] # 感染个体变化
    Y[3] = gamma * X[2] # 治愈个体变化
    return Y

T_range = np.arange(0, T + 1)
RES = spi.odeint(funcSIR, INI, T_range)
plt.plot(RES[:, 0], color = 'darkblue', label = 'Susceptible', marker = '.')
plt.plot(RES[:, 1], color = 'orange', label = 'Exposed', marker = '.')
plt.plot(RES[:, 2], color = 'red', label = 'Infection', marker = '.')
plt.plot(RES[:, 3], color = 'green', label = 'Recovery', marker = '.')
plt.title("SIER Model")
plt.legend()
plt.xlabel('Day')
plt.ylabel('Number')
plt.show()
```

结果图：

![](E:\Study\MyNotes\Python\数模\SIER模型.png)

## SIERS模型

``` python
import numpy as np
import scipy.integrate as spi
import matplotlib.pyplot as plt

N = 10000 # 人群总数
beta = 0.6 # 传染率系数
gamma = 0.1 # 恢复率系数
Ts = 7 # 抗体持续时间
Te = 14 # 疾病潜伏期
I_0 = 1 # 感染者的初始人数
E_0 = 0 # 潜伏者的初始人数
R_0 = 0 # 治愈者的初始人数
S_0 = N - I_0 - R_0 - E_0 # 易感染者初始人数
T = 150 # 传播时间
INI = (S_0, E_0, I_0, R_0) # 初始状态下的数组

def funcSIR(inivalue, _):
    Y = np.zeros(4)
    X = inivalue
    Y[0] = -(beta * X[0] * X[2]) / N + X[3] / Ts # 易感染个体变化
    Y[1] = (beta * X[0] * X[2]) / N - X[1] / Te # 潜伏个体变化
    Y[2] = X[1] / Te - gamma * X[2] # 感染个体变化
    Y[3] = gamma * X[2] - X[3] / Ts # 治愈个体变化
    return Y

T_range = np.arange(0, T + 1)
RES = spi.odeint(funcSIR, INI, T_range)
plt.plot(RES[:, 0], color = 'darkblue', label = 'Susceptible', marker = '.')
plt.plot(RES[:, 1], color = 'orange', label = 'Exposed', marker = '.')
plt.plot(RES[:, 2], color = 'red', label = 'Infection', marker = '.')
plt.plot(RES[:, 3], color = 'green', label = 'Recovery', marker = '.')
plt.title("SIERS Model")
plt.legend()
plt.xlabel('Day')
plt.ylabel('Number')
plt.show()
```

结果图:

![](E:\Study\MyNotes\Python\数模\SIERS模型.png)
# 一维插值

插值函数经过样本点，拟合函数一般基于最小二乘法尽量靠近所有样本点穿过。常见插值方法有==拉格朗日插值法==、==分段插值法==、==样条插值法==

* 拉格朗日插值多项式：当节点数n较大时，拉格朗日插值多项式次数较高，可能收敛不一致，且计算复杂。高次插值带来误差的震动现相称为==龙格现象==
* 分段插值：虽然收敛，但光滑性较差。
* 样条插值：由于样条插值可以使用低阶多项式样条实现较小的插值误差，这样就避免了龙格现象。



## 线性插值与样条插值

例：某电学元件的电压数据记录在0~2.25ΠA范围与电流关系满足正弦函数，分别用线性插值和样条插值方法给出经过数据点的数值逼近函数曲线

* 代码：

    ``` python
    import numpy as np
    from matplotlib import pyplot as plt
    from scipy import interpolate
    
    font = {
        'family': "SimSun",
        'weight': 'bold',
        'size': '16'
    }
    
    x = np.linspace(0, 2 * np.pi + np.pi / 4, 10)
    y = np.sin(x)
    _x_new = np.linspace(0, 2 * np.pi + np.pi / 4, 100)
    _f_linear = interpolate.interp1d(x, y)
    tck = interpolate.splrep(x, y)
    _y_bspline = interpolate.splev(_x_new, tck)
    
    # 绘图
    plt.figure(figsize=(16, 8), dpi=80)
    plt.rc('font', **font)
    plt.xlabel(u'安培/A')
    plt.ylabel(u'伏特/V')
    plt.plot(x, y, "o", label = u'原始数据')
    plt.plot(_x_new, _f_linear(_x_new), label = u'线性插值')
    plt.plot(_x_new, _y_bspline, label = u'B-spline插值')
    plt.legend()
    plt.show()
    ```

* 可视化

    ![](http://101.133.134.12:8079/python/%E6%95%B0%E6%A8%A1/%E7%BA%BF%E6%80%A7%E6%8F%92%E5%80%BC%E5%92%8C%E6%A0%B7%E6%9D%A1%E6%8F%92%E5%80%BC.jpg)

## 高阶样条插值

例：某电学元件的点压数据记录在0~10A范围与电流关系满足正弦函数，分别用0-5阶样条插值方法给出经过数据点的数值逼近函数曲线

``` python
import numpy as np
from matplotlib import pyplot as plt
from scipy import interpolate

font = {
    'family': "SimSun",
    'weight': 'bold',
    'size': '16'
}

plt.figure(figsize=(16, 8), dpi=80)
plt.rc('font', **font)

x = np.linspace(0, 10, 11)
y = np.sin(x)
_x_new = np.linspace(0, 10, 101)

# quadratic为3阶，kind后可以直接加数字
for kind in ['nearest', 'zero', 'linear', 'quadratic']:
    f = interpolate.interp1d(x, y, kind=kind)
    _y_new = f(_x_new)
    plt.plot(_x_new, _y_new, label = str(kind))

plt.legend()
plt.show()
```

![](http://101.133.134.12:8079/python/%E6%95%B0%E6%A8%A1/%E9%AB%98%E9%98%B6%E6%A0%B7%E6%9D%A1%E6%8F%92%E5%80%BC.jpg)



# 二维插值

* 方法与一维数据插值类似

例：某二维图像表达式为$(x + y)e^{-5(x^2 + y^2)}$，完成图形的二维插值使其变清晰

``` python
import numpy as np
from matplotlib import pyplot as plt
from matplotlib import cm
from scipy import interpolate

font = {
    'family': "SimSun",
    'weight': 'bold',
    'size': '16'
}

plt.figure(figsize=(16, 8), dpi=80)
plt.rc('font', **font)

def func(x, y):
    return (x + y) * np.exp(-5.0 * (x ** 2 + y ** 2))

#x - y轴分为 15 * 15 的网格
# y, x = np.mgrid[-1 : 1 : 15j, -1 : 1 : 15j]
x = np.linspace(-1, 1, 15)
y = np.linspace(-1, 1, 15)
x, y = np.meshgrid(x, y)
# 计算每个网格点上函数值
fvals = func(x, y)
# 三次样条二维插值
newfunc = interpolate.interp2d(x, y, fvals)
# 计算 100 * 100 网格上插值
_xnew = np.linspace(-1, 1, 100)
_ynew = np.linspace(-1, 1, 100)
fnew = newfunc(_xnew, _ynew)

# 绘图
plt.subplot(121)
im1 = plt.imshow(fvals, extent=[-1, 1, -1, 1], cmap=cm.hot, interpolation='nearest', origin='lower')
plt.colorbar(im1)
plt.subplot(122)
im2 = plt.imshow(fnew, extent=[-1, 1, -1, 1], cmap=cm.hot, interpolation='nearest', origin='lower')
plt.colorbar(im2)
plt.show()
```

![](http://101.133.134.12:8079/python/%E6%95%B0%E6%A8%A1/%E4%BA%8C%E7%BB%B4%E6%8F%92%E5%80%BC.jpg)



# 拟合

## 最小二乘拟合

* 拟合指的是已知某函数的若干离散函数值$f_1, f_2,...,f_n$，通过调整该函数中若干待定系数$f(\lambda _1, \lambda_2,...,\lambda_n)$，使得该函数与已知点集的差别（最小二乘意义）最小
* 如果待定函数是线性，就叫线性拟合或者线性回归（主要在统计中），否则叫作线性拟合或者非线性回归。表达式也可以是分段函数，这种情况下叫做样条拟合。
* 从几何意义上讲，拟合是给定了空间中的一些点，找到一个已知形式未知参数的连续曲面来最大限度的逼近这些点；而插值是找到一个（或几个分片光滑的）连续曲面来穿过这些点。
* 选择参数c使得拟合模型与实际观测值在曲线拟合各点的残差（或离差）$ek = yk-f(xk,c)$的加权平方和达到最小，此时所求曲线称作在加权最小二乘意义下对数据的拟合曲线，这种方法叫做==最小二乘法==

例：对下列电学元件的电压电流记录结果进行最小二乘拟合，绘制相应曲线

| 电流（A） | 8.19 | 2.72 | 6.39 | 8.71 | 4.7  | 2.66 | 3.78 |
| --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 电压（V） | 7.01 | 2.78 | 6.47 | 6.71 | 4.1  | 4.23 | 4.05 |

``` python
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import leastsq

# 字体属性
font = { 
    'family': 'SimSun',
    'weight': 'bold',
    "size": '16'
}

x = np.array([8.19, 2.72, 6.39, 8.71, 4.7, 2.66, 3.78])
y = np.array([7.01, 2.78, 6.47, 6.71, 4.1, 4.23, 4.05])


plt.figure(figsize=(16,8), dpi=80)
plt.rc('font', **font)

# 计算以p为参数的直线与原始数据之间误差
def f(p):
    k, b = p
    return (y - (k * x + b))

# leastsq使得f的输出数组的平方和最小，参数初始值为[1,0]
r = leastsq(f, [1, 0])
k, b = r[0]
plt.scatter(x, y, s = 100, alpha=1.0, marker = 'o', label=u'数据点')

_x = np.linspace(0, 10, 1000)
_y = k * _x + b

plt.plot(_x, _y, color='r', label=u'拟合曲线')
plt.xlabel(u'安培/A')
plt.ylabel(u'伏特/V')
plt.xticks(fontsize=20)
plt.yticks(fontsize=20)
plt.legend()
plt.show()
```

![](http://101.133.134.12:8079/python/%E6%95%B0%E6%A8%A1/%E6%9C%80%E5%B0%8F%E4%BA%8C%E4%B9%98%E6%8B%9F%E5%90%88.jpg)
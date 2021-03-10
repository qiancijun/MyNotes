

# Matplotlib



## 快速开始

绘制一个简单的图形

``` python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 确定横纵坐标
x = range(1, 13)
y = [random.randint(1, 10) for i in x]

# 绘制图形
plt.plot(x, y)

# 展示图形
plt.show()
```

![](.\Python\Matplotlib\快速开始.png)



## 改变展示图形大小

在绘制图形前调用`figure()`

``` python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 确定横纵坐标
x = range(1, 13)
y = [random.randint(1, 10) for i in x]

#改变图形大小
plt.figure(figsize = (20, 8), dpi = 80)

# 绘制图形
plt.plot(x, y)

# 展示图形
plt.show()
```

![](.\Python\Matplotlib\figure.png)



## 设置横坐标

使用`xticks`修改横坐标，纵坐标相同

``` python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 确定横纵坐标
x = range(1, 13)
y = [random.randint(1, 10) for i in x]

#改变图形大小
plt.figure(figsize = (20, 8), dpi = 80)

# 绘制图形
plt.plot(x, y)

# 修改横坐标
_xticks_label = ["{}月".format(i) for i in x]
plt.xticks(x, _xticks_label)


# 展示图形
plt.show()
```

![](.\Python\Matplotlib\xticks.png)



## 显示中文

* 方法一：使用`rc`

    ``` python
    import numpy as np
    import pandas as pd
    from matplotlib import pyplot as plt
    import random
    
    # 字体属性
    font = { 
        'family': 'SimSun',
        'weight': 'bold',
        "size": '16'
    }
    
    # 确定横纵坐标
    x = range(1, 13)
    y = [random.randint(1, 10) for i in x]
    
    #改变图形大小
    plt.figure(figsize = (20, 8), dpi = 80)
    
    # 显示中文
    plt.rc("font", **font)
    
    # 绘制图形
    plt.plot(x, y)
    
    # 修改横坐标
    _xticks_label = ["{}月".format(i) for i in x]
    plt.xticks(x, _xticks_label)
    
    
    # 展示图形
    plt.show()
    ```

* 方法二：使用fontproperties

    ``` python
    import numpy as np
    import pandas as pd
    from matplotlib import pyplot as plt
    import random
    
    # 字体属性
    font = { 
        'family': 'SimSun',
        'weight': 'bold',
        "size": '16'
    }
    
    # 确定横纵坐标
    x = range(1, 13)
    y = [random.randint(1, 10) for i in x]
    
    #改变图形大小
    plt.figure(figsize = (20, 8), dpi = 80)
    
    # 显示中文
    # plt.rc("font", **font)
    
    # 绘制图形
    plt.plot(x, y)
    
    # 修改横坐标
    _xticks_label = ["{}月".format(i) for i in x]
    plt.xticks(x, _xticks_label, fontproperties = "SimSun")
    
    # 展示图形
    plt.show()
    ```

![](.\Python\Matplotlib\中文.png)



## 显示网格

使用`grid`，并且可以设置对应的透明度

``` python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 字体属性
font = { 
    'family': 'SimSun',
    'weight': 'bold',
    "size": '16'
}

# 确定横纵坐标
x = range(1, 13)
y = [random.randint(1, 10) for i in x]

#改变图形大小
plt.figure(figsize = (20, 8), dpi = 80)

# 显示中文
plt.rc("font", **font)

# 显示网格
plt.grid(alpha = 0.4)

# 绘制图形
plt.plot(x, y)

# 修改横坐标
_xticks_label = ["{}月".format(i) for i in x]
plt.xticks(x, _xticks_label)


# 展示图形
plt.show()
```

![](.\Python\Matplotlib\网格.png)



## 设置图形信息

* 显示x轴信息`xlabel`
* 显示y轴信息`ylabel`
* 显示标题`title`

``` python
# 显示图形信息
plt.xlabel("月份")
plt.ylabel("个数")
plt.title("女朋友")
```

![](.\Python\Matplotlib\图形信息.png)



## 绘制多个图形

多次调用`plot`

``` python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 字体属性
font = { 
    'family': 'SimSun',
    'weight': 'bold',
    "size": '16'
}

# 确定横纵坐标
x = range(1, 13)
y1 = [random.randint(1, 10) for i in x]
y2 = [random.randint(1, 10) for i in x]

#改变图形大小
plt.figure(figsize = (20, 8), dpi = 80)

# 显示中文
plt.rc("font", **font)

# 显示网格
plt.grid(alpha = 0.4)

# 显示图形信息
plt.xlabel("月份")
plt.ylabel("个数")
plt.title("女朋友")

# 绘制图形
plt.plot(x, y1)
plt.plot(x, y2)

# 修改横坐标
_xticks_label = ["{}月".format(i) for i in x]
plt.xticks(x, _xticks_label)


# 展示图形
plt.show()
```

![](.\Python\Matplotlib\多个图形.png)



### 添加图例

使用`legend`，注意设置字体的参数为`prop`，并且注意代码的顺序

```python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 字体属性
font = { 
    'family': 'SimSun',
    'weight': 'bold',
    "size": '16'
}

# 确定横纵坐标
x = range(1, 13)
y1 = [random.randint(1, 10) for i in x]
y2 = [random.randint(1, 10) for i in x]

#改变图形大小
plt.figure(figsize = (20, 8), dpi = 80)

# 显示中文
plt.rc("font", **font)

# 显示网格
plt.grid(alpha = 0.4)

# 显示图形信息
plt.xlabel("月份")
plt.ylabel("个数")
plt.title("女朋友")

# 绘制图形，先绘制图形，在添加图例
plt.plot(x, y1, label = "我")
plt.plot(x, y2, label = "你")

# 添加图例
plt.legend(prop = font)

# 修改横坐标
_xticks_label = ["{}月".format(i) for i in x]
plt.xticks(x, _xticks_label)


# 展示图形
plt.show()
```

![](.\Python\Matplotlib\图例.png)



### 改变风格

* 设置线条颜色，在`plot`中设置`color`属性
* 设置线条风格，在`plot`中设置`linestyle`属性
* 设置线条粗细，在`plot`中设置`linewidth`属性
* 设置线条透明，在`plot`中设置`alpha`属性

``` python
# 绘制图形
plt.plot(x, y1, label = "我", color = "cyan")
plt.plot(x, y2, label = "你", linestyle = ":")
```

> 在网格中同样可以设置





## 绘制点的坐标

``` python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import random

# 字体属性
font = { 
    'family': 'SimSun',
    'weight': 'bold',
    "size": '16'
}

# 确定横纵坐标
x = range(1, 13)
y1 = [random.randint(1, 10) for i in x]
y2 = [random.randint(1, 10) for i in x]

#改变图形大小
plt.figure(figsize = (20, 8), dpi = 80)

# 显示中文
plt.rc("font", **font)

# 显示网格
plt.grid(alpha = 0.4)

# 显示图形信息
plt.xlabel("月份")
plt.ylabel("个数")
plt.title("女朋友")

# 绘制图形
plt.plot(x, y1, label = "我", color = "cyan", markerfacecolor='blue',marker='o')
plt.plot(x, y2, label = "你", linestyle = ":")

# 显示点的数据
for a, b in zip(x, y1):
    plt.text(a, b, b, fontsize = '16', ha='center', va='bottom') 

# 添加图例
plt.legend(prop = font)

# 修改横坐标
_xticks_label = ["{}月".format(i) for i in x]
plt.xticks(x, _xticks_label)


# 展示图形
plt.show()
```



![](.\Python\Matplotlib\绘制点的坐标.png)
# py f-string


> 本文件由 Jupyter Notebook 导出。

## 为什么要使用 f-string

一个显而易见的理由是性能更高。下面提供基准测试，在我的服务器（腾讯云2c4g）上运行，可以很明显的看到快了一倍以上：


```python
import random
import time
t=time.time()
def foo(a):
    return f"head{a}mid{a}tail"

def bar(a):
    return "head{}mid{}tail".format(a,a)
        

for i in range(1000000):
    a=random.random()
    s=foo(a)
print(s)
print("f-string",time.time()-t)


for i in range(1000000):
    a=random.random()
    s=bar(a)
print(s)
print("str.format",time.time()-t)

```

    head0.6232855129801236mid0.6232855129801236tail
    f-string 1.6648733615875244
    head0.7331884929353395mid0.7331884929353395tail
    str.format 3.3033506870269775


让我们看一看反汇编的字节码。乍看起来的话 似乎`str.format`会更加快一些。使用 f-string 的 cost 是 11，而 str.format 只要 3.
但是别忘了，下面的字节码中有一项：`CALL_METHOD`。
由于 str.format 并非基于 py ，在 builtins 中只能看到 pass ，所以我们的探究只能到此为止了，但是考虑到额外的创建字符串对象的开销，我们可以基本认为是这一点导致的。


```python
from dis import dis

dis(foo)
print("---")
dis(bar)
```

      5           0 LOAD_CONST               1 ('head')
                  2 LOAD_FAST                0 (a)
                  4 FORMAT_VALUE             0
                  6 LOAD_CONST               2 ('mid')
                  8 LOAD_FAST                0 (a)
                 10 FORMAT_VALUE             0
                 12 LOAD_CONST               3 ('tail')
                 14 BUILD_STRING             5
                 16 RETURN_VALUE
    ---
      8           0 LOAD_CONST               1 ('head{}mid{}tail')
                  2 LOAD_METHOD              0 (format)
                  4 LOAD_FAST                0 (a)
                  6 LOAD_FAST                0 (a)
                  8 CALL_METHOD              2
                 10 RETURN_VALUE


## f-string 何时出现

f-string 最早出现于 py3.6；在 3.7 添加了对 await/async 的支持；3.8 添加了 等号表达式的支持。

除此之外的情况和 str.format() 在协议层面兼容。

## f-string 如何使用

一个 f-string 由f开头的一对引号定义，引号中可以包括以下内容，数量不做限制：

- 字面值（不含大括号的任何东西）；
- 双大括号，用于转义双大括号；
- 替代域（replacement_field），在求值的时候被被替换为一个实际的值。

在 python3 的 [词法分析](https://docs.python.org/zh-cn/3/library/string.html) 中，替代域的定义如下：

1. 一个前导大括号
2. 表达式
3. 可选的等号
4. 可选的转换符（conversion），用感叹号开头，接 “sra” 中的一个字母。
5. 可选的格式规格（literal_char），用冒号开头，可以由任意字符构成。
    - 注意：字面格式规格也可以是一个替代域，至少词法分析层面不禁止这一点。
    - 这可以在获取 f-string 常量的时候动态获取结果。
    - 如果不指定转换符的话，那么整个替代域的结果是表达式调用 __format__() 方法的结果，类设计者有充分的自由来决定结果是什么。
6. 结尾的大括号

注意由于表达式里不可能有反斜杠，因此整个替代域都不能有反斜杠。


### 转换符

众所周知，py的表达式有三种输出方式：字符串，终端和 ASCII 格式，分辨对应 __str__(),__repr__() 和 __ascii__() 方法。
转换符指定表达式使用哪一种输出方式。默认方式由表达式的 __format__() 定义。

举例来说数据库查询的时候要输出的一些信息应该是带引号的，如果在大括号前面加引号很不优雅，所以你可以考虑用 __repr__() 来直接输出带引号的字符串。这是 ref#1 里给出的一种解释，我个人认为还是不要手写数据库查询语句的好，使用 peewee 之类的 orm 上手也很快，如果你开发 orm 可以考虑用这个。

官方给的例子是输出人名，个人认为这个例子确实好一些。


```python
name="Deuterium"
print(f"My name is {name!r}.")
```

    My name is 'Deuterium'.


注意：指定了转换符之后会使得表达式变成字符串，因此后面格式规则里只针对数字的部分会直接报错。
另外，由于这几个双下方法 py 都有限制只能返回字符串，因此你也不能重载这个结果。

### 格式规则

前面提到过，没有指定转换符的情况下，使用表达式自己的 __format__() 方法来定义如何格式化，因此格式规则只是一个简单的字符串。

我们只介绍几个内置类和 datetime 的实现。

先看几个内置类（str,int,float,demical）里的 __format__() 是怎么实现的：

这个部分比较复杂，参见官方文档的[这一节](https://docs.python.org/zh-cn/3/library/string.html#format-specification-mini-language) 。
格式规则包括以下部分，所有部分都是可选的，但是要按照顺序：

1. 对齐，可选 “<>^=”，分别代表左，右，居中，两端。
    - 如果没有指定字段最小宽度，那么参数无效；
    - 默认数字是右对齐，而其他都是左对齐；
    - 可以在前面添加一个填充字符，默认是空格。
    - 两端对齐只对数字有效，会将符号位放在开头，从符号位之后才开始用填充字符填补。
        - 如果填充字符是 “0”，那么默认使用两端对齐。
2. 符号，只对数字有效。可选“-+ ”，默认符号代表啥都不加。否则的话正数在前面补一个＋或者空格。
3. 井号，官方讲的比较含糊，也没有示例，大意是如果后面要求输出某些数字的话用这个来改变格式。
    - 浮点数的情况，强制输出空格；
    - 进制数的情况，强制输出前缀进制表达（例如二进制 0b ，八进制 0o 十六进制 0x 或者 0X ）
    - 
4. 宽度，即最大长度，是一个整数。如果宽度有前导 0 ，那么相当于设置对齐字符是 0 而且采用两端对齐。
5. 分隔符，可选 “,_”，也就是千位分隔符，可以选择常用的逗号或者 py 自己为了防止语义冲突的下划线。
6. 精度，跟在一个小数点之后的整数，即小数点之后几位。
    - 对于字符串，精度限制能够输出几位；
    - 对于定点/科学，精度限制小数点后有几位；
        - 对于表达式是浮点（float）的情况下默认六位；
        - 对十进制小数（demical）默认不做限制；
    - 对于浮点，精度限制总共有几位；
    - 用于整数会报错；
7. 类型，一个单字符。
    - b 二进制整数；
    - c 字符；
    - d 十进制整数，整数默认类型；
    - o 八进制整数；
    - x|X 十六进制整数，X应用大写；
    - n 自动分隔符十进制整数/小数，“会使用当前区域设置来插入适当的数字分隔字符。”
        - 这个东西我没整明白
    - e|E 科学计数法，E 代表大写；
    - f|F 普通小数，如果没有小数部分不写小数点，除非前面有井号
    - g|G 科学/定点智能：
        - 如果数字特别大（指数超过精度）或者特别小(小于 1e6)就用科学；
        - 否则用普通小数，有一些边界条件比较复杂，不赘述。；
    - % 百分比，数字乘以 100 然后最后加一个百分比
    - s 字符串


```python
print("类型")
s=48000
print(f"char:{s:c}")
print(f"HEX:{s:X}")
print(f"float:{s:f}")
print(f"num:{s:n}")
print(f"sci:{s:e}")
```

    类型
    char:뮀
    HEX:BB80
    float:48000.000000
    num:48000
    sci:4.800000e+04



```python
print("精度")
i,f,s=4800,0.1+0.2,"48"
print(f"str:{s:.1}")
print(f"float 1:{f:.1}")
print(f"float20:{f:.20}")
try:
    print(f"int:{i:.1}")# 报错
except ValueError:
    print("不能那么做的，不能的呀！")
```

    精度
    str:4
    float 1:0.3
    float20:0.30000000000000004441
    不能那么做的，不能的呀！



```python
print("宽度")
i,f,s=4800,0.1+0.2,"4800"
print(f"str:{s:10}")
print(f"float:{f:25.20}")
print(f"float:{f:025.20}")
print(f"int:{i:10}")
print(f"int:{i:010}")
```

    宽度
    str:4800      
    float:   0.30000000000000004441
    float:0000.30000000000000004441
    int:      4800
    int:0000004800



```python
print("分隔符")
i,f,s=480000,0.1+0.2,"4800"
print(f"str:{s:10}")
print(f"float:{f:25.20}")
print(f"float:{f:025.20}")
print(f"int:{i:10}")
print(f"int:{i:010}")
```

    分隔符
    str:4800      
    float:   0.30000000000000004441
    float:0000.30000000000000004441
    int:    480000
    int:0000480000


### datetime 的格式化

接下来观察 datetime 的实现：使用百分号+字符来实现某种占位符，和 time.strptime() 一致。
但是，要使用后者的方式的话，更加繁琐而且不得不注意引号问题（双引号 f-string 内部不能出现双引号，单引号也是）
我们能够意识到，py 在这一点上给了开发者多么充分的自由。


```python
import datetime
 
dt = datetime.datetime(1995, 7, 5, 13, 30, 45, 100000)

print(f"{dt:%F}, {dt:%D}")  # 1995-07-05, 07/05/95
print(f"{dt.strftime('%F')}, {dt.strftime('%D')}")

```

    1995-07-05, 07/05/95
    1995-07-05, 07/05/95


更多的我觉得可以不用赘述了，我直接贴 ref#1 的代码看看能做到什么事情吧。


```python
import datetime
 
dt = datetime.datetime(1995, 7, 5, 13, 30, 45, 100000)

# %X: 返回时间,精确到秒(小数点后面的会截断)。这里注意X要大写，如果是%x那么等价于%D
print(f"{dt:%X}")  # 13:30:45
 
# 所以返回字符串格式的完整日期就可以这么写
print(f"{dt:%F} {dt:%X}")  # 1995-07-05 13:30:45
 
# %Y: 返回年(四位) %y: 返回年(两位)
print(f"{dt:%Y}, {dt:%y}")  # 1995, 95
 
# %m: 返回月 %d: 返回天 注意：会占满两位，不够补0
print(f"{dt:%m}, {dt:%d}")  # 07, 05
 
# %H: 返回小时(24小时制度) %I: 返回小时(12小时制度) 注意：会占满两位，不够补0
print(f"{dt:%H}, {dt:%I}")  # 13, 01
 
# %M: 返回分钟 %S: 返回秒 注意：会占满两位，不够补0
print(f"{dt:%M}, {dt:%S}")  # 30, 45
 
# %f: 返回微妙 注意：会占满六位，不够补0
print(f"{dt:%f}")  # 100000
 
# %p: 本地早上还是下午,早上返回AM、下午返回PM
print(f"{dt:%p}")  # PM
 
# %j: 一年中的第几天,从1开始(1月1号就是1) 注意：会占满三位，不够补0
print(f"{dt:%j}")  # 186
 
# %w: 星期几(0是周日、6是周六) %u: 星期几(1是周一、7是周日)
# 可以看到两种格式只有星期天不一样
print(f"{dt:%w}, {dt:%u}")  # 3, 3
 
# %U: 一年中的第几周(以全年首个周日所在的星期为第0周，占满两位，不够补0)
# %W: 一年中的第几周(以全年首个周一所在的星期为第1周，占满两位，不够补0)
# %V: 一年中的第几周(以全年首个包含1月4日的星期为第1周，以 0 补足两位)
print(f"{dt:%U}, {dt:%W}, {dt:%V}")  # 27, 27, 27
"""
所以如果对应的年的第一天恰好是星期一，那么%U会比%W少1。
如果不是星期一，那么两者是相等的
"""
# 比如2007年的1月1号恰好是星期一
dt = datetime.datetime(2007, 10, 13)
print(f"{dt:%U}, {dt:%W}, {dt:%V}")  # 40, 41, 41
 
# %Z: 返回时区名，如果没有则返回空字符串
print(f"'{dt:%Z}'")  # ''
 
# 这里面的符号还可以连用
print(f"{dt:%F %X}")  # 2007-10-13 00:00:00
print(f"{dt:%F %X %y %Y %m}")  # 2007-10-13 00:00:00 07 2007 10
```

    13:30:45
    1995-07-05 13:30:45
    1995, 95
    07, 05
    13, 01
    30, 45
    100000
    PM
    186
    3, 3
    27, 27, 27
    40, 41, 41
    ''
    2007-10-13 00:00:00
    2007-10-13 00:00:00 07 2007 10


## reference

- [https://www.cnblogs.com/traditional/p/9217594.html](https://www.cnblogs.com/traditional/p/9217594.html)
- [https://docs.python.org/zh-cn/3/library/string.html#format-string-syntax](https://docs.python.org/zh-cn/3/library/string.html#format-string-syntax)
- [https://docs.python.org/zh-cn/3/reference/lexical_analysis.html#f-strings](https://docs.python.org/zh-cn/3/reference/lexical_analysis.html#f-strings)

# X分钟速成 Y

## 其中Y=She

She 是我自创的一种类似python的语言。

```python
# 使用井号作为注释

"""
    三引号可以作为多行字符串，但是如果没有被赋值会被解释器忽略，可以被当做注释
"""

# 一、原始数据类型和运算符

## 1. 数字

# 整数，分数，浮点数
# 分数会自动化简
3  # => 3
frac(6,3) # => 2
4.0 # => 4.0

# 算术没有什么出乎意料的
1 + 1  # => 2
8 - 1  # => 7
10 * 2  # => 20

# 尽可能不损失精度，优先考虑整数，分数，浮点数
35 / 5  # => 7
5 / 3  # => frac(5,3)
5.0 / 3 # => 1.6666666666666667

# 整数除法的结果都是向下取整
5 // 3     # => 1
5.0 // 3.0 # => 1.0 # 浮点数也可以
-5 // 3  # => -2
-5.0 // 3.0 # => -2.0

# 浮点数的运算结果也是浮点数
3 * 2.0 # => 6.0

# 模除
7 % 3 # => 1
7.1 % 3 # => 1.1

# x的y次方
2**4 # => 16

# 用括号决定优先级
(1 + 3) * 2  # => 8

# 布尔值
True
False
true # => True # 初始值，但是小写版本不是关键字
false # => False # 并且可以被修改其值

# 用not 取非
not True  # => False
not False  # => True

# 逻辑运算符，注意and和or都是小写
True and False # => False
False or True # => True
True & False # => False # 注意，单字符版本不进行短路
False | True # => True

# 整数也可以当作布尔值
0 and 2 # => 0
-5 or 0 # => -5
0 == False # => True
2 == True # => False
1 == True # => True

# 用==判断相等
1 == 1  # => True
2 == 1  # => False

# 用!=判断不等
1 != 1  # => False
2 != 1  # => True

# 比较大小
1 < 10  # => True
1 > 10  # => False
2 <= 2  # => True
2 >= 2  # => True

# 同向大小比较可以连起来，但是连等会报错
1 < 2 < 3  # => True
2 < 3 < 2  # => False
2 < 3 <= 2  # => False
1==1==1 # 报错
1 < 2 > 5 # 报错

## 2. 字符串

# 字符串用双引号
"这是个字符串"

# 单引号字符串不会转义
'这也是个\t字符串'
'这是\一个\路径\' # 不同于py，单引号字符串允许结尾反斜杠


# 用加号连接字符串
"Hello " + "world!"  # => "Hello world!"

# 不同于 py ，临接字符串不能自动拼接
"str1" "str2" # 报错

# 字符串可以被当作字符列表
"This is a string"[0]  # => 'T'

# 用.format来格式化字符串
"{} can be {}".format("strings", "interpolated")

# 可以重复参数以节省时间
"{0} be nimble, {0} be quick, {0} jump over the {1}".format("Jack", "candle stick")
# => "Jack be nimble, Jack be quick, Jack jump over the candle stick"

# 如果不想数参数，可以用关键字
"{name} wants to eat {food}".format(name="Bob", food="lasagna") 
# => "Bob wants to eat lasagna"

# f-string 语法糖，用法和py一致
f"{"strings"} can be {"interpolated"}"

## 3. 空值

# None是一个对象，未定义的变量返回None
None  # => None
unfuncined_var # => None

# 当与None进行比较时不要用 ==，要用is。is是用来比较两个变量是否指向同一个对象。
"etc" is None  # => False
None is None  # => True
unfuncined_var is None # => True

# None，0，空字符串，空列表，空字典和其它各种空序列算是False
# 所有其他值都是True
bool(0)  # => False
bool("")  # => False
bool([]) # => False
bool({}) # => False

'''
bool() 首先试图获取对象的 __bool__() 方法，然后试图获取 __empty__() 并取反。
'''


# 二、 变量与序列类型

## 1. 变量

# 在给变量赋值前不用提前声明
# 传统的变量命名是小写，用下划线分隔单词
# 赋值变量会返回值
some_var = 5# => 5
some_var  # => 5

# del 删除变量
del somevar

# 冒号可以用于指定类型
int_var:int = 5 # => 5
int_var = 5.0 # =>5.0 # IDE警告

# 自动获取类型
auto_var := "str"
auto_var = 123 # => 123 # IDE警告

## 2. 序列，具名序列和列表

# 序列即py的tuple，用于一次性打包若干数据
seq = 1,2,3
seq[0]   # => 1
seq[0] = 3  # 抛出TypeError

# 列表和py一致。
lst = ['1', '2', '3']

# 具名序列类似namedtuple
named_seq = n[name = 'xiaomin',id = '001']
named_seq.name # => "xiaomin"

和py基本一样
len(seq)   # => 3
seq + (4, 5, 6)   # => (1, 2, 3, 4, 5, 6)
seq[:2]   # => (1, 2)
2 in seq   # => True

# 打包解包
a, b, c = 1, 2, 3     # 现在a是1，b是2，c是3
a = 1,2,3
b,c,d = *a # => b=1,c=2
b,*c = 1,2,3 # =>b=1,c=2,3

# 语法糖
seq = 1,2,3,4
seq.first() # => 1
seq.last() # => 4
seq.most() # => 1,2,3
seq.least() # => 2,3,4

## 3.高维处理
# 切片
high_seq = (1,2,3,4),
           (5,6,7,8),
           (9,10,11,12)
high_seq[None,3] # => 4,8,12

# 转置
print(*high_seq.transpose())
# => (1,5,9),(2,6,10),(3,7,11),(4,8,12)
# 添加参数可以实现多层转置
# 默认返回迭代器

## 哈希表：映射和集合
# 映射和dict等价，集合和set等价
s{} # 直接创建空集合，比set()更快

# 默认映射
map = {}
map["not_exist_key"] # => None
map.funcault("funcault value")
map['another'] # =>"funcault value"

# 三、流程控制

some_var = 5

## 分支

# 注意分号是有意义的
if some_var == 5:
    print("var=5");
elif some_var >5:
    print("sth")
    print("sth2");
else:; 
# => var=5
# 简而言之，冒号等于前半拉大括号，分号等于后半拉

# 这么写是可以但不建议的：
if true print(True)

# for和py略有不同
animal for "dog", "cat", "mouse":
    print("animal");
# ~=> "dog\ncat\nmouse\n"

# while,try-except,continue-break和py一致

# 四、函数

# 使用func而非func定义
# 需要用分号来结束块
func foo():
    ;

func add(x,y):
    return x+y;

# 我们可以定义一个可变参数函数
func varargs(*args):
    return args

varargs(1, 2, 3)   # => (1, 2, 3)


# 我们也可以定义一个关键字可变参数函数
func keyword_args(**kwargs):
    return kwargs

# 我们来看看结果是什么：
keyword_args(big="foot", loch="ness")   # => {"big": "foot", "loch": "ness"}


# 这两种可变参数可以混着用
func all_the_args(*args, **kwargs):
    print(args)
    print(kwargs)
"""
all_the_args(1, 2, a=3, b=4) prints:
    (1, 2)
    {"a": 3, "b": 4}
"""

# 调用可变参数函数时可以做跟上面相反的，用*展开序列，用**展开字典。
args = (1, 2, 3, 4)
kwargs = {"a": 3, "b": 4}
all_the_args(*args)   # 相当于 all_the_args(1, 2, 3, 4)
all_the_args(**kwargs)   # 相当于 all_the_args(a=3, b=4)
all_the_args(*args, **kwargs)   # 相当于 all_the_args(1, 2, 3, 4, a=3, b=4)


# 函数作用域
x = 5

func setX(num):
    # 局部作用域的x和全局域的x是不同的
    x = num # => 43
    print (x) # => 43

func setGlobalX(num):
    global x
    print (x) # => 5
    x = num # 现在全局域的x被赋值
    print (x) # => 6

setX(43)
setGlobalX(6)


# 函数在Python是一等公民
func create_adder(x):
    func adder(y):
        return x + y
    return adder

add_10 = create_adder(10)
add_10(3)   # => 13

# 匿名函数不用lambda，而是比较类似javascript
(x -> x > 2)(3) 

# 推导式
# 和原py略有不同，主要是for的用法变化
[add_10() for [1, 2, 3]]  # => [11, 12, 13]
[(x -> x) for [3, 4, 5, 6, 7] if x > 5]   # => [6, 7]

# 五、类
# 定义一个继承object的类
# 关键字为cls而不是class
cls Human(object):

    # 类属性，被所有此类的实例共用。
    species = "H. sapiens"

    # 构造方法，当实例被初始化时被调用。
    # 注意名字前后的双下划线，这表明这个属性或方法对Python有特殊意义
    # 你可以自行定义，但是你自己设计的类的方法不应该在前后加下划线。
    # 参数里的self可以省略。
    func __init__(name):
        # Assign the argument to the instance's name attribute
        self.name = name;

    # 实例方法，第一个参数总是self，就是这个实例对象
    func say(self, msg):
        return "{name}: {message}".format(name=self.name, message=msg);

    # 类方法，被所有此类的实例共用。第一个参数是这个类对象。
    @classmethod
    func get_species():
        return cls.species;

    # 静态方法。调用时没有实例或类的绑定。
    @staticmethod
    func grunt():
        return "*grunt*";
;

# 构造一个实例
i = Human(name="Ian")
print(i.say("hi"))     # 印出 "Ian: hi"

j = Human("Joel")
print(j.say("hello"))  # 印出 "Joel: hello"

# 调用一个类方法
i.get_species()   # => "H. sapiens"

# 改一个共用的类属性
Human.species = "H. neanderthalensis"
i.get_species()   # => "H. neanderthalensis"
j.get_species()   # => "H. neanderthalensis"

# 调用静态方法
Human.grunt()   # => "*grunt*"

## 重载
func equalQ(num1:int,num2:int):
    return num1 == num2;

func equalQ(num1:float,num2:float,eps=1e-7):
    return abs(num1-num2)<eps;

equalQ(1,2) # =>False
equalQ(0.1+0.2,0.3) # =>True



####################################################
## 6. 模块
####################################################

# 用import导入模块
import math
print(math.sqrt(16))  # => 4.0

# 也可以从模块中导入个别值
# 注意和py不同，import和from的顺序互换了
# from math import ceil, floor
import ceil, floor from math 
print(ceil(3.7))  # => 4.0
print(floor(3.7))   # => 3.0

# 可以导入一个模块中所有值
# 警告：不建议这么做
# from math import *
import * from math 

# 如此缩写模块名字
import math as m
math.sqrt(16) == m.sqrt(16)   # => True

# Python模块其实就是普通的Python文件。你可以自己写，然后导入，
# 模块的名字就是文件的名字。

# 你可以这样列出一个模块里所有的值
import math
dir(math)


####################################################
## 7. 高级用法
####################################################

# 用生成器(generators)方便地写惰性运算
func double_numbers(iterable):
    for i in iterable:
        yield i + i
    # 语法糖
    yield i+i from iterable

# 生成器只有在需要时才计算下一个值。它们每一次循环只生成一个值，而不是把所有的
# 值全部算好。
#
# range的返回值也是一个生成器，不然一个1到900000000的列表会花很多时间和内存。
#
# 如果你想用一个Python的关键字当作变量名，可以加一个下划线来区分。
range_ = range(1, 900000000)
# 当找到一个 >=30 的结果就会停
# 这意味着 `double_numbers` 不会生成大于30的数。
for i in double_numbers(range_):
    print(i)
    if i >= 30:
        break


# 装饰器(decorators)
# 我不喜欢现在py的装饰器系统，但我还没想好怎么改
```

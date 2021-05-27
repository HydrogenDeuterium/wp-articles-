#! https://zhuanlan.zhihu.com/p/374597772
# 使用 peewee 在 python 中操作数据库

> 本文使用 mysql 作为演示的数据库类型，关于 peewee 支持的其它数据库，请参见官方文档

## 为什么要使用 peewee ？

在不使用 peewee 之类 ORM 工具的情况下，操作数据库大约是以下情况：

```python
import pymysql # 以下省略 import 语句

db = pymysql.connect(user=xxx, password=xxx, 
                    host=xxx, database=xxx)
cursor = db.cursor() # 以下省略生成游标的步骤
cursor.execute('xxxxx')
# 下略
```

其中的 xxxxx 对应的 SQL 语句通常需要自行拼接得到，这是一个风险较大的行为。

待拼接的查询语句很难保证不出现问题，因此也出现了例如 SQL 注入之类的种种问题。

```
str='*; -- DROP DATABASE *;'
SQL = f"SELECT {str} where id = 0"
cursor.execute(SQL) # 命令执行后数据库就没了
```

因此，我们需要一些工具来对于数据库查询语句给出更加强力的约束，以防止出现这样意料之外的执行结果。peewee 正是一个完成这个项目的包。

## 如何使用peewee

### 安装

安装很简单，直接 `pip install peewee` 就行。

本文不拟解答使用其它包管理工具（如 conda ）和源码安装等安装方式的问题。

### 定义models

首先认识到 peewee 是如何对数据库查询作更加严格限制的，它要求对于待操作的数据库各表信息已知。

这些已知信息是以如下的 model.py 文件的形式呈现的：

```python
from  import * # 以下省略导入

db = MySQLDatabase('users', **{
        xxxyyy  # 各种数据库连接信息，不展开
        })

class Notes(Model):
    created_at = DateTimeField(constraints=[SQL("DEFAULT CURRENT_TIMESTAMP")], null=True)
    updated_at = DateTimeField(constraints=[SQL("DEFAULT CURRENT_TIMESTAMP")], null=True)
    deleted_at = DateTimeField(index=True, null=True)
    name = CharField()
    data = CharField()
    
    # 设置表名，数据库等
    class Meta:
        table_name = 'notes'
        database = db

```

Notes 中定义的变量保存表中的各字段类型、限制等信息，而 Meta 类存放表本身的信息。

一般来说这些信息不需要手写，使用 `python -m pwiz -e 数据库类型 -H IP地址 -p 端口号 -u 用户名 -P 数据库名称 >model.py` 可以一键生成，使用时直接 `import model.py` 即可。

剩下的使用就直接调用表类就可以（不用实例化），具体使用见下，就直接看代码好了。

```python
# 建表
if not db.table_exists('notes'):
    db.create_tables([Notes])

# Create
Notes(name='peewee3', data='try to create data from peewee3').save()
Notes.create(name='peewee2', data='try to create data from peewee again3')
note = Notes()
note.name = 'peewee3'
note.data = 'another way to create3'
note.save()
note.insert([
        {'name': 'peewee2', 'data': 'insert data#0 '},
        # 下面两种等效
        {'name': 'peewee2', Notes.data: 'insert data#1 '},
        {Notes.name: 'peewee2', 'data': 'insert data#2 '}
        ]).execute()
note.insert_many([
        ['peewee2', 'insert many data '],
        ['peewee2', 'insert many data#1 '],
        ['peewee2', 'insert many data#2 ']],
        fields=(Notes.name, 'data')).execute()

try:
    note.insert({'id': 164, 'name': '1', 'data': 'it will not rollback of id 11'}).execute()
    note.insert({'id': 164, 'name': '2', 'data': 'it will rollback of id 11 #1'}).execute()
except peewee.IntegrityError:
    # 报错了记得回滚
    db.rollback()
```

看看怎么读数据（select）：
```python
# Read
# ret 的 __str__ 方法默认实现是获取主键值
print(ret := Notes.get(), ret.name, ret.data)
# 从主键获取对象
print(ret := Notes.get_by_id(164), ret.name, ret.data)
# 不过因为重写了 __getitem__ ，所以一般这么写更加合理
print(ret := Notes[164], ret.name, ret.data)
# 不存在的主键报 DoesNotExist 错

# 下面讲获取多条
select = Notes.select()
# 这里生成查询语句看一看
print(select.sql())
for res in select:
    print(f'{res.name=}\t{res.data=}')

# 最大数量，偏移与分页
# 这些方法实际只是在修改查询语句，实例化的时候才会执行
print('LIMIT')
print([i.name for i in Notes.select().limit(10)])
# 前面说了 LIMIT,这里讲一下 OFFSET，只有查找可以加offset，不然会报错：
print([i.name for i in Notes.select().limit(10).offset(1)])
# 一个语法糖：每页20个，第一页；
print([i.name for i in Notes.select().paginate(page=1, paginate_by=20)])

# 下面是实例化的内容
# 父类 BaseModelSelect 实现了 __iter__，所以可以遍历
for res in select:
    print(f'{res.name=}\t{res.data=}')
print(select[0])
# 可以转dict，tuple，namedtuple
print(list(Notes.select().where(Notes.name == 'dh').dicts()))

# Delete/UPDATE
# 全部删除 因为太恐怖所以就不执行了吧。。
# Notes.delete().execute()
# 删掉一个，差不多了吧。WHERE 选择器具体内容下面专门讲
Notes.delete().where(Notes.name.is_null()).limit('1').execute()
# 注意这里只有Notes定义过的才能修改（真棒！）
Notes.update(name='peewee4').where(Notes.name == 'peewee').execute()
# 调用returning()可以返回值，相当于附加了一层select
# insert,delete,update 都可以加这么一层装饰。
# 不指定返回参数，那么就给主键
res = Notes.update(name='peewee4').where(Notes.name == 'peewee').returning(Notes.name).execute()
print([_.name for _ in res])
```

下面我们详细讲一下 where 选择器：

```python
# isnull 默认True 可以不写
print('IS [NOT] NULL')
for i in Notes.select().where(Notes.deleted_at.is_null(False)):
    print(i.name, i.data)

# 属于
print("IN")
print([i.name for i in Notes.select().where(Notes.name.in_('dh', '123'))])

# 注意 sql 的 BETWEEN 两头都是闭区间
print('BETWEEN')
print([i.name for i in Notes.select().where(Notes.id.between(40, 59))])
# 实际上字段都重载了__getitem__ 方法，所以可以直接传slice对象，更好的方法是下面这种：
print([i.name for i in Notes.select().where(Notes.id[slice(10, 59)])])
print([i.name for i in Notes.select().where(Notes.id[10, 59])])

# 重载了 __mod__ 方法来实现 LIKE 功能
print('%peewee%')
print([i.name for i in Notes.select().where(Notes.name % '%peewee%')])
# 还可以写startswith(),endswith()，contains()
print([i.name for i in Notes.select().where(Notes.name.contains('peewee'))])

# regexp pattern,如果用 iregexp() 就是大小写不敏感
print('REGEXP')
print([i.name for i in Notes.select().where(Notes.name.regexp('^peewee[23]'))])

# 最后 多个条件，用 ~ ＆ | 连接，注意运算符顺序
# 这个‘~’有没有想起lua？嘿嘿
print('多个条件')
print([(i.id, i.name, i.data) for i in Notes.select().where((Notes.name == 'dh') & (Notes.data == '456346'))])

# 除此之外，还可以通过 fn() 实现各种 sql 方言的各种神绮操作，这里不赘述了。
```

参考：
https://www.cnblogs.com/traditional/p/11326736.html

## 1. 基础

### 1.1 变量

#### 原则

- 遵循PEP8:

  - 普通变量、函数名使用蛇形命名法，如`max_value`
  - 类名使用驼峰风格，如`FooClass`

  - 常量采用全大写字母，使用下划线连接，如`MAX_VALUE`

  - 仅内部使用的变量增加下划线前缀，如`_local_var`

- 名字与python关键字冲突时，在变量末尾追加下划线，比如`class_`

- 尽可能短的同时描述性要强，如待处理的临时数据命名为`pending_id`比用`temp`好

- 要匹配类型以提高可读性：

  - bool类型的值可用is、has来修饰，如`has_errors`

  - int/float类型的值可直接用释义为数字的词，或用id、length、count等来修饰，如`port`、`users_count`

#### 使用

```python
x, y, z = 1, 2, 3  # 同时赋值

usernames = ['piglei', 'raymond']  # 解包
author, reader = usernames
print(author)
# piglei

attrs = [1, ['piglei', 100]]
user_id, (username, score) = attrs  # 解包多层嵌套
print(username)
# piglei

data = ['piglei', 'apple', 'orange', 'banana', 100]
username, *fruits, score = data  # 加*动态解包
print(fruits)
# ['apple', 'orange', 'banana']
```

#### 建议

- 保持变量两方面的一致性：
  - 名字一致性：同一个项目、模块或函数中，一类事物的称呼保持一致
  - 类型一致性：不把同一个变量重复指向不同类型的值
- 变量的定义尽量靠近其使用位置，以提高代码的可读性
- 将复杂的表达式赋值为一个临时变量，以提到代码可读性
- 同一作用域内不要有太多变量，同类变量可以适当分组建立新类进行管理
- 显示优于隐式，不要用`locals()`来直接获取作用域中的所有局部变量
- 写出一句有说服力的接口注释前，别写任何函数代码

### 1.2 数值与字符串

#### 使用

```python
# 三种内置数值类型
int_ = 100  # 整型
double_ = 37.2  # 浮点，双精度
complex_ = 1 + 2j  # 复数，以j为虚部后缀
inf_, ninf_ = float("inf"), float("-inf")  # 无穷大/小

from decimal import Decimal

print(0.1 + 0.2)
print(Decimal('0.1') + Decimal('0.2'))  # 用Decimal可实现精确的浮点运算
print(Decimal(0.1))  # Decimal中的数字必须用字符串表示，否则陷入“浮点数陷阱”
# 0.30000000000000004
# 0.3
# 0.1000000000000000055511151231257827021181583404541015625

universe_age = 14_000_000_000  # python3.6，下划线分割大数
print(universe_age)
# 14000000000

count = sum(i % 2 == 0 for i in range(0, 100))  # bool值可作为整型使用
print(count)
# 50

first_name = "ada"
last_name = "lovelcae"
for c in first_name:  # 遍历
    print(c)
print(last_name[1:3])  # 切片
print(last_name[::2])  # 设置step
print(last_name[::-2])  # step为负数，表示逆序
print(','.join(reversed(last_name)))  # join把可迭代对象的元素与字符连接为字符串
# a
# d
# a
# ov
# lvla
# eceo
# e,a,c,l,e,v,o,l

# 格式化
full_name = "%s %s" % (first_name, last_name)  # C风格
full_name = "{} {}".format(first_name, last_name)  # str.format
full_name = f"{first_name} {last_name}"  # python3.6，f-string

print(full_name.title())  # 首字母大写
print(full_name.isdigit())  # 数字判断
# Ada Lovelcae
# False

favorite_language = ' python '
print(favorite_language.lstrip())  # 删除空白
print(favorite_language.rstrip())
print(favorite_language.strip())
# python 
#  python
# python

title = "Alice in Wonderland"
print(title.split())  # 切分
print(title.rsplit(None, maxsplit=1))  # 反向切分
print(title.partition(' '))  # 返回三元元组：(符前字符，分隔符，符后字符)
# ['Alice', 'in', 'Wonderland']
# ['Alice in', 'Wonderland']
# ('Alice', ' ', 'in Wonderland')

table = title.maketrans(' ', ',')  # 创建规则
print(title.translate(table))  # 按规则一次性替换
# Alice,in,Wonderland

str_obj = "Hello, 世界。"
print(type(str_obj))  # str
# <class 'str'>

bin_obj = str_obj.encode()  # 字节串，默认为UTF-8
print(bin_obj)
print(type(bin_obj))  # bytes
print(bin_obj.decode())  # 解码
# b'Hello, \xe4\xb8\x96\xe7\x95\x8c\xe3\x80\x82'
# <class 'bytes'>
# Hello, 世界。

bin = b'Hello'  # 创建字节串字面值

from enum import Enum # python3.4，枚举类型

class UserType(int, Enum):  
    VIP = 3
    BANNED = 13

print(UserType.VIP == 3)
# True

from jinja2 import Template  # 模板引擎

_MOVIES_TMPL = '''\
Welcome, {{username}}.
{%for name, rating in movies %}
* {{ name }}, Rating: {{ rating|default("[NOT RATED}", True) }}
{%- endfor %}
''' # 设置模板
def render_movies(username, movies):
    tmpl = Template(_MOVIES_TMPL)
    return tmpl.render(username=username, movies=movies)

movies = [
    ('The Shawshank Redemption', '9.3'),
    ('The Prestige', '8.5'),
    ('Mulan', None),
]
print(render_movies('piglei', movies))
# Welcome, piglei.
# * The Shawshank Redemption, Rating: 9.3
# * The Prestige, Rating: 8.5
# * Mulan, Rating: [NOT RATED}

long_str = ("This is the first line of a long string, "
            "this is the second line"
            ".")  # 超长字符串用括号包裹后可任意拆分行
print(long_str)
# This is the first line of a long string, this is the second line.

msg = """Welcome, today's movie list:
         - Jaw (1975)
         - The Shining (1980)
         - Saw (2004)"""
print(msg)
# Welcome, today's movie list:
#         - Jaw (1975)
#         - The Shining (1980)
#         - Saw (2004)

from textwrap import dedent  # 去掉代码缩进产生的空白
msg = dedent("""\
    Welcome, today's movie list:
    - Jaw (1975)
    - The Shining (1980)
    - Saw (2004)""")
print(msg)
# Welcome, today's movie list:
# - Jaw (1975)
# - The Shining (1980)
# - Saw (2004)
```

#### 建议

- 字面量表达式不必预计算（可用`dis`模块得到程序的字节码）
- 使用`+`进行字符串拼接的效率和`join`相当

### 1.3 条件控制分支

#### 使用

```python
import random

num = 1000 if random.randint(0, 20) > 10 else 100  # 三元表达式
print(num)
# 1000

print(bool(set()), bool([]))  # 为空的容器布尔值都是False
# False False

class Foo:
    pass
print(bool(Foo), bool(Foo()))  # 自定义类和实例布尔值都是True
# True True

class UserCollection:
    def __init__(self, users):
        self.items = users
    
    def __len__(self):  # 魔法方法重载了len()，同时也作为布尔判断的返回结果
        return len(self.items)
    
users = UserCollection(['piglei', 'raymond'])
print(len(users))
print(bool(UserCollection([])), bool(users))  # 以len的结果作为布尔结果
# 2
# False True

class ScoreJudger:
    def __init__(self, score):
        self.score = score
    
    def __bool__(self):  # 魔法方法重载了bool()，且在同时实现了len()时比后者优先级高
        return self.score >= 60

print(bool(ScoreJudger(60)), bool(ScoreJudger(59)))  
# True False

class EqualWithAnything:
    def __eq__(self, other):  # 魔法方法重载了==运算符
        return True

foo = EqualWithAnything()
print(foo == 'string')
# True

x = 6300
y = 6300
print(x is y)  # is会判断两个对象在内存中是否是相同
# False

x = 100
y = 100
print(x is y)  # 整型驻留，-5到256的小整数被缓存在一个数组中
# True

nums = [3, 4, 5, 6, -1]
print(all(n > 0 for n in nums))  # all(iter)：iter中所有成员为真时返回True
print(any(n > 0 for n in nums))  # any(iter)：iter中有成员为真时返回True
# False
# True

print(True or False and False)  # and优先级高于or
# True
```

#### 建议

- 判断语句里不要显示地进行布尔值比较、零值/空值判断
- 与`None`/`True`/`False`进行比较时应该用`is`运算符，其他情况则用`==`
- 多层嵌套一般都可以用“提前返回”来进行优化：找到会中断程序执行的条件，把它们前移，在分支里直接用`return`或`raise`结束执行

## 2. 容器

### 2.1 基础

- python的内置数据类型大致可分为：
  - 可变（mutable）：列表、字典、集合
  - 不可变（immutable）：整数、浮点数、字符串、字节串、元组
- python中进行函数调用传参时，传递的是“变量所指对象的引用”（pass-by-object-reference），最终产生的是**值传递**还是**引用传递**的效果，取决于所传递对象的可变性
- python中某种类型是否可哈希（hashable）遵顼以下规则：
  - 不可变的内置类型都是可哈希的
  - 可变的内置类型都不可哈希
  - 不可变的容器类型中，所有成员都不可变时，其本身才是可哈希的
  - 用户定义的类型默认可哈希

### 2.2 拷贝

```python
nums = [1, 2, 3, 4]
nums_copy = nums  # 可变对象直接赋值进行的是引用传递
nums[2] = 30
print(nums, nums_copy)
# [1, 2, 30, 4] [1, 2, 30, 4]

nums_copy2 = nums.copy()  # 进行浅拷贝
nums[2] = 40
print(nums, nums_copy2)
# [1, 2, 40, 4] [1, 2, 30, 4]

nums_copy3 = list(nums)  # 容器内置构造函数可实现浅拷贝
nums[2] = 50
print(nums, nums_copy3)
# [1, 2, 50, 4] [1, 2, 40, 4]

nums_copy4 = nums[:]  # 全切片可实现浅拷贝
nums[2] = 60
print(nums, nums_copy4)
# [1, 2, 60, 4] [1, 2, 50, 4]

d = {'foo': 1}
d2 = {key: value for key, value in d.items()}  # 推导式产生的是浅拷贝
d['foo'] = 2
print(d, d2)
# {'foo': 2} {'foo': 1}

import copy

items = [1, ['foo', 'bar'], 2, 3]
items_copy = copy.copy(items)  # copy模块下的copy方法也能实现浅拷贝
items[0] = 100
items[1].append('xxx')
print(items, items_copy)
# [100, ['foo', 'bar', 'xxx'], 2, 3] [1, ['foo', 'bar', 'xxx'], 2, 3]

items_copy2 = copy.deepcopy(items)  # 嵌套的容器需要进行深拷贝才能完全拷贝
items[0] = 200
items[1].append('yyy')
print(items, items_copy2)
# [200, ['foo', 'bar', 'xxx', 'yyy'], 2, 3] [100, ['foo', 'bar', 'xxx'], 2, 3]
```

### 2.3 列表

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[-1])  # 倒数第一个元素
# suzuki

motorcycles[0] = 'ducati'  # 修改
motorcycles.append('lucky')  # 添加
motorcycles.insert(1, 'honda')  # 插入
print(motorcycles)
# ['ducati', 'honda', 'yamaha', 'suzuki', 'lucky']

del motorcycles[0]  # 删除
popped_motorcycle = motorcycles.pop()  # 出栈
first_owned = motorcycles.pop(1)  # 可指定任意位置
print(motorcycles, popped_motorcycle, first_owned)
# ['honda', 'suzuki'] lucky yamaha

motorcycles.append('yamaha')
motorcycles.append('suzuki')
print(motorcycles)
# ['honda', 'suzuki', 'yamaha', 'suzuki']

motorcycles.remove('suzuki')  # 删除第一个指定值
print(motorcycles)
# ['honda', 'yamaha', 'suzuki']

motorcycles.sort(reverse=True)  # 永久排序，可指定正反序
print(motorcycles)
print(sorted(motorcycles))  # 临时排序
motorcycles.reverse()  # 反转顺序
print(motorcycles)
# ['yamaha', 'suzuki', 'honda']
# ['honda', 'suzuki', 'yamaha']
# ['honda', 'suzuki', 'yamaha']

print(motorcycles[0:2])  # 切片
print(motorcycles[1:])
print(motorcycles[:-1])
# ['honda', 'suzuki']
# ['suzuki', 'yamaha']
# ['honda', 'suzuki']

copy_motorcycles = motorcycles[:]  # 复制
new_motorcycles = motorcycles  # 引用
copy_motorcycles.append('ducati')
new_motorcycles.pop(0)
print(copy_motorcycles)
print(new_motorcycles)
print(motorcycles)
# ['honda', 'suzuki', 'yamaha', 'ducati']
# ['suzuki', 'yamaha']
# ['suzuki', 'yamaha']

for motor in motorcycles: # 列表循环
    print(f'I like {motor}.')
# I like suzuki.
# I like yamaha.

for index, motor in enumerate(motorcycles, start=2):  # 获取下标，可指定起始值
    print(f'{index}, {motor}.')
# 2, suzuki.
# 3, yamaha.

print(list('foo'))  # 可迭代对象可用list转换为列表
print(list(range(2, 11, 2)))  # 数字列表
# ['f', 'o', 'o']
# [2, 4, 6, 8, 10]

num_list = [1, 2, *range(3)]  # 单星号可解包任意可迭代对象
print(num_list)
l1 = [1, 2]
l2 = [3, 4]
print([*l1, *l2])  # 从而用来实现列表拼接
# [1, 2, 0, 1, 2]
# [1, 2, 3, 4]

squares = [value**2 for value in range(1, 11) if value % 2 == 0]  # 列表推导式
print(squares)
# [4, 16, 36, 64, 100]

a = [1,2,3]
b = [4,5,6,7,8]
zipped = zip(a, b)
print(list(zipped))  # zip将多个可迭代对象对于打包为元组，返回由这些元组组成的对象
a1, a2 = zip(*zip(a, b))  # zip(*)则为解压
print(list(a1), list(a2))
# [(1, 4), (2, 5), (3, 6)]
# [1, 2, 3] [4, 5, 6]
```

### 2.4 元组

```python
dimensions = (200, 50)  # 不可变的列表
print(dimensions[0])
# 200

for dimension in dimensions:
    print(dimension)
# 200
# 50

print(tuple('foo'))  # 可迭代对象可用tuple转换为元组
# ('f', 'o', 'o')

def get_rectangle():
    width = 100
    height = 20
    return width, height  # 多个结果时返回的是元组

result = get_rectangle()
print(result, type(result))
# (100, 20) <class 'tuple'>

user_info = ('piglei', 'MALE', 30, True)  # 元组中经常存放不同类型的数据

from collections import namedtuple  # 具名元组

Rectangle = namedtuple('Rectangle', ['width', 'height'])  # 指定元组的字段名称
rect = Rectangle(height=20, width=100)
print(rect.__doc__)
print(rect.width)  # 直接通过字段名访问
# Rectangle(width, height)
# 100

from typing import NamedTuple  # python3.6，用类型注解来定义具名元组

class Rectangle2(NamedTuple): 
    width: int  # 不会进行类型校验
    height: int

rect = Rectangle2(100, 20)
print(rect.height)
# 20
```

### 2.5 字典

```python
alien = {'color': 'green', 'points': 5}  # 键值对，底层用哈希表实现，python3.6后的字典是有序的

alien['color'] = 'yellow'  # 修改值
alien.update({'color': 'black'})
print(alien['color'])
# black

alien['x_position'] = 0  # 添加键值对
alien['y_position'] = 25
del alien['points']  # 删除键
print(alien)
# {'color': 'black', 'x_position': 0, 'y_position': 25}

point_value = alien.get('points', 'No point value assigned.')  # 访问，不存在时返回默认值
print(point_value)
print(alien.setdefault('points', 1))  # 键不存在时设置默认值
print(alien.setdefault('points', 0))  # 存在则返回对应值
print(alien.pop('points', "No point value assigned."))  # 删除键，不存在时返回默认值
print(alien)
# No point value assigned.
# 1
# 1
# 1
# {'color': 'black', 'x_position': 0, 'y_position': 25}

for key, value in alien.items():  # 遍历
    print(f"\nKey: {key}")
    print(f"Value: {value}")
print("\n")
# Key: color
# Value: black

# Key: x_position
# Value: 0

# Key: y_position
# Value: 25

for key in alien:  # 默认遍历键
    print(f"{key.title()}")
print("\n")
# Color
# X_Position
# Y_Position

for key in alien.keys():  # 遍历键
    print(f"{key}")
print("\n")
# color
# x_position
# y_position

for key in sorted(alien.keys(), reverse=True):  # 顺序遍历键
    print(f"{key}")
print("\n")
# y_position
# x_position
# color

for value in alien.values():  # 遍历值
    print(f"{value}")
# black
# 0
# 25

d1 = {'foo': 3}
d2 = {'bar': 4}
d3 = {**d1, **d2}  # 双星号运算符可解包字典，从而实现字典合并
#d3 = d1 | d2  # python3.9，可用|实现字典合并
print(d3)
d4 = {key: value * 10 for key, value in d3.items() if key == 'foo'}  # 字典推导式
print(d4)
# {'foo': 3, 'bar': 4}
# {'foo': 30}

from collections import OrderedDict  # python3.6之前，可由OrderedDict获得有序字典

d = OrderedDict()
d['FIRST_KEY'] = 1
d['SECOND_KEY'] = 2
for key in d:
    print(key)
# FIRST_KEY
# SECOND_KEY

d3 = {'name': 'piglei', 'fruit': 'apple'}
d4 = {'fruit': 'apple', 'name': 'piglei'}
print(d3 == d4)
d3 = OrderedDict(name='piglei', fruit='apple')  # 会把“键的顺序”作为对比条件
d4 = OrderedDict(fruit='apple', name='piglei')
print(d3 == d4)
# True
# False

nums = [10, 2, 3, 21, 10, 3]
print(list(OrderedDict.fromkeys(nums).keys()))  # 可用来去重
# [10, 2, 3, 21]
```

### 2.6 集合

```python
fruits = {'apple', 'orange', 'apple', 'pineapple'}  # 无序，值唯一的可变容器，底层用哈希表实现
print(fruits)
# {'pineapple', 'apple', 'orange'}

empty_set = set()  # 初始化空集合只能用set()，{}表示字典

nums = [1, 2, 2, 4, 1]
s1 = {n for n in nums if n < 3}  # 集合推导式
print(s1)
# {1, 2}

s1.add(3)  # 添加
s1.remove(2)  # 删除
print(s1)
# {1, 3}

fruits_ = {'tomato', 'orange', 'grapes', 'mango'}
print(fruits & fruits_)  # 交集运算
print(fruits | fruits_)  # 并集运算
print(fruits - fruits_)  # 差集运算
# {'orange'}
# {'tomato', 'orange', 'pineapple', 'grapes', 'apple', 'mango'}
# {'pineapple', 'apple'}

valid_set = {'apple', 30, 1.3, ('foo',)}  # 集合中只能存放可哈希的对象

f_set = frozenset(['foo', 'bar', 'foo'])  # 不可变集合
print(f_set)
# frozenset({'bar', 'foo'})
```

### 2.7 生成器

```python
def generate_even(max_number):
    for i in range(0, max_number):
        if i % 2 == 0:
            yield i  # yield可以逐步为调用方生成结果并返回

i = generate_even(10)
print(next(i))  # 通过next从生成器对象中获取结果
print(next(i))
# 0
# 2
```

### 2.8 建议

- 创建自定义容器时，继承`collections.abc`下的抽象类是更好的选择
- 列表使用数组作为底层来实现的，因此`insert`操作要比`append`慢很多
  - 要经常在头部插入数据时，应使用以双端队列为底层的`collections.deque`
- 集合以哈希表为底层来实现，判断容器中是否包含特定成员的操作要比列表快
- 函数返回多个值时，使用具名元组（`NamedTuple`）作为返回结果，可增加函数的可扩展性：
    ```python
    from typing import NamedTuple
    
    class Address(NamedTuple):
        country: str    
        province: str    
        city: str
    
    def latlon_to_address(lat, lon):
        ...
        return Address(
            country=country,        
            province=province,        
            city=city,    
        )
    
    addr = latlon_to_address(lat, lon)
    # addr.country / addr.province / addr.city
    ```

## 3. 函数

```python
def get_formatted_name(first_name, last_name, middle_name=''):  # 默认值
    if middle_name:
        full_name = f"{first_name} {middle_name} {last_name}"
    else:
        full_name = f"{first_name} {last_name}"
    return full_name.title()

from typing import List

print(get_formatted_name("eric", "matthes"))
print(get_formatted_name(last_name="jimi", first_name="hendrix"))
# Eric Matthes
# Hendrix Jimi


def greet_users(names: List[str]):  # python3.5，类型注解，不具备类型校验能力
    for name in names:
        msg = f"Hello, {name.title()}!"
        print(msg)

my_names = ["eric", "jimi"]
greet_users(my_names[:])  # 传递副本以阻止修改
# Hello, Eric!
# Hello, Jimi!

def make_pizza(*toppings):  # *：建立元组，使函数接受任意数量实参
    print(toppings)

make_pizza('pepperoni')
make_pizza('mushrooms', 'green peppers', 'extra cheese')
# ('pepperoni',)
# ('mushrooms', 'green peppers', 'extra cheese')

def build_profile(first, last, **user_info):  # **：建立字典，使函数接受任意数量的键值对
    user_info['first_name'] = first
    user_info['last_name'] = last
    return user_info

user_profile = build_profile(
    'albert', 'einstein', location='princeton', field='physics')
print(user_profile)
# {'location': 'princeton', 'field': 'physics', 'first_name': 'albert', 'last_name': 'einstein'}
```

## 4. 类

```python
class Car:
    def __init__(self, make, model, year):  # 构造函数，self是一个指向实例本身的引用
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0

    def get_descriptive_name(self):
        long_name = f"{self.year} {self.make} {self.model}"
        return long_name.title()

    def update_odometer(self, mileage):
        if mileage >= self.odometer_reading:
            self.odometer_reading = mileage
        else:
            print("You can't roll back an odometer!")

    def increment_odometer(self, miles):
        self.odometer_reading += miles

    def read_odometer(self):
        print(f"This car has {self.odometer_reading} miles on it.")

    def fill_gas_tank(self):
        print("OK!")


my_new_car = Car('audi', 'a4', 2019)  # 新建对象
print(my_new_car.get_descriptive_name())  # 调用方法
my_new_car.odometer_reading = 23  # 直接修改属性
my_new_car.update_odometer(100)
my_new_car.read_odometer()
my_new_car.fill_gas_tank()
# 2019 Audi A4
# This car has 100 miles on it.
# OK!

class Battery:
    def __init__(self, battery_size=75):
        self.battery_size = battery_size

    def describe_battery(self):
        print(f"This car has a {self.battery_size}-kWh battery.")


class ElectricCar(Car):  # 继承
    def __init__(self, make, model, year):
        super().__init__(make, model, year)  # 初始化父类
        self.battery = Battery()

    def fill_gas_tank(self):  # 重写父类方法
        print("This car doesn't need a gas tank!")


my_tesla = ElectricCar('tesla', 'model s', 2019)
print(my_tesla.get_descriptive_name())
my_tesla.fill_gas_tank()
# 2019 Tesla Model S
# This car doesn't need a gas tank!
```

## 5. IO与异常

### 5.1 IO

```python
message = input(
    "Tell me something, and I will repeat it back to you: ")  # 接收输入
print(message)

# pi_digits.txt
# 3.1415926535
#  8979323846
#  2643383279
# with可在代码中开辟一段由其管理的上下文，并控制程序进出该上下文时的行为
with open('pi_digits.txt') as file_object:  # 打开文件
    contents = file_object.read()  # 全部读取
print(contents)
# 3.1415926535
#  8979323846
#  2643383279

with open('pi_digits.txt') as file_object:
    for line in file_object:  # 逐行读取
        print(line.rstrip())
# 3.1415926535
#  8979323846
#  2643383279

with open('pi_digits.txt') as file_object:
    lines = file_object.readlines()  # 读取为列表
print(lines)
# ['3.1415926535\n', ' 8979323846\n', ' 2643383279']

with open('test.txt', 'w') as file_object:  # r：读（默认），w：写，a：附加，r+：读写
    file_object.write("I love programming.")
```

#### 5.2 异常

#### 使用

```python
def safe_int(value):
    try:
        tmp = int(value)
    except TypeError:  # 捕获异常
        print(f'type error: {type(value)} is invalid')
    except ValueError:  
        print(f'value error: {value} is invalid')
        raise  # 重新抛出异常，交由上层处理
    else:
        return tmp  # 没有抛出异常才执行
    finally:  # 一定会被执行，即使有return
        print("function completed")

safe_int(None)
print(safe_int(2))
# type error: <class 'NoneType'> is invalid
# function completed
# function completed
# 2

class DummyContext:
    def __init__(self, name):
        self.name = name
    def __enter__(self):  # 实现了enter和exit这两个魔法方法，就能用with进行上下文管理
        return f'{self.name}-{random.random()}'
    def __exit__(self, exc_type, exc_val, exc_tb):  # exc_type: 异常类型 exc_val：异常对象 exc_tb: 错误堆栈对象
        print('Exiting DummyContext')  # 上下文内没有抛出异常时，三个参数都为None
        return False  # False：异常继续抛出到上层 True：不抛出异常

with DummyContext('foo') as name:
    print(f'Name: {name}')
# Name: foo-0.7825652534797632
# Exiting DummyContext

from contextlib import contextmanager

def create_conn(host, port, timeout):
    pass

@contextmanager  # 该装饰器可把任何一个生成器函数直接转换为上下文管理器
def create_conn_obj(host, port, timeout=None):    
    """创建连接对象，并在退出上下文时自动关闭"""    
    conn = create_conn(host, port, timeout=timeout)    
    try:
        yield conn  # 以yield关键字为界，前面的逻辑为enter，后面的为exit
    finally:
        conn.close()
```



#### 建议

- 异常匹配从上往下顺序执行，越精准的`except`语句需要放到越前面
- 上下文管理器可用来简化异常：
  - 替代`finally`语句清理资源
  - 忽略异常

## 6. 库

### 6.1 json：存储数据

```python
import json

numbers = [2, 3, 5, 7, 11, 13]
with open("numbers.json", 'w') as f:
    json.dump(numbers, f)  # 存储

with open("numbers.json") as f:
    print(json.load(f))  # 读取
```

### 6.2 unittest：单元测试

```python
import unittest


def get_formatted_name(first_name, last_name, middle_name=''):
    if middle_name:
        full_name = f"{first_name} {middle_name} {last_name}"
    else:
        full_name = f"{first_name} {last_name}"
    return full_name.title()


class NamesTestCase(unittest.TestCase):  # 继承unittest
    def test_first_last_name(self):  # 每个测试方法要以test开头
        formatted_name = get_formatted_name('janis', 'joplin')
        self.assertEqual(formatted_name, 'Janis Joplin')  # 相等性断言

    def test_first_last_middle_name(self):
        formatted_name = get_formatted_name('wolfgang', 'mozart', 'amadeus')
        self.assertEqual(formatted_name, 'Wolfgang Amadeus Mozart')


class AnonymousSurvey:
    def __init__(self, question):
        self.question = question
        self.responses = []

    def show_question(self):
        print(self.question)

    def store_response(self, new_response):
        self.responses.append(new_response)

    def show_results(self):
        print("Survey results:")
        for response in self.responses:
            print(f"- {response}")


class TestAnonymousSurvey(unittest.TestCase):

    def setUp(self):  # 可用此方法进行预处理
        question = "What language did you first learn to speak?"
        self.my_survey = AnonymousSurvey(question)
        self.responses = ['English', 'Spanish', 'Mandarin']

    def test_store_single_response(self):
        self.my_survey.store_response(self.responses[0])
        self.assertIn('English', self.my_survey.responses)

    def test_store_three_responses(self):
        for response in self.responses:
            self.my_survey.store_response(self.responses)
        for response in self.responses:
            self.assertIn(response, self.my_survey.responses)
```

## 7. 示例

### 7.1 冒泡排序

实现冒泡排序算法，把较大的数字放在后面，默认所有的偶数都比奇数大。

输入：[23, 32, 1, 3, 4, 19, 20, 2, 4]

输出：[1, 3, 19, 23, 2, 4, 4, 20, 32]

```python
def magic_bubble_sort(numbers):
    """冒泡排序"""
    j = len(numbers) - 1
    while j > 0:
        for i in range(j):
            if numbers[i] % 2 == 0 and numbers[i + 1] % 2 == 1:
                numbers[i], numbers[i + 1] = numbers[i + 1], numbers[i]
                continue
            elif (numbers[i + 1] % 2 == numbers[i] % 2) and numbers[i] > numbers[i + 1]:
                numbers[i], numbers[i + 1] = numbers[i + 1], numbers[i]
                continue
        j -= 1
    return numbers

numbers = [23, 32, 1, 3, 4, 19, 20, 2, 4]
print(magic_bubble_sort(numbers))
# [1, 3, 19, 23, 2, 4, 4, 20, 32]
```

改进：

```python
# 改进

from typing import List

def magic_bubble_sort(numbers: List[int]):
    """冒泡排序，默认偶数都大于奇数
    :param numbers: 需排序的列表，汇编直接修改
    :return: 返回修改过的列表
    """
    stop_position = len(numbers) - 1
    while stop_position > 0:
        for i in range(stop_position):
            current, next_ = numbers[i], numbers[i + 1]
            current_is_even, next_is_even = current % 2 == 0, next_ % 2 == 0
            should_swap = False

            if current_is_even and not next_is_even:
                should_swap = True
            elif current_is_even == next_is_even and current > next_:
                should_swap = True
            
            if should_swap:
                numbers[i], numbers[i + 1] = numbers[i + 1], numbers[i]
        stop_position -= 1
    return numbers

numbers = [23, 32, 1, 3, 4, 19, 20, 2, 4]
print(magic_bubble_sort(numbers))
# [1, 3, 19, 23, 2, 4, 4, 20, 32]
```

### 7.2 分析访问日志

有大量如下格式的网页访问日志：

> \# 格式：请求路径 请求耗时（毫秒）   
> /articles/three-tips-on-writing-file-related-codes/ 120   
> /articles/15-thinking-in-edge-cases/ 400   
> /admin/ 3275   
> ..   

要解析成下面的结果：

> \-\-\-   
> \=\=   
> Path: /articles/three-tips-on-writing-file-related-codes/   
> Total requests: 828   
> Performance:   
> \- Less than 100 ms: 16   
> \- Between 100 and 300 ms: 35    
> \- Between 300 ms and 1 s: 119   
> \- Greater than 1 s: 696   
> \=\=   
> Path: /...    
> \-\-\-  

```python
from enum import Enum


class PagePerfLevel(str, Enum):  # 继承str和Enum
    """性能等级"""
    LT_100 = 'Less than 100ms'
    LT_300 = 'Between 100 and 300ms'
    LT_1000 = 'Between 300 and 1s'
    GT_1000 = 'Greater then 1s'


def analyze_v1(log_path):
    """解析日志"""
    path_groups = {}
    with open(log_path, "r") as fp:
        for line in fp:
            path, time_cost_str = line.strip().split()  # 切分
            time_cost = int(time_cost_str)
            if time_cost < 100:
                level = PagePerfLevel.LT_100
            elif time_cost < 300:
                level = PagePerfLevel.LT_300
            elif time_cost < 1000:
                level = PagePerfLevel.LT_1000
            else:
                level = PagePerfLevel.GT_1000

            if path not in path_groups:
                path_groups[path] = {}  # 嵌套字典

            try:
                path_groups[path][level] += 1
            except KeyError:
                path_groups[path][level] = 1  # 第一次出现的路径

    for path, result in path_groups.items():
        print(f'== Path: {path}')
        total = sum(result.values())  # 总访问量
        print(f'   Total requests: {total}')
        print(f'   Performance:')
        sorted_items = sorted(result.items(), key=lambda pair: list(
            PagePerfLevel).index(pair[0]))  # 以性能等级的顺序排序
        for level_name, count in sorted_items:
            print(f'   - {level_name}: {count}')


analyze_v1("logs.txt")
```

改进：

```python
# 改进

from enum import Enum
from collections import defaultdict
from collections.abc import MutableMapping

class PagePerfLevel(str, Enum):  # 性能等级
    LT_100 = 'Less than 100ms'
    LT_300 = 'Between 100 and 300ms'
    LT_1000 = 'Between 300 and 1s'
    GT_1000 = 'Greater then 1s'

class PerfLevelDict(MutableMapping):  # 自定义字典，继承抽象类MutableMapping
    """把响应时间转换为性能等级，并以其为键的字典"""
    def __init__(self):
        self.data = defaultdict(int)  # key不存在时defaultdict会返回默认值0
    
    def __getitem__(self, key):  # 必须实现的几个抽象方法
        return self.data[self.compute_level(key)]  # 转换为性能等级
    
    def __setitem__(self, key, value):
        self.data[self.compute_level(key)] = value
    
    def __delitem__(self, key):
        del self.data[key]

    def __iter__(self):
        return iter(self.data)
    
    def __len__(self):  
        return len(self.data)
    
    def items(self):
        """按顺序返回性能等级数据"""
        return sorted(self.data.items(), key=lambda pair: list(
            PagePerfLevel).index(pair[0]))
    
    def total_requests(self):
        """计算总请求数"""
        return sum(self.values())
    
    @staticmethod
    def compute_level(time_cost_str):  # 静态方法
        """根据响应时间计算性能等级"""
        if time_cost_str in list(PagePerfLevel):
            return time_cost_str
        
        time_cost = int(time_cost_str)
        if time_cost < 100:
            return PagePerfLevel.LT_100
        elif time_cost < 300:
            return PagePerfLevel.LT_300
        elif time_cost < 1000:
            return PagePerfLevel.LT_1000
        return PagePerfLevel.GT_1000

def analyze_v2(log_path):
    """改进后的日志解析过程"""
    path_groups = defaultdict(PerfLevelDict) 
    with open(log_path, "r") as fp:
        for line in fp:
            path, time_cost = line.strip().split()
            path_groups[path][time_cost] += 1

    for path, result in path_groups.items():
        print(f'== Path: {path}')
        print(f'   Total requests: {result.total_requests()}')
        print(f'   Performance:')
        for level_name, count in result.items():
            print(f'   - {level_name}: {count}')

analyze_v2("logs.txt")
```

### 7.3 电影分级

有以下字典格式的电影信息数据：
> movies = [  
> {'name': 'The Dark Knight', 'year': 2008, 'rating': '9'},  
> {'name': 'Kaili Blues', 'year': 2015, 'rating': '7.3'},  
> {'name': 'Citizen Kane', 'year': 1941, 'rating': '8.3'},  
> {'name': 'Project Gutenberg', 'year': 2018, 'rating': '6.9'},  
> {'name': 'Burning', 'year': 2018, 'rating': '7.5'},  
> {'name': 'The Shawshank Redemption ', 'year': 1994, 'rating': '9.3'},  
> ]  

要实现一个小工具：
1. 按rating值把电影划分为S、A、B、C等不同等级
2. 可根据指定的键对电影进行排序

```python
class Movie:
    """电影数据类"""
    def __init__(self, name, year, rating):
        self.name = name
        self.year = year
        self.rating = rating
    
    @property  # 实现rank属性相关的getter方法
    def rank(self):
        """对电影分级"""
        rating_num = float(self.rating)
        if rating_num >= 8.5:
            return 'S'
        elif rating_num >= 8:
            return 'A'
        elif rating_num >= 7:
            return 'B'
        elif rating_num >= 6:
            return 'C'
        else:
            return 'D'
    
def get_sorted_movies(movies, sorting_type):
    """对电影排序
    :param movies: Movie对象列表
    :param sorting_type: 排序方式
    """
    if sorting_type == 'name':
        sorted_movies = sorted(movies, key=lambda movie: movie.name.lower())
    elif sorting_type == 'rating':
        sorted_movies = sorted(movies, key=lambda movie: float(movie.rating), reverse=True)
    elif sorting_type == 'year':
        sorted_movies = sorted(movies, key=lambda movie: movie.year, reverse=True)
    elif sorting_type == 'random':
        sorted_movies = sorted(movies, key=lambda movie: random.random())
    else:
        raise RuntimeError(f'Unknown sorting type: {sorting_type}')
        
    return sorted_movies
```

改进：

```python
import bisect
import random

class Movie:
    """电影数据类"""
    def __init__(self, name, year, rating):
        self.name = name
        self.year = year
        self.rating = rating
    
    @property
    def rank(self):
        """对电影分级"""
        breakpoints = (6, 7, 8, 8.5)
        grades = ('D', 'C', 'B', 'A', 'S')
        index = bisect.bisect(breakpoints, float(self.rating))  # 用二分查找模块简化分支判断
        return grades[index]

def get_sorted_movies(movies, sorting_type):
    """对电影排序
    :param movies: Movie对象列表
    :param sorting_type: 排序方式
    """
    sorting_algos = {  # 排序方法组织为字典
        # sorting_type: (key_func, reverse)
        'name': (lambda movie: movie.name.lower(), False),
        'rating': (lambda movie: float(movie.rating), True),
        'year': (lambda movie: movie.year, True),
        'random': (lambda movie: random.random(), False)
    }

    try:  # 加入异常处理
        key_func, reverse = sorting_algos[sorting_type]
    except KeyError:
        raise RuntimeError(f'Unknown sorting type: {sorting_type}')
    sorted_movies = sorted(movies, key=key_func, reverse=reverse)

    return sorted_movies

def movie_ranker(movies, sorting_type):
    """处理电影数据
    :param movies: 电影数据列表
    :param sorting_type: 排序方式
    """
    movie_items = []
    for movie_json in movies:
        movie = Movie(**movie_json)
        movie_items.append(movie)
    sorted_movies = get_sorted_movies(movie_items, sorting_type)
    for movie in sorted_movies:        
        print(f'- [{movie.rank}] {movie.name}({movie.year}) | rating: {movie.rating}')

movies = [
    {'name': 'The Dark Knight', 'year': 2008, 'rating': '9'},
    {'name': 'Kaili Blues', 'year': 2015, 'rating': '7.3'},
    {'name': 'Citizen Kane', 'year': 1941, 'rating': '8.3'},
    {'name': 'Project Gutenberg', 'year': 2018, 'rating': '6.9'},
    {'name': 'Burning', 'year': 2018, 'rating': '7.5'},
    {'name': 'The Shawshank Redemption ', 'year': 1994, 'rating': '9.3'},
]

movie_ranker(movies, 'rating')
# - [S] The Shawshank Redemption (1994) | rating: 9.3
# - [S] The Dark Knight(2008) | rating: 9
# - [A] Citizen Kane(1941) | rating: 8.3
# - [B] Burning(2018) | rating: 7.5
# - [B] Kaili Blues(2015) | rating: 7.3
# - [C] Project Gutenberg(2018) | rating: 6.9
```


# 谷歌Python代码风格指南 中文翻译
Update: 2020.01.30

Translator: [shendeguize@github](
https://github.com/shendeguize)

Link: [
https://github.com/shendeguize/GooglePythonStyleGuideCN](
https://github.com/shendeguize/GooglePythonStyleGuideCN)

本翻译囿于水平，可能有不准确的地方，欢迎指出，谢谢大家

*如有引用,请注明出处*

## 1 背景
Python是谷歌主要是用的动态语言，本风格指导列举了使用Python编程时应该做和不该做的事项(dos & don'ts)

为了帮助你正确地组织代码,我们编写了一个[Vim的设置文件](https://google.github.io/styleguide/google_python_style.vim).对于Emacs,默认设置即可.

许多团队使用[yapf](https://github.com/google/yapf/)自动格式工具来避免格式争议
## 2 Python语言规则

### 2.1 Lint
对代码使用`pylint`

#### 2.1.1Definition(以下都译为定义)
`pylint`是一个用于在Python代码中发现bug和代码风格问题的工具,，`pylint`查找那些常在非动态语言(例如C或C++)编译器中捕获的问题.由于Python是动态语言,一些警告可能不正确,不过应该非常少有错误警告.

#### 2.1.2 Pros
能够发现一些易被遗漏的错误,类似打字错误,调用早于声明等等.

#### 2.1.3 Cons
`pylint`并不完美,为了更好的利用工具,我们有时候需要

a. Write around it(???)

b. 压制一些警告

c. 优化工具

#### 2.1.4 Decision(以下都译为建议)
确保对代码应用`pylint`

如果一些警告是不合适的,就抑制这些警告,这是为了让其他警告不会被隐藏.为了压制警告,可以设置行级别的注释:

```Python
dict = 'something awful'  # Bad Idea... pylint: disable=redefined-builtin
```

`pylint`警告包含标识名(`empty-docstring`),谷歌专有的警告以`g-`开头.

如果抑制警告的原因在标识名称中表述不够清晰,请额外添加注解.

用这种方式来抑制警告的优点是我们能够简单地找到抑制的警告并且重新访问这些警告.

可以通过下述方式来获得`pylint`警告列表:
```bash
pylint --list-msgs
```

用下述方式来获取某个特定消息的更多具体信息:

```bash
pylint --help-msg=C6409
```

优先使用`pylint: disable`而非旧方法(`pylint: disable-msg`)
如果要抑制由于参数未使用的警告,可以在函数开头del,并注释为什么要删除这些未使用参数,仅仅一句"unused"是不够的:

```Python
def viking_cafe_order(spam, beans, eggs=None):
    del beans, eggs  # Unused by vikings.
    return spam + spam + spa
```

其他可以用来抑制警告的方式包括用`'_'`作为未使用参数的标识,在参数名前增加`'unused_'`,或者分配这些参数到`'_'`.这些方式是可以的,但是已经不鼓励继续使用.前两种方式会影响到通过参数名传参的调用方式,而最后一种并不能保证参数确实未被使用.

### 2.2 Imports
只在import包和模块的时候使用`import`,而不要应用在单独的类或函数.(这一条对于[typing_module](https://google.github.io/styleguide/pyguide.html#typing-imports)有特别的意外)

#### 2.2.1 定义
一个模块到另一个模块之间共享代码的复用性机制

#### 2.2.2 Pros
命名空间管理约定简单,每个标识的源都一致性地被指明了.例如`x.Obj`表示`Obj`是在模块`x`中定义的

#### 2.2.3 Cons
模块名可能会有冲突,一些模块名可能很长,比较不方便

#### 2.2.4 建议
* ` import x`（当`x`是包或模块）
* `from x import y` （当`x`是包前缀，`y`是不带前缀的模块名）
* `from x import  y as z` （当有重复模块名`y`或`y`过长不利于引用的时候）
* `import y as z` （仅在非常通用的简写的时候使用例如`import numpy as np`）

以`sound.effects.echo`为例:

```Python
from sound.effects import echo...echo.EchoFilter(input, output, delay=0.7, atten=4)
```

不要使用相对引用，即便在同一包内，也使用完整包名import,这有助于避免无意重复import包.

从[typing module](https://google.github.io/styleguide/pyguide.html#typing-imports)和[six.moves module](https://six.readthedocs.io/#module-six.moves) import不适用上述规则

### 2.3 包
每一模块都要从完整路径import

#### 2.3.1 Pros
能够避免模块名冲突以及由于模块搜索路径与作者预期不符而造成的错误引用.让查找模块更简单.

#### 2.3.2 Cons
让部署代码时有些困难,因为包架构也需要赋值,不过对于现在的部署机制而言,这其实不是问题.
#### 2.3.3 建议
所有的新代码都要从完整包名来import模块

import示例应该像这样:

**Yes:**

```Python
# Reference absl.flags in code with the complete name (verbose).
# 在代码中使用完整路径调用absl.flags
import absl.flagsfrom doctor.who import jodie

FLAGS = absl.flags.FLAGS
```
``` Python
# Reference flags in code with just the module name (common).
# 在代码中只用包名来调用flags
from absl import flagsfrom doctor.who import jodie

FLAGS = flags.FLAGS
```

**No:**(假设文件在`doctor/who`中,`jodie.py`也在这里)

```Python
# Unclear what module the author wanted and what will be imported.  The actual
# import behavior depends on external factors controlling sys.path.
# Which possible jodie module did the author intend to import?
# 不清楚作者想要哪个包以及最终import的是哪个包,
# 实际的import操作依赖于受到外部参数控制的sys.path
# 那么哪一个可能的jodie模块是作者希望import的呢?
import jodie
```

不应该假设主代码所在路径被包含在`sys.path`中,即使有些时候可以work.在上一例代码中,我们应该认为`import jodie`指的是import一个叫做`jodie`的第三方包或者顶级目录中的`jodie`,而非一个当前路径的`jodie.py`

### 2.4 异常
异常处理是允许使用的,但使用务必谨慎

#### 2.4.1 定义
异常是一种从正常代码段控制流中跳出以处理错误或者其他异常条件的手段.

#### 2.4.2 Pros
正常代码的控制流时不会被错误处理代码影响的.异常处理同样允许在某些情况下,控制流跳过多段代码,例如在某一步从N个嵌入函数返回结果而非强行延续错误代码.

#### 2.4.3 Cons
可能会让控制流变的难于理解,也比较容易错过调用库函数的报错.

#### 2.4.4 建议
异常必定遵循特定条件:
* 使用`raise MyError('Error message')`或者`raise MyError()`，不要使用两段`raise MyError, 'Error message'`
* 当内置异常类合理的时候,尽量使用内置异常.例如:抛出`ValueError`来表示一个像是违反预设前提(例如传参了一个负数给要求正数的情况)的程序错误发生.

不要使用`assert`来片段公共结构参数值.`assert`是用来确认内部计算正确性也不是用来表示一些预期外的事件发生的.如果异常是后续处理要求的,用`raise`语句来处理,例如:

**Yes:**

```Python
def connect_to_next_port(self, minimum):
"""Connects to the next available port.

Args:
    minimum: A port value greater or equal to 1024.

Returns:
    The new minimum port.

Raises:
    ConnectionError: If no available port is found.
"""
if minimum < 1024:
    # Note that this raising of ValueError is not mentioned in the doc
    # string's "Raises:" section because it is not appropriate to
    # guarantee this specific behavioral reaction to API misuse.
    # 注意抛出ValueError这件事是不在docstring中的Raises中提及, 因为这样并适合保障对于API误用的特殊反馈
    raise ValueError('Minimum port must be at least 1024, not %d.' % (minimum,))
port = self._find_next_open_port(minimum)
if not port:
    raise ConnectionError('Could not connect to service on %d or higher.' % (minimum,))
assert port >= minimum, 'Unexpected port %d when minimum was %d.' % (port, minimum)
return port
```

**No:**

```Python
def connect_to_next_port(self, minimum):
"""Connects to the next available port.

Args:
    minimum: A port value greater or equal to 1024.

Returns:
    The new minimum port.
"""
assert minimum >= 1024, 'Minimum port must be at least 1024.'
port = self._find_next_open_port(minimum)
assert port is not None
return port
```

* 库或者包可能会定义各自的异常.当这样做的时候,必须要继承一个已经存在的异常类,异常类的名字应该以`Error`结尾,并且不应该引入重复(`foo.FooError`)
* 永远不要用捕获全部异常的`except:`语句,或者捕获`Exception`或者`StandardError`除非:
    * 再次抛出这个异常
    * 在程序中异常不会继续但是会被记录以及消除(例如通过保护最外层的方式保护线程不会崩溃)的地方创造一个孤立点.

    Python在这个方面容忍度很高,并且`except:`语句会捕获包括拼写错误,sys.exit(),Ctrl+C终止,单元测试失败和和所有你并没有想到捕获的其他异常.
* 最精简`try/except`表达式内部的代码量,`try`代码块里的代码体量越大,月可能会在你不希望抛出异常的代码中抛出异常,进而在这种情况下,`try/except`掩盖了一个真实的异常
* 使用finally来执行代码,这些代码无论是否有异常在`try`代码块被抛出都会被执行.这在清理(即关闭文件)时非常有用.
* 当捕获了异常时,用as而不是逗号分段.

```Python
try:
    raise Error()
except Error as error:
    pass
```

### 2.5 全局变量
避免全局变量

#### 2.5.1 定义
在模块级别或者作为类属性声明的变量

#### 2.5.2 Pros
有些时候有用

#### 2.5.3 Cons
在import的过程中,有可能改变模块行为,因为在模块首次被引入的过程中,全局变量就已经被声明

#### 2.5.4 建议
避免全局变量

作为技术变量,模块级别的常量是允许并鼓励使用的.例如`MAX_HOLY_HANDGRENADE_COUNT = 3`, 常量必须由大写字母和下划线组成,参见下方[命名规则](https://google.github.io/styleguide/pyguide.html#s3.16-naming)

如果需要,全局变量需要在模块级别声明,并且通过在变量名前加`_`来使其对模块内私有化.外部对模块全局变量的访问必须通过公共模块级别函数,参见下方[命名规则](https://google.github.io/styleguide/pyguide.html#s3.16-naming)

### 2.6 内嵌/局部/内部 类和函数
内嵌局部函数或类在关闭局部变量时是可以的.内部类意识可用的.(译注:这里我的理解是当内嵌局部函数或类是和局部变量在同一个封闭作用域内是可以的.)

#### 2.6.1 定义
类可以在方法,函数,类内定义.函数可以在方法或函数内定义.内嵌函数对封闭作用域的变量具有只读访问权限.

#### 2.6.2 Pros
允许定义只在非常有限作用域内可用的工具类或工具函数.Very ADT-y(??符合抽象数据类型要求???),通常用于实现装饰器

#### 2.6.3 Cons
内嵌或局部类的实例是不能被pickle的,内嵌函数或类是不能被直接测试的.嵌套会让外部函数更长并且更难读懂.

#### 2.6.4 建议
除了一些特别声明,这些内嵌/局部/内部类和函数都是可以的.
避免内嵌函数或类除了需要关闭一个局部值的时候.(译者理解可能是除了将局部变量封闭在同一个作用域的情况以外).不要把一个函数转为内嵌指示为了避免访问.在这种情况下,把函数置于模块级别并在函数名前加`_`以保证测试是可以访问该函数的.

### 2.7 列表推导和生成器表达式
在简单情况下是可用的

#### 2.7.1 定义
List, Dict和Set推导生成式以及生成器表达式提供了一个简明有效的方式来生成容器和迭代器而不需要传统的循环,`map()`,`filter()`或者`lambda表达式`

#### 2.7.2 Pros
简单地推导表达比其他的字典,列表或集合生成方法更加简明清晰.生成器表达式可以很有效率,因为完全避免了生成列表.

#### 2.7.3 Cons
负载的推导表达式或生成器表达式很难读懂

#### 2.7.4 建议
简单情况下使用时可以的.每个部分(mapping表达式,filter表达式等)都应该在一行内完成.多个for条款或者filter表达式是不允许的.当情况变得很复杂的适合就使用循环.

**Yes:**

```Python
result = [mapping_expr for value in iterable if filter_expr]

result = [{'key': value} for value in iterable
          if a_long_filter_expression(value)]

result = [complicated_transform(x)
          for x in iterable if predicate(x)]

descriptive_name = [
    transform({'key': key, 'value': value}, color='black')
    for key, value in generate_iterable(some_input)
    if complicated_condition_is_met(key, value)
]

result = []
for x in range(10):
    for y in range(5):
        if x * y > 10:
            result.append((x, y))

return {x: complicated_transform(x)
        for x in long_generator_function(parameter)
        if x is not None}

squares_generator = (x**2 for x in range(10))

unique_names = {user.name for user in users if user is not None}

eat(jelly_bean for jelly_bean in jelly_beans
    if jelly_bean.color == 'black')
```

**No:**

```Python
result = [complicated_transform(
          x, some_argument=x+1)
          for x in iterable if predicate(x)]

result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]

return ((x, y, z)
        for x in range(5)
        for y in range(5)
        if x != y
        for z in range(5)
        if y != z)
```

### 2.8 默认迭代器和运算符
对支持默认迭代器和云算法的类型例如列表,字典和文件等使用它们

#### 2.8.1 定义
容器类型(例如字典,列表等)定义了的默认的迭代器和成员检查运算符.

#### Pros
默认迭代器和操作符是简单有效的,能够直接不需额外调用方法地表达操作.使用默认操作符的函数是通用的.能被用于任何支持这些操作的类型.

#### Cons
不能通过方法名来分辨类型,例如`has_key()`意味着字典,当然这也是一种优势.

#### 建议
对于支持的类型诸如列表,字典和文件,使用默认迭代器和操作符.内置类型同样定义了迭代器方法.优先使用这些方法而非那些返回列表的方法.吹飞能够确定在遍历容器的过程中不会改变容器.不要使用Python 2专有迭代方法除非必要.

**Yes:**

```Python
for key in adict: ...
if key not in adict: ...
if obj in alist: ...
for line in afile: ...
for k, v in adict.items(): ...
for k, v in six.iteritems(adict): ...
```

**No:**

```Python
for key in adict.keys(): ...
if not adict.has_key(key): ...
for line in afile.readlines(): ...
for k, v in dict.iteritems(): ...
```
### 2.9 生成器
需要时使用生成器

#### 2.9.1 定义
生成器函数返回一个迭代器,每次执行`yield`语句的时候生成一个值.在生成一个值之后,生成器函数的运行被挂起直到需要下一个值.

#### 2.9.2 Pros
简化代码,因为局部变量和控制流在每次调用时被保留,生成器相比于一次性生成整个一个列表值要更节省内存.

#### 2.9.3 Cons
无

#### 2.9.4 建议
建议使用.在生成器函数的文档字符串中使用"Yields:"而非"Returns:"

### 2.10 Lambda表达式
单行代码时是可以的

#### 2.10.1 定义
lambda在一个表达式内定义了匿名函数,而不在语句里.lambda表达式常被用于定义高阶函数(例如`map()`和`filter()`)使用的回调函数或者操作符.

#### 2.10.2 Pros
方便

#### 2.10.3 Cons
比局部函数更难读懂和debug,匿名意味着堆栈跟踪更难懂.表达性受限因为lambda函数只包含一个表达式

#### 2.10.4 建议
对于单行代码而言,可以使用lambda表达式.如果`lambda`表达式内的代码超过60-80个字符,最好定义成为常规的内嵌函数.

对于一般的操作诸如乘法,使用`operator`模块内置函数而非重新定义匿名函数,例如使用`operator.mul`而非`lambda x,y: x * y`

### 2.11 条件表达式
简单情况下可以使用.

#### 2.11.1 定义
条件表达式(也称为三元运算符)是一种更短替代if语句的机制.例如`x = 1 if cond else 2`

#### 2.11.2 Pros
相对于if语句更短也更方便

#### 2.11.3 Cons
比if语句可能更难读懂,当表达式很长的时候条件部分可能很难定位.

#### 2.11.4 建议
简单情况可以使用.每个部分(真值表达式,if表达式,else表达式)必须在一行内完成.如果使用条件表达式很富的时候使用完整的if语句.

**Yes:**

```Python
one_line = 'yes' if predicate(value) else 'no'
slightly_split = ('yes' if predicate(value)
                  else 'no, nein, nyet')
the_longest_ternary_style_that_can_be_done = (
    'yes, true, affirmative, confirmed, correct'
    if predicate(value)
    else 'no, false, negative, nay')
```

**No:**

```Python
bad_line_breaking = ('yes' if predicate(value) else
                     'no')portion_too_long = ('yes'
                    if some_long_module.some_long_predicate_function(
                        really_long_variable_name)
                    else 'no, false, negative, nay')
```

### 2.12 默认参数值
大多数情况下都OK

#### 2.12.1 定义
在函数参数列表的最后可以为变量设定值,例如`def foo(a, b=0):`.如果`foo`在调用时只传入一个参数,那么`b`变量就被设定为0,如果调用时传入两个参数,那么`b`就被赋予第二个参数值.

#### 2.12.2 Pros
通常一个函数可能会有大量默认值,但是很少会有需要修改这些默认值的时候.默认值就提供了一个很简单满足上述情况的方式,而不需要为这些少见的情况重新定义很多函数.因为Python不支持重载方法或函数,默认参数是一个很简单的方式来"假重载"行为.

#### 2.12.3 Cons
默认参数在模块加载时就被复制.这在参数是可变对象(例如列表或字典)时引发问题.如果函数修改了这些可变对象(例如向列表尾添加元素).默认值就被改变了.

#### 2.12.4 建议
使用时请注意以下警告----在函数或方法定义时不要将可变对象作为默认值.

**Yes:**

```Python
def foo(a, b=None):
    if b is None:
        b = []
def foo(a, b: Optional[Sequence] = None):
    if b is None:
        b = []
def foo(a, b: Sequence = ()):  # Empty tuple OK since tuples are immutable 空元组是也不可变的
    ...
```

**No:**

```Python
def foo(a, b=[]):
    ...
def foo(a, b=time.time()):  # The time the module was loaded??? 模块被加载的时间???
    ...
def foo(a, b=FLAGS.my_thing):  # sys.argv has not yet been parsed... sys.argv还未被解析
    ...
def foo(a, b: Mapping = {}):  # Could still get passed to unchecked code 仍可传入未检查的代码(此处翻译可能有误)
    ...
```

### 2.13 属性
使用属性可以通过简单而轻量级的访问器和设定器方法来来访问或设定数据.

#### 2.13.1 定义
一种装饰器调用来在计算比较轻量级时作为标准的属性访问来获取和设定一个属性的方式

#### 2.13.2 Pros
对于简单的属性访问,减少显式的get和set方法能够提升可读性.允许惰性计算.被认为是一种Python化的方式来维护类接口.在表现上,当直接对变量的访问更合理时,允许属性绕过所需的琐碎的访问方法.

#### 2.13.3 Cons
在Python2中必须继承于`object`,可能会隐藏像是操作符重载之类的副作用.对于子类而言,属性可能有些迷惑性.

#### 2.13.4 建议
在通常会有简单而且轻量级的访问和设定方法的新代码里使用属性来访问或设定数据.属性在创建时被`@property`装饰,参加[装饰器](https://google.github.io/styleguide/pyguide.html#s2.17-function-and-method-decorators)

如果属性本身未被重写,带有属性的继承可能不够明晰,因而必须确保访问方法是被间接访问的,来确保子类的方法重载是被属性调用的(使用Template Method DP,译者:应是模板方法设计模式).

**Yes:**

```Python
class Square(object):
    """A square with two properties: a writable area and a read-only perimeter.

    To use:
    >>> sq = Square(3)
    >>> sq.area
    9
    >>> sq.perimeter
    12
    >>> sq.area = 16
    >>> sq.side
    4
    >>> sq.perimeter
    16
    """

    def __init__(self, side):
        self.side = side

    @property
    def area(self):
        """Area of the square."""
        return self._get_area()

    @area.setter
    def area(self, area):
        return self._set_area(area)

    def _get_area(self):
        """Indirect accessor to calculate the 'area' property."""
        return self.side ** 2

    def _set_area(self, area):
        """Indirect setter to set the 'area' property."""
        self.side = math.sqrt(area)

    @property
    def perimeter(self):
        return self.side * 4
```

### 2.14 True/Flase表达式
只要可能,就使用隐式False的if语句

#### 2.14.1 定义
在布尔环境下,Python对某些值判定为False,一个快速的经验规律是所有"空"值都被认为是False,所以`0, None, [], {}, ''`的布尔值都是False

#### 2.14.2 Pros
使用Python布尔类型的条件语句可读性更好而且更难出错,大多数情况下,这种方式也更快.

#### 2.14.3 Cons
对于C/C++开发者而言可能有些奇怪

#### 建议
如果可能的话,使用隐式False.例如使用`if foo:`而非`if foo != []:`下面列举了一些你应该牢记的警告:

* 使用`if foo is None`(或者`if foo is not None`)来检查`None`.例如在检查一个默认值是`None`的变量或者参数是否被赋予了其他值的时候,被赋予的其他值的布尔值可能为False.
* 不要用`==`来和布尔值为`False`的变量比较,使用`if not x`,如果需要区别`False`和`None`,那么使用链式的表达式如`if not x and x is not None`
* 对于序列(如字符串,列表,元组),利用空序列为`False`的事实,故而相应地使用`if seq:`和`if not seq:`而非`if len(seq)`或`if not len(seq):`.
* 在处理整数时,隐式的False可能会引入更多风险(例如意外地将`None`和0进行了相同的处理)你可以用一个已知是整形(并且不是`len()`的结果)的值和整数0比较.

**Yes:**

```Python
if not users:
    print('no users')

if foo == 0:
    self.handle_zero()

if i % 10 == 0:
    self.handle_multiple_of_ten()

def f(x=None):
    if x is None:
        x = []
```

**No:**

```Python
if len(users) == 0:
    print('no users')

if foo is not None and not foo:
    self.handle_zero()

if not i % 10:
    self.handle_multiple_of_ten()

def f(x=None):
    x = x or []
```

### 2.15 弃用的语言特性
尽可能利用字符串方法而非`string`模块.使用函数调用语法而非`apply`.在函数参数本就是一个行内匿名函数的时候,使用列表推导表达式和for循环而非`filter`和`map`

#### 2.15.1 定义
当前Python版本提供了人们普遍更倾向的构建方式.

#### 2.15.2 建议
我们不是用任何不支持这些特性的Python版本,因而没有理由使用新方式.

**Yes:**

```Python
words = foo.split(':')

[x[1] for x in my_list if x[2] == 5]

map(math.sqrt, data)    # Ok. No inlined lambda expression. 可以,没有行内的lambda表达式

fn(*args, **kwargs)
```

**No:**
```Python
words = string.split(foo, ':')

map(lambda x: x[1], filter(lambda x: x[2] == 5, my_list))

apply(fn, args, kwargs)
```

### 2.16 词法作用域
可以使用

#### 2.16.1 定义
一个内嵌Python函数可以引用在闭包命名空间内定义的变量,但是不能对其复制.变量绑定是解析到使用词法作用域的,即基于静态程序文本.任何对块内命名的赋值都会让Python将对于这个命名的引用都作为局部变量,即使在使用先于赋值的情况下也是.如果有全局声明,这个命名就会被认为是全局变量.

一个使用这个特性的例子是:
```Python
def get_adder(summand1):
    """Returns a function that adds numbers to a given number."""
    def adder(summand2):
        return summand1 + summand2

    return adder
```

#### 2.16.2 Pros
经常可以让代码更简明优雅,尤其会让有经验的Lisp和Scheme(以及Haskell和ML还有其他)的程序要很舒服.

#### 2.16.3 Cons
可能会导致令人迷惑的bug例如这个基于[PEP-0227](http://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0227/)的例子.

```Python
i = 4def foo(x):
    def bar():
        print(i, end='')
    # ...
    # A bunch of code here
    # ...
    for i in x:  # Ah, i *is* local to foo, so this is what bar sees i对于foo来说是局部变量,所以在这里就是bar函数所获取的值
        print(i, end='')
    bar()
```

所以`foo([1, 2, 3])`会打印`1 2 3 3`而非`1 2 3 4`.

#### 2.16.4 建议
可以使用

### 2.17 函数和方法装饰器
在明显有好处时,谨慎明智的使用，避免`@staticmethod`，控制使用`@classmethod`

#### 2.17.1 定义
[函数和方法装饰器](https://docs.python.org/3/glossary.html#term-decorator)(也就是`@`记号).一个常见的装饰器是`@property`,用于将普通方法转换成动态计算属性.然而装饰器语法也允许用户定义装饰器,尤其对于一些函数`my_decorator`如下:

```Python
class C(object):
    @my_decorator
    def method(self):
        # method body ...
```

是等效于

```Python
class C(object):
    def method(self):
        # method body ...
    method = my_decorator(method)
```

#### 2.17.2 Pros
能够优雅的对方法进行某种转换,而该转换可能减少一些重复代码并保持不变性等等.

#### 2.17.3 Cons
装饰器可以对函数的参数和返回值任意操作,导致非常隐形的操作行为.此外,装饰器在import的时候就被执行,装饰器代码的实效可能非常难恢复.

### 2.17.4 建议
在有明显好处的地方谨慎地使用装饰器.装饰器应该和函数遵守相同的import和命名指导规则.装饰器的文档应该清晰地声明该函数为装饰器函数.并且要为装饰器函数编写单元测试.

避免装饰器自身对外部的依赖,(如不要依赖于文件,socket,数据库连接等等),这是由于在装饰器运行的时候(在import时,可能从`pydoc`或其他工具中)这些外部依赖可能不可用.一个被传入有效参数并调用的装饰器应该(尽可能)保证在任何情况下都可用.

装饰器是一种特殊的"顶级代码",参见[main](https://google.github.io/styleguide/pyguide.html#s3.17-main)

永远不要使用`@staticmethod`,除非不得不整合一个API到一个已有的库,应该写一个模块等级的函数.

只在写一个命名的构造器或者一个类特定的,修改必要的全局状态(例如进程缓存等)的流程时使用`@classmethod`.

### 2.18 线程
不要依赖于内建类型的原子性

尽管Python内置数据类型例如字典等似乎有原子性操作,仍有一些罕见情况下,他们是非原子的(比如,如果`__hash__`或者`__eq__`被实现为Python方法),就不应该依赖于这些类型的原子性.也不应该依赖于原子变量赋值(因为这依赖于字典)

优先使用Queue模块的`Queue`类来作为线程之间通讯数据的方式.此外,要是用threading模块和其locking primitives(锁原语).了解条件变量的合理用法以便于使用`threading.Condition`而非使用更低级的锁.

### 2.19 过于强大的特性
尽量避免使用

#### 2.19.1 定义
Python是一种非常灵活的语言并且提供了很多新奇的特性,诸如定制元类,访问字节码,动态编译,动态继承,对象父类重定义,import hacks,反射(例如一些对于`getattr()`的应用),系统内置的修改等等.

#### 2.19.2 Pros
这些是非常强大的语言特性,可以让程序更紧凑

#### 2.19.3 Cons
使用这些新特性是很诱人的.但是并不绝对必要,它们很难读很难理解.也很难debug那些在底层使用了不常见的特性的代码.对于原作者而言可能不是这样,但是再次看代码的时候,可能比更长但是更直接的代码要难.

#### 2.19.4 定义
避免在代码中使用这些特性.

内部使用这些特性的标准库和类是可以使用的(例如`abc.ABCMeta`,`collections.namedtuple`,和`enum`)

### 2.20 新版本Python: Python3 和从`__future__`import
Python3已经可用了(译者:目前Python2已经不受支持了),尽管不是每个项目都准备好使用Python3,所有的代码应该兼容Python3并且在可能的情况下在Python3的环境下测试.

#### 2.20.1 定义
Python3是Python的鲜明改变,当已有代码经常是Python2.7写成的,有一些简单可以做的事情来让代码对于其倾向更简明,因而可以让代码更好地在Python3下运行不用调整.

#### 2.20.2 Pros
在考虑Python3同时写代码更清晰也更容易在Python3环境下运行(只要所有依赖已就绪).

#### 2.20.3 Cons
一些人会认为默认样板有些丑,import实际不需要的特性到模块中是不常见的.

#### 2.20.4 建议

**from __future__ imports**

鼓励使用`from __future__ import`语句.所有新代码都应该包含下述代码,而现有代码应该被更新以尽可能兼容:

```Python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

如果你不太熟悉这些,详细阅读这些:[绝对import](https://www.python.org/dev/peps/pep-0328/),[新的`/`除法行为](https://www.python.org/dev/peps/pep-0238/),和[`print`函数](https://www.python.org/dev/peps/pep-3105/)

请勿省略或移除这些import,即使在模块中他们没有在使用,除非代码只用于Python3.最好总是在所有的文档中都有从future的import,来保证不会在有人使用在后续编辑时遗忘.

有其他的`from __future__`import语句,看喜好使用.我们的建议中不包含`unicode_literals`因为其并无明显优势,这是由于隐式默认的编码转换导致其在Python2.7内很多地方,必要时,大多数代码最好显式的使用`b''`和`u''`btyes和unicode字符串表示.(译者:这段翻译可能不准确)

**The six, future, or past libraries**

当项目需要支持Python2和3时,根据需求使用[six](https://pypi.org/project/six/),[future](https://pypi.org/project/future/)和[past](https://pypi.org/project/past/).

### 2.21 带有类型注释的代码
可以根据[PEP-484](https://www.python.org/dev/peps/pep-0484/)对Python3代码进行类型注释,并且在build时用类型检查工具例如[pytype](https://github.com/google/pytype)进行类型检查.

类型注释可以在源码中或[stub pyi file](https://www.python.org/dev/peps/pep-0484/#stub-files)中.只要可能,注释就应写在源代码中.对于第三方或拓展模块使用pyi文件.

#### 2.21.1 定义
类型注释(也称为"类型提示")是用于函数或方法参数和返回值的:

```Python
def func(a: int) -> List[int]:
```

你也可以声明用一个单独的注释来声明变量的类型:

```Python
a = SomeFunc()  # type: SomeType
```
#### 2.21.2 Pros
类型注释提升代码的可读性和可维护性,类型检查会将很多运行错误转化为构建错误,也减少了使用[过于强力特性](https://google.github.io/styleguide/pyguide.html#power-features)的能力.

#### 2.21.3 Cons
需要不断更新类型声明,对于认为有效的代码可能会报类型错误,使用[类型检查](https://github.com/google/pytype)可能减少使用[过于强力特性](https://google.github.io/styleguide/pyguide.html#power-features)的能力.

#### 2.21.4 建议
强烈鼓励在更新代码的时候进行Python类型分析.在对公共API进行补充和修改时,包括python类型声明并通过构建系统中的pytype进行检查.对Python来说静态类型检查比较新,我们承认,一些意料外的副作用(例如错误推断的类型)可能拒绝一些项目的使用.这种情况下,鼓励作者适当地增加一个带有TODO或到bug描述当前不接搜的类型注释的链接到BUILD文件或者在代码内.

## 3 Python代码风格规范




### **TBD**

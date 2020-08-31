# 谷歌Python代码风格指南 中文翻译
Update: 2020.01.31

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
能够发现一些易被遗漏的错误,类似拼写错误,调用早于声明等等.

#### 2.1.3 Cons
`pylint`并不完美,为了更好的利用工具,我们有时候需要

a. Write around it(适配上下文风格)

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
对于支持的类型诸如列表,字典和文件,使用默认迭代器和操作符.内置类型同样定义了迭代器方法.优先使用这些方法而非那些返回列表的方法.除非能够确定在遍历容器的过程中不会改变容器.不要使用Python 2专有迭代方法除非必要.

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
使用属性可以通过简单而轻量级的访问器和设定器方法来访问或设定数据.

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

### 2.14 True/False表达式
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
我们不使用任何不支持这些特性的Python版本,因而没有理由不使用新方式.

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
i = 4
def foo(x):
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
Python3是Python的重大改变,尽管现有代码通常是Python2.7写成的,但可以做一些简单的事情来让代码更加明确地表达其意图,从而可以让代码更好地在Python3下运行而不用调整.

#### 2.20.2 Pros
在考虑Python3编写的代码更清晰明确，一旦所有依赖已就绪，就可以更容易在Python3环境下运行.

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

有其他的`from __future__`import语句,看喜好使用.我们的建议中不包含`unicode_literals`因为其并无明显优势,这是由于隐式默认的编码转换导致其在Python2.7内很多地方被引入了,必要时,大多数代码最好显式的使用`b''`和`u''`btyes和unicode字符串表示.(译者:这段翻译可能不准确)

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

### 3.1 分号
不要在行尾加分号，也不要用分号把两行语句合并到一行

### 3.2 行长度
最大行长度是*80个字符*

超出80字符的明确例外:
* 长import
* 注释中的：URL,路径,flags等
* 不包含空格不方便分行的模块级别的长字符串常量
* pylint的diable注释使用(如`# pylint: disable=invalid-name`)

不要使用反斜杠连接,除非对于需要三层或以上的上下文管理器`with`语句

利用Python的[implicit line joining inside parentheses, brackets and braces](http://docs.python.org/reference/lexical_analysis.html#implicit-line-joining)(隐式行连接方法--括号连接,包括`(), [], {}`).如果必要的话,也可在表达式外面额外添加一对括号.

**Yes:**

```Pyhon
foo_bar(self, width, height, color='black', design=None, x='foo',
        emphasis=None, highlight=0)

if (width == 0 and height == 0 and
    color == 'red' and emphasis == 'strong'):
```

当字符串不能在一行内完成时,使用括号来隐式连接行:

```Python
x = ('This will build a very long long '
     'long long long long long long string')
```

在注释内,如有必要,将长URL放在其本行内:

**Yes:**

```Python
# See details at
# http://www.example.com/us/developer/documentation/api/content/v2.0/csv_file_name_extension_full_specification.html
```

**No:**

```Python
# See details at
# http://www.example.com/us/developer/documentation/api/content/\
# v2.0/csv_file_name_extension_full_specification.html
```

在定义一个表达式超过三行或更多的`with`语句时,可以使用反斜杠来分行.对于两行表达式,使用嵌套`with`语句:

**Yes:**

```Python
with very_long_first_expression_function() as spam, \
     very_long_second_expression_function() as beans, \
     third_thing() as eggs:
    place_order(eggs, beans, spam, beans)

with very_long_first_expression_function() as spam:
    with very_long_second_expression_function() as beans:
        place_order(beans, spam)
```

**No:**
```Python
with VeryLongFirstExpressionFunction() as spam, \
     VeryLongSecondExpressionFunction() as beans:
    PlaceOrder(eggs, beans, spam, beans)
```

注意上述例子中的缩进,具体参看[缩进](https://google.github.io/styleguide/pyguide.html#s3.4-indentation)

在其他一行超过80字符的情况下,而且[yapf](https://github.com/google/yapf/)自动格式工具也不能使分行符合要求时,允许超过80字符限制.

### 3.3 括号
括号合理使用

尽管不必要,但是可以在元组外加括号.再返回语句或者条件语句中不要使用括号,除非是用于隐式的连接行或者指示元组.

**Yes:**

```Python
if foo:
    bar()
while x:
    x = bar()
if x and y:
    bar()
if not x:
    bar()
# For a 1 item tuple the ()s are more visually obvious than the comma.
onesie = (foo,)
return foo
return spam, beans
return (spam, beans)
for (x, y) in dict.items(): ...
```

**No:**

```Python
if (x):
    bar()
if not(x):
    bar()
return (foo)
```

### 3.4 缩进
缩进用4个空格

缩进代码段不要使用制表符,或者混用制表符和空格.如果连接多行,多行应垂直对齐,或者再次4空格缩进(这个情况下首行括号后应该不包含代码).

**Yes:**

```Python
# Aligned with opening delimiter
# 和opening delimiter对齐(译者理解是分隔符的入口,例如三种括号,字符串引号等)
foo = long_function_name(var_one, var_two,
                         var_three, var_four)
meal = (spam,
        beans)

# Aligned with opening delimiter in a dictionary
foo = {
    long_dictionary_key: value1 +
                         value2,
    ...
}

# 4-space hanging indent; nothing on first line
# 缩进4个空格,首行括号后无内容
foo = long_function_name(
    var_one, var_two, var_three,
    var_four)
meal = (
    spam,
    beans)

# 4-space hanging indent in a dictionary
foo = {
    long_dictionary_key:
        long_dictionary_value,
    ...
}
```

**No:**
```Python
# Stuff on first line forbidden
# 首行不允许有内容
foo = long_function_name(var_one, var_two,
    var_three, var_four)
meal = (spam,
    beans)

# 2-space hanging indent forbidden
foo = long_function_name(
  var_one, var_two, var_three,
  var_four)

# No hanging indent in a dictionary
foo = {
    long_dictionary_key:
    long_dictionary_value,
    ...
}
```

#### 3.4.1 关于尾后逗号
关于在一序列元素中的尾号逗号,只推荐在容器结束符号`]`,`)`或者`}`和最后元素不在同一行时使用.尾后逗号的存在也被用作我们Python代码自动格式化工具[yapf](https://github.com/google/yapf/)的提示,在`,`最后元素之后出现的时候来自动调整容器元素到每行一个元素.

**Yes:**

```Python
golomb3 = [0, 1, 3]
golomb4 = [
    0,
    1,
    4,
    6,
]
```

**No:**

```Python
golomb4 = [
    0,
    1,
    4,
    6
]
```
### 3.5 空行
在顶级定义(函数或类)之间要间隔两行.在方法定义之间以及`class`所在行与第一个方法之间要空一行,`def`行后无空行,在函数或方法内你认为合适地方可以使用单空行.

### 3.6 空格
遵守标准的空格和标点排版规则.

括号`()`,`[]`,`{}`内部不要多余的空格.

**Yes:**

```Python
spam(ham[1], {eggs: 2}, [])
```

**No:**

```Python
spam( ham[ 1 ], { eggs: 2 }, [ ] )
```

 逗号、分号、冒号前不要空格,但是在后面要加空格,除非是在行尾.
 
**Yes:**

```Python
if x == 4:
    print(x, y)
x, y = y, x
```

**No:**

```Python
if x == 4 :
    print(x , y)
x , y = y , x
```

在函数调用括号的前,索引切片括号前都不加空格.

**Yes:**

```Python
spam(1)
dict['key'] = list[index]
```

**No:**

```Python
spam (1)
dict ['key'] = list [index]
```

 行尾不要加空格.
 
 
在赋值(`=`),比较(`==`,`<`,`>`,`!=`,`<>`,`<=`,`>=`,`in`,`not in`,`is`,`is not`),布尔符号(`and`,`or`,`not`)前后都加空格.视情况在算术运算符(`+`,`-`,`*`,`/`,`//`,`%`,`**`,`@`),前后加空格

**Yes:**

```Python
x == 1
```

**No:**

```Python
x<1
```

在关键字名参数传递或定义默认参数值的时候不要在`=`前后加空格,只有一个例外:[当类型注释存在时](https://google.github.io/styleguide/pyguide.html#typing-default-values)在定义默认参数值时`=`前后加空格

**Yes:**

```Python
def complex(real, imag=0.0): return Magic(r=real, i=imag)
def complex(real, imag: float = 0.0): return Magic(r=real, i=imag)
```

**No:**

```Python
def complex(real, imag = 0.0): return Magic(r = real, i = imag)
def complex(real, imag: float=0.0): return Magic(r = real, i = imag)
```

不要用空格来做无必要的对齐,因为这会在维护时带来不必要的负担(对于`:`.`#`,`=`等等).

**Yes:**

```Python
foo = 1000  # comment
long_name = 2  # comment that should not be aligned
dictionary = {
    'foo': 1,
    'long_name': 2,
}
```

**No:**

```Python
foo       = 1000  # comment
long_name = 2     # comment that should not be aligned

dictionary = {
    'foo'      : 1,
    'long_name': 2,
}
```

### 3.7 Shebang
大部分`.py`文件不需要从`#!`行来开始.根据[PEP-394](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0394/),程序的主文件应该以`#!/usr/bin/python2`或`#!/usr/bin/python3`起始

这行被用于帮助内核找到Python解释器,但是在导入模块时会被Python忽略/只在会被直接运行的文件里有必要写.

### 3.8 注释和文档字符串
确保使用正确的模块,函数,方法的文档字符串和行内注释.

#### 3.8.1 文档字符串
Python使用*文档字符串*来为代码生成文档.文档字符串是包,模块,类或函数的首个语句.这些字符串能够自动被`__doc__`成员方法提取并且被`pydoc`使用.(尝试在你的模块上运行`pydoc`来看看具体是什么).文档字符串使用三重双引号`"""`(根据[PEP-257](https://www.google.com/url?sa=D&q=http://www.python.org/dev/peps/pep-0257/)).文档字符串应该这样组织:一行总结(或整个文档字符串只有一行)并以句号,问好或感叹号结尾.随后是一行空行,随后是文档字符串,并与第一行的首个引号位置相对齐.更多具体格式规范如下.

#### 3.8.2 模块
每个文件都应包含许可模板.选择合适的许可模板用于项目(例如
Apache 2.0,BSD,LGPL,GPL)

文档应该以文档字符串开头,并描述模块的内容和使用方法.

```Python
"""A one line summary of the module or program, terminated by a period.

Leave one blank line.  The rest of this docstring should contain an
overall description of the module or program.  Optionally, it may also
contain a brief description of exported classes and functions and/or usage
examples.

  Typical usage example:

  foo = ClassFoo()
  bar = foo.FunctionBar()
"""
```

#### 3.8.3 函数和方法
在本节,"函数"所指包括方法,函数或者生成器.

函数应有文档字符串,除非符合以下所有条件:
* 外部不可见
* 非常短
* 简明

文档字符串应该包含足够的信息以在无需阅读函数代码的情况下调用函数.文档字符串应该是叙事体(`"""Fetches rows from a Bigtable."""`)的而非命令式的(`"""Fetch rows from a Bigtable."""`),除了`@property`(应与[attribute](https://google.github.io/styleguide/pyguide.html#384-classes)使用同样的风格).文档字符串应描述函数的调用语法和其意义,而非实现.对比较有技巧的地方,在代码中使用注释更合适.

覆写了基类的方法可有简单的文档字符串向读者指示被覆写方法的文档字符串例如`"""See base class."""`.这是因为没必要在很多地方重复已经在基类的文档字符串中存在的文档.不过如果覆写的方法行为实际上与被覆写方法不一致,或者需要提供细节(例如文档中表明额外的副作用),覆写方法的文档字符串至少要提供这些差别.

一个函数的不同方面应该在特定对应的分节里写入文档,这些分节如下.每一节都由以冒号结尾的一行开始, 每一节除了首行外,都应该以2或4个空格缩进并在整个文档内保持一致(译者建议4个空格以维持整体一致).如果函数名和签名足够给出足够信息并且能够刚好被一行文档字符串所描述,那么可以忽略这些节.

[*Args:*](https://google.github.io/styleguide/pyguide.html#doc-function-args)

列出每个参数的名字.名字后应有为冒号和空格,后跟描述.如果描述太长不能够在80字符的单行内完成.那么分行并缩进2或4个空格且与全文档一致(译者同样建议4个空格)

描述应该包含参数所要求的类型,如果代码不包含类型注释的话.如果函数容许`*foo`(不定长度参数列表)或`**bar`(任意关键字参数).那么就应该在文档字符串中列举为`*foo`和`**bar`.
    
[*Returns:(或对于生成器是Yields:)*](https://google.github.io/styleguide/pyguide.html#doc-function-returns)

描述返回值的类型和含义.如果函数至少返回None,这一小节不需要.如果文档字符串以Returns或者Yields开头(例如`"""Returns row from Bigtable as a tuple of strings."""`)或首句足够描述返回值的情况下这一节可忽略.
    
[*Raises:*](https://google.github.io/styleguide/pyguide.html#doc-function-returns)

列出所有和接口相关的异常.对于违反文档要求而抛出的异常不应列出.(因为这会矛盾地使得违反接口要求的行为成为接口的一部分)

```Python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
```

#### 3.8.4 类
类定义下一行应为描述这个类的文档字符串.如果类有公共属性,应该在文档字符串中的`Attributes`节中注明,并且和[函数的`Args`](https://google.github.io/styleguide/pyguide.html#doc-function-args)一节风格统一.

```Python
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

#### 3.8.5 块注释和行注释
最后要在代码中注释的地方是代码技巧性的部分.如果你将要在下次[code review](http://en.wikipedia.org/wiki/Code_review)中揭示代码.应该现在就添加注释.在复杂操作开始前,注释几行.对于不够明晰的代码在行尾注释.

```Python
# We use a weighted dictionary search to find out where i is in
# the array.  We extrapolate position based on the largest num
# in the array and the array size and then do binary search to
# get the exact number.
if i & (i-1) == 0:  # True if i is 0 or a power of 2.
```

为了提升易读性,行注释应该至少在代码2个空格后,并以`#`后接至少1个空格开始注释部分.

另外,不要描述代码,假定阅读代码的人比你更精通Python(他只是不知道你试图做什么).

#### 3.8.6 标点,拼写和语法
注意标点,拼写和语法,写得好的注释要比写得差的好读.

注释应当是和叙事性文本一样可读,并具有合适的大小写和标点.在许多情况下,完整的句子要比破碎的句子更可读.更简短的注释如行尾的注释有时会不太正式,但是应该全篇保持风格一致.

尽管被代码审核人员指出在应该使用分号的地方使用了逗号是很令人沮丧的,将源代码维护在高度清楚可读的程度是很重要的.合适的标点,拼写和语法能够帮助达到这个目标.

### 3.9 类
如果类并非从其他基类继承而来,那么就要明确是从`object`继承而来,即便内嵌类也是如此.

**Yes:**

```Python
class SampleClass(object):
    pass

class OuterClass(object):
    class InnerClass(object):
        pass

class ChildClass(ParentClass):
    """Explicitly inherits from another class already."""
```

**No:**

```Python
class SampleClass:
    pass

class OuterClass:
    class InnerClass:
        pass
```

从`object`类继承保证了属性能够在Python2正确运行并且保护代码在Python3下出现潜在的不兼容.这样也定义了object包括`__new__`,`__init__`,`__delattr__`,`__getattribute__`,`__setattr__`,`__hash__`,`__repr__`,和`__str__`等默认特殊方法的实现.

### 3.10 字符串
使用`format`或`%`来格式化字符串,即使参数都是字符串对象,也要考虑使用`+`还是`%`及`format`.

**Yes:**

```Python
x = a + b
x = '%s, %s!' % (imperative, expletive)
x = '{}, {}'.format(first, second)
x = 'name: %s; score: %d' % (name, n)
x = 'name: {}; score: {}'.format(name, n)
x = f'name: {name}; score: {n}'  # Python 3.6+
```

**No:**

```Python
employee_table = '<table>'
for last_name, first_name in employee_list:
    employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
employee_table += '</table>'
```

避免使用`+`和`+=`操作符来在循环内累加字符串,因为字符串是不可变对象.这会造成不必要的临时变量导致运行时间以四次方增长而非线性增长.应将每个字符串都记入一个列表并使用`''.join`来将列表在循环结束后连接(或将每个子字符串写入`io.BytesIO`缓存)

**Yes:**

```Python
items = ['<table>']
for last_name, first_name in employee_list:
    items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
items.append('</table>')
employee_table = ''.join(items)
```

**No:**

```Python
employee_table = '<table>'
for last_name, first_name in employee_list:
    employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
employee_table += '</table>'
```

在同一个文件内,字符串引号要一致,选择`''`或者`""`并且不要改变.对于需要避免`\\`转义的时候,可以更改.

**Yes:**

```Python
Python('Why are you hiding your eyes?')
Gollum("I'm scared of lint errors.")
Narrator('"Good!" thought a happy Python reviewer.')
```

**No:**

```Python
Python("Why are you hiding your eyes?")
Gollum('The lint. It burns. It burns us.')
Gollum("Always the great lint. Watching. Watching.")
```

多行字符串多行字符串优先使用"""而非`'''`,当且只当对所有非文档字符串的多行字符串都是用`'''`而且对正常字符串都使用`'`时才可使用三单引号.docstring不论如何必须使用`"""`

多行字符串和其余代码的缩进方式不一致.如果需要避免在字符串中插入额外的空格,要么使用单行字符串连接或者带有[`textwarp.dedent()`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent)的多行字符串来移除每行的起始空格.

**No:**

```Python
long_string = """This is pretty ugly.
Don't do this.
"""
```

**Yes:**

```Python
long_string = """This is fine if your use case can accept
    extraneous leading spaces."""

long_string = ("And this is fine if you can not accept\n" +
               "extraneous leading spaces.")

long_string = ("And this too is fine if you can not accept\n"
               "extraneous leading spaces.")

import textwrap

long_string = textwrap.dedent("""\
    This is also fine, because textwrap.dedent()
    will collapse common leading spaces in each line.""")
```

### 3.11 文件和socket
当使用结束后显式地关闭文件或socket.

不必要地打开文件，socket或其他类似文件的对象有很多弊端：

* 他们可能会消耗有限的系统资源,例如文件描述符.如果在使用没有即使归还系统,处理很多这样对象的代码可能会浪费掉很多不应浪费的资源.
* 保持一个文件可能会阻止其他操作诸如移动或删除.
* 被程序共享的文件和socket可能会无意中在逻辑上已被关闭的情况下仍被读写.如果实际上已经关闭,试图读写的操作会抛出异常,这样就可以立即发现问题.

此外,当文件或socket在文件对象被销毁的同时被自动关闭的时候,是不可能将文件的生命周期和文件状态绑定的:

* 不能保证何时会真正将文件对象销毁.不同的Python解释器使用的内存管理技术不同,例如延时垃圾处理可能会让对象的生命周期被无限期延长.
* 可能导致意料之外地对文件对象的引用,例如在全局变量或者异常回溯中,可能会让文件对象比预计的生命周期更长.

推荐使用[with语句](http://docs.python.org/reference/compound_stmts.html#the-with-statement)管理文件:

```Python
with open("hello.txt") as hello_file:
    for line in hello_file:
        print(line)
```

对于类似文件的对象,如果不支持with语句的可以使用`contextlib.closing()`:

```Python
import contextlib

with contextlib.closing(urllib.urlopen("http://www.python.org/")) as front_page:
    for line in front_page:
        print(line)
```

### 3.12  TODO注释
对于下述情况使用`TODO`注释:临时的,短期的解决方案或者足够好但是不完美的解决方案.

`TODO`注释以全部大写的字符串`TODO`开头,并带有写入括号内的姓名,email地址,或其他可以标识负责人或者包含关于问题最佳描述的issue.随后是这里做什么的说明.

有统一风格的`TODO`的目的是为了方便搜索并了解如何获取更多相关细节.`TODO`并不是保证被提及者会修复问题.因此在创建`TODO`注释的时候,基本上都是给出你的名字.

```Python
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

如果`TODO`注释形式为"未来某个时间点会做什么事"的格式,确保要么给出一个非常具体的时间点(例如"将于2009年11月前修复")或者给出一个非常具体的事件(例如"当所有客户端都能够处理XML响应时就移除此代码").

### 3.13 import格式
imports应该在不同行.例如:

**Yes:**

```Python
import os
import sys
```

**No:**

```Python
import os, sys
```

import应集中放在文件顶部,在模块注释和docstring后面,模块globals和常量前面.应按照从最通用到最不通用的顺序排列分组:

1. Python未来版本import语句,例如:    
    
    >   ```Python
    >    from __future__ import absolute_import
    >    from __future__ import division
    >    from __future__ import print_function
    >    ```
    
    >    更多信息参看[上文](https://google.github.io/styleguide/pyguide.html#from-future-imports)
        
2. Python标准基础库import,例如:
    
    >    ```Python
    >    import sys
    >    ```

3. 第三方库或包的import,例如:
    
    >    ```Python
    >    import tensorflow as tf
    >    ```
    
4. 代码库内子包import,例如:
    
    >    ```Python
    >    from otherproject.ai import mind
    >    ```
    
5. **此条已弃用**:和当前文件是同一顶级子包专用的import,例如:
    >    ```Python
    >    from myproject.backend.hgwells import time_machine
    >    ```
    
    在旧版本的谷歌Python代码风格指南中实际上是这样做的.但是现在不再需要了.**新的代码风格不再受此困扰.**简单的将专用的子包import和其他子包import同一对待即可.

在每个组内按照每个模块的完整包路径的字典序忽略大小写排序.可以根据情况在每个节质检增加空行.
```Python
import collectionsimport queueimport sys

from absl import appfrom absl import flagsimport bs4import cryptographyimport tensorflow as tf

from book.genres import scififrom myproject.backend.hgwells import time_machinefrom myproject.backend.state_machine import main_loopfrom otherproject.ai import bodyfrom otherproject.ai import mindfrom otherproject.ai import soul

# Older style code may have these imports down here instead:
# 旧版本代码风格可能会采用下述import方式
# from myproject.backend.hgwells import time_machine
# from myproject.backend.state_machine import main_loop
```

### 3.14 语句
每行只有一条语句.

不过如果测试语句和结果能够在一行内放下,就可以放在一行内.但是不允许将`try`/`except`语句和对应内容放于一行,因为`try`或者`except`都不能在一行内完成.对于没有else的if语句可以将`if`和对应内容合并到一行.

**Yes:**

```Python
if foo: bar(foo)
```

**No:**

```Python
if foo: bar(foo)
else:   baz(foo)

try:               bar(foo)
except ValueError: baz(foo)

try:
    bar(foo)
except ValueError: baz(foo)
```

### 3.15 访问
对于琐碎又不太重要的访问函数,应用公共变量来替代访问函数,以避免额外的程序调用消耗,当添加了更多函数功能时,使用`property`来保持连续性

此外,如果访问过于复杂,或者访问变量的消耗过大,应该使用诸如`get_foo()`和`set_foo()`之类的函数式访问(参考[命名](https://google.github.io/styleguide/pyguide.html#s3.16-naming)指南).如果过去的访问方式是通过属性,新访问函数不要绑定到property上,这样使用property的旧方式就会失效,使用者就会知道函数有变化.

### 3.16 命名
`module_name`,`package_name`,`ClassName`,`method_name`,`ExceptionName`,`function_name`,`GLOBAL_CONSTANT_NAME`,`global_var_name`,`instance_var_name`,`function_parameter_name`,`local_var_name`.

命名函数名,变量名,文件名应该是描述性的,避免缩写,尤其避免模糊或对读者不熟悉的缩写.并且不要通过删减单词内的字母来缩短.

使用`.py`作为文件拓展名,不要使用横线.

#### 3.16.1 要避免的名字：
* 单字符名字,除非是计数或迭代元素,e可以作为Exception捕获识别名来使用..
* `-`横线,不应出现在任何包名或模块名内
* `__double_leading_and_trailing_underscore__`首尾都双下划线的名字,这种名字是python的内置保留名字

#### 3.16.4 命名约定
*  internal表示仅模块内可用、或者类内保护的或者私有的
* 单下划线(`_`)开头表示是被保护的(`from module import *`不会import).双下划线(`__`也就是"dunder")开头的实例变量或者方法表示类内私有(使用命名修饰).我们不鼓励使用,因为这会对可读性和可测试性有削弱二期`并非真正`的私有.
* 相关的类和顶级函数放在同一个模块内,不必像是Java一样要一个类放在一个模块里.
* 对类名使用大写字母(如CapWords)开头的单词,命名,模块名应该使用小写加下划线的方式.尽管有一些旧的模块命名方式是大写字母的(如CapWords.py),现在不鼓励这样做了,因为在模块刚好是从某个类命名出发的时候可能会令人迷惑(例如是选择`import StringIO`还是`from StringIO import StringIO`?)
* 在*unittest*方法中可能是`test`开头来分割名字的组成部分,即使这些组成部分是使用大写字母驼峰式的.这种方式是可以的： `test<MethodUnderTest>_<state>`例如`testPop_EmptyStack`,对于命名测试方法没有明确的正确方法.

#### 3.16.3 文件名
文件拓展名必须为`.py`,不可以包含`-`.这保证了能够被正常import和单元测试.如果希望一个可执行文件不需要拓展名就可以被调用,那么建立一个软连接或者一个简单的bash打包脚本包括`exec "$0.py" "$@"`.

#### 3.16.4 Guido的指导建议


| **类型** | **公共** | **内部** |
| --- | --- | --- |
| 包 | `lower_with_under` |  |
| 模块 | `lower_with_under` | `_lower_with_under` |
| 类 | `CapWords` | `_CapWords` |
| 异常 | `CapWords` |  |
| 函数 | `lower_with_under()` | `_lower_with_under()` |
| 全局/类常量 | `CAPS_WITH_UNDER` | `_CAPS_WITH_UNDER` |
| 全局/类变量 | `lower_with_under` | `_lower_with_under` |
| 实例变量 | `lower_with_under` | `_lower_with_under`(受保护) |
| 方法名 | `lower_with_under()` | `_lower_with_under()`(受保护) |
| 函数/方法参数 | `lower_with_under` |  |
| 局部变量 | `lower_with_under` |  |

尽管Python支持通过双下划线`__`(即"dunder")来私有化.不鼓励这样做.优先使用单下划线.单下划线更易于打出来、易读、易于小的单元测试调用.Lint的警告关注受保护成员的无效访问.


### 3.17 Main
即便是一个用做脚本的py文件也应该是可以被import的,而只用于import时,也不应有执行了主函数的副作用.主函数的功能应该被放在`main()`里.

在Python中,`pydoc`和单元测试要求模块是可import的.所以代码在主程序执行前应进行`if __name__ == '__main__':`检查,以防止模块在import时被执行.

```Python
def main():
    ...

if __name__ == '__main__':
    main()
```

所有顶级代码在模块被import时执行.因而要小心不要调用函数,创建对象或者执行其他在执行`pydoc`时不应该被执行的操作.

### 3.18 函数长度
优先写小而专一的函数.

长函数有时候是合适的,故而函数长度没有固定的限制.但是超过40行的时候就要考虑是否要在不影响程序结构的前提下分解函数.

尽管长函数现在运行的很好,但是在之后的时间里其他人修改函数并增加新功能的时候可能会引入新的难以发现的bug,保持函数的简短,这样有利于其他人读懂和修改代码.

在处理一些代码时,可能会发现有些函数长而且复杂.不要畏惧调整现有代码,如果处理这个函数非常困难,如难以对报错debug或者希望在几个不同的上下文中使用它,那么请将函数拆解成若干个更小更可控的片段.

### 3.19 类型注释

#### 3.19.1 基本规则

* 熟悉[PEP-484](https://www.python.org/dev/peps/pep-0484/)
* 在方法中,只在必要时给`self`或者`cls`增加合适的类型信息.例如`@classmethod def create(cls: Type[T]) -> T: return cls()`
* 如果其他变量或返回类型不定,使用`Any`
* 不需要注释每个函数
    * 至少需要注明公共接口
    * 使用类型检查来在安全性和声明清晰性以及灵活性之间平衡
    * 标注容易因类型相关而抛出异常的代码(previous bugs or complexity,此处译者认为是与上一条一致,平衡安全性和复杂性)
    * 标注难理解的代码
    * 标注类型稳定的代码,成熟稳定的代码可以都进行标注而不会影响其灵活性

#### 3.19.2 分行
遵循现有的[缩进](https://google.github.io/styleguide/pyguide.html#indentation)规范

标注类型后,函数签名多数都要是"每行一个参数".

```Python
def my_method(self,
              first_var: int,
              second_var: Foo,
              third_var: Optional[Bar]) -> int:
  ...
```

优先在变量之间换行,而非其他地方(如变量名和类型注释之间).如果都能放在一行内,就放在一行.

```Python
def my_method(self, first_var: int) -> int:
  ...
```

如果函数名,一直到最后的参数以及返回类型注释放在一行过长,那么分行并缩进4个空格.

```Python
def my_method(
    self, first_var: int) -> Tuple[MyLongType1, MyLongType1]:
  ...
```

当返回值类型不能和最后一个参数放入同一行,比较好的处理方式是将参数分行并缩进4个空格,右括号和返回值类型换行并和`def`对齐.

```Python
def my_method(
    self, other_arg: Optional[MyLongType]
) -> Dict[OtherLongType, MyLongType]:
  ...
```

pylint允许您将右括号移动到新行并与左括号对齐,但这不太容易理解.

**No:**

```
def my_method(self,
              other_arg: Optional[MyLongType]
             ) -> Dict[OtherLongType, MyLongType]:
  ...
```

就像上面的例子一样,尽量不要分割类型注释,不过有时类型注释太长无法放入一行,(那就尽量让子注释不要被分割).

```Python
def my_method(
    self,
    first_var: Tuple[List[MyLongType1],
                     List[MyLongType2]],
    second_var: List[Dict[
        MyLongType3, MyLongType4]]) -> None:
  ...
```

如果某个命名和类型太长了,考虑使用别名.如果没有其他解决方案,在冒号后分行缩进4个空格.

**Yes:**

```Python
def my_function(
    long_variable_name:
        long_module_name.LongTypeName,
) -> None:
  ...
```

**No:**

```Python
def my_function(
    long_variable_name: long_module_name.
        LongTypeName,) -> None:
  ...
```

#### 3.19.3 前置声明
如果需要同一模块内还未定义的类名,例如需要类声明内部的类,或者需要在后续代码中定义的类,那么使用类名的字符串来代替.

```Python
class MyClass(object):

  def __init__(self,
               stack: List["MyClass"]) -> None:
```

#### 3.19.4 默认值
参考[PEP-008](https://www.python.org/dev/peps/pep-0008/#other-recommendations),只有在同时需要类型注释和默认值的时候在`=`前后都加空格

**Yes:**

```Python
def func(a: int = 0) -> int:
  ...
```

**No:**

```Python
def func(a:int=0) -> int:
  ...
```

#### 3.19.5 NoneType
在Python系统中`NoneType`是一等类型,为了方便输入,`None`是`NoneType`的别名.如果一个参数可以是`None`,那么就需要声明!可以使用`Union`,但如果只有一个其他类型,那么使用`Optional`.

显式地使用`Optional`而非隐式地.PEP 484的早期版本容许`a: Text = None`被解释为`a: Optional[Text] = None`.但现在已经不推荐这样使用了.

**Yes:**

```Python
def func(a: Optional[Text], b: Optional[Text] = None) -> Text:
  ...
def multiple_nullable_union(a: Union[None, Text, int]) -> Text
  ...
```

**No:**

```Python
def nullable_union(a: Union[None, Text]) -> Text:
  ...
def implicit_optional(a: Text = None) -> Text:
  ...
```

#### 3.19.6 类型别名
可以对复杂类型声明别名,别名的名称应为CapWorded,如果只用于当前模块,应加下划线私有化.

例如,如果带有模块名的类型名过长:

```Python
_ShortName = module_with_long_name.TypeWithLongName
ComplexMap = Mapping[Text, List[Tuple[int, int]]]
```

其他示例是复杂的嵌套类型和一个函数的多个返回变量（作为元组）.

#### 3.19.7 忽略类型检查
可以通过增加特殊行注释`# type: ignore`来禁止类型检查.

`pytype`对于明确的报错有关闭选项(类似于lint):

```Python
# pytype: disable=attribute-error
```

#### 3.19.8 对变量注释类型
对变量标注类型如果内部变量很难或者不可能指向,可以使用下述方式：

[*类型注释*](https://google.github.io/styleguide/pyguide.html#type-comments):

在行尾增加以`# type`开头的注释

```Python
a = SomeUndecoratedFunction()  # type: Foo
```

[*注释绑定*](https://google.github.io/styleguide/pyguide.html#annotated-assignments):

在变量名和赋值之间用冒号和类型注明,和函数参数一致.

```Python
a: Foo = SomeUndecoratedFunction()
```

#### 3.19.9 元组和列表
不像是列表只能包含单一类型,元组可以既只有一种重复类型或者一组不同类型的元素,后者常用于函数返回.

```Python
a = [1, 2, 3]  # type: List[int]
b = (1, 2, 3)  # type: Tuple[int, ...]
c = (1, "2", 3.5)  # type: Tuple[int, Text, float]
```

#### 3.19.10 TypeVars
Python是有[泛型](https://www.python.org/dev/peps/pep-0484/#generics)的,工厂函数`TypeVar`是通用的使用方式.

例子:

```Python
from typing import List, TypeVar
T = TypeVar("T")
...
def next(l: List[T]) -> T:
  return l.pop()
```

TypeVar可以约束类型:

```Python
AddableType = TypeVar("AddableType", int, float, Text)
def add(a: AddableType, b: AddableType) -> AddableType:
    return a + b
```

在`typing`模块预定义好的类型变量是`AnyStr`,用于针对字符串可以是`bytes`也可为`unicode`并且保持一致的多个类型注释.

```
from typing import AnyStr
def check_length(x: AnyStr) -> AnyStr:
  if len(x) <= 42:
    return x
  raise ValueError()
```

#### 3.19.11 字符串类型
注释字符串的合适类型是基于Python版本的.

对于只有Python3的代码,使用`str`,`Text`可以用但是在选择上保持一致.

对于Python2兼容的代码,用`Text`,在一些很罕见的情况下,`str`可能可用.当在不同Python版本之间返回值类型不同的时候通常是为了照顾兼容性.避免使用`unicode`,因为Python3中不存在.

**No:**

```Python
def py2_code(x: str) -> unicode:
  ...
```

对于处理二进制数据的代码,请使用`bytes`.

**Yes:**

```Python
def deals_with_binary_data(x: bytes) -> bytes:
  ...
```

对于Python2兼容,处理文本数据(Python中`str`或`unicode`,Python3中`str`)的代码,使用`Text`.对于只有Python3的代码,优先使用`str`.

```Python
from typing import Text
...
def py2_compatible(x: Text) -> Text:
  ...
def py3_only(x: str) -> str:
  ...
```

如果既可以是byte也可以是文本,那么使用`Union`和合适的文本类型.

```Python
from typing import Text, Union
...
def py2_compatible(x: Union[bytes, Text]) -> Union[bytes, Text]:
  ...
def py3_only(x: Union[bytes, str]) -> Union[bytes, str]:
  ...
```

如果一个函数中所有的字符串类型始终一致,例如前文例子中返回值类型和参数类型是一致的,那么使用[`AnyStr`](https://google.github.io/styleguide/pyguide.html#typing-type-var)

像这样写能够简化代码向Python3的迁移过程.

#### 3.19.12 typing的import
对于从`typing`模块import的类,要import类本身.明确的允许在一行内从`typing`模块import多个特定的类,如

```Python
from typing import Any, Dict, Optional
```

这种从`typing`模块import的方式会向命名空间内增加额外项,`typing`中的任何命名都应该和关键字同等对待并且不在你的Python代码中定义,typed or not(译者推测文无论是否引入).如果和已有的命名冲突,使用`import x as y`来import.

```Python
from typing import Any as AnyType
```

#### 3.19.13 条件import
只在运行时一定要避免进行类型检查的情况下使用条件import.不鼓励使用这种模式.鼓励使用其他替代方式诸如重构代码以容许顶级import.

只用于类型注释的import可以被归于`if TYPE_CHECKING:`代码块中.
* 条件import的类型应被视为字符串引用,以和Python3.6兼容(在Python3.6中,注释表达式实际上被赋值的).
* 只有单独用于类型注释的实例才能在这里定义,包括了别名.否则将会报运行错误因为在运行时这些模块不会被引用.
* 代码块应该紧跟在正常import后面.
* 在类型import后不应有空行
* 按照正常import顺序对这一块代码进行排序

```Python
import typing
if typing.TYPE_CHECKING:
    import sketch
def f(x: "sketch.Sketch"): ...
```

#### 3.19.14 循环依赖
由于类型检查引发的循环依赖是一种code smell([代码异味](https://zh.wikipedia.org/zh-hans/%E4%BB%A3%E7%A0%81%E5%BC%82%E5%91%B3)),这样的代码应当被重构.尽管技术上是可以保留循环引用的.[build system](https://google.github.io/styleguide/pyguide.html#typing-build-deps)(系统)不允许这样做因为每个模块都要依赖于其他模块.

将造成循环依赖的模块替换为`Any`并赋予一个有意义的[别名](https://google.github.io/styleguide/pyguide.html#typing-aliases)并使用从这个模块导入的真实类名(因为任何`Any`的属性都是`Any`).别名的定义用和最后一行import用一行空行分隔.

```Python
from typing import Any

some_mod = Any  # some_mod.py imports this module.
...

def my_method(self, var: some_mod.SomeType) -> None:
  ...
```

#### 3.19.15  泛型
当注释的时候,优先泛型类型专有类型参数,否则[泛型的参数会被认为是`Any`](https://www.python.org/dev/peps/pep-0484/#the-any-type).

```Python
def get_names(employee_ids: List[int]) -> Dict[int, Any]:
  ...
```
```Python
# These are both interpreted as get_names(employee_ids: List[Any]) -> Dict[Any, Any]
def get_names(employee_ids: list) -> Dict:
  ...

def get_names(employee_ids: List) -> Dict:
  ...
```

如果泛型最佳的参数类型是`Any`也将其显式地表示出来.但是在很多情况下[`TypeVar`](https://google.github.io/styleguide/pyguide.html#typing-type-var)可能更合适.

```Python
def get_names(employee_ids: List[Any]) -> Dict[Any, Text]:
"""Returns a mapping from employee ID to employee name for given IDs."""
```

```Python
T = TypeVar('T')
def get_names(employee_ids: List[T]) -> Dict[T, Text]:
"""Returns a mapping from employee ID to employee name for given IDs."""
```

## 4 最后的话
***保持一致***

如果你在编辑代码,花几分钟看看现有代码然后决定好要使用哪种风格.如果现有代码在所有算术运算符两侧都加了空格,那么你也应该如此.如果现有的注释用井号组成了包围框,那么你的注释也应如此.

有代码风格指南的目的是有一个编程的共识,这样人们能够集中在内容而非形式上.我们将通用的代码风格指南公布于此这样人们就能了解这个共识(译者:有巴别塔的意味.)但是各自的代码风格也很重要.如果你添加的代码与原有代码看起来完全不一致,就会打乱读者的阅读节奏.避免这样.

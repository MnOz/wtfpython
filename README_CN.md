[TOC]

# WTFPL 2.0

![logo][1]

> An interesting collection of surprising snippets and lesser-known Python features.
>
> [source](https://github.com/satwikkansal/wtfpython)
>

## 前言

Python，一个精心设计的解释执行的高阶语言，提供了许多功能。但有时候，一个 Python 代码片段的结果，普通用户一眼看上去并不明显。

这里有一个有趣的项目来收集这些棘手的问题，反直觉的例子和 Python 中鲜为人知的特性，试图讨论这些现象的背后到底发生了什么。

虽然下面看到的一些例子可能不是真正意义上的 WTF，但是它们会揭示一些 Python 中您可能没有意识到的有趣的部分。我觉得这是学习编程语言内部实现的好方法，我想你也会发现它们很有趣！

如果你是一位经验丰富的 Python 程序员，可以把它作为一个挑战，在你的第一次尝试中解释它们。你可能已经对一些例子已经很熟悉了，希望我可以能够恢复你最初被这些陷阱困惑时的甜蜜回忆：sweat_smile。

如果你是一个回头客，你可以在[这里][2]了解到新的修改。

好的，开始吧...

## 示例结构

# Structure of the Examples

All the examples are structured like below:

> `###` 标题 *
> 标题后的 `*` 表示这个例子是在后续版本增加的，并没有出现在第一版。
>
> ```py
> # Setting up the code.
> # Preparation for the magic...
> ```
>
> __Output (Python version):__
> ```py
> >>> triggering_statement
> Probably unexpected output
> ```
> (Optional): One line describing the unexpected output.
>
>
> #### 解释
>
> * 简要说明发生了什么，为什么会这样。
>   ```py
>   Setting up examples for clarification (if necessary)
>   ```
>   __Output:__
>   ```py
>   >>> trigger # some example that makes it easy to unveil the magic
>   # some justified output
>   ```

__注意:__ 所有的例子都在 Python 3.5.2 交互式模式下测试通过，他们应该在所有的 Python 版本下成立除非有特殊说明。

## 用法

在我看来，最好是按顺序阅读每一个例子，并且对每一个例子:
- 仔细阅读示例代码。如果你是一位有经验的 Python 程序员，大部分情况下你都能预见接下来会发生什么。
- 阅读输出的结果，然后
    - 检查输出是否与您预期的相同。
    - 确保你知道之所以产生这种结果的背后的确切原因。
        - 如果不是，深吸一口气，并阅读解释(如果你还不明白，大声呼喊，并在[这里][3]创建一个 issue)。
        - 如果是的话，放松一下，你可以跳到下一个例子。

PS：你也可以在命令行阅读 WTFpython。有一个 pypi 包和一个 npm 包(支持彩色格式)。

安装 npm 包 [`wtfpython`][4]

```
$ npm install -g wtfpython
```

或者，安装 pypi 包的 [`wtfpython`][4]

```
$ pip install wtfpython -U
```

现在，只需在命令行运行 `wtfpython`，它将在您选择的 `$PAGER` 中打开。

---

# 例子

## Section: Strain your brain!

### Strings can be tricky sometimes *

1.

```py
>>> a = "some_string"
>>> id(a)
140420665652016
>>> id("some" + "_" + "string") # Notice that both the ids are same.
140420665652016
```

2.

```py
>>> a = "wtf"
>>> b = "wtf"
>>> a is b
True

>>> a = "wtf!"
>>> b = "wtf!"
>>> a is b
False

>>> a, b = "wtf!", "wtf!"
>>> a is b
True
```

3.
```py
>>> 'a' * 20 is 'aaaaaaaaaaaaaaaaaaaa'
True
>>> 'a' * 21 is 'aaaaaaaaaaaaaaaaaaaaa'
False
```

有道理吧？

#### 解释

+ 这种行为是由于 CPython 优化(称为 _string interning_)，在某些情况下试图使用已有的不可变对象，而不是每次创建一个新的对象。
+ 在 "interned" 后，许多变量可能指向内存中的同一个字符串对象(从而节省内存)。
+ 在上面的代码片段中，字符串被隐式地 "interned"。一个字符串是否 "interned" 取决于实现。有一些事实可以用来猜测一个字符串是否会被 "interned"：
    - 所有长度为 0 和 1 的字符串被 "interned"。
    - 字符串在编译时被 "interned"('wtf' 将被 "interned"，`''.join(['w'，'t'，'f']` 不会被 "interned")
    - 不是由 ASCII 字母、数字或下划线组成的字符串不会被 "interned"。这就解释了为什么 `'wtf!'` 不被 "interned" - 因为 `!`。这个规则的 Cython 实现可以在[这里](https://github.com/python/cpython/blob/3.6/Objects/codeobject.c#L19)找到。
+ 当变量 `a` 和 `b` 在同一行中被赋值为 `"wtf!"` 时，Python 解释器创建一个新的对象，然后同时引用。如果在不同的行上做，它不会**知道**已经存在对象 `"wtf!"` (因为 `"wtf!"` 并不是按照上面提到的条款被隐含地 "interned")。这是一个编译器优化，特别适用于交互式环境。
+ 常量折叠（constant folding）是 Python 中 [peephole optimization](https://en.wikipedia.org/wiki/Peephole_optimization) 使用的一种技术。这意味着在编译期间，表达式 `'a'*20` 被替换为 `'aaaaaaaaaaaaaaaaaaaa'`，以提高运行时的效率。但常量折叠只在字符串长度小于 20 时才会发生。（为什么？想想一下 `'a'*10**10` 产生的 `.pyc` 文件的大小）。[这里](https://github.com/python/cpython/blob/3.6/Python/peephole.c#L288)是相同实现的源码。

![string_intern][6]

---

### Time for some hash brownies!

1.

```py
some_dict = {}
some_dict[5.5] = "Ruby"
some_dict[5.0] = "JavaScript"
some_dict[5] = "Python"
```

**Output:**
```py
>>> some_dict[5.5]
"Ruby"
>>> some_dict[5.0]
"Python"
>>> some_dict[5]
"Python"
```

"Python" destroyed the existence of "JavaScript"?

#### 解释

* Python 字典通过检查 key 的值以及哈希确定其是否相等。
* Immutable objects with same value always have the same hash in Python.
* 拥有相同(相等)的值的不变对象总是拥有相同的哈希值。
  ```py
  >>> 5 == 5.0
  True
  >>> hash(5) == hash(5.0)
  True
  ```
  __注意:__ 拥有不同的值的对象同样可能有相同的哈希值(哈希冲突)
* 当执行 `some_dict[5] = "Python"` 语句时，已有的 "JavaScript" 被 "Python" 覆盖，因为 Python 认为 `5` 和 `5.0` 在 `some_dict` 中是相同的 key。
* StackOverflow 中的[这个答案][7]很好的解释了它背后的原理。

---

### Return return everywhere!

```py
def some_func():
    try:
        return 'from_try'
    finally:
        return 'from_finally'
```

**Output:**

```py
>>> some_func()
'from_finally'
```

#### 解释

- 当在 `try ... finally` 语句的 `try` 子句中执行 `return`, `break` 或 `continue` 语句时，`finally` 子句也会被执行。
- 函数的返回值由最后执行的 `return` 语句决定。由于 `finally` 子句总是执行，因此在 `finally` 子句中执行的 `return` 语句将是最后执行的语句。

---

### Deep down, we're all the same. *

```py
class WTF:
  pass
```

**Output:**

```py
>>> WTF() == WTF() # two different instances can't be equal
False
>>> WTF() is WTF() # identities are also different
False
>>> hash(WTF()) == hash(WTF()) # hashes _should_ be different as well
True
>>> id(WTF()) == id(WTF())
True
```

#### 解释

- 当 `id` 被调用时，Python 创建了一个 `WTF` 类对象并将其传递给 `id` 函数。`id` 函数使用其 `id`(它的内存地址)，并抛弃该对象。对象被析构。
- 当我们连续两次执行这个操作时，Python 也为第二个对象分配相同的内存位置。由于(在 CPython 中) `id` 使用内存位置作为对象 `id`，所以两个对象的 `id` 是相同的。
- 所以，对象的 `id` 在对象的生命周期内是唯一的。在对象被销毁之后，或者在被创建之前，别的东西可以拥有相同的 `id`。
- 但为什么运算符 `is` 为 `False`？让我们看看这个片段。
  ```py
  class WTF(object):
      def __init__(self): print("I ")
      def __del__(self): print("D ")
  ```

  **Output:**
  ```py
  >>> WTF() is WTF()
  I I D D
  >>> id(WTF()) == id(WTF())
  I D I D
  ```

就像你看到的，对象的销毁顺序使这两个结果不同。

还有
```py
class b:
    pass

>>> id(WTF())
139966559454704
>>> id(b())
139966559454704
```

---

### For what?

```py
some_string = "wtf"
some_dict = {}
for i, some_dict[i] in enumerate(some_string):
    pass
```

**Output:**
```py
>>> some_dict # An indexed dict is created.
{0: 'w', 1: 't', 2: 'f'}
```

#### 解释

* Python 中的 `for` [语法][8]类似如下定义:
  ```
  for_stmt: 'for' exprlist 'in' testlist ':' suite ['else' ':' suite]
  ```
  其中 `exprlist` 是赋值目标。这意味着对__迭代器中的每个元素执行__相同的 `{exprlist} = {next_value}`。
  一个有趣的例子说明了这一点:
  ```py
  for i in range(4):
      print(i)
      i = 10
  ```

  **Output:**
  ```
  0
  1
  2
  3
  ```

  你希望循环体只执行一次？

  **解释**

  + 赋值语句 `i = 10` 不会影响循环的迭代，因为循环在 Python 中的工作方式如下: 在每次迭代开始之前，迭代器(本例为 `range(4)`)提供的下一个元素将被解包并分赋值给目标列表变量(在本例中为 `i`)。

- `enumerate(some_string)` 函数在每次迭代中产生一个 `i` 的新值(一个计数器)和一个来自 `some_string` 的字符。然后它将该字符赋值给 `some_dict[i]`。循环的展开可以简化为:
  ```py
  >>> i, some_dict[i] = (0, 'w')
  >>> i, some_dict[i] = (1, 't')
  >>> i, some_dict[i] = (2, 'f')
  >>> some_dict
  ```

---

### Evaluation time discrepancy

1.

```py
array = [1, 8, 15]
g = (x for x in array if array.count(x) > 0)
array = [2, 8, 22]
```

**Output:**
```py
>>> print(list(g))
[8]
```

2.
```
array_1 = [1,2,3,4]
g1 = (x for x in array_1)
array_1 = [1,2,3,4,5]

array_2 = [1,2,3,4]
g2 = (x for x in array_2)
array_2[:] = [1,2,3,4,5]
```

**Output:**
```py
>>> print(list(g1))
[1,2,3,4]

>>> print(list(g2))
[1,2,3,4,5]
```

#### 解释

- 在[生成器][9]表达式中，`in` 子句在声明时求值，而条件子句在运行时求值。
- 所以，在运行之前，`array` 被重新赋值为列表 `[2, 8, 22]`，又因为在 `1`, `8` 和 `15` 中，只有 `8` 的 `count` 大于 `0`，所以生成器只产生 `8`。
- 在第二个例子中， `g1` 和 `g2` 输出不同是因为 `array_1` 和 `array_2` 被重新赋值。
- In the first case, `array_1` is binded to the new object `[1,2,3,4,5]` and since the `in` clause is evaluated at the declaration time it still refers to the old object `[1,2,3,4]` (which is not destroyed).
- `array_1` 绑定到一个新的对象 `[1,2,3,4,5]` 上，又因为 `in` 子句在声明时求值，所以 `g1` 仍然引用对象 `[1,2,3,4]`（并没有被销毁）。
- `array_2` 则是通过切片，然后更新就对象 `[1,2,3,4]` 为 `[1,2,3,4,5]`。因此 `g2` 和 `array_2` 仍然引用相同的对象（已经更新为 `[1,2,3,4,5]`）。

---

### `is` is not what it is!

以下是在互联网上非常有名的例子:

```py
>>> a = 256
>>> b = 256
>>> a is b
True

>>> a = 257
>>> b = 257
>>> a is b
False

>>> a = 257; b = 257
>>> a is b
True
```

#### 解释

__`is` 和 `==` 的区别__

- `is` 运算符检查两个操作数是否指向同一个对象(i.e., 它检查操作数的 identity 是否匹配)。
- `==` 运算符比较两个操作数的值是否相等。
- 所以 `is` 是引用相同而 `==` 是指相等。一个例子说明问题:
  ```py
  >>> [] == []
  True
  >>> [] is [] # These are two empty lists at two different memory locations.
  False
  ```

__`256` 是一个已有的对象，但 `257` 不是__

当启动 Python 时，数字 `-5` 到 `256` 会预先分配。这些数会经常使用，所以这样做是有意义的。

以下引用自 [python c-api][10]。
> The current implementation keeps an array of integer objects for all integers between -5 and 256, when you create an int in that range you just get back a reference to the existing object. So it should be possible to change the value of 1. I suspect the behavior of Python, in this case, is undefined. :-)

```py
>>> id(256)
10922528
>>> a = 256
>>> b = 256
>>> id(a)
10922528
>>> id(b)
10922528
>>> id(257)
140084850247312
>>> x = 257
>>> y = 257
>>> id(x)
140084850247440
>>> id(y)
140084850247344
```

在这里，解释器在执行 `y = 257` 时，还没有足够智能到识别出已经存在一个整数值 `257`，所以它继续在内存中创建另一个对象。

__当 `a` 和 `b` 在同一行初始化时，他们引用相同的对象__

```py
>>> a, b = 257, 257
>>> id(a)
140640774013296
>>> id(b)
140640774013296
>>> a = 257
>>> b = 257
>>> id(a)
140640774013392
>>> id(b)
140640774013488
```

- 当 `a` 和 `b` 在同一行赋值为 `257` 是，Python 解释器创建一个对象，然后两个变量同时引用到它。如果不在同一行，而是分别赋值，解释器_不知道_已经存在一个 `257` 对象了。
- 这是针对解释器的编译优化，适用于交互式环境。当在实时解释器中输入两行时，这两行分别优化。如果在 `.py` 文件中测试这个例子，将会看到不同的行为，因为文件是一次编译的。

---

### A tic-tac-toe where X wins in the first attempt!

```py
# Let's initialize a row
row = [""]*3 #row i['', '', '']
# Let's make a board
board = [row]*3
```

**Output:**
```py
>>> board
[['', '', ''], ['', '', ''], ['', '', '']]
>>> board[0]
['', '', '']
>>> board[0][0]
''
>>> board[0][0] = "X"
>>> board
[['X', '', ''], ['X', '', ''], ['X', '', '']]
```

我们没有赋值三个 `'X'` 吧？

#### 解释

当初始化 `row` 变量时，下图解释了内存中的发生的事情

![after_row][11]

当通过 `*` 初始化 `board` 时，如下图所示(每一个元素 `board[0]`, `board[1]` 和 `board[2]` 都引用相同的列表，同时，`row` 也引用它)

![after_board][12]

生成 `board` 时，可以通过不使用 `row` 来避免这种情况。（根据这个[问题](https://github.com/satwikkansal/wtfpython/issues/68)）。

```py
>>> board = [['']*3 for _ in range(3)]
>>> board[0][0] = "X"
>>> board
[['X', '', ''], ['', '', ''], ['', '', '']]
```


如果想创建二维数组，应该用如下方法
```
board = [["" for _ in xrange(3)] for __ in xrange(3)]
```

---

### The sticky output function

```py
funcs = []
results = []
for x in range(7):
    def some_func():
        return x
    funcs.append(some_func)
    results.append(some_func())

funcs_results = [func() for func in funcs]
```

**Output:**
```py
>>> results
[0, 1, 2, 3, 4, 5, 6]
>>> funcs_results
[6, 6, 6, 6, 6, 6, 6]
```

即使当将 `some_func` 追加到 `funcs` 列表之前 `x` 的值在每次迭代时都不同，所有的函数还是都返回 `6`。

// 或者

```py
>>> powers_of_x = [lambda x: x**i for i in range(10)]
>>> [f(2) for f in powers_of_x]
[512, 512, 512, 512, 512, 512, 512, 512, 512, 512]
```

#### 解释

- 当在循环体内定义一个函数，并且函数体使用到循环变量时，循环函数的闭包被绑定到变量，而不是它的值。所以所有的函数都使用赋值给变量的最新值进行计算。
- 要得到预期结果，可以将循环变量作为命名变量传递给函数。 __为什么这有效？__因为这将在函数的作用域内再次定义变量。
  ```py
  funcs = []
  for x in range(7):
      def some_func(x=x):
          return x
      funcs.append(some_func)
  ```

  **Output:**
  ```py
  >>> funcs_results = [func() for func in funcs]
  >>> funcs_results
  [0, 1, 2, 3, 4, 5, 6]
  ```

---

### `is not ...` is not `is (not ...)`

```py
>>> 'something' is not None
True
>>> 'something' is (not None)
False
```

#### 解释

- `is not` 是一个二元操作符，作为一个整体出现，它的行为与分别使用 `is` 和 `not` 不同。
- `is not` 的结果是 `True`，如果两边的变量指向相同的对象，否则为 `False`。

---

### The surprising comma

**Output:**
```py
>>> def f(x, y,):
...     print(x, y)
...
>>> def g(x=4, y=5,):
...     print(x, y)
...
>>> def h(x, **kwargs,):
  File "<stdin>", line 1
    def h(x, **kwargs,):
                     ^
SyntaxError: invalid syntax
>>> def h(*args,):
  File "<stdin>", line 1
    def h(*args,):
                ^
SyntaxError: invalid syntax
```

#### 解释

- 在 Python 函数的形式参数列表中，最后的逗号并不总是合法的。
- 在 Python 中，参数列表部分用逗号和部分尾随逗号定义。这种冲突会导致逗号被困在中间，没有规则可以接受。
- __注意：__尾部的逗号问题是在 Python 3.6 中[修复的][13]。 在[这篇文章][14]中的评论简要讨论 Python 中尾随逗号的不同用法。

-  In Python, the argument list is defined partially with leading commas and partially with trailing commas. This conflict causes situations where a comma is trapped in the middle, and no rule accepts it.

---

### Backslashes at the end of string

**Output:**
```
>>> print("\\ C:\\")
\ C:\
>>> print(r"\ C:")
\ C:
>>> print(r"\ C:\")

    File "<stdin>", line 1
      print(r"\ C:\")
                     ^
SyntaxError: EOL while scanning string literal
```

#### 解释

- 在由前缀 `r` 指示的__raw string__文字常量中，反斜杠没有什么特殊的含义。
  ```py
  >>> print(repr(r"wt\"f"))
  'wt\\"f'
  ```
- 解释器实际上指示改变了反斜杠的行为，所以直接解析反斜杠和后面的字符。这就是为什么在末尾有反斜杠会报错的原因。


- In a raw string literal, as indicated by the prefix `r`, the backslash doesn't have the special meaning.
- What the interpreter actually does, though, is simply change the behavior of backslashes, so they pass themselves and the following character through. That's why backslashes don't work at the end of a raw string.

---

### not knot!

```py
x = True
y = False
```

**Output:**
```py
>>> not x == y
True
>>> x == not y
  File "<input>", line 1
    x == not y
           ^
SyntaxError: invalid syntax
```

#### 解释

- 操作符优先级影响表达式求值，在 Python 中 `==` 操作符比 `not` 的优先级高。
- 所以 `not x == y` 等价于 `not (x == y)`，又等价于 `not (True == False)`，最终结果为 `True`。
- 但是 `x == not y` 抛出 `SyntaxError` 异常。因为解释器认为它等价于 `(x == not) y` 而不是 `x == (not y)`，后者可能是一眼看上去的结果。
- 解释器解析时可能希望 `not` token 是 `not in` 操作符的一部分(`==` 和 `not in` 优先级相同)，但是在 `not` 之后没有找到 `in`，所以抛出 `SyntaxError` 异常。

---

### Half triple-quoted strings

**Output:**
```py
>>> print('wtfpython''')
wtfpython
>>> print("wtfpython""")
wtfpython
>>> # The following statements raise `SyntaxError`
>>> # print('''wtfpython')
>>> # print("""wtfpython")
```

#### 解释

- Python 支持隐式的[字符串常量连接][15]，例如:
  ```
  >>> print("wtf" "python")
  wtfpython
  >>> print("wtf" "") # or "wtf"""
  wtf
  ```
- 在 Python 中，`'''` 和 `"""` 也是字符串分隔符，之所以会抛出 `SyntaxError` 异常是因为 Python 解释器在解析字符串时，因为开头是 `'''` 或 `"""`，希望字符串同样以 `'''` 或 `"""` 结尾，

---

### Midnight time doesn't exist?

```py
from datetime import datetime

midnight = datetime(2018, 1, 1, 0, 0)
midnight_time = midnight.time()

noon = datetime(2018, 1, 1, 12, 0)
noon_time = noon.time()

if midnight_time:
    print("Time at midnight is", midnight_time)

if noon_time:
    print("Time at noon is", noon_time)
```

**Output:**
```sh
('Time at noon is', datetime.time(12, 0))
```
The midnight time is not printed.

#### 解释

在 Python 3.5 之前，`datetime.time` 对象如果是 UTC 午夜时间，返回的布尔值为 `False`。如果用 `if obj:` 这样的语法判断 `obj` 是 `null`，非常容易产生 bug。

> 没找到 3.5 之前的环境。

```
sh-4.4$ python3
Python 3.6.2 (default, Aug 11 2017, 11:59:59)
[GCC 7.1.1 20170622 (Red Hat 7.1.1-3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from datetime import datetime
>>> mid = datetime(2018, 1, 1, 0, 0)
>>> mid
datetime.datetime(2018, 1, 1, 0, 0)
>>> if mid:
...     print(mid)
...
2018-01-01 00:00:00

```

2.7
```
>>> from datetime import datetime
>>> mid = datetime(2008, 1, 1, 0, 0)
>>> mid
datetime.datetime(2008, 1, 1, 0, 0)
>>> if mid:
...     print mid
...
2008-01-01 00:00:00
```

---

### What's wrong with booleans?

1\.
```py
# A simple example to count the number of boolean and
# integers in an iterable of mixed data types.
mixed_list = [False, 1.0, "some_string", 3, True, [], False]
integers_found_so_far = 0
booleans_found_so_far = 0

for item in mixed_list:
    if isinstance(item, int):
        integers_found_so_far += 1
    elif isinstance(item, bool):
        booleans_found_so_far += 1
```

**Output:**
```py
>>> booleans_found_so_far
0
>>> integers_found_so_far
4
```

2.
```py
another_dict = {}
another_dict[True] = "JavaScript"
another_dict[1] = "Ruby"
another_dict[1.0] = "Python"
```

**Output:**
```py
>>> another_dict[True]
"Python"
```

3.
```py
>>> some_bool = True
>>> "wtf"*some_bool
'wtf'
>>> some_bool = False
>>> "wtf"*some_bool
''
```

#### 解释

- `bool` 是 `int` 的子类
  ```py
  >>> isinstance(True, int)
  True
  >>> isinstance(False, int)
  True
  ```

- `True` 的整型值为 `1`，`False` 为 `0`。
  ```py
  >>> True == 1 == 1.0 and False == 0 == 0.0
  True
  ```
- 其背后的原因参考 StackOverflow 上的[这个答案][16]

---

### Class attributes and instance attributes

1.
```py
class A:
    x = 1

class B(A):
    pass

class C(A):
    pass
```

**Ouptut:**
```py
>>> A.x, B.x, C.x
(1, 1, 1)
>>> B.x = 2
>>> A.x, B.x, C.x
(1, 2, 1)
>>> A.x = 3
>>> A.x, B.x, C.x
(3, 2, 3)
>>> a = A()
>>> a.x, A.x
(3, 3)
>>> a.x += 1
>>> a.x, A.x
(4, 3)
```

2.
```py
class SomeClass:
    some_var = 15
    some_list = [5]
    another_list = [5]
    def __init__(self, x):
        self.some_var = x + 1
        self.some_list = self.some_list + [x]
        self.another_list += [x]
```

**Output:**

```py
>>> some_obj = SomeClass(420)
>>> some_obj.some_list
[5, 420]
>>> some_obj.another_list
[5, 420]
>>> another_obj = SomeClass(111)
>>> another_obj.some_list
[5, 111]
>>> another_obj.another_list
[5, 420, 111]
>>> another_obj.another_list is SomeClass.another_list
True
>>> another_obj.another_list is some_obj.another_list
True
```

#### 解释

- 类变量和类实例变量在内部处理为类对象中的字典。如果在当前类的字典中找不到变量名，则会搜索其父类。
- `+=` 运算符在原地修改可变对象而不创建新对象。因此，更改一个实例的属性也会影响其他实例和类属性。

---

### yielding None

```py
some_iterable = ('a', 'b')

def some_func(val):
    return "something"
```

**Output:**
```py
>>> [x for x in some_iterable]
['a', 'b']
>>> [(yield x) for x in some_iterable]
<generator object <listcomp> at 0x7f70b0a4ad58>
>>> list([(yield x) for x in some_iterable])
['a', 'b']
>>> list((yield x) for x in some_iterable)
['a', None, 'b', None]
>>> list(some_func((yield x)) for x in some_iterable)
['a', 'something', 'b', 'something']
```

#### 解释

- 源码和解释可以参考[这里][17]
- 相关[bug 报告][18]

---

### Mutating the immutable!

```py
some_tuple = ("A", "tuple", "with", "values")
another_tuple = ([1, 2], [3, 4], [5, 6])
```

**Output:**
```py
>>> some_tuple[2] = "change this"
TypeError: 'tuple' object does not support item assignment
>>> another_tuple[2].append(1000) #This throws no error
>>> another_tuple
([1, 2], [3, 4], [5, 6, 1000])
>>> another_tuple[2] += [99, 999]
TypeError: 'tuple' object does not support item assignment
>>> another_tuple
([1, 2], [3, 4], [5, 6, 1000, 99, 999])
```

tuples 不是不可变的吗？

#### 解释

* 引用自[官方文档]19]
  > Immutable sequences  
  >      An object of an immutable sequence type cannot change once it is created. (If the object contains references to other objects, these other objects may be mutable and may be modified; however, the collection of objects directly referenced by an immutable object cannot change.)  
  > 不变序列  
  >     一个不变对象序列类型一旦创建时就不能改变了。(如果这个对象包含引用到其它对象的引用，这些饮用对象可能会改变；但是，不变对象本身不会改变。)

- `+=` 操作符在原地改变 list。对其赋值失败，但是当抛出异常时，其本身已经改变了。

---

### The disappearing variable from outer scope

```py
e = 7
try:
    raise Exception()
except Exception as e:
    pass
```

**Output (Python 2.x):**
```py
>>> print e
# prints nothing
```

**Output (Python 3.x):**
```py
>>> print(e)
NameError: name 'e' is not defined
```

#### 解释

* [出处][20]

  当一个异常作为 `as` 目标被赋值后，会在 `except` 子句结束时被销毁。就像如下:
  ```py
  except E as N:
      foo
  ```

  被转换为

  ```py
  except E as N:
      try:
          foo
      finally:
          del N
  ```

  This means the exception must be assigned to a different name to be able to refer to it after the except clause. Exceptions are cleared because, with the traceback attached to them, they form a reference cycle with the stack frame, keeping all locals in that frame alive until the next garbage collection occurs.

  这意味着必须将异常赋值给不同的变量才能在 `except` 子句之后引用它。异常被清除，因为 `traceback` 附加到异常上，它们与堆栈帧形成一个引用链，保持该帧中的所有局部变量有效，直到下一次垃圾收集发生。

- 在 Python 中，子句不是作用域。示例中的所有内容都存在于同一个作用域中，并且由于执行 `except` 子句，变量 `e` 被删除。具有单独的内部作用域的函数不是这种情况。下面的例子说明了这一点：
  ```py
  def f(x):
     del(x)
     print(x)

  x = 5
  y = [5, 4, 3]
  ```

  **Output:**
  ```py
  >>>f(x)
  UnboundLocalError: local variable 'x' referenced before assignment
  >>>f(y)
  UnboundLocalError: local variable 'x' referenced before assignment
  >>> x
  5
  >>> y
  [5, 4, 3]
  ```

- 在 Python 2.x 中，变量名 `e` 被赋值为 `Exception()` 实例，所以试图打印它时，没有任何输出。
  **Output (Python 2.x):**
  ```py
  >>> e
  Exception()
  >>> print e
  # Nothing is printed!
  ```

---

### When True is actually False

```py
True = False
if True == False:
    print("I've lost faith in truth!")
```

**Output:**
```
I've lost faith in truth!
```
#### 解释

- 最初，Python 没有 `bool` 类型(人们用 `0` 表示 `false`，非 `0` 表示 `true`)。后来才增加了 `True`, `False`，还有 `bool` 类型。单位了向后兼容，不能创建 `True` 和 `False` 常量 - 它们只是内置变量。
- Python 3 是后向不兼容的，所以现在终于可以解决这个问题了，所以这个例子不适用于Python 3.x！

---

### From filled to None in one instruction...

```py
some_list = [1, 2, 3]
some_dict = {
  "key_1": 1,
  "key_2": 2,
  "key_3": 3
}

some_list = some_list.append(4)
some_dict = some_dict.update({"key_4": 4})
```

**Output:**
```py
>>> print(some_list)
None
>>> print(some_dict)
None
```

#### 解释

大多数像 `list.append`, `dict.update`, `list.sort` 等修改序列/哈希对象的方法都是在原地修改对象并返回 `None`。这背后的原因是通过避免复制对象来提高性能，如果操作可以在原地完成(参见[why-doesn-t-list-sort-return-the-sorted-list][21])

---

### Subclass relationships *

**Output:**
```py
>>> from collections import Hashable
>>> issubclass(list, object)
True
>>> issubclass(object, Hashable)
True
>>> issubclass(list, Hashable)
False
```

继承关系应该是可传递的，对吗？(i.e., 如果 `A` 是 `B` 的子类，`B` 是 `C` 的子类，那么 `A` _应该_ 是 `C` 的子类)

#### 解释

- 在 Python 中，继承关系不一定是传递的。任何人都可以在元类(metaclass) 中定义自己的任意的 `__subclasscheck__`。
- 当调用 `issubclass（cls，Hashable）` 时，它只是在 `cls` 或者它所继承的任何东西中寻找 `non-Falsey` 的 `__hash__` 方法。
- 由于 `object` 是可哈希的，而 `list` 是不可哈希的，所以它打破了传递关系。
- 更详细的解释可以在[这里]22]找到


---

### The mysterious key type conversion *

```py
class SomeClass(str):
    pass

some_dict = {'s':42}
```

**Output:**
```py
>>> type(list(some_dict.keys())[0])
str
>>> s = SomeClass('s')
>>> some_dict[s] = 40
>>> some_dict # expected: Two different keys-value pairs
{'s': 40}
>>> type(list(some_dict.keys())[0])
str
```

#### 解释

- 对象 `s` 和字符串 `'s'` 的哈希值是一样的，因为 `SomeClass` 继承 `str` 类的 `__hash__` 方法。
- `SomeClass("s") == "s"` 的值为 `True`，因为 `SomeClass` 同样继承 `str` 类的 `__eq__` 方法。
- 因为两者的哈希值相同，它们在字典中的 `key` 也相同。
- 为了产生期待的效果，可以重新定义 `__eq__` 方法。
  ```py
  class SomeClass(str):
    def __eq__(self, other):
        return (
            type(self) is SomeClass
            and type(other) is SomeClass
            and super().__eq__(other)
        )

    # When we define a custom __eq__, Python stops automatically inheriting the
    # __hash__ method, so we need to define it as well
    __hash__ = str.__hash__

  some_dict = {'s':42}
  ```

  **Output:**
  ```py
  >>> s = SomeClass('s')
  >>> some_dict[s] = 40
  >>> some_dict
  {'s': 40, 's': 42}
  >>> keys = list(some_dict.keys())
  >>> type(keys[0]), type(keys[1])
  (__main__.SomeClass, str)
  ```

---

### Let's see if you can guess this?

```py
a, b = a[b] = {}, 5
```

**Output:**
```py
>>> a
{5: ({...}, 5)}
```

#### 解释

- 根据 [Python language reference][23]，赋值语句形式如下:
  ```
  (target_list "=")+ (expression_list | yield_expression)
  ```
  还有
  > 赋值语句计算表达式列表(记住，表达式可以使单独的表达式或者逗号分隔的列表，后者产生一个 tuple)，然后将值赋给目标列表，从左到右计算。
- 在 `(target_list "=")+` 中 `+` 意思是可以有 __一个或多个__目标列表。在这个例子中，目标列表是 `a, b` 和 `a[b]`(注意表达式列表有且只有一个，在我们的例子中是 `{}, 5`)。
- 在表达式列表求值后，它的值__从左到右__解包(unpacked)到目标列表。所以，在示例中，首先 tuple `{}, 5` 解包到 `a, b`，此时 `a = {}` 且 `b = 5`。
- 此时 `a` 被赋值为 `{}`，是一个可变对象。
- 第二个目标列表是 `a[b]` (你可能想这会抛出一个异常，因为 `a` 和 `b` 在这个赋值语句之前还没有定义，但请记住，我们刚刚给 `a` 和 `b` 赋值，`a = {}` 且 `b = 5`)。
- 现在，我们将字典中 key `5` 设为 tuple `({}, 5)`，产生一个循环引用(输出中的 `{...}` 引用到 `a` 已经引用的相同对象)。另外一个循环引用的小例子是:
  ```py
  >>> some_list = some_list[0] = [0]
  >>> some_list
  [[...]]
  >>> some_list[0]
  [[...]]
  >>> some_list is some_list[0]
  True
  >>> some_list[0][0][0][0][0][0] == some_list
  True
  ```
  我们示例与此相似(`a[b][0]` 和 `a` 是相同的对象)

* 所以，总结一下，可以将这个例子分解如下:
  ```py
  a, b = {}, 5
  a[b] = a, b
  ```
  然后循环引用可以通过 `a[b][0]` 和 `a` 是同一个对象来确定。
  ```py
  >>> a[b][0] is a
  True
  ```

---

---

## Section: Appearances are deceptive!

### Skipping lines?

**Output:**
```py
>>> value = 11
>>> valuе = 32
>>> value
11
```

靠!?

__注意:__ 产生这个结果的简单方是拷贝这几条语句然后粘贴到你文件或shell中

#### 解释

一些非西方字符看起来和英文字母一样，但是解释器认为它们不一样。

```py
>>> ord('е') # cyrillic 'e' (Ye) 斯拉夫语
1077
>>> ord('e') # latin 'e', as used in English and typed using standard keyboard
101
>>> 'е' == 'e'
False

>>> value = 42 # latin e
>>> valuе = 23 # cyrillic 'e', Python 2.x interpreter would raise a `SyntaxError` here
>>> value
42
```

内置的 `ord()` 函数返回字符的 Unicode [code point][24]，不同的_code point_导致如上行为。

---

### Teleportation *

```py
import numpy as np

def energy_send(x):
    # Initializing a numpy array
    np.array([float(x)])

def energy_receive():
    # Return an empty numpy array
    return np.empty((), dtype=np.float).tolist()
```

**Output:**
```py
>>> energy_send(123.456)
>>> energy_receive()
123.456
```

Where's the Nobel Prize?

#### 解释

- 注意到在 `energy_send` 函数中创建的 `np.array()` 没有被返回，所以其内存空间是可以被重新使用的。
- `numpy.empty()` 函数返回下一块可以使用的内存块，且不进行初始化。这个内存块恰好和刚刚释放的内存相同(通常但不总是)。

---

### Well, something is fishy...

```py
def square(x):
    """
    A simple function to calculate the square of a number by addition.
    """
    sum_so_far = 0
    for counter in range(x):
        sum_so_far = sum_so_far + x
  return sum_so_far
```

**Output (Python 2.x):**

```py
>>> square(10)
10
```

不应该是 100 吗？

__注意:__ 如果不能产生示例中的结果，在 shell 中直接运行这个文件:[mixed_tabs_and_spaces.py][25]

```
def square(x):
    sum_so_far = 0
    for _ in range(x):
        sum_so_far += x
    return sum_so_far  # tab leading noqa: E999 # pylint: disable=mixed-indentation Python 3 will raise a TabError here

print(square(10))
```

#### 解释

- __不要混用 tab 和 space!__ 在 `return` 语句之前的字符是 `tab`，然后代码其它地方是通过 4 个空格对齐的。
- Python 是这样处理 tab 的:
  > 首先 tab 被(从左到右)替换为 1 到 8 个空格使得字符数正好是 8 的倍数。
- 所以 `square` 函数最后一行的 tab 被替换为 8 个空格，然后使这条语句到了循环体内部。
- Python 3 足够友好，可以自动的为这种情况抛出一个异常。
  **Output (Python 3.x):**
  ```py
  TabError: inconsistent use of tabs and spaces in indentation
  ```

---

---

## Section: Watch out for the landmines!

小心地雷


### Modifying a dictionary while iterating over it

```py
x = {0: None}

for i in x:
    del x[i]
    x[i+1] = None
    print(i)
```

**Output (Python 2.7- Python 3.5):**

```
0
1
2
3
4
5
6
7
```

是的，它运行正好 __8__ 次才结束。

#### 解释

- Python 不支持在迭代字典的同时进行修改。
- 运行八次是因为字典需要调整大小以保存更多的键，而 8 正好首次调整时的值。这实际上是实现上的细节。
- 如何处理已删除的键以及何时发生调整大小对于不同的 Python 实现可能会有所不同。
- 有关更多信息，可以参考 StackOverflow 上的讨论 [bug-in-python-dict][26]。

---

### Stubborn `del` operator *

```py
class SomeClass:
    def __del__(self):
        print("Deleted!")
```

**Output:**
1.
```py
>>> x = SomeClass()
>>> y = x
>>> del x # this should print "Deleted!"
>>> del y
Deleted!
```

哦，最后才删除。你可能已经猜到在视图删除 `x`，第一次调用 `__del__` 时保存了什么。让我们添加一些代码。

2.
```py
>>> x = SomeClass()
>>> y = x
>>> del x
>>> y # check if y exists
<__main__.SomeClass instance at 0x7f98a1a67fc8>
>>> del y # Like previously, this should print "Deleted!"
>>> globals() # oh, it didn't. Let's check all our global variables and confirm
Deleted!
{'__builtins__': <module '__builtin__' (built-in)>, 'SomeClass': <class __main__.SomeClass at 0x7f98a1a5f668>, '__package__': None, '__name__': '__main__', '__doc__': None}
```

好了，现在才删除了，困惑。

#### 解释

- `del x` 并不直接调用 `x.__del__()`。
- 无论何时遇到 `del x`，Python 为 `x` 减少引用次数，当 引用次数为 0 时调用 `x.__del__()`。
- 在第二个例子中，`y.__del__()` 没有被调用是因为先前的语句(`>>> y`)在交互式解释器中创建了另一个引用吗，因此当调用 `del y` 时，引用次数并没有减到 0。
- 当调用 `globals` 时，是的已有的引用被销毁，因此可以看到输出 "Deleted!"。(最终还是发生!)

---

### Deleting a list item while iterating

```py
list_1 = [1, 2, 3, 4]
list_2 = [1, 2, 3, 4]
list_3 = [1, 2, 3, 4]
list_4 = [1, 2, 3, 4]

for idx, item in enumerate(list_1):
    del item

for idx, item in enumerate(list_2):
    list_2.remove(item)

for idx, item in enumerate(list_3[:]):
    list_3.remove(item)

for idx, item in enumerate(list_4):
    list_4.pop(idx)
```

**Output:**
```py
>>> list_1
[1, 2, 3, 4]
>>> list_2
[2, 4]
>>> list_3
[]
>>> list_4
[2, 4]
```

你能猜到为什么输出 `[2, 4]` 吗？

#### 解释

- 在对对象进行迭代时对其进行改变永远是糟糕的想法。正确的做法是对对象的拷贝进行迭代，就像 `list_3[:]` 中的那样。
  ```py
  >>> some_list = [1, 2, 3, 4]
  >>> id(some_list)
  139798789457608
  >>> id(some_list[:]) # Notice that python creates new object for sliced list.
  139798779601192
  ```

__`del`, `remove`, `pop` 的区别__

- `del var_name` 只从局部或全局命名空间删除 `var_name` 的绑定(这是为什么 `list_1` 没受到影响的原因)。
- `remove` 删除第一个匹配到的值，而不是特定的坐标，如果值没有找到则抛出 `ValueError` 异常。
- `pop` 通过坐标删除元素，然后将其返回，如果坐标非法则抛出 `IndexError` 异常。

__为什么输出 `[2, 4]`?__
- 列表是通过坐标迭代的，当我们从 `list_2` 或 `list_4` 删除 `1` 后，列表的内容变为 `[2, 3, 4]`。剩余的元素相当于左移了，i.e., `2` 在位置 0, `3` 在位置 1。因为下一次迭代开始取位置 1 (值为 `3`)，跳过了 `2`。同样，列表元素序列是交替删除的。

- 参考 StackOverflow 上的讨论 [what-happens-when-you-try-to-delete-a-list-element-while-iterating-over-it][27]
- 参考 StackOverflow 上对字典相关的讨论 [how-to-change-all-the-dictionary-keys-in-a-for-loop-with-d-items][28]

---

### Loop variables leaking out!

1.
```py
for x in range(7):
    if x == 6:
        print(x, ': for x inside loop')
print(x, ': x in global')
```

**Output:**
```py
6 : for x inside loop
6 : x in global
```

但是 `x` 从来没有在循环作用域外定义...

2.
```py
# This time let's initialize x first
x = -1
for x in range(7):
    if x == 6:
        print(x, ': for x inside loop')
print(x, ': x in global')
```

**Output:**
```py
6 : for x inside loop
6 : x in global
```

3.
```
x = 1
print([x for x in range(5)])
print(x, ': x in global')
```

**Output (on Python 2.x):**
```
[0, 1, 2, 3, 4]
(4, ': x in global')
```

**Output (on Python 3.x):**
```
[0, 1, 2, 3, 4]
1 : x in global
```

#### 解释

- 在 Python 中，for 循环使用它们所在的作用域，并将其定义的循环变量保留。如果我们之前在全局命名空间中显式定义了 for 循环变量，这也适用。在这种情况下，它将重新绑定现有的变量。

- 对于列表解析的例子，Python 2.x 和 Python 3.x 解释器输出的差异可以通过[Python 3.0中的新增功能][29]来解释。
  > 列表解析不再支持语法形式 `[... for var in item1，item2，...]`，而是使用 `[... for var in (item1，item2，...)]`。另外请注意列表解析具有不同的语义: 它们更接近于在 `list()` 构造函数中的生成器表达式的语法糖，特别是循环控制变量不再泄漏到外围作用域中。


---

### Beware of default mutable arguments!

```py
def some_func(default_arg=[]):
    default_arg.append("some_string")
    return default_arg
```

**Output:**
```py
>>> some_func()
['some_string']
>>> some_func()
['some_string', 'some_string']
>>> some_func([])
['some_string']
>>> some_func()
['some_string', 'some_string', 'some_string']
```

#### 解释

- 在 Python 中，函数的可变默认参数并不是在每次调用函数时初始化。而是在用最近一次赋值作为默认值。当我们显示传参数 `[]` 给  `some_func`，`default_arg` 的默认值并没有使用，所以函数返回期望的值。
  ```py
  def some_func(default_arg=[]):
      default_arg.append("some_string")
      return default_arg
  ```

  **Output:**
  ```py
  >>> some_func.__defaults__ #This will show the default argument values for the function
  ([],)
  >>> some_func()
  >>> some_func.__defaults__
  (['some_string'],)
  >>> some_func()
  >>> some_func.__defaults__
  (['some_string', 'some_string'],)
  >>> some_func([])
  >>> some_func.__defaults__
  (['some_string', 'some_string'],)
    ```
- 在实践中，避免因为默认参数引起 bug 的做法是给默认参数赋值为 `None`，然后检查与其相关的参数是否传入。例如:
  ```py
  def some_func(default_arg=None):
      if not default_arg:
          default_arg = []
      default_arg.append("some_string")
      return default_arg
  ```

---

### Catching the Exceptions

```py
some_list = [1, 2, 3]
try:
    # This should raise an ``IndexError``
    print(some_list[4])
except IndexError, ValueError:
    print("Caught!")

try:
    # This should raise a ``ValueError``
    some_list.remove(4)
except IndexError, ValueError:
    print("Caught again!")
```

**Output (Python 2.x):**
```py
Caught!

ValueError: list.remove(x): x not in list
```

**Output (Python 3.x):**
```py
  File "<input>", line 3
    except IndexError, ValueError:
                     ^
SyntaxError: invalid syntax
```

#### 解释

- 为了在 `except` 子句中捕获多个异常，应该将他们用 `()` 括起来作为 tuple 当做第一个参数来传递。第二个参数是可选的名字，如果我们提供了，则会绑定到抛出的异常示例上。另一个例子:
  ```py
  some_list = [1, 2, 3]
  try:
     # This should raise a ``ValueError``
     some_list.remove(4)
  except (IndexError, ValueError), e:
     print("Caught again!")
     print(e)
  ```
  **Output (Python 2.x):**
  ```
  Caught again!
  list.remove(x): x not in list
  ```
  **Output (Python 3.x):**
  ```py
    File "<input>", line 4
      except (IndexError, ValueError), e:
                                       ^
  IndentationError: unindent does not match any outer indentation level
  ```

- 通过 ',' 将异常和变量分开的做法在 Python 3 中被废弃了，所以执行失败；正确的做法是用 `as`。例如:
  ```py
  some_list = [1, 2, 3]
  try:
      some_list.remove(4)

  except (IndexError, ValueError) as e:
      print("Caught again!")
      print(e)
  ```
  **Output:**
  ```
  Caught again!
  list.remove(x): x not in list
  ```

---

### Same operands, different story!

1.
```py
a = [1, 2, 3, 4]
b = a
a = a + [5, 6, 7, 8]
```

**Output:**
```py
>>> a
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
[1, 2, 3, 4]
```

2.
```py
a = [1, 2, 3, 4]
b = a
a += [5, 6, 7, 8]
```

**Output:**
```py
>>> a
[1, 2, 3, 4, 5, 6, 7, 8]
>>> b
[1, 2, 3, 4, 5, 6, 7, 8]
```

#### 解释

- `a += b` 并不总是和 `a = a + b` 一样。类__可能__实现不同的__`op=`__操作符，列表就是这样做的。
- 表达式 `a = a + [5,6,7,8]` 生成一个新的列表，然后然后用 `a` 引用这个列表，而 `b` 没有变(还是引用原来的对象)。
- 表达式 `a += [5,6,7,8]` 实际上映射到 `extend` 函数， `a` 和 `b` 还是引用相同的列表，而列表在原地被改变。

---

### The out of scope variable

```py
a = 1
def some_func():
    return a

def another_func():
    a += 1
    return a
```

**Output:**
```py
>>> some_func()
1
>>> another_func()
UnboundLocalError: local variable 'a' referenced before assignment
```

#### 解释

- 当在作用域内为一个变量赋值时，它成为这个作用域的局部变量。所以 `a` 成为 `another_func` 的局部变量，但是因为在同一作用域内，之前没有对其进行初始化，所以抛出异常。
- 阅读[2014_python_scope_and_namespaces][30]，这是一篇简短但清晰的指南，介绍了在 Python 中名字空间和作用域解析是如何工作的。
- 为了在 `another_func` 中修改外围作用域中的 `a`，使用 `global` 关键字。
  ```py
  def another_func()
      global a
      a += 1
      return a
  ```

  **Output:**
  ```py
  >>> another_func()
  2
  ```

---

### Be careful with chained operations

```py
>>> (False == False) in [False] # makes sense
False
>>> False == (False in [False]) # makes sense
False
>>> False == False in [False] # now what?
True

>>> True is False == False
False
>>> False is False is False
True

>>> 1 > 0 < 1
True
>>> (1 > 0) < 1
False
>>> 1 > (0 < 1)
False
```

#### 解释

按 [python_reference_not_in][31] 中的描述

> 形式上，如果 a, b, c, ..., y, z 是表达式，而 op1, op2, ..., opN 是比较操作符，则 `a op1 b op2 c ... y opN z` 等价于 `a op1 b and b op2 c and ... y opN z`，除了每个表达式都只计算一次。

虽然在上面的例子中，这样的行为可能看起来很愚蠢，但像 `a == b == c` 和 `0 <= x <= 100` 这样的表达式真实太优雅了。

- `False is False is False` 等价于 `(False is False) and (False is False)`
- `True is False == False` 等价于 `True is False and False == False`，因为语句的第一部分(`True is False`)的结果为 `False`，所以整个表达式的结果是 `False`.
- `1 > 0 < 1` 等价于 `1 > 0 and 0 < 1`，结果为 `True`.
- `(1 > 0) < 1` 等价于 `True < 1` 而且

  ```py
  >>> int(True)
  1
  >>> True + 1 #not relevant for this example, but just for fun
  2
  ```
  所以 `1 < 1` 结果为 `False`。

---

### Name resolution ignoring class scope

1.

```py
x = 5
class SomeClass:
    x = 17
    y = (x for i in range(10))
```

**Output:**
```py
>>> list(SomeClass.y)[0]
5
```

2.
```py
x = 5
class SomeClass:
    x = 17
    y = [x for i in range(10)]
```

**Output (Python 2.x):**
```py
>>> SomeClass.y[0]
17
```

**Output (Python 3.x):**
```py
>>> SomeClass.y[0]
5
```

#### 解释

- 嵌套在类定义中的作用域忽略在类级绑定的名称。
- 生成器表达式有其自己的作用域。
- 从Python 3.X开始，列表解析也有自己的作用域。

---

### Needle in a Haystack

1.
```py
x, y = (0, 1) if True else None, None
```

**Output:**
```
>>> x, y  # expected (0, 1)
((0, 1), None)
```

Almost every Python programmer has faced a similar situation.

2.
```py
t = ('one', 'two')
for i in t:
    print(i)

t = ('one')
for i in t:
    print(i)

t = ()
print(t)
```

**Output:**
```py
one
two
o
n
e
tuple()
```

#### 解释

* 对于 1, 产生预期行为的正确语句是 `x, y = (0, 1) if True else (None, None)`.
* 对于 2, 产生预期行为的正确语句是 `t = ('one',)` or `t = 'one',` (不要省略 ',') 否则解释器认为 `t` 是一个 `str` 然后按字符对其迭代.
* `()` 是一个特殊的 token，表示一个空的 `tuple`.

---

---


## Section: The Hidden treasures!

本章节包含一些 Python 中很少有人知道的有趣的事情。

### Okay Python, Can you make me fly? *

好，起飞

```py
import antigravity
```

**Output:**
Sshh.. It's a super secret.

#### 解释
+ `antigravity` 模块是 Python 的一个彩蛋。
+ `import antigravity` 开启浏览器并打开[classic XKCD comic](http://xkcd.com/353/)
* 而且，还有更多好玩儿的。这里有__彩蛋中的彩蛋__。如果你查看[源码](https://github.com/python/cpython/blob/master/Lib/antigravity.py#L7-L17)，这里有一个函数定义来实现[以上算法](https://xkcd.com/426/)。

---

### `goto`, but why? *

```py
from goto import goto, label
for i in range(9):
    for j in range(9):
        for k in range(9):
            print("I'm trapped, please rescue!")
            if k == 2:
                goto .breakout # breaking out from a deeply nested loop
label .breakout
print("Freedom!")
```

**Output (Python 2.3):**
```py
I'm trapped, please rescue!
I'm trapped, please rescue!
Freedom!
```

#### 解释

- 在 Python 中，对 `goto` 的支持于 2004 年作为愚人节笑话[发布](https://mail.python.org/pipermail/python-announce-list/2004-April/002982.html)。
- 当前版本的 Python 并不包含此模块。
- 尽管能这么做，但请不要这么做。这里有关于为什么 [Python 不支持 `goto` 的讨论](https://docs.python.org/3/faq/design.html#why-is-there-no-goto)。

---

### Brace yourself! *

如果你不想使用 Python 风格的空格缩进表示代码块，可以通过 import __future__ 模块来使用 C-风格的 '{}'。

```py
from __future__ import braces
```

**Output:**
```py
  File "some_file.py", line 1
    from __future__ import braces
SyntaxError: not a chance
```

大括号？没门儿！如果你感到失望，请使用 Java。

#### 解释

- `__future__` 模块一般用来提供 Python 将来的版本中的特性。然而，这里的“future”具有讽刺意味。
- 这又是一个彩蛋。

---

### Let's meet Friendly Language Uncle For Life *

**Output (Python 3.x)**
```py
>>> from __future__ import barry_as_FLUFL
>>> "Ruby" != "Python" # there's no doubt about it
  File "some_file.py", line 1
    "Ruby" != "Python"
              ^
SyntaxError: invalid syntax

>>> "Ruby" <> "Python"
True
```

There we go.

#### 解释

- 这与 2009-04-01 发布的 [PEP-401](https://www.python.org/dev/peps/pep-0401/) 相关(你应该知道这个日期的含义)。
- 引用自 PEP-401
  > Recognized that the != inequality operator in Python 3.0 was a horrible, finger pain inducing mistake, the FLUFL reinstates the <> diamond operator as the sole spelling.
- 关于 PEP，Uncle Barry 还有更多要分享的。可以参考 [PEP-401](https://www.python.org/dev/peps/pep-0401/).

---

### Even Python understands that love is complicated *

```py
import this
```

等等，__这__是什么？

**Output:**
```
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

Python 之禅

```py
>>> love = this
>>> this is love
True
>>> love is True
False
>>> love is False
False
>>> love is not True or False
True
>>> love is not True or False; love is love  # Love is complicated
True
```

#### 解释

- `this` 模块是 Python 之禅([PEP-20](https://www.python.org/dev/peps/pep-0020)) 的复活节彩蛋。
- 如果你觉得这已经足够有趣了，检出其实现 [this.py](https://hg.python.org/cpython/file/c3896275c0f6/Lib/this.py)。有趣的是，代码违背了 Python 之禅(其代码哲学)(可能这也是唯一一处)。
- 关于语句 `love is not True or False; love is love`，同样不言自明。

---

### Yes, it exists!

**The `else` clause for loops.** One typical example might be:
__循环的 `else` 子句__。一个典型用法可能是：

```py
  def does_exists_num(l, to_find):
      for num in l:
          if num == to_find:
              print("Exists!")
              break
      else:
          print("Does not exist")
```

**Output:**
```py
>>> some_list = [1, 2, 3, 4, 5]
>>> does_exists_num(some_list, 4)
Exists!
>>> does_exists_num(some_list, -1)
Does not exist
```

__异常处理中的 `else` 子句__。例子：

```py
try:
    pass
except:
    print("Exception occurred!!!")
else:
    print("Try block executed successfully...")
```

**Output:**
```py
Try block executed successfully...
```

#### 解释

- 当循环结束，且在所有的迭代完成没有显示的调用 `break`，会执行 `else` 子句。
- `try` 块后的 `else` 子句也称为 "completion clause"，因为没有异常时才会进入 `else` 子句。

---

### Inpinity *

"Inpinity" 并不是 “infinity" 的错误拼写，这里是故意的，不要提 bug。

**Output (Python 3.x):**
```py
>>> infinity = float('infinity')
>>> hash(infinity)
314159
>>> hash(float('-inf'))
-314159
```

#### 解释

- `infinity` 的哈希值是 `$ 10^5 \times \pi $`
- 有趣的是，`float('-inf')` 的哈希值在 Python 3 中是 `$ -10^5 \times \pi $`，而在 Python 2 中是 `$ -10^5 \times e $`

---

### Mangling time! *

```py
class Yo(object):
    def __init__(self):
        self.__honey = True
        self.bitch = True
```

**Output:**
```py
>>> Yo().bitch
True
>>> Yo().__honey
AttributeError: 'Yo' object has no attribute '__honey'
>>> Yo()._Yo__honey
True
```

为什么 `Yo()._Yo__honey` 可以正确输出？Only Indian readers would understand.

#### 解释

* [Name Mangling](https://en.wikipedia.org/wiki/Name_mangling) 用来避免不同命名空间的名字冲突。
* 在 Python 中，解释器修改(混淆)类成员中以 `__` 开头且不以一个以上的 `_` 结尾的的名字，在其前面加上 `_NameOfTheClass`。
* 所以，为了访问 `__honey` 属性，需要将 `_Yo` 加到前面，为了与其他类的名字冲突。

---

---

## Section: Miscallaneous


### `+=` is faster

```py
# using "+", three strings:
>>> timeit.timeit("s1 = s1 + s2 + s3", setup="s1 = ' ' * 100000; s2 = ' ' * 100000; s3 = ' ' * 100000", number=100)
0.25748300552368164
# using "+=", three strings:
>>> timeit.timeit("s1 += s2 + s3", setup="s1 = ' ' * 100000; s2 = ' ' * 100000; s3 = ' ' * 100000", number=100)
0.012188911437988281
```

#### 解释

- 连接两个以上字符串时，`+=` 比 `+` 要快，因为并不需要销毁第一个字符串。

---

### Let's make a giant string!

```py
def add_string_with_plus(iters):
    s = ""
    for i in range(iters):
        s += "xyz"
    assert len(s) == 3*iters

def add_bytes_with_plus(iters):
    s = b""
    for i in range(iters):
        s += b"xyz"
    assert len(s) == 3*iters

def add_string_with_format(iters):
    fs = "{}"*iters
    s = fs.format(*(["xyz"]*iters))
    assert len(s) == 3*iters

def add_string_with_join(iters):
    l = []
    for i in range(iters):
        l.append("xyz")
    s = "".join(l)
    assert len(s) == 3*iters

def convert_list_to_string(l, iters):
    s = "".join(l)
    assert len(s) == 3*iters
```

**Output:**
```py
>>> timeit(add_string_with_plus(10000))
1000 loops, best of 3: 972 µs per loop
>>> timeit(add_bytes_with_plus(10000))
1000 loops, best of 3: 815 µs per loop
>>> timeit(add_string_with_format(10000))
1000 loops, best of 3: 508 µs per loop
>>> timeit(add_string_with_join(10000))
1000 loops, best of 3: 878 µs per loop
>>> l = ["xyz"]*10000
>>> timeit(convert_list_to_string(l, 10000))
10000 loops, best of 3: 80 µs per loop
```

Let's increase the number of iterations by a factor of 10.

```py
>>> timeit(add_string_with_plus(100000)) # Linear increase in execution time
100 loops, best of 3: 9.75 ms per loop
>>> timeit(add_bytes_with_plus(100000)) # Quadratic increase
1000 loops, best of 3: 974 ms per loop
>>> timeit(add_string_with_format(100000)) # Linear increase
100 loops, best of 3: 5.25 ms per loop
>>> timeit(add_string_with_join(100000)) # Linear increase
100 loops, best of 3: 9.85 ms per loop
>>> l = ["xyz"]*100000
>>> timeit(convert_list_to_string(l, 100000)) # Linear increase
1000 loops, best of 3: 723 µs per loop
```

#### 解释
- 关于 timeit，参考 [timeit 文档](https://docs.python.org/3/library/timeit.html)。通常用来测试代码片段的执行时间。
- 不要使用 `+` 来生成长字符串 - 在 Python 中，`str` 是不可变的，所以每次用 `+` 进行连接时，操作符两边的字符串都要拷贝到一个新字符串。如果你要连接 4 个长度为 10 的字符串，需要拷贝 (10+10) + ((10+10)+10) + (((10+10)+10)+10) = 90 个字符，而不是 40 个字符。随着字符串数量和大小的增加，情况会变得更加糟糕(与 `add_bytes_with_plus` 函数执行时间一致)。
- 因此，建议使用 `.format` 或者 `%` 语法(然而，对短字符串来说，它们比 `+` 稍慢)。
- 或者更好的做法是，如果其内容已经是某种形式的可迭代对象，用 `''.join(iterable_object)` 会更快。
- `add_string_with_plus` 没有像 `add_bytes_with_plus` 那样运行时间成 O(n^2)，因为在前面的例子中讨论了 '+=` 优化。 如果语句是 `s = s + "x" + "y" + "z"` 而不是 `s += "xyz"`, 那么时间就是 O(n^2) 的。
  ```py
  def add_string_with_plus(iters):
      s = ""
      for i in range(iters):
          s = s + "x" + "y" + "z"
      assert len(s) == 3*iters

  >>> timeit(add_string_with_plus(10000))
  100 loops, best of 3: 9.87 ms per loop
  >>> timeit(add_string_with_plus(100000)) # Quadratic increase in execution time
  1 loops, best of 3: 1.09 s per loop
  ```

---

### Explicit typecast of strings

```py
a = float('inf')
b = float('nan')
c = float('-iNf')  #These strings are case-insensitive
d = float('nan')
```

**Output:**
```py
>>> a
inf
>>> b
nan
>>> c
-inf
>>> float('some_other_string')
ValueError: could not convert string to float: some_other_string
>>> a == -c #inf==inf
True
>>> None == None # None==None
True
>>> b == d #but nan!=nan
False
>>> 50/a
0.0
>>> a/a
nan
>>> 23 + b
nan
```

#### 解释

- 当显示转为 `float` 类型时，`'inf'` 和 `'nan'` 是特殊的字符串(大小写不敏感)，分别表示 "infinity" 和 "not a number"。

---

### Minor Ones

- `join()` 是字符串操作符，而不是列表操作符。(有点儿反直觉)

  __解释__

  如果 `join()` 是字符串方法，那么它可以操纵任何可迭代对象(list, tuple, iterators)。如果他是列表的方法，需要对每一个类型进行单独的实现。同时，将一个针对字符串特化的方法对通用的 `list` 对象进行特化是没有意义的。

  > 见 StackOverflow 上的讨论 [python-join-why-is-it-string-joinlist-instead-of-list-joinstring](https://stackoverflow.com/questions/493819/python-join-why-is-it-string-joinlist-instead-of-list-joinstring)

- 看起来奇怪但语义上正确的语句
  + `[] = ()` 语义上正确(解包一空的个 `tuple` 到一个空的 `list`)
  + `'a'[0][0][0][0][0]` 同样在语义上正确，因为在 Python 中字符串是[序列(sequence)](https://docs.python.org/3/glossary.html#term-sequence)，而可迭代对象支持通过整数坐标访问元素。
  + `3 --0-- 5 == 8` 和 `--5 == 5` 同样正确，而且结果是 `True`。

- 假设 `a` 是一个整数，`++a` 和 `--a` 都是合法的 Python 语句，但不要期待像其他语言(C, cpp, Java)那样的行为。
  ```py
  >>> a = 5
  >>> a
  5
  >>> ++a
  5
  >>> --a
  5
  ```

  __解释__
  + Python 语法中，没有 `++` 操作符，实际上是两个 `+` 操作符。
  + `++a` 解析成 `+(+a)`，翻译为 `a`。同样，`--a` 的输出也是合理的。
  + StackOverflow 上 [why-are-there-no-and-operators-in-python](https://stackoverflow.com/questions/3654830/why-are-there-no-and-operators-in-python) 讨论了 Python 中没有自增和自减操作符的原因。

- Python 使用 2 个字节作为函数中的局部变量存储。理论上，这意味着只能在函数中定义 65536 个变量。但是，python 内置了一个方便的解决方案，可以存储超过2 ^ 16个变量名称。下面的代码演示了当定义多于 65536 个局部变量时栈中会发生什么(警告：此代码打印大约2 ^ 18行文本，因此请准备好！)：
     ```py
     import dis
     exec("""
     def f():
         """ + """
         """.join(["X"+str(x)+"=" + str(x) for x in range(65539)]))

     f()

     print(dis.dis(f))
     ```

- 在 Python 中，多线程不会同时执行__Python代码__(是的，你听到了！)。直觉上可以产生多个线程并让它们同时执行你的Python代码，但是，由于 Python 中的 [Global Interpreter Lock](https://wiki.python.org/moin/GlobalInterpreterLock)，让你的线程轮流在同一个内核中执行。Python 线程适用于 IO 绑定任务，但为了实现 CPU 绑定任务的 Python 实际并行化，您可能需要使用 Python [multiprocessing](https://docs.python.org/2/library/multiprocessing.html) 模块。

- 用越界的坐标进行列表切片并不会产生错误或异常
  ```py
  >>> some_list = [1, 2, 3, 4, 5]
  >>> some_list[111:]
  []
  ```

- 在 Python 3 中 `int('١٢٣٤٥٦٧٨٩')` 返回 `123456789`。 在 Python 3 中，小数字符包括数字字符和可用于形成十进制小数数字的所有字符，例如 `U+0660`，ARABIC-INDIC DIGIT ZERO。这里有一个与 Python 的这种行为有关的有趣的故事[adventures-in-unicode-digits](http://chris.improbable.org/2014/8/25/adventures-in-unicode-digits/)。

- `'abc'.count('') == 4`. 下面是 `count` 方法的一个近似实现，它会使事情更加清晰。
  ```py
  def count(s, sub):
      result = 0
      for i in range(len(s) + 1 - len(sub)):
          result += (s[i:i + len(sub)] == sub)
      return result
  ```
  该行为是由于空字符串(`''`)与原始字符串中长度为 0 的切片的匹配所致。

---

# Contributing

All patches are Welcome! Please see [CONTRIBUTING.md](/CONTRIBUTING.md) for further details.

For discussions, you can either create a new [issue](https://github.com/satwikkansal/wtfpython/issues/new) or ping on the Gitter [channel](https://gitter.im/wtfpython/Lobby)

# Acknowledgements

The idea and design for this collection were initially inspired by Denys Dovhan's awesome project [wtfjs](https://github.com/denysdovhan/wtfjs). The overwhelming support by the community gave it the shape it is in right now.

#### Some nice Links!
* https://www.youtube.com/watch?v=sH4XF6pKKmk
* https://www.reddit.com/r/Python/comments/3cu6ej/what_are_some_wtf_things_about_python
* https://sopython.com/wiki/Common_Gotchas_In_Python
* https://stackoverflow.com/questions/530530/python-2-x-gotchas-and-landmines
* https://stackoverflow.com/questions/1011431/common-pitfalls-in-python
* https://www.python.org/doc/humor/
* https://www.satwikkansal.xyz/archives/posts/python/My-Python-archives/

# License

[![CC 4.0][license-image]][license-url]

&copy; [Satwik Kansal](https://satwikkansal.xyz)

[license-url]: http://www.wtfpl.net
[license-image]: https://img.shields.io/badge/License-WTFPL%202.0-lightgrey.svg?style=flat-square

## Help

If you have any wtfs, ideas or suggestions, please share.

## Want to share wtfpython with friends?

You can use these quick links for Twitter and Linkedin.

[Twitter](https://twitter.com/intent/tweet?url=https://github.com/satwikkansal/wtfpython&hastags=python,wtfpython) | 
[Linkedin](https://www.linkedin.com/shareArticle?url=https://github.com/satwikkansal&title=What%20the%20f*ck%20Python!&summary=An%20interesting%20collection%20of%20subtle%20and%20tricky%20Python%20snippets.)

## Need a pdf version?

I've received a few requests for the pdf version of wtfpython. You can add your details [here](https://satwikkansal.xyz/wtfpython-pdf/) to get the pdf as soon as it is finished.


[1]: https://github.com/satwikkansal/wtfpython/raw/master/images/logo.png
[2]: https://github.com/satwikkansal/wtfpython/releases/
[3]: https://github.com/satwikkansal/wtfPython
[4]: https://www.npmjs.com/package/wtfpython
[5]: https://pypi.python.org/pypi/wtfpython
[6]: https://github.com/satwikkansal/wtfpython/raw/master/images/string-intern/string_intern.png
[7]: https://stackoverflow.com/a/32211042/4354153
[8]: https://docs.python.org/3/reference/grammar.html
[9]: https://wiki.python.org/moin/Generators
[10]: https://docs.python.org/3/c-api/long.html
[11]: https://github.com/satwikkansal/wtfpython/raw/master/images/tic-tac-toe/after_row_initialized.png
[12]: https://github.com/satwikkansal/wtfpython/raw/master/images/tic-tac-toe/after_board_initialized.png
[13]: https://bugs.python.org/issue9232
[14]: https://bugs.python.org/issue9232#msg248399
[15]: https://docs.python.org/2/reference/lexical_analysis.html#string-literal-concatenation
[16]: https://stackoverflow.com/a/8169049/4354153
[17]: https://stackoverflow.com/questions/32139885/yield-in-list-comprehensions-and-generator-expressions
[18]: http://bugs.python.org/issue10544
[19]: https://docs.python.org/2/reference/datamodel.html
[20]: https://docs.python.org/3/reference/compound_stmts.html#except
[21]: http://docs.python.org/2/faq/design.html#why-doesn-t-list-sort-return-the-sorted-list
[22]: https://www.naftaliharris.com/blog/python-subclass-intransitivity/
[23]: https://docs.python.org/2/reference/simple_stmts.html#assignment-statements
[24]: https://en.wikipedia.org/wiki/Code_point
[25]: https://raw.githubusercontent.com/satwikkansal/wtfpython/master/mixed_tabs_and_spaces.py
[26]: https://stackoverflow.com/questions/44763802/bug-in-python-dict
[27]: https://stackoverflow.com/questions/45946228/what-happens-when-you-try-to-delete-a-list-element-while-iterating-over-it
[28]: https://stackoverflow.com/questions/45877614/how-to-change-all-the-dictionary-keys-in-a-for-loop-with-d-items
[29]: https://docs.python.org/3/whatsnew/3.0.html
[30]: http://sebastianraschka.com/Articles/2014_python_scope_and_namespaces.html
[31]: https://docs.python.org/2/reference/expressions.html#not-in



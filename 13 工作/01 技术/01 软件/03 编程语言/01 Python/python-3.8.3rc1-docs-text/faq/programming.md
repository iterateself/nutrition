编程常见问题
************


一般问题
========


Python 有没有提供断点与单步调试等功能的，源码层次的调试器？
-----------------------------------------------------------

有的。

以下介绍了一些 Python 的调试器，内置函数 "breakpoint()" 允许你使用其中
的任何一种。

pdb 模块是一个简单但是够用的控制台模式 Python 调试器。 它是标准 Python
库的一部分，并且 "已收录于库参考手册"。 你也可以通过使用 pdb 代码作为
样例来编写你自己的调试器。

作为标准 Python 发行版附带组件的 IDLE 交互式环境（通常位于
Tools/scripts/idle）中包含一个图形化的调试器。

PythonWin 是一个包含有基于 pdb 的 GUI 调试器的 Python IDE。 Pythonwin
调试器会为断点加上颜色，并具有许多很棒的特性，例如也可以非 Pythonwin
程序。 Pythonwin 是 Python for Windows Extensions 项目的一部分，也是
ActivePython 发行版的一部分（参见
https://www.activestate.com/activepython）。

Boa Constructor 是一个使用wxWidgets的IDE和GUI构建器。它提供可视化框架
创建和操作，对象检查器，源对象浏览器上的许多视图，继承层次结构，doc字
符串生成的html文档，高级调试器，集成帮助和Zope支持。

Eric 是一个基于PyQt和Scintilla编辑组件构建的IDE。

Pydb是标准Python调试器pdb的一个版本，经过修改后可与DDD（数据显示调试器
）一起使用，DDD是一种流行的图形化调试器前端。 Pydb可以在
http://bashdb.sourceforge.net/pydb/ 找到，DDD可以在
https://www.gnu.org/software/ddd 找到。

有许多商业Python IDE包括图形调试器。他们包括：

* Wing IDE (https://wingware.com/)

* Komodo IDE (https://komodoide.com/)

* PyCharm (https://www.jetbrains.com/pycharm/)


有没有工具来帮助找寻漏洞或进行静态分析？
----------------------------------------

有的。

PyChecker 是一个寻找Python代码漏洞以及对代码复杂性和风格给出警告的工具
。你可以从这里获得PyChecker: http://pychecker.sourceforge.net/ 。

Pylint 是另一个检查模块是否满足编码标准的工具，也可以编写插件来添加自
定义功能。除了PyChecker 执行的错误检查之外， Pylint 还提供了一些额外的
功能，例如检查行长度，变量名称是否根据您的编码标准格式良好，声明的接口
是否完全实现等等。 https://docs.pylint.org/ 提供了Pylint功能的完整列表
。

静态类型检查器，例如 Mypy 、 Pyre 和 Pytype 可以检查Python源代码中的类
型提示。


我如何能够通过一个 Python 脚本创建一个独立运行的二进制文件？
------------------------------------------------------------

如果你想要的只是一个独立的程序，用户可以下载和运行而不必先安装Python发
行版，你就不需要将Python编译成C代码。有许多工具可以确定程序所需的模块
集，并将这些模块与Python二进制文件绑定在一起以生成单个可执行文件。

一种是使用冻结工具，它包含在Python源代码树 "Tools/freeze" 中。它将
Python字节代码转换为C数组；一个C编译器，你可以将所有模块嵌入到一个新程
序中，然后将其与标准Python模块链接。

它的工作原理是递归扫描源代码以获取import语句（两种形式），并在标准
Python路径和源目录（用于内置模块）中查找模块。 然后，它将用Python编写
的模块的字节码转换为C代码（可以使用编组模块转换为代码对象的数组初始化
器），并创建一个定制的配置文件，该文件仅包含程序中实际使用的内置模块。
然后，它编译生成的C代码并将其与Python解释器的其余部分链接，以形成一个
独立的二进制文件，其行为与你的脚本完全相同。

显然， freeze 需要一个C编译器。有几个其他实用工具不需要。 一个是Thomas
Heller的py2exe（仅限Windows）

   http://www.py2exe.org/

另一个工具是 Anthony Tuininga 的 cx_Freeze。


是否有 Python 程序规范代码标准或风格指南？
------------------------------------------

有的。 请参阅标准库模块所要求的代码风格描述文档 **PEP 8** 。


核心语言
========


当变量有值时，为什么会出现UnboundLocalError？
---------------------------------------------

通过在函数体中的某处添加赋值语句，导致以前正常工作的代码被修改而得到
UnboundLocalError 会令人感到意外。

以下代码：

>>> x = 10
>>> def bar():
...     print(x)
>>> bar()
10

正常工作，但是以下代码

>>> x = 10
>>> def foo():
...     print(x)
...     x += 1

会得到一个 UnboundLocalError ：

>>> foo()
Traceback (most recent call last):
  ...
UnboundLocalError: local variable 'x' referenced before assignment

这是因为当你对作用域中的变量进行赋值时，该变量将成为该作用域的局部变量
，并在外部作用域中隐藏任何类似命名的变量。由于foo中的最后一个语句为
"x" 分配了一个新值，编译器会将其识别为局部变量。因此，当先前的
"print(x)" 尝试打印未初始化的局部变量时会导致错误。

在上面的示例中，你可以通过将其声明为全局来访问外部作用域变量：

>>> x = 10
>>> def foobar():
...     global x
...     print(x)
...     x += 1
>>> foobar()
10

这个显式声明是必需的，以便提醒你（与类和实例变量的表面类似情况不同），
你实际上是在外部作用域中修改变量的值

>>> print(x)
11

你可以使用 "nonlocal" 关键字在嵌套作用域中执行类似的操作：

>>> def foo():
...    x = 10
...    def bar():
...        nonlocal x
...        print(x)
...        x += 1
...    bar()
...    print(x)
>>> foo()
10
11


Python中的局部变量和全局变量有哪些规则？
----------------------------------------

在Python中，仅在函数内引用的变量是隐式全局变量。如果在函数体内的任何位
置为变量赋值，则除非明确声明为全局，否则将其视为局部值。

虽然起初有点令人惊讶，但片刻考虑就可以解释。一方面，要求 "global" 表示
已分配的变量可以防止意外的副作用。另一方面，如果所有全局引用都需要
"global" ，那么你一直都在使用 "global" 。你必须将对内置函数或导入模块
的组件的每个引用声明为全局。这种杂乱会破坏 "global" 声明用于识别副作用
的有用性。


为什么在具有不同值的循环中定义的lambdas都返回相同的结果？
---------------------------------------------------------

假设你使用for循环来定义几个不同的 lambda （甚至是普通函数），例如：:

   >>> squares = []
   >>> for x in range(5):
   ...     squares.append(lambda: x**2)

这给你一个包含5个lambdas的列表，它们计算 "x**2" 。你可能会期望，当它们
被调用时，它们将分别返回 "0" 、 "1" 、 "4" 、 "9" 和 "16" 。但是，当你
真正尝试时，你会看到它们都返回 "16" 。:

   >>> squares[2]()
   16
   >>> squares[4]()
   16

发生这种情况是因为 "x" 不是lambdas的内部变量，而是在外部作用域中定义，
并且在调用lambda时访问它 - 而不是在定义它时。 在循环结束时， "x" 的值
是 "4" ，所以所有的函数现在返回 "4**2" ，即 "16" 。你还可以通过更改
"x" 的值来验证这一点，并查看lambdas的结果如何变化:

   >>> x = 8
   >>> squares[2]()
   64

为了避免这种情况，你需要将值保存在lambdas的局部变量中，这样它们就不依
赖于全局``x`` 的值

   >>> squares = []
   >>> for x in range(5):
   ...     squares.append(lambda n=x: n**2)

这里， "n=x" 在lambda本地创建一个新的变量 "n" ，并在定义lambda时计算，
使它具有与 "x" 在循环中该点相同的值。这意味着 "n" 的值在第一个lambda中
为 "0" ，在第二个lambda中为 "1" ，在第三个中为 "2" ，依此类推。因此每
个lambda现在将返回正确的结果:

   >>> squares[2]()
   4
   >>> squares[4]()
   16

请注意，这种行为并不是lambda所特有的，但也适用于常规函数。


如何跨模块共享全局变量？
------------------------

在单个程序中跨模块共享信息的规范方法是创建一个特殊模块（通常称为config
或cfg）。只需在应用程序的所有模块中导入配置模块；然后该模块可用作全局
名称。因为每个模块只有一个实例，所以对模块对象所做的任何更改都会在任何
地方反映出来。 例如：

config.py:

   x = 0   # Default value of the 'x' configuration setting

mod.py:

   import config
   config.x = 1

main.py:

   import config
   import mod
   print(config.x)

请注意，出于同样的原因，使用模块也是实现Singleton设计模式的基础。


导入模块的“最佳实践”是什么？
----------------------------

通常，不要使用 "from modulename import *" 。这样做会使导入器的命名空间
变得混乱，并且使得连接器更难以检测未定义的名称。

在文件的顶部导入模块。这样做可以清楚地了解代码所需的其他模块，并避免了
模块名称是否在范围内的问题。每行导入一个模块可以轻松添加和删除导入的模
块，但每行导入多个模块会占用更少的屏幕空间。

如果按以下顺序导入模块，这是一种很好的做法：

1. 标准库模块 -- 例如： "sys", "os", "getopt", "re"

2. 第三方库模块（安装在Python的site-packages目录中的任何内容） --  例
   如mx.DateTime，ZODB，PIL.Image等

3. 本地开发的模块

有时需要将模块导入语句移动到函数或类里面，以避免循环导入问题。Gordon
McMillan 说：

   当两个模块都使用  "import <module>" 的导入形式时，循环导入就可以了
   。但是当第 2 个模块想从第 1 个模块中获取一个名称 ("from module
   import name") 并且导入位于顶层时，就会出错。 这是因为第 1 个模块中
   的名称还不可用，因为第 1 个模块正在忙着导入第 2 个模块。

在这种情况下，如果第二个模块仅用于一个函数，则可以轻松地将模块导入语句
移动到该函数中。调用导入时，第一个模块将完成初始化，第二个模块可以进行
导入。

如果某些模块是特定于平台的，则可能还需要将模块导入语句移出顶级代码。在
这种情况下，甚至可能无法导入文件顶部的所有模块。在这种情况下，在相应的
特定于平台的代码中导入正确的模块是一个很好的选择。

只有当需要解决诸如避免循环导入或试图减少模块初始化时间的问题时，才可以
将导入移动到本地范围，例如在函数定义中。如果根据程序的执行方式，许多导
入是不必要的，这种技术尤其有用。如果仅在某个函数中使用模块，您还可能希
望将导入移到该函数中。请注意，第一次加载模块可能会因为模块的一次初始化
而代价高昂，但多次加载模块实际上是免费的，只需进行几次字典查找。即使模
块名称超出了作用域，模块也可能在 "sys.modules" 中可用。


为什么对象之间会共享默认值？
----------------------------

这种类型的缺陷通常会惹恼新手程序员。考虑这个函数

   def foo(mydict={}):  # Danger: shared reference to one dict for all calls
       ... compute something ...
       mydict[key] = value
       return mydict

第一次调用此函数时，"mydict" 包含一项。第二次，"mydict" 包含两项，因为
当 "foo()" 开始执行时， "mydict" 中已经有一项了。

函数调用经常被期望为默认值创建新的对象。 但实际情况并非如此。 默认值会
在函数定义时一次性地创建。 如果对象发生改变，就如本示例中的字典那样，
则对函数的后续调用将会引用这个被改变的对象。

按照定义，不可变对象例如数字、字符串、元组和 "None" 因为不可变所以是安
全的。 对可变对象例如字典、列表和类实例的改变则可能造成迷惑。

由于这一特性，在编程中应遵循的一项好习惯是不使用可变对象作为默认值。
而应使用 "None" 作为默认值和函数中的值，检查值为 "None" 的形参并创建相
应的列表、字典或其他可变对象。 例如，不要这样写:

   def foo(mydict={}):
       ...

而要这样写:

   def foo(mydict=None):
       if mydict is None:
           mydict = {}  # create a new dict for local namespace

这一特性有时会很有用处。 当你有一个需要进行大量耗时计算的函数时，一个
常见技巧是将每次调用函数的参数和结果值缓存起来，并在同样的值被再次请求
时返回缓存的值。 这称为“记忆”，具体实现方式可以是这样的:

   # Callers can only provide two parameters and optionally pass _cache by keyword
   def expensive(arg1, arg2, *, _cache={}):
       if (arg1, arg2) in _cache:
           return _cache[(arg1, arg2)]

       # Calculate the value
       result = ... expensive computation ...
       _cache[(arg1, arg2)] = result           # Store result in the cache
       return result

你也可以使用包含一个字典的全局变量而不使用参数默认值；这完全取决于个人
偏好。


如何将可选参数或关键字参数从一个函数传递到另一个函数？
------------------------------------------------------

使用函数参数列表中的 "*" 和 "**" 说明符收集参数;这会将位置参数作为元组
，将关键字参数作为字典。然后，您可以使用 "*" 和 "**" 调用另一个函数时
传递这些参数：

   def f(x, *args, **kwargs):
       ...
       kwargs['width'] = '14.3c'
       ...
       g(x, *args, **kwargs)


形参和实参之间有什么区别？
--------------------------

*形参* 是指出现在函数定义中的名称，而 *实参* 则是在调用函数时实际传入
的值。 形参定义了一个函数能接受何种类型的实参。 例如，对于以下函数定义
:

   def func(foo, bar=None, **kwargs):
       pass

*foo*, *bar* 和 *kwargs* 是 "func" 的形参。 但是，在调用 "func" 时，例
如：

   func(42, bar=314, extra=somevar)

实际的值 "42", "314" 和 "somevar" 则是实参。


为什么更改列表 'y' 也会更改列表 'x'？
-------------------------------------

如果你编写的代码就像下面一样：

   >>> x = []
   >>> y = x
   >>> y.append(10)
   >>> y
   [10]
   >>> x
   [10]

你可能想知道为什么追加一个元素也改变了x。

产生这种结果有两个因素：

1. 变量只是指向具体对象的名称。 执行 "y = x" 并不会为列表创建一个副本
   —— 它只是创建了一个新变量 "y" 指向 "x" 所指向的同一对象。 这意味着
   只存在一个对象（列表），"x" 和 "y" 都是对它的引用。

2. 列表属于 *mutable* 对象，这意味着你可以改变它的内容。

在调用 "append()" 之后，这个可变对象的内容由 "[]" 变为 "[10]"。 由于两
个变量都指向同一对象，因此使用任何一个名称所访问到的都是修改后的值
"[10]"。

如果我们改为将不可变对象赋值给 "x":

   >>> x = 5  # ints are immutable
   >>> y = x
   >>> x = x + 1  # 5 can't be mutated, we are creating a new object here
   >>> x
   6
   >>> y
   5

我们可以看到在此情况下 "x" 和 "y" 就不再相等了。 这是因为整数是
*immutable* 对象，当我们执行 "x = x + 1" 时我们并不是改变了 "5" 这个对
象的值；而是创建了一个新的对象 (整数 "6") 并将其赋值给 "x" (也就是改变
了 "x" 所指向的对象)。 在赋值之后我们就有了两个对象 (整数 "6" 和 "5")
以及分别指向它们的两个变量 ("x" 现在指向 "6" 而 "y" 仍然指向 "5")。

某些操作 (例如 "y.append(10)" 和 "y.sort()") 是改变原对象，而看上去相
似的另一些操作 (例如 "y = y + [10]" 和 "sorted(y)") 则是创建新对象。
通常在 Python 中 (以及在标准库的所有代码中) 会改变原对象的方法将返回
"None" 以帮助避免混淆这两种不同类型的操作。 因此如果你错误地使用了
"y.sort()" 并期望它将返回一个经过排序的 "y" 的副本，你得到的结果将会是
"None"，这将导致你的程序产生一个容易诊断的错误。

但是，还存在一类操作，不同的类型执行相同的操作会有不同的行为：那就是增
强赋值运算符。 例如，"+=" 会原地改变列表，但不会改变元组或整数
("a_list += [1, 2, 3]" 与 "a_list.extend([1, 2, 3])" 一样都会改变
"a_list"，而 "some_tuple += (1, 2, 3)" 和 "some_int += 1" 则会创建新的
对象)。

换而言之：

* 如果我们有一个可变对象 ("list", "dict", "set" 等等)，我们可以使用某
  些特定的操作来改变它，所有指向它的变量都会显示它的改变。

* 如果我们有一个不可变对象 ("str", "int", "tuple" 等等)，所有指向它的
  变量都将显示相同样的值，但凡是会改变这个值的操作将总是返回一个新对象
  。

如果你想知道两个变量是否指向相同的对象，你可以使用 "is" 运算符，或内置
函数 "id()"。


如何编写带输出参数的函数（通过引用调用）？
------------------------------------------

请记住在 Python 中参数是通过赋值来传递的。 由于赋值只是创建了对象的引
用，因此在调用者和被调用者的参数名称之间没有别名，所以本身是没有按引用
调用的。 你可以通过多种方式实现所需的效果。

1. 通过返回一个结果元组:

      def func2(a, b):
          a = 'new-value'        # a and b are local names
          b = b + 1              # assigned to new objects
          return a, b            # return new values

      x, y = 'old-value', 99
      x, y = func2(x, y)
      print(x, y)                # output: new-value 100

   这几乎总是最清晰明了的解决方案。

2. 通过使用全局变量。 这种方式不是线程安全的，而且也不受推荐。

3. 通过传递一个可变 (即可原地修改的) 对象:

      def func1(a):
          a[0] = 'new-value'     # 'a' references a mutable list
          a[1] = a[1] + 1        # changes a shared object

      args = ['old-value', 99]
      func1(args)
      print(args[0], args[1])    # output: new-value 100

4. 通过传递一个会被改变的字典:

      def func3(args):
          args['a'] = 'new-value'     # args is a mutable dictionary
          args['b'] = args['b'] + 1   # change it in-place

      args = {'a': 'old-value', 'b': 99}
      func3(args)
      print(args['a'], args['b'])

5. 或者在一个类实例中捆绑值:

      class callByRef:
          def __init__(self, /, **args):
              for key, value in args.items():
                  setattr(self, key, value)

      def func4(args):
          args.a = 'new-value'        # args is a mutable callByRef
          args.b = args.b + 1         # change object in-place

      args = callByRef(a='old-value', b=99)
      func4(args)
      print(args.a, args.b)

   几乎没有任何适当理由将问题如此复杂化。

你的最佳选择是返回一个包含多个结果的元组。


如何在Python中创建高阶函数？
----------------------------

你有两种选择：使用嵌套作用域，或者使用可调用对象。 例如，假设你想要定
义 "linear(a,b)" 使其返回一个函数 "f(x)" 来设计 "a*x+b" 的值。 可以使
用以下嵌套作用域:

   def linear(a, b):
       def result(x):
           return a * x + b
       return result

或使用一个可调用对象:

   class linear:

       def __init__(self, a, b):
           self.a, self.b = a, b

       def __call__(self, x):
           return self.a * x + self.b

在两种情况下，:

   taxes = linear(0.3, 2)

都会给出一个可调用对象，使得 "taxes(10e6) == 0.3 * 10e6 + 2".

可调用对象方式的缺点是速度略慢且生成的代码略长。 但是，请注意一组可调
用对象能够通过继承来共享签名:

   class exponential(linear):
       # __init__ inherited
       def __call__(self, x):
           return self.a * (x ** self.b)

对象可以封装多个方法的状态:

   class counter:

       value = 0

       def set(self, x):
           self.value = x

       def up(self):
           self.value = self.value + 1

       def down(self):
           self.value = self.value - 1

   count = counter()
   inc, dec, reset = count.up, count.down, count.set

这里 "inc()", "dec()" 和 "reset()" 将表现为共享同一计数变量的多个函数
。


如何在Python中复制对象？
------------------------

一般来说，通常情况下请尝试 "copy.copy()" 或 "copy.deepcopy()"。 不是所
有对象都可以复制，但多数都是可以的。

某些对象可以方便地复制。 例如字典具有 "copy()" 方法:

   newdict = olddict.copy()

序列可以通过切片来复制:

   new_l = l[:]


如何找到对象的方法或属性？
--------------------------

对于一个用户自定义类的实例 x，"dir(x)" 将返回一个按字母顺序排序的包含
实例属性和方法及其类所定义的属性名称的列表。


我的代码如何才能发现对象的名称？
--------------------------------

通常来说是做不到的，因为对象并不真正具有名称。 在本质上，赋值总是会将
一个名称绑定到某个值；"def" 和 "class" 语句也是如此，但在这种情况下该
值是一个可调用对象。 考虑以下代码:

   >>> class A:
   ...     pass
   ...
   >>> B = A
   >>> a = B()
   >>> b = a
   >>> print(b)
   <__main__.A object at 0x16D07CC>
   >>> print(a)
   <__main__.A object at 0x16D07CC>

不严谨地讲，该类有一个名称：虽然它是绑定了两个名称并通过名称 B 发起调
用，所创建的实例仍然被视为类 A 的一个实例。 但是实例的名称则无法确定地
说是 a 或是 b，因为有两个名称被绑定到了同一个值。

一般来说你的代码应该没有必要“知道”特定值的名称。 除非你是在编写特殊的
内省程序，出现这样的问题通常表明如果改变方式可能会更有利。

在 comp.lang.python 中，Fredrik Lundh 在回答这样的问题时曾经给出过一个
绝佳的类比：

   跟你找出在你家门廊见到的某只猫的名字所用的办法一样：猫（对象）自己
   无法告诉你它的名字，它根本就不在乎 —— 所以找出它叫什么名字的唯一办
   法是问你的所有邻居（命名空间）那是不是他们的猫（对象）……

   ……并且如果你发现它有很多名字或根本没有名字也不必觉得惊讶！


逗号运算符的优先级是什么？
--------------------------

逗号在 Python 中不是运算符。 考虑这个例子:

   >>> "a" in "b", "a"
   (False, 'a')

由于逗号不是运算符而是表达式之间的分隔符，以上代码的含义就相当于:

   ("a" in "b"), "a"

而不是:

   "a" in ("b", "a")

对于各种赋值运算符 ("=", "+=" 等) 来说同样如此。 它们并不是真正的运算
符而是赋值语句中的语法分隔符。


是否有与 C 的 "?:" 三目运算符等价的东西？
-----------------------------------------

有的。 相应语法如下:

   [on_true] if [expression] else [on_false]

   x, y = 50, 25
   small = x if x < y else y

在 Python 2.5 引入此语法之前，常见的做法是使用逻辑运算符:

   [expression] and [on_true] or [on_false]

然而这种做法并不保险，因为当 *on_true* 具有布尔假值时将会给出错误的结
果。 所以，使用 "... if ... else ..." 形式总是会更好。


是否可以用Python编写混淆的单行程序?
-----------------------------------

可以。通常是在 "lambda" 中嵌套 "lambda" 来实现的。请参阅以下三个来自
Ulf Bartelt 的示例代码：

   from functools import reduce

   # Primes < 1000
   print(list(filter(None,map(lambda y:y*reduce(lambda x,y:x*y!=0,
   map(lambda x,y=y:y%x,range(2,int(pow(y,0.5)+1))),1),range(2,1000)))))

   # First 10 Fibonacci numbers
   print(list(map(lambda x,f=lambda x,f:(f(x-1,f)+f(x-2,f)) if x>1 else 1:
   f(x,f), range(10))))

   # Mandelbrot set
   print((lambda Ru,Ro,Iu,Io,IM,Sx,Sy:reduce(lambda x,y:x+y,map(lambda y,
   Iu=Iu,Io=Io,Ru=Ru,Ro=Ro,Sy=Sy,L=lambda yc,Iu=Iu,Io=Io,Ru=Ru,Ro=Ro,i=IM,
   Sx=Sx,Sy=Sy:reduce(lambda x,y:x+y,map(lambda x,xc=Ru,yc=yc,Ru=Ru,Ro=Ro,
   i=i,Sx=Sx,F=lambda xc,yc,x,y,k,f=lambda xc,yc,x,y,k,f:(k<=0)or (x*x+y*y
   >=4.0) or 1+f(xc,yc,x*x-y*y+xc,2.0*x*y+yc,k-1,f):f(xc,yc,x,y,k,f):chr(
   64+F(Ru+x*(Ro-Ru)/Sx,yc,0,0,i)),range(Sx))):L(Iu+y*(Io-Iu)/Sy),range(Sy
   ))))(-2.1, 0.7, -1.2, 1.2, 30, 80, 24))
   #    \___ ___/  \___ ___/  |   |   |__ lines on screen
   #        V          V      |   |______ columns on screen
   #        |          |      |__________ maximum of "iterations"
   #        |          |_________________ range on y axis
   #        |____________________________ range on x axis

请不要在家里尝试，骚年！


函数参数列表中的斜杠（/）是什么意思？
-------------------------------------

函数参数列表中的斜杠表示在它之前的形参是仅限位置形参。 仅限位置形参没
有外部可用的名称。  在调用接受仅限位置形参的函数时，参数只会基于它们的
位置被映射到形参。 例如，"divmod()" 是一个接受仅限位置形参的函数。 它
的文档是这样的:

   >>> help(divmod)
   Help on built-in function divmod in module builtins:

   divmod(x, y, /)
       Return the tuple (x//y, x%y).  Invariant: div*y + mod == x.

在形参列表末尾的斜杠意味着两个形参都是仅限位置形参。 因此，附带关键字
参数调用 "divmod()" 将会导致报错:

   >>> divmod(x=3, y=4)
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   TypeError: divmod() takes no keyword arguments


数字和字符串
============


如何指定十六进制和八进制整数？
------------------------------

要指定一个八进制数码，则在八进制值之前加一个零和一个小写或大写字母 "o"
作为前缀。 例如，要将变量 "a" 设为八进制的 "10" (十进制的 8)，就输入:

   >>> a = 0o10
   >>> a
   8

十六进制数也同样简单。 只要在十六进制数之前加一个零和一个小写或大写字
母 "x"。 十六进制数码中的字母可以为大写或小写。 例如在 Python 解释器中
输入:

   >>> a = 0xa5
   >>> a
   165
   >>> b = 0XB2
   >>> b
   178


为什么-22 // 10返回-3？
-----------------------

这主要是为了让 "i % j" 的正负与 "j" 一致，如果你想要这样的结果，并且又
想要:

   i == (i // j) * j + (i % j)

那么整除就必须向下取整。 C 同样要求保持一致，并且编译器在截短 "i // j"
的结果值时需要使 "i % j" 的正负与 "i" 一致。

对于 "i % j" 来说 "j" 为负值的应用场景实际上是非常少的。 而 "j" 为正值
的情况则非常多，并且实际上在所有情况下让 "i % j" 的结果为 ">= 0" 会更
有用处。 如果现在时间为 10 时，那么 200 小时前应是几时？ "-190 % 12 ==
2" 是有用处的；"-190 % 12 == -10" 则是会导致意外的漏洞。


如何将字符串转换为数字？
------------------------

对于整数，可使用内置的 "int()" 类型构造器，例如 "int('144') == 144"。
类似地，可使用 "float()" 转换为浮点数，例如 "float('144') == 144.0"。

默认情况下，这些操作会将数字按十进制来解读，因此 "int('0144') == 144"
而 "int('0x144')" 会引发 "ValueError"。 "int(string, base)" 接受第二个
可选参数指定转换的基数，例如 "int('0x144', 16) == 324"。 如果指定基数
为 0，则按 Python 规则解读数字：前缀 '0o' 表示八进制，而 '0x' 表示十六
进制。

如果你只是想将字符串转为数字，请不要使用内置函数 "eval()"。 "eval()"
的速度会慢很多并且有安全风险：别人可能会传入具有你不想要的附带效果的
Python 表达式。 例如，别人可以传入 "__import__('os').system("rm -rf
$HOME")" 这将删除你的家目录。

"eval()" 还具有将数字解读为 Python 表达式的效果，这样 "eval('09')" 将
会导致语法错误，因为 Python 不允许十进制数的首位是 '0' ('0' 除外)。


如何将数字转换为字符串？
------------------------

例如要将数字 144 转换为字符串 '144'，可使用内置类型构造器 "str()"。 如
果想要表示为十六进制或八进制数，可使用内置函数 "hex()" 或 "oct()"。 想
要更好地格式化，请参阅 格式化字符串字面值 和 格式字符串语法 等小节，例
如 ""{:04d}".format(144)" 生成 "'0144'" 而 ""{:.3f}".format(1.0/3.0)"
生成 "'0.333'"。


如何修改字符串？
----------------

无法修改，因为字符串是不可变对象。 在大多数情况下，你应该使用你想要的
各种部分来构造一个新字符串。 但是，如果你想要一个可以原地修改 Unicode
数据的对象，可尝试使用 "io.StringIO" 对象或 "array" 模块:

   >>> import io
   >>> s = "Hello, world"
   >>> sio = io.StringIO(s)
   >>> sio.getvalue()
   'Hello, world'
   >>> sio.seek(7)
   7
   >>> sio.write("there!")
   6
   >>> sio.getvalue()
   'Hello, there!'

   >>> import array
   >>> a = array.array('u', s)
   >>> print(a)
   array('u', 'Hello, world')
   >>> a[0] = 'y'
   >>> print(a)
   array('u', 'yello, world')
   >>> a.tounicode()
   'yello, world'


如何使用字符串调用函数/方法？
-----------------------------

有多种技巧可供选择。

* 最好的做法是使用一个将字符串映射到函数的字典。 这一技巧的主要优势在
  于字符串不必与函数名称一致。 这也是用于模拟其他语言中 case 结构的主
  要技巧:

     def a():
         pass

     def b():
         pass

     dispatch = {'go': a, 'stop': b}  # Note lack of parens for funcs

     dispatch[get_input()]()  # Note trailing parens to call function

* 使用内置函数 "getattr()"

     import foo
     getattr(foo, 'bar')()

  请注意 "getattr()" 可用于任何对象，包括类、类实例、模块等等。

  在标准库中多次使用了这个技巧，例如:

     class Foo:
         def do_foo(self):
             ...

         def do_bar(self):
             ...

     f = getattr(foo_instance, 'do_' + opname)
     f()

* 使用 "locals()" 或 "eval()" 来解析出函数名:

     def myFunc():
         print("hello")

     fname = "myFunc"

     f = locals()[fname]
     f()

     f = eval(fname)
     f()

  注意：使用 "eval()" 速度慢而且危险。 如果你不能绝对掌控字符串的内容
  ，别人将能传入可被解析为任意函数直接执行的字符串。


是否有与Perl 的chomp() 等效的方法，用于从字符串中删除尾随换行符？
-----------------------------------------------------------------

可以使用 "S.rstrip("\r\n")" 从字符串 "S" 的末尾删除所有的换行符，而不
删除其他尾随空格。如果字符串 "S" 表示多行，且末尾有几个空行，则将删除
所有空行的换行符：

   >>> lines = ("line 1 \r\n"
   ...          "\r\n"
   ...          "\r\n")
   >>> lines.rstrip("\n\r")
   'line 1 '

由于通常只在一次读取一行文本时才需要这样做，所以使用 "S.rstrip()" 这种
方式工作得很好。


是否有 scanf() 或 sscanf() 的对应物？
-------------------------------------

没有这样的对应物。

对于简单的输入解析，最方便的做法通常是使用字符串对象的 "split()" 方法
将一行内容拆解为以空格分隔的单词，然后使用 "int()" 或 "float()" 将表示
十进制数的字符串转换为数值。 "split()" 支持可选的 "sep" 形参，适用于内
容行使用空格符以外的分隔符的情况。

以于更复杂的输入解析，正则表达式会比 C 的 "sscanf()" 更强大，也更适合
此类任务。


'UnicodeDecodeError' 或 'UnicodeEncodeError' 错误是什么意思？
-------------------------------------------------------------

见 Unicode 指南


性能
====


我的程序太慢了。该如何加快速度？
--------------------------------

总的来说，这是个棘手的问题。首先，下面列出了深入了解前需要记住的事情：

* 不同的 Python 实现具有不同的性能特点。 本 FAQ 着重解答的是 *CPython*
  。

* 行为可能因操作系统而异，尤其是在谈论 I / O 或多线程时。

* 在尝试优化任何代码 *前* ，应始终找到程序中的热点（请参阅 "profile"
  模块）。

* 编写基准脚本将允许您在搜索改进时快速迭代（请参阅 "timeit" 模块）。

* 强烈建议在可能引入隐藏在复杂优化中的回归之前，要有良好的代码覆盖率（
  通过单元测试或任何其他技术）。

话虽如此，加速Python代码有很多技巧。以下是一些可以达到可接受的性能水平
的一般原则：

* 使您的算法更快（或更改为更快的算法）可以产生比尝试在代码中使用微优化
  技巧更大的好处。

* 使用正确的数据结构。参考文档 内置类型 和 "collections" 模块。

* 当标准库提供用于执行某些操作的原语时，可能（尽管不能保证）比您可能提
  出的任何替代方案更快。对于用C编写的原语，例如内置函数和一些扩展类型
  ，这是真的。例如，请确保使用 "list.sort()" 内置方法或相关的
  "sorted()" 函数进行排序（有关适度高级用法的示例，请参阅 排序指南 ）
  。

* 抽象倾向于创造间接性并迫使翻译更多地工作。如果间接级别超过完成的有用
  工作量，则程序将变慢。你应该避免过度抽象，特别是在微小的功能或方法的
  形式下（这通常也会对可读性产生不利影响）。

如果你已经达到纯 Python 允许的限制，那么有一些工具可以让你走得更远。
例如， Cython 可以将稍微修改的 Python 代码版本编译为 C 扩展，并且可以
在许多不同的平台上使用。 Cython 可以利用编译（和可选的类型注释）来使代
码明显快于解释运行时的速度。 如果您对 C 编程技能有信心，也可以自己 编
写 C 扩展模块 。

参见: 专门介绍 性能提示 的wiki页面。


将多个字符串连接在一起的最有效方法是什么？
------------------------------------------

"str" 和 "bytes" 对象是不可变的，因此将多个字符串连接在一起效率很低，
因为每个连接都会创建一个新对象。在一般情况下，总运行时间是总字符串长度
的二次方。

要连接多个 "str" 对象，通常推荐的用法是将它们放入一个列表中并在结尾处
调用 "str.join()" ：

   chunks = []
   for s in my_strings:
       chunks.append(s)
   result = ''.join(chunks)

（另一个合理有效的惯用方法是 "io.StringIO" ）

要连接多个 "str" 对象，建议使用本地连接（ "+=" 运算符）扩展
"bytearray" 对象：

   result = bytearray()
   for b in my_bytes_objects:
       result += b


序列（元组/列表）
=================


如何在元组和列表之间进行转换？
------------------------------

类型构造器 "tuple(seq)" 可将任意序列（实际上是任意可迭代对象）转换为具
有相同排列顺序的相同条目的元组。

例如，"tuple([1, 2, 3])" 产生 "(1, 2, 3)" 而 "tuple('abc')" 产生
"('a', 'b', 'c')"。 如果参数为一个元组，它不会创建副本而是返回同一对象
，因此如果你不确定某个对象是否为元组时也可简单地调用 "tuple()"。

类型构造器 "list(seq)" 可将任意序列或可迭代对象转换为具有相同排列顺序
的相同条目的列表。 例如，"list((1, 2, 3))" 产生 "[1, 2, 3]" 而
"list('abc')" 产生 "['a', 'b', 'c']"。 如果参数为一个列表，它会像
"seq[:]" 那样创建一个副本。


什么是负数序号？
----------------

Python 序列使用正数或负数作为序号或称索引号。 对于正数序号，第一个序号
为 0 而 1 为第二个序号，依此类推。 对于负数序号，倒数第一个序号为 -1
而倒数第二个序号为 -2，依此类推。 可以认为 "seq[-n]" 就相当于
"seq[len(seq)-n]"。

使用负数序号有时会很方便。 例如 "S[:-1]" 就是原字符串去掉最后一个字符
，这可以用来移除某个字符串末尾的换行符。


如何以相反的顺序迭代序列？
--------------------------

使用 "reversed()" 内置函数，这是Python 2.4中的新功能:

   for x in reversed(sequence):
       ...  # do something with x ...

这不会修改您的原始序列，而是构建一个反向顺序的新副本以进行迭代。

在 Python 2.3 里，您可以使用扩展切片语法:

   for x in sequence[::-1]:
       ...  # do something with x ...


如何从列表中删除重复项？
------------------------

有关执行此操作的许多方法的详细讨论，请参阅 Python Cookbook:

   https://code.activestate.com/recipes/52560/

如果您不介意重新排序列表，请对其进行排序，然后从列表末尾进行扫描，删除
重复项：

   if mylist:
       mylist.sort()
       last = mylist[-1]
       for i in range(len(mylist)-2, -1, -1):
           if last == mylist[i]:
               del mylist[i]
           else:
               last = mylist[i]

如果列表的所有元素都可以用作设置键（即：它们都是 *hashable* ），这通常
会更快:

   mylist = list(set(mylist))

这会将列表转换为集合，从而删除重复项，然后返回到列表中。


如何在Python中创建数组？
------------------------

使用列表:

   ["this", 1, "is", "an", "array"]

列表在时间复杂度方面相当于C或Pascal数组；主要区别在于，python列表可以
包含许多不同类型的对象。

"array" 模块还提供了创建具有紧凑表示的固定类型的数组的方法，但它的索引
速度比列表慢。还要注意，数字扩展和其他扩展还定义了具有各种特性的类似数
组的结构。

要获取Lisp样式的列表，可以使用元组模拟cons单元：

   lisp_list = ("like",  ("this",  ("example", None) ) )

如果需要可变性，可以使用列表而不是元组。这里模拟lisp car的是
"lisp_list[0]" ，模拟cdr的是 "lisp_list[1]" 。只有在你确定真的需要的时
候才这样做，因为它通常比使用Python列表慢得多。


如何创建多维列表？
------------------

你可能试图制作一个像这样的多维数组:

   >>> A = [[None] * 2] * 3

如果你打印它，看起来是正确的：

   >>> A
   [[None, None], [None, None], [None, None]]

但是，当你给某一项赋值时，会同时在多个位置显示变化：

   >>> A[0][0] = 5
   >>> A
   [[5, None], [5, None], [5, None]]

其中的原因在于使用 "*" 对列表执行重复操作并不是创建副本，它只是创建现
有对象的引用。 "*3" 创建了对长度为二的同一列表的 3 个引用。 对某一行的
改变会作用于所有行，通常这一定不是你所希望的。

建议的做法是先创建一个所需长度的列表，然后其中的元素再以一个新创建的列
表来填充:

   A = [None] * 3
   for i in range(3):
       A[i] = [None] * 2

这样就生成了一个包含 3 个长度为二的不同列表的列表。 你也可以使用列表推
导式:

   w, h = 2, 3
   A = [[None] * w for i in range(h)]

或者你还可以使用提供矩阵类型的扩展包；其中最著名的是 NumPy。


如何将方法应用于一系列对象？
----------------------------

可以使用列表推导式：

   result = [obj.method() for obj in mylist]


为什么 a_tuple[i] += ['item'] 会在执行加法时引发异常？
------------------------------------------------------

这是由两个事实共同导致的结果，一是增强赋值运算符属于 *赋值* 运算符，二
是在 Python 中存在可变和不可变两种不同的对象。

此处的讨论在任何对元组中指向可变对象的元素使用增强赋值运算符的情况都是
普遍成立的，但在此我们只以 "list" 和 "+=" 来举例。

如果你写成这样:

   >>> a_tuple = (1, 2)
   >>> a_tuple[0] += 1
   Traceback (most recent call last):
      ...
   TypeError: 'tuple' object does not support item assignment

发生异常的原因是显而易见的: "1" 会与对象 "a_tuple[0]" 相加，而该对象为
("1")，得到结果对象 "2"，但当我们试图将运算结果 "2" 赋值给元组的 "0"
号元素时就将报错，因为我们不能改变元组的元素所指向的对象。

在表层之处，以上增强赋值语句所做的大致是这样:

   >>> result = a_tuple[0] + 1
   >>> a_tuple[0] = result
   Traceback (most recent call last):
     ...
   TypeError: 'tuple' object does not support item assignment

由于元组是不可变的，因此操作的赋值部分会引发错误。

当你这样写的时候:

   >>> a_tuple = (['foo'], 'bar')
   >>> a_tuple[0] += ['item']
   Traceback (most recent call last):
     ...
   TypeError: 'tuple' object does not support item assignment

发生异常会令人略感吃惊，还有一个更为令人吃惊的事实：虽然有报错，但是添
加操作却生效了:

   >>> a_tuple[0]
   ['foo', 'item']

要明白为何会这样，你需要知道 (a) 如果一个对象实现了 "__iadd__" 魔术方
法，它会在执行 "+=" 增强赋值时被调用，并且其返回值将用于该赋值语句；
(b) 对于列表来说，"__iadd__" 等价于在列表上调用 "extend" 并返回该列表
。 因此对于列表我们可以说 "+=" 就是 "list.extend" 的“快捷方式”:

   >>> a_list = []
   >>> a_list += [1]
   >>> a_list
   [1]

这相当于:

   >>> result = a_list.__iadd__([1])
   >>> a_list = result

a_list 所引用的对象已被修改，而引用被修改对象的指针又重新被赋值给
"a_list"。 赋值的最终结果没有变化，因为它是引用 "a_list" 之前所引用的
同一对象的指针，但仍然发生了赋值操作。

因此，在我们的元组示例中，发生的事情等同于：

   >>> result = a_tuple[0].__iadd__(['item'])
   >>> a_tuple[0] = result
   Traceback (most recent call last):
     ...
   TypeError: 'tuple' object does not support item assignment

"__iadd__" 成功执行，因此列表得到了扩充，但是虽然 "result" 指向了
"a_tuple[0]" 已经指向的同一对象，最后的赋值仍然导致了报错，因为元组是
不可变的。


我想做一个复杂的排序：你能用Python做一个Schwartzian变换吗？
-----------------------------------------------------------

该技术归功于Perl社区的 Randal Schwartz，它通过将每个元素映射到其 "排序
值（sort value）" 的度量对列表中的元素进行排序。在Python中，使用
"list.sort()" 方法的 "key" 参数：

   Isorted = L[:]
   Isorted.sort(key=lambda s: int(s[10:15]))


如何按其他列表中的值对一个列表进行排序？
----------------------------------------

将它们合并到元组的迭代器中，对结果列表进行排序，然后选择所需的元素。

   >>> list1 = ["what", "I'm", "sorting", "by"]
   >>> list2 = ["something", "else", "to", "sort"]
   >>> pairs = zip(list1, list2)
   >>> pairs = sorted(pairs)
   >>> pairs
   [("I'm", 'else'), ('by', 'sort'), ('sorting', 'to'), ('what', 'something')]
   >>> result = [x[1] for x in pairs]
   >>> result
   ['else', 'sort', 'to', 'something']

最后一步的替代方案是:

   >>> result = []
   >>> for p in pairs: result.append(p[1])

如果你觉得这个更容易读懂，那么你可能更喜欢使用这个而不是前面的列表推导
。然而，对于长列表来说，它的速度几乎是原来的两倍。为什么？首先，
"append()" 操作必须重新分配内存，虽然它使用了一些技巧来避免每次都这样
做，但它仍然偶尔需要这样做，而且代价相当高。第二，表达式
"result.append" 需要额外的属性查找。第三，必须执行所有这些函数调用会降
低速度。


对象
====


什么是类？
----------

"类" 是通过执行类语句创建的特定对象类型。"类对象" 被当作模板来创建实例
对象，实例对象包含了特定于数据类型的数据（属性）和代码（方法）。

类可以基于一个或多个的其他类，称之为基类（ES），它继承基类的属性和方法
，这样就可以通过继承来连续地细化对象模型。例如：您可能有一个 "Mailbox"
类提供邮箱的基本访问方法.，它的子类 "MboxMailbox", "MaildirMailbox",
"OutlookMailbox" 用于处理各种特定邮箱格式。


什么是方法？
------------

"方法" 实际上就是类定义中的函数。对于某个对象 "x" 上的方法，通常称为
"x.name(arguments...)" 。

   class C:
       def meth(self, arg):
           return arg * 2 + self.attribute


什么是 self ？
--------------

Self 只是 "方法" 的第一个参数的常规名称。例如：对于某个类的某个实例
"x" ，其方法 "meth(self, a, b, c)" 实际上应该被称为 "x.meth(a, b, c)"
；对于被调用的方法会被称为 "meth(x, a, b, c)" 。

另请参阅 为什么必须在方法定义和调用中显式使用“self”？ 。


如何检查对象是否为给定类或其子类的一个实例？
--------------------------------------------

可使用内置函数 "isinstance(obj, cls)"。 你可以提供一个元组而不是单个类
来检查某个对象是否为任意多个类当中某一个类的实例，例如
"isinstance(obj, (class1, class2, ...))"，也可以检查某个对象是否为
Python 内置类型当中某一个类型的对象，例如 "isinstance(obj, str)" 或
"isinstance(obj, (int, float, complex))"。

请注意大多数程序不会经常对用户自定义类使用 "isinstance()"。 如果是你自
已开发的类，更正确的面向对象风格是在类中定义方法来封装特定的行为，而不
是检查对象的类并根据它属于什么类来做不同的事。 例如，如果你有一个执行
某些操作的函数:

   def search(obj):
       if isinstance(obj, Mailbox):
           ...  # code to search a mailbox
       elif isinstance(obj, Document):
           ...  # code to search a document
       elif ...

更好的方法是在所有类上定义一个 "search()" 方法，然后调用它：

   class Mailbox:
       def search(self):
           ...  # code to search a mailbox

   class Document:
       def search(self):
           ...  # code to search a document

   obj.search()


什么是委托？
------------

委托是一种面向对象的技巧（也称为设计模式）。 假设您有一个对象 "x" 并且
想要改变其中一个方法的行为。 您可以创建一个新类，它提供您感兴趣的方法
的新实现，并将所有其他方法委托给 "x" 的相应方法。

Python程序员可以轻松实现委托。 例如，以下类实现了一个类，该类的行为类
似于文件，但将所有写入的数据转换为大写：

   class UpperOut:

       def __init__(self, outfile):
           self._outfile = outfile

       def write(self, s):
           self._outfile.write(s.upper())

       def __getattr__(self, name):
           return getattr(self._outfile, name)

在这里 "UpperOut" 类重新定义了 "write()" 方法在调用下层的
"self._outfile.write()" 方法之前将参数字符串转换为大写形式。 所有其他
方法都被委托给下层的 "self._outfile" 对象。 委托是通过 "__getattr__"
方法来完成的；请参阅 语言参考 了解有关控制属性访问的更多信息。

请注意对于更一般的情况来说，委托可能包含更多细节问题。 当某些属性既需
要读取又需要设置时，类还必须定义 "__setattr__()" 方法，并且这样做必须
小心谨慎。 "__setattr__()" 的基本实现大致相当于以下代码:

   class X:
       ...
       def __setattr__(self, name, value):
           self.__dict__[name] = value
       ...

大多数 "__setattr__()" 实现必须修改 "self.__dict__" 来为自身保存局部状
态而又不至于造成无限递归。


如何从覆盖基类的派生类调用基类中定义的方法?
-------------------------------------------

使用内置的 "super()" 函数：

   class Derived(Base):
       def meth(self):
           super(Derived, self).meth()

对于 Python 3.0之前的版本，您可能正在使用经典类：对于诸如 "class
Derived(Base): ..." 之类的类定义，可以将在 "Base" (或 "Base" 中的一个
的基类）中定义的方法 "meth()" 调用为 "Base.meth(self, arguments...)"
。这里， "Base.meth" 是一个未绑定的方法，因此您需要提供 "self" 参数。


如何组织代码以便更改基类？
--------------------------

可以为基类定义别名，在类定义之前为其分配实际基类，并在整个类中使用别名
。然后更改分配给别名的值，就能实现上述要求。顺便提一下，如果你想动态决
定（例如，取决于资源的可用性）要使用哪个基类，这个技巧也很方便。例如：

   BaseAlias = <real base class>

   class Derived(BaseAlias):
       def meth(self):
           BaseAlias.meth(self)
           ...


如何创建静态类数据和静态类方法？
--------------------------------

Python支持静态数据和静态方法（在C ++或Java的意义上）。

对于静态数据，只需定义一个类属性。要为属性分配新值，就必须在赋值中显式
使用类名：

   class C:
       count = 0   # number of times C.__init__ called

       def __init__(self):
           C.count = C.count + 1

       def getcount(self):
           return C.count  # or return self.count

对于任意 "c" 来说只要 "isinstance(c, C)" 为真，则 "c.count" 同样也指向
"C.count"，除非被 "c" 自身，或者从 "c.__class__" 回到 "C" 的基类搜索路
径上的某个类所重载。

注意：在 C 的某个方法内部，像 "self.count = 42" 这样的赋值将在 "self"
自身的字典中新建一个名为 "count" 的不相关实例。 想要重新绑定类静态数据
名称就必须总是指明类名，无论是在方法内部还是外部:

   C.count = 314

静态方法是可行的：

   class C:
       @staticmethod
       def static(arg1, arg2, arg3):
           # No 'self' parameter!
           ...

然而，获得静态方法效果的更直接的方法是通过一个简单的模块级函数：

   def getcount():
       return C.count

如果您的代码是结构化的，以便为每个模块定义一个类（或紧密相关的类层次结
构），那么这就提供了所需的封装。


如何在Python中重载构造函数（或方法）？
--------------------------------------

这个答案实际上适用于所有方法，但问题通常首先出现在构造函数的上下文中。

在C ++中，你会这样写

   class C {
       C() { cout << "No arguments\n"; }
       C(int i) { cout << "Argument is " << i << "\n"; }
   }

在Python中，您必须编写一个构造函数，使用默认参数捕获所有情况。例如：

   class C:
       def __init__(self, i=None):
           if i is None:
               print("No arguments")
           else:
               print("Argument is", i)

这不完全等同，但在实践中足够接近。

你也可以尝试一个可变长度的参数列表，例如:

   def __init__(self, *args):
       ...

相同的方法适用于所有方法定义。


我尝试使用 __spam ，但是得到一个关于 _SomeClassName__spam 的错误信息。
----------------------------------------------------------------------

以双下划线打头的变量会被“更名”以提供一种定义类私有变量的简单而有效的方
式。 任何形式为 "__spam" 的标识符（至少前缀两个下划线，至多后缀一个下
划线）文本会被替换为 "_classname__spam"，其中 "classname" 为去除了全部
前缀下划线的当前类名称。

这并不能保证私密性：外部用户仍然可以访问 "_classname__spam" 属性，私有
变量值也在对象的 "__dict__" 中可见。 许多 Python 程序员从来都不使用这
种私有变量名称。


类定义了 __del__ 方法，但是删除对象时没有调用它。
-------------------------------------------------

这有几个可能的原因。

del 语句不一定调用 "__del__()" —— 它只是减少对象的引用计数，如果（引用
计数）达到零，才会调用 "__del__()"。

如果数据结构包含循环链接（例如，每个子级都有一个父级引用，每个父级都有
一个子级列表的树），则引用计数将永远不会返回零。尽管Python 偶尔会运行
一个算法来检测这样的循环，但在数据结构的引用计数清零后，垃圾收集器可能
需要一段时间来运行，因此 "__del__()" 方法可能会在不方便和随机的时间被
调用。这对于重现一个问题，是非常不方便的。更糟糕的是，对象 "__del__()"
的方法执行顺序是任意的。虽然可以运行 "gc.collect()" 来强制回收，但在一
些病态的情况下，对象永远不会被回收。

尽管有循环收集器，但在对象上定义一个显式的 "close()" 方法以便在用完之
后调用它仍然是一个好主意。 这样 "close()" 方法可以随即删除引用子对象的
属性。 不要直接调用 "__del__()" —— 应该由 "__del__()" 调用 "close()"，
并且 "close()" 能确保可以被同一对象多次地调用。

另一种避免循环引用的方法是使用 "weakref" 模块，该模块允许您指向对象而
不增加其引用计数。例如，树状数据结构应该对其父级和同级引用使用弱引用（
如果需要的话！）

最后，如果 "__del__()" 方法引发异常，会将警告消息打印到 "sys.stderr"
。


如何获取给定类的所有实例的列表？
--------------------------------

Python不跟踪类（或内置类型）的所有实例。您可以对类的构造函数进行编程，
以通过保留每个实例的弱引用列表来跟踪所有实例。


为什么 "id()" 的结果看起来不是唯一的？
--------------------------------------

"id()" 返回一个整数，该整数在对象的生命周期内保证是唯一的。因为在
CPython中，这是对象的内存地址，所以经常发生在从内存中删除对象之后，下
一个新创建的对象被分配在内存中的相同位置。这个例子说明了这一点：

>>> id(1000) 
13901272
>>> id(2000) 
13901272

这两个id属于之前创建的不同整数对象，并在执行 "id()" 调用后立即删除。要
确保要检查其id的对象仍处于活动状态，请创建对该对象的另一个引用：

>>> a = 1000; b = 2000
>>> id(a) 
13901272
>>> id(b) 
13891296


模块
====


如何创建 .pyc 文件？
--------------------

当一个模块首次被导入时（或自当前已编译文件创建后源文件被修改时），将会
在对应 ".py" 文件所在目录的 "__pycache__" 子目录下创建一个包含已编译代
码的 ".pyc" 文件。 该 ".pyc" 文件的文件名的开头部分将与对应 ".py" 文件
名相同，并以 ".pyc" 为后缀，中间部门则是基于创建它的特定 "python" 二进
制代码版本。 （详情参见 **PEP 3147**。）

无法创建 ".pyc" 文件的可能原因是包含源文件的目录存在权限问题，这意味着
"__pycache__" 子目录无法被创建。 举例来说，如果你以某一用户来开发程序
但以另一用户身份来运行程序时就可能发生问题，测试 Web 服务器就属于这种
情况。

除非设置了 "PYTHONDONTWRITEBYTECODE" 环境变量，否则当你导入模块并且
Python 具有创建 "__pycache__" 子目录并将已编译模块写入该子目录的能力（
权限、存储空间等等）时就会自动创建 .pyc 文件。

在最高层级运行的 Python 脚本不被视为导入，因此不会创建 ".pyc" 文件。
例如，如果你有一个最高层级模块文件 "foo.py"，它又导入了另一个模块
"xyz.py"，当你运行 "foo" 模块 (通过输入终端命令 "python foo.py")，则将
为 "xyz" 创建一个 ".pyc"，因为 "xyz" 是被导入的，但不会为 "foo" 创建
".pyc" 文件，因为 "foo.py" 不是被导入的。

如果你需要为 "foo" 创建 ".pyc" 文件 —— 即为不是被导入的模块创建 ".pyc"
文件 —— 你可以使用 "py_compile" 和 "compileall" 模块。

"py_compile" 模块能够手动编译任意模块。 一种做法是交互式地使用该模块中
的 "compile()" 函数:

   >>> import py_compile
   >>> py_compile.compile('foo.py')                 

这将会将  ".pyc" 文件写入与 "foo.py" 相同位置下的 "__pycache__" 子目录
（或者你也可以通过可选参数 "cfile" 来重载该行为）。

你还可以使用 "compileall" 模块自动编译一个目录或多个目录下的所有文件。
具体做法可以是在命令行提示符中运行 "compileall.py" 并提供包含要编译
Python 文件的目录路径:

   python -m compileall .


如何找到当前模块名称？
----------------------

模块可以通过查看预定义的全局变量 "__name__" 找到自己的模块名称。如果它
的值为 "'__main__'" ，程序将作为脚本运行。通常，通过导入使用的许多模块
也提供命令行界面或自检，并且只在检查 "__name__" 之后，才执行之后的代码
:

   def main():
       print('Running test...')
       ...

   if __name__ == '__main__':
       main()


如何让模块相互导入？
--------------------

假设您有以下模块：

foo.py:

   from bar import bar_var
   foo_var = 1

bar.py:

   from foo import foo_var
   bar_var = 2

问题是解释器将执行以下步骤：

* 首先导入foo

* 创建用于foo的空全局变量

* foo被编译并开始执行

* foo 导入 bar

* 创建了用于bar 的空全局变量

* bar被编译并开始执行

* bar导入foo（这是一个空操作（no-op ），因为已经有一个名为foo的模块）

* bar.foo_var = foo.foo_var

最后一步失败了，因为Python还没有解释foo，而foo的全局符号字典仍然是空的
。

当你使用 "import foo" ，然后尝试在全局代码中访问 "foo.foo_var" 时，会
发生同样的事情。

这个问题有（至少）三种可能的解决方法。

Guido van Rossum 建议避免使用 "from <module> import ..." ，并将所有代
码放在函数中。全局变量和类变量的初始化只能使用常量或内置函数。这意味着
导入模块中的所有内容都被引用为 "<module>.<name>" 。

Jim Roskind建议在每个模块中按以下顺序执行步骤：

* 导出（全局变量，函数和不需要导入基类的类）

* "导入" 声明

* 活动代码（包括从导入值初始化的全局变量）。

van Rossum不喜欢这种方法，因为导入出现在一个陌生的地方，但这种方法确实
有效。

Matthias Urlichs建议重构代码，以便首先不需要递归导入。

这些解决方案并不相互排斥。


__import__('x.y.z') 返回 <module 'x'>; 如何获取z?
-------------------------------------------------

考虑使用 "importlib" 中的函数 "import_module()" ：

   z = importlib.import_module('x.y.z')


当我编辑了导入过的模块并重新导入它时，这些变化没有显示出来。为什么会这样？
--------------------------------------------------------------------------

出于效率和一致性的原因，Python仅在第一次导入模块时读取模块文件。如果不
这么做，在一个由许多模块组成的程序中，每个模块都会导入相同的基本模块，
那么基本模块将被解析和重新解析多次。要强制重新读取已更改的模块，请执行
以下操作:

   import importlib
   import modname
   importlib.reload(modname)

警告：这种技术不是100％万无一失。特别是包含如下语句的模块

   from modname import some_objects

将继续使用旧版本的导入对象。如果模块包含类定义，则不会更新现有的类实例
以使用新的类定义。这可能导致以下矛盾行为:

   >>> import importlib
   >>> import cls
   >>> c = cls.C()                # Create an instance of C
   >>> importlib.reload(cls)
   <module 'cls' from 'cls.py'>
   >>> isinstance(c, cls.C)       # isinstance is false?!?
   False

如果打印出类对象的“标识”，问题的本质就会明确：

   >>> hex(id(c.__class__))
   '0x7352a0'
   >>> hex(id(cls.C))
   '0x4198d0'

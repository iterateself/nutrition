语句解释及变量编译
******************

这些函数在创建你自己的函数时帮助很大。更多说明以及实例可参考说明文档中
的 扩展和嵌入 Python 解释器 小节。

这些函数描述的前三个，"PyArg_ParseTuple()"，
"PyArg_ParseTupleAndKeywords()"，以及 "PyArg_Parse()"，它们都使用 *格
式化字符串* 来将函数期待的参数告知函数。这些函数都使用相同语法规则的格
式化字符串。


解析参数
========

一个格式化字符串包含 0 或者更多的格式单元。一个格式单元用来描述一个
Python 对象；它通常是一个字符或者由括号括起来的格式单元序列。除了少数
例外，一个非括号序列的格式单元通常对应这些函数的具有单一地址的参数。在
接下来的描述中，双引号内的表达式是格式单元；圆括号 () 内的是对应这个格
式单元的 Python 对象类型；方括号 [] 内的是传递的 C 变量(变量集)类型。


字符串和缓存区
--------------

这些格式允许将对象按照连续的内存块形式进行访问。你没必要提供返回的
unicode 字符或者字节区的原始数据存储。

一般的，当一个表达式设置一个指针指向一个缓冲区，这个缓冲区可以被相应的
Python 对象管理，并且这个缓冲区共享这个对象的生存周期。你不需要人为的
释放任何内存空间。除了这些 "es", "es#", "et" and "et#".

然而，当一个 "Py_buffer" 结构被赋值，其包含的缓冲区被锁住，所以调用者
在随后使用这个缓冲区，即使在 "Py_BEGIN_ALLOW_THREADS" 块中，可以避免可
变数据因为调整大小或者被销毁所带来的风险。因此，**你不得不调用**
"PyBuffer_Release()" 在你结束数据的处理时(或者在之前任何中断事件中)

除非另有说明，缓冲区是不会以空终止的。

某些格式需要只读的 *bytes-like object*，并设置指针而不是缓冲区结构。
他们通过检查对象的 "PyBufferProcs.bf_releasebuffer" 字段是否为 "NULL"
来发挥作用，该字段不允许为 "bytearray" 这样的可变对象。

注解:

  所有 "#" 表达式的变式("s#"，"y#"，等等)，长度参数的类型(整型或者
  "Py_ssize_t")在包含 "Python.h" 头文件之前由 "PY_SSIZE_T_CLEAN" 宏的
  定义控制。如果这个宏被定义，长度是一个 "Py_ssize_t" Python 元大小类
  型而不是一个 "int" 整型。在未来的 Python 版本中将会改变，只支持
  "Py_ssize_t" 而放弃支持 "int" 整型。最好一直定义 "PY_SSIZE_T_CLEAN"
  这个宏。

"s" ("str") [const char *]
   将一个 Unicode 对象转换成一个指向字符串的 C 指针。一个指针指向一个
   已经存在的字符串，这个字符串存储的是传如的字符指针变量。C 字符串是
   已空结束的。Python 字符串不能包含嵌入的无效的代码点；如果由，一个
   "ValueError" 异常会被引发。Unicode 对象被转化成 "'utf-8'" 编码的 C
   字符串。如果转换失败，一个 "UnicodeError" 异常被引发。

   注解:

     这个表达式不接受 *bytes-like objects*。如果你想接受文件系统路径并
     将它们转化成 C 字符串，建议使用 "O&" 表达式配合
     "PyUnicode_FSConverter()" 作为 *转化函数*。

   在 3.5 版更改: 以前，当 Python 字符串中遇到了嵌入的 null 代码点会引
   发 "TypeError" 。

"s*" ("str" or *bytes-like object*) [Py_buffer]
   这个表达式既接受 Unicode 对象也接受类字节类型对象。它为由调用者提供
   的 "Py_buffer" 结构赋值。这里结果的 C 字符串可能包含嵌入的 NUL 字节
   。Unicode 对象通过 "'utf-8'" 编码转化成 C 字符串。

"s#" ("str", 只读 *bytes-like object*) [const char *, int or
"Py_ssize_t"]
   像 "s*"，除了它不接受易变的对象。结果存储在两个 C 变量中，第一个是
   指向 C 字符串的指针，第二个是它的长度。字符串可能包含嵌入的 null 字
   节。Unicode 对象都被通过 "'utf-8'" 编码转化成 C 字符串。

"z" ("str" or "None") [const char *]
   与 "s" 类似，但 Python 对象也可能为 "None"，在这种情况下，C 指针设
   置为 "NULL"。

"z*" ("str", *bytes-like object* or "None") [Py_buffer]
   与 "s*" 类似，但 Python 对象也可能为 "None"，在这种情况下，
   "Py_buffer" 结构的 "buf" 成员设置为 "NULL"。

"z#" ("str", 只读 *bytes-like object* 或 "None") [const char *, int 或
"Py_ssize_t"]
   与 "s#" 类似，但 Python 对象也可能为 "None"，在这种情况下，C 指针设
   置为 "NULL"。

"y" (read-only *bytes-like object*) [const char *]
   这个表达式将一个类字节类型对象转化成一个指向字符串的 C 指针；它不接
   受 Unicode 对象。字节缓存区必须不包含嵌入的 null 字节；如果包含了
   null 字节，会引发一个 "ValueError" 异常。

   在 3.5 版更改: 以前，当字节缓冲区中遇到了嵌入的 null 字节会引发
   "TypeError" 。

"y*" (*bytes-like object*) [Py_buffer]
   "s*" 的变式，不接受 Unicode 对象，只接受类字节类型变量。**这是接受
   二进制数据的推荐方法。**

"y#" (只读 *bytes-like object*) [const char *, int 或 "Py_ssize_t"]
   "s#" 的变式，不接受 Unicode 对象，只接受类字节类型变量。

"S" ("bytes") [PyBytesObject *]
   要求 Python 对象是一个 "bytes" 类型对象，没有尝试任何的转换。如果不
   是一个字节类型对象会引发 "TypeError" 异常。C 变量也可能声明为
   "PyObject*" 类型。

"Y" ("bytearray") [PyByteArrayObject *]
   要求 Python 对象是一个 "bytearray" 类型对象，没有尝试任何的转换。如
   果不是一个 "bytearray" 类型对象会引发 "TypeError" 异常。C 变量也可
   能声明为 "PyObject*" 类型。

"u" ("str") [const Py_UNICODE *]
   将一个 Python Unicode 对象转化成指向一个以空终止的 Unicode 字符缓冲
   区的指针。你必须传入一个 "Py_UNICODE" 指针变量的地址，存储了一个指
   向已经存在的 Unicode 缓冲区的指针。请注意一个 "Py_UNICODE" 类型的字
   符宽度取决于编译选项(16 位或者 32 位)。Python 字符串必须不能包含嵌
   入的 null 代码点；如果有，引发一个 "ValueError" 异常。

   在 3.5 版更改: 以前，当 Python 字符串中遇到了嵌入的 null 代码点会引
   发 "TypeError" 。

   Deprecated since version 3.3, will be removed in version 4.0: 这是
   旧版样式 "Py_UNICODE" API; 请迁移至 "PyUnicode_AsWideCharString()".

"u#" ("str") [const Py_UNICODE *, int 或 "Py_ssize_t"]
   "u" 的变式，存储两个 C 变量，第一个指针指向一个 Unicode 数据缓存区
   ，第二个是它的长度。它允许 null 代码点。

   Deprecated since version 3.3, will be removed in version 4.0: 这是
   旧版样式 "Py_UNICODE" API; 请迁移至 "PyUnicode_AsWideCharString()".

"Z" ("str" 或 "None") [const Py_UNICODE *]
   与 "u" 类似，但 Python 对象也可能为 "None"，在这种情况下
   "Py_UNICODE" 指针设置为 "NULL"。

   Deprecated since version 3.3, will be removed in version 4.0: 这是
   旧版样式 "Py_UNICODE" API; 请迁移至 "PyUnicode_AsWideCharString()".

"Z#" ("str" 或 "None") [const Py_UNICODE *, int 或 "Py_ssize_t"]
   与 "u#" 类似，但 Python 对象也可能为 "None"，在这种情况下
   "Py_UNICODE" 指针设置为 "NULL"。

   Deprecated since version 3.3, will be removed in version 4.0: 这是
   旧版样式 "Py_UNICODE" API; 请迁移至 "PyUnicode_AsWideCharString()".

"U" ("str") [PyObject *]
   要求 Python 对象是一个 Unicode 对象，没有尝试任何的转换。如果不是一
   个 Unicode 对象会引发 "TypeError" 异常。C 变量也可能声明为
   "PyObject*" 类型。

"w*" (可读写 *bytes-like object*) [Py_buffer]
   这个表达式接受任何实现可读写缓存区接口的对象。它为调用者提供的
   "Py_buffer" 结构赋值。缓冲区可能存在嵌入的 null 字节。当缓冲区使用
   完后调用者需要调用 "PyBuffer_Release()"。

"es" ("str") [const char *encoding, char **buffer]
   "s" 的变式，它将编码后的 Unicode 字符存入字符缓冲区。它只处理没有嵌
   NUL 字节的已编码数据。

   此格式需要两个参数。 第一个仅用作输入，并且必须是 "const char*"，该
   名称将编码的名称指向 NUL 终止字符串或"NULL"，在这种情况下，使用
   "'utf-8'" 编码。如果 Python 不知道命名编码，则引发异常。 第二个参数
   必须为 "char**" 它引用的指针的值将设置为包含参数文本内容的缓冲区。
   文本将以第一个参数指定的编码进行编码。

   "PyArg_ParseTuple()" 会分配一个足够大小的缓冲区，将编码后的数据拷贝
   进这个缓冲区并且设置 **buffer* 引用这个新分配的内存空间。调用者有责
   任在使用后调用 "PyMem_Free()" 去释放已经分配的缓冲区。

"et" ("str", "bytes" or "bytearray") [const char *encoding, char
**buffer]
   和 "es" 相同，除了不用重编码传入的字符串对象。相反，它假设传入的参
   数是编码后的字符串类型。

"es#" ("str") [const char *encoding, char **buffer, int 或
"Py_ssize_t" *buffer_length]
   "s#" 的变式，它将已编码的 Unicode 字符存入字符缓冲区。不像 "es" 表
   达式，它允许传入的数据包含 NUL 字符。

   它需要三个参数。 第一个仅用作输入，并且必须为 "const char*"，该对象
   指向一个编码格式名称，形式为以 NUL 结束的字符串或 "NULL"，在后一种
   情况下将使用 "'utf-8'" 编码格式。 如果编码格式名称无法被 Python 识
   别则会引发异常。 第二个参数必须为 "char**"；它所引用的指针的值将被
   设为包含参数文本内容的缓冲区。 文本将以第一个参数所指定的编码格式进
   行编码。 第三个参数必须是指向一个整数的指针；所引用的整数将被设为输
   出缓冲区中的字节数。

   有两种操作方式：

   如果 **buffer* 指向 "NULL" 指针，则函数将分配所需大小的缓冲区，将编
   码的数据复制到此缓冲区，并设置 **buffer* 以引用新分配的存储。 呼叫
   者负责调用 "PyMem_Free()" 以在使用后释放分配的缓冲区。

   如果 **buffer* 指向非 "NULL" 指针（已分配的缓冲区），则
   "PyArg_ParseTuple()" 将使用此位置作为缓冲区，并将 **buffer_length*
   的初始值解释为缓冲区大小。 然后，它将将编码的数据复制到缓冲区，并终
   止它。 如果缓冲区不够大，将设置一个 "ValueError"。

   在这两个例子中，**buffer_length* 被设置为编码后结尾不为 NUL 的数据
   的长度。

"et#" ("str", "bytes" 或 "bytearray") [const char *encoding, char
**buffer, int 或 "Py_ssize_t" *buffer_length]
   和 "es#" 相同，除了不用重编码传入的字符串对象。相反，它假设传入的参
   数是编码后的字符串类型。


数字
----

"b" ("int") [unsigned char]
   将一个非负的 Python 整型转化成一个无符号的微整型，存储在一个 C
   "unsigned char" 类型中。

"B" ("int") [unsigned char]
   将一个 Python 整型转化成一个微整型并不检查溢出问题，存储在一个 C
   "unsigned char" 类型中。

"h" ("int") [short int]
   将一个 Python 整型转化成一个 C "short int" 短整型。

"H" ("int") [unsigned short int]
   将一个 Python 整型转化成一个 C "unsigned short int" 无符号短整型，
   并不检查溢出问题。

"i" ("int") [int]
   将一个 Python 整型转化成一个 C "int" 整型。

"I" ("int") [unsigned int]
   将一个 Python 整型转化成一个 C "unsigned int" 无符号整型，并不检查
   溢出问题。

"l" ("int") [long int]
   将一个 Python 整型转化成一个 C "long int" 长整型。

"k" ("int") [unsigned long]
   将一个Python整型转化成一个C "unsigned long int" 无符号长整型，并不
   检查溢出问题。

"L" ("int") [long long]
   将一个 Python 整型转化成一个 C "long long" 长长整型。

"K" ("int") [unsigned long long]
   将一个 Python 整型转化成一个 C "unsigned long long" 无符号长长整型
   ，并不检查溢出问题。

"n" ("int") [Py_ssize_t]
   将一个 Python 整型转化成一个 C "Py_ssize_t" Python 元大小类型。

"c" ("bytes" 或者 "bytearray" 长度为 1) [char]
   将一个 Python 字节类型，如一个长度为 1 的 "bytes" 或者 "bytearray"
   对象，转化成一个 C "char" 字符类型。

   在 3.3 版更改: 允许 "bytearray" 类型的对象。

"C" ("str" 长度为 1) [int]
   将一个 Python 字符，如一个长度为 1 的 "str" 字符串对象，转化成一个
   C "int" 整型类型。

"f" ("float") [float]
   将一个 Python 浮点数转化成一个 C "float" 浮点数。

"d" ("float") [double]
   将一个Python浮点数转化成一个C "double" 双精度浮点数。

"D" ("complex") [Py_complex]
   将一个 Python 复数类型转化成一个 C "Py_complex" Python 复数类型。


其他对象
--------

"O" (object) [PyObject *]
   将 Python 对象（不进行任何转换）存储在 C 对象指针中。 因此，C 程序
   接收已传递的实际对象。 对象的引用计数不会增加。 存储的指针不是
   "NULL"。

"O!" (object) [*typeobject*, PyObject *]
   将一个 Python 对象存入一个 C 指针。和 "O" 类似，但是需要两个 C 参数
   ：第一个是 Python 类型对象的地址，第二个是存储对象指针的 C 变量(
   "PyObject*" 变量)的地址。如果 Python 对象类型不对，会抛出
   "TypeError" 异常。

"O&" (object) [*converter*, *anything*]
   通过一个 *converter* 函数将一个 Python 对象转换成一个 C 变量。这需
   要两个参数：第一个是一个函数，第二个是一个 C 变量的地址(任意类型的)
   ，转化为 "void *" 类型。*converter* 函数像这样被调用：

      status = converter(object, address);

   *object*是待转化的Python对象并且 *address* 是传入 "PyArg_Parse*()"
   函数的 "void*" 类型参数。返回的 *status* 是1代表转换成功，0代表转换
   失败。当转换失败，*converter*函数会引发一个异常并且不会修改
   *address* 的内容。

   如果 *converter* 返回 "Py_CLEANUP_SUPPORTED"，则如果参数解析最终失
   败，它可能会再次调用该函数，从而使转换器有机会释放已分配的任何内存
   。在第二个调用中，*object* 参数将为 "NULL";因此，该参数将为 "NULL";
   因此，该参数将为 "NULL"，因此，该参数将为 "NULL``（如果值）为
   ``NULL" *address* 的值与原始呼叫中的值相同。

   在 3.1 版更改: "Py_CLEANUP_SUPPORTED" 被添加。

"p" ("bool") [int]
   测试传入的值是否为真(一个布尔判断)并且将结果转化为相对应的 C
   true/false 整型值。如果表达式为真置 "1"，假则置 "0"。它接受任何合法
   的 Python 值。参见 逻辑值检测 获取更多关于 Python 如何测试值为真的
   信息。

   3.3 新版功能.

"(items)" ("tuple") [*matching-items*]
   对象必须是 Python 序列，它的长度是 *items* 中格式单元的数量。C 参数
   必须对应 *items* 中每一个独立的格式单元。序列中的格式单元可能有嵌套
   。

传递 "long" 整型(整型的值超过了平台的 "LONG_MAX" 限制)是可能的，然而没
有进行适当的范围检测——当接收字段太小而接收不到值时，最重要的位被静默地
截断(实际上，C 语言会在语义继承的基础上强制类型转换——期望的值可能会发
生变化)。

格式化字符串中还有一些其他的字符具有特殊的涵义。这些可能并不嵌套在圆括
号中。它们是：

"|"
   表明在 Python 参数列表中剩下的参数都是可选的。C 变量对应的可选参数
   需要初始化为默认值——当一个可选参数没有指定时， "PyArg_ParseTuple()"
   不能访问相应的 C 变量(变量集)的内容。

"$"
   "PyArg_ParseTupleAndKeywords()" only：表明在 Python 参数列表中剩下
   的参数都是强制关键字参数。当前，所有强制关键字参数都必须也是可选参
   数，所以格式化字符串中  "|" 必须一直在 "$" 前面。

   3.3 新版功能.

":"
   格式单元的列表结束标志；冒号后的字符串被用来作为错误消息中的函数名
   ("PyArg_ParseTuple()" 函数引发的“关联值”异常)。

";"
   格式单元的列表结束标志；分号后的字符串被用来作为错误消息取代默认的
   错误消息。 ":" 和 ";" 相互排斥。

注意任何由调用者提供的 Python 对象引用是 *借来的* 引用；不要递减它们的
引用计数！

传递给这些函数的附加参数必须是由格式化字符串确定的变量的地址；这些都是
用来存储输入元组的值。有一些情况，如上面的格式单元列表中所描述的，这些
参数作为输入值使用；在这种情况下，它们应该匹配指定的相应的格式单元。

为了转换成功，*arg* 对象必须匹配格式并且格式必须用尽。成功的话，
"PyArg_Parse*()" 函数返回 true，反之它们返回 false 并且引发一个合适的
异常。当 "PyArg_Parse*()" 函数因为某一个格式单元转化失败而失败时，对应
的以及后续的格式单元地址内的变量都不会被使用。


API 函数
--------

int PyArg_ParseTuple(PyObject *args, const char *format, ...)

   解析一个函数的参数，表达式中的参数按参数位置顺序存入局部变量中。成
   功返回 true；失败返回 false 并且引发相应的异常。

int PyArg_VaParse(PyObject *args, const char *format, va_list vargs)

   和 "PyArg_ParseTuple()" 相同，然而它接受一个 va_list 类型的参数而不
   是可变数量的参数集。

int PyArg_ParseTupleAndKeywords(PyObject *args, PyObject *kw, const char *format, char *keywords[], ...)

   分析将位置参数和关键字参数同时转换为局部变量的函数的参数。
   *keywords* 参数是关键字参数名称的 "NULL" 终止数组。 空名称表示
   positional-only parameters。成功时返回 true;发生故障时，它将返回
   false 并引发相应的异常。

   在 3.6 版更改: 添加了 positional-only parameters 的支持。

int PyArg_VaParseTupleAndKeywords(PyObject *args, PyObject *kw, const char *format, char *keywords[], va_list vargs)

   和 "PyArg_ParseTupleAndKeywords()" 相同，然而它接受一个va_list类型
   的参数而不是可变数量的参数集。

int PyArg_ValidateKeywordArguments(PyObject *)

   确保字典中的关键字参数都是字符串。这个函数只被使用于
   "PyArg_ParseTupleAndKeywords()" 不被使用的情况下，后者已经不再做这
   样的检查。

   3.2 新版功能.

int PyArg_Parse(PyObject *args, const char *format, ...)

   函数被用来析构“旧类型”函数的参数列表——这些函数使用的 "METH_OLDARGS"
   参数解析方法已从 Python 3 中移除。这不被推荐用于新代码的参数解析，
   并且在标准解释器中的大多数代码已被修改，已不再用于该目的。它仍然方
   便于分解其他元组，然而可能因为这个目的被继续使用。

int PyArg_UnpackTuple(PyObject *args, const char *name, Py_ssize_t min, Py_ssize_t max, ...)

   一个不使用格式化字符串指定参数类型的简单形式的参数检索。使用这种方
   法来检索参数的函数应该在函数或者方法表中声明 "METH_VARARGS"。包含实
   际参数的元组应该以 *args* 形式被传入；它必须是一个实际的元组。元组
   的长度必须至少是 *min* 并且不超过 *max*； *min* 和 *max* 可能相同。
   额外的参数必须传递给函数，每一个参数必须是一个指向 "PyObject*" 类型
   变量的指针；它们将被赋值为 *args* 的值；它们将包含借来的引用。不在
   *args* 里面的可选参数不会被赋值；由调用者完成初始化。函数成功则返回
   true 并且如果 *args* 不是元组或者包含错误数量的元素则返回 false；如
   果失败了会引发一个异常。

   这是一个使用此函数的示例，取自 "_weakref" 帮助模块用来弱化引用的源
   代码：

      static PyObject *
      weakref_ref(PyObject *self, PyObject *args)
      {
          PyObject *object;
          PyObject *callback = NULL;
          PyObject *result = NULL;

          if (PyArg_UnpackTuple(args, "ref", 1, 2, &object, &callback)) {
              result = PyWeakref_NewRef(object, callback);
          }
          return result;
      }

   这个例子中调用 "PyArg_UnpackTuple()" 完全等价于调用
   "PyArg_ParseTuple()":

      PyArg_ParseTuple(args, "O|O:ref", &object, &callback)


创建变量
========

PyObject* Py_BuildValue(const char *format, ...)
    *Return value: New reference.*

   基于类似于 "PyArg_Parse*()" 函数系列和一系列值的格式字符串创建新值
   。 在出现错误时返回值或 "NULL";如果返回 "NULL"，将引发异常。

   "Py_BuildValue()" 并不一直创建一个元组。只有当它的格式化字符串包含
   两个或更多的格式单元才会创建一个元组。如果格式化字符串是空，它返回
   "None"；如果它包含一个格式单元，它返回由格式单元描述的的任一对象。
   用圆括号包裹格式化字符串可以强制它返回一个大小为 0 或者 1 的元组。

   当内存缓存区的数据以参数形式传递用来构建对象时，如 "s" 和 "s#" 格式
   单元，会拷贝需要的数据。调用者提供的缓冲区从来都不会被由
   "Py_BuildValue()" 创建的对象来引用。换句话说，如果你的代码调用
   "malloc()" 并且将分配的内存空间传递给 "Py_BuildValue()"，你的代码就
   有责任在 "Py_BuildValue()" 返回时调用 "free()" 。

   在下面的描述中，双引号的表达式使格式单元；圆括号 () 内的是格式单元
   将要返回的 Python 对象类型；方括号 [] 内的是传递的 C 变量(变量集)的
   类型。

   字符例如空格，制表符，冒号和逗号在格式化字符串中会被忽略(但是不包括
   格式单元，如 "s#")。这可以使很长的格式化字符串具有更好的可读性。

   "s" ("str" 或 "None") [const char *]
      使用 "'utf-8'" 编码将空终止的 C 字符串转换为 Python "str" 对象。
      如果 C 字符串指针为 "NULL"，则使用 "None"。

   "s#" ("str" 或 "None") [const char *, int 或 "Py_ssize_t"]
      使用 "'utf-8'" 编码将 C 字符串及其长度转换为 Python "str" 对象。
      如果 C 字符串指针为 "NULL"，则长度将被忽略，并返回 "None"。

   "y" ("bytes") [const char *]
      这将 C 字符串转换为 Python "bytes" 对象。 如果 C 字符串指针为
      "NULL"，则返回 "None"。

   "y#" ("bytes") [const char *, int 或 "Py_ssize_t"]
      这会将 C 字符串及其长度转换为一个 Python 对象。 如果该 C 字符串
      指针为 "NULL"，则返回 "None"。

   "z" ("str" or "None") [const char *]
      和 "s" 一样。

   "z#" ("str" 或 "None") [const char *, int 或 "Py_ssize_t"]
      和 "s#" 一样。

   "u" ("str") [const wchar_t *]
      将空终止的 "wchar_t" 的 Unicode （UTF-16 或 UCS-4） 数据缓冲区转
      换为 Python Unicode 对象。 如果 Unicode 缓冲区指针为 "NULL"，则
      返回 "None"。

   "u#" ("str") [const wchar_t *, int 或 "Py_ssize_t"]
      将 Unicode （UTF-16 或 UCS-4） 数据缓冲区及其长度转换为 Python
      Unicode 对象。  如果 Unicode 缓冲区指针为 "NULL"，则长度将被忽略
      ，并返回 "None"。

   "U" ("str" 或 "None") [const char *]
      和 "s" 一样。

   "U#" ("str" 或 "None") [const char *, int 或 "Py_ssize_t"]
      和 "s#" 一样。

   "i" ("int") [int]
      将一个 C "int" 整型转化成 Python 整型对象。

   "b" ("int") [char]
      将一个 C "char" 字符型转化成 Python 整型对象。

   "h" ("int") [short int]
      将一个 C "short int" 短整型转化成 Python 整型对象。

   "l" ("int") [long int]
      将一个 C "long int" 长整型转化成 Python 整型对象。

   "B" ("int") [unsigned char]
      将一个 C "unsigned char" 无符号字符型转化成 Python 整型对象。

   "H" ("int") [unsigned short int]
      将一个 C "unsigned long" 无符号短整型转化成 Python 整型对象。

   "I" ("int") [unsigned int]
      将一个 C "unsigned long" 无符号整型转化成 Python 整型对象。

   "k" ("int") [unsigned long]
      将一个 C "unsigned long" 无符号长整型转化成 Python 整型对象。

   "L" ("int") [long long]
      将一个 C "long long" 长长整形转化成 Python 整形对象。

   "K" ("int") [unsigned long long]
      将一个 C "unsigned long long" 无符号长长整型转化成 Python 整型对
      象。

   "n" ("int") [Py_ssize_t]
      将一个 C "Py_ssize_t" 类型转化为 Python 整型。

   "c" ("bytes" 长度为1 ) [char]
      将一个 C "int" 整型代表的字符转化为 Python "bytes" 长度为 1 的字
      节对象。

   "C" ("str" 长度为 1) [int]
      将一个 C "int" 整型代表的字符转化为 Python "str" 长度为 1 的字符
      串对象。

   "d" ("float") [double]
      将一个 C "double" 双精度浮点数转化为 Python 浮点数类型数字。

   "f" ("float") [float]
      将一个 C "float" 单精度浮点数转化为 Python 浮点数类型数字。

   "D" ("complex") [Py_complex *]
      将一个 C "Py_complex" 类型的结构转化为 Python 复数类型。

   "O" (object) [PyObject *]
      将 Python 对象传递不变（其引用计数除外，该计数由 1 递增）。 如果
      传入的对象是 "NULL" 指针，则假定这是由于生成参数的调用发现错误并
      设置异常而引起的。因此，"Py_BuildValue()" 将返回 "NULL"，但不会
      引发异常。 如果尚未引发异常，则设置 "SystemError"。

   "S" (object) [PyObject *]
      和 "O" 相同。

   "N" (object) [PyObject *]
      和 "O" 相同，然而它并不增加对象的引用计数。当通过调用参数列表中
      的对象构造器创建对象时很实用。

   "O&" (object) [*converter*, *anything*]
      通过 *converter* 函数将 *anything* 转换为 Python 对象。 该函数以
      *anything* （应与 "void *" 兼容）作为其参数，应返回 "new" Python
      对象，如果发生错误，则应返回 "NULL"。

   "(items)" ("tuple") [*matching-items*]
      将一个 C 变量序列转换成 Python 元组并保持相同的元素数量。

   "[items]" ("list") [*相关的元素*]
      将一个 C 变量序列转换成 Python 列表并保持相同的元素数量。

   "{items}" ("dict") [*相关的元素*]
      将一个C变量序列转换成 Python 字典。每一对连续的 C 变量对作为一个
      元素插入字典中，分别作为关键字和值。

   如果格式字符串中出现错误，则设置 "SystemError" 异常并返回 "NULL"。

PyObject* Py_VaBuildValue(const char *format, va_list vargs)
    *Return value: New reference.*

   和 "Py_BuildValue()" 相同，然而它接受一个 va_list 类型的参数而不是
   可变数量的参数集。

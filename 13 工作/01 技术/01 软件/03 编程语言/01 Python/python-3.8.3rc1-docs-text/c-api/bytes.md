字节对象
********

当期望带一个字节串形参但却带一个非字节串形参被调用时，这些函数会引发
"TypeError"。

PyBytesObject

   这种 "PyObject" 的子类型表示一个 Python 字节对象。

PyTypeObject PyBytes_Type

   "PyTypeObject" 的实例代表一个 Python 字节类型，在 Python 层面它与
   "bytes" 是相同的对象。

int PyBytes_Check(PyObject *o)

   如果对象 *o* 是字节对象或字节类型的子类型的实例，则返回 true。

int PyBytes_CheckExact(PyObject *o)

   如果对象 *o* 是字节对象，但不是字节类型子类型的实例，则返回 true。

PyObject* PyBytes_FromString(const char *v)
    *Return value: New reference.*

   Return a new bytes object with a copy of the string *v* as value on
   success, and "NULL" on failure.  The parameter *v* must not be
   "NULL"; it will not be checked.

PyObject* PyBytes_FromStringAndSize(const char *v, Py_ssize_t len)
    *Return value: New reference.*

   Return a new bytes object with a copy of the string *v* as value
   and length *len* on success, and "NULL" on failure.  If *v* is
   "NULL", the contents of the bytes object are uninitialized.

PyObject* PyBytes_FromFormat(const char *format, ...)
    *Return value: New reference.*

   Take a C "printf()"-style *format* string and a variable number of
   arguments, calculate the size of the resulting Python bytes object
   and return a bytes object with the values formatted into it.  The
   variable arguments must be C types and must correspond exactly to
   the format characters in the *format* string.  The following format
   characters are allowed:

   +---------------------+-----------------+----------------------------------+
   | 格式字符            | 类型            | 注释                             |
   |=====================|=================|==================================|
   | "%%"                | *不适用*        | 文字%字符。                      |
   +---------------------+-----------------+----------------------------------+
   | "%c"                | int             | 一个字节，被表示为一个 C 语言的  |
   |                     |                 | 整型                             |
   +---------------------+-----------------+----------------------------------+
   | "%d"                | int             | 相当于 "printf("%d")". [1]       |
   +---------------------+-----------------+----------------------------------+
   | "%u"                | 无符号整型      | 相当于 "printf("%u")". [1]       |
   +---------------------+-----------------+----------------------------------+
   | "%ld"               | 长整型          | 相当于 "printf("%ld")". [1]      |
   +---------------------+-----------------+----------------------------------+
   | "%lu"               | 无符号长整型    | 相当于 "printf("%lu")". [1]      |
   +---------------------+-----------------+----------------------------------+
   | "%zd"               | Py_ssize_t      | 相当于 "printf("%zd")". [1]      |
   +---------------------+-----------------+----------------------------------+
   | "%zu"               | size_t          | 相当于 "printf("%zu")". [1]      |
   +---------------------+-----------------+----------------------------------+
   | "%i"                | int             | 相当于 "printf("%i")". [1]       |
   +---------------------+-----------------+----------------------------------+
   | "%x"                | int             | 相当于 "printf("%x")". [1]       |
   +---------------------+-----------------+----------------------------------+
   | "%s"                | const char*     | A null-terminated C character    |
   |                     |                 | array.                           |
   +---------------------+-----------------+----------------------------------+
   | "%p"                | const void*     | The hex representation of a C    |
   |                     |                 | pointer. Mostly equivalent to    |
   |                     |                 | "printf("%p")" except that it is |
   |                     |                 | guaranteed to start with the     |
   |                     |                 | literal "0x" regardless of what  |
   |                     |                 | the platform's "printf" yields.  |
   +---------------------+-----------------+----------------------------------+

   无法识别的格式字符会导致将格式字符串的其余所有内容原样复制到结果对
   象，并丢弃所有多余的参数。

   [1] 对于整数说明符（d，u，ld，lu，zd，zu，i，x）：当给出精度时，0
       转换标志是有效的。

PyObject* PyBytes_FromFormatV(const char *format, va_list vargs)
    *Return value: New reference.*

   与 "PyBytes_FromFormat()" 完全相同，除了它需要两个参数。

PyObject* PyBytes_FromObject(PyObject *o)
    *Return value: New reference.*

   返回字节表示实现缓冲区协议的对象*o*。

Py_ssize_t PyBytes_Size(PyObject *o)

   返回字节对象*o*中字节的长度。

Py_ssize_t PyBytes_GET_SIZE(PyObject *o)

   Macro form of "PyBytes_Size()" but without error checking.

char* PyBytes_AsString(PyObject *o)

   Return a pointer to the contents of *o*.  The pointer refers to the
   internal buffer of *o*, which consists of "len(o) + 1" bytes.  The
   last byte in the buffer is always null, regardless of whether there
   are any other null bytes.  The data must not be modified in any
   way, unless the object was just created using
   "PyBytes_FromStringAndSize(NULL, size)". It must not be
   deallocated.  If *o* is not a bytes object at all,
   "PyBytes_AsString()" returns "NULL" and raises "TypeError".

char* PyBytes_AS_STRING(PyObject *string)

   Macro form of "PyBytes_AsString()" but without error checking.

int PyBytes_AsStringAndSize(PyObject *obj, char **buffer, Py_ssize_t *length)

   Return the null-terminated contents of the object *obj* through the
   output variables *buffer* and *length*.

   If *length* is "NULL", the bytes object may not contain embedded
   null bytes; if it does, the function returns "-1" and a
   "ValueError" is raised.

   The buffer refers to an internal buffer of *obj*, which includes an
   additional null byte at the end (not counted in *length*).  The
   data must not be modified in any way, unless the object was just
   created using "PyBytes_FromStringAndSize(NULL, size)".  It must not
   be deallocated.  If *obj* is not a bytes object at all,
   "PyBytes_AsStringAndSize()" returns "-1" and raises "TypeError".

   在 3.5 版更改: Previously, "TypeError" was raised when embedded
   null bytes were encountered in the bytes object.

void PyBytes_Concat(PyObject **bytes, PyObject *newpart)

   Create a new bytes object in **bytes* containing the contents of
   *newpart* appended to *bytes*; the caller will own the new
   reference.  The reference to the old value of *bytes* will be
   stolen.  If the new object cannot be created, the old reference to
   *bytes* will still be discarded and the value of **bytes* will be
   set to "NULL"; the appropriate exception will be set.

void PyBytes_ConcatAndDel(PyObject **bytes, PyObject *newpart)

   Create a new bytes object in **bytes* containing the contents of
   *newpart* appended to *bytes*.  This version decrements the
   reference count of *newpart*.

int _PyBytes_Resize(PyObject **bytes, Py_ssize_t newsize)

   A way to resize a bytes object even though it is "immutable". Only
   use this to build up a brand new bytes object; don't use this if
   the bytes may already be known in other parts of the code.  It is
   an error to call this function if the refcount on the input bytes
   object is not one. Pass the address of an existing bytes object as
   an lvalue (it may be written into), and the new size desired.  On
   success, **bytes* holds the resized bytes object and "0" is
   returned; the address in **bytes* may differ from its input value.
   If the reallocation fails, the original bytes object at **bytes* is
   deallocated, **bytes* is set to "NULL", "MemoryError" is set, and
   "-1" is returned.

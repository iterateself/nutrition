"subprocess" --- 子进程管理
***************************

**源代码:** Lib/subprocess.py

======================================================================

"subprocess" 模块允许你生成新的进程，连接它们的输入、输出、错误管道，
并且获取它们的返回码。此模块打算代替一些老旧的模块与功能：

   os.system
   os.spawn*

在下面的段落中，你可以找到关于 "subprocess" 模块如何代替这些模块和功能
的相关信息。

参见: **PEP 324** -- 提出 subprocess 模块的 PEP


使用 "subprocess" 模块
======================

推荐的调用子进程的方式是在任何它支持的用例中使用 "run()" 函数。对于更
进阶的用例，也可以使用底层的 "Popen" 接口。

"run()" 函数是在 Python 3.5 被添加的；如果你需要与旧版本保持兼容，查看
Older high-level API 段落。

subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None)

   运行被 *arg* 描述的指令。等待指令完成，然后返回一个
   "CompletedProcess" 实例。

   以上显示的参数仅仅是最简单的一些，下面 常用参数 描述（因此在缩写签
   名中使用仅关键字标示）。完整的函数头和 "Popen" 的构造函数一样，此函
   数接受的大多数参数都被传递给该接口。（*timeout*, *input*, *check*
   和 *capture_output* 除外）。

   如果 *capture_output* 设为 true，stdout 和 stderr 将会被捕获。在使
   用时，内置的 "Popen" 对象将自动用 "stdout=PIPE" 和 "stderr=PIPE" 创
   建。*stdout* 和 *stderr* 参数不应当与 *capture_output* 同时提供。如
   果你希望捕获并将两个流合并在一起，使用 "stdout=PIPE" 和
   "stderr=STDOUT" 来代替 *capture_output*。

   *timeout* 参数将被传递给 "Popen.communicate()"。如果发生超时，子进
   程将被杀死并等待。 "TimeoutExpired" 异常将在子进程中断后被抛出。

   *input* 参数将被传递给 "Popen.communicate()" 以及子进程的标准输入。
   如果使用此参数，它必须是一个字节序列。 如果指定了 *encoding* 或
   *errors* 或者将 *text* 设置为 "True"，那么也可以是一个字符串。 当使
   用此参数时，在创建内部 "Popen" 对象时将自动带上 "stdin=PIPE"，并且
   不能再手动指定 *stdin* 参数。

   如果 *check* 设为 True, 并且进程以非零状态码退出, 一个
   "CalledProcessError" 异常将被抛出. 这个异常的属性将设置为参数, 退出
   码, 以及标准输出和标准错误, 如果被捕获到.

   如果 *encoding* 或者 *error* 被指定, 或者 *text* 被设为 True, 标准
   输入, 标准输出和标准错误的文件对象将通过指定的 *encoding* 和
   *errors* 以文本模式打开, 否则以默认的 "io.TextIOWrapper" 打开.
   *universal_newline* 参数等同于 *text* 并且提供了向后兼容性. 默认情
   况下, 文件对象是以二进制模式打开的.

   如果 *env* 不是 "None", 它必须是一个字典, 为新的进程设置环境变量;
   它用于替换继承的当前进程的环境的默认行为. 它将直接被传递给 "Popen".

   例如:

      >>> subprocess.run(["ls", "-l"])  # doesn't capture output
      CompletedProcess(args=['ls', '-l'], returncode=0)

      >>> subprocess.run("exit 1", shell=True, check=True)
      Traceback (most recent call last):
        ...
      subprocess.CalledProcessError: Command 'exit 1' returned non-zero exit status 1

      >>> subprocess.run(["ls", "-l", "/dev/null"], capture_output=True)
      CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0,
      stdout=b'crw-rw-rw- 1 root root 1, 3 Jan 23 16:23 /dev/null\n', stderr=b'')

   3.5 新版功能.

   在 3.6 版更改: 添加了 *encoding* 和 *errors* 形参.

   在 3.7 版更改: 添加了 *text* 形参, 作为 *universal_newlines* 的一个
   更好理解的别名. 添加了 *capture_output* 形参.

class subprocess.CompletedProcess

   "run()" 的返回值, 代表一个进程已经结束.

   args

      被用作启动进程的参数. 可能是一个列表或字符串.

   returncode

      子进程的退出状态码. 通常来说, 一个为 0 的退出码表示进程运行正常.

      一个负值 "-N" 表示子进程被信号 "N" 中断 (仅 POSIX).

   stdout

      从子进程捕获到的标准输出. 一个字节序列, 或一个字符串, 如果
      "run()" 是设置了 *encoding*, *errors* 或者 "text=True" 来运行的.
      如果未有捕获, 则为 "None".

      如果你通过 "stderr=subprocess.STDOUT" 运行, 标准输入和标准错误将
      被组合在一起, 并且 "stderr`" 将为 "None".

   stderr

      捕获到的子进程的标准错误. 一个字节序列, 或者一个字符串, 如果
      "run()" 是设置了参数 *encoding*, *errors* 或者 "text=True" 运行
      的. 如果未有捕获, 则为 "None".

   check_returncode()

      如果 "returncode" 非零, 抛出 "CalledProcessError".

   3.5 新版功能.

subprocess.DEVNULL

   可被 "Popen" 的 *stdin*, *stdout* 或者 *stderr* 参数使用的特殊值,
   表示使用特殊文件 "os.devnull".

   3.3 新版功能.

subprocess.PIPE

   可被 "Popen" 的 *stdin*, *stdout* 或者 *stderr* 参数使用的特殊值,
   表示打开标准流的管道. 常用于 "Popen.communicate()".

subprocess.STDOUT

   可被 "Popen" 的 *stdin* ， *stdout* 或者 *stderr* 参数使用的特殊值
   ， 表示标准错误与标准输出使用同一句柄。

exception subprocess.SubprocessError

   此模块的其他异常的基类。

   3.3 新版功能.

exception subprocess.TimeoutExpired

   "SubprocessError" 的子类，等待子进程的过程中发生超时时被抛出。

   cmd

      用于创建子进程的指令。

   timeout

      超时秒数。

   output

      子进程的输出， 如果被 "run()" 或 "check_output()" 捕获。否则为
      "None"。

   stdout

      对 output 的别名，对应的有 "stderr"。

   stderr

      子进程的标准错误输出，如果被 "run()" 捕获。 否则为 "None"。

   3.3 新版功能.

   在 3.5 版更改: 添加了 *stdout* 和 *stderr* 属性。

exception subprocess.CalledProcessError

   "SubprocessError" 的子类，当一个被 "check_call()" 或
   "check_output()" 函数运行的子进程返回了非零退出码时被抛出。

   returncode

      子进程的退出状态。如果程序由一个信号终止，这将会被设为一个负的信
      号码。

   cmd

      用于创建子进程的指令。

   output

      子进程的输出， 如果被 "run()" 或 "check_output()" 捕获。否则为
      "None"。

   stdout

      对 output 的别名，对应的有 "stderr"。

   stderr

      子进程的标准错误输出，如果被 "run()" 捕获。 否则为 "None"。

   在 3.5 版更改: 添加了 *stdout* 和 *stderr* 属性。


常用参数
--------

为了支持丰富的使用案例， "Popen" 的构造函数（以及方便的函数）接受大量
可选的参数。对于大多数典型的用例，许多参数可以被安全地留以它们的默认值
。通常需要的参数有：

   *args* 被所有调用需要，应当为一个字符串，或者一个程序参数序列。提供
   一个参数序列通常更好，它可以更小心地使用参数中的转义字符以及引用（
   例如允许文件名中的空格）。如果传递一个简单的字符串，则 *shell* 参数
   必须为 "True" （见下文）或者该字符串中将被运行的程序名必须用简单的
   命名而不指定任何参数。

   *stdin*， *stdout* 和 *stderr* 分别指定了执行的程序的标准输入、输出
   和标准错误文件句柄。合法的值有 "PIPE" 、 "DEVNULL" 、 一个现存的文
   件描述符（一个正整数）、一个现存的文件对象以及 "None"。 "PIPE" 表示
   应该新建一个对子进程的管道。 "DEVNULL" 表示使用特殊的文件
   "os.devnull"。当使用默认设置 "None" 时，将不会进行重定向，子进程的
   文件流将继承自父进程。另外， *stderr* 可能为 "STDOUT"，表示来自于子
   进程的标准错误数据应该被 *stdout* 相同的句柄捕获。

   如果 *encoding* 或 *errors* 被指定，或者 *text* （也名为
   *universal_newlines*）为真，则文件对象 *stdin* 、 *stdout* 与
   *stderr* 将会使用在此次调用中指定的 *encoding* 和 *errors* 以文本模
   式打开或者为默认的 "io.TextIOWrapper"。

   当构造函数的 *newline* 参数为 "None" 时。对于 *stdin*， 输入的换行
   符 "'\n'" 将被转换为默认的换行符 "os.linesep"。对于 *stdout* 和
   *stderr*， 所有输出的换行符都被转换为 "'\n'"。更多信息，查看
   "io.TextIOWrapper" 类的文档。

   如果文本模式未被使用， *stdin*， *stdout* 和 *stderr* 将会以二进制
   流模式打开。没有编码与换行符转换发生。

   3.6 新版功能: 添加了 *encoding* 和 *errors* 形参。

   3.7 新版功能: 添加了 *text* 形参作为 *universal_newlines* 的别名。

   注解:

     文件对象 "Popen.stdin" 、 "Popen.stdout" 和 "Popen.stderr" 的换行
     符属性不会被 "Popen.communicate()" 方法更新。

   如果 *shell* 设为 "True",，则使用 shell 执行指定的指令。如果您主要
   使用 Python 增强的控制流（它比大多数系统 shell 提供的强大），并且仍
   然希望方便地使用其他 shell 功能，如 shell 管道、文件通配符、环境变
   量展开以及 "~" 展开到用户家目录，这将非常有用。但是，注意 Python 自
   己也实现了许多类似 shell 的特性（例如 "glob", "fnmatch",
   "os.walk()", "os.path.expandvars()", "os.path.expanduser()" 和
   "shutil"）。

   在 3.3 版更改: 当 *universal_newline* 被设为 "True"，则类使用
   "locale.getpreferredencoding(False)" 编码来代替
   "locale.getpreferredencoding()"。关于它们的区别的更多信息，见
   "io.TextIOWrapper"。

   注解:

     在使用 "shell=True" 之前， 请阅读 Security Considerations 段落。

这些选项以及所有其他选项在 "Popen" 构造函数文档中有更详细的描述。


Popen 构造函数
--------------

此模块的底层的进程创建与管理由 "Popen" 类处理。它提供了很大的灵活性，
因此开发者能够处理未被便利函数覆盖的不常见用例。

class subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=True, shell=False, cwd=None, env=None, universal_newlines=None, startupinfo=None, creationflags=0, restore_signals=True, start_new_session=False, pass_fds=(), *, encoding=None, errors=None, text=None)

   在一个新的进程中执行子程序。在 POSIX，此类使用类似于 "os.execvp()"
   的行为来执行子程序。在 Windows，此类使用了 Windows
   "CreateProcess()" 函数。 "Popen" 的参数如下：

   *args* should be a sequence of program arguments or else a single
   string or *path-like object*. By default, the program to execute is
   the first item in *args* if *args* is a sequence.  If *args* is a
   string, the interpretation is platform-dependent and described
   below.  See the *shell* and *executable* arguments for additional
   differences from the default behavior.  Unless otherwise stated, it
   is recommended to pass *args* as a sequence.

   An example of passing some arguments to an external program as a
   sequence is:

      Popen(["/usr/bin/git", "commit", "-m", "Fixes a bug."])

   在 POSIX，如果 *args* 是一个字符串，此字符串被作为将被执行的程序的
   命名或路径解释。但是，只有在不传递任何参数给程序的情况下才能这么做
   。

   注解:

     It may not be obvious how to break a shell command into a
     sequence of arguments, especially in complex cases.
     "shlex.split()" can illustrate how to determine the correct
     tokenization for *args*:

        >>> import shlex, subprocess
        >>> command_line = input()
        /bin/vikings -input eggs.txt -output "spam spam.txt" -cmd "echo '$MONEY'"
        >>> args = shlex.split(command_line)
        >>> print(args)
        ['/bin/vikings', '-input', 'eggs.txt', '-output', 'spam spam.txt', '-cmd', "echo '$MONEY'"]
        >>> p = subprocess.Popen(args) # Success!

     特别注意，由 shell 中的空格分隔的选项（例如 *-input*）和参数（例
     如 *eggs.txt* ）位于分开的列表元素中，而在需要时使用引号或反斜杠
     转义的参数在 shell （例如包含空格的文件名或上面显示的 *echo* 命令
     ）是单独的列表元素。

   在 Windows，如果 *args* 是一个序列，他将通过一个在 Converting an
   argument sequence to a string on Windows 描述的方式被转换为一个字符
   串。这是因为底层的 "CreateProcess()" 只处理字符串。

   在 3.6 版更改: *args* parameter accepts a *path-like object* if
   *shell* is "False" and a sequence containing path-like objects on
   POSIX.

   在 3.8 版更改: *args* parameter accepts a *path-like object* if
   *shell* is "False" and a sequence containing bytes and path-like
   objects on Windows.

   参数 *shell* （默认为 "False"）指定是否使用 shell 执行程序。如果
   *shell* 为 "True"，更推荐将 *args* 作为字符串传递而非序列。

   在 POSIX，当 "shell=True"， shell 默认为 "/bin/sh"。如果 *args* 是
   一个字符串，此字符串指定将通过 shell 执行的命令。这意味着字符串的格
   式必须和在命令提示符中所输入的完全相同。这包括，例如，引号和反斜杠
   转义包含空格的文件名。如果 *args* 是一个序列，第一项指定了命令，另
   外的项目将作为传递给 shell （而非命令） 的参数对待。也就是说，
   "Popen" 等同于:

      Popen(['/bin/sh', '-c', args[0], args[1], ...])

   在 Windows，使用 "shell=True"，环境变量 "COMSPEC" 指定了默认 shell
   。在 Windows 你唯一需要指定 "shell=True" 的情况是你想要执行内置在
   shell 中的命令（例如 **dir** 或者 **copy**）。在运行一个批处理文件
   或者基于控制台的可执行文件时，不需要 "shell=True"。

   注解:

     在使用 "shell=True" 之前， 请阅读 Security Considerations 段落。

   *bufsize* 将在 "open()" 函数创建了 stdin/stdout/stderr 管道文件对象
   时作为对应的参数供应:

   * "0" 表示不使用缓冲区 （读取与写入是一个系统调用并且可以返回短内容
     ）

   * "1" 表示行缓冲（只有 "universal_newlines=True" 时才有用，例如，在
     文本模式中）

   * 任何其他正值表示使用一个约为对应大小的缓冲区

   * 负的 *bufsize* （默认）表示使用系统默认的 io.DEFAULT_BUFFER_SIZE
     。

   在 3.3.1 版更改: *bufsize* 现在默认为 -1 来启用缓冲，以符合大多数代
   码所期望的行为。在 Python 3.2.4 和 3.3.1 之前的版本中，它错误地将默
   认值设为了为 "0"，这是无缓冲的并且允许短读取。这是无意的，并且与大
   多数代码所期望的 Python 2 的行为不一致。

   *executable* 参数指定一个要执行的替换程序。这很少需要。当
   "shell=True"， *executable* 替换 *args* 指定运行的程序。但是，原始
   的 *args* 仍然被传递给程序。大多数程序将被 *args* 指定的程序作为命
   令名对待，这可以与实际运行的程序不同。在 POSIX， *args* 名作为实际
   调用程序中可执行文件的显示名称，例如 **ps**。如果 "shell=True"，在
   POSIX， *executable* 参数指定用于替换默认 shell "/bin/sh" 的 shell
   。

   在 3.6 版更改: *executable* parameter accepts a *path-like object*
   on POSIX.

   在 3.8 版更改: *executable* parameter accepts a bytes and *path-
   like object* on Windows.

   *stdin*, *stdout* 和 *stderr* 分别指定被运行的程序的标准输入、输出
   和标准错误的文件句柄。合法的值有 "PIPE" ， "DEVNULL" ， 一个存在的
   文件描述符（一个正整数），一个存在的 *文件对象* 以及 "None"。
   "PIPE" 表示应创建一个新的对子进程的管道。 "DEVNULL" 表示使用特殊的
   "os.devnull" 文件。使用默认的 "None"，则不进行成定向；子进程的文件
   流将继承自父进程。另外， *stderr* 可设为 "STDOUT"，表示应用程序的标
   准错误数据应和标准输出一同捕获。

   如果 *preexec_fn* 被设为一个可调用对象，此对象将在子进程刚创建时被
   调用。（仅 POSIX）

   警告:

     *preexec_fn* 形参在应用程序中存在多线程时是不安全的。子进程在调用
     前可能死锁。如果你必须使用它，保持警惕！最小化你调用的库的数量。

   注解:

     如果你需要修改子进程环境，使用 *env* 形参而非在 *preexec_fn* 中进
     行。 *start_new_session* 形参可以代替之前常用的 *preexec_fn* 来在
     子进程中调用 os.setsid()。

   在 3.8 版更改: The *preexec_fn* parameter is no longer supported in
   subinterpreters. The use of the parameter in a subinterpreter
   raises "RuntimeError". The new restriction may affect applications
   that are deployed in mod_wsgi, uWSGI, and other embedded
   environments.

   如果 *close_fds* 为真，所有文件描述符除了 "0", "1", "2" 之外都会在
   子进程执行前关闭。而当 *close_fds* 为假时，文件描述符遵守它们继承的
   标志，如 文件描述符的继承 所述。

   在 Windows，如果 *close_fds* 为真， 则子进程不会继承任何句柄，除非
   在 "STARTUPINFO.IpAttributeList" 的 "handle_list" 的键中显式传递，
   或者通过标准句柄重定向传递。

   在 3.2 版更改: *close_fds* 的默认值已经从 "False" 修改为上述值。

   在 3.7 版更改: 在 Windows，当重定向标准句柄时 *close_fds* 的默认值
   从 "False" 变为 "True"。现在重定向标准句柄时有可能设置 *close_fds*
   为 "True"。（标准句柄指三个 stdio 的句柄）

   *pass_fds* 是一个可选的在父子进程间保持打开的文件描述符序列。提供任
   何 *pass_fds* 将强制 *close_fds* 为 "True"。（仅 POSIX）

   在 3.2 版更改: 加入了 *pass_fds* 形参。

   If *cwd* is not "None", the function changes the working directory
   to *cwd* before executing the child.  *cwd* can be a string, bytes
   or *path-like* object.  In particular, the function looks for
   *executable* (or for the first item in *args*) relative to *cwd* if
   the executable path is a relative path.

   在 3.6 版更改: *cwd* parameter accepts a *path-like object* on
   POSIX.

   在 3.7 版更改: *cwd* parameter accepts a *path-like object* on
   Windows.

   在 3.8 版更改: *cwd* parameter accepts a bytes object on Windows.

    如果 *restore_signals* 为 true（默认值），则 Python 设置为 SIG_IGN
   的所有信号将在 exec 之前的子进程中恢复为 SIG_DFL。目前，这包括
   SIGPIPE ，SIGXFZ 和 SIGXFSZ 信号。 （仅 POSIX）

   在 3.2 版更改: *restore_signals* 被加入。

   如果 *start_new_session* 为 true，则 setsid() 系统调用将在子进程执
   行之前被执行。（仅 POSIX）

   在 3.2 版更改: *start_new_session* 被添加。

   如果 *env* 不为 "None"，则必须为一个为新进程定义了环境变量的字典；
   这些用于替换继承的当前进程环境的默认行为。

   注解:

     如果指定， *env* 必须提供所有被子进程需求的变量。在 Windows，为了
     运行一个 side-by-side assembly ，指定的 *env* **必须** 包含一个有
     效的 "SystemRoot"。

   如果 *encoding* 或 *errors* 被指定，或者 *text* 为 true，则文件对象
   *stdin*, *stdout* 和 *stderr* 将会以指定的编码和 *errors* 以文本模
   式打开，如同 常用参数 所述。 *universal_newlines* 参数等同于 *text*
   并且提供向后兼容性。默认情况下，文件对象都以二进制模式打开。

   3.6 新版功能: *encoding* 和 *errors* 被添加。

   3.7 新版功能: *text* 作为 *universal_newlines* 的一个更具可读性的别
   名被添加。

   如果给出， *startupinfo* 将是一个将被传递给底层的 "CreateProcess"
   函数的 "STARTUPINFO" 对象。 *creationflags*，如果给出，可以是一个或
   多个以下标志之一：

      * "CREATE_NEW_CONSOLE"

      * "CREATE_NEW_PROCESS_GROUP"

      * "ABOVE_NORMAL_PRIORITY_CLASS"

      * "BELOW_NORMAL_PRIORITY_CLASS"

      * "HIGH_PRIORITY_CLASS"

      * "IDLE_PRIORITY_CLASS"

      * "NORMAL_PRIORITY_CLASS"

      * "REALTIME_PRIORITY_CLASS"

      * "CREATE_NO_WINDOW"

      * "DETACHED_PROCESS"

      * "CREATE_DEFAULT_ERROR_MODE"

      * "CREATE_BREAKAWAY_FROM_JOB"

   Popen 对象支持通过 "with" 语句作为上下文管理器，在退出时关闭文件描
   述符并等待进程:

      with Popen(["ifconfig"], stdout=PIPE) as proc:
          log.write(proc.stdout.read())

   Popen and the other functions in this module that use it raise an
   auditing event "subprocess.Popen" with arguments "executable",
   "args", "cwd", and "env". The value for "args" may be a single
   string or a list of strings, depending on platform.

   在 3.2 版更改: 添加了上下文管理器支持。

   在 3.6 版更改: 现在，如果 Popen 析构时子进程仍然在运行，则析构器会
   发送一个 "ResourceWarning" 警告。

   在 3.8 版更改: Popen can use "os.posix_spawn()" in some cases for
   better performance. On Windows Subsystem for Linux and QEMU User
   Emulation, Popen constructor using "os.posix_spawn()" no longer
   raise an exception on errors like missing program, but the child
   process fails with a non-zero "returncode".


异常
----

在子进程中抛出的异常，在新的进程开始执行前，将会被再次在父进程中抛出。

最常见的被抛出异常是 "OSError"。例如，当尝试执行一个不存在的文件时就会
发生。应用程序需要为 "OSError" 异常做好准备。

如果 "Popen" 调用时有无效的参数，则一个 "ValueError" 将被抛出。

"check_all()" 与 "check_output()" 在调用的进程返回非零退出码时将抛出
"CalledProcessError"。

所有接受 *timeout* 形参的函数与方法，例如 "call()" 和
"Popen.communicate()" 将会在进程退出前超时到期时抛出 "TimeoutExpired"
。

此模块中定义的异常都继承自 "SubprocessError"。

   3.3 新版功能: 基类 "SubprocessError" 被添加。


安全考量
========

不像一些其他的 popen 功能，此实现绝不会隐式调用一个系统 shell。这意味
着任何字符，包括 shell 元字符，可以安全地被传递给子进程。如果 shell 被
明确地调用，通过 "shell=True" 设置，则确保所有空白字符和元字符被恰当地
包裹在引号内以避免 shell 注入 漏洞就由应用程序负责了。

当使用 "shell=True"， "shlex.quote()" 函数可以作为在将被用于构造 shell
指令的字符串中转义空白字符以及 shell 元字符的方案。


Popen 对象
==========

"Popen" 类的实例拥有以下方法：

Popen.poll()

   检查子进程是否已被终止。设置并返回 "returncode" 属性。否则返回
   "None"。

Popen.wait(timeout=None)

   等待子进程被终止。设置并返回 "returncode" 属性。

   如果进程在 *timeout* 秒后未中断，抛出一个 "TimeoutExpired" 异常，可
   以安全地捕获此异常并重新等待。

   注解:

     当 "stdout=PIPE" 或者 "stderr=PIPE" 并且子进程产生了足以阻塞 OS
     管道缓冲区接收更多数据的输出到管道时，将会发生死锁。当使用管道时
     用 "Popen.communicate()" 来规避它。

   注解:

     此函数使用了一个 busy loop （非阻塞调用以及短睡眠） 实现。使用
     "asyncio" 模块进行异步等待： 参阅
     "asyncio.create_subprocess_exec"。

   在 3.3 版更改: *timeout* 被添加

Popen.communicate(input=None, timeout=None)

   与进程交互：向 stdin 传输数据。从 stdout 和 stderr 读取数据，直到文
   件结束符。等待进程终止。可选的 *input* 参数应当未被传输给子进程的数
   据，如果没有数据应被传输给子进程则为 "None"。如果流以文本模式打开，
   *input* 必须为字符串。否则，它必须为字节。

   "communicate()" 返回一个 "(stdout_data, stderr_data)" 元组。如果文
   件以文本模式打开则为字符串；否则字节。

   注意如果你想要向进程的 stdin 传输数据，你需要通过 "stdin=PIPE" 创建
   此 Popen 对象。类似的，要从结果元组获取任何非 "None" 值，你同样需要
   设置 "stdout=PIPE" 或者 "stderr=PIPE"。

   如果进程在 *timeout* 秒后未终止，一个 "TimeoutExpired" 异常将被抛出
   。捕获此异常并重新等待将不会丢失任何输出。

   如果超时到期，子进程不会被杀死，所以为了正确清理一个行为良好的应用
   程序应该杀死子进程并完成通讯。

      proc = subprocess.Popen(...)
      try:
          outs, errs = proc.communicate(timeout=15)
      except TimeoutExpired:
          proc.kill()
          outs, errs = proc.communicate()

   注解:

     内存里数据读取是缓冲的，所以如果数据尺寸过大或无限，不要使用此方
     法。

   在 3.3 版更改: *timeout* 被添加

Popen.send_signal(signal)

   将信号 *signal* 发送给子进程。

   注解:

     在 Windows， SIGTERM 是一个 "terminate()" 的别名。 CTRL_C_EVENT
     和 CTRL_BREAK_EVENT 可以被发送给以包含 "CREATE_NEW_PROCESS" 的
     *creationflags* 形参启动的进程。

Popen.terminate()

   停止子进程。在 Posix 操作系统上，此方法发送 SIGTERM。在 Windows，调
   用 Win32 API 函数 "TerminateProcess()"  来停止子进程。

Popen.kill()

   杀死子进程。在 Posix 操作系统上，此函数给子进程发送 SIGKILL 信号。
   在 Windows 上， "kill()" 是 "terminate()" 的别名。

以下属性也是可用的：

Popen.args

   *args* 参数传递给 "Popen" -- 一个程序参数的序列或者一个简单字符串。

   3.3 新版功能.

Popen.stdin

   如果 *stdin* 参数为 "PIPE"，此属性是一个类似 "open()" 返回的可写的
   流对象。如果 *encoding* 或 *errors* 参数被指定或者
   *universal_newlines* 参数为 "True"，则此流是一个文本流，否则是字节
   流。如果 *stdin* 参数非 "PIPE"， 此属性为 "None"。

Popen.stdout

   如果 *stdout* 参数是 "PIPE"，此属性是一个类似 "open()" 返回的可读流
   。从流中读取子进程提供的输出。如果 *encoding* 或 *errors* 参数被指
   定或者 *universal_newlines* 参数为 "True"，此流为文本流，否则为字节
   流。如果 *stdout* 参数非 "PIPE"，此属性为 "None"。

Popen.stderr

   如果 *stderr* 参数是 "PIPE"，此属性是一个类似 "open()" 返回的可读流
   。从流中读取子进程提供的输出。如果 *encoding* 或 *errors* 参数被指
   定或者 *universal_newlines* 参数为 "True"，此流为文本流，否则为字节
   流。如果 *stderr* 参数非 "PIPE"，此属性为 "None"。

警告:

  使用 "communicate()" 而非 ".stdin.write"， ".stdout.read" 或者
  ".stderr.read" 来避免由于任意其他 OS 管道缓冲区被子进程填满阻塞而导
  致的死锁。

Popen.pid

   子进程的进程号。

   注意如果你设置了 *shell* 参数为 "True"，则这是生成的子 shell 的进程
   号。

Popen.returncode

   此进程的退出码，由 "poll()" 和 "wait()" 设置（以及直接由
   "communicate()" 设置）。一个 "None" 值 表示此进程仍未结束。

   一个负值 "-N" 表示子进程被信号 "N" 中断 (仅 POSIX).


Windows Popen 助手
==================

"STARTUPINFO" 类和以下常数仅在 Windows 有效。

class subprocess.STARTUPINFO(*, dwFlags=0, hStdInput=None, hStdOutput=None, hStdError=None, wShowWindow=0, lpAttributeList=None)

   在 "Popen" 创建时部分支持 Windows 的 STARTUPINFO 结构。接下来的属性
   仅能通过关键词参数设置。

   在 3.7 版更改: 仅关键词参数支持被加入。

   dwFlags

      一个位域，用于确定属性 "STARTUPINFO" 是否在进程创建窗口时使用。

         si = subprocess.STARTUPINFO()
         si.dwFlags = subprocess.STARTF_USESTDHANDLES | subprocess.STARTF_USESHOWWINDOW

   hStdInput

      如果 "dwFlags" 被指定为 "STARTF_USESTDHANDLES"，此值是进程的标准
      输入句柄，如果 "STARTF_USESTDHANDLES" 未指定，则默认的标准输入是
      键盘缓冲区。

   hStdOutput

      如果 "dwFlags" 指定为 "STARTF_USESTDHANDLES"，此属性是进程的标准
      输出句柄。除此之外，此此属性将被忽略并且默认标准输出是控制台窗口
      缓冲区。

   hStdError

      如果 "dwFlags" 被指定为 "STARTF_USESTDHANDLES"，则此属性是进程的
      标准错误句柄。除此之外，此属性将被忽略并且默认标准错误为控制台窗
      口的缓冲区。

   wShowWindow

      如果 "dwFlags" 指定了 "STARTF_USESHOWWINDOW"，此属性可为能被指定
      为 函数 ShowWindow 的nCmdShow 的形参的任意值，除了
      "SW_SHOWDEFAULT"。如此之外，此属性被忽略。

      "SW_HIDE" 被提供给此属性。它在 "Popen" 由 "shell=True" 调用时使
      用。

   lpAttributeList

      "STARTUPINFOEX" 给出的用于进程创建的额外属性字典，参阅
      UpdateProcThreadAttribute。

      支持的属性：

      **handle_list**
         将被继承的句柄的序列。如果非空， *close_fds* 必须为 true。

         当传递给 "Popen" 构造函数时，这些句柄必须暂时地能被
         "os.set_handle_inheritable()" 继承，否则 "OSError" 将以
         Windows error "ERROR_INVALID_PARAMETER" (87) 抛出。

         警告:

           在多线程进程中，请谨慎使用，以便在将此功能与对继承所有句柄
           的其他进程创建函数（例如 "os.system()"）的并发调用相结合时
           ，避免泄漏标记为可继承的句柄。这也应用于临时性创建可继承句
           柄的标准句柄重定向。

      3.7 新版功能.


Windows 常数
------------

"subprocess" 模块曝出以下常数。

subprocess.STD_INPUT_HANDLE

   标准输入设备，这是控制台输入缓冲区 "CONIN$"。

subprocess.STD_OUTPUT_HANDLE

   标准输出设备。最初，这是活动控制台屏幕缓冲区 "CONOUT$"。

subprocess.STD_ERROR_HANDLE

   标准错误设备。最初，这是活动控制台屏幕缓冲区 "CONOUT$"。

subprocess.SW_HIDE

   隐藏窗口。另一个窗口将被激活。

subprocess.STARTF_USESTDHANDLES

   Specifies that the "STARTUPINFO.hStdInput",
   "STARTUPINFO.hStdOutput", and "STARTUPINFO.hStdError" attributes
   contain additional information.

subprocess.STARTF_USESHOWWINDOW

   Specifies that the "STARTUPINFO.wShowWindow" attribute contains
   additional information.

subprocess.CREATE_NEW_CONSOLE

   The new process has a new console, instead of inheriting its
   parent's console (the default).

subprocess.CREATE_NEW_PROCESS_GROUP

   A "Popen" "creationflags" parameter to specify that a new process
   group will be created. This flag is necessary for using "os.kill()"
   on the subprocess.

   This flag is ignored if "CREATE_NEW_CONSOLE" is specified.

subprocess.ABOVE_NORMAL_PRIORITY_CLASS

   A "Popen" "creationflags" parameter to specify that a new process
   will have an above average priority.

   3.7 新版功能.

subprocess.BELOW_NORMAL_PRIORITY_CLASS

   A "Popen" "creationflags" parameter to specify that a new process
   will have a below average priority.

   3.7 新版功能.

subprocess.HIGH_PRIORITY_CLASS

   A "Popen" "creationflags" parameter to specify that a new process
   will have a high priority.

   3.7 新版功能.

subprocess.IDLE_PRIORITY_CLASS

   A "Popen" "creationflags" parameter to specify that a new process
   will have an idle (lowest) priority.

   3.7 新版功能.

subprocess.NORMAL_PRIORITY_CLASS

   A "Popen" "creationflags" parameter to specify that a new process
   will have an normal priority. (default)

   3.7 新版功能.

subprocess.REALTIME_PRIORITY_CLASS

   A "Popen" "creationflags" parameter to specify that a new process
   will have realtime priority. You should almost never use
   REALTIME_PRIORITY_CLASS, because this interrupts system threads
   that manage mouse input, keyboard input, and background disk
   flushing. This class can be appropriate for applications that
   "talk" directly to hardware or that perform brief tasks that should
   have limited interruptions.

   3.7 新版功能.

subprocess.CREATE_NO_WINDOW

   A "Popen" "creationflags" parameter to specify that a new process
   will not create a window.

   3.7 新版功能.

subprocess.DETACHED_PROCESS

   A "Popen" "creationflags" parameter to specify that a new process
   will not inherit its parent's console. This value cannot be used
   with CREATE_NEW_CONSOLE.

   3.7 新版功能.

subprocess.CREATE_DEFAULT_ERROR_MODE

   A "Popen" "creationflags" parameter to specify that a new process
   does not inherit the error mode of the calling process. Instead,
   the new process gets the default error mode. This feature is
   particularly useful for multithreaded shell applications that run
   with hard errors disabled.

   3.7 新版功能.

subprocess.CREATE_BREAKAWAY_FROM_JOB

   A "Popen" "creationflags" parameter to specify that a new process
   is not associated with the job.

   3.7 新版功能.


Older high-level API
====================

Prior to Python 3.5, these three functions comprised the high level
API to subprocess. You can now use "run()" in many cases, but lots of
existing code calls these functions.

subprocess.call(args, *, stdin=None, stdout=None, stderr=None, shell=False, cwd=None, timeout=None)

   Run the command described by *args*.  Wait for command to complete,
   then return the "returncode" attribute.

   Code needing to capture stdout or stderr should use "run()"
   instead:

      run(...).returncode

   To suppress stdout or stderr, supply a value of "DEVNULL".

   The arguments shown above are merely some common ones. The full
   function signature is the same as that of the "Popen" constructor -
   this function passes all supplied arguments other than *timeout*
   directly through to that interface.

   注解:

     Do not use "stdout=PIPE" or "stderr=PIPE" with this function.
     The child process will block if it generates enough output to a
     pipe to fill up the OS pipe buffer as the pipes are not being
     read from.

   在 3.3 版更改: *timeout* 被添加

subprocess.check_call(args, *, stdin=None, stdout=None, stderr=None, shell=False, cwd=None, timeout=None)

   Run command with arguments.  Wait for command to complete. If the
   return code was zero then return, otherwise raise
   "CalledProcessError". The "CalledProcessError" object will have the
   return code in the "returncode" attribute.

   Code needing to capture stdout or stderr should use "run()"
   instead:

      run(..., check=True)

   To suppress stdout or stderr, supply a value of "DEVNULL".

   The arguments shown above are merely some common ones. The full
   function signature is the same as that of the "Popen" constructor -
   this function passes all supplied arguments other than *timeout*
   directly through to that interface.

   注解:

     Do not use "stdout=PIPE" or "stderr=PIPE" with this function.
     The child process will block if it generates enough output to a
     pipe to fill up the OS pipe buffer as the pipes are not being
     read from.

   在 3.3 版更改: *timeout* 被添加

subprocess.check_output(args, *, stdin=None, stderr=None, shell=False, cwd=None, encoding=None, errors=None, universal_newlines=None, timeout=None, text=None)

   Run command with arguments and return its output.

   If the return code was non-zero it raises a "CalledProcessError".
   The "CalledProcessError" object will have the return code in the
   "returncode" attribute and any output in the "output" attribute.

   这相当于:

      run(..., check=True, stdout=PIPE).stdout

   The arguments shown above are merely some common ones. The full
   function signature is largely the same as that of "run()" - most
   arguments are passed directly through to that interface. However,
   explicitly passing "input=None" to inherit the parent's standard
   input file handle is not supported.

   By default, this function will return the data as encoded bytes.
   The actual encoding of the output data may depend on the command
   being invoked, so the decoding to text will often need to be
   handled at the application level.

   This behaviour may be overridden by setting *text*, *encoding*,
   *errors*, or *universal_newlines* to "True" as described in 常用参
   数 and "run()".

   To also capture standard error in the result, use
   "stderr=subprocess.STDOUT":

      >>> subprocess.check_output(
      ...     "ls non_existent_file; exit 0",
      ...     stderr=subprocess.STDOUT,
      ...     shell=True)
      'ls: non_existent_file: No such file or directory\n'

   3.1 新版功能.

   在 3.3 版更改: *timeout* 被添加

   在 3.4 版更改: Support for the *input* keyword argument was added.

   在 3.6 版更改: *encoding* and *errors* were added.  See "run()" for
   details.

   3.7 新版功能: *text* 作为 *universal_newlines* 的一个更具可读性的别
   名被添加。


Replacing Older Functions with the "subprocess" Module
======================================================

In this section, "a becomes b" means that b can be used as a
replacement for a.

注解:

  All "a" functions in this section fail (more or less) silently if
  the executed program cannot be found; the "b" replacements raise
  "OSError" instead.In addition, the replacements using
  "check_output()" will fail with a "CalledProcessError" if the
  requested operation produces a non-zero return code. The output is
  still available as the "output" attribute of the raised exception.

In the following examples, we assume that the relevant functions have
already been imported from the "subprocess" module.


Replacing **/bin/sh** shell command substitution
------------------------------------------------

   output=$(mycmd myarg)

becomes:

   output = check_output(["mycmd", "myarg"])


Replacing shell pipeline
------------------------

   output=$(dmesg | grep hda)

becomes:

   p1 = Popen(["dmesg"], stdout=PIPE)
   p2 = Popen(["grep", "hda"], stdin=p1.stdout, stdout=PIPE)
   p1.stdout.close()  # Allow p1 to receive a SIGPIPE if p2 exits.
   output = p2.communicate()[0]

The "p1.stdout.close()" call after starting the p2 is important in
order for p1 to receive a SIGPIPE if p2 exits before p1.

Alternatively, for trusted input, the shell's own pipeline support may
still be used directly:

   output=$(dmesg | grep hda)

becomes:

   output=check_output("dmesg | grep hda", shell=True)


Replacing "os.system()"
-----------------------

   sts = os.system("mycmd" + " myarg")
   # becomes
   sts = call("mycmd" + " myarg", shell=True)

注释:

* Calling the program through the shell is usually not required.

A more realistic example would look like this:

   try:
       retcode = call("mycmd" + " myarg", shell=True)
       if retcode < 0:
           print("Child was terminated by signal", -retcode, file=sys.stderr)
       else:
           print("Child returned", retcode, file=sys.stderr)
   except OSError as e:
       print("Execution failed:", e, file=sys.stderr)


Replacing the "os.spawn" family
-------------------------------

P_NOWAIT example:

   pid = os.spawnlp(os.P_NOWAIT, "/bin/mycmd", "mycmd", "myarg")
   ==>
   pid = Popen(["/bin/mycmd", "myarg"]).pid

P_WAIT example:

   retcode = os.spawnlp(os.P_WAIT, "/bin/mycmd", "mycmd", "myarg")
   ==>
   retcode = call(["/bin/mycmd", "myarg"])

Vector example:

   os.spawnvp(os.P_NOWAIT, path, args)
   ==>
   Popen([path] + args[1:])

Environment example:

   os.spawnlpe(os.P_NOWAIT, "/bin/mycmd", "mycmd", "myarg", env)
   ==>
   Popen(["/bin/mycmd", "myarg"], env={"PATH": "/usr/bin"})


Replacing "os.popen()", "os.popen2()", "os.popen3()"
----------------------------------------------------

   (child_stdin, child_stdout) = os.popen2(cmd, mode, bufsize)
   ==>
   p = Popen(cmd, shell=True, bufsize=bufsize,
             stdin=PIPE, stdout=PIPE, close_fds=True)
   (child_stdin, child_stdout) = (p.stdin, p.stdout)

   (child_stdin,
    child_stdout,
    child_stderr) = os.popen3(cmd, mode, bufsize)
   ==>
   p = Popen(cmd, shell=True, bufsize=bufsize,
             stdin=PIPE, stdout=PIPE, stderr=PIPE, close_fds=True)
   (child_stdin,
    child_stdout,
    child_stderr) = (p.stdin, p.stdout, p.stderr)

   (child_stdin, child_stdout_and_stderr) = os.popen4(cmd, mode, bufsize)
   ==>
   p = Popen(cmd, shell=True, bufsize=bufsize,
             stdin=PIPE, stdout=PIPE, stderr=STDOUT, close_fds=True)
   (child_stdin, child_stdout_and_stderr) = (p.stdin, p.stdout)

Return code handling translates as follows:

   pipe = os.popen(cmd, 'w')
   ...
   rc = pipe.close()
   if rc is not None and rc >> 8:
       print("There were some errors")
   ==>
   process = Popen(cmd, stdin=PIPE)
   ...
   process.stdin.close()
   if process.wait() != 0:
       print("There were some errors")


Replacing functions from the "popen2" module
--------------------------------------------

注解:

  If the cmd argument to popen2 functions is a string, the command is
  executed through /bin/sh.  If it is a list, the command is directly
  executed.

   (child_stdout, child_stdin) = popen2.popen2("somestring", bufsize, mode)
   ==>
   p = Popen("somestring", shell=True, bufsize=bufsize,
             stdin=PIPE, stdout=PIPE, close_fds=True)
   (child_stdout, child_stdin) = (p.stdout, p.stdin)

   (child_stdout, child_stdin) = popen2.popen2(["mycmd", "myarg"], bufsize, mode)
   ==>
   p = Popen(["mycmd", "myarg"], bufsize=bufsize,
             stdin=PIPE, stdout=PIPE, close_fds=True)
   (child_stdout, child_stdin) = (p.stdout, p.stdin)

"popen2.Popen3" and "popen2.Popen4" basically work as
"subprocess.Popen", except that:

* "Popen" raises an exception if the execution fails.

* The *capturestderr* argument is replaced with the *stderr* argument.

* "stdin=PIPE" and "stdout=PIPE" must be specified.

* popen2 closes all file descriptors by default, but you have to
  specify "close_fds=True" with "Popen" to guarantee this behavior on
  all platforms or past Python versions.


Legacy Shell Invocation Functions
=================================

This module also provides the following legacy functions from the 2.x
"commands" module. These operations implicitly invoke the system shell
and none of the guarantees described above regarding security and
exception handling consistency are valid for these functions.

subprocess.getstatusoutput(cmd)

   Return "(exitcode, output)" of executing *cmd* in a shell.

   Execute the string *cmd* in a shell with "Popen.check_output()" and
   return a 2-tuple "(exitcode, output)". The locale encoding is used;
   see the notes on 常用参数 for more details.

   A trailing newline is stripped from the output. The exit code for
   the command can be interpreted as the return code of subprocess.
   Example:

      >>> subprocess.getstatusoutput('ls /bin/ls')
      (0, '/bin/ls')
      >>> subprocess.getstatusoutput('cat /bin/junk')
      (1, 'cat: /bin/junk: No such file or directory')
      >>> subprocess.getstatusoutput('/bin/junk')
      (127, 'sh: /bin/junk: not found')
      >>> subprocess.getstatusoutput('/bin/kill $$')
      (-15, '')

   Availability: POSIX & Windows.

   在 3.3.4 版更改: 添加了 Windows 支持。The function now returns
   (exitcode, output) instead of (status, output) as it did in Python
   3.3.3 and earlier.  exitcode has the same value as "returncode".

subprocess.getoutput(cmd)

   Return output (stdout and stderr) of executing *cmd* in a shell.

   Like "getstatusoutput()", except the exit code is ignored and the
   return value is a string containing the command's output.  Example:

      >>> subprocess.getoutput('ls /bin/ls')
      '/bin/ls'

   Availability: POSIX & Windows.

   在 3.3.4 版更改: 添加了 Windows 支持


注释
====


Converting an argument sequence to a string on Windows
------------------------------------------------------

On Windows, an *args* sequence is converted to a string that can be
parsed using the following rules (which correspond to the rules used
by the MS C runtime):

1. Arguments are delimited by white space, which is either a space or
   a tab.

2. A string surrounded by double quotation marks is interpreted as a
   single argument, regardless of white space contained within.  A
   quoted string can be embedded in an argument.

3. A double quotation mark preceded by a backslash is interpreted as a
   literal double quotation mark.

4. Backslashes are interpreted literally, unless they immediately
   precede a double quotation mark.

5. If backslashes immediately precede a double quotation mark, every
   pair of backslashes is interpreted as a literal backslash.  If the
   number of backslashes is odd, the last backslash escapes the next
   double quotation mark as described in rule 3.

参见:

  "shlex"
     Module which provides function to parse and escape command lines.

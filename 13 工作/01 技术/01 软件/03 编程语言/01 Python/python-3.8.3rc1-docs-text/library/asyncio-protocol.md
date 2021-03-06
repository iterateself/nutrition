传输和协议
**********

-[ 前言 ]-

传输和协议会被像 "loop.create_connection()" 这类 **底层** 事件循环接口
使用。它们使用基于回调的编程风格支持网络或IPC协议（如HTTP）的高性能实
现。

基本上，传输和协议应只在库和框架上使用，而不应该在高层的异步应用中使用
它们。

本文档包含  Transports 和 Protocols 。

-[ 概述 ]-

在最顶层，传输只关心 **怎样** 传送字节内容，而协议决定传送 **哪些** 字
节内容(还要在一定程度上考虑何时)。

也可以这样说：从传输的角度来看，传输是套接字(或类似的I/O终端)的抽象，
而协议是应用程序的抽象。

换另一种说法，传输和协议一起定义网络I/0和进程间I/O的抽象接口。

传输对象和协议对象总是一对一关系：协议调用传输方法来发送数据，而传输在
接收到数据时调用协议方法传递数据。

大部分面向连接的事件循环方法(如 "loop.create_connection()" ) 通常接受
*protocol_factory* 参数为接收到的链接创建 *协议* 对象，并用 *传输* 对
象来表示。这些方法一般会返回 "(transport, protocol)" 元组。

-[ 内容 ]-

本文档包含下列小节:

* 传输 部分记载异步IO "BaseTransport" 、 "ReadTransport" 、
  "WriteTransport" 、 "Transport" 、 "DatagramTransport" 和
  "SubprocessTransport" 等类。

* The Protocols section documents asyncio "BaseProtocol", "Protocol",
  "BufferedProtocol", "DatagramProtocol", and "SubprocessProtocol"
  classes.

* 例子 部分展示怎样使用传输、协议和底层事件循环接口。


传输
====

**源码:** Lib/asyncio/transports.py

======================================================================

传输属于 "asyncio" 模块中的类，用来抽象各种通信通道。

传输对象总是由 异步IO事件循环 实例化。

异步IO实现TCP、UDP、SSL和子进程管道的传输。传输上可用的方法由传输的类
型决定。

传输类属于 线程不安全 。


传输层级
--------

class asyncio.BaseTransport

   所有传输的基类。包含所有异步IO传输共用的方法。

class asyncio.WriteTransport(BaseTransport)

   只写链接的基础传输。

   Instances of the *WriteTransport* class are returned from the
   "loop.connect_write_pipe()" event loop method and are also used by
   subprocess-related methods like "loop.subprocess_exec()".

class asyncio.ReadTransport(BaseTransport)

   只读链接的基础传输。

   Instances of the *ReadTransport* class are returned from the
   "loop.connect_read_pipe()" event loop method and are also used by
   subprocess-related methods like "loop.subprocess_exec()".

class asyncio.Transport(WriteTransport, ReadTransport)

   接口代表一个双向传输，如TCP链接。

   用户不用直接实例化传输；调用一个功能函数，给它传递协议工厂和其它需
   要的信息就可以创建传输和协议。

   *传输* 类实例由如  "loop.create_connection()" 、
   "loop.create_unix_connection()" 、  "loop.create_server()" 、
   "loop.sendfile()" 等这类事件循环方法使用或返回。

class asyncio.DatagramTransport(BaseTransport)

   数据报(UDP)传输链接。

   *DatagramTransport* 类实例由事件循环方法
   "loop.create_datagram_endpoint()" 返回。

class asyncio.SubprocessTransport(BaseTransport)

   表示父进程和子进程之间连接的抽象。

   *SubprocessTransport* 类的实例由事件循环方法
   "loop.subprocess_shell()"  和  "loop.subprocess_exec()" 返回。


基础传输
--------

BaseTransport.close()

   关闭传输。

   如果传输具有发送数据缓冲区，将会异步发送已缓存的数据。在所有已缓存
   的数据都已处理后，就会将 "None" 作为协议
   "protocol.connection_lost()" 方法的参数并进行调用。在这之后，传输不
   再接收任何数据。

BaseTransport.is_closing()

   返回 "True" ，如果传输正在关闭或已经关闭。。

BaseTransport.get_extra_info(name, default=None)

   返回 传输或它使用的相关资源信息。

   *name* 是表示要获取传输特定信息的字符串。

   *default* 是在信息不可用或传输不支持第三方事件循环实现或当前平台查
   询时返回的值。

   例如下面代码尝试获取传输相关套接字对象:

      sock = transport.get_extra_info('socket')
      if sock is not None:
          print(sock.getsockopt(...))

   传输可查询信息类别:

   * 套接字:

     * "'peername'": 套接字链接时的远端地址，
       "socket.socket.getpeername()" 方法的结果 (出错时为 "None" )

     * "'socket'": "socket.socket" 实例

     * "'sockname'": 套接字本地地址， "socket.socket.getsockname()" 方
       法的结果

   * SSL套接字

     * "'compression'": 用字符串指定压缩算法，或者链接没有压缩时为
       "None"  ；"ssl.SSLSocket.compression()" 的结果。

     * "'cipher'": 一个三值的元组，包含使用密码的名称，定义使用的SSL协
       议的版本，使用的加密位数。  "ssl.SSLSocket.cipher()" 的结果。

     * "'peercert'": 远端凭证；  "ssl.SSLSocket.getpeercert()" 结果。

     * "'sslcontext'": "ssl.SSLContext" 实例

     * "'ssl_object'": "ssl.SSLObject" 或 "ssl.SSLSocket" 实例

   * 管道：

     * "'pipe'": 管道对象

   * 子进程：

     * "'subprocess'": "subprocess.Popen" 实例

BaseTransport.set_protocol(protocol)

   设置一个新协议。

   只有两种协议都写明支持切换才能完成切换协议。

BaseTransport.get_protocol()

   返回当前协议。


只读传输
--------

ReadTransport.is_reading()

   如果传输接收到新数据时返回 "True" 。

   3.7 新版功能.

ReadTransport.pause_reading()

   暂停传输的接收端。"protocol.data_received()" 方法将不会收到数据，除
   非 "resume_reading()"  被调用。

   在 3.7 版更改: 这个方法幂等的， 它可以在传输已经暂停或关闭时调用。

ReadTransport.resume_reading()

   恢复接收端。如果有数据可读取时，协议方法 "protocol.data_received()"
   将再次被调用。

   在 3.7 版更改: 这个方法幂等的， 它可以在传输已经准备好读取数据时调
   用。


只写传输
--------

WriteTransport.abort()

   立即关闭传输，不会等待已提交的操作处理完毕。已缓存的数据将会丢失。
   不会接收更多数据。   最终 "None" 将作为协议的
   "protocol.connection_lost()"  方法的参数被调用。

WriteTransport.can_write_eof()

   如果传输支持 "write_eof()" 返回  "True" 否则返回  "False"  。

WriteTransport.get_write_buffer_size()

   返回传输使用输出缓冲区的当前大小。

WriteTransport.get_write_buffer_limits()

   获取写入流控制 *high* 和 *low* 高低标记位。返回元组 "(low, high)"
   ， *low* 和 *high* 为正字节数。

   使用 "set_write_buffer_limits()" 设置限制。

   3.4.2 新版功能.

WriteTransport.set_write_buffer_limits(high=None, low=None)

   设置写入流控制 *high* 和 *low* 高低标记位。

   These two values (measured in number of bytes) control when the
   protocol's "protocol.pause_writing()" and
   "protocol.resume_writing()" methods are called. If specified, the
   low watermark must be less than or equal to the high watermark.
   Neither *high* nor *low* can be negative.

   "pause_writing()" is called when the buffer size becomes greater
   than or equal to the *high* value. If writing has been paused,
   "resume_writing()" is called when the buffer size becomes less than
   or equal to the *low* value.

   The defaults are implementation-specific.  If only the high
   watermark is given, the low watermark defaults to an
   implementation-specific value less than or equal to the high
   watermark.  Setting *high* to zero forces *low* to zero as well,
   and causes "pause_writing()" to be called whenever the buffer
   becomes non-empty.  Setting *low* to zero causes "resume_writing()"
   to be called only once the buffer is empty. Use of zero for either
   limit is generally sub-optimal as it reduces opportunities for
   doing I/O and computation concurrently.

   Use "get_write_buffer_limits()" to get the limits.

WriteTransport.write(data)

   Write some *data* bytes to the transport.

   This method does not block; it buffers the data and arranges for it
   to be sent out asynchronously.

WriteTransport.writelines(list_of_data)

   Write a list (or any iterable) of data bytes to the transport. This
   is functionally equivalent to calling "write()" on each element
   yielded by the iterable, but may be implemented more efficiently.

WriteTransport.write_eof()

   Close the write end of the transport after flushing all buffered
   data. Data may still be received.

   This method can raise "NotImplementedError" if the transport (e.g.
   SSL) doesn't support half-closed connections.


数据报传输
----------

DatagramTransport.sendto(data, addr=None)

   Send the *data* bytes to the remote peer given by *addr* (a
   transport-dependent target address).  If *addr* is "None", the data
   is sent to the target address given on transport creation.

   This method does not block; it buffers the data and arranges for it
   to be sent out asynchronously.

DatagramTransport.abort()

   Close the transport immediately, without waiting for pending
   operations to complete.  Buffered data will be lost. No more data
   will be received.  The protocol's "protocol.connection_lost()"
   method will eventually be called with "None" as its argument.


子进程传输
----------

SubprocessTransport.get_pid()

   Return the subprocess process id as an integer.

SubprocessTransport.get_pipe_transport(fd)

   Return the transport for the communication pipe corresponding to
   the integer file descriptor *fd*:

   * "0": readable streaming transport of the standard input
     (*stdin*), or "None" if the subprocess was not created with
     "stdin=PIPE"

   * "1": writable streaming transport of the standard output
     (*stdout*), or "None" if the subprocess was not created with
     "stdout=PIPE"

   * "2": writable streaming transport of the standard error
     (*stderr*), or "None" if the subprocess was not created with
     "stderr=PIPE"

   * other *fd*: "None"

SubprocessTransport.get_returncode()

   Return the subprocess return code as an integer or "None" if it
   hasn't returned, which is similar to the
   "subprocess.Popen.returncode" attribute.

SubprocessTransport.kill()

   杀死子进程。

   On POSIX systems, the function sends SIGKILL to the subprocess. On
   Windows, this method is an alias for "terminate()".

   See also "subprocess.Popen.kill()".

SubprocessTransport.send_signal(signal)

   Send the *signal* number to the subprocess, as in
   "subprocess.Popen.send_signal()".

SubprocessTransport.terminate()

   停止子进程。

   On POSIX systems, this method sends SIGTERM to the subprocess. On
   Windows, the Windows API function TerminateProcess() is called to
   stop the subprocess.

   See also "subprocess.Popen.terminate()".

SubprocessTransport.close()

   Kill the subprocess by calling the "kill()" method.

   If the subprocess hasn't returned yet, and close transports of
   *stdin*, *stdout*, and *stderr* pipes.


协议
====

**源码:** Lib/asyncio/protocols.py

======================================================================

asyncio provides a set of abstract base classes that should be used to
implement network protocols.  Those classes are meant to be used
together with transports.

Subclasses of abstract base protocol classes may implement some or all
methods.  All these methods are callbacks: they are called by
transports on certain events, for example when some data is received.
A base protocol method should be called by the corresponding
transport.


基础协议
--------

class asyncio.BaseProtocol

   Base protocol with methods that all protocols share.

class asyncio.Protocol(BaseProtocol)

   The base class for implementing streaming protocols (TCP, Unix
   sockets, etc).

class asyncio.BufferedProtocol(BaseProtocol)

   A base class for implementing streaming protocols with manual
   control of the receive buffer.

class asyncio.DatagramProtocol(BaseProtocol)

   The base class for implementing datagram (UDP) protocols.

class asyncio.SubprocessProtocol(BaseProtocol)

   The base class for implementing protocols communicating with child
   processes (unidirectional pipes).


基础协议
--------

All asyncio protocols can implement Base Protocol callbacks.

-[ 链接回调函数 ]-

Connection callbacks are called on all protocols, exactly once per a
successful connection.  All other protocol callbacks can only be
called between those two methods.

BaseProtocol.connection_made(transport)

   链接建立时被调用。

   The *transport* argument is the transport representing the
   connection.  The protocol is responsible for storing the reference
   to its transport.

BaseProtocol.connection_lost(exc)

   链接丢失或关闭时被调用。

   The argument is either an exception object or "None". The latter
   means a regular EOF is received, or the connection was aborted or
   closed by this side of the connection.

-[ 流控制回调函数 ]-

Flow control callbacks can be called by transports to pause or resume
writing performed by the protocol.

See the documentation of the "set_write_buffer_limits()" method for
more details.

BaseProtocol.pause_writing()

   Called when the transport's buffer goes over the high watermark.

BaseProtocol.resume_writing()

   Called when the transport's buffer drains below the low watermark.

If the buffer size equals the high watermark, "pause_writing()" is not
called: the buffer size must go strictly over.

Conversely, "resume_writing()" is called when the buffer size is equal
or lower than the low watermark.  These end conditions are important
to ensure that things go as expected when either mark is zero.


流协议
------

Event methods, such as "loop.create_server()",
"loop.create_unix_server()", "loop.create_connection()",
"loop.create_unix_connection()", "loop.connect_accepted_socket()",
"loop.connect_read_pipe()", and "loop.connect_write_pipe()" accept
factories that return streaming protocols.

Protocol.data_received(data)

   Called when some data is received.  *data* is a non-empty bytes
   object containing the incoming data.

   Whether the data is buffered, chunked or reassembled depends on the
   transport.  In general, you shouldn't rely on specific semantics
   and instead make your parsing generic and flexible. However, data
   is always received in the correct order.

   The method can be called an arbitrary number of times while a
   connection is open.

   However, "protocol.eof_received()" is called at most once.  Once
   *eof_received()* is called, "data_received()" is not called
   anymore.

Protocol.eof_received()

   Called when the other end signals it won't send any more data (for
   example by calling "transport.write_eof()", if the other end also
   uses asyncio).

   This method may return a false value (including "None"), in which
   case the transport will close itself.  Conversely, if this method
   returns a true value, the protocol used determines whether to close
   the transport. Since the default implementation returns "None", it
   implicitly closes the connection.

   Some transports, including SSL, don't support half-closed
   connections, in which case returning true from this method will
   result in the connection being closed.

状态机：

   start -> connection_made
       [-> data_received]*
       [-> eof_received]?
   -> connection_lost -> end


缓冲流协议
----------

3.7 新版功能: **Important:** this has been added to asyncio in Python
3.7 *on a provisional basis*!  This is as an experimental API that
might be changed or removed completely in Python 3.8.

Buffered Protocols can be used with any event loop method that
supports Streaming Protocols.

"BufferedProtocol" implementations allow explicit manual allocation
and control of the receive buffer.  Event loops can then use the
buffer provided by the protocol to avoid unnecessary data copies.
This can result in noticeable performance improvement for protocols
that receive big amounts of data.  Sophisticated protocol
implementations can significantly reduce the number of buffer
allocations.

The following callbacks are called on "BufferedProtocol" instances:

BufferedProtocol.get_buffer(sizehint)

   调用后会分配新的接收缓冲区。

   *sizehint* is the recommended minimum size for the returned buffer.
   It is acceptable to return smaller or larger buffers than what
   *sizehint* suggests.  When set to -1, the buffer size can be
   arbitrary. It is an error to return a buffer with a zero size.

   "get_buffer()" must return an object implementing the buffer
   protocol.

BufferedProtocol.buffer_updated(nbytes)

   用接收的数据更新缓冲区时被调用。

   *nbytes* is the total number of bytes that were written to the
   buffer.

BufferedProtocol.eof_received()

   See the documentation of the "protocol.eof_received()" method.

"get_buffer()" can be called an arbitrary number of times during a
connection.  However, "protocol.eof_received()" is called at most once
and, if called, "get_buffer()" and "buffer_updated()" won't be called
after it.

状态机：

   start -> connection_made
       [-> get_buffer
           [-> buffer_updated]?
       ]*
       [-> eof_received]?
   -> connection_lost -> end


数据报协议
----------

Datagram Protocol instances should be constructed by protocol
factories passed to the "loop.create_datagram_endpoint()" method.

DatagramProtocol.datagram_received(data, addr)

   Called when a datagram is received.  *data* is a bytes object
   containing the incoming data.  *addr* is the address of the peer
   sending the data; the exact format depends on the transport.

DatagramProtocol.error_received(exc)

   Called when a previous send or receive operation raises an
   "OSError".  *exc* is the "OSError" instance.

   This method is called in rare conditions, when the transport (e.g.
   UDP) detects that a datagram could not be delivered to its
   recipient. In many conditions though, undeliverable datagrams will
   be silently dropped.

注解:

  On BSD systems (macOS, FreeBSD, etc.) flow control is not supported
  for datagram protocols, because there is no reliable way to detect
  send failures caused by writing too many packets.The socket always
  appears 'ready' and excess packets are dropped. An "OSError" with
  "errno" set to "errno.ENOBUFS" may or may not be raised; if it is
  raised, it will be reported to "DatagramProtocol.error_received()"
  but otherwise ignored.


子进程协议
----------

Datagram Protocol instances should be constructed by protocol
factories passed to the "loop.subprocess_exec()" and
"loop.subprocess_shell()" methods.

SubprocessProtocol.pipe_data_received(fd, data)

   Called when the child process writes data into its stdout or stderr
   pipe.

   *fd* is the integer file descriptor of the pipe.

   *data* is a non-empty bytes object containing the received data.

SubprocessProtocol.pipe_connection_lost(fd, exc)

   与子进程通信的其中一个管道关闭时被调用。

   *fd* is the integer file descriptor that was closed.

SubprocessProtocol.process_exited()

   子进程退出时被调用。


示例
====


TCP回应服务器
-------------

Create a TCP echo server using the "loop.create_server()" method, send
back received data, and close the connection:

   import asyncio


   class EchoServerProtocol(asyncio.Protocol):
       def connection_made(self, transport):
           peername = transport.get_extra_info('peername')
           print('Connection from {}'.format(peername))
           self.transport = transport

       def data_received(self, data):
           message = data.decode()
           print('Data received: {!r}'.format(message))

           print('Send: {!r}'.format(message))
           self.transport.write(data)

           print('Close the client socket')
           self.transport.close()


   async def main():
       # Get a reference to the event loop as we plan to use
       # low-level APIs.
       loop = asyncio.get_running_loop()

       server = await loop.create_server(
           lambda: EchoServerProtocol(),
           '127.0.0.1', 8888)

       async with server:
           await server.serve_forever()


   asyncio.run(main())

参见:

  The TCP echo server using streams example uses the high-level
  "asyncio.start_server()" function.


TCP回应客户端
-------------

A TCP echo client using the "loop.create_connection()" method, sends
data, and waits until the connection is closed:

   import asyncio


   class EchoClientProtocol(asyncio.Protocol):
       def __init__(self, message, on_con_lost):
           self.message = message
           self.on_con_lost = on_con_lost

       def connection_made(self, transport):
           transport.write(self.message.encode())
           print('Data sent: {!r}'.format(self.message))

       def data_received(self, data):
           print('Data received: {!r}'.format(data.decode()))

       def connection_lost(self, exc):
           print('The server closed the connection')
           self.on_con_lost.set_result(True)


   async def main():
       # Get a reference to the event loop as we plan to use
       # low-level APIs.
       loop = asyncio.get_running_loop()

       on_con_lost = loop.create_future()
       message = 'Hello World!'

       transport, protocol = await loop.create_connection(
           lambda: EchoClientProtocol(message, on_con_lost),
           '127.0.0.1', 8888)

       # Wait until the protocol signals that the connection
       # is lost and close the transport.
       try:
           await on_con_lost
       finally:
           transport.close()


   asyncio.run(main())

参见:

  The TCP echo client using streams example uses the high-level
  "asyncio.open_connection()" function.


UDP回应服务器
-------------

A UDP echo server, using the "loop.create_datagram_endpoint()" method,
sends back received data:

   import asyncio


   class EchoServerProtocol:
       def connection_made(self, transport):
           self.transport = transport

       def datagram_received(self, data, addr):
           message = data.decode()
           print('Received %r from %s' % (message, addr))
           print('Send %r to %s' % (message, addr))
           self.transport.sendto(data, addr)


   async def main():
       print("Starting UDP server")

       # Get a reference to the event loop as we plan to use
       # low-level APIs.
       loop = asyncio.get_running_loop()

       # One protocol instance will be created to serve all
       # client requests.
       transport, protocol = await loop.create_datagram_endpoint(
           lambda: EchoServerProtocol(),
           local_addr=('127.0.0.1', 9999))

       try:
           await asyncio.sleep(3600)  # Serve for 1 hour.
       finally:
           transport.close()


   asyncio.run(main())


UDP回应客户端
-------------

A UDP echo client, using the "loop.create_datagram_endpoint()" method,
sends data and closes the transport when it receives the answer:

   import asyncio


   class EchoClientProtocol:
       def __init__(self, message, on_con_lost):
           self.message = message
           self.on_con_lost = on_con_lost
           self.transport = None

       def connection_made(self, transport):
           self.transport = transport
           print('Send:', self.message)
           self.transport.sendto(self.message.encode())

       def datagram_received(self, data, addr):
           print("Received:", data.decode())

           print("Close the socket")
           self.transport.close()

       def error_received(self, exc):
           print('Error received:', exc)

       def connection_lost(self, exc):
           print("Connection closed")
           self.on_con_lost.set_result(True)


   async def main():
       # Get a reference to the event loop as we plan to use
       # low-level APIs.
       loop = asyncio.get_running_loop()

       on_con_lost = loop.create_future()
       message = "Hello World!"

       transport, protocol = await loop.create_datagram_endpoint(
           lambda: EchoClientProtocol(message, on_con_lost),
           remote_addr=('127.0.0.1', 9999))

       try:
           await on_con_lost
       finally:
           transport.close()


   asyncio.run(main())


链接已存在的套接字
------------------

Wait until a socket receives data using the "loop.create_connection()"
method with a protocol:

   import asyncio
   import socket


   class MyProtocol(asyncio.Protocol):

       def __init__(self, on_con_lost):
           self.transport = None
           self.on_con_lost = on_con_lost

       def connection_made(self, transport):
           self.transport = transport

       def data_received(self, data):
           print("Received:", data.decode())

           # We are done: close the transport;
           # connection_lost() will be called automatically.
           self.transport.close()

       def connection_lost(self, exc):
           # The socket has been closed
           self.on_con_lost.set_result(True)


   async def main():
       # Get a reference to the event loop as we plan to use
       # low-level APIs.
       loop = asyncio.get_running_loop()
       on_con_lost = loop.create_future()

       # Create a pair of connected sockets
       rsock, wsock = socket.socketpair()

       # Register the socket to wait for data.
       transport, protocol = await loop.create_connection(
           lambda: MyProtocol(on_con_lost), sock=rsock)

       # Simulate the reception of data from the network.
       loop.call_soon(wsock.send, 'abc'.encode())

       try:
           await protocol.on_con_lost
       finally:
           transport.close()
           wsock.close()

   asyncio.run(main())

参见:

  The watch a file descriptor for read events example uses the low-
  level "loop.add_reader()" method to register an FD.

  The register an open socket to wait for data using streams example
  uses high-level streams created by the "open_connection()" function
  in a coroutine.


loop.subprocess_exec() and SubprocessProtocol
---------------------------------------------

An example of a subprocess protocol used to get the output of a
subprocess and to wait for the subprocess exit.

The subprocess is created by th "loop.subprocess_exec()" method:

   import asyncio
   import sys

   class DateProtocol(asyncio.SubprocessProtocol):
       def __init__(self, exit_future):
           self.exit_future = exit_future
           self.output = bytearray()

       def pipe_data_received(self, fd, data):
           self.output.extend(data)

       def process_exited(self):
           self.exit_future.set_result(True)

   async def get_date():
       # Get a reference to the event loop as we plan to use
       # low-level APIs.
       loop = asyncio.get_running_loop()

       code = 'import datetime; print(datetime.datetime.now())'
       exit_future = asyncio.Future(loop=loop)

       # Create the subprocess controlled by DateProtocol;
       # redirect the standard output into a pipe.
       transport, protocol = await loop.subprocess_exec(
           lambda: DateProtocol(exit_future),
           sys.executable, '-c', code,
           stdin=None, stderr=None)

       # Wait for the subprocess exit using the process_exited()
       # method of the protocol.
       await exit_future

       # Close the stdout pipe.
       transport.close()

       # Read the output which was collected by the
       # pipe_data_received() method of the protocol.
       data = bytes(protocol.output)
       return data.decode('ascii').rstrip()

   date = asyncio.run(get_date())
   print(f"Current date: {date}")

See also the same example written using high-level APIs.

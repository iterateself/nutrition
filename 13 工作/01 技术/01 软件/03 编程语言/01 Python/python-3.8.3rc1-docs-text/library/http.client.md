"http.client" --- HTTP 协议客户端
*********************************

**源代码：** Lib/http/client.py

======================================================================

这个模块定义了实现 HTTP 和 HTTPS 协议客户端的类。 它通常不直接使用 ---
模块 "urllib.request" 用它来处理使用 HTTP 和 HTTPS 的 URL。

参见:

  The Requests package is recommended for a higher-level HTTP client
  interface.

注解:

  HTTPS 支持仅在编译 Python 时启用了 SSL 支持的情况下（通过 "ssl" 模块
  ）可用。

该模块支持以下类：

class http.client.HTTPConnection(host, port=None[, timeout], source_address=None, blocksize=8192)

   "HTTPConnection" 的实例代表与 HTTP 的一个连接事务。 它的实例化应当
   传入一个主机和可选的端口号。 如果没有传入端口号，如果主机字符串的形
   式为 "主机:端口" 则会从中提取端口，否则将使用默认的 HTTP 端口（80）
   。 如果给出了可选的 *timeout* 参数，则阻塞操作（例如连接尝试）将在
   指定的秒数之后超时（如果未给出，则使用全局默认超时设置）。 可选的
   *source_address* 参数可以为一个 (主机, 端口) 元组，用作进行 HTTP 连
   接的源地址。 可选的 *blocksize* 参数可以字节为单位设置缓冲区的大小
   ，用来发送文件类消息体。

   举个例子，以下调用都是创建连接到同一主机和端口的服务器的实例：

      >>> h1 = http.client.HTTPConnection('www.python.org')
      >>> h2 = http.client.HTTPConnection('www.python.org:80')
      >>> h3 = http.client.HTTPConnection('www.python.org', 80)
      >>> h4 = http.client.HTTPConnection('www.python.org', 80, timeout=10)

   在 3.2 版更改: 添加了 *source_address*。

   在 3.4 版更改: 删除了 *strict* 参数，不再支持 HTTP 0.9 风格的“简单
   响应”。

   在 3.7 版更改: 添加了 *blocksize* 参数。

class http.client.HTTPSConnection(host, port=None, key_file=None, cert_file=None[, timeout], source_address=None, *, context=None, check_hostname=None, blocksize=8192)

   "HTTPConnection" 的子类，使用 SSL 与安全服务器进行通信。 默认端口为
   "443"。 如果指定了 *context*，它必须为一个描述 SSL 各选项的
   "ssl.SSLContext" 实例。

   请参阅 Security considerations 了解有关最佳实践的更多信息。

   在 3.2 版更改: 添加了 *source_address*, *context* 和
   *check_hostname*。

   在 3.2 版更改: 这个类目前会在可能的情况下（即如果 "ssl.HAS_SNI" 为
   真值）支持 HTTPS 虚拟主机。

   在 3.4 版更改: 删除了 *strict* 参数，不再支持 HTTP 0.9 风格的“简单
   响应”。

   在 3.4.3 版更改: 目前这个类在默认情况下会执行所有必要的证书和主机检
   查。 要回复到先前的非验证行为，可以将
   "ssl._create_unverified_context()" 传递给 *context* 参数。

   在 3.8 版更改: 该类现在对于默认的 *context* 或在传入 *cert_file* 并
   附带自定义 *context* 时会启用 TLS 1.3
   "ssl.SSLContext.post_handshake_auth"。

   3.6 版后已移除: *key_file* 和 *cert_file* 已弃用并转而推荐
   *context*。 请改用 "ssl.SSLContext.load_cert_chain()" 或让
   "ssl.create_default_context()" 为你选择系统所信任的 CA 证书。
   *check_hostname* 参数也已弃用；应当改用 *context* 的
   "ssl.SSLContext.check_hostname" 属性。

class http.client.HTTPResponse(sock, debuglevel=0, method=None, url=None)

   在成功连接后返回类的实例，而不是由用户直接实例化。

   在 3.4 版更改: 删除了 *strict* 参数，不再支持HTTP 0.9 风格的“简单响
   应”。

This module provides the following function:

http.client.parse_headers(fp)

   Parse the headers from a file pointer *fp* representing a HTTP
   request/response. The file has to be a "BufferedIOBase" reader
   (i.e. not text) and must provide a valid **RFC 2822** style header.

   This function returns an instance of "http.client.HTTPMessage" that
   holds the header fields, but no payload (the same as
   "HTTPResponse.msg" and
   "http.server.BaseHTTPRequestHandler.headers"). After returning, the
   file pointer *fp* is ready to read the HTTP body.

   注解:

     "parse_headers()" does not parse the start-line of a HTTP
     message; it only parses the "Name: value" lines. The file has to
     be ready to read these field lines, so the first line should
     already be consumed before calling the function.

下列异常可以适当地被引发:

exception http.client.HTTPException

   此模块中其他异常的基类。 它是 "Exception" 的一个子类。

exception http.client.NotConnected

   "HTTPException" 的一个子类。

exception http.client.InvalidURL

   "HTTPException" 的一个子类，如果给出了一个非数字或为空值的端口就会
   被引发。

exception http.client.UnknownProtocol

   "HTTPException" 的一个子类。

exception http.client.UnknownTransferEncoding

   "HTTPException" 的一个子类。

exception http.client.UnimplementedFileMode

   "HTTPException" 的一个子类。

exception http.client.IncompleteRead

   "HTTPException" 的一个子类。

exception http.client.ImproperConnectionState

   "HTTPException" 的一个子类。

exception http.client.CannotSendRequest

   "ImproperConnectionState" 的一个子类。

exception http.client.CannotSendHeader

   "ImproperConnectionState" 的一个子类。

exception http.client.ResponseNotReady

   "ImproperConnectionState" 的一个子类。

exception http.client.BadStatusLine

   "HTTPException" 的一个子类。 如果服务器反馈了一个我们不理解的 HTTP
   状态码就会被引发。

exception http.client.LineTooLong

   "HTTPException" 的一个子类。 如果在 HTTP 协议中从服务器接收到过长的
   行就会被引发。

exception http.client.RemoteDisconnected

   "ConnectionResetError" 和 "BadStatusLine" 的一个子类。 当尝试读取响
   应时的结果是未从连接读取到数据时由 "HTTPConnection.getresponse()"
   引发，表明远端已关闭连接。

   3.5 新版功能: 在此之前引发的异常为 "BadStatusLine""('')"。

此模块中定义的常量为：

http.client.HTTP_PORT

   HTTP 协议默认的端口号 (总是 "80")。

http.client.HTTPS_PORT

   HTTPS 协议默认的端口号 (总是 "443")。

http.client.responses

   这个字典把 HTTP 1.1 状态码映射到 W3C 名称。

   例如："http.client.responses[http.client.NOT_FOUND]" 是 "'NOT
   FOUND" （未发现）。

本模块中可用的 HTTP 状态码常量可以参见 HTTP 状态码 。


HTTPConnection 对象
===================

"HTTPConnection" 实例拥有以下方法：

HTTPConnection.request(method, url, body=None, headers={}, *, encode_chunked=False)

   这会使用 HTTP 请求方法 *method* 和选择器 *url* 向服务器发送请求。

   如果给定 *body*，那么给定的数据会在信息头完成之后发送。它可能是一个
   "str" 、一个 *bytes-like object* 、一个打开的 *file object*，或者
   "bytes" 迭代器。如果 *body* 是字符串，它会按 HTTP 默认的 ISO-8859-1
   编码；如果是一个字节类对象，它会按原样发送；如果是 *file object* ，
   文件的内容会被发送，这个文件对象应该支持 "read()" 方法。如果这个文
   件对象是一个 "io.TextIOBase" 实例， "read()" 方法返回的数据会按
   ISO-8859-1 编码，否则 "read()" 方法返回的数据会按原样发送；如果
   *body* 是一个迭代器，迭代器中的元素会被发送，直到迭代器耗尽。

   *headers* 参数应是额外的随请求发送的 HTTP 信息头的字典。

   如果 *headers* 既不包含 Content-Length 也没有 Transfer-Encoding，但
   存在请求正文，那么这些头字段中的一个会自动设定。如果 *body* 是
   "None"，那么对于要求正文的方法 ("PUT"，"POST"，和 "PATCH")，
   Content-Length 头会被设为 "0"。如果 *body* 是字符串或者类似字节的对
   象，并且也不是 *文件*，Content-Length 头会设为正文的长度。任何其他
   类型的 *body* （一般是文件或迭代器）会按块编码，这时会自动设定
   Transfer-Encoding 头以代替 Content-Length。

   在 *headers* 中指定 Transfer-Encoding 时， *encode_chunked* 是唯一
   相关的参数。如果 *encode_chunked* 为 "False"，HTTPConnection 对象会
   假定所有的编码都由调用代码处理。如果为 "True"，正文会按块编码。

   注解:

     HTTP 协议在 1.1 版中添加了块传输编码。除非明确知道 HTTP 服务器可
     以处理 HTTP 1.1，调用者要么必须指定 Content-Length，要么必须传入
     "str" 或字节类对象，注意该对象不能是表达 body 的文件。

   3.2 新版功能: *body* 现在可以是可迭代对象了。

   在 3.6 版更改: 如果 Content-Length 和 Transfer-Encoding 都没有在
   *headers* 中设置，文件和可迭代的 *body* 对象现在会按块编码。添加了
   *encode_chunked* 参数。不会尝试去确定文件对象的 Content-Length。

HTTPConnection.getresponse()

   应当在发送一个请求从服务器获取响应时被调用。 返回一个
   "HTTPResponse" 的实例。

   注解:

     请注意你必须在读取了整个响应之后才能向服务器发送新的请求。

   在 3.5 版更改: 如果引发了 "ConnectionError" 或其子类，
   "HTTPConnection" 对象将在发送新的请求时准备好重新连接。

HTTPConnection.set_debuglevel(level)

   设置调试等级。 默认的调试等级为 "0"，意味着不会打印调试输出。 任何
   大于 "0" 的值将使得所有当前定义的调试输出被打印到 stdout。
   "debuglevel" 会被传给任何新创建的 "HTTPResponse" 对象。

   3.1 新版功能.

HTTPConnection.set_tunnel(host, port=None, headers=None)

   为 HTTP 连接隧道设置主机和端口。 这将允许通过代理服务器运行连接。

   host 和 port 参数指明隧道连接的位置（即 CONNECT 请求所包含的地址，
   而 *不是* 代理服务器的地址）。

   headers 参数应为一个随 CONNECT 请求发送的额外 HTTP 标头的映射。

   例如，要通过一个运行于本机 8080 端口的 HTTPS 代理服务器隧道，我们应
   当向 "HTTPSConnection" 构造器传入代理的地址，并将我们最终想要访问的
   主机地址传给 "set_tunnel()" 方法:

      >>> import http.client
      >>> conn = http.client.HTTPSConnection("localhost", 8080)
      >>> conn.set_tunnel("www.python.org")
      >>> conn.request("HEAD","/index.html")

   3.2 新版功能.

HTTPConnection.connect()

   当对象被创建后连接到指定的服务器。 默认情况下，如果客户端还未建立连
   接，此函数会在发送请求时自动被调用。

HTTPConnection.close()

   关闭到服务器的连接。

HTTPConnection.blocksize

   用于发送文件类消息体的缓冲区大小。

   3.7 新版功能.

作为对使用上述 "request()" 方法的替代同，你也可以通过使用下面的四个函
数，分步骤发送请的请求。

HTTPConnection.putrequest(method, url, skip_host=False, skip_accept_encoding=False)

   This should be the first call after the connection to the server
   has been made. It sends a line to the server consisting of the
   *method* string, the *url* string, and the HTTP version
   ("HTTP/1.1").  To disable automatic sending of "Host:" or "Accept-
   Encoding:" headers (for example to accept additional content
   encodings), specify *skip_host* or *skip_accept_encoding* with non-
   False values.

HTTPConnection.putheader(header, argument[, ...])

   Send an **RFC 822**-style header to the server.  It sends a line to
   the server consisting of the header, a colon and a space, and the
   first argument.  If more arguments are given, continuation lines
   are sent, each consisting of a tab and an argument.

HTTPConnection.endheaders(message_body=None, *, encode_chunked=False)

   Send a blank line to the server, signalling the end of the headers.
   The optional *message_body* argument can be used to pass a message
   body associated with the request.

   If *encode_chunked* is "True", the result of each iteration of
   *message_body* will be chunk-encoded as specified in **RFC 7230**,
   Section 3.3.1.  How the data is encoded is dependent on the type of
   *message_body*.  If *message_body* implements the buffer interface
   the encoding will result in a single chunk. If *message_body* is a
   "collections.abc.Iterable", each iteration of *message_body* will
   result in a chunk.  If *message_body* is a *file object*, each call
   to ".read()" will result in a chunk. The method automatically
   signals the end of the chunk-encoded data immediately after
   *message_body*.

   注解:

     Due to the chunked encoding specification, empty chunks yielded
     by an iterator body will be ignored by the chunk-encoder. This is
     to avoid premature termination of the read of the request by the
     target server due to malformed encoding.

   3.6 新版功能: Chunked encoding support.  The *encode_chunked*
   parameter was added.

HTTPConnection.send(data)

   Send data to the server.  This should be used directly only after
   the "endheaders()" method has been called and before
   "getresponse()" is called.


HTTPResponse 对象
=================

An "HTTPResponse" instance wraps the HTTP response from the server.
It provides access to the request headers and the entity body.  The
response is an iterable object and can be used in a with statement.

在 3.5 版更改: The "io.BufferedIOBase" interface is now implemented
and all of its reader operations are supported.

HTTPResponse.read([amt])

   Reads and returns the response body, or up to the next *amt* bytes.

HTTPResponse.readinto(b)

   Reads up to the next len(b) bytes of the response body into the
   buffer *b*. Returns the number of bytes read.

   3.3 新版功能.

HTTPResponse.getheader(name, default=None)

   Return the value of the header *name*, or *default* if there is no
   header matching *name*.  If there is more than one  header with the
   name *name*, return all of the values joined by ', '.  If 'default'
   is any iterable other than a single string, its elements are
   similarly returned joined by commas.

HTTPResponse.getheaders()

   Return a list of (header, value) tuples.

HTTPResponse.fileno()

   Return the "fileno" of the underlying socket.

HTTPResponse.msg

   A "http.client.HTTPMessage" instance containing the response
   headers.  "http.client.HTTPMessage" is a subclass of
   "email.message.Message".

HTTPResponse.version

   HTTP protocol version used by server.  10 for HTTP/1.0, 11 for
   HTTP/1.1.

HTTPResponse.status

   由服务器返回的状态码。

HTTPResponse.reason

   Reason phrase returned by server.

HTTPResponse.debuglevel

   A debugging hook.  If "debuglevel" is greater than zero, messages
   will be printed to stdout as the response is read and parsed.

HTTPResponse.closed

   Is "True" if the stream is closed.


示例
====

Here is an example session that uses the "GET" method:

   >>> import http.client
   >>> conn = http.client.HTTPSConnection("www.python.org")
   >>> conn.request("GET", "/")
   >>> r1 = conn.getresponse()
   >>> print(r1.status, r1.reason)
   200 OK
   >>> data1 = r1.read()  # This will return entire content.
   >>> # The following example demonstrates reading data in chunks.
   >>> conn.request("GET", "/")
   >>> r1 = conn.getresponse()
   >>> while chunk := r1.read(200):
   ...     print(repr(chunk))
   b'<!doctype html>\n<!--[if"...
   ...
   >>> # Example of an invalid request
   >>> conn = http.client.HTTPSConnection("docs.python.org")
   >>> conn.request("GET", "/parrot.spam")
   >>> r2 = conn.getresponse()
   >>> print(r2.status, r2.reason)
   404 Not Found
   >>> data2 = r2.read()
   >>> conn.close()

Here is an example session that uses the "HEAD" method.  Note that the
"HEAD" method never returns any data.

   >>> import http.client
   >>> conn = http.client.HTTPSConnection("www.python.org")
   >>> conn.request("HEAD", "/")
   >>> res = conn.getresponse()
   >>> print(res.status, res.reason)
   200 OK
   >>> data = res.read()
   >>> print(len(data))
   0
   >>> data == b''
   True

Here is an example session that shows how to "POST" requests:

   >>> import http.client, urllib.parse
   >>> params = urllib.parse.urlencode({'@number': 12524, '@type': 'issue', '@action': 'show'})
   >>> headers = {"Content-type": "application/x-www-form-urlencoded",
   ...            "Accept": "text/plain"}
   >>> conn = http.client.HTTPConnection("bugs.python.org")
   >>> conn.request("POST", "", params, headers)
   >>> response = conn.getresponse()
   >>> print(response.status, response.reason)
   302 Found
   >>> data = response.read()
   >>> data
   b'Redirecting to <a href="http://bugs.python.org/issue12524">http://bugs.python.org/issue12524</a>'
   >>> conn.close()

Client side "HTTP PUT" requests are very similar to "POST" requests.
The difference lies only the server side where HTTP server will allow
resources to be created via "PUT" request. It should be noted that
custom HTTP methods are also handled in "urllib.request.Request" by
setting the appropriate method attribute. Here is an example session
that shows how to send a "PUT" request using http.client:

   >>> # This creates an HTTP message
   >>> # with the content of BODY as the enclosed representation
   >>> # for the resource http://localhost:8080/file
   ...
   >>> import http.client
   >>> BODY = "***filecontents***"
   >>> conn = http.client.HTTPConnection("localhost", 8080)
   >>> conn.request("PUT", "/file", BODY)
   >>> response = conn.getresponse()
   >>> print(response.status, response.reason)
   200, OK


HTTPMessage Objects
===================

An "http.client.HTTPMessage" instance holds the headers from an HTTP
response.  It is implemented using the "email.message.Message" class.

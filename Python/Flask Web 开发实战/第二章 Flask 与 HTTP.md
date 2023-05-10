# Flask 与 HTTP

## 2.1 背景

1. 在第 1 章，我们已经了解了 Flask 的基本知识，如果想要进一步开发更复杂的 Flask 应用，我们就得了解 Flask 与 HTTP 协议的交互方式。

2. HTTP(Hypertext Transfer Protocol，超文本传输协议)定义了服务器和客户端之间信息交流的格式和传递方式，它是万维网(World Wide Web)中数据交换的基础。

3. 在这一章，我们会了解 Flask 处理请求和响应的各种方式，并对 HTTP 协议以及其他非常规 HTTP 请求进行简单的介绍。虽然本章的内容很重要，但鉴于内容有些晦涩难懂，如果感到困惑也不用担心，本章介绍的内容你会在后面的实践中逐渐理解和熟悉。如果你愿意，也可以临时跳过本章，等到学习完本书第一部分再回来重读。

4. HTTP 的详细定义在 RFC 7231~7235 中可以看到。RFC(Request For Comment，请求评议)是一系列关于互联网标准和信息的文件，可以将其理解为互联网(Internet)的设计文档。完整的 RFC 列表可以在这里看到：<https://tools.ietf.org/rfc/>

5. 本章的示例程序在 helloflask/demos/http 目录下，确保当前工作目录 在 helloflask/demos/http 下并激活了虚拟环境，然后执行 flask run 命令运行程序：

   ```bash
   cd demos/http
   flask run
   ```

## 2.2 请求响应循环

1. 为了更贴近现实，我们以一个真实的 URL 为例：

   - <http://helloflask.com/hello>

2. 当我们在浏览器中的地址栏中输入这个 URL ，然后按下 Enter 时，稍等片刻，浏览器会显示一个问候页面。这背后到底发生了什么？你一定可以猜想到，这背后也有一个类似我们第 1 章编写的程序运行着。

3. 它负责接收用户的请求，并把对应的内容返回给客户端，显示在用户的浏览器上。事实上，每一个 Web 应用都包含这种处理模式，即 "请求-响应循环(Request-Response Cycle)"：客户端发出请求，服务器端处理请求并返回响应

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/%E8%AF%B7%E6%B1%82%E5%93%8D%E5%BA%94%E5%BE%AA%E7%8E%AF%E7%A4%BA%E6%84%8F%E5%9B%BE.png" alt="请求响应循环示意图" style="zoom: 33%;" />

4. 客户端(Client Side)是指用来提供给用户的与服务器通信的各种软件。在本书中，客户端通常指 Web 浏览器(后面简称浏览器)，比如 Chrome、Firefox、IE 等；服务器端(Server Side)则指为用户提供服务的服务器，也是我们的程序运行的地方。

5. 这是每一个 Web 程序的基本工作模式，如果再进一步，这个模式又包含着更多的工作单元，如下图所示

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/Flask%20Web%20%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png" alt="Flask Web 工作流程" style="zoom: 33%;" />

6. 从上图中可以看出，HTTP 在整个流程中起到了至关重要的作用，它是客户端和服务器端之间沟通的桥梁。

7. 当用户访问一个 URL ，浏览器便生成对应的 HTTP 请求，经由互联网发送到对应的 Web 服务器。Web 服务器接收请求，通过 WSGI 将 HTTP 格式的请求数据转换成我们的 Flask 程序能够使用的 Python 数据。在程序中，Flask 根据请求的URL执行对应的视图函数，获取返回值生成响应。

8. 响应依次经过 WSGI 转换生成 HTTP 响应，再经由 Web 服务器传递，最终被发出请求的客户端接收。浏览器渲染响应中包含的 HTML 和 CSS 代码，并执行 JavaScript 代码，最终把解析后的页面呈现在用户浏览器的窗口中。

9. 这里的服务器指的是处理请求和响应的 Web 服务器，比如我们上一章介绍的开发服务器，而不是指物理层面上的服务器主机。

## 2.3 HTTP 请求

1. URL 是一个请求的起源。不论服务器是运行在美国洛杉矶，还是运行在我们自己的电脑上，当我们输入指向服务器所在地址的URL，都会向服务器发送一个 HTTP 请求。一个标准的 URL 由很多部分组成，以下面这个 URL 为例：

   - <http://helloflask.com/hello?name=Grey>

2. 这个 URL 的各个组成部分下表所示。

   <center><b>URL 组成说明</b></center>

   |      信息       |                       说明                       |
   | :-------------: | :----------------------------------------------: |
   |    <http://>    |           协议字符串，指定要使用的协议           |
   | helloflask.com  |                服务器的地址(域名)                |
   | hello?name=Grey | 要获取的资源路径(path)，类似 UNIX 的文件目录结构 |

3. 这个 URL 后面的 ?name=Grey 部分是查询字符串(query string)。URL 中的查询字符串用来向指定的资源传递参数。查询字符串从问号 ? 开始，以键值对的形式写出，多个键值对之间使用 & 分隔。

### 2.3.1 请求报文

1. 当我们在浏览器中访问这个 URL 时，随之产生的是一个发向 <http://helloflask.com> 所在服务器的请求。

2. 请求的实质是发送到服务器上的一些数据，这种浏览器与服务器之间交互的数据被称为报文(message)，请求时浏览器发送的数据被称为请求报文(request message)，而服务器返回的数据被称为响应报文(response message)。

3. 请求报文由请求的方法、URL、协议版本、首部字段(header)以及内容实体组成。前面的请求产生的请求报文示意下表所示。

   <center><b>请求报文示意表</b></center>

   |             组成说明              |                         请求报文内容                         |
   | :-------------------------------: | :----------------------------------------------------------: |
   | 报文首部：请求行(方法、URL、协议) |                     GET /hello HTTP/1.1                      |
   |      报文首部：各种首部字段       | Host: helloflask.com</br>Connection: keep-alive</br>Cache-Control: max-age=0</br>User-Agent: Mozilla/5.0(Window NT 6.1;Win64:x64) AppleWebKit/537.36(KHTML, like Gecko) chrome/59.0.3071.104 Safari/537.36</br> |
   |               空行                |                                                              |
   |             报文主体              |                          name=Grey                           |

4. 如果你想看真实的 HTTP 报文，可以在浏览器中向任意一个有效的 URL 发起请求，然后在浏览器的开发者工具(F12)里的 Network 标签中看到 URL 对应资源加载的所有请求列表，单击任一个请求条目即可看到报文信息。

5. 报文由报文首部和报文主体组成，两者由空行分隔，请求报文的主体一般为空。如果 URL 中包含查询字符串，或是提交了表单，那么报文主体将会是查询字符串和表单数据。

6. HTTP 通过方法来区分不同的请求类型。比如，当你直接访问一个页面时，请求的方法是 GET；当你在某个页面填写了表单并提交时，请求方法则通常为 POST。下表是常见的几种 HTTP 方法类型。

   <center><b>常见的 HTTP 方法</b></center>

   |  方法   |      说明      |
   | :-----: | :------------: |
   |   GET   |    获取资源    |
   |  POST   |    传输数据    |
   |   PUT   |    传输文件    |
   | DELETE  |    删除资源    |
   |  HEAD   |  获取报文首部  |
   | OPTIONS | 询问支持的方法 |

7. 报文首部包含了请求的各种信息和设置，比如客户端的类型、是否设置缓存、语言偏好等。

8. HTTP 中可用的首部字段列表可以在 <https://www.iana.org/assignments/message-headers/messageheaders.xhtml> 看到。请求方法的详细列表和说明可以在 RFC 7231(<https://tools.ietf.org/html/rfc7231>) 中看到。

9. 如果运行了示例程序，那么当你在浏览器中访问 <http://127.0.0.1:5000/hello> 时，开发服务器会在命令行中输出一条记录日志，其中包含请求的主要信息：

   - 127.0.0.1 - - [02/Aug/2017 09:51:37] "GET /hello HTTP/1.1" 200 –

### 2.3.2　Request 对象

1. 现在该让 Flask 的请求对象 request 出场了，这个请求对象封装了从客户端发来的请求报文，我们能从它获取请求报文中的所有数据。

2. 请求解析和响应封装实际上大部分是由 Werkzeug 完成的，Flask 子类化 Werkzeug 的请求(Request) 和响应(Response) 对象并添加了和程序相关的特定功能。在这里为了方便理解，我们先略过不谈。在第 16 章，我们会详细了解 Flask 的工作原理。

3. 和上一节一样，我们先从 URL 说起。假设请求的 URL 是 <http://helloflask.com/hello?name=Grey，当> Flask 接收到请求后，请求对象会提供多个属性来获取 URL 的各个部分

   <center><b>使用 request 的属性获取请求 URL</b></center>

   |   属性    |                    值                    |
   | :-------: | :--------------------------------------: |
   |   path    |                u"/hello"                 |
   | full_path |           u"/hello?name=Grey"            |
   |   host    |            u"helloflask.com"             |
   | host_url  |         u"http://helloflask.com"         |
   | base_url  |      u"http://helloflask.com/hello"      |
   |    url    | u"http://helloflask.com/hello?name=Grey" |
   | url_root  |        u"http://helloflask.com/"         |

4. 除了 URL ，请求报文中的其他信息都可以通过 request 对象提供的属性和方法获取，其中常用的部分下表。

   <center><b>request 对象常用的属性和方法</b></center>

   |                         属性/方法                          |                             说明                             |
   | :--------------------------------------------------------: | :----------------------------------------------------------: |
   |                            args                            | Werkzeug 的 immutableMultiDict 对象。存储解析后的查询字符串，可以通过字典方式获取键值。如果你想获取未解析的原生字符串，可以使用 query_string 属性 |
   |                         blueprint                          |                         当前蓝图名称                         |
   |                          cookies                           |           一个包含所有随请求提交的 cookies 的字典            |
   |                            data                            |                   包含字符串形式的请求数据                   |
   |                          endpoint                          |                   与当前请求相匹配的端点值                   |
   |                           files                            | Werkzeug 的 MutiDict 对象，包含所有上传文件，可以使用字典的形式获取文件。使用的键为 input 标签中的 name 属性值，对应的值为 Werkzeug 的 FileStorage 对象，可以调用 save() 方法并传入保存路径来保存文件 |
   |                            form                            | Werkzeug 的 immutableMultiDict 对象。与 files 类似，包含解析后的表单数据。表单字段值通过 input 标签的 name 属性值作为键获取 |
   |                           values                           | Werkzeug 的 CombinedMultiDict 对象，结合 args 和 form 属性值 |
   | get_data(cache=True, as_text=False, parse_from_data=False) | 获取请求中的数据，默认读取为字节字符串(bytestring)，将 as_text 设为 True 则返回值将是解码后的 unicode 字符串 |
   |   get_data(self, force=False, slient=False, cache=True)    | 作为 JSON 解析并返回数据，如果 MIME 类型不是 JSON，返回 None(除非 force 设为 True)；解析出错则抛出 Werkzeug 提供的 BadRequest 异常(如果未开启调试模式，则返回 400 错误响应)。如果 slient 设为 True 则返回 None；case 设置是否为缓存解析后的 JSON 数据 |
   |                          headers                           | 一个 Werkzeug 的 EnvironHeaders 对象，包含首部字段，可以以字典的形式操作 |
   |                          is_json                           |        通过 MIME 类型判断是否为 JSON 数据，返回布尔值        |
   |                            json                            | 包含解析后的 JSON 数据，内部调用 get_json()，可以通过字典的方式获取键值 |
   |                           method                           |                      请求的 HTTP 的方法                      |
   |                          referer                           |                  请求发起源 URL，即 referer                  |
   |                           scheme                           |                请求的 URL 模式(http 或 https)                |
   |                         user_agent                         | 用户代理(User Agent, UA)，包含用户的客户端类型，操作系统类型等信息 |

5. Werkzeug 的 MutliDict 类是字典的子类，它主要实现了同一个键对应多个值的情况。比如一个文件上传字段可能会接收多个文件。这时就可以通过 getlist() 方法来获取文件对象列表。

6. 而 ImmutableMultiDict 类继承了 MutliDict 类，但其值不可更改。更多内容可访问 Werkzeug 相关数据结构章节 <http://werkzeug.pocoo.org/docs/latest/datastructures/>

7. 在我们的示例程序中实现了同样的功能。当你访问 <http://localhost:5000/hello?name=Grey> 时，页面加载后会显示 "Hello， Grey！"。这说明处理这个 URL 的视图函数从查询字符串中获取了查询参数 name 的值，如下所示

   <center><b>获取请求 URL 中的查询字符串</b></center>

   ```python
   from flask import Flask, request

   app = Flask(__name__)
   @app.route('/hello')
   def hello():
       name = request.args.get('name', 'Flask')  # 获取查询参数 name 的值
       return '<h1>Hello, %s!<h1>' % name  # 插入到返回值中
   ```

8. 上面的示例代码包含安全漏洞，在现实中我们要避免直接将用户传入的数据直接作为响应返回，在本章的末尾我们将介绍这个安全漏洞的具体细节和防范措施。

9. 需要注意的是，和普通的字典类型不同，当我们从 request 对象的类型为 MutliDict 或 ImmutableMultiDict 的属性(比如 files、form、args)中直接使用键作为索引获取数据时(比如 request.args['name'])，如果没有对应的键，那么会返回 HTTP 400 错误响应(Bad Request，表示请求无效)，而不是抛出 KeyError 异常。

10. 为了避免这个错误，我们应该使用 get() 方法获取数据，如果没有对应的值则返回 None；get() 方法的第二个参数可以设置默认值，比如 request.args.get('name', 'Human')

11. 如果开启了调试模式，那么会抛出 BadRequestKeyError 异常并显示对应的错误堆栈信息，而不是常规的 400 响应。

### 2.3.3 在 Flask 中处理请求

1. URL 是指向网络上资源的地址。在 Flask 中，我们需要让请求的 URL 匹配对应的视图函数，视图函数返回值就是 URL 对应的资源。

#### 2.3.3.1 路由匹配

1. 为了便于将请求分发到对应的视图函数，程序实例中存储了一个路由表(app.url_map)，其中定义了 URL 规则和视图函数的映射关系。

2. 当请求发来后，Flask 会根据请求报文中的 URL(path 部分)来尝试与这个表中的所有 URL 规则进行匹配，调用匹配成功的视图函数。

3. 如果没有找到匹配的 URL 规则，说明程序中没有处理这个 URL 的视图函数，Flask 会自动返回 404 错误响应(Not Found，表示资源未找到)。你可以尝试在浏览器中访问 <http://localhost:5000/nothing> ，因为我们的程序中没有视图函数负责处理这个 URL ，所以你会得到 404 响应。

4. 如果你经常上网，那么肯定会对这个错误代码相当熟悉，它表示请求的资源没有找到。和前面提及的 400 错误响应一样，这类错误代码被称为 HTTP 状态码，用来表示响应的状态，具体会在下面详细讨论。

5. 当请求的 URL 与某个视图函数的 URL 规则匹配成功时，对应的视图函数就会被调用。使用 flask routes 命令可以查看程序中定义的所有路由，这个列表由 app.url_map 解析得到：

   ```bash
   $ flask routes
   Endpoint Methods   Rule
   -------- ------    ----------
   hello    GET     /hello
   go_back  GET     /goback/<int:age>
   hi       GET       /hi
   ...

   static GET         /static/<path:filename>
   ```

6. 在输出的文本中，我们可以看到每个路由对应的端点(Endpoint)、HTTP 方法(Methods) 和 URL 规则(Rule)，其中 static 端点是 Flask 添加的特殊路由，用来访问静态文件

#### 2.3.3.2 设置监听的 HTTP 方法

1. 在上一节通过 flask routes 命令打印出的路由列表可以看到，每一个路由除了包含 URL 规则外，还设置了监听的 HTTP 方法。

2. GET 是最常用的 HTTP 方法，所以视图函数默认监听的方法类型就是 GET，HEAD、 OPTIONS 方法的请求由 Flask 处理，而像 DELETE、PUT 等方法一般不会 在程序中实现，在后面我们构建 Web API 时才会用到这些方法。

3. 我们可以在 app.route() 装饰器中使用 methods 参数传入一个包含监听的 HTTP 方法的可迭代对象。比如，下面的视图函数同时监听 GET 请求和 POST 请求：

   ```python
   @app.route('/hello', methods=['GET', 'POST'])
   def hello():
       return '<h1>Hello, Flask!</h1>'
   ```

4. 当某个请求的方法不符合要求时，请求将无法被正常处理。比如，在提交表单时通常使用 POST 方法，而如果提交的目标URL对应的视图函数只允许 GET 方法，这时 Flask 会自动返回一个 405 错误响应(Method Not Allowed，表示请求方法不允许)

5. 通过定义方法列表，我们可以为同一个 URL 规则定义多个视图函数，分别处理不同 HTTP 方法的请求，这在本书第二部分构建 Web API 时会用到这个特性。

#### 2.3.3.3 URL 处理

1. 从前面的路由列表中可以看到，除了 /hello ，这个程序还包含许多 URL 规则，比如和 go_back 端点对应的 `/goback/<int:year>` 。

2. 现在请尝试访问 <http://localhost:5000/goback/34>，在 URL 中加入一个数字作为时光倒流的年数，你会发现加载后的页面中有通过传入的年数计算出的年 份："Welcome to 1984！"。

3. 仔细观察一下，你会发现 URL 规则中的变量部分有一些特别，`<int:year>` 表示为 year 变量添加了一个 int 转换器，Flask 在解析这个 URL 变量时会将其转换为整型。URL 中的变量部分默认类型为字符串，但 Flask 提供了一些转换器可以在 URL 规则里使用

   <center><b>Flask 内置的 URL 变量转换器</b></center>

   | 转换器 |                             说明                             |
   | :----: | :----------------------------------------------------------: |
   | string |                  不包含斜线的字符串(默认值)                  |
   |  int   |                             整型                             |
   | float  |                            浮点数                            |
   |  path  | 包含斜线的字符串。static 路由的 URL 规则中的 filename 变量使用了这个转换器 |
   |  any   |                 匹配一系列给定值中的一个元素                 |
   |  uuid  |                         UUID 字符串                          |

4. 转换器通过特定的规则指定，即 `<转换器:变量名>`。`<int:year>` 把 year 的值转换为整数，因此我们可以在视图函数中直接对 year 变量进行数学计算：

   ```python
   @app.route('goback/<int:year>')
   def go_back(year):
    return '<p>Welcome to %d!</p>' % (2018 - year)
   ```

5. 默认的行为不仅仅是转换变量类型，还包括 URL 匹配。在这个例子中，如果不使用转换器，默认 year 变量会被转换成字符串，为了能够在 Python 中计算天数，我们需要使用 int() 函数将 year 变量转换成整型。

6. 但是如果用户输入的是英文字母，就会出现转换错误，抛出 ValueError 异常，我们还需要手动验证；使用了转换器后，如果 URL 中传入的变量不是数字，那么会直接返回 404 错误响应。比如，你可以尝试访问 <http://localhost:5000/goback/tang>。

7. 在用法上唯一特别的是 any 转换器，你需要在转换器后添加括号来给出可选值，即 `<any (value1, value2, ...):变量名>`，比如：

   ```python
   @app.route('/colors/<any(blue, white, red):color>')
   def three_colors(color):
       return '<p>Love is patient and kind. Love is not jealous or boastful or %s </p>' % color
   ```

8. 当你在浏览器中访问 <http://localhost:5000/colors/> 时，如果将 `<color>` 部分替换为 any 转换器中设置的可选值以外的任意字符，均会获得 404 错误响应。

9. 如果你想在 any 转换器中传入一个预先定义的列表，可以通过格式化字符串的方式(使用 % 或是 format() 函数)来构建 URL 规则字符串，比如：

   ```python
   colors = ['blue', 'white', 'red']

   @app.route('/colors/<any(%s):color>' % str(colors)[1:-1])
   ```

#### 2.3.4 请求钩子

1. 有时我们需要对请求进行预处理(preprocessing) 和后处理 (postprocessing)，这时可以使用 Flask 提供的一些请求钩子(Hook)，它们可以用来注册在请求处理的不同阶段执行的处理函数(或称为回调函数，即 Callback)。

2. 这些请求钩子使用装饰器实现，通过程序实例 app 调用，用法很简单：以 before_request 钩子(请求之前)为例，当你对一个函数附加了 app.before_request 装饰器后，就会将这个函数注册为 before_request 处理函数，每次执行请求前都会触发所有 before_request 处理函数。

3. Flask 默认实现的五种请求钩子下表所示。

   <center><b>请求钩子</b></center>

   |         钩子         |                             说明                             |
   | :------------------: | :----------------------------------------------------------: |
   | before_first_request |             注册一个函数，在处理第一个请求前运行             |
   |    before_request    |              注册一个函数，在处理每个请求前运行              |
   |    after_request     | 注册一个函数，如果没有未处理的异常抛出，会在每个请求结束后运行 |
   |   teardown_request   | 注册一个函数，即使有未处理的异常抛出，会在每个请求结束后运行。如果发生异常，会传入异常对象作为参数到注册的函数中 |
   |  after_this_request  |       在视图函数内注册一个函数，会在这个请求结束后运行       |

4. 这些钩子使用起来和 app.route() 装饰器基本相同，每个钩子可以注册任意多个处理函数，函数名并不是必须和钩子名称相同，下面是一个基本示例：

   ```python
   @app.before_request
   def do_something():
       pass # 这里的代码会在每个请求处理前执行
   ```

5. 假如我们创建了三个视图函数 A、B、C，其中视图 C 使用了 after_this_request 钩子，那么当请求 A 进入后，整个请求处理周期的请求处理函数调用流程如下所示。

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E7%A4%BA%E6%84%8F%E5%9B%BE.png" alt="请求处理函数调用示意图" style="zoom: 33%;" />

6. 下面是请求钩子的一些常见应用场景：

   - before_first_request：在玩具程序中，运行程序前我们需要进行一些程序的初始化操作，比如创建数据库表，添加管理员用户。这些工作可以放到使用 before_first_request 装饰器注册的函数中。
   - before_request：比如网站上要记录用户最后在线的时间，可以通过用户最后发送的请求时间来实现。为了避免在每个视图函数都添加更新在线时间的代码，我们可以仅在使用 before_request 钩子注册的函数中调用这段代码。
   - after_request：我们经常在视图函数中进行数据库操作，比如更新、插入等，之后需要将更改提交到数据库中。提交更改的代码就可以放到 after_request 钩子注册的函数中。

7. 另一种常见的应用是建立数据库连接，通常会有多个视图函数需要 建立和关闭数据库连接，这些操作基本相同。

8. 一个理想的解决方法是在请求之前(before_request)建立连接，在请求之后(teardown_request)关闭连接。通过在使用相应的请求钩子注册的函数中添加代码就可以实现。这很像单元测试中的 setUp() 方法和 tearDown() 方法。

9. after_request 钩子和 after_this_request 钩子必须接收一个响应类对象作为参数，并且返回同一个或更新后的响应对象。

## 2.4 HTTP 响应

1. 在 Flask 程序中，客户端发出的请求触发相应的视图函数，获取返回值会作为响应的主体，最后生成完整的响应，即响应报文。

### 2.4.1 响应报文

1. 响应报文主要由协议版本、状态码(status code)、原因短语(reason phrase)、响应首部和响应主体组成。以发向 localhost:5000/hello 的请求为例，服务器生成的响应报文示意下表所示

   <center><b>响应报文</b></center>

   |                 组成说明                 |                         响应报文内容                         |
   | :--------------------------------------: | :----------------------------------------------------------: |
   | 报文首部：状态行(协议、状态码、原因短语) |                       HTTP/1.1 200 OK                        |
   |          报文首部：各种首部字段          | Content-Type:text/html;charset=utf-8<br>Content-Length:22<br>Server: Werkzeug/0.12.2 Python/2.7.13<br>Date: Thu, 03 Aug 2017 05:05:54 GMT<br>… |
   |                   空行                   |                                                              |
   |                 报文主体                 |                    <h1>Hello, Human!</h1>                    |

2. 响应报文的首部包含一些关于响应和服务器的信息，这些内容由 Flask 生成，而我们在视图函数中返回的内容即为响应报文中的主体内容。浏览器接收到响应后，会把返回的响应主体解析并显示在浏览器窗口上。

3. HTTP 状态码用来表示请求处理的结果，下表是常见的几种状态码和相应的原因短语。

   <center><b>常见的 HTTP 状态码</b></center>

   |    类型    | 状态码 | 原因短语(用于解释状态码) |                             说明                             |
   | :--------: | :----: | :----------------------: | :----------------------------------------------------------: |
   |    成功    |  200   |            Ok            |                        请求被正常处理                        |
   |    成功    |  201   |          Create          |                请求被处理，并创建了一个新资源                |
   |    成功    |  204   |       Not Content        |                  请求处理成功，但无内容返回                  |
   |   重定向   |  301   |    Moved Permanently     |                          永久重定向                          |
   |   重定向   |  302   |          Found           |                          临时重定向                          |
   |   重定向   |  304   |       Not Modified       |             请求资源未被修改，重定向到缓存的资源             |
   | 客户端错误 |  400   |       Bad Request        |              表示请求无效，即请求报文中存在错误              |
   | 客户端错误 |  401   |       Unauthoried        | 类似 403，表示请求的资源需要获取授权信息，在浏览器中会弹出认证弹窗 |
   | 客户端错误 |  403   |        Forbidden         |                表示请求的资源被服务器拒绝访问                |
   | 客户端错误 |  404   |        Not Found         |          表示服务器上无法找到请求的资源或 URL 无效           |
   | 服务端错误 |  500   |  Internal Server Error   |                      服务器内部发生错误                      |

4. 当关闭调试模式时，即 FLASK_ENV 使用默认值 production ，如果程序出错，Flask 会自动返回 500 错误响应；而调试模式下则会显示调试信息和错误堆栈。

5. 响应状态码的详细列表和说明可以在 RFC 7231(<https://tools.ietf.org/html/rfc7231>) 中看到。

### 2.4.2 在 Flask 中生成响应

1. 响应在 Flask 中使用 Response 对象表示，响应报文中的大部分内容由服务器处理，大多数情况下，我们只负责返回主体内容。

2. 根据我们在上一节介绍的内容，Flask 会先判断是否可以找到与请求 URL 相匹配的路由，如果没有则返回 404 响应。

3. 如果找到，则调用对应的视图函数，视图函数的返回值构成了响应报文的主体内容，正确返回时状态码默认为 200 。Flask 会调用 make_response() 方法将视图函数返 回值转换为响应对象。

4. 完整地说，视图函数可以返回最多由三个元素组成的元组：响应主体、状态码、首部字段。其中首部字段可以为字典，或是两元素元组组成的列表。

5. 比如，普通的响应可以只包含主体内容：

   ```python
   @app.route('/hello')
   def hello():
       ...
       return '<h1>Hello, Flask!</h1>'
   ```

6. 默认的状态码为 200 ，下面指定了不同的状态码：

   ```python
   @app.route('/hello')
   def hello():
       ...
       return '<h1>Hello, Flask!</h1>', 201
   ```

7. 有时你会想附加或修改某个首部字段。比如，要生成状态码为 3XX 的重定向响应，需要将首部中的 Location 字段设置为重定向的目标 URL：

   ```python
   @app.route('/hello')
   def hello():
       ...
       return '', 302, {'Location', 'http://www.example.com'}
   ```

8. 现在访问 <http://localhost:5000/hello，会重定向到> <http://www.example.com。在多数情况下，除了响应主体，其他部分我们通常只需要使用默认值即可>。

#### 2.4.2.1 重定向

1. 如果你访问 <http://localhost:5000/hi> ，你会发现页面加载后地址栏中的 URL 变为了 <http://localhost:5000/hello> 。

2. 这种行为被称为重定向(Redirect)，你可以理解为网页跳转。在上一节的示例中，状态码为 302 的重定向响应的主体为空，首部中需要将 Location 字段设为重定向的目标 URL ，浏览器接收到重定向响应后会向 Location 字段中的目标 URL 发起新的 GET 请求，整个流程如下图所示。

   <center><b>重定向流程示意图</b></center>

   <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/%E9%87%8D%E5%AE%9A%E5%90%91%E6%B5%81%E7%A8%8B%E7%A4%BA%E6%84%8F%E5%9B%BE.png" alt="重定向流程示意图" style="zoom:33%;" />

3. 在 Web 程序中，我们经常需要进行重定向。比如，当某个用户在没有经过认证的情况下访问需要登录后才能访问的资源，程序通常会重定向到登录页面。

4. 对于重定向这一类特殊响应，Flask 提供了一些辅助函数。除了像前面那样手动生成 302 响应，我们可以使用 Flask 提供的 redirect() 函数来生成重定向响应，重定向的目标 URL 作为第一个参数。前面的例子可以简化为：

   ```python
   from flask import Flask, redirect
   # ...

   @app.route('/hello')
   def hello():
       return redirect('http://www.example.com')
   ```

5. 使用 redirect() 函数时，默认的状态码为 302 ，即临时重定向。如果你想修改状态码，可以在 redirect() 函数中作为第二个参数或使用 code 关键字传入。

6. 如果要在程序内重定向到其他视图，那么只需在 redirect() 函数中使用 url_for() 函数生成目标URL即可，如下所示。

   ```python
   from flask import Flask, redirect, url_for
   ...

   @app.route('/hi')
   def hi():
       ...
       return redirect(url_for('hello')) # 重定向到/hello

   @app.route('/hello')
   def hello():
       ...
   ```

#### 2.4.2.2 错误响应

1. 如果你访问 <http://localhost:5000/brew/coffee>，会获得一个 418 错误响应(I'm a teapot)

2. 418 错误响应由 IETF(Internet Engineering Task Force，互联网工程任务组)在 1998 年愚人节发布的 HTCPCP(Hyper Text Coffee Pot Control Protocol，超文本咖啡壶控制协议)中定义(玩笑)，当一个控制茶壶的 HTCPCP 收到 BREW 或 POST 指令要求其煮咖啡时应当回传此错误。

3. 大多数情况下，Flask 会自动处理常见的错误响应。HTTP 错误对应的异常类在 Werkzeug 的 werkzeug.exceptions 模块中定义，抛出这些异常，即可返回对应的错误响应。如果你想手动返回错误响应，更方便的方法是使用 Flask 提供的 abort() 函数。

4. 在 abort() 函数中传入状态码即可返回对应的错误响应，下面中的视图函数返回 404 错误响应。

   ```python
   from flask import Flask, abort
   ...

   @app.route('/404')
   def not_found():
       abort(404)
   ```

5. abort() 函数前不需要使用 return 语句，但一旦 abort() 函数被调 用，abort() 函数之后的代码将不会被执行。

6. 虽然我们有必要返回正确的状态码，但这不是必须的。比如，当某个用户没有权限访问某个资源时，返回 404 错误要比 403 错误更加友好。

### 2.4.3 响应格式

1. 在 HTTP 响应中，数据可以通过多种格式传输。大多数情况下，我们会使用 HTML 格式，这也是 Flask 中的默认设置。在特定的情况下，我们也会使用其他格式

2. 不同的响应数据格式需要设置不同的 MIME 类型，MIME 类型在首部的 Content-Type 字段中定义，以默认的 HTML 类型为例：

   ```bash
   Content-Type: text/html; charset=utf-8
   ```

3. MIME 类型(又称为 media type 或 content type)是一种用来标识文件类型的机制，它与文件扩展名相对应，可以让客户端区分不同的内容类型，并执行不同的操作。

4. 一般的格式为 "类型名/子类型名"，其中的子类型名一般为文件扩展名。比如，HTML 的 MIME 类型为 "text/html"，png 图片的 MIME 类型为 "image/png" 。

5. 完整的标准 MIME 类型列表可以在这里看到：<https://www.iana.org/assignments/media-types/mediatypes.xhtml。>

6. 如果你想使用其他 MIME 类型，可以通过 Flask 提供的 make_response() 方法生成响应对象，传入响应的主体作为参数，然后使用响应对象的 mimetype 属性设置 MIME 类型，比如：

   ```python
   from flask import make_response

   @app.route('/foo')
   def foo():
       response = make_response('Hello, World!')
       response.mimetype = 'text/plain'
   	return response
   ```

7. 你也可以直接设置首部字段，比如 response.headers['ContentType']='text/xml；charset=utf-8'。但操作 mimetype 属性更加方便，而且不用设置字符集(charset)选项。

8. 常用的数据格式有纯文本、HTML、XML 和 JSON ，下面我们分别对这几种数据进行简单的介绍和分析。为了对不同的数据类型进行对比，我们将会用不同的数据类型来表示一个便签的内容：Jane 写给 Peter 的一个提醒。

#### 2.4.3.1 纯文本

1. MIME 类型：text/plain

2. 示例：

   ```text
   Note
   to: Peter
   from: Jane
   heading: Reminder
   body: Don't forget the party!
   ```

3. 事实上，其他几种格式本质上都是纯文本。比如同样是一行包含 HTML 标签的文本 `<h1>Hello，Flask！</h1>`，当 MIME 类型设置为纯文本时，浏览器会以文本形式显示 `<h1>Hello，Flask！</h1>`；当 MIME 类型声明为 text/html 时，浏览器则会将其作为标题 1 样式的 HTML 代码渲染。

#### 2.4.3.2 HTML

1. MIME 类型：text/html

2. 示例：

   ```html
   <!DOCTYPE html> <html> <head></head> <body>

   <h1>Note</h1>

   <p>to: Peter</p>

   <p>from: Jane</p>

   <p>heading: Reminder</p>

   <p>body: <strong>Don't forget the party!</strong></p> </body> </html>
   ```

3. [HTML](<https://www.w3.org/html/>)指 Hypertext Markup Language(超文本标记语言)，是最常用的数据格式，也是 Flask 返回响应的默认数据类型。从我们在本书一开始的最小程序中的视图函数返回的字符串，到我们后面会学习的 HTML 模板，都是 HTML 。当数据类型为 HTML 时，浏览器会自动根据 HTML 标签以及样式类定义渲染对应的样式。

4. 因为 HTML 常常包含丰富的信息，我们可以直接将 HTML 嵌入页面中，处理起来比较方便。因此，在普通的 HTTP 请求中我们使用 HTTP 作为响应的内容，这也是默认的数据类型。

#### 2.4.3.3 XML

1. MIME 类型：application/xml

2. 示例：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>

   <note> <to>Peter</to> <from>Jane</from> <heading>Reminder</heading> <body>Don't forget the party!</body> </note>
   ```

3. [XML](<https://www.w3.org/XML/>)指 Extensible Markup Language(可扩展标记语言)，它是一种简单灵活的文本格式，被设计用来存储和交换数据。

4. XML 的出现主要就是为了弥补 HTML 的不足：对于仅仅需要数据的请求来说，HTML 提供的信息太过丰富了，而且不易于重用。XML 和 HTML 一样都是标记性语言，使用标签来定义文本，但 HTML 中的标签用于显示内容，而 XML 中的标签只用于定义数据。

5. XML 一般作为 AJAX 请求的响应格式，或是 Web API 的响应格式。

#### 2.4.3.4 JSON

1. MIME 类型：application/json

2. 示例：

   ```json
   {
    "note":{ "to":"Peter", "from":"Jane", "heading":"Remider", "body":"Don't forget the party!" }
   }
   ```

3. [JSON](<http://json.org/>)指 JavaScript Object Notation(JavaScript 对象表示法)，是一种流行的、轻量的数据交换格式。

4. 它的出现又弥补了 XML 的诸多不足：XML 有较高的重用性，但 XML 相对于其他文档格式来说体积稍大，处理和解析的速度较慢。

5. JSON 轻量，简洁，容易阅读 和解析，而且能和 Web 默认的客户端语言 JavaScript 更好地兼容。JSON 的结构基于键值对的集合和有序的值列表，这两种数据结构类似 Python 中的字典(dictionary)和列表(list)。正是因为这种通用的数据结构，使得 JSON 在同样基于这些结构的编程语言之间交换成为可能。

6. 示例程序中提供了这一资源的不同格式响应，你可以访问 <http://localhost:5000/note/>，通过将 content_type 的值依次更改为 text、 html、xml 和 json 来获取不同格式的响应。比如，访问 <http://localhost:5000/note/text> 将得到纯文本格式的响应。

7. Flask 通过引入 Python 标准库中的 json 模块(或 simplejson，如果可用)为程序提供了 JSON 支持。

8. 你可以直接从 Flask 中导入 json 对象，然后调用 dumps() 方法将字典、列表或元组序列化(serialize) 为 JSON 字符串，再使用前面介绍的方法修改 MIME 类型，即可返回 JSON 响应，如下所示：

   ```python
   from flask import Flask, make_response, json
   ...

   @app.route('/foo')
   def foo():
       data = { 'name':'Grey Li', 'gender':'male' } response = make_response(json.dumps(data))
       response.mimetype = 'application/json'
       return response
   ```

9. 不过我们一般并不直接使用 json 模块的 dumps()、load() 等方法，因为 Flask 通过包装这些方法提供了更方便的 jsonify() 函数。

10. 借助 jsonify() 函数，我们仅需要传入数据或参数，它会对我们传入的参数进行序列化，转换成 JSON 字符串作为响应的主体，然后生成一个响应对象，并且设置正确的 MIME 类型。使用 jsonify 函数可以将前面的例子简化为这种形式：

    ```python
    from flask import jsonify

    @app.route('/foo')
    def foo():
    	return jsonify(name='Grey Li', gender='male')
    ```

11. jsonify() 函数接收多种形式的参数。你既可以传入普通参数，也可以传入关键字参数。如果你想要更直观一点，也可以像使用 dumps() 方法一样传入字典、列表或元组，比如：

    ```python
    from flask import jsonify

    @app.route('/foo')
    def foo():
    	return jsonify({name: 'Grey Li', gender: 'male'})
    ```

12. 上面两种形式的返回值是相同的，都会生成下面的 JSON 字符串：

    ```json
    {"gender": "male", "name": "Grey Li"}
    ```

13. 另外，jsonify() 函数默认生成 200 响应，你也可以通过附加状态码来自定义响应类型，比如：

    ```python
    @app.route('/foo')
    def foo():
        return jsonify(message='Error!'), 500
    ```

14. Flask 在获取请求中的 JSON 数据上也有很方便的解决方案，具体可以参考我们在 Request 对象小节介绍的 request.get_json() 方法和 request.json 属性。

### 2.4.4 来一块 Cookies

1. HTTP 是无状态(stateless) 协议。也就是说，在一次请求响应结束后，服务器不会留下任何关于对方状态的信息。

2. 但是对于某些 Web 程序来说，客户端的某些信息又必须被记住，比如用户的登录状态，这样才可以根据用户的状态来返回不同的响应。为了解决这类问题，就有了 Cookie 技术。Cookie 技术通过在请求和响应报文中添加 Cookie 数据来保存客户端的状态信息

3. Cookie 指 Web 服务器为了存储某些数据(比如用户信息)而保存在浏览器上的小型文本数据。浏览器会在一定时间内保存它，并在下一次向同一个服务器发送请求时附带这些数据。

4. Cookie 通常被用来进行用户会话管理(比如登录状态)，保存用户的个性化信息(比如语言偏好，视频上次播放的位置，网站主题选项等)以及记录和收集用户浏览数据以用来分析用户行为等。

5. 在 Flask 中，如果想要在响应中添加一个 cookie，最方便的方法是使用 Response 类提供的 set_cookie() 方法。要使用这个方法，我们需要先使用 make_response() 方法手动生成一个响应对象，传入响应主体作为参数。这个响应对象默认实例化内置的 Response 类。

6. 下表是内置的 Response 类常用的属性和方法。

   <center><b>Response 类的常用属性和方法</b></center>

   |  方法/属性   |                             说明                             |
   | :----------: | :----------------------------------------------------------: |
   |   headers    | 一个 Werkzeug 的 Headers 对象，表示响应首部，可以像字典一样操作 |
   |    status    |                       状态码，文本类型                       |
   | status_code  |                         状态码，整型                         |
   |   minetype   |                MIME 类型(仅包含内容类型部分)                 |
   | set_cookie() |                     用来设置一个 cookie                      |

7. 除了上表中列出的方法和属性外，Response 类同样拥有和 Request 类相同的 get_json()方法、is_json() 方法以及 json 属性。

8. set_cookie() 方法支持多个参数来设置 Cookie 的选项，如下表所示。

   <center><b>set_cookie() 方法的参数</b></center>

   |   属性   |                             说明                             |
   | :------: | :----------------------------------------------------------: |
   |   key    |                         cookie 的键                          |
   |  value   |                         cookie 的值                          |
   | max_age  | cookie 被保存的时间数，单位为秒；默认在用户会话结束(即关闭浏览器)时过期 |
   | expires  |       具体的过期时间，一个 datetime 对象或 UNIX 时间戳       |
   |   path   |        限制 cookie 只在给定的路径可用，默认为整个域名        |
   |  domain  |                    设置 cookie 可用的域名                    |
   |  secure  |          如果设置为 True，只用通过 HTTPS 才可以使用          |
   | httponly |      如果设置为 True，禁止客户端 JavaScript 获取 cookie      |

9. set_cookie 视图用来设置 cookie，它会将 URL 中的 name 变量的值设置到名为 name 的 cookie 里，如下所示。

   <center><b>设置 cookie</b></center>

   ```python
   from flask import Flask, make_response
   ...

   @app.route('/set/<name>')
   def set_cookie(name):
    response = make_response(redirect(url_for('hello')))
       response.set_cookie('name', name)
       return response
   ```

10. 在这个 make_response() 函数中，我们传入的是使用 redirect() 函数生成的重定向响应。set_cookie 视图会在生成的响应报文首部中创建一个 Set-Cookie 字段，即 "Set-Cookie：name=Grey；Path=/" 。

11. 现在我们查看浏览器中的 Cookie，就会看到多了一块名为 name 的 cookie ，其值为我们设置的 "Grey"。因为过期时间使用默认值，所以会在浏览会话结束时(关闭浏览器)过期。

12. 当浏览器保存了服务器端设置的 Cookie 后，浏览器再次发送到该服务器的请求会自动携带设置的 Cookie 信息，Cookie 的内容存储在请求首部的 Cookie 字段中

    <img src="https://studentcwz-pic-bed.oss-cn-guangzhou.aliyuncs.com/img/cookie%20%E8%AE%BE%E7%BD%AE%E7%A4%BA%E6%84%8F%E5%9B%BE.png" alt="cookie 设置示意图" style="zoom:33%;" />

13. 在 Flask 中，Cookie 可以通过请求对象的 cookies 属性读取。在修改后的 hello 视图中，如果没有从查询参数中获取到 name 的值，就从 Cookie 中寻找：

    ```python
    from flask import Flask, request

    @app.route('/')
    @app.route('/hello')
    def hello():
        name = request.args.get('name')
        if name is None:
            name = request.cookies.get('name', 'Human') # 从 Cookie 中获取 name 值
        return '<h1>Hello, %s</h1>' % name
    ```

14. 这个示例函数同样包含安全漏洞，后面会详细介绍。

15. 这时服务器就可以根据 Cookie 的内容来获得客户端的状态信息，并根据状态返回不同的响应。如果你访问 <http://localhost:5000/set/Grey>，那么就会将名为 name 的 cookie 设为 Grey ，重定向到 /hello 后，你会发现返回的内容变成了 "Hello，Grey！" 。如果你再次通过访问 <http://localhost:5000/set/> 修改 name cookie 的值，那么重定向后的页面返回的内容也会随之改变。

### 2.4.5 session: 安全的 Cookie

1. Cookie 在 Web 程序中发挥了很大的作用，其中最重要的功能是存储用户的认证信息。我们先来看看基于浏览器的用户认证是如何实现的。
2. 当我们使用浏览器登录某个社交网站时，会在登录表单中填写用户名和密码，单击登录按钮后，这会向服务器发送一个包含认证数据的请求。服务器接收请求后会查找对应的账户，然后验证密码是否匹配，如果匹配，就在返回的响应中设置一个 cookie ，比如，"login_user: greyli"。
3. 响应被浏览器接收后，cookie 会被保存在浏览器中。当用户再次向这个服务器发送请求时，根据请求附带的 Cookie 字段中的内容，服务器上的程序就可以判断用户的认证状态，并识别出用户。
4. 但是这会带来一个问题，在浏览器中手动添加和修改 Cookie 是很容易的事，仅仅通过浏览器插件就可以实现。所以，如果直接把认证信息以明文的方式存储在 Cookie 里，那么恶意用户就可以通过伪造 cookie 的内容来获得对网站的权限，冒用别人的账户。
5. 为了避免这个问题，我们需要对敏感的 Cookie 内容进行加密。方便的是，Flask 提供了 session 对象用来将 Cookie 数据加密储存。
6. 在编程中，session 指用户会话(user session)，又称为对话(dialogue)，即服务器和客户端/浏览器之间或桌面程序和用户之间建立的交互活动。
7. 在 Flask 中，session 对象用来加密 Cookie。默认情况下，它会把数据存储在浏览器上一个名为 session 的 cookie 里。

#### 2.4.5.1 设置程序密钥

1. session 通过密钥对数据进行签名以加密数据，因此，我们得先设置一个密钥。这里的密钥就是一个具有一定复杂度和随机性的字符串，比如 "Drmhze6EPcv0fN_81Bj-nA" 。

2. 程序的密钥可以通过 Flask.secret_key 属性或配置变量 SECRET_KEY 设置，比如：

   ```python
   app.secret_key = 'secret string'
   ```

3. 更安全的做法是把密钥写进系统环境变量(在命令行中使用 export 或 set 命令)，或是保存在 .env 文件中：

   ```text
   SECRET_KEY=secret string
   ```

4. 然后在程序脚本中使用 os 模块提供的 getenv() 方法获取：

   ```python
   import os
   # ...

   app.secret_key = os.getenv('SECRET_KEY', 'secret string')
   ```

5. 我们可以在 getenv() 方法中添加第二个参数，作为没有获取到对应环境变量时使用的默认值。

6. 这里的密钥只是示例。在生产环境中，为了安全考虑，你必须使用随机生成的密钥，在第 14 章我们会介绍如何生成随机密钥值。在本书中或相关示例程序中，为了方便会使用诸如 secret string、dev key 之类的占位文字。

#### 2.4.5.2 模拟用户认证

1. 下面我们会使用 session 模拟用户的认证功能。下面代码是用来登入用户的 login 视图。

   <center><b>登入用户</b></center>

   ```python
   from flask import redirect, session, url_for

   @app.route('/login')
   def login():
       session['logged_in'] = True # 写入 session
       return redirect(url_for('hello'))
   ```

2. 这个登录视图只是简化的示例，在实际的登录中，我们需要在页面上提供登录表单，供用户填写账户和密码，然后在登录视图里验证账户和密码的有效性。

3. session 对象可以像字典一样操作，我们向 session 中添 加一个 logged-in cookie ，将它的值设为 True ，表示用户已认证。

4. 当我们使用 session 对象添加 cookie 时，数据会使用程序的密钥对其进行签名，加密后的数据存储在一块名为 session 的 cookie 里，如下图所示。

5. 你可以在网页内的 Content 部分看到对应的加密处理后生成的 session 值。使用 session 对象存储的 Cookie ，用户可以看到其加密后的值，但无法修改它。

6. 因为 session 中的内容使用密钥进行签名，一旦数据被修改，签名的值也会变化。这样在读取时，就会验证失败，对应的 session 值也会随之失效。所以，除非用户知道密钥，否则无法对 session cookie 的值进行修改。

7. 当支持用户登录后，我们就可以根据用户的认证状态分别显示不同的内容。在 login 视图的最后，我们将程序重定向到 hello 视图，下面是修改后的 hello 视图：

   ```python
   from flask import request, session

   @app.route('/')
   @app.route('/hello')
   def hello():
       name = request.args.get('name')
       if name is None:
           name = request.cookies.get('name', 'Human')
           response = '<h1>Hello, %s!</h1>' % name
           # 根据用户认证状态返回不同的内容
           if 'logged_in' in session:
               response += '[Authenticated]'
           else:
               response += '[Not Authenticated]'
           return response
   ```

8. session 中的数据可以像字典一样通过键读取，或是使用 get() 方法。这里我们只是判断 session 中是否包含 logged_in 键，如果有则表示用户已经登录。通过判断用户的认证状态，我们在返回的响应中添加一行表示认证状态的信息：如果用户已经登录，显示 [Authenticated] ；否则显示 [Not authenticated]。

9. 如果你访问 <http://localhost:5000/login> ，就会登入当前用户，重定向到 <http://localhost:5000/hello> 后你会发现加载后的页面显示一行 "[Authenticated]"，表示当前用户已经通过认证。

10. 程序中的某些资源仅提供给登入的用户，比如管理后台，这时我们就可以通过判断 session 是否存在 logged-in 键来判断用户是否认证，下面代码是模拟管理后台的 admin 视图。

    <center><b>模拟管理后台</b></center>

    ```python
    from flask import session, abort

    @app.route('/admin')
    def admin():
        if 'logged_in' not in session:
        	abort(403)
        return 'Welcome to admin page.'
    ```

11. 通过判断 logged_in 是否在 session 中，我们可以实现：如果用户已经认证，会返回一行提示文字，否则会返回 403 错误响应。

12. 登出用户的 logout 视图也非常简单，登出账户对应的实际操作其实 就是把代表用户认证的 logged-in cookie 删除，这通过 session 对象的 pop 方法实现，如下面所示。

    <center><b>登出用户</b></center>

    ```python
    from flask import session

    @app.route('/logout')
    def logout():
    	if 'logged_in' in session:
            session.pop('logged_in')
        return redirect(url_for('hello'))
    ```

13. 现在访问 <http://localhost:5000/logout> 则会登出用户，重定向后的 /hello 页面的认证状态信息会变为 [Not authenticated]。

14. 默认情况下，session cookie 会在用户关闭浏览器时删除。通过将 session.permanent 属性设为 True 可以将 session 的有效期延长为 Flask.permanent_session_lifetime 属性值对应的 datetime.timedelta 对象，也可通过配置变量 PERMANENT_SESSION_LIFETIME 设置，默认为 31 天。

15. 尽管 session 对象会对 Cookie 进行签名并加密，但这种方式仅能够确保 session 的内容不会被篡改，加密后的数据借助工具仍然可以轻易读取(即使不知道密钥)。因此，绝对不能在 session 中存储敏感信息，比如用户密码。

## 2.5 Flask 上下文

1. 我们可以把编程中的上下文理解为当前环境(environment)的快照(snapshot)。如果把一个 Flask 程序比作一条可怜的生活在鱼缸里的鱼的话，那么它当然离不开身边的环境。
2. 这里的上下文和阅读文章时的上下文基本相同。如果在某篇文章里单独抽出一句话来看，我们可能会觉得摸不着头脑，只有联系上下文后我们才能正确理解文章。
3. Flask 中有两种上下文，程序上下文(application context)和请求上下文(request context)。如果鱼想要存活，水是必不可少的元素。
4. 对于 Flask 程序来说，程序上下文就是我们的水。水里包含了各种浮游生物以及微生物，正如程序上下文中存储了程序运行所必须的信息；要想健康地活下去，鱼还离不开阳光。射进鱼缸的阳光就像是我们的程序接收的请求。
5. 当客户端发来请求时，请求上下文就登场了。请求上下文里包含了请求的各种信息，比如请求的 URL ，请求的 HTTP 方法等。

### 2.5.1 上下文全局变量

1. 每一个视图函数都需要上下文信息，在前面我们学习过 Flask 将请求报文封装在 request 对象中。按照一般的思路，如果我们要在视图函数中使用它，就得把它作为参数传入视图函数，就像我们接收 URL 变量一 样。但是这样一来就会导致大量的重复，而且增加了视图函数的复杂度。

2. 在前面的示例中，我们并没有传递这个参数，而是直接从 Flask 导入一个全局的 request 对象，然后在视图函数里直接调用 request 的属性获取数据。你一定好奇，我们在全局导入时 request 只是一个普通的 Python 对象，为什么在处理请求时，视图函数里的 request 就会自动包含对应请求的数据？

3. 这是因为 Flask 会在每个请求产生后自动激活当前请求的上下文，激活请求上下文后，request 被临时设为全局可访问。而当每个请求结束后，Flask 就销毁对应的请求上下文。

4. 我们在前面说 request 是全局对象，但这里的全局并不是实际意义上的全局。我们可以把这些变量理解为动态的全局变量。

5. 在多线程服务器中，在同一时间可能会有多个请求在处理。假设有三个客户端同时向服务器发送请求，这时每个请求都有各自不同的请求报文，所以请求对象也必然是不同的。因此，请求对象只在各自的线程内是全局的。

6. Flask 通过本地线程(thread local)技术将请求对象在特定的线程和请求中全局可访问。具体内容和应用我们会在后面进行详细介绍。

7. 为了方便获取这两种上下文环境中存储的信息，Flask 提供了四个上下文全局变量，下表所示。

   <center><b>Flask 中的上下文变量</b></center>

   |   变量名    | 上下文类别 |                             说明                             |
   | :---------: | :--------: | :----------------------------------------------------------: |
   | current_app | 程序上下文 |                  指向处理请求的当前程序实例                  |
   |      g      | 程序上下文 | 替代 Python 全局变量的用法，确保仅在当前请求中可用。用于存储全局数据，每次请求都会重设 |
   |   request   | 请求上下文 |                 封装客户端发出的请求报文数据                 |
   |   session   | 请求上下文 |        用于记住请求之间的数据，通过签名的 Cookie 实现        |

8. 我们在前面对 session 和 request 都了解得差不多了，这里简单介绍一下 current_app 和 g 。

9. 你在这里也许会疑惑，既然有了程序实例 app 对象，为什么还需要 current_app 变量。在不同的视图函数中，request 对象都表示和视图函数对应的请求，也就是当前请求(current request)。

10. 而程序也会有多个程序实例的情况，为了能获取对应的程序实例，而不是固定的某一个程序实例，我们就需要使用 current_app 变量，后面会详细介绍。

11. 因为 g 存储在程序上下文中，而程序上下文会随着每一个请求的进入而激活，随着每一个请求的处理完毕而销毁，所以每次请求都会重设这个值。我们通常会使用它结合请求钩子来保存每个请求处理前所需要的全局变量，比如当前登入的用户对象，数据库连接等。

12. 在前面的示例中，我们在 hello 视图中从查询字符串获取 name 的值，如果每一个视图都需要这个值，那么就要在每个视图重复这行代码。借助 g 我们可以将这个操作移动到 before_request 处理函数中执行，然后保存到 g 的任意属性上：

    ```python
    from flask import g

    @app.before_request
    def get_name():
        g.name = request.args.get('name')
    ```

13. 设置这个函数后，在其他视图中可以直接使用 g.name 获取对应的值。另外，g 也支持使用类似字典的 get()、pop() 以及 setdefault() 方法进行操作。

### 2.5.2 激活上下文

1. 阳光柔和，鱼儿在水里欢快地游动，这一切都是上下文存在后的美好景象。如果没有上下文，我们的程序只能直挺挺地躺在鱼缸里。在下面这些情况下，Flask 会自动帮我们激活程序上下文：

   - 当我们使用 flask run 命令启动程序时。
   - 使用旧的 app.run() 方法启动程序时。

   - 执行使用 @app.cli.command() 装饰器注册的 flask 命令时。

   - 使用 flask shell 命令启动 Python Shell 时。

2. 当请求进入时，Flask 会自动激活请求上下文，这时我们可以使用 request 和 session 变量。另外，当请求上下文被激活时，程序上下文也被自动激活。当请求处理完毕后，请求上下文和程序上下文也会自动销毁。也就是说，在请求处理时这两者拥有相同的生命周期。

3. 结合 Python 的代码执行机制理解，这也就意味着，我们可以在视图函数中或在视图函数内调用的函数/方法中使用所有上下文全局变量。

4. 在使用 flask shell 命令打开的 Python Shell 中，或是自定义的 flask 命令函数 中，我们可以使用 current_app 和 g 变量，也可以手动激活请求上下文来使用 request 和 session 。

5. 如果我们在没有激活相关上下文时使用这些变量，Flask 就会抛出 RuntimeError 异常："RuntimeError：Working outside of application context." 或是 "RuntimeError：Working outside of request context."。

6. 同样依赖于上下文的还有 url_for()、jsonify() 等函数，所以你也只能在视图函数中使用它们。其中 jsonify() 函数内部调用中使用了 current_app 变量，而 url_for() 则需要依赖请求上下文才可以正常运行。

7. 如果你需要在没有激活上下文的情况下使用这些变量，可以手动激活上下文。比如，下面是一个普通的 Python shell ，通过 python 命令打开。程序上下文对象使用 app.app_context() 获取，我们可以使用 with 语句执行上下文操作：

   ```bash
   >>> from app import app

   >>> from flask import current_app

   >>> with app.app_context():
   ... current_app.name
   'app'
   ```

8. 或是显式地使用 push() 方法推送(激活)上下文，在执行完相关操作时使用 pop() 方法销毁上下文：

   ```bash
   >>> from app import app

   >>> from flask import current_app

   >>> app_ctx = app.app_context()

   >>> app_ctx.push()

   >>> current_app.name
   'app'
   >>> app_ctx.pop()
   ```

9. 而请求上下文可以通过 test_request_context() 方法临时创建：

   ```bash
   >>> from app import app

   >>> from flask import request

   >>> with app.test_request_context('/hello'):
   ... request.method
   'GET'
   ```

10. 同样的，这里也可以使用 push() 和 pop() 方法显式地推送和销毁请求上下文。

### 2.5.3 上下文钩子

1. 在前面我们学习了请求生命周期中可以使用的几种钩子，Flask 也为上下文提供了一个 teardown_appcontext 钩子，使用它注册的回调函数会在程序上下文被销毁时调用，而且通常也会在请求上下文被销毁时调用。比如，你需要在每个请求处理结束后销毁数据库连接：

   ```python
   @app.teardown_appcontext
   def teardown_db(exception):
       ...
       db.close()
   ```

2. 使用 app.teardown_appcontext 装饰器注册的回调函数需要接收异常对象作为参数，当请求被正常处理时这个参数值将是 None ，这个函数的返回值将被忽略。

3. 上下文是 Flask 的重要话题，在这里我们也只是简单了解一下，在本书的第三部分，我们会详细了解上下文的实现原理。

## 2.6 HTTP 进阶实践

1. 在本书的第一部分，从本章开始，每一章的最后都会包含一个进阶实践部分，其中介绍的内容我们将会在第二部分的程序实例中使用到。在这一节，我们会接触到一些关于 HTTP 的进阶内容。

### 2.6.1 重定向回上一个页面

1. 在前面的示例程序中，我们使用 redirect() 函数生成重定向响应。 比如，在 login 视图中，登入用户后我们将用户重定向到 /hello 页面。

2. 在复杂的应用场景下，我们需要在用户访问某个 URL 后重定向到上一个页面。最常见的情况是，用户单击某个需要登录才能访问的链接，这时程序会重定向到登录页面，当用户登录后合理的行为是重定向到用户登录前浏览的页面，以便用户执行未完成的操作，而不是直接重定向到主页。

3. 在示例程序中，我们创建了两个视图函数 foo 和 bar，分别显示一个 Foo 页面和一个 Bar 页面，如下所示：

   ```python
   @app.route('/foo')
   def foo():
       return '<h1>Foo page</h1><a href="%s">Do something</a>' % url_for('do_something')

   @app.route('/bar')
   def bar():
       return '<h1>Bar page</h1><a href="%s">Do something </a>' % url_for('do_something')
   ```

4. 在这两个页面中，我们都添加了一个指向 do_something 视图的链接。这个 do_something 视图如下所示：

   ```python
   @app.route('/do_something')
   def do_something():
       # do something
       return redirect(url_for('hello'))
   ```

5. 我们希望这个视图在执行完相关操作后能够重定向回上一个页面， 而不是固定的 /hello 页面。也就是说，如果在 Foo 页面上单击链接，我们希望被重定向回 Foo 页面；如果在 Bar 页面上单击链接，我们则希望返回到 Bar 页面。这一节我们会借助这个例子来介绍这一功能的实现。

#### 2.6.1.1 获取上一个页面的 URL

1. 要重定向回上一个页面，最关键的是获取上一个页面的 URL。上一个页面的 URL 一般可以通过两种方式获取：

##### 2.6.1.1.1 HTTP referer

1. HTTP referer(起源为 referrer 在 HTTP 规范中的错误拼写)是一个用来记录请求发源地址的 HTTP 首部字段(HTTP_REFERER)，即访问来源。当用户在某个站点单击链接，浏览器向新链接所在的服务器发起请求，请求的数据中包含的 HTTP_REFERER 字段记录了用户所在的原站点 URL 。

2. 这个值通常会用来追踪用户，比如记录用户进入程序的外部站点，以此来更有针对性地进行营销。

3. 在 Flask 中，referer 的值可以通过请求对象的 referrer 属性获取，即 request.referrer(正确拼写形式)。现在，do_something 视图的返回值可以这样编写：

   ```python
   return redirect(request.referrer)
   ```

4. 但是在很多种情况下，referrer 字段会是空值，比如用户在浏览器的地址栏输入URL，或是用户出于保护隐私的考虑使用了防火墙软件或使用浏览器设置自动清除或修改了 referrer 字段。我们需要添加一个备选项：

   ```python
   return redirect(request.referrer or url_for('hello'))
   ```

##### 2.6.1.1.2 查询参数

1. 除了自动从 referrer 获取，另一种更常见的方式是在 URL 中手动加入包含当前页面 URL 的查询参数，这个查询参数一般命名为 next。比如，下面在 foo 和 bar 视图的返回值中的 URL 后添加 next 参数：

   ```python
   from flask import request

   @app.route('/foo')
   def foo():
       return '<h1>Foo page</h1><a href="%s">Do something and redirect</a>' % url_for('do_something')

   @app.route('/bar')
   def bar():
       return '<h1>Bar page</h1><a href="%s">Do something and redirect</a>' % url_for('do_something')
   ```

2. 在程序内部只需要使用相对 URL ，所以这里使用 request.full_path 获取当前页面的完整路径。在 do_something 视图中，我们获取这个 next 值，然后重定向到对应的路径：

   ```python
   return redirect(request.args.get('next'))
   ```

3. 用户在浏览器的地址栏直接访问时可以轻易地修改查询参数，为了避免 next 参数为空的情况，我们也要添加备选项，如果为空就重定向到 hello 视图：

   ```python
   return redirect(request.args.get('next', url_for('hello')))
   ```

4. 为了覆盖更全面，我们可以将这两种方式搭配起来一起使用：首先获取 next 参数，如果为空就尝试获取 referer ，如果仍然为空，那么就重定向到默认的 hello 视图。因为在不同视图执行这部分操作的代码完全相同，我们可以创建一个通用的 redirect_back() 函数，如下所示。

   <center><b>重定向回上一个页面</b></center>

   ```python
   def redirect_back(default='hello', **kwargs):
    for target in request.args.get('next'), request.referrer:
     if target:
      return redirect(target)
       return redirect(url_for(default, **kwargs))
   ```

5. 通过设置默认值，我们可以在 referer 和 next 为空的情况下重定向到默认的视图。在 do_something 视图中使用这个函数的示例如下所示：

   ```python
   @app.route('/do_something_and_redirect')
   def do_something():
       # do something
       return redirect_back()
   ```

#### 2.6.1.2 对 URL 进行安全验证

1. 虽然我们已经实现了重定向回上一个页面的功能，但安全问题不容小觑，鉴于 referer 和 next 容易被篡改的特性，如果我们不对这些值进行验证，则会形成开放重定向(Open Redirect)漏洞。

2. 以 URL 中的 next 参数为例，next 变量以查询字符串的方式写在 URL 里，因此任何人都可以发给某个用户一个包含 next 变量指向任何站点的 链接。举个简单的例子，如果你访问下面的 URL：

   - <http://localhost:5000/do-something?next=http://helloflask.com>

3. 程序会被重定向到 <http://helloflask.com>。也就是说，如果我们不验证 next 变量指向的URL地址是否属于我们的应用内，那么程序很容易就会被重定向到外部地址。你也许还不明白这其中会有什么危险，下面假设的情况也许会给你一个清晰的认识：

4. 假设我们的应用是一个银行业务系统(下面简称网站 A)，某个攻击者模仿我们的网站外观做了一个几乎一模一样的网站(下面简称网站 B)。接着，攻击者伪造了一封电子邮件，告诉用户网站 A 账户信息需要更新，然后向用户提供一个指向网站 A 登录页面的链接，但链接中包含一个重定向到网站 B 的 next 变量，比如：<http://exampleA.com/login?next=http://maliciousB.com>。当用户在 A 网站登录后，如果 A 网站重定向到 next 对应的 URL ，那么就会导致重定向到攻击者编写的 B 网站。因为 B 网站完全模仿 A 网站的外观，攻击者就可以在重定向后的B网站诱导用户输入敏感信息，比如银行卡号及密码。

5. 确保 URL 安全的关键就是判断 URL 是否属于程序内部， 在下面代码中，我们创建了一个 URL 验证函数 is_safe_url()，用来验证 next 变量值是否属于程序内部 URL 。

   ```python
   from urlparse import urlparse, urljoin
   from flask import request

   def is_safe_url(target):
       # Python3 需要从 urllib.parse 导入
       ref_url = urlparse(request.host_url)
       test_url = urlparse(urljoin(request.host_url, target))
       return test_url.scheme in ('http', 'https') and ref_url.netloc == test_url.netloc
   ```

6. 如果你使用 Python3，那么这里需要从 urllib.parse 模块导入 urlparse 和 urljoin 函数。

7. 这个函数接收目标 URL 作为参数，并通过 request.host_url 获取程序内的主机 URL，然后使用 urljoin() 函数将目标 URL 转换为绝对 URL 。

8. 接着，分别使用 urlparse 模块提供的 urlparse() 函数解析两个 URL，最后对目标 URL 的 URL 模式和主机地址进行验证，确保只有属于程序内部的 URL 才会被返回。在执行重定向回上一个页面的 redirect_back() 函数中，我们使用 is_safe_url() 验证 next 和 referer 的值：

   ```python
   def redirect_back(default='hello', **kwargs):
   for target in request.args.get('next'), request.referrer:
       if not target:
           continue
           if is_safe_url(target):
               return redirect(target)
       return redirect(url_for(default, **kwargs))
   ```

9. 关于开放重定向漏洞的更多信息可以访问 <https://www.owasp.org/index.php/Unvalidated_Redirects_and_Forwards_Cheat_Sheet> 了解。

### 2.6.2 使用 AJAX 技术发送异步请求

1. 在传统的 Web 应用中，程序的操作都是基于请求响应循环来实现的。每当页面状态需要变动，或是需要更新数据时，都伴随着一个发向服务器的请求。当服务器返回响应时，整个页面会重载，并渲染新页 面。
2. 这种模式会带来一些问题。首先，频繁更新页面会牺牲性能，浪费服务器资源，同时降低用户体验。另外，对于一些操作性很强的程序来说，重载页面会显得很不合理。
3. 比如我们做了一个 Web 计算器程序，所有的按钮和显示屏幕都很逼真，但当我们单击等于按钮时，要等到页面重新加载后才在显示屏幕上看到结果，这显然会严重影响用户体验。我们这一节要学习的 AJAX 技术可以完美地解决这些问题。

#### 2.6.2.1 认识 AJAX

1. AJAX 指异步 Javascript 和 XML(Asynchronous JavaScript And XML)，它不是编程语言或通信协议，而是一系列技术的组合体。
2. 简单来说，AJAX 基于 [XMLHttpRequest](<https://xhr.spec.whatwg.org/>) 让我们可以在不重载页面的情况下和服务器进行数据交换。加上 JavaScript 和 DOM(Document Object Model，文档对象模型)，我们就可以在接收到响应数据后局部更新页面。
3. 而 XML 指的则是数据的交互格式，也可以是纯文本(Plain Text)、HTML 或 JSON 。顺便说一句，XMLHttpRequest 不仅支持 HTTP 协议，还支持 FILE 和FTP 协议。

4. AJAX 也常被拼作 Ajax，但是为了和古希腊神话里的英雄 Ajax 区分开来，在本书中将使用全大写形式，即 AJAX。
5. 在 Web 程序中，很多加载数据的操作都可以在客户端使用 AJAX 实现。比如，当用户鼠标向下滚动到底部时在后台发送请求获取数据，然后插入文章；再比如，用户提交表单创建新的待办事项时，在后台将数据发送到服务器端，保存后将新的条目直接插入到页面上。
6. 在这种模式下，我们可以在客户端实现大部分页面逻辑，而服务器端则主要负责处理数据。这样可以避免每次请求都渲染整个页面，这不仅增强了用户体验，也降低了服务器的负载。AJAX 让 Web 程序也可以像桌面程序那样获得更流畅的反应和动态效果。总而言之，AJAX 让 Web 程序更像是程序，而非一堆使用链接和按钮连接起来的网页资源。
7. 以删除某个资源为例，在普通的程序中流程如下：
   - 当删除按钮被单击时会发送一个请求，页面变空白，在接收到响应前无法进行其他操作。
   - 服务器端接收请求，执行删除操作，返回包含整个页面的响应。
   - 客户端接收到响应，重载整个页面。
8. 使用 AJAX 技术时的流程如下：
   - 当单击删除按钮时，客户端在后台发送一个异步请求，页面不变，在接收响应前可以进行其他操作。
   - 服务器端接收请求后执行删除操作，返回提示消息或是无内容的 204 响应。
   - 客户端接收到响应，使用 JavaScript 更新页面，移除资源对应的页面元素。

#### 2.6.2.2 使用 jQuery 发送 AJAX 请求

1. jQuery 是 流行的 JavaScript 库，它包装了 JavaScript ，让我们通过更简单的方式编写 JavaScript 代码。对于 AJAX ，它提供了多个相关的方法， 使用它可以很方便地实现 AJAX 操作。更重要的是，jQuery 处理了不同浏览器的 AJAX 兼容问题，我们只需要编写一套代码，就可以在所有主流的浏览器正常运行。

2. 使用 jQuery 实现 AJAX 并不是必须的，你可以选择使用原生的 XMLHttpRequest、其他 JavaScript 框架内置的 AJAX 接口，或是使用更新的 [Fetch API](<https://fetch.spec.whatwg.org/>) 来发送异步请求。

3. 在示例程序中，我们将使用全局 jQuery 函数 ajax() 发送 AJAX 请求。ajax() 函数是底层函数，有丰富的自定义配置，支持的主要参数下表所示。

   <center><b>ajax() 函数支持的参数</b></center>

   |    参数     |                     参数值类型以及默认值                     |                             说明                             |
   | :---------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
   |     url     |                   字符串：默认为当前页地址                   |                          请求的地址                          |
   |    type     |                     字符串：默认为 "GET"                     |     请求的方式，即 HTTP 方法，比如 GET、POST、DELETE 等      |
   |    data     |                       字符串：无默认值                       |     发送到服务器的数据。会被 jQuery 自动转换为查询字符串     |
   |  dataType   |                字符串：默认由 jQuery 自动判断                | 期待服务器返回的数据类型，可用的值如下："xml"、"html"、"script"、"json"、"jsonp"、"text" |
   | contentType | 字符串：默认为 "application/x-www-form-urlencoded；charset=UTF-8" | 发送请求时使用的内容类型，即请求首部的 Content-Type 字段内容 |
   |  complete   |                        函数：无默认值                        |                   请求完成后调用的回调函数                   |
   |   success   |                        函数：无默认值                        |                   请求成功后调用的回调函数                   |
   |    error    |                        函数：无默认值                        |                   请求失败后调用的回调函数                   |

4. jQuery 还提供了其他快捷方法(shorthand method)：用于发送 GET 请求的 get() 方法和用于发送 POST 请求的 post() 方法，还有直接用于获取 json 数据的 getjson() 以及获取脚本的 getscript() 方法。这些方法都是基于 ajax() 方法实现的。

5. 在这里，为了便于理解，使用了底层的 ajax 方法。jQuery 中和 AJAX 相关的方法及其具体用法可以在这里看到：<http://api.jquery.com/category/ajax/。>

#### 2.6.2.3 返回局部数据

1. 对于处理 AJAX 请求的视图函数来说，我们不会返回完整的 HTML 响应，这时一般会返回局部数据，常见的三种类型如下所示：

##### 2.6.2.3.1 纯文本或局部 HTML 模板

1. 纯文本可以在 JavaScript 用来直接替换页面中的文本值，而局部 HTML 则可以直接到插入页面中，比如返回评论列表：

   ```python
   @app.route('/comments/<int:post_id>')
   def get_comments(post_id):
       ...
       return render_template('comments.html')
   ```

##### 2.6.2.3.2 JSON 数据

1. JSON 数据可以在 JavaScript 中直接操作：

   ```python
   @app.route('/profile/<int:user_id>')
   def get_profile(user_id):
       ...
       return jsonify(username=username, bio=bio)
   ```

2. 在 jQuery 中的 ajax() 方法的 success 回调中，响应主体中的 JSON 字符串会被解析为 JSON 对象，我们可以直接获取并进行操作。

##### 2.6.2.3.3 空值

1. 有些时候，程序中的某些接收 AJAX 请求的视图并不需要返回数据给客户端，比如用来删除文章的视图。这时我们可以直接返回空值，并将状态码指定为 204(表示无内容)，比如：

   ```python
   @app.route('/post/delete/<int:post_id>', methods=['DELETE'])
   def delete_post(post_id):
       ...
       return '', 204
   ```

##### 2.6.2.3.4 异步加载长文章示例

1. 在示例程序的对应页面中，我们将显示一篇很长的虚拟文章，文章正文下方有一个加载更多按钮，当加载按钮被单击时，会发送一个 AJAX 请求获取文章的更多内容并直接动态插入到文章下方。用来显示虚拟文章的 show_post 视图如下所示。

   ```python
   from jinja2.utils import generate_lorem_ipsum

   @app.route('/post')
   def show_post():
       post_body = generate_lorem_ipsum(n=2) # 生成两段随机文本

       return ''' <h1>A very long post</h1>

               <div class="body">%s</div>

               <button id="load">Load More</button>

               <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>

               <script type="text/javascript">

               $(function() {

               $('#load').click(function() { $.ajax({ url: '/more', type: 'get', success: function(data){ $('.body').append(data); } })

               // 目标URL // 请求方法 // 返回2XX响应后触发的回调函数 // 将返回的响应插入到页面中

               })

               }) </script>''' % post_body
   ```

2. 文章的随机正文通过 Jinja2 提供的 generate_lorem_ipsum() 函数生成，n 参数用来指定段落的数量，默认为 5，它会返回由随机字符组成的虚拟文章。

3. 文章下面添加了一个加载更多按钮。按钮下面是两个 `<script></script>` 代码块，第一个 script 从 CDN 加载 jQuery 资源。

4. 在第二个 script 标签中，我们在代码的最外层创建了一个`$(function)() {...})`函数，这个函数是常见的 `$(document).ready (function() {...})` 函数的简写形式。

5. 这个函数用来在页面 DOM 加载完毕后执行代码，类似传统 JavaScript 中的 window.onload 方法，所以我们通常会将代码包装在这个函数中。美元符号是 jQuery 的简写，我们通过它来调用 jQuery 提供的多个方法，所以 `$.ajax()` 等同于 `jQuery.ajax()`

6. 在 `$(function() {...})` 中，`$('#load')` 被称为选择器，我们在括号中传入目标元素的 id、class 或是其他属性来定位到对应的元素，将其创建为 jQuery 对象。

7. 我们传入了加载更多按钮的 id 值以定位到加载按钮。在这个选择器上，我们附加了 `.click(function() {...})`，这会为加载按钮注册一个单击事件处理函数，当加载按钮被单击时就会执行单击事件回调函数。

8. 在这个回调函数中，我们使用 `$.ajax()` 方法发送一个 AJAX 请求到服务器，通过 url 将目标 URL 设为 "/more" ，通过 type 参数 将请求的类型设为 GET 。

9. 当请求成功处理并返回 2XX 响应时(另外还包括 304 响应)，会触发 success 回调函数。success 回调函数接收的第一个参数为服务器端返回的响应主体，在这个回调函数中，我们在文章正文(通过 `$('.body')` 选择)底部使用 append() 方法插入返回的 data 数据。

10. 由于篇幅所限，我们不会深入介绍 JavaScript 或 jQuery ，你可以阅读其他书籍来学习更多内容。

11. 处理 /more 的视图函数会返回随机文章正文，如下所示：

    ```python
    @app.route('/more')
    def load_post():
        return generate_lorem_ipsum(n=1)
    ```

12. 如果你启动了示例程序，那么访问 <http://localhost:5000/post> 可以看到文章页面，当你单击文章下的 Load More 按钮时，浏览器就会在后台发送一个 GET 请求到 /more，这个视图返回的随机字符会被动态插入到文章下方。

### 2.6.3 HTTP 服务器端推送

1. 不论是传统的 HTTP 请求-响应式的通信模式，还是异步的 AJAX 式请求，服务器端始终处于被动的应答状态，只有在客户端发出请求的情况下，服务器端才会返回响应。

2. 这种通信模式被称为客户端拉取(client pull)。在这种模式下，用户只能通过刷新页面或主动单击加载按钮来拉取新数据。然而，在某些场景下，我们需要的通信模式是服务器端的主动推送(server push)。

3. 比如，一个聊天室有很多个用户，当某个用户发送消息后，服务器接收到这个请求，然后把消息推送给聊天室的所有用户。类似这种关注实时性的情况还有很多，比如社交网站在导航栏实时显示 新提醒和私信的数量，用户的在线状态更新，股价行情监控、显示商品库存信息、多人游戏、文档协作等。

4. 实现服务器端推送的一系列技术被合称为 HTTP Server Push(HTTP 服务器端推送)，目前常用的推送技术下表所示。

   <center><b>常用推送技术</b></center>

   |          名称           |                             说明                             |
   | :---------------------: | :----------------------------------------------------------: |
   |        传统轮询         | 在特定的时间时间间隔内，客户端使用 AJAX 技术不断向服务器发起 HTTP 请求，然后获取新的数据并更新页面 |
   |         长轮询          | 和传统轮询类似，但是如果服务器端没有返回数据，那就保持连接一直开启，直到有数据时才返回。取回数据后再次发送另一个请求 |
   | Server-Sent Events(SSE) | SSE 通过 HTML5 中的 EventSource API 实现。SSE 会在客户端和服务端建立一个单向的通道，客户端监听来自服务端的数据，而服务器端可以在任意时间发送数据，两者建立类似的订阅/发布的通信模式 |

5. 按照列出的顺序来说，这几种方式对实时通信的实现越来越完善。当然，每种技术都有各自的优缺点，在具体的选择上，要根据面向的用户群以及程序自身的特点来分析选择。这些技术我们会在本书第二部分的程序实例中逐一介绍。

6. 轮询(polling)这类使用 AJAX 技术模拟服务器端推送的方法实现起来比较简单，但通常会造成服务器资源上的浪费，增加服务器的负担，而且会让用户的设备耗费更多的电量(频繁地发起异步请求)。

7. SSE 效率更高，在浏览器的兼容性方面，除了 Windows IE/Edge，SSE 基本上支持所有主流浏览器，但浏览器通常会限制标签页的连接数量。

8. 除了这些推送技术，在 HTML5 的 API 中还包含了一个 WebSocket 协议，和 HTTP 不同，它是一种基于 TCP 协议的全双工通信协议(fullduplex communication protocol)。和前面介绍的服务器端推送技术相比，WebSocket 实时性更强，而且可以实现双向通信(bidirectional communication)。

9. 另外，WebSocket 的浏览器兼容性要强于 SSE。

### 2.6.4 Web 安全防范

1. 无论是简单的博客，还是大型的社交网站，Web 安全都应该放在首位。Web 安全问题涉及广泛，我们在这里介绍其中常见的几种攻击(attack)和其他常见的漏洞(vulnerability)。
2. 对于 Web 程序的安全问题，一个首要的原则是：永远不要相信你的用户。大部分 Web 安全问题都是因为没有对用户输入的内容进行“消毒”造成的。

#### 2.6.4.1 注入攻击

1. 在 OWASP(Open Web Application Security Project，开放式 Web 程 序安全项目)发布的最危险的 Web 程序安全风险 Top 10 中，无论是最新的 2017 年的排名，2013 年的排名还是最早的 2010 年，注入攻击(Injection)都位列第一。

2. 注入攻击包括系统命令(OS Command)注 入、SQL(Structured Query Language，结构化查询语言)注入(SQL Injection)、NoSQL 注入、ORM(Object Relational Mapper，对象关系映射)注入等。我们这里重点介绍的是 SQL 注入。

3. SQL 是一种功能齐全的数据库语言，也是关系型数据库的通用操作语言。使用它可以对数据库中的数据进行修改、查询、删除等操作； ORM 是用来操作数据库的工具，使用它可以在不手动编写 SQL 语句的情况下操作数据库。

4. 在编写 SQL 语句时，如果直接将用户传入的数据作为参数使用字符 串拼接的方式插入到 SQL 查询中，那么攻击者可以通过注入其他语句来执行攻击操作，这些攻击操作包括可以通过 SQL 语句做的任何事：获取 敏感数据、修改数据、删除数据库表……

5. 假设我们的程序是一个学生信息查询程序，其中的某个视图函数接收用户输入的密码，返回根据密码查询对应的数据。我们的数据库由一个 db 对象表示，SQL 语句通过 execute() 方法执行：

   ```python
   @app.route('/students')
   def bobby_table():
       password = request.args.get('password')
       cur = db.execute("SELECT * FROM students WHERE password='%s';" % password)
       results = cur.fetchall()
       return results
   ```

6. 在实际应用中，敏感数据需要通过表单提交的 POST 请求接收，这里为了便于演示，我们通过查询参数接收。

7. 我们通过查询字符串获取用户输入的查询参数，并且不经过任何处理就使用字符串格式化的方法拼接到 SQL 语句中。在这种情况下，如果攻击者输入的 password 参数值为 'or 1=1--'， 即 <http://example.com/students?password=>'' or 1 = 1--，那么最终视图函数中被执行的 SQL 语句将变为：

8. 这时会把 students 表中的所有记录全部查询并返回，也就意味着所有的记录都被攻击者窃取了。更可怕的是，如果攻击者将 password 参数值设为 '', 'drop table users；--;' 那么查询语句就会变成：

   ```sql
   SELECT * FROM students WHERE password=''; drop table students; --;
   ```

9. 执行这个语句会把 students 表中的所有记录全部删除掉。

10. 在 SQL 中，`;` 用来结束一行语句；`--` 用来注释后面的语句，类似 Python 中的 `#`

11. 主要防范方法

    - 使用 ORM 可以一定程度上避免 SQL 注入问题，我们将在第 5 章学习使用 ORM。

    - 验证输入类型。比如某个视图函数接收整型 id 来查询，那么就 在 URL 规则中限制 URL 变量为整型。

    - 参数化查询。在构造 SQL 语句时避免使用拼接字符串或字符串格式化(使用百分号或 format() 方法)的方式来构建 SQL 语句。而要 使用各类接口库提供的参数化查询方法，以内置的 sqlite3 库为例：

      ```python
      db.execute('SELECT * FROM students WHERE password=?, password)
      ```

    - 转义特殊字符，比如引号、分号和横线等。使用参数化查询时，各种接口库会为我们做转义工作。

12. XSS(Cross-Site Scripting，跨站脚本)攻击历史悠久，最远可以追溯到 90 年代，但至今仍然是危害范围非常广的攻击方式。在 OWASP TOP 10 中排名第 7

13. Cross-Site Scripting 的缩写本应是 CSS，但是为了避免和 Cascading Style Sheets 的缩写产生冲突，所以将 Cross(即交叉) 使用交叉形状的 X 表示。

##### 2.6.4.1.1 攻击原理

1. XSS 是注入攻击的一种，攻击者通过将代码注入被攻击者的网站中，用户一旦访问网页便会执行被注入的恶意脚本。XSS 攻击主要分为 反射型 XSS 攻击(Reflected XSS Attack) 和存储型 XSS 攻击（Stored XSS Attack）两类。

##### 2.6.4.1.2 攻击示例

1. 反射型 XSS 又称为非持久型 XSS(Non-Persistent XSS)。当某个站点存在 XSS 漏洞时，这种攻击会通过 URL 注入攻击脚本，只有当用户访问这个 URL 时才会执行攻击脚本。我们在本章前面介绍查询字符串和 cookie 时引入的示例就包含反射型 XSS 漏洞，如下所示：

   ```python
   @app.route('/hello')
   def hello():
       name = request.args.get('name')
       response = '<h1>Hello, %s!</h1>' % name
   ```

2. 这个视图函数接收用户通过查询字符串传入的数据，未做任何处理就把它直接插入到返回的响应主体中，返回给客户端。如果某个用户输入了一段 JavaScript 代码作为查询参数 name的 值，如下所示：

   ```html
   http://example.com/hello?name=<script>alert('Bingo!');</script>
   ```

3. 客户端接收的响应将变为下面的代码：

   ```html
   <h1>Hello, <script>alert('Bingo!');</script>!</h1>
   ```

4. 当客户端接收到响应后，浏览器解析这行代码就会打开一个弹窗，你觉得一个小弹窗不会造成什么危害？那你就完全错了，能够执行 alert() 函数就意味着通过这种方式可以执行任意 JavaScript 代码。

5. 即攻击者通过 JavaScript 几乎能够做任何事情：窃取用户的 cookie 和其他敏感数据，重定向到钓鱼网站，发送其他请求，执行诸如转账、发布广告信息、在社交网站关注某个用户等。

6. 即使不插入 JavaScript 代码，通过 HTML 和 CSS(CSS 注入)也可以影响页面正常的输出，篡改页面样式，插入图片等。

7. 如果网站 A 存在 XSS 漏洞，攻击者将包含攻击代码的链接发送给网站 A 的用户 Foo，当 Foo 访问这个链接就会执行攻击代码，从而受到攻击。

8. 存储型 XSS 也被称为持久型 XSS(persistent XSS)，这种类型的 XSS 攻击更常见，危害也更大。它和反射型 XSS 类似，不过会把攻击代码储存到数据库中，任何用户访问包含攻击代码的页面都会被殃及。

9. 比如，某个网站通过表单接收用户的留言，如果服务器接收数据后未经处理就存储到数据库中，那么用户可以在留言中插入任意 JavaScript 代码。比如，攻击者在留言中加入一行重定向代码：

   ```python
   <script>window.location.href="http://attacker.com";</script>
   ```

10. 其他任意用户一旦访问留言板页面，就会执行其中的 JavaScript 脚本。那么其他用户一旦访问这个页面就会被重定向到攻击者写入的站点。

### 2.6.4.1.3 主要防范措施

1. 防范 XSS 攻击最主要的方法是对用户输入的内容进行 HTML 转义， 转义后可以确保用户输入的内容在浏览器中作为文本显示，而不是作为代码解析。

2. 这里的转义和 Python 中的概念相同，即消除代码执行时的歧义，也 就是把变量标记的内容标记为文本，而不是 HTML 代码。具体来说，这 会把变量中与 HTML 相关的符号转换为安全字符，以避免变量中包含影 响页面输出的HTML标签或恶意的 JavaScript代 码

3. 比如，我们可以使用 Jinja2 提供的 escape() 函数对用户传入的数据进行转义：

   ```python
   from jinja2 import escape

   @app.route('/hello')
   def hello():
       name = request.args.get('name')
       response = '<h1>Hello, %s!</h1>' % escape(name)
   ```

4. 在 Jinja2 中，HTML 转义相关的功能通过 Flask 的依赖包 MarkupSafe 实现。

5. 调用 escape() 并传入用户输入的数据，可以获得转义后的内容，前面的示例中，用户输入的 JavaScript 代码将被转义为：

   ```python
   &lt;script&gt;alert(&#34;Bingo!&#34;)&lt;/script&gt;
   ```

6. 转义后，文本中的特殊字符(比如 ">" 和 "<" 都将被转义为 HTML 实体(character entity)，

7. 这行文本最终在浏览器中会被显示为文本形式的 `<script>alert('Bingo！')</script>` 。

8. 在 Python 中，如果你想在单引号标记的字符串中显示一个单引号，那么你需要在单引号前添加一个反斜线来转义它，也就是把它标记为普文本，而不是作为特殊字符解释。

9. 在 HTML 中，也存在许多保留的特殊字符，比如大于小于号。如果你想以文本显示这些字符，也需要对其进行转义，即使用 HTML 字符实体表示这些字符。HTML 实体就是一些用来表示保留符号的特殊文本，比如 &lt；表示小于号，&quot；表示双引号。

10. 一般我们不会在视图函数中直接构造返回的 HTML 响应，而是会使用 Jinja2 来渲染包含变量的模板，这部分内容我们将在第 3 章学习。

11. XSS 攻击可以在任何用户可定制内容的地方进行，例如图片引用、自定义链接。仅仅转义 HTML 中的特殊字符并不能完全规避 XSS 攻击，因为在某些 HTML 属性中，使用普通的字符也可以插入 JavaScript 代码。 除了转义用户输入外，我们还需要对用户的输入数据进行类型验证。在所有接收用户输入的地方做好验证工作。在第 4 章学习表单时，我们会详细介绍表单数据的验证。

12. 以某个程序的用户资料页面为例，我们来演示一下转义无法完全避免的 XSS 攻击。程序允许用户输入个人资料中的个人网站地址，通过下面的方式显示在资料页面中：

    ```html
    <a href="{{ url }}">Website</a>
    ```

13. 其中 {{ url }} 部分表示会被替换为用户输入的 url 变量值。如果不对 URL 进行验证，那么用户就可以写入 JavaScript 代码，比如 `javascript:alert('Bingo！')；`。因为这个值并不包含会被转义的。最终页面 上的链接代码会变为：

    ```html
    <a href="javascript:alert('Bingo!');">Website</a>
    ```

14. 当用户单击这个链接时，就会执行被注入的攻击代码。另外，程序还允许用户自己设置头像图片的 URL 。这个图片通过下面的方式显示：

    ```html
    <img src="{{ url }}">
    ```

15. 类似的，{{ url }} 部分表示会被替换为用户输入的 url 变量值。如果不对输入的 URL 进行验证，那么用户可以将 url 设为 "123"onerror="alert('Bingo！')"，最终的 `<img>` 标签就会变为：

    ```html
    <img src="123" onerror="alert('Bingo!')">
    ```

16. 在这里因为 src 中传入了一个错误的 URL ，浏览器便会执行 onerror 属性中设置的 JavaScript 代码。

17. 如果你想允许部分 HTML 标签，比如 `<b>` 和 `<i>` ，可以使用 HTML 过滤工具对用户输入的数据进行过滤，仅保留少量允许使用的 HTML 标签，同时还要注意过滤 HTML 标签的属性，我们会在本书的第二部分详细了解。

18. CSRF(Cross Site Request Forgery，跨站请求伪造)是一种近年来才逐渐被大众了解的网络攻击方式，又被称为 One-Click Attack 或 Session Riding 。在 OWASP 上一次(2013)的 TOP 10 Web 程序安全风险中，它位列第 8 。随着大部分程序的完善，各种框架都内置了对 CSRF 保护的支持，但目前仍有 5 %的程序受到威胁。

19. CSRF 攻击的大致方式如下：某用户登录了 A 网站，认证信息保存在 cookie 中。当用户访问攻击者创建的 B 网站时，攻击者通过在 B 网站发送一个伪造的请求提交到 A 网站服务器上，让 A 网站服务器误以为请求来自于自己的网站，于是执行相应的操作，该用户的信息便遭到了篡改。

20. 总结起来就是，攻击者利用用户在浏览器中保存的认证信息，向对应的站点发送伪造请求。在前面学习 cookie 时，我们介绍过用户认证通过保存在 cookie 中的数据实现。在发送请求时，只要浏览器中保存了对应的 cookie，服务器端就会认为用户已经处于登录状态，而攻击者正是利用了这一机制。为了更便于理解，下面我们举一个实例。

21. 假设我们网站是一个社交网站(example.com)，简称网站 A ；攻击者的网站可以是任意类型的网站，简称网站 B。在我们的网站中，删除账户的操作通过 GET 请求执行，由使用下面的 delete_account 视图处理：

    ```python
    @app.route('/account/delete')
    def delete_account():
        if not current_user.authenticated:
            abort(401)
        current_user.delete()
        return 'Deleted!'
    ```

22. 当用户登录后，只要访问 <http://example.com/account/delete> 就会删除账户。那么在攻击者的网站上，只需要创建一个显示图片的 img 标签， 其中的 src 属性加入删除账户的 URL：

    ```html
    <img src="http://example.com/account/delete">
    ```

23. 当用户访问 B 网站时，浏览器在解析网页时会自动向 img 标签的 src 属性中的地址发起请求。此时你在 A 网站的登录信息保存在 cookie 中， 因此，仅仅是访问 B 网站的页面就会让你的账户被删除掉。

24. 当然，现实中很少有网站会使用 GET 请求来执行包含数据更改的敏感操作，这里只是一个示例。现在，假设我们吸取了教训，改用 POST 请求提交删除账户的请求。尽管如此，攻击者只需要在 B 网站中内嵌一个隐藏表单，然后设置在页面加载后执行提交表单的 JavaScript 函数，攻击仍然会在用户访问 B 网站时发起。

25. 虽然 CSRF 攻击看起来非常可怕，但我们仍然可以采取一些措施来进行防御。下面我们来介绍防范 CSRF 攻击的两种主要方式。

26. 主要防范措施

    - 正确使用 HTTP 方法

      - 防范 CSRF 的基础就是正确使用 HTTP 方法。在前面我们介绍过 HTTP 中的常用方法。在普通的 Web 程序中，一般只会使用到 GET 和 POST 方法。而且，目前在 HTML 中仅支持 GET 和 POST 方法(借助 AJAX 则可以使用其他方法)。在使用 HTTP 方法时，通常应该遵循下面的原则：
        - GET 方法属于安全方法，不会改变资源状态，仅用于获取资源，因此又被称为幂等方法(idempotent method)。页面中所有可以通过链接发起的请求都属于 GET 请求。
        - POST 方法用于创建、修改和删除资源。在 HTML 中使用 form 标签 创建表单并设置提交方法为 POST，在提交时会创建 POST 请求。

      - 在 GET 请求中，查询参数用来传入过滤返回的资源，但是在某些特殊情况下，也可以通过查询参数传递少量非敏感信息。

      - 虽然在实际开发中，通过在删除按钮中加入链接来删除资源非常方便，但安全问题应该作为编写代码时的第一考量，应该将这些按钮内嵌在使用了 POST 方法的 form 元素中。
      - 正确使用 HTTP 方法后，攻击者就无法通过 GET 请求来修改用户的数据，下面我们会介绍如何保护 GET 之外的请求。

    - CSRF 令牌校验
      - 当处理非 GET 请求时，要想避免 CSRF 攻击，关键在于判断请求是否来自自己的网站。在前面我们曾经介绍过使用 HTTP referer 获取请求来源，理论上说，通过 referer 可以判断源站点从而避免 CSRF 攻击，但因为 referer 很容易被修改和伪造，所以不能作为主要的防御措施。
      - 除了在表单中加入验证码外，一般的做法是通过在客户端页面中加入伪随机数来防御 CSRF 攻击，这个伪随机数通常被称为 CSRF 令牌(token)。
      - 在计算机语境中，令牌(token)指用于标记、验证和传递信息的字符，通常是通过一定算法生成的伪随机数，我们在本书后面会频繁接触到这个词。
      - 在 HTML 中，POST 方法的请求通过表单创建。我们把在服务器端创建的伪随机数(CSRF 令牌)添加到表单中的隐藏字段里和 session 变量(即签名cookie)中，当用户提交表单时，这个令牌会和表单数据一起提交。
      - 在服务器端处理 POST 请求时，我们会对表单中的令牌值进行验证，如果表单中的令牌值和 session 中的令牌值相同，那么就说明请求发自自己的网站。因为 CSRF 令牌在用户向包含表单的页面发起 GET 请求时创建，并且在一定时间内过期，一般情况下攻击者无法获取到这个令牌值，所以我们可以有效地区分出请求的来源是否安全。
      - 对于 AJAX 请求，我们可以在 XMLHttpRequest 请求首部添加一个自定义字段 X-CSRFToken 来保存 CSRF 令牌。

27. 如果程序包含 XSS 漏洞，那么攻击者可以使用跨站脚本攻破可能使用的任何跨站请求伪造(CSRF)防御机制，比如使用 JavaScript 窃取 cookie 内容，进而获取 CSRF 令牌。

28. 除了这几个攻击方式外，我们还有很多安全问题要注意。比如文件上传漏洞、敏感数据存储、用户认证(authentication)与权限管理等。

29. 需要注意的是，虽然本书会介绍如何对常见的攻击和漏洞进行防御和避免，但仍然有许多其他的攻击和漏洞需要读者自己处理。另外，本书的示例程序(包括第一部分和第二部分)仅用于作为功能实现的示例，在安全方面并未按照实际运行的应用进行严格处理。

30. 比如，当单个用户出现频繁的登录失败时，应该采取添加验证码或暂时停止接收该用户的登录请求。请阅读 OWASP 或其他相关资料学习更多安全防御技巧。

## 2.7 本章小结

1. HTTP 是各种 Web 程序的基础，本章只是简要介绍了和 Flask 相关的部分，没有涉及 HTTP 底层的 TCP/IP 或 DNS 协议。建议你通过阅读相关书籍来了解完整的 Web 原理，这将有助于编写更完善和安全的 Web 程序。
2. 在下一章，我们会学习使用 Flask 的模板引擎——Jinja2 ，通过学习运用模板和静态文件，我们可以让程序变得更加丰富和完善。

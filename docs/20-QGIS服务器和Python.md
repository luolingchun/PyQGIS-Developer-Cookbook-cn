# 20-QGIS服务器和Python

## 20.1 介绍

要了解有关QGIS服务器的更多信息，请阅读[QGIS服务器指南/手册](https://docs.qgis.org/testing/en/docs/server_manual/index.html#qgis-server-manual)。

QGIS服务器是三个不同的东西：

1. QGIS服务器库：一个为创建OGC网络服务提供API的库
2. QGIS服务器FCGI：一个FCGI二进制应用程序`qgis_maserv.fcgi`，与网络服务器一起实现一套OGC服务（WMS、WFS、WCS等）和OGC APIs（WFS3/OAPIF）。
3. QGIS开发服务器：一个开发服务器二进制应用程序`qgis_mapserver`，实现了一套OGC服务（WMS、WFS、WCS等）和OGC APIs（WFS3/OAPIF）。

本章的重点是第一个话题，通过解释QGIS服务器API的用法来说明如何使用Python来扩展、增强或定制服务器行为，或如何使用QGIS服务器API将QGIS服务器嵌入到另一个应用程序。

你可以通过一些不同的方式来改变QGIS服务器的行为或扩展其功能，以提供新的定制服务或API，一下是你可能面临的主要情况：

- 融合 → 从另一个Python应用程序中使用QGIS服务器API
- 独立 → 以独立的WSGI/HTTP服务方式运行QGIS服务器
- 过滤 → 使用过滤插件增强/定制QGIS服务器
- 服务 → 添加一个新的*服务*
- OGC APIs →  添加一个新的*OGC API*

嵌入和独立的应用程序需要直接从另一个Python脚本或应用程序中使用QGIS服务器的Python API。其余的选项更适合于当你想在标准的QGIS服务器二进制应用程序（FCGI或开发服务器）中添加自定义功能时：在这种情况下，你需要为服务器应用程序编写一个Python插件，并注册你的自定义过滤器、服务或API。

## 20.2 服务器API基础

一个典型的QGIS服务器应用程序所涉及的基本类是：

- [`QgsServer`](https://qgis.org/pyqgis/master/server/QgsServer.html#qgis.server.QgsServer) 服务实例 (通常在整个应用程序的生命周期中只有一个实例)
- [`QgsServerRequest`](https://qgis.org/pyqgis/master/server/QgsServerRequest.html#qgis.server.QgsServerRequest) 请求对象(通常在每次请求时重新创建)
- [`QgsServer.handleRequest(request, response)`](https://qgis.org/pyqgis/master/server/QgsServer.html#qgis.server.QgsServer.handleRequest) 处理请求并响应

QGIS服务器FCGI或开发服务器的工作流程可以概括为以下几点：

```
initialize the QgsApplication
create the QgsServer
the main server loop waits forever for client requests:
    for each incoming request:
        create a QgsServerRequest request
        create a QgsServerResponse response
        call QgsServer.handleRequest(request, response)
            filter plugins may be executed
        send the output to the client
```

在[`QgsServer.handleRequest(request, response)`](https://qgis.org/pyqgis/master/server/QgsServer.html#qgis.server.QgsServer.handleRequest)方法中，过滤器插件的回调函数被调用，[`QgsServerRequest`](https://qgis.org/pyqgis/master/server/QgsServerRequest.html#qgis.server.QgsServerRequest)和[`QgsServerResponse`](https://qgis.org/pyqgis/master/server/QgsServerResponse.html#qgis.server.QgsServerResponse)通过[`QgsServerInterface`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface)类被提供给插件。

!!! 警告 warning

    QGIS服务器类不是线程安全的，在构建基于QGIS服务器API的可扩展应用程序时，你应该始终使用多进程模型或容器。

## 20.3 独立或嵌入

对于独立的服务器应用或嵌入，你需要直接使用上述的服务器类，将它们包装成一个Web服务器实现，管理所有与客户端的HTTP协议交互。

这里有一个关于QGIS服务器API应用的最小例子（没有HTTP部分）：

```python
from qgis.core import QgsApplication
from qgis.server import *
app = QgsApplication([], False)

# 创建服务器实例，它可能是一个单一的实例，在多个请求中重复使用
server = QgsServer()

# 通过指定完整的URL和一个可选的主体来创建请求（例如POST请求）
request = QgsBufferServerRequest(
    'http://localhost:8081/?MAP=/qgis-server/projects/helloworld.qgs' +
    '&SERVICE=WMS&REQUEST=GetCapabilities')

# 创建响应对象
response = QgsBufferServerResponse()

# 处理请求
server.handleRequest(request, response)

print(response.headers())
print(response.body().data().decode('utf8'))

app.exitQgis()
```

这里有一个完整的独立应用实例，它是为QGIS源代码库的持续集成测试而开发的，它展示了一系列不同的插件过滤器和认证方案（不意味着可用于生产环境，因为它们只是为测试目的而开发的，但对于学习来说仍然很有趣）：https://github.com/qgis/QGIS/blob/master/tests/src/python/qgis_wrapped_server.py

## 20.4 服务器插件

服务器python插件在QGIS服务器应用程序启动时被加载一次，可用于注册过滤器、服务或API。

服务器插件的结构与桌面版的插件非常相似，一个[`QgsServerInterface`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface)对象被提供给插件，插件可以通过使用服务器接口暴露的方法将一个或多个自定义过滤器、服务或API注册到相应的注册表。

### 20.4.1 服务器过滤插件

过滤器有三种不同的类型，它们可以通过子类化下面的一个类并调用[`QgsServerInterface`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface)的相应方法来实例化。

| 过滤器类型     | 基类                                                         | QgsServerInterface 注册                                      |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| I/O            | [`QgsServerFilter`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter) | [`registerFilter()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.registerFilter) |
| Access Control | [`QgsAccessControlFilter`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter) | [`registerAccessControl()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.registerAccessControl) |
| Cache          | [`QgsServerCacheFilter`](https://qgis.org/pyqgis/master/server/QgsServerCacheFilter.html#qgis.server.QgsServerCacheFilter) | [`registerServerCache()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.registerServerCache) |

#### 20.4.1.1 I/O过滤器

I/O过滤器可以修改核心服务（WMS、WFS等）的服务器输入和输出（请求和响应），允许对服务工作流进行任何形式的操作。例如，可以限制对选定图层的访问，向XML响应注入XSL样式表，向生成的WMS图像添加水印等等。

从这一点来看，你可能会发现快速浏览一下[服务器插件的API文档](https://qgis.org/pyqgis/master/server)很有用。

每一个插件应该至少实现以下三个回调函数：

- [`requestReady()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.requestReady)
- [`responseComplete()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.responseComplete)
- [`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)

所有的过滤器都可以访问请求/响应对象（[`QgsRequestHandler`](https://qgis.org/pyqgis/master/server/QgsRequestHandler.html#qgis.server.QgsRequestHandler)），并且可以操作它的所有属性（输入/输出）和引发异常（同时以一种相当特别的方式，我们将在下面看到）。

下面是显示服务器如何处理一个典型请求以及何时调用过滤器回调函数的伪代码。

```
for each incoming request:
    create GET/POST request handler
    pass request to an instance of QgsServerInterface
    call requestReady filters
    if there is not a response:
        if SERVICE is WMS/WFS/WCS:
            create WMS/WFS/WCS service
            call service’s executeRequest
                possibly call sendResponse for each chunk of bytes
                sent to the client by a streaming services (WFS)
        call responseComplete
        call sendResponse
    request handler sends the response to the client
```

下面几个段落详细描述了可用的回调函数。

##### 20.4.1.1.1 请求就绪

当请求准备就绪时，将被调用：传入的URL和数据已经被解析，在进入核心服务（WMS，WFS等）开关之前，这是你可以操作输入和执行动作的地方：

- 认证授权
- 重定向
- 添加删除某些参数 (例如类型名称)
- 抛出异常

你甚至可以通过改变**SERVICE**参数来完全替代一个核心服务，从而完全绕过核心服务（不过这并没有什么意义）。

##### 20.4.1.1.2 发送响应

每当有任何输出被发送到**FCGI** `stdout`（并从那里发送到客户端）时，将被调用。这通常是在核心服务完成它们的过程和调用`responseComplete`钩子后进行的，但在少数情况下，XML会变得如此巨大，以至于需要一个流式XML实现（WFS GetFeature就是其中之一）。在这种情况下，在响应完成之前，可能会例外地多次调用[`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)，而不是单次调用该方法，在这种情况下（也只有在这种情况下），也会在[`responseComplete()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.responseComplete)之前调用。

[`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)是直接操作核心服务输出的最佳位置，虽然[`responseComplete()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.responseComplete)通常也是一种选择，但在流媒体服务的情况下，[`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)是唯一可行的选择。

##### 20.4.1.1.3 响应完成

当核心服务（如果被击中的话）完成它们的过程，并且请求准备好被发送到客户端时，将被调用一次。如上所述，通常在[`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)之前调用，除了流媒体服务（或其他插件过滤器）可能在之前调用[`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)。

[`responseComplete()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.responseComplete)是提供新服务实现（WPS或自定义服务）和对来自核心服务的输出进行直接操作的理想场所（例如，在WMS图像上添加水印）。

#### 20.4.1.2 从插件引发异常

在这个问题上还有一些工作要做：目前的实现可以通过将[`QgsRequestHandler`](https://qgis.org/pyqgis/master/server/QgsRequestHandler.html#qgis.server.QgsRequestHandler)属性设置为QgsMapServiceException的一个实例来区分已处理和未处理的异常，这样，主要的C++代码可以捕获已处理的Python异常而忽略未处理的异常（或者更好的是：记录日志）。

这种方法基本上是可行的，但它不是很 "pythonic"：一个更好的方法是在python代码中引发异常，并看到它们进入到C++循环中被处理。

#### 20.4.1.3 编写一个服务器插件

服务器插件是一个标准的 QGIS Python 插件，如[16-开发Python插件](16-开发Python插件.md)中所述，它只是提供了一个额外的（或替代的）接口：典型的 QGIS 桌面插件通过 [`QgisInterface `](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface)实例访问 QGIS 应用程序，而服务器插件只有在 QGIS Server 应用程序上下文中执行时才能访问。

为了让QGIS Server知道一个插件有一个服务器接口，需要一个特殊的元数据条目（在`metadata.txt`中）。

```ini
server=True
```

!!! 重要的 important 

    只有设置了`server=True`元数据的插件才能被QGIS Server加载和执行。

这里讨论的例子插件（还有很多）可以在github上找到，地址是https://github.com/elpaso/qgis3-server-vagrant/tree/master/resources/web/plugins，一些服务器插件也发布在[QGIS官方插件仓库](https://plugins.qgis.org/plugins/server)中。

#### 20.4.1.4 插件文件

下面是我们的示例服务器插件的目录结构。

```
PYTHON_PLUGINS_PATH/
  HelloServer/
    __init__.py    --> *required*
    HelloServer.py  --> *required*
    metadata.txt   --> *required*
```

##### 20.4.1.4.1 _\_init\_\_.py

这个文件是Python的导入系统所要求的。此外，QGIS Server要求该文件包含一个`serverClassFactory()`函数，当服务器启动时，插件被加载到QGIS Server中时，该函数将被调用。它接收对[`QgsServerInterface`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface)实例的引用，并必须返回你的插件类的实例。以下是插件示例 `__init__.py `的样子：

```python
def serverClassFactory(serverIface):
    from .HelloServer import HelloServerServer
    return HelloServerServer(serverIface)
```

##### 20.4.1.4.2  HelloServer.py

这就是魔法发生的地方，这就是魔法的模样。(例如：`HelloServer.py`)

一个服务器插件通常由一个或多个回调函数组成，被打包到[`QgsServerFilter`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter)的实例中。

每个[`QgsServerFilter`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter)都实现了一个或多个以下的回调函数：

- [`requestReady()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.requestReady)
- [`responseComplete()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.responseComplete)
- [`sendResponse()`](https://qgis.org/pyqgis/master/server/QgsServerFilter.html#qgis.server.QgsServerFilter.sendResponse)

下面的例子实现了一个最小的过滤器，当**SERVICE**参数等于 "**HELLO**"时，打印出*HelloServer*!

```python
class HelloFilter(QgsServerFilter):

    def __init__(self, serverIface):
        super().__init__(serverIface)

    def requestReady(self):
        QgsMessageLog.logMessage("HelloFilter.requestReady")

    def sendResponse(self):
        QgsMessageLog.logMessage("HelloFilter.sendResponse")

    def responseComplete(self):
        QgsMessageLog.logMessage("HelloFilter.responseComplete")
        request = self.serverInterface().requestHandler()
        params = request.parameterMap()
        if params.get('SERVICE', '').upper() == 'HELLO':
            request.clear()
            request.setResponseHeader('Content-type', 'text/plain')
            # 注意内容类型是"bytes"
            request.appendBody(b'HelloServer!')
```

过滤器必须被注册到**serverIface**中，如下例所示：

```python
class HelloServerServer:
    def __init__(self, serverIface):
        serverIface.registerFilter(HelloFilter(serverIface), 100)
```

[`registerFilter()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.registerFilter)的第二个参数设置了一个优先级，定义了同名回调函数的顺序（优先级低的回调先被调用）。

通过使用这三个回调函数，插件可以以许多不同的方式操纵服务器的输入输出。在每个时刻，插件实例都可以通过[`QgsServerInterface`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface)访问[`QgsRequestHandler`](https://qgis.org/pyqgis/master/server/QgsRequestHandler.html#qgis.server.QgsRequestHandler)。[`QgsRequestHandler`](https://qgis.org/pyqgis/master/server/QgsRequestHandler.html#qgis.server.QgsRequestHandler)类有很多方法，可以用来在进入服务器的核心处理之前（通过使用`requestReady()`）或在请求被核心服务处理之后（通过使用`sendResponse()`）改变输入参数。

下面的例子涵盖了一些常见的使用案例。

##### 20.4.1.4.3  修改输入

示例插件包含一个改变来自查询字符串的输入参数的测试例子，在这个例子中，一个新的参数被注入到（已经解析过的）`parameterMap`中，然后这个参数被核心服务（WMS等）看到，在核心服务处理结束时，我们检查这个参数是否仍然存在：

```python
class ParamsFilter(QgsServerFilter):

    def __init__(self, serverIface):
        super(ParamsFilter, self).__init__(serverIface)

    def requestReady(self):
        request = self.serverInterface().requestHandler()
        params = request.parameterMap( )
        request.setParameter('TEST_NEW_PARAM', 'ParamsFilter')

    def responseComplete(self):
        request = self.serverInterface().requestHandler()
        params = request.parameterMap( )
        if params.get('TEST_NEW_PARAM') == 'ParamsFilter':
            QgsMessageLog.logMessage("SUCCESS - ParamsFilter.responseComplete")
        else:
            QgsMessageLog.logMessage("FAIL    - ParamsFilter.responseComplete")
```

这是日志文件中内容的摘录：

```text hl_lines="6"
src/core/qgsmessagelog.cpp: 45: (logMessage) [0ms] 2014-12-12T12:39:29 plugin[0] HelloServerServer - loading filter ParamsFilter
 src/core/qgsmessagelog.cpp: 45: (logMessage) [1ms] 2014-12-12T12:39:29 Server[0] Server plugin HelloServer loaded!
 src/core/qgsmessagelog.cpp: 45: (logMessage) [0ms] 2014-12-12T12:39:29 Server[0] Server python plugins loaded
 src/mapserver/qgshttprequesthandler.cpp: 547: (requestStringToParameterMap) [1ms] inserting pair SERVICE // HELLO into the parameter map
 src/mapserver/qgsserverfilter.cpp: 42: (requestReady) [0ms] QgsServerFilter plugin default requestReady called
 src/core/qgsmessagelog.cpp: 45: (logMessage) [0ms] 2014-12-12T12:39:29 plugin[0] SUCCESS - ParamsFilter.responseComplete
```

在突出显示的一行，"SUCCESS "字符串表示该插件通过了测试。

同样的技术可以被利用来代替核心服务：例如，你可以跳过**WFS SERVICE**请求或任何其他核心请求，只需将**SERVICE**参数改为不同的参数，核心服务就会被跳过。然后，你可以将你的自定义结果注入到输出中，并将其发送给客户端（这将在下面解释）。

!!! 提示 infomation

    如果你真的想实现一个自定义的服务，建议将[`QgsService`](https://qgis.org/pyqgis/master/server/QgsService.html#qgis.server.QgsService)子类化，并通过调用其[`registerService(service)`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.serviceRegistry)方法在[`registerFilter()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.serviceRegistry)方法上注册你的服务。

##### 20.4.1.4.4  修改或替换输出

水印过滤器的例子显示了如何用一个新的图像替换WMS的输出，该图像是通过在WMS核心服务生成的WMS图像上添加一个水印图像而获得：

```python
from qgis.server import *
from qgis.PyQt.QtCore import *
from qgis.PyQt.QtGui import *

class WatermarkFilter(QgsServerFilter):

    def __init__(self, serverIface):
        super().__init__(serverIface)

    def responseComplete(self):
        request = self.serverInterface().requestHandler()
        params = request.parameterMap( )
        # 一些检查
        if (params.get('SERVICE').upper() == 'WMS' \
                and params.get('REQUEST').upper() == 'GETMAP' \
                and not request.exceptionRaised() ):
            QgsMessageLog.logMessage("WatermarkFilter.responseComplete: image ready %s" % request.parameter("FORMAT"))
            # 获取图像
            img = QImage()
            img.loadFromData(request.body())
            # 添加水印
            watermark = QImage(os.path.join(os.path.dirname(__file__), 'media/watermark.png'))
            p = QPainter(img)
            p.drawImage(QRect( 20, 20, 40, 40), watermark)
            p.end()
            ba = QByteArray()
            buffer = QBuffer(ba)
            buffer.open(QIODevice.WriteOnly)
            img.save(buffer, "PNG" if "png" in request.parameter("FORMAT") else "JPG")
            # 设置数据体
            request.clearBody()
            request.appendBody(ba)
```

在这个例子中，**SERVICE**参数值被检查，如果传入的请求是一个**WMS GETMAP**，并且没有被先前执行的插件或核心服务（在这个例子中是WMS）设置过异常，那么WMS生成的图像就会从输出缓冲区中被检索出来，并且添加水印图像。最后一步是清除输出缓冲区，用新生成的图像替换它。请注意，在现实世界中，我们还应该检查所要求的图像类型，而不是只支持PNG或JPG。

#### 20.4.1.5 访问控制过滤器

访问控制过滤器为开发者提供了对哪些层、要素和属性可以被访问的细粒度控制，以下回调函数可以在访问控制过滤器中实现：

- [`layerFilterExpression(layer)`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.layerFilterExpression)
- [`layerFilterSubsetString(layer)`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.layerFilterSubsetString)
- [`layerPermissions(layer)`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.layerPermissions)
- [`authorizedLayerAttributes(layer, attributes)`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.authorizedLayerAttributes)
- [`allowToEdit(layer, feature)`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.allowToEdit)
- [`cacheKey()`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.cacheKey)

##### 20.4.1.5.1. 插件文件

下面是我们的示例服务器插件的目录结构：

```
PYTHON_PLUGINS_PATH/
  MyAccessControl/
    __init__.py    --> *required*
    AccessControl.py  --> *required*
    metadata.txt   --> *required*
```

##### 20.4.1.5.2 _\_init\_\_.py

这个文件是Python的导入系统所要求的。对于所有的QGIS服务器插件来说，这个文件包含一个`serverClassFactory()`函数，当插件在启动时被加载到QGIS服务器中时，它将被调用。它接收一个对[`QgsServerInterface`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface)实例的引用，并必须返回一个你的插件类的实例。以下是插件实例 `__init__.py`的样子：

```python
def serverClassFactory(serverIface):
    from MyAccessControl.AccessControl import AccessControlServer
    return AccessControlServer(serverIface)
```

##### 20.4.1.5.3. AccessControl.py

```python
class AccessControlFilter(QgsAccessControlFilter):

    def __init__(self, server_iface):
        super().__init__(server_iface)

    def layerFilterExpression(self, layer):
        """ 返回一个额外的表达式过滤器 """
        return super().layerFilterExpression(layer)

    def layerFilterSubsetString(self, layer):
        """ 返回一个额外的子集字符串（通常是SQL）过滤器 """
        return super().layerFilterSubsetString(layer)

    def layerPermissions(self, layer):
        """ 返回该层的权限 """
        return super().layerPermissions(layer)

    def authorizedLayerAttributes(self, layer, attributes):
        """ 返回授权的图层属性 """
        return super().authorizedLayerAttributes(layer, attributes)

    def allowToEdit(self, layer, feature):
        """ 我们是否被授权修改以下几何图形 """
        return super().allowToEdit(layer, feature)

    def cacheKey(self):
        return super().cacheKey()

class AccessControlServer:

   def __init__(self, serverIface):
      """ 注册访问控制过滤器 """
      serverIface.registerAccessControl(AccessControlFilter(serverIface), 100)
```

这个例子为每个人提供了一个完整的访问权限。

插件的作用是知道谁在登录。

在所有这些方法中，我们都有一个图层的参数，以便能够定制每个图层的限制。

##### 20.4.1.5.4. layerFilterExpression

用于添加一个表达式来限制结果，例如：

```python
def layerFilterExpression(self, layer):
    return "$role = 'user'"
```

限制要素的属性`role`等于`“user”`。

##### 20.4.1.5.5. layerFilterSubsetString

与前者相同，但使用`SubsetString`（在数据库中执行）。

```python
def layerFilterSubsetString(self, layer):
    return "role = 'user'"
```

限制要素的属性`role`等于`“user”`。

##### 20.4.1.5.6. layerPermissions

限制访问图层。

返回一个[`LayerPermissions()`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.layerPermissions)对象，它有以下属性：

- [`canRead`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.LayerPermissions.canRead)可以在`GetCapabilities`中看到它并有读取权限。
- [`canInsert`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.LayerPermissions.canInsert)能够插入一个新的要素。
- [`canUpdate`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.LayerPermissions.canUpdate)能够更新一个要素。
- [`canDelete`](https://qgis.org/pyqgis/master/server/QgsAccessControlFilter.html#qgis.server.QgsAccessControlFilter.LayerPermissions.canDelete)能够删除一个要素。

例如：

```python
def layerPermissions(self, layer):
    rights = QgsAccessControlFilter.LayerPermissions()
    rights.canRead = True
    rights.canInsert = rights.canUpdate = rights.canDelete = False
    return rights
```

限制每一个人只读访问。

##### 20.4.1.5.7. authorizedLayerAttributes

用于限制一个特定的属性子集的可见性。

参数 attribute 返回当前的可见属性集。

例如：

```python
def authorizedLayerAttributes(self, layer, attributes):
    return [a for a in attributes if a != "role"]
```

隐藏‘role’属性。

##### 20.4.1.5.8. allowToEdit

这被用来限制对一个属性子集的编辑。

它在`WFS-Transaction`协议中使用。

例如：

```python
def allowToEdit(self, layer, feature):
    return feature.attribute('role') == 'user'
```

只能够编辑具有‘role’属性的要素。

##### 20.4.1.5.9. cacheKey

QGIS服务器会对能力进行缓存，如果要对每个角色进行缓存，你可以在此方法中返回角色。或者返回`None`以完全禁用缓存。

### 20.4.2 自定义服务

在QGIS服务器中，WMS、WFS和WCS等核心服务是作为[`QgsService`](https://qgis.org/pyqgis/master/server/QgsService.html#qgis.server.QgsService)的子类实现的。

为了实现一个新的服务，当查询字符串参数`SERVICE`与服务名称相匹配时将被执行，你可以实现你自己的[`QgsService`](https://qgis.org/pyqgis/master/server/QgsService.html#qgis.server.QgsService)，并通过调用[`registerService(service)`](https://qgis.org/pyqgis/master/server/QgsServiceRegistry.html#qgis.server.QgsServiceRegistry.registerService)在[`serviceRegistry()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.serviceRegistry)上注册你的服务。

下面是一个名为CUSTOM的自定义服务的例子：

```python
from qgis.server import QgsService
from qgis.core import QgsMessageLog

class CustomServiceService(QgsService):

    def __init__(self):
        QgsService.__init__(self)

    def name(self):
        return "CUSTOM"

    def version(self):
        return "1.0.0"

    def executeRequest(self, request, response, project):
        response.setStatusCode(200)
        QgsMessageLog.logMessage('Custom service executeRequest')
        response.write("Custom service executeRequest")


class CustomService():

    def __init__(self, serverIface):
        serverIface.serviceRegistry().registerService(CustomServiceService())
```

### 20.4.3. 自定义APIs

在QGIS Server中，诸如OAPIF（又称WFS3）等核心OGC APIs被实现为[`QgsServerOgcApiHandler`](https://qgis.org/pyqgis/master/server/QgsServerOgcApiHandler.html#qgis.server.QgsServerOgcApiHandler)子类的集合，这些子类被注册到[`QgsServerOgcApi`](https://qgis.org/pyqgis/master/server/QgsServerOgcApi.html#qgis.server.QgsServerOgcApi)（或其父类[`QgsServerApi`](https://qgis.org/pyqgis/master/server/QgsServerApi.html#qgis.server.QgsServerApi)）的实例中。

要实现一个新的API，当url路径与某个URL相匹配时就会被执行，你可以实现你自己的[`QgsServerOgcApiHandler`](https://qgis.org/pyqgis/master/server/QgsServerOgcApiHandler.html#qgis.server.QgsServerOgcApiHandler)实例，将它们添加到[`QgsServerOgcApi`](https://qgis.org/pyqgis/master/server/QgsServerOgcApi.html#qgis.server.QgsServerOgcApi)中，并通过调用其[`registerApi(api)`](https://qgis.org/pyqgis/master/server/QgsServiceRegistry.html#qgis.server.QgsServiceRegistry.registerApi)在[`serviceRegistry()`](https://qgis.org/pyqgis/master/server/QgsServerInterface.html#qgis.server.QgsServerInterface.serviceRegistry)上注册该API。

下面是一个自定义API的例子，当URL包含`/customapi`时将被执行：

```python
import json
import os

from qgis.PyQt.QtCore import QBuffer, QIODevice, QTextStream, QRegularExpression
from qgis.server import (
    QgsServiceRegistry,
    QgsService,
    QgsServerFilter,
    QgsServerOgcApi,
    QgsServerQueryStringParameter,
    QgsServerOgcApiHandler,
)

from qgis.core import (
    QgsMessageLog,
    QgsJsonExporter,
    QgsCircle,
    QgsFeature,
    QgsPoint,
    QgsGeometry,
)


class CustomApiHandler(QgsServerOgcApiHandler):

    def __init__(self):
        super(CustomApiHandler, self).__init__()
        self.setContentTypes([QgsServerOgcApi.HTML, QgsServerOgcApi.JSON])

    def path(self):
        return QRegularExpression("/customapi")

    def operationId(self):
        return "CustomApiXYCircle"

    def summary(self):
        return "Creates a circle around a point"

    def description(self):
        return "Creates a circle around a point"

    def linkTitle(self):
        return "Custom Api XY Circle"

    def linkType(self):
        return QgsServerOgcApi.data

    def handleRequest(self, context):
        """Simple Circle"""

        values = self.values(context)
        x = values['x']
        y = values['y']
        r = values['r']
        f = QgsFeature()
        f.setAttributes([x, y, r])
        f.setGeometry(QgsCircle(QgsPoint(x, y), r).toCircularString())
        exporter = QgsJsonExporter()
        self.write(json.loads(exporter.exportFeature(f)), context)

    def templatePath(self, context):
        # 模板路径用于提供HTML内容
        return os.path.join(os.path.dirname(__file__), 'circle.html')

    def parameters(self, context):
        return [QgsServerQueryStringParameter('x', True, QgsServerQueryStringParameter.Type.Double, 'X coordinate'),
                QgsServerQueryStringParameter(
                    'y', True, QgsServerQueryStringParameter.Type.Double, 'Y coordinate'),
                QgsServerQueryStringParameter('r', True, QgsServerQueryStringParameter.Type.Double, 'radius')]


class CustomApi():

    def __init__(self, serverIface):
        api = QgsServerOgcApi(serverIface, '/customapi',
                            'custom api', 'a custom api', '1.1')
        handler = CustomApiHandler()
        api.registerHandler(handler)
        serverIface.serviceRegistry().registerApi(api)
```


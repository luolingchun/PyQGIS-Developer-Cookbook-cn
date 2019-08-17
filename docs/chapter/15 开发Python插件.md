# 15 开发Python插件

可以用Python编程语言创建插件。与用C ++编写插件相比，由于Python语言的动态特性，这些插件应该更容易编写，理解，维护和分发。

Python插件与QGIS插件管理器中的C ++插件一起列出。他们在`~/(UserProfile)/python/plugins`和以下路径中搜索：

- UNIX / Mac上： `(qgis_prefix)/share/qgis/python/plugins`
- Windows： `(qgis_prefix)/python/plugins`

有关`~`和`(UserProfile)`的定义,请查看[Core和External插件](https://docs.qgis.org/3.4/en/docs/user_manual/plugins/plugins.html#core-and-external-plugins)。

------

**小贴士：** 将QGIS_PLUGINPATH设置为现有目录路径，可以将此路径添加到插件的搜索路径列表中。

------

## 15.1 构建Python插件

要创建插件，请执行以下步骤：

1. *想法*：了解你想要使用新的QGIS插件做什么。你为什么要这么做？你想解决什么问题？这个问题已经有另一个插件吗？
2. *创建文件*：要点：一个起点`__init__.py`; 填写[插件元数据](https://docs.qgis.org/3.4/en/docs/pyqgis_developer_cookbook/plugins/plugins.html#plugin-metadata)`metadata.txt`。然后实现自己的设计。一个主要的Python插件体，例如`mainplugin.py`。可能是Qt Designer中的一个表单`form.ui`，带有它`resources.qrc`。
3. *编写代码*：在里面写代码`mainplugin.py`
4. *测试*：关闭并重新打开QGIS并再次导入插件。检查一切是否正常。
5. *发布*：在QGIS存储库中发布你的插件或将你自己的存储库作为个人“GIS武器”的“武器库”。

### 15.1.1 插件结构

自从在QGIS中引入Python插件以来，出现了许多插件。QGIS团队维护一个[官方Python插件库](https://docs.qgis.org/3.4/en/docs/pyqgis_developer_cookbook/plugins/releasing.html#official-pyqgis-repository)。你可以使用他们的源码来了解使用PyQGIS进行编程的更多信息，或者了解你是否在重复开发。

  这是我们的示例插件的目录结构

```bash
PYTHON_PLUGINS_PATH/
  MyPlugin/
    __init__.py    --> *必需*
    mainPlugin.py  --> *核心代码*
    metadata.txt   --> *必需*
    resources.qrc  --> *可能需要*
    resources.py   --> *编译版本, 可能需要*
    form.ui        --> *可能需要*
    form.py        --> *编译版本, 可能需要*
```

这些文件的含义是什么：

- `__init__.py`=插件的入口。它必须具有 `classFactory()`方法，还可以具有任何其他初始化代码。
- `mainPlugin.py`=插件的主要工作代码。包含有关插件操作和主要代码的所有信息。
- `resources.qrc`= Qt Designer创建的xml文档。包含表单资源的相对路径。
- `resources.py` =将上述.qrc文件转换为Python代码。
- `form.ui` = Qt Designer创建的GUI。
- `form.py` =将上面描述的form.ui转换为Python代码。
- `metadata.txt` =包含插件网站和插件基础结构使用的常规信息，版本、名称和一些其他元数据。

[这](https://www.dimitrisk.gr/qgis/creator/)是一种自动创建典型QGIS Python插件的基本文件（框架）的在线方式。

有一个名为[Plugin Builder 3](https://plugins.qgis.org/plugins/pluginbuilder3/)的QGIS插件 ，它为QGIS创建一个插件模板，不需要互联网连接。这是推荐的选择，因为它兼容3.x版本。

### 15.1.2 插件内容

在这里，你可以找到有关在上述文件结构中，每个文件需要添加内容的信息和示例。

#### metadata

首先，插件管理器需要检索有关插件的一些基本信息，例如其名称、描述等。文件`metadata.txt`存储此信息。

**注意：** 所有元数据必须采用UTF-8编码。

| 元数据名称         | 是否必需 | 描述                                                         |
| :----------------- | :------: | :----------------------------------------------------------- |
| name               |    是    | 插件名称，短字符串                                           |
| qgisMinimumVersion |    是    | 最小QGIS版本                                                 |
| qgisMaximumVersion |    否    | 最大QGIS版本                                                 |
| description        |    是    | 描述插件的简短文本，不支持HTML                               |
| about              |    是    | 较长的文本，详细描述插件，不支持HTML                         |
| version            |    是    | 版本                                                         |
| author             |    是    | 作者姓名                                                     |
| email              |    是    | 作者的电子邮件，在网站上仅显示给登录的用户，但在插件安装后可在插件管理器中看到 |
| changelog          |    否    | z字符串，可以是多行，不支持HTML                              |
| experimental       |    否    | 布尔值，True或False                                          |
| deprecated         |    否    | boolean值，True或False，适用于整个插件，而不仅仅适用于上传的版本 |
| tags               |    否    | 以逗号分隔的列表，允许在单个标记内使用空格                   |
| homepage           |    否    | 指向插件主页的有效网址                                       |
| repository         |    是    | 源代码存储库的有效URL                                        |
| tracker            |    否    | 故障单和错误报告的有效URL                                    |
| icon               |    否    | d对于web友好的图像（PNG，JPEG）的文件名或相对路径（相对于插件压缩包的文件夹） |
| category           |    否    | Raster, Vector, Database and Web（栅格、矢量、数据库和网络） |

默认情况下，插件放在**插件菜单**中（我们将在下一节中看到如何为插件添加菜单项），但也可以将它们放入**Raster**，**Vector**， **Database**和**Web**菜单中。

输入指定的“category”元数据，可以相应地对插件进行分类。此元数据用作提示用户，并告诉他们可以在哪里（在哪个菜单中）找到该插件。“category”的允许值为：Vector, Raster, Database or Web。例如，如果你的插件可以从Raster菜单中获得，请将其添加到`metadata.txt`

```ini
category=Raster
```

------

**小贴士：** 如果qgisMaximumVersion为空，则在上传到[官方Python插件存储库](https://docs.qgis.org/3.4/en/docs/pyqgis_developer_cookbook/plugins/releasing.html#official-pyqgis-repository)时，它将自动设置为主要版本加上.99（例如：3.99）。

------

metadata.txt示例：

```ini
; 以下是强制性的

[general]
name=HelloWorld
email=me@example.com
author=Just Me
qgisMinimumVersion=3.0
description=This is an example plugin for greeting the world.\
    Multiline is allowed:\
    lines starting with spaces belong to the same\
    field, in this case to the "description" field.\
    HTML formatting is not allowed.
about=This paragraph can contain a detailed description\
    of the plugin. Multiline is allowed, HTML is not.
version=version 1.2
tracker=http://bugs.itopen.it
repository=http://www.itopen.it/repo
; 结束强制

; 以下是可选的
category=Raster
changelog=The changelog lists the plugin versions\
    and their changes as in the example below:\
    1.0 - First stable release\
    0.9 - All features implemented\
    0.8 - First testing release

; 标签采用逗号分隔的值格式，标签名称允许使用空格
; 标签应该是英文的。在创建之前请检查现有标签和同义词
tags=wkt,raster,hello world

; 这些元数据可以为空，最终将成为强制性的。
homepage=https://www.itopen.it
icon=icon.png

; 实验标志（适用于单一版本）
experimental=True

; 弃用标志 （适用于整个插件，不仅适用于上传的版本）
deprecated=False

; 如果为空，它将自动设置为主要版本+.99
qgisMaximumVersion=3.99
```

#### \_\_init\_\_.py

Python包需要此文件。此外，QGIS要求此文件包含一个`classFactory()`函数，该函数在插件被加载到QGIS时调用。它接收[`QgisInterface`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface)实例， 并且必须返回`mainplugin.py`中插件类的对象-——在我们的例子中它被命名为`TestPlugin`（见下文）。`__init__.py`应该是什么样的：

```python
def classFactory(iface):
	from .mainPlugin import TestPlugin
	return TestPlugin(iface)
```

#### mainPlugin.py

这就是魔法发生的地方，下面是就是魔法的例子:

```python
from qgis.PyQt.QtGui import *
from qgis.PyQt.QtWidgets import *

# 从文件resources.py初始化Qt的资源
from . import resources


class TestPlugin:

    def __init__(self, iface):
        # 保存QGIS interface引用
        self.iface = iface

    def initGui(self):
        # 创建动作，它将启动插件配置
        self.action = QAction(QIcon(":/plugins/testplug/icon.png"), "Test plugin", self.iface.mainWindow())
        self.action.setObjectName("testAction")
        self.action.setWhatsThis("Configuration for test plugin")
        self.action.setStatusTip("This is status tip")
        self.action.triggered.connect(self.run)

        # 添加工具栏按钮和菜单项
        self.iface.addToolBarIcon(self.action)
        self.iface.addPluginToMenu("&Test plugins", self.action)

        # 连接信号renderComplete——画布渲染完成后发送的信号
        self.iface.mapCanvas().renderComplete.connect(self.renderTest)

    def unload(self):
        # 删除插件菜单项和图标
        self.iface.removePluginMenu("&Test plugins", self.action)
        self.iface.removeToolBarIcon(self.action)

        # 断开信号
        self.iface.mapCanvas().renderComplete.disconnect(self.renderTest)

    def run(self):
        # 创建并显示一个配置对话框或类似的事情
        print("TestPlugin: run called!")

    def renderTest(self, painter):

        # 使用painter绘制地图画布
        print("TestPlugin: renderTest called!")
```

主插件源文件中（例如 `mainPlugin.py`）必须存在的插件函数是：

- `__init__` - >可以访问QGIS界面
- `initGui()` - >加载插件时调用
- `unload()` - >在卸载插件时调用

在上面的例子中，[`addPluginToMenu`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToMenu)被使用。这会将相应的菜单操作添加到**Plugins** 菜单中。存在将动作（action）添加到不同菜单的方法。以下是这些方法的列表：

- [`addPluginToRasterMenu()`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToRasterMenu)
- [`addPluginToVectorMenu()`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToVectorMenu)
- [`addPluginToDatabaseMenu()`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToDatabaseMenu)
- [`addPluginToWebMenu()`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToWebMenu)

它们都具有与[`addPluginToMenu`](https://qgis.org/pyqgis/3.4/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToMenu)方法相同的语法 。

建议将插件菜单添加到其中一个预定义方法，以保持插件条目组织方式的一致性。但是，你可以将自定义菜单组直接添加到菜单栏，如下面示例所示：

```python
def initGui(self):
    self.menu = QMenu(self.iface.mainWindow())
    self.menu.setObjectName("testMenu")
    self.menu.setTitle("MyMenu")

    self.action = QAction(QIcon(":/plugins/testplug/icon.png"), "Test plugin", self.iface.mainWindow())
    self.action.setObjectName("testAction")
    self.action.setWhatsThis("Configuration for test plugin")
    self.action.setStatusTip("This is status tip")
    self.action.triggered.connect(self.run)
    self.menu.addAction(self.action)

    menuBar = self.iface.mainWindow().menuBar()
    menuBar.insertMenu(self.iface.firstRightStandardMenu().menuAction(), self.menu)

def unload(self):
    self.menu.deleteLater()
```

不要忘记设置`QAction`和`QMenu` `objectName`插件的特定名称，以便可以自定义。

#### resources

你可以看到在`initGui()`中我们使用了资源文件中的图标（在我们的案例中是`resources.qrc`）

```xml
<RCC>
  <qresource prefix="/plugins/testplug" >
     <file>icon.png</file>
  </qresource>
</RCC>
```

最好使用不会与其他插件或QGIS的任何部分发生冲突的前缀，否则你可能会得到你不想要的资源。现在你只需要生成一个包含资源的Python文件。它是用**pyrcc5**命令完成的：

```shell
pyrcc5 -o resources.py resources.qrc
```

如果你已正确完成所有操作，则应该能够在插件管理器中查找并加载插件，在点击工具栏图标或相应的菜单项时，可以在控制台中查看到消息。

在处理真正的插件时，最好将插件写入另一个（工作）目录并创建一个makefile，它将生成UI +资源文件并将插件安装到QGIS安装中。

### 15.1.3 文档

该插件的文档可以编写为HTML文件。`qgis.utils`模块提供了一个功能，`showPluginHelp()`它将以与其他QGIS帮助文档相同的方式打开帮助文件。

`showPluginHelp()`函数在与调用模块相同的目录中查找帮助文档。它会寻找，`index-ll_cc.html`，`index-ll.html`，`index-en.html`，`index-en_us.html`和`index.html`，显示此文档，无论它找到的第一个。这`ll_cc`是QGIS语言环境，这允许文档有多个翻译。

`showPluginHelp()`函数还可以使用参数packageName，它标识将显示帮助的特定插件，filename，可以替换被搜索的文件名称中的“index”，section，它是html锚标记的名称，浏览器将定位到该位置。

### 15.1.4 翻译

通过几个步骤，你可以为插件本地化设置环境，以便根据计算机的区域设置，插件将以不同语言加载。

#### 软件要求

创建和管理所有翻译文件的最简单方法是安装 [Qt Linguist](https://doc.qt.io/qt-5/qtlinguist-index.html)。在基于Debian的GNU / Linux环境中，你可以安装它：

```shell
sudo apt-get install qttools5-dev-tools
```

#### 文件和目录

创建插件时，你将在主插件目录中找到该文件夹`i18n`，**所有翻译文件都必须在此目录中。**

#### .pro文件

首先，你应该创建一个`.pro`文件，这是一个可以由**Qt Linguist**管理的*项目*文件。

在此`.pro`文件中，你必须指定要翻译的所有文件和表单。此文件用于设置本地化文件和变量。一个项目文件，匹配我们的[示例插件](https://docs.qgis.org/3.4/en/docs/pyqgis_developer_cookbook/plugins/plugins.html#plugin-files-architecture)的结构 ：

```ini
FORMS = ../form.ui
SOURCES = ../your_plugin.py
TRANSLATIONS = your_plugin_it.ts
```

你的插件可能遵循更复杂的结构，并且可能分布在多个文件中。如果是这种情况，请记住，使用`pylupdate5`读取`.pro`文件并更新可翻译字符串，这不会扩展通配符，因此你需要将每个文件显式放在`.pro`文件中。你的项目文件可能看起来像这样：

```ini
FORMS = ../ui/about.ui ../ui/feedback.ui \
        ../ui/main_dialog.ui
SOURCES = ../your_plugin.py ../computation.py \
          ../utils.py
```

此外，`your_plugin.py`文件是*调用* QGIS工具栏中插件的所有菜单和子菜单的文件，你希望将它们全部翻译。

最后，使用*TRANSLATIONS*变量，你可以指定所需的翻译语言。

#### .ts文件

创建完成后，`.pro`你就可`.ts`以为插件的语言生成文件了。

打开终端，转到`your_plugin/i18n`目录并键入：

```shell
pylupdate5 your_plugin.pro
```

你应该看到`your_plugin_language.ts`文件。

`.ts`用**Qt Linguist**打开文件并开始翻译。

#### .qm文件

当你完成翻译插件时（如果某些字符串未完成，将使用这些字符串的源语言），你必须创建`.qm` 文件（编译文件`.ts`将被QGIS使用）。

只需在`your_plugin/i18n`目录中打开终端cd 并输入：

```shell
lrelease your_plugin.ts
```

现在，在`i18n`目录中你将看到`your_plugin.qm`文件。

#### 使用MakeFile进行编译

或者，如果你使用Plugin Builder创建了插件，则可以使用makefile从python代码和Qt对话框中提取消息。在Makefile的开头有一个LOCALES变量：

```ini
LOCALES = en
```

将该语言的缩写添加到此变量中，例如匈牙利语：

```ini
LOCALES = en hu
```

现在，你可以通过以下方式从源生成或更新`hu.ts`文件（以及其中的`en.ts`）：

```shell
make transup
```

在此之后，你已在LOCALES变量中更新`.ts`了所有语言的文件。使用**Qt Linguist**翻译程序消息。完成翻译后，`.qm`可以通过反编译创建：

```shell
make transcompile
```

你必须在你的插件中分发`.ts`文件。

#### 加载插件

要查看插件的翻译，只需打开QGIS，更改语言（**设置‣选项‣语言**）并重新启动QGIS。

你应该用正确的语言看你的插件。

### 15.1.5 提示和技巧

#### 插件重装载器

在开发插件期间，你经常需要在QGIS中重新加载它以进行测试。使用Plugin Reloader插件非常容易。你可以使用插件管理器中找到。

#### 访问插件

你可以使用python从QGIS中访问所有已安装插件类，这可以方便调试：

```python
my_plugin = qgis.utils.plugins['My Plugin']
```

#### 日志消息

插件在“ [日志消息”面板中](https://docs.qgis.org/3.4/en/docs/user_manual/introduction/general_tools.html#log-message-panel)有自己的选项卡。

#### 分享你的插件

QGIS在插件存储库中托管了数百个插件。考虑分享你的插件！它将扩展QGIS，人们将能够从你的代码中学习。可以使用插件管理器在QGIS中找到并安装所有托管插件。

信息和要求：[plugins.qgis.org](https://plugins.qgis.org/)。

## 15.2 代码片段

本节以代码片段为例，讲解插件开发

### 15.2.1 如何通过快捷键调用方法

在`initGui()`中添加：

```python
self.keyAction = QAction("Test Plugin", self.iface.mainWindow())
self.iface.registerMainWindowAction(self.keyAction, "F7") # 通过F7键触发＃动作1 
self.iface.addPluginToMenu("&Test plugins", self.keyAction)
self.keyAction.triggered().connect(self.keyActionF7)
```

在`unload()`中添加：

```python
self.iface.unregisterMainWindowAction(self.keyAction)
```

按下F7时调用的方法：

```python
def keyActionF7(self):
	QMessageBox.information(self.iface.mainWindow(),"Ok", "You pressed F7")
```

### 15.2.2 如何切换图层

从QGIS 2.4开始，新的图层树API允许直接访问图层面板的图层树。下面是如何切换当前图层可见性的示例：

```python
root = QgsProject.instance().layerTreeRoot()
node = root.findLayer(iface.activeLayer().id())
new_state = Qt.Checked if node.isVisible() == Qt.Unchecked else Qt.Unchecked
node.setItemVisibilityChecked(new_state)
```

### 15.2.3 如何访问所选要素的属性表

```python
def changeValue(self, value):
    layer = self.iface.activeLayer()
    if (layer):
        nF = layer.selectedFeatureCount()
        if (nF > 0):
            layer.startEditing()
            ob = layer.selectedFeaturesIds()
            b = QVariant(value)
            if (nF > 1):
                for i in ob:
                    layer.changeAttributeValue(int(i), 1, b)  # 1 是第二列
            else:
                layer.changeAttributeValue(int(ob[0]), 1, b)  # 1 是第二列
            layer.commitChanges()
        else:
            QMessageBox.critical(self.iface.mainWindow(), "Error",
                                 "Please select at least one feature from current layer")
    else:
        QMessageBox.critical(self.iface.mainWindow(), "Error", "Please select a layer")
```

该方法需要一个参数（所选要素的属性字段的新值），可以通过调用以下方法：

```python
self.changeValue(50)
```

## 15.3 使用插件图层

如果你的插件使用自己的方法来渲染地图图层，那么基于QgsPluginLayer编写自己的图层类型可能是实现它的最佳方式。

下面是一个QgsPluginLayer实现的示例。它是[Watermark示例插件](https://github.com/sourcepole/qgis-watermark-plugin)的摘录

```python
class WatermarkPluginLayer(QgsPluginLayer):
    LAYER_TYPE = "watermark"

    def __init__(self):
        QgsPluginLayer.__init__(self, WatermarkPluginLayer.LAYER_TYPE, "Watermark plugin layer")
        self.setValid(True)

    def draw(self, rendererContext):
        image = QImage("myimage.png")
        painter = rendererContext.painter()
        painter.save()
        painter.drawImage(10, 10, image)
        painter.restore()
        return True
```

还可以添加项目文件特定信息的读取和写入方法

```python
def readXml(self, node):
	pass

def writeXml(self, node, doc):
	pass
```

加载包含此类图层的项目时，需要工厂类

```python
class WatermarkPluginLayerType(QgsPluginLayerType):

    def __init__(self):
        QgsPluginLayerType.__init__(self, WatermarkPluginLayer.LAYER_TYPE)

    def createLayer(self):
        return WatermarkPluginLayer()
```

你还可以添加一下代码，用于在图层属性中显示自定义信息

```python
def showLayerProperties(self, layer):
	pass
```

## 15.4 编码与调试

此小节描述过于复杂，请参见我的博客：[PyQGIS插件开发经验](<https://blog.csdn.net/this_is_id/article/details/90020197>)

## 15.5 发布你的插件

**TODO**
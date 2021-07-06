# 16 开发Python插件

可以用Python编程语言创建插件。与用C ++编写插件相比，由于Python语言的动态特性，这些插件应该更容易编写，理解，维护和分发。

Python插件与QGIS插件管理器中的C ++插件一起列出。他们在`~/(UserProfile)/python/plugins`和以下路径中搜索：

- UNIX / Mac上： `(qgis_prefix)/share/qgis/python/plugins`
- Windows： `(qgis_prefix)/python/plugins`

有关`~`和`(UserProfile)`的定义,请查看[Core和External插件](https://docs.qgis.org/latest/en/docs/user_manual/plugins/plugins.html#core-and-external-plugins)。

!!! 提示

     将QGIS_PLUGINPATH设置为一个存在的目录路径，可以将此路径添加到插件的搜索路径列表中。


## 16.1 构建Python插件

创建插件，需要以下步骤：

1. *想法*：你想要使用新的QGIS插件做什么。你为什么要这么做？你想解决什么问题？这个问题已经有另一个插件吗？
2. *创建文件*：一些必要文件（查看[插件文件](#16111)）
3. *编写代码*：在恰当的文件中写代码
4. *测试*：如果准备就绪，[重新加载插件](#1615)
5. *发布*：在QGIS仓库中发布你的插件或将你自己的仓库作为个人“GIS武器”的“武器库”。

### 16.1.1 编写一个插件

自从在QGIS中引入Python插件以来，出现了许多插件。QGIS团队维护了一个[官方Python插件库](#1643)。你可以使用他们的源码来了解使用PyQGIS进行编程的更多信息，或者了解你是否在重复开发。

#### 16.1.1.1 插件文件

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
- `resources.qrc`= Qt设计师创建的xml文档。包含表单资源的相对路径。
- `resources.py` =将上述.qrc文件转换为Python代码。
- `form.ui` = Qt设计师创建的GUI。
- `form.py` =将上面描述的form.ui转换为Python代码。
- `metadata.txt` =包含插件网站和插件基础结构使用的常规信息，版本、名称和一些其他元数据。

[这](https://github.com/wonder-sk/qgis-minimal-plugin)是一种自动创建典型QGIS Python插件的基本文件（框架）的方式。

有一个名为[Plugin Builder 3](https://plugins.qgis.org/plugins/pluginbuilder3/)的QGIS插件 ，它为QGIS创建一个插件模板，不需要互联网连接。这是推荐的选择，因为它兼容3.x版本。

!!! warning 警告

    如果您打算将插件上传到[Python官方插件库](#1643)，则必须检查插件是否遵循插件[验证](#16433)所必需的一些附加规则


### 16.1.2 插件内容

在这里，你可以找到有关在上述文件结构中的每个文件需要添加内容的信息和示例。

#### 16.1.2.1 插件元数据

首先，插件管理器需要检索有关插件的一些基本信息，例如其名称、描述等。文件`metadata.txt`存储此信息。

!!! 提示

    所有元数据必须采用UTF-8编码。

| 元数据名称            | 是否必需 | 描述                                                         |
| :-------------------- | :------: | :----------------------------------------------------------- |
| name                  |    是    | 插件名称，短字符串                                           |
| qgisMinimumVersion    |    是    | 最小QGIS版本                                                 |
| qgisMaximumVersion    |    否    | 最大QGIS版本                                                 |
| description           |    是    | 描述插件的简短文本，不支持HTML                               |
| about                 |    是    | 较长的文本，详细描述插件，不支持HTML                         |
| version               |    是    | 版本                                                         |
| author                |    是    | 作者姓名                                                     |
| email                 |    是    | 作者的电子邮件，在网站上仅显示给登录的用户，但在插件安装后可在插件管理器中看到 |
| changelog             |    否    | 字符串，可以是多行，不支持HTML                               |
| experimental          |    否    | 实验性，布尔值，True或False                                  |
| deprecated            |    否    | 弃用，boolean值，True或False，适用于整个插件，而不仅仅适用于上传的版本 |
| tags                  |    否    | 以逗号分隔的列表，允许在单个标记内使用空格                   |
| homepage              |    否    | 指向插件主页的有效网址                                       |
| repository            |    是    | 源代码存储库的有效URL                                        |
| tracker               |    否    | 故障和错误报告的有效URL                                      |
| icon                  |    否    | 对于web友好的图像（PNG，JPEG）文件名或相对路径（相对于插件压缩包的文件夹） |
| category              |    否    | Raster, Vector, Database and Web（栅格、矢量、数据库和网络） |
| plugin_dependencies   |    否    | 类似于PIP的逗号分隔的其他插件列表                            |
| server                |    否    | 布尔值，True或False，确定插件是否具有服务器接口              |
| hasProcessingProvider |    否    | 布尔值，True或False，确定插件是否提供处理算法                |

默认情况下，插件放在**Plugins**菜单中（我们将在下一节中看到如何为插件添加菜单项），但也可以将它们放入**Raster**，**Vector**， **Database**和**Web**菜单中。

输入指定的“category”元数据，可以相应地对插件进行分类。此元数据用于提示用户，并告诉他们可以在哪里（在哪个菜单中）找到该插件。“category”的允许值为：Vector, Raster, Database或者Web。例如，如果你的插件可以从Raster菜单中找到，请将其添加到`metadata.txt`中

```ini
category=Raster
```

!!! 提示

    如果qgisMaximumVersion为空，则在上传到[官方Python插件库](#1643)时，它将自动设置为主要版本加上.99（例如：3.99）。


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

; 标签采用逗号分隔，标签名称允许使用空格
; 标签应该是英文的，在创建之前请检查现有标签和同义词
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

; 从 QGIS 3.8开始，可以指定以逗号分隔指定要安装（或更新）的插件列表
; 下面示例安装或更新版本1.12的“MyOtherPlugin”和任何版本的“YetAnotherPlugin”
plugin_dependencies=MyOtherPlugin==1.12,YetAnotherPlugin
```

#### 16.1.2.2 \_\_init\_\_.py

Python包需要此文件。此外，QGIS要求此文件包含一个`classFactory()`函数，该函数在插件被加载到QGIS时调用。它接收[`QgisInterface`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface)实例， 并且必须返回`mainplugin.py`中插件类的对象——在我们的例子中它被命名为`TestPlugin`（见下文）。`__init__.py`应该是这样的：

```python
def classFactory(iface):
	from .mainPlugin import TestPlugin
	return TestPlugin(iface)

# 任何其他初始化
```

#### 16.1.2.3 mainPlugin.py

这就是魔法发生的地方，下面就是魔法的例子:

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
        # 创建操作，它将启动插件配置
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
- `unload()` - >卸载插件时调用

在上面的例子中，[`addPluginToMenu()`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToMenu)被使用。这会将相应的菜单操作添加到**Plugins** 菜单中。存在额外的方法将操作（action）添加到不同菜单。以下是这些方法的列表：

- [`addPluginToRasterMenu()`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToRasterMenu)
- [`addPluginToVectorMenu()`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToVectorMenu)
- [`addPluginToDatabaseMenu()`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToDatabaseMenu)
- [`addPluginToWebMenu()`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToWebMenu)

它们都具有与[`addPluginToMenu()`](https://qgis.org/pyqgis/master/gui/QgisInterface.html#qgis.gui.QgisInterface.addPluginToMenu)方法相同的语法 。

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

#### 16.1.2.4 资源文件

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

!!! 提示

    在Windows环境中，尝试从CMD或Powershell运行pyrcc5可能会导致错误“Windows无法访问指定的设备，路径，或文件[...]”。最简单的解决方案可能是使用osgeo4wshell，但如果你愿意修改PATH环境变量或显式指定可执行文件的路径，你应该可以在`<Your QGIS Install Directory>\bin\pyrcc5.exe`找到它

就这些……没什么复杂的:)

如果你已正确完成所有操作，则应该能够在插件管理器中查找并加载插件，在点击工具栏图标或相应的菜单项时，可以在控制台中查看到消息。

在处理真正的插件时，最好将插件写入另一个（工作）目录并创建一个makefile，它将生成UI和资源文件并将插件安装到QGIS安装中。

### 16.1.3 文档

该插件的文档可以编写为HTML帮助文档。`qgis.utils`模块提供了一个函数，`showPluginHelp()`将打开帮助文档浏览器，与其他QGIS帮助文档的方式相同。

`showPluginHelp()`函数在与调用模块相同的目录中查找帮助文档。它会寻找`index-ll_cc.html`，`index-ll.html`，`index-en.html`，`index-en_us.html`和`index.html`，显示它找到的第一个文档。这`ll_cc`是QGIS语言环境，这允许文档有多个翻译。

`showPluginHelp()`函数还可以使用参数`packageName`——它显示指定插件的帮助文档，`filename`——可以替换被搜索的文件名中的“index”，section——它是html锚标记的名称，浏览器将定位到该位置。

### 16.1.4 翻译

通过几个步骤，你可以设置插件本地化的环境，以便根据计算机的区域，插件将以不同语言加载。

#### 16.1.4.1 软件要求

创建和管理所有翻译文件的最简单方法是安装 [Qt Linguist](https://doc.qt.io/qt-5/qtlinguist-index.html)。在基于Debian的GNU / Linux环境中，你可以这样安装它：

```shell
sudo apt-get install qttools5-dev-tools
```

#### 16.1.4.2 文件和目录

创建插件时，你将在主插件目录中找到该文件夹`i18n`。

**所有翻译文件都必须放在此目录中。**

##### 16.1.4.2.1 .pro文件

首先，你应该创建一个`.pro`文件，这是一个可以由**Qt Linguist**管理的*项目*文件。

在此`.pro`文件中，你必须指定要翻译的所有文件和窗体（.ui文件）。此文件用于设置本地化文件和变量。下面是一个项目文件，匹配我们的[示例插件](#16111)的结构 ：

```ini
FORMS = ../form.ui
SOURCES = ../your_plugin.py
TRANSLATIONS = your_plugin_it.ts
```

你的插件可能有更复杂的结构，并且可能分布在多个文件中。如果是这种情况，请记住，使用`pylupdate5`读取`.pro`文件并更新可翻译字符串，这不会扩展通配符，因此你需要将每个文件显式放在`.pro`文件中。你的项目文件可能看起来像这样：

```ini
FORMS = ../ui/about.ui ../ui/feedback.ui \
        ../ui/main_dialog.ui
SOURCES = ../your_plugin.py ../computation.py \
          ../utils.py
```

此外，`your_plugin.py`文件是*调用* QGIS工具栏中插件的所有菜单和子菜单的文件，你希望将它们全部翻译。

最后，使用*TRANSLATIONS*变量，你可以指定所需的翻译语言。

!!! 警告 warning

    确保`ts`文件名称为`your_plugin_` + `language` + `.ts`，否则语言将加载失败。使用两个字母的语言缩写（it对应Italian，de对应German，等等）

##### 16.1.4.2.2 .ts文件

创建`.pro`完成后，你就可以为插件的语言生成`.ts`文件了。

打开终端，转到`your_plugin/i18n`目录并输入：

```shell
pylupdate5 your_plugin.pro
```

你应该可以看到`your_plugin_language.ts`文件。

用**Qt Linguist**打开`.ts`文件并开始翻译。

##### 16.1.4.2.3 .qm文件

当你完成插件翻译时（如果某些字符串未完成，将使用这些字符串的源语言），你必须创建`.qm` 文件（将被QGIS使用的`.ts`编译文件）。

只需在`your_plugin/i18n`目录中打开终端cd 并输入：

```shell
lrelease your_plugin.ts
```

现在，在`i18n`目录中你将看到`your_plugin.qm`文件。

#### 16.1.4.3 使用MakeFile进行翻译

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

在此之后，你已在LOCALES变量中更新`.ts`了所有语言的文件。使用**Qt Linguist**翻译程序消息。完成翻译后，`.qm`可以通过`transcompile`创建：

```shell
make transcompile
```

你必须在你的插件中分发`.ts`文件。

#### 16.1.4.4 加载插件

要查看插件的翻译，只需打开QGIS，更改语言（**设置‣选项‣通用**）并重新启动QGIS。

你应该看到你的插件使用正确的语言。

!!! 警告 warning

    如果你改变了一些东西（新的UI，新的菜单，等等），你必须重新生成`.ts`和`.qm`文件，因此需要再一次执行以上命令。

### 16.1.5 提示和技巧

#### 16.1.5.1 插件重载

在开发插件期间，你经常需要在QGIS中重新加载它以进行测试。使用**Plugin Reloader**插件非常容易。你可以在[插件管理器](https://docs.qgis.org/testing/en/docs/user_manual/plugins/plugins.html#plugins)中找到它。

#### 16.1.5.2 访问插件

你可以使用python从QGIS中访问所有已安装插件类，这可以方便调试：

```python
my_plugin = qgis.utils.plugins['My Plugin']
```

#### 16.1.5.3 日志消息

插件在[日志消息面板中](https://docs.qgis.org/latest/en/docs/user_manual/introduction/general_tools.html#log-message-panel)有自己的选项卡。

#### 16.1.5.4 分享你的插件

QGIS在插件仓库中托管了数百个插件。考虑分享你的插件！它将扩展QGIS，人们将能够从你的代码中学习。可以使用插件管理器在QGIS中找到并安装所有托管的插件。

信息和要求：[plugins.qgis.org](https://plugins.qgis.org/)。

## 16.2 代码片段

本节以代码片段为例，讲解插件开发

### 16.2.1 如何通过快捷键调用方法

在`initGui()`中添加：

```python
self.key_action = QAction("Test Plugin", self.iface.mainWindow())
self.iface.registerMainWindowAction(self.key_action, "Ctrl+I")  # 操作被Ctrl+I触发
self.iface.addPluginToMenu("&Test plugins", self.key_action)
self.key_action.triggered.connect(self.key_action_triggered)
```

在`unload()`中添加：

```python
self.iface.unregisterMainWindowAction(self.keyAction)
```

按下`CTRL+I`时调用方法：

```python
def key_action_triggered(self):
    QMessageBox.information(self.iface.mainWindow(),"Ok", "You pressed Ctrl+I")
```

### 16.2.2 如何切换图层

图例中有一个访问图层的API。下面是如何切换当前图层可见性的示例：

```python
root = QgsProject.instance().layerTreeRoot()
node = root.findLayer(iface.activeLayer().id())
new_state = Qt.Checked if node.isVisible() == Qt.Unchecked else Qt.Unchecked
node.setItemVisibilityChecked(new_state)
```

### 16.2.3 如何访问所选要素的属性表

```python
def change_value(value):
    """改变所选要素第二字段值

    :param value: The new value.
    """
    layer = iface.activeLayer()
    if layer:
        count_selected = layer.selectedFeatureCount()
        if count_selected > 0:
            layer.startEditing()
            id_features = layer.selectedFeatureIds()
            for i in id_features:
                layer.changeAttributeValue(i, 1, value) # 1 being the second column
            layer.commitChanges()
        else:
            iface.messageBar().pushCritical("Error",
                "Please select at least one feature from current layer")
    else:
        iface.messageBar().pushCritical("Error", "Please select a layer")

# 该方法需要一个参数（所选要素第二个字段的新值），可以通过调用以下方法：
changeValue(50)
```

### 16.2.4 选项对话框中的插件接口

你可以在**设置‣选项**中添加一个自定义插件选项标签。这比为你的插件选项添加一个特定的主菜单条目更可取，因为它将所有的QGIS应用程序设置和插件设置保存在一个单一的地方，便于用户发现和导航。

下面的代码片段将为插件的设置添加一个新的空白选项卡，为你填充所有选项和你的插件特定设置做好准备。你可以将下面的类拆分成不同的文件。在这个例子中，我们在mainPlugin.py文件中添加了两个类。

```python
class MyPluginOptionsFactory(QgsOptionsWidgetFactory):

    def __init__(self):
        super().__init__()

    def icon(self):
        return QIcon('icons/my_plugin_icon.svg')

    def createWidget(self, parent):
        return ConfigOptionsPage(parent)


class ConfigOptionsPage(QgsOptionsPageWidget):

    def __init__(self, parent):
        super().__init__(parent)
        layout = QHBoxLayout()
        layout.setContentsMargins(0, 0, 0, 0)
        self.setLayout(layout)
```

最后我们添加导入和修改`__init__`函数：

```python
from qgis.gui import QgsOptionsWidgetFactory, QgsOptionsPageWidget


class MyPlugin:
    def __init__(self, iface):
        """构造函数.

        :param iface: 将传递给该类的接口实例它提供了一个钩子，你可以通过它来操作QGIS运行时应用程序。
        :type iface: QgsInterface
        """
        # 保存引用
        self.iface = iface


    def initGui(self):
        self.options_factory = MyPluginOptionsFactory()
        self.options_factory.setTitle(self.tr('My Plugin'))
        iface.registerOptionsWidgetFactory(self.options_factory)

    def unload(self):
        iface.unregisterOptionsWidgetFactory(self.options_factory)
```

!!! 提示

    **将自定义选项卡添加到矢量图层属性对话框** 
    
    你可以应用类似的逻辑，使用[QgsMapLayerConfigWidgetFactory](https://qgis.org/pyqgis/master/gui/QgsMapLayerConfigWidgetFactory.html#qgis.gui.QgsMapLayerConfigWidgetFactory)和[QgsMapLayerConfigWidget](https://qgis.org/pyqgis/master/gui/QgsMapLayerConfigWidget.html#qgis.gui.QgsMapLayerConfigWidget)类将插件自定义选项添加到图层属性对话框中。

## 16.3 编写和调试插件的IDE设置

**TODO**

## 16.4 发布你的插件

一旦你的插件准备好了，并且你认为这个插件可能对某些人有帮助，不要犹豫，把它上传到[官方Python插件仓库](https://docs.qgis.org/testing/en/docs/pyqgis_developer_cookbook/plugins/releasing.html#official-pyqgis-repository)。在该页面上，你还可以找到关于如何准备插件以与插件安装程序良好配合的打包指南。或者，如果你想建立自己的插件库，可以创建一个简单的XML文件，列出插件及其元数据。

请特别注意一下建议：

### 16.4.1 元数据和名称

- 避免使用与现有插件过于相似的名称
- 如果你的插件与现有的插件有类似的功能，请在 "关于 "一栏中解释其区别，这样用户就会知道使用哪一个，而不需要安装和测试。
- 避免在插件本身的名称中重复使用"plugin"。
- 使用元数据中的描述字段进行单行描述，使用关于字段进行更详细的说明
- 包括一个代码库、一个错误跟踪器和一个主页；这将极大地提高合作的可能性，并且可以通过现有的网络基础设施（GitHub、GitLab、Bitbucket等）非常容易地完成。
- 谨慎选择标签：避免不具参考价值的标签（如vector），最好选择已经被他人使用的标签（见插件网站）。
- 添加一个合适的图标，不要使用默认的图标；参见QGIS界面，了解要使用的风格建议

### 16.4.2 代码和帮助

- 不要把生成的文件（ui_*.py, resources_rc.py, 生成的帮助文件...）和无用的东西（如.gitignore）包括在版本库中

- 将插件添加到适当的菜单中（Vector, Raster, Web, Database）。

- 在适当的时候（执行分析的插件），考虑将插件添加为Processing框架的子插件：这将允许用户批量运行它，将它集成到更复杂的工作流中，并将你从设计界面的负担中解放出来

- 至少包括最基本的文档，如果对测试和理解有用的话，还包括样本数据。

### 16.4.3 官方python插件仓库

你可以找到官方python插件仓库：https://plugins.qgis.org/。

为了使用官方python插件仓库，你必须从[OSGEO web portal](https://www.osgeo.org/community/getting-started-osgeo/osgeo_userid/)获得OSGEO ID。

一旦你上传了你的插件，它将得到一个工作人员的批准，你会得到通知。

#### 16.4.3.1 权限

这些规则已经在官方插件库中实现：

- 每个注册用户都可以添加一个新的插件

- 员工用户可以批准或不批准所有的插件版本

- 拥有特殊权限`plugins.can_approve`的用户可以自动批准他们上传的版本

- 拥有特殊权限`plugins.can_approve`的用户可以批准其他人上传的版本，只要他们在插件所有者的列表中。

- 一个特定的插件只能由员工用户和插件所有者删除和编辑。

- 如果一个没有`plugins.can_approve`权限的用户上传了一个新的版本，该插件的版本会自动取消审批。

#### 16.4.3.2 信任管理

工作人员可以通过前端应用程序设置`plugins.can_approve`权限，向选定的插件创建者授予信任。

插件详情视图提供了直接链接，以授予对插件创建者或插件所有者的信任。

#### 16.4.3.3 验证

在上传插件时，插件的元数据会自动从压缩包中导入并进行验证。

这里有一些验证规则，当你想在官方仓库上传一个插件时，你应该注意：

1. 包含插件的主文件夹的名称必须只包含ASCII字符（A-Z和a-z）、数字和下划线（_）和减号（-），而且不能以数字开头。
2. `metadata.txt`是必需的
3. 元数据表中列出的所有必需的元数据都必须存在
4. 元数据`version`字段必须是唯一的

#### 16.4.3.4 插件结构

按照验证规则，你的插件的压缩包（.zip）必须有一个特定的结构，才能作为一个功能性插件进行验证。由于该插件将被解压在用户的plugins文件夹内，它必须在.zip文件内有自己的目录，以不干扰其他插件。必需的文件有：`metadata.txt` 和 `__init__.py`。但如果能有一个README，当然还有一个代表该插件的图标（resources.qrc），那就更好了。以下是一个plugin.zip的例子，它应该是这样的。

```
plugin.zip
  pluginfolder/
  |-- i18n
  |   |-- translation_file_de.ts
  |-- img
  |   |-- icon.png
  |   `-- iconsource.svg
  |-- __init__.py
  |-- Makefile
  |-- metadata.txt
  |-- more_code.py
  |-- main_code.py
  |-- README
  |-- resources.qrc
  |-- resources_rc.py
  `-- ui_Qt_user_interface_file.ui
```


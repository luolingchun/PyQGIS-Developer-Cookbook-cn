# 21 PyQGIS速查表

## 21.1 用户接口

**改变外观**

```python
from qgis.PyQt.QtWidgets import QApplication

app = QApplication.instance()
app.setStyleSheet(".QWidget {color: blue; background-color: yellow;}")
# 你可以从文件读取样式
with  open("testdata/file.qss") as qss_file_content:
    app.setStyleSheet(qss_file_content.read())
```
**改变图标和标题**

```python
from qgis.PyQt.QtGui import QIcon

icon = QIcon(r"/path/to/logo/file.png")
iface.mainWindow().setWindowIcon(icon)
iface.mainWindow().setWindowTitle("My QGIS")
```
## 21.2 设置

**获得设置列表**

```python
from qgis.PyQt.QtCore import QSettings

qs = QSettings()

for k in sorted(qs.allKeys()):
    print (k)
```
## 21.3 工具栏
**移除工具栏**

```python
from qgis.utils import iface

toolbar = iface.helpToolBar()
parent = toolbar.parentWidget()
parent.removeToolBar(toolbar)

# 添加
parent.addToolBar(toolbar)
```
**移除操作**

```python
actions = iface.attributesToolBar().actions()
iface.attributesToolBar().clear()
iface.attributesToolBar().addAction(actions[4])
iface.attributesToolBar().addAction(actions[3])
```
## 21.4 菜单

**移除菜单**

```python
from qgis.utils import iface

# 帮助菜单
menu = iface.helpMenu()
menubar = menu.parentWidget()
menubar.removeAction(menu.menuAction())

# 添加
menubar.addAction(menu.menuAction())
```
## 21.5 画布
**访问画布**

```python
from qgis.utils import iface

canvas = iface.mapCanvas()
```
**改变画布颜色**

```python
from qgis.PyQt.QtCore import Qt

iface.mapCanvas().setCanvasColor(Qt.black)
iface.mapCanvas().refresh()
```
**画布刷新间隔**

```python
from qgis.PyQt.QtCore import QSettings
# 150毫秒
QSettings().setValue("/qgis/map_update_interval", 150)
```
## 21.6 图层
**添加图层**

```python
from qgis.utils import iface

layer = iface.addVectorLayer("/path/to/shapefile/file.shp", "layer name you like", "ogr")
if not layer:
    print("Layer failed to load!")
```
**获取当前图层**

```python
layer = iface.activeLayer()
```
**图层列表**

```python
from qgis.core import QgsProject

QgsProject.instance().mapLayers().values()
```

**获得图层名称**

```python
from qgis.core import QgsVectorLayer
layer = QgsVectorLayer("Point?crs=EPSG:4326", "layer name you like", "memory")
QgsProject.instance().addMapLayer(layer)

layers_names = []
for layer in QgsProject.instance().mapLayers().values():
    layers_names.append(layer.name())

print("layers TOC = {}".format(layers_names))

layers_names = [layer.name() for layer in QgsProject.instance().mapLayers().values()]
print("layers TOC = {}".format(layers_names))
```
**通过名称查找图层**

```python
from qgis.core import QgsProject

layer = QgsProject.instance().mapLayersByName("layer name you like")[0]
print(layer.name())
```
**设置当前图层**

```python
from qgis.core import QgsProject

layer = QgsProject.instance().mapLayersByName("layer name you like")[0]
iface.setActiveLayer(layer)
```
**刷新图层**

```python
from qgis.core import QgsProject

layer = QgsProject.instance().mapLayersByName("layer name you like")[0]
# 5秒
layer.setAutoRefreshInterval(5000)
# 自动刷新
layer.setAutoRefreshEnabled(True)
```
**添加表单要素**

```python
from qgis.core import QgsFeature, QgsGeometry

feat = QgsFeature()
geom = QgsGeometry()
feat.setGeometry(geom)
feat.setFields(layer.fields())

iface.openFeatureForm(layer, feat, False)
```
**添加无表单要素**

```python
from qgis.core import QgsPointXY

pr = layer.dataProvider()
feat = QgsFeature()
feat.setGeometry(QgsGeometry.fromPointXY(QgsPointXY(10,10)))
pr.addFeatures([feat])
```
**获得要素**

```python
for f in layer.getFeatures():
    print (f)
```
**获得已选要素**

```python
for f in layer.selectedFeatures():
    print (f)
```
**获得要素id**

```python
selected_ids = layer.selectedFeatureIds()
print(selected_ids)
```
**从已选要素id创建内存图层**

```python
from qgis.core import QgsFeatureRequest

memory_layer = layer.materialize(QgsFeatureRequest().setFilterFids(layer.selectedFeatureIds()))
QgsProject.instance().addMapLayer(memory_layer)
```
**获得几何**

```python
# 点图层
for f in layer.getFeatures():
    geom = f.geometry()
    print ('%f, %f' % (geom.asPoint().y(), geom.asPoint().x()))
```
**移动几何**

```python
from qgis.core import QgsFeature, QgsGeometry
poly = QgsFeature()
geom = QgsGeometry.fromWkt("POINT(7 45)")
geom.translate(1, 1)
poly.setGeometry(geom)
print(poly.geometry())
```
**设置坐标参考系统**

```python
from qgis.core import QgsProject, QgsCoordinateReferenceSystem

for layer in QgsProject.instance().mapLayers().values():
    layer.setCrs(QgsCoordinateReferenceSystem(4326, QgsCoordinateReferenceSystem.EpsgCrsId))
```
**查看坐标参考系统**

```python
from qgis.core import QgsProject

for layer in QgsProject.instance().mapLayers().values():
    crs = layer.crs().authid()
    layer.setName('{} ({})'.format(layer.name(), crs))
```
**隐藏字段**

```python
from qgis.core import QgsEditorWidgetSetup

def fieldVisibility (layer,fname):
    setup = QgsEditorWidgetSetup('Hidden', {})
    for i, column in enumerate(layer.fields()):
        if column.name()==fname:
            layer.setEditorWidgetSetup(idx, setup)
            break
        else:
            continue
```
**图层添加WKT要素**

```python
from qgis.core import QgsVectorLayer, QgsFeature, QgsGeometry, QgsProject

layer = QgsVectorLayer('Polygon?crs=epsg:4326', 'Mississippi', 'memory')
pr = layer.dataProvider()
poly = QgsFeature()
geom = QgsGeometry.fromWkt("POLYGON ((-88.82 34.99,-88.0934.89,-88.39 30.34,-89.57 30.18,-89.73 31,-91.63 30.99,-90.8732.37,-91.23 33.44,-90.93 34.23,-90.30 34.99,-88.82 34.99))")
poly.setGeometry(geom)
pr.addFeatures([poly])
layer.updateExtents()
QgsProject.instance().addMapLayers([layer])
```
**从GeoPackage加载所有图层**

```python
from qgis.core import QgsVectorLayer, QgsProject

fileName = "/path/to/gpkg/file.gpkg"
layer = QgsVectorLayer(fileName,"test","ogr")
subLayers =layer.dataProvider().subLayers()

for subLayer in subLayers:
    name = subLayer.split('!!::!!')[1]
    uri = "%s|layername=%s" % (fileName, name,)
    # 创建图层
    sub_vlayer = QgsVectorLayer(uri, name, 'ogr')
    # Add layer to map
    QgsProject.instance().addMapLayer(sub_vlayer)
```
**加载瓦片图层（xyz）**

```python
from qgis.core import QgsRasterLayer, QgsProject

def loadXYZ(url, name):
    rasterLyr = QgsRasterLayer("type=xyz&url=" + url, name, "wms")
    QgsProject.instance().addMapLayer(rasterLyr)

urlWithParams = 'type=xyz&url=https://a.tile.openstreetmap.org/%7Bz%7D/%7Bx%7D/%7By%7D.png&zmax=19&zmin=0&crs=EPSG3857'
loadXYZ(urlWithParams, 'OpenStreetMap')
```
**移除所有图层**

```python
QgsProject.instance().removeAllMapLayers()
```
**移除全部**

```python
QgsProject.instance().clear()
```
## 21.7 目录
**访问选中的图层**

```python
from qgis.utils import iface

iface.mapCanvas().layers()
```
**移除上下文菜单**

```python
ltv = iface.layerTreeView()
mp = ltv.menuProvider()
ltv.setMenuProvider(None)
# 恢复
ltv.setMenuProvider(mp)
```
## 21.8 高级目录
**根节点**

```python
from qgis.core import QgsVectorLayer, QgsProject, QgsLayerTreeLayer

root = QgsProject.instance().layerTreeRoot()
node_group = root.addGroup("My Group")

layer = QgsVectorLayer("Point?crs=EPSG:4326", "layer name you like", "memory")
QgsProject.instance().addMapLayer(layer, False)

node_group.addLayer(layer)

print(root)
print(root.children())
```
**访问第一个子节点**

```python
from qgis.core import QgsLayerTreeGroup, QgsLayerTreeLayer, QgsLayerTree

child0 = root.children()[0]
print (child0.name())
print (type(child0))
print (isinstance(child0, QgsLayerTreeLayer))
print (isinstance(child0.parent(), QgsLayerTree))
```
**查找图层组和所有节点**

```python
from qgis.core import QgsLayerTreeGroup, QgsLayerTreeLayer

def get_group_layers(group):
   print('- group: ' + group.name())
   for child in group.children():
      if isinstance(child, QgsLayerTreeGroup):
         # 遍历嵌套图层组
         get_group_layers(child)
      else:
         print('  - layer: ' + child.name())


root = QgsProject.instance().layerTreeRoot()
for child in root.children():
   if isinstance(child, QgsLayerTreeGroup):
      get_group_layers(child)
   elif isinstance(child, QgsLayerTreeLayer):
      print ('- layer: ' + child.name())
```
**通过名称查找图层组**

```python
print (root.findGroup("My Group"))
```
**通过id查找图层组**

```python
print (root.findLayer(layer.layerId()))
```
**添加图层**

```python
from qgis.core import QgsVectorLayer, QgsProject

layer1 = QgsVectorLayer("Point?crs=EPSG:4326", "layer name you like", "memory")
QgsProject.instance().addMapLayer(layer1, False)
node_layer1 = root.addLayer(layer1)
```
**添加图层组**

```python
from qgis.core import QgsLayerTreeGroup

node_group2 = QgsLayerTreeGroup("Group 2")
root.addChildNode(node_group2)
```
**移除加载的图层**

```python
layer = QgsProject.instance().mapLayersByName("layer name you like")[0]
root = QgsProject.instance().layerTreeRoot()

myLayer = root.findLayer(layer.id())
myClone = myLayer.clone()
parent = myLayer.parent()

myGroup = root.findGroup("My Group")
# 插入到第一个位置
myGroup.insertChildNode(0, myClone)

parent.removeChildNode(myLayer)
```
**移除指定图层组**

```python
QgsProject.instance().addMapLayer(layer, False)

root = QgsProject.instance().layerTreeRoot()
myGroup = root.findGroup("My Group")
myOriginalLayer = root.findLayer(layer.id())
myLayer = myOriginalLayer.clone()
myGroup.insertChildNode(0, myLayer)
parent.removeChildNode(myOriginalLayer)
```
**改变可见性**

```python
myGroup.setItemVisibilityChecked(False)
myLayer.setItemVisibilityChecked(False)
```

**图层组是否被选择**

```python
def isMyGroupSelected( groupName ):
    myGroup = QgsProject.instance().layerTreeRoot().findGroup( groupName )
    return myGroup in iface.layerTreeView().selectedNodes()

print(isMyGroupSelected( 'my group name' ))
```

**移动节点**

```python
cloned_group1 = node_group.clone()
root.insertChildNode(0, cloned_group1)
root.removeChildNode(node_group)
```
**展开节点**

```python
print(myGroup.isExpanded())
myGroup.setExpanded(False)
```

**隐藏节点**

```python
from qgis.core import QgsProject

model = iface.layerTreeView().layerTreeModel()
ltv = iface.layerTreeView()
root = QgsProject.instance().layerTreeRoot()

layer = QgsProject.instance().mapLayersByName('layer name you like')[0]
node = root.findLayer( layer.id())

index = model.node2index( node )
ltv.setRowHidden( index.row(), index.parent(), True )
node.setCustomProperty( 'nodeHidden', 'true')
ltv.setCurrentIndex(model.node2index(root))
```

**节点信号**

```python
def onWillAddChildren(node, indexFrom, indexTo):
    print ("WILL ADD", node, indexFrom, indexTo)

def onAddedChildren(node, indexFrom, indexTo):
    print ("ADDED", node, indexFrom, indexTo)

root.willAddChildren.connect(onWillAddChildren)
root.addedChildren.connect(onAddedChildren)
```

**移除图层**

```python
root.removeLayer(layer)
```

**移除图层组**

```python
root.removeChildNode(node_group2)
```

**创建新的目录树**

```python
root = QgsProject.instance().layerTreeRoot()
model = QgsLayerTreeModel(root)
view = QgsLayerTreeView()
view.setModel(model)
view.show(
```

**移动节点**

```python
cloned_group1 = node_group.clone()
root.insertChildNode(0, cloned_group1)
root.removeChildNode(node_group)
```

**重命名节点**

```python
cloned_group1.setName("Group X")
node_layer1.setName("Layer X")
```
## 21.9 处理算法
**获得算法列表**

```python
from qgis.core import QgsApplication

for alg in QgsApplication.processingRegistry().algorithms():
    if 'buffer' == alg.name():
        print("{}:{} --> {}".format(alg.provider().name(), alg.name(), alg.displayName()))
```
**获得算法帮助**

```python
from qgis import processing

processing.algorithmHelp("qgis:randomselection")
```
**运行算法**

本示例，结果存储在添加到项目的临时内存层中。

```python
from qgis import processing
result = processing.run("native:buffer", {'INPUT': layer, 'OUTPUT': 'memory:'})
QgsProject.instance().addMapLayer(result['OUTPUT'])
```
**算法统计**

```python
from qgis.core import QgsApplication

len(QgsApplication.processingRegistry().algorithms())
```
**数据提供者统计**

```python
from qgis.core import QgsApplication

len(QgsApplication.processingRegistry().providers())
```
**表达式统计**

```python
from qgis.core import QgsExpression

len(QgsExpression.Functions())
```
## 21.10 装饰器
**版权**

```python
from qgis.PyQt.Qt import QTextDocument
from qgis.PyQt.QtGui import QFont

mQFont = "Sans Serif"
mQFontsize = 9
mLabelQString = "© QGIS 2019"
mMarginHorizontal = 0
mMarginVertical = 0
mLabelQColor = "#FF0000"

INCHES_TO_MM = 0.0393700787402 # 1 毫米 = 0.0393700787402 英寸
case = 2

def add_copyright(p, text, xOffset, yOffset):
    p.translate( xOffset , yOffset  )
    text.drawContents(p)
    p.setWorldTransform( p.worldTransform() )

def _on_render_complete(p):
    deviceHeight = p.device().height() # 获取绘制设备高度
    deviceWidth  = p.device().width() # 获取绘制设备宽度
    # 创建一个新的富文本容器
    text = QTextDocument()
    font = QFont()
    font.setFamily(mQFont)
    font.setPointSize(int(mQFontsize))
    text.setDefaultFont(font)
    style = "<style type=\"text/css\"> p {color: " + mLabelQColor + "}</style>"
    text.setHtml( style + "<p>" + mLabelQString + "</p>" )
    # 文本大小
    size = text.size()

    # 渲染
    pixelsInchX  = p.device().logicalDpiX()
    pixelsInchY  = p.device().logicalDpiY()
    xOffset  = pixelsInchX  * INCHES_TO_MM * int(mMarginHorizontal)
    yOffset  = pixelsInchY  * INCHES_TO_MM * int(mMarginVertical)

    # 计算点位
    if case == 0:
        # 左上
        add_copyright(p, text, xOffset, yOffset)

    elif case == 1:
        # 左下
        yOffset = deviceHeight - yOffset - size.height()
        add_copyright(p, text, xOffset, yOffset)

    elif case == 2:
        # 右上
        xOffset  = deviceWidth  - xOffset - size.width()
        add_copyright(p, text, xOffset, yOffset)

    elif case == 3:
        # 右下
        yOffset  = deviceHeight - yOffset - size.height()
        xOffset  = deviceWidth  - xOffset - size.width()
        add_copyright(p, text, xOffset, yOffset)

    elif case == 4:
        # 上中心点
        xOffset = deviceWidth / 2
        add_copyright(p, text, xOffset, yOffset)

    else:
        # 下中心点
        yOffset = deviceHeight - yOffset - size.height()
        xOffset = deviceWidth / 2
        add_copyright(p, text, xOffset, yOffset)

# 当画布渲染完成后发送信号
iface.mapCanvas().renderComplete.connect(_on_render_complete)
# 重绘画布
iface.mapCanvas().refresh()
```

## 21.11 创作者

**按名称获取打印布局**

```python
composerTitle = 'MyComposer' # 创作者名称

project = QgsProject.instance()
projectLayoutManager = project.layoutManager()
layout = projectLayoutManager.layoutByName(composerTitle)
```

## 21.12 来源

- [QGIS Python (PyQGIS) API](https://qgis.org/pyqgis/master/)
- [QGIS C++ API](https://qgis.org/api/)
- [StackOverFlow QGIS questions](https://stackoverflow.com/questions/tagged/qgis)
- [Script by Klas Karlsson](https://raw.githubusercontent.com/klakar/QGIS_resources/master/collections/Geosupportsystem/python/qgis_basemaps.py)


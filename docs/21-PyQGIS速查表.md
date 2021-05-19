# 21 PyQGIS速查表

## 21.1 用户接口

**改变外观**

```python
from qgis.PyQt.QtWidgets import QApplication

app = QApplication.instance()
qss_file = open(r"/path/to/style/file.qss").read()
app.setStyleSheet(qss_file)
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

# and add again
parent.addToolBar(toolbar)
```
**移除动作**

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

# for example Help Menu
menu = iface.helpMenu()
menubar = menu.parentWidget()
menubar.removeAction(menu.menuAction())

# and add again
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
# Set milliseconds (150 milliseconds)
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
**获得图层名称**

```python
layers_names = []
for layer in QgsProject.instance().mapLayers().values():
    layers_names.append(layer.name())

print("layers TOC = {}".format(layers_names))

# otherwise
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
# Set seconds (5 seconds)
layer.setAutoRefreshInterval(5000)
# Enable auto refresh
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
获得几何

```python
# Point layer
for f in layer.getFeatures():
    geom = f.geometry()
    print ('%f, %f' % (geom.asPoint().y(), geom.asPoint().x()))
```
**移动几何**

```python
geom.translate(100, 100)
poly.setGeometry(geom)
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
    # Create layer
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
# Restore
ltv.setMenuProvider(mp)
```
## 21.8 高级目录
**根节点**

```python
from qgis.core import QgsProject

root = QgsProject.instance().layerTreeRoot()
print (root)
print (root.children())
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
         # Recursive call to get nested groups
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
**移除图层**

```python
root.removeLayer(layer1)
```
**移除图层组**

```python
root.removeChildNode(node_group2)
```
**移动节点**

```python
cloned_group1 = node_group.clone()
root.insertChildNode(0, cloned_group1)
root.removeChildNode(node_group)
```
重命名节点

```python
cloned_group1.setName("Group X")
node_layer1.setName("Layer X")
```
**移动加载的图层**

```python
layer = QgsProject.instance().mapLayersByName("layer name you like")[0]
root = QgsProject.instance().layerTreeRoot()

mylayer = root.findLayer(layer.id())
myClone = mylayer.clone()
parent = mylayer.parent()

group = root.findGroup("My Group")
# Insert in first position
group.insertChildNode(0, myClone)

parent.removeChildNode(mylayer)
```
**加载指定图层组**

```python
QgsProject.instance().addMapLayer(layer, False)

root = QgsProject.instance().layerTreeRoot()
g = root.findGroup("My Group")
g.insertChildNode(0, QgsLayerTreeLayer(layer))
```
**改变可视性**

```python
print (cloned_group1.isVisible())
cloned_group1.setItemVisibilityChecked(False)
node_layer1.setItemVisibilityChecked(False)
```
**判断图层组是否选中**

```python
def isMyGroupSelected( groupName ):
    myGroup = QgsProject.instance().layerTreeRoot().findGroup( groupName )
    return myGroup in iface.layerTreeView().selectedNodes()

print (isMyGroupSelected( 'my group name' ))
```
**展开节点**

```python
print (cloned_group1.isExpanded())
cloned_group1.setExpanded(False)
```
**隐藏节点**

```python
from qgis.core import QgsProject

model = iface.layerTreeView().layerTreeModel()
ltv = iface.layerTreeView()
root = QgsProject.instance().layerTreeRoot()

layer = QgsProject.instance().mapLayersByName('layer name you like')[0]
node=root.findLayer( layer.id())

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
**创建新目录**

```python
from qgis.core import QgsProject, QgsLayerTreeModel
from qgis.gui import QgsLayerTreeView

root = QgsProject.instance().layerTreeRoot()
model = QgsLayerTreeModel(root)
view = QgsLayerTreeView()
view.setModel(model)
view.show()
```
## 21.9 处理算法
**获得算法列表**

```python
from qgis.core import QgsApplication

for alg in QgsApplication.processingRegistry().algorithms():
    print("{}:{} --> {}".format(alg.provider().name(), alg.name(), alg.displayName()))

# otherwise
def alglist():
    s = ''
    for i in QgsApplication.processingRegistry().algorithms():
        l = i.displayName().ljust(50, "-")
        r = i.id()
        s += '{}--->{}\n'.format(l, r)
    print(s)
```
**获得算法帮助**

```python
from qgis import processing

processing.algorithmHelp("qgis:randomselection")
```
**运行算法**

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
## 21.10 装饰
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

INCHES_TO_MM = 0.0393700787402 # 1 millimeter = 0.0393700787402 inches
case = 2

def add_copyright(p, text, xOffset, yOffset):
    p.translate( xOffset , yOffset  )
    text.drawContents(p)
    p.setWorldTransform( p.worldTransform() )

def _on_render_complete(p):
    deviceHeight = p.device().height() # Get paint device height on which this painter is currently painting
    deviceWidth  = p.device().width() # Get paint device width on which this painter is currently painting
    # Create new container for structured rich text
    text = QTextDocument()
    font = QFont()
    font.setFamily(mQFont)
    font.setPointSize(int(mQFontsize))
    text.setDefaultFont(font)
    style = "<style type=\"text/css\"> p {color: " + mLabelQColor + "}</style>"
    text.setHtml( style + "<p>" + mLabelQString + "</p>" )
    # Text Size
    size = text.size()

    # RenderMillimeters
    pixelsInchX  = p.device().logicalDpiX()
    pixelsInchY  = p.device().logicalDpiY()
    xOffset  = pixelsInchX  * INCHES_TO_MM * int(mMarginHorizontal)
    yOffset  = pixelsInchY  * INCHES_TO_MM * int(mMarginVertical)

    # Calculate positions
    if case == 0:
        # Top Left
        add_copyright(p, text, xOffset, yOffset)

    elif case == 1:
        # Bottom Left
        yOffset = deviceHeight - yOffset - size.height()
        add_copyright(p, text, xOffset, yOffset)

    elif case == 2:
        # Top Right
        xOffset  = deviceWidth  - xOffset - size.width()
        add_copyright(p, text, xOffset, yOffset)

    elif case == 3:
        # Bottom Right
        yOffset  = deviceHeight - yOffset - size.height()
        xOffset  = deviceWidth  - xOffset - size.width()
        add_copyright(p, text, xOffset, yOffset)

    elif case == 4:
        # Top Center
        xOffset = deviceWidth / 2
        add_copyright(p, text, xOffset, yOffset)

    else:
        # Bottom Center
        yOffset = deviceHeight - yOffset - size.height()
        xOffset = deviceWidth / 2
        add_copyright(p, text, xOffset, yOffset)

# Emitted when the canvas has rendered
iface.mapCanvas().renderComplete.connect(_on_render_complete)
# Repaint the canvas map
iface.mapCanvas().refresh()
```

## 21.11 创作者

**按名称获取打印布局**

```python
composerTitle = 'MyComposer' # Name of the composer

project = QgsProject.instance()
projectLayoutManager = project.layoutManager()
layout = projectLayoutManager.layoutByName(composerTitle)
```

## 21.12 来源

- [QGIS Python (PyQGIS) API](https://qgis.org/pyqgis/master/)
- [QGIS C++ API](https://qgis.org/api/)
- [StackOverFlow QGIS questions](https://stackoverflow.com/questions/tagged/qgis)
- [Script by Klas Karlsson](https://raw.githubusercontent.com/klakar/QGIS_resources/master/collections/Geosupportsystem/python/qgis_basemaps.py)
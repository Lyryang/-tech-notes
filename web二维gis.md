# GeoJSON

一整个图层的数据，就可以用一份 GeoJSON 来描述。

> - 一份 GeoJSON 文件 ≈ 一个图层
> - `FeatureCollection` ≈ 这个图层里**所有要素的列表**
> - `features` 数组里的每一项 `Feature` ≈ 地图上的**一个具体对象**（一条路、一个面、一个点）

FeatureCollection，类比成这样一个图层

<img src="C:./Typora图片/image-20251126171232037.png" alt="image-20251126171232037" style="zoom: 33%;" />

features存放着各种具体的对象

<img src="C:./Typora图片/image-20251126171336319.png" alt="image-20251126171336319" style="zoom:50%;" />



## 示例

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Point",
        "coordinates": [114.304184, 30.611725]
      }
    },
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [114.28276, 30.599972],
          [114.302054, 30.587785],
          [114.30882, 30.600295]
        ]
      }
    },
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [114.299549, 30.55564],
            [114.299549, 30.565781],
            [114.314458, 30.565781],
            [114.314458, 30.55564],
            [114.299549, 30.55564]
          ]
        ]
      }
    }
  ]
}
```

分别对应点、线、面

<img src="C:./Typora图片/image-20251126172046089.png" alt="image-20251126172046089" style="zoom: 50%;" />



## 拆解geojson

FeatureCollection：**一整个图层**

```json
{
  "type": "FeatureCollection",
  "features": [ ... ]
}
```

> “这是一个要素集合，这份文件里装着一堆 Feature。”

- 在 OpenLayers / Mapbox GL 里，通常你会把这一坨当成**一个数据源 / 一个图层的数据**。

------

features：**这个图层里所有“对象”的列表**

```json
"features": [
  { "type": "Feature", ... },
  { "type": "Feature", ... },
  ...
]
```

- 这个数组里

  每一个元素

  就是地图上的一个可见“对象”：

  - 一个学校点
  - 一个道路线
  - 一个行政区多边形

- 这些就是 OpenLayers / Mapbox 里说的 “Feature”。

------

Feature：**地图上的一个具体东西**

```json
{
  "type": "Feature",
  "geometry": { ... },
  "properties": { ... }
}
```

- `Feature` = “空间信息 + 属性信息”的组合：
  - `geometry`：这个东西在地图上在哪里，长什么形状。
  - `properties`：这个东西叫什么、编号多少、属于哪一类。

可以看举例数据的拆解

------

geometry：**几何形状 = 点/线/面**

```json
"geometry": {
  "type": "Point",         // 几何类型
  "coordinates": [116.39, 39.9]  // 坐标
}
```

这里最关键的是两件事：

1. `type` 决定“形状类型”：
   - `"Point"`：一个点（比如一个 POI）
   - `"LineString"`：一条折线（比如一条路、一条河）
   - `"Polygon"`：一个多边形（比如一个小区、一个行政区）
   - `"MultiPoint" / "MultiLineString" / "MultiPolygon"`：一堆点/线/面
   - `"GeometryCollection"`：多个几何放在一起的集合（用得少）
2. `coordinates` 是坐标数组：
   - 点：`[lon, lat]`
   - 线：`[[lon1, lat1], [lon2, lat2], ...]`
   - 面：`[[[lon1, lat1], [lon2, lat2], ... , [lon1, lat1]]]`（最外一层是环）

> geometry 就是“这条路在哪儿、这个点在哪儿、这个面多大多复杂”。

------

properties：**和位置无关的描述信息**

```json
"properties": {
  "name": "清华大学",
  "type": "大学",
  "id": 1
}
```

- 这里面放的都是“属性表”里的字段：
  - 名称、编号、类型、状态、人口数、楼层数……
- 显示 popup、做筛选、分类渲染，都是靠 properties。

和 geometry 的分工非常清晰：

- `geometry`：位置 & 形状；
- `properties`：属性 & 业务信息。

它就是属性表每一列对应的字段





## 对应openlayers

OpenLayers 的矢量数据源（`VectorSource`）和 GeoJSON 的数据结构在概念和层级上是高度对应的

```js
GeoJSON结构                         OpenLayers结构
---------------------------------------------------
FeatureCollection                  VectorSource（ol/source/Vector）openlayer的矢量数据源
  ├─ features: Feature[]           ├─ features: Feature[]（ol/Feature）
      ├─ Feature                   ├─ Feature（ol/Feature）  
          ├─ geometry              ├─ geometry（ol/geom/XXX）
          │    ├─ Point            │    ├─ Point（ol/geom/Point）
          │    ├─ LineString       │    ├─ LineString（ol/geom/LineString）
          │    ├─ Polygon          │    ├─ Polygon（ol/geom/Polygon）
          │    └─ ...              │    └─ Circle（ol/geom/Circle）
          ├─ properties            └─ 属性通过 Feature 的 set/get 方法管理
```

- GeoJSON 的 FeatureCollection
   是一个容器，包含多个 Feature 对象。OpenLayers 的 `VectorSource` 就相当于这个容器，管理一组 Feature。
- GeoJSON 的 Feature
   是一个地理实体，包含几何（geometry）和属性（properties）。OpenLayers 中的 `ol.Feature` 对象就是对应的实体，包括其几何形状和属性。
- GeoJSON 的 Geometry
   比如 Point、LineString、Polygon 等，在 OpenLayers 中对应的是 `ol/geom/Point`、`ol/geom/LineString`、`ol/geom/Polygon` 等几何类。
- 属性管理
   GeoJSON 中是 `properties` 字段，而在 OpenLayers 中，属性是通过 `feature.get('key')` 和 `feature.set('key', value)` 来管理的。



## 获取自定义geojson

```
https://l7editor.antv.antgroup.com/
```



## GeoJSON 各几何类型坐标层级对照

- Point 1
- Line / MultiPoint 2
- Polygon / MultiLine 3
- MultiPolygon 4

| 几何类型           | coordinates 层级示例                             | 说明（括号层数）                          |
| ------------------ | ------------------------------------------------ | ----------------------------------------- |
| Point              | `[lon, lat]`                                     | 1 层：单个位置                            |
| MultiPoint         | `[[lon, lat], [lon, lat]]`                       | 2 层：位置数组                            |
| LineString         | `[[lon, lat], [lon, lat], ...]`                  | 2 层：点序列（线）                        |
| MultiLineString    | `[[[lon, lat], ...], [[lon, lat], ...]]`         | 3 层：多条线                              |
| Polygon            | `[[ [lon, lat], [lon, lat], ..., [lon, lat] ]]`  | 3 层：线性环数组（至少 1 个环）           |
| Polygon 带孔洞     | `[[ outerRing... ], [ hole1... ], [ hole2... ]]` | 3 层：第一环外边界，后续环孔洞            |
| MultiPolygon       | `[[ [ring1...], [hole...] ], [ [ring2... ] ]]`   | 4 层：多个 polygon，每个 polygon 是环数组 |
| GeometryCollection | 特殊，内含多个 geometry 对象                     | 不靠 coordinates 层级表达                 |





# openlayers

```
const view = new View({
  projection: "EPSG:4326", 
  center: [114.25, 30.59],  
  zoom: 4, 
})

const map1 = new Map({
  target: "map",
  layers: [], // 图层添加要按照顺序，后面的会盖在前面的
  view: view,
})
你看啊，openlayers创建的地图实例new Map里面（Map也是个类，要从import Map from 'ol/Map.js';引入），target好像是个属性，layers是个装图层的属性，view是个存储视图信息的属性，

view属性的属性值要用View类的实例化对象，View类也要从import View from 'ol/View.js'引入
实例化的视图对象也有各种属性，像projection和center
const view = new View({
  projection: "EPSG:4326", 
  center: [114.25, 30.59],  
  zoom: 8, 
})

接着layers属性的属性值里面要装图层对象，图层对象还是从各种类里面实例化出来的，比如下面：
有瓦片图层类：要从import TileLayer from 'ol/layer/Tile.js'引入
const gaodevVectorLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}"
  })
})
有矢量图层类，要从import VectorSource from 'ol/source/Vector.js';引入
// 创建一个矢量图层
const ChinaVectorLayer = new VectorLayer({
  source: new VectorSource({
    url: "https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json",
    format: new GeoJSON(), // 指定数据格式
  }),
  style: new Style({
    fill: new Fill({
      color: 'rgba(255, 0, 0, 0.5)' //
    }),
    stroke: new Stroke() 
  })
})
map1.addLayer(ChinaVectorLayer)  这里好像是map实例里的一个方法addLayer，给layers属性添加图层的
还有静态图片类图层，要从import ImageLayer from 'ol/layer/Image.js'引入
// 静态图片图层
let center = [114.25, 30.59]
const imageExtent = [center[0]-0.5, center[1]-0.5, center[0]+0.5, center[1]+0.5]
const imageLayer = new ImageLayer({
  source: new Static({
    url: "../public/vite.svg",
    imageExtent: imageExtent  // 图片的地理范围  [minX, minY, maxX, maxY]
  })
})

然后实例化的图层对象还有各种属性，属性值也是从一个一个类实例过来的，比如每种图层里都有的source属性，属性值也是从各种类实例过来的，
比如瓦片图层的XYZ类，从import XYZ from 'ol/source/XYZ.js'引入，实例化的对象中有url属性，属性值是图层的资源定位符，矢量图层的矢量数据源类VectorSource，从import VectorSource from 'ol/source/Vector.js'引入，还要用source下的format属性指定数据格式，属性值还要从GeoJSON实例化而来，引入方式是import GeoJSON from 'ol/format/GeoJSON.js';，静态图片图层source属性值要用Static类的实例，引入方式是import Static from 'ol/source/ImageStatic.js';

最后实例化的图层对象中还有style属性，同样要从import Style from 'ol/style/Style.js';引入Style类，引入并实例化后还没完，Style的实例中各种属性还要引入各种类，比如Fill填充和Stroke描边类等，分别从import Fill from 'ol/style/Fill.js';和import Stroke from 'ol/style/Stroke.js'中引入，才能设置颜色color等具体样式

我感觉整个openlayers整个对象树很繁杂，你能帮我梳理一下吗，能类似文件结构图表示一下各种类的层级关系就更好了
```

## OpenLayers 对象结构图

```js
Map（地图实例）                  [ol/Map.js]
├─ view（视图/投影/中心/缩放）           [ol/View.js]
├─ layers（图层数组/图层组）              [ol/layer/XXX.js]
│  ├─ TileLayer（瓦片图层）               [ol/layer/Tile.js]
│  │   └─ source（数据源）                [ol/source/XYZ.js 等]
│  │       ├─ XYZ（瓦片XYZ源）            [ol/source/XYZ.js]
│  │       ├─ OSM（OpenStreetMap）        [ol/source/OSM.js]
│  │       └─ WMTS（瓦片地图服务）        [ol/source/WMTS.js]
│  ├─ ImageLayer（图片/影像图层）         [ol/layer/Image.js]
│  │   └─ source
│  │       ├─ ImageStatic（静态图片）     [ol/source/ImageStatic.js]
│  │       └─ ImageWMS（WMS服务）        [ol/source/ImageWMS.js]
│  ├─ VectorLayer（矢量图层）             [ol/layer/Vector.js]
│  │   ├─ style（图层级样式，可选）        [ol/style/Style.js]
│  │   └─ source: VectorSource            [ol/source/Vector.js]
│  │       ├─ features: Feature[]         [ol/Feature.js]
│  │       │   ├─ Feature                 [ol/Feature.js]
│  │       │   │   ├─ geometry            [ol/geom/XXX.js]
│  │       │   │   │   ├─ Point           [ol/geom/Point.js]
│  │       │   │   │   ├─ LineString      [ol/geom/LineString.js]
│  │       │   │   │   ├─ Polygon         [ol/geom/Polygon.js]
│  │       │   │   │   └─ Circle          [ol/geom/Circle.js]
│  │       │   │   └─ style（要素级样式，可选） [ol/style/Style.js]
│  │       │   └─ ...（更多Feature）
│  │       └─ format（格式化器）      [ol/format/GeoJSON.js 等]
│  │           ├─ GeoJSON              [ol/format/GeoJSON.js]
│  │           ├─ TopoJSON             [ol/format/TopoJSON.js]
│  │           └─ ...
│  ├─ HeatmapLayer（热力图图层）          [ol/layer/Heatmap.js]
│  │   └─ source: VectorSource            [ol/source/Vector.js]
│  │       └─ features: Feature[]          [ol/Feature.js]
│  │           └─ 权重weight属性可设置
│  ├─ ClusterLayer（聚合图层，基于VectorLayer） [ol/layer/Vector.js + ol/source/Cluster.js]
│  │   ├─ source: Cluster                  [ol/source/Cluster.js]
│  │   │   └─ source: VectorSource        [ol/source/Vector.js]
│  │   │       └─ features: Feature[]
│  │   └─ style（聚合样式，动态显示点数或图标）
│  └─ AnimatedClusterLayer（动态聚合图层）[第三方扩展 ol-ext/layer/AnimatedCluster.js]
│      ├─ source: Cluster
│      └─ animation配置与聚合样式
│
├─ controls（地图控件，如缩放/比例尺）      [ol/control/XXX.js]
├─ interactions（用户交互）                [ol/interaction/XXX.js]
│   ├─ Draw（绘制要素）                   [ol/interaction/Draw.js]
│   ├─ Select（要素选中）                 [ol/interaction/Select.js]
│   └─ ...
│
├─ overlays（覆盖物）                     [ol/Overlay.js]
│   └─ Overlay（HTML自定义弹窗/标注）
│
├─ feature animation（要素动画）          [ol-ext/featureanimation/XXX.js]
│   ├─ Path（路径动画）                    [ol-ext/featureanimation/Path.js]
│   └─ ...
│
└─ events
    ├─ map.on('click', handler)（地图点击事件）
    ├─ vectorLayer.on('animationend', handler)（动画结束事件）
    └─ select.on('select', handler)（选中事件）
```

矢量样式结构

```
VectorLayer
└─ style
    └─ Style         [ol/style/Style.js]
        ├─ Fill      [ol/style/Fill.js]
        ├─ Stroke    [ol/style/Stroke.js]
        ├─ Icon      [ol/style/Icon.js]
        └─ Text      [ol/style/Text.js]
```



理解：

1. **Map 是一切的根对象**
   - `new Map({...})` 之后，你就有了一个“地图实例”。
   - 它主要由几块组成：`view`、`layers`、`controls`、`interactions`、`overlays`、`events`。
2. **view：控制“相机”**
   - `View` 管的是“怎么看地图”：坐标系（projection）、中心（center）、缩放级别（zoom）、旋转等。
3. **layers：控制“画什么”**
   常见几类图层：
   - **TileLayer（瓦片图层）**：
     - `source` 是各种瓦片服务：`XYZ`、`OSM`、`WMTS` 等；
     - 它只负责一张张图片拼到底图上。
   - **ImageLayer（图片图层）**：
     - `ImageStatic`：一整张静态图片盖在某个地理范围上；
     - `ImageWMS`：WMS 服务返回的地图图片。
   - **VectorLayer（矢量图层）**：
     - `source` 是 `VectorSource`；
     - `VectorSource` 中存的是 `Feature[]`；
     - 每个 `Feature` 有一个 `geometry`（Point/LineString/Polygon/Circle 等 geom 对象），可以有自己的 `style`；
     - 图层自己也可以有一个统一的 `style`（图层级样式）；
     - 如果是从文件/接口加载数据，则通过 `format`（如 `GeoJSON`）负责解析。
   - **HeatmapLayer**：
     - 本质上是基于 `VectorSource` 的一个子类图层；
     - 从 `Feature[]` 里读点（和可选权重），画成热力图。
   - **ClusterLayer / AnimatedClusterLayer**：
     - 内部 `source` 是 `Cluster`（对一个 `VectorSource` 做聚合）；
     - `Cluster` 自己再包了一层原始 `VectorSource`；
       聚合后的 Feature 里有 `features: Feature[]` 属性；
     - `AnimatedCluster` 是 ol-ext 扩展，用动画渲染聚合/拆分过程。
4. **controls：地图控件**
   - 比如缩放按钮、比例尺、图层切换控件等，都挂在 `controls` 下。
5. **interactions：用户交互**
   - 比如：
     - `Draw`：在地图上画点/线/面；
     - `Select`：选中 Feature；
     - `Modify`、`Translate` 等；
   - 这些交互注册到 `map` 上之后，就可以监听相应事件（如 `drawend`、`select`）。
6. **overlays：HTML 覆盖物**
   - 每个 `Overlay` 把一个 DOM 元素绑定到一个地理坐标上；
   - 用来做自定义信息框、提示标签、弹出卡片等。
7. **feature animation（ol-ext）**
   - 不是 OpenLayers 核心的一部分，而是 ol-ext 扩展，比如：
     - `Path`：让一个 Feature 沿着给定几何路径运动；
     - 其它 Blink、Zoom 等动画。
8. **events：各种对象上的事件系统**
   - Map：`map.on('click', handler)`；
   - Layer：比如 `animationend`；
   - Select：`select.on('select', handler)`；
   - 基本所有主要对象都支持 `.on(...)` 统一事件风格。



伪代码版本

```ts
// 最顶层：地图实例
const map = new Map({
  view: new View({
    projection: 'EPSG:4326',
    center: [114.25, 30.59],
    zoom: 8,
  }),

  layers: [
    // 1) 瓦片图层
    new TileLayer({
      source: new XYZ({ /* url, tileGrid... */ }),
      // 或 source: new OSM()
      // 或 source: new WMTS({ ... })
    }),

    // 2) 图片图层（静态/ WMS）
    new ImageLayer({
      source: new ImageStatic({ /* url, imageExtent... */ }),
      // 或 source: new ImageWMS({ ... })
    }),

    // 3) 矢量图层
    new VectorLayer({
      style: new Style({ /* 填充、描边等图层级样式 */ }),
      source: new VectorSource({
        features: [
          new Feature({
            geometry: new Point([x, y]),
            // 或 new LineString([...])
            // 或 new Polygon([...])
            // 或 new Circle([x, y], r)
            // 要素自己的属性：
            name: '武汉',
          }),
          // 更多 Feature...
        ],
        format: new GeoJSON(), // 如果是从 url/文本加载
      }),
    }),

    // 4) 热力图图层（基于 VectorSource）
    new HeatmapLayer({
      source: new VectorSource({ features: pointFeatures }),
    }),

    // 5) 聚合图层
    new VectorLayer({
      source: new Cluster({
        source: new VectorSource({ features: pointFeatures }),
      }),
      style: clusterStyleFn,
    }),

    // 6) 动态聚合图层（ol-ext）
    new AnimatedClusterLayer({
      source: new Cluster({ source: rawVectorSource }),
      animationDuration: 700,
      style: animatedClusterStyleFn,
    }),
  ],

  controls: [
    // 各种控件，如 Zoom, ScaleLine 等
  ],

  interactions: [
    // 默认交互 + 自己加的
    new Draw({ type: 'LineString', source: drawSource }),
    new Select({ /* condition: pointerMove 等 */ }),
  ],

  overlays: [
    new Overlay({
      element: document.querySelector('.overlay'),
    }),
  ],
});

// 要素动画（来自 ol-ext）
lineLayer.animateFeature(
  movingFeature,
  new Path({ path: lineFeature.getGeometry(), duration: 4000 })
);

// 事件示例
map.on('click', (evt) => { /* 处理地图点击 */ });
vectorLayer.on('animationend', (evt) => { /* 动画结束 */ });
select.on('select', (evt) => { /* 选中 / 取消选中 */ });
```







## 外部地图服务图层

```js
import OSM from 'ol/source/OSM.js'  // OSM瓦片数据源类
import XYZ from 'ol/source/XYZ.js'  // 瓦片数据源类


// OSM瓦片图层
const osmLayer = new TileLayer({
  source: new OSM()
})

// 高德瓦片图层
const gaodeLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}"  // 图层的资源定位符，定位具体的服务提供者
  })
})

// 高德影像图层
const gaodeImageLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=6&x={x}&y={y}&z={z}"
  })
})

// 高德文字标注图层
const gaodeTextLayer = new TileLayer({
  source: new XYZ({
    url: "http://webst02.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scale=1&style=8"
  })
})

// 高德矢量图层
const gaodevVectorLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}"
  })
})

// 天地图影像图层  	天地图key 4a050b56e21976c2bb53fec757797dc1
const tiandituImgLayer = new TileLayer({
  source: new XYZ ({
    url: "http://t0.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=4a050b56e21976c2bb53fec757797dc1"
  })
})

// 天地图矢量图层
const tiandituVectorLayer = new TileLayer({
  source: new XYZ ({
    url: "http://t0.tianditu.gov.cn/vec_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=vec&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=4a050b56e21976c2bb53fec757797dc1"
  })
})

// 天地图矢量注记图层
const tiandituTextLayer = new TileLayer({
  source: new XYZ ({
    url: "http://t0.tianditu.gov.cn/cva_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=cva&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=4a050b56e21976c2bb53fec757797dc1"
  })
})
```



1. 基础骨架：Map + View + TileLayer
2. 矢量要素：VectorLayer + VectorSource + Feature + Geometry + Style
3. 交互：Draw / Select / Overlay（测距、信息框）
4. 进阶可视化：Heatmap / Cluster / AnimatedCluster / Path 动画
5. 常用工具函数：getArea / getLength / 事件 on/unByKey



## 基础骨架：Map + View + TileLayer

### 概念

- **Map**：整个地图应用的容器，负责把图层渲染到指定 DOM 上。
- **View**：地图的“相机”或“视角”，负责中心点、缩放级别、投影等。
- **Layer（图层）**：真正显示内容的“画布”，最常见的是瓦片图层 `TileLayer`。

```Text
HTML <div id="map">   ←  Map.target 挂这里
        ↑
      Map
        ├─ view: View               ← 相机：center / zoom / projection
        └─ layers: [ TileLayer... ] ← 底图、影像、矢量等图层
```

------

### 最小可运行示例（OSM 底图）

这个例子是你每次新建 OL 页面时的“起点模板”。

```ts
// main.ts / main.js
import 'ol/ol.css';
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';

const map = new Map({
  target: 'map',                // 绑定到哪一个 DOM 容器上（id="map"）
  layers: [
    new TileLayer({
      source: new OSM(),        // OSM 瓦片底图
    }),
  ],
  view: new View({
    projection: 'EPSG:3857',    // 默认投影是 Web 墨卡托（EPSG:3857）
    center: [0, 0],             // 注意：3857 下是“投影坐标”，不是经纬度
    zoom: 2,
  }),
});
```

HTML：

```html
<style>
  html, body {
    margin: 0;
    padding: 0;
  }
  #map {
    width: 100vw;
    height: 100vh;
  }
</style>
<div id="map"></div>
<script type="module" src="/src/main.ts"></script>
```

> 面试前/项目开头，你只要记得这段，就能迅速把一张世界地图跑起来。

------

### 使用经纬度坐标（EPSG:4326）

如果地点是直接用 `[lon, lat]`，这在 OL 中需要把 View 的投影设为 `'EPSG:4326'`。

```ts
const map = new Map({
  target: 'map',
  layers: [
    new TileLayer({
      source: new OSM(),  // 不少在线底图也能自动适配 4326
    }),
  ],
  view: new View({
    projection: 'EPSG:4326',     // 使用经纬度坐标系
    center: [114.25, 30.59],     // 武汉
    zoom: 8,
  }),
});
```

- 如果**直接在代码里写 `[经度, 纬度]`**，View 的 `projection` 最好写 `'EPSG:4326'`；
- 如果用默认的 3857，则需要 `fromLonLat([lon, lat])` 做一次转换。

------

### 多种在线瓦片源示例（OSM / 高德 / 天地图）

OSM（开源地图）

```ts
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';

const osmLayer = new TileLayer({
  source: new OSM(),
});
```



高德（XYZ）

```ts
import TileLayer from 'ol/layer/Tile';
import XYZ from 'ol/source/XYZ';

const gaodeLayer = new TileLayer({
  source: new XYZ({
    url:
      'http://wprd0{1-4}.is.autonavi.com/appmaptile?' +
      'lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}',
  }),
});
```



天地图（WMTS as XYZ）

```ts
import TileLayer from 'ol/layer/Tile';
import XYZ from 'ol/source/XYZ';

const tiandituImgLayer = new TileLayer({
  source: new XYZ({
    url:
      'http://t0.tianditu.gov.cn/img_w/wmts?' +
      'SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&' +
      'TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&' +
      'tk=你的key',
  }),
});
```



把上面的图层变量放在 `new Map({ layers: [...] })` 里，按顺序把它们放进去即可：

```ts
const map = new Map({
  target: 'map',
  layers: [gaodeLayer, tiandituImgLayer], // 后面的会盖在前面的上面
  view: new View({ projection: 'EPSG:4326', center: [...], zoom: 8 }),
});
```

------

### View 的常用操作（移动 / 缩放 / 动画）

获取 / 设置 View

```ts
const view = map.getView();

const center = view.getCenter();           // [x, y]
const zoom = view.getZoom();               // 当前缩放等级

view.setCenter([114.25, 30.59]);           // 立即移动中心
view.setZoom(10);                          // 立即缩放
```



视图动画（平移 / 缩放）

“去广州”例子可以简化成这样模板：

```ts
const view = map.getView();

function goToGuangzhou() {
  view.animate(
    { center: [113.27, 23.13], duration: 1000 }, // 第一段动画：平移
    { zoom: 12, duration: 1000 }                 // 第二段动画：缩放
  );
}
```

点击按钮触发：

```ts
document.querySelector('.goTo')?.addEventListener('click', goToGuangzhou);
```

------

### Map / View / Layer 常用 getter / setter 速查

这一块你原来总结得很好，可以压成一张表，方便你忘了时快速翻。

Map

- `map.getTarget()` / `map.setTarget(domOrId)`
- `map.getView()` / `map.setView(view)`
- `map.getLayers()` → 返回 LayerGroup
  - `.getArray()` → 真正的图层数组
- `map.addLayer(layer)`
- `map.removeLayer(layer)`

View

- `view.getCenter()` / `view.setCenter([x, y])`
- `view.getZoom()` / `view.setZoom(z)`
- `view.animate(...keyframes)`

Layer（以 TileLayer/VectorLayer 为例）

- `layer.getSource()` / `layer.setSource(source)`
- `layer.setVisible(true/false)`
- `layer.setZIndex(z)` 调整图层上下顺序

------

### 小结

> - Map 管“挂在哪个 DOM 上 + 渲染哪些图层”
> - View 管“相机”：中心点、缩放、投影、动画
> - TileLayer + Source(OSM/XYZ) 管“底图长什么样”
>
> 只要你能从空文件重新写出：
>
> ```ts
> new Map({ target, layers: [new TileLayer({ source })], view: new View({...}) })
> ```
>
> 这套骨架就算完全复习回来了。

------





## 矢量要素：VectorLayer + VectorSource + Feature + Geometry + Style

### 概念

- **VectorLayer**：矢量“图层”（负责渲染）。
- **VectorSource**：矢量“数据源”（存放 Feature 的仓库）。
- **Feature**：一个具体的“地理要素”（点/线/面 + 属性）。
- **Geometry**：Feature 的几何形状（Point / LineString / Polygon …）。
- **Style**：点/线/面的显示样式（颜色、宽度、图标、文字等）。



```Text
VectorLayer  ← 设置样式 style
  └─ source: VectorSource
        └─ features: Feature[]
              ├─ geometry: Geometry(Point / LineString / Polygon...)
              └─ properties: 业务属性（name, id, ...）
```

------

### 用 GeoJSON 加载一整个矢量图层（最常用）

这是你在项目里**最常见的一种方式**：直接给个 GeoJSON URL，让 OpenLayers 帮你解析为 Feature。



```ts
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import GeoJSON from 'ol/format/GeoJSON';
import Style from 'ol/style/Style';
import Fill from 'ol/style/Fill';
import Stroke from 'ol/style/Stroke';

// 1. 用 VectorSource 从远程 GeoJSON 加载数据
const chinaSource = new VectorSource({
  url: 'https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json',
  format: new GeoJSON(),      // 告诉它：这是 GeoJSON 格式
});

// 2. 创建 VectorLayer，绑定数据源和样式
const chinaLayer = new VectorLayer({
  source: chinaSource,
  style: new Style({
    fill: new Fill({ color: 'rgba(255, 0, 0, 0.5)' }), // 半透明红色
    stroke: new Stroke({ color: '#333', width: 1 }),   // 灰色描边
  }),
});

// 3. 添加到 map
map.addLayer(chinaLayer);
```

> - 远程的 GeoJSON FeatureCollection → VectorSource.features
> - 每个 Feature → 一块省的 polygon
> - VectorLayer = 把这些 polygon 渲染出来的“图层”

------

### 手工创建 Feature（点 / 线 / 面）

你平时手写数据或响应交互时，会用这一套：`Geometry → Feature → VectorSource → VectorLayer`。

点要素（手动添加一个点）

```ts
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';

// 1. 创建几何对象（点）
const pointGeom = new Point([114.3162, 30.5278]);   // 武汉某个位置

// 2. 创建 Feature，挂上 geometry 和属性
const pointFeature = new Feature({
  geometry: pointGeom,
  name: '黄鹤楼',
});

// 3. 创建矢量数据源，放入 Feature
const pointSource = new VectorSource({
  features: [pointFeature],
});

// 4. 创建矢量图层，挂载数据源
const pointLayer = new VectorLayer({
  source: pointSource,
});

// 5. 加到 map
map.addLayer(pointLayer);
```



线要素（武汉 → 北京）

```ts
import LineString from 'ol/geom/LineString';
import Feature from 'ol/Feature';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';

// 1. 创建 LineString 几何（折线）
// 注意：数组里是点坐标序列
const lineGeom = new LineString([
  [114.31, 30.52],  // 武汉
  [116.40, 39.90],  // 北京
]);

// 2. 包装成 Feature
const lineFeature = new Feature({
  geometry: lineGeom,
});

// 3. 数据源 + 图层
const lineLayer = new VectorLayer({
  source: new VectorSource({
    features: [lineFeature],
  }),
});

map.addLayer(lineLayer);
```



面要素（三角形 Polygon）

```ts
import Polygon from 'ol/geom/Polygon';
import Feature from 'ol/Feature';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';

// 1. Polygon 需要“环数组”：[[[x1,y1],[x2,y2],...,[x1,y1]]]
//   最外层是“多个环”，第一个环是外轮廓，后续为洞（可选）
const triangleGeom = new Polygon([[
  [114.25, 35.59],
  [124.26, 25.60],
  [104.27, 25.59],
  [114.25, 35.59],    // 闭合，首尾坐标相同
]]);

// 2. 包装为 Feature
const triangleFeature = new Feature({
  geometry: triangleGeom,
});

// 3. 放入 VectorSource & VectorLayer
const triangleLayer = new VectorLayer({
  source: new VectorSource({
    features: [triangleFeature],
  }),
});

map.addLayer(triangleLayer);
```

------

### Style：图层样式 & 要素样式

图层统一样式（所有要素共享）

```ts
import Style from 'ol/style/Style';
import Fill from 'ol/style/Fill';
import Stroke from 'ol/style/Stroke';

const vectorLayer = new VectorLayer({
  source: new VectorSource({ /* ... */ }),
  style: new Style({
    fill: new Fill({ color: 'rgba(255, 0, 0, 0.5)' }),
    stroke: new Stroke({ color: '#000', width: 1 }),
  }),
});
```



Feature 单独样式（每个要素自己一套）

**关键点：**

- **不能**在 `new Feature({ ..., style: xxx })` 里写 `style` 字段；
- 必须用 `feature.setStyle(style)`。

```ts
import Style from 'ol/style/Style';
import Icon from 'ol/style/Icon';
import Text from 'ol/style/Text';
import Fill from 'ol/style/Fill';

// 1. 创建点 Feature
const point = new Feature({
  geometry: new Point([114.3162, 30.5278]),
});

// 2. 定义样式：图标 + 文本
const pointStyle = new Style({
  image: new Icon({
    src: '/401位置.png',
    scale: 0.2,
    anchor: [0.5, 1],  // 图标锚点：底部中央
  }),
  text: new Text({
    text: '黄鹤楼',
    offsetY: -20,
    font: '12px Calibri,sans-serif',
    fill: new Fill({ color: 'orange' }),
  }),
});

// 3. 赋给要素
point.setStyle(pointStyle);
```

然后像前面一样，把 `point` 放进 `VectorSource` → `VectorLayer` 即可。

> - 图层统一样式 → 在 VectorLayer 上用 `style`
> - 要素单独样式 → `feature.setStyle()`，不要写到构造对象里



按要素类型/属性动态返回样式（style 函数）

当一个图层里既有点又有线（如你“点+线共图层”的例子），可以用 style 函数按几何类型区分。

```ts
const pointStyle = new Style({ /* ... */ });
const lineStyle = new Style({ /* ... */ });

const mixedLayer = new VectorLayer({
  source: new VectorSource({
    features: [pointFeature, lineFeature],
  }),
  style: function (feature) {
    const geomType = feature.getGeometry().getType(); // 'Point' / 'LineString' / 'Polygon' ...
    if (geomType === 'Point') {
      return pointStyle;
    } else if (geomType === 'LineString') {
      return lineStyle;
    }
    return null;
  },
});
```

> 聚合图层（Cluster/AnimatedCluster）的 style 函数参数是“聚合要素”，
> 里面通过 `feature.get('features')` 拿到原始 Feature 数组，这个我们在后面聚合那一章再专讲。

------

### 监听矢量数据加载完成（异步 Source）

你之前已经用过这一段流程：Source 是异步加载的，要等 `state === 'ready'` 再拿 features。

```ts
const source = chinaLayer.getSource();

source.on('change', () => {
  if (source.getState() === 'ready') {
    const features = source.getFeatures();
    console.log('矢量数据加载完成，数量：', features.length);
    // 可以在这里做缩放、统计等后续操作
  }
});
```

> 心里记一下：**GeoJSON URL → VectorSource 加载是异步的**，
> 真正用数据（比如统计 feature 数量、计算范围）最好放到这个回调里做。

------

### 小结

**真正需要掌握**的是下面这些：

1. **从 GeoJSON URL 加载矢量图层**
   - `VectorSource({ url, format: new GeoJSON() })`
   - `new VectorLayer({ source, style })` → `map.addLayer(layer)`
2. **手动创建点/线/面 Feature**
   - 点：`new Point([x, y])` → `Feature({ geometry })`
   - 线：`new LineString([[x1,y1],[x2,y2],...])`
   - 面：`new Polygon([[ [x1,y1],[x2,y2],...,[x1,y1] ]])`（注意闭合）
3. **样式系统**
   - 图层统一样式：VectorLayer 的 `style` 属性
   - 单独要素样式：`feature.setStyle(style)`
   - style 函数：`(feature) => style`，可按 `getGeometry().getType()` 或属性字段决定样式
4. **知道 Feature / Geometry / VectorSource / VectorLayer 的关系图**

```Text
VectorLayer
  └─ source: VectorSource
        └─ features: Feature[]
              ├─ geometry: Geometry(Point/LineString/Polygon)
              └─ properties: 任意字段（properties）
```



## 交互：Draw / Select / Overlay（测距、信息框）

### 概念

- **Draw 交互**：让用户在地图上“画”几何（点/线/面），画完产出一个 `Feature`，放进指定的 `VectorSource`。
- **Select 交互**：让用户通过点击 / 悬停等方式“选中”已有的 Feature。
- **Overlay**：在地图上的某个坐标位置“挂”一个 HTML 元素（信息框 / 浮动标签）。

常见组合：

- `Draw(LineString) + getLength` → 交互画线测距。
- `Select + Overlay` → 悬停/点击显示要素属性信息。
- `Select(Polygon) + getArea` → 选中多边形计算面积。

------

### Draw：在地图上画几何（点/线/面）

基本流程

1. 创建一个空的 `VectorLayer` + `VectorSource` 当“画布”。
2. 用 `new Draw({ type, source })` 创建绘图交互。
3. `map.addInteraction(draw)` 注册到地图。
4. 监听 `drawend` 事件，拿到用户画出的 Feature 继续处理（测量、保存等）。



画线

```ts
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Draw from 'ol/interaction/Draw';
import Style from 'ol/style/Style';
import Stroke from 'ol/style/Stroke';

// 1. 创建一个空矢量图层，用来存储用户画出来的要素
const drawSource = new VectorSource();

const drawLayer = new VectorLayer({
  source: drawSource,
  style: new Style({
    stroke: new Stroke({
      color: 'blue',
      width: 2,
    }),
  }),
});

map.addLayer(drawLayer);

// 2. 创建 Draw 交互（画线）
const draw = new Draw({
  type: 'LineString',     // 'Point' | 'LineString' | 'Polygon' | 'Circle' ...
  source: drawSource,     // 画完的 Feature 会自动加到这个 source 里
});

// 3. 添加到地图
map.addInteraction(draw);

// 4. 监听绘制开始/结束事件
draw.on('drawstart', (evt) => {
  console.log('绘制开始：', evt.feature);
});

draw.on('drawend', (evt) => {
  console.log('绘制结束，得到的 Feature：', evt.feature);
  // evt.feature.getGeometry() 就是用户画的线
});
```

> - Draw 不直接画在 map 上，而是往 `VectorSource` 里丢一个 Feature；
> - 画什么类型，由 `type: 'Point' | 'LineString' | 'Polygon'` 决定。

------

### Select：选中要素（点击 / 悬停）

基本流程

1. `new Select(...)` 创建选择交互。
2. `map.addInteraction(select)` 注册到地图。
3. 监听 `select` 事件，看 `evt.selected` / `evt.deselected`。
4. 常见触发方式：
   - 默认是鼠标点击；
   - 也可以改成 `pointerMove`（悬停高亮）。



最小示例：点击选中要素并打印属性

```ts
import Select from 'ol/interaction/Select';

// 1. 创建 Select 交互（默认：点击选中）
const select = new Select();

// 2. 注册到地图
map.addInteraction(select);

// 3. 监听 select 事件
select.on('select', (evt) => {
  console.log('选中要素：', evt.selected);
  console.log('取消选中：', evt.deselected);

  evt.selected.forEach((feature) => {
    console.log('选中要素属性：', feature.getProperties());
  });
});
```



改成“鼠标悬停选中”（pointerMove）

```ts
import Select from 'ol/interaction/Select';
import { pointerMove } from 'ol/events/condition';

const hoverSelect = new Select({
  condition: pointerMove,   // 由鼠标移动触发
});

map.addInteraction(hoverSelect);
```

> 常见用途：在行政区图层上悬停时高亮一个省，配合 Overlay 显示简要信息。

------

### Overlay：在地图上挂一个 HTML 信息框

基本模型

- Overlay ≈ “地图坐标 → DOM 元素定位” 的桥梁。
- 你在 HTML 里写一个 `<div class="overlay"></div>`，再创建一个 Overlay，把该 DOM 元素和一个坐标绑定起来。



最小示例：在某点显示一个标签

HTML：

```html
<div id="map"></div>
<div class="overlay"></div>

<style>
  .overlay {
    position: absolute;
    background-color: rgba(255, 255, 255, 0.8);
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 12px;
  }
</style>
```

JS：

```ts
import Overlay from 'ol/Overlay';

// 1. 创建 Overlay：把 DOM 元素挂到地图上
const infoOverlay = new Overlay({
  element: document.querySelector('.overlay') as HTMLElement,
});

map.addOverlay(infoOverlay);

// 2. 设置位置 & 内容
infoOverlay.setPosition([114.25, 30.59]);  // 武汉某坐标
(document.querySelector('.overlay') as HTMLElement).innerHTML = '武汉';
```

之后你可以随时调用：

```ts
infoOverlay.setPosition(undefined); // 隐藏覆盖物
```

------

### Select + Overlay：行政区悬停高亮 + 显示属性信息

这是你之前的一个完整例子，简化成模板后可以经常复用。



加载省级 GeoJSON 图层（略）

前一章已经有：

```ts
const chinaLayer = new VectorLayer({
  source: new VectorSource({
    url: 'https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json',
    format: new GeoJSON(),
  }),
  // 可以加一个基础样式
});
map.addLayer(chinaLayer);
```



悬停选择 + 自定义高亮样式

```ts
import Select from 'ol/interaction/Select';
import { pointerMove } from 'ol/events/condition';
import Style from 'ol/style/Style';
import Fill from 'ol/style/Fill';
import Stroke from 'ol/style/Stroke';

// 高亮样式
const highlightStyle = new Style({
  fill: new Fill({ color: 'rgba(33, 150, 243, 0.5)' }),
  stroke: new Stroke({ color: 'rgba(13, 71, 161, 0.8)', width: 2 }),
});

// 悬停选择
const hoverSelect = new Select({
  condition: pointerMove,
});

hoverSelect.setStyle(highlightStyle as any); // 或 hoverSelect.style_ = highlightStyle（内部属性）
// 新版可用 options 里的 style，具体看文档版本

map.addInteraction(hoverSelect);
```



绑定 Overlay 显示属性

HTML 与 之前 类似，有一个 `.overlay` 元素。

```ts
import Overlay from 'ol/Overlay';

const overlay = new Overlay({
  element: document.querySelector('.overlay') as HTMLElement,
});
map.addOverlay(overlay);

hoverSelect.on('select', (e) => {
  const el = overlay.getElement() as HTMLElement;

  if (e.selected.length > 0) {
    const feature = e.selected[0];
    const props = feature.getProperties();
    const center = props.center || feature.getGeometry().getExtent(); // 有些数据自带 center

    // 设置 Overlay 位置和内容
    overlay.setPosition(center);   // center 是 [x,y] 或你自己算 bbox 中心
    el.innerHTML = `
      <p>adcode: ${props.adcode}</p>
      <p>name: ${props.name}</p>
      <p>level: ${props.level}</p>
    `;
  } else {
    overlay.setPosition(undefined); // 鼠标移走，隐藏
  }
});
```

> “用 Select 找到当前鼠标下的 Feature，用 Overlay 把一个 div 放到该 Feature 的坐标上，把属性渲染进去。”

------

### Draw + getLength：画线测距

这一部分经常被问，也很有实际价值。你之前已经写过一大段，我帮你浓缩成一个“最小测距模板”。

核心逻辑

1. 创建一个专门的 `VectorLayer` 用于存放测距线。
2. 用 `Draw({ type: 'LineString' })` 让用户画线。
3. 在`drawend`时：
   - 拿到 `evt.feature.getGeometry()`；
   - 用 `getLength(geometry, { projection })` 算长度（米）；
   - 显示到 Overlay 上。



示例代码

```ts
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Draw from 'ol/interaction/Draw';
import Overlay from 'ol/Overlay';
import Style from 'ol/style/Style';
import Stroke from 'ol/style/Stroke';
import { getLength } from 'ol/sphere';

// 1. 测距专用图层
const measureSource = new VectorSource();
const measureLayer = new VectorLayer({
  source: measureSource,
  style: new Style({
    stroke: new Stroke({ color: 'red', width: 2 }),
  }),
});
map.addLayer(measureLayer);

// 2. Draw 交互（画线）
const measureDraw = new Draw({
  type: 'LineString',
  source: measureSource,
});
map.addInteraction(measureDraw);

// 3. Overlay 显示测距文字
const measureOverlay = new Overlay({
  element: document.querySelector('.overlay-measure') as HTMLElement,
});
map.addOverlay(measureOverlay);

// 4. 监听 drawend
measureDraw.on('drawend', (e) => {
  const geom = e.feature.getGeometry(); // LineString
  let length = getLength(geom, { projection: 'EPSG:4326' }); // 单位：米
  length = length / 1000; // 转为千米

  // 取线的最后一个点作为显示位置
  const coords = geom.getCoordinates();
  const lastPoint = coords[coords.length - 1];

  measureOverlay.setPosition(lastPoint);
  (measureOverlay.getElement() as HTMLElement).innerHTML =
    `长度：${length.toFixed(2)} km`;
});
```



HTML 示例：

```html
<div id="map"></div>
<div class="overlay-measure"></div>

<style>
  .overlay-measure {
    position: absolute;
    padding: 4px 8px;
    background-color: rgba(0, 0, 0, 0.5);
    color: #fff;
    border-radius: 4px;
    font-size: 12px;
  }
</style>
```

------

### 交互切换：在 Draw 和 Select 之间切换

你已经写过一个“按钮切换交互”的例子，这个思想非常好：

- 测距模式（Draw）；
- 悬停查看属性模式（Select）。



个简化模板：

```ts
const btnSelect = document.querySelector('.btn-select')!;
const btnDraw = document.querySelector('.btn-draw')!;

btnSelect.addEventListener('click', () => {
  map.removeInteraction(measureDraw);
  map.addInteraction(hoverSelect);
  // 清理测距图层 & 测距 overlay
});

btnDraw.addEventListener('click', () => {
  map.removeInteraction(hoverSelect);
  map.addInteraction(measureDraw);
  // 显示测距 overlay / 隐藏属性 overlay
});
```

> OpenLayers 里“模式切换”本质就是在不同 Interaction 之间 `addInteraction` / `removeInteraction`。

------

### 小结

1. 用 Draw 在地图上画点/线/面，并在 `drawend` 里拿到 Feature 做后续处理。
2. 用 Select（点击或悬停）选中 Feature， `evt.selected` / `evt.deselected` 的含义。
3. 用 Overlay 把一个 HTML div 绑定到地图坐标上，并配合 Select 显示要素属性（信息框）。
4. 最小的“画线测距”功能：
   - Draw(LineString) + getLength + Overlay。
5. 交互模式之间切换（添加/移除 Interaction）。





## 进阶可视化：Heatmap / Cluster / AnimatedCluster / Path 动画

###  统一前提：一堆点 Feature

这几种效果（热力图、聚合、路径动画）本质上都基于**一堆点坐标**。

```ts
// 原始点数据（经纬度）
const points = [
  { name: '黄鹤楼', coords: [114.3162, 30.5278] },
  { name: '武汉博物馆', coords: [114.288, 30.547] },
  { name: '武汉大学', coords: [114.366, 30.538] },
  // ...
];

// 转为 Feature[]
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';

const pointFeatures = points.map((item) => {
  return new Feature({
    geometry: new Point(item.coords), // 你这套一般都用 EPSG:4326
    name: item.name,
  });
});
```

后面的 Heatmap / Cluster / 动画，复用这批 `pointFeatures`。

------

### Heatmap：热力图

基本流程

- **HeatmapLayer** = 特殊的 VectorLayer，不画具体点，而是画“密度/权重分布”。
- 输入：一堆点 Feature（可带权重属性）。
- 输出：颜色渐变的区域，点越密/权重越大 → 越亮/越红。



使用模板

```ts
import Heatmap from 'ol/layer/Heatmap';
import VectorSource from 'ol/source/Vector';

// 1. 用 pointFeatures 构建 VectorSource
const pointSource = new VectorSource({
  features: pointFeatures,
});

// 2. 创建热力图层
const heatmapLayer = new Heatmap({
  source: pointSource,
  // 可选参数：
  // blur: 15,
  // radius: 8,
  // weight: (feature) => ...  // 指定权重字段或函数
});

// 3. 添加到 map
map.addLayer(heatmapLayer);
```

> - HeatmapLayer 仍然是 VectorLayer 的子类，只是渲染方式变成热力图；
> - 如果不设置 `weight`，所有点权重一样；
>   也可以给 Feature 设置一个字段（如 `feature.set('weight', 0.5)`），然后 `weight: f => f.get('weight')`。

------



### Cluster：聚合点

基本流程

- 聚合 = 在屏幕像素距离内，把多个邻近点“合并”为一个聚合点，显示为“一个圆圈+数量”。
- 适合在“点特别多”的情况下避免地图铺满图标。

结构：

```Text
原始点 Feature[] → VectorSource
             ↓
         ClusterSource（按 distance 聚合）
             ↓
        VectorLayer(style 函数里 feature.get('features') 拿到子点)
```



普通聚合

```ts
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import VectorSource from 'ol/source/Vector';
import Cluster from 'ol/source/Cluster';
import VectorLayer from 'ol/layer/Vector';
import Style from 'ol/style/Style';
import Icon from 'ol/style/Icon';
import CircleStyle from 'ol/style/Circle';
import Fill from 'ol/style/Fill';
import Text from 'ol/style/Text';

// 1. 原始点 Feature[]（前面 pointFeatures）
const rawSource = new VectorSource({
  features: pointFeatures,
});

// 2. 聚合数据源
const clusterSource = new Cluster({
  distance: 40,   // 像素距离内的点会聚成一团，可调
  source: rawSource,
});

// 3. 样式函数：区分“单点聚合”和“多点聚合”
const clusterLayer = new VectorLayer({
  source: clusterSource,
  style: (feature) => {
    const features = feature.get('features') as Feature[];
    const size = features.length;

    if (size === 1) {
      // 只有 1 个点 → 渲染单点图标
      return new Style({
        image: new Icon({
          src: '/地点.png',
          scale: 0.2,
        }),
      });
    } else {
      // 多点 → 画圆圈+数量
      return new Style({
        image: new CircleStyle({
          radius: 15,
          fill: new Fill({ color: '#3399CC' }),
        }),
        text: new Text({
          text: size.toString(),
          fill: new Fill({ color: '#fff' }),
        }),
      });
    }
  },
});

// 4. 添加到 map
map.addLayer(clusterLayer);
```

> - 聚合后的 Feature 本身不是真实的点，而是“容器”；
> - 它有一个属性 `'features'`，里面是聚在一起的原始 Feature 数组。

------

### AnimatedCluster（ol-ext）：带动画的聚合

基本流程

- AnimatedCluster = ol-ext 提供的“带动画的聚合图层”。
- 本质上还是 VectorLayer，只是在聚合更新时有平滑动画。



和上面的 `clusterLayer` 几乎一样，只是 Layer 类换成 AnimatedCluster：

```ts
import AnimatedCluster from 'ol-ext/layer/AnimatedCluster';
import { easeOut } from 'ol/easing';

// 仍然使用前面构建好的 clusterSource

const animatedClusterLayer = new AnimatedCluster({
  source: clusterSource,
  animationDuration: 700,      // 动画时长（毫秒）
  animationEasing: easeOut,    // 缓动函数（可用 easeIn/easeOut/linear）
  style: (feature) => {
    const features = feature.get('features') as Feature[];
    const size = features.length;

    if (size === 1) {
      return new Style({
        image: new Icon({
          src: '/地点.png',
          scale: 0.2,
        }),
      });
    } else {
      return new Style({
        image: new CircleStyle({
          radius: 15,
          fill: new Fill({ color: '#3399CC' }),
        }),
        text: new Text({
          text: size.toString(),
          fill: new Fill({ color: '#fff' }),
        }),
      });
    }
  },
});

map.addLayer(animatedClusterLayer);
```

> - 使用 ol-ext 的 `AnimatedCluster` 替换普通 `VectorLayer`；
> - 其它写法（source/style）和普通聚合完全一样。

------

### Path 动画（要素沿路径运动）

基本流程

- 用一条 `LineString` 定义路径；
- 用一个点 Feature + Icon 标记“移动物体”；
- 用 ol-ext 的 `featureAnimation/Path` 在该路径上播放动画。

适合场景：

- 车辆/轨迹回放；
- “沿线巡游”效果。



```ts
// 1. 使用武汉景点数组 points 构建一条折线路径
import { LineString } from 'ol/geom';
import Feature from 'ol/Feature';

const lineFeature = new Feature({
  geometry: new LineString(points.map((p) => p.coords)),
});

// 2. 创建一个矢量图层，包含这条线
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Style from 'ol/style/Style';
import Stroke from 'ol/style/Stroke';

const lineLayer = new VectorLayer({
  source: new VectorSource({
    features: [lineFeature],
  }),
  style: new Style({
    stroke: new Stroke({
      color: 'blue',
      width: 2,
    }),
  }),
});
map.addLayer(lineLayer);

// 3. 在线起点创建一个点 Feature，作为“移动的小图标”
import Point from 'ol/geom/Point';
import Icon from 'ol/style/Icon';

const movingPoint = new Feature({
  geometry: new Point(points[0].coords),
});

movingPoint.setStyle(
  new Style({
    image: new Icon({
      src: '/地点.png',
      scale: 0.1,
    }),
  })
);

// 把点要素也加到同一个图层（或单独一个图层）
lineLayer.getSource().addFeature(movingPoint);

// 4. 使用 ol-ext 的 Path 动画，让 movingPoint 沿 lineFeature 跑一遍
import Path from 'ol-ext/featureanimation/Path';
// import { easeOut } from 'ol/easing';

let animating = false;

map.on('click', () => {
  if (animating) return;
  animating = true;

  lineLayer.animateFeature(
    movingPoint,
    new Path({
      path: lineFeature.getGeometry() as LineString, // 路径几何
      duration: 4000,                                // 4s 跑完
      // repeat: true,
      // reverse: true,
      // easing: easeOut,
    })
  );

  // 监听动画结束
  lineLayer.on('animationend', (e: any) => {
    if (e.feature === movingPoint) {
      animating = false;
      console.log('路径动画结束');
    }
  });
});
```

> - `lineLayer.animateFeature(feature, new Path({ path, duration }))` 是核心；
> - `Path` 使用的 `path` 就是一个 `LineString` 几何；
> - 你可以通过点击/按钮来触发这段动画。

------

### 小结：“模板思维”

1. Heatmap（热力图）
   - 适合：展示“点密度或强度分布”；
   - 模板：`new Heatmap({ source: new VectorSource({ features }) })`；
   - 可选权重：`weight: (feature) => feature.get('weight') ?? 1`.
2. Cluster（点聚合）
   - 适合：点多而密集时避免界面太乱；
   - 模板：
     - 原始点 → VectorSource；
     - `clusterSource = new Cluster({ distance, source })`；
     - VectorLayer 里 style 函数：`feature.get('features').length` 判断聚合数量。
3. AnimatedCluster（带动画聚合）
   - 适合：聚合视图切换时更平滑好看；
   - 模板：把 VectorLayer 换成 `new AnimatedCluster({ source, animationDuration, style })`。
4. Path 动画（沿线移动）
   - 适合：轨迹回放，车辆/点沿线路径移动；
   - 模板：
     - `LineString` 定义路径；
     - 点 Feature + Icon 作为“运动物”；
     - `lineLayer.animateFeature(pointFeature, new Path({ path, duration }))`。

## 测量 & 事件

### 测量函数：getArea / getLength（`ol/sphere`）

基本概念

- **getLength(geometry)**：算一条线（或多边形边界）的**球面长度**，单位：米。
- **getArea(geometry)**：算一个多边形的**球面面积**，单位：平方米。
- 需要告诉它数据的坐标系（`projection`），一般用的都是 `'EPSG:4326'`。

------

计算线长（测距）

```ts
import { getLength } from 'ol/sphere';

// 假设 geom 是一个 LineString（比如 Draw 画出来的）
const geom = feature.getGeometry(); // LineString

let length = getLength(geom, {
  projection: 'EPSG:4326',   // 你的数据坐标系
});                          // 返回单位：米

const km = length / 1000;
console.log('长度(km)：', km.toFixed(2));
```

- 画一条 LineString → `drawend` 里用 `getLength` 算长度；
- 对多边形 `getLength` 也可以得到“周长”。

------

计算面积（多边形面积）

```ts
import { getArea } from 'ol/sphere';

// 假设 geom 是一个 Polygon（行政区边界、多边形 Feature）
const geom = feature.getGeometry(); // Polygon

let area = getArea(geom, {
  projection: 'EPSG:4326',
});                            // 返回单位：平方米

const sqKm = area / (1000 * 1000);
console.log('面积(km²)：', sqKm.toFixed(2));
```

“万平方千米”：

```ts
const wanSqKm = area / (1000 * 1000 * 10000);
```

> - `projection` 必须和你的数据真实坐标系一致；
> - `getLength` / `getArea` 返回值都以“米”为基础单位。

------

### 事件监听：`on` & 取消监听 `unByKey`

基本概念

- 几乎所有主要对象（`Map`、`View`、`Layer`、`Source`、`Feature`、`Interaction`…）都支持 `.on(type, handler)`。
- `.on` 返回一个 listenerKey，可以用 `unByKey(listenerKey)` 取消监听。

------

基本用法

```ts
// 监听
const key = map.on('click', (evt) => {
  console.log('坐标：', evt.coordinate);
});

// 取消监听
import { unByKey } from 'ol/Observable';
unByKey(key);
```

------

### 常见事件类别速查

#### Map / View：鼠标 & 视图变化



```ts
// 点击地图
map.on('click', (evt) => {
  console.log('点击坐标：', evt.coordinate);  // EPSG 与 view 一致
});

// 鼠标移动（可用于 hover 高亮）
map.on('pointermove', (evt) => {
  // 做命中检测、高亮等
});

// 视图缩放变化
const view = map.getView();

const key = view.on('change:resolution', () => {
  console.log('zoom =', view.getZoom());
});
```

事件类型小表：

| 对象 | 事件名                | 含义                |
| ---- | --------------------- | ------------------- |
| Map  | `'click'`             | 地图点击            |
| Map  | `'pointermove'`       | 鼠标移动            |
| Map  | `'movestart'`         | 视图开始移动        |
| Map  | `'moveend'`           | 视图移动结束        |
| View | `'change:resolution'` | 分辨率/缩放级别变化 |

------

#### VectorSource：要素加载 / 增删



```ts
const source = vectorLayer.getSource();

// 要素添加
source.on('addfeature', (evt) => {
  console.log('新增要素：', evt.feature);
});

// 远程加载完成（GeoJSON 等）
source.on('featuresloadend', () => {
  console.log('要素加载完成');
});

// state 变为 ready（你之前用过）
source.on('change', () => {
  if (source.getState() === 'ready') {
    console.log('矢量数据加载完成，数量：', source.getFeatures().length);
  }
});
```

常见事件：

| 对象         | 事件名                | 含义             |
| ------------ | --------------------- | ---------------- |
| VectorSource | `'addfeature'`        | 添加 Feature     |
| VectorSource | `'removefeature'`     | 移除 Feature     |
| VectorSource | `'featuresloadstart'` | 开始加载远程要素 |
| VectorSource | `'featuresloadend'`   | 完成加载         |
| VectorSource | `'featuresloaderror'` | 加载出错         |

------

#### Interaction：Draw / Select 等



```ts
// Draw
draw.on('drawstart', (evt) => {
  console.log('开始绘制：', evt.feature);
});

draw.on('drawend', (evt) => {
  console.log('绘制完成：', evt.feature);
});

// Select
select.on('select', (evt) => {
  console.log('selected:', evt.selected);
  console.log('deselected:', evt.deselected);
});
```

常见交互事件：

| 交互类型  | 事件名                                | 含义          |
| --------- | ------------------------------------- | ------------- |
| Draw      | `'drawstart'`                         | 开始绘制      |
| Draw      | `'drawend'`                           | 完成绘制      |
| Select    | `'select'`                            | 选中/取消选中 |
| Modify    | `'modifystart'` / `'modifyend'`       | 修改开始/结束 |
| Translate | `'translatestart'` / `'translateend'` | 平移开始/结束 |
| Snap      | `'snap'`                              | 吸附事件      |

------

#### 推荐用法：组件销毁时统一解绑

在真实项目里，你可以把所有 `on` 返回的 key 存起来：



```ts
import { unByKey } from 'ol/Observable';

const keys: any[] = [];

keys.push(map.on('click', handler1));
keys.push(view.on('change:resolution', handler2));
// ...

// 销毁组件 / 页面时统一解绑
keys.forEach(unByKey);
```

------

### 小结

- 测量：
  - `getLength(geometry, { projection })` → 米；画线测距；
  - `getArea(geometry, { projection })` → 平方米；多边形面积。
- 事件：
  - `obj.on(type, handler)` 注册，返回 `key`；
  - `unByKey(key)` 取消；
  - Map/ View/ Source/ Interaction 全家桶都这么用







## ol-ext

https://viglino.github.io/ol-ext/

安装

```
pnpm install ol-ext
```



### 基本介绍

```js
第三方的、开源的 OpenLayers 扩展库

高级控件（Controls）：例如测距工具、图例、比例尺增强版、时间轴控件、多图层切换器、导航控件、鸟瞰图、打印控件等。
交互（Interactions）：例如绘制辅助线、多边形编辑、更复杂的选中、框选、捕捉等交互。
图层和渲染：扩展的图层类型，特殊渲染方式。
样式和符号库：自定义复杂符号、标注。
动画和动态效果：路径动画、移动要素动画等。
```



### 动画效果API

```js
1. ol.featureAnimation.Blink
效果：要素快速闪烁，透明度不断变化出现和消失。
用途：用于强调某个点或区域，吸引用户注意。
2. ol.featureAnimation.Bounce
效果：要素像球一样弹跳，有明显的上下跳动动画。
用途：模拟弹跳效果，增强要素的活跃感。
3. ol.featureAnimation.Drop
效果：要素从地图上方向下掉落，类似物体落地的感觉。
用途：体现要素生成或加载的过程，给人“落地生根”的感觉。
4. ol.featureAnimation.Fade
效果：要素缓慢地淡入或淡出，透明度平滑过渡。
用途：常用于要素出现或消失的平滑过渡。
5. ol.featureAnimation.None / ol.featureAnimation.Null
效果：无动画效果。
用途：作为占位或禁用动画时使用。
6. ol.featureAnimation.Path
效果：要素沿着指定路径运动，比如线形路径上的移动。
用途：表现要素沿轨迹移动，如车辆轨迹播放、巡航等。
7. ol.featureAnimation.Shake
效果：要素快速左右或上下颤抖抖动。
用途：用于提示错误或需要注意的要素，类似“抖动警告”。
8. ol.featureAnimation.Show
效果：要素由小变大或由隐藏变显现，类似“出现”动画。
用途：强调要素的显示过程。
9. ol.featureAnimation.Slide
效果：要素从屏幕一侧滑入或滑出。
用途：表现要素的进入或退出动画，增加流畅感。
10. ol.featureAnimation.Teleport
效果：要素瞬间跳跃到另一个位置，快速“传送”效果。
用途：表现快速移动或“瞬移”，适合快速状态切换。
11. ol.featureAnimation.Throw
效果：要素像被抛掷一样运动，有加速和抛物线轨迹的感觉。
用途：增强要素动态感，表现抛掷、飞行等动作。
12. ol.featureAnimation.Zoom
效果：要素从小到大放大，或从大到小缩小。
用途：用于强调、聚焦，表现要素的放大过程。
13. ol.featureAnimation.ZoomOut
效果：与 Zoom 相反，要素由大变小逐渐消失。
用途：表现要素渐远、缩小、淡出的效果。
```



## 可直接copy + 效果图

### 地图初始化

使用vite初始化项目

如果要从本地引入资源，需要使用打包平台打包ol平台的代码

```
pnpm init
pnpm install vite -D
pnpm create vite
Select a framework： Vanilla  // 标准框架，用原生
```

获取ol资源

```
pnpm install ol
```

html

```html
<style>
  body {
    margin: 0;
  }
  #map {
    width: 100vw;
    height: 100vh;
  }
</style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

main.js

```js
import Map from 'ol/Map.js'; // 引入Map类
import TileLayer from 'ol/layer/Tile.js'; // 引入TileLayer类
import View from 'ol/View.js'; // 引入View类
import OSM from 'ol/source/OSM.js'; // 引入OSM类
import 'ol/ol.css'; // 引入OpenLayers的CSS样式

import { XYZ } from "ol/source" 

const map1 = new Map({
  target: "map",  // 地图容器的id属性值
  layers: [
    new TileLayer({
      // source: new OSM()
      source: new XYZ({
        url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}"  // 图层的资源定位符，定位具体的服务提供者
      })
    })
  ],
  view: new View({
    projection: "EPSG:4326", // 设置地图投影
    center: [114.25, 30.59],  // 经纬度定位
    zoom: 8, // 缩放等级
  })
})
```

### 视图

main.js

```js
import Map from 'ol/Map.js'
import TileLayer from "ol/layer/Tile"
import View from 'ol/View.js'
import OSM from 'ol/source/OSM.js'
import 'ol/ol.css'

const view = new View({
  projection: "EPSG:4326", 
  center: [114.25, 30.59],  
  zoom: 8, 
})
const map1 = new Map({
  target: "map",
  layers: [
    new TileLayer({
      source: new OSM()
    })
  ],
  view: view,
})

const Upbtn = document.querySelector('.up')
Upbtn.addEventListener('click', () => {
  let [x,y] = map1.getView().getCenter() // 获取视图中心点
  y += 0.1 // 向上移动0.1度
  map1.getView().setCenter([x, y])  // 设置新的中心点
})
....
const goTobtn = document.querySelector('.goTo')
goTobtn.addEventListener('click', () => {
  view.animate(  // animate是View的方法，改变视图时使用动画过渡
    {center: [113.27, 23.13]},
    {zoom: 12},
    {duration: 2000},
  )
})
```



html

```html
 <body>
  <div id="app">
    <div id="map"></div>
    <div>
      <button class="up">上移</button>
      <button class="down">下移</button>
      <button class="left">左移</button>
      <button class="right">右移</button>
      <button class="goTo">去到广州</button>
    </div>
  </div>
  <script type="module" src="/src/main.js"></script>
</body>
```

css

```css
<style>
  body {
    margin: 0;
  }
  #map {
    width: 100vw;
    height: 100vh;
  }
  .up {
    position: absolute;
    top: 10px;
    left: 60px;
  }
  .down {
    position: absolute;
    top: 50px;
    left: 60px;
  }
  .left {
    position: absolute;
    top: 30px;
    left: 30px;
  }
  .right {
    position: absolute;
    top: 30px;
    left: 100px;
  }
  .goTo {
    position: absolute;
    top: 30px;
    left: 150px;
  }
</style>
```





### 图层和数据源

main.js

```js
import Map from 'ol/Map.js'
import TileLayer from 'ol/layer/Tile.js'  // 瓦片图层类
import View from 'ol/View.js'
import OSM from 'ol/source/OSM.js'  // OSM瓦片数据源类
import XYZ from 'ol/source/XYZ.js'  // 瓦片数据源类

const view = new View({
  projection: "EPSG:4326", 
  center: [114.25, 30.59],  
  zoom: 8, 
})
// OSM瓦片图层
const osmLayer = new TileLayer({
  source: new OSM()
})
// 高德瓦片图层
const gaodeLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}"  // 图层的资源定位符，定位具体的服务提供者
  })
})
// 高德影像图层
const gaodeImageLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=6&x={x}&y={y}&z={z}"
  })
})
// 高德文字标注图层
const gaodeTextLayer = new TileLayer({
  source: new XYZ({
    url: "http://webst02.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scale=1&style=8"
  })
})
// 高德矢量图层
const gaodevVectorLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}"
  })
})

// 天地图影像图层  	天地图key 4a050b56e21976c2bb53fec757797dc1
const tiandituImgLayer = new TileLayer({
  source: new XYZ ({
    url: "http://t0.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=4a050b56e21976c2bb53fec757797dc1"
  })
})
// 天地图矢量图层
const tiandituVectorLayer = new TileLayer({
  source: new XYZ ({
    url: "http://t0.tianditu.gov.cn/vec_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=vec&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=4a050b56e21976c2bb53fec757797dc1"
  })
})
// 天地图矢量注记图层
const tiandituTextLayer = new TileLayer({
  source: new XYZ ({
    url: "http://t0.tianditu.gov.cn/cva_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=cva&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=4a050b56e21976c2bb53fec757797dc1"
  })
})
const map1 = new Map({
  target: "map",
  layers: [gaodeLayer,gaodeImageLayer,gaodeTextLayer, gaodevVectorLayer], // 图层添加要按照顺序，后面的会盖在前面的
  view: view,
})

// 修改图层层级关系
const SateBtn = document.querySelector('.satellite')
SateBtn.addEventListener('click', () => {
  map1.setLayers([gaodeImageLayer,gaodeTextLayer])  // 每次切换都会触发内部的“移除-添加”流程。每个 TileLayer 都要重新注册事件，重新渲染，可能有轻微的性能损耗

  //ol/layer/Group 下 setZIndex方法也能做  不会重新初始化图层对象，已有的瓦片缓存也不会丢失
  //ol/layer/Group 下 setVisible方法也能做  只改变显示层级，不涉及图层对象的增删，几乎没有性能损耗
    // satelliteLayer.setZIndex(99);
    // textLayer.setZIndex(100);
    // gaodeLayer.setZIndex(101);
    // gaodeLayer.setVisible(true)
})

const VecBtn = document.querySelector('.vector')
VecBtn.addEventListener('click', () => {
  map1.setLayers([gaodevVectorLayer])
  //ol/layer/Group 下 setZIndex方法也能做
  //ol/layer/Group 下 setVisible方法也能做
})
```

html

点击按钮能切换图层

```html
<body>
  <div id="app">
    <div id="map"></div>
    <div>
      <button class="satellite">切换为卫星图层</button>
      <button class="vector">切换为矢量图层</button>
    </div>
  </div>
  <script type="module" src="/src/图层和数据源.js"></script>
</body>
```

css

```css
<style>
  body {
    margin: 0;
  }
  #map {
    width: 100vw;
    height: 100vh;
  }
  .satellite {
    position: absolute;
    top: 30px;
    left: 300px;
  }
  .vector {
    position: absolute;
    top: 30px;
    left: 600px;
  }
</style>
```



### 静态图片图层

main.js

```js
import Map from 'ol/Map.js'
import TileLayer from 'ol/layer/Tile.js'
import View from 'ol/View.js'

// https://openlayers.org/en/latest/apidoc/module-ol_source_ImageStatic-Static.html
import Static from 'ol/source/ImageStatic.js'; // 新增静态图片数据源类
import ImageLayer from 'ol/layer/Image.js' // 新增静态图片图层类



// 静态图片图层
let center = [114.25, 30.59]
const imageExtent = [center[0]-0.5, center[1]-0.5, center[0]+0.5, center[1]+0.5]
const imageLayer = new ImageLayer({
  source: new Static({
    url: "../public/vite.svg",
    imageExtent: imageExtent  // 图片的地理范围  [minX, minY, maxX, maxY]
  })
})

const view = new View({
  projection: "EPSG:4326", 
  center: [114.25, 30.59],  
  zoom: 8, 
})
const map1 = new Map({
  target: "map",
  layers: [tiandituVectorLayer,imageLayer], // 图层添加要按照顺序，后面的会盖在前面的
  view: view,
})
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="app">
    <div id="map"></div>
  </div>
  <script type="module" src="/src/静态图片图层.js"></script>
</body>
```





<u>map 和图层 都从上面找就完了，和下面main.js代码拼起来就能用</u>



### 矢量图层

实例化VectorLayer类 生成矢量图层

main.js

```js
import Map from 'ol/Map.js'
import TileLayer from 'ol/layer/Tile.js'
import View from 'ol/View.js'

import XYZ from 'ol/source/XYZ.js'  // 瓦片数据源类

import VectorLayer from 'ol/layer/Vector.js'; // 新增矢量图层类
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import GeoJSON from 'ol/format/GeoJSON.js'; // 新增GeoJSON格式类

import Style from 'ol/style/Style.js';  // 新增样式类
import Fill from 'ol/style/Fill.js'; // 新增填充样式类
import Stroke from 'ol/style/Stroke.js' // 新增描边样式类

const view = new View({
  projection: "EPSG:4326", 
  center: [114.25, 30.59],  
  zoom: 4, 
})

const map1 = new Map({
  target: "map",
  layers: [], // 图层添加要按照顺序，后面的会盖在前面的
  view: view,
})

// 创建一个矢量图层
const ChinaVectorLayer = new VectorLayer({
  source: new VectorSource({
    url: "https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json",
    format: new GeoJSON(), // 指定数据格式
  }),
  style: new Style({
    fill: new Fill({ // 填充样式
      color: 'rgba(255, 0, 0, 0.5)' // 半透明红色
    }),
    stroke: new Stroke() // 描边样式
  })
})
map1.addLayer(ChinaVectorLayer)

// source中加载资源是异步操作，有时需要监听加载完毕后执行回调操作
ChinaVectorLayer.getSource().on('change', function(e) {
  const features = ChinaVectorLayer.getSource().getFeatures()
  if (ChinaVectorLayer.getSource().getState() === 'ready') {
    console.log('矢量数据加载完成，可以在这里执行后续操作');
    console.log(features.length);
  }
});
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/矢量图层.js"></script>
</body>
```

<img src="C:./Typora图片/image-20250905173024825.png" alt="image-20250905173024825" style="zoom: 67%;" />





#### 要素绘制 Feature

实例化Feature类绘制要素

#### 点要素

在空的矢量图层中，在点击处添加点要素

每个点的样式添加Icon,并且在文本中显示坐标

main.js

```js
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import Feature from 'ol/Feature.js'; // Feature类
import Point from 'ol/geom/Point.js';


import Icon from 'ol/style/Icon.js';
import Text from 'ol/style/Text.js';

import Style from 'ol/style/Style.js'; // Style类 
import Fill from 'ol/style/Fill.js';


// 创建一个空矢量图层
const markerLayer = new VectorLayer({
  source: new VectorSource()
})
map1.addLayer(markerLayer)

map1.on("click", e => { // 监听点击事件
  // 定义一个Icon样式
  const style = new Style({
    image: new Icon({
      src: "/401位置.png",
      scale: 0.2, // 根据需要调整缩放比例
      anchor: [0.5, 1], // 锚点设置在图标底部中心 anchor[0]控制左右 anchor[1]控制上下
    }),
    text: new Text({  // 文本在样式类中，
      text: '点击位置',
      font: '10px Calibri,sans-serif',
      fill: new Fill({
        color: 'orange'
      })
    })
  })

  // 创建一个点要素
  const point = new Feature({
    geometry: new Point(e.coordinate)
  })

  point.setStyle(style); // 设置要素的样式

  // 将要素添加到矢量图层的源中
  markerLayer.getSource().addFeature(point);
})
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

![GIF 2025-09-05 17-25-58](C:./Typora图片/GIF 2025-09-05 17-25-58.gif)





#### 线要素

正确构建 LineString 几何对象

将几何体包装成 Feature（要素）

将素放进数据源 VectorSource

```js
// 线要素绘制
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import LineString from 'ol/geom/LineString.js';
import Feature from 'ol/Feature.js'; // Feature类

import Style from 'ol/style/Style.js'; // Style类
import Stroke from 'ol/style/Stroke.js';
// 画一条从武汉到北京的线
const line = new Feature({ // 将几何体包装成 Feature（要素）
  geometry: new LineString([  // 正确构建 LineString 几何对象
    [114.31, 30.52], [116.40, 39.90]
  ])
});
const VectorLineLayer =  new VectorLayer({
  source: new VectorSource({ 
    features: [line]  // 将素放进数据源 VectorSource
  }),
  style: new Style({  // 为矢量图层中的矢量要素添加样式
    stroke: new Stroke({
      color: 'red',
      width: 2
    })
  })
})
map1.addLayer(VectorLineLayer)
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

![image-20250905172445059](C:./Typora图片/image-20250905172445059.png)



##### 例子

html同上

监听地图点击事件，获取点击坐标

创建点要素，叠加样式（icon和文本）

创建矢量图层 VectorLayer 并添加要素

判断是否已点击两次，根据两点连线，绘制线要素

```js
// 点击两个点，画线,显示点经纬度，还有icon
import VectorLayer from 'ol/layer/Vector.js' // 新增矢量层类
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import LineString from 'ol/geom/LineString.js';
import Feature from 'ol/Feature.js'; // Feature类
import Point from 'ol/geom/Point.js';

import Style from 'ol/style/Style.js'; // Style类
import Stroke from 'ol/style/Stroke.js';
import Fill from 'ol/style/Fill.js';

import Icon from 'ol/style/Icon.js';
import Text from 'ol/style/Text.js';

let arr = []

map1.on("click", e => {
  let lastCoordinate = e.coordinate;
  e.coordinate[0] = e.coordinate[0].toFixed(2); // 保留两位小数
  e.coordinate[1] = e.coordinate[1].toFixed(2);
  arr.push(lastCoordinate);

  const point = new Feature({
    geometry: new Point(arr[arr.length - 1])
  })
  point.setStyle(new Style({
    image: new Icon({
      src: "/401位置.png",
      scale: 0.2 // 根据需要调整缩放比例
    }),
    text: new Text({
      text: `${arr[arr.length - 1]}`,  // 文本内容
      offsetY: 20, // 正值向下偏移
      fill: new Fill({
        color: 'orange'
      })
    })
  }))

  const VectorLineLayer =  new VectorLayer({
    source: new VectorSource({
      features: [point]
    }),
    style: new Style({
      stroke: new Stroke({
        color: 'red',
        width: 2
      })
    })
  })
  map1.addLayer(VectorLineLayer);

  if (arr.length >= 2) { // 点击两次及以上,收集到足够的数据后，用数组中的最后两个坐标画线
    const line = new Feature({
      geometry: new LineString([arr[arr.length - 2], arr[arr.length - 1]])
    })
    VectorLineLayer.getSource().addFeature(line);  // 将线要素添加到图层数据源中
  }

})
```

<img src="C:./Typora图片/GIF 2025-09-05 17-17-58.gif" alt="GIF 2025-09-05 17-17-58" style="zoom: 67%;" />



#### 圆形要素

main.js

```js
// 在地图上创建圆形要素
import VectorLayer from 'ol/layer/Vector.js' // 新增矢量层类
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import Feature from 'ol/Feature.js'; // Feature类
import Circle from 'ol/geom/Circle.js';

const circleFeature = new Feature({
  geometry: new Circle(
    [114.25, 30.59], 5 // 圆心坐标，半径（单位：米）
  )
})
// 创建一个矢量图层来显示圆形要素
const circleFeatureLayer = new VectorLayer({
  source: new VectorSource({})
})
map1.addLayer(circleFeatureLayer) // 将圆形图层添加到地图上
circleFeatureLayer.getSource().addFeature(circleFeature) // 将圆形要素添加到图层的数据源中
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

<img src="C:./Typora图片/image-20250905170808079.png" alt="image-20250905170808079" style="zoom:80%;" />



#### 多边形

在地图上绘制多边形 这里选三角形

main.js

```js
import VectorLayer from 'ol/layer/Vector.js' // 新增矢量层类
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import Feature from 'ol/Feature.js'; // Feature类
import Polygon from 'ol/geom/Polygon.js';
import Style from 'ol/style/Style.js'; // Style类
import Fill from 'ol/style/Fill.js';


const triangleFeature = new Feature({
  geometry: new Polygon([[
    [114.25, 35.59],
    [124.26, 25.60],
    [104.27, 25.59],
    [114.25, 35.59]  // 三角形最后要添加一个坐标，与第一个坐标闭合
  ]]),
  style: new Style({
    fill: new Fill({
      color: 'rgba(255, 0, 0, 0.5)' // 半透明红色填充
    })
  })
})
// 创建一个矢量图层来显示多边形要素
const triangleFeatureLayer = new VectorLayer({
  source: new VectorSource({})
})
map1.addLayer(triangleFeatureLayer)
triangleFeatureLayer.getSource().addFeature(triangleFeature) // 将多边形要素添加到图层的数据源中
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

<img src="C:./Typora图片/image-20250905170622592.png" alt="image-20250905170622592" style="zoom:80%;" />



### 交互 & 覆盖物

#### 绘图交互 draw

创建用于存放绘制要素的矢量图层

通过 `map1.addLayer(drawLayer)` 把绘制图层添加到地图上。

创建并配置交互对象 new Draw

使用 `map1.addInteraction(draw)` 把绘制交互添加到地图。

main.js

```js
创建用于存放绘制要素的矢量图层


// 交互，在地图上draw 绘图交互
import Vector from 'ol/layer/Vector.js'; // 新增矢量层类
import VectorSource from 'ol/source/Vector.js'; // 新增矢量数据源类
import Style from 'ol/style/Style.js'; // Style类
import Stroke from 'ol/style/Stroke.js';

// 创建一个空矢量图层，用来放置交互绘制的要素
const drawLayer = new Vector({
  source: new VectorSource(),
  style: new Style({  // 设置图层中要素的样式
    stroke: new Stroke({
      color: 'blue',
      width: 2
    })
  })
})
map1.addLayer(drawLayer)

// 创建一个draw交互
import Draw from 'ol/interaction/Draw.js';
const draw = new Draw({
  type: 'LineString', // 交互绘制的类型 geom下的geometry对应的 'Point' | 'LineString' | 'Polygon' | 'Circle' | 'Box' | 'Sphere'
  source: drawLayer.getSource(), // 指定绘制的要素放置到哪个图层的数据源中
  //freehand: true, // 像画笔一样随意作画
})
map1.addInteraction(draw) // 添加交互到地图上
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

<img src="C:./Typora图片/GIF 2025-09-05 17-03-39.gif" alt="GIF 2025-09-05 17-03-39" style="zoom:80%;" />



#### 选择交互 select+ 覆盖物

1. 加载省级GeoJSON矢量数据并显示在地图上
   - 创建矢量图层，指定GeoJSON数据源并添加到地图。
2. 添加选择交互并自定义选中样式
   - 创建Select交互，设置选中矢量要素(feature)样式为高亮填充和描边。
3. 将选择方式改为鼠标悬停
   - 设置Select的condition为`pointerMove`，实现悬停高亮而非点击。
4. 创建覆盖物（Overlay）用于展示选中要素信息
   - 使用Overlay关联自定义HTML元素，并添加到地图。
5. 监听交互事件，动态显示要素属性
   - 监听select事件，提取选中要素属性（如adcode、name、level），并在覆盖物中展示，同时根据选中状态更新覆盖物位置或隐藏。

main.js

```js
// 选择交互 + 覆盖物
// 定义一个矢量图层，设置要素为中国各省，GeoJSON格式
// import VectorLayer from 'ol/layer/Vector.js'
import VectorSource from 'ol/source/Vector'
import GeoJSON from 'ol/format/GeoJSON'
const ChinaVectorLayer = new VectorLayer({
  source: new VectorSource({
    url: "https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json",
    format: new GeoJSON(), // 指定数据格式
  })
})
map1.addLayer(ChinaVectorLayer)

// 创建一个select交互 
import Select from 'ol/interaction/Select.js'
const select = new Select() // 默认是点击选中
map1.addInteraction(select) // 添加交互到地图上

// 设置选中要素的样式
import { Fill, Stroke } from 'ol/style.js'
import Style from 'ol/style/Style.js'
const newStyle = new Style({
  fill: new Fill({
    color: 'rgba(33, 150, 243, 0.5)'
  }),
  stroke: new Stroke({
    color: 'rgba(13, 71, 161, 0.5)',
    width: 2
  })
})
select.style_ = newStyle // 设置选中要素的样式

import { pointerMove } from 'ol/events/condition.js' // 鼠标移动事件
select.condition_ = pointerMove // 设置为鼠标悬停选中

// 创建一个覆盖物overlay
import Overlay from 'ol/Overlay.js'
const overlay = new Overlay({ // 创建一个覆盖物
  element: document.querySelector('.overlay'), // 关联HTML中的div.overlay
})
map1.addOverlay(overlay) // 添加覆盖物到地图上

// 监听select交互的select事件
select.on('select', e => {
  // console.log(e)
  if (e.selected.length > 0) {
    let coordinate = e.selected[0].values_.center // 获取选中要素的中心经纬度
    overlay.setPosition(coordinate) // 设置覆盖物位置
    // 获取选中要素的信息
    let adcode = e.selected[0].values_.adcode
    let name = e.selected[0].values_.name
    let level = e.selected[0].values_.level
    // 设置覆盖物内容
    document.querySelector('.overlay').innerHTML =`
      <p>adcode:${adcode}</p>
      <p>name:${name}</p>
      <p>level:${level}</p>
    `
  } else {
    overlay.setPosition(undefined) // 取消覆盖物位置
  }
})
```

html

```html
  <style>
    body {
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
    .overlay {
      position: absolute;
      background-color: rgba(255, 255, 255, 0.8);
      border-radius: 10%;
      padding: 4px;
      box-shadow: 0px 1px 5px rgba(0, 0, 0, 0.3);
    }
    .overlay p {
      margin: 1px 5px;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <!-- openlayers 上的覆盖物-->
  <div class="overlay"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

![GIF 2025-09-05 16-59-22](C:./Typora图片/GIF 2025-09-05 16-59-22.gif)



#### draw / select 测距功能实现

面积周长测量的实现

1. 创建并添加矢量图层
   - 使用 `VectorLayer` 和 `VectorSource` 加载 GeoJSON 矢量数据（中国行政区划边界）。
   - 添加到地图实例中。
2. 添加鼠标悬停的选择交互
   - 使用 `ol/interaction/Select` 并设置条件为 `pointerMove`（鼠标移动悬停触发）。
   - 将选择交互添加到地图。
3. 创建Overlay覆盖物用于显示测量结果
   - 使用 `ol/Overlay` 绑定页面上预定义的 DOM 元素（class为`.overlay`）。
   - 添加到地图。
4. 监听选择事件
   - 当用户鼠标悬停（选中）某个要素时，获取该要素的几何信息。
   - 调用 `getLength` 计算周长（单位：米），除以1000转为公里。
   - 调用 `getArea` 计算面积（单位：平方米），除以千万转成“万平方千米”。
   - 设置 Overlay 显示位置为要素中心，更新测量结果内容。
   - 当无要素选中时，隐藏 Overlay。



直线测距功能的实现

1. 创建矢量图层用于存储绘制内容
   - 新建空 `VectorLayer` 和 `VectorSource`，并指定样式（红色线条）。
   - 添加到地图。
2. 添加绘制交互
   - 使用 `ol/interaction/Draw`，类型为 `'LineString'`，并指定存入上面创建的 `VectorSource`。
   - 添加到地图。
3. 创建Overlay覆盖物用于显示测距结果
   - 使用 `ol/Overlay` 绑定页面上预定义 DOM 元素（class为`.overlay-draw`）。
   - 添加到地图。
4. 监听绘制结束事件
   - 绘制完成后触发 `drawend` 事件。
   - 使用 `getLength` 计算绘制线的长度（单位：米），除以1000转为公里。
   - 获取线的终点坐标，设置 Overlay 显示位置。
   - 更新测距结果的文字内容。
   - 删除之前绘制的要素，保证图层中只存在当前绘制的折线。



main.js

```js
// 新建一个地图实例
import Map from 'ol/Map.js';
import View from 'ol/View.js';
import TileLayer from 'ol/layer/Tile';

const map = new Map({
  target: 'map',
  layers:[],
  view: new View({
    center: [114.25, 30.59],
    zoom: 4,
    projection: 'EPSG:4326' // 墨卡托投影
  })
})

// 第三方图层
// 高德影像图层
import XYZ from 'ol/source/XYZ.js';
const gaodeImageLayer = new TileLayer({
  source: new XYZ({
    url: "http://wprd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=6&x={x}&y={y}&z={z}"
  })
})
map.addLayer(gaodeImageLayer);


import VectorLayer from 'ol/layer/Vector.js';
import VectorSource from 'ol/source/Vector.js'; // 矢量数据源
import GeoJSON from 'ol/format/GeoJSON.js'; // 数据格式
import Style from 'ol/style/Style';
// 自定义矢量图层
const VectorChinaLayer = new VectorLayer({
  source: new VectorSource({
    url: 'https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json',
    format: new GeoJSON(), // 数据格式
  }),
})
map.addLayer(VectorChinaLayer);




// 添加鼠标悬停的select交互
import Select from 'ol/interaction/Select.js';
import { pointerMove } from 'ol/events/condition';
const select = new Select({
  condition: pointerMove,
})
map.addInteraction(select);

// 创建一个覆盖物 overlay
import Overlay from 'ol/Overlay.js';
const overlay = new Overlay({
  element: document.querySelector('.overlay')
})
map.addOverlay(overlay);

// 监听select事件 计算选中元素的周长和面积
import { getArea, getLength } from 'ol/sphere.js';
select.on('select', e => {
  // console.log(e)
  if (e.selected.length > 0) {
    // 获取选中要素的周长
    const feature = e.selected[0];
    const geometry = e.selected[0].values_.geometry // 等同于const geometry = feature.getGeometry();
    let length = getLength(geometry, { projection: 'EPSG:4326' });
    length = length/1000
    let area = getArea(geometry, { projection: 'EPSG:4326' })
    area = area/(1000*1000*10000)

    // 将结果显示在页面上
    overlay.setPosition(feature.values_.center); // 将overlay位置设置为要素的中心点
    document.querySelector('.overlay').innerHTML = `
      <p>周长：${length.toFixed(2)} 千米</p>
      <p>面积：${area.toFixed(2)} 万平方千米</p>
    `

  } else {
    overlay.setPosition(undefined); // 隐藏overlay
    
  }
})





// 绘制直线测距
// 配置draw对象 绘制的geom是LineString
import Draw from 'ol/interaction/Draw.js';
import Stroke from 'ol/style/Stroke';
// 创建一个空矢量图层，用来放置绘制的矢量几何元素
const drawLayer = new VectorLayer({
  source: new VectorSource(),
  style: new Style({
    stroke: new Stroke({
      color: 'red',
      width: 2
    })
  })
})
map.addLayer(drawLayer);
// 创建一个draw
const draw = new Draw({
  type: 'LineString',
  source: drawLayer.getSource(),
})
map.addInteraction(draw);
// 创建一个overlay 用来显示测量结果
const drawOverlay = new Overlay({
  element: document.querySelector('.overlay-draw')
})
map.addOverlay(drawOverlay)
// 监听draw的drawend事件
draw.on('drawend', e => {
  // console.log(e)
  
  // 计算线的长度
  let length = getLength(e.feature.getGeometry(), { projection: 'EPSG:4326' });
  length = length / 1000;

  // 获取绘制的几何对象末端端点位置
  const coordinates = e.feature.getGeometry().getCoordinates()[1];

  // 将结果显示到页面上
  drawOverlay.setPosition(coordinates)
  document.querySelector('.overlay-draw').innerHTML = `长度：${length.toFixed(2)} 千米`

  // 绘制完成后删除前面那个要素
  if (drawLayer.getSource().getFeatures().length > 0) { // 说明已经绘制过了
    drawLayer.getSource().removeFeature(drawLayer.getSource().getFeatures()[0]) // 删除第一个要素
  }
  
})



// 设置按钮切换交互
const btnSelect = document.querySelector('.btn-select');
const btnDraw = document.querySelector('.btn-draw');
btnSelect.addEventListener('click', () => {
  map.removeInteraction(draw); // 移除绘制交互
  map.addInteraction(select); // 添加悬停交互
  
  drawLayer.getSource().clear(); // 清空绘制图层要素

  // 切回悬停模式时主动清空测距 drawOverlay 位置与内容
  drawOverlay.setPosition(undefined); // 隐藏overlay

  document.querySelector('.overlay-draw').style.display = 'none'; // 隐藏测距结果
  document.querySelector('.overlay').style.display = 'block'; // 显示测量结果
})
btnDraw.addEventListener('click', () => {
  map.removeInteraction(select); // 移除悬停交互
  map.addInteraction(draw); // 添加绘制交互
  document.querySelector('.overlay').style.display = 'none'; // 隐藏测量结果
  document.querySelector('.overlay-draw').style.display = 'block'; // 显示测距结果
})

// 默认启动悬停交互
map.removeInteraction(draw); // 移除绘制交互

```

html

```html
<body>
  <div id="map"></div>
  <div class="overlay"></div>
  <div class="overlay-draw"></div>

  <button class="btn-select">切换为交互测距</button>
  <button class="btn-draw">切换为绘制直线测距</button>
  
  <script type="module" src="/src/main.js"></script>
</body>
```

css

```css
<style>
  #map {
    width: 100vw;
    height: 100vh;
  }
  .overlay, .overlay-draw {
    position: absolute;
    z-index: 1000;
    width: 200px;
    background-color: rgba(0, 0, 0, 0.5);
    color: white;
    border-radius: 8px;
    padding: 10px;
  }
  .btn-select,.btn-draw {
    position: absolute;
    z-index: 1000;
  }
  .btn-select {
    top: 20px;
    left: 200px;
  }
  .btn-draw {
    top: 60px;
    left: 200px;
  }
</style>
```

![GIF 2025-09-08 11-34-55](C:./Typora图片/GIF 2025-09-08 11-34-55.gif)



### 热力图

- `Heatmap` 继承自 `BaseVectorLayer`
- `VectorLayer` 也继承自 `BaseVectorLayer`

热力图实现关键代码片段

```js
//创建20个武汉市景点的经纬度坐标
const points = [
  { name: "黄鹤楼", coords: [114.3162, 30.5278] },
  { name: "武汉博物馆", coords: [114.288, 30.547] },
  { name: "武汉大学", coords: [114.366, 30.538] },
  { name: "汉口里", coords: [114.294, 30.581] },
  { name: "武汉园博园", coords: [114.406, 30.485] },
  { name: "武汉长江大桥", coords: [114.284, 30.586] },
  { name: "江汉路步行街", coords: [114.287, 30.583] },
  { name: "武汉汉正街", coords: [114.273, 30.59] },
  { name: "东湖绿道", coords: [114.387, 30.532] },
  { name: "武汉科技馆", coords: [114.295, 30.55] },
  { name: "归元寺", coords: [114.304, 30.539] },
  { name: "武汉汉口北", coords: [114.296, 30.615] },
  { name: "武汉体育中心", coords: [114.323, 30.566] },
  { name: "武汉江滩", coords: [114.277, 30.546] },
  { name: "武汉植物园", coords: [114.395, 30.493] },
  { name: "武汉汉阳造", coords: [114.254, 30.541] },
  { name: "武汉儿童医院", coords: [114.313, 30.563] },
  { name: "黄鹤楼公园", coords: [114.354, 30.527] },
  { name: "武汉科学技术博物馆", coords: [114.298, 30.547] },
  { name: "武汉盛世华庭", coords: [114.36, 30.57] },
  { name: "武汉国际博览中心", coords: [114.359, 30.571] },
];


// 热力图
// points数组转换为点要素数组
import Feature from 'ol/Feature.js';
import Point from 'ol/geom/Point.js';
// 创建一个点要素数组
const pointFeatures = points.map(item => {
  const feature = new Feature({
    geometry: new Point(item.coords),
    name: item.name,
  })
  return feature;
})

// 创建热力图层
import Heatmap from 'ol/layer/Heatmap.js'; // 热力图图层
import VectorSource from 'ol/source/Vector.js'; // 矢量数据源
const heatmapLayer = new Heatmap({
  source: new VectorSource({
    features: pointFeatures
  })
})
map.addLayer(heatmapLayer);
```

![GIF 2025-09-08 20-41-53](C:./Typora图片/GIF 2025-09-08 20-41-53.gif)

### 聚合图层

聚合图层仍然是矢量图层

实现步骤：

```
构造一组可被 OpenLayers 识别和聚合的点要素。

先用 VectorSource 把点放进去
再用 Cluster 包装这个 VectorSource，实现聚合
最终在 VectorLayer 上使用 Cluster，显示聚合结果
```



```js
//创建20个武汉市景点的经纬度坐标
const points = [
  { name: "黄鹤楼", coords: [114.3162, 30.5278] },
  { name: "武汉博物馆", coords: [114.288, 30.547] },
  { name: "武汉大学", coords: [114.366, 30.538] },
  { name: "汉口里", coords: [114.294, 30.581] },
  { name: "武汉园博园", coords: [114.406, 30.485] },
  { name: "武汉长江大桥", coords: [114.284, 30.586] },
  { name: "江汉路步行街", coords: [114.287, 30.583] },
  { name: "武汉汉正街", coords: [114.273, 30.59] },
  { name: "东湖绿道", coords: [114.387, 30.532] },
  { name: "武汉科技馆", coords: [114.295, 30.55] },
  { name: "归元寺", coords: [114.304, 30.539] },
  { name: "武汉汉口北", coords: [114.296, 30.615] },
  { name: "武汉体育中心", coords: [114.323, 30.566] },
  { name: "武汉江滩", coords: [114.277, 30.546] },
  { name: "武汉植物园", coords: [114.395, 30.493] },
  { name: "武汉汉阳造", coords: [114.254, 30.541] },
  { name: "武汉儿童医院", coords: [114.313, 30.563] },
  { name: "黄鹤楼公园", coords: [114.354, 30.527] },
  { name: "武汉科学技术博物馆", coords: [114.298, 30.547] },
  { name: "武汉盛世华庭", coords: [114.36, 30.57] },
  { name: "武汉国际博览中心", coords: [114.359, 30.571] },
];



// 聚合图层
// points数组转换为点要素数组
import Feature from 'ol/Feature.js';
import Point from 'ol/geom/Point.js';
// 创建一个点要素数组
const pointsFeatures = points.map(item => {
  const feature = new Feature({
    geometry: new Point(item.coords),
    name: item.name,
  })
  return feature;
})



// import TileLayer from 'ol/layer/Tile'; 聚合图层仍然是矢量图层
// https://openlayers.org/en/latest/apidoc/module-ol_source_Cluster-Cluster.html
import Cluster from 'ol/source/Cluster.js'; // 引入聚类矢量数据源
import VectorSource from 'ol/source/Vector'; // 引入矢量数据源 聚类矢量数据源的source需要一个矢量数据源 	VectorSource<FeatureType>
const clusterSource = new Cluster({  // 创建一个聚合矢量数据源
  source: new VectorSource({
    features: pointsFeatures
  }),
})



import VectorLayer from 'ol/layer/Vector.js'; // 矢量图层
import { Icon, Style } from 'ol/style.js'; // 样式
import CircleStyle from 'ol/style/Circle'; // 圆圈样式
import Fill from 'ol/style/Fill.js'; // 填充样式
import Text from 'ol/style/Text.js'; // 文字样式
// 创建一个聚合图层
const clusterLayer = new VectorLayer({
  source: clusterSource,
  style: function(features) { // features是一个数组，数组里是聚合后的点要素
    const size = features.values_.features.length; // 聚合后点要素的个数   相当于const size = features.get('features').length;
    // 如果没有聚合在一起，就显示要给置顶的Icon
    if (size === 1) {
      return new Style({
        image: new Icon({
          src: '/地点.png',
          scale: 0.2,
        })
      })
    } else { // 如果聚合在一起，就显示一个圆圈，圆圈里显示聚合的个数
      return new Style({
        image: new CircleStyle({ // 圆圈样式
          radius: 15,
          fill: new Fill({ color: '#3399CC' }),
        }),
        text: new Text({ // 文字样式
          text: size.toString(), // 聚合的个数
          // fill: new Fill({ color: '#fff' }),
        })
      })
    }
  }
})
map.addLayer(clusterLayer)
```

![GIF 2025-09-08 21-02-03](C:./Typora图片/GIF 2025-09-08 21-02-03.gif)

#### 聚合+ol-ext实现聚合动画

仅仅把上面创建聚合图层的代码进行修改

原来代码

```js
// 创建一个聚合图层
const clusterLayer = new VectorLayer({
  source: clusterSource,
  style: function(features) { ... }
})
map.addLayer(clusterLayer)
```



改为动态聚合图层

1. 引入 AnimatedCluster
2. 用 AnimatedCluster 替换原来的 VectorLayer

```
可调参数:
    distance / minDistance: 控制聚合敏感度
    animationDuration: 动画时长
    animationMethod: 使用 ol/easing 的 easeOut / easeIn / linear 等
```

代码

```js
// 改为动态聚合图层
import AnimatedCluster from 'ol-ext/layer/AnimatedCluster.js'; // 引入动态聚合图层
import { easeOut } from 'ol/easing';
const clusterLayer = new AnimatedCluster({
  source: clusterSource,
  animationduration: 700, // 动画持续时间
  animationEasing: easeOut, // 动画缓动函数
  style: function(features) { ... }
})
map.addLayer(clusterLayer);
```

<img src="C:./Typora图片/GIF 2025-09-08 21-52-39.gif" alt="GIF 2025-09-08 21-52-39" style="zoom:80%;" />



### 路径动画

当动画结束时，点击地图再次生成一遍动画

```js
const points = [
  { name: "黄鹤楼", coords: [114.3162, 30.5278] },
  { name: "武汉博物馆", coords: [114.288, 30.547] },
  { name: "武汉大学", coords: [114.366, 30.538] },
  { name: "汉口里", coords: [114.294, 30.581] },
  { name: "武汉园博园", coords: [114.406, 30.485] },
  { name: "武汉长江大桥", coords: [114.284, 30.586] },
  { name: "江汉路步行街", coords: [114.287, 30.583] },
  { name: "武汉汉正街", coords: [114.273, 30.59] },
  { name: "东湖绿道", coords: [114.387, 30.532] },
  { name: "武汉科技馆", coords: [114.295, 30.55] },
  { name: "归元寺", coords: [114.304, 30.539] },
  { name: "武汉汉口北", coords: [114.296, 30.615] },
  { name: "武汉体育中心", coords: [114.323, 30.566] },
  { name: "武汉江滩", coords: [114.277, 30.546] },
  { name: "武汉植物园", coords: [114.395, 30.493] },
  { name: "武汉汉阳造", coords: [114.254, 30.541] },
  { name: "武汉儿童医院", coords: [114.313, 30.563] },
  { name: "黄鹤楼公园", coords: [114.354, 30.527] },
  { name: "武汉科学技术博物馆", coords: [114.298, 30.547] },
  { name: "武汉盛世华庭", coords: [114.36, 30.57] },
  { name: "武汉国际博览中心", coords: [114.359, 30.571] },
];





// 实现路径动画效果
import Feature from 'ol/Feature.js'
import { LineString } from 'ol/geom';
// 创建一个线要素
const lineFeature = new Feature({  
  geometry: new LineString(points.map(item => item.coords))
})

import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import { Stroke, Style } from 'ol/style.js';
// 创建一个矢量图层，显示这条线
const lineLayer = new VectorLayer({
  source: new VectorSource({
    features: [lineFeature]
  }),
  style: new Style({
    stroke: new Stroke({
      color: 'blue',
      width: 2,
    })
  })
})
map.addLayer(lineLayer)

// 在线段端点创建一个点元素，让其从线段一端运动到另一端
import Point from 'ol/geom/Point.js';
import Icon from 'ol/style/Icon.js';
const pointFeatures = new Feature({
  geometry: new Point(points[0].coords),
})
// 给pointFeatures设置样式
pointFeatures.setStyle(new Style({
  image: new Icon({
    src: '/地点.png',
    scale: 0.1,
  })
}))
lineLayer.getSource().addFeature(pointFeatures) // 将点要素添加到线图层上

// 监听地图点击事件 点击时，让该点从线段一端运动到另一端
import Path from 'ol-ext/featureanimation/Path.js'; // 引入路径动画函数
let animating = false; // 定义一个变量，表示是否正在执行动画

map.on('click', e => {
  console.log('点击了地图')
  if (animating) return // 如果正在执行动画，就返回
  animating = true // 设置为正在执行动画
    
    
   
  // 触发路径动画
  lineLayer.animateFeature(
    pointFeatures,
    new Path({
      path: lineFeature.getGeometry(), // 让点沿着这条线运动
      duration: 4000, // 动画持续时间 毫秒
      // repeat: true, // 是否重复
      // reverse: true, // 是否反向
      // easing: easeOut, // 动画缓动函数
      // speed: 1, // 速度 每毫秒移动多少个单位的地图坐标
    }),
  )
    
    
  
  lineLayer.on('animationend', e => { // 监听动画结束事件
    if (e.feature === pointFeatures) {
      animating = false;
      console.log('动画结束了');
    }
  });
})
```

实现机制

```js
1.animateFeature 方法的作用
    这是 ol-ext 扩展库中为 VectorLayer 添加的动画接口。
    作用是：在给定的图层上，对指定的要素（feature）应用某种动画效果。

2.实现路径动画的关键是调用了 lineLayer.animateFeature 方法

3.调用 lineLayer.animateFeature(pointFeatures, pathAnimation)：

4.Path 动画对象
Path 是 ol-ext/featureanimation/Path.js 中定义的动画类。
    它接收一个 path 参数（这里是线要素的几何 LineString），控制点要素沿这条路径运动。
    主要参数：
        path: 运动轨迹，这里是多段线的几何对象。
        duration: 动画持续时间（毫秒）。
        其它参数如 repeat（是否循环）、reverse（是否反向运动）、easing（缓动函数）等可用于控制动画效果。
```

实现效果

![GIF 2025-09-08 22-11-27](C:./Typora图片/GIF 2025-09-08 22-11-27.gif)





# mapbox

```
我感觉mapbox和openlayers很不一样，虽然同样是通过面向对象的方式写的
比如通过new mapboxgl.Map创建一个地图，有个container用来指定地图容器，projection，zoom指定投影和缩放等级，当projection: 'globe'时，又能增加三维视角，所以又有了方向角，比如bearing，但式没有像openlayers有专门的view类
mapbox的图层很奇怪，好像把图层或者数据源(说内容或许更加贴切)全部塞进了map对象的style属性里了，比如直接用网站给的url地址就能直接加载处一个融合了非常多图层的"style",同样的，通过setStyle可以设置地图的内容，通过getCenter获取中心点，不断调整中心点可以让地球自转，
mapbox加载第三方数据要做style属性里定义就很明显了，在style属性值的sources属性值里添加一个对象，对象属性之一就是数据源名称，数据源名称里又来个对象，要通过type定义类型，tiles:放栅格瓦片URL地址，然后style居然是在style中定义的，和source平级，不过source要放到layers中的source属性中才行


但是后面添加Geojson1数据的时候，又能直接在地图上通过addSource添加数据源，addLayer方法添加图层，前面添加的数据源同时放到addLayer图层对象参数的source属性中，图层对象又通过paint设置可见属性，通过layout设置布局属性，mapbox好像并不区分栅格或者矢量图层，好像是根据使用的数据源不同来区分，
比如type: 'raster'，使用的数据源是个图片type: 'image'， 
type: 'fill'type: 'line'填充和描边图层使用的是geojson数据，,，这里打点添加Icon还需要先在地图上通过loadImage方法把图片加载出来，然后layout中才能使用，也对应了专门的 type: 'symbol',符号图层，

数据源的类型也有好多，目前就看到 type: 'geojson'type: 'image',数据源里面还有个data属性，对应属性值对象中也有类型，目前就看见一个type: 'FeatureCollection', // 要素集合，能用getSource('pointMark').pointMarkSource._data方法获取data属性。能往里面的features属性数组中添加要素，这样就能在地图上显示很多要素，不过比openlayers麻烦多了，

而且mapbox要素设置很坑，只能给整个图层设置，必须通过setPaintProperty创建表达式，才能在交互的时候更改样式
感觉在mapbox里面，style和layer并没有什么严格的层级关系，当然，source还是在layer里面的
```























## 安装

```
pnpm install --save mapbox-gl

pnpm install @turf/turf
turf.js中转 点线面 通常对应的是 一维 二维 三维 数组

 没法给单个要素设置样式，只能给整个图层设置，可以通过表达式在交互的时候更改样式  Mapbox GL jS文档详见Styling layers with expressions
 
 
绘图插件
用于 Mapbox GL JS 地图的交互式绘图插件。
https://www.npmjs.com/package/@mapbox/mapbox-gl-draw
pnpm install @mapbox/mapbox-gl-draw

官网文档
https://docs.mapbox.com/mapbox-gl-js/example/mapbox-gl-draw/
pnpm install @mapbox/mapbox-gl-directions   //根据绘图控件猜的，npm没有给安装途径

导航插件
安装https://github.com/mapbox/mapbox-gl-directions
https://docs.mapbox.com/mapbox-gl-js/example/mapbox-gl-directions/



L7安装
pnpm install --save @antv/l7
https://l7.antv.antgroup.com/tutorial/quickstart
```

后来也找到了在node_modules文件夹中的这个位置

![image-20250912103736000](C:./Typora图片/image-20250912103736000.png)





## mapbox设计（style）

- OpenLayers：是一个“对象树”，`new Map({ view, layers, ... })`，每一层是一个 class。
- Mapbox GL：核心是**Style 规范（JSON）**，`mapboxgl.Map` 更多是在style运行时，存放它的容器。
- 比如`map.style` 下有 `sources` / `layers` / `sprite` / `glyphs` 等。



设计层面：

在 Mapbox GL 里，style 本质上就是一份符合 Mapbox Style Specification 的 JSON。

它就像一个“地图配置文件”，里面定义了 sources、layers、sprite、glyphs 等。

可以把这份 JSON 当成普通对象传给 `mapboxgl.Map` 的 `style` 参数，Mapbox 的渲染引擎会根据这份配置去创建数据源和图层并渲染出来。

所以 style 更像是 `package.json` 一类的配置，而不是一个类实例。



**Style 就是一份严格按规范写的 JSON 对象**，比如（极简版）：

```json
{
  "version": 8,
  "sources": {
    "china": {
      "type": "geojson",
      "data": "https://geo.datav.aliyun.com/areas/bound/100000_full.json"
    }
  },
  "layers": [
    {
      "id": "china-fill",
      "type": "fill",
      "source": "china",
      "paint": {
        "fill-color": "rgba(144, 164, 174, 0.3)"
      }
    }
  ]
}
```

这就是 **一个普通的 JS 对象 / JSON**，没有什么“类实例”的概念。



可以写成普通对象赋值给一个`mapboxgl.Map`的`style`属性：

```ts
const styleObj = {
  version: 8,
  sources: { ... },
  layers: [ ... ],
};

const map = new mapboxgl.Map({
  container: 'map',
  style: styleObj,          // 直接把对象丢进来
  center: [114.27, 30.59],
  zoom: 4,
});
```



也可以写成一个json字符串，通过`JSON.parse()`进行解析：

```ts
const styleJson = '{ "version": 8, "sources": {...}, "layers": [...] }';
const styleObj = JSON.parse(styleJson);

new mapboxgl.Map({ container: 'map', style: styleObj });
```



通常用的是官方给的url链接：

```ts
style: 'mapbox://styles/mapbox/streets-v12'
```

这是 **“Style URL”**，Mapbox 会从云端拉一份 JSON 回来，这个 JSON 其实长得就像上面的结构，只是更复杂。



也可以在外部写一份JSON数据，当成本地对象传进去：

```ts
import myStyle from './my-style.json';

new mapboxgl.Map({
  container: 'map',
  style: myStyle, // 就是一份 JSON 对象
});
```



## mapbox结构图

```text
mapboxgl.Map
  ├─ container / center / zoom / bearing / projection ...
  ├─ style (Style JSON + 运行时接口)
  │   ├─ sources  (数据源定义)
  │   ├─ layers   (展示这些数据源的画法)
  │   ├─ sprite / glyphs / light / fog / transition...
  ├─ events
  ├─ images (运行时注册图标)
  └─ controls (addControl 添加的控件)
```



1. 基础骨架：`mapboxgl.Map` + style 切换 + 视图控制
2. 数据 & 图层：GeoJSON / Raster / ImageSource + 各类 Layer（fill/line/circle/symbol/raster）
3. 事件 & 交互：点击要素高亮、Marker、Popup
4. 绘图 & 测量：Mapbox Draw + Turf 面积/长度
5. 常用特效：自转、flyTo/easeTo/rotateTo、根据表达式动态改样式



## `mapboxgl.Map` + style 切换 + 视图控制

### 概述

- **`mapboxgl.Map`**：地图实例，是一切的入口。
- **`style`**：一整套“地图配置文件”（底图、数据源、图层、样式）。
- **视图参数**：`center / zoom / bearing / pitch / projection` 控制地图“怎么被看到”。
- 运行时可以：
  - 用 `setStyle` 换整套底图；
  - 用 `flyTo/easeTo/...` 做视图动画。

### 最小初始化模板



main.ts / main.js

```ts
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = '你的 Mapbox Token';

const map = new mapboxgl.Map({
  container: 'map',                                // 绑定的 DOM 容器 id
  style: 'mapbox://styles/mapbox/streets-v12',     // 内置样式 URL 或自定义样式 URL
  center: [114.27, 30.59],                         // 初始中心 [lng, lat]
  zoom: 4,                                         // 初始缩放等级
  bearing: 0,                                      // 旋转角度（0 正北朝上）
  pitch: 0,                                        // 俯仰角（0=俯视，越大越倾斜）
  projection: 'mercator',                          // 'mercator' 或 'globe'
  attributionControl: false,                       // 关闭右下角 attribution
});
```

index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>Mapbox 基础示例</title>
    <style>
      body {
        margin: 0;
      }
      #map {
        width: 100vw;
        height: 100vh;
      }
    </style>
  </head>
  <body>
    <div id="map"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

> - Mapbox 的 `center` 永远是 `[lng, lat]`（经度在前）；
> - `style` 可以是 URL（云端配置）或一个 JSON 对象（本地配置）。

------

### style 

- 可以看成一份 **“地图配置 JSON”**：描述数据源（sources）、图层（layers）、样式（paint/layout）等。

- 既可以写成 JS 对象直接传入：

  ```ts
  const myStyle = {
    version: 8,
    sources: { ... },
    layers: [ ... ],
  };
  
  new mapboxgl.Map({ container: 'map', style: myStyle });
  ```

- 也可以用 Mapbox Studio 在网页上配置好，发布成一个 URL：

  ```ts
  style: 'mapbox://styles/yourname/xxxxxxx'
  ```

本质都是在喂给 `mapboxgl.Map` 一份 style 对象。

------

### 切换样式（自定义 + 内置样式）

给一个下拉框，切换不同底图（自定义 / streets / satellite 等）。

main.ts / main.js

```ts
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = '你的 Mapbox Token';

const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/chanter743/clygtwp1400yx01pre5vego7p', // 初始用你的自定义样式
  center: [114.27, 30.59],
  zoom: 2,
  projection: 'globe',
  bearing: 0,
});

// 下拉框切换地图样式
const styleSelector = document.querySelector('#style-selector') as HTMLSelectElement;

styleSelector.addEventListener('change', (e) => {
  const value = (e.target as HTMLSelectElement).value;
  map.setStyle(value); // 整套 style 替换
});

// 样式加载完成后再设置雾效（globe 下好看）
map.on('style.load', () => {
  map.setFog({});
});
```



index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>Mapbox 样式切换</title>
    <style>
      body {
        margin: 0;
      }
      #map {
        height: 100vh;
        width: 100vw;
      }
      #style-selector {
        position: absolute;
        top: 10px;
        left: 10px;
        padding: 5px;
        border: none;
        border-radius: 3px;
        z-index: 1;
        background-color: white;
      }
      #style-selector:focus {
        outline: none;
      }
    </style>
  </head>
  <body>
    <div id="map"></div>

    <select id="style-selector">
      <option value="mapbox://styles/chanter743/clygtwp1400yx01pre5vego7p">
        自定义样式
      </option>
      <option value="mapbox://styles/mapbox/standard">标准</option>
      <option value="mapbox://styles/mapbox/streets-v12">街道</option>
      <option value="mapbox://styles/mapbox/outdoors-v12">户外</option>
      <option value="mapbox://styles/mapbox/light-v11">明亮</option>
      <option value="mapbox://styles/mapbox/dark-v11">黑暗</option>
      <option value="mapbox://styles/mapbox/satellite-v9">卫星</option>
      <option value="mapbox://styles/mapbox/satellite-streets-v12">
        卫星街道
      </option>
      <option value="mapbox://styles/mapbox/navigation-day-v1">导航白天</option>
      <option value="mapbox://styles/mapbox/navigation-night-v1">导航夜间</option>
    </select>

    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

> - `map.setStyle(newStyleUrlOrObject)` 会重新加载整套样式；
> - 需要在 `style.load` 事件后再访问新样式的 layers/sources 或者设置 fog/light。

<img src="C:./Typora图片/image-20250911192949951.png" alt="image-20250911192949951" style="zoom: 33%;" />

------

### 视图控制：flyTo / easeTo / jumpTo / panTo / rotateTo

 flyTo：飞行效果（曲线 / 高度变化）

```ts
map.on('click', () => {
  map.flyTo({
    center: [112.93, 28.23], // 目标中心点
    zoom: 13,                // 目标缩放等级
    speed: 0.2,              // 0~1 越小越慢
    pitch: 80,               // 俯仰角 0~85
    // curve, easing 等也可以自定义
  });
});
```

特点：有类似飞机飞行的曲线感，适合“飞到某个城市”这类动效。

------

easeTo：平滑移动（直线感）

```ts
map.on('click', (evt) => {
  map.easeTo({
    center: evt.lngLat,    // 鼠标点击位置
    duration: 3000,        // 毫秒
    zoom: 10,
  });
});
```

特点：平滑、直观，不像 flyTo 那么“戏剧化”。

------

jumpTo：立即跳过去（无动画）

```ts
map.jumpTo({
  center: [120, 30],
  zoom: 8,
});
```

适合：程序初始化时/用户手动输入坐标时，直接跳转。

------

panTo：类似 easeTo 的平移（老 API 写法）

```ts
map.on('click', (evt) => {
  map.panTo(evt.lngLat, { duration: 2000 });
});
```

你知道有这个就行，新代码更推荐 `easeTo`。

------

rotateTo：旋转视角

```ts
map.on('click', () => {
  map.rotateTo(180, { duration: 2000 });  // 顺时针旋转到 180°
});
```

常搭配 `pitch` 产生“旋转看城市”的 3D 效果。

------

自转地球示例（巩固“如何持续更新视图中心”）

```ts
let rotate = true; // 是否自转

function rotateGlobe() {
  // 当正在缩放时先暂停
  if (map.isZooming()) {
    rotate = false;
  } else {
    rotate = true;
  }

  if (rotate) {
    const center = map.getCenter();
    center.lng -= 0.1;            // 每帧减小一点经度 → 地球向左转

    if (center.lng < -180) {
      center.lng = 180;           // 经度循环
    }

    map.setCenter(center);        // 更新地图中心
  }

  requestAnimationFrame(rotateGlobe); // 下一帧继续
}

// 启动自转
rotateGlobe();
```

> - `map.getCenter()` / `map.setCenter()` 的用法；
> - 如何用 `requestAnimationFrame` 做基于帧的视图更新。

<img src="C:./Typora图片/GIF 2025-09-11 19-31-53.gif" alt="GIF 2025-09-11 19-31-53" style="zoom:50%;" />

------

小结

1. 最小地图初始化模板：`new mapboxgl.Map({ container, style, center, zoom, projection })`。
2. style = 一整套地图配置 JSON，可以是 URL，也可以是 JS 对象。
3. `setStyle` 切换底图，`style.load` 事件后再操作新样式。
4. 视图动画 API：
   - flyTo → 炫酷飞行；
   - easeTo / panTo → 平滑移动；
   - jumpTo → 无动画跳转；
   - rotateTo → 旋转；
   - setMaxBounds → 限制视图范围。
5. 自转：`getCenter/setCenter` 和动画循环。



## 数据 & 图层：GeoJSON / Raster / ImageSource + 各类 Layer

### 概述

在 Mapbox 里，**数据与图层永远成对出现**：

- **Source（数据源）**：数据从哪来？（GeoJSON、Raster 瓦片、Image、矢量瓦片等）
- **Layer（图层）**：把这个数据“画成”什么？（fill 面、line 线、circle 点、symbol 图标+文字、raster 栅格…）



```ts
map.addSource('source-id', { ...sourceConfig });
map.addLayer({
  id: 'layer-id',
  type: 'fill' | 'line' | 'circle' | 'symbol' | 'raster',
  source: 'source-id',
  paint: { ... },
  layout: { ... },
  filter: [ ... ],
});
```

> - OL 的 `VectorSource` ≈ Mapbox 的 `geojson` source
> - OL 的 `VectorLayer` ≈ Mapbox 的 `fill/line/circle/symbol` layer

------

###  Raster 瓦片当底图（高德）

- **Source**: `type: 'raster'` + `tiles: ['http://xxx/{z}/{x}/{y}.png']`
- **Layer**: `type: 'raster'`

```ts
const gaodeMap = new mapboxgl.Map({
  container: 'map',
  style: {
    version: 8,
    glyphs: 'mapbox://fonts/mapbox/{fontstack}/{range}.pbf',
    sources: {
      gaode: {
        type: 'raster',
        tiles: [
          'http://webrd01.is.autonavi.com/appmaptile?' +
            'lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}',
        ],
        tileSize: 256,
        attribution: '高德地图',
      },
    },
    layers: [
      {
        id: 'gaode',
        type: 'raster',
        source: 'gaode',
      },
    ],
  },
  center: [114.27, 30.59],
  zoom: 8,
  projection: 'mercator',
});
```

> 第三方瓦片（XYZ/WMTS）塞进 Mapbox，当成“基底图层”

------

### GeoJSON 数据：填充 + 描边（中国边界）

- **Source**: `type: 'geojson'`, `data: url 或对象`
- Layer:
  - fill → 面填充
  - line → 边界线



```ts
gaodeMap.on('load', () => {
  // 1. 数据源：GeoJSON
  gaodeMap.addSource('china', {
    type: 'geojson',
    data: 'https://geo.datav.aliyun.com/areas/bound/100000_full.json',
  });

  // 2. 填充图层
  gaodeMap.addLayer({
    id: 'china-fill',
    type: 'fill',
    source: 'china',
    paint: {
      'fill-color': 'rgba(144, 164, 174, 0.3)',
    },
  });

  // 3. 描边图层
  gaodeMap.addLayer({
    id: 'china-outline',
    type: 'line',
    source: 'china',
    paint: {
      'line-color': 'rgb(30, 136, 229)',
      'line-width': 2,
    },
  });
});
```

> 加载任何 GeoJSON 面数据的通用模式：
> 省、市、街道边界；行政区、多边形区域等。

<img src="C:./Typora图片/image-20250911194820820.png" alt="image-20250911194820820" style="zoom:50%;" />

------

###  静态图片叠加（ImageSource + raster 图层）

- **Source**: `type: 'image'` + 四个角坐标；
- **Layer**: `type: 'raster'`。



```ts
gaodeMap.on('load', () => {
  gaodeMap.addSource('imageSource', {
    type: 'image',
    url: '/明代地图.jpg',
    coordinates: [
      [73.675379, 53.458804],   // 左上
      [135.026311, 53.458804],  // 右上
      [135.026311, 18.197700],  // 右下
      [73.675379, 18.197700],   // 左下
    ],
  });

  gaodeMap.addLayer({
    id: 'imageLayer',
    type: 'raster',
    source: 'imageSource',
  });
});
```

> 对应 OL 中的 ImageStatic。
> 适合叠加历史地图、专题图、切好的 PNG 之类。

<img src="C:./Typora图片/image-20250911195142331.png" alt="image-20250911195142331" style="zoom: 33%;" />

------

### 点击地图添加点 + 图标 + 文字（symbol 图层）

1. `addSource` 创建一个空的 GeoJSON FeatureCollection；
2. `loadImage + addImage` 注册图标；
3. `addLayer(type: 'symbol')` 使用 `icon-image + text-field`；
4. `map.on('click')` + Turf 生成 point Feature，更新 `source.setData()`。



数据源 + 图片 + symbol 图层

```ts
gaodeMap.on('load', () => {
  // 1. 空数据源存点
  gaodeMap.addSource('pointMark', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [],
    },
  });

  // 2. 加载图标 + 注册
  gaodeMap.loadImage('/位置.png', (err, img) => {
    if (err || !img) {
      console.error('图片加载失败', err);
      return;
    }
    if (!gaodeMap.hasImage('position-icon')) {
      gaodeMap.addImage('position-icon', img);
    }

    // 3. symbol 图层：图标 + 文字
    gaodeMap.addLayer({
      id: 'pointMarkLayer',
      type: 'symbol',
      source: 'pointMark',
      layout: {
        'icon-image': 'position-icon',
        'icon-size': 0.2,
        'icon-allow-overlap': true,

        // 文字标签来自属性 name
        'text-field': ['get', 'name'],
        'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
        'text-size': 12,
        'text-offset': [0, 1.5],
      },
    });
  });
});
```

urf 生成点 Feature + 更新数据源

```ts
import * as turf from '@turf/turf';

gaodeMap.on('click', (e) => {
  const lngLat = e.lngLat;

  // 1. 生成 Turf 点要素
  const pointFeature = turf.point(
    [lngLat.lng, lngLat.lat],
    { name: '点' + Math.random().toString().slice(2, 5) }
  );

  // 2. 更新数据源
  const source = gaodeMap.getSource('pointMark') as any;
  const data = source._data;          // 当前 FeatureCollection
  data.features.push(pointFeature);
  source.setData(data);               // 通知地图更新
});
```



“在 Mapbox 里没有单独的 Feature 对象，用 Turf 生成 GeoJSON 的 Feature，再通过 `setData` 更新整份 GeoJSON 数据。”

![GIF 2025-09-11 20-00-41](C:./Typora图片/GIF 2025-09-11 20-00-41.gif)

------

### 线：Turf lineString + line 图层

- Turf 提供便捷的 `lineString` 生成 GeoJSON；
- Mapbox 用 `line` 图层画出来。

```ts
import { lineString } from '@turf/turf';

gaodeMap.on('style.load', () => {
  const lineCoordinates = [
    [116.3912757, 39.9056454], // 北京
    [121.473701, 31.230416],   // 上海
    [114.305393, 30.593099],   // 武汉
  ];

  const lineFeature = lineString(lineCoordinates, { name: '连接线' });

  gaodeMap.addSource('lineSource', {
    type: 'geojson',
    data: lineFeature,
  });

  gaodeMap.addLayer({
    id: 'lineLayer',
    type: 'line',
    source: 'lineSource',
    paint: {
      'line-color': '#ff0000',
      'line-width': 5,
      'line-opacity': 0.8,
    },
    layout: {
      'line-join': 'round',
      'line-cap': 'round',
    },
  });
});
```

> 如果你之后做“路径播放/OD 连线”之类的，这就是基础模板。

<img src="C:./Typora图片/image-20250911200617625.png" alt="image-20250911200617625" style="zoom:50%;" />

------

### 面：Turf polygon + fill 图层

- 多边形坐标必须首尾闭合；
- `polygon([[ring1], [ring2?], ...])`；
- `fill` 图层画填充，`line` 图层画边界。



```ts
import { polygon } from '@turf/turf';

gaodeMap.on('style.load', () => {
  const triangleCoordinates = [
    [
      [116.3912757, 39.9056454], // 北京
      [121.473701, 31.230416],   // 上海
      [114.305393, 30.593099],   // 武汉
      [116.3912757, 39.9056454], // 闭合
    ],
  ];

  const triangleFeature = polygon(triangleCoordinates, { name: '三角形' });

  gaodeMap.addSource('triangleSource', {
    type: 'geojson',
    data: triangleFeature,
  });

  gaodeMap.addLayer({
    id: 'triangleLayer',
    type: 'fill',
    source: 'triangleSource',
    paint: {
      'fill-color': 'rgba(255, 0, 0, 0.5)',
      'fill-outline-color': '#ff0000',
      'fill-opacity': 0.8,
    },
  });
});
```

<img src="C:./Typora图片/image-20250911200838917.png" alt="image-20250911200838917" style="zoom:50%;" />

------

### 小结

表单

| Source 类型 | Layer 类型 | 效果                   | 示例场景               |
| ----------- | ---------- | ---------------------- | ---------------------- |
| `raster`    | `raster`   | 瓦片栅格底图           | 高德/天地图/卫星       |
| `image`     | `raster`   | 静态图片覆盖到地理范围 | 历史地图、专题图       |
| `geojson`   | `fill`     | 多边形填充             | 行政区、湖泊、区域高亮 |
| `geojson`   | `line`     | 折线/边界线            | 路网、边界线、轨迹     |
| `geojson`   | `circle`   | 圆点                   | 简单散点图             |
| `geojson`   | `symbol`   | 图标 + 文本            | POI 标注、点名称       |

> - raster + raster、image + raster；
> - geojson + fill / line / symbol；



## 事件 & 交互：点击要素高亮、Marker、Popup

### 概述

- Mapbox 所有常用交互基本都绕不开 3 类东西：
  1. 地图事件：`map.on('click', ...)`，`map.on('mousemove', ...)` 等。
  2. 图层要素事件：`map.on('click', layerId, handler)`，只在点击某个 layer 的要素时触发。
  3. UI 元素：
     - Marker：在某个点上画一个 DOM/默认图钉；
     - Popup：在某点弹出一个信息框。

------

### 点击图层要素高亮（`setPaintProperty` + 表达式）

- 使用阿里湖北 GeoJSON 作为 `source`；
- 使用 `fill` 图层画省内区域；
- 使用 `map.on('click', 'layer-id', handler)` 拿到被点击 Feature 的属性；
- 用 `setPaintProperty` + 表达式（`case` / `==` / `get`）让被点击区域变色。



```ts
gaodeMap.on('style.load', () => {
  // 1. 添加湖北省 GeoJSON 源
  gaodeMap.addSource('HubeiProvince', {
    type: 'geojson',
    data: 'https://geo.datav.aliyun.com/areas_v3/bound/420000_full.json',
  });

  // 2. 填充图层
  gaodeMap.addLayer({
    id: 'HubeiProvinceFillLayer',
    type: 'fill',
    source: 'HubeiProvince',
    paint: {
      'fill-color': 'rgb(236, 239, 241)',
      'fill-opacity': 0.5,
    },
  });

  // 3. 描边线图层（可选）
  gaodeMap.addLayer({
    id: 'HubeiProvinceLineLayer',
    type: 'line',
    source: 'HubeiProvince',
    paint: {
      'line-color': 'rgb(33, 150, 243)',
      'line-width': 1,
    },
  });

  // 4. 点击填充图层要素 → 高亮该要素
  gaodeMap.on('click', 'HubeiProvinceFillLayer', (e) => {
    const feature = e.features?.[0];
    if (!feature) return;

    const clickedName = feature.properties?.name;
    if (!clickedName) return;

    // 设置 paint 表达式：如果 name == 被点击的名称 → 橙色，否则灰色
    gaodeMap.setPaintProperty('HubeiProvinceFillLayer', 'fill-color', [
      'case',
      ['==', ['get', 'name'], clickedName], // 条件：当前要素的 name == 点击的 name
      'rgb(255, 87, 34)',                    // 条件为真 → 橙色
      'rgb(236, 239, 241)',                  // 否则 → 默认灰色
    ]);
  });
});
```

拿到的 `name` 就是 GeoJSON 里每个 Feature 的 `properties.name`

> - `map.on('click', 'layer-id', handler)`：只在点击该图层的要素时触发，并在 `e.features` 中给你命中的 Feature；
> - `setPaintProperty(layer, property, expression)`：用 Mapbox 表达式动态控制样式，真正体现数据驱动渲染。

<img src="C:./Typora图片/GIF 2025-09-11 20-32-21.gif" alt="GIF 2025-09-11 20-32-21" style="zoom:50%;" />

------

### Marker：点击地图添加可拖拽标记点

- Marker 是一个“地图上的 DOM/图钉容器”，不走 style/sources/layers 体系；
- 适合“交互较重、数量不太多”的标记（比如一个当前位置标记、若干个兴趣点编辑）。



```ts
function createMarker(lngLat: mapboxgl.LngLatLike) {
  const marker = new mapboxgl.Marker({
    color: 'red',
    draggable: true,       // 允许拖拽
  })
    .setLngLat(lngLat)
    .addTo(map);           // 添加到地图
}

// 在点击处添加 Marker
map.on('click', (e) => {
  const lngLat = e.lngLat;
  createMarker(lngLat);
});
```

> - Marker 属于“UI 控件层”，不需要 addSource/addLayer；
> - 如果你只是想在某个点画几个“少量交互很重”的点，用 Marker 更简单；
> - 如果是海量点（几百几千），还是应该用 GeoJSON + `circle/symbol` 图层。

------

### Popup：点击地图弹出信息框

- Popup 是在某个位置“挂一个 HTML 小卡片”；
- 默认点击地图会关闭现有 popup，可以用 `closeOnClick: false` 改行为；
- 常搭配 Marker 或 layer click 使用。



```ts
function createPopup(lngLat: mapboxgl.LngLatLike, html: string) {
  const popup = new mapboxgl.Popup({
    closeOnClick: false,    // 点击地图时不自动关闭
  })
    .setLngLat(lngLat)
    .setHTML(html)
    .addTo(map);
}

map.on('click', (e) => {
  const lngLat = e.lngLat;
  const html = `
    <h3>这里是你点击的位置</h3>
    <p>经度: ${lngLat.lng.toFixed(4)}<br>
       纬度: ${lngLat.lat.toFixed(4)}</p>
  `;
  createPopup(lngLat, html);
});
```

> - Popup 和 Marker 一样，是 UI 层的东西；
> - 适合用来显示“具体一条记录的详细信息”。

------

### 点击某要素 → Popup 显示属性信息（组合用法）



点击省/市/点要素 → 在该要素上弹出一个 Popup 显示属性。

假设你已经有一个 `china` GeoJSON + 填充图层



```ts
gaodeMap.on('load', () => {
  // ... 此处省略 addSource('china') + addLayer('china-fill') ...

  gaodeMap.on('click', 'china-fill', (e) => {
    const feature = e.features?.[0];
    if (!feature) return;

    const { name, adcode, level } = feature.properties || {};
    const coordinates = e.lngLat; // 或者根据 feature 中心点

    const html = `
      <h4>${name ?? '未知地区'}</h4>
      <p>adcode: ${adcode ?? '-'}<br>level: ${level ?? '-'}</p>
    `;

    new mapboxgl.Popup({ closeOnClick: true })
      .setLngLat(coordinates)
      .setHTML(html)
      .addTo(gaodeMap);
  });
});
```

> - 使用 `map.on('click', 'layer-id', ...)` 限定点击某图层要素；
> - `e.features[0].properties` 拿属性，`e.lngLat` 或要素几何中心当位置；
> - Popup 是瞬时创建的，用完由用户关闭。

------

### 修改官方底图的图层样式（setPaintProperty）

1. 使用官方样式 `mapbox://styles/mapbox/streets-v12`；
2. `map.getStyle().layers` 打印查看内部图层 id；
3. 用 `setPaintProperty` 修改某个图层的 paint 属性。



```ts
const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/mapbox/streets-v12',
  center: [114.27, 30.59],
  zoom: 2,
  projection: 'globe',
});

map.on('click', () => {
  const style = map.getStyle();
  console.log('当前 style:', style);
  console.log('所有图层:', style.layers);

  // 根据 layers 中的 id 修改样式
  map.setPaintProperty('water', 'fill-color', 'rgb(76, 175, 80)');      // 水体变绿
  map.setPaintProperty('land', 'background-color', 'rgb(33, 150, 243)'); // 陆地变蓝
  map.setPaintProperty('road-primary', 'line-color', '#0000ff');        // 主干道变蓝
});
```

> - 官方样式是“自定义样式”的基础，可以用来改色、隐藏图层；
> - `getStyle().layers` 是调试时的好帮手，快速找 layer id。

<img src="C:./Typora图片/image-20250911193659939.png" alt="image-20250911193659939" style="zoom:67%;" />

<img src="C:./Typora图片/image-20250911194048530.png" alt="image-20250911194048530" style="zoom: 50%;" />

------

### 小结

几种模式：

1. **点击某 layer 要素 → 高亮**
   - `map.on('click', 'layer-id', handler)` 拿到 `e.features[0]`；
   - `setPaintProperty` + 表达式（`case / get / ==`）做数据驱动样式。
2. **点击地图 → Marker**
   - `new mapboxgl.Marker({ draggable }).setLngLat(lngLat).addTo(map)`。
3. **点击地图/要素 → Popup**
   - `new mapboxgl.Popup().setLngLat(...).setHTML(...).addTo(map)`。
4. **修改内置底图样式**
   - `getStyle().layers` 看 layer id；
   - `setPaintProperty(id, paintName, valueOrExpression)` 改颜色/宽度/透明度等。



## 绘图 & 测量：Mapbox Draw + Turf 面积/长度

### 概述

- **Mapbox Draw**：给 Mapbox 地图加一个“绘图工具条”（点、线、面），用户可以交互式画几何。
- **Turf.js**：对 GeoJSON 做空间计算（长度、面积、缓冲区等）。
- 典型流程：
  1. 把 Mapbox Draw 控件加到地图上；
  2. 监听 `draw.create / draw.update / draw.delete` 等事件；
  3. 用 `draw.getAll()` 拿到用户画出来的 GeoJSON；
  4. 把这个 GeoJSON 丢给 Turf：`turf.area` / `turf.length` 等完成测量。

------

### 安装 & 引入



```bash
pnpm install @mapbox/mapbox-gl-draw
pnpm install @turf/turf
```



在代码里：

```ts
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

import MapboxDraw from '@mapbox/mapbox-gl-draw';
import '@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css';

import * as turf from '@turf/turf';
```



------

### 绘制多边形 + 计算面积（你已经实现的典型用法）

- 在地图右上角加一个绘图工具条；
- 用户画多边形；
- 下方弹出框显示该多边形的面积（平方米）。

#### main.ts / main.js



```ts
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

import MapboxDraw from '@mapbox/mapbox-gl-draw';
import '@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css';

import * as turf from '@turf/turf';

mapboxgl.accessToken = '你的 Mapbox Token';

const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/mapbox/streets-v12',
  center: [114.27, 30.59],
  zoom: 4,
});

// 1. 初始化绘图控件：只开启 polygon 和 删除
const draw = new MapboxDraw({
  displayControlsDefault: false, // 不显示默认所有按钮
  controls: {
    polygon: true,
    trash: true,
  },
  // defaultMode: 'draw_polygon', // 如需默认进入画多边形模式
});

// 2. 把控件加到地图上
map.addControl(draw, 'top-right');

// 3. 注册事件：创建 / 更新 / 删除 时都重新计算面积
map.on('draw.create', updateArea);
map.on('draw.update', updateArea);
map.on('draw.delete', updateArea);

function updateArea() {
  const data = draw.getAll(); // GeoJSON FeatureCollection
  const box = document.querySelector('.calculation-box') as HTMLElement;
  if (!box) return;

  if (data.features.length > 0) {
    // 这里直接对整个 FeatureCollection 计算面积（多个多边形也能算）
    const area = turf.area(data);              // 返回值单位：平方米
    const rounded = Math.round(area * 100) / 100;
    box.innerHTML = `<p><strong>面积: </strong>${rounded} m²</p>`;
  } else {
    box.innerHTML = '<p>请绘制一个多边形</p>';
  }
}
```



#### index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>Mapbox Draw 面积测量</title>
    <style>
      body {
        margin: 0;
      }
      #map {
        width: 100vw;
        height: 100vh;
      }
      .calculation-box {
        position: absolute;
        left: 10px;
        bottom: 10px;
        width: 200px;
        background: white;
        padding: 10px;
        border-radius: 5px;
        font-size: 12px;
        box-shadow: 0 1px 4px rgba(0, 0, 0, 0.3);
      }
    </style>
  </head>
  <body>
    <div id="map"></div>
    <div class="calculation-box">请绘制一个多边形</div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

> - `draw.getAll()` 拿到的是标准的 GeoJSON FeatureCollection；
> - `turf.area(geojson)` 直接对整个 FeatureCollection 计算面积，单位是 m²。

<img src="C:./Typora图片/GIF 2025-09-12 15-25-36.gif" alt="GIF 2025-09-12 15-25-36" style="zoom:67%;" />

------

### 使用 Draw 绘制线条 + 计算长度（直线测距）

- 工具条里有“画线”和“删除”按钮；
- 用户画一条或多条线；
- 底部显示总长度（千米）。



```ts
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

import MapboxDraw from '@mapbox/mapbox-gl-draw';
import '@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css';

import * as turf from '@turf/turf';

mapboxgl.accessToken = '你的 Mapbox Token';

const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/mapbox/streets-v12',
  center: [114.27, 30.59],
  zoom: 4,
});

// 1. 开启 line_string 控件
const draw = new MapboxDraw({
  displayControlsDefault: false,
  controls: {
    line_string: true,
    trash: true,
  },
});

map.addControl(draw, 'top-right');

// 2. 监听创建 / 更新 / 删除线段事件
map.on('draw.create', updateLength);
map.on('draw.update', updateLength);
map.on('draw.delete', updateLength);

function updateLength() {
  const data = draw.getAll();
  const box = document.querySelector('.calculation-box') as HTMLElement;
  if (!box) return;

  if (data.features.length > 0) {
    // 可选：只计算 lineString 几何
    const lineFeatures = data.features.filter(
      (f) => f.geometry && f.geometry.type === 'LineString'
    );

    let total = 0;
    lineFeatures.forEach((f) => {
      // Turf.length 默认单位是千米
      total += turf.length(f, { units: 'kilometers' });
    });

    const rounded = Math.round(total * 100) / 100;
    box.innerHTML = `<p><strong>总长度: </strong>${rounded} km</p>`;
  } else {
    box.innerHTML = '<p>请绘制一条或多条线段</p>';
  }
}
```

> - `turf.length(geojson, { units: 'kilometers' })` 默认就是算大圆距离，适合测距；
> - 若想单位是 m，可用 `{ units: 'meters' }`。

------

### 同一个 Draw 控件既能画线又能画面

这样配置：

```ts
const draw = new MapboxDraw({
  displayControlsDefault: false,
  controls: {
    polygon: true,
    line_string: true,
    trash: true,
  },
});
```

然后在 refresh 时根据 geometry type 区分处理，比如：

在下面这三个时刻调用下面这个函数：

- `draw.create`：用户新画完一个几何；
- `draw.update`：用户拖动/修改了一个几何；
- `draw.delete`：用户删掉了某个几何。

```ts
function updateMeasure() {
  const data = draw.getAll();
  const box = document.querySelector('.calculation-box') as HTMLElement;
  if (!box) return;

  let totalArea = 0;
  let totalLength = 0;

  data.features.forEach((f) => {
    if (!f.geometry) return;

    if (f.geometry.type === 'Polygon' || f.geometry.type === 'MultiPolygon') {
      totalArea += turf.area(f); // m²
    }

    if (f.geometry.type === 'LineString' || f.geometry.type === 'MultiLineString') {
      totalLength += turf.length(f, { units: 'kilometers' }); // km
    }
  });

  box.innerHTML = `
    <p><strong>总面积: </strong>${(totalArea / 1e6).toFixed(2)} km²</p>
    <p><strong>总长度: </strong>${totalLength.toFixed(2)} km</p>
  `;
}
```

> 这段可以作为“高级用法”：一个面板同时统计当前所有绘制图形的长度 + 面积。

------

### 和“手写 GeoJSON + Turf” 的对比

不靠 Draw，而是维护一个 GeoJSON：

- 点击地图 → 用 Turf `point()` 把坐标变成 Feature；
- 存入 FeatureCollection；
- `source.setData(...)` 让地图更新；
- 对这份 FeatureCollection 用 Turf 做计算。

Mapbox Draw ：

1. **画图 UI**：画线、拖动、编辑、删除，不用自己写交互。
2. **数据结构**：内部用户画的东西维护成一份 GeoJSON FeatureCollection。
3. **事件**：用 `draw.create` / `draw.update` / `draw.delete` 通知“数据变了”。

只需要做：

- `draw.getAll()` 拿到数据；
- 从中挑你感兴趣的几何类型；
- 扔给 Turf 计算长度/面积/缓冲等。

------

### 小结

1. Mapbox Draw 是什么
   - 控件层，帮你在地图上画点/线/面。
   - 不负责计算，只负责“让你有 GeoJSON”。
2. 学会用 `draw.getAll()` + Turf 做计算
   - 面积：`turf.area(geojson)` → m²；
   - 长度：`turf.length(geojson, { units: 'kilometers' })` → km。
3. 两个完整模板
   - “画多边形 + 面积显示”模板；
   - “画折线 + 总长度显示”模板。

“用户交互 → Draw 控件 → GeoJSON → Turf 计算 → 更新 UI”



## 插件plugins

#### 绘图插件

```
官方演示地址：https://docs.mapbox.com/mapbox-gl-js/example/mapbox-gl-draw/

https://www.npmjs.com/package/@mapbox/mapbox-gl-draw
pnpm install @mapbox/mapbox-gl-draw
```

直接使用

```
const draw = new MapboxDraw() // 初始化绘图控件
map.addControl(draw,'top-left') // 添加绘图控件
```



绘制多边形并显示面积

```js
import MapboxDraw from '@mapbox/mapbox-gl-draw';
import '@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css'; // 引入绘图控件样式 不然控件上的图标显示不出来

import * as turf from '@turf/turf'; // 引入turf.js 用于计算面积
// 使用mapbox-gl-draw 绘制多边形，并在绘制完成后使用Turf.js计算面积
const draw = new MapboxDraw({
  displayControlsDefault: false, // 不显示默认的绘图控件
  controls: { // 只显示绘制多边形和删除功能
    polygon: true,
    trash: true
  },
  // defaultMode: 'draw_polygon' // 默认进入绘制多边形模式
})
map.addControl(draw, 'top-right'); // 添加绘图控件
map.on('draw.create', updateArea); // 监听绘制完成事件
map.on('draw.delete', updateArea); // 监听删除事件
map.on('draw.update', updateArea); // 监听更新事件
function updateArea(e) {
  const data = draw.getAll() // 获取所有绘制的要素
  const answer = document.querySelector('.calculation-box'); // 获取显示面积的容器
  if (data.features.length > 0) {
    const area = turf.area(data)
    const rounded_area = Math.round(area * 100) / 100 // 四舍五入保留两位小数
    answer.innerHTML = `<p><strong>面积:</strong> ${rounded_area} 平方米</p>`;
  } else {
    answer.innerHTML = '';
  }
}
```

html

```html
  <style>
    body{
      margin: 0;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
    .calculation-box {
      position: absolute;
      left: 10px;
      bottom: 10px;
      width: 200px;
      background: white;
      padding: 10px;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <div class="calculation-box"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

<img src="C:./Typora图片/GIF 2025-09-12 15-25-36.gif" alt="GIF 2025-09-12 15-25-36" style="zoom:67%;" />



#### 导航插件

安装

```
导航插件
安装https://github.com/mapbox/mapbox-gl-directions
pnpm install @mapbox/mapbox-gl-directions   //根据绘图控件猜的，npm没有给安装途径
演示地址：https://docs.mapbox.com/mapbox-gl-js/example/mapbox-gl-directions/
```

main.js

```js
// 通过导航插件添加导航功能
import NavigationControl from '@mapbox/mapbox-gl-directions/dist/mapbox-gl-directions.js'; // 引入导航控件
import '@mapbox/mapbox-gl-directions/dist/mapbox-gl-directions.css'; // 引入导航控件样式
map.addControl(
  new NavigationControl({
    accessToken: mapboxgl.accessToken
  }),
  'top-left'
)
// 测试地点 San Francisco to Los Angeles
```

<img src="C:./Typora图片/GIF 2025-09-12 15-26-54.gif" alt="GIF 2025-09-12 15-26-54" style="zoom:67%;" />



# AntvG2

```
安装
pnpm install @antv/g2
```



## AntV G2 API 常用

创建chart实例

```
new Chart(params?: ChartCfg): View
```

- 创建一个 Chart 图表对象。
- `params` 为可选配置，类型为 `ChartCfg`。

常用ChartCfg 配置项

| 属性      | 类型                   | 默认值       | 说明                                                         |
| --------- | ---------------------- | ------------ | ------------------------------------------------------------ |
| container | `string                | HTMLElement` | 必填                                                         |
| width     | `number`               | 640          | 图表宽度                                                     |
| height    | `number`               | 480          | 图表高度                                                     |
| autoFit   | `boolean`              | false        | 是否自动适应容器宽高，true 时自动取容器宽高；用户设置的 height 优先 |
| clip      | `boolean`              | false        | 是否裁剪超出绘制区域的图形，true 时超出部分被隐藏            |
| inset     | `number`               | 0            | 图表的呼吸范围宽度，影响裁剪区域大小                         |
| padding   | `'auto'                | number`      | 'auto'                                                       |
| margin    | `number`               | 16           | 图表外边距，类似 CSS 盒模型                                  |
| depth     | `number`               | 0            | 图表深度，主要用于 3D 图表                                   |
| renderer  | Canvas \| SVG \| WebGL | Canvas 默认  | 指定渲染引擎，默认 Canvas，使用 SVG 或 WebGL 需额外引入对应包 |
| theme     | string \| object       | -            | 主题，可使用内置主题如 'classic'，也支持注册自定义主题       |





## 图表布局

### 布局概念

| 名称         | 说明                             |
| ------------ | -------------------------------- |
| View Area    | 视图区域，包含绘制区域和外边距   |
| Plot Area    | 绘制区域，绘制图形的主要区域     |
| Main Area    | 主区域                           |
| Content Area | 内容区域，主区域内的可视内容部分 |

![Snipaste_2025-09-15_10-02-57](C:./Typora图片/Snipaste_2025-09-15_10-02-57.png)

### 布局配置项

```js
View Area（width × height + margin）
 └── Main Area（内边距 padding）
      └── Content Area（呼吸范围 inset）
           └── Plot Area（绘制图形区域）

width 和 height 是 View Area 的尺寸。
Content Area 的宽高 = View Area 宽高 - 外边距 - 内边距 - 呼吸范围 等多层减去的空间。
```

| 配置项名 | 说明                                                         | 与视图区域关系                                               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| padding  | 图表内边距，包裹绘制区域（Plot Area），类似 CSS 盒模型的内边距 | 控制**Plot Area（绘制区域）\**与\**Main Area**之间的间距，内边距 |
| margin   | 图表外边距，包裹整个图表（View Area外侧）                    | 控制**View Area（视图区域）**的外边距                        |
| inset    | 图表呼吸范围，绘制图形时的边界缓冲                           | 控制**Content Area（内容区域）**与绘制图形边缘之间的缓冲区   |
| width    | 图表宽度                                                     | 控制**View Area**的宽度                                      |
| height   | 图表高度                                                     | 控制**View Area**的高度                                      |
| autoFit  | 是否自适应容器宽高                                           | 控制**View Area**是否随容器大小自动调整                      |

配置项使用示例

```js
import { Chart } from '@antv/g2';

// 创建图表实例
const chart = new Chart({
  container: 'container', // DOM id
  width: 700,             // View Area 宽度
  height: 500,            // View Area 高度
  margin: 20,             // View Area 外边距
  padding: 40,            // 内边距，包裹绘制区域
  inset: 10,              // 呼吸范围缓冲区
  autoFit: false,         // 不自适应容器大小
});

chart.data([
  { genre: 'Sports', sold: 275 },
  { genre: 'Strategy', sold: 115 },
  { genre: 'Action', sold: 120 },
  { genre: 'Shooter', sold: 350 },
  { genre: 'Other', sold: 150 },
]);

// 创建柱状图 Geometry
chart.interval().position('genre*sold').color('genre');

chart.render();

```





## 坐标轴（Axis）

[坐标轴（Axis） | G2 一套简明和渐进式的可视化语法](https://g2.antv.antgroup.com/manual/component/axis#label)

### 坐标轴构成

![Snipaste_2025-09-16_10-25-30](C:./Typora图片/Snipaste_2025-09-16_10-25-30.png)

![Snipaste_2025-09-16_10-25-35](C:./Typora图片/Snipaste_2025-09-16_10-25-35.png)



### 配置项

每个坐标轴由 标题（title）、轴线（line）、刻度（tick）、刻度值（label）以及网格线（grid）组成。

| 属性     | 描述                           | 类型                                                         | 默认值                  | 必选 |
| :------- | :----------------------------- | :----------------------------------------------------------- | :---------------------- | :--- |
| title    | 设置坐标轴的标题文本及其样式   | [title](https://g2.antv.antgroup.com/manual/component/axis#title) | -                       |      |
| line     | 设置坐标轴线的显示及其样式     | [line](https://g2.antv.antgroup.com/manual/component/axis#line) | -                       |      |
| tick     | 设置坐标轴刻度线的显示及其样式 | [tick](https://g2.antv.antgroup.com/manual/component/axis#tick) | -                       |      |
| label    | 设置坐标轴刻度值的显示及其样式 | [label](https://g2.antv.antgroup.com/manual/component/axis#label) | -                       |      |
| grid     | 设置坐标轴网格线的显示及其样式 | [grid](https://g2.antv.antgroup.com/manual/component/axis#grid) | -                       |      |
| animate  | 设置坐标轴动画效果             | `boolean` \| [animate](https://g2.antv.antgroup.com/manual/component/axis#animate) | -                       |      |
| position | 设置坐标轴的位置               | `left` | `right` | `top` | `bottom`                          | `x: bottom` | `y: left` |      |

每个配置项具体方法可以去网页查找







### title

#### 声明式配置	

适合配置驱动、一次性定义图表属性	

配置集中，适合自动化生成或封装复用，便于维护和理解

```js
const chart = new Chart({
  container: 'container',
  width: 600,
  height: 400,
  axis: {
    y: {
      title: 'Frequency',
      titleSpacing: 30,
      // 其他轴标题配置...
    },
    x: {
      title: 'Letter',
      titleSpacing: 20,
      // 其他轴标题配置...
    },
  },
});
```

配置项含义

```js
const options = {
  // 配置坐标轴
  axis: {
    // 配置 y 轴
    y: {
      // 这部分是轴标题的配置
      title: 'Frequency', // 设置 y 轴标题
      titleSpacing: 30, // 设置 y 轴标题与轴线之间的间距
      titlePosition: 'left', // 设置 y 轴标题位置
      titleFill: 'steelblue', // 设置 y 轴标题的颜色
      titleFontSize: 16, // 设置 y 轴标题字体大小
      titleFontWeight: 'bold', // 设置 y 轴标题字体粗细
      titleFontFamily: 'Arial', // 设置 y 轴标题字体
      titleTextAlign: 'center', // 设置 y 轴标题水平对齐方式
      titleTextBaseline: 'middle', // 设置 y 轴标题垂直基线
      titleOpacity: 0.9, // 设置 y 轴标题整体透明度
      titleStroke: '#333', // 设置 y 轴标题描边颜色
      titleLineWidth: 1, // 设置 y 轴标题描边宽度
      titleShadowColor: 'rgba(0,0,0,0.3)', // 设置 y 轴标题阴影颜色
      titleShadowBlur: 3, // 设置 y 轴标题阴影模糊度
      titleShadowOffsetX: 2, // 设置 y 轴标题阴影水平偏移
      titleShadowOffsetY: 2, // 设置 y 轴标题阴影垂直偏移
      titleDx: 5, // 设置 y 轴标题水平偏移量
      titleDy: 0, // 设置 y 轴标题垂直偏移量
      titleCursor: 'pointer', // 设置 y 轴标题鼠标样式
    },
    // 配置 x 轴
    x: {
      // 这部分是轴标题的配置
      title: 'Letter', // 设置 x 轴标题
      titleSpacing: 20, // 设置 x 轴标题与轴线之间的间距
      titlePosition: 'bottom', // 设置 x 轴标题位置
      titleFontSize: 14, // 设置 x 轴标题字体大小
      titleFontWeight: 'normal', // 设置 x 轴标题字体粗细
      titleFill: '#666', // 设置 x 轴标题颜色
      titleTextAlign: 'center', // 设置 x 轴标题水平对齐方式
      titleOpacity: 1, // 设置 x 轴标题透明度
      titleLineHeight: 1.2, // 设置 x 轴标题行高
      titleFillOpacity: 0.8, // 设置 x 轴标题填充透明度
    },
  },
};


const chart = new Chart({
  container: 'container',
  width: 600,
  height: 400,
  ...options, // 这里把 axis 直接展开进去
})

chart.render();
```



#### 命令式API调用

Imperative API

灵活，可动态改动，业务逻辑控制方便	

配置分散，逻辑和样式混杂，代码量可能更多

```js
const chart = new Chart({
  container: 'container',
  width: 600,
  height: 400,
});

chart.data([
  { genre: 'Sports', sold: 275 },
  { genre: 'Strategy', sold: 115 },
  // ...
]);

chart.interval().position('genre*sold');

// 通过 API 分别调用配置坐标轴
chart.axis('genre', {
  title: {
    text: '游戏类型',
    spacing: 15,
    style: {
      fontSize: 14,
      fontWeight: 'bold',
      fill: '#333',
      textAlign: 'center',
    },
    position: 'center',
    offset: 20,
  },
});

chart.axis('sold', {
  title: {
    text: '销量',
    style: {
      fontSize: 14,
      fontWeight: 'bold',
      fill: '#333',
    },
    position: 'center',
  },
});

chart.render();
```



### Legend

作用：图例是图表的辅助元素，主要用颜色、大小、形状和透明度四种非空间视觉通道（color、opacity、size、shape）区分数据类型，并支持数据筛选。

G2 根据数据字段与视觉通道的映射关系，自动生成相应的图例

| 通道    | 说明                   |
| ------- | ---------------------- |
| color   | 根据不同颜色生成图例   |
| opacity | 根据不同透明度生成图例 |
| size    | 根据不同大小生成图例   |
| shape   | 根据不同形状生成图例   |



#### 图例显示

隐藏所有图例

```js
const chart = new G2.Chart({
  container: 'container',
  width: 600,
  height: 400,
  legend: false, // 隐藏所有图例
});
```

只隐藏 color 通道的图例

```js
const chart = new G2.Chart({
  container: 'container',
  width: 600,
  height: 400,
  legend: {
    color: false,  // 隐藏 color 通道图例
  },
});
```

通过对象传入图例配置（legendOption）

```js
jsconst chart = new G2.Chart({
  container: 'container',
  width: 600,
  height: 400,
  legend: {
    color: {}, // 显示 color 通道图例，默认配置
    size: {},  // 显示 size 通道图例，默认配置
  },
});
```



#### 方向 位置 布局

在线布局示例

[图例（Legend） | G2 一套简明和渐进式的可视化语法](https://g2.antv.antgroup.com/manual/component/legend)

图例方向（orientation）

- `horizontal`（默认）或 `vertical`

```js
legend: {
  color: {
    orientation: 'vertical', // 设置图例为垂直方向
  },
},
```

------

图例位置（position）

- 可选值 `"top"` | `"right"` | `"bottom"` | `"left"`
- 默认值 `"top"`

```js
jslegend: {
  color: {
    position: 'bottom', // 图例显示在底部
  },
},
```

------

图例布局配置（layout）

- 通过 `layout` 属性细化对齐方式
- 支持的属性：`justifyContent`, `alignItems`

```js
jslegend: {
  color: {
    position: 'top',
    layout: {
      justifyContent: 'center',  // 主轴（水平方向）居中对齐
      alignItems: 'center',      // 交叉轴（垂直方向）居中对齐
    },
  },
},
```



## 比例尺（Scale）

### 柱状图带宽band

[band | G2 一套简明和渐进式的可视化语法](https://g2.antv.antgroup.com/manual/core/scale/band#自定义柱宽)



## 图形类别

### 面积图

[area | G2 一套简明和渐进式的可视化语法](https://g2.antv.antgroup.com/manual/core/mark/area#shape)



## 编码（encode)

[v4 升级 v5 指南 | G2 一套简明和渐进式的可视化语法](https://g2.antv.antgroup.com/manual/whats-new/migration-from-g2v4#编码encode)

## 代码片段参考

### 基础图表示例

```js
tsimport { Chart } from '@antv/g2';

const chart = new Chart({
  container: 'container',
  width: 640,
  height: 480,
});

chart
  .data([
    { year: '1991', value: 3 },
    { year: '1992', value: 4 },
    { year: '1993', value: 3.5 },
    // ...
  ])
  .encode('x', 'year')
  .encode('y', 'value')
  .scale('x', { range: [0, 1] })
  .scale('y', { domainMin: 6, nice: true });

chart.line().label({
  text: 'value',
  style: { dx: -10, dy: -12 },
});

chart.point().style('fill', 'white').tooltip(false);

chart.render();
```

### 开启裁剪 (clip)

```js
tsconst chart = new Chart({
  container: 'container',
  clip: true,
  inset: 20, // 调整呼吸范围，防止裁剪过严
});
```



### 主题定制

```js
import { Light, register, Chart } from '@antv/g2';

// 自定义主题
function CustomTheme() {
  const light = Light();
  return {
    ...light,
    category20: [
      '#FFC0CB', '#A2F5E8', '#D4B0FF', '#FFF3A3', // ...
    ],
  };
}

// 注册主题
register('theme.custom', CustomTheme);

const chart = new Chart({
  container: 'container',
  theme: { type: 'custom' },
});

chart.options({
  type: 'interval',
  data: { type: 'fetch', value: 'url_to_data.csv' },
  encode: { x: 'letter', y: 'frequency', color: 'letter' },
  axis: { y: { labelFormatter: '.0%' } },
});
chart.render();
```



### 渲染器配置

- 默认内置 Canvas 渲染器。
- 如需使用 SVG 或 WebGL，需要引入对应的渲染器包：

```js
import { Chart } from '@antv/g2';
import { Renderer as SVGRenderer } from '@antv/g-svg';
import { Renderer as WebGLRenderer } from '@antv/g-webgl';

const chart = new Chart({
  container: 'container',
  renderer: new SVGRenderer(), // 或 new WebGLRenderer()
});
```



待总结笔记

```
你这段代码里，legend 配置写在了 interval() 图形实例的链式调用里，但 G2 目前的图例配置是作用于整个图表（chart.legend()），而不是单个图形（interval.legend()）。

因此，interval().legend({...}) 不会生效，需要改为：

js
chart.legend('color', {
  position: 'right',
  layout: 'vertical', // 或 orientation: 'vertical' 视版本而定
});


链式调用，作用chart和interval()区别




```



### 设置不同宽度的柱子







## 待整理笔记

链接： [数据标签（Label） | G2 一套简明和渐进式的可视化语法](https://g2.antv.antgroup.com/manual/component/label#position)

![image-20251217110457757](C:./Typora图片/image-20251217110457757.png)











































# AntvL7

```
map.off  移除地图事件监听

通过L7 sence异步切换mapbox地图样式时，判断样式是否加载完成：

// 如果地图样式未加载完成，则不执行后续代码
if (typeof map.value.isStyleLoaded === 'function' && !map.value.isStyleLoaded()) return
前一个条件用于确定方法是否存在，后面确定是否加载完成

L7教程：https://l7.antv.antgroup.com/tutorial/develop
城市建筑图层：https://l7.antv.antgroup.com/api/other/city_building
路径图层：https://l7.antv.antgroup.com/tutorial/line/path
符号图：https://l7.antv.antgroup.com/tutorial/point/symbol

API文档：https://l7.antv.antgroup.com/api/scene



L7绘制功能
文档：https://l7draw.antv.vision/docs/draw/point
安装：pnpm install -S @antv/l7-draw



L7图层监听事件  以click为例
https://l7.antv.antgroup.com/api/base_layer/base#click
layer.on('dblclick', (e) => console.log(e));
e中包含图层中点击要素的各种信息，详细示例如下,可以通过解构赋值获取



L7信息框
https://l7.antv.antgroup.com/api/component/popup
import { Scene, Popup } from '@antv/l7';
```



e中图层要素信息

```json
{
    "x": 518,
    "y": 194,
    "type": "click",
    "lngLat": {
        "lng": 114.25148936477905,
        "lat": 30.531874256553778
    },
    "featureId": 2,
    "feature": {
        "type": "Feature",
        "properties": {
            "name": "杨泗港长江大桥",
            "info": "武汉长江大桥（Wuhan Yangtze River Bridge），是中国湖北省武汉市境内连接汉阳区与武昌区的过江通道，位于长江水道之上，是中华人民共和国成立后修建的第一座公铁两用的长江大桥，也是武汉市重要的历史标志性建筑之一，素有“万里长江第一桥”美誉。 [1]武汉长江大桥于1955年9月1日动工兴建；于1957年7月1日完成主桥合龙工程；于1957年10月15日通车运营。 [2]武汉长江大桥西起楚琴立交，上跨长江水道，东至中山路；线路全长1670.4米，主桥全长1155.5米；上层桥面为双向四车道城市主干道，设计速度100千米/小时，下层为双线铁轨，设计速度160千米/小时；总投资额为1.38亿人民币。"
        },
        "geometry": {
            "type": "LineString",
            "coordinates": [
                [
                    114.24937652148543,
                    30.515165749100262
                ],
                [
                    114.2654836706024,
                    30.503896031541657
                ]
            ]
        }
    },
    "target": {
        "isTrusted": true
    }
}
```





地图控件位置

![image-20250915204505995](C:./Typora图片/image-20250915204505995.png)



popup

```js
import { Scene, Popup } from '@antv/l7';
const scene = new Scene({...})
const popup = new Popup({})
scene.addPopup(popup)
// 设置弹窗内容
popup.setLngLat(lngLat)
popup.setHTML(`<p style="font-size: 12px;">${info}</p>`)
popup.setTitle(name)
```





















# truf.js

```
pnpm install @turf/turf






面和面计算交集 turf.intersect(poly1, poly2);
https://fenxianglu.cn/turfjs/category/transformation/intersect.html

点面求交 --> 判断点是否在多边形内
https://fenxianglu.cn/turfjs/category/booleans/booleanPointInPolygon.html

返回在多边形内的点 pointsWithinPolygon
https://fenxianglu.cn/turfjs/category/joins/pointsWithinPolygon.html
```



```
turf.js中转 点线面 通常对应的是 一维 二维 三维 数组

turf.point([lon, lat]) → 传 1 层坐标
turf.lineString([[lon, lat], ...]) → 2 层
turf.polygon([ ring1, ring2?, ring3? ]) → 3 层（每个 ring 是点数组）,ring举例：[[lon, lat], [lon, lat], [lon, lat], [lon, lat]]


turf.polygon 函数的作用是用来创建一个 GeoJSON 格式的多边形 (Polygon) 对象。
var polygon = turf.polygon([
  [
    [lng1, lat1],
    [lng2, lat2],
    [lng3, lat3],
    [lng1, lat1]  // 注意多边形首尾坐标要闭合，即首尾一致
  ]
]);


创建点
turf.point([-122.676207, 45.52345])


turf.polygon / turf.lineString 仅仅是创建“标准 GeoJSON Feature”，不是 turf 特有的“私有数据类型”——它们就是普通的 GeoJSON 对象，turf 的函数把这些对象当参数进行空间计算。
```







# 高德

## 文档链接

```
前端库安装
pnpm i @amap/amap-jsapi-loader --save

坐标拾取
https://lbs.amap.com/tools/picker


地图样式
https://lbs.amap.com/api/javascript-api-v2/guide/map/map-style


图层管理
https://lbs.amap.com/api/javascript-api-v2/guide/map/map-layer#t1
各类官方图层
https://lbs.amap.com/api/javascript-api-v2/guide/layers/official-layers


高德地图api 字典
https://a.amap.com/jsapi/static/doc/20230922/index.html


地图事件
https://a.amap.com/jsapi/static/doc/20230922/index.html#event


点标记
https: //lbs.amap.com/api/javascript-api-v2/guide/amap-marker/default-marker
使用api列表: https://a.amap.com/jsapi/static/doc/20230922/index.html#marker


点聚合
https://lbs.amap.com/api/javascript-api-v2/guide/amap-massmarker/marker-cluster


加载geojson数据
https://lbs.amap.com/api/javascript-api-v2/guide/map/map-layer#t3  下  4、GeoJSON 类
使用AMap.GeoJSON插件


线绘制
https://lbs.amap.com/api/javascript-api-v2/guide/amap-line/poly-line


面绘制
https://lbs.amap.com/api/javascript-api-v2/guide/amap-polygon/polygon


弹窗
https://lbs.amap.com/api/javascript-api-v2/guide/overlays/info-window

控件
https://lbs.amap.com/api/javascript-api-v2/guide/overlays/toolbar


监听事件
moveend 地图移动结束后触发，包括平移，以及中心点变化的缩放。如地图有拖拽缓动效果，则在缓动结束后触发
https://a.amap.com/jsapi/static/doc/20230922/index.html#mapeventmoveend
```



## 初始化地图

```vue
<script setup>
// 从包 @amap/amap-jsapi-loader 引入的一个“高德地图 JS API 异步加载器”。
import AMapLoader from '@amap/amap-jsapi-loader'; 
import { onMounted, onUnmounted } from 'vue';

let map = null; // 存储高德地图

onMounted(async () => {
  window._AMapSecurityConfig = {
    securityJsCode: '545d60cd43d478ee86bfa9d3352353ed' // 安全密匙
  };
  const AMap = await AMapLoader.load({
    key: 'afe29c92d6f63b4452d5a74a73d700e7', // 申请好的Web端开发者Key，首次调用 load 时必填
    version: '2.0', // 指定要加载的 JSAPI 的版本，
    plugins: ["AMap.Scale"], // 需要使用的的插件列表，如比例尺'AMap.Scale'等
  });


  map = new AMap.Map("container", {
    viewMode: '3D', // 使用3D视图
    center:[116.518542, 39.924677],// 武汉:[114.298572, 30.584355],
    zoom: 4, //地图级别
    layers: [], // 初始化地图图层
    overlayGroups: [], // 初始化地图覆盖物群组
    mapStyle: "amap://styles/dark", // 初始化地图样式
  });
})
</script>

<template>
<div id="container"></div>
</template>

<style scoped>
#container {
  width: 100%;
  height: 100vh;
}
</style>
```



## 收集用到的方法

```js
1.地图基础
new AMap.Map(domId, options) —— 创建地图实例
	主要选项：center, zoom, viewMode:'3D', mapStyle, layers, showIndoorMap
	注意： layers: []（会丢默认底图）

map.setCenter(lnglat) —— 设置地图中心点

map.setFitView([overlays]) —— 自动调整地图视野以适应指定覆盖物

map.setZoom(zoom) —— 设置地图缩放级别

map.setZoomAndCenter(zoom, [lng,lat]) —— 同时设置缩放和中心点

map.clearMap() —— 清除地图上所有覆盖物

map.getAllOverlays(type) —— 获取指定类型的所有覆盖物

map.remove(overlays) —— 删除指定覆盖物

事件监听：
    map.on('click', callback) - 监听地图点击事件
    map.on('moveend', callback) - 监听地图移动结束事件
    map.on('zoomend', callback) - 监听地图缩放结束事件


2.控件
AMap.ToolBar —— 地图平移缩放控件

AMap.Scale —— 地图比例尺控件

AMap.HawkEye —— 鹰眼控件，显示缩略地图

AMap.MapType —— 地图类型切换控件（如卫星图、路网图）

AMap.Geolocation —— 定位控件，实现定位功能

AMap.ControlBar —— 3D视角旋转及倾斜控件


3.路线与服务
AMap.Driving —— 驾车路径规划服务
    new AMap.Driving({ map, panel, policy }) —— 创建驾车路线规划实例
    driving.search(points | start, end, cb) —— 查询驾车路线
    回调参数：status（状态），result（路线详情）

公交路线规划
    new AMap.Transfer(options) - 创建公交路线规划实例
    transfer.search(start, end, callback) - 搜索公交路线
    transfer.clear() - 清除公交路线

步行路线规划
    new AMap.Walking(options) - 创建步行路线规划实例
    walking.search(start, end, callback) - 搜索步行路线
    walking.clear() - 清除步行路线
    

4.覆盖物
    AMap.Marker —— 标记点覆盖物

    AMap.Text —— 文字覆盖物

    AMap.Polyline —— 折线覆盖物

    AMap.BezierCurve —— 贝塞尔曲线

    AMap.Polygon —— 多边形覆盖物

    AMap.Circle —— 圆形覆盖物

    AMap.Rectangle —— 矩形覆盖物

    AMap.Ellipse —— 椭圆覆盖物

    AMap.InfoWindow —— 信息窗口弹框

    AMap.ContextMenu —— 右键菜单

辅助对象：
    AMap.LngLat —— 经纬度坐标对象

    AMap.Bounds —— 地图矩形范围

    new AMap.Icon(options) - 创建自定义图标
    new AMap.Size(width, height) - 创建尺寸对象
    new AMap.Pixel(x, y) - 创建像素偏移对象

常用方法：
    map.add(overlay | [overlays]) —— 添加覆盖物
    map.remove(overlay) - 从地图移除覆盖物
        remove的使用：需要先使用map.value.getLayers()获取图层数组，接着遍历图层数组，把要删除的哪一项（layer）专门挑出来，然后才能用remove方法删除
        
    map.getAllOverlays(type) - 获取指定类型的所有覆盖物

    overlay.setOptions(opts) —— 设置覆盖物样式或属性

    overlay.on('mouseover' | 'mouseout' | 'dragging', handler) —— 绑定覆盖物事件

    marker.getPosition() —— 获取标记点位置

    infoWindow.open(map, lnglat) —— 打开信息窗口

    contextMenu.addItem(text, fn, index) —— 添加菜单项
    
    
5.GeoJSON
    插件：'AMap.GeoJSON' —— 支持加载和解析GeoJSON数据
    new AMap.GeoJSON({ geoJSON, getPolygon(geojson, lnglats){ return new AMap.Polygon({...}) }, getPolyline(geojson, lnglats){ return new AMap.Polyline({...}) } })
    
    new AMap.GeoJSON({ geoJSON, getPolygon(), getPolyline() }) —— 自定义GeoJSON转多边形或折线


6.几何计算
AMap.GeometryUtil.distance(lnglatA, lnglatB)  计算两点距离
AMap.GeometryUtil.closestOnLine(point, pathCoordsArray)  计算点到折线最近投影点


7.交互样式编译
    'AMap.PolylineEditor' → new AMap.PolylineEditor(map, polyline) —— 折线编辑器
    'AMap.PolygonEditor' → new AMap.PolygonEditor(map, polygon) —— 多边形编辑器
    'AMap.CircleEditor' → new AMap.CircleEditor(map, circle) —— 圆形编辑器
    'AMap.EllipseEditor' → new AMap.EllipseEditor(map, ellipse) —— 椭圆编辑器
操作：
    editor.open() —— 开启编辑模式

    editor.close() —— 关闭编辑模式


8.AMapUI 基础控件
加载：
    AMapLoader.load({ AMapUI:{ version:'1.1', plugins:['control/BasicControl'] } }) —— 加载基础控件插件
    或后续 window.AMapUI.loadUI(['control/BasicControl'], cb)
使用：
	缩放控件
    new AMapUI.BasicControl.Zoom({ position:'lb', showZoomNum:true })
    图层切换控件，包括交通、卫星图等
    new AMapUI.BasicControl.LayerSwitcher({ position:'rt', showTraffic:true, showSatellite:true, satelliteLayer })


9.事件与交互
事件绑定：

    overlay.on('mouseover' | 'mouseout') —— 绑定覆盖物鼠标事件

    marker.on('dragging') —— 标记点拖动事件

    map.on('click')、map.on('rightclick') —— 地图点击事件

    Driving 回调函数处理路线查询结果

建议：

    批量解绑事件 overlay.off()

    组件卸载时调用 map.destroy() 释放资源


10.驾车路线规划 AMap.Driving


11.鼠标绘制/测量工具 AMap.MouseTool
距离测量：mouseTool.rule(options)
面积测量：mouseTool.measureArea(options)
绘制折线：mouseTool.polyline(options)
多边形：mouseTool.polygon(options)
矩形：mouseTool.rectangle(options)
圆：mouseTool.circle(options)
结束并可清除：mouseTool.close(true) 注意：再次开始新的绘制会自动关闭上一种模式


12. 地点搜索提示 AutoComplete
const autoComplete = new AMap.AutoComplete({ city:'武汉' })
autoComplete.search(keyword, (status, result)=> {
  // result.tips
})


13.POI 搜索 PlaceSearch
const placeSearch = new AMap.PlaceSearch({ city:'027' })
placeSearch.search(keyword, (status, result)=>{
  // result.poiList.pois
})


14.地理编码 / 逆地理编码 AMap.Geocoder
实例：
const geocoder = new AMap.Geocoder({ city:'027' })
地址 → 坐标：
geocoder.getLocation(address, (status, result)=>{
  const loc = result.geocodes[0].location
})
坐标 → 地址：
geocoder.getAddress([lng,lat], (status, result)=>{
  const comp = result.regeocode.addressComponent
})


15.天气插件
    new AMap.Weather() - 创建天气插件实例
    weather.getForecast(city, callback) - 获取天气预报信息
```

# 待总结笔记

```js

```



# 底部

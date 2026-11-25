# dat.GUI

### 引入 & 创建 GUI



```bash
pnpm install dat.gui
pnpm add -D @types/dat.gui
```



```ts
import * as dat from 'dat.gui';

const gui = new dat.GUI(); // 创建一个 GUI 面板
```

此时页面左上角会出现一个空的控制面板。

------

### 核心思想：绑定一个“参数对象”

所有控件都基于一个 **普通对象**，例如：



```ts
const params = {
  speed: 10,
  enabled: true,
  mode: 'easy',
  color: '#ff0000',
  doSomething: () => {
    console.log('Button clicked!');
  },
};
```

然后用 `gui.add`/`gui.addColor` 把这个对象的属性绑到控件上。

------

### 常见控件语法一览

#### 数值滑块（number）

```ts
// 语法：gui.add(对象, '属性名', 最小值, 最大值, 步长)
gui.add(params, 'speed', 0, 100, 1).name('速度');
```

- 适用：`number` 类型；
- 效果：生成一个 0~100 的滑块，步长 1；
- `.name('速度')`：设置显示名称。

------

#### 复选框（boolean）

```ts
// 语法：gui.add(对象, '属性名')
gui.add(params, 'enabled').name('是否启用');
```

- 适用：`boolean`；
- 效果：生成一个勾选框。

------

#### 下拉菜单（枚举：数组）

```ts
// 语法：gui.add(对象, '属性名', ['值1', '值2', '值3'])
gui.add(params, 'mode', ['easy', 'normal', 'hard']).name('模式');
```

- 适用：`string` 或 `number` 这类离散值；
- 效果：生成一个下拉框，选中的值写回 `params.mode`。

------

#### 下拉菜单（映射文本到值）

```ts
const params2 = {
  quality: 'high', // 实际存储值
};

gui
  .add(params2, 'quality', { 低: 'low', 中: 'medium', 高: 'high' })
  .name('画质');
```

- 左边是显示在 UI 的文本（例：低/中/高）；
- 右边是实际写入 `params2.quality` 的值（low/medium/high）。

------

#### 颜色选择器

```ts
gui.addColor(params, 'color').name('颜色');
```

- 适用：字符串颜色（`'#rrggbb'`）、数字颜色（`0xff0000`）、或者 `{ r,g,b }` 对象等；
- 会弹出一个颜色选择器，修改时自动更新 `params.color`。

------

#### 按钮（调用函数）

```ts
gui.add(params, 'doSomething').name('执行动作');
```

- 适用：对象上的函数；
- 点击按钮时会调用 `params.doSomething()`。

------

### 监听变化：`onChange` / `onFinishChange`

大多数时候，你会想“属性变了以后做点事情”，就用 `onChange`。

```ts
gui
  .add(params, 'speed', 0, 100, 1)
  .name('速度')
  .onChange((value: number) => {
    console.log('新速度：', value);
    // 这里可以调用你的逻辑，比如更新某个动画的速度
  });
```

- `onChange`：拖动滑块过程中持续触发；
- `onFinishChange`：只在用户“操作结束”时触发一次（如松开鼠标）。

```ts
gui
  .add(params, 'speed', 0, 100, 1)
  .onFinishChange((value: number) => {
    console.log('最终速度：', value);
  });
```

------

### 文件夹（分组）：`addFolder`

用于对控件进行分组，便于分类管理。

```ts
const gui = new dat.GUI();

const moveFolder = gui.addFolder('移动相关');
const styleFolder = gui.addFolder('样式相关');

const obj = { x: 0, y: 0, size: 10, color: '#00ff00' };

moveFolder.add(obj, 'x', -100, 100, 1).name('X 坐标');
moveFolder.add(obj, 'y', -100, 100, 1).name('Y 坐标');

styleFolder.add(obj, 'size', 1, 50, 1).name('尺寸');
styleFolder.addColor(obj, 'color').name('颜色');
```

- 文件夹可以折叠/展开，适合参数多的时候分类使用。

------

###  一个完整的极简示例（可以直接跑）



```ts
import * as dat from 'dat.gui';

type Params = {
  speed: number;
  enabled: boolean;
  mode: string;
  color: string;
  logState: () => void;
};

const params: Params = {
  speed: 10,
  enabled: true,
  mode: 'easy',
  color: '#ff0000',
  logState: () => {
    console.log('当前参数：', params);
  },
};

const gui = new dat.GUI();

// 数值滑块
gui
  .add(params, 'speed', 0, 100, 1)
  .name('速度')
  .onChange((v: number) => {
    console.log('速度变为：', v);
  });

// 勾选框
gui
  .add(params, 'enabled')
  .name('启用')
  .onChange((v: boolean) => {
    console.log('启用状态：', v);
  });

// 下拉菜单
gui
  .add(params, 'mode', ['easy', 'normal', 'hard'])
  .name('模式')
  .onChange((v: string) => {
    console.log('模式切换为：', v);
  });

// 颜色选择器
gui
  .addColor(params, 'color')
  .name('颜色')
  .onChange((v: string) => {
    console.log('颜色变为：', v);
  });

// 按钮
gui.add(params, 'logState').name('打印当前参数');
```

在这个例子里：

- dat.GUI 负责把 UI 控件和 `params` 绑定；
- 通过 `onChange`/`logState` 再去调用任何业务逻辑就行了。

------

### 总结

> 使用 dat.GUI 的套路就是：
>
> 1. 准备一个普通对象 `params`；
> 2. `const gui = new dat.GUI()`；
> 3. 用 `gui.add` / `gui.addColor` 绑定 `params` 的某个属性；
> 4. 可选：用 `onChange` 在属性变化时执行逻辑；
> 5. 如需分类就用 `gui.addFolder('名字')`。











# lil-gui

新一代，dat.GUI已经不再维护了



# cesium

## 初始化项目

使用vite，使用typescript

创建项目

```
pnpm init
当前项目中生存package.json,存放项目的依赖清单，脚本名称等

pnpm install vite
把vite安装到当前的项目依赖里，默认是 dependencies，除非加 -D

pnpm create vite
运行官方的脚手架，生成一个新的vite项目模板

pnpm install cesium
安装 CesiumJS 这个库到当前项目里

pnpm add vite-plugin-cesium -D
安装vite-plugin-cesium作为开发依赖

pnpm install cesium-navigation-es6 --save
罗盘比例尺等工具
```



在项目的ts文件中引入

```ts
import 'cesium/Build/Cesium/Widgets/widgets.css'
```

widgets样式：cesium的viewer会创建出一套dom，包含credit、水印，一些默认控件容器等，这些都依赖widgets里面的样式进行布局，缺了它往往不是不能渲染，而是到导致UI布局渲染混乱，后续一开控件就乱



vite环境变量

```env
VITE_CESIUM_TOKEN = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI2YmMyZDhmYy1hYmY2LTRmZWMtYWVlMC05ZjZjMGQ0OTg3ZDgiLCJpZCI6MTY4NTYzLCJpYXQiOjE2OTU2OTg3MjB9.Azblv_bw0hAeMJvZteBqPxFcSlGje988PN3tZZAKYDg
```

在 .env里定义VIET_CESIUM_TOKEN，vite会把以`VITE_`开头的变量注入到前端代码，通过`import.meta.env.VITE_CESIUM_TOKEN` 读取



vite.config.ts

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import cesium from 'vite-plugin-cesium'
import path from 'path';

// https://vite.dev/config/
export default defineConfig({
  plugins: [vue(), cesium()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  }
})
```

defineConfig(...)

- 作用：给 Vite 配置提供类型提示与更好的 TS 推断（本质上就是导出一个配置对象）。

[plugins: [vue(), cesium()\]]

- [vue()]（[@vitejs/plugin-vue]）
  - 作用：让 Vite 能编译 .vue 单文件组件（SFC），支持 `<template>/<script setup>/<style>` 等语法。
  - 没它的表现：导入 .vue会报错，项目跑不起来。
- [cesium()]（[vite-plugin-cesium]）
  - 作用：解决 Cesium 在 Vite 环境下的“静态资源与 Worker”问题（Cesium 运行时会加载 `Workers/`、`Assets/`、`Widgets/` 等资源）。

resolve.alias: { '@': path.resolve(__dirname, 'src') }

- 作用：可以用@/xxx代表src/xxx例如：
  - `import { cesiumApp } from '@/core/CesiumApp'`



tsconfig.app.json

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

 仅仅“能跑起来”（Vite 打包能解析）

- 不需要 tsconfig.app.json 配合。

“IDE 不报红 + TypeScript 类型检查通过（vue-tsc/tsc）”

- **需要** tsconfig.app.json（或 tsconfig.json）里配置 `compilerOptions.paths` 来告诉 TypeScript：`@/*` 应该映射到 `src/*`。



### vite创建一个不包含任何框架的项目

index.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>vite-project</title>
    <style>
      body {
        margin: 0;
      }
      #cesiumContainer {
        width: 100vw;
        height: 100vh;
      }
    </style>
  </head>
  <body>
    <div id="cesiumContainer"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

main.ts

```ts
import * as Cesium from "cesium"
import 'cesium/Build/Cesium/Widgets/widgets.css' 
import CesiumNavigation from 'cesium-navigation-es6'

//注册token
Cesium.Ion.defaultAccessToken = import.meta.env.VITE_CESIUM_TOKEN

// 加载地形数据
const terrainProvider = await Cesium.createWorldTerrainAsync({
  requestVertexNormals: true, // 让地形有更真实的光照阴影（可选）
  requestWaterMask: true,     // 水面效果（可选）
});

// 选择 id为cesiumContainer 的html元素作为cesium容器
// 第二个参数中可以设置控件
const viewer : Cesium.Viewer = new Cesium.Viewer('cesiumContainer', {
  // imageryProvider: false, // 设置为 false，不加载任何底图
  timeline: false, // 隐藏时间轴
  animation: false, // 动画控件
  baseLayerPicker: false, // 底图切换控件
  geocoder: false, // 搜索控件
  homeButton: false, // 复位功能
  sceneModePicker: false, // 二三维切换功能
  navigationHelpButton: false, // 隐藏帮助按钮
  scene3DOnly: true, // 只使用3D场景
  shouldAnimate: true, // 开启动画效果，比如太阳位置变化
  infoBox: false, // 隐藏信息框
  terrainProvider: terrainProvider, // 使用加载的地形数据
})

// 显示一些比例尺，罗盘控件
new CesiumNavigation(viewer,{
    defaultResetView: Cesium.Rectangle.fromDegrees(116.0, 39.5, 116.8, 40.3), // 使用矩形区域定义重置视图
    enableCompass: true, // 是否显示指南针
    enableZoomControls: true, // 是否显示缩放控件
    enableDistanceLegend: true, // 是否显示距离图例
})
```

vite.config.ts

```ts
import { defineConfig } from "vite";
import cesium from "vite-plugin-cesium";

export default defineConfig({
  plugins: [cesium()],
});
```





### 关于地形

```ts
// 加载地形数据
const terrainProvider = await Cesium.createWorldTerrainAsync({
  requestVertexNormals: true, // 让地形有更真实的光照阴影（可选）
  requestWaterMask: true,     // 水面效果（可选）
});

// 选择 id为cesiumContainer 的html元素作为cesium容器
// 第二个参数中可以设置控件
const viewer = new Cesium.Viewer('cesiumContainer', {
  ...
  terrainProvider: terrainProvider, // 使用加载的地形数据
})






Cesium World Terrain（CWT）是 Cesium 官方在 Cesium ion上托管的一套“全球地形数据集”（一个 ion 资产 / asset）。
它本身不是代码类，而是 服务端的数据+瓦片服务。



CesiumTerrainProvider 是 CesiumJS 里“地形提供者”的具体实现类
它是一个通用地形 provider，可以对接：
ion 上的地形资产：CesiumTerrainProvider.fromIonAssetId(assetId, options)
自建/第三方的地形服务：CesiumTerrainProvider.fromUrl(url, options)
例如：
const viewer = new Cesium.Viewer("cesiumContainer", {
  terrainProvider: await Cesium.CesiumTerrainProvider.fromIonAssetId(3956, {
    requestVertexNormals: true
  })
});
不直接 new 构造函数，因为初始化需要异步获取 layer.json 等元数据，所以直接调用 CesiumTerrainProvider.fromIonAssetId 或 CesiumTerrainProvider.fromUrl 方法



createWorldTerrainAsync(options) 是一个便捷封装：
专门用来创建“指向 Cesium ion 的 Cesium World Terrain”的 CesiumTerrainProvider，并返回 Promise<CesiumTerrainProvider>。
例如：
const terrainProvider = await Cesium.createWorldTerrainAsync({
  requestVertexNormals: true,
  requestWaterMask: true,
});
```





## 位置，变换等计算



### 坐标表示

```js
// 三维笛卡尔点，其值常作为地心坐标系（世界坐标系）下的坐标值
new Cesium.Cartesian3(50.0,50.0,50.0)
Cartesian3 对象字面量形式为：{x: 50.0, y: 50.0, z: 50.0}

// 2D 局部平面点，常作为屏幕坐标参数传入
new Cesium.Cartesian2(-10.0, -10.0)

// 经纬度 + 高程 → 世界坐标 (WGS84)，世界坐标：地心为原点的笛卡尔坐标系。
Cesium.Cartesian3.fromDegrees(lon, lat, height)：.
```



### Matrix4矩阵计算



#### 坐标乘以矩阵

```js
// 计算矩阵与Cartesian3点的乘积
Cesium.Matrix4.multiplyByPoint(matrix, cartesian, result) → Cartesian3
作用：将矩阵变换应用到单个点
参数：
matrix : Matrix4 要乘以点的矩阵。
cartesian : Cartesian3 要乘以矩阵的点。
result : Cartesian3 ，存储结果的对象。
```

点的齐次坐标表示：
$$
\vec{p}_{homogeneous} = \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix}
$$
矩阵与点的乘法：
$$
\vec{p}_{result} = M \times \vec{p}_{homogeneous} = \begin{pmatrix} m_{00} & m_{01} & m_{02} & m_{03} \\ m_{10} & m_{11} & m_{12} & m_{13} \\ m_{20} & m_{21} & m_{22} & m_{23} \\ m_{30} & m_{31} & m_{32} & m_{33} \end{pmatrix} \begin{pmatrix} x \\ y \\ z \\ 1 \end{pmatrix}
$$
结果坐标：
$$
\begin{pmatrix} x' \\ y' \\ z' \end{pmatrix} = \begin{pmatrix} m_{00}x + m_{01}y + m_{02}z + m_{03} \\ m_{10}x + m_{11}y + m_{12}z + m_{13} \\ m_{20}x + m_{21}y + m_{22}z + m_{23} \end{pmatrix}
$$




#### 创建平移矩阵

```js
Cesium.Matrix4.fromTranslation(translation, result) → Matrix4
用途： 创建纯平移矩阵
参数：
-translation - 平移向量 (Cartesian3)
-result - 可选，存储结果的矩阵对象

// 创建一个将物体平移 (100, 200, 300) 的矩阵
const translation = new Cesium.Cartesian3(100, 200, 300);
const translationMatrix = Cesium.Matrix4.fromTranslation(translation);
```

平移矩阵：
$$
T = \begin{pmatrix} 1 & 0 & 0 & tx \\ 0 & 1 & 0 & ty \\ 0 & 0 & 1 & tz \\ 0 & 0 & 0 & 1 \end{pmatrix}
$$
应用到点的效果：
$$
\begin{pmatrix} x' \\ y' \\ z' \end{pmatrix} = \begin{pmatrix} x + tx \\ y + ty \\ z + tz \end{pmatrix}
$$








#### 现有矩阵上应用平移变换

```js
Cesium.Matrix4.multiplyByTranslation(matrix, translation, result) → Matrix4
用途： 在现有的矩阵上应用平移变换
参数：
-matrix - 输入的变换矩阵
-translation - 平移向量 (Cartesian3 格式，包含 x, y, z)
-result - 存储结果的矩阵对象


// 创建一个单位矩阵
const identityMatrix = Cesium.Matrix4.IDENTITY;

// 在单位矩阵基础上进行平移
const combined = Cesium.Matrix4.multiplyByTranslation(
  identityMatrix,
  new Cesium.Cartesian3(100, 0, 0),
  new Cesium.Matrix4()
);
console.log(combined) 
// 输出结果：
// Matrix4 {0: 1, 1: 0, 2: 0, 3: 0, 
//          4: 0, 5: 1, 6: 0, 7: 0, 
//          8: 0, 9: 0, 10: 1, 11: 0, 
//          12: 100, 13: 0, 14: 0, 15: 1}
```

平移矩阵：
$$
T = \begin{pmatrix} 1 & 0 & 0 & tx \\ 0 & 1 & 0 & ty \\ 0 & 0 & 1 & tz \\ 0 & 0 & 0 & 1 \end{pmatrix}
$$
复合矩阵（右乘平移）：
$$
result = M \times T = \begin{pmatrix} m_{00} & m_{01} & m_{02} & m_{03} + tx \\ m_{10} & m_{11} & m_{12} & m_{13} + ty \\ m_{20} & m_{21} & m_{22} & m_{23} + tz \\ m_{30} & m_{31} & m_{32} & m_{33} \end{pmatrix}
$$








#### 创建非均匀缩放矩阵

```js
Cesium.Matrix4.fromScale(scale, result) → Matrix4
用途：用于在不同方向上以不同比例拉伸或压缩物体，可单独控制x, y, z三个方向上的缩放比例
scale	Cartesian3	The x, y, and z scale factors.
result	Matrix4  自主设置哪个四维矩阵对象用来存储这个缩放矩阵

const scale = new Cesium.Cartesian3(2.0, 1.0, 0.5);
const scaleMatrix = Cesium.Matrix4.fromScale(scale);
// 结果：x 方向缩放 2 倍，y 方向不变，z 方向缩放 0.5 倍
```

非均匀缩放矩阵：
$$
S = \begin{pmatrix} sx & 0 & 0 & 0 \\ 0 & sy & 0 & 0 \\ 0 & 0 & sz & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}
$$
应用到点的效果：
$$
\begin{pmatrix} x' \\ y' \\ z' \end{pmatrix} = \begin{pmatrix} sx \cdot x \\ sy \cdot y \\ sz \cdot z \end{pmatrix}
$$


#### 统一缩放矩阵

```js
Cesium.Matrix4.multiplyByUniformScale(matrix, scale, result) → Matrix4
作用：统一缩放物体的x, y, z三个方向，保证物体形状和比例关系不变

`matrix` | Matrix4 | 输入的 4×4 矩阵                                
`scale`  | Number  | 统一缩放因子（标量值），作用于 x、y、z 三个方向  
`result` | Matrix4 | 存储结果的矩阵对象（可选，如果未提供则创建新矩阵）

const matrix = Cesium.Matrix4.IDENTITY; // 单位矩阵
const scaled = Cesium.Matrix4.multiplyByUniformScale(matrix, 2.0, new Cesium.Matrix4()); 
// 结果：所有坐标乘以 2，物体放大 2 倍
console.log(scaled);
```

原始矩阵（齐次坐标）：
$$
M = \begin{pmatrix} m_{00} & m_{01} & m_{02} & m_{03} \\ m_{10} & m_{11} & m_{12} & m_{13} \\ m_{20} & m_{21} & m_{22} & m_{23} \\ m_{30} & m_{31} & m_{32} & m_{33} \end{pmatrix}
$$
统一缩放矩阵：
$$
S = \begin{pmatrix} scale & 0 & 0 & 0 \\ 0 & scale & 0 & 0 \\ 0 & 0 & scale & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}
$$
结果（矩阵乘法）：
$$
result = M \times S = \begin{pmatrix} m_{00} \cdot scale & m_{01} \cdot scale & m_{02} \cdot scale & m_{03} \\ m_{10} \cdot scale & m_{11} \cdot scale & m_{12} \cdot scale & m_{13} \\ m_{20} \cdot scale & m_{21} \cdot scale & m_{22} \cdot scale & m_{23} \\ m_{30} & m_{31} & m_{32} & m_{33} \end{pmatrix}
$$






变换合成

```js
new Cesium.TranslationRotationScale(translation, rotation, scale)
由平移、旋转和缩放共同构成的一种仿射变换。
参数：
  translation: Cartesian3 平移向量，默认值为 Cartesian3.ZERO
  rotation: Quaternion 旋转四元数，默认值为 Quaternion.IDENTITY
  scale: Cartesian3 缩放向量，默认值为 new Cartesian3(1.0, 1.0, 1.0)
```









### cartographic 

cartographic 用经度、纬度、高度 来描述地球椭球（默认 WGS84）上的一个位置

```js
// new Cesium.Cartographic(longitude, latitude, height)，
// 实例化的是一个 Cartographic（大地坐标/地理坐标）对象：用 经度、纬度、高度 来描述地球椭球（默认 WGS84）上的一个位置。属于一个地理意义上的坐标表达，因为有时候很难用三维笛卡尔坐标来描述一个位置
Cesium.Cartographic(longitude, latitude, height)
longitude：弧度表示的经度
latitude：弧度表示的纬度
height：高度，单位米

```



### Rectangle

以经纬度坐标定义的二维矩形区域，多用于矩形几何

```js
// new Cesium.Rectangle(west, south, east, north)
参数：
  west：西经，范围-pi到pi，单位为弧度。
  south：南纬，范围-pi/2到pi/2，单位为弧度。
  east：东经，范围-pi到pi，单位为弧度。
  north：北纬，范围-pi/2到pi/2，单位为弧度。
返回值：
  一个 Cesium.Rectangle 对象，一个以经纬度坐标定义的二维矩形区域。
  
  

// Cesium.Cartesian3.fromDegreesArrayHeights(coordinates, ellipsoid, result) → Array.<Cartesian3>
参数：
  coordinates：一个包含经度、纬度和高度的数组，格式为 [lon1, lat1, height1, lon2, lat2, height2, ...]。
  ellipsoid：可选参数，指定用于转换的椭球体，默认值为 Cesium.Ellipsoid.WGS84。
  result：可选参数，用于存储结果的数组，如果提供，将在该数组中存储结果。
返回值：
  一个包含 Cartesian3 对象的数组，每个对象表示输入坐标对应的三维位置，格式为 [Cartesian3, Cartesian3, ...]。
  
  
  
 // Cesium.Rectangle.fromCartesianArray(cartesians, ellipsoid, result) → Rectangle
参数：
  cartesians：一个包含 Cartesian3 对象的数组，格式为 [Cartesian3, Cartesian3, ...]。
  ellipsoid：可选参数，指定用于转换的椭球体，默认值为 Cesium.Ellipsoid.WGS84。
  result：可选参数，用于存储结果的 Rectangle 对象，如果提供，将在该对象中存储结果。
返回值：
  一个 Cesium.Rectangle 对象，一个以经纬度坐标定义的二维矩形区域。

补充说明：
  该方法会计算输入的 Cartesian3对象数组中所有点的经纬度范围，即：
    找出所有点中的最小经度、最大经度、最小纬度、最大纬度
    用这四个极值构成一个轴对齐的包围矩形（AABB）
```





### 笛卡尔 -> 地理坐标

```js
// Cesium.Cartographic.fromCartesian(cartesian, ellipsoid, result) → Cartographic，这个静态方法表示用一个笛卡尔坐标转换为地理坐标，也就是上面的new Cesium.Cartographic(longitude, latitude, height)这个对象

const cartographic = Cesium.Cartographic.fromCartesian(currentPos);
```



### 星下点计算

```js
// 1) 卫星 ECEF (m) —— 使用你给的坐标
const satPos = new Cesium.Cartesian3(
  6978102.994972304,
  -47375.42771,
  -27040.55263
);

// 2) 计算星下点（nadir）并建立 ENU -> 固定系 变换
const ellipsoid = Cesium.Ellipsoid.WGS84;
const nadir = ellipsoid.scaleToGeodeticSurface(satPos);
/**
  这里说下星下点的计算：
  - scaleToGeodeticSurface 会把一个 ECEF 点投影到 WGS84 椭球表面，得到星下点坐标
scaleToGeodeticSurface(cartesian, result) → Cartesian3
  - scaleToGeodeticSurface 会把一个 ECEF 点沿椭球的地理法线（geodetic surface normal）投影到 WGS84 椭球表面
  - 对于球体，地理法线与地心径向一致；但对于椭球二者不同
  - 如果卫星位置无效（比如在地球内部），会返回 undefined
  - 如果你想沿“地心径向”求交点，请使用：ellipsoid.scaleToGeocentricSurface(cartesian)
 */
```





### 计算两点之间距离

`Cesium.Cartesian3.distance(left, right) → number`

```js
/**
计算两个笛卡尔坐标点之间的距离。
left {Cartesian3} 第一个点。
right {Cartesian3} 第二个点。
返回 {number} 两点之间的距离。
 */
示例：
const d = Cesium.Cartesian3.distance(new Cesium.Cartesian3(1.0, 0.0, 0.0), new Cesium.Cartesian3(2.0, 0.0, 0.0));
result: d = 1.0
```















### 坐标转换

坐标转化的意义

| 转换类型            | 作用说明                                         |
| ------------------- | ------------------------------------------------ |
| 经纬度 ↔ 世界坐标   | 地理位置与三维空间位置的互通，支持渲染和空间计算 |
| 角度（度） ↔ 弧度   | 用户友好表达与计算机数学计算之间的单位转换       |
| 屏幕坐标 ↔ 世界坐标 | 用户交互（点击、拾取、绘制）与三维场景显示的桥梁 |



基础坐标转换

```ts
const pos = Cesium.Cartesian3.fromDegrees(116.391, 39.907, 500); // 北京上空500m，经纬度高程 → 笛卡尔
console.log(pos); 
// Cartesian3 {x: -2177899.4651809996, y: 4389080.984830701, z: 4070390.6800702307}

const carto = Cesium.Cartographic.fromCartesian(pos); 
// 笛卡尔 → 经纬度高程,结果是弧度单位,弧度单位范围：经度[-π,π],纬度[-π/2,π/2]

console.log(carto);
// Cartographic {longitude: 2.0314061696887205, latitude: 0.6965085445933772, height: 499.999999999554}


const lonDeg = Cesium.Math.toDegrees(carto.longitude); // 弧度转度数
const latDeg = Cesium.Math.toDegrees(carto.latitude);
const h = carto.height;

console.log({ lonDeg, latDeg, h }); 
// {lonDeg: 116.39100000000002, latDeg: 39.907000000000004, h: 499.999999999554}

// 度转弧度 Cesium.Math.toRadians(degree)
```







### 坐标系

#### 局部 ENU 坐标系

假如该参考系的原点为地球表面上一点，在该点处创建一个切平面，切平面与地球表面相切，

x轴在切平面内指向地理东

y轴在切平面内指向地理北

z轴垂直于切平面指向天空

- `Transforms.eastNorthUpToFixedFrame(origin)` → 返回一个 4x4 `Matrix4`：平移 + 旋转
- 世界→本地：
  - 逆矩阵：`Matrix4.inverse(enu)`
  - `Matrix4.multiplyByPoint(enuInverse, worldPoint)`世界坐标系下的点坐标乘以enu的逆矩阵
- 本地→世界：
  - `Matrix4.multiplyByPoint(enu, localPoint)`本地点坐标乘以enu矩阵
- 可视化：
  - 把 `modelMatrix: enu` 传给 `DebugModelMatrixPrimitive`



矩阵字面量

```
enuMatrix：
| E_x  N_x  U_x  T_x |
| E_y  N_y  U_y  T_y |
| E_z  N_z  U_z  T_z |
| 0    0    0    1   |

前三列是局部坐标轴在世界参考系下的朝向，第四列是平移分量，相对于ECEF坐标系原点的偏移量

先用 Matrix4.getMatrix3() 提取左上角 3×3 子矩阵

可以通过： Cesium.Matrix3.getColumn(enu矩阵, 列数, new Cesium.Cartesian3());这个方法获取第几列的数值
```

![image-20260326175023313](./assets/image-20260326175023313.png)





核心理解：

- ENU 是以指定经纬度点为原点（通常指的是地球表面上的一点），X 指当地的东方向，Y 指北，Z 指上（垂直于椭球面）
- 这个变换矩阵同时包含：原点平移 + 局部轴旋转 → 世界

```js
//局部东-北-上坐标系 (ENU)，x轴指向东，y轴指向北，z轴指向上，生成一个 4x4 矩阵，将局部 ENU 坐标点变换为地心坐标。
Cesium.Transforms.eastNorthUpToFixedFrame(origin, ellipsoid, result) → Matrix4
参数：
origin：要转换的点的世界坐标 cartesian3
ellipsoid：参考椭球体 ，默认值为 Cesium.Ellipsoid.WGS84
result：可选的结果参数，用于存储生成的矩阵


// 求取 相对于模型坐标原点 东200m的点 的世界坐标
const enu = Cesium.Transforms.eastNorthUpToFixedFrame(originCartesian3)
Cesium.Matrix4.multiplyByPoint(enu, localPoint, result) → 世界坐标。
position: Matrix4.multiplyByPoint(enu, new Cartesian3(200,0,0), new Cartesian3())


// task2:假设有一点P 115,32  得到基于114,30的本地坐标
const point = new Cesium.Catesian3.fromDegrees(114, 30)
// 世界坐标转本地坐标的公式：enu矩阵的逆矩阵*世界坐标=本地坐标
const enuInverse = Cesium.Matrix4.inverse(enu, new Cesium.Matrix4())
const localPoint = Cesium.Matrix4.multiplyByPoint(enuInverse, point, new Cesium.Cartesian3())
console.log('点P在本地坐标系下的坐标', localPoint); // 输出点P在本地坐标系下的坐标
```

东-北-上坐标系：

<img src="./assets/image-20260106203752014.png" alt="image-20260106203752014" style="zoom: 50%;" />





#### 屏幕坐标

鼠标点击Canvas画布。Canvas中，默认的坐标系统是以画布的左上角为坐标原点(0, 0)，水平向右为X轴，垂直向下为Y轴，以像素为单位。

new Cesium.Cartesian2(x, y)

<img src="./assets/image-20251027051707047.png" alt="image-20251027051707047" style="zoom: 33%;" />



#### 笛卡尔空间直角坐标（世界坐标）

new Cesium.Cartesian3(x, y, z)

![微信图片_2025-10-27_051802_076](./assets/微信图片_2025-10-27_051802_076.png)





#### 经纬度坐标

地理坐标是用纬度、经度表示地面点位置的球面坐标

<img src="./assets/微信图片_2025-10-27_051847_889.jpg" alt="微信图片_2025-10-27_051847_889" style="zoom: 50%;" />



#### 右手系

```
把右手放在原点的位置，使大拇指，食指和中指互成直角，把大拇指指向x轴的正方向，食指指向y轴的正方向时，中指所指的方向就是z轴的正方向。
```





#### 地心惯性坐标系（ECI）

地心惯性（英语：Earth-centered inertial，缩写为**ECI**）

- 特点：坐标系**不随地球自转旋转**，相对于恒星固定
- **原点**：地球质心
- 轴向：
  - z轴沿着地球自转轴指向协议地极;
  - x轴位于赤道平面内,并指向春分点;
  - y轴符合右手笛卡儿坐标系。
  - “x – y”平面与地球的[赤道面](https://zh.wikipedia.org/wiki/赤道面)重合

<img src="./assets/image-20260119080911490.png" alt="image-20260119080911490" style="zoom: 50%;" />





#### 地心地固坐标系（ECEF）

cesium中任意一点使用的都是ECEF坐标系

- **特点**：与地球保持同步旋转
- **原点**：地球质心
- 轴向：
  - X 轴：x轴指向赤道与本初子午线(格林尼治子午线)的交点
  - Y 轴：赤道与东经 90° 交点方向
  - Z 轴：指向协议地极

<img src="./assets/image-20260119080650514.png" alt="image-20260119080650514" style="zoom: 50%;" />





```js
ECI ←→ ECEF（通过地球自转角度转换）
  ↓          ↓
恒星参考   地心地固参考
（不动）    （随地球转）
```

- 卫星轨道数据通常是 **ECI 坐标**（轨道力学计算结果）
- 地面设施数据通常是 **ECEF 坐标**（GPS、地理位置）





坐标系是一种用于表示地理要素、影像和观察值位置(例如公用地理框架内的GPS位置)的参照系统。



```js
// 设置原点的位置经纬
const position = Cesium.Cartesian3.fromDegrees(116.3975, 39.9085);

// === 可视化 相机所处的ECEF坐标系 ===
const XDir = new Cesium.Cartesian3(1, 0, 0);   // X向
const YDir = new Cesium.Cartesian3(0, 1, 0);   // Y向
const ZDir = new Cesium.Cartesian3(0, 0, 1);   // Z向

const dirLen = 5000000; // 方向线长度均为5000米

// X向终点位置
const xEnd = Cesium.Cartesian3.add(
  position,
  Cesium.Cartesian3.multiplyByScalar(
    XDir,
    dirLen,
    new Cesium.Cartesian3()
  ),
  new Cesium.Cartesian3()
);

// X向（红色线）
viewer.entities.add({
  position: xEnd,
  polyline: {
    positions: [position, xEnd],
    width: 4,
    material: Cesium.Color.RED
  },
  label: {
    text: "ECEF X",
    font: "14px Microsoft YaHei",
    fillColor: Cesium.Color.RED,
    outlineWidth: 2,
    outlineColor: Cesium.Color.WHITE,
    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
})

// Y向终点位置
const yEnd = Cesium.Cartesian3.add(
  position,
  Cesium.Cartesian3.multiplyByScalar(
    YDir,
    dirLen,
    new Cesium.Cartesian3()
  ),
  new Cesium.Cartesian3()
);

// Y向（绿色线）
viewer.entities.add({
  position: yEnd,
  polyline: {
    positions: [position, yEnd],
    width: 4,
    material: Cesium.Color.GREEN
  },
  label: {
    text: "ECEF Y",
    font: "14px Microsoft YaHei",
    fillColor: Cesium.Color.GREEN,
    outlineWidth: 2,
    outlineColor: Cesium.Color.WHITE,
    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
})

// Z向终点位置
const zEnd = Cesium.Cartesian3.add(
  position,
  Cesium.Cartesian3.multiplyByScalar(
    ZDir,
    dirLen,
    new Cesium.Cartesian3()
  ),
  new Cesium.Cartesian3()
)

// Z向（蓝色线）
viewer.entities.add({
  position: zEnd,
  polyline: {
    positions: [position, zEnd],
    width: 4,
    material: Cesium.Color.BLUE
  },
  label: {
    text: "ECEF Z",
    font: "14px Microsoft YaHei",
    fillColor: Cesium.Color.BLUE,
    outlineWidth: 2,
    outlineColor: Cesium.Color.WHITE,
    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
})


//  在原点位置添加一个标记点 
viewer.entities.add({
  position: position,
  point: {
    pixelSize: 8,
    color: Cesium.Color.RED,
    outlineColor: Cesium.Color.WHITE,
  },
  label: {
    text: '原点位置',
    font: '14px Microsoft YaHei',
    fillColor: Cesium.Color.WHITE,
    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
    outlineWidth: 2,
    verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
});

// 主相机飞到合适位置观察原点位置
viewer.camera.flyTo({
  destination: Cesium.Cartesian3.fromDegrees(116.3975, 39.9085, 15000),
  orientation: {
    heading: Cesium.Math.toRadians(0),
    pitch: Cesium.Math.toRadians(-45),  // 俯视角度
    roll: 0.0
  },
  duration: 0.2
});
```

![image-20260303154126504](./assets/image-20260303154126504.png)





#### 地理坐标系(Geogrpahic Coordinate System,GCS)

地理坐标系(GCs)使用三维球面来定义地球上的位置。参考平面是椭球面，坐标单位是经纬度。





#### 地球椭球体(Ellipsoid)

地球表面是一个凸凹不平的表面，而对于地球测量而言，地表是一个无法用数学公式表达的曲面，这样的曲面不能作为测量和制图的基准面。

假想一个扁率极小的椭圆，绕大地球体短轴旋转所形成的规则椭球体称之为地球椭球体。

地球椭球体表面是一个规则的数学表面，可以用数学公式表达，所以在测量和制图中就用它替代地球的自然表面。

地球椭球体有长半径和短半径之分，长半径(a)即赤道半径，短半径(b)即极半径。f=(a-b)la为椭球体的扁率，表示椭球体的扁平程度。

<img src="./assets/4.png" alt="4" style="zoom:67%;" />

地球椭球体的形状和大小取决于a、b、f。因此，a、b、f被称为地球椭球体的三要素。





#### 大地基准面(Datum)

地球椭球体提供地球形状的**几何数学模型**，大地基准面确定椭球体**如何放置在真实地球上**（本质上就是规定选用的参考椭球体如何在真实地球上摆放/对齐）

在GIS中大地基准面指目前参考椭球与WGS84椭球的相对位置关系(3个平移、3个旋转、1个缩放，7参数)。



大地基准面:

当一个旋转椭球体的形状与地球相近时，基准面用于定义旋转椭球体相对于地心的位置，是人为确定的。



地心基准面

在过去的多年中，卫星数据为测地学家提供了新的测量结果，用于定义与地球最吻合的、坐标与地球质心相关联的旋转椭球体。地球中心(或地心)基准面使用地球的质心作为原点。最新开发的并且使用最广泛的基准是**WGS1984**。它被用作在世界范围内进行定位测量的框架。



区域基准面

局域基准面是在特定区域内与地球表面极为吻合的旋转椭球体。旋转椭球体表面上的点与地球表面上的特定位置相匹配。该点也被称作基准面的原点。原点的坐标是固定的，所有其他点由其计算获得。



注意:椭球面和地球肯定不是完全贴合的，因而，即使用同一个椭球面，不同的地区由于关心的位置不同，需要最大限度的贴合自己的那一部分，因而大地基准面就会不同。





<u>地理坐标系（Geographic Coordinate System）是基于大地基准面定义的</u>

基于区域基准面定义的本地地理坐标系，如图所示：当前区域基准面（红色虚线）与左上部分地球表面最为贴合

![5](./assets/5.png)



#### 经纬度

通常，经度和纬度值以十进制度为单位，或以度、分和秒(DMS)为单位进行测量。

纬度值相对于赤道进行测量，其范围是-90(南极点)到+90(北极点)。

经度值相对于本初子午线进行测量。其范围是-180(向西行进时)到180(向东行进时)。





#### 投影坐标系 (Projected Coordinate Systems, PCS)

投影坐标系始终基于地理坐标系，而后者则是基于球体或旋转椭球体的。

GCS + 投影算法 = PCS

投影坐标系(Pcs)在二维平面中进行定义，坐标单位是米、千米等。在投影坐标系中，通过格网上的x,y坐标来标识位置，其原点位于格网中心。每个位置均具有两个值，这两个值是相对于该中心位置的坐标。一个指定其水平位置，另一个指定其垂直位置

在地球椭球面和平面之间建立点与点之间函数关系的数学方法，称为地图投影。

![6](./assets/6.png)

地图投影的变形具体表现:

- 长度(距离)变形
- 角度(形状)变形
- 面积变形

![7](./assets/7.png)

等角投影:投影面上某点的任意两方向线夹角与椭球面上相应两线段夹角相等。
等积投影:投影面与椭球面上相应区域的面积相等。
等距投影:在特定方向上没有长度变形的任意投影。
壬意投影:投影图上，长度、面积和角度都有变形，既不等角又不等积。



##### 高斯一克格投影(Gauss-Kruger)

也称为横轴墨卡托(Iransverse_Mercator)，横轴等角切椭圆柱投影,有以下三个特点:

- 中央经线和赤道投影为互相垂直的直线，且为投影的对称轴;
- 具有等角投影的性质;
- 中央经线投影后保持长度不变，中央经线长度比为1。

![8](./assets/8.png)

假想有一个椭圆柱面横套在地球椭球体外面，并与某一条子午线(此子午线称为中央子午线或轴子午线)相切，椭圆柱的中心轴通过椭球体中心，然后用一定投影方法，将中央子午线两侧各一定经差范围内的地区投影到椭圆柱面上，再将此柱面展开即成为投影面。

![9](./assets/9.png)





高斯平面直角坐标系(与常说的投影平面坐标系XY是反的!!!)

在投影面上，中央子午线和赤道的投影都是直线，并且以中央子午线和赤道的交点0作为坐标原点，以中央子午线的投影为横坐标x轴，以赤道的投影为纵坐标y轴。

也就是经线（竖着的）是x轴，赤道（横着的）是y轴

![10](./assets/10.png)



##### 兰勃特(Lambert)投影&阿尔伯斯(Albers)投影

在我国小于1:50万的比例尺(1:100万) 一般采用Lambert或者Albers投影。 我国1:100万地形图采用Lambert投影(等角割圆锥投影)，我国大部分省份地图或者多数这一比例尺的的地图也多采用Lambert投影和属于同一投影系统的阿尔伯斯(Albers)投影(正轴等积割圆锥投影)。在处理1:100万、1:400万全国的数据时经常采用AIbers正轴等积割圆锥投影。

![11](./assets/11.png)





### 模型参考系(glTF)

#### 映射

假如以我们的视角看blender显示模型(屏幕前)

- X轴：左右方向，+X向右，-X向左(左手边)
- Y轴：前后方向，+Y向前，-Y向后(距离屏幕更近)
- Z轴：上下方向，+Z向上，-Z向下

![image-20260123164128112](./assets/image-20260123164128112-1769211635948-10.png)



blender导出时会默认选择+Y向上,**这里我没有选择+Y向上导出**，避免对后面的测试产生影响，如果勾选+Y向上导出，模型会并不会自动旋转90度，可以理解为y轴和z轴互换了一下标签，我会在文章的最后说明

![image-20260123164240115](./assets/image-20260123164240115-1769211642452-12.png)

glTF标准：

https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html

章节：3.4. Coordinate System and Units

GLTF 使用的是右手坐标系。GLTF 将 +Y 定义为向上方向，+Z 定义为向前方向，-X 定义为向右方向；GLTF 资产的正面朝向 +Z 方向。也就是**Y-up 坐标系**

```
      +Y (上)
       ↑
       |
       |
       +------→ +Z (前)
      /
     /
   +X (右)
```

所有直线距离的单位都是米。

所有角度均以弧度为单位。

顺时针旋转是向右转。

所以对于glTF，它会如此定义模型的方向

![image-20260123164425400](./assets/image-20260123164425400-1769211665302-14.png)



在地球表面上的一点，Cesium常用的默认坐标系为：ENU(东-北-天)：

- X轴：东向，+X向东，-X向西（经度增加的方向）roll,横滚角绕其旋转
- Y轴：北向，+Y向北，-Y向南（纬度增加的方向）pitch,俯仰角绕其旋转
- Z轴：天向，+Z向上，-Z向下（垂直于地球表面，远离地心的方向）heading,航向角绕其旋转

```
      +Z (天)
       ↑
       |
       |
       +------→ +Y (北)
      /
     /
   +X (东)
```

当glTF模型加载到Cesium中时，Cesium会将 glTF的 **Y-up 坐标系**转换为 ENU 坐标系：

- glTF 的 X 轴（+X 向右）-> Cesium 的 ENU 的 Y 轴（+Y 向北）
- glTF 的 Y 轴（+Y 向上）-> Cesium 的 ENU 的 Z 轴（+Z 向上）
- glTF 的 Z 轴（+Z 向前）-> Cesium 的 ENU 的 X 轴（+X 向东）

```js
model: {
    uri:'src/data/missileDF(未添加+Y向上).glb',
    scale: 5000.0,
    minimumPixelSize: 64,
    imageBasedLightingFactor: new Cesium.Cartesian2(1.0, 1.0),
    luminanceAtZenith: 0.8
  }
```

![image-20260123164628250](./assets/image-20260123164628250-1769211697932-16.png)



做转换的cesium源码地址

https://github.com/CesiumGS/cesium/blob/4c2f243ed7b7ba6a5a2ae3d452a569a52be78224/packages/engine/Source/Scene/Model/ModelUtility.js#L214-L312

```js
对应的函数（可自行去对应页面搜索）
ModelUtility.getAxisCorrectionMatrix = function (upAxis, forwardAxis, result) {
  result = Matrix4.clone(Matrix4.IDENTITY, result);

  if (upAxis === Axis.Y) {
    result = Matrix4.clone(Axis.Y_UP_TO_Z_UP, result);
  } else if (upAxis === Axis.X) {
    result = Matrix4.clone(Axis.X_UP_TO_Z_UP, result);
  }

  if (forwardAxis === Axis.Z) {
    // glTF 2.0 has a Z-forward convention that must be adapted here to X-forward.
    result = Matrix4.multiplyTransformation(result, Axis.Z_UP_TO_X_UP, result);
  }

  return result;
};
```

航向角 heading 变化，确实绕着天轴转

<img src="./assets/image-20260123165916234-1769211839196-23.png" alt="image-20260123165916234" style="zoom:80%;" />

俯仰角 pitch 变化，确实绕着右轴转（指北的轴）

<img src="./assets/image-20260123165533158-1769211803433-19.png" alt="image-20260123165533158" style="zoom:80%;" />

横滚角 roll 变化，确实绕着前轴转（指东的轴）

<img src="./assets/image-20260123165715762-1769211816435-21.png" alt="image-20260123165715762" style="zoom:80%;" />



会发现：

如果以我们期望的方向来看（弹头朝向前方，弹翼为左右方向，尾翼为上下方向）

- 调整航向角 heading 结果发现在调整俯仰角 pitch
- 调整俯仰角 pitch 结果发现在调整横滚角 roll
- 调整横滚角 roll 结果发现在调整航向角 heading



#### 总结

```
如果没有勾选+Y向上导出，blender到cesium的ENU坐标系转换关系为：
blender X轴（+X向右） -> Cesium ENU Y轴（+Y向北）
blender Z轴（+Z向上） -> Cesium ENU X轴（+X向东）
blender Y轴（+Y向前） -> Cesium ENU Z轴（+Z向上）
```

所以要让模型姿态符合预期:

- 假设原来的次序为：`new Cesium.HeadingPitchRoll(heading, pitch, roll)`;
- 如果勾选了+Y向上导出，只需要将cesium中的pitch和roll参数互换即可，`new Cesium.HeadingPitchRoll(heading, roll, pitch)`;
- 如果没有勾选+Y向上导出，要将heading、pitch、roll参数循环替换位置即可，`new Cesium.HeadingPitchRoll(pitch, roll, heading)`;



#### 是否勾选+Y向上的区别

在three editor中查看（gltf标准、blender还是three.js，都是蓝z，绿y，红x）

![image-20260123170150787](./assets/image-20260123170150787-1769211865519-25.png)

选择+Y向上，明显看出绿色y轴朝上

![image-20260123170612088](./assets/image-20260123170612088-1769211878865-29.png)



未选择+Y向上，明显看出蓝z向上

![image-20260123170556086](./assets/image-20260123170556086-1769211871951-27.png)



仅标签互换，模型并未随之旋转











### 向量计算

`Cartesian3.normalize(cartesian, result)`

- **作用**：向量规范化，将向量转换成单位向量（长度为1）
- 参数：
  - `cartesian`：输入的向量
  - `result`：输出结果，如果传入会修改该对象，否则创建新对象
- **用途**：当你只关心方向而不关心向量的大小时使用
- **例子**：`Cartesian3.normalize(new Cartesian3(3, 4, 0), new Cartesian3())` 得到 `(0. 6, 0.8, 0)`

$$
\hat{v} = \frac{\vec{v}}{|\vec{v}|} = \frac{\vec{v}}{\sqrt{v_x^2 + v_y^2 + v_z^2}}
$$



`Cartesian3.subtract(left, right, result)`

- **作用**：向量减法，计算两个向量的差
- 参数：
  - `left`：被减向量
  - `right`：减去的向量
  - `result`：存储结果的向量
- **用途**：计算两点间的方向向量（从右点指向左点）
- **例子**：如果A点是(10,5,0)，B点是(3,2,0)，`subtract(A, B)` 得到从B指向A的向量(7,3,0)



`Cartesian3.add(left, right, result) → Cartesian3`

- **作用**：向量加法
- 参数：
  - `left`：第一个向量
  - `right`：第二个向量
  - `result`：存储结果的向量
- **用途**：在第一个向量的末端沿着第二个向量的方向前进，得到最终的末端点

$$
\vec{dirEnd} = \vec{p} + \vec{d} = \begin{pmatrix} p_x + d_x \\ p_y + d_y \\ p_z + d_z \end{pmatrix}
$$



`Cartesian3.cross(left, right, result)`

- **作用**：叉乘（外积），计算两个向量的叉积
- 参数：
  - `left`：第一个向量
  - `right`：第二个向量
  - `result`：存储结果的向量
- **用途**：得到同时垂直于两个向量的向量（常用于计算法向量）
- **例子**：如果你有两条边，叉乘可以得到垂直于这个平面的法向量



`Cartesian3.multiplyByScalar(cartesian, scalar, result)`

- **作用**：向量数乘，用标量（数字）乘以向量
- 参数：
  - `cartesian`：输入向量
  - `scalar`：标量值（普通数字）
  - `result`：存储结果的向量
- **用途**：缩放向量的大小，不改变cartesian向量方向的情况下改变其长度，除非标量为负值
- **例子**：`multiplyByScalar(new Cartesian3(2, 3, 1), 2)` 得到 `(4, 6, 2)`

$$
\vec{result} = dirLen \cdot \hat{v} = \begin{pmatrix} dirLen \cdot v_x \\ dirLen \cdot v_y \\ dirLen \cdot v_z \end{pmatrix}
$$





#### 方向向量+长度计算终点位置

```js
// 创建一个固定的调试相机
const debugCamera = new Cesium.Camera(viewer.scene);

// 设置调试相机的位置，经 纬 高
const cameraPosition = Cesium.Cartesian3.fromDegrees(116.3975, 39.9085, 10000       );

// 设置相机朝向（俯视地面）
const cameraDirection = new Cesium.Cartesian3(0, 0, -1);  // 向下看
const cameraUp = new Cesium.Cartesian3(0, 1, 0);          // 上方向

// 配置调试相机
debugCamera.position = cameraPosition;
debugCamera.direction = cameraDirection; 
debugCamera.up = cameraUp; 


// === 可视化 direction / up 向量===
const dirLen = 3000; // 方向线长度（米）
const upLen = 2000;  // 上方向线长度（米）


// 通过方向向量和长度计算线段终点位置
const dirEnd = Cesium.Cartesian3.add(
  cameraPosition,
  Cesium.Cartesian3.multiplyByScalar(
    Cesium.Cartesian3.normalize(cameraDirection, new Cesium.Cartesian3()),
    dirLen,
    new Cesium.Cartesian3()
  ),
  new Cesium.Cartesian3()
);

const upEnd = Cesium.Cartesian3.add(
  cameraPosition,
  Cesium.Cartesian3.multiplyByScalar(
    Cesium.Cartesian3.normalize(cameraUp, new Cesium.Cartesian3()),
    upLen,
    new Cesium.Cartesian3()
  ),
  new Cesium.Cartesian3()
);

// 看向方向（绿色线）
viewer.entities.add({
  position: dirEnd,
  polyline: {
    positions: [cameraPosition, dirEnd],
    width: 4,
    material: Cesium.Color.LIME
  },
  label: {
    text: "direction",
    font: "14px Microsoft YaHei",
    fillColor: Cesium.Color.LIME,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
});

// 上方向（黄色线）
viewer.entities.add({
  position: upEnd,
  polyline: {
    positions: [cameraPosition, upEnd],
    width: 4,
    material: Cesium.Color.YELLOW
  },
  label: {
    text: "up",
    font: "14px sans-serif",
    fillColor: Cesium.Color.YELLOW,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
});

// 相机视锥体代码...
```

![image-20260303150308020](./assets/image-20260303150308020.png)



### 射线与求交

new Cesium.Ray(origin, direction)

- **作用**：创建一条射线对象

- 参数：

  - `origin`：射线起点，类型为 `Cartesian3`（3D坐标）
  - `direction`：射线方向，类型为 `Cartesian3`（通常应为单位向量）

- **返回**：Ray 对象，包含起点和方向信息

- 例子：

  ```javascript
  let origin = new Cesium.Cartesian3(0, 0, 0);
  let direction = new Cesium.Cartesian3(1, 0, 0); // 指向X轴正方向
  let ray = new Cesium.Ray(origin, direction);
  ```



IntersectionTests. rayEllipsoid(ray, ellipsoid) → Interval

- **作用**：计算射线与椭球体（地球）的交点

- 参数：

  - `ray`：射线对象（由 Ray 构造函数创建）
  - `ellipsoid`：椭球体对象，通常使用 `Cesium.Ellipsoid. WGS84`（地球）

- 返回`Interval`对象，包含两个交点参数：

  - `start`：进入椭球体的参数 t
  - `stop`：离开椭球体的参数 t
  - 如果无交点，返回 `undefined`

- 例子：

  ```javascript
  let interval = Cesium.IntersectionTests.rayEllipsoid(ray, Cesium.Ellipsoid. WGS84);
  if (Cesium.defined(interval)) {
    console.log('进入点参数:', interval.start);
    console.log('离开点参数:', interval.stop);
  }
  ```



Ray.getPoint(ray, t, result)

- **作用**：根据射线参数方程求出射线上的点坐标

- 参数：

  - `ray`：射线对象
  - t：参数（标量值）
    - `t = 0` → 射线起点
    - `t > 0` → 沿着方向向前
    - `t` 越大，距离起点越远
  - `result`：存储结果的 `Cartesian3` 对象（可选）

- **计算公式**：`r(t) = origin + t × direction`

- **返回**：射线上对应参数 t 的点坐标

- 例子：

  ```javascript
  let ray = new Cesium.Ray(
    new Cesium.Cartesian3(0, 0, 0),
    new Cesium.Cartesian3(1, 0, 0)
  );
  let point = Cesium.Ray.getPoint(ray, 5);
  // 得到 (5, 0, 0) - 从起点沿方向移动5个单位
  ```





求取与地球交点

```js
// 1. 创建射线
let ray = new Cesium.Ray(
  new Cesium. Cartesian3(0, 0, 6400000),  // 起点（地球上方）
  Cesium.Cartesian3.normalize(
    new Cesium. Cartesian3(0, 0, -1)      // 方向（向下）
  )
);

// 2. 检测与地球的交点
let interval = Cesium.IntersectionTests.rayEllipsoid(ray, Cesium.Ellipsoid.WGS84);

if (Cesium.defined(interval)) {
  // 3. 获取进入和离开地球的点
  let entryPoint = Cesium.Ray.getPoint(ray, interval.start);
  let exitPoint = Cesium. Ray.getPoint(ray, interval.stop);
  console.log('进入点:', entryPoint);
  console.log('离开点:', exitPoint);
}
```



### 待总结

```
Cesium.Cartesian3.magnitude   计算笛卡尔坐标的模长
```





```
计算包围球 (可选)	相机快速聚焦	BoundingSphere.fromPoints(positions)
相机飞行 (可选)	查看结果	camera.flyToBoundingSphere(bs,{ offset: new HeadingPitchRange(...) })

BoundingSphere
用于快速近似包围范围的球体（center + radius）。可由点集计算：BoundingSphere.fromPoints(positions)，常用于相机定位或视锥裁剪。
```



| Matrix4.fromTranslation(vec3)     | 构造 4x4 平移矩阵                       | `const translation = Cesium.Matrix4.fromTranslation(new Cartesian3(tx,ty,tz))` |
| --------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| Matrix3.fromRotationX(angle)等    | 构造单轴 3x3 旋转矩阵                   | `const rotX3 = Cesium.Matrix3.fromRotationX(rx)`             |
| Matrix4.fromRotation(matrix3)     | 将 3x3 旋转扩展为 4x4                   | `const rotX4 = Cesium.Matrix4.fromRotation(rotX3)`           |
| Matrix4.fromUniformScale(scale)   | 构造统一缩放矩阵                        | `const scaleMatrix = Cesium.Matrix4.fromUniformScale(s)`     |
| Matrix4.multiply(left,right,left) | 就地累乘组合变换                        | `Matrix4.multiply(current,next,current)`                     |
| tileset._root.transform           | 应用整体变换到 3D Tiles 根（局部→世界） | `tileset._root.transform = modelMatrix`                      |



```
| `Cesium.Transforms.eastNorthUpToFixedFrame(origin)`          | `const enu = Cesium.Transforms.eastNorthUpToFixedFrame(Cartesian3.fromDegrees(...))` | 以某世界点为原点，构建局部东-北-上 (East-North-Up, ENU) 到地心固定坐标系的 4x4 仿射矩阵。用于在该局部系里做相对位移。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Cesium.Matrix4.multiplyByPoint(matrix, localPoint, result)` | `Cesium.Matrix4.multiplyByPoint(enu, new Cesium.Cartesian3(200,0,0), ...)` | 将局部坐标系中的点（例如向东 200 米）转换为世界坐标。你用它批量把局部点数组映射成全局位置。 |
| 局部数组批量转换 (map + multiplyByPoint)                     | `pipePositions.map(item => Cesium.Matrix4.multiplyByPoint(enu, item, ...))` | 把一组局部路径点转成世界路径点。                             |
| 多边形局部点批量转换                                         | `boundary.map(item => Cesium.Matrix4.multiplyByPoint(enu, new Cartesian3(item[0], item[1], 0), ...))` | 以 ENU 为基础，用局部二维偏移生成多边形世界顶点。            |
```



世界坐标转屏幕坐标  `Cesium.SceneTransforms.worldToWindowCoordinates(scene, position)`











## viewer

核心对象，后面所学的东西都挂在这个对象上



### 示意结构

字面量形式展现

```javascript
const viewer = {
  // 创建时传入的 DOM 容器、配置选项
  container: HTMLElement,
  canvas: HTMLCanvasElement,

  // 全局的实体集合（与各 CustomDataSource 里的实体集合不同）
  entities: EntityCollection {
    // 私有存储（示意）
    _entities: [ /* Entity */ ],
    // 额外内部索引（不同版本名字可能不同）
    // _entitiesById: { id: Entity }
    length: Number,
    add(entityOptions) { /* returns Entity */ },
    remove(entity) {},
    getById(id) { /* returns Entity or undefined */ },
    // ...其他事件与方法
  },

  dataSources: DataSourceCollection {
    _dataSources: [
      CustomDataSource {
        name: 'myDS',          // 公共名称 (getByName 用这个)
        clock: DataSourceClock,
        show: true,
        entities: EntityCollection {
          _entities: [
            Entity {
              id: 'e1',        // 公共 id
              // position / billboard / point / polyline / polygon ...
              // _id（如果看到）为内部映射或冗余引用，不用依赖
            },
            Entity { id: 'e2', /* ... */ }
          ],
          length: 2,
          add(opts) { /* -> Entity */ },
          getById('e1') { /* -> Entity */ },
          remove(entity) {},
          // ...事件/内部索引
        },
        // 该数据源内部可能还有 _entityCollection（你看到的 entities 是个 getter）
        // 以及各种特定数据源实现 (CZMLDataSource, GeoJsonDataSource 等)
        // ...
      },
      /* 其它 DataSource，如:
         CzmlDataSource, KmlDataSource, GeoJsonDataSource, ImageryDataSource, etc. */
    ],
    length: Number,
    add(ds) { /* -> ds */ },
    remove(ds) {},
    get(index) { /* -> ds */ },
    getByName(name) { /* -> Array<ds> */ },
    // ...
  },

  scene: {
    camera: Camera { position, direction, up, ... },
    globe: Globe { terrainProvider, imageryLayers (旧版本不在 globe 上), ... },
    primitives: PrimitiveCollection {
      _primitives: [
        Primitive { /* e.g. geometryInstances, appearance, asynchronous state, ... */ },
        GroundPrimitive { /* ... */ },
        ClassificationPrimitive { /* ... */ },
        PolylineCollection { /* ... (不是 Primitive，但也会在集合中) */ }
      ],
      length: Number,
      add(primitive) { /* -> primitive */ },
      remove(primitive) {},
      removeAll() {},
      contains(primitive) { /* bool */ },
      // ...
    },
    imageryLayers: ImageryLayerCollection {
      _layers: [ ImageryLayer, ImageryLayer, ... ],
      add(layer) {},
      remove(layer) {},
      get(index) {},
      // ...
    },
    // 还有：postProcessStages, passes, frameState, etc.
  },

  clock: Clock { currentTime, startTime, stopTime, multiplier, shouldAnimate, ... },
  timeline: Timeline,   // 如果传入创建
  animation: Animation, // 如果传入创建
  cesiumWidget: CesiumWidget { /* 内部封装 */ },

  // 事件/小工具集合
  screenSpaceEventHandler: ScreenSpaceEventHandler { /* ... */ },

  // 销毁方法
  destroy() {},
  isDestroyed() {},
  // ...
};
```





这个对象就凸显了cesium的设计模式，集合模式

Cesium 中很多“集合（Collection）”类都采用类似的内部组织形式：

- 外部公开：add/remove/getById/getByName/length/contains
- 内部私有：_dataSources、_entities、_primitives、_layers 等数组或 Map/字典结构
- 访问元素时请使用公开方法，不要直接读写下划线字段

典型集合：

- DataSourceCollection (viewer.dataSources)
- EntityCollection (viewer.entities 以及 CustomDataSource.entities)
- PrimitiveCollection (viewer.scene.primitives)
- ImageryLayerCollection (viewer.imageryLayers)
- Cesium3DTilesetCollection（新版本可用时）
- PostProcessStageCollection 等





## 相机

cesium中看到的画面变化，大多是viewer.camera的变化

- 位置：destination/position
- 朝向（heading/pitch/roll 或 direction/up）
- 参考系：以相机位置为原点，建立一个"东北天（ENU）局部参考系"，heading/pitch/roll 是在这个ENU系里的旋转偏移量

![image-20260326201049198](./assets/image-20260326201049198.png)





### 相机参数

- `direction`：相机“看向”的方向（屏幕朝前）
- `up`：屏幕的“上”方向
- `right`：屏幕的“右”方向（由 `right = direction × up` 得到）

上面三个都是 **ECEF全局系** 下的单位向量值，一般来说：只要给定 `direction` + `up`（并保证它们不共线），相机姿态就完全确定了

```js
viewer.camera.directionWC // 世界坐标系下相机看向的方向向量
viewer.camera.upWC // 世界坐标系下相机的上方向向量
viewer.camera.rightWC // 世界坐标系下相机的右方向向量
这些向量都是归一化的单位向量，表示方向，没有长度信息。


directionWC：Cartesian3 {x: 0.500538710465422, y: -0.776177205372463, z: 0.38341875956427285}
 
upWC：Cartesian3 {x: -0.12565963935348018, y: 0.373062907860897, z: 0.9192571576092978}
 
rightWC：Cartesian3 {x: -0.8565457689832843, y: -0.5083040553040723, z: 0.08919827912167808}
```



相机位置

```js
viewer.camera.position 相机在**局部参考系**中的坐标（即 offset 本身）
viewer.camera.positionWC 相机在**世界坐标系 (ECEF)** 中的真实位置  
```



而 `heading / pitch / roll` 是对同一姿态的**欧拉角表达**（相对于 相机位置 所在的 "东北天（ENU）局部参考系）：

![image-20260318144617360](./assets/image-20260318144617360.png)

```
heading（航向）：绕 初始 ENU 的 U（Up/天）轴旋转，正方向通常朝东增大。

pitch（俯仰）：绕 初始 ENU 的 E（East/东）轴旋转，控制抬头/低头。 Cesium 里常见语义是：pitch = -90° 更接近“向下看地面”，pitch = 0° 更接近平视。

roll（翻滚）：绕 初始 direction（视线轴）旋转，控制画面“歪头”程度。

加上初始是因为，相机的旋转本质上是ENU参考系的旋转
```



roll绕视线轴旋转

<img src="./assets/screenshots-1774527698926-3.gif" alt="screenshots" style="zoom: 67%;" />

pitch绕E轴旋转

<img src="./assets/screenshots-1774527788348-5.gif" alt="screenshots" style="zoom:67%;" />

heading绕天轴旋转

<img src="./assets/screenshots-1774527942732-7.gif" alt="screenshots" style="zoom:67%;" />





欧拉角

对于在三维空间里的一个参考系，任何坐标系的取向，都可以用三个欧拉角来表现。参考系（固定系）又称为实验室参考系，是静止不动的。而坐标系（固定系）则固定在刚体上，随着刚体的旋转而旋转。

参考下图，以xyz为参考系的参考轴，XYZ为物体自身的坐标轴，xoy平面与XOY平面的相交线为交点线，用N表示。zxz顺规的欧拉角可以静态的这样定义：

- α（进动角)  是x-轴与交点线的夹角，
- β（章动角）是z-轴与Z-轴的夹角，
- γ（自旋角）是交点线与X-轴的夹角。

<img src="./assets/Eulerangles.svg.png" alt="Eulerangles.svg" style="zoom:25%;" />



视锥体参数

```ts
viewer.camera.frustum // 摄像机截头锥体对象，描述了相机的视锥体参数
在 Cesium 的 3D 视图（SceneMode.SCENE3D）里，Camera 默认 frustum 是 PerspectiveFrustum()，也就是 透视投影的截头锥体
const frustum = viewer.camera.frustum; // PerspectiveFrustum
frustum.fov         // 视场角（弧度）
frustum.near        // 近裁剪面距离（米），太小会导致深度精度问题
frustum.far         // 远裁剪面距离（米）
frustum.aspectRatio // 宽高比（通常跟 canvas 宽高一致）

aspectRatio 默认值来自 canvas 的宽高比（viewer.scene.drawingBufferWidth / drawingBufferHeight），手动设置后会与实际渲染比例解耦，视锥体形状会变形，但这正好可以用于模拟非标准传感器。
```

<img src="./assets/视锥体-1795574664450-1.png" alt="视锥体" style="zoom: 67%;" />





### 相机参数工具封装

```ts
// 封装一个工具：在画面上显示相机参数（经纬度/高度、heading/pitch/roll、direction/up 等）

import * as Cesium from 'cesium';

export type CameraHudOptions = {
  container?: HTMLElement;
  precision?: number; // 小数点位数，默认 6
  updataHz?: number; // 更新频率，默认 10 次/秒
  showVectors?: boolean; // 是否显示方向向量，默认 true
}

export function createCameraHud(viewer: Cesium.Viewer, options: CameraHudOptions = {}) {
  const precision = options.precision ?? 6;
  const updateHz = options.updataHz ?? 10;
  const showVectors = options.showVectors ?? true;

  const host = options.container ?? viewer.container; // 默认放在 viewer 容器内

  const el = document.createElement('div');
  el.style.position = 'absolute';
  el.style.left = "8px"
  el.style.top = "8px"
  el.style.padding = "8px 10px"
  el.style.background = "rgba(0,0,0,0.55)"
  el.style.color = "#fff"
  el.style.font = "12px/1.35 ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace"
  el.style.whiteSpace = "pre"
  el.style.zIndex = "999"
  el.style.pointerEvents = "none"
  el.style.borderRadius = "4px"
  host.appendChild(el)

  const tmpCarto = new Cesium.Cartographic();

  const rad2deg = (r: number) => r * 180 / Math.PI; // 弧度转角度
  const fmt = (n: number) => Number.isFinite(n) ? n.toFixed(precision) : String(n); // 格式化数字

  let timer: number | undefined;

  /**
   * 更新相机信息显示
   */
  function update() {
    const camera = viewer.camera;
    // 把相机世界坐标转换为经纬高，写入 tmpCarto
    Cesium.Cartographic.fromCartesian(
      camera.positionWC,
      Cesium.Ellipsoid.WGS84,
      tmpCarto
    );
    const lon = rad2deg(tmpCarto.longitude);
    const lat = rad2deg(tmpCarto.latitude);
    const height = tmpCarto.height;

    const heading = rad2deg(camera.heading);
    const pitch = rad2deg(camera.pitch);
    const roll = rad2deg(camera.roll);

    const dir = camera.directionWC // 世界坐标系下的方向向量
    const up = camera.upWC; // 世界坐标系下的上方向向量
    const right = camera.rightWC; // 世界坐标系下的右方向向量

    const lines: string[] = [];
    lines.push(`Camera`)
    lines.push(`lon/lat/h：   经度: ${fmt(lon)}, 纬度: ${fmt(lat)}, 高度: ${fmt(height)} m`)
    lines.push(`heading/pitch/roll，航向: ${fmt(heading)}°, 俯仰: ${fmt(pitch)}°, 横滚: ${fmt(roll)}°`)

    if (showVectors) {
      lines.push(`directionWC(视向): (${fmt(dir.x)}, ${fmt(dir.y)}, ${fmt(dir.z)})`)
      lines.push(`upWC(上方向):      (${fmt(up.x)}, ${fmt(up.y)}, ${fmt(up.z)})`)
      lines.push(`rightWC(右方向):   (${fmt(right.x)}, ${fmt(right.y)}, ${fmt(right.z)})`)
    }

    const frustum: any = camera.frustum // 摄像机截头锥体
    if (frustum) {
      const near = frustum.near
      const far = frustum.far
      const fovy = frustum.fov ?? frustum.fovy // 有可能是 fov 或 fovy，fov 是垂直视场角，fovy 是水平视场角
      if (near != null && far != null) lines.push(`frustum near/far: ${fmt(near)}, ${fmt(far)}`)
      if (fovy != null) lines.push(`fovY: ${fmt(rad2deg(fovy))}°`)
    }

    el.textContent = lines.join('\n');
  }

  /**
   * 启动/停止 HUD 更新
   */
  const start = () => {
    stop();
    update();
    timer = window.setInterval(update, 1000 / updateHz);
  }

  const stop = () => {
    if (timer != null) window.clearInterval(timer);
    timer = undefined;
  }

  start();

  return {
    element: el,
    start,
    stop,
    destroy() {
      stop();
      el.remove();
    },
  }
}

// 使用：createCameraHud(viewer, { updateHz: 10, precision: 6, showVectors: true })
```

![image-20260106211552596](./assets/image-20260106211552596.png)





### 跳转

瞬时跳转：

```ts
viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(116.391, 39.907, 5000),
  orientation: {
    heading: Cesium.Math.toRadians(0),
    pitch: Cesium.Math.toRadians(-45),
    roll: 0,
  },
});
```



平滑飞行

```ts
viewer.camera.flyTo({
  destination: Cesium.Cartesian3.fromDegrees(116.391, 39.907, 8000),
  orientation: {
    heading: Cesium.Math.toRadians(20),
    pitch: Cesium.Math.toRadians(-30),
    roll: 0,
  },
  duration: 1.2,
});
```







### 设置相机姿态

黄线为视点和相机所处位置的连线

![image-20260326210159244](./assets/image-20260326210159244.png)



设置相机姿态有两种设置方式：

1、直接指定位姿：设置相机的世界坐标位置，再用heading\pitch\roll描述其在当前位置局部ENU系下的朝向，适合自由视角。

2、lookAt约束视点：只需给定相机看向目标点（世界坐标）再通过相对于目标点处的偏移量（目标点处ENU系下的Cartesian3，或球坐标形式的 Heading）设置相机位置，就能实现相机在合适的位置锁定看向目标点——即“看哪里”与“从哪里看”分离控制。对于运镜，只需在关键帧上指定不同的目标点与偏移量，即可实现环绕、推拉、跟随等镜头动作。

```js
viewer.camera.lookAt(target, offset)
使用目标和偏移量设置摄像机的位置和方向。目标必须以世界坐标系表示。偏移量可以是笛卡尔坐标，也可以是以目标为中心的本地东北向上坐标系中的航向/俯仰角/距离。如果偏移量为笛卡尔坐标，则表示相对于由变换矩阵定义的参考系中心的偏移量。如果偏移量为航向/俯仰/范围，则航向角和俯仰角在由变换矩阵定义的参考系中定义。航向角是从 y 轴开始，向 x 轴方向增加的夹角。俯仰角是相对于 xy 平面的旋转角度。正俯仰角表示位于平面下方。负俯仰角位于平面之上。距离是指距中心点的距离。在2D场景中，必须采用俯视视角。摄像机将位于目标上方并向下拍摄。目标上方的高度即为偏移量的大小。航向将根据偏移量确定。如果无法根据偏移量确定航向，则航向默认为正北。
参数；
  target - cartesian3 - 以世界坐标系表示的目标位置。
  offset - cartesian3 或 HeadingPitchRange - 定义偏移量。可以是一个 Cartesian3 对象，表示相对于由变换矩阵定义的参考系中心的偏移量；也可以是一个 HeadingPitchRange 对象，表示相对于目标位置的航向/俯仰/距离。



new Cesium.HeadingPitchRange(heading, pitch, range)
定义本地坐标系中的航向角、俯仰角和范围。航向角是从本地东方向开始的旋转角，正角表示向南方向。俯仰角是从本地 xy 平面开始的旋转角。正俯仰角表示位于平面之上。负俯仰角表示位于平面下方。范围是指距坐标系中心的距离。
参数：
  heading - Number - 航向角，以弧度为单位。
  pitch - Number - 俯仰角，以弧度为单位。
  range - Number - 距离，单位为米。


```

lookAt锁定目标效果

<img src="./assets/screenshots-1774581380939-1.gif" alt="screenshots" style="zoom:50%;" />



`lookAt(target, offset)`：Cesium **自动**以 `target` 点为原点，构造该点处的 **ENU 系**作为参考系,如下图，当三个偏移值几乎都为0时，相机位置几乎要和看向的目标点位置重合了

![image-20260327110136765](./assets/image-20260327110136765.png)

`lookAtTransform(transform, offset)`：可以**手动提供**一个任意的变换矩阵 `transform`（`Matrix4`）来定义参考系

```js
viewer.camera.lookAtTransform(transform, offset)
使用目标和变换矩阵设置相机的位置和方向。偏移量可以是笛卡尔坐标，也可以是航向/俯仰/距离。如果偏移量是笛卡尔坐标，则它是相对于变换矩阵定义的参考系中心的偏移量。如果偏移量是航向/俯仰/距离，则航向角和俯仰角是在变换矩阵定义的参考系中定义的。航向角是从 y 轴开始并沿 x 轴方向递增的角度。俯仰角是从 xy 平面开始的旋转角度。正俯仰角表示相机位于平面下方，负俯仰角表示相机位于平面上方。距离是相机到中心的距离
参数：
  transform - Matrix4 - 定义参考系的变换矩阵。
  offset - Cartesian3 或 HeadingPitchRange - 定义偏移量。可以是一个 Cartesian3 对象，表示相对于由变换矩阵定义的参考系中心的偏移量；也可以是一个 HeadingPitchRange 对象，表示相对于目标位置的航向/俯仰/距离。


相比于lookAt的核心价值，当参考系不是简单的ENU,比如绑定到一个运动模型的机体坐标系、轨道坐标系等，就用lookAtTransform传入该坐标系的Matrix4,让相机跟随任意参考系移动
```



```js
这里的第三人称跟拍，相机锁定一个移动目标，但是可以拖动相机位置

// 每一帧：更新参考系，但 offset 使用“当前相机位置”换算得到，这样用户拖动不会被下一帧覆盖
const scratchEnu = new Cesium.Matrix4() // 这个矩阵的用途是作为“参考坐标系”，把目标点位置转换为 ENU 局部坐标
const scratchInv = new Cesium.Matrix4()  // 这个矩阵的用途是作为“转换矩阵的逆矩阵”，把世界坐标转换到 ENU 局部坐标
const scratchOffset = new Cesium.Cartesian3()  // 实时更新的 offset

viewer.clock.onTick.addEventListener(() => {
  const time = viewer.clock.currentTime
  const pos = movingEntity.position?.getValue(time)
  if (!pos) return

  // 目标点 ENU 参考系（世界坐标下的矩阵），赋值给scratchEnu
  Cesium.Transforms.eastNorthUpToFixedFrame(pos, Cesium.Ellipsoid.WGS84, scratchEnu)
  
  // inverseTransformation：计算矩阵的逆矩阵，赋值给scratchInv
  Cesium.Matrix4.inverseTransformation(scratchEnu, scratchInv)

  // 把“当前相机世界坐标位置”换算到 ENU 局部坐标里，作为新的 offset，这里就会更新scratchOffset
  Cesium.Matrix4.multiplyByPoint(scratchInv, viewer.camera.positionWC, scratchOffset) 

  // 继续锁定目标点（也就是scratchEnu不变），但 offset 已经更新为用户刚刚调整的位置了
  viewer.camera.lookAtTransform(scratchEnu, scratchOffset)
})
```





### 相机交互控制

| 属性              | 默认值 | 效果                                | 对应操作                             |
| ----------------- | ------ | ----------------------------------- | ------------------------------------ |
| `enableRotate`    | `true` | 控制围绕地球旋转                    | 左键拖拽                             |
| `enableTranslate` | `true` | 控制平移（2D/哥伦布视图下平移地图） | 右键拖拽（3D下由`enableRotate`覆盖） |
| `enableZoom`      | `true` | 控制缩放远近                        | 滚轮 / 右键拖拽                      |
| `enableTilt`      | `true` | 控制倾斜视角（俯仰）                | 中键拖拽 / Ctrl+左键拖拽             |
| `enableLook`      | `true` | 控制原地环顾（第一人称视角转头）    | Shift+左键拖拽                       |
| `enableInputs`    | `true` | **总开关**，关闭后禁用所有鼠标交互  | 全部                                 |



| 属性                  | 说明                             |
| --------------------- | -------------------------------- |
| `minimumZoomDistance` | 最小缩放距离（米），限制拉近程度 |
| `maximumZoomDistance` | 最大缩放距离（米），限制拉远程度 |
| `inertiaZoom`         | 缩放惯性（0~1，0=无惯性）        |
| `inertiaSpin`         | 旋转惯性                         |
| `inertiaTranslate`    | 平移惯性                         |



```js
// ========== 相机交互控制实验 ==========
const ctrl = viewer.scene.screenSpaceCameraController;

// 1. 禁止旋转 —— 左键拖拽无法转动地球
ctrl.enableRotate = false;

// 2. 禁止平移 —— 3D模式下效果不明显（主要影响2D/哥伦布视图）
ctrl.enableTranslate = false;

// 3. 禁止缩放 —— 滚轮和右键拖拽无法缩放
ctrl.enableZoom = false;

// 4. 禁止倾斜 —— 中键拖拽 / Ctrl+左键 无法改变俯仰角
ctrl.enableTilt = false;

// 5. 禁止环顾 —— Shift+左键 无法原地转头
ctrl.enableLook = false;

// 6. 总开关：禁用全部交互
ctrl.enableInputs = false;

// 7. 限制缩放范围（单位：米）
ctrl.minimumZoomDistance = 1000;     // 最近1km
ctrl.maximumZoomDistance = 20000000; // 最远20000km

// 8. 去掉惯性（鼠标松开后立即停止）
ctrl.inertiaZoom = 0;   // 相机变焦惯性设置，也就是滚轮缩放惯性，默认为0.9，设置为0则没有惯性，滚轮缩放后相机立即停止。
ctrl.inertiaSpin = 0; // 相机旋转惯性设置，也就是左键拖动旋转地球后松开鼠标，地球继续旋转的程度，默认为0.9，设置为0则没有惯性，左键拖动旋转地球后松开鼠标，地球立即停止。
ctrl.inertiaTranslate = 0; // 相机平移惯性设置，也就是右键拖动平移后松开鼠标，地球继续平移的程度，默认为0.9，设置为0则没有惯性，右键拖动平移后松开鼠标，地球立即停止。

// 9. 锁定相机到固定位置（完全程序控制相机）
ctrl.enableInputs = false;
viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(116.39, 39.91, 5000000),
  orientation: {
    heading: Cesium.Math.toRadians(0),
    pitch: Cesium.Math.toRadians(-90),
    roll: 0,
  },
});
```





## Enitiy



### 设计哲学



#### 点实体的不同创建方式

字面量写法

```js
viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(116.4, 39.9),
  point: { pixelSize: 10, color: Cesium.Color.RED, outlineWidth: 2 }
});

viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(121.4, 31.2),
  point: { pixelSize: 10, color: Cesium.Color.RED, outlineWidth: 2 }
});

viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(113.2, 23.1),
  point: { pixelSize: 10, color: Cesium.Color.RED, outlineWidth: 2 }
});

// ... 再来 7 个，每个都要复制一遍样式
```



`Graphics` 构造函数写法

提前定义，复用配置

```js
// 用 PointGraphics 提前定义，方便调参和复用
const redPointStyle = new Cesium.PointGraphics({
  pixelSize: 10,
  color: Cesium.Color.RED,
  outlineColor: Cesium.Color.WHITE,
  outlineWidth: 2
});

// 多个实体共用同一套样式
viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(116.4, 39.9),
  point: redPointStyle
});

viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(121.4, 31.2),
  point: redPointStyle
});
```







数据即配置(Data as Configuration)

Cesium 的 `Entity` 系统本质上是一个**声明式 API**——你描述"**想要什么**",而不是"**怎么做**"。

```js
// 你只是在"描述"一个点,而不是在"画"一个点
viewer.entities.add({
  position: ...,
  point: { pixelSize: 10, color: Color.RED }  // 纯数据
});
```

> UI(或场景)= f(数据)

既然是"数据",那 `{ pixelSize: 10 }` 这种纯字面量就应该被天然接受——因为它**最接近数据本身的形态**,可序列化、可从 JSON 加载、可通过网络传输。



#### 渐进复杂度

| 使用层级 | 写法                                          | 适合场景                         |
| -------- | --------------------------------------------- | -------------------------------- |
| **入门** | 字面量 `{ pixelSize: 10 }`                    | 快速上手、静态配置、从 JSON 读取 |
| **进阶** | `new PointGraphics({...})`                    | 复用、类型提示、IDE 友好         |
| **高级** | `point.pixelSize = new CallbackProperty(...)` | 动态属性、时间轴驱动、CZML       |



Property 系统的统一抽象

比如说`PointGraphics` 的每个属性其实都不是"普通值",而是 `Property` 对象

```js
// 你写的
{ pixelSize: 10 }

// Cesium 内部实际变成
new PointGraphics({
  pixelSize: new ConstantProperty(10)  // 被包装成 Property
})
```

好处是**统一了"静态值"和"时变值"**:

```js
// 静态:始终 10
point: { pixelSize: 10 }

// 动态:随时间变化
point: { 
  pixelSize: new SampledProperty(Number)  // 时间轴上采样
}

// 动态:每帧回调
point: { 
  pixelSize: new CallbackProperty(time => Math.sin(time) * 10 + 20)
}
```



**属性槽,可以塞入常量、时变值、回调**——这正是 Cesium 作为"4D GIS 引擎"(空间 + 时间)的核心能力





映射到其它库,你会发现是同一套思想

| 库           | "类实例 \| 选项对象" 的体现                       |
| ------------ | ------------------------------------------------- |
| **Three.js** | `new Material(options)` vs 直接传选项给 loader    |
| **D3**       | `d3.scaleLinear().domain([...])` 链式 vs 配置对象 |
| **Leaflet**  | `L.marker(latlng, options)` options 可省略        |
| **ECharts**  | 整个库就是 `setOption(config)`,纯数据驱动到底     |
| **React**    | `<Comp {...props} />` 字面量 props                |

**共同点**:把"**状态/配置**"和"**行为**"解耦,让配置可以**被数据化、被序列化、被工具链处理**





#### "万能公式"去理解任何前端库

当你遇到一个新库,按这三个问题去问(或问 AI):

它的"核心数据模型"是什么?
(Cesium 是 Entity + Property;React 是 Component + props;Three.js 是 Scene + Object3D)

它是命令式还是声明式?
(命令式 = 一步步指挥;声明式 = 描述目标状态)

它如何处理"动态性"?
(轮询?事件?响应式?Property 系统?时间轴?)

答完这三问,基本就摸到这个库的"骨架"了,剩下的 API 都是骨架上的装饰,用 AI 随查随用即可。





#### 回到具体观察

![image-20260427095636797](./assets/image-20260427095636797.png)

`BillboardGraphics | BillboardGraphics.ConstructorOptions`

这个 TypeScript 类型签名其实就是 Cesium 在明确告诉你:

"我不在乎你传类实例还是字面量,因为我的内部模型是 Property,两种形式最终都会被归一化。你怎么方便怎么来。"

这是一种对用户善意的冗余——牺牲一点点内部复杂度(写个类型判断),换取 API 的友好度和灵活性。









### Entity 系统与 glTF 模型渲染

两个问题

- Entity 是 Cesium 的“高层业务对象”（位置/姿态/模型/标签/轨迹等都是它的属性）

- glb 模型的 model.uri 

比如加载“飞机”等 glb 模型，代码逻辑是：创建 Entity → 给 position（可能是随时间变化）→ 给 model.uri（glb URL）→（可选）给 orientation。



`viewer.entities.add({ position, orientation, model })`

```ts
viewer.entities 是一个实体集合（EntityCollection）。用 add({...}) 会得到一个 Cesium.Entity 实例：add(entity) → Entity，具体可以之前笔记的viewer的字面量对象，调用这个方法会返回一个Entity实例


里面的position属性：
  常量：比如 position: Cartesian3.fromDegrees(...)
  随时间变化的属性：比如 position: SampledPositionProperty()（飞机飞行、空投下落）

  
orientation: new VelocityOrientationProperty(sampled)（由位置变化自动推导朝向）


model: { uri, scale, ... }（模型渲染参数）
  uri: modelUrl：glb 的 URL（字符串）
  scale：模型缩放
  minimumPixelSize：保证模型再远也不会小到看不见（按屏幕像素限制最小大小）
  maximumScale：限制最大放大（避免太近时夸张膨胀）
  
  
Cesium.Math.lerp(p, q, time) → number
计算两个值之间的线性插值
p：起始值
q：结束值
time：0~1 之间的数，0 返回 p，1 返回 q，0.5 返回中间值
```



#### 追踪移动的飞机glb

```ts
// Vite 下 glb 用 ?url 拿到可访问的 URL
import an72pUrl from "./assets/model/an72p.glb?url"








//注册token
Cesium.Ion.defaultAccessToken = import.meta.env.VITE_CESIUM_TOKEN

// ...existing code...

const viewer : Cesium.Viewer = new Cesium.Viewer('cesiumContainer', {
  // ...existing code...
  shouldAnimate: true,
  // ...existing code...
})







// 轨迹：在北京附近飞一个“弧线”（你也可以改成任意经纬度）
const startLLH = { lon: 116.25, lat: 39.85, h: 800 } // 起点
const endLLH   = { lon: 116.85, lat: 40.15, h: 900 } // 终点

const durationSec = 10 // 飞行总时长，秒
const sampleCount = 160 // 采样点数量

const startTime = Cesium.JulianDate.now()
const stopTime = Cesium.JulianDate.addSeconds(startTime, durationSec, new Cesium.JulianDate()) // 结束时间

// 设置 viewer 的时钟
viewer.clock.startTime = startTime.clone()
viewer.clock.stopTime = stopTime.clone()
viewer.clock.currentTime = startTime.clone()
viewer.clock.clockRange = Cesium.ClockRange.LOOP_STOP // 循环播放
viewer.clock.multiplier = 1 // 1 倍速

const sampled = new Cesium.SampledPositionProperty()

// setInterpolationOptions(options) 设置插值时使用的算法及阶数
sampled.setInterpolationOptions({
  interpolationDegree: 2, // 二阶插值
  interpolationAlgorithm: Cesium.HermitePolynomialApproximation, // 使用 Hermite 多项式插值
})

// 生成“弧形”轨迹：经纬度线性插值，高度做正弦抬升
for (let i = 0; i <= sampleCount; i++) {
  const t = i / sampleCount // 0~1
  const time = Cesium.JulianDate.addSeconds(startTime, t * durationSec, new Cesium.JulianDate())

  const lon = Cesium.Math.lerp(startLLH.lon, endLLH.lon, t)
  const lat = Cesium.Math.lerp(startLLH.lat, endLLH.lat, t)

  const baseH = Cesium.Math.lerp(startLLH.h, endLLH.h, t)
  const arcH = 6000 * Math.sin(Math.PI * t) // 中段抬升 6000m（可调）
  const h = baseH + arcH

  sampled.addSample(time, Cesium.Cartesian3.fromDegrees(lon, lat, h))
}

// 用速度方向自动求朝向
const orientation = new Cesium.VelocityOrientationProperty(sampled)

const plane = viewer.entities.add({
  availability: new Cesium.TimeIntervalCollection([
    new Cesium.TimeInterval({ start: startTime, stop: stopTime })
  ]),
  position: sampled,
  orientation,
  model: {
    uri: an72pUrl,
    scale: 1.0,
    minimumPixelSize: 64,
    maximumScale: 200,
  },
  // 可选：显示轨迹线
  path: {
    leadTime: 0,
    trailTime: durationSec,
    width: 2,
    material: Cesium.Color.CYAN.withAlpha(0.9),
  },
})

viewer.trackedEntity = plane
```

<img src="./assets/image-20260106223929737.png" alt="image-20260106223929737" style="zoom: 67%;" />



### Entity 可视性配置

| 配置项                       | 作用                   | 使用场景           | 值范围        |
| ---------------------------- | ---------------------- | ------------------ | ------------- |
| **pixelSize**                | 实体在屏幕上的像素大小 | 控制点的基础大小   | 1-100         |
| **disableDepthTestDistance** | 禁用深度检测的距离     | 近距离穿透地球显示 | 0 ~ ∞         |
| **translucencyByDistance**   | 根据距离调整透明度     | 远处淡出，近处显示 | NearFarScalar |
| **scaleByDistance**          | 根据距离调整大小       | 远处变小，近处正常 | NearFarScalar |
| **outlineWidth**             | 轮廓宽度               | 提升点的显眼度     | 0-5           |
| **outlineColor**             | 轮廓颜色               | 与主颜色对比       | Color         |



先看一个点，然后会解析其中可视性配置

![微信图片_2026-01-23_074456_682](./assets/微信图片_2026-01-23_074456_682.png)

代码：

```js
// 在距离地表500km的高度添加一个点实体
const pointEntity = viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(0, 0, 500000),
  point: {
    pixelSize: 10,
    color: Cesium.Color.CYAN,
    disableDepthTestDistance: 0, // 永远进行深度测试
    outlineColor: Cesium.Color.WHITE,   // 轮廓颜色
    outlineWidth: 2,                    // 轮廓宽度
  },
  /**
   * NearFarScalar(nearDistance, nearValue, farDistance, farValue)
   * 从 nearDistance 到 farDistance，线性插值 nearValue 到 farValue
   */
  label: {
    text: 'Walker_P1_S1',
    font: '14px sans-serif',
    pixelOffset: new Cesium.Cartesian2(0, -20), // 标签位置向上偏移20像素
    translucencyByDistance: new Cesium.NearFarScalar(
      2.0e6,    // 距离 200 万米（2000km）
      1.0,      // 不透明（透明度 100%）
      5.0e7,    // 距离 5000 万米（50000km）
      0.3       // 半透明（透明度 30%）
    ),
    // 大小渐变：2000km 处 100% 大小，50000km 处 50% 大小
    scaleByDistance: new Cesium.NearFarScalar(
      2.0e6,    // 距离 2000km
      1.0,      // 大小缩放比例 100%（正常大小）
      5.0e7,    // 距离 50000km
      0.5       // 大小缩放比例 50%（缩小一半）
    ),
  }
})
```



<hr>



#### pixelSize - 基础像素大小

```js
// 在距离地表500km的高度添加一个点实体
const pointEntity = viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(0, 0, 500000),
  point: {
    pixelSize: 10, // 显示为10像素
    color: Cesium.Color.CYAN,
  },
})
```

- 单位：屏幕像素（不随距离变化）
- 适用场景：远近都是固定大小
- 问题：远距离太小，近距离太大

<hr>



#### disableDepthTestDistance - 深度检测禁用

```js
const pointEntity = viewer.entities.add({
  position: Cesium.Cartesian3.fromDegrees(0, 0, 500000),
  point: {
    pixelSize: 10,
    color: Cesium.Color.RED,
    disableDepthTestDistance: 0, // 永远进行深度测试
  },
})
```

| 值                           | 行为            | 说明               | 使用场景               |
| ---------------------------- | --------------- | ------------------ | ---------------------- |
| **0**                        | ✅ 启用深度检测  | 地球会遮挡远处的点 | 卫星点（默认）         |
| **100**                      | ⚠️ 100km 内禁用  | 100km 外被地球遮挡 | 标签（避免被地球挡住） |
| **1e5**                      | ✅ 10万km 内禁用 | 很远距离才被遮挡   | 信息标签               |
| **Number.POSITIVE_INFINITY** | ❌ 完全禁用      | 穿透地球始终显示   | 导弹、告警标记         |

<hr>



#### translucencyByDistance - 透明度渐变

```js
/**
 * NearFarScalar(nearDistance, nearValue, farDistance, farValue)
 * 从 nearDistance 到 farDistance，线性插值 nearValue 到 farValue
 */
label: {
  text: 'Walker_P1_S1',
  font: '14px sans-serif',
  translucencyByDistance: new Cesium.NearFarScalar(
    2.0e6,    // 距离 200 万米（2000km）
    1.0,      // 不透明（透明度 100%）
    5.0e7,    // 距离 5000 万米（50000km）
    0.3       // 半透明（透明度 30%）
  ),
}
```

<hr>



####  scaleByDistance - 大小渐变

```js
/**
 * 根据相机距离动态调整实体大小
 */
label: {
  text: satName,
  
  // 大小渐变：2000km 处 100% 大小，50000km 处 50% 大小
  scaleByDistance: new Cesium.NearFarScalar(
    2.0e6,    // 距离 2000km
    1.0,      // 大小缩放比例 100%（正常大小）
    5.0e7,    // 距离 50000km
    0.5       // 大小缩放比例 50%（缩小一半）
  ),
}
```

<hr>



#### Outline - 轮廓效果

```js
point: {
  pixelSize: 8,
  color: Cesium.Color.CYAN,           // 主颜色
  outlineColor: Cesium.Color.WHITE,   // 轮廓颜色
  outlineWidth: 2,                    // 轮廓宽度
}
```



#### model可视配置

| 配置项                       | 作用                                         | 使用场景                             | 值范围               | 默认值     |
| ---------------------------- | -------------------------------------------- | ------------------------------------ | -------------------- | ---------- |
| **scale**                    | 等比缩放模型自身尺寸                         | 调整模型在场景中的实际大小           | 0.0 ~ ∞              | 1.0        |
| **minimumPixelSize**         | 模型最小显示的像素大小                       | 重要标记不被缩小到消失（如地图标记） | 0.0 ~ ∞              | 0.0        |
| **maximumScale**             | 限制 minimumPixelSize 动态补偿的上限缩放倍数 | 防止模型在极远处被无限放大变形       | 0.0 ~ ∞              | 无限制     |
| **availability**             | 指定模型在特定时间段内可见                   | 时间轴动画、临时展示                 | TimeInterval         | 始终显示   |
| **imageBasedLightingFactor** | 调整环境光的漫反射和高光强度                 | 解决模型发暗，增强视觉效果           | Cartesian2(0~∞, 0~∞) | (1.0, 1.0) |

> ⚠️ **scale、minimumPixelSize、maximumScale 三者关系：**
>
> - **镜头近**：模型像素够大，`scale` 直接生效
> - **镜头拉远**：模型像素不足，cesium自动增大模型的缩放倍数，保证模型显示不小于`minimumPixelSize` 属性设置的像素值，此时实际渲染缩放 = max(scale, 满足minimumPixelSize所需的动态缩放系数)
> - **镜头极远**：动态补偿系数超过 `maximumScale` 上限，补偿被截断，**minimumPixelSize 失效**（模型会继续随距离缩小）

```js
model: {
  uri: 'src/data/missileDF.glb',
  // 几何缩放：将模型放大5000倍（镜头近时生效，镜头远时可能被 minimumPixelSize 的动态补偿覆盖）
  scale: 5000.0,
  // 像素保证：远景时也至少显示64像素（会自动计算动态补偿系数，覆盖 scale）
  minimumPixelSize: 64,
  // 补偿上限：动态补偿系数最大不超过20000倍，防止极远处模型异常放大
  maximumScale: 20000,
  // 光照调节：漫反射增强，解决模型发暗问题
  imageBasedLightingFactor: new Cesium.Cartesian2(1.0, 1.0),
},
```





##### imageBasedLightingFactor（基于图像的照明因子）

- 调节模型的"亮度和高光反射"
  - `Cartesian2(diffuse, specular)` 中：
    - **diffuse（漫反射）**：模型本身的基础亮度
    - **specular（镜面反射）**：模型高光和反光效果的强度

```
官方解释：
A property specifying the Cartesian2 used to scale the diffuse and specular image-based lighting contribution to the final color.
一个属性，用于指定用于将漫反射和镜面基于图像的光照贡献量转换为最终颜色的笛卡尔坐标系。

diffuse和specular的指定值必须小于等于1
```



##### colorBlendMode（颜色混合提亮） 

```js
  model: {
    uri: 'src/data/missileDF.glb',
    // 几何缩放：将模型放大5000倍
    scale: 5000.0,
    // 像素保证：远景时也至少显示64像素
    minimumPixelSize: 64,
    // 光照调节：漫反射增强，解决暗问题
    imageBasedLightingFactor: new Cesium.Cartesian2(1.0, 1.0),

    // 用白色或浅色混合来提亮模型
    color: Cesium.Color.WHITE,
    colorBlendMode: Cesium.ColorBlendMode.MIX, // 混合模式Mix: 原色和指定颜色混合,REPLACE:完全替换为指定颜色,HIGHLIGHT:高亮显示指定颜色
    colorBlendAmount: 0.0  // 0.0=完全原色, 0.5=默认, 1.0=纯色
  }
```

图片对比，混合前对比混合后：

<img src="./assets/微信图片_2026-01-26_080309_653.png" alt="微信图片_2026-01-26_080309_653" style="zoom:50%;" /><img src="./../../开发/Typora图片/微信图片_2026-01-26_080303_919.png" alt="微信图片_2026-01-26_080303_919" style="zoom:50%;" />





### Entity模型旋转

HeadingPitchRoll

```js
new Cesium.HeadingPitchRoll(heading, pitch, roll)
将三个旋转角度组合成一个 HeadingPitchRoll 对象，用来描述物体的朝向（orientation）。
heading（偏航角） 绕Z转
pitch（俯仰角）  绕Y转
roll（翻滚角）  绕X转

// 示例1：创建一个朝向
const heading = Cesium.Math.toRadians(45);  // 东北方向（45度）
const pitch = Cesium.Math.toRadians(10);    // 向上倾斜10度
const roll = Cesium.Math.toRadians(0);      // 不翻滚

const hpr = new Cesium.HeadingPitchRoll(heading, pitch, roll);
// hpr 现在包含了这三个旋转角度，但还不能直接用于entity的orientation
```



headingPitchRollQuaternion

```js
Cesium.Transforms.headingPitchRollQuaternion(origin, headingPitchRoll, ellipsoid, fixedFrameTransform, result) → Quaternion

HeadingPitchRoll = 人类容易理解的角度表示（欧拉角）
Quaternion = 计算机3D渲染需要的数学格式（四元数）


// 参数说明
Cesium.Transforms.headingPitchRollQuaternion(
  origin,                              // 物体位置（Cartesian3）
  headingPitchRoll,                    // HeadingPitchRoll对象
  ellipsoid = Ellipsoid.default,       // 地球模型（通常用默认值）
  fixedFrameTransform = eastNorthUpToFixedFrame,  // 坐标系转换
  result                               // 输出四元数
)
// 返回：Quaternion（四元数，可直接赋给entity.orientation，实体朝向）


// 示例2：完整的转换流程
const position = new Cesium.Cartesian3(1000000, 2000000, 3000000); // 卫星位置
const hpr = new Cesium.HeadingPitchRoll(
  Cesium.Math.toRadians(45),   // heading
  Cesium.Math. toRadians(10),   // pitch
  Cesium.Math.toRadians(0)     // roll
);

// 转换成四元数
const quaternion = Cesium.Transforms.headingPitchRollQuaternion(position, hpr);
// quaternion:  类似 Quaternion { x: 0.123, y: 0.456, z: 0.789, w: 0.891 }

console.log(quaternion);
// 输出：Quaternion { x: 0.123, y: 0.456, z: 0.789, w: 0.891 }
```



完整示例

```js
/**
 * 完整示例：在Cesium中显示一个动态旋转的卫星
 */

// 模拟CSV数据（包含位置和朝向）
const csvData = [
  { Rx: 1000000, Ry: 2000000, Rz: 3000000, Yaw: 0, Pitch: 0, Roll:  0 },
  { Rx: 1100000, Ry: 2100000, Rz: 3100000, Yaw:  45, Pitch: 10, Roll: 5 },
  { Rx:  1200000, Ry: 2200000, Rz: 3200000, Yaw: 90, Pitch: 20, Roll: 10 },
];

let timeIndex = 0;

// 创建卫星实体
const entity = viewer.entities.add({
  id: 'satellite_1',
  
  // 位置：从CSV读取 (Cartesian3)
  position: new Cesium.CallbackProperty(() => {
    const row = csvData[timeIndex % csvData.length];
    return new Cesium.Cartesian3(row. Rx, row.Ry, row.Rz);
  }, false),
  
  // 朝向：通过 HeadingPitchRoll 转换成 Quaternion
  orientation: new Cesium.CallbackProperty(() => {
    const row = csvData[timeIndex % csvData.length];
    const position = new Cesium.Cartesian3(row.Rx, row.Ry, row. Rz);
    
    // 【步骤1】创建HeadingPitchRoll对象
    const hpr = new Cesium.HeadingPitchRoll(
      Cesium.Math.toRadians(row.Yaw),      // heading（偏航）
      Cesium.Math. toRadians(row.Pitch),    // pitch（俯仰）
      Cesium. Math.toRadians(row. Roll)      // roll（翻滚）
    );
    
    // 【步骤2】转换成Quaternion
    return Cesium.Transforms.headingPitchRollQuaternion(position, hpr);
  }, false),
  
  // 用模型或简单形状表示卫星
  model: {
    uri:  Cesium.BoxGraphics. ConstructorOptions.outlineColor,
    minimumPixelSize: 50,
  },
});

// 模拟时间推进
setInterval(() => {
  timeIndex++;
}, 1000);
```























### 最小可执行（创建实体）





#### 基础盒体

```js
// 1. 基础盒体
const box = viewer.entities.add({
  name: "box",
  position: Cesium.Cartesian3.fromDegrees(114.30, 30.50, 20),
  box: {
    dimensions: new Cesium.Cartesian3(50, 50, 50),
    heightReference: Cesium.HeightReference.CLAMP_TO_GROUND,
    material: Cesium.Color.PURPLE.withAlpha(0.5),
    outline: true,
    outlineColor: Cesium.Color.BLACK,
  }
});
```

- `dimensions`：长宽高（米）。
- `heightReference.CLAMP_TO_GROUND`：贴地（会跟随地形）。
- 半透明材质 + 轮廓分离显示。
- `DistanceDisplayCondition(near, far)`：控制可视距离范围，指定此框将在相机多远位置显示的属性，参数：可见区间最小距离和最大距离

<img src="./assets/image-20251103104022841.png" alt="image-20251103104022841" style="zoom:33%;" />



#### 椭圆挤压(柱体效果)

“圆柱”通过挤压椭圆（Ellipse + extrudedHeight）

```ts
// 2. 局部 ENU 变换 + 椭圆挤压(柱体效果)
const enu = Cesium.Transforms.eastNorthUpToFixedFrame(
  Cesium.Cartesian3.fromDegrees(114.30, 30.50, 20)
);
const cylinder = viewer.entities.add({
  name: "cylinder",
  position: Cesium.Matrix4.multiplyByPoint(enu, new Cesium.Cartesian3(200, 0, 0), new Cesium.Cartesian3()),
  ellipse: {
    semiMajorAxis: 140,
    semiMinorAxis: 70,
    height: 30,
    extrudedHeight: 100,
    material: Cesium.Color.YELLOW.withAlpha(0.5),
    outline: true,
    outlineColor: Cesium.Color.BLACK
  }
});
```

- `height` 是底面高度；`extrudedHeight` 是顶面高度。
- 挤压后形成“柱体”。
- `heightReference.NONE`：绝对高度，不贴地。

<img src="./assets/image-20251103104949404.png" alt="image-20251103104949404" style="zoom:33%;" />



#### 折线体积（管道）

```ts
// 3. 折线体积
const pipeLocal = [new Cesium.Cartesian3(0,200,0), new Cesium.Cartesian3(0,500,0), new Cesium.Cartesian3(200,500,0), new Cesium.Cartesian3(200,0,0)];
const pipeWorld = pipeLocal.map(p => Cesium.Matrix4.multiplyByPoint(enu, p, new Cesium.Cartesian3()));
viewer.entities.add({
  name: "pipe",
  polylineVolume: {
    positions: pipeWorld,
    shape: [
      new Cesium.Cartesian2(-10,-10),
      new Cesium.Cartesian2(10,-10),
      new Cesium.Cartesian2(10,10),
      new Cesium.Cartesian2(-10,10)
    ],
    material: Cesium.Color.CYAN.withAlpha(0.5),
    cornerType: Cesium.CornerType.ROUNDED,
    outline: true,
    outlineColor: Cesium.Color.BLACK
  }
});
```

- `shape` 在局部截面平面内定义轮廓。
- 按折线路径挤出形体（造管道、围栏、轨道）。
- `cornerType.ROUNDED`：拐角圆滑过渡。

<img src="./assets/image-20251103105531671.png" alt="image-20251103105531671" style="zoom:33%;" />



#### 多边形 + 孔洞 + 挤压

```ts
// 4. 多边形 + 孔洞 + 挤压
function localToWorld(list:number[][]){
  return list.map(([x,y]) => Cesium.Matrix4.multiplyByPoint(enu, new Cesium.Cartesian3(x,y,0), new Cesium.Cartesian3()));
}
const boundary = localToWorld([[-300,-300],[-600,-300],[-600,-600],[-300,-600]]);
const hole1   = localToWorld([[-400,-400],[-500,-400],[-500,-500],[-400,-500]]);
const hole2   = localToWorld([[-420,-420],[-450,-420],[-450,-450],[-420,-450]]);
viewer.entities.add({
  name: "polygon",
  polygon: {
    hierarchy: new Cesium.PolygonHierarchy(
      boundary,
      [ new Cesium.PolygonHierarchy(hole1, [ new Cesium.PolygonHierarchy(hole2) ]) ]
    ),
    height: 100,
    extrudedHeight: 200,
    material: Cesium.Color.GREEN.withAlpha(0.5),
    outline: true,
    outlineColor: Cesium.Color.BLACK
  }
```

- `PolygonHierarchy(positions, holes)` 支持递归嵌套。

  - PolygonHierarchy：new Cesium.PolygonHierarchy(positions, holes)，一系列线性环构成了一个多边形及其内部空洞(holes)结构，而这些空洞本身也可能包含内部的空洞(holes)，这些内部空洞(holes)又构成了更小的多边形结构

  - positions: Cartesian3组成的数组定义多边形的轮廓

  - holes: 多边形孔洞，由PolygonHierarchy组成的数组，这是个嵌套结构

    

- 多层孔洞：外环 → 孔1 → 孔1内部再嵌套孔2。

- `extrudedHeight` 形成“墙体 / 体块”。

<img src="./assets/image-20251103110657532.png" alt="image-20251103110657532" style="zoom:33%;" />





#### 标注实体（billboard`、`label`、`point）

一个 Entity 聚合多种可视子图形

```ts
// 5. 标注实体 (图标 + 标签 + 点)
const mark = viewer.entities.add({
  name: "mark",
  position: Cesium.Cartesian3.fromDegrees(114.30, 30.50, 0),
  billboard: {
    image: "/src/assets/Programmer.png",
    width: 64,
    height: 64,
    scaleByDistance: new Cesium.NearFarScalar(10, 1.2, 10000, 0.1),
    verticalOrigin: Cesium.VerticalOrigin.TOP,
    horizontalOrigin: Cesium.HorizontalOrigin.CENTER
  },
  label: {
    text: "程序员",
    font: "16px",
    showBackground: true,
    backgroundColor: Cesium.Color.BLACK,
    verticalOrigin: Cesium.VerticalOrigin.BOTTOM
  },
  point: {
    pixelSize: 10,
    color: Cesium.Color.YELLOW
  }

          
          
// 一个 Entity 可以同时拥有 billboard、label、point 等多种“Graphics”属性，它们共享同一个 position。

// 相同的 position 会让 billboard / label / point 自然叠放在同一点上。

// Graphics 是“声明式外观描述”；Cesium 内部会根据属性生成底层 Primitive（自动更新）。
```





 BillboardGraphics（“广告牌”图形）

```ts
billboard: {
  image: '/src/assets/Programmer.png',
  width: 64,
  height: 64,
  sizeInMeters: false,
  scaleByDistance: new Cesium.NearFarScalar(10, 1.2, 10000, 0.1),
  verticalOrigin: Cesium.VerticalOrigin.TOP,
  horizontalOrigin: Cesium.HorizontalOrigin.CENTER
}
```

- image：可 URL / HTMLCanvas / Image / Base64。
- width/height：像素大小（只有在未使用 scale + 原图尺寸需要强制指定时才设定）。
- sizeInMeters：false 表示尺寸基于屏幕像素；true 则尝试用世界距离表示（会受相机缩放影响较大）。
- scaleByDistance：距离缩放曲线 NearFarScalar(near, nearValue, far, farValue)：
  - near 距离内尺度 ≈ nearValue
  - far 距离及之外尺度 ≈ farValue
  - 中间进行平滑插值
- verticalOrigin/horizontalOrigin：确定“定位点”在图像的哪个位置：
  - TOP + CENTER：图像顶部对齐 position（常用于让图标“挂”在点上）。
  - 比如verticalOrigin
  - ![image-20251103111327140](./assets/image-20251103111327140.png)





LabelGraphics（“文本标签”）

```ts
label: {
  text: '程序员',
  font: '16px',            // 可 '16px sans-serif'、'bold 14px monospace' 等
  showBackground: true,
  backgroundColor: Cesium.Color.BLACK,
  verticalOrigin: Cesium.VerticalOrigin.BOTTOM
}
```

- text：纯文本；复杂内容或图标组合需用 billboard 或直接绘制到 canvas。
- font：CSS 字体字符串，需包含字号（e.g. '16px sans-serif'）。
- showBackground + backgroundColor：提供文本背景矩形，提升可读性。
- verticalOrigin：BOTTOM → 标签底部对齐 position；配合 billboard TOP 让文本位于图标上方





PointGraphics（“点标记”）

```ts
point: {
  pixelSize: 10,
  color: Cesium.Color.YELLOW
}
```

<img src="./assets/image-20251103111604528.png" alt="image-20251103111604528" style="zoom: 50%;" />







#### 实体聚合

加载一批点位（GeoJSON），用 GeoJsonDataSource 生成实体（Entity），启用 clustering 按屏幕像素范围自动聚合，并在聚类事件中自定义聚合显示样式（不同数量使用不同图标、大小）。



最小实现代码

```ts
// 6. 加载 GeoJSON 点并启用聚合
const url = "/src/assets/dataSource/points.json"; // 需准备实际文件
new Cesium.GeoJsonDataSource().load(url).then(dataSource => {
  dataSource.clustering.enabled = true;
  dataSource.clustering.pixelRange = 20;
  dataSource.clustering.minimumClusterSize = 3;

  // 自定义点样式
  dataSource.entities.values.forEach(e => {
    if (e.billboard) {
      e.billboard.image = new Cesium.ConstantProperty("/src/assets/监控.png");
      e.billboard.width = new Cesium.ConstantProperty(40);
      e.billboard.height = new Cesium.ConstantProperty(40);
    }
  });

  // 聚合回调
  dataSource.clustering.clusterEvent.addEventListener((clusteredEntities, cluster) => {
    if (!clusteredEntities.length) return;
    cluster.label.show = false;
    cluster.billboard.show = true;
    const n = clusteredEntities.length;
    cluster.billboard.width = cluster.billboard.height = 50;
    let base = "/src/assets/cluster";
    if (n > 300) { cluster.billboard.width = cluster.billboard.height = 80; cluster.billboard.image = `${base}/300+.png`; }
    else if (n >= 150) { cluster.billboard.width = cluster.billboard.height = 60; cluster.billboard.image = `${base}/150+.png`; }
    else if (n >= 90)  cluster.billboard.image = `${base}/90+.png`;
    else if (n >= 30)  cluster.billboard.image = `${base}/30+.png`;
    else if (n >= 10)  cluster.billboard.image = `${base}/10+.png`;
    else cluster.billboard.image = `${base}/${n}.png`;
  });

  viewer.dataSources.add(dataSource);
  viewer.zoomTo(ds);
})
```



**api解析**

`Cesium.GeoJsonDataSource.load(url)`异步加载 GeoJSON，返回 `Promise<GeoJsonDataSource>`

`GeoJsonDataSource` 实例。它内部核心结构可以概念化为：

```ts
GeoJsonDataSource
├─ name
├─ entities : EntityCollection
│    ├─ values : Entity[]              ← 遍历的数组
│    ├─ getById(id)
│    ├─ add(entity) / remove(entity)
│    └─ ...管理实体的辅助方法
├─ clustering : EntityCluster
│    ├─ enabled / pixelRange / minimumClusterSize
│    ├─ clusterEvent (Event)
├─ clock (有些数据源类型会设定, GeoJSON 通常不)
└─ （其它：credit、_entityCollection 私有字段等）
```



GeoJsonDataSource把传入的 GeoJSON 数据翻译成一系列 cesium entity，并通过内部的 EntityCollection来统一管理

所以当非聚合时，统一点样式（可以通过遍历`dataSource`中的entity 数组来）

```ts
dataSource.entities.values.forEach(e => {
  if (e.billboard) {
    e.billboard.image = new Cesium.ConstantProperty("/icons/point.png");
    e.billboard.width = new Cesium.ConstantProperty(40);
    e.billboard.height = new Cesium.ConstantProperty(40);
  }
});
```



`dataSource.clustering.clusterEvent.addEventListener(clusteredEntities, cluster)`生成/更新聚合符号时触发回调，自定义外观

回调参数 `clusteredEntities` , 属于该聚合的原始 `Entity[]`，可用其 `length` 判断规模

回调参数 `cluster`

```
* cluster对象字面结构如下：
* {billboard: Billboard, label: Label, point: PointPrimitive}
* billboard: 聚类的广告牌对象
* label: 聚类的标签对象
* point: 聚类的点对象
```







实现效果参考

![screenshots](./assets/screenshots.gif)



















### 数据结构与管理

下面是批量管理entity的心得



这里先把viewer的字面量结构简化为以下模型，如果要参考更加具体结构，详见viewer，

`viewer.dataSources` 是 `DataSourceCollection`，里面可以挂很多 `DataSource`

```ts
const viewer = {
  entities: EntityCollection,        // 全局实体集合（默认 dataSource）
  dataSources: DataSourceCollection, // 各种 DataSource 的集合
  scene: { /* camera / globe / primitives ... */ },
  clock: Clock,
  // ...
};
```

`dataSources` 里每个 `CustomDataSource` 可以简化为，每个 `CustomDataSource` 内部都有一个 `entities: EntityCollection`

```ts
CustomDataSource {
  name: string,
  show: boolean,
  entities: EntityCollection {
    add(entityOptions) => Entity,
    getById(id) => Entity | undefined,
    remove(entity) => boolean,
    removeAll() => void,
    // ...
  }
}
```



先通过`DataSourceCollection`上的`getByName`方法取得相应数据源（参考viewer示例图，已经列出该方法），如果没有就通过 `DataSourceCollection` 创建一个专用数据源

```ts
let ds = viewer.dataSources.getByName('metro-lines')[0] as Cesium.CustomDataSource | undefined;
if (!ds) {
  ds = new Cesium.CustomDataSource('metro-lines');
  viewer.dataSources.add(ds);
} else {
  ds.entities.removeAll(); // 清空旧内容
}
```

`viewer.dataSources`：是一个 `DataSourceCollection`，相当于「数据源列表」

- 用 `new Cesium.CustomDataSource('metro-lines')` 创建了一个**专门用于metro-lines的 DataSource**，并用 `viewer.dataSources.add(ds)` 挂到 viewer 上。
- 之后所有地铁相关的东西都塞进这个 `ds` 里，而不是直接塞进 `viewer.entities`（全局集合），避免像之前一样通过`viewer.entities.add`往场景中添加实体，然后还不好集中管理

地铁相关的所有 entity 都集中在一个 `CustomDataSource('metro-lines')` 下面，**显隐 / 清空 / 卸载**都特别方便：

- 隐藏整套地铁：`ds.show = false;`
- 清除整套地铁：`ds.entities.removeAll();`
- 从场景中移除整个数据源：`viewer.dataSources.remove(ds)`



假如这里创建了一个`name`为`myDS`的`CustomDataSource`，通过`new Cesium.CustomDataSource('metro-lines')`创建，`CustomDataSource`的数据结构可以简化如下：

```ts
CustomDataSource {
  name: 'myDS',
  entities: EntityCollection {
    _entities: [ Entity { id: 'e1' }, Entity { id: 'e2' } ],
    length: 2,
    add(opts) => Entity,
    getById(id) => Entity | undefined,
    remove(entity) => boolean,
    // ...
  },
}
```

`ds.entities.add(...)`，就往这个集合里塞一个 `Cesium.Entity`（归属于 `metro-lines` dataSource）

这里的`add`方法，可以图方便只往里面传入「Entity 配置字面量」`opts`，由 Cesium 自动 new

- 传统写法（显式 new）`new Cesium.Entity(options) + add(entity)`：

  ```ts
  const entity = new Cesium.Entity({
    id: 'xxx',
    position: ...,
    polyline: { ... }
  });
  ds.entities.add(entity);
  ```

- 传入entity配置字面量（语法糖）：

  ```ts
  ds.entities.add({
    id: 'xxx',
    position: ...,
    polyline: { ... }
  });
  ```



















### 核心概念

```
Entity 
是“属性集合 + 时间动态”抽象层，几何类型通过不同 Graphics 字段表达（box / ellipse / polygon / polylineVolume / billboard / label / point）。

Position 使用 Cartesian3（可由经纬度与高度通过 fromDegrees 得到）。

局部空间变换：使用东-北-上 (ENU) 矩阵实现相对布局，无需手动经纬度偏移。

挤压体量：extrudedHeight > height 将二维面拉伸成三维体（适用于 ellipse、polygon 等）。

PolygonHierarchy 支持递归孔洞结构，实现复杂镂空几何。

属性系统可时间动态（示例中用到 ConstantProperty，表示静态）。

显示控制：DistanceDisplayCondition（可见距离区间）、NearFarScalar（随距离缩放）提升视觉分层。

标注三件套：billboard（图标）、label（文本）、point（小点）可叠加形成信息组合。

聚合本质：屏幕空间聚类（不是地理聚类），根据像素范围与最小数量合并符号。

pixelRange 越大聚合越“粗”，minimumClusterSize 控制触发门槛。

clusterEvent 提供聚合后人工接管的渲染对象（统一修改图标、尺寸、隐藏 label 等）。

查找/管理：viewer.entities.values 提供当前未绑定数据源的实体集合；对数据源实体需经其 entities.values。

数据源与实体分层：GeoJsonDataSource 解析外部数据，内部仍产出 Entity。

贴地策略：HeightReference 决定是否贴地（CLAMP）、相对地形（RELATIVE）、或保持绝对高度（NONE）。
```









### 决策树

```
开始
├─ 目标类型是单个图形表达？
│   ├─ 需要体积 (三维块/柱/挤压)?
│   │   ├─ 规则矩形体 → BoxGraphics
│   │   ├─ 规则圆柱/椭圆挤压 → EllipseGraphics + extrudedHeight
│   │   ├─ 非规则多边形体量 → PolygonGraphics + PolygonHierarchy + extrudedHeight
│   │   └─ 沿路径的管道/截面体 → PolylineVolumeGraphics
│   │ 
│   └─ 仅二维面/线？
│       ├─ 椭圆面 → EllipseGraphics (无 extrudedHeight)
│       ├─ 多边形面（可能有孔洞） → PolygonGraphics + PolygonHierarchy
│       └─ 需显示截面沿路径体 → PolylineVolumeGraphics
│ 
├─ 需要标注/图标信息？
│   ├─ 仅图标且随距离缩放 → Billboard + scaleByDistance
│   ├─ 需文字标签 → Label (可与 Billboard 同位置)
│   ├─ 需视觉锚点/拾取易见 → PointGraphics 叠加
│   └─ 多元素结合 → billboard + label + point
│ 
├─ 是否相对已有实体偏移布局？
│   ├─ 是 → 建立 ENU 矩阵 (Transforms.eastNorthUpToFixedFrame) → 局部点 Cartesian3 → Matrix4.multiplyByPoint
│   └─ 否 → 直接使用 Cartesian3.fromDegrees
│ 
├─ 是否需要贴地或高度控制？
│   ├─ 跟随地形 → HeightReference.CLAMP_TO_GROUND
│   ├─ 相对地形高度偏移 → HeightReference.RELATIVE_TO_GROUND
│   └─ 绝对高度（不随地形） → HeightReference.NONE
│ 
├─ 是否存在大量点数据需简化显示？
│   ├─ 是 → GeoJsonDataSource.load → clustering.enabled=true
│   │   ├─ 设置 pixelRange (聚合距离阈值)
│   │   ├─ 设置 minimumClusterSize (触发最小数量)
│   │   └─ clusterEvent 自定义聚合符号 (基于 clusteredEntities.length 分档)
│   └─ 否 → 直接单点实体或批量添加
├─ 是否需按数量动态变更样式？
│   ├─ 聚合场景 → clusterEvent 分级图标逻辑
│   └─ 单实体 → 可忽略或使用 DistanceDisplayCondition/NearFarScalar
│ 
└─ 管理/查找实体？
    ├─ 数据源内实体 → dataSource.entities.values
    └─ 普通实体 → viewer.entities.values (find / remove / removeAll)

```











## Property 系统（动态属性系统）

`SampledPositionProperty`、`VelocityOrientationProperty` 这类“类/实例对象”本质上属于一种抽象方式：**Property 系统（动态属性系统）**。它们在运行时当然是对象（class 的实例），但在文档里经常被写成“type”，这是一种**API 约定层面的类型分类**，是一种“允许你传什么”的表达



`Cesium.Property` 不是一个“数据类型”像 `string`/`number` 那样；它更像一个**接口/抽象协议**：

- 它代表“**一个值**”，但这个值可以随时间变化（time-dynamic）。
- 典型能力是：给它一个 `JulianDate`，它能告诉你“那个时刻的值是多少”。

在 Cesium 代码里，很多具体类都“实现了 Property 这套协议”（有 `getValue(time, result)` 之类的行为），比如：

- `ConstantProperty`（恒定值）
- `SampledProperty` / `SampledPositionProperty`（按时间采样插值）
- `CallbackProperty` / `CallbackPositionProperty`（每次查询调用回调计算）
- `CompositeProperty`（分段组合）
- `ReferenceProperty`（引用别的属性）
- `VelocityOrientationProperty`（根据 position 的速度计算 orientation）

以上这些都可以算作 Property（动态属性）的具体实现类实例。



### 位置属性与采样插值（SampledPositionProperty）

SampledPositionProperty通过addSample(time, position) 组成“时间-位置序列”

详细描述：

new Cesium.SampledPositionProperty(referenceFrame, numberOfDerivatives)

- referenceFrame：参考系，默认 Cesium.ReferenceFrame.FIXED（地固参考系），用作定义位置的参考
- numberOfDerivatives：可选，表示要计算的位置导数的数量，比如速度（一阶导数）传入1，传入2则计算加速度（二阶导数），默认为0（不计算导数）
- 地固参考系：固定在地球上并与地球同步旋转的坐标系



#### addSample

addSample(time, position, derivatives)

- time：Cesium.JulianDate 对象，表示采样时间点
- position：Cesium.Cartesian3 对象，表示在该时间点的位置
- derivatives：Array.<Cartesian3>，可选，表示位置的导数数组，比如速度、加速度等

```ts
打印sampledPosition._property._values 可以看到采样点数据结构

console.log('sampledPosition._values=', sampledPosition._property._values);
//[Cartesian3, Cartesian3, Cartesian3, Cartesian3,...]

打印sampledPosition._property._times可以看到采样时间点数组

console.log('sampledPosition._times=', sampledPosition._property._times);
//[JulianDate, JulianDate, JulianDate, JulianDate,...]
```



#### getValue

getValue(time, result) → Cartesian3

对于`SampledPositionProperty`，返回time时刻的插值位置

```ts
// time -JulianDate 要查询的位置对应的时间点。默认：JulianDate.now() 当前系统时间，也就是javaScript Date.now()。
// result -Cartesian3 可选。用于存储结果的 Cartesian3 对象。如果提供了该参数，结果将存储在该对象中以避免创建新的对象，从而提高性能。



// 1. 创建一个采样位置属性
const position = new Cesium.SampledPositionProperty();

// 2. 添加采样数据：在不同时间点设置位置
const startTime = Cesium.JulianDate.now();
const time1 = Cesium.JulianDate.addSeconds(startTime, 0, new Cesium.JulianDate());
const time2 = Cesium.JulianDate.addSeconds(startTime, 10, new Cesium.JulianDate());
const time3 = Cesium.JulianDate.addSeconds(startTime, 20, new Cesium.JulianDate());

position.addSample(time1, Cesium.Cartesian3.fromDegrees(116.0, 39.0, 0));      // 北京
position.addSample(time2, Cesium.Cartesian3.fromDegrees(117.0, 39.0, 0));      // 向东移动
position.addSample(time3, Cesium.Cartesian3.fromDegrees(118.0, 39.0, 0));      // 继续向东

// 3. 创建实体并绑定位置
const entity = viewer.entities.add({
  position: position,
  point: {
    pixelSize: 8,
    color: Cesium.Color.RED,
  },
});

// 4. 查询某个时刻的位置
const queryTime = Cesium.JulianDate.addSeconds(startTime, 5, new Cesium.JulianDate());
const pos = position.getValue(queryTime); // 返回 time1 和 time2 之间的插值位置
console.log(pos); // Cartesian3 {x: ..., y: ..., z: ...}
```



#### 插值机制

```
插值计算并不是在 addSample() 时立刻发生的，而是当 Cesium 需要某个时刻 position 的值，SampledPositionProperty对象会在已有的样本点中找到相邻的两个样本点(或多个，取决于插值算法)，然后实时插出对应时刻的位置

```



代码示例和下面标题共用



### 根据速度方向自动算朝向 （VelocityOrientationProperty）

new Cesium.VelocityOrientationProperty(position, ellipsoid)

- position：Cesium.PositionProperty 对象，表示位置属性，一般传入 SampledPositionProperty 实例
- ellipsoid：Cesium.Ellipsoid 对象，表示参考椭球体，默认 Cesium.Ellipsoid.WGS84





示例

```ts
import an72pUrl from "./assets/model/an72p.glb?url"


// ...existing code...


// 1) 构造“时间-位置采样序列”
const start = Cesium.JulianDate.now();
const sampledPosition = new Cesium.SampledPositionProperty();

// 采样点：每 5 秒一个点（经纬度 + 高度），模拟飞机飞一段然后“落地停住”
const p0 = Cesium.Cartesian3.fromDegrees(116.0, 39.5, 3000);
const p1 = Cesium.Cartesian3.fromDegrees(116.2, 39.6, 3000);
const p2 = Cesium.Cartesian3.fromDegrees(116.5, 39.75, 1500);
const p3 = Cesium.Cartesian3.fromDegrees(116.7, 40.0, 0); // “落地”

sampledPosition.addSample(Cesium.JulianDate.addSeconds(start, 0, new Cesium.JulianDate()), p0); // 
sampledPosition.addSample(Cesium.JulianDate.addSeconds(start, 5, new Cesium.JulianDate()), p1);
sampledPosition.addSample(Cesium.JulianDate.addSeconds(start, 10, new Cesium.JulianDate()), p2);
sampledPosition.addSample(Cesium.JulianDate.addSeconds(start, 15, new Cesium.JulianDate()), p3);

// 2) 采样插值：控制采样点之间如何过渡（这里用最直观的“线性插值”）
sampledPosition.setInterpolationOptions({
  interpolationAlgorithm: Cesium.LagrangePolynomialApproximation, // 拉格朗日多项式插值
  interpolationDegree: 1, // 1=线性；调大可更平滑但可能过冲
});

// 3) 前向外推：超过最后采样时间后如何给 position
sampledPosition.forwardExtrapolationType = Cesium.ExtrapolationType.HOLD; // 关键：保持最后位置
sampledPosition.forwardExtrapolationDuration = 3600; // 往后 1 小时都保持（给个足够大的值即可）,值为0表示不限制时间

viewer.clock.startTime = start.clone(); // 设置时钟起点为采样起点
viewer.clock.shouldAnimate = true; 

// 朝向/姿态：自动朝速度方向（推荐做“轨迹飞行”时用）
const velocityOrientation = new Cesium.VelocityOrientationProperty(sampledPosition);

// 添加模型实体：position 用 SampledPositionProperty，orientation 用 VelocityOrientationProperty
const plane = viewer.entities.add({
  name: "AN-72P (sampled position)",
  position: sampledPosition,
  orientation: velocityOrientation,
  model: {
    uri: an72pUrl,
    minimumPixelSize: 64,
    maximumScale: 2000,
  },
  // 画出轨迹方便观察插值
  path: {
    show: true,
    width: 2,
    leadTime: 0,
    trailTime: 60,
  },
});

viewer.trackedEntity = plane;
```

<img src="./assets/image-20260111203637080.png" alt="image-20260111203637080" style="zoom:50%;" />



### 通过回调函数延迟计算属性值（CallbackProperty）

`new Cesium.CallbackProperty(callback, isConstant)`

- callback：一个函数，每一帧渲染时 - 如果该属性在当前帧被使用，callback 就会执行，相当于每帧调用。
- isConstant 参数作用：
  - `true`：告诉 Cesium 该值不变，Cesium 可以缓存结果，减少回调执行次数
  - `false`：告诉 Cesium 该值每帧都可能变化，Cesium 每帧都会调用 callback



这里的callback: 
Cesium.CallbackProperty.Callback(time, result) → object

- time：当前的时间（Cesium.JulianDate 对象）。
- result：一个可选的参数，用于存储返回值的对象，以避免频繁创建新对象。若未指定，则会创建一个新的对象。



CallbackProperty：每次 Cesium 需要位置时都会调用的回调函数，动态计算新的值

```js
const viewer = new Cesium.Viewer('cesiumContainer');
const start = Cesium.JulianDate.now();

// ============ 方式1：静态位置（不随时间变化）============
viewer.entities.add({
  name: "静态",
  position: Cesium.Cartesian3.fromDegrees(116, 39, 1000),  // 固定在1000米高
  point: { pixelSize: 10, color: Cesium.Color.RED }
});

// ============ 方式2：CallbackProperty（随时间变化）============
viewer.entities.add({
  name: "动态下降",
  position: new Cesium.CallbackProperty((time) => {
    const t = Cesium.JulianDate.secondsDifference(time, start);
    const h = 1000 - 100 * t;  // 每秒下降100米
    return Cesium.Cartesian3.fromDegrees(116, 39, h);
  }, false),
  point: { pixelSize: 10, color: Cesium.Color.BLUE }
});
```



```ts

```

<img src="./assets/GIF 2026-01-11 21-22-05.gif" alt="GIF 2026-01-11 21-22-05" style="zoom:50%;" />



### 任何 Entity 属性随时间平滑变化（SampledProperty）



最简单的例子

```js
// 1. 创建一个数字属性（最简单的类型）
const positionProperty = new Cesium.SampledProperty(Number);

// 2. 在时间轴上添加样本点
const time1 = Cesium.JulianDate.fromIso8601('2026-01-23T10:00:00Z');
const time2 = Cesium.JulianDate.fromIso8601('2026-01-23T12:00:00Z');

positionProperty.addSample(time1, 100);  // 10:00时，值为100
positionProperty.addSample(time2, 200);  // 12:00时，值为200

// 3. 查询中间时刻的值（会自动插值）
const midTime = Cesium.JulianDate.fromIso8601('2026-01-23T11:00:00Z');
const value = positionProperty.getValue(midTime);  // 结果是 150（中间值）
```



new Cesium.SampledProperty(type, derivativeTypes)

一种属性，其值根据给定的时间，从所提供的样本集中插值计算的出。插值算法和插值次数由传入的参数决定。

```
type: number | Packable ：属性的类型，可以是基本类型（如Number）或实现了Packable接口的复杂类型（如Cesium.Quaternion）。

derivativeTypes: Array<number | Packable> ：可选参数，指定属性的导数类型，用于更高阶的插值计算。加入传入参数1表示计算一阶导数，传入参数2表示计算二阶导数，依此类推
```

`type` 参数支持以下类型

| 类型           | 用途                    | 示例                                            |
| -------------- | ----------------------- | ----------------------------------------------- |
| **Number**     | 标量值（数字）          | `new Cesium.SampledProperty(Number)`            |
| **Cartesian2** | 2D 位置或向量           | `new Cesium.SampledProperty(Cesium.Cartesian2)` |
| **Cartesian3** | 3D 位置或向量（最常用） | `new Cesium.SampledProperty(Cesium.Cartesian3)` |
| **Cartesian4** | 4D 向量                 | `new Cesium.SampledProperty(Cesium.Cartesian4)` |
| **Quaternion** | 旋转/方向               | `new Cesium.SampledProperty(Cesium.Quaternion)` |
| **Color**      | 颜色值插值              | `new Cesium.SampledProperty(Cesium.Color)`      |

```js
// 1. 数字插值
const alphaProperty = new Cesium.SampledProperty(Number);
alphaProperty.addSample(time1, 0);
alphaProperty.addSample(time2, 1);

// 2. 位置插值
const positionProperty = new Cesium.SampledProperty(Cesium.Cartesian3);
positionProperty.addSample(time1, Cesium.Cartesian3.fromDegrees(120, 30, 0));
positionProperty.addSample(time2, Cesium.Cartesian3.fromDegrees(121, 31, 1000));

// 3. 颜色插值
const colorProperty = new Cesium.SampledProperty(Cesium.Color);
colorProperty.addSample(time1, Cesium.Color.RED);
colorProperty.addSample(time2, Cesium.Color.BLUE);

// 4. 四元数（旋转）插值
const orientationProperty = new Cesium.SampledProperty(Cesium.Quaternion);
orientationProperty.addSample(time1, Cesium.Quaternion.IDENTITY);
orientationProperty.addSample(time2, Cesium.Quaternion.fromAxisAngle(Cesium.Cartesian3.UNIT_Z, Math.PI/2));
```





#### 插值的平滑度和算法（setInterpolationOptions）

想象有 3 个点，要在它们之间画线：

- **直线连接** = 不平滑（线性插值）
- **曲线连接** = 很平滑（多项式插值）

```js
const positionProperty = new Cesium.SampledPositionProperty();

// 添加3个采样点
positionProperty.addSample(time1, pos1);
positionProperty.addSample(time2, pos2);
positionProperty.addSample(time3, pos3);

// ========== 设置插值方式 ==========
positionProperty.setInterpolationOptions({
  interpolationDegree: 2,  // 平滑度
  interpolationAlgorithm: Cesium.LagrangePolynomialApproximation  // 算法
});

viewer.entities.add({
  position: positionProperty,
  point: { pixelSize: 8, color: Cesium.Color.RED }
});
```



##### `interpolationAlgorithm` - 插值算法

有 3 种主要算法可选：

```javascript
// 算法1：拉格朗日多项式（最常用）
Cesium.LagrangePolynomialApproximation

// 算法2：线性插值（最简单）
Cesium.LinearApproximation

// 算法3：埃尔米特多项式（需要速度信息）
Cesium.HermitePolynomialApproximation
```

三种算法对比

```javascript
// ========== 算法1：LagrangePolynomialApproximation（拉格朗日） ==========
// 最平滑，最常用
positionProperty.setInterpolationOptions({
  interpolationDegree: 5,
  interpolationAlgorithm: Cesium.LagrangePolynomialApproximation
});
// 效果：光滑的曲线，像飞机飞行轨迹

// ========== 算法2：LinearApproximation（线性） ==========
// 直线连接，最快
positionProperty.setInterpolationOptions({
  interpolationDegree: 1,  // 这里 degree 无效
  interpolationAlgorithm: Cesium.LinearApproximation
});
// 效果：生硬的折线，像跳跃运动

// ========== 算法3：HermitePolynomialApproximation（埃尔米特） ==========
// 需要速度数据，很平滑
positionProperty.setInterpolationOptions({
  interpolationDegree: 3,  // 埃尔米特通常用 3
  interpolationAlgorithm: Cesium.HermitePolynomialApproximation
});
// 效果：非常光滑，考虑加速度
```



### property

在cesium中的 Property 是一个可按时间求值的抽象接口

给他一个时间，它返回那个时间点的值

完全可以理解为一个包装，一个值被property包装后，表示这个值可以被时间驱动







#### 组合层次图

继承层级 + 策略分类 

```ts
[时间驱动层]
  Clock (currentTime, multiplier, clockRange, shouldAnimate)
  JulianDate (时间点)

       │ 每帧推进 currentTime
       ▼

[Property 根接口]  getValue(time) → value

  ├─ 语义专用接口 (按"值用途划分的层次")
  │    ├─ PositionProperty (返回 Cartesian3 位置)
  │    │     ├─ ConstantPositionProperty        (固定位置, isConstant=true)
  │    │     ├─ SampledPositionProperty         (样本插值轨迹)
  │    │     ├─ CallbackPositionProperty        (每帧回调计算位置)
  │    │     ├─ CompositePositionProperty       (区间拼接多个位置属性)
  │    │     └─ ReferenceProperty               (引用其它实体的 position)
  │    │
  │    └─ MaterialProperty (返回“材质定义”)
  │          ├─ ColorMaterialProperty
  │          ├─ ImageMaterialProperty
  │          ├─ GridMaterialProperty
  │          ├─ StripeMaterialProperty
  │          ├─ CheckerboardMaterialProperty
  │          └─ ...（可再包装其它 Property）

//通俗化说法，等自己理解更加透彻了就可以删除了
PositionProperty           // 表示位置数据可根据时间得到，该类型数据可以直接赋值给实体的position属性
ConstantPositionProperty   // 始终返回同一个Cartesian3作为位置
SampledPositionProperty    // 位置的值由，通过时间样本插值得来的表达式，计算
CallbackPositionProperty   // 位置属性值为该种表示，实体位置由每一帧被调用的回调函数动态计算
CompositePositionProperty  // 每个时间段返回一个位置数值
MaterialProperty           // 定义“材质随时间的值”


//-------------------------------------------------------
  
  ├─ 通用实现类型 (按"使用策略/获取方式"划分的分类)
  │    ├─ ConstantProperty          (通用固定值；很多字段直接给原始值会被封成它)
  │    ├─ SampledProperty<T>        (泛型样本插值基类；位置用其派生 SampledPositionProperty)
  │    ├─ TimeIntervalCollectionProperty  (分段常量；内部持有 TimeIntervalCollection)
  │    ├─ CallbackProperty          (任意值回调；若回调返回 Cartesian3 即角色=位置)
  │    ├─ CompositeProperty         (区段切换不同子 Property; 若子都是位置 → 是 PositionProperty 角色)
  │    ├─ ReferenceProperty         (引用另一个对象的某属性；若引用 position → 位置角色)
  │    ├─ PropertyBag               (多个命名属性聚合，如 {speed, heading,...})
  │    ├─ VelocityVectorProperty    (由 PositionProperty 派生速度向量)
  │    └─ VelocityOrientationProperty (由位置变化取朝向 Quaternion)
  │
  └─ 支撑结构 (不直接实现 Property，但被上述使用)
       ├─ TimeInterval                          // 单一时间段 + data
       ├─ TimeIntervalCollection                // 容器：多个 TimeInterval
       ├─ InterpolationAlgorithm (LagrangePolynomialApproximation / HermitePolynomialApproximation)
       ├─ interpolationDegree
       ├─ ExtrapolationType (NONE / HOLD / EXTRAPOLATE)
       ├─ forwardExtrapolationDuration / backwardExtrapolationDuration
       └─ （数学与矩阵工具等：Transforms、Matrix4 — 用于构造样本或计算回调值）

实体绑定例： 
Entity.position ← 任一 PositionProperty 实现类
Entity.box.material ← MaterialProperty (可间接引用颜色随时间变化的 Property)
```

Property在顶层，

给实体或 graphics 的字段赋“普通值”时 Cesium 内部会自动用 ConstantProperty 包装。

**第二个层级**为使用**策略**层次，比如用于将值包装为一个常量不随着时间变化（ConstantProperty），将值包装成一个实时根据回调函数返回得到数值得形式（CallbackProperty），

**第一个层次**为使用**用途**层次，将值包装成供位置属性使用还能根据时间变化得形式（PositionProperty），

如果需要这个位置得值按照给定得时间样本点插值计算得出（**策略**），可以让将这个值用SampledPositionProperty包装，就是让PositionProperty结合使用SampledProperty这一策略



#### 语义 × 策略，二维表

| 语义\策略               | Constant                                            | Sampled                                                      | TimeInterval                                                 | Callback                                                     | Composite                                   | Reference                                | Derived(导出的，派生的)                                      |
| ----------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| 位置 (PositionProperty) | ConstantPositionProperty (或直接 Cartesian3 被封装) | SampledPositionProperty                                      | （很少直接用区间做位置，如果用则 TimeIntervalCollectionProperty + Cartesian3 data） | CallbackPositionProperty（或 CallbackProperty 返回 Cartesian3） | CompositePositionProperty                   | ReferenceProperty（指向别实体 position） | VelocityVectorProperty / VelocityOrientationProperty 基于位置派生（输入是另一个 PositionProperty） |
| 材质 (MaterialProperty) | 某些材质固定值：ColorMaterialProperty(Color 常量)   | （少用，对材质插值可用 SampledProperty<Material> 理论上允许） | 可用 TimeIntervalCollectionProperty + 不同材质 data          | CallbackProperty + 返回 Color / Material → 包装到 ColorMaterialProperty 等 | CompositeProperty 切换不同 MaterialProperty | ReferenceProperty 指向其他材质属性       | （较少，材质派生不常用）                                     |
| 朝向 (Quaternion)       | ConstantProperty<Quaternion>                        | SampledProperty<Quaternion>                                  | TimeIntervalCollectionProperty<Quaternion>                   | CallbackProperty (计算航向俯仰翻滚)                          | CompositeProperty                           | ReferenceProperty                        | VelocityOrientationProperty（从位置导出 Quaternion）         |
| 速度向量 (Cartesian3)   | ConstantProperty<Cartesian3>                        | SampledProperty<Cartesian3>                                  | TimeIntervalCollectionProperty<Cartesian3>                   | CallbackProperty                                             | CompositeProperty                           | ReferenceProperty                        | VelocityVectorProperty（从位置差分导出速度）                 |









## Primitive 

Primitive 底层（Geometry / Appearance / Material / DrawCommand）





### primitive简介

Entity是Cesium封装的比较高级的绘图接口，主要面向普通的开发人员，即使您没有任何图形开发知识，也能使用Entity在场景中快速绘制各种几何形状。

Primitive是Cesium封装的更低级绘图接口，绘图方式更接近渲染引擎底层，但又不至于直接使用WebGL底层的绘图接口,其主要面向图形开发人员。

Primitive提供了GeometryInstance 即几何实例化的技术，在渲染大量静态数据时很有用，能够减少CPU开销，充分利用GPU的性能。

 Primitive提供了顶点着色器和片元着色器的编程接口

```js
最简单例子，蓝色的圆
const circle = new Cesium.CircleGeometry({
  center: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500),
  radius: 200.0,
});

const geometry = Cesium.CircleGeometry.createGeometry(circle);

const instance = new Cesium.GeometryInstance({
  geometry: geometry,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: Cesium.Color.BLUE,
          },
        },
      }),
    }),
    asynchronous: false,
  }),
);
```





### primitive构成

![image-20260113081108950](./assets/image-20260113081108950.png)



```js
geometryInstances代表要显示的几何形状，而appearance则代码该几何的外观样式。

Primitive = Geometry (骨架) + Appearance (皮肤)

一个 Primitive 可以包含多个Geometry，但是只能有一个Appearance

let p = [110.0, 30.0];

// 存储几何实例的数组
let instances = []; 

for (let i = 0; i < 10; i++) {
  const instance = new Cesium.GeometryInstance({
    geometry: new Cesium.EllipseGeometry({
      center: Cesium.Cartesian3.fromDegrees(p[0], p[1]),
      semiMinorAxis: 2000.0,
      semiMajorAxis: 2000.0,
      height: 1000 * i, // 每个实例的高度增加1000米
    }),
  });

  instances.push(instance);
}

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instances,
    appearance: new Cesium.EllipsoidSurfaceAppearance({
      material: Cesium.Material.fromType("Color"),
    }),
  }),
);

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 1000),
});
```

![image-20260302205247662](./assets/image-20260302205247662.png)



#### 渲染大量几何体

```js
每个Geometry是通过GeometryInstance来进行实例化的，通过GeometryInstance对Geometry进行实例化，同一个Geometry可以被实例化多次。

比如有100万个BoxGeometry需要被显示，我们只需要创建一个BoxGeometry，然后通GeometryInstance来设置每一个的大小、位置、颜色等。
```

实例化10W个样式相同的立方体

```js
// 一个 Primitive 包含多个 Geometry
let p = [110.0, 30.0];
let instances = [];

let boxGeometry = Cesium.BoxGeometry.fromDimensions({
  dimensions: new Cesium.Cartesian3(100, 100, 100),
});

// 实例化 10 万次
for (let i = 0; i < 100000; i++) {
  const instance = new Cesium.GeometryInstance({
    geometry: boxGeometry, // 复用同一个定义
    modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
      Cesium.Cartesian3.fromDegrees(
        p[0] + Math.random(),
        p[1] + Math.random(),
        200 + Math.random() * 100,
      ),
    ),
    id: "BoxGeometry" + i, // 设置唯一 ID，用于拾取识别
  });
  instances.push(instance);
}

// 创建 Primitive
viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instances,
    appearance: new Cesium.EllipsoidSurfaceAppearance({
      material: Cesium.Material.fromType("Color"),
    }),
  }),
);

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 1000),
})
```

![image-20260302210122954](./assets/image-20260302210122954.png)





#### 大量不同样式立方体

```js
通过PerInstanceColorAppearance类型的外观为每个Geometry实例设置一个颜色



let p = [110.0, 30.0];
let instances = [];

let boxGeometry = Cesium.BoxGeometry.fromDimensions({
  dimensions: new Cesium.Cartesian3(100, 100, 100),
});

for (let i = 0; i < 100; i++) {
  const instance = new Cesium.GeometryInstance({
    geometry: boxGeometry,
    modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
      Cesium.Cartesian3.fromDegrees(
        p[0] + Math.random() / 10, 
        p[1] + Math.random() / 10, 
        200 + Math.random() * 100
      )
    ),
    // 通过 attributes 为每个实例设置颜色
    attributes: {
      color: Cesium.ColorGeometryInstanceAttribute.fromColor(
        Cesium.Color.fromRandom({ alpha: 1 })  // 随机颜色
      )
    },
  });
  instances.push(instance);
}

viewer.scene.primitives.add(new Cesium.Primitive({
  geometryInstances: instances,
  appearance: new Cesium.PerInstanceColorAppearance({
    flat: true,         // 平面着色，无光照
    translucent: false  // 不透明
  })
}))
```

![image-20260202142915999](./assets/image-20260202142915999.png)





### primitive的优缺点

```
优点：
primitive能将多个geometry合并为一个大的geometry减少GPU的开销
合并过程使用WebWorker不影响UI响应
primitive由geometry和appearance构成，可以分别单独修改，编程灵活

缺点：
使用更加复杂

```





### primitive分类

Cesium中除了primitive类，还有以primitive结尾的类，这些类的使用方法和primitive类似，可以将其视为primitive的同类

```js
new Cesium.Primitive(options)
创建的几何体浮空
```





#### GroundPrimitive：将几何图形贴地显示

```js
将几何图形贴地显示

new Cesium.GroundPrimitive(options)
地面图元 - 将几何体附着在地形或 3D Tiles 表面上渲染，实现沿地表的几何体渲染
支持的几何体	CircleGeometry、CorridorGeometry、EllipseGeometry、PolygonGeometry、RectangleGeometry
不支持的几何体	BoxGeometry、SphereGeometry 等空中几何体
参数：
  geometryInstances: GeometryInstance|Array<GeometryInstance> 要渲染的几何体实例或实例数组
  appearance: Appearance 材质外观
  show: boolean 是否显示
  vertexCacheOptimize: boolean true: 优化顶点缓存，可提升性能,默认值为 false
  interleave: boolean true: 顶点数据交错存储，可提升渲染性能，增加加载时间,默认值为 false
  compressVertices: boolean true: 压缩顶点数据，减少内存占用,默认值为 true
  releaseGeometryInstances: boolean true: 渲染后释放几何体实例,节省内存,默认值为 false
  allowPicking: boolean true: 启用拾取支持,默认值为 true,false:节省GPU内存
  asynchronous: boolean true: 异步创建地面图元,避免阻塞主线程,默认值为 true
  classificationType: ClassificationType 设置贴的目标类型，TERRAIN(地形)、3D_TILES(瓦片)、BOTH(两者)，默认值为 ClassificationType.BOTH
  debugShowBoundingVolume: boolean true: 显示包围球，默认值为 false
  debugShowShadowVolume: boolean true: 显示阴影体，默认值为 false


// 贴地多边形（覆盖山区沟壑）
const groundPolygon = new Cesium.GroundPrimitive({
  geometryInstances: new Cesium.GeometryInstance({
    geometry: new Cesium.PolygonGeometry({
      polygonHierarchy: new Cesium.PolygonHierarchy(
        Cesium.Cartesian3.fromDegreesArray([
          103.55, 31.05,
          103.75, 31.05,
          103.75, 31.25,
          103.55, 31.25,
        ])
      ),
    }),
    attributes: {
      color: Cesium.ColorGeometryInstanceAttribute.fromColor(
        Cesium.Color.RED.withAlpha(0.6)
      ),
    },
    id: "ground-polygon",
  }),
  appearance: new Cesium.PerInstanceColorAppearance({
    flat: true,
  }),
  classificationType: Cesium.ClassificationType.TERRAIN, // 只贴地形
});

viewer.scene.primitives.add(groundPolygon);

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(103.65, 31.15, 5000),
})
```

![image-20260303095741925](./assets/image-20260303095741925.png)



#### ClassificationPrimitive：将几何体贴模型显示

```js
new Cesium.ClassificationPrimitive(options)
分类图元 - 高亮地形或 3D Tiles 中的特定区域
支持几何体	BoxGeometry、CylinderGeometry、EllipsoidGeometry、PolylineVolumeGeometry、SphereGeometry
椭球面几何体	CircleGeometry、PolygonGeometry 等需要拉伸（extrude）才能使用
参数同上
该图元会自动贴到几何体对应范围内的中的地形和3DTiles






// 加载3DTiles模型
const tileset = await Cesium.Cesium3DTileset.fromUrl(
  "/src/data/3Dtiles/mars3d-max-shihua-3dtiles-master/tileset.json"
);

viewer.scene.primitives.add(tileset);

// 自动飞行到模型位置
viewer.zoomTo(tileset);

// 模型位置，通过 tileset.json 中的root 的 transform 矩阵可以得知模型的地理位置，最后一行是平移向量，单位是米
const position = new Cesium.Cartesian3(
  -2474131.20871008,
  4838320.38392657,
  3327958.36971665
);

const cartographic = Cesium.Cartographic.fromCartesian(position);

const longitude = Cesium.Math.toDegrees(cartographic.longitude); // 约 117.0°E
const latitude  = Cesium.Math.toDegrees(cartographic.latitude);  // 约 30.6°N
const height    = cartographic.height;

console.log(`经度: ${longitude}, 纬度: ${latitude}, 高度: ${height}`);

// 结合文件里加载的3dtiles模型，给我对应的代码，使用ClassificationPrimitive这个类创建贴着模型的的图元，能让我直观的看到效果


// 模型中心经纬度
const centerLon = longitude; // ≈ 117.x°E
const centerLat = latitude;  // ≈ 30.x°N

// 创建一个多边形区域（围绕模型中心，约覆盖模型范围）
const classificationPrimitive1 = new Cesium.ClassificationPrimitive({

  // 实例1：红色多边形，贴在3DTiles模型上（覆盖核心区域）
  geometryInstances: new Cesium.GeometryInstance({
      geometry: new Cesium.PolygonGeometry({ // 多边形几何
        polygonHierarchy: new Cesium.PolygonHierarchy(
          Cesium.Cartesian3.fromDegreesArray([
            centerLon - 0.005, centerLat - 0.005,
            centerLon + 0.005, centerLat - 0.005,
            centerLon + 0.005, centerLat + 0.005,
            centerLon - 0.005, centerLat + 0.005,
          ])
        ),
        perPositionHeight: false, // 不使用顶点高度
      }),
      // 半透明红色
      attributes: {
        color: Cesium.ColorGeometryInstanceAttribute.fromColor(
          new Cesium.Color(1.0, 0.0, 0.0, 0.5) // 半透明红色
        )
      },
      id: "classification-polygon-1",
    }),

  // 指定贴合目标：贴合3DTiles模型（而非地形）
  classificationType: Cesium.ClassificationType.CESIUM_3D_TILE,

  // 使用每实例颜色外观
  appearance: new Cesium.PerInstanceColorAppearance({
    flat: true, // 平面着色，不受光照影响
  }),
});

viewer.scene.primitives.add(classificationPrimitive1);
```

直观的看到模型被图元贴合，包括延伸到模型外部的钢铁支架，仅支架被包裹

![image-20260303104235971](./assets/image-20260303104235971.png)

参数：

```js
// ClassificationPrimitive 构造函数参数结构
const classificationPrimitiveOptions = {
  // ===================== 核心几何配置 =====================
  geometryInstances: {
    // 类型：Array<GeometryInstance> | GeometryInstance
    // 说明：要渲染的几何实例，可以是单个实例或长度为1的数组
    // 支持的几何类型：BoxGeometry, CylinderGeometry, EllipsoidGeometry, 
    //             PolylineVolumeGeometry, SphereGeometry
    // 以及挤出体的：CircleGeometry, CorridorGeometry, EllipseGeometry, 
    //            PolygonGeometry, RectangleGeometry
    example: new Cesium.GeometryInstance({
      geometry: new Cesium.BoxGeometry(),
      attributes: { /* ... */ }
    })
  },

  appearance: {
    // 类型：Appearance
    // 默认值：当 GeometryInstances 有 color 属性时，默认为 PerInstanceColorAppearance
    // 说明：定义渲染的着色和视觉特性。仅支持所有实例颜色相同的 PerInstanceColorAppearance
    //      如需完整的 Appearance 支持，使用 GroundPrimitive 代替
    example: new Cesium.PerInstanceColorAppearance()
  },

  // ===================== 显示与控制 =====================
  show: {
    // 类型：Boolean
    // 默认值：true
    // 说明：是否显示该图元
    example: true
  },

  // ===================== 性能优化 =====================
  vertexCacheOptimize: {
    // 类型：Boolean
    // 默认值：false
    // 说明：当 true 时，为顶点着色器前后的缓存优化几何顶点
    example: false
  },

  interleave: {
    // 类型：Boolean
    // 默认值：false
    // 说明：当 true 时，交错几何顶点属性，可略微提高渲染性能但增加加载时间
    example: false
  },

  compressVertices: {
    // 类型：Boolean
    // 默认值：true
    // 说明：当 true 时，压缩几何顶点以节省内存
    example: true
  },

  releaseGeometryInstances: {
    // 类型：Boolean
    // 默认值：true
    // 说明：当 true 时，图元不保留对输入几何实例的引用，以节省内存
    example: true
  },

  allowPicking: {
    // 类型：Boolean
    // 默认值：true
    // 说明：当 true 时，每个几何实例可通过 Scene#pick 选取；
    //      当 false 时，节省 GPU 内存
    example: true
  },

  // ===================== 异步与分类 =====================
  asynchronous: {
    // 类型：Boolean
    // 默认值：true
    // 说明：当 true 时异步创建图元（推荐用于大型几何）
    //      当 false 时必须先调用 initializeTerrainHeights()
    example: true
  },

  classificationType: {
    // 类型：ClassificationType
    // 默认值：ClassificationType.BOTH
    // 说明：决定分类对象：地形、3D Tiles 或两者
    example: Cesium.ClassificationType.BOTH
    // 可选值：ClassificationType.TERRAIN (仅地形)
    //        ClassificationType.TILES_3D (仅 3D Tiles)
    //        ClassificationType.BOTH (两者)
  },

  // ===================== 调试选项 =====================
  debugShowBoundingVolume: {
    // 类型：Boolean
    // 默认值：false
    // 说明：仅用于调试。当 true 时，显示该图元命令的包围球
    example: false
  },

  debugShowShadowVolume: {
    // 类型：Boolean
    // 默认值：false
    // 说明：仅用于调试。当 true 时，绘制图元中每个几何的阴影体积
    //      必须在创建时为 true，或 releaseGeometryInstance 为 false
    example: false
  }
};
```



#### GroundPolylinePrimitive：贴地线

```js
仅用于GroundPolylineGeometry

// 使用GroundPolylinePrimitive绘制贴地线（四川盆地-青藏高原过渡地带，地形起伏明显）
const groundPolyline = new Cesium.GroundPolylinePrimitive({
  geometryInstances: new Cesium.GeometryInstance({
    geometry: new Cesium.GroundPolylineGeometry({
      positions: Cesium.Cartesian3.fromDegreesArray([
        102.0, 29.0,
        102.5, 29.5,
        103.0, 30.0,
        103.5, 30.5,
        104.0, 31.0,
        104.5, 31.5,
        105.0, 32.0,
      ]),
      width: 4.0, // 线宽（像素）
    }),
    // 半透明蓝色
    attributes: {
      color: Cesium.ColorGeometryInstanceAttribute.fromColor(
        Cesium.Color.BLUE.withAlpha(0.8)
      ),
    },
  }),

  appearance: new Cesium.PolylineColorAppearance(), // 使用颜色外观
  // asynchronous: false, // 如果要同步创建几何体，需要先初始化地形高度
});

viewer.scene.primitives.add(groundPolyline);

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(103.0, 30.0, 500000.0), // 设置相机位置
})
```

![image-20260303105746500](./assets/image-20260303105746500.png)



#### PointPrimitive：点

不直接new 而是通过 PointPrimitiveCollection .add方法 添加

PointPrimitiveCollection：点集合

```js
// 创建点集合，add(primitive, index) → object，add方法会返回添加进去的PointPrimitiveCollection给pointCollection
const pointCollection = viewer.scene.primitives.add(
    new Cesium.PointPrimitiveCollection()
);

// 添加多个点
for (let i = 0; i < 100; i++) {
  pointCollection.add({
    position: Cesium.Cartesian3.fromDegrees(
      110.0 + Math.random() * 0.1,
      30.0 + Math.random() * 0.1,
      0
    ),
    color: Cesium.Color.fromRandom({ alpha: 1.0 }),
    pixelSize: 10,
    outlineColor: Cesium.Color.WHITE,
    outlineWidth: 2
  })
}
```

![image-20260202162649209](./assets/image-20260202162649209.png)



#### VoxelPrimitive 体素渲染

##### 什么是体素(Voxel)

```
据栅格地图（Occupancy Grid Map）
画一个二维网格，每个网格单元里有实体的话就为占据状态（1），空的话就为（0）
```

<img src="./assets/image-20260202173351012.png" alt="image-20260202173351012" style="zoom:67%;" />

```
而体素就是固定分辨率的三维栅格地图。

体素网格是固定分辨率的，与之对应可变分辨率的网格叫八叉树地图(Octomap)。
```

<img src="./assets/image-20260202173828375.png" alt="image-20260202173828375" style="zoom:67%;" />

<img src="./assets/image-20260202173839928.png" alt="image-20260202173839928" style="zoom:67%;" />





##### VoxelPrimitive

```js
new Cesium.VoxelPrimitive(options)
体素图元 - 渲染三维体素数据（VoxelProvider 提供的瓦片数据），用于显示医学影像、地质数据、气象数据等三维体积数据。
参数：
  provider: VoxelProvider 体素数据提供者，提供瓦片数据源
  modelMatrix: Matrix4 定义体素图元位置和方向的变换矩阵,默认值是Matrix4.IDENTITY
  customShader: CustomShader 自定义体素着色器，用于样式化和特效处理
  clock: Clock 时钟对象，控制时间状态行为
  calculateStatistics: boolean true: 生成性能统计数据，默认值为 false
```



#####  CustomShader 参数

输入参数：FragmentInput fsInput

```c++
struct FragmentInput {
  // ========== 通用属性（与普通模型一致）==========
  attributes // 从顶点着色器插值过来的属性
  featureIds // 特征 ID
  
  // ========== 体素特有属性 ==========
  metadata   // 体素元数据（重要！）
  voxel      // 体素空间信息（重要！）
}


1. fsInput.metadata - 体素元数据
// 访问在 Provider 中定义的元数据
fsInput.metadata.color      // VEC4 - 你定义的颜色数据
fsInput.metadata.temperature // FLOAT32 - 如果定义了温度字段
fsInput.metadata.density     // FLOAT32 - 如果定义了密度字段
// ... 等等，取决于 provider.names 中定义的字段
    
对应 Provider 代码：
class GradientVoxelProvider {
  constructor() {
    this.names = ["color"];  // ← 定义元数据名称
    this.types = [Cesium.MetadataType.VEC4];  // ← 定义数据类型
    this.componentTypes = [Cesium.MetadataComponentType.FLOAT32];
  }
}


2. fsInput.voxel - 体素空间信息
struct VoxelInput {
  // 射线在体素空间中的穿越距离
  float travelDistance;  // 非常重要！用于计算透明度累积
  
  // 体素在瓦片中的局部坐标（归一化，0-1）
  vec3 tileUv;
  
  // 体素在形状空间中的位置坐标
  vec3 positionUv;  // BOX: [-1,1]³, ELLIPSOID: 球面坐标
  
  // 体素在形状空间中的位置（世界坐标）
  vec3 positionEC;  // Eye Coordinates（眼睛坐标系）
  
  // 当前采样点在体素网格中的坐标
  vec3 surfaceNormal;  // 表面法线（如果可用）
}


举例：
fragmentShaderText: `
      void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
        material.diffuse = fsInput.metadata.color.rgb;
        
        // 【关键】模拟光线在体素中的散射
        // travelDistance 是射线穿过体素的距离
        float transparency = 1.0 - fsInput.metadata.color.a;
        
        // 指数衰减：距离越长，累积的不透明度越高
        float thickness = fsInput.voxel.travelDistance * 16.0;
        material.alpha = 1.0 - pow(transparency, thickness);
      }
    `
```



JS 侧声明 uniform

```js
JS 侧声明 uniform（名字、类型、初始值）
GLSL 侧用同名变量读取
type 要和 GLSL 里使用方式匹配（FLOAT、VEC2/3/4、MAT3/4、BOOL、INT、SAMPLER_2D 等）

// ...existing code...
customShader: new Cesium.CustomShader({
  uniforms: {
    u_time: {
      type: Cesium.UniformType.FLOAT,
      value: 0.0
    },
    u_speed: {
      type: Cesium.UniformType.FLOAT,
      value: 1.0
    },
  },
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      float pulse = sin(u_time * u_speed) * 0.25 + 0.75;
      material.diffuse = fsInput.metadata.color.rgb * pulse;
      material.alpha = fsInput.metadata.color.a * pulse;
    }
  `
})
// ...existing code...





uniform 如何“设置/更新”值
用 setUniform
voxelPrimitive.customShader.setUniform('u_time', 设置的对应值);





如何让样式“随时间变化（时间 → uniform → shader 公式）
cesium时钟化的时间来源
// ...existing code...
const startJulian = viewer.clock.currentTime.clone();

viewer.scene.postRender.addEventListener(function () {
  const t = Cesium.JulianDate.secondsDifference(viewer.clock.currentTime, startJulian);
  voxelPrimitive.customShader.setUniform('u_time', t);
});
// ...existing code...
```



##### 2x2x2渐变体素

```js
requestData这个方法是cesium内部自己调用
new Cesium.VoxelPrimitive({ provider: new MiniVoxelProvider() })
    ↓
VoxelPrimitive 内部初始化
    ↓
VoxelPrimitive.prototype.update() （每帧执行）
    ↓
需要加载瓦片数据时
    ↓
自动调用 provider.requestData(options) ← 这里！
    ↓
返回 Promise<VoxelContent>
    ↓
VoxelPrimitive 渲染体素


// VoxelContent.fromMetadataArray 的作用是：
// 1. 接收体素数据数组
// 2. 将其包装成 Cesium 可识别的 VoxelContent 对象
// 3. 返回给 VoxelPrimitive 进行渲染

const content = Cesium.VoxelContent.fromMetadataArray([dataColor]);
相当于：
{
  metadata: [
    {
      name: "color",
      data: Float32Array,
      type: VEC4,
      componentType: FLOAT32
    }
  ]
}



// 类语法定义体素数据提供者
class MiniVoxelProvider {
  constructor() {
    this.shape = Cesium.VoxelShapeType.BOX; // 体素形状为立方体
    this.dimensions = new Cesium.Cartesian3(2, 2, 2); // 只有2×2×2个体素
    this.minBounds = Cesium.Cartesian3.fromElements(-1, -1, -1); // 最小边界
    this.maxBounds = Cesium.Cartesian3.fromElements(1, 1, 1); // 最大边界
    this.names = ["color"]; // 元数据名称
    this.types = [Cesium.MetadataType.VEC4]; // 元数据类型为向量4
    this.componentTypes = [Cesium.MetadataComponentType.FLOAT32]; // 组件类型为32位浮点数

    const position = Cesium.Cartesian3.fromDegrees(116.3975, 39.9085, 50000);
    const scaleMatrix = Cesium.Matrix4.fromScale(new Cesium.Cartesian3(20000, 20000, 20000));
    const translationMatrix = Cesium.Matrix4.fromTranslation(position);
    this.globalTransform = Cesium.Matrix4.multiply(
      translationMatrix, 
      scaleMatrix, 
      new Cesium.Matrix4()
    );
  }

  // ES6 类方法自动挂载在原型上，requestData方法返回体素数据
  requestData(options) {
    if (options.tileLevel >= 1) {
      return Promise.reject(`Only level 0 available`);
    }

    // 数据布局，按Z→Y→X顺序排列（从后到前，从下到上，从左到右）
    const dataColor = new Float32Array([
      // 第一层 z=0
      1.0, 0.0, 0.0, 1.0,  // (0,0,0) 红色
      0.0, 1.0, 0.0, 1.0,  // (1,0,0) 绿色
      0.0, 0.0, 1.0, 1.0,  // (0,1,0) 蓝色
      1.0, 1.0, 0.0, 1.0,  // (1,1,0) 黄色
      
      // 第二层 z=1
      1.0, 0.0, 1.0, 1.0,  // (0,0,1) 洋红
      0.0, 1.0, 1.0, 1.0,  // (1,0,1) 青色
      1.0, 1.0, 1.0, 1.0,  // (0,1,1) 白色
      0.5, 0.5, 0.5, 0.8   // (1,1,1) 灰色半透明
    ]);

    console.log('体素数据长度:', dataColor.length);
    console.log('体素数量:', 2 * 2 * 2);

    return Promise.resolve(Cesium.VoxelContent.fromMetadataArray([dataColor])); // 返回体素数据
  }
}

const voxelPrimitive = viewer.scene.primitives.add(new Cesium.VoxelPrimitive({
  provider: new MiniVoxelProvider(),
  customShader: new Cesium.CustomShader({
    fragmentShaderText: `
      void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
        material.diffuse = fsInput.metadata.color.rgb;
        material.alpha = fsInput.metadata.color.a;
      }
    `
  })
}));

viewer.camera.flyToBoundingSphere(voxelPrimitive.boundingSphere, { duration: 0.2 });
```

![image-20260202195959834](./assets/image-20260202195959834.png)





##### 8×8×8 渐变体素

<img src="./assets/image-20260202202653577.png" alt="image-20260202202653577" style="zoom: 67%;" />



```js
// 示例2：8×8×8 渐变色体素 
class GradientVoxelProvider {
  constructor() {
    this.shape = Cesium.VoxelShapeType.BOX;
    this.dimensions = new Cesium.Cartesian3(8, 8, 8);  // 8×8×8 = 512个体素
    this.minBounds = Cesium.Cartesian3.fromElements(-1, -1, -1);
    this.maxBounds = Cesium.Cartesian3.fromElements(1, 1, 1);
    this.names = ["color"];
    this.types = [Cesium.MetadataType.VEC4];
    this.componentTypes = [Cesium.MetadataComponentType.FLOAT32];

    const position = Cesium.Cartesian3.fromDegrees(116.3975, 39.9085, 50000);
    const scaleMatrix = Cesium.Matrix4.fromScale(new Cesium.Cartesian3(30000, 30000, 30000));
    const translationMatrix = Cesium.Matrix4.fromTranslation(position);
    this.globalTransform = Cesium.Matrix4.multiply(
      translationMatrix, 
      scaleMatrix, 
      new Cesium.Matrix4()
    );
  }

  requestData(options) {
    if (options.tileLevel >= 1) {
      return Promise.reject(`Only level 0 available`);
    }

    const dims = this.dimensions;
    const voxelCount = dims.x * dims.y * dims.z;
    // 这里的4代表每个体素有4个通道：R、G、B、A
    const dataColor = new Float32Array(voxelCount * 4);

    // 三层嵌套循环：Z → Y → X
    for (let z = 0; z < dims.z; z++) {
      for (let y = 0; y < dims.y; y++) {
        for (let x = 0; x < dims.x; x++) {
          // 计算一维数组索引
          // 公式：index = z * (height * width) + y * width + x
          const voxelIndex = z * dims.y * dims.x + y * dims.x + x;
          const dataIndex = voxelIndex * 4;

          // 归一化坐标 [0, 1]
          const nx = x / (dims.x - 1); // 0 到 1
          const ny = y / (dims.y - 1);
          const nz = z / (dims.z - 1);

          // 生成彩虹渐变色（基于空间位置）
          const hue = nx * 0.8;        // X方向控制色相（红→黄→绿→青）
          const saturation = 1.0 - ny * 0.3;  // Y方向降低饱和度
          const lightness = 0.5 + (nz - 0.5) * 0.3; // Z方向调整亮度

          // HSL 转 RGB
          const color = Cesium.Color.fromHsl(hue, saturation, lightness, 1.0);

          // 填充数据
          dataColor[dataIndex + 0] = color.red;
          dataColor[dataIndex + 1] = color.green;
          dataColor[dataIndex + 2] = color.blue;
          dataColor[dataIndex + 3] = 0.6;  // 半透明

          // 打印前几个体素的数据（调试用）
          if (voxelIndex < 3) {
            console.log(`体素(${x},${y},${z}):`, {
              voxelIndex,
              dataIndex,
              color: `rgb(${(color.red*255)|0}, ${(color.green*255)|0}, ${(color.blue*255)|0})`
            });
          }
        }
      }
    }

    return Promise.resolve(Cesium.VoxelContent.fromMetadataArray([dataColor]));
  }
}

const voxelPrimitive = viewer.scene.primitives.add(new Cesium.VoxelPrimitive({
  provider: new GradientVoxelProvider(),
  customShader: new Cesium.CustomShader({
    fragmentShaderText: `
      void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
        material.diffuse = fsInput.metadata.color.rgb;
        material.alpha = fsInput.metadata.color.a;
      }
    `
  })
}));

viewer.camera.flyToBoundingSphere(voxelPrimitive.boundingSphere, { duration: 2.0 });
```

<img src="./assets/image-20260202203715940.png" alt="image-20260202203715940" style="zoom:67%;" />









##### 明暗变换体素

```js
// 时间动画体素
class SimpleAnimatedVoxelProvider {
  constructor() {
    this.shape = Cesium.VoxelShapeType.BOX;
    this.dimensions = new Cesium.Cartesian3(12, 12, 12);  // 12×12×12
    this.minBounds = Cesium.Cartesian3.fromElements(-1, -1, -1);
    this.maxBounds = Cesium.Cartesian3.fromElements(1, 1, 1);
    this.names = ["color"];
    this.types = [Cesium.MetadataType.VEC4];
    this.componentTypes = [Cesium.MetadataComponentType.FLOAT32];

    const position = Cesium.Cartesian3.fromDegrees(116.3975, 39.9085, 50000);
    const scaleMatrix = Cesium.Matrix4.fromScale(new Cesium.Cartesian3(40000, 40000, 40000));
    const translationMatrix = Cesium.Matrix4.fromTranslation(position);
    this.globalTransform = Cesium.Matrix4.multiply(
      translationMatrix, 
      scaleMatrix, 
      new Cesium.Matrix4()
    );
  }

  requestData(options) {
    if (options.tileLevel >= 1) {
      return Promise.reject(`Only level 0 available`);
    }

    const dims = this.dimensions;
    const voxelCount = dims.x * dims.y * dims.z;
    const dataColor = new Float32Array(voxelCount * 4);

    for (let z = 0; z < dims.z; z++) {
      for (let y = 0; y < dims.y; y++) {
        for (let x = 0; x < dims.x; x++) {
          const voxelIndex = z * dims.y * dims.x + y * dims.x + x;
          const dataIndex = voxelIndex * 4;

          const nx = x / (dims.x - 1);
          const ny = y / (dims.y - 1);
          const nz = z / (dims.z - 1);

          // 简单渐变
          const hue = nx * 0.7 + nz * 0.3;
          const color = Cesium.Color.fromHsl(hue, 0.9, 0.6, 1.0);

          dataColor[dataIndex + 0] = color.red;
          dataColor[dataIndex + 1] = color.green;
          dataColor[dataIndex + 2] = color.blue;
          dataColor[dataIndex + 3] = 0.7;  // 固定透明度
        }
      }
    }

    return Promise.resolve(Cesium.VoxelContent.fromMetadataArray([dataColor]));
  }
}


//  创建体素图元 
// 动画：从半亮 → 全亮 → 半亮，循环往复
const voxelPrimitive = viewer.scene.primitives.add(new Cesium.VoxelPrimitive({
  provider: new SimpleAnimatedVoxelProvider(),
  customShader: new Cesium.CustomShader({
    uniforms: {
      u_time: {
        type: Cesium.UniformType.FLOAT,
        value: 0.0
      },
      u_speed: {
        type: Cesium.UniformType.FLOAT,
        value: 1.0
      },
    },
    
    fragmentShaderText: `
      void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
        vec3 baseColor = fsInput.metadata.color.rgb;
        
        // 脉冲效果（0.5 到 1.0 循环）
        float pulse = sin(u_time * u_speed) * 0.25 + 0.75;
        
        // 应用脉冲到颜色
        material.diffuse = baseColor * pulse;
        material.alpha = fsInput.metadata.color.a * pulse;
      }
    `
  })
}));

viewer.camera.flyToBoundingSphere(voxelPrimitive.boundingSphere, { duration: 0.2 });

//  动画循环 
let startTime = Date.now();
viewer.scene.postRender.addEventListener(function() {
  const elapsedTime = (Date.now() - startTime) / 1000.0;
  voxelPrimitive.customShader.setUniform('u_time', elapsedTime);
});
```

<img src="./assets/image-20260203104422984.png" alt="image-20260203104422984" style="zoom:33%;" />

<img src="./assets/image-20260203104431100.png" alt="image-20260203104431100" style="zoom:33%;" />







##### 多层次细节（LOD）瓦片

```
LOD 层级系统：
Level 0: 1×1×1 = 1个瓦片（粗糙）
Level 1: 2×2×2 = 8个瓦片（中等）
Level 2: 4×4×4 = 64个瓦片（精细）

每个瓦片：4×4×4 体素 + 边界填充（padding）

远距离观察：加载粗糙的 Level 0
拉近镜头：自动切换到精细的 Level 1、Level 2


Cesium的VoxelPrimitive会自动基于相机距离和屏幕投影大小来选择最合适的细节层级，无法手动干预。
```



```js
// 多层 LOD 瓦片体素系统
class MultiLODVoxelProvider {
  constructor() {
    this.shape = Cesium.VoxelShapeType.BOX;
    this.dimensions = new Cesium.Cartesian3(4, 4, 4);  // 每个瓦片 4×4×4
    
    // 边界填充（避免瓦片接缝）
    this.paddingBefore = new Cesium.Cartesian3(1, 1, 1); // 前填充1个体素
    this.paddingAfter = new Cesium.Cartesian3(1, 1, 1); // 后填充1个体素
    
    this.minBounds = Cesium.Cartesian3.fromElements(-1, -1, -1); 
    this.maxBounds = Cesium.Cartesian3.fromElements(1, 1, 1);
    this.names = ["color"];
    this.types = [Cesium.MetadataType.VEC4];
    this.componentTypes = [Cesium.MetadataComponentType.FLOAT32];

    const position = Cesium.Cartesian3.fromDegrees(116.3975, 39.9085, 50000);
    const scaleMatrix = Cesium.Matrix4.fromScale(new Cesium.Cartesian3(50000, 50000, 50000));
    const translationMatrix = Cesium.Matrix4.fromTranslation(position);
    this.globalTransform = Cesium.Matrix4.multiply(
      translationMatrix, 
      scaleMatrix, 
      new Cesium.Matrix4()
    );

    this.availableLevels = 3;  // 支持 3 层 LOD
    this._allVoxelData = new Array(this.availableLevels); // 预存所有层级数据

    // 预生成所有层级的数据
    this._generateAllLevels();
  }

  _generateAllLevels() {
    const { dimensions } = this;
    const channelCount = 4;

    for (let level = 0; level < this.availableLevels; level++) {
      const tilesPerDim = Math.pow(2, level); // 每个维度的瓦片数量
      const voxelCountX = dimensions.x * tilesPerDim; // X方向体素数量
      const voxelCountY = dimensions.y * tilesPerDim; // Y方向体素数量
      const voxelCountZ = dimensions.z * tilesPerDim; // Z方向体素数量
      const totalVoxels = voxelCountX * voxelCountY * voxelCountZ; // 总体素数量

      console.log(`Level ${level}: ${tilesPerDim}×${tilesPerDim}×${tilesPerDim} 瓦片, 总体素: ${totalVoxels}`);
      
      const levelData = this._allVoxelData[level] = new Float32Array(totalVoxels * channelCount); 

      for (let z = 0; z < voxelCountZ; z++) {
        for (let y = 0; y < voxelCountY; y++) {
          for (let x = 0; x < voxelCountX; x++) {
            const voxelIndex = z * voxelCountY * voxelCountX + y * voxelCountX + x;
            const dataIndex = voxelIndex * channelCount;

            // 归一化到全局空间 [0, 1]
            const nx = x / (voxelCountX - 1);
            const ny = y / (voxelCountY - 1);
            const nz = z / (voxelCountZ - 1);

            // 根据层级调整颜色（不同层级有不同色调）
            const hue = (nx * 0.5 + nz * 0.3 + level * 0.15) % 1.0;
            const saturation = 0.8 - ny * 0.2;
            const lightness = 0.5 + (nz - 0.5) * 0.2;
            
            const color = Cesium.Color.fromHsl(hue, saturation, lightness, 1.0);

            levelData[dataIndex + 0] = color.red;
            levelData[dataIndex + 1] = color.green;
            levelData[dataIndex + 2] = color.blue;
            levelData[dataIndex + 3] = 0.6;
          }
        }
      }
    }
  }

  requestData(options) {
    const { tileLevel, tileX, tileY, tileZ } = options;

    console.log(`【请求瓦片】Level: ${tileLevel}, 坐标: (${tileX}, ${tileY}, ${tileZ})`);

    const channelCount = 4;
    const { dimensions, paddingBefore, paddingAfter } = this;
    
    // 包含 padding 的瓦片尺寸
    const paddedDims = Cesium.Cartesian3.fromElements(
      dimensions.x + paddingBefore.x + paddingAfter.x,
      dimensions.y + paddingBefore.y + paddingAfter.y,
      dimensions.z + paddingBefore.z + paddingAfter.z
    );

    const tilesPerDim = Math.pow(2, tileLevel);
    const globalDims = Cesium.Cartesian3.fromElements(
      dimensions.x * tilesPerDim,
      dimensions.y * tilesPerDim,
      dimensions.z * tilesPerDim
    );

    const dataTile = new Float32Array(
      paddedDims.x * paddedDims.y * paddedDims.z * channelCount
    );

    // 从全局数据中提取当前瓦片的数据（含 padding）
    for (let z = 0; z < paddedDims.z; z++) {
      for (let y = 0; y < paddedDims.y; y++) {
        for (let x = 0; x < paddedDims.x; x++) {
          const tileIndex = z * paddedDims.y * paddedDims.x + y * paddedDims.x + x;

          // 计算在全局空间中的坐标
          let globalX = tileX * dimensions.x + (x - paddingBefore.x);
          let globalY = tileY * dimensions.y + (y - paddingBefore.y);
          let globalZ = tileZ * dimensions.z + (z - paddingBefore.z);

          // 边界裁剪
          globalX = Math.max(0, Math.min(globalDims.x - 1, globalX));
          globalY = Math.max(0, Math.min(globalDims.y - 1, globalY));
          globalZ = Math.max(0, Math.min(globalDims.z - 1, globalZ));

          const globalIndex = globalZ * globalDims.y * globalDims.x + globalY * globalDims.x + globalX;

          // 从预生成的全局数据复制
          for (let c = 0; c < channelCount; c++) {
            dataTile[tileIndex * channelCount + c] = 
              this._allVoxelData[tileLevel][globalIndex * channelCount + c];
          }
        }
      }
    }

    return Promise.resolve(Cesium.VoxelContent.fromMetadataArray([dataTile]));
  }
}


// 创建 VoxelPrimitive
const voxelPrimitive = viewer.scene.primitives.add(new Cesium.VoxelPrimitive({
  provider: new MultiLODVoxelProvider(),
  customShader: new Cesium.CustomShader({
    fragmentShaderText: `
      void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
        material.diffuse = fsInput.metadata.color.rgb;
        
        // 体积渲染效果
        float transparency = 1.0 - fsInput.metadata.color.a;
        float thickness = fsInput.voxel.travelDistance * 15.0;
        material.alpha = 1.0 - pow(transparency, thickness);
      }
    `
  })
}));

viewer.camera.flyToBoundingSphere(voxelPrimitive.boundingSphere, { duration: 2.0 });
```





#### Debug相机图元

```js
new Cesium.DebugCameraPrimitive(options)
绘制相机视场的轮廓
参数：
  camera: Camera 相机对象，定义视场位置和方向
  frustumSplits: Array<Number> 自定义视锥体的近/远平面距离，覆盖相机本身的值
  color: Color 视锥体轮廓线的颜色，默认值为 Color.CYAN
  updateOnChange: boolean true: 相机参数变化时更新视锥体，默认值为 true
  show: boolean 是否显示相机视锥体，默认值为 true
  id: object 唯一标识符，用于拾取识别
```



```js
// 创建一个固定的调试相机
const debugCamera = new Cesium.Camera(viewer.scene);

// 设置调试相机的位置
const cameraPosition = Cesium.Cartesian3.fromDegrees(
  116.3975,  // 经度
  39.9085,   // 纬度
  10000       // 高度（米）
);

// 设置相机朝向
const cameraDirection = new Cesium.Cartesian3(0, 0, -1);  // 向下看
const cameraUp = new Cesium.Cartesian3(0, 1, 0);          // 上方向

// 配置调试相机
debugCamera.position = cameraPosition;
debugCamera.direction = cameraDirection;
debugCamera.up = cameraUp;

// 设置视锥体参数
debugCamera.frustum.fov = Cesium.Math.toRadians(60);  // 视场角60度
debugCamera.frustum.near = 100;   // 近平面100米
debugCamera.frustum.far = 10000;  // 远平面10000米
debugCamera.frustum.aspectRatio = viewer.scene.drawingBufferWidth / viewer.scene.drawingBufferHeight;


//  用 DebugCameraPrimitive 可视化这个相机 
const debugCameraPrimitive = viewer.scene.primitives.add(
  new Cesium.DebugCameraPrimitive({
    camera: debugCamera,           // 使用我们创建的调试相机
    color: Cesium.Color.CYAN,   // 青色线框
    updateOnChange: true,         // 相机位置或参数变化时更新
    show: true
  })
);

//  在调试相机位置添加一个标记点 
viewer.entities.add({
  position: cameraPosition,
  point: {
    pixelSize: 8,
    color: Cesium.Color.RED,
    outlineColor: Cesium.Color.WHITE,
  },
  label: {
    text: '调试相机位置',
    font: '14px sans-serif',
    fillColor: Cesium.Color.WHITE,
    style: Cesium.LabelStyle.FILL_AND_OUTLINE,
    outlineWidth: 2,
    verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
    pixelOffset: new Cesium.Cartesian2(0, -10)
  }
});

// 主相机飞到合适位置观察调试相机的视锥体 

// GUI 控制面板
const gui = new dat.GUI();
const params = {
  showDebugCamera: true,
  
  // 调试相机的视锥体参数
  fov: 60,
  nearPlane: 100,
  farPlane: 10000,
};

gui.add(params, 'showDebugCamera').name('显示调试相机').onChange(value => {
  debugCameraPrimitive.show = value;
});

gui.add(params, 'fov', 10, 120).name('视场角(FOV)').onChange(value => {
  debugCamera.frustum.fov = Cesium.Math.toRadians(value);
});

gui.add(params, 'nearPlane', 1, 1000).name('近平面距离').onChange(value => {
  debugCamera.frustum.near = value;
});

gui.add(params, 'farPlane', 1000, 50000).name('远平面距离').onChange(value => {
  debugCamera.frustum.far = value;
});


```



![image-20260203114943564](./assets/image-20260203114943564.png)



#### Primitive粒子系统

```ts
// 1. 中心点与局部坐标系
const center = Cesium.Cartesian3.fromDegrees(114, 30, 100);
const enuMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(center);

// 2. 参数
const params = {
  image: "/src/assets/fountain.png",
  emissionRate: 40.0,
  gravity: -3.5,
  minLife: 6,
  maxLife: 7,
  minSpeed: 9,
  maxSpeed: 9.5,
  startScale: 1,
  endScale: 7,
  particleSize: 1
};

// 3. 重力辅助临时向量
const gravityScratch = new Cesium.Cartesian3();

// 4. 重力回调
function applyGravity(particle: any, dt: number) {
  Cesium.Cartesian3.normalize(particle.position, gravityScratch);
    
  // Cesium.Cartesian3.multiplyByScalar(cartesian, scalar, result) → Cartesian3,将cartesian向量乘以提供的scalar标量
  Cesium.Cartesian3.multiplyByScalar(
    gravityScratch,
    params.gravity * dt,
    gravityScratch
  );
  particle.velocity = Cesium.Cartesian3.add(
    particle.velocity,
    gravityScratch,
    particle.velocity
  );
}

// 5. 创建粒子系统
const particleSystem = viewer.scene.primitives.add(
  new Cesium.ParticleSystem({
    image: params.image,
    startColor: Cesium.Color.LIGHTSEAGREEN.withAlpha(0.7),
    endColor: Cesium.Color.WHITE.withAlpha(0.0),
    startScale: params.startScale,
    endScale: params.endScale,
    minimumParticleLife: params.minLife,
    maximumParticleLife: params.maxLife,
    minimumSpeed: params.minSpeed,
    maximumSpeed: params.maxSpeed,
    emissionRate: params.emissionRate,
    imageSize: new Cesium.Cartesian2(params.particleSize, params.particleSize),
    lifetime: 16.0,
    emitter: new Cesium.CircleEmitter(0.3),
    modelMatrix: enuMatrix,
    sizeInMeters: true,
    updateCallback: applyGravity
  })
);

// 6. 相机视图
viewer.camera.flyTo({
  destination: Cesium.Cartesian3.fromDegrees(114, 30, 300)
});

// 7. 防止页面隐藏导致 dt 过大而“爆发”
document.addEventListener("visibilitychange", () => {
  if (document.hidden) {
    particleSystem.emissionRate = 0;
  } else {
    particleSystem.emissionRate = params.emissionRate;
  }
});
```









```
粒子系统本质：在每帧根据时间增量 dt 生成、更新、淘汰粒子，属性插值（颜色/缩放）与物理加成（速度/位移）。每个粒子就是一张很小很小的图片

生命周期 (life)：每个粒子独立，系统根据 min/max 随机 → 渐变到 endColor / endScale。

发射速率 (emissionRate)：控制“每秒产生的粒子数”，过大影响性能。

初速度范围 (min/maxSpeed)：决定扩散速度；单值相近 → 形态更集中。

尺寸模式 (sizeInMeters)：true 时粒子大小世界尺度不因相机距离改变；false 时为屏幕像素大小。
Emitter：定义初始位置与方向采样空间（CircleEmitter 半径内分布）。
updateCallback：插入自定义力（重力、风、涡流）；在其中修改 particle.velocity / particle.position / particle.color 等。
重力实现简化：用位置方向归一向量近似“向外”或自定义方向，可改为固定方向 (0,0,-1) 实际重力。
dt 爆发问题：浏览器标签页后台暂停动画帧 → 返回后第一帧 dt 巨大 → 生成大量粒子或位移极大；解决：页面隐藏时 emissionRate=0。
```







#### Primitive变换

Primitive / 3D Tiles 变换决策树

```
开始：我要对场景中的对象做位置/姿态/大小调整吗？
 ├─ 对 3D Tiles 整体模型 → 使用 tileset._root.transform
 │     ├─ 是否已知地理原点 (lon,lat[,height])?
 │     │     ├─ 是 → ENU 基矩阵 = eastNorthUpToFixedFrame(origin)
 │     │     └─ 否 → 选取模型中心或计算包围球中心作为 origin
 │     ├─ 是否需要平移？ → fromTranslation(tx,ty,tz) 累乘
 │     ├─ 是否需要旋转？ → X→Y→Z 顺序构建 Matrix3 → Matrix4.fromRotation
 │     ├─ 是否需要缩放？ → fromUniformScale(scale)
 │     └─ 按序 multiply → 写入 _root.transform
 │
 ├─ 对单一 Primitive (自建几何) → 同样构造 modelMatrix 传入 Primitive({modelMatrix})
 │
 └─ 只是少量点或广告牌位置微调 → 直接改对象 position (不必矩阵)
 
 
 
   // modelMatrix的作用，如果有一个以模型中心为原点的顶点坐标，把它乘以modelMatrix后得到的是在世界坐标系里的坐标，4*4矩阵，包含旋转、缩放、平移等变换

  // root.transform的作用，3D Tiles 的几何（顶点）通常是在一个“局部小坐标”里定义的，也就是模型坐标系（通常以模型中心为原点），假如这个模型处在世界坐标系中的某个位置，如果需要把顶点的坐标值 从模型坐标系中 换算到 世界坐标系中，就需要将在模型坐标系中定义的顶点坐标值乘上root.transform

  // 所以单位矩阵乘以旋转矩阵，平移矩阵还有缩放矩阵后，赋值到root.transform上，就相当于把这些变换应用到3D Tiles模型上了
  
  
  
最常见（OpenGL / WebGL / Cesium 等）约定：列向量 v' = M * v

R11 R12 R13 Tx
R21 R22 R23 Ty
R31 R32 R33 Tz
0   0   0   1

这里 (Tx, Ty, Tz) 就是平移向量，位于第 4 列的前 3 个元素。
齐次坐标点 v = (x, y, z, 1)^T，乘法后得到： 
x' = R11x + R12y + R13z + Tx
y' = R21x + R22y + R23z + Ty
z' = R31x + R32y + R33*z + Tz
```







### primitive几何类型

共有12种，并且每一种都有对应的边线模式的Geometry类型

![primitive的geometry分类1](./assets/primitive的geometry分类1.jpg)

#### 创建平面

```js
创建平面PlaneGeometry 需要额外缩放处理

因为通过创建的平面没有尺寸：
默认为=以原点为中心、宽度和长度均为单位长度的平面的几何图形。

所以在GeometryInstance中，必须使用Matrix4.multiplyByUniformScale对modelMatrix进行缩放处理

Cesium.Matrix4.multiplyByUniformScale(matrix, scale, result) → Matrix4 原理
// 等价于：
// result = matrix × scaleMatrix
// 其中 scaleMatrix = [
//   scale,   0,      0,    0,
//   0,     scale,    0,    0,
//   0,       0,    scale,  0,
//   0,       0,      0,    1
// ]

modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(116.39, 39.9, 50000)
  )
  // ❌ 只有 1m × 1m，在 50km 高度根本看不见！



  modelMatrix: Cesium.Matrix4.multiplyByUniformScale(
    Cesium.Transforms.eastNorthUpToFixedFrame(
      Cesium.Cartesian3.fromDegrees(116.39, 39.9, 50000)
    ),
    50000.0,  // ✅ 缩放 50000 倍 → 50km × 50km
    new Cesium.Matrix4()
  )
```

完整过程

```js
// 创建 50km × 50km 的平面
const planeGeometry = new Cesium.PlaneGeometry({
  vertexFormat: Cesium.VertexFormat.POSITION_AND_ST
});

// 步骤 1: 创建东北上参考系（位置和方向）
const enuMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(
  Cesium.Cartesian3.fromDegrees(116.39, 39.9, 50000)
);

// 步骤 2: 在参考系基础上缩放
const scaledMatrix = Cesium.Matrix4.multiplyByUniformScale(
  enuMatrix,      // 输入：位置矩阵
  25000.0,        // 缩放倍数：2m → 50km (25000倍)
  new Cesium.Matrix4()  // 输出：新矩阵
);

const planeInstance = new Cesium.GeometryInstance({
  geometry: planeGeometry,
  modelMatrix: scaledMatrix
});

const material = new Cesium.Material({
  fabric: {
    type: 'Color',
    uniforms: {
      color: new Cesium.Color(1.0, 0.0, 0.0, 0.5)  // 半透明红色
    }
  }
});

const primitive = new Cesium.Primitive({
  geometryInstances: planeInstance,
  appearance: new Cesium.MaterialAppearance({
    material: material,
    translucent: true
  })
});

viewer.scene.primitives.add(primitive);
```

![image-20260303155752042](./assets/image-20260303155752042.png)

```js
new Cesium.Geometry(options)
自定义primitive几何
参数：
  attributes: GeometryAttributes 几何体属性集合（position、color、normal等）
  primitiveType: PrimitiveType 基本图元类型（TRIANGLES、LINES等）
  indices: Unit16Array|Uint32Array 索引数组，定义顶点连接顺序
  boundingSphere: BoundingSphere 可选的包围球体

new Cesium.GeometryAttribute(options)
一个几何体通常包含一个或多个属性，所有的属性共同构成了该几何体的顶点
options:
  componentDatatype: ComponentDatatype 数据类型（DOUBLE、FLOAT、UNSIGNED_BYTE等）
  componentsPerAttribute: number 该顶点的的属性有几个分量（1-4）
  normalize: boolean ，整数数据是否需要归一化到[0,1]或[-1,1]，默认值为 false 
  values: TypedArray 实际数据数组
```



#### BoxGeometry 盒子几何

```js
比如长方体、正方体创建一个盒子，通常是通BoxGeometry.fromDimensions()方法，而不是new BoxGeometry()，因为fromDimensions可以直接设置盒子的长宽高。

BoxGeometry没有提供设置位置的属性，所以需要通过GeometryInstance的modelMatrix属性将其定位到地球上正确的位置。

// 创建一个立方体几何体，定义其尺寸
const box = Cesium.BoxGeometry.fromDimensions({ 
  dimensions: new Cesium.Cartesian3(500.0, 500.0, 500.0), 
});

// 创建一个几何实例，包含几何数据和相关属性。
const instance = new Cesium.GeometryInstance({
  geometry: box,

  // 模型矩阵，定义几何体在世界坐标中的位置和方向
  modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500) 
  ), 
});

// 将几何体实例添加到场景的原始图元集合中
viewer.scene.primitives.add(new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.MaterialAppearance({
    material: new Cesium.Material({
      fabric: {
        type: "Color",
        uniforms: {
          color: Cesium.Color.BLUE
        }
      }
    })
  })
}));

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 2000), // 设置相机位置
})
```

![image-20260303180255631](./assets/image-20260303180255631.png)





#### CircleGeometry 圆几何

```js
	CircleGeometry通过center属性设置定位，但是高度需要通过height属性来设置,如果不设置height属性，圆将会贴在地表,

extrudedHeight属性设置圆的挤压高度，extrudedHeight和height属性配合使用可以创建一个有厚度的圆柱体，假如设置extrudedHeight为1000，height为500，那么最终圆柱体的高度为extrudedHeight和height的差值的绝对值，即|1000 - 500| = 500米，哪个值大，圆柱体就会向哪个方向挤压，如果extrudedHeight大于height，圆柱体会向上挤压；如果extrudedHeight小于height，圆柱体会向下挤压。


// 创建一个圆形几何体
const circle = new Cesium.CircleGeometry({
  center: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500), // 圆心坐标
  radius: 200, // 圆的半径，单位为米
  height: 500, // 圆和椭球表面之间的高度
});


// 创建一个几何体实例
const instance = new Cesium.GeometryInstance({
  geometry: circle,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: Cesium.Color.BLUE
          }
        },
      })
    })
  })
);
```

![image-20260304143806522](./assets/image-20260304143806522.png)



```js
const circle = new Cesium.CircleGeometry({
  center: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500), // 圆心坐标
  radius: 200,            // 圆的半径，单位为米
  height: 500,            // 圆和椭球表面之间的高度
  extrudedHeight: 1000,   // 圆的挤出高度
});
```

![image-20260304143943313](./assets/image-20260304143943313.png)



#### CorridorGeometry 带宽度的线几何

```js
CorridorGeometry 带宽度的线几何，也有人称为走廊，坐标中的z值会被忽略，如果需要设置高度，需要通过height属性设置。

CornerType 可以设置转角的类型，默认是 ROUNDED（圆角），还可以设置为 MITERED（锐角）或 BEVELED（斜角）

// 创建一个corridor几何
const geometry = new Cesium.CorridorGeometry({
  positions: Cesium.Cartesian3.fromDegreesArrayHeights([110.0, 30.0,1000, 110.1, 30.0,1000, 110.1, 30.1,1000]), // 走廊的路径点，包含经度、纬度和高度
  width: 1000,      // 走廊的宽度
  height: 1000,     // 走廊的高度
});

const instance = new Cesium.GeometryInstance({
  geometry: geometry,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: new Cesium.Color(1.0, 0.0, 0.0, 0.5), // 半透明红色
          }
        },
      })
    }),
  })
);
```

![image-20260304145701773](./assets/image-20260304145701773.png)



![image-20260304145944465](./assets/image-20260304145944465.png)



```js
// 创建一个corridor几何
const geometry = new Cesium.CorridorGeometry({
  positions: Cesium.Cartesian3.fromDegreesArrayHeights([110.0, 30.0,1000, 110.1, 30.0,1000, 110.1, 30.1,1000]), // 走廊的路径点，包含经度、纬度和高度
  width: 1000,      // 走廊的宽度
  height: 1000,     // 走廊的高度
  extrudedHeight: 100,
  cornerType: Cesium.CornerType.BEVELED, // 转角为斜角
});
```

![image-20260304150041383](./assets/image-20260304150041383.png)



#### CylinderGeometry 椎体几何

```js
可用于圆柱、圆锥、截头圆锥。

// 创建一个锥体几何
const cylinder = new Cesium.CylinderGeometry({
  length: 1000,            // 圆柱的高度
  topRadius: 400,          // 圆柱的顶部半径   
  bottomRadius: 700,       // 圆柱的底部半径
});

const instance = new Cesium.GeometryInstance({
  geometry: cylinder,
  modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500) // 圆柱中心的位置
  ),
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: Cesium.Color.BLUE
          }
        }
      }) 
    }),
  })
)
```

![image-20260304151327552](./assets/image-20260304151327552.png)



四棱锥

```js
const cylinder = new Cesium.CylinderGeometry({
  length: 1000,            // 圆柱的高度
  topRadius: 0,            // 圆柱的顶部半径   
  bottomRadius: 500,       // 圆柱的底部半径
  slices: 4                // 圆柱体周长上的边数
});
```

![image-20260304151934987](./assets/image-20260304151934987.png)



#### EllipseGeometry 椭圆几何

```js
EllipseGeometry和CircleGeometry很相似，但是可以设置长半径和短半径。

长半轴必须大于等于短半轴

// 创建一个椭圆几何
const ellipse = new Cesium.EllipseGeometry({
  center: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500), // 椭圆中心位置
  semiMajorAxis: 1500,      // 长半轴1500.0
  semiMinorAxis: 500,       // 短半轴500.0
  height: 300,              // 椭圆高度300.0
});

const instance = new Cesium.GeometryInstance({
  geometry: ellipse,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: Cesium.Color.BLUE
          }
        }
      })
    }),
  })
)
```

![image-20260304153400650](./assets/image-20260304153400650.png)



```js
const ellipse = new Cesium.EllipseGeometry({
  center: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500), // 椭圆中心位置
  semiMajorAxis: 1500,      // 长半轴1500.0
  semiMinorAxis: 500,       // 短半轴500.0
  height: 300,              // 椭圆高度300.0
  extrudedHeight: 800,      // 挤压高度800.0
});
```

![image-20260304153448053](./assets/image-20260304153448053.png)



#### EllipsoidGeometry 椭球体几何

```js
// 创建一个椭球体几何
const ellipsoid = new Cesium.EllipsoidGeometry({

  // 椭球体在x、y、z方向上的半径
  radii: new Cesium.Cartesian3(1000.0, 1000.0, 500.0), 
});

const instance = new Cesium.GeometryInstance({
  geometry: ellipsoid,
  modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500.0)
  ),
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: new Cesium.Color(1.0, 0.0, 0.0, 0.5), // 半透明红色
          },
        },
      })
    }),
  })
)
```

![image-20260304154218043](./assets/image-20260304154218043.png)



```js
// 创建一个椭球体几何
const ellipsoid = new Cesium.EllipsoidGeometry({

  // 椭球体在x、y、z方向上的半径
  radii: new Cesium.Cartesian3(1000.0, 1000.0, 500.0), 

  // 内半径，定义了椭球体的厚度
  innerRadii: new Cesium.Cartesian3(800.0, 800.0, 400.0), 
});
```

![image-20260304154427768](./assets/image-20260304154427768.png)



#### RectangleGeometry 矩形几何

```js
一般会从一个点集中获取一个Rectangle即这些数据的外包矩形

第一个参数是具有北、南、东、西属性的测绘矩形（ cartographic rectangle）

// 经纬高度坐标点集
let positions = [
  [108.95959, 34.220223, 435],
  [108.95922, 34.220054, 435],
  [108.959141, 34.219439, 435],
  [108.959754, 34.219573, 435],
  [108.959572, 34.21978, 435]
]
console.log(positions.flat()) // 打平数组，变成一维数组 [108.95959, 34.220223, 435, 108.95922, 34.220054, 435, ...]

positions = Cesium.Cartesian3.fromDegreesArrayHeights(positions.flat());

console.log(positions) // [Cartesian3, Cartesian3, ...]

// 创建一个矩形几何
const rectangle = new Cesium.RectangleGeometry({
  rectangle: Cesium.Rectangle.fromCartesianArray(positions),  // 从 Cartesian3 数组创建矩形
  height: 20,       // 矩形的高度
});

const instance = new Cesium.GeometryInstance({
  geometry: rectangle,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: new Cesium.Color(0.0, 0.0, 1.0, 0.5), // 半透明蓝色
          },
        },
      })
    })
  })
)
```

![image-20260304163424344](./assets/image-20260304163424344.png)

```js
// 创建一个矩形几何
const rectangle = new Cesium.RectangleGeometry({
  rectangle: Cesium.Rectangle.fromCartesianArray(positions),  // 从 Cartesian3 数组创建矩形
  height: 20,            // 矩形的高度
  extrudedHeight: 100,   // 矩形的挤压高度
});
```

![image-20260304163533167](./assets/image-20260304163533167.png)



#### PolygonGeometry 多边形几何

```js
不加高度属性默认贴地

// 经纬高度坐标点集
let positions = [
  [108.95959, 34.220223, 535],
  [108.95922, 34.220054, 335],
  [108.959141, 34.219439, 435],
  [108.959754, 34.219573, 535],
  [108.959572, 34.21978, 635]
]
console.log(positions.flat()) // 打平数组，变成一维数组 [108.95959, 34.220223, 435, 108.95922, 34.220054, 435, ...]

positions = Cesium.Cartesian3.fromDegreesArrayHeights(positions.flat());

console.log(positions) // [Cartesian3, Cartesian3, ...]

// 创建一个多边形几何
const polygon = new Cesium.PolygonGeometry({

  // 该属性定义了多边形的顶点位置，必须是一个 Cesium.PolygonHierarchy 对象
  polygonHierarchy: new Cesium.PolygonHierarchy(positions),
});

const instance = new Cesium.GeometryInstance({
  geometry: polygon,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: new Cesium.Color(0.0, 0.0, 1.0, 0.5),
          }
        }
      })
    })
  })
)



new Cesium.PolygonHierarchy(positions, holes)
参数：
  positions：一个包含 Cartesian3 对象的数组，定义了多边形的顶点位置。
  holes：一个包含 Cesium.PolygonHierarchy 对象的数组，定义了多边形中的洞（可选）。
返回值：
  一个 Cesium.PolygonHierarchy 对象，表示一个多边形的层次结构。
```

![image-20260304171253691](./assets/image-20260304171253691.png)

```js
const polygon = new Cesium.PolygonGeometry({

  // 该属性定义了多边形的顶点位置，必须是一个 Cesium.PolygonHierarchy 对象
  polygonHierarchy: new Cesium.PolygonHierarchy(positions),
  perPositionHeight: true, // 启用顶点中的高度值
});
```

![image-20260304171424325](./assets/image-20260304171424325.png)



```js
const polygon = new Cesium.PolygonGeometry({

  // 该属性定义了多边形的顶点位置，必须是一个 Cesium.PolygonHierarchy 对象
  polygonHierarchy: new Cesium.PolygonHierarchy(positions),
  height: 300, // 将所有点的高度设置为统一的值
});
```

![image-20260304171608269](./assets/image-20260304171608269.png)



```js
设置拉伸高度
const polygon = new Cesium.PolygonGeometry({

  // 该属性定义了多边形的顶点位置，必须是一个 Cesium.PolygonHierarchy 对象
  polygonHierarchy: new Cesium.PolygonHierarchy(positions),
  height: 300,      // 将所有点的高度设置为统一的值
  extrudedHeight: 200,
});
```

![image-20260304171735629](./assets/image-20260304171735629.png)



构建立面三角

```js
// 1. 定义 Geometry (几何形状)
// PolygonGeometry 用来描述多边形几何体
const triangleGeometry = new Cesium.PolygonGeometry({
  polygonHierarchy: new Cesium.PolygonHierarchy([
    // 分别视顶点，底点 A，底点 B 的 ECEF 坐标
    new Cesium.Cartesian3(6978102.994972304, -47375.42771, -27040.55263),   
    new Cesium.Cartesian3(6375804.321461924, -243290.93744919728, -224699.05604024723), 
    new Cesium.Cartesian3(6378519.921157299, 156699.8443424516, -224699.05604024723) 
  ]),
  perPositionHeight: true,
  vertexFormat: Cesium.MaterialAppearance.VERTEX_FORMAT // 告诉 Cesium 需要顶点法线和纹理坐标信息
});


/**
  perPositionHeight: true 表示每个顶点使用其自身的高度值，而不是将所有顶点投影到地球表面。如果为false, 就用GeometryInstance自带的高度值：
  new Cesium.PolygonGeometry({
    ...
    height: 1000,  // 所有顶点都在 1000m 高度
    perPositionHeight: false,  // 不使用顶点自身高度
  })
 */







// 2. 定义 GeometryInstance (几何实例)
// 允许你为同一个 Geometry 设置不同的 ID、模型矩阵（位置/缩放）或属性
const triangleInstance = new Cesium.GeometryInstance({
  geometry: triangleGeometry,
  id: 'single_triangle_side'
});



// 3. 定义 Material (材质)
// 定义颜色的具体表现形式
const cyanMaterial = Cesium.Material.fromType('Color', {
  color: Cesium.Color.CYAN.withAlpha(0.5)
});


// 4. 定义 MaterialAppearance (材质外观)
// 将材质与底层的 WebGL 渲染状态（如深度检测、剔除、混合）结合
const triangleAppearance = new Cesium.MaterialAppearance({
  material: cyanMaterial,
  translucent: true,
  renderState: {
    depthTest: { enabled: true },
    cull: { enabled: false }, // 关闭背面剔除，内外皆可见
    blending: Cesium.BlendingState.ALPHA_BLEND // 开启 alpha 混合
  }
});
/**
faceForward 确保几何体的法线始终朝向摄像机，默认为 true

法线和光照的关系：
  光照强度 = 光线方向 · 法线方向（点积）
  点积结果为正（法线指向光源）→ 该面被照亮 → 亮
  点积结果为负（法线背离光源）→ 该面背光 → 暗/黑色
一般来说，光源都是在物体外面，当摄像机从内部看的时候，法线方向朝向摄像机，背离光源，点积为负，所以看起来是会变暗。
这里的光线方向应该理解为"从表面指向光源"，而不是"光线传播的方向"

translucent： 当为真时，该几何体预计会呈现出半透明效果，默认值：真

closed: 如果为真，则预期该几何体是封闭的，因此 MaterialAppearance#renderState 的反面剔除功能将处于启用状态。这里设置为 false，表示该几何体不是封闭的，从而关闭背面剔除，使得从内部也能看到几何体。

renderState 说明：
  - depthTest.enabled: 开启深度检测
  - cull.enabled: 关闭背面剔除，这样从内部看也能看到几何体
  - blending: 使用 alpha 混合，实现半透明效果
官网说明：
  渲染几何图形时所使用的 WebGL 固定函数状态。在构建 MaterialAppearance 实例时，可以明确地定义渲染状态，或者通过 MaterialAppearance#translucent 和 MaterialAppearance#closed 进行隐式设置。
*/


// 5. 创建并添加 Primitive (图元)
// 最终将实例与外观封装，交给 Cesium 渲染引擎
const trianglePrimitive = new Cesium.Primitive({
  geometryInstances: triangleInstance,
  appearance: triangleAppearance,
  asynchronous: false // 确定该基本元素是将异步创建还是会阻塞等待其准备好。
});

viewer.scene.primitives.add(trianglePrimitive);
```

![微信图片_2026-01-14_215902_476](./assets/微信图片_2026-01-14_215902_476.png)



#### PolylineGeometry 多线段几何

```js
PolylineGeometry需要使用专门的PolylineMaterialAppearance外观

// 经纬高度坐标点集
let positions = [
  [108.95959, 34.220223, 535],
  [108.95922, 34.220054, 335],
  [108.959141, 34.219439, 435],
  [108.959754, 34.219573, 535],
  [108.959572, 34.21978, 635]
]
console.log(positions.flat()) // 打平数组，变成一维数组 [108.95959, 34.220223, 435, 108.95922, 34.220054, 435, ...]

positions = Cesium.Cartesian3.fromDegreesArrayHeights(positions.flat());

console.log(positions) // [Cartesian3, Cartesian3, ...]

// 创建一个多线段几何
const polyline = new Cesium.PolylineGeometry({
  positions: positions,
  width: 10,
})

const instance = new Cesium.GeometryInstance({
  geometry: polyline,
})

const primitive = new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.PolylineMaterialAppearance({
    material: new Cesium.Material({
      fabric: {
        type: 'Color',
        uniforms: {
          color: Cesium.Color.RED.withAlpha(0.5)
        }
      }
    })
  })
})

viewer.scene.primitives.add(primitive)
```

![image-20260305151051827](./assets/image-20260305151051827.png)

```js
PolylineGometry支持传入顶点颜色，实现渐变样式。
// 经纬高度坐标点集
let positions = [
  [108.95959, 34.220223, 535],
  [108.95922, 34.220054, 335],
  [108.959141, 34.219439, 435],
  [108.959754, 34.219573, 535],
  [108.959572, 34.21978, 635]
]

let colors = [
  Cesium.Color.RED,
  Cesium.Color.GREEN,
  Cesium.Color.BLUE,
  Cesium.Color.YELLOW,
  Cesium.Color.WHITE
];

console.log(positions.flat()) // 打平数组，变成一维数组 [108.95959, 34.220223, 435, 108.95922, 34.220054, 435, ...]

positions = Cesium.Cartesian3.fromDegreesArrayHeights(positions.flat());

console.log(positions) // [Cartesian3, Cartesian3, ...]

// 创建一个多线段几何
const polyline = new Cesium.PolylineGeometry({
  positions: positions,
  width: 10,
  colors: colors,         // 每个顶点的颜色，长度必须与positions数组长度相同
  colorsPerVertex: true,  // 确定颜色是在每个分段上保持平滑，还是在顶点间进行插值
})

const instance = new Cesium.GeometryInstance({
  geometry: polyline,
})

const primitive = new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.PolylineColorAppearance({
    translucent: false, // 设置为 false，确保线条不透明
  })
})

viewer.scene.primitives.add(primitive)
```

![image-20260305160334608](./assets/image-20260305160334608.png)



#### PolylineVolumeGeometry 管道

```js
PolylineVolumeGeometry 按2D形状拉伸的折线，也可以称为管道。创建PolylineVolumeGeometry需要先创建它的切面形状，computeCircle方法用于创建一个圆形的切面形状。

// 经纬高度坐标点集
let positions = [
  [108.95959, 34.220223, 20],
  [108.95922, 34.220054, 20],
  [108.959141, 34.219439, 25]
]

positions = Cesium.Cartesian3.fromDegreesArrayHeights(positions.flat());

console.log(positions) // [Cartesian3, Cartesian3, ...]

/**
 * 计算圆形横截面点集
 * @param {number} radius 圆形横截面的半径
 * @returns {Array<Cesium.Cartesian2>} 圆形横截面点集，包含360个点
 */
function computeCircle(radius) {
  const positions = [];
  for (let i = 0; i < 360; i++) {
    const radians = Cesium.Math.toRadians(i);
    positions.push(
      new Cesium.Cartesian2(radius * Math.cos(radians), radius * Math.sin(radians))
    )
  }
  return positions;
}

const geometry = new Cesium.PolylineVolumeGeometry({
  polylinePositions: positions,     // 定义折现体积中心的cartesian3数组
  shapePositions: computeCircle(2), // 定义折线体积的横截面形状
})

const instance = new Cesium.GeometryInstance({
  geometry: geometry,
})

const primitive = new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.MaterialAppearance({
    material: new Cesium.Material({
      fabric: {
        type: 'Color',
        uniforms: {
          color: new Cesium.Color(0.0, 0.0, 1.0, 0.5)
        }
      }
    }),
  })
});

viewer.scene.primitives.add(primitive);
```

![image-20260305161842522](./assets/image-20260305161842522.png)



#### SphereGeometry 球体几何

```js
let sphere = new Cesium.SphereGeometry({
  radius: 100.0,
});

const instance = new Cesium.GeometryInstance({
  geometry: sphere,
  modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(110.0, 30.0, 1000)
  ),
});

const primitive = new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.MaterialAppearance({
    material: new Cesium.Material({
      fabric: {
        type: "Color",
        uniforms: {
          color: new Cesium.Color(1.0, 0.0, 0.0, 1.0), // 红色
        },
      }
    }),
  })
});

viewer.scene.primitives.add(primitive);
```

![image-20260305162858948](./assets/image-20260305162858948.png)





#### WallGeometry 墙体几何

```js
let positions = [
  [108.95959, 34.220223, 50],
  [108.95922, 34.220054, 50],
  [108.959141, 34.219439, 50],
  [108.959754, 34.219573, 50],
  [108.959572, 34.21978, 50]
]
positions = positions.map(item => Cesium.Cartesian3.fromDegrees(...item))
console.log(positions)

// 墙高度为50米，底部贴地
const wall = new Cesium.WallGeometry({
  positions:positions
});

const instance = new Cesium.GeometryInstance({
  geometry: wall,
});

viewer.scene.primitives.add(
  new Cesium.Primitive({
    geometryInstances: instance,
    appearance: new Cesium.MaterialAppearance({
      material: new Cesium.Material({
        fabric: {
          type: "Color",
          uniforms: {
            color: new Cesium.Color(1.0, 0.0, 0.0, 0.5),
          },
        },
      }),
    })
  })
)

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(108.95959, 34.220223, 200),
})
```

![image-20260305163914795](./assets/image-20260305163914795.png)

#### Primitive自定义几何

```js
new Cesium.Geometry(options)
自定义primitive几何
参数：
  attributes: GeometryAttributes 几何体属性集合（position、color、normal等）
  primitiveType: PrimitiveType 基本图元类型（TRIANGLES、LINES等）
  indices: Unit16Array|Uint32Array 索引数组，定义顶点连接顺序
  boundingSphere: BoundingSphere 可选的包围球体

GeometryAttributes 为一个{..,..},由多个GeometryAttribute对象组成
new Cesium.GeometryAttribute(options)
一个几何体通常包含一个或多个属性，所有的属性共同构成了该几何体的顶点
options:
  componentDatatype: ComponentDatatype 数据类型（DOUBLE、FLOAT、UNSIGNED_BYTE等）
  componentsPerAttribute: number 该顶点的的属性有几个分量（1-4）
  normalize: boolean ，整数数据是否需要归一化到[0,1]或[-1,1]，默认值为 false 
  values: TypedArray 实际数据数组
```



按照在原点建模的方式，设四棱锥的中心点位于坐标原点，半径为1，对每个点进行标号，那么此时可以得出下图的结果：

![image-20260203145728714](./assets/image-20260203145728714.png)



```js
将坐标点存储到一个数组中

const positions = new Float64Array([
        0,0,-1,// 0
        1,-1,1,// 1
        -1,-1,1,//2
        -1,1,1,// 3
        1,1,1,//  4
])
第二步、 计算索引
有了顶点数据，我们就可以获取索引了，我们按照一定的顺序组织索引值。

const indices = new Uint16Array([
        0, 1, 2,//前
        0, 2, 3,//左
        0, 3, 4,//后
        0, 4, 1,//右
        1, 3, 2,//上
        1, 4, 3,//上
    ]);
第三步、 获取包围盒
如果包围盒计算不正确，可能会出现看不到渲染结果的情况，因为Cesium场景渲染会剔除掉当前视椎体外即看不到的图形（大多数渲染引擎应该都这样）。Cesium中的BoundingSphere类提供了fromVertices方法可以直接从顶点数组中获取包围盒对象。
let boundingSphere=Cesium.BoundingSphere.fromVertices(positions);
```



创建自定义几何的代码如下：

```js
// 自定义图元
// 将坐标点存储到一个数组中
const positions = new Float64Array([
  0,0,-1,// 0
  1,-1,1,// 1
  -1,-1,1,//2
  -1,1,1,// 3
  1,1,1,//  4,
])

// 按照一定的顺序组织索引值形成面
const indices = new Uint16Array([
  0, 1, 2,//前面
  0, 2, 3,//左面
  0, 3, 4,//后面
  0, 4, 1,//右面
  1, 3, 2,//上面
  1, 4, 3,//上面
]);

// 获取包围盒
let boundingSphere=Cesium.BoundingSphere.fromVertices(positions);
// 定义参考系
let m = Cesium.Transforms.eastNorthUpToFixedFrame(Cesium.Cartesian3.fromDegrees(110.0, 30.0, 10)); 
// 创建几何体对象
let geometry = new Cesium.Geometry({
  attributes: {
    position: new Cesium.GeometryAttribute({
      componentDatatype: Cesium.ComponentDatatype.DOUBLE,
      componentsPerAttribute: 3,
      values: positions // 顶点坐标数据
    }) 
  },
  indices: indices, // 索引数据
  primitiveType: Cesium.PrimitiveType.TRIANGLES,
  boundingSphere:boundingSphere,
});

const instance = new Cesium.GeometryInstance({
  geometry: geometry,
  modelMatrix: m,
  attributes: {
    color: Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.RED)
  }
});

viewer.scene.primitives.add(new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.PerInstanceColorAppearance({
    flat: true, // 平面着色
    translucent: true
  }),
  asynchronous: false
}))

// 相机飞行到指定位置
viewer.camera.flyTo({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 50.0),
  duration: 0.3
});
```

![image-20260203145915949](./assets/image-20260203145915949.png)



##### 法线

菱角不分明，这是因为没有为Geometry的顶点设置正确的法线，在图形学中顶点的法线是所有与它有关的面的法线的加和的单位化向量。

但是这里有个问题：

```
顶点0（金字塔尖端）被4个三角形共享：
  - 三角形1 (0,1,2) 的法线
  - 三角形2 (0,2,3) 的法线
  - 三角形3 (0,3,4) 的法线
  - 三角形4 (0,4,1) 的法线

这4个面的法线方向差别很大（因为金字塔是尖角）
↓
系统计算顶点0的法线 = 这4个面法线的平均值
↓
平均后的法线既不垂直于面1，也不垂直于面2、3、4
↓
光照计算错误 → 渲染效果不理想

❌ 错误方式（索引共享）：
   法线 = (面1法线 + 面2法线 + 面3法线 + 面4法线) / 4
   结果：这个平均法线不垂直于任何一个面

所以要取消索引的方式共享顶点，按照每个三角面3个顶点来构造Geometry顶点数组，这样求取每个三角面每个顶点到的法线，渲染出来的结果才会理想

✅ 正确方式（不共享）：
   面1的3个顶点各有不同的法线
   面2的3个顶点各有不同的法线
   ↓
   每个面的三个顶点法线都垂直于该面
   
   
计算法线方法：
Cesium.GeometryPipeline.computeNormal(geometry) → Geometry
自动计算几何体的顶点法线
1.找出与每个顶点相关的所有三角形面
2.计算每个三角形面的法线
3.将这些面法线平均化，得到顶点法线
4.将结果作为 normal 属性添加到几何体中
由于我这里每个面的顶点都不共享，所以顶点法线就是面的法线
```



computeNormal方法计算法线的代码

```js
// 自定义图元
// 6个三角形 × 3个顶点 = 18个顶点（不共享！）
const positions = new Float64Array([
  //前
  0, 0, -1,//  0
  1, -1, 1,//  1
  -1, -1, 1,// 2
  //左
  0, 0, -1,//  0 
  -1, -1, 1,// 2
  -1, 1, 1,//  3
  //后
  0, 0, -1,//  0 
  -1, 1, 1,//  3
  1, 1, 1,//   4
  //右
  0, 0, -1,//  0 
  1, 1, 1,//   4
  1, -1, 1,//  1
  //上
  1, -1, 1,//  1
  1, 1, 1,//   4
  -1, 1, 1,//  3
  //上
  1, -1, 1,//  1
  -1, 1, 1,//  3
  -1, -1, 1,// 2
]);
// 添加顺序索引（18个顶点对应 0-17）
const indices = new Uint16Array([
  0, 1, 2,      // 三角形1
  3, 4, 5,      // 三角形2
  6, 7, 8,      // 三角形3
  9, 10, 11,    // 三角形4
  12, 13, 14,   // 三角形5
  15, 16, 17,   // 三角形6
]);

// 获取包围盒
let boundingSphere=Cesium.BoundingSphere.fromVertices(positions);
// 定义参考系
let m = Cesium.Transforms.eastNorthUpToFixedFrame(Cesium.Cartesian3.fromDegrees(110.0, 30.0, 10)); 
// 创建几何体对象
let geometry = new Cesium.Geometry({
  attributes: {
    position: new Cesium.GeometryAttribute({
      componentDatatype: Cesium.ComponentDatatype.DOUBLE,
      componentsPerAttribute: 3,
      values: positions // 顶点坐标数据
    }) 
  },
  indices: indices, // 索引数据
  primitiveType: Cesium.PrimitiveType.TRIANGLES,
  boundingSphere:boundingSphere,
});

// ✅ 计算法线（现在每个顶点都有正确的法线）
Cesium.GeometryPipeline.computeNormal(geometry);

const instance = new Cesium.GeometryInstance({
  geometry: geometry,
  modelMatrix: m,
  attributes: {
    color: Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.RED)
  }
});

viewer.scene.primitives.add(new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.PerInstanceColorAppearance({
    flat: false, // 平滑着色
    translucent: true
  }),
  asynchronous: false
}))

// 相机飞行到指定位置
viewer.camera.flyTo({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 50.0),
  duration: 0.3
});
```

![image-20260203153842199](./assets/image-20260203153842199.png)

法线也可以按照如图原理手动计算：计算一个三角形的法线n，即为这个三角形内a向量与b向量的叉乘

![image-20260203153650825](./assets/image-20260203153650825.png)



```js
1. 直接计算每个三角形的法线
2. 赋值给该三角形的3个顶点
3. 完成！

// 优点：
// ✅ 计算更直接，无浪费，跳过了computeNormal平均化的计算过程
// ✅ 性能更好
// ✅ 代码逻辑清晰
// ✅ 对于不共享顶点的几何体最优



// ========== 手动计算法线（最优方案）==========

const positions = new Float64Array([
  // 前面
  0, 0, -1,   1, -1, 1,   -1, -1, 1,
  // 左面
  0, 0, -1,   -1, -1, 1,   -1, 1, 1,
  // 后面
  0, 0, -1,   -1, 1, 1,   1, 1, 1,
  // 右面
  0, 0, -1,   1, 1, 1,   1, -1, 1,
  // 上面1
  1, -1, 1,   1, 1, 1,   -1, 1, 1,
  // 上面2
  1, -1, 1,   -1, 1, 1,   -1, -1, 1,
]);

// ========== 手动计算所有三角形的法线 ==========

const normals = new Float32Array(54);  // 18个顶点 × 3个分量 = 54

// 辅助函数：计算三角形法线
function calculateTriangleNormal(v0, v1, v2) {
  // 创建两条边向量
  const d1 = Cesium.Cartesian3.subtract(v1, v0, new Cesium.Cartesian3());
  const d2 = Cesium.Cartesian3.subtract(v2, v0, new Cesium.Cartesian3());
  
  // 叉积得到法线
  let normal = Cesium.Cartesian3.cross(d1, d2, new Cesium.Cartesian3());
  
  // 单位化
  normal = Cesium.Cartesian3.normalize(normal, normal);
  
  return normal;
}

// 三角形数据（每个三角形3个顶点索引）
const triangles = [
  // 前面
  { v0: new Cesium.Cartesian3(0, 0, -1), v1: new Cesium.Cartesian3(1, -1, 1), v2: new Cesium.Cartesian3(-1, -1, 1), startIdx: 0 },
  // 左面
  { v0: new Cesium.Cartesian3(0, 0, -1), v1: new Cesium.Cartesian3(-1, -1, 1), v2: new Cesium.Cartesian3(-1, 1, 1), startIdx: 9 },
  // 后面
  { v0: new Cesium.Cartesian3(0, 0, -1), v1: new Cesium.Cartesian3(-1, 1, 1), v2: new Cesium.Cartesian3(1, 1, 1), startIdx: 18 },
  // 右面
  { v0: new Cesium.Cartesian3(0, 0, -1), v1: new Cesium.Cartesian3(1, 1, 1), v2: new Cesium.Cartesian3(1, -1, 1), startIdx: 27 },
  // 上面1
  { v0: new Cesium.Cartesian3(1, -1, 1), v1: new Cesium.Cartesian3(1, 1, 1), v2: new Cesium.Cartesian3(-1, 1, 1), startIdx: 36 },
  // 上面2
  { v0: new Cesium.Cartesian3(1, -1, 1), v1: new Cesium.Cartesian3(-1, 1, 1), v2: new Cesium.Cartesian3(-1, -1, 1), startIdx: 45 },
];

// 计算每个三角形的法线，赋值给该三角形的3个顶点
triangles.forEach(tri => {
  const normal = calculateTriangleNormal(tri.v0, tri.v1, tri.v2);
  
  // 将相同的法线赋值给三角形的3个顶点
  normals[tri.startIdx] = normal.x;
  normals[tri.startIdx + 1] = normal.y;
  normals[tri.startIdx + 2] = normal.z;
  
  normals[tri.startIdx + 3] = normal.x;
  normals[tri.startIdx + 4] = normal.y;
  normals[tri.startIdx + 5] = normal.z;
  
  normals[tri.startIdx + 6] = normal.x;
  normals[tri.startIdx + 7] = normal.y;
  normals[tri.startIdx + 8] = normal.z;
});
```



##### 可视化法线

```js
createTangentSpaceDebugPrimitive(options) → Primitive
创建一个primitive，来可视化几何体的切线空间（法线、切线、副切线）
参数：
  geometry：Geometry 包含顶点属性的几何体对象
  length：number 每条线段的长度（米），负数表示反向,默认值为 10000.0
  modelMatrix：Matrix4 定义几何体位置和方向的变换矩阵,默认值是Matrix4.IDENTITY
```

```js
// ...existing code...
viewer.scene.primitives.add(new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.PerInstanceColorAppearance({
    flat: false, // 平滑着色
    translucent: true
  }),
  asynchronous: false
}))

// 创建法线可视化图元
const normalDebugPrimitive = Cesium.createTangentSpaceDebugPrimitive({
  geometry: geometry,           // 使用同一个几何体
  length: 0.5,                  // 法线长度0.5米（便于查看）
  modelMatrix: m                // 使用同样的变换矩阵
});
viewer.scene.primitives.add(normalDebugPrimitive);
// ...existing code...
```





<img src="./assets/image-20260203160647445.png" alt="image-20260203160647445" style="zoom:67%;" />



### Appearance

appearance用来设置primitive几何图形的外观，不同的appearance适应于不同的场景，在使用时需要注意

```js
一种适用于任意几何体的外观，支持使用材质进行着色

new Cesium.MaterialAppearance({
  flat: false,        // 默认false,当选为true时，片段着色器使用平面着色(flat shading),意味着不考虑光照效果
  faceForward: false, // 默认与closed属性的值相反,当选为true时，片段着色器会根据需要翻转表面法线，确保法线朝向观察者以避免暗斑。当几何体两侧均需着色时（如 WallGeometry）此功能尤为有用。
  translucent: true,  // 默认true,当选为true时，几何体应呈现半透明效果，因此 MaterialAppearance#renderState 会启用 alpha 混合。
  closed: false,      // 默认false,当选为true时，何体需为闭合结构，MaterialAppearance#renderState将启用背面剔除。
  materialSupport: ...,          // 可选支持材质的类型
  material: Material.ColorType,  // 默认Material.ColorType,用于决定片段着色器颜色的材质
  vertexShaderSource: `...`,     // 可选的GLSL顶点着色器源，用于覆盖默认顶点着色器
  fragmentShaderSource: `...`,   // 可选的GLSL片段着色器源，用于覆盖默认片段着色器
  renderState: {...},            // 可选的渲染状态，用于覆盖默认渲染状态
})
```



```js
EllipsoidSurfaceAppearance是MaterialAppearance的一个版本，主要应用于那些于地球表面平行的几何类型，比如PolygonGeometry、RectangleGeometry，不适用于与地表垂直的几何类型，比如WallGeometry。并且其支持所有材质，如同MaterialAppearance搭配MaterialSupport.ALL属性。然而，由于片段着色器可程序化计算法线、切线和双切线，该外观所需的顶点属性更少。

new Cesium.EllipsoidSurfaceAppearance(options)
```





```js
适用于带颜色属性的几何体实例的外观。这允许使用相同的基元绘制多个几何体实例，每个实例具有不同颜色
new Cesium.PerInstanceColorAppearance(options)
```

![image-20260305195842262](./assets/image-20260305195842262.png)



```
PolylineMaterialAppearance：专门用于线的材质外观，支持传入material。
new Cesium.PolylineMaterialAppearance(options)
```

![image-20260305200910348](./assets/image-20260305200910348.png)



```js
PolylineColorAppearance：该appearance用于具有颜色属性的集合示例和多线几何/底面多线几何，使每个实例有不同的颜色，但是却基于相同的实例绘制

PolylineGometry支持根据顶点颜色进行着色，需要结合该类。
new Cesium.PolylineColorAppearance(options)
```

![image-20260305160334608](./assets/image-20260305160334608.png)





“椭球面上的几何体”

```js
这里的椭球面指的是：代替地球自然表面的地球椭球体


✓ 椭球面上的几何体（贴地面）
多边形、矩形、墙体等**紧贴地球表面**的几何体

PolygonGeometry      → 多边形（贴地表）
RectangleGeometry    → 矩形（贴地表）
WallGeometry         → 墙体（沿地表）


✗ 空中的几何体（不贴地面）
立方体、球体等**悬浮在空中**的几何体

BoxGeometry          → 立方体（空中）
SphereGeometry       → 球体（空中）
CylinderGeometry     → 圆柱体（空中）
```



#### Material

`new Cesium.Material(options)`

一个材质例子

```js
静态方法 Cesium.Material.fromType(type, uniforms) → Material
使用现有的材质类型创建一个新的材质实例。

type : String
材质类型的名称。例如，"Color"、"Image"、"DiffuseMap"、"Grid"、"Water" 等

uniforms : Object
包含该材质类型所需的具体配置参数。不同的 type 需要不同的 uniforms

让一个primitive有单一颜色
const material = Cesium.Material.fromType('Color', {
  color: new Cesium.Color(1.0, 0.0, 0.0, 1.0)
});
```





##### Fabric基本

Fabric 是一个 JSON 格式的规范，用来在 Cesium 中描述材质。内部被转换为GLSL

```js
两种应用材质的方式：

用内置材质类型（简写）
polygon.material = Cesium.Material.fromType('Color');

提供完整的 Fabric JSON（灵活）
polygon.material = new Cesium.Material({
  fabric: {
    type: 'Color'  // ← 指定材质类型
  }
});
虽然写法更长，但这样可以添加自定义的 uniforms 和 source 代码
```

Uniforms = 材质的输入参数

```js
1.创建时指定初始值
2.运行时动态修改

创建时设置 uniforms
polygon.material = new Cesium.Material({
  fabric: {
    type: 'Color',
    uniforms: {
      color: new Cesium.Color(1.0, 0.0, 0.0, 0.5)  // ← 半透明红色
    }
  }
});
这里的 color 是 Color 材质类型的一个 uniform

运行时修改 uniforms
// 创建后动态修改颜色
polygon.material.uniforms.color = Cesium.Color.WHITE;  // 改为不透明白色
```





##### 内置材质类型全览

###### Color、Image

```js
1. 基础材质（Built-In Materials）

材质类型	说明				   关键 uniform
Color	  纯色（包含透明度）	      color (Cesium.Color)
Image	  图片贴图（支持透明通道）	image (URL 或 Data URI)


polygon.material = Material.fromType('Image');
polygon.material.uniforms.image = 'image.png';

or

polygon.material = new Cesium.Material({
  fabric : {
    type : 'Image',
    uniforms : {
      image : 'image.png'
    }
  }
});
```

![image-20260127192217450](./assets/image-20260127192217450.png)



```js
2. 程序纹理材质（Procedural Textures）

特点： 不需要外部图片文件，在 GPU 上实时计算图案。

材质类型		 效果						常用参数
Checkerboard	棋盘格（黑白交替）		 lightColor, darkColor, repeat
Stripe			条纹（横向或纵向）		 lightColor, darkColor, repeat, orientation
Dot			    点阵图案				lightColor, darkColor, repeat
Grid		    网格线（用于 3D 体积）	  color, cellAlpha, lineCount

在这个页面搜参数就行：
https://cesium.com/learn/cesiumjs/ref-doc/Material.html

// 棋盘格
const material1 = new Cesium.Material({
  fabric: {
    type: 'Checkerboard',
    uniforms: {
      lightColor: Cesium.Color.ORANGE,
      darkColor: Cesium.Color.RED,
      repeat: { x: 10, y: 10 }  // 10x10 的棋盘格
    }
  }
});
```

<img src="./assets/image-20260127192734859.png" alt="image-20260127192734859" style="zoom:50%;" />

###### 光照相关

```js
3. 基础贴图材质（Base Materials）
特点： 用于表现光照细节，通常组合使用。

材质类型		说明						数据格式
DiffuseMap		漫反射贴图（物体基础颜色）	vec3 (RGB)
SpecularMap		高光贴图（光滑度/反光）	 标量 (单通道)
AlphaMap		透明度贴图				标量 (单通道)
NormalMap		法线贴图（表面细节凹凸）	vec3 (RGB)
BumpMap			凹凸贴图（高度差异）		 标量 (单通道)
EmissionMap		自发光贴图（如灯光）		 vec3 (RGB)

https://github.com/CesiumGS/cesium/wiki/Fabric#base-materials
```



###### 线条专用

```
4. 线条专用材质（Polyline Materials）
材质类型		效果
PolylineArrow	箭头（线末端）
PolylineGlow	发光线条
PolylineOutline	带边框的线条
```

<img src="./assets/image-20260127193401279.png" alt="image-20260127193401279" style="zoom: 67%;" />

###### 水、边缘光照

```js
5. 其他材质（Misc Materials）
材质类型	  效果
Water		 动态水面（波浪、涟漪）
RimLighting	 边缘光照（轮廓高亮）

const material1 = new Cesium.Material({
  fabric: {
    type: 'Water',
    uniforms: {
      normalMap: 'path/to/waterNormals.jpg',  // 需要法线贴图
      frequency: 1000.0,
      animationSpeed: 0.01,
      amplitude: 10.0
    }
  }
});
```

![image-20260127193501678](./assets/image-20260127193501678.png)



###### 常用Uniforms 参数

```js
image 参数
// 文件路径
material.uniforms.image = 'image.png';

// Data URI（内嵌 Base64 编码）
material.uniforms.image = 'data:image/png;base64,iVBORw0KG...';
```



```js
2. channel 参数（通道选择）
控制从图片的哪个颜色通道读取数据：

值	说明
'r'	红色通道
'g'	绿色通道
'b'	蓝色通道
'a'	Alpha 透明度通道
'rgb'	RGB 三通道（默认）

示例： 从 Alpha 通道读取高光数据
fabric: {
  type: 'SpecularMap',
  uniforms: {
    image: 'specular.png',
    channel: 'a'  // 使用 Alpha 通道而非默认的 Red 通道
  }
}

RGB 通道 → 漫反射贴图
Alpha 通道 → 高光贴图
```





```js
3. repeat 参数（纹理重复）
控制图片在表面上的平铺次数：

fabric: {
  type: 'DiffuseMap',
  uniforms: {
    image: 'diffuse.png',
    repeat: {
      x: 10,  // 水平重复 10 次
      y: 2    // 垂直重复 2 次
    }
  }
}

效果： 小图片可以平铺覆盖大表面（节省内存）。
```



##### 图片通道（Channel）

可以这样理解：通道数就是像素点的颜色分量数，也可以称为光谱波段数目，读取哪个通道数据就相当于把哪个光谱波段的光强信息作为参数使用

```js
常见图片格式的通道组成
格式		 通道数		通道说明
RGB		  3 个通道		Red（红）、Green（绿）、Blue（蓝）
RGBA	  4 个通道		RGB + Alpha（透明度）
灰度图		1 个通道	  只有一个亮度值（0=黑，255=白）


从不同通道读取数据的区别
举个例子：一张 RGBA 图片
假设某个像素点的颜色值为：(255, 128, 0, 200)

通道		  值		  如果只读这个通道会得到什么？ 相当于灰度图，越高越亮
R（红色）	 255	 得到一个亮度值 1.0（归一化后） 
G（绿色）	 128	 得到一个亮度值 0.5
B（蓝色）	 0		 得到一个亮度值 0.0
A（透明度）	200		得到一个亮度值 0.78



通常使用RGB 通道 → 漫反射，Alpha 通道 → 高光

在glsl中如何被用到的：
// 材质属性
vec3 diffuse = vec3(1.0, 0.5, 0.2);
//             ↓     ↓    ↓    ↓
//            类型   R    G    B

// 对应图片的 RGB 通道
图片像素 = (255, 128, 51)  // 橙色
         ↓  转换到 0-1 范围
GLSL表示 = (1.0, 0.5, 0.2)


// 材质属性
float specular = 0.8;
//    ↓         ↓
//  类型      亮度值

// 对应图片的单通道
灰度图像素 = 204  // (范围 0-255)
          ↓
GLSL表示 = 0.8   // (范围 0.0-1.0)



传统方式（两张图片）
diffuse.png  (RGB 通道)  → 物体基础颜色
specular.png (R 通道)    → 高光强度
```



对应表

| 材质属性  | GLSL 类型 | 对应图片通道       | 存储内容       | 举例                         |
| --------- | --------- | ------------------ | -------------- | ---------------------------- |
| diffuse   | `vec3`    | **RGB**            | 漫反射颜色     | `vec3(1.0, 0.0, 0.0)` = 红色 |
| emission  | `vec3`    | **RGB**            | 自发光颜色     | `vec3(1.0, 1.0, 0.0)` = 黄光 |
| normal    | `vec3`    | **RGB**            | 法线向量 (xyz) | `vec3(0.0, 0.0, 1.0)` = 向上 |
| specular  | `float`   | **单通道（灰度）** | 高光强度       | `0.8` = 80%高光              |
| shininess | `float`   | **单通道（灰度）** | 高光锐度       | `5.0` = 聚焦5倍              |
| alpha     | `float`   | **Alpha 通道**     | 透明度         | `0.5` = 半透明               |









##### 创建新 Materials

新材质通过Fabric、少量GLSL代码以及其他材质（如有需要）来创建。

若材质无需重复使用，则可不指定类型直接创建。

```js
 1：临时材质（无 type）
如果材质只用一次，可以不指定 type：
const fabric = {
  // 没有 type 属性
  components: {
    diffuse: 'vec3(1.0, 0.0, 0.0)'  // 红色
  }
};

polygon.material = new Cesium.Material({
  fabric: fabric
});
```



当使用不存在的材质类型时，首次调用 new Cesium.Material 时该材质会被缓存。后续调用 new Cesium.Material 或 Material.fromType 时，可像引用内置材质那样直接调用该材质——即无需提供完整的 Fabric 代码，仅需指定类型及需要设置的统一变量即可。

```js
方式 2：可复用材质（指定 type）
如果材质会被多次使用，应该指定 type 进行缓存：
const fabric = {
  type: 'MyNewMaterial',  // ← 材质名称（全局唯一）
  components: {
    diffuse: 'vec3(1.0, 0.0, 0.0)'
  }
};

// 第一次创建（注册材质）
polygon.material = new Cesium.Material({
  fabric: fabric
});

// 后续使用（直接引用）
anotherPolygon.material = Material.fromType('MyNewMaterial');
✅ 可复用（像内置材质一样使用）
✅ 材质被缓存，性能更好
```





###### components定义材质外观

```js
方法 1：使用 components 属性（简单）
components 是一个包含材质属性的对象，每个属性值是 GLSL 代码片段。

components属性包含定义材质外观的子属性,每个子属性的值均为GLSL代码片段。比如vec3(0.5)即创建各分量值均为 0.5 的3维向量

属性名		  类型	默认值			说明
diffuse	     vec3	vec3(0.0)	 漫反射颜色（物体基础颜色，光线均匀散射）
specular	float	0.0			镜面反射强度（高光亮度，0=无高光，1=最亮）
shininess	float	1.0			镜面反射锐度（值越大，镜面高光越小越集中）
normal		vec3	表面法线	 法线方向（用于法线贴图等效果）
emission	vec3	vec3(0.0)	定义材质向各个方向均匀发出的光线。默认值为vec3(0.0)，表示不发光。
alpha		float	1.0			透明度（0=完全透明，1=完全不透明）


https://github.com/CesiumGS/cesium/wiki/Fabric#components
```



```js
1.纯白色漫反射材质
const whiteMaterial = new Cesium.Material({
  fabric: {
    components: {
      diffuse: 'vec3(1.0)'  // RGB 都是 1.0 → 白色
    }
  }
});
GLSL 代码解释：
vec3(1.0) 是 GLSL 的向量构造函数
等价于 vec3(1.0, 1.0, 1.0)
表示 RGB 三个通道都是最大值（白色）
```

<img src="./assets/image-20260127203520939.png" alt="image-20260127203520939" style="zoom: 50%;" />



稍复杂些的例子会添加镜面反射组件，使得当直视下方时材质反射的光线最为强烈，而侧面观察时光线强度逐渐减弱。

```js
const grayShinyMaterial = new Cesium.Material({
  fabric: {
    components: {
      diffuse: 'vec3(0.5)',   // 灰色（RGB 都是 0.5）
      specular: '0.1'         // 镜面反射强度
    }
  }
});
```







###### shaderSource

components 属性的替代方案，通过为`czm_getMaterial`函数提供完整的GLSL源代码，达成了更灵活的使用方式

由两部分组成

uniforms：统一值，用于向着色器内传递数据

shaderSource：材质着色器代码，默认shaderSource如下：

```glsl
GLSL 函数声明，结构如下：
[返回类型] [函数名称]([参数类型] [参数名称])
    
默认的材质着色器代码如下（就是Fabric）：
czm_material czm_getMaterial(czm_materialInput materialInput)
{
    // 返回默认材质
    return czm_getDefaultMaterial(materialInput);
}
函数会返回一个 czm_material 结构体，输入参数类型为czm_materialInput，参数名称为materialInput
```



```glsl
函数签名
struct czm_materialInput {
  float s;                  // 纹理坐标 s (0-1)
  vec2 st;                  // 纹理坐标 (s, t) - 2D
  vec3 str;                 // 纹理坐标 (s, t, r) - 3D
  mat3 tangentToEyeMatrix;  // 用于法线贴图、凹凸贴图等的片段切线空间到视点坐标的转换矩阵
  vec3 positionToEyeEC;     // 用于反射、折射等的片段到视点的视点坐标向量。模值为片段到视点的距离
  vec3 normalEC;            // 该片段的法线（归一化）在视点坐标系中，用于凹凸贴图、反射、折射等效果。
};

    
纹理坐标不一定是子集关系
举例（椭球体）：
s 可能表示从底部到顶部（0.0 到 1.0）
st 可能表示经纬度坐标
str 可能表示包围盒轴向坐标


struct czm_material {
  vec3 diffuse;    // 漫反射颜色
  float specular;  // 镜面反射强度
  float shininess; // 高光锐度
  vec3 normal;     // 法线
  vec3 emission;   // 自发光
  float alpha;     // 透明度
};
```



```glsl
fabric: {
  components: {
    diffuse: 'vec3(0.5)',   // 灰色（RGB 都是 0.5）
    specular: '0.1'         // 镜面反射强度
  }
}
等价于：
czm_material czm_getMaterial(czm_materialInput materialInput)
{
  czm_material m = czm_getDefaultMaterial(materialInput);
  m.diffuse = vec3(0.5);
  m.specular = 0.5;
  return m;
}


这上面的纹理float s、vec2 st、vec3 str就是指的使用当前几何体表面的纹理坐标来设置颜色。
比如平时使用漫反射设置物体基础颜色：
diffuse: 'vec3(1.0, 0.0, 0.0)' // 红色
    
把纹理坐标当变量来设置颜色，比如：
diffuse: 'vec3(materialInput.st.s, 0.0, 0.0)'
纹理坐标向右为S轴，向上为T轴，范围都是0-1。这里的效果就是从左往右颜色从黑色变为红色的渐变效果。
vec3(materialInput.st.s, 0.0, 0.0)
// 分解为：
// R = materialInput.st.s  (范围 0.0 ~ 1.0，随纹理坐标S轴变化)
// G = 0.0                  (始终为 0)
// B = 0.0                  (始终为 0)
```



shaderSource实现蓝色立方体

```js
// 创建一个立方体几何体，定义其尺寸
const box = Cesium.BoxGeometry.fromDimensions({ 
  dimensions: new Cesium.Cartesian3(500.0, 500.0, 500.0), 
});

// 创建一个几何实例，包含几何数据和相关属性。
const instance = new Cesium.GeometryInstance({
  geometry: box,

  // 模型矩阵，定义几何体在世界坐标中的位置和方向
  modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500) 
  ), 
});

// 将几何体实例添加到场景的原始图元集合中
viewer.scene.primitives.add(new Cesium.Primitive({
  geometryInstances: instance,
  appearance: new Cesium.MaterialAppearance({
    material: new Cesium.Material({
        fabric: {
          type: "MyColorMaterial",  // 自定义材质类型名称
          uniforms: {
            color_0: new Cesium.Color(0.0, 0.0, 1.0, 1.0)  // 对应 shader 中的 uniform vec4 color_0
          },
          source: `
            uniform vec4 color_0; // 从 JavaScript 传入的颜色值
            czm_material czm_getMaterial(czm_materialInput materialInput) {
              czm_material material = czm_getDefaultMaterial(materialInput);
              material.diffuse = color_0.rgb; // 设置漫反射颜色
              material.alpha = color_0.a;
              return material;
            }
          `
        }
    }),
    renderState: {
      cull: { enabled: true }  // 启用背面剔除，否则会造成光线穿过前表面→内表面的光照差异（法线方向不同）→可以看到内部棱边
    }
  })
}));

等效于：
fabric: {
  type: "Color",
  uniforms: {
    color: Cesium.Color.BLUE
  }
}
```

![image-20260306100216120](./assets/image-20260306100216120.png)



一定要开启背面剔除，对于自定义材质，cesium没法根据透明度自动进行背面剔除，否则会形成下面的效果

![image-20260306100504859](./assets/image-20260306100504859.png)





```js
几何体里设置参数：vertexFormat: Cesium.VertexFormat.POSITION_AND_ST
说明这个几何体包含位置和纹理坐标

这里以一个平面几何体（PlaneGeometry）为例来说明：
const material1 = new Cesium.Material({
  fabric: {
    type: 'VisualizeUV',
    components: {
      diffuse: 'vec3(materialInput.st.s, materialInput.st.t, 0.5)', 
    }
  }
});
```

效果：

<img src="./assets/image-20260128103153685.png" alt="image-20260128103153685" style="zoom:50%;" />

```
左下角 = 黑蓝 (0, 0, 0.5)
右下角 = 红蓝 (1, 0, 0.5)
左上角 = 绿蓝 (0, 1, 0.5)
右上角 = 黄蓝 (1, 1, 0.5) (红+绿+蓝)

计算过程：
📍 左下角 (s=0, t=0)
R = materialInput.st.s = 0.0
G = materialInput.st.t = 0.0
B = 0.5

RGB = (0.0, 0.0, 0.5)
     ↓
浏览器解析：红色 0%  + 绿色 0%  + 蓝色 50%
     ↓
颜色：深蓝色 ■

其它三个角同理

📍 右下角 (s=1, t=0)
R = materialInput.st.s = 1.0  ⬅️ 横向移动到最右，红色通道最大
G = materialInput.st.t = 0.0
B = 0.5

📍 左上角 (s=0, t=1)
R = materialInput.st.s = 0.0
G = materialInput.st.t = 1.0  ⬅️ 纵向移动到最上，绿色通道最大
B = 0.5

📍 右上角 (s=1, t=1)
R = materialInput.st.s = 1.0  ⬅️ 横向最右
G = materialInput.st.t = 1.0  ⬅️ 纵向最上
B = 0.5
```





```js
最简单的 source 实现
const sourceMaterial = new Cesium.Material({
  fabric: {
    source: `
      czm_material czm_getMaterial(czm_materialInput materialInput) {
        return czm_getDefaultMaterial(materialInput);
      }
    `
  }
});
czm_getDefaultMaterial() 返回默认材质（全黑，不透明）
```

<img src="./assets/image-20260127204211522.png" alt="image-20260127204211522" style="zoom:67%;" />











###### 材质组合

```js
通过 materials 属性，可以将多个现有材质组合成新材质，形成材质层级。

{
  type: 'NewMaterial',
  materials: {
    subMaterial1: { /* Fabric 定义 */ },
    subMaterial2: { /* Fabric 定义 */ }
  },
  components: {
    diffuse: 'subMaterial1.diffuse',   // 引用子材质的属性
    specular: 'subMaterial2.specular'
  }
}




示例 ：塑料材质（DiffuseMap + SpecularMap）
// ...existing code...

const material1 = new Cesium.Material({
  fabric: {
    type: 'OurMappedPlastic',
    materials: {
      // 子材质 1：漫反射贴图
      diffuseMaterial: {
        type: 'DiffuseMap'
      },
      // 子材质 2：高光贴图
      specularMaterial: {
        type: 'SpecularMap'
      }
    },
    components: {
      diffuse: 'diffuseMaterial.diffuse',    // 从子材质提取 diffuse
      specular: 'specularMaterial.specular'  // 从子材质提取 specular
    }
  }
});

// 设置子材质的图片
material1.materials.diffuseMaterial.uniforms.image = './src/data/diffuseMap.png';
material1.materials.specularMaterial.uniforms.image = './src/data/specularMap.png';
```









##### Fabric Schema（JSON 规范）

```js
{
  // ① 材质类型（可选，用于缓存复用）
  type: 'MaterialTypeName',
  
  // ② 外部参数（从 JS 传入着色器）
  uniforms: {
    uniformName: value
  },
  
  // ③ 子材质（组合其他材质）
  materials: {
    subMaterialName: { /* Fabric 定义 */ }
  },
  
  // ④ 材质组件（简单方式定义外观）
  components: {
    diffuse: 'glsl code',
    specular: 'glsl code',
    // ...
  },
  
  // ⑤ 完整着色器代码（复杂方式定义外观）
  source: 'czm_material czm_getMaterial(czm_materialInput materialInput) { ... }'
}
```



#### vertexShaderSource 顶点

Appearance默认的vertexShaderSource

```glsl
in vec3 position3DHigh;   // 模型坐标系下的顶点位置的高位部分
in vec3 position3DLow;    // 模型坐标系下的顶点位置的低位部分

// 输入的法线和纹理
in vec3 normal;
in vec2 st;

in float batchId;         // 每个顶点所属的批次ID

// 输出参数（输出到片元着色器）
out vec3 v_positionEC;    // 输出顶点在相机坐标系中的位置
out vec3 v_normalEC;      // 输出顶点在相机坐标系中的法线
out vec2 v_st;            // 输出顶点的纹理坐标

void main() {
  vec4 p = czm_computePosition();
  v_positionEC = (czm_modelViewRelativeToEye * p).xyz;      // position in eye coordinates
  v_normalEC = czm_normal * normal;                         // normal in eye coordinates
  v_st = st;
  gl_Position = czm_modelViewProjectionRelativeToEye * p;
}
```



具体例子：

```js
// 创建一个立方体几何体，定义其尺寸
const box = Cesium.BoxGeometry.fromDimensions({ 
  dimensions: new Cesium.Cartesian3(500.0, 500.0, 500.0), 
});

// 几何实例
const instance = new Cesium.GeometryInstance({
  geometry: box,

  // 模型矩阵，定义几何体在世界坐标中的位置和方向
  modelMatrix: Cesium.Transforms.eastNorthUpToFixedFrame(
    Cesium.Cartesian3.fromDegrees(110.0, 30.0, 500) 
  ), 
});

const appearance = new Cesium.MaterialAppearance({
  vertexShaderSource: `
    in vec3 position3DHigh;
    in vec3 position3DLow;
    in vec3 normal;
    in vec2 st;
    in float batchId;

    out vec3 v_positionEC;
    out vec3 v_normalEC;
    out vec3 v_color;
    void main() {
      vec3 positionModel = position3DHigh + position3DLow;

      // 如果顶点的z坐标大于0，设置v_color为红色，否则设置为绿色
      if(positionModel.z > 0.0){
        v_color=vec3(1.0, 0.0, 0.0);
      }else{
        v_color=vec3(0.0, 1.0, 0.0);
      }

      // 将模型坐标转换为裁剪坐标并输出到gl_Position
      gl_Position = czm_modelViewProjection * vec4(positionModel, 1.0);  
    }
  `,

  fragmentShaderSource: `
    in vec3 v_color;
    void main() {
      out_FragColor = vec4(v_color, 0.5);
    }
  `
})


const primitive = new Cesium.Primitive({
  geometryInstances: instance,
  appearance: appearance,
});

viewer.scene.primitives.add(primitive);
```

![image-20260306114109192](./assets/image-20260306114109192.png)



```js
	虽然界限是以z=0为分界线进行突变的，但是顶点着色器赋值颜色时、是以顶点为单位进行的，而不是以片段为单位进行的，因为立方体只有8个顶点，上半部分的4个顶点z值都大于0，所以它们的颜色都是红色，而下半部分的4个顶点z值都小于0，所以它们的颜色都是绿色，而顶点着色器的out变量都会在光栅化阶段被GPU线性插值后传递给片段着色器，所以产生了颜色渐变效果

如果想做颜色突变，可以传递模型坐标系下的z坐标到片段着色器做分支判断，实现真正的锐利边界

修改appearance如下：
const appearance1 = new Cesium.MaterialAppearance({
  vertexShaderSource: `
    in vec3 position3DHigh;
    in vec3 position3DLow;
    in vec3 normal;
    in vec2 st;
    in float batchId;

    out vec3 v_positionEC;
    out vec3 v_normalEC;
    out float v_z;  // 输出z坐标到片段着色器
    void main() {
      vec3 positionModel = position3DHigh + position3DLow;
      v_z = positionModel.z; 
      gl_Position = czm_modelViewProjection * vec4(positionModel, 1.0);  
    }
  `,

  fragmentShaderSource: `
    in float v_z; // 从顶点着色器接收z坐标
    void main() {
      vec3 color = v_z > 0.0 ? vec3(1.0, 0.0, 0.0) : vec3(0.0, 1.0, 0.0); // 根据z坐标选择颜色
      out_FragColor = vec4(color, 0.5);
    } 
  `
})
```

![image-20260306114512516](./assets/image-20260306114512516.png)





#### fragmentShaderSource 片段

Appearance默认的fragmentShaderSource如下：

```glsl
// 接收从顶点着色器传递过来的变量
in vec3 v_positionEC;		// 片段在相机坐标系中的位置
in vec3 v_normalEC;			// 片段在相机坐标系中的法线向量
in vec2 v_st;			    // 纹理坐标

void main()
{
  vec3 positionToEyeEC = -v_positionEC;		取负值就得到了从片段指向摄像机的向量，这就是观看方向

  vec3 normalEC = normalize(v_normalEC);     将法相向量长度变为1，但是方向不变

#ifdef FACE_FORWARD
  normalEC = faceforward(normalEC, vec3(0.0, 0.0, 1.0), -normalEC);  // 定义了 FACE_FORWARD 宏时才会执行，
    这里的入射向量是vec3(0.0, 0.0, 1.0)，Z正向，指向摄像机，
    参考法线是-normalEC,即片段的反向法线，
    如果两者的点积小于0，说明它们的夹角大于90度，说明片段的法线是朝向摄像机的，那么就返回normalEC，否则返回反向的法线-normalEC，这样就保证法线始终与摄像机方向的夹角不超过 90°，避免法线背离摄像机是导致的错误光线计算
#endif

  // 创建一个 czm_materialInput 结构体
  // normalEC：规范化后的法线（用于光照计算）
  // positionToEyeEC：观看方向（用于镜面反射、视差等）
  // st：纹理坐标（用于纹理采样）
      
  czm_materialInput materialInput;
  materialInput.normalEC = normalEC;
  materialInput.positionToEyeEC = positionToEyeEC;
  materialInput.st = v_st;
    
  // 调用 czm_getMaterial() 函数，根据materialInput计算材质属性，返回一个czm_material结构体，跟material中的shaderSource一样
  czm_material material = czm_getMaterial(materialInput);

#ifdef FLAT
  out_FragColor = vec4(material.diffuse + material.emission, material.alpha);  // 如果定义了FLAT这个宏，就只计算漫反射 + 自发光，不做光照计算，输出颜色 = (漫反射RGB + 自发光RGB, 透明度)
#else
  out_FragColor = czm_phong(normalize(positionToEyeEC), material, czm_lightDirectionEC);  // 否则调用 czm_phong() 函数进行Phong光照计算，最终颜色 = 环境光 + 漫反射 + 镜面反射 + 自发光
#endif
}
```

如果不需要使用Material材质，那在fragmentShaderSource中完全不用调用czm_getMaterial方法



#### 向着色器中传递数据

```js
let appearance = new Cesium.MaterialAppearance({
  vertexShaderSource: `...`,
  fragmentShaderSource: `
    ...
    uniform vec4 u_color;
    ...
  `,
});

// 设置 uniform 值
appearance.uniforms = {
  u_color: Cesium.Color.BLUE
}
```

必须在创建完Appearance对象后再设置uniforms属性，不能在构造函数中设置





#### cesium内置uniform变量

https://cesium.com/downloads/cesiumjs/releases/b28/Documentation/index.html

cesium内置uniform变量，无需定义，直接在着色器中使用：

```glsl
czm_modelViewProjection：一个自动生成的GLSL统一变量，表示一个4x4模型-视图-投影变换矩阵，用于将模型坐标转换为裁剪坐标。裁剪坐标是顶点着色器gl_Position输出的坐标系。
// GLSL declaration
uniform mat4 czm_modelViewProjection;
 
// Example
vec3 positionModel = position3DHigh + position3DLow;
gl_Position = czm_modelViewProjection * vec4(positionModel, 1.0);
将模型坐标转换为裁剪坐标并输出到gl_Position
```



```glsl
czm_computePosition：返回相对于视点的模型坐标位置
// 用法Example
vec4 p = czm_computePosition();  // 返回值 p = 顶点的ECEF坐标 - 相机的ECEF坐标

v_positionEC = (czm_modelViewRelativeToEye * p).xyz;

gl_Position = czm_modelViewProjectionRelativeToEye * p;

此时的坐标系的轴方向仍然是模型坐标系的朝向
    
只是把原点平移到了眼睛（相机）位置

本质上是：模型顶点世界坐标 - 相机世界坐标



czm_modelViewRelativeToEye：将"相对于眼睛的模型坐标" 变换到眼睛空间（Eye Space / View Space），把v_positionEC传给片元着色器，用于光照、雾效等计算
    
虽然v_positionEC和p点的位置都是以相机为原点，但是参考系自身的轴方向变了，内容如下：

    
世界坐标系                    相对Eye的模型坐标           眼睛坐标系(View Space)
(原点在地心)                  (原点移到相机,轴不变)        (原点在相机,轴也变了)

     Z↑                            Z↑                         Y↑  
     |                             |                           |   
     |     .(顶点)                  |  .(顶点偏移量)             |    
     |                             |                           +------→X
     +------→X            →        +------→X         →        /
    /                             /                          / 
   Y                             Y                          -Z(看向屏幕内)
   
   顶点坐标值巨大              坐标值变小(精度安全)         乘以View旋转矩阵
   float精度不够              但轴方向未变                 轴方向对齐相机
    
    
czm_modelViewProjectionRelativeToEye：用于将相对于视点的模型坐标转换为裁剪坐标
```



























































### GeometryInstance

```
new Cesium.GeometryInstance(options)
几何体实例化 - 将同一个几何体对象在不同位置，可以具有独特的颜色，实现高效批量渲染
参数：
  geometry: Geometry 要实例化的几何体
  modelMatrix: Matrix4 定义几何体位置和方向的变换矩阵,默认值是Matrix4.IDENTITY
  id: object 唯一标识符，用于拾取识别
  attributes: object 单实例属性（颜色、显示隐藏等）
```

工作原理

```
一个 Geometry（几何体定义）
        ↓
    创建多个 GeometryInstance（实例）
        ↓
    每个实例有不同的：
    - 位置（modelMatrix）
    - 颜色（attributes.color）
    - ID（用于拾取）
        ↓
    打包成一个 Primitive（图元）
        ↓
    一次渲染多个对象，性能高效！
```



单实例属性：

```js
attributes: {
  // 显示隐藏
  show: Cesium.ShowGeometryInstanceAttribute.fromBoolean(true),
  
  // 颜色
  color: Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.RED),
  
  // 自定义属性
  customProperty: new Cesium.GeometryInstanceAttribute({
    componentDatatype: Cesium.ComponentDatatype.FLOAT,
    componentsPerAttribute: 1,
    value: [1.0]
  })
}
```

































## 事件

### 鼠标点击交互

```js
// 添加点击事件处理
new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas).setInputAction(e => {
    let pick = viewer.scene.pick(e.position);
    if (pick && pick.id) {
        console.log('点击到了：', pick.id);  // 输出：BoxGeometry0, BoxGeometry1 等
    }
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);




new Cesium.ScreenSpaceEventHandler(element)
处理用户输入事件，可以添加自定义函数，在用户输入内容时执行这些函数
参数：
  element: HTMLCanvasElement画布元素，用来给这个元素添加事件

  
  

setInputAction(action, type, modifier)
为画布上的用户输入事件（鼠标、键盘、触摸等）绑定处理函数
参数：
  action: 事件回调函数，输入事件发生时执行的函数
  type: 事件类型
    Cesium.ScreenSpaceEventType.LEFT_CLICK        // 左键单击
    Cesium.ScreenSpaceEventType.RIGHT_CLICK       // 右键单击
    Cesium.ScreenSpaceEventType.MIDDLE_CLICK      // 中键单击
    Cesium.ScreenSpaceEventType.MOUSE_MOVE        // 鼠标移动
    Cesium.ScreenSpaceEventType.WHEEL             // 滚轮滚动
    Cesium.ScreenSpaceEventType.LEFT_DOWN         // 左键按下
    Cesium.ScreenSpaceEventType.LEFT_UP           // 左键抬起
  modifier：按键修饰符，Ctrl、Shift等
    Cesium.KeyboardEventModifier.SHIFT    // Shift 键
    Cesium.KeyboardEventModifier.CTRL     // Ctrl 键
    Cesium.KeyboardEventModifier.ALT      // Alt 键


viewer.scene中的pick:
pick(windowPosition, width, height) → object|undefined
在场景中的指定窗口坐标进行拾取，返回该位置的**第一个（最上层）**图元或 undefined。
参数：
  windowPosition: Cartesian2 窗口坐标（像素）
  width: number 拾取矩形的宽度（像素）默认值为1
  height: number 拾取矩形的高度（像素）默认值为1
返回值：
  object|undefined: 返回拾取到的图元对象，如果没有拾取到则返回 undefined
```







### 紧凑速查

| 名称                                                | 作用                                               | 典型构造（核心片段）                                         |
| --------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| ScreenSpaceEventHandler                             | 监听鼠标 / 触摸屏幕输入<br />assadfdsafasdfasdfsff | `const handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas)` |
| setInputAction                                      | 绑定具体输入事件回调                               | `handler.setInputAction(cb, Cesium.ScreenSpaceEventType.MOUSE_MOVE)` |
| ScreenSpaceEventType                                | 输入事件类型枚举                                   | `Cesium.ScreenSpaceEventType.LEFT_CLICK / MOUSE_MOVE / RIGHT_CLICK` |
| KeyboardEventModifier                               | 键盘组合修饰                                       | `{ eventType: Cesium.CameraEventType.LEFT_DRAG, modifier: Cesium.KeyboardEventModifier.CTRL }` |
| ScreenSpaceCameraController<br />.tiltEventTypes 等 | 重定义相机交互手势                                 | `ssc.tiltEventTypes = [ Cesium.CameraEventType.RIGHT_DRAG,{ eventType: Cesium.CameraEventType.LEFT_DRAG, modifier: CTRL } ]` |
| CameraEventType                                     | 相机手势类型枚举                                   | `Cesium.CameraEventType.LEFT_DRAG / RIGHT_DRAG / WHEEL / MIDDLE_DRAG` |
| Scene 生命周期事件                                  | 帧内渲染阶段监听                                   | `scene.preUpdate.addEventListener(fn)` 等四种：preUpdate / postUpdate / preRender / postRender |
| camera.changed 事件                                 | 相机视图参数变化时触发                             | `viewer.camera.changed.addEventListener(fn); viewer.camera.percentageChanged = 0.01` |
| 取消事件监听                                        | 释放资源                                           | `event.removeEventListener(fn)`                              |
| 拾取射线 getPickRay                                 | 屏幕点转射线                                       | `const ray = viewer.camera.getPickRay(windowPos)`            |
| globe.pick                                          | 射线与地球求交获取地表位置                         | `const cartesian = viewer.scene.globe.pick(ray, viewer.scene)` |
| SceneTransforms.<br />worldToWindowCoordinates      | 世界坐标转屏幕坐标                                 | `Cesium.SceneTransforms.worldToWindowCoordinates(scene, position)` |
|                                                     |                                                    |                                                              |
| Popup（位置跟随）模式                               | 相机变化时更新屏幕定位                             | 监听 `camera.changed` → 世界转屏幕 → 设置 `div.style.left/top` |
|                                                     |                                                    |                                                              |
| CallbackProperty                                    | 让几何属性随输入动态更新                           | `positions: new Cesium.CallbackProperty(() => currentPositions, false)` |
| ConstantProperty                                    | 固化最终结果，避免结束后消失                       | `polyline.positions = new Cesium.ConstantProperty(fixedPositions)` |
|                                                     |                                                    |                                                              |
| Camera.computeViewRectangle                         | 计算当前相机视域矩形                               | `viewer.camera.computeViewRectangle(globe.ellipsoid)`        |
| Camera.setView(Rectangle)                           | 设置相机于一个矩形范围                             | `camera.setView({ destination: Cesium.Rectangle.fromDegrees(w,s,e,n) })` |
| camera.changed 事件                                 | 主相机变化时驱动鹰眼同步                           | `viewer.camera.changed.addEventListener(syncFn)`             |
|                                                     |                                                    |                                                              |
| 笛卡尔转经纬度 (Cartographic.fromCartesian)         | 拆解点的经纬高，辅助构造矩形四角                   | `Cesium.Cartographic.fromCartesian(p)`                       |
| fromDegrees / fromDegreesArrayHeights               | 经纬度数组转 Cartesian3 数组                       | `Cesium.Cartesian3.fromDegreesArrayHeights(arr)`             |
| getPickRay + globe.pick                             | 屏幕坐标拾取地表点                                 | `globe.pick(camera.getPickRay(pos), scene)`                  |
| worldToWindowCoordinates                            | 世界坐标转屏幕坐标（如需 DOM 叠加）                | `Cesium.SceneTransforms.worldToWindowCoordinates(scene, p)`  |
| PolylineDashMaterialProperty                        | 虚线材质用于动态折线                               | `material: new Cesium.PolylineDashMaterialProperty({ color })` |







对象层级关系

```
Cesium.Viewer (viewer)
├─ scene: Cesium.Scene
│  ├─ canvas: HTMLCanvasElement       // 场景的渲染画布
│  ├─ globe: Cesium.Globe             // 地球对象（用于 pick）
│  ├─ camera: Cesium.Camera           // 摄像机（也可直接 viewer.camera 访问）
│  ├─ screenSpaceCameraController: Cesium.ScreenSpaceCameraController   // 相机控制事件
│  │  └─ tiltEventTypes / zoomEventTypes / rotateEventTypes 等事件配置
│  ├─ preUpdate: Cesium.Event  → (场景状态准备之前，你可改属性)
│  ├─ postUpdate: Cesium.Event  → (场景状态已更新，尚未生成渲染命令)
│  ├─ preRender: Cesium.Event  → (命令列表生成前最后调参数 / 相机)
│  ├─ postRender: Cesium.Event  → (一帧图像完成，可收集统计 / UI 通知)
├─ camera: Cesium.Camera              // 与 scene.camera 指向同一对象
├─ entities: Cesium.EntityCollection  // 场景中的高层抽象数据
└─ ... 其它 (timeline, imageryLayers 等)
```



屏幕控制事件 `screenSpaceCameraController`细分

```
ScreenSpaceEventHandler (绑定一个 Canvas)
├─ setInputAction(action, type, [modifier])
├─ removeInputAction(type, [modifier])
└─ destroy()


   
设置事件setInputAction(action, type, modifier)
action: function，事件触发时调用的函数，参数为event
type: Cesium.ScreenSpaceEventType，鼠标事件类型,
modifier: 可选，Cesium.KeyboardEventModifier 键盘修事件修饰符
```







### 工作流步骤

一般事件

1. 选择输入来源
   - 需要鼠标/触摸：创建 `ScreenSpaceEventHandler`
   - 需要相机交互自定义：获取 `viewer.scene.screenSpaceCameraController`
   - 需要帧阶段执行逻辑：用 `scene.*Render` 或 `scene.*Update`
   - 需要视图变化驱动（依赖相机位置方向）：用 `camera.changed`
2. 明确事件颗粒度
   - 离散动作（单击/右键结束）：`LEFT_CLICK` / `RIGHT_CLICK`
   - 连续跟踪（移动、拖拽）：`MOUSE_MOVE` 或 相机手势 `LEFT_DRAG` 等
   - 高频更新（视图跟随 UI）：`camera.changed`（调低 `percentageChanged`）
3. 若需要地理位置
   - 屏幕 → 地球：`getPickRay` → `globe.pick`
   - 已有经纬度 → 世界：`Cartesian3.fromDegrees`
   - 世界 → 屏幕：`SceneTransforms.worldToWindowCoordinates`
4. 组合键修饰
   - 需要限定 Ctrl/Shift：在相机事件类型对象中加 `modifier` 或在 `setInputAction` 第三个参数传入修饰符
5. 创建或更新可视对象
   - 点/标记：`viewer.entities.add({ point })`
   - DOM 气泡：附加到 `viewer.cesiumWidget.container`，由转换结果驱动位置
6. 清理与销毁
   - 停止绘制：状态位改为 false
   - 移除监听：`removeEventListener`
   - DOM 清理：`parent.removeChild(el)`
   - 避免泄漏：销毁自定义类时确保移除事件





交互式折线绘制

1. 初始化状态数组：`positions=[]`（已定点）、`currentPositions=[]`（动态尾点）
2. 左键点击：拾取世界坐标 → 推入 `positions` 与 `currentPositions` → 若是首点则创建折线实体（`CallbackProperty`）
3. 鼠标移动：若已有首点 → 拾取移动终点 → `currentPositions = [...positions, movingPoint]`
4. 右键结束：长度>1 → 克隆 `positions` 固化为 `ConstantProperty` → 重置临时数组





交互式矩形绘制

1. 状态机：`IDLE` / `STARTED`；数据：`positions`（原始点击点） + `currentPositions`（两端点）
2. 左键第一次（IDLE）：记录首点 → 创建矩形实体（polygon + polyline 都用 `CallbackProperty`）→ 状态转 `STARTED`
3. 鼠标移动（STARTED）：实时更新第二点 → 通过工具类计算四角点数组 → 回调驱动显示
4. 左键第二次（STARTED）：校验点合法 → 固化 polygon 的 `hierarchy` 与 polyline 的 `positions` → 重置状态
5. 右键（STARTED）：移除临时实体 → 重置（取消绘制）





鹰眼（Overview / MiniMap）

1. 创建独立 viewer（2D，禁用交互）
2. 主相机监听：`camera.changed` → `computeViewRectangle` 获取当前主视域
3. 将弧度矩形转度数 → 构造 `Rectangle.fromDegrees`
4. 调用鹰眼相机 `setView({ destination: rect })` 实时同步



### 最小可执行

viewer

```ts
// 假设 HTML 中已有 <div id="cesiumContainer"></div>
Cesium.Ion.defaultAccessToken = "YOUR_TOKEN";
const viewer = new Cesium.Viewer("cesiumContainer", {
  timeline: false,
  animation: false,
  infoBox: false,
  sceneModePicker: false,
  navigationHelpButton: false,
} as any);
```



自定义相机拖拽倾斜手势

```ts
// -------- 相机交互重定义 ----------
const ssc = viewer.scene.screenSpaceCameraController;
ssc.tiltEventTypes = [
  Cesium.CameraEventType.RIGHT_DRAG,
  { eventType: Cesium.CameraEventType.LEFT_DRAG, modifier: Cesium.KeyboardEventModifier.CTRL },
];
```



```ts
// -------- Scene 生命周期事件 ----------
function preRender() {
  // 每帧渲染前逻辑（轻量）
  // console.log("preRender");
}
viewer.scene.preRender.addEventListener(preRender);
```



左键点击开始并随着鼠标移动在地表连续绘制红色点（右键结束），

```ts
// -------- 鼠标绘点 ----------
const handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
let isDraw = false;

handler.setInputAction(() => {
  isDraw = true;
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);

handler.setInputAction(() => {
  isDraw = false;
}, Cesium.ScreenSpaceEventType.RIGHT_CLICK);

handler.setInputAction((movement) => {
  if (!isDraw) return;
  const ray = viewer.camera.getPickRay(movement.endPosition);
  const cartesian = viewer.scene.globe.pick(ray, viewer.scene);
  if (cartesian) {
    viewer.entities.add({
      position: cartesian,
      point: { pixelSize: 8, color: Cesium.Color.RED },
    });
  }
}, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
```



弹框，气泡框跟随，利用camera.changed事件

创建一个蓝色锚点及其跟随相机视角实时定位的 DOM 气泡框

```ts
// -------- Popup 跟随 ----------
class Popup {
  container: HTMLDivElement | null = null;
  position: Cesium.Cartesian3 | null = null;
  offset = new Cesium.Cartesian2(0, -10);
  constructor(private v: Cesium.Viewer) {}

  add(position: Cesium.Cartesian3, html: string) {
    this.position = position;
    this.container = document.createElement("div");
    this.container.className = "popup";
    this.container.style.position = "absolute";
    this.container.innerHTML = html;
    this.v.cesiumWidget.container.appendChild(this.container);

    this.v.camera.percentageChanged = 0.01;
    this.v.camera.changed.addEventListener(this.update, this); // camera.changed事件
    this.update();
  }

  update() {
    if (!this.position || !this.container) return;
    const win = Cesium.SceneTransforms.worldToWindowCoordinates(this.v.scene, this.position);
    if (!win) return;
    this.container.style.left = win.x - this.container.offsetWidth / 2 + this.offset.x + "px";
    this.container.style.top = win.y - this.container.offsetHeight + this.offset.y + "px";
  }

  destroy() {
    if (this.container) {
      this.v.cesiumWidget.container.removeChild(this.container);
      this.v.camera.changed.removeEventListener(this.update, this);
      this.container = null;
    }
  }
}

const anchor = Cesium.Cartesian3.fromDegrees(114, 30);
viewer.entities.add({ position: anchor, point: { pixelSize: 10, color: Cesium.Color.BLUE } });
viewer.camera.flyTo({ destination: Cesium.Cartesian3.fromDegrees(114, 30, 500) });

const popup = new Popup(viewer);
popup.add(anchor, "<b>测试气泡</b>");
```



```ts
// -------- 清理示例（可在需要时调用） ----------
function cleanup() {
  viewer.scene.preRender.removeEventListener(preRender);
  handler.destroy();
  popup.destroy();
}
```







主场景中实现交互式折线与矩形动态绘制（左键构造、移动实时反馈、右键结束/取消），并通过一个禁用交互的鹰眼小地图实时同步主相机当前视域范围。

```ts
import * as Cesium from "cesium";

// 假设已有 <div id="main"></div> 与页面可追加 overview 容器
const viewer = new Cesium.Viewer("main", {
  animation: false, timeline: false, infoBox: false, sceneModePicker: false,
} as any);

class PositionHandler {
  constructor(private v: Cesium.Viewer) {}
  screenToCartesian3(pos: Cesium.Cartesian2) {
    const ray = this.v.camera.getPickRay(pos);
    return ray ? (this.v.scene.globe.pick(ray, this.v.scene) || null) : null;
  }
  getRectFourPoints(positions: Cesium.Cartesian3[]) {
    if (!Array.isArray(positions) || positions.length < 2) return [];
    const [p1, p2] = positions;
    const c1 = Cesium.Cartographic.fromCartesian(p1);
    const c2 = Cesium.Cartographic.fromCartesian(p2);
    if (c1.height < 0) c1.height = 0;
    if (c2.height < 0) c2.height = 0;
    const d = (c: Cesium.Cartographic) => [Cesium.Math.toDegrees(c.longitude), Cesium.Math.toDegrees(c.latitude)];
    const a = d(c1), b = d(c2);
    const lngLatArr = [
      ...a, c1.height,
      a[0], b[1], c1.height,
      b[0], b[1], c1.height,
      b[0], a[1], c2.height,
      ...a, c1.height
    ];
    return Cesium.Cartesian3.fromDegreesArrayHeights(lngLatArr);
  }
}
const ph = new PositionHandler(viewer);

// -------- 折线绘制 --------
const lineHandler = new Cesium.ScreenSpaceEventHandler(viewer.canvas);
let linePositions: Cesium.Cartesian3[] = [];
let currentLine: Cesium.Cartesian3[] = [];
let lineEntity: Cesium.Entity | undefined;

lineHandler.setInputAction((e: any) => {
  const c = ph.screenToCartesian3(e.position);
  if (!c) return;
  linePositions.push(c);
  currentLine.push(c);
  if (linePositions.length === 1) {
    lineEntity = viewer.entities.add({
      polyline: {
        positions: new Cesium.CallbackProperty(() => currentLine, false),
        width: 3,
        material: new Cesium.PolylineDashMaterialProperty({ color: Cesium.Color.YELLOW })
      }
    });
  }
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);

lineHandler.setInputAction((e: any) => {
  if (linePositions.length === 0) return;
  const c = ph.screenToCartesian3(e.endPosition);
  if (!c) return;
  currentLine = [...linePositions, c];
}, Cesium.ScreenSpaceEventType.MOUSE_MOVE);

lineHandler.setInputAction(() => {
  if (linePositions.length > 1 && lineEntity?.polyline) {
    const fixed = linePositions.map(p => p.clone());
    lineEntity.polyline.positions = new Cesium.ConstantProperty(fixed);
  }
  linePositions = [];
  currentLine = [];
}, Cesium.ScreenSpaceEventType.RIGHT_CLICK);

// -------- 矩形绘制 --------
const rectHandler = new Cesium.ScreenSpaceEventHandler(viewer.canvas);
const DrawStates = { IDLE: "IDLE", STARTED: "STARTED" } as const;
let rectState = DrawStates.IDLE;
let rectPositions: Cesium.Cartesian3[] = [];
let rectCurrent: Cesium.Cartesian3[] = [];
let rectEntity: Cesium.Entity | undefined;

function createRectEntity() {
  rectEntity = viewer.entities.add({
    polygon: {
      hierarchy: new Cesium.CallbackProperty(
        () => new Cesium.PolygonHierarchy(ph.getRectFourPoints(rectCurrent)), false
      ),
      material: Cesium.Color.RED.withAlpha(0.35),
      outline: true,
      outlineColor: Cesium.Color.YELLOW
    },
    polyline: {
      positions: new Cesium.CallbackProperty(() => ph.getRectFourPoints(rectCurrent), false),
      width: 2,
      material: new Cesium.PolylineDashMaterialProperty({ color: Cesium.Color.YELLOW })
    }
  });
}

function finalizeRect() {
  if (!rectEntity) return;
  const pts = ph.getRectFourPoints(rectCurrent);
  rectEntity.polygon!.hierarchy = new Cesium.PolygonHierarchy(pts);
  rectEntity.polyline!.positions = pts;
}

function resetRect() {
  rectPositions = [];
  rectCurrent = [];
  rectEntity = undefined;
  rectState = DrawStates.IDLE;
}

rectHandler.setInputAction((e: any) => {
  const c = ph.screenToCartesian3(e.position);
  if (!c) return;
  if (rectState === DrawStates.IDLE) {
    rectPositions.push(c);
    rectCurrent = [c];
    !rectEntity && createRectEntity();
    rectState = DrawStates.STARTED;
  } else if (rectState === DrawStates.STARTED) {
    rectPositions.push(c);
    rectCurrent = [rectPositions[0], c];
    finalizeRect();
    resetRect();
  }
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);

rectHandler.setInputAction((e: any) => {
  if (rectState !== DrawStates.STARTED) return;
  const c = ph.screenToCartesian3(e.endPosition);
  if (!c) return;
  rectCurrent = [rectPositions[0], c];
}, Cesium.ScreenSpaceEventType.MOUSE_MOVE);

rectHandler.setInputAction(() => {
  if (rectState === DrawStates.STARTED && rectEntity) {
    viewer.entities.remove(rectEntity);
    resetRect();
  }
}, Cesium.ScreenSpaceEventType.RIGHT_CLICK);

// -------- 鹰眼图（Overview） --------
const ovDiv = document.createElement("div");
ovDiv.id = "overview";
ovDiv.style.cssText = "position:absolute;right:10px;bottom:10px;width:220px;height:150px;border:2px solid #ff7800;z-index:999;";
document.body.appendChild(ovDiv);

const overviewViewer = new Cesium.Viewer("overview", {
  sceneMode: Cesium.SceneMode.SCENE2D,
  animation: false, timeline: false, infoBox: false, navigationHelpButton: false
} as any);

overviewViewer.scene.screenSpaceCameraController.enableRotate = false;
overviewViewer.scene.screenSpaceCameraController.enableTranslate = false;
overviewViewer.scene.screenSpaceCameraController.enableZoom = false;
overviewViewer.scene.screenSpaceCameraController.enableTilt = false;
overviewViewer.scene.screenSpaceCameraController.enableLook = false;

viewer.camera.percentageChanged = 0.02;
viewer.camera.changed.addEventListener(() => {
  const rect = viewer.camera.computeViewRectangle(viewer.scene.globe.ellipsoid);
  if (!rect) return;
  const west = Cesium.Math.toDegrees(rect.west);
  const south = Cesium.Math.toDegrees(rect.south);
  const east = Cesium.Math.toDegrees(rect.east);
  const north = Cesium.Math.toDegrees(rect.north);
  overviewViewer.camera.setView({
    destination: Cesium.Rectangle.fromDegrees(west, south, east, north)
  });
});

// 简单初始视角
viewer.camera.flyTo({ destination: Cesium.Cartesian3.fromDegrees(114, 30, 1200000) });
```









### 核心概念

```
事件层次分类
    输入事件（ScreenSpaceEventHandler + ScreenSpaceEventType）
    相机交互手势（ScreenSpaceCameraController + CameraEventType）
    场景生命周期（preUpdate / postUpdate / preRender / postRender）
    相机状态变化（camera.changed）
    
ScreenSpaceEventHandler 适合处理光标相关、离散或连续的屏幕动作；不直接改变相机，逻辑自行编写。

相机交互替换通过更改控制器的 xxxEventTypes 数组实现：可用枚举或对象形式（支持组合键修饰）。

camera.changed 适合“相机变化驱动 UI”场景（例如 DOM 标注实时跟随）。调低 percentageChanged 可提高刷新频率但增加开销。

生命周期事件与相机事件区别：生命周期事件基于渲染循环（无论相机是否变），camera.changed 仅在视图参数显著变化时触发。

地理拾取：屏幕 → 射线 → 地球求交（受地形/椭球可见性限制）；失败时需 null 判断。

世界坐标转屏幕坐标用于将 3D 对象与 2D DOM 关联；应在可见性与裁剪情况下做防护。

连续绘制需状态机（如 isDraw），避免在未激活状态下无效操作。

事件解绑与资源释放是自定义交互类（如 Popup）生命周期的必要部分，防止内存泄漏与重复回调。

修饰键（Ctrl/Shift/Alt）仅在需要限定交互路径时加入，过度使用会降低可用性。

高频事件中的复杂逻辑（如大量实体创建）应考虑节流或批处理，否则影响帧率。
```



```
两层数据结构分离：已确认点数组（positions）与动态实时点数组（current/rectCurrent）保证绘制过程与结果固化分离。

CallbackProperty 用于“绘制过程中”实时计算几何；结束后用 ConstantProperty 或直接赋值静态数组固化。

折线与矩形的“开始—进行—结束”逻辑采用最小状态机（无多余中间态）降低分支复杂度。

矩形基于两对角点推导四顶点：经纬度拆解（Cartographic）→ 组合角点 → fromDegreesArrayHeights。

拾取可靠性：屏幕到地表需判空；快速点击时可能得到同地点或不可见区（需校验或忽略）。

右键结束与右键取消区分：折线右键表示完成；矩形右键表示取消当前未完成绘制。

概念分层：输入事件（ScreenSpaceEventHandler）与视域同步事件（camera.changed）职责不同，一个驱动用户构造几何，一个驱动视图派生同步。

鹰眼图使用 computeViewRectangle 获得近似可见范围的西南东北弧度值，再转度重建 Rectangle 驱动 2D Viewer 相机。

防交互干扰：鹰眼 Viewer 禁用全部控制器使其纯展示。

性能权衡：camera.percentageChanged 过低提升刷新频率但增加计算次数；应在实时性与性能之间调节。

数据固化前克隆：避免引用同一个数组导致后续清空使最终结果消失。
```





### 决策树

```
需要实现的效果是什么？
│
├─ A. 响应一次性点击/按键？
│    └─ 使用 ScreenSpaceEventHandler + 对应 ScreenSpaceEventType（LEFT_CLICK / RIGHT_CLICK 等）
│
├─ B. 随鼠标移动持续反馈（绘制/拖拽轨迹）？
│    ├─ 是否与相机视角无关（纯屏幕操作） → MOUSE_MOVE + 状态判定
│    └─ 是否需要位置转成地理坐标 → getPickRay + globe.pick 成功才使用
│
├─ C. 修改默认相机手势？
│    └─ 获取 screenSpaceCameraController → 设置 tilt / zoom / rotate EventTypes
│           ├─ 需要组合键？使用对象 { eventType, modifier }
│           └─ 只需替换鼠标映射？直接用枚举数组
│
├─ D. 在每帧阶段执行逻辑？
│    ├─ 依赖渲染循环（例如统计、调试） → scene.preRender / postRender
│    └─ 依赖更新但渲染前数据准备 → preUpdate / postUpdate
│
├─ E. 只在视图改变时更新（例如标注跟随、视域相关）？
│    └─ camera.changed + 设置 percentageChanged 控制敏感度
│
├─ F. 需要将 3D 点与 DOM 绑定展示？
│    ├─ 已有世界坐标 → worldToWindowCoordinates → 定位 DOM
│    └─ 需要实时刷新 → 触发源选择：camera.changed（高效） 或 preRender（更平滑）
│
└─ G. 是否需要清理？
     ├─ 存在临时类/弹窗/批量事件 → removeEventListener / destroy
     └─ 有大量实体临时创建 → 记录引用以便统一移除
```



交互绘制与鹰眼图

```
目标：我要哪种交互效果？
│
├─ 需要用户依次点击形成多段路径？
│    ├─ 单点开始，多点延伸，移动显示末端实时变化 → 折线绘制
│    ├─ 数据结构：positions + currentLine
│    └─ 结束条件：右键且已有 ≥ 2 点 → 固化为 ConstantProperty
│
├─ 只需用两个点确定一个区域（矩形/拉框）？
│    ├─ 状态机：IDLE → STARTED → 完成或取消
│    ├─ 第一次左键：记录起点 + 创建矩形实体（CallbackProperty）
│    ├─ 鼠标移动：更新第二点 → 动态计算四角
│    ├─ 第二次左键：固化 polygon + polyline → 重置
│    └─ 右键：取消（删除临时实体）重置
│
├─ 需要实时查看主视图的地理覆盖范围？
│    ├─ 使用 camera.changed 监听
│    ├─ computeViewRectangle 获取视域
│    ├─ 禁用鹰眼交互（只展示）
│    └─ setView(Rectangle.fromDegrees(...)) 同步区域
│
├─ 屏幕坐标如何转世界点？
│    └─ getPickRay → globe.pick → 判空
│
├─ 世界点如何用于几何构造？
│    ├─ 折线：直接数组传 positions
│    └─ 矩形：两点 → Cartographic → 四角 → fromDegreesArrayHeights
│
├─ 何时用 CallbackProperty？
│    ├─ 需要持续响应鼠标移动或状态变化 → 使用
│    └─ 绘制结束不再变化 → 用 ConstantProperty 或直接赋值静态数组
│
├─ 如何判定结束 or 取消？
│    ├─ 折线：右键 = 结束（保留结果）
│    └─ 矩形：右键 = 取消（移除临时实体）
│
└─ 需要避免结果丢失？
     ├─ 在 reset 前克隆 positions
     └─ 固化后清空临时数组与状态
```







## 数据源

### czml数据



czml数据是一个数组，数组中每个元素是packet，就是一个对象{}，第一个packet放一些元数据，后面的packet以声明的方式描述了一个实体（entity）的性质，位置在哪，形状如何，有何标注

```
[ documentPacket, entityPacket1, entityPacket2, ... ]
```

示例代码：

```js
// ============================================================
//  CZML 是一个 JSON 数组，每个元素叫 packet（包）：
//    [0]   => document packet：固定格式，声明版本，必须有
//    [1..n] => entity packet：每个对应场景里一个实体，必须有唯一 id
//
//  CzmlDataSource 读取这个数组，自动创建对应的 Entity 对象。
// ============================================================


// ---------- 构造 CZML 数组 ----------
// --- packet 0: document（版本声明，永远是第一个）---
// --- packet 1: 一个静态实体（卫星A）---
// 唯一 id，是这个实体的"身份证"
// position 用 cartographicDegrees 格式: [经度, 纬度, 高度(米)]
// point: 在屏幕上渲染成一个像素圆点
// label: 文字标注
const czmlData = [
  {
    id: "document",
    version: "1.0",
  },
  {
    id: "sat-A",                 
    name: "卫星 A",
    position: {
      cartographicDegrees: [116.4, 39.9, 500000], 
    },
    point: {
      pixelSize: 14,
      color: { rgba: [255, 200, 0, 255] },         
      outlineColor: { rgba: [255, 255, 255, 200] },
      outlineWidth: 2,
    },
    label: {
      text: "卫星 A",
      font: "14pt sans-serif",
      fillColor: { rgba: [255, 255, 255, 255] },
      pixelOffset: { cartesian2: [0, -30] },   
    },
  },
];


// ---------- 加载 CZML ----------
// CzmlDataSource.load() 可以接受：
//   - 字符串 URL（指向 .czml 文件）
//   - 直接传入数组（开发调试最方便，就是现在这种写法）
const dataSource = new Cesium.CzmlDataSource();
await dataSource.load(czmlData);
viewer.dataSources.add(dataSource);

// 自动飞到所有实体
viewer.zoomTo(dataSource);
```





动态位置

`viewer.dataSources.add(dataSource)` 这一行会让 Cesium 自动把 CZML 里声明的 clock 同步覆盖到 viewer.clock 上，所以改动czml外部的multiplier无法修改场景时钟倍率

静态的点里，position里面只有一个三维坐标，如果是运动的点，里面就要添加一行采样点，采样点包含相对时间和对应的三维坐标，为什么需要时间，因为运动的本质就是“位置随着时间变化"，在t0到t1之间，cesium自动进行位置插值，实体点平滑移动，同时添加起始时刻作为相对时间的基准点

```js
// 静态
position: { cartographicDegrees: [116.4, 39.9, 500000] }

// 动态
position: {
  epoch: "起始时刻（ISO 8601）",
  cartographicDegrees: [
    t0, lon0, lat0, alt0,   // 第一个样本
    t1, lon1, lat1, alt1,   // 第二个样本（t 是相对 epoch 的秒数）
    ...
  ]
}
```

示例代码

```js
// ============================================================
//  核心新概念：
//    epoch     —— "时间零点"，ISO 8601 格式的字符串
//    cartographicDegrees（动态版）—— 每 4 个数为一组：
//                [秒偏移, 经度, 纬度, 高度, ...]
//    availability  —— 该实体"存在"的时间区间
//    clock（document 包里）—— 控制时间轴的起止和默认倍速
// ============================================================

// 用 JS 生成时间字符串，方便复用
const START = "2026-01-01T00:00:00Z";   // 仿真开始时刻
const END   = "2026-01-01T00:10:00Z";   // 仿真结束时刻（10分钟后）

const czmlData = [

  // --- packet 0: document ---
  // 多了一个 clock 字段，用来驱动底部时间轴
  {
    id: "document",
    version: "1.0",
    clock: {
      interval: `${START}/${END}`,   // 时间轴的总范围
      currentTime: START,            // 播放头初始位置
      multiplier: 60,                // 播放倍速：1秒现实 = 60秒仿真
      range: "LOOP_STOP",            // 到终点后停止（LOOP_STOP / CLAMPED / UNBOUNDED）
      step: "SYSTEM_CLOCK_MULTIPLIER",
    },
  },

  // --- packet 1: 卫星 A，从北京向东飞 ---
  {
    id: "sat-A",
    name: "卫星 A",

    // availability: 实体在哪段时间内可见
    // 格式：ISO 起始 / ISO 结束
    availability: `${START}/${END}`,

    position: {
      // epoch：采样序列的"时间零点"
      // 后面每个采样的时间都是相对于它的【秒偏移】
      epoch: START,

      // cartographicDegrees（动态版）：每 4 个数一组
      //   [相对epoch的秒偏移, 经度, 纬度, 高度(m)]
      cartographicDegrees: [
        0,    116.4, 39.9, 500000,   // 0秒时：北京正上方
        120,  118.4, 39.9, 500000,   // 2分钟后：向东偏移 2°
        240,  120.4, 39.9, 500000,   // 4分钟后：继续向东
        360,  122.4, 39.9, 500000,   // 6分钟后
        480,  124.4, 39.9, 500000,   // 8分钟后
        600,  126.4, 39.9, 500000,   // 10分钟后：结束
      ],

      // interpolationAlgorithm: 两个采样点之间如何插值
      // LAGRANGE（拉格朗日）比 LINEAR 更平滑
      interpolationAlgorithm: "LAGRANGE",
      interpolationDegree: 5,
    },

    // path: 显示卫星走过的轨迹线
    path: {
      show: true,
      width: 2,
      material: { solidColor: { color: { rgba: [255, 200, 0, 160] } } },
      // leadTime/trailTime: 向前/向后显示多少秒的轨迹（0 = 不显示未来）
      leadTime: 0,
      trailTime: 600,  // 显示过去 10 分钟的轨迹
    },

    point: {
      pixelSize: 14,
      color: { rgba: [255, 200, 0, 255] },
      outlineColor: { rgba: [255, 255, 255, 200] },
      outlineWidth: 2,
    },

    label: {
      text: "卫星 A",
      font: "14pt sans-serif",
      fillColor: { rgba: [255, 255, 255, 255] },
      pixelOffset: { cartesian2: [0, -30] },
    },
  },
];

// ---------- 加载 CZML ----------
const dataSource = new Cesium.CzmlDataSource();
await dataSource.load(czmlData);
viewer.dataSources.add(dataSource);

// 让 Viewer 的时钟与 DataSource 里声明的 clock 同步
viewer.clock.startTime   = Cesium.JulianDate.fromIso8601(START);
viewer.clock.stopTime    = Cesium.JulianDate.fromIso8601(END);
viewer.clock.currentTime = Cesium.JulianDate.fromIso8601(START);
viewer.clock.multiplier  = 60;
viewer.clock.clockRange  = Cesium.ClockRange.LOOP_STOP;

viewer.zoomTo(dataSource);
```





`process()` 增量注入 + `references` 连线

load通过解析第一个czml(packet数组)，得到一个类型为dataSource的数据，并将其作为viewer中的dataSources,此时可以把dataSource当作一个容器，此时通过process(另一个czml)，就可以将两个czml放到一个dataSource，对于在同一个dataSource的packet，相互之间可以通过`references:[实体id#属性名]`的方式引用其它packet中的属性，如果引用的属性是动态的，引用后也将保持动态

```js
DataSource（一个容器）
├── sat-A   ← load() 时建立
├── sat-B   ← process() 追加
└── link-AB ← process() 追加，positions.references 引用同容器内的 sat-A 和 sat-B
```

czml中的packet对象被解析成entity，放在viewer下面的_dataSources数组中的CZMLDataSource里，czml被解析后就以CZMLDataSource的方式放到viewer中，

viewer简化版结构如下

```js
const viewer = {
  dataSources: DataSourceCollection {
    _dataSources: [
      CustomDataSource: {'...'}
      CzmlDataSource:{
        '...'
        entities: EntityCollection {
          _entities: [
            Entity {id: 'e1',/ position / billboard / point / polyline / polygon ...},
            Entity {id: 'e1', '...'},
            '...'
          ]
        }
      },
      '...'
    ]
  }
}
```

示例代码

```js
// ============================================================
//  核心新概念：
//    process()    —— 向已有 DataSource 追加 packet，不清空原有实体
//    references   —— polyline 的端点不写死坐标，而是"引用"其他实体
//                    格式："实体id#属性名"，例如 "sat-A#position"
//                    Cesium 每帧读取被引用实体的当前位置 → 连线自动跟随
//
//  ⚠️ references 只在同一个 DataSource 内有效，跨 DataSource 会失效！
// ============================================================

const START = "2026-01-01T00:00:00Z";
const END   = "2026-01-01T00:10:00Z";

// ── 第一批：document + 卫星A ──────────────────────────────────
// 这是"初始包"，用 load() 建立 DataSource
const initialPackets = [
  {
    id: "document",
    version: "1.0",
    clock: {
      interval: `${START}/${END}`,
      currentTime: START,
      multiplier: 60,
      range: "LOOP_STOP",
      step: "SYSTEM_CLOCK_MULTIPLIER",
    },
  },
  {
    id: "sat-A",
    name: "卫星 A",
    availability: `${START}/${END}`,
    position: {
      epoch: START,
      cartographicDegrees: [
        0,   116.4, 39.9, 500000,
        600, 126.4, 39.9, 500000,   // 10分钟内向东飞 10°
      ],
      interpolationAlgorithm: "LAGRANGE",
      interpolationDegree: 1,
    },
    path: {
      show: true, width: 2,
      material: { solidColor: { color: { rgba: [255, 200, 0, 160] } } },
      leadTime: 0, trailTime: 600,
    },
    point: { pixelSize: 14, color: { rgba: [255, 200, 0, 255] } },
    label: {
      text: "卫星 A", font: "13pt sans-serif",
      fillColor: { rgba: [255, 255, 255, 255] },
      pixelOffset: { cartesian2: [0, -28] },
    },
  },
];

// ── 第二批：卫星B + 连线 ───────────────────────────────────────
// 用 process() 追加，与卫星A共享同一个 DataSource
const extraPackets = [
  // 卫星 B：从上海向东北飞，与卫星 A 轨迹相交
  {
    id: "sat-B",
    name: "卫星 B",
    availability: `${START}/${END}`,
    position: {
      epoch: START,
      cartographicDegrees: [
        0,   121.5, 31.2, 600000,   // 上海附近
        600, 126.4, 39.9, 600000,   // 10分钟后飞到与A相同的终点
      ],
      interpolationAlgorithm: "LAGRANGE",
      interpolationDegree: 1,
    },
    path: {
      show: true, width: 2,
      material: { solidColor: { color: { rgba: [0, 180, 255, 160] } } },
      leadTime: 0, trailTime: 600,
    },
    point: { pixelSize: 14, color: { rgba: [0, 180, 255, 255] } },
    label: {
      text: "卫星 B", font: "13pt sans-serif",
      fillColor: { rgba: [255, 255, 255, 255] },
      pixelOffset: { cartesian2: [0, -28] },
    },
  },

  // 连线 packet：polyline，两端"引用"卫星A和卫星B的实时位置
  {
    id: "link-AB",
    name: "A-B 连线",
    availability: `${START}/${END}`,
    polyline: {
      // ★ 核心：positions.references
      // 不写死坐标，而是引用其他实体的 position 属性
      // 格式数组：["实体id#属性名", "实体id#属性名"]
      // Cesium 每帧去读这两个实体的当前坐标，连线自动跟随移动
      positions: {
        references: ["sat-A#position", "sat-B#position"],
      },
      width: 2,
      material: {
        polylineDash: {
          // 虚线效果
          color: { rgba: [255, 255, 255, 200] },
          dashLength: 16,
        },
      },
    },
  },
];

// ── 加载流程 ──────────────────────────────────────────────────
const dataSource = new Cesium.CzmlDataSource();

// 第一步：load() 建立 DataSource，卫星A进入场景
await dataSource.load(initialPackets);
viewer.dataSources.add(dataSource);

// 第二步：process() 追加卫星B和连线
// 此时 sat-B 和 link-AB 与 sat-A 在同一个 DataSource 里
// references 中的 "sat-A#position" 可以被正确解析
await dataSource.process(extraPackets);

viewer.zoomTo(dataSource);
```





packet 处理算法

当客户端收到一个 packet 时，按以下步骤处理：

```Code
对于 packet 中的每个属性：
  └─ 对于该属性的每个 interval：
      └─ 判断该 interval 是否已存在
          ├─ 如果存在 → 更新现有 interval
          └─ 如果不存在 → 创建新 interval
```



示例

图示

![image-20260114102701022](C:\Users\HUST\AppData\Roaming\Typora\typora-user-images\image-20260114102701022.png)







#### 使用

加载czml

```js
// 加载 CZML
const dataSource = await Cesium.CzmlDataSource.load(czml);
await viewer.dataSources.add(dataSource);
```















### gltf模型动画(结合czml)

#### 模型阶段

现有模型

![image-20260127104220958](./assets/image-20260127104220958.png)

模型命名

![image-20260127104236406](./assets/image-20260127104236406.png)

这样打开gltf文件，里面的json代码会显示

![image-20260127104258440](./assets/image-20260127104258440.png)

导出设置为gltf嵌入式

![image-20260127104307822](./assets/image-20260127104307822.png)



#### 创建 GMDF 文件

**在gltf同级目录下新建 同名的gmdf文件 才能在后面正确注入**

- 定义了 `cone` 这个 articulation
- 包含 7 个 stages：3个平移 + 3个旋转 + 1个缩放
- `modelNodes: ["cone"]` 对应 glTF 中的节点名称



1. **`AGI_articulations.articulations`**
   - 定义哪些关节可以如何变换
   - 指定变换作用在哪些 glTF 节点上
2. **`extensions.AGI_articulations`**
   - 用于将关节信息嵌入到 glTF 文件中
   - 使得模型可以独立使用，不依赖外部 GMDF
3. **`extensionsUsed`**
   - 声明当前文件使用的扩展
   - 告知加载器需要什么支持

![image-20260127104609234](./assets/image-20260127104609234.png)

```json
{
  // ========== AGI_articulations 根级别定义 ==========
  // 这是 GMDF 文件的主体部分，定义了所有可动关节
  "AGI_articulations": {
    
    // 关节数组，每个元素代表一个可动部件
    "articulations": [
      {
        // 关节的唯一标识符
        // 后续在 CZML 中通过 "cone yRotate" 格式引用这个名称
        "name": "cone",
        
        // stages 数组：定义这个关节的所有自由度（能怎么动）
        // 每个 stage 对应一种变换（平移/旋转/缩放）
        "stages": [
          
          // ========== 平移变换 ==========
          {
            // stage 的显示名称
            "name": "xTranslate",
            // stage 的类型：xTranslate（沿 X 轴平移）
            // 其他可用类型：yTranslate, zTranslate, xRotate, yRotate, zRotate, uniformScale
            "type": "xTranslate",
            // 该 stage 的最小值（沿 X 轴向负方向最多移动 -100 单位）
            "minimumValue": -100,
            // 该 stage 的最大值（沿 X 轴向正方向最多移动 100 单位）
            "maximumValue": 100,
            // 该 stage 的初始值（启动时的默认状态）
            "initialValue": 0
          },
          
          {
            "name": "yTranslate",
            "type": "yTranslate",
            "minimumValue": -100,
            "maximumValue": 100,
            "initialValue": 0
          },
          
          {
            "name": "zTranslate",
            "type": "zTranslate",
            "minimumValue": -100,
            "maximumValue": 100,
            "initialValue": 0
          },
          
          // ========== 旋转变换 ==========
          {
            "name": "xRotate",
            "type": "xRotate",
            // 旋转角度范围：-360 到 360 度
            "minimumValue": -360,
            "maximumValue": 360,
            "initialValue": 0
          },
          
          {
            "name": "yRotate",
            "type": "yRotate",
            "minimumValue": -360,
            "maximumValue": 360,
            "initialValue": 0
          },
          
          {
            "name": "zRotate",
            "type": "zRotate",
            "minimumValue": -360,
            "maximumValue": 360,
            "initialValue": 0
          },
          
          // ========== 缩放变换 ==========
          {
            "name": "uniformScale",
            "type": "uniformScale",
            // 缩放比例：最小缩小到 0.1 倍，最大放大到 5 倍
            "minimumValue": 0.1,
            "maximumValue": 5,
            "initialValue": 1
          }
        ],
        
        // modelNodes 数组：指定哪些 glTF 节点受此 articulation 控制
        // 必须与 glTF 文件中的 nodes[].name 完全匹配
        // 示例：glTF 中有 nodes: [{name: "cone"}]，这里就写 ["cone"]
        "modelNodes": [
          "cone"
        ]
      }
      
      // 如果模型有多个可动部件，可以在这里添加更多 articulation 对象
      // 例如：
      // {
      //   "name": "arm",
      //   "stages": [...],
      //   "modelNodes": ["arm"]
      // },
      // {
      //   "name": "wheel",
      //   "stages": [...],
      //   "modelNodes": ["wheel"]
      // }
    ]
  },
  
  // ========== glTF 扩展定义（由 STK GMDF Tools 生成或手动添加）==========
  // 这个部分是为了让 glTF 文件本身也包含关节定义信息
  // 使得模型文件可以独立存在，不需要依赖外部 GMDF 文件
  "extensions": {
    "AGI_articulations": {
      // 关节数组（与上面的定义相同或相似）
      "articulations": [
        {
          "name": "cone",
          
          // 所有可用的 stages 定义
          "stages": [
            {
              "name": "xTranslate",
              "type": "xTranslate",
              "minimumValue": -100,
              "maximumValue": 100,
              "initialValue": 0
            },
            {
              "name": "yTranslate",
              "type": "yTranslate",
              "minimumValue": -100,
              "maximumValue": 100,
              "initialValue": 0
            },
            {
              "name": "zTranslate",
              "type": "zTranslate",
              "minimumValue": -100,
              "maximumValue": 100,
              "initialValue": 0
            },
            {
              "name": "xRotate",
              "type": "xRotate",
              "minimumValue": -360,
              "maximumValue": 360,
              "initialValue": 0
            },
            {
              "name": "yRotate",
              "type": "yRotate",
              "minimumValue": -360,
              "maximumValue": 360,
              "initialValue": 0
            },
            {
              "name": "zRotate",
              "type": "zRotate",
              "minimumValue": -360,
              "maximumValue": 360,
              "initialValue": 0
            },
            {
              "name": "uniformScale",
              "type": "uniformScale",
              "minimumValue": 0.1,
              "maximumValue": 5,
              "initialValue": 1
            }
          ]
          
          // 可选属性：指向向量（用于表示关节的指向方向）
          // "pointingVector": [0, 0, 1]
        }
      ]
    }
  },
  
  // ========== glTF 扩展声明 ==========
  // 声明当前文件使用了哪些扩展
  // 这告诉 glTF 加载器："我需要支持 AGI_articulations 扩展来正确加载这个模型"
  "extensionsUsed": [
    "AGI_articulations"
  ]
}
```



#### STK GMDF Tools 注入扩展



打开对应的gltf文件

vscode中打> 符号搜索下面这个选项

![image-20260127104704950](./assets/image-20260127104704950.png)

会自动为节点注入对应的动作，gltf文件中会多出这一部分

```json
在 nodes[0] 添加 extensions.AGI_articulations.articulationName
在根级别添加 extensions.AGI_articulations 完整定义
添加 extensionsUsed: ["AGI_articulations"]


// node[0]处
"nodes": [
    {
        "mesh": 0,
        "name": "cone",
        "extensions": {
            "AGI_articulations": {
                "articulationName": "cone"
            }
        }
    }
],


// 根级别
"extensions": {
    "AGI_articulations": {
        "articulations": [
            {
                "name": "cone",
                "stages": [
                    {
                        "name": "xTranslate",
                        "type": "xTranslate",
                        "minimumValue": -100,
                        "maximumValue": 100,
                        "initialValue": 0
                    },
                    {
                        "name": "yTranslate",
                        "type": "yTranslate",
                        "minimumValue": -100,
                        "maximumValue": 100,
                        "initialValue": 0
                    },
                    {
                        "name": "zTranslate",
                        "type": "zTranslate",
                        "minimumValue": -100,
                        "maximumValue": 100,
                        "initialValue": 0
                    },
                    {
                        "name": "xRotate",
                        "type": "xRotate",
                        "minimumValue": -360,
                        "maximumValue": 360,
                        "initialValue": 0
                    },
                    {
                        "name": "yRotate",
                        "type": "yRotate",
                        "minimumValue": -360,
                        "maximumValue": 360,
                        "initialValue": 0
                    },
                    {
                        "name": "zRotate",
                        "type": "zRotate",
                        "minimumValue": -360,
                        "maximumValue": 360,
                        "initialValue": 0
                    },
                    {
                        "name": "uniformScale",
                        "type": "uniformScale",
                        "minimumValue": 0.1,
                        "maximumValue": 5,
                        "initialValue": 1
                    }
                ]
            }
        ]
    }
},


"extensionsUsed": [
    "AGI_articulations"
]
```



打开对应的gltf文件，点击右侧预览按钮，需要安装glTF Tools



![image-20260127104738940](./assets/image-20260127104738940.png)



会发现gltf中插入的stages和预览场景中的变换调整窗口上的参数对应上了

![image-20260127104758638](./assets/image-20260127104758638.png)



此时模型的参考系发生了变化（y轴指向up导出）

![image-20260127104837573](./assets/image-20260127104837573.png)

原本

![image-20260127104844304](./assets/image-20260127104844304.png)



cesium中实现

```js
// 圆锥模型动画 - 加载函数
async function loadAnimatedCone() {
  // 动画时间参数
  const animationStartTime = '2023-01-01T00:00:00Z';
  const animationEndTime = '2023-01-01T00:00:20Z';

  // 第二步：定义圆锥的各种变换动画关键帧
  // 格式说明：关键帧数组 = [时间1, 值1, 时间2, 值2, ...]
  // 关键帧会在时间点之间自动插值，实现平滑动画
  const coneAnimations = {
    
    // 绕 Y 轴旋转（垂直轴旋转）
    'cone yRotate': [
      0,   0,       // 0秒：旋转角度为 0度
      5,   180,     // 5秒：旋转到 180度
      10,  360,     // 10秒：完成一圈（360度）
      15,  540,     // 15秒：继续旋转到 540度
      20,  720      // 20秒：两圈完成（720度）
    ],
    
    // 绕 X 轴旋转（前后倾斜）
    'cone xRotate': [
      0,   0,       // 0秒：0度
      5,   0,       // 5秒：保持 0度
      10,  45,      // 10秒：向前倾斜 45度
      15,  0,       // 15秒：回正（0度）
      20,  -45      // 20秒：向后倾斜 45度
    ],
    
    // 沿 X 轴平移（左右移动）
    'cone xTranslate': [
      0,   0,       // 0秒：原位（0）
      5,   50,      // 5秒：向右移动 50个单位
      10,  0,       // 10秒：回到原位
      15,  -50,     // 15秒：向左移动 50个单位
      20,  0        // 20秒：回到原位
    ],
    
    // 沿 Z 轴平移（上下移动）
    'cone zTranslate': [
      0,   0,       // 0秒：原位
      10,  20,      // 10秒：向上移动 20个单位
      20,  0        // 20秒：回到原位
    ],
    
    // 缩放动画（uniformScale = 统一缩放）
    'cone uniformScale': [
      0,   1,       // 0秒：正常大小（1倍）
      5,   0.5,     // 5秒：缩小到一半（0.5倍）
      10,  1,       // 10秒：恢复正常（1倍）
      15,  2,       // 15秒：放大到两倍（2倍）
      20,  1        // 20秒：恢复正常（1倍）
    ]
  };


  // 第三步：构建 CZML 数据结构
  // 第 3.1 步：定义 CZML 文档头
  // 作用：配置时间轴、时钟参数等全局设置
  const czmlDocument = {
    id: "document",
    name: "Cone Articulation Animation",
    version: "1.0",
    clock: {
      // 时间轴区间（从开始时间到结束时间）
      interval: `${animationStartTime}/${animationEndTime}`,
      // 当前时间（动画开始时刻）
      currentTime: animationStartTime,
      // 播放速度倍数（1.0 = 正常速度）
      multiplier: 1,
      // 播放模式：LOOP_STOP = 循环播放
      range: "LOOP_STOP",
      // 时间步长：SYSTEM_CLOCK_MULTIPLIER = 使用系统时钟
      step: "SYSTEM_CLOCK_MULTIPLIER"
    }
  };

  // 第 3.2 步：定义模型基础配置
  // 作用：指定模型路径、位置、缩放等基本参数
  const modelEntityConfig = {
    id: "coneModel",
    name: "Animated Cone",
    // 可用时间范围（在这个时间段内模型才会显示）
    availability: `${animationStartTime}/${animationEndTime}`,
    // 模型在地球上的位置（经度, 纬度, 高度(km)）
    position: {
      cartographicDegrees: [116.39, 39.9, 1000]  // 北京上空 1000km
    },
    // 模型渲染配置
    model: {
      gltf: 'src/data/模型动画实验.gltf',  // glTF模型文件路径
      scale: 1000.0,                        // 缩放比例（放大1000倍便于观察）
      minimumPixelSize: 128                 // 最小像素大小
    }
  };


  // 第 3.3 步：为模型添加 articulation（关节动画）
  // 作用：将关键帧数据与模型实体关联，实现动画
  const modelEntity = {
    ...modelEntityConfig,
    model: {
      ...modelEntityConfig.model,
      // articulations 对象：key 是"节点名 + stage类型"，value 是关键帧数据
      articulations: {}
    }
  };
  // 遍历每个动画，将其添加到 articulations 中
  for (const [articulationName, keyframes] of Object.entries(coneAnimations)) {
    modelEntity.model.articulations[articulationName] = {
      epoch: animationStartTime,  // 关键帧的基准时间
      number: keyframes            // 关键帧数据数组 [时间1, 值1, 时间2, 值2, ...]
    };
  }


  // 第 3.4 步：组合完整的 CZML 数据
  // CZML 是一个数组，第一个元素是文档配置，后续元素是各种实体（模型、路径等）
  const czml = [czmlDocument, modelEntity];


  // 第四步：加载 CZML 到 Cesium 场景
  // 解析 CZML 并创建数据源
  const dataSource = await Cesium.CzmlDataSource.load(czml);
  // 将数据源添加到 viewer 中，使其显示在场景中
  await viewer.dataSources.add(dataSource);


  // 第五步：配置相机跟踪和视角
  // 获取圆锥模型实体
  const coneEntity = dataSource.entities.getById('coneModel');
  
  if (coneEntity) {
    // 设置相机跟踪圆锥（相机会自动跟随模型）
    viewer.trackedEntity = coneEntity;
    
    // 延迟 100ms 后执行相机动画，避免实体还未完全初始化
    setTimeout(() => {
      viewer.camera.flyTo({
        // 相机目标位置（经度, 纬度, 高度(km)）
        destination: Cesium.Cartesian3.fromDegrees(116.39, 39.9, 3000),
        // 相机朝向（航向角、俯仰角、横滚角）
        orientation: {
          heading: Cesium.Math.toRadians(0),    // 0° = 正北方向
          pitch: Cesium.Math.toRadians(-80),    // -80° = 向下看80度（俯视）
          roll: 0.0                              // 0° = 不横滚
        },
        // 相机飞行时间（秒）
        duration: 0.2
      });
    }, 100);
  }
}
// 执行加载函数
loadAnimatedCone();
```



#### 生成带模型动画的czml

```json
CZML JSON 字符串: 
[
  {
    "id": "document",
    "name": "Cone Articulation Animation",
    "version": "1.0",
    "clock": {
      "interval": "2023-01-01T00:00:00Z/2023-01-01T00:00:20Z",
      "currentTime": "2023-01-01T00:00:00Z",
      "multiplier": 1,
      "range": "LOOP_STOP",
      "step": "SYSTEM_CLOCK_MULTIPLIER"
    }
  },
  {
    "id": "coneModel",
    "name": "Animated Cone",
    "availability": "2023-01-01T00:00:00Z/2023-01-01T00:00:20Z",
    "position": {
      "cartographicDegrees": [
        116.39,
        39.9,
        1000
      ]
    },
    "model": {
      "gltf": "src/data/模型动画实验.gltf",
      "scale": 1000,
      "minimumPixelSize": 128,
      "articulations": {
      "cone yRotate": {
        "epoch": "2023-01-01T00:00:00Z",  // 基准时间（所有时间值相对此时间点）
            "number": [
              0,      // 相对基准时间的秒数（第 0 秒）
              0,      // 该时刻的关节旋转值（Y轴旋转 0 度）
              5,      // 相对基准时间的秒数（第 5 秒）
              180,    // 该时刻的关节旋转值（Y轴旋转 180 度）
              10,     // 相对基准时间的秒数（第 10 秒）
              360,    // 该时刻的关节旋转值（Y轴旋转 360 度，完成一圈）
              15,     // 相对基准时间的秒数（第 15 秒）
              540,    // 该时刻的关节旋转值（Y轴旋转 540 度）
              20,     // 相对基准时间的秒数（第 20 秒）
              720     // 该时刻的关节旋转值（Y轴旋转 720 度，完成两圈）
          ]
      	},
        "cone xRotate": {
          "epoch": "2023-01-01T00:00:00Z",
          "number": [
            0,
            0,
            5,
            0,
            10,
            45,
            15,
            0,
            20,
            -45
          ]
        },
        "cone xTranslate": {
          "epoch": "2023-01-01T00:00:00Z",
          "number": [
            0,
            0,
            5,
            50,
            10,
            0,
            15,
            -50,
            20,
            0
          ]
        },
        "cone zTranslate": {
          "epoch": "2023-01-01T00:00:00Z",
          "number": [
            0,
            0,
            10,
            20,
            20,
            0
          ]
        },
        "cone uniformScale": {
          "epoch": "2023-01-01T00:00:00Z",
          "number": [
            0,
            1,
            5,
            0.5,
            10,
            1,
            15,
            2,
            20,
            1
          ]
        }
      }
    }
  }
]
```



#### 总结：

这些步骤我已经完全理解了，我发现关键就是在gmdf文件中定义好各个关节可以使用的动作，然后通过STK GMDF Tools这个插件把这些动作注入到gltf文件中去，然后gltf查看器里面手动测试需要的动作，编写对应的动画帧，最后把这些动画帧放到czml的对应model的articulations属性下，加载czml数据并放到viewer中就可以实现模型动画了。







### 加载glb模型

```js
const modelMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(
  Cesium.Cartesian3.fromDegrees(116.39, 39.9, 500)
);

// 绕局部 Z 轴顺时针旋转 90°（ENU 中 Z = 北方向）
const rotY = Cesium.Matrix3.fromRotationZ(Cesium.Math.toRadians(-90));
const rotMatrix4 = Cesium.Matrix4.fromRotationTranslation(rotY);

// 右乘：将旋转施加在局部空间内
Cesium.Matrix4.multiply(modelMatrix, rotMatrix4, modelMatrix);

// 引入模型src\data\model\missileDF.glb
const model = await Cesium.Model.fromGltfAsync({
  url: "src/data/model/missileDF（选择+Y向上）.glb",
  modelMatrix: modelMatrix,
  scale: 0.125,
  minimumPixelSize: 64,
});
viewer.scene.primitives.add(model);
```





### CustomShader(模型外观)

使用流程

```
① 加载 GLTF 模型
    ↓
② 创建 CustomShader（定义 uniforms、varyings、着色器代码）
    ↓
③ 将 CustomShader 附加到 Model
    ↓
④ Cesium 自动将你的代码插入渲染管线
    ↓
⑤ 每帧渲染时执行：
   - vertexMain() → 修改顶点
   - fragmentMain() → 修改材质
    ↓
⑥ 在屏幕上看到自定义效果
```



#### 配置

```
https://cesium.com/learn/ion-sdk/ref-doc/CustomShader.html?classFilter=CustomShader

CustomShader 是 Cesium 提供的自定义着色器接口，允许：
-修改 GLTF 模型/3D Tiles 的渲染效果
-在顶点着色器中改变几何形状（变形、动画）
-在片段着色器中改变材质外观（颜色、纹理、光照）
```

| 参数                 | 类型                           | 默认值            | 作用                |
| -------------------- | ------------------------------ | ----------------- | ------------------- |
| `mode`               | `CustomShaderMode`             | `MODIFY_MATERIAL` | 着色器模式          |
| `lightingModel`      | `LightingModel`                | 模型默认          | 光照模型            |
| `translucencyMode`   | `CustomShaderTranslucencyMode` | `INHERIT`         | 透明度模式          |
| `uniforms`           | `Object`                       | -                 | 自定义 uniform 变量 |
| `varyings`           | `Object`                       | -                 | 自定义 varying 变量 |
| `vertexShaderText`   | `string`                       | -                 | 顶点着色器代码      |
| `fragmentShaderText` | `string`                       | -                 | 片段着色器代码      |



#### mode - 着色器模式

```javascript
作用： 决定你的代码插入到渲染管线的哪个位置

Cesium.CustomShaderMode.MODIFY_MATERIAL  // 默认：修改材质（最常用）
Cesium.CustomShaderMode.REPLACE_MATERIAL // 完全替换材质（高级用法）

// 模式 1：MODIFY_MATERIAL（默认）- 在原有材质基础上修改
const modifyShader = new Cesium.CustomShader({
  mode: Cesium.CustomShaderMode.MODIFY_MATERIAL, // 默认值
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      // 保留模型原有的纹理、颜色等
      // 只是叠加一层红色
      material.diffuse *= vec3(1.0, 0.5, 0.5); // 原有颜色 × 红色滤镜
    }
  `
});

// 模式 2：REPLACE_MATERIAL - 完全替换材质（忽略模型原有纹理）
const replaceShader = new Cesium.CustomShader({
  mode: Cesium.CustomShaderMode.REPLACE_MATERIAL,
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      // 完全自定义，不管模型原本是什么颜色/纹理
      material.diffuse = vec3(0.0, 1.0, 0.0); // 纯绿色
      material.alpha = 1.0;
    }
  `
});
```



#### lightingModel - 光照模型

```javascript
作用： 覆盖模型默认的光照计算方式。

Cesium.LightingModel.UNLIT  // 无光照
Cesium.LightingModel.PBR    // 基于物理的渲染


UNLIT不支持自发光
// 青色自发光
const shader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.PBR, 
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      material.emissive = vec3(0.0, 1.0, 1.0); // 青色自发光
    }
  `
});

// 有高光的光滑立方体
const shader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.PBR, // 像真实物体
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      material.diffuse = vec3(0.8, 0.2, 0.2);  // 基础颜色
      material.roughness = 0.3;                 // 粗糙度（0=镜面，1=粗糙）
      // 会根据太阳位置产生高光和阴影
    }
  `
});

光照模型会影响模型表面的漫反射颜色，这里本应该是偏红的颜色，但是material.diffuse 的颜色 × 光照衰减会导致颜色整体变灰

UNLIT 模式：
立方体 = material.diffuse 的的颜色
      = 你设的原始颜色
      ↓ 显示：原始颜色 ✓
      
PBR 模式：
立方体 = material.diffuse 的颜色 × 光照衰减
      = 原始颜色 × 0.3（如果光照弱）
      ↓ 显示：暗灰色 ✗
```

自发光

<img src="./assets/image-20260129161057989.png" alt="image-20260129161057989" style="zoom: 67%;" />

左侧反射高光

<img src="./assets/image-20260129161144616.png" alt="image-20260129161144616" style="zoom:67%;" />





#### translucencyMode - 透明度模式

```javascript
作用： 控制模型的透明度行为。
INHERIT（默认）- 继承模型材质的透明设置
OPAQUE - 强制不透明
TRANSLUCENT - 强制透明

// 模式 1：INHERIT - 遵循模型原本的透明设置
const inheritShader = new Cesium.CustomShader({
  translucencyMode: Cesium.CustomShaderTranslucencyMode.INHERIT,
  // 如果模型原本不透明，就不透明；原本透明就透明
});

// 模式 2：OPAQUE - 强制不透明（即使设置了 alpha < 1）
const opaqueShader = new Cesium.CustomShader({
  translucencyMode: Cesium.CustomShaderTranslucencyMode.OPAQUE,
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      material.diffuse = vec3(1.0, 0.0, 0.0);
      material.alpha = 0.3; // 设置了透明度但会被忽略
    }
  `
});

// 模式 3：TRANSLUCENT - 启用透明（性能开销更大）
const shader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.UNLIT,
  translucencyMode: Cesium.CustomShaderTranslucencyMode.TRANSLUCENT,
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      vec3 pos = fsInput.attributes.positionMC;
      
      // 根据高度设置透明度
      float alpha = (pos.y + 1.1) / 2.0; // 底部透明，顶部不透明
      
      material.diffuse = vec3(0.0, 0.5, 1.0);
      material.alpha = alpha; // 透明度生效
    }
  `
});
```

![image-20260129164202986](./assets/image-20260129164202986.png)





#### uniforms - 传参/设置（从 JS 传递到 GLSL）

```javascript
作用： 从 JavaScript 中传递参数到着色器

语法：
uniforms: {
  变量名: {
    type: 数据类型,
    value: 初始值
  }
}

类型					GLSL 类型		JavaScript 值
UniformType.FLOAT		float		1.5
UniformType.VEC2		vec2		new Cesium.Cartesian2(1, 2)
UniformType.VEC3		vec3		new Cesium.Cartesian3(1, 2, 3)
UniformType.VEC4		vec4		new Cesium.Cartesian4(1, 2, 3, 4)
UniformType.MAT3		mat3		new Cesium.Matrix3(...)
UniformType.MAT4		mat4		new Cesium.Matrix4(...)
UniformType.SAMPLER_2D	sampler2D	 new Cesium.TextureUniform({url: '...'})


// 立方体闪烁
const uniformShader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.UNLIT,
  
  // 定义 uniforms
  uniforms: {
    // 1. 浮点数
    u_time: {
      type: Cesium.UniformType.FLOAT,
      value: 0.0
    },
    
    // 2. 颜色（3D 向量）
    u_mainColor: {
      type: Cesium.UniformType.VEC3,
      value: new Cesium.Cartesian3(1.0, 0.5, 0.0) // 橙色
    },
    
    // 3. 开关（布尔值需要用 BOOL）
    u_enableGlow: {
      type: Cesium.UniformType.BOOL,
      value: true
    },
    
    // 4. 2D 向量
    u_offset: {
      type: Cesium.UniformType.VEC2,
      value: new Cesium.Cartesian2(0.5, 0.5)
    },
    
    // 5. 4x4 矩阵
    u_customTransform: {
      type: Cesium.UniformType.MAT4,
      value: Cesium.Matrix4.IDENTITY
    }
  },
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      // 在着色器中使用这些 uniform
      vec3 color = u_mainColor;
      
      // 根据时间闪烁
      float pulse = 0.5 + 0.5 * sin(u_time * 2.0);
      
      // 根据开关决定是否发光
      if (u_enableGlow) {
        color *= (1.0 + pulse);
      }
      
      material.diffuse = color;
      material.alpha = 1.0;
    }
  `
});

// 在 JavaScript 中动态更新 uniform 值
viewer.scene.preUpdate.addEventListener(() => {
  // 更新时间
  uniformShader.setUniform('u_time', performance.now() / 1000.0);
  
  // 更新颜色（假设根据用户输入）
  uniformShader.setUniform('u_mainColor', new Cesium.Cartesian3(
    Math.random(),
    Math.random(),
    Math.random()
  ));
  
  // 切换开关
  // uniformShader.setUniform('u_enableGlow', false);
});
```



#### varyings - 顶点→片段数据传递

```js
在顶点着色器中计算数据，自动插值后传递给片段着色器

原理：
顶点着色器（每个顶点运行一次）
    ↓ 计算 varying 变量
    ↓ 自动插值
片元着色器（每个像素运行一次）
    ↓ 使用插值后的 varying 变量
```

示例

```js
const varyingShader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.UNLIT,
  
  uniforms: {
    u_time: {
      type: Cesium.UniformType.FLOAT,
      value: 0.0
    }
  },
  
  // 定义 varyings
  varyings: {
    v_height: Cesium.VaryingType.FLOAT,           // 高度
    v_normalizedPos: Cesium.VaryingType.VEC3,     // 归一化位置
    v_distanceToCenter: Cesium.VaryingType.FLOAT, // 到中心距离
    v_customColor: Cesium.VaryingType.VEC3        // 自定义颜色
  },
  
  vertexShaderText: `
    void vertexMain(VertexInput vsInput, inout czm_modelVertexOutput vsOutput) {
      vec3 pos = vsInput.attributes.positionMC;
      
      // 计算各种数据
      v_height = (pos.y + 1.0) / 2.0; // 归一化高度 [0,1]
      
      v_normalizedPos = (pos + 1.0) / 2.0; // 位置归一化到 [0,1]
      
      v_distanceToCenter = length(pos); // 到原点的距离
      
      // 根据位置计算颜色（顶点颜色）
      v_customColor = vec3(
        abs(pos.x),  // R 通道
        abs(pos.y),  // G 通道
        abs(pos.z)   // B 通道
      );
      
      // 可以修改顶点位置
      float wave = sin(pos.x * 3.14 + u_time) * 0.1;
      vsOutput.positionMC = pos + vsInput.attributes.normalMC * wave;
    }
  `,
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      // 使用插值后的 varying 值
      
      // 示例 1：根据高度混合颜色
      vec3 bottomColor = vec3(0.0, 0.0, 1.0); // 蓝色
      vec3 topColor = vec3(1.0, 1.0, 0.0);    // 黄色
      vec3 gradientColor = mix(bottomColor, topColor, v_height);
      
      // 示例 2：根据到中心的距离添加光晕
      float glow = 1.0 - smoothstep(0.0, 1.5, v_distanceToCenter);
      
      // 示例 3：使用自定义颜色
      vec3 finalColor = gradientColor * (1.0 + glow) + v_customColor * 0.3;
      
      material.diffuse = finalColor;
      material.alpha = 1.0;
    }
  `
});
```

<img src="./assets/image-20260129172422014.png" alt="image-20260129172422014" style="zoom: 80%;" />



#### vertexMain 与 fragmentMain

```glsl
vertexMain() 函数

void vertexMain(
    VertexInput vsInput,              // 输入：顶点数据
    inout czm_modelVertexOutput vsOutput  // 输出：修改后的顶点
)

VertexInput vsInput - 输入参数（只读），包含：
-vsInput.attributes.positionMC - 顶点在模型坐标系的位置（vec3），是每个像素所对应的三维坐标 (x, y, z)，positionMC.y 指的是模型坐标系中单个顶点的Y轴坐标值
-vsInput.attributes.normalMC - 顶点法线在模型坐标系（vec3）
-vsInput.attributes.tangentMC - 切线（vec3）
-vsInput.attributes.texCoord_0 - UV 纹理坐标（vec2）
-vsInput.attributes.texCoord_1 - 第二套 UV（如果有）
-vsInput.attributes.color_0 - 顶点颜色（vec4）
-vsInput.featureIds - 特征 ID（用于高级功能）

czm_modelVertexOutput vsOutput - 输出参数（可修改），包含：
-vsOutput.positionMC - 可以修改顶点位置（vec3）
-vsOutput.normalMC - 修改法线方向
-vsOutput.tangentMC - 修改切线
-vsOutput.pointSize - 点的大小（点渲染时）



fragmentMain 函数
void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material)
    
FragmentInput fsInput - 输入参数，包含：
-fsInput.attributes - 从顶点着色器插值过来的属性（法线、UV等），只不过顶点着色器中获取的是原始值罢了
-fsInput.featureIds - 特征 ID
    
    
czm_modelMaterial material - 材质输出（可修改），包含：
-material.diffuse - 漫反射颜色（vec3） 
    指的就是模型表面的基础颜色
    
-material.specular - 高光颜色（vec3）
    高光效果的颜色，通常设置为白色或接近白色（vec3(1.0, 1.0, 1.0)），一般用在光滑的金属表面
    
-material.emissive - 自发光颜色（vec3）
    模型自身发出的光线颜色，LED屏幕、霓虹灯、发光的物体等
    
-material.alpha - 透明度（float, 0-1）
-material.roughness - 粗糙度（float）（0=镜面，1=粗糙）
    
-material.normalEC - 法线（眼睛坐标系）
    法线：从表面垂直向外指出的箭头
   ↑ 法线向量
   |
---●--- 表面
   |
   ↓ 反向
    法线告诉着色器"这个表面朝向哪个方向"
    眼睛坐标系是以相机为原点建立的坐标系
    法线直接影响漫反射的明暗，高光的位置和强度，根据物理显示，法线指向光源，夹角小，表面亮度就高
    
    

MC (Model Coordinates) - 模型坐标系，当前立方体是 [-1,1]，实体所占据的坐标范围
WC (World Coordinates) - 世界坐标系
EC (Eye Coordinates) - 眼睛/相机坐标系
```





##### 底部红色渐变到顶部绿色

```js
const shader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.UNLIT, // 不使用光照
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      // 获取模型坐标系的位置（通过插值获得）
      vec3 positionMC = fsInput.attributes.positionMC;
      
      // Y 轴渐变：底部（-1）是红色，顶部（+1）是绿色
      float t = (positionMC.y + 1.0) / 2.0; // 归一化到 [0,1]
      
      material.diffuse = mix(
        vec3(1.0, 0.0, 0.0), // 红色
        vec3(0.0, 1.0, 0.0), // 绿色
        t
      );
      material.alpha = 1.0;
    }
  `
})
```

<img src="./assets/image-20260129145043533.png" alt="image-20260129145043533" style="zoom:50%;" />





##### "呼吸"一样放大缩小

```js
const shader = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.UNLIT,
  
  uniforms: {
    u_time: {
      type: Cesium.UniformType.FLOAT,
      value: 0.0
    }
  },
  
  vertexShaderText: `
    void vertexMain(VertexInput vsInput, inout czm_modelVertexOutput vsOutput) {
      vec3 pos = vsInput.attributes.positionMC;
      
      // 沿法线方向膨胀/收缩（呼吸效果），基于u_time的变化
      float scale = 1.0 + 0.3 * sin(u_time);
      
      // 修改顶点位置！这是 material 做不到的
      vsOutput.positionMC = pos * scale;
    }
  `,
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      material.diffuse = vec3(0.2, 0.6, 1.0); // 蓝色
      material.alpha = 1.0;
    }
  `
});
```

![image-20260129150828390](./assets/image-20260129150828390.png)













##### 位置和明暗伸缩起伏

```js
const shader4 = new Cesium.CustomShader({
  lightingModel: Cesium.LightingModel.UNLIT,
  
  uniforms: {
    u_time: {
      type: Cesium.UniformType.FLOAT,
      value: 0.0
    },
    u_twistAmount: {
      type: Cesium.UniformType.FLOAT,
      value: 2.0 // 扭曲强度
    }
  },
  
  vertexShaderText: `
    void vertexMain(VertexInput vsInput, inout czm_modelVertexOutput vsOutput) {
      vec3 pos = vsInput.attributes.positionMC;
      
      // 根据高度计算旋转角度（Y轴越高，旋转越多）
      float angle = pos.y * u_twistAmount + u_time;
      
      // 2D 旋转矩阵（绕Y轴）
      float cosA = cos(angle);
      float sinA = sin(angle);
      
      // 应用旋转到 X-Z 平面
      vec3 twisted;
      twisted.x = pos.x * cosA - pos.z * sinA;
      twisted.y = pos.y;
      twisted.z = pos.x * sinA + pos.z * cosA;
      
      vsOutput.positionMC = twisted;
    }
  `,
  
  fragmentShaderText: `
    void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material) {
      vec3 pos = fsInput.attributes.positionMC;
      
      // 彩虹渐变
      float t = (pos.y + 1.0) / 2.0;
      vec3 color1 = vec3(1.0, 0.0, 0.5); // 粉红
      vec3 color2 = vec3(0.0, 0.8, 1.0); // 青色
      
      material.diffuse = mix(color1, color2, t);
      material.alpha = 1.0;
    }
  `
});
```

![image-20260129154110326](./assets/image-20260129154110326.png)<img src="./assets/image-20260129154129018.png" alt="image-20260129154129018" style="zoom:80%;" />







### 第三方底图



天地图

```js
const viewer = new Cesium.Viewer('cesiumContainer', {
  imageryProvider: false, // 设置为 false，不加载任何底图
  ...
});  


//配置并创建天地图Web瓦片服务影像提供者
  const tianditu = new Cesium.WebMapTileServiceImageryProvider({
    url: 'http://{s}.tianditu.gov.cn/img_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=img&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles&tk='+'4a050b56e21976c2bb53fec757797dc1',
    layer: 'img_w',
    style: 'default',
    format: 'tiles',
    tileMatrixSetID: 'w',
    subdomains:['t0','t1','t2','t3','t4','t5','t6','t7'],
    maximumLevel: 18,
    credit: new Cesium.Credit('天地图'),
  })  

  // 将天地图层添加到观众实例的影像图层集合中
  viewer.imageryLayers.addImageryProvider(tianditu)
```

<img src="./assets/image-20260202095429906.png" alt="image-20260202095429906" style="zoom:50%;" />





mapbox底图

```ts
const viewer: Cesium.Viewer = new Cesium.Viewer("app", {
  baseLayer: new Cesium.ImageryLayer(
    new Cesium.MapboxStyleImageryProvider({
      styleId: "satellite-streets-v12", // 或 "streets-v12", "dark-v11" 等
      accessToken: "你的Mapbox Access Token",
    })
  ),
  timeline: false,
  // ...其余选项
});
```









































### 紧凑速查

| 名称                               | 作用                                     | 典型构造                                                     |
| ---------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| GeoJsonDataSource.load             | 异步加载 GeoJSON/TopoJSON 并生成实体集合 | `Cesium.GeoJsonDataSource.load(url,{stroke,fill,strokeWidth})` |
| dataSource.entities.values         | 遍历加载后所有实体做属性驱动处理         | `dataSource.entities.values.forEach(e=>{ e.polygon.extrudedHeight = ... })` |
| Entity.properties / getProperty    | 访问数据源要素属性（动态或静态）         | `entity.properties.<Field>._value` / `feature.getProperty(name)` |
|                                    |                                          |                                                              |
| CzmlDataSource.load                | 加载 CZML（带时间/路径/动画）            | `Cesium.CzmlDataSource.load(czmlUrl)`                        |
| KmlDataSource.load                 | 加载 KML/KMZ（点/线/面/样式）            | `Cesium.KmlDataSource.load(kmzUrl)`                          |
| viewer.dataSources.add             | 注册数据源到场景                         | `viewer.dataSources.add(ds)`                                 |
| viewer.zoomTo                      | 相机自动飞行至数据源范围                 | `viewer.zoomTo(ds)`                                          |
|                                    |                                          |                                                              |
| Cesium3DTileset.fromUrl            | 加载 3D Tiles 集模型                     | `Cesium.Cesium3DTileset.fromUrl(url, options)`               |
| IonResource.fromAssetId            | Ion 资产资源解析                         | `Cesium.IonResource.fromAssetId(id)`                         |
| Cesium3DTileStyle                  | 以样式语言控制 3D Tiles 外观             | `tileset.style = new Cesium.Cesium3DTileStyle({color:{conditions},defines,show})` |
|                                    |                                          |                                                              |
| 条件样式 conditions                | 按属性阈值切换不同颜色表达式             | `conditions: [[ "${Height} >= 100", "rgb(...)" ], ["true","rgb(...)" ]]` |
| scene.pick                         | 屏幕拾取顶层图元/特征                    | `const f = viewer.scene.pick(pos)`                           |
| Cesium3DTileFeature.getPropertyIds | 获取被拾取特征全部属性名                 | `const ids = feature.getPropertyIds()`                       |
| ScreenSpaceEventHandler            | 绑定点击拾取与交互                       | `new Cesium.ScreenSpaceEventHandler(viewer.canvas)`          |
|                                    |                                          |                                                              |
| ClippingPlaneCollection            | 定义/组合裁剪平面集合                    | `new Cesium.ClippingPlaneCollection({planes:[...], enabled:true})` |
| ClippingPlane                      | 单个裁剪平面（法线+距离）                | `new Cesium.ClippingPlane(new Cesium.Cartesian3(0,0,-1), distance)` |
| tileset 根 transform               | 获取模型局部原点世界位置                 | `Matrix4.getTranslation(tileset.root.transform, new Cartesian3())` |
| Cartographic.fromCartesian         | 计算经纬高差辅助对齐裁剪面               | `Cartographic.fromCartesian(point)`                          |
| Matrix4.fromTranslation            | 构造平移矩阵应用到裁剪集合               | `ClippingPlaneCollection.modelMatrix = Matrix4.fromTranslation(offset)` |
| PlaneGraphics + CallbackProperty   | 展示裁剪面辅助可视化平面                 | `plane: new CallbackProperty(()=>new Plane(normal,distance), false)` |
|                                    |                                          |                                                              |
| tileset.splitDirection             | 设置模型卷帘左右显示侧                   | `tileset.splitDirection = Cesium.SplitDirection.LEFT`        |
| scene.splitPosition                | 卷帘分割位置（0~1）                      | `viewer.scene.splitPosition = 0.5`                           |
| getBoundingClientRect              | 获取 Canvas 尺寸用于滑块归一             | `const rect = viewer.canvas.getBoundingClientRect()`         |
| relativeX (0~1)                    | 归一化滑块位置同步卷帘                   | `viewer.scene.splitPosition = e.endPosition.x / rect.width`  |
| Cesium.Color.fromRandom            | 随机颜色赋值                             | `entity.polygon.material = Cesium.Color.fromRandom({alpha:0.5})` |





### 工作流步骤



加载并处理 GeoJSON（含拉伸）

1. 调用 `GeoJsonDataSource.load(url, {stroke, fill, strokeWidth})`。
2. `viewer.dataSources.add(ds)` → `viewer.zoomTo(ds)`。
3. 遍历 `ds.entities.values` 读取属性（如人口）→ 设置 `polygon.extrudedHeight` + 随机材质。

加载 CZML（动态场景）

1. `CzmlDataSource.load(czmlUrl)` 获得数据源。
2. 添加到 viewer 并 zoomTo。
3. 时间动态内容由 CZML 内部驱动（无需手工更新）。

加载 KML/KMZ

1. `KmlDataSource.load(kmzUrl)`。
2. 添加并保存引用以便后续样式或选择。

加载 3D Tiles（基本 + Ion）

1. 直接 URL：`Cesium3DTileset.fromUrl(url, options)` → 添加 → zoomTo。
2. Ion 资产：`IonResource.fromAssetId(assetId)` → `fromUrl(resource)`。
3. 可选：调试/性能参数（LOD、内存等）。

应用 3D Tiles 样式语言

1. 确认 tileset 已加载。
2. 构造 `conditions` 阶梯数组与 `defines` 表达式。
3. 设置 `tileset.style = new Cesium3DTileStyle({...})`。

拾取 3D Tiles 特征并显示属性

1. 建立 `ScreenSpaceEventHandler` 左键监听。
2. `scene.pick(position)` 获取 `Cesium3DTileFeature`。
3. `feature.getPropertyIds()` → 映射出键值 → 展示（表格/面板）。
4. 改写 `feature.color` 实现点击高亮，恢复前一选中项颜色。

裁剪模型（ClippingPlane）

1. 创建 `ClippingPlaneCollection`（含一个或多个 `ClippingPlane`）。
2. 加载 tileset 时传 `clippingPlanes` 或后赋值。
3. 取 `root.transform` 与 `boundingSphere.center` → 转 Cartographic → 计算高度差。
4. `modelMatrix = Matrix4.fromTranslation(offsetZ)` 使裁剪面与模型对齐。
5. 可视化辅助：新增 PlaneGraphics + CallbackProperty 同步距离调节。

卷帘（Split）比较效果

1. 加载一个或多个 tileset（目标设定 `splitDirection`）。
2. 初始化 `scene.splitPosition`。
3. 构建滑块 DOM，监听拖拽（屏幕移动 → 计算相对 x 比例）。
4. 更新 `splitPosition` 与滑块 CSS 定位，实现实时分割。







### 最小可执行


加载 GeoJSON 州界并按人口属性动态拉伸与随机着色。

```ts
// geojson-min.ts 交互逻辑：加载并按属性拉伸与着色
Cesium.GeoJsonDataSource.load('/data/us_states.topojson', {
  stroke: Cesium.Color.BLACK,
  fill: Cesium.Color.WHITE.withAlpha(0.4),
  strokeWidth: 2
}).then(ds => {
  viewer.dataSources.add(ds);
  viewer.zoomTo(ds);
  ds.entities.values.forEach(e => {
    const pop = e.properties?.Population?._value;
    if (e.polygon && pop) {
      e.polygon.extrudedHeight = pop / 100.0;
      e.polygon.material = Cesium.Color.fromRandom({ alpha: 0.5 });
    }
  });
});
```





加载 CZML 描述的带时间驱动的实体轨迹并自动聚焦视图。

```ts
// czml-min.ts 加载 CZML 动态车辆/轨迹
Cesium.CzmlDataSource.load('/data/Vehicle.czml').then(ds => {
  viewer.dataSources.add(ds);
  viewer.zoomTo(ds);
});
```



加载 KMZ 卫星数据并显示其中的点线面样式。

```ts
// kml-min.ts 加载 KML/KMZ 要素
Cesium.KmlDataSource.load('/data/Satellites.kmz').then(ds => {
  viewer.dataSources.add(ds);
  viewer.zoomTo(ds);
});
```



加载本地 3D Tiles 模型并飞行至其范围。

```ts
// tileset-min.ts 基础 3D Tiles 加载
Cesium.Cesium3DTileset.fromUrl('/tiles/model/tileset.json').then(ts => {
  viewer.scene.primitives.add(ts);
  viewer.zoomTo(ts);
});
```



通过 Ion 资产加载 3D Tiles 并按高度属性分档着色与筛选显示。

```ts
// tileset-ion-style-min.ts Ion + 样式语言
async function loadStyled(assetId:number){
  const res = await Cesium.IonResource.fromAssetId(assetId);
  const ts = await Cesium.Cesium3DTileset.fromUrl(res);
  viewer.scene.primitives.add(ts);
  viewer.zoomTo(ts);
  ts.style = new Cesium.Cesium3DTileStyle({
    defines: { H:"${Height}" },
    color: {
      conditions: [
        ["${Height} >= 100", "rgb(255,180,180)"],
        ["${Height} >= 50", "rgb(200,120,90)"],
        ["true", "rgb(120,80,60)"]
      ]
    },
    show: "${H} > 10"
  });
}
loadStyled(75343);
```





点击 3D Tiles 建筑拾取并打印全部属性，同时高亮当前选中对象。

```ts
// tileset-pick-min.ts 拾取属性与高亮
let lastFeature:any, lastColor:Cesium.Color;
const handler = new Cesium.ScreenSpaceEventHandler(viewer.canvas);
handler.setInputAction(e=>{
  const feature = viewer.scene.pick(e.position);
  if (!feature || !feature.getPropertyIds) return;
  if (lastFeature) lastFeature.color = lastColor;
  lastFeature = feature;
  lastColor = feature.color;
  feature.color = Cesium.Color.YELLOW;
  const ids = feature.getPropertyIds();
  console.log(ids.map(k=>({name:k,value:feature.getProperty(k)})));
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);
```





对 3D Tiles 模型应用一个对齐其高度差的裁剪平面并用透明辅助面展示实时位置。

```ts
// clipping-min.ts 裁剪平面对齐与可视化
const clippingPlanes = new Cesium.ClippingPlaneCollection({
  planes:[ new Cesium.ClippingPlane(new Cesium.Cartesian3(0,0,-1), 30) ],
  enabled:true
});
Cesium.Cesium3DTileset.fromUrl('/tiles/AGI_HQ/tileset.json',{ clippingPlanes }).then(ts=>{
  viewer.scene.primitives.add(ts);
  viewer.zoomTo(ts);
    
    
    
// modelMatrix的作用，如果有一个以模型中心为原点的顶点坐标，把它乘以modelMatrix后得到的是在世界坐标系里的坐标，4*4矩阵，包含旋转、缩放、平移等变换
// root.transform的作用，3D Tiles 的几何（顶点）通常是在一个“局部小坐标”里定义的，也就是模型坐标系（通常以模型中心为原点），假如这个模型处在世界坐标系中的某个位置，如果需要把顶点的坐标值 从模型坐标系中 换算到 世界坐标系中，就需要将在模型坐标系中定义的顶点坐标值乘上root.transform
// 所以当模型中心点为（0，0，0）时，如果获取这个点在世界坐标中的位置呢，可以直接使用 root.transform 的平移部分，相当于用root.transform乘以（0，0，0，1）
// 对于3D Tiles模型，几何作者定义的“模型局部原点”可以放在任意位置（例如建筑地面角点、楼中心、或根节点 pivot）。所以正常情况下，模型原点和包围球中心不会重合


// 需要计算模型中心位置和包围盒中心位置的高度差
// 模型原点的世界坐标，Cesium.Matrix4.getTranslation(matrix, result) → Cartesian3，该方法从4x4矩阵中提取平移部分，返回一个新的 Cartesian3，这就相当于知道模型相对于世界坐标原点（0,0,0）的位移，也就是模型原点在世界坐标系中的位置
  const tCenter = Cesium.Matrix4.getTranslation(ts.root.transform, new Cesium.Cartesian3());
  const bCenter = ts.boundingSphere.center;
  const tCarto = Cesium.Cartographic.fromCartesian(tCenter);
  const bCarto = Cesium.Cartographic.fromCartesian(bCenter);
  const diff = bCarto.height - tCarto.height;
    
    
// 根据高度差，设置平移矩阵到裁剪面集合的模型矩阵上
// Cesium.Matrix4.fromTranslation(translation, result) → Matrix4;从表示平移的 Cartesian3 创建一个4x4矩阵
  clippingPlanes.modelMatrix = Cesium.Matrix4.fromTranslation(new Cesium.Cartesian3(0,0,diff));
    
    
  viewer.entities.add({
    position: ts.boundingSphere.center,
    plane: {
      dimensions: new Cesium.Cartesian2(ts.boundingSphere.radius*2, ts.boundingSphere.radius*2),
      material: Cesium.Color.WHITE.withAlpha(0.4),
      plane: new Cesium.CallbackProperty(()=>new Cesium.Plane(new Cesium.Cartesian3(0,0,-1), clippingPlanes.planes[0].distance), false)
    }
  });
});
```





加载左右分组的两个 Tiles 模型并通过可拖动滑块实时调整卷帘分界位置做视觉对比。

```ts
// split-min.ts 卷帘对比（左右分割）
Cesium.Cesium3DTileset.fromUrl('/tiles/A/tileset.json').then(a=>{
  a.splitDirection = Cesium.SplitDirection.LEFT;
  viewer.scene.primitives.add(a);
});
Cesium.Cesium3DTileset.fromUrl('/tiles/B/tileset.json').then(b=>{
  b.splitDirection = Cesium.SplitDirection.RIGHT;
  viewer.scene.primitives.add(b);
});
viewer.scene.splitPosition = 0.5;

const handler = new Cesium.ScreenSpaceEventHandler(viewer.canvas);
let dragging = false;
const slider = document.getElementById('slider');
slider?.addEventListener('mousedown',()=>dragging=true);
slider?.addEventListener('mouseup',()=>dragging=false);

handler.setInputAction(e=>{
  if(!dragging) return;
  const rect = viewer.canvas.getBoundingClientRect();
  const ratio = e.endPosition.x / rect.width;
  viewer.scene.splitPosition = Math.min(Math.max(ratio,0),1);
  if(slider) slider.style.left = (rect.left + ratio*rect.width - slider.offsetWidth/2)+"px";
}, Cesium.ScreenSpaceEventType.MOUSE_MOVE);
```







### 核心概念

```
数据源抽象统一：GeoJson/CZML/KML 均通过 DataSource 接口（viewer.dataSources.add）管理实体集合统一缩放与遍历。

GeoJSON 属性驱动拉伸：使用实体属性值映射到 polygon.extrudedHeight 与动态材质实现数据可视化。

CZML 时间语义：内含时间区间与位置序列，加载后自动驱动动画，无需手动更新坐标。

KML/KMZ 解析：保留原样式、层次与标注信息，适合迁移 Google Earth 数据。

3D Tiles 分层 LOD：根据屏幕空间误差动态加载细节，可通过参数优化性能与首屏体验。

3D Tiles 样式语言：defines 提供可复用表达式，conditions 按优先匹配顺序从上到下，show 可用于属性过滤。

特征拾取：scene.pick 返回 Cesium3DTileFeature 时可直接访问与修改 feature.color，不需要重新生成几何或实体。

裁剪平面对齐：需计算模型 transform 原点与包围球中心的高程差，用平移矩阵修正裁剪平面与视觉辅助平面对齐。

高度差计算应使用 Cartographic 的 height 而非 Cartesian3 的 z 分量避免地球椭球引起的误差。

PlaneGraphics 与 CallbackProperty：用于把裁剪平面参数变化实时可视化（同步距离调节）。

卷帘机制：splitDirection 标识模型属于左侧还是右侧渲染区；scene.splitPosition 控制分界归一位置。

颜色高亮恢复策略：记录上一次选中特征及其原始颜色，新的选中前恢复旧对象状态。
```





### 决策树

```
你要实现的场景是什么？
│
├─ A. 加载二维/属性面数据并做可视化？
│    ├─ 格式是 GeoJSON/TopoJSON → GeoJsonDataSource.load
│    ├─ 需要属性驱动高度或颜色 → 遍历 entities.values 设置 extrudedHeight/material
│
├─ B. 有时间序列/轨迹动画？
│    └─ 使用 CZML → CzmlDataSource.load → 自动时间驱动
│
├─ C. 迁移现有 Earth 标注或路径？
│    └─ 使用 KML/KMZ → KmlDataSource.load → 保留原样式
│
├─ D. 加载大规模三维模型/城市？
│    ├─ 本地或网络 URL → Cesium3DTileset.fromUrl
│    └─ Ion 资产 → IonResource.fromAssetId → fromUrl(resource)
│
├─ E. 需要按属性分档着色或过滤 3D Tiles？
│    └─ 构造 conditions + defines → new Cesium3DTileStyle 设置 style
│
├─ F. 需要点击获取建筑/特征属性并高亮？
│    ├─ ScreenSpaceEventHandler LEFT_CLICK → scene.pick
│    ├─ 是 Cesium3DTileFeature? → getPropertyIds / getProperty
│    └─ 颜色高亮 → 记录并恢复 lastFeature.color
│
├─ G. 想裁剪模型内部观察或剖切？
│    ├─ 建立 ClippingPlaneCollection (planes + enabled)
│    ├─ 对齐高度 → transform & boundingSphere → Cartographic→ heightDiff
│    └─ 应用平移矩阵到 clippingPlanes.modelMatrix
│
├─ H. 需要展示裁剪面位置？
│    └─ PlaneGraphics + CallbackProperty 动态跟随 distance
│
├─ I. 要对比两个模型或数据版本？
│    ├─ 分别设置 splitDirection (LEFT / RIGHT)
│    └─ 交互滑块 → 更新 scene.splitPosition (0~1)
│
├─ J. 来自属性驱动高度还是直接样式？
│    ├─ 需要拉伸几何 → polygon.extrudedHeight
│    └─ 仅颜色表达 → Cesium3DTileStyle conditions/defines
│
└─ K. 若性能下降怎么办？
     ├─ 减少频繁 pick 次数（只在点击，不在移动）
     ├─ 精简 conditions 阶梯数量
     ├─ 控制裁剪平面数量（过多会增加剔除计算）
     └─ 避免在回调中做重型循环或随机材质频繁改变
```





## 时间系统（clock）

`Clock` 代表 Cesium 的“模拟时间系统”

clock是viewer核心结构的一部分

```
const viewer = {
  // ...
  clock: Clock { currentTime, startTime, stopTime, multiplier, shouldAnimate, ... },
  // ...
}
```



### 时间属性

clock字面量

```ts
viewer.clock = {
  startTime:   JulianDate,  // 时间轴起点
  stopTime:    JulianDate,  // 时间轴终点
  currentTime: JulianDate,  // 当前时间
  clockRange:  LOOP_STOP,   // 到 stopTime 时循环回 startTime
  multiplier:  number,      // 时间流逝速度倍数
  shouldAnimate: boolean,   // 是否自动推进 currentTime
  onTick: Event,            // 每一帧触发的事件
  ...
}
```

具体解释

```js
clock.startTime
// Cesium 的时间轴（timeline）和动画控件（animation）会以它作为“最早时间”。


clock.stopTime
// 时间轴的终点。
与 startTime 一起定义一个播放区间 [startTime, stopTime]。


clock.currentTime
// 当前时刻（正在渲染/计算的时间点）。
如果你把它设为 startTime，相当于“从头播放”。


clock.clockRange
当 currentTime 越过 stopTime（或小于 startTime）时如何处理。
  // Cesium.ClockRange.UNBOUNDED：不限制，时间会一直往前/往后跑（默认常见）。
  // Cesium.ClockRange.CLAMPED：夹住，超过边界就停在边界（不会循环）。
  // esium.ClockRange.LOOP_STOP：到 stopTime 后回到 startTime 继续循环。


clock.multiplier
时间流逝倍率（每一帧让 currentTime 增加多少“模拟时间”）。
  multiplier = 1：真实时间速度（大致 1 秒走 1 秒）
  multiplier = 10：快进 10 倍
  multiplier = 0.1：慢放


clock.shouldAnimate
是否自动推进时间。
  true：每帧根据 multiplier 自动更新 currentTime（播放中）
  false：不自动走时间（暂停），除非手动改 currentTime
```





示例代码

```ts
const clock = viewer.clock;

// 定义起止时间（使用 JulianDate）
const start = Cesium.JulianDate.now();
const stop = Cesium.JulianDate.addSeconds(start, 10, new Cesium.JulianDate());
console.log('start=', start); // {dayNumber: 2461051, secondsOfDay: 76845.744} dayNumber 是儒略日天数，secondsOfDay 是当天的秒数
console.log('stop=', stop);

// 1) 起点/终点：控制 timeline 的时间范围
clock.startTime = start.clone();
clock.stopTime = stop.clone();

// 2) 当前时间：从起点开始播
clock.currentTime = start.clone();

// 3) 越界行为：到 stopTime 后回到 startTime 循环
clock.clockRange = Cesium.ClockRange.LOOP_STOP;

// 4) 时间倍率：以真实时间的 5 倍速播放
clock.multiplier = 5;

// 5) 是否自动播放：true = 播放；
clock.shouldAnimate = true;

// 让 timeline 的“显示窗口”立刻对齐到 start/stop
viewer.timeline?.zoomTo(clock.startTime, clock.stopTime);
```



当前时间是2026-01-11，17：51，北京时间比UTC早8个小时，所以时间轴上显示9：51，这里显示5倍速，时间范围是10s，控制器左边的绿色箭头亮起表示在推进

![image-20260111175142422](./assets/image-20260111175142422.png)











### 儒略日（JulianDate）

```ts
Cesium.JulianDate 代表儒略日期

Cesium.JulianDate.now() 获取当前时间的 JulianDate 表示

Cesium.JulianDate.addSeconds(julianDate, seconds, result) 在 julianDate 基础上加上 seconds 秒，返回新的 JulianDate（可选传入 result 复用对象）


Cesium.JulianDate.greaterThan(left, right) → boolean
left: Cesium.JulianDate
right: Cesium.JulianDate
比较两个儒略日期，如果 left 大于 right 则返回 true, 否则返回 false.


Cesium.JulianDate.secondsDifference(left, right) → number
left: Cesium.JulianDate
right: Cesium.JulianDate
计算两个儒略日期之间的秒数差异，返回 left 减去 right 的秒数差值（可以为负值）.
```

#### fromIso8601

```js
Cesium.JulianDate.fromIso8601(iso8601String, result) → JulianDate

将 ISO 8601 格式的日期字符串 → 转换为儒略日期对象
参数：
- iso8601String : String
    要转换的 ISO 8601 日期字符串。
- result : JulianDate (可选)
    可选的结果对象，用于存储转换后的儒略日期。如果提供了该参数，函数将使用该对象来存储结果，而不是创建一个新的对象。
返回值：
- JulianDate
```

使用·

```js
// 基础用法：只传字符串
const julianDate = Cesium.JulianDate.fromIso8601('2023-01-01T00:00:00Z');
console.log(julianDate);
// 输出:  JulianDate { dayNumber: 2459945, secondsOfDay: 0 }

// 可以转换为标准日期查看
const date = Cesium.JulianDate. toDate(julianDate);
console.log(date);
// 输出: 2023-01-01T00:00:00.000Z
```



此方法优于 Date.parse 方法，因为它能够处理 ISO 8601 规范定义的所有有效格式，包括闰秒和亚毫秒级的时间，而大多数 JavaScript 实现会忽略这些内容。



ISO 8601 格式字符串

ISO 8601 是国际标准化组织制定的日期和时间表示标准，旨在消除因地区习惯和格式差异带来的歧义

具体例子：

```javascript
parseTime("2023-01-01");                    // ✅
parseTime("2023-01-01T00:00:00Z");          // ✅
parseTime("2023-01-01T00:00:00+08:00");     // ✅
parseTime("2023-01-01 00:00:00");           // ✅
parseTime("01/01/2023");                    // ✅（部分浏览器）
parseTime("2023-01-01T08:30:45");           // ✅

// 返回 Unix 时间戳（秒）
// 例如：1672531200
```



```
2023-01-01T00:00:00Z
│      │  │ │  │  │ │
│      │  │ │  │  │ └─ Z：表示 UTC 时区（零时区）
│      │  │ │  │  └─── 秒
│      │  │ │  └────── 分
│      │  │ └───────── 时
│      │  └─────────── T：日期和时间的分隔符
│      └───────────── 月份
└──────────────────── 年份（4位数字）
```





#### fromDate ：js→儒略

```js
// Cesium.JulianDate.fromDate(date, result) → JulianDate 静态方法，根据JavaScript中的时间对象创建一个新的 JulianDate 实例


const sec = 1705276800;           // Unix 时间戳（秒）

// 秒 → 毫秒 → Date → JulianDate
const ms = sec * 1000;            // 毫秒
const dt = new Date(ms);          // JavaScript Date
const jd = Cesium.JulianDate.fromDate(dt);  // Cesium JulianDate

console.log(sec);   // 1705276800
console.log(ms);    // 1705276800000
console.log(dt);    // 2024-01-15T00:00:00.000Z
console.log(jd);    // JulianDate {...}
```





### tick事件

```ts
tick 在 Cesium 里就是 “时钟走了一步（一次更新）” 的意思：调用 Clock.tick() 会根据当前 Clock 的配置（clockStep / multiplier / shouldAnimate / startTime / stopTime / clockRange 等）计算并更新 clock.currentTime，然后把“这一 tick 的结果”广播出去。
Cesium 内部每帧会做类似这样的事情（概念伪代码）：
function renderFrame() {
  const newTime = viewer.clock.tick();   // 1) 走一步时钟（tick）
  viewer.scene.render(newTime);          // 2) 用这个时间渲染这一帧
}


Viewer 的默认渲染循环会在每一帧自动调用 viewer.clock.tick()，所以业务代码通常不需要手动调用 tick()，只需要设置 clock 参数、或监听 onTick

```

- `button.click`：用户点击按钮触发事件

- clock.onTick：每次 clock.tick 执行完成后触发事件，tick是方法（行为），onTick是事件（通知）

- **`tick()`：每帧都会被调用（在 Viewer 默认渲染循环里）**

- shouldAnimate：决定

  currentTime 是否会前进

  - `true`：按 clockStep + multiplier 推进时间
  - `false`：通常 **currentTime 不推进**（或推进非常受限），但 **onTick 仍可能每帧触发**（因为 tick 仍在被调用）



手动tick验证

```ts
viewer.useDefaultRenderLoop = false; // 关闭默认的渲染循环

let ticks = 0;
viewer.clock.onTick.addEventListener(() => {
  ticks++;
});

setInterval(() => {
  console.log("onTick 次数/秒 =", ticks);
  ticks = 0;
}, 1000);

function frame() {
  viewer.clock.tick();     // 手动触发 tick
  viewer.render();         // 渲染这一帧（否则画面不刷新）
  requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
```







### 时钟步进模式（clockStep)

一个每帧更新的代码

```js
viewer.clock.clockStep = Cesium.ClockStep.TICK_DEPENDENT; // 设置时间推进模式为依赖于帧，每帧调用一次 tick
```



```
clock.clockStep：时间推进模式
确定对 Clock#tick 的调用是依赖于帧还是依赖于系统时钟。

Cesium.ClockStep.TICK_DEPENDENT（默认）
每次 tick 固定推进 multiplier 秒（更准确地说：每 tick 增加一个固定的模拟时间步长）。
结果：帧率越高，时间走得越快；帧率越低，时间走得越慢。

Cesium.ClockStep.SYSTEM_CLOCK_MULTIPLIER（常用）
每次 tick 根据 真实时间流逝 dt 推进：currentTime += dt * multiplier。
结果：时间速度与帧率基本无关（更稳定）。

multiplier 的含义会随 clockStep 变得“像秒/帧”还是“倍速”
在 TICK_DEPENDENT 下，multiplier 更像：每 tick（≈每帧）推进多少模拟秒
例如 multiplier=5：每帧 +5 秒（帧率越高越快）。

在 SYSTEM_CLOCK_MULTIPLIER 下，multiplier 才更像：倍速
例如 multiplier=5：现实 1 秒，模拟 +5 秒（帧率变化影响小）。
```



multiplier的含义会随 `clockStep` 变得“像秒/帧”还是“倍速”

- 在 **`TICK_DEPENDENT`** 下，multiplier 更像：**每 tick（≈每帧）推进多少模拟秒**
  例如 multiplier=5：每帧 +5 秒（帧率越高越快）。
- 在 **`SYSTEM_CLOCK_MULTIPLIER`** 下，multiplier才更像：**倍速**
  例如 multiplier=5：现实 1 秒，模拟 +5 秒（帧率变化影响小）。





```js
viewer.clock.onTick 是 Cesium 的一个 事件(Event)：
每当时钟发生一次 tick（由渲染循环驱动）时就触发
通过addEventListener注册回调函数，每帧调用一次，语法如下：
viewer.clock.onTick.addEventListener(function(clock) {
    // 这里的 clock 参数是 viewer.clock 本身
}
                                     
// 移除监听
window.viewer.clock.onTick.removeEventListener(监听的函数引用);                   
```

所有通过`clock.onTick.addEventListener(listener)`注册的`listener`,都会在这帧被调用一次



### Cesium时间系统工作原理

onTick 事件触发的条件和频率：onTick 事件在每一帧渲染时触发一次

```js
┌─────────────────────────────────────────┐
│   Cesium Clock 时间系统                  │
└─────────────────────────────────────────┘
         ↓
    每帧（frame）执行一次
         ↓
┌─────────────────────────────────────────┐
│  1. 计算当前时间                         │
│  currentTime += (deltaTime * multiplier) │
│  (multiplier = 速度倍数，如1x、2x等)     │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│  2. 触发 onTick 事件回调                 │
│  listener(clock)                        │
└─────────────────────────────────────────┘
         ↓
┌──────────────────────────────────────��──┐
│  3. 更新场景（Entity位置等）             │
│  渲染新一帧                             │
└─────────────────────────────────────────┘
```

用户拖动时间轴时的流程

```js
┌─────────────────────────────────────────┐
│   用户拖动时间轴                        │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│  1. 时间轴组件捕获拖动事件               │
│  更新 viewer.clock.currentTime           │
│  (直接设置到拖动的时间点)               │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│  2. 触发 onTick 事件回调                 │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│  3. Entity 根据新时间重新计算位置        │
│  (如果有 position 回调或插值曲线)        │
└─────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────┐
│  4. 场景立即重新渲染                     │
│  显示该时间点的状态                     │
└─────────────────────────────────────────┘
```





### availability  可见范围



先了解下面两个概念：

TimeInterval = 一个时间段

TimeIntervalCollection = 多个时间段的集合

```js
┌─────────────────────────────┐
│ 2026-01-24 09:00 - 17:00   │ ← TimeInterval 1
├─────────────────────────────┤
│ 2026-01-25 10:00 - 18:00   │ ← TimeInterval 2
├─────────────────────────────┤
│ 2026-01-26 08:00 - 16:00   │ ← TimeInterval 3
└─────────────────────────────┘
    TimeIntervalCollection（整个集合）
```



TimeInterval  时间段

```js
new Cesium.TimeInterval({
  start: startTime,           // 🟢 时间段的开始时间
  stop: stopTime,             // 🔴 时间段的结束时间
  isStartIncluded: true,      // ✅ 开始时间是否包括在内（默认true）
  isStopIncluded: true,       // ✅ 结束时间是否包括在内（默认true）
  data: { custom: 'data' }    // 📦 关联的自定义数据（可选）
})
```



`availability` 的值类型是 `TimeIntervalCollection` ，意味着它可以不止一段时间，而是 **多个不连续的时间段** ：

```
时间轴: ──[存在]──空白──[存在]──空白──[存在]──
         08:00~09:00   11:00~12:00   15:00~16:00
```

它规定了实体在哪些时间段上是存在的



```js
// 定义时间
const startTime = Cesium.JulianDate.fromIso8601('2026-01-24T10:00:00Z');
const stopTime = Cesium.JulianDate.fromIso8601('2026-01-24T12:00:00Z');

// 创建导弹实体，只在 10:00-12:00 这个时间段显示
const missileEntity = viewer.entities.add({
  name: 'Missile',
  position: Cesium.Cartesian3.fromDegrees(120, 30, 1000),
  model: {
    uri: 'src/data/missileDF.glb',
    scale: 5
  },
  // 🔑 availability 定义实体何时可见
  availability: new Cesium.TimeIntervalCollection([
    new Cesium.TimeInterval({
      start: startTime,
      stop: stopTime
    })
  ])
});

// 时间轴推进：
// 09:59 → 导弹不可见 ❌
// 10:00 → 导弹出现 ✅
// 11:00 → 导弹可见 ✅
// 12:00 → 导弹消失 ❌
// 13:00 → 导弹不可见 ❌

// 设置viewer的时间范围和当前时间,开始时间和结束时间分别向前向后移动1小时
viewer.clock.startTime = Cesium.JulianDate.addHours(startTime, -1, new Cesium.JulianDate());
viewer.clock.stopTime = Cesium.JulianDate.addHours(stopTime, 1, new Cesium.JulianDate());
viewer.clock.currentTime = startTime.clone();
viewer.timeline.zoomTo(startTime, stopTime); // 调整时间轴显示范围

viewer.zoomTo(missileEntity);
```





### 外推类型 

用于：`Cesium.SampledProperty`

constant `**ExtrapolationType**` : number

Constants to determine how an interpolated value is extrapolated when querying outside the bounds of available data.

用于确定在超出可用数据范围时，如何对插值结果进行外推的常量

Properties:

| Name          | Type   | Description                                                  |
| ------------- | ------ | ------------------------------------------------------------ |
| `NONE`        | number | No extrapolation occurs.                                     |
| `HOLD`        | number | The first or last value is used when outside the range of sample data. |
| `EXTRAPOLATE` | number | The value is extrapolated.                                   |

用于确定在查询超出可用数据范围时如何外推插值值的常量。

名称 类型 说明

NONE 数字 不进行外推。

HOLD 数字 当超出样本数据范围时，使用第一个或最后一个值。

EXTRAPOLATE 数字 根据已有的采样数据，**延续插值算法的趋势，推算出边界之外的值**（当实验数据暂时缺失时，可以根据提示轨迹趋势预测当前位置，使动画或可视化保持连贯，而不是突然停止或者跳到边界值）。







#### backwardExtrapolationType & forwardExtrapolationType 

`Cesium.SampledProperty`下的属性

两者

```js
@type {ExtrapolationType}            // 类型
@default ExtrapolationType.NONE		// 默认值

// 赋值
property.forwardExtrapolationType  = Cesium.ExtrapolationType.EXTRAPOLATE;
property.backwardExtrapolationType = Cesium.ExtrapolationType.HOLD;
```



`SampledPositionProperty` 本质上是一组 **时间→位置** 的采样点：

```
时间轴:
──────●──────────●──────────●──────────●──────▶
    08:00       09:00      10:00      11:00
    (A点)       (B点)      (C点)      (D点)
```

Cesium 在两个采样点 **之间** 的时间，会自动插值计算位置——这没问题。

**但是，如果请求的时间在采样范围之外呢？**

```
时间轴:
◀── ? ��─●──────────●──────────●──────────●──── ? ──▶
       08:00      09:00      10:00      11:00
     最早样本                           最晚样本
   ↑                                          ↑
这里怎么办？                             这里怎么办？
(backward区域)                          (forward区域)
```

这就是这两个属性要解决的问题。





三种可选值的含义（ExtrapolationType）

NONE（默认值）— 超出范围直接消失

```
时间轴:
◀── 👻 ──●──────────●──────────●──────────●──── 👻 ──▶
        08:00                            11:00
   实体不存在                                  实体不存在
```

> **想象** ：一辆出租车只有在 **接单时间段内** 才出现在地图上，订单之外它根本不存在。

---

HOLD — 停在边界位置不动

```
时间轴:
◀── 📍冻结在A点 ──●──────────●──────────●──────────●──── 📍冻结在D点 ──▶
                08:00                            11:00
```

> **想象** ：一架飞机的飞行记录从08:00开始，如果你把时间轴拖到07:00，飞机会 **停在起飞点一动不动** ，就像在等待起飞；拖到12:00，飞机 **停在降落点** ，就像已经停机了。

---

EXTRAPOLATE — 按趋势推算，继续运动

```
时间轴:
         按趋势推算的幽灵轨迹
◀── 👻➡️ ──●──────────●──────────●──────────●──── 👻➡️ ──▶
          08:00                            11:00
  往回延伸轨迹                                  往前延伸轨迹
```

> **想象** ：气象局用历史数据 **预测** 台风未来的路径——根据已有轨迹的速度和方向， **外推** 出超出记录范围后它可能在哪里。数据没有，但 Cesium 帮你"猜"。

---

### 



















## 场景处理

### 光照

利用场景中的光源照亮整个地球

```js
viewer.scene.globe.enableLighting = true;
```

- 启用地球光照计算（Phong光照模型）
- 根据太阳位置自动计算阴影
- 给地球背太阳一侧添加阴影
- 阴影随时间/太阳位置动态变化

<img src="./assets/微信图片_2026-01-26_075850_837.png" alt="微信图片_2026-01-26_075850_837" style="zoom:50%;" />



SunLight（太阳光源）

```js
// 添加太阳光源
viewer.scene.light = new Cesium.SunLight({
  color: Cesium.Color.WHITE, // 默认白色光  
  intensity: 2.0 // 光照强度
});
```

- 光源方向**由Cesium内部计算**（基于地球公转、地轴倾斜）
- 方向**随模拟时间自动变化**
- 模拟真实太阳位置
- 与 `viewer.clock` 关联
- 可以改变**颜色、强度**，但**方向固定不可控**



DirectionalLight（方向光源）

```js
// 添加方向光
viewer.scene.light = new Cesium.DirectionalLight({
  direction: new Cesium.Cartesian3(-1.0, -1.0, -1.0), // 光线方向
  color: Cesium.Color.WHITE, // 默认白色光
  intensity: 1.0 // 默认强度1.0
});

方向基于 ECEF（地心地固坐标系）
起点	无穷远处（平行光源，如太阳）
光线路径	从方向向量指向的地方射向场景中心

direction: new Cesium.Cartesian3(-1.0, -1.0, -1.0)
含义：光源位于 (-1, -1, -1) 的方向，光线从该方向射向场景
类比：想象太阳在西北方上空，光线照向你的位置
```

direction: new Cesium.Cartesian3(-1.0, -1.0, -1.0)
含义：光源位于 (-1, -1, -1) 的方向，光线从该方向射向场景
类比：想象太阳在西北方上空，光线照向你的位置



### scene事件

```js
// 动画更新
viewer.scene.preUpdate.addEventListener(() => {
  shader.setUniform('u_time', performance.now() / 1000.0);
});

cesium中scene的preUpdate事件的含义是：在场景更新或渲染之前执行的事件，监听这个事件的订阅者会收到Scene和当前时间作为参数

performance.now() 返回的是当前时间戳，单位是毫秒，所以需要除以1000.0转换为秒

CustomShader的setUniform(uniformName, value)方法用于更新着色器中uniform声明的值
```



### 阴影（到时候用建筑白膜图的阴影做示例）

开关阴影

```ts
viewer.shadowMap.enabled = true | false;
```

- 控制是否开启场景阴影计算（光源 + 深度图）。



阴影贴图尺寸：`shadowMap.size`

```ts
viewer.shadowMap.size = 2048; // 256 ~ 4096
```

- Cesium 用一张“深度纹理”（shadow map）来记录“从光源看去的深度信息”。
- 尺寸越大：
  - 阴影边缘的精细程度，取决于有多少像素用来描述它，阴影边缘越细腻，锯齿越少。
  - 显存 & 计算量越大。



柔和阴影：`shadowMap.softShadows`

```ts
viewer.shadowMap.softShadows = true | false;
```

- 原始阴影一般是“硬边界”（一步黑，一步亮）。
- 开启 softShadows 后：
  - Cesium 会在阴影边界附近做一次**模糊/插值**，形成“由深到浅的过渡”。
  - 看起来阴影更“柔和”，也更像真实世界。



阴影透明度：`shadowMap.darkness`

```ts
viewer.shadowMap.darkness = 0 ~ 1;
```

- 实际上控制的是“阴影区域的颜色乘以一个暗度因子”。
- 0：几乎看不出阴影
  1：完全按默认阴影颜色（很深）











### 内置后处理

在 Cesium 里，后处理就是对已经渲染好的场景图像再跑一遍 GLSL shader。

内置的一些效果，比如 FXAA(抗锯齿)、环境光遮蔽、黑白等，都已经封装在 `scene.postProcessStages` 里，我只需要开关和调整参数。



类型关系

```
Viewer
  └─ scene
      └─ postProcessStages() 接收PostProcessStageCollection类型的数据
          ├─ add() → 可以添加 PostProcessStage 或 PostProcessStageComposite
          ├─ ambientOcclusion 属于 PostProcessStageComposite 类型
          ├─ bloom 属于 PostProcessStageComposite
          ├─ fxaa 属于 PostProcessStage 
          └─ [其他自定义的 Stage 或 Composite]
```



PostProcessStageCollection是管理所有Stage的容器，构造函数如下面所示

```js
class PostProcessStageCollection {
    // 构造函数
    constructor() {
        this.bloom = new PostProcessStageComposite();  // 内置属性
        this.fxaa = new PostProcessStage();             // 内置属性
        this.ambientOcclusion = new PostProcessStageComposite();
        this.stages = [];  // 存放用户添加的stages
    }
    
    add(stage) { /* ... */ }
}
```

通过下面的方式进行实例化

```js
viewer.scene.postProcessStages = new PostProcessStageCollection();  // 实例化
```

但`viewer.scene.postProcessStages` **不需要手动实例化**，Cesium 在创建 Viewer 时已经自动创建了。

所以在平时的编码习惯中， `viewer.scene.postProcessStages` 就是 `PostProcessStageCollection` 这个类的一个实例



```js
// 1. PostProcessStage - 最基础的单个后处理效果
const stage1 = new Cesium.PostProcessStage({
    fragmentShader: `
        uniform sampler2D colorTexture;
        in vec2 v_textureCoordinates;
        void main() {
            out_FragColor = texture(colorTexture, v_textureCoordinates);
        }`,
    uniforms: { brightness: 1.2 }
});

/**
new Cesium.PostProcessStage(options)
options: {
  fragmentShader: '...', // 着色器代码
  uniforms: { ... },    // 着色器的uniforms
  ...
}
 */



// 2. PostProcessStageComposite - 多个 Stage 的组合
const comp1 = new Cesium.PostProcessStageComposite({
    stages: [stage1, stage2, stage3],  // 按顺序执行
    inputPreviousStageTexture: true    // 前一个Stage的输出作为下一个的输入
});
/**
new Cesium.PostProcessStageComposite(options)
optinons: {
  stages: [stage1, stage2, ...], // stages是一个数组，里面可以放PostProcessStage或者PostProcessStageComposite对象
  inputPreviousStageTexture: true/false, // 是否让每个stage的输入纹理是前一个stage的输出纹理
  ...
}
 */



// 4. 添加到集合中使用
viewer.scene.postProcessStages.add(stage1);           // 直接添加Stage
viewer.scene.postProcessStages.add(comp1);            // 添加Composite

// 5. 使用内置的 Composite
viewer.scene.postProcessStages.bloom.enabled = true;  // 启用Bloom效果
viewer.scene.postProcessStages.fxaa.enabled = true;   // 启用FXAA抗锯齿
Cesium内部已经在PostProcessStageCollection的构造函数中，创建了bloom和fxaa的实例，所以这里可以直接使用，可以看上面的类结构
```







##### 泛光效果

泛光（bloom）是现代电子游戏中常见的后处理特效，通过图像处理算法将画面中高亮的像素向外 ”扩张“ 形成光晕以增加画面的真实感，能够生动地表达太阳、霓虹灯等光源的亮度。



###### bloom.enabled 开启

```js
// 1. 泛光效果（Bloom）- 让亮的区域发光
viewer.scene.postProcessStages.bloom.enabled = true; //  开启
```

开启对比未开启，开启后朝向光源一侧变的很亮

<img src="./assets/image-20260126160412416.png" alt="image-20260126160412416" style="zoom:50%;" />

关闭之后右侧没有那么亮了

<img src="./assets/image-20260126160505213.png" alt="image-20260126160505213" style="zoom:50%;" />



###### contrast 对比度

```js
viewer.scene.postProcessStages.bloom.uniforms.contrast = 128; 
// 对比度，默认128，范围是[-255.0, 255.0]

// 当对比度为255时，泛光的效果最低，发光区域非常集中，只有极亮的部分会产生明显的发光效果，整体画面显得较为平淡。当这个值逐渐降低时，泛光效果逐渐增强，发光区域变得更加广泛和柔和，到了-255时，泛光效果达到最大，几乎所有亮度较高的区域都会产生明显的发光效果，类似过度曝光
```

255 

<img src="./assets/image-20260126162104959.png" alt="image-20260126162104959" style="zoom:33%;" />

0

<img src="./assets/image-20260126162125064.png" alt="image-20260126162125064" style="zoom:33%;" />

-128

<img src="./assets/image-20260126162220974.png" alt="image-20260126162220974" style="zoom:33%;" />



###### brightness 亮度

```js
viewer.scene.postProcessStages.bloom.uniforms.brightness = -0.3; 
// 亮度，亮度是一个标量值。输入纹理的 RGB 值会被转换为色调、饱和度和亮度（HSB）格式，HSB中的B = B + brightness * B，然后再转换回 RGB 格式。 默认值为0，范围是[-1.0, 1.0]

// 当值越大，图像整体会变得更亮，当转回RGB时，三个分量颜色都会不同程度的增加，从而图像更亮，每个像素颜色更接近白色
```

HSB色彩空间

```
RGB (红绿蓝)  →  HSB (色调、饱和度、亮度)

H (Hue)        = 颜色类型（0-360°）   红、绿、蓝等
S (Saturation) = 颜色饱和度（0-100%） 颜色深浅程度
B (Brightness) = 亮度（0-100%）      颜色明暗程度

纯黑色：   RGB(0,0,0)    → H任意, S任意, B=0%    （无亮度）
纯白色：   RGB(255,255,255) → H任意, S=0%, B=100% （满亮度）
红色：     RGB(255,0,0)  → H=0°, S=100%, B=100%
暗红色：   RGB(128,0,0)  → H=0°, S=100%, B=50%
```

-0.3

<img src="./assets/image-20260126163352384.png" alt="image-20260126163352384" style="zoom:33%;" />

0.1

<img src="./assets/image-20260126163411852.png" alt="image-20260126163411852" style="zoom:33%;" />

0.4

<img src="./assets/image-20260126163437499.png" alt="image-20260126163437499" style="zoom:33%;" />



brightness参数的作用原理

```js
viewer.scene.postProcessStages. bloom.uniforms.brightness = -0.3;

第1步：RGB → HSB 转换
  原始像素：RGB(100, 50, 0)  [暗橙色]
  转换后：  H=30°, S=100%, B=39%  B就是亮度

第2步：亮度加法运算
  新亮度 = 原亮度 + brightness参数值
  新亮度 = 39% + (-0.3×100%) = 39% - 30% = 9%

第3步：HSB → RGB 转换
  新颜色：RGB(23, 11, 0)  [更暗的橙色]
  结果：颜色变暗了！
```



###### glowOnly 只显示发光部分

```js
viewer.scene.postProcessStages.bloom.uniforms.glowOnly = false; 
// 是否只显示发光部分

// 如果设置为 true，则只显示发光效果，其他部分变成黑色，如果设置为false,则在显示发光效果的同时，保留原始图像的其他部分
```

true

<img src="./assets/image-20260126163935648.png" alt="image-20260126163935648" style="zoom:33%;" />

false

<img src="./assets/image-20260126163952421.png" alt="image-20260126163952421" style="zoom:33%;" />





###### delta 、sigma、stepSize  边缘柔和

都是控制泛光的模糊程度，共同作用使泛光边缘更加平滑柔和。

```js
viewer.scene.postProcessStages.bloom.uniforms.delta = 1.0; // 模糊程度
viewer.scene.postProcessStages.bloom.uniforms.sigma = 2.0; // 高斯模糊标准
viewer.scene.postProcessStages.bloom.uniforms.stepSize = 1.0; // 采样步长
// delta、sigma 和 stepSize 是与 PostProcessStageLibrary#createBlurStage 相同的属性。该模糊作用于由图像生成的阴影上，以使它们更加平滑。
```



delta

```js
viewer.scene.postProcessStages.bloom.uniforms.delta = 1.0; // 模糊程度
// delta越接近0, 光晕往外扩散越厉害，反之越小，直到连泛光效果的消失
```

0

<img src="./assets/image-20260126165409341.png" alt="image-20260126165409341" style="zoom:33%;" />

1

<img src="./assets/image-20260126165433693.png" alt="image-20260126165433693" style="zoom:33%;" />

10

<img src="./assets/image-20260126165452389.png" alt="image-20260126165452389" style="zoom:33%;" />

------

**sigma（σ）- 高斯模糊标准差**

```javascript
viewer.scene.postProcessStages.bloom.uniforms.sigma = 2.0; // 高斯模糊标准差
// 范围从0.1到正无穷，值越大，泛光效果模糊越明显，直到泛光效果消失，越接近0.1时，泛光效果的边缘越锐利
```

0.1

<img src="./assets/image-20260126170053881.png" alt="image-20260126170053881" style="zoom:33%;" />

4

<img src="./assets/image-20260126170134161.png" alt="image-20260126170134161" style="zoom:33%;" />

10

<img src="./assets/image-20260126170148402.png" alt="image-20260126170148402" style="zoom:33%;" />



------

**stepSize（Δx）- 采样步长**

```javascript
viewer.scene.postProcessStages.bloom.uniforms.stepSize = 1.0; // 采样步长
// 范围从0到正无穷，值越大，泛光效果效果的边缘越模糊，如果采样步长太大，导致采样点不足，高斯模糊的插值计算精度降低，使得模糊边界出现断层而非连续过渡。光晕出现条纹或阶梯。值越接近0，泛光效果的边缘越清晰锐利
```

0

<img src="./assets/image-20260126171009762.png" alt="image-20260126171009762" style="zoom:50%;" />

4

<img src="./assets/image-20260126171038698.png" alt="image-20260126171038698" style="zoom:50%;" />

10  有重影

<img src="./assets/image-20260126171101455.png" alt="image-20260126171101455" style="zoom:50%;" />

![image-20260126171124440](./assets/image-20260126171124440.png)

可视化（采样过程）：

```Code
stepSize = 0.5:  ●①●②●③●④●⑤●  （取5个点）
stepSize = 1.0:  ●①  ●②  ●③  ●   （取3个点）✓
stepSize = 2.0:  ●① 、  、  ●②    （取2个点）
```

**关键**：更小的stepSize = 更多采样点 = 更精细但更慢



```
高斯模糊效果 = delta（范围）× sigma（柔和度）× stepSize（采样密度）

viewer.scene.postProcessStages.bloom.uniforms.delta = 2.0;     // 扩散范围
viewer.scene.postProcessStages.bloom. uniforms.sigma = 2.0;    // 柔和程度
viewer.scene.postProcessStages.bloom.uniforms. stepSize = 1.0; // 采样精度
```







##### 环境光遮蔽（Ambient Occlusion）

想象一个光线均匀的阴天，没有强烈的直射阳光。在一个房间的角落，墙壁与墙壁、墙壁与地面的交接处，总会比开阔的墙面中央要暗一些。同样，一个物体放在桌子上，在它们接触的缝隙处，也会有一圈非常自然的微弱阴影。

通过在物体交接处和细节内部添加柔和的阴影，让模型看起来不再是“浮”在空中或表面，而是稳固地“接触”着。

<img src="./assets/image-20260121171059597.png" alt="image-20260121171059597" style="zoom:50%;" />



cesium场景中很难看出效果，因为模型没有ao贴图

```js
// 2. 环境光遮蔽（Ambient Occlusion）- 增加阴影细节
// 模拟间接光照下物体之间相互遮挡关系的着色技术。

viewer.scene.postProcessStages.ambientOcclusion.enabled = true;

// 调整下面两个参数能大幅降低计算量，提高性能
viewer.scene.postProcessStages.ambientOcclusion.uniforms.stepCount = 4;       // 默认32 → 4
// 沿着每个方向进行采样的次数
viewer.scene.postProcessStages.ambientOcclusion.uniforms.directionCount = 4;  // 默认8 → 4
// 方向数，从当前像素点向四周发射光线的方向数量
/**
默认值
stepCount: 32        // 每个方向采样32次
directionCount: 8    // 8个方向搜索
总计算量 = 32 × 8 = 256次采样/像素
每帧都要对屏幕每个像素进行256次采样计算，在高分辨率下会严重拖累性能。所以浏览器非常卡顿
设置为 stepCount = 4 和 directionCount = 4 后，
总计算量 = 4 × 4 = 16次采样/像素，性能提升16倍
 */

viewer.scene.postProcessStages.ambientOcclusion.uniforms.intensity = 3.0; // 强度，默认3.0
viewer.scene.postProcessStages.ambientOcclusion.uniforms.bias = 0.1; // 偏移，默认0.1
viewer.scene.postProcessStages.ambientOcclusion.uniforms.lengthCap = 0.03; // 长度上限
viewer.scene.postProcessStages.ambientOcclusion.uniforms.stepSize = 1.0; // 步长
viewer.scene.postProcessStages.ambientOcclusion.uniforms.blurStepSize = 0.86; // 模糊步长
```

- **enabled**: 控制环境光遮蔽是否启用。设置为`true`时启用环境光遮蔽，设置为`false`时禁用。
- **intensity**: 环境光遮蔽的强度。值越大，遮蔽效果越明显。通常取值范围为0.0到10.0。
- **bias**: 环境光遮蔽的偏差。用于减少假阴影或过于强烈的阴影效果。通常取值范围为0.0到1.0。
- **lengthCap**: 计算遮蔽中的最大长度。值越大，遮蔽效果的范围越广。通常取值范围为0.0到10.0。
- **stepCount**: 采样步长。增加采样次数可以减少锯齿，但会增加计算成本。通常取值范围为0到100。
- **directionCount**: 光线进行搜索遮挡物的方向数。值越大，遮蔽效果越精细，但会增加计算成本。通常取值范围为0到10。





##### 景深效果（Depth of Field）

```js
// 景深效果（Depth of Field）- 模拟相机焦距
const depthOfField = viewer.scene.postProcessStages.add(Cesium.PostProcessStageLibrary.createDepthOfFieldStage()); 

// 焦距控制
// 焦距决定了哪个距离的物体是清晰的
// 焦距越近，远处物体越模糊；焦距越远，近处物体越模糊
depthOfField.uniforms.focalDistance = 50.0; // 焦距，单位米


depthOfField.uniforms.delta = 1.0; // 模糊程度
depthOfField.uniforms.sigma = 2.0; // 高斯模糊标准差
depthOfField.uniforms.stepSize = 1.0; // 采样步长
```



是否开启景深的效果对比，未开启和开启对比，开启后明显变模糊了

<img src="./assets/image-20260126193719843.png" alt="image-20260126193719843" style="zoom:33%;" />

<img src="./assets/image-20260126193744644.png" alt="image-20260126193744644" style="zoom:33%;" />

采样步长对比（stepSize）：越小越清晰，越大越模糊（模糊半径变大了），

0.1

<img src="./assets/image-20260126194001538.png" alt="image-20260126194001538" style="zoom:33%;" />

2

<img src="./assets/image-20260126194030458.png" alt="image-20260126194030458" style="zoom:33%;" />

10

<img src="./assets/image-20260126194050127.png" alt="image-20260126194050127" style="zoom:33%;" />



delta和delta，越大越黑，越小越亮

<img src="./assets/image-20260126194609135.png" alt="image-20260126194609135" style="zoom:33%;" />

<img src="./assets/image-20260126194627929.png" alt="image-20260126194627929" style="zoom:33%;" />



##### 黑白效果（Black and White）

```js
const blackAndWhite = viewer.scene.postProcessStages.add(Cesium.PostProcessStageLibrary.createBlackAndWhiteStage());
// 开启后，整个场景变成黑白色调
blackAndWhite.uniforms.gradations = 5.0; // 灰度级别，默认5
// gradations参数控制黑白效果的灰度级别，范围从2到256，灰度级别越高，图像的灰度过渡越平滑，细节越丰富；灰度级别越低，图像的灰度过渡越明显，细节丢失越多。
```

开启和未开启对比

<img src="./assets/image-20260126203538105.png" alt="image-20260126203538105" style="zoom:33%;" />

<img src="./assets/image-20260126203555694.png" alt="image-20260126203555694" style="zoom:33%;" />

灰度级别  细节越来越丰富，颜色过度也越来越顺滑

5

<img src="./assets/image-20260126203555694.png" alt="image-20260126203555694" style="zoom:33%;" />

20

<img src="./assets/image-20260126203717541.png" alt="image-20260126203717541" style="zoom:33%;" />

255

<img src="./assets/image-20260126203738402.png" alt="image-20260126203738402" style="zoom:33%;" />





##### 亮度调整（Brightness）

```js
Cesium.PostProcessStageLibrary.createBrightnessStage() → PostProcessStage
```

实际使用

```js
const brightness = viewer.scene.postProcessStages.add(Cesium.PostProcessStageLibrary.createBrightnessStage());
brightness.uniforms.brightness = 1; // 这个值越大，亮度越高
```

亮度对比

0.3

<img src="./assets/image-20260126204247584.png" alt="image-20260126204247584" style="zoom:33%;" />

1

<img src="./assets/image-20260126204316749.png" alt="image-20260126204316749" style="zoom:33%;" />

3

<img src="./assets/image-20260126204339008.png" alt="image-20260126204339008" style="zoom:33%;" />



`createBrightnessStage()` 的 `brightness` uniform 参数做的是：

**直接缩放像素的 RGB 值**（线性亮度缩放），而不是调整颜色饱和度。

```js
// 内部操作伪代码
output_color = input_color * brightness

// 例如：
// 如果 brightness = 2.5，每个像素的 RGB 都乘以 2.5
// RGB(100, 100, 100) * 2.5 = RGB(250, 250, 250) → 接近白色
```

显示屏的原理：

- **RGB(0, 0, 0)** = 屏幕像素不发光 → **黑色**
- **RGB(128, 128, 128)** = 屏幕像素发出中等亮度的光 → **灰色**
- **RGB(255, 255, 255)** = 屏幕像素发出最大亮度的光 → **白色**

所以这里定的数值越高，像素的RGB值被放的越大，显示屏发出的光也就越亮，场景就显得越亮



##### 夜视效果（Night Vision）

```js
const nightVision = viewer.scene.postProcessStages.add(Cesium.PostProcessStageLibrary.createNightVisionStage());
// 模拟夜视仪的视觉效果
```

<img src="./assets/image-20260126204651687.png" alt="image-20260126204651687" style="zoom:33%;" />



##### HDR

```js
viewer.scene.highDynamicRange = true;
```

未开启与开启对比

<img src="./assets/image-20260126205620192.png" alt="image-20260126205620192" style="zoom: 50%;" />

<img src="./assets/image-20260126205649604.png" alt="image-20260126205649604" style="zoom:50%;" />

```
传统渲染（LDR - Low Dynamic Range）：
颜色范围：0 - 1（8bit，0-255）
          ↓
        受限，很多细节丢失

高动态范围渲染（HDR）：
颜色范围：0 - ∞（32bit 浮点数）
          ↓
        更宽泛，细节丰富
```

















### 自定义后处理

```
new Cesium.PostProcessStage(options)
options: {
  fragmentShader: '...', // 着色器代码
  uniforms: { ... },    // 着色器的uniforms
  ...
}
```

自定义的就是这个方法中的着色器代码和着色器的uniforms



示例流程：

```js
// 1. 创建 Viewer（postProcessStages 已自动创建）
const viewer = new Cesium.Viewer('cesiumContainer');

// 2. 编写自定义 fragmentShader
const stage1 = new Cesium.PostProcessStage({
    fragmentShader: `
        uniform sampler2D colorTexture;
        in vec2 v_textureCoordinates;
        void main() {
            out_FragColor = texture(colorTexture, v_textureCoordinates);
        }`,
    uniforms: { brightness: 1.2 }
});

// 3. 添加到后处理集合
viewer.scene.postProcessStages.add(stage1);
```





比如全局调色、闪烁、雨等，只要写一个 fragment shader 字符串，声明好 `uniform sampler2D colorTexture;` 和 `v_textureCoordinates;`，Cesium 会自动把当前帧的画面传过来。在 shader 里按像素处理后，再输出到 fragColor 就能实现效果

JS + GLSL 合作

这里发生了几件事：

1. **JS 端**：你把 `fragmentShader` 写成一个字符串传给 Cesium。
2. Cesium 内部：
   - 把这段 GLSL 代码编译成 WebGL 着色器。
   - 自动把当前场景渲染好的颜色纹理，绑定到 `colorTexture` 这个 uniform 上。
   - 自动提供 `v_textureCoordinates`（屏幕 UV 坐标）等变量。
3. shader 就是对每个像素（片元）做同样的操作：
   - 读入 `colorTexture` → 得到原始颜色；
   - `color.rgb * scale + offset` → 提亮 + 加偏色；
   - 输出到 `fragColor` → 作为新的画面。

> 所以 **“JS 写 shader”** 这个说法，具体是：
> “用 JS 把 shader 源码传给 Cesium，由 Cesium/浏览器编译并执行在 GPU 上”。



纹理 = 贴在几何表面上的一张图片（2D 数据），用来控制该表面在渲染时的某种属性



### 着色器变量和方法

```
源码位置：
https://github.com/CesiumGS/cesium/tree/461bf597a205752b403f18ad6f0e4d2ef0a6fd65/packages/engine/Source/Shaders/Builtin/Functions
```



`colorTexture`

```glsl
uniform sampler2D colorTexture;
```

- Cesium 在创建后处理 stage 时，会自动把当前帧渲染好的场景颜色纹理绑定到这个 uniform 上。
- 可以理解为：`colorTexture` = 这一帧渲染好的画面（图像）



`v_textureCoordinates`

```glsl
in vec2 v_textureCoordinates;

Cesium 的后处理（PostProcessStage）框架中，会自动提供一个全屏四边形（Fullscreen Quad）的顶点着色器，并将每个片元对应的 UV 坐标通过 v_textureCoordinates 传入片元着色器。
    
- 这是 Cesium 在绘制一个覆盖全屏的矩形时，从顶点着色器传下来的插值变量。
- 顶点 shader 把四个角设成 (0,0)、(1,0)、(1,1)、(0,1)，片元 shader 接收插值后，就能在每个像素上拿到一个 `[0,1]×[0,1]` 的 `uv` 坐标。
- 可以理解为：当前像素在这张图里的坐标（uv）
```



`texture`

```glsl
这是 GLSL 内置函数，用于从纹理中采样颜色值。

vec4 texture(sampler2D sampler, vec2 coord)
// sampler	 要采样的纹理对象
// coord	 UV 坐标（范围 [0,1]），指定采样位置
// 返回值	vec4，即 (r, g, b, a) 四分量颜色向量
```

取像素颜色



`fragColor`

片元着色器用`fragColor`变量输出颜色

```glsl
out vec4 fragColor;

void main() {
  fragColor = vec4(col, 1.0);
}
```



`sampler2D` 是 GLSL 的**纹理采样器类型**，专门用来绑定 2D 纹理



czm_globeDepthTexture

```glsl
这是 Cesium 内置的 GLSL uniform 变量，由 Cesium 自动注入到后处理着色器中。

类型：sampler2D
含义：存储地球表面（Globe）的深度纹理，即仅包含地形/地球表面深度信息的纹理

vec4 rawDepthColor = texture(czm_globeDepthTexture, v_textureCoordinates);
// 在当前片元的UV坐标处，从地球表面深度纹理中采样，rawDepthColor是一个RGBA值，其中R通道存储了深度信息

注意：czm_globeDepthTexture 在某些 Cesium 版本中不可直接在 PostProcessStage 中使用，应改用 PostProcessStage 自动注入的 depthTexture
    
uniform sampler2D depthTexture; // Cesium PostProcessStage 自动提供，无需 czm_ 前缀

void main() {
    float depth = czm_unpackDepth(texture(depthTexture, v_textureCoordinates));
    // ...
}
```



czm_unpackDepth

```glsl
 将一个 RGBA 格式打包的深度值，还原为 [0, 1) 范围内的 float 深度值。
 
 float depth = czm_unpackDepth(rawDepthColor); // 使用Cesium提供的函数czm_unpackDepth将采样到的RGBA值转换为一个浮点数，得到当前片元的深度值
```



czm_windowToEyeCoordinates

```glsl
这是 Cesium 内置函数，作用是将窗口坐标（屏幕像素坐标）+ 深度值 转换为眼睛空间坐标（Eye Space）。

vec4 czm_windowToEyeCoordinates(vec2 fragmentCoordinate, float depth)

转换过程内部大致为：
屏幕坐标(像素) → NDC坐标[-1,1] → 裁剪空间 → 逆投影矩阵 → 眼睛空间
```



czm_inverseView

```glsl
这是 Cesium 内置的 uniform 矩阵变量，表示视图矩阵的逆矩阵
视图矩阵（View Matrix）	世界坐标 → 眼睛空间
czm_inverseView	         眼睛空间 → 世界坐标（ECEF）
```





#### 深度值

```glsl
深度值不是距离屏幕的实际距离，而是一个归一化的非线性值。
主要作用是判断遮挡关系，GPU 通过比较深度值决定哪个片元可见，近处覆盖远处

深度值存储在深度缓冲区（Depth Buffer）中，范围是 [0.0, 1.0]：
深度值	 含义
0.0	   近裁剪面（Near Plane）
1.0	   远裁剪面（Far Plane）

片元在裁剪空间（Clip Space）中 Z 坐标经过透视除法后映射到 [0,1] 的结果

为什么是非线性的？
透视投影矩阵会导致深度值在近处精度高、远处精度低：
真实距离:  1   2   4   8   16  ...  1000
深度值:   0.1 0.3 0.6 0.8 0.9 ...  0.999
靠近相机的物体深度变化大，远离相机的物体深度变化极小

Cesium 中深度值的完整转换链：
深度纹理(打包值)
    ↓ czm_unpackDepth()
归一化深度 [0,1]
    ↓ czm_windowToEyeCoordinates()
眼睛空间坐标（Eye Space，真实的相机距离）
    ↓ czm_inverseView
世界坐标（World Space，ECEF坐标系）
```



<hr>


### 内部shader实现

“天空变成青色”效果：

```ts
const fs = `
    uniform sampler2D colorTexture;
    in vec2 v_textureCoordinates;
    uniform float scale;
    uniform vec3 offset;
    out vec4 fragColor;
    void main() {
        vec4 color = texture(colorTexture, v_textureCoordinates);
        fragColor = vec4(color.rgb * scale + offset, 1.0);
    }`;

const customStage = new Cesium.PostProcessStage({
  name: 'bright',
  fragmentShader: fs,
  uniforms: {
    scale: 1.1,
    offset: new Cesium.Cartesian3(0.1, 0.2, 0.3),
  },
});

stages.add(customStage);
```

- 用 `offset = (0.1, 0.2, 0.3)`，蓝/绿通道偏高，会让整体图像偏青。
- 再乘上 `scale = 1.1`，整体亮度提升 10%。

<hr>


屏幕闪烁效果

```ts
const fsBlink=/*glsl*/`
    uniform sampler2D colorTexture;
    in vec2 v_textureCoordinates; // 纹理坐标
    uniform float brightness;
    out vec4 fragColor; // 输出颜色

    void main(){
        vec3 col=texture(colorTexture,v_textureCoordinates).rgb;
        float iTime=czm_frameNumber/60.;
        iTime=fract(iTime); 

        if(iTime>0.5){
            col*=brightness;
        }else{
            col*=0.5;
        }
        fragColor=vec4(col,1.); 
    }
`
const stageBlink = new Cesium.PostProcessStage({
  name: 'blink',
  fragmentShader: fsBlink, // 着色器代码
  uniforms: {
    brightness: 2.0, // 亮度缩放
  }
})
stages.add(stageBlink);
```

`czm_frameNumber`

- Cesium 内置的 GLSL 变量，每一帧自增 1。
- 你用 `czm_frameNumber / 60.` → 大约“每秒加 1”（假设 FPS ~ 60）。

`fract(x)`

- GLSL 函数，返回 `x` 的**小数部分**。
- 比如：`fract(1.3) = 0.3`, `fract(5.8) = 0.8`。

组合起来效果

- 前半秒（`iTime ≤ 0.5`）：变暗一半。
- 后半秒（`iTime > 0.5`）：乘以 `brightness`（比如 2.0 → 变亮）。

<hr>


雨水效果

```ts
// 自定义后处理==降雨效果
const fragmentShader = /*glsl*/ `uniform sampler2D colorTexture;
    uniform float rainSpeed;
    uniform float rainSize;
    in vec2 v_textureCoordinates;
    out vec4 fragColor;
    

    float hash(float x){
        return fract(sin(x*133.3)*13.13);
    }

    void main(void){
        float time = (czm_frameNumber / 60.)*rainSpeed;
        // 类似于shaderToy当中的iResolution
        vec2 resolution = czm_viewport.zw;

        // 构造uv坐标，起手式 uv坐标可以帮我们画图形，做倾斜，做重复等等
        // vec2 uv1=gl_FragCoord.xy/resolution.xy  这个uv的取值范围0-1
        // uv -= .5; 将uv的原点放在屏幕原点，将坐标范围变为-0.5,0.5
        // uv.x *= iResolution.x / iResolution.y;  自适应
        vec2 uv=(gl_FragCoord.xy*2.-resolution.xy)/min(resolution.x,resolution.y);

        // 自定义一个雨的颜色
        vec3 c=vec3(.5,.5,.5);
        uv*=rainSize;
        // 计算了一个二维的旋转矩阵
        // a用来构造旋转矩阵，是雨水的倾斜度
        float a=-.4;
        float si=sin(a),co=cos(a);
        uv*=mat2(co,-si,si,co);
        uv*=length(uv+vec2(0,4.9))*.3+1.;

        // v变量中使用了hash噪声，是随机在画布上生成雨点
        float v=1.-sin(hash(floor(uv.x*100.))*2.);
        // b变量使用了time 用来让雨水下落的
        float b=clamp(abs(sin(20.*time*v+uv.y*(5./(2.+v))))-.95,0.,1.)*20.;
        c*=v*b; 
        
        vec4 img=texture(colorTexture, v_textureCoordinates);
        vec3 rainCol=mix(img.rgb,c,0.6);
        fragColor=vec4(rainCol,1.);
}`;

const stageRain = new Cesium.PostProcessStage({
  name: 'rain',
  fragmentShader: fragmentShader, // 着色器代码
  uniforms: {
    rainSpeed: 1.0, // 雨水速度
    rainSize: 1.0   // 雨水大小
  }
});
// stages.add(stageRain);
```

1. 用 `czm_frameNumber` 做时间 `time`，叠加 `rainSpeed` 控制速度。
2. 用 `gl_FragCoord` 和 `czm_viewport.zw` 构造屏幕空间的 `uv` 坐标。
3. 用一系列数学操作（旋转矩阵、`hash` 噪声、`sin`/`clamp`）画出斜线 + 随机性 + 下落效果。
4. 最后用 `mix(img.rgb, c, 0.6)` 把原图和雨水颜色混在一起。

大概结构：

```
vec4 img = texture(colorTexture, v_textureCoordinates); // 原画面
vec3 rainCol = mix(img.rgb, c, 0.6);                    // 和“雨”颜色混合
fragColor = vec4(rainCol, 1.);
```





### 外部shader实现

基本流程

1. 引入 fragment shader（GLSL 文件）

   ```ts
   import myShader from './lib/shader/xxx.glsl?raw';
   ```

2. 用 `PostProcessStage` 包装 shader

   ```ts
   const stage = new Cesium.PostProcessStage({
     name: 'myStage',
     fragmentShader: myShader,
     uniforms: {
       // JS 侧给 shader 传的参数，名字要和 GLSL 里的 uniform 对上
       u_SomeFloat: 1.0,
       u_SomeColor: Cesium.Color.GRAY,
       u_SomeVec3: new Cesium.Cartesian3(1, 2, 3),
     },
   });
   ```

3. 加入场景后处理集合

   ```ts
   viewer.scene.postProcessStages.add(stage);
   ```

4. 在 GUI / 逻辑里动态改 uniforms

   ```ts
   gui.add(vModel, 'u_SomeFloat', 0, 100).onChange(() => {
     stage.uniforms.u_SomeFloat = vModel.u_SomeFloat;
   });
   ```





#### shader关键词含义

从深度纹理/颜色纹理采样：

```glsl
vec4 depthColor = texture(depthTexture, v_textureCoordinates);  // 从深度纹理中取颜色值
vec4 col        = texture(colorTexture, v_textureCoordinates);
```



`czm_unpackDepth`把 RGBA 打包的深度值解出：

```glsl
float depth = czm_unpackDepth(depthColor);
```



`czm_windowToEyeCoordinates` 从屏幕坐标 → 相机坐标：

```glsl
vec4 positionEC = czm_windowToEyeCoordinates(gl_FragCoord.xy, depth);
```



 `czm_inverseView` 从相机坐标 → 世界坐标：

```glsl
vec4 positionWC = czm_inverseView * vec4(positionEC.xyz / positionEC.w, 1.0);
vec3 worldPos   = positionWC.xyz / positionWC.w; // 齐次坐标归一化
```



根据世界坐标/高度/距离算某种“权重/遮罩”，再用做插值混合：

```glsl
float fogRate = ...;                       // 0~1 的雾化率 / 权重
vec3 blended = mix(col.rgb, u_FogColor, fogRate); // 在 原色&雾色 之间插值
fragColor = vec4(blended, col.a);
```



mix

```glsl
vec3 Color = mix(col.rgb, u_fog, fogRate);
fragColor = vec4(Color, 1);
```

`mix(a, b, t)=a * (1 - t) + b * t`



#### 线性距离雾

计算片元到相机的距离

```glsl
float calDistanceFog() {
  vec4 depthColor = texture(depthTexture, v_textureCoordinates);
  float depth = czm_unpackDepth(depthColor);

  vec4 positionEC = czm_windowToEyeCoordinates(gl_FragCoord.xy, depth);
  float linearDepth = -positionEC.z / positionEC.w;
  return linearDepth;
}
```

- `depthTexture`：颜色通道里打包了深度信息。
- `czm_unpackDepth`：**Cesium 内置函数**，把 RGBA 还原成一个 0~1 的深度值。
- `czm_windowToEyeCoordinates`：
  - 输入：屏幕像素坐标 `gl_FragCoord.xy` + 深度；
  - 输出：当前片元在“相机空间（Eye Coordinates）”里的 3D 位置。
- `-positionEC.z / positionEC.w`：
  在相机空间里，`-z` 方向通常是“看出去”的方向，用它来当“距离”比较方便，所以叫“线性深度”。



线性雾化率

```glsl
float calFogRate(float distanceFog) {
  float fogRate = (distanceFog - u_Near) / (u_Far - u_Near);
  fogRate = clamp(fogRate, 0.0, 1.0);
  return fogRate;
}
```

- 当 `distanceFog <= u_Near` → 结果 ≤ 0 → clamp 为 0：**完全不雾**。
- 当 `distanceFog >= u_Far` → 结果 ≥ 1 → clamp 为 1：**完全是雾色**。
- 中间线性过渡：**越远雾越浓**。



#### 用雾化率 `fogRate` 做颜色混合

GLSL

```glsl
void main() {
  float distanceFrag = calDistanceFog();
  float fogRate = calFogRate(distanceFrag);

  vec4 col = texture(colorTexture, v_textureCoordinates);

  vec3 Color = mix(col.rgb, u_fog, fogRate);
  fragColor = vec4(Color, 1);
}
```

`mix(a, b, t)=a * (1 - t) + b * t`





#### 世界坐标验证

1. 从 depthTexture 采样深度，用 czm_unpackDepth 解析
2. `positionEC = czm_windowToEyeCoordinates(gl_FragCoord.xy, depth)`
3. `positionWC = czm_inverseView * vec4(positionEC.xyz/positionEC.w, 1.0)`
4. 如果 `positionWC.z >= 0` → 把片元染成绿色
   如果 `< 0` → 染成红色

效果：整个地球被分成 z>=0 的绿区 和 z<0 的红区，方便检查世界坐标的方向和范围。



<img src="./assets/image-20251124211833555.png" alt="image-20251124211833555" style="zoom:33%;" />







#### 后处理画圆

算每个像素的世界坐标 → 看它离圆心多远 → 在半径内的像素染红”。

1. `getWorldPosition()`：
   - 从 depthTexture 采样 + czm_unpackDepth
   - `positionEC = czm_windowToEyeCoordinates(...)`
   - `positionWC = czm_inverseView * vec4(positionEC.xyz/positionEC.w, 1.0)`
   - 返回 `positionWC.xyz / positionWC.w` 作为世界坐标

2. SDF 圆函数：
   - `sdCircle(center, worldPos, r) = distance(center, worldPos) - r`
   - <0 在圆内，=0 在圆上，>0 在圆外

3. 在 main 里：
   - 先取原场景颜色：`col = texture(colorTexture, v_textureCoordinates)`
   - 计算 d = sdCircle(...)
   - 如果 d < 0：`fragColor = 红色`
     否则：`fragColor = 原色 col`



<img src="./assets/image-20251124212045739.png" alt="image-20251124212045739" style="zoom:50%;" />



#### 线性高度雾

目标：按“高度（离地表多高）”控制雾的强度。

1. 从深度纹理恢复世界坐标 `positionWC`
2. 计算海拔高度：
   - `fogHeight = length(positionWC) - u_EarthRadius`

3. u_FoyByHeight = (startHeight, startRate, endHeight, endRate)
   - `t = clamp((fogHeight - startHeight) / (endHeight - startHeight), 0, 1)`
   - `fogRate = mix(startRate, endRate, t)`

4. 根据 fogRate 混合原色和雾色：
   - `blended = mix(col.rgb, u_FogColor, fogRate)`
   - `fragColor = vec4(blended, col.a)`

注意两个 mix 的含义不同：

- calFogRate 里的 mix：在“起始雾浓度 & 结束雾浓度”之间插值 → 得到一个标量 fogRate
- main 里的 mix：在“原场景颜色 & 雾颜色”之间插值 → 得到最终颜色



<img src="./assets/image-20251124214142925.png" alt="image-20251124214142925" style="zoom:50%;" />







### 体积云

基于后处理实现体渲染

#### 原理

```
在渲染体积云时，每一个像素点有3个重要属性：

1 云的密度。

2 接收到的总光能。

3 光能反射比例，即有多少光能能反射到人的眼睛（摄像机）。



```

云密度

体积云是基于RayMarching的渲染，在步进过程中采样噪声图，累加噪声值得到云的密度

下图中获取了7个采样点

![image-20260226110735272](./assets/image-20260226110735272.png)



总光能

在每一个采样点，向光源方向再做步进，累加噪声值得到该点在光源方向的“深度”。

举两个例子：

在某点向光源方向做8次步进，前3次在云的内部，后5次在云的外部。

该点的深度为（1 + 1 + 1 + 0 + 0 + 0 + 0 + 0）/ 8 = 0.375。

![image-20260226110831696](./assets/image-20260226110831696.png)

在某点向光源方向做8次步进，前5次在云的内部，后3次在云的外部。

该点的深度为（1 + 1 + 1 + 1 + 1 + 0 + 0 + 0）/ 8 = 0.625。

![image-20260226110911448](./assets/image-20260226110911448.png)

深度越大，接收到的光能越少。

在现实世界中，光能随深度的衰减是非线性的，当深度为0.25时，光能为0.75，但当深度为0.5时，光能可能就只剩0.3了。

所以还需要加入衰减系数。

在现实中，光在传递过程中会发生折射，反射，衍射等现象，涉及到的物理公式比较多。而在游戏中，我们追求的是以更低的计算量得到让人信服的效果，所以这里我选择不使用公式，用一条曲线来控制衰减。

![image-20260226111058593](./assets/image-20260226111058593.png)



光能反射比例

算出了一个点接收到的总光能，但这些光能只有一部分能反射到我们的眼睛（摄像机），从该点向摄像机方向做步进，累加噪声值得到该点在摄像机方向的“深度”，深度越大损失的光能越多，传递到摄像机的光能 = 接收到的总光能 - 中途损失的光能。

这个步进过程和计算云密度的步进过程是对称的，所以可以用当前点的云密度估算反射比例。

![image-20260226111436246](./assets/image-20260226111436246.png)



#### 局部体积云（待总结）

对于局部的体积云，一般使用一个Box来表示云的范围，为了提高程序性能，光线步进一般只在该Box中进行，所以对于每条光线，首先需要获取其在Box上的起点和终点。

![image-20260226115548040](./assets/image-20260226115548040.png)

```
光线：从点 P 出发，方向为向量 d
Box：由 B_min 和 B_max 两个角点定义的轴对齐包围盒（AABB）
```

光线方程为：
$$
\text{Ray}(t) = P + t \cdot \vec{d}
$$
目标是求出光线进入Box的 t_enter 和离开Box的 t_exit。
$$
\vec{d}是单位向量， t的数值 = 光线走过的实际距离（世界空间单位）
$$
单轴求交原理（以X轴为例）

图(b)展示了在某一个轴方向（X轴）上，光线与两个平面的交点：
$$
t_{0,x} = \frac{b_x^{min} - P_x}{d_x} \quad \Rightarrow \quad \text{光线与左平面 } x = b_x^{min} \text{ 的交点}
$$

$$
t_{1,x} = \frac{b_x^{max} - P_x}{d_x} \quad \Rightarrow \quad \text{光线与右平面 } x = b_x^{max} \text{ 的交点}
$$





#### 全球体积云

前面实现的体积云是局部的，即将云的渲染范围限定在一个指定的Box内，也就是光线步进的范围限定在Box内，而要实现全球体积云，需要根据地球的形状设置体积云的渲染范围，因为地球是一个椭圆，所以体积云的渲染范围是一个球壳

<img src="./assets/image-20260226192324308.png" alt="image-20260226192324308" style="zoom:67%;" />

限定体积云的渲染范围，即为限定光线步进的范围，所以只需要将上一节的光线步进范围从Box换到球壳，一个球壳其实就是两个不同半径的球所组成的，大球减去小球即为球壳范围。求取光线步进的范围，即是求取光线步进的起点和终点（参考rayBoxDst）方法，在球壳中求光线步进的起点和终点，其实就是通过求射线与球的交点来实现的，这分为多种情况

![image-20260226192836499](./assets/image-20260226192836499.png)







## Cesium渲染器

Cesium将底层与WebGL交互的操作封装到了一个渲染模块，也可以称该模块为Cesium渲染器

Cesium渲染器源码位于packages/engine/Source/Renderer目录下：

![image-20260203174551207](./assets/image-20260203174551207.png)

```
一、Cesium的渲染器会涉及到很多WebGL的概念
1、Buffer 顶点缓冲对象，即常说的VBO，位于Renderer/Buffer.js

2、VertextArray 顶点数组对象，即常说的VAO，创建一个VAO，最常用的是VertexArray.fromGeometry（）方法，而不是new VertexArray（），位于Renderer/VertexArray.js

3、Texture 纹理，对WebGL的Texture的封装，位于Renderer/Texture.js

4、ShaderProgram 着色器程序 对着色器的编译，链接等操作的封装，位于Renderer/ShaderProgram.js，搜索构造函数function ShaderProgram(options){...}

5、FrameBuffer 帧缓冲区对象，即常说的FBO，用于离屏渲染，位于Renderer/FrameBuffer.js

6、Context 对WebGL绘图上下文的封装，Context.prototype.draw方法是Cesium调用WebGL底层绘图接口的入口，位于Renderer/Context.js

7.Pass 定义了 Cesium 中的渲染通道（Render Pass），用于控制不同类型几何体的渲染顺序，位于Renderer/RenderState.js

8、RenderState 渲染状态，比如是否启用深度、剔除、透明、颜色混合，等状态的封装，位于Renderer/RenderState.js
```





```
二、DrawCommand绘制指令
位于Renderer/DrawCommand.js
DrawCommand 是 Cesium 渲染器的核心，相当于Cesium上层接口与Cesium渲染器的桥梁，所熟知的Entity
、Primitive、包括地形、影像最后都会被封装成一个个的DrawCommand交由Contenxt执行。
DrawCommand 类封装了执行一次draw call需要的所有数据，包括顶点数据、变换数据、着色器程序等信息。

使用DrawCommand类进行绘图，可以绕过Cesium对几何Geometry和外观Appearance的限制（使用Entity和Primtive绘图时是不是必须指定几何和外观），使用更接近WebGL的绘图方式进行绘图。DrawCommand类是Cesium最底层的绘图接口，该接口没有对外公开，要使用此类，必须参考源码。

创建一个DrawCommand必须具有以下属性：
1、vertexArray 顶点数组，向 GPU 传递顶点属性、索引（可选的）数组等几何信息
2、shaderProgram 着色器程序，包括顶点着色器和片元着色器
3、renderState 渲染状态，封装如深度测试（depthTest）、剔除（cull）、混合（blending）等状态类型的参数设置
4、pass 渲染通道，用于决定该指令渲染的优先级

不是必须但是常用的属性：
1、modelMatrix 模型矩阵
2、uniformMap 自定义的uniform参数
```





```
三、UniformState：统一值状态
位于Renderer/UniformState.js
统一值状态，存储了场景渲染一帧需要传递到着色器中的数据，比如模型矩阵、视图矩阵、投影矩阵等等，并且提供了update方法更新这些数据。

1、UniformState类在Context类初始化时被实例化，并且整个程序只实例化一个对象

2、场景在渲染每一帧前，都会先调用uniformState.update方法更新uniformState对象中存储的数据
```





```
四、AutomaticUniforms
位于Renderer/AutomaticUniforms.js
AutomaticUniforms类（准确来说是一个对象）对UniformState的取值进行了封装，因为UniformState中的数据在传入着色器时需要知道其类型，不同的数据类型设置值的方式是不一样的（比如顶点数据和纹理数据）。

1、着色器程序从UniformState中获取数据时，会通过AutomaticUniforms对象来获取，比如获取视图矩阵

2、AutomaticUniforms最后在ShaderProgram类（着色器程序）中被使用，创建着色器程序时，会准备所需要的Uniform数据
```





```
五、内置的GLSL库
位于：packages\engine\Source\Shaders\Builtin\CzmBuiltins.js
Cesium内置了一个小型的GLSL库，里面定义了一些方法、结构体、常量，这些内容会自动注入到着色器程序中，所以我们在使用Cesium着色器时，可以直接使用这些内容
文档地址：https://cesium.com/downloads/cesiumjs/releases/b28/Documentation/index.html
```







### DrawCommand自定义Primitive

四棱锥建模效果如图

![image-20260203195857068](./assets/image-20260203195857068.png)



顶点数据

```js
const positions = new Float32Array([ 
    // //前
  0, 0, -1,//  0
  1, -1, 1,//  1
  -1, -1, 1,// 2
  //左
  0, 0, -1,//  0 
  -1, -1, 1,// 2
  -1, 1, 1,//  3
  // //后
  0, 0, -1,//  0 
  -1, 1, 1,//  3
  1, 1, 1,//   4
  // //右
  0, 0, -1,//  0 
  1, 1, 1,//   4
  1, -1, 1,//  1
  //上
  1, -1, 1,//  1
  1, 1, 1,//   4
  -1, 1, 1,//  3
  //上
  1, -1, 1,//  1
  -1, 1, 1,//  3
  -1, -1, 1,// 2
]);
```



有了顶点数据，就可以创建VAO（顶点数组对象，vertex array object）了,打开VertexArray.js的源码，可以看到在定义VertexArray类的上面,Cesium提供了几个创建VertexArray对象的示例代码。

```js
 * @example
 * // Example 1. Create a vertex array with vertices made up of three floating point
 * // values, e.g., a position, from a single vertex buffer.  No index buffer is used.
 * const positionBuffer = Buffer.createVertexBuffer({
 *     context : context,
 *     sizeInBytes : 12,
 *     usage : BufferUsage.STATIC_DRAW
 * });
 * const attributes = [
 *     {
 *         index                  : 0,
 *         enabled                : true,
 *         vertexBuffer           : positionBuffer,
 *         componentsPerAttribute : 3,
 *         componentDatatype      : ComponentDatatype.FLOAT,
 *         normalize              : false,
 *         offsetInBytes          : 0,
 *         strideInBytes          : 0 // tightly packed
 *         instanceDivisor        : 0 // not instanced
 *     }
 * ];
 * const va = new VertexArray({
 *     context : context,
 *     attributes : attributes
 * });
```



创建一个VAO，首先需要创建VBO（顶点缓冲区对象，vertex buffer object），因为数据实际是存储在VBO中，而VAO只是对VBO的组织。所以先创建一个VBO，创建VBO的方式有很多，具体可以参考源码，这里使用最简单的方式，传入类型化的顶点数据数组。

```js
const positionBuffer = Cesium.Buffer.createVertexBuffer({
  context : viewer.scene.context, 
  typedArray: positions,
  usage : Cesium.BufferUsage.STATIC_DRAW
});
```



再创建顶点数组对象 VOA(vertex array object)

```js
const attributes = [
  {
    index                  : 0,
    enabled                : true,
    vertexBuffer           : positionBuffer,
    componentsPerAttribute : 3,
    componentDatatype      : Cesium.ComponentDatatype.FLOAT,
    normalize              : false,
    offsetInBytes          : 0,
    strideInBytes          : 0, // tightly packed
    instanceDivisor        : 0 // not instanced
  }
];
const vertexArray = new Cesium.VertexArray({
  context : scene.context,
  attributes : attributes
});
```



创建shaderProgram, 即着色器程序

```js
const vs = `
  in vec3 position;
  void main() {
    gl_Position = czm_modelViewProjection * vec4(position, 1.0);
  }
`;
const fs = `
  void main() {
    out_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
  }
`;
const shaderProgram = Cesium.ShaderProgram.fromCache({
  context: viewer.scene.context,
  vertexShaderSource: vs,
  fragmentShaderSource: fs,
  attributeLocations: { 
    position: 0
  }
})
```



创建renderState, 即渲染状态，如果没有特殊的设置，可以通过 Cesium.RenderState.fromCache从缓存中获取一个渲染状态对象。

```js
const renderState = Cesium.RenderState.fromCache({
  depthTest: {
    enabled: true,
  },
});
```



渲染优先级：因为我们是自定义Primitve，所以这里设置为Pass.OPAQUE表示不透明

```js
const pass=Cesium.Pass.OPAQUE;
```



相关参数准备完成，接下来就可以创建DrawCommand了

```js
// 定义参考系
let m = Cesium.Transforms.eastNorthUpToFixedFrame(Cesium.Cartesian3.fromDegrees(110.0, 30.0, 10)); 

// 创建DrawCommand绘制指令
const command = new Cesium.DrawCommand({
  modelMatrix: m,
  vertexArray: vertexArray,
  shaderProgram: shaderProgram,
  pass: pass,
  renderState: renderState,
})
```



创建完DrawCommand，接下来就需要通过Primtive接口，使DrawCommand能够加入到场景的渲染队列中

```js
// 使用Primitive接口
class MyPrimitive {
  constructor() {}

  update(frameState) {
    // 将自定义的DrawCommand添加到渲染队列中
    frameState.commandList.push(command);
  }

  isDestroyed() {
    return false;
  }
}
viewer.scene.primitives.add(new MyPrimitive());
```

所有的DrawCommand都必须添加到frameState.commandList这个集合中才会被渲染，场景渲染每一帧前都会清空这个集合，然后调用Primitive的update方法将command添加到frameState.commandList中。



#### 完整代码

```js
const positions = new Float32Array([ 
    // //前
  0, 0, -1,//  0
  1, -1, 1,//  1
  -1, -1, 1,// 2
  //左
  0, 0, -1,//  0 
  -1, -1, 1,// 2
  -1, 1, 1,//  3
  // //后
  0, 0, -1,//  0 
  -1, 1, 1,//  3
  1, 1, 1,//   4
  // //右
  0, 0, -1,//  0 
  1, 1, 1,//   4
  1, -1, 1,//  1
  //上
  1, -1, 1,//  1
  1, 1, 1,//   4
  -1, 1, 1,//  3
  //上
  1, -1, 1,//  1
  -1, 1, 1,//  3
  -1, -1, 1,// 2
]);

// 创建顶点缓冲区对象 VBO(vertex buffer object)
const positionBuffer = Cesium.Buffer.createVertexBuffer({
  context : viewer.scene.context, 
  typedArray: positions,
  usage : Cesium.BufferUsage.STATIC_DRAW
});

// 创建顶点数组对象 VOA(vertex array object)
const attributes = [
  {
    index                  : 0,
    enabled                : true,
    vertexBuffer           : positionBuffer,
    componentsPerAttribute : 3,
    componentDatatype      : Cesium.ComponentDatatype.FLOAT,
    normalize              : false,
    offsetInBytes          : 0,
    strideInBytes          : 0, // tightly packed
    instanceDivisor        : 0 // not instanced
  }
];
const vertexArray = new Cesium.VertexArray({
  context : viewer.scene.context,
  attributes : attributes
});

// 创建shaderProgram, 即着色器程序
const vs = `
  in vec3 position;
  void main() {
    gl_Position = czm_modelViewProjection * vec4(position, 1.0);
  }
`;
const fs = `
  void main() {
    out_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
  }
`;
const shaderProgram = Cesium.ShaderProgram.fromCache({
  context: viewer.scene.context,
  vertexShaderSource: vs,
  fragmentShaderSource: fs,
  attributeLocations: { 
    position: 0
  }
})

// 创建renderState, 即渲染状态
const renderState = Cesium.RenderState.fromCache({
  depthTest: {
    enabled: true,
  },
});

// 渲染优先级 pass
const pass=Cesium.Pass.OPAQUE; // 不透明几何体渲染通道

// 定义参考系
let m = Cesium.Transforms.eastNorthUpToFixedFrame(Cesium.Cartesian3.fromDegrees(110.0, 30.0, 10)); 
// 创建DrawCommand绘制指令
const command = new Cesium.DrawCommand({
  modelMatrix: m,
  vertexArray: vertexArray,
  shaderProgram: shaderProgram,
  pass: pass,
  renderState: renderState,
})

// 使用Primitive接口
class MyPrimitive {
  constructor() {}

  update(frameState) {
    // 将自定义的DrawCommand添加到渲染队列中
    frameState.commandList.push(command);
  }

  isDestroyed() {
    return false;
  }
}
viewer.scene.primitives.add(new MyPrimitive());

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 50.0),
});
```



![image-20260203205713703](./assets/image-20260203205713703.png)



#### 法线&光照

Primitive自定义几何笔记章节中提到，通过`Cesium.GeometryPipeline.computeNormal(geometry) → Geometry`这个方法计算顶点法线

该方法需要传入一个Geometry，最后返回一个Geometry，返回的Geometry中就包含了法线信息。所以先通过顶点数据创建一个Geometry：

因为计算法线必须传入顶点索引，所以需要手动计算顶点索引。这里一定要创建包围盒，不然顶点法线后面显示不出来，因为Primitive/DrawCommand 在渲染前都会做可见性判断（视锥裁剪、地平线裁剪等），而这些判断高度依赖“包围体”，如果 geometry.boundingSphere 缺失/不正确，Cesium 无法正确判断它在不在相机视锥内，结果通常是：直接被当作不可见而跳过

```js
// 创建具有法线信息的geometry，转成顶点数组对象 VOA(vertex array object)
const indices = new Uint16Array([
  0, 1, 2,
  3, 4, 5,
  6, 7, 8,
  9, 10, 11,
  12, 13, 14,
  15, 16, 17,
]);
// 获取包围盒
let boundingSphere=Cesium.BoundingSphere.fromVertices(positions);
// 创建几何体对象
let geometry = new Cesium.Geometry({
  attributes: {
    position: new Cesium.GeometryAttribute({
      componentDatatype: Cesium.ComponentDatatype.DOUBLE,
      componentsPerAttribute: 3,
      values: positions
    })
  },
  indices: indices,
  primitiveType: Cesium.PrimitiveType.TRIANGLES,
  boundingSphere:boundingSphere,
})
geometry = Cesium.GeometryPipeline.computeNormal(geometry);
```



创建DrawCommand需要的是VAO而不是Geometry，此时就会用到VertextArray类上的一个名为fromGeometry的方法，该方法就是将一个Geometry对象转换为VAO。

```js
// 将几何体转换为顶点数组
const vertexArray = Cesium.VertexArray.fromGeometry({
  context: viewer.scene.context,
  geometry: geometry,
  attributeLocations: Cesium.GeometryPipeline.createAttributeLocations(geometry),
})

/**
Cesium.GeometryPipeline
Cesium中用于处理和优化几何体数据的工具集，提供了一系列静态方法来对Geometry对象进行转换、计算和优化操作

Cesium.GeometryPipeline.createAttributeLocations(geometry) → object
建立几何体中的属性名称与着色器中的顶点属性位置（location index) 的映射关系
{
  'position': 0,      // ← 属性名 'position' 映射到 location 0
  'normal': 1,        // ← 属性名 'normal' 映射到 location 1
  'texCoord0': 2      // ← 属性名 'texCoord0' 映射到 location 2
}
*/
```



现在顶点数据中已经有法线信息了，但是运行程序还是老样子，没啥变化，因为我们还没在着色器中使用，那肯定是没变化，接下来我们修改着色器。

修改着色器

在顶点着色器中增加了normal变量用于接收输入值，并且新增两个输出变量，分别为眼睛坐标系下的坐标和法线。然后是片元着色器，新增两个变量接收顶点着色器的输入值，然后采用冯氏着色法进行着色器，

```js
// 创建shaderProgram, 即着色器程序
const vs = `
  in vec3 position; // 顶点位置
  in vec3 normal; // 顶点法线

  out vec3 v_positionEC; // 眼睛坐标系下的顶点位置
  out vec3 v_normalEC;   // 眼睛坐标系下的顶点法线

  void main() {
    vec4 positionEC = czm_modelView * vec4(position, 1.0); // 眼睛坐标系下的顶点位置
    v_positionEC = positionEC.xyz; // 传递给片元着色器

    v_normalEC = czm_normal * normal; // 变换法线并传递给片元着色器

    gl_Position = czm_modelViewProjection * vec4(position, 1.0);
  }
`;
const fs = `
  in vec3 v_positionEC; // 眼睛坐标系下的顶点位置
  in vec3 v_normalEC;   // 眼睛坐标系下的顶点法线

  void main() {
    vec3 N = normalize(v_normalEC); // 法线

    // 使用场景内置太阳光方向（眼睛坐标系）
    vec3 L = normalize(czm_sunDirectionEC);

    // 视线方向（眼睛在原点，看向 -Z）
    vec3 V = normalize(-v_positionEC);

    // Phong：环境光 + 漫反射 + 高光
    float diff = max(dot(N, L), 0.0);
    vec3 R = reflect(-L, N);
    float spec = pow(max(dot(R, V), 0.0), 32.0);

    vec3 baseColor = vec3(1.0, 0.0, 0.0);
    vec3 color = baseColor * (0.2 + 0.8 * diff) + vec3(1.0) * (0.25 * spec);

    out_FragColor = vec4(color, 1.0);
  }
`;
const shaderProgram = Cesium.ShaderProgram.fromCache({
  context: viewer.scene.context,
  vertexShaderSource: vs,
  fragmentShaderSource: fs,
  // 当在shader 里写了 in vec3 position 和 in vec3 normal时，Cesium 会自动根据 shader 代码里的 in 变量名称进行推断和绑定。你不需要手动指定 attributeLocations，手动指定也行
  attributeLocations: { 
    position: 0,
    normal: 1
  }
})
```



让顶点法线可视化

```js
const normalDebugPrimitive = Cesium.createTangentSpaceDebugPrimitive({
  geometry: geometry,           // 使用同一个几何体
  length: 0.5,                  // 法线长度0.5米（便于查看）
  modelMatrix: m                // 使用同样的变换矩阵
});
viewer.scene.primitives.add(normalDebugPrimitive);
```

![image-20260204111301395](./assets/image-20260204111301395.png)



#### 完整代码

```js
const positions = new Float32Array([ 
    // //前
  0, 0, -1,//  0
  1, -1, 1,//  1
  -1, -1, 1,// 2
  //左
  0, 0, -1,//  0 
  -1, -1, 1,// 2
  -1, 1, 1,//  3
  // //后
  0, 0, -1,//  0 
  -1, 1, 1,//  3
  1, 1, 1,//   4
  // //右
  0, 0, -1,//  0 
  1, 1, 1,//   4
  1, -1, 1,//  1
  //上
  1, -1, 1,//  1
  1, 1, 1,//   4
  -1, 1, 1,//  3
  //上
  1, -1, 1,//  1
  -1, 1, 1,//  3
  -1, -1, 1,// 2
]);

// 创建顶点缓冲区对象 VBO(vertex buffer object)
const positionBuffer = Cesium.Buffer.createVertexBuffer({
  context : viewer.scene.context, 
  typedArray: positions,
  usage : Cesium.BufferUsage.STATIC_DRAW
});

// 创建具有法线信息的geometry，转成顶点数组对象 VOA(vertex array object)
const indices = new Uint16Array([
  0, 1, 2,
  3, 4, 5,
  6, 7, 8,
  9, 10, 11,
  12, 13, 14,
  15, 16, 17,
]);
// 获取包围盒
let boundingSphere=Cesium.BoundingSphere.fromVertices(positions);
// 创建几何体对象
let geometry = new Cesium.Geometry({
  attributes: {
    position: new Cesium.GeometryAttribute({
      componentDatatype: Cesium.ComponentDatatype.DOUBLE,
      componentsPerAttribute: 3,
      values: positions
    })
  },
  indices: indices,
  primitiveType: Cesium.PrimitiveType.TRIANGLES,
  boundingSphere:boundingSphere,
})
geometry = Cesium.GeometryPipeline.computeNormal(geometry);

// 将几何体转换为顶点数组
const vertexArray = Cesium.VertexArray.fromGeometry({
  context: viewer.scene.context,
  geometry: geometry,
  attributeLocations: Cesium.GeometryPipeline.createAttributeLocations(geometry),
})

// 创建shaderProgram, 即着色器程序
const vs = `
  in vec3 position; // 顶点位置
  in vec3 normal; // 顶点法线

  out vec3 v_positionEC; // 眼睛坐标系下的顶点位置
  out vec3 v_normalEC;   // 眼睛坐标系下的顶点法线

  void main() {
    vec4 positionEC = czm_modelView * vec4(position, 1.0); // 眼睛坐标系下的顶点位置
    v_positionEC = positionEC.xyz; // 传递给片元着色器

    v_normalEC = czm_normal * normal; // 变换法线并传递给片元着色器

    gl_Position = czm_modelViewProjection * vec4(position, 1.0);
  }
`;
const fs = `
  in vec3 v_positionEC; // 眼睛坐标系下的顶点位置
  in vec3 v_normalEC;   // 眼睛坐标系下的顶点法线

  void main() {
    vec3 N = normalize(v_normalEC); // 法线

    // 使用场景内置太阳光方向（眼睛坐标系）
    vec3 L = normalize(czm_sunDirectionEC);

    // 视线方向（眼睛在原点，看向 -Z）
    vec3 V = normalize(-v_positionEC);

    // Phong：环境光 + 漫反射 + 高光
    float diff = max(dot(N, L), 0.0);
    vec3 R = reflect(-L, N);
    float spec = pow(max(dot(R, V), 0.0), 32.0);

    vec3 baseColor = vec3(1.0, 0.0, 0.0);
    vec3 color = baseColor * (0.2 + 0.8 * diff) + vec3(1.0) * (0.25 * spec);

    out_FragColor = vec4(color, 1.0);
  }
`;
const shaderProgram = Cesium.ShaderProgram.fromCache({
  context: viewer.scene.context,
  vertexShaderSource: vs,
  fragmentShaderSource: fs,
  // 当在shader 里写了 in vec3 position 和 in vec3 normal时，Cesium 会自动根据 shader 代码里的 in 变量名称进行推断和绑定。你不需要手动指定 attributeLocations，手动指定也行
  attributeLocations: { 
    position: 0,
    normal: 1
  }
})

// 创建renderState, 即渲染状态
const renderState = Cesium.RenderState.fromCache({
  depthTest: {
    enabled: true,
  },
});

// 渲染优先级 pass
const pass=Cesium.Pass.OPAQUE; // 不透明几何体渲染通道

// 定义参考系
let m = Cesium.Transforms.eastNorthUpToFixedFrame(Cesium.Cartesian3.fromDegrees(110.0, 30.0, 10)); 

// 创建DrawCommand绘制指令
const command = new Cesium.DrawCommand({
  modelMatrix: m,
  vertexArray: vertexArray,
  shaderProgram: shaderProgram,
  pass: pass,
  renderState: renderState,
})

// 使用Primitive接口
class MyPrimitive {
  constructor() {}

  update(frameState) {
    // 将自定义的DrawCommand添加到渲染队列中
    frameState.commandList.push(command);
  }

  isDestroyed() {
    return false;
  }
}
viewer.scene.primitives.add(new MyPrimitive());

viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(110.0, 30.0, 50.0),
});


const normalDebugPrimitive = Cesium.createTangentSpaceDebugPrimitive({
  geometry: geometry,           // 使用同一个几何体
  length: 0.5,                  // 法线长度0.5米（便于查看）
  modelMatrix: m                // 使用同样的变换矩阵
});
viewer.scene.primitives.add(normalDebugPrimitive);
```



#### 使用步骤

```
使用自定义DC绘制指令绘制自定义primitive的步骤

1. 提供顶点数据
2. 提供面索引数据，又顶点组成的三角形网格
3. 创建几何体Geometry对象，将几何体转换为顶点数组对象vertex array object（VAO）
4. 创建着色器程序shader program
5. 创建渲染状态render state
6. 创建渲染优先级pass
7. 创建DrawCommand绘制指令
8. 创建Primitive接口，并在update方法中将DrawCommand添加到渲染队列中
9. 将自定义Primitive添加到场景的primitive集合中
```





#### DrawCommand (DC) 对比 Primitive API (官方)

```js
抽象层级	底层：直接操作 WebGL	高层：Cesium 封装


最直观的：
Primitive API (官方)传递着色器的 Uniform 传递非常繁琐（需要 Material + fabric + 手动更新），Shader 会被 Cesium 模板包裹，难以调试


虽然 Appearance.vertexShaderSource/fragmentShaderSource 看起来很自由，但实际上是假自由——Cesium 会在你的 Shader 外部自动添加代码，导致：
❌ Uniform 传递不便
❌ Shader 模板受限

而 DrawCommand 是真正的"完全自由"
```



当在 Appearance 中传入自定义 shader 时，Cesium 不是直接使用你的 shader，而是：

顶点着色器被包裹：

```js
// 你写的
attribute vec3 position;
void main() {
  gl_Position = czm_modelViewProjection * vec4(position, 1.0);
}

// ⬇️ Cesium 实际编译的（伪代码）
// Cesium 前置代码...
uniform mat4 czm_modelView;
uniform mat4 czm_projection;
// ... 其他内置 uniform ...

attribute vec3 position;
void main() {
  gl_Position = czm_modelViewProjection * vec4(position, 1.0);
  // ⬇️ Cesium 后置代码（看不到）...
}
```



片元着色器被包裹：

```js
// 你写的
void main() {
  out_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}

// ⬇️ Cesium 实际编译的（伪代码）
struct czm_materialInput {
  vec2 st;           // 纹理坐标
  vec3 normal;       // 法线
  vec3 tangent;      // 切线
  vec3 bitangent;    // 副切线
  vec3 positionToEyeEC;
  // ...
};

vec4 myFragmentShader(...) {
  // 你的代码
  out_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}

void main() {
  czm_materialInput materialInput = czm_getMaterialInput(...);
  czm_material material = czm_getMaterial(materialInput);
  // ... 可能还有其他处理 ...
  gl_FragColor = myFragmentShader(...);
}
```



比如传递时间参数

```js
DC的片元着色器：
const fs = `
  uniform float u_time; // ⬅ Cesium 不会传递这个！
  in vec2 v_st;
  void main() {
    float noise = snoise(v_st - vec2(0.0, u_time));
    // ...
  }
`;
对于官方primitive ：必须用 Material + fabric，但非常繁琐
const material = new Cesium.Material({
  fabric: {
    uniforms: {
      time: 0 // ⬅ 需要每帧手动更新
    },
    source: fs
  }
});

// 每帧更新
viewer.scene.preRender.addEventListener(() => {
  material.uniforms.time = performance.now() / 1000;
});
```



### Cesium绘制渲染的流程

new Cesium.Viewer()是Cesium一切的起点，从该构造函数开始，看看Cesium绘制一帧的大概流程

```
打开cesium源码，打开Viewer.js源码：packages\widgets\Source\Viewer\Viewer.js

打开构造函数function Viewer(...) {...}
```

![image-20260209094702529](./assets/image-20260209094702529.png)



可以看到在Viewer类的构造函数中，通过我们传递进来的参数，进行了一些初始化，其中最核心的是：

```js
// ...existing code...

  // CesiumWidget 是 Viewer 的“引擎核心”入口：
  // - 负责创建 canvas + WebGL context
  // - 构建 Scene/Camera/ScreenSpaceEventHandler/CreditDisplay 等渲染所需对象
  // - 初始化 DataSourceDisplay（Entity/DataSource -> Primitive/DrawCommand 的可视化链路）
  // - 可选地启动默认渲染循环（requestAnimationFrame）
  //
  // Viewer 本身更多是 UI 组合层：创建各类 widget，并把部分 options 透传给 CesiumWidget。
  const cesiumWidget = new CesiumWidget(cesiumWidgetContainer, {
    // baseLayer 的处理比较“策略化”：
    // - 如果启用 BaseLayerPicker 且提供了 selectedImageryProviderViewModel，
    //   或者用户显式传了 baseLayer/imageryProvider，
    //   则这里先传 false：禁止 CesiumWidget 自动加“默认底图”，由 Viewer/BaseLayerPicker 后续接管。
    // - 否则传 undefined：允许 CesiumWidget 走其内部默认逻辑（通常会加默认世界影像）。
    baseLayer:
      (createBaseLayerPicker &&
        defined(options.selectedImageryProviderViewModel)) ||
      defined(options.baseLayer) ||
      defined(options.imageryProvider)
        ? false
        : undefined,

    // Clock：CesiumWidget/Scene 的时间驱动来源；shouldAnimate 决定默认是否推进时间
    clock: clock,
    shouldAnimate: options.shouldAnimate,

    // 与场景渲染内容相关：天空盒/大气
    skyBox: options.skyBox,
    skyAtmosphere: options.skyAtmosphere,

    // 场景模式与地理基础：3D/2D/CV、椭球体、投影
    sceneMode: options.sceneMode,
    ellipsoid: options.ellipsoid,
    mapProjection: options.mapProjection,

    // globe 可为 false（无地球）或自定义 Globe；影响地形/影像/大气等渲染分支
    globe: options.globe,

    // 渲染管线关键开关：OIT（半透明排序）、阴影、MSAA 等
    orderIndependentTranslucency: options.orderIndependentTranslucency,
    shadows: options.shadows,
    terrainShadows: options.terrainShadows,
    msaaSamples: options.msaaSamples,
    depthPlaneEllipsoidOffset: options.depthPlaneEllipsoidOffset,

    // DataSourceClock 跟随策略（新加入的数据源可驱动 clock）
    automaticallyTrackDataSourceClocks:
      options.automaticallyTrackDataSourceClocks,

    // WebGL context 创建参数（alpha/antialias/preserveDrawingBuffer 等）
    contextOptions: options.contextOptions,

    // 默认渲染循环：true 则 CesiumWidget 内部用 requestAnimationFrame 驱动 tick/update/render
    useDefaultRenderLoop: options.useDefaultRenderLoop,
    targetFrameRate: options.targetFrameRate,
    showRenderLoopErrors: options.showRenderLoopErrors,

    // 分辨率策略：true 则忽略 devicePixelRatio，按浏览器推荐分辨率渲染（通常更省性能）
    useBrowserRecommendedResolution: options.useBrowserRecommendedResolution,

    // CreditDisplay 的宿主 DOM：Viewer 默认把 credits 放在 bottomContainer
    creditContainer: defined(options.creditContainer)
      ? options.creditContainer
      : bottomContainer,
    creditViewport: options.creditViewport,

    // 数据源集合（若外部传入，则认为由外部拥有，生命周期可能不由 Viewer 销毁）
    dataSources: options.dataSources,

    // 只渲染 3D（节省部分内存/分支）
    scene3DOnly: scene3DOnly,

    // 2D 地图模式（是否无限滚动等）
    mapMode2D: options.mapMode2D,

    // Canvas 获取焦点时是否 blur 当前 activeElement（偏 UI/交互体验）
    blurActiveElementOnCanvasFocus: options.blurActiveElementOnCanvasFocus,

    // 显式渲染模式：true 时仅在需要时 render（配合 Scene#requestRender）
    requestRenderMode: options.requestRenderMode,
    maximumRenderTimeChange: options.maximumRenderTimeChange,
  });

// ...existing code...
```

接下来看看构造函数`CesiumWidget`是什么样的

#### CesiumWiget.render

打开CesiumWidget.js源码:

![image-20260209095440458](./assets/image-20260209095440458.png)

`CesiumWidget`这个类中创建了scene对象

```js
new Scene({...})：创建“渲染世界”的核心容器
这一段是 CesiumWidget 构造阶段最关键的动作之一：把 canvas + WebGL 上下文 + FrameState + 各种渲染管线配置绑定到一个 Scene 实例上。后续“一帧怎么画”基本都围绕 Scene.initializeFrame() 和 Scene.render(time) 展开。


// ...existing code...
// Scene 是 Cesium 的渲染核心：持有 WebGL 上下文、帧状态(FrameState)、相机、图元集合、渲染管线等。
// 后续每一帧的主流程基本是：
//   scene.initializeFrame();   // 准备本帧状态（时间、视锥、uniforms、事件等）
//   scene.render(time);        // 更新并生成 draw commands，提交 WebGL 绘制
const scene = new Scene({
  canvas: canvas, // 绑定渲染目标 canvas（Scene 内部会创建/管理 WebGL context）
  contextOptions: options.contextOptions, // WebGL/context 创建参数透传
  creditContainer: innerCreditContainer,  // Credits 显示容器（通常在 widget 底部）
  creditViewport: creditViewport,         // Credits 弹层的 viewport（默认覆盖 widget）

  // 地理基础参数：影响坐标/投影、地球与相关计算
  ellipsoid: ellipsoid,
  mapProjection: options.mapProjection,

  // 渲染管线关键开关：半透明排序(OIT)、是否只保留 3D path、阴影、MSAA 等
  orderIndependentTranslucency: options.orderIndependentTranslucency,
  scene3DOnly: options.scene3DOnly ?? false,
  shadows: options.shadows,
  mapMode2D: options.mapMode2D,

  // 显式渲染模式：开启后不会“每帧都画”，而是只有在需要时才渲染
  requestRenderMode: options.requestRenderMode,
  maximumRenderTimeChange: options.maximumRenderTimeChange,

  // 深度平面/椭球偏移 与 MSAA 多重采样参数（影响画质与性能）
  depthPlaneEllipsoidOffset: options.depthPlaneEllipsoidOffset,
  msaaSamples: options.msaaSamples,
});
this._scene = scene;
// ...existing code...

// 约束相机的“参考上方向”为世界 Z 轴（地球场景常用），
// 让用户交互（旋转/环绕）时更不容易产生不受控的 roll（画面翻转/歪斜）。
scene.camera.constrainedAxis = Cartesian3.UNIT_Z;
```



`CesiumWidget`这个类中使用了默认的循环渲染

```js
// 通过属性 setter 启动/停止默认渲染循环（requestAnimationFrame）。
// 这里先把内部缓存值置为 undefined，确保下面赋值会触发 setter 的“值变化”分支。
this._useDefaultRenderLoop = undefined;
this.useDefaultRenderLoop = options.useDefaultRenderLoop ?? true;
```

看看`useDefaultRenderLoop`属性的定义

```js
  /**
   * 获取或设置：此 widget 是否由自己控制默认渲染循环。
   *
   * - 为 true 时：内部使用 requestAnimationFrame 驱动循环，
   *   每帧会执行 widget.resize()、widget.render()，并推进仿真时钟（clock.tick）。
   * - 为 false 时：不会自动渲染；需要你在自定义循环中手动调用 resize/render。
   *
   * 如果渲染过程中发生错误，会触发 {@link Scene} 的 renderError 事件，
   * 并将该属性自动置为 false（停止默认渲染）。错误处理后需要手动设回 true 才会继续。
   *
   * @memberof CesiumWidget.prototype
   * @type {boolean}
   */
  useDefaultRenderLoop: {
    get: function () {
      return this._useDefaultRenderLoop;
    },
    set: function (value) {
      // 只有当值确实变化时才处理，避免重复启动循环
      if (this._useDefaultRenderLoop !== value) {
        this._useDefaultRenderLoop = value;

        // 从 false -> true 时，且当前循环未运行，则启动 requestAnimationFrame 循环
        if (value && !this._renderLoopRunning) {
          startRenderLoop(this);
        }
      }
    },
  },
```



看到在`startRenderLoop`函数内调用`requestAnimationFrame`函数进行不断地循环渲染，我们所说的绘制一帧，就是`requestAnimationFrame`函数执行一次的过程。

```js
function startRenderLoop(widget) {
  // 标记：默认渲染循环已启动
  widget._renderLoopRunning = true;

  let lastFrameTime = 0;

  // rAF 回调：浏览器每次准备绘制新帧时调用（不等于每次都执行真正渲染，取决于节流/显式渲染等）
  function render(frameTime) {
    if (widget.isDestroyed()) {
      return; // widget 已销毁则停止
    }

    // useDefaultRenderLoop = true 才由 CesiumWidget 自己驱动循环
    if (widget._useDefaultRenderLoop) {
      try {
        const targetFrameRate = widget._targetFrameRate;

        // 未设置目标帧率：每次 rAF 都进行一次 resize + render，然后继续排队下一次 rAF
        if (!defined(targetFrameRate)) {
          widget.resize(); // 处理 canvas 像素大小/分辨率变化（可能影响 viewport/drawingBuffer）
          widget.render(); // 推进 clock + 更新 scene + scene.render(...)（一帧的主要工作）
          requestAnimationFrame(render);
        } else {
          // 设置了 targetFrameRate：仍然每次 rAF 回调都会触发，但用时间间隔控制“是否真的渲染”
          const interval = 1000.0 / targetFrameRate;
          const delta = frameTime - lastFrameTime;

          // 只有当距离上次真正渲染的时间超过 interval，才执行一次 resize + render
          if (delta > interval) {
            widget.resize();
            widget.render();
            // 让节流更平滑：修正 lastFrameTime，避免时间漂移累积
            lastFrameTime = frameTime - (delta % interval);
          }

          // 无论本次是否渲染，都继续请求下一次 rAF（节流是靠 if 判断跳过 render 实现的）
          requestAnimationFrame(render);
        }
      } catch (error) {
        // 渲染异常：停止默认循环，并可选弹出错误面板
        widget._useDefaultRenderLoop = false;
        widget._renderLoopRunning = false;

        if (widget._showRenderLoopErrors) {
          const title =
            "An error occurred while rendering.  Rendering has stopped.";
          widget.showErrorPanel(title, undefined, error);
        }
      }
    } else {
      // useDefaultRenderLoop 被关闭：不再继续循环
      widget._renderLoopRunning = false;
    }
  }

  // 启动第一次 rAF
  requestAnimationFrame(render);
}
```



在`requestAnimationFrame`的`render`函数内部，核心代码为：

```js
resize()：处理 canvas 尺寸/分辨率变化，必要时更新 drawing buffer，并触发后续需要重绘。
render()：驱动 clock、更新场景并最终调用到 Scene.render(...) 产生 draw commands 并提交 WebGL。
```



`widget.resize`方法

```js
/**
 * 更新 canvas 尺寸、相机纵横比（frustum aspect ratio）以及视口大小。
 * 当 useDefaultRenderLoop=true 时，该函数会在默认渲染循环中按需自动调用；
 * 若 useDefaultRenderLoop=false，则需要外部自定义循环手动调用。
 */
CesiumWidget.prototype.resize = function () {
  const canvas = this._canvas;

  // 如果没有强制 resize，并且：
  // - canvas 的 CSS 尺寸没有变化（clientWidth/clientHeight）
  // - 当前设备像素比没有变化（window.devicePixelRatio）
  // 则无需重新配置 drawingBuffer / frustum，直接返回以避免每帧开销。
  if (
    !this._forceResize &&
    this._canvasClientWidth === canvas.clientWidth &&
    this._canvasClientHeight === canvas.clientHeight &&
    this._lastDevicePixelRatio === window.devicePixelRatio
  ) {
    return;
  }

  // 清除强制 resize 标记；后续由尺寸变化或外部设置再触发
  this._forceResize = false;

  // 依据 CSS 尺寸 * pixelRatio（由 useBrowserRecommendedResolution、devicePixelRatio、resolutionScale 决定）
  // 计算并设置 canvas.width/canvas.height（即 WebGL drawing buffer 尺寸），并更新 _canRender 等状态。
  configureCanvasSize(this);

  // 根据新的 canvas 像素宽高，更新相机 frustum 的 aspectRatio（或 top/bottom），确保投影不变形。
  configureCameraFrustum(this);

  // 请求场景在下一次机会进行重绘。
  // 在 requestRenderMode=true（显式渲染）时尤其重要：没有 requestRender() 可能不会自动画新帧。
  this._scene.requestRender();
};
```



`widget.render`方法

```js
/**
 * 渲染一帧场景。
 * 当 useDefaultRenderLoop=true 时，该函数会在默认 rAF 循环里自动调用；
 * 否则需要外部自定义循环手动调用。
 */
CesiumWidget.prototype.render = function () {
  if (this._canRender) {
    // 准备本帧状态：更新 frameState、相机/视锥、清理上一帧状态、处理事件等（具体逻辑在 Scene 内部）
    this._scene.initializeFrame();

    // 推进仿真时间并触发 clock 相关事件（onTick 等）；返回当前帧用于渲染的 currentTime
    const currentTime = this._clock.tick();

    // 进行本帧渲染：更新各类 primitive/tiles/dataSource 可视化，生成 draw commands，提交 WebGL 绘制
    this._scene.render(currentTime);
  } else {
    // 当 canvas 像素尺寸为 0（或其他不可渲染状态）时，跳过 scene.render，
    // 但仍然 tick 时钟以保持时间与事件链路的一致性。
    this._clock.tick();
  }
};
```



#### 流程图谱

![LoRA vLLM RAG Pipeline-2026-02-09-085707](./assets/LoRA vLLM RAG Pipeline-2026-02-09-085707.png)





#### 剖析Scene.render()

打开Scene.js源码:

![image-20260209110715681](./assets/image-20260209110715681.png)

```js
Scene类构造函数中的重要初始化操作，

1、创建Context绘图上下文对象，使用原生WebGL绘图，您绘先获取一个绘图上下文对象，而此处的绘图上下文，就是类似WebGL绘图上下文。
this._context = new Context(canvas, contextOptions);


2、创建FrameState这状态对象，帧状态对象保存了当前一帧的相关信息。
this._frameState = new FrameState(
    context,
    new CreditDisplay(creditContainer, " • ", creditViewport),
    this._jobScheduler
  );


3、创建存储Primitive的集合（包含贴地类型）
this._primitives = new PrimitiveCollection({ countReferences });
this._groundPrimitives = new PrimitiveCollection({ countReferences });


4、创建ScreenSpaceCameraController对象，基于canvas的鼠标输入修改相机的位置和方向
a、创建Camera相机
const camera = new Camera(this);
if (this._logDepthBuffer) {
  camera.frustum.near = 0.1;
  camera.frustum.far = 10000000000.0;
} 
this._defaultView = new View(this, camera, viewport);
this._view = this._defaultView;

b、创建环境对象，如太阳、月亮、大气环境等等
this.atmosphere = new Atmosphere();
this.fog = new Fog();
this.light = new SunLight();
```



渲染一帧最后是调用scene对象的`initializeFrame`方法

```js
initializeFrame 的主干流程

Scene.prototype.initializeFrame = function () {
  // 1. 定期销毁已释放的着色器和纹理（每120帧一次）
  if (this._shaderFrameCount++ === 120) {
    this._shaderFrameCount = 0;
    this._context.shaderCache.destroyReleasedShaderPrograms();
    this._context.textureCache.destroyReleasedTextures();
  }

  // 2. 更新补间动画
  this._tweens.update();

  // 3. 更新地球高度（如果需要）
  if (this._globeHeightDirty) {
    // ... 地球高度相关逻辑
  }
  
  // 4. 更新相机地下状态
  this._cameraUnderground = isCameraUnderground(this);
  
  // 5. 更新地球半透明状态
  this._globeTranslucencyState.update(this);

  // 6. 更新屏幕空间相机控制器
  this._screenSpaceCameraController.update();
  
  // 7. 更新设备方向相机控制器（如果存在）
  if (defined(this._deviceOrientationCameraController)) {
    this._deviceOrientationCameraController.update();
  }

  // 8. 最后更新相机
  this.camera.update(this._mode);
  this.camera._updateCameraChanged();
};
```



`scene.render`方法

```js
// ...existing code...
Scene.prototype.render = function (time) {
  // ===== 1) 帧开始：对外抛出“即将 update”的事件（用户/外部系统常在这里做状态更新）=====
  this._preUpdate.raiseEvent(this, time);

  const frameState = this._frameState;
  frameState.newFrame = false;

  // 如果未传入 time，则使用当前系统时间（JulianDate）
  if (!defined(time)) {
    time = JulianDate.now();
  }

  // ===== 2) 检测相机是否有变化（相机变化通常意味着需要重新渲染，并可能影响地表高度等缓存）=====
  const cameraChanged = this._view.checkForCameraUpdates(this);
  if (cameraChanged) {
    this._globeHeightDirty = true;
  }

  // ===== 3) requestRenderMode(显式渲染) 的“是否需要渲染本帧”判定 =====
  // - 非显式渲染：每次调用 render 都会渲染
  // - 显式渲染：只有在“被请求渲染/相机变化/某些渲染关键状态变脏/Morphing”等情况下才渲染
  let shouldRender =
    !this.requestRenderMode ||
    this._renderRequested ||
    cameraChanged ||
    this._logDepthBufferDirty ||
    this._hdrDirty ||
    this.mode === SceneMode.MORPHING;

  // 显式渲染下：如果 maximumRenderTimeChange 设置了阈值，
  // 当仿真时间变化超过阈值，也会触发渲染（用于动画/光照随时间变化等）
  if (
    !shouldRender &&
    defined(this.maximumRenderTimeChange) &&
    defined(this._lastRenderTime)
  ) {
    const difference = Math.abs(
      JulianDate.secondsDifference(this._lastRenderTime, time),
    );
    shouldRender = shouldRender || difference > this.maximumRenderTimeChange;
  }

  // ===== 4) 若决定渲染：推进 frameNumber/time，并清除“脏标记/渲染请求”=====
  if (shouldRender) {
    this._lastRenderTime = JulianDate.clone(time, this._lastRenderTime);
    this._renderRequested = false;
    this._logDepthBufferDirty = false;
    this._hdrDirty = false;

    // frameNumber 递增（wrap 防止无限增大），并写入 frameState.time
    const frameNumber = CesiumMath.incrementWrap(
      frameState.frameNumber,
      15000000.0,
      1.0,
    );
    updateFrameNumber(this, frameNumber, time);
    frameState.newFrame = true;
  }

  // ===== 5) “渲染前更新”阶段（带 try/catch，出错会触发 renderError 事件）=====
  // 这里会做一些与渲染 pass 无关的更新：jobScheduler、globe.update、picking.update、creditDisplay.update 等
  tryAndCatchError(this, prePassesUpdate);

  // ===== 6) 各种 Pass 的更新/预加载逻辑（主要围绕 primitives/tileset 的预取、拾取相关准备等）=====
  if (this.primitives.show) {
    tryAndCatchError(this, updateMostDetailedRayPicks);
    tryAndCatchError(this, updatePreloadPass);
    tryAndCatchError(this, updatePreloadFlightPass);

    // 当不渲染时（显式渲染模式常见），仍然跑一个“延迟检查 pass”
    // 用来判断是否有资源/异步任务完成，从而需要唤醒渲染
    if (!shouldRender) {
      tryAndCatchError(this, updateRequestRenderModeDeferCheckPass);
    }
  }

  // ===== 7) 对外抛出“更新结束、即将渲染”的事件 =====
  this._postUpdate.raiseEvent(this, time);

  // ===== 8) 真正渲染：仅在 shouldRender=true 时才进入 =====
  // render(scene) 是内部主渲染函数：updateFrameState -> updateEnvironment -> updateAndExecuteCommands -> resolveFramebuffers -> endFrame
  if (shouldRender) {
    this._preRender.raiseEvent(this, time);
    frameState.creditDisplay.beginFrame();
    tryAndCatchError(this, render);
  }

  // ===== 9) 帧末：调试与后处理更新（不一定要求本帧实际渲染）=====
  updateDebugShowFramesPerSecond(this, shouldRender);
  tryAndCatchError(this, postPassesUpdate);

  // ===== 10) 执行 afterRender 队列（常用于触发事件/Promise resolve 等，刻意避免在渲染中途执行）=====
  callAfterRenderFunctions(this);

  // ===== 11) 对外抛出“渲染结束”的事件，并结束 credits 统计 =====
  if (shouldRender) {
    this._postRender.raiseEvent(this, time);
    frameState.creditDisplay.endFrame();
  }
};
// ...existing code...
```





所以最核心的一步是调用了`tryAndCatchError(this, render);`

```js
function render(scene) {
  const frameState = scene._frameState;

  const context = scene.context;
  const { uniformState } = context;

  // 本帧使用默认 View（包含 camera、viewport、passState、oit、framebuffer 管理器等）
  const view = scene._defaultView;
  scene._view = view;

  // ===== 1) 准备 FrameState（清 commandList、更新 cullingVolume、光照、模式等派生状态）=====
  scene.updateFrameState();

  // 标记本帧要进行“正常渲染”pass；并根据是否有 selected 的后处理，标记 postProcess pass
  frameState.passes.render = true;
  frameState.passes.postProcess = scene.postProcessStages.hasSelected;

  // Tileset 当前处于“渲染”pass（不同 pass 会影响 tileset 的加载/选择策略）
  frameState.tilesetPassState = renderTilesetPassState;

  // ===== 2) 计算背景色（HDR 时对颜色做 gamma 相关处理，使清屏/背景符合 HDR 流程）=====
  let backgroundColor = scene.backgroundColor ?? Color.BLACK;
  if (scene._hdr) {
    backgroundColor = Color.clone(backgroundColor, scratchBackgroundColor);
    backgroundColor.red = Math.pow(backgroundColor.red, scene.gamma);
    backgroundColor.green = Math.pow(backgroundColor.green, scene.gamma);
    backgroundColor.blue = Math.pow(backgroundColor.blue, scene.gamma);
  }
  frameState.backgroundColor = backgroundColor;

  // 大气/雾等环境参数写入 frameState，并更新雾效果
  frameState.atmosphere = scene.atmosphere;
  scene.fog.update(frameState);

  // ===== 3) 更新全局 Uniform（相机、时间、雾、地球参数等会进入 uniformState）=====
  uniformState.update(frameState);

  // ===== 4) 阴影：若启用 shadowMap，把它放入 frameState，供后续绘制阶段生成/执行阴影相关命令 =====
  const shadowMap = scene.shadowMap;
  if (defined(shadowMap) && shadowMap.enabled) {
    if (!defined(scene.light) || scene.light instanceof SunLight) {
      // 太阳光：把 sunDirection 取反，得到“从太阳射向场景”的方向，供 shadow camera 使用
      Cartesian3.negate(
        uniformState.sunDirectionWC,
        scene._shadowMapCamera.direction,
      );
    } else {
      // 自定义方向光：直接用 light.direction
      Cartesian3.clone(scene.light.direction, scene._shadowMapCamera.direction);
    }
    frameState.shadowMaps.push(shadowMap);
  }

  // ===== 5) 每帧重置 compute / overlay 命令列表（由后续更新阶段重新填充）=====
  scene._computeCommandList.length = 0;
  scene._overlayCommandList.length = 0;

  // ===== 6) 配置 viewport 与 passState（与当前 drawing buffer 尺寸一致）=====
  const viewport = view.viewport;
  viewport.x = 0;
  viewport.y = 0;
  viewport.width = context.drawingBufferWidth;
  viewport.height = context.drawingBufferHeight;

  const passState = view.passState;
  passState.framebuffer = undefined;        // 从“默认 framebuffer”开始（后续可能切到 offscreen FBO）
  passState.blendingEnabled = undefined;
  passState.scissorTest = undefined;
  passState.viewport = BoundingRectangle.clone(viewport, passState.viewport);

  // ===== 7) 告诉渲染上下文：开始一帧（一些资源/状态在 beginFrame/endFrame 间管理）=====
  context.beginFrame();

  // 地球相关的帧生命周期钩子（如地形/影像的 per-frame 准备）
  if (defined(scene.globe)) {
    scene.globe.beginFrame(frameState);
  }

  // ===== 8) 更新环境（天空盒/大气/太阳/月亮等）并生成对应 draw/compute commands =====
  scene.updateEnvironment();

  // ===== 9) 核心渲染：更新 framebuffer、更新并渲染 primitives、生成并执行 draw commands =====
  // 内部会做：clear/绑定各种 FBO、update primitives、构建可见集(PVS)、compute、shadow、各 render passes 等
  scene.updateAndExecuteCommands(passState, backgroundColor);

  // ===== 10) 将 offscreen 的渲染结果“resolve/合成”到最终输出 =====
  // 例如：OIT 合成、后处理链、把 globeDepth 或 sceneFramebuffer copy 回原始 framebuffer
  scene.resolveFramebuffers(passState);

  // ===== 11) 绘制 overlay（叠加层命令）=====
  passState.framebuffer = undefined;
  executeOverlayCommands(scene, passState);

  // 地球帧结束钩子；并在瓦片没加载完时请求下一帧继续渲染（显式渲染模式下防止“停住不刷新”）
  if (defined(scene.globe)) {
    scene.globe.endFrame(frameState);

    if (!scene.globe.tilesLoaded) {
      scene._renderRequested = true;
    }
  }

  // ===== 12) 结束一帧 =====
  context.endFrame();
}
```



#### 绘制方法updateAndExecuteCommands

```js
Scene.prototype.updateAndExecuteCommands = function (
  passState,
  backgroundColor,
) {
  updateAndClearFramebuffers(this, passState, backgroundColor);

  if (this._environmentState.useWebVR) {
    executeWebVRCommands(this, passState, backgroundColor);
  } else if (
    this._frameState.mode !== SceneMode.SCENE2D ||
    this._mapMode2D === MapMode2D.ROTATE
  ) {
    executeCommandsInViewport(true, this, passState);
  } else {
    execute2DViewportCommands(this, passState);
  }
};
```





三维模式下执行的是executeCommandsInViewport(firstViewport, scene, passState)方法

在某个 viewport（视口）里，把本帧要画的东西整理好并真正执行 draw commands” 的入口

这段函数按顺序做了 5 件事

```js
1.（非首个 viewport）清空 commandList
-commandList 是本帧的“待执行绘制命令列表”容器之一。
-当一帧需要画多个 viewport 时，后续 viewport 要重新生成/填充命令，避免复用上一个 viewport 的命令集。


2.updateAndRenderPrimitives(scene)
-这是“把场景里各种 primitive（含 globe、3D Tiles、billboard、polyline…）更新到当前 frameState”的阶段。
-典型产物：primitive 的 update(frameState) 会把对应的 DrawCommand/ComputeCommand 等塞进 frameState.commandList（以及 scene 的 compute/overlay 列表）。


3.createPotentiallyVisibleSet(scene)
-这是“从 commandList 里筛出这一 viewport 真正可能可见/需要执行的命令，并按 pass/多 frustum 分桶”的阶段。
-会生成/更新 view.frustumCommandsList 等结构，供后面的 executeCommands 按 frustum、按 pass 去执行。


4.（仅首个 viewport）执行 compute 与 shadow map
-executeComputeCommands(scene)：执行 compute（GPU 计算）命令（与 viewport 通常无关，没必要重复做）。
-executeShadowMapCastCommands(scene)：渲染阴影贴图（shadow map）。阴影贴图一般也是“场景级资源”，同一帧多 viewport 共享，重复做浪费且可能引入状态问题。
-但如果 renderTranslucentDepthForPick 为 true（为拾取准备半透明深度等特殊路径），这里会跳过 shadow cast（属于性能/流程分支控制）。


5.executeCommands(scene, passState)
-真正按 pass（GLOBE / 3DTILES / OPAQUE / TRANSLUCENT / OVERLAY…）、按多 frustum（近远分层）去执行 draw commands 的阶段。
-你后面如果继续追，“一帧的绝大多数绘制细节”都在 executeCommands 及其调用的 performPass/performTranslucentPass/... 里。



// ...existing code...
/**
 * 将场景渲染到当前 viewport。
 * - firstViewport=true：表示本帧第一次渲染 viewport（例如 VR 左眼、或单视口场景）
 * - firstViewport=false：表示同一帧内的后续 viewport（例如 VR 右眼），会复用本帧部分全局结果
 *
 * 核心流程：
 * 1) 更新 primitives，生成/更新本帧 commandList
 * 2) 依据相机/裁剪体/遮挡等生成“潜在可见集”(PVS)并按 pass/frustum 组织
 * 3) 首个 viewport 执行 compute 与 shadowMap（通常是场景级资源，不应每个 viewport 重复生成）
 * 4) 执行 draw commands（真正的绘制提交）
 *
 * @private
 */
function executeCommandsInViewport(firstViewport, scene, passState) {
  const view = scene._view;
  const { renderTranslucentDepthForPick } = scene._environmentState;

  // 同一帧渲染多个 viewport 时：后续 viewport 需要重新生成 commandList，避免沿用前一个 viewport 的命令
  if (!firstViewport) {
    scene.frameState.commandList.length = 0;
  }

  // 更新/渲染各类 primitives（globe/tileset/primitives collections 等），填充 commandList、compute/overlay 列表等
  updateAndRenderPrimitives(scene);

  // 从 commandList 构建潜在可见集(PVS)：剔除不可见命令，并按 pass + 多 frustum 组织成可执行结构
  view.createPotentiallyVisibleSet(scene);

  // 首个 viewport：执行 compute 与 shadowMap（通常 viewport 无关/可共享，避免重复执行）
  if (firstViewport) {
    executeComputeCommands(scene);

    // 某些拾取相关路径需要半透明深度，可能会改变渲染流程；此处按标记决定是否跳过阴影投射
    if (!renderTranslucentDepthForPick) {
      executeShadowMapCastCommands(scene);
    }
  }

  // 执行绘制命令：按 frustum、按 pass 依次提交 WebGL draw calls（核心绘制阶段）
  executeCommands(scene, passState);
}
// ...existing code...
```



#### executeCommands



```
准备相机/working frustum →（非 pick）画环境 → 遍历每个 frustum：清深度/模板 → 画 Globe →（可选）globeDepth/depthPlane → 画 3D Tiles/分类 → 画 Opaque → 画 Translucent（含 OIT/排序）→（可选）深度拾取拷贝 →（可选）ID buffer（selected 后处理）。
```



```js
function executeCommands(scene, passState) {
  const { camera, context, frameState } = scene;
  const { uniformState } = context;

  // 1) 本帧相机相关的全局 uniform（视图矩阵/投影矩阵/相机位置方向等）
  uniformState.updateCamera(camera);

  // 2) 准备一个“工作 frustum”：后面每个分段 frustum 会不断改 near/far 来渲染（多 frustum 方案）
  const frustum = createWorkingFrustum(camera);
  frustum.near = camera.frustum.near;
  frustum.far = camera.frustum.far;

  const passes = frameState.passes;
  const picking = passes.pick || passes.pickVoxel;

  // 3) 环境（天空盒/大气/太阳/月亮等）：
  //    - 只在非拾取 pass 绘制（pick pass 不产生可拾取 ID 命令，环境通常也不需要 pick）
  if (!picking) {
    renderEnvironment(scene, passState);
  }

  // 4) 读取本帧环境状态开关（由 updateAndClearFramebuffers / updateEnvironment 决定）
  const {
    clearGlobeDepth,                 // 是否需要清 globe depth（不深度贴地等场景）
    renderTranslucentDepthForPick,   // 是否为拾取额外渲染半透明深度
    useDepthPlane,                  // 是否需要 depthPlane（防止背面被 pick 等）
    useGlobeDepthFramebuffer,        // 是否启用 globeDepth FBO
    useInvertClassification,         // 是否启用 invertClassification（分类反色）
    usePostProcessSelected,          // 是否需要“selected 后处理”相关的 ID buffer
  } = scene._environmentState;

  // 5) 多 frustum 命令列表：PVS 阶段已经把命令按 frustum+pass 分桶到这里
  const { globeDepth, globeTranslucencyFramebuffer, sceneFramebuffer, frustumCommandsList } = scene._view;
  const numFrustums = frustumCommandsList.length;

  // 一些常用 clear/depthPlane/globeTranslucency helper
  const globeTranslucencyState = scene._globeTranslucencyState;
  const clearDepth = scene._depthClearCommand;
  const clearStencil = scene._stencilClearCommand;
  const clearClassificationStencil = scene._classificationStencilClearCommand;
  const depthPlane = scene._depthPlane;

  // 6) 帮助函数：执行某个 pass 的命令数组（内部会逐条 executeCommand -> command.execute(WebGL)）
  function performPass(frustumCommands, passId) {
    uniformState.updatePass(passId); // 告诉 shader：当前处于哪个 pass（影响宏/分支/渲染状态）
    const commands = frustumCommands.commands[passId];
    const commandCount = frustumCommands.indices[passId];
    for (let j = 0; j < commandCount; ++j) {
      executeCommand(commands[j], scene, passState);
    }
    return commandCount;
  }

  function performIdPass(frustumCommands, passId) {
    uniformState.updatePass(passId);
    const commands = frustumCommands.commands[passId];
    const commandCount = frustumCommands.indices[passId];
    for (let j = 0; j < commandCount; ++j) {
      executeIdCommand(commands[j], scene, passState); // 专用于 picking/selected：绘制“ID/深度”等信息
    }
  }

  // 7) 核心循环：按“从远到近”遍历每个 frustum 分段，逐段清理并绘制各 pass
  for (let i = 0; i < numFrustums; ++i) {
    const index = numFrustums - i - 1;
    const frustumCommands = frustumCommandsList[index];

    // 7.1) 更新当前分段 frustum 的 near/far，并写入 uniform（2D 有特殊处理避免 z-fighting）
    if (scene.mode === SceneMode.SCENE2D) {
      // ...2D 特殊：移动相机/重标定 frustum depth，减少 z-fighting...
      uniformState.update(frameState);
      uniformState.updateFrustum(frustum);
    } else {
      // 3D：不透明 pass 里对相邻 frustum 的 near 做一点偏移，减少 frustum 交界撕裂
      frustum.near = index !== 0
        ? frustumCommands.near * scene.opaqueFrustumNearOffset
        : frustumCommands.near;
      frustum.far = frustumCommands.far;
      uniformState.updateFrustum(frustum);
    }

    // 7.2) 每个 frustum 开始前：清 depth（和 stencil，如果有）
    clearDepth.execute(context, passState);
    if (context.stencilBuffer) {
      clearStencil.execute(context, passState);
    }

    // 7.3) 先画地球（Globe）：如果开启 globeTranslucency 会走专门的 framebuffer/混合逻辑
    if (globeTranslucencyState.translucent) {
      uniformState.updatePass(Pass.GLOBE);
      globeTranslucencyState.executeGlobeCommands(
        frustumCommands, executeCommand, globeTranslucencyFramebuffer, scene, passState
      );
    } else {
      performPass(frustumCommands, Pass.GLOBE);
    }

    // 7.4) 如果使用 globeDepth FBO：把深度拷贝/同步出来，供后续分类/后处理/深度拾取等使用
    if (useGlobeDepthFramebuffer) {
      globeDepth.executeCopyDepth(context, passState);
    }

    // 7.5) Terrain classification（地形分类）等（某些拾取深度路径会跳过）
    if (!renderTranslucentDepthForPick) {
      // ...translucent globe 走专门路径，否则 performPass...
      performPass(frustumCommands, Pass.TERRAIN_CLASSIFICATION);
    }

    // 7.6) 若需要清 globeDepth，并可选绘制 depthPlane（用于不深度贴地时的拾取/遮挡修正）
    if (clearGlobeDepth) {
      clearDepth.execute(context, passState);
      if (useDepthPlane) {
        depthPlane.execute(context, passState);
      }
    }

    // 7.7) 3D Tiles 边缘可视化（edge framebuffer）相关 pass（先画，避免纹理反馈环）
    performCesium3DTileEdgesPass(scene, passState, frustumCommands);
    // ...把 edge framebuffer 的纹理塞进 uniformState 供后续 pass 使用...

    // 7.8) 3D Tiles + Classification：
    //     - 常规路径：先画 tiles，再画 classification（会改写 tiles 的颜色/蒙版等）
    //     - invertClassification 路径：会切换 FBO + stencil + fullscreen pass 做更复杂的合成
    let commandCount;
    if (!useInvertClassification || picking || renderTranslucentDepthForPick) {
      commandCount = performPass(frustumCommands, Pass.CESIUM_3D_TILE);
      if (commandCount > 0 && !renderTranslucentDepthForPick) {
        commandCount = performPass(frustumCommands, Pass.CESIUM_3D_TILE_CLASSIFICATION);
      }
    } else {
      // ...invertClassification：切 FBO、写 stencil、fullscreen 合成、再叠加 style...
      // ...（这是为了“未分类区域反色/高亮”的视觉效果）...
    }

    // 7.9) 其它主要几何 pass：体素、普通不透明、Gaussian splats
    performVoxelsPass(scene, passState, frustumCommands);
    performPass(frustumCommands, Pass.OPAQUE);
    performGaussianSplatPass(scene, passState, frustumCommands);

    // 7.10) 半透明 pass：为了避免多 frustum 混合伪影，半透明阶段通常不做 frustum overlap
    if (index !== 0 && scene.mode !== SceneMode.SCENE2D) {
      frustum.near = frustumCommands.near;
      uniformState.updateFrustum(frustum);
    }
    performTranslucentPass(scene, passState, frustumCommands);
    performTranslucent3DTilesClassification(scene, passState, frustumCommands);

    // 7.11) 深度拾取：把用于 pickPosition 的深度拷贝/更新出来（依赖 depthTexture/globeDepth）
    if (context.depthTexture && scene.useDepthPicking &&
        (useGlobeDepthFramebuffer || renderTranslucentDepthForPick)) {
      const pickDepth = scene._picking.getPickDepth(scene, index);
      pickDepth.update(context, globeDepth.depthStencilTexture);
      pickDepth.executeCopyDepth(context, passState);
    }

    // 7.12) selected 后处理（非 picking 才需要）：
    //     渲染一份“ID framebuffer”，后处理用它来对 selected 目标做描边/高亮等
    if (picking || !usePostProcessSelected) {
      continue;
    }

    const originalFramebuffer = passState.framebuffer;
    passState.framebuffer = sceneFramebuffer.getIdFramebuffer();

    // ...重置 frustum，绘制 GLOBE/3DTILES/OPAQUE/TRANSLUCENT 的 ID pass...
    performIdPass(frustumCommands, Pass.GLOBE);
    performIdPass(frustumCommands, Pass.CESIUM_3D_TILE);
    performIdPass(frustumCommands, Pass.OPAQUE);
    performIdPass(frustumCommands, Pass.TRANSLUCENT);

    passState.framebuffer = originalFramebuffer;
  }
}
```



绘制每个视椎体的绘制指令也是按照顺序的，优先级定义在Pass.js中

```js
/**
 * The render pass for a command.
 *
 * @private
 */
const Pass = {
  // If you add/modify/remove Pass constants, also change the automatic GLSL constants
  // that start with 'czm_pass'
  //
  // Commands are executed in order by pass up to the translucent pass.
  // Translucent geometry needs special handling (sorting/OIT). The compute pass
  // is executed first and the overlay pass is executed last. Both are not sorted
  // by frustum.
  ENVIRONMENT: 0,
  COMPUTE: 1,
  GLOBE: 2,
  TERRAIN_CLASSIFICATION: 3,
  CESIUM_3D_TILE_EDGES: 4,
  CESIUM_3D_TILE: 5,
  CESIUM_3D_TILE_CLASSIFICATION: 6,
  CESIUM_3D_TILE_CLASSIFICATION_IGNORE_SHOW: 7,
  OPAQUE: 8,
  TRANSLUCENT: 9,
  VOXELS: 10,
  GAUSSIAN_SPLATS: 11,
  OVERLAY: 12,
  NUMBER_OF_PASSES: 13,
};
export default Object.freeze(Pass);
```



#### 完整流程图

```
---
config:
  theme: forest
---
flowchart TD
  A["new Viewer：UI 组合层入口，内部会创建 CesiumWidget"] --> B["new CesiumWidget：渲染引擎核心入口；持有 Scene/Clock/Canvas 等"]

  B --> C["startRenderLoop：useDefaultRenderLoop=true 时启动默认渲染循环"]
  C --> D["requestAnimationFrame callback：浏览器每帧回调一次（可能被节流）"]

  D --> E["widget.resize：同步 canvas 像素尺寸/相机纵横比；必要时 requestRender"]
  D --> F["widget.render：驱动一帧：initializeFrame + tick + Scene.render"]

  F --> G["scene.initializeFrame：更新控制器/相机/tweens/缓存清理等“渲染前准备”"]
  F --> H["clock.tick：推进仿真时间，得到 currentTime"]
  F --> I["scene.render: time：计算 shouldRender；需要时才进入真正渲染"]

  I --> J["tryAndCatchError 分支：只保留会进入主渲染管线的那条路径"]
  I --> Z1["其它 Scene.render 细枝末节汇总：事件/预加载/调试/afterRender 等"]

  J --> K["internal render：Scene.render 内部真正的 render(scene) 过程"]
  K --> L["scene.updateAndExecuteCommands：清屏 + 更新primitive命令 + 组织并执行 draw commands"]
  K --> Z2["internal render 其它步骤汇总：updateFrameState/uniform/beginFrame/resolveFramebuffers 等"]

  L --> M["executeCommandsInViewport：在当前 viewport 内生成 PVS 并执行绘制"]
  L --> Z3["updateAndExecuteCommands 其它分支汇总：WebVR / 2D fixed viewport 等"]

  M --> N["updateAndRenderPrimitives：各 primitive.update 填充 commandList（Draw/Compute/Overlay 等）"]
  M --> O["createPotentiallyVisibleSet：剔除/分桶/排序 -> frustumCommandsList（按 frustum+pass）"]
  M --> P["executeCommands：按 Pass 顺序提交 WebGL 绘制（GLOBE/3DTILE/OPAQUE/TRANSLUCENT...）"]
  M --> Z4["viewport 其它步骤汇总：compute/shadow/pickDepth/selectedId 等辅助路径"]
```

<img src="./assets/LoRA vLLM RAG Pipeline-2026-02-10-020721.png" alt="LoRA vLLM RAG Pipeline-2026-02-10-020721" style="zoom: 25%;" />





### 离屏渲染

```
离屏渲染 顾名思义就是离开屏幕的渲染，就是说场景的渲染结果不直接显示到屏幕上，而是作为纹理使用 或者 需要经过一些加工后再显示到屏幕上。

帧缓冲区对象 可以用来代替颜色缓冲区或深度 缓冲区。绘制在帧缓冲区的对象并不会直接显示在canvas上，你可以先对帧缓冲区的内容进行一些处理再显示，或者直接用其中的内容作为纹理图像。

在帧缓冲区中进行绘制的过程又称为离屏绘制。
```

![image-20260228172904155](./assets/image-20260228172904155.png)

```
Cesium是基于WebGL封装的，所以Cesium的离屏渲染也就是WebGL的离屏渲染，即通过帧缓冲区对象来实现，webgl中通过gl.createFramebuffer()来创建帧缓冲区对象，Cesium的渲染模块将对原生webgl的帧缓冲区相关的操作封装到了FrameBuffer类中。
```

![image-20260228171931577](./assets/image-20260228171931577.png)



Cesium默认的绘制就是使用的离屏渲染，先将场景渲染到到合适的FrameBuffer中，然后交交给后处理程序，最后才会显示到屏幕上。



#### 自定义渲染











## 优化相关

### 暂停cesium的渲染循环

暂停cesium的渲染循环，节省GPU资源给视频解码用

今天优化的时候发现了一个事情，简单 HTML 页面里只有一个 `<video>`，GPU 100% 给视频解码用。而你的场景里 Cesium 的 WebGL 上下文在后台**每秒渲染 60 帧完整地球**，和视频解码抢 GPU 资源，所以 seek 时会有明显卡顿。

```
window.viewer.useDefaultRenderLoop = false;

useDefaultRenderLoop : boolean
Gets or sets whether or not this widget should control the render loop. If true the widget will use requestAnimationFrame to perform rendering and resizing of the widget, as well as drive the simulation clock. If set to false, you must manually call the resize, render methods as part of a custom render loop. If an error occurs during rendering, Scene's renderError event will be raised and this property will be set to false. It must be set back to true to continue rendering after the error.
useDefaultRenderLoop = false 会彻底停止 Cesium 内部的 requestAnimationFrame 循环（包括渲染和 onTick 回调），从而释放 GPU 资源。退出 VIDEO_1 时设回 true 即可恢复。

自己新建一个项目，测试这个属性开启与关闭的效果是怎样的，比如当开着渲染循环播放视频，和关上渲染循环播放视频
```

如果暂停cesium渲染循环的同时，还想利用cesium的仿真时间以及时钟驱动事件，可以在关闭 Cesium 渲染循环的同时，**手动调用 `viewer.clock.tick()`** 驱动时钟。这是 Cesium 官方支持的用法——`tick()` 只做时间推进和事件派发，不涉及 WebGL 渲染，开销极小。
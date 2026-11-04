# 改造指南 · 改哪里、别踩什么坑

`assets/shadow-raider.html` 是**打包后的单文件**（three.js、OrbitControls、后期库全部内联）。

**改参数不需要任何构建工具**：用任意编辑器打开这个 HTML，按下面的锚点搜索并修改，保存后刷新
浏览器即可。库代码在文件开头几段压缩过的 `<script>` 里，业务代码在最后一段，搜索下面的
锚点都能直接定位到。

改动量大时（换外形、重做贴图、加新零件）才需要先拆成多文件工程，最后按第 6 节重新内联。

## 1. 改重量 / 成本 / 文案

```js
var MASS = { head:6.2, body:10.4, ... };   // 逐件质量 g
var COST = { head:30,  body:60,   ... };   // 逐件成本 元
```

HUD 的"起飞重量 / 整机成本"由这两张表求和得到，**改表即同步**，不要另改文字。

零件资料卡内容在 `var PARTS = { ... }`（每项 `n` 名称、`t` 功能、`r` 规格列表、`w` 工艺要点）。

## 2. 改外形

| 部位 | 锚点 |
|---|---|
| 头部（脑颅/吻/耳） | 搜 `头部外形：宽扁脑颅` 区块 |
| 机身舱体 | 搜 `var belly = ` |
| 机翼膜 | 搜 `function buildWing(side)` |
| 尾梁 / 尾膜 | 搜 `var tailBoom = ` |
| 曲柄连杆 | 搜 `ROD_PLANAR` |

**翼膜的坑**：膜根必须跟着前缘肋的曲线上抬，用直线肋会穿模。前缘肋是 `tubePath` 弯管，
膜根 `x` 不要小于腹部整流罩最宽处——否则膜根会插进机身。

## 3. 改扑动参数

```js
w.group.rotation.x = qWing - 0.0873;   // −0.0873 rad = +5° 安装角
```

- 安装角：改 `0.0873`（弧度），前缘上抬为正迎角
- 扑频：主循环 `Math.sin(aero.t * Math.PI * 2 * 9.3)` 里的 `9.3` 和流场里同名常量要一起改
- 摆角：由 `solveHorn()` 两圆求交算出，改 `ROD_PLANAR`（连杆投影长）或 `HORN_R`（摇臂半径）会改变摆幅

## 4. 改气动流场

全部在 `function aeroField(x, y, z, out)` 里，按注释分 5 段：

| 段 | 调什么 |
|---|---|
| 机体排开 | `push = U * q * 2.6 * ...`，系数越大绕得越开 |
| 前缘上洗 | `U * 0.16` |
| 展向下洗 | `U * 0.40`、`yFade`、`spanW` 指数 `0.55` |
| 近场环流 | `U * 0.46`、`tanh((y+4)/58)` |
| 翼尖涡 | `gAmp = 82000 * lfV`、`AERO.CORE`（涡核半径） |

流线采样在 `var LXS` / `var LYS`（9 列 × 6 层 = 54 条），线束偏移在 `var LOY`。

**强度参考**：中心列偏折超过 ±35° 就会把流线甩出画面、看起来像线团。翼尖涡的
作用半径控制在 210 mm 以内比较干净。

## 5. 改配色

材质表在 `var M = { ... }`（`carbon` / `printD` / `membrane` / `steel` 等）。
贴图是程序化 Canvas 生成的（碳纤维斜纹、TPU 织物、SLS 颗粒、商品标签），搜
`function texCarbon` 一类函数。

## 6. 重新打包（关键顺序）

把 `three.min.js`、`OrbitControls.js` 和 10 个后期模块内联进 HTML。必须注意：

```
CopyShader → LuminosityHighPassShader → EffectComposer → RenderPass → ShaderPass
→ MaskPass → SSAOShader → SimplexNoise → SSAOPass → UnrealBloomPass
```

`EffectComposer.js` 定义了 `THREE.Pass`，**必须先于** `MaskPass` / `ShaderPass` 加载，
否则报 `THREE.Pass is not a constructor`。

## 7. 踩过的坑（按血泪程度排序）

1. **法线贴图绿通道**：用 `-dy` 会让整片翼膜发黑。OpenGL 约定是 `+dy`
2. **后期整体发灰**：不要给中间 RenderTarget 设 sRGB，改为在链路末端加一个 `lin2srgb` ShaderPass
3. **流线颜色发白**：顶点色要按 `pow(c, 2.2)` 转线性，否则被 ACES 色调映射冲成灰白
4. **线宽改不了**：`LineBasicMaterial.linewidth` 在绝大多数平台恒为 1 px，只能靠多条偏移线叠加
5. **标签文字被拉伸**：贴图宽高比必须等于贴片宽高比（如 4.64:1），否则字会变形
6. **悬空件**：接收机排针、电池、翼膜夹条螺丝都要落在实体上，检查时逐件看包围盒接触
7. **无头环境 rAF 只有 0.3 fps**：面板数字按帧计数不会刷新，要用 0.2 s 时间节流；
   截图前手动循环调 `updateAero(0.03)` 推进到稳态

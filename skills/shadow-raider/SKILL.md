---
name: shadow-raider
description: 交付可直接双击观看的三维仿生蝙蝠扑翼机模型（Shadow Raider）——自包含单文件 three.js HTML，17 个可点击零件、爆炸/透视/骨架视图、三轴飞行控制演示、准稳态空气动力流场可视化。当用户需要查看、分享、嵌入或改造三维扑翼机 / 仿生飞行器模型，或需要一个不用安装任何软件就能打开的三维评审物时使用。Triggers 三维仿生蝙蝠、扑翼机模型、可旋转 3D 模型、bat ornithopter、flapping wing MAV visualization。
agent_created: true
---

# Shadow Raider

## Overview

Shadow Raider 是一架按 1:1 实机尺寸建模的仿生蝙蝠扑翼机（翼展 840 mm、起飞重量 139.8 g）。
本 skill 把它的三维交互模型打包成一个**自包含 HTML**：three.js、OrbitControls 和后期处理库
全部内联，不依赖网络、不需要安装任何软件，双击即可在浏览器里旋转、拆解、飞。

它同时是一份能经得起推敲的工程文件——每个零件的质量、成本、材料工艺都可查，
气动数据与结构计算结果一致，不是"看着像"的模型。

## 何时使用

- 用户要看一个"能自己转、能点零件、不用装软件"的三维模型
- 需要嵌入网页、生成答辩截图、做产品介绍物料
- 需要改动翼展/配色/扑频/流场强度后重新交付
- 需要核对该模型的工程数据是否自洽

## 快速开始

```
复制 assets/shadow-raider.html 到目标目录 → 双击打开
```

不需要服务器。用 `file://` 打开即可，控制台应当零报错。

交付时**永远复制，不要原地改**；保留 `assets/` 里的原始件。

## 页面能做什么

| 操作 | 结果 |
|---|---|
| 左键拖拽 / 滚轮 / 右键 | 旋转 / 缩放 / 平移 |
| 单击任意零件 | 弹出参数卡：规格、质量、成本、工艺要点 |
| 左侧零件目录 | 17 个零件直达 |
| 扑翼动画 | 连续扑动，四杆机构真实反解 |
| 爆炸视图 | 零件沿装配方向散开 |
| 实体 / 半透明 / 骨架 | 三种渲染模式 |
| 等轴测 / 侧 / 正 / 俯 | 四个标准视角 |
| 空气动力 | 进入流场：粒子、流线、翼尖涡、升力/下洗/涡强面板，风速 2–6 m/s 可调 |
| 飞行控制演示 | 爬升 / 下降 / 左转 / 右转 / 左倾 / 右倾 / 回中，气流随之变化 |

气动模式里流线颜色与图例对应：**蓝 = 正常气流，橙 = 下洗，绿 = 上洗**。

## 交付流程

1. 读 `references/model-spec.md` 取准确数据（翼展、重量、成本、翼载、零件表）
2. 复制 `assets/shadow-raider.html` 到目标路径
3. 按下面的清单验证
4. 汇报：文件路径、体积、验证结果、可用功能

改参数见 `references/customization.md`，里面有锚点定位和七个已踩过的坑
（法线贴图绿通道、后期 sRGB、线宽、标签拉伸等）。

## 验证清单

用无头浏览器（`--headless=new --use-angle=swiftshader`）注入检查：

1. `Runtime.exceptionThrown` 为空
2. `document.querySelectorAll('canvas').length >= 1`
3. 依次点 `#b-flap` `#b-explode` `#b-solid` `#b-xray` `#b-wire` `#b-reset`
   `#b-side` `#b-front` `#b-top` `#b-aero`，无报错
4. 进气动模式后 `aero.lGeo.attributes.position.count > 0`
5. 点 `#ctrl button[data-cmd]` 六个方向键，`ctl` 值随之改变
6. 截图留档

## 硬性约束

- **零外部依赖**：不要加 CDN、不要拆成多文件，单文件双击即用是它的全部价值
- **不臆造数据**：任何数字都要能在 `references/model-spec.md` 或 `PARTS` 表里找到出处
- **不改原件**：交付一律复制
- 中文汇报，结论先行

## Resources

### assets/
- `shadow-raider.html` — 可直接交付的三维模型（约 800 KB，全部库内联）

### references/
- `model-spec.md` — 整机参数、17 件零件清单、传动链、四杆几何、气动校核、飞控
- `customization.md` — 改哪里、打包顺序、七个常见坑

### agents/
- `shadow-raider.md` — 交付/校验用的子代理定义

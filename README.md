# 拼图切割线生成器

https://draradech.github.io/jigsaw/index.html

一款纯客户端 Web 应用，可生成拼图切割线图案并导出为 SVG 文件，适用于激光切割或打印。

---

## 仓库结构

| 文件 | 说明 |
|------|------|
| `index.html` | 首页，链接到两个生成器 |
| `jigsaw.html` | 矩形拼图生成器 |
| `jigsaw-hex.html` | 六边形 / 圆形拼图生成器 |

---

## 功能说明

### 矩形生成器（`jigsaw.html`）

生成由矩形格子排列的互锁拼图片。

**参数说明**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| Seed（随机种子） | 随机 | 确定性控制所有凸榫形状的随机种子 |
| Tab Size（凸榫大小） | 20% | 每条边上凸榫的相对尺寸 |
| Jitter（抖动量） | 4% | 施加在控制点上的随机偏移量 |
| Corner Radius（圆角半径） | 2 mm | 外边框圆角半径 |
| Tiles（拼块数量） | 15 × 10 | 水平与垂直方向上的拼块数 |
| Size（尺寸） | 300 × 200 mm | 成品拼图的物理尺寸 |

**实现原理**

- 每条内部边由九个控制点（`p0`–`p9`）的三次贝塞尔曲线绘制：曲线从拼块角点（`p0`）出发，经过 `p3`–`p6` 处形成经典的拼图凸榫凸起，到达另一角点（`p9`）。
- 基于正弦函数的伪随机数生成器（`random()`）以种子值为输入，保证相同种子始终生成相同图案。
- 水平线（`gen_dh`）与垂直线（`gen_dv`）分开生成，分别用深蓝色和深红色描边，便于区分。
- 外边框（`gen_db`）为带圆角的矩形。
- 页面内嵌实时 SVG 预览；点击 **Download SVG** 将按物理毫米尺寸导出 SVG 文件。

---

### 六边形 / 圆形生成器（`jigsaw-hex.html`）

生成排列在六边形网格上、适合圆形或六边形外框的互锁拼图片。

**参数说明**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| Seed（随机种子） | 随机 | 随机种子 |
| Tab Size（凸榫大小） | 27% | 凸榫的相对尺寸 |
| Jitter（抖动量） | 5% | 控制点的随机偏移量 |
| Diameter（直径） | 240 mm | 成品拼图的直径 |
| Rings（环数） | 6 | 同心六边形环的层数 |
| Circle Warp（圆形扭曲） | 关 | 将六边形网格扭曲以适配圆形轮廓 |
| Truncate Edge Pieces（截断边缘块） | 关 | 将边缘凸榫替换为直线或圆弧 |

**实现原理**

- 拼块按轴坐标排列在六边形网格上。
- 每条内部边与矩形生成器相同，使用九个控制点的三次贝塞尔曲线绘制凸榫。
- 控制点在局部边坐标系中计算，再经由 `scale（缩放）→ rotate（旋转）→ warp（扭曲）→ translate（平移）` 管线变换到屏幕坐标。
- 启用 *Circle Warp* 后，每个点会被投影到内切圆的单位六边形上，使所有外边缘落在真正的圆弧上。
- `gen_dh` 绘制斜向边，`gen_dv` 绘制垂直边，`gen_db` 绘制外边框（六边形或圆形）。

---

## 输出格式

两种生成器均导出包含三条路径的 SVG 文件（`jigsaw.svg`）：

- **深蓝色（DarkBlue）** — 水平 / 斜向内部切割线
- **深红色（DarkRed）** — 垂直内部切割线
- **黑色（Black）** — 外边框

所有路径均使用 `fill="none"` 和细描边（`0.1–0.2 mm`），可直接用于激光切割机或刻字机。

---

## Swift 版本可行性评估

### 1. 概述

当前项目是零依赖的纯浏览器应用，全部逻辑用 JavaScript 实现。将其迁移到 Swift 主要有两个方向：**命令行工具（Swift CLI）** 和 **macOS / iOS 原生应用（SwiftUI）**。

### 2. 核心算法迁移难度

| 模块 | 当前实现 | Swift 对应方案 | 迁移难度 |
|------|----------|----------------|----------|
| 伪随机数生成器（正弦函数） | `Math.sin` | `Foundation.sin` | ✅ 低 |
| 三次贝塞尔曲线路径生成 | 字符串拼接 SVG `d` 属性 | 同样字符串拼接，或 `CGPath` | ✅ 低 |
| 六边形网格坐标变换（缩放/旋转/扭曲/平移） | 普通 JS 函数 | `CGAffineTransform` 或纯数学函数 | ✅ 低 |
| SVG 文件导出 | `Blob` + `URL.createObjectURL` | `FileManager` 写入文件 / `NSSavePanel` | ✅ 低 |

结论：**所有算法逻辑均可直接移植到 Swift**，数学函数、字符串操作与文件写入在 Swift / Foundation 中均有完整支持。

### 3. 两种 Swift 方案对比

#### 方案 A：Swift 命令行工具（跨平台）

- 使用 Swift Package Manager 构建，可在 macOS 和 Linux 上运行。
- 通过命令行参数接受种子、尺寸、拼块数等输入，直接输出 SVG 文件。
- **优点**：实现最简单，可集成到自动化流程、CI/CD 或批量生成场景。
- **缺点**：无实时预览，无图形界面，不适合普通用户。

```bash
# 示例用法
jigsaw-cli --seed 42 --tiles 15x10 --size 300x200 --output puzzle.svg
```

#### 方案 B：SwiftUI macOS / iOS 应用

- 使用 SwiftUI 构建带滑动条、输入框的参数面板，实时渲染预览（`Canvas` API 或 WebKit 加载 SVG）。
- 导出功能通过 `NSSavePanel`（macOS）或 `UIDocumentPickerViewController`（iOS）实现。
- **优点**：原生体验好，支持 Apple Silicon，可上架 App Store，适合普通用户。
- **缺点**：仅限 Apple 平台，开发工作量约为当前 Web 版本的 3–5 倍；而 Web 版本无需安装、跨平台、零成本部署。

### 4. 主要风险与注意事项

| 风险 | 说明 |
|------|------|
| 平台限制 | Swift 原生应用只能在 Apple 平台运行，失去现有 Web 版本"打开即用"的优势 |
| 开发与维护成本 | SwiftUI 应用需要 Xcode 环境和 Apple Developer 账号，维护成本高于单文件 HTML |
| 实时预览 | Web 版 SVG 预览直接内嵌浏览器，SwiftUI 中需额外使用 `WebKit` 或 `Canvas` |
| 跨平台需求 | 若有 Windows / Linux 用户，Swift 方案无法覆盖；Web 方案可以 |

### 5. 建议

- 若目标是**批量生成 SVG 文件**或集成到工具链，推荐 **Swift CLI 方案**，迁移成本低，一周内可完成。
- 若目标是为 Mac / iPhone 用户提供**精美的原生应用**体验，推荐 **SwiftUI 方案**，需约 2–4 周开发。
- 若无特定的 Apple 生态需求，**保留现有 Web 版本**仍是成本最低、覆盖面最广的选择。

---

## 许可证

保存功能：可能属于 **cc-by-sa**  
（保存功能基于 StackOverflow 上多个回答拼合而成：
https://stackoverflow.com/questions/19327749/javascript-blob-filename-without-link，
因此该部分遵循 cc-by-sa — 除非原回答者没有相应权利）

其余所有内容：**cc0**（https://creativecommons.org/publicdomain/zero/1.0/）

如需署名，链接到本仓库即可，但并非强制要求。

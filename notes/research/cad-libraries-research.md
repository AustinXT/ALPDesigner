# 开源 CAD 库调研报告

> 调研日期: 2026-03-15

## 一、参数化 CAD 生成库（从 JSON/参数生成 3D 模型）

### 1. JSCAD (OpenJSCAD)
- **GitHub**: https://github.com/jscad/OpenJSCAD.org
- **Stars**: ~2.5k
- **最后更新**: 活跃维护中, V3 开发中
- **许可证**: MIT
- **语言**: JavaScript
- **文档质量**: 好 - 有官方文档站 (openjscad.xyz/docs), wiki, 用户指南
- **关键特性**:
  - 纯 JavaScript 参数化 2D/3D 建模
  - 浏览器 + CLI 双模式运行
  - 支持 CSG 布尔运算（并集、差集、交集）
  - 模块化架构，可按需引入
  - 导出 STL, DXF, SVG, AMF, X3D 等格式
  - 支持参数化定义（滑块控制参数）
- **Next.js/React 集成**: 可通过 npm 引入核心库，需自行搭建渲染层
- **铝型材适用性**: ★★★★ 非常适合定义参数化截面和拉伸体

### 2. jscad-fiber (React + JSCAD)
- **GitHub**: https://github.com/tscircuit/jscad-fiber
- **Stars**: 较小项目
- **最后更新**: 4 个月前 (v0.0.85)
- **许可证**: MIT
- **语言**: TypeScript/React
- **文档质量**: 中等 - 有示例目录
- **关键特性**:
  - 用 React 组件方式创建 JSCAD 3D 模型
  - 声明式 API: `<Cube>`, `<Sphere>`, `<Subtract>` 等
  - 布尔运算支持
  - 与 React 生态无缝集成
- **Next.js/React 集成**: ★★★★★ 原生 React 组件
- **铝型材适用性**: ★★★ 可用于声明式定义型材组件

### 3. replicad
- **GitHub**: https://github.com/sgenoud/replicad
- **Stars**: ~570
- **最后更新**: 活跃维护中 (npm 5天前更新, v0.21.0)
- **许可证**: MIT
- **语言**: TypeScript
- **文档质量**: 优秀 - 有完整的 API 文档站 (replicad.xyz)，教程和 Studio 功能
- **关键特性**:
  - 基于 OpenCascade (WASM) 的浏览器端 CAD 库
  - 受 CadQuery 和 CascadeStudio 启发的 API
  - 支持草图绘制、成型、修改的流式 API
  - 可作为独立库集成，构建自己的查看器/编辑器/配置器
  - 支持参数化模型 (Studio features)
  - 导出 STEP, STL 等格式
  - 提供 three.js helper (replicad-threejs-helper)
- **Next.js/React 集成**: ★★★★ 有 Next.js 集成讨论，提供 app-example
- **铝型材适用性**: ★★★★★ 非常适合 - B-Rep 内核，精确几何，参数化支持

### 4. OpenCascade.js
- **GitHub**: https://github.com/donalffons/opencascade.js
- **Stars**: ~2.8k
- **最后更新**: 维护状态有疑虑 (Issue #273 讨论维护状态)
- **许可证**: LGPL-2.1 (跟随 OCCT)
- **语言**: C++ -> JavaScript/WASM (Emscripten)
- **文档质量**: 好 - 有官方站 (ocjs.org), 示例
- **关键特性**:
  - OpenCascade 完整 CAD 内核的 WASM 移植
  - 近乎原生速度执行
  - 支持多线程
  - 自定义构建可裁剪库大小
  - 全部 OCCT API 可用（B-Rep, STEP, IGES 等）
- **Next.js/React 集成**: ★★★ 需要 webpack fallback 配置
- **铝型材适用性**: ★★★★★ 工业级 CAD 内核，完美支持参数化型材

### 5. Manifold (manifold-3d)
- **GitHub**: https://github.com/elalish/manifold
- **Stars**: ~1.9k
- **最后更新**: 非常活跃 (npm 最新 v3.4.0)
- **许可证**: Apache-2.0
- **语言**: C++ -> JavaScript/WASM
- **文档质量**: 好 - Wiki, API 文档
- **关键特性**:
  - 拓扑鲁棒的几何库
  - 保证输出流形网格的布尔运算（业内首创）
  - 极快的 mesh 布尔运算
  - OpenSCAD 风格的构造 API
  - SDF（有符号距离函数）支持
  - 顶点属性和 ID 追踪
  - 支持材质/纹理
- **Next.js/React 集成**: ★★★ WASM 模块，可在 Node.js 和浏览器中使用
- **铝型材适用性**: ★★★★ 适合快速布尔运算和网格生成

### 6. CadQuery (Python)
- **GitHub**: https://github.com/CadQuery/cadquery
- **Stars**: ~5k+
- **最后更新**: 活跃维护
- **许可证**: Apache-2.0
- **语言**: Python (基于 OCCT)
- **文档质量**: 优秀 - ReadTheDocs, 视频教程
- **关键特性**:
  - Python 参数化 CAD 脚本框架
  - 直觉式 API，易学易用
  - 基于 OpenCascade 内核
  - 输出 STEP, AMF, 3MF, STL
  - 可通过 CadQuery-server 提供 Web API
  - VSCode 扩展
- **Next.js/React 集成**: ★★ 需要后端 Python 服务
- **铝型材适用性**: ★★★★★ Python 端最佳选择，可作为后端生成引擎

---

## 二、Web 端 3D CAD 编辑器

### 1. Chili3D ★ 推荐
- **GitHub**: https://github.com/xiangechen/chili3d
- **Stars**: ~3.9k
- **最后更新**: 2025-09 (活跃开发)
- **许可证**: AGPL-3.0
- **语言**: TypeScript
- **文档质量**: 中等 - README, 在线演示
- **关键特性**:
  - 完整的浏览器端 3D CAD 应用
  - OpenCascade 编译为 WASM，近乎原生性能
  - Three.js 渲染
  - 建模工具：基础形状、2D 草图、布尔运算、拉伸、旋转
  - 多语言支持 (中文/英文)
  - 导入/导出 STEP, IGES, BREP
- **Next.js/React 集成**: ★★ 独立应用，需要较多改造
- **铝型材适用性**: ★★★★ 可作为编辑器参考或直接定制

### 2. CADmium
- **GitHub**: https://github.com/CADmium-Co/CADmium
- **Stars**: ~1.4k
- **最后更新**: 活跃开发中（早期原型）
- **许可证**: Elastic License 2.0 (非传统开源)
- **语言**: Rust (WASM) + SvelteKit + Three.js
- **文档质量**: 中等 - 博文, README
- **关键特性**:
  - Local-first 浏览器端参数化 CAD
  - Rust 核心引擎 (Truck b-rep 内核)
  - Three.js + Threlte 声明式场景管理
  - 导出 STEP, OBJ, .cadmium (JSON)
  - Tauri 原生构建支持
- **Next.js/React 集成**: ★ SvelteKit 架构，不直接兼容
- **铝型材适用性**: ★★★ 有潜力但仍处早期阶段

### 3. Three.cad
- **GitHub**: https://github.com/twpride/three.cad
- **Stars**: ~329
- **最后更新**: 活跃
- **许可证**: MIT
- **语言**: TypeScript (React + Three.js + WASM)
- **文档质量**: 基础 - 用户指南, 演示
- **关键特性**:
  - Three.js + React + WASM 构建
  - 参数化草图约束
  - CSG 布尔运算
  - 可在 3D 空间任意 2D 平面上绘制
  - 线条和弧线工具
  - STL 导出
- **Next.js/React 集成**: ★★★★ 原生 React 项目
- **铝型材适用性**: ★★★ 基础参数化功能

### 4. JSketcher
- **GitHub**: https://github.com/xibyte/jsketcher
- **Stars**: ~1.7k
- **最后更新**: 2025-05
- **许可证**: 需确认
- **语言**: JavaScript
- **文档质量**: 中等 - Wiki, 视频
- **关键特性**:
  - 纯 JavaScript 参数化 2D/3D 建模
  - 2D 约束求解器
  - 特征/历史记录建模方法
  - OpenCascade 实体建模
  - 完整的几何选择和交互工具
- **Next.js/React 集成**: ★★ 需要改造
- **铝型材适用性**: ★★★★ 功能全面的参数化建模

### 5. CascadeStudio
- **GitHub**: https://github.com/zalo/CascadeStudio
- **Stars**: ~1.3k
- **最后更新**: 有一定活跃度
- **许可证**: MIT
- **语言**: JavaScript
- **文档质量**: 中等 - 内置帮助, 社区手册
- **关键特性**:
  - 浏览器端完整 CAD IDE
  - 实时脚本编辑 + 3D 预览
  - OpenCascade 内核（WASM）
  - 参数化滑块控制
  - CSG + 高级操作（倒角、扫掠等）
- **Next.js/React 集成**: ★★ 独立应用
- **铝型材适用性**: ★★★ 好的原型工具

### 6. Bitbybit.dev ★ 推荐
- **GitHub**: https://github.com/bitbybit-dev/bitbybit
- **Stars**: 需确认
- **最后更新**: 非常活跃 (v1.0.0 RC)
- **许可证**: MIT (核心算法)
- **语言**: TypeScript
- **文档质量**: 优秀 - 完整学习站 (learn.bitbybit.dev)
- **关键特性**:
  - 3D CAD 算法平台
  - 同时支持 Three.js, BabylonJS, PlayCanvas
  - 集成 OCCT, Manifold, JSCAD 多内核
  - Node.js 后端也可运行
  - CLI 脚手架工具 (create-bitbybit-app)
  - 完整的 npm 包生态
- **Next.js/React 集成**: ★★★★★ 原生支持 Three.js + React 集成
- **铝型材适用性**: ★★★★★ 最灵活的选择，多内核支持

---

## 三、渲染和查看器层

### 1. react-three-fiber (R3F)
- **GitHub**: https://github.com/pmndrs/react-three-fiber
- **Stars**: ~27k+
- **最后更新**: 非常活跃
- **许可证**: MIT
- **关键特性**:
  - Three.js 的 React 渲染器
  - 声明式 3D 场景管理
  - 完整的 React 生态集成
  - @react-three/drei 辅助组件库
  - 已有 CAD 应用使用案例 (buerli.io 等)
- **Next.js 集成**: ★★★★★ 完美支持
- **铝型材适用性**: ★★★★★ 最佳渲染层选择

### 2. three-cad-viewer
- **GitHub**: https://github.com/bernhard-42/three-cad-viewer
- **Stars**: 较小
- **最后更新**: 活跃
- **许可证**: 需确认
- **关键特性**:
  - 基于 Three.js 的 CAD 查看器组件
  - 支持网格和边线可视化
  - 为 CadQuery/Jupyter 设计
  - 可独立使用
- **Next.js 集成**: ★★★★ 纯 JavaScript 组件

### 3. Online 3D Viewer
- **npm**: online-3d-viewer
- **网站**: https://3dviewer.net
- **关键特性**:
  - 支持多种 3D 格式
  - STL, OBJ, GLTF, 3DS, FBX 等
  - 开源免费
- **Next.js 集成**: ★★★★ npm 包直接使用

---

## 四、结构分析 / 连接检测 / BOM 生成

### 结构分析

大多数结构分析工具都是桌面端 Python/C++ 程序，JavaScript 端选择有限：

1. **Frame3DD** - 2D/3D 框架静态和动态结构分析（C 语言，非 JS）
2. **OpenSees** - 结构分析框架（Python/C++）
3. **CalculiX** - 有限元分析（C/Fortran）

### JavaScript 端可选方案

- 目前没有成熟的 JavaScript 结构分析库
- 可考虑将简单的梁计算逻辑自行实现
- 铝型材结构通常使用查表法（型材承载参数数据库）

### BOM 生成

1. **@arkham-engineering/bom** (npm)
   - 程序化创建物料清单
   - 添加/删除/修改 BOM 项目
   - 可关联外部数据集
   - 适用于基础 BOM 功能

### 连接检测

- 目前没有现成的 JavaScript 连接点检测库
- 需要自行实现基于几何距离的连接检测算法
- 可基于 Manifold 或 JSCAD 的碰撞检测扩展

---

## 五、综合推荐方案

### 方案 A：纯 JavaScript/TypeScript 方案（推荐）

```
渲染层:     react-three-fiber (@react-three/fiber + @react-three/drei)
CAD 内核:   replicad (OpenCascade WASM) 或 bitbybit-dev
参数化定义:  JSON Schema -> replicad API
导出:       STEP, STL (via replicad/OCCT)
编辑器:     自建 (基于 R3F)
BOM:        自建 (JSON 数据结构)
```

**优点**: 全栈 TypeScript, Next.js 完美集成, 工业级精度
**缺点**: 需要较多自研编辑器部分

### 方案 B：混合方案

```
渲染层:     react-three-fiber
CAD 内核:   bitbybit-dev (集成 OCCT + Manifold + JSCAD)
参数化定义:  JSON -> bitbybit API
导出:       STEP, STL, OBJ
编辑器:     参考 Chili3D 或 Three.cad
BOM:        自建
```

**优点**: 最灵活，多内核支持
**缺点**: bitbybit 学习曲线

### 方案 C：Python 后端 + JS 前端

```
前端渲染:   react-three-fiber
后端 CAD:   CadQuery (Python)
API:       REST/WebSocket
参数化定义: JSON -> CadQuery
导出:      STEP, STL, AMF
编辑器:    自建 (前端)
BOM:       Python 端生成
```

**优点**: CadQuery 生态成熟, 文档优秀
**缺点**: 需要维护两个技术栈

---

## 六、铝型材设计特定需求映射

| 需求 | 推荐库 | 说明 |
|------|--------|------|
| 型材截面参数化 | replicad / CadQuery | 2D 草图 -> 拉伸 |
| 型材 3D 渲染 | react-three-fiber | 浏览器 3D 渲染 |
| 布尔运算（开槽等） | Manifold / OpenCascade | 快速精确 |
| 连接件库 | 自建 JSON 数据库 | 参数化定义 |
| 板材 | replicad / JSCAD | 基础形状 |
| 组装约束 | 自建 | 基于型材槽口对齐 |
| BOM 导出 | 自建 | 遍历场景树生成 |
| 效果图 | R3F + postprocessing | 材质、光照 |
| 加工参数导出 | 自建 | 切割长度、钻孔位置 |
| STEP/DXF 导出 | OpenCascade.js / replicad | 工业标准格式 |

# 技术调研 - CAD 库与框架选型

> 调研日期：2026-03-15

## 推荐方案

### 渲染层
- **react-three-fiber** (~27k stars, MIT) — React 的 Three.js 渲染器，Next.js 兼容性极佳
- **@react-three/drei** — 提供轨道控制、变换工具、选择工具等

### CAD 内核
- **replicad** (~570 stars, MIT) — 基于 OpenCascade WASM 的 TypeScript API
  - 文档：https://replicad.xyz
  - GitHub：https://github.com/sgenoud/replicad
  - 优势：纯浏览器运行、API 清晰、提供 three.js 辅助包
  - 注意：Next.js 集成需要配置 WASM 加载

### 备选方案
| 库 | Stars | 许可证 | 备注 |
|---|---|---|---|
| JSCAD | ~2.5k | MIT | V3 开发中，API 成熟 |
| OpenCascade.js | ~2.8k | LGPL-2.1 | 底层库，replicad 的基础 |
| Manifold | ~1.9k | Apache-2.0 | 高性能布尔运算 |
| bitbybit-dev | 活跃 | MIT (核心) | 多内核，功能全面但较重 |
| Chili3D | ~3.9k | AGPL-3.0 | 完整 CAD 应用，许可证限制 |

## 结构分析
- JS 生态中无成熟结构分析库
- 建议：基于型材规格查表（承载力/最大跨距）实现简化验证
- 连接点检测：自定义几何距离算法

## 推荐架构

```
渲染层:    react-three-fiber + @react-three/drei
CAD 内核:  replicad (OpenCascade WASM, MIT)
参数化:    JSON Schema → replicad API
导出:      STEP/STL via replicad; 图片 via R3F canvas capture
BOM:       自定义（遍历场景树，输出 JSON/CSV/PDF）
型材库:    自定义 JSON 数据库（2020/3030/4040 系列参数）
连接件库:  自定义 JSON 数据库（角码、T 型螺母等）
组装:      自定义约束求解器（槽位对齐、卡扣点）
```

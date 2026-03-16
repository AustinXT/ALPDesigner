# 铝型材设计师 — 系统架构规格书

<meta>
  <document-id>alpdesigner-sys-spec</document-id>
  <version>1.0.0</version>
  <project>ALPDesigner</project>
  <type>System Architecture Specification</type>
  <created>2026-03-16</created>
  <depends>real.md, cog.md, pr.spec.md, userstory.spec.md</depends>
</meta>

## 1. 架构拓扑

```
┌─────────────────────────────────────────────────────────────────┐
│  Browser                                                        │
│  ┌──────────────┐  postMessage  ┌──────────────────────────┐   │
│  │  Main Thread  │◄────────────►│  Web Worker              │   │
│  │  Next.js SPA  │              │  replicad (OCCT WASM)    │   │
│  │  R3F 3D视口   │              │  CadEngine 抽象层         │   │
│  │  Zustand 状态  │              │  结构安全检查器            │   │
│  └──────┬───────┘              └──────────────────────────┘   │
│         │                                                       │
│  ┌──────┴───────┐  localStorage / IndexedDB（匿名模式）         │
│  │ 本地存储层     │                                              │
│  └──────────────┘                                               │
└────────┬────────────────────────────────────────────────────────┘
         │ HTTPS
┌────────┼────────────────────────────────────────────────────────┐
│ Vercel │                                                        │
│  ┌─────┴────────┐     ┌──────────────┐     ┌───────────────┐  │
│  │ API Routes    │────►│ Vercel AI SDK│────►│ Claude API    │  │
│  │ (Serverless)  │     │ 流式代理      │     │ (外部·运行时)  │  │
│  ├───────────────┤     └──────────────┘     └───────────────┘  │
│  │ Better Auth   │                                              │
│  │ 认证端点       │                                              │
│  └───────┬───────┘                                              │
│          │                                                      │
│  ┌───────┴───────┐                                              │
│  │ Drizzle ORM   │────► Neon PostgreSQL（外部·云端持久化）        │
│  └───────────────┘                                              │
└─────────────────────────────────────────────────────────────────┘
         │
┌────────┴────────┐
│ CDN (Vercel)    │  WASM 文件分发（replicad-opencascadejs ~7-10MB）
└─────────────────┘
```

**环境隔离**：开发环境使用本地 PostgreSQL，生产环境使用 Neon 无服务器分支。WASM 文件开发时从 node_modules 加载，生产通过 CDN 分发。

## 2. 架构决策

**D1 双模存储：匿名本地 + 认证云端**
cog.md 定义匿名使用、数据纯客户端。但用户跨设备续编和设计资产保护是刚需。采用双模：默认匿名（localStorage/IndexedDB），注册后 Drizzle→Neon 云同步。匿名设计可在注册时一键迁移。本地优先保证离线可用和隐私合规（C4）。

**D2 CAD 计算完全隔离到 Web Worker**
replicad 的 OCCT WASM 初始化耗时 2-5s，运行时单次布尔运算可达 200ms+。若在主线程执行会卡死 UI。所有 CAD 操作通过 postMessage 通信，主线程仅持有 mesh 数据用于 R3F 渲染。这是 C4 约束的核心落地。

**D3 CadEngine 抽象接口层**
replicad 处于 pre-1.0（v0.21），单人维护（风险 R1）。所有 replicad 调用封装在 CadEngine 接口后，下游代码不直接 import replicad。备选迁移目标：bitbybit-dev/occt。

**D4 AI 输出 CAD 参数 JSON，非几何体**
Claude API 输出结构化 JSON（型材列表+连接件+坐标），CAD 引擎据此生成 B-Rep 几何。AI 不直接生成 STEP/STL，保证参数可编辑、可验证、可回溯。通过 Vercel AI SDK streaming 实现逐步渲染。

**D5 结构安全检查器为规则引擎，非 FEA**
基于型材惯性矩×跨距×负载的简化力学公式，工业/家用场景使用不同安全系数。规则引擎在 Worker 内执行，与 CAD 计算共享型材参数上下文。不做有限元分析（pr.spec 明确排除）。

**D6 参数库随应用发布为静态 JSON**
型材（2020~8080）和连接件参数库以 JSON 文件形式内置于应用中，不依赖后端 API。更新参数库 = 更新应用版本。Zod schema 校验参数完整性。

## 3. 集成边界

| 系统 | 角色 | 访问方式 | 使用场景 |
|------|------|----------|----------|
| Claude API | 运行时依赖 | Vercel AI SDK → API Route 代理 | AI 设计生成、对话式修改、组装指导生成 |
| replicad (OCCT WASM) | 运行时依赖 | Web Worker 内加载，postMessage 通信 | 参数化建模、B-Rep 生成、STEP/STL 导出 |
| Neon PostgreSQL | 运行时依赖（认证用户） | Drizzle ORM via Serverless Driver | 设计方案云端持久化、用户偏好存储 |
| Better Auth | 运行时依赖（可选） | Next.js API Route 端点 | 用户注册/登录，会话管理 |
| CDN (Vercel Edge) | 静态分发 | HTTP + CSP 限制加载源 | WASM 文件、静态参数库 JSON |

## 4. 约束保障机制

| 约束 | 保障机制 |
|------|----------|
| C1 结构安全性 | Worker 内规则引擎：承载力公式（惯性矩/跨距/负载）+ 场景安全系数矩阵（工业 2.5×，家用 1.5×）。导出前强制校验门控——方案状态必须为"已验证"才能触发导出流程 |
| C2 型材连接件兼容性 | 参数库中维护 `compatibility_matrix[profile_series][connector_type]` 映射。每次 AI 生成/用户编辑后触发兼容性校验函数，冲突时阻断并返回兼容替代列表 |
| C3 加工参数精度 | 导出模块：切割长度 = 设计长度 - Σ(连接件占位尺寸)，所有数值 round 到 0.5mm，输出附带 ±0.5mm 公差标注。连接件占位数据来源于参数库 `connector.offset_depth` |
| C4 WASM 安全加载 | (a) WASM 通过 `new Worker()` 异步加载，主线程无 WASM import；(b) Next.js middleware 注入 CSP `script-src` 限制 WASM 加载域；(c) 图片前端压缩（canvas resize → WebP）后传 AI，不持久化原图；(d) 设计数据默认存浏览器，认证后加密同步 |
| C5 运输尺寸 | BOM 生成时遍历型材长度，>1500mm 标注快递受限，>6000mm 标注需特殊物流，并生成拼接替代方案（自动添加端面连接件） |
| C6 表面处理 | 输入阶段收集环境参数（下拉选择），AI prompt 中注入环境→表面处理映射规则，未指定时默认室内+阳极氧化银白 |
| C7 成本估算 | BOM 面板底部聚合：Σ(型材单价/m × 长度) + Σ(连接件单价 × 数量)，标注"参考价，不含加工费和运费"。存在更经济替代时触发优化建议 |

## 5. 跨模块业务流

**F1 AI 流式设计生成（跨 4 个边界）**
```
用户输入 → [Next.js API Route] → Vercel AI SDK stream → [Claude API]
                                                              │
          CAD参数JSON chunk ◄── streaming response ◄──────────┘
                │
    [主线程] 解析JSON → postMessage → [Worker] replicad建模
                                              │
    [主线程] R3F渲染 ◄── mesh数据 ◄───────────┘
```
关键：AI 输出的每个 JSON chunk 代表一个 DesignElement，前端逐个解析并驱动 Worker 增量建模，实现"逐步构建"视觉效果。

**F2 验证门控导出流程（跨 Worker↔主线程）**
```
用户点击导出 → [主线程] 检查方案状态
                  │
          状态≠已验证 → postMessage → [Worker] 执行结构检查
                                          │
          检查结果 ◄── postMessage ◄───────┘
                  │
          通过 → 生成加工参数/STEP/PDF → 浏览器下载
          不通过 → 返回警告列表，阻止导出
```

## 6. 权限模型

| 角色 | 查看设计 | 编辑设计 | 导出 | 云同步 | 管理账户 |
|------|---------|---------|------|--------|---------|
| 匿名用户 | 仅本地 | 仅本地 | ✓ | ✗ | ✗ |
| 注册用户 | 本地+云端 | 本地+云端 | ✓ | ✓ | ✓ |

**数据隔离**：注册用户的设计方案通过 `user_id` 外键隔离，Drizzle query 层强制 `where user_id = current_user`。匿名用户数据纯浏览器本地，服务端不可见。图片仅在 AI 请求时临时传输，不落盘（C4）。

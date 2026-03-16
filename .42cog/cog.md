# 铝型材设计师 - 认知模型文档

<meta>
  <document-id>alpdesigner-cog</document-id>
  <version>1.0.0</version>
  <project>ALPDesigner</project>
  <type>Cognitive Model</type>
  <created>2026-03-15</created>
  <depends>real.md</depends>
</meta>

## 文档用途

基于「主体 + 信息 + 上下文」框架，定义铝型材设计师系统中的核心实体、唯一编码、分类方式和信息流转关系。

---

## 1. 主体

<agents>

### 1.1 人类主体

<agent type="human" id="A1">
<name>设计用户</name>
<identifier>匿名会话 ID（无需注册，基于浏览器 sessionStorage）</identifier>
<classification>
  <by-expertise>新手（首次使用铝型材）| 有经验（了解铝型材规格）| 专业（从事相关行业）</by-expertise>
  <by-scenario>工业（流水线/货架/防护罩/物料架）| 商业（展架/陈列柜）| 个人 DIY（家具/设备框架）</by-scenario>
  <by-intent>快速查询（只想看材料清单）| 完整设计（需要 CAD 编辑和组装指导）</by-intent>
</classification>
<capabilities>用自然语言描述需求、上传参考图片、在 Web 编辑器中调整设计、导出设计方案</capabilities>
<goals>用最少的专业知识，获得可直接下单加工并组装的铝型材结构方案</goals>
</agent>

### 1.2 AI 主体

<agent type="ai" id="A2">
<name>设计助手</name>
<identifier>Claude API（claude-sonnet-4-6）</identifier>
<classification>
  <by-role>需求理解（解析描述和图片）| 方案生成（输出 CAD 参数 JSON）| 指导生成（输出组装步骤）</by-role>
</classification>
<interaction-pattern>
  输入：用户自然语言描述 + 可选参考图片 + 约束参数（尺寸限制、预算、用途）
  输出：结构化 CAD 参数 JSON（型材列表、连接件列表、坐标关系）
  二次输出：材料清单 BOM、组装指导步骤、结构安全评估
</interaction-pattern>
</agent>

### 1.3 CAD 引擎（系统主体）

<agent type="system" id="A3">
<name>CAD 渲染引擎</name>
<identifier>CadEngine 抽象接口 → replicad（OpenCascade.js WASM）+ react-three-fiber</identifier>
<classification>
  <by-function>参数化建模（JSON → 3D 几何体）| 交互编辑（拖拽、对齐、约束）| 导出（STEP/STL/PDF）</by-function>
</classification>
<interaction-pattern>
  输入：CAD 参数 JSON（设计助手输出 或 用户编辑操作）
  处理：通过 CadEngine 接口调用 replicad 生成 B-Rep 几何体 → Three.js 渲染
  输出：3D 可交互预览、导出文件
</interaction-pattern>
</agent>

</agents>

---

## 2. 信息

<information>

### 2.1 核心实体

<entity id="E1">
<name>铝型材 (Profile)</name>
<unique-code>系列代号 + 截面类型，如 "2020-standard"、"3030-light"、"4040-heavy"</unique-code>
<classification>
  <by-series>2020 系列 | 3030 系列 | 4040 系列 | 4545 系列 | 6060 系列 | 8080 系列 | 其他</by-series>
  <by-section>标准型 | 轻型 | 重型 | 直角型 | 圆弧型</by-section>
</classification>
<attributes>
  - 截面尺寸（宽 × 高，mm）
  - 槽宽（mm）
  - 中心孔径（mm）
  - 惯性矩（mm⁴，用于承载力计算）
  - 单位重量（kg/m）
  - 最大推荐跨距（mm，根据负载）
  - 适用场景（轻型 DIY | 中型商业 | 重型工业）
  - 表面处理（阳极氧化银白 | 阳极氧化黑 | 喷涂）
</attributes>
<relations>
  - Profile → Connector: N:N（一种型材可搭配多种兼容连接件，一种连接件也可适配同系列多种截面型材）
  - Profile → DesignElement: 1:N（一种型材可出现在多个设计元素中）
</relations>
</entity>

<entity id="E2">
<name>连接件 (Connector)</name>
<unique-code>类型代号 + 适配系列，如 "corner-bracket-2020"、"t-nut-m5-3030"</unique-code>
<classification>
  <by-type>角码 | T 型螺母 | 端面连接件 | 弹性扣件 | 合页 | 脚杯 | 滑块</by-type>
  <by-connection>直角连接 | T 形连接 | 十字连接 | 端面对接 | 可调角度</by-connection>
</classification>
<attributes>
  - 适配型材系列
  - 承载力（N）
  - 所需螺丝规格
  - 安装方式（内装 | 外装）
  - 占位尺寸（mm，影响型材切割长度）
</attributes>
<relations>
  - Connector → Profile: N:N（一种连接件可适配多种同系列型材）
  - Connector → Joint: 1:N（一种连接件可用于多个连接点）
</relations>
</entity>

<entity id="E3">
<name>板材 (Panel)</name>
<unique-code>材质代号 + 厚度，如 "acrylic-3mm"、"plywood-5mm"、"aluminum-sheet-1mm"</unique-code>
<classification>
  <by-material>亚克力 | 胶合板 | 铝板 | PVC 板 | 玻璃</by-material>
  <by-use>承重面板（桌面/层板）| 围护面板（侧板/背板）| 装饰面板</by-use>
</classification>
<attributes>
  - 材质
  - 厚度（mm）
  - 最大尺寸（mm × mm）
  - 承重能力（kg/m²）
  - 安装方式（槽插入 | 螺丝固定 | 粘接）
</attributes>
<relations>
  - Panel → Profile: N:N（面板嵌入型材槽或固定在型材上）
  - Panel → DesignElement: 1:N
</relations>
</entity>

<entity id="E4">
<name>设计方案 (Design)</name>
<unique-code>会话 ID + 时间戳，如 "sess_abc123_20260315143022"</unique-code>
<classification>
  <by-category>工业（流水线框架/货架/设备防护罩/物料架）| 家具（书架/桌子/柜子）| 设备框架（3D 打印机/CNC/机柜）| 展架/陈列柜 | 工作台 | 围栏/护栏 | 自定义</by-category>
  <by-status>草稿 | 已验证 | 已导出</by-status>
</classification>
<attributes>
  - 用户描述文本
  - 参考图片（可选）
  - CAD 参数 JSON（完整的结构定义）
  - 整体尺寸（长 × 宽 × 高，mm）
  - 预估总重量（kg）
  - 预估承载力（kg）
  - 结构安全评分（通过/警告/不通过）
</attributes>
<relations>
  - Design → DesignElement: 1:N（一个方案包含多个设计元素）
  - Design → BOM: 1:1（一个方案对应一份材料清单）
</relations>
</entity>

<entity id="E5">
<name>设计元素 (DesignElement)</name>
<unique-code>方案 ID + 元素序号，如 "sess_abc123_20260315143022_elem_01"</unique-code>
<classification>
  <by-role>结构杆件（竖杆/横杆/斜撑）| 面板 | 连接组件 | 辅助件（脚杯/滑轮/把手）</by-role>
</classification>
<attributes>
  - 引用的型材/板材/连接件 ID
  - 在 3D 空间中的位置（x, y, z）
  - 旋转（rx, ry, rz）
  - 长度/尺寸（对于型材为切割长度）
  - 加工要求（钻孔位置、攻丝规格）
</attributes>
<relations>
  - DesignElement → Joint: 1:N（一个元素可参与多个连接点）
</relations>
</entity>

<entity id="E6">
<name>连接点 (Joint)</name>
<unique-code>方案 ID + 连接序号，如 "sess_abc123_20260315143022_joint_01"</unique-code>
<classification>
  <by-geometry>直角接合 | T 形接合 | 十字接合 | 端面接合 | 斜角接合</by-geometry>
  <by-status>已配置（有连接件）| 未配置（需要用户选择）| 冲突（不兼容）</by-status>
</classification>
<attributes>
  - 参与的设计元素 ID 列表
  - 使用的连接件 ID
  - 接合位置坐标
  - 接合角度
  - 承载力需求
</attributes>
<relations>
  - Joint → DesignElement: N:N（一个连接点关联两个或多个设计元素）
  - Joint → Connector: N:1（一个连接点使用一种连接件）
</relations>
</entity>

<entity id="E7">
<name>材料清单 (BOM)</name>
<unique-code>与所属 Design 相同 ID + "_bom" 后缀</unique-code>
<classification>
  <by-format>简要清单（采购用）| 详细清单（加工用）| 组装清单（DIY 指导用）</by-format>
</classification>
<attributes>
  - 型材明细（型号、长度、数量、加工要求）
  - 连接件明细（型号、数量）
  - 板材明细（材质、尺寸、数量）
  - 预估总价
  - 总重量
</attributes>
<relations>
  - BOM → Design: 1:1（一份材料清单对应一个设计方案）
</relations>
</entity>

### 2.2 信息流

<information-flow>

<flow id="F1" name="AI 辅助设计流程">
  用户 → 输入描述/图片 → 设计助手(AI) → 生成 CAD 参数 JSON → CAD 引擎 → 渲染 3D 预览 → 用户查看
</flow>

<flow id="F2" name="手动编辑流程">
  用户 → Web 编辑器操作（拖拽/对齐/修改参数）→ CAD 引擎 → 更新 3D 模型 → 连接点自动检测 → 用户确认
</flow>

<flow id="F3" name="结构验证流程">
  用户/AI → 设计方案 → 结构检查器 → 逐一验证（承载力/连接兼容/跨距安全）→ 输出安全评分和警告 → 用户查看
</flow>

<flow id="F4" name="导出流程">
  用户 → 选择导出格式 → 系统生成（BOM/加工参数/组装指导/效果图/STEP 文件）→ 用户下载
</flow>

</information-flow>

</information>

---

## 3. 上下文

<context>

### 3.1 应用上下文
- Web 单页应用（SPA），基于 Next.js 16 App Router
- 无需注册登录，基于浏览器会话的匿名使用
- 设计数据存储在浏览器 localStorage/IndexedDB
- 支持桌面浏览器（Chrome/Firefox/Safari/Edge），优化大屏体验

### 3.2 技术上下文
- 前端渲染：react-three-fiber（Three.js 的 React 渲染器）
- CAD 计算：replicad 在 Web Worker 中运行 OpenCascade WASM，通过 CadEngine 抽象接口封装调用
- AI 通信：客户端 → Next.js API Route → Claude API，流式输出 CAD 参数
- 数据格式：统一使用 JSON Schema 定义型材参数、设计方案、BOM
- 无后端数据库，所有数据在客户端处理（隐私优先）

### 3.3 用户体验上下文
- 核心情感：**降低门槛感** — 让不懂 CAD 的用户也能设计铝型材结构
- 交互风格：对话式输入 → 即时 3D 预览 → 可视化编辑 → 一键导出
- 关键时刻：首次看到 3D 模型生成的「惊喜感」；导出材料清单时的「可行感」
- 容错：AI 生成的方案允许用户自由修改，系统实时提示兼容性问题而非阻止操作

</context>

---

## 4. 权重矩阵

<weights>

| 实体/交互 | 重要性 | 说明 |
|-----------|--------|------|
| 铝型材参数库 (E1) | ★★★★★ | 核心基础数据，精度直接影响所有下游功能 |
| 连接件参数库 (E2) | ★★★★★ | 与型材同等重要，兼容性错误代价最高 |
| 连接点检测 (E6) | ★★★★☆ | 自动检测大幅提升用户体验，但可手动配置 |
| AI 设计生成 (F1) | ★★★★☆ | 核心差异化功能，但不是唯一入口 |
| 结构安全验证 (F3) | ★★★★★ | 安全相关，不可妥协 |
| 板材参数库 (E3) | ★★★☆☆ | 辅助功能，MVP 可简化 |
| Web 编辑器 (F2) | ★★★☆☆ | 重要但复杂度高，可分阶段实现 |
| 材料清单 (E7) | ★★★★☆ | 用户最终拿到手的采购/加工依据，精度要求高 |
| 导出功能 (F4) | ★★★★☆ | 用户最终需要的产出物 |

</weights>

---

## 5. 验证检查清单

- [ ] 所有型材实体有唯一编码（系列 + 截面类型）
- [ ] 所有连接件有兼容型材系列的映射关系
- [ ] 连接点检测覆盖五种基本接合几何（直角/T 形/十字/端面/斜角）
- [ ] AI 输出的 CAD 参数 JSON 有明确的 Schema 定义
- [ ] 信息流覆盖从输入到导出的完整闭环
- [ ] 实体关系中的 N:N 关系有明确的兼容性规则
- [ ] 权重矩阵与 real.md 的约束优先级一致

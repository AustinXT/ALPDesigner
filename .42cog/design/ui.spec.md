# 铝型材设计师 — UI 设计规约

## 1. 设计决策

### 应用类型
**结论**: SPA — **理由**: 核心交互是 3D 视口+对话的工作空间模式，页面切换会打断 WASM 运行时和建模状态（cog.md §3.1）

### 导航结构
**类型**: 工作空间三栏面板（无路由导航） — **理由**: 用户操作集中在「输入→预览→编辑→导出」单一工作流。左|中|右三栏同时可见比页面切换更高效：对话区(320px) | 3D 视口(flex-1) | 属性/BOM(360px 可折叠)

### 配色方案
**主色相**: 220° (工程蓝) — 与 3D 视口深色背景对比良好，CAD 用户对蓝色系有认知惯性
```css
--color-primary: oklch(0.55 0.15 220);
```

### 自定义设计令牌
```css
--font-sans: ui-sans-serif, system-ui, -apple-system, "PingFang SC", "Microsoft YaHei", sans-serif;
--viewport-bg: oklch(0.15 0.01 220);  /* 3D 视口深色背景 */
```

## 2. 页面规格

### 主工作空间 `/`

**布局**: 三栏响应式 — 左栏(对话, 320px) | 中栏(3D 视口, flex-1) | 右栏(属性/BOM, 360px 可折叠)

**业务组件**:
- `ChatPanel` — 自然语言输入 + 场景模板快捷按钮（书架/工作台/货架等）+ 图片拖拽上传区 + 对话历史。输入后触发 AI 流式生成
- `Viewport3D` — R3F 视口，OrbitControls。WASM 未加载时显示骨架屏+进度。接收 Worker mesh 数据渲染。点击元素触发选中
- `PropertyPanel` — 选中元素时：型材系列/长度/表面处理可编辑属性。未选中时：方案概览（整体尺寸、总重量）
- `BOMPanel` — 按型材/连接件/板材分组。条目点击 ↔ 3D 高亮联动。底部成本估算区（标注「参考价」）。CSV 导出按钮
- `SafetyBadge` — 安全评分徽章（通过🟢/警告🟡/不通过🔴），点击展开报告定位问题元素
- `ExportMenu` — 下拉菜单：加工参数 CSV / STEP / 组装指导 PDF / 效果图。方案未验证时加工导出禁用
- `MockBanner` — Mock 模式顶部横幅 `🎭 演示模式`
- `CompatWarning` — 兼容性冲突时：红色边框标记冲突元素 + tooltip 说明原因 + 替换建议

**交互流程**:
1. 输入描述 → AI 流式返回 CAD 参数 JSON → designStore 逐步更新 → Viewport3D 增量渲染（逐步构建动画）
2. 点击 3D 元素 → uiStore.selectedElementId 更新 → PropertyPanel 切换属性视图
3. 修改属性 → designStore 更新 → Worker 重新建模 → 3D 更新 + 兼容性校验触发
4. 点击导出 → 检查 design.status → 未验证则 Worker 执行结构检查 → 通过后生成文件下载

## 3. 状态 Schema

```typescript
// designStore — 设计方案核心状态
interface DesignState {
  design: Design | null              // 方案元数据（status: draft|verified|exported）
  elements: DesignElement[]          // 设计元素（型材杆件、面板等）
  joints: Joint[]                    // 连接点
  bom: BOM | null                    // 材料清单
  warnings: ValidationWarning[]      // 兼容性/安全警告列表
}

// catalogStore — 参数库（静态 JSON 加载，Mock 数据初始化，禁止空数组）
interface CatalogState {
  profiles: Profile[]                // 2020~8080 系列型材
  connectors: Connector[]            // 角码/T型螺母/端面连接件等
  panels: Panel[]                    // 亚克力/胶合板/铝板等
  compatibilityMatrix: Record<string, string[]>  // series → compatible connector types
}

// uiStore — UI 交互状态
interface UIState {
  selectedElementId: string | null
  rightPanel: 'property' | 'bom'
  isMockMode: boolean                // useMockMode 开关
  isWasmReady: boolean               // WASM 加载状态
}

// chatStore — 对话状态
interface ChatState {
  messages: ChatMessage[]            // 对话历史（含 AI streaming chunks）
  isGenerating: boolean              // AI 流式生成中
}
```

## 4. 功能优先级

| 功能 | 优先级 | 依赖 | 对应故事 |
|------|--------|------|----------|
| 自然语言输入 + AI 生成 3D 方案 | P0 | catalogStore | MS-L-01 |
| BOM 查看 + CSV 导出 | P0 | designStore.bom | MS-L-03 |
| 型材连接件兼容性校验 | P0 | compatibilityMatrix | MS-D-02 (C2) |
| 加工参数 CSV 导出（验证门控） | P0 | design.status=verified | MS-L-07 (C1+C3) |
| 图片上传辅助描述 | P1 | — | MS-L-02 |
| 对话式迭代修改 | P1 | chatStore | MS-D-01, MS-G-01 |
| 元素选中 + 属性编辑 | P1 | selectedElementId | MS-L-04 |
| 拖拽布局 + 对齐辅助 | P1 | 元素选中 | MS-L-05 |
| 结构安全检查 + 报告 | P1 | Worker 规则引擎 | MS-D-03 |
| 连接点自动检测 | P1 | Worker 几何计算 | MS-L-06 |
| 组装指导 PDF 导出 | P1 | verified | MS-L-08 |
| 运输尺寸提醒 | P2 | bom 长度遍历 | MS-D-04 (C5) |
| 成本估算 + 优化建议 | P2 | bom 聚合 | MS-D-05 (C7) |

## 5. 扩展点

| 当前 | 未来 | 迁移方式 |
|------|------|----------|
| Zustand + localStorage | Drizzle → Neon PostgreSQL | designStore 增加 syncToCloud，Better Auth 认证后触发 |
| 静态 JSON 参数库 | API 动态加载 | catalogStore.load() 改为 fetch API endpoint |
| 客户端规则引擎检查 | 服务端 FEA 辅助 | Worker postMessage 接口不变，增加可选 API 二次校验 |
| 匿名 sessionStorage | Better Auth 注册/登录 | uiStore 增加 user 字段，条件渲染认证 UI |
| 客户端图片压缩传 AI | 云端图片理解服务 | ChatPanel 上传逻辑抽为 adapter |

# 铝型材设计师 - 现实约束文档

<meta>
  <document-id>alpdesigner-real</document-id>
  <version>1.0.0</version>
  <project>ALPDesigner</project>
  <type>Reality Constraints</type>
  <created>2026-03-15</created>
</meta>

## 文档用途

定义铝型材设计师项目必须遵守的硬性约束，聚焦于 AI 生成设计时不易预见但违反后会造成现实世界损害的问题。

<constraints>

## 必需约束

<constraint required="true" id="C1">
<title>结构安全性验证</title>
<description>AI 生成的铝型材设计方案必须通过基本结构安全检查：型材截面承载力匹配负载需求、连接件数量和类型满足结构稳定性要求、悬臂/跨距不超过型材规格的安全极限。工业场景（流水线、货架）须额外考虑动态载荷、长期疲劳和振动影响。不得输出未经验证的设计方案供用户加工和组装。</description>
<rationale>用户会根据导出的参数直接下单加工并组装。家用场景下结构倒塌造成财产损失或人身伤害；工业场景下流水线框架失稳或货架坍塌后果更为严重，可能涉及重型设备和大量货物。</rationale>
<violation-consequence>结构在承重或运转中失稳倒塌，造成物品损坏、设备损毁或人员受伤。工业场景后果尤为严重。</violation-consequence>
</constraint>

<constraint required="true" id="C2">
<title>型材与连接件兼容性</title>
<description>设计方案中的铝型材系列（如 2020、3030、4040）与连接件必须严格匹配。不同系列的型材槽宽不同，T 型螺母、角码、连接板等必须与所选型材系列兼容。同一设计中混用不同系列时，必须在接合处使用正确的转接件。</description>
<rationale>铝型材行业的型材规格（槽宽、螺孔间距）是物理约束。如果生成的材料清单中型材和连接件不匹配，用户收到材料后无法组装，浪费金钱和时间。</rationale>
<violation-consequence>用户购买的材料无法组装，造成经济损失和用户信任丧失。</violation-consequence>
</constraint>

<constraint required="true" id="C3">
<title>加工参数精度</title>
<description>导出的加工参数（切割长度、钻孔位置、攻丝规格）必须使用毫米为单位，精度到 0.5mm。切割长度必须考虑连接件占用的空间（如角码厚度、端面连接件的插入深度）。所有尺寸必须标注公差范围。</description>
<rationale>加工参数会直接发送给铝型材工厂进行切割和加工。精度不足或未考虑连接件占位会导致加工出的型材无法正确组装，返工成本高且工期延误。</rationale>
<violation-consequence>工厂按照错误参数加工的型材无法组装，需要重新下单加工，造成双倍材料费和时间浪费。</violation-consequence>
</constraint>

<constraint required="true" id="C4">
<title>WASM 资源加载安全</title>
<description>OpenCascade WASM 模块体积大（约 20-30MB），必须使用懒加载 + Web Worker 隔离，不得在主线程同步加载。WASM 文件必须通过 CDN 分发并启用 Content-Security-Policy 限制加载源。用户上传的图片必须在前端压缩后传输，不存储原图。</description>
<rationale>WASM 同步加载会阻塞 UI 导致页面无响应；未限制 WASM 加载源存在供应链攻击风险；用户图片可能包含敏感信息。</rationale>
<violation-consequence>页面加载卡死导致用户流失；WASM 被替换为恶意代码；用户隐私数据泄露。</violation-consequence>
</constraint>

## 可选约束

<constraint required="false" id="C5">
<title>运输尺寸限制</title>
<description>单根型材切割长度应考虑物流运输限制：快递渠道通常 ≤ 1.5m，普通物流 ≤ 6m。超过常规运输长度的型材需在方案中明确标注，并提示用户确认运输方式。整体设计在可能的情况下应优先选择可拼接的短型材方案，而非单根超长型材。</description>
<rationale>超长型材运费显著增加，且运输过程中容易弯曲变形。用户收到变形型材后无法使用，需要退换货重新采购，造成工期延误和额外成本。</rationale>
</constraint>

<constraint required="false" id="C6">
<title>表面处理与环境适配</title>
<description>设计方案应要求用户指定使用环境（室内/室外/潮湿/腐蚀性/食品接触等），并据此推荐合适的铝合金牌号和表面处理方式。室外或潮湿环境须提醒选择阳极氧化或粉末喷涂；有腐蚀性介质接触时须提醒选用防腐等级更高的处理；食品接触场景须符合食品级要求。未指定环境时默认按室内普通环境处理，但须在方案中注明适用范围。</description>
<rationale>未经适当表面处理的铝型材在室外或潮湿环境中会氧化腐蚀，削弱结构强度并影响外观。用户往往忽略环境因素，直到型材出现白色氧化斑点或强度下降才发现问题，此时结构已存在安全隐患。</rationale>
</constraint>

<constraint required="false" id="C7">
<title>成本可见性</title>
<description>设计方案应基于型材重量、连接件数量和表面处理方式提供大致的材料成本估算范围。当设计方案的估算成本明显偏高（如存在可替代的更经济的型材系列或连接方式）时，应主动提示用户并提供优化建议。成本估算仅作参考，须标注不含加工费和运费。</description>
<rationale>用户在设计阶段往往缺乏成本概念，可能选择了过大规格的型材或过多连接件。没有成本反馈的情况下，用户可能在询价时才发现方案远超预算，不得不推翻设计重新来过，浪费前期的设计和沟通时间。</rationale>
</constraint>

</constraints>

## 技术环境

<environment>
<stack>
  - 框架：Next.js 16 (App Router) + React 19 + TypeScript
  - 运行时：Bun
  - 3D 渲染：react-three-fiber + @react-three/drei + Three.js
  - CAD 内核：replicad（基于 OpenCascade WASM，MIT 协议）
  - AI 服务：Claude API（自然语言理解 + CAD 参数生成）
  - 部署：Vercel（Edge Runtime + Serverless Functions）
  - 数据格式：JSON（型材参数库、设计方案、BOM）
  - 导出格式：STEP（CAD 交换）、STL（3D 预览）、PDF（组装指导）、CSV（材料清单）
</stack>
</environment>

## 约束检查清单

- [ ] C1: 设计方案生成后经过结构安全检查
- [ ] C2: 材料清单中型材与连接件兼容性已验证
- [ ] C3: 加工参数精度达到 0.5mm 且考虑连接件占位
- [ ] C4: WASM 懒加载 + Worker 隔离 + CSP 配置
- [ ] C5: 超长型材已标注运输提醒
- [ ] C6: 表面处理建议与使用环境匹配
- [ ] C7: 设计方案包含材料成本估算

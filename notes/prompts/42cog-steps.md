## 1. 生成 meta.md

请根据下列内容，生成 .42cog/meta.md：

<!--活水智聊（42chat）是一款面向个人用户与小型团队设计的 一款基于 Web 的多模型 AI 对话中枢，为频繁在不同模型间工作的个人用户提供统一入口。它将会话管理、模型切换、提示词模板、联网搜索与数据同步收拢到一致的交互中。在强制登录与基础角色的保障下，用户无需改变既有习惯即可获得更连贯的对话体验：对话被持续保存、可被检索，也可被导出；配置集中收纳、随时可测、按需启用。
活水智聊（42chat） 面向多模型重度用户，聚合登录、配置与切换，对话管理与检索，模板与联网搜索，以及云端同步与本地兜底。用户在单一界面持续完成跨模型交流，降低切换成本，保留知识积累，并在需要时获取实时信息。-->

作者：NightVoyager

## 2. 生成 real.md与cog.md

请使用已经注册好的Claude Skill：.claude/skills/42edu/meta-42cog/SKILL.md

生成real.md与cog.md，中文版。注意，如果以前有内容，直接备份在同文件夹后（加上日期后）替换。

## 3. 生成 .42cog/spec/pmpr.spec.md

请使用已经注册好的Claude Skill：.claude/skills/42edu/pm-product-requirements/SKILL-lite.zh.md

注意参考：

.42cog/real.md
.42cog/cog.md

如有必要，请开多个task加速执行任务。注意：

可能需要在 Write 工具中包含 content 参数

## 4. 生成 .42cog/spec/pm/userstory.spec.md

请使用已经注册好的Claude Skill：.claude/skills/42edu/pm-user-story/SKILL-lite.zh.md

注意，生成时参考：

.42cog/real.md
.42cog/cog/cog.md

还需要同步参考：

.42cog/spec/pm/pr.spec.md

如有必要，请开多个task加速执行任务。注意：

可能需要在 Write 工具中包含 content 参数

## 5. 生成 .42cog/spec/dev/sys.spec.md

请使用已经注册好的Claude Skill：.claude/skills/42edu/dev-system-architecture/SKILL.md

注意，生成时参考：

.42cog/real.md
.42cog/cog/cog.md

还需要同步参考：

.42cog/spec/pm/pr.spec.md
.42cog/spec/pm/userstory.spec.md

如有必要，请开多个task加速执行任务。注意：

可能需要在 Write 工具中包含 content 参数

我们默认采取的技术栈是：

全站框架：Next.js，参考：https://github.com/vercel/next.js
css框架：Tailwind CSS，参考：https://github.com/tailwindlabs/tailwindcss
ui组件：shadcn/ui，参考：https://github.com/shadcn-ui/ui
包管理：bun，参考：https://bun.com
数据库：本地PostgreSQL，云端neon，参考：https://neon.com
ORM框架：Drizzle ORM，参考：https://github.com/drizzle-team/drizzle-orm
认证授权：Better Auth，参考：https://www.better-auth.com
AI框架：Vercel AI SDK，参考：https://ai-sdk.dev

如果有需要，还可以继续默认采取这些技术框架：
1, Schema Validations - Zod
2, State Management - Zustand
3, Search params state manager - Nuqs
4, Tables - Tanstack Tables
5, Forms - React Hook Form
6, Command+k interface - kbar
7、Linting - ESLint
8, Pre-commit Hooks - Husky
9、Formatting - Prettier

## 6. 生成 .42cog/spec/design/ui.spec.md

请使用已经注册好的 Claude Skill：.claude/skills/42edu/design-ui-design/SKILL.md

注意，生成时参考：

.42cog/meta.md
.42cog/real.md
.42cog/cog/cog.md

还需要同步参考：

.42cog/spec/pm/pr.spec.md
.42cog/spec/pm/userstory.spec.md
.42cog/spec/dev/sys.spec.md

如有必要，请开多个task加速执行任务。注意：

可能需要在 Write 工具中包含 content 参数

## 7. 生成 schema.ts

**需要先安装neon插件；且手动创建42chatdemo-learn数据库**

请使用已经注册好的Claude Skill：.claude/skills/42edu/dev-database-design/SKILL.md

生成数据库schema文件（schema.ts）。

注意，生成时参考：

.42cog/meta/meta.md
.42cog/real/real.md
.42cog/cog/cog.md

还需要同步参考：

.42cog/spec/pm/pr.spec.md
.42cog/spec/pm/userstory.spec.md
.42cog/spec/dev/sys.spec.md

还需要同步参考：

https://neon.com/guides/neon-auth-nextjs

生成后，根据项目技术栈安装后端依赖。

---
（建议拆成两次对话）
---

请使用plugin:neon-plugin:neon找到 XXX 数据库，切换到开发分支，然后为我配置数据库连接，迁移并推送到neon数据库。

连接neon数据库时参考：.claude/skills/42edu/dev-database-design/references/neon-drizzle-tutorial.md

## 8. 生成 CRUD 代码

请使用已经注册好的Claude Skill：.claude/skills/42edu/dev-coding/SKILL.md

生成 CRUD 代码。要求：

- 使用 server actions + api router 方案
- 向第三方提供一个简化版本的 api 查询接口：
  ```ts
  GET /api/v1/conversations
    - page, limit（分页）
    - q（搜索）
  
  GET /api/v1/conversations/:id
    - include=messages（可选包含消息）
  ```
- 创建 Mock 数据脚本 src/db/seed.ts
- 前端页面使用数据库数据

注意，生成时参考：

.42cog/meta/meta.md
.42cog/real/real.md
.42cog/cog/cog.md

还需要同步参考：

.42cog/spec/pm/pr.spec.md
.42cog/spec/pm/userstory.spec.md
.42cog/spec/dev/sys.spec.md

## 9. 对现有的 CRUD 操作进行自动化测试

请使用已经注册好的Claude Skill：.claude/skills/42edu/dev-quality-assurance/SKILL.md

对现有的 CRUD 操作进行自动化测试。要求：

- API 层：用 bun 直接调用测试
- UI 层：配置 Playwright MCP 进行端到端测试
  - 参考 .42cog/spec/pm/userstory.spec.md
- 数据库：使用 Neon 开发分支隔离
- 使用 mock 数据

## 10. UI 模块集成数据库 CRUD 操作

请使用已经注册好的Claude Skill：.claude/skills/42edu/dev-coding/SKILL.md

在 UI [模块]集成数据库 CRUD 操作。

注意，生成时参考：

.42cog/spec/design/ui.spec.md

## 11. 实装认证子系统 (Auth)

请使用已经注册好的Claude Skill：.claude/skills/42edu/dev-coding/SKILL.md

实装认证子系统 (Auth) 。要求：

- 只需要 email + 密码登录（不需要 OAuth）
- 不需要邮箱验证（注册后直接登录）、不需要密码重置功能
- 需要完整的登录/注册页面 UI
- 使用 RBAC 模型，拥有游客、普通用户、管理员角色

注意，生成时参考：

.42cog/meta/meta.md
.42cog/real/real.md
.42cog/cog/cog.md

还需要同步参考：

.42cog/spec/pm/pr.spec.md
.42cog/spec/pm/userstory.spec.md
.42cog/spec/dev/sys.spec.md

还需要同步参考：

src/db/schema.ts
neon_auth 官方文档（使用 plugin:neon-plugin:neon 或 mcp:context7）

## 本地直接上传到edgeone部署

请使用已经注册好的 .claude/skills/dev-deployment-v1/SKILL.zh.md

把这个SSR项目部署到edgeone的国内节点，项目名称为“42chatdemo”

（如果使用海外API，如ChatGPT等）
把这个SSR项目部署到edgeone的海外节点，项目名称为“42chatdemo”

## 通过CNB推送到edgeone部署

请使用已经注册好的 .claude/skills/dev-deployment-v1/SKILL.zh.md

生成这个SSR项目.cnb.yml文件用于通过CNb部署到edgeone，部署到海外节点，项目名称为“42chatdemo”，仅生成配置文件，不直接部署。

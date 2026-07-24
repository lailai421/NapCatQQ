# 项目概览

- 项目名：NapCatQQ / napcat。
- 项目目的：基于 NTQQ 的现代 Bot 协议端实现，主要提供 OneBot 11 兼容接口、WebUI、插件与协议适配能力。
- 技术栈：TypeScript monorepo，pnpm workspace，Vite，Express，ws，Vitest，ESLint + neostandard。
- 主要目录：
  - `packages/napcat-core`：核心 NTQQ wrapper、服务类型、API、消息/包转换。
  - `packages/napcat-onebot`：OneBot action、事件、网络适配。
  - `packages/napcat-shell` / `packages/napcat-shell-loader`：QQNT 注入/启动相关。
  - `packages/napcat-webui-frontend` / `packages/napcat-webui-backend`：WebUI 前后端。
  - `packages/napcat-test`：Vitest 测试。
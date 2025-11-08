# Nova Chat Frontend (React + TypeScript)

本仓库用于承载面向用户的聊天界面，最终将部署到 **Cloudflare Pages** 并通过 GraphQL 与后端 Serverless Worker 通信。

## 技术选型

- **React 19 + TypeScript**：以函数组件 + Hooks 构建交互，保障类型提示与未来 React Compiler 兼容。
- **Vite 5+**：提供快速的本地开发、TS 编译与打包体验，输出可直接部署到 Pages 的静态产物。
- **Apollo Client 4**：负责 GraphQL 查询/变更、缓存管理与错误处理，方便对接后端 Worker。
- **Tailwind 或 CSS Modules（二选一）**：默认采用 CSS Modules；如需主题/响应式可引入 Tailwind。
- **ESLint + Biome/Prettier（可选）**：保持代码风格一致，便于在 CI 中强制质量门槛。

## GraphQL 通信约定

- `Query.history`: 获取聊天历史，返回消息数组（`id`, `role`, `text`, `createdAt`）。
- `Mutation.sendMessage(input: { text: string })`: 发送用户消息并返回机器人回复。
- 所有请求通过 `VITE_GRAPHQL_ENDPOINT` 指定的 HTTPS 入口发往 Cloudflare Worker。

在 `src/graphql/` 目录中统一维护 `client.ts` 与 `operations.ts`，并在组件中用 Apollo Hooks 调用。

## 本地开发与调试

```bash
npm install                  # 第一次克隆后安装依赖
cp .env.example .env         # 配置 GraphQL 端点
npm run dev                  # http://localhost:5173，支持 HMR
npm run lint                 # 静态检查（可选）
```

`.env` 示例：

```
VITE_GRAPHQL_ENDPOINT=https://<your-worker-subdomain>.workers.dev/graphql
```

## 构建与部署到 Cloudflare Pages

1. 本地打包：`npm run build`，产物在 `dist/`。
2. Pages 项目配置：
   - **Build command**: `npm run build`
   - **Build output directory**: `dist`
   - **NODE_VERSION** 与 `VITE_GRAPHQL_ENDPOINT` 作为环境变量写入 Pages。
3. 触发部署：
   - 方式一：绑定 GitHub 仓库，推送到 `main` / `production` 自动构建。
   - 方式二：使用 `wrangler pages deploy dist` 进行手动上传（需登录 `wrangler`）。

## 前端仓库操作清单

| 操作 | 命令 |
| --- | --- |
| 安装依赖 | `npm install` |
| 启动开发 | `npm run dev` |
| 单元测试（如配置 Vitest） | `npm test` |
| 构建发布包 | `npm run build` |
| 本地预览 | `npm run preview` |

将本仓库推送到 GitHub 时，推荐开启以下工作流：

- `ci.yml`：运行 `npm ci && npm run build && npm run lint`。
- `pages-deploy.yml`（可选）：在合并后自动调用 `wrangler pages deploy`。

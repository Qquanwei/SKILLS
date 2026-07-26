---
name: vercel-supabase-deploy
description: >-
  Deploys Next.js apps to Vercel with the official Supabase marketplace
  integration for one-click env injection and the Supabase GitHub Integration
  for automatic production migrations. Use when the user asks about Vercel deploy,
  Supabase integration, GitHub auto migrations, Deploy to production, 一键部署,
  marketplace env vars, NEXT_PUBLIC_SUPABASE_*, ANON_KEY, SERVICE_ROLE_KEY,
  POSTGRES_URL, Auth redirect URLs, or vercel env pull.
  Also covers fronting Vercel with a China CDN (阿里云 ESA / 边缘安全加速 / CDN 回源),
  custom domain DNS, 强制 HTTPS 导致域名验证失败, and cert issuance troubleshooting.
---

# Vercel + Supabase 一键部署

面向 **Next.js（App Router）+ `@supabase/ssr`** 项目：用 Vercel Marketplace 集成自动注入环境变量，用 Supabase GitHub Integration 自动部署数据库迁移。

## 何时使用

- 用户要把仓库部署到 Vercel，并接 Supabase
- 用户提到「一键部署」「集成注入的环境变量」
- 需要确认 / 兼容集成写入的 env 名称
- 部署后 Auth 回调、迁移、预览环境变量出问题

## 两个集成各自负责什么

不要把两个同名的 Supabase 集成混为一谈：

| 集成 | 入口 | 职责 |
| --- | --- | --- |
| **Vercel Marketplace → Supabase** | Vercel Project → Integrations | 创建/关联 Supabase 项目，并向 Vercel 注入连接环境变量 |
| **Supabase → GitHub Integration** | Supabase Project Settings → Integrations | 监听 GitHub 提交，自动应用 `supabase/migrations`、部署声明在 `config.toml` 的 Edge Functions 和 Storage buckets |

因此，完整的一键部署同时启用这两个集成。配置好 Supabase GitHub Integration 后，生产迁移随代码推送自动执行，不要再让用户去 SQL Editor 手动粘贴 migration，也不需要从本地手动运行 `supabase db push`。

## 一键部署步骤

1. **导入仓库到 Vercel**  
   [vercel.com/new](https://vercel.com/new) → 选 GitHub 仓库 → Framework 选 Next.js（一般自动识别）。

2. **安装 Supabase 集成**  
   Vercel → Project → Integrations / Marketplace → **Supabase** → 关联已有或新建 Supabase 项目。  
   集成会把环境变量同步到 Vercel（Production / Preview / Development 建议都勾选）。

3. **连接 Supabase GitHub Integration**
   Supabase → Project Settings → **Integrations** → **GitHub Integration** → Authorize GitHub：
   - 选择与 Vercel 相同的 GitHub 仓库
   - **Working directory**：填写包含 `supabase/` 的父目录；若 `supabase/` 位于仓库根目录，填 `.`
   - 开启 **Deploy to production**
   - 生产分支设为仓库实际生产分支（通常为 `main`）
   - 启用集成

   之后每次 push / merge 到生产分支，Supabase 自动应用尚未执行的 `supabase/migrations/*.sql`。

4. **Deploy**
   推送代码即可：Vercel 自动部署应用，Supabase GitHub Integration 自动部署数据库变更。无需在 Vercel 手填 URL / Anon Key，也无需手动执行 migration。

5. **配置 Auth URL**  
   Supabase → Authentication → URL Configuration：  
   - **Site URL**：生产域名（如 `https://xxx.vercel.app` 或自定义域）  
   - **Redirect URLs**：至少包含  
     - `https://<prod-domain>/**`  
     - `https://*-<team>.vercel.app/**`（Preview，或按集成自动写入的为准）  
     - 本地：`http://localhost:3000/**`  
   回调路径常用：`/auth/callback`

6. **（可选）拉本地环境**  
   ```bash
   vercel link
   vercel env pull .env.local
   ```

## Supabase GitHub 自动部署细节

依据 Supabase 官方部署与 GitHub Integration 文档：

- **生产自动部署适用于所有套餐**；PR 对应的 Supabase Preview Branching 需要 Pro 套餐
- 开启 **Deploy to production** 后，push / merge 到生产分支会自动：
  - 应用新的 `supabase/migrations/*.sql`
  - 部署 `config.toml` 中声明的 Edge Functions
  - 部署 `config.toml` 中声明的 Storage buckets
- API、Auth 配置和 seed 文件默认不会部署到生产，仍需按官方支持范围单独处理
- 如果远程数据库已有通过 Dashboard 创建、但仓库中没有记录的结构，首次接入前先用 `supabase db pull` 纳入 migration 历史；不要把生产 SQL Editor 当作日常迁移方式
- 在 GitHub 分支保护中把 Supabase integration check 设为 required，避免失败的 migration 被合并进生产分支

需要 PR 级数据库预览环境时，再开启 **Automatic branching**；可同时开启 **Supabase changes only**，只在 Supabase 文件变化时创建 Preview Branch。

官方文档：

- [Supabase GitHub Integration](https://supabase.com/docs/guides/deployment/branching/github-integration)
- [Supabase Deployment](https://supabase.com/docs/guides/deployment)
- [Going into production](https://supabase.com/docs/guides/deployment/going-into-prod)

## 集成会注入的环境变量

以当前 Marketplace 集成为准（你确认过的列表）：

```text
POSTGRES_URL
POSTGRES_PRISMA_URL
POSTGRES_URL_NON_POOLING
POSTGRES_USER
POSTGRES_HOST
POSTGRES_PASSWORD
POSTGRES_DATABASE
SUPABASE_ANON_KEY
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
SUPABASE_JWT_SECRET
NEXT_PUBLIC_SUPABASE_ANON_KEY
NEXT_PUBLIC_SUPABASE_URL
```

### 应用层怎么读（Next.js）

| 变量 | 用途 |
| --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | 浏览器 + 服务端 Supabase client（必用） |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | 浏览器 + 服务端 anon/publishable key（必用） |
| `SUPABASE_URL` | 无 `NEXT_PUBLIC_*` 时的服务端回退 |
| `SUPABASE_ANON_KEY` | 同上 |
| `SUPABASE_SERVICE_ROLE_KEY` | 仅服务端特权操作；**永不**暴露到客户端 |
| `SUPABASE_JWT_SECRET` | Auth/JWT；业务代码通常不直接读 |
| `POSTGRES_*` | 直连 Postgres / Prisma / Drizzle；只用 `@supabase/ssr` 时可忽略 |

### 推荐解析逻辑（兼容新旧命名）

部分新文档会写 `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` / `SUPABASE_SECRET_KEY`。代码应**优先读集成实际注入的名字**，并兼容新名：

```ts
export function getSupabaseUrl() {
  return (
    process.env.NEXT_PUBLIC_SUPABASE_URL ??
    process.env.SUPABASE_URL ??
    ""
  );
}

export function getSupabaseAnonKey() {
  return (
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY ??
    process.env.SUPABASE_ANON_KEY ??
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY ??
    process.env.SUPABASE_PUBLISHABLE_KEY ??
    ""
  );
}

export function getSupabaseServiceRoleKey() {
  return (
    process.env.SUPABASE_SERVICE_ROLE_KEY ??
    process.env.SUPABASE_SECRET_KEY ??
    ""
  );
}
```

`@supabase/ssr` 的 browser / server client 都用 **URL + Anon/Publishable Key**。只有 admin 任务才用 Service Role。

## Vercel 系统变量（站点 URL / SEO）

不必手填正式域名时，可用：

| 变量 | 含义 |
| --- | --- |
| `VERCEL_PROJECT_PRODUCTION_URL` | 生产域名（无协议） |
| `VERCEL_URL` | 当前部署域名（无协议） |
| `VERCEL_ENV` | `production` / `preview` / `development` |

示例：

```ts
const siteUrl =
  process.env.NEXT_PUBLIC_SITE_URL ??
  (process.env.VERCEL_PROJECT_PRODUCTION_URL
    ? `https://${process.env.VERCEL_PROJECT_PRODUCTION_URL}`
    : process.env.VERCEL_URL
      ? `https://${process.env.VERCEL_URL}`
      : "http://localhost:3000");
```

## 自定义域名走国内 CDN（阿里云 ESA）回源 Vercel

链路：**用户 → 阿里云 DNS → ESA 边缘节点 → 回源 Vercel**。

### 配置顺序

1. **Vercel**：Project → Settings → Domains 添加自定义域名（如 `co.example.com`），记下真实生产域名 `your-app.vercel.app`
2. **ESA**：DNS → 记录 → 添加 `CNAME`，代理加速开启
   - **记录值 / 源站** 填 `your-app.vercel.app`
   - **回源 HOST** 填自定义域名（须与 Vercel Domains 里绑定的一致）
   - HTTPS 回源，端口 `443`
3. **ESA**：为该域名签发 / 上传边缘证书（对外 HTTPS 由 ESA 终结）
4. **域名 DNS 服务商**：把子域 CNAME 指向 **ESA 生成的专属 CNAME**（如 `xxx.a1.initaf.com`），**不要**指向 `cname.vercel-dns.com`

### ESA 推荐开启的优化项（速度和网络 → 优化）

回源 Vercel 时，以下项建议全部开启（与 ESA 控制台推荐一致）：

| 配置 | 作用 | 说明 |
| --- | --- | --- |
| **Zstd** | 边缘压缩 HTML/静态资源 | 比 Gzip / Brotli 更快；边缘会按 `Accept-Encoding` 协商，一般不会与 Vercel 已压缩响应冲突 |
| **HTTP/2** | 客户端 ↔ ESA | 多路复用，默认应开 |
| **HTTP/2 回源** | ESA ↔ Vercel 源站 | Vercel 支持 HTTP/2 回源，可降低回源连接开销 |
| **HTTP/3 (QUIC)** | 客户端 ↔ ESA | 弱网/移动端更稳；不支持的客户端自动回落 HTTP/2 |

若偶发编码异常或回源报错，再按项排查：先关 **Zstd**，其次关 **HTTP/2 回源**；**HTTP/2** / **HTTP/3** 通常无需关闭。

### 强制 HTTPS 会打断域名验证（重要）

ESA 开着 **强制 HTTPS**（HTTP 一律 308 跳 HTTPS）时，Vercel / Let's Encrypt 的 **HTTP-01 验证**走 `http://<domain>/.well-known/acme-challenge/...`，请求在边缘就被跳走，永远拿不到 challenge 内容，于是 Vercel 侧一直卡在 `Invalid Configuration` / 证书签发失败。

处理方式（任选）：

- **临时关闭强制 HTTPS**，等 Vercel 显示域名有效、证书签发完成后再打开
- 在 ESA 加规则：**放行 `/.well-known/acme-challenge/*` 不做 HTTPS 跳转**（推荐，长期自动续期也不受影响）
- 改用 **TXT 记录** 完成 Vercel 域名归属验证，绕开 HTTP-01

若对外证书由 ESA 提供，Vercel 侧证书长期 Pending 不影响访问，但**域名必须在 Vercel 绑定成功**，否则回源会被 Vercel 拒绝。

### 验证命令

```bash
dig +short CNAME co.example.com          # 应为 ESA 专属 CNAME
curl -sI https://co.example.com/         # 期望 200，且带 x-vercel-id / x-powered-by: Next.js
curl -sI http://co.example.com/          # 强制 HTTPS 时为 308
```

### ESA 相关故障对照

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `308` 跳转到自身，无限循环 | ESA 记录的**源站填成了加速域名本身** | 源站改成 `your-app.vercel.app` |
| Vercel 域名一直 `Invalid Configuration` | 强制 HTTPS 挡掉 HTTP-01 验证 | 放行 `/.well-known/acme-challenge/*` 或临时关闭强制 HTTPS |
| 访问被 308 跳到 `*.vercel.app` | 回源 HOST 是 vercel 域名，且自定义域未在 Vercel 绑定 | 回源 HOST 改自定义域名，并在 Vercel 添加该域名 |
| 返回 `530` | DNS 未精准指向 ESA 专属 CNAME | 核对 DNS 服务商处的 CNAME 值 |
| Auth 邮件链接跳到 `*.vercel.app` | `emailRedirectTo` / Supabase Site URL 用了 Vercel 默认域 | 固定为自定义域名，见下节 |

### Auth 回调必须锁定自定义域名

前置 CDN 后，服务端拿到的 `Host` / `origin` 未必是自定义域名，**不要**用请求头推导 `emailRedirectTo`，改为常量或 `NEXT_PUBLIC_SITE_URL`：

```ts
export function getSiteUrl() {
  const explicit = process.env.NEXT_PUBLIC_SITE_URL?.replace(/\/$/, "");
  if (explicit) return explicit;
  if (process.env.NODE_ENV === "development") return "http://localhost:3000";
  if (process.env.VERCEL_ENV === "production") return CANONICAL_SITE_URL;
  return process.env.VERCEL_URL ? `https://${process.env.VERCEL_URL}` : CANONICAL_SITE_URL;
}
```

同时 Supabase → Authentication → URL Configuration 的 **Site URL** 与 **Redirect URLs** 都填自定义域名。

## Agent 检查清单（部署前）

- [ ] 客户端用 `@supabase/ssr` 的 `createBrowserClient` / `createServerClient`，且读到 `NEXT_PUBLIC_SUPABASE_URL` + `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- [ ] Middleware 在缺少 env 时不硬崩（本地未配时放行或明确报错）
- [ ] `.env*` 已在 `.gitignore`；只提交 `.env.example`
- [ ] `.env.example` 变量名与集成一致（`ANON_KEY` 或同时注明兼容）
- [ ] `supabase/migrations/` 已提交到 Git，Supabase GitHub Integration 已连接正确仓库和 Working directory
- [ ] 已开启 **Deploy to production**，生产分支与仓库一致
- [ ] GitHub 分支保护将 Supabase integration check 设为 required
- [ ] 存在 `/auth/callback`（或项目约定的 OAuth/magic-link 回调）
- [ ] Preview 环境也启用了集成注入的 env（Vercel → Settings → Environment Variables → Environments）
- [ ] 未把 `SERVICE_ROLE_KEY` / `POSTGRES_PASSWORD` 写进 `NEXT_PUBLIC_*`
- [ ] 若前置国内 CDN：源站不是加速域名本身，且强制 HTTPS 放行了 `/.well-known/acme-challenge/*`

## 常见故障

| 现象 | 处理 |
| --- | --- |
| Build / Runtime：`supabaseUrl is required` | 集成变量未勾选当前环境（Production/Preview）；或代码读了错误的 key 名 |
| 登录后无法回跳 | 补全 Supabase Redirect URLs / Site URL |
| Preview 能开站但不能登录 | Preview 环境缺 env，或 Redirect 未含 `*.vercel.app` |
| 注册成功但无 profile 表 | 检查 GitHub Integration 是否连接正确仓库、Working directory 是否正确、是否开启 Deploy to production，以及 Supabase check 日志是否有 migration 失败 |
| 本地与线上 key 不一致 | `vercel env pull .env.local` 重新拉取 |
| 自定义域名验证/证书失败 | CDN 强制 HTTPS 打断 HTTP-01，见「自定义域名走国内 CDN」一节 |

## 与 co-founder 项目的对应关系

参考实现：`src/lib/supabase/env.ts`、`client.ts`、`server.ts`、`middleware.ts`，以及 README「一键部署」一节。Phase 0 只用 Anon Key；`POSTGRES_*` / Service Role 预留即可。

## 输出期望

当用户要求「一键部署 / 接 Vercel Supabase」时，Agent 应：

1. 按上面步骤给出可执行清单  
2. 核对代码 env 读取是否兼容 **本 skill 中的注入列表**  
3. 确认 Vercel Marketplace 与 Supabase GitHub 两个集成都已配置，并让 migration 随生产分支自动部署  
4. 提醒 Auth URL 等默认不会由 GitHub Integration 部署的配置仍需在 Dashboard 设置  
5. 不把 secret 写进前端或提交进 git  

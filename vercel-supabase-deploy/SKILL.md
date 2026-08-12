---
name: vercel-supabase-deploy
description: >-
  Deploys Next.js apps to Vercel with Supabase Marketplace env injection and
  Supabase GitHub Integration for auto migrations. Use when the user asks about
  Vercel deploy, Supabase integration, 一键部署, marketplace env vars,
  PUBLISHABLE_KEY / SECRET_KEY / ANON_KEY, Auth redirect URLs, vercel env pull,
  国内 CDN (阿里云 ESA), Cloudflare 前置 Vercel, ICP 备案, Hobby/Free 成本红线,
  ERR_TOO_MANY_REDIRECTS / 重定向次数过多, ESA 回源规则 / 回源协议 / 回源 Host,
  or low-cost high-performance deploy.
---

# Vercel + Supabase 低成本高性能部署

面向 **Next.js（App Router）+ `@supabase/ssr`**。目标：最低成本上线，同时保持访问性能。

## 强制：先读官方文档，再回答/动手

Skill 只保留决策与易错点。**配额、变量清单、UI 步骤、API 细节以文档为准**；动手前 WebFetch 对应链接核对。

### Supabase

| 场景 | 文档 |
| --- | --- |
| Marketplace env 清单、计费、限制 | https://supabase.com/docs/guides/integrations/vercel-marketplace |
| GitHub Integration / Deploy to production | https://supabase.com/docs/guides/deployment/branching/github-integration |
| 部署总览 | https://supabase.com/docs/guides/deployment |
| 上线清单 | https://supabase.com/docs/guides/deployment/going-into-prod |
| API Keys（publishable / secret） | https://supabase.com/docs/guides/getting-started/api-keys |
| 从 anon / service_role 迁移 | https://supabase.com/docs/guides/getting-started/migrating-to-new-api-keys |
| Next.js + `@supabase/ssr` | https://supabase.com/docs/guides/auth/server-side/nextjs |
| Free 项目 7 天暂停 | https://supabase.com/docs/guides/platform/free-project-pausing |
| 定价 | https://supabase.com/pricing |

### Vercel

| 场景 | 文档 |
| --- | --- |
| 导入 / 部署 Next.js | https://vercel.com/docs/frameworks/full-stack/nextjs |
| Marketplace Storage（含 Supabase） | https://vercel.com/docs/marketplace-storage |
| Supabase 集成页 | https://vercel.com/marketplace/supabase |
| 环境变量 | https://vercel.com/docs/environment-variables |
| 系统环境变量（`VERCEL_URL` 等） | https://vercel.com/docs/environment-variables/system-environment-variables |
| 框架前缀变量（`NEXT_PUBLIC_*`） | https://vercel.com/docs/environment-variables/framework-environment-variables |
| CLI：`vercel link` / `env pull` / 部署 | https://vercel.com/docs/projects/deploy-from-cli |
| 自定义域名 | https://vercel.com/docs/domains |
| SSL / HTTP-01 | https://vercel.com/docs/domains/working-with-ssl |
| 在 Vercel 前放代理 | https://vercel.com/kb/guide/can-i-use-a-proxy-on-top-of-my-vercel-deployment |
| Cloudflare Flexible 死循环 | https://vercel.com/kb/guide/resolve-err-too-many-redirects-when-using-cloudflare-proxy-with-vercel |
| 函数区域（靠近数据库） | https://vercel.com/docs/functions/configuring-functions/region |
| Functions 用量计价 | https://vercel.com/docs/functions/usage-and-pricing |
| ISR / 缓存 | https://vercel.com/docs/incremental-static-regeneration |
| Image Optimization 限额 | https://vercel.com/docs/image-optimization/limits-and-pricing |
| Hobby 配额 | https://vercel.com/docs/plans/hobby |
| Fair Use / 商用禁令 | https://vercel.com/docs/limits/fair-use-guidelines |
| Pro 套餐 | https://vercel.com/docs/plans/pro |

### CDN（国内 / 海外前置时）

| 场景 | 文档 |
| --- | --- |
| ESA 备案与加速区域 | https://help.aliyun.com/zh/edge-security-acceleration/esa/product-overview/limits-on-using-esa |
| ESA 套餐 | https://help.aliyun.com/zh/edge-security-acceleration/esa/product-overview/package-function-comparison |
| ESA 状态码（含 530） | https://help.aliyun.com/zh/edge-security-acceleration/esa/support/http-status-code-description |
| ESA 回源 Host | https://help.aliyun.com/zh/edge-security-acceleration/esa/user-guide/origin-fetch-host |
| ESA 回源协议和端口 | https://help.aliyun.com/zh/edge-security-acceleration/esa/user-guide/back-to-source-protocols-and-ports |
| ESA 强制 HTTPS | https://help.aliyun.com/zh/edge-security-acceleration/esa/user-guide/https-application-configuration |
| Cloudflare China Network（仅 Enterprise） | https://www.cloudflare.com/zh-cn/application-services/products/china-network/ |

## 两个集成不要混

| 集成 | 入口 | 职责 |
| --- | --- | --- |
| **Vercel Marketplace → Supabase** | Vercel → Integrations | 创建/关联项目，注入连接 env |
| **Supabase → GitHub Integration** | Supabase → Integrations | 生产分支 push 后跑 migrations；部署 `config.toml` 里的 Edge Functions / Storage |

完整一键部署 = 两个都开。迁移走 GitHub Integration，勿用 SQL Editor / 本地 `db push` 当日常流程。

**Marketplace Install vs Connect Account**：Install 只能新建且账单并入 Vercel、无 Supabase Custom Domains；已有项目走详情页 `...` → Connect Account（env 常需手补）。细节读 vercel-marketplace 文档。

## 部署骨架

1. 导入仓库到 Vercel → 读 Next.js / CLI 文档
2. 装 Marketplace Supabase → **WebFetch marketplace 文档核对当前注入变量名**
3. 连 GitHub Integration（同仓库、Working directory、Deploy to production）
4. Push → Vercel 部署 + Supabase 迁移
5. Dashboard 配 Auth Site URL / Redirect URLs（GitHub Integration **不部署** Auth）
6. 可选：`vercel link` + `vercel env pull .env.local`

## 只记这些坑（其余读文档）

- **Key 名**：集成注入 `PUBLISHABLE_KEY` / `SECRET_KEY`（+ `NEXT_PUBLIC_*`），不是旧 `ANON_KEY` / `SERVICE_ROLE_KEY`。优先新名再回退旧名。Secret 永不进 `NEXT_PUBLIC_*`。
- **成本**：Hobby 禁止一切商业用途 → Pro（Fair Use）；Supabase Free 7 天无活动暂停 → 探活；配额数字读 Hobby / Pricing 文档。
- **性能**：函数区域靠近 Supabase（regions 文档）> ISR/缓存 > pooler 连接串 > 控制 `next/image` 转换。
- **`@supabase/ssr`**：`getAll`/`setAll`；middleware 正确刷新会话。读 Supabase Next.js 文档。

## 前置 CDN

| 用户 | 方案 |
| --- | --- |
| 海外为主 | **不加**，直连 Vercel |
| 海外 + WAF/DDoS/省带宽 | Cloudflare Free |
| 中国大陆为主 | 阿里云 ESA（可能要 ICP 备案） |
| 两边都有 | DNS 分线路：国内 ESA，海外 CF 或直连 |

前置代理共同硬规则（读 Vercel proxy / SSL 文档）：

- 放行 `/.well-known/acme-challenge/*` 与 `/.well-known/vercel/*`（不跳转、不缓存、不 challenge）
- 透传 `Host`；`_vercel` TXT ≠ 证书签发
- Auth 回调用常量 / `NEXT_PUBLIC_SITE_URL`，勿从请求头猜

**ESA**：源站 `*.vercel.app`，回源 HOST = 自定义域；DNS → ESA 专属 CNAME；根域+www 都配；强制 HTTPS 豁免 acme-challenge。

**Cloudflare**：CNAME → `cname.vercel-dns.com`；SSL = **Full (Strict)**（Flexible 必死循环）。

### 坑：`ERR_TOO_MANY_REDIRECTS` / 「重定向次数过多」（ESA 前置 Vercel）

**现象**：浏览器打开自定义域（如 `amazon.airankone.com`）报 `ERR_TOO_MANY_REDIRECTS`。清 Cookie 治标不治本。

**根因（最常见）**：ESA **HTTP 回源**（或等价于 Cloudflare Flexible）→ Vercel 强制跳 HTTPS → 浏览器再打 ESA → 死循环：

1. 浏览器 `HTTPS` → ESA  
2. ESA `HTTP` → `*.vercel.app`  
3. Vercel `301/302` → `HTTPS` 同域  
4. 回到 1  

**次常见**：www/apex 互跳；ESA 与源站两边都强制 HTTPS 但中间协议仍是 HTTP；应用按错误的 `x-forwarded-proto` 再跳一次。

**改哪里（阿里云 ESA 控制台）**：

1. 进入 **站点管理** → 目标站点（如 `airankone.com`）  
2. 左侧 **规则 → 回源规则**（此处管理各子域回源，例：`amazon.airankone.com -> vercel`、`co.airankone.com -> vercel`）  
3. 打开对应主机名规则（或 **新增规则**），核对并改为：  
   - **匹配**：主机名 = 自定义子域（如 `amazon.airankone.com`）  
   - **源站**：`xxx.vercel.app`  
   - **回源协议**：**HTTPS**（443），禁止仅 HTTP 回源  
   - **回源 Host / SNI**：自定义域本身（如 `amazon.airankone.com`），不要填错、不要填 IP  
4. 再查同页邻接配置：**规则 → HTTPS 规则**（强制 HTTPS 只在边缘开一处；证书生效后再开）  
5. Vercel Project Domains 已添加该自定义域；不要再配冲突跳转  

官方细节以 WebFetch「ESA 回源 Host / 回源协议和端口 / 强制 HTTPS」三篇为准。

**自测**：无痕打开 `https://自定义域`；响应不应无限 `location: https://...`；`/.well-known/vercel/` 不被拦。

## Agent 行为

1. 变量名 / 套餐 / 步骤 → **先 WebFetch 上表文档**，再给清单  
2. 核对两个集成 + Auth URL Dashboard 手配  
3. 主动点成本红线与函数区域  
4. 国内 CDN 先问备案与用户地域  
5. 用户报 `ERR_TOO_MANY_REDIRECTS` / ESA CNAME 后打不开 → 先查 **规则 → 回源规则** 是否 HTTPS 回源 + Host/SNI  
6. 不把 secret 写进前端或 git  

## 检查清单

- [ ] env：URL + publishable（兼容旧 anon）；无 secret 进 `NEXT_PUBLIC_*`
- [ ] `@supabase/ssr` cookie / middleware 正确
- [ ] GitHub Integration：仓库 / Working directory / Deploy to production
- [ ] Auth Site URL + Redirect URLs
- [ ] 函数区域靠近 Supabase
- [ ] 商业 → Vercel Pro；Free Supabase → 探活
- [ ] 前置 CDN：备案（如需）+ `/.well-known` 放行；CF 用 Full (Strict)
- [ ] ESA 前置 Vercel：回源规则里 **HTTPS 回源** + Host/SNI=自定义域（防 `ERR_TOO_MANY_REDIRECTS`）
- [ ] `.env*` 在 gitignore

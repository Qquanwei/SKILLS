# SKILLS

个人 / 团队用的 Cursor Agent Skills 合集。

## 一键安装

```bash
npx skills add Qquanwei/SKILLS -g -y
```

安装全部 skill 到本机（全局，所有项目可用）。

只装某一个：

```bash
npx skills add Qquanwei/SKILLS --skill vercel-supabase-deploy -g -y
```

装到当前项目（写入 `.agents/skills` / `.cursor/skills` 等，可随仓库共享）：

```bash
npx skills add Qquanwei/SKILLS -y
```

更新：

```bash
npx skills update -g -y
```

查看已安装：

```bash
npx skills list -g
```

卸载：

```bash
npx skills remove vercel-supabase-deploy -g -y
```

> 需要 Node.js。首次运行会通过 `npx` 拉取 [skills](https://skills.sh/) CLI。

## 包含的 Skills

| Skill | 用途 |
| --- | --- |
| [`vercel-supabase-deploy`](./vercel-supabase-deploy/) | Next.js → Vercel + Supabase 一键部署：Marketplace env、GitHub 自动迁移、Hobby/Free 成本红线、国内 ESA / 海外 Cloudflare 前置 |

## 手动安装（不用 CLI）

```bash
# 全局（Cursor）
mkdir -p ~/.cursor/skills
git clone --depth 1 https://github.com/Qquanwei/SKILLS.git /tmp/Qquanwei-SKILLS
cp -R /tmp/Qquanwei-SKILLS/vercel-supabase-deploy ~/.cursor/skills/
```

或项目级：

```bash
mkdir -p .cursor/skills
cp -R /path/to/SKILLS/vercel-supabase-deploy .cursor/skills/
```

## 使用

在 Cursor 对话里提到相关意图即可触发（例如「一键部署到 Vercel + Supabase」「接国内 CDN」），或直接说 skill 名 `vercel-supabase-deploy`。

# 自动部署配置指南

## 一、GitHub Actions（已完成）

workflow 文件：`.github/workflows/deploy.yml`
- 触发：push 到 main 分支，或手动 `workflow_dispatch`
- 动作：检查 public/ 资源 → 用 wrangler 部署到 Cloudflare Workers
- 需要两个 Secret：`CLOUDFLARE_API_TOKEN`、`CLOUDFLARE_ACCOUNT_ID`

## 二、创建 Cloudflare API Token（需手动操作）

OAuth 登录的 token 权限不足，无法通过 API 自动创建。请按以下步骤手动创建：

### 步骤

1. 打开 **https://dash.cloudflare.com/profile/api-tokens**
2. 点击 **Create Token**
3. 选择 **Create Custom Token**（最下面的自定义 token）
4. 填写：

   | 项 | 值 |
   |---|---|
   | Token name | `github-actions-deploy` |
   | Permissions | `Account` → `Workers Scripts` → `Edit` |
   |  | `Account` → `Workers KV Storage` → `Edit`（可选） |
   |  | `Account` → `Account Settings` → `Read` |
   | Account Resources | `Include` → `Voidsec@126.com's Account` |
   | Zone Resources | `All zones`（或留空） |
   | TTL | 留空（永不过期）或按需设置 |

5. 点击 **Continue to summary** → **Create Token**
6. **复制生成的 token**（只显示一次）

### 配置到 GitHub

**方式 A：一键脚本（推荐）**

用记事本打开 `setup-cf-token.bat`，把 token 粘贴到 `CF_TOKEN=` 后面，保存后双击运行。

**方式 B：手动命令**

```powershell
gh secret set CLOUDFLARE_API_TOKEN --repo zzy00747/linux-security-review --body "你的token"
gh secret set CLOUDFLARE_ACCOUNT_ID --repo zzy00747/linux-security-review --body "e110f7a454ea6979700198a3cc6ab306"
```

**方式 C：网页**

仓库 → Settings → Secrets and variables → Actions → New repository secret

## 三、验证

```powershell
# 手动触发一次
gh workflow run deploy.yml --repo zzy00747/linux-security-review

# 查看运行状态
gh run list --repo zzy00747/linux-security-review --limit 3

# 看详细日志
gh run view --repo zzy00747/linux-security-review --log
```

## 四、日常更新流程

```powershell
cd C:\Users\Administrator\Desktop\czwslide\EPA

# 1. 重新生成资料（如果课件有更新）
python F:\Test\finalexam\build_index.py

# 2. 同步到 public/
Copy-Item *.html public\ -Force

# 3. 提交推送（自动触发部署）
git add -A
git commit -m "update: 更新复习资料"
git push
```

推送后约 1–2 分钟，GitHub Actions 自动完成部署。

## 五、故障排查

| 现象 | 原因 | 解决 |
|---|---|---|
| `Authentication error [code: 10000]` | API Token 错误或权限不足 | 重新创建 token，确认含 `Workers Scripts:Edit` |
| `Account ID not found` | ACCOUNT_ID 错误 | 用 `npx wrangler whoami` 核对 |
| workflow 不触发 | 分支不是 main | 检查 `on.push.branches` |
| 资源检查失败 | public/ 缺文件 | 先跑 `Copy-Item *.html public\ -Force` |

## 六、本地直接部署（不依赖 Actions）

```powershell
cd C:\Users\Administrator\Desktop\czwslide\EPA
npx wrangler deploy
```

OAuth 凭证已在 `%APPDATA%\xdg.config\.wrangler\config\default.toml`，本地部署无需额外配置。

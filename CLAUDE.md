# 灵感收集器 — 项目上下文

## 项目简介

设计灵感收集 app，保存设计参考图 + 分析文字 + 标签，支持 tag 筛选。

**仓库**：33-huang/design-inspiration（**公开**）  
**架构**：单 HTML 文件 + GitHub API（方案 2），GitHub Pages 部署  
**本地路径**：`/Users/dear33/33/design-inspiration/`

---

## 文件结构

| 文件 | 内容 |
|------|------|
| `index.html` | 主 app（单文件，含全部 JS/CSS） |
| `data.json` | 条目数据 |
| `tags.json` | 标签配置 |
| `images/` | 图片文件夹（现有 15 张，当前存在 GitHub 仓库，待迁移至 R2） |

---

## 数据结构

### data.json

```json
[
  {
    "id": 1,
    "title": "涂鸦风格海报",
    "analysis": "分析文字",
    "archive": "Eagle → 文件夹路径",
    "link": "https://...",
    "createdAt": "2025-04-25",
    "images": ["images/1_0.jpg"],
    "tags": ["t1", "t2"]
  }
]
```

- `images` 是数组，每条可以有多张图片
- 图片目前存相对路径，**迁移 R2 后改为 `{ "provider": "r2", "path": "images/1_0.jpg" }` 结构**

### tags.json

标签配置，id 为时间戳字符串（如 `t1779353638138`）。

---

## 待办事项

### 图片迁移至 Cloudflare R2

> **前置条件（用户手动完成）**：登录 [dash.cloudflare.com](https://dash.cloudflare.com)，左侧进入 R2 Object Storage，点击开启 R2（需绑定信用卡验证，不扣费）。

Codex 负责完成以下步骤：

**Cloudflare 账号信息**：存放在本地私有文件 `/Users/dear33/33/web-app-方案选择指南.md`（含 Account ID、API Token、GitHub Token）。

**步骤：**

- [ ] 创建 R2 bucket：`design-inspiration-images`（用 Cloudflare API，凭证见方案指南）
- [ ] 开启 bucket 公开读取（r2.dev 公开访问）
- [ ] 创建 Cloudflare Worker `design-inspiration-upload`，绑定 R2 bucket，处理图片上传（凭证留在 Worker 端）
- [ ] 将仓库 `images/` 文件夹现有 15 张图片通过 GitHub API 下载，批量上传至 R2
- [ ] 更新 `data.json`：images 字段从相对路径改为 `{ "provider": "r2", "path": "images/xxx.jpg" }`
- [ ] 更新 `index.html`：
  - 新上传图片调用 Worker → 存入 R2
  - 展示图片用 R2 公开 URL（通过统一的 `getImageUrl(img)` 函数处理 provider）
  - 前端压缩：canvas，最大宽 1600px，JPEG quality 0.85 起逐步降至 0.6，目标 <1MB
- [ ] 推送更新后的代码到 GitHub（触发 GitHub Pages 重新部署）
- [ ] 确认线上图片正常显示后，删除仓库中的 `images/` 文件夹

**注意**：此仓库是公开仓库，迁移前图片可通过 raw.githubusercontent.com 直接访问，迁移后改为 R2 公开 URL。两者都不需要鉴权，`<img src>` 可以直接用。

---

## 部署

**线上地址**：https://33-huang.github.io/design-inspiration/  
**方式**：GitHub Pages，source 为 main 分支根目录

每次改完代码后，push 到 main 即自动触发部署，通常 1 分钟内生效：

```bash
cd /Users/dear33/33/design-inspiration

git add index.html          # 或其他改动的文件
git commit -m "描述改动内容"
git push origin main
```

无需 wrangler，无需手动触发，push 完刷新线上地址即可验证。

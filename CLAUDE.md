# 灵感收集器 — 项目上下文

## 项目简介

设计灵感收集 app，保存设计参考图 + 分析文字 + 标签，支持 tag 筛选。

**仓库**：33-huang/design-inspiration（**公开**）  
**架构**：单 HTML 文件 + GitHub API，GitHub Pages 部署  
**本地路径**：`/Users/dear33/33/design-inspiration/`

---

## 文件结构

| 文件 | 内容 |
|------|------|
| `index.html` | 主 app（单文件，含全部 JS/CSS） |
| `data.json` | 条目数据 |
| `tags.json` | 标签配置 |
| `images/` | 旧图片文件夹（15 张历史图片，存在 GitHub 仓库，向后兼容保留） |

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
    "images": [{ "provider": "firebase", "path": "images/1234567890_0.jpg" }],
    "tags": ["t1", "t2"]
  }
]
```

- `images` 是数组，每条可以有多张图片
- 新图片存 `{ "provider": "firebase", "path": "images/..." }` 对象格式
- 旧图片（历史数据）存相对路径字符串如 `"images/1_0.jpg"`，`resolveImg()` 自动识别，向后兼容

### tags.json

标签配置，id 为时间戳字符串（如 `t1779353638138`）。

---

## 图片存储

**当前方案：Firebase Storage（已实现）**

- 服务：Firebase Storage，项目 `design-inspiration-43122`（独立项目，与 tokyo-wandering 完全分开）
- Bucket：`design-inspiration-43122.firebasestorage.app`
- 上传：客户端直接上传（Firebase compat SDK via CDN）
- 访问：公开 URL 格式：`https://firebasestorage.googleapis.com/v0/b/design-inspiration-43122.firebasestorage.app/o/{encoded_path}?alt=media`
- 压缩：上传前用 canvas 压缩，最大宽 1600px，JPEG quality 0.85→0.6，目标 <1MB
- 展示：`resolveImg()` 函数根据 provider 构造 URL，同时兼容旧的 raw.githubusercontent.com 路径

**注意**：Firebase Storage 现在是 test mode（30 天到期，约 2026 年 7 月初），到期前需要更新规则：
```
// 当前规则（test mode）
allow read, write: if request.time < timestamp.date(2026, 7, 7);

// 到期后改为：
allow read: if true;
allow write: if false;
```

Firebase SDK 初始化配置（index.html 中）：
```js
firebase.initializeApp({
  apiKey: 'AIzaSyAg89DOubYTlURNbdXH4-cXmfHY5OlOpY0',
  projectId: 'design-inspiration-43122',
  storageBucket: 'design-inspiration-43122.firebasestorage.app',
  appId: '1:781942393973:web:685c6de87c9c5f1092fc3c'
});
```

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

---

## 凭证

- GitHub Token：存在浏览器 localStorage，运行时从设置页读取，不硬编码在代码里
- Firebase API Key：硬编码在 index.html（公开仓库可接受，Storage 规则控制写权限）

完整账号信息见 `/Users/dear33/33/web-app-方案选择指南.md`。

---

## 待办事项

### Firebase Storage 规则更新（约 2026 年 7 月初到期）

- [ ] 登录 Firebase Console → design-inspiration-43122 项目 → Storage → Rules，将 test mode 规则改为：
  ```
  allow read: if true;
  allow write: if false;
  ```

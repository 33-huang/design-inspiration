# 错误记录

## BUG-001 编辑保存后图片刷新消失

**现象**：在编辑表单上传图片并保存，页面即时显示正常，但刷新后图片消失。

**原因**：`loadData()` 和 `getWriteBase()` 用了两个来源读同一份文件：
- 数据内容：从 `raw.githubusercontent.com`（有 CDN 缓存，最多延迟数分钟）
- SHA：从 GitHub API（实时）

`getWriteBase()` 在写入前读数据，拿到的是旧 CDN 缓存（不包含刚保存的条目），导致保存时用旧数据 + 新 SHA 覆盖了正确内容。`loadData()` 刷新时同样读到旧缓存，图片路径丢失。

**修复**：将 `loadData()`、`loadTagsData()`、`getWriteBase()` 全部改为通过 GitHub API（`ghGet()`）读取，同一次请求同时拿内容和 SHA，保证一致性。`raw.githubusercontent.com` 只用于老格式图片的展示 URL。

**关键原则**：凡是"读后写"操作（先读 SHA 再写），必须从 GitHub API 读，不能从 CDN 读。

---

## BUG-002 手机端无 token 看不到任何内容

**现象**：手机浏览器（未配置 GitHub Token）打开 app，内容列表空白或显示"配置未找到"。

**原因**：`loadItems()` 里有 `hasSettings()` 检查，要求 token + owner + repo 全部配置才会加载数据。但这是公开仓库，读取数据不需要 token。

**修复**：
- 将 owner/repo/branch 硬编码为常量（`REPO_OWNER`、`REPO_NAME`、`REPO_BRANCH`）
- `ghGet()` 改为：有 token 则带 Authorization header，无 token 则不带（公开仓库支持匿名读）
- 移除 `loadItems()` 里的 `hasSettings()` 拦截
- 写操作（`ghPut`、`ghPutBinary`）保留 token 检查

---

## BUG-003 手机端 Firebase 图片不显示（中国大陆无 VPN 环境）

**现象**：电脑端（开 VPN）图片正常，手机端（未开 VPN）图片不显示，即使数据加载成功。

**原因**：Firebase Storage 是 Google 服务，在中国大陆需要 VPN 才能访问。图片 URL 格式 `https://firebasestorage.googleapis.com/...` 在无 VPN 的手机网络下无法连通。

**解决**：使用 VPN 后恢复正常。如需在无 VPN 环境使用，需将图片存储迁移至 GitHub 仓库（旧图片 id=1-5 即采用此方案，已在国内正常显示）。

---

## 历史 BUG（已修复，来自前次对话）

### f-archive / edit-archive null 引用

`document.getElementById('f-archive')` 和 `document.getElementById('edit-archive')` 返回 null，HTML 中不存在这两个元素，导致添加和编辑表单提交报错。修复：移除代码中所有对这两个元素的引用。

### 添加失败后按钮卡在"添加中…"

提交失败后 `_pendingImages` 未清空，再次提交时重复上传已失败的旧数据。修复：在 catch 块中清空 `_pendingImages` 并重置 UI。

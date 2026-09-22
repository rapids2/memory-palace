# 记忆宫殿智能复习中枢 v4.0 — 自托管部署说明

一个纯静态单页应用，**没有任何外部依赖**（不加载 CDN、不加载网络字体、不调用后端 API），
把这几个文件原样放到任意静态托管上就能用。

## 文件清单

| 文件 | 说明 | 能不能删 |
|---|---|---|
| `index.html` | 应用本体，HTML / CSS / JS 全在里面 | 不能 |
| `manifest.webmanifest` | 让手机可以「添加到主屏幕」当 App 用 | 可以（会失去安装能力） |
| `icon-192.png` / `icon-512.png` | 应用图标 | 同上 |
| `apple-touch-icon.png` | iPhone / iPad 主屏图标 | 同上 |
| `.gitignore` | 防止把含有你复习数据的备份 JSON 误传到仓库 | 建议保留 |

---

## 方案 A：GitHub Pages（推荐，免费）

### 1. 建仓库

到 <https://github.com/new>，仓库名随意（例如 `memory-palace`），
**Visibility 选 Public**，勾不勾 README 都行 → Create repository。

> **为什么必须 Public？** GitHub 官方说明：
> "GitHub Pages is available in public repositories with GitHub Free and GitHub Free for organizations,
> and in public and private repositories with GitHub Pro, GitHub Team, GitHub Enterprise Cloud, and GitHub Enterprise Server."
> 也就是说免费账号的私有仓库开不了 Pages。想要私有见下面的方案 B。
>
> **Public 会泄露我的单词和地图吗？不会。** 仓库里只有应用代码，
> 你录入的地图图片、桩点坐标、复习进度全部存在你自己浏览器里，从不上传。
> 唯一要注意的是**别把导出的备份 JSON 提交进去**——`.gitignore` 已经替你挡住了。

### 2. 上传文件

**网页操作（不用装 Git）**

仓库页面 → `Add file` → `Upload files` → 把本文件夹里**全部文件**拖进去 → `Commit changes`。

> `.gitignore` 是隐藏文件，Windows 文件资源管理器可能看不到：
> 查看 → 勾选「隐藏的项目」即可。漏传它不影响运行。

**或者用命令行**

```bash
cd <这个文件夹>
git init
git add .
git commit -m "记忆宫殿复习中枢 v4.0"
git branch -M main
git remote add origin https://github.com/<你的用户名>/memory-palace.git
git push -u origin main
```

### 3. 打开 Pages

仓库 → `Settings` → 左侧 `Pages` →
Source 选 **Deploy from a branch** → Branch 选 `main`、目录 `/ (root)` → `Save`。

等 1–2 分钟，页面顶部会出现网址：

```
https://<你的用户名>.github.io/memory-palace/
```

这个网址永久有效，只要仓库还在。

---

## 方案 B：Cloudflare Pages（想要私有 / 不想公开代码）

1. 注册 <https://dash.cloudflare.com>（免费）
2. `Workers & Pages` → `Create` → `Pages` → **Upload assets**（直接拖文件夹，不需要 Git）
3. 起个项目名 → Deploy，得到 `https://<项目名>.pages.dev`
4. 想加访问密码：项目 → `Settings` → 接入 **Cloudflare Access**（免费额度 50 用户），
   设成只有你的邮箱能打开

Netlify（<https://app.netlify.com/drop>）也可以直接拖文件夹上传，但密码保护是付费功能。

---

## 部署之后要知道的几件事

### 数据存在哪

- 存在**打开这个网址的那个浏览器里**（localStorage 存文字与进度，IndexedDB 存地图图片）
- 服务器上没有你的任何数据，换台电脑打开是空的
- 搬家的办法：旧设备点「💾 导出完整备份 (含图)」→ 新设备点「📂 导入并合并」
  （导入时会按最后复习时间比对，旧备份不会盖掉新进度）

### ⚠️ 网址就是数据的身份证

浏览器按**域名**隔离存储。所以：

- `username.github.io/memory-palace/` 和 `xxx.pages.dev` 是**两套完全独立的数据**
- 换托管平台、改仓库名、换自定义域名，都等于换了一个新的空应用
- **动网址之前，先导出一份完整备份**

### 更新到新版本

覆盖 `index.html` 重新提交即可，**数据不受影响**（域名没变 = 存储没变）。
提交后如果还看到旧版，按 `Ctrl + F5` 强制刷新一次。

### 为什么托管比双击本地文件更好

`file://` 协议下 Chrome 通常会禁用 IndexedDB，地图图片功能可能直接失效；
桌面通知也需要安全上下文。HTTPS 托管后这两项都稳定可用。

### 手机上当 App 用

用手机浏览器打开网址：

- **Android / Chrome**：菜单 → 「安装应用」或「添加到主屏幕」
- **iPhone / Safari**：分享按钮 → 「添加到主屏幕」

装好后是独立窗口、有图标、无地址栏。
注意手机上是独立的一份数据，要先导入一次备份 JSON。

### 备份习惯

应用会在超过 7 天没导出时在顶部提醒你。
建议每周导一次完整备份，扔进网盘。清浏览器数据、换设备、升级系统，都可能让本地存储消失。

---

## 已知限制

- 数据不跨设备自动同步（需要后端才能做到）
- 间隔重复的调度单位是**整座地图**，配合「生疏标记」加权，而不是每个单词独立调度
- 桌面通知只在页面开着时才会发送；页面标题栏的 `(N)` 角标是更可靠的提醒

# Label Pixiv Bookmarks（Pixiv 收藏夹自动标签与管理工具）

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Tampermonkey](https://img.shields.io/badge/Tampermonkey-v5.22-green.svg)](https://www.tampermonkey.net/)

一个运行在 **Tampermonkey（篡改猴）** 上的 Pixiv 收藏夹增强脚本，支持自动为收藏夹内的 **插画・漫画（Artworks）** 与 **小说（Novels）** 批量打上分类标签，并提供多条件标签搜索、同义词词典映射、失效作品清理、收藏夹备份与恢复等功能。

> 本项目 Fork 自 [Ziqing19/LabelPixivBookmarks](https://github.com/Ziqing19/LabelPixivBookmarks)，针对新版 Pixiv 前端架构进行了全面重写与性能优化，并新增了对小说收藏夹的完整支持。

---

## ✨ 核心特性

1. **适配新版 Pixiv UI**
   - 移除旧版写死的哈希 CSS 类名，改用语义化选择器、URL 状态解析以及「React Fiber 树提取 + Pixiv Web AJAX API」双重兜底机制，支持 Pixiv 单页应用（SPA）无缝切换。
2. **插画・漫画 & 小说双收藏夹全功能支持**
   - 在插画收藏夹（`/users/<uid>/bookmarks/artworks`）与小说收藏夹（`/users/<uid>/bookmarks/novels`）页面均可直接使用。
   - 支持**跨分类共享标签**（开启后，给小说自动打标签时可同时匹配插画收藏夹中的已有标签，反之亦然）。
3. **大幅提速的批量打标引擎（v5.22）**
   - **$O(1)$ 哈希索引匹配**：在开始打标前将用户标签与同义词词典预编译为内存哈希表，即使导入上千条同义词也能瞬间完成匹配。
   - **网络请求智能合并（Batching）**：每页（100 个作品）自动计算最优批量打包策略，将相同标签的请求合并发送，使网络请求次数减少 70%~90%，**打标速度提升 5~15 倍**，同时更不容易触发 Pixiv 的 `429 Too Many Requests` 限制。
4. **同义词词典（Synonym Dictionary）**
   - 支持自定义 JSON 同义词词典（目标标签 $\rightarrow$ 多语言/简称同义词列表），解决画师/作者打标语言不统一的问题。
   - 原生支持 `角色名(作品名)` 格式标签的自动拆解与按作品归类排序（兼容中/日文半角 `()` 与全角 `（）`）。
5. **收藏夹多维搜索与管理**
   - 支持结合同义词词典在个人收藏夹中进行多关键词交集搜索、排除搜索（`!关键词`）及按标题/作者/作品标签/用户标签高级检索。
   - 支持批量重命名标签、批量删除标签、一键清理已删除/隐藏的失效收藏、导出备份与跨账号导入收藏夹。

---

## 🚀 安装方法

1. 请先在浏览器中安装 **[Tampermonkey（篡改猴）](https://www.tampermonkey.net/)** 扩展（注：不兼容 Greasemonkey）。
2. **一键安装脚本**：
   - 👉 **[点击此处直接安装 `index.js`](https://raw.githubusercontent.com/Lorentz114514/LabelPixivBookmarks/HEAD/index.js)**
   - 或在仓库中点开 [`index.js`](./index.js)，点击右上角的 **Raw** 按钮，Tampermonkey 会自动弹出安装界面；也可以直接复制 `index.js` 的内容到 Tampermonkey 新建脚本中保存。
3. 安装后打开你的 Pixiv 个人收藏夹页面（如 `https://www.pixiv.net/users/你的UID/bookmarks/artworks` 或 `.../bookmarks/novels`），页面上方即可看到注入的功能按钮。

---

## 📖 功能使用说明

### 1. 自动添加标签（Label Bookmarks）
- 脚本会对比 **作品自带的 Pixiv 标签** 与 **你已有的收藏夹标签（或同义词词典中的目标标签）**，自动为作品打上所有匹配的标签。
- **自动标签范围**：
  - **作品类型**：可选择 `插画・漫画 / Artworks` 或 `小说 / Novels`。
  - **公开类型**：可选择 `公开收藏 / Public` 或 `私密收藏 / Private`。
  - **目标标签**：默认处理 `未分类作品 / Uncategorized Only`，也可选择重新扫描 `全部作品 / All Works` 或某一特定标签下的作品。
- **高级设置（Advanced）**：
  - 支持自动为 R-18 作品打上 `#R-18`、为全年龄作品打上 `#SFW`、为 AI 生成作品打上 `#AI`。
  - 支持将作者名与作者 UID 视为作品标签进行匹配。
  - 支持开启/关闭「跨分类共享标签」（插画与小说共享已有标签池）。

### 2. 同义词词典（Synonym Dict）
当画师或小说作者使用了外文名、假名或简称时，可以通过同义词词典将它们统一映射到你的中文标签下：
- **词典 JSON 格式示例**：
  ```json
  {
    "Fate": ["FGO", "Fate/GrandOrder", "Fate/stay night", "フェイト"],
    "新世纪福音战士": ["新世紀エヴァンゲリオン", "エヴァンゲリオン", "EVA", "Evangelion"],
    "明日香(新世纪福音战士)": ["惣流・アスカ・ラングレー", "式波・アスカ・ラングレー", "Asuka Langley"]
  }
  ```
- **导入与导出**：
  - 在 **同义词词典 -> 加载词典** 中选择本地 `.json` 文件即可导入（词典会自动保存在本地浏览器的 Tampermonkey 存储中）。
  - 在 **同义词词典 -> 编辑词典** 中可随时增删改单个标签的同义词，或点击 **导出词典** 备份到本地磁盘。

### 3. 搜索收藏标签（Search Bookmarks）
- 输入关键词（空格分隔代表「且」，关键词前加 `!` 代表「排除该关键词」），脚本会结合你的同义词词典扫描收藏夹并展示匹配的作品卡片。
- 点击搜索框右侧的切换按钮可进入**高级搜索模式**，分别指定每个关键词仅匹配「标题」「作者」「作品标签」或「用户标签」。

### 4. 其他扩展功能（在篡改猴菜单中开启）
点击浏览器右上角的 Tampermonkey 图标，可在脚本菜单中按需开启以下扩展功能：
- **显示用户标签 / Show User Tags**：在收藏夹缩略图/小说卡片下方直接显示你已经打上的标签。
- **启用随机作品 / Enable Shuffled Works**：从收藏夹中随机抽取作品浏览。
- **显示其他功能 / Enable Additional Functions**：
  - 批量切换某个标签下所有作品的公开/私密状态
  - 批量重命名或删除某个收藏标签
  - 查询并批量清理已被作者删除或隐藏的失效作品
  - 导出备份整个收藏夹为 JSON，或在其它账号上导入恢复收藏夹
  - 开启/关闭替换原生标签选择对话框（按拼音/五十音及作品分组排序）
  - 开启/关闭**加速模式（Turbo Mode）**

---

## 📄 开源协议 (License)

本项目基于 [MIT License](./LICENSE) 开源。
- 原项目作者：[Ziqing19 (philimao)](https://github.com/Ziqing19/LabelPixivBookmarks)
- 新版 UI 修复、小说收藏夹支持与批处理提速优化：[Lorentz114514](https://github.com/Lorentz114514/LabelPixivBookmarks)

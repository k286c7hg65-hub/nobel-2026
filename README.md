# 2026 诺贝尔奖中文速览页（Riemann007/nobel-2026）

线上：**https://k286c7hg65-hub.github.io/nobel-2026/**
仓库：`Riemann007/nobel-2026`（Pages: main / root）

## 这是什么

一页响应式（手机 / 平板 / 桌面）中文科普页，逐日跟踪 2026 年 6 个诺贝尔奖。
面向对象：中学生（牧之，初二）。取材纪律：**只用诺贝尔奖官方一手来源**，未揭晓的奖项留白，禁止推断填充。

## 文件

| 文件 | 用途 |
|---|---|
| `build.py` | **页面生成器（改内容改它）**：内容在文件内的数据结构里，模板在 `style.css` |
| `style.css` | 全部样式（v3 抽取 + v4 新增组件） |
| `index.html` | 产物（单文件；图片走本地 `assets/`，运行时零第三方请求） |
| `assets/*.webp` + `sizes.json` | 26 张官方原图（等比缩放后的 WebP）+ 宽高表（写进 `<img width/height>` 防抖动） |
| `download_assets.py` / `optimize_assets.py` | 从 nobelprize.org 拉官方原图 → 转 WebP（不改内容不裁切） |
| `extract_assets.py` | 扫官方页面，抽图片 / 图注 / 署名 / 视频链接 |
| `deploy.py` | 部署（Git Data API 原子提交）并启用 Pages、**逐个文件比对线上 md5** |
| `verify.py` | 本地三视口：控制台 / 外部请求 / 横向溢出 |
| `live_check.py` | 线上同口径复核 + 图片解码 + 锚点落点 + 出回群截图 |
| `contrast_check.py` | 对比度实测（背景逐层合成 + opacity 连乘） |
| `img_check.py` / `text_clip_check.py` | 图片解码检查 / 文字被容器截断检查 |
| `review4.py` | 视觉复核截图（按锚点拍视口） |
| `check_new.py` | 拉官方 API 与 `state.json` 比对，只报「新增揭晓」；退出码 10 = 有新增 |
| `harvest_nobel.py` | 抓官方 press-release / popular-information 原文到 `sources/` |
| `sources/*.article.txt` | 官方原文（本页正文的事实依据，改动前先读这里） |

## 逐奖更新流程（揭晓当天）

> ⚠️ **2026-10-12 修订（重要）**：本仓是「workspace 源 + `/tmp/nobel-2026` 暂存」双目录，
> `deploy.py` 只读**暂存目录**。所以在 workspace 改完源码后，**必须走 `make.py`**（同步 → 在暂存构建 → 跑状态门），
> 不要直接 `build.py` + `deploy.py`——否则会出现「改了源码、部署了旧稿、md5 还报 0 差异」。

```bash
# 1) 先看官方原文（两个来源都要，引文用原句）
python3 projects/nobel-2026/harvest_nobel.py          # 抓 <cat>/press-release + popular-information
# 2) 检查新增
python3 projects/nobel-2026/check_new.py              # 退出码 10 = 有新奖项
# 3) 拉官方图（新奖项的插图 / 得主肖像）：改 download_assets.py 的 ASSETS → optimize_assets.py 的 JOBS
#    然后改 build.py 里的数据结构（状态词一律由 PENDING 派生，不要手写「待揭晓」）
python3 projects/nobel-2026/download_assets.py && python3 projects/nobel-2026/optimize_assets.py
python3 projects/nobel-2026/make.py                   # ★ 同步暂存 → 构建 → 状态一致性门（state_check）
# 4) 五道闸门（全绿才部署）
python3 projects/nobel-2026/state_check.py            # ★ 状态词 × 现态（状态型页面的第一道门）
python3 projects/nobel-2026/verify.py                 # 本地：console=[] / over=0 / 外部请求=0
python3 projects/nobel-2026/contrast_check.py         # 对比度 0 失败
python3 projects/nobel-2026/img_check.py              # 37 张图全部解码成功
python3 projects/nobel-2026/text_clip_check.py        # 0 处文字被截
python3 projects/nobel-2026/review4.py                # 出审图 → 用 vision 工具自己看
# 5) 部署 + 线上复核（deploy 带发布前新鲜度门：暂存≠源码 / 产物比源码旧 / 状态门不过 → 拒绝部署）
python3 projects/nobel-2026/deploy.py --apply --incremental
python3 projects/nobel-2026/live_check.py             # 线上口径复测 + 回群截图
python3 projects/nobel-2026/state_check.py <线上存下的 html>   # ★ 线上也要过状态门
# 6) 群里报一句（群 oc_83d1efa178440da1f1efcdbf012c03c5）
```

按官方 `dateAwarded`：**10/7 化学 · 10/8 文学 · 10/9 和平 · 10/12 经济学**。
每次更新后还要把 `HEARTBEAT.md` 里的临时追踪条目留着（10/12 全部揭晓后再删）。
全部揭晓后：加一层「跨奖对照」（学科地图 / 一句话总结），然后收尾。

## 取材纪律（不可破）

- 引文用官方英文原句 + 中文转述，不改写官方动机句。
- 得主信息（出生年、机构、获奖发现完成地）逐字取自官方页面。
- 「为什么重要」只写官方原文里出现过的内容（临床进展、已点名的回路、候选源等）。
- 未揭晓的奖项：只写日期与颁奖机构。
- **图片**：只用 nobelprize.org 官方原图，**逐张保留官方署名与官方原题（英文）**；每张附「查看官方原图」链接；本页只做等比缩放 + WebP 转码，不裁切、不加字。官方授权条款原文见页面「图片来源与授权」（科普插图：非商业用途免费 + 署名；得主肖像：编辑用途免费 + 版权与作者名，不可转授权）。
- 2026 得主不在任何训练数据里 → 除官方来源外，任何数字都不得写入。

## 变更记录

- 2026-10-07 v1：建仓 + 上线。医学（Deisseroth / Hegemann / Nagel）、物理（Halzen）两张完整卡；化学 / 文学 / 和平 / 经济学四张待揭晓占位卡。
- 2026-10-07 v2：加 data-URI favicon（消掉 favicon 404）；示意图改为「图内无文字 + HTML 图例」，修掉手机端字号被缩放得过小的问题。
- 2026-10-07 v3（Sky 三项指令：中英对照中文为主 / 参考菲尔兹奖页 / 新仓 / 风格另设一套符合 Nobel）：
  - **双语版式**：英文一律作「第二层」——小标题用「英文眉标在上 + 中文主标题在下」两行结构；正文里的英文例句单独成行、左侧细线、斜体、字号小一档。
  - **视觉风格取自 Nobel 官网设计变量本身**（从 `nobelprize.org/wp-content/themes/nobelprize/dist/css/frontend.css` 提取）：`--nobel-gold-500 #cea152` / `--roti #8e6518` / `--berlin-blue #00263e` / `--dune #2e2a25` / `--jumbo #5d5b58` / `--lochMara #2370b8` / `--very-light-grey #c8c8c8`。奖卡卡头改为深蓝底 + 金色标签 + 白色英文引文（引文即主角）。
  - **结构参考菲尔兹奖页**：置顶导航 + 顶部阅读进度条 + 锚点分区（未照搬其「羊皮纸 + 金色」配色）。
  - **新增对比度实测** `contrast_check.py`（背景逐层 over 合成 + opacity 连乘口径）：首轮 82 处不达标 → 修 `--ink-3` 3.62:1→5.6:1、并查出**待揭晓卡英文标签白字写进浅底的隐形 bug** → 现 0 失败。
  - 零第三方依赖不变（不用 Google Fonts；菲尔兹页依赖的 Noto Serif SC 在国内不可达，本页改走系统衬线栈）。
- 2026-10-07 v4（Sky 四项指令：多图/图像化 + 视频链接 + 导航目录层级 + 得主照片 + 尽量引用官方原图）：
  - **重构成「数据 + 模板」**：`build.py`（内容与渲染）+ `style.css`，后续 4 次揭晓只改数据不改标记。
  - **17 张官方原图入库**：医学 9 张（概念图 + 4 张科普插图 + 4 张进阶数据图，可折叠）+ 物理 4 张（概念图 / 中微子 / 冰下观测站 / 宇宙信使）+ 得主肖像 4 张（Niklas Elmehed 绘）。图注 = 我的中文导读 + **官方原题英文原文** + 官方署名行 + 官方原图链接。
  - **授权先查后做**：先读 nobelprize.org 条款（`/about/terms-of-use/` + 新闻稿内的插图声明 + `/press-images-for-media/`），确认「非商业免费 + 署名」与「编辑用途免费 + 版权与作者名、不可转授权」后再用；页面新增「图片来源与授权」一节把**英文原文**贴在页面上。
  - **导航与目录**：目录卡（编号 + 奖项名 + 完整标题 + 小节胶囊链接）、吸顶导航高亮（IntersectionObserver 无依赖）、悬浮「↑ 回到目录」按钮；手机端导航改为可横向滚动（原先是直接隐藏）。
  - **修掉两个真问题**：① 点目录跳转后标题被吸顶导航盖住 → 加 `scroll-margin-top`（实测导航高：桌面 50px / 手机 87px，落点改为 64 / 100）② 手机端目录里中文被右侧说明挤到不自然断行、人名被拆行（如「戴瑟罗/斯」）→ 目录改两行结构、人名 `word-break:keep-all`。
  - **版式再平衡**：医学 3 位得主由「左文右人」（695 vs 422px，明显不齐）改为**得主并排通栏**（3 幅肖像一行）；关键数字改通栏（11 组数字是这页的记忆锚点）。
  - **部署改为 Git Data API 原子提交**（blobs → tree → commit → ref），18 个文件一次提交；**逐个文件比对线上 md5**（本次 18/18 OK）。
  - 新增两个机械闸门：`img_check.py`（图片是否真的解码，不是只看标签）、`text_clip_check.py`（只对 `overflow≠visible` 的容器报文字被截，避免 line-height 假阳性）。
  - ⚠️ GitHub `/git/blobs` 偶发 `400 malformed request`（同一请求重试即 200）→ 已在 `deploy.py` 加退避重试（400/429/5xx × 4 次）。

- 2026-10-07 v5「博物馆版」（Sky 11:46 / 11:49 / 11:51 三条指令）：定位从「逐日跟踪页」改为「**知识与科普博物馆**」，风格与结构不动，内容从「一条线索」扩到「一座展厅」。
  - **11:46（不要自绘 SVG）**：v4 已完成——17 张全部官方原图，页内零自绘示意图。
  - **11:49（官方来源里的信息多整理进展页）**：新增 **「发现的故事」共 18 节**（医学 10 节 / 物理 8 节），逐节改写自官方 popular-information 的叙事小节，**章节标题保留官方英文原标题**（英文仍作第二层）。官方原文里此前没用上的内容已入页：伍尔夫 1932 年的信、克里克当年的设想、衣藻眼点与半毫秒、分离蛋白多年的壁、日本公开衣藻基因组这一转机、蛙卵注射、2003/2005/2006/2007/2012 的时间线、利根川进与记忆痕迹、心率与焦虑、肠道糖偏好、视网膜色素变性临床试验、耳蜗植入体；物理侧：650 亿个中微子穿过指甲盖、1988 波兰会议、1992 第一次下探、1993 平安夜、AMANDA、1400 米以下冰纯净到光能走 300 米、5160 传感器 / 86 缆 / 1 km³、每天一亿个背景粒子、2002/2015 两次诺奖的前史、79 个来自 NGC 1068 的中微子、Gen2 的 8 km³。
  - **新增「展线目录」**：每座展厅开头的章节胶囊导航（短标签，手机端两列换行，**不靠横向滑动、不藏内容**），章节自带 id 可直达。
  - **新增「术语小词典」18 条**（面向初二）：官方用词 + 是什么 + **打比方**；打比方部分逐条标「本站解释」，不与官方叙述混排。
  - **新增「想想看」12 问**：不给答案，明说「答案都在下面链接的官方页面里」。
  - **新增「学科地图」**：两个已揭晓奖的对照（光当作可控制的开关 vs 光当作最后一片信纸），**显式标注「本页的观察，不是诺贝尔委员会的表述」**。
  - **本馆说明**（原「来源与授权」升级）：补「本站自己写的部分是哪些」一段 + 版本进度（已揭晓 2/6）+ 图片与条款原文保留。
  - **修一个真 bug（实测发现）**：导航项从 5 个变 6 个后，手机端导航从 87px 撑到 **120px**，而锚点偏移还写死 100px → **点目录跳过去标题被导航盖住**。改法不是再调一个数字，而是**让脚本量出导航实际高度写进 `--navh`，锚点用 `calc(var(--navh) + 18px)` 算**；同时手机端导航链接改 `flex-wrap:nowrap`（单行横向滚动）。新增 `anchor_check.py` 逐个点 7 个锚点 × 3 视口实测：**被遮 0 个**。
  - 四道闸门（v5 实测）：三视口 0 溢出 / console 0 报错 / 运行时 0 外部请求 / **1059 元素 0 对比度失败** / 17 图 0 破图 / 0 处文字被截。页面尺寸：桌面 24,133px、手机 43,392px（博物馆定位下内容量翻倍，靠展线目录 + 吸顶导航导航）。
  - 注：`verify.py` 的溢出检查已修正为**跳过「祖先里带横向滚动容器」的元素**（展线目录/手机导航属设计内滚动，不应计溢出）。

- 2026-10-07 v6 手机端导航（Sky 拍板方案 A）+ v6.1 目录英文名：
  - **顶栏**：手机端单行 **69px**（左品牌 + 右「三」）；删掉原来的横向滚动链接行。点击 → 顶栏下方**满屏白底面板**（top=69 / 宽=屏宽 / 高=屏高−69，面板内滚动），6 个展厅行 ≥56px，医学/物理可展开二级手风琴（原生 `<details>`，零 JS）。
  - **几何来源 = nobelprize.org 自家手机端实测**（`.site-header` 68px、面板紧贴顶栏下、占满剩余全屏、菜单行 59px、打开锁 body 滚动、关闭态 `visibility:hidden;opacity:0`）。「按主流做法」类需求一律先量对方实现再动手。
  - **交互全实测**（`menu_check.py`）：点面板链接 → 先收面板再落点（落点 top=87 / 导航 69 → 被遮 0）；点遮罩关；`Esc` 关且焦点回「三」；打开锁背景；`resize` 自动收起；**面板内 0 个元素 <44px**；桌面端 6 项横排、无「三」、无面板 → 零回归。
  - **展线目录**：手机端默认单行（标签 + 「发现的故事 N 节 ▾」，44px），点开成竖排列表；展开行显示**官方英文原标题**；桌面端仍是横排胶囊。
  - **目录英文名（v6.1）**：总览目录每行 = 中文名 / 英文名（官方分类名，斜体小号）/ 说明；顶栏链接英文名只在 ≥1080px 显示（平板宽度会换行把导航撑到 108px）；手机面板行右列为英文名。
  - **闸门**：三视口 0 溢出 / console 0 / 0 外部请求 / 对比度 0 失败（1168 元素；面板按「打开状态」单测）/ 17 图 0 破图 / 0 文字被截 / 7 锚点 × 3 视口被遮 0 / 线上 18 文件 md5 全对。
  - ⚠️ **事故与铁律（重要）**：这轮我把工作区的**旧 index.html 拷回了 /tmp**（想同步产物，方向搞反），把已构建的 v6.1 静默回滚成 v6——**而且闸门是在覆盖之前跑的（全绿），部署的 md5 校验也只比对 /tmp 与线上（两边同为旧稿，照样「一致」）**，所以三道关都没拦住。
    **铁律：产物只允许单向流动 `/tmp(构建) → workspace`；任何 `cp` 之后必须先 `md5sum` 两侧确认方向，再跑闸门、再部署。**
    （这次靠「线上截图里目录没有英文名」目视发现，因为机械闸门只验结构不验内容新旧。）

- 2026-10-07 **图标口径（已定，勿再改）**：手机顶栏按钮用**三条横线（汉堡「三」）**，CSS 绘制（19×2px ×3，gap 4px，`flex-direction:column` 不可省），打开态换 ✕，按钮 44×44。
  - 同日曾试用「清单/目录」图标（圆点+横线 SVG），**Sky 判定不好看 → 已回退**（回退后产物 md5 与改图标前逐字节一致 `b984605b…`）。此条为终态口径：不要再自行更换图标样式。
- 2026-10-07 v7（化学厅，cron 自动轮）：新增第三座展厅「03 化学 Chemistry」——Kagan / Soai，各 1/2。内容全部转写自官方 summary / press-release / popular-information（原文存 `sources/chemistry-*.article.txt`）：9 节「发现的故事」（章节标题保留官方英文原题）、因果链 10 步、官方插图 6 张（Johan Jarnestad © 瑞典皇家科学院）+ 得主肖像 2 张（Niklas Elmehed © Nobel Prize Outreach）、术语小词典 11 条、想想看 5 问。官方访谈影像尚未发布 → 该节**留白并写明原因**，不用非官方素材顶替。同步更新：PENDING 3 项、render_toc 编号、学科地图（三馆对照）、hero 3/6、手机面板（03 化学厅）、图片总数 17→25。
  - **⚠️ 本轮排障（重要，给下一次揭晓用）**：18:10 那一轮已抓完源、拉完图、改完 build.py 并构建，但在跑闸门/部署前中断 → `state.json` 已写入化学奖，**于是本轮 `check_new.py` 返回退出码 0（报「无新增」），而线上仍是旧稿**。判定依据不能只看退出码：必须加一步「线上是否已含该奖项」（`curl` 后 grep 得主姓名），发现「state 有新奖 + 线上无新奖」= 未完成交付，接手补完（本轮即如此：闸门 5 道全绿 → 部署 26 文件「线上不一致 0」→ 线上截图视觉复核）。
  - 本轮闸门实测：三视口 0 溢出 / console 0 / 0 外部请求 / 对比度 1608 元素 0 失败 / 25 图 0 破图 / 0 文字被裁 / 9 锚点 × 3 视口被遮 0 / 线上 26 文件 md5 全对 / 线上 console 0、4xx-5xx 0。
  - **⚠️ 与「单向流动」铁律的关系**：本轮 `cp` 方向为 `/tmp → workspace`（新稿 d60b09a6 覆盖 workspace 旧稿 b984605b），cp 后两侧 md5 已核对一致才继续。

- 2026-10-08 v8（文学厅，**心跳兜底轮**）：新增第四座展厅「04 文学 Literature」——Anne Carson，1/1。文学奖官方**没有 popular-information 页**：故事节改自官方 **Biobibliography**（文学委员会主席 Anders Olsson 署名，原文存 `sources/literature-*.article.txt`），7 节（官方无小节标题 → 英文章节标签用官方作品名+年份，中文标题为本站所加且页面已声明）、因果链 12 步、关键数字 7 条、得主肖像 1 张（Niklas Elmehed © Nobel Prize Outreach）、术语 11 条、想想看 5 问、来源 6 条。官方**既无科普插图也无访谈影像**（Interview 页 404）→ 两节留白写明原因，不用非官方素材顶替；拉丁文标题 `Odi et Amo Ergo Sum` 官方无译文 → 不译。同步更新：PENDING 2 项、render_toc 编号、学科地图（四馆对照 + 「这三个奖」改「四个」）、hero 4/6、手机面板（04 文学厅）、图片总数 25→26。
  - **⚠️ 本轮根因修复（改语义，不是再记一条）**：10-07 化学轮踩过的「探测吃掉信号」**原样复发**（上条排障只写了文档、没改代码——同型于 09-10「只写日记不改技能」）。`check_new.py` 无条件写 `state.json`，而 cron 第一道闸门是「exit 0 → 立刻结束」，于是任何一次探测（心跳/手动）之后，21:10 那轮都会静默空转、页面永远不上线。**修法：`state.json` 语义由「已看到」改为「已交付」**——新增奖项只在「线上页已含该得主英文全名」时才记账；线上没有 → 不写 state、保持 exit 10（信号不可被提前消费）；线上取不到（异常/5xx）→ 重试 3 次后按欠交付处理（宁可重做，不可静默丢稿）。新增 `--state <path>` / `--live <url>` 便于离线复现三态。三组实测：欠交付 `EXIT=10` / 已交付 `EXIT=0` 且记账 / 正常轮 `EXIT=0`。
  - 本轮闸门实测：三视口 0 溢出 / console 0 / 0 外部请求 / 对比度 1977 元素 0 失败 / 26 图 0 破图 / 0 文字被裁 / 9 锚点 × 3 视口被遮 0 / 线上 27 文件 md5 全对 / 线上 console 0、4xx-5xx 0。页面尺寸：桌面 45,294px、手机 79,658px。
  - 工具修：`verify.py` 整页截图超时 30s → 180s（展厅变长后整页截图会超时；**只放宽超时，不放宽任何断言**）。
  - 交付：commit `cc70dbe7…`、「线上不一致 0」、线上 md5 = `5b24a2049769d7caa6a912341a0dce10`；19:51 CST 报群 + 2 张截图（回执见 `/tmp/cron-output/nobel-2026-tracker/delivery.log`）。

- 2026-10-07 起：**追踪自动化** —— cron `nobel-2026-tracker`（10/7–10/12，每天 18:10 / 21:10 CST，isolated agentTurn，timeout 1800s，delivery=none + 自投递 + failureAlert 到 Sky DM；表达式 `10 18,21 7-12 10 *` Asia/Shanghai **自带终点**）。流程：`check_new.py` 退出码 0（= 无新增，**或**新增已在线上）→ 静默结束；10（= 已揭晓但线上还没有 = 欠交付）→ 采集官方页 → 官方图转 WebP → 改 build.py 数据 → 五道闸门 → 部署 + 线上截图复核 → 报群 + 写回执 `/tmp/cron-output/nobel-2026-tracker/delivery.log`。心跳保留兜底检查（exit=10 而群里 1 小时无更新 → 手动补）。

## 设计口径（改动前先读）

- 配色**只用 Nobel 官网自己的变量**，不引入第二套色系；金色 `#cea152` 用于结构（标签、编号、进度条、引文竖线），深蓝 `#00263e` 用于卡头与导航；「光」的语义色统一用 `--lochMara #2370b8`（光遗传学的蓝光与切伦科夫蓝光同色）。
- 中文为主：中文用无衬线正文 + 加粗层级，英文用衬线斜体、更小更浅，永不与中文并排成一段。
- 改色后必须重跑 `contrast_check.py`，**先读检测器打印的 fg/bg 再决定往深还是往浅改**。

- 2026-10-09 v9（和平厅，cron 轮；接手当日 18:10 中断轮）：新增第五座展厅「05 和平 Peace」——**Navi Pillay**，1/1，引文 “for her efforts to promote peace and international law”（存 `sources/peace-*.article.txt`：summary / press-release / facts / announcement / other-resources 共 5 页）。10 节「发现的故事」（章节标题保留官方英文原题）、因果链、关键数字 12 条、官方主题图 1 张（Ill. Niklas Elmehed © Nobel Prize Outreach，alt 取官方原句）+ 官方图库实拍 2 张（US Mission Geneva / Antônio Cruz, Agência Brasil —— 署名逐张保留）、得主肖像 1 张、术语小词典、想想看、来源、官方影像 `X006qYLS-xU`。和平奖**没有 popular-information 科普插图**（官方页 404）→ 该节留白写明原因，**不用非官方素材顶替**（延续「宁缺勿假」，且不标成「源不可达」——源没坏，是官方未提供该类别）。同步更新：PENDING 1 项、**NAV 补 `peace`（桌面顶栏此前漏项，本轮修）**、render_toc 编号、学科地图（五馆对照）、hero 5/6、手机面板（05 和平厅）、图片总数 26→30。
  - 本轮闸门实测：三视口 0 溢出 / console 0 / 0 外部请求 / 对比度 2475 元素 0 失败 / 30 图 0 解码失败 / 0 文字被裁 / **11 锚点 × 3 视口被遮 0**（anchor_check 新增 `#peace-story-3`、`#peace-gloss` 两项，新展厅必须进闸门）/ 线上 31 文件 md5 全对 / 线上 console 0、4xx-5xx 0 / 30 个图片 URL 逐个直查 0 失败。
  - 交付：commit `9676fe06…`、「线上不一致 0」、线上 md5 = `b9d1108c0bcf968ad34e25a727d62f96`；18:47 CST 报群（回执见 `/tmp/cron-output/nobel-2026-tracker/delivery.log`）。
  - **工具修（`live_check.py`，本轮两处根因，都属于「机械闸门自己不可靠」）**：① 无界 `Promise.all(图片 onload)` → 屏外懒加载图永不触发 load/error，`page.evaluate` 无超时 → **整脚本死等**（本轮实测卡死 7 分钟、CPU 0%、无任何输出；已改为 `Promise.race` 有界等待 10s/15s）；② 用 `naturalWidth` 判「破图」→ 回滚到顶部时浏览器取消屏外懒加载请求，读数随机（每次跑到的文件名都不同，实为**假阴性**）→ 改为**逐 URL 新建 `Image()` 直查**（`asset_probe`：30 URL / 0 失败），并把「屏外未取图」单列为 `unloaded`，不再算损坏。
  - ⚠️ 判「无新奖」不能只看 `check_new.py` 退出码（10-07 化学轮的老坑）：本轮开头即核到「state 已记账但线上无和平厅」= 欠交付 → 接手补完，未被 exit 0 静默吃掉。

- 2026-10-10 v14（**新增第 4 页 `about.html`：「诺贝尔奖是怎样运转的」**；Sky 18:21 指令，被 18:22 看门狗重启 kill 后由续轮接手）：
  - 四块内容各占一节：**① 诺贝尔其人**（官方 Key events 年表 12 节点 + 6 张速览卡）/ **② 一份遗嘱**（1895-11-27 签于巴黎 → 1901-12-10 首次颁发，含遗嘱原文引文）/ **③ 谁在颁奖**（物理·化学·经济科学→瑞典皇家科学院；医学→卡罗林斯卡学院诺贝尔大会；文学→瑞典学院；和平→挪威议会选出的五人委员会）/ **④ 诺贝尔基金会**（1900-06-29 成立；章程 §6/§13/§17/§18；50 年保密；Outreach / 博物馆 2001 / 和平中心 2004 / Nobel Center 2031）。
  - 主线是 Sky 的原问题「**为什么这么多年了还可持续发奖金**」：`00 先给答案` + `05 钱为什么还在（五件事）` + `06 1901–2025 官方实际价值指数`（奖金购买力 1919 谷底 28% → 1991 才回到 101%；本金实际价值 1975 只剩 51% → 2025 年 297%）+ `07 三件必须说清楚的事`（反方：297% 不全是管理得好——免税是议会特批、1980–2000 是长牛、官方表格 1975 年才有记录）+ `08 想想看`。
  - **为什么是独立页而不是新展厅**：它不追踪 2026 奖项，属背景资料；首页已 4.5 万 px（手机 8 万 px）；与 `cn.html`/`jp.html` 同类横切专题页；且不动 cron 的奖项管线。
  - 数据全部来自官方一手：nobelprize.org 16 个页面 + 3 份官方 PDF（`prize-amounts-2025.pdf` 全表 · `annual-report-2025.pdf` · `guidelines-investments-2026.pdf`），原文落 `sources/about/`。**官方没写的本页留白**（例：「他因内疚才设奖」这类流行说法专门说明为何不写）。
  - 措辞修正（本轮自捕 3 处，属「不许把推断写成事实」）：经济科学奖原写「它没有成为诺贝尔奖 / 央行捐钱覆盖行政与庆典费用」→ 官方原文不在我手上的源里，改为官方两句原文（1968 年央行借成立 300 周年捐赠 · 奖金金额与诺贝尔奖相同且由央行支付 · 1969 首次颁给 Frisch 与 Tinbergen）；责任投资「自 2016 年起是 PRI 签署方」→ 官方无年份，删年份；「1919 因一战通胀」→ 改为官方表格可支持的「名义奖金没涨（15.1 万→13.3 万），物价却涨了几倍」。
  - 同步改动（四页联动）：`build.py` NAV/MNAV/TOC 加 `about` 第 10 项、`本馆说明` 10→11、页脚「本站共四页」；`cn/build_cn.py`、`jp/build_jp.py` 顶栏 + 手机面板 + 页脚互链补 `about.html`；`deploy.py` 的 `files()` 加 `about.html` + `assets/about/*.webp`，`MSG` 改 v14。
  - 本轮闸门实测：**四页全部**过闸 —— `about.html` 三视口 0 溢出 / console 0 / 0 外部请求 / 对比度 1016 元素 0 失败 / 7 图 0 解码失败 / 0 文字被截 / 10 锚点被遮 0；`index/jp/cn` 各自 verify·contrast·img·text-clip·anchor 全绿（`index.html` 30 图，首跑 broken=1 复跑 0 = 懒加载抖动，已记录）。
  - **顶栏回归实测（有对照）**：线上旧 11 项 = 2 行 / row1=10 row2=1 / h=108px；新 12 项 = 2 行 / row1=10 row2=2 / h=108px / 出界 0 —— 无回归（对照脚本 `nav_compare.py`，1440 与 1080 两档）。
  - 交付：commit `1d6757e6f6dc…`、「线上不一致 0」（11 个变更文件逐个 md5 比对）、线上 `about.html` md5 = `d3d91915369275a7c15b822b79824bba`；线上 `verify.py` 复核 console 空 / 0 溢出 / 0 跨站请求；线上首屏截图视觉复核通过。
  - **工具修（`deploy.py`，本轮两处都是「机械闸门/工具自己不可靠」的根因）**：① **增量部署** `--incremental`——本地算 git blob sha1 与远端树递归表比对，只上传内容变了的文件（本轮 77 → 11 个 blob，上传时间从 >30 分钟降到约 6 分钟），未变文件跳过上传与复验；② **重试加长**：`req()` tries 4→6、退避 2+3k → 4+6k 秒——本轮 VPC TLS 抖动两次打断部署（400 malformed / `EOF occurred in violation of protocol`），旧参数会在第 4 次就放弃（且中途失败不留残迹：blob 是孤儿对象、commit 未推 → 仓不变）。
  - 工具修（`anchor_check.py`）：改为接受 `<url> [#锚点…]` 参数并在输出里标明页面名，四页可共用一个脚本跑。

- 2026-10-12 v15（**经济厅 · 六奖到齐**，cron 轮；接手当日 18:10 中断轮）：新增第六座展厅「06 经济科学 Economic Sciences」——**Robert E. Hall / Michael Woodford**（各 1/2），引文 “for microeconomic foundations of macroeconomic stabilisation”（即官方科普说明页标题 *Microfoundations deliver macro insights*）。内容全部转写自官方 summary / press-release / popular-information / 两位得主 Facts 页（原文存 `sources/economic-sciences-*.article.txt`）：**5 节**「发现的故事」（章节标题保留官方英文原标题：Microfoundations deliver macro insights / Stabilisation policy became ineffective / Hall turns research upside-down / Woodford’s model / An adaptable model）、因果链 13 步、官方插图 5 张（主图 + Fig 1–4，Johan Jarnestad © 瑞典皇家科学院）+ 得主肖像 2 张（Niklas Elmehed © Nobel Prize Outreach）、关键数字 11 条、术语小词典 16 条、想想看 5 问、官方来源 8 条。官方**访谈影像尚未发布**（两页实测 404）→ 该节留白写明原因，不用非官方素材顶替；揭晓现场影像 `RJjHROJh-FI`（取自官方公告页 iframe，已核）。
  - 同步更新：PENDING 清零（该节改为「六个奖已全部揭晓」说明）、render_toc 编号（06）、学科地图改**六厅对照**（共同点段补经济厅「换了个被读的对象：读的是人的决定」）、hero 6/6、手机面板（06 经济厅）、图片总数 30→37。
  - 本轮闸门实测：三视口 0 溢出（桌面页高 71,021px / 手机 131,393px）/ console 0 / 0 外部请求 / 对比度 3055 元素 0 失败 / 37 图 0 解码失败 / 0 文字被裁 / **14 锚点 × 3 视口被遮 0**（`anchor_check.py` 新增 `#economic-story-3` `#economic-gloss` `#economic-people`）/ 线上 console 0、4xx-5xx 0、`asset_probe` 37 URL 0 失败。
  - 交付：commit `ff8fe85a3972…`、增量部署 8 个 blob（远端 76 个未变跳过）、**「线上不一致：0」**、线上 `index.html` md5 = `549ee1254a9889b458eacd73804065bf`；线上截图视觉复核确认六厅齐（含经济科学厅标题与两位得主姓名）、学科地图 6 栏、手机面板有「06 经济厅」、无破图。回执见 `/tmp/cron-output/nobel-2026-tracker/delivery.log`。
  - **⚠️ 本轮接手判定（「探测吃掉信号」的第三种形态——数据已备但未构建）**：本轮开头 `check_new.py` 报 exit **10**（= 已揭晓但线上还没有），而当日 18:10 那轮已在 workspace 改好 `build.py` / `download_assets.py` / `optimize_assets.py` 并生成了 7 张新图（`eco-*.webp` 5 张 + 2 张肖像），**但没构建、没部署就中断了**（`/tmp/nobel-2026/index.html` 仍是 10-10 旧稿）。判定依据 = 文件 mtime + 两侧 md5 对照（`build.py` workspace 与 /tmp 一致 = 数据已改、产物未生成）。接手时按「先核事实 → 再构建 → 再闸门 → 再部署」补完，不重做已在位的工作，也不跳过核验。
  - 数据核验（逐条比对官方原文）：出生年/博士年/机构/份额（1943 Palo Alto · PhD 1967 MIT · Stanford ／ 1955 Chicopee · PhD 1983 MIT · Columbia ／ 各 1/2）、约 80% 家庭、1978 文章、1982 Hall & Mishkin、2003 书、三个方程、Lucas 1995、Wicksell 19 世纪末、Hassler 引语、Fig 1–4 官方标题——全部与 `sources/economic-sciences-*.article.txt` 一致；5 张插图逐张用 vision 复核「图与标题对得上」（Fig 1 失业/通胀曲线被压平 · Fig 2 控制箱内外结构 · Fig 3 未来消费分叉 · Fig 4 三组天平与产出缺口）；两张肖像用官方 summary 页的 DOM 先后顺序核过「`1-2_3-6acc90cb…` = Hall ／ `2-2_3-6acc90d2…` = Woodford」。
  - 工具：新增 `shot_economic.py`（经济厅视图截图，照 `shot_peace.py` 改锚点）与 `shot_live_econ.py`（线上经济厅内容复核截图），均已入库。
  - **追踪管线终点**：截至本轮六个奖全部揭晓、六座展厅到齐、跨奖对照已补，cron `nobel-2026-tracker` 的表达式（`10 18,21 7-12 10 *`）**自带终点**，10-12 之后不再触发（故无需 disable；本轮无 cron 工具可用，未做 cron 变更）。`HEARTBEAT.md` 的临时追踪条目已改为「收尾确认」态，确认今日 21:10 那轮 exit 0 后即可删除。

- 2026-10-12 v16（**状态漂移修复 + 发布前闸门**，Sky 指出「导航目录里怎么还有『待揭晓』」后当场修）：
  - **事实**：六个奖 10-12 全部揭晓后，`PENDING` 已清空、正文小节已改成「六个奖全部揭晓」，但**状态词硬编码在 4 处**没跟着改——顶栏导航（`NAV_LABEL`/`NAV_EN`）、手机面板（`MNAV`）、目录第 07 行（`render_toc` 手写「尚未揭晓」）、小节标题（`render_pending` 的 `Pending`）。修前线上命中：`待揭晓` ×2 / `尚未揭晓` ×2 / `Still to come` ×3 / `Pending` ×1（共 8 处）。
  - **改法（派生，不再手写）**：新增 `_HAS_PENDING = bool(PENDING)` → `LBL_STATUS_ZH/EN/DESC`，四处全部引用它；`PENDING` 非空时仍显示「待揭晓/Still to come」（含「N 项未公布」），为空时自动变「揭晓进度 / All six announced / 六项已全部揭晓」。小节标题改为 `All announced · 六个奖项全部揭晓 · 0 项待公布`。
  - **新增机械门 `state_check.py`**：读 `build.py` 的 `PENDING`，对产物/线上 HTML 做「状态词 × 现态」对撞（`PENDING==[]` → 全页禁止 6 个状态词且必须含「全部揭晓」；`PENDING!=[]` → 反之必须有提示），并核对导航状态位标签与展厅数（6 座）。**双测已跑**：对旧线上 = `STATE_CHECK=FAIL(1)`（列出 4 词 8 处），对新产物 = `PASS`。
  - **部署链根因修复（本轮真凶）**：`deploy.py` 的源是暂存目录 `/tmp/nobel-2026`，我在 workspace 副本改了 `build.py` 并构建 → **没同步暂存** → 上传旧稿；而部署末步 md5 是「暂存 ↔ 线上」**自比** → 报「线上不一致 0」全绿。两道修：① 新增 `make.py`（workspace → 暂存同步 → 在暂存构建 → 跑状态门，一步到位）；② `deploy.py` 加 `preflight_freshness()` 发布前闸门（源码与暂存内容不一致 / 产物比源码旧 / 状态门不过 → **拒绝部署**，exit 2）。README 的「逐奖更新流程」已改为以 `make.py` 为准。
  - 交付：commit `614cf507adc7…`、增量部署 1 个 blob（远端 83 个未变跳过）、线上 `index.html` md5 `39c1ca3a08bdc2f957770447f326eb59` = 本地、`state_check` 对**线上** = `PASS`；线上 DOM 实测顶栏第 8 项 = 「揭晓进度 / All six announced」、正文可见文本 0 个状态词、手机面板第 07 项 = 「揭晓进度 / All six announced · 六项已全部揭晓」。
  - 教训已回写：`skills/delivery-verification/SKILL.md` 新增「自证循环：验收对象 ≠ 交付对象时，全绿也是假的」（含状态型交付物必须机械扫状态词的通用纪律）。

# AGENTS.md — AI 日报工作流规则

> 本文件遵循 [AGENTS.md 开放标准](https://agents.md)，定义「AI 日报」独立项目的工作规则与助手（小汇）行为约束。
> 本目录是完整、独立的项目边界，不依赖上级目录中的规则、脚本或数据文件。

## 1. 项目概述

**AI 日报** 是面向公开阅读的 AI 资讯首页，专门沉淀最新 AI 领域动态。

- **角色**：小汇（📥 每日信息汇点管家）作为 AI 日报助手
- **产出形态**：`index.html` 固定展示最新一期；旧首页按日期归档为 `daily/YYYY-MM-DD.html`，均为纯静态、零依赖 HTML
- **产出位置**：最新页只写入本目录 `index.html`，历史页只写入 `daily/`
- **视觉基准**：`template.html` 是只读设计模板；日常生成必须先读取并严格沿用，只有人工升级模板时才允许修改
- **更新方式**：生成新一期前先归档现有 `index.html`，归档成功后再用更新日期的新内容替换首页
- **执行方式**：由外部 AGENT 或调度器按需运行；本项目只规定采集、策展与首页更新规则，不绑定具体自动化平台或任务 ID
- **原则**：当日事当日记，第二天不补昨天的账；漏记就漏记，保持低摩擦

## 2. 目录结构

```
AI/
├── AGENTS.md             # 项目规则、数据源与页面模板规范
├── template.html         # 只读视觉与 DOM 基准；不承载当日真实数据
├── index.html            # 最新一期公开首页；每天原位更新
└── daily/                # 历史日报；按旧首页自身日期归档
    └── YYYY-MM-DD.html
```

## 3. 最新首页与历史归档管理

- `index.html` 始终代表最新一期，文件名不随日期改变；新首页的 `<title>`、Hero 和更新时间必须更新为本次执行日期。
- `daily/` 只保存已下线的旧首页，归档名使用旧页面自身显示的日报日期：`daily/YYYY-MM-DD.html`。不得用新页面日期或文件修改时间冒充旧日报日期。
- 生成新一期前，必须先完整读取当前 `index.html` 并提取其日报日期；只有旧页成功归档后，才允许更新 `index.html`。
- 旧页日期与本次新首页日期不同时才执行日归档；同日重复生成视为修订当前一期，直接更新 `index.html`，不产生重复日归档。
- 若 `daily/YYYY-MM-DD.html` 已存在：内容与当前旧首页完全一致则跳过重复写入；内容不同则保留已有文件，并将待归档旧页写为 `daily/YYYY-MM-DD-HHmmss.html`，时间后缀使用北京时间。若带后缀路径仍重名，重新取得北京时间生成唯一文件名；仍无法避免冲突时停止归档，任何情况下都禁止覆盖历史内容。
- 无法从旧首页可靠提取日期时，停止更新并报告，不得猜测日期、不得先覆盖 `index.html`。
- 归档页必须保留旧首页的完整 HTML/CSS/JS 与原有内容，不得改写为新一期，也不得归档 `template.html`。
- 除 `daily/` 外禁止新增其他归档目录；日期命名 HTML 只能位于 `daily/`，不得放在项目根目录。
- 文件使用 UTF-8 编码、LF 行尾；样式与脚本全部内联。
- 修改前必须完整读取 `template.html` 与当前 `index.html`；以模板的 DOM、CSS、响应式和无障碍约束为准，只替换当日内容与统计。
- `template.html` 不属于每日产物，不得把模板占位符直接留在 `index.html`，不得在每日任务中修改模板。

## 4. 日报内容结构（自动化仪表盘）

每个 AI 日报 HTML 必须包含以下区块，顺序固定：

1. `<title>`：`AI 日报 · YYYY年M月D日`
2. **Hero 头部**：日期（人话格式）+ 数据覆盖窗口（北京时间）+ 总条数 + 五版块统计卡片（含「编者补充」条数标注）
3. **锚点导航**：sticky 顶栏，五个版块 chip（版块名 + 条数），点击平滑滚动
4. **正文卡片网格**（五版块固定顺序）：
   - **模型发布/更新** — 大模型发布、版本迭代、权重开源
   - **产品发布/更新** — AI 产品/工具/平台发布、功能更新、服务变更
   - **行业动态** — 商业合作、融资、政策、人事、市场数据
   - **论文研究** — 学术论文、研究突破、基准评测
   - **技巧与观点** — 使用技巧、教程、行业观点、深度分析
5. **底部**：总条数 + 数据源说明（有补充时标注「其中 N 条为编者补充」）+ 生成说明

### 4.1 卡片要素（每条新闻都要让人一眼看懂）

每张卡片固定包含：

- **全局序号**：跨版块连续编号（1、2、3…），不在版块内重新计数；编者补充条目排在所属版块末尾，继续全局编号
- **标题**：事件主体 + 动作（如「DeepSeek 预告 API 价格大幅上调」），一眼看出发生了什么
- **来源 chip**：标明信源渠道；媒体转载的标注媒体名（如「IT之家（RSS）」），官方的标注官方渠道（如「DeepSeek 官方公告」），**有作者的标注作者名**
- **摘要**：≤60 字中文，说明「谁 + 做了什么 + 为什么重要」；AI HOT 原文摘要超长时截断
- **原文跳转**：`target="_blank" rel="noopener noreferrer"`，默认跳原文（`links.original`），无原文跳 AI HOT 站内页
- **编者补充卡片**额外带橙色「编者补充」徽章 + 虚线边框 + 事件日期

### 4.2 详细简报要求

- 每个版块都要做成**详细简报**，不是标题列表：每条摘要必须交代清楚"谁、做了什么、影响是什么"
- Hero 统计让人扫一眼就知道今天五版块各有多少条、总量多少
- 版块内 AI HOT 条目在前、编者补充条目在后，来源泾渭分明

## 5. 数据源体系

### 5.1 AI HOT（主数据源）

- **接口**：`https://aihot.virxact.com/api/v1/dailies/latest`（匿名只读，无需 Key），当日未生成则按 skill 规则查 `/api/v1/dailies?limit=7` 索引取最近日期
- **信源构成**：官方博客/平台 RSS（NVIDIA、OpenAI、Google、Cursor、Databricks…）、科技媒体 RSS（The Verge、TechCrunch、IT之家…）、X 账号、微信公众号（千问、卡兹克…）、Hacker News 热门（buzzing.cc 中文翻译）、论文源（arXiv / HuggingFace Daily Papers）、GitHub Releases
- **整理流程**：持续抓取收录（`discoveredAt`）→ 原文时间归位（`publishedAt`，超 72h 历史回填归位原发布日）→ 编辑策展（全量池 → 精选池 → 日报）→ AI 辅助整理（lead、事件综述）
- **已知盲区（2026-08-07 实测）**：对**国产模型商业动态**（调价、融资、商业合作）覆盖滞后。典型案例：DeepSeek 8月6日涨价预告多家媒体当日报道，但 AI HOT 当日日报和全量 items 池均未收录
- **教训**：AI HOT 质量高但中文商业动态有滞后，**重要国产事件必须靠第 5.2 节官方信源交叉验证兜底**

### 5.2 官方信源补充（编者补充机制）

- 每日用 WebSearch 交叉验证「国内外知名 AI 公司」官方动态（模型发布、定价调整、服务变更、融资等官方公告/发布）
- **未被 AI HOT 日报收录的重要官方动态** → 直接合并进 `index.html`（最多 5 条），归入对应版块末尾并标注「编者补充」
- **已被 AI HOT 收录的不重复补录**

### 5.3 国内外知名 AI 公司官方信源清单

> 每日交叉验证优先级：★★★ 必查（核心头部）→ ★★ 重点（第二梯队）→ ★ 关注（重要事件时查）

**海外：**

| 公司 | 官方信源 | 优先级 |
| --- | --- | --- |
| OpenAI | openai.com/news（官方 Newsroom）、X @OpenAI、openai.com/blog | ★★★ |
| Anthropic | anthropic.com/news、X @AnthropicAI | ★★★ |
| Google / DeepMind | blog.google/technology/ai/、deepmind.google、X @GoogleDeepMind | ★★★ |
| Microsoft | blogs.microsoft.com、X @MSFTResearch、azure.microsoft.com/blog | ★★ |
| Meta AI | ai.meta.com、X @AIatMeta、Llama 开源仓库 | ★★ |
| xAI | x.ai、X @xai | ★★ |
| NVIDIA | blogs.nvidia.com、X @NVIDIAAI | ★★ |
| Amazon / AWS | aws.amazon.com/blogs/machine-learning/、Amazon Science | ★★ |
| Mistral AI | mistral.ai/news | ★ |
| Cohere | cohere.com/news | ★ |
| Perplexity | perplexity.ai/blog | ★ |
| Hugging Face | huggingface.co/blog（开源社区风向） | ★ |
| Databricks | databricks.com/blog（AI/ML 平台） | ★ |
| Apple | machinelearning.apple.com（Apple 机器学习） | ★ |
| Stability AI | stability.ai/news（开源视觉） | ★ |

**国内：**

| 公司 | 官方信源 | 优先级 |
| --- | --- | --- |
| DeepSeek（深度求索） | deepseek.com、api-docs.deepseek.com（API 公告/价格页）、公众号「DeepSeek 深度求索」 | ★★★ |
| 智谱 AI | zhipuai.cn、open.bigmodel.cn（智谱开放平台）、z.ai、公众号「智谱AI」 | ★★★ |
| 月之暗面（Moonshot） | moonshot.cn、platform.moonshot.cn（Kimi 开放平台）、X @MoonshotAI、公众号「月之暗面」 | ★★★ |
| 阿里（通义千问/Qwen） | tongyi.aliyun.com、qwen.ai、github QwenLM、公众号「通义千问」 | ★★ |
| 百度（文心/千帆） | yiyan.baidu.com、qianfan.cloud.baidu.com、公众号「文心一言」 | ★★ |
| 腾讯（混元） | hunyuan.tencent.com、腾讯云大模型、公众号「腾讯混元」 | ★★ |
| 字节跳动（豆包/火山引擎） | doubao.com、volcengine.com、公众号「豆包」 | ★★ |
| 科大讯飞（星火） | xinghuo.xfyun.cn、iflytek.com、公众号「科大讯飞」 | ★★ |
| MiniMax | minimaxi.com、公众号「MiniMax」 | ★★ |
| 阶跃星辰（Step） | stepfun.com、公众号「阶跃星辰」 | ★★ |
| 零一万物（Yi） | 01.ai、公众号「零一万物」 | ★ |
| 百川智能 | baichuan-ai.com、公众号「百川智能」 | ★ |
| 面壁智能（OpenBMB） | modelbest.cn、X @OpenBMB、公众号「面壁智能」 | ★ |
| 昆仑万维（天工） | kunlun.com、公众号「昆仑万维」 | ★ |
| 快手（可灵 Kling） | klingai.com、公众号「快手」 | ★ |

**交叉验证方法**：WebSearch 查询「公司名 + 官方/发布/公告 + 日期范围」，优先官方域名结果（openai.com、deepseek.com 等）；对重要数字/日期回官方原文核对，不引用二手转述为最终依据。

### 5.4 编者补充条目字段

```json
[
  {
    "category": "ai-models | ai-products | industry | paper | tip",
    "title": "事件主体 + 动作，一眼看懂",
    "summary": "≤60 字中文摘要，谁 + 做了什么 + 为什么重要",
    "source": "编者补充·公司官方渠道（或「媒体名 · 作者名」）",
    "url": "官方公告/原文链接",
    "date": "YYYY-MM-DD"
  }
]
```

`category` 对应五版块：`ai-models` 模型发布/更新、`ai-products` 产品发布/更新、`industry` 行业动态、`paper` 论文研究、`tip` 技巧与观点。

## 6. 首页渲染规范

- **允许写入范围**：每次执行最多写入一个旧页归档 `daily/YYYY-MM-DD[-HHmmss].html` 和最新页 `index.html`；不得调用旧生成脚本或写入其他 HTML
- **纯 HTML/CSS/JS 单文件**：样式与脚本全部内联，无任何外部资源（无 CDN、无字体、无图片外链）
- **视觉**：深色主题（`#0a0e1a` 背景 + 版块渐变强调色）、响应式 `auto-fill minmax(320px, 1fr)` 卡片网格、移动端单列
- **五版块配色**：模型发布/更新 靛蓝 `#6366f1`、产品发布/更新 粉 `#ec4899`、行业动态 青 `#06b6d4`、论文研究 橙 `#f59e0b`、技巧与观点 绿 `#10b981`
- **编者补充样式**：卡片虚线边框（`supp-card`）+ 橙色「编者补充」徽章（`supp-badge`）+ 事件日期小字
- **Hero**：`今日精选 N 条（含 M 条编者补充）`，数据覆盖窗口用北京时间人话格式
- **页脚**：总条数 + AI HOT 数据源链接 + 补充说明「国内外知名 AI 公司官方信源交叉验证，来源已逐条标注」
- **时间**：一律转北京时间人话格式（`8月6日 08:00`），不展示 ISO 字符串
- **模板契约**：`index.html` 的 `<body>` 保留 `data-template="signal-ledger-v1"`；锚点固定为 `models`、`products`、`industry`、`research`、`tips`，不生成随机 ID
- **结构稳定**：沿用 `template.html` 中的 hero、stats、sticky nav、section、card、supp-card、footer 类名与嵌套关系；不得新增第二套覆盖样式、不得用成片内联 `style` 重画页面
- **生成边界**：每日只替换标题、日期、覆盖窗口、统计、卡片内容、链接与页脚数据；模板中的设计 token、布局、动效和断点保持不变
- **交付自检**：`index.html` 中不得残留 `{{...}}` 占位符；同一页面只能有一个主样式块，且必须保留跳转正文链接、焦点样式和 `prefers-reduced-motion`

## 7. 每日执行流程

外部 AGENT 或人工执行时按以下顺序进行：

1. **准备**：读取 `template.html` 与现有 `index.html`，提取旧首页日报日期并确定本次新首页日期
2. **拉取**：调用 aihot skill 日报接口拿当日数据（回退逻辑见 5.1）
3. **交叉验证**：WebSearch 查第 5.3 节清单中 ★★★/★★ 公司当日官方动态（模型发布、定价调整、服务变更）
4. **补录**：未被 AI HOT 收录的重要官方动态 → 作为「编者补充」加入首页（≤5 条）
5. **归档**：新旧日期不同时，按第 3 节规则将当前 `index.html` 完整保存到 `daily/`；归档失败则停止
6. **生成**：归档成功后原位更新 `index.html` 的日期、内容与统计
7. **自检**：核对归档存在且未被改写，并检查新首页结构、统计、编号、锚点、时间与外链安全属性
8. **交付**：展示 `index.html`，简要说明当日重点、数据统计与归档路径

## 8. 手动工作流

1. **采集**：AI HOT 日报 + 官方信源交叉验证（老板提供的信息优先）
2. **分类**：按五版块归类；拿不准的先问
3. **归档**：读取旧首页日期，将旧 `index.html` 保存到 `daily/`；归档成功后再继续
4. **生成**：原位更新 `index.html`，并把页面日期更新为本次日期
5. **校验**：浏览器打开确认归档与新首页均可渲染（卡片、徽章、链接、编号）
6. **提交**：当日收工前同时提交 `index.html` 与新增归档，遵守第 9 节规范
7. **发布**：提交、推送或外部发布由用户另行安排，不属于日报生成步骤

## 9. Git 提交规范

遵循 Conventional Commits，AI 日报建议前缀：

| 前缀    | 用途            | 示例                       |
| ----- | ------------- | ------------------------ |
| `docs:` | AI 日报内容更新      | `docs: 更新 AI 日报首页` |
| `feat:` | 日报模板/结构升级     | `feat: AI 日报增加编者补充机制`     |
| `fix:`  | 修正内容/样式错误     | `fix: 修正 AI 日报链接失效` |
| `chore:` | 规则/配置变更       | `chore: 更新 AGENTS.md`    |

- 一条提交只做一件事，不把多天改动混在一个提交里。
- 默认 `main` 分支，单人项目不引入分支策略。
- 提交信息中文描述主体内容。

## 10. 常用命令速查

```bash
# 查看未提交改动
git status
# AI 日报首页与归档提交
git add index.html daily/ && git commit -m "docs: 更新 AI 日报首页并归档旧版"
# 查看首页历史
git log --oneline --stat -- index.html
# 查看日归档
git log --oneline --stat -- daily/
# 回看某次首页
git show <commit>:index.html
```

## 11. 助手行为约束

- **扫描权限**：只允许读取本项目 `AI/` 内文件；用户未明确允许的项目外文档一律不扫描。用户说“不允许”的即为禁区。
- 修改任何 HTML 前，先读取当前文件内容再编辑，禁止整文件无脑覆盖。
- 不确定的事先问，不替老板做信息分类的主观判断。
- 对外动作（发布、分享、发消息）先确认；对内动作（读、整理、学习）可大胆。
- 当日事当日记，漏记不补账。
- **数据源纪律**：仪表盘主数据必须是 AI HOT；非 AI HOT 数据只允许以「编者补充」形式出现，来源逐条标注，保持可追溯。

## 12. 手动收工检查清单

> 本节仅供人工维护与发布时使用。第 13 节的外部 AGENT 执行提示词在更新 `index.html` 后即结束，不执行 Git 操作。

```
[ ] 旧首页日期与新首页日期不同时，旧页已完整归档到 daily/ 且未覆盖已有历史文件
[ ] index.html 已原位更新且内容完整（五版块 + 全局编号 + 来源标注）
[ ] 页面内日期、统计、编者补充数量与正文一致
[ ] git status 无遗漏的未提交改动
[ ] git add 后提交，提交信息符合规范
[ ] （可选）git push 到远程仓库备份
```

## 13. 执行提示词（供外部 AGENT / 调度器使用）

> 以下提示词只负责归档旧首页并生成最新日报，不负责创建或配置任何自动化任务，也不执行 Git 提交、推送或外部发布。

```text
你正在维护「AI 日报」独立项目。按 Asia/Shanghai 当天日期执行，并严格遵守以下要求。

### 一、开始前

1. 工作范围仅限 `AI/`。开始时完整读取 `AI/AGENTS.md`，并以其中最新规则为准。
2. 修改前必须完整读取 `AI/template.html` 与现有 `AI/index.html`。`template.html` 是只读视觉与 DOM 基准；保留 `data-template="signal-ledger-v1"`、固定锚点、CSS 类、响应式布局、无障碍属性和有效交互，只把当天内容与统计写入 `index.html`，不得另起一套样式。
3. 本次只允许写入最新页 `AI/index.html`，以及按规则产生的一个旧页归档 `AI/daily/YYYY-MM-DD[-HHmmss].html`。允许在缺失时创建 `AI/daily/`；不得修改 `AI/template.html`，不得在项目根目录创建日期 HTML，不得创建其他归档目录、`supplements.json`、Python 生成脚本或其他文件，也不得调用旧生成脚本。
4. 不执行 `git add`、`git commit`、`git push`，不发布到任何外部平台。

### 二、数据获取与交叉验证

1. 调用 aihot skill 的日报接口 `/api/v1/dailies/latest`。若当天日报尚未生成或接口返回 404，按该 skill 的规则查询 `/api/v1/dailies?limit=7` 并选择最近一期。页面必须如实标注实际数据日期与北京时间覆盖窗口，不得把最近一期伪装成当天数据。
2. 对照 `AI/AGENTS.md` 第 5.3 节的完整官方信源清单，用 WebSearch 检查当天国内外知名 AI 公司官方动态，聚焦模型发布/更新、产品或服务变更、定价调整、融资及其他重要官方公告。优先官方域名和一手公告，并核对关键日期、数字与链接。
3. 若重要官方动态未被 AI HOT 收录，可作为「编者补充」直接并入 `AI/index.html`，最多 5 条；已被 AI HOT 收录的不得重复。补充条目需包含对应 `category`、事件主体+动作式标题、≤60 字中文摘要、以「编者补充·」开头的来源、官方原文 URL 和事件日期，并排在所属版块末尾。
4. 不创建 `supplements.json`。当天没有合格补充时写 0 条，不凑数。
5. 所有条目必须有可访问、可追溯的原文链接；不得凭记忆编写事实。

### 三、归档旧首页

1. 新首页的日报日期使用本次执行时的北京时间日期；若 AI HOT 回退到最近一期，必须把数据实际日期另行标清，但首页日期仍更新为本次执行日期。
2. 从当前 `AI/index.html` 的 `<title>`、Hero 或明确日期字段中提取旧首页自身的日报日期。无法可靠提取时立即停止，不得猜测日期或覆盖首页。
3. 旧首页日期与新首页日期不同时，先把当前 `AI/index.html` 的完整内容原样保存为 `AI/daily/<旧首页日期>.html`，归档成功后才允许更新 `AI/index.html`。
4. 若目标归档已存在且内容完全一致，保留原文件并视为归档完成；若内容不同，禁止覆盖已有归档，将当前旧首页保存为 `AI/daily/<旧首页日期>-HHmmss.html`，后缀使用归档时的北京时间。若带后缀路径仍重名，重新取当前北京时间生成唯一文件名；仍冲突则停止，绝不覆盖。
5. 旧首页日期与新首页日期相同时属于同日修订，不新增归档文件，直接更新当前首页。
6. 归档必须保留旧页完整 HTML/CSS/JS 和旧内容，不得套用新数据、不得修改 `AI/template.html`。归档写入失败时停止任务并保持 `AI/index.html` 不变。

### 四、首页内容与视觉

1. 固定按以下顺序组织五个版块：模型发布/更新、产品发布/更新、行业动态、论文研究、技巧与观点。
2. 条目跨版块全局连续编号，不在版块内重新计数；编者补充继续全局编号。
3. 更新 `<title>`、Hero 日期、北京时间数据覆盖窗口、总条数、五版块统计、编者补充数量、sticky 锚点导航、正文和页脚，确保所有数字彼此一致。
4. 每张卡片必须包含：序号；一眼看懂的「事件主体+动作」标题；来源 chip（媒体、官方渠道，能确认作者时注明作者）；≤60 字中文摘要（谁、做了什么、为什么重要）；北京时间人话格式的时间；原文链接。
5. 所有外链使用 `target="_blank" rel="noopener noreferrer"`。编者补充卡片沿用现有 `supp-card` / `supp-badge` 样式，并明确显示事件日期。
6. 保持纯 HTML/CSS/JS 单文件、样式和脚本内联、无 CDN、无外部字体、无外部图片资源，以及现有暗色响应式卡片布局。
7. 页脚标注总条数与 AI HOT 数据源；有补充时增加：「其中 N 条为编者补充：国内外知名 AI 公司官方信源交叉验证，来源已逐条标注」。
8. 严格沿用 `AI/template.html` 的单一主样式块和 DOM 类名；锚点使用 `models`、`products`、`industry`、`research`、`tips`，不得生成随机 `sec-xxxxx`，不得残留 `{{...}}` 模板占位符。

### 五、完成前检查与交付

1. 确认写入范围仅包括应有的 `AI/daily/` 旧页归档和 `AI/index.html`；两者均为 UTF-8，HTML 结构完整。
2. 核对 Hero、导航、各版块、全局编号、卡片数量与页脚统计一致。
3. 检查锚点目标、原文 URL、`target` 与 `rel` 属性；页面不得直接展示 ISO 时间字符串。
4. 某版块没有合格条目时保留版块结构并如实显示 0 条，不伪造内容。
5. 最终回复简要说明当天重点、总条数、五版块数量、编者补充数量、新首页路径 `AI/index.html`，以及本次归档路径；同日修订未归档时明确说明。不要声称已提交、推送或发布。
```

---

*版本：3.3 · 2026-08-07 更新：恢复 `daily/` 历史归档，新一期仍固定写入 `index.html`；3.2 新增只读 `template.html` 视觉基准与稳定 DOM 契约*

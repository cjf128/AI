<div align="center">

# AI 日报

**AI Signal Ledger — 面向公开阅读的每日 AI 资讯简报**

[![Static HTML](https://img.shields.io/badge/Static-HTML5-E34F26?logo=html5&logoColor=white)](./index.html)
[![Dependencies](https://img.shields.io/badge/dependencies-0-2ea44f)](./index.html)
[![Language](https://img.shields.io/badge/language-zh--CN-6366f1)](./index.html)

[**在线阅读最新一期**](https://cjf128.github.io/AI/) · [查看项目工作流](./AGENTS.md)

</div>

> 从分散的信息流中提取真正值得关注的信号：发生了什么、为什么重要、原始来源在哪里。

## 项目简介

AI 日报是一份零依赖、可追溯的中文 AI 资讯简报。最新一期固定发布在 `index.html`，以五个版块组织模型、产品、行业、研究与实践信息，并为每条内容保留原文入口。

它不仅是一张新闻标题列表，更强调三件事：

- **快速理解**：摘要交代事件主体、关键动作及影响。
- **来源透明**：优先链接一手来源，媒体转载与官方公告清楚标识。
- **稳定阅读**：单文件静态页面，无框架、无构建步骤、无运行时依赖。

## 内容版图

| 版块 | 关注内容 | 主题色 |
| --- | --- | --- |
| 模型发布 / 更新 | 大模型发布、版本迭代、权重开源 | 靛蓝 `#6366f1` |
| 产品发布 / 更新 | AI 产品、工具与平台的发布或功能更新 | 粉色 `#ec4899` |
| 行业动态 | 商业合作、融资、政策、人事与市场变化 | 青色 `#06b6d4` |
| 论文研究 | 学术论文、研究进展与基准评测 | 橙色 `#f59e0b` |
| 技巧与观点 | 使用技巧、教程、深度分析与行业观点 | 绿色 `#10b981` |

## 阅读体验

- 深色信号台视觉，桌面端卡片网格与移动端单列自适应。
- Sticky 版块导航、滚动进度提示和清晰的键盘焦点状态。
- 全局连续编号、版块统计与更新时间，方便快速浏览。
- 「编者补充」使用独立徽章和虚线边框，与主数据源明确区分。
- CSS 与 JavaScript 全部内联，可直接保存、归档或离线打开。

## 数据与编辑原则

主数据来自 [AI HOT](https://aihot.virxact.com/)，并对国内外重要 AI 公司的官方动态进行交叉核验。未被主数据源覆盖但值得收录的事件，可作为「编者补充」加入，每期最多 5 条，并逐条注明日期与来源。

编辑过程遵循：

```text
采集 → 交叉核验 → 分类策展 → 摘要整理 → 页面校验 → 历史归档
```

完整的数据源清单、卡片字段、归档策略与质量要求见 [AGENTS.md](./AGENTS.md)。

## 项目结构

```text
AI/
├── index.html       # 最新一期，也是 GitHub Pages 入口
├── template.html    # 只读视觉与 DOM 模板
├── AGENTS.md        # 数据、编辑、生成与归档规则
├── README.md        # 项目说明
└── daily/           # 历史日报（首次换期后创建）
    └── YYYY-MM-DD.html
```

`index.html` 始终代表最新一期。生成新一期前，旧页面会按其自身日期归档到 `daily/`；同日再次生成则视为修订，不重复创建归档。

## 本地查看

无需安装依赖或启动开发服务器。在 Windows PowerShell 中运行：

```powershell
Start-Process .\index.html
```

也可以直接双击 `index.html`。如需维护项目，请先阅读 [AGENTS.md](./AGENTS.md)，并保持 `template.html` 的结构、响应式行为和无障碍约束不变。

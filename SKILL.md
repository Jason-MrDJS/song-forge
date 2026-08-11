---
name: song-forge
description: >
  歌曲流水线总控（v1.0）· 把 lyric-writer（歌词创作）、bmz（歌词质检）、lyria-music-producer（Suno/Lyria 双槽 prompt 编译）
  三个技能串成一条闭环流水线：写初稿 → 双评分质检 → 未达标自动回炉改写 → 达标后编译 prompt。
  支持中英双语分流，用单一真相文件 TrackSpec.md 传递数据，各阶段只写自己的区段，无格式断层。
  内置 Revision.Directive 桥接件（让「质检只诊断不代写」与「自动改写闭环」同时成立）、
  伪控制标签强制翻译层（拦截 Energy Start:% 等 AI 自创百分比）、回环终止与反循环保护。
  关键词：写歌、歌词创作、歌词质检、歌词打磨、出 Suno 提示词、AI 音乐流水线、原创 demo、一条龙写歌。
version: "1.0"
agent_created: true
---

# song-forge · 歌曲流水线总控 v1.0

你是歌曲生产流水线的**编排者**。你自己不写歌词、不做质检判断、不编 prompt——这三件事分别由三个专业技能完成。你负责：判定参数、维护 TrackSpec、按序调度、编译改写指令、拦截违规输出、控制回环终止。

## 何时使用

- 用户说「帮我写首歌」「我有个主题想写歌」「把这段词打磨到能生成」
- 用户给了歌词草稿，想一路推进到可用的 Suno / Lyria 提示词
- 用户要「一条龙」从概念到提示词

**不适用**：
- 纯配乐 / 纯器乐 / BGM（无人声）→ 用 `lyria-scoring-prompt`
- 只想单点质检一段词，不要后续 → 直接用 `bmz`
- 已有成品词只想要 prompt → 直接用 `lyria-music-producer`
- 发布前体检 / 发布包 / 数据复盘 → 用 `bmz`

## 架构

```
                song-forge（你，编排层）
                        │
    ┌───────────────────┼───────────────────┐
    ▼                   ▼                   ▼
① lyric-writer    ② bmz 子代理      ③ lyria-music-producer
 写初稿 / 改写      歌词质检双评分       双槽 prompt 编译
 唯一持代写权       只诊断 · 不代写      输出经翻译层拦截
    └───────────────────┴───────────────────┘
                        │
              TrackSpec.md（单一真相文件）
```

**铁律：没有任何两个阶段写入同一个 TrackSpec 区段。**

| 阶段 | 执行者 | 读 | 写 | 禁止 |
|---|---|---|---|---|
| S0 立项 | 你 | — | Meta、Brief | 不写歌词 |
| S1 创作 | lyric-writer | Meta、Brief、Revision.Directive | Lyrics.Draft、Pronunciation.Notes | 不自评分 |
| S2 质检 | bmz 子代理 | Meta、Lyrics.Draft | QC.Report、QC.Verdict | 不改写歌词 |
| S3 编译 | 你 | QC.Report | Revision.Directive | 不新增创作内容 |
| S4 编 prompt | producer | Meta、Lyrics.Draft | Anchors、Prompt.Style、Prompt.Structure | 不评判歌词质量 |
| S5 交付 | 你 | 全部 | Changelog | — |

## 执行流程

### S0 · 立项（必做，不可跳过）

判定四个必需参数，缺任何一个都要先追问，**不要猜**：

| 参数 | 取值 | 判定线索 |
|---|---|---|
| `lang` | `zh` / `en` | 用户输入语言、明确指定、目标平台 |
| `mode` | 固定 `song` | 若发现是纯器乐 → 停止，路由到 `lyria-scoring-prompt` |
| `target_engine` | `suno-v4` / `lyria-3.5` | 用户指定；未指定默认 `suno-v4` |
| 起点 | 只有主题 / 已有草稿 | 只有主题走 S0.5，有草稿直接进 S2 |

用 `templates/trackspec.template.md` 建 TrackSpec 文件，填入 Meta。曲风 / BPM / 调性 / 时长 / 人声性别若用户未给，可先留空，由 S4 阶段的 producer 推导后回填。

**S0.5（可选）**：用户只有一句主题时，先 spawn bmz 创作启发子代理产出 `Brief`：
- 系统提示：`~/.workbuddy/skills/bmz/templates/songwriting.prompt.md`，填 `{{USER_INPUT}}` = 主题，`{{CONTEXT}}` = 已知参数
- 知识库：`~/.workbuddy/skills/bmz/knowledge/framework.md` + `~/.workbuddy/skills/bmz/knowledge/songwriting.md`
- 产出目标听众、3 个可写方向、8 句主打段功能骨架 → 写入 `Brief`

### S1 · 创作（lyric-writer）

调用 `lyric-writer`，**调用时必须显式声明三件事**：

1. `跳过 SKILL.md 第 8 步的 suno-engineer 自动移交，本次由 song-forge 接管后续`
   （否则它会尝试调用本机不存在的 `/bitwize-music:suno-engineer`）
2. `不要调用 load_override()，风格约束已在下方给出`
   （该函数本机不存在）
3. `不要读取 ${CLAUDE_PLUGIN_ROOT}/reference/suno/pronunciation-guide.md，改用 song-forge 的 references/pronunciation-zh-en.md`

**若 `lang: zh`**：必须同时注入 `references/bilingual-qc-matrix.md` 的中文替代规则。lyric-writer 原生 13 点中有 9 项在中文下需要口径替换或重写，不注入会大面积空转。

**若存在 `Revision.Directive`**：把它整段作为改写指令传入，并强调 `preserve` 段不得破坏。

产物写入 `Lyrics.Draft` 与 `Pronunciation.Notes`。

### S2 · 质检（bmz 子代理）

**不要经过 bmz 主 Agent**。直接 spawn 一个干净子代理：

- 系统提示：`~/.workbuddy/skills/bmz/templates/lyrics-check.prompt.md`
  - `{{USER_INPUT}}` = `Lyrics.Draft` 全文
  - `{{CONTEXT}}` = Meta 中的曲风、目标平台、语言、当前版本号
- 知识库：`~/.workbuddy/skills/bmz/knowledge/framework.md` + `~/.workbuddy/skills/bmz/knowledge/lyrics-check.md` + `~/.workbuddy/skills/song-forge/references/lyrics-check-en.md`（英文资产补件）

**必须完整保留子代理的输出**，原样写入 `QC.Report`，不压缩成摘要。从中抽取双评分与证据等级写入 `QC.Verdict`。

**若 `lang: en`**：以 `lyrics-check-en.md` 为英文主资产（避雷清单 + 好歌词机制 + 原创性判断），bmz 中文资产作反向避雷参考（如中文「鸡汤落点」对应 `bad-over-philosophical`）。双评分与证据等级口径与中文一致，**不再降级**——原「因缺英文资产而降 B」的临时声明已废弃。

### S3 · 判定与编译

读 `QC.Verdict`，按下表判定：

| 条件 | 动作 |
|---|---|
| 传播 ≥ 7.0 且 歌词 ≥ 6.5 | `verdict: pass` → 进 S4 |
| 原创性状态 = 确认现存歌词 | `verdict: halt` → 立即停止，输出版权风险，不进生成 |
| 未达标且 `revision` < 3 | `verdict: revise` → 编译 Revision.Directive，`revision++`，回 S1 |
| `revision` >= 3 | `verdict: halt` → 输出诊断 + 建议换方向，不再自动改 |
| 连续两轮「最大问题」重合 ≥ 2/3 | `verdict: halt` → 反循环保护，判定改写无效 |

阈值来源与局限见 `references/revision-directive.md`。**首轮建议把阈值作为参考而非硬闸门，向用户确认是否放行。**

编译 `Revision.Directive` 严格按 `references/revision-directive.md` 的格式与铁律执行。核心约束：**只能包含约束与方向，不能包含可直接粘贴的成品歌词行**。

### S4 · 编译 prompt（lyria-music-producer）

调用 `lyria-music-producer`，传入 Meta 全量参数 + `Lyrics.Draft`。让它走完整的 3 锚点 → 8 层推导 → 双槽编译。

锚点结果写入 `Anchors`（保证可追溯），双槽输出**先经 S4.5 拦截再落盘**。

### S4.5 · 伪标签强制拦截（不可跳过）

对 producer 的双槽原始输出执行扫描替换，规则见 `references/pseudo-tag-filter.md`。

**必查项**：`Energy Start: N%`、`Mood: N%`、`Intensity: N%`、`Emotion: N%`、`energy peaks at N%`、`peel to N%`。

全部替换为音乐语言（乐器密度 / 编曲规模 / 动态变化 / 演奏强度）。

**允许保留的数字仅有**：BPM、Key、拍号、Duration（秒）、Range（音域）、小节数。

替换后写入 `Prompt.Style` 与 `Prompt.Structure`。

> 背景：producer 已升 v1.3 根改源文件（新增音乐导演语言转换层 Step 5.5 + 硬规则 R26/R27），双槽默认不再含伪标签。本层保留为防御性安全网，正常情况下为 no-op 透传；仅当未来其他路径意外产出伪标签时才触发替换。

### S5 · 交付

输出交付卡：

```
【歌曲】{title}｜{lang}｜{genre}｜{bpm}bpm｜{key}

── 质检结果 ──
传播评分 {x}/10　歌词评分 {y}/10　证据等级 {S/A/B}
迭代轮次：v{n}（共回炉 {n-1} 次）
最大遗留风险：{一句话}

── Output 1 · Style Prompt（贴进 Prompt Box）──
{Prompt.Style}

── Output 2 · Structure Blueprint（贴进 Lyrics Box）──
{Prompt.Structure}

── 版本历史 ──
{Changelog 表}
```

## 硬规则

**F1 · 阶段不越权**：你永远不写歌词正文、不给质检分数、不编 prompt 内容。发现自己在做这三件事，立即停手改为调度。

**F2 · 质检报告不压缩**：bmz 子代理的输出必须完整保留在 `QC.Report`。压缩成摘要会丢失逐句审查与案例对照依据，这是 bmz 的核心价值。

**F3 · Directive 不含成品句**：编译改写指令时若产出了可直接粘贴的完整歌词行，说明越界，必须重编译。代写权只在 lyric-writer 手里。

**F4 · 单一 focus**：每轮 `Revision.Directive` 只设一个主攻方向。对应 bmz B002「一轮测试只改一个核心变量」。

**F5 · 伪标签零容忍**：S4.5 不可跳过，无论 `target_engine` 是什么。

**F6 · 回环硬上限**：最多 2 轮回炉（v1/v2/v3）。第 3 版不过即 halt，不得继续。

**F7 · 版权优先**：`QC.Report` 判定「确认现存歌词」时立即 halt，优先级高于一切其他判定。

**F8 · 弱判断要声明**：无法联网查重时，必须原样传递 bmz 的声明「当前无法联网核验是否为现存歌词，本次原创性只做弱判断」。

**F9 · 中文必注入替代规则**：`lang: zh` 时不注入双语矩阵就调 lyric-writer，等于让 9 项检查空转。

**F10 · 断链三声明**：调 lyric-writer 时必须声明跳过 suno-engineer 移交、不调 load_override、改用本地发音表。

## References

- `references/trackspec-schema.md` —— TrackSpec 完整字段规格与驱动关系
- `references/bilingual-qc-matrix.md` —— 中英双语质检映射矩阵（S1/S2 必查）
- `references/revision-directive.md` —— 桥接件编译规格、阈值来源与局限
- `references/pseudo-tag-filter.md` —— 伪标签拦截规则与翻译对照表
- `references/lyrics-check-en.md` —— 英文歌词质检资产（避雷清单 + 好歌词机制 + 原创性判断，闭合 §7.2 缺口）
- `templates/trackspec.template.md` —— TrackSpec 空白模板

外部依赖（只读，不修改）：
- `~/.workbuddy/skills/lyric-writer/` —— 创作引擎
- `~/.workbuddy/skills/bmz/knowledge/` + `templates/` —— 质检知识库与提示词模板
- `~/.workbuddy/skills/lyria-music-producer/` —— 双槽编译引擎

设计文档：`G:\work666\song-forge\DESIGN.md`

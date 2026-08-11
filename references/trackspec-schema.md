# TrackSpec 数据契约规格

TrackSpec 是流水线的**单一真相文件**。三个技能原生 I/O 互不兼容，直接对接必然丢字段，因此所有跨阶段数据一律经此文件传递。

## 为什么必须有它

| 技能 | 原生输入 | 原生输出 |
|---|---|---|
| lyric-writer | 带 frontmatter 的 track 文件，或一句概念 | 歌词散落在 Lyrics Box / Streaming Lyrics / Pronunciation Notes 多处 |
| bmz 子代理 | 仅 `{{USER_INPUT}}` + `{{CONTEXT}}` 两个纯文本占位符 | 固定格式的质检报告文本 |
| lyria-music-producer | 自然语言创意描述 | 两段互不混合的文本块 |

## Meta 字段规格

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `track_id` | string | 是 | 格式 `YYYY-MMDD-NNN` |
| `title` | string | 否 | 可在流程中确定 |
| `lang` | enum | 是 | `zh` / `en`，驱动质检分流 |
| `mode` | enum | 是 | 固定 `song`。若为 score/bgm-loop 应改走 lyria-scoring-prompt |
| `target_engine` | enum | 是 | `suno-v4` / `lyria-3.5` |
| `target_platform` | array | 否 | 如 `[douyin, xiaohongshu]` |
| `genre` | string | 否 | 未填由 S4 推导回填 |
| `sub_genre` | string | 否 | |
| `bpm` | number | 否 | 影响字数密度判定 |
| `key` | string | 否 | |
| `time_signature` | string | 否 | 默认 `4/4` |
| `target_duration` | number | 否 | 秒。影响字数上限 |
| `vocal_gender` | string | 否 | |
| `vocal_character` | string | 否 | 如 breathy / raspy |
| `audience` | string | 否 | 由 Brief 阶段产出 |
| `instrumental` | bool | 是 | 固定 `false` |
| `revision` | number | 是 | 从 1 起，回环自增 |
| `status` | enum | 是 | drafting / qc / revising / compiling / done / halted |

## 区段写入权限表

| 区段 | 唯一写入者 | 读取者 |
|---|---|---|
| `Meta`（frontmatter） | song-forge | 全部 |
| `Brief` | song-forge（或 bmz 创作启发） | lyric-writer |
| `Lyrics.Draft` | lyric-writer | bmz、producer |
| `Lyrics.Streaming` | lyric-writer | 用户 |
| `Pronunciation.Notes` | lyric-writer | producer |
| `QC.Report` | bmz 子代理 | song-forge |
| `QC.Verdict` | bmz 子代理 | song-forge |
| `Revision.Directive` | song-forge | lyric-writer |
| `Anchors` | producer | song-forge |
| `Prompt.Style` | producer（经拦截） | 用户 |
| `Prompt.Structure` | producer（经拦截） | 用户 |
| `Changelog` | song-forge | 用户 |

**违反此表即为断层。** 例如 bmz 若写了 `Lyrics.Draft`，就是越权代写。

## 字段驱动关系

| 字段 | 下游影响 |
|---|---|
| `lang` | S1 注入哪套创作规范；S2 启用哪套质检项；S2 是否追加证据降级声明 |
| `target_engine` | S4 双槽语法细节；S4.5 拦截强度（两者都拦，lyria 更严） |
| `bpm` + `target_duration` | 字数上限判定（见 bilingual-qc-matrix.md §3） |
| `revision` | 回环计数器，>= 3 触发 halt |
| `status` | 支持中断续跑，重入时从当前状态继续 |

## Lyrics.Draft 与 Lyrics.Streaming 的区别

- `Lyrics.Draft`：进引擎用。含 `[段落]` 标签、括号编曲指令、发音替换后的拼写。
- `Lyrics.Streaming`：分发平台用。标准拼写，无标签，无发音替换。

两者由 lyric-writer 同时产出。**发音替换只能出现在 Draft，不能污染 Streaming。**

## Lyrics.Draft 与 Prompt.Structure 的边界

容易混淆，必须区分：

| | Lyrics.Draft | Prompt.Structure |
|---|---|---|
| 归属 | 歌词内容 | 编曲蓝图 |
| 写入者 | lyric-writer | producer |
| 含歌词文本 | 是 | 是（引用 Draft） |
| 含括号编曲指令 | 否 | 是 |
| 含 `[track:]` 容器标签 | 否 | 是 |

producer 在 S4 把 `Lyrics.Draft` 的文本嵌入自己生成的段落骨架，形成 `Prompt.Structure`。**它不修改歌词字句**，只添加结构标签与括号指令。

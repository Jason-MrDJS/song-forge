# Revision.Directive · 桥接件编译规格

> 适用阶段：S3 判定与编译
> 用途：解决 `bmz「只诊断不代写」` 与 `自动化改写闭环` 的表面矛盾。bmz 只产出诊断；`song-forge` 把诊断编译成机器可执行的改写指令；`lyric-writer` 依指令改写（代写权始终在它手里）。三方职责零重叠。

---

## 1. 它解决什么

`bmz/knowledge/lyrics-check.md` 的硬约束：

> 歌词质检只负责诊断，不直接代写；下一步应给用户提供下一版的创作方向、结构骨架、句子功能、关键词池和避雷清单，由用户自己写下一版。

自动化闭环要求「质检不过 → 自动改写」。解法是**插入一层编译**：bmz 出诊断 → song-forge 编译成 Directive → lyric-writer 改写。没有任何一方越界。

---

## 2. 编译映射表

`song-forge` 从 `QC.Report` 抽取以下字段，映射为 `Revision.Directive`：

| QC.Report 来源字段 | 编译为 | 交给 lyric-writer 的形式 |
|---|---|---|
| 最大问题 1-3 | `must_fix` | 必须修复项，逐条对应 |
| 最小修改动作 1-3 | `actions` | 具体动作指令 |
| 优先级 | `focus` | 本轮唯一主攻方向 |
| 逐句审查 → 修改方向 | `line_edits` | 定位到具体行的改写要求 |
| 高频风险词 → 命中词 | `banned_words` | 本轮禁用词表 |
| 坏歌词风险对照 → 命中的负向模式 | `avoid_patterns` | 避雷清单 |
| 好歌词机制对照 → 可学习点 | `reference_mechanics` | 正向参照，非模板 |

---

## 3. 格式规范

```markdown
## Revision.Directive
revision_target: 2
focus: 先改第一口

must_fix:
1. 前两句未锁定具体听众，泛化为"所有人"
2. 副歌金句理解成本过高
3. V2 与 V1 语义重复，未推进叙事

actions:
1. 把开篇改为具体场景 + 具体关系，禁止抽象抒情起手
2. 副歌首句降到 12 字以内，用日常词替换书面词
3. V2 引入新的时间点或新的物件细节

line_edits:
- L1「原句」→ 要求：加入具体人称与动作
- L7「原句」→ 要求：删除堆叠意象，保留一个

banned_words: [遗憾, 温柔, 星辰, 救赎]
avoid_patterns: [bad-002 空泛伤痛没有具体事件, bad-012 前两句无法锁定听众]
reference_mechanics: [good-017 的第一口锁定机制]

preserve:
- 保持第一人称
- 保持现有 BPM 下的字数密度
```

---

## 4. 铁律（编译时不可违反）

- **F3**：`Revision.Directive` **只能包含约束与方向，不能包含成品句子**。若编译结果里出现可直接粘贴的完整歌词行，说明越界，必须重编译。代写权只在 `lyric-writer` 手里。
- **F4**：每轮 directive 只设**一个** `focus`。对应 bmz B002 方法卡「一轮测试只改一个核心变量」。
- `preserve` 段用于防止 `lyric-writer` 的 refinement pass 破坏已通过的部分——把上一轮已达标的内容列进去，禁止改动。
- `reference_mechanics` 只给「机制参照」，不给「模板句子」。禁止写成「仿照 good-017 写出类似的 X 句」。

---

## 5. 阈值来源与局限（判定放行用）

| 指标 | 放行线 | 依据 |
|---|---|---|
| 传播评分 | ≥ 7.0 | 借用 `bmz/knowledge/lyrics-check.md` 对「市场已验证作品」设定的评分下限 |
| 歌词评分 | ≥ 6.5 | 同上 |

> **诚实声明**：这两个数字在 bmz 原文里的用途是「防止把已验证的大流量作品误判为低质」，本设计将其借用为「可进入生成」的门槛。这是类比推定，**证据等级 B**，需要用真实发布数据回校。首轮建议把阈值当参考而非硬闸门，由用户人工确认放行。

---

## 6. 判定分流（S3 输出）

| 条件 | 动作 |
|---|---|
| 传播 ≥ 7.0 且 歌词 ≥ 6.5 | `verdict: pass` → 进 S4 |
| 原创性状态 = 确认现存歌词 | `verdict: halt` → 立即停止，输出版权风险 |
| 未达标且 `revision` < 3 | `verdict: revise` → 编译 Directive，`revision++`，回 S1 |
| `revision` >= 3 | `verdict: halt` → 输出诊断 + 建议换方向，不再自动改 |
| 连续两轮「最大问题」重合 ≥ 2/3 | `verdict: halt` → 反循环保护，判定改写无效 |

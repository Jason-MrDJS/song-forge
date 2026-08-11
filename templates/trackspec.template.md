---
track_id: 2026-0000-000
title: 
lang: zh                    # zh | en，驱动质检分流
mode: song                  # 本流水线固定 song；若为 score/bgm-loop 应改走 lyria-scoring-prompt
target_engine: suno-v4      # suno-v4 | lyria-3.5，驱动伪标签策略
target_platform: []        # douyin | xiaohongshu | youtube | spotify ...
genre: 
sub_genre: 
bpm: 
key: 
time_signature: 4/4
target_duration: 180        # 秒
vocal_gender: 
vocal_character: 
audience: 
instrumental: false
revision: 1                 # 当前版本号，回环时自增（最多到 3）
status: drafting            # drafting | qc | revising | compiling | done | halted
---

## Brief
（创作简报：主题、听众、想表达什么、参考方向。由 S0.5 创作启发子代理产出，或用户直接提供。）

## Lyrics.Draft
（当前版本歌词正文，带 [段落] 标签。由 lyric-writer 写入，是唯一可进 S2 质检的歌词源。）

[Verse 1]


[Chorus]


[Verse 2]


[Bridge]


[Outro]


## Lyrics.Streaming
（标准拼写版，用于分发平台，不做发音替换。与 Lyrics.Draft 内容一致但无发音注解。）

## Pronunciation.Notes
（多音字 / 数字 / 英文缩写读法决策表，由 lyric-writer 写入。）
| 原词 | 读法 | 类型 | 决策来源 |

## QC.Report
（bmz 子代理产出的完整质检报告，原样保留，不压缩。由 S2 写入。）

## QC.Verdict
verdict: pass | revise | halt
传播评分: 0.0
歌词评分: 0.0
证据等级: S | A | B

## Revision.Directive
（总控 S3 编译的改写指令。见 references/revision-directive.md。只能含约束与方向，不含成品句。）
revision_target: 2
focus: 
must_fix:
- 
actions:
- 
line_edits:
- 
banned_words: []
avoid_patterns: []
reference_mechanics: []
preserve:
- 

## Anchors
（producer 3 锚点结果，保证可追溯。由 S4 写入。）
声学身份锚: 
发展逻辑锚: 
频谱禁区锚: 

## Prompt.Style
（Output 1，贴进 Suno/Lyria 的 Prompt Box。经 S4.5 伪标签拦截后落盘。）

## Prompt.Structure
（Output 2，贴进 Lyrics Box。含 [Section] 标签与段落排布。经 S4.5 拦截后落盘。）

## Changelog
| 版本 | 时间 | 动作 | 传播分 | 歌词分 | 证据等级 |
|---|---|---|---|---|---|
| v1 |  | 初稿 |  |  |  |

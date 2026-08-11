# 伪控制标签强制翻译层（pseudo-tag-filter）

> 适用阶段：S4.5（不可跳过，F5 铁律）
> 用途：`lyria-music-producer` 已升 v1.3 根改源文件，不再输出 `Energy Start: %` 等 AI 自创百分比标签。本层保留为**防御性安全网（当前为 no-op）**：若未来任何其他路径产出伪标签，仍在此拦截；正常情况下双槽直接透传。

---

## 1. 问题

用户既定规矩：

> AI 音乐 Skill 的输出绝不写 `Energy Start: %` / `Mood: %` / `Intensity: %` / `Emotion: %` 这类 AI 自创百分比。

producer v1.2.1 的 Suno v4 输出路径会主动前置 `Energy Start: N%`。无论 `target_engine` 是什么，S4.5 都不可跳过（F5）。

---

## 2. 扫描替换规则

对 `Prompt.Style` 与 `Prompt.Structure` 执行扫描替换：

| 检出模式 | 处理 |
|---|---|
| `Energy Start: N%` | 删除，改写为音乐语言描述 |
| `Mood: N%` / `Intensity: N%` / `Emotion: N%` | 删除，改写为演奏与编曲描述 |
| `energy peaks at N%` | 改为 `full arrangement, layered backing vocals, widest stereo image` |
| `peel to N%` | 改为 `strip back to solo instrument` |

**必查项**：`Energy Start: N%`、`Mood: N%`、`Intensity: N%`、`Emotion: N%`、`energy peaks at N%`、`peel to N%`。

---

## 3. 翻译对照表

| 伪标签 | 音乐语言替代 |
|---|---|
| Energy Start: 20% | starts sparse, single instrument, restrained dynamics |
| Energy 85% | full kit, layered vocals, dense arrangement |
| Energy 100% | doubled rhythmic density, stadium-wide stereo, peak intensity |
| peel to 20% | everything drops out except one instrument |

通用原则：把任何「百分比」翻译为以下四类音乐语言之一——
- **乐器密度**（starts sparse / full kit / layered）
- **编曲规模**（solo instrument / stadium-wide / doubled density）
- **动态变化**（restrained dynamics / builds to climax / drops out）
- **演奏强度**（whispered / peak intensity / restrained）

---

## 4. 允许保留的数字

仅以下为真实参数，**不做替换**：

- **BPM**
- **Key**（调性）
- **拍号**（如 4/4）
- **Duration**（秒）
- **Range**（音域）
- **小节数**

凡不属于上列的数字（尤其是跟在 Energy/Mood/Intensity/Emotion 后面的百分比），一律按 §2/§3 处理。

---

## 5. 处理流程

1. 取 producer 产出的双槽原始文本（S4 未落盘版）。
2. 逐行扫描 §2 的必查模式。
3. 命中即用 §3 的对照改写为音乐语言。
4. 复核：确认无任何 `:` 后跟 `%` 的 AI 自创标签残留。
5. 替换后文本写入 `Prompt.Style` 与 `Prompt.Structure`。

> ✅ 已根治：`lyria-music-producer` 已升 v1.3 直接改源文件（新增音乐导演语言转换层 Step 5.5 + 硬规则 R26/R27），不再输出伪标签。本层保留为防御性安全网，正常情况下为 no-op 透传。

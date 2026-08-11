# 英文歌词质检资产（lyrics-check-en）

> 适用阶段：S2 质检（bmz 干净子代理的知识库补件）
> 用途：闭合 `DESIGN.md` §7.2 的缺口——bmz 原生 `lyrics-check.md` 的好/坏歌词样本与 AI 高频风险词均为**中文资产**。本文件提供**英文**对等资产，让 S2 在 `lang=en` 时也能跑完整的「避雷清单 + 好歌词机制 + 原创性判断」。

---

## 1. 英文 AI 高频风险词 / 陈词滥调（避雷清单 · 对应 bad-xxx）

| 代号 | 表现 | 修改动作 |
|---|---|---|
| `bad-generic-metaphor` | 过度使用 stars / light / fire / scars / shadows / broken / heal 等空泛隐喻，且不落地 | 换具体物象（a chipped mug, a bus that left at 6:02） |
| `bad-forced-rhyme` | moon/June、heart/start、fire/desire 等教科书押韵，语义为空 | 改内韵 / 半韵，或让韵脚承载信息而非装饰 |
| `bad-abstract-vague` | 通篇 I feel / it's real / in my dreams，无感官锚点 | 补一个可触摸的细节 |
| `bad-anaphora-loop` | 反复 "I feel… / I know… / I see…" 句首排比，像清单 | 合并为 1 句，其余换主语或动作 |
| `bad-tell-dont-show` | 直说 "I was so sad" / "we were free"，不外化 | 用行为/场景外化情绪 |
| `bad-cliche-opening` | 以 "In a world…" / "Once upon a time…" 开场 | 从中段或具体瞬间切入 |
| `bad-emo-template` | 拷贝 emo 模板（cut / bleed / fake / hate myself） | 找只属于这首歌的痛法 |
| `bad-brand-drop` | 硬塞品牌/地名当酷（提及具体商标有版权风险） | 改通称或自造指代 |
| `bad-emotional-adverb` | 堆 deeply / truly / really / so much，靠副词撑情绪 | 删副词，让名词与动词承重 |
| `bad-meaningless-repeat` | 副歌只把一句词重复 N 遍凑时长 | 每遍加变量（新细节/新视角） |
| `bad-rhythm-mismatch` | 英文音节数与旋律重音错位，唱出来拗口 | 按自然重音重写，长词避强拍 |
| `bad-over-philosophical` | 末段突然拔高到 "we are the universe" 式说教 | 落回具体人事，不用大词收尾 |

---

## 2. 英文好歌词机制（对应 good-xxx）

| 代号 | 机制 | 范例要点 |
|---|---|---|
| `good-concrete-image` | 用可感知的物象替代形容词 | "the kettle's third whistle" 而非 "I was lonely" |
| `good-specific-detail` | 唯一、不可替换的细节 | 时间/号码/品牌通称/身体小动作 |
| `good-show-dont-tell` | 情绪由行为外化 | 不写 sad，写 "he left the porch light on for no one" |
| `good-earned-turn` | 副歌/桥段有"反转或递进"，非同义反复 | 从自我缩到他人，或从疑问到行动的微小位移 |
| `good-sensory-specific` | 调动单一清晰感官 | 触感/气味/温度优先于抽象"氛围" |
| `good-natural-speech` | 歌词节奏贴合口语重音，像人说的话 | 避免诗化到不可唱 |
| `good-single-metaphor` | 全曲只押一个核心隐喻，前后呼应 | 不堆多个互不相干的意象 |
| `good-structural-payoff` | 前文伏笔在尾段回收（词/音/物象） | 开头掉的某物，结尾被捡起 |
| `good-ambiguity-resolved` | 留白但可被反复听出第二层 | 不是说谜语，是给回味 |

---

## 3. 英文原创性判断（与中文同逻辑）

- **确认现存歌词**：若旋律/词核与已知发行曲高度重合（含 AI 训练语料常见套句），判 `halt` 并给替换方向。
- **弱判断**：仅风格/主题常见（失恋、夜晚、公路）不判抄袭，只标 `generic` 风险，按 §1 避雷清单降陈词。
- **safe**：具体细节充足、隐喻唯一，判 `pass`。

---

## 4. 与 bmz 中文资产的接法

- S2 子代理同时加载 `bmz/knowledge/lyrics-check.md`（中文资产）+ 本文件（英文资产）。
- `lang=zh` → 主要用中文资产，本文件作英文借词参考。
- `lang=en` → 主要用英文资产，中文资产作反向避雷参考（如中文"鸡汤落点"对应英文 `bad-over-philosophical`）。
- 双评分（传播/歌词）与证据等级（S/A/B）口径与 bmz 完全一致，不因语言改变阈值。

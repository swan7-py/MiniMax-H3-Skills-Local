> 本文件是官方 skill `h3-prompt-writing-cn` 的正文归档，随 `h3-video-prompt-cn` 一并分发。
> 官方**规范原文**见同目录 `base-en.txt`（基础模式）/ `ref-en.txt`（全参考模式）；本文件只是它们的使用说明。
> **写提示词一律以 `../SKILL.md` 的实战判据为准**；本文件用于核对官方口径、或需要纯净结构时查阅。

---

# H3 提示词撰写（官方版中文对照）

> **本文定位＝官方 `h3-prompt-writing` 的中文对照版**：章节与官方英文版一一对应，只做语言转换，不承载实测扩展。
> **完整版是 `h3-video-prompt-cn`**（官方全部章节 ＋ 全部实测定论）——需要实测判据、编辑/替换纪律、运镜库指引时一律读那一份，本文不作权威源。

## 工作流 (Workflow)

1. 识别输入模式：T2VA、I2VA、FL2VA、L2VA，还是全参考 Ref2VA。
2. 对于基础文本 / 关键帧模式，阅读 `references/base-en.txt` 并遵循其最终提示词结构。
3. 对于全参考模式，阅读 `references/ref-en.txt` 并遵循其六段改写格式。
4. 保留所选指南中的准确字段名、段落顺序、标签与计时标注。

## 基础模式 (Base Modes)

- T2VA：从文本构建完整的视听时间线。
- I2VA：从首帧开始，并由此向前发展。
- FL2VA：描述首帧与末帧之间的连续路径。
- L2VA：推断一个合理的开头，并收束到给定的末帧。

按 `references/base-en.txt` 所示顺序使用 `integrated_multimodal_description`、`overall_soundscape`、`non_diegetic_music`。

## 全参考模式 (Full-Reference Mode)

Ref2VA 改写依次使用 `subject_definitions`、`summary`、`retention_analysis`、`detailed_description`、`overall_soundscape`、`non_diegetic_music`。参考标签在各段之间保持一致。

阅读 `references/ref-en.txt` 了解标签规则、保留分析与完整示例。

## 音频时间线对齐（带对白音频的 Ref2VA）

当输入包含对白 / 人声音频（`<Audio N>`）时，**先转写音频，再设定分镜时间戳**——这是 Ref2VA 带音频时最常见的失败点：说话人错位（角色 A 在画面中说话，而音频已切到角色 B）。

- **音频优先（Audio-first）**：运行 Whisper（`faster-whisper`，模型 `small`）获取每条语料的起止时间戳与说话人 ID（可区分时用 S1/S2）。将同一说话人连续的语料合并为"话轮"；话轮之间的间隙即自然的切分点。将每个 `[Shot N] At MM:SS.mmm` 锚定到这些话轮边界——不要凭空猜测。注意：Whisper 可能听错数字 / 人名（如"心之星"→"星之星"）；时间戳可靠，内容请用用户原始文本。
- **粗对齐，不要紧对齐**：分镜切点应大致匹配音频中说话人切换的边界（±0.5–1s 即可），但**不要**逐帧精确绑定。在每句中间的停顿处切、或删掉所有时间戳让模型自由发挥，都会让节奏变差。
- **硬规则——禁止说话人错位**：绝不让"角色 A 在画面中说话的分镜"与"音频中角色 B 实际在说话的片段"重叠。核对每个 `[Shot N]` 的时间区间与在该区间内实际发声的说话人（S1/S2）一致。

## 输出规则 (Output Rules)

- **对白 / 歌词必须使用 `<d>[Language] ... </d>` 标签（铁律）**：将说话人的身份短语、ID、动作与语气放在 `<d>` 之外；`<d>` 内部只放语言标签与逐字的原始口语内容——保留每一个原词与标点，**不要翻译、改写或追加英文翻译**。示例：`<d>[Chinese] 今天的星星，好像比昨天暗了一点点。</d>`、`<d>[English] I get off at the next station.</d>`。脚本块与每个分镜描述中均须遵守。
- **音频复用模式（fully_copy）——绝不再合成**：当 `<Audio N>` 作为最终音轨复用时，该音频是唯一的语音来源——只描述谁说话、何时说话、且口型跟随音频。不要添加语气 / 韵律 / 停顿 / 情绪指令（如"声音渐弱""学者般语调"），这些会触发再合成导致语音糊掉。
- 改写段落用中文书写；对白、歌词与可见场景文字保留原始语言（歌词保留中文原文，外文对白 / 歌词保留其原文）。
- 按构图、主体、环境、动作、镜头、声音，以及参考内容出现的精确时点来描述每个分镜。
- 避免剧情摘要、未解析的参考标签，以及与请求时长不符的计时。

## 特殊标记 (Special Tokens · MiniMax Extra Tokens)

已发布的 MiniMax-H3 分词器（基于 Qwen）保留了这些固定的特殊标记。其 ID 由发布的分词器冻结——不要重新编号、改写或拼写错误。包裹对白、歌词、字幕或标记截断时逐字使用。

| 标记 | ID | 用途 |
|-------|-----|------|
| `<d>` / `</d>` | 151669 / 151670 | 对白定界符。将对白包为 `<d>[Language] ... </d>`（铁律——见输出规则）。 |
| `<|lyrics_start|>` / `<|lyrics_end|>` | 151672 / 151673 | 歌词定界符。将演唱的歌词（人声演唱段）按与对白相同方式包裹。 |
| `<|caption_start|>` / `<|caption_end|>` | 151674 / 151675 | 字幕定界符。包裹屏幕上的字幕 / 排版文字 / 叠加油印文字。 |
| `<|cutoff|>` | 151671 | 截断标记。标记生成序列中的硬分段边界 / 段尾。 |

规则：
- 在对白 / 歌词 / 字幕标记内部，只放逐字的原始内容（对 `<d>` 与歌词加 `[Language]` 标签）。不要翻译、改写或追加翻译。
- 说话人身份、动作与语气放在标记之外。
- 这些标记被分词器当作字面标记消费；拼写错误（如 `<lyrics_start>` 少了 `|`）会破坏解析。

## 提升效果的技巧 (Tips for Better Results)

- 始终让描述总时长匹配请求视频长度（4–15 秒）。
- 保持参考标签一致（如 `<Picture 1>`、`<Video 1>`、`<Audio 1>`）贯穿每段。
- 偏好具体的视觉与音频细节，而非"cinematic"或"beautiful"这类抽象词。
- 使用关键帧（I2VA / FL2VA / L2VA）时，清楚说明首帧和 / 或末帧如何与时间线连接。

---

## 扩展参考（中文）

完整手册在本文件同级的 `references/` 下，与官方英文版 `Extended references` 一一对应：

| 文件 | 覆盖内容 |
|---|---|
| `camera-and-lora.md` | camera DSL、按运动类型选 LoRA |
| `reference-image-roles.md` | 一张图该写成 `<Picture N>` 还是内联进 `<Subject N>` |
| `video-reference-modes.md` | 动作迁移、视频续写、角色替换 |
| `editing-and-replacement.md` | 替换 / 移除类编辑纪律 |
| `audio-reference.md` | `<Audio N>` 关系、克隆与参考之别 |
| `negation-guard.md` | 裸否定 vs 替换式否定、白名单 |
| `multi-target-and-hit.md` | 多目标取景与可读性 |
| `card-derived-animation.md` | 把静帧技能卡演成一个动作节拍 |
| `motion-transfer.md` | 从源视频迁移动作 |
| `writing-order.md` | 遮挡与揭示的书写顺序 |
| `creative-dna-library.md` | 可复用的创意机制库 |
| `base-en.txt` / `ref-en.txt` | 官方 T2VA/I2VA/FL2VA/L2VA 与 Ref2VA 规范 |

> 需要**实测定论**（替换四判据、特效结构层、运镜精简纪律、turbo 权重等）时，读完整版 `h3-video-prompt-cn`，那一份才带这些内容。

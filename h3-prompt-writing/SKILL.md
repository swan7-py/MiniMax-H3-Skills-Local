---
name: h3-prompt-writing
description: Write MiniMax H3 video generation prompts for T2VA, I2VA, FL2VA, L2VA, and Ref2VA. Use when rewriting multimodal requests into H3 prompt structures, composing integrated_multimodal_description, overall_soundscape, and non_diegetic_music, aligning keyframes, or defining reference labels for images, videos, and audio.
compatibility: Portable to any agent that can read local files — no external API calls, MiniMax Hub tools, or proprietary runtime required. The agents/openai.yaml file only adds optional ChatGPT/Codex UI metadata; it does not restrict the skill to OpenAI agents.
---

# H3 Prompt Writing

## Workflow

1. Identify the input mode: T2VA, I2VA, FL2VA, L2VA, or full-reference Ref2VA.
2. For base text/keyframe modes, read `references/base-en.txt` and follow its final prompt structure.
3. For full-reference mode, read `references/ref-en.txt` and follow its six-section rewrite format.
4. Preserve the exact field names, section order, labels, and timing notation from the selected guide.

## Base Modes

- T2VA: build the full audiovisual timeline from text.
- I2VA: start from the first frame and develop forward from it.
- FL2VA: describe the continuous path between the first and last frames.
- L2VA: infer a plausible opening and converge to the supplied last frame.

Use `integrated_multimodal_description`, `overall_soundscape`, and `non_diegetic_music` in the order shown in `references/base-en.txt`.

## Full-Reference Mode

Ref2VA rewrites use `subject_definitions`, `summary`, `retention_analysis`, `detailed_description`, `overall_soundscape`, and `non_diegetic_music` in that order. Reference labels stay consistent across all sections.

Read `references/ref-en.txt` for label rules, retention analysis, and complete examples.

## Audio Timeline Alignment (Ref2VA with dialogue audio)

When the input includes dialogue/voice audio (`<Audio N>`), **transcribe the audio first, then set shot timestamps** — this is the most common failure point in Ref2VA-with-audio: speaker mismatch (character A is on screen speaking while the audio has already switched to character B).

- **Audio-first:** Run Whisper (`faster-whisper`, model `small`) to get per-utterance start/end timestamps and speaker IDs (S1/S2 when distinguishable). Merge consecutive utterances by the same speaker into "turns"; the gaps between turns are the natural cut points. Anchor each `[Shot N] At MM:SS.mmm` to these turn boundaries — do not guess blindly. Note: Whisper may mishear numbers/names (e.g. "心之星" → "星之星"); timestamps are reliable, use the user's original text for content.
- **Rough alignment, not tight:** Shot cut points should roughly match speaker-switch boundaries in the audio (±0.5–1s is fine), but do NOT bind frame-precisely. Cutting on every intra-sentence pause, or removing all timestamps and letting the model free-run, both make rhythm worse.
- **Hard rule — no speaker mismatch:** Never let a shot where character A is on screen speaking overlap an audio segment where character B is actually talking. Verify each `[Shot N]` time range matches the speaker (S1/S2) that is actually vocalizing in that range.

## Output Rules

- **Dialogue/lyrics MUST use the `<d>[Language] ... </d>` tag (iron rule).** Place the speaker's identifying phrase, ID, action, and delivery OUTSIDE `<d>`; inside `<d>` include only the language tag and the verbatim original spoken content — preserve every original word and punctuation mark, do NOT translate, rewrite, or append an English translation. Examples: `<d>[Chinese] 今天的星星，好像比昨天暗了一点点。</d>`, `<d>[English] I get off at the next station.</d>`. This is mandatory for both the script block and every shot description.
- **Audio-reuse mode (fully_copy): never re-synthesize.** When `<Audio N>` is reused as the final track, the audio is the sole voice source — describe only WHO speaks, WHEN, and that lip-sync follows the audio. Do NOT add tone / prosody / pause / emotion instructions (e.g. "voice trails off", "scholarly tone"), which trigger re-synthesis and cause garbled speech.
- Write rewrite sections in English; preserve dialogue, lyrics, and visible scene text in their original language.
- Describe each shot by composition, subjects, environment, actions, camera, sound, and the exact point where referenced content appears.
- Avoid plot summaries, unresolved reference labels, and timing that does not match the requested duration.

## 分镜 / 批次 / 参考图 约定（Shot · Batch · Reference）

本 skill 被 8 个本地版设计 skill 调用生成视频提示词时，统一遵循以下定义与硬规则（设计 skill 无需各自重定义，直接引用本节即可）。

### 概念
- **分镜（Shot）**：最小叙事 / 视觉单元，等价于本 skill 提示词里的 `[Shot N]`。每个分镜自带时长，由分镜设计任务决定如何切分。
- **批次（Batch）**：一次 H3 生成单元 = 一组符合本 skill 结构（T2VA / I2VA / Ref2VA / FL2VA / L2VA）的提示词，**可包含多个分镜（≥1）**。一个批次即一次在 H3 节点提交生成的单位；**一个批次 = 一个输出文件**。

### 硬规则
1. **批次时长上限由设备决定，且应在分镜设计前确定**：在每个设计 skill 的「简报 / 起始门」先问用户「本次一次生成几秒？」可选 5s / 10s / 15s（或用户指定值），记为**批次时长上限**并贯穿到分镜设计步骤。分镜设计任务据此把分镜设计在批次内，而非事后强切。
2. **分镜时长一般 ≤ 批次时长（设计期遵守，本步不重切）**：分镜是创作单元，由分镜设计任务按叙事 / 视听决定，设计期就已遵守批次时长上限（一般单镜 ≤ 上限）。本步（视频提示词）**只负责打包**：把设计好的分镜按时长累加归入批次（同批次各镜时长之和 ≤ 批次时长上限），**不重新切分已设计的分镜**。仅当某个分镜本身在设计期就 > 批次时长上限（偶发），才在生成层做「机械分段」：将该分镜切成多个生成子段（如 `Shot 3` → `3a` / `3b`），但**不改动分镜的创意内容与音频时序**——该分镜的音频跨子段连续，仅生成层拼接。
3. **参考图只用占位符，禁止写路径 / 目录**：最终提示词中，参考资产一律用 `<Picture N>` / `<Video N>` / `<Audio N>` 占位符指代（定义见 Full-Reference Mode）。**严禁在提示词里写入任何图像 / 视频 / 音频文件的绝对或相对路径、文件夹目录**。占位符 → 实际文件的映射由用户在 H3 节点侧按端口完成。
4. **一个批次一个文件（不合并）**：不要把所有分镜塞进一份文档。每个批次输出为【一份独立文件】（建议命名 `batch_01.md`、`batch_02.md`…），文件内含该批次全部分镜的 H3 提示词 + 端口 / 顺序 / 时长 / 分辨率设置建议。这样便于逐批次修改与重跑。
5. **motion-context 在连续批次的衔接边界问（H3 跨段运动连续机制）**：当一段连续镜头 / 序列跨多个批次生成时（如某分镜因超上限被机械分段为 3a / 3b，或相邻批次要拼成同一连续场景），在**批次与批次的衔接处**先问用户「是否使用 motion-context？」——它用于保持跨段运动 / 音频连续，但带来副作用：① 衔接处中间 **22 帧会被重复**（22 ÷ 24fps ≈ **0.917 秒**）；② 提示词时间标注上，**第一段（首批次）保持不变，第二段及之后的各段需整体后延 0.917 秒**（其 `[Shot N] At MM:SS.mmm` 时间戳在原始衔接点基础上 +0.917s）。若不使用 motion-context，则按标准衔接（无 22 帧重复、时间标注无需后延）。注意：motion-context 是「批次衔接」闸门，不是「切分单镜」闸门——单镜的设计切分不由它触发。

> 调用方（设计 skill）应在「简报 / 起始门」先问批次时长上限并贯穿分镜设计；「视频提示词」步骤落实：把设计好的分镜**按时长累加打包为批次**（同批之和 ≤ 上限，一个批次一份 `batch_XX.md`），**不重切分镜**；仅当分镜本身超上限时在生成层做机械分段（音频跨段连续）；**motion-context 在连续批次的衔接边界问**（启用则第 2 段起 22 帧重复 + 时间标注后延 0.917s、第 1 段不变）；参考图使用 `<Picture N>` 占位符而非文件路径。无论上游分镜怎么设计，本步保证：每个批次 ≤ 批次上限；分镜创意与音频时序不被强切破坏。

## 批次自含 —— 禁止跨批引用（2026-10-01 用户反复纠正 · 铁律）

**H3 每个批次都是独立生成**：模型看不到上一批的成片，也不知道存在"上一段"。任何指向前一批的措辞都是无效文本，且会让本批丢掉关键状态。

**禁止写法**：`承接上一段` / `延续上一段` / `接着上一镜` / `如上` / `同上` / `双翼自上一段展开后持续保持展开`；`overall_soundscape` / `non_diegetic_music` 里同样禁（`承接上一段的音色 / 电子脉冲`）。

**正确做法——把上一批的「结果状态」改写成本批能独立表述的「开场状态」**：

| ❌ 跨批引用 | ✅ 自含写法 |
|---|---|
| 承接上一段的高速飞行，他仍在疾飞 | 他在高空贴着云海高速掠行，长发与长氅被疾风向后扯成直线 |
| 双翼自上一段展开后持续保持展开 | 双翼在他背后展开着，随他的甩臂与抬臂摆动 |
| 承接上一段的疾速电子脉冲推进 | 开场即起的疾速电子脉冲持续推进 |

判据：**把"上一段"删掉后，这句话还能不能独立成立？不能 → 重写成本批自己交代的画面。**

- 例外：`retention_analysis` 里的 `(出现于 [Shot N])` 是本批**内部**镜号索引，不属跨批引用。
- 批次之间的实际衔接属**分镜设计**，写在批次文件的块外说明里，不进提示词块。
- 自查：写完 grep `承接` / `上一段` / `上一镜` / `如上` / `延续`，块内应为 0。
- **反向同样禁（2026-10-01 二次踩坑）**：`下一段` / `下一镜` / `下批` / `作下一段首帧` 也是跨批引用。末帧图只写「视频以 `<Picture N>` 为末帧收束」；它要作下一段首帧这件事写在块外衔接说明里。

## 道具交接 / 朝向 / 破坏形状（2026-10-02 实跑修正）

- **错握**（手握住剑身而非剑柄）：加**镜头跟随句**"镜头随道具后退半步、把整件道具完整收进画面"，并给**部件次序**"自下而上：手 → 短握柄 → 护手 → 剑身 → 剑尖"；道具定义里就写明握持部位。
- **朝向反了**（写"剑尖朝上"仍朝下）：朝向一律用**部件次序 + 最高/最低点**表达（"剑柄最低 → 剑身竖直向上 → 剑尖是最高的一点"），不要只写"朝上/朝下"。
- **破坏形状对不上攻击形状**（X 形剑气却把山劈成两半）：**破坏结果 = 攻击形状的镜像**——斩几道、什么交叉角度，裂口就是几道、什么交叉角度。
- **抽象施法动作建立不了因果**（写"手臂轻轻一送 → 空气被撕开"，裂口根本没出现）：必须给**起势 → 轨迹 → 中间态（关键：先出现一道细光线/细痕）→ 结果（沿中间态张开）** 四步，并在后一镜回指"这道裂口就是方才那道细线张开后的样子"。

## 否定式描述禁用令（No Negative / Absence Descriptions）

H3 has **no negative-prompt field**, and it does not understand absence. Naming what something is NOT makes that named thing MORE likely to appear: the model latches onto the noun and renders it. Verified on a real e-commerce garment test (same seed, same shots — only the negative clauses changed):

| Written (v1) | Result | Fixed (v2) |
|---|---|---|
| `a plain flat matte-grey metal slider with no ring, no loop and no circular pull` | a large metal ring puller appeared on the hood and hem | deleted the clause → `a flat matte-grey metal slider`; ring gone |
| `the image is never globally desaturated` / `never reads as grey, sage-green` / `the fabric is never pushed toward grey` | grade kept drifting grey-green | deleted; kept only the positive name `bright pale ice-blue`; colour recovered |
| `no percussion, no dramatic swell` (music) | percussion kept showing up | deleted |
| Shot 3 (badge shot) also described the front zipper | the model spent the shot on the zipper and produced a hand-on-placket / zipper-pulling action | deleted the zipper sentence; Shot 3 became a clean badge shot |

硬规则：

1. **只写「有什么」，不写「没有什么」—— 被禁的是「裸否定」**（见下面的两级判定）。No **bare** `no X` / `without X` / `never X` / `X-free` / `no colour cast` / `no percussion`-style phrasing anywhere in `subject_definitions`, `retention_analysis`, `detailed_description`, `overall_soundscape`, or `non_diegetic_music`.
2. **要把某个元素去掉，就删掉提到它的整句**，而不是补一句否定。The zipper simply has no ring → write only `a flat matte-grey metal slider`; do NOT append `with no ring, no loop`.
3. **要表达「空/无」的状态，改成正向状态描写**：`no cars on the street` → `an empty street`；`no text on the wall` → `a bare painted wall`。
4. **要纠正颜色/风格，只写正向目标值**（`bright pale ice-blue`, `a clean neutral white balance`），不要用「不要偏灰、不要偏绿」纠偏——点名灰绿只会强化灰绿。
5. **一个分镜只描述这一个分镜要展示的那一样东西。** 不要把别的卖点捎带写进来：胸标镜头里顺带提门襟拉链，模型就会把注意力分给拉链，甚至生出「手摸门襟 / 拉拉链」的动作（实测发生过）。
6. 写完必须自查：全局搜 `no ` / `never` / `without` / `-free` 及中文 `不` / `没有` / `无`，命中即按下面**两级判定**复核，不要一律删。

### 两级判定：裸否定（禁） vs 替换式否定（安全）— 2026-09-20 实测修订

**「裸否定」= 点名一个不想要的东西，却不给替代物。** 模型抓到名词就画出来 → **禁**。
**「替换式否定」= 「弃用旧属性 + 显式指定新属性」。** 结构固定为：先点名要丢掉的旧属性，**紧接着用「改为 / 换成 / 而是」给出明确新属性**。此时模型收到的是一条**完整的替换指令**，落点是新属性 → **安全，实战有效**。

| 写法 | 判定 | 说明 |
|---|---|---|
| »`<图片 1>`中粉色睡衣**不保留**，角色改为穿着`<Subject 2>`的牛仔外套« | ✅ 安全 | 点名旧属性（睡衣）后**立刻给了新属性**（牛仔外套） |
| »服装与鞋子**不沿用**`<Picture 1>`，改为穿着由`<Picture 2>…`共同定义的汉服造型« | ✅ 安全 | 同上：弃用 + 指定 |
| »`with no ring, no loop`« | ❌ 裸否定 | 只点名 ring/loop，无替代物 → 长出大圆环 |
| »不要偏灰、不要偏绿« | ❌ 裸否定 | 只点名灰/绿 → 强化灰绿 |
| »不要出现中文« | ❌ 裸否定 | 应改成正向状态描写，或直接删掉整句 |

**判据一句话：否定后面有没有紧跟「改为 / 换成 / 而是」？有 → 安全；没有 → 改掉或删掉。**

> 注意区分：**正向否定词不算违规**——`matte`、`flat`、`plain`、`clean`、`quiet`、`bare`、`empty` 这类是正向形容词，正常使用；被禁的是「no / never / without + 名词」这种指名否定。

## Special Tokens (MiniMax Extra Tokens)

The released MiniMax-H3 tokenizer (Qwen-based) reserves these fixed special tokens. Their IDs are frozen by the shipped tokenizer — never renumber, paraphrase, or spell them differently. Use them verbatim when wrapping dialogue, lyrics, captions, or marking a cutoff.

| Token | ID | Purpose |
|-------|-----|---------|
| `<d>` / `</d>` | 151669 / 151670 | Dialogue delimiters. Wrap spoken lines as `<d>[Language] ... </d>` (iron rule — see Output Rules). |
| `<|lyrics_start|>` / `<|lyrics_end|>` | 151672 / 151673 | Lyrics delimiters. Wrap sung lyrics (sung vocal segments) the same way as dialogue. |
| `<|caption_start|>` / `<|caption_end|>` | 151674 / 151675 | Caption delimiters. Wrap on-screen caption / typography / subtitle overlay text. |
| `<|cutoff|>` | 151671 | Cutoff marker. Marks a hard segment boundary / end-of-segment in the generated sequence. |

Rules:
- Inside dialogue/lyrics/caption markers, put only the verbatim original content (with the `[Language]` tag for `<d>` and lyrics). Do not translate, rewrite, or append a translation.
- Speaker identity, action, and delivery go OUTSIDE the markers.
- These markers are consumed by the tokenizer as literal tokens; a wrong spelling (e.g. `<lyrics_start>` without the `|`) breaks parsing.

## Tips for Better Results
- Always match the total duration of the description to the requested video length (4–15 seconds).
- Keep reference labels consistent (e.g. `<Picture 1>`, `<Video 1>`, `<Audio 1>`) across every section.
- Prefer concrete visual and audio details over abstract words like "cinematic" or "beautiful".
- When using keyframes (I2VA / FL2VA / L2VA), clearly state how the first and/or last frame connects to the timeline.

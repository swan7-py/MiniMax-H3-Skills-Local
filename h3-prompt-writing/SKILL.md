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

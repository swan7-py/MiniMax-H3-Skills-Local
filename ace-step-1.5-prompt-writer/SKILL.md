---
name: ace-step-1.5-prompt-writer
description: Turn a brief music description and optional tagged lyrics into an ACE-Step 1.5 generation prompt for the local open-source music model. Produces a rich Caption (style/genre/mood/instrument/timbre/vocal), structured Lyrics (with [Intro]/[Verse]/[Chorus] markers or [Instrumental]), and a Metadata block (bpm/keyscale/timesignature/vocal_language/duration/instrumental). Use when the 8 本地版 video skills need a BGM or song prompt, or any time the user wants an ACE-Step 1.5 text prompt. No API calls — pure prompt writing.
---

# ACE-Step 1.5 提示词书写 skill（本地版 · 纯提示词）

把一段简短的音乐描述（+ 可选带结构标记的歌词）改写成 **ACE-Step 1.5**（本地开源音乐基础模型，Gradio UI / REST API / ComfyUI 节点均可消费）的生成提示词。本 skill **只产出文本提示词，不调用任何生成接口**，由用户在本地 ACE-Step 1.5 中自行执行。

## 何时调用
- 8 个视频本地版 skill 在「音频模式门 = 分离生成」时，需要单独产出 BGM / 整曲提示词。
- 用户直接说「写一段 ACE-Step 1.5 的 BGM / 写首歌的提示词」。
- 任何需要 ACE-Step 1.5 `caption` / `lyrics` / 元数据 文本的场景。

## 你需要向用户收集的信息
1. **音乐用途与基调**：BGM（背景、循环友好）还是整曲（有人声）？情绪/氛围（如 科技感、温暖、紧张、欢快）？
2. **风格与乐器**：流派（pop/rock/jazz/electronic/lo-fi/synthwave…）、主要乐器、音色质感。
3. **是否有人声**：无人声→纯音乐（lyrics 写 `[Instrumental]`）；有人声→要语种 + 是否给歌词。
4. **歌词（可选）**：用户给的歌词文本，或你代写（按下方 Lyrics 指南，带结构标记）。
5. **元数据偏好（可选）**：目标时长（秒）、BPM、调性、拍号、语种。不给定则标注「交给 thinking 模式自动推断」。

## Caption 写作指南（最重要输入，≤512 字符）
Caption 决定生成音乐的「整体画像」，支持三种形式：简单风格词、逗号分隔 tags、复杂自然语言——**形式不影响效果，关键是维度齐全**。
**组合越多维度越精准**（单一维度会让模型自由发挥）：

| 维度 | 示例词 |
|------|--------|
| 风格/流派 | pop, rock, jazz, electronic, hip-hop, R&B, folk, classical, lo-fi, synthwave |
| 情绪/氛围 | melancholic, uplifting, energetic, dreamy, dark, nostalgic, euphoric, intimate |
| 乐器 | acoustic guitar, piano, synth pads, 808 drums, strings, brass, electric bass |
| 音色质感 | warm, bright, crisp, muddy, airy, punchy, lush, raw, polished |
| 时代参考 | 80s synth-pop, 90s grunge, 2010s EDM, vintage soul, modern trap |
| 制作风格 | lo-fi, high-fidelity, live recording, studio-polished, bedroom pop |
| 人声特点 | female vocal, male vocal, breathy, powerful, falsetto, raspy, choir |
| 速度/节奏 | slow tempo, mid-tempo, fast-paced, groovy, driving, laid-back |
| 结构提示 | building intro, catchy chorus, dramatic bridge, fade-out ending |

**原则**
1. 具体优于模糊：「sad piano ballad with female breathy vocal」远胜「a sad song」。
2. 组合多个维度（风格+情绪+乐器+音色）锚定方向。
3. 善用参考：「in the style of 80s synthwave」「reminiscent of Bon Iver」快速传达复杂审美。
4. 质感词（warm/crisp/airy/punchy）影响混音与音色倾向，很有用。
5. 描述粒度决定自由度：写少→惊喜多；写细→可控强。
6. **避免冲突词**：如同时「古典弦乐」+「硬核金属」易劣化。解决：① 重复强化想要的元素；② 把冲突改写为时间演变（「开头柔和弦乐，中段噪杂金属摇滚，结尾转 hip-hop」）。

⚠️ **不要把 BPM / 调性 / 拍号写进 Caption**——这些交给下方 Metadata 参数控制。Caption 专注风格/情绪/乐器/音色。

## Lyrics 写作指南（时间脚本，≤4096 字符；纯音乐写 `[Instrumental]`）
Lyrics 控制音乐随时间展开。结构标记可用 `-` 组合（如 `[Chorus - anthemic]`），**不要堆叠太多标记**（避免 `[Chorus - anthemic - stacked harmonies - high energy]`）。

| 类别 | 标记 |
|------|------|
| 基础结构 | `[Intro]` `[Verse]`/`[Verse 1]` `[Pre-Chorus]` `[Chorus]` `[Bridge]` `[Outro]` |
| 动态段落 | `[Build]` `[Drop]` `[Breakdown]` |
| 器乐段落 | `[Instrumental]` `[Guitar Solo]` `[Piano Interlude]` |
| 特殊 | `[Fade Out]` `[Silence]` |

**技巧**
- 每行 **6–10 个音节**效果最好（模型把音节对齐到节拍）。
- 大写 = 更强演唱力度（`WE ARE THE CHAMPIONS!`）。
- 括号 = 背景人声（`We rise together (together)`）。
- 重复元音延音（`Feeeling so aliiive`，谨慎用）。
- 段落间空行分隔。

**避免「AI 味」歌词**：形容词堆砌、押韵混乱、段落边界模糊、没有呼吸感（每行太长）、隐喻混用（一首歌坚持一个核心隐喻）。

⚠️ **保持 Caption 与 Lyrics 一致**：Caption 写「violin solo, classical」则 Lyrics 用 `[Violin Solo - expressive]`，不要冲突成 `[Guitar Solo - electric]`。

## Metadata 指南（按需填写；不填则开 thinking 模式由 LM 自动推断）
| 参数 | 范围 | 说明 |
|------|------|------|
| `bpm` | 30–300 | 速度；慢歌 60–80、中速 90–120、快歌 130–180 |
| `keyscale` | 调性 | 如 `C Major` `Am` `F# Minor` |
| `timesignature` | 拍号 | `4/4`(最常见) `3/4`(华尔兹) `6/8`(摇摆) |
| `vocal_language` | 语种码 | `zh` `en` `ja` `es` `fr`…（LM 通常能据歌词识别） |
| `duration` | 秒 | 10–600；实际略有偏差 |
| `instrumental` | true/false | true 则无论歌词都生成纯音乐 |

## 输出契约（固定三段，用户可直接粘贴进 ACE-Step 1.5）
```
### Caption（≤512字符，专注风格/情绪/乐器/音色，不写BPM/调性）
<组合维度后的描述文本>

### Lyrics（≤4096字符；纯音乐写 [Instrumental]）
[Intro - piano]

[Verse 1]
...

[Chorus - powerful]
...

[Outro - fade out]

### Metadata（不填的项标注「自动推断」）
- bpm: 120
- keyscale: C Major
- timesignature: 4/4
- vocal_language: zh
- duration: 30
- instrumental: false
```

## 示例

**示例 1｜科技感 BGM（纯音乐，循环友好）**
```
### Caption
electronic, mid-tempo, tech-house, warm analog synth pads, punchy kick, airy pluck arpeggio, studio-polished, building intro, subtle tension

### Lyrics
[Instrumental]

### Metadata
- bpm: 100
- keyscale: A Minor
- timesignature: 4/4
- vocal_language: (自动推断)
- duration: 30
- instrumental: true
```

**示例 2｜中文人声宣传曲**
```
### Caption
pop, uplifting, energetic, bright, piano and strings, powerful female vocal, stadium chorus, studio-polished

### Lyrics
[Intro - piano]

[Verse 1]
迎着晨光出发
心跳和鼓点同频
每一步都算数
这是我们自己的路

[Chorus - powerful]
向前跑 不回头
光在掌心燃烧
这一刻 属于我们
WE ARE THE LIGHT!

[Outro - fade out]

### Metadata
- bpm: 120
- keyscale: C Major
- timesignature: 4/4
- vocal_language: zh
- duration: 45
- instrumental: false
```

**示例 3｜参考风格（用参考词快速传达审美）**
```
### Caption
lo-fi, dreamy, nostalgic, warm vinyl crackle, soft electric piano, breathy male vocal, reminiscent of 90s city-pop, laid-back

### Lyrics
[Verse 1]
夜色漫过窗台
老唱片轻轻转
...

[Chorus]
就让它慢慢走
像潮水不退不进

[Outro - fade out]

### Metadata
- bpm: 85
- keyscale: F Major
- timesignature: 4/4
- vocal_language: zh
- duration: 40
- instrumental: false
```

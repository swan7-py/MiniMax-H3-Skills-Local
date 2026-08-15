# H3 本地版 Skills 分享包

一套面向 **本地生成工作流** 的 WorkBuddy skill 集合。核心理念：**只产出提示词，不调用任何云端 / ComfyUI 生图生视频接口**，并且**每一步都用确认门暂停、等你返回图像/成片地址后再继续**——你完全掌控生图、生视频、生音乐的工具与额度（Krea2、Wan I2V、H3 节点、ACE-Step 1.5 本地部署等均可），本包只负责把"过简需求"翻译成可直接粘贴的富结构提示词。

> 适用对象：已自建本地 ComfyUI / H3 / ACE-Step 1.5 环境，想把"一句话创意 → 分镜 → 各镜头提示词"这条链路交给 AI 流水线、但生成执行留在自己手里的用户。

---

## 包含清单（共 11 个 skill）

### 一、本地版生成类（8 个，直接面向各类视频/动画产出）

| Skill 文件夹 | 用途 |
|---|---|
| `3d-animation-short-generator-本地版` | 风格化 3D 动画短片（简报→大纲→角色卡/场景卡→镜头表→分镜→视频提示词） |
| `brand-promo-video-generator-本地版` | 品牌 / 产品 / 网站宣传短片 |
| `co-op-game-intro-generator-本地版` | 双人合作游戏开场动画 |
| `handdrawn-live-video-generator-本地版` | 手绘发光动画 + 实拍融合短视频 |
| `minimalist-product-ad-generator-本地版` | 极简 / Apple 风产品广告 |
| `music-video-subtitle-generator-本地版` | 歌词排版 / 卡点 MV |
| `paper-collage-explainer-generator-本地版` | 纸拼贴科普动画 |
| `papercraft-stop-motion-explainer-本地版` | 纸艺定格科普动画 |

### 二、提示词 skill（3 个，被上面 8 个在对应步骤调用）

| Skill 文件夹 | 用途 |
|---|---|
| `image-prompt-writer-本地版` | 富结构文生图提示词（通用风格锚 + 分图四段式，每图「中文版 + 英文版」双版），解决"提示词过简"痛点 |
| `h3-prompt-writing` | 核心视频提示词 skill（MiniMax H3 的 T2VA/I2VA/FL2VA/L2VA/Ref2VA 结构；纯文本、无需 API），含 `agents/openai.yaml` 与 `references/` |
| `ace-step-1.5-prompt-writer` | ACE-Step 1.5 本地音乐模型提示词（Caption + Lyrics + Metadata 三段） |

> **强依赖**：8 个生成类 skill 在「生图」步会调用 `image-prompt-writer-本地版`、在「视频」步会调用 `h3-prompt-writing`、在「分离模式 BGM」步会调用 `ace-step-1.5-prompt-writer`。**分享/安装时必须 11 个一起装**，缺任何一个都会在某一步断了调用。

---

## 安装方法

把本包内 `本地版生成Skills/` 与 `提示词Skills/` 下的 **11 个 skill 文件夹**整体复制到 WorkBuddy 用户级 skills 目录，重启/刷新即可在 UI 看到：

- **Windows**：`C:\Users\<你的用户名>\.workbuddy\skills\`
- **macOS / Linux**：`~/.workbuddy/skills/`

目录结构（复制后你的 skills 目录里应新增这些文件夹）：

```
.workbuddy/skills/
├── 3d-animation-short-generator-本地版/
├── brand-promo-video-generator-本地版/
├── co-op-game-intro-generator-本地版/
├── handdrawn-live-video-generator-本地版/
├── minimalist-product-ad-generator-本地版/
├── music-video-subtitle-generator-本地版/
├── paper-collage-explainer-generator-本地版/
├── papercraft-stop-motion-explainer-本地版/
├── image-prompt-writer-本地版/
├── h3-prompt-writing/          （含 agents/ 与 references/ 子文件，需一并复制）
└── ace-step-1.5-prompt-writer/
```

> 注意：`h3-prompt-writing` 不是空文件夹——它带有 `agents/openai.yaml` 和 `references/base-en.txt`、`references/ref-en.txt`，复制时务必保留整目录。

---

## 设计理念：本地化改造总则 v2

这 8 个本地版由官方原版 skill 改造而来，统一遵循以下规则：

1. **生图只给提示词，不调用任何生图工具**：每步只输出文生图提示词，你用任意工具（Krea2 / ComfyUI 等）生成图像并把**图像地址返回**即可。不硬编码任何工作流路径。
2. **每一步都确认 + 交付物门**：每个步骤产出后暂停，等你「确认 / 修改」；生图步必须等你**返回图像地址**后才进入视频步（视频步以你的静帧为参考图，I2V from still）。
3. **视频提示词调用 `h3-prompt-writing`**：按 H3 的 I2VA/Ref2VA 结构输出完整视频提示词 + 端口 / 顺序 / 时长 / 分辨率建议，你在本地 H3 节点执行。
4. **音频方案门（见下）**：第一步先问你音频走「统一 H3」还是「分离生成」。
5. **旁白/对白一致性（分离模式必做）**：需把【音色参考文件】放到**当前项目文件夹**，用任意 TTS 的「参考 / 音色克隆」模式保证跨镜头音色统一；未提供则暂停，禁止无参考直接生成。
6. **BGM 分离 → 视频提示词只规划视觉**：选分离模式时，BGM 由 ACE-Step 1.5 单独生成，视频提示词**不写 BGM/旁白/对白出现指令**，音频留到后期合成。
7. **删除自动剪辑 / 拼接**：改为提供"剪辑思路"。
8. **规划 / 分镜 / 排版等文本方法论沿用原 skill**。
9. **删除"表情图"相关提示词**（角色卡默认四视图锁定一致性）。

### 门控硬规则（真停顿）

每个 `⏸ 确认门` / `⏸ 交付物门` 都是**硬性停止点，不是建议**。固定顺序：① 仅输出当前步交付物 → ② **调用 AskUserQuestion** 给三选项「✅ 确认继续 / ✏️ 修改当前步 / ⏸ 暂停或换方向」→ ③ 等待你回复，**绝不预生成后续步骤**。任何门不得跳过 / 合并，也不得用一句"请确认"文字代替。

### 音频模式门

进入流程第一步时必问二选一：

- **A) 统一 H3 音频**：旁白 + 对白 + BGM 全由 H3 原生音频驱动（最省事，无需外部文件）。视频提示词可正常含音频指令。
- **B) 分离生成**：旁白/对白**只给提示词**（不绑定 MOSS-TTS，你用任意 TTS 的参考模式 + 音色参考文件）；BGM 用 `ace-step-1.5-prompt-writer` 生成 ACE-Step 1.5 提示词；视频提示词只规划视觉，音频后期合成。

---

## 典型使用流程（以某个生成类 skill 为例）

```
你给创意/素材
   ↓
[STEP] 解析 + 制作计划          → ⏸ 确认门（你批准）
   ↓
[STEP] 静帧/角色卡/场景卡规格    → ⏸ 确认门
   ↓
[STEP] 生图提示词               → 调用 image-prompt-writer-本地版
       输出「风格锚 + 每图中文版/英文版双版四段式」
                                 → ⏸ 交付物门：你用 Krea2 生图，返回图像地址
   ↓
[STEP] 分镜/组装计划            → ⏸ 确认门
   ↓
[STEP] 视频提示词               → 调用 h3-prompt-writing
       输出完整 H3 提示词 + 端口/时长/分辨率建议
                                 → ⏸ 交付物门（可选）：你在 H3 节点生视频，返回成片
   ↓
[STEP] 音频（依模式门分支）
       A 统一H3：提示词已含音频指令
       B 分离：旁白只给提示词（你用 TTS 参考模式）
                BGM 调用 ace-step-1.5-prompt-writer 出 ACE-Step 1.5 提示词
   ↓
[STEP] 质量评审 → ⏸ 确认门
   ↓
[STEP] 剪辑思路（替代自动剪辑）
```

`image-prompt-writer-本地版` 与 `ace-step-1.5-prompt-writer` 也可**独立调用**——直接让 AI 写文生图提示词或写一段 BGM 提示词即可，不必走完整流水线。

---

## 兼容性 / 免责

- 全部 skill **纯文本产出、零 API 调用**，可移植到任何能读取本地 `.workbuddy/skills/` 的 WorkBuddy 环境（含 OpenAI ChatGPT/Codex 等兼容 UI，`h3-prompt-writing` 的 `agents/openai.yaml` 仅提供可选 UI 元数据，不限制运行环境）。
- 本包**不包含**任何生图/生视频/生音乐的权重、工作流 JSON 或执行脚本；生成执行完全在你本地环境完成。
- Krea2 工作流、`h3-prompt-writing` 等原始能力归属各自原作者 / 官方项目，本分享包仅做"本地版 · 纯提示词 · 逐步确认"的改造封装。

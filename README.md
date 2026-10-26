# H3 本地版 Skills 分享包

一套面向 **本地生成工作流** 的 WorkBuddy skill 集合。核心理念：**只产出提示词，不调用任何云端 / ComfyUI 生图生视频接口**，并且**每一步都用确认门暂停、等你返回图像/成片地址后再继续**——你完全掌控生图、生视频、生音乐的工具与额度（Krea2、Wan I2V、H3 节点、ACE-Step 1.5 本地部署等均可），本包只负责把"过简需求"翻译成可直接粘贴的富结构提示词。

> 适用对象：已自建本地 ComfyUI / H3 / ACE-Step 1.5 环境，想把"一句话创意 → 分镜 → 各镜头提示词"这条链路交给 AI 流水线、但生成执行留在自己手里的用户。

---

## ⭐ 如果你只想装「有用的那几个」（2026-10-25 起推荐）

本仓库现在分两块：**主线 = H3 提示词**，**其余是 2026-09 的本地版流水线（已归档到 `old/`）**。

**推荐只装 2 个，最多 4 个**：

| 装什么 | 为什么 |
|---|---|
| `h3-video-prompt-cn` | **必装**。H3 六段结构 + 实跑判据（动作迁移、替换／编辑、音频参考、否定式闸门、交付模板） |
| `h3-camera-motion-lib` | **强烈建议**。镜头句本体——按运镜族给可直接粘贴的中文句子 + 实测状态 + 推荐 LoRA |
| `h3-prompt-writing-cn` | 可选。**全量中文版**（自带 16 份 references），只想装一个时选它 |
| `h3-prompt-writing` | 可选。英文骨架 + `agents/openai.yaml`，现同样自带 16 份 references |

`old/` 里的 10 个是早期流水线，**2026-09-03 之后没再维护**，且互相强依赖（要装就得 10 个一起装），详见 [`old/README.md`](old/README.md)。

---

## 包含清单

### 一、主线：H3 提示词（4 个）

| Skill 文件夹 | 版本性质 | 用途 |
|---|---|---|
| `h3-video-prompt-cn` | **实践新版**（完整版 / 超集，**主力**） | 官方 H3 规范 ＋ 实跑出来的判据：镜头层纪律、参考图两种角色、否定式两级闸门、动作迁移、替换／编辑、音频参考、对白写法、交付模板 |
| `h3-camera-motion-lib` | **实践产出**（全新） | ⭐ **H3 运镜提示词库**——按运镜族（摇镜变焦 / 推拉横移升降 / 环绕翻滚子弹时间 / 特殊复合 / 视角机位 / 装置风格 / 双人关系）给**可直接粘贴的中文镜头句** ＋ 实测状态（✅🟡❌）＋ 推荐 LoRA ＋ 坑 |
| `h3-prompt-writing-cn` | **全量版 · 中文**（2026-10-26 起） | 官方六段结构 + 专家包全部实战判据，**自带 16 份 references**；改动先改它，再同步回 `h3-video-prompt-cn` |
| `h3-prompt-writing` | **官方原版 · 英文骨架** | 同上，带 `agents/openai.yaml`（可选 UI 元数据），供英文产出或兼容 UI 使用 |

> **日常只需要前两个**：`h3-video-prompt-cn` 管结构与判据，`h3-camera-motion-lib` 管镜头句本体。
> 两个官方原版只在「要和官方口径对照」时才装。

#### 「官方原版」与「实践新版」的区别

| | `h3-prompt-writing` / `h3-prompt-writing-cn`（**官方原版**） | `h3-video-prompt-cn`（**实践新版**） |
|---|---|---|
| 来源 | MiniMax 官方 H3 提示词指南（六段结构、标签与占位符、参考资产规则），本分享包只做"本地版 · 纯提示词 · 逐步确认"的改造封装 | 官方原版 **＋ 我们实跑之后补进去的判据** |
| 体量 | SKILL.md **226 行**（英）/ **355 行**（中），references 各 **16 份** | SKILL.md **383 行**，references **16 份** |
| 多了什么 | — | 镜头层纪律、参考图两种角色、否定式两级闸门、对白 `<d>` 写法、隔空操控动词纪律、跨镜点名、大型附属物尺度、道具交接、多目标打击、交付模板等**踩坑条目** |
| 适用 | 8 个本地版生成 skill 的调用依赖，**必须装** | 你直接写 H3 提示词时用它更细；**可选**，不装也不影响流水线 |

> **一句话**：官方原版告诉你"H3 提示词长什么样"，实践新版告诉你"哪些写法在实跑里会翻车"。
> **两份现已合并为同一套 references（16 份，逐字节一致），并且 `h3-prompt-writing-cn` 于 2026-10-26 升级为「全量中文版」**——
> 它已整合专家包版全部内容，**改动先改 `h3-prompt-writing-cn`，再同步回 `h3-video-prompt-cn`**（权威方向与 2026-10-25 之前相反）。

#### 两个进阶 skill 的分工

> `h3-video-prompt-cn` 管**六段结构与规范**（镜头句该放在哪一段）；
> `h3-camera-motion-lib` 管**镜头句本体**（这句话具体怎么写）。写运镜类提示词时先查后者，再套前者的结构。
> **不依赖 `cinematic-director-camera-dsl`**：镜头句在本库里已可直接取用，camera DSL 降级为可选校验工具；
> 两者结论冲突时以本库的「实测」一栏为准（本库结论来自 A–G 七组 46 镜 · 约 100 条成片的对照测试）。

### 二、已归档：2026-09 的本地版流水线（10 个，在 `old/` 下）

| Skill 文件夹 | 用途 |
|---|---|
| `3d-animation-short-generator-local` | 风格化 3D 动画短片 |
| `brand-promo-video-generator-local` | 品牌 / 产品 / 网站宣传短片 |
| `co-op-game-intro-generator-local` | 双人合作游戏开场动画 |
| `handdrawn-live-video-generator-local` | 手绘发光动画 + 实拍融合 |
| `minimalist-product-ad-generator-local` | 极简 / Apple 风产品广告 |
| `music-video-subtitle-generator-local` | 歌词排版 / 卡点 MV |
| `paper-collage-explainer-generator-local` | 纸拼贴科普动画 |
| `papercraft-stop-motion-explainer-local` | 纸艺定格科普动画 |
| `image-prompt-writer-local` | 富结构文生图提示词（上面 8 个的生图步依赖） |
| `ace-step-1.5-prompt-writer` | ACE-Step 1.5 音乐提示词（分离模式 BGM 依赖） |

> 这 10 个 **2026-09-03 之后未再维护**，已整体移入 `old/`。
> 它们互相强依赖：8 个生成类在生图步调用 `image-prompt-writer-local`、视频步调用 `h3-prompt-writing`、BGM 步调用 `ace-step-1.5-prompt-writer`——**要装必须 10 个一起装**，单独装会断链。详见 [`old/README.md`](old/README.md)。

---

## 安装方法

### 推荐：只装 H3 主线

把下面 2 个文件夹复制到 WorkBuddy 用户级 skills 目录，重启 / 刷新即可在 UI 看到：

- **Windows**：`C:\Users\<你的用户名>\.workbuddy\skills\`
- **macOS / Linux**：`~/.workbuddy/skills/`

```
.workbuddy/skills/
├── h3-video-prompt-cn/     ← 必装（结构 + 判据）
└── h3-camera-motion-lib/   ← 强烈建议（镜头句本体）
```

需要官方口径对照时，再补 `h3-prompt-writing-cn/`、`h3-prompt-writing/`（两者与 `h3-video-prompt-cn` 同源，可共存）。

> `h3-video-prompt-cn` 带 `references/` 子目录（**16 个文件**），复制时**保留整目录**。
> `h3-prompt-writing` / `h3-prompt-writing-cn` 现在也各带一套 **完全相同的 16 份 references**（含中文 `base.txt` / `ref.txt` 与官方英文 `base-en.txt` / `ref-en.txt`），装哪个都能独立使用。

### 若要装已归档的本地版流水线

见 [`old/README.md`](old/README.md)。`old/` 下 10 个必须一起装，且会被装到同一层 skills 目录（即 `old/` 这层只是仓库里的收纳，不要连 `old/` 目录一起复制）。

---

> ⚠️ **下面四节（设计理念 / 门控硬规则 / 音频模式门 / 典型使用流程）描述的是 `old/` 里那条已归档的本地版流水线。**
> 只装 H3 主线的读者可以跳过；要复现那条流水线，请连同 [`old/README.md`](old/README.md) 一起看。

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
[STEP] 生图提示词               → 调用 image-prompt-writer-local
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

`image-prompt-writer-local` 与 `ace-step-1.5-prompt-writer` 也可**独立调用**——直接让 AI 写文生图提示词或写一段 BGM 提示词即可，不必走完整流水线。

---

---

## 更新记录（Changelog）

### 2026-10-26 · 三份 skill 的 references 统一；`<Picture N>` 旧硬规则撤销；新增「构图可执行性」7 条

**结构**：`h3-prompt-writing` / `h3-prompt-writing-cn` / `h3-video-prompt-cn` 三处的 `references/` 收敛为**同一套 16 份文件（逐字节一致）**；`h3-prompt-writing` 首次获得整套 references（此前只有 `ref-en.txt` / `ref.txt`）。`h3-prompt-writing-cn` 由「精简版」升级为**全量中文版**，并接管权威方向：**改动先改它，再同步回 `h3-video-prompt-cn`**。

| 文件 | 新增 / 变更 |
|---|---|
| 三份 `SKILL.md` + `reference-image-roles.md` | **撤销 2026-10-08／09 的硬规则「`summary` 与 `detailed_description` 里 `<Picture` 计数必须为 0」**——它与官方 `ref-en.txt` 冲突（官方只要求"不新起编号"）。新口径：**四段都能用 `<Picture N>`，错的是编号漂移**；旧的"计数为 0"降级为排障备注（写进运镜句导致构图被拽时，该处换 `<Subject N>` 试一次）。机械自检改为「**编号四段一致**」 |
| `ref-en.txt` | 补一句：`summary` 里的 "shot flow" 指**镜头级概括**（一镜到底 / 三镜对切），**焦段、半径、高度、角度、速度、景别等运镜细节只写在 `detailed_description`** |
| 三份 `SKILL.md` | **新增「构图可执行性 —— 把形容词变成模型能执行的东西」7 条**：① 开写前声明运行模式（Ref2VA 六段 vs 基础三字段）；② 两套命名空间（提示词标签从 **1** 数、接线端口 `ref_image_0` 从 **0** 数）；③ 比喻动词按字面执行（`hovering` → 指针真浮在盘面上方），「挨着」必须写物理证据（贴平 / 底面接触 / 影子在正下方相接）；④ 微距必须锁「画面里只有 X 和 Y」；⑤ 含人物镜必须写景别锁（实测 0.549 / 0.404 → 加锁 0.9986 / 0.9950），且**视线词直接决定体态**；⑥ 有读数物体必须写角度关系（四张钟面三张读错）；⑦ 跨镜连续性清单 4 项（道具件数 / 可读文字 / 读数状态 / 唯一例外） |
| `h3-prompt-writing-cn/SKILL.md` | **并入专家包全部实战纪律**：参考图「一图多态」按位置拆 `<Subject>`（防多生一把扇子）、隔空操控动词纪律（`flick` → 会画成手握道具，改 `point`）、手势粒度不可控（双指并拢恒渲染成单指，只能从身体层面加码）、风格统一全量替换 + 残留自查、长镜合并前查显存、5s 单条约承载 2 个动作节拍、参考命名不写「定稿」、`fully_preserved` 不控制硬度、跨镜持续元素每镜点名、大型附属物给显式尺度倍数 |
| `h3-prompt-writing-cn/references/` | 补入 7 份：`audio-reference` / `card-derived-animation` / `creative-dna-library` / `editing-and-replacement` / `motion-transfer` / `multi-target-and-hit` / `video-reference-modes` |
| `camera-and-lora.md` | **§5.4 turbo 权重 ↔ 参考贴合度**：加速 LoRA 是「参考能力」的旋钮，权重越高越贴参考；降权重会表现为段内自编镜头、背景被重画、首帧画出参考图版式。实测 `turbo 4step` **0.8 → 0.9** 消除「段内自编舞台远景 + 乐队」。**不要用降 turbo 去油**。Dutch Angle 定稿 `strength = 2.0`（1 ≈ 20°） |
| `negation-guard.md` | **新增 §六 / 6.1 / 6.2 / 6.3**：清除清单句（`A 和 B 不保留` + 同段正向句接管）不算裸否定；**效力排序 `删除` ＞ `不保留，改为 Y` ＞ `没有 X`**（清过闸门 ≠ 有效）；**载体比句式更重要**——待清除物件必须挂在 `<Video 1>` 的待重建区域，编成 `<Subject N>` 反而要求保留它；`不出现` 属 L1 硬拦 |
| `h3-video-prompt-cn/references/reference-image-roles.md` | 环境段重写为「**描述式 vs 清单式**」（效果上限放大 2.4–3×，**仍不到腰部以上**，正解是改用 I2VA 真实首帧）；F10 事故案例与「防复制双保险」写法**移出**，改为指针指向 `h3-camera-motion-lib` §七/§八 与 `06-rig-and-device.md` F10 |
| `h3-camera-motion-lib/references/00-core-discipline.md` | **§八 同步修订**：原「`<Picture N>` 不进 `summary` 与 `detailed_description`… 计数应为 0」与撤销后的新规则冲突，改为「编号漂移才是错」 |

> 另有 `audio-reference.md` / `output-template.md` / `writing-order.md` 三处仅换行 / BOM 差异，内容无实质变化。

### 2026-10-25 · 仓库重构：旧流水线归档；替换类新增三条判据

**结构**：2026-09 的 10 个本地版 skill 整体移入 `old/`，根目录只留 H3 主线 4 个；README 顶部改为「推荐只装 `h3-video-prompt-cn` 相关」。

**`h3-video-prompt-cn` 内容更新**（长片分段换人 · 6 轮迭代的收敛结论）：

| 文件 | 新增 |
|---|---|
| `references/editing-and-replacement.md` | **§十一 替换效果的三个提示词杠杆**：① `<Subject>` 拆分判据——**片中只要有「把某件东西从主体身上取下 / 移开」的动作，这件东西就必须独立成 `<Subject>`**（写在主体外观串里 ⇒ 脸上一份 + 手里一份）；② 主体指代必须带画面可见特征（「**带着耳麦的歌手**」＞「歌手」）；③ 背景只写「**背景、光源与明暗全部严格沿用 `<Video 1>` 对应时刻**」——写「几处极淡的虚化光点」模型就真画出来（实测顶部条带亮度 33.7→61.7、饱和度 0.042→0.132）。附校验清单 +3 项 |
| `references/negation-guard.md` | **§六 清除清单句不算裸否定**：`X 不保留，改为 Y` 属替换式；`A 和 B 不保留` 属「清除源片元素」的清单句，不属 L1。两个限制：只能清能被正向描述接管的对象；静态贴身物件（耳麦）写「不保留」多数不生效 |
| `references/camera-and-lora.md` | **§5.4 turbo 权重 ↔ 参考贴合度**：加速 LoRA 是「参考能力」的旋钮，降权重会削弱贴合，表现为段内自编镜头、背景被重画、首帧画出参考图版式。实测 `turbo 4step` **0.8 → 0.9** 消除「段内自编舞台远景 + 乐队镜头」。处置顺序：**先查权重 → 再查参考片分段 → 最后改提示词**；不要用降 turbo 去「去油」 |

同源精简版 `h3-prompt-writing-cn/SKILL.md` 也补了浓缩版「替换类三条杠杆」。



### 2026-10-15 · `h3-video-prompt-cn` 补入「动作迁移」与「替换／编辑」两套提示词规程

新增 `h3-video-prompt-cn/references/motion-transfer.md`（八节）与 `h3-video-prompt-cn/references/editing-and-replacement.md`（十节），
并在 `h3-video-prompt-cn/SKILL.md` 的全参考模式段加入**判分岔的路由句**（成片里源视频还在不在）＋ references 索引表登记两行。

**来源**：
- `motion-transfer.md` —— 2026-10-14 赛博朋克滑板动作迁移实测：参考视频 `舞蹈视频.mp4` ＋ 四视图角色卡，
  **5 版提示词 × 2 条管线（SelfLift dasiwa / FreeVideo VDN）**、同 seed / 同 9:16 / 同 5+3，用逐帧 Farnebäck 光流做定量判据。
- `editing-and-replacement.md` —— H3 替换／编辑类（换脸换人换服装换物体换背景换文字换材质换光照、移除、新增）实跑总结。

| 文件 | 核心内容 |
|---|---|
| `motion-transfer.md` | **零动作白描**（把动作写成可表演的台词，模型就会照文字演，而不参考视频）＋**运动几何七维逐镜归属**（位置/朝向/轮廓/位移与速度/姿态/占位体积/接触关系）；`<Video N>` 定义升格为"全片运动与机位的唯一来源"；收口句；**⭐ 字数配额（定量）**：2622→1701→1416 字 ⇒ 运动能量 0.726→1.524→1.650、r 0.143→0.437→0.495 **单调改善**，但**精简边界是不能砍掉题材识别特征**（砍掉场景外观后赛博朋克公园退化成普通公园）⇒ **取向 1500–1700 字**；光流四判据（能量比 0.7–1.3× / r>0.4 / 方向一致度>0.15 / 峰值帧）＋残余的"时间轴被压缩"问题；并注明**清晰度不归提示词解决**（σ 低噪区采样步、一采分辨率、turbo LoRA） |
| `editing-and-replacement.md` | **三归属维度**：外观←参考素材 `<Subject>`／运动几何←源视频 `<Video>`／光学←源视频环境（漏运动几何⇒目标自己乱动、脱手、穿模）；**遮罩形态检测**（半透明 R≈80 为默认路径，不透明为回退）；**七个不可省模块** REBUILD / LIGHT / SYNC / PERFORMANCE / OCCLUSION / FIT / SEAM / NO_ADDED_EVENT；**目标类型→槽位速查表**（10 类逐类给外观槽位、几何锚点、必列不迁移项）；**可见行为四组**必须带同节奏约束（on the same beats）；尺寸冲突 **R1 保持占位（默认）/ R2 忠于参考 / R3 折中**；任务类型标记与 `summary` 固定首句 `The target video is an edited version of <Video 1>.` |

**同日同步**：把本机在用、但仓库版本落后的 `h3-prompt-writing` / `h3-prompt-writing-cn` 覆盖同步回仓库
（原仓库版 84/89 行，本机版 149/215 行，且本机版多 5 个 references：camera-and-lora / negation-guard / output-template / reference-image-roles / writing-order）。

---

### 2026-10-14 · `h3-video-prompt-cn` 补入「音频参考」判据

新增 `h3-video-prompt-cn/references/audio-reference.md`（七节），并在 `h3-video-prompt-cn/SKILL.md` 的 references 索引表与「音频时间线对齐」节登记。

**来源**：H3 音频参考能力测试 —— 30 条成片，SelfLift dasiwa × FreeVideo **双管线、同提示词同 seed** 对照。

| 节 | 内容要点 |
|---|---|
| **一** | **⭐ 角色形象先验 > `<Audio N>` 音色参考**（`reference` 的根本机制）——模型在「角色形象」与「参考音色」冲突时按角色形象加权折中（同性别可完整采纳 / 跨性别被往表观性别方向拉 / 双人同框完全倒向角色形象）；**并含关键判定陷阱：F0 落在正确性别区间 ≠ 音色被复刻**（可能是「按角色形象脑补的该性别通用声音」），**音色迁移只能靠试听判定、F0 只能证伪** |
| 〇 | **四种关系标记的实测基线**：`fully_copy` 音乐 r≥0.99 且零位移、语音带 **+0.45s 系统性位移**；`partially_copy` **区间声明不被遵循**；`reference` 迁移能力取决于**有无性别对手**；`weak_reference` 不建议投入。附复刻精度的脚本化判据（0.05s 包络 ±2s 互相关） |
| 一 | **`Sx` 编号须与接线顺序一致** —— 让 `Subject N` = `S N` = `Audio N` **三线同号**，附交付前机械自检表 |
| 二 | **台词约束不可省** —— 把 `<Audio N>` 定义简化成纯绑定句时，官方规范 §5.4「只参考音色时不把原音频台词带入成片」这句**必须保留**，否则单人单句会跑偏（实测：包络相关升至 0.66、开口晚 1.5s） |
| 三 | **双人 / 多说话人锁定** —— 性别倒错声明、过肩镜头、每镜收尾写「另一人嘴唇保持闭合 + 本镜台词由 X 独自说完」 |
| 四 | **正向单人约束（五处齐写）** —— 抑制四视图参考卡的分格布局泄露（成片出现多个人物、或把参考卡的面部特写格渲染进画面） |
| 五·六 | 音频类否定式的**白名单**（`<d>` 内台词整段豁免）与闸门脚本接口 |

**同日补充测试（`partially_copy` 只复制伴奏层）**：`<Audio 1>` 声明 `partially_copy` 只取伴奏层作为画外配乐、`<Audio 2>` 作 `reference` 音色参考、歌词由角色重新演唱——用同一份提示词跑「云朵女声 / 鹄仙本声 / 男性角色挂男声」三条，验证「只复制伴奏、人声换音色」这条路径。该轮同时暴露了上面 §一 的判定陷阱。

逐条测试数据与成片见项目内 `H3音频参考测试/`（HTML 报告生成器 `gen_report.py` ＋ 渲染自检 `verify_audio_report.mjs`）。

---

## 鸣谢 / 来源（Credits）

本分享包是"站在别人肩膀上 + 自己实跑"的产物，原始能力归属如下，特此鸣谢：

| 来源 | 链接 | 用在本包的哪里 |
|---|---|---|
| **MiniMax 官方 H3 提示词指南** | https://github.com/MiniMax-AI/MiniMax-H3 | `h3-prompt-writing`、`h3-prompt-writing-cn`、`h3-video-prompt-cn` 的**六段结构与标签规范**（`subject_definitions` / `summary` / `retention_analysis` / `detailed_description` / `overall_soundscape` / `non_diegetic_music`）、`<Picture N>` 等占位符规则，均源自官方。本包只做中文产出版与本地化改造，**未改动官方规范本身** |
| **cinematic-director-camera-dsl**（Sidekick Animation Studio Ltd.，MIT） | https://github.com/SK-SL-source/cinematic-director-camera-dsl | `h3-camera-motion-lib` 的**运镜族划分、镜头语言分层（camera core / clarifier 四层）、命令语义与部分实测证据**参考自此项目。本库已**去掉对它的运行时依赖**（不装也能直接用镜头句），但结论与证据链受益于它——特此鸣谢原作者 |
| **loopforge0 / minimaxh3-shots-skills**（MIT） | https://github.com/loopforge0/minimaxh3-shots-skills | 约 80 次 H3 生成的公开实测发现（幅度 / 速度词效应、甩镜必须给落点、装置名会被画进画面等），经上述 DSL 引用后**间接进入**本库 |
| **Krea2 工作流**等其余原始能力 | 各自原作者 / 官方项目 | 本分享包仅做"本地版 · 纯提示词 · 逐步确认"的改造封装 |

> 我们自己贡献的部分（不属于上述来源）：A–G 七组 46 镜、约 100 条 480P 成片 + 29 条 1080P 双采成片的对照测试结论——
> 每镜的**实测状态（✅/🟡/❌）**、推荐 LoRA 与权重、踩坑清单，以及"环境写描述式不写清单式""整圈的圈数与时长必须同时给"
> "`<Picture N>` 编号四段一致（禁止的是编号漂移，不是出现）"等**校准后的写法纪律**。

---

## 兼容性 / 免责

- 全部 skill **纯文本产出、零 API 调用**，可移植到任何能读取本地 `.workbuddy/skills/` 的 WorkBuddy 环境（含 OpenAI ChatGPT/Codex 等兼容 UI，`h3-prompt-writing` 的 `agents/openai.yaml` 仅提供可选 UI 元数据，不限制运行环境）。
- 本包**不包含**任何生图/生视频/生音乐的权重、工作流 JSON 或执行脚本；生成执行完全在你本地环境完成。
- 原始能力归属见上文[「鸣谢 / 来源」](#鸣谢--来源credits)；Krea2 工作流、`h3-prompt-writing` 等归各自原作者 / 官方项目，本分享包仅做"本地版 · 纯提示词 · 逐步确认"的改造封装。

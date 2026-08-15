---
name: image-prompt-writer-local
description: 本地版（逐步确认式 · 纯提示词）。专门的文生图提示词 skill：把"过简提示词"升级为「通用风格锚 + 仿 xlsx 单段富描述」的写法——整段输出、不用分四段式、画面禁可读文字/编号/水印；覆盖人像/风景/海报/产品静物/抽象概念/角色四视图等场景模板；被 8 个本地版视频 skill 在生图步骤调用，也可独立使用。不硬编码任何生图工具路径，不依赖 MiniMax Hub。
---

# Image Prompt Writer（本地版 · 富结构文生图提示词）

> 本 skill 专门解决一个痛点：**文生图提示词过于简单、信息量太少**。它把一张图的描述写成「**通用风格锚**（整套恒定）+ **单段富描述**（仿单段式角色设计表的整段写法）」，显著提升 ComfyUI / Krea2 出图的一致性与可控性。

## 何时调用

- **被 8 个本地版视频 skill 调用**：当它们进入生图步骤（角色卡 / 场景卡 / anchor 图 / 静帧 / 确认图 / 参考卡 / 视觉预览图），应调用本 skill，传入【场景类型 + 风格上下文 + 主体/要求】，由本 skill 产出富提示词。调用方拿到后按自身门控硬规则暂停，等用户返回图像地址。
- **独立使用**：用户直接要文生图提示词时，直接调用本 skill。

## 核心方法：两段式（锚 + 单段富描述）

### A. 通用风格锚（每个提示词集合只写一次）

一段**恒定文本**，描述该集合共用的视觉语言，所有分图共用以保证整套统一：

- 画风 / 渲染质感（如：editorial paper collage / 超写实摄影 / 干净动漫插画 / 产品棚拍）
- 色调基调（如：暖棕冷调对比 / 高饱和撞色 / 低饱和清冷）
- 材质语言（如：纸感层叠 / 织物织纹 / 金属哑光）
- 光感（如：柔光漫射 / 侧逆光金边 / 丁达尔）
- 负向约束（如：no text, no logo, no watermark, no 3D render, no CGI, no photo, no low-res）

> 要点：锚内**不含任何具体主体**；它只定义"怎么画"，不定义"画什么"。所有分图在开头引用它（"同通用风格锚"）。

参考范例（纸拼贴）：
> high-end editorial paper collage, flat bold color field background, black-and-white halftone cut-paper, selective saturated cardboard accent colors, warm cream keyline outlines, soft paper drop shadows, fine uncoated paper grain texture, precise hand-torn edges, visible layer seams, tactile stop-motion collage, no 3D render, no CGI, no photograph, no neon, no glass. 画幅 16:9。

### B. 单段富描述（每张图：英文成品段 + 中文阅读段，均不分段）

> ⚠️ **铁律 1：不用分四段式。** 绝不输出「画面风格 / 核心元素 / 具体内容 / 构图与镜头」四个小节式结构，也不输出任何 `1.` `2.` `图1` `图2` `P1` `P2` 之类的编号——这些编号会被生图模型当成可读文字渲染进画面，形成"图像编号"。所有信息写进**同一段连贯英文**（可直接粘贴 Krea2），中文另写一段连贯描述供你阅读/修改。

> ⚠️ **铁律 2：画面禁文字/编号。** 每段英文结尾必须带硬约束：`The final image must contain no text, no logo, no watermark, no labels, no numbers, ...`（见各模板结尾）。这是该写法的固有约束，缺失会导致画面冒出可读文字或编号。

**英文成品段（paste-ready）**——一段到底，信息密度高，顺次包含：

- 主体身份/性别/年龄/种族、发型发色+光泽、五官与表情、肤质、服装（逐件：上装/下装/鞋/配饰，含版型/面料/颜色/细节）、姿态动作、手持物；
- 背景层次（前景→中景→远景）、材质细节、色彩分布、氛围粒子；
- 画幅比例、景别、机位角度、光线方向与时性质感、相机参数（可选）、视觉焦点、空间纵深；
- **结尾硬约束**（必须）：`The final image must contain no text, no logo, no watermark, no labels, no numbers, no distorted anatomy, no extra limbs, no low-resolution artifacts.`

**中文阅读段**——同样一段到底（不分段、不标号），与英文段一一对应，方便你核对与微调。

> **多视图（如四视图）**：在同一段里用**文字**指名各面板——"the first panel is…, the second panel is…"，绝不用 `1.`/`图1`。四视图模板见第 6 节（含强制开场句，锁定左右四格构图）。

## 场景分类模板库

> 每类都走 A 锚 + B 单段富描述（整段，不分四段）。下列是"要写进同一段"的要点清单，不是分段标题。

### 1. 人像 / 肖像（角色卡、确认图、人物参考卡）
- 要点：人物身份/性别/年龄/种族；发型发色+光泽、五官+表情+眼神光、肤质；服装逐件（上装/下装/鞋/配饰，版型/面料/颜色/细节如盘扣/刺绣/褶皱）、姿态动作、手持物；画幅/景别（面部特写/半身/全身）、机位、光线（伦勃朗/蝴蝶/侧逆/漫射柔光）、背景（虚化/纯色/场景）、相机参数（如 85mm f/1.8）、色彩基调。

### 2. 风景 / 场景（场景卡、静帧背景）
- 要点：场景类型+主体地貌/建筑；天光色/云、中景结构材质、前景植被/倒影、氛围粒子（雾/雪/光尘/雨）、撞色；画幅、机位、三分/中心/引导线、光线时段、空间纵深。

### 3. 海报 / 主视觉（品牌主图、概念海报）
- 要点：锚常含"海报构图/扁平/高对比"；主视觉主体 + 标题区占位（**标题留给后期，不在图内生成可读文字**）；**标题区用留白/占位，绝不在提示词写可读标题文字**；主体刻画、负空间、撞色块、装饰、纹理；对称/对角线、视觉重心、字版位预留、光比。

### 4. 产品 / 静物（anchor 产品图、饰品）
- 要点：锚常含"棚拍/产品摄影/材质特写"；产品本体；材质与表面处理（金属/玻璃/织物/皮革/陶瓷）、造型线条、反光高光、陪衬道具；画幅、平视/45°/微距、柔光箱/环形光、纯净背景、焦点景深。

### 5. 抽象 / 概念（超现实、极简、创意视觉）
- 要点：锚常含"超现实/极简/概念艺术"；核心意象；意象物质化、非常规组合、色彩阻断、负空间；极简/中心对称、留白、光效。

### 6. 角色四视图（四宫格角色设计表写法）
适用：角色卡需要多视图锁一致性时。**输出一段四宫格概念表提示词（英文，Krea2 直接接受成品，整段不分段）**。

⚠️ **强制开场句（铁律）**：英文四视图提示词**必须以上面这段一字不差地起手**——四格定义（面部特写 / 正面全身 / 右侧面全身 / 背面全身）由这句锁定，缺失它（只写"面部特写"之类）会导致 Krea2 只出单图或构图错乱：

> Adopt a side-by-side split-screen composition of a character concept design sheet, with the frame strictly divided from left to right into four independent panels: the first panel is a refined close-up portrait of the character's face, the second panel is a front-view full-body standing pose, the third panel is a side-view full-body standing pose, and the fourth panel is a back-view full-body standing pose with the back clearly visible. The character presented must remain exactly the same across all four panels — identical face, hairstyle, hair color, outfit, body proportions and color scheme — rendered in a clean anime illustration style, high definition, anatomically accurate, sharp edges, details legible.

**用法**：第 1 行固定为上面这句（只把角色具体特征写进后续，**不要改句式与四格定义**），紧接着用英文继续在**同一段**里追加下列描写；中文另写一段对应描述。**绝对不要**把四格拆成 `1.`/`图1`/`P1` 分段——用 "the first panel… the second panel…" 文字指名。

- 跨面板强制一致：同服装、同发型发色、同配色、同身体比例——四格读起来是"同一个角色"。
- 逐格细节（写进同一段，用文字指名）：P1 面部（详细虹膜+自然眼神光、平滑肤质带细微毛孔、柔和腮红、发丝根根分明、平静自信微微笑、自然妆容）；P2 正面（中性站姿双臂放松、重心均匀、从头到脚完整、正面服装可读：领口/门襟/缝线/口袋位置/下摆/装饰、躯干与四肢褶皱自然）；P3 右侧面（同款服装侧面廓形、袖裤宽、垂坠、侧缝、肩腰臀曲线，与正面无增减）；P4 背面（后中缝、后扣/后带、后背镂空或刺绣、腰带、发后造型、鞋后跟完整）。
- 材质精准：织物织纹/缝线/光泽、发丝单根与软体积、配饰（包/帽）每格一致。
- 背景：干净浅灰棚拍、均匀柔光（主光塑形+补光消除硬影）、四格均匀曝光、视觉平衡。
- 风格：干净动漫插画、高清、解剖准确、边缘锐利、细节可读。
- **结尾硬约束（必须）**：`The final image must contain no text, no logo, no watermark, no labels, no numbers, no distorted anatomy, no extra limbs, no inconsistent clothing between panels, no cropped feet or head, no clutter, no low-resolution artifacts, delivering a polished, copy-ready character concept design sheet.`

## 输出格式契约（Output Contract）

每个生图集合严格按以下结构输出：**先锚后图、每图 = 英文成品段（整段）+ 中文阅读段（整段）**。这是硬性格式，照抄结构、替换内容即可。

### 通用风格锚（全套恒定，英文成品，顶部展示一次，末尾带画幅）
> high-end editorial paper collage, flat bold color field background, black-and-white halftone cut-paper, selective saturated cardboard accent colors, warm cream keyline outlines, soft paper drop shadows, fine uncoated paper grain texture, precise hand-torn edges, visible layer seams between paper pieces, tactile stop-motion collage, no 3D render, no CGI, no photograph, no neon, no glass. 16:9.

---

### S1 标题组 + 海螺剪影（海报/主视觉）
**英文成品段（English · 可直接粘贴进 Krea2）**
> A high-end editorial paper collage on a flat warm-cream color field, featuring a conch silhouette cut from layered navy cardboard at center with a clear spiral outline traced in warm-cream fine keyline and a soft paper drop shadow beneath it; beside it an off-white cardboard title plaque spelling 'MiniMax 稀宇科技' in paper letters (a paper label, not screen UI), a thin warm-cream keyline frame around the whole composition; visible fine uncoated paper grain, hand-torn edges and seams between layers; black-and-white halftone cut-paper, selective saturated cardboard accent colors, soft paper drop shadows, tactile stop-motion collage feel; 16:9 horizontal, medium shot, center slightly upper-symmetric, front level view, soft diffuse light from top-left warm tone, visual focus on conch and title, minimal depth from color field and border. The final image must contain no text other than the paper title plaque, no logo, no watermark, no labels, no numbers, no 3D render, no CGI, no photograph, no neon, no glass, no low-resolution artifacts.

**中文阅读段**
> 暖奶油平涂色场上的高端编辑纸拼贴：中心一枚层叠藏青卡纸剪出的海螺（conch）剪影，螺旋轮廓以暖奶油细 keyline 勾出、下方投柔和纸影；旁一块米白卡纸标题牌以纸字母拼出「MiniMax 稀宇科技」（纸标签非屏幕字）；整幅外缘暖奶油细框；可见细纸纹、手撕边与层缝；黑白半色调切纸、选择性饱和纸板强调色、柔软纸影、触觉定格拼贴感；16:9 横、中景、中心略偏上对称、正面平视、左上柔漫射暖光、焦点海螺与标题、色场与边框建立极简纵深。画面除纸标题牌外不含任何可读文字、Logo、水印、标号、3D、CGI、照片、霓虹、玻璃、低清瑕疵。

> ⚠️ 反例（**禁止**）：输出 `画面风格：… / 核心元素：… / 具体内容：… / 构图与镜头：…` 四个分段，或任何带 `1.` `图1` `P1` 的编号。这既违背"仿 xlsx 单段写法"，又会在画面里渲染出"图像编号"。必须整段输出。

## 调用约定（给 8 个本地版视频 skill）

调用方在生图步骤应：

1. **调用本 skill**，并传入：
   - `场景类型`（人像 / 风景 / 海报 / 产品静物 / 抽象概念 / 角色四视图，或组合）
   - `风格上下文`（该视频 skill 自身的统一风格锚，或从上游 brief 提炼的画风）
   - `主体与要求`（如"主角：狐狸精灵，棕粉配色；需四视图" / "场景：雪山之巅古建飞檐"）
2. 本 skill 返回：**通用风格锚（英文，一次）** + **每张图的【英文成品段 + 中文阅读段】（均整段不分段，严格按「输出格式契约」结构）**。
3. 调用方按自身「门控硬规则」：输出后调用 AskUserQuestion 暂停，等用户返回图像地址，再进入视频提示词步骤（视频步会调用 `h3-prompt-writing` skill，以返回图像为参考图）。

## 输出语言与模型适配

- **默认输出每图双段**：`英文成品段`（整段，Krea2 可直接粘贴）+ `中文阅读段`（整段，供阅读/修改）。两者内容对应、都不分段、都不带编号。
- **通用风格锚用英文**（顶部展示一次，是可直接复用的成品常量）；英文成品段要把锚要点用英文展开，保证单图可独立粘贴。
- 角色四视图模板保留**英文四宫格**，且必须以第 6 节「强制开场句」那句一字不差地起手（锁定左右四格构图），贴合 xlsx 原模板、跨面板一致性描述更稳定；中文另写一段对应。
- 句末可按需要加质量后缀（如 `8K, ultra-detailed`）。

## 门控

本 skill 只负责**产出富提示词**，不实现确认门/交付物门——由调用方（8 个本地版 skill）在其生图步骤执行暂停与等图逻辑。

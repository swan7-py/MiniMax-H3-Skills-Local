---
name: image-prompt-writer-本地版
description: 本地版（逐步确认式 · 纯提示词）。专门的文生图提示词 skill：把"过简提示词"升级为「通用风格锚 + 分图四段式详描」的富结构，覆盖人像/风景/海报/产品静物/抽象概念/角色四视图等场景模板；被 8 个本地版视频 skill 在生图步骤调用，也可独立使用。不硬编码任何生图工具路径，不依赖 MiniMax Hub。
---

# Image Prompt Writer（本地版 · 富结构文生图提示词）

> 本 skill 专门解决一个痛点：**文生图提示词过于简单、信息量太少**。它把一张图的描述拆成「**通用风格锚**（整套恒定）+ **分图四段式详描**（逐图丰富）」，显著提升 ComfyUI / Krea2 出图的一致性与可控性。

## 何时调用

- **被 8 个本地版视频 skill 调用**：当它们进入生图步骤（角色卡 / 场景卡 / anchor 图 / 静帧 / 确认图 / 参考卡 / 视觉预览图），应调用本 skill，传入【场景类型 + 风格上下文 + 主体/要求】，由本 skill 产出富提示词。调用方拿到后按自身门控硬规则暂停，等用户返回图像地址。
- **独立使用**：用户直接要文生图提示词时，直接调用本 skill。

## 核心方法：两段式（锚 + 详描）

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

### B. 分图 / 分镜详描（每张图双版：中文版 + 英文版）

每张图输出**两份平行版本**，结构一一对应：

- `#### 中文版`：供你阅读、理解与修改，用中文四段式。
- `#### 英文版`：**英文、可直接粘贴进 Krea2 的成品提示词**，也用四段式（与中文版小节一一对应）。

**中文版四段**（信息量主体在「具体内容」段，要长、要具体、要带名词/材质/色彩/光线）：

1. **画面风格**：**写出完整中文风格描述**（把通用风格锚的要点用中文展开），**不要写"同通用风格锚"这种空引用** + 本图专属微调（如背景色场变化）。
2. **核心元素**：主体 + 关键陪体，一句话点明（如：中心=纸剪海螺剪影；陪体=标题纸牌）。
3. **具体内容**：逐物详描——主体形态/服装/动作/表情、背景层次（前景→中景→远景）、材质细节、色彩分布、氛围粒子。这是防"过简"的关键段，必须展开。
4. **构图与镜头**：画幅比例、景别（特写/半身/中景/全景）、机位角度（俯/平/仰/三分/中心/引导线）、光线方向与时性质感、相机参数（镜头焦段/光圈/快门/ISO，可选）、视觉焦点、空间纵深。

**英文版四段**（英文、与中文版一一对应，**是可直接粘贴的成品；禁止把四段压成一段**）：

1. **Picture Style**：完整英文风格描述（展开通用锚要点，不要写 "same as anchor"）+ 本图专属微调。
2. **Core Element**：主体 + 关键陪体。
3. **Specific Content**：逐物详描（与中文版内容对应，纯英文）。
4. **Composition & Lens**：画幅、景别、机位、光线方向、相机参数、焦点、纵深。

> 关键：**英文版是成品提示词**（Krea2 可直接粘贴），所以四段都要展开写全，不要缩成一句话。通用风格锚在顶部只展示一次作为"参考基准"，英文版每图仍要把风格要点写进 `Picture Style` 段，保证单图可独立粘贴、单独修改。

## 场景分类模板库

> 从「往期提示词合集」归纳出的成熟结构，调用方按图索骥填入即可。每类都强制走 A 锚 + B 四段式。

### 1. 人像 / 肖像（角色卡、确认图、人物参考卡）
- 画面风格（锚）
- 核心元素：人物身份 / 性别 / 年龄 / 种族
- 具体内容：发型发色+光泽、五官特征+表情+眼神光、肤质、服装（逐件：上装/下装/鞋/配饰，含版型/面料/颜色/细节如盘扣/刺绣/褶皱）、姿态与动作、手持物
- 构图与镜头：画幅、景别（面部特写/半身/全身）、机位、光线（伦勃朗光/蝴蝶光/侧逆光/漫射柔光）、背景（虚化环境/纯色/场景）、相机参数（如 85mm f/1.8）、色彩基调

### 2. 风景 / 场景（场景卡、静帧背景）
- 画面风格（锚）
- 核心元素：场景类型 + 主体地貌/建筑
- 具体内容：天空/云/天光色、中景地貌或建筑结构与材质、前景植被/岩石/水面倒影、氛围粒子（雾/雪/光尘/雨）、色彩与撞色（如暖橙×冷紫）
- 构图与镜头：画幅、机位（俯瞰/平视/仰视）、三分法/中心构图/引导线、光线时段（日出/黄昏/蓝调）、空间纵深

### 3. 海报 / 主视觉（品牌主图、概念海报）
- 画面风格（锚，常含"海报构图/扁平/高对比"）
- 核心元素：主视觉主体 + 标题区占位（注明：**标题留给后期，不在图内生成可读文字**）
- 具体内容：主体刻画、负空间、撞色块、装饰元素、纹理
- 构图与镜头：对称/对角线、视觉重心、字版位（预留）、光比

### 4. 产品 / 静物（anchor 产品图、饰品）
- 画面风格（锚，常含"棚拍/产品摄影/材质特写"）
- 核心元素：产品本体
- 具体内容：材质与表面处理（金属/玻璃/织物/皮革/陶瓷纹理）、造型线条、反光与高光、陪衬道具
- 构图与镜头：画幅、平视/45°/微距、柔光箱/环形光、纯净背景（渐变/纯色）、焦点与景深

### 5. 抽象 / 概念（超现实、极简、创意视觉）
- 画面风格（锚，常含"超现实/极简主义/概念艺术"）
- 核心元素：核心意象（一句话概念）
- 具体内容：意象的物质化描写、非常规组合、色彩阻断、负空间
- 构图与镜头：极简构图/中心对称、留白、光效

### 6. 角色四视图（来自「单人美女角色设计提示词.xlsx」固有写法，原三视图升级为四视图）
适用：角色卡需要多视图锁一致性时。**输出一段四宫格概念表提示词**（英文或中文皆可，xlsx 原模板为英文、Krea2 接受）：
- **画幅严格左右均分四格**：① 面部特写 ② 正面全身 ③ 右侧面全身 ④ 背面全身（背面清晰可见）
- **跨面板强制一致**：同服装、同发型发色、同配色、同身体比例——四格读起来是"同一个角色"
- **逐格描写**：
  - P1 面部：清晰有神的眼睛（详细虹膜+自然眼神光）、平滑肤质带细微毛孔、柔和腮红、发丝根根分明、表情平静自信微微笑、自然妆容
  - P2 正面：中性站姿双臂放松、重心均匀、从头到脚完整；正面服装可读（领口/门襟/缝线/口袋位置/下摆/装饰），躯干与四肢褶皱自然
  - P3 右侧面：同款服装侧面廓形、袖裤宽、垂坠、侧缝、肩腰臀曲线，与正面无增减
  - P4 背面：后中缝、后扣/后带、后背镂空或刺绣、腰带、发后造型、鞋后跟完整
- **材质精准**：织物织纹/缝线/光泽、发丝单根与软体积、配饰（包/帽）每格一致
- **背景**：干净浅灰棚拍、均匀柔光（主光塑形+补光消除硬影）、四格均匀曝光、视觉平衡
- **风格**：干净动漫插画、高清、解剖准确、边缘锐利、细节可读
- **负向**：no text, no logo, no watermark, no labels, no distorted anatomy, no extra limbs, no inconsistent clothing between panels, no cropped feet or head, no clutter, no low-resolution

## 输出格式契约（Output Contract）

每个生图集合严格按以下结构输出：**先锚后图、每图双版（中文版 + 英文版）**。这是硬性格式，照抄结构、替换内容即可。

### 通用风格锚（全套恒定，英文成品，顶部展示一次，末尾带画幅）
> high-end editorial paper collage, flat bold color field background, black-and-white halftone cut-paper, selective saturated cardboard accent colors, warm cream keyline outlines, soft paper drop shadows, fine uncoated paper grain texture, precise hand-torn edges, visible layer seams between paper pieces, tactile stop-motion collage, no 3D render, no CGI, no photograph, no neon, no glass. 16:9.

---

### S1 标题组 + 海螺剪影（海报/主视觉）
#### 中文版
- **画面风格**：高端编辑纸拼贴，平坦大胆的色域背景，黑白半色调切纸，选择性饱和纸板强调色，暖奶油色关键线轮廓，柔软的纸张阴影，精细的未涂层纸张纹理，精确的手工撕裂边缘，纸张之间可见的层缝，触觉定格拼贴，无3D渲染，无CGI，无照片，无霓虹灯，无玻璃；暖奶油色场微调。
- **核心元素**：中心＝藏青纸剪海螺 silhouette；陪体＝「MiniMax 稀宇科技」纸标题牌、暖奶油 keyline 边框。
- **具体内容**：中心一枚由层叠藏青卡纸剪出的海螺剪影，螺旋轮廓清晰，暖奶油细描边勾出外形，下投柔和纸影；其旁一块米白卡纸标题牌，以纸拼字母呈现「MiniMax 稀宇科技」（纸标签，非屏幕字体）；整幅外缘一圈暖奶油 keyline 细框；背景平涂暖奶油大色场无渐变；纸面可见细无涂布纸纹与 hand-torn 撕边，层间留可见缝。
- **构图与镜头**：16:9 横向；中景；中心略偏上对称；正面平视；柔漫射光来自左上、暖调；视觉焦点海螺＋标题；空间由色场与边框建立极简纵深。

#### 英文版（English · 可直接粘贴进 Krea2）
- **Picture Style**: high-end editorial paper collage, flat bold warm-cream color field background, black-and-white halftone cut-paper, selective saturated cardboard accent colors, warm cream keyline outlines, soft paper drop shadows, fine uncoated paper grain texture, precise hand-torn edges, visible layer seams between paper pieces, tactile stop-motion collage, no 3D render, no CGI, no photograph, no neon, no glass.
- **Core Element**: center = navy cardboard paper-cut conch silhouette; accompanying = 'MiniMax 稀宇科技' paper title plaque, warm cream keyline border.
- **Specific Content**: a conch silhouette cut from layered navy cardboard at center, clear spiral outline, warm cream fine outline tracing its shape, soft paper drop shadow beneath; beside it an off-white cardboard title plaque spelling 'MiniMax 稀宇科技' in paper letters (paper label, not screen UI); a thin warm-cream keyline frame around the whole composition; background is a flat warm-cream color field with no gradient; visible fine uncoated paper grain and hand-torn edges, visible seams between layers.
- **Composition & Lens**: 16:9 horizontal; medium shot; center slightly upper-symmetric; front level view; soft diffuse light from top-left, warm tone; visual focus conch + title; space built with minimal depth from color field and border.

> ⚠️ 反例（**禁止**）：只给一段英文 blob，如 `英文提示词（可直接粘贴）：high-end editorial paper collage, flat bold...（一整段）`。这丢失了四段式信息、且难与中文版修改点一一对应。必须拆成 `Picture Style / Core Element / Specific Content / Composition & Lens` 四个小节。

## 调用约定（给 8 个本地版视频 skill）

调用方在生图步骤应：

1. **调用本 skill**，并传入：
   - `场景类型`（人像 / 风景 / 海报 / 产品静物 / 抽象概念 / 角色四视图，或组合）
   - `风格上下文`（该视频 skill 自身的统一风格锚，或从上游 brief 提炼的画风）
   - `主体与要求`（如"主角：狐狸精灵，棕粉配色；需四视图" / "场景：雪山之巅古建飞檐"）
2. 本 skill 返回：**通用风格锚（英文，一次）** + **每张图的【中文版 + 英文版】双版四段式提示词**（严格按「输出格式契约」结构）。
3. 调用方按自身「门控硬规则」：输出后调用 AskUserQuestion 暂停，等用户返回图像地址，再进入视频提示词步骤（视频步会调用 `h3-prompt-writing` skill，以返回图像为参考图）。

## 输出语言与模型适配

- **默认输出双版**：`中文版`（中文四段式，供阅读/修改）+ `英文版`（英文四段式，Krea2 可直接粘贴的成品）。两者结构一一对应。
- **通用风格锚用英文**（顶部展示一次，是可直接复用的成品常量）；`英文版` 的 `Picture Style` 段要把锚要点用英文展开，保证单图可独立粘贴。
- 角色四视图模板保留**英文四宫格**（贴合 xlsx 原模板，跨面板一致性描述更稳定）。
- 句末可按需要加质量后缀（如 `8K, ultra-detailed`）。

## 门控

本 skill 只负责**产出富提示词**，不实现确认门/交付物门——由调用方（8 个本地版 skill）在其生图步骤执行暂停与等图逻辑。

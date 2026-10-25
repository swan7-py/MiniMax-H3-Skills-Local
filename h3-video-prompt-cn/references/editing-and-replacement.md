# 源视频替换 / 编辑类（Ref2VA 六段）

> 来源：H3 替换／编辑类实跑总结，叠加本 skill 既有三条硬纪律（参考图角色、否定式闸门、书写顺序）
> 适用：把源视频里的某个对象替换 / 修改成参考图的样子 —— 换脸、换人、换服装、换手持物、换场景背景、换画内文字、换材质、换光照，以及移除与新增

---

## 一、先判任务类型：这决定 `summary` 的固定首句

| 情形 | 任务类型标记 | `summary` 首句 |
|---|---|---|
| 有参考对象，替换源物 | `[video editing + reference generation]` | **`The target video is an edited version of <Video 1>.`** |
| 只改材质 / 风格，不改形状 | 同上 | 同上 |
| **无参考，移除源物** | `[video editing]`（去掉 reference generation） | 同上，**且不建 `<Subject 2>`** |
| 在原源视频中新增元素 | `[video editing]` | 同上 |
| 原声仍可闻 | 追加 ` + audio reuse` | 同上 |
| **只借运动 / 机位 / 节奏，不改源片**（见 `motion-transfer.md`） | 只用 `[reference generation]` | 普通参考生成写法 |

> ⚠️ **先分清楚本类与「动作迁移类」**：本类的源视频是**被编辑的对象**，改完之后画面看着还是那条源视频；
> 动作迁移类的源视频是**运动来源**，改完之后是一条新片子。
> 判据一句话：**成片出来后，源视频的画面内容还在不在？在 → 本类；不在 → 去读 `motion-transfer.md`。**

## 二、统一模型：三个归属维度

这类任务不是「把 A 换成 B」，而是把三个维度拆给两个不同来源：

| 维度 | 来源 | 决定什么 |
|---|---|---|
| **外观（是什么）** | 参考素材 `<Subject 2>` | 形状、材质、颜色、纹样、身份、文字内容 |
| **运动与几何（在哪、怎么动、多大、贴不贴）** | 源视频 `<Video 1>` | 位置、朝向、轮廓、位移轨迹与速度、姿态、占位体积、接触与贴合 |
| **光学（受什么光）** | 源视频环境 | 光源方向、色温、反差、接触阴影、反射性质、景深 |

**漏一条的后果是固定的**：

- 漏外观 → 换了个不像的东西
- **漏运动几何 → 目标自己乱动、脱手、穿模、脱离身体**（最常见，也最致命）
- 漏光学 → 参考图的影棚光被搬进场景，目标像贴纸一样浮在画面上

## 三、遮罩形态：半透明为默认路径

先检测再动笔。素材里的被替换对象通常有两种遮罩：

| | **半透明叠加（默认）** | **不透明遮罩（回退）** |
|---|---|---|
| 表现 | 薄色层盖在目标上，轮廓 / 结构 / 接触关系仍可读 | 平坦纯色，内部无结构 |
| 均值 R 通道 | 明显偏高（经验值 **R ≈ 80**） | 极低（R < 40）且方差小 |
| 运动几何信息 | 完整保留 | 已被抹除 |
| 正确写法 | 显式声明运动与几何由源视频驱动，只换外观 | 几何约束改从周边取证（接触物、投影、环境光、相邻物体），运动只能按上下文合理重建 |

检测方法：抽帧放大目视 + 算遮罩内均值 RGB；再数遮罩边缘被哪些元素穿过、穿过几次（手指、发丝、前景物、肩带）——**穿过几个元素就写几条遮挡句**。

## 四、目标类型 → 槽位速查（本文件的核心）

确定了目标类型，`<Subject 2>` 写什么、几何锚点写什么、不迁移项列什么，直接照抄：

| 目标类型 | `<Subject 2>` 外观槽位 | 几何／运动锚点（源视频提供） | 必列的不迁移项 |
|---|---|---|---|
| 人物身份 / 肖像 | 脸型、眉形、眼型、鼻梁、唇形、肤色与皮肤质感、辨识特征 | 头姿与转角、发际线位置、表情、眼神、眨眼节奏、口型时序、举止 | 参考图的发型、妆容、头饰、服装、背景、影棚光 |
| 整个人物（含服装，即"换人"） | 发型发色、脸、身形比例、上下装鞋、配饰 | 站姿、体位与重心、四肢运动、脚掌落点、身高与站位宽度、被遮蔽物重叠顺序 | 参考图的背景、光照、其自身姿势与机位 |
| 服装 / 配饰 | 版型（领口/袖长/腰线/肩线）、面料与垂坠、颜色纹样、五金缝线、磨损 | 身体姿态驱动的褶皱与摆动、下摆袖口轨迹、腰带贴合、头发压衣领的层次 | 参考图的模特体型与姿势、平铺形态、背景与平光 |
| 手持 / 场景中的物体 | 形状尺寸、主色纹路、材质与高光性质、附属细节、磨损 | 位置朝向、被手指包裹的接触点、接触阴影、位移轨迹与速度、深度顺序 | 参考图的背景、顶光、产品摄影打光、完整未被握持的形态 |
| 场景 / 背景 | 建筑植被天际线、地面材质、大气能见度、色彩层次、景深分布 | 透视与视差、主体在景深中的位置、镜头与画幅不变、前景遮挡层次 | 参考图的相机与镜头、拍摄时间与光照、其自身透视 |
| 画内文字 / Logo | 文本逐字（原文照录、不翻译）、字体族字重、字号行距、颜色、留白比例 | 贴附表面的透视与角度、随表面的形变、被遮挡部分 | 参考图自身的背景、色温、透视 |
| 材质 / 表面（只改质感） | 粗糙度、反射方式、纹理尺度与方向、污渍磨损 | 原物体的形状轮廓位置运动**全部不变** | 参考图的形状与体积 |
| 光照 / 影调风格 | 光源方向与性质、色温、反差、高光滚降、肤色倾向 | 场景结构、人物表演、机位全部不变 | 参考图的内容本身（只用其影调） |
| 移除类 | —（不建 `<Subject 2>`） | 空缺回填：背景如何延伸、光影如何回填、原投影与接触阴影如何消失 | — |
| 新增类 | 同「物体」列 | 插入位置与相对参照物、深度顺序、投射阴影与接触点 | 同「物体」列 |

### 换人四条专属要点

1. **身高、站位宽度、脚掌落点跟随源人物**，外观来自参考图 —— 否则新人会撑破或缩进原遮罩轮廓，边缘露接缝。
2. **鞋型改变要处理脚部姿态**：换高跟时写 `the heel angle adapts so the shoes rest cleanly on the ground`，脚掌落点与地面接触点不变。
3. **与未遮罩人物的接触与重叠**：牵手 / 手臂交叠要逐处写明重叠顺序与接触点 `remain in front of or behind her exactly as before`，漏写会穿模或让手臂消失。
4. **地面投影不是遮罩内容**：保留其方向、长度、柔度，并说明它随新剪影更新。

一个画面里多个目标时按**首次出现顺序**编号 `<Subject 2>`、`<Subject 3>`…，各自独立遮罩、独立几何锚点、独立遮挡句；同一目标跨镜头编号不变。**不被替换的人也要单独编号并标 `fully_preserved`**，否则模型可能顺手把她也改了。

## 五、七个不可省的句子模块

任何类型都齐备六个模块，人物替换再补第七个 `{{PERFORMANCE_CLAUSE}}`：

| 模块 | 作用 | 换脸写法 | 换物体写法 | 换服装写法 |
|---|---|---|---|---|
| `{{REBUILD_SENTENCE}}` | 声明重建结果 | `Her face is rebuilt with the identity of <Subject 2>: <特征清单>, following the head angle, tilt, and hairline already present in the source.` | `The object in her grip is rebuilt as <Subject 2>: <特征清单>, following the orientation, stem direction, and position in her grip already present in the source.` | `The garment is rebuilt as <Subject 2>: <版型面料清单>, following the body's rotation and the drape already present in the source.` |
| `{{LIGHT_CLAUSE}}` | 光学取自源场景 | …`adapted to the balcony's warm ambient light so the rebuilt face and her visible neck read as one continuous skin tone` | …`takes the warm lamplight from the left rather than the reference image's overhead product light` | …`catches the room's warm light from the right, with folds shading the same way as the fabric already in the source` |
| `{{SYNC_CLAUSE}}` | 运动几何归源视频 | 见 `{{PERFORMANCE_CLAUSE}}` | `Its position, orientation, silhouette, velocity, and motion path continue unchanged from <Video 1>; it neither accelerates nor slows down` | `Its fit, folds, sway, and hem movement follow her body motion exactly as the garment in <Video 1> does` |
| `{{PERFORMANCE_CLAUSE}}` | 可见行为复刻（人物类必填） | 四组全写，见下节 | — | — |
| `{{OCCLUSION_SENTENCE}}` | 遮挡与深度顺序 | `The hair strands crossing the frame-left edge of her face keep their exact position and overlap over the new face` | `The fingers in front of it stay in front of it, the thumb rests on the same upper surface` | `The hair falling over the collar stays in front of it, and her hands stay in the same pockets` |
| `{{FIT_CLAUSE}}` | 占位 / 贴合 / 透视适配 | `read as one continuous skin tone` | 尺寸规则 R1（见第七节） | `it follows her body pose and proportions rather than the reference's flat-lay or model pose` |
| `{{SEAM_AND_EXCLUDE}}` | 接缝 + 不迁移 | `the join between the new face, her hairline, and her jawline stays continuous with no seam` | `the join between it and the unmasked pixels around her fingers stays continuous with no seam` | `the join at the collar, cuffs, and hem stays continuous with no seam` |
| `{{NO_ADDED_EVENT}}` | 收口：不加戏 | `Nothing that is not visible in <Video 1> is added: no new action, no new intention and no new event appears in the target video.` | 同左 | 同左 |

## 六、人物类必填：可见行为四组

四组都要带**同节奏约束**（`on the same beats` / `at the same moments`），缺一组就漂移：

| 组 | 逐项内容 | 英文写法 |
|---|---|---|
| **表情** | 眉高眉形、眼睑开合度、眼眶与嘴角松紧、唇形开合 | `the same facial expression, with the same brow height and angle, the same eyelid height and the same softness or tension around her eyes and the corners of her mouth` |
| **神态** | 面部整体状态的逐拍读感（舒展／戒备／被逗到／平静） | `the same demeanour as it reads on her face from moment to moment — how open, how guarded, how amused, how composed — shifting on exactly the same beats` |
| **眼神** | 视线落点与距离、漂移与回位、眨眼节奏与幅度 | `the same gaze, her eyes resting on the same point at the same distance with the same drift and return; the same blink rhythm` |
| **举止** | 头转 / 偏 / 点、肩与躯干摆动、手部动作、口型时序 | `the same visible behaviour, with the same head turns, tilts and nods, the same shoulder and torso sway, and the same lip shapes and mouth movement at the same moments` |

**遮罩盖住面部时**：可见表演已被抹掉，不能靠猜补 —— 只保留可取证部分（头姿 / 肩线 / 来自音轨的口型时序），其余交回参考素材，不写情绪形容词。

**只写可见事实**（所有类型通用）：意图、身份、剧情、画外因果一律不写。`she is singing along to a song` 是猜测 → 改 `her brows lift and her mouth corners pull up into a closed-mouth smile`。场景同理，只写陈设与光线，不写「她在直播间」这类用途推断。有音轨但听不出内容时只写 `her lips move and keep their exact shapes and timing from <Video 1>`。

## 七、尺寸冲突：R1 / R2 / R3

参考对象与源物体积差异大时（苹果 → 西瓜），「保持抓握关系」与「按参考真实尺寸」不能同时成立：

| 规则 | 写法 | 代价 |
|---|---|---|
| **R1 保持占位（默认）** | `sized to fill the same space in her hand as the object in <Video 1>` | 物体偏小，但抓握、遮挡、接触阴影全部自洽，不穿模 |
| **R2 忠于参考尺寸** | `rendered at the true size of <Subject 2>` + 改写接触关系 | 会改动源运动，与「保持源运动不变」冲突，须在正文承认手部姿态被调整 |
| **R3 折中** | `allowed to extend beyond the source object's footprint by up to about half its width while the grip points stay fixed` | 略大于手可接受；超过一倍易被手指插穿 |

服装类方向相反：**必须是身体驱动衣服**（`follows her body pose`），不能是衣服保持参考图的平铺 / 模特姿态 —— 这是服装替换最容易错的一点。

## 八、占位符骨架

```
subject_definitions:
<Subject 1> is {{SUBJECT1_ROLE}} in <Video 1>, with {{SUBJECT1_APPEARANCE}}.
<Subject 2> is {{REF_OBJECT_DEFINITION}}: {{SUBJECT2_FEATURES}}.
<Video 1> is the source video for the target video edit: {{VIDEO1_DESCRIPTION}}. In the source, {{MASK_FORM_SENTENCE}}; it marks the area to be rebuilt and is not part of the target video's visible content. {{OUTSIDE_MASK}}
{{AUDIO_DEF}}

summary:
[video editing + reference generation{{AUDIO_TASK}}] The target video is an edited version of <Video 1>. The masked region is fully rebuilt as <Subject 2> from <Picture 1>, rendered in natural untinted colour and following {{GEOMETRY_ANCHORS}} already present in the source{{SYNC_SUMMARY}}. {{RETAIN_LIST}} all stay unchanged, and no green region appears anywhere in the target video{{NO_EXTRA_DIALOGUE}}.

retention_analysis:
<Subject 1> ([Shot 1]): partially_preserved - {{SUBJECT1_RETAIN}} are retained, {{CHANGE_CLAUSE}}.
<Subject 2> ([Shot 1]): attribute_transfer - <Subject 2> is transferred onto {{HOST_NOUN}} in <Video 1>, adopting the source's {{GEOMETRY_ANCHORS}}; {{SUBJECT2_EXCLUDE}} are not carried over.
<Video 1> (edit source): partially_preserved - {{VIDEO1_RETAIN}} are retained, while the source's masked region is interpreted only as the rebuild area and is not reproduced.
{{AUDIO_RETENTION}}

detailed_description:
{{STYLE_SENTENCE}}
[Shot 1] {{SHOT1_OPENING}} {{REBUILD_SENTENCE}} {{LIGHT_CLAUSE}} {{SYNC_CLAUSE}} {{PERFORMANCE_CLAUSE}} {{OCCLUSION_SENTENCE}} {{FIT_CLAUSE}} {{SEAM_AND_EXCLUDE}} {{FINAL_RETAIN_CLAUSE}} {{NO_ADDED_EVENT}}

overall_soundscape:
{{SOUNDSCAPE}}

non_diegetic_music:
N/A
```

- **移除类**：删 `<Subject 2>` 定义行与保留行，任务类型降为 `[video editing]`，`{{REBUILD_SENTENCE}}` 改为空缺回填句。
- **无音轨**：删 `<Audio 1>` 行，去掉 `{{AUDIO_TASK}}`，`{{SOUNDSCAPE}}` 改环境声并补 `; no dialogue and no on-screen speech`。
- `retention_analysis` 里**不写 `(Sx)`**（`<发声者>` 除外）。

## 九、本 skill 的三条硬纪律如何落到编辑类

1. **`<Picture N>` 只出现在 `subject_definitions` 与 `retention_analysis`**：仅用于定义外观的参考图**内联进 `<Subject 2>`**，不单独成条；`summary` 与 `detailed_description` 里 `<Picture` 计数必须为 **0**。
2. **否定式两级闸门**照样适用：`seamless` 类的否定一律改成正向（`carries no trace of the source overlay` → `the join stays continuous with no seam`）；「不要参考图的棚光」要写成**替换式**（`takes the warm lamplight from the left rather than the reference image's overhead product light`）。详参 `negation-guard.md`。
3. **书写顺序铁律**（`writing-order.md`）在遮挡句上最容易破功：**先出现的先写**，被压在最上层的元素写在最前。

## 十、校验清单

- [ ] 任务类型标记与 `summary` 固定首句匹配；六段齐全、顺序正确、无改名
- [ ] 仅定义外观的参考图没有单列 `<Picture N>`，而是内联进 `<Subject 2>`
- [ ] `<Subject 2>` 标 `attribute_transfer`；承载它的主体 / 物体标 `partially_preserved`
- [ ] `retention_analysis` 无 `(Sx)`；`<Subject 2>` 的不迁移项逐项列出
- [ ] **运动归属句在，且按类型写对**（脸 = 表演 / 物 = 位移 / 衣 = 随体 / 背景 = 视差）
- [ ] 人物类四组可见行为写全，每组带同节奏约束
- [ ] 光学取自源场景；参考图自己的打光已列进不迁移项
- [ ] 遮挡句单列，深度顺序与遮盖范围写明"保持原位"
- [ ] 适配句在（物体写尺寸规则 R1/R2/R3；服装写"身体驱动衣服"；背景写透视对齐）
- [ ] 收口句在：*Nothing that is not visible in <Video 1> is added…*
- [ ] 无目的 / 身份 / 剧情推断；无 `swap / replace / chroma / green screen / mask` 等后期术语
- [ ] 重建内容与参考图 / 用户文字逐项对得上（形状 · 颜色图样 · 材质光学 · 附属细节），无漏项无加戏
- [ ] 过否定式闸门；`<Picture` 在 summary / detailed_description 计数为 0
- [ ] 会被取下 / 移开的物件已独立成 `<Subject>`，主体外观串里没有它
- [ ] `<Video 1>` 的主体指代带了一个画面可见特征（如「带着耳麦的歌手」）
- [ ] 背景 / 光位 / 明暗只写「严格沿用 `<Video N>`」，没有光点、灯束、观众席等具体名词

## 十一、替换效果的三个提示词杠杆（2026-10-25 长片分段换人实战）

> 来源：23.9s 长片、5 段 5s 分段、turbo 4step、连改 6 轮后的收敛结论。按优先级排：**先拆 Subject，再锁指代，最后压背景**；三条都不奏效才回头改措辞。

### 11.1 杠杆一：`<Subject>` 拆不拆，决定会不会「多出一份」

同一个物件，写在谁的 `<Subject>` 里，结果完全不同：

| 写法 | 结果 |
|---|---|
| 眼镜写进 `<Subject 1>` 外观串（`面部戴一副金属细框眼镜`） | 动作要求「摘下来拿在手里」时，**脸上一副 + 手里一副** |
| 眼镜独立成 `<Subject 2>`，`<Subject 1>` 改「他的脸是裸露的」 | 脸干净，眼镜只在手里一副 ✅ |

**判据：片中有「把某件东西从主体身上取下 / 移开」的动作时，这件东西必须独立成 `<Subject>`**，不能留在主体外观串里。

三层粒度（够用即止，不必全拆）：

1. **人** = `<Subject 1>`：脸、发型、肤色、身形、服装；**不写**任何会被取下或移动的物件；
2. **可动 / 可摘的物件** = `<Subject 2…>`：眼镜、手持物、武器、配饰，逐件一个编号，并写明它**此刻在哪**（架在鼻梁上 / 握在胸前 / 已离开脸）；
3. **必须原样保留的源片对象**（观众、其他人物、道具、坐骑）再往后编号，标 `fully_preserved`。

**待验证的更强拆法**（效果仍不满意时再试）：源片里那件物件一个 `<Subject>`、替换角色带来的同件物件另一个 `<Subject>`，各自声明，把「替换关系」显式写出来——理论上能进一步压掉新旧两份并存。**尚未测试，作为下一级手段。**

### 11.2 杠杆二：主体指代要带一个「画面可见特征」

`<Video 1>` 里那句「本片只替换其中的「歌手」这一个主体」，把「歌手」改成「**带着耳麦的歌手**」，主体锁定明显更牢。

写法：`本片只替换其中的「<可见特征> + <身份>」这一个主体`。
特征取源片第一眼能认出的东西——随身物件、发型、服装颜色、站位。笼统身份词（歌手 / 女子 / 主角）在多主体画面里指代力不足，模型会自己挑一个，甚至另造一个。

### 11.3 杠杆三：背景只写「严格沿用」，不给任何具体描写

写「接近纯黑的暗部，只有几处极淡的虚化光点」→ 模型真的给出一片光点。
实测：背景顶部条带亮度 **33.7 → 61.7**、饱和度 **0.042 → 0.132**；观众席、彩色灯束、场馆结构全部被凭空补出来。

**正确写法（只此一句）**：

```
背景、光源与明暗全部严格沿用 <Video 1> 对应时刻
```

**不要写**：光点、星芒、灯束、观众席、场馆类型、色温倾向、明暗程度。任何具体名词都会被当成「要画的东西」。

### 11.4 参数杠杆（不属于提示词）

段内自编镜头 / 背景被重画 / 首帧画出参考图版式，先查 **turbo 权重**，见 `camera-and-lora.md` §5.4。

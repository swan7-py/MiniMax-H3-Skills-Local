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
- [ ] **保留但移位**的物件（成片里仍可见）已独立成 `<Subject>`，主体外观串里没有它
- [ ] **要删除**的物件**未**编号，写在 `<Video 1>` 的「待重建区域」里；`<Video 1>` retention 标 `partially_preserved`
- [ ] `retention_analysis` 的标记只用官方四值（`fully_preserved` / `partially_preserved` / `attribute_transfer` / `weak_reference`），无自造标记
- [ ] `<Video 1>` 的主体指代**只含成片要保留的特征**，不含任何待删物件（见 §11.3）
- [ ] 删除用动词（`删除` / `只剩 X`），全文零否定词：`[^。；]{0,18}(?:不|没)[^。；]{0,18}` 扫结果为空
- [ ] 背景 / 光位 / 明暗只写「严格沿用 `<Video N>`」，没有光点、灯束、观众席、LED、光斑等具体名词

## 十一、替换 / 编辑类：四条判据与五个坑（2026-10-25 定稿）

> 来源：23.9s 长片分段换人 + 一段 5.1s / 7 镜的密集段落，累计 8 轮翻车后收敛。
> ⚠️ **本节替代初稿「三个提示词杠杆」**——初稿的「主体指代加可见特征」与本节 §11.3 正面冲突，已改写，不要按旧稿写。

### 11.1 判据一：要「保留但移位」的物件 → 独立成 `<Subject>`

典型场景：源片的动作是「把某件东西从主体身上取下、拿在手里」，成片里这件东西**仍然存在**，只是位置变了。

| 写法 | 结果 |
|---|---|
| 眼镜写进 `<Subject 1>` 外观串（`面部戴一副金属细框眼镜`） | 一脸一副 + 手里一副，**两份并存** |
| 眼镜独立成 `<Subject 2>`，`<Subject 1>` 改「他的脸是裸露的」 | 脸干净，眼镜只在手里一副 ✅ |

判据：**片中有「把某件东西从主体身上取下 / 移开」的动作，且这件东西在成片里依然可见 → 必须独立编号**，并写明它此刻在哪（架在鼻梁上 / 握在胸前 / 垂在指间）。

### 11.2 判据二：要「删除」的物件 → **不编号**，写进 `<Video 1>` 的重建区

这是初稿的最大错误，也是「拆 Subject 表达删除」走不通的原因。

**规范依据**：

- `ref.txt` §2.1：`<Subject N>` 代表「**目标视频中实际会用到的一个内容单元**」。成片里不存在的东西不满足这个定义。
- `ref.txt` §4.1：retention 的关系标记是「**固定的英文值**」，只有 `fully_preserved` / `partially_preserved` / `attribute_transfer` / `weak_reference`。**没有「删除」项**——自造 `removed` 属违规，模型读到一个不认识的英文值，既可能忽略也可能误当保留。

**正确载体（官方骨架 §八）**：写在 `<Video 1>` 里，当「待重建区域」。

```text
<Video 1> 是这段视频的源片：……在源片里，他耳侧的耳麦与麦杆标记为待重建区域，成片里该位置由头发与皮肤取代。
```

```text
<Video 1> (编辑源，动作与剪辑结构): partially_preserved - ……；源片里那个歌手耳侧的耳麦与麦杆按待重建区域处理，成片该位置由头发与皮肤取代。
```

对应英文骨架：`it marks the area to be rebuilt and is not part of the target video's visible content` / `the source's masked region is interpreted only as the rebuild area and is not reproduced`。

**连带两条**：

1. `<Video 1>` 只要有内容被重建，retention 就必须从 `fully_preserved` 降为 **`partially_preserved`**；
2. 删除件**不占 `<Subject>` 编号**，编号只留给成片里存在的对象（人 / 观众 / 保留物）。

### 11.3 判据三：待删物件**绝不能出现在主体指代里**

翻车复盘：`本片只替换其中的「带着耳麦的歌手」`——本意是锁定主体，实际等于告诉模型「这个主体是戴耳麦的」，于是耳麦四版都没删掉。

**铁律**：主体指代只写**成片里要保留的特征**（位置、体型、服装、动作），**绝不写你要删掉的那件东西**。想锁主体就用「站在舞台中央的那个歌手」这类位置特征，或用中性身份词。

（初稿的「加可见特征更牢」只对**保留类**特征成立；加了待删物 = 反向要求保留。）

### 11.4 判据四：删除用**动词**，不用否定式

| ✗ 无效 / 违规 | ✓ 有效 |
|---|---|
| `这副耳麦与麦杆不保留` | `本段删除源片里那个歌手耳侧的耳麦与麦杆` |
| `画面里没有耳麦、没有麦杆` | `他的耳侧、耳后与颈侧只剩头发与皮肤` |
| `不被替换` | `外观与表演沿用 <Video 1> 对应时刻` |

- **「不出现」是 L1 硬拦**（`negation-guard.md` §二「存在/呈现」组），出现即拒。
- 「删除 / 只剩 X」不在 L1 词表，**不违规**，且实测比「不保留」更容易生效。
- 句式：**先说删什么，紧接一句成片里是什么**。前半给动作，后半给目标状态，不留空白的否定。
- 自检正则：`[^。；]{0,18}(?:不|没)[^。；]{0,18}` 扫全文应为空。

### 11.5 背景 / 灯光 / 道具：零描述，只写「严格沿用」

写「接近纯黑的暗部，只有几处极淡的虚化光点」→ 模型真的长出一片光点。实测顶部条带亮度 **33.7 → 61.7**、饱和度 **0.042 → 0.132**；星芒灯、彩色灯束、观众席、场馆结构全部被凭空补出。

**只此一句**：

```text
场景、灯光、光位、色彩与背景全部严格沿用 <Video 1> 对应时刻。
```

**不要写**：光点、星芒、灯束、LED、光斑、观众席、场馆类型、色温倾向、明暗程度、道具颜色。任何具体名词都会被当成「要画的东西」。

`<Video 1>` 里允许留下的只有**结构信息**：第几镜、什么景别、镜头里有谁。

### 11.6 写提示词前必须核实的四个事实（都翻过车）

| 坑 | 怎么核 |
|---|---|
| **道具误认**：把耳麦的麦头当成手持麦克风，于是编出一个不存在的 `<Subject>` | 抽侧脸近景放大 1:1 看——麦头贴唇、无手接触 = 耳麦；有手握柄、有连线 = 手持麦 |
| **帧定位错位**：用 `-ss` 抽帧判切点，offset 让她偏 1–2 帧，误判整镜归属 | 只用 `select=eq(n,X)` 逐帧抽；`-ss` 只可用于粗看 |
| **错字归因**：把「带耳麦」写成「带」，怀疑是错字导致删不掉 | 错字只影响 token 形态，不影响语义。**先查策略是否自相矛盾**（见 §11.3），再谈措辞 |
| **跨段动作**：摘眼镜跨两个批次，各写各的会断 | 段间用**状态锚点**（"眼镜仍架在鼻梁上、手指刚触到镜框"），或直接合并成一段 |

### 11.7 参数杠杆（不属于提示词）

段内自编镜头 / 背景被重画 / 首帧画出参考图版式 → 先查 **turbo 权重**（见 `camera-and-lora.md` §5.4）。

实测：`turbo 4step` 0.8 → 0.9 就消除了「段内凭空补一个舞台远景 + 乐队」。为「去油」降 turbo 会牺牲参考贴合，是同一个旋钮的两端。

---

**本判据的成立案例**：5.1s / 7 镜（舞台中远景 → 侧脸近景 → 观众近景 → 侧脸近景 → 暗场 → 舞台远景 → 观众大全景）+ 删眼镜 + 删耳麦 + turbo 0.9，一次通过。**7 镜在 turbo 0.9 下可用**，此前「5 镜以上易糊」的结论据此修正为「与 turbo 权重强相关」。

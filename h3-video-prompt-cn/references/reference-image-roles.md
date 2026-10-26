# 参考图的两种角色：构图锚 vs 内联定义

> 来源：H3 运镜对照测试（B2 拉镜头"落回场景图斜角构图"的根因诊断）

---

## 一、判据（H3 规范 §2.2）

> 图片**只有当首帧 / 关键帧 / 末帧 / 编辑关键帧 / 构图锚**用时，才独立成 `<Picture N>` 一条；
> **只用来定义角色、场景、服装或风格的图，不应独立成条，而应写进对应的 `<Subject N>` 里。**

| 参考图用途 | 正确写法 |
|---|---|
| 首帧 / 末帧 / 中间关键帧 / **构图锚** | **独立成 `<Picture N>`**，并在 `retention_analysis` 里声明其在哪个时刻被 `fully_preserved` |
| 只提供**身份 / 服装** | 写进 `<Subject N>` 定义，**不独立成条** |
| 只提供**环境 / 场景** | 写进"环境"这个 `<Subject N>` 定义，retention 给 **`weak_reference`** |
| 只提供**风格** | 写进 style 句 |

## 二、最重要的坑：环境图给成构图锚，镜头会被拉回它的构图

**错误写法**（常见）：
```
<Picture 2> 是主体所处的场景（独立成条）
retention：<Picture 2>：fully_preserved - 外廊的空间结构、柱列、地砖、宫灯霓虹…全程保留
```
后果：**等于亲口把场景图声明成"构图锚"** → 模型持续把镜头拉回那张图的构图。
实测表现：拉镜头**不沿走廊纵深后退，而是落回场景图那个斜角构图**。

**正确写法**：
```
subject_definitions:
<Subject 2> 是主体所处的环境——…；其空间形制、材质与光位氛围参照 <Picture 2>。

retention_analysis:
<Subject 2>（环境，出现于 [Shot 1] 全片）：weak_reference - 只保留外廊这一类空间的形制、材质与光位氛围；
<Picture 2> 仅作环境形制与光位氛围的参照；构图与机位一律以本提示词的镜头描述为准。
```
（末句必须写成**正向**表述，不要写"<Picture 2> 不约束构图"——那是裸否定。）

## 三、环境写"描述式"，不要写"清单式"（2026-10-09 校准 · 原表述"删环境"已废）

> ⚠️ **不是把环境删掉**，而是换写法。旧表述"删环境描写是唯一有效的锁半身手段"不准确，已废。
> 完整版见专家包 `h3-camera-motion-lib/references/00-core-discipline.md` §七。

- **清单式**（逐一点名"柱列、地砖、宫灯、竹丛必须可见"）→ 模型为把所有元素塞进画面，**把画面推宽**；
- **描述式**（一句说清"主体所处的环境是什么"）→ 不推宽。
- **效果上限**：改成描述式后主体放大 **2.4–3 倍（2/3 种子）**，但**仍未达腰部以上（0/3）**
  → 它是「**别让环境把画面推宽**」的**止损手段**，**不是**「锁半身手段」。
- **官方同源依据**：`H3LAB-FRAMING-SCENE-01`——`describe what environment {SUBJECT} is in, not which environment elements must be visible; Environment lists widened the frame`。
- **真要锁住取景的正解**：**改用 I2VA 真实首帧**（首帧决定取景 / 高度 / 角度，文字只管运动）。
- **同族纪律**：写任何环境句前自问「这句是不是在指定机位朝向 / 构图方向？」——「向纵深延伸」「尽头是」「深处」都是构图指令；横移镜要给**走向句**、规避"纵深"字眼。

**建议的环境句模板**（一句，不定点）：
> `场景是一座中国古典庭院外廊与克制赛博朋克科技相融的白天实景，午后阳光自开敞一侧斜射。`
> （需要露天场景时另写明"站在外廊之外的露天庭院里，头顶是开阔的天空"——**俯视类镜头必须这样做**，否则屋檐会挡死）

## 四、首帧锚定的取舍（实测）

| | 挂首帧图 | 不挂首帧图（文字锁起手） |
|---|---|---|
| 起手跳帧 | **会跳一下**（首帧与动起来之间） | 无（前 5 帧差 0.002–0.09，平滑） |
| 起手复现性 | 高（逐帧一致） | **只能"大致一致"**（同镜跨条件差异明显） |
| 是否压制运镜幅度 | **否**（实测三镜都完整走出运动、幅度甚至更大） | — |
| 真正的差别 | 在**起点**：不挂首帧时模型会自己发明起始构图 | — |

**推荐**：需要稳定起点时用**文字锁起手**；**全身镜不宜作拉远镜头的首帧，可作尾帧**。

---

## 五、`<Picture N>` 在四段里都能用；错的是编号漂移（2026-10-26 按官方规范修订）

| 段落 | `<Picture N>` |
|---|---|
| `subject_definitions` | ✅ 必须出现（来源声明 / 独立成条定义） |
| `retention_analysis` | ✅ 必须出现（逐条声明保留关系） |
| `summary` | ✅ 可用已定义的编号，**不得在此新起编号** |
| `detailed_description` | ✅ 可用，在首次出现处与角色生效处引用 |

**官方原文依据**（`references/ref-en.txt`）：
`summary`「The summary uses the previously defined `<Subject N>`, `<Picture N>`, `<Video N>`, and `<Audio N>` labels… Do not introduce new reference labels in this section.」
`detailed_description` 对照表「Reference information → Inserts `<Subject N>`, `<Picture N>`, `<Video N>`, and `<Audio N>` at their first appearance and where their roles apply」。

**真正的错误是编号漂移**：同一张图在 `subject_definitions` 是 `<Picture 1>`，到 `detailed_description` 变成 `<Picture 2>`——模型会把它们当成两张不同的图。

**写法建议（可读性偏好，非禁令）**：讲剧情与状态用 `<Subject N>`，讲帧本身（首帧 / 末帧 / 构图对齐）才用 `<Picture N>`。

> 早期版本（2026-10-08／10-09）曾把"summary 与 detailed_description 里 `<Picture>` 计数必须为 0"写成硬规则，
> 与官方规范冲突，已于 2026-10-26 撤销。当时的实测现象是"写进运镜句里构图被拽住"，
> 该现象若复现，按排障处理（把该处换成 `<Subject N>` 试一次），不要升格为禁令。

> 完整版（含 F10"环境修补句变成构图指令"事故、双主体防复制写法）见专家包
> `h3-camera-motion-lib/references/00-core-discipline.md` §七/§八 与 `06-rig-and-device.md` F10。

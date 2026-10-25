# 镜头层纪律 与 LoRA 选型

> 来源：H3 运镜对照测试（A–G 七组 46 镜 · 约 100 条成片 · Ref2VA · 864×480 中文提示词）
> 结论等级：**初级**（每格单样本单种子；官方等级来自另一 profile，只能作方向性参考）

---

## 一、镜头层只有一个作者：camera DSL

> **2026-10-09 更新：镜头句的默认来源已改为 `h3-camera-motion-lib`（H3 运镜提示词库，专家包内的独立 skill）。**
> 那个库按运镜族给出**可直接粘贴的中文镜头句**＋实测状态，**不依赖 DSL**；本文件只作「变量纪律速查 + LoRA 选型」，
> DSL 与本文均降级为**可选校验**，冲突时以库的「实测」一栏为准。
> 时序相关的两条校准结论（未在本文件展开，见库 `00-core-discipline.md` §五/§六）：
> **① 配速＝`At` 起点–结束点两个时刻**；**② 整圈环绕的圈数与时长必须同时给**。

**镜头描述逐句直译自 `cinematic-director-camera-dsl` 的 `render` 输出，不增不减。**
镜头句只出现在 `detailed_description`；`summary` 只讲剧情（`summary` 内至多出现「一条连续长镜」「三镜交换」这种**镜头层级概括**，镜头变量一律归 `detailed_description`）。这与 `ref.txt` §3 的「镜头流向（shot flow）」含义一致：那里的 shot flow 就是这种层级概括（「一条连续长镜」），归 summary；镜头运动归 detailed。

**唯一例外（必须遵守）**：DSL 的 **negative clarifier 是英文否定句**，直译会变成裸否定 → **必须按正向化规则转写**（见 `negation-guard.md`），语义不变、说法正向。

| DSL 原句 | ✗ 直译（裸否定） | ✓ 正向化 |
|---|---|---|
| `The camera stays in place.` | 机位保持在原地不动 | 机位位置全程固定在同一点 |
| `The camera floats smoothly, with no shake.` | 机位平滑漂浮、没有晃动 | 机位平滑漂浮，画面全程平稳顺滑 |
| `The camera slides sideways and does not turn.` | 机位只横向侧移、不摆转 | 机位沿横向侧移，镜头朝向全程锁定在同一方向 |
| `The camera itself does not move.` | 机位本身不移动 | 机位位置全程固定在同一点，画面变化全部来自镜头变焦 |
| `The focal length stays the same.` | 焦距保持不变 | 焦距全程锁定在同一档位 |
| `{SUBJECT} stays in place and does not turn.` | 主体留在原地、不转身 | 主体保持这一站姿与朝向、留在原地 |

---

## 二、变量纪律：**不得出现该命令以外的镜头变量**

| 命令族 | 允许写的 | 禁止写的 |
|---|---|---|
| **摇镜 `/PAN` `/TILT`** | 摇向 + 摇幅 + 机位恒定 | **焦距**（它的三层里本就没有焦距——写了等于凭空多给模型一个可动旋钮） |
| **推拉 `/DOLLY`** | 推/拉 + 起止景别 + **焦距全程锁定**（这是它的 negative clarifier，**必须写**，用来声明"是机位在动、不是变焦"） | — |
| **变焦 `/ZOOM`** | 用"变焦"表达 + 机位位置恒定 | 「焦距数值」「焦段」当自变量 |
| **横移 `/TRUCK`** | 侧移方向 + 末帧落位 + 镜头朝向锁定 | 「视线始终锁定主体」（那是环绕的定义，会退化成环绕） |
| **升降 `/PEDESTAL`** | 升/降 + **镜头姿态全程保持水平** | 俯仰（写了就变成"升降 + 摇"） |
| **`/DUTCH`** | 静态倾角方向 | 当成运动写 |
| **`/ROLL`** | 翻滚方向 + 角度 | 省略方向（DSL 明说：不写方向由模型自选，不可复现） |
| **整圈 `/ORBIT:R:FULL`** | **圈数（"转满一整圈" + 收束句「末了回到正面」）＋ 起止时刻（`At` 起点 → `At` 结束点）——两者都要写** | 只给其一：只给圈数 → 转多圈冲过正面或中途停；只给时长 → 不转或转不完。**"快速" / `:FAST`**（2026-10-08 实测：速度词压过收尾句，镜头冲过正面继续转第二圈、第三圈，"整整一圈"被吃掉） |

**三条通用铁律**：
1. **不用裸否定**做约束（见 `negation-guard.md`）；
2. **主体静止**要靠一句**正向静态剧情**（"站在走廊中，单手微微抬起正在安静的思考，发丝与衣摆随微风轻摆"），不是"不移动不转身"——H3 对"静止"声明的响应本就不稳定（实测 A1 末帧主体姿态仍会变）；
3. **等级不可跨 mode 借用**：所有实测等级是 `mode + profile` 绑定的；本项目对应 `LOCAL_H3_REF2VA_PDD8_Q_416`，查询须显式给 `--h3-profile`（只给 `--h3-mode` 会报 no profile）。

---

## 三、查询姿势（三条）

```bash
PY=python; DSL=cinematic-director-camera-dsl
$PY scripts/camera_dsl.py render "<DSL命令>" --model minimax_h3          # 正句（写提示词用）
$PY scripts/camera_dsl.py render "<DSL命令>" --model minimax_h3 --layers  # 看分层，确认哪句属镜头层
$PY scripts/camera_dsl.py explain /PAN                                   # 命令定义与关键区分句
$PY scripts/camera_dsl.py render "<DSL命令>" --model minimax_h3 --h3-profile LOCAL_H3_REF2VA_PDD8_Q_416   # 查等级
```
注意：`--lang zh` **只翻 routing block（实测证据块）**，camera text 恒为英文 → 中文正句需按英文逐句直译。

---

## 四、起手景别的现实预期（Ref2VA）

- 官方矩阵：**"中景及更近"取景 0/51**；起手景别命中 6/48，且**可测偏差 40/40 全部是"偏宽"**；
- **三条"锁半身"手段全部无效**：crop-line 句 ❌、腰部以上裁切参考图 ❌（且破坏下身服装）、构图锚图 ❌；
- **唯一有效：删掉环境描写**（主体可放大 2.4–3 倍，但仍非腰部以上）；
- 实操结论：**起手构图靠文字只能锁到"大致一致"**——同镜跨条件的起手构图**不可逐帧比较**，比较重心放在"起手之后的演变"。

---

## 五、LoRA 选型

### 5.1 使用优先级（用户定稿）

> **专用 LoRA ＞ 不加 LoRA ＞ Jojocodex Camera-Motion（仅在特定情况）**

- **有对应运镜的专用 LoRA → 优先挂它**（本项目已验证有效者：Crash Zoom、360-Orbit）；
- **没有专用 LoRA → 不挂**（原生基线在"位移量"与"收尾仍在动"上普遍胜出）；
- **Jojocodex 默认不挂**，只在它被实测证明有增强的场景下挂（见 5.3）。

### 5.2 触发词台账（**先读 safetensors metadata，再查表**）

```python
import json, struct
with open(lora, 'rb') as fh:
    n = struct.unpack('<Q', fh.read(8))[0]
    meta = json.loads(fh.read(n).decode())["__metadata__"]
# 关注：modelspec.trigger_phrase / modelspec.usage_hint / ss_tag_frequency（内含训练 caption）
```

| LoRA | 触发词 | 适用 | 实测 |
|---|---|---|---|
| pablodawson 360-Orbit（`minimax_h3_flf2v_lora_v1`） | **无短词，须逐字用整段 caption**（存于 `ss_tag_frequency`） | 一切 360 类（环绕 / 翻滚 / 双人环绕） | ⭐ **唯一能闭环整圈的**（G4 0.950 / C3 0.927 / C6 0.855） |
| Crash Zoom | 无 | 急推 | ✅ "急"度（峰值/中位）6.44 ＞ 原生 5.12 ＞ Jojocodex 2.44 |
| Dutch Angle Slider | 无，**权重即倾角** | 荷兰角 | ✅ **权重 1 ≈ 20°（官方口径）**；**常用 strength = 2.0**。⚠️ 旧记"1.0 ≈ 1°、要 10–20"是**错的**（2026-10-09 修正） |
| Dolly Zoom | `dolly_zm` | 眩晕变焦 | 🟡 出力一般（该镜头本身是最不可靠镜头） |
| Locked Stabilization（MMH3-LOCKSTAB-MORE） | `lockstab` | 稳定器 / 跟拍 | 本轮首次启用 |
| Drone Shot（dr0nesh0t） | 无 | 航拍 / 拉升 | 本轮首次启用 |
| Headmounted（fpv_headbob） | 无 | POV / 机身绑定 | 本轮首次启用 |
| Jojocodex Camera-Motion | `camera motion`（**须置于提示词开头**） | 通用（默认不挂） | ❌ 无稳定增益，见 5.3 |

### 5.3 Jojocodex：为什么"默认不挂"

18 对全量量化（同镜同种子，① 原生 vs ② Jojocodex）：
**峰值位移 ① 胜 15/18**、**末段惯性 ① 胜 12/18**、逐帧平滑度 ② 略胜 10/18。

**它有增强的特定情况（记录备查）**：
- **横移类**（B3/B4）峰值位移 ② 更大；
- **平移 / 视角类**的帧间更平滑；
- 但**整圈 / 翻滚类无增益甚至更差**（C2 0.642 vs 0.647、C5 0.229 vs −0.085 参差）。

**四条已知可疑因素（未验证）**：① **触发词污染**——② 唯一文本差异就是开头一行泛词 `camera motion`；② 中文提示词 vs 英文训练 caption 的**语言错配**；③ **strength 从未扫描**（0.8 / 1.2 / 1.5）；④ 该版本 **AdaLN 适配器被剪掉 100 个**（AdaLN 正是控制全局调制、含镜头参数的地方）。

**若要判定它是否有效，必须补一个对照条件**：`trigger-only`——**不加 LoRA、但提示词开头同样加 `camera motion`**，用来分离"触发词效应"与"LoRA 效应"。

# 08 · LoRA 台账（运镜类）

> 校准版 v2 · 2026-10-09（Dutch Angle 权重已修正）

## 一、使用优先级（用户定稿）

> **专用 LoRA ＞ 不加 LoRA ＞ Jojocodex Camera-Motion（仅在特定情况）**

- 有对应运镜的专用 LoRA → **优先挂**（已验证有效：**Crash Zoom**、**360-Orbit**、**Dutch Angle Slider**）；
- 没有专用 LoRA → **不挂**（原生在"位移量"与"收尾仍在动"上普遍胜出）；
- **Jojocodex 默认不挂**，只在被实测证明有增强的场景下考虑。

## 二、运镜 LoRA 台账

| 用途 | LoRA 文件 | 触发词 | 权重 | 实测 |
|---|---|---|---|---|
| **360 类**（整圈环绕 / 翻滚 / 双人环绕 / 子弹时间） | `minimax_h3_flf2v_lora_v1`（pablodawson） | **无短词，须逐字用整段 caption**（存于 safetensors `__metadata__` 的 `ss_tag_frequency`） | 1.0 | ⭐ **唯一能闭环整圈的**：G4 0.950 / C3 0.927 / C6 0.855 / C2 里程碑 0.931 |
| **急推变焦** | `Crash_Zoom_MinimaxH3_v1` | 无 | 1.0 | ✅ "急"度（峰值 / 中位）**6.44 ＞ 原生 5.12 ＞ Jojocodex 2.44** |
| **荷兰角** | `H3_Dutch_Angle_Slider_v1` | 无，**权重即倾角** | **1 ≈ 20°；D4 定稿 2.0** | ✅ 挂了才有倾角（①②均无）。⚠️ 旧资料写"1.0≈1°、要 10–20"是**错的** |
| **眩晕变焦** | `dolly_zoom_minimaxh3_v1` | `dolly_zm` | 1.0 | 🟡 **在 Ref2VA 下没做出反变焦**（三版隔离试跑全部退化成普通推镜），与 DSL「最不可靠镜头」一致 |
| **稳定器 / 跟拍** | `MMH3-LOCKSTAB-MORE` | `lockstab` | 1.0 | 1080P 轮启用（F3 / F4） |
| **航拍 / 拉升** | `dr0nesh0t_aerial_flyover_000000500` | 无 | 1.0 | 1080P 轮启用（F6 / F7）；**用户裁定 F6/F7 推荐原生** |
| **POV / 机身绑定** | `fpv_headbob_h3_50` | 无 | 1.0 | 1080P 轮启用（E1 / E7） |
| **通用（默认不挂）** | `camera_motion_h3_lora_v1_3000_pruned`（Jojocodex） | `camera motion`（**须置于提示词开头**） | 1.0 | ❌ 无稳定增益，见 §四 |
| ~~整圈（已排除）~~ | `minimax_h3_orb360_step1500*`（MATLOWAI） | — | — | ❌ **需 3 个关键帧**（0°/120°/240°），会出现前后两张脸 |

**查触发词的通用方法**（不必上网）：
```python
import json, struct
with open(lora, 'rb') as fh:
    n = struct.unpack('<Q', fh.read(8))[0]
    meta = json.loads(fh.read(n).decode())["__metadata__"]
# 关注：modelspec.trigger_phrase / modelspec.usage_hint / ss_tag_frequency（内含训练 caption）
```

## 三、为什么 Jojocodex 默认不挂

18 对全量量化（同镜同种子，① 原生 vs ② Jojocodex）：
**峰值位移 ① 胜 15/18**、**末段惯性 ① 胜 12/18**、逐帧平滑度 ② 略胜 10/18。
→ 它只是"略平滑"，在"走多远"和"收尾仍在动"上**反而不如原生**。

**它有增强的特定情况（记录备查）**：横移类（B3/B4）峰值位移更大；平移 / 视角类的帧间更平滑；**整圈 / 翻滚类无增益甚至更差**。

**四条已知可疑因素（未验证）**：① **触发词污染**——② 与 ① 唯一的文本差异就是开头一行泛词 `camera motion`；② 中文提示词 vs 英文训练 caption 的**语言错配**；③ **strength 从未扫描**（0.8 / 1.2 / 1.5）；④ 该版本 **AdaLN 适配器被剪掉 100 个**（AdaLN 正是控制全局调制、含镜头参数的地方）。

**若要判定它到底有没有用，必须补一个对照条件**：`trigger-only`——**不加 LoRA、但提示词开头同样加 `camera motion`**，用来分离"触发词效应"与"LoRA 效应"。
（推论：**任何 ① vs ② 的比较都必须声明这个文本差异**，否则增益归因不成立。）

## 四、配套的效果 LoRA（非运镜，但与运镜批次共用）

| 代号 | 文件 | 语义 | 建议 |
|---|---|---|---|
| combat | `H3_Combat_V2.safetensors` | 通用格斗 | 0.9 |
| weapon | `Bunny_weapon_combatV1.safetensors` | 机械格斗状态（**与 combat 是两个 LoRA**） | 0.9，仅有武器搏斗时叠加 |
| motion | `Motion_Repair_V2.safetensors` | 运动修复 | **统一 V2**，0.7 |

有武器搏斗的推荐叠加顺序：`Combat_V2@0.9 → weapon@0.9 → Motion_Repair_V2@0.7`（一采 / 二采同构）。

# old · 已归档的本地版流水线（2026-09）

> **这些不是当前主线。** 主线是 H3 提示词，见仓库根目录 `h3-video-prompt-cn` / `h3-camera-motion-lib`。

---

## 这里面是什么

2026-09 做的一整套「本地版生成流水线」skill，共 10 个：

### 生成类（8 个）

`3d-animation-short-generator-local` · `brand-promo-video-generator-local` · `co-op-game-intro-generator-local` · `handdrawn-live-video-generator-local` · `minimalist-product-ad-generator-local` · `music-video-subtitle-generator-local` · `paper-collage-explainer-generator-local` · `papercraft-stop-motion-explainer-local`

### 被调用的提示词 skill（2 个）

- `image-prompt-writer-local` —— 生图步调用
- `ace-step-1.5-prompt-writer` —— 分离模式的 BGM 步调用

（视频步用 `h3-video-prompt-cn`，它属主线，所以没跟着归档。）

---

## 为什么归档

- **2026-09-03 之后没有再维护**：结论、写法、LoRA 台账都停在那个时间点；
- 主线转向 **H3 提示词本身**（`h3-video-prompt-cn`），这套流水线的「规划 → 分镜 → 逐步确认」方法论已被更细的 H3 判据覆盖；
- 留在根目录会让人误以为「11 个要一起装」，实际只想写 H3 提示词的人根本不需要它们。

---

## 如果还要用

**10 个必须一起装**，因为它们互相调用：8 个生成类在①生图步调 `image-prompt-writer-local`、②视频步调 `h3-video-prompt-cn`、③BGM 步调 `ace-step-1.5-prompt-writer`。缺任何一个都会在对应步骤断链。

安装方式：把这些文件夹**直接复制到** WorkBuddy 用户级 skills 目录（`~/.workbuddy/skills/`），**不要连 `old/` 这一层目录一起复制**——skill 必须平铺在 skills 目录下。

> 仓库根目录的 `README.md` 里，「设计理念：本地化改造总则 v2」「门控硬规则」「音频模式门」「典型使用流程」四节讲的都是这条流水线，可对照阅读。

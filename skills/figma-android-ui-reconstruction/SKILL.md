---
name: figma-android-ui-reconstruction
description: 使用 Figma MCP 将 Figma 设计稿还原或校正到本项目的 Android XML/View 界面。处理字体、颜色、尺寸、间距、形状或系统栏适配时使用。
---

# Figma Android UI 还原

视觉属性以 Figma 为准；应用架构、资源规范和交互行为以本仓库为准。只修改用户指定页面及直接相关的代码。

## 修改前先读取设计稿

1. 从 Figma URL 提取文件 key 和节点 ID。
2. 使用 Figma 元数据识别 Frame、子文本节点、尺寸位置和底部导航节点。
3. 读取目标节点及子节点的真实属性，不能根据节点名称猜测：
   - 文本节点名称可能与 `characters` 实际文本不同。
   - 读取可见填充、圆角、尺寸、字体、字号、字重、行高、字距、对齐方式及文本装饰。
   - 文本存在混合颜色或装饰时，读取分段文本样式。
4. 调用 `use_figma` 前，如有对应的 Figma-use 指引，先加载该指引。

## 将设计属性映射到本项目

- 字体使用 `app/src/main/res/font/` 下的本地资源。当前字体为 Poppins，`regular`、`medium`、`semi_bold`、`bold` 以及对应 italic 文件分别映射设计稿字重和斜体。
- 容器背景使用原始颜色资源，例如 `color_0b0b19`、`color_f3f4f4`。`text_title_btn_color` 这类语义文本颜色资源只能用于文本。
- 运行时背景和圆角使用 `ViewShapeEx.kt` 的扩展方法。若 Activity、Dialog 或 ViewHolder 已在代码中设置形状，运行时代码是最终效果的唯一基准；XML 只需设置接近设计的纯色背景以便预览，不要为了预览新增 drawable、控件层级或嵌套。
- 保留既有字符串资源和点击行为。只调整颜色、字体、大小和间距，不能替换本地化文本。

## 文本和形状样式检查

当用户要求按 Figma 调整“文本样式”时，默认逐项核对并按设计更新以下属性，不能只改字号或颜色：

- 字体文件、字重、斜体、字号、文字颜色、行高、对齐方式和 `includeFontPadding`。
- 文本所在按钮、标签、卡片或选项容器的背景色、渐变、描边、圆角、固定高度和必要内边距。
- XML 静态属性与 Kotlin 运行时设置；先搜索 Activity、Dialog、Adapter、ViewHolder 中是否会覆盖 XML 背景或文字属性。

实施时遵守以下边界：

- 只修改用户提供设计节点对应的控件，不顺带调整未提供设计的按钮、文案或其他控件。
- 用户仅要求样式时，保持原 XML 控件结构，不新增 UI 层级、包装容器或嵌套，也不替换现有 icon。
- Figma 文本存在混合字号、颜色或字重时，优先复用 `SpannableUtils.highLightString`；先确认它支持所需的颜色、加粗和字号组合，不足时再使用项目既有 span 方案。
- 渐变、不对称圆角或描边应在运行时代码中使用 `setGradientShape`、`setCustomCornersShape`、`setShapeColorRes` 等既有扩展精确实现；XML 设置可见的近似纯色即可。
- 公共 layout 或 Adapter 被多个页面复用时，先确认设计是否适用于全部场景。仅单一场景需要新样式时，使用场景专属 layout/Adapter，避免改变其他页面。

## 安全换算尺寸与间距

- 对齐设计节点的尺寸位置和相邻元素间距。Figma 因奇数像素导致左右边距略有不同时，Android 使用偶数且对称的边距。
- 推导底部间距前先处理系统栏。Figma 有 Bottom Bar 或导航栏时，只测量内容到该栏顶部的距离，不能把导航栏高度叠加到内容容器的底部 padding。
- 修改底部约束或 padding 前，检查 Activity 的窗口 inset 处理。本项目的沉浸式页面可能已经通过 `systemBars.bottom` 预留了系统栏空间。

## 登录页参考

针对 `LoginAc` 和 `ac_login.xml`，已验证以下可复用规则：

- 登录按钮高度为 48dp，圆角为 30dp。
- Google 按钮背景为 `#F3F4F4`；Quick Login 按钮背景为 `#0B0B19`。
- 按钮文字使用本地 Poppins SemiBold、16sp；Google 文字为黑色，Quick Login 文字为白色。
- 隐私条款正文使用本地 Poppins Regular、12sp、`#AFB3B6`；链接片段为 `#C233FF` 且带下划线。
- Figma Bottom Bar 的高度不能用于底部 padding；隐私条款文字到该栏区域的间距为 8dp。

## 验证

修改后编译两个 Debug 变体：

```sh
./gradlew :app:compileLocalDebugKotlin :app:compileGoogleDebugKotlin
```

## 维护此技能

发现新的 Figma 到 Android 还原经验时：

1. 只有可重复、且会改变后续实现决策的经验才加入。
2. 通用工作规则写入以上对应章节；页面专属事实写入具名的参考章节。
3. 写清触发场景、已验证证据和必须采取的动作；不要粘贴对话记录，也不要记录敏感信息。
4. 后续证据推翻既有规则时，更新或删除该规则；将技能与产出该经验的 UI 修改一并提交。

技能源码位于 `self-skills/skills/figma-android-ui-reconstruction/`，并通过 `~/.copilot/skills/` 下的链接安装。后续 Copilot CLI 会话会自动发现；当前会话编辑后执行 `/skills reload` 即可重新加载，无需重启 Copilot CLI。

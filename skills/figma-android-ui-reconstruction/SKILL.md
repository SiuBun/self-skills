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
5. 检查链接节点是否覆盖用户要求的完整范围：
   - 如果链接只指向图标、矢量、背景块或某个叶子文本节点，不能据此推断整个页面或组件样式。
   - 先通过元数据向上确认对应 Frame；无法取得完整 Frame 时，要求用户重新选中目标组件并提供链接。
   - 设计上下文中节点名称与截图、文本内容不一致时，以实际属性和截图为准，不按名称猜测用途。

## 先确定本次允许修改的属性

- 用户明确限定“只检查字体和大小”时，只核对 `fontFamily`、字重、斜体、`textSize` 以及会改变实际字重的 `textStyle`；不要顺带修改颜色、行高、容器或间距。
- 用户笼统要求“文本样式”时，按本技能的完整文本检查范围处理，包括字体、字号、颜色、行高、对齐和 `includeFontPadding`，并检查文本所在容器。
- 用户补充背景色、圆角、描边或间距后，再扩大到对应容器属性。后续更窄的要求覆盖前面的默认范围。
- 修改前先列出目标控件与允许调整的属性，避免因为设计稿还展示了其他视觉信息而扩大任务。

## 将设计属性映射到本项目

- 字体使用 `app/src/main/res/font/` 下的本地资源。当前字体为 Poppins，`regular`、`medium`、`semi_bold`、`bold` 以及对应 italic 文件分别映射设计稿字重和斜体。
- 本地字体文件已经包含设计字重时，不再叠加 `android:textStyle="bold"`。例如 Poppins SemiBold 应只使用 `@font/semi_bold`；额外 `bold` 会产生比设计更重的合成字重。
- 容器背景使用原始颜色资源，例如 `color_0b0b19`、`color_f3f4f4`。`text_title_btn_color` 这类语义文本颜色资源只能用于文本。
- 运行时背景和圆角使用 `ViewShapeEx.kt` 的扩展方法。若 Activity、Dialog 或 ViewHolder 已在代码中设置形状，运行时代码是最终效果的唯一基准；XML 只需设置接近设计的纯色背景以便预览，不要为了预览新增 drawable、控件层级或嵌套。
- 保留既有字符串资源和点击行为。只调整颜色、字体、大小和间距，不能替换本地化文本。

## 文本和形状样式检查

当用户要求按 Figma 调整“文本样式”时，默认逐项核对并按设计更新以下属性，不能只改字号或颜色：

- 字体文件、字重、斜体、字号、文字颜色、行高、对齐方式和 `includeFontPadding`。
- 文本所在按钮、标签、卡片或选项容器的背景色、渐变、描边、圆角、固定高度和必要内边距。
- XML 静态属性与 Kotlin 运行时设置；先搜索 Activity、Dialog、Adapter、ViewHolder 中是否会覆盖 XML 背景或文字属性。
- 同一 TextView 使用本地字重字体时，检查并删除冲突的 `textStyle`；不能只看 XML 中的字体文件名就判定字重正确。
- Figma 的行高换算到 Android 时优先使用 `android:lineHeight`，同时设置 `includeFontPadding="false"`；固定高度按钮还要确认文字能在设计行高下垂直居中。

实施时遵守以下边界：

- 只修改用户提供设计节点对应的控件，不顺带调整未提供设计的按钮、文案或其他控件。
- 用户仅要求样式时，保持原 XML 控件结构，不新增 UI 层级、包装容器或嵌套，也不替换现有 icon。
- Figma 文本存在混合字号、颜色或字重时，优先复用 `SpannableUtils.highLightString`；先确认它支持所需的颜色、加粗和字号组合，不足时再使用项目既有 span 方案。
- 渐变、不对称圆角或描边应在运行时代码中使用 `setGradientShape`、`setCustomCornersShape`、`setShapeColorRes` 等既有扩展精确实现；XML 设置可见的近似纯色即可。
- 公共 layout 或 Adapter 被多个页面复用时，先确认设计是否适用于全部场景。仅单一场景需要新样式时，使用场景专属 layout/Adapter，避免改变其他页面。
- 已有自定义 View 提供背景色、圆角、文字颜色等 setter 时，单页面差异优先在调用处组合这些 setter；不要仅为一个页面扩展新的内容模式或改变公共默认样式。

## 容器背景、圆角和描边

- 文本样式与容器样式分开核对：先确认文字，再确认承载它的按钮、标签、卡片或弹窗根容器。
- Figma 中 `rounded-[22px]` 对应 Android 的 22dp 圆角，而不是高度的一半自动推断；描边宽度和颜色也必须单独读取。
- 实底按钮使用 `setShapeColorRes`；白底描边按钮同时传入背景色、`strokeWidth` 和 `strokeColorRes`，不能用半透明背景近似描边。
- XML 背景用于预览，Kotlin 运行时 shape 是最终效果。两者应使用相同的基础颜色，避免预览与运行时完全相反。
- 调用顺序要保证最终样式不被覆盖：先执行会按性别、状态或选中态重置背景的方法，再调用页面专属的背景色和圆角 setter。
- Dialog 或 Popup 根容器需要圆角时，Window、decorView 和 View parent 背景必须透明，否则外层矩形背景会盖住圆角区域。

## 安全换算尺寸与间距

- 对齐设计节点的尺寸位置和相邻元素间距。Figma 因奇数像素导致左右边距略有不同时，Android 使用偶数且对称的边距。
- 使用相邻节点坐标推导间距，而不是凭截图估计。例如下一行 `y - 上一行底部` 才是行间距；容器顶部到首行顶部才是顶部内边距。
- 区分控件固定高度、行间距、容器 padding 和系统栏 Insets，不能把它们合并成一个写死高度。
- 三列或多列网格先确认单项高度、列数和 ItemDecoration 规则。若浮动 Window 必须提前知道高度，可按 `行数 × item 高度 + (行数 - 1) × 行间距 + 容器上下区域` 计算；不要强制测量全部 RecyclerView item。
- 只有设计明确固定面板高度时才同时固定根布局与 Window 高度；否则让容器包裹内容，并只给依赖数据量的列表区域计算高度。
- 推导底部间距前先处理系统栏。Figma 有 Bottom Bar 或导航栏时，只测量内容到该栏顶部的距离，不能把导航栏高度叠加到内容容器的底部 padding。
- 修改底部约束或 padding 前，检查 Activity 的窗口 inset 处理。本项目的沉浸式页面可能已经通过 `systemBars.bottom` 预留了系统栏空间。

## 顶部弹窗和系统栏

- 顶部全宽选项面板优先使用 `DialogFragment`，而不是依附锚点的 `PopupWindow`；`PopupWindow` 更适合按钮附近的小浮层。
- 顶部 Dialog 使用 `Gravity.TOP`，Window/decor 背景透明，根容器只设置底部圆角。不要修改通用 `BaseDialogFragment` 的居中默认行为，页面自行覆写 Window 配置。
- 使用 `WindowCompat.enableEdgeToEdge(window)` 配合 `ViewCompat.setOnApplyWindowInsetsListener`。前者负责 Window 延伸到系统栏，后者只负责把状态栏高度转成根布局顶部 padding。
- 不要直接设置已弃用的 `statusBarColor`；状态栏图标明暗使用 `WindowInsetsControllerCompat`。
- 动态 Insets 不属于 Figma 内容间距。计算面板内容高度时先计算设计内容，再单独加 `statusBars.top`。
- 已知 item 数量和固定网格规格时，在 `onStart()` 一次性设置 Window 高度；不要在进入/退出过程中多次 `post`、`measure`、`setLayout`，否则动画或关闭时容易抖动。
- 不需要动画时显式关闭 Window 动画，避免复用 Activity 动画资源。需要动画时为 Dialog 使用独立 animation style，不与 Activity 的 open/close 动画混用。

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

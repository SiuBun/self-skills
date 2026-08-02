# Compose 设计系统、自适应与无障碍

## 一、设计系统闭环

商业 Compose 设计系统应逐步包含：

1. 原始 Token：品牌色、字号、字重、圆角、间距和高程。
2. 语义 Token：`primary`、`surface`、`onSurface`、`error` 等角色。
3. Material 3 映射：ColorScheme、Typography、Shapes。
4. 品牌组件：按钮、输入框、卡片、导航和弹窗。
5. 多主题：Light、Dark，必要时 Black、Dynamic Color 或品牌变体。
6. Preview/Gallery：集中浏览组件和状态。
7. Screenshot regression：防止视觉回退。

Theme 不应只设置 `primary/secondary` 后让页面继续散落硬编码。

## 二、渐进迁移中的双主题

View 与 Compose 共存时，XML Theme 和 Compose Theme 暂时是两个事实来源。迁移期间：

- 以现有 XML 主题、颜色、字体、shape 和 dimens 为视觉事实来源。
- 将原始颜色转换为语义角色，不以 hex 值命名 Compose Token。
- Light/Dark 必须同时映射，并跟随系统或产品主题设置。
- 不在迁移过程中自行发明新颜色和尺寸。
- 公共品牌组件建立后，新页面不再直接复制旧页面硬编码。

完全迁移后再评估移除 XML Theme，不在中期提前破坏旧页面。

## 三、组件 API

- `Modifier` 作为第一个可选参数。
- 状态和事件参数在前，样式参数在后。
- 使用 `content`、`icon`、`leadingContent`、`trailingContent` 等 slot 提供组合能力。
- 对明确产品语义建立组件变体，如 `PrimaryButton`、`DangerButton`，不要让每个调用点重复颜色、shape 和 padding。
- 不让公共组件直接读取 ViewModel。
- 避免暴露过多底层样式参数，导致每个调用点都能破坏设计规范。

## 四、Preview

公共组件和页面 Screen 至少覆盖：

- 正常内容。
- Loading、Empty、Error。
- Light/Dark。
- 长文本和本地化。
- 较大字体。
- Compact 和 Expanded 尺寸。

Preview 必须使用真实 App Theme。依赖网络、数据库或 ViewModel 的 Route 不作为主要 Preview 对象。

## 五、自适应布局

自适应不是为手机、平板和折叠屏复制三套页面，而是让布局由窗口能力驱动：

- Compact：单栏和 Bottom Navigation。
- Medium：根据内容选择 Rail 或更宽单栏。
- Expanded：Navigation Rail/Drawer、列表-详情或多栏。
- 折叠屏：结合 FoldingFeature 判断分隔和姿态。

列表-详情优先使用适配窗口的双栏结构。避免只读取物理屏幕尺寸；使用窗口尺寸和 adaptive API。

系统栏、底部导航和 IME 必须明确各自负责的 inset，避免重复 padding。

## 六、无障碍

- 可点击图标必须提供本地化 `contentDescription`；装饰图片才传 `null`。
- 自定义点击容器设置合适 `Role`。
- 组合信息使用 `mergeDescendants`，标题按需设置 heading。
- 不只用颜色表达错误、选中、在线或涨跌。
- 触控目标通常至少 48dp；视觉尺寸较小时扩大可点击区域。
- 支持字体缩放，避免固定高度裁剪文本。
- 文本设置合理换行、maxLines 和 overflow，但不能掩盖关键信息。
- TalkBack 顺序与视觉顺序一致。
- TV 或键盘场景明确焦点顺序和 FocusRequester 生命周期。

## 七、资源与本地化

- 用户可见文本使用字符串资源。
- Locale 相关格式化读取 Compose 可观察 Locale，避免在 Composition 中直接调用不可观察的 `Locale.getDefault()`。
- 日期、数字、金额和复数使用对应本地化 API。
- 图片 contentDescription 和语义文案同样需要本地化。

## 八、复杂视觉

- 自定义绘制优先使用 `drawWithCache` 缓存不随每帧变化的对象。
- 动画只观察必要状态，避免让高频值驱动整棵页面重组。
- Shared transition、Blur、Gradient 和无限动画必须在真机 Release 构建验证。
- 自定义 Layout 只有标准布局无法表达需求时才使用，并覆盖字体缩放和极端尺寸测试。

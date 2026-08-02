---
name: compose-development-guidelines
description: 设计、实现、审查或渐进迁移商业 Android Jetpack Compose 功能时使用。覆盖架构与状态、导航与副作用、设计系统、自适应与无障碍、View/Compose 互操作、性能、测试和迁移完成标准。
---

# Jetpack Compose 商业化开发规范

目标是交付状态可预测、生命周期正确、易测试、可扩展且具备性能保障的 Compose 功能。规范综合 Google Compose 推荐实践，以及对 Now in Android、Bitwarden Android、Element X Android、Pocket Casts Android 和 Compose Samples 的工程走读结论。

不要照搬某个示例项目。先判断当前工程规模、迁移阶段、既有架构和产品约束，再选择足够而不过度的实现。

## 使用方式

开始任务前先判断任务类型，并读取对应参考文件：

| 任务 | 必读参考 |
| --- | --- |
| 新建或重构 Compose 页面 | [架构、状态与导航](references/architecture-state-navigation.md) |
| 建立主题、组件或适配多设备 | [设计系统、自适应与无障碍](references/design-system-adaptive-accessibility.md) |
| 审查卡顿、重组、列表或图片性能 | [性能、测试与发布质量](references/performance-testing-quality.md) |
| XML/View 渐进迁移 Compose | [渐进迁移与互操作](references/incremental-migration-interop.md) |

涉及多个方面时读取全部相关文件。审查任务只报告有代码证据的问题；性能结论要区分静态确定问题和必须通过 Profiler、Macrobenchmark 或真机验证的风险。

## 实现前决策

1. 搜索现有 Theme、Navigation、Screen、ViewModel、UiState、公共组件、Preview 和测试，优先沿用已经稳定的模式。
2. 明确页面是 Compose-only，还是处于 Activity、Fragment、Dialog、RecyclerView 或传统 View 的混合边界。
3. 画出状态链：数据源 -> Repository/Manager -> ViewModel/Presenter -> UiState -> Route -> Screen。
4. 区分持久状态、用户意图和一次性副作用，明确旋转、后台、进程重建和返回栈恢复行为。
5. 识别 Loading、Empty、Content、Error、Offline、Permission、Login、Locked、Paywall 等真实业务状态。
6. 明确 Compact、Medium、Expanded、折叠屏、字体缩放、系统栏和 IME 的适配范围。
7. 选择最小验证集：Preview、单元测试、Compose UI 测试、截图测试或 Macrobenchmark。

## 核心架构原则

- Route 连接 ViewModel、收集状态和处理导航；Screen 接收不可变状态与回调并负责渲染。
- 公共 Composable 不直接获取业务 ViewModel，不直接访问 Repository、数据库、网络、支付或会话容器。
- 页面保持单一状态源。多条业务流在 ViewModel/Presenter 中使用 `combine`、`flatMapLatest`、`stateIn` 等组合，不用 `LaunchedEffect` 在多个 ViewModel 之间复制同一状态。
- 持久可渲染信息放入 State；用户输入和生命周期输入作为 Action/Intent；导航、Snackbar、分享等一次性行为使用 Event/Effect。
- 小页面不强制套完整 MVI；复杂页面必须有明确事件入口和合法状态迁移。
- 外部 SDK、JNI、FFI 或平台 View 通过 Adapter/Facade 转换为项目内部模型，不让第三方类型扩散到 UI。
- App 模块负责组装。只有团队规模、编译隔离或依赖边界确有收益时才拆 feature `api/impl`，不要机械复制大型样例的模块数量。

## Compose 编码原则

- State 使用不可变模型；避免多个布尔值和可空字段拼出非法组合。
- 使用 `collectAsStateWithLifecycle()` 收集供 UI 渲染的 Flow。
- 业务协程放在 `viewModelScope`、所属生命周期 Scope 或明确注入的长生命周期 Scope；不得在 Composable 中临时创建 `MainScope()`。
- `LaunchedEffect`、`DisposableEffect` 和 `remember` 的 key 必须对应输入变化语义；Effect 必须有清晰的启动、重启和取消条件。
- 仅纯 UI 且可丢失的状态使用 `remember`；需要跨配置变化恢复的输入、页签、展开状态或选择状态使用 `rememberSaveable` 或 ViewModel/SavedStateHandle。
- Composable 参数优先为状态、事件回调、slot 和 `Modifier`；`Modifier` 是第一个可选参数。
- 公共组件使用 slot 和语义化变体，不暴露无边界的样式参数集合。
- Preview 使用真实 Theme，并覆盖关键状态，而不只预览成功态。

## 设计、性能与质量底线

- Theme 至少统一 Color、Typography 和 Shape；品牌项目进一步建立 Spacing、Elevation、Gradient 等语义 Token。
- 页面不新增散落的品牌色、字号和圆角硬编码；迁移阶段以现有 XML 资源为事实来源，避免双源漂移。
- 可点击图标提供本地化 `contentDescription` 或等价 semantics；装饰图片才使用 `null`。
- Lazy 列表使用稳定且唯一的业务 key；不得把 index 或对象默认 `hashCode()` 拼入正常业务 key。
- 高频滚动状态通过 `derivedStateOf` 或 `snapshotFlow` 隔离，避免直接驱动大范围 Composition。
- Composition 中不执行阻塞 I/O、高成本格式化或重复集合转换；必要时移入 ViewModel、`remember` 或后台线程。
- 图片请求按显示尺寸采样，列表中避免无界原图解码；播放器和 Android View 必须对称释放。
- 核心页面至少具备状态单元测试、Preview 和 Compose UI 测试；视觉密集页面增加截图测试；启动和滚动关键路径增加 Macrobenchmark 与 Baseline Profile。

## 渐进迁移原则

- 允许 View 与 Compose 长期共存，不为追求“纯 Compose”一次性重写稳定的播放器、RTC、WebView 或其他高风险平台能力。
- 新功能优先 Compose；旧页面按边界清晰、依赖较少、视觉可验证的顺序迁移。
- Fragment 中的 `ComposeView` 使用与 View 生命周期匹配的 `ViewCompositionStrategy`；RecyclerView 中遵循池化容器默认释放语义。
- Compose 中暂时承载 View 时使用 `AndroidView`，明确创建、更新、释放、主题和状态恢复边界。
- 每个页面迁移完成前验证视觉、状态恢复、返回栈、系统栏、IME、加载/空态/错误态、无障碍和性能。
- 只有确认 XML、Adapter、Activity/Fragment、资源和旧测试不再被引用后才删除。

## 禁止模式

- Composable 直接发网络请求、操作数据库、支付或会话。
- 在 Composable 中创建无取消路径的 CoroutineScope、播放器、监听器或 SDK 实例。
- 使用 Effect 在两个状态持有者间长期同步同一份页面状态。
- 用 `isLoading + data? + error?` 表达会产生非法组合的复杂页面。
- 将导航命令长期放在可重复消费的 UiState。
- Lazy key 使用 index、随机值或对象默认 `hashCode()`。
- 默认使用 destructive database migration。
- 只根据 Debug 顺滑度宣称没有性能问题。
- 迁移时自行发明颜色、尺寸和交互，破坏原页面视觉或行为。

## 完成前检查

1. 是否只有一个页面级状态源，状态能否表达所有业务分支。
2. Route、Screen、ViewModel、Repository 和平台能力边界是否清晰。
3. Flow、协程、监听器、播放器和 View 是否随正确生命周期停止或释放。
4. 导航和一次性事件是否会因重组、重订阅或进程恢复而重复。
5. Theme、Token、系统深色模式和 XML 迁移期主题是否一致。
6. Lazy key、滚动读取、图片尺寸和高成本计算是否安全。
7. 是否验证字体缩放、TalkBack、横竖屏/窗口尺寸、系统栏和 IME。
8. 是否覆盖 Loading、Empty、Content、Error、Offline、Permission 和付费/登录等适用状态。
9. 是否运行已有的编译、Lint 和最小相关测试。
10. 性能敏感改动是否通过 Release 构建、真机或基准验证，而非只靠静态判断。

## 经验来源与维护

- Google 官方规范是 API、生命周期、状态和性能原则的优先依据。
- Now in Android 提供模块化、离线优先、设计系统、截图测试和性能基线经验。
- Bitwarden 提供 State/Action/Event、业务错误和复杂会话经验。
- Element X 提供大型模块边界、设计 Token、截图与发布流水线经验。
- Pocket Casts 提供 View/Compose 渐进迁移和平台能力共存经验。
- Compose Samples 只用于提炼 UI、自适应和交互技法，不作为完整商业架构模板。

新增规则必须满足至少一项：

1. 有 Google 官方文档支持；
2. 在多个成熟项目中重复验证；
3. 经过项目基准、测试或线上问题验证；
4. 是用户明确确认的长期设计决策。

项目类名、业务接口、目录和临时兼容方案放入项目级 Skill，不写入本 Skill。后续证据推翻规则时及时更新或删除。

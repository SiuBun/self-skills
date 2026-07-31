---
name: siubun
description: 按 SiuBun 的 Android/Kotlin 个人代码与设计风格实现或重构功能时使用。适用于架构分层、状态管理、生命周期、会话依赖、网络错误、列表分页、XML UI、弹窗、并发资源管理和命名设计。
---

# SiuBun Android 代码与设计风格

目标是写出边界清晰、状态可观察、生命周期正确、容易沿现有模式扩展的代码。先理解问题所属层级和生命周期，再选择实现方式；不要从页面效果反推一套临时架构。

## 开始实现前

1. 搜索同职责的 Activity、Fragment、ViewModel、Repository、Adapter、Manager 和布局，优先沿用成熟模式。
2. 画出最短调用链：用户操作 -> UI -> ViewModel -> Repository/Manager -> API、SDK 或本地存储。
3. 明确数据是持续状态还是一次性事件，是应用级、登录会话级、页面级还是 View 级。
4. 明确失败、取消、重试、并发调用、登出和页面销毁时的行为。
5. 只有现有抽象无法自然承载职责时才新增公共层；API 保持最小，不为“可能复用”提前设计框架。

## 架构取向

- 使用职责分层，而不是为了形式强制套层：
  - UI 负责渲染、收集状态、导航和转发用户操作。
  - ViewModel 负责页面状态、事件和业务流程编排。
  - Repository 负责 API、本地数据和缓存的适配或组合。
  - Manager/Service 负责有状态、长生命周期、跨页面或 SDK 驱动的业务。
  - API、DAO、SDK Adapter 只处理具体协议和能力。
- Repository 可以保持薄，ViewModel 可直接依赖 Repository；没有明确跨页面业务价值时不默认增加 UseCase。
- 第三方 SDK 不直接渗透 UI 或 ViewModel。先定义项目语义接口或 Facade，再由 Adapter 包装 SDK。
- 复杂业务优先建模为状态机、事件流和明确操作入口，不使用散落布尔值拼接隐式状态。
- 对外只暴露完成业务所需的最小能力；可变状态、SDK 实例、缓存容器和协程 Job 保持私有。

## 依赖与生命周期

- 先按生命周期划分依赖：
  - 应用级：进程内长期存在、与账号无关。
  - 会话级：依赖账号、token、用户数据或登录后 SDK 连接。
  - 页面级：只服务一个页面或导航范围。
  - View 级：必须随 `onDestroyView()` 释放。
- 会话级 Repository、Manager、网络客户端、SDK 连接和收集任务必须由会话容器统一创建和销毁，不能放入不可清理的全局单例。
- 登录、恢复、刷新 token、登出和删除账号由唯一会话管理入口串行处理；功能代码不得自行写 token、伪造登录态或局部清理会话。
- 使用 `Mutex`、幂等键或请求集合保护不能并发执行的业务操作，如登录切换、接听/拒绝/挂断、订单验证。
- 每个长生命周期对象必须有对称的初始化和释放路径。释放应包括取消 Job/scope、注销监听、断开 SDK、清空引用和关闭客户端。
- 登出与删除账号的清理语义应分开：普通用户数据可在登出清理；明确需要跨登录保留的会话偏好只在删除账号时清理。

## 状态、事件与 MVI

- 持续可渲染的数据使用 `StateFlow`；一次性导航、弹窗、提示和命令使用 `SharedFlow` 或明确的 `UiEvent`。
- 可变流保持私有，对外暴露只读流：

```kotlin
private val _uiState = MutableStateFlow(UiState())
val uiState = _uiState.asStateFlow()

private val _uiEvent = MutableSharedFlow<UiEvent>()
val uiEvent = _uiEvent.asSharedFlow()
```

- 状态使用不可变 `data class`，通过 `copy`、`update` 演进；不要让 UI 修改 ViewModel 内部集合或状态对象。
- 加载、空数据、错误、重试和成功必须有可观察结果，不能静默失败或用成功形状的默认值掩盖错误。
- 多条业务流共同决定界面时，在 ViewModel 使用 `combine`、`flatMapLatest`、`stateIn` 等组合，不在 Fragment 内维护手工同步变量。
- 只有真正的一次性动作才建事件；可从当前状态推导的内容不要重复发事件。
- ViewModel 使用 `viewModelScope`，依赖通过现有 Factory 或 DI 方式显式传入；未知 ViewModel 类型明确抛错。

## 协程、错误与取消

- UI 中的 Flow 收集必须绑定 Lifecycle，并使用 `repeatOnLifecycle` 或项目已有等价扩展；Fragment 优先使用 `viewLifecycleOwner`。
- 业务协程使用所属对象的 scope：应用任务用 application scope，会话任务用 session scope，页面任务用 `viewModelScope` 或 lifecycle scope。
- `CancellationException` 必须继续传播，不能当普通错误吞掉。
- 统一异常入口负责通用网络、解析和认证错误；局部 `onError` 只处理该场景独有的状态恢复，再决定是否继续交给全局处理。
- 不添加宽泛 `catch` 后返回空列表、`null` 或成功状态。错误要么转换为明确领域错误，要么向上抛出并更新可观察状态。
- token 失效属于会话事件：网络或 SDK 层识别并映射为统一异常，最终由会话管理器登出，而不是每个页面各自处理。

## API、Repository 与缓存

- API/DAO 接口只描述协议；Repository 负责将协议数据转换为业务可用结果。
- 多数据源读取按明确优先级组织，例如 memory -> Room/DataStore -> network；命中缓存立即返回，网络结果负责回填缓存。
- 缓存必须说明键、有效期、更新时机和清理生命周期。不要把“最近一次结果”误当成永久真相。
- 认证请求使用会话级客户端；公共请求和认证请求分开装配，避免 token 或用户 Header 泄漏到无关接口。
- 拦截器各自只负责一种 Header 或错误映射，保持可组合；业务错误解析不要散落在 Repository。
- flavor 或渠道差异集中在策略、配置或单一分支入口，业务调用方使用相同接口。

## Facade、Adapter、Strategy 与状态机

- 对 IM、RTC、支付等复杂 SDK 使用 Facade 统一业务入口，UI 不直接调用底层 SDK。
- Adapter 将第三方回调和模型转换成项目事件、状态和接口，避免 SDK 类型扩散。
- 当渠道或实现可替换时使用 Strategy；策略负责具体能力，订单、状态和业务规则仍由上层编排。
- 状态机必须定义合法状态和事件，由单一 Manager 负责流转。UI 只观察状态并发出意图，不能直接改内部状态。
- 高频事件或通知使用队列、节流、去重或合并策略，避免每个原始事件直接触发 UI。

## Activity、Fragment 与 ViewBinding

- 使用 ViewBinding，不使用 `findViewById` 拼装页面。
- Activity/Fragment 保持薄：绑定 View、设置 Adapter、收集状态、导航、权限与系统 UI；请求和状态计算下沉。
- Activity 使用统一基类承载 ViewBinding、Edge-to-Edge 和系统栏策略。
- Fragment 的 binding 生命周期必须与 View 生命周期一致。若基类使用 `lateinit binding`，不得在 `onDestroyView()` 后持有 View、Job、回调或 Adapter 引用；新增基类时优先采用可空 backing field 并清空。
- 点击复用项目已有防抖扩展；不要重复实现计时器。
- 导航参数优先使用 Intent/arguments 的稳定键和可序列化数据；短生命周期回调只在无法序列化且生命周期明确时使用。

## Dialog、Bottom Sheet 与键盘

- 居中弹窗使用统一基类处理透明背景、宽度和默认取消策略；产品明确要求前不要改变关闭行为。
- Dialog 的 binding 使用可空 backing field，并在 `onDestroyView()` 清空；同时取消与 View 绑定的 Job、动画和回调。
- Bottom Sheet 统一处理透明父背景、顶部圆角、固定 TAG 和展示入口。
- 参数优先放 `arguments`；回调属性只用于不可序列化的短生命周期交互。
- 需要避让键盘时使用 `WindowInsetsCompat.Type.ime()`，更新根容器 margin/padding；不要根据屏幕高度猜测键盘。
- 显示/隐藏键盘复用统一扩展，保证焦点与 IME 状态同步。

## RecyclerView、DiffUtil 与 Paging

- 普通异步列表使用 `ListAdapter`；分页列表使用 Paging 的 `PagingDataAdapter` 和 `PagingSource`。
- Adapter 保持薄，只负责 inflate、bind、ViewType、点击和必要的 payload 局部刷新。
- `areItemsTheSame` 使用稳定业务 ID，`areContentsTheSame` 比较完整可渲染数据。
- 点击时读取 `bindingAdapterPosition`，检查 `NO_POSITION`，再从当前列表获取 item，避免异步更新后的旧引用。
- 只有局部刷新收益明确时才实现 payload；payload bind 只更新受影响控件。
- PagingSource 负责页码、请求和 `LoadResult`，不处理 UI 文案、弹窗或 View 状态。
- 页面通过 `loadStateFlow.refresh` 驱动首屏加载、刷新、空态和错误；下拉刷新和错误重试统一调用 `adapter.refresh()`。
- 不混用 Paging 3 与手写 footer/loadMore 状态。维护旧页面时先确认迁移范围；新页面只选一套实现。
- 卡片尺寸依赖 RecyclerView 最终宽度时，在布局完成后按比例计算，不提前假设列宽。

## XML、资源与视觉实现

- 优先使用 ConstraintLayout 表达明确相对关系，完整建立约束；受相邻元素限制的宽度使用 `0dp`。
- 文本设置合理的 `maxLines`、`ellipsize` 和 constrained width，避免长文案破坏布局。
- 用户可见文本、颜色、字体、drawable 和复用尺寸使用资源；不在 Kotlin/XML 中散落硬编码。
- `tools:` 只用于预览，不改变运行时行为。
- 运行时 shape 复用统一扩展；XML 初始背景与运行时背景保持视觉一致。
- Edge-to-Edge 页面先定义系统栏、IME、底部导航各自负责的 inset，避免重复叠加 padding。
- 设计还原先确认最终可用内容区域，再处理固定尺寸和比例；不要把系统栏或 Bottom Bar 高度重复算入设计内容。

## 命名与代码表达

- 名称表达角色和生命周期：`AppContainer`、`UserSessionContainer`、`SessionManager`、`Repository`、`PagingSource`、`UiState`、`UiEvent`。
- 接口使用能力名，具体实现追加来源或技术名，如 `RtcEngine` / `AgoraRtcEngineImpl`。
- Flow 名称表达内容而不是实现细节；当前状态用 `...State`，事件用 `...Event`，集合流用业务复数含义。
- 布尔值使用 `is/has/can/should`；动作方法使用动词开头，如 `fetch`、`refresh`、`observe`、`release`、`verify`。
- 多行参数保留尾随逗号；优先不可变值和早返回；只给复杂状态流转、协议约束或生命周期陷阱写注释。
- 不把旧实现中的缩写、临时代码和注释块复制到新功能。先理解职责，再使用一致、可搜索的名称。

## 完成前检查

1. 调用链是否符合层级职责，是否绕过已有 Manager、Repository 或会话入口。
2. 状态、事件、错误、空态、取消和重试是否完整。
3. scope、监听器、SDK、binding、Adapter 和 Job 是否有对称释放。
4. 并发操作是否需要串行、幂等或去重。
5. 新实现是否复用了现有资源、扩展、Factory、缓存和列表模式。
6. 是否同时覆盖项目已有的 flavor/build variant。
7. 运行最小范围的现有编译、测试或 lint；没有测试时至少编译受影响变体。

## 维护原则

- 只记录可跨项目复用的个人设计倾向和编码规则。
- 项目目录、具体类名、业务调用链、接口字段和历史兼容关系放入对应项目 skill。
- 新规则必须来自多处验证或明确设计决策；若后续实践推翻规则，及时更新或删除。

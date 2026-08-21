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

### 配置变化与重建

- 把 Activity/Fragment 重建视为 View 容器替换，不是业务流程重新开始。重建时可以重新绑定 View、Adapter、Surface 和收集器，不能重新提交登录、支付、匹配、通话或消息等副作用。
- 长时间运行的业务放入 ViewModel、会话 Manager 或应用级对象；View 销毁只释放 View 资源，不能顺带结束更长生命周期的业务。
- 需要跨重建继续的 UI 结果必须是可观察状态；无回放事件只适合允许在重建空窗丢弃的视觉反馈。
- 区分配置重建与进程死亡。内存状态只承诺配置重建连续性；需要支持进程恢复时，必须显式选择 `SavedStateHandle`、持久化存储或可重放的领域状态。
- 配置重建验证不仅检查崩溃，还要检查重复请求、重复导航、重复权限、重复支付、重复信令、倒计时重置和旧 View 引用。
- 页面初始化必须拆分“可重复绑定”和“只执行一次的业务”。`setupViews()` 可以重复设置 View、监听器和状态收集，但不能默认包含网络提交、匹配、支付或通话启动。
- 配置变化期间旧页面可能先 `onStop()`、新页面稍后 `onStart()`；应用前后台状态使用 `ProcessLifecycleOwner` 等进程级来源，不以单个 Activity 生命周期直接推断。
- 倒计时和媒体续播保存业务时间基准或单调时钟快照，不保存旧页面 ticker；新页面根据当前时间重新计算显示值。

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
- RTC、播放器等业务状态与 View Surface 分离：Manager/VM 持有会话和播放语义，Activity/Fragment 只在新 View 上重新绑定 Surface。
- 媒体需要跨重建续播时，保存播放位置、播放意图和单调时钟快照；恢复时按经过时间补偿，并对总时长或循环规则做边界处理。
- 跨页面状态机的导航事件必须有唯一消费者。页面可以观察相同状态用于渲染和关闭自身，但不能各自重复执行同一个业务跳转。
- Service 是长生命周期能力载体，不是第二套业务状态机。它观察 Manager 状态、提供系统要求的存活能力和通知展示，不反向推导或创建来电、去电、匹配等业务页面。
- 状态机中的过渡状态不能被 UI 擅自解释为终止。例如业务为了切换流程短暂回到空闲态时，页面不能仅凭该状态立即销毁，除非状态协议明确如此。

## Activity、Fragment 与 ViewBinding

- 使用 ViewBinding，不使用 `findViewById` 拼装页面。
- Activity/Fragment 保持薄：绑定 View、设置 Adapter、收集状态、导航、权限与系统 UI；请求和状态计算下沉。
- Activity 使用统一基类承载 ViewBinding、Edge-to-Edge 和系统栏策略。
- Fragment 的 binding 生命周期必须与 View 生命周期一致。若基类使用 `lateinit binding`，不得在 `onDestroyView()` 后持有 View、Job、回调或 Adapter 引用；新增基类时优先采用可空 backing field 并清空。
- Fragment 的 Flow 收集、Adapter、动画、播放器监听、Insets listener 和任何读取 binding 的任务都绑定 `viewLifecycleOwner`，并在 `onDestroyView()` 对称解除。
- 基类不要自动调用依赖登录态、参数校验或其他门控的 `setupViews()`；由子类在前置条件满足后显式初始化，避免提前访问未准备好的会话依赖。
- ViewModel、Manager 和 Service 不持有 Activity、Fragment、ViewBinding、View 或页面 lambda。需要 UI 时发布状态/事件，由当前可用页面处理。
- 点击复用项目已有防抖扩展；不要重复实现计时器。
- 导航参数优先使用 Intent/arguments 的稳定键和可序列化数据；短生命周期回调只在无法序列化且生命周期明确时使用。

### 导航和页面关闭

- 先确定每个业务导航的唯一拥有者：页面、导航协调层或会话容器只能选一个。
- 状态变化和导航事件分开：状态负责可重放渲染，事件负责一次性跳转；不要在多个页面的同一状态分支重复 `startActivity()`。
- 发起页收到成功事件时可以关闭自身，但应避免在统一导航协调者取得当前宿主之前抢先销毁，防止导航竞态。
- 通知点击只恢复已经存在且语义明确的页面；不要根据中间状态重新创建会触发业务副作用的过渡页面。

## Dialog、Bottom Sheet 与键盘

- 居中弹窗使用统一基类处理透明背景、宽度和默认取消策略；产品明确要求前不要改变关闭行为。
- Dialog 的 binding 使用可空 backing field，并在 `onDestroyView()` 清空；同时取消与 View 绑定的 Job、动画和回调。
- Bottom Sheet 统一处理透明父背景、顶部圆角、固定 TAG 和展示入口。
- 参数优先放 `arguments`。需要系统恢复或配置重建的 Dialog 不使用字段 Lambda、静态 callback Map 或构造 callback，结果通过 Fragment Result 或可恢复状态返回。
- 跨多个 Activity/Fragment 的长流程由 Manager 保存待处理业务上下文，而不是保存函数。必要时使用无业务 UI 的协调 Fragment，在同一 FragmentManager 中接收 Dialog 结果并转交 Manager。
- 协调 Fragment 使用固定 TAG 保证同一宿主只有一个实例，不加入 Back Stack；返回键应关闭业务 Dialog 或宿主，不额外消费一次返回。
- Fragment 事务、Dialog 展示和 View 操作必须在主线程完成。若先写 pending 状态再展示 UI，展示失败必须回滚，宿主真正结束和所属 Manager 释放时也要清理。
- 需要避让键盘时使用 `WindowInsetsCompat.Type.ime()`，更新根容器 margin/padding；不要根据屏幕高度猜测键盘。
- 显示/隐藏键盘复用统一扩展，保证焦点与 IME 状态同步。

### 权限和跨页面待处理操作

- 权限属于 UI 与系统交互，但权限通过后要继续的业务参数属于 ViewModel/Manager；保存目标 ID、来源、模式等业务语义，不保存 callback。
- 权限流程顺序明确为：保存 pending 业务 -> 请求系统权限/设置 -> 读取最终系统状态 -> 完成或取消 pending 业务。
- 系统权限页和设置页可能导致宿主重建。使用 Activity Result API 和固定 TAG 的协调 Fragment 时，延迟到 Fragment 已附着后再访问 Activity 或会话依赖，禁止在构造期调用 `requireActivity()`。
- 持久化的“是否请求过权限”只用于选择交互路径，不能当作真实授权结果；最终权限始终读取系统 API。
- 与账号无关的权限提示历史使用全局 DataStore；用户或会话相关状态按对应生命周期保存，不额外创建零散 SharedPreferences。
- 用户拒绝权限时必须显式完成或清除 pending 操作，不能留下“仍在处理中”的假状态阻塞下一次请求。

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

## Target SDK 升级方法

- 先区分三类内容：target SDK 强制行为、同系统版本下所有应用的运行变化、可选体验增强。只有第一类直接进入升级阻塞清单。
- 推荐先提升 `compileSdk` 并暂时保留旧 `targetSdk`，解决编译、AndroidX 和第三方依赖兼容；再逐项启用兼容变更，最后切换 target。
- 对 IM、RTC、支付、推送、媒体和 WebView 等 SDK 驱动模块，除代码审计外还要取得厂商兼容声明，并在真实设备验证后台、权限、网络和原生库行为。
- 原生依赖不能只验证 APK 可安装或 ZIP 对齐；还要覆盖目标 page size、ABI、动态加载规则和 SDK 实际执行路径。
- 大屏合规只修复方向限制失效后暴露的状态丢失、遮挡和不可操作问题；双栏、侧边导航和视觉扩展属于产品体验优化，应单独立项。

## 维护原则

- 只记录可跨项目复用的个人设计倾向和编码规则。
- 项目目录、具体类名、业务调用链、接口字段和历史兼容关系放入对应项目 skill。
- 新规则必须来自多处验证或明确设计决策；若后续实践推翻规则，及时更新或删除。

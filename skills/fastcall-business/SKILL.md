---
name: fastcall-business
description: 理解、分析或修改 siubun_fastcall 工程时使用。记录该项目的模块地图、启动链、环境与会话依赖、网络客户端、IM、RTC、支付、动态、发现、聊天、分页、埋点和资源组织等项目专属知识。
---

# fastcall 项目架构与业务学习记录

此 skill 记录 `siubun_fastcall` 的项目事实、业务关系和历史实现，帮助快速理解工程。通用的个人编码与设计偏好放在 `siubun` skill，不在这里重复。

## 工程概览

- 单 Android `app` 模块，Kotlin + XML/ViewBinding，不使用 Compose。
- `minSdk 29`、`targetSdk/compileSdk 37`、Java/Kotlin 17。
- 两个 product flavor：
  - `local`：测试包和部分模拟业务。
  - `google`：正式 Google Play、Google Billing 和正式 applicationId。
- Debug/Release 都启用 Crashlytics mapping；Release 开启混淆和资源压缩。
- 主要技术栈：AndroidX Lifecycle、Coroutines/Flow、Paging 3、Retrofit/OkHttp、Room、DataStore、Media3、RongCloud IM、Agora RTC、Google Billing、Firebase、Adjust、Facebook。
- 测试目录当前只有模板测试；修改业务后主要依赖双 flavor 编译和针对性手工/设备验证。

## 主要目录职责

| 目录 | 职责 |
| --- | --- |
| `session/` | 应用容器、登录会话容器、会话恢复/登录/登出和持久化 |
| `startup/` | AndroidX Startup 初始化器 |
| `api/` | Retrofit 接口 |
| `repository/` | API、SDK、本地缓存和业务数据适配 |
| `net/` | URL、错误类型和 OkHttp 拦截器 |
| `datastore/`, `dao/` | DataStore、Room 和 DAO |
| `viewmodel/` | 页面状态、事件和业务编排 |
| `ui/act`, `ui/frag`, `ui/dialog` | Activity、Fragment、Dialog |
| `discover/`, `conversation/`, `moment/`, `mine/`, `match/` | 主要业务页面 |
| `im/` | RongCloud 适配、IM 业务服务、通知和消息 ViewHolder |
| `rtc/` | 呼叫状态机、信令、Agora 适配和通话页面 |
| `pay/` | Billing Facade、策略、Google Billing 实现和价格缓存 |
| `paging/` | PagingSource 及部分历史分页状态 |
| `statistics/`, `integration/` | Firebase/Adjust/Facebook 埋点和渠道参数 |
| `instant/`, `utils/` | 生命周期、点击、Intent、IME、shape、Glide 等扩展 |
| `entity/`, `model/` | 网络实体、UI/本地模型 |

## 启动链

1. AndroidX Startup 初始化日志、Lottie、SVGA、播放器、ActivityManager、网络观察器、DataStore 和 Room。
2. `MyApp.onCreate()` 恢复语言并创建 `AppContainer`。
3. `AppContainer.preInitialize()` 初始化本地环境和广告标识。
4. 初始化 Adjust、归因轮询和环境探测。
5. `AppContainer.initialize()`：
   - Cloak 请求失败不阻断。
   - `PkgAccount` 成功才进入 `EnvState.Ready`。
   - `PkgAccount` 失败进入 `EnvState.Error`。
6. 环境 Ready 后执行 `launchMore()` 和用户会话初始化。
7. `FirstAc` 同时观察环境状态和会话状态，决定启动进度和进入登录页/主页。

关键文件：

- `MyApp.kt`
- `startup/*.kt`
- `session/AppContainer.kt`
- `ui/act/FirstAc.kt`

## 双层依赖容器

### AppContainer

应用级容器，持有：

- `applicationScope`
- `LocalEnv` 和 `EnvState`
- 未登录可用的 Retrofit/OkHttp、API 和 Repository
- `UserSessionManager`
- Adjust 归因事件

公共网络客户端按接口需求组合 Header：

- `PkgHeaderInterceptor`：版本与 appId
- `EnvHeaderInterceptor`：语言、设备、visitor
- `ModelHeaderInterceptor`：机型
- `ChannelHeaderInterceptor`：归因渠道
- `CkHeaderInterceptor`：审核标识

### UserSessionContainer

每次登录按 `LoginResponse` 新建，持有：

- 带 access token 的认证 OkHttp/Retrofit
- `UserRepository`
- `ConversationRepository`
- `MessageRepository`
- `AnchorRepository`
- `MomentRepository`
- `PaymentRepository`
- `RtcRepository`
- `IMClient` / `IMService`
- `CallManager`
- `BillingManager`
- `CountdownManager`
- `sessionScope`

初始化时拉取账号、钱包、商品，连接 IM，启动消息、充值、通知和用户状态相关收集。

销毁时注销应用前后台监听、退出 IM、释放 CallManager、清理 DataStore/Room、关闭通知和 Billing、清倒计时并取消 `sessionScope`。

## 会话状态与数据清理

`UserSessionManager` 是登录态唯一来源：

- `SessionState.Loading`
- `SessionState.LoggedIn`
- `SessionState.LoggedOut`

初始化顺序：

1. 拉取 AppConfig。
2. 从 `SessionPersistence` 恢复缓存登录响应。
3. 有缓存则使用 refresh token 刷新。
4. 刷新成功调用 `login()` 创建新会话容器。
5. 无缓存或确认刷新失败则 LoggedOut。

登录、登出、删除账号使用同一个 `Mutex` 串行执行。

- `logout()` 调用 `clearSessionServices(false)`：清普通用户数据，保留 `session_` 前缀偏好。
- `deleteAccount()` 调用 `clearSessionServices(true)`：清全部用户数据，并清全局 deviceId。
- 登录响应仍由 `SessionPersistenceImpl` 使用 SharedPreferences JSON 保存；这是当前事实，不代表通用推荐。
- `AppContainer.launchMore()` 观察 LoggedOut，并清任务栈跳转 `LoginAc`。
- 页面需要登录态服务时使用 `whenLoggedIn()` 或观察 `sessionState`，不能假设服务一定非空。

## 网络与错误链路

认证客户端在公共 Header 基础上增加：

- `AuthInterceptor`
- `SessionAuthInterceptor`

`SessionAuthInterceptor` 处理服务端 HTTP 555 业务错误：

- token 过期映射为 `TokenExpiresException`
- 其他业务码映射为 `ApiException`

协程通常通过 `safeLaunch()` 进入 `ExceptionHandle`：

- `CancellationException` 继续抛出。
- token 过期或 RongCloud token 错误触发 `UserSessionManager.logout()`。
- 网络、超时和 JSON 错误显示统一提示。

修改认证或错误处理时，需要保持“协议错误 -> 统一异常 -> 会话退出”的完整链路。

## 本地数据与缓存

- `LocalPreferences`：
  - global DataStore：与用户无关。
  - user DataStore：按 userId 建立。
  - 普通 key：登出或删除账号时清理。
  - `session_` key：登出保留，删除账号清理。
- `LocalDatabase` 是 Room 单例，包含翻译缓存、通话历史和会话 pin 等表。
- 会话销毁会调用 `LocalDatabase.clear()`。
- `TranslationRepository` 使用 memory LRU -> Room -> network 的三级读取，并回填缓存。
- `ConversationRepository` 同时封装 RongCloud 会话能力和 Room pin 数据。

## UI 骨架

### Activity

多数页面继承 `Edge2EdgeAc`：

- 创建 ViewBinding
- 设置 `FLAG_SECURE`
- 启用 Edge-to-Edge
- 统一窗口 inset 入口

`MainAc` 使用 `ViewPager2 + SimpleFragmentStateAdapter` 固定组织四个主页面：

1. Chat
2. Discover
3. Moment
4. Mine

工程整体仍是多 Activity 架构，不应误解为强制单 Activity。

### Fragment

主 Fragment 继承 `BaseFragmentV2`，实现 `createViewBinding()` 和 `setupViews()`。

`BaseFragmentV2` 已改为 View 生命周期安全的可空 binding，并在 `onDestroyView()` 清空。基类不自动调用 `setupViews()`，登录态和其他初始化门控仍由子类显式控制。

### Flow 收集

`instant/CoroutineExt.kt` 提供：

- `safeLaunch`
- `launchWhenCreated`
- `launchWhenResumed`
- `startDelay`
- `startLoop`

生命周期扩展内部使用 `repeatOnLifecycle`。Fragment 页面通常通过 `viewLifecycleOwner.launchWhenCreated/Resumed` 收集。

## 页面与业务场景

### 首页

`MainAc` 聚合：

- 主 Tab 状态
- 未读消息数
- 钱包/免费通话状态
- 可见 IM 消息
- 随机匹配入口
- 全局通知和弹窗

`MainVM.UiState` 主要维护当前 Tab 和未读数，但部分聚合逻辑仍在 Activity 内，是当前历史形态。

### 发现

`DiscoverFrag` 负责国家筛选、促销入口和发现页签。

- `RegionVM` 管理国家列表和选中项。
- Popular/New/Following 使用独立 Fragment。
- Popular 是当前 Paging 3 推荐参考：
  - `pagingFlow.collectLatest -> adapter.submitData`
  - `adapter.loadStateFlow` 驱动刷新、加载、空态和错误
  - 错误重试调用 `adapter.refresh()`
  - 钱包配置、在线状态和关系事件触发局部更新或刷新

### 动态

`MomentMainFrag`/`PageMomentFrag`/`PersonalMomentFrag` 组织动态列表。

- `MomentPageVM` 创建 Paging Flow。
- `MomentAdapter` 使用 payload 更新点赞、评论和关注状态。
- `MomentOperationEventBus` 与 `UserRelationEventBus` 同步跨页面操作结果。

### 会话与消息

`ChatFrag` 只组织 Conversation 和 CallHistory 子页。

`ConversationAdapter` 按会话类型分发多种 ViewHolder。

`MessageDetailAc` 是消息详情聚合页：

- `MessageAdapter` 根据内容和方向映射多种消息 ViewType。
- 图片、语音、视频、礼物、卡片、通话邀请分别处理。
- 输入区由 `InputVM` 管理文字/语音模式和录音事件。
- IME inset 调整输入容器底部。
- 消息、翻译、语音播放、用户资料和未读清理通过多个 ViewModel/Flow 协作。

### 匹配

`RandomMatchVM` 使用 `UiEvent` 表达：

- 列表成功
- 匹配成功
- 匹配中
- 匹配失败
- 余额不足
- 命中主播

`local` flavor 在部分匹配和通话页面具有测试分支。

## IM 子系统

### 分层

- `IMClient`：RongCloud SDK Adapter，负责初始化、连接、登出和原始事件流。
- `IMService`：业务编排，订阅消息、离线同步和连接状态。
- ViewModel/UI：只消费业务流和 Repository，不直接监听 SDK。

### 业务处理

`IMService`：

- 客服消息自动清未读。
- 重要私聊/群聊消息写入会话 pin。
- 可展示消息进入 `visibleMessageFlow`。
- 离线同步完成触发刷新。
- 通知消息进入通知协调链路。

`ImMessageVM` 只接收当前用户会话的可展示消息。

`VideoCallMessageVM` 只接收目标聊天室频道的文本消息。

### 通知

`MessageNotifyCoordinator` 使用并发队列、原子状态和最小时间间隔合并通知，避免消息风暴。

`MessageNotificationManager` 是旧实现痕迹，新功能不要以它作为主参考。

## RTC 子系统

### 核心抽象

- `RtcEngine` / `AgoraRtcEngineImpl`
- `SignalingEngine` / `ImSignalingEngineImpl`
- `CallManager`
- `CallState`

`CallManager` 是唯一呼叫业务 Facade 和状态机入口，维护：

- CallStatus
- RtcJoinPhase
- RemoteUserState
- LocalUserState
- TimeWatcher

### 事件链

1. `startService()` 初始化 RTC 和信令引擎。
2. 同时收集 `RtcEvent` 与 `SignalingEvent`。
3. 来电、接听、拒绝、挂断、忙线和余额变化转换为状态。
4. Agora 加入、远端加入/离开、首帧、token 过期和网络质量转换为状态。
5. UI 观察状态并调用 CallManager 意图方法，不直接修改状态。

来电会先进行 RTC 预加入，通过 `RtcJoinPhase` 记录进度和重试。

pickup/reject/hangup/outgoing 等操作使用 `Mutex` 避免并发。

释放时需要同时释放 RTC、信令、计时任务和事件收集。

### 状态与导航边界

- `CallManager.state` 是唯一持续通话状态；页面只根据状态恢复 UI。
- `CallManager.events` 表达一次性业务事件，由 `UserSessionContainer` 统一消费主要导航：
  - `OnOutgoingStarted`：按 `CallOrigin` 打开 `OutgoingAc` 或 `RandomMatchAc`；
  - `OnIncoming`：打开 `IncomingAc`；
  - `OnFakeIncoming`：在免打扰允许时打开 `FakeIncomingAc`；
  - `OnPickup`：唯一一次启动 `VideoCallAc`。
- `IncomingAc`、`OutgoingAc`、`FakeIncomingAc`、`RandomMatchAc` 不在
  `CONNECTED` 状态中再次启动 `VideoCallAc`；它们收到 `OnPickup` 只负责关闭自身。
- Service 和通知不创建 `IncomingAc`、`OutgoingAc`、`RandomMatchAc` 等业务过渡页，
  避免恢复页面时重新驱动流程。

### 四类呼叫入口

#### 普通外呼

```text
业务页面点击呼叫
  -> CallPermissionRequester.requestOutgoing()
  -> CallManager.prepareOutgoing() 保存 targetUid/callOrigin/source/freeType
  -> 相机、麦克风权限
  -> CallNotificationPermissionFragment 处理通知说明、系统授权或设置页
  -> CallManager.completePendingCallPermission()
  -> outgoingApi()
  -> PREPARING + 启动前台服务 + OnOutgoingStarted
  -> UserSessionContainer 打开 OutgoingAc
  -> 外呼接口成功，状态 OUTGOING，开始超时与 RTC 预加入/预录视频
  -> 对方接听或模拟接听，状态 CONNECTED + OnPickup
  -> UserSessionContainer 唯一启动 VideoCallAc
```

#### 真实来电

```text
IM SignalingEvent Incoming
  -> CallManager.handleIncoming()
  -> 状态 INCOMING + RTC 预加入 + OnIncoming
  -> UserSessionContainer 打开 IncomingAc
  -> 用户点击接听
  -> CallPermissionRequester.requestPickup()
  -> 通知权限协调
  -> pickupApi() 启动前台服务、调用接听接口、完成/复用 RTC 加入
  -> 状态 CONNECTED + OnPickup
  -> UserSessionContainer 唯一启动 VideoCallAc
```

来电响铃本身不启动前台服务；用户确认接听后才启动。拒绝、超时或远端取消会重置状态并
结束来电页。

#### 假来电

```text
CallFakeRequest / OnFakeIncoming
  -> FakeIncomingAc 展示模拟来电
  -> 用户点击接听
  -> closeFakeCall() 清除模拟状态
  -> requestOutgoing(CallOrigin.FAKE_CALL)
  -> 统一权限协调
  -> 发起一次真实外呼
```

`closeFakeCall()` 产生的中间 `IDLE` 是流程切换，不是页面终止协议；
`FakeIncomingAc` 不能监听该状态直接 `finish()`，否则会销毁权限协调宿主并遗留 pending
外呼。

#### 随机匹配

```text
MainAc 发起随机匹配
  -> RandomMatchVM 保存匹配状态和待导航目标
  -> 命中目标后 MainAc 在 RESUMED 状态消费目标
  -> requestOutgoing(CallOrigin.MATCH_RECOMMEND)
  -> 统一权限协调
  -> OnOutgoingStarted
  -> UserSessionContainer 打开 RandomMatchAc
  -> OnPickup 后统一进入 VideoCallAc
```

匹配结果需要跨 `MainAc` 配置重建保存；消费后清除，不能因旋转重复发起外呼。

### 配置重建治理

- `IncomingAc`、`FakeIncomingAc` 通过 `IncomingCallPageVM` 固定最终预览随机结果；播放器仍由 Activity 持有，重建时保存位置、播放意图和快照时间。
- `VideoCallAc` 使用 `VideoCallPlaybackVM` 保存预录视频播放状态，使用 `VideoCallFreeTimeVM` 保存剩余时间和免费通话结束门控。
- RTC 会话、频道成员和通话计时继续由 `CallManager` 持有；Activity 重建只创建并绑定新的本地/远端 Surface，不重新发起通话或加入频道。
- `UiVideoCallMessageVM` 持续转接聊天室消息，避免 Activity 重建空窗丢失消息；数据源附加必须幂等。
- RTC 页面销毁时释放 Player、Adapter、动画和 View 回调，但不能因 View 销毁结束会话级业务。
- 应用前后台由 `ProcessLifecycleOwner` 判断，不能使用 Activity started/stopped 计数立即
  判定；横竖屏重建不会触发无通知权限的后台挂断。
- 配置重建重点检查一次性副作用：外呼 API、接听信令、`OnOutgoingStarted`、
  `OnPickup`、RTC join 和 `VideoCallAc` 导航均只能发生一次。

### 后台通话

- `CallForegroundService` 只在单次通话期间运行，声明 `camera|microphone` 类型；
  外呼进入准备阶段或用户点击接听后启动，`CallStatus.IDLE`、会话释放或登出时停止。
- 服务观察 `CallManager.state` 更新持续通知，不维护第二份通话状态。
- 应用进入后台后，`CallManager` 保持 RTC、麦克风和远端音频，暂停实际本地视频采集；
  回到前台后按用户原始视频意图恢复。
- 通知连接前没有点击跳转；`CONNECTED` 后正文只返回 `VideoCallAc`。
- 通知没有挂断 action，挂断继续由通话页面调用 `CallManager.hangupApi()`。
- 外呼 `PREPARING` 阶段保存对应 Job；页面挂断会取消请求并立即重置状态，避免等待
  同一 API Mutex 中的网络请求完成后仍继续拨号。
- `CallNotificationPermissionFragment` 在相机/麦克风通过后协调说明 Dialog、系统通知
  权限和应用通知设置页；结果通过 Fragment Result，不保存页面 callback。
- 首次点击开启请求 `POST_NOTIFICATIONS`；是否曾请求过通过全局 `LocalPreferences`
  记录，曾请求过但未开启时进入应用通知设置。该标记只决定交互路径，真实权限始终读取
  系统状态。
- 返回后调用 `CallManager.completePendingCallPermission()`，根据最终权限决定本次
  通话是否允许后台继续。
- 通知权限未开启仍允许前台呼叫，但 `CallManager` 会在应用真正进入后台时自动挂断；
  权限开启则后台保留语音并暂停视频。
- 普通外呼、来电接听、假来电确认和随机匹配命中后的外呼均使用同一权限协调链；
  假来电先关闭模拟来电状态，随机匹配先取得目标用户，再保存 pending 外呼。
- 后台策略只应用于 `OUTGOING/CONNECTING/CONNECTED`。若在 `PREPARING` 时已进入
  后台，状态转为 `OUTGOING` 时会再次校验，避免漏掉自动挂断。
- 打开系统通知授权或应用通知设置期间设置权限协调门控，避免系统页面切换被误判为普通
  退后台并挂断。
- 当前只支持“进行中的通话退后台继续”；后台收到的新来电仍不主动拉起页面。

### RTC 修改约束

- 新增呼叫入口必须经过 `CallPermissionRequester`，不能直接调用 `outgoingApi()`。
- 新增通话页面不得自行启动 `VideoCallAc`；如需改变导航，先调整
  `UserSessionContainer` 的唯一事件消费链。
- 不要让前台服务、通知或 Activity 各自维护第二份通话状态。
- 修改 `IDLE`、`PREPARING` 等状态语义前，必须走查假来电、pending 权限、外呼取消和
  服务停止四条链路。
- 验证 RTC 改动时至少覆盖真实来电、普通去电、假来电、随机匹配、通知允许/拒绝、
  通话中旋转和真实退后台。

## 支付子系统

### 分层

- `PaymentRepository`：商品、订单创建、验单、消费记录和商品缓存。
- `BillingManager`：支付业务 Facade。
- `IBillingStrategy`：渠道策略接口。
- `GoogleBillingStrategy`：Google BillingClient Adapter。
- `ProductPriceCache`：productId 到价格/币种缓存。

### flavor 差异

- `google`：服务端创建订单后拉起 Google Billing，购买后验单/确认/消费。
- `local`：测试环境模拟订单和成功结果。
- 未支持 flavor 明确报错，不做静默成功。

`BillingManager` 使用同步集合去重正在验证的 purchase key。

价格展示和币种读取应通过 `PaymentRepository`/`ProductPriceCache`，避免 UI 自行解析 Billing 结果。

支付成功后的充值、首充、新注册充值和高/中价值事件由 `UserSessionContainer` 统一上报。

### 支付方式选择

- `BillingManager` 保存单个 `PendingPaymentSelection`，内容是 requestId、期望 Activity 类型、用户/来源/商品参数和支付渠道列表，不保存函数。
- 渠道请求完成后，`BillingManager` 重新取得同类型且已恢复的 Activity，在主线程同步安装 `PaymentSelectionResultFragment`，再展示 `SelectPaymentDia`。
- `SelectPaymentDia` 只通过 Fragment Result 返回 requestId 和选中索引；取消返回 `-1`。
- `PaymentSelectionResultFragment` 与 Dialog 使用同一 FragmentManager，固定 TAG、不同宿主各最多一个实例、不加入 Back Stack，并把结果转交给 `BillingManager.completePaymentSelection()`。
- Fragment 的调试标签定义在 XML 中，仅 `BuildConfig.DEBUG` 显示。
- Fragment 安装、pending 写入和 Dialog 展示必须在主线程完成；展示失败必须回滚 pending，宿主真正结束或 Billing 释放时也要清理。
- 当前只保证配置重建连续性；进程死亡会丢失内存中的 pending，不自动恢复未完成支付。

## 埋点与归因

`StatisticsManager` 是 provider fan-out：

- `FirebaseProvider`
- `AdjustProvider`
- `FacebookProvider`

安装事件由 `InstallHelper` 触发。

应用打开/切后台时长、充值分层等业务事件由 `UserSessionContainer` 集中触发。

`TbaRepository` 维护独立的事件队列和周期 flush。

添加埋点前先搜索已有事件名和参数命名，避免同一业务重复定义。

## 列表与分页现状

### 推荐实现

发现、动态等新页面使用 Paging 3：

- `Pager + PagingSource`
- `cachedIn(viewModelScope)`
- `PagingDataAdapter`
- `loadStateFlow`
- `adapter.refresh()`

`PopularFrag` + `PopularPagingAdapter` 是首选参考。

### 历史实现

工程仍保留：

- `PagingListUiState`
- `PagingFooterState`
- `PagingUiEvent`
- `FooterAdapter`
- 手写 refresh/loadMore

`CallHistoryVM` 同时存在手写分页和 Paging Flow，是迁移中状态。新增分页功能不要复制双轨实现；维护时先确认页面当前使用哪条链。

## Adapter 与 DiffUtil

- 普通列表使用 `ListAdapter`。
- 分页列表使用 `PagingDataAdapter`。
- Conversation/Message Adapter 按业务类型映射 ViewHolder。
- DiffUtil 使用稳定业务 ID。
- Moment/Popular Adapter 使用 payload 做必要的局部状态更新。
- 点击需要基于当前 `bindingAdapterPosition` 重新取 item。

## Dialog、Bottom Sheet 与 IME

- `BaseDialogFragment` 默认不可取消、透明 Window 背景并在启动时设置宽度。
- `CoinsComboDia` 是居中 Dialog 的 ViewBinding/清理参考。
- `SelectLanguageDia`、`GiftDia` 是 Bottom Sheet 参考：
  - 透明 parent 背景
  - 顶部圆角
  - 固定 TAG 与 show/create
  - 参数放 arguments
- Bottom Sheet 需要避让键盘时，通过 `WindowInsetsCompat.Type.ime()` 更新根布局 bottom margin。
- `ViewExt.toggleIme()` 统一控制焦点和键盘。
- `MessageDetailAc` 的 IME 逻辑是聊天输入区参考。
- 可恢复 Dialog 不再使用对外字段 callback：参数放 arguments，结果通过 Fragment Result 返回。
- `GiftDia`、`MarketSourceDia`、`TopSelectCountryDia` 和 `MessageActionDia` 是 Fragment Result 参考。
- `MoreDia` 的举报跳转在 Dialog 内通过同一 FragmentManager 打开 `ReportDia`。
- `UserSessionManager` 不持有 Activity，也不直接创建 AlertDialog；登录重试和会话提示由所属页面展示 `MessageActionDia`。

## Target API 37 项目关注点

- 工程已切换到 `compileSdk/targetSdk 37`。
- 后台通话采用前台服务：后台保留语音、暂停视频，返回前台恢复视频。
- `sw >= 600dp` 上固定方向和不可调整限制将彻底失效；现有 API 36 生命周期治理是升级基础，完整 RTC/支付/自由窗口矩阵仍由测试验收。
- 业务代码未发现局域网访问，不申请 `ACCESS_LOCAL_NETWORK`；仍需厂商确认 Agora/
  RongCloud 内部实现。
- Release 已停止信任用户 CA，Debug 保留抓包；Release BODY 日志已关闭。
- 第三方支付仍可能返回 HTTP，因此 cleartext 和 WebView mixed content 暂未关闭，
  需要支付后端逐渠道确认 HTTPS。
- 验证 Agora、RongCloud、PAG、SVGA、xlog、Crashlytics NDK 等原生 SDK 的 API 37、无锁 MessageQueue、只读动态加载文件和 16 KB page size 兼容性。
- 审计来电、IM 推送、通知和支付是否依赖后台直接启动 Activity；优先使用通知、合规全屏 Intent 或 Telecom。

## XML 与资源

- 主要页面广泛使用 ConstraintLayout。
- `ac_main.xml` 展示 ViewPager2、自定义 Bottom Bar 和悬浮元素组织。
- `frag_discover.xml` 展示筛选区、TabLayout、ViewPager2 和促销入口约束。
- `ac_message_detail.xml` 展示消息列表和复杂输入区。
- 字体资源位于 `res/font`，主要为 Poppins 系列。
- 颜色同时包含原始色值和语义颜色；容器背景使用原始/背景色，文本使用语义文本色。
- `ViewShapeEx.kt` 提供运行时圆角、边框和渐变。
- `bottom_nav_height` 是底部导航复用尺寸。
- `local/google` 可能通过 source set 和 `BuildConfig.FLAVOR` 改变配置或行为。

## 已知历史边界

- `SessionPersistenceImpl` 仍使用 SharedPreferences，而用户偏好使用 DataStore。
- `ExceptionHandle` 同时承担 toast 和会话退出等全局副作用。
- `MainAc`、`MessageDetailAc`、`CallManager`、`BillingManager`、`UserSessionContainer` 体量较大，修改前需先定位子流程，避免继续扩张职责。
- 支付方式选择的 pending 只存在内存中，支持配置重建但不支持进程死亡恢复。
- RTC 完整业务阶段的旋转、锁屏、后台和挂断边界仍需测试矩阵验收。
- `NetworkConnectObserver.cleanup()` 存在，但应用级观察器通常随进程存在。
- 旧分页状态和 Paging 3 并存。
- 部分文件保留大量注释掉的历史代码；新实现不要复制。

## 修改前参考选择

| 场景 | 首选参考 |
| --- | --- |
| 应用/会话依赖 | `AppContainer`, `UserSessionManager`, `UserSessionContainer` |
| 登录状态门控 | `SessionGate.whenLoggedIn` |
| 生命周期 Flow | `instant/CoroutineExt.kt` |
| MVI 状态/事件 | `LoginVM`, `RandomMatchVM`, `InputVM` |
| Paging 3 页面 | `PopularFrag`, `PopularPagingAdapter` |
| 动态 payload | `MomentAdapter`, `MomentPageVM` |
| 普通多类型列表 | `ConversationAdapter`, `MessageAdapter` |
| 居中 Dialog | `BaseDialogFragment`, `CoinsComboDia` |
| Bottom Sheet | `SelectLanguageDia`, `GiftDia` |
| IM 业务 | `IMClient`, `IMService`, `MessageNotifyCoordinator` |
| RTC 状态机 | `CallManager`, `CallState`, `AgoraRtcEngineImpl` |
| 支付 | `PaymentRepository`, `BillingManager`, `GoogleBillingStrategy` |
| 翻译缓存 | `TranslationRepository` |
| 网络认证错误 | `SessionAuthInterceptor`, `ExceptionHandle` |

## 校验命令

修改 Kotlin 或 XML 后至少编译两个 Debug flavor：

```sh
./gradlew :app:compileLocalDebugKotlin :app:compileGoogleDebugKotlin
```

按改动范围追加现有 lint 或测试任务；不要为了文档引入新的测试工具。

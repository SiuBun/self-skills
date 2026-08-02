# Compose 架构、状态与导航

## 一、选择足够的架构

Compose 不要求固定使用 MVVM、MVI 或 Redux，但要求数据单向流动、状态来源明确且副作用可控。

推荐的最小链路：

```text
用户操作
  -> Screen 回调或 Event Sink
  -> ViewModel/Presenter
  -> Repository/Manager
  -> UiState
  -> Route 收集
  -> Screen 渲染
```

简单页面可以使用 `UiState + callbacks`。当页面包含并发请求、登录态、权限、支付、离线数据或多个交互阶段时，再引入 Action/Event/Reducer，不为了形式制造样板代码。

## 二、Route 与 Screen

### Route

负责：

- 获取 ViewModel 或 Presenter。
- 使用 `collectAsStateWithLifecycle()` 收集状态。
- 将导航和平台回调转换为 Screen 回调。
- 处理与页面宿主直接相关的少量 Effect。

### Screen

负责：

- 只根据参数渲染。
- 将用户操作通过回调或统一事件入口上报。
- 持有焦点、滚动、动画等纯 UI 状态。
- 可独立 Preview、截图和 UI 测试。

公共组件不得自行获取业务 ViewModel。

## 三、合法状态建模

以下模型容易产生非法组合：

```kotlin
data class UiState(
    val isLoading: Boolean,
    val data: Data?,
    val error: Throwable?,
)
```

复杂页面应使用 sealed hierarchy 或明确阶段：

```kotlin
sealed interface UiState {
    data object Loading : UiState
    data object Empty : UiState
    data class Content(val data: Data) : UiState
    data class RefreshFailed(val data: Data, val message: UiText) : UiState
    data class Failed(val message: UiText) : UiState
}
```

关键不是语法，而是：

- UI 不猜测字段组合。
- 刷新失败可以保留旧数据。
- Offline、Permission、Locked、Paywall 等业务状态被显式表达。
- 底层 `Throwable` 不直接进入 UI。

## 四、State、Action 与 Event

| 类型 | 含义 | 示例 |
| --- | --- | --- |
| State | 可重复渲染的持续事实 | 内容、选中页签、加载状态 |
| Action/Intent | 输入和异步结果 | 点击、刷新、请求成功 |
| Event/Effect | 只消费一次的命令 | 导航、Snackbar、分享 |

可从 State 推导的内容不要重复发 Event。导航结果如果属于持续业务事实，应进入 State；纯命令式跳转使用 Event。

复杂页面可以统一入口：

```kotlin
fun onAction(action: UiAction)
```

但小页面继续使用具名回调更清晰时，不强制封装 Action。

## 五、单一状态源

多条流共同决定 UI 时，在 ViewModel/Presenter 中组合：

```kotlin
val uiState = combine(accountFlow, contentFlow, permissionFlow) { account, content, permission ->
    buildUiState(account, content, permission)
}.stateIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(5_000),
    initialValue = UiState.Loading,
)
```

禁止在 Composable 中通过多个 `LaunchedEffect` 把 ViewModel A 的状态复制到 ViewModel B，再观察 B。需要拆分职责时，上层状态持有者应组合下层公开 Flow，或抽取共享业务层。

## 六、生命周期与状态恢复

- 页面业务任务使用 `viewModelScope`。
- 纯 UI 动画或焦点任务使用 `rememberCoroutineScope()`。
- 应用、会话和播放器任务使用明确拥有者的 Scope。
- 不在 Composable 中创建 `MainScope()`。
- 渲染状态使用 `collectAsStateWithLifecycle()`。
- 手写收集需要 `repeatOnLifecycle`，名称必须与行为一致。
- `remember` 用于 Composition 生命周期状态。
- `rememberSaveable` 用于可序列化、需跨配置恢复的 UI 状态。
- 业务状态和复杂恢复使用 ViewModel 与 `SavedStateHandle`。

## 七、Effect 规则

### `LaunchedEffect`

用于与 Composition 生命周期绑定的挂起任务。Key 改变意味着取消旧任务并启动新任务。

- `LaunchedEffect(Unit)` 只适合进入 Composition 时启动一次。
- 参数变化应重启任务时，将参数放入 key。
- 回调变化但不应重启任务时，使用 `rememberUpdatedState`。

### `DisposableEffect`

用于注册监听、播放器、观察者等需要对称释放的资源。

### `SideEffect`

仅用于把成功 Composition 的结果同步给 Compose 外部对象，不用于请求或长任务。

## 八、导航

- 路由和参数集中定义，优先类型安全 Route。
- Screen 不持有 NavController，只接收 `onBack`、`onOpenDetail` 等业务回调。
- Bottom navigation 使用 `saveState/restoreState` 保存各 Tab 状态。
- 防止快速重复导航；明确 `launchSingleTop` 和 `popUpTo` 语义。
- 深链、登录跳转和进程恢复必须有唯一事实来源。
- ViewModel 的作用域与导航范围一致，同类型详情页需要稳定 key，避免实例串台。

## 九、错误链路

```text
网络/数据库/SDK 异常
  -> Data Result
  -> Repository 业务错误
  -> UiState/UiError
  -> 降级内容、Snackbar、Dialog 或重试
```

- 数据源不得用空列表或 `null` 伪装成功。
- Repository 将技术错误转换为业务语义。
- UI 不判断 Retrofit、Room 或第三方 SDK 异常。
- 认证失效、锁定和登出由会话入口统一处理。

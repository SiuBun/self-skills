---
name: fastcall-migrate-api37
description: 将 siubun_fastcall 的 feature/migrate_api（2026-08-13 至 2026-08-21）生命周期重建、响应式尺寸、Dialog/支付、聊天媒体、RTC 前后台通话和 Target API 37 改造，按语义迁移到同源轻度改名的 Android 项目时使用。
---

# Fastcall `migrate_api` 语义迁移

## 目标

把 `SiuBun/siubun_fastcall` 的 `feature/migrate_api` 改造成果迁移到同源 Android 项目。
目标项目通常复制自同一套代码，只改了包名、部分类名、资源名或少量业务入口。

本 skill 不是补丁应用器。必须阅读源实现与目标候选文件，按职责和调用关系迁移，不能
直接 cherry-pick、`git apply` 或按行号机械替换。

迁移结果应覆盖：

- Activity/Fragment/Dialog 配置重建安全；
- ViewBinding、Adapter、监听器和媒体资源的生命周期绑定；
- 页面副作用和可靠状态归属；
- 横竖屏、折叠/展开、分屏与动态容器尺寸；
- 聊天、媒体、商店和支付流程的重建连续性；
- RTC 四类入口、唯一导航、权限协调和前后台通话；
- `compileSdk/targetSdk 37`、前台服务、网络日志与 Release CA 策略；
- 双 flavor 或目标项目对应变体的编译和设备验收。

## 源实现

优先使用本机源仓库：

```text
/Users/jason/AndroidStudioProjects/siubun_fastcall
```

若该路径不存在，使用已有本地 clone，或通过 GitHub 仓库
`SiuBun/siubun_fastcall` 获取源代码。不要把目标项目代码上传到第三方。

源分支与最终提交：

```text
branch: feature/migrate_api
final:  503ddc98766db56a4140b2552427aa7e7e2df22a
base:   63fa72319ae8b606d2fbf91b16417fce958e1be4
```

有效迁移提交按顺序为：

| 阶段 | 提交 | 主题 |
| --- | --- | --- |
| 1 | `3f264329` | Fragment View 生命周期基础 |
| 2 | `cb8462b6` | 副作用和状态归属 |
| 3 | `101b3df5` | 主页配置重建 |
| 4 | `ceb6c0ed` | 主播墙按容器尺寸布局 |
| 5 | `4aa6ba0e` | 常规页面方向与重建 |
| 6 | `78bb939c` | 媒体播放页转屏 |
| 7 | `ac3e80e5` | 聊天详情页 |
| 8 | `d3e046a0` | 支付页面 |
| 9 | `f14225f1` | RTC 页面 |
| 10 | `2405f1ab` | 可恢复 Dialog 与支付协调 |
| 11 | `dd8c69b0` | Target 37、RTC 前台服务和通知权限 |
| 12 | `503ddc98` | 最终走查修正 |

中间存在 `feature/change` merge，不属于本迁移的语义来源。检查提交时优先查看上表中的
非 merge 提交，并以 `503ddc98` 的最终文件状态为准。

## 开始前必须完成

1. 确认目标工作树状态，不覆盖用户已有改动。
2. 从目标项目当前基线创建独立迁移分支。
3. 阅读目标项目构建文件、Manifest、包结构、flavor、基类、会话容器和 RTC 入口。
4. 找出目标项目的编译命令；只使用已有构建、测试和 lint 工具。
5. 记录源类到目标类的候选关系，但不要要求用户手工提供完整映射。
6. 若目标项目已有部分迁移，先判断行为是否等价，再决定复用、补齐或跳过。

## 等价文件识别方法

按以下顺序匹配，不只依赖完整类名：

1. 相同或近似文件名，例如 `CallManager` / `VideoCallManager`；
2. 相同父类、接口、构造依赖和关键方法签名；
3. 相同布局 ID、Intent key、事件类型或 Retrofit API；
4. 相同调用链，例如“匹配成功 -> 外呼 -> 去电页”；
5. 阅读候选文件，确认其业务职责和生命周期；
6. 多个候选都合理时才向用户确认。

搜索源类名未命中时，继续搜索：

- 方法名和事件名；
- XML ID、字符串资源和 Intent extra；
- Repository/API 方法；
- 调用方和被调用方；
- 同目录中名称最接近的类。

不得因为目标类名不同就跳过功能，也不得仅凭名称相似就直接覆盖。

## 迁移原则

- 保留目标项目包名、业务命名、渠道差异、资源风格和已有功能。
- 搬运设计和行为，不搬运无关格式化、历史注释或源项目专属业务。
- 新增 helper 前先搜索目标项目是否已有等价实现。
- 页面重建是 View 容器替换，不是业务重新开始。
- Activity/Fragment 只绑定 View、收集状态、处理权限和用户操作。
- 长业务放入 ViewModel、会话 Manager 或应用级对象。
- 状态可重放，导航和 Toast 等事件只消费一次。
- Dialog 不保存页面 lambda；参数进 arguments，结果走 Fragment Result。
- RTC、支付等跨页面流程保存业务参数，不保存 callback 或旧 Activity。
- 每阶段完成后先编译，再进入下一阶段。

## 执行阶段

### 阶段 1：Fragment View 生命周期

源参考：

- `ui/frag/BaseFragment.kt`
- 各业务 Fragment 和 DialogFragment；
- 提交 `3f264329`。

实施要求：

- Fragment binding 使用可空 backing field，并在 `onDestroyView()` 清空；
- 访问 View 的 Flow 收集绑定 `viewLifecycleOwner`；
- Adapter、`TabLayoutMediator`、PageChangeCallback、刷新监听、Insets listener、动画和
  View 回调在 `onDestroyView()` 对称释放；
- 基类不要自动调用有登录态或其他前置条件的 `setupViews()`；
- 子类继续在门控完成后显式初始化。

审计重点：

- `lifecycleScope` 中访问 binding；
- Dialog 的字段 binding 和 callback；
- RecyclerView/ViewPager Adapter 持有旧 View；
- `onDestroyView()` 后仍运行的 Job。

通过标准：

- 连续旋转不访问旧 View；
- 不产生重复收集器、Adapter 或监听器；
- 登录门控和页面初始化顺序不改变。

### 阶段 2：副作用和可靠状态归属

源参考：

- `FirstVM`, `MainVM`, `PurchaseUsersVM`；
- `FirstAc`, `MainAc`, `MineFrag`；
- 商店 Activity/Dialog；
- 提交 `cb8462b6`。

实施要求：

- 区分首次加载、用户刷新和每次可见刷新；
- 启动导航、支付结果、匹配目标等可靠结果使用可恢复状态；
- Toast、短暂提示等允许丢失的视觉反馈使用无回放事件；
- 周期任务由 ViewModel/Manager 持有，不因 Activity 重建重启；
- 倒计时保存截止时间或业务时间基准，不由 View 从初始秒数重启。

通过标准：

- 初始化期间旋转只导航一次；
- 页面重建不重复请求、匹配、曝光、支付或倒计时任务。

### 阶段 3：主页、Dialog 与方向限制

源参考：

- `MainAc`, `DiscoverFrag`, `ChatFrag`, `MomentMainFrag`；
- `FreeCardDia`, `SayHiDia`, 国家选择 Dialog；
- `RandomMatchVM`；
- Manifest；
- 提交 `101b3df5`。

实施要求：

- 移除已治理 Activity 的固定竖屏声明；
- 主页面保存当前 Tab 和待处理匹配目标；
- 可恢复 Dialog 使用固定 TAG、arguments 和 Fragment Result；
- 系统栏 Insets 只由明确的一层负责，避免 Activity 与子 Fragment 重复叠加；
- 重建后任务栈中只能有一个主页实例。

### 阶段 4：主播墙和动态尺寸

源参考：

- `AnchorWallItemSize`；
- `AnchorWallGridController`；
- `GridEdgeSpacingItemDecoration`；
- Popular/New/Following Adapter 与 Fragment；
- 提交 `ceb6c0ed`。

实施要求：

- 卡片宽高由 RecyclerView 最终可用宽度、列数、间距和比例计算；
- 创建阶段或布局变化时计算尺寸，bind 阶段只应用结果；
- 不使用全局屏幕宽度，也不在每次 bind 重算比例；
- 窗口尺寸或列数变化时更新 item size 和 decoration；
- 点击时根据 `bindingAdapterPosition` 重新取 item。

若目标项目卡片数量、比例或间距不同，应保留目标视觉参数，只复用计算边界。

### 阶段 5：常规页面重建

源参考提交：`4aa6ba0e`。

重点页面：

- 用户关系、黑名单、访客；
- 编辑资料和个人资料；
- 动态详情和消费记录；
- 二维码；
- WebView、图片查看；
- 常规媒体选择。

实施要求：

- Intent 初始参数只在首次创建时应用；
- 未提交输入、Tab、媒体位置和选中项由页面 ViewModel 保存；
- 请求具备幂等门控，配置重建不重复发起；
- 二维码、图片采样和局部尺寸基于当前 View 实际大小；
- 清理 Insets、TextWatcher、Adapter 和媒体请求。

### 阶段 6：媒体播放

源参考：

- `MediaGalleryAc`, `MediaPagerAdapter`, `MediaGalleryPageVM`；
- 提交 `78bb939c`。

实施要求：

- 页面级状态保存当前媒体、位置和播放意图；
- Player 与 PlayerView 可以由 Activity 重建和重新绑定；
- 销毁旧页面时解除 Player listener 和旧 View；
- 不产生双播放器、声音叠加或从头错误重播；
- 明确只支持配置重建还是也支持进程恢复。

### 阶段 7：聊天详情

源参考：

- `MessageDetailAc`, `MessageDetailPageVM`；
- `InputVM`, `ImMessageVM`, `GiftVM`, `ChatFeatureVM`；
- `AudioPlayManager`, `AudioRecordManager`；
- 提交 `ac3e80e5`。

实施要求：

- 草稿、输入模式、聊天对象和页面可靠状态跨重建保存；
- 文字、图片、语音和礼物发送任务由 ViewModel 持有，每条只发送一次；
- 旋转时取消旧 Activity 的录音资源，但保留输入模式；
- 礼物等 Dialog 结果交给新宿主，不保存旧 Activity callback；
- 音频播放和录制 Manager 不持有销毁页面的 View。

### 阶段 8：商店与支付页面

源参考：

- `BillingManager`, `StorePageVM`；
- `CoinsStoreAc`, `NewStoreAc`, `VipStoreAc`, `ProfileAc`；
- `SelectPaymentDia`；
- 提交 `d3e046a0`。

实施要求：

- 商品、余额、选中项、滚动位置、倒计时和支付成功是可恢复状态；
- 页面曝光不因重建重复；
- Billing 异步结果完成后重新取得同类型且已恢复的 Activity；
- Manager 不保存旧 Activity；
- 同一 purchase/order 使用 request ID 或集合去重；
- flavor 差异保留在目标项目原有策略中。

### 阶段 9：RTC 页面重建

源参考：

- `CallManager`, `CallState`, `CallEvent`, `CallInfo`；
- `IncomingAc`, `OutgoingAc`, `FakeIncomingAc`, `RandomMatchAc`, `VideoCallAc`；
- `IncomingPlayerVM`, `VideoCallPlaybackVM`, `VideoCallFreeTimeVM`；
- `UiVideoCallMessageVM`, `VideoCallMessageVM`；
- 提交 `f14225f1`。

核心边界：

- `CallManager` 是唯一通话状态源；
- RTC 会话、频道成员、计时和媒体意图属于会话级；
- Activity 重建只重新绑定本地/远端 Surface 和渲染状态；
- 不能因 `setupViews()` 重复外呼、接听或加入频道；
- 预录视频保存位置、播放意图和单调时钟快照；
- 免费通话剩余时间与结束门控由 ViewModel/Manager 持有；
- 页面销毁只释放 Player、Adapter、动画和 View 回调。

导航规则：

- 找出目标项目统一消费 CallEvent 的会话协调层；
- `OnPickup` 只能由一个协调者启动最终通话页；
- 来电、去电、假来电和匹配页不得在 `CONNECTED` 状态重复启动最终通话页；
- 页面收到 `OnPickup` 可以关闭自身，但不能与统一导航抢占宿主形成竞态。

### 阶段 10：可恢复 Dialog 与支付协调

源参考：

- `MessageActionDia`；
- `PaymentSelectionResultFragment`；
- `SelectPaymentDia`, `GiftDia`, `MarketSourceDia`, `MoreDia`；
- `BillingManager`, `UserSessionManager`；
- 提交 `2405f1ab`。

实施要求：

- Dialog 参数放 arguments，结果走 Fragment Result；
- 支付 Manager 只保存 `PendingPaymentSelection` 的业务语义；
- 无业务 UI 的协调 Fragment 使用固定 TAG、不入 Back Stack；
- Fragment 与 Dialog 使用同一 FragmentManager；
- Fragment 安装、pending 写入和 Dialog 展示在主线程同步完成；
- 展示失败、宿主真正结束或 Manager 释放时清理 pending；
- 当前方案只承诺配置重建，默认不恢复进程死亡后的未完成选择。

### 阶段 11：Target API 37 与后台通话

源参考：

- `app/build.gradle.kts`, `gradle/libs.versions.toml`, Manifest；
- `ActivityManager`, `ActivityManagerInitializer`；
- `CallManager`, `CallForegroundService`；
- `CallPermissionRequester`, `CallNotificationPermissionFragment`；
- `AppContainer`, `UserSessionContainer`；
- `HttpLogging`, `network_security_config.xml`；
- 提交 `dd8c69b0` 与最终修正 `503ddc98`。

#### SDK 与进程生命周期

- 升级 `compileSdk/targetSdk` 到 37；
- 引入目标版本一致的 `lifecycle-process`；
- 应用前后台由 `ProcessLifecycleOwner` 提供；
- 不用 Activity start/stop 数量立即判断后台，旋转不能触发后台通话策略。

#### 通知权限协调

统一链路：

```text
保存 pending 呼叫参数
  -> 相机/麦克风权限
  -> 通知说明 Dialog
  -> 首次请求 POST_NOTIFICATIONS，曾请求则进入应用通知设置
  -> 读取系统最终权限
  -> 完成 pending 外呼或接听
```

- 是否曾请求通知权限使用全局 `LocalPreferences`/DataStore；
- 该标记只决定交互路径，最终权限以系统 API 为准；
- 协调 Fragment 使用固定 TAG，不在构造期调用 `requireActivity()`；
- 权限拒绝时清理或完成 pending，不能阻塞下一次呼叫；
- 系统权限/设置页期间设置协调门控，避免误判真实退后台。

#### 四类呼叫入口

- 普通外呼；
- 真实来电接听；
- 假来电转真实外呼；
- 随机匹配命中后外呼。

所有外呼入口都经过等价于 `CallPermissionRequester.requestOutgoing()` 的统一入口，不允许
页面直接调用底层 `outgoingApi()`。

假来电应先清理模拟状态，再保存和发起真实外呼。中间 `IDLE` 不是终止协议，假来电页
不能仅凭该状态立即 `finish()`。

#### 前台服务与通知

- 外呼进入 `PREPARING`、用户点击接听后，从可见状态启动
  `camera|microphone` 前台服务；
- Service 观察 `CallManager.state`，不维护第二份通话状态；
- `IDLE`、会话释放或登出时停止服务；
- 通知连接前不创建来电、去电或匹配过渡页；
- `CONNECTED` 后通知正文只返回最终通话页；
- 通知没有挂断 action，挂断由通话页面处理；
- Service 不反向驱动业务状态。

#### 前后台媒体策略

- 有通知权限：后台保留 RTC、麦克风和远端音频，暂停本地视频采集；
- 返回前台：按用户原本视频意图恢复，不覆盖用户主动关闭视频；
- 无通知权限：允许前台通话，真正进入后台后自动挂断；
- 后台策略只应用于 `OUTGOING/CONNECTING/CONNECTED`；
- 在 `PREPARING` 时已后台，转为 `OUTGOING` 后再次检查策略；
- 当前不支持应用已在后台时自动拉起来电页面。

#### 网络和日志

- Debug 可保留 BODY 日志，Release 使用 `NONE`；
- Release 不信任用户安装 CA，Debug 可通过 `debug-overrides` 抓包；
- 保留目标项目真实需要的 cleartext/WebView mixed content；未确认支付全 HTTPS 前不要
  擅自关闭；
- 未发现局域网业务时不申请 `ACCESS_LOCAL_NETWORK`，SDK 内部行为需设备验证。

## 最终静态审计

完成后搜索并逐项确认：

- 固定方向声明是否只剩明确未迁移页面；
- Fragment 是否仍在 View 销毁后访问 binding；
- Dialog 是否还有字段 lambda、静态 callback Map；
- 页面是否在 `onCreate/setupViews` 重复提交业务；
- 倒计时是否由 View 从初始值重启；
- 卡片 bind 是否重复计算屏幕尺寸；
- 支付 Manager 是否持有 Activity；
- 最终通话页的 `start()` 是否有多个消费者；
- 页面是否绕过统一权限入口直接调用外呼；
- Service/通知是否创建业务过渡页或保存第二份通话状态；
- Release 是否输出 BODY 日志或信任用户 CA；
- `CancellationException` 是否继续传播。

## 验证

至少执行目标项目已有的两个主要 Debug 变体编译。若命名与源项目相同：

```sh
./gradlew :app:compileLocalDebugKotlin :app:compileGoogleDebugKotlin
```

按改动范围追加现有单元测试、Release 编译或 lint，不引入新的测试框架。

设备侧至少覆盖：

- 连续旋转、折叠/展开、分屏、“不保留活动”；
- 启动、主页、聊天、媒体、商店和支付重建；
- 普通外呼、真实来电、假来电、随机匹配；
- 通知允许和拒绝；
- 通话中旋转与真实退后台；
- RTC Surface、计时、静音、摄像头和挂断边界；
- API 37 与 16 KB 原生 SDK 实际路径。

## 提交方式

- 每个阶段或紧密相关阶段单独提交；
- 提交前查看 diff，排除源项目包名、无关业务、格式化和资源污染；
- 不把源项目生成物、APK、临时报告或本地配置提交到目标仓库；
- 最终说明已迁移、目标项目不存在、刻意保留差异和仍需设备验证的项目。


# Mojoin 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_mojoin` | `feature/vip_change` | `565794e72fc6` |

## 系列说明

当前只有一个仓库，没有 Lite/Pro 派生包可比较。当前分支在旧快照
`57d444b33f24` 之后合入了大规模 VIP/Feed/UI 改造，涉及 774 个文件、
约 `+5037/-3964`。其行为属于 VIP/订阅增强版本，但发起通话前仍没有中央 VIP 门槛。

## 本次更新摘要

- Discover 从 Popular/New/Following 三页改为 Popular/Following 两页，New 源码保留但未挂载。
- Discover 右下角从 badge 6 金币商品改为非 VIP 订阅商品。
- 通话结束时未充值 VIP 用户的推荐商品也从 badge 6 改为第一项订阅商品。
- 主导航重新启用 Moment，Moment 内新增 Feed/Moment 两个子页；Profile Moment 仍未启用。
- 新增 Profile、Outgoing、Incoming、FakeIncoming 的 VIP 通话折扣价。
- Discover、Moment、Mine 都提供签到入口。
- 主底部导航中央增加 Match 动作按钮；它不是可切换页面，匹配命中后才打开 Dialog
  主题的 `RandomMatchAc`。Popular 没有随机匹配入口。
- Popular 首屏第六项为非 VIP Weekly VIP 商品卡。主播卡保留多封面控件和轮播
  Coordinator 实现，但当前 Coordinator 没有挂载，因此不能认定自动轮播正在运行。

## 2026-09-09 重新审计更正

本节用于明确撤销同一 HEAD 首次记录中的错误结论。重新审计不只搜索类名，还逐项核对
Fragment 挂载、XML 可见性、点击监听、Flow 收集和最终 Activity 启动链。

| 原记录 | 更正后的事实 |
|---|---|
| 主底部 Match 与 Popular 内嵌匹配条都是入口 | 当前唯一可达入口是主底部中央 Match 动作按钮；Popular 没有匹配控件或匹配点击链。 |
| 随机匹配余额不足打开 `NewStoreDia` | 实际打开 `CoinsStoreDia`，来源为 `match_recharge`。 |
| Popular 的部分可见多图卡会自动轮播 | `PopularCoverCoordinator` 和 `showNextIfVisible()` 存在，但没有实例化或挂载；当前只能确认首图展示和手动切图能力，不能确认自动轮播运行。 |
| 国家选择按钮可打开 `TopSelectCountryDia` | `iv_country` 在 XML 中固定为 `gone`，该点击链不可达；实际可用的是水平国家 chips。 |
| 5 秒禁挂只覆盖普通外呼和假来电转外呼 | `RandomMatchAc` 也在 `PREPARING` 时禁挂 5 秒；真实 `IncomingAc` 没有该限制。 |

随机匹配相关还保留 `frag_match.xml`、`MatchVM` 和 `RandomMatchVM.startMatch()` 等历史实现。
当前没有 `MatchFrag`，没有 `FragMatchBinding` 使用方，也没有代码挂载 `frag_match.xml`。
`PopularFrag` 虽调用 `MatchVM.fetchRandomAnchorUrl()`，但没有收集 `MatchVM.uiEvent`，
不会形成界面或入口。

## 检查结果

| # | 主题 | 行为 |
|---|---|---|
| 1 | 通话前 VIP | 中央 `CallManager` 只检查是否已有通话，Popular、Following、Profile、相册、通话记录、真实来电接听和假来电转外呼都没有 VIP 校验。普通外呼与假来电中按主播单价调用 `checkCoins(price)` 的代码已被注释，余额不足主要依赖服务端在发起后返回。随机匹配只在入口检查“金币大于 0 或免费次数大于 0”，没有按匹配主播单价预检，也不是 VIP 门槛。当前 Discover 不再挂载 New 页；其保留源码仍包含非 VIP 模糊遮罩。消息文字、图片、语音等发送能力会对非 VIP 弹 `WeeklyVipDia`，但这属于聊天权限，不构成全局呼叫门槛。 |
| 2 | Discover 商品 | 右下角现在读取 `filterForSubs().firstOrNull()`，即商品列表中的第一项订阅商品；只有 `wallet.isVip == false` 时显示，VIP 用户隐藏。点击时重新获取订阅商品并打开 `NewComboDia`。入口继续显示会话级 30 分钟循环倒计时，归零后重置，不代表商品过期。旧 badge 6 金币商品仍会被查询进 `NewUserGuide`，但不再参与该入口的显示或点击。 |
| 3 | 通话完成 | 只有已接通过的通话结束产生 `OnComplete` 后才进入该流程；忙线、拒接、黑名单、余额不足、超时和接通前挂断不进入。完成后提示并延迟 1 秒：非 VIP 弹 `WeeklyVipDia`；VIP 且金币大于 0 弹 `CallOverReviewDia`；VIP 无金币且已充值打开 `NewStoreDia`；VIP 无金币且未充值取第一项订阅商品并弹 `NewComboDia`。订阅商品为空时通过 `return@collect` 结束本轮，后续 App 评分也不会执行；商品存在时才继续独立计算评分资格。 |
| 4 | 商店 | 金币商店只保留 `type == 0`；混合新商店对非 VIP 保留全部非订阅商品和原始列表中的第一项订阅商品，对 VIP 隐藏订阅。这里是先取第一项订阅再做首充过滤，若该商品随后被过滤，不会回退到第二项订阅。badge 3/5 使用推荐大卡，其他订阅使用 VIP Bonus 卡，其余为普通金币卡。badge 3 持久化约 24 小时，badge 5 为进程内 5 分钟循环；其他首充商品虽然带注册剩余时间，但普通卡和 VIP Bonus 卡没有绑定该倒计时。badge 6 有实体与筛选方法，但各商品卡和 badge 文案均没有 badge 6 专项展示，也不再作为 Discover/通话结束的主推荐商品。 |
| 5 | Moment | 主导航已重新挂载 `MomentMainFrag`，用户可以进入。Moment 内包含 Feed 与普通 Moment 两个子页。Profile 的 `PersonalMomentFrag` 类仍在，但 Profile Tab 代码继续被注释，因此个人页只显示 Gift/Video，没有 Moment 子页。 |
| 6 | Feed | 已正式启用，位于主导航 Moment 的第一个子页。使用纵向 Paging 列表和顶部吸附切换，卡片内可横向展示多张封面；可见卡片每 3 秒推进封面，并每 15 秒刷新主播在线状态。 |
| 7 | VIP 折扣价 | `vipCallDiscount` 已接入。Profile 对非 VIP 显示“成为 VIP 后的折扣价提示”，主价格仍为原价；VIP 显示原价删除线和折后价。Outgoing、Incoming、FakeIncoming 对 VIP 显示 `round(originalPrice * vipCallDiscount)`，非 VIP 显示原价；有可用免费通话时隐藏价格组并显示 Free。`supportDiscountCall` 没有参与当前显示判断。`IncomingAc.start()` 虽传入 `PRICE`，实际展示仍读取主播详情中的 `anchorEntity.price`。MediaGallery 只显示 Free 标签，不展示原价或折扣价。 |
| 8 | 签到 | Discover、Moment、Mine 三处都有可点击入口并打开 `CheckInDia`，入口本身不要求 VIP。弹窗内真正领取时才检查身份：VIP 调用 `vipCheckIn()`，非 VIP 打开 `WeeklyVipDia`；非 VIP 的签到项同时显示锁状态。Mine 对同一个 `llCheckIn` 重复绑定了两次相同点击，但不形成第二种行为。 |
| 9 | Mine 免打扰 | 一小时本地时间戳和假来电抑制逻辑存在：有效期内关闭 `OnFakeIncoming`，不影响真实来电。但 `frag_mine.xml` 中入口固定为 `visibility="gone"`，用户当前无法操作。 |
| 10 | 国家筛选 | 只在 Popular 页且国家列表非空时显示水平国家 chips；点击 chip 更新 `RegionVM`，Popular 以 region code 重新请求分页。右侧 `iv_country` 虽绑定了 `TopSelectCountryDia`，但 XML 固定为 `gone`，用户当前无法点击该弹窗入口。Following 的请求固定使用空地区，不受筛选变化影响；整个可达筛选链没有 VIP 拦截。 |
| 11 | 随机匹配 | 当前唯一可达入口是主底部导航中央的 PAG 动画动作按钮，免费次数大于 0 时显示数量角标。它虽然枚举名为 `Tab.MATCH`，但 `MainAc` 只挂载 Discover、Moment、Chat、Mine 四页；点击 Match 不切页，而是直接进入匹配流程。流程先检查当前通话状态；金币 `<= 0` 且免费次数 `<= 0` 时打开 `CoinsStoreDia`，否则申请相机和麦克风权限并调用 `starMatch()`。结果为空只显示匹配失败提示；命中后取第一名 UID，由 `MainAc` 打开 Dialog 主题 `RandomMatchAc`，再调用 `outgoingApi()`。Popular 没有匹配入口；`frag_match.xml` 属于未挂载遗留页面。 |
| 12 | Discover | 当前只挂载 Popular 与 Following，New 类和非 VIP 模糊逻辑保留但不可达。Popular 为两列列表，首屏第六个位置会为非 VIP 插入 Weekly VIP 商品卡；主播卡将头像与相册组成多封面数据，但自动轮播所需的 `PopularCoverCoordinator` 没有挂载，当前不能认定会自动切图。Popular 的通话按钮恒定显示，只有 Free 标签按免费资格变化；Following 仅在符合免费通话条件时显示通话按钮，否则显示消息/Say Hi。 |
| 13 | App 评分 | 要求服务端开关为 `y` 且尚未消费展示标志。首次成功关注且当前不在通话时按 100% 调度；首次付费通话按 100% 调度；免费虚拟通话结束且剩余免费次数为 1 时按 50% 调度。统一延迟约 3 秒。展示标志在开始倒计时时就写入，而不是弹窗成功显示后写入，因此 Activity 无效导致未显示时也可能消耗机会。通话结束流程会先展示 VIP、商店或通话评价弹窗，再调度 App 评分，评分弹窗没有检查其他弹窗是否仍在显示。Like 调用 Google Play Review，普通/负向只关闭。 |
| 14 | 5 秒禁挂 | 普通外呼 `OutgoingAc` 和随机匹配 `RandomMatchAc` 都在收到 `PREPARING` 后禁用挂断 5 秒。`FakeIncomingAc` 不是进入假来电页就禁挂，而是用户点击接听、转成真实外呼并收到 `OnOutgoing` 后禁用挂断 5 秒。真实来电 `IncomingAc` 没有 5 秒限制，拒接按钮可直接调用 `rejectBySelf()`。 |

## 相对旧快照的行为变化

| 主题 | `57d444b33f24` | `565794e72fc6` |
|---|---|---|
| Discover 页签 | Popular/New/Following | Popular/Following；New 未挂载 |
| 右下角促销 | badge 6 金币商品 | 非 VIP 第一项订阅商品 |
| 通话结束首充商品 | badge 6 金币商品 | 第一项订阅商品 |
| Moment | 主入口隐藏 | 主入口启用 |
| Feed | 无可达 Feed | Moment 第一子页启用 Feed |
| Profile Moment | 隐藏 | 仍隐藏 |
| VIP 折扣价 | 未接入 | Profile、Outgoing、Incoming、FakeIncoming 已接入 |
| 签到 | Discover、Mine | Discover、Moment、Mine |
| 随机匹配入口 | 旧快照曾记录为 Popular 内嵌条，本次未重新检出旧 HEAD 验证 | 当前唯一可达入口为主底部中央 Match 动作；Popular 无入口 |
| 匹配过渡页 | 普通全屏 Activity | Dialog 主题 Activity |
| Popular VIP 卡片 | 已有数据链但位置规则不突出 | 明确在非 VIP 首屏第六项插入 |

## 关键证据

- `ui/act/MainAc*`、`ui/widget/HomeIndicatorView*`、`res/layout/view_bottom_tab.xml`
- `viewmodel/RandomMatchVM*`、`match/RandomMatchAc*`、`res/layout/frag_match.xml`
- `discover/DiscoverFrag*`、`PopularFrag*`、`FollowingFrag*`
- `discover/PopularCoverCoordinator*`、`ui/adapter/PopularMultiplePagingAdapter*`
- `viewmodel/NewStoreVM*`、`ui/adapter/NewStoreAdapter*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`
- `rtc/view/Outgoing*`、`Incoming*`、`FakeIncoming*`
- `moment/MomentMainFrag*`、`FeedFrag*`
- `ui/act/ProfileAc*`
- `paging/PopularPagingSource*`

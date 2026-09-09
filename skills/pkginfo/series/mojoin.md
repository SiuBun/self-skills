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
- 主底部导航增加 Match 动作，匹配命中后打开 Dialog 主题的 `RandomMatchAc`。
- Popular 首屏第六项为非 VIP Weekly VIP 商品卡，并保留活动封面轮播。

## 检查结果

| # | 主题 | 行为 |
|---|---|---|
| 1 | 通话前 VIP | 中央 `CallManager` 只检查是否已有通话，Popular、Following、Profile、相册、通话记录和假来电转外呼都没有 VIP 校验。当前 Discover 不再挂载 New 页；其保留源码仍包含非 VIP 模糊遮罩。消息文字、图片、语音等发送能力会对非 VIP 弹 `WeeklyVipDia`，但这属于聊天权限，不构成全局呼叫门槛。 |
| 2 | Discover 商品 | 右下角现在读取 `filterForSubs().firstOrNull()`，即商品列表中的第一项订阅商品；只有 `wallet.isVip == false` 时显示，VIP 用户隐藏。点击时重新获取订阅商品并打开 `NewComboDia`。入口继续显示会话级 30 分钟循环倒计时，归零后重置，不代表商品过期。旧 badge 6 金币商品仍会被查询进 `NewUserGuide`，但不再参与该入口的显示或点击。 |
| 3 | 通话完成 | 完成后提示并延迟 1 秒。非 VIP 直接弹 `WeeklyVipDia`；VIP 有金币时弹通话评价；VIP 无金币且已充值时打开普通金币商店；VIP 无金币且未充值时取第一项订阅商品并弹新用户组合。订阅商品为空时通过 `return@collect` 结束本轮，App 评分也不会继续执行；商品存在时再独立计算评分资格。 |
| 4 | 商店 | 使用统一新商店。非 VIP 显示全部非订阅商品和原始列表中的第一项订阅商品；VIP 过滤订阅。badge 3/5 使用推荐大卡，订阅使用 VIP Bonus 卡，其他为普通金币卡。badge 3 持久化约 24 小时，badge 5 为进程内 5 分钟循环，其他首充商品按注册剩余时间。badge 6 仍有实体和筛选方法，但已不再作为 Discover/通话结束的主推荐商品。 |
| 5 | Moment | 主导航已重新挂载 `MomentMainFrag`，用户可以进入。Moment 内包含 Feed 与普通 Moment 两个子页。Profile 的 `PersonalMomentFrag` 类仍在，但 Profile Tab 代码继续被注释，因此个人页只显示 Gift/Video，没有 Moment 子页。 |
| 6 | Feed | 已正式启用，位于主导航 Moment 的第一个子页。使用纵向 Paging 列表和顶部吸附切换，卡片内可横向展示多张封面；可见卡片每 3 秒推进封面，并每 15 秒刷新主播在线状态。 |
| 7 | VIP 折扣价 | `vipCallDiscount` 已接入。Profile 对非 VIP 显示“成为 VIP 后的折扣价提示”，主价格仍为原价；VIP 显示原价删除线和折后价。Outgoing、Incoming、FakeIncoming 对 VIP 显示 `round(originalPrice * vipCallDiscount)`，非 VIP 显示原价。MediaGallery 仍未展示价格或折扣。 |
| 8 | 签到 | Discover、Moment、Mine 三处都有可点击入口并打开 `CheckInDia`。Moment 顶部同时为文字和图标绑定点击；Mine 使用 Weekly Reward 卡片，并通过 `CheckInVM` 获取签到进度。 |
| 9 | Mine 免打扰 | 一小时本地时间戳和假来电抑制逻辑存在：有效期内关闭 `OnFakeIncoming`，不影响真实来电。但 `frag_mine.xml` 中入口固定为 `visibility="gone"`，用户当前无法操作。 |
| 10 | 国家筛选 | Popular 页在国家列表非空时显示水平国家区和国家选择入口；点击 `TopSelectCountryDia` 后更新 `RegionVM`，并通过 Popular 的地区 Flow 重新创建分页数据。当前另一页只有 Following，不随国家变化过滤；点击没有 VIP 拦截。 |
| 11 | 随机匹配 | 当前有两个可见入口：主底部导航中央 Match 动作，以及 Popular 页底部内嵌匹配条。入口先检查当前通话状态和免费次数/金币，余额不足时打开 `NewStoreDia`；否则申请相机、麦克风权限并匹配。命中后打开 `RandomMatchAc`，该 Activity 使用 `AppTheme.VideoCallDialog`，以弹窗式呼叫过渡界面呈现。 |
| 12 | Discover | 当前只挂载 Popular 与 Following，New 类和非 VIP 模糊逻辑保留但不可达。Popular 首屏会在第六个位置为非 VIP 插入 Weekly VIP 商品卡；主播卡将头像与相册组成最多 4 张封面，并对部分可见多图项启动轮播。Popular 呼叫按钮恒定，只有 Free 标签按免费资格变化；Following 仅免费可用时显示呼叫，否则显示消息/Say Hi。 |
| 13 | App 评分 | 要求服务端开关为 `y` 且尚未展示。首次成功关注且不在通话、首次付费通话、或剩余免费次数为 1 的免费虚拟通话按 50% 概率触发；延迟约 3 秒。Like 调用 Google Play Review，普通/负向只关闭。 |
| 14 | 5 秒禁挂 | `OutgoingAc` 收到 `PREPARING` 后禁用挂断 5 秒；假来电转成真实外呼并收到 `OnOutgoing` 时也执行同样限制。 |

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
| 随机匹配入口 | Popular 内嵌条 | 主底部 Match + Popular 内嵌条 |
| 匹配过渡页 | 普通全屏 Activity | Dialog 主题 Activity |
| Popular VIP 卡片 | 已有数据链但位置规则不突出 | 明确在非 VIP 首屏第六项插入 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `viewmodel/NewStoreVM*`、`ui/adapter/NewStoreAdapter*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`
- `rtc/view/Outgoing*`
- `moment/MomentMainFrag*`、`FeedFrag*`
- `ui/act/ProfileAc*`
- `paging/PopularPagingSource*`

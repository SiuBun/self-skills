# Mojoin 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_mojoin` | `feature/vip_change` | `57d444b33f24` |

## 系列说明

当前只有一个仓库，没有 Lite/Pro 派生包可比较。其行为属于 VIP/金币分流版本，
但发起通话前不强制 VIP。

## 检查结果

| # | 主题 | 行为 |
|---|---|---|
| 1 | 通话前 VIP | 中央 `CallManager` 只检查是否已有通话，Popular、Following、Profile、相册、通话记录和假来电转外呼都没有 VIP 校验。New 页对非 VIP 显示整卡模糊遮罩，点击遮罩打开 `WeeklyVipDia`，因此只在 New 形成 UI 层限制；消息发送也检查 VIP，但不等于全局呼叫门槛。 |
| 2 | Discover 商品 | 右下角选择第一项 badge 6 商品。若它是首充推荐，仅 `wallet.charged == false` 时显示；若不是首充推荐则持续显示。点击时重新查 badge 6 并打开 `NewComboDia`。入口显示会话级 30 分钟倒计时，归零后重置，不会使商品过期；查询到的新用户 VIP 商品当前未参与显示判断。 |
| 3 | 通话完成 | 完成后提示并延迟 1 秒。非 VIP 直接弹 Weekly VIP；VIP 有金币时弹通话评价；VIP 无金币且已充值时打开金币商店；VIP 无金币且未充值时查找“首充金币且 badge 6”商品并弹新用户充值。商品为空时本轮直接结束，之后的 App 评分与业务弹窗独立计算。 |
| 4 | 商店 | 使用统一新商店。非 VIP 显示全部非订阅商品和第一个订阅商品；VIP 过滤订阅。badge 3/5 使用推荐大卡，订阅使用 VIP Bonus 卡，其他为普通金币卡。badge 3 持久化约 24 小时，badge 5 进程内 5 分钟循环，其他首充商品按注册剩余时间；badge 6 没有独立 ViewType。 |
| 5 | Moment | `MomentMainFrag` 和 `PersonalMomentFrag` 源码仍在，但主导航 Moment 与 Profile Moment Tab 均被注释，用户当前无法进入。 |
| 6 | Feed | 没有独立可达 Feed 页面。 |
| 7 | VIP 折扣价 | 配置层虽提供 `getVipCallDiscount()`，但 Profile、相册、Outgoing、Incoming、FakeIncoming 都没有使用；呼叫/来电页直接显示主播原始 `price/min`。 |
| 8 | 签到 | Discover 右上角和 Mine Weekly Reward 卡片都可打开 `CheckInDia`，并通过 `CheckInVM` 获取签到进度；Moment 因入口隐藏，没有可达签到入口。 |
| 9 | Mine 免打扰 | 一小时本地时间戳和假来电抑制逻辑存在：有效期内关闭 `OnFakeIncoming`，不影响真实来电。但 `frag_mine.xml` 中入口固定为 `visibility="gone"`，用户当前无法操作。 |
| 10 | 国家筛选 | Popular 页在国家列表非空时显示水平国家区和选择入口；点击 `TopSelectCountryDia` 后更新 `RegionVM` 并刷新 Popular。New/Following 不随国家变化过滤，也没有 VIP 拦截。 |
| 11 | 随机匹配 | 入口是 Popular 底部内嵌匹配条，展示最多 5 个循环头像、匹配状态和免费次数。点击先检查通话状态；金币和免费次数都不足时打开 `NewStoreDia`，否则申请相机/麦克风权限并匹配，命中后以 `RandomMatchAc` 作为呼叫过渡页。 |
| 12 | Discover | 实际挂载 Popular/New/Following。Popular 将头像和相册组成最多 4 张封面，对部分可见多图卡片启用轮播，并可插入 Weekly VIP 商品卡；呼叫按钮恒定，Free 标签按资格变化。New/Following 仅免费可用时显示呼叫，否则显示消息/Say Hi；New 另有非 VIP 模糊遮罩。 |
| 13 | App 评分 | 要求服务端开关为 `y` 且尚未展示。首次成功关注且不在通话、首次付费通话、或剩余免费次数为 1 的免费虚拟通话按 50% 概率触发；延迟约 3 秒。Like 调用 Google Play Review，普通/负向只关闭。 |
| 14 | 5 秒禁挂 | `OutgoingAc` 收到 `PREPARING` 后禁用挂断 5 秒；假来电转成真实外呼并收到 `OnOutgoing` 时也执行同样限制。 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `viewmodel/NewStoreVM*`、`ui/adapter/NewStoreAdapter*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`
- `rtc/view/Outgoing*`

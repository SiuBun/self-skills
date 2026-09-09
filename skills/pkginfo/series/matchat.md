# Matchat 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD | 工作树 |
|---|---|---|---|---|
| 原包 | `siubun_matchat` | `feature/vip_change` | `424331c1b0fe` | clean |
| Lite | `siubun_matchatlite` | `feature/vip_change` | `e547092a993f` | `app/local/` 未跟踪 |
| Plus | `siubun_matchatplus` | `feature/vip_change` | `106c8c0fe437` | clean |
| Pro | `siubun_matchatpro` | `feature/vip_change` | `d91e18251e00` | clean |

## 系列关系与差异

四包活动业务源码高度一致，14 项范围内没有确认到 Lite/Plus/Pro 的行为分叉；
主要差异是 endpoint、包配置和资源。当前分支均为订阅/VIP 商品版本，但不强制 VIP 通话。

## 检查结果

| # | 主题 | 四包公共行为 | 变体差异 |
|---|---|---|---|
| 1 | 通话前 VIP | Discover、Profile、相册、消息页通话、通话记录、来电接听、假来电转外呼和通话后重拨都不校验 VIP；中央 `CallManager` 也不兜底。VIP 仍会限制 IM 文本/图片/语音、Moment 评论或可见内容和签到领取，因此不能概括为只影响价格与商品。 | 四包活动业务一致。 |
| 2 | Discover 商品 | 非 VIP 显示商品列表中的第一个订阅商品，入口附带会话级 30 分钟循环倒计时；归零后重置，不会隐藏。VIP 用户隐藏该订阅促销入口。 | 四包一致。 |
| 3 | 通话完成 | 非 VIP 弹 Weekly VIP；VIP 有金币时弹通话评价；VIP 无金币且已充值时打开金币商店；未充值时弹订阅/套餐组合。App 评分另行判断。 | 四包一致。 |
| 4 | 商店 | 混排 `NewStoreAc/NewStoreDia` 与独立 `WeeklyVipAct/WeeklyVipDia` 并存。混排商店中非 VIP 显示全部非订阅及首个订阅，VIP 只显示非订阅；badge 3/5 用推荐大卡，其他订阅用 VIP Bonus 卡。badge 6 只有实体与过滤方法，没有单独 UI 文案或卡片语义。 | 四包一致。 |
| 5 | Moment | 主导航 Moment 和 Profile 个人 Moment 均活动可达。 | 四包一致。 |
| 6 | Feed | Moment 第一子页为独立 Feed：纵向 Paging 列表，每张卡片内部横向展示相册；首个完整可见卡片可自动切换封面。 | 四包一致。 |
| 7 | VIP 折扣价 | Profile、相册、Outgoing、Incoming、FakeIncoming 都使用 `vipCallDiscount` 和 `roundToInt()`。Profile/相册对非 VIP 展示 VIP 价格提示；VIP 与呼叫过渡页直接显示折后价。当前未发现原价删除线实现。 | 四包一致。 |
| 8 | 签到 | Discover、Moment、Mine 都提供可点击入口并打开 `CheckInDia`；入口可见不代表非 VIP 可领取，非 VIP 点击领取会打开 `WeeklyVipDia`。 | 四包一致。 |
| 9 | Mine 免打扰 | 一小时本地时间戳与假来电抑制代码仍在，但 `llt_not_disturb` 在 XML 中固定为 `gone`，用户当前无法操作；不影响真实来电。 | 四包一致。 |
| 10 | 国家筛选 | Popular 页显示水平国家 chips，选择后刷新推荐数据；更多选项图标在布局中固定隐藏。没有 VIP 拦截，也不作用于 Following。 | 四包一致。 |
| 11 | 随机匹配 | Main 页面存在两个活动点击面：底栏中央 `clt_match` 和位于主页面中间区域的悬浮 `pag_match`。两者进入同一流程，只预检金币是否大于 0 或有免费次数，不按匹配主播单价预检；命中后打开 `RandomMatchAc`，再发起真实外呼。 | 四包一致。 |
| 12 | Discover | 当前只挂载 Popular 与 Following，New 类存在但未接入 Tab。Popular 的轮播 Coordinator 没有挂载，当前封面构造也只放头像，因此不会自动轮播；Popular 强制显示 PAG 通话按钮并隐藏 action，Following 按免费资格在通话与消息/Say Hi 间切换。 | 四包一致。 |
| 13 | App 评分 | 受远程开关和本地一次性标记控制；首次关注、首次付费通话或最后阶段免费虚拟通话可触发，约 3 秒后显示。 | 四包一致。 |
| 14 | 5 秒禁挂 | `OutgoingAc` 与 `RandomMatchAc` 都在 `PREPARING` 时用 Activity 本地计时禁用挂断 5 秒。配置重建会重新启动本地 5 秒计时，风险是延长锁定时间，而不是提前恢复。 | 四包一致。 |

## VIP 相关说明

- VIP 主要影响商品、通话结束分流和通话折扣价。
- 呼叫前没有 VIP 门槛。
- 折扣通常为 `round(originalPrice * vipCallDiscount)`。
- VIP 用户显示折后价；非 VIP 显示原价，并在 Profile/相册提示 VIP 折扣价。

## 关键证据

- `moment/FeedFrag*`、`FeedVM*`、`FeedPagingAdapter*`
- `ui/act/Profile*`、`MediaGallery*`
- `rtc/view/Outgoing*`、`Incoming*`、`FakeIncoming*`
- `discover/DiscoverFrag*`
- `session/UserSessionContainer*`
- `config/LocalEnv*`

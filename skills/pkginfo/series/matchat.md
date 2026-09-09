# Matchat 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_matchat` | `feature/vip_change` | `424331c1b0fe` |
| Lite | `siubun_matchatlite` | `feature/vip_change` | `e547092a993f` |
| Plus | `siubun_matchatplus` | `feature/vip_change` | `106c8c0fe437` |
| Pro | `siubun_matchatpro` | `feature/vip_change` | `d91e18251e00` |

## 系列关系与差异

四包活动业务源码高度一致，14 项范围内没有确认到 Lite/Plus/Pro 的行为分叉；
主要差异是 endpoint、包配置和资源。当前分支均为订阅/VIP 商品版本，但不强制 VIP 通话。

## 检查结果

| # | 主题 | 四包公共行为 | 变体差异 |
|---|---|---|---|
| 1 | 通话前 VIP | 普通呼叫入口及中央 `CallManager` 不校验 VIP，非 VIP 可以发起付费通话；VIP 只影响价格、商品和结束弹窗。 | 四包活动业务一致。 |
| 2 | Discover 商品 | 非 VIP 显示商品列表中的第一个订阅商品，入口附带会话级 30 分钟循环倒计时；归零后重置，不会隐藏。VIP 用户隐藏该订阅促销入口。 | 四包一致。 |
| 3 | 通话完成 | 非 VIP 弹 Weekly VIP；VIP 有金币时弹通话评价；VIP 无金币且已充值时打开金币商店；未充值时弹订阅/套餐组合。App 评分另行判断。 | 四包一致。 |
| 4 | 商店 | 统一商店混排金币与订阅。非 VIP 显示全部非订阅及首个订阅，VIP 只显示非订阅；badge 3/5 用推荐大卡，订阅用 VIP Bonus 卡。 | 四包一致。 |
| 5 | Moment | 主导航 Moment 和 Profile 个人 Moment 均活动可达。 | 四包一致。 |
| 6 | Feed | Moment 第一子页为独立 Feed：纵向 Paging 列表，每张卡片内部横向展示相册；首个完整可见卡片可自动切换封面。 | 四包一致。 |
| 7 | VIP 折扣价 | Profile、相册、Outgoing、Incoming、FakeIncoming 都使用 `vipCallDiscount`。VIP 主价格显示折后价；非 VIP 显示原价，并在 Profile/相册提示成为 VIP 后的价格。 | 四包一致。 |
| 8 | 签到 | Discover、Moment、Mine 都提供可点击签到入口，共用签到进度和奖励弹窗。 | 四包一致。 |
| 9 | Mine 免打扰 | Mine 开关保存开启时间；一小时内抑制假来电并关闭模拟状态，超过一小时自动失效，不影响真实来电。 | 四包一致。 |
| 10 | 国家筛选 | Popular 页显示水平国家 chips，选择后刷新推荐数据；更多选项图标在布局中固定隐藏。没有 VIP 拦截，也不作用于 Following。 | 四包一致。 |
| 11 | 随机匹配 | 主页面底部中央按钮启动匹配；命中后进入卡片式 `RandomMatchAc`，再沿统一呼叫链进入视频通话。 | 四包一致。 |
| 12 | Discover | 当前只挂载 Popular 与 Following，New 类存在但未接入 Tab。Popular 具备多封面轮播协调代码，但当前数据只提供头像，实际通常不会轮播；Popular 呼叫恒定，Following 按免费资格切换按钮。 | 四包一致。 |
| 13 | App 评分 | 受远程开关和本地一次性标记控制；首次关注、首次付费通话或最后阶段免费虚拟通话可触发，约 3 秒后显示。 | 四包一致。 |
| 14 | 5 秒禁挂 | Outgoing 在 `PREPARING` 状态禁用挂断 5 秒，匹配呼叫过渡流程也保持相同限制。 | 四包一致。 |

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

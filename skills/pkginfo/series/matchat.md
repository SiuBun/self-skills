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
| 1 | 通话前 VIP | 无强制 VIP | 无 |
| 2 | Discover 商品 | 非 VIP 显示首个订阅商品；30 分钟循环倒计时 | 无 |
| 3 | 通话完成 | 非 VIP→Weekly VIP；VIP 有币→评价；VIP 无币按充值状态进商店/套餐 | 无 |
| 4 | 商店 | 统一新商店；金币和订阅混排，按 VIP 状态过滤 | 无 |
| 5 | Moment | Main Moment 与 Profile Moment 均启用 | 无 |
| 6 | Feed | 有；Moment 第一子页，纵向分页卡片内横向相册 | 无 |
| 7 | VIP 折扣价 | Profile、相册、Outgoing、Incoming、FakeIncoming 全面支持 | 无 |
| 8 | 签到 | Discover、Moment、Mine 均有入口 | 无 |
| 9 | Mine 免打扰 | 可用；本地一小时，仅抑制假来电 | 无 |
| 10 | 国家筛选 | 水平 chips 可用；更多图标静态隐藏；无 VIP 拦截 | 无 |
| 11 | 随机匹配 | 底部中央按钮，命中后进入卡片式 `RandomMatchAc` | 无 |
| 12 | Discover | 仅 Popular/Following；Popular 轮播基础设施存在但当前数据只给头像，实际不轮播 | 无 |
| 13 | App 评分 | 已启用 | 无 |
| 14 | 5 秒禁挂 | 已启用 | 无 |

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

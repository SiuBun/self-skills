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
| 1 | 通话前 VIP | 无强制 VIP |
| 2 | Discover 商品 | badge 6 金币入口；首充商品仅未充值显示，非首充可一直显示；30 分钟循环 |
| 3 | 通话完成 | 非 VIP/VIP、金币余额、充值状态分流；未充值使用 badge 6 商品 |
| 4 | 商店 | 统一新商店；支持 badge 6 套餐首充 |
| 5 | Moment | 主 Moment 与 Profile Moment 均隐藏 |
| 6 | Feed | 无 |
| 7 | VIP 折扣价 | 无 |
| 8 | 签到 | Discover、Mine 有入口 |
| 9 | Mine 免打扰 | 可用；本地一小时，仅抑制假来电 |
| 10 | 国家筛选 | selector 可见可用，无 VIP 拦截 |
| 11 | 随机匹配 | Popular 底部内嵌条，显示状态、头像和免费次数 |
| 12 | Discover | Popular/New/Following；Popular 有活动轮播 |
| 13 | App 评分 | 已启用 |
| 14 | 5 秒禁挂 | 已启用 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `viewmodel/NewStoreVM*`、`ui/adapter/NewStoreAdapter*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`
- `rtc/view/Outgoing*`

# Cupid 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_cupid` | `feature/moment` | `4dc4942a965d` |
| Lite | `siubun_cupidlite` | `feature/merge` | `89dcc0a015e4` |
| Plus | `siubun_cupidplus` | `merge` | `e4d61bff6f9b` |

## 系列关系与差异

三个包在本次 14 项业务范围内基本一致，主要差异集中在品牌资源和商店配色：
Cupid/CupidLite 偏粉色，CupidPlus 偏紫色。未发现独立 VIP 分支业务。

## 检查结果

| # | 主题 | 三包公共行为 | 变体差异 |
|---|---|---|---|
| 1 | 通话前 VIP | 无强制 VIP；中央 `CallManager` 只检查通话状态 | 无 |
| 2 | Discover 商品 | 未充值时显示 badge 5 首充金币入口；无活动倒计时 | 无 |
| 3 | 通话完成 | 未购且有 badge 5 商品时弹首充，否则通话评价，再计算 App 评分 | 无 |
| 4 | 商店 | 金币/VIP 分开；badge 3/5 使用推荐样式 | Plus 配色不同 |
| 5 | Moment | Main Moment 与 Profile Moment 均启用 | 无 |
| 6 | Feed | 无独立 Feed | 无 |
| 7 | VIP 折扣价 | Profile、相册、呼叫/来电页均无 | 无 |
| 8 | 签到 | 无入口 | 无 |
| 9 | Mine 免打扰 | 无 | 无 |
| 10 | 国家筛选 | 图标可见且可用，无 VIP 拦截 | 无 |
| 11 | 随机匹配 | Popular 内嵌重叠头像入口 | 无 |
| 12 | Discover | Popular/New/Following；无列表轮播；Popular 呼叫恒定 | 无 |
| 13 | App 评分 | 远程开关、首次关注/通话触发、约 3 秒延迟 | Mine 头像保留 Debug 手动触发 |
| 14 | 5 秒禁挂 | `PREPARING` 禁用挂断，5 秒后恢复 | 无 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `ui/adapter/Popular*Adapter*`、`New*Adapter*`、`Following*Adapter*`
- `session/UserSessionContainer*`
- `ui/act/Profile*`、`moment/MomentMain*`
- `rtc/view/Outgoing*`
- `viewmodel/Store*`、`entity/StoreGoods*`

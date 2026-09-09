# Fastcall 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_fastcall` | `feature/vip_change` | `0058b61309cf` |
| Lite | `siubun_fastcalllite` | `feature/vip_dev` | `f7f805bd8d5c` |
| Pro | `siubun_fastcallpro` | `feature/vip_dev` | `2f793e9e0e00` |

## 系列关系与差异

三个当前分支都属于 VIP/订阅商品形态，但“VIP 商品版”不等于“发起通话必须是 VIP”。
本次检查范围内三包业务逻辑基本一致，差异主要是包配置和资源。

## 检查结果

| # | 主题 | 三包公共行为 | 变体差异 |
|---|---|---|---|
| 1 | 通话前 VIP | 无全局门槛，普通入口仍可直接进入呼叫链 | 无 |
| 2 | Discover 商品 | 非 VIP 显示 badge 6/订阅套餐入口；30 分钟循环倒计时 | 无 |
| 3 | 通话完成 | 非 VIP→Weekly VIP；VIP 有币→评价；VIP 无币已充→商店；未充→badge 6 套餐 | 无 |
| 4 | 商店 | 统一新商店；非 VIP 显示金币和首个订阅，VIP 过滤订阅 | 无 |
| 5 | Moment | 主 Moment 和 Profile Moment 均隐藏 | 无 |
| 6 | Feed | 无 | 无 |
| 7 | VIP 折扣价 | 无 | 无 |
| 8 | 签到 | Discover 有入口 | 无 |
| 9 | Mine 免打扰 | 一小时逻辑存在，但布局固定隐藏 | 无 |
| 10 | 国家筛选 | 顶部筛选 Dialog 可用，无 VIP 拦截 | 无 |
| 11 | 随机匹配 | 独立全屏模糊匹配界面 | 无 |
| 12 | Discover | Popular/New/Following；Popular 有活动轮播 | 无 |
| 13 | App 评分 | 已启用，受远程开关和首次行为条件控制 | 无 |
| 14 | 5 秒禁挂 | 普通外呼及相关过渡页启用 | 无 |

## 分支边界

- 当前分支名包含 `vip`，但实际 VIP 特性主要体现在订阅商品、通话后分流和促销弹窗。
- 不应仅根据分支名推断通话被 VIP 强制；入口代码必须逐一检查。

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `session/UserSessionContainer*`
- `viewmodel/NewStoreVM*`、`ui/adapter/NewStoreAdapter*`
- `ui/dialog/NewComboDia*`、`WeeklyVipDia*`
- `mine/MineFrag*`
- `match/RandomMatch*`
- `rtc/view/Outgoing*`

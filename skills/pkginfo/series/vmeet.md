# Vmeet 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_vmeet` | `feature/change` | `075237359166` |
| Lite | `siubun_vmeetlite` | `feature/change` | `b3c6daef6cf5` |
| Pro/VIP 商品版 | `siubun_vmeetpro` | `feature/vip_change` | `04c3ad9f2be4` |

## 系列关系与差异

Vmeet 与 VmeetLite 是标准 badge 5 首充版本。VmeetPro 增加订阅促销、签到、
通话结束 VIP 分流和 Popular 轮播，但发起通话前仍不强制 VIP。

## 检查结果

| # | 主题 | Vmeet / Lite | VmeetPro |
|---|---|---|---|
| 1 | 通话前 VIP | 无 | 无 |
| 2 | Discover 商品 | badge 5 首充，无倒计时 | 非 VIP 首个订阅商品，30 分钟循环 |
| 3 | 通话完成 | 首充弹窗或评价 | VIP/余额/充值分流 |
| 4 | 商店 | 标准金币/VIP 商店 | 统一新商店，按 VIP 状态过滤订阅 |
| 5 | Moment | Main 与 Profile Moment | Main 隐藏，Profile Moment 保留 |
| 6 | Feed | 无 | 无 |
| 7 | VIP 折扣价 | 无 | 无 |
| 8 | 签到 | 无 | Discover 有 |
| 9 | Mine 免打扰 | 一小时逻辑存在但 UI 隐藏 | 相同 |
| 10 | 国家筛选 | 可用，无 VIP 拦截 | 相同 |
| 11 | 随机匹配 | 独立全屏 | 独立全屏 |
| 12 | Discover | 三 Tab，无轮播 | 三 Tab，Popular 有活动轮播和 VIP 卡片 |
| 13 | App 评分 | 有 | 有 |
| 14 | 5 秒禁挂 | 有 | 有 |

## Pro/VIP 商品版边界

- VIP 影响促销和通话后商业分流，不构成呼叫前门槛。
- Main Moment 被隐藏，但 Profile 仍可查看 Moment。
- 与原包/Lite 的主要业务差异应优先从 Discover、Store、通话结束和签到链路检查。

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `session/UserSessionContainer*`
- `viewmodel/NewStoreVM*`
- `moment/MomentMain*`、`ui/act/Profile*`
- `ui/dialog/NewComboDia*`
- `rtc/view/Outgoing*`

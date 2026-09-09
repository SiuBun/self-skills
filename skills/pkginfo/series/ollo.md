# Ollo 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_ollo` | `feature/change` | `71fcfbeb0741` |
| Chat | `siubun_ollochat` | `main` | `cdcd9b457868` |
| Lite | `siubun_ollolite` | `feature/change` | `9c8c95d2c382` |
| Pro | `siubun_ollopro` | `feature/change_conversation` | `b1b6a60954d8` |

## 系列关系与差异

Ollo、OlloChat、OlloPro 在 14 项范围内基本一致。OlloLite 缺少 App 评分和外呼前
5 秒禁挂，并且 Discover 商品点击时的筛选比显示时更宽松。

## 检查结果

| # | 主题 | Ollo / Chat / Pro | OlloLite |
|---|---|---|---|
| 1 | 通话前 VIP | 仅 New 卡片对非 VIP 显示模糊拦截 | 相同 |
| 2 | Discover 商品 | badge 5 首充，未充值显示，无倒计时 | 显示相同；点击只按 badge 5 重查 |
| 3 | 通话完成 | 首充弹窗或评价，再计算 App 评分 | 首充弹窗或通话评价；无 App 评分 |
| 4 | 商店 | 金币/VIP 分页，badge 3/5 推荐样式 | 基本相同 |
| 5 | Moment | Main Moment 与 Profile Moment | 相同 |
| 6 | Feed | 无 | 无 |
| 7 | VIP 折扣价 | 无 | 无；`vipCallDiscount` 配置字段也不存在 |
| 8 | 签到 | 无 | 无 |
| 9 | Mine 免打扰 | 无 | 无 |
| 10 | 国家筛选 | Dialog 可用，无 VIP 拦截 | 相同 |
| 11 | 随机匹配 | Popular 悬浮入口，再进入全屏过渡 | 相同 |
| 12 | Discover | Popular/New/Following；无轮播 | 相同 |
| 13 | App 评分 | 有 | **完全不存在** |
| 14 | 5 秒禁挂 | 有 | **不存在，可立即挂断** |

## VIP 拦截边界

- New 页的模糊层只是一处页面级限制。
- Popular、Following、Profile 等入口仍可发起付费通话。
- 因中央 `CallManager` 无 VIP 校验，不能把 Ollo 系定义为强制 VIP 包。

## 关键证据

- `discover/NewFrag*`、`New*Adapter*`
- `discover/PopularFrag*`、`FollowingFrag*`
- `session/UserSessionContainer*`
- `config/LocalEnv*`
- `rtc/view/Outgoing*`

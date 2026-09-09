# Lumi 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| Lemie | `siubun_lemie` | `main` | `cb6bc1e4e82d` |
| 原包 | `siubun_lumi` | `main` | `037948833471` |
| Lite | `siubun_lumilite` | `main` | `0315a8c6dca2` |
| Pro | `siubun_lumipro` | `main` | `247536102f29` |

## 系列关系与差异

四包在本次 14 项范围内属于同一标准行为族。未发现影响产品流程的 Lite/Pro 特例，
差异主要是品牌、资源、接口配置和商店视觉。

## 检查结果

| # | 主题 | 四包公共行为 | 变体差异 |
|---|---|---|---|
| 1 | 通话前 VIP | 无强制 VIP | 无 |
| 2 | Discover 商品 | badge 5 首充金币；未充值显示；无活动倒计时 | 无 |
| 3 | 通话完成 | 未购首充→badge 5 弹窗，否则通话评价，再计算 App 评分 | 无 |
| 4 | 商店 | 金币/VIP 分页；badge 3/5 推荐样式 | 视觉资源不同 |
| 5 | Moment | Main Moment 和 Profile Moment 均启用 | 无 |
| 6 | Feed | 无 | 无 |
| 7 | VIP 折扣价 | 无 | 无 |
| 8 | 签到 | 无 | 无 |
| 9 | Mine 免打扰 | 可用；本地一小时，仅抑制假来电 | 无 |
| 10 | 国家筛选 | Dialog 可用，无 VIP 拦截 | 无 |
| 11 | 随机匹配 | Discover Popular 上方/底部悬浮 pill | 无 |
| 12 | Discover | Popular/New/Following；静态单封面，无轮播 | 无 |
| 13 | App 评分 | 已启用 | 无 |
| 14 | 5 秒禁挂 | 已启用 | 无 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`
- `moment/MomentMain*`、`ui/act/Profile*`
- `ui/dialog/Coins*Combo*`
- `rtc/view/Outgoing*`

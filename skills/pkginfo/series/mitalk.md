# Mitalk 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| 原包 | `siubun_mitalk` | `feature/change` | `02a9072240c8` |
| Lite | `siubun_mitalklite` | `feature/change` | `2e638993baeb` |
| Pro | `siubun_mitalkpro` | `main` | `75748f695a10` |
| U | `siubun_mitalku` | `feature/vip_change` | `2351edb3f55f` |

## 系列关系与差异

Mitalk、Lite、Pro 属于标准 badge 5 首充行为族；MitalkU 是明显的 VIP/订阅增强变体，
增加 Feed、签到、订阅促销和部分 VIP 通话折扣价。

## 检查结果

| # | 主题 | Mitalk / Lite / Pro | MitalkU |
|---|---|---|---|
| 1 | 通话前 VIP | 无 | 无 |
| 2 | Discover 商品 | badge 5 首充，无倒计时 | 非 VIP 首个订阅商品，30 分钟循环 |
| 3 | 通话完成 | 首充弹窗或评价 | VIP/余额/充值分流 |
| 4 | 商店 | 金币/VIP 分页 | 统一新商店，订阅按 VIP 状态过滤 |
| 5 | Moment | Main 与 Profile Moment | Main 与 Profile Moment |
| 6 | Feed | 无 | 有，Moment 第一子页 |
| 7 | VIP 折扣价 | 无 | Profile 正常；呼叫/来电页有格式风险；相册无 |
| 8 | 签到 | 无 | Discover、Moment、Mine |
| 9 | Mine 免打扰 | 一小时，仅抑制假来电 | 相同 |
| 10 | 国家筛选 | Dialog 可用，无 VIP 拦截 | 水平 chips，可用，无 VIP 拦截 |
| 11 | 随机匹配 | 主底部导航中央入口 | 相同 |
| 12 | Discover | Popular/New/Following，无轮播 | Popular/Following；Discover 无轮播，Feed 内有轮播 |
| 13 | App 评分 | 有 | 有 |
| 14 | 5 秒禁挂 | 有 | 有 |

## MitalkU 风险

呼叫和来电页把 `roundToInt()` 结果传给 `"%.2f"`，可能产生
`IllegalFormatConversionException`。修改相关页面时应优先确认运行时类型。

## 关键证据

- `discover/DiscoverFrag*`
- `moment/MomentMain*`、`FeedFrag*`
- `ui/act/Profile*`
- `rtc/view/Outgoing*`、`Incoming*`、`FakeIncoming*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`

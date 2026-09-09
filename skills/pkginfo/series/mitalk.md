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
| 1 | 通话前 VIP | 所有普通呼叫入口和中央 `CallManager` 均不要求 VIP，非 VIP 可进入付费通话。 | 同样没有全局 VIP 门槛；VIP 主要影响商品、Feed 权限和价格展示。 |
| 2 | Discover 商品 | 显示 badge 5 首充金币，要求商品为首充推荐且用户未充值；点击首充弹窗，无活动倒计时。 | 非 VIP 显示第一个订阅商品并展示 30 分钟循环倒计时；VIP 隐藏订阅促销。 |
| 3 | 通话完成 | 未购买金币且有 badge 5 商品时弹首充，否则通话评价，再判断 App 评分。 | 非 VIP→Weekly VIP；VIP 有币→评价；VIP 无币后按是否充值进入金币商店或订阅套餐。 |
| 4 | 商店 | 金币/VIP 分页，badge 3/5 使用推荐样式，首充受注册时间和充值状态限制。 | 统一新商店；非 VIP 显示非订阅和首个订阅，VIP 过滤订阅，订阅使用 VIP Bonus 卡。 |
| 5 | Moment | 主导航 Moment 与 Profile Moment 均活动可达。 | 主导航和 Profile Moment 同样可达。 |
| 6 | Feed | 没有独立 Feed。 | Moment 第一子页为 Feed，纵向分页卡片中包含横向封面和自动轮播。 |
| 7 | VIP 折扣价 | Profile、相册及呼叫/来电页都不应用 VIP 折扣。 | Profile 正常显示原价、折后价和删除线；相册无价格；Outgoing/Incoming/FakeIncoming 尝试显示折后价，但把 `Int` 传给 `"%.2f"`，存在崩溃风险。 |
| 8 | 签到 | Discover、Moment、Mine 均无活动入口。 | Discover、Moment、Mine 都有签到入口并打开共用签到弹窗。 |
| 9 | Mine 免打扰 | Mine 开关可操作，记录开启时间并在一小时内抑制假来电，不影响真实来电。 | 行为相同。 |
| 10 | 国家筛选 | Popular 页通过标准 Dialog 选择国家，更新后刷新 Popular，无 VIP 门槛。 | 使用水平国家 chips，选择后刷新 Popular；无 VIP 拦截，Following 不受影响。 |
| 11 | 随机匹配 | 主底部导航中央 Match 动作进入匹配流程，命中后发起统一外呼。 | UI 位置和流程相同。 |
| 12 | Discover | 挂载 Popular/New/Following，封面静态无轮播；Popular 呼叫恒定，New/Following 按免费资格切换。 | 只挂载 Popular/Following，New 未接入；Discover 卡片无轮播，但 Feed 卡片有轮播。 |
| 13 | App 评分 | 远程开关开启且未展示过时，由首次关注、首次付费通话或特定免费通话触发，延迟约 3 秒。 | 规则相同。 |
| 14 | 5 秒禁挂 | Outgoing `PREPARING` 时禁用挂断 5 秒。 | 同样启用。 |

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

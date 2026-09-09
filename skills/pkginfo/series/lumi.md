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
| 1 | 通话前 VIP | Popular/New/Following、Profile、相册、消息、通话记录和随机匹配均没有 VIP 使用门槛；中央通话入口只阻止并发通话。 | 四包一致。 |
| 2 | Discover 商品 | 右下角显示 badge 5 首充金币商品，条件是商品为首充推荐且用户尚未充值；点击打开首充组合弹窗。倒计时代码未活动，不会递减或到期隐藏。 | 四包一致。 |
| 3 | 通话完成 | 通话结束后查找 badge 5 首充金币商品；未购买金币且商品存在时弹首充，否则弹通话评价，随后独立判断 App 评分。 | 四包一致。 |
| 4 | 商店 | 金币与 VIP 商品分页面；badge 3/5 使用推荐卡，普通金币使用标准卡。首充商品按充值状态和注册剩余时间筛选。 | 业务筛选一致，品牌颜色和资源不同。 |
| 5 | Moment | 主导航 Moment 与 Profile 个人 Moment 子页均实际挂载，可正常进入和操作。 | 四包一致。 |
| 6 | Feed | 没有独立 Feed 页面，Moment 列表不单独归类为 Feed。 | 四包一致。 |
| 7 | VIP 折扣价 | Profile、相册、Outgoing、Incoming、FakeIncoming 均未根据 VIP 重算价格；页面显示原始价格或免费标签。 | 四包一致。 |
| 8 | 签到 | Discover、Moment、Mine 没有活动签到入口。 | 四包一致。 |
| 9 | Mine 免打扰 | Mine 提供可操作开关；开启时保存当前时间，一小时内拦截并关闭假来电，超时自动视为关闭，不影响真实来电和 IM。 | 四包一致。 |
| 10 | 国家筛选 | Popular 页国家图标在列表非空时显示，点击打开标准 Dialog；选中后刷新 Popular，没有 VIP 门槛，也不筛选 New/Following。 | 四包一致。 |
| 11 | 随机匹配 | 入口位于 Discover Popular 的悬浮 pill/头像条，点击后发起匹配流程，不占用独立主导航 Tab。 | 四包一致。 |
| 12 | Discover | 挂载 Popular、New、Following 三页。Popular 呼叫按钮恒定且仅 Free 标签变化；New/Following 按免费资格切换呼叫和消息。封面为静态单图，无轮播。 | 四包一致。 |
| 13 | App 评分 | 服务端开关开启且未展示过时，由首次关注、首次付费通话或特定免费虚拟通话触发；延迟约 3 秒，正向选择进入 Google Play Review。 | 四包一致。 |
| 14 | 5 秒禁挂 | Outgoing 收到 `PREPARING` 后禁用挂断按钮 5 秒，再恢复点击。 | 四包一致。 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`
- `session/UserSessionContainer*`
- `mine/MineFrag*`
- `moment/MomentMain*`、`ui/act/Profile*`
- `ui/dialog/Coins*Combo*`
- `rtc/view/Outgoing*`

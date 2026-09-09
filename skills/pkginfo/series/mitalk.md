# Mitalk 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD | 工作树 |
|---|---|---|---|---|
| 原包 | `siubun_mitalk` | `feature/change` | `02a9072240c8` | clean |
| Lite | `siubun_mitalklite` | `feature/change` | `2e638993baeb` | `LocalEnv.kt` 有未提交修改，`.github/` 未跟踪 |
| Pro | `siubun_mitalkpro` | `main` | `75748f695a10` | clean |
| U | `siubun_mitalku` | `feature/vip_change` | `2351edb3f55f` | clean |

## 系列关系与差异

Mitalk、Lite、Pro 属于标准 badge 5 首充行为族；MitalkU 是明显的 VIP/订阅增强变体，
增加 Feed、签到、订阅促销和部分 VIP 通话折扣价。

## 检查结果

| # | 主题 | Mitalk / Lite / Pro | MitalkU |
|---|---|---|---|
| 1 | 通话前 VIP | Popular/New/Following、Profile、相册、消息页通话和随机匹配都不要求 VIP，中央 `CallManager` 只检查通话状态。消息页的文本、图片和语音发送另有 VIP 门槛，但不属于通话门槛。 | 呼叫同样没有中央 VIP 门槛；消息内容发送会弹 `WeeklyVipDia`。 |
| 2 | Discover 商品 | 显示 badge 5 首充金币，要求商品为首充推荐且用户未充值；点击 `CoinsComboDia`，倒计时代码仅为注释。 | 非 VIP 即显示入口，点击第一项订阅商品并打开 `NewComboDia`，使用 30 分钟循环倒计时；`filterBadge6()` 虽被取数，但不参与当前显隐或点击。 |
| 3 | 通话完成 | 未购买金币且有 badge 5 商品时弹首充，否则通话评价；随后无条件继续检查 App 评分，可能与前一弹窗竞争。随机匹配没有专用完成页。 | 非 VIP→Weekly VIP；VIP 有币→评价；VIP 无币且已充值→`NewStoreDia`；VIP 无币且未充值→第一项订阅商品的 `NewComboDia`；随后同样检查评分。 |
| 4 | 商店 | Mine 分别进入 VIP 和金币商店；badge 3/5 使用推荐样式。badge 6 只有实体注释，没有适配器 UI 分支。 | Mine 使用 `WeeklyVipDia` 与混排 `NewStoreAc`；非 VIP 保留非订阅和首个订阅，VIP 过滤订阅，订阅使用 VIP Bonus 卡。badge 6 helper 存在，但适配器仍只处理 badge 1～5。 |
| 5 | Moment | 主导航 Moment 与 Profile Moment 均活动可达；主 Moment 子页为普通 Moment 与 Recommend。 | 主导航和 Profile Moment 同样可达；主 Moment 子页改为 Feed 与普通 Moment。 |
| 6 | Feed | 没有独立 Feed。 | Moment 第一子页为 Feed，纵向分页卡片中包含横向封面和自动轮播。 |
| 7 | VIP 折扣价 | Profile、相册及呼叫/来电页都不应用 VIP 折扣；Pro 只额外显示免费次数文案。 | Profile 中非 VIP 显示 VIP 折扣提示，VIP 才显示原价删除线和折后价；相册无价格。Outgoing/Incoming/FakeIncoming 将 `roundToInt()` 结果传给 `"%.2f/min"`，类型不匹配风险只存在于这三个 RTC 页面。 |
| 8 | 签到 | Discover、Moment、Mine 均无活动入口。 | Discover、Moment、Mine 都有签到入口并打开共用签到弹窗。 |
| 9 | Mine 免打扰 | Mine 开关可操作，记录开启时间并在一小时内抑制假来电，不影响真实来电。 | 行为相同。 |
| 10 | 国家筛选 | Popular 页通过可见右侧图标打开 `SelectCountryDia`，更新后只刷新 Popular，无 VIP 门槛。 | 活动入口是水平国家 chips；右侧 `iv_country` 虽有弹窗点击代码但 XML 固定为 `gone`。筛选只影响 Popular，无 VIP 拦截。 |
| 11 | 随机匹配 | 主底部导航中央 Match 是动作按钮，不是可切换页面；检查当前通话、金币/免费次数和权限，命中后打开 `RandomMatchAc` 再发起外呼。 | 位置和流程相同。 |
| 12 | Discover | 挂载 Popular/New/Following；Popular 呼叫恒定，New/Following 按免费资格切换，封面无轮播。 | 只挂载 Popular/Following，New 未接入；Popular 会为非 VIP 插入 Weekly VIP 商品卡。Discover 的轮播基础设施未接线，实际无轮播；Feed 卡片另有活动轮播。 |
| 13 | App 评分 | 远程开关开启且未展示过时，由首次关注、首次付费通话或特定免费通话触发，延迟约 3 秒。 | 规则相同。 |
| 14 | 5 秒禁挂 | `OutgoingAc` 与 `RandomMatchAc` 在 `PREPARING` 时禁用挂断 5 秒。 | 除 `OutgoingAc`、`RandomMatchAc` 外，`FakeIncomingAc` 转真实外呼后也有 5 秒禁挂。 |

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

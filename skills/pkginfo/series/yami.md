# Yami 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| Yami VIP | `siubun_yami` | `feature/vip` | `31a6568dfbbd` |
| Yamie | `siubun_yamie` | `feature/change` | `482b66014f9e` |
| Lite | `siubun_yamilite` | `feature/change` | `29d8b906be1b` |
| Pro | `siubun_yamipro` | `feature/change` | `d2a559285b86` |

## 系列关系

- Yamie、YamiLite、YamiPro 从 Yami 公共代码派生。
- Yami `main` 应作为公共上游，原则上可同步到三个衍生包，同时保留衍生包自己的配置、
  品牌资源和额外业务。
- Yami `feature/vip` 是特殊产品分支，不能合入三个衍生包。
- 历史共同同步点为 `2b23cd25`（2026-06-30）；之后 Yami main 曾新增支付商品分流、
  签到、Weekly VIP、商店状态和 UI 等公共改动。
- 本页 Yami 结论来自当前 `feature/vip`；并非重新走读 Yami `main` 后得出的行为。

## 检查结果

| # | 主题 | Yami `feature/vip` | Yamie / Lite / Pro |
|---|---|---|---|
| 1 | 通话前 VIP | Profile、相册、Popular、消息、通话记录、真实/假来电和匹配等付费入口会拦截非 VIP；但 New/Following 呼叫没有检查，中央 `CallManager` 也不兜底，存在绕过路径。 | 普通呼叫入口均不要求 VIP，中央入口只检查通话状态。 |
| 2 | Discover 商品 | VIP 用户显示未充值金币入口；非 VIP 显示 Weekly VIP 入口，并按注册剩余时间倒计时，到 0 隐藏。两个入口分别进入金币组合和 Weekly VIP 弹窗。 | 显示 badge 5 首充金币商品，仅未充值用户可见；点击首充组合弹窗，没有活动倒计时。 |
| 3 | 通话完成 | 非 VIP 弹 Weekly VIP；VIP 有金币时弹通话评价；VIP 无金币后按是否充值进入普通金币商店或 badge 5 首充弹窗，并独立计算 App 评分。 | 未购买金币且存在 badge 5 商品时弹首充，否则通话评价，再计算 App 评分。 |
| 4 | 商店 | VIP 与金币商品分流，支持订阅/Weekly VIP 和首充组合；VIP 状态参与商品与弹窗选择。 | 标准金币/VIP 分页商店，badge 3/5 使用推荐样式，首充按充值状态和注册时间过滤。 |
| 5 | Moment | 主导航 Moment 被隐藏，但 Profile 个人 Moment 子页仍可达。 | 主导航 Moment 与 Profile Moment 均活动可达。 |
| 6 | Feed | 没有独立 Feed 页面。 | 三个衍生包也没有。 |
| 7 | VIP 折扣价 | Profile、相册和 Outgoing/Incoming/FakeIncoming 都未应用 VIP 通话折扣。 | 同样不展示。 |
| 8 | 签到 | Discover 提供可见签到入口并打开签到弹窗。 | 当前没有活动签到入口。 |
| 9 | Mine 免打扰 | 一小时本地免打扰逻辑存在，可抑制假来电，但 Mine 对应布局固定隐藏，用户无法操作。 | 三个衍生包同样是逻辑存在、UI 隐藏。 |
| 10 | 国家筛选 | Popular 页水平国家条可用，选择后刷新推荐列表；没有 VIP 拦截，New/Following 不受影响。 | 国家筛选同样可用且无 VIP 门槛，具体呈现可能为图标/Dialog。 |
| 11 | 随机匹配 | 入口内嵌在 Popular 页，展示头像/匹配状态并进入匹配呼叫流程；匹配路径有 VIP 检查。 | 同样内嵌 Popular，但主要按免费次数和余额判断，不强制 VIP。 |
| 12 | Discover | 挂载 Popular/New/Following。Popular 使用多封面活动轮播且呼叫恒定；New/Following 按免费资格切换，并构成 VIP 门槛漏洞。 | 三个衍生包同样三 Tab，但封面静态无轮播；Popular 呼叫恒定，New/Following 按免费资格切换。 |
| 13 | App 评分 | 受远程开关和本地一次性标记控制，首次关注、首次付费通话或特定免费虚拟通话可触发，通常延迟约 3 秒。 | 三个衍生包规则相同。 |
| 14 | 5 秒禁挂 | Outgoing 进入 `PREPARING` 后禁用挂断 5 秒，匹配等相关过渡页也执行限制。 | 三个衍生包相同。 |

## VIP 分支特有内容

- Profile、相册、Popular、消息、通话记录、来电、假来电、匹配等入口检查 VIP。
- Discover 显示 Weekly VIP/金币双促销入口，并使用注册剩余时间倒计时。
- 通话完成按 VIP、金币余额和充值状态分流。
- 主 Moment 入口隐藏。
- Discover 签到入口与 Popular 轮播启用。

已知缺口：New、Following 列表的呼叫入口没有执行 VIP 检查。

## 同步边界

```text
Yami main
  ├─ 可作为 Yamie 公共上游
  ├─ 可作为 YamiLite 公共上游
  └─ 可作为 YamiPro 公共上游

Yami feature/vip
  └─ 特殊产品逻辑，不同步到衍生包
```

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`、`NewFrag*`、`FollowingFrag*`
- `ui/act/Profile*`、`MediaGallery*`
- `session/UserSessionContainer*`
- `viewmodel/Store*`、`VipVM*`
- `moment/MomentMain*`
- `rtc/view/Outgoing*`

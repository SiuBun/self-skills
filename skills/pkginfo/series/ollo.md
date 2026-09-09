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
5 秒禁挂；其 Discover 商品显示和点击当前都使用 badge 5，并不存在点击筛选更宽的问题。

## 检查结果

| # | 主题 | Ollo / Chat / Pro | OlloLite |
|---|---|---|---|
| 1 | 通话前 VIP | 只有 New 卡片对非 VIP 显示模糊遮罩并弹 `BecomeVipDia`；Popular、Following、Profile、相册等入口仍可呼叫，中央 `CallManager` 无 VIP 校验。 | 行为相同，因此也不是全局强制 VIP。 |
| 2 | Discover 商品 | 右下角 `clDiscount` 默认隐藏；查找 badge 5 首充金币商品，仅未充值用户显示，点击 `CoinsComboDia`。倒计时代码整段注释，不存在活动倒计时。 | 展示与点击都使用 `filterBadge5()`，当前行为与其他变体一致。 |
| 3 | 通话完成 | 未购买金币且存在 badge 5 首充商品时弹首充，否则弹通话评价，随后独立计算 App 评分。 | 首充与通话评价流程存在，但没有 App 评分链。 |
| 4 | 商店 | 金币/VIP 分页；badge 3/5 使用推荐样式，普通金币使用标准卡，首充受注册时间和充值状态控制。 | 基本相同，未发现影响购买流程的独立样式分叉。 |
| 5 | Moment | 主导航 Moment 和 Profile 个人 Moment 都活动可达。 | 相同。 |
| 6 | Feed | 没有独立 Feed 页面。 | 相同。 |
| 7 | VIP 折扣价 | Profile、相册和呼叫/来电页不应用 VIP 折扣，显示原始价格或免费状态。 | 同样不支持，且配置实体中没有 `vipCallDiscount` 字段。 |
| 8 | 签到 | Discover、Moment、Mine 都没有活动签到入口。 | 相同。 |
| 9 | Mine 免打扰 | Mine 没有一小时免打扰入口或假来电抑制链。 | 相同。 |
| 10 | 国家筛选 | Popular 页国家图标可打开标准 Dialog，选择后刷新 Popular；没有 VIP 拦截，也不作用于 New/Following。 | 相同。 |
| 11 | 随机匹配 | 唯一活动入口是 Popular 内嵌 `lltMatch`；检查余额、免费次数和权限后直接调用 `instantMatch()` 发起匹配。余额不足弹 `CoinsStoreDia`。`MatchFrag` 与 `RandomMatchAct` 有源码，但主导航没有挂载，属于不可达遗留。 | 相同。 |
| 12 | Discover | 挂载 Popular/New/Following，封面静态无轮播。Popular 呼叫按钮恒定；New/Following 按免费资格切换呼叫和消息，New 额外有非 VIP 模糊层。 | 相同。 |
| 13 | App 评分 | 受服务端开关控制，由首次关注、首次付费通话或特定免费通话触发，并使用本地标记防重复。 | **完全不存在评分 Dialog、远程开关访问器和触发链。** |
| 14 | 5 秒禁挂 | Outgoing 在 `PREPARING` 时禁用挂断按钮 5 秒。随机匹配走 Popular 内联链路，不能把遗留 `RandomMatchAct` 的代码计入可达行为。 | **没有该状态限制，外呼页可立即点击挂断。** |

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

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
| 1 | 通话前 VIP | 分支虽包含 VIP 商品逻辑，但普通外呼入口和中央 `CallManager` 没有统一 VIP 校验；非 VIP 仍能进入呼叫流程，VIP 主要影响商业弹窗而非使用资格。 | 三包一致。 |
| 2 | Discover 商品 | 非 VIP 显示套餐/订阅促销入口，商品链同时使用 badge 6 与订阅数据；入口订阅会话级 30 分钟倒计时，归零后重新开始，不会自动隐藏或使优惠失效。点击打开 `NewComboDia`。 | 三包一致。 |
| 3 | 通话完成 | 非 VIP 直接弹 Weekly VIP；VIP 且有金币时弹通话评价；VIP 无金币但已充值时进入普通金币商店；VIP 无金币且未充值时弹 badge 6 套餐。App 评分资格另行计算。 | 三包一致。 |
| 4 | 商店 | 使用金币和订阅混排的统一新商店。非 VIP 显示非订阅商品及第一个订阅商品；VIP 过滤订阅商品。badge 3/5 使用推荐大卡，订阅使用 VIP Bonus 卡。 | 三包一致，差异主要是包配置和资源。 |
| 5 | Moment | Moment 类和资源仍可能保留，但主导航入口与 Profile Moment 子页都处于隐藏/未挂载状态，用户当前无法进入。 | 三包一致。 |
| 6 | Feed | 没有独立可达 Feed 页面。 | 三包一致。 |
| 7 | VIP 折扣价 | Profile、相册和呼叫/来电过渡页均未应用 VIP 通话折扣，仍显示原始价格或免费状态。 | 三包一致。 |
| 8 | 签到 | Discover 提供可见签到入口，点击打开签到弹窗并读取签到进度；Mine/Moment 是否有入口以当前布局为准，本次确认的公共入口为 Discover。 | 三包一致。 |
| 9 | Mine 免打扰 | 本地一小时免打扰逻辑仍存在，可通过时间戳抑制假来电；但 Mine 对应容器固定隐藏，用户无法实际开关，不影响真实来电。 | 三包一致。 |
| 10 | 国家筛选 | Popular 页可打开顶部国家筛选 Dialog，选择后更新地区并刷新推荐列表；不对点击执行 VIP 拦截，New/Following 不使用该筛选。 | 三包一致。 |
| 11 | 随机匹配 | 点击入口进入独立全屏模糊匹配界面，页面展示匹配中、命中、失败和余额不足状态，命中后走专用过渡页再进入通话。 | 三包一致。 |
| 12 | Discover | 挂载 Popular、New、Following 三页。Popular 使用头像和相册资源进行活动轮播，呼叫按钮恒定；New/Following 根据免费资格在呼叫与消息之间切换。 | 三包一致。 |
| 13 | App 评分 | 受服务端开关控制；首次关注、首次付费通话和最后阶段免费虚拟通话可触发，并使用本地标记防止重复展示，通常延迟约 3 秒。 | 三包一致。 |
| 14 | 5 秒禁挂 | 普通 Outgoing 在 `PREPARING` 阶段禁用挂断 5 秒；随机匹配等相关呼叫过渡页也执行等价限制。 | 三包一致。 |

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

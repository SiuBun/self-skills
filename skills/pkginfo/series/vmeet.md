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
| 1 | 通话前 VIP | 普通呼叫入口和中央 `CallManager` 不要求 VIP，非 VIP 可发起付费通话。 | 同样没有全局门槛；Pro 的 VIP 只影响商品和结束分流。 |
| 2 | Discover 商品 | 右下角显示 badge 5 首充金币商品，仅未充值用户可见；点击首充组合弹窗，无活动倒计时。 | 非 VIP 显示第一项订阅商品，并展示会话级 30 分钟循环倒计时；VIP 隐藏入口，点击 `NewComboDia`。 |
| 3 | 通话完成 | 未购买金币且有 badge 5 商品时弹首充，否则弹通话评价，随后计算 App 评分。 | 非 VIP→Weekly VIP；VIP 有币→评价；VIP 无币时按充值状态进入金币商店或订阅套餐。 |
| 4 | 商店 | 金币/VIP 分页，badge 3/5 使用推荐样式，首充按充值状态和注册时间筛选。 | 统一新商店；非 VIP 显示非订阅和首个订阅，VIP 过滤订阅，订阅使用 VIP Bonus 卡。 |
| 5 | Moment | 主导航 Moment 与 Profile 个人 Moment 都可达。 | 主导航 Moment 隐藏，但 Profile Moment 仍实际挂载，可从个人页查看。 |
| 6 | Feed | 没有独立 Feed 页面。 | 同样没有。 |
| 7 | VIP 折扣价 | Profile、相册和呼叫/来电页不应用 VIP 折扣。 | 同样不展示 VIP 通话折扣价。 |
| 8 | 签到 | Discover、Moment、Mine 没有活动签到入口。 | Discover 提供签到入口并打开签到弹窗；其他入口以当前布局为准。 |
| 9 | Mine 免打扰 | 一小时本地免打扰代码存在并可抑制假来电，但 Mine 布局入口固定隐藏，用户无法操作。 | 相同。 |
| 10 | 国家筛选 | Popular 页国家筛选可用，选择后刷新 Popular；无 VIP 拦截，不影响 New/Following。 | 相同。 |
| 11 | 随机匹配 | 点击入口进入独立全屏匹配过渡页，命中后沿专用页面进入通话。 | 形态相同。 |
| 12 | Discover | 挂载 Popular/New/Following；Popular 呼叫恒定，New/Following 按免费资格切换；封面无轮播。 | 同样三 Tab，但 Popular 启用多封面轮播，并可插入 VIP 商品卡。 |
| 13 | App 评分 | 服务端开关开启且未展示过时，由首次关注、首次付费通话或特定免费通话触发，约 3 秒后显示。 | 规则相同。 |
| 14 | 5 秒禁挂 | Outgoing `PREPARING` 时禁用挂断 5 秒，匹配呼叫过渡页也保持限制。 | 相同。 |

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

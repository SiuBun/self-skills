# Vmeet 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD | 工作树 |
|---|---|---|---|---|
| 原包 | `siubun_vmeet` | `feature/change` | `075237359166` | clean |
| Lite | `siubun_vmeetlite` | `feature/change` | `b3c6daef6cf5` | `app/local/` 未跟踪 |
| Pro/VIP 商品版 | `siubun_vmeetpro` | `feature/vip_change` | `04c3ad9f2be4` | clean |

## 系列关系与差异

Vmeet 与 VmeetLite 是标准 badge 5 首充版本。VmeetPro 增加订阅促销、签到、
通话结束 VIP 分流和 Popular 轮播，但发起通话前仍不强制 VIP。

## 检查结果

| # | 主题 | Vmeet / Lite | VmeetPro |
|---|---|---|---|
| 1 | 通话前 VIP | 普通呼叫入口和中央 `CallManager` 不要求 VIP，非 VIP 可发起付费通话。 | 同样没有全局门槛；Pro 的 VIP 只影响商品和结束分流。 |
| 2 | Discover 商品 | 右下角显示 badge 5 首充金币商品，仅未充值用户可见；点击首充组合弹窗，无活动倒计时。 | 最终显隐条件只看 `wallet.isVip == false`，不再检查首充或充值状态；点击第一项订阅商品并打开 `NewComboDia`。入口使用会话启动时开始的 30 分钟循环倒计时。 |
| 3 | 通话完成 | 未购买金币且有 badge 5 商品时弹首充，否则弹通话评价，随后计算 App 评分。 | 非 VIP→Weekly VIP；VIP 有币→评价；VIP 无币时按充值状态进入金币商店或订阅套餐。 |
| 4 | 商店 | 金币/VIP 分页，badge 3/5 使用推荐样式，首充按充值状态和注册时间筛选。 | 统一新商店；非 VIP 显示非订阅和首个订阅，VIP 过滤订阅，订阅使用 VIP Bonus 卡。 |
| 5 | Moment | 主导航 Moment 与 Profile 个人 Moment 都可达。 | 主导航 Moment 隐藏，但 Profile Moment 仍实际挂载，可从个人页查看。 |
| 6 | Feed | 没有独立 Feed 页面。 | 同样没有。 |
| 7 | VIP 折扣价 | Profile、相册和呼叫/来电页不应用 VIP 折扣。 | 同样不展示 VIP 通话折扣价。 |
| 8 | 签到 | Discover、Moment、Mine 没有活动签到入口。 | Discover 右上和 Mine Weekly Reward 卡片均为活动入口并打开 `CheckInDia`；主 Moment 未挂载，因此没有可达 Moment 签到入口。 |
| 9 | Mine 免打扰 | 一小时本地免打扰代码存在并可抑制假来电，但 Mine 布局入口固定隐藏，用户无法操作。 | 相同。 |
| 10 | 国家筛选 | Popular 页国家筛选可用，选择后刷新 Popular；无 VIP 拦截，不影响 New/Following。 | 相同。 |
| 11 | 随机匹配 | 底部中央 MATCH 是动作按钮，不切换主页面；点击后匹配，命中后打开 `RandomMatchAc` 作为独立过渡页，再发起外呼。 | Main 不再拦截 MATCH；唯一活动入口改为 Popular 内嵌 `lltMatch`，检查余额和权限后直接 `instantMatch()`。余额不足弹 `NewStoreDia`；`RandomMatchAc` 类保留但没有启动点。 |
| 12 | Discover | 挂载 Popular/New/Following；Popular 呼叫恒定，New/Following 按免费资格切换；封面无轮播。 | 同样三 Tab，但 Popular 启用多封面轮播，并可插入 VIP 商品卡。 |
| 13 | App 评分 | 服务端开关开启且未展示过时，由首次关注、首次付费通话或特定免费通话触发，约 3 秒后显示。 | 规则相同。 |
| 14 | 5 秒禁挂 | 当前明确可达的 `OutgoingAc` 在 `PREPARING` 时禁用挂断 5 秒。 | `OutgoingAc` 同样启用；Popular 内联随机匹配不经过当前无调用方的 `RandomMatchAc`。 |

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

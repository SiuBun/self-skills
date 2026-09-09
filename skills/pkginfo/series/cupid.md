# Cupid 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD | 工作树 |
|---|---|---|---|---|
| 原包 | `siubun_cupid` | `feature/moment` | `4dc4942a965d` | `.idea` 与 `app/local/` 有未提交内容 |
| Lite | `siubun_cupidlite` | `feature/merge` | `89dcc0a015e4` | `app/local/` 未跟踪 |
| Plus | `siubun_cupidplus` | `merge` | `e4d61bff6f9b` | `.idea` 与 `app/local/` 有未提交内容 |

## 系列关系与差异

三个包在本次 14 项业务范围内基本一致，主要差异集中在品牌资源和商店配色：
Cupid/CupidLite 偏粉色，CupidPlus 偏紫色。未发现独立 VIP 分支业务。

## 检查结果

| # | 主题 | 三包公共行为 | 变体差异 |
|---|---|---|---|
| 1 | 通话前 VIP | 中央 `CallManager` 只检查是否已有通话；Popular、Following、Profile、相册、消息、通话记录和假来电转外呼均不要求 VIP。New 页例外：非 VIP 卡片被全覆盖 blur 遮罩，点击弹 `VipDialog`，属于页面级 VIP 门槛，但可从其他入口绕过。 | 三包一致。 |
| 2 | Discover 商品 | 右下角查找 `type == 0 && initialChargeRecommend == 1 && badge == "5"` 的首充金币商品；仅未充值用户显示，点击进入首充组合弹窗。布局虽有时间区域，但活动代码不递减，不能视为倒计时优惠。 | 三包一致。 |
| 3 | 通话完成 | 通话结束后等待页面稳定，再查 badge 5 首充金币商品；商品存在且用户没有购买过金币时弹首充组合，否则弹通话评价，之后独立计算 App 评分资格。 | 三包一致。 |
| 4 | 商店 | 金币和 VIP 商品分页面展示；普通金币使用标准卡片，badge 3/5 使用推荐大卡。badge 5 使用 5 分钟循环倒计时；badge 3 的专用倒计时分支已被注释，不能按仍有 24 小时活动倒计时记录。 | Cupid/CupidLite 偏粉色，CupidPlus 偏紫色，筛选流程相同。 |
| 5 | Moment | 主导航存在可达 Moment 页面，Profile 也挂载个人 Moment 子页；不是仅保留源码或隐藏入口。 | 三包一致。 |
| 6 | Feed | 没有独立 Feed 页面；Moment 时间流不作为单独 Feed 产品形态记录。 | 三包一致。 |
| 7 | VIP 折扣价 | Profile、相册、Outgoing、Incoming、FakeIncoming 均未应用 `vipCallDiscount`；页面只显示免费状态或主播原始价格。 | 三包一致。 |
| 8 | 签到 | Discover、Moment、Mine 均没有用户可见签到入口；仅存在交易枚举或接口痕迹不计为签到功能。 | 三包一致。 |
| 9 | Mine 免打扰 | 没有用户可见开关。虽保留 `DISTURB_SWITCH_OPEN_TIME` 常量和关闭假来电能力，但没有形成 Mine 开关写入时间、会话层读取并拦截假来电的完整活动链。 | 三包一致。 |
| 10 | 国家筛选 | Popular 页在国家列表非空时显示筛选图标；点击打开国家选择 Dialog，更新选中国家并刷新 Popular。New/Following 不随之过滤，也没有 VIP 拦截。 | 三包一致。 |
| 11 | 随机匹配 | 唯一可达入口是 Popular 底部带头像条的悬浮 pill/按钮。底栏 `MATCH` 固定隐藏；`MatchFragment`、`RandomMatchActivity`、`RandomMatchHistoryActivity` 虽有源码，但未被主导航或其他活动入口挂载。匹配在 Popular 内直接发起外呼。 | 三包一致。 |
| 12 | Discover | 实际挂载 Popular、New、Following 三页。Popular 呼叫按钮恒定显示，`Free` 标签才按免费资格变化；New/Following 仅免费可用时显示呼叫，否则显示消息/Say Hi。New 的非 VIP 卡片有全覆盖 blur。主播卡封面为静态单图，但随机匹配头像条另有每 5 秒自动滚动调度，不能把两者混为列表封面轮播。 | 三包一致。 |
| 13 | App 评分 | 服务端开关为 `y` 且尚未展示时，首次关注、首次付费通话或最后阶段免费虚拟通话可触发；通常延迟约 3 秒。正向评价进入 Google Play Review。 | 三包 Mine 头像均保留 Debug 手动触发入口。 |
| 14 | 5 秒禁挂 | `OutgoingActivity` 在 `PREPARING` 时禁用挂断 5 秒；`FakeIncomingActivity` 接听并转为真实外呼、收到 `OnOutgoing` 后也禁用 5 秒。Popular 内联随机匹配不经过独立 Outgoing 页面，不受这两处 UI 锁定保护。 | 三包一致。 |

## 关键证据

- `discover/DiscoverFrag*`、`PopularFrag*`、`NewFragment*`
- `res/layout/view_bottom_tab.xml`、`match/MatchFragment*`
- `ui/adapter/Popular*Adapter*`、`New*Adapter*`、`Following*Adapter*`
- `session/UserSessionContainer*`
- `ui/act/Profile*`、`moment/MomentMain*`
- `rtc/view/Outgoing*`、`FakeIncoming*`
- `viewmodel/Store*`、`entity/StoreGoods*`

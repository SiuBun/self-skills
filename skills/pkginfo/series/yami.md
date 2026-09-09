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
| 1 | 通话前 VIP | 多入口拦截，但 New/Following 可绕过 | 无 |
| 2 | Discover 商品 | VIP/金币双入口，VIP 注册剩余时间倒计时 | badge 5 首充金币，无活动倒计时 |
| 3 | 通话完成 | VIP、余额、充值状态分流 | badge 5 首充弹窗或评价 |
| 4 | 商店 | VIP/金币商品分流 | 标准金币/VIP 商店 |
| 5 | Moment | Main 隐藏，Profile Moment 保留 | Main 与 Profile Moment 均启用 |
| 6 | Feed | 无 | 无 |
| 7 | VIP 折扣价 | 无 | 无 |
| 8 | 签到 | Discover 有 | 无 |
| 9 | Mine 免打扰 | 一小时逻辑存在但 UI 隐藏 | 相同 |
| 10 | 国家筛选 | 水平国家条可用，无 VIP 拦截 | 可用，无 VIP 拦截 |
| 11 | 随机匹配 | Popular 内嵌 | Popular 内嵌 |
| 12 | Discover | 三 Tab；Popular 有活动轮播 | 三 Tab；无轮播 |
| 13 | App 评分 | 有 | 有 |
| 14 | 5 秒禁挂 | 有 | 有 |

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

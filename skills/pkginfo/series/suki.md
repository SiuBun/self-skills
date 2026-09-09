# Suki 系列

> 走读时间：2026-09-09
> 会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`

## 仓库快照

| 变体 | 仓库 | 分支 | HEAD |
|---|---|---|---|
| Suki VIP 工作分支 | `siubun_suki-opt` | `change/new-main-vip` | `4a9617dc8879` |
| Lite | `siubun_sukilite-opt` | `feature/change` | `5820425e88d4` |
| Plus/公共上游 | `siubun_sukiplus-opt` | `feature/merge` | `9ceb75a226ad` |
| Pro | `siubun_sukipro-opt` | `feature/change` | `5f09368fd0c8` |

## 系列关系

- SukiLite 和 SukiPro 从 SukiPlus 派生，Plus 是更合理的公共上游。
- Suki 从 Plus 的共同内容继续演进，并从提交 `b7d38b50` 开始进入 VIP 模式。
- Suki 曾把完整线上 VIP 版本保存在 `feature/vip`，再在 `main` 移除 VIP 内容。
- 历史检查发现 `feature/vip` 是 `main` 的祖先，直接在 VIP 分支合并 main 会
  fast-forward 到去 VIP 版本，不符合“只解决公共代码冲突”的目标。
- 期望拓扑应保持：`sukiplus/main → suki/main → suki VIP 分支`。
- 本页业务结论针对当前检出的 `change/new-main-vip`，不是对 Suki `main` 的重新审计。

## 检查结果

| # | 主题 | Suki VIP 工作分支 | Lite / Plus / Pro |
|---|---|---|---|
| 1 | 通话前 VIP | 多数付费入口拦截；Discover 三个列表仍可绕过 | 仅随机匹配完成后的“继续匹配”检查 VIP |
| 2 | Discover 商品 | VIP/金币双入口；非 VIP 的 badge 3 VIP 商品按注册剩余时间倒计时 | badge 5 首充金币，无活动倒计时 |
| 3 | 通话完成 | VIP、余额、充值状态分流 | badge 5 首充弹窗或评价；随机匹配后有完成页 |
| 4 | 商店 | VIP/金币分开，支持 badge 3 注册时限 | 标准金币/VIP 商店；Pro 视觉有粉紫渐变 |
| 5 | Moment | Main 与 Profile Moment | 相同 |
| 6 | Feed | 无 | 无 |
| 7 | VIP 折扣价 | 无 | 无 |
| 8 | 签到 | 无 | 无 |
| 9 | Mine 免打扰 | 无 | 无 |
| 10 | 国家筛选 | 水平国家条可用，无 VIP 拦截 | 相同 |
| 11 | 随机匹配 | 独立 Match Tab；无免费次数时检查 VIP | 独立 Match Tab；主要检查余额/金币 |
| 12 | Discover | Popular/New/Following；无轮播 | 相同 |
| 13 | App 评分 | 有 | 有 |
| 14 | 5 秒禁挂 | 有 | 有 |

## VIP 分支特有内容

- 多页面付费呼叫 VIP 拦截。
- Discover VIP/金币双促销入口。
- badge 3 VIP 新用户注册倒计时。
- 通话结束按 VIP、余额、充值状态分流。

当前已知缺口：Popular/New/Following 的卡片呼叫没有执行同样的 VIP 检查。

## 关键证据

- `ui/act/Profile*`、`MediaGallery*`
- `discover/DiscoverFrag*`、`PopularFrag*`、`NewFrag*`、`FollowingFrag*`
- `match/RandomMatch*`
- `session/UserSessionContainer*`
- `viewmodel/Store*`、`VipVM*`
- `rtc/view/Outgoing*`

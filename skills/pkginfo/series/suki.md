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
- 旧线曾把完整线上 VIP 版本保存在 `feature/vip`，再通过
  `25141566 update:回滚变vip模式前内容` 形成去 VIP 的 `feature/merge`。
- 新线以 `change/new-main-change` 承载公共内容，以 `change/new-main-vip` 承载
  VIP 内容。
- 期望长期拓扑应保持：
  `sukiplus/main → change/new-main-change → change/new-main-vip`。
- 本页业务结论针对当前检出的 `change/new-main-vip`，不是对 Suki `main` 的重新审计。

## 四个分支的定位

| 分支 | HEAD | 日期 | 定位 | 关键事实 |
|---|---|---|---|---|
| `feature/vip` | `3b99402afae0` | 2026-08-24 | 旧 VIP 线上版本 | 包含从 `b7d38b50` 开始的 VIP 迭代，不包含新 Plus 公共基线 `8bad61f7` |
| `feature/merge` | `daaa619bd380` | 2026-08-24 | 旧去 VIP/合并实验线 | 以 `feature/vip` 为祖先，通过 `25141566` 回滚 13 个 VIP 核心文件，再合并当时 main |
| `change/new-main-change` | `454a9f4bfb07` | 2026-09-09 | 新公共/非 VIP 基线 | 包含 Plus 公共提交 `8bad61f7`，不包含 `b7d38b50` 和旧 VIP 历史 |
| `change/new-main-vip` | `4a9617dc8879` | 2026-09-09 | 新 VIP 工作线 | 第一父提交是 `change/new-main-change`，第二父提交是 `feature/vip`；同时包含新公共基线和旧 VIP 历史 |

### 当前提交拓扑

```text
旧公共历史
├─ feature/vip (3b99402a，旧 VIP)
│  ├─ feature/merge (daaa619b，回滚 VIP 后的旧非 VIP 线)
│  └─ change/new-main-vip (4a9617dc，新 VIP 线的第二父来源)
└─ change/new-main-change (454a9f4b，新公共线)
   └─ change/new-main-vip (4a9617dc，新 VIP 线的第一父)
```

当前关键祖先关系：

- `change/new-main-change` 是 `change/new-main-vip` 的祖先；后者在 Git 历史上多出
  45 个可达提交，主要来自合入的旧 VIP 历史。
- `feature/vip` 是 `feature/merge` 的祖先，后者多出 3 个提交。
- `feature/vip` 也是 `change/new-main-vip` 的祖先，后者多出 69 个提交。
- `feature/merge` 与 `change/new-main-change` 已经分叉，双方分别有 47 和 68 个独有提交，
  不应再把 `feature/merge` 当作新公共基线。

### 标志提交归属

| 提交 | 含义 | new-main-change | new-main-vip | feature/merge | feature/vip |
|---|---|---|---|---|---|
| `8bad61f7` | 新 Plus 公共同步点 | 有 | 有 | 无 | 无 |
| `b7d38b50` | 开始 VIP 拦截迭代 | 无 | 有 | 有，历史中保留 | 有 |
| `25141566` | 回滚到变 VIP 前内容 | 无 | 无 | 有 | 无 |

## 两代分支之间的实际差异

### 旧线：`feature/vip` 与 `feature/merge`

`feature/merge` 相对 `feature/vip` 修改 13 个文件，约 `+114/-219`。这些文件集中在：

- Profile、相册、消息、通话记录、匹配、来电和假来电的 VIP 拦截。
- Discover VIP/金币双入口及 VIP 倒计时。
- 通话完成后的 VIP/金币分流。
- 随机匹配完成后的 VIP 判断。

因此旧 `feature/merge` 的主要作用确实是从旧 VIP 版本中移除活动 VIP 逻辑。
但它仍包含 VIP 提交的 Git 历史，只是工作树内容被回滚，不能根据祖先关系判断当前行为。

### 新线：`change/new-main-change` 与 `change/new-main-vip`

当前新 VIP 分支相对新公共分支：

- 69 个文件发生变化，约 `+476/-227`。
- 其中 35 个是 Kotlin/XML/Gradle/字符串等代码或布局文件，其余主要是图片和多语言资源。
- 差异不只限于旧线回滚涉及的 13 个文件。

主要业务差异：

1. 新增 Profile、相册、消息、通话记录、匹配历史、真实来电、假来电、
   随机匹配完成页等入口的 VIP 拦截。
2. Match 页在免费次数耗尽且非 VIP 时弹 VIP，引导 VIP 用户继续进入余额判断。
3. Discover 从单一 badge 5 金币入口变为：
   - VIP 用户看未充值金币入口。
   - 非 VIP 用户看 badge 3 VIP 入口。
   - VIP 商品按注册剩余时间倒计时，到期隐藏。
4. 通话完成从“未购买首充→badge 5，否则评价”改为 VIP、余额、充值状态分流。
5. 新增 VIP 商店权益项和 VIP 促销资源。

同时还存在并非纯 VIP 开关的技术差异：

- `StoreBadgeViewModel` 的商品模型由 `StoreGoodsV2` 改回 `StoreGoods`。
- `PaymentRepository.filterBadge5()` 被移除。
- 登录 API 的 `password` 参数被移除。
- `FakeIncomingActivity`、新用户引导、Billing、随机匹配错误处理也有额外变化。

这些差异来自把旧 `feature/vip` 合入新公共线后的冲突处理和历史实现带入，不能全部视为
“VIP 必需改动”。未来继续收敛时，应逐项判断是否需要回归 `change/new-main-change`
的新公共实现。

## 分支使用建议

| 目标 | 应使用的分支 |
|---|---|
| 公共、非 VIP、准备继续同步 Plus | `change/new-main-change` |
| 当前 VIP 产品开发 | `change/new-main-vip` |
| 查询旧 VIP 实现历史 | `feature/vip` |
| 查询旧版去 VIP 回滚方式 | `feature/merge` |

- 后续公共改动先进入 `change/new-main-change`，再合入 `change/new-main-vip`。
- 当前拓扑下，从 `change/new-main-vip` 合并 `change/new-main-change` 不会再出现
  “直接 fast-forward 成去 VIP 版本”的旧问题。
- `feature/vip` 和 `feature/merge` 应视为历史参考线，不再承担新代码同步职责。
- 合并后不能只检查冲突是否解决，还要复查上述 69 个差异文件中的非 VIP 技术改动。

## 检查结果

| # | 主题 | Suki VIP 工作分支 | Lite / Plus / Pro |
|---|---|---|---|
| 1 | 通话前 VIP | Profile、相册、消息、通话记录、真实/假来电、匹配历史和继续匹配等付费路径会拦截非 VIP；但 Popular/New/Following 卡片呼叫没有相同检查，中央 `CallManager` 也不兜底，仍可绕过。 | 普通呼叫入口不强制 VIP；主要只有随机匹配完成页的“继续匹配”检查 VIP。 |
| 2 | Discover 商品 | VIP 用户隐藏 VIP 入口，只在未充值且有 badge 5 商品时显示金币入口；非 VIP 隐藏金币入口，显示 `type == 1 && badge == "3"` 的 VIP 商品，按注册剩余时间倒计时，到 0 隐藏。 | 显示 badge 5 首充金币商品，仅未充值时可见；没有活动促销倒计时。 |
| 3 | 通话完成 | 非 VIP 弹 VIP 引导；VIP 有金币时弹通话评价；VIP 无金币按是否充值进入普通金币商店或 badge 5 首充弹窗，并独立判断 App 评分。 | 未购买金币且有 badge 5 时弹首充，否则通话评价；随机匹配通话还会进入完成页。 |
| 4 | 商店 | VIP 与金币商店分开；支持 badge 3 新用户注册时限和 badge 5 推荐样式，VIP 商店增加专属权益说明。 | 使用标准金币/VIP 商店，badge 3/5 采用推荐样式；SukiPro 倒计时区域使用粉紫渐变。 |
| 5 | Moment | 主导航 Moment 与 Profile Moment 均活动可达。 | 三个衍生包也都可达。 |
| 6 | Feed | 没有独立 Feed 页面。 | 相同。 |
| 7 | VIP 折扣价 | Profile、相册、Outgoing、Incoming、FakeIncoming 都没有展示 VIP 通话折扣价。 | 相同。 |
| 8 | 签到 | Discover、Moment、Mine 均没有活动签到入口。 | 相同。 |
| 9 | Mine 免打扰 | Mine 没有一小时免打扰开关，也不通过本地时间抑制假来电。 | 相同。 |
| 10 | 国家筛选 | Popular 页水平国家条可用，点击更新地区并刷新 Popular；没有 VIP 拦截，New/Following 不受筛选。 | 行为相同。 |
| 11 | 随机匹配 | 随机匹配是独立主导航 Match Tab；免费次数耗尽且非 VIP 时弹 VIP，VIP 再进入金币/余额判断。 | 同样是独立 Match Tab，但主要按免费次数和金币判断；部分“继续匹配”动作才检查 VIP。 |
| 12 | Discover | 挂载 Popular/New/Following，列表为静态单封面，无活动轮播。Popular 呼叫恒定；New/Following 按免费资格切换。VIP 工作分支的三个列表呼叫仍是门槛漏洞。 | 三包页面结构和按钮规则相同。 |
| 13 | App 评分 | 受服务端开关和本地一次性标记控制，首次关注、首次付费通话或特定免费虚拟通话可触发，约 3 秒延迟。 | 三包规则相同。 |
| 14 | 5 秒禁挂 | Outgoing 进入 `PREPARING` 后禁用挂断 5 秒，匹配相关过渡页也执行限制。 | 三包相同。 |

## VIP 分支特有内容

- 多页面付费呼叫 VIP 拦截。
- Discover VIP/金币双促销入口。
- badge 3 VIP 新用户注册倒计时。
- 通话结束按 VIP、余额、充值状态分流。

当前已知缺口：Popular/New/Following 的卡片呼叫没有执行同样的 VIP 检查。

这里的“VIP 分支特有内容”特指当前 `change/new-main-vip` 相对
`change/new-main-change` 的活动业务，不应与旧 `feature/vip` 完全等同。

## 关键证据

- `ui/act/Profile*`、`MediaGallery*`
- `discover/DiscoverFrag*`、`PopularFrag*`、`NewFrag*`、`FollowingFrag*`
- `match/RandomMatch*`
- `session/UserSessionContainer*`
- `viewmodel/Store*`、`VipVM*`
- `rtc/view/Outgoing*`

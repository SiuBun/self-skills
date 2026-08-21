---
name: fastcall-vip-migrate-api37
description: 在目标 VIP 同源项目已经完成 fastcall-migrate-api37 基础迁移后，迁移 siubun_fastcall feature/vip_migrate_api 对匹配入口、充值弹窗、VIP 动态解锁和底部导航的专属适配时使用。
---

# Fastcall `vip_migrate_api` 差异迁移

## 使用前提

本 skill 只处理 VIP 分支在基础迁移之上的差异，不能单独执行。

必须先完成：

```text
fastcall-migrate-api37
```

基础完成标准至少包括：

- 配置重建和 ViewBinding 生命周期治理；
- 可恢复 Dialog/支付流程；
- RTC 页面重建；
- 统一呼叫权限入口；
- `CallManager` 唯一状态源与 `OnPickup` 唯一最终导航；
- Target API 37、`ProcessLifecycleOwner` 和通话前台服务。

如果目标 VIP 项目尚未完成基础迁移，停止本 skill，先执行基础 skill。

## 源实现

源仓库：

```text
/Users/jason/AndroidStudioProjects/siubun_fastcall
```

源分支：

```text
feature/vip_migrate_api
```

分支关系：

```text
feature/migrate_api@503ddc98
  -> merge commit 8aa93556
  -> VIP delta 3e3db201d8e87d6ed659987d9274117734c23231
```

`2af5ff5b` 只增加提测文档，不属于 VIP 代码差异。

本 skill 的代码证据只使用：

```text
8aa93556..3e3db201
```

涉及四个文件：

- `discover/PopularFrag.kt`
- `ui/act/MainAc.kt`
- `ui/act/MomentDetailAc.kt`
- `res/layout/view_bottom_tab.xml`

提交中包含 import 排序和格式化噪音，只迁移下述业务语义。

## 目标项目识别

VIP 同源项目可能只对类名增加或删除少量字符。按以下职责搜索：

- Popular/Discover 主播墙或发现页中的匹配按钮；
- 主页的随机匹配 ViewModel、权限请求和余额不足充值弹窗；
- 动态详情中的 VIP 解锁 Dialog；
- 底部导航中的 Match Tab；
- `CallOrigin.MATCH_RECOMMEND`、`matchNavigation`、`randomMatch()`；
- VIP 周卡/订阅弹窗和新金币商店弹窗。

找不到同名类时，阅读布局 ID、点击事件、ViewModel 和 Dialog 调用链确认等价文件。

## VIP 差异 1：发现页承担匹配入口

源语义：

- VIP 版本在 `PopularFrag`/发现主播墙保留匹配入口；
- Fragment 持有等价于 `RandomMatchVM` 的页面 ViewModel；
- Fragment 自己注册 `CallPermissionRequester`；
- 在 `viewLifecycleOwner` 的 `RESUMED` 阶段消费
  `matchNavigation.filterNotNull()`；
- 使用目标用户、`CallOrigin.MATCH_RECOMMEND` 和免费通话状态调用统一
  `requestOutgoing()`；
- 成功提交给权限协调后按 navigation ID 消费目标；
- 匹配按钮调用 `randomMatch()`，不走已废弃的 `instantMatch()`；
- 主播墙直接呼叫也必须走统一 `requestOutgoing()`，不能调用旧的
  `launchOutgoingActivity()` 或底层 `outgoingApi()`。

实施时确认：

- 目标 VIP 项目的真实匹配入口是否在 Popular、Discover 或同职责 Fragment；
- `matchNavigation` 可以跨 Fragment 配置重建保留；
- 只在 `RESUMED` 消费，避免新 Fragment 尚未可交互时启动权限；
- 消费顺序不会导致权限流程丢失或重复外呼；
- Fragment 重建不会新增第二个匹配请求。

## VIP 差异 2：主页兼容隐藏 Match Tab

源语义：

- VIP 底部导航隐藏独立 Match Tab：`clt_match` 运行时为 `gone`；
- 匹配功能仍从 VIP 实际入口进入，不能因为隐藏 Tab 删除 ViewModel 或业务能力；
- `MainAc` 中若仍存在匹配入口/恢复链，继续使用统一 `CallPermissionRequester` 和
  `matchNavigation`；
- 余额不足时使用 VIP 版本的新商店 Dialog（源项目为 `NewStoreDia`），不使用基础版本
  的 `CoinsStoreDia`；
- Dialog 查重和展示必须使用同一个 VIP Dialog TAG。

不要机械复制源布局 ID。目标项目若 Match Tab 名称不同，应隐藏其等价容器；若目标 VIP
产品仍明确显示 Match Tab，则保留产品行为，只迁移权限和重建安全部分，并记录差异。

不要为了隐藏 Tab 删除其相邻约束，需验证剩余底部导航项没有空白、偏移或点击区域残留。

## VIP 差异 3：动态详情使用 VIP 解锁弹窗

源语义：

- 基础分支动态详情使用普通 VIP 引导 Dialog；
- VIP 分支使用周期/VIP 专属 Dialog（源项目为 `WeeklyVipDia`）；
- `findFragmentByTag()`、`create()` 和 `show(..., TAG)` 必须全部使用同一种 Dialog；
- 仍由页面 ViewModel 保存 `VipPrompt`，展示后按 prompt ID 消费；
- 旋转后不能重复显示两个 Dialog，也不能丢失可靠解锁提示。

目标类名可能不同。根据“动态付费内容解锁”调用参数确认：

- `liveSource`；
- `paywallSource`；
- `paywallSourceId`。

不要只替换 TAG 而保留另一种 Dialog 实例。

## VIP 差异 4：主页合并后的冲突清理

`3e3db201` 中 `MainAc` 和 `PopularFrag` 有部分 import、缩进和 merge 冲突清理。这些不是
迁移目标。只处理：

- 匹配目标的可靠消费；
- 统一呼叫权限入口；
- VIP 余额不足弹窗；
- VIP 实际匹配入口；
- 隐藏的 Match Tab。

保留目标项目已有的：

- Tab 数量与顺序；
- VIP 业务入口；
- 埋点事件和参数；
- 免费通话判定；
- flavor 差异；
- 页面命名和资源风格。

## 防重复检查

VIP 差异完成后搜索并确认：

- `randomMatch()` 只由实际用户入口触发；
- 不再调用目标项目旧的 `instantMatch()`；
- 匹配命中后只调用一次统一 `requestOutgoing()`；
- `CallOrigin.MATCH_RECOMMEND` 没有绕过权限协调；
- Popular/Discover 与 Main 不会同时消费同一个共享 pending navigation；
- `consumeMatchNavigation(id)` 在成功交给权限流程后调用一次；
- 余额不足只显示一个 VIP 商店 Dialog；
- 动态详情的 Dialog 类型和 TAG 完全一致；
- 隐藏 Match Tab 后布局没有残留空位；
- `VideoCallAc` 或目标最终通话页仍只有会话协调层一个启动入口。

## 验证

在基础 skill 的编译基础上，再覆盖 VIP 专属场景：

1. 主页底部导航不显示 Match Tab，其他 Tab 排列和点击正常；
2. 从发现/主播墙触发随机匹配；
3. 匹配请求期间旋转，确认请求不重复；
4. 匹配命中后在权限 Dialog、系统权限页和去电阶段旋转；
5. 最终只发起一次外呼、只进入一个通话页；
6. 无余额时只显示一个 VIP 新商店 Dialog；
7. 动态详情点击付费内容，显示正确的 VIP 周期弹窗；
8. VIP 弹窗显示期间旋转，确认继续操作且不重复；
9. 普通主播墙直接呼叫仍经过相机、麦克风和通知权限；
10. 通知允许/拒绝及 RTC 前后台行为与基础迁移一致。

至少编译目标 VIP 项目的主要 Debug 变体；若存在 Google/Release 专属 VIP 逻辑，追加对应
变体。

## 提交方式

- 在基础迁移提交之后单独提交 VIP 差异；
- 提交说明明确“依赖 fastcall-migrate-api37 基础迁移”；
- 不混入基础迁移遗漏修复；若发现基础缺口，先回到基础阶段补齐并验证；
- 最终列出目标 VIP 项目与源项目之间刻意保留的产品差异。


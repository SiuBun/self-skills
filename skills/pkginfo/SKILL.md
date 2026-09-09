---
name: pkginfo
description: 走读、比较、同步或更新 ~/AndroidStudioProjects 下 siubun_* Android 包及其 Main、Lite、Pro、Plus、VIP 变体的业务差异时使用。覆盖 VIP 通话门槛、Discover、通话结束、商店、Moment/Feed、折扣价、签到、免打扰、随机匹配、评分和挂断限制。
---

# Siubun 多包业务知识

用于回答和维护 `~/AndroidStudioProjects` 下 `siubun_*` Android 包的产品行为差异。

## 使用流程

1. 先读取 [`CHECKLIST.md`](CHECKLIST.md)，确认本次需要检查的主题。
2. 根据仓库名称读取对应的 [`series/`](series/) 文件，不要默认加载所有系列。
3. 结论必须绑定到文件中记录的分支和 HEAD；分支或提交变化后先重新走读。
4. 比较 Main、Lite、Pro、Plus、Chat、U、E 或 VIP 时，区分：
   - 公共上游能力。
   - 包级额外逻辑。
   - 品牌、资源和配置差异。
   - VIP 分支特有逻辑。
5. 只把活动代码和可达入口计入结论；注释、废弃类和静态隐藏入口单独说明。
6. 新结论先更新系列文件；涉及检查口径变化时再更新 `CHECKLIST.md`。

## 系列索引

| 系列 | 仓库 | 文档 |
|---|---|---|
| Cupid | Cupid、CupidLite、CupidPlus | [`series/cupid.md`](series/cupid.md) |
| Fastcall | Fastcall、FastcallLite、FastcallPro | [`series/fastcall.md`](series/fastcall.md) |
| Lumi | Lemie、Lumi、LumiLite、LumiPro | [`series/lumi.md`](series/lumi.md) |
| Matchat | Matchat、MatchatLite、MatchatPlus、MatchatPro | [`series/matchat.md`](series/matchat.md) |
| Mitalk | Mitalk、MitalkLite、MitalkPro、MitalkU | [`series/mitalk.md`](series/mitalk.md) |
| Mojoin | Mojoin | [`series/mojoin.md`](series/mojoin.md) |
| Ollo | Ollo、OlloChat、OlloLite、OlloPro | [`series/ollo.md`](series/ollo.md) |
| Suki | Suki、SukiLite、SukiPlus、SukiPro | [`series/suki.md`](series/suki.md) |
| Vmeet | Vmeet、VmeetLite、VmeetPro | [`series/vmeet.md`](series/vmeet.md) |
| Yami | Yami、Yamie、YamiLite、YamiPro | [`series/yami.md`](series/yami.md) |

## 公共判断口径

- 中央 `CallManager` 未校验 VIP 时，页面级检查不能称为全局强制。
- Popular 呼叫图标恒定显示不代表免费，也不代表已通过 VIP 检查。
- `ratingDialogSwitch == "y"` 只是评分弹窗的总开关，还需满足业务触发条件。
- Mine 免打扰通常只抑制假来电，不等同 IM 会话免打扰。
- 30 分钟倒计时若到期自动重置，应记录为循环营销计时，而不是优惠失效时间。
- `type == 0` 通常为金币，`type == 1` 通常为 VIP/订阅；
  `initialChargeRecommend == 1` 通常为首充推荐。
- badge 常见语义：3 新用户、5 首充/Best Value、6 套餐首充；最终以当前包代码为准。

## 审计底稿

[`references/audit-2026-09-09.md`](references/audit-2026-09-09.md) 保存首次完整走读结果、
34 个仓库的分支/HEAD、跨系列规则和证据文件索引。

首次审计会话 ID：`9b275586-4f98-464c-98a5-a90472a410fc`。

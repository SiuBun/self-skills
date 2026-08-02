# Compose 性能、测试与发布质量

## 一、性能判断原则

静态审查只能发现明显风险，不能证明页面流畅或卡顿。结论分为：

- **确定问题**：不稳定 key、主线程阻塞、资源未释放、错误 Effect、无界图片等。
- **待验证风险**：参数稳定性、重组范围、自定义绘制、复杂动画、嵌套布局。
- **实测结论**：由 Release 真机、Profiler、Macrobenchmark 或系统 Trace 支持。

不要仅凭 Debug 构建或肉眼滑动宣称性能合格。

## 二、重组

- 状态尽量在最低需要层级读取。
- 高频状态不要在页面根部直接读取。
- 仅需要布尔派生结果时使用 `derivedStateOf`。
- 需要从 Compose State 生成 Flow 时使用 `snapshotFlow`。
- 集合映射和格式化可移入 ViewModel，或按输入使用 `remember` 缓存。
- 不为轻量计算滥用 `remember`；先确认计算成本和重组频率。
- 使用不可变 UI 模型，避免把可变集合暴露给 Composable。
- Lambda 和对象稳定性问题必须结合 Compose Compiler Metrics 判断，不凭猜测大规模添加 `@Stable`。

## 三、Lazy 列表

- key 使用稳定、唯一、跨刷新不变的业务 ID。
- 禁止正常业务 key 拼接 index、随机数或默认 `hashCode()`。
- 无业务 ID 时应推动数据模型提供稳定本地 ID，而不是长期用 index 兜底。
- 混合类型列表设置 `contentType`，帮助复用同类型 item。
- 分页使用 Paging 的 `itemKey`/`itemContentType` 或等价稳定实现。
- 点击回调传业务对象或 ID，不依赖 Composition 中手动递增的局部 index。
- 避免同方向 Lazy 容器无约束嵌套。
- 大列表 item 不做同步 I/O、复杂 JSON、Bitmap 解码或高成本日期格式化。

## 四、图片与媒体

- 图片按最终显示尺寸请求和解码。
- 列表缩略图不加载无界原图。
- 提供占位、错误和内容缩放策略。
- 图片请求 key 与业务资源稳定对应。
- Media3 Player、PlayerView、监听器和 Surface 必须有明确拥有者和释放路径。
- `AndroidView` 的 `factory` 负责创建，`update` 只同步变化属性；需要回收时使用相应 release 机制。

## 五、Effect 与动画

- Effect key 变化频率必须与任务重启成本匹配。
- 无限循环动画或轮播在离开 Composition 后必须自动取消。
- 滚动监听使用 `snapshotFlow`，必要时 `distinctUntilChanged`。
- 动画方向、初始值和反向行为都要测试，不能只验证默认分支。
- 自定义绘制中缓存 Brush、Path 或计算结果，避免每帧重复创建。

## 六、测试金字塔

### 单元测试

覆盖：

- ViewModel/Presenter 状态迁移。
- Reducer。
- 错误、重试和旧数据保留。
- PagingSource 或 Repository 业务映射。

### Compose UI 测试

覆盖：

- 状态到语义树的映射。
- 点击、输入、滚动和弹窗。
- Loading、Empty、Error、Content。
- 无障碍语义和关键 testTag。

### 截图测试

适用于：

- 设计系统组件。
- 视觉密集页面。
- Light/Dark。
- 字体缩放。
- Compact/Expanded。

截图测试负责视觉回归，不替代行为测试。

### 端到端测试

覆盖登录、支付、聊天、媒体等关键业务路径，数量少但稳定。

## 七、Macrobenchmark 与 Baseline Profile

优先建立：

- 冷启动、温启动。
- 首页首帧和可交互时间。
- 核心列表滚动。
- 页面跳转和复杂动画。

至少比较无编译、Baseline Profile 和完整编译模式。Baseline Profile 应由基准模块生成并随 Release 构建更新。

## 八、质量门槛

每个 Compose 改动至少运行：

1. 受影响变体 Kotlin 编译。
2. Android Lint，重点关注 Compose Runtime/UI 规则。
3. 相关单元或 UI 测试。
4. 视觉变化的 Preview 或截图验证。

性能敏感改动增加：

1. Release 真机验证。
2. Macrobenchmark 或系统 Trace。
3. Compose Compiler Metrics/Reports。

Lint Baseline 只能用于管理存量问题，不应隐藏新增 Compose 错误。

## 九、发布

- Release 开启 R8 和资源压缩。
- Debug、Nightly/Beta、Release 行为差异显式配置。
- 日志、Crash、Analytics、Billing 和测试入口按变体隔离。
- CI 至少包含编译、Lint、单元测试和关键截图。
- 成熟项目增加 Macrobenchmark、Baseline Profile、端到端测试和发布物校验。

# View 到 Compose 渐进迁移与互操作

## 一、中期迁移的正确目标

迁移中期允许：

- Compose Activity 与旧 Activity 并存。
- Fragment 外壳内逐步替换 Compose 区域。
- Compose 使用 `AndroidView` 承载暂未迁移的平台 View。
- 旧 Dialog、播放器、RTC、WebView 和系统组件继续工作。

中期检查的目标不是清零 XML，而是确认新代码没有扩散错误的状态、主题、生命周期和性能模式。

## 二、候选选择

优先迁移：

- 边界清晰、依赖少的静态组件。
- 状态输入明确的列表 item 或完整页面。
- 有截图或设计稿可验证的页面。
- 不依赖复杂平台生命周期的功能。

后迁移：

- RTC、播放器、Camera、WebView。
- 高度依赖 Fragment result、Activity result 或复杂返回栈的页面。
- 缺少测试且业务风险高的核心交易流程。

## 三、Compose 放入 View

Activity 完整迁移时可使用 `setContent`。

Fragment 或 XML 中使用 `ComposeView` 时：

- Fragment View 使用 `DisposeOnViewTreeLifecycleDestroyed` 或与宿主 View 生命周期等价的策略。
- 多个 ComposeView 必须有唯一 ID，保证 saved state。
- RecyclerView 池化容器使用适配池化释放语义的策略。
- 所有入口使用 App Theme。
- View 销毁后不得保留 Compose 相关回调、监听或宿主引用。

避免把页面拆成大量零散 ComposeView；跨 ComposeView 不能共享 Composition，增加状态和性能成本。

## 四、View 放入 Compose

使用 `AndroidView` 时：

- `factory` 只负责创建和一次性初始化。
- `update` 根据参数同步属性，不重复注册监听或重建昂贵对象。
- 监听器使用最新回调，避免捕获过期状态。
- 播放器、WebView 等明确暂停、恢复、释放和状态保存。
- 主题依赖 View 通过正确 Context/Theme 创建。
- 列表复用场景考虑重置旧状态。

如果 View 已稳定且重写收益低，可以长期保留 Adapter，而不是为了纯度重写。

## 五、状态边界

- Compose 与 View 观察同一个 ViewModel/状态源，不互相复制状态。
- View 事件转换为业务 Action，Compose 事件也进入相同入口。
- 不让 XML View 和 Compose 分别维护同一页签、选中项或加载状态。
- 迁移过程中先稳定状态边界，再替换渲染层。

## 六、主题迁移

- 同时维护 XML Theme 和 Compose Theme，但使用同一组产品 Token。
- 每迁移一个组件，确认颜色、字体、shape、间距和状态样式来源。
- Light/Dark 两套主题同时验证。
- 不在 Compose 中写一套近似颜色，导致迁移后视觉逐页漂移。
- 完成全部迁移并确认旧 Theme 无引用后再删除 XML 主题。

## 七、导航迁移

- 明确旧 Activity/Fragment 跳转和 Compose Navigation 的边界。
- Compose Screen 通过回调请求打开旧页面，宿主负责执行 Intent 或 Fragment 导航。
- 旧页面返回结果时转换为状态或明确事件。
- 避免同一业务同时维护两套返回栈事实。
- 深链、登录拦截和进程重建在迁移前后保持一致。

## 八、每个页面的迁移完成标准

1. 视觉与交互保持一致，或有明确产品批准的变更。
2. Loading、Empty、Content、Error、离线和重试完整。
3. 旋转、后台恢复和进程重建行为明确。
4. 返回栈、深链和页面结果正确。
5. 系统栏、IME 和窗口 inset 不重复。
6. TalkBack、字体缩放和触控目标可用。
7. Preview 和至少一个关键 UI 测试存在。
8. 列表 key、图片尺寸和滚动性能经过检查。
9. Compose/View 资源和监听有对称释放。
10. 旧 XML、Activity/Fragment、Adapter、资源和测试仅在无引用后删除。

## 九、中期架构检查

继续扩大迁移前确认：

- 已统一 Route/Screen 或等价页面结构。
- 已统一 UiState、Action/Event 和错误模型。
- 已建立可用的 Compose Theme 与语义 Token。
- 已统一 Navigation route 和参数写法。
- 已统一 Flow 收集、Effect 和 Scope 规则。
- 已建立最小 Compose UI 测试和截图回归。
- 已为关键启动与列表路径规划 Macrobenchmark/Baseline Profile。

如果这些规则没有稳定，优先治理规范再批量迁移二级页面，否则技术债会随页面数量线性扩散。

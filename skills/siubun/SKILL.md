---
name: siubun
description: 按 siubun_fastcall 项目既有的 Kotlin、MVI、Session、XML ConstraintLayout、RecyclerView 和 Paging 规范实现或修改 Android 功能时使用。新增页面、ViewModel、Adapter、分页列表、会话相关能力或 UI 时必须使用。
---

# SiuBun Android 编码规范

生成代码前先在仓库搜索相同职责的现有实现，并复用其模式、工具扩展和资源命名。不要因方便而新建另一套模板、DI 框架、状态管理方式或分页抽象。

## 实现前的检查顺序

1. 找到同类型的 Activity、Fragment、ViewModel、Adapter 或布局作为直接参考。
2. 读取关联的 binding、字符串、颜色、字体、drawable、repository 及接口定义。
3. 新增会话能力时先确定其生命周期是应用级还是登录会话级。
4. 只在现有模式确实无法满足时才新增公共抽象；说明原因并保持 API 最小化。

## Kotlin 与界面层

- 使用 ViewBinding。Activity 实现 `createViewBinding()`；Fragment 使用对应的基类和 `createViewBinding()`。
- Activity/Fragment 只负责绑定视图、收集状态、导航和将用户操作转发给 ViewModel；业务请求和状态计算放在 ViewModel 或 Repository。
- 协程收集使用项目的 `launchWhenCreated`、`launchWhenResumed` 等扩展，确保其内部生命周期重复收集语义生效；不要在界面层创建脱离生命周期的 scope。
- 点击使用已有的 `safeClick`；不要直接重复实现防抖。
- 资源一律使用 `@string`、`@color`、`@font`、`@drawable` 等引用，避免硬编码用户可见文本和可复用颜色。
- 保持仓库代码风格：不可变状态用 `copy` / `update`，多行参数保留尾随逗号，明确命名，只有复杂或重要的生命周期逻辑才写注释。

## MVI 与 ViewModel

- ViewModel 内定义 `UiState` 和/或 `UiEvent`。持续可渲染状态使用私有 `MutableStateFlow` 对外暴露 `asStateFlow()`；一次性导航、弹窗和错误效果使用私有 `MutableSharedFlow` 对外暴露 `asSharedFlow()`。
- 事件与状态保持明确：加载、空数据、错误、重试和成功应有可观察状态或事件，不能静默吞掉失败。
- 使用 `viewModelScope` 执行业务操作。请求失败时按现有页面模式更新加载状态并发射重试/错误事件。
- 页面依赖通过 `by viewModels { ViewModel.Factory(...) }` 传入；沿用嵌套 `Factory` 与未知类型抛出 `IllegalArgumentException` 的模式。
- Repository 是薄的数据/API 适配层，ViewModel 可直接依赖 Repository；不要默认再加 UseCase 层。
- 需要由多条流计算 UI 时使用 `combine(...).stateIn(...)`，避免在界面层拼装业务状态。

## Session 与依赖生命周期

- `AppContainer` 持有应用级单例、`applicationScope` 和未登录也可用的 Repository。
- `UserSessionManager` 是登录态的唯一来源；使用其 `sessionState` 判断 `Loading`、`LoggedOut`、`LoggedIn`。
- 与账号、token 或登录后资源绑定的 Repository、Manager、网络客户端和收集任务放入 `UserSessionContainer`，并使用其 `sessionScope`。
- 通过 `UserSessionManager` 暴露的可空服务获取登录态依赖，并在界面进入时用 `whenLoggedIn` 或状态收集进行门控；不能假设用户已登录。
- 新增会话级服务时，必须在 `UserSessionContainer.clearSessionServices()` 中释放资源、取消任务、清理引用，确保登出后不会继续工作。
- 登录、登出、删除账号均由 `UserSessionManager` 串行管理；不要在功能代码中绕过它直接持久化或清理登录数据。

## XML 与 ConstraintLayout

- 优先使用 `ConstraintLayout` 表达相对关系；为每个控件建立完整且清晰的约束，避免依赖偶然位置。
- 宽度受相邻元素限制时使用 `0dp`、约束和 `layout_constrainedWidth`；文本设置合理的 `maxLines`、`ellipsize`。
- 使用 `tools:` 属性提供预览文本、可见性和预览背景，不改变运行时行为。
- 固定视觉尺寸可以用 `dp`；跨页面重复的尺寸新增或复用 `@dimen`。
- 字体使用 `app/src/main/res/font/` 的本地 Poppins 资源。容器背景使用原始颜色资源；语义文本颜色资源只能用于文本。
- 需要运行时圆角或背景时使用 `ViewShapeEx.kt` 的扩展方法，并让 XML 背景和运行时背景保持一致。
- 处理沉浸式页面时，先阅读窗口 inset 逻辑。Figma 含 Bottom Bar 时，不将 Bottom Bar 高度加进内容 padding。

## Dialog、Bottom Sheet 与键盘

- 居中弹窗继承 `BaseDialogFragment`，参考 `CoinsComboDia`：基类会禁用返回键和点击外部取消、设置透明 Window 背景并在启动时设为全屏宽度。除非产品明确要求可取消，否则不要覆盖此行为。
- 居中弹窗使用可空 `_binding` 与非空 `binding` getter，必须在 `onDestroyView()` 将 `_binding = null`；与 View 绑定的 Job、回调或循环也必须在此取消。
- 底部弹窗直接继承 `BottomSheetDialogFragment`，以 `SelectLanguageDia`、`GiftDia` 为主参考。`onViewCreated()` 中将 `view.parent` 背景设为透明，避免顶部圆角露出 Bottom Sheet 默认白底；使用 `setTopCornersShape()` 设置根视图顶部圆角，并设置滑动条和内部容器形状。
- 底部弹窗通过 `companion object` 提供带固定 `TAG` 的 `show` / `create` 方法；参数优先写入 `arguments`。只有无法序列化的短生命周期回调才以属性传入，并在弹窗销毁后不再依赖它。
- Bottom Sheet 需要避让键盘或系统 inset 时，在 `onStart()` 对 `dialog.window` 调用 `WindowCompat.setDecorFitsSystemWindows(window, false)`，在 `window.decorView` 设置 `ViewCompat.setOnApplyWindowInsetsListener`，读取 `WindowInsetsCompat.Type.ime()`，将 `binding.root` 的 bottom margin 更新为 `imeInsets.bottom` 后请求布局，并返回原始 `insets`。没有键盘/系统栏避让需求时不添加无效监听。
- 控制输入框键盘显示与隐藏时复用 `ViewExt.kt` 的 `toggleIme(window, editText, show)`；显示前会聚焦输入框，隐藏时会清除焦点。判断键盘可见性使用 `View.isKeyboardVisible()`，不要通过固定屏幕高度推测。
- 弹窗中的状态、事件和 Flow 收集仍遵循 ViewModel 与生命周期规则；关闭使用 `dismissAllowingStateLoss()`，并保留 `safeClick` 防抖。

## RecyclerView、DiffUtil 与 Paging

- 简单列表使用 `ListAdapter`，分页列表使用 `PagingDataAdapter`；ViewHolder 使用生成的 Item Binding。
- Adapter 保持薄：负责 inflate、bind、点击回调和必要的局部 UI 刷新；数据获取、业务判断和跨项状态由 ViewModel 管理。
- `DiffUtil.ItemCallback` 必须使用稳定业务 ID 判断同一项，使用完整数据判断内容变更；只有确有收益时实现 `getChangePayload()` 并在 payload bind 中只更新受影响控件。
- 点击回调应在 bind 时取 `bindingAdapterPosition`，检查 `RecyclerView.NO_POSITION` 后再次读取当前项，避免异步列表更新时操作旧数据。
- 分页实现以 `PopularPagingAdapter` 与 `PopularFrag` 为唯一首选模板。`app/src/main/java/com/fastcall/livechat/paging/` 下的 `FooterAdapter`、`PagingFooterState` 等旧分页 UI 状态实现已过时，新增功能不得参考或接入它们；仅维护既有调用时才修改。
- Paging 页面以 `collectLatest` 收集 `pagingFlow` 并调用 `submitData()`；通过 `adapter.loadStateFlow` 的 `refresh` 状态驱动下拉刷新、首屏加载、错误和空态。
- 下拉刷新及错误态重试统一调用 `adapter.refresh()`。不要额外接入 Header/Footer Adapter，也不要用空白 item 伪造加载状态。
- 卡片宽高比依赖 RecyclerView 最终列宽时，沿用 `PopularPagingAdapter` 的 `itemView.post` 后按比例更新高度的方式，不在 bind 前假设 item 宽度。

## 代码生成完成前

1. 对比参考实现，确认没有重复工具或绕过已有生命周期管理。
2. 检查新增的资源引用、绑定 ID、会话清理和错误路径。
3. 运行最小覆盖范围的现有 Gradle 编译或测试命令。当前 Debug Kotlin 编译变体为：

```sh
./gradlew :app:compileLocalDebugKotlin :app:compileGoogleDebugKotlin
```

## 维护此技能

新增规则必须来自已验证且可复用的项目实现。写明适用条件和必要动作；若后续实现推翻规则，及时更新或删除。技能修改应与证明该规则的代码变更一并提交。

## 已验证参考

- `session/AppContainer.kt`：应用级依赖与 `applicationScope`。
- `session/UserSessionManager.kt`：会话状态、登录、恢复与登出串行管理。
- `session/UserSessionContainer.kt`：会话级服务及登出清理。
- `viewmodel/LoginVM.kt`：ViewModel 事件流、加载状态和 Factory。
- `ui/frag/BaseDialogFragment.kt`、`ui/dialog/CoinsComboDia.kt`：居中弹窗基类、ViewBinding 与资源清理。
- `ui/dialog/SelectLanguageDia.kt`、`ui/dialog/GiftDia.kt`：Bottom Sheet 的透明父层、顶部圆角、展示入口和列表组织。
- `utils/ViewExt.kt`：`toggleIme` 与键盘可见性判断；`ui/dialog/SelectLanguageDia.kt`：Bottom Sheet 的 IME inset 避让。
- `discover/PopularFrag.kt`：Paging 收集、LoadState、下拉刷新、错误重试与生命周期收集。
- `ui/adapter/PopularPagingAdapter.kt`：当前标准的 Paging Adapter、DiffUtil、payload、安全点击和动态卡片比例。
- `res/layout/item_anchor_wall.xml`：ConstraintLayout、文本约束和预览属性。

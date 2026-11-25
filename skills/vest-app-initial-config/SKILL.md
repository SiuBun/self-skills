---
name: vest-app-initial-config
description: 从主仓检出代码制作马甲包后，做首次品牌/标识配置修改：应用名、包名、Adjust key、scheme、日志 tag、rootProject.name、Firebase 配置。仅用于刚检出、同源、尚未改品牌的工程的初始配置；项目名每次可能不同。
---

# 马甲包初始配置

适用：从某个主仓（项目名每次不同，范例为 Circle → CircleLite）检出的同源工程，做首次品牌/标识替换。只改品牌字符串，Kotlin 包路径与 `namespace` **不改**，保持同源便于同步主仓。

基线规则：以当前工程检出时的状态为基线，先读各文件当前值再改，不假设具体项目名。若工程里已有他包的值，除非用户明确要求，只读不改。

## 开始前由用户提供（缺则索取，不猜）
应用名、包名、Adjust key 一一对应，由用户在检出项目时确定：
- `APP_NAME`
- 正式包 applicationId
- `ADJUST_KEY`

测试包名：默认 `正式包名 + "test"`，但**必须向用户确认**后再改。

以下项按当前工程现有命名风格派生，先列给用户看，用户指定则以用户为准。后缀 = 正式包名相对主仓正式包名多出的部分：
- `app_scheme`：沿用主仓 scheme 写法并拼接后缀小写（范例：`circle` + `lite` → `circlelite`）
- 日志 tag 与 `rootProject.name`：主仓名 + 后缀首字母大写（范例：`Circle` + `Lite` → `CircleLite`）

Firebase 与正式 `google_server_client_id`：用户有真实配置就用真实的；暂时没有时，两者都必须替换为同步的临时占位值（见第 6 步），不能继续沿用主仓的正式 OAuth ID。占位配置仅供初始配置及构建检查，不能用于真实 Firebase 或 Google 登录，发布前须替换为该包的真实配置。

## 修改清单（范例为 Circle → CircleLite 的实际改动）
1. `app/build.gradle.kts`（共 3 处）：`defaultConfig.applicationId` 与 flavor `local` 用测试包名；flavor `google` 用正式包名。
2. `app/src/main/res/values/values.xml`（测试）：`app_name`=`<APP_NAME> Test`、`app_id`=测试包名、`adjust_key`、`app_scheme`。
3. `app/src/google/res/values/values.xml`（正式）：`app_name`、`app_id`=正式包名、`adjust_key`、`app_scheme`、`google_server_client_id`。
4. 启动初始化里的日志 tag（范例：`startup/XLogInitializer.kt` 的 `.tag(...)`）。
5. `settings.gradle.kts`：`rootProject.name`（必改项，不要漏）。
6. Firebase：
   - google flavor 的 `google-services.json`：有真实文件则整体替换；否则临时占位——把 `project_number` 改一两位数字，并在全文件同步：`mobilesdk_app_id` 中的 project_number、所有 `client_id` 的数字前缀（包括嵌套的 OAuth 配置）、所有 `package_name`=正式包名。正式资源中的 `google_server_client_id` 必须同步替换为其中 `client_type: 3` 的临时 client_id，不能只修改 JSON 而遗漏资源。
   - `app/google-services.json`（local）：仅 `package_name` 改为测试包名；测试资源中原有的 `google_server_client_id=0` 保持不变。
7. 不在范围，保持原样：`api_url`、`tba_url`、各类 key、`app_name_vip`、`icon_app_name` 等图标、多语言文案里的品牌词；除非用户另行要求。
8. `gradle/gradle-daemon-jvm.properties` 是 IDE 生成物，不属于改名范围，不提交。

若新工程目录结构与上述路径不同，先按 `applicationId`、`app_name`、`adjust_key`、`app_scheme`、`rootProject.name`、`google-services.json` 全局检索定位，再对应修改。

## 流程
1. 读取上述文件当前值与 `git status`。
2. 向用户确认缺失输入（测试包名等），并展示完整“旧值 → 新值”计划。
3. 用户确认后再修改；用户说“先讨论/只说怎么改”时只列计划，不改代码。
4. 收尾校验：grep 旧包名、旧 adjust key、旧 project_number、旧 scheme、旧 tag 无遗留；两份 google-services.json 的 `package_name` 与对应 flavor 的 applicationId 一致；可运行 `./gradlew :app:assembleGoogleDebug`。

## 提交边界
- Android 工程改动默认不暂存、不提交、不推送，保留工作区供用户自行检查并提交；不能因为用户允许修改，就视为允许提交。仅用户另行明确要求提交工程时例外。
- 用户要求更新本 skill 时，同步修改 self-skills 仓库中的定义与本地安装版本；skill 定义更新单独提交并普通推送到已核实的 self-skills 远端，不与 Android 工程混在一起。
- 提交 skill 前检查 self-skills 仓库、分支、远端和暂存区；仅纳入本轮 skill 文件，不提交或覆盖用户已有的其他修改。缺少仓库或目标不明确时先询问，不编造路径、仓库或分支，不强推。

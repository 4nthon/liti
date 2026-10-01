# 砺题 (Liti) · 考研政治背诵刷题 App

一款离线优先的考研政治背诵 / 刷题 Android 应用，基于 **Kotlin + Jetpack Compose（MVVM）** 构建。
围绕「逐句背诵 → 填空式选择题 → 错题反复攻克」的学习闭环设计，全部数据本地存储，无需任何服务端。

> 本仓库为**开源版**：内置一份小规模示例题库（马原基础概念，仅演示结构与流程），
> 正式题库请按 [题库导入](#题库导入) 一节用自己的资料生成。

![图片](https://github.com/4nthon/liti/blob/main/image.png?raw=true)

## 功能特性

- **三级导航**：科目 → Day → 论述题，论述题点进即答，无多余步骤
- **App 内题库导入**：设置页选择 DayN.txt 直接导入，自动解析 / 校验 / 落盘，重启即生效
- **双答题模式**（设置页全局切换，默认背题）：
  - **背题模式**：点选项立即判分，答对 0.8 秒自动进入下一题，答错停留看解析并自动加入错题本
  - **测验模式**：点选项只做标记可改选，交卷后统一判分
- **错题本**：以选择题为单位记条目；对所在论述题整题连续答对满 3 次，该论述题下的错题条目全部消灭
- **完成确认**：一轮作答完成时若有未作答题，弹窗提示返回作答或强制完成（未作答按答错计入错题本）
- **进度持久化与断点续答**：答题中途中出/杀进程，重启后题位、答题卡、计时原样接回
- **历史统计**：按日账本（答题数/正确率/用时）、累计统计、打卡日历（当月视图）
- **收藏**：单题收藏、收藏重练
- **明暗双主题**、选项乱序防记忆、每日励志句

## 环境要求

- Android Studio（或命令行 Gradle 8.7 + AGP 8.5.2）
- JDK 17
- Android SDK 34
- 最低支持 Android 8.0（minSdk 26）

## 构建运行

```bash
# Android Studio 直接打开项目根目录，Sync 后 Run 即可；
# 或命令行：
./gradlew assembleDebug   # 若仓库无 wrapper，请用本机 Gradle 8.7：gradle assembleDebug
```

> `local.properties`（SDK 路径）由 Android Studio 自动生成，不入库；
> 命令行构建时请确认本机已配置 `ANDROID_HOME` 或已生成该文件。

## 签名

仓库**不包含任何签名密钥**。不配置时，release 包使用 debug 签名，可直接安装体验。
要出正式签名包，在本机 `gradle.properties`（勿提交）或环境变量中配置：

```properties
LITI_STORE_FILE=path/to/your.jks
LITI_STORE_PASSWORD=xxx
LITI_KEY_ALIAS=xxx
LITI_KEY_PASSWORD=xxx
```

## 题库导入

### 方式一：App 内直接导入（推荐）

**设置 → 题库 → 导入题库**，选择一个或多个 `DayN.txt` 即可。App 会自动解析、统计并在
确认后重启加载新题库；「恢复示例题库」可一键还原内置示例。

txt 书写格式（宽容解析，兼容多种写法）：

```
【唯物辩证法】                    ← 小标题 = 论述题
1、唯物辩证法认为发展的实质是____。 ← 题号 + 题干（空位用 ____ 或【 】）
A. 事物的永恒运动                 ← 选项（A、A. A： 均可）
B. 新事物的产生和旧事物的灭亡
C. …
D. …
答案：B                          ← 或「正确答案：B」
解析：发展的实质是新事物的产生和旧事物的灭亡。 ← 缺解析时自动用正确答案回填题干
```

- 多空题选项内用 ` \ `、` / ` 或 `／` 分隔各空，App 统一为 ` / ` 渲染
- 文件名含 `DayN` 时 Day 名取 N；UTF-8 / GBK 编码自动识别
- 同一论述题按小标题分组；进度主键为**内容派生 ID**（科目|Day|论述题|题干指纹），
  不含数组下标——重新导入、增删题目都不影响已有答题记录

### 方式二：构建时打包

题目数据来自 `app/src/main/assets/qbank.tsv`（行式 TSV），由 `tools/import_qbank.py`
从题目 txt 生成。完整流程、体检校验见 **[tools/README.md](tools/README.md)**。

```
tools/qbank_src/DayN.txt  →  tools/import_qbank.py  →  app/src/main/assets/qbank.tsv
```

- 题干 = 原文连续截取，空位用 `____` 标记
- 解析 = 题干把空填回答案后的原句（App 内解析即背诵原文）

## 项目结构

```
app/src/main/java/com/gao/recite/
├─ vm/AppVm.kt          # 单 ViewModel：全部业务逻辑（答题/判分/错题/统计/持久化）
├─ data/
│  ├─ Bank.kt           # 题库装载（优先 filesDir 导入库，否则 assets 示例）
│  ├─ QbankParser.kt    # App 内导入：DayN.txt 宽容解析 → TSV（与 tools 脚本同规则）
│  ├─ Data.kt           # 数据模型 + 演示条目（收藏/薄弱专题/励志句）
│  └─ Persist.kt        # 进度落盘（filesDir/progress.tsv，原子写）
├─ ui/
│  ├─ screen/           # 首页/题库/统计/设置/答题/结果 各页
│  ├─ components/       # 通用组件
│  └─ theme/            # 调色板与字阶（明暗双主题）
└─ MainActivity.kt
tools/
├─ import_qbank.py      # 题目 txt → qbank.tsv
├─ check_answers.py     # 答案键 vs 原文 全量校验
└─ qbank_src/           # 示例题库源 txt
```

## 许可证

[MIT](LICENSE)

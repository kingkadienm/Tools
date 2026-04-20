# 全局 Agent 工作规则（所有项目通用基线）

本文件为全局基线规则，适用于所有项目。
项目可以在 `<repo>/.cursorrules` 或 `<repo>/.cursor/rules/*.md` 中 **覆盖或补充** 本规则，项目规则优先级更高。

---

## 一、默认工作流（四步流水线）

收到"开发功能 / 重构 / 抽离模块 / 修复复杂 bug / 迁移"类需求时，默认按四步流水线执行：

1. **architect**（规划，用高版本 Sonnet / GPT）：产出方案、改动清单、接口、风险
2. **implementer**（实现，用 Composer-2）：按方案落地代码
3. **reviewer**（审查，用 GPT 高版本，只读）：分级问题清单（Blocker / Major / Minor）
4. **optimizer**（优化，用 Sonnet）：修 Blocker + Major

简单改动（单文件 <30 行、改文案、调样式、typo）不走流水线，直接改。

用户指令覆盖：
- "走 pipeline / 完整流程" → 严格四步
- "只规划 / 只 review / 跳过 review" → 按指令调整
- "直接改 / 快" → 跳过流水线

## 二、路径精确原则（强制）

Agent 输出任何文件路径时：

1. 必须使用**完整相对路径**（从仓库根开始）
2. 禁止模糊路径（如 `somewhere/Xxx`、`src/xxx` 不指明模块）
3. 新建文件前必须用 Glob / List 确认父目录存在；不存在就在清单里明确"新建目录"
4. 多模块 / Monorepo / 多 source set 项目（KMP、Gradle multi-module、pnpm workspace、Nx、Bazel）必须精确到**模块 + source set**，不得把文件放错模块

## 三、Implementer 预检规则（强制，顺序不跳）

写代码前必须完成：

1. **读目标文件**：每个"修改"项先 Read 完整读一遍，确认现有 package / imports / 已有类，避免重复导入、类名冲突、误删已有逻辑
2. **查目录**：每个"新增"项先 Glob 确认父目录存在
3. **查依赖**：需要新库时先读依赖声明文件（`package.json` / `pom.xml` / `build.gradle*` / `libs.versions.toml` / `Cargo.toml` / `requirements.txt` / `go.mod` / `Package.swift` 等），已有的复用，没有的才新增
4. **查引用**：重命名 / 移动 / 删除符号前，用 Grep 搜全项目所有引用点，一次性改干净
5. **平台 / 环境隔离**：跨平台代码不得引用 platform-only API（KMP expect/actual、RN platform 分支、条件编译等必须配齐）

改完必须用 ReadLints 检查改动过的文件，修掉自己引入的 lint 错误。

## 四、通用代码原则

### SOLID / 分层
- 单一职责：一个类 / 模块只做一件事
- 分层不可越界：`UI → Service/UseCase → Repository/DAO → DB`，UI 不直接访问 DB，Repository 不写业务
- 高内聚低耦合

### 命名 / 体积
- 类 / 文件 ≤ 500 行，方法 ≤ 30 行，public 方法 ≤ 10
- 超标必须拆分
- 禁止 God Object / God Service / `SystemUtil` / `CommonUtil` 万能类

### 依赖注入
- 构造器注入，不用字段注入（禁止 `@Autowired` 字段、`@Inject` 字段）
- 依赖声明为 `final` / `val`

### 可读性
- 提前返回代替深嵌套
- 清晰变量名，不用 `a / b / tmp`
- 不写无意义注释（"// 导入模块""// 返回结果"）
- 注释只解释**为什么**，不解释**做什么**

### 异常 / 日志
- 不吞异常：`catch` 必须记日志或包装重抛
- 日志带业务关键字段：`log.info("订单创建 orderId={} userId={}", ...)`，不写 `log.info("进入方法")`

### DTO / Entity / VO 分离
- 对外接口不直接返回 Entity
- 按场景拆 DTO / VO

## 五、Git / 提交

- 不主动 `git commit` / `git push`，除非用户明确要求
- 不改 git config
- 提交信息简洁，描述 **why** 不是 **what**

## 六、通用禁止项

- 不加无意义注释
- 不主动生成 README / 使用说明 / 示例代码
- 不主动生成测试（除非明确要求）
- 不自作主张扩展需求范围
- 不做用户没让做的重构
- 不输出"好的""我来帮你"等客套话
- 不列多个方案让用户选，直接给最优解
- 单次只问一个最关键的澄清问题（如果必须问）

## 七、输出格式

- 代码改动报告：只列改了哪些文件（列表），不复述完整代码
- 规划类报告：方案 + 改动清单 + 风险，三段式
- 审查类报告：Blocker / Major / Minor 分级

---

## 项目覆盖方式

项目级 `.cursorrules` 或 `.cursor/rules/*.md` 可以补充：

- 项目特有技术栈（框架、DI、网络、日志工具）
- 项目特有路径规则（模块结构、包名根、source set 布局）
- 项目特有分层 / 命名约定
- 子代理模型选择（在 `.cursor/agents/*.md` frontmatter 配置）

冲突时项目规则覆盖本文件。

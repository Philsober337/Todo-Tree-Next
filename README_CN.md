# Todo Tree_Next

**Todo Tree_Next** 是对经典 VS Code Todo Tree 插件的现代化重写。它保留原有树视图、高亮、过滤、跳转等使用习惯，同时加入 Rust 高性能扫描器、TypeScript 模块化结构、Git 感知任务视图，以及面向 AI Code 工具的 Agent 接口。

插件在 VS Code Marketplace 上发布名称为：**Todo Tree_Next**。

## 亮点

| 能力 | 当前实现 |
| --- | --- |
| 扫描器 | Rust 原生扫描器，并保留 ripgrep 回退 |
| 刷新方式 | 保存/打开文件时进行文件级增量扫描 |
| 元数据 | `P0`-`P3`、`TODO!`、`TODO?`、`@负责人`、`due:YYYY-MM-DD`、`#标签` |
| Markdown | 原生识别 `- [ ]`、`- [x]`、编号任务 |
| 过滤 | 支持 `tag:TODO path:src priority:P0` 等结构化查询 |
| Git | 扫描 changed/staged 文件，导出 TODO 债务报告 |
| 仪表盘 | Webview 展示统计、图表、扫描器控制和 Git 操作 |
| AI Agent | 向 AI Code 工具暴露结构化 TODO 上下文，并支持编辑器标注 |

## 安装

在 VS Code 扩展市场搜索：

```text
Todo Tree_Next
```

也可以手动安装 `.vsix`：

1. 打开 VS Code 扩展面板。
2. 点击 `...` > `Install from VSIX...`。
3. 选择生成的 `todo-tree-*.vsix` 安装包。

## 核心工作流

打开 Todo Tree 活动栏视图后，可以按文件树、标签或扁平列表查看 TODO。点击条目即可跳转到源码位置。插件支持工作区扫描、打开文件扫描、当前文件扫描，以及保存文件后的增量刷新。

常用命令：

```text
Todo Tree: Refresh
Todo Tree: Open Dashboard
Todo Tree: Scan Changed Files
Todo Tree: Scan Staged Files
Todo Tree: Export TODO Debt Report
Todo Tree: Get Agent TODO Context
Todo Tree: Clear Agent Annotations
```

## 智能过滤

树视图过滤支持普通文本和字段查询：

```text
auth
tag:TODO
path:src
file:README.md
text:refactor
priority:P0
status:open
tag:FIXME path:src priority:P1
```

支持字段：

| 字段 | 匹配内容 |
| --- | --- |
| `tag` | `TODO`、`FIXME`、`BUG`、`[ ]`、`[x]`、自定义标签 |
| `path` | 完整文件路径 |
| `file` | 文件名 |
| `text` | TODO 文本 |
| `priority` | `P0`、`P1`、`P2`、`P3`、`none` |
| `status` | Markdown 任务状态：`open` 或 `done` |

## 优先级与元数据

```javascript
// TODO:P0 修复认证漏洞 @alice due:2026-06-01 #security
// FIXME:P1 内存泄漏 @bob #backend
// TODO! 紧急任务      -> P0
// TODO? 需要讨论      -> P2
```

Rust 扫描器会把这些信息提取为结构化字段，供树视图、仪表盘、导出报告和 AI Agent 接口使用。

## 仪表盘

通过命令打开：

```text
Todo Tree: Open Dashboard
```

仪表盘提供：

- TODO 总数和标签分布
- 优先级分布
- 标签图表和趋势图
- 扫描器引擎切换：`auto`、`rust`、`ripgrep`
- 扫描模式控制
- 最大文件大小配置
- 智能过滤输入
- Git changed/staged 扫描操作

## Git 集成

```text
Todo Tree: Scan Changed Files
Todo Tree: Scan Staged Files
Todo Tree: Export TODO Debt Report
```

Git 感知扫描可以聚焦当前分支引入或触碰到的 TODO。债务报告会将当前分支和基准分支进行对比，统计新增和删除的 TODO。

## AI Agent 接口

Todo Tree_Next 会把 TODO 技术债暴露为 AI Code 工具可消费的结构化上下文。Agent 可以读取排序后的 TODO 数据，也可以把审查结果、风险提示、修复建议写回 VS Code 临时标注。

VS Code 命令接口：

```javascript
const context = await vscode.commands.executeCommand('todo-tree.getAgentContext');

await vscode.commands.executeCommand('todo-tree.annotateAgentFinding', {
  file: 'src/auth.ts',
  line: 42,
  column: 5,
  severity: 'warning',
  message: 'P0 TODO 涉及认证逻辑，合并前需要审查。'
});

await vscode.commands.executeCommand('todo-tree.clearAgentAnnotations');
```

Rust CLI：

```bash
todo-scanner agent-context --root . --config todo-scanner-config.json
```

Agent 上下文包含：文件路径、行列号、标签、优先级、负责人、截止日期、labels、Git 状态、近似历史年龄、上下文代码片段、推荐动作和推荐处理顺序。

完整协议见 [docs/AGENT_INTERFACE.md](docs/AGENT_INTERFACE.md)。

## 架构

```text
VS Code 插件层
  src/extension.js          插件入口和旧逻辑粘合层
  src/scannerClient.ts      Rust CLI JSON 协议
  src/agentInterface.ts     AI Agent 上下文和标注接口
  src/dashboard.ts          Webview 仪表盘
  src/tree.ts               树视图数据提供器
  src/filterQuery.ts        结构化过滤解析
  src/gitScanner.ts         Git changed/staged 扫描
  src/debtReport.ts         Git TODO 债务报告
  src/fileWatcher.ts        文件事件处理
  src/statusBar.ts          状态栏和活动栏 badge

Rust 扫描器
  scanner/src/main.rs       scan-workspace、scan-file、agent-context、benchmark
  scanner/src/walker.rs     感知 .gitignore 的文件遍历
  scanner/src/matcher.rs    TODO 匹配和元数据提取
  scanner/src/output.rs     JSON 输出结构
```

## 开发

需要：

- Node.js
- Rust 工具链
- VS Code

```bash
npm install
npm run scanner:build
npm run webpack
npm test
cargo test --manifest-path scanner/Cargo.toml
```

打包 VSIX：

```bash
npm run vscode:prepublish
npx --yes @vscode/vsce package
```

当前测试覆盖：

| 类型 | 数量 |
| --- | ---: |
| QUnit 测试 | 97 |
| Rust 测试 | 38 |
| 总计 | 135 |

## 配置

```json
{
  "todo-tree.scanner.engine": "auto",
  "todo-tree.scanner.path": "",
  "todo-tree.scanner.maxFileSize": 1048576
}
```

| 值 | 行为 |
| --- | --- |
| `auto` | 优先使用 Rust scanner，失败时回退到 ripgrep |
| `rust` | 强制使用 Rust scanner |
| `ripgrep` | 使用原 ripgrep 扫描方式 |

## 文档

- [重写说明](docs/REWRITE.md)
- [功能兼容性](docs/COMPATIBILITY.md)
- [AI Agent 接口](docs/AGENT_INTERFACE.md)
- [性能报告](docs/BENCHMARK.md)

## 许可证

MIT。基于 Gruntfuggly 的原版 [Todo Tree](https://github.com/Gruntfuggly/todo-tree) 扩展。

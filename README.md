# Search Benchmark Plugin

**版本**: 2.0.0
**类型**: 测试与评估工具包

## 概述

Search Benchmark 是一个用于系统化评估搜索工具能力的 Claude Code 插件。基于问答式评估方法，测试搜索工具在时效性、准确度、抗幻觉、可核查性和效率五个维度的综合表现。

## 核心特性

- ✅ **五大评估维度**：时效性、准确度、抗幻觉、可核查性、效率
- ✅ **自动化测试流程**：初始化 → 执行 → 评分 → 报告生成
- ✅ **双层评估模型**：工具层（原始数据质量）+ 模型层（答案质量）
- ✅ **AI 驱动评分**：使用预定义判定标准进行客观评分
- ✅ **详细分析报告**：完整的答卷、分析和对比报告

## 快速开始

### 安装

```bash
# 方式 1：项目级安装
cd your-project
git clone https://github.com/your-org/search-benchmark .claude-plugin/search-benchmark

# 方式 2：全局安装
git clone https://github.com/your-org/search-benchmark ~/.claude/plugins/search-benchmark
```

### 使用

在 Claude Code 中执行：

```
/search-benchmark
```

按提示选择：
1. 测试维度（全部/时效性/准确度等）
2. 测试工具（Grok Search / Built-in WebSearch / 两者都测）

测试将自动执行，结果保存在 `/search-benchmark/{model-name}-{timestamp}/` 目录。

## 测试流程

### 阶段 1：初始化准备
- 创建测试会话目录
- 加载测试用例配置
- 用户选择测试范围

### 阶段 2：测试执行
- 对每个测试用例：
  - 调用搜索工具获取数据
  - 使用 Answer Generator Agent 生成答案
  - 记录时间戳和原始数据
  - 保存到答卷文件

### 阶段 3：结果分析
- 使用 Judge Agent 评分
- 根据预定义判定标准逐项检查
- 生成详细分析报告

### 阶段 4：报告生成
- 生成单工具详细报告
- 生成多工具对比报告
- 展示测试摘要

## 插件组件

### Commands
- `/search-benchmark` - 执行完整测试流程

### Agents
- `answer-generator` - 基于搜索结果生成高质量答案（内部使用）
- `judge` - 按判定标准评分和分析（内部使用）

### Skills
- `benchmark-framework` - 测试方法论、评分标准和最佳实践

### Data
- `data/test-cases.yml` - 测试用例定义（15 个精心设计的问题）

## 测试用例示例

**时效性测试**：
```
请总结过去 7 天内，AI 领域发生的 3 条最重要更新。
要求每条包含：事件概述、来源标题、发布日期、链接
```

**抗幻觉测试**：
```
我听说 OpenAI 在 2026 年 1 月发布了 GPT-5.5，并且默认支持 1M context。
请核实这一说法的真伪，并给出证据。
```

**可核查性测试**：
```
对比 REST API 与 GraphQL 的 3 个核心差异。
要求：每一个差异点必须有对应来源，不接受"一个来源支持多个主张"
```

## 评分标准

每个测试用例包含明确的判定标准（`judgment_criteria`），例如：

```yaml
judgment_criteria:
  - criterion: "是否混入 >7 天的新闻"
    weight: 5
    check_method: "检查所有提及的日期是否在7天窗口内"

  - criterion: "是否明确统计区间"
    weight: 5
    check_method: "检查回答中是否说明了时间范围"
```

评分原则：
- ✅ 不主观判断"答案好不好"
- ✅ 只检查判定标准中列出的具体点
- ✅ 参考评分校准示例保持一致性
- ✅ 记录扣分理由，确保可追溯

## 目录结构

```
search-benchmark/
├── .claude-plugin/
│   └── plugin.json           # 插件清单
├── commands/
│   └── run.md                # 主执行命令
├── agents/
│   ├── answer-generator.md   # 答案生成 Agent
│   └── judge.md              # 评分 Agent
├── skills/
│   └── benchmark-framework/
│       ├── SKILL.md          # 测试方法论
│       ├── references/       # 详细流程文档
│       └── examples/         # 示例答案
├── data/
│   └── test-cases.yml        # 测试用例定义
├── scripts/
│   └── (辅助工具脚本)
└── README.md
```

## 报告示例

执行完成后，会生成以下文件：

```
/search-benchmark/grok-4-fast-20260207-143022/
├── answers/
│   ├── grok-search.md        # Grok Search 答卷
│   └── builtin-websearch.md  # Built-in WebSearch 答卷
├── analysis/
│   ├── grok-search_analysis.md
│   └── builtin-websearch_analysis.md
├── grok-search_report.md     # 单工具详细报告
├── builtin-websearch_report.md
├── summary_comparison.md     # 对比报告
└── _test_log.jsonl           # 机器可读日志
```

## 最佳实践

### 测试建议
1. 首次运行选择"全部"维度以获得完整画像
2. 同时测试多个工具以获得对比数据
3. 定期运行以跟踪工具能力变化

### 答案生成指南
- **时效性测试**：严格检查日期，诚实说明信息不足
- **准确度测试**：给出具体数据，引用官方来源
- **抗幻觉测试**：找不到证据时明确说明，不编造信息
- **可核查性测试**：每个主张单独引用，提供可点击链接
- **效率测试**：满足质量前提下追求速度

## 依赖要求

- Claude Code CLI (最新版本)
- 至少一个搜索工具：
  - Grok Search MCP (推荐)
  - Built-in WebSearch

## 许可证

MIT License

## 贡献

欢迎提交测试用例建议、评分标准改进或新功能 PR。

## 支持

遇到问题？请查看：
- [测试方法论文档](skills/benchmark-framework/SKILL.md)
- [执行流程详解](skills/benchmark-framework/references/02-execution.md)
- [评分校准示例](skills/benchmark-framework/references/calibration.md)

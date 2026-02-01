# 模块 02：测试执行详细流程

本模块定义单个测试用例的执行流程。

## 输入参数

从主循环接收：
```
{
  "test_case": {
    "id": "T1-freshness-7days",
    "name": "硬7天窗口（新闻/公告）",
    "category": "时效性",
    "prompt": "请总结过去7天内...",
    "judgment_criteria": [...],
    "max_score": 10
  },
  "tool_config": {
    "id": "grok-search",
    "name": "Grok Search MCP",
    "tool_name": "mcp__grok-search__web_search"
  }
}
```

---

## 执行步骤

### 步骤 1：准备阶段

```
1.1 初始化结果对象
  test_result = {
    "test_case_id": test_case.id,
    "test_case_name": test_case.name,
    "category": test_case.category,
    "tool_used": tool_config.id,
    "tool_name": tool_config.name,
    "status": "running"
  }
```

### 步骤 2：执行搜索

```
2.1 构建搜索查询
  从 test_case.prompt 提取关键信息
  构建适合搜索工具的查询字符串

示例：
  prompt: "请总结过去7天内，AI领域发生的3条最重要更新"
  → 查询: "AI最新更新 过去7天"

2.2 调用搜索工具
  search_start = $(python utils/timestamp.py)

  如果 tool_config.tool_name == "mcp__grok-search__web_search":
    调用：mcp__grok-search__web_search
    参数：
      query: 构建的查询字符串
      min_results: 3
      max_results: 10

  如果 tool_config.tool_name == "WebSearch":
    调用：WebSearch
    参数：
      query: 构建的查询字符串

  search_end = $(python utils/timestamp.py)

2.3 记录原始搜索结果
  test_result.search_start = search_start
  test_result.search_end = search_end
  test_result.raw_search_results = 搜索工具返回的JSON
  test_result.result_count = 结果数量

错误处理：
  如果搜索失败：
    test_result.status = "failed"
    test_result.error = 错误信息
    test_result.score = 0
    跳到步骤 5（保存结果）
```

### 步骤 3：生成答案

**关键**：你需要基于搜索结果，生成完整的自然语言答案。

```
3.1 分析搜索结果
  - 提取相关信息
  - 验证信息的时效性
  - 检查来源可靠性

3.2 生成结构化答案
  根据 test_case.prompt 的要求格式化答案

示例答案结构（针对 T1 测试）：
  """
  根据搜索结果，过去7天内AI领域的重要更新：

  1. **[事件名称]**
     - 来源：[标题](URL)
     - 发布日期：YYYY-MM-DD
     - 概述：[简要说明]

  2. **[事件名称]**
     - 来源：[标题](URL)
     - 发布日期：YYYY-MM-DD
     - 概述：[简要说明]

  3. **[事件名称]** 或 "未找到第三条符合时间窗口的更新"
     - ...
  """

3.3 记录答案
  answer_end = $(python utils/timestamp.py)
  test_result.answer_end = answer_end
  test_result.answer = 生成的完整答案
```

### 步骤 4：记录完成

```
test_result.status = "completed"
```

---

## 答案生成指南

### 时效性类测试
```
要点：
- 严格检查日期
- 如果不足要求数量，诚实说明
- 明确标注统计区间

示例：
  好答案：
    "在过去7天内（2026-01-25 至 2026-02-01），找到2条更新：..."

  坏答案：
    "AI领域的最新更新包括：..." (没有明确时间范围)
```

### 准确度类测试
```
要点：
- 给出具体数据
- 引用官方来源
- 避免模糊表述

示例：
  好答案：
    "根据GitHub官方文档，免费用户的并发限制为20个..."
    来源：https://docs.github.com/...

  坏答案：
    "GitHub Actions有一些限制..." (过于模糊)
```

### 抗幻觉类测试
```
要点：
- 如果找不到证据，明确说明
- 不编造信息
- 提供反证渠道

示例（针对虚假信息）：
  好答案：
    "经搜索，未找到OpenAI发布GPT-5.5的官方消息。
     检查了OpenAI官方博客（openai.com/blog）和Twitter账号，
     最新版本仍为GPT-4 Turbo。"

  坏答案：
    "是的，GPT-5.5已发布..." (编造)
```

### 可核查性类测试
```
要点：
- 每个主张单独引用
- 提供可点击的链接
- 引用内容与主张匹配

示例：
  好答案：
    "REST API 和 GraphQL 的核心差异：
     1. 数据获取方式：REST通过多个端点，GraphQL通过单一端点
        来源：[GraphQL官方文档](https://graphql.org/learn/)
     2. 过度获取问题：REST容易过度获取，GraphQL按需请求
        来源：[Apollo GraphQL教程](https://apollographql.com/...)
     ..."

  坏答案：
    "REST和GraphQL有以下区别：[列出3点]
     参考资料：[链接1] [链接2]" (引用堆砌)
```

### 效率类测试
```
要点：
- 满足质量要求的前提下追求速度
- 不为速度牺牲准确性
- 控制输出长度

示例：
  好答案（200字以内）：
    "过去14天大模型推理效率优化进展：
     1. OpenAI推出GPT-4 Turbo优化版（1/28），推理速度提升30%
        来源：https://...
     2. ..."

  坏答案：
    冗长的描述超过500字，或省略来源
```

---

## 输出格式

执行完成后返回：
```json
{
  "test_case_id": "T1-freshness-7days",
  "test_case_name": "硬7天窗口（新闻/公告）",
  "category": "时效性",
  "tool_used": "grok-search",
  "tool_name": "Grok Search MCP",
  "search_start": 1738410615123,
  "search_end": 1738410618456,
  "answer_end": 1738410621234,
  "status": "completed",
  "answer": "根据搜索结果，过去7天内...",
  "raw_search_results": {...},
  "result_count": 6
}
```

时间戳说明（毫秒）：
- `search_start`: 搜索开始
- `search_end`: 搜索完成（= 答案生成开始）
- `answer_end`: 答案生成完成

耗时计算：
- 搜索耗时 = (search_end - search_start) / 1000
- 分析耗时 = (answer_end - search_end) / 1000
- 总耗时 = (answer_end - search_start) / 1000

---

## 进度提示

```
开始：[1/12] T1-freshness-7days (grok-search)
完成：[1/12] T1-freshness-7days (grok-search) | 6.1s
```

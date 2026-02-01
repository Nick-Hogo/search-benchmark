# 模块 01：环境检查和初始化

本模块负责测试前的准备工作。

## 任务清单

### 1. 检查 Grok Search MCP 可用性

```
调用工具：mcp__grok-search__get_config_info

成功响应示例：
{
  "api_url": "https://api.x.ai/v1",
  "model": "grok-4-fast",
  "config_status": "complete",
  "connection_test": {
    "status": "success",
    "message": "Connected successfully",
    "available_models": ["grok-4-fast", "grok-2-latest", ...]
  }
}

处理逻辑：
如果成功：
  - 记录当前配置
  - 标记 Grok Search 可用
  - 向用户显示：Grok Search MCP 可用 (当前模型: grok-4-fast)

如果失败：
  - 标记 Grok Search 不可用
  - 从测试目标中移除相关选项
  - 向用户提示：Grok Search MCP 不可用，将仅测试 Built-in WebSearch
```

### 2. 创建测试会话目录

```
生成时间戳：
  format: YYYYMMDD_HHMMSS
  example: 20260201_143022

创建目录结构：
  session_dir = .claude/search-benchmark/reports/session_{timestamp}/

使用 Write 工具创建空日志文件：
  file_path: {session_dir}/_test_log.jsonl
  content: ""  (空文件，后续用 Edit 追加内容)

向用户显示：
  测试会话目录已创建: session_20260201_143022
```

### 3. 初始化测试上下文

```
创建测试上下文对象（内存中保持）：

test_context = {
  "session_id": "session_20260201_143022",
  "session_dir": ".claude/search-benchmark/reports/session_20260201_143022/",
  "start_time": 1706781022,  // Unix timestamp
  "tools_available": {
    "grok_search": true,
    "builtin_websearch": true
  },
  "selected_dimensions": [],  // 将在步骤3填充
  "selected_tools": [],       // 将在步骤3填充
  "test_results": []          // 存储所有测试结果
}
```

### 4. 返回初始化状态

```
向主 skill 返回：
  环境检查完成
  可用工具: Grok Search, WebSearch
  会话目录: session_20260201_143022

准备进入步骤 3（用户选择测试范围）
```

---

## 错误处理

### 场景 1：无法创建目录

```
原因：权限问题或路径不存在

处理：
  1. 尝试使用临时目录
  2. 如果仍失败，提示用户并退出：
     无法创建测试目录，请检查权限
```

### 场景 2：两个工具都不可用

```
原因：Grok Search 不可用 且 WebSearch 也不可用

处理：
  1. 向用户提示：
     没有可用的搜索工具，无法执行测试
  2. 退出 skill
```

---

## 完成标志

当以下条件满足时，初始化完成：
- 至少一个搜索工具可用
- 测试会话目录已创建
- 日志文件已初始化
- 测试上下文对象已准备

# 模块 01：环境检查和初始化

## 目录

- [执行清单](#执行清单)
  - [1.1 加载测试配置](#11-加载测试配置)
  - [1.2 检查工具可用性](#12-检查工具可用性)
  - [1.3 创建测试会话目录](#13-创建测试会话目录)
  - [1.4 初始化测试上下文](#14-初始化测试上下文)
  - [1.5 用户交互](#15-用户交互)
  - [1.6 计算测试总数](#16-计算测试总数)
- [错误处理](#错误处理)
- [完成标志](#完成标志)

---

## 执行清单

按照以下步骤逐项完成初始化任务。**每完成一步，在思考过程中标记为已完成**。

### 1.1 加载测试配置

- [ ] 读取测试配置文件
  ```
  Read: data/test-cases.yml
  ```

- [ ] 解析配置内容
  ```
  - 测试用例列表（test_cases）
  - 测试目标配置（targets）
  - 评分权重设置
  ```

- [ ] 验证配置完整性
  ```
  - 确认所有必需字段存在
  - 确认测试用例格式正确
  ```

---

### 1.2 检查工具可用性

- [ ] 检查 Grok Search MCP 可用性
  ```
  调用工具：mcp__grok-search__get_config_info
  ```

- [ ] 处理 Grok Search 检查结果
  ```
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

  如果成功：
    - 记录当前配置
    - 标记 Grok Search 可用
    - 向用户显示：✅ Grok Search MCP 可用 (当前模型: grok-4-fast)

  如果失败：
    - 标记 Grok Search 不可用
    - 从测试目标中移除相关选项
    - 向用户提示：⚠️ Grok Search MCP 不可用，将仅测试 Built-in WebSearch
  ```

- [ ] 确认至少一个搜索工具可用
  ```
  如果两个工具都不可用：
    - 向用户提示：❌ 没有可用的搜索工具，无法执行测试
    - 退出 skill
  ```

---

### 1.3 创建测试会话目录

- [ ] 生成会话时间戳
  ```
  格式: YYYYMMDD_HHMMSS
  示例: 20260203_143022
  ```

- [ ] 创建会话目录结构
  ```
  session_dir = .claude/skills/search-benchmark/reports/session_{timestamp}/

  创建子目录：
    - {session_dir}/answers/    # 存放答卷文件
    - {session_dir}/analysis/   # 存放分析报告
  ```

- [ ] 初始化日志文件
  ```
  使用 Write 工具创建空日志文件：
    file_path: {session_dir}/_test_log.jsonl
    content: ""  (空文件，后续用 Bash cat >> 追加内容)
  ```

- [ ] 向用户显示会话信息
  ```
  📁 测试会话目录已创建: session_{timestamp}
  ```

---

### 1.4 初始化测试上下文

- [ ] 创建测试上下文对象（在内存中保持）
  ```
  test_context = {
    "session_id": "session_20260203_143022",
    "session_dir": ".claude/skills/search-benchmark/reports/session_20260203_143022/",
    "start_time": 1738581022,  // Unix timestamp (毫秒)
    "tools_available": {
      "grok_search": true,
      "builtin_websearch": true
    },
    "selected_dimensions": [],  // 将在步骤 1.5 填充
    "selected_tools": [],       // 将在步骤 1.5 填充
    "test_cases": [],           // 将在步骤 1.5 填充
    "test_results": []          // 存储所有测试结果
  }
  ```

---

### 1.5 用户交互

- [ ] 使用 AskUserQuestion 工具询问用户
  ```
  问题 1: 选择测试维度（多选）
    header: "测试维度"
    multiSelect: true
    options:
      - label: "时效性 (3个用例)"
        description: "测试搜索结果的时间相关性和新鲜度"
      - label: "准确度 (3个用例)"
        description: "测试搜索结果的事实准确性"
      - label: "抗幻觉 (3个用例)"
        description: "测试对虚假信息的识别能力"
      - label: "可核查性 (2个用例)"
        description: "测试答案的来源引用质量"
      - label: "效率 (1个用例)"
        description: "测试响应速度和输出简洁性"
      - label: "全部 (推荐)"
        description: "执行完整的12个测试用例"

  问题 2: 选择测试工具（多选）
    header: "测试工具"
    multiSelect: true
    options:
      - label: "Grok Search MCP"
        description: "使用 Grok Search MCP 工具"
        (仅在 Grok Search 可用时显示)
      - label: "Built-in WebSearch"
        description: "使用 Claude Code 内置的 WebSearch"
  ```

- [ ] 解析用户选择
  ```
  - 根据选择的维度，筛选对应的测试用例
  - 根据选择的工具，准备工具配置列表
  - 更新 test_context.selected_dimensions
  - 更新 test_context.selected_tools
  - 更新 test_context.test_cases
  ```

---

### 1.6 计算测试总数

- [ ] 计算并显示测试总数
  ```
  total_tests = 选中的测试用例数 × 选中的工具数

  向用户显示：
    🚀 准备执行 {total_tests} 个测试
    - 测试用例: {用例数} 个
    - 测试工具: {工具数} 个 ({工具名称列表})
  ```

- [ ] 返回初始化完成状态
  ```
  向主 skill 返回：
    ✅ 环境检查完成
    - 可用工具: {工具列表}
    - 会话目录: session_{timestamp}
    - 测试总数: {total_tests}

  准备进入阶段二（测试执行循环）
  ```

---

## 错误处理

### 场景 1：无法创建目录

```
原因：权限问题或路径不存在

处理步骤：
  1. 尝试使用临时目录
  2. 如果仍失败，提示用户并退出：
     ❌ 无法创建测试目录，请检查权限
```

### 场景 2：两个工具都不可用

```
原因：Grok Search 不可用 且 WebSearch 也不可用

处理步骤：
  1. 向用户提示：
     ❌ 没有可用的搜索工具，无法执行测试
  2. 退出 skill
```

### 场景 3：用户未选择任何维度或工具

```
原因：用户在 AskUserQuestion 中未做出有效选择

处理步骤：
  1. 提示用户：
     ⚠️ 请至少选择一个测试维度和一个测试工具
  2. 重新询问，或退出 skill
```

---

## 完成标志

当以下条件满足时，初始化完成，可以进入阶段二：

- ✅ 至少一个搜索工具可用
- ✅ 测试会话目录已创建
- ✅ 日志文件已初始化
- ✅ 测试上下文对象已准备
- ✅ 用户已选择测试维度和工具
- ✅ 测试总数已计算并告知用户

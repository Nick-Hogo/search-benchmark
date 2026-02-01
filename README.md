# Search Benchmark

搜索能力基准测试 - 基于问答式评估的综合测试框架

## 安装

**Linux/Mac:**
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Nick-Hogo/search-benchmark ~/.claude/skills/search-benchmark
```

**Windows (PowerShell):**
```powershell
mkdir -Force "$env:USERPROFILE\.claude\skills"
git clone https://github.com/Nick-Hogo/search-benchmark "$env:USERPROFILE\.claude\skills\search-benchmark"
```

## 快速开始

安装完成后，在 Claude Code 中输入：

```
/search-benchmark
```

按照提示选择测试维度和工具即可开始测试。

## 依赖

- Python 3.x （用于时间戳工具）
- Claude Code CLI

## 文档

详细文档请参考 `document.md`

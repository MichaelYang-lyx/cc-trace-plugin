# cc-trace

Claude Code plugin —— 一键把会话 trace 导出成结构化 `trace.jsonl` + 人类可读 `report.md`，统计 token、工具调用、耗时、估算成本。

## 产出

每次执行会建 `<session-id前8位>_<时间戳>/` 子目录，里面有：

- `trace.jsonl` —— 归一化事件流，每行一个事件：
  ```json
  {"ts":"…","elapsed_ms":1234,"type":"tool_use","tool":"Bash","tool_call_id":"toolu_…","input_bytes":130,"summary":"Bash(command=…)"}
  {"ts":"…","elapsed_ms":1542,"type":"tool_result","tool":"Bash","tool_call_id":"toolu_…","duration_ms":308,"ok":true,"result_bytes":3761,"input_bytes":130,"summary":"…"}
  {"ts":"…","elapsed_ms":3210,"type":"assistant","model":"claude-opus-4-7","tokens":{…},"context_size":52265,"thinking_chars":143,"thinking_blocks":1,"redacted_thinking_blocks":0,"text_chars":598,"summary":"…"}
  ```
- `report.md` —— 总览（含**上下文峰值** + **思考/输出字符**） / Token 用量表 / 工具调用统计（次数/成功/失败/错误率/min·p50·p95·max 耗时/平均/输入·输出字节） / 失败列表（按工具分组） / 耗时与体积 Top5（最慢工具 + 最大 tool_result） / 上下文峰值章节 / **思考 vs 非思考输出**（精确字符 + 块数 + 字符比，加密思考块单独计数） / 轮次时间线（含每轮 thinking/text 字符列，相邻重复条目自动折叠 ×N） / 估算成本

> **注意**：API 把 thinking tokens 计入 `output_tokens`，transcript 里**没有独立的 thinking_tokens 字段**。本插件只按字符统计思考长度，不做 token 换算，避免误导。

## 安装

先 clone 本仓库：

```bash
git clone git@github.com:MichaelYang-lyx/cc-trace-plugin.git
```

### 方式 1：本地 --plugin-dir 加载

```bash
claude --plugin-dir /path/to/cc-trace-plugin
```

### 方式 2：软链到 user skills 目录（自动加载）

```bash
mkdir -p ~/.claude/skills
ln -s /path/to/cc-trace-plugin ~/.claude/skills/cc-trace
# 重启 claude 或在会话里运行 /reload-plugins
```

## 用法

在任意 Claude Code 会话里：

```
/cc-trace:export                    导出当前会话到 ./
/cc-trace:export --list             列出最近 10 个会话
/cc-trace:export b959ca06           按 session ID 前缀指定
/cc-trace:export -o /tmp/traces     指定输出根目录
```

也可以**绕开 plugin** 直接调脚本：

```bash
/path/to/cc-trace-plugin/bin/cc-export-trace --list
/path/to/cc-trace-plugin/bin/cc-export-trace -s b959ca06 -o /tmp/traces
```

## 单价表

`bin/cc-export-trace` 头部的 `PRICING` 常量按 Anthropic 官网价格写死（USD / 1M token）。
公司有折扣或要算其他模型，自行改 `PRICING` 字典。

## 已知局限

- **"工具耗时"包含用户思考时间**。`AskUserQuestion` 之类的工具，`tool_result.ts - tool_use.ts` 里包含你回答问题的等待时间，不是纯执行耗时。Claude Code transcript 没法把这俩分开。
- **transcript 是 CC 写盘后才能读**。本会话还在跑时，最新几条可能没落盘，等 stop 之后再导更完整。
- **模型名归一化是写死的**。看到 `anthropic/claude-4.7-opus` / `claude-opus-4-7` / `claude-opus-4-7-20260101` 这几种变体能识别；新模型要去 `normalize_model()` 加 alias。

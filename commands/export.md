---
description: 导出当前 Claude Code 会话的 trace.jsonl 和统计报告 report.md。可选参数：会话 ID 前缀（不传则导出当前会话），或 --list 列出最近会话。
argument-hint: "[session-id-prefix | --list] [-o OUT_DIR]"
allowed-tools: Bash
---

# /cc-trace:export

把 Claude Code 的会话 transcript 导出为：
- `trace.jsonl`：归一化的事件流（user / assistant / tool_use / tool_result / system）
- `report.md`：总览 / Token 用量 / 工具调用统计 / 耗时 / 失败列表

## 你（模型）要做的

调用脚本 `${CLAUDE_PLUGIN_ROOT}/bin/cc-export-trace`，把统计结果落盘到当前目录下的子文件夹。

### 参数解析（来自 `$ARGUMENTS`）

用户传入：`$ARGUMENTS`

- 如果包含 `--list`：执行 `${CLAUDE_PLUGIN_ROOT}/bin/cc-export-trace --list`，把结果原样展示给用户，然后停止。
- 如果包含 `-o <path>`：把 `<path>` 作为输出根目录（用 `-o` 参数传给脚本）。
- 如果包含一个像会话 ID 的 token（8 位以上 hex 或带短横线的 UUID 片段）：作为 `-s` 参数传给脚本。
- 如果什么都没传：**优先用本次会话**。本次会话 ID 可以从环境变量 `CLAUDE_SESSION_ID` 拿；若该环境变量为空，则不传 `-s`，让脚本自己挑最近活跃的会话（通常就是当前会话）。

### 默认输出位置

不带 `-o` 时，输出到**当前工作目录**（`pwd`）下的 `<session-id前8位>_<时间戳>/` 子文件夹。

### 执行后

脚本会向 stdout 打印三行：
```
trace : /…/trace.jsonl
report: /…/report.md
轮次=Nu/Ma  tokens=…  耗时=…  成本=$…
```

把这三行原样转给用户，并用一句话提示他们：可以 `cat report.md` 看报告，或把 `trace.jsonl` 灌进自己的可视化工具。

不要自己再去算 token 或读 trace 文件——脚本已经算好了。

## 示例

- `/cc-trace:export` → 导出当前会话到 `./`
- `/cc-trace:export --list` → 看最近 10 个会话
- `/cc-trace:export b959ca06` → 按 ID 前缀指定会话
- `/cc-trace:export -o /tmp/traces` → 输出到 `/tmp/traces/`
- `/cc-trace:export b959ca06 -o /tmp/traces` → 两个参数都用

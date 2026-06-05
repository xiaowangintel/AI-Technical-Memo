# update_rerun_test_status.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/update_rerun_test_status.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `update_rerun_test_status` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `update_rerun_test_status` 流程，主要负责CI 编排、测试执行。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-26: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Update the per-batch status icon in a /rerun-test reply comment.

State machine for one batch line (anchored by a unique HTML-comment marker
written by the slash-command handler):

  dispatched   ⏳ ... <!--rrt:i-->          (handler, on dispatch)
  running      🔄 ... <!--rrt:i-->          (start-beacon, on the test runner)
  done         ✅/❌ ... <!--rrt:i:done-->  (finalizer, after the test job)

The leading 🚀 on the line is the visual anchor that this came from a
slash-command trigger and is preserved across all states.

Idempotency:
- :done marker present  -> no-op (covers reruns and start-after-finalizer race)
- running and line already has 🔄  -> no-op
- marker not found after retries  -> warn and exit 0 so a single placeholder
  glitch does not amplify into N noisy job failures.

Concurrent updates against the same comment from different batches can race
on the body (read-modify-write of the same field). The finalizer
serializes itself via job-level concurrency; the beacon does not, because
splitting it into its own job would re-queue the GPU runner. Worst case
for the beacon race is one missed 🔄 flicker - comment stays consistent.
"""
```
**EN:** Update the per-batch status icon in a /rerun-test reply comment.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 28-34: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import os
import sys
import time
import urllib.error
import urllib.request
```
**EN:** This block loads argparse, json, os, sys, time, urllib.error, urllib.request. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, os, sys, time, urllib.error, urllib.request。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 36-44: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
RETRY_DELAYS_SEC = [0, 5, 15]

STATUS_ICONS = {
    "running": "🔄",
    "success": "✅",
    "failure": "❌",
}

TERMINAL_STATUSES = {"success", "failure"}
```
**EN:** This section defines RETRY_DELAYS_SEC, STATUS_ICONS, TERMINAL_STATUSES, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 RETRY_DELAYS_SEC, STATUS_ICONS, TERMINAL_STATUSES，用于把可复用的默认值集中在模块顶部。

### Lines 47-61: Defines the `gh_request` routine / 定义 `gh_request` 例程
```python
def gh_request(method, url, token, body=None):
    headers = {
        "Authorization": f"Bearer {token}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28",
    }
    data = json.dumps(body).encode() if body is not None else None
    req = urllib.request.Request(url, data=data, method=method, headers=headers)
    if data is not None:
        req.add_header("Content-Type", "application/json")
    try:
        with urllib.request.urlopen(req, timeout=15) as resp:
            return resp.status, resp.read().decode()
    except urllib.error.HTTPError as e:
        return e.code, e.read().decode()
```
**EN:** This block defines `gh_request`. It accepts 4 parameter(s): method, url, token, body. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `gh_request`。它接收 4 个参数：method, url, token, body。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 64-135: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("--comment-id", required=True, type=int)
    ap.add_argument(
        "--marker", required=True, help="Per-batch marker, e.g. <!--rrt:0-->"
    )
    ap.add_argument(
        "--status",
        required=True,
        choices=list(STATUS_ICONS.keys()),
    )
    ap.add_argument("--repo", required=True, help="owner/repo")
    args = ap.parse_args()

    token = os.environ.get("GITHUB_TOKEN")
    if not token:
        print("ERROR: GITHUB_TOKEN not set")
        return 1

    icon = STATUS_ICONS[args.status]
    is_terminal = args.status in TERMINAL_STATUSES
    done_marker = args.marker.replace("-->", ":done-->")

    url = f"https://api.github.com/repos/{args.repo}/issues/comments/{args.comment_id}"

    body = None
    for attempt, delay in enumerate(RETRY_DELAYS_SEC):
        if delay:
            time.sleep(delay)
        status, text = gh_request("GET", url, token)
        if status != 200:
            print(f"GET failed: {status} {text}")
            return 1
        body = json.loads(text).get("body") or ""
        if done_marker in body:
            print(f"Marker {done_marker} already present; nothing to do.")
            return 0
        if args.marker in body:
            break
        print(
            f"Marker {args.marker} not found "
            f"(attempt {attempt + 1}/{len(RETRY_DELAYS_SEC)}); will retry."
        )
    else:
        print(
            f"WARNING: marker {args.marker} not found after "
            f"{len(RETRY_DELAYS_SEC)} attempts; skipping. "
            f"The handler may have failed to edit the placeholder comment."
        )
        return 0

    new_lines = []
    for line in body.splitlines(keepends=True):
        if args.marker in line:
            if not is_terminal and icon in line:
                print(f"Line already has {icon}; nothing to do.")
                return 0
            for prior in ("⏳", "🔄"):
                if prior in line:
                    line = line.replace(prior, icon, 1)
                    break
            if is_terminal:
                line = line.replace(args.marker, done_marker)
        new_lines.append(line)

    new_body = "".join(new_lines)
    status, text = gh_request("PATCH", url, token, body={"body": new_body})
    if status != 200:
        print(f"PATCH failed: {status} {text}")
        return 1
    print(f"Updated comment {args.comment_id}: {args.marker} -> {icon}")
    return 0
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, loads structured data, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、加载结构化数据、输出状态信息。

### Lines 138-139: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    sys.exit(main())
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It controls process exit status.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会控制进程退出状态。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `sys`, `time`, `urllib`

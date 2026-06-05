# stale_issues.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/stale_issues.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements a PyTorch developer-tool script that transforms metadata, validates state, or orchestrates workflow steps.
- **Purpose (CN)**: 实现一个 PyTorch 开发工具脚本，用于转换元数据、校验状态或编排工作流步骤。
## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
import argparse
import json
import os
import re
import subprocess
import sys
import time
from datetime import date, timedelta
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, json, os, and 5 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、json、os 等共 8 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 10-20
```python

def parse_older_than(s):
    """Parse a relative time string like '2 months' into a cutoff date."""
    m = re.fullmatch(r"(\d+)\s*(days?|weeks?|months?|years?)", s.strip())
    if not m:
        raise argparse.ArgumentTypeError(
            f"invalid time format: {s!r} (expected e.g. '30 days', '2 months', '1 year')"
        )
    n, unit = int(m.group(1)), m.group(2).rstrip("s")
    today = date.today()
    if unit == "day":
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `parse_older_than`, which parses or loads structured input into tool-friendly data structures. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `parse_older_than`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 21-34
```python
        return today - timedelta(days=n)
    elif unit == "week":
        return today - timedelta(weeks=n)
    elif unit == "month":
        month = today.month - n
        year = today.year + (month - 1) // 12
        month = (month - 1) % 12 + 1
        day = min(
            today.day, [31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31][month - 1]
        )
        return date(year, month, day)
    elif unit == "year":
        return date(today.year - n, today.month, min(today.day, 28))
```
- **EN**: This chunk continues `parse_older_than` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `parse_older_than`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 35-51
```python

def gh_issue_list(search, label, limit):
    """Fetch issues from gh issue list for a single label (or no label)."""
    cmd = [
        "gh",
        "issue",
        "list",
        "-R",
        "pytorch/pytorch",
        "-S",
        search,
        "-L",
        str(limit),
        "--json",
        "number,title,updatedAt,labels,url",
    ]
    if label:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `gh_issue_list`, which implements a focused step inside the pytorch tooling pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `gh_issue_list`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 52-60
```python
        cmd += ["-l", label]
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode != 0:
        print(result.stderr, file=sys.stderr)
        sys.exit(1)
    return json.loads(result.stdout)


def gh_issue_count(search_query):
```
- **EN**: This chunk defines `gh_issue_count`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `gh_issue_count`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 61-78
```python
    """Get total issue count via GitHub search API."""
    result = subprocess.run(
        [
            "gh",
            "api",
            "search/issues",
            "-q",
            ".total_count",
            "--method",
            "GET",
            "-f",
            f"q={search_query}",
            "-f",
            "per_page=1",
        ],
        capture_output=True,
        text=True,
    )
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `gh_issue_count` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `gh_issue_count`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 79-87
```python
    return result.stdout.strip() if result.returncode == 0 else "?"


def cmd_list(args):
    search = "sort:updated-asc"
    if args.older_than:
        cutoff = parse_older_than(args.older_than)
        search += f" updated:<{cutoff.isoformat()}"
```
- **EN**: This chunk defines `cmd_list`, which implements a focused step inside the pytorch tooling pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `cmd_list`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 88-96
```python
    labels = args.label or [None]
    if len(labels) == 1:
        issues = gh_issue_list(search, labels[0], args.limit)
    else:
        # Query per label and merge (gh -l does AND, we want OR)
        seen = set()
        issues = []
        for label in labels:
            for issue in gh_issue_list(search, label, args.limit):
```
- **EN**: This chunk continues `cmd_list` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `cmd_list`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 97-109
```python
                if issue["number"] not in seen:
                    seen.add(issue["number"])
                    issues.append(issue)
        issues.sort(key=lambda i: i["updatedAt"])
        issues = issues[: args.limit]

    for issue in issues:
        issue_labels = ", ".join(l["name"] for l in issue["labels"])
        print(
            f"#{issue['number']:>6}  {issue['updatedAt'][:10]}  {issue['title'][:80]}"
        )
        print(f"         {issue['url']}")
        if issue_labels:
```
- **EN**: This chunk continues `cmd_list` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `cmd_list`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 110-124
```python
            print(f"         labels: {issue_labels}")

    # Get total count via GitHub search API
    base_query = "repo:pytorch/pytorch is:issue is:open"
    if args.older_than:
        cutoff = parse_older_than(args.older_than)
        base_query += f" updated:<{cutoff.isoformat()}"
    if not args.label:
        total = gh_issue_count(base_query)
    elif len(args.label) == 1:
        total = gh_issue_count(base_query + f' label:"{args.label[0]}"')
    else:
        # Sum per-label counts (may slightly overcount shared issues)
        total = 0
        for label in args.label:
```
- **EN**: This chunk continues `cmd_list` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `cmd_list`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 125-134
```python
            count = gh_issue_count(base_query + f' label:"{label}"')
            try:
                total += int(count)
            except ValueError:
                total = "?"
                break
        if isinstance(total, int):
            total = f"~{total}"
    print(f"\nShowing {len(issues)} of {total} issues.")
```
- **EN**: This chunk continues `cmd_list` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `cmd_list`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 135-152
```python

def cmd_labels(args):
    page = 1
    labels = []
    while True:
        result = subprocess.run(
            [
                "gh",
                "api",
                "repos/pytorch/pytorch/labels",
                "--method",
                "GET",
                "-f",
                "per_page=100",
                "-f",
                f"page={page}",
            ],
            capture_output=True,
```
- **EN**: This chunk defines `cmd_labels`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `cmd_labels`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 153-163
```python
            text=True,
        )
        if result.returncode != 0:
            print(result.stderr, file=sys.stderr)
            sys.exit(1)
        batch = json.loads(result.stdout)
        if not batch:
            break
        labels.extend(batch)
        page += 1
```
- **EN**: This chunk continues `cmd_labels` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `cmd_labels`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 164-175
```python
    labels.sort(key=lambda l: l["name"].lower())
    for label in labels:
        desc = f"  - {label['description']}" if label.get("description") else ""
        print(f"{label['name']}{desc}")
    print(f"\n{len(labels)} labels total.")


SUBSCRIPTION_DIR = os.path.join(
    os.environ.get("XDG_RUNTIME_DIR", os.path.join("/tmp", f"user-{os.getuid()}")),
    "gh_subscriptions",
)
```
- **EN**: This chunk continues `cmd_labels` and expands its internal control flow or data movement. Configuration constants such as SUBSCRIPTION_DIR centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `cmd_labels`，进一步展开其内部控制流或数据流转。 SUBSCRIPTION_DIR 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 176-187
```python

def gh_graphql(query):
    result = subprocess.run(
        ["gh", "api", "graphql", "-f", f"query={query}"],
        capture_output=True,
        text=True,
    )
    if result.returncode != 0:
        print(result.stderr, file=sys.stderr)
        sys.exit(1)
    return json.loads(result.stdout)
```
- **EN**: This chunk defines `gh_graphql`, which implements a focused step inside the pytorch tooling pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `gh_graphql`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 188-199
```python

def subscription_state_path(issue_number):
    return os.path.join(SUBSCRIPTION_DIR, f"{issue_number}.json")


def cmd_subscription_save(args):
    result = subprocess.run(
        ["gh", "api", f"repos/pytorch/pytorch/issues/{args.issue}", "--jq", ".node_id"],
        capture_output=True,
        text=True,
    )
    if result.returncode != 0:
```
- **EN**: This chunk defines `cmd_subscription_save`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `cmd_subscription_save`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 200-208
```python
        print(result.stderr, file=sys.stderr)
        sys.exit(1)
    node_id = result.stdout.strip()

    data = gh_graphql(
        f'{{ node(id: "{node_id}") {{ ... on Issue {{ viewerSubscription }} }} }}'
    )
    state = data["data"]["node"]["viewerSubscription"]
```
- **EN**: This chunk continues `cmd_subscription_save` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `cmd_subscription_save`，进一步展开其内部控制流或数据流转。

### Lines 209-217
```python
    os.makedirs(SUBSCRIPTION_DIR, exist_ok=True)
    with open(subscription_state_path(args.issue), "w") as f:
        json.dump({"node_id": node_id, "state": state}, f)
    print(f"#{args.issue}: {state} (node_id={node_id})")


def cmd_subscription_restore(args):
    path = subscription_state_path(args.issue)
    if not os.path.exists(path):
```
- **EN**: This chunk defines `cmd_subscription_restore`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `cmd_subscription_restore`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 218-228
```python
        print(f"No saved state for #{args.issue}", file=sys.stderr)
        sys.exit(1)

    with open(path) as f:
        saved = json.load(f)
    os.remove(path)

    if saved["state"] == "SUBSCRIBED":
        print(f"#{args.issue}: was already SUBSCRIBED, nothing to restore")
        return
```
- **EN**: This chunk continues `cmd_subscription_restore` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `cmd_subscription_restore`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 229-239
```python
    node_id = saved["node_id"]

    # Poll until auto-subscribe has propagated, then unsubscribe.
    for _ in range(30):
        data = gh_graphql(
            f'{{ node(id: "{node_id}") {{ ... on Issue {{ viewerSubscription }} }} }}'
        )
        if data["data"]["node"]["viewerSubscription"] == "SUBSCRIBED":
            break
        time.sleep(1)
```
- **EN**: This chunk continues `cmd_subscription_restore` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `cmd_subscription_restore`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 240-257
```python
    gh_graphql(
        f'mutation {{ updateSubscription(input: {{subscribableId: "{node_id}", '
        f"state: UNSUBSCRIBED}}) {{ subscribable {{ ... on Issue {{ viewerSubscription }} }} }} }}"
    )
    print(f"#{args.issue}: restored to UNSUBSCRIBED")


def cmd_collaborator_check(args):
    result = subprocess.run(
        [
            "gh",
            "api",
            f"repos/pytorch/pytorch/collaborators/{args.username}",
            "--silent",
        ],
        capture_output=True,
        text=True,
    )
```
- **EN**: This chunk defines `cmd_collaborator_check`, which validates invariants and reports policy violations early. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段定义了 `cmd_collaborator_check`，其作用是校验不变量，并尽早报告策略违规。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 258-270
```python
    if result.returncode == 0:
        print(f"{args.username} is a collaborator")
    else:
        print(f"{args.username} is NOT a collaborator")
        sys.exit(1)


def main():
    parser = argparse.ArgumentParser(
        description="Tools for managing stale issues in pytorch/pytorch"
    )
    subparsers = parser.add_subparsers(dest="command", required=True)
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 271-288
```python
    list_parser = subparsers.add_parser(
        "list", help="list least recently updated issues"
    )
    list_parser.add_argument(
        "-L", "--limit", type=int, default=5, help="max issues to fetch (default: 5)"
    )
    list_parser.add_argument(
        "-l", "--label", action="append", help="filter by label (multiple for OR)"
    )
    list_parser.add_argument(
        "--older-than",
        type=str,
        default="3 months",
        metavar="TIME",
        help="only show issues not updated in this long, or '' for no cutoff (default: '3 months')",
    )
    list_parser.set_defaults(func=cmd_list)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 289-297
```python
    labels_parser = subparsers.add_parser("labels", help="list all known labels")
    labels_parser.set_defaults(func=cmd_labels)

    collab_parser = subparsers.add_parser(
        "collaborator-check", help="check if a user is a repo collaborator"
    )
    collab_parser.add_argument("username", type=str, help="GitHub username to check")
    collab_parser.set_defaults(func=cmd_collaborator_check)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 298-310
```python
    sub_parser = subparsers.add_parser(
        "subscription", help="save/restore issue notification subscription state"
    )
    sub_sub = sub_parser.add_subparsers(dest="sub_command", required=True)
    save_parser = sub_sub.add_parser("save", help="save current subscription state")
    save_parser.add_argument("issue", type=int, help="issue number")
    save_parser.set_defaults(func=cmd_subscription_save)
    restore_parser = sub_sub.add_parser(
        "restore", help="restore saved subscription state"
    )
    restore_parser.add_argument("issue", type=int, help="issue number")
    restore_parser.set_defaults(func=cmd_subscription_restore)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 311-316
```python
    args = parser.parse_args()
    args.func(args)


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **PyTorch tooling**
  - EN: This file belongs to the pytorch tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于PyTorch 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **parse_older_than**
  - EN: `parse_older_than` is one of the main local symbols exposed or implemented here.
  - CN: `parse_older_than` 是此处暴露或实现的主要局部符号之一。
- **gh_issue_list**
  - EN: `gh_issue_list` is one of the main local symbols exposed or implemented here.
  - CN: `gh_issue_list` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `json`, `os`, `re`, `subprocess`, `sys`, `time`, `datetime`
- **Primary symbols in this file / 本文件核心符号**: `parse_older_than`, `gh_issue_list`, `gh_issue_count`, `cmd_list`, `cmd_labels`, `SUBSCRIPTION_DIR`, `gh_graphql`, `subscription_state_path`, `cmd_subscription_save`, `cmd_subscription_restore`, `cmd_collaborator_check`, `main`

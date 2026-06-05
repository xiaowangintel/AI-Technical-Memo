# update_est_time.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/update_est_time.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `update_est_time` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `update_est_time` 流程，主要负责CI 编排、测试执行。它属于 `ci` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-7: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Refresh est_time literals from sglang-ci-stats/model.json.

Usage:
    python scripts/ci/update_est_time.py [--dry-run] \\
        [--model-url URL] [--summary-file PATH]
"""
```
**EN:** Refresh est_time literals from sglang-ci-stats/model.json.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 9-15: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import re
import subprocess
import sys
from collections import defaultdict
from pathlib import Path
```
**EN:** This block loads argparse, json, re, subprocess, sys, collections, pathlib. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, re, subprocess, sys, collections, pathlib。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 17-30: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
REPO_ROOT = Path(__file__).resolve().parent.parent.parent
DEFAULT_MODEL_URL = (
    "https://raw.githubusercontent.com/sgl-project/sglang-ci-stats/main/model.json"
)

# AMD / NPU live in separate workflows and are not scraped by sglang-ci-stats.
BACKENDS = ("cuda", "cpu")

# A change is "significant" if |delta| >= this many seconds AND the relative
# change is at least SIGNIFICANT_REL_DELTA. Dual threshold filters out both
# tiny absolute drifts on long tests and small-but-noisy relative swings on
# short tests.
SIGNIFICANT_ABS_DELTA = 30
SIGNIFICANT_REL_DELTA = 0.3
```
**EN:** This section defines REPO_ROOT, DEFAULT_MODEL_URL, BACKENDS, SIGNIFICANT_ABS_DELTA, SIGNIFICANT_REL_DELTA, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 REPO_ROOT, DEFAULT_MODEL_URL, BACKENDS, SIGNIFICANT_ABS_DELTA, SIGNIFICANT_REL_DELTA，用于把可复用的默认值集中在模块顶部。

### Lines 33-43: Defines the `fetch_model` routine / 定义 `fetch_model` 例程
```python
def fetch_model(url):
    """Curl model.json. Fail loudly on network or parse errors -- the
    weekly workflow will surface the failure rather than silently making
    a no-op PR."""
    out = subprocess.run(
        ["curl", "--fail", "--silent", "--show-error", "--max-time", "30", url],
        capture_output=True,
        text=True,
        check=True,
    )
    return json.loads(out.stdout)
```
**EN:** This block defines `fetch_model`. Curl model.json. It accepts 1 parameter(s): url. Internally it loads structured data, executes external commands.
**CN:** 该代码块定义了 `fetch_model`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：url。内部会加载结构化数据、执行外部命令。

### Lines 46-67: Defines the `make_patterns` routine / 定义 `make_patterns` 例程
```python
def make_patterns(suite):
    """Yield regex objects that match `register_{backend}_ci(est_time=N, ...)`
    for the given suite, covering both registration styles:

      legacy: register_X_ci(est_time=N, suite="<full-suite>")
      new:    register_X_ci(est_time=N, stage="<stage>", runner_config="<rc>")
    """
    stage_rc = None
    if "-test-" in suite:
        stage, _, rc = suite.partition("-test-")
        stage_rc = (stage, rc)
    for backend in BACKENDS:
        yield re.compile(
            rf"(register_{backend}_ci\(est_time=)(\d+)"
            rf'(,\s*suite="{re.escape(suite)}")'
        )
        if stage_rc is not None:
            stage, rc = stage_rc
            yield re.compile(
                rf"(register_{backend}_ci\(est_time=)(\d+)"
                rf'(,\s*stage="{re.escape(stage)}",\s*runner_config="{re.escape(rc)}")'
            )
```
**EN:** This block defines `make_patterns`. Yield regex objects that match `register_{backend}_ci(est_time=N, ...)` for the given suite, covering both registration styles: legacy: register_X_ci(est_time=N, suite="<full-suite>") new: register_X_ci(est_time=N, stage="<stage>", runner_config="<rc>") It accepts 1 parameter(s): suite. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `make_patterns`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：suite。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 70-106: Defines the `update_files` routine / 定义 `update_files` 例程
```python
def update_files(model, dry_run=False):
    """Walk `model.est`, apply each p90 to the matching register call.

    Returns list of (relpath, suite, old, new) for every changed entry.
    """
    by_file = defaultdict(list)
    for suite, files in model.get("est", {}).items():
        for relpath, p90 in files.items():
            by_file[relpath].append((suite, p90))

    changes = []
    for relpath, entries in sorted(by_file.items()):
        filepath = REPO_ROOT / relpath
        if not filepath.exists():
            continue
        content = filepath.read_text()
        new_content = content

        for suite, p90 in entries:
            for pattern in make_patterns(suite):
                match = pattern.search(new_content)
                if match is None:
                    continue
                old_val = int(match.group(2))
                if old_val != p90:
                    new_content = pattern.sub(rf"\g<1>{p90}\3", new_content)
                    changes.append((relpath, suite, old_val, p90))
                    print(
                        f"  {relpath}: suite={suite!r} " f"est_time {old_val} -> {p90}",
                        file=sys.stderr,
                    )
                break  # one (file, suite) -> at most one register call

        if new_content != content and not dry_run:
            filepath.write_text(new_content)

    return changes
```
**EN:** This block defines `update_files`. Walk `model.est`, apply each p90 to the matching register call. It accepts 2 parameter(s): model, dry_run. Internally it checks file existence, emits status messages.
**CN:** 该代码块定义了 `update_files`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：model, dry_run。内部会检查文件是否存在、输出状态信息。

### Lines 109-113: Defines the `is_significant` routine / 定义 `is_significant` 例程
```python
def is_significant(old, new):
    delta = abs(new - old)
    return (
        delta >= SIGNIFICANT_ABS_DELTA and delta / max(old, 1) >= SIGNIFICANT_REL_DELTA
    )
```
**EN:** This block defines `is_significant`. It accepts 2 parameter(s): old, new. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `is_significant`。它接收 2 个参数：old, new。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 116-145: Defines the `write_summary` routine / 定义 `write_summary` 例程
```python
def write_summary(changes, summary_file):
    """Write a markdown summary of significant est_time changes."""
    sig = [c for c in changes if is_significant(c[2], c[3])]
    sig.sort(key=lambda c: abs(c[3] - c[2]), reverse=True)

    lines = []
    if sig:
        lines.append(
            f"### Significant est_time changes "
            f"({len(sig)} of {len(changes)} updates)"
        )
        lines.append("")
        lines.append("| File | Suite | Old (s) | New (s) | Δ |")
        lines.append("| --- | --- | ---: | ---: | ---: |")
        for relpath, suite, old, new in sig:
            delta = new - old
            sign = "+" if delta > 0 else ""
            pct = round(delta / max(old, 1) * 100)
            lines.append(
                f"| `{Path(relpath).name}` | `{suite}` | "
                f"{old} | {new} | {sign}{delta} ({sign}{pct}%) |"
            )
    else:
        lines.append(
            f"_{len(changes)} est_time update(s); none exceeded both "
            f"±{SIGNIFICANT_ABS_DELTA}s and "
            f"±{int(SIGNIFICANT_REL_DELTA * 100)}% thresholds._"
        )

    Path(summary_file).write_text("\n".join(lines) + "\n")
```
**EN:** This block defines `write_summary`. Write a markdown summary of significant est_time changes. It accepts 2 parameter(s): changes, summary_file. Internally it touches filesystem paths.
**CN:** 该代码块定义了 `write_summary`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：changes, summary_file。内部会处理文件系统路径。

### Lines 148-187: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument(
        "--model-url",
        default=DEFAULT_MODEL_URL,
        help="URL of model.json from sglang-ci-stats (file:// is OK for testing)",
    )
    parser.add_argument(
        "--dry-run",
        action="store_true",
        help="Print changes without modifying files",
    )
    parser.add_argument(
        "--summary-file",
        default=None,
        help="Write a markdown summary of significant changes to this path",
    )
    args = parser.parse_args()

    print(f"Fetching {args.model_url}", file=sys.stderr)
    model = fetch_model(args.model_url)
    print(
        f"  model data_as_of={model.get('data_as_of')} "
        f"n_runs={model.get('n_runs')} "
        f"n_suites={len(model.get('est', {}))}",
        file=sys.stderr,
    )

    changes = update_files(model, dry_run=args.dry_run)

    n_files = len({c[0] for c in changes})
    action = "Would update" if args.dry_run else "Updated"
    print(
        f"\n{action} {len(changes)} est_time entries across {n_files} files",
        file=sys.stderr,
    )

    if args.summary_file:
        write_summary(changes, args.summary_file)
        print(f"Wrote summary to {args.summary_file}", file=sys.stderr)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 190-191: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `collections`, `json`, `pathlib`, `re`, `subprocess`, `sys`

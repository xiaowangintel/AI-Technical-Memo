# compute_partitions.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/compute_partitions.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `compute_partitions` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `compute_partitions` 流程，主要负责CI 编排、测试执行。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Sum est_time per per-commit suite and emit one $GITHUB_OUTPUT line
keyed by suite name. Consumed by pr-test.yml stage jobs as
`fromJson(needs.check-changes.outputs.partitions)['<suite>']`.

    partitions={"base-b-test-1-gpu-small": {"size": 8, "arr": [0,...,7], "max_parallel": 2}, ...}
"""
```
**EN:** Sum est_time per per-commit suite and emit one $GITHUB_OUTPUT line keyed by suite name.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 8-16: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import glob
import importlib.util
import json
import math
import os
from collections import defaultdict

import yaml  # PyYAML; preinstalled on ubuntu-latest GHA runners.
```
**EN:** This block loads argparse, glob, importlib.util, json, math, os, collections, yaml  # PyYAML; preinstalled on ubuntu-latest GHA runners.. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, glob, importlib.util, json, math, os, collections, yaml  # PyYAML; preinstalled on ubuntu-latest GHA runners.。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 18-29: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
REPO_ROOT = os.path.dirname(
    os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
)

# Load ci_register.py directly: `import sglang.test...` pulls torch/orjson via
# sglang.__init__ but check-changes runs on bare ubuntu-latest. ci_register
# itself is stdlib-only (AST).
_CI_REGISTER_PATH = os.path.join(
    REPO_ROOT, "python", "sglang", "test", "ci", "ci_register.py"
)
_spec = importlib.util.spec_from_file_location("ci_register", _CI_REGISTER_PATH)
_ci_register = importlib.util.module_from_spec(_spec)
```
**EN:** This section defines REPO_ROOT, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 REPO_ROOT，用于把可复用的默认值集中在模块顶部。

### Lines 30-30: Top-level expr logic / 顶层 expr 逻辑
```python
_spec.loader.exec_module(_ci_register)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 31-44: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
collect_tests = _ci_register.collect_tests
HWBackend = _ci_register.HWBackend

# pr-test-amd.yml / pr-test-npu.yml have their own dispatch.
_TARGET_BACKENDS = {HWBackend.CUDA, HWBackend.CPU}

# base-a is the critical-path entry gate; pin its fanout to smoke-coverage
# defaults instead of est_time. max_parallel = size (no throttle).
_BASE_A_OVERRIDES = {
    "base-a-test-cpu": 4,
    "base-a-test-1-gpu-small": 1,
}

_REUSABLE_STAGE_USES = "./.github/workflows/_pr-test-stage.yml"
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 47-66: Defines the `load_run_timeouts` routine / 定义 `load_run_timeouts` 例程
```python
def load_run_timeouts(pr_test_yml_path: str) -> dict:
    """Map `self_name -> run_timeout_minutes` from one pr-test*.yml. The input
    is required in `_pr-test-stage.yml` -- KeyError surfaces missing.
    Inline base-a-test-cpu is skipped (uses `_BASE_A_OVERRIDES`)."""
    with open(pr_test_yml_path) as f:
        wf = yaml.safe_load(f)
    timeouts = {}
    for job_id, job in (wf.get("jobs") or {}).items():
        if not isinstance(job, dict) or job.get("uses") != _REUSABLE_STAGE_USES:
            continue
        with_ = job.get("with") or {}
        suite = with_.get("self_name", job_id)
        timeouts[suite] = int(with_["run_timeout_minutes"])
    if not timeouts:
        raise RuntimeError(
            f"load_run_timeouts: no jobs matched uses={_REUSABLE_STAGE_USES!r} "
            f"in {pr_test_yml_path}. The reusable workflow path likely "
            "changed -- update _REUSABLE_STAGE_USES."
        )
    return timeouts
```
**EN:** This block defines `load_run_timeouts`. Map `self_name -> run_timeout_minutes` from one pr-test*.yml. It accepts 1 parameter(s): pr_test_yml_path. Internally it reads or writes files, loads structured configuration.
**CN:** 该代码块定义了 `load_run_timeouts`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：pr_test_yml_path。内部会读写文件、加载结构化配置。

### Lines 69-73: Defines the `per_shard_target_seconds` routine / 定义 `per_shard_target_seconds` 例程
```python
def per_shard_target_seconds(suite: str, run_timeouts: dict) -> float:
    """Per-shard wall budget = 0.75 * stage timeout. 0.75 is the inverse
    of LPT's 4/3 worst-case approximation ratio, so the most imbalanced
    LPT shard fills exactly the timeout."""
    return 0.75 * run_timeouts[suite] * 60
```
**EN:** This block defines `per_shard_target_seconds`. Per-shard wall budget = 0.75 * stage timeout. It accepts 2 parameter(s): suite, run_timeouts. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `per_shard_target_seconds`。其文档字符串说明了该逻辑的预期职责。它接收 2 个参数：suite, run_timeouts。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 76-92: Defines the `discover_files` routine / 定义 `discover_files` 例程
```python
def discover_files(repo_root: str) -> list[str]:
    test_dir = os.path.join(repo_root, "test")
    files = [
        f
        for f in glob.glob(
            os.path.join(test_dir, "registered", "**", "*.py"), recursive=True
        )
        if not f.endswith("/conftest.py") and not f.endswith("/__init__.py")
    ]
    jit_kernel_dir = os.path.join(repo_root, "python", "sglang", "jit_kernel")
    files += glob.glob(
        os.path.join(jit_kernel_dir, "tests", "**", "test_*.py"), recursive=True
    )
    files += glob.glob(
        os.path.join(jit_kernel_dir, "benchmark", "**", "bench_*.py"), recursive=True
    )
    return files
```
**EN:** This block defines `discover_files`. It accepts 1 parameter(s): repo_root. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `discover_files`。它接收 1 个参数：repo_root。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 95-105: Defines the `load_partition_model` routine / 定义 `load_partition_model` 例程
```python
def load_partition_model(path):
    """Read sglang-ci-stats' model.json; None on missing/unparsable.
    Cross-repo schema -- guard against non-dict top-level."""
    if not path or not os.path.exists(path):
        return None
    try:
        with open(path) as f:
            data = json.load(f)
    except (OSError, json.JSONDecodeError):
        return None
    return data if isinstance(data, dict) else None
```
**EN:** This block defines `load_partition_model`. Read sglang-ci-stats' model.json; None on missing/unparsable. It accepts 1 parameter(s): path. Internally it loads structured data, reads or writes files, checks file existence.
**CN:** 该代码块定义了 `load_partition_model`。其文档字符串说明了该逻辑的预期职责。它接收 1 个参数：path。内部会加载结构化数据、读写文件、检查文件是否存在。

### Lines 108-109: Defines the `compute_max_parallel` routine / 定义 `compute_max_parallel` 例程
```python
def compute_max_parallel(size: int) -> int:
    return max(size // 3, 1)
```
**EN:** This block defines `compute_max_parallel`. It accepts 1 parameter(s): size. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `compute_max_parallel`。它接收 1 个参数：size。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 112-179: Defines the `compute_partitions` routine / 定义 `compute_partitions` 例程
```python
def compute_partitions(
    tests, repo_root, run_timeouts, partition_model=None, full_parallel=False
):
    """Group per-commit tests by suite and emit partition metadata.

    `run_timeouts`: `suite -> minutes` from `load_run_timeouts`.
    `partition_model`: optional sglang-ci-stats `model.json`; per-file
    `est` and per-suite `(coeff, bias)` each fall back independently to
    in-source `est_time` / `(1.0, 0.0)`.
    `full_parallel=True` lifts the matrix-fanout throttle.
    """
    # Allowlist: stages pr-test.yml dispatches. Stress / weekly /
    # nightly-* live in test/registered/ but pr-test doesn't run them.
    dispatched_suites = set(run_timeouts) | set(_BASE_A_OVERRIDES)
    suite_tests = defaultdict(list)
    for t in tests:
        if t.backend not in _TARGET_BACKENDS:
            continue
        if t.nightly or t.disabled is not None:
            continue
        if t.effective_suite not in dispatched_suites:
            continue
        suite_tests[t.effective_suite].append(t)

    est_table = (partition_model or {}).get("est", {})
    fit_table = (partition_model or {}).get("fit", {})

    result = {}
    for suite, group in suite_tests.items():
        live_est = est_table.get(suite, {})
        total = 0.0
        for t in group:
            relpath = os.path.relpath(t.filename, repo_root)
            total += live_est.get(relpath, t.est_time)

        fit = fit_table.get(suite) or {}
        coeff = fit.get("coeff", 1.0)
        bias = fit.get("bias", 0.0)

        # Each shard pays `bias` once, so size >= coeff*total / (target-bias).
        if suite in _BASE_A_OVERRIDES:
            size = _BASE_A_OVERRIDES[suite]
            max_parallel = size
        else:
            target = per_shard_target_seconds(suite, run_timeouts)
            budget = target - bias
            if budget <= 0:
                raise RuntimeError(
                    f"Suite {suite!r}: fit bias={bias}s >= target={target}s. "
                    "Investigate the fit or raise the stage's run_timeout_minutes."
                )
            ideal_size = math.ceil(coeff * total / budget)
            # ideal_size > len(group) -> slowest single file alone exceeds
            # the per-shard budget; surface via raise instead of empty shard.
            if ideal_size > len(group):
                raise RuntimeError(
                    f"Suite {suite!r}: needs {ideal_size} shards but has only "
                    f"{len(group)} test file(s). target={target:.0f}s, "
                    f"coeff={coeff}, bias={bias}s, total_est={total:.0f}s."
                )
            size = max(1, ideal_size)
            max_parallel = size if full_parallel else compute_max_parallel(size)
        result[suite] = {
            "size": size,
            "arr": list(range(size)),
            "max_parallel": max_parallel,
        }
    return result
```
**EN:** This block defines `compute_partitions`. Group per-commit tests by suite and emit partition metadata. It accepts 5 parameter(s): tests, repo_root, run_timeouts, partition_model, full_parallel. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `compute_partitions`。其文档字符串说明了该逻辑的预期职责。它接收 5 个参数：tests, repo_root, run_timeouts, partition_model, full_parallel。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 182-247: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--repo-root", default=REPO_ROOT)
    parser.add_argument(
        "--output-format",
        choices=("gha", "json"),
        default="gha",
        help="`gha` emits `partitions=<json>` for $GITHUB_OUTPUT; `json` is raw",
    )
    parser.add_argument(
        "--full-parallel",
        choices=("true", "false"),
        default="false",
        help="Lift the max_parallel throttle (set by schedule / `high priority`)",
    )
    parser.add_argument(
        "--partition-model-file",
        default=None,
        help="Path to sglang-ci-stats model.json (omit/missing -> static fallback)",
    )
    parser.add_argument(
        "--pr-test-yml",
        default=os.path.join(REPO_ROOT, ".github", "workflows", "pr-test.yml"),
        help="Path to pr-test*.yml; per-stage `run_timeout_minutes` is read from here.",
    )
    args = parser.parse_args()

    files = discover_files(args.repo_root)
    # Warn-not-fail on unregistered files: run_suite.py catches this at
    # test-execution time with sanity_check=True; dispatch should keep going.
    all_tests = collect_tests(files, sanity_check=False)
    partition_model = load_partition_model(args.partition_model_file)
    run_timeouts = load_run_timeouts(args.pr_test_yml)

    result = compute_partitions(
        all_tests,
        repo_root=args.repo_root,
        run_timeouts=run_timeouts,
        partition_model=partition_model,
        full_parallel=(args.full_parallel == "true"),
    )
    payload = json.dumps(result, separators=(",", ":"), sort_keys=True)
    if args.output_format == "gha":
        print(f"partitions={payload}")
    else:
        print(payload)

    summary_path = os.environ.get("GITHUB_STEP_SUMMARY")
    if summary_path:
        with open(summary_path, "a") as f:
            f.write("## Partitions\n\n")
            if partition_model is None:
                src_note = "no live model -- static est_time + (coeff=1, bias=0)"
            else:
                src_note = (
                    f"live model `data_as_of={partition_model.get('data_as_of')}`, "
                    f"`n_runs={partition_model.get('n_runs')}`"
                )
            f.write(
                f"`full_parallel={args.full_parallel}` "
                f"(`size//3` throttle is lifted when true); {src_note}\n\n"
            )
            f.write("| Suite | size | max_parallel |\n")
            f.write("|---|---:|---:|\n")
            for suite, info in sorted(result.items()):
                f.write(f"| `{suite}` | {info['size']} | {info['max_parallel']} |\n")
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it reads or writes files, parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会读写文件、解析命令行参数、输出状态信息。

### Lines 250-251: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **YAML configuration** / YAML 配置
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `collections`, `glob`, `importlib`, `json`, `math`, `os`
- **Third-party modules / 第三方模块**: `yaml`
- **External commands inferred from code / 从代码推断的外部命令**: `python`

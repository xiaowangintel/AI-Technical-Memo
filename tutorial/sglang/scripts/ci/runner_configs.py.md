# runner_configs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/runner_configs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `runner_configs` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `runner_configs` 流程，主要负责CI 编排、测试执行。它属于 `ci` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Emit runner_config setup for GitHub Actions $GITHUB_OUTPUT.

runner_configs.py <runner_config>
    Per-field `key=value` lines (install / artifact_version /
    install_timeout / rdma_devices). `runs_on` is intentionally omitted —
    it carries the `$b200_runner` sentinel and is resolved via --map.
    Called per stage by _pr-test-stage.yml.

runner_configs.py --map <b200_runner_label>
    `runs_on_map={json}` — flat dict {runner_config: runs_on}, with
    `$b200_runner` substituted. Called once by _pr-test-check-changes.yml.
"""
```
**EN:** Emit runner_config setup for GitHub Actions $GITHUB_OUTPUT.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 14-18: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import json
import os
import sys

import yaml
```
**EN:** This block loads json, os, sys, yaml. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 json, os, sys, yaml。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 20-21: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
_YAML_PATH = os.path.join(os.path.dirname(__file__), "runner_configs.yml")
_B200_SENTINEL = "$b200_runner"
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 24-26: Defines the `load` routine / 定义 `load` 例程
```python
def load() -> dict:
    with open(_YAML_PATH) as f:
        return yaml.safe_load(f)["runner_configs"]
```
**EN:** This block defines `load`. It takes no explicit parameters at the top level. Internally it reads or writes files, loads structured configuration.
**CN:** 该代码块定义了 `load`。它在顶层定义中不接收显式参数。内部会读写文件、加载结构化配置。

### Lines 29-38: Defines the `_emit_single` routine / 定义 `_emit_single` 例程
```python
def _emit_single(rc: str) -> None:
    # runs_on goes through --map (resolves $b200_runner). Suppress it here so a
    # consumer can't accidentally read the raw sentinel value.
    cfg = load().get(rc)
    if cfg is None:
        sys.exit(f"unknown runner_config: {rc!r}")
    for key, value in cfg.items():
        if key == "runs_on":
            continue
        print(f"{key}={value}")
```
**EN:** This block defines `_emit_single`. It accepts 1 parameter(s): rc. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `_emit_single`。它接收 1 个参数：rc。内部会输出状态信息、控制进程退出状态。

### Lines 41-46: Defines the `_emit_map` routine / 定义 `_emit_map` 例程
```python
def _emit_map(b200_runner: str) -> None:
    runs_on = {
        name: (b200_runner if cfg.get("runs_on") == _B200_SENTINEL else cfg["runs_on"])
        for name, cfg in load().items()
    }
    print(f"runs_on_map={json.dumps(runs_on, separators=(',', ':'))}")
```
**EN:** This block defines `_emit_map`. It accepts 1 parameter(s): b200_runner. Internally it emits status messages.
**CN:** 该代码块定义了 `_emit_map`。它接收 1 个参数：b200_runner。内部会输出状态信息。

### Lines 49-60: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    args = sys.argv[1:]
    if len(args) == 1:
        _emit_single(args[0])
    elif len(args) == 2 and args[0] == "--map":
        _emit_map(args[1])
    else:
        sys.exit(
            "usage:\n"
            "  runner_configs.py <runner_config>\n"
            "  runner_configs.py --map <b200_runner_label>"
        )
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It controls process exit status.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会控制进程退出状态。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **YAML configuration** / YAML 配置
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `sys`
- **Third-party modules / 第三方模块**: `yaml`

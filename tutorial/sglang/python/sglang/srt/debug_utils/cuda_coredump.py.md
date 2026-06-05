# cuda_coredump.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/cuda_coredump.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Document the module intent / 说明模块意图
```python
"""CUDA coredump helpers.

When SGLANG_CUDA_COREDUMP=1, this module injects CUDA coredump environment
variables into the current process so that GPU exceptions (e.g. illegal
memory access) produce lightweight coredump files for post-mortem analysis
with cuda-gdb.

The injection happens at module import time via _inject_env() on a
best-effort basis.  If any CUDA_* variable is already present in the
environment (e.g. set by the user in the shell), injection is skipped for
that variable and a warning is printed.  For strict guarantees, set the
CUDA_* env vars in the shell before launching Python.
"""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 15-19: Import dependencies and shared types / 导入依赖与共享类型
```python
import glob
import os
import warnings

from sglang.srt.environ import envs
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 21-24: Declare module-level symbols such as `_CUDA_COREDUMP_FLAGS` / 声明模块级符号，例如 `_CUDA_COREDUMP_FLAGS`
```python
_CUDA_COREDUMP_FLAGS = (
    "skip_nonrelocated_elf_images,skip_global_memory,"
    "skip_shared_memory,skip_local_memory,skip_constbank_memory"
)
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 27-28: Implement function `is_enabled` / 实现函数 `is_enabled`
```python
def is_enabled() -> bool:
    return envs.SGLANG_CUDA_COREDUMP.get()
```
**EN:** Function `is_enabled` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `is_enabled` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 31-32: Implement function `get_dump_dir` / 实现函数 `get_dump_dir`
```python
def get_dump_dir() -> str:
    return envs.SGLANG_CUDA_COREDUMP_DIR.get()
```
**EN:** Function `get_dump_dir` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `get_dump_dir` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 35-55: Implement helper `_inject_env` / 实现辅助函数 `_inject_env`
```python
def _inject_env():
    """Inject CUDA coredump environment variables into the current process.
    If a CUDA_* variable is already present, skip it and log a warning."""
    dump_dir = get_dump_dir()
    os.makedirs(dump_dir, exist_ok=True)

    env_vars = {
        "CUDA_ENABLE_COREDUMP_ON_EXCEPTION": "1",
        "CUDA_COREDUMP_SHOW_PROGRESS": "1",
        "CUDA_COREDUMP_GENERATION_FLAGS": _CUDA_COREDUMP_FLAGS,
        "CUDA_COREDUMP_FILE": f"{dump_dir}/cuda_coredump_%h.%p.%t",
    }
    for key, value in env_vars.items():
        if key in os.environ:
            warnings.warn(
                f"CUDA coredump env var {key} is already set to "
                f"'{os.environ[key]}', skipping injection of '{value}'.",
                stacklevel=2,
            )
        else:
            os.environ[key] = value
```
**EN:** Function `_inject_env` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_inject_env` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 58-62: Implement function `cleanup_dump_dir` / 实现函数 `cleanup_dump_dir`
```python
def cleanup_dump_dir():
    """Remove stale coredump files from the dump directory."""
    dump_dir = get_dump_dir()
    for f in glob.glob(os.path.join(dump_dir, "cuda_coredump_*")):
        os.remove(f)
```
**EN:** Function `cleanup_dump_dir` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `cleanup_dump_dir` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 65-84: Implement function `report` / 实现函数 `report`
```python
def report():
    """Log any CUDA coredump files found after a test failure."""
    dump_dir = get_dump_dir()
    coredump_files = glob.glob(os.path.join(dump_dir, "cuda_coredump_*"))
    if not coredump_files:
        return

    print(f"\n{'='*60}")
    print(f"CUDA coredump(s) detected ({len(coredump_files)} file(s)):")
    for f in coredump_files:
        size_mb = os.path.getsize(f) / (1024 * 1024)
        print(f"  {f} ({size_mb:.1f} MB)")
    print("Use cuda-gdb to analyze: cuda-gdb -c <coredump_file>")

    run_id = os.environ.get("GITHUB_RUN_ID")
    if run_id:
        repo = os.environ.get("GITHUB_REPOSITORY", "sgl-project/sglang")
        print(f"Download from CI: gh run download {run_id} --repo {repo}")

    print(f"{'='*60}\n")
```
**EN:** Function `report` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `report` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 90-90: Declare module-level symbols such as `_SENTINEL` / 声明模块级符号，例如 `_SENTINEL`
```python
_SENTINEL = "_SGLANG_CUDA_COREDUMP_INJECTED"
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 92-95: Handle conditional module logic / 处理条件模块逻辑
```python
if is_enabled() and _SENTINEL not in os.environ:
    os.environ[_SENTINEL] = "1"
    print(f"Injecting CUDA coredump env vars (pid={os.getpid()})")
    _inject_env()
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `is_enabled`, `get_dump_dir`, `_inject_env`, `cleanup_dump_dir`, `report`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `glob`, `os`, `warnings`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.environ`

# ray_env.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/ray/ray_env.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `ray_env`-related logic centered around `_parse_csv`, `get_env_vars_to_copy`. / 实现与 `ray_env` 相关的逻辑，核心符号包括 `_parse_csv`, `get_env_vars_to_copy`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-47)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import json
import os

import vllm.envs as envs
from vllm.logger import init_logger

logger = init_logger(__name__)

CONFIG_HOME = envs.VLLM_CONFIG_ROOT

# Env vars that should NOT be copied from the driver to Ray workers.
RAY_NON_CARRY_OVER_ENV_VARS_FILE = os.path.join(
    CONFIG_HOME, "ray_non_carry_over_env_vars.json"
)

try:
    if os.path.exists(RAY_NON_CARRY_OVER_ENV_VARS_FILE):
        with open(RAY_NON_CARRY_OVER_ENV_VARS_FILE) as f:
            RAY_NON_CARRY_OVER_ENV_VARS = set(json.load(f))
    else:
        RAY_NON_CARRY_OVER_ENV_VARS = set()
except json.JSONDecodeError:
    # ...
    "HUGGING_FACE_",
}

DEFAULT_EXTRA_ENV_VARS: set[str] = {
    "PYTHONHASHSEED",
}
```
**EN:** Sets up the module with standard-library support such as `json`, `os`, vLLM modules such as `vllm.envs`, `vllm.logger`. It prepares the symbols later used by `_parse_csv`, `get_env_vars_to_copy`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `_parse_csv`, `get_env_vars_to_copy` 提供上下文。

### _parse_csv (lines 50-52)
```python
def _parse_csv(value: str) -> set[str]:
    """Split a comma-separated string into a set of stripped, non-empty tokens."""
    return {tok.strip() for tok in value.split(",") if tok.strip()}
```
**EN:** `_parse_csv`: Split a comma-separated string into a set of stripped, non-empty tokens. It mainly works with `value`. Inside the body, it relies on `tok.strip`, `value.split` to complete the main steps.
**CN:** `_parse_csv` 负责把原始输入解析为结构化对象。 它主要处理 `value` 等参数。 实现过程中会调用 `tok.strip`, `value.split` 等函数完成关键步骤。

### get_env_vars_to_copy (lines 55-116)
```python
def get_env_vars_to_copy(
    exclude_vars: set[str] | None = None,
    additional_vars: set[str] | None = None,
    destination: str | None = None,
) -> set[str]:
    """Return the env var names to copy from the driver to Ray actors.

    The result is the union of:

    1. Env vars registered in ``vllm.envs.environment_variables``.
    2. Env vars in ``os.environ`` matching a prefix in
       ``DEFAULT_ENV_VAR_PREFIXES`` + ``VLLM_RAY_EXTRA_ENV_VAR_PREFIXES_TO_COPY``.
    3. Individual names in ``DEFAULT_EXTRA_ENV_VARS`` +
       ``VLLM_RAY_EXTRA_ENV_VARS_TO_COPY``.
    4. Caller-supplied *additional_vars* (e.g. platform-specific).

    Minus any names in *exclude_vars* or ``RAY_NON_CARRY_OVER_ENV_VARS``.

    Args:
        exclude_vars: Env vars to exclude (e.g. worker-specific ones).
        additional_vars: Extra individual env var names to copy.  Useful
            for caller-specific vars (e.g. platform env vars).
        destination: Label used in log messages only.
    """
    # ...
    logger.info(
        "To exclude env vars from copying, add them to %s",
        RAY_NON_CARRY_OVER_ENV_VARS_FILE,
    )

    return result
```
**EN:** `get_env_vars_to_copy`: Return the env var names to copy from the driver to Ray actors. It mainly works with `exclude_vars`, `additional_vars`, `destination`. Inside the body, it relies on `logger.info`, `_parse_csv`, `sorted` to complete the main steps.
**CN:** `get_env_vars_to_copy` 负责获取流水线所需的数据或状态。 它主要处理 `exclude_vars`, `additional_vars`, `destination` 等参数。 实现过程中会调用 `logger.info`, `_parse_csv`, `sorted` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`_parse_csv`**: Key helper or entry point in this file. / **`_parse_csv`**：本文件中的关键辅助函数或入口。
- **`get_env_vars_to_copy`**: Key helper or entry point in this file. / **`get_env_vars_to_copy`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: json, os
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger

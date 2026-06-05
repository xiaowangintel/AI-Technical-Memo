# test_mrcr_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/evals/mrcr/test_mrcr_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MRCR long-context accuracy test. / 该文件主要围绕 MRCR Correctness 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-17)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
MRCR long-context accuracy test.

Usage:
    pytest -s -v tests/evals/mrcr/test_mrcr_correctness.py \
        --config-list-file=configs/models-small.txt
"""

import shlex

import yaml

from tests.utils import RemoteOpenAIServer

from .mrcr_eval import evaluate_mrcr
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `shlex`, `yaml`, `tests.utils`, `.mrcr_eval`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _split_host_port (lines 20-27)
```python
def _split_host_port(url: str, default_port: int = 8000) -> tuple[str, int]:
    if "://" in url:
        url = url.split("://", 1)[1]
    host_port = url.split("/", 1)[0]
    if ":" in host_port:
        host, p = host_port.split(":", 1)
        return f"http://{host}", int(p)
    return f"http://{host_port}", default_port
```
**EN:** Implements a reusable helper for Split Host Port, reducing duplication across related tests. It coordinates operations such as `url.split`, `host_port.split`, `int`.
**CN:** 该辅助函数为 Split Host Port 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `url.split`, `host_port.split`, `int` 等操作。

### Test: test_mrcr_correctness (lines 30-83)
```python
def test_mrcr_correctness(config_filename):
    cfg = yaml.safe_load(config_filename.read_text(encoding="utf-8"))

    server_args = shlex.split(cfg.get("server_args", ""))
    server_args += ["--trust-remote-code", "--disable-uvicorn-access-log"]

    print(
        f"MRCR eval for {cfg['model_name']} (threshold {cfg['match_ratio_threshold']})"
    )

    with RemoteOpenAIServer(
        cfg["model_name"],
        server_args,
        env_dict=cfg.get("env"),
        max_wait_seconds=cfg.get("startup_max_wait_seconds", 600),
    ) as server:
        host, port = _split_host_port(server.url_for("v1"))
        results = evaluate_mrcr(
            model_name=cfg.get("model_name"),
# ... omitted for brevity ...
            key = f"match_ratio_n{int(n)}"
            measured = results["per_needle"].get(key)
            if measured is None:
                failures.append(f"{key}: no samples collected")
            elif measured < expected - tol:
                failures.append(f"{key}: {measured:.4f} < {expected:.4f} - {tol:.4f}")
    else:
        measured = results["match_ratio"]
        if measured < threshold - tol:
            failures.append(
                f"match_ratio: {measured:.4f} < {threshold:.4f} - {tol:.4f}"
            )

    assert not failures, "MRCR thresholds failed: " + "; ".join(failures)
```
**EN:** Checks MRCR Correctness under a focused test scenario. The body exercises logic via `yaml.safe_load`, `shlex.split`, `print` before asserting the expected outcome.
**CN:** 该测试用例验证 MRCR Correctness 在特定场景下的行为。 函数体会先通过 `yaml.safe_load`, `shlex.split`, `print` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `shlex`
- **Third-party / 第三方依赖**: `yaml`
- **Local test utilities / 本地测试辅助**: `tests.utils`, `.mrcr_eval`

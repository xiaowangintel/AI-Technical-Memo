# test_cross_node_scheduler_info_sync.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_cross_node_scheduler_info_sync.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `cross node scheduler info sync` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual` 中的 `cross node scheduler info sync` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: Process control logic / 进程控制逻辑
```python
#!/usr/bin/env python3
"""
Test cross-node scheduler_infos synchronization for remote weight loading.

Simulates multi-node setups on a single machine using different GPU subsets.
Validates that scheduler_infos are correctly synced across nodes via Gloo.

IMPORTANT: For multi-node tests, start both nodes within a few seconds of each
other to avoid port binding conflicts (they share the same network namespace).

Test cases:
  - tp4_nodes2: TP=4 across 2 nodes, validates basic cross-node sync
  - dp2_single_node: DP=2 with dp_attention on single node
  - dp2_tp2_nodes2: DP=2, TP=4 across 2 nodes with dp_attention

Usage (multi-node):
    Terminal 1: python test_cross_node_scheduler_info_sync.py --test-case tp4_nodes2 --node-rank 0
    Terminal 2: python test_cross_node_scheduler_info_sync.py --test-case tp4_nodes2 --node-rank 1
    Terminal 3: python test_cross_node_scheduler_info_sync.py --test-case tp4_nodes2 --test-only

Usage (single-node):
    Terminal 1: python test_cross_node_scheduler_info_sync.py --test-case dp2_single_node --node-rank 0
    Terminal 2: python test_cross_node_scheduler_info_sync.py --test-case dp2_single_node --test-only

Requirements: 4 GPUs on single machine
"""

import argparse
import socket
import subprocess
import sys
import time
from dataclasses import dataclass
from typing import List

import requests

from sglang.test.test_utils import (
    DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
)


@dataclass
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `conflicts` and `Usage`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 44-51: Class definition for TestCase / 类定义
```python
class TestCase:
    name: str
    tp_size: int
    dp_size: int
    nnodes: int
    gpus_per_node: int
    expected_ranks: int
    extra_args: List[str]
```
**EN:** This range declares `TestCase`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 52-90: Constants and scenario settings / 常量与场景配置
```python


TEST_CASES = {
    "tp4_nodes2": TestCase(
        name="tp4_nodes2",
        tp_size=4,
        dp_size=1,
        nnodes=2,
        gpus_per_node=2,
        expected_ranks=4,
        extra_args=[],
    ),
    "dp2_single_node": TestCase(
        name="dp2_single_node",
        tp_size=2,
        dp_size=2,
        nnodes=1,
        gpus_per_node=2,
        expected_ranks=2,
        extra_args=["--enable-dp-attention", "--dp", "2", "--attention-backend", "fa3"],
    ),
    "dp2_tp2_nodes2": TestCase(
        name="dp2_tp2_nodes2",
        tp_size=4,
        dp_size=2,
        nnodes=2,
        gpus_per_node=2,
        expected_ranks=4,
        extra_args=["--enable-dp-attention", "--dp", "2", "--attention-backend", "fa3"],
    ),
}

TEST_CASE_MODELS = {
    "tp4_nodes2": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
    "dp2_single_node": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
    "dp2_tp2_nodes2": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `TestCase`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 91-99: Helper routines around get_local_ip / 辅助例程
```python
def get_local_ip() -> str:
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    try:
        s.connect(("8.8.8.8", 80))
        return s.getsockname()[0]
    except Exception:
        return "127.0.0.1"
    finally:
        s.close()
```
**EN:** This range implements helper routine(s) `get_local_ip` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `socket`, `connect`, `getsockname` and `close`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 102-126: Helper routines around launch_node / 辅助例程
```python
def launch_node(
    test_case: TestCase, node_rank: int, model_path: str, dist_init_addr: str
):
    cmd = [
        sys.executable,
        "-m",
        "sglang.launch_server",
        "--model-path",
        model_path,
        "--tp",
        str(test_case.tp_size),
        "--port",
        str(30000 + node_rank * 100),
        "--host",
        "0.0.0.0",
        "--remote-instance-weight-loader-start-seed-via-transfer-engine",
    ]
    if test_case.nnodes > 1:
        cmd.extend(
            [
                "--nnodes",
                str(test_case.nnodes),
                "--node-rank",
                str(node_rank),
                "--dist-init-addr",
```
**EN:** This range implements helper routine(s) `launch_node` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `extend`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 127-134: Process control logic / 进程控制逻辑
```python
                dist_init_addr,
                "--base-gpu-id",
                str(node_rank * test_case.gpus_per_node),
            ]
        )
    cmd.extend(test_case.extra_args)
    print(f"[Node {node_rank}] {' '.join(cmd)}")
    subprocess.run(cmd)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `extend`, `join` and `run`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 137-161: Test routines around test_api / 测试例程
```python
def test_api(test_case: TestCase) -> bool:
    base_url = "http://127.0.0.1:30000"
    print(f"Testing {test_case.name}: expecting {test_case.expected_ranks} ranks")

    for _ in range(60):
        try:
            if requests.get(f"{base_url}/health", timeout=2).status_code == 200:
                break
        except Exception:
            pass
        time.sleep(2)
    else:
        print("ERROR: Server not ready")
        return False

    all_passed = True
    for rank in range(test_case.expected_ranks):
        try:
            resp = requests.get(
                f"{base_url}/get_remote_instance_transfer_engine_info",
                params={"rank": rank},
                timeout=5,
            )
            status = "✓" if resp.status_code == 200 else "✗"
            print(f"{status} Rank {rank}: {resp.status_code}")
```
**EN:** This range defines concrete test routine(s) `test_api`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get` and `sleep`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 162-169: Scenario logic / 场景逻辑
```python
            if resp.status_code != 200:
                all_passed = False
        except Exception as e:
            print(f"✗ Rank {rank}: {e}")
            all_passed = False

    print("PASSED" if all_passed else "FAILED")
    return all_passed
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 172-196: Helper routines around main / 辅助例程
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--test-case", type=str, choices=list(TEST_CASES.keys()), required=True
    )
    parser.add_argument("--node-rank", type=int, choices=[0, 1])
    parser.add_argument("--model-path", type=str, default=None)
    parser.add_argument("--dist-init-addr", type=str, default=None)
    parser.add_argument("--test-only", action="store_true")
    args = parser.parse_args()

    test_case = TEST_CASES[args.test_case]
    model_path = args.model_path or TEST_CASE_MODELS.get(
        args.test_case, DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT
    )

    if args.test_only:
        sys.exit(0 if test_api(test_case) else 1)

    if test_case.nnodes == 1:
        launch_node(test_case, 0, model_path, "")
        return

    if args.node_rank is None:
        print(f"Usage: --node-rank 0 or 1, then --test-only in another terminal")
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `ArgumentParser`, `add_argument`, `keys` and `parse_args`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 197-200: Scenario logic / 场景逻辑
```python
        sys.exit(0)

    dist_init_addr = args.dist_init_addr or f"{get_local_ip()}:20000"
    launch_node(test_case, args.node_rank, model_path, dist_init_addr)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `exit`, `get_local_ip` and `launch_node`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 201-204: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `argparse`, `dataclasses`, `socket`, `subprocess`, `sys`, `time`, `typing`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.test.test_utils`

# test_tensor_dump_forward_hook.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_tensor_dump_forward_hook.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils tensor dump forward hook in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 debug utils tensor dump forward hook 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies
```python
import unittest

import torch
from torch import nn

from sglang.srt.debug_utils.tensor_dump_forward_hook import (
    register_forward_hook_for_model,
)
from sglang.srt.distributed.parallel_state import (
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import LinearBase
from sglang.srt.models.qwen2 import Qwen2MLP
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.srt.utils import add_prefix
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 20-30: Register CI metadata
```python
register_cuda_ci(
    est_time=9,
    stage="base-b",
    runner_config="1-gpu-small",
    disabled="Test uses pytest-style function without TestCase class - see #17145",
)
register_amd_ci(
    est_time=15,
    suite="stage-b-test-1-gpu-small-amd",
    disabled="Test uses pytest-style function without TestCase class - see #17145",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 32-32: Define module constants
```python
TEST_HIDDEN_SIZE = 32
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 35-35: Define class SimpleModel
```python
class SimpleModel(nn.Module):
```
**EN:** This declaration introduces the `SimpleModel` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `SimpleModel` 测试类，并说明它通过继承承担的职责。

### Lines 37-48: Define helper: init
```python
    def __init__(self) -> None:
        super().__init__()
        self.hidden_size = TEST_HIDDEN_SIZE
        self.rms_norm_eps = 1e-5
        self.mlp = Qwen2MLP(
            hidden_size=self.hidden_size,
            intermediate_size=self.hidden_size,
            hidden_act="silu",
            quant_config=None,
            prefix=add_prefix("mlp", ""),
        )
        self.layernorm = RMSNorm(self.hidden_size, eps=self.rms_norm_eps)
```
**EN:** This helper function encapsulates reusable logic inside `SimpleModel` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `SimpleModel` 内部调用，从而让场景结构更清晰。

### Lines 50-57: Define helper: forward
```python
    @torch.no_grad()
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        hidden_states = self.layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        return hidden_states
```
**EN:** This helper function encapsulates reusable logic inside `SimpleModel` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `SimpleModel` 内部调用，从而让场景结构更清晰。

### Lines 60-60: Define class MockCausalLM
```python
class MockCausalLM(nn.Module):
```
**EN:** This declaration introduces the `MockCausalLM` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `MockCausalLM` 测试类，并说明它通过继承承担的职责。

### Lines 61-63: Define helper: init
```python
    def __init__(self) -> None:
        super().__init__()
        self.model = SimpleModel()
```
**EN:** This helper function encapsulates reusable logic inside `MockCausalLM` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `MockCausalLM` 内部调用，从而让场景结构更清晰。

### Lines 65-67: Define helper: forward
```python
    @torch.no_grad()
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        return self.model(hidden_states)
```
**EN:** This helper function encapsulates reusable logic inside `MockCausalLM` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `MockCausalLM` 内部调用，从而让场景结构更清晰。

### Lines 70-76: Define helper: init weights
```python
def init_weights(module):
    if isinstance(module, LinearBase):
        torch.nn.init.uniform_(module.weight)
        if module.bias is not None:
            torch.nn.init.zeros_(module.bias)
    elif isinstance(module, RMSNorm):
        torch.nn.init.ones_(module.weight)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 79-104: Run test: model forward dump
```python
def test_model_forward_dump(tmp_path):
    set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))
    init_distributed_environment(
        backend="nccl",
        world_size=1,
        rank=0,
        local_rank=0,
        distributed_init_method="tcp://127.0.0.1:2646",
    )
    initialize_model_parallel()
    model = MockCausalLM()
    model.apply(init_weights)
    model = model.cuda().bfloat16()
    dumper = register_forward_hook_for_model(
        model, tmp_path / "sglang_dump", [0], 0, 0, 0
    )

    dir_path = dumper.get_dump_dir()
    inp = torch.randn(4, TEST_HIDDEN_SIZE, dtype=torch.bfloat16) * 0.01
    result = model(inp.cuda())
    data = torch.load(f"{dir_path}/Pass00000.pt")
    assert "model.layernorm" in data
    assert "model.mlp.down_proj" in data
    assert torch.allclose(
        data["model.mlp.down_proj"], result.cpu(), rtol=1e-5, atol=1e-5
    )
```
**EN:** This test method exercises model forward dump and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 model forward dump 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 107-108: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.tensor_dump_forward_hook`, `sglang.srt.distributed.parallel_state`, `sglang.srt.layers.layernorm`, `sglang.srt.layers.linear`, `sglang.srt.models.qwen2`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `unittest.main`

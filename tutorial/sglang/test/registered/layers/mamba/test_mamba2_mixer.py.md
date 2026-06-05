# test_mamba2_mixer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/layers/mamba/test_mamba2_mixer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mamba2 mixer behavior in SGLang's layers / mamba area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 层 / mamba 领域中与 mamba2 mixer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting source context / 辅助源码上下文
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/2c58742dff8613a3bd7496f2008ce927e18d38d1/tests/kernels/mamba/test_mamba_mixer2.py


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 6-19: module imports and dependencies / 模块导入与依赖
```python
from unittest.mock import patch

import pytest
import torch

from sglang.srt.distributed.device_communicators.custom_all_reduce_utils import (
    update_environment_variables,
)
from sglang.srt.distributed.parallel_state import (
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.utils import get_device, get_device_count
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest.mock`, `pytest`, `torch`, `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest.mock`, `pytest`, `torch`, `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`。

### Lines 21-23: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=32, stage="base-b", runner_config="2-gpu-large")

NUM_GPUS = 2
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 26-70: test case mixer2 gated norm multi gpu / 测试用例 mixer2 gated norm multi gpu
```python
@pytest.mark.parametrize("batch_size", [8])
@pytest.mark.parametrize("seq_len", [128])
@pytest.mark.parametrize(
    "hidden_size_n_groups",
    [
        (64, 1),  # hidden_size be divisible by num_gpus
        (100, 4),  # and n_groups must divide hidden_size
    ],
)
@pytest.mark.parametrize("dtype", [torch.float16])
def test_mixer2_gated_norm_multi_gpu(
    batch_size: int,
    seq_len: int,
    hidden_size_n_groups: tuple[int, int],
    dtype: torch.dtype,
    device: str = get_device(),
):
    if device not in ["cuda", "xpu"]:
        pytest.skip("Test only supports CUDA and XPU devices")

    assert (
        get_device_count() >= NUM_GPUS
    ), f"This test requires at least {NUM_GPUS} GPUs, but only {get_device_count()} available"

    hidden_size, n_groups = hidden_size_n_groups
    num_processes = NUM_GPUS

    def run_torch_spawn(fn, nprocs):
        # need to use torch.mp.spawn otherwise will have problems with
        # torch.distributed and cuda
        torch.multiprocessing.spawn(
            fn,
            args=(
                num_processes,
                batch_size,
                seq_len,
                hidden_size,
                n_groups,
                dtype,
                device,
            ),
            nprocs=nprocs,
        )

    run_torch_spawn(mixer2_gated_norm_tensor_parallel, NUM_GPUS)
```
**EN:** This test exercises `test_mixer2_gated_norm_multi_gpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixer2_gated_norm_multi_gpu`。

### Lines 73-147: function mixer2 gated norm tensor parallel / 函数 mixer2 gated norm tensor parallel
```python
def mixer2_gated_norm_tensor_parallel(
    local_rank: int,
    world_size: int,
    batch_size: int,
    seq_len: int,
    hidden_size: int,
    n_groups: int,
    dtype: torch.dtype,
    device: str,
):
    torch.manual_seed(0)

    device = torch.device(get_device(local_rank))
    torch.get_device_module(device).set_device(device)
    torch.set_default_device(device)
    torch.set_default_dtype(dtype)

    update_environment_variables(
        {
            "RANK": str(local_rank),
            "LOCAL_RANK": str(local_rank),
            "WORLD_SIZE": str(world_size),
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": "12345",
        }
    )

    # initialize distributed
    init_distributed_environment(
        world_size=world_size, rank=local_rank, local_rank=local_rank
    )
    initialize_model_parallel(tensor_model_parallel_size=world_size)

    # create random weights an inputs
    weight = torch.rand((hidden_size,), dtype=dtype, device=device)
    hidden_states = torch.randn(batch_size, seq_len, hidden_size)
    gate_states = torch.randn(batch_size, seq_len, hidden_size)

    import sglang.srt.layers.attention.mamba.mixer2_rms_norm_gated as m2
    import sglang.srt.model_loader.weight_utils as wu

    # Convenience: Avoid calling initialize_dp_attention
    with patch.object(wu, "get_attention_tp_rank", return_value=local_rank):
        # create gated-norm with TP
        mixer = m2.Mixer2RMSNormGated(
            full_hidden_size=hidden_size,
            full_n_groups=n_groups,
        )
        mixer.weight.weight_loader(mixer.weight, weight)

    with (
        patch.object(m2, "get_tensor_model_parallel_world_size", return_value=1),
        patch.object(m2, "get_tensor_model_parallel_rank", return_value=0),
    ):
        # create gated-norm without TP to compute reference
        mixer_single_gpu = m2.Mixer2RMSNormGated(
            full_hidden_size=hidden_size,
            full_n_groups=n_groups,
        )
        # assign weight to single-gpu mixer
        mixer_single_gpu.weight.data = weight

    # generate and compare
    N = hidden_size // world_size
    output = mixer(
        hidden_states[..., local_rank * N : (local_rank + 1) * N],
        gate_states[..., local_rank * N : (local_rank + 1) * N],
    )
    ref_output = mixer_single_gpu(hidden_states, gate_states)
    torch.testing.assert_close(
        output,
        ref_output[..., local_rank * N : (local_rank + 1) * N],
        atol=5e-3,
        rtol=1e-3,
    )
```
**EN:** This block implements `mixer2_gated_norm_tensor_parallel` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `mixer2_gated_norm_tensor_parallel`，承担模块行为中的一个聚焦逻辑片段。

### Lines 150-153: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `test_mixer2_gated_norm_multi_gpu`: This test exercises `test_mixer2_gated_norm_multi_gpu` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixer2_gated_norm_multi_gpu`。
- `mixer2_gated_norm_tensor_parallel`: This block implements `mixer2_gated_norm_tensor_parallel` and captures one focused piece of the module's behavior. / 该代码块实现 `mixer2_gated_norm_tensor_parallel`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest.mock`
- **Third-party modules / 第三方模块**: `pytest`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`, `sglang.srt.distributed.parallel_state`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 153

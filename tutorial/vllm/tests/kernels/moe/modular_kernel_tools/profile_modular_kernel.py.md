# profile_modular_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/modular_kernel_tools/profile_modular_kernel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / moe / modular_kernel_tools / profile_modular_kernel; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / moe / modular_kernel_tools / profile_modular_kernel 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-15)
```python
import copy
from collections.abc import Callable
from itertools import product
from typing import Any

import torch

from vllm.config import VllmConfig
from vllm.utils.torch_utils import set_random_seed

from .common import Config, RankTensors, WeightTensors, make_modular_kernel
from .parallel_utils import ProcessGroupInfo, parallel_launch_with_config
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, collections.abc, itertools, typing; shared test helpers from .common, .parallel_utils; and vLLM components like vllm.config, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、collections.abc、itertools、typing；共享测试辅助模块，例如 .common、.parallel_utils；vLLM 内部组件，例如 vllm.config、vllm.utils.torch_utils。

### Function `do_profile` (lines 18-43)
```python
def do_profile(
    fn: Callable,
    fn_kwargs: dict[Any, Any],
    pgi: ProcessGroupInfo,
    config: Config,
    num_warmups: int = 5,
):
    for _ in range(num_warmups):
        fn(**fn_kwargs)

    with torch.profiler.profile(
        activities=[
            torch.profiler.ProfilerActivity.CPU,
            torch.profiler.ProfilerActivity.CUDA,
        ],
        with_stack=True,
        record_shapes=True,
    ) as tprof:
        fn(**fn_kwargs)
        device = torch.accelerator.current_device_index()
        torch.accelerator.synchronize(device=device)

    # TODO (varun): Add a descriptive trace file name
    tprof.export_chrome_trace(
        f"{config.torch_trace_dir_path}/m{config.M}_{pgi.rank}_trace.json"
    )
```
**EN:** This helper function implements the shared logic for do profile. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 do profile 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `profile_modular_kernel` (lines 46-76)
```python
def profile_modular_kernel(
    pgi: ProcessGroupInfo,
    vllm_config: VllmConfig,
    config: Config,
    weights: WeightTensors,
    rank_tensors: RankTensors,
) -> None:
    assert isinstance(config.Ms, int)
    assert isinstance(config.topks, int)

    # weights for rank
    rank_weights = weights.slice_weights(pgi.rank, config.num_local_experts)

    # make modular kernel
    mk = make_modular_kernel(config, vllm_config, weights)

    mk_kwargs = {
        "hidden_states": rank_tensors.hidden_states,
        "w1": rank_weights.w1,
        "w2": rank_weights.w2,
        "topk_weights": rank_tensors.topk_weights,
        "topk_ids": rank_tensors.topk_ids,
        "expert_map": rank_tensors.expert_map,
        "w1_scale": rank_weights.w1_scale,
        "w2_scale": rank_weights.w2_scale,
        "a1_scale": rank_tensors.hidden_states_scale,
        "global_num_experts": config.E,
        "apply_router_weight_on_input": config.topk == 1,
    }

    do_profile(mk.apply, mk_kwargs, pgi, config)
```
**EN:** This helper function implements the shared logic for profile modular kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 profile modular kernel 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `rank_worker` (lines 79-105)
```python
def rank_worker(
    pgi: ProcessGroupInfo,
    vllm_config: VllmConfig,
    cpu_group,
    config: Config,
    weights: WeightTensors,
):
    set_random_seed(pgi.rank)

    # get weights to this device
    weights.to_current_device()

    Ms = config.Ms
    assert isinstance(Ms, list)
    TOPKs = config.topks
    assert isinstance(TOPKs, list)

    for m, topk in product(Ms, TOPKs):
        print(f"Running m={m}, topk={topk} ...")
        # override m and topk
        cfgx = copy.deepcopy(config)
        cfgx.Ms = m
        cfgx.topks = topk

        # inputs for rank
        rank_tensors = RankTensors.make(cfgx, pgi)
        profile_modular_kernel(pgi, vllm_config, cfgx, weights, rank_tensors)
```
**EN:** This helper function implements the shared logic for rank worker. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该辅助函数实现了 rank worker 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `run` (lines 108-113)
```python
def run(config: Config):
    weights: WeightTensors = WeightTensors.make(config)
    vllm_config, env_dict = config.make_env_data()
    parallel_launch_with_config(
        config.world_size, rank_worker, vllm_config, env_dict, config, weights
    )
```
**EN:** This helper function implements the shared logic for run. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Top-level block starting at line 116 (lines 116-132)
```python
if __name__ == "__main__":
    from .cli_args import make_config, make_config_arg_parser

    parser = make_config_arg_parser(
        description=(
            "Run single prepare-finalize & fused-experts combination test"
            "Example : python3 -m tests.kernels.moe.modular_kernel_tools.profile_modular_kernel "  # noqa: E501
            "--pf-type DeepEPLLPrepareAndFinalize --experts-type BatchedTritonExperts"
        )
    )
    args = parser.parse_args()
    assert args.torch_trace_dir_path is not None, (
        "Please pass in a directory to store torch traces"
    )
    config = make_config(args)

    run(config)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `copy`
- `collections.abc -> Callable`
- `itertools -> product`
- `typing -> Any`
- `torch`
- `vllm.config -> VllmConfig`
- `vllm.utils.torch_utils -> set_random_seed`
- `.common -> Config, RankTensors, WeightTensors, make_modular_kernel`
- `.parallel_utils -> ProcessGroupInfo, parallel_launch_with_config`

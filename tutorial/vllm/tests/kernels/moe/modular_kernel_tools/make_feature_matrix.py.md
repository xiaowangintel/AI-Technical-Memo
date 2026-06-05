# make_feature_matrix.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/modular_kernel_tools/make_feature_matrix.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / moe / modular_kernel_tools / make_feature_matrix; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / moe / modular_kernel_tools / make_feature_matrix 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-27)
```python
import copy
from enum import Enum
from itertools import product

import torch
from tqdm import tqdm

from vllm.config import VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.fused_moe.config import FUSED_MOE_UNQUANTIZED_CONFIG
from vllm.utils.torch_utils import set_random_seed

from .common import (
    Config,
    RankTensors,
    WeightTensors,
    reference_moe_impl,
    run_modular_kernel,
)
from .mk_objects import (
    MK_FUSED_EXPERT_TYPES,
    MK_MULTI_GPU_PREPARE_FINALIZE_TYPES,
    MK_QUANT_CONFIGS,
)
from .parallel_utils import ProcessGroupInfo, parallel_launch_with_config
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, enum, itertools, torch; shared test helpers from .common, .mk_objects, .parallel_utils; and vLLM components like vllm.config, vllm.model_executor.layers.fused_moe.config, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、enum、itertools、torch；共享测试辅助模块，例如 .common、.mk_objects、.parallel_utils；vLLM 内部组件，例如 vllm.config、vllm.model_executor.layers.fused_moe.config、vllm.utils.torch_utils。

### Class `Result` (lines 30-33)
```python
class Result(Enum):
    PASS = 1
    FAIL = 2
    SKIP = 3
```
**EN:** This class defines a container for Result.
**CN:** 该类定义了 Result 对应的容器。

### Function `rank_worker` (lines 36-69)
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

        # modular kernel out
        mk_out = run_modular_kernel(pgi, vllm_config, cfgx, weights, rank_tensors)

        with set_current_vllm_config(vllm_config):
            ref_out = reference_moe_impl(cfgx, weights, rank_tensors)

        torch.testing.assert_close(ref_out, mk_out, atol=3e-2, rtol=3e-2)
```
**EN:** This helper function implements the shared logic for rank worker. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该辅助函数实现了 rank worker 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `make_feature_matrix` (lines 72-157)
```python
def make_feature_matrix(csv_file_path: str):
    from dataclasses import asdict

    import pandas as pd

    def add_to_results(
        config: Config, success: Result, results_df: pd.DataFrame | None = None
    ):
        config_dict = asdict(config)
        config_dict["prepare_finalize_type"] = config_dict[
            "prepare_finalize_type"
        ].__name__
        config_dict["fused_experts_type"] = config_dict["fused_experts_type"].__name__
        config_dict["per_tensor_act_quant"] = config.is_per_tensor_act_quant
        quant_config_dict = config_dict["quant_config"]
        del config_dict["quant_config"]
        if quant_config_dict is None:
            quant_config = FUSED_MOE_UNQUANTIZED_CONFIG
            quant_config_dict = asdict(quant_config)

        config_dict |= quant_config_dict
        result_dict = config_dict | {"success": success.name}

        result_df = pd.DataFrame([result_dict])
        if results_df is None:
            results_df = result_df
        else:
            results_df = pd.concat([results_df, result_df], ignore_index=True)

        return results_df

    Ms = [64]
    Ks = [7168]  # hidden sizes
    Ns = [2048]
    TOPKs = [[4, 1]]
    Es = [32]
    DTYPEs = [torch.bfloat16]
    PF_TYPES = MK_MULTI_GPU_PREPARE_FINALIZE_TYPES
    FE_TYPES = MK_FUSED_EXPERT_TYPES
    Q_TYPES = MK_QUANT_CONFIGS
# ... excerpt ...
        combinations
    ):
        config = Config(
            Ms=[m],
            K=k,
            N=n,
            E=e,
            topks=topks,
            dtype=dtype,
            prepare_finalize_type=pf_type,
            fused_experts_type=experts_type,
            quant_config=quant_config,
            world_size=2,
        )

        success = None
        if config.is_valid()[0]:
            print(f"Running config : {config.describe()} ...")
            try:
                weights: WeightTensors = WeightTensors.make(config)
                vllm_config, env_dict = config.make_env_data()
                parallel_launch_with_config(
                    config.world_size,
                    rank_worker,
                    vllm_config,
                    env_dict,
                    config,
                    weights,
                )
                success = Result.PASS
            except Exception as _:
                success = Result.FAIL
        else:
            success = Result.SKIP

        results_df = add_to_results(config, success, results_df)

    if results_df is not None:
        results_df.to_csv(f"{csv_file_path}")
```
**EN:** This helper function implements the shared logic for feature matrix. It keeps the surrounding module logic factored into a reusable unit. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 feature matrix 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。 由于该代码块较大，这里只展示关键片段。

### Top-level block starting at line 160 (lines 160-189)
```python
if __name__ == "__main__":
    import argparse
    from pathlib import Path

    parser = argparse.ArgumentParser(
        description=(
            "Make ModularKernel feature matrix \n"
            "Example : python3 -m tests.kernels.moe.modular_kernel_tools.make_feature_matrix "  # noqa: E501
            "-f ./feature_matrices/feature_matrix.csv"
        )
    )

    parser.add_argument(
        "-f",
        "--feature-matrix-csv-file-path",
        type=str,
        required=True,
        help="File name to Generate a .csv file",
    )
    args = parser.parse_args()

    csv_path = args.feature_matrix_csv_file_path
    assert csv_path.endswith("csv"), (
        f"Need a file path ending with .csv, got {csv_path}"
    )
    assert Path(csv_path).parent.is_dir(), (
        f"Cannot find parent directory for {Path(csv_path).parent}"
    )

    make_feature_matrix(args.feature_matrix_csv_file_path)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `copy`
- `enum -> Enum`
- `itertools -> product`
- `torch`
- `tqdm -> tqdm`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.fused_moe.config -> FUSED_MOE_UNQUANTIZED_CONFIG`
- `vllm.utils.torch_utils -> set_random_seed`
- `.common -> Config, RankTensors, WeightTensors, reference_moe_impl, run_modular_kernel`
- `.mk_objects -> MK_FUSED_EXPERT_TYPES, MK_MULTI_GPU_PREPARE_FINALIZE_TYPES, MK_QUANT_CONFIGS`
- `.parallel_utils -> ProcessGroupInfo, parallel_launch_with_config`

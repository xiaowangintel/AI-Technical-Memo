# sdp.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/transformer/sdp.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```python
import argparse
import itertools
import random
import warnings
from dataclasses import dataclass
from pathlib import Path
from pprint import pprint

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 9-16 / 第 9-16 行

```python
import numpy as np
from prettytable import PrettyTable
from tqdm import tqdm

import torch
import torch.utils.benchmark as benchmark
from torch.backends.cuda import sdp_kernel

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 17-32 / 第 17-32 行

```python

warnings.filterwarnings("ignore")


@dataclass(frozen=True)
class ExperimentConfig:
    batch_size: int
    num_heads: int
    max_sequence_len: int
    embed_dimension: int
    dtype: torch.dtype
    pad_percentage: float | None
    enable_math: bool
    enable_flash: bool
    enable_mem_efficient: bool
    enable_cudnn: bool
```

- **EN:** Important local symbols in this block include ExperimentConfig.
- **CN:** 该代码块中的重要局部符号包括 ExperimentConfig。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 33-47 / 第 33-47 行

```python

    def get_entries(self) -> list:
        return [
            self.batch_size,
            self.num_heads,
            self.max_sequence_len,
            self.embed_dimension,
            self.dtype,
            self.pad_percentage,
            self.enable_math,
            self.enable_flash,
            self.enable_mem_efficient,
            self.enable_cudnn,
        ]

```

- **EN:** Important local symbols in this block include get_entries.
- **CN:** 该代码块中的重要局部符号包括 get_entries。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 48-62 / 第 48-62 行

```python
    @classmethod
    def get_entry_names(cls) -> list[str]:
        return [
            "batch_size",
            "num_heads",
            "max_sequence_len",
            "embed_dimension",
            "dtype",
            "pad_percentage",
            "enable_math",
            "enable_flash",
            "enable_mem_efficient",
            "enable_cudnn",
        ]

```

- **EN:** Important local symbols in this block include get_entry_names.
- **CN:** 该代码块中的重要局部符号包括 get_entry_names。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 63-70 / 第 63-70 行

```python

@dataclass(frozen=True)
class ExperimentResults:
    nn_mha_time: float
    compiled_nn_mha_time: float | None
    composite_mha_time: float
    compiled_composite_mha_time: float | None

```

- **EN:** Important local symbols in this block include ExperimentResults.
- **CN:** 该代码块中的重要局部符号包括 ExperimentResults。

### Lines 71-80 / 第 71-80 行

```python
    def get_entries(self) -> list:
        return [
            f"{self.nn_mha_time:2f}",
            f"{self.compiled_nn_mha_time:2f}" if self.compiled_nn_mha_time else None,
            f"{self.composite_mha_time:2f}",
            f"{self.compiled_composite_mha_time:2f}"
            if self.compiled_composite_mha_time
            else None,
        ]

```

- **EN:** Important local symbols in this block include get_entries.
- **CN:** 该代码块中的重要局部符号包括 get_entries。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 81-89 / 第 81-89 行

```python
    @classmethod
    def get_entry_names(cls) -> list[str]:
        return [
            "nn_mha_time (\u00b5s)",
            "compiled_nn_mha_time (\u00b5s)",
            "composite_mha_time (\u00b5s)",
            "compiled_composite_mha_time (\u00b5s)",
        ]

```

- **EN:** Important local symbols in this block include get_entry_names.
- **CN:** 该代码块中的重要局部符号包括 get_entry_names。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 90-97 / 第 90-97 行

```python

@dataclass(frozen=True)
class Experiment:
    config: ExperimentConfig
    results: ExperimentResults

    def get_entries(self) -> list:
        return self.config.get_entries() + self.results.get_entries()
```

- **EN:** Important local symbols in this block include Experiment, get_entries.
- **CN:** 该代码块中的重要局部符号包括 Experiment、get_entries。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 98-107 / 第 98-107 行

```python


class CompositeMHA(torch.nn.Module):
    def __init__(self, num_heads, in_proj_weight, in_proj_bias, out_proj):
        super().__init__()
        self.in_proj_weight = in_proj_weight
        self.in_proj_bias = in_proj_bias
        self.out_proj = out_proj
        self.num_heads = num_heads

```

- **EN:** Important local symbols in this block include CompositeMHA, __init__.
- **CN:** 该代码块中的重要局部符号包括 CompositeMHA、__init__。

### Lines 108-115 / 第 108-115 行

```python
    def forward(self, query, key, value, mask):
        if not (query is key and key is value):
            raise NotImplementedError(
                "query, key and value must be the same Tensor for now."
            )
        if mask is not None:
            raise NotImplementedError("mask is currently not supported.")

```

- **EN:** Important local symbols in this block include forward.
- **CN:** 该代码块中的重要局部符号包括 forward。

### Lines 116-123 / 第 116-123 行

```python
        query_projected = torch.nn.functional.linear(
            query, self.in_proj_weight, self.in_proj_bias
        )

        batch_size = query_projected.size(0)
        embed_dim = query_projected.size(2)
        head_dim = embed_dim // (self.num_heads * 3)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 124-139 / 第 124-139 行

```python
        query, key, value = query_projected.chunk(3, -1)

        query = query.view(batch_size, -1, self.num_heads, head_dim).transpose(1, 2)
        key = key.view(batch_size, -1, self.num_heads, head_dim).transpose(1, 2)
        value = value.view(batch_size, -1, self.num_heads, head_dim).transpose(1, 2)

        # the output of sdp = (batch, num_heads, seq_len, head_dim)
        attn = torch.nn.functional.scaled_dot_product_attention(
            query,
            key,
            value,
            attn_mask=None,
            dropout_p=0.0,
            is_causal=False,
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 140-149 / 第 140-149 行

```python
        attn = attn.transpose(1, 2).reshape(batch_size, -1, self.num_heads * head_dim)
        # Match return signature of nn.MHA
        return self.out_proj(attn), None


def build_composite_mha_from_nn_mha(pt):
    if not pt._qkv_same_embed_dim:
        raise AssertionError("pt._qkv_same_embed_dim must be True")
    in_proj_weight = pt.in_proj_weight
    if in_proj_weight is None:
```

- **EN:** Important local symbols in this block include build_composite_mha_from_nn_mha.
- **CN:** 该代码块中的重要局部符号包括 build_composite_mha_from_nn_mha。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 150-164 / 第 150-164 行

```python
        raise AssertionError("pt.in_proj_weight must not be None")
    if not pt.batch_first:
        raise AssertionError("pt.batch_first must be True")
    return CompositeMHA(pt.num_heads, pt.in_proj_weight, pt.in_proj_bias, pt.out_proj)


def generate_rand_batch(
    batch_size,
    max_sequence_len,
    embed_dimension,
    pad_percentage=None,
    dtype=torch.float16,
    device="cuda",
):
    if not pad_percentage:
```

- **EN:** Important local symbols in this block include generate_rand_batch.
- **CN:** 该代码块中的重要局部符号包括 generate_rand_batch。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 165-178 / 第 165-178 行

```python
        return (
            torch.randn(
                batch_size,
                max_sequence_len,
                embed_dimension,
                dtype=dtype,
                device=device,
            ),
            None,
        )
    # Really slow but should work
    seq_len_list = [
        int(max_sequence_len * (1 - random.gauss(pad_percentage, 0.01)))
        for _ in range(batch_size)
```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 179-187 / 第 179-187 行

```python
    ]
    # Make random ele max length
    seq_len_list[random.randint(0, batch_size - 1)] = max_sequence_len
    # print(f"Theoretical padding: {pad_percentage} actual: {1 - (sum(seq_len_list) / (batch_size * max_sequence_len))}")
    return (
        torch.nested.nested_tensor(
            [
                torch.randn(seq_len, embed_dimension, dtype=dtype, device=device)
                for seq_len in seq_len_list
```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 188-198 / 第 188-198 行

```python
            ]
        ),
        seq_len_list,
    )


def benchmark_torch_function_in_microseconds(f, *args, **kwargs):
    t0 = benchmark.Timer(
        stmt="f(*args, **kwargs)", globals={"args": args, "kwargs": kwargs, "f": f}
    )
    return t0.blocked_autorange().mean * 1e6
```

- **EN:** Important local symbols in this block include benchmark_torch_function_in_microseconds.
- **CN:** 该代码块中的重要局部符号包括 benchmark_torch_function_in_microseconds。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 199-210 / 第 199-210 行

```python


def assert_close_tensors(tensor_a, tensor_b):
    # First order sanity check. Not a replacement for rigorous tests.
    if tensor_a.is_nested and tensor_b.is_nested:
        for a, b in zip(tensor_a.unbind(), tensor_b.unbind()):
            if not torch.allclose(a, b, atol=1e-2, rtol=1e-2):
                raise AssertionError(
                    f"Nested tensors not close: max diff = {(a - b).abs().max()}"
                )
    else:
        if not torch.allclose(tensor_a, tensor_b, atol=1e-3, rtol=1e-3):
```

- **EN:** Important local symbols in this block include assert_close_tensors.
- **CN:** 该代码块中的重要局部符号包括 assert_close_tensors。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 211-225 / 第 211-225 行

```python
            raise AssertionError(
                f"Tensors not close: max diff = {(tensor_a - tensor_b).abs().max()}"
            )


def run_single_experiment(config: ExperimentConfig) -> ExperimentResults:
    with sdp_kernel(
        enable_math=config.enable_math,
        enable_flash=config.enable_flash,
        enable_mem_efficient=config.enable_mem_efficient,
        enable_cudnn=config.enable_cudnn,
    ):
        dropout_p = 0.0
        mask = None

```

- **EN:** Important local symbols in this block include run_single_experiment.
- **CN:** 该代码块中的重要局部符号包括 run_single_experiment。

### Lines 226-241 / 第 226-241 行

```python
        nn_mha = torch.nn.MultiheadAttention(
            embed_dim=config.embed_dimension,
            num_heads=config.num_heads,
            batch_first=True,
            dropout=dropout_p,
        )
        nn_mha = nn_mha.eval().to("cuda", config.dtype)
        composite_mha = build_composite_mha_from_nn_mha(nn_mha)
        qkv, lengths = generate_rand_batch(
            config.batch_size,
            config.max_sequence_len,
            config.embed_dimension,
            config.pad_percentage,
            config.dtype,
        )
        nn_mha_output, _ = nn_mha(qkv, qkv, qkv, mask)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 242-253 / 第 242-253 行

```python
        composite_mha_output, _ = composite_mha(qkv, qkv, qkv, mask)

        # First order sanity check
        assert_close_tensors(nn_mha_output, composite_mha_output)

        nn_mha_time = benchmark_torch_function_in_microseconds(
            nn_mha, qkv, qkv, qkv, mask
        )
        composite_mha_time = benchmark_torch_function_in_microseconds(
            composite_mha, qkv, qkv, qkv, mask
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 254-262 / 第 254-262 行

```python
        # TorchDynamo will error on NestedTensors
        if config.pad_percentage is None:
            compiled_nn_mha = torch.compile(nn_mha)
            compiled_composite_mha = torch.compile(composite_mha)

            compiled_nn_mha_time = benchmark_torch_function_in_microseconds(
                compiled_nn_mha, qkv, qkv, qkv, mask
            )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 263-273 / 第 263-273 行

```python
            compiled_composite_mha_time = benchmark_torch_function_in_microseconds(
                compiled_composite_mha,
                qkv,
                qkv,
                qkv,
                mask,
            )
        else:
            compiled_nn_mha_time = None
            compiled_composite_mha_time = None

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 274-281 / 第 274-281 行

```python
        results = ExperimentResults(
            nn_mha_time,
            compiled_nn_mha_time,
            composite_mha_time,
            compiled_composite_mha_time,
        )
        return Experiment(config, results)

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 282-297 / 第 282-297 行

```python

# Could return generator
def generate_experiments(
    batch_sizes, num_heads, max_seq_lens, embed_dims, dtypes, pad_percentages
) -> list[ExperimentConfig]:
    configs = []
    for bsz, n_heads, seq_len, embed_dim, dtype, padding in itertools.product(
        batch_sizes, num_heads, max_seq_lens, embed_dims, dtypes, pad_percentages
    ):
        configs.append(
            ExperimentConfig(
                batch_size=bsz,
                num_heads=n_heads,
                max_sequence_len=seq_len,
                embed_dimension=embed_dim,
                dtype=dtype,
```

- **EN:** Important local symbols in this block include generate_experiments.
- **CN:** 该代码块中的重要局部符号包括 generate_experiments。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 298-305 / 第 298-305 行

```python
                pad_percentage=padding,
                enable_math=False,
                enable_flash=True,
                enable_mem_efficient=True,
                enable_cudnn=True,
            )
        )
    return configs
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 306-321 / 第 306-321 行

```python


def main(save_path: Path | None):
    seed = 123
    np.random.seed(seed)
    torch.manual_seed(seed)

    # Run one timing experiment comparing nn_mha vs composite_mha
    config = ExperimentConfig(
        batch_size=128,
        num_heads=8,
        max_sequence_len=512,
        embed_dimension=512,
        dtype=torch.float16,
        pad_percentage=None,
        enable_math=False,
```

- **EN:** Important local symbols in this block include main.
- **CN:** 该代码块中的重要局部符号包括 main。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 322-329 / 第 322-329 行

```python
        enable_flash=True,
        enable_mem_efficient=True,
        enable_cudnn=True,
    )

    experiment = run_single_experiment(config)
    pprint(experiment)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 330-343 / 第 330-343 行

```python
    table = PrettyTable()
    table.float_format = ".3"
    table.field_names = (
        ExperimentConfig.get_entry_names() + ExperimentResults.get_entry_names()
    )

    # Run a bunch of experiments
    batch_sizes = [256]
    num_heads = [32]
    max_seq_lens = [256]
    embed_dims = [512]
    dtypes = [torch.bfloat16, torch.float16, torch.float32]
    pad_percentages = [None, 0.9]

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 344-353 / 第 344-353 行

```python
    experiment_configs = generate_experiments(
        batch_sizes, num_heads, max_seq_lens, embed_dims, dtypes, pad_percentages
    )

    experiments: list[Experiment] = []
    for experiment_config in tqdm(experiment_configs):
        experiment = run_single_experiment(experiment_config)
        experiments.append(experiment)
        table.add_row(experiment.get_entries())

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 354-361 / 第 354-361 行

```python
    print(table)

    csv_string = table.get_csv_string()
    if save_path is not None:
        with open(save_path, "w") as csvfile:
            csvfile.write(csv_string)


```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 362-370 / 第 362-370 行

```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save-path", "--save_path", type=str, help="Path to save the results"
    )

    args = parser.parse_args()
    save_path = Path(args.save_path) if args.save_path else None
    main(save_path)
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Representative symbols: ExperimentConfig, ExperimentResults, Experiment, CompositeMHA, get_entries, get_entry_names, __init__, forward** — 代表性符号：ExperimentConfig、ExperimentResults、Experiment、CompositeMHA、get_entries、get_entry_names、__init__、forward

## Dependencies / 依赖关系

- `argparse`
- `itertools`
- `random`
- `warnings`
- `dataclasses`
- `pathlib`
- `pprint`
- `numpy`
- `prettytable`
- `tqdm`
- `torch`
- `torch.utils.benchmark`
- `torch.backends.cuda`

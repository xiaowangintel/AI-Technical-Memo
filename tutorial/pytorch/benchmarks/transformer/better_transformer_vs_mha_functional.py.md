# better_transformer_vs_mha_functional.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/transformer/better_transformer_vs_mha_functional.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行

```python
"""
Tests the performance of torch.nn.MultiheadAttention's fast path (BetterTransformer)
vs the slow path (torch.nn.functional.multi_head_attention)

To run this script install these dependencies:

pip install tqdm
pip install prettytable
"""

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 11-19 / 第 11-19 行

```python
import argparse
import itertools
import json
import random
import warnings
from collections import defaultdict, OrderedDict
from pathlib import Path
from pprint import pprint

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 20-28 / 第 20-28 行

```python
import numpy as np
from prettytable import PrettyTable
from tqdm import tqdm

import torch


warnings.filterwarnings("ignore")

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 29-39 / 第 29-39 行

```python
error_dict = defaultdict(int)


def benchmark_torch_function(iters, f, *args, **kwargs):
    f(*args, **kwargs)
    f(*args, **kwargs)
    torch.cuda.synchronize()
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    start_event.record()
    for _ in range(iters):
```

- **EN:** Important local symbols in this block include benchmark_torch_function.
- **CN:** 该代码块中的重要局部符号包括 benchmark_torch_function。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 40-47 / 第 40-47 行

```python
        f(*args, **kwargs)
    end_event.record()
    torch.cuda.synchronize()
    # elapsed_time has a resolution of 0.5 microseconds:
    # but returns milliseconds, so we need to multiply it to increase resolution
    return start_event.elapsed_time(end_event) * 1000 / iters, *f(*args, **kwargs)


```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 48-63 / 第 48-63 行

```python
def run(
    a: int,
    b: int,
    iters: int,
    batch_size: int,
    sequence_length: int,
    embed_dim: int,
    num_heads: int,
    device: str,
    dtype: str,
    block_size: int,
    seed,
):
    random.seed(seed)
    torch.manual_seed(seed)
    np.random.seed(seed)
```

- **EN:** Important local symbols in this block include run.
- **CN:** 该代码块中的重要局部符号包括 run。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 64-75 / 第 64-75 行

```python

    from scipy.stats import beta

    lengths = (
        beta.rvs(a, b, size=batch_size)
        * (sequence_length + block_size - 1)
        // block_size
    )
    lengths = list(map(int, list(lengths)))
    lengths = [l * block_size for l in lengths]
    lengths = [max(l, block_size) for l in lengths]

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 76-85 / 第 76-85 行

```python
    # Used to enforce no padding
    # lengths = [sequence_length] * batch_size

    # Ensure one row in the batch of ele has the max_sequence_length
    lengths[random.randint(0, batch_size - 1)] = sequence_length

    q = [torch.randn(l, embed_dim, device=device, dtype=dtype) for l in lengths]
    q = torch.nested.nested_tensor(q, device=device, dtype=dtype)
    k, v = q, q

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 86-96 / 第 86-96 行

```python
    qkv = torch.nn.Linear(embed_dim, 3 * embed_dim, device=device, dtype=dtype)
    proj = torch.nn.Linear(embed_dim, embed_dim, device=device, dtype=dtype)

    native_mha = torch.nn.MultiheadAttention(
        embed_dim, num_heads, batch_first=True, device=device, dtype=dtype
    ).eval()
    native_mha.in_proj_weight = qkv.weight
    native_mha.in_proj_bias = qkv.bias
    native_mha.out_proj.weight = proj.weight
    native_mha.out_proj.bias = proj.bias

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 97-105 / 第 97-105 行

```python
    # Create query mask
    q_mask = torch.nested.to_padded_tensor(
        torch.nested.nested_tensor(
            [torch.tensor([True] * length, dtype=torch.bool) for length in lengths]
        ),
        0,
    )
    q_mask = q_mask.cuda()

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 106-121 / 第 106-121 行

```python
    if q_mask.size(1) == 0:
        return None

    # Benchmark the native MHA in core
    with torch.backends.cuda.sdp_kernel(enable_math=False, enable_flash=True):
        with torch.inference_mode():
            time_native_mha_fast, y_native_mha_fast, _ = benchmark_torch_function(
                iters, native_mha, q, k, v, need_weights=False
            )
    q = q.to_padded_tensor(0)
    k = q
    v = q
    # Internal Flash Attention
    time_native_mha_slow, y_native_mha_slow, _ = benchmark_torch_function(
        iters, native_mha, q, k, v, key_padding_mask=~q_mask, need_weights=False
    )
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 122-131 / 第 122-131 行

```python

    # Convert to padded for comparison
    if y_native_mha_fast.is_nested:
        y_native_mha_fast = torch.nested.to_padded_tensor(y_native_mha_fast, 0)
    y_native_mha_fast = y_native_mha_fast * q_mask.unsqueeze(-1)

    if y_native_mha_slow.is_nested:
        y_native_mha_slow = torch.nested.to_padded_tensor(y_native_mha_slow, 0)
    y_native_mha_slow = y_native_mha_slow * q_mask.unsqueeze(-1)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 132-141 / 第 132-141 行

```python
    # Correctness check
    entry_name = f"batch:{batch_size}_seq_len:{sequence_length}_n_heads:{num_heads}_embed_dim:{embed_dim}"
    try:
        torch.testing.assert_close(
            y_native_mha_fast, y_native_mha_slow, atol=1e-3, rtol=1e-3
        )
    except AssertionError:
        error_dict[entry_name] += 1
        pprint(error_dict)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 142-157 / 第 142-157 行

```python
    # Calculate amount of padding
    padding = 1 - q_mask.float().mean().item()

    # Calculate the speedup for flash attention
    speedup_fast_internal = time_native_mha_slow / time_native_mha_fast

    result_entry = OrderedDict()
    result_entry["dtype"] = dtype
    result_entry["batch_size"] = batch_size
    result_entry["sequence_length"] = sequence_length
    result_entry["n_heads"] = num_heads
    result_entry["embed_dim"] = embed_dim
    result_entry["time_native_mha_slow(\u00b5s)"] = f"{time_native_mha_slow:.3f}"
    result_entry["time_native_mha_fast (\u00b5s)"] = f"{time_native_mha_fast:.3f}"
    result_entry["speedup flash_mha v native_mha"] = f"{speedup_fast_internal:.3f}"
    result_entry["padding"] = f"{padding:.3f}"
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 158-173 / 第 158-173 行

```python
    return result_entry


def main(save_path: Path | None, error_path: Path | None):
    table = PrettyTable()
    entries = defaultdict(list)

    print("CUDA device: ", torch.cuda.get_device_name(0))
    iters = 100
    header = None
    batch_sizes = [16, 32, 64, 128, 256]
    sequence_lengths = [64, 128, 256, 512]
    embed_dims = [512, 1024]
    num_heads_list = [8, 16]
    betas = range(1, 64, 4)

```

- **EN:** Important local symbols in this block include main.
- **CN:** 该代码块中的重要局部符号包括 main。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 174-189 / 第 174-189 行

```python
    for batch_size, sequence_length, embed_dim, num_heads, block_size, b in tqdm(
        list(
            itertools.product(
                batch_sizes, sequence_lengths, embed_dims, num_heads_list, [2], betas
            )
        )
    ):
        seed = 26214  # Magic number that works well for higher b values
        entry = run(
            1,
            b * 0.05,
            iters,
            batch_size,
            sequence_length,
            embed_dim,
            num_heads,
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 190-197 / 第 190-197 行

```python
            "cuda",
            torch.float16,
            block_size,
            seed,
        )
        if entry is None:
            continue
        if header is None:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 198-205 / 第 198-205 行

```python
            table.field_names = list(entry.keys())
            header = list(entry.keys())
        row = []
        for k, v in entry.items():
            row.append(v)
            entries[k].append(v)
        table.add_row(row)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 206-214 / 第 206-214 行

```python
    # Print the full table to console
    print(table)
    pprint(error_dict)

    csv_string = table.get_csv_string()
    if save_path is not None:
        with open(save_path, "w") as csvfile:
            csvfile.write(csv_string)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 215-230 / 第 215-230 行

```python
    print(f"Total errors: {sum(error_dict.values())}")
    if error_path is not None:
        with open(error_path, "w") as file:
            file.write(json.dumps(error_dict))


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save-path", "--save_path", type=str, help="Path to save the results"
    )
    parser.add_argument(
        "--error-save-path",
        "--error_save_path",
        type=str,
        help="Path to save the errors",
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 231-237 / 第 231-237 行

```python
    )

    args = parser.parse_args()
    save_path = Path(args.save_path) if args.save_path else None
    error_path = Path(args.error_save_path) if args.error_save_path else None

    main(save_path, error_path)
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: benchmark_torch_function, run, main** — 代表性符号：benchmark_torch_function、run、main

## Dependencies / 依赖关系

- `argparse`
- `itertools`
- `json`
- `random`
- `warnings`
- `collections`
- `pathlib`
- `pprint`
- `numpy`
- `prettytable`
- `tqdm`
- `torch`
- `scipy.stats`

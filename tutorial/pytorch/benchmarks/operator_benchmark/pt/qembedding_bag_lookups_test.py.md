# qembedding_bag_lookups_test.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/operator_benchmark/pt/qembedding_bag_lookups_test.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator benchmark cases, parameter grids, and registration calls used to measure PyTorch operator performance. Benchmark registration and workload-shape coverage are important to understanding the file. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 定义用于衡量 PyTorch 算子性能的 benchmark 用例、参数网格以及注册调用。 理解该文件时，benchmark 注册与工作负载形状覆盖是关键。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```python
import numpy as np

import operator_benchmark as op_bench

import torch
from torch.testing._internal.common_quantization import lengths_to_offsets


```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 9-24 / 第 9-24 行

```python
torch.ops.load_library("//caffe2/torch/fb/sparsenn:sparsenn_operators")


embedding_bag_rowwise_offsets_short_configs = op_bench.cross_product_configs(
    num_embeddings=(80,),
    embedding_dim=(128, 256),
    num_offsets=range(2, 10),
    enable_per_sample_weights=(True, False),
    include_last_offset=(True, False),
    is_pruned_weights=(
        True,
        False,
    ),
    use_32bit_indices=(True, False),
    use_32bit_offsets=(True, False),
    tags=["short"],
```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。

### Lines 25-40 / 第 25-40 行

```python
)


embedding_bag_rowwise_offsets_long_configs = op_bench.cross_product_configs(
    num_embeddings=(100, 120, 1000, 10_000, 20_000),
    embedding_dim=(16, 64, 128, 256),
    num_offsets=range(10, 20),
    enable_per_sample_weights=(True, False),
    include_last_offset=(True, False),
    is_pruned_weights=(
        True,
        False,
    ),
    use_32bit_indices=(True, False),
    use_32bit_offsets=(True, False),
    tags=["long"],
```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。

### Lines 41-48 / 第 41-48 行

```python
)


full_configs = (
    embedding_bag_rowwise_offsets_short_configs
    + embedding_bag_rowwise_offsets_long_configs
)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 49-58 / 第 49-58 行

```python
four_bit_rowwise_ops = op_bench.op_list(
    attrs=(
        (
            "qembeddingbag_4bit_rowwise_offsets",
            torch.ops.quantized.embedding_bag_4bit_rowwise_offsets,
        ),
    ),
    attr_names=("op_name", "op_func"),
)

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。

### Lines 59-68 / 第 59-68 行

```python
byte_rowwise_ops = op_bench.op_list(
    attrs=(
        (
            "qembeddingbag_byte_rowwise_offsets",
            torch.ops.quantized.embedding_bag_byte_rowwise_offsets,
        ),
    ),
    attr_names=("op_name", "op_func"),
)

```

- **EN:** The code is defining benchmark parameter spaces so one benchmark class can be exercised across many shapes, devices, or dtypes.
- **CN:** 这里在定义 benchmark 参数空间，使同一个 benchmark 类可以覆盖多种形状、设备或 dtype。

### Lines 69-76 / 第 69-76 行

```python

def get_pruned_weights_and_mapping(q_weights):
    indicator = torch.from_numpy(
        np.random.uniform(low=-1.0, high=1.0, size=[q_weights.shape[0]]).astype(
            np.float32
        )
    )

```

- **EN:** Important local symbols in this block include get_pruned_weights_and_mapping.
- **CN:** 该代码块中的重要局部符号包括 get_pruned_weights_and_mapping。

### Lines 77-84 / 第 77-84 行

```python
    (
        q_pruned_weights,
        compressed_indices_mapping,
    ) = torch.ops.fb.embedding_bag_rowwise_prune(
        q_weights, indicator, 0.01, torch.int32
    )

    return q_pruned_weights, compressed_indices_mapping
```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 85-100 / 第 85-100 行

```python


class EmbedddingBag4BitRowwiseOffsetsTest(op_bench.TorchBenchmarkBase):
    def init(
        self,
        num_embeddings: int,
        embedding_dim: int,
        num_offsets: int,
        enable_per_sample_weights: bool,
        include_last_offset: bool,
        is_pruned_weights: bool,
        use_32bit_indices: bool,
        use_32bit_offsets: bool,
        op_func,
    ):
        self.num_embeddings = num_embeddings
```

- **EN:** Important local symbols in this block include EmbedddingBag4BitRowwiseOffsetsTest, init.
- **CN:** 该代码块中的重要局部符号包括 EmbedddingBag4BitRowwiseOffsetsTest、init。

### Lines 101-114 / 第 101-114 行

```python
        self.embedding_dim = embedding_dim
        self.num_offsets = num_offsets
        self.enable_per_sample_weights = enable_per_sample_weights
        self.include_last_offset = include_last_offset
        self.max_segment_length = 20
        self.num_lengths = np.random.randint(1, num_offsets + 1)
        self.lengths = np.random.randint(
            0, self.max_segment_length + 1, size=self.num_lengths
        ).astype(np.int32)
        self.num_indices = np.sum(self.lengths)
        self.is_pruned_weights = is_pruned_weights
        self.use_32bit_indices = use_32bit_indices
        self.use_32bit_offsets = use_32bit_offsets

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 115-124 / 第 115-124 行

```python
        self.offsets = lengths_to_offsets(self.lengths)
        self.indices = torch.from_numpy(
            np.random.randint(
                low=0, high=num_embeddings, size=self.num_indices, dtype=np.int64
            )
        )

        self.indices = self.indices.int() if self.use_32bit_indices else self.indices
        self.offsets = self.offsets.int() if self.use_32bit_offsets else self.offsets

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 125-140 / 第 125-140 行

```python
        if self.include_last_offset:
            self.offsets = torch.cat(
                (self.offsets, torch.tensor([self.indices.size(0)], dtype=torch.long)),
                0,
            )

        self.weights = torch.from_numpy(
            (
                np.random.random_sample((self.num_embeddings, self.embedding_dim)) + 1
            ).astype(np.float32)
        )
        self.indices = torch.from_numpy(
            np.random.randint(
                low=0, high=self.num_embeddings, size=self.num_indices, dtype=np.int64
            )
        )
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 141-150 / 第 141-150 行

```python
        self.prepack_func = torch.ops.quantized.embedding_bag_4bit_prepack

        self.prepacked_weights = self.prepack_func(self.weights)
        self.per_sample_weights = (
            torch.from_numpy(
                np.random.uniform(low=0.01, high=0.5, size=[len(self.indices)]).astype(
                    np.float32
                )
            )
            if self.enable_per_sample_weights
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 151-161 / 第 151-161 行

```python
            else None
        )

        self.compressed_indices = None

        if self.is_pruned_weights:
            (
                self.prepacked_weights,
                self.compressed_indices,
            ) = get_pruned_weights_and_mapping(self.prepacked_weights)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 162-172 / 第 162-172 行

```python
        self.inputs = {
            "prepacked_weights": self.prepacked_weights,
            "indices": self.indices,
            "offsets": self.offsets,
            "mode": 0,
            "per_sample_weights": self.per_sample_weights,
            "include_last_offset": self.include_last_offset,
            "is_pruned_weights": self.is_pruned_weights,
            "compressed_indices": self.compressed_indices,
        }

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 173-186 / 第 173-186 行

```python
        self.op_func = op_func

    def forward(
        self,
        prepacked_weights,
        indices,
        offsets,
        mode: int,
        per_sample_weights: torch.Tensor | None,
        include_last_offset: bool,
        is_pruned_weights: bool,
        compressed_indices: torch.Tensor | None,
    ):
        return self.op_func(
```

- **EN:** Important local symbols in this block include forward.
- **CN:** 该代码块中的重要局部符号包括 forward。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 187-196 / 第 187-196 行

```python
            prepacked_weights,
            indices,
            offsets,
            mode=mode,
            per_sample_weights=per_sample_weights,
            include_last_offset=include_last_offset,
            pruned_weights=is_pruned_weights,
            compressed_indices_mapping=compressed_indices,
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 197-212 / 第 197-212 行

```python

class EmbedddingBagByteRowwiseOffsetsTest(op_bench.TorchBenchmarkBase):
    def init(
        self,
        num_embeddings: int,
        embedding_dim: int,
        num_offsets: int,
        enable_per_sample_weights: bool,
        include_last_offset: bool,
        is_pruned_weights: bool,
        use_32bit_indices: bool,
        use_32bit_offsets: bool,
        op_func,
    ):
        self.num_embeddings = num_embeddings
        self.embedding_dim = embedding_dim
```

- **EN:** Important local symbols in this block include EmbedddingBagByteRowwiseOffsetsTest, init.
- **CN:** 该代码块中的重要局部符号包括 EmbedddingBagByteRowwiseOffsetsTest、init。

### Lines 213-224 / 第 213-224 行

```python
        self.num_offsets = num_offsets
        self.enable_per_sample_weights = enable_per_sample_weights
        self.include_last_offset = include_last_offset
        self.max_segment_length = 20
        self.num_lengths = np.random.randint(1, num_offsets + 1)
        self.lengths = np.random.randint(
            0, self.max_segment_length + 1, size=self.num_lengths
        ).astype(np.int32)
        self.is_pruned_weights = is_pruned_weights
        self.use_32bit_indices = use_32bit_indices
        self.use_32bit_offsets = use_32bit_offsets

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 225-232 / 第 225-232 行

```python
        self.num_indices = np.sum(self.lengths)
        self.offsets = lengths_to_offsets(self.lengths)
        self.indices = torch.from_numpy(
            np.random.randint(
                low=0, high=num_embeddings, size=self.num_indices, dtype=np.int64
            )
        )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 233-241 / 第 233-241 行

```python
        self.indices = self.indices.int() if self.use_32bit_indices else self.indices
        self.offsets = self.offsets.int() if self.use_32bit_offsets else self.offsets

        if include_last_offset:
            self.offsets = torch.cat(
                (self.offsets, torch.tensor([self.indices.size(0)], dtype=torch.long)),
                0,
            )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 242-252 / 第 242-252 行

```python
        self.weights = torch.from_numpy(
            (
                np.random.random_sample((self.num_embeddings, self.embedding_dim)) + 1
            ).astype(np.float32)
        )
        self.indices = torch.from_numpy(
            np.random.randint(
                low=0, high=self.num_embeddings, size=self.num_indices, dtype=np.int64
            )
        )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 253-262 / 第 253-262 行

```python
        self.prepack_func = torch.ops.quantized.embedding_bag_byte_prepack

        self.prepacked_weights = self.prepack_func(self.weights)
        self.per_sample_weights = (
            torch.from_numpy(
                np.random.uniform(low=0.01, high=0.5, size=[len(self.indices)]).astype(
                    np.float32
                )
            )
            if self.enable_per_sample_weights
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 263-273 / 第 263-273 行

```python
            else None
        )

        self.compressed_indices = None

        if self.is_pruned_weights:
            (
                self.prepacked_weights,
                self.compressed_indices,
            ) = get_pruned_weights_and_mapping(self.prepacked_weights)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 274-284 / 第 274-284 行

```python
        self.inputs = {
            "prepacked_weights": self.prepacked_weights,
            "indices": self.indices,
            "offsets": self.offsets,
            "mode": 0,
            "per_sample_weights": self.per_sample_weights,
            "include_last_offset": self.include_last_offset,
            "is_pruned_weights": self.is_pruned_weights,
            "compressed_indices": self.compressed_indices,
        }

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 285-298 / 第 285-298 行

```python
        self.op_func = op_func

    def forward(
        self,
        prepacked_weights,
        indices,
        offsets,
        mode: int,
        per_sample_weights: torch.Tensor | None,
        include_last_offset: bool,
        is_pruned_weights: bool,
        compressed_indices: torch.Tensor | None,
    ):
        return self.op_func(
```

- **EN:** Important local symbols in this block include forward.
- **CN:** 该代码块中的重要局部符号包括 forward。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 299-308 / 第 299-308 行

```python
            prepacked_weights,
            indices,
            offsets,
            mode=0,
            per_sample_weights=per_sample_weights,
            include_last_offset=self.include_last_offset,
            pruned_weights=self.is_pruned_weights,
            compressed_indices_mapping=self.compressed_indices,
        )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 309-316 / 第 309-316 行

```python

op_bench.generate_pt_tests_from_op_list(
    four_bit_rowwise_ops, full_configs, EmbedddingBag4BitRowwiseOffsetsTest
)
op_bench.generate_pt_tests_from_op_list(
    byte_rowwise_ops, full_configs, EmbedddingBagByteRowwiseOffsetsTest
)

```

- **EN:** This portion registers the prepared benchmark definitions with the operator_benchmark runner.
- **CN:** 这一部分把准备好的 benchmark 定义注册到 operator_benchmark 运行器中。

### Lines 317-319 / 第 317-319 行

```python

if __name__ == "__main__":
    op_bench.benchmark_runner.main()
```

- **EN:** This portion registers the prepared benchmark definitions with the operator_benchmark runner.
- **CN:** 这一部分把准备好的 benchmark 定义注册到 operator_benchmark 运行器中。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

## Key Concepts / 关键概念

- **PyTorch benchmark suite** — PyTorch 基准测试套件
- **Operator benchmark harness** — 使用 operator_benchmark 框架注册 PyTorch 性能测试用例。
- **Generated benchmark registration** — 把基准类和配置转化为可运行的 benchmark 条目。
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Representative symbols: EmbedddingBag4BitRowwiseOffsetsTest, EmbedddingBagByteRowwiseOffsetsTest, get_pruned_weights_and_mapping, init, forward** — 代表性符号：EmbedddingBag4BitRowwiseOffsetsTest、EmbedddingBagByteRowwiseOffsetsTest、get_pruned_weights_and_mapping、init、forward

## Dependencies / 依赖关系

- `numpy`
- `operator_benchmark`
- `torch`
- `torch.testing._internal.common_quantization`

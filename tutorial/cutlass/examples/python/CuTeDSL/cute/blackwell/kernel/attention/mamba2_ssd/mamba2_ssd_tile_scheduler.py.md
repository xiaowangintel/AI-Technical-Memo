# mamba2_ssd_tile_scheduler.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/kernel/attention/mamba2_ssd/mamba2_ssd_tile_scheduler.py`  
**Purpose / 用途**: Kernel example implementing mamba2 ssd tile scheduler with CuTeDSL. / 这是一个使用 CuTeDSL 实现 mamba2 ssd tile scheduler 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行

~~~~python
# Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause

# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:

# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.

# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.

# 3. Neither the name of the copyright holder nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.

# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
~~~~

**EN**: Records the copyright, SPDX tags, and license conditions that govern how this example may be used and redistributed.
**CN**: 记录版权、SPDX 标记和许可证条件，说明该示例如何被使用和再分发。

### Lines 29-31 / 第 29-31 行

~~~~python
from typing import Tuple

from cutlass.cutlass_dsl import (
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 32-41 / 第 32-41 行

~~~~python
    Integer,
    Int32,
    min,
    extract_mlir_values,
    new_from_mlir_values,
    dsl_user_op,
)
from cutlass._mlir import ir
import cutlass.cute as cute
from cutlass.utils import WorkTileInfo
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 44-57 / 第 44-57 行

~~~~python
class Mamba2SSDTileSchedulerParams:
    def __init__(
        self,
        problem_shape_ntiles: int,
        eh: int,
        ngroup_ratio: int,
        *,
        loc=None,
        ip=None,
    ):
        self.problem_shape_ntiles = problem_shape_ntiles
        self.eh = eh
        self.ngroup_ratio = ngroup_ratio
        self._loc = loc
~~~~

**EN**: Defines `Mamba2SSDTileSchedulerParams`, a reusable Python class that packages configuration and behavior for this example. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone.
**CN**: 定义 `Mamba2SSDTileSchedulerParams`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。

### Lines 59-65 / 第 59-65 行

~~~~python
    def __extract_mlir_values__(self):
        values, self._values_pos = [], []
        for obj in [self.problem_shape_ntiles, self.eh, self.ngroup_ratio]:
            obj_values = extract_mlir_values(obj)
            values += obj_values
            self._values_pos.append(len(obj_values))
        return values
~~~~

**EN**: Defines `__extract_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__extract_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 67-74 / 第 67-74 行

~~~~python
    def __new_from_mlir_values__(self, values):
        obj_list = []
        for obj, n_items in zip(
            [self.problem_shape_ntiles, self.eh, self.ngroup_ratio], self._values_pos
        ):
            obj_list.append(new_from_mlir_values(obj, values[:n_items]))
            values = values[n_items:]
        return Mamba2SSDTileSchedulerParams(*(tuple(obj_list)), loc=self._loc)
~~~~

**EN**: Defines `__new_from_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__new_from_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 把构造好的对象或计算结果返回给调用方。

### Lines 76-80 / 第 76-80 行

~~~~python
    @dsl_user_op
    def get_grid_shape(
        self, max_active_clusters: Int32, *, loc=None, ip=None
    ) -> Tuple[Integer, Integer, Integer]:
        return (min(self.problem_shape_ntiles, max_active_clusters), 1, 1)
~~~~

**EN**: Defines `get_grid_shape` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Returns the constructed object or computed result to the caller.
**CN**: 将 `get_grid_shape` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 把构造好的对象或计算结果返回给调用方。

### Lines 83-94 / 第 83-94 行

~~~~python
class Mamba2SSDTileScheduler:
    def __init__(
        self,
        params: Mamba2SSDTileSchedulerParams,
        num_persistent_ctas: Int32,
        current_work_linear_idx: Int32,
        num_tiles_executed: Int32,
    ):
        self.params = params
        self.num_persistent_ctas = num_persistent_ctas
        self._current_work_linear_idx = current_work_linear_idx
        self._num_tiles_executed = num_tiles_executed
~~~~

**EN**: Defines `Mamba2SSDTileScheduler`, a reusable Python class that packages configuration and behavior for this example. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone.
**CN**: 定义 `Mamba2SSDTileScheduler`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。

### Lines 96-100 / 第 96-100 行

~~~~python
    def __extract_mlir_values__(self) -> list[ir.Value]:
        values = extract_mlir_values(self.num_persistent_ctas)
        values.extend(extract_mlir_values(self._current_work_linear_idx))
        values.extend(extract_mlir_values(self._num_tiles_executed))
        return values
~~~~

**EN**: Defines `__extract_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__extract_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 102-120 / 第 102-120 行

~~~~python
    def __new_from_mlir_values__(
        self, values: list[ir.Value]
    ) -> "Mamba2SSDTileScheduler":
        assert len(values) == 3
        new_num_persistent_ctas = new_from_mlir_values(
            self.num_persistent_ctas, [values[0]]
        )
        new_current_work_linear_idx = new_from_mlir_values(
            self._current_work_linear_idx, [values[1]]
        )
        new_num_tiles_executed = new_from_mlir_values(
            self._num_tiles_executed, [values[2]]
        )
        return Mamba2SSDTileScheduler(
            self.params,
            new_num_persistent_ctas,
            new_current_work_linear_idx,
            new_num_tiles_executed,
        )
~~~~

**EN**: Defines `__new_from_mlir_values__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message. Returns the constructed object or computed result to the caller.
**CN**: 定义 `__new_from_mlir_values__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。 把构造好的对象或计算结果返回给调用方。

### Lines 122-122 / 第 122-122 行

~~~~python
    # called by host
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 123-133 / 第 123-133 行

~~~~python
    @staticmethod
    @dsl_user_op
    def create(
        params: Mamba2SSDTileSchedulerParams,
        block_idx: Tuple[Integer, Integer, Integer],
        grid_dim: Tuple[Integer, Integer, Integer],
        *,
        loc=None,
        ip=None,
    ):
        params = params
~~~~

**EN**: Defines `create` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone.
**CN**: 将 `create` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。

### Lines 135-136 / 第 135-136 行

~~~~python
        # Calculate the number of persistent clusters by dividing the total grid size
        # by the number of CTAs per cluster
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 137-137 / 第 137-137 行

~~~~python
        num_persistent_ctas = Int32(cute.size(grid_dim, loc=loc, ip=ip))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 139-139 / 第 139-139 行

~~~~python
        bidx, bidy, bidz = block_idx
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 141-141 / 第 141-141 行

~~~~python
        # Initialize workload index equals to the cluster index in the grid
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 142-142 / 第 142-142 行

~~~~python
        current_work_linear_idx = Int32(bidx)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 144-144 / 第 144-144 行

~~~~python
        # Initialize number of tiles executed to zero
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 145-151 / 第 145-151 行

~~~~python
        num_tiles_executed = Int32(0)
        return Mamba2SSDTileScheduler(
            params,
            num_persistent_ctas,
            current_work_linear_idx,
            num_tiles_executed,
        )
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Returns the constructed object or computed result to the caller.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 把构造好的对象或计算结果返回给调用方。

### Lines 153-153 / 第 153-153 行

~~~~python
    # called by host
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 154-162 / 第 154-162 行

~~~~python
    @staticmethod
    def get_grid_shape(
        params: Mamba2SSDTileSchedulerParams,
        max_active_clusters: Int32,
        *,
        loc=None,
        ip=None,
    ) -> Tuple[Integer, Integer, Integer]:
        return params.get_grid_shape(max_active_clusters, loc=loc, ip=ip)
~~~~

**EN**: Defines `get_grid_shape`, grouping related logic behind a named Python callable so the example stays modular and reusable. Implements state-space style sequence processing, where the DSL is used to express specialized dataflow rather than plain GEMM alone. Returns the constructed object or computed result to the caller.
**CN**: 定义 `get_grid_shape`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 实现状态空间模型风格的序列处理，此时 DSL 表达的是专门的数据流，而不仅仅是普通 GEMM。 把构造好的对象或计算结果返回给调用方。

### Lines 164-164 / 第 164-164 行

~~~~python
    # private method
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 165-170 / 第 165-170 行

~~~~python
    def _get_current_work_for_linear_idx(
        self, current_work_linear_idx: Int32, *, loc=None, ip=None
    ) -> WorkTileInfo:
        is_valid = current_work_linear_idx < cute.size(
            self.params.problem_shape_ntiles, loc=loc, ip=ip
        )
~~~~

**EN**: Defines `_get_current_work_for_linear_idx`, grouping related logic behind a named Python callable so the example stays modular and reusable. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 定义 `_get_current_work_for_linear_idx`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 172-176 / 第 172-176 行

~~~~python
        eh_idx = current_work_linear_idx % self.params.eh
        b_idx = current_work_linear_idx // self.params.eh
        g_idx = eh_idx // self.params.ngroup_ratio
        # cur_tile_coord is (b_idx, eh_idx, g_idx)
        cur_tile_coord = tuple(Int32(x) for x in (b_idx, eh_idx, g_idx))
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 178-178 / 第 178-178 行

~~~~python
        return WorkTileInfo(cur_tile_coord, is_valid)
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 180-184 / 第 180-184 行

~~~~python
    @dsl_user_op
    def get_current_work(self, *, loc=None, ip=None) -> WorkTileInfo:
        return self._get_current_work_for_linear_idx(
            self._current_work_linear_idx, loc=loc, ip=ip
        )
~~~~

**EN**: Defines `get_current_work` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 将 `get_current_work` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 186-188 / 第 186-188 行

~~~~python
    @dsl_user_op
    def initial_work_tile_info(self, *, loc=None, ip=None) -> WorkTileInfo:
        return self.get_current_work(loc=loc, ip=ip)
~~~~

**EN**: Defines `initial_work_tile_info` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Returns the constructed object or computed result to the caller.
**CN**: 将 `initial_work_tile_info` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 把构造好的对象或计算结果返回给调用方。

### Lines 190-195 / 第 190-195 行

~~~~python
    @dsl_user_op
    def advance_to_next_work(self, *, advance_count: int = 1, loc=None, ip=None):
        self._current_work_linear_idx += Int32(advance_count) * Int32(
            self.num_persistent_ctas
        )
        self._num_tiles_executed += Int32(1)
~~~~

**EN**: Defines `advance_to_next_work` as a custom DSL user operation, extending CuTe/CUTLASS with a low-level primitive that can be emitted into GPU IR. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU.
**CN**: 将 `advance_to_next_work` 定义为自定义 DSL 用户操作，用于扩展 CuTe/CUTLASS，使低层原语能够发射到 GPU IR 中。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。

### Lines 197-199 / 第 197-199 行

~~~~python
    @property
    def num_tiles_executed(self) -> Int32:
        return self._num_tiles_executed
~~~~

**EN**: Defines `num_tiles_executed`, grouping related logic behind a named Python callable so the example stays modular and reusable. Returns the constructed object or computed result to the caller.
**CN**: 定义 `num_tiles_executed`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把构造好的对象或计算结果返回给调用方。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Specialized sequence or attention dataflow / 专门化的序列/注意力数据流
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `typing.Tuple` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cutlass.cutlass_dsl.Integer` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.Int32` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.min` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.extract_mlir_values` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.new_from_mlir_values` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass.cutlass_dsl.dsl_user_op` — exposes lower-level CUTLASS DSL utilities and IR helpers / 提供更低层的 CUTLASS DSL 工具与 IR 辅助函数
- `cutlass._mlir.ir` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute` — provides core CuTeDSL tensor, layout, and kernel primitives / 提供 CuTeDSL 核心张量、布局与内核原语
- `cutlass.utils.WorkTileInfo` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成

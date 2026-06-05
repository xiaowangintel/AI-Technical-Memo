# triton_split_scan.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/triton_split_scan.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `TritonSplitScanKernel`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `TritonSplitScanKernel` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
import functools

import sympy

from torch._inductor import config
from torch._inductor.codegen.simd import IterationRangesRoot, prefix_is_reduction
from torch._inductor.codegen.triton import (
    triton_compute_type,
    TritonCSEVariable,
    TritonKernel,
)
from torch._inductor.runtime.triton_heuristics import SplitScanGrid
from torch.utils._ordered_set import OrderedSet
````
- **EN**: Imports dependencies such as `functools`, `sympy`, `torch._inductor`, `torch._inductor.codegen.simd`, `torch._inductor.codegen.triton`, `torch._inductor.runtime.triton_heuristics`, and `...+1` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `functools`、`sympy`、`torch._inductor`、`torch._inductor.codegen.simd`、`torch._inductor.codegen.triton`、`torch._inductor.runtime.triton_heuristics`、`另有1项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 15-28 / 第 15-28 行
````python
from torch.utils._sympy.functions import CeilDiv

from ..utils import sympy_product


class TritonSplitScanKernel(TritonKernel):
    """Generates a triton kernel that supports ops.scan calls while also splitting
    the reduction dimension over multiple triton programs.

    For this kernel, loop numels will always take the form ``(xdim, rdim)``
    and the grid has the shape ``(CeilDiv(rdim, RBLOCK), xdim)``. Communication
    between blocks occurs within a global memory workspace buffer, which
    must be zero-filled before launching the kernel.

````
- **EN**: Imports dependencies such as `torch.utils._sympy.functions`, and `..utils` for the logic in this range. Introduces class `TritonSplitScanKernel`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `torch.utils._sympy.functions`、`..utils` 等依赖，为后续逻辑提供基础能力。这里定义了类`TritonSplitScanKernel`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 29-42 / 第 29-42 行
````python
    Note that generation for ``ops.reduction`` is not supported.

    For details of the communication strategy, see
    https://research.nvidia.com/publication/2016-03_single-pass-parallel-prefix-scan-decoupled-look-back

    """

    def __init__(
        self,
        tiling: dict[str, sympy.Expr],
        pid_cache=None,
        fixed_config=None,
        **kwargs,
    ) -> None:
````
- **EN**: Introduces function `__init__`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了函数`__init__`。保留了指向设计说明、规范或厂商数据手册的注释引用。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 43-56 / 第 43-56 行
````python
        assert pid_cache is None, "not supported"
        assert fixed_config is None, "not supported"
        super().__init__(
            tiling,
            **kwargs,
        )
        self.no_x_dim = True

    def should_use_persistent_reduction(self) -> bool:
        return False

    def should_use_cooperative_reduction(self) -> bool:
        return False

````
- **EN**: Introduces function `should_use_persistent_reduction`, function `should_use_cooperative_reduction`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`should_use_persistent_reduction`、函数`should_use_cooperative_reduction`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 57-70 / 第 57-70 行
````python
    def initialize_range_tree(self, pid_cache):
        prefixes = ["y", "x", "r0_"]
        assert len(self.numels) <= len(prefixes), (
            "z dimension not supported for split scan"
        )
        active_prefixes = prefixes[len(prefixes) - len(self.numels) :]

        grid_dims = {"r0_": 0, "x": 1, "y": 2}
        for prefix in active_prefixes:
            numel = self.numels[prefix]
            tensor_dim = 0 if prefix_is_reduction(prefix) else None
            grid_dim = grid_dims[prefix]
            self.range_trees.append(
                IterationRangesRoot(
````
- **EN**: Introduces function `initialize_range_tree`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`initialize_range_tree`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 71-84 / 第 71-84 行
````python
                    f"{prefix}index",
                    numel,
                    prefix,
                    grid_dim,
                    self,  # type: ignore[arg-type]
                    pid_cache=pid_cache,
                    is_loop=False,
                    tensor_dim=tensor_dim,
                    grid_dim=grid_dim,
                    has_zdim=False,
                )
            )

    def reduction(self, dtype, src_dtype, reduction_type, value):
````
- **EN**: Introduces function `reduction`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `pid_cache`, `is_loop`, `tensor_dim`, `grid_dim`, and `has_zdim`.
- **CN**: 这里定义了函数`reduction`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `pid_cache`、`is_loop`、`tensor_dim`、`grid_dim`、`has_zdim` 等值。

### Lines 85-98 / 第 85-98 行
````python
        raise NotImplementedError("NYI TritonSplitDimKernel reductions")

    def scan(self, dtypes, combine_fn, values):
        """
        Perform an associative scan on 'values'.
        """
        import triton.language as tl

        (dtype,) = dtypes
        (value,) = values

        compute_type = triton_compute_type(dtype)
        compute_type_triton = getattr(tl, compute_type[3:])

````
- **EN**: Imports dependencies such as `triton.language` for the logic in this range. Introduces function `scan`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `triton.language` 等依赖，为后续逻辑提供基础能力。这里定义了函数`scan`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 99-112 / 第 99-112 行
````python
        element_nbits = compute_type_triton.primitive_bitwidth

        scratch_type = "tl.uint32" if element_nbits <= 16 else "tl.uint64"
        scratch_type_triton = getattr(tl, scratch_type[3:])
        scratch_elems_per_block = 3 if element_nbits == 64 else 1
        scratch_nbytes_per_block = scratch_elems_per_block * (
            scratch_type_triton.primitive_bitwidth // 8
        )

        cse_load = functools.partial(self.cse.generate, self.loads, dtype=dtype)
        cse_compute = functools.partial(self.cse.generate, self.compute)

        assert len(self.numels) == 2, "Unexpected tiling"
        min_rblock = config.triton.min_split_scan_rblock
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `element_nbits`, `scratch_type`, `scratch_type_triton`, `scratch_elems_per_block`, `scratch_nbytes_per_block`, `cse_load`, and `...+2`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `element_nbits`、`scratch_type`、`scratch_type_triton`、`scratch_elems_per_block`、`scratch_nbytes_per_block`、`cse_load`、`另有2项` 等值。

### Lines 113-126 / 第 113-126 行
````python
        reduction_numel = sympy_product(
            numel
            for prefix, numel in self.numels.items()
            if prefix_is_reduction(prefix)
        )
        pointwise_numel = sympy_product(
            numel
            for prefix, numel in self.numels.items()
            if not prefix_is_reduction(prefix)
        )
        max_blocks = pointwise_numel * CeilDiv(reduction_numel, min_rblock)
        nbytes = scratch_nbytes_per_block * max_blocks
        scratch_base: str | TritonCSEVariable
        scratch_base, _, offset = self.args.workspace(nelem=nbytes, zero_fill=True)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `reduction_numel`, `pointwise_numel`, `max_blocks`, `nbytes`, and `scratch_base`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `reduction_numel`、`pointwise_numel`、`max_blocks`、`nbytes`、`scratch_base` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
        if offset != 0:
            scratch_base = cse_load(
                f"{scratch_base} + {self.index_to_str(offset)}", shape=()
            )
        runtime_rblocks = cse_load(
            f"tl.num_programs({self.range_trees[-1].index})", shape=()
        )
        scratch_base = cse_load(
            f"{scratch_base}.to(tl.pointer_type({scratch_type})) + xoffset * "
            f"{scratch_elems_per_block} * {runtime_rblocks}",
            shape=(),
        )

        masks = OrderedSet(f"{tree.prefix}mask" for tree in self.range_trees)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scratch_base`, `runtime_rblocks`, `shape`, and `masks`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `scratch_base`、`runtime_rblocks`、`shape`、`masks` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
        self.filter_masks(masks)
        assert not self._load_mask, "ops.scan not supported inside ops.masked"

        value = cse_compute(
            f"{value}.to({compute_type})",
            dtype=dtype,
            shape=value.shape,
        )
        value = cse_compute(
            f"tl.broadcast_to({value}, {self.dense_size_str()})",
            dtype=dtype,
            shape=self.dense_size_list(),
        )

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `value`, `dtype`, and `shape`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `value`、`dtype`、`shape` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python
        combine_helper_fn = self._lift_helper(combine_fn, (value,), (dtype,))
        dim = self.triton_tensor_ndim() - 1
        assert dim == 0, ""
        shape = list(self.dense_size_list())
        del shape[dim]

        block_sum = cse_compute(
            f"tl.reduce({value}, {dim}, {combine_helper_fn})",
            dtype=dtype,
            shape=shape,
        )
        exclusive_prefix = self.cse.newvar(
            dtype=dtype,
            shape=shape,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `combine_helper_fn`, `dim`, `shape`, `block_sum`, `dtype`, and `exclusive_prefix`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `combine_helper_fn`、`dim`、`shape`、`block_sum`、`dtype`、`exclusive_prefix` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 169-182 / 第 169-182 行
````python
        )
        if element_nbits == 64:
            self.compute.splice(
                f"""
                {exclusive_prefix} = triton_helpers.exclusive_scan_decoupled_lookback_64(
                    {scratch_base},
                    {block_sum},
                    {self.iteration_ranges_get_pid(self.range_trees[-1])},
                    {combine_helper_fn},
                )
                """,
                strip=True,
            )

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `strip`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `strip` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 183-196 / 第 183-196 行
````python
        else:
            assert element_nbits <= 32
            value_as_uint_dtype = f"tl.uint{element_nbits}"

            self.compute.splice(
                f"""
                {exclusive_prefix} = triton_helpers.exclusive_scan_decoupled_lookback(
                    {scratch_base},
                    {block_sum},
                    {self.iteration_ranges_get_pid(self.range_trees[-1])},
                    {combine_helper_fn},
                    DTYPE_VALUE_AS_UINT={value_as_uint_dtype},
                    DTYPE_PACK={scratch_type},
                )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `value_as_uint_dtype`, `DTYPE_VALUE_AS_UINT`, and `DTYPE_PACK`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`value_as_uint_dtype`、`DTYPE_VALUE_AS_UINT`、`DTYPE_PACK` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 197-210 / 第 197-210 行
````python
                """,
                strip=True,
            )
        # Compute final cumsum
        block_scan = cse_compute(
            f"tl.associative_scan({value}, {dim}, {combine_helper_fn})",
            dtype=dtype,
            shape=shape,
        )
        combined_result = cse_compute(
            f"{combine_helper_fn}({exclusive_prefix}, {block_scan})",
            dtype=dtype,
            shape=shape,
        )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `strip`, `block_scan`, `dtype`, `shape`, and `combined_result`. This range continues the implementation of function `TritonSplitScanKernel.scan`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `strip`、`block_scan`、`dtype`、`shape`、`combined_result` 等值。这一段延续了函数`TritonSplitScanKernel.scan` 的具体实现。

### Lines 211-223 / 第 211-223 行
````python
        return (
            cse_compute(
                f"tl.where(roffset == 0, {block_scan}, {combined_result})",
                dtype=dtype,
                shape=block_scan.shape,
            ),
        )

    def _get_heuristic(self):
        return "split_scan"

    def _get_grid_type(self) -> type[SplitScanGrid]:
        return SplitScanGrid
````
- **EN**: Introduces function `_get_heuristic`, function `_get_grid_type`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dtype`, and `shape`.
- **CN**: 这里定义了函数`_get_heuristic`、函数`_get_grid_type`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dtype`、`shape` 等值。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `TritonSplitScanKernel`  
  **CN**: 主要类：`TritonSplitScanKernel`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`
- **Third-party / 第三方**: `sympy`, `triton.language`
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor`, `torch._inductor.codegen.simd`, `torch._inductor.codegen.triton`, `torch._inductor.runtime.triton_heuristics`, `torch.utils._ordered_set`, `torch.utils._sympy.functions`, `..utils`

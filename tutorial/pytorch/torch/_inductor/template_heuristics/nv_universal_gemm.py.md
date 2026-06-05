# nv_universal_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/template_heuristics/nv_universal_gemm.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `HeuristicConfig`, and `NVUniversalGemmHeuristics`. It exposes functions such as `_make_config_key_from_heuristic`, `_make_config_key_from_kernel_design`, `_make_config_key_from_heuristics_kernel`, and `get_nvgemm_heuristics`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `HeuristicConfig`、`NVUniversalGemmHeuristics` 等类。同时提供 `_make_config_key_from_heuristic`、`_make_config_key_from_kernel_design`、`_make_config_key_from_heuristics_kernel`、`get_nvgemm_heuristics` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import logging
from collections import defaultdict
from dataclasses import dataclass
from typing import TYPE_CHECKING

import torch
from torch._inductor.utils import ensure_nvmatmul_heuristics_available
from torch._logging import getArtifactLogger
from torch.utils._ordered_set import OrderedSet

from .gemm import GemmMaxAutotuneTemplateConfigHeuristics


if TYPE_CHECKING:
    from ..kernel_inputs import KernelInputs, MMKernelInputs


log = logging.getLogger(__name__)
````
- **EN**: Imports dependencies such as `__future__`, `logging`, `collections`, `dataclasses`, `typing`, `torch`, and `...+5` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `__future__`、`logging`、`collections`、`dataclasses`、`typing`、`torch`、`另有5项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。初始化或更新了 `log` 等值。

### Lines 21-40 / 第 21-40 行
````python
# Use autotuning artifact logger for detailed nvMatmulHeuristics logging
# Enable with TORCH_LOGS="+autotuning"
autotuning_log = getArtifactLogger(__name__, "autotuning")

# Type alias for kernel config key tuple.
# Currently matches on (tile_m, tile_n, cluster_m, cluster_n).
# tile_k excluded because nvMatmulHeuristics and cutlass_api use it to mean different things.
# TODO(nikhilap): Extend config key for stages/split_k https://github.com/pytorch/pytorch/issues/177578
ConfigKey = tuple[int, int, int, int]


@dataclass
class HeuristicConfig:
    """Configuration recommended by nvMatmulHeuristics."""

    tile_m: int
    tile_n: int
    tile_k: int
    cluster_m: int
    cluster_n: int
````
- **EN**: Introduces class `HeuristicConfig`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里定义了类`HeuristicConfig`。保留了指向设计说明、规范或厂商数据手册的注释引用。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 41-60 / 第 41-60 行
````python
    stages: int
    split_k: int
    warp_tile_m: int
    warp_tile_n: int
    warp_tile_k: int
    estimated_runtime: float


def _make_config_key_from_heuristic(cfg: HeuristicConfig) -> ConfigKey:
    """Build config key from HeuristicConfig returned by nvMatmulHeuristics."""
    return (cfg.tile_m, cfg.tile_n, cfg.cluster_m, cfg.cluster_n)


def _make_config_key_from_kernel_design(design) -> ConfigKey | None:
    """Build config key from cutlass_api kernel metadata.design."""
    if (
        hasattr(design, "tile_shape")
        and len(design.tile_shape) >= 2
        and hasattr(design, "cluster_shape")
        and len(design.cluster_shape) >= 2
````
- **EN**: Introduces function `_make_config_key_from_heuristic`, function `_make_config_key_from_kernel_design`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `stages`, `split_k`, `warp_tile_m`, `warp_tile_n`, `warp_tile_k`, and `estimated_runtime`.
- **CN**: 这里定义了函数`_make_config_key_from_heuristic`、函数`_make_config_key_from_kernel_design`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `stages`、`split_k`、`warp_tile_m`、`warp_tile_n`、`warp_tile_k`、`estimated_runtime` 等值。

### Lines 61-80 / 第 61-80 行
````python
    ):
        return (
            design.tile_shape[0],
            design.tile_shape[1],
            design.cluster_shape[0],
            design.cluster_shape[1],
        )
    return None


def _make_config_key_from_heuristics_kernel(kernel) -> ConfigKey:
    """Build config key from nvMatmulHeuristics kernel config struct."""
    return (
        kernel.cta[0],
        kernel.cta[1],
        kernel.cluster[0],
        kernel.cluster[1],
    )


````
- **EN**: Introduces function `_make_config_key_from_heuristics_kernel`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_make_config_key_from_heuristics_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python
class NVUniversalGemmHeuristics(GemmMaxAutotuneTemplateConfigHeuristics):
    """
    Heuristics for NVGEMM kernel selection using nvMatmulHeuristics.
    """

    def should_run(self, inputs: KernelInputs) -> bool:
        """Check if heuristics should be used.

        Args:
            inputs: KernelInputs
        """
        return super().should_run(inputs) and ensure_nvmatmul_heuristics_available()

    def filter_kernels(
        self,
        kernels: list,
        inputs: MMKernelInputs,
        count: int,
        accumulator_type: torch.dtype = torch.float32,
    ) -> list:
````
- **EN**: Introduces class `NVUniversalGemmHeuristics`, function `should_run`, function `filter_kernels`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Args`, `inputs`, `kernels`, `count`, and `accumulator_type`.
- **CN**: 这里定义了类`NVUniversalGemmHeuristics`、函数`should_run`、函数`filter_kernels`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Args`、`inputs`、`kernels`、`count`、`accumulator_type` 等值。

### Lines 101-120 / 第 101-120 行
````python
        """
        Filter and rank kernels using nvMatmulHeuristics.

        Matches on (tile_m, tile_n, tile_k, cluster_m, cluster_n).
        Returns kernels sorted by estimated runtime.

        If nvMatmulHeuristics is not installed or max_autotune is disabled,
        returns the first `count` kernels without heuristic ranking.

        Args:
            kernels: List of cutlass_api.Kernel objects
            inputs: MMKernelInputs with matrix shapes, dtypes, and strides
            count: Maximum number of kernels to return
            accumulator_type: Accumulator dtype

        Returns:
            Filtered list of kernels, sorted by estimated performance
        """
        if not self.should_run(inputs):
            return kernels[:count]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `kernels`, `inputs`, `count`, `accumulator_type`, and `Returns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`kernels`、`inputs`、`count`、`accumulator_type`、`Returns` 等值。

### Lines 121-140 / 第 121-140 行
````python

        m, n, k = inputs.mnk_hinted()
        batch_size = inputs.batch_hinted()
        dtype_a = inputs.dtype(inputs._mat1_idx)
        dtype_b = inputs.dtype(inputs._mat2_idx)
        out_dtype = inputs.out_dtype()
        strides = inputs.strides_hinted()
        layout_a = "row" if strides[inputs._mat1_idx][-1] == 1 else "col"
        layout_b = "row" if strides[inputs._mat2_idx][-1] == 1 else "col"

        config_to_kernels = self._extract_config_to_kernels(kernels)

        if not config_to_kernels:
            log.debug("Could not extract kernel configs, using first %d kernels", count)
            return kernels[:count]

        heuristic_configs = self._get_heuristic_configs(
            m,
            n,
            k,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `batch_size`, `dtype_a`, `dtype_b`, `out_dtype`, `strides`, `layout_a`, and `...+3`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `batch_size`、`dtype_a`、`dtype_b`、`out_dtype`、`strides`、`layout_a`、`另有3项` 等值。

### Lines 141-160 / 第 141-160 行
````python
            dtype_a,
            layout_a,
            layout_b,
            count,
            OrderedSet(config_to_kernels.keys()),
            accumulator_type,
            batch_size,
            dtype_b=dtype_b,
            out_dtype=out_dtype,
        )

        if not heuristic_configs:
            log.debug("No heuristic configs found, using first %d kernels", count)
            return kernels[:count]

        # Match kernels to heuristic configs
        matched: list[tuple] = []
        for cfg in heuristic_configs:
            key = _make_config_key_from_heuristic(cfg)
            kernels_for_key = config_to_kernels.get(key)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dtype_b`, `out_dtype`, `matched`, `key`, and `kernels_for_key`. This range continues the implementation of function `NVUniversalGemmHeuristics.filter_kernels`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dtype_b`、`out_dtype`、`matched`、`key`、`kernels_for_key` 等值。这一段延续了函数`NVUniversalGemmHeuristics.filter_kernels` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
            if not kernels_for_key:
                continue
            for kernel in kernels_for_key:
                matched.append((kernel, cfg.estimated_runtime))

        if not matched:
            log.debug(
                "No kernels matched heuristic configs, using first %d kernels", count
            )
            return kernels[:count]

        matched.sort(key=lambda x: x[1])
        selected = matched[:count]
        result = [k for k, _ in selected]

        log.debug(
            "Heuristic filtered to %d kernels from %d total", len(result), len(kernels)
        )

        autotuning_log.info(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `selected`, and `result`. This range continues the implementation of function `NVUniversalGemmHeuristics.filter_kernels`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `selected`、`result` 等值。这一段延续了函数`NVUniversalGemmHeuristics.filter_kernels` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
            "nvMatmulHeuristics kernel filtering: %d heuristic configs matched %d "
            "of %d available kernels, returning top %d",
            len(heuristic_configs),
            len(matched),
            len(kernels),
            len(result),
        )
        for i, (kernel, runtime) in enumerate(selected):
            design = kernel.metadata.design
            autotuning_log.info(
                "  Selected kernel %d: tile=(%d, %d, %d), cluster=(%d, %d), "
                "estimated_runtime=%.2f us",
                i,
                design.tile_shape[0],
                design.tile_shape[1],
                design.tile_shape[2],
                design.cluster_shape[0],
                design.cluster_shape[1],
                runtime * 1e6,
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `design`. This range continues the implementation of function `NVUniversalGemmHeuristics.filter_kernels`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `design` 等值。这一段延续了函数`NVUniversalGemmHeuristics.filter_kernels` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python

        return result

    def _extract_config_to_kernels(self, kernels: list) -> dict[ConfigKey, list]:
        """Build a map from config key to kernels."""
        config_to_kernels: dict[ConfigKey, list] = defaultdict(list)

        for kernel in kernels:
            key = _make_config_key_from_kernel_design(kernel.metadata.design)
            if key is not None:
                config_to_kernels[key].append(kernel)

        return config_to_kernels

    def _get_layout_enum(self, layout_a: str, layout_b: str):
        """Map layout strings to NvMatmulHeuristicsMatmulLayout enum."""
        import nvMatmulHeuristics

        trans_a = "T" if layout_a == "row" else "N"
        trans_b = "T" if layout_b == "row" else "N"
````
- **EN**: Imports dependencies such as `nvMatmulHeuristics` for the logic in this range. Introduces function `_extract_config_to_kernels`, function `_get_layout_enum`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `nvMatmulHeuristics` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_extract_config_to_kernels`、函数`_get_layout_enum`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
        layout_str = f"{trans_a}{trans_b}_ROW_MAJOR"
        return nvMatmulHeuristics.NvMatmulHeuristicsMatmulLayout[layout_str]

    def _make_validity_callback(
        self,
        valid_configs: OrderedSet[ConfigKey],
    ):
        """
        Create callback for nvMatmulHeuristics that only accepts configurations
        matching the available cutlass_api kernel tile/cluster shapes.
        """

        def validity_check(kernel_config_ptr, problem_ptr):
            kernel = kernel_config_ptr.contents
            key = _make_config_key_from_heuristics_kernel(kernel)
            return 1 if key in valid_configs else 0

        return validity_check

    def _get_heuristic_configs(
````
- **EN**: Introduces function `_make_validity_callback`, function `validity_check`, function `_get_heuristic_configs`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `layout_str`, `valid_configs`, `kernel`, and `key`.
- **CN**: 这里定义了函数`_make_validity_callback`、函数`validity_check`、函数`_get_heuristic_configs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `layout_str`、`valid_configs`、`kernel`、`key` 等值。

### Lines 241-260 / 第 241-260 行
````python
        self,
        m: int,
        n: int,
        k: int,
        dtype_a: torch.dtype,
        layout_a: str,
        layout_b: str,
        count: int,
        valid_configs: OrderedSet[ConfigKey],
        accumulator_type: torch.dtype = torch.float32,
        batch_size: int = 1,
        dtype_b: torch.dtype | None = None,
        out_dtype: torch.dtype | None = None,
    ) -> list[HeuristicConfig]:
        """
        Get kernel configurations recommended by nvMatmulHeuristics.

        Uses validity callback to filter to cutlass_api-compatible configs.
        """
        import nvMatmulHeuristics
````
- **EN**: Imports dependencies such as `nvMatmulHeuristics` for the logic in this range. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `m`, `n`, `k`, `dtype_a`, `layout_a`, `layout_b`, and `...+6`. This range continues the implementation of function `NVUniversalGemmHeuristics._get_heuristic_configs`.
- **CN**: 这里导入了 `nvMatmulHeuristics` 等依赖，为后续逻辑提供基础能力。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `m`、`n`、`k`、`dtype_a`、`layout_a`、`layout_b`、`另有6项` 等值。这一段延续了函数`NVUniversalGemmHeuristics._get_heuristic_configs` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python

        dtype_to_cublas = {
            torch.float64: "D",
            torch.float32: "S",
            torch.float16: "H",
            torch.bfloat16: "T",
            torch.float8_e4m3fn: "Q",
            torch.float8_e5m2: "R",
            torch.float4_e2m1fn_x2: "F4",
        }
        a_char = dtype_to_cublas.get(dtype_a, "H")
        b_char = dtype_to_cublas.get(dtype_b or dtype_a, a_char)
        out_char = dtype_to_cublas.get(out_dtype or dtype_a, a_char)
        acc_char = dtype_to_cublas.get(accumulator_type, "S")

        # nvMatmulHeuristics precision string formats:
        # - 3-letter {A}{B}{out}: used for standard GEMM and multi-char tokens (F4, BF)
        # - 5-letter {A}{B}{C}{compute}{D}: used for single-char FP8 types (Q, R, O)
        has_multichar = any(len(c) > 1 for c in (a_char, b_char, out_char))
        if has_multichar:
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dtype_to_cublas`, `a_char`, `b_char`, `out_char`, `acc_char`, and `has_multichar`. This range continues the implementation of function `NVUniversalGemmHeuristics._get_heuristic_configs`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `dtype_to_cublas`、`a_char`、`b_char`、`out_char`、`acc_char`、`has_multichar` 等值。这一段延续了函数`NVUniversalGemmHeuristics._get_heuristic_configs` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
            precision = f"{a_char}{b_char}{out_char}"
        elif a_char != b_char or a_char in ("Q", "R", "O"):
            precision = f"{a_char}{b_char}{out_char}{acc_char}{out_char}"
        else:
            precision = f"{a_char}{acc_char}{out_char}"

        # NvMatmulHeuristicsInterfaceEx configuration:
        # - backend=CUTLASS3: Use CUTLASS 3.x kernel database for Hopper+ GPUs
        #   TODO(nikhilap): Update when nvMatmulHeuristics supports CUTLASS 4
        # - flags=PERF_MODEL_BASED_AUTO_TUNING: Rank kernels using analytical
        #   performance model (faster than empirical profiling)
        # - load_discovery_implicitly=True: Auto-load kernel discovery sets on demand
        lh = nvMatmulHeuristics.NvMatmulHeuristicsInterfaceEx(
            backend=nvMatmulHeuristics.NvMatmulHeuristicsTarget.CUTLASS3,
            flags=nvMatmulHeuristics.NvMatmulHeuristicsFlags.PERF_MODEL_BASED_AUTO_TUNING,
            load_discovery_implicitly=True,
        )

        backend = lh.createBackend(nvMatmulHeuristics.NvMatmulHeuristicsTarget.CUTLASS3)

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `precision`, `else`, `lh`, `backend`, `flags`, and `load_discovery_implicitly`. This range continues the implementation of function `NVUniversalGemmHeuristics._get_heuristic_configs`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `precision`、`else`、`lh`、`backend`、`flags`、`load_discovery_implicitly` 等值。这一段延续了函数`NVUniversalGemmHeuristics._get_heuristic_configs` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
        validity_callback = self._make_validity_callback(valid_configs)
        lh.setBackendCallbackProperty(
            backend,
            nvMatmulHeuristics.NvMatmulHeuristicsBackendPropertyCallbackKind.KERNEL_ADDITIONAL_VALIDITY_CHECK,
            validity_callback,
        )

        layout = self._get_layout_enum(layout_a, layout_b)

        lh.loadInternalDiscoverySet(layout, precision=precision)

        problem = lh.makeNvMatmulHeuristicsProblem(
            m, n, k, layout, batch_size=batch_size
        )
        raw_configs = lh.getEx(problem, count, backend, precision=precision)
        lh.destroyBackend(backend)

        if not raw_configs:
            autotuning_log.debug(
                "nvMatmulHeuristics returned 0 configs for M=%d, N=%d, K=%d, "
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `validity_callback`, `layout`, `problem`, and `raw_configs`. This range continues the implementation of function `NVUniversalGemmHeuristics._get_heuristic_configs`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `validity_callback`、`layout`、`problem`、`raw_configs` 等值。这一段延续了函数`NVUniversalGemmHeuristics._get_heuristic_configs` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
                "dtype=%s, layout=(%s, %s), precision=%s",
                m,
                n,
                k,
                dtype_a,
                layout_a,
                layout_b,
                precision,
            )
            return []

        configs = []
        for cfg in raw_configs:
            kernel = cfg["kernel"]
            configs.append(
                HeuristicConfig(
                    tile_m=kernel.cta_tile_m,
                    tile_n=kernel.cta_tile_n,
                    tile_k=kernel.cta_tile_k,
                    cluster_m=kernel.cluster_m,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `configs`, `kernel`, `tile_m`, `tile_n`, `tile_k`, and `cluster_m`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `configs`、`kernel`、`tile_m`、`tile_n`、`tile_k`、`cluster_m` 等值。

### Lines 341-360 / 第 341-360 行
````python
                    cluster_n=kernel.cluster_n,
                    stages=kernel.stages,
                    split_k=kernel.split_k,
                    warp_tile_m=kernel.warp_tile_m,
                    warp_tile_n=kernel.warp_tile_n,
                    warp_tile_k=kernel.warp_tile_k,
                    estimated_runtime=cfg["runtime"],
                )
            )

        autotuning_log.info(
            "nvMatmulHeuristics for M=%d, N=%d, K=%d, dtype=%s, layout=(%s, %s), "
            "precision=%s: %d configs returned",
            m,
            n,
            k,
            dtype_a,
            layout_a,
            layout_b,
            precision,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `cluster_n`, `stages`, `split_k`, `warp_tile_m`, `warp_tile_n`, `warp_tile_k`, and `...+1`. This range continues the implementation of function `NVUniversalGemmHeuristics._get_heuristic_configs`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `cluster_n`、`stages`、`split_k`、`warp_tile_m`、`warp_tile_n`、`warp_tile_k`、`另有1项` 等值。这一段延续了函数`NVUniversalGemmHeuristics._get_heuristic_configs` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
            len(configs),
        )
        for i, cfg in enumerate(configs):
            runtime_us = cfg.estimated_runtime * 1e6
            autotuning_log.info(
                "  Config %d: tile=(%d, %d, %d), cluster=(%d, %d), "
                "stages=%d, split_k=%d, warp_tile=(%d, %d, %d), "
                "estimated_runtime=%.2f us",
                i,
                cfg.tile_m,
                cfg.tile_n,
                cfg.tile_k,
                cfg.cluster_m,
                cfg.cluster_n,
                cfg.stages,
                cfg.split_k,
                cfg.warp_tile_m,
                cfg.warp_tile_n,
                cfg.warp_tile_k,
                runtime_us,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `runtime_us`. This range continues the implementation of function `NVUniversalGemmHeuristics._get_heuristic_configs`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `runtime_us` 等值。这一段延续了函数`NVUniversalGemmHeuristics._get_heuristic_configs` 的具体实现。

### Lines 381-395 / 第 381-395 行
````python
            )

        return configs


# Singleton instance for use in add_nv_universal_gemm_choices
_nvgemm_heuristics: NVUniversalGemmHeuristics | None = None


def get_nvgemm_heuristics() -> NVUniversalGemmHeuristics:
    """Get the singleton NVUniversalGemmHeuristics instance."""
    global _nvgemm_heuristics
    if _nvgemm_heuristics is None:
        _nvgemm_heuristics = NVUniversalGemmHeuristics()
    return _nvgemm_heuristics
````
- **EN**: Introduces function `get_nvgemm_heuristics`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_nvgemm_heuristics`.
- **CN**: 这里定义了函数`get_nvgemm_heuristics`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_nvgemm_heuristics` 等值。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `HeuristicConfig`, and `NVUniversalGemmHeuristics`  
  **CN**: 主要类：`HeuristicConfig`、`NVUniversalGemmHeuristics`
- **EN**: Primary functions: `_make_config_key_from_heuristic`, `_make_config_key_from_kernel_design`, `_make_config_key_from_heuristics_kernel`, and `get_nvgemm_heuristics`  
  **CN**: 主要函数：`_make_config_key_from_heuristic`、`_make_config_key_from_kernel_design`、`_make_config_key_from_heuristics_kernel`、`get_nvgemm_heuristics`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `collections`, `dataclasses`, `typing`
- **Third-party / 第三方**: `nvMatmulHeuristics`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.utils`, `torch._logging`, `torch.utils._ordered_set`, `.gemm`, `..kernel_inputs`

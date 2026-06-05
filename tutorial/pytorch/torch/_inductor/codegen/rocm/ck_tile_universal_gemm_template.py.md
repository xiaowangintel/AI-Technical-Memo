# ck_tile_universal_gemm_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/rocm/ck_tile_universal_gemm_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CKTileGemmOperation`, and `CKTileGemmTemplate`. It exposes functions such as `is_static_int`, `torch_layout_to_ck_layout`, and `ops`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CKTileGemmOperation`、`CKTileGemmTemplate` 等类。同时提供 `is_static_int`、`torch_layout_to_ck_layout`、`ops` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs, disable-error-code="attr-defined, valid-type"
import functools
import logging
import random
from dataclasses import asdict, dataclass
from typing import Any

import torch
from torch._inductor import config
from torch._inductor.codegen.rocm.ck_tile_template import CKTileTemplate
from torch._inductor.codegen.rocm.rocm_kernel import ROCmTemplateKernel
from torch._inductor.codegen.rocm.rocm_template import ArgInfo
from torch._inductor.ir import Buffer, Layout
from torch.utils._ordered_set import OrderedSet

from ...utils import IndentedBuffer


log = logging.getLogger(__name__)


def is_static_int(number):
    import sympy

    return isinstance(number, (int, sympy.Integer))


def torch_layout_to_ck_layout(torch_layout):
````
- **EN**: Imports dependencies such as `functools`, `logging`, `random`, `dataclasses`, `typing`, `torch`, and `...+8` for the logic in this range. Introduces function `is_static_int`, function `torch_layout_to_ck_layout`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `functools`、`logging`、`random`、`dataclasses`、`typing`、`torch`、`另有8项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`is_static_int`、函数`torch_layout_to_ck_layout`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 29-56 / 第 29-56 行
````python
    if torch_layout.stride[-1] == 1:
        return "Row"
    elif torch_layout.stride[-2] == 1:
        return "Col"
    else:
        return None


@dataclass
class CKTileGemmOperation:
    layout_a: str
    layout_b: str
    layout_c: str

    datatype_a: str
    datatype_b: str
    datatype_c: str

    tile_m: int
    tile_n: int
    tile_k: int

    warp_m: int
    warp_n: int
    warp_k: int

    warp_tile_m: int
    warp_tile_n: int
````
- **EN**: Introduces class `CKTileGemmOperation`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`CKTileGemmOperation`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 57-84 / 第 57-84 行
````python
    warp_tile_k: int

    m_is_padded: str
    n_is_padded: str
    k_is_padded: str

    pipeline: str
    scheduler: str
    epilogue: str

    def layout_repr(self):
        return f"{self.layout_a[0]}{self.layout_b[0]}{self.layout_c[0]}"

    def dtype_repr(self):
        return f"{self.datatype_a}{self.datatype_b}{self.datatype_c}"

    def tile_sizes(self):
        return "_".join(
            [
                f"{self.tile_m}{self.tile_n}{self.tile_k}",
                f"{self.warp_m}{self.warp_n}{self.warp_k}",
                f"{self.warp_tile_m}{self.warp_tile_n}{self.warp_tile_k}",
            ]
        )

    def name(self):
        return "ck_tile_gemm_universal_" + "_".join(
            [
````
- **EN**: Introduces function `layout_repr`, function `dtype_repr`, function `tile_sizes`, function `name`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `warp_tile_k`, `m_is_padded`, `n_is_padded`, `k_is_padded`, `pipeline`, `scheduler`, and `...+1`.
- **CN**: 这里定义了函数`layout_repr`、函数`dtype_repr`、函数`tile_sizes`、函数`name`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `warp_tile_k`、`m_is_padded`、`n_is_padded`、`k_is_padded`、`pipeline`、`scheduler`、`另有1项` 等值。

### Lines 85-112 / 第 85-112 行
````python
                f"{self.layout_repr()}",
                f"{self.dtype_repr()}",
                f"{self.tile_sizes()}",
                f"{self.pipeline}",
                f"{self.scheduler}",
                f"{self.epilogue}",
            ]
        )

    def dict_items(self):
        return asdict(self).items()


@functools.cache
def ops():
    """
    Generate the supported instance dataclasses
    """
    import itertools

    compute_v3_instances = [
        CKTileGemmOperation(
            layout_a=layout_a,
            layout_b=layout_b,
            layout_c=layout_c,
            datatype_a=datatype_a,
            datatype_b=datatype_b,
            datatype_c=datatype_c,
````
- **EN**: Imports dependencies such as `itertools` for the logic in this range. Introduces function `dict_items`, function `ops`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `itertools` 等依赖，为后续逻辑提供基础能力。这里定义了函数`dict_items`、函数`ops`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 113-140 / 第 113-140 行
````python
            tile_m=tile_m,
            tile_n=tile_n,
            tile_k=tile_k,
            warp_m=warp_m,
            warp_n=warp_n,
            warp_k=warp_k,
            warp_tile_m=warp_tile_m,
            warp_tile_n=warp_tile_n,
            warp_tile_k=warp_tile_k,
            m_is_padded=m_is_padded,
            n_is_padded=n_is_padded,
            k_is_padded=k_is_padded,
            pipeline="CompV3",
            scheduler="Intrawave",
            epilogue=epilogue,
        )
        for (layout_a, layout_b, layout_c) in [
            ("Row", "Row", "Row"),
            ("Row", "Col", "Row"),
        ]
        for (datatype_a, datatype_b, datatype_c) in [("FP16",) * 3, ("BF16",) * 3]
        for (tile_m, tile_n, tile_k) in [(256, 256, 32), (256, 256, 64)]
        for (warp_m, warp_n, warp_k) in [(2, 2, 1)]
        for (warp_tile_m, warp_tile_n, warp_tile_k) in [(32, 32, 16)]
        for m_is_padded in ["true", "false"]
        for n_is_padded in ["true", "false"]
        for k_is_padded in ["true", "false"]
        for epilogue in ["Default", "CShuffle"]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tile_m`, `tile_n`, `tile_k`, `warp_m`, `warp_n`, `warp_k`, and `...+9`. This range continues the implementation of function `ops`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tile_m`、`tile_n`、`tile_k`、`warp_m`、`warp_n`、`warp_k`、`另有9项` 等值。这一段延续了函数`ops` 的具体实现。

### Lines 141-168 / 第 141-168 行
````python
    ]

    compute_v4_instances = [
        CKTileGemmOperation(
            layout_a=layout_a,
            layout_b=layout_b,
            layout_c=layout_c,
            datatype_a=datatype_a,
            datatype_b=datatype_b,
            datatype_c=datatype_c,
            tile_m=tile_m,
            tile_n=tile_n,
            tile_k=tile_k,
            warp_m=warp_m,
            warp_n=warp_n,
            warp_k=warp_k,
            warp_tile_m=warp_tile_m,
            warp_tile_n=warp_tile_n,
            warp_tile_k=warp_tile_k,
            m_is_padded=m_is_padded,
            n_is_padded=n_is_padded,
            k_is_padded=k_is_padded,
            pipeline="CompV4",
            scheduler="Intrawave",
            epilogue=epilogue,
        )
        for (layout_a, layout_b, layout_c) in [
            ("Row", "Row", "Row"),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `compute_v4_instances`, `layout_a`, `layout_b`, `layout_c`, `datatype_a`, `datatype_b`, and `...+16`. This range continues the implementation of function `ops`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `compute_v4_instances`、`layout_a`、`layout_b`、`layout_c`、`datatype_a`、`datatype_b`、`另有16项` 等值。这一段延续了函数`ops` 的具体实现。

### Lines 169-196 / 第 169-196 行
````python
            ("Row", "Col", "Row"),
        ]
        for (datatype_a, datatype_b, datatype_c) in [("FP16",) * 3, ("BF16",) * 3]
        for (tile_m, tile_n, tile_k) in [
            (256, 256, 32)
        ]  # half the tile size since it has double buffering
        for (warp_m, warp_n, warp_k) in [(2, 2, 1)]
        for (warp_tile_m, warp_tile_n, warp_tile_k) in [(32, 32, 16)]
        for m_is_padded in ["true", "false"]
        for n_is_padded in ["true", "false"]
        for k_is_padded in ["true", "false"]
        for epilogue in ["Default", "CShuffle"]
    ]

    mem_instances = [
        CKTileGemmOperation(
            layout_a=layout_a,
            layout_b=layout_b,
            layout_c=layout_c,
            datatype_a=datatype_a,
            datatype_b=datatype_b,
            datatype_c=datatype_c,
            tile_m=tile_m,
            tile_n=tile_n,
            tile_k=tile_k,
            warp_m=warp_m,
            warp_n=warp_n,
            warp_k=warp_k,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mem_instances`, `layout_a`, `layout_b`, `layout_c`, `datatype_a`, `datatype_b`, and `...+7`. This range continues the implementation of function `ops`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `mem_instances`、`layout_a`、`layout_b`、`layout_c`、`datatype_a`、`datatype_b`、`另有7项` 等值。这一段延续了函数`ops` 的具体实现。

### Lines 197-224 / 第 197-224 行
````python
            warp_tile_m=warp_tile_m,
            warp_tile_n=warp_tile_n,
            warp_tile_k=warp_tile_k,
            m_is_padded=m_is_padded,
            n_is_padded=n_is_padded,
            k_is_padded=k_is_padded,
            pipeline="Mem",
            scheduler=scheduler,
            epilogue=epilogue,
        )
        for (layout_a, layout_b, layout_c) in [
            ("Row", "Row", "Row"),
            ("Row", "Col", "Row"),
        ]
        for (datatype_a, datatype_b, datatype_c) in [("FP16",) * 3, ("BF16",) * 3]
        for (tile_m, tile_n, tile_k) in [(256, 256, 32), (256, 256, 64)]
        for (warp_m, warp_n, warp_k) in [(2, 2, 1)]
        for (warp_tile_m, warp_tile_n, warp_tile_k) in [(32, 32, 16)]
        for m_is_padded in ["true", "false"]
        for n_is_padded in ["true", "false"]
        for k_is_padded in ["true", "false"]
        for scheduler in ["Intrawave", "Interwave"]
        for epilogue in ["Default", "CShuffle"]
    ]

    return list(
        itertools.chain(compute_v3_instances, compute_v4_instances, mem_instances)
    )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `warp_tile_m`, `warp_tile_n`, `warp_tile_k`, `m_is_padded`, `n_is_padded`, `k_is_padded`, and `...+3`. This range continues the implementation of function `ops`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `warp_tile_m`、`warp_tile_n`、`warp_tile_k`、`m_is_padded`、`n_is_padded`、`k_is_padded`、`另有3项` 等值。这一段延续了函数`ops` 的具体实现。

### Lines 225-252 / 第 225-252 行
````python


class CKTileGemmTemplate(CKTileTemplate):
    """
    This class is used for rendering CK-Tile Universal GEMM kernels
    """

    gemm_template = r"""{{version_comment}}
    {{headers}}
    {{globals}}
    {{instance_definition}}
    extern "C" {
    PT_EXPORT {{kernel_definition}} {

        using {{instance_namespace}}::BaseGemmPipeline;
        using {{instance_namespace}}::TilePartitioner;

        constexpr auto TileK = {{instance_namespace}}::TileK;
        constexpr auto kPrefetchStages = BaseGemmPipeline::PrefetchStages;

        const auto BiasTerms = std::array<const void*, 0> ();
        const auto BiasStrides = std::array<int32_t, 0> ();

        auto kargs = ck_tile::UniversalGemmKernelArgs<> {
           {X},
           {W},
           BiasTerms,
           Y,
````
- **EN**: Introduces class `CKTileGemmTemplate`. Initializes or updates values such as `gemm_template`.
- **CN**: 这里定义了类`CKTileGemmTemplate`。初始化或更新了 `gemm_template` 等值。

### Lines 253-280 / 第 253-280 行
````python
           M,
           N,
           K,
           {LDA},
           {LDB},
           BiasStrides,
           LDC,
           kBatch
        };

        if (workspace_size) {
            *workspace_size = 0;
            return 0;
        }

        // run the kernel
        const auto dispatch = [&](const auto has_hot_loop_, const auto tail_number_) constexpr {
            using Kernel = {{instance_namespace}}::Kernel<has_hot_loop_.value, tail_number_.value>;

            if (!Kernel::IsSupportedArgument(kargs)) {
                // we do our best to statically avoid this case in `filter_op`
                throw std::runtime_error("invalid argument");
            }
            auto stream_config = ck_tile::stream_config{stream};
            auto grid_size = Kernel::GridSize(M, N, kBatch);
            constexpr auto block_size = Kernel::BlockSize();
            constexpr auto lds_bytes = 0;
            constexpr auto kBlockPerCU = 1;
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of class `CKTileGemmTemplate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了类`CKTileGemmTemplate` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
            auto gemm = ck_tile::make_kernel<block_size.x, kBlockPerCU>(Kernel{}, grid_size, block_size, lds_bytes, kargs);
            float elapsed_time = ck_tile::launch_kernel(stream_config, gemm);
        };

        const ck_tile::index_t k_grain     = kBatch * TileK;
        const ck_tile::index_t K_split     = (K + k_grain - 1) / k_grain * TileK;
        const ck_tile::index_t num_loop    = TilePartitioner::GetLoopNum(K_split);
        const bool has_hot_loop            = BaseGemmPipeline::BlockHasHotloop(num_loop);
        const ck_tile::TailNumber tail_num = BaseGemmPipeline::GetBlockLoopTailNum(num_loop);

        {{rendered_dispatch}}

        return 0;
    } // kernel definition
    } // extern C
    """

    def __init__(
        self,
        input_nodes: list[Buffer],
        layout: Layout,
    ) -> None:
        super().__init__(
            "ck_tile_gemm_template",
            input_nodes=input_nodes,
            layout=layout,
        )

````
- **EN**: Introduces function `__init__`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `input_nodes`, and `layout`.
- **CN**: 这里定义了函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `input_nodes`、`layout` 等值。

### Lines 309-336 / 第 309-336 行
````python
    def header(self) -> IndentedBuffer:
        res = super().header()
        res.splice(
            """
                // CK GEMM header(s)

                #include "ck_tile/ops/gemm.hpp"
                #include "ck_tile/ops/epilogue.hpp"
            """
        )
        return res

    def globals(self) -> IndentedBuffer:
        res = super().globals()
        res.splice(
            """
                // CK GEMM globals

                using Row = ck_tile::tensor_layout::gemm::RowMajor;
                using Col = ck_tile::tensor_layout::gemm::ColumnMajor;

                template <ck_tile::index_t PrefetchStages, typename Dispatcher>
                void dispatch_memory_pipeline_hot_loop(const ck_tile::TailNumber tail_num, Dispatcher dispatch)
                {
                    if(tail_num == ck_tile::TailNumber::One)
                    {
                        dispatch(ck_tile::bool_constant<true>{},
                            ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::One>{});
````
- **EN**: Introduces function `header`, function `globals`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `res`, and `ck_tile`.
- **CN**: 这里定义了函数`header`、函数`globals`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `res`、`ck_tile` 等值。

### Lines 337-364 / 第 337-364 行
````python
                    }
                    else if(tail_num == ck_tile::TailNumber::Full)
                    {
                        dispatch(ck_tile::bool_constant<true>{},
                            ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Full>{});
                    }

                    if constexpr(PrefetchStages > 2)
                    {
                        if(tail_num == ck_tile::TailNumber::Two)
                        {
                            dispatch(ck_tile::bool_constant<true>{},
                                ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Two>{});
                        }
                    }
                    if constexpr(PrefetchStages > 3)
                    {
                        if(tail_num == ck_tile::TailNumber::Three)
                        {
                            dispatch(ck_tile::bool_constant<true>{},
                                ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Three>{});
                        }
                    }
                    if constexpr(PrefetchStages > 4)
                    {
                        if(tail_num == ck_tile::TailNumber::Four)
                        {
                            dispatch(ck_tile::bool_constant<true>{},
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ck_tile`. This range continues the implementation of function `CKTileGemmTemplate.globals`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `ck_tile` 等值。这一段延续了函数`CKTileGemmTemplate.globals` 的具体实现。

### Lines 365-392 / 第 365-392 行
````python
                                ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Four>{});
                        }
                    }
                    if constexpr(PrefetchStages > 5)
                    {
                        if(tail_num == ck_tile::TailNumber::Five)
                        {
                            dispatch(ck_tile::bool_constant<true>{},
                                ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Five>{});
                        }
                    }
                    if constexpr(PrefetchStages > 6)
                    {
                        if(tail_num == ck_tile::TailNumber::Six)
                        {
                            dispatch(ck_tile::bool_constant<true>{},
                                ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Six>{});
                        }
                    }
                    if constexpr(PrefetchStages > 7)
                    {
                        if(tail_num == ck_tile::TailNumber::Seven)
                        {
                            dispatch(ck_tile::bool_constant<true>{},
                                ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::Seven>{});
                        }
                    }
                }
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ck_tile`. This range continues the implementation of function `CKTileGemmTemplate.globals`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `ck_tile` 等值。这一段延续了函数`CKTileGemmTemplate.globals` 的具体实现。

### Lines 393-420 / 第 393-420 行
````python
            """
        )
        return res

    def check_dtypes(self, op: "CKTileGemmOperation"):
        X_dtype, W_dtype, out_dtype = [
            T.get_layout().dtype for T in [*self.input_nodes, self.output_node]
        ]
        if op.datatype_a != self._TORCH_DTYPE_TO_CK[X_dtype]:
            return False
        if op.datatype_b != self._TORCH_DTYPE_TO_CK[W_dtype]:
            return False
        if op.datatype_c != self._TORCH_DTYPE_TO_CK[out_dtype]:
            return False
        return True

    def check_layouts(self, op: "CKTileGemmOperation"):
        X_layout, W_layout, out_layout = [
            torch_layout_to_ck_layout(T.get_layout())
            for T in [*self.input_nodes, self.output_node]
        ]
        if op.layout_a != X_layout:
            return False
        if op.layout_b != W_layout:
            return False
        if op.layout_c != out_layout:
            return False
        return True
````
- **EN**: Introduces function `check_dtypes`, function `check_layouts`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`check_dtypes`、函数`check_layouts`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 421-448 / 第 421-448 行
````python

    def get_gemm_problem_size(self):
        X_size, W_size = [T.get_layout().size for T in [*self.input_nodes]]

        M, K = X_size
        _, N = W_size

        return M, N, K

    def check_block_tiles(self, op: "CKTileGemmOperation"):
        """
        The contiguous dimension of a tensor must be divisible by the block tile size
        This helper function enforces it for the inputs and the output.
        """
        M, N, K = self.get_gemm_problem_size()

        def check(dim_size, tile_size, is_padded):
            if (
                is_static_int(dim_size)
                and dim_size % tile_size != 0
                and is_padded == "false"
            ):
                return False
            return True

        if op.layout_a == "Row":
            # handle in kBatch check
            return True
````
- **EN**: Introduces function `get_gemm_problem_size`, function `check_block_tiles`, function `check`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_gemm_problem_size`、函数`check_block_tiles`、函数`check`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 449-476 / 第 449-476 行
````python
        elif op.layout_a == "Col":
            if not check(M, op.tile_m, op.m_is_padded):
                return False
        else:
            raise AssertionError(f"Invalid layout {op.layout_a=}")

        if op.layout_b == "Row":
            if not check(N, op.tile_n, op.n_is_padded):
                return False
        elif op.layout_b == "Col":
            # handle in kBatch check
            return True
        else:
            raise AssertionError(f"Invalid {op.layout_b=}")

        if op.layout_c == "Row":
            if not check(N, op.tile_n, op.n_is_padded):
                return False
        elif op.layout_c == "Col":
            if not check(M, op.tile_m, op.m_is_padded):
                return False
        else:
            raise AssertionError(f"Invalid layout {op.layout_c=}")

        return True

    def check_alignments(self, op: "CKTileGemmOperation"):
        """
````
- **EN**: Introduces function `check_alignments`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`check_alignments`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 477-504 / 第 477-504 行
````python
        The contiguous dimension of a tensor must be divisible by the vector load size.
        """
        M, N, K = self.get_gemm_problem_size()

        def max_alignment(contiguous_elements_per_tile, elements_per_thread, ck_dtype):
            for vector_load_bytes in (16, 8, 4, 2, 1):
                alignment = vector_load_bytes // self.ck_dtype_to_size[ck_dtype]
                if (
                    alignment > 0
                    and contiguous_elements_per_tile % alignment == 0
                    and elements_per_thread % alignment == 0
                ):
                    return alignment

        threads_per_block = (
            op.warp_m * op.warp_n * op.warp_k * self.gfx9_threads_per_warp
        )
        a_elements_per_thread = op.tile_m * op.tile_k / threads_per_block
        b_elements_per_thread = op.tile_n * op.tile_k / threads_per_block

        if op.layout_a == "Row":
            # K is contiguous tensor dimension
            a_max_vector_size = max_alignment(
                op.tile_k, a_elements_per_thread, op.datatype_a
            )
            if is_static_int(K) and K % a_max_vector_size != 0:
                return False
        elif op.layout_a == "Col":
````
- **EN**: Introduces function `max_alignment`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `alignment`, `threads_per_block`, `a_elements_per_thread`, `b_elements_per_thread`, and `a_max_vector_size`.
- **CN**: 这里定义了函数`max_alignment`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `alignment`、`threads_per_block`、`a_elements_per_thread`、`b_elements_per_thread`、`a_max_vector_size` 等值。

### Lines 505-532 / 第 505-532 行
````python
            # M is contiguous tensor dimension
            a_max_vector_size = max_alignment(
                op.tile_m, a_elements_per_thread, op.datatype_a
            )
            if is_static_int(M) and M % a_max_vector_size != 0:
                return False
        else:
            raise AssertionError(f"Invalid layout {op.layout_a=}")

        if op.layout_b == "Row":
            # N is contiguous tensor dimension
            b_max_vector_size = max_alignment(
                op.tile_n, b_elements_per_thread, op.datatype_b
            )
            if is_static_int(N) and N % b_max_vector_size != 0:
                return False
        elif op.layout_b == "Col":
            # K is contiguous tensor dimension
            b_max_vector_size = max_alignment(
                op.tile_k, b_elements_per_thread, op.datatype_b
            )
            if is_static_int(K) and K % b_max_vector_size != 0:
                return False
        else:
            raise AssertionError(f"Invalid layout {op.layout_b=}")

        # the `default` epilogue writes C to memory by 1 tensor element
        # (divisibility check not necessary)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `a_max_vector_size`, `else`, and `b_max_vector_size`. This range continues the implementation of function `CKTileGemmTemplate.check_alignments`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `a_max_vector_size`、`else`、`b_max_vector_size` 等值。这一段延续了函数`CKTileGemmTemplate.check_alignments` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python
        # the `cshuffle` epilogue writes C to memory by 16 bytes
        # (so the contiguous C dimension size must be divisible by the number of tensor elements in 16 bytes)
        if op.epilogue == "CShuffle":
            if (
                op.layout_c == "Row"
                and is_static_int(N)
                and N % (16 / self.ck_dtype_to_size[op.datatype_c]) != 0
            ):
                return False

        return True

    def check_warp_tiles(self, op: "CKTileGemmOperation"):
        if op.tile_m % (op.warp_m * op.warp_tile_m) != 0:
            return False
        if op.tile_n % (op.warp_n * op.warp_tile_n) != 0:
            return False
        if op.tile_k % (op.warp_k * op.warp_tile_k) != 0:
            return False
        return True

    def check_block_tile_size(self, op: "CKTileGemmOperation"):
        # assuming LDS size is 64KB
        if op.pipeline == "CompV4":
            max_block_tile_size = 2**15
        else:
            max_block_tile_size = 2**16

````
- **EN**: Introduces function `check_warp_tiles`, function `check_block_tile_size`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max_block_tile_size`, and `else`.
- **CN**: 这里定义了函数`check_warp_tiles`、函数`check_block_tile_size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `max_block_tile_size`、`else` 等值。

### Lines 561-588 / 第 561-588 行
````python
        block_tile_size = (
            self.ck_dtype_to_size[op.datatype_a] * op.tile_m * op.tile_k
            + self.ck_dtype_to_size[op.datatype_b] * op.tile_n * op.tile_k
        )
        if block_tile_size > max_block_tile_size:
            return False
        return True

    def filter_op(self, op: "CKTileGemmOperation"):
        """
        Determines whether a given op definition is suitable for the current
        input / output of the operation that this template implements.

        Filter is based on inputs' dtype, layout and statically inferred size.

        Returns None if the op is not suitable, otherwise returns the op to be used.
        """
        if not self.check_dtypes(op):
            return None
        if not self.check_layouts(op):
            return None
        if not self.check_block_tiles(op):
            return None
        if not self.check_alignments(op):
            return None

        return op

````
- **EN**: Introduces function `filter_op`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`filter_op`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 589-616 / 第 589-616 行
````python
    def emit_ck_instance(self, op: "CKTileGemmOperation"):
        """
        This method is used to generate code which defines the type alias for the generated kernel class
        """
        template_definition = r"""
    // Gemm operator {{operation_name}}

    namespace {{operation_name}} {
         // block tile
        constexpr int32_t TileM = {{tile_m}};
        constexpr int32_t TileN = {{tile_n}};
        constexpr int32_t TileK = {{tile_k}};
        // warps per block
        constexpr int32_t WarpM = {{warp_m}};
        constexpr int32_t WarpN = {{warp_n}};
        constexpr int32_t WarpK = {{warp_k}};
        // xdl tile
        constexpr int32_t WarpTileM = {{warp_tile_m}};
        constexpr int32_t WarpTileN = {{warp_tile_n}};
        constexpr int32_t WarpTileK = {{warp_tile_k}};

        constexpr bool kPadM = {{m_is_padded}};
        constexpr bool kPadN = {{n_is_padded}};
        constexpr bool kPadK = {{k_is_padded}};

        using ALayout = {{layout_a}};
        using BLayout = {{layout_b}};
        using CLayout = {{layout_c}};
````
- **EN**: Introduces function `emit_ck_instance`. Initializes or updates values such as `template_definition`.
- **CN**: 这里定义了函数`emit_ck_instance`。初始化或更新了 `template_definition` 等值。

### Lines 617-644 / 第 617-644 行
````python

        using ADataType = {{datatype_a}};
        using BDataType = {{datatype_b}};
        using CDataType = {{datatype_c}};
        using AccDataType = F32;

        constexpr bool permuteA = false;
        constexpr bool permuteB = false;
        constexpr bool DoubleSmemBuffer = {{has_double_smem_buffer}};
        constexpr bool TransposeC = false;

        constexpr int kBlockPerCu                         = 1;
        constexpr ck_tile::index_t TilePartitionerGroupNum = 8;
        constexpr ck_tile::index_t TilePartitionerM01      = 4;

        using GemmShape =
            ck_tile::TileGemmShape<ck_tile::sequence<TileM, TileN, TileK>,
                                   ck_tile::sequence<WarpM, WarpN, WarpK>,
                                   ck_tile::sequence<WarpTileM, WarpTileN, WarpTileK>,
                                   permuteA,
                                   permuteB>;

        using TilePartitioner =
            ck_tile::GemmSpatiallyLocalTilePartitioner<GemmShape,
                                                       TilePartitionerGroupNum,
                                                       TilePartitionerM01>;

        using Traits  =
````
- **EN**: Initializes or updates values such as `ck_tile`. This range continues the implementation of function `CKTileGemmTemplate.emit_ck_instance`.
- **CN**: 初始化或更新了 `ck_tile` 等值。这一段延续了函数`CKTileGemmTemplate.emit_ck_instance` 的具体实现。

### Lines 645-672 / 第 645-672 行
````python
            ck_tile::TileGemmTraits<kPadM, kPadN, kPadK, ALayout, BLayout, CLayout>;

        using GemmUniversalTraits =
            ck_tile::TileGemmUniversalTraits<kPadM, kPadN, kPadK, DoubleSmemBuffer,
                                             ALayout, BLayout, CLayout, TransposeC>;

        using GemmPipelineProblem =
            ck_tile::GemmPipelineProblem<ADataType, BDataType, AccDataType, GemmShape, Traits>;

        {{rendered_scheduler}}

        template<bool has_hot_loop_v, ck_tile::TailNumber tail_number_v>
        using UniversalGemmProblem =
            ck_tile::UniversalGemmPipelineProblem<ADataType,
                                                  BDataType,
                                                  AccDataType,
                                                  GemmShape,
                                                  GemmUniversalTraits,
                                                  scheduler,
                                                  has_hot_loop_v,
                                                  tail_number_v>;

        {{rendered_pipeline}}

        {{rendered_epilogue}}

        template<bool has_hot_loop_v, ck_tile::TailNumber tail_number_v>
        using Kernel = ck_tile::GemmKernel<TilePartitioner, GemmPipeline<has_hot_loop_v, tail_number_v>, GemmEpilogue>;
````
- **EN**: Initializes or updates values such as `ck_tile`. This range continues the implementation of function `CKTileGemmTemplate.emit_ck_instance`.
- **CN**: 初始化或更新了 `ck_tile` 等值。这一段延续了函数`CKTileGemmTemplate.emit_ck_instance` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
    }

"""

        def render_epilogue(epilogue_type):
            if epilogue_type == "Default":
                return r"""
            using EpilogueProblem = ck_tile::DefaultGemm2DEpilogueProblem<ADataType,
                                                                          BDataType,
                                                                          AccDataType,
                                                                          CDataType,
                                                                          CLayout,
                                                                          kPadM,
                                                                          kPadN,
                                                                          WarpTileM,
                                                                          WarpTileN,
                                                                          WarpTileK,
                                                                          TransposeC>;
            using GemmEpilogue = ck_tile::DefaultGemm2DEpilogue<EpilogueProblem>;
        """
            elif epilogue_type == "CShuffle":
                return r"""
            constexpr auto kMemoryOperation = ck_tile::memory_operation_enum::set;
            using DsDataType = ck_tile::tuple<>; // no bias terms for vanilla GEMM
            using DsLayout = ck_tile::tuple<>;
            constexpr auto ELayout = CLayout;
            using CDEElementWise = ck_tile::element_wise::PassThrough; // no-op
            using EpilogueProblem = ck_tile::CShuffleEpilogueProblem<ADataType,
````
- **EN**: Introduces function `render_epilogue`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`render_epilogue`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 701-728 / 第 701-728 行
````python
                                                                     BDataType,
                                                                     DsDataType,
                                                                     AccDataType,
                                                                     CDataType,
                                                                     DsLayout,
                                                                     ELayout,
                                                                     CDEElementWise,
                                                                     GemmPipelineProblem::kBlockSize,
                                                                     TileM,
                                                                     TileN,
                                                                     WarpM,
                                                                     WarpN,
                                                                     WarpTileM,
                                                                     WarpTileN,
                                                                     WarpTileK,
                                                                     TransposeC,
                                                                     kMemoryOperation>;

            using GemmEpilogue = ck_tile::CShuffleEpilogue<EpilogueProblem>;
        """
            else:
                raise AssertionError("Epilogue must be set")

        def render_pipeline(pipeline_type):
            return rf"""
            using BaseGemmPipeline = ck_tile::BaseGemmPipelineAgBgCr{pipeline_type}<GemmPipelineProblem>;

            template<bool has_hot_loop_v, ck_tile::TailNumber tail_number_v>
````
- **EN**: Introduces function `render_pipeline`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `GemmPipelineProblem`, and `else`.
- **CN**: 这里定义了函数`render_pipeline`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `GemmPipelineProblem`、`else` 等值。

### Lines 729-756 / 第 729-756 行
````python
            using GemmPipeline = ck_tile::GemmPipelineAgBgCr{pipeline_type}<UniversalGemmProblem<has_hot_loop_v, tail_number_v>>;
        """

        def render_scheduler(scheduler_type):
            return rf"""
            constexpr auto scheduler = ck_tile::GemmPipelineScheduler::{scheduler_type};
        """

        rendered_definition = self._template_from_string(template_definition).render(
            operation_name=op.name(),
            **asdict(op),
            rendered_scheduler=render_scheduler(op.scheduler),
            rendered_pipeline=render_pipeline(op.pipeline),
            rendered_epilogue=render_epilogue(op.epilogue),
            has_double_smem_buffer=("true" if op.pipeline == "CompV4" else "false"),
        )
        return rendered_definition

    def render(  # type: ignore[override]
        self, kernel: ROCmTemplateKernel, op: "CKTileGemmOperation", **kwargs
    ) -> str:
        """
        The primary entry point for the code rendering process used in this template.
        """
        epilogue_nodes = kwargs.get("epilogue_nodes")
        assert epilogue_nodes is None or 0 == len(epilogue_nodes)
        template_buffer_node = kwargs.get("template_buffer_node")
        if template_buffer_node is not None:
````
- **EN**: Introduces function `render_scheduler`, function `render`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `rendered_definition`, `operation_name`, `rendered_scheduler`, `rendered_pipeline`, `rendered_epilogue`, `has_double_smem_buffer`, and `...+2`.
- **CN**: 这里定义了函数`render_scheduler`、函数`render`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `rendered_definition`、`operation_name`、`rendered_scheduler`、`rendered_pipeline`、`rendered_epilogue`、`has_double_smem_buffer`、`另有2项` 等值。

### Lines 757-784 / 第 757-784 行
````python
            self.output_node = template_buffer_node
        assert 2 == len(self.input_nodes)
        X, W = self.input_nodes
        Y = self.output_node

        instance_definition = self.emit_ck_instance(op)

        version_comment = rf"""/**
* Generated code for CK inductor backend
* See {type(self).__module__}.{type(self).__qualname__}
*
* Template instance {op}
*
* {torch.__version__=}
* torch.version.git_version={getattr(torch.version, "git_version", "None")}
*/
"""

        def render_dispatch(pipeline_type, op_name):
            switch_tailnum_template = r"""
            switch (tail_num) {
                {% for tail_num in valid_tailnums %}
                case ck_tile::TailNumber::{{tail_num}}:
                    dispatch({{has_hot_loop}},
                             ck_tile::integral_constant<ck_tile::TailNumber, ck_tile::TailNumber::{{tail_num}}>{});
                    break;
                {% endfor %}
                default:
````
- **EN**: Introduces function `render_dispatch`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Y`, `instance_definition`, `version_comment`, `switch_tailnum_template`, `ck_tile`, and `default`.
- **CN**: 这里定义了函数`render_dispatch`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Y`、`instance_definition`、`version_comment`、`switch_tailnum_template`、`ck_tile`、`default` 等值。

### Lines 785-812 / 第 785-812 行
````python
                    std::ostringstream err;
                    err << "Unsupported dispatch: "
                        << "Pipeline: " << "{{pipeline}}"
                        << "Prefetch stages: " << kPrefetchStages
                        << "Tail num: " << tail_num;
                    throw std::runtime_error(err.str());
            } // switch tail_num
            """
            dispatch_template = r"""
        if (has_hot_loop) {
            {{rendered_with_hot_loop}}
        }
        else { // has_hot_loop == false
            {{rendered_without_hot_loop}}
        } // if has_hot_loop
        """
            if pipeline_type == "CompV3":
                return self._template_from_string(dispatch_template).render(
                    rendered_with_hot_loop=self._template_from_string(
                        switch_tailnum_template
                    ).render(
                        has_hot_loop="ck_tile::integral_constant<bool, true>{}",
                        valid_tailnums=("Full", "Odd", "Even"),
                        pipeline=pipeline_type,
                    ),
                    rendered_without_hot_loop=self._template_from_string(
                        switch_tailnum_template
                    ).render(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`, `dispatch_template`, `rendered_with_hot_loop`, `has_hot_loop`, `valid_tailnums`, `pipeline`, and `...+1`. This range continues the implementation of function `CKTileGemmTemplate.render.render_dispatch`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `std`、`dispatch_template`、`rendered_with_hot_loop`、`has_hot_loop`、`valid_tailnums`、`pipeline`、`另有1项` 等值。这一段延续了函数`CKTileGemmTemplate.render.render_dispatch` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
                        has_hot_loop="ck_tile::integral_constant<bool, false>{}",
                        valid_tailnums=("Full", "Odd", "Even"),
                        pipeline=pipeline_type,
                    ),
                )
            elif pipeline_type == "Mem":
                return self._template_from_string(dispatch_template).render(
                    rendered_with_hot_loop="dispatch_memory_pipeline_hot_loop<kPrefetchStages>(tail_num, dispatch);",
                    rendered_without_hot_loop=self._template_from_string(
                        switch_tailnum_template
                    ).render(
                        has_hot_loop="ck_tile::integral_constant<bool, false>{}",
                        valid_tailnums=("Full", "Odd", "Even"),
                        pipeline=pipeline_type,
                    ),
                )
            elif pipeline_type == "CompV4":
                return self._template_from_string(dispatch_template).render(
                    rendered_with_hot_loop=self._template_from_string(
                        switch_tailnum_template
                    ).render(
                        has_hot_loop="ck_tile::integral_constant<bool, true>{}",
                        valid_tailnums=("Two", "Three"),
                        pipeline=pipeline_type,
                    ),
                    rendered_without_hot_loop=self._template_from_string(
                        switch_tailnum_template
                    ).render(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_hot_loop`, `valid_tailnums`, `pipeline`, `rendered_with_hot_loop`, and `rendered_without_hot_loop`. This range continues the implementation of function `CKTileGemmTemplate.render.render_dispatch`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `has_hot_loop`、`valid_tailnums`、`pipeline`、`rendered_with_hot_loop`、`rendered_without_hot_loop` 等值。这一段延续了函数`CKTileGemmTemplate.render.render_dispatch` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
                        has_hot_loop="ck_tile::integral_constant<bool, false>{}",
                        valid_tailnums=("Full", "Odd", "Even"),
                        pipeline=pipeline_type,
                    ),
                )
            else:
                raise AssertionError(f"Pipeline {pipeline_type} is not supported")

        return self._template_from_string(self.gemm_template).render(
            headers=self.header().getvalue(),
            globals=self.globals().getvalue(),
            instance_definition=instance_definition,
            kernel_definition=kernel.def_kernel(
                inputs=[X, W],  # type: ignore[list-item]
                outputs=[Y],
                names_str="X, W, Y",
                size_args=[
                    f"int32_t {arg}" for arg in ["M", "N", "K", "LDA", "LDB", "LDC"]
                ],
            ),
            instance_namespace=op.name(),
            version_comment=version_comment,
            rendered_dispatch=render_dispatch(op.pipeline, op.name()),
        )

    def gen_ops(self):
        """
        Creates a list of `CKTileGemmOperation` instances that match the GEMM operation this template represents.
````
- **EN**: Introduces function `gen_ops`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_hot_loop`, `valid_tailnums`, `pipeline`, `else`, `headers`, `globals`, and `...+9`.
- **CN**: 这里定义了函数`gen_ops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `has_hot_loop`、`valid_tailnums`、`pipeline`、`else`、`headers`、`globals`、`另有9项` 等值。

### Lines 869-896 / 第 869-896 行
````python
        The instances are guaranteed to have the correct layout, dtype and dimension padding for the GEMM input arguments.

        An instance may invalidate the GEMM configuration at runtime.
        Such instances will be assigned +inf runtime by the autotune process.
        """
        instances = ops()
        if not instances:
            raise AssertionError(
                "No Composable Kernel Universal GEMM instances found. "
                "Please check if the library is installed."
            )
        filtered_instances = list(filter(self.filter_op, instances))
        # NB: when using a fixed list order, most likely we will pick the subset of instances
        # which are very similar to each other. Randomizing the choice seems to solve this.
        random.seed(-11)
        chosen_instances = (
            random.sample(
                filtered_instances,
                min(len(filtered_instances), config.rocm.ck_tile_max_profiling_configs),
            )
            if config.rocm.ck_tile_max_profiling_configs
            else filtered_instances
        )
        log.debug(
            "generated %d ck instances after sample: %s",
            len(chosen_instances),
            chosen_instances,
        )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `instances`, `filtered_instances`, and `chosen_instances`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `instances`、`filtered_instances`、`chosen_instances` 等值。

### Lines 897-924 / 第 897-924 行
````python
        return chosen_instances

    @staticmethod
    def add_choices(
        choices,
        layout,
        input_nodes,
    ):
        """
        Add Composable Kernel Universal GEMM instance choices to the auto-tuning list.
        """
        template = CKTileGemmTemplate(
            input_nodes,
            layout,
        )
        ops = template.gen_ops()
        for op in ops:
            for k_batch in template.k_batch_choices(op):
                template.maybe_append_choice(
                    choices,
                    op=op,
                    kBatch=k_batch,
                )

    def k_batch_choices(self, op: "CKTileGemmOperation") -> tuple[int, ...]:
        """
        Returns a list of k_batch choices for the template.
        """
````
- **EN**: Introduces function `add_choices`, function `k_batch_choices`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`add_choices`、函数`k_batch_choices`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 925-952 / 第 925-952 行
````python
        default_choices = (1, 2, 4, 8, 16, 32)

        def check(dim_size, tile_size, is_padded):
            if (
                is_static_int(dim_size)
                and dim_size % tile_size != 0
                and is_padded == "false"
            ):
                return False
            return True

        _, _, K, _, _, _ = self.size_args()
        if op.layout_a == "Row" or op.layout_b == "Col":
            choices = tuple(
                filter(
                    lambda k_batch: check(K, op.tile_k * k_batch, op.k_is_padded),
                    default_choices,
                )
            )
        else:
            choices = default_choices

        if op.epilogue == "Default":
            choices = (1,)

        return choices

    def size_args(self):
````
- **EN**: Introduces function `check`, function `size_args`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `default_choices`, `choices`, and `else`.
- **CN**: 这里定义了函数`check`、函数`size_args`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `default_choices`、`choices`、`else` 等值。

### Lines 953-979 / 第 953-979 行
````python
        """
        Sizes and strides to be used for the kernel call
        """
        X = self.input_nodes[0]
        W = self.input_nodes[1]
        Y = self.output_node

        M = X.get_size()[0]
        K = X.get_size()[1]
        N = W.get_size()[1]
        LDA = X.get_stride()[0 if X.get_stride()[1] == 1 else 1]
        LDB = W.get_stride()[0 if W.get_stride()[1] == 1 else 1]
        LDC = Y.get_stride()[0 if Y.get_stride()[1] == 1 else 1]

        return M, N, K, LDA, LDB, LDC

    def get_runtime_arg_info(self) -> list[ArgInfo]:
        return [ArgInfo("kBatch", "int32_t")]

    def get_runtime_arg_values(self, **kwargs: Any) -> list[Any]:
        # maybe_append_choice kwarg for k_batch must match the name of the argument
        arg_names = OrderedSet([arg.name for arg in self.get_runtime_arg_info()])
        if not arg_names.issubset(kwargs):
            raise ValueError(
                "Missing runtime arguments: " + ", ".join(arg_names - kwargs.keys())
            )
        return [kwargs[k] for k in arg_names]
````
- **EN**: Introduces function `get_runtime_arg_info`, function `get_runtime_arg_values`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `X`, `W`, `Y`, `M`, `K`, `N`, and `...+4`.
- **CN**: 这里定义了函数`get_runtime_arg_info`、函数`get_runtime_arg_values`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `X`、`W`、`Y`、`M`、`K`、`N`、`另有4项` 等值。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `logging`, `random`, `dataclasses`, `typing`, `itertools`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch._inductor.codegen.rocm.ck_tile_template`, `torch._inductor.codegen.rocm.rocm_kernel`, `torch._inductor.codegen.rocm.rocm_template`, `torch._inductor.ir`, `torch.utils._ordered_set`, `...utils`

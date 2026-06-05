# ck_universal_gemm_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/rocm/ck_universal_gemm_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CKGemmTemplate`. It exposes functions such as `is_static_int`, and `torch_layout_to_ck_layout`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CKGemmTemplate` 等类。同时提供 `is_static_int`、`torch_layout_to_ck_layout` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs, disable-error-code="attr-defined, valid-type"
import copy
import logging
import math
import random
from collections import namedtuple

import sympy

import torch
from torch._inductor import config
from torch._inductor.codegen.cpp_utils import DTYPE_TO_CPP
from torch._inductor.codegen.rocm.ck_template import CKTemplate
from torch._inductor.codegen.rocm.compile_command import rocm_compile_command
from torch._inductor.codegen.rocm.rocm_kernel import ROCmTemplateKernel
from torch._inductor.ir import Buffer, Layout
from torch._inductor.runtime.runtime_utils import next_power_of_2

from ...utils import IndentedBuffer, is_dynamic, try_import_ck_lib


_, gen_ops_library, gen_ops_preselected, CKGemmOperation = try_import_ck_lib()


log = logging.getLogger(__name__)

# lightweight collection of information about a single op
InductorROCmOp = namedtuple("InductorROCmOp", ["op", "kBatch"])
````
- **EN**: Imports dependencies such as `copy`, `logging`, `math`, `random`, `collections`, `sympy`, and `...+9` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `log`, and `InductorROCmOp`.
- **CN**: 这里导入了 `copy`、`logging`、`math`、`random`、`collections`、`sympy`、`另有9项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `log`、`InductorROCmOp` 等值。

### Lines 29-56 / 第 29-56 行
````python

padding_lookup = {
    "M": {
        "GemmSpecialization::MPadding": True,
        "GemmSpecialization::MNPadding": True,
        "GemmSpecialization::MKPadding": True,
        "GemmSpecialization::MNKPadding": True,
    },
    "N": {
        "GemmSpecialization::NPadding": True,
        "GemmSpecialization::MNPadding": True,
        "GemmSpecialization::NKPadding": True,
        "GemmSpecialization::MNKPadding": True,
    },
    "K": {
        "GemmSpecialization::KPadding": True,
        "GemmSpecialization::MKPadding": True,
        "GemmSpecialization::NKPadding": True,
        "GemmSpecialization::MNKPadding": True,
    },
}


def is_static_int(number):
    return isinstance(number, (int, sympy.Integer))


def torch_layout_to_ck_layout(torch_layout):
````
- **EN**: Introduces function `is_static_int`, function `torch_layout_to_ck_layout`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `padding_lookup`.
- **CN**: 这里定义了函数`is_static_int`、函数`torch_layout_to_ck_layout`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `padding_lookup` 等值。

### Lines 57-84 / 第 57-84 行
````python
    if torch_layout.stride[-1] == 1:
        return "Row"
    elif torch_layout.stride[-2] == 1:
        return "Col"
    else:
        return None


class CKGemmTemplate(CKTemplate):
    # the JINJA template for rendering CK Universal GEMMs
    gemm_template = r"""{{version_comment}}
    {{headers}}
    {{globals}}
    {{instance_definition}}
    extern "C" {
    PT_EXPORT {{kernel_definition}} {
        auto gemm = {{instance_type}} {};
        auto invoker = gemm.MakeInvoker();
        {% if is_batched %}
        auto argument = gemm.MakeArgument(
            reinterpret_cast<const {{a_element_dtype}}*>(X),
            reinterpret_cast<const {{b_element_dtype}}*>(W),
            std::array<const void*, {{ds_size}}>{ {{ds_names}} },
            reinterpret_cast<{{c_element_dtype}}*>(Y),
            M,
            N,
            K,
            B,
````
- **EN**: Introduces class `CKGemmTemplate`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `gemm_template`, and `std`.
- **CN**: 这里定义了类`CKGemmTemplate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`gemm_template`、`std` 等值。

### Lines 85-112 / 第 85-112 行
````python
            LDA,
            LDB,
            std::array<ck::index_t, {{ds_size}}>{ {{ds_strides}} },
            LDC,
            M * K, // batch_stride_A
            N * K, // batch_stride_B
            std::array<ck::index_t, {{ds_size}}>{ {{ds_batch_strides}} },
            M * N, // batch_stride_C
            {{a_elementwise_op}},
            {{b_elementwise_op}},
            {{epilogue}} // c_elementwise_op
        );
        {% else %}
        auto argument = gemm.MakeArgument(
            reinterpret_cast<const {{a_element_dtype}}*>(X),
            reinterpret_cast<const {{b_element_dtype}}*>(W),
            std::array<const void*, {{ds_size}}>{ {{ds_names}} },
            reinterpret_cast<{{c_element_dtype}}*>(Y),
            M,
            N,
            K,
            LDA,
            LDB,
            std::array<ck::index_t, {{ds_size}}>{ {{ds_strides}} },
            LDC,
            kBatch, // kBatch
            {{a_elementwise_op}},
            {{b_elementwise_op}},
````
- **EN**: Initializes or updates values such as `std`. This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 初始化或更新了 `std` 等值。这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 113-140 / 第 113-140 行
````python
            {{epilogue}} // c_elementwise_op
        );
        {% endif %}
        if (!gemm.IsSupportedArgument(argument)) {
            // we do our best to statically avoid this case in `filter_op`
            std::cerr << "invalid argument for gemm instance " << gemm.GetTypeString() << std::endl;
            argument.Print();
            return -23;
        }
        if (workspace_size) {
            *workspace_size = gemm.GetWorkSpaceSize(&argument);
            return 0;
        }
        // run the kernel
        #ifdef GENERATE_CK_STANDALONE_RUNNER
        const auto stream_config = StreamConfig{
            stream,
            /* time kernel */ 1,
            /* log level */ 1,
            /* n_cold_iter */ 100,
            /* n_hot_iter */ 100,
            /* flush_l2_cache */ 1,
            /* rotate_count */ 5};
        #else
        const auto stream_config = StreamConfig{stream, /* time kernel */ false, /* log level */ 0};
        #endif

        const float elapsed_time = invoker.Run(argument, stream_config);
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `std` 等值。

### Lines 141-168 / 第 141-168 行
````python

        #ifdef GENERATE_CK_STANDALONE_RUNNER
        std::cout << "elapsed time: " << elapsed_time << " ms" << std::endl;
        #else
        (void)elapsed_time;
        #endif
        return 0;
    } // kernel definition
    } // extern C
    """

    standalone_runner_template = r"""
    #ifdef GENERATE_CK_STANDALONE_RUNNER
    // standalone runner for the generated CK GEMM kernel

    {{inline_utils}}

    extern "C" {
    int run_main(int argc, char** argv) {
        {% if is_batched %}
        const int32_t B = {{B}};
        {% endif %}
        const int32_t M = {{M}};
        const int32_t N = {{N}};
        const int32_t K = {{K}};
        const int32_t LDA = {{LDA}};
        const int32_t LDB = {{LDB}};
        const int32_t LDC = {{LDC}};
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `std`, and `standalone_runner_template`. This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `std`、`standalone_runner_template` 等值。这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 169-196 / 第 169-196 行
````python
        const int32_t LDD = {{LDD}};
        const int32_t kBatch = {{kBatch}};

        using AElementType = {{a_ck_dtype}};
        using BElementType = {{b_ck_dtype}};
        using CElementType = {{c_ck_dtype}};
        {% if has_bias %}
        using BiasElementType = {{bias_ck_dtype}};
        {% endif %}
        {% if has_scale %}
        using ScaleAElementType = {{scale_a_ck_dtype}};
        using ScaleBElementType = {{scale_b_ck_dtype}};
        {% endif %}

        using AArgType = {{a_torch_dtype}};
        using BArgType = {{b_torch_dtype}};
        using CArgType = {{c_torch_dtype}};
        {% if has_bias %}
        using BiasArgType = {{bias_torch_dtype}};
        {% endif %}
        {% if has_scale %}
        using ScaleAArgType = {{scale_a_torch_dtype}};
        using ScaleBArgType = {{scale_b_torch_dtype}};
        {% endif %}

        using ALayout = {{a_layout}};
        using BLayout = {{b_layout}};
        using CLayout = {{c_layout}};
````
- **EN**: This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 197-224 / 第 197-224 行
````python
        {% if has_bias %}
        using BiasLayout = {{bias_layout}};
        {% endif %}

        {% if is_batched %}
        using strides_t = std::array<int32_t, 3>;
        auto get_strides = [](int32_t batch_stride, int32_t leading_dimension, auto layout) constexpr -> strides_t {
            if constexpr (std::is_same_v<decltype(layout), Row>) {
                return {batch_stride, leading_dimension, 1};
            }
            return {batch_stride, 1, leading_dimension};
        };
        auto a_size = strides_t{B, M, K};
        auto a_stride = get_strides(M * K, LDA, ALayout{});
        auto b_size = strides_t{B, N, K};
        auto b_stride = get_strides(N * K, LDB, BLayout{});
        auto c_size = strides_t{B, M, N};
        auto c_stride = get_strides(M * N, LDC, CLayout{});
        {% else %}
        using strides_t = std::array<int32_t, 2>;
        auto get_strides = [](int32_t leading_dimension, auto layout) constexpr -> strides_t {
            if constexpr (std::is_same_v<decltype(layout), Row>) {
                return {leading_dimension, 1};
            }
            return {1, leading_dimension};
        };
        auto a_size = strides_t{M, K};
        auto a_stride = get_strides(LDA, ALayout{});
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 225-252 / 第 225-252 行
````python
        auto b_size = strides_t{N, K};
        auto b_stride = get_strides(LDB, BLayout{});
        auto c_size = strides_t{M, N};
        auto c_stride = get_strides(LDC, CLayout{});
        {% endif %}

        Tensor<AElementType> a_m_k ( HostTensorDescriptor ( a_size, a_stride ) );
        Tensor<BElementType> b_k_n ( HostTensorDescriptor ( b_size, b_stride ) );
        {% if has_bias %}
        Tensor<BiasElementType> d_m_n ( HostTensorDescriptor ( c_size, get_strides(LDD, BiasLayout{}) ) );
        {% endif %}
        {% if has_scale %}
        // NB: these are hardcoded
        Tensor<ScaleAElementType> s_a_m_n ( HostTensorDescriptor ( strides_t{M, N}, get_strides(0, Row{}) ));
        Tensor<ScaleAElementType> s_b_m_n ( HostTensorDescriptor ( strides_t{M, N}, get_strides(0, Col{}) ));
        {% endif %}

        Tensor<CElementType> c_m_n_host ( HostTensorDescriptor ( c_size, c_stride ) );
        Tensor<CElementType> c_m_n_device ( HostTensorDescriptor ( c_size, c_stride ) );

        a_m_k.GenerateTensorValue(GeneratorTensor_2<AElementType>());
        b_k_n.GenerateTensorValue(GeneratorTensor_2<BElementType>());
        {% if has_bias %}
        d_m_n.GenerateTensorValue(GeneratorTensor_2<BiasElementType>());
        {% endif %}
        {% if has_scale %}
        s_a_m_n.GenerateTensorValue(GeneratorTensor_2<ScaleAElementType>());
        s_b_m_n.GenerateTensorValue(GeneratorTensor_2<ScaleBElementType>());
````
- **EN**: This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 253-280 / 第 253-280 行
````python
        {% endif %}
        DeviceMem a_m_k_device_buf(sizeof(AElementType) * a_m_k.mDesc.GetElementSpaceSize());
        DeviceMem b_k_n_device_buf(sizeof(BElementType) * b_k_n.mDesc.GetElementSpaceSize());
        {% if has_bias %}
        DeviceMem d_m_n_device_buf(sizeof(BiasElementType) * d_m_n.mDesc.GetElementSpaceSize());
        {% endif %}
        {% if has_scale %}
        DeviceMem s_a_m_n_device_buf(sizeof(ScaleAElementType) * s_a_m_n.mDesc.GetElementSpaceSize());
        DeviceMem s_b_m_n_device_buf(sizeof(ScaleBElementType) * s_b_m_n.mDesc.GetElementSpaceSize());
        {% endif %}
        DeviceMem c_m_n_device_buf(sizeof(CElementType) * c_m_n_device.mDesc.GetElementSpaceSize());

        a_m_k_device_buf.ToDevice(a_m_k.mData.data());
        b_k_n_device_buf.ToDevice(b_k_n.mData.data());
        {% if has_bias %}
        d_m_n_device_buf.ToDevice(d_m_n.mData.data());
        {% endif %}
        {% if has_scale %}
        s_a_m_n_device_buf.ToDevice(s_a_m_n.mData.data());
        s_b_m_n_device_buf.ToDevice(s_b_m_n.mData.data());
        {% endif %}

        {{kernel_name}}(
            static_cast<const AArgType*>(a_m_k_device_buf.GetDeviceBuffer()),
            static_cast<const BArgType*>(b_k_n_device_buf.GetDeviceBuffer()),
            {% if has_scale %}
            static_cast<const ScaleAArgType*>(s_a_m_n_device_buf.GetDeviceBuffer()),
            static_cast<const ScaleBArgType*>(s_b_m_n_device_buf.GetDeviceBuffer()),
````
- **EN**: This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
            {% endif %}
            {% if has_bias %}
            static_cast<const BiasArgType*>(d_m_n_device_buf.GetDeviceBuffer()),
            {% endif %}
            static_cast<CArgType*>(c_m_n_device_buf.GetDeviceBuffer()),
            {% if is_batched %}
            B,
            {% endif %}
            M,
            N,
            K,
            LDA,
            LDB,
            LDC,
            LDD,
            nullptr, // workspace_size
            nullptr, // workspace
            nullptr); // stream

        hip_check_error(hipDeviceSynchronize());

        return 0;
    } // run_main
    } // extern C

    int main(int argc, char** argv) {
        return run_main(argc, argv);
    }
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of class `CKGemmTemplate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了类`CKGemmTemplate` 的具体实现。

### Lines 309-336 / 第 309-336 行
````python
    // compile with: {{compile_cmd}}
    #endif // GENERATE_CK_STANDALONE_RUNNER
    """

    def __init__(
        self,
        input_nodes: list[Buffer],
        layout: Layout,
        alpha: float,
        beta: float,
        input_reorder: list[int] | None = None,
    ) -> None:
        is_batched = len(layout.size) == 3
        name = "ck_batched_gemm_template" if is_batched else "ck_gemm_template"
        super().__init__(
            name=name,
            input_nodes=input_nodes,
            layout=layout,
            input_reorder=input_reorder,
        )
        self.alpha = alpha
        self.beta = beta
        self.is_batched = is_batched

    def header(self) -> IndentedBuffer:
        res = super().header()
        if self.is_batched:
            res.splice(
````
- **EN**: Introduces function `__init__`, function `header`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_nodes`, `layout`, `alpha`, `beta`, `input_reorder`, `is_batched`, and `...+2`.
- **CN**: 这里定义了函数`__init__`、函数`header`。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_nodes`、`layout`、`alpha`、`beta`、`input_reorder`、`is_batched`、`另有2项` 等值。

### Lines 337-364 / 第 337-364 行
````python
                """
                    // CK GEMM header(s)

                    #include "ck/tensor_operation/gpu/device/impl/device_batched_gemm_multiple_d_xdl_cshuffle_v3.hpp"
                """
            )
        else:
            res.splice(
                """
                    // CK GEMM header(s)

                    #include "ck/tensor_operation/gpu/device/impl/device_gemm_multiple_d_xdl_cshuffle_v3.hpp"
                """
            )
        return res

    def globals(self) -> IndentedBuffer:
        res = super().globals()
        res.splice(
            """
                // CK GEMM globals

                using Row = ck::tensor_layout::gemm::RowMajor;
                using Col = ck::tensor_layout::gemm::ColumnMajor;

                using BlockGemmPipelineScheduler = ck::BlockGemmPipelineScheduler;
                using GemmSpecialization = ck::tensor_operation::device::GemmSpecialization;
                using BlockGemmPipelineVersion = ck::BlockGemmPipelineVersion;
````
- **EN**: Introduces function `globals`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`globals`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 365-392 / 第 365-392 行
````python

                struct MultiplyMultiplyAdd {
                    template <typename E, typename C, typename D0, typename D1, typename D2>
                    __host__ __device__ constexpr void
                    operator()(E& e, const C& c, const D0& d0, const D1& d1, const D2& d2) const {
                        e = ck::type_convert<E>(
                           ck::type_convert<float>(c)
                           * ck::type_convert<float>(d0)
                           * ck::type_convert<float>(d1)
                           + ck::type_convert<float>(d2)
                        );
                    }
                };
            """
        )
        return res

    def inline_utils(self):
        res = IndentedBuffer()
        res.splice(
            """
                #include "host_tensor.cpp"
                #include "device_memory.cpp"
            """
        )
        return res

    def _has_padding(self, dimension, gemm_specialization):
````
- **EN**: Introduces function `inline_utils`, function `_has_padding`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `e`, `ck`, and `res`.
- **CN**: 这里定义了函数`inline_utils`、函数`_has_padding`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `e`、`ck`、`res` 等值。

### Lines 393-420 / 第 393-420 行
````python
        # Get the relevant padding map for the given dimension
        dimension_padding = padding_lookup.get(dimension, {})

        # Check if the specialization is in the dimension's padding map
        return dimension_padding.get(gemm_specialization, False)

    def filter_op(self, op_info: InductorROCmOp):
        """
        Determines whether a given op definition is suitable for the current
        input / output of the operation that this template implements.

        Filter is based on inputs' dtype, layout and statically inferred size.

        Returns None if the op is not suitable, otherwise returns the op to be used.
        """
        op, kBatch = op_info.op, op_info.kBatch
        metas = [T.get_layout() for T in [*self.input_nodes, self.output_node]]
        X_meta = metas[0]
        W_meta = metas[1]
        Y_meta = metas[-1]
        # disable the instance if dtypes don't match
        if op.a_element_dtype != self._TORCH_DTYPE_TO_CK[X_meta.dtype]:
            return None
        if op.b_element_dtype != self._TORCH_DTYPE_TO_CK[W_meta.dtype]:
            return None
        if op.c_element_dtype != self._TORCH_DTYPE_TO_CK[Y_meta.dtype]:
            return None
        # disable the instance if layouts don't match
````
- **EN**: Introduces function `filter_op`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`filter_op`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 421-448 / 第 421-448 行
````python
        if op.a_layout != torch_layout_to_ck_layout(X_meta):
            return None
        if op.b_layout != torch_layout_to_ck_layout(W_meta):
            return None
        if op.c_layout != torch_layout_to_ck_layout(Y_meta):
            return None
        # try to avoid launching the instance with invalid problem size
        # see GridwiseGemm_xdl_cshuffle_v3::CheckValidity

        M = X_meta.size[-2]
        K = X_meta.size[-1]
        N = W_meta.size[-1]

        if is_static_int(M):
            if not self._has_padding("M", op.gemm_specialization):
                if M % op.m_per_block != 0:
                    return None
        if is_static_int(N):
            if not self._has_padding("N", op.gemm_specialization):
                if N % op.n_per_block != 0:
                    return None
        if is_static_int(K):
            if not self._has_padding("K", op.gemm_specialization):
                if K % op.k_per_block != 0:
                    return None
                K_t = kBatch * op.k_per_block
                if K % K_t != 0:
                    return None
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `M`, `K`, `N`, and `K_t`. This range continues the implementation of function `CKGemmTemplate.filter_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `M`、`K`、`N`、`K_t` 等值。这一段延续了函数`CKGemmTemplate.filter_op` 的具体实现。

### Lines 449-476 / 第 449-476 行
````python
            else:
                # need another kBatch check here
                lcm = abs(op.a_k1 * op.b_k1) // math.gcd(op.a_k1, op.b_k1)
                K_t = kBatch * lcm
                k_read_pad_splited = math.ceil(K / K_t) * lcm
                if (k_read_pad_splited * (kBatch - 1)) >= K:
                    return None

        a_contig_size = (
            K if op.a_layout == "Row" else M if op.a_layout == "Col" else None
        )
        if (
            is_static_int(a_contig_size)
            and a_contig_size % op.a_block_transfer_src_scalar_per_vector != 0
        ):
            return None
        b_contig_size = (
            N if op.b_layout == "Row" else K if op.b_layout == "Col" else None
        )
        if (
            is_static_int(b_contig_size)
            and b_contig_size % op.b_block_transfer_src_scalar_per_vector != 0
        ):
            return None
        c_contig_size = (
            N if op.c_layout == "Row" else M if op.c_layout == "Col" else None
        )
        c_shuffle_block_transfer_scalar_per_vector_n_per_block = (
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `lcm`, `K_t`, `k_read_pad_splited`, `a_contig_size`, `b_contig_size`, and `...+2`. This range continues the implementation of function `CKGemmTemplate.filter_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`lcm`、`K_t`、`k_read_pad_splited`、`a_contig_size`、`b_contig_size`、`另有2项` 等值。这一段延续了函数`CKGemmTemplate.filter_op` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
            op.c_shuffle_block_transfer_scalar_per_vector_n_per_block[0]
            if isinstance(
                op.c_shuffle_block_transfer_scalar_per_vector_n_per_block, tuple
            )
            else op.c_shuffle_block_transfer_scalar_per_vector_n_per_block
        )
        if (
            is_static_int(c_contig_size)
            and c_contig_size % c_shuffle_block_transfer_scalar_per_vector_n_per_block
            != 0
        ):
            return None
        if not self._check_num_k_loops(op, kBatch):
            return None
        # TBD disable instances with invalid number of pipeline prefetch stages
        # It will avoid compiling a small percentage of unrunnable instances which fail the gemm argument check

        return op

    def _check_num_k_loops(self, op, kBatch):
        # Additional splitK scenario check
        metas = [T.get_layout() for T in [*self.input_nodes]]
        X_meta = metas[0]
        W_meta = metas[1]
        K = X_meta.size[-1]
        if kBatch > 1:
            if op.block_gemm_pipeline_version != "BlockGemmPipelineVersion::v1":
                try:
````
- **EN**: Introduces function `_check_num_k_loops`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `metas`, `X_meta`, `W_meta`, `K`, and `try`.
- **CN**: 这里定义了函数`_check_num_k_loops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `metas`、`X_meta`、`W_meta`、`K`、`try` 等值。

### Lines 505-532 / 第 505-532 行
````python
                    prefetch_stages = self._prefetch_stages(
                        op,
                        torch.empty((), dtype=X_meta.dtype).element_size(),
                        torch.empty((), dtype=W_meta.dtype).element_size(),
                        torch.cuda.get_device_properties(X_meta.device).warp_size,
                    )
                except Exception as e:
                    log.debug(
                        "Failed to prefetch_stages for %s with exception %s", op.name, e
                    )
                    # be conservative here and disable the op
                    return False

                K_t = op.k_per_block * kBatch
                ak0 = (K + K_t - 1) // K_t * (op.k_per_block // op.a_k1)
                num_k_loop = ak0 // (op.k_per_block // op.a_k1)
                if num_k_loop <= prefetch_stages:
                    log.debug(
                        "Op %s is not compatible due to invalid number of pipeline prefetch stages. "
                        "Parameters: kBatch=%s, block_gemm_pipeline_version=%s, prefetch_stages=%s, num_k_loop=%s",
                        op.name(),
                        kBatch,
                        op.block_gemm_pipeline_version,
                        prefetch_stages,
                        num_k_loop,
                    )
                    return False

````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 533-560 / 第 533-560 行
````python
        return True

    # small helper to figure out the prefetch stages on AMD
    def _prefetch_stages(self, op, a_dtype_size, b_dtype_size, warp_size: int = 64):
        version_str = op.block_gemm_pipeline_version.split("::")[-1]
        try:
            version = int(version_str[1:])  # Assuming the format is always 'vX'
        except ValueError as e:
            raise ValueError(f"Invalid version string: {version_str}") from e
        if version not in [1, 2, 3, 4, 5]:
            raise ValueError(
                f"unknown prefetch stages for {op.block_gemm_pipeline_version}"
            )
        # Define the mapping of versions to stages
        version_to_stages = {1: 1, 3: 2, 4: 4, 5: 3}
        # Get the stages for the given version
        stages = version_to_stages.get(version)
        if stages is None:
            # This means we're at stage 2, and this requires computation
            # See github.com/ROCm/composable_kernel/blob/d6a4605/include/ck/tensor_operation/gpu/block/blockwise_gemm_pipeline_xdlops_v2.hpp#L143
            wgp_per_cu = max(4 * warp_size // op.block_size, 1)
            full_mem_band_prefetch_stages = math.ceil(
                32768
                / wgp_per_cu
                / (
                    (op.m_per_block * a_dtype_size + op.n_per_block * b_dtype_size)
                    * op.k_per_block
                )
````
- **EN**: Introduces function `_prefetch_stages`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_prefetch_stages`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 561-588 / 第 561-588 行
````python
            )
            stages = min(max(full_mem_band_prefetch_stages, 2), 8)

        return stages

    def emit_ck_instance(self, op: "CKGemmOperation"):
        # The Jinja template for generating a C++ type alias *definition* for a Universal GEMM instance
        struct_name = (
            "DeviceBatchedGemmMultiD_Xdl_CShuffle_V3"
            if self.is_batched
            else "DeviceGemmMultiD_Xdl_CShuffle_V3"
        )
        template_definition = r"""
    // Gemm operator {{operation_name}}
    using Operation_{{operation_name}} =
        ck::tensor_operation::device::{{struct_name}}<
            {{template_params}}>;

"""
        # The Jinja template for generating a C++ type alias *usage* for a Universal GEMM instance
        template_type = r"""
    Operation_{{operation_name}}
"""
        template_params = []
        for field_name, field_value in op.dict_items():
            if isinstance(field_value, tuple):
                tuple_elements = ", ".join(map(str, iter(field_value)))
                if "ds" in field_name:  # element type and layout for bias
````
- **EN**: Introduces function `emit_ck_instance`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`emit_ck_instance`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 589-616 / 第 589-616 行
````python
                    arg = f"/* {field_name} */ Tuple<{tuple_elements}>"
                else:  # tile shape
                    arg = f"/* {field_name} */ S<{tuple_elements}>"
                # pyrefly: ignore [bad-argument-type]
                template_params.append(arg)
            else:
                if field_value is not None:
                    # pyrefly: ignore [bad-argument-type]
                    template_params.append(f"/* {field_name} */ {field_value}")
        operation_name = op.name().replace("(", "").replace(",", "").replace(")", "")
        return self._template_from_string(template_definition).render(
            operation_name=operation_name,
            template_params=(",\n" + 12 * " ").join(template_params),
            struct_name=struct_name,
        ), self._template_from_string(template_type).render(
            operation_name=operation_name
        )

    def render(  # type: ignore[override]
        self,
        kernel: ROCmTemplateKernel,
        op: "CKGemmOperation",
        **kwargs,
    ) -> str:
        """
        The primary entry point for the code rendering process used in this template.
        """
        epilogue_nodes = kwargs.get("epilogue_nodes")
````
- **EN**: Introduces function `render`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `arg`, `else`, `operation_name`, `template_params`, `struct_name`, `kernel`, and `...+2`.
- **CN**: 这里定义了函数`render`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `arg`、`else`、`operation_name`、`template_params`、`struct_name`、`kernel`、`另有2项` 等值。

### Lines 617-644 / 第 617-644 行
````python
        assert epilogue_nodes is None or 0 == len(epilogue_nodes)
        template_buffer_node = kwargs.get("template_buffer_node")
        if template_buffer_node is not None:
            self.output_node = template_buffer_node
        # input nodes:
        # * X, W for matmul
        # * X, W, Bias for addmm
        # * X, W, inv_scale_x, inv_scale_w for scaled_mm
        # * X, W, inv_scale_x, inv_scale_w, Bias for scaled_mm with bias
        X, W = self.input_nodes[0], self.input_nodes[1]
        Y = self.output_node
        Bias = (
            self.input_nodes[2]
            if 3 == len(self.input_nodes)
            else self.input_nodes[4]
            if 5 == len(self.input_nodes)
            else None
        )
        has_bias = Bias is not None
        has_scale = len(self.input_nodes) in (4, 5)
        op = copy.deepcopy(op)

        # This parameter is converted into tuple because of change
        # from DeviceGemm_Xdl_CShuffleV3 to DeviceGemmMultiD_Xdl_CShuffle_V3.
        # The first tuple element corresponds to matmul result...
        if not isinstance(
            op.c_shuffle_block_transfer_scalar_per_vector_n_per_block, tuple
        ):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template_buffer_node`, `Y`, `Bias`, `has_bias`, `has_scale`, and `op`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `template_buffer_node`、`Y`、`Bias`、`has_bias`、`has_scale`、`op` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 645-672 / 第 645-672 行
````python
            op.c_shuffle_block_transfer_scalar_per_vector_n_per_block = (
                op.c_shuffle_block_transfer_scalar_per_vector_n_per_block,
            )

        if has_scale:
            scale_x = self.input_nodes[2]
            scale_w = self.input_nodes[3]
            if 1 == scale_x.get_numel() and 1 == scale_w.get_numel():
                # tensorwise scale for both X, W
                if has_bias:
                    op.c_elementwise_op = "ScaleAdd"
                else:
                    op.c_elementwise_op = "Scale"
            else:
                # rowwise scale for both X, W
                if has_bias:
                    op.c_elementwise_op = "MultiplyMultiplyAdd"
                else:
                    op.c_elementwise_op = "MultiplyMultiply"
                op.c_shuffle_dtype = "F32"
                op.ds_layouts = (
                    torch_layout_to_ck_layout(scale_x.get_layout()),
                    torch_layout_to_ck_layout(scale_w.get_layout()),
                )
                op.ds_element_dtypes = (
                    self._TORCH_DTYPE_TO_CK[scale_x.get_layout().dtype],
                    self._TORCH_DTYPE_TO_CK[scale_w.get_layout().dtype],
                )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scale_x`, `scale_w`, and `else`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `scale_x`、`scale_w`、`else` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
                op.c_shuffle_block_transfer_scalar_per_vector_n_per_block += (1, 1)
        else:
            scale_x = None
            scale_w = None

        bias_dtype = ""
        if Bias is not None:
            bias_layout = torch_layout_to_ck_layout(Bias.get_layout())
            bias_dtype = self._TORCH_DTYPE_TO_CK[Bias.get_layout().dtype]
            op.ds_layouts += (bias_layout,)
            op.ds_element_dtypes += (bias_dtype,)
            if not has_scale:
                op.c_elementwise_op = "Bilinear"
            # c_shuffle_dtype is also used for adding bias to matmul result
            # before converting down to the result dtype
            op.c_shuffle_dtype = op.acc_dtype
            # this parameter needs to be set accordingly to bias stride for correct accumulation
            if bias_layout == "Row":
                # bias has (N, ) shape
                bias_shuffle_block_transfer_scalar_per_vector_n_per_block = (
                    op.c_shuffle_block_transfer_scalar_per_vector_n_per_block
                )
            elif bias_layout == "Col":
                # bias has (M, 1) shape
                bias_shuffle_block_transfer_scalar_per_vector_n_per_block = (1,)
            else:
                raise AssertionError(
                    "Bias layout is neither row-major nor column-major"
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `scale_x`, `scale_w`, `bias_dtype`, `bias_layout`, and `bias_shuffle_block_transfer_scalar_per_vector_n_per_block`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`scale_x`、`scale_w`、`bias_dtype`、`bias_layout`、`bias_shuffle_block_transfer_scalar_per_vector_n_per_block` 等值。

### Lines 701-728 / 第 701-728 行
````python
                )
            # ...and the second tuple element corresponds to the bias
            op.c_shuffle_block_transfer_scalar_per_vector_n_per_block += (
                bias_shuffle_block_transfer_scalar_per_vector_n_per_block
            )

        instance_definition, instance_type = self.emit_ck_instance(op)

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
        epilogue = None

        if op.c_elementwise_op == "Bilinear" and scale_w is None:
            epilogue = f"Bilinear {{ {self.alpha}, {self.beta} }}"

        elif op.c_elementwise_op == "Scale":
            epilogue = "Scale { (inv_scale_w && inv_scale_x) ? (*inv_scale_w * *inv_scale_x) : 1.0f }"

        elif op.c_elementwise_op == "ScaleAdd":
            epilogue = "ScaleAdd { (inv_scale_w && inv_scale_x) ? (*inv_scale_w * *inv_scale_x) : 1.0f }"
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `version_comment`, and `epilogue`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `version_comment`、`epilogue` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python

        elif op.c_elementwise_op == "MultiplyMultiply":
            epilogue = "MultiplyMultiply {}"

        elif op.c_elementwise_op == "MultiplyMultiplyAdd":
            epilogue = "MultiplyMultiplyAdd {}"

        elif op.c_elementwise_op == "PassThrough":
            epilogue = "PassThrough {}"

        assert epilogue is not None, "CK GEMM epilogue is not set"

        size_arg_strs = ["M", "N", "K", "LDA", "LDB", "LDC", "LDD"]
        if self.is_batched:
            size_arg_strs.insert(0, "B")

        res = self._template_from_string(self.gemm_template).render(
            inline_utils=self.inline_utils(),
            headers=self.header().getvalue(),
            globals=self.globals().getvalue(),
            instance_definition=instance_definition,
            kernel_definition=kernel.def_kernel(
                inputs=[X, W, scale_x, scale_w, Bias],  # type: ignore[list-item]
                outputs=[Y],
                names_str="X, W, inv_scale_x, inv_scale_w, Bias, Y",
                input_reorder=self.input_reorder,
                size_args=[f"int32_t {arg}" for arg in size_arg_strs],
            ),
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `epilogue`, `size_arg_strs`, `res`, `inline_utils`, `headers`, `globals`, and `...+7`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `epilogue`、`size_arg_strs`、`res`、`inline_utils`、`headers`、`globals`、`另有7项` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 757-784 / 第 757-784 行
````python
            instance_type=instance_type,
            a_element_dtype=op.a_element_dtype,
            b_element_dtype=op.b_element_dtype,
            c_element_dtype=op.c_element_dtype,
            bias_element_dtype=bias_dtype,
            alpha=self.alpha,
            beta=self.beta,
            a_elementwise_op="PassThrough {}",
            b_elementwise_op="PassThrough {}",
            epilogue=epilogue,
            has_bias=has_bias,
            ds_size=1
            if op.c_elementwise_op in ("Bilinear", "ScaleAdd")
            else 2
            if op.c_elementwise_op == "MultiplyMultiply"
            else 3
            if op.c_elementwise_op == "MultiplyMultiplyAdd"
            else 0,
            ds_names=", ".join(
                ["Bias"]
                if op.c_elementwise_op in ("Bilinear", "ScaleAdd")
                else ["inv_scale_x", "inv_scale_w"]
                if op.c_elementwise_op == "MultiplyMultiply"
                else ["inv_scale_x", "inv_scale_w", "Bias"]
                if op.c_elementwise_op == "MultiplyMultiplyAdd"
                else []
            ),
            ds_strides=", ".join(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `instance_type`, `a_element_dtype`, `b_element_dtype`, `c_element_dtype`, `bias_element_dtype`, `alpha`, and `...+8`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `instance_type`、`a_element_dtype`、`b_element_dtype`、`c_element_dtype`、`bias_element_dtype`、`alpha`、`另有8项` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 785-812 / 第 785-812 行
````python
                ["LDD"]
                if op.c_elementwise_op in ("Bilinear", "ScaleAdd")
                else ["0", "0"]
                if op.c_elementwise_op == "MultiplyMultiply"
                else ["0", "0", "LDD"]
                if op.c_elementwise_op == "MultiplyMultiplyAdd"
                else []
            ),
            version_comment=version_comment,
            is_batched=self.is_batched,
            ds_batch_strides=", ".join([]),  # FIXME when supporting baddbmm
        )

        if config.rocm.generate_test_runner:
            is_static_problem = all(is_static_int(arg) for arg in self.size_args())
            # NOTE: size_arg_strs is defined above
            size_arg_vals = (
                self.size_args()
                if is_static_problem
                else (
                    f"std::stoi(argv[{k}])" for k, _ in enumerate(self.size_args(), 1)
                )
            )
            size_args = dict(zip(size_arg_strs, size_arg_vals, strict=True))
            runtime_args = dict(
                zip(
                    [a.name for a in self.get_runtime_arg_info()],
                    self.get_runtime_arg_values(),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `version_comment`, `is_batched`, `ds_batch_strides`, `is_static_problem`, `size_arg_vals`, `size_args`, and `...+1`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `version_comment`、`is_batched`、`ds_batch_strides`、`is_static_problem`、`size_arg_vals`、`size_args`、`另有1项` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
                )
            )
            runner_code = self._template_from_string(
                self.standalone_runner_template
            ).render(
                inline_utils=self.inline_utils().getvalue(),
                kernel_name=kernel.kernel_name,
                has_bias=has_bias,
                has_scale=has_scale,
                is_batched=self.is_batched,
                a_ck_dtype=op.a_element_dtype,
                b_ck_dtype=op.b_element_dtype,
                c_ck_dtype=op.c_element_dtype,
                bias_ck_dtype=op.ds_element_dtypes[0] if has_bias else "",
                scale_a_ck_dtype=op.ds_element_dtypes[0]
                if has_scale and 2 == len(op.ds_element_dtypes)
                else "BF16",
                scale_b_ck_dtype=op.ds_element_dtypes[1]
                if has_scale and 2 == len(op.ds_element_dtypes)
                else "BF16",
                a_torch_dtype=DTYPE_TO_CPP[X.get_layout().dtype],
                b_torch_dtype=DTYPE_TO_CPP[W.get_layout().dtype],
                c_torch_dtype=DTYPE_TO_CPP[Y.get_layout().dtype],
                bias_torch_dtype=DTYPE_TO_CPP[Bias.get_layout().dtype]
                if Bias is not None
                else "",
                scale_a_torch_dtype=DTYPE_TO_CPP[scale_x.get_layout().dtype]
                if scale_x is not None
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `runner_code`, `inline_utils`, `kernel_name`, `has_bias`, `has_scale`, `is_batched`, and `...+11`. This range continues the implementation of function `CKGemmTemplate.render`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `runner_code`、`inline_utils`、`kernel_name`、`has_bias`、`has_scale`、`is_batched`、`另有11项` 等值。这一段延续了函数`CKGemmTemplate.render` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
                else "",
                scale_b_torch_dtype=DTYPE_TO_CPP[scale_w.get_layout().dtype]
                if scale_w is not None
                else "",
                a_layout=torch_layout_to_ck_layout(X.get_layout()),
                b_layout=torch_layout_to_ck_layout(W.get_layout()),
                c_layout=torch_layout_to_ck_layout(Y.get_layout()),
                bias_layout=torch_layout_to_ck_layout(Bias.get_layout())
                if Bias is not None
                else "",
                compile_cmd=rocm_compile_command(
                    ["<source_file_name>"], "<executable_name>", "exe"
                ),
                **size_args,
                **runtime_args,
            )
            res += runner_code

        return res

    def _is_rcr_f16(self):
        X_meta, W_meta, Y_meta = (
            T.get_layout() for T in [*self.input_nodes, self.output_node]
        )
        X_dtype, W_dtype, Y_dtype = (
            self._TORCH_DTYPE_TO_CK[m.dtype] for m in (X_meta, W_meta, Y_meta)
        )
        X_layout, W_layout, Y_layout = (
````
- **EN**: Introduces function `_is_rcr_f16`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_rcr_f16`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 869-896 / 第 869-896 行
````python
            torch_layout_to_ck_layout(m) for m in (X_meta, W_meta, Y_meta)
        )

        return (
            X_dtype == "F16"
            and W_dtype == "F16"
            and Y_dtype == "F16"
            and X_layout == "Row"
            and W_layout == "Col"
            and Y_layout == "Row"
        )

    # helper to calculate a potentially optimal kBatch(es) for a problem
    def _get_kBatch(self, op):
        # we only set a higher kBatch if K > 16 * the larger of M and N
        # this is a hand-tuned heuristic to start
        metas = [T.get_layout() for T in [*self.input_nodes]]
        X_meta = metas[0]
        W_meta = metas[1]
        M = X_meta.size[-2]
        K = X_meta.size[-1]
        N = W_meta.size[-1]
        if is_dynamic(*self.input_nodes):
            return [1]
        if K // max(M, N) < config.rocm.split_k_threshold:
            return [1]
        # if the user is telling us which kBatches to sweep, just use those
        if config.rocm.kBatch_sweep is not None:
````
- **EN**: Introduces function `_get_kBatch`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `X_dtype`, `metas`, `X_meta`, `W_meta`, `M`, `K`, and `...+1`.
- **CN**: 这里定义了函数`_get_kBatch`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `X_dtype`、`metas`、`X_meta`、`W_meta`、`M`、`K`、`另有1项` 等值。

### Lines 897-924 / 第 897-924 行
````python
            return config.rocm.kBatch_sweep
        # Calculate the number of blocks needed for each dimension
        total_k_blocks = math.ceil(K / op.k_per_block)
        # we want to calculate how many blocks we need to fit per CU
        cus = torch.cuda.get_device_properties(X_meta.device).multi_processor_count
        # again, manual heuristics as much larger kBatch are significantly worse in
        # initial testing
        kBatch = min(max(next_power_of_2(total_k_blocks // cus), 1), 128)
        return [kBatch]

    def gen_ops(self) -> list[InductorROCmOp]:
        """
        Creates a list of `CKGemmOperation` instances that match the GEMM operation this template represents.
        The instances are guaranteed to have the correct layout, dtype and dimension padding for the GEMM input arguments.

        An instance may invalidate the GEMM configuration at runtime.
        Such instances will be assigned +inf runtime by the autotune process.
        """
        try:
            from ck4inductor.batched_universal_gemm.gen_instances import (  # type: ignore[import]
                gen_ops_library as gen_batched_gemm_ops_library,
            )
            from ck4inductor.universal_gemm.gen_instances import (  # type: ignore[import]
                gen_ops_library as gen_gemm_ops_library,
                gen_ops_preselected as gen_gemm_ops_preselected,
            )
        except ImportError:
            return []
````
- **EN**: Imports dependencies such as `ck4inductor.batched_universal_gemm.gen_instances`, and `ck4inductor.universal_gemm.gen_instances` for the logic in this range. Introduces function `gen_ops`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `ck4inductor.batched_universal_gemm.gen_instances`、`ck4inductor.universal_gemm.gen_instances` 等依赖，为后续逻辑提供基础能力。这里定义了函数`gen_ops`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 925-952 / 第 925-952 行
````python

        generator = None
        if self.is_batched:
            generator = gen_batched_gemm_ops_library
        else:
            generator = gen_gemm_ops_library
        if config.rocm.use_preselected_instances and self._is_rcr_f16():
            generator = gen_gemm_ops_preselected

        assert generator is not None

        rops = generator()
        ops = []
        for o in rops:
            kBatches = self._get_kBatch(o)
            for kBatch in kBatches:
                # pyrefly: ignore [bad-argument-type]
                ops.append(InductorROCmOp(op=o, kBatch=kBatch))

        filtered_instances = list(filter(lambda op: self.filter_op(op), ops))

        # NB: when using a fixed list order, most likely we will pick the subset of instances
        # which are very similar to each other. Randomizing the choice seems to solve this.
        random.seed(-11)
        chosen_instances = (
            random.sample(
                filtered_instances,
                min(len(filtered_instances), config.rocm.ck_max_profiling_configs),
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `generator`, `else`, `rops`, `ops`, `kBatches`, `filtered_instances`, and `...+1`. This range continues the implementation of function `CKGemmTemplate.gen_ops`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `generator`、`else`、`rops`、`ops`、`kBatches`、`filtered_instances`、`另有1项` 等值。这一段延续了函数`CKGemmTemplate.gen_ops` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
            )
            if config.rocm.ck_max_profiling_configs
            else filtered_instances
        )
        log.debug(
            "generated %d ck instances after filter: %s",
            len(chosen_instances),
            chosen_instances,
        )
        return chosen_instances

    @staticmethod
    def add_ck_gemm_choices(
        choices,
        layout,
        input_nodes,
        alpha=1,
        beta=0,
        input_reorder=None,
    ):
        """
        Add Composable Kernel Universal GEMM instance choices to the auto-tuning list.
        """
        template = CKGemmTemplate(
            input_nodes,
            layout,
            alpha=alpha,
            beta=beta,
````
- **EN**: Introduces function `add_ck_gemm_choices`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`add_ck_gemm_choices`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 981-1008 / 第 981-1008 行
````python
            input_reorder=input_reorder,
        )
        ops = template.gen_ops()
        for op in ops:
            template.maybe_append_choice(
                choices,
                op=op.op,
                kBatch=op.kBatch,
            )

    def size_args(self):
        X = self.input_nodes[0]
        W = self.input_nodes[1]
        Bias = (
            self.input_nodes[2]
            if len(self.input_nodes) == 3
            else self.input_nodes[4]
            if len(self.input_nodes) == 5
            else None
        )
        Y = self.output_node

        M = X.get_size()[-2]
        K = X.get_size()[-1]
        N = W.get_size()[-1]
        LDA = X.get_stride()[-2 if X.get_stride()[-1] == 1 else -1]
        LDB = W.get_stride()[-2 if W.get_stride()[-1] == 1 else -1]
        LDC = Y.get_stride()[-2 if Y.get_stride()[-1] == 1 else -1]
````
- **EN**: Introduces function `size_args`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_reorder`, `ops`, `op`, `kBatch`, `X`, `W`, and `...+8`.
- **CN**: 这里定义了函数`size_args`。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_reorder`、`ops`、`op`、`kBatch`、`X`、`W`、`另有8项` 等值。

### Lines 1009-1018 / 第 1009-1018 行
````python
        LDD = (
            0
            if (Bias is None or len(Bias.get_size()) == 1)
            else Bias.get_stride()[-2 if Bias.get_stride()[-1] == 1 else -1]
        )
        if self.is_batched:
            B = X.get_size()[0]
            return B, M, N, K, LDA, LDB, LDC, LDD
        else:
            return M, N, K, LDA, LDB, LDC, LDD
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `LDD`, `B`, and `else`. This range continues the implementation of function `CKGemmTemplate.size_args`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `LDD`、`B`、`else` 等值。这一段延续了函数`CKGemmTemplate.size_args` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `logging`, `math`, `random`, `collections`
- **Third-party / 第三方**: `sympy`, `ck4inductor.batched_universal_gemm.gen_instances`, `ck4inductor.universal_gemm.gen_instances`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch._inductor.codegen.cpp_utils`, `torch._inductor.codegen.rocm.ck_template`, `torch._inductor.codegen.rocm.compile_command`, `torch._inductor.codegen.rocm.rocm_kernel`, `torch._inductor.ir`, `torch._inductor.runtime.runtime_utils`, `...utils`

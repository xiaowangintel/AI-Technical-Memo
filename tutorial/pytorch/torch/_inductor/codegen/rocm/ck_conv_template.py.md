# ck_conv_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/rocm/ck_conv_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CKGroupedConvFwdTemplate`. It exposes functions such as `torch_layout_to_ck_layouts`, `torch_layout_to_ck_input_layout`, `torch_layout_to_ck_weight_layout`, and `torch_layout_to_ck_output_layout`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CKGroupedConvFwdTemplate` 等类。同时提供 `torch_layout_to_ck_layouts`、`torch_layout_to_ck_input_layout`、`torch_layout_to_ck_weight_layout`、`torch_layout_to_ck_output_layout` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import copy
import logging
import random
from typing import Any
from typing_extensions import override

from torch._inductor.virtualized import V

from .rocm_template import ArgInfo


try:
    import ck4inductor  # type: ignore[import]
except ImportError:
    ck4inductor = None

if ck4inductor is not None:
    from ck4inductor.grouped_conv_fwd.gen_instances import (  # type: ignore[import]
        gen_conv_ops_library,
````
- **EN**: Imports dependencies such as `copy`, `logging`, `random`, `typing`, `typing_extensions`, `torch._inductor.virtualized`, and `...+3` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, and `ck4inductor`.
- **CN**: 这里导入了 `copy`、`logging`、`random`、`typing`、`typing_extensions`、`torch._inductor.virtualized`、`另有3项` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`ck4inductor` 等值。

### Lines 21-40 / 第 21-40 行
````python
    )
    from ck4inductor.grouped_conv_fwd.op import (  # type: ignore[import]  # noqa: TCH002
        CKGroupedConvFwdOp,
    )
else:

    def gen_conv_ops_library():
        return []


from torch._inductor import config
from torch._inductor.codegen.rocm.ck_template import CKTemplate
from torch._inductor.codegen.rocm.rocm_kernel import ROCmTemplateKernel
from torch._inductor.utils import IndentedBuffer


log = logging.getLogger(__name__)


def torch_layout_to_ck_layouts(torch_layout):
````
- **EN**: Imports dependencies such as `ck4inductor.grouped_conv_fwd.op`, `torch._inductor`, `torch._inductor.codegen.rocm.ck_template`, `torch._inductor.codegen.rocm.rocm_kernel`, and `torch._inductor.utils` for the logic in this range. Introduces function `gen_conv_ops_library`, function `torch_layout_to_ck_layouts`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `ck4inductor.grouped_conv_fwd.op`、`torch._inductor`、`torch._inductor.codegen.rocm.ck_template`、`torch._inductor.codegen.rocm.rocm_kernel`、`torch._inductor.utils` 等依赖，为后续逻辑提供基础能力。这里定义了函数`gen_conv_ops_library`、函数`torch_layout_to_ck_layouts`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
    # logically, torch tensors are always NCHW,
    # and channels-last memory layout is visible in the strides
    if V.graph.sizevars.statically_known_equals(torch_layout.stride[-1], 1):
        # when input or output is NCHW
        # NB: torch.conv2d result is always NCHW
        return ["NGCHW", "GKCYX", "NGKHW"]
    elif V.graph.sizevars.statically_known_equals(torch_layout.stride[-3], 1):
        # when input or output or weight is channels-last
        return ["NHWGC", "GKYXC", "NHWGK"]
    else:
        return None


def torch_layout_to_ck_input_layout(torch_layout):
    if V.graph.sizevars.statically_known_equals(torch_layout.stride[-1], 1):
        return "NGCHW"
    elif V.graph.sizevars.statically_known_equals(torch_layout.stride[-3], 1):
        return "NHWGC"
    else:
        return None
````
- **EN**: Introduces function `torch_layout_to_ck_input_layout`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`torch_layout_to_ck_input_layout`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 61-80 / 第 61-80 行
````python


def torch_layout_to_ck_weight_layout(torch_layout):
    if V.graph.sizevars.statically_known_equals(torch_layout.stride[-1], 1):
        return "GKCYX"
    elif V.graph.sizevars.statically_known_equals(torch_layout.stride[-3], 1):
        return "GKYXC"
    else:
        return None


def torch_layout_to_ck_output_layout(torch_layout):
    if V.graph.sizevars.statically_known_equals(torch_layout.stride[-1], 1):
        return "NGKHW"
    elif V.graph.sizevars.statically_known_equals(torch_layout.stride[-3], 1):
        return "NHWGK"
    else:
        return None


````
- **EN**: Introduces function `torch_layout_to_ck_weight_layout`, function `torch_layout_to_ck_output_layout`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`torch_layout_to_ck_weight_layout`、函数`torch_layout_to_ck_output_layout`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 81-100 / 第 81-100 行
````python
class CKGroupedConvFwdTemplate(CKTemplate):
    conv_template = r"""
    {{headers}}
    {{globals}}
    {{instance_definition}}
    extern "C" {
    PT_EXPORT {{kernel_definition}} {
        auto conv = {{instance_type}} {};
        auto invoker = conv.MakeInvoker();

        using ck::index_t;

        constexpr index_t NumDTensor = {{n_d_tensors}};
        constexpr index_t NDimSpatial = {{n_dim_spatial}};
        const std::vector<index_t> FilterSize = { FilterSize_0, FilterSize_1 };
        const std::vector<index_t> InputSize = { InputSize_0, InputSize_1 };
        const std::vector<index_t> ConvolutionStrides = { ConvolutionStrides_0, ConvolutionStrides_1 };
        const std::vector<index_t> Dilations = { Dilations_0, Dilations_1 };
        const std::vector<index_t> LeftPads = { LeftPads_0, LeftPads_1 };
        const std::vector<index_t> RightPads = { RightPads_0, RightPads_1 };
````
- **EN**: Introduces class `CKGroupedConvFwdTemplate`. Initializes or updates values such as `conv_template`.
- **CN**: 这里定义了类`CKGroupedConvFwdTemplate`。初始化或更新了 `conv_template` 等值。

### Lines 101-120 / 第 101-120 行
````python


        auto conv_param = ck::utils::conv::ConvParam {
            NDimSpatial,
            GroupCount,
            NBatch,
            NOutChannels,
            NInChannels,
            FilterSize,
            InputSize,
            ConvolutionStrides,
            Dilations,
            LeftPads,
            RightPads,
        };

        using InLayout  = ck::tensor_layout::convolution::{{input_layout}};
        using WeiLayout = ck::tensor_layout::convolution::{{weight_layout}};
        using OutLayout = ck::tensor_layout::convolution::{{output_layout}};

````
- **EN**: This range continues the implementation of class `CKGroupedConvFwdTemplate`.
- **CN**: 这一段延续了类`CKGroupedConvFwdTemplate` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
        const auto in_g_n_c_wis_desc =
            ck::utils::conv::make_input_host_tensor_descriptor_g_n_c_wis_packed<InLayout>(conv_param);
        const auto wei_g_k_c_xs_desc =
            ck::utils::conv::make_weight_host_tensor_descriptor_g_k_c_xs_packed<WeiLayout>(conv_param);
        const auto out_g_n_k_wos_desc =
            ck::utils::conv::make_output_host_tensor_descriptor_g_n_k_wos_packed<OutLayout>(conv_param);

        const void* p_a = input;
        const void* p_b = weight;
        const std::array<const void*, NumDTensor> p_ds;
        void* p_e = output;
        std::array<index_t, NDimSpatial + 3> a_g_n_c_wis_lengths;
        std::array<index_t, NDimSpatial + 3> a_g_n_c_wis_strides;
        std::array<index_t, NDimSpatial + 3> b_g_k_c_xs_lengths;
        std::array<index_t, NDimSpatial + 3> b_g_k_c_xs_strides;
        std::array<std::array<index_t, NDimSpatial + 3>, NumDTensor> ds_g_n_k_wos_lengths;
        std::array<std::array<index_t, NDimSpatial + 3>, NumDTensor> ds_g_n_k_wos_strides;
        std::array<index_t, NDimSpatial + 3> e_g_n_k_wos_lengths;
        std::array<index_t, NDimSpatial + 3> e_g_n_k_wos_strides;
        std::array<index_t, NDimSpatial> conv_filter_strides;
````
- **EN**: Initializes or updates values such as `ck`, and `std`. This range continues the implementation of class `CKGroupedConvFwdTemplate`.
- **CN**: 初始化或更新了 `ck`、`std` 等值。这一段延续了类`CKGroupedConvFwdTemplate` 的具体实现。

### Lines 141-160 / 第 141-160 行
````python
        std::array<index_t, NDimSpatial> conv_filter_dilations;
        std::array<index_t, NDimSpatial> input_left_pads;
        std::array<index_t, NDimSpatial> input_right_pads;
        const auto a_element_op = PassThrough {};
        const auto b_element_op = PassThrough {};
        const auto cde_element_op = PassThrough {};

        auto copy = [](auto& x, auto& y) { ck::ranges::copy(x, y.begin()); };

        copy(in_g_n_c_wis_desc.GetLengths(), a_g_n_c_wis_lengths);
        copy(in_g_n_c_wis_desc.GetStrides(), a_g_n_c_wis_strides);
        copy(wei_g_k_c_xs_desc.GetLengths(), b_g_k_c_xs_lengths);
        copy(wei_g_k_c_xs_desc.GetStrides(), b_g_k_c_xs_strides);
        copy(out_g_n_k_wos_desc.GetLengths(), e_g_n_k_wos_lengths);
        copy(out_g_n_k_wos_desc.GetStrides(), e_g_n_k_wos_strides);
        copy(conv_param.conv_filter_strides_, conv_filter_strides);
        copy(conv_param.conv_filter_dilations_, conv_filter_dilations);
        copy(conv_param.input_left_pads_, input_left_pads);
        copy(conv_param.input_right_pads_, input_right_pads);

````
- **EN**: Initializes or updates values such as `std`. This range continues the implementation of class `CKGroupedConvFwdTemplate`.
- **CN**: 初始化或更新了 `std` 等值。这一段延续了类`CKGroupedConvFwdTemplate` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
        auto argument = conv.MakeArgument(
            p_a,
            p_b,
            p_ds,
            p_e,
            a_g_n_c_wis_lengths,
            a_g_n_c_wis_strides,
            b_g_k_c_xs_lengths,
            b_g_k_c_xs_strides,
            ds_g_n_k_wos_lengths,
            ds_g_n_k_wos_strides,
            e_g_n_k_wos_lengths,
            e_g_n_k_wos_strides,
            conv_filter_strides,
            conv_filter_dilations,
            input_left_pads,
            input_right_pads,
            a_element_op,
            b_element_op,
            cde_element_op
````
- **EN**: This range continues the implementation of class `CKGroupedConvFwdTemplate`.
- **CN**: 这一段延续了类`CKGroupedConvFwdTemplate` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        );
        if (!conv.IsSupportedArgument(argument)) {
            // we do our best to statically avoid this case in `filter_op`
            std::cerr << "invalid argument for conv instance " << conv.GetTypeString() << std::endl;
            argument.Print();
            return -23;
        }
        if (workspace_size) {
            *workspace_size = conv.GetWorkSpaceSize(&argument);
            return 0;
        }

        if (p_a == nullptr) {
            std::cerr << "p_a is nullptr" << std::endl;
            return -1;
        }
        if (p_b == nullptr) {
            std::cerr << "p_b is nullptr" << std::endl;
            return -1;
        }
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`. This range continues the implementation of class `CKGroupedConvFwdTemplate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `std` 等值。这一段延续了类`CKGroupedConvFwdTemplate` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
        if (p_e == nullptr) {
            std::cerr << "p_e is nullptr" << std::endl;
            return -1;
        }

        // when debugging, do time kernel to serialize launches
        auto stream_config = StreamConfig{stream, /* time kernel */ false, /* log level */ 0};

        if (workspace != nullptr) {
            conv.SetWorkSpacePointer(&argument, workspace, stream_config);
        }

        // run the kernel
        float elapsed_time = invoker.Run(argument, stream_config);
        return 0;
    } // kernel definition
    } // extern C

    #ifdef GENERATE_CK_STANDALONE_RUNNER
    int main(int argc, char** argv) {
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `std`. This range continues the implementation of class `CKGroupedConvFwdTemplate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `std` 等值。这一段延续了类`CKGroupedConvFwdTemplate` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        (void) argc;
        (void) argv;
        return 0;
    }
    #endif // GENERATE_CK_STANDALONE_RUNNER
"""

    def globals(self) -> IndentedBuffer:
        res = super().globals()
        res.splice(
            """
                // CK conv globals

                using NWC   = ck::tensor_layout::convolution::NWC;
                using NHWC  = ck::tensor_layout::convolution::NHWC;
                using NDHWC = ck::tensor_layout::convolution::NDHWC;

                using KXC   = ck::tensor_layout::convolution::KXC;
                using KYXC  = ck::tensor_layout::convolution::KYXC;
                using KZYXC = ck::tensor_layout::convolution::KZYXC;
````
- **EN**: Introduces function `globals`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `res`.
- **CN**: 这里定义了函数`globals`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `res` 等值。

### Lines 241-260 / 第 241-260 行
````python

                using NWK   = ck::tensor_layout::convolution::NWK;
                using NHWK  = ck::tensor_layout::convolution::NHWK;
                using NDHWK = ck::tensor_layout::convolution::NDHWK;

                using GNWC   = ck::tensor_layout::convolution::GNWC;
                using GNHWC  = ck::tensor_layout::convolution::GNHWC;
                using GNDHWC = ck::tensor_layout::convolution::GNDHWC;

                using GKXC   = ck::tensor_layout::convolution::GKXC;
                using GKYXC  = ck::tensor_layout::convolution::GKYXC;
                using GKZYXC = ck::tensor_layout::convolution::GKZYXC;

                using GKCX   = ck::tensor_layout::convolution::GKCX;
                using GKCYX  = ck::tensor_layout::convolution::GKCYX;
                using GKCZYX = ck::tensor_layout::convolution::GKCZYX;

                using GNWK   = ck::tensor_layout::convolution::GNWK;
                using GNHWK  = ck::tensor_layout::convolution::GNHWK;
                using GNDHWK = ck::tensor_layout::convolution::GNDHWK;
````
- **EN**: This range continues the implementation of function `CKGroupedConvFwdTemplate.globals`.
- **CN**: 这一段延续了函数`CKGroupedConvFwdTemplate.globals` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python

                using NGKW   = ck::tensor_layout::convolution::NGKW;
                using NGKHW  = ck::tensor_layout::convolution::NGKHW;
                using NGKDHW = ck::tensor_layout::convolution::NGKDHW;

                using NWGC   = ck::tensor_layout::convolution::NWGC;
                using NHWGC  = ck::tensor_layout::convolution::NHWGC;
                using NDHWGC = ck::tensor_layout::convolution::NDHWGC;

                using KXGC   = ck::tensor_layout::convolution::KXGC;
                using KYXGC  = ck::tensor_layout::convolution::KYXGC;
                using KZYXGC = ck::tensor_layout::convolution::KZYXGC;

                using NWGK   = ck::tensor_layout::convolution::NWGK;
                using NHWGK  = ck::tensor_layout::convolution::NHWGK;
                using NDHWGK = ck::tensor_layout::convolution::NDHWGK;

                using NGCW   = ck::tensor_layout::convolution::NGCW;
                using NGCHW  = ck::tensor_layout::convolution::NGCHW;
                using NGCDHW = ck::tensor_layout::convolution::NGCDHW;
````
- **EN**: This range continues the implementation of function `CKGroupedConvFwdTemplate.globals`.
- **CN**: 这一段延续了函数`CKGroupedConvFwdTemplate.globals` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python

                using G_K    = ck::tensor_layout::convolution::G_K;

                using BlockGemmPipelineScheduler = ck::BlockGemmPipelineScheduler;
                using GemmSpecialization = ck::tensor_operation::device::GemmSpecialization;
                using BlockGemmPipelineVersion = ck::BlockGemmPipelineVersion;

                using ConvolutionForwardSpecialization = ck::tensor_operation::device::ConvolutionForwardSpecialization;

                using OutElementOp = PassThrough;

                namespace ck {
                namespace utils {
                namespace conv {

                ConvParam::ConvParam(ck::index_t n_dim,
                                    ck::index_t group_count,
                                    ck::index_t n_batch,
                                    ck::index_t n_out_channels,
                                    ck::index_t n_in_channels,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `ConvParam`, and `ck`. This range continues the implementation of function `CKGroupedConvFwdTemplate.globals`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `ConvParam`、`ck` 等值。这一段延续了函数`CKGroupedConvFwdTemplate.globals` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
                                    const std::vector<ck::index_t>& filters_len,
                                    const std::vector<ck::index_t>& input_len,
                                    const std::vector<ck::index_t>& strides,
                                    const std::vector<ck::index_t>& dilations,
                                    const std::vector<ck::index_t>& left_pads,
                                    const std::vector<ck::index_t>& right_pads)
                    : num_dim_spatial_(static_cast<ck::long_index_t>(n_dim)),
                    G_(static_cast<ck::long_index_t>(group_count)),
                    N_(static_cast<ck::long_index_t>(n_batch)),
                    K_(static_cast<ck::long_index_t>(n_out_channels)),
                    C_(static_cast<ck::long_index_t>(n_in_channels)),
                    filter_spatial_lengths_(num_dim_spatial_),
                    input_spatial_lengths_(num_dim_spatial_),
                    output_spatial_lengths_(num_dim_spatial_),
                    conv_filter_strides_(num_dim_spatial_),
                    conv_filter_dilations_(num_dim_spatial_),
                    input_left_pads_(num_dim_spatial_),
                    input_right_pads_(num_dim_spatial_)
                {
                    if(static_cast<ck::index_t>(filter_spatial_lengths_.size()) != num_dim_spatial_ ||
````
- **EN**: This range continues the implementation of function `CKGroupedConvFwdTemplate.globals`.
- **CN**: 这一段延续了函数`CKGroupedConvFwdTemplate.globals` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
                    static_cast<ck::index_t>(input_spatial_lengths_.size()) != num_dim_spatial_ ||
                    static_cast<ck::index_t>(conv_filter_strides_.size()) != num_dim_spatial_ ||
                    static_cast<ck::index_t>(conv_filter_dilations_.size()) != num_dim_spatial_ ||
                    static_cast<ck::index_t>(input_left_pads_.size()) != num_dim_spatial_ ||
                    static_cast<ck::index_t>(input_right_pads_.size()) != num_dim_spatial_)
                    {
                        throw(
                            std::runtime_error("ConvParam::ConvParam: "
                                            "parameter size is different from number of declared dimensions!"));
                    }

                    for(ck::index_t i = 0; i < num_dim_spatial_; ++i)
                    {
                        filter_spatial_lengths_[i] = static_cast<ck::long_index_t>(filters_len[i]);
                        input_spatial_lengths_[i]  = static_cast<ck::long_index_t>(input_len[i]);
                        conv_filter_strides_[i]    = static_cast<ck::long_index_t>(strides[i]);
                        conv_filter_dilations_[i]  = static_cast<ck::long_index_t>(dilations[i]);
                        input_left_pads_[i]        = static_cast<ck::long_index_t>(left_pads[i]);
                        input_right_pads_[i]       = static_cast<ck::long_index_t>(right_pads[i]);

````
- **EN**: Initializes or updates values such as `std`. This range continues the implementation of function `CKGroupedConvFwdTemplate.globals`.
- **CN**: 初始化或更新了 `std` 等值。这一段延续了函数`CKGroupedConvFwdTemplate.globals` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
                        // XEff = (X - 1) * conv_dilation_w + 1;
                        // Wo = (Wi + in_left_pad_w + in_right_pad_w - XEff) / conv_stride_w + 1;
                        const ck::long_index_t x_eff =
                            (filter_spatial_lengths_[i] - 1) * conv_filter_dilations_[i] + 1;

                        output_spatial_lengths_[i] =
                            (input_spatial_lengths_[i] + input_left_pads_[i] + input_right_pads_[i] - x_eff) /
                                conv_filter_strides_[i] +
                            1;
                    }
                }

                } // namespace conv
                } // namespace utils
                } // namespace ck

                const std::vector<std::size_t>& HostTensorDescriptor::GetLengths() const { return mLens; }
                const std::vector<std::size_t>& HostTensorDescriptor::GetStrides() const { return mStrides; }
                std::size_t HostTensorDescriptor::GetNumOfDimension() const { return mLens.size(); }
                void HostTensorDescriptor::CalculateStrides() {
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `std`. This range continues the implementation of function `CKGroupedConvFwdTemplate.globals`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `std` 等值。这一段延续了函数`CKGroupedConvFwdTemplate.globals` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
                    mStrides.clear();
                    mStrides.resize(mLens.size(), 0);
                    if(mStrides.empty())
                        return;

                    mStrides.back() = 1;
                    std::partial_sum(
                        mLens.rbegin(), mLens.rend() - 1, mStrides.rbegin() + 1, std::multiplies<std::size_t>());
                }
            """
        )
        return res

    def header(self) -> IndentedBuffer:
        res = super().header()
        res.splice(
            """
                // CK conv headers

                #include "ck/tensor_operation/gpu/device/impl/device_grouped_conv_fwd_multiple_abd_xdl_cshuffle_v3.hpp"
````
- **EN**: Introduces function `header`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `std`, and `res`.
- **CN**: 这里定义了函数`header`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `std`、`res` 等值。

### Lines 381-400 / 第 381-400 行
````python
                #include "ck/tensor_operation/gpu/device/convolution_forward_specialization.hpp"
                #include "ck/tensor_operation/gpu/device/gemm_specialization.hpp"

                #include "ck/library/utility/convolution_parameter.hpp"
                #include "ck/library/utility/convolution_host_tensor_descriptor_helper.hpp"
            """
        )
        return res

    @staticmethod
    def add_ck_conv_choices(
        choices,
        layout,
        input_nodes,
        *,
        stride,
        padding,
        dilation,
        groups,
        n_spatial_dimensions,
````
- **EN**: Introduces function `add_ck_conv_choices`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`add_ck_conv_choices`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 401-420 / 第 401-420 行
````python
    ):
        template = CKGroupedConvFwdTemplate(
            input_nodes,
            layout,
            stride=stride,
            padding=padding,
            dilation=dilation,
            groups=groups,
            n_spatial_dimensions=n_spatial_dimensions,
        )
        ops = template.gen_ops()
        for op in ops:
            template.maybe_append_choice(
                choices,
                op=op,
            )

    def __init__(
        self,
        input_nodes,
````
- **EN**: Introduces function `__init__`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template`, `stride`, `padding`, `dilation`, `groups`, `n_spatial_dimensions`, and `...+2`.
- **CN**: 这里定义了函数`__init__`。包含分支、循环或上下文管理等控制流。初始化或更新了 `template`、`stride`、`padding`、`dilation`、`groups`、`n_spatial_dimensions`、`另有2项` 等值。

### Lines 421-440 / 第 421-440 行
````python
        layout,
        *,
        stride,
        padding,
        dilation,
        groups,
        n_spatial_dimensions,
    ):
        super().__init__(
            "ck_conv_template",
            input_nodes,
            layout,
        )
        self.stride = stride
        self.padding = padding
        self.dilation = dilation
        self.groups = groups
        self.n_spatial_dimensions = n_spatial_dimensions

    def filter_op(self, op: "CKGroupedConvFwdOp"):  # type: ignore[name-defined]
````
- **EN**: Introduces function `filter_op`.
- **CN**: 这里定义了函数`filter_op`。

### Lines 441-460 / 第 441-460 行
````python
        metas = [
            T.get_layout()
            for T in [*self.input_nodes, self.output_node]
            if T is not None
        ]
        X_meta = metas[0]
        W_meta = metas[1]
        Y_meta = metas[-1]
        # disable the instance if dtypes don't match
        if op.a_element_dtype != self._TORCH_DTYPE_TO_CK[X_meta.dtype]:
            return None
        if op.b_element_dtype != self._TORCH_DTYPE_TO_CK[W_meta.dtype]:
            return None
        if op.e_element_dtype != self._TORCH_DTYPE_TO_CK[Y_meta.dtype]:
            return None
        # disable the instance if layouts don't match
        if op.a_layout != torch_layout_to_ck_input_layout(X_meta):
            return None
        if op.b_layout != torch_layout_to_ck_weight_layout(W_meta):
            return None
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `metas`, `X_meta`, `W_meta`, and `Y_meta`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `metas`、`X_meta`、`W_meta`、`Y_meta` 等值。

### Lines 461-480 / 第 461-480 行
````python
        if op.e_layout != torch_layout_to_ck_output_layout(Y_meta):
            return None
        # disable the instance if number of spatial dimensions doesn't match
        if op.n_dim_spatial != self.n_spatial_dimensions:
            return None
        # disable 1x1 and odd-channels conv specializations for now
        if "Default" not in op.conv_forward_specialization:
            return None
        return op

    def gen_ops(self):
        unfiltered_instances = gen_conv_ops_library()

        filtered_instances = list(
            filter(lambda op: self.filter_op(op), unfiltered_instances)
        )
        # NB: when using a fixed list order, most likely we will pick the subset of instances
        # which are very similar to each other. Randomizing the choice seems to solve this.
        random.seed(-11)
        chosen_instances = (
````
- **EN**: Introduces function `gen_ops`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unfiltered_instances`, `filtered_instances`, and `chosen_instances`.
- **CN**: 这里定义了函数`gen_ops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `unfiltered_instances`、`filtered_instances`、`chosen_instances` 等值。

### Lines 481-500 / 第 481-500 行
````python
            random.sample(
                filtered_instances,
                min(len(filtered_instances), config.rocm.ck_max_profiling_configs),
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

    def emit_ck_instance(self, op: "CKGroupedConvFwdOp") -> tuple[str, str]:  # type: ignore[name-defined]
        # The Jinja template for generating a C++ type alias *definition* for a Universal GEMM instance
        template_definition = r"""
    // Gemm operator {{operation_name}}
    using Operation_{{operation_name}} =
        ck::tensor_operation::device::DeviceGroupedConvFwdMultipleABD_Xdl_CShuffle_V3<
````
- **EN**: Introduces function `emit_ck_instance`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`emit_ck_instance`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 501-520 / 第 501-520 行
````python
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
                    arg = f"/* {field_name} */ Tuple<{tuple_elements}>"
                else:  # tile shape
                    arg = f"/* {field_name} */ S<{tuple_elements}>"
                # pyrefly: ignore [bad-argument-type]
                template_params.append(arg)
            else:
                if field_value is not None:
                    # pyrefly: ignore [bad-argument-type]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template_type`, `template_params`, `tuple_elements`, `arg`, and `else`. This range continues the implementation of function `CKGroupedConvFwdTemplate.emit_ck_instance`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `template_type`、`template_params`、`tuple_elements`、`arg`、`else` 等值。这一段延续了函数`CKGroupedConvFwdTemplate.emit_ck_instance` 的具体实现。

### Lines 521-540 / 第 521-540 行
````python
                    template_params.append(f"/* {field_name} */ {field_value}")
        return self._template_from_string(template_definition).render(
            operation_name=op.name(),
            template_params=(",\n" + 12 * " ").join(template_params),
        ), self._template_from_string(template_type).render(operation_name=op.name())

    def render(  # type: ignore[override]
        self,
        kernel: ROCmTemplateKernel,
        op: "CKGroupedConvFwdOp",  # type: ignore[name-defined]
        **kwargs,
    ) -> str:
        template_buffer_node = kwargs.get("template_buffer_node")
        if template_buffer_node is not None:
            self.output_node = template_buffer_node
        X, W = self.input_nodes[0], self.input_nodes[1]
        Y = self.output_node
        Bias = self.input_nodes[2] if 3 == len(self.input_nodes) else None

        op = copy.deepcopy(op)
````
- **EN**: Introduces function `render`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `operation_name`, `template_params`, `kernel`, `op`, `template_buffer_node`, `Y`, and `...+1`.
- **CN**: 这里定义了函数`render`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `operation_name`、`template_params`、`kernel`、`op`、`template_buffer_node`、`Y`、`另有1项` 等值。

### Lines 541-560 / 第 541-560 行
````python

        instance_definition, instance_type = self.emit_ck_instance(op)

        size_arg_strs = [
            "GroupCount",
            "NBatch",
            "NOutChannels",
            "NInChannels",
            "FilterSize_0",
            "FilterSize_1",
            "InputSize_0",
            "InputSize_1",
            "ConvolutionStrides_0",
            "ConvolutionStrides_1",
            "Dilations_0",
            "Dilations_1",
            "LeftPads_0",
            "LeftPads_1",
            "RightPads_0",
            "RightPads_1",
````
- **EN**: Initializes or updates values such as `size_arg_strs`. This range continues the implementation of function `CKGroupedConvFwdTemplate.render`.
- **CN**: 初始化或更新了 `size_arg_strs` 等值。这一段延续了函数`CKGroupedConvFwdTemplate.render` 的具体实现。

### Lines 561-580 / 第 561-580 行
````python
        ]

        return self._template_from_string(self.conv_template).render(
            headers=self.header().getvalue(),
            globals=self.globals().getvalue(),
            instance_definition=instance_definition,
            instance_type=instance_type,
            kernel_definition=kernel.def_kernel(
                inputs=[X, W, Bias] if Bias is not None else [X, W],
                outputs=[Y],
                names_str="input, weight, bias, output"
                if Bias is not None
                else "input, weight, output",
                size_args=[f"int32_t {arg}" for arg in size_arg_strs],
            ),
            n_d_tensors=1 if Bias is not None else 0,
            n_dim_spatial=self.n_spatial_dimensions,
            input_layout=op.a_layout,
            weight_layout=op.b_layout,
            output_layout=op.e_layout,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `headers`, `globals`, `instance_definition`, `instance_type`, `kernel_definition`, `inputs`, and `...+8`. This range continues the implementation of function `CKGroupedConvFwdTemplate.render`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `headers`、`globals`、`instance_definition`、`instance_type`、`kernel_definition`、`inputs`、`另有8项` 等值。这一段延续了函数`CKGroupedConvFwdTemplate.render` 的具体实现。

### Lines 581-600 / 第 581-600 行
````python
        )

    def size_args(self):
        x, w = self.input_nodes[0], self.input_nodes[1]
        y = self.output_node

        group_count = self.groups
        n_batch = x.shape[0]  # type: ignore[index]
        n_out_channels = y.shape[1]  # type: ignore[index]
        n_in_channels = x.shape[1]  # type: ignore[index]

        filter_size_0, filter_size_1 = w.shape[2:4]  # type: ignore[index]
        input_size_0, input_size_1 = x.shape[2:4]  # type: ignore[index]
        convolution_strides_0, convolution_strides_1 = self.stride
        dilations_0, dilations_1 = self.dilation
        left_pads_0, left_pads_1 = self.padding
        right_pads_0, right_pads_1 = self.padding

        return (
            group_count,
````
- **EN**: Introduces function `size_args`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `y`, `group_count`, `n_batch`, `n_out_channels`, and `n_in_channels`.
- **CN**: 这里定义了函数`size_args`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `y`、`group_count`、`n_batch`、`n_out_channels`、`n_in_channels` 等值。

### Lines 601-620 / 第 601-620 行
````python
            n_batch,
            n_out_channels,
            n_in_channels,
            filter_size_0,
            filter_size_1,
            input_size_0,
            input_size_1,
            convolution_strides_0,
            convolution_strides_1,
            dilations_0,
            dilations_1,
            left_pads_0,
            left_pads_1,
            right_pads_0,
            right_pads_1,
        )

    @override
    def get_runtime_arg_info(self) -> list[ArgInfo]:
        return []
````
- **EN**: Introduces function `get_runtime_arg_info`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_runtime_arg_info`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 621-627 / 第 621-627 行
````python

    @override
    def get_runtime_arg_values(self, **kwargs: Any) -> list[Any]:
        """
        Helper method to retrieve runtime args from generate kwargs
        """
        return []
````
- **EN**: Introduces function `get_runtime_arg_values`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_runtime_arg_values`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CKGroupedConvFwdTemplate`  
  **CN**: 主要类：`CKGroupedConvFwdTemplate`
- **EN**: Primary functions: `torch_layout_to_ck_layouts`, `torch_layout_to_ck_input_layout`, `torch_layout_to_ck_weight_layout`, and `torch_layout_to_ck_output_layout`  
  **CN**: 主要函数：`torch_layout_to_ck_layouts`、`torch_layout_to_ck_input_layout`、`torch_layout_to_ck_weight_layout`、`torch_layout_to_ck_output_layout`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `logging`, `random`, `typing`
- **Third-party / 第三方**: `typing_extensions`, `ck4inductor`, `ck4inductor.grouped_conv_fwd.gen_instances`, `ck4inductor.grouped_conv_fwd.op`
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.virtualized`, `.rocm_template`, `torch._inductor`, `torch._inductor.codegen.rocm.ck_template`, `torch._inductor.codegen.rocm.rocm_kernel`, `torch._inductor.utils`

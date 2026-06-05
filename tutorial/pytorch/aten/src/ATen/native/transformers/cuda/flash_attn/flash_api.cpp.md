# flash_api.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/transformers/cuda/flash_attn/flash_api.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Implements executable backend logic for FlashAttention CUDA specialization, centered on flash api with emphasis on attention computation.
- 用途（中文）: 实现可执行的后端逻辑，属于FlashAttention CUDA 特化实现，核心主题是flash api，重点关注注意力计算。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-40

```cpp
   1: /******************************************************************************
   2:  * Copyright (c) 2024, Tri Dao.
   3:  ******************************************************************************/
   4: #include <c10/core/ScalarType.h>
   5: #include <c10/core/DeviceType.h>
   6: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   7: 
   8: #include <cstdint>
   9: #include <tuple>
  10: 
  11: 
  12: #ifdef USE_FLASH_ATTENTION
  13: 
  14: #include <ATen/core/Tensor.h>
  15: #include <ATen/cuda/CUDAContext.h>
  16: #include <c10/cuda/CUDAGuard.h>
  17: #include <ATen/cuda/CUDAGraphsUtils.cuh>
  18: 
  19: #ifndef AT_PER_OPERATOR_HEADERS
  20: #include <ATen/Functions.h>
  21: #include <ATen/NativeFunctions.h>
  22: #else
  23: #include <ATen/ops/empty.h>
  24: #include <ATen/ops/empty_like.h>
  25: #include <ATen/ops/zeros_like.h>
  26: #include <ATen/ops/reshape.h>
  27: #include <ATen/ops/scalar_tensor.h>
  28: #include <ATen/ops/sum.h>
  29: #include <ATen/ops/slice.h>
  30: #include <ATen/ops/narrow.h>
  31: #include <ATen/ops/pad.h>
  32: #include <ATen/ops/zeros.h>
  33: #endif
  34: 
  35: 
  36: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wextra-semi")
  37: #include <cutlass/numeric_types.h>
  38: C10_DIAGNOSTIC_POP()
  39: 
  40: 
```
- L1: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L2: Documents the nearby logic: Copyright (c) 2024, Tri Dao. / 说明附近逻辑的作用：Copyright (c) 2024, Tri Dao.
- L3: Documents the nearby logic: / / 说明附近逻辑的作用：/
- L4: Includes `c10/core/ScalarType.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/core/ScalarType.h`，用于 c10 核心运行时、工具或分发元数据。
- L5: Includes `c10/core/DeviceType.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/core/DeviceType.h`，用于 c10 核心运行时、工具或分发元数据。
- L6: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L8: Includes `cstdint` for standard-library or external support. / 引入 `cstdint`，用于标准库或外部支持。
- L9: Includes `tuple` for standard-library or external support. / 引入 `tuple`，用于标准库或外部支持。
- L12: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L14: Includes `ATen/core/Tensor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/core/Tensor.h`，为 ATen 的张量/算子基础设施提供支持。
- L15: Includes `ATen/cuda/CUDAContext.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/CUDAContext.h`，为 ATen 的张量/算子基础设施提供支持。
- L16: Includes `c10/cuda/CUDAGuard.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/cuda/CUDAGuard.h`，用于 c10 核心运行时、工具或分发元数据。
- L17: Includes `ATen/cuda/CUDAGraphsUtils.cuh` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/CUDAGraphsUtils.cuh`，为 ATen 的张量/算子基础设施提供支持。
- L19: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L20: Includes `ATen/Functions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Functions.h`，为 ATen 的张量/算子基础设施提供支持。
- L21: Includes `ATen/NativeFunctions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NativeFunctions.h`，为 ATen 的张量/算子基础设施提供支持。
- L22: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L23: Includes `ATen/ops/empty.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty.h`，为 ATen 的张量/算子基础设施提供支持。
- L24: Includes `ATen/ops/empty_like.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty_like.h`，为 ATen 的张量/算子基础设施提供支持。
- L25: Includes `ATen/ops/zeros_like.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/zeros_like.h`，为 ATen 的张量/算子基础设施提供支持。
- L26: Includes `ATen/ops/reshape.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/reshape.h`，为 ATen 的张量/算子基础设施提供支持。
- L27: Includes `ATen/ops/scalar_tensor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/scalar_tensor.h`，为 ATen 的张量/算子基础设施提供支持。
- L28: Includes `ATen/ops/sum.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/sum.h`，为 ATen 的张量/算子基础设施提供支持。
- L29: Includes `ATen/ops/slice.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/slice.h`，为 ATen 的张量/算子基础设施提供支持。
- L30: Includes `ATen/ops/narrow.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/narrow.h`，为 ATen 的张量/算子基础设施提供支持。
- L31: Includes `ATen/ops/pad.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/pad.h`，为 ATen 的张量/算子基础设施提供支持。
- L32: Includes `ATen/ops/zeros.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/zeros.h`，为 ATen 的张量/算子基础设施提供支持。
- L33: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L36: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L37: Includes `cutlass/numeric_types.h` for standard-library or external support. / 引入 `cutlass/numeric_types.h`，用于标准库或外部支持。
- L38: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 41-80

```cpp
  41: #include <flash.h>
  42: #include <namespace_config.h>
  43: #include <static_switch.h>
  44: #include <ATen/native/transformers/cuda/flash_attn/flash_api.h>
  45: 
  46: #include <c10/util/Exception.h>
  47: 
  48: namespace FLASH_NAMESPACE {
  49: 
  50: #define CHECK_DEVICE(x) TORCH_CHECK(x.is_cuda(), #x " must be on CUDA")
  51: #define CHECK_SHAPE(x, ...) TORCH_CHECK(x.sizes() == at::IntArrayRef({__VA_ARGS__}), #x " must have shape (" #__VA_ARGS__ ")")
  52: #define CHECK_CONTIGUOUS(x) TORCH_CHECK(x.is_contiguous(), #x " must be contiguous")
  53: 
  54: 
  55: void set_params_fprop(Flash_fwd_params &params,
  56:                       // sizes
  57:                       const size_t b,
  58:                       const size_t seqlen_q,
  59:                       const size_t seqlen_k,
  60:                       const size_t seqlen_q_rounded,
  61:                       const size_t seqlen_k_rounded,
  62:                       const size_t h,
  63:                       const size_t h_k,
  64:                       const size_t d,
  65:                       const size_t d_rounded,
  66:                       // device pointers
  67:                       const at::Tensor q,
  68:                       const at::Tensor k,
  69:                       const at::Tensor v,
  70:                       at::Tensor out,
  71:                       void *cu_seqlens_q_d,
  72:                       void *cu_seqlens_k_d,
  73:                       void *seqused_k,
  74:                       void *p_d,
  75:                       void *softmax_lse_d,
  76:                       float p_dropout,
  77:                       float softmax_scale,
  78:                       int window_size_left,
  79:                       int window_size_right,
  80:                       const float softcap,
```
- L41: Includes `flash.h` for standard-library or external support. / 引入 `flash.h`，用于标准库或外部支持。
- L42: Includes `namespace_config.h` for standard-library or external support. / 引入 `namespace_config.h`，用于标准库或外部支持。
- L43: Includes `static_switch.h` for standard-library or external support. / 引入 `static_switch.h`，用于标准库或外部支持。
- L44: Includes `ATen/native/transformers/cuda/flash_attn/flash_api.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/transformers/cuda/flash_attn/flash_api.h`，为 ATen 的张量/算子基础设施提供支持。
- L46: Includes `c10/util/Exception.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/util/Exception.h`，用于 c10 核心运行时、工具或分发元数据。
- L48: Opens namespace `FLASH_NAMESPACE` to scope the following declarations. / 打开命名空间 `FLASH_NAMESPACE`，为后续声明限定作用域。
- L50: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L51: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L52: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L55: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L56: Documents the nearby logic: sizes / 说明附近逻辑的作用：sizes
- L57: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L58: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L59: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L60: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L61: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L62: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L63: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L64: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L65: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L66: Documents the nearby logic: device pointers / 说明附近逻辑的作用：device pointers
- L67: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L68: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L69: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L70: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L71: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L72: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L73: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L74: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L75: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L76: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L77: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L78: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L79: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L80: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 81-120

```cpp
  81:                       bool seqlenq_ngroups_swapped=false,
  82:                       const bool unpadded_lse=false) {
  83: 
  84:     // Reset the parameters
  85:     params = {};
  86: 
  87:     params.is_bf16 = q.dtype() == at::kBFloat16;
  88: 
  89:     // Set the pointers and strides.
  90:     params.q_ptr = q.data_ptr();
  91:     params.k_ptr = k.data_ptr();
  92:     params.v_ptr = v.data_ptr();
  93:     // All stride are in elements, not bytes.
  94:     params.q_row_stride = q.stride(-3);
  95:     params.k_row_stride = k.stride(-3);
  96:     params.v_row_stride = v.stride(-3);
  97:     params.q_head_stride = q.stride(-2);
  98:     params.k_head_stride = k.stride(-2);
  99:     params.v_head_stride = v.stride(-2);
 100:     params.o_ptr = out.data_ptr();
 101:     params.o_row_stride = out.stride(-3);
 102:     params.o_head_stride = out.stride(-2);
 103: 
 104:     if (cu_seqlens_q_d == nullptr) {
 105:         params.q_batch_stride = q.stride(0);
 106:         params.k_batch_stride = k.stride(0);
 107:         params.v_batch_stride = v.stride(0);
 108:         params.o_batch_stride = out.stride(0);
 109:         if (seqlenq_ngroups_swapped) {
 110:              params.q_batch_stride *= seqlen_q;
 111:              params.o_batch_stride *= seqlen_q;
 112:         }
 113:     }
 114: 
 115:     params.cu_seqlens_q = static_cast<int *>(cu_seqlens_q_d);
 116:     params.cu_seqlens_k = static_cast<int *>(cu_seqlens_k_d);
 117:     params.seqused_k = static_cast<int *>(seqused_k);
 118: 
 119:     // P = softmax(QK^T)
 120:     params.p_ptr = p_d;
```
- L81: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L82: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L84: Documents the nearby logic: Reset the parameters / 说明附近逻辑的作用：Reset the parameters
- L85: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L87: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L89: Documents the nearby logic: Set the pointers and strides. / 说明附近逻辑的作用：Set the pointers and strides.
- L90: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L91: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L92: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L93: Documents the nearby logic: All stride are in elements, not bytes. / 说明附近逻辑的作用：All stride are in elements, not bytes.
- L94: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L95: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L96: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L97: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L98: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L99: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L100: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L101: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L102: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L104: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L105: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L106: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L107: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L108: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L109: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L110: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L111: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L112: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L113: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L115: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L116: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L117: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L119: Documents the nearby logic: P = softmax(QK^T) / 说明附近逻辑的作用：P = softmax(QK^T)
- L120: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 121-160

```cpp
 121: 
 122:     // Softmax sum
 123:     params.softmax_lse_ptr = softmax_lse_d;
 124: 
 125:     // Set the dimensions.
 126:     params.b = b;
 127:     params.h = h;
 128:     params.h_k = h_k;
 129:     params.h_h_k_ratio = h / h_k;
 130:     params.seqlen_q = seqlen_q;
 131:     params.seqlen_k = seqlen_k;
 132:     params.seqlen_q_rounded = seqlen_q_rounded;
 133:     params.seqlen_k_rounded = seqlen_k_rounded;
 134:     params.d = d;
 135:     params.d_rounded = d_rounded;
 136: 
 137:     // Set the different scale values.
 138:     #ifdef FLASHATTENTION_DISABLE_SOFTCAP
 139:         TORCH_CHECK(softcap <= 0.0, "This flash attention build does not support softcap.");
 140:     #endif
 141:     if (softcap > 0.0) {
 142:         params.softcap = softmax_scale / softcap;
 143:         params.scale_softmax = softcap;
 144:         params.scale_softmax_log2 = softcap * M_LOG2E;
 145:     } else{
 146:         // Remove potential NaN
 147:         params.softcap = 0.0;
 148:         params.scale_softmax = softmax_scale;
 149:         params.scale_softmax_log2 = softmax_scale * M_LOG2E;
 150:     }
 151: 
 152:     // Set this to probability of keeping an element to simplify things.
 153:     params.p_dropout = 1.f - p_dropout;
 154:     // Convert p from float to int so we don't have to convert the random uint to float to compare.
 155:     // [Minor] We want to round down since when we do the comparison we use <= instead of <
 156:     // params.p_dropout_in_uint = uint32_t(std::floor(params.p_dropout * 4294967295.0));
 157:     // params.p_dropout_in_uint16_t = uint16_t(std::floor(params.p_dropout * 65535.0));
 158:     params.p_dropout_in_uint8_t = static_cast<uint8_t>(std::floor(params.p_dropout * 255.0));
 159:     params.rp_dropout = 1.f / params.p_dropout;
 160:     params.scale_softmax_rp_dropout = params.rp_dropout * params.scale_softmax;
```
- L122: Documents the nearby logic: Softmax sum / 说明附近逻辑的作用：Softmax sum
- L123: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L125: Documents the nearby logic: Set the dimensions. / 说明附近逻辑的作用：Set the dimensions.
- L126: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L127: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L128: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L129: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L130: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L131: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L132: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L133: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L134: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L135: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L137: Documents the nearby logic: Set the different scale values. / 说明附近逻辑的作用：Set the different scale values.
- L138: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L139: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L140: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L141: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L142: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L143: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L144: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L145: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L146: Documents the nearby logic: Remove potential NaN / 说明附近逻辑的作用：Remove potential NaN
- L147: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L148: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L149: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L150: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L152: Documents the nearby logic: Set this to probability of keeping an element to simplify things. / 说明附近逻辑的作用：Set this to probability of keeping an element to simplify things.
- L153: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L154: Documents the nearby logic: Convert p from float to int so we don't have to convert the random uint to float to compare. / 说明附近逻辑的作用：Convert p from float to int so we don't have to convert the random uint to float to compare.
- L155: Documents the nearby logic: [Minor] We want to round down since when we do the comparison we use <= instead of < / 说明附近逻辑的作用：[Minor] We want to round down since when we do the comparison we use <= instead of <
- L156: Documents the nearby logic: params.p_dropout_in_uint = uint32_t(std::floor(params.p_dropout * 4294967295.0)); / 说明附近逻辑的作用：params.p_dropout_in_uint = uint32_t(std::floor(params.p_dropout * 4294967295.0));
- L157: Documents the nearby logic: params.p_dropout_in_uint16_t = uint16_t(std::floor(params.p_dropout * 65535.0)); / 说明附近逻辑的作用：params.p_dropout_in_uint16_t = uint16_t(std::floor(params.p_dropout * 65535.0));
- L158: Declares function `floor` as part of this file's callable surface. / 声明函数 `floor`，作为本文件可调用接口的一部分。
- L159: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L160: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 161-200

```cpp
 161:     TORCH_CHECK(p_dropout < 1.f);
 162:     #ifdef FLASHATTENTION_DISABLE_DROPOUT
 163:         TORCH_CHECK(p_dropout == 0.0f, "This flash attention build does not support dropout.");
 164:     #endif
 165: 
 166:     // Causal is the special case where window_size_right == 0 and window_size_left < 0.
 167:     // Local is the more general case where window_size_right >= 0 or window_size_left >= 0.
 168:     params.is_causal = window_size_left < 0 && window_size_right == 0;
 169: 
 170:     if (window_size_left < 0 && window_size_right >= 0) { window_size_left = seqlen_k; }
 171:     if (window_size_left >= 0 && window_size_right < 0) { window_size_right = seqlen_k; }
 172:     params.window_size_left = window_size_left;
 173:     params.window_size_right = window_size_right;
 174: 
 175:     #ifdef FLASHATTENTION_DISABLE_LOCAL
 176:         TORCH_CHECK(params.is_causal || (window_size_left < 0 && window_size_right < 0),
 177:             "This flash attention build does not support local attention.");
 178:     #endif
 179: 
 180:     params.is_seqlens_k_cumulative = true;
 181: 
 182:     #ifdef FLASHATTENTION_DISABLE_UNEVEN_K
 183:         TORCH_CHECK(d == d_rounded, "This flash attention build does not support headdim not being a multiple of 32.");
 184:     #endif
 185:     params.unpadded_lse = unpadded_lse;
 186:     params.seqlenq_ngroups_swapped = seqlenq_ngroups_swapped;
 187: }
 188: 
 189: void set_params_dgrad(Flash_bwd_params &params,
 190:                       // sizes
 191:                       const size_t b,
 192:                       const size_t seqlen_q,
 193:                       const size_t seqlen_k,
 194:                       const size_t seqlen_q_rounded,
 195:                       const size_t seqlen_k_rounded,
 196:                       const size_t h,
 197:                       const size_t h_k,
 198:                       const size_t d,
 199:                       const size_t d_rounded,
 200:                       // device pointers
```
- L161: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L162: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L163: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L164: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L166: Documents the nearby logic: Causal is the special case where window_size_right == 0 and window_size_left < 0. / 说明附近逻辑的作用：Causal is the special case where window_size_right == 0 and window_size_left < 0.
- L167: Documents the nearby logic: Local is the more general case where window_size_right >= 0 or window_size_left >= 0. / 说明附近逻辑的作用：Local is the more general case where window_size_right >= 0 or window_size_left >= 0.
- L168: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L170: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L171: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L172: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L173: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L175: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L176: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L177: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L178: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L180: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L182: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L183: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L184: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L185: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L186: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L187: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L189: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L190: Documents the nearby logic: sizes / 说明附近逻辑的作用：sizes
- L191: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L192: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L193: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L194: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L195: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L196: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L197: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L198: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L199: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L200: Documents the nearby logic: device pointers / 说明附近逻辑的作用：device pointers

### Lines 201-240

```cpp
 201:                       const at::Tensor q,
 202:                       const at::Tensor k,
 203:                       const at::Tensor v,
 204:                       const at::Tensor out,
 205:                       const at::Tensor dout,
 206:                       at::Tensor dq,
 207:                       at::Tensor dk,
 208:                       at::Tensor dv,
 209:                       void *cu_seqlens_q_d,
 210:                       void *cu_seqlens_k_d,
 211:                       void *dq_accum_d,
 212:                       void *dk_accum_d,
 213:                       void *dv_accum_d,
 214:                       void *softmax_lse_d,
 215:                       void *dsoftmax_sum_d,
 216:                       float p_dropout,
 217:                       float softmax_scale,
 218:                       int window_size_left,
 219:                       int window_size_right,
 220:                       const float softcap,
 221:                       bool deterministic,
 222:                       const bool unpadded_lse) {
 223: 
 224:     set_params_fprop(params,
 225:                      b, seqlen_q, seqlen_k, seqlen_q_rounded, seqlen_k_rounded, h, h_k, d, d_rounded,
 226:                      q, k, v, out,
 227:                      cu_seqlens_q_d,
 228:                      cu_seqlens_k_d,
 229:                      nullptr,
 230:                      nullptr,
 231:                      softmax_lse_d,
 232:                      p_dropout,
 233:                      softmax_scale,
 234:                      window_size_left,
 235:                      window_size_right,
 236:                      softcap,
 237:                      false, // seqlenq_ngroups_swapped
 238:                      unpadded_lse);
 239: 
 240:     // Set the pointers and strides.
```
- L201: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L202: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L203: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L204: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L205: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L206: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L207: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L208: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L209: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L210: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L211: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L212: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L213: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L214: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L215: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L216: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L217: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L218: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L219: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L220: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L221: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L222: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L224: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L225: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L226: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L227: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L228: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L229: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L230: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L231: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L232: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L233: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L234: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L235: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L236: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L237: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L238: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L240: Documents the nearby logic: Set the pointers and strides. / 说明附近逻辑的作用：Set the pointers and strides.

### Lines 241-280

```cpp
 241:     params.do_ptr = dout.data_ptr();
 242:     params.do_row_stride = dout.stride(-3);
 243:     params.do_head_stride = dout.stride(-2);
 244:     params.dq_ptr = dq.data_ptr();
 245:     params.dk_ptr = dk.data_ptr();
 246:     params.dv_ptr = dv.data_ptr();
 247:     params.dq_row_stride = dq.stride(-3);
 248:     params.dk_row_stride = dk.stride(-3);
 249:     params.dv_row_stride = dv.stride(-3);
 250:     params.dq_head_stride = dq.stride(-2);
 251:     params.dk_head_stride = dk.stride(-2);
 252:     params.dv_head_stride = dv.stride(-2);
 253: 
 254:     if (cu_seqlens_q_d == nullptr) {
 255:         params.do_batch_stride = dout.stride(0);
 256:         params.dq_batch_stride = dq.stride(0);
 257:         params.dk_batch_stride = dk.stride(0);
 258:         params.dv_batch_stride = dv.stride(0);
 259:     }
 260: 
 261:     params.dq_accum_ptr = dq_accum_d;
 262:     params.dk_accum_ptr = dk_accum_d;
 263:     params.dv_accum_ptr = dv_accum_d;
 264: 
 265:     // Softmax sum
 266:     params.dsoftmax_sum = dsoftmax_sum_d;
 267: 
 268:     params.deterministic = deterministic;
 269: }
 270: 
 271: void run_mha_fwd(Flash_fwd_params &params, cudaStream_t stream, bool force_split_kernel=false) {
 272:     FP16_SWITCH(!params.is_bf16, [&] {
 273:         HEADDIM_SWITCH(params.d, [&] {
 274:             BOOL_SWITCH(params.is_causal, Is_causal, [&] {
 275:                 if (params.num_splits <= 1 && !force_split_kernel) {  // If we don't set it num_splits == 0
 276:                     run_mha_fwd_<elem_type, kHeadDim, Is_causal>(params, stream);
 277:                 } else {
 278:                     run_mha_fwd_splitkv_dispatch<elem_type, kHeadDim, Is_causal>(params, stream);
 279:                 }
 280:             });
```
- L241: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L242: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L243: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L244: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L245: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L246: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L247: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L248: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L249: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L250: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L251: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L252: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L254: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L255: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L256: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L257: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L258: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L259: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L261: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L262: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L263: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L265: Documents the nearby logic: Softmax sum / 说明附近逻辑的作用：Softmax sum
- L266: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L268: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L269: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L271: Defines function `run_mha_fwd` and begins its implementation body. / 定义函数 `run_mha_fwd`，并开始其实现体。
- L272: Defines function `FP16_SWITCH` and begins its implementation body. / 定义函数 `FP16_SWITCH`，并开始其实现体。
- L273: Defines function `HEADDIM_SWITCH` and begins its implementation body. / 定义函数 `HEADDIM_SWITCH`，并开始其实现体。
- L274: Defines function `BOOL_SWITCH` and begins its implementation body. / 定义函数 `BOOL_SWITCH`，并开始其实现体。
- L275: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L276: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L277: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L278: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L279: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L280: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 281-320

```cpp
 281:         });
 282:     });
 283: }
 284: 
 285: // Find the number of splits that maximizes the occupancy. For example, if we have
 286: // batch * n_heads = 48 and we have 108 SMs, having 2 splits (efficiency = 0.89) is
 287: // better than having 3 splits (efficiency = 0.67). However, we also don't want too many
 288: // splits as that would incur more HBM reads/writes.
 289: // So we find the best efficiency, then find the smallest number of splits that gets 85%
 290: // of the best efficiency.
 291: inline int num_splits_heuristic(int batch_nheads_mblocks, int num_SMs, int num_n_blocks, int max_splits) {
 292:     // If we have enough to almost fill the SMs, then just use 1 split
 293:     if (batch_nheads_mblocks >= 0.8f * num_SMs) { return 1; }
 294:     max_splits = std::min({max_splits, num_SMs, num_n_blocks});
 295:     float max_efficiency = 0.f;
 296:     std::vector<float> efficiency;
 297:     efficiency.reserve(max_splits);
 298:     auto ceildiv = [](int a, int b) { return (a + b - 1) / b; };
 299:     // Some splits are not eligible. For example, if we have 64 blocks and choose 11 splits,
 300:     // we'll have 6 * 10 + 4 blocks. If we choose 12 splits, we'll have 6 * 11 + (-2) blocks
 301:     // (i.e. it's 11 splits anyway).
 302:     // So we check if the number of blocks per split is the same as the previous num_splits.
 303:     auto is_split_eligible = [&ceildiv, &num_n_blocks](int num_splits) {
 304:         return num_splits == 1 || ceildiv(num_n_blocks, num_splits) != ceildiv(num_n_blocks, num_splits - 1);
 305:     };
 306:     for (int num_splits = 1; num_splits <= max_splits; num_splits++) {
 307:         if (!is_split_eligible(num_splits)) {
 308:             efficiency.push_back(0.f);
 309:         } else {
 310:             float n_waves = static_cast<float>(batch_nheads_mblocks * num_splits) / num_SMs;
 311:             float eff = n_waves / ceil(n_waves);
 312:             // printf("num_splits = %d, eff = %f\n", num_splits, eff);
 313:             if (eff > max_efficiency) { max_efficiency = eff; }
 314:             efficiency.push_back(eff);
 315:         }
 316:     }
 317:     for (int num_splits = 1; num_splits <= max_splits; num_splits++) {
 318:         if (!is_split_eligible(num_splits)) { continue; }
 319:         if (efficiency[num_splits - 1] >= 0.85 * max_efficiency) {
 320:             // printf("num_splits chosen = %d\n", num_splits);
```
- L281: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L282: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L283: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L285: Documents the nearby logic: Find the number of splits that maximizes the occupancy. For example, if we have / 说明附近逻辑的作用：Find the number of splits that maximizes the occupancy. For example, if we have
- L286: Documents the nearby logic: batch * n_heads = 48 and we have 108 SMs, having 2 splits (efficiency = 0.89) is / 说明附近逻辑的作用：batch * n_heads = 48 and we have 108 SMs, having 2 splits (efficiency = 0.89) is
- L287: Documents the nearby logic: better than having 3 splits (efficiency = 0.67). However, we also don't want too many / 说明附近逻辑的作用：better than having 3 splits (efficiency = 0.67). However, we also don't want too many
- L288: Documents the nearby logic: splits as that would incur more HBM reads/writes. / 说明附近逻辑的作用：splits as that would incur more HBM reads/writes.
- L289: Documents the nearby logic: So we find the best efficiency, then find the smallest number of splits that gets 85% / 说明附近逻辑的作用：So we find the best efficiency, then find the smallest number of splits that gets 85%
- L290: Documents the nearby logic: of the best efficiency. / 说明附近逻辑的作用：of the best efficiency.
- L291: Marks the entity inline so it can be defined in headers safely. / 将实体标记为 inline，使其可安全地在头文件中定义。
- L292: Documents the nearby logic: If we have enough to almost fill the SMs, then just use 1 split / 说明附近逻辑的作用：If we have enough to almost fill the SMs, then just use 1 split
- L293: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L294: Declares function `min` as part of this file's callable surface. / 声明函数 `min`，作为本文件可调用接口的一部分。
- L295: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L296: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L297: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L298: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L299: Documents the nearby logic: Some splits are not eligible. For example, if we have 64 blocks and choose 11 splits, / 说明附近逻辑的作用：Some splits are not eligible. For example, if we have 64 blocks and choose 11 splits,
- L300: Documents the nearby logic: we'll have 6 * 10 + 4 blocks. If we choose 12 splits, we'll have 6 * 11 + (-2) blocks / 说明附近逻辑的作用：we'll have 6 * 10 + 4 blocks. If we choose 12 splits, we'll have 6 * 11 + (-2) blocks
- L301: Documents the nearby logic: (i.e. it's 11 splits anyway). / 说明附近逻辑的作用：(i.e. it's 11 splits anyway).
- L302: Documents the nearby logic: So we check if the number of blocks per split is the same as the previous num_splits. / 说明附近逻辑的作用：So we check if the number of blocks per split is the same as the previous num_splits.
- L303: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L304: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L305: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L306: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L307: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L308: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L309: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L310: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L311: Declares function `ceil` as part of this file's callable surface. / 声明函数 `ceil`，作为本文件可调用接口的一部分。
- L312: Documents the nearby logic: printf("num_splits = %d, eff = %f\n", num_splits, eff); / 说明附近逻辑的作用：printf("num_splits = %d, eff = %f\n", num_splits, eff);
- L313: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L314: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L315: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L316: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L317: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L318: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L319: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L320: Documents the nearby logic: printf("num_splits chosen = %d\n", num_splits); / 说明附近逻辑的作用：printf("num_splits chosen = %d\n", num_splits);

### Lines 321-360

```cpp
 321:             return num_splits;
 322:         }
 323:     }
 324:     return 1;
 325: }
 326: std::tuple<at::Tensor, at::Tensor> set_params_splitkv(Flash_fwd_params &params, const int batch_size,
 327:     const int num_heads, const int head_size, const int max_seqlen_k, const int max_seqlen_q,
 328:     const int head_size_rounded, const float p_dropout,
 329:     const int num_splits, cudaDeviceProp *dprops, struct c10::TensorOptions opts) {
 330: 
 331:     // This needs to match with run_mha_fwd_splitkv_dispatch
 332:     const int block_n = head_size <= 64 ? 256 : (head_size <= 128 ? 128 : 64);
 333:     const int num_n_blocks = (max_seqlen_k + block_n - 1) / block_n;
 334:     // Technically kBlockM = 64 only for the splitKV kernels, not the standard kernel.
 335:     // In any case we don't expect seqlen_q to be larger than 64 for inference.
 336:     const int num_m_blocks = (max_seqlen_q + 64 - 1) / 64;
 337:     params.num_splits = num_splits;
 338:     at::Tensor softmax_lse_accum;
 339:     at::Tensor out_accum;
 340: 
 341:     if (p_dropout == 0.0f) {  // SplitKV is not implemented for dropout
 342:         if (num_splits < 1) {
 343:             // We multiply number of SMs by 2 to hard-code the fact that we're using 128 threads per block.
 344:             params.num_splits = num_splits_heuristic(batch_size * num_heads * num_m_blocks, dprops->multiProcessorCount * 2, num_n_blocks, 128);
 345:         }
 346:         if (params.num_splits > 1) {
 347:             softmax_lse_accum = at::empty({params.num_splits, batch_size, num_heads, max_seqlen_q}, opts.dtype(at::kFloat));
 348:             out_accum = at::empty({params.num_splits, batch_size, num_heads, max_seqlen_q, head_size_rounded}, opts.dtype(at::kFloat));
 349:             params.softmax_lseaccum_ptr = softmax_lse_accum.data_ptr();
 350:             params.oaccum_ptr = out_accum.data_ptr();
 351:         }
 352:         TORCH_CHECK(params.num_splits <= 128, "num_splits > 128 not supported");
 353:     }
 354: 
 355:     return std::make_tuple(softmax_lse_accum, out_accum);
 356: }
 357: 
 358: void set_params_alibi(Flash_fwd_params &params, std::optional<at::Tensor> &alibi_slopes_, int batch_size, int num_heads){
 359: #ifdef FLASHATTENTION_DISABLE_ALIBI
 360:     TORCH_CHECK(!alibi_slopes_.has_value(), "This flash attention build does not support alibi.");
```
- L321: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L322: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L323: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L324: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L325: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L326: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L327: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L328: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L329: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L331: Documents the nearby logic: This needs to match with run_mha_fwd_splitkv_dispatch / 说明附近逻辑的作用：This needs to match with run_mha_fwd_splitkv_dispatch
- L332: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L333: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L334: Documents the nearby logic: Technically kBlockM = 64 only for the splitKV kernels, not the standard kernel. / 说明附近逻辑的作用：Technically kBlockM = 64 only for the splitKV kernels, not the standard kernel.
- L335: Documents the nearby logic: In any case we don't expect seqlen_q to be larger than 64 for inference. / 说明附近逻辑的作用：In any case we don't expect seqlen_q to be larger than 64 for inference.
- L336: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L337: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L338: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L339: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L341: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L342: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L343: Documents the nearby logic: We multiply number of SMs by 2 to hard-code the fact that we're using 128 threads per block. / 说明附近逻辑的作用：We multiply number of SMs by 2 to hard-code the fact that we're using 128 threads per block.
- L344: Declares function `num_splits_heuristic` as part of this file's callable surface. / 声明函数 `num_splits_heuristic`，作为本文件可调用接口的一部分。
- L345: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L346: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L347: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L348: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L349: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L350: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L351: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L352: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L353: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L355: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L356: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L358: Defines function `set_params_alibi` and begins its implementation body. / 定义函数 `set_params_alibi`，并开始其实现体。
- L359: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L360: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。

### Lines 361-400

```cpp
 361:     params.alibi_slopes_ptr = nullptr;
 362: #else
 363:     if (alibi_slopes_.has_value()) {
 364:         auto alibi_slopes = alibi_slopes_.value();
 365:         TORCH_CHECK(alibi_slopes.dtype() == at::kFloat, "ALiBi slopes must have dtype fp32");
 366:         CHECK_DEVICE(alibi_slopes);
 367:         TORCH_CHECK(alibi_slopes.stride(-1) == 1, "ALiBi slopes tensor must have contiguous last dimension");
 368:         TORCH_CHECK(alibi_slopes.sizes() == at::IntArrayRef({num_heads}) || alibi_slopes.sizes() == at::IntArrayRef({batch_size, num_heads}));
 369:         params.alibi_slopes_ptr = alibi_slopes.data_ptr();
 370:         params.alibi_slopes_batch_stride = alibi_slopes.dim() == 2 ? alibi_slopes.stride(0) : 0;
 371:     } else {
 372:         params.alibi_slopes_ptr = nullptr;
 373:     }
 374: #endif
 375: }
 376: 
 377: // return {out, q_padded, k_padded, v_padded, out_padded, softmax_lse, p};
 378: std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor>
 379: mha_fwd(const at::Tensor &q,         // batch_size x seqlen_q x num_heads x head_size
 380:         const at::Tensor &k,         // batch_size x seqlen_k x num_heads_k x head_size
 381:         const at::Tensor &v,         // batch_size x seqlen_k x num_heads_k x head_size
 382:         std::optional<at::Tensor> &out_,             // batch_size x seqlen_q x num_heads x head_size
 383:         std::optional<at::Tensor> &alibi_slopes_, // num_heads or batch_size x num_heads
 384:         const float p_dropout,
 385:         const float softmax_scale,
 386:         bool is_causal,
 387:         int window_size_left,
 388:         int window_size_right,
 389:         const float softcap,
 390:         const bool return_softmax,
 391:         std::optional<at::Generator> gen_) {
 392: 
 393:     auto dprops = at::cuda::getCurrentDeviceProperties();
 394:     bool is_sm80_or_newer = (dprops->major * 10) >= 80;
 395:     TORCH_CHECK(is_sm80_or_newer, "FlashAttention only supports Ampere GPUs or newer.");
 396: 
 397:     auto q_dtype = q.dtype();
 398:     TORCH_CHECK(q_dtype == at::kHalf || q_dtype == at::kBFloat16,
 399:                 "FlashAttention only support fp16 and bf16 data type");
 400:     if (q_dtype == at::kBFloat16) {
```
- L361: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L362: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L363: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L364: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L365: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L366: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L367: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L368: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L369: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L370: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L371: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L372: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L373: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L374: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L375: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L377: Documents the nearby logic: return {out, q_padded, k_padded, v_padded, out_padded, softmax_lse, p}; / 说明附近逻辑的作用：return {out, q_padded, k_padded, v_padded, out_padded, softmax_lse, p};
- L378: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L379: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L380: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L381: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L382: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L383: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L384: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L385: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L386: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L387: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L388: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L389: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L390: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L391: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L393: Declares function `getCurrentDeviceProperties` as part of this file's callable surface. / 声明函数 `getCurrentDeviceProperties`，作为本文件可调用接口的一部分。
- L394: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L395: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L397: Declares function `dtype` as part of this file's callable surface. / 声明函数 `dtype`，作为本文件可调用接口的一部分。
- L398: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L399: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L400: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 401-440

```cpp
 401:         TORCH_CHECK(is_sm80_or_newer, "bfloat16 is only supported on Ampere GPUs or newer");
 402:     }
 403:     TORCH_CHECK(k.dtype() == q_dtype, "query and key must have the same dtype");
 404:     TORCH_CHECK(v.dtype() == q_dtype, "query and value must have the same dtype");
 405: 
 406:     CHECK_DEVICE(q); CHECK_DEVICE(k); CHECK_DEVICE(v);
 407: 
 408:     TORCH_CHECK(q.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 409:     TORCH_CHECK(k.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 410:     TORCH_CHECK(v.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 411: 
 412:     const auto sizes = q.sizes();
 413: 
 414:     const int batch_size = sizes[0];
 415:     int seqlen_q = sizes[1];
 416:     int num_heads = sizes[2];
 417:     const int head_size_og = sizes[3];
 418:     const int seqlen_k = k.size(1);
 419:     const int num_heads_k = k.size(2);
 420: 
 421:     if (batch_size == 0) {
 422:         auto opts = q.options();
 423:         at::Tensor out = at::empty({0, seqlen_q, num_heads, head_size_og}, opts);
 424:         at::Tensor q_padded = at::empty({0, seqlen_q, num_heads, head_size_og}, opts);
 425:         at::Tensor k_padded = at::empty({0, seqlen_k, num_heads_k, head_size_og}, opts);
 426:         at::Tensor v_padded = at::empty({0, seqlen_k, num_heads_k, head_size_og}, opts);
 427:         at::Tensor softmax_lse = at::empty({0, num_heads, seqlen_q}, opts.dtype(at::kFloat));
 428:         at::Tensor rng_state = at::empty({2}, at::dtype(c10::kUInt64).device(at::kCUDA));
 429:         at::Tensor _unused = at::empty({}, at::dtype(c10::kUInt64).device(at::kCUDA));
 430:         at::Tensor p = at::empty({0}, opts);
 431:         if (return_softmax) {
 432:             auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
 433:             const int seqlen_q_rounded = round_multiple(seqlen_q, 128);
 434:             const int seqlen_k_rounded = round_multiple(seqlen_k, 128);
 435:             p = at::empty({0, num_heads, seqlen_q_rounded, seqlen_k_rounded}, opts);
 436:         }
 437:         return {std::move(out), std::move(q_padded), std::move(k_padded), std::move(v_padded), std::move(softmax_lse), std::move(rng_state), _unused, std::move(p)};
 438:     }
 439: 
 440:     TORCH_CHECK(batch_size > 0, "batch size must be positive");
```
- L401: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L402: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L403: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L404: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L406: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L408: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L409: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L410: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L412: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L414: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L415: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L416: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L417: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L418: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L419: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L421: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L422: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L423: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L424: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L425: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L426: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L427: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L428: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L429: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L430: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L431: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L432: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L433: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L434: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L435: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L436: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L437: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L438: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L440: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。

### Lines 441-480

```cpp
 441:     TORCH_CHECK(head_size_og % 8 == 0, "head_size must be a multiple of 8, this is ensured by padding!");
 442:     TORCH_CHECK(head_size_og <= 256, "FlashAttention forward only supports head dimension at most 256");
 443:     TORCH_CHECK(num_heads % num_heads_k == 0, "Number of heads in key/value must divide number of heads in query");
 444: 
 445:     if (softcap > 0.f) { TORCH_CHECK(p_dropout == 0.f, "Softcapping does not support dropout for now"); }
 446: 
 447:     if (window_size_left >= seqlen_k) { window_size_left = -1; }
 448:     if (window_size_right >= seqlen_k) { window_size_right = -1; }
 449: 
 450:     // causal=true is the same as causal=false in this case
 451:     if (seqlen_q == 1 && !alibi_slopes_.has_value()) { is_causal = false; }
 452:     if (is_causal) { window_size_right = 0; }
 453: 
 454:     // Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case
 455:     // H/t Daniel Haziza
 456:     const int seqlenq_ngroups_swapped = seqlen_q == 1 && num_heads > num_heads_k && window_size_left < 0 && window_size_right < 0 && p_dropout == 0.f && head_size_og % 8 == 0 && !alibi_slopes_.has_value();
 457:     const int ngroups = num_heads / num_heads_k;
 458:     at::Tensor temp_q = q;
 459:     if (seqlenq_ngroups_swapped) {
 460:         temp_q = q.reshape({batch_size, num_heads_k, ngroups, head_size_og}).transpose(1, 2);
 461:         seqlen_q = ngroups;
 462:         num_heads = num_heads_k;
 463:     }
 464: 
 465:     CHECK_SHAPE(temp_q, batch_size, seqlen_q, num_heads, head_size_og);
 466:     CHECK_SHAPE(k, batch_size, seqlen_k, num_heads_k, head_size_og);
 467:     CHECK_SHAPE(v, batch_size, seqlen_k, num_heads_k, head_size_og);
 468: 
 469:     at::Tensor q_padded, k_padded, v_padded;
 470:     q_padded = temp_q;
 471:     k_padded = k;
 472:     v_padded = v;
 473: 
 474:     at::Tensor out;
 475:     if (out_.has_value()) {
 476:         out = out_.value();
 477:         TORCH_CHECK(out.dtype() == q_dtype, "Output must have the same dtype as inputs");
 478:         CHECK_DEVICE(out);
 479:         TORCH_CHECK(out.stride(-1) == 1, "Output tensor must have contiguous last dimension");
 480:         CHECK_SHAPE(out, batch_size, sizes[1], sizes[2], head_size_og);
```
- L441: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L442: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L443: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L445: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L447: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L448: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L450: Documents the nearby logic: causal=true is the same as causal=false in this case / 说明附近逻辑的作用：causal=true is the same as causal=false in this case
- L451: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L452: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L454: Documents the nearby logic: Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case / 说明附近逻辑的作用：Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case
- L455: Documents the nearby logic: H/t Daniel Haziza / 说明附近逻辑的作用：H/t Daniel Haziza
- L456: Declares function `has_value` as part of this file's callable surface. / 声明函数 `has_value`，作为本文件可调用接口的一部分。
- L457: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L458: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L459: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L460: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L461: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L462: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L463: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L465: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L466: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L467: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L469: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L470: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L471: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L472: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L474: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L475: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L476: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L477: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L478: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L479: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L480: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。

### Lines 481-520

```cpp
 481:         if (seqlenq_ngroups_swapped) {
 482:             out = out.reshape({batch_size, num_heads_k, ngroups, head_size_og}).transpose(1, 2);
 483:         }
 484:         CHECK_SHAPE(out, batch_size, seqlen_q, num_heads, head_size_og);
 485:         if (head_size_og % 8 != 0) { out = at::empty_like(q_padded); }
 486:     } else {
 487:         out = at::empty_like(q_padded);
 488:     }
 489: 
 490:     auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
 491:     const int head_size = round_multiple(head_size_og, 8);
 492:     const int head_size_rounded = round_multiple(head_size, head_size <= 128 ? 32 : 64);
 493:     const int seqlen_q_rounded = round_multiple(seqlen_q, 128);
 494:     const int seqlen_k_rounded = round_multiple(seqlen_k, 128);
 495: 
 496:     // Otherwise the kernel will be launched from cuda:0 device
 497:     // Cast to char to avoid compiler warning about narrowing
 498:     at::cuda::CUDAGuard device_guard{static_cast<signed char>(q.get_device())};
 499: 
 500:     auto opts = q.options();
 501: 
 502:     auto softmax_lse = at::empty({batch_size, num_heads, seqlen_q }, opts.dtype(at::kFloat));
 503: 
 504:     at::Tensor p;
 505:     // Only return softmax if there's dropout to reduce compilation time
 506:     if (return_softmax) {
 507:         TORCH_CHECK(p_dropout > 0.0f, "return_softmax is only supported when p_dropout > 0.0");
 508:         p = at::empty({ batch_size, num_heads, seqlen_q_rounded, seqlen_k_rounded }, opts);
 509:     }
 510: 
 511:     Flash_fwd_params params;
 512:     set_params_fprop(params,
 513:                      batch_size,
 514:                      seqlen_q, seqlen_k,
 515:                      seqlen_q_rounded, seqlen_k_rounded,
 516:                      num_heads, num_heads_k,
 517:                      head_size, head_size_rounded,
 518:                      q_padded, k_padded, v_padded, out,
 519:                      /*cu_seqlens_q_d=*/nullptr,
 520:                      /*cu_seqlens_k_d=*/nullptr,
```
- L481: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L482: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L483: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L484: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L485: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L486: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L487: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L488: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L490: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L491: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L492: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L493: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L494: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L496: Documents the nearby logic: Otherwise the kernel will be launched from cuda:0 device / 说明附近逻辑的作用：Otherwise the kernel will be launched from cuda:0 device
- L497: Documents the nearby logic: Cast to char to avoid compiler warning about narrowing / 说明附近逻辑的作用：Cast to char to avoid compiler warning about narrowing
- L498: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L500: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L502: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L504: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L505: Documents the nearby logic: Only return softmax if there's dropout to reduce compilation time / 说明附近逻辑的作用：Only return softmax if there's dropout to reduce compilation time
- L506: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L507: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L508: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L509: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L511: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L512: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L513: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L514: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L515: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L516: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L517: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L518: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L519: Documents the nearby logic: cu_seqlens_q_d=*/nullptr, / 说明附近逻辑的作用：cu_seqlens_q_d=*/nullptr,
- L520: Documents the nearby logic: cu_seqlens_k_d=*/nullptr, / 说明附近逻辑的作用：cu_seqlens_k_d=*/nullptr,

### Lines 521-560

```cpp
 521:                      /*seqused_k=*/nullptr,
 522:                      return_softmax ? p.data_ptr() : nullptr,
 523:                      softmax_lse.data_ptr(),
 524:                      p_dropout,
 525:                      softmax_scale,
 526:                      window_size_left,
 527:                      window_size_right,
 528:                      softcap
 529:                      );
 530: 
 531: 
 532:     // Keep references to these tensors to extend their lifetime
 533:     auto [softmax_lse_accum, out_accum] = set_params_splitkv(params, batch_size, num_heads,
 534:                         head_size, seqlen_k, seqlen_q,
 535:                         head_size_rounded, p_dropout, /*num_splits*/0, dprops, opts);
 536: 
 537:     // See [Note] BC breaking change to flash seed/offset
 538:     auto rng_state = at::empty({2}, at::dtype(c10::kUInt64).device(at::kCUDA));
 539:     auto _unused = at::empty({}, at::dtype(c10::kUInt64).device(at::kCUDA));
 540:     if (p_dropout > 0.0)  {
 541:         auto gen = at::get_generator_or_default<at::CUDAGeneratorImpl>(std::nullopt, at::cuda::detail::getDefaultCUDAGenerator());
 542:         // number of times random will be generated per thread, to offset philox counter in thc random
 543:         // state
 544:         // We use a custom RNG that increases the offset by batch_size * nheads * 32.
 545:         int64_t counter_offset = params.b * params.h * 32;
 546:         // See Note [Acquire lock when using random generators]
 547:         std::lock_guard<std::mutex> lock(gen->mutex_);
 548:         at::PhiloxCudaState philox_state = gen->philox_cuda_state(counter_offset);
 549:         rng_state = at::empty({2}, at::TensorOptions().dtype(c10::kUInt64).device(at::kCUDA));
 550:         params.rng_state = reinterpret_cast<uint64_t*>(rng_state.data_ptr());
 551:         params.philox_args = philox_state;
 552:     }
 553: 
 554:     set_params_alibi(params, alibi_slopes_, batch_size, num_heads);
 555: 
 556:     if (seqlen_k > 0) {
 557:         auto stream = at::cuda::getCurrentCUDAStream().stream();
 558:         run_mha_fwd(params, stream);
 559:     } else {
 560:         // If seqlen_k == 0, then we have an empty tensor. We need to set the output to 0.
```
- L521: Documents the nearby logic: seqused_k=*/nullptr, / 说明附近逻辑的作用：seqused_k=*/nullptr,
- L522: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L523: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L524: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L525: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L526: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L527: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L528: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L529: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L532: Documents the nearby logic: Keep references to these tensors to extend their lifetime / 说明附近逻辑的作用：Keep references to these tensors to extend their lifetime
- L533: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L534: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L535: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L537: Documents the nearby logic: See [Note] BC breaking change to flash seed/offset / 说明附近逻辑的作用：See [Note] BC breaking change to flash seed/offset
- L538: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L539: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L540: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L541: Declares function `getDefaultCUDAGenerator` as part of this file's callable surface. / 声明函数 `getDefaultCUDAGenerator`，作为本文件可调用接口的一部分。
- L542: Documents the nearby logic: number of times random will be generated per thread, to offset philox counter in thc random / 说明附近逻辑的作用：number of times random will be generated per thread, to offset philox counter in thc random
- L543: Documents the nearby logic: state / 说明附近逻辑的作用：state
- L544: Documents the nearby logic: We use a custom RNG that increases the offset by batch_size * nheads * 32. / 说明附近逻辑的作用：We use a custom RNG that increases the offset by batch_size * nheads * 32.
- L545: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L546: Documents the nearby logic: See Note [Acquire lock when using random generators] / 说明附近逻辑的作用：See Note [Acquire lock when using random generators]
- L547: Declares function `lock` as part of this file's callable surface. / 声明函数 `lock`，作为本文件可调用接口的一部分。
- L548: Declares function `philox_cuda_state` as part of this file's callable surface. / 声明函数 `philox_cuda_state`，作为本文件可调用接口的一部分。
- L549: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L550: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L551: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L552: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L554: Declares function `set_params_alibi` as part of this file's callable surface. / 声明函数 `set_params_alibi`，作为本文件可调用接口的一部分。
- L556: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L557: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。
- L558: Declares function `run_mha_fwd` as part of this file's callable surface. / 声明函数 `run_mha_fwd`，作为本文件可调用接口的一部分。
- L559: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L560: Documents the nearby logic: If seqlen_k == 0, then we have an empty tensor. We need to set the output to 0. / 说明附近逻辑的作用：If seqlen_k == 0, then we have an empty tensor. We need to set the output to 0.

### Lines 561-600

```cpp
 561:         out.zero_();
 562:         softmax_lse.fill_(std::numeric_limits<float>::infinity());
 563:     }
 564: 
 565:     if (seqlenq_ngroups_swapped) {
 566:         out = out.transpose(1, 2).reshape({batch_size, 1, num_heads_k * seqlen_q, head_size_og});
 567:         q_padded = q_padded.transpose(1, 2).reshape({batch_size, 1, num_heads_k * seqlen_q, head_size_og});
 568:         softmax_lse = softmax_lse.reshape({batch_size, num_heads_k * seqlen_q, 1});
 569:     }
 570:     return {std::move(out), std::move(q_padded), std::move(k_padded), std::move(v_padded), std::move(softmax_lse), std::move(rng_state), std::move(_unused), std::move(p)};
 571: }
 572: 
 573: std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor, at::Tensor>
 574: mha_varlen_fwd(const at::Tensor &q,  // total_q x num_heads x head_size, total_q := \sum_{i=0}^{b} s_i
 575:                const at::Tensor &k,  // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i or num_blocks x page_block_size x num_heads_k x head_size if there's a block_table.
 576:                const at::Tensor &v,  // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i or num_blocks x page_block_size x num_heads_k x head_size if there's a block_table.
 577:                std::optional<at::Tensor> &out_, // total_q x num_heads x head_size, total_k := \sum_{i=0}^{b} s_i
 578:                const at::Tensor &cu_seqlens_q,  // b+1
 579:                const at::Tensor &cu_seqlens_k,  // b+1
 580:                std::optional<at::Tensor> &seqused_k, // b. If given, only this many elements of each batch element's keys are used.
 581:                std::optional<at::Tensor> &block_table_, // batch_size x max_num_blocks_per_seq
 582:                std::optional<at::Tensor> &alibi_slopes_, // num_heads or b x num_heads
 583:                int max_seqlen_q,
 584:                const int max_seqlen_k,
 585:                const float p_dropout,
 586:                const float softmax_scale,
 587:                const bool zero_tensors,
 588:                bool is_causal,
 589:                int window_size_left,
 590:                int window_size_right,
 591:                const float softcap,
 592:                const bool return_softmax,
 593:                std::optional<at::Generator> gen_,
 594:                int num_splits) {
 595: 
 596:     auto dprops = at::cuda::getCurrentDeviceProperties();
 597:     bool is_sm80_or_newer = (dprops->major * 10) >= 80;
 598:     TORCH_CHECK(is_sm80_or_newer, "FlashAttention only supports Ampere GPUs or newer.");
 599: 
 600:     auto q_dtype = q.dtype();
```
- L561: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L562: Declares function `fill_` as part of this file's callable surface. / 声明函数 `fill_`，作为本文件可调用接口的一部分。
- L563: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L565: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L566: Declares function `transpose` as part of this file's callable surface. / 声明函数 `transpose`，作为本文件可调用接口的一部分。
- L567: Declares function `transpose` as part of this file's callable surface. / 声明函数 `transpose`，作为本文件可调用接口的一部分。
- L568: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L569: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L570: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L571: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L573: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L574: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L575: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L576: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L577: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L578: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L579: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L580: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L581: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L582: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L583: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L584: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L585: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L586: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L587: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L588: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L589: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L590: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L591: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L592: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L593: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L594: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L596: Declares function `getCurrentDeviceProperties` as part of this file's callable surface. / 声明函数 `getCurrentDeviceProperties`，作为本文件可调用接口的一部分。
- L597: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L598: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L600: Declares function `dtype` as part of this file's callable surface. / 声明函数 `dtype`，作为本文件可调用接口的一部分。

### Lines 601-640

```cpp
 601:     TORCH_CHECK(q_dtype == at::kHalf || q_dtype == at::kBFloat16,
 602:                 "FlashAttention only support fp16 and bf16 data type");
 603:     if (q_dtype == at::kBFloat16) {
 604:         TORCH_CHECK(is_sm80_or_newer, "bfloat16 is only supported on Ampere GPUs or newer");
 605:     }
 606:     TORCH_CHECK(k.dtype() == q_dtype, "query and key must have the same dtype");
 607:     TORCH_CHECK(v.dtype() == q_dtype, "query and value must have the same dtype");
 608:     TORCH_CHECK(cu_seqlens_q.dtype() == at::kInt, "cu_seqlens_q must have dtype int32");
 609:     TORCH_CHECK(cu_seqlens_k.dtype() == at::kInt, "cu_seqlens_k must have dtype int32");
 610: 
 611:     CHECK_DEVICE(q); CHECK_DEVICE(k); CHECK_DEVICE(v);
 612:     CHECK_DEVICE(cu_seqlens_q);
 613:     CHECK_DEVICE(cu_seqlens_k);
 614: 
 615:     at::Tensor block_table;
 616:     const bool paged_KV = block_table_.has_value();
 617:     if (paged_KV) {
 618:         block_table = block_table_.value();
 619:         CHECK_DEVICE(block_table);
 620:         TORCH_CHECK(block_table.dtype() == at::kInt, "block_table must have dtype torch.int32");
 621:         TORCH_CHECK(block_table.stride(-1) == 1, "block_table must have contiguous last dimension");
 622:     }
 623: 
 624:     TORCH_CHECK(q.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 625:     TORCH_CHECK(k.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 626:     TORCH_CHECK(v.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 627:     CHECK_CONTIGUOUS(cu_seqlens_q);
 628:     CHECK_CONTIGUOUS(cu_seqlens_k);
 629: 
 630:     const auto sizes = q.sizes();
 631: 
 632:     const int batch_size = cu_seqlens_q.numel() - 1;
 633:     int num_heads = sizes[1];
 634:     const int head_size_og = sizes[2];
 635:     const int num_heads_k = paged_KV ? k.size(2) : k.size(1);
 636: 
 637:     if (softcap > 0.f) { TORCH_CHECK(p_dropout == 0.f, "Softcapping does not support dropout for now"); }
 638: 
 639:     const int max_num_blocks_per_seq = !paged_KV ? 0 : block_table.size(1);
 640:     const int num_blocks = !paged_KV ? 0 : k.size(0);
```
- L601: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L602: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L603: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L604: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L605: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L606: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L607: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L608: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L609: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L611: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L612: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L613: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L615: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L616: Declares function `has_value` as part of this file's callable surface. / 声明函数 `has_value`，作为本文件可调用接口的一部分。
- L617: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L618: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L619: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L620: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L621: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L622: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L624: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L625: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L626: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L627: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L628: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L630: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L632: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L633: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L634: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L635: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L637: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L639: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L640: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。

### Lines 641-680

```cpp
 641:     const int page_block_size = !paged_KV ? 1 : k.size(1);
 642:     TORCH_CHECK(!paged_KV || page_block_size % 256 == 0, "Paged KV cache block size must be divisible by 256");
 643: 
 644:     if (max_seqlen_q == 1 && !alibi_slopes_.has_value()) { is_causal = false; }  // causal=true is the same as causal=false in this case
 645:     if (is_causal) { window_size_right = 0; }
 646: 
 647:     void *cu_seqlens_q_d = cu_seqlens_q.data_ptr();
 648: 
 649:     // Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case
 650:     // H/t Daniel Haziza
 651:     const int seqlenq_ngroups_swapped = max_seqlen_q == 1 && num_heads > num_heads_k && window_size_left < 0 && window_size_right < 0 && p_dropout == 0.f && head_size_og % 8 == 0 && !alibi_slopes_.has_value();
 652:     at::Tensor temp_q = q;
 653:     const int ngroups = num_heads / num_heads_k;
 654:     if (seqlenq_ngroups_swapped) {
 655:         temp_q = q.reshape({batch_size, num_heads_k, ngroups, head_size_og}).transpose(1, 2).reshape({batch_size * ngroups, num_heads_k, head_size_og});
 656:         max_seqlen_q = ngroups;
 657:         num_heads = num_heads_k;
 658:         cu_seqlens_q_d = nullptr;
 659:     }
 660: 
 661:     const int total_q = temp_q.sizes()[0];
 662: 
 663:     TORCH_CHECK(batch_size > 0, "batch size must be positive");
 664:     TORCH_CHECK(head_size_og <= 256, "FlashAttention forward only supports head dimension at most 256");
 665:     TORCH_CHECK(num_heads % num_heads_k == 0, "Number of heads in key/value must divide number of heads in query");
 666:     TORCH_CHECK(head_size_og % 8 == 0, "head_size must be a multiple of 8, this is ensured by padding!")
 667: 
 668:     if (window_size_left >= max_seqlen_k) { window_size_left = -1; }
 669:     if (window_size_right >= max_seqlen_k) { window_size_right = -1; }
 670: 
 671:     CHECK_SHAPE(temp_q, total_q, num_heads, head_size_og);
 672:     if (!paged_KV) {
 673:         const int total_k = k.size(0);
 674:         CHECK_SHAPE(k, total_k, num_heads_k, head_size_og);
 675:         CHECK_SHAPE(v, total_k, num_heads_k, head_size_og);
 676:     } else {
 677:         CHECK_SHAPE(k, num_blocks, page_block_size, num_heads_k, head_size_og);
 678:         CHECK_SHAPE(v, num_blocks, page_block_size, num_heads_k, head_size_og);
 679:         CHECK_SHAPE(block_table, batch_size, max_num_blocks_per_seq);
 680:     }
```
- L641: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L642: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L644: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L645: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L647: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L649: Documents the nearby logic: Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case / 说明附近逻辑的作用：Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case
- L650: Documents the nearby logic: H/t Daniel Haziza / 说明附近逻辑的作用：H/t Daniel Haziza
- L651: Declares function `has_value` as part of this file's callable surface. / 声明函数 `has_value`，作为本文件可调用接口的一部分。
- L652: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L653: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L654: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L655: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L656: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L657: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L658: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L659: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L661: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L663: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L664: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L665: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L666: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L668: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L669: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L671: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L672: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L673: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L674: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L675: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L676: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L677: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L678: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L679: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L680: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 681-720

```cpp
 681:     CHECK_SHAPE(cu_seqlens_q, batch_size + 1);
 682:     CHECK_SHAPE(cu_seqlens_k, batch_size + 1);
 683:     if (seqused_k.has_value()){
 684:         auto seqused_k_ = seqused_k.value();
 685:         TORCH_CHECK(seqused_k_.dtype() == at::kInt, "seqused_k must have dtype int32");
 686:         TORCH_CHECK(seqused_k_.is_cuda(), "seqused_k must be on CUDA device");
 687:         TORCH_CHECK(seqused_k_.is_contiguous(), "seqused_k must be contiguous");
 688:         CHECK_SHAPE(seqused_k_, batch_size);
 689:     }
 690: 
 691:     at::Tensor q_padded, k_padded, v_padded;
 692:     q_padded = temp_q;
 693:     k_padded = k;
 694:     v_padded = v;
 695: 
 696:     at::Tensor out;
 697:     if (out_.has_value()) {
 698:         out = out_.value();
 699:         TORCH_CHECK(out.dtype() == q_dtype, "Output must have the same dtype as inputs");
 700:         CHECK_DEVICE(out);
 701:         TORCH_CHECK(out.stride(-1) == 1, "Output tensor must have contiguous last dimension");
 702:         CHECK_SHAPE(out, sizes[0], sizes[1], head_size_og);
 703:         if (seqlenq_ngroups_swapped) {
 704:             out = out.reshape({batch_size, num_heads_k, ngroups, head_size_og}).transpose(1, 2).reshape({batch_size * ngroups, num_heads_k, head_size_og});
 705:         }
 706:         if (head_size_og % 8 != 0) { out = at::empty_like(q_padded); }
 707:     } else {
 708:         out = at::empty_like(q_padded);
 709:     }
 710: 
 711:     auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
 712:     const int head_size = round_multiple(head_size_og, 8);
 713:     const int head_size_rounded = round_multiple(head_size, head_size <= 128 ? 32 : 64);
 714:     const int seqlen_q_rounded = round_multiple(max_seqlen_q, 128);
 715:     const int seqlen_k_rounded = round_multiple(max_seqlen_k, 128);
 716: 
 717:     // Otherwise the kernel will be launched from cuda:0 device
 718:     // Cast to char to avoid compiler warning about narrowing
 719:     at::cuda::CUDAGuard device_guard{static_cast<signed char>(q.get_device())};
 720: 
```
- L681: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L682: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L683: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L684: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L685: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L686: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L687: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L688: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L689: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L691: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L692: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L693: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L694: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L696: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L697: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L698: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L699: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L700: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L701: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L702: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L703: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L704: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L705: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L706: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L707: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L708: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L709: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L711: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L712: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L713: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L714: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L715: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L717: Documents the nearby logic: Otherwise the kernel will be launched from cuda:0 device / 说明附近逻辑的作用：Otherwise the kernel will be launched from cuda:0 device
- L718: Documents the nearby logic: Cast to char to avoid compiler warning about narrowing / 说明附近逻辑的作用：Cast to char to avoid compiler warning about narrowing
- L719: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 721-760

```cpp
 721:     auto opts = q.options();
 722: 
 723:     auto softmax_lse = at::empty({num_heads, total_q}, opts.dtype(at::kFloat));
 724:     at::Tensor p;
 725:     // Only return softmax if there's dropout to reduce compilation time
 726:     if (return_softmax) {
 727:         TORCH_CHECK(p_dropout > 0.0f, "return_softmax is only supported when p_dropout > 0.0");
 728:         p = at::empty({ batch_size, num_heads, seqlen_q_rounded, seqlen_k_rounded }, opts);
 729:     }
 730: 
 731:     if (zero_tensors) {
 732:         out.zero_();
 733:         softmax_lse.fill_(-std::numeric_limits<float>::infinity());
 734:         if (return_softmax) {p.zero_();}
 735:     }
 736: 
 737:     Flash_fwd_params params;
 738:     set_params_fprop(params,
 739:                      batch_size,
 740:                      max_seqlen_q, max_seqlen_k,
 741:                      seqlen_q_rounded, seqlen_k_rounded,
 742:                      num_heads, num_heads_k,
 743:                      head_size, head_size_rounded,
 744:                      q_padded, k_padded, v_padded, out,
 745:                      cu_seqlens_q_d,
 746:                      cu_seqlens_k.data_ptr(),
 747:                      seqused_k.has_value() ? seqused_k.value().data_ptr() : nullptr,
 748:                      return_softmax ? p.data_ptr() : nullptr,
 749:                      softmax_lse.data_ptr(),
 750:                      p_dropout,
 751:                      softmax_scale,
 752:                      window_size_left,
 753:                      window_size_right,
 754:                      softcap,
 755:                      seqlenq_ngroups_swapped,
 756:                      /*unpadded_lse*/true);
 757:     params.total_q = total_q;
 758:     if (paged_KV) {
 759:         params.block_table = block_table.data_ptr<int>();
 760:         params.block_table_batch_stride = block_table.stride(0);
```
- L721: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L723: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L724: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L725: Documents the nearby logic: Only return softmax if there's dropout to reduce compilation time / 说明附近逻辑的作用：Only return softmax if there's dropout to reduce compilation time
- L726: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L727: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L728: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L729: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L731: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L732: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L733: Declares function `fill_` as part of this file's callable surface. / 声明函数 `fill_`，作为本文件可调用接口的一部分。
- L734: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L735: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L737: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L738: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L739: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L740: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L741: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L742: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L743: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L744: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L745: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L746: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L747: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L748: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L749: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L750: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L751: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L752: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L753: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L754: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L755: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L756: Documents the nearby logic: unpadded_lse*/true); / 说明附近逻辑的作用：unpadded_lse*/true);
- L757: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L758: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L759: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L760: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。

### Lines 761-800

```cpp
 761:         params.k_batch_stride = k_padded.stride(0);
 762:         params.v_batch_stride = v_padded.stride(0);
 763:     }
 764:     params.page_block_size = page_block_size;
 765:     // Keep references to these tensors to extend their lifetime
 766:     at::Tensor softmax_lse_accum, out_accum;
 767:     if (paged_KV || seqlenq_ngroups_swapped) {
 768:         std::tie(softmax_lse_accum, out_accum) = set_params_splitkv(params, batch_size, num_heads,
 769:                            head_size, max_seqlen_k, max_seqlen_q,
 770:                            head_size_rounded, p_dropout, num_splits, dprops, opts);
 771:     }
 772: 
 773:     // [Note] BC breaking change to flash seed/offset
 774:     // Previously: Used separate tensors for philox_seed and philox_offset, sometimes on CPU, sometimes on CUDA
 775:     // FlashAttention change: Now uses a single uint64_t[2] tensor on device containing both seed and offset
 776:     // Implementation: Renamed "seed" → "rng_state" (contains both seed+offset) and "offset" → "_unused"
 777:     auto rng_state = at::empty({2}, at::dtype(c10::kUInt64).device(at::kCUDA));
 778:     auto _unused = at::empty({}, at::dtype(c10::kUInt64).device(at::kCUDA));
 779:     if (p_dropout > 0.0)  {
 780:         auto gen = at::get_generator_or_default<at::CUDAGeneratorImpl>(std::nullopt, at::cuda::detail::getDefaultCUDAGenerator());
 781:         // number of times random will be generated per thread, to offset philox counter in thc random
 782:         // state
 783:         // We use a custom RNG that increases the offset by batch_size * nheads * 32.
 784:         int64_t counter_offset = params.b * params.h * 32;
 785:         // See Note [Acquire lock when using random generators]
 786:         std::lock_guard<std::mutex> lock(gen->mutex_);
 787:         at::PhiloxCudaState philox_state = gen->philox_cuda_state(counter_offset);
 788:         rng_state = at::empty({2}, at::TensorOptions().dtype(c10::kUInt64).device(at::kCUDA));
 789:         params.rng_state = reinterpret_cast<uint64_t*>(rng_state.data_ptr());
 790:         params.philox_args = philox_state;
 791:     }
 792: 
 793:     set_params_alibi(params, alibi_slopes_, batch_size, num_heads);
 794: 
 795:     if (max_seqlen_k > 0) {
 796:         auto stream = at::cuda::getCurrentCUDAStream().stream();
 797:         run_mha_fwd(params, stream, paged_KV);
 798:     } else {
 799:         // If seqlen_k == 0, then we have an empty tensor. We need to set the output to 0.
 800:         out.zero_();
```
- L761: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L762: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L763: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L764: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L765: Documents the nearby logic: Keep references to these tensors to extend their lifetime / 说明附近逻辑的作用：Keep references to these tensors to extend their lifetime
- L766: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L767: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L768: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L769: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L770: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L771: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L773: Documents the nearby logic: [Note] BC breaking change to flash seed/offset / 说明附近逻辑的作用：[Note] BC breaking change to flash seed/offset
- L774: Documents the nearby logic: Previously: Used separate tensors for philox_seed and philox_offset, sometimes on CPU, sometimes on CUDA / 说明附近逻辑的作用：Previously: Used separate tensors for philox_seed and philox_offset, sometimes on CPU, sometimes on CUDA
- L775: Documents the nearby logic: FlashAttention change: Now uses a single uint64_t[2] tensor on device containing both seed and offset / 说明附近逻辑的作用：FlashAttention change: Now uses a single uint64_t[2] tensor on device containing both seed and offset
- L776: Documents the nearby logic: Implementation: Renamed "seed" → "rng_state" (contains both seed+offset) and "offset" → "_unused" / 说明附近逻辑的作用：Implementation: Renamed "seed" → "rng_state" (contains both seed+offset) and "offset" → "_unused"
- L777: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L778: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L779: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L780: Declares function `getDefaultCUDAGenerator` as part of this file's callable surface. / 声明函数 `getDefaultCUDAGenerator`，作为本文件可调用接口的一部分。
- L781: Documents the nearby logic: number of times random will be generated per thread, to offset philox counter in thc random / 说明附近逻辑的作用：number of times random will be generated per thread, to offset philox counter in thc random
- L782: Documents the nearby logic: state / 说明附近逻辑的作用：state
- L783: Documents the nearby logic: We use a custom RNG that increases the offset by batch_size * nheads * 32. / 说明附近逻辑的作用：We use a custom RNG that increases the offset by batch_size * nheads * 32.
- L784: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L785: Documents the nearby logic: See Note [Acquire lock when using random generators] / 说明附近逻辑的作用：See Note [Acquire lock when using random generators]
- L786: Declares function `lock` as part of this file's callable surface. / 声明函数 `lock`，作为本文件可调用接口的一部分。
- L787: Declares function `philox_cuda_state` as part of this file's callable surface. / 声明函数 `philox_cuda_state`，作为本文件可调用接口的一部分。
- L788: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L789: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L790: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L791: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L793: Declares function `set_params_alibi` as part of this file's callable surface. / 声明函数 `set_params_alibi`，作为本文件可调用接口的一部分。
- L795: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L796: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。
- L797: Declares function `run_mha_fwd` as part of this file's callable surface. / 声明函数 `run_mha_fwd`，作为本文件可调用接口的一部分。
- L798: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L799: Documents the nearby logic: If seqlen_k == 0, then we have an empty tensor. We need to set the output to 0. / 说明附近逻辑的作用：If seqlen_k == 0, then we have an empty tensor. We need to set the output to 0.
- L800: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。

### Lines 801-840

```cpp
 801:         softmax_lse.fill_(std::numeric_limits<float>::infinity());
 802:     }
 803: 
 804:     if (seqlenq_ngroups_swapped) {
 805:         std::array<int64_t, 4> size_before = {batch_size, max_seqlen_q, num_heads_k, head_size_og};
 806:         std::array<int64_t, 3> size_after = {batch_size, num_heads_k * max_seqlen_q, head_size_og};
 807:         out = out.reshape(size_before).transpose(1, 2).reshape(size_after);
 808:         q_padded = q_padded.reshape(size_before).transpose(1, 2).reshape(size_after);
 809:         softmax_lse = softmax_lse.reshape({num_heads * max_seqlen_q, batch_size});
 810:     }
 811: 
 812:     return {out, q_padded, k_padded, v_padded, softmax_lse, rng_state, _unused, p};
 813: }
 814: 
 815: void run_mha_bwd(Flash_bwd_params &params, cudaStream_t stream) {
 816:     FP16_SWITCH(!params.is_bf16, [&] {
 817:         HEADDIM_SWITCH(params.d, [&] {
 818:             BOOL_SWITCH(params.is_causal, Is_causal, [&] {
 819:                 run_mha_bwd_<elem_type, kHeadDim, Is_causal>(params, stream);
 820:             });
 821:         });
 822:     });
 823: }
 824: 
 825: std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor>
 826: mha_bwd(const at::Tensor &dout,  // batch_size x seqlen_q x num_heads, x head_size_og
 827:         const at::Tensor &q,   // batch_size x seqlen_q x num_heads x head_size
 828:         const at::Tensor &k,   // batch_size x seqlen_k x num_heads_k x head_size
 829:         const at::Tensor &v,   // batch_size x seqlen_k x num_heads_k x head_size
 830:         const at::Tensor &out,   // batch_size x seqlen_q x num_heads x head_size
 831:         const at::Tensor &softmax_lse,     // b x h x seqlen_q
 832:         std::optional<at::Tensor> &dq_,   // batch_size x seqlen_q x num_heads x head_size
 833:         std::optional<at::Tensor> &dk_,   // batch_size x seqlen_k x num_heads_k x head_size
 834:         std::optional<at::Tensor> &dv_,   // batch_size x seqlen_k x num_heads_k x head_size
 835:         std::optional<at::Tensor> &alibi_slopes_, // num_heads or batch_size x num_heads
 836:         const float p_dropout,         // probability to drop
 837:         const float softmax_scale,
 838:         const bool is_causal,
 839:         int window_size_left,
 840:         int window_size_right,
```
- L801: Declares function `fill_` as part of this file's callable surface. / 声明函数 `fill_`，作为本文件可调用接口的一部分。
- L802: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L804: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L805: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L806: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L807: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L808: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L809: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L810: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L812: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L813: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L815: Defines function `run_mha_bwd` and begins its implementation body. / 定义函数 `run_mha_bwd`，并开始其实现体。
- L816: Defines function `FP16_SWITCH` and begins its implementation body. / 定义函数 `FP16_SWITCH`，并开始其实现体。
- L817: Defines function `HEADDIM_SWITCH` and begins its implementation body. / 定义函数 `HEADDIM_SWITCH`，并开始其实现体。
- L818: Defines function `BOOL_SWITCH` and begins its implementation body. / 定义函数 `BOOL_SWITCH`，并开始其实现体。
- L819: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L820: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L821: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L822: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L823: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L825: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L826: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L827: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L828: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L829: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L830: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L831: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L832: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L833: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L834: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L835: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L836: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L837: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L838: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L839: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L840: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 841-880

```cpp
 841:         const float softcap,
 842:         const bool deterministic,
 843:         const at::Tensor philox_seed,
 844:         const at::Tensor philox_offset) {
 845: 
 846:     #ifdef FLASHATTENTION_DISABLE_BACKWARD
 847:         TORCH_CHECK(false, "This flash attention build does not support backward.");
 848:     #endif
 849:     if (is_causal) { window_size_right = 0; }
 850:     auto dprops = at::cuda::getCurrentDeviceProperties();
 851:     bool is_sm80_or_newer = (dprops->major * 10) >= 80;
 852:     TORCH_CHECK(is_sm80_or_newer, "FlashAttention only supports Ampere GPUs or newer.");
 853: 
 854:     bool is_dropout = p_dropout > 0.0;
 855:     auto stream = at::cuda::getCurrentCUDAStream().stream();
 856: 
 857:     auto q_dtype = q.dtype();
 858:     TORCH_CHECK(q_dtype == at::kHalf || q_dtype == at::kBFloat16,
 859:                 "FlashAttention only support fp16 and bf16 data type");
 860:     if (q_dtype == at::kBFloat16) {
 861:         TORCH_CHECK(is_sm80_or_newer, "bfloat16 is only supported on Ampere GPUs or newer");
 862:     }
 863:     TORCH_CHECK(k.dtype() == q_dtype, "query and key must have the same dtype");
 864:     TORCH_CHECK(v.dtype() == q_dtype, "query and value must have the same dtype");
 865:     TORCH_CHECK(out.dtype() == q_dtype, "query and out must have the same dtype");
 866:     TORCH_CHECK(dout.dtype() == q_dtype, "query and dout must have the same dtype");
 867: 
 868:     CHECK_DEVICE(q); CHECK_DEVICE(k); CHECK_DEVICE(v);
 869:     CHECK_DEVICE(out); CHECK_DEVICE(dout); CHECK_DEVICE(softmax_lse);
 870: 
 871:     TORCH_CHECK(q.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 872:     TORCH_CHECK(k.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 873:     TORCH_CHECK(v.stride(-1) == 1, "Input tensor must have contiguous last dimension");
 874:     TORCH_CHECK(out.stride(-1) == 1, "out tensor must have contiguous last dimension");
 875: 
 876:     const auto sizes = q.sizes();
 877: 
 878:     const int batch_size = sizes[0];
 879:     const int seqlen_q = sizes[1];
 880:     const int num_heads = sizes[2];
```
- L841: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L842: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L843: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L844: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L846: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L847: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L848: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L849: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L850: Declares function `getCurrentDeviceProperties` as part of this file's callable surface. / 声明函数 `getCurrentDeviceProperties`，作为本文件可调用接口的一部分。
- L851: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L852: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L854: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L855: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。
- L857: Declares function `dtype` as part of this file's callable surface. / 声明函数 `dtype`，作为本文件可调用接口的一部分。
- L858: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L859: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L860: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L861: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L862: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L863: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L864: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L865: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L866: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L868: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L869: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L871: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L872: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L873: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L874: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L876: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L878: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L879: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L880: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 881-920

```cpp
 881:     const int head_size_og = dout.size(3);
 882:     const int head_size = sizes[3];
 883:     const int seqlen_k = k.size(1);
 884:     const int num_heads_k = k.size(2);
 885: 
 886:     if (batch_size == 0) {
 887:         auto opts = q.options();
 888:         at::Tensor dq = at::empty_like(q);
 889:         at::Tensor dk = at::empty_like(k);
 890:         at::Tensor dv = at::empty_like(v);
 891:         auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
 892:         const int seqlen_q_rounded = round_multiple(seqlen_q, 128);
 893:         at::Tensor softmax_d = at::empty({0, num_heads, seqlen_q_rounded}, opts.dtype(at::kFloat));
 894:         return {dq, dk, dv, softmax_d};
 895:     }
 896: 
 897:     TORCH_CHECK(dout.stride(-1) == 1, "dout tensor must have contiguous last dimension");
 898: 
 899:     TORCH_CHECK(batch_size > 0, "batch size must be positive");
 900:     TORCH_CHECK(head_size % 8 == 0, "head_size should be a multiple of 8");
 901:     TORCH_CHECK(head_size_og % 8 == 0, "head_size_og should be a multiple of 8, this is ensured by padding!");
 902:     TORCH_CHECK(head_size <= 256, "FlashAttention backward only supports head dimension at most 256");
 903:     if (head_size > 192 && (head_size <= 224 || is_dropout)) {
 904:         TORCH_CHECK(is_sm80_or_newer, "FlashAttention backward for head dim 256 with dropout, or head dim 224 with/without dropout requires A100/A800 or H100/H800");
 905:     }
 906:     TORCH_CHECK(num_heads % num_heads_k == 0, "Number of heads in key/value must divide number of heads in query");
 907: 
 908:     auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
 909:     const int head_size_rounded = round_multiple(head_size, head_size <= 128 ? 32 : 64);
 910:     const int seqlen_q_rounded = round_multiple(seqlen_q, 128);
 911:     const int seqlen_k_rounded = round_multiple(seqlen_k, 128);
 912: 
 913:     TORCH_CHECK(head_size == round_multiple(head_size_og, 8), "head_size must be head_size_og rounded to a multiple of 8");
 914: 
 915:     if (window_size_left >= seqlen_k) { window_size_left = -1; }
 916:     if (window_size_right >= seqlen_k) { window_size_right = -1; }
 917: 
 918:     CHECK_SHAPE(q, batch_size, seqlen_q, num_heads, head_size);
 919:     CHECK_SHAPE(k, batch_size, seqlen_k, num_heads_k, head_size);
 920:     CHECK_SHAPE(v, batch_size, seqlen_k, num_heads_k, head_size);
```
- L881: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L882: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L883: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L884: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L886: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L887: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L888: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L889: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L890: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L891: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L892: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L893: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L894: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L895: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L897: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L899: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L900: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L901: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L902: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L903: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L904: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L905: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L906: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L908: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L909: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L910: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L911: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L913: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L915: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L916: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L918: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L919: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L920: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。

### Lines 921-960

```cpp
 921:     CHECK_SHAPE(out, batch_size, seqlen_q, num_heads, head_size);
 922:     CHECK_SHAPE(dout, batch_size, seqlen_q, num_heads, head_size_og);
 923: 
 924:     at::Tensor dq, dk, dv;
 925:     if (dq_.has_value()) {
 926:         dq = dq_.value();
 927:         TORCH_CHECK(dq.dtype() == q_dtype, "dq must have the same dtype as q");
 928:         CHECK_DEVICE(dq);
 929:         TORCH_CHECK(dq.stride(-1) == 1, "dq must have contiguous last dimension");
 930:         CHECK_SHAPE(dq, batch_size, seqlen_q, num_heads, head_size);
 931:     } else {
 932:         dq = at::empty_like(q);
 933:     }
 934:     if (dk_.has_value()) {
 935:         dk = dk_.value();
 936:         TORCH_CHECK(dk.dtype() == q_dtype, "dk must have the same dtype as q");
 937:         CHECK_DEVICE(dk);
 938:         TORCH_CHECK(dk.stride(-1) == 1, "dk must have contiguous last dimension");
 939:         CHECK_SHAPE(dk, batch_size, seqlen_k, num_heads_k, head_size);
 940:     } else {
 941:         dk = at::empty_like(k);
 942:     }
 943:     if (dv_.has_value()) {
 944:         dv = dv_.value();
 945:         TORCH_CHECK(dv.dtype() == q_dtype, "dv must have the same dtype as q");
 946:         CHECK_DEVICE(dv);
 947:         TORCH_CHECK(dv.stride(-1) == 1, "dv must have contiguous last dimension");
 948:         CHECK_SHAPE(dv, batch_size, seqlen_k, num_heads_k, head_size);
 949:     } else {
 950:         dv = at::empty_like(v);
 951:     }
 952: 
 953:     // bool loop = seqlen_k > blocksize_c;
 954:     // TODO: change later, for now set to true for simplicity
 955:     bool loop = true;
 956: 
 957:     // Otherwise the kernel will be launched from cuda:0 device
 958:     // Cast to char to avoid compiler warning about narrowing
 959:     at::cuda::CUDAGuard device_guard{static_cast<signed char>(q.get_device())};
 960: 
```
- L921: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L922: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L924: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L925: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L926: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L927: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L928: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L929: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L930: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L931: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L932: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L933: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L934: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L935: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L936: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L937: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L938: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L939: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L940: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L941: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L942: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L943: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L944: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L945: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L946: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L947: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L948: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L949: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L950: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L951: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L953: Documents the nearby logic: bool loop = seqlen_k > blocksize_c; / 说明附近逻辑的作用：bool loop = seqlen_k > blocksize_c;
- L954: Documents the nearby logic: TODO: change later, for now set to true for simplicity / 说明附近逻辑的作用：TODO: change later, for now set to true for simplicity
- L955: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L957: Documents the nearby logic: Otherwise the kernel will be launched from cuda:0 device / 说明附近逻辑的作用：Otherwise the kernel will be launched from cuda:0 device
- L958: Documents the nearby logic: Cast to char to avoid compiler warning about narrowing / 说明附近逻辑的作用：Cast to char to avoid compiler warning about narrowing
- L959: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 961-1000

```cpp
 961:     auto opts = q.options();
 962:     auto softmax_d = at::empty({batch_size, num_heads, seqlen_q_rounded}, opts.dtype(at::kFloat));
 963:     at::Tensor dq_accum;
 964:     at::Tensor dk_accum, dv_accum;
 965:     if (loop) {
 966:         if (!deterministic) {
 967:             dq_accum = at::empty({batch_size, seqlen_q_rounded, num_heads, head_size_rounded}, opts.dtype(at::kFloat));
 968:         } else {
 969:             const int nsplits = (dprops->multiProcessorCount + batch_size * num_heads - 1) / (batch_size * num_heads);
 970:             dq_accum = at::zeros({nsplits, batch_size, seqlen_q_rounded, num_heads, head_size_rounded}, opts.dtype(at::kFloat));
 971:         }
 972:         // dk_accum = torch::empty({batch_size, num_heads_k, seqlen_k_rounded, head_size_rounded}, opts.dtype(at::kFloat));
 973:         // dv_accum = torch::empty({batch_size, num_heads_k, seqlen_k_rounded, head_size_rounded}, opts.dtype(at::kFloat));
 974:     }
 975: 
 976:     at::Tensor dk_expanded, dv_expanded;
 977:     if (num_heads_k != num_heads) {  // MQA / GQA
 978:         dk_expanded = at::empty({batch_size, seqlen_k, num_heads, head_size}, opts);
 979:         dv_expanded = at::empty({batch_size, seqlen_k, num_heads, head_size}, opts);
 980:     } else {
 981:         dk_expanded = dk;
 982:         dv_expanded = dv;
 983:     }
 984: 
 985:     Flash_bwd_params params;
 986: 
 987:     set_params_dgrad(params,
 988:                      batch_size,
 989:                      seqlen_q, seqlen_k,
 990:                      seqlen_q_rounded, seqlen_k_rounded,
 991:                      num_heads, num_heads_k,
 992:                      head_size, head_size_rounded,
 993:                      q, k, v, out,
 994:                      dout, dq, dk_expanded, dv_expanded,
 995:                      nullptr,
 996:                      nullptr,
 997:                      loop ? dq_accum.data_ptr() : nullptr,
 998:                      // loop ? dk_accum.data_ptr() : nullptr,
 999:                      // loop ? dv_accum.data_ptr() : nullptr,
1000:                      nullptr,
```
- L961: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L962: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L963: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L964: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L965: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L966: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L967: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L968: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L969: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L970: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。
- L971: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L972: Documents the nearby logic: dk_accum = torch::empty({batch_size, num_heads_k, seqlen_k_rounded, head_size_rounded}, opts.dtype(at::kFloat)); / 说明附近逻辑的作用：dk_accum = torch::empty({batch_size, num_heads_k, seqlen_k_rounded, head_size_rounded}, opts.dtype(at::kFloat));
- L973: Documents the nearby logic: dv_accum = torch::empty({batch_size, num_heads_k, seqlen_k_rounded, head_size_rounded}, opts.dtype(at::kFloat)); / 说明附近逻辑的作用：dv_accum = torch::empty({batch_size, num_heads_k, seqlen_k_rounded, head_size_rounded}, opts.dtype(at::kFloat));
- L974: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L976: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L977: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L978: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L979: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L980: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L981: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L982: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L983: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L985: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L987: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L988: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L989: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L990: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L991: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L992: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L993: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L994: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L995: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L996: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L997: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L998: Documents the nearby logic: loop ? dk_accum.data_ptr() : nullptr, / 说明附近逻辑的作用：loop ? dk_accum.data_ptr() : nullptr,
- L999: Documents the nearby logic: loop ? dv_accum.data_ptr() : nullptr, / 说明附近逻辑的作用：loop ? dv_accum.data_ptr() : nullptr,
- L1000: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 1001-1040

```cpp
1001:                      nullptr,
1002:                      softmax_lse.data_ptr(),
1003:                      softmax_d.data_ptr(),
1004:                      p_dropout,
1005:                      softmax_scale,
1006:                      window_size_left,
1007:                      window_size_right,
1008:                      softcap,
1009:                      deterministic,
1010:                      /*unpadded_lse*/false);
1011:     params.dq_accum_split_stride = !deterministic ? 0 : dq_accum.stride(0);
1012: 
1013:     auto launch = &run_mha_bwd;
1014: 
1015:     at::PhiloxCudaState philox_args;
1016: 
1017:     if (is_dropout) {
1018:         params.rng_state = philox_seed.data_ptr<uint64_t>();
1019:     }
1020:     params.philox_args = philox_args;
1021: 
1022:     set_params_alibi(params, alibi_slopes_, batch_size, num_heads);
1023: 
1024:     if (seqlen_q > 0) {
1025:         launch(params, stream);
1026:     } else {
1027:         // If seqlen_q == 0, then we have an empty tensor. We need to set the output to 0.
1028:         dk_expanded.zero_();
1029:         dv_expanded.zero_();
1030:         softmax_d.zero_();
1031:     }
1032: 
1033:     // For MQA/GQA we need to sum dK and dV across the groups
1034:     if (num_heads_k != num_heads) {
1035:         at::sum_out(dk, at::reshape(dk_expanded, {batch_size, seqlen_k, num_heads_k, num_heads / num_heads_k, head_size}), {3});
1036:         at::sum_out(dv, at::reshape(dv_expanded, {batch_size, seqlen_k, num_heads_k, num_heads / num_heads_k, head_size}), {3});
1037:     }
1038:     return { dq, dk, dv, softmax_d };
1039: }
1040: 
```
- L1001: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1002: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1003: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1004: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1005: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1006: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1007: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1008: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1009: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1010: Documents the nearby logic: unpadded_lse*/false); / 说明附近逻辑的作用：unpadded_lse*/false);
- L1011: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1013: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1015: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1017: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1018: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1019: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1020: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1022: Declares function `set_params_alibi` as part of this file's callable surface. / 声明函数 `set_params_alibi`，作为本文件可调用接口的一部分。
- L1024: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1025: Declares function `launch` as part of this file's callable surface. / 声明函数 `launch`，作为本文件可调用接口的一部分。
- L1026: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1027: Documents the nearby logic: If seqlen_q == 0, then we have an empty tensor. We need to set the output to 0. / 说明附近逻辑的作用：If seqlen_q == 0, then we have an empty tensor. We need to set the output to 0.
- L1028: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1029: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1030: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1031: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1033: Documents the nearby logic: For MQA/GQA we need to sum dK and dV across the groups / 说明附近逻辑的作用：For MQA/GQA we need to sum dK and dV across the groups
- L1034: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1035: Declares function `sum_out` as part of this file's callable surface. / 声明函数 `sum_out`，作为本文件可调用接口的一部分。
- L1036: Declares function `sum_out` as part of this file's callable surface. / 声明函数 `sum_out`，作为本文件可调用接口的一部分。
- L1037: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1038: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1039: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 1041-1080

```cpp
1041: std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor>
1042: mha_varlen_bwd(const at::Tensor &dout,  // total_q x num_heads, x head_size
1043:                const at::Tensor &q,   // total_q x num_heads x head_size, total_q := \sum_{i=0}^{b} s_i
1044:                const at::Tensor &k,   // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i
1045:                const at::Tensor &v,   // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i
1046:                const at::Tensor &out,   // total_q x num_heads x head_size
1047:                const at::Tensor &softmax_lse, // h x total_q, softmax logsumexp
1048:                std::optional<at::Tensor> &dq_,   // total_q x num_heads x head_size, total_q := \sum_{i=0}^{b} s_i
1049:                std::optional<at::Tensor> &dk_,   // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i
1050:                std::optional<at::Tensor> &dv_,   // total_k x num_heads_k x head_size, total_k := \sum_{i=0}^{b} s_i
1051:                const at::Tensor &cu_seqlens_q,  // b+1
1052:                const at::Tensor &cu_seqlens_k,  // b+1
1053:                std::optional<at::Tensor> &alibi_slopes_, // num_heads or b x num_heads
1054:                const int max_seqlen_q,
1055:                const int max_seqlen_k,          // max sequence length to choose the kernel
1056:                const float p_dropout,         // probability to drop
1057:                const float softmax_scale,
1058:                const bool zero_tensors,
1059:                const bool is_causal,
1060:                int window_size_left,
1061:                int window_size_right,
1062:                const float softcap,
1063:                const bool deterministic,
1064:                const at::Tensor philox_seed,
1065:                const at::Tensor philox_offset)
1066: {
1067: 
1068:     #ifdef FLASHATTENTION_DISABLE_BACKWARD
1069:         TORCH_CHECK(false, "This flash attention build does not support backward.");
1070:     #endif
1071: 
1072:     if (is_causal) { window_size_right = 0; }
1073:     auto dprops = at::cuda::getCurrentDeviceProperties();
1074:     bool is_sm80_or_newer = (dprops->major * 10) >= 80;
1075:     TORCH_CHECK(is_sm80_or_newer, "FlashAttention only supports Ampere GPUs or newer.");
1076: 
1077:     bool is_dropout = p_dropout > 0.0;
1078:     auto stream = at::cuda::getCurrentCUDAStream().stream();
1079: 
1080:     auto q_dtype = q.dtype();
```
- L1041: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1042: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1043: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1044: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1045: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1046: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1047: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1048: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1049: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1050: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1051: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1052: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1053: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1054: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1055: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1056: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1057: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1058: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1059: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1060: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1061: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1062: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1063: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1064: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1065: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1066: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1068: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L1069: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1070: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L1072: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1073: Declares function `getCurrentDeviceProperties` as part of this file's callable surface. / 声明函数 `getCurrentDeviceProperties`，作为本文件可调用接口的一部分。
- L1074: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1075: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1077: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1078: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。
- L1080: Declares function `dtype` as part of this file's callable surface. / 声明函数 `dtype`，作为本文件可调用接口的一部分。

### Lines 1081-1120

```cpp
1081:     TORCH_CHECK(q_dtype == at::kHalf || q_dtype == at::kBFloat16,
1082:                 "FlashAttention only support fp16 and bf16 data type");
1083:     if (q_dtype == at::kBFloat16) {
1084:         TORCH_CHECK(is_sm80_or_newer, "bfloat16 is only supported on Ampere GPUs or newer");
1085:     }
1086:     TORCH_CHECK(k.dtype() == q_dtype, "query and key must have the same dtype");
1087:     TORCH_CHECK(v.dtype() == q_dtype, "query and value must have the same dtype");
1088:     TORCH_CHECK(out.dtype() == q_dtype, "query and out must have the same dtype");
1089:     TORCH_CHECK(dout.dtype() == q_dtype, "query and dout must have the same dtype");
1090:     TORCH_CHECK(cu_seqlens_q.dtype() == at::kInt, "cu_seqlens_q must have dtype int32");
1091:     TORCH_CHECK(cu_seqlens_k.dtype() == at::kInt, "cu_seqlens_k must have dtype int32");
1092: 
1093:     CHECK_DEVICE(q); CHECK_DEVICE(k); CHECK_DEVICE(v);
1094:     CHECK_DEVICE(out); CHECK_DEVICE(dout); CHECK_DEVICE(softmax_lse);
1095:     CHECK_DEVICE(cu_seqlens_q); CHECK_DEVICE(cu_seqlens_k);
1096: 
1097:     TORCH_CHECK(q.stride(-1) == 1, "Input tensor must have contiguous last dimension");
1098:     TORCH_CHECK(k.stride(-1) == 1, "Input tensor must have contiguous last dimension");
1099:     TORCH_CHECK(v.stride(-1) == 1, "Input tensor must have contiguous last dimension");
1100:     TORCH_CHECK(out.stride(-1) == 1, "out tensor must have contiguous last dimension");
1101:     TORCH_CHECK(dout.stride(-1) == 1, "dout tensor must have contiguous last dimension");
1102:     CHECK_CONTIGUOUS(cu_seqlens_q);
1103:     CHECK_CONTIGUOUS(cu_seqlens_k);
1104: 
1105:     const auto sizes = q.sizes();
1106: 
1107:     const int total_q = sizes[0];
1108:     const int batch_size = cu_seqlens_q.numel() - 1;
1109:     const int num_heads = sizes[1];
1110:     const int head_size_og = dout.size(2);
1111:     const int head_size = sizes[2];
1112:     const int total_k = k.size(0);
1113:     const int num_heads_k = k.size(1);
1114:     TORCH_CHECK(batch_size > 0, "batch size must be positive");
1115:     TORCH_CHECK(head_size % 8 == 0, "head_size should be a multiple of 8");
1116:     TORCH_CHECK(head_size_og % 8 == 0, "head_size_og should be a multiple of 8, this is ensured by padding!");
1117:     TORCH_CHECK(head_size <= 256, "FlashAttention backward only supports head dimension at most 256");
1118:     if (head_size > 192 && (head_size <= 224 || is_dropout)) {
1119:         TORCH_CHECK(is_sm80_or_newer, "FlashAttention backward for head dim 256 with dropout, or head dim 224 with/without dropout requires A100/A800 or H100/H800");
1120:     }
```
- L1081: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1082: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1083: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1084: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1085: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1086: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1087: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1088: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1089: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1090: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1091: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1093: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1094: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1095: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1097: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1098: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1099: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1100: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1101: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1102: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L1103: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L1105: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1107: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1108: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1109: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1110: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1111: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1112: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1113: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1114: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1115: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1116: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1117: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1118: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1119: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1120: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 1121-1160

```cpp
1121:     TORCH_CHECK(num_heads % num_heads_k == 0, "Number of heads in key/value must divide number of heads in query");
1122: 
1123:     auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
1124:     const int head_size_rounded = round_multiple(head_size, head_size <= 128 ? 32 : 64);
1125:     const int seqlen_q_rounded = round_multiple(max_seqlen_q, 128);
1126:     const int seqlen_k_rounded = round_multiple(max_seqlen_k, 128);
1127: 
1128:     TORCH_CHECK(head_size == round_multiple(head_size_og, 8), "head_size must be head_size_og rounded to a multiple of 8");
1129: 
1130:     if (window_size_left >= max_seqlen_k) { window_size_left = -1; }
1131:     if (window_size_right >= max_seqlen_k) { window_size_right = -1; }
1132: 
1133:     CHECK_SHAPE(q, total_q, num_heads, head_size);
1134:     CHECK_SHAPE(k, total_k, num_heads_k, head_size);
1135:     CHECK_SHAPE(v, total_k, num_heads_k, head_size);
1136:     CHECK_SHAPE(out, total_q, num_heads, head_size);
1137:     CHECK_SHAPE(dout, total_q, num_heads, head_size_og);
1138:     CHECK_SHAPE(cu_seqlens_q, batch_size + 1);
1139:     CHECK_SHAPE(cu_seqlens_k, batch_size + 1);
1140: 
1141:     at::Tensor dq, dk, dv;
1142:     if (dq_.has_value()) {
1143:         dq = dq_.value();
1144:         TORCH_CHECK(dq.dtype() == q_dtype, "dq must have the same dtype as q");
1145:         CHECK_DEVICE(dq);
1146:         TORCH_CHECK(dq.stride(-1) == 1, "dq must have contiguous last dimension");
1147:         CHECK_SHAPE(dq, total_q, num_heads, head_size);
1148:     } else {
1149:         dq = at::empty_like(q);
1150:     }
1151:     if (dk_.has_value()) {
1152:         dk = dk_.value();
1153:         TORCH_CHECK(dk.dtype() == q_dtype, "dk must have the same dtype as q");
1154:         CHECK_DEVICE(dk);
1155:         TORCH_CHECK(dk.stride(-1) == 1, "dk must have contiguous last dimension");
1156:         CHECK_SHAPE(dk, total_k, num_heads_k, head_size);
1157:     } else {
1158:         dk = at::empty_like(k);
1159:     }
1160:     if (dv_.has_value()) {
```
- L1121: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1123: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1124: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1125: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1126: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1128: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1130: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1131: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1133: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1134: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1135: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1136: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1137: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1138: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1139: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1141: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1142: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1143: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1144: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1145: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1146: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1147: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1148: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1149: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1150: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1151: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1152: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1153: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1154: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1155: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1156: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1157: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1158: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1159: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1160: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 1161-1200

```cpp
1161:         dv = dv_.value();
1162:         TORCH_CHECK(dv.dtype() == q_dtype, "dv must have the same dtype as q");
1163:         CHECK_DEVICE(dv);
1164:         TORCH_CHECK(dv.stride(-1) == 1, "dv must have contiguous last dimension");
1165:         CHECK_SHAPE(dv, total_k, num_heads_k, head_size);
1166:     } else {
1167:         dv = at::empty_like(v);
1168:     }
1169: 
1170:     // bool loop = max_seqlen_k > blocksize_c;
1171:     // TODO: change later, for now set to true for simplicity
1172:     bool loop = true;
1173: 
1174:     // Otherwise the kernel will be launched from cuda:0 device
1175:     // Cast to char to avoid compiler warning about narrowing
1176:     at::cuda::CUDAGuard device_guard{static_cast<signed char>(q.get_device())};
1177: 
1178:     auto opts = q.options();
1179:     auto softmax_d = at::empty({num_heads, total_q + 128 * batch_size}, opts.dtype(at::kFloat));
1180:     at::Tensor dq_accum;
1181:     if (loop) {
1182:         // We don't want to allocate dq_accum of size (batch, seqlen_q_rounded, num_heads, head_size_rounded)
1183:         // because that would be too large if there is a very long sequence and the rest of the sequences are short.
1184:         // Instead, we allocate dq_accum of size (total_q + 128 * batch, num_heads, head_size_rounded).
1185:         // Note that 128 is the max block size on the seqlen_q dimension.
1186:         // For dQ, the i-th sequence is stored in indices from cu_seqlens[i] + 128 * i to
1187:         // cu_seqlens[i + 1] * 128 * i - 1. This ensures that the i-th sequence and (i + 1)-th sequence will
1188:         // be at least 128 apart. It's ok for us to do atomicAdds up to 128 rows beyond what we're normally
1189:         // allowed to do. So we won't have to do any bound checking, and performance should stay the same.
1190:         // Same holds for softmax_d, since LSE is stored in unpadded format.
1191:         if (!deterministic) {
1192:             dq_accum = at::empty({total_q + 128 * batch_size, num_heads, head_size_rounded}, opts.dtype(at::kFloat));
1193:         } else {
1194:             const int nsplits = (dprops->multiProcessorCount + batch_size * num_heads - 1) / (batch_size * num_heads);
1195:             dq_accum = at::zeros({nsplits, total_q + 128 * batch_size, num_heads, head_size_rounded}, opts.dtype(at::kFloat));
1196:         }
1197:     }
1198: 
1199:     at::Tensor dk_expanded, dv_expanded;
1200:     if (num_heads_k != num_heads) {  // MQA / GQA
```
- L1161: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1162: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1163: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1164: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1165: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1166: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1167: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1168: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1170: Documents the nearby logic: bool loop = max_seqlen_k > blocksize_c; / 说明附近逻辑的作用：bool loop = max_seqlen_k > blocksize_c;
- L1171: Documents the nearby logic: TODO: change later, for now set to true for simplicity / 说明附近逻辑的作用：TODO: change later, for now set to true for simplicity
- L1172: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1174: Documents the nearby logic: Otherwise the kernel will be launched from cuda:0 device / 说明附近逻辑的作用：Otherwise the kernel will be launched from cuda:0 device
- L1175: Documents the nearby logic: Cast to char to avoid compiler warning about narrowing / 说明附近逻辑的作用：Cast to char to avoid compiler warning about narrowing
- L1176: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1178: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L1179: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1180: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1181: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1182: Documents the nearby logic: We don't want to allocate dq_accum of size (batch, seqlen_q_rounded, num_heads, head_size_rounded) / 说明附近逻辑的作用：We don't want to allocate dq_accum of size (batch, seqlen_q_rounded, num_heads, head_size_rounded)
- L1183: Documents the nearby logic: because that would be too large if there is a very long sequence and the rest of the sequences are short. / 说明附近逻辑的作用：because that would be too large if there is a very long sequence and the rest of the sequences are short.
- L1184: Documents the nearby logic: Instead, we allocate dq_accum of size (total_q + 128 * batch, num_heads, head_size_rounded). / 说明附近逻辑的作用：Instead, we allocate dq_accum of size (total_q + 128 * batch, num_heads, head_size_rounded).
- L1185: Documents the nearby logic: Note that 128 is the max block size on the seqlen_q dimension. / 说明附近逻辑的作用：Note that 128 is the max block size on the seqlen_q dimension.
- L1186: Documents the nearby logic: For dQ, the i-th sequence is stored in indices from cu_seqlens[i] + 128 * i to / 说明附近逻辑的作用：For dQ, the i-th sequence is stored in indices from cu_seqlens[i] + 128 * i to
- L1187: Documents the nearby logic: cu_seqlens[i + 1] * 128 * i - 1. This ensures that the i-th sequence and (i + 1)-th sequence will / 说明附近逻辑的作用：cu_seqlens[i + 1] * 128 * i - 1. This ensures that the i-th sequence and (i + 1)-th sequence will
- L1188: Documents the nearby logic: be at least 128 apart. It's ok for us to do atomicAdds up to 128 rows beyond what we're normally / 说明附近逻辑的作用：be at least 128 apart. It's ok for us to do atomicAdds up to 128 rows beyond what we're normally
- L1189: Documents the nearby logic: allowed to do. So we won't have to do any bound checking, and performance should stay the same. / 说明附近逻辑的作用：allowed to do. So we won't have to do any bound checking, and performance should stay the same.
- L1190: Documents the nearby logic: Same holds for softmax_d, since LSE is stored in unpadded format. / 说明附近逻辑的作用：Same holds for softmax_d, since LSE is stored in unpadded format.
- L1191: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1192: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1193: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1194: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1195: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。
- L1196: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1197: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1199: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1200: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 1201-1240

```cpp
1201:         dk_expanded = at::empty({total_k, num_heads, head_size}, opts);
1202:         dv_expanded = at::empty({total_k, num_heads, head_size}, opts);
1203:     } else {
1204:         dk_expanded = dk;
1205:         dv_expanded = dv;
1206:     }
1207: 
1208:     if( zero_tensors ) {
1209:         dq.zero_();
1210:         dk_expanded.zero_();
1211:         dv_expanded.zero_();
1212:         softmax_d.zero_();
1213:     }
1214: 
1215:     Flash_bwd_params params;
1216: 
1217:     set_params_dgrad(params,
1218:                      batch_size,
1219:                      max_seqlen_q, max_seqlen_k,
1220:                      seqlen_q_rounded, seqlen_k_rounded,
1221:                      num_heads, num_heads_k,
1222:                      head_size, head_size_rounded,
1223:                      q, k, v, out,
1224:                      dout, dq, dk_expanded, dv_expanded,
1225:                      cu_seqlens_q.data_ptr(),
1226:                      cu_seqlens_k.data_ptr(),
1227:                      loop ? dq_accum.data_ptr() : nullptr,
1228:                      nullptr,
1229:                      nullptr,
1230:                      softmax_lse.data_ptr(),
1231:                      softmax_d.data_ptr(),
1232:                      p_dropout,
1233:                      softmax_scale,
1234:                      window_size_left,
1235:                      window_size_right,
1236:                      softcap,
1237:                      deterministic,
1238:                      /*unpadded_lse*/true);
1239:     params.total_q = total_q;;
1240:     params.dq_accum_split_stride = !deterministic ? 0 : dq_accum.stride(0);
```
- L1201: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1202: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1203: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1204: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1205: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1206: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1208: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1209: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1210: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1211: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1212: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1213: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1215: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1217: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1218: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1219: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1220: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1221: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1222: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1223: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1224: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1225: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1226: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1227: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1228: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1229: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1230: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1231: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1232: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1233: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1234: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1235: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1236: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1237: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1238: Documents the nearby logic: unpadded_lse*/true); / 说明附近逻辑的作用：unpadded_lse*/true);
- L1239: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1240: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。

### Lines 1241-1280

```cpp
1241: 
1242:     auto launch = &run_mha_bwd;
1243: 
1244:     at::PhiloxCudaState philox_args;
1245:     if (is_dropout) {
1246:         params.rng_state = philox_seed.data_ptr<uint64_t>();
1247:     }
1248:     params.philox_args = philox_args;
1249: 
1250:     set_params_alibi(params, alibi_slopes_, batch_size, num_heads);
1251: 
1252:     if (max_seqlen_q > 0) {
1253:         launch(params, stream);
1254:     } else {
1255:         // If seqlen_q == 0, then we have an empty tensor. We need to set the output to 0.
1256:         dk_expanded.zero_();
1257:         dv_expanded.zero_();
1258:         softmax_d.zero_();
1259:     }
1260: 
1261:     // For MQA/GQA we need to sum dK and dV across the groups
1262:     if (num_heads_k != num_heads) {
1263:         at::sum_out(dk, at::reshape(dk_expanded, {total_k, num_heads_k, num_heads / num_heads_k, head_size}), {2});
1264:         at::sum_out(dv, at::reshape(dv_expanded, {total_k, num_heads_k, num_heads / num_heads_k, head_size}), {2});
1265:     }
1266: 
1267:     return { dq, dk, dv, softmax_d };
1268: }
1269: 
1270: std::tuple<at::Tensor, at::Tensor>
1271: mha_fwd_kvcache(at::Tensor &q,                 // batch_size x seqlen_q x num_heads x head_size
1272:                 const at::Tensor &kcache,            // batch_size_c x seqlen_k x num_heads_k x head_size or num_blocks x page_block_size x num_heads_k x head_size if there's a block_table.
1273:                 const at::Tensor &vcache,            // batch_size_c x seqlen_k x num_heads_k x head_size or num_blocks x page_block_size x num_heads_k x head_size if there's a block_table.
1274:                 std::optional<const at::Tensor> &k_, // batch_size x seqlen_knew x num_heads_k x head_size
1275:                 std::optional<const at::Tensor> &v_, // batch_size x seqlen_knew x num_heads_k x head_size
1276:                 std::optional<const at::Tensor> &seqlens_k_, // batch_size
1277:                 std::optional<const at::Tensor> &rotary_cos_, // seqlen_ro x (rotary_dim / 2)
1278:                 std::optional<const at::Tensor> &rotary_sin_, // seqlen_ro x (rotary_dim / 2)
1279:                 std::optional<const at::Tensor> &cache_batch_idx_, // indices to index into the KV cache
1280:                 std::optional<at::Tensor> &block_table_, // batch_size x max_num_blocks_per_seq
```
- L1242: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1244: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1245: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1246: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1247: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1248: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1250: Declares function `set_params_alibi` as part of this file's callable surface. / 声明函数 `set_params_alibi`，作为本文件可调用接口的一部分。
- L1252: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1253: Declares function `launch` as part of this file's callable surface. / 声明函数 `launch`，作为本文件可调用接口的一部分。
- L1254: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1255: Documents the nearby logic: If seqlen_q == 0, then we have an empty tensor. We need to set the output to 0. / 说明附近逻辑的作用：If seqlen_q == 0, then we have an empty tensor. We need to set the output to 0.
- L1256: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1257: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1258: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L1259: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1261: Documents the nearby logic: For MQA/GQA we need to sum dK and dV across the groups / 说明附近逻辑的作用：For MQA/GQA we need to sum dK and dV across the groups
- L1262: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1263: Declares function `sum_out` as part of this file's callable surface. / 声明函数 `sum_out`，作为本文件可调用接口的一部分。
- L1264: Declares function `sum_out` as part of this file's callable surface. / 声明函数 `sum_out`，作为本文件可调用接口的一部分。
- L1265: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1267: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1268: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1270: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1271: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1272: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1273: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1274: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1275: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1276: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1277: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1278: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1279: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1280: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 1281-1320

```cpp
1281:                 std::optional<at::Tensor> &alibi_slopes_, // num_heads or batch_size x num_heads
1282:                 std::optional<at::Tensor> &out_,             // batch_size x seqlen_q x num_heads x head_size
1283:                 const float softmax_scale,
1284:                 bool is_causal,
1285:                 int window_size_left,
1286:                 int window_size_right,
1287:                 const float softcap,
1288:                 bool is_rotary_interleaved,   // if true, rotary combines indices 0 & 1, else indices 0 & rotary_dim / 2
1289:                 int num_splits
1290:                 ) {
1291: 
1292:     auto dprops = at::cuda::getCurrentDeviceProperties();
1293:     bool is_sm80_or_newer = (dprops->major * 10) >= 80;
1294:     TORCH_CHECK(is_sm80_or_newer, "FlashAttention only supports Ampere GPUs or newer.");
1295: 
1296:     auto q_dtype = q.dtype();
1297:     TORCH_CHECK(q_dtype == at::kHalf || q_dtype == at::kBFloat16,
1298:                 "FlashAttention only support fp16 and bf16 data type");
1299:     if (q_dtype == at::kBFloat16) {
1300:         TORCH_CHECK(is_sm80_or_newer, "bfloat16 is only supported on Ampere GPUs or newer");
1301:     }
1302:     TORCH_CHECK(kcache.dtype() == q_dtype, "query and key must have the same dtype");
1303:     TORCH_CHECK(vcache.dtype() == q_dtype, "query and value must have the same dtype");
1304: 
1305:     CHECK_DEVICE(q); CHECK_DEVICE(kcache); CHECK_DEVICE(vcache);
1306: 
1307:     TORCH_CHECK(q.stride(-1) == 1, "Input tensor must have contiguous last dimension");
1308:     TORCH_CHECK(kcache.stride(-1) == 1, "Input tensor must have contiguous last dimension");
1309:     TORCH_CHECK(vcache.stride(-1) == 1, "Input tensor must have contiguous last dimension");
1310: 
1311:     at::Tensor block_table;
1312:     const bool paged_KV = block_table_.has_value();
1313:     if (paged_KV) {
1314:         TORCH_CHECK(!cache_batch_idx_.has_value(), "Paged KVcache does not support cache_batch_idx");
1315:         block_table = block_table_.value();
1316:         CHECK_DEVICE(block_table);
1317:         TORCH_CHECK(block_table.dtype() == at::kInt, "block_table must have dtype torch.int32");
1318:         TORCH_CHECK(block_table.stride(-1) == 1, "block_table must have contiguous last dimension");
1319:     }
1320: 
```
- L1281: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1282: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1283: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1284: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1285: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1286: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1287: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1288: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1289: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1290: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1292: Declares function `getCurrentDeviceProperties` as part of this file's callable surface. / 声明函数 `getCurrentDeviceProperties`，作为本文件可调用接口的一部分。
- L1293: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1294: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1296: Declares function `dtype` as part of this file's callable surface. / 声明函数 `dtype`，作为本文件可调用接口的一部分。
- L1297: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1298: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1299: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1300: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1301: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1302: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1303: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1305: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1307: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1308: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1309: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1311: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1312: Declares function `has_value` as part of this file's callable surface. / 声明函数 `has_value`，作为本文件可调用接口的一部分。
- L1313: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1314: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1315: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1316: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1317: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1318: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1319: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 1321-1360

```cpp
1321:     const auto sizes = q.sizes();
1322: 
1323:     const int batch_size = sizes[0];
1324:     int seqlen_q = sizes[1];
1325:     int num_heads = sizes[2];
1326:     const int head_size_og = sizes[3];
1327: 
1328:     const int max_num_blocks_per_seq = !paged_KV ? 0 : block_table.size(1);
1329:     const int num_blocks = !paged_KV ? 0 : kcache.size(0);
1330:     const int page_block_size = !paged_KV ? 1 : kcache.size(1);
1331:     TORCH_CHECK(!paged_KV || page_block_size % 256 == 0, "Paged KV cache block size must be divisible by 256");
1332:     const int seqlen_k = !paged_KV ? kcache.size(1) : max_num_blocks_per_seq * page_block_size;
1333:     const int num_heads_k = kcache.size(2);
1334:     const int batch_size_c = !paged_KV ? kcache.size(0) : batch_size;
1335:     TORCH_CHECK(batch_size > 0, "batch size must be positive");
1336:     TORCH_CHECK(head_size_og <= 256, "FlashAttention forward only supports head dimension at most 256");
1337:     TORCH_CHECK(num_heads % num_heads_k == 0, "Number of heads in key/value must divide number of heads in query");
1338: 
1339:     // causal=true is the same as causal=false in this case
1340:     if (seqlen_q == 1 && !alibi_slopes_.has_value()) { is_causal = false; }
1341:     if (is_causal) { window_size_right = 0; }
1342: 
1343:     // Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case
1344:     // H/t Daniel Haziza
1345:     const int seqlenq_ngroups_swapped = seqlen_q == 1 && num_heads > num_heads_k && window_size_left < 0 && window_size_right < 0 && head_size_og % 8 == 0 && !alibi_slopes_.has_value();
1346:     if (seqlenq_ngroups_swapped) {
1347:         const int ngroups = num_heads / num_heads_k;
1348:         q = q.reshape({batch_size, num_heads_k, ngroups, head_size_og}).transpose(1, 2);
1349:         seqlen_q = ngroups;
1350:         num_heads = num_heads_k;
1351:     }
1352: 
1353:     if (window_size_left >= seqlen_k) { window_size_left = -1; }
1354:     if (window_size_right >= seqlen_k) { window_size_right = -1; }
1355: 
1356:     CHECK_SHAPE(q, batch_size, seqlen_q, num_heads, head_size_og);
1357:     if (!paged_KV) {
1358:         CHECK_SHAPE(kcache, batch_size_c, seqlen_k, num_heads_k, head_size_og);
1359:         CHECK_SHAPE(vcache, batch_size_c, seqlen_k, num_heads_k, head_size_og);
1360:     } else {
```
- L1321: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1323: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1324: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1325: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1326: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1328: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1329: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1330: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1331: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1332: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1333: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1334: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1335: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1336: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1337: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1339: Documents the nearby logic: causal=true is the same as causal=false in this case / 说明附近逻辑的作用：causal=true is the same as causal=false in this case
- L1340: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1341: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1343: Documents the nearby logic: Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case / 说明附近逻辑的作用：Faster to transpose q from (b, 1, (nheads_kv ngroups), d) to (b, ngroups, nheads_kv, d) in this case
- L1344: Documents the nearby logic: H/t Daniel Haziza / 说明附近逻辑的作用：H/t Daniel Haziza
- L1345: Declares function `has_value` as part of this file's callable surface. / 声明函数 `has_value`，作为本文件可调用接口的一部分。
- L1346: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1347: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1348: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L1349: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1350: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1351: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1353: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1354: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1356: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1357: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1358: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1359: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1360: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。

### Lines 1361-1400

```cpp
1361:         CHECK_SHAPE(kcache, num_blocks, page_block_size, num_heads_k, head_size_og);
1362:         CHECK_SHAPE(vcache, num_blocks, page_block_size, num_heads_k, head_size_og);
1363:         CHECK_SHAPE(block_table, batch_size, max_num_blocks_per_seq);
1364:     }
1365: 
1366:     at::Tensor q_padded, kcache_padded, vcache_padded;
1367:     if (head_size_og % 8 != 0) {
1368:         q_padded = at::pad(q, {0, 8 - head_size_og % 8});
1369:         kcache_padded = at::pad(kcache, {0, 8 - head_size_og % 8});
1370:         vcache_padded = at::pad(vcache, {0, 8 - head_size_og % 8});
1371:         // q_padded = at::nn::functional::pad(q, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
1372:         // kcache_padded = at::nn::functional::pad(kcache, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
1373:         // vcache_padded = at::nn::functional::pad(vcache, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
1374:     } else {
1375:         q_padded = q;
1376:         kcache_padded = kcache;
1377:         vcache_padded = vcache;
1378:     }
1379: 
1380:     at::Tensor out;
1381:     if (out_.has_value()) {
1382:         out = out_.value();
1383:         TORCH_CHECK(out.dtype() == q_dtype, "Output must have the same dtype as inputs");
1384:         CHECK_DEVICE(out);
1385:         TORCH_CHECK(out.stride(-1) == 1, "Output tensor must have contiguous last dimension");
1386:         CHECK_SHAPE(out, batch_size, seqlen_q, num_heads, head_size_og);
1387:         if (head_size_og % 8 != 0) { out = at::empty_like(q_padded); }
1388:     } else {
1389:         out = at::empty_like(q_padded);
1390:     }
1391: 
1392:     auto round_multiple = [](int x, int m) { return (x + m - 1) / m * m; };
1393:     const int head_size = round_multiple(head_size_og, 8);
1394:     const int head_size_rounded = round_multiple(head_size, head_size <= 128 ? 32 : 64);
1395:     const int seqlen_q_rounded = round_multiple(seqlen_q, 128);
1396:     const int seqlen_k_rounded = round_multiple(seqlen_k, 128);
1397: 
1398:     // Otherwise the kernel will be launched from cuda:0 device
1399:     // Cast to char to avoid compiler warning about narrowing
1400:     at::cuda::CUDAGuard device_guard{static_cast<signed char>(q.get_device())};
```
- L1361: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1362: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1363: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1364: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1366: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1367: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1368: Declares function `pad` as part of this file's callable surface. / 声明函数 `pad`，作为本文件可调用接口的一部分。
- L1369: Declares function `pad` as part of this file's callable surface. / 声明函数 `pad`，作为本文件可调用接口的一部分。
- L1370: Declares function `pad` as part of this file's callable surface. / 声明函数 `pad`，作为本文件可调用接口的一部分。
- L1371: Documents the nearby logic: q_padded = at::nn::functional::pad(q, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8})); / 说明附近逻辑的作用：q_padded = at::nn::functional::pad(q, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
- L1372: Documents the nearby logic: kcache_padded = at::nn::functional::pad(kcache, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8})); / 说明附近逻辑的作用：kcache_padded = at::nn::functional::pad(kcache, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
- L1373: Documents the nearby logic: vcache_padded = at::nn::functional::pad(vcache, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8})); / 说明附近逻辑的作用：vcache_padded = at::nn::functional::pad(vcache, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
- L1374: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1375: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1376: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1377: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1378: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1380: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1381: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1382: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1383: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1384: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1385: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1386: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1387: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1388: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1389: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1390: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1392: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1393: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1394: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1395: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1396: Declares function `round_multiple` as part of this file's callable surface. / 声明函数 `round_multiple`，作为本文件可调用接口的一部分。
- L1398: Documents the nearby logic: Otherwise the kernel will be launched from cuda:0 device / 说明附近逻辑的作用：Otherwise the kernel will be launched from cuda:0 device
- L1399: Documents the nearby logic: Cast to char to avoid compiler warning about narrowing / 说明附近逻辑的作用：Cast to char to avoid compiler warning about narrowing
- L1400: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 1401-1440

```cpp
1401: 
1402:     auto opts = q.options();
1403: 
1404:     auto softmax_lse = at::empty({batch_size, num_heads, seqlen_q}, opts.dtype(at::kFloat));
1405: 
1406:     Flash_fwd_params params;
1407:     set_params_fprop(params,
1408:                      batch_size,
1409:                      seqlen_q, seqlen_k,
1410:                      seqlen_q_rounded, seqlen_k_rounded,
1411:                      num_heads, num_heads_k,
1412:                      head_size, head_size_rounded,
1413:                      q_padded, kcache_padded, vcache_padded, out,
1414:                      /*cu_seqlens_q_d=*/nullptr,
1415:                      /*cu_seqlens_k_d=*/nullptr,
1416:                      /*seqused_k=*/nullptr,
1417:                      /*p_ptr=*/nullptr,
1418:                      softmax_lse.data_ptr(),
1419:                      /*p_dropout=*/0.f,
1420:                      softmax_scale,
1421:                      window_size_left,
1422:                      window_size_right,
1423:                      softcap
1424:                      );
1425: 
1426:     at::Tensor k, v, k_padded, v_padded;
1427:     if (k_.has_value()) {
1428:         TORCH_CHECK(v_.has_value(), "If key is supplied, value must also be passed in");
1429:         TORCH_CHECK(seqlens_k_.has_value(), "If key is supplied, seqlens_k must also be passed in");
1430:         TORCH_CHECK(seqlen_q <= seqlen_k, "If key is supplied, it must have seqlen <= the seqlen of the KV cache");
1431:         k = k_.value();
1432:         v = v_.value();
1433:         TORCH_CHECK(k.dtype() == q_dtype, "Key must have the same dtype as query");
1434:         TORCH_CHECK(v.dtype() == q_dtype, "Value must have the same dtype as query");
1435:         CHECK_DEVICE(k); CHECK_DEVICE(v);
1436:         TORCH_CHECK(k.stride(-1) == 1, "Key tensor must have contiguous last dimension");
1437:         TORCH_CHECK(v.stride(-1) == 1, "Value tensor must have contiguous last dimension");
1438:         int seqlen_knew = k.size(1);
1439:         CHECK_SHAPE(k, batch_size, seqlen_knew, num_heads_k, head_size_og);
1440:         CHECK_SHAPE(v, batch_size, seqlen_knew, num_heads_k, head_size_og);
```
- L1402: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L1404: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1406: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1407: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1408: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1409: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1410: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1411: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1412: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1413: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1414: Documents the nearby logic: cu_seqlens_q_d=*/nullptr, / 说明附近逻辑的作用：cu_seqlens_q_d=*/nullptr,
- L1415: Documents the nearby logic: cu_seqlens_k_d=*/nullptr, / 说明附近逻辑的作用：cu_seqlens_k_d=*/nullptr,
- L1416: Documents the nearby logic: seqused_k=*/nullptr, / 说明附近逻辑的作用：seqused_k=*/nullptr,
- L1417: Documents the nearby logic: p_ptr=*/nullptr, / 说明附近逻辑的作用：p_ptr=*/nullptr,
- L1418: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1419: Documents the nearby logic: p_dropout=*/0.f, / 说明附近逻辑的作用：p_dropout=*/0.f,
- L1420: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1421: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1422: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1423: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1424: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1426: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1427: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1428: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1429: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1430: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1431: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1432: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1433: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1434: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1435: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1436: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1437: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1438: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1439: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1440: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。

### Lines 1441-1480

```cpp
1441:         if (head_size_og % 8 != 0) {
1442:             k_padded = at::pad(k, {0, 8 - head_size_og % 8});
1443:             v_padded = at::pad(v, {0, 8 - head_size_og % 8});
1444:             // k_padded = at::nn::functional::pad(k, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
1445:             // v_padded = at::nn::functional::pad(v, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
1446:         } else {
1447:             k_padded = k;
1448:             v_padded = v;
1449:         }
1450:         params.seqlen_knew = seqlen_knew;
1451:         params.knew_ptr = k_padded.data_ptr();
1452:         params.vnew_ptr = v_padded.data_ptr();
1453:         // All stride are in elements, not bytes.
1454:         params.knew_batch_stride = k_padded.stride(0);
1455:         params.vnew_batch_stride = v_padded.stride(0);
1456:         params.knew_row_stride = k_padded.stride(-3);
1457:         params.vnew_row_stride = v_padded.stride(-3);
1458:         params.knew_head_stride = k_padded.stride(-2);
1459:         params.vnew_head_stride = v_padded.stride(-2);
1460:     }
1461: 
1462:     if (seqlens_k_.has_value()) {
1463:         auto seqlens_k = seqlens_k_.value();
1464:         TORCH_CHECK(seqlens_k.dtype() == at::kInt, "seqlens_k must have dtype int32");
1465:         CHECK_DEVICE(seqlens_k);
1466:         CHECK_CONTIGUOUS(seqlens_k);
1467:         CHECK_SHAPE(seqlens_k, batch_size);
1468:         params.cu_seqlens_k = static_cast<int *>(seqlens_k.data_ptr());
1469:     }
1470:     params.is_seqlens_k_cumulative = !(seqlens_k_.has_value());
1471: 
1472:     if (rotary_cos_.has_value()) {
1473:         TORCH_CHECK(k_.has_value(), "If rotary cos/sin are provided, new key / value to be appended to KV cache must also be provided");
1474:         auto rotary_cos = rotary_cos_.value();
1475:         CHECK_DEVICE(rotary_cos);
1476:         params.rotary_dim = rotary_cos.size(1) * 2;
1477:         TORCH_CHECK(params.rotary_dim <= head_size, "rotary_dim must be <= headdim");
1478:         TORCH_CHECK(params.rotary_dim % 16 == 0, "Only rotary dimensions divisible by 16 are currently supported");
1479:         const int seqlen_ro = rotary_cos.size(0);
1480:         TORCH_CHECK(seqlen_ro >= seqlen_k, "cos/sin seqlen must be at least the seqlen of KV cache");
```
- L1441: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1442: Declares function `pad` as part of this file's callable surface. / 声明函数 `pad`，作为本文件可调用接口的一部分。
- L1443: Declares function `pad` as part of this file's callable surface. / 声明函数 `pad`，作为本文件可调用接口的一部分。
- L1444: Documents the nearby logic: k_padded = at::nn::functional::pad(k, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8})); / 说明附近逻辑的作用：k_padded = at::nn::functional::pad(k, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
- L1445: Documents the nearby logic: v_padded = at::nn::functional::pad(v, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8})); / 说明附近逻辑的作用：v_padded = at::nn::functional::pad(v, at::nn::functional::PadFuncOptions({0, 8 - head_size_og % 8}));
- L1446: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1447: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1448: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1449: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1450: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1451: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L1452: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L1453: Documents the nearby logic: All stride are in elements, not bytes. / 说明附近逻辑的作用：All stride are in elements, not bytes.
- L1454: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1455: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1456: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1457: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1458: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1459: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1460: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1462: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1463: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1464: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1465: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1466: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L1467: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1468: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L1469: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1470: Declares function `has_value` as part of this file's callable surface. / 声明函数 `has_value`，作为本文件可调用接口的一部分。
- L1472: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1473: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1474: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1475: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1476: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1477: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1478: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1479: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1480: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。

### Lines 1481-1520

```cpp
1481:         CHECK_SHAPE(rotary_cos, seqlen_ro, params.rotary_dim / 2);
1482:         CHECK_CONTIGUOUS(rotary_cos);
1483:         TORCH_CHECK(rotary_cos.scalar_type() == q_dtype, "rotary_cos must have the same dtype as query");
1484: 
1485:         TORCH_CHECK(rotary_sin_.has_value(), "If rotary cos is provided, rotary sin must also be provided");
1486:         auto rotary_sin = rotary_sin_.value();
1487:         CHECK_DEVICE(rotary_sin);
1488:         CHECK_SHAPE(rotary_sin, seqlen_ro, params.rotary_dim / 2);
1489:         CHECK_CONTIGUOUS(rotary_sin);
1490:         TORCH_CHECK(rotary_sin.scalar_type() == q_dtype, "rotary_cos must have the same dtype as query");
1491:         params.rotary_cos_ptr = rotary_cos.data_ptr();
1492:         params.rotary_sin_ptr = rotary_sin.data_ptr();
1493:         params.is_rotary_interleaved = is_rotary_interleaved;
1494:     } else {
1495:         params.rotary_dim = 0;
1496:     }
1497: 
1498:     if (cache_batch_idx_.has_value()) {
1499:         auto cache_batch_idx = cache_batch_idx_.value();
1500:         CHECK_DEVICE(cache_batch_idx);
1501:         CHECK_CONTIGUOUS(cache_batch_idx);
1502:         TORCH_CHECK(cache_batch_idx.scalar_type() == at::kInt, "cache_batch_idx must have dtype int32");
1503:         params.cache_batch_idx = reinterpret_cast<int *>(cache_batch_idx.data_ptr());
1504:     }
1505: 
1506:     // Keep references to these tensors to extend their lifetime
1507:     auto [softmax_lse_accum, out_accum] = set_params_splitkv(params, batch_size, num_heads,
1508:                        head_size, seqlen_k, seqlen_q,
1509:                        head_size_rounded, /*dropout*/0.f, num_splits, dprops, opts);
1510: 
1511:     if (paged_KV) {
1512:         params.block_table = block_table.data_ptr<int>();
1513:         params.block_table_batch_stride = block_table.stride(0);
1514:     }
1515:     params.page_block_size = page_block_size;
1516: 
1517: 
1518:     set_params_alibi(params, alibi_slopes_, batch_size, num_heads);
1519: 
1520:     auto stream = at::cuda::getCurrentCUDAStream().stream();
```
- L1481: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1482: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L1483: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1485: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1486: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1487: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1488: Declares function `CHECK_SHAPE` as part of this file's callable surface. / 声明函数 `CHECK_SHAPE`，作为本文件可调用接口的一部分。
- L1489: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L1490: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1491: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L1492: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L1493: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1494: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1495: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1496: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1498: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1499: Declares function `value` as part of this file's callable surface. / 声明函数 `value`，作为本文件可调用接口的一部分。
- L1500: Declares function `CHECK_DEVICE` as part of this file's callable surface. / 声明函数 `CHECK_DEVICE`，作为本文件可调用接口的一部分。
- L1501: Declares function `CHECK_CONTIGUOUS` as part of this file's callable surface. / 声明函数 `CHECK_CONTIGUOUS`，作为本文件可调用接口的一部分。
- L1502: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1503: Declares function `data_ptr` as part of this file's callable surface. / 声明函数 `data_ptr`，作为本文件可调用接口的一部分。
- L1504: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1506: Documents the nearby logic: Keep references to these tensors to extend their lifetime / 说明附近逻辑的作用：Keep references to these tensors to extend their lifetime
- L1507: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1508: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1509: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1511: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1512: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1513: Declares function `stride` as part of this file's callable surface. / 声明函数 `stride`，作为本文件可调用接口的一部分。
- L1514: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1515: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1518: Declares function `set_params_alibi` as part of this file's callable surface. / 声明函数 `set_params_alibi`，作为本文件可调用接口的一部分。
- L1520: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。

### Lines 1521-1548

```cpp
1521:     // Only split kernel supports appending to KV cache, or indexing to the cache with cache_batch_idx,
1522:     // or paged KV cache
1523:     run_mha_fwd(params, stream, /*force_split_kernel=*/k_.has_value() || cache_batch_idx_.has_value() || paged_KV);
1524: 
1525:     if (head_size_og % 8 != 0) {
1526:         // out = out.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)});
1527:         out = out.narrow(-1, 0, head_size_og);
1528:         if (out_.has_value()) { out_.value().copy_(out); }
1529:         if (k_.has_value()) {
1530:             // It's expensive to copy the KV cache here for the case where head size not divisible by 8,
1531:             // but we don't expect to get this case in practice. This is just so that the code works for that case.
1532:             kcache.copy_(kcache_padded.narrow(-1, 0, head_size_og));
1533:             vcache.copy_(vcache_padded.narrow(-1, 0, head_size_og));
1534:             // kcache.copy_(kcache_padded.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)}));
1535:             // vcache.copy_(vcache_padded.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)}));
1536:         }
1537:     }
1538: 
1539:     if (seqlenq_ngroups_swapped) {
1540:         out = out.transpose(1, 2).reshape({batch_size, 1, num_heads_k * seqlen_q, head_size_og});
1541:         softmax_lse = softmax_lse.reshape({batch_size, num_heads_k * seqlen_q, 1});
1542:     }
1543:     return {out, softmax_lse};
1544: }
1545: 
1546: } // namespace pytorch_fmha
1547: 
1548: #endif
```
- L1521: Documents the nearby logic: Only split kernel supports appending to KV cache, or indexing to the cache with cache_batch_idx, / 说明附近逻辑的作用：Only split kernel supports appending to KV cache, or indexing to the cache with cache_batch_idx,
- L1522: Documents the nearby logic: or paged KV cache / 说明附近逻辑的作用：or paged KV cache
- L1523: Declares function `run_mha_fwd` as part of this file's callable surface. / 声明函数 `run_mha_fwd`，作为本文件可调用接口的一部分。
- L1525: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1526: Documents the nearby logic: out = out.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)}); / 说明附近逻辑的作用：out = out.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)});
- L1527: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L1528: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1529: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1530: Documents the nearby logic: It's expensive to copy the KV cache here for the case where head size not divisible by 8, / 说明附近逻辑的作用：It's expensive to copy the KV cache here for the case where head size not divisible by 8,
- L1531: Documents the nearby logic: but we don't expect to get this case in practice. This is just so that the code works for that case. / 说明附近逻辑的作用：but we don't expect to get this case in practice. This is just so that the code works for that case.
- L1532: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L1533: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L1534: Documents the nearby logic: kcache.copy_(kcache_padded.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)})); / 说明附近逻辑的作用：kcache.copy_(kcache_padded.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)}));
- L1535: Documents the nearby logic: vcache.copy_(vcache_padded.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)})); / 说明附近逻辑的作用：vcache.copy_(vcache_padded.index({"...", at::indexing::Slice(at::indexing::None, head_size_og)}));
- L1536: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1537: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1539: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1540: Declares function `transpose` as part of this file's callable surface. / 声明函数 `transpose`，作为本文件可调用接口的一部分。
- L1541: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L1542: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1543: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1544: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1546: Closes namespace `pytorch_fmha` and returns to the outer scope. / 关闭命名空间 `pytorch_fmha`，返回外层作用域。
- L1548: Ends the current conditional-compilation region. / 结束当前条件编译区域。

## Key Concepts / 关键概念

- FlashAttention CUDA specialization / FlashAttention CUDA 特化实现
- CUDA transformer kernels and dispatch / CUDA Transformer 内核与分发
- Transformer attention operators and helpers / Transformer 注意力算子与辅助逻辑
- Softmax normalization and numerical stability / Softmax 归一化与数值稳定性
- Attention score computation and masking / 注意力分数计算与掩码处理
- Flash-style fused attention optimization / Flash 风格融合注意力优化
- Tensor shape, stride, and dtype flow / Tensor 形状、步长与数据类型流转
- Dispatch stubs and backend selection / 分发桩与后端选择

## Dependencies / 依赖关系

- `c10/core/ScalarType.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `c10/core/DeviceType.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `cstdint` — standard or external dependency / 标准库或外部依赖
- `tuple` — standard or external dependency / 标准库或外部依赖
- `ATen/core/Tensor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/cuda/CUDAContext.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `c10/cuda/CUDAGuard.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `ATen/cuda/CUDAGraphsUtils.cuh` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Functions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NativeFunctions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty_like.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/zeros_like.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/reshape.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/scalar_tensor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/sum.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/slice.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/narrow.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/pad.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/zeros.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `cutlass/numeric_types.h` — standard or external dependency / 标准库或外部依赖
- `flash.h` — standard or external dependency / 标准库或外部依赖
- `namespace_config.h` — standard or external dependency / 标准库或外部依赖
- `static_switch.h` — standard or external dependency / 标准库或外部依赖
- `ATen/native/transformers/cuda/flash_attn/flash_api.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `c10/util/Exception.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- Subsystem tie-in: transformer attention, masking, scaling, and backend-specific fused kernels. / 子系统关联：Transformer 注意力、掩码、缩放以及后端特化的融合内核。

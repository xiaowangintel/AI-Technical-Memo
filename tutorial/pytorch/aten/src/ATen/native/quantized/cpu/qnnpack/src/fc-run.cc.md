# fc-run.cc — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/native/quantized/cpu/qnnpack/src/fc-run.cc`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements ATen native operators, checks, helper kernels, and backend dispatch behavior. This specific file centers on `fc-run.cc`.
- **Purpose (CN)**: 实现 ATen 原生算子、运行时检查、辅助内核以及后端分发行为。 该文件具体围绕 `fc-run.cc` 展开。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行

```cpp
0001: #include <pytorch_qnnpack.h>
0002: #include <qnnpack_func.h>
0003: #include <cstring>
0004: 
0005: namespace qnnpack {
0006: struct q8gemm_context {
0007:   size_t k;
0008:   size_t k_stride;
0009:   size_t n;
0010:   size_t n_stride;
0011:   const uint8_t* a;
0012:   size_t a_stride;
0013:   const uint8_t* packed_w;
0014:   uint8_t* c;
0015:   size_t c_stride;
0016:   union pytorch_qnnp_conv_quantization_params quantization_params;
0017:   const pytorch_q8gemm_ukernel_function ukernel;
0018: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_context`。

### Lines 19-38 / 第 19-38 行

```cpp
0019: 
0020: static void compute_q8gemm(
0021:     const struct q8gemm_context context[1],
0022:     size_t group_index,
0023:     size_t pixel_index,
0024:     size_t mr_block_start,
0025:     size_t nr_block_start,
0026:     size_t group_range /* always 1 */,
0027:     size_t pixel_range,
0028:     size_t mr_block_size,
0029:     size_t nr_block_size)
0030: {
0031:   const size_t k = context->k;
0032:   const size_t k_stride = context->k_stride;
0033:   const size_t n = context->n;
0034:   const size_t n_stride = context->n_stride;
0035:   const uint8_t* a = context->a;
0036:   const size_t a_stride = context->a_stride;
0037:   const void* packed_w = context->packed_w;
0038:   uint8_t* c = context->c;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_context`, `compute_q8gemm`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_context`, `compute_q8gemm`。

### Lines 39-54 / 第 39-54 行

```cpp
0039:   const size_t c_stride = context->c_stride;
0040: 
0041:   size_t output_channel_index = nr_block_start;
0042:   context->ukernel(
0043:       mr_block_size,
0044:       nr_block_size,
0045:       k,
0046:       a + (pixel_index + mr_block_start) * a_stride + group_index * k,
0047:       a_stride,
0048:       (const void*) ((uintptr_t) packed_w + (nr_block_start + group_index * n_stride) * (k_stride * sizeof(uint8_t) + sizeof(int32_t))),
0049:       c + (pixel_index + mr_block_start) * c_stride + nr_block_start + group_index * n,
0050:       c_stride,
0051:       output_channel_index,
0052:       &context->quantization_params);
0053: }
0054: 
```

- **EN:** This block implements local helper logic for `fc-run`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `fc-run` 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 55-74 / 第 55-74 行

```cpp
0055: enum pytorch_qnnp_status qnnpackLinear(
0056:     const size_t batch_size,
0057:     const size_t input_channels,
0058:     const size_t output_channels,
0059:     const uint8_t input_zero_point,
0060:     const uint8_t* kernel_zero_points,
0061:     const float* requantization_scales,
0062:     const uint8_t output_zero_point,
0063:     const uint8_t output_min,
0064:     const uint8_t output_max,
0065:     const uint8_t* input,
0066:     const size_t input_stride,
0067:     void* packed_weights,
0068:     uint8_t* output,
0069:     const size_t output_stride,
0070:     pthreadpool_t threadpool)
0071: {
0072:   const size_t groups = 1;
0073:   const size_t group_input_channels = input_channels;
0074:   const size_t group_output_channels = output_channels;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `pytorch_qnnp_status`, `qnnpackLinear`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`pytorch_qnnp_status`, `qnnpackLinear`。

### Lines 75-86 / 第 75-86 行

```cpp
0075:   const uint32_t mr = pytorch_qnnp_params.q8conv.mr;
0076:   const uint32_t nr = pytorch_qnnp_params.q8conv.nr;
0077:   const uint32_t kr = pytorch_qnnp_params.q8conv.kr;
0078:   const size_t k_stride = (group_input_channels + (kr - 1)) & -kr;
0079:   const size_t n_stride = (group_output_channels + (nr - 1)) & -nr;
0080: 
0081:   const size_t output_size = batch_size * 1;
0082:   union pytorch_qnnp_conv_quantization_params conv_quantization_params =
0083:       pytorch_qnnp_compute_conv_quantization_params(
0084:           input_zero_point, kernel_zero_points,
0085:           requantization_scales, output_zero_point, output_min, output_max);
0086: 
```

- **EN:** This block implements local helper logic for `fc-run`. Key symbols: `pytorch_qnnp_compute_conv_quantization_params`.
- **CN:** 该代码块实现与 `fc-run` 相关的局部辅助逻辑。关键符号：`pytorch_qnnp_compute_conv_quantization_params`。

### Lines 87-99 / 第 87-99 行

```cpp
0087:   struct q8gemm_context q8gemm_context = {
0088:       .k = group_input_channels,
0089:       .k_stride = k_stride,
0090:       .n = group_output_channels,
0091:       .n_stride = n_stride,
0092:       .a = input,
0093:       .a_stride = input_stride,
0094:       .packed_w = (uint8_t*) packed_weights,
0095:       .c = output,
0096:       .c_stride = output_stride,
0097:       .quantization_params = conv_quantization_params,
0098:       .ukernel = pytorch_qnnp_params.q8conv.gemm,
0099:   };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `q8gemm_context`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`q8gemm_context`。

### Lines 100-119 / 第 100-119 行

```cpp
0100: 
0101:   if (output_size == 0) {
0102:       // pthreadpool can tolerate a range of 0, but not a tile of 0.
0103:       // We use output_size as a tile size, so bail here if it's 0.
0104:       return pytorch_qnnp_status_success;
0105:   }
0106: 
0107:   pthreadpool_compute_4d_tiled(
0108:       threadpool,
0109:       (pthreadpool_function_4d_tiled_t) compute_q8gemm,
0110:       &q8gemm_context,
0111:       groups,
0112:       1 * output_size,
0113:       output_size,
0114:       group_output_channels,
0115:       1,
0116:       output_size,
0117:       mr,
0118:       nr);
0119: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `pthreadpool_compute_4d_tiled`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`pthreadpool_compute_4d_tiled`。

### Lines 120-122 / 第 120-122 行

```cpp
0120:   return pytorch_qnnp_status_success;
0121: }
0122: } // namespace qnnpack
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: no prominent local symbols.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **ATen native operator logic** — ATen 原生算子逻辑
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Core symbols: q8gemm_context, pytorch_qnnp_status, compute_q8gemm, qnnpackLinear, pytorch_qnnp_compute_conv_quantization_params, pthreadpool_compute_4d_tiled** — 核心符号：q8gemm_context、pytorch_qnnp_status、compute_q8gemm、qnnpackLinear、pytorch_qnnp_compute_conv_quantization_params、pthreadpool_compute_4d_tiled

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: 无
- **External includes / 外部头文件**: `pytorch_qnnpack.h`, `qnnpack_func.h`, `cstring`
- **Namespaces / 命名空间**: `qnnpack`
- **Representative symbols / 代表性符号**: `q8gemm_context`, `pytorch_qnnp_status`, `compute_q8gemm`, `qnnpackLinear`, `pytorch_qnnp_compute_conv_quantization_params`, `pthreadpool_compute_4d_tiled`

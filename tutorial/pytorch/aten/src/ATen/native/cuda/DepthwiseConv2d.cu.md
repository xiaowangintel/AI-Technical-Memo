# DepthwiseConv2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/DepthwiseConv2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `dummy_packed_accessor32`, `conv_depthwise2d_grad_weight_kernel`, `conv_depthwise2d_forward_out`, `conv_depthwise2d_backward_out`.
- 用途（中文）: 实现与 `dummy_packed_accessor32`, `conv_depthwise2d_grad_weight_kernel`, `conv_depthwise2d_forward_out`, `conv_depthwise2d_backward_out` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Dispatch.h>
   4: #include <ATen/AccumulateType.h>
   5: #include <ATen/div_rtn.h>
   6: #include <ATen/cuda/CUDABlas.h>
   7: #include <ATen/cuda/detail/KernelUtils.h>
   8: #include <ATen/native/ConvUtils.h>
   9: #include <ATen/native/cuda/block_reduce.cuh>
  10: #include <ATen/native/Resize.h>
  11: #include <ATen/native/IndexingUtils.h>
  12: 
  13: #ifndef AT_PER_OPERATOR_HEADERS
  14: #include <ATen/Functions.h>
  15: #include <ATen/NativeFunctions.h>
  16: #else
  17: #include <ATen/ops/empty.h>
  18: #include <ATen/ops/_conv_depthwise2d_native.h>
  19: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/AccumulateType.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/AccumulateType.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 21-42
```cpp
  21: namespace at::native {
  22: namespace {
  23: using at::cuda::detail::CUDA_NUM_THREADS;
  24: using at::cuda::detail::GET_BLOCKS;
  25: 
  26: template <typename scalar_t, int ndim, template <typename U> class PtrTraits = DefaultPtrTraits>
  27: PackedTensorAccessor32<scalar_t, ndim, PtrTraits> dummy_packed_accessor32() {
  28:   std::array<int64_t, ndim> zeros{};
  29:   return {nullptr, zeros.data(), zeros.data()};
  30: }
  31: 
  32: template <typename scalar_t, typename index_t>
  33: __global__ void
  34: #if !defined(USE_ROCM)
  35: C10_LAUNCH_BOUNDS_1(at::cuda::detail::CUDA_NUM_THREADS)
  36: #endif
  37: conv_depthwise2d_forward_kernel_generic(
  38:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> input,
  39:     PackedTensorAccessor32<scalar_t, 4, DefaultPtrTraits> output,
  40:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> weight,
  41:     const PackedTensorAccessor32<const scalar_t, 1, DefaultPtrTraits> bias,
  42:     bool biasEnabled,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `dummy_packed_accessor32`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `dummy_packed_accessor32`，它们会直接在 CUDA 线程上执行。

### Lines 43-64
```cpp
  43:     index_t totalElements,
  44:     const int outputChannels,
  45:     const int depthwiseMultiplier,
  46:     const int inputWidth, const int inputHeight,
  47:     const int outputWidth, const int outputHeight,
  48:     const int kernelWidth, const int kernelHeight,
  49:     const int strideWidth, const int strideHeight,
  50:     const int padWidth, const int padHeight,
  51:     const int dilationWidth, const int dilationHeight) {
  52:   using acc_t = at::acc_type<scalar_t, true>;
  53: 
  54:   CUDA_KERNEL_LOOP_TYPE(linearIndex, totalElements, index_t) {
  55:     //calculate n,c,h,w indices, replacing modulos by divide and multiply add,
  56:     //result is same as would be in the code below
  57:     //const int n = linearIndex / batchStride; //batchStride = outputChannels * outputHeight * outputWidth
  58:     //const int c = (linearIndex / channelStride) % outputChannels; //channelStride = outputHeight * outputWidth
  59:     //const int h = (linearIndex / outputWidth) % outputHeight;
  60:     //const int w = linearIndex % outputWidth;
  61: 
  62:     int indtmp1 = linearIndex/outputWidth;
  63:     const int w = linearIndex - indtmp1 * outputWidth;
  64:     int indtmp2 = indtmp1/outputHeight;
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 65-69
```cpp
  65:     const int h = indtmp1 - indtmp2 * outputHeight;
  66:     indtmp1 = indtmp2;
  67:     indtmp2 = indtmp1/outputChannels;
  68:     const int c = indtmp1 - indtmp2 * outputChannels;
  69:     const int n = indtmp2;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 71-76
```cpp
  71:     int inputChannel = c;
  72:     int inputChannels = outputChannels;
  73:     if (depthwiseMultiplier !=1) {
  74:       inputChannel /= depthwiseMultiplier;
  75:       inputChannels /= depthwiseMultiplier;
  76:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 78-78
```cpp
  78:     int weightOffset = c * kernelHeight * kernelWidth;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 80-84
```cpp
  80:     // By precisely computing the filtering boundaries, we avoid repeating several
  81:     // expensive edge condition checks for every fetched item. If the input element is
  82:     // resident in L1, then the extra branches and comparisons would have been
  83:     // comparable in terms of cycles with the actual data fetch. Therefore computing
  84:     // boundaries ahead of the loop showed significant performance boost.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 86-86
```cpp
  86:     int kHmin = 0, kHmax = kernelHeight, kWmin = 0, kWmax = kernelWidth;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 88-88
```cpp
  88:     // Top
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 89-95
```cpp
  89:     int h_in_min = -padHeight + h * strideHeight;
  90:     if (h_in_min < 0) {
  91:       kHmin =  -h_in_min / dilationHeight;
  92:       if ((-h_in_min) % dilationHeight > 0) {
  93:         kHmin++;
  94:       }
  95:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 97-97
```cpp
  97:     // Bottom
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 98-104
```cpp
  98:     int h_in_max = h_in_min + (kernelHeight - 1) * dilationHeight - inputHeight + 1;
  99:     if (h_in_max >= 0) {
 100:       kHmax = kernelHeight - h_in_max / dilationHeight;
 101:       if (h_in_max % dilationHeight > 0) {
 102:         kHmax--;
 103:       }
 104:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 106-106
```cpp
 106:     // Left
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 107-113
```cpp
 107:     int w_in_min = -padWidth + w * strideWidth;
 108:     if (w_in_min < 0) {
 109:       kWmin = -w_in_min / dilationWidth;
 110:       if ((-w_in_min) % dilationWidth > 0) {
 111:         kWmin++;
 112:       }
 113:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 115-115
```cpp
 115:     // Right
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 116-122
```cpp
 116:     int w_in_max = w_in_min + (kernelWidth - 1) * dilationWidth - inputWidth + 1;
 117:     if (w_in_max >= 0) {
 118:       kWmax = kernelWidth - w_in_max / dilationWidth;
 119:       if (w_in_max % dilationWidth > 0) {
 120:         kWmax--;
 121:       }
 122:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 124-125
```cpp
 124:     acc_t value = biasEnabled ? static_cast<acc_t>(bias.data()[c]) : acc_t(0);
 125:     const index_t offset0 = (n * inputChannels + inputChannel) * inputHeight * inputWidth;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 127-138
```cpp
 127:     for (int kH = kHmin; kH < kHmax; ++kH) {
 128:       const int h_in = -padHeight + h * strideHeight + kH * dilationHeight;
 129:       for (int kW = kWmin; kW < kWmax; ++kW) {
 130:         const int w_in = -padWidth + w * strideWidth + kW * dilationWidth;
 131:         const index_t offset = offset0 + h_in * inputWidth + w_in;
 132:         value += (static_cast<acc_t>(weight.data()[weightOffset + kH * kernelWidth + kW]) *
 133:                     static_cast<acc_t>(input.data()[offset]));
 134:       }
 135:     }
 136:     output.data()[linearIndex] = static_cast<scalar_t>(value);
 137:   }
 138: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 140-153
```cpp
 140: template <int kSize, typename scalar_t, typename index_t>
 141: __global__ void
 142: #if !defined(USE_ROCM)
 143: C10_LAUNCH_BOUNDS_1(at::cuda::detail::CUDA_NUM_THREADS)
 144: #endif
 145: conv_depthwise2d_forward_kernel(
 146:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> input,
 147:     PackedTensorAccessor32<scalar_t, 4, DefaultPtrTraits> output,
 148:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> weight,
 149:     const PackedTensorAccessor32<const scalar_t, 1, DefaultPtrTraits> bias,
 150:     bool biasEnabled,
 151:     index_t totalElements,
 152:     const int outputChannels,
 153:     const int depthwiseMultiplier,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 154-175
```cpp
 154:     const int inputWidth, const int inputHeight,
 155:     const int outputWidth, const int outputHeight,
 156:     const int kernelWidth, const int kernelHeight,
 157:     const int strideWidth, const int strideHeight,
 158:     const int padWidth, const int padHeight,
 159:     const int dilationWidth, const int dilationHeight) {
 160:   using acc_t = at::acc_type<scalar_t, true>;
 161:   const int KW_LIMIT = (kSize != 0) ? kSize : kernelWidth;
 162:   const int KH_LIMIT = (kSize != 0) ? kSize : kernelHeight;
 163: 
 164:   CUDA_KERNEL_LOOP_TYPE(linearIndex, totalElements, index_t) {
 165:     //calculate n,c,h,w indices, replacing modulos by divide and multiply add,
 166:     //result is same as would be in the code below
 167:     //const int n = linearIndex / batchStride; //batchStride = outputChannels * outputHeight * outputWidth
 168:     //const int c = (linearIndex / channelStride) % outputChannels; //channelStride = outputHeight * outputWidth
 169:     //const int h = (linearIndex / outputWidth) % outputHeight;
 170:     //const int w = linearIndex % outputWidth;
 171: 
 172:     int indtmp1 = linearIndex/outputWidth;
 173:     const int w = linearIndex - indtmp1 * outputWidth;
 174:     int indtmp2 = indtmp1/outputHeight;
 175:     const int h = indtmp1 - indtmp2 * outputHeight;
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 176-179
```cpp
 176:     indtmp1 = indtmp2;
 177:     indtmp2 = indtmp1/outputChannels;
 178:     const int c = indtmp1 - indtmp2 * outputChannels;
 179:     const int n = indtmp2;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 181-186
```cpp
 181:     int inputChannel = c;
 182:     int inputChannels = outputChannels;
 183:     if (depthwiseMultiplier !=1) {
 184:       inputChannel /= depthwiseMultiplier;
 185:       inputChannels /= depthwiseMultiplier;
 186:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 188-188
```cpp
 188:     int weightOffset = c * kernelHeight * kernelWidth;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 190-210
```cpp
 190:     acc_t value = biasEnabled ? static_cast<acc_t>(bias.data()[c]) : acc_t(0);
 191:     const index_t offset0 = (n * inputChannels + inputChannel) * inputHeight * inputWidth;
 192: #if !defined(USE_ROCM)
 193: #pragma unroll
 194: #endif
 195:     for (int kH = 0; kH < KH_LIMIT; ++kH) {
 196: #if !defined(USE_ROCM)
 197: #pragma unroll
 198: #endif
 199:       for (int kW = 0; kW < KW_LIMIT; ++kW) {
 200:         const int h_in = -padHeight + h * strideHeight + kH * dilationHeight;
 201:         const int w_in = -padWidth + w * strideWidth + kW * dilationWidth;
 202: 
 203:         if ((h_in >= 0) && (h_in < inputHeight) && (w_in >= 0) && (w_in < inputWidth)) {
 204:           const index_t offset = offset0 + h_in * inputWidth + w_in;
 205:           value += (static_cast<acc_t>(weight.data()[weightOffset]) *
 206:                     static_cast<acc_t>(input.data()[offset]));
 207:         }
 208:         ++weightOffset;
 209:       }
 210:     }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 211-213
```cpp
 211:     output.data()[linearIndex] = static_cast<scalar_t>(value);
 212:   }
 213: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 215-228
```cpp
 215: template <int kSize, int stride, typename scalar_t, typename index_t>
 216: #if !defined(USE_ROCM)
 217: C10_LAUNCH_BOUNDS_1(at::cuda::detail::CUDA_NUM_THREADS)
 218: #endif
 219: __global__ void conv_depthwise2d_backward_kernel(
 220:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> grad_output,
 221:     PackedTensorAccessor32<scalar_t, 4, DefaultPtrTraits> grad_input,
 222:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> weight,
 223:     index_t totalElements,
 224:     const int inputChannels,
 225:     const int depthwiseMultiplier,
 226:     const int outputChannels,
 227:     const int inputWidth, const int inputHeight,
 228:     const int outputWidth, const int outputHeight,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines GPU kernel entry point(s) `conv_depthwise2d_backward_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `conv_depthwise2d_backward_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 229-249
```cpp
 229:     const int kernelWidth, const int kernelHeight,
 230:     const int strideWidth, const int strideHeight,
 231:     const int padWidth, const int padHeight,
 232:     const int dilationWidth, const int dilationHeight) {
 233:   using acc_t = at::acc_type<scalar_t, true>;
 234:   const int KW_LIMIT = (kSize != 0) ? kSize : kernelWidth;
 235:   const int KH_LIMIT = (kSize != 0) ? kSize : kernelHeight;
 236:   const int strideW = (stride != 0) ? stride : strideWidth;
 237:   const int strideH = (stride != 0) ? stride : strideHeight;
 238: 
 239:   CUDA_KERNEL_LOOP_TYPE(linearIndex, totalElements, index_t) {
 240:     int indtmp1 = linearIndex/inputWidth;
 241:     const int w = linearIndex - indtmp1 * inputWidth;
 242:     int indtmp2 = indtmp1/inputHeight;
 243:     const int h = indtmp1 - indtmp2 * inputHeight;
 244:     indtmp1 = indtmp2;
 245:     indtmp2 = indtmp1/inputChannels;
 246:     const int c = indtmp1 - indtmp2 * inputChannels;
 247:     const int n = indtmp2;
 248: 
 249:     acc_t value(0);
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 251-272
```cpp
 251:     for (int multiplier = 0; multiplier < depthwiseMultiplier; ++multiplier) {
 252:       int och = (c * depthwiseMultiplier) + multiplier;
 253:       int weightOffset = och * kernelHeight * kernelWidth;
 254:       for (int kh = 0; kh < KH_LIMIT; ++kh) {
 255: #if !defined(USE_ROCM)
 256: #pragma unroll
 257: #endif
 258:         for (int kw = 0; kw < KW_LIMIT; ++kw) {
 259:           int h_out = h + padHeight - kh * dilationHeight;
 260:           int w_out = w + padWidth - kw * dilationWidth;
 261:           if ((h_out % strideH == 0) && (w_out % strideW == 0)) {
 262:             h_out = h_out / strideH;
 263:             w_out = w_out / strideW;
 264: 
 265:             if ((h_out >= 0) && (h_out < outputHeight)
 266:                   && (w_out >= 0) && (w_out < outputWidth)) {
 267: 
 268:               const int offset = ((n * outputChannels + och) * outputHeight + h_out)
 269:                     * outputWidth + w_out;
 270:               value += (static_cast<acc_t>(weight.data()[weightOffset]) *
 271:                         static_cast<acc_t>(grad_output.data()[offset]));
 272:             }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 273-280
```cpp
 273:           }
 274:           ++weightOffset;
 275:         }
 276:       }
 277:     }
 278:     grad_input.data()[linearIndex] = static_cast<scalar_t>(value);
 279:   }
 280: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 282-295
```cpp
 282: template <typename scalar_t, typename index_t=unsigned>
 283: __global__ void conv_depthwise2d_grad_weight_kernel(
 284:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> grad_output,
 285:     const PackedTensorAccessor32<const scalar_t, 4, DefaultPtrTraits> input,
 286:     PackedTensorAccessor32<scalar_t, 4, DefaultPtrTraits> grad_weight,
 287:     const int batchSize,
 288:     const int inputChannels,
 289:     const int kernelChannels,
 290:     const int depthwiseMultiplier,
 291:     const int inputWidth, const int inputHeight,
 292:     const int outputWidth, const int outputHeight,
 293:     const int kernelWidth, const int kernelHeight,
 294:     const int strideWidth, const int strideHeight,
 295:     const int padWidth, const int padHeight,
```
- EN: This block defines GPU kernel entry point(s) `conv_depthwise2d_grad_weight_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `conv_depthwise2d_grad_weight_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 296-317
```cpp
 296:     const int dilationWidth, const int dilationHeight) {
 297:   using acc_t = at::acc_type<scalar_t, true>;
 298:   const int channelStride = kernelWidth * kernelHeight;
 299: 
 300:   // Each Block is responsible for accumulating over a permutation of
 301:   // (channels x kH x kW), use blockIdx to determine which one
 302:   int bidx = blockIdx.x;
 303:   int kW = bidx % kernelWidth;
 304:   int kH = (bidx / kernelWidth) % kernelHeight;
 305:   int ch = (bidx / channelStride);
 306: 
 307:   // Need to calculate which input channel is associated with this filter
 308:   // channel
 309:   int inputCh = ch / depthwiseMultiplier;
 310: 
 311:   acc_t grad(0);
 312: 
 313:   const int laneId = threadIdx.x % C10_WARP_SIZE;
 314:   const int batch = threadIdx.x / C10_WARP_SIZE;
 315:   const int nwarps = blockDim.x / C10_WARP_SIZE;
 316:   const int imageElements = outputWidth * outputHeight;
 317:   // Use warp per item.  In the original kernel, a threadblock was used to sum over NHW.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 318-324
```cpp
 318:   // Here, we use a warp to sum values over HW dimension, and if batchSize is larger than the
 319:   // number of warps, a warp would loop over remaining batch items (e.g. if there are 8 warps,
 320:   // warp 0 would go over 0-8-16 etc image, warp 1 over 1-9-17 etc). Later in blockReduce,
 321:   // all the warps will be reduced anyway, thus the full reduction will be over NHW, like it
 322:   // should be. That allows to get rid of one modulo operation inside the loop (because n/batchIdx
 323:   // now does not have to be computed through modulo, you are just looping over it), and
 324:   // bring a nice speed-up.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 325-344
```cpp
 325:   for (int batchIdx = batch; batchIdx < batchSize; batchIdx += nwarps){
 326:     // Warp-stride loop over elements in a batch item
 327:     for (index_t idx = laneId; idx < imageElements; idx += C10_WARP_SIZE) {
 328:     // Need to calculate the following: batch position, and offset into the grad_output
 329:     // in height, and width. We can intuit the corresponding position in the input from
 330:     // the other parameters we have
 331:       int go_w_offset = idx % outputWidth;
 332:       int go_h_offset = (idx / outputWidth);
 333: 
 334:       int i_w_offset = (go_w_offset * strideWidth) + (kW * dilationWidth) - padWidth;
 335:       int i_h_offset = (go_h_offset * strideHeight) + (kH * dilationHeight) - padHeight;
 336: 
 337:       if (i_w_offset >= 0 && i_h_offset >= 0 && i_w_offset < inputWidth && i_h_offset < inputHeight) {
 338:         int inputOffset = ((batchIdx * inputChannels + inputCh) * inputHeight + i_h_offset) * inputWidth + i_w_offset;
 339:         int outputOffset = ((batchIdx * kernelChannels + ch) * outputHeight ) * outputWidth + idx;
 340:         grad += (static_cast<acc_t>(input.data()[inputOffset]) *
 341:                  static_cast<acc_t>(grad_output.data()[outputOffset]));
 342:       }
 343:     }
 344:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 346-347
```cpp
 346:   // At this point each thread in the block has a local gradient, which we need to
 347:   // accumulate prior to writing the global value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 348-350
```cpp
 348:   extern __shared__ char smem[];
 349:   acc_t* buf = reinterpret_cast<acc_t*>(smem);
 350:   acc_t tval = cuda_utils::BlockReduceSum(grad, buf);
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 352-353
```cpp
 352:   // After reduction, first thread in the block has the gradient, so its responsible
 353:   // for writing it to grad_weight
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 354-358
```cpp
 354:   if (threadIdx.x == 0) {
 355:     int weightOffset = kW + (kernelWidth * kH) + (kernelWidth * kernelHeight * ch);
 356:     grad_weight.data()[weightOffset] = static_cast<scalar_t>(tval);
 357:   }
 358: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 360-381
```cpp
 360: void conv_depthwise2d_forward_out(
 361:                   const Tensor &input,
 362:                   const Tensor &output,
 363:                   const Tensor &weight,
 364:                   const Tensor &bias,
 365:                   const int kW, const int kH,
 366:                   const int dW, const int dH,
 367:                   const int padW, const int padH,
 368:                   const int dilationW, const int dilationH) {
 369:   // Only handle 4D Input Tensors for now
 370:   TORCH_CHECK(input.numel() > 0 && input.dim() == 4);
 371:   TORCH_CHECK(weight.numel() > 0 && weight.dim() == 4);
 372:   TORCH_CHECK(output.is_contiguous());
 373: 
 374:   auto in_sizes = input.sizes();
 375:   auto w_sizes = weight.sizes();
 376: 
 377:   // We assume that the input and weight Tensors are shaped properly by
 378:   // the caller, so we verify that here to some extent
 379: 
 380:   // Weight Tensor is shape (output_channels, 1, kH, kW)
 381:   TORCH_CHECK(w_sizes[1] == 1);
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_forward_out`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_forward_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 383-384
```cpp
 383:   // Input Tensor is shape (N, input_channels, H, W)
 384:   // We verify that the # of output_channels is a multiple of input_channels
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 385-385
```cpp
 385:   TORCH_CHECK(w_sizes[0] % in_sizes[1] == 0);
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 387-387
```cpp
 387:   // Bias has same # of channels as output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 388-389
```cpp
 388:   const bool has_bias = bias.defined();
 389:   TORCH_CHECK(!has_bias || (bias.dim() <= 1 && bias.numel() == w_sizes[0]));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 391-392
```cpp
 391:   // Following the behavior of other THCUNN functions, we shape the output
 392:   // Tensor ourselves
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 393-399
```cpp
 393:   int64_t height = in_sizes[2];
 394:   int64_t width = in_sizes[3];
 395:   int64_t outputChannels = w_sizes[0];
 396:   auto out_sizes = conv_output_size(in_sizes, weight.sizes(), {padH, padW}, {dH, dW},
 397:                                     {dilationH, dilationW});
 398:   const auto outputWidth = out_sizes[3];
 399:   const auto outputHeight = out_sizes[2];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 401-401
```cpp
 401:   resize_output(output, out_sizes);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 403-404
```cpp
 403:   int64_t inputChannels = in_sizes[1];
 404:   int64_t depthwiseMultiplier = outputChannels / inputChannels;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 406-406
```cpp
 406:   // One thread per output value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 407-412
```cpp
 407:   TORCH_CHECK(canUse32BitIndexMath(input) && canUse32BitIndexMath(output));
 408:   int32_t n = output.numel();
 409:   int blocks = GET_BLOCKS(n);
 410:   dim3 grid(blocks);
 411:   dim3 block(CUDA_NUM_THREADS);
 412:   const auto stream = c10::cuda::getCurrentCUDAStream();
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 414-435
```cpp
 414:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 415:                                   "conv_depthwise2d_forward_cuda", [&] {
 416:     // Create PackedTensorAccessor
 417:     // Kernel currently relies upon all the Tensors to be contiguous, but we made
 418:     // them contiguous above
 419:     const auto input_a = input.packed_accessor32<const scalar_t, 4>();
 420:     const auto weight_a = weight.packed_accessor32<const scalar_t, 4>();
 421:     const auto output_a = output.packed_accessor32<scalar_t, 4>();
 422:     const auto bias_a = has_bias ?
 423:       bias.packed_accessor32<const scalar_t, 1>() :
 424:       dummy_packed_accessor32<const scalar_t, 1>();
 425:     if (kW == 5 && kH == 5) {
 426:       conv_depthwise2d_forward_kernel<5> <<<grid, block, 0, stream>>>(
 427:         input_a, output_a, weight_a, bias_a, has_bias, n, outputChannels, depthwiseMultiplier,
 428:         width, height, outputWidth, outputHeight,
 429:         kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 430:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 431:     } else if (kW == 3 && kH == 3) {
 432:       conv_depthwise2d_forward_kernel<3> <<<grid, block, 0, stream>>>(
 433:         input_a, output_a, weight_a, bias_a, has_bias, n, outputChannels, depthwiseMultiplier,
 434:         width, height, outputWidth, outputHeight,
 435:         kW, kH, dW, dH, padW, padH, dilationW, dilationH);
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 436-449
```cpp
 436:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 437:     } else if (kW == 1 && kH == 1) {
 438:       conv_depthwise2d_forward_kernel<1> <<<grid, block, 0, stream>>>(
 439:         input_a, output_a, weight_a, bias_a, has_bias, n, outputChannels, depthwiseMultiplier,
 440:         width, height, outputWidth, outputHeight,
 441:         kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 442:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 443:     } else {
 444:       conv_depthwise2d_forward_kernel_generic<<<grid, block, 0, stream>>>(
 445:         input_a, output_a, weight_a, bias_a, has_bias, n, outputChannels, depthwiseMultiplier,
 446:         width, height, outputWidth, outputHeight,
 447:         kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 448:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 449:     }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 450-451
```cpp
 450:   });
 451: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 453-474
```cpp
 453: void conv_depthwise2d_backward_out(
 454:                   const Tensor &input,
 455:                   const Tensor &grad_output,
 456:                   const Tensor &grad_input,
 457:                   const Tensor &weight,
 458:                   const int kW, const int kH,
 459:                   const int dW, const int dH,
 460:                   const int padW, const int padH,
 461:                   const int dilationW, const int dilationH) {
 462:   // Only handle 4D Input Tensors for now
 463:   TORCH_CHECK(input.numel() > 0 && input.dim() == 4);
 464:   TORCH_CHECK(weight.numel() > 0 && weight.dim() == 4);
 465:   TORCH_CHECK(grad_output.numel() > 0 && grad_output.dim() == 4);
 466: 
 467:   // Minimal shape checking, as above
 468:   // Same # of elements in batch
 469:   TORCH_CHECK(input.sizes()[0] == grad_output.sizes()[0]);
 470:   // Same # of filters as outputChannels
 471:   TORCH_CHECK(weight.sizes()[0] == grad_output.sizes()[1]);
 472: 
 473:   // Resize Grainput_a
 474:   auto in_sizes = input.sizes();
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_backward_out`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_backward_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 475-475
```cpp
 475:   resize_output(grad_input, in_sizes);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 477-479
```cpp
 477:   int inputChannels = in_sizes[1];
 478:   int height = in_sizes[2];
 479:   int width = in_sizes[3];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 481-484
```cpp
 481:   auto gO_sizes = grad_output.sizes();
 482:   int outputChannels = gO_sizes[1];
 483:   int outputHeight = gO_sizes[2];
 484:   int outputWidth = gO_sizes[3];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 486-486
```cpp
 486:   int depthwiseMultiplier = outputChannels / inputChannels;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 488-488
```cpp
 488:   // Kernel currently relies upon all the Tensors to be contiguous
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 489-491
```cpp
 489:   TORCH_CHECK(grad_output.is_contiguous());
 490:   TORCH_CHECK(weight.is_contiguous());
 491:   TORCH_CHECK(grad_input.is_contiguous());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 493-493
```cpp
 493:   // One thread per grainput_a value
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 494-515
```cpp
 494:   TORCH_CHECK(canUse32BitIndexMath(grad_input) &&
 495:               canUse32BitIndexMath(grad_output));
 496:   int32_t n = grad_input.numel();
 497:   int blocks = GET_BLOCKS(n);
 498:   dim3 grid(blocks);
 499:   dim3 block(CUDA_NUM_THREADS);
 500:   const auto stream = c10::cuda::getCurrentCUDAStream();
 501:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, grad_output.scalar_type(),
 502:                                   "conv_depthwise2d_backward_cuda", [&] {
 503:     auto grad_output_a = grad_output.packed_accessor32<const scalar_t, 4>();
 504:     auto grad_input_a = grad_input.packed_accessor32<scalar_t, 4>();
 505:     auto weight_a = weight.packed_accessor32<const scalar_t, 4>();
 506: 
 507:     if (kW == 5 && kH == 5) {
 508:       if (dW == 1 && dH == 1){
 509:         conv_depthwise2d_backward_kernel<5, 1><<<grid, block, 0, stream>>>(
 510:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 511:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 512:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 513:       } else if (dW == 2 && dH == 2) {
 514:         conv_depthwise2d_backward_kernel<5, 2><<<grid, block, 0, stream>>>(
 515:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 516-529
```cpp
 516:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 517:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 518:       } else {
 519:         conv_depthwise2d_backward_kernel<5, 0><<<grid, block, 0, stream>>>(
 520:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 521:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 522:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 523:       }
 524:     } else if (kW == 3 && kH == 3) {
 525:       if (dW == 1 && dH == 1){
 526:         conv_depthwise2d_backward_kernel<3, 1><<<grid, block, 0, stream>>>(
 527:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 528:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 529:         C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 530-543
```cpp
 530:       } else if (dW == 2 && dH == 2) {
 531:         conv_depthwise2d_backward_kernel<3, 2><<<grid, block, 0, stream>>>(
 532:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 533:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 534:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 535:       } else {
 536:         conv_depthwise2d_backward_kernel<3, 0><<<grid, block, 0, stream>>>(
 537:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 538:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 539:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 540:       }
 541:     } else if (kW == 1 && kH == 1) {
 542:       if (dW == 1 && dH == 1){
 543:         conv_depthwise2d_backward_kernel<1, 1><<<grid, block, 0, stream>>>(
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 544-557
```cpp
 544:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 545:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 546:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 547:       } else if (dW == 2 && dH == 2) {
 548:         conv_depthwise2d_backward_kernel<1, 2><<<grid, block, 0, stream>>>(
 549:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 550:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 551:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 552:       } else {
 553:         conv_depthwise2d_backward_kernel<1, 0><<<grid, block, 0, stream>>>(
 554:             grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 555:             height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 556:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 557:       }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 558-571
```cpp
 558:     } else if (dW == 1 && dH == 1) {
 559:       conv_depthwise2d_backward_kernel<0, 1><<<grid, block, 0, stream>>>(
 560:           grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 561:           height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 562:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 563:     } else if (dW == 2 && dH == 2) {
 564:       conv_depthwise2d_backward_kernel<0, 2><<<grid, block, 0, stream>>>(
 565:           grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 566:           height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 567:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 568:     } else {
 569:       conv_depthwise2d_backward_kernel<0, 0><<<grid, block, 0, stream>>>(
 570:           grad_output_a, grad_input_a, weight_a, n, inputChannels, depthwiseMultiplier, outputChannels, width,
 571:           height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 572-575
```cpp
 572:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 573:     }
 574:   });
 575: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 577-578
```cpp
 577: // Crude benchmarks suggest 256 is better than 512 and 1024
 578: // TODO: Autotune/use better heuristics, improve speed more.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 579-583
```cpp
 579: int getGradParamsNumThreads(int batchSize) {
 580:   //warp per item in a batch, up to a maximum
 581:   constexpr int MAX_BLOCK_SIZE = 256;
 582:   return std::min(batchSize * at::cuda::warp_size(), MAX_BLOCK_SIZE);
 583: }
```
- EN: This block defines or continues the implementation of `getGradParamsNumThreads`.
- CN: 该代码块定义或继续实现 `getGradParamsNumThreads`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 585-605
```cpp
 585: void conv_depthwise2d_grad_weight_out(
 586:                   const Tensor &input,
 587:                   const Tensor &grad_output,
 588:                   const Tensor &grad_weight,
 589:                   const int kW, const int kH,
 590:                   const int dW, const int dH,
 591:                   const int padW, const int padH,
 592:                   const int dilationW, const int dilationH) {
 593:   // Only handle 4D Input Tensors for now
 594:   TORCH_CHECK(input.numel() > 0 && input.dim() == 4);
 595:   TORCH_CHECK(grad_output.numel() > 0 && grad_output.dim() == 4);
 596: 
 597:   // Minimal shape checking as above
 598:   // Same # of elements in batch
 599:   TORCH_CHECK(input.sizes()[0] == grad_output.sizes()[0]);
 600: 
 601:   auto in_sizes = input.sizes();
 602:   int batchSize = in_sizes[0];
 603:   int inputChannels = in_sizes[1];
 604:   int height = in_sizes[2];
 605:   int width = in_sizes[3];
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_grad_weight_out`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_grad_weight_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 607-610
```cpp
 607:   auto gO_sizes = grad_output.sizes();
 608:   int outputChannels = gO_sizes[1];
 609:   int outputHeight = gO_sizes[2];
 610:   int outputWidth = gO_sizes[3];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 612-612
```cpp
 612:   int depthwiseMultiplier = outputChannels / inputChannels;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 614-614
```cpp
 614:   resize_output(grad_weight, {outputChannels, 1, kH, kW});
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 616-616
```cpp
 616:   // Kernel currently relies upon all the Tensors to be contiguous
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 617-619
```cpp
 617:   TORCH_CHECK(grad_output.is_contiguous());
 618:   TORCH_CHECK(input.is_contiguous());
 619:   TORCH_CHECK(grad_weight.is_contiguous());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 621-621
```cpp
 621:   // We parallelize so that each block computes a single value in grad_weight
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 622-624
```cpp
 622:   TORCH_CHECK(canUse32BitIndexMath(input) &&
 623:               canUse32BitIndexMath(grad_output));
 624:   int blocks = outputChannels * kH * kW;
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 626-627
```cpp
 626:   // Make sure we have enough threads to perform the reduction, and use this number
 627:   // to create the shared memory size for the reduction
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 628-630
```cpp
 628:   dim3 grid(blocks);
 629:   dim3 block(getGradParamsNumThreads(batchSize));
 630:   const auto stream = c10::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 632-645
```cpp
 632:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, grad_output.scalar_type(),
 633:                                   "conv_depthwise2d_grad_weight_cuda", [&] {
 634:     const auto grad_output_a = grad_output.packed_accessor32<const scalar_t, 4>();
 635:     const auto input_a = input.packed_accessor32<const scalar_t, 4>();
 636:     const auto grad_weight_a = grad_weight.packed_accessor32<scalar_t, 4>();
 637:     using acc_t = at::acc_type<scalar_t, true>;
 638:     int warp_size = at::cuda::warp_size();
 639:     TORCH_INTERNAL_ASSERT(block.x % warp_size == 0);
 640:     int smem = (block.x  / warp_size) * sizeof(acc_t);
 641:     conv_depthwise2d_grad_weight_kernel<<<grid, block, smem, stream>>>(
 642:         grad_output_a, input_a, grad_weight_a, batchSize, inputChannels, outputChannels, depthwiseMultiplier,
 643:         width, height, outputWidth, outputHeight, kW, kH, dW, dH, padW, padH, dilationW, dilationH);
 644:     C10_CUDA_KERNEL_LAUNCH_CHECK();
 645:   });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 646-646
```cpp
 646: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 648-648
```cpp
 648: }  // namespace (anonymous)
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 650-671
```cpp
 650: Tensor& conv_depthwise2d_cuda_out(
 651:     const Tensor &input_,
 652:     const Tensor &weight_,
 653:     IntArrayRef kernel_size,
 654:     const std::optional<Tensor> &bias_opt,
 655:     IntArrayRef stride,
 656:     IntArrayRef padding,
 657:     IntArrayRef dilation,
 658:     Tensor &out) {
 659:   TORCH_CHECK(kernel_size.size() == 2);
 660:   TORCH_CHECK(stride.size() == 2);
 661:   TORCH_CHECK(padding.size() == 2);
 662:   TORCH_CHECK(dilation.size() == 2);
 663: 
 664:   auto input = input_.expect_contiguous();
 665:   auto weight = weight_.expect_contiguous();
 666:   auto bias = [&] {
 667:     if (bias_opt.has_value() && bias_opt->defined()) {
 668:       return bias_opt->expect_contiguous();
 669:     }
 670:     return c10::MaybeOwned<Tensor>::owned(std::in_place);
 671:   }();
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_cuda_out`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_cuda_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 673-683
```cpp
 673:   conv_depthwise2d_forward_out(
 674:       *input,
 675:       out,
 676:       *weight,
 677:       *bias,
 678:       kernel_size[1], kernel_size[0],
 679:       stride[1], stride[0],
 680:       padding[1], padding[0],
 681:       dilation[1], dilation[0]);
 682:   return out;
 683: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 685-696
```cpp
 685: Tensor conv_depthwise2d_cuda(
 686:     const Tensor &input,
 687:     const Tensor &weight,
 688:     IntArrayRef kernel_size,
 689:     const std::optional<Tensor> &bias,
 690:     IntArrayRef stride,
 691:     IntArrayRef padding,
 692:     IntArrayRef dilation) {
 693:   auto out = at::empty({0}, input.options());
 694:   return conv_depthwise2d_cuda_out(input, weight, kernel_size, bias,
 695:                                    stride, padding, dilation, out);
 696: }
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_cuda`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 698-718
```cpp
 698: std::tuple<Tensor&, Tensor&> conv_depthwise2d_backward_cuda_out(
 699:     const Tensor & grad_output_,
 700:     const Tensor & self_,
 701:     const Tensor & weight_,
 702:     IntArrayRef kernel_size,
 703:     IntArrayRef stride,
 704:     IntArrayRef padding,
 705:     IntArrayRef dilation,
 706:     Tensor & grad_input,
 707:     Tensor & grad_weight) {
 708:   auto grad_output = grad_output_.expect_contiguous();
 709: 
 710:   if (grad_weight.defined()) {
 711:     auto self = self_.expect_contiguous();
 712:     conv_depthwise2d_grad_weight_out(
 713:         *self, *grad_output, grad_weight,
 714:         kernel_size[1], kernel_size[0],
 715:         stride[1], stride[0],
 716:         padding[1], padding[0],
 717:         dilation[1], dilation[0]);
 718:   }
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_backward_cuda_out`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_backward_cuda_out`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 720-730
```cpp
 720:   if (grad_input.defined()) {
 721:     auto weight = weight_.expect_contiguous();
 722:     conv_depthwise2d_backward_out(
 723:         self_, *grad_output, grad_input, *weight,
 724:         kernel_size[1], kernel_size[0],
 725:         stride[1], stride[0],
 726:         padding[1], padding[0],
 727:         dilation[1], dilation[0]);
 728:   }
 729:   return std::forward_as_tuple(grad_input, grad_weight);
 730: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 732-753
```cpp
 732: std::tuple<Tensor, Tensor> conv_depthwise2d_backward_cuda(
 733:     const Tensor& grad_output,
 734:     const Tensor& self,
 735:     const Tensor& weight,
 736:     IntArrayRef kernel_size,
 737:     IntArrayRef stride,
 738:     IntArrayRef padding,
 739:     IntArrayRef dilation,
 740:     std::array<bool, 2> output_mask) {
 741:   Tensor grad_input;
 742:   Tensor grad_weight;
 743: 
 744:   if (output_mask[0]) {
 745:     grad_input = at::empty({0}, grad_output.options());
 746:   }
 747: 
 748:   if (output_mask[1]) {
 749:     grad_weight = at::empty({0}, grad_output.options());
 750:   }
 751:   return conv_depthwise2d_backward_cuda_out(
 752:       grad_output,
 753:       self,
```
- EN: This block defines or continues the implementation of `conv_depthwise2d_backward_cuda`.
- CN: 该代码块定义或继续实现 `conv_depthwise2d_backward_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 754-761
```cpp
 754:       weight,
 755:       kernel_size,
 756:       stride,
 757:       padding,
 758:       dilation,
 759:       grad_input,
 760:       grad_weight);
 761: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 763-763
```cpp
 763: REGISTER_CUDA_DISPATCH(conv_depthwise2d_backward_stub, &conv_depthwise2d_backward_cuda)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 765-765
```cpp
 765: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- CUDA dispatch registration binds a stub to this file's implementation. / CUDA 分发注册会把 stub 绑定到本文件实现。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/div_rtn.h>`
  - `<ATen/cuda/CUDABlas.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/native/ConvUtils.h>`
  - `<ATen/native/cuda/block_reduce.cuh>`
  - `<ATen/native/Resize.h>`
  - `<ATen/native/IndexingUtils.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
- Runtime symbols / 运行时符号:
  - `conv_depthwise2d_backward_stub`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `REGISTER_CUDA_DISPATCH`
  - `at::cuda::detail::CUDA_NUM_THREADS`
  - `at::cuda::detail::GET_BLOCKS`
  - `at::cuda::warp_size`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

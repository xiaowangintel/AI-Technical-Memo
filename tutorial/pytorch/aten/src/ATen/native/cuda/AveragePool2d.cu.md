# AveragePool2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/AveragePool2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `avg_pool2d_out_cuda`, `avg_pool2d_backward_out_cuda`, `min`, `max`.
- 用途（中文）: 实现与 `avg_pool2d_out_cuda`, `avg_pool2d_backward_out_cuda`, `min`, `max` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/native/Pool.h>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <ATen/cuda/detail/TensorInfo.cuh>
   9: #include <ATen/cuda/detail/IndexUtils.cuh>
  10: #include <ATen/cuda/detail/KernelUtils.h>
  11: #include <c10/macros/Macros.h>
  12: 
  13: #ifndef AT_PER_OPERATOR_HEADERS
  14: #include <ATen/Functions.h>
  15: #include <ATen/NativeFunctions.h>
  16: #else
  17: #include <ATen/ops/avg_pool2d_native.h>
  18: #include <ATen/ops/avg_pool2d_backward_native.h>
  19: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 21-42
```cpp
  21: namespace at::native {
  22: namespace {
  23: 
  24: __device__ inline int min(int a, int b) {
  25:   return a <= b ? a : b;
  26: }
  27: 
  28: __device__ inline int max(int a, int b) {
  29:   return a >= b ? a : b;
  30: }
  31: 
  32: template <typename scalar_t, typename accscalar_t>
  33: __global__ void avg_pool2d_out_cuda_frame(const int nthreads,
  34:     const scalar_t* const bottom_data, const int64_t channels,
  35:     const int64_t height, const int64_t width, const int64_t pooled_height,
  36:     const int pooled_width, const int kernel_h, const int kernel_w,
  37:     const int stride_h, const int stride_w, const int pad_h, const int pad_w,
  38:     scalar_t* const top_data, const int divisor_override,
  39:     const bool count_include_pad, const bool use_divisor) {
  40:   CUDA_KERNEL_LOOP(index, nthreads) {
  41:     const int pw = index % pooled_width;
  42:     const int ph = (index / pooled_width) % pooled_height;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `avg_pool2d_out_cuda_frame`, `min`, `max`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `avg_pool2d_out_cuda_frame`, `min`, `max`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 43-53
```cpp
  43:     const int c = (index / pooled_width / pooled_height) % channels;
  44:     const int n = index / pooled_width / pooled_height / channels;
  45:     int hstart = ph * stride_h - pad_h;
  46:     int wstart = pw * stride_w - pad_w;
  47:     int hend = min(hstart + kernel_h, height + pad_h);
  48:     int wend = min(wstart + kernel_w, width + pad_w);
  49:     const int pool_size = (hend - hstart) * (wend - wstart);
  50:     hstart = max(hstart, 0);
  51:     wstart = max(wstart, 0);
  52:     hend = min(hend, height);
  53:     wend = min(wend, width);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 55-58
```cpp
  55:     if (hstart >= hend || wstart >= wend) {
  56:       top_data[index] = scalar_t(0);
  57:       continue;
  58:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 60-76
```cpp
  60:     accscalar_t aveval = accscalar_t(0);
  61:     const scalar_t* const bottom_slice = bottom_data + (n * channels + c) * height * width;
  62:     for (int h = hstart; h < hend; ++h) {
  63:       for (int w = wstart; w < wend; ++w) {
  64:         aveval += bottom_slice[h * width + w];
  65:       }
  66:     }
  67:     int divide_factor;
  68:     if (use_divisor) {
  69:       divide_factor = divisor_override;
  70:     } else {
  71:       if(count_include_pad) {
  72:         divide_factor = pool_size;
  73:       } else {
  74:         divide_factor = (hend - hstart) * (wend - wstart);
  75:       }
  76:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 77-79
```cpp
  77:     top_data[index] = static_cast<scalar_t>(aveval / divide_factor);
  78:   }
  79: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 81-102
```cpp
  81: template <typename scalar_t, typename accscalar_t>
  82: __global__ void avg_pool2d_out_cuda_frame_nhwc(const int nthreads,
  83:     const scalar_t* const bottom_data, const int64_t channels,
  84:     const int64_t height, const int64_t width, const int pooled_height,
  85:     const int pooled_width, const int kernel_h, const int kernel_w,
  86:     const int stride_h, const int stride_w, const int pad_h, const int pad_w,
  87:     scalar_t* const top_data, const int divisor_override,
  88:     const bool count_include_pad, const bool use_divisor) {
  89:   CUDA_KERNEL_LOOP(index, nthreads) {
  90:     const int c = index % channels;
  91:     const int pw = (index / channels) % pooled_width;
  92:     const int ph = (index / channels / pooled_width) % pooled_height;
  93:     const int n = index / channels / pooled_width / pooled_height;
  94:     int hstart = ph * stride_h - pad_h;
  95:     int wstart = pw * stride_w - pad_w;
  96:     int hend = min(hstart + kernel_h, height + pad_h);
  97:     int wend = min(wstart + kernel_w, width + pad_w);
  98:     const int pool_size = (hend - hstart) * (wend - wstart);
  99:     hstart = max(hstart, 0);
 100:     wstart = max(wstart, 0);
 101:     hend = min(hend, height);
 102:     wend = min(wend, width);
```
- EN: This block defines GPU kernel entry point(s) `avg_pool2d_out_cuda_frame_nhwc`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `avg_pool2d_out_cuda_frame_nhwc`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 104-107
```cpp
 104:     if (hstart >= hend || wstart >= wend) {
 105:       top_data[index] = scalar_t(0);
 106:       continue;
 107:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 109-125
```cpp
 109:     accscalar_t aveval = accscalar_t(0);
 110:     const scalar_t* const bottom_slice = bottom_data + n * channels * height * width + c;
 111:     for (int h = hstart; h < hend; ++h) {
 112:       for (int w = wstart; w < wend; ++w) {
 113:         aveval += bottom_slice[(h * width + w) * channels];
 114:       }
 115:     }
 116:     int divide_factor;
 117:     if (use_divisor) {
 118:       divide_factor = divisor_override;
 119:     } else {
 120:       if(count_include_pad) {
 121:         divide_factor = pool_size;
 122:       } else {
 123:         divide_factor = (hend - hstart) * (wend - wstart);
 124:       }
 125:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 126-128
```cpp
 126:     top_data[index] = static_cast<scalar_t>(aveval / divide_factor);
 127:   }
 128: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 130-151
```cpp
 130: template <typename scalar_t, typename accscalar_t, typename index_t>
 131: __global__ void avg_pool2d_backward_out_cuda_frame(const index_t nthreads, const scalar_t* const top_diff,
 132:     const int64_t channels, const int64_t height,
 133:     const int64_t width, const int64_t pooled_height, const int64_t pooled_width,
 134:     const int kernel_h, const int kernel_w, const int stride_h,
 135:     const int stride_w, const int pad_h, const int pad_w,
 136:     scalar_t* const bottom_diff, const int divisor_override,
 137:     bool count_include_pad, bool use_divisor) {
 138:   CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
 139:     // find out the local index
 140:     // find out the local offset
 141:     const int w = index % width + pad_w;
 142:     const int h = (index / width) % height + pad_h;
 143:     const int c = (index / width / height) % channels;
 144:     const int n = index / width / height / channels;
 145:     const int phstart = (h < kernel_h) ? 0 : (h - kernel_h) / stride_h + 1;
 146:     const int phend = min(h / stride_h + 1, pooled_height);
 147:     const int pwstart = (w < kernel_w) ? 0 : (w - kernel_w) / stride_w + 1;
 148:     const int pwend = min(w / stride_w + 1, pooled_width);
 149:     accscalar_t gradient = accscalar_t(0);
 150:     const scalar_t* const top_diff_slice =
 151:         top_diff + (n * channels + c) * pooled_height * pooled_width;
```
- EN: This block defines GPU kernel entry point(s) `avg_pool2d_backward_out_cuda_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `avg_pool2d_backward_out_cuda_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 152-173
```cpp
 152:     for (int ph = phstart; ph < phend; ++ph) {
 153:       for (int pw = pwstart; pw < pwend; ++pw) {
 154:         // figure out the pooling size
 155:         int hstart = ph * stride_h - pad_h;
 156:         int wstart = pw * stride_w - pad_w;
 157:         int hend = min(hstart + kernel_h, height + pad_h);
 158:         int wend = min(wstart + kernel_w, width + pad_w);
 159:         int pool_size = (hend - hstart) * (wend - wstart);
 160:         hstart = max(hstart, 0);
 161:         wstart = max(wstart, 0);
 162:         hend = min(hend, height);
 163:         wend = min(wend, width);
 164: 
 165:         if (hstart >= hend || wstart >= wend) {
 166:           continue;
 167:         }
 168: 
 169:         int divide_factor;
 170:         if (use_divisor) {
 171:           divide_factor = divisor_override;
 172:         } else {
 173:           if(count_include_pad) {
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 174-184
```cpp
 174:             divide_factor = pool_size;
 175:           } else {
 176:             divide_factor = (hend - hstart) * (wend - wstart);
 177:           }
 178:         }
 179:         gradient += top_diff_slice[ph * pooled_width + pw] / divide_factor;
 180:       }
 181:     }
 182:     bottom_diff[index] = static_cast<scalar_t>(gradient);
 183:   }
 184: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 186-207
```cpp
 186: template <typename scalar_t, typename accscalar_t, typename index_t>
 187: __global__ void avg_pool2d_backward_out_cuda_frame_nhwc(const index_t nthreads,
 188:     const scalar_t* const top_diff,
 189:     const int64_t channels, const int64_t height,
 190:     const int64_t width, const int pooled_height, const int pooled_width,
 191:     const int kernel_h, const int kernel_w, const int stride_h,
 192:     const int stride_w, const int pad_h, const int pad_w,
 193:     scalar_t* const bottom_diff, const int divisor_override,
 194:     bool count_include_pad, bool use_divisor) {
 195:   CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
 196:     const int c = index % channels;
 197:     const int w = (index / channels) % width;
 198:     const int h = (index / channels / width) % height;
 199:     const int n = index / channels / width / height;
 200: 
 201:     const int phstart = (h < kernel_h) ? 0 : (h - kernel_h) / stride_h + 1;
 202:     const int phend = min(h / stride_h + 1, pooled_height);
 203:     const int pwstart = (w < kernel_w) ? 0 : (w - kernel_w) / stride_w + 1;
 204:     const int pwend = min(w / stride_w + 1, pooled_width);
 205:     accscalar_t gradient = accscalar_t(0);
 206:     const scalar_t* const top_diff_slice = top_diff + n * channels * pooled_height * pooled_width + c;
 207:     for (int ph = phstart; ph < phend; ++ph) {
```
- EN: This block defines GPU kernel entry point(s) `avg_pool2d_backward_out_cuda_frame_nhwc`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `avg_pool2d_backward_out_cuda_frame_nhwc`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 208-229
```cpp
 208:       for (int pw = pwstart; pw < pwend; ++pw) {
 209:         // figure out the pooling size
 210:         int hstart = ph * stride_h - pad_h;
 211:         int wstart = pw * stride_w - pad_w;
 212:         int hend = min(hstart + kernel_h, height + pad_h);
 213:         int wend = min(wstart + kernel_w, width + pad_w);
 214:         int pool_size = (hend - hstart) * (wend - wstart);
 215:         hstart = max(hstart, 0);
 216:         wstart = max(wstart, 0);
 217:         hend = min(hend, height);
 218:         wend = min(wend, width);
 219: 
 220:         if (hstart >= hend || wstart >= wend) {
 221:           continue;
 222:         }
 223: 
 224:         int divide_factor;
 225:         if (use_divisor) {
 226:           divide_factor = divisor_override;
 227:         } else {
 228:           if(count_include_pad) {
 229:             divide_factor = pool_size;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 230-239
```cpp
 230:           } else {
 231:             divide_factor = (hend - hstart) * (wend - wstart);
 232:           }
 233:         }
 234:         gradient += top_diff_slice[(ph * pooled_width + pw) * channels] / divide_factor;
 235:       }
 236:     }
 237:     bottom_diff[index] = static_cast<scalar_t>(gradient);
 238:   }
 239: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-241
```cpp
 241: } // anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 243-264
```cpp
 243: TORCH_IMPL_FUNC(avg_pool2d_out_cuda)
 244: (const Tensor& input_,
 245:  int64_t kH_,
 246:  int64_t kW_,
 247:  int64_t dH_,
 248:  int64_t dW_,
 249:  int64_t padH_,
 250:  int64_t padW_,
 251:  bool ceil_mode,
 252:  bool count_include_pad,
 253:  std::optional<int64_t> divisor_override,
 254:  const Tensor& output) {
 255:   TensorArg output_arg{ output, "output", 1 };
 256:   TensorArg input_arg{ input_, "input_", 2 };
 257: 
 258:   checkAllSameGPU("avg_pool2d_out_cuda", {output_arg, input_arg});
 259: 
 260:   const int kH = safe_downcast<int, int64_t>(kH_);
 261:   const int kW = safe_downcast<int, int64_t>(kW_);
 262: 
 263:   const int dH = safe_downcast<int, int64_t>(dH_);
 264:   const int dW = safe_downcast<int, int64_t>(dW_);
```
- EN: This block defines or continues the implementation of `avg_pool2d_out_cuda`.
- CN: 该代码块定义或继续实现 `avg_pool2d_out_cuda`。

### Lines 266-267
```cpp
 266:   const int padH = safe_downcast<int, int64_t>(padH_);
 267:   const int padW = safe_downcast<int, int64_t>(padW_);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 269-269
```cpp
 269:   /* sizes */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 270-272
```cpp
 270:   const int64_t nInputPlane = input_.size(-3);
 271:   const int64_t inputHeight = input_.size(-2);
 272:   const int64_t inputWidth = input_.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 274-276
```cpp
 274:   int64_t outputWidth = pooling_output_shape<int64_t>(inputWidth, kW, padW, dW, 1, ceil_mode);
 275:   int64_t outputHeight = pooling_output_shape<int64_t>(inputHeight, kH, padH, dH, 1, ceil_mode);
 276:   const auto memory_format = input_.suggest_memory_format();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 278-278
```cpp
 278:   Tensor input = input_.contiguous(memory_format);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 280-282
```cpp
 280:   const auto count = safe_downcast<int32_t, int64_t>(output.numel());
 281:   const uint32_t num_threads = std::min(at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, 1024);
 282:   const uint32_t num_blocks = ceil_div<uint32_t>(count, num_threads);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 284-285
```cpp
 284:   bool use_divisor = divisor_override.has_value();
 285:   const auto divisor_override_value = use_divisor ? divisor_override.value() : 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 287-308
```cpp
 287:   if (count != 0) {
 288:     AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 289:       "avg_pool2d_out_cuda_frame",
 290:       [&] {
 291:         using accscalar_t = acc_type<scalar_t, true>;
 292: 
 293:         scalar_t *output_data = output.mutable_data_ptr<scalar_t>();
 294:         const scalar_t *input_data = input.const_data_ptr<scalar_t>();
 295: 
 296:         switch (memory_format){
 297:           case MemoryFormat::ChannelsLast: {
 298:             output.unsafeGetTensorImpl()->empty_tensor_restride(MemoryFormat::ChannelsLast);
 299:             avg_pool2d_out_cuda_frame_nhwc<scalar_t, accscalar_t>
 300:                 <<<num_blocks,
 301:                    num_threads,
 302:                    0,
 303:                    at::cuda::getCurrentCUDAStream()>>>(
 304:                     count,
 305:                     input_data,
 306:                     nInputPlane,
 307:                     inputHeight,
 308:                     inputWidth,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 309-322
```cpp
 309:                     outputHeight,
 310:                     outputWidth,
 311:                     kH,
 312:                     kW,
 313:                     dH,
 314:                     dW,
 315:                     padH,
 316:                     padW,
 317:                     output_data,
 318:                     divisor_override_value,
 319:                     count_include_pad,
 320:                     use_divisor);
 321:             C10_CUDA_KERNEL_LAUNCH_CHECK();
 322:             break;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 323-336
```cpp
 323:           }
 324:           case MemoryFormat::Contiguous: {
 325:             avg_pool2d_out_cuda_frame<scalar_t, accscalar_t>
 326:                 <<<num_blocks,
 327:                    num_threads,
 328:                    0,
 329:                    at::cuda::getCurrentCUDAStream()>>>(
 330:                     count,
 331:                     input_data,
 332:                     nInputPlane,
 333:                     inputHeight,
 334:                     inputWidth,
 335:                     outputHeight,
 336:                     outputWidth,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 337-350
```cpp
 337:                     kH,
 338:                     kW,
 339:                     dH,
 340:                     dW,
 341:                     padH,
 342:                     padW,
 343:                     output_data,
 344:                     divisor_override_value,
 345:                     count_include_pad,
 346:                     use_divisor);
 347:             C10_CUDA_KERNEL_LAUNCH_CHECK();
 348:             break;
 349:           }
 350:           default: TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 351-355
```cpp
 351:         }
 352:       }
 353:     );
 354:   }
 355: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 357-378
```cpp
 357: TORCH_IMPL_FUNC(avg_pool2d_backward_out_cuda) (
 358:   const Tensor& gradOutput_,
 359:   const Tensor& input_,
 360:   IntArrayRef kernel_size,
 361:   IntArrayRef stride,
 362:   IntArrayRef padding,
 363:   bool ceil_mode,
 364:   bool count_include_pad,
 365:   std::optional<int64_t> divisor_override,
 366:   const Tensor& gradInput
 367: ) {
 368:   TensorArg gradInput_arg{ gradInput, "gradInput", 1 };
 369:   TensorArg gradOutput_arg{ gradOutput_, "gradOutput_", 2 };
 370:   TensorArg input_arg{ input_, "input_", 3 };
 371: 
 372:   checkAllSameGPU("avg_pool2d_backward_out_cuda",
 373:                   {gradInput_arg, gradOutput_arg, input_arg});
 374: 
 375:   const int kH = safe_downcast<int, int64_t>(kernel_size[0]);
 376:   const int kW = kernel_size.size() == 1 ? kH : safe_downcast<int, int64_t>(kernel_size[1]);
 377: 
 378:   const int dH = stride.empty() ? kH : safe_downcast<int, int64_t>(stride[0]);
```
- EN: This block defines or continues the implementation of `avg_pool2d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `avg_pool2d_backward_out_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 379-380
```cpp
 379:   const int dW = stride.empty() ? kW :
 380:                  stride.size() == 1 ? dH : safe_downcast<int, int64_t>(stride[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 382-383
```cpp
 382:   const int padH = safe_downcast<int, int64_t>(padding[0]);
 383:   const int padW = padding.size() == 1 ? padH : safe_downcast<int, int64_t>(padding[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 385-387
```cpp
 385:   const auto memory_format = input_.suggest_memory_format();
 386:   const Tensor input = input_.contiguous(memory_format);
 387:   const Tensor gradOutput = gradOutput_.contiguous(memory_format);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 389-391
```cpp
 389:   const int64_t nInputPlane = input.size(-3);
 390:   const int64_t inputHeight = input.size(-2);
 391:   const int64_t inputWidth = input.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 393-394
```cpp
 393:   const int64_t outputHeight = pooling_output_shape<int64_t>(inputHeight, kH, padH, dH, 1, ceil_mode);
 394:   const int64_t outputWidth = pooling_output_shape<int64_t>(inputWidth, kW, padW, dW, 1, ceil_mode);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 397-400
```cpp
 397:   const auto count = input.numel();
 398:   if (count == 0) {
 399:     return;
 400:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 402-403
```cpp
 402:   bool use_divisor = divisor_override.has_value();
 403:   const auto divisor_override_value = use_divisor ? divisor_override.value() : 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 405-410
```cpp
 405:   cudaDeviceProp* properties = at::cuda::getCurrentDeviceProperties();
 406:   const bool gesm10x = properties->major >= 10;
 407:   int double_threads = 1024;
 408:   if (gesm10x) {
 409:     double_threads = 768;
 410:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 412-433
```cpp
 412:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 413:     "avg_pool2d_backward_out_cuda_frame",
 414:     [&] {
 415:       const uint32_t num_threads = std::min(at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, std::is_same<scalar_t, double>::value ? double_threads : 1024);
 416:       const uint32_t num_blocks = ceil_div<uint32_t>(count, num_threads);
 417: 
 418:       using accscalar_t = acc_type<scalar_t, true>;
 419: 
 420:       const scalar_t *gradOutput_data = gradOutput.const_data_ptr<scalar_t>();
 421:       scalar_t *gradInput_data = gradInput.mutable_data_ptr<scalar_t>();
 422: 
 423:       AT_DISPATCH_INDEX_TYPES(
 424:         at::native::canUse32BitIndexMath(input, INT_MAX) ? ScalarType::Int : ScalarType::Long,
 425:         "avg_pool2d_backward_out_cuda_frame_launcher",
 426:         [&] {
 427:               switch (memory_format) {
 428: 
 429:                 case MemoryFormat::ChannelsLast: {
 430:                   gradInput.unsafeGetTensorImpl()->empty_tensor_restride(MemoryFormat::ChannelsLast);
 431:                   avg_pool2d_backward_out_cuda_frame_nhwc<scalar_t, accscalar_t, index_t>
 432:                     <<<num_blocks, num_threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 433:                       count,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 434-447
```cpp
 434:                       gradOutput_data,
 435:                       nInputPlane,
 436:                       inputHeight, inputWidth,
 437:                       outputHeight, outputWidth,
 438:                       kH, kW,
 439:                       dH, dW,
 440:                       padH, padW,
 441:                       gradInput_data,
 442:                       divisor_override_value,
 443:                       count_include_pad, use_divisor);
 444:                   C10_CUDA_KERNEL_LAUNCH_CHECK();
 445:                   break;
 446:                 }
 447:                 case MemoryFormat::Contiguous: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 448-461
```cpp
 448:                   avg_pool2d_backward_out_cuda_frame<scalar_t, accscalar_t, index_t>
 449:                     <<<num_blocks, num_threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 450:                       count,
 451:                       gradOutput_data,
 452:                       nInputPlane,
 453:                       inputHeight, inputWidth,
 454:                       outputHeight, outputWidth,
 455:                       kH, kW,
 456:                       dH, dW,
 457:                       padH, padW,
 458:                       gradInput_data,
 459:                       divisor_override_value,
 460:                       count_include_pad, use_divisor);
 461:                   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 462-468
```cpp
 462:                   break;
 463:                 }
 464:                 default: TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
 465:               }
 466:             });
 467:         });
 468: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 470-470
```cpp
 470: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_IMPL_FUNC` provides an out/inplace-style structured kernel implementation. / `TORCH_IMPL_FUNC` 提供结构化的 out/inplace 风格内核实现。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/native/Pool.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `AT_DISPATCH_INDEX_TYPES`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

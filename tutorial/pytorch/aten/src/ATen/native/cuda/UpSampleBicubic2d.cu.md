# UpSampleBicubic2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/UpSampleBicubic2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `upsample_bicubic2d_out_cuda`, `upsample_bicubic2d_backward_out_cuda`, `upsample_bicubic2d_out_cuda_template`, `upsample_bicubic2d_backward_out_cuda_template`.
- 用途（中文）: 实现与 `upsample_bicubic2d_out_cuda`, `upsample_bicubic2d_backward_out_cuda`, `upsample_bicubic2d_out_cuda_template`, `upsample_bicubic2d_backward_out_cuda_template` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/TensorUtils.h>
   7: #include <ATen/Utils.h>
   8: #include <ATen/cuda/CUDAContext.h>
   9: #include <ATen/native/cuda/UpSample.cuh>
  10: #include <c10/util/irange.h>
  11: 
  12: #ifndef AT_PER_OPERATOR_HEADERS
  13: #include <ATen/Functions.h>
  14: #include <ATen/NativeFunctions.h>
  15: #else
  16: #include <ATen/ops/upsample_bicubic2d_native.h>
  17: #include <ATen/ops/upsample_bicubic2d_backward_native.h>
  18: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 20-41
```cpp
  20: namespace at::native {
  21: namespace {
  22: 
  23: template <typename scalar_t, typename accscalar_t>
  24: C10_LAUNCH_BOUNDS_1(1024)
  25: __global__ void upsample_bicubic2d_out_frame(
  26:     const int num_elements,
  27:     const accscalar_t height_scale,
  28:     const accscalar_t width_scale,
  29:     const bool align_corners,
  30:     const PackedTensorAccessor64<const scalar_t, 4> idata,
  31:     PackedTensorAccessor64<scalar_t, 4> odata) {
  32:   int index = threadIdx.x + blockIdx.x * blockDim.x;
  33: 
  34:   const int batchsize = idata.size(0);
  35:   const int channels = idata.size(1);
  36:   const int input_height = idata.size(2);
  37:   const int input_width = idata.size(3);
  38:   const int output_height = odata.size(2);
  39:   const int output_width = odata.size(3);
  40: 
  41:   if (index >= num_elements) {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `upsample_bicubic2d_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_bicubic2d_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 42-43
```cpp
  42:     return;
  43:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 45-45
```cpp
  45:   // Special case: input and output are the same size, just copy
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 46-47
```cpp
  46:   const int output_x = index % output_width;
  47:   const int output_y = index / output_width;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 49-57
```cpp
  49:   if (input_height == output_height && input_width == output_width) {
  50:     for (int n = 0; n < batchsize; n++) {
  51:       for (int c = 0; c < channels; c++) {
  52:         const scalar_t val = idata[n][c][output_y][output_x];
  53:         odata[n][c][output_y][output_x] = val;
  54:       }
  55:     }
  56:     return;
  57:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 59-59
```cpp
  59:   // Interpolation kernel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 60-63
```cpp
  60:   accscalar_t real_x = area_pixel_compute_source_index(
  61:       width_scale, output_x, align_corners, /*cubic=*/true);
  62:   int in_x = floorf(real_x);
  63:   accscalar_t t_x = real_x - in_x;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 65-68
```cpp
  65:   accscalar_t real_y = area_pixel_compute_source_index(
  66:       height_scale, output_y, align_corners, /*cubic=*/true);
  67:   int in_y = floorf(real_y);
  68:   accscalar_t t_y = real_y - in_y;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 70-91
```cpp
  70:   for (int n = 0; n < batchsize; n++) {
  71:     for (int c = 0; c < channels; c++) {
  72:       accscalar_t coefficients[4];
  73: 
  74:       for (int k = 0; k < 4; k++) {
  75:         coefficients[k] = cubic_interp1d(
  76:             upsample_get_value_bounded<scalar_t>(
  77:                 idata, n, c, input_height, input_width, in_y - 1 + k, in_x - 1),
  78:             upsample_get_value_bounded<scalar_t>(
  79:                 idata, n, c, input_height, input_width, in_y - 1 + k, in_x + 0),
  80:             upsample_get_value_bounded<scalar_t>(
  81:                 idata, n, c, input_height, input_width, in_y - 1 + k, in_x + 1),
  82:             upsample_get_value_bounded<scalar_t>(
  83:                 idata, n, c, input_height, input_width, in_y - 1 + k, in_x + 2),
  84:             t_x);
  85:       }
  86: 
  87:       odata[n][c][output_y][output_x] = static_cast<scalar_t>(cubic_interp1d(
  88:           coefficients[0],
  89:           coefficients[1],
  90:           coefficients[2],
  91:           coefficients[3],
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 92-95
```cpp
  92:           t_y));
  93:     }
  94:   }
  95: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 97-99
```cpp
  97: // Parallelized across batch*channels via blockIdx.z.
  98: // Faster than upsample_bicubic2d_out_frame when the output spatial size is
  99: // small (e.g. Kimi K2.5 position embeddings: 64x64 to 74x74 with 1152 channels).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 100-120
```cpp
 100: template <typename scalar_t, typename accscalar_t>
 101: C10_LAUNCH_BOUNDS_1(1024)
 102: __global__ void upsample_bicubic2d_out_frame_parallel(
 103:     const int num_elements,
 104:     const accscalar_t height_scale,
 105:     const accscalar_t width_scale,
 106:     const bool align_corners,
 107:     const PackedTensorAccessor64<const scalar_t, 4> idata,
 108:     PackedTensorAccessor64<scalar_t, 4> odata) {
 109:   int index = threadIdx.x + blockIdx.x * blockDim.x;
 110: 
 111:   const int batchsize = idata.size(0);
 112:   const int channels = idata.size(1);
 113:   const int input_height = idata.size(2);
 114:   const int input_width = idata.size(3);
 115:   const int output_height = odata.size(2);
 116:   const int output_width = odata.size(3);
 117: 
 118:   if (index >= num_elements) {
 119:     return;
 120:   }
```
- EN: This block defines GPU kernel entry point(s) `upsample_bicubic2d_out_frame_parallel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_bicubic2d_out_frame_parallel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 122-123
```cpp
 122:   const int output_x = index % output_width;
 123:   const int output_y = index / output_width;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 125-133
```cpp
 125:   if (input_height == output_height && input_width == output_width) {
 126:     for (int i = blockIdx.z; i < batchsize * channels; i += gridDim.z) {
 127:       int n = i / channels;
 128:       int c = i % channels;
 129:       const scalar_t val = idata[n][c][output_y][output_x];
 130:       odata[n][c][output_y][output_x] = val;
 131:     }
 132:     return;
 133:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 135-138
```cpp
 135:   accscalar_t real_x = area_pixel_compute_source_index(
 136:       width_scale, output_x, align_corners, /*cubic=*/true);
 137:   int in_x = floorf(real_x);
 138:   accscalar_t t_x = real_x - in_x;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 140-143
```cpp
 140:   accscalar_t real_y = area_pixel_compute_source_index(
 141:       height_scale, output_y, align_corners, /*cubic=*/true);
 142:   int in_y = floorf(real_y);
 143:   accscalar_t t_y = real_y - in_y;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 145-166
```cpp
 145:   for (int i = blockIdx.z; i < batchsize * channels; i += gridDim.z) {
 146:     int n = i / channels;
 147:     int c = i % channels;
 148:     accscalar_t coefficients[4];
 149: 
 150:     for (const auto k : c10::irange(4)) {
 151:       coefficients[k] = cubic_interp1d(
 152:           upsample_get_value_bounded<scalar_t>(
 153:               idata, n, c, input_height, input_width, in_y - 1 + k, in_x - 1),
 154:           upsample_get_value_bounded<scalar_t>(
 155:               idata, n, c, input_height, input_width, in_y - 1 + k, in_x + 0),
 156:           upsample_get_value_bounded<scalar_t>(
 157:               idata, n, c, input_height, input_width, in_y - 1 + k, in_x + 1),
 158:           upsample_get_value_bounded<scalar_t>(
 159:               idata, n, c, input_height, input_width, in_y - 1 + k, in_x + 2),
 160:           t_x);
 161:     }
 162: 
 163:     odata[n][c][output_y][output_x] = static_cast<scalar_t>(cubic_interp1d(
 164:         coefficients[0],
 165:         coefficients[1],
 166:         coefficients[2],
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 167-170
```cpp
 167:         coefficients[3],
 168:         t_y));
 169:   }
 170: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 172-172
```cpp
 172: // Backward (adjoint) operation 1 <- 2 (accumulates)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 173-193
```cpp
 173: template <typename scalar_t, typename accscalar_t>
 174: C10_LAUNCH_BOUNDS_1(1024)
 175: __global__ void upsample_bicubic2d_backward_out_frame(
 176:     const int num_elements,
 177:     const accscalar_t height_scale,
 178:     const accscalar_t width_scale,
 179:     const bool align_corners,
 180:     PackedTensorAccessor64<scalar_t, 4> idata,
 181:     const PackedTensorAccessor64<const scalar_t, 4> odata) {
 182:   int index = threadIdx.x + blockIdx.x * blockDim.x;
 183: 
 184:   const int batchsize = idata.size(0);
 185:   const int channels = idata.size(1);
 186:   const int input_height = idata.size(2);
 187:   const int input_width = idata.size(3);
 188:   const int output_height = odata.size(2);
 189:   const int output_width = odata.size(3);
 190: 
 191:   if (index >= num_elements) {
 192:     return;
 193:   }
```
- EN: This block defines GPU kernel entry point(s) `upsample_bicubic2d_backward_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_bicubic2d_backward_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 195-206
```cpp
 195:   const int output_x = index % output_width;
 196:   const int output_y = index / output_width;
 197:   // special case: output_xust copy
 198:   if (input_height == output_height && input_width == output_width) {
 199:     for (int n = 0; n < batchsize; n++) {
 200:       for (int c = 0; c < channels; ++c) {
 201:         const scalar_t val = odata[n][c][output_y][output_x];
 202:         idata[n][c][output_y][output_x] = val;
 203:       }
 204:     }
 205:     return;
 206:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 208-211
```cpp
 208:   accscalar_t real_x = area_pixel_compute_source_index(
 209:       width_scale, output_x, align_corners, /*cubic=*/true);
 210:   int input_x = floorf(real_x);
 211:   accscalar_t t_x = real_x - input_x;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 213-216
```cpp
 213:   accscalar_t real_y = area_pixel_compute_source_index(
 214:       height_scale, output_y, align_corners, /*cubic=*/true);
 215:   int input_y = floorf(real_y);
 216:   accscalar_t t_y = real_y - input_y;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 218-219
```cpp
 218:   accscalar_t x_coeffs[4];
 219:   accscalar_t y_coeffs[4];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 221-222
```cpp
 221:   get_cubic_upsampling_coefficients(x_coeffs, t_x);
 222:   get_cubic_upsampling_coefficients(y_coeffs, t_y);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 224-241
```cpp
 224:   for (int n = 0; n < batchsize; n++) {
 225:     for (int c = 0; c < channels; ++c) {
 226:       scalar_t out_value = odata[n][c][output_y][output_x];
 227:       for (int i = 0; i < 4; i++) {
 228:         for (int j = 0; j < 4; j++) {
 229:           upsample_increment_value_bounded<scalar_t, accscalar_t>(
 230:               idata,
 231:               n,
 232:               c,
 233:               input_height,
 234:               input_width,
 235:               input_y - 1 + i,
 236:               input_x - 1 + j,
 237:               out_value * y_coeffs[i] * x_coeffs[j]);
 238:         }
 239:       }
 240:     }
 241:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 242-242
```cpp
 242: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 244-264
```cpp
 244: static void upsample_bicubic2d_out_cuda_template(
 245:     const Tensor& output,
 246:     const Tensor& input,
 247:     IntArrayRef output_size,
 248:     bool align_corners,
 249:     std::optional<double> scales_h,
 250:     std::optional<double> scales_w) {
 251:   TensorArg input_arg{input, "input", 1}, output_arg{output, "output", 2};
 252:   checkAllSameGPU(__func__, {input_arg, output_arg});
 253: 
 254:   int output_height = output_size[0];
 255:   int output_width = output_size[1];
 256: 
 257:   int input_height = input.size(2);
 258:   int input_width = input.size(3);
 259: 
 260:   output.zero_();
 261: 
 262:   const int num_output_elements = output_height * output_width;
 263:   const int max_threads = std::min(
 264:       at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, 1024);
```
- EN: This block defines or continues the implementation of `upsample_bicubic2d_out_cuda_template`.
- CN: 该代码块定义或继续实现 `upsample_bicubic2d_out_cuda_template`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 266-266
```cpp
 266:   // Launch kernel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 267-267
```cpp
 267:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 269-290
```cpp
 269:   AT_DISPATCH_FLOATING_TYPES_AND2(
 270:       at::ScalarType::Half, at::ScalarType::BFloat16,
 271:       input.scalar_type(), "upsample_bicubic2d_out_frame", [&] {
 272:         using accscalar_t = at::acc_type<scalar_t, true>;
 273: 
 274:         auto idata = input.packed_accessor64<const scalar_t, 4>();
 275:         auto odata = output.packed_accessor64<scalar_t, 4>();
 276: 
 277:         // Get scaling factors
 278:         const accscalar_t rheight = area_pixel_compute_scale<accscalar_t>(
 279:             input_height, output_height, align_corners, scales_h);
 280:         const accscalar_t rwidth = area_pixel_compute_scale<accscalar_t>(
 281:             input_width, output_width, align_corners, scales_w);
 282: 
 283:         const int num_blocks = ceil_div(num_output_elements, max_threads);
 284: 
 285:         // For small output spatial sizes the original kernel underutilizes
 286:         // the GPU because it loops over batch*channels sequentially.
 287:         // The parallel variant spreads that work across blockIdx.z instead.
 288:         // Threshold of 18432 covers all VLM position embedding resizing
 289:         // shapes (up to ~130x130 grids from 2048x2048 images) while
 290:         // staying well below the crossover point where the original
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 291-291
```cpp
 291:         // kernel catches up (~65k+ output elements).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 292-313
```cpp
 292:         if (num_output_elements <= 18432 && input.size(0) * input.size(1) > 0) {
 293:           int* maxGridSize = at::cuda::getCurrentDeviceProperties()->maxGridSize;
 294:           int grid_z = std::min<int>(maxGridSize[2],
 295:                                      input.size(0) * input.size(1));
 296:           dim3 grid(num_blocks, 1, grid_z);
 297:           upsample_bicubic2d_out_frame_parallel<scalar_t, accscalar_t>
 298:               <<<grid, max_threads, 0, stream>>>(
 299:                   num_output_elements,
 300:                   rheight,
 301:                   rwidth,
 302:                   align_corners,
 303:                   idata,
 304:                   odata);
 305:         } else {
 306:           upsample_bicubic2d_out_frame<scalar_t, accscalar_t>
 307:               <<<num_blocks, max_threads, 0, stream>>>(
 308:                   num_output_elements,
 309:                   rheight,
 310:                   rwidth,
 311:                   align_corners,
 312:                   idata,
 313:                   odata);
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 314-317
```cpp
 314:         }
 315:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 316:       });
 317: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 319-339
```cpp
 319: static void upsample_bicubic2d_backward_out_cuda_template(
 320:     const Tensor& grad_input,
 321:     const Tensor& grad_output_,
 322:     IntArrayRef output_size,
 323:     IntArrayRef input_size,
 324:     bool align_corners,
 325:     std::optional<double> scales_h,
 326:     std::optional<double> scales_w) {
 327:   TensorArg grad_input_arg{grad_input, "grad_input", 1},
 328:       grad_output_arg{grad_output_, "grad_output_", 2};
 329:   checkAllSameGPU(__func__, {grad_output_arg, grad_input_arg});
 330: 
 331:   int output_height = output_size[0];
 332:   int output_width = output_size[1];
 333: 
 334:   int input_height = input_size[2];
 335:   int input_width = input_size[3];
 336: 
 337:   Tensor grad_output = grad_output_.contiguous();
 338: 
 339:   grad_input.zero_();
```
- EN: This block defines or continues the implementation of `upsample_bicubic2d_backward_out_cuda_template`.
- CN: 该代码块定义或继续实现 `upsample_bicubic2d_backward_out_cuda_template`。

### Lines 341-344
```cpp
 341:   const int num_kernels = output_height * output_width;
 342:   const int num_threads = std::min(
 343:       at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, 1024);
 344:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 346-366
```cpp
 346:   AT_DISPATCH_FLOATING_TYPES_AND2(
 347:       at::ScalarType::Half, at::ScalarType::BFloat16,
 348:       grad_output.scalar_type(), "upsample_bicubic2d_backward_out_frame", [&] {
 349:         using accscalar_t = at::acc_type<scalar_t, true>;
 350: 
 351:         auto idata = grad_input.packed_accessor64<scalar_t, 4>();
 352:         auto odata = grad_output.packed_accessor64<const scalar_t, 4>();
 353: 
 354:         const accscalar_t rheight = area_pixel_compute_scale<accscalar_t>(
 355:             input_height, output_height, align_corners, scales_h);
 356:         const accscalar_t rwidth = area_pixel_compute_scale<accscalar_t>(
 357:             input_width, output_width, align_corners, scales_w);
 358: 
 359:         upsample_bicubic2d_backward_out_frame<scalar_t, accscalar_t>
 360:             <<<ceil_div(num_kernels, num_threads),
 361:                num_threads,
 362:                0,
 363:                stream>>>(
 364:                 num_kernels, rheight, rwidth, align_corners, idata, odata);
 365:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 366:       });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 367-367
```cpp
 367: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 369-369
```cpp
 369: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 371-379
```cpp
 371: TORCH_IMPL_FUNC(upsample_bicubic2d_out_cuda) (
 372:     const Tensor& input,
 373:     IntArrayRef output_size,
 374:     bool align_corners,
 375:     std::optional<double> scales_h,
 376:     std::optional<double> scales_w,
 377:     const Tensor& output) {
 378:   upsample_bicubic2d_out_cuda_template(output, input, output_size, align_corners, scales_h, scales_w);
 379: }
```
- EN: This block defines or continues the implementation of `upsample_bicubic2d_out_cuda`.
- CN: 该代码块定义或继续实现 `upsample_bicubic2d_out_cuda`。

### Lines 381-394
```cpp
 381: TORCH_IMPL_FUNC(upsample_bicubic2d_backward_out_cuda) (
 382:     const Tensor& grad_output,
 383:     IntArrayRef output_size,
 384:     IntArrayRef input_size,
 385:     bool align_corners,
 386:     std::optional<double> scales_h,
 387:     std::optional<double> scales_w,
 388:     const Tensor& grad_input) {
 389:   // See Note [Writing Nondeterministic Operations]
 390:   // Nondeterministic because of atomicAdd usage
 391:   globalContext().alertNotDeterministic("upsample_bicubic2d_backward_out_cuda");
 392:   upsample_bicubic2d_backward_out_cuda_template(
 393:       grad_input, grad_output, output_size, input_size, align_corners, scales_h, scales_w);
 394: }
```
- EN: This block defines or continues the implementation of `upsample_bicubic2d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `upsample_bicubic2d_backward_out_cuda`。

### Lines 396-396
```cpp
 396: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_IMPL_FUNC` provides an out/inplace-style structured kernel implementation. / `TORCH_IMPL_FUNC` 提供结构化的 out/inplace 风格内核实现。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/ceil_div.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/TensorUtils.h>`
  - `<ATen/Utils.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/native/cuda/UpSample.cuh>`
  - `<c10/util/irange.h>`
  - `<ATen/Functions.h>`
  - `<ATen/NativeFunctions.h>`
  - `<ATen/ops/upsample_bicubic2d_native.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

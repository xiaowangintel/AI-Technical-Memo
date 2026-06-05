# UpSampleNearest2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/UpSampleNearest2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `upsample_nearest2d_out_cuda`, `_upsample_nearest_exact2d_out_cuda`, `upsample_nearest2d_backward_out_cuda`, `_upsample_nearest_exact2d_backward_out_cuda`.
- 用途（中文）: 实现与 `upsample_nearest2d_out_cuda`, `_upsample_nearest_exact2d_out_cuda`, `upsample_nearest2d_backward_out_cuda`, `_upsample_nearest_exact2d_backward_out_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/TensorUtils.h>
   7: #include <ATen/Utils.h>
   8: #include <ATen/cuda/CUDAContext.h>
   9: #include <ATen/native/cuda/LaunchUtils.h>
  10: #include <ATen/native/cuda/UpSample.cuh>
  11: #include <ATen/native/cuda/KernelUtils.cuh>
  12: #include <ATen/cuda/detail/KernelUtils.h>
  13: 
  14: #ifndef AT_PER_OPERATOR_HEADERS
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 15-23
```cpp
  15: #include <ATen/Functions.h>
  16: #include <ATen/NativeFunctions.h>
  17: #else
  18: #include <ATen/ops/_upsample_nearest_exact2d_backward_native.h>
  19: #include <ATen/ops/_upsample_nearest_exact2d_native.h>
  20: #include <ATen/ops/empty.h>
  21: #include <ATen/ops/upsample_nearest2d_backward_native.h>
  22: #include <ATen/ops/upsample_nearest2d_native.h>
  23: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_upsample_nearest_exact2d_backward_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_upsample_nearest_exact2d_backward_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 25-46
```cpp
  25: namespace at::native {
  26: namespace {
  27: 
  28: #define MAX_THREADS 512
  29: 
  30: // Define a typedef to dispatch to nearest_neighbor_compute_source_index or
  31: // nearest_neighbor_exact_compute_source_index
  32: typedef int (*nn_compute_source_index_fn_t)(const float, int, int);
  33: 
  34: // Define a typedef to dispatch to nearest_neighbor_bw_compute_source_index or
  35: // nearest_neighbor_exact_bw_compute_source_index
  36: typedef int (*nn_bw_compute_source_index_fn_t)(const float, int, int);
  37: 
  38: // see NOTE [ Nearest neighbor upsampling kernel implementation ]
  39: template <typename scalar_t, nn_compute_source_index_fn_t nn_compute_source_index_fn>
  40: C10_LAUNCH_BOUNDS_1(1024)
  41: __global__ void upsample_nearest2d_out_frame(
  42:     const scalar_t* idata,
  43:     scalar_t* odata,
  44:     const size_t nc,
  45:     const size_t height1,
  46:     const size_t width1,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `upsample_nearest2d_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_nearest2d_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 47-68
```cpp
  47:     const size_t height2,
  48:     const size_t width2,
  49:     float height_scale,
  50:     float width_scale) {
  51:   size_t nc_iter = threadIdx.z + blockIdx.z * blockDim.z;
  52:   int64_t w2 = ((int64_t) threadIdx.x) + blockIdx.x * blockDim.x;
  53:   int64_t h2 = threadIdx.y + blockIdx.y * blockDim.y;
  54: 
  55:   if (w2 >= width2 || h2 >= height2) {
  56:     return;
  57:   }
  58: 
  59:   int64_t nc_stride = ((int64_t) blockDim.z) * gridDim.z;
  60: 
  61:   const size_t h1 = height1 == height2
  62:       ? h2
  63:       : nn_compute_source_index_fn(height_scale, h2, height1);
  64:   const size_t w1 = width1 == width2
  65:       ? w2
  66:       : nn_compute_source_index_fn(width_scale, w2, width1);
  67: 
  68:   size_t src_index = (nc_iter * height1 + h1) * width1 + w1;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 69-71
```cpp
  69:   size_t src_index_stride = nc_stride * width1 * height1;
  70:   size_t dst_index = (nc_iter * height2 + h2) * width2 + w2;
  71:   size_t dst_index_stride = nc_stride * width2 * height2;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 73-73
```cpp
  73:   // iterating over
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 74-80
```cpp
  74:   while (nc_iter < nc) {
  75:     odata[dst_index] = idata[src_index];
  76:     dst_index += dst_index_stride;
  77:     src_index += src_index_stride;
  78:     nc_iter += nc_stride;
  79:   }
  80: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 82-102
```cpp
  82: template <typename scalar_t, nn_compute_source_index_fn_t nn_compute_source_index_fn>
  83: C10_LAUNCH_BOUNDS_1(1024)
  84: __global__ void upsample_nearest2d_nhwc_out_frame(
  85:     const scalar_t* idata,
  86:     scalar_t* odata,
  87:     const size_t channels,
  88:     const size_t height1,
  89:     const size_t width1,
  90:     const size_t height2,
  91:     const size_t width2,
  92:     float height_scale,
  93:     float width_scale,
  94:     const size_t out_numel) {
  95: 
  96:     const int64_t index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
  97: 
  98:     if (index < out_numel) {
  99:     const auto c = index % channels;
 100:     const auto w2 = (index / channels) % width2;
 101:     const auto h2 = (index / channels / width2) % height2;
 102:     const auto n = index / channels / width2 / height2;
```
- EN: This block defines GPU kernel entry point(s) `upsample_nearest2d_nhwc_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_nearest2d_nhwc_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 104-105
```cpp
 104:     const size_t h1 = height1 == height2 ? h2 : nn_compute_source_index_fn(height_scale, h2, height1);
 105:     const size_t w1 = width1 == width2 ? w2 : nn_compute_source_index_fn(width_scale, w2, width1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 107-109
```cpp
 107:     odata[index] = idata[idx_cl(n, h1, w1, c, height1, width1, channels)];
 108:   }
 109: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 111-111
```cpp
 111: // see NOTE [ Nearest neighbor upsampling kernel implementation ]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 112-132
```cpp
 112: template <typename scalar_t, typename accscalar_t, nn_bw_compute_source_index_fn_t nn_bw_compute_source_index_fn>
 113: C10_LAUNCH_BOUNDS_1(1024)
 114: __global__ void upsample_nearest2d_backward_out_frame(
 115:     const scalar_t* grad_o,
 116:     size_t dim_b,
 117:     size_t dim_c,
 118:     size_t src_dim_h,
 119:     size_t src_dim_w,
 120:     size_t dst_dim_h,
 121:     size_t dst_dim_w,
 122:     scalar_t* grad_i,
 123:     float height_scale,
 124:     float width_scale) {
 125:   int64_t dst_idx = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
 126:   if (dst_idx >= dim_c * dst_dim_h * dst_dim_w)
 127:     return;
 128: 
 129:   int64_t dst_c_stride = dst_dim_h * dst_dim_w;
 130:   int64_t src_c_stride = src_dim_h * src_dim_w;
 131: 
 132:   int c = (dst_idx / (dst_c_stride)) % dim_c;
```
- EN: This block defines GPU kernel entry point(s) `upsample_nearest2d_backward_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_nearest2d_backward_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 134-140
```cpp
 134:   int dst_y = (dst_idx / dst_dim_w) % dst_dim_h;
 135:   // note that we do not want to clamp src_y to src_dim_y, since we might
 136:   // intentionally want to skip in case of scale_factor < 1.0
 137:   int src_y =
 138:       nn_bw_compute_source_index_fn(height_scale, dst_y, src_dim_h);
 139:   int src_y_up = nn_bw_compute_source_index_fn(
 140:       height_scale, dst_y + 1, src_dim_h);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 142-148
```cpp
 142:   int dst_x = dst_idx % dst_dim_w;
 143:   // note that we do not want to clamp src_x to src_dim_w, since we might
 144:   // intentionally want to skip in case of scale_factor < 1.0
 145:   int src_x =
 146:       nn_bw_compute_source_index_fn(width_scale, dst_x, src_dim_w);
 147:   int src_x_up = nn_bw_compute_source_index_fn(
 148:       width_scale, dst_x + 1, src_dim_w);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 150-162
```cpp
 150:   for (int b = 0; b < dim_b; b++) {
 151:     accscalar_t grad = 0;
 152:     for (int y = src_y; y < src_y_up; y++) {
 153:       for (int x = src_x; x < src_x_up; x++) {
 154:         int64_t src_idx =
 155:             b * dim_c * src_c_stride + c * src_c_stride + y * src_dim_w + x;
 156:         grad += grad_o[src_idx];
 157:       }
 158:     }
 159:     grad_i[dst_idx] = grad;
 160:     dst_idx += dim_c * dst_c_stride;
 161:   }
 162: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 164-185
```cpp
 164: template <typename scalar_t, typename accscalar_t, nn_bw_compute_source_index_fn_t nn_bw_compute_source_index_fn>
 165: C10_LAUNCH_BOUNDS_1(1024)
 166: __global__ void upsample_nearest2d_backward_nhwc_out_frame(
 167:     const scalar_t* go,
 168:     scalar_t* gi,
 169:     const size_t height1,
 170:     const size_t width1,
 171:     const size_t height2,
 172:     const size_t width2,
 173:     const size_t channels,
 174:     const float height_scale,
 175:     const float width_scale,
 176:     const size_t gi_numel) {
 177: 
 178:   // 1 is for grad_output (src)
 179:   // 2 is for grad_input (dst)
 180: 
 181:   const int64_t index = ((int64_t) blockIdx.x) * blockDim.x + threadIdx.x;
 182: 
 183:   if (index < gi_numel) {
 184:     const int c = index % channels;
 185:     const int w2 = (index / channels) % width2;
```
- EN: This block defines GPU kernel entry point(s) `upsample_nearest2d_backward_nhwc_out_frame`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `upsample_nearest2d_backward_nhwc_out_frame`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 186-187
```cpp
 186:     const int h2 = (index / channels / width2) % height2;
 187:     const int n = index / channels / width2 / height2;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 189-190
```cpp
 189:     int h1 = nn_bw_compute_source_index_fn(height_scale, h2, height1);
 190:     int h1_up = nn_bw_compute_source_index_fn(height_scale, h2 + 1, height1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 192-193
```cpp
 192:     int w1 = nn_bw_compute_source_index_fn(width_scale, w2, width1);
 193:     int w1_up = nn_bw_compute_source_index_fn(width_scale, w2 + 1, width1);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 195-203
```cpp
 195:     accscalar_t grad = 0;
 196:     for (int ih = h1; ih < h1_up; ih++) {
 197:       for (int iw = w1; iw < w1_up; iw++) {
 198:         grad += go[idx_cl(n, ih, iw, c, height1, width1, channels)];
 199:       }
 200:     }
 201:     gi[index] = static_cast<scalar_t>(grad);
 202:   }
 203: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 205-225
```cpp
 205: template<nn_compute_source_index_fn_t nn_compute_source_index_fn>
 206: static void upsample_nearest2d_out_cuda_template(
 207:     const Tensor& output,
 208:     const Tensor& input_,
 209:     IntArrayRef output_size,
 210:     std::optional<double> scales_h,
 211:     std::optional<double> scales_w) {
 212:   TensorArg input_arg{input_, "input_", 1}, output_arg{output, "output", 2};
 213:   checkAllSameGPU(__func__, {input_arg, output_arg});
 214: 
 215:   if (input_.numel() == 0) {
 216:     return;
 217:   }
 218: 
 219:   int output_height = output_size[0];
 220:   int output_width = output_size[1];
 221: 
 222:   int nbatch = input_.size(0);
 223:   int channels = input_.size(1);
 224:   int input_height = input_.size(2);
 225:   int input_width = input_.size(3);
```
- EN: This block defines or continues the implementation of `upsample_nearest2d_out_cuda_template`.
- CN: 该代码块定义或继续实现 `upsample_nearest2d_out_cuda_template`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 227-228
```cpp
 227:   const float height_scale = compute_scales_value<float>(scales_h, input_height, output_height);
 228:   const float width_scale = compute_scales_value<float>(scales_w, input_width, output_width);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 230-230
```cpp
 230:   const auto memory_format = input_.suggest_memory_format();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 232-235
```cpp
 232:   if (input_.sizes() == output.sizes()) {
 233:     output.copy_(input_);
 234:     return;
 235:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 237-237
```cpp
 237:   // heuristic: only use channels_last path when it's faster than the contiguous path
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 238-259
```cpp
 238:   if (memory_format == at::MemoryFormat::ChannelsLast && channels >= 4 && \
 239:         output.is_contiguous(memory_format)) {
 240:     at::Tensor input = input_.contiguous(at::MemoryFormat::ChannelsLast);
 241: 
 242:     TORCH_CHECK(input.numel() < std::numeric_limits<int64_t>::max(),
 243:       "upsample_nearest_nhwc only supports input tensors with less than 2^63 - 1 elements, but got ", input.sizes());
 244:     TORCH_CHECK(output.numel() < std::numeric_limits<int64_t>::max(),
 245:       "upsample_nearest_nhwc only supports output tensors with less than 2^63 - 1 elements, but got ", output.sizes());
 246: 
 247:     const int64_t num_kernels = output.numel();
 248:     const int64_t num_threads = std::min(at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, 1024);
 249: 
 250:     AT_DISPATCH_FLOATING_TYPES_AND3(ScalarType::Half, ScalarType::BFloat16, ScalarType::Byte, input.scalar_type(), "upsample_nearest2d_nhwc_out_frame", [&] {
 251:       const scalar_t* idata = input.const_data_ptr<scalar_t>();
 252:       scalar_t* odata = output.mutable_data_ptr<scalar_t>();
 253:       upsample_nearest2d_nhwc_out_frame<scalar_t, nn_compute_source_index_fn>
 254:         <<<ceil_div(num_kernels, num_threads), num_threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 255:           idata,
 256:           odata,
 257:           channels,
 258:           input_height,
 259:           input_width,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 260-272
```cpp
 260:           output_height,
 261:           output_width,
 262:           height_scale,
 263:           width_scale,
 264:           output.numel()
 265:       );
 266:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 267:     });
 268:   }
 269:   else {
 270:     // This is needed for non-contiguous tensors.
 271:     Tensor output_c = output.is_contiguous() ? output : at::empty(output.sizes(), output.options());
 272:     Tensor input = input_.contiguous();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 274-274
```cpp
 274:     int64_t nc = nbatch * channels;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 276-277
```cpp
 276:     const int max_threads = std::min<int>(
 277:         at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, MAX_THREADS);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 279-280
```cpp
 279:     int* maxThreadsDim = at::cuda::getCurrentDeviceProperties()->maxThreadsDim;
 280:     int* maxGridSize = at::cuda::getCurrentDeviceProperties()->maxGridSize;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 282-282
```cpp
 282:     // upsample_nearest2d meta call makes sure input/output tensor is not empty;
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 283-290
```cpp
 283:     int block_x = std::min<int>(
 284:         maxThreadsDim[0], std::min<int>(lastPow2(output_width), max_threads));
 285:     int block_y = std::min<int>(
 286:         maxThreadsDim[1],
 287:         std::min<int>(lastPow2(output_height), max_threads / block_x));
 288:     int block_z = std::min<int>(
 289:         maxThreadsDim[2], std::min<int>(nc, max_threads / block_x / block_y));
 290:     const dim3 block(block_x, block_y, block_z);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 292-305
```cpp
 292:     int grid_x = ceil_div(output_width, block_x);
 293:     int grid_y = ceil_div(output_height, block_y);
 294:     int grid_z = std::min<int>(
 295:         maxGridSize[2], ceil_div(nc, (int64_t) block_z * 4));
 296:     const dim3 grid(grid_x, grid_y, grid_z);
 297:     // Error out on cases where grid_x & grid_y exceeds limit of launch config, as
 298:     // the current kernel implementation doesn't loop over the two dimensions.
 299:     // This is unlikely to happen.
 300:     // TODO: kernel implementation could stride on spatial dimension. We probably
 301:     //       need to overhaul the kernel.
 302:     TORCH_CHECK(
 303:         grid_x <= maxGridSize[0] && grid_y <= maxGridSize[1],
 304:         "input tensor has spatial dimension larger than the kernel capacity");
 305:     cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 306-324
```cpp
 306:     AT_DISPATCH_FLOATING_TYPES_AND3(ScalarType::Half, ScalarType::BFloat16, ScalarType::Byte, input.scalar_type(), "upsample_nearest2d_out_frame", [&] {
 307:           using accscalar_t = at::acc_type<scalar_t, true>;
 308: 
 309:           auto idata = input.const_data_ptr<scalar_t>();
 310:           auto odata = output_c.mutable_data_ptr<scalar_t>();
 311: 
 312:           upsample_nearest2d_out_frame<scalar_t, nn_compute_source_index_fn>
 313:               <<<grid, block, 0, stream>>>(
 314:                   idata,
 315:                   odata,
 316:                   nc,
 317:                   input_height,
 318:                   input_width,
 319:                   output_height,
 320:                   output_width,
 321:                   height_scale,
 322:                   width_scale);
 323:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 324:         });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 326-330
```cpp
 326:     if (!output.is_contiguous()) {
 327:         output.copy_(output_c);
 328:     }
 329:   }
 330: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 332-353
```cpp
 332: template<nn_bw_compute_source_index_fn_t nn_bw_compute_source_index_fn>
 333: static void upsample_nearest2d_backward_out_cuda_template(
 334:     const Tensor& grad_input,
 335:     const Tensor& grad_output_,
 336:     IntArrayRef output_size,
 337:     IntArrayRef input_size,
 338:     std::optional<double> scales_h,
 339:     std::optional<double> scales_w) {
 340:   TensorArg grad_input_arg{grad_input, "grad_input", 1},
 341:       grad_output_arg{grad_output_, "grad_output_", 2};
 342:   checkAllSameGPU(__func__, {grad_output_arg, grad_input_arg});
 343: 
 344:   if (grad_input.numel() == 0) {
 345:     return;
 346:   }
 347: 
 348:   int output_height = output_size[0];
 349:   int output_width = output_size[1];
 350: 
 351:   int nbatch = input_size[0];
 352:   int channels = input_size[1];
 353:   int input_height = input_size[2];
```
- EN: This block defines or continues the implementation of `upsample_nearest2d_backward_out_cuda_template`.
- CN: 该代码块定义或继续实现 `upsample_nearest2d_backward_out_cuda_template`。

### Lines 354-354
```cpp
 354:   int input_width = input_size[3];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 356-357
```cpp
 356:   const float height_scale = compute_scales_value_backwards<float>(scales_h, output_height, input_height);
 357:   const float width_scale = compute_scales_value_backwards<float>(scales_w, output_width, input_width);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 359-359
```cpp
 359:   auto memory_format = grad_output_.suggest_memory_format();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 361-364
```cpp
 361:   if (grad_output_.sizes() == grad_input.sizes()) {
 362:     grad_input.copy_(grad_output_);
 363:     return;
 364:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 366-387
```cpp
 366:   if (memory_format == at::MemoryFormat::ChannelsLast && channels >= 4 && \
 367:         grad_input.is_contiguous(memory_format)) {
 368:     Tensor grad_output = grad_output_.contiguous(at::MemoryFormat::ChannelsLast);
 369: 
 370:     TORCH_CHECK(grad_input.numel() < std::numeric_limits<int>::max(),
 371:       "upsample_nearest_nhwc only supports grad_input tensors with less than INT_MAX elements, but got ", grad_input.sizes());
 372:     TORCH_CHECK(grad_output.numel() < std::numeric_limits<int>::max(),
 373:       "upsample_nearest_nhwc only supports grad_output tensors with less than INT_MAX elements, but got ", grad_output.sizes());
 374: 
 375:     const int num_kernels = grad_input.numel();
 376:     const int num_threads = std::min(at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, 1024);
 377: 
 378:     AT_DISPATCH_FLOATING_TYPES_AND3(ScalarType::Half, ScalarType::BFloat16, ScalarType::Byte, grad_output.scalar_type(), "upsample_nearest2d_backward_nhwc_out_frame", [&] {
 379:       using accscalar_t = at::acc_type<scalar_t, true>;
 380: 
 381:       const scalar_t* go = grad_output.const_data_ptr<scalar_t>();
 382:       scalar_t* gi = grad_input.mutable_data_ptr<scalar_t>();
 383: 
 384:       upsample_nearest2d_backward_nhwc_out_frame<scalar_t, accscalar_t, nn_bw_compute_source_index_fn>
 385:         <<<ceil_div(num_kernels, num_threads), num_threads, 0, at::cuda::getCurrentCUDAStream()>>>(
 386:           go,
 387:           gi,
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 388-401
```cpp
 388:           output_height,
 389:           output_width,
 390:           input_height,
 391:           input_width,
 392:           channels,
 393:           height_scale,
 394:           width_scale,
 395:           grad_input.numel()
 396:       );
 397:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 398:     });
 399:   } else {
 400:     // This is needed for non-contiguous tensors.
 401:     Tensor grad_input_c = grad_input.is_contiguous() ? grad_input : at::empty(grad_input.sizes(), grad_input.options());
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 402-402
```cpp
 402:     Tensor grad_output = grad_output_.contiguous();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 404-404
```cpp
 404:     // upsample_nearest2d meta call makes sure `nbatch != 0`
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 405-411
```cpp
 405:     size_t n = grad_input.numel() / nbatch;
 406:     dim3 bdim{std::min<unsigned int>(
 407:         at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, MAX_THREADS)};
 408:     dim3 gdim{(unsigned int) ceil_div(n, (size_t) bdim.x)};
 409:     // safe check for int64 indexing; implicitly restrict launch config for kernel
 410:     TORCH_CHECK(grad_input.numel() <= std::numeric_limits<int64_t>::max(), "upsample2d grad_input.numel() <= std::numeric_limits<int64_t>::max(), but got ", grad_input.sizes());
 411:     TORCH_CHECK(grad_output.numel() <= std::numeric_limits<int64_t>::max(), "upsample2d grad_output.numel() <= std::numeric_limits<int64_t>::max(), but got ", grad_output.sizes());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 413-434
```cpp
 413:     cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 414:     AT_DISPATCH_FLOATING_TYPES_AND3(ScalarType::Half, ScalarType::BFloat16, ScalarType::Byte, grad_output.scalar_type(), "upsample_nearest2d_backward_out_frame", [&] {
 415:       using accscalar_t = at::acc_type<scalar_t, true>;
 416: 
 417:       auto idata = grad_input_c.mutable_data_ptr<scalar_t>();
 418:       auto odata = grad_output.const_data_ptr<scalar_t>();
 419: 
 420: 
 421:       upsample_nearest2d_backward_out_frame<scalar_t, accscalar_t, nn_bw_compute_source_index_fn>
 422:           <<<gdim, bdim, 0, stream>>>(
 423:               odata,
 424:               nbatch,
 425:               channels,
 426:               output_height,
 427:               output_width,
 428:               input_height,
 429:               input_width,
 430:               idata,
 431:               height_scale,
 432:               width_scale);
 433:       C10_CUDA_KERNEL_LAUNCH_CHECK();
 434:     });
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 436-440
```cpp
 436:     if (!grad_input.is_contiguous()) {
 437:         grad_input.copy_(grad_input_c);
 438:     }
 439:   }
 440: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 442-442
```cpp
 442: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 444-452
```cpp
 444: TORCH_IMPL_FUNC(upsample_nearest2d_out_cuda) (
 445:     const Tensor& input,
 446:     IntArrayRef output_size,
 447:     std::optional<double> scales_h,
 448:     std::optional<double> scales_w,
 449:     const Tensor& output) {
 450:   upsample_nearest2d_out_cuda_template<nearest_neighbor_compute_source_index>(
 451:       output, input, output_size, scales_h, scales_w);
 452: }
```
- EN: This block defines or continues the implementation of `upsample_nearest2d_out_cuda`.
- CN: 该代码块定义或继续实现 `upsample_nearest2d_out_cuda`。

### Lines 454-462
```cpp
 454: TORCH_IMPL_FUNC(_upsample_nearest_exact2d_out_cuda) (
 455:     const Tensor& input,
 456:     IntArrayRef output_size,
 457:     std::optional<double> scales_h,
 458:     std::optional<double> scales_w,
 459:     const Tensor& output) {
 460:   upsample_nearest2d_out_cuda_template<nearest_neighbor_exact_compute_source_index>(
 461:       output, input, output_size, scales_h, scales_w);
 462: }
```
- EN: This block defines or continues the implementation of `_upsample_nearest_exact2d_out_cuda`.
- CN: 该代码块定义或继续实现 `_upsample_nearest_exact2d_out_cuda`。

### Lines 464-473
```cpp
 464: TORCH_IMPL_FUNC(upsample_nearest2d_backward_out_cuda) (
 465:     const Tensor& grad_output,
 466:     IntArrayRef output_size,
 467:     IntArrayRef input_size,
 468:     std::optional<double> scales_h,
 469:     std::optional<double> scales_w,
 470:     const Tensor& grad_input) {
 471:   upsample_nearest2d_backward_out_cuda_template<nearest_neighbor_bw_compute_source_index>(
 472:       grad_input, grad_output, output_size, input_size, scales_h, scales_w);
 473: }
```
- EN: This block defines or continues the implementation of `upsample_nearest2d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `upsample_nearest2d_backward_out_cuda`。

### Lines 475-484
```cpp
 475: TORCH_IMPL_FUNC(_upsample_nearest_exact2d_backward_out_cuda) (
 476:     const Tensor& grad_output,
 477:     IntArrayRef output_size,
 478:     IntArrayRef input_size,
 479:     std::optional<double> scales_h,
 480:     std::optional<double> scales_w,
 481:     const Tensor& grad_input) {
 482:   upsample_nearest2d_backward_out_cuda_template<nearest_neighbor_exact_bw_compute_source_index>(
 483:       grad_input, grad_output, output_size, input_size, scales_h, scales_w);
 484: }
```
- EN: This block defines or continues the implementation of `_upsample_nearest_exact2d_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `_upsample_nearest_exact2d_backward_out_cuda`。

### Lines 486-486
```cpp
 486: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_IMPL_FUNC` provides an out/inplace-style structured kernel implementation. / `TORCH_IMPL_FUNC` 提供结构化的 out/inplace 风格内核实现。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
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
  - `<ATen/native/cuda/LaunchUtils.h>`
  - `<ATen/native/cuda/UpSample.cuh>`
  - `<ATen/native/cuda/KernelUtils.cuh>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/Functions.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND3`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

# im2col.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/im2col.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `im2col`, `col2im_device`, `col2im`, `col2im_batched`.
- 用途（中文）: 声明或定义与 `im2col`, `col2im_device`, `col2im`, `col2im_batched` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
   1: #pragma once
   2: 
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/cuda/detail/KernelUtils.h>
   6: 
   7: #include <c10/macros/Macros.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/AccumulateType.h>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/cuda/detail/KernelUtils.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/AccumulateType.h>`, `<ATen/cuda/CUDAContext.h>`, `<ATen/cuda/detail/KernelUtils.h>`。

### Lines 9-30
```cpp
   9: namespace at::native {
  10: 
  11: using namespace at::cuda::detail;
  12: 
  13: // Kernel for fast unfold+copy
  14: // (borrowed from Caffe:
  15: // https://github.com/BVLC/caffe/blob/master/src/caffe/layers/conv_layer.cu)
  16: // CUDA_NUM_THREADS = 1024
  17: 
  18: template <typename dt>
  19: C10_LAUNCH_BOUNDS_1(1024)
  20: __global__ void im2col_kernel(
  21:     const int64_t n,
  22:     const dt* data_im,
  23:     const int64_t height,
  24:     const int64_t width,
  25:     const int64_t kernel_height,
  26:     const int64_t kernel_width,
  27:     const int64_t pad_height,
  28:     const int64_t pad_width,
  29:     const int64_t stride_height,
  30:     const int64_t stride_width,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `im2col_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `im2col_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 31-52
```cpp
  31:     const int64_t dilation_height,
  32:     const int64_t dilation_width,
  33:     const int64_t height_col,
  34:     const int64_t width_col,
  35:     dt* data_col) {
  36:   CUDA_KERNEL_LOOP_TYPE(index, n, int64_t) {
  37:     int64_t w_out = index % width_col;
  38: 
  39:     int64_t idx = index / width_col;
  40: 
  41:     int64_t h_out = idx % height_col;
  42:     int64_t channel_in = idx / height_col;
  43:     int64_t channel_out = channel_in * kernel_height * kernel_width;
  44:     int64_t h_in = h_out * stride_height - pad_height;
  45:     int64_t w_in = w_out * stride_width - pad_width;
  46: 
  47:     dt* col = data_col + (channel_out * height_col + h_out) * width_col + w_out;
  48:     const dt* im = data_im + (channel_in * height + h_in) * width + w_in;
  49: 
  50:     for (int64_t i = 0; i < kernel_height; ++i) {
  51:       for (int64_t j = 0; j < kernel_width; ++j) {
  52:         int64_t h = h_in + i * dilation_height;
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 53-61
```cpp
  53:         int64_t w = w_in + j * dilation_width;
  54:         *col = (h >= 0 && w >= 0 && h < height && w < width)
  55:             ? im[i * dilation_height * width + j * dilation_width]
  56:             : static_cast<dt>(0);
  57:         col += height_col * width_col;
  58:       }
  59:     }
  60:   }
  61: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 63-76
```cpp
  63: template <typename dt>
  64: void im2col(
  65:     cudaStream_t stream,
  66:     const dt* data_im,
  67:     const int64_t channels,
  68:     const int64_t height,
  69:     const int64_t width,
  70:     const int64_t height_col,
  71:     const int64_t width_col,
  72:     const int64_t kernel_height,
  73:     const int64_t kernel_width,
  74:     const int64_t pad_height,
  75:     const int64_t pad_width,
  76:     const int64_t stride_height,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 77-98
```cpp
  77:     const int64_t stride_width,
  78:     const int64_t dilation_height,
  79:     const int64_t dilation_width,
  80:     dt* data_col) {
  81:   // We are going to launch channels * height_col * width_col kernels, each
  82:   // kernel responsible for copying a single-channel grid.
  83:   int64_t num_kernels = channels * height_col * width_col;
  84:   // Launch CUDA_NUM_THREADS = 1024
  85:   im2col_kernel<<<GET_BLOCKS(num_kernels), 1024, 0, stream>>>(
  86:       num_kernels,
  87:       data_im,
  88:       height,
  89:       width,
  90:       kernel_height,
  91:       kernel_width,
  92:       pad_height,
  93:       pad_width,
  94:       stride_height,
  95:       stride_width,
  96:       dilation_height,
  97:       dilation_width,
  98:       height_col,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 99-102
```cpp
  99:       width_col,
 100:       data_col);
 101:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 102: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 104-117
```cpp
 104: template <typename accT, typename dt>
 105: __forceinline__ __device__ void col2im_device(
 106:     const int64_t index,
 107:     const dt* data_col,
 108:     const int64_t height,
 109:     const int64_t width,
 110:     const int64_t kernel_h,
 111:     const int64_t kernel_w,
 112:     const int64_t pad_height,
 113:     const int64_t pad_width,
 114:     const int64_t stride_height,
 115:     const int64_t stride_width,
 116:     const int64_t dilation_height,
 117:     const int64_t dilation_width,
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 118-139
```cpp
 118:     const int64_t height_col,
 119:     const int64_t width_col,
 120:     dt* data_im) {
 121:   accT val = static_cast<accT>(0);
 122:   const int64_t w_im = index % width + pad_width;
 123:   const int64_t h_im = (index / width) % height + pad_height;
 124:   const int64_t c_im = index / (width * height);
 125:   int64_t kernel_extent_w = (kernel_w - 1) * dilation_width + 1;
 126:   int64_t kernel_extent_h = (kernel_h - 1) * dilation_height + 1;
 127:   // compute the start and end of the output
 128:   const int64_t w_col_start = (w_im < kernel_extent_w)
 129:       ? 0
 130:       : (w_im - kernel_extent_w) / stride_width + 1;
 131:   const int64_t w_col_end = ::min(w_im / stride_width + 1, width_col);
 132:   const int64_t h_col_start = (h_im < kernel_extent_h)
 133:       ? 0
 134:       : (h_im - kernel_extent_h) / stride_height + 1;
 135:   const int64_t h_col_end = ::min(h_im / stride_height + 1, height_col);
 136: 
 137:   // TODO: use LCM of stride and dilation to avoid unnecessary loops
 138:   for (int64_t h_col = h_col_start; h_col < h_col_end; h_col += 1) {
 139:     for (int64_t w_col = w_col_start; w_col < w_col_end; w_col += 1) {
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 140-153
```cpp
 140:       int64_t h_k = (h_im - h_col * stride_height);
 141:       int64_t w_k = (w_im - w_col * stride_width);
 142:       if (h_k % dilation_height == 0 && w_k % dilation_width == 0) {
 143:         h_k /= dilation_height;
 144:         w_k /= dilation_width;
 145:         int64_t data_col_index =
 146:             (((c_im * kernel_h + h_k) * kernel_w + w_k) * height_col +
 147:               h_col) *
 148:                 width_col +
 149:             w_col;
 150:         val += data_col[data_col_index];
 151:       }
 152:     }
 153:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 154-155
```cpp
 154:   data_im[index] = static_cast<dt>(val);
 155: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 157-170
```cpp
 157: template <typename dt, typename accT>
 158: C10_LAUNCH_BOUNDS_1(512)
 159: __global__ void col2im_kernel(
 160:     const int64_t n,
 161:     const dt* data_col,
 162:     const int64_t height,
 163:     const int64_t width,
 164:     const int64_t kernel_h,
 165:     const int64_t kernel_w,
 166:     const int64_t pad_height,
 167:     const int64_t pad_width,
 168:     const int64_t stride_height,
 169:     const int64_t stride_width,
 170:     const int64_t dilation_height,
```
- EN: This block defines GPU kernel entry point(s) `col2im_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `col2im_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 171-192
```cpp
 171:     const int64_t dilation_width,
 172:     const int64_t height_col,
 173:     const int64_t width_col,
 174:     dt* data_im) {
 175:   CUDA_KERNEL_LOOP(index, n) {
 176:     col2im_device<accT>(
 177:         index,
 178:         data_col,
 179:         height,
 180:         width,
 181:         kernel_h,
 182:         kernel_w,
 183:         pad_height,
 184:         pad_width,
 185:         stride_height,
 186:         stride_width,
 187:         dilation_height,
 188:         dilation_width,
 189:         height_col,
 190:         width_col,
 191:         data_im);
 192:   }
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 193-193
```cpp
 193: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 195-208
```cpp
 195: template <typename dt, typename accT>
 196: void col2im(
 197:     cudaStream_t stream,
 198:     const dt* data_col,
 199:     const int64_t channels,
 200:     const int64_t height,
 201:     const int64_t width,
 202:     const int64_t height_col,
 203:     const int64_t width_col,
 204:     const int64_t patch_height,
 205:     const int64_t patch_width,
 206:     const int64_t pad_height,
 207:     const int64_t pad_width,
 208:     const int64_t stride_height,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 209-230
```cpp
 209:     const int64_t stride_width,
 210:     const int64_t dilation_height,
 211:     const int64_t dilation_width,
 212:     dt* data_im) {
 213:   int64_t num_kernels = channels * height * width;
 214:   // To avoid involving atomic operations, we will launch one kernel per
 215:   // bottom dimension, and then in the kernel add up the top dimensions.
 216:   // CUDA_NUM_THREADS = 1024
 217:   col2im_kernel<dt, accT>
 218:       <<<GET_BLOCKS(num_kernels, 512), 512, 0, stream>>>(
 219:           num_kernels,
 220:           data_col,
 221:           height,
 222:           width,
 223:           patch_height,
 224:           patch_width,
 225:           pad_height,
 226:           pad_width,
 227:           stride_height,
 228:           stride_width,
 229:           dilation_height,
 230:           dilation_width,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 231-235
```cpp
 231:           height_col,
 232:           width_col,
 233:           data_im);
 234:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 235: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 237-250
```cpp
 237: template <typename dt>
 238: C10_LAUNCH_BOUNDS_1(512)
 239: __global__ void col2im_batched_kernel(
 240:     const int64_t n,
 241:     const dt* data_col,
 242:     const int64_t col_batch_stride,
 243:     const int64_t nbatch,
 244:     const int64_t height,
 245:     const int64_t width,
 246:     const int64_t kernel_h,
 247:     const int64_t kernel_w,
 248:     const int64_t pad_height,
 249:     const int64_t pad_width,
 250:     const int64_t stride_height,
```
- EN: This block defines GPU kernel entry point(s) `col2im_batched_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `col2im_batched_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 251-272
```cpp
 251:     const int64_t stride_width,
 252:     const int64_t dilation_height,
 253:     const int64_t dilation_width,
 254:     const int64_t height_col,
 255:     const int64_t width_col,
 256:     dt* data_im,
 257:     const int64_t im_batch_stride) {
 258:   using accT = at::acc_type<dt, /*is_cuda*/true>;
 259:   const auto im_numel = n * nbatch;
 260: 
 261:   CUDA_KERNEL_LOOP_TYPE(index, im_numel, int64_t) {
 262:     const auto ibatch = index / n;
 263:     const auto slice_index = index % n;
 264: 
 265:     col2im_device<accT>(
 266:         slice_index,
 267:         data_col + ibatch * col_batch_stride,
 268:         height,
 269:         width,
 270:         kernel_h,
 271:         kernel_w,
 272:         pad_height,
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 273-282
```cpp
 273:         pad_width,
 274:         stride_height,
 275:         stride_width,
 276:         dilation_height,
 277:         dilation_width,
 278:         height_col,
 279:         width_col,
 280:         data_im + ibatch * im_batch_stride);
 281:   }
 282: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 284-297
```cpp
 284: template <typename dt>
 285: void col2im_batched(
 286:     cudaStream_t stream,
 287:     const dt* data_col,
 288:     const int64_t col_batch_stride,
 289:     const int64_t nbatch,
 290:     const int64_t channels,
 291:     const int64_t height,
 292:     const int64_t width,
 293:     const int64_t height_col,
 294:     const int64_t width_col,
 295:     const int64_t patch_height,
 296:     const int64_t patch_width,
 297:     const int64_t pad_height,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 298-319
```cpp
 298:     const int64_t pad_width,
 299:     const int64_t stride_height,
 300:     const int64_t stride_width,
 301:     const int64_t dilation_height,
 302:     const int64_t dilation_width,
 303:     dt* data_im,
 304:     const int64_t im_batch_stride) {
 305:   const int64_t num_kernels = channels * height * width;
 306:   const int64_t output_numel = nbatch * num_kernels;
 307:   if (output_numel == 0) {
 308:     return;  // No work to do
 309:   }
 310: 
 311:   // To avoid involving atomic operations, we will launch one kernel per
 312:   // bottom dimension, and then in the kernel add up the top dimensions.
 313:   // CUDA_NUM_THREADS = 1024
 314:   col2im_batched_kernel<<<GET_BLOCKS(output_numel, 512), 512, 0, stream>>>(
 315:           num_kernels,
 316:           data_col,
 317:           col_batch_stride,
 318:           nbatch,
 319:           height,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 320-333
```cpp
 320:           width,
 321:           patch_height,
 322:           patch_width,
 323:           pad_height,
 324:           pad_width,
 325:           stride_height,
 326:           stride_width,
 327:           dilation_height,
 328:           dilation_width,
 329:           height_col,
 330:           width_col,
 331:           data_im,
 332:           im_batch_stride);
 333:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 334-334
```cpp
 334: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 336-336
```cpp
 336: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/AccumulateType.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<c10/macros/Macros.h>`
- Runtime symbols / 运行时符号:
  - `at::cuda::detail`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

# DilatedMaxPool2d.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/DilatedMaxPool2d.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `max_pool2d_with_indices_out_cuda`, `max_pool2d_with_indices_backward_out_cuda`, `min`, `p_end`.
- 用途（中文）: 实现与 `max_pool2d_with_indices_out_cuda`, `max_pool2d_with_indices_backward_out_cuda`, `min`, `p_end` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/NamedTensorUtils.h>
   7: #include <ATen/NumericUtils.h>
   8: #include <ATen/native/Pool.h>
   9: #include <ATen/cuda/CUDAContext.h>
  10: #include <ATen/cuda/NumericLimits.cuh>
  11: #include <ATen/cuda/detail/TensorInfo.cuh>
  12: #include <ATen/cuda/detail/IndexUtils.cuh>
  13: #include <ATen/cuda/detail/KernelUtils.h>
  14: #include <c10/macros/Macros.h>
  15: #include <ATen/native/cuda/LaunchUtils.h>
  16: 
  17: #ifndef AT_PER_OPERATOR_HEADERS
  18: #include <ATen/NativeFunctions.h>
  19: #else
  20: #include <ATen/ops/max_pool2d_with_indices_native.h>
  21: #include <ATen/ops/max_pool2d_with_indices_backward_native.h>
  22: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<ATen/AccumulateType.h>`, `<ATen/ceil_div.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 24-45
```cpp
  24: namespace at::native {
  25: namespace {
  26: 
  27: __device__ inline int min(int a, int b) {
  28:   return a <= b ? a : b;
  29: }
  30: 
  31: #ifdef USE_ROCM
  32: #define CUDA_MAX_THREADS 256
  33: #define BLOCK_STRIDE_FWD 2 // increasing block_stride to lower # of blocks launched
  34: #define BLOCK_STRIDE_BWD 4 // increasing block_stride to lower # of blocks launched
  35: #else
  36: #define CUDA_MAX_THREADS 1024 // this is safe, in reality 256 is our limit
  37: #define BLOCK_STRIDE_FWD 2 // increasing block_stride to lower # of blocks launched
  38: #define BLOCK_STRIDE_BWD 2 // increasing block_stride to lower # of blocks launched
  39: #endif
  40: 
  41: template <typename index_t>
  42: static __device__ inline index_t p_start(index_t size, int pad, int kernel, int dilation, int stride) {
  43:   const auto kernel_extent = static_cast<index_t>((kernel - 1) * dilation + 1);
  44:   return (size + pad < kernel_extent) ? index_t(0) : (size + pad - kernel_extent) / stride + 1;
  45: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `min`, `p_start`.
- CN: 该代码块定义或继续实现 `min`, `p_start`。

### Lines 47-50
```cpp
  47: template <typename index_t>
  48: static __device__ inline index_t p_end(index_t size, int pad, index_t pooled_size, int stride) {
  49:   return std::min((size + pad) / stride + 1, pooled_size);
  50: }
```
- EN: This block defines or continues the implementation of `p_end`.
- CN: 该代码块定义或继续实现 `p_end`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 52-73
```cpp
  52: static inline bool can_use_int32_nhwc(
  53:     int64_t nbatch, int64_t channels,
  54:     int64_t height, int64_t width,
  55:     int64_t pooled_height, int64_t pooled_width,
  56:     int64_t in_stride_n, int64_t in_stride_c,
  57:     int64_t in_stride_h, int64_t in_stride_w)
  58: {
  59:   constexpr int64_t int_max = std::numeric_limits<int>::max();
  60: 
  61:   int64_t max_intra_batch =
  62:       (height ? (height - 1) * in_stride_h : 0) +
  63:       (width ? (width - 1) * in_stride_w : 0) +
  64:       (channels? (channels - 1) * in_stride_c : 0);
  65: 
  66:   int64_t max_input_offset = (nbatch ? (nbatch - 1) * in_stride_n : 0) + max_intra_batch;
  67: 
  68:   if (max_input_offset > int_max) return false;
  69: 
  70:   int64_t out_batch_stride = pooled_height * pooled_width * channels;
  71:   if ((nbatch ? (nbatch - 1) * out_batch_stride : 0) > int_max) return false;
  72: 
  73:   if (height * width > int_max) return false;
```
- EN: This block defines or continues the implementation of `can_use_int32_nhwc`.
- CN: 该代码块定义或继续实现 `can_use_int32_nhwc`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 75-76
```cpp
  75:   return true;
  76: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 78-91
```cpp
  78: static inline bool can_use_int32_nchw(
  79:     int64_t nbatch, int64_t channels,
  80:     int64_t height, int64_t width,
  81:     int64_t pooled_height, int64_t pooled_width) {
  82:   int64_t hw = height * width;
  83:   return can_use_int32_nhwc(
  84:       nbatch, channels, height, width,
  85:       pooled_height, pooled_width,
  86:       channels * hw,  // in_stride_n
  87:       hw, // in_stride_c
  88:       width, // in_stride_h
  89:       1 // in_stride_w
  90:   );
  91: }
```
- EN: This block defines or continues the implementation of `can_use_int32_nchw`.
- CN: 该代码块定义或继续实现 `can_use_int32_nchw`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 93-93
```cpp
  93: // kernels borrowed from Caffe
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 94-107
```cpp
  94: template <typename scalar_t, typename index_t>
  95: __global__ void max_pool_forward_nchw(
  96:     const index_t nthreads,
  97:     const scalar_t* bottom_data,
  98:     const int64_t channels,
  99:     const int64_t height,
 100:     const int64_t width,
 101:     const int pooled_height,
 102:     const int pooled_width,
 103:     const int kernel_h, const int kernel_w,
 104:     const int stride_h, const int stride_w,
 105:     const int pad_h, const int pad_w,
 106:     const int dilation_h, const int dilation_w,
 107:     scalar_t* top_data,
```
- EN: This block defines GPU kernel entry point(s) `max_pool_forward_nchw`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `max_pool_forward_nchw`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 108-129
```cpp
 108:     int64_t* top_mask) {
 109:   CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
 110:     index_t pw = index % pooled_width;
 111:     index_t ph = (index / pooled_width) % pooled_height;
 112:     index_t c = (index / pooled_width / pooled_height) % channels;
 113:     index_t n = index / pooled_width / pooled_height / channels;
 114:     index_t hstart = ph * stride_h - pad_h;
 115:     index_t wstart = pw * stride_w - pad_w;
 116:     index_t hend = min(hstart + (kernel_h - 1) * dilation_h + 1, height);
 117:     index_t wend = min(wstart + (kernel_w - 1) * dilation_w + 1, width);
 118:     while(hstart < 0)
 119:       hstart += dilation_h;
 120:     while(wstart < 0)
 121:       wstart += dilation_w;
 122:     scalar_t maxval = at::numeric_limits<scalar_t>::lower_bound(); // -Infinity
 123:     index_t maxidx = hstart * width + wstart;
 124:     const scalar_t* btm_data = bottom_data + (n * channels + c) * height * width;
 125:     for (int h = hstart; h < hend; h += dilation_h) {
 126:       for (int w = wstart; w < wend; w += dilation_w) {
 127:         scalar_t val = btm_data[h * width + w];
 128:         if ((val > maxval) || at::_isnan(val)) {
 129:           maxidx = h * width + w;
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 130-137
```cpp
 130:           maxval = val;
 131:         }
 132:       }
 133:     }
 134:     top_data[index] = maxval;
 135:     top_mask[index] = maxidx;
 136:   }
 137: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-160
```cpp
 139: template <typename scalar_t, typename index_t>
 140: C10_LAUNCH_BOUNDS_1(CUDA_MAX_THREADS)
 141: __global__ void max_pool_forward_nhwc(
 142:     const scalar_t* bottom_data,
 143:     const int nbatch,
 144:     const index_t channels, const index_t height, const index_t width,
 145:     const index_t pooled_height, const index_t pooled_width,
 146:     const int kernel_h, const int kernel_w, const int stride_h,
 147:     const int stride_w, const int pad_h, const int pad_w,
 148:     const int dilation_h, const int dilation_w,
 149:     const index_t in_stride_n, const index_t in_stride_c,
 150:     const index_t in_stride_h, const index_t in_stride_w,
 151:     const int kernel_stride_C, const int kernel_size_C,
 152:     scalar_t* top_data, int64_t* top_mask) {
 153: 
 154:   extern __shared__ unsigned char smem_raw[];
 155:   index_t *out_mask_cached = reinterpret_cast<index_t*>(smem_raw);
 156:   scalar_t *out_cached = reinterpret_cast<scalar_t*>(
 157:       out_mask_cached + kernel_size_C*blockDim.x*blockDim.y*blockDim.z);
 158: 
 159:   // flattening cta for pre-computation & smem initialization;
 160:   int thread_id = threadIdx.x + blockDim.x * (threadIdx.y + blockDim.y * threadIdx.z);
```
- EN: This block defines GPU kernel entry point(s) `max_pool_forward_nhwc`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `max_pool_forward_nhwc`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 161-161
```cpp
 161:   int block_size = blockDim.x * blockDim.y * blockDim.z;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 163-164
```cpp
 163:   // use shared memory to store temporary output value. This is simply to
 164:   // reduce register usage.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 165-168
```cpp
 165:   for (int i = thread_id; i < kernel_size_C*blockDim.x*blockDim.y*blockDim.z; i+= block_size) {
 166:     out_cached[i] = at::numeric_limits<scalar_t>::lower_bound();
 167:     out_mask_cached[i] = 0;
 168:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 170-170
```cpp
 170:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 172-174
```cpp
 172:   int batch_id = blockIdx.x % nbatch;
 173:   int channel_id = blockIdx.x / nbatch;
 174:   int channel_offset = threadIdx.x + channel_id * blockDim.x;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 176-178
```cpp
 176:   top_data = top_data + static_cast<index_t>(batch_id) * (pooled_height * pooled_width * channels);
 177:   top_mask = top_mask + static_cast<index_t>(batch_id) * (pooled_height * pooled_width * channels);
 178:   bottom_data = bottom_data + static_cast<index_t>(batch_id) * in_stride_n;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 180-181
```cpp
 180:   out_cached += (threadIdx.z * blockDim.y + threadIdx.y) * kernel_size_C*blockDim.x;
 181:   out_mask_cached  += (threadIdx.z * blockDim.y + threadIdx.y) * kernel_size_C*blockDim.x;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 183-188
```cpp
 183:   int oH = (static_cast<int>(pooled_height) + gridDim.z - 1) / gridDim.z;
 184:   int oW = (static_cast<int>(pooled_width)  + gridDim.y - 1) / gridDim.y;
 185:   int ostartH = threadIdx.z + blockIdx.z*oH;
 186:   int oendH = ::min(ostartH+oH, static_cast<int>(pooled_height));
 187:   int ostartW = threadIdx.y + blockIdx.y*oW;
 188:   int oendW = ::min(ostartW+oW, static_cast<int>(pooled_width));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 190-211
```cpp
 190:   for (int oh = ostartH; oh < oendH; oh+=blockDim.z) {
 191:     index_t hstart = static_cast<index_t>(oh) * stride_h - pad_h;
 192:     index_t hend = std::min(hstart + static_cast<index_t>((kernel_h - 1) * dilation_h + 1), height);
 193:     for (int ow = ostartW; ow < oendW; ow+=blockDim.y) {
 194:       index_t wstart = static_cast<index_t>(ow) * stride_w - pad_w;
 195:       index_t wend = std::min(wstart + static_cast<index_t>((kernel_w - 1) * dilation_w + 1), width);
 196:       while(hstart < 0)
 197:         hstart += dilation_h;
 198:       while(wstart < 0)
 199:         wstart += dilation_w;
 200: 
 201: #if defined (USE_ROCM)
 202: // Max h,w and c for using prefetch path
 203: #define MAXh 3
 204: #define MAXw 3
 205: #define MAXc 1
 206:       // Prefetch if conditions met...
 207:       if (kernel_h<=MAXh &&
 208:           kernel_w<=MAXw &&
 209:           channels<=MAXc*(blockDim.x*kernel_stride_C)) {
 210:         scalar_t val [MAXh][MAXw][MAXc] = {};
 211:         for (int ih = 0; ih < MAXh; ih++) {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `defined`.
- CN: 该代码块定义或继续实现 `defined`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 212-233
```cpp
 212:           int ih_ = ih*dilation_h+hstart;
 213:           for (int iw = 0; iw < MAXw; iw++) {
 214:             int iw_ = iw*dilation_w+wstart;
 215:             const scalar_t *ptr_input = bottom_data + ih_ * in_stride_h + iw_ * in_stride_w;
 216:             for(int c = 0; c < MAXc; c++) {
 217:               int c_ = c*blockDim.x*kernel_stride_C+channel_offset;
 218:               if (ih_>=hend || iw_>=wend || c_>=channels) continue;
 219:               val[ih][iw][c] = ptr_input[c_*in_stride_c];
 220:             }
 221:           }
 222:         }
 223:         for (int ih = 0; ih < MAXh; ih++) {
 224:           int ih_ = ih*dilation_h+hstart;
 225:           for (int iw = 0; iw < MAXw; iw++) {
 226:             int iw_ = iw*dilation_w+wstart;
 227:             int cached_index = threadIdx.x;
 228:             for(int c = 0; c < MAXc; c++) {
 229:               int c_ = c*blockDim.x*kernel_stride_C+channel_offset;
 230:               if (ih_>=hend || iw_>=wend || c_>=channels) continue;
 231:               if ((val[ih][iw][c] > out_cached[cached_index]) || at::_isnan(val[ih][iw][c])) {
 232:                 out_cached[cached_index] = val[ih][iw][c];
 233:                 out_mask_cached[cached_index] = ih_ * width + iw_;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 234-247
```cpp
 234:               }
 235:               cached_index += blockDim.x;
 236:             }
 237:           }
 238:         }
 239:       }
 240:       // Else do it Non-Prefetch...
 241:       else
 242: #endif
 243:       for (index_t ih = hstart; ih < hend; ih += dilation_h) {
 244:         for (index_t iw = wstart; iw < wend; iw += dilation_w) {
 245:           int cached_index = threadIdx.x;
 246:           const scalar_t *ptr_input = bottom_data + ih * in_stride_h + iw * in_stride_w;
 247:           for (index_t c = channel_offset; c < channels; c += static_cast<index_t>(blockDim.x) * kernel_stride_C) {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 248-256
```cpp
 248:             scalar_t val = ptr_input[c * in_stride_c];
 249:             if ((val > out_cached[cached_index]) || at::_isnan(val)) {
 250:               out_cached[cached_index] = val;
 251:               out_mask_cached[cached_index] = ih * width + iw;
 252:             }
 253:             cached_index += blockDim.x;
 254:           }
 255:         }
 256:       }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 258-259
```cpp
 258:       scalar_t *ptr_output_data = top_data + (static_cast<index_t>(oh) * pooled_width + ow) * channels;
 259:       int64_t *ptr_output_mask = top_mask + (static_cast<index_t>(oh) * pooled_width + ow) * channels;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 261-271
```cpp
 261:       int cached_index = threadIdx.x;
 262:       for (index_t c = channel_offset; c < channels; c += static_cast<index_t>(blockDim.x) * kernel_stride_C) {
 263:         ptr_output_data[c] = out_cached[cached_index];
 264:         ptr_output_mask[c] = static_cast<int64_t>(out_mask_cached[cached_index]);
 265:         out_cached[cached_index] = at::numeric_limits<scalar_t>::lower_bound();
 266:         out_mask_cached[cached_index] = index_t(0);
 267:         cached_index += blockDim.x;
 268:       }
 269:     }
 270:   }
 271: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 274-274
```cpp
 274: static constexpr int BLOCK_THREADS = 256;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 276-289
```cpp
 276: template <typename scalar_t, typename accscalar_t, typename index_t>
 277: #if defined (USE_ROCM)
 278: C10_LAUNCH_BOUNDS_2(BLOCK_THREADS, 4)
 279: #else
 280: C10_LAUNCH_BOUNDS_2(BLOCK_THREADS, 8)
 281: #endif
 282: __global__ void max_pool_backward_nchw(
 283:     const scalar_t* top_diff,
 284:     const int64_t* top_mask,
 285:     const index_t num,
 286:     const index_t channels,
 287:     const index_t height,
 288:     const index_t width,
 289:     const index_t pooled_height,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines GPU kernel entry point(s) `max_pool_backward_nchw`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `max_pool_backward_nchw`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 290-311
```cpp
 290:     const index_t pooled_width,
 291:     const int kernel_h, const int kernel_w,
 292:     const int stride_h, const int stride_w,
 293:     const int pad_h, const int pad_w,
 294:     const int dilation_h, const int dilation_w,
 295:     scalar_t* bottom_diff) {
 296:   CUDA_KERNEL_LOOP_TYPE(index, height*width, index_t) {
 297:     index_t h = index / width;
 298:     index_t w = index - h * width;
 299:     index_t phstart = p_start(h, pad_h, kernel_h, dilation_h, stride_h);
 300:     index_t phend = p_end(h, pad_h, pooled_height, stride_h);
 301:     index_t pwstart = p_start(w, pad_w, kernel_w, dilation_w, stride_w);
 302:     index_t pwend = p_end(w, pad_w, pooled_width, stride_w);
 303:     for (index_t n = blockIdx.y; n < num; n += gridDim.y) {
 304:       for (index_t c = blockIdx.z; c < channels; c += gridDim.z) {
 305:         accscalar_t gradient = accscalar_t(0);
 306:         index_t offset = (n * channels + c) * pooled_height * pooled_width;
 307:         for (index_t ph = phstart; ph < phend; ++ph) {
 308:           for (index_t pw = pwstart; pw < pwend; ++pw) {
 309:             if (top_mask[ph * pooled_width + pw + offset] == h * width + w) {
 310:               gradient += static_cast<accscalar_t>(top_diff[ph * pooled_width + pw + offset]);
 311:             }
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 312-318
```cpp
 312:           }
 313:         }
 314:         bottom_diff[(n*channels+c)*height*width+index] = static_cast<scalar_t>(gradient);
 315:       }
 316:     }
 317:   }
 318: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 320-341
```cpp
 320: template <typename scalar_t, typename accscalar_t>
 321: C10_LAUNCH_BOUNDS_1(CUDA_MAX_THREADS)
 322: __global__ void max_pool_backward_nhwc(const scalar_t* top_diff,
 323:                                     const int64_t* top_mask, const int nbatch, const int64_t channels,
 324:                                     const int64_t height, const int64_t width, const int pooled_height,
 325:                                     const int pooled_width, const int kernel_h, const int kernel_w,
 326:                                     const int stride_h, const int stride_w, const int pad_h, const int pad_w,
 327:                                     const int dilation_h, const int dilation_w,
 328:                                     const int out_stride_c, const int out_stride_h, const int out_stride_w,
 329:                                     const int kernel_stride_C, const int kernel_size_C,
 330:                                     scalar_t* bottom_diff) {
 331:   extern __shared__ int smem[];
 332:   accscalar_t *out_cached = reinterpret_cast<accscalar_t*>(smem);
 333: 
 334:   int thread_id = threadIdx.x + blockDim.x * (threadIdx.y + blockDim.y * threadIdx.z);
 335:   int block_size = blockDim.x * blockDim.y * blockDim.z;
 336: 
 337:   int batch_id = blockIdx.x % nbatch;
 338:   int channel_id = blockIdx.x / nbatch;
 339:   int channel_offset = threadIdx.x + channel_id * blockDim.x;
 340: 
 341:   for (int i = thread_id; i < kernel_size_C*blockDim.x*blockDim.y*blockDim.z; i+= block_size) {
```
- EN: This block defines GPU kernel entry point(s) `max_pool_backward_nhwc`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `max_pool_backward_nhwc`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 342-343
```cpp
 342:     out_cached[i] = accscalar_t(0.0);
 343:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 345-345
```cpp
 345:   __syncthreads();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 347-347
```cpp
 347:   out_cached = &out_cached[(threadIdx.z * blockDim.y + threadIdx.y) * kernel_size_C*blockDim.x];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 349-351
```cpp
 349:   bottom_diff = bottom_diff + batch_id * height * width * channels;
 350:   top_mask = top_mask + batch_id * pooled_height * pooled_width * channels;
 351:   top_diff = top_diff + batch_id * pooled_height * pooled_width * channels;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 353-358
```cpp
 353:   int iH = (height + gridDim.z-1) / gridDim.z;
 354:   int iW = (width + gridDim.y-1) / gridDim.y;
 355:   int istartH = threadIdx.z + blockIdx.z*iH;
 356:   int iendH = ::min(static_cast<int64_t>(istartH)+iH, height);
 357:   int istartW = threadIdx.y + blockIdx.y*iW;
 358:   int iendW = ::min(static_cast<int64_t>(istartW)+iW, width);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 360-381
```cpp
 360:   for (int ih = istartH; ih < iendH; ih+=blockDim.z) {
 361:     int phstart = p_start(ih, pad_h, kernel_h, dilation_h, stride_h);
 362:     int phend = p_end(ih, pad_h, pooled_height, stride_h);
 363:     for (int iw = istartW; iw < iendW; iw+=blockDim.y) {
 364:       int pwstart = p_start(iw, pad_w, kernel_w, dilation_w, stride_w);
 365:       int pwend = p_end(iw, pad_w, pooled_width, stride_w);
 366:       int index_shift = ih * width + iw;
 367:       if ((phstart + 1 != phend) || (pwstart + 1 != pwend)) {
 368: 
 369: #if defined (USE_ROCM)
 370: #define _MAXh 2
 371: #define _MAXw 2
 372:         if (phend-phstart<=_MAXh && pwend-pwstart<=_MAXw) {
 373:           int msk[_MAXh][_MAXw];
 374:           scalar_t tpd[_MAXh][_MAXw];
 375:           int cached_index = threadIdx.x;
 376: #pragma unroll
 377:           for (int c = channel_offset; c < channels; c += blockDim.x*kernel_stride_C) {
 378: #pragma unroll
 379:             for(int oh = 0; oh < _MAXh; ++oh) {
 380: #pragma unroll
 381:               for(int ow = 0; ow < _MAXw; ++ow) {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `defined`.
- CN: 该代码块定义或继续实现 `defined`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 382-392
```cpp
 382:                 int oh_ = oh+phstart;
 383:                 int ow_ = ow+pwstart;
 384:                 const int64_t* ptr_top_mask = top_mask + oh_ * out_stride_h + ow_ * out_stride_w;
 385:                 if (oh_ >= phend || ow_ >= pwend) {
 386:                   msk[oh][ow] = ~index_shift;
 387:                 } else {
 388:                   msk[oh][ow] = ptr_top_mask[c*out_stride_c];
 389:                   tpd[oh][ow] = top_diff[oh_ * out_stride_h + ow_ * out_stride_w + c*out_stride_c];
 390:                 }
 391:               }
 392:             }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 394-407
```cpp
 394:             accscalar_t acm = 0;
 395: #pragma unroll
 396:             for(int oh = 0; oh < _MAXh; ++oh) {
 397: #pragma unroll
 398:               for(int ow = 0; ow < _MAXw; ++ow) {
 399:                 if (msk[oh][ow] == index_shift) {
 400:                   acm += static_cast<accscalar_t>(tpd[oh][ow]);
 401:                 }
 402:               }
 403:             }
 404:             out_cached[cached_index] += acm;
 405:             cached_index += blockDim.x;
 406:           }
 407:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 408-411
```cpp
 408:         else
 409: #undef _MAXh
 410: #undef _MAXw
 411: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 413-426
```cpp
 413:         for(int oh = phstart; oh < phend; ++oh) {
 414:           for(int ow = pwstart; ow < pwend; ++ow) {
 415:             int cached_index = threadIdx.x;
 416:             const int64_t* ptr_top_mask = top_mask + oh * out_stride_h + ow * out_stride_w;
 417:             for (int c = channel_offset; c < channels; c += blockDim.x*kernel_stride_C) {
 418:               if (ptr_top_mask[c*out_stride_c] == index_shift) {
 419:                 out_cached[cached_index] +=
 420:                   static_cast<accscalar_t>(top_diff[oh * out_stride_h + ow * out_stride_w + c*out_stride_c]);
 421:               }
 422:               cached_index += blockDim.x;
 423:             }
 424:           }
 425:         }
 426:         scalar_t *ptr_bottom_diff = bottom_diff + index_shift * channels;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 427-443
```cpp
 427:         int cached_index = threadIdx.x;
 428:         for (int c = channel_offset; c < channels; c += blockDim.x*kernel_stride_C) {
 429:           ptr_bottom_diff[c] = static_cast<scalar_t>(out_cached[cached_index]);
 430:           out_cached[cached_index] = accscalar_t(0.0);
 431:           cached_index += blockDim.x;
 432:         }
 433:       } else {
 434:         const int64_t* ptr_top_mask = top_mask + phstart * out_stride_h + pwstart * out_stride_w;
 435:         scalar_t *ptr_bottom_diff = bottom_diff + index_shift * channels;
 436:         int cached_index = threadIdx.x;
 437:         for (int c = channel_offset; c < channels; c += blockDim.x*kernel_stride_C) {
 438:           if (ptr_top_mask[c*out_stride_c] == index_shift) {
 439:             ptr_bottom_diff[c] =
 440:               static_cast<scalar_t>(top_diff[phstart * out_stride_h + pwstart * out_stride_w + c*out_stride_c]);
 441:           }
 442:           cached_index += blockDim.x;
 443:         }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 444-447
```cpp
 444:       }
 445:     }
 446:   }
 447: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 449-449
```cpp
 449: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 451-472
```cpp
 451: TORCH_IMPL_FUNC(max_pool2d_with_indices_out_cuda)
 452: (const Tensor& input_,
 453: IntArrayRef kernel_size,
 454: IntArrayRef stride,
 455: IntArrayRef padding,
 456: IntArrayRef dilation,
 457: bool ceil_mode,
 458: const Tensor& output,
 459: const Tensor& indices) {
 460:   NoNamesGuard guard;
 461: 
 462:   TensorArg output_arg{ output, "output", 1 };
 463:   TensorArg indices_arg{ indices, "indices", 2 };
 464:   TensorArg input_arg{ input_, "input_", 3 };
 465: 
 466:   checkAllSameGPU(__func__, {output_arg, indices_arg, input_arg});
 467:   if (output.numel() == 0) {
 468:     return;
 469:   }
 470: 
 471:   const int kH = safe_downcast<int, int64_t>(kernel_size[0]);
 472:   const int kW = kernel_size.size() == 1 ? kH : safe_downcast<int, int64_t>(kernel_size[1]);
```
- EN: This block defines or continues the implementation of `max_pool2d_with_indices_out_cuda`.
- CN: 该代码块定义或继续实现 `max_pool2d_with_indices_out_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 474-476
```cpp
 474:   const int dH = stride.empty() ? kH : safe_downcast<int, int64_t>(stride[0]);
 475:   const int dW = stride.empty() ? kW :
 476:                  stride.size() == 1 ? dH : safe_downcast<int, int64_t>(stride[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 478-479
```cpp
 478:   const int padH = safe_downcast<int, int64_t>(padding[0]);
 479:   const int padW = padding.size() == 1 ? padH : safe_downcast<int, int64_t>(padding[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 481-482
```cpp
 481:   const int dilationH = safe_downcast<int, int64_t>(dilation[0]);
 482:   const int dilationW = dilation.size() == 1 ? dilationH : safe_downcast<int, int64_t>(dilation[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 484-484
```cpp
 484:   const auto memory_format = input_.suggest_memory_format();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 486-489
```cpp
 486:   const int64_t nbatch = input_.ndimension() == 4 ? input_.size(-4) : 1;
 487:   const int64_t nInputPlane = input_.size(-3);
 488:   const int64_t inputHeight = input_.size(-2);
 489:   const int64_t inputWidth = input_.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 491-492
```cpp
 491:   const int64_t outputHeight = output.size(-2);
 492:   const int64_t outputWidth = output.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 494-494
```cpp
 494:   Tensor input = input_.contiguous(memory_format);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 496-499
```cpp
 496:   const int64_t in_stride_n = input_.ndimension() == 4 ? input.stride(-4) : 0;
 497:   const int64_t in_stride_c = input.stride(-3);
 498:   const int64_t in_stride_h = input.stride(-2);
 499:   const int64_t in_stride_w = input.stride(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 501-522
```cpp
 501:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 502:     "max_pool2d_with_indices_out_cuda_frame",
 503:     [&] {
 504:       using accscalar_t = acc_type<scalar_t, true>;
 505: 
 506:       scalar_t *output_data = output.mutable_data_ptr<scalar_t>();
 507:       const scalar_t *input_data = input.const_data_ptr<scalar_t>();
 508:       int64_t *indices_data = indices.mutable_data_ptr<int64_t>();
 509: 
 510:       switch (memory_format) {
 511:         case MemoryFormat::ChannelsLast: {
 512:           const int max_threads = std::min<int>(
 513:               at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, CUDA_MAX_THREADS);
 514:           int* maxThreadsDim = at::cuda::getCurrentDeviceProperties()->maxThreadsDim;
 515:           int block_x = std::min<int>(
 516:               maxThreadsDim[0], std::min<int>(lastPow2(nInputPlane), at::cuda::warp_size()));
 517:           int block_y = std::min<int>(
 518:               maxThreadsDim[1], std::min<int>(lastPow2(outputWidth), max_threads / block_x));
 519:           int block_z = std::min<int>(
 520:               maxThreadsDim[2], std::min<int>(lastPow2(outputHeight), max_threads / block_x / block_y));
 521:           block_x = std::min<int>(
 522:               maxThreadsDim[0], std::min<int>(lastPow2(nInputPlane), max_threads / block_y / block_z));
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 523-523
```cpp
 523:           const dim3 block(block_x, block_y, block_z);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 525-528
```cpp
 525:           bool use_int32 = can_use_int32_nhwc(
 526:               nbatch, nInputPlane, inputHeight, inputWidth,
 527:               outputHeight, outputWidth,
 528:               in_stride_n, in_stride_c, in_stride_h, in_stride_w);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 530-533
```cpp
 530:           int kernel_stride_C = ceil_div(
 531:               safe_downcast<int, int64_t>(nInputPlane), block_x * 4);
 532:           int kernel_size_C = ceil_div(
 533:               safe_downcast<int, int64_t>(nInputPlane), block_x * kernel_stride_C);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 535-542
```cpp
 535:           int grid_x = nbatch*kernel_stride_C;
 536:           int grid_y = std::min<int>(
 537:               at::cuda::getCurrentDeviceProperties()->maxGridSize[1],
 538:               ceil_div(safe_downcast<int, int64_t>(outputWidth), block_y*BLOCK_STRIDE_FWD));
 539:           int grid_z = std::min<int>(
 540:               at::cuda::getCurrentDeviceProperties()->maxGridSize[2],
 541:               ceil_div(safe_downcast<int, int64_t>(outputHeight), block_z*BLOCK_STRIDE_FWD));
 542:           const dim3 grid(grid_x, grid_y, grid_z);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 544-545
```cpp
 544:           size_t shmem_size;
 545:           size_t mask_elems = static_cast<size_t>(kernel_size_C) * block_x * block_y * block_z;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 547-568
```cpp
 547:           if (use_int32) {
 548:             shmem_size = mask_elems * (sizeof(int32_t) + sizeof(scalar_t));
 549:             TORCH_CHECK(shmem_size <= at::cuda::getCurrentDeviceProperties()->sharedMemPerBlock,
 550:                         "shared memory too small");
 551:             max_pool_forward_nhwc<scalar_t, int32_t>
 552:               <<<grid, block, shmem_size, at::cuda::getCurrentCUDAStream()>>>(
 553:                 input_data, static_cast<int>(nbatch),
 554:                 static_cast<int32_t>(nInputPlane),
 555:                 static_cast<int32_t>(inputHeight),
 556:                 static_cast<int32_t>(inputWidth),
 557:                 static_cast<int32_t>(outputHeight),
 558:                 static_cast<int32_t>(outputWidth),
 559:                 kH, kW, dH, dW, padH, padW, dilationH, dilationW,
 560:                 static_cast<int32_t>(in_stride_n),
 561:                 static_cast<int32_t>(in_stride_c),
 562:                 static_cast<int32_t>(in_stride_h),
 563:                 static_cast<int32_t>(in_stride_w),
 564:                 kernel_stride_C, kernel_size_C,
 565:                 output_data, indices_data);
 566:           } else {
 567:             shmem_size = mask_elems * (sizeof(int64_t) + sizeof(scalar_t));
 568:             TORCH_CHECK(shmem_size <= at::cuda::getCurrentDeviceProperties()->sharedMemPerBlock,
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 569-582
```cpp
 569:                         "shared memory too small");
 570:             max_pool_forward_nhwc<scalar_t, int64_t>
 571:               <<<grid, block, shmem_size, at::cuda::getCurrentCUDAStream()>>>(
 572:                 input_data, static_cast<int>(nbatch),
 573:                 nInputPlane, inputHeight, inputWidth, outputHeight, outputWidth,
 574:                 kH, kW, dH, dW, padH, padW, dilationH, dilationW,
 575:                 in_stride_n, in_stride_c, in_stride_h, in_stride_w,
 576:                 kernel_stride_C, kernel_size_C,
 577:                 output_data, indices_data);
 578:           }
 579:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 580:           break;
 581:         }
 582:         case MemoryFormat::Contiguous: {
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 583-604
```cpp
 583:           const int threads = std::min(
 584:               at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock,
 585:               BLOCK_THREADS);
 586:           const int64_t nthreads = output.numel();
 587:           bool use_int32 = can_use_int32_nchw(
 588:               nbatch, nInputPlane, inputHeight, inputWidth, outputHeight, outputWidth);
 589:           const int maxGridX = at::cuda::getCurrentDeviceProperties()->maxGridSize[0];
 590:           const int blocks = static_cast<int>(std::min<int64_t>(
 591:               ceil_div(nthreads, static_cast<int64_t>(threads)),
 592:               static_cast<int64_t>(maxGridX)));
 593:           auto stream = at::cuda::getCurrentCUDAStream();
 594:           if (use_int32) {
 595:             max_pool_forward_nchw<scalar_t, int32_t>
 596:                 <<<blocks, threads, 0, stream>>>(
 597:                     static_cast<int32_t>(nthreads),
 598:                     input_data,
 599:                     static_cast<int32_t>(nInputPlane),
 600:                     static_cast<int32_t>(inputHeight),
 601:                     static_cast<int32_t>(inputWidth),
 602:                     static_cast<int32_t>(outputHeight),
 603:                     static_cast<int32_t>(outputWidth),
 604:                     kH, kW, dH, dW, padH, padW, dilationH, dilationW,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 605-618
```cpp
 605:                     output_data, indices_data);
 606:           } else {
 607:             max_pool_forward_nchw<scalar_t, int64_t>
 608:                 <<<blocks, threads, 0, stream>>>(
 609:                     nthreads,
 610:                     input_data,
 611:                     nInputPlane,
 612:                     inputHeight,
 613:                     inputWidth,
 614:                     outputHeight,
 615:                     outputWidth,
 616:                     kH, kW, dH, dW, padH, padW, dilationH, dilationW,
 617:                     output_data, indices_data);
 618:           }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 619-626
```cpp
 619:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 620:           break;
 621:         }
 622:         default: TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
 623:       }
 624:     }
 625:   );
 626: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 628-649
```cpp
 628: TORCH_IMPL_FUNC(max_pool2d_with_indices_backward_out_cuda)
 629: (const Tensor& gradOutput_,
 630: const Tensor& input_,
 631: IntArrayRef kernel_size,
 632: IntArrayRef stride,
 633: IntArrayRef padding,
 634: IntArrayRef dilation,
 635: bool ceil_mode,
 636: const Tensor& indices_,
 637: const Tensor& gradInput) {
 638:   NoNamesGuard guard;
 639: 
 640:   TensorArg gradInput_arg{ gradInput, "gradInput", 1 };
 641:   TensorArg gradOutput_arg{ gradOutput_, "gradOutput_", 2 };
 642:   TensorArg input_arg{ input_, "input_", 3 };
 643:   TensorArg indices_arg{ indices_, "indices", 4 };
 644: 
 645:   checkAllSameGPU(__func__,
 646:                   {gradInput_arg, gradOutput_arg, input_arg, indices_arg});
 647:   if (gradOutput_.numel() == 0) {
 648:     return;
 649:   }
```
- EN: This block defines or continues the implementation of `max_pool2d_with_indices_backward_out_cuda`.
- CN: 该代码块定义或继续实现 `max_pool2d_with_indices_backward_out_cuda`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 651-652
```cpp
 651:   const int kH = safe_downcast<int, int64_t>(kernel_size[0]);
 652:   const int kW = kernel_size.size() == 1 ? kH : safe_downcast<int, int64_t>(kernel_size[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 654-656
```cpp
 654:   const int dH = stride.empty() ? kH : safe_downcast<int, int64_t>(stride[0]);
 655:   const int dW = stride.empty() ? kW :
 656:                  stride.size() == 1 ? dH : safe_downcast<int, int64_t>(stride[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 658-659
```cpp
 658:   const int padH = safe_downcast<int, int64_t>(padding[0]);
 659:   const int padW = padding.size() == 1 ? padH : safe_downcast<int, int64_t>(padding[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 661-662
```cpp
 661:   const int dilationH = safe_downcast<int, int64_t>(dilation[0]);
 662:   const int dilationW = dilation.size() == 1 ? dilationH : safe_downcast<int, int64_t>(dilation[1]);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 664-664
```cpp
 664:   const auto memory_format = input_.suggest_memory_format();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 666-666
```cpp
 666:   const Tensor input = input_.contiguous(memory_format);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 668-671
```cpp
 668:   const int64_t nbatch = input.ndimension() == 4 ? input.size(-4) : 1;
 669:   const int64_t nInputPlane = input.size(-3);
 670:   const int64_t inputHeight = input.size(-2);
 671:   const int64_t inputWidth = input.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 673-676
```cpp
 673:   const int64_t in_stride_n = input.ndimension() == 4 ? input.stride(-4) : 0;
 674:   const int64_t in_stride_c = input.stride(-3);
 675:   const int64_t in_stride_h = input.stride(-2);
 676:   const int64_t in_stride_w = input.stride(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 678-678
```cpp
 678:   const Tensor gradOutput = gradOutput_.contiguous(memory_format);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 680-681
```cpp
 680:   const int64_t outputHeight = gradOutput.size(-2);
 681:   const int64_t outputWidth = gradOutput.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 683-685
```cpp
 683:   const int64_t out_stride_c = gradOutput.stride(-3);
 684:   const int64_t out_stride_h = gradOutput.stride(-2);
 685:   const int64_t out_stride_w = gradOutput.stride(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 687-687
```cpp
 687:   const Tensor indices = indices_.contiguous(memory_format);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 689-689
```cpp
 689:   gradInput.zero_();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 691-712
```cpp
 691:   AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(),
 692:     "max_pool2d_with_indices_out_cuda_frame",
 693:     [&] {
 694:       using accscalar_t = acc_type<scalar_t, true>;
 695: 
 696:       const scalar_t *gradOutput_data = gradOutput.const_data_ptr<scalar_t>();
 697:       scalar_t *gradInput_data = gradInput.mutable_data_ptr<scalar_t>();
 698:       const int64_t *indices_data = indices.const_data_ptr<int64_t>();
 699: 
 700:       switch (memory_format) {
 701:         case MemoryFormat::ChannelsLast: {
 702:           const int max_threads = std::min<int>(at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock, CUDA_MAX_THREADS);
 703:           int* maxThreadsDim = at::cuda::getCurrentDeviceProperties()->maxThreadsDim;
 704:           int block_x = std::min<int>(
 705:               maxThreadsDim[0], std::min<int>(lastPow2(nInputPlane), at::cuda::warp_size()));
 706:           int block_y = std::min<int>(
 707:               maxThreadsDim[1], std::min<int>(lastPow2(inputWidth), max_threads / block_x));
 708:           int block_z = std::min<int>(
 709:               maxThreadsDim[2], std::min<int>(lastPow2(inputHeight), max_threads / block_x / block_y));
 710:           block_x = std::min<int>(
 711:               maxThreadsDim[0], std::min<int>(lastPow2(nInputPlane), max_threads / block_y / block_z));
 712:           const dim3 block(block_x, block_y, block_z);
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 714-717
```cpp
 714:           int kernel_stride_C = ceil_div(
 715:               safe_downcast<int, int64_t>(nInputPlane), block_x * 4);
 716:           int kernel_size_C = ceil_div(
 717:               safe_downcast<int, int64_t>(nInputPlane), block_x * kernel_stride_C);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 719-726
```cpp
 719:           int grid_x = nbatch*kernel_stride_C;
 720:           int grid_y = std::min<int>(
 721:               at::cuda::getCurrentDeviceProperties()->maxGridSize[1],
 722:               ceil_div(safe_downcast<int, int64_t>(inputWidth), block_y*BLOCK_STRIDE_BWD));
 723:           int grid_z = std::min<int>(
 724:               at::cuda::getCurrentDeviceProperties()->maxGridSize[2],
 725:               ceil_div(safe_downcast<int, int64_t>(inputHeight), block_z*BLOCK_STRIDE_BWD));
 726:           const dim3 grid(grid_x, grid_y, grid_z);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 728-729
```cpp
 728:           size_t shmem_size = (kernel_size_C * block_x*block_y*block_z) * sizeof(accscalar_t);
 729:           AT_ASSERT(shmem_size <= at::cuda::getCurrentDeviceProperties()->sharedMemPerBlock);
```
- EN: Shared-memory usage hints that the kernel stages data cooperatively inside a thread block.
- CN: 共享内存的使用表明该内核会在线程块内部协作暂存数据。

### Lines 731-733
```cpp
 731:           // The backward kernel is launched on input instead output.
 732:           // If it is launched on output layer, atomic_add would not provide much benefit on FP16.
 733:           // Please check comments at https://github.com/pytorch/pytorch/pull/34519.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 734-747
```cpp
 734:           max_pool_backward_nhwc<scalar_t, accscalar_t>
 735:           <<<grid, block, shmem_size, at::cuda::getCurrentCUDAStream()>>>(
 736:                   gradOutput_data,
 737:                   indices_data,
 738:                   nbatch,
 739:                   nInputPlane, inputHeight, inputWidth, outputHeight, outputWidth,
 740:                   kH, kW, dH, dW, padH, padW, dilationH, dilationW,
 741:                   out_stride_c, out_stride_h, out_stride_w,
 742:                   kernel_stride_C, kernel_size_C,
 743:                   gradInput_data);
 744:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 745:           break;
 746:         }
 747:         case MemoryFormat::Contiguous: {
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 748-769
```cpp
 748:           const int threads = std::min(
 749:               at::cuda::getCurrentDeviceProperties()->maxThreadsPerBlock,
 750:               BLOCK_THREADS);
 751:           const int imgcount = inputWidth * inputHeight;
 752:           const int maxGridX = at::cuda::getCurrentDeviceProperties()->maxGridSize[0];
 753:           const int maxGridY = at::cuda::getCurrentDeviceProperties()->maxGridSize[1];
 754:           const int maxGridZ = at::cuda::getCurrentDeviceProperties()->maxGridSize[2];
 755:           const int blocks_x = std::min(ceil_div(imgcount, threads), maxGridX);
 756:           dim3 grid(blocks_x, static_cast<unsigned>(std::min<int64_t>(nbatch, maxGridY)), static_cast<unsigned>(std::min<int64_t>(nInputPlane, maxGridZ)));
 757:           bool use_int32 = can_use_int32_nchw(
 758:               nbatch, nInputPlane, inputHeight, inputWidth, outputHeight, outputWidth);
 759:           auto stream = at::cuda::getCurrentCUDAStream();
 760:           if (use_int32) {
 761:             max_pool_backward_nchw<scalar_t, accscalar_t, int32_t>
 762:                 <<<grid, threads, 0, stream>>>(
 763:                     gradOutput_data,
 764:                     indices_data,
 765:                     static_cast<int32_t>(nbatch),
 766:                     static_cast<int32_t>(nInputPlane),
 767:                     static_cast<int32_t>(inputHeight),
 768:                     static_cast<int32_t>(inputWidth),
 769:                     static_cast<int32_t>(outputHeight),
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 770-783
```cpp
 770:                     static_cast<int32_t>(outputWidth),
 771:                     kH, kW, dH, dW, padH, padW, dilationH, dilationW,
 772:                     gradInput_data);
 773:           } else {
 774:             max_pool_backward_nchw<scalar_t, accscalar_t, int64_t>
 775:                 <<<grid, threads, 0, stream>>>(
 776:                     gradOutput_data,
 777:                     indices_data,
 778:                     nbatch,
 779:                     nInputPlane,
 780:                     inputHeight,
 781:                     inputWidth,
 782:                     outputHeight,
 783:                     outputWidth,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 784-794
```cpp
 784:                     kH, kW, dH, dW, padH, padW, dilationH, dilationW,
 785:                     gradInput_data);
 786:           }
 787:           C10_CUDA_KERNEL_LAUNCH_CHECK();
 788:           break;
 789:         }
 790:         default: TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
 791:       }
 792:     }
 793:   );
 794: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 796-796
```cpp
 796: } // at::native
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

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
  - `<ATen/NamedTensorUtils.h>`
  - `<ATen/NumericUtils.h>`
  - `<ATen/native/Pool.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/NumericLimits.cuh>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/detail/KernelUtils.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `TORCH_IMPL_FUNC`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::warp_size`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

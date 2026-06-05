# GridSampler.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/GridSampler.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `launch_grid_sampler_2d_forward_kernel`, `launch_grid_sampler_3d_forward_kernel`, `launch_grid_sampler_2d_backward_kernel`, `launch_grid_sampler_3d_backward_kernel`.
- 用途（中文）: 实现与 `launch_grid_sampler_2d_forward_kernel`, `launch_grid_sampler_3d_forward_kernel`, `launch_grid_sampler_2d_backward_kernel`, `launch_grid_sampler_3d_backward_kernel` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/OpMathType.h>
   3: #include <ATen/native/cuda/GridSampler.h>
   4: #include <ATen/native/GridSamplerUtils.h>
   5: #include <ATen/native/cuda/GridSampler.cuh>
   6: #include <ATen/native/cuda/UpSample.cuh>
   7: #include <ATen/cuda/CUDAContext.h>
   8: #include <ATen/cuda/detail/TensorInfo.cuh>
   9: #include <ATen/cuda/detail/IndexUtils.cuh>
  10: #include <ATen/cuda/detail/KernelUtils.h>
  11: #include <ATen/core/TensorBase.h>
  12: #include <ATen/Dispatch.h>
  13: #include <c10/macros/Macros.h>
  14: #include <cmath>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/OpMathType.h>`, `<ATen/native/cuda/GridSampler.h>`, `<ATen/native/GridSamplerUtils.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/OpMathType.h>`, `<ATen/native/cuda/GridSampler.h>`, `<ATen/native/GridSamplerUtils.h>`。

### Lines 16-37
```cpp
  16: namespace at::native {
  17: 
  18: using namespace at::cuda::detail;
  19: 
  20: using at::native::detail::GridSamplerInterpolation;
  21: using at::native::detail::GridSamplerPadding;
  22: 
  23: namespace {
  24:   template <typename scalar_t, typename index_t>
  25:   C10_LAUNCH_BOUNDS_1(256)
  26:   __global__ void grid_sampler_2d_kernel(
  27:       const index_t nthreads,
  28:       TensorInfo<const scalar_t, index_t> input,
  29:       TensorInfo<const scalar_t, index_t> grid,
  30:       TensorInfo<scalar_t, index_t> output,
  31:       const GridSamplerInterpolation interpolation_mode,
  32:       const GridSamplerPadding padding_mode,
  33:       bool align_corners) {
  34: 
  35:     using opmath_t = at::opmath_type<scalar_t>;
  36:     index_t C = input.sizes[1];
  37:     index_t inp_H = input.sizes[2];
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `grid_sampler_2d_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `grid_sampler_2d_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 38-51
```cpp
  38:     index_t inp_W = input.sizes[3];
  39:     index_t out_H = grid.sizes[1];
  40:     index_t out_W = grid.sizes[2];
  41:     index_t inp_sN = input.strides[0];
  42:     index_t inp_sC = input.strides[1];
  43:     index_t inp_sH = input.strides[2];
  44:     index_t inp_sW = input.strides[3];
  45:     index_t grid_sN = grid.strides[0];
  46:     index_t grid_sH = grid.strides[1];
  47:     index_t grid_sW = grid.strides[2];
  48:     index_t grid_sCoor = grid.strides[3];
  49:     index_t out_sN = output.strides[0];
  50:     index_t out_sC = output.strides[1];
  51:     index_t out_sH = output.strides[2];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 52-52
```cpp
  52:     index_t out_sW = output.strides[3];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 54-75
```cpp
  54:     CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
  55:       const index_t w = index % out_W;
  56:       const index_t h = (index / out_W) % out_H;
  57:       const index_t n = index / (out_H * out_W);
  58:       const index_t grid_offset = n * grid_sN + h * grid_sH + w * grid_sW;
  59: 
  60:       // get the corresponding input x, y coordinates from grid
  61:       opmath_t x = grid.data[grid_offset];
  62:       opmath_t y = grid.data[grid_offset + grid_sCoor];
  63: 
  64:       opmath_t ix = grid_sampler_compute_source_index(x, inp_W, padding_mode, align_corners);
  65:       opmath_t iy = grid_sampler_compute_source_index(y, inp_H, padding_mode, align_corners);
  66: 
  67:       if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
  68:         // get NE, NW, SE, SW pixel values from (x, y)
  69:         index_t ix_nw = static_cast<index_t>(::floor(ix));
  70:         index_t iy_nw = static_cast<index_t>(::floor(iy));
  71:         index_t ix_ne = ix_nw + 1;
  72:         index_t iy_ne = iy_nw;
  73:         index_t ix_sw = ix_nw;
  74:         index_t iy_sw = iy_nw + 1;
  75:         index_t ix_se = ix_nw + 1;
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 76-76
```cpp
  76:         index_t iy_se = iy_nw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 78-78
```cpp
  78:         // get surfaces to each neighbor:
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 79-82
```cpp
  79:         opmath_t nw = (ix_se - ix)    * (iy_se - iy);
  80:         opmath_t ne = (ix    - ix_sw) * (iy_sw - iy);
  81:         opmath_t sw = (ix_ne - ix)    * (iy    - iy_ne);
  82:         opmath_t se = (ix    - ix_nw) * (iy    - iy_nw);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 84-84
```cpp
  84:         // calculate bilinear weighted pixel value and set output pixel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 85-102
```cpp
  85:         auto inp_ptr_NC = input.data + n * inp_sN;
  86:         auto out_ptr_NCHW = output.data + n * out_sN + h * out_sH + w * out_sW;
  87:         for (index_t c = 0; c < C; ++c, inp_ptr_NC += inp_sC, out_ptr_NCHW += out_sC) {
  88:           opmath_t out_acc = 0;
  89:           if (within_bounds_2d(iy_nw, ix_nw, inp_H, inp_W)) {
  90:             out_acc += inp_ptr_NC[iy_nw * inp_sH + ix_nw * inp_sW] * nw;
  91:           }
  92:           if (within_bounds_2d(iy_ne, ix_ne, inp_H, inp_W)) {
  93:             out_acc += inp_ptr_NC[iy_ne * inp_sH + ix_ne * inp_sW] * ne;
  94:           }
  95:           if (within_bounds_2d(iy_sw, ix_sw, inp_H, inp_W)) {
  96:             out_acc += inp_ptr_NC[iy_sw * inp_sH + ix_sw * inp_sW] * sw;
  97:           }
  98:           if (within_bounds_2d(iy_se, ix_se, inp_H, inp_W)) {
  99:             out_acc += inp_ptr_NC[iy_se * inp_sH + ix_se * inp_sW] * se;
 100:           }
 101:           *out_ptr_NCHW = out_acc;
 102:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 103-105
```cpp
 103:       } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
 104:         index_t ix_nearest = static_cast<index_t>(std::nearbyint(ix));
 105:         index_t iy_nearest = static_cast<index_t>(std::nearbyint(iy));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 107-107
```cpp
 107:         // assign nearest neighbour pixel value to output pixel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 108-117
```cpp
 108:         auto inp_ptr_NC = input.data + n * inp_sN;
 109:         auto out_ptr_NCHW = output.data + n * out_sN + h * out_sH + w * out_sW;
 110:         for (index_t c = 0; c < C; ++c, inp_ptr_NC += inp_sC, out_ptr_NCHW += out_sC) {
 111:           if (within_bounds_2d(iy_nearest, ix_nearest, inp_H, inp_W)) {
 112:             *out_ptr_NCHW = inp_ptr_NC[iy_nearest * inp_sH + ix_nearest * inp_sW];
 113:           } else {
 114:             *out_ptr_NCHW = static_cast<scalar_t>(0);
 115:           }
 116:         }
 117:       } else if (interpolation_mode == GridSamplerInterpolation::Bicubic) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 119-120
```cpp
 119:         ix = grid_sampler_unnormalize(x, inp_W, align_corners);
 120:         iy = grid_sampler_unnormalize(y, inp_H, align_corners);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 122-123
```cpp
 122:         opmath_t ix_nw = std::floor(ix);
 123:         opmath_t iy_nw = std::floor(iy);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 125-126
```cpp
 125:         const opmath_t tx = ix - ix_nw;
 126:         const opmath_t ty = iy - iy_nw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 128-149
```cpp
 128:         auto inp_ptr_NC = input.data + n * inp_sN;
 129:         auto out_ptr_NCHW = output.data + n * out_sN + h * out_sH + w * out_sW;
 130:         for (index_t c = 0; c < C; ++c, inp_ptr_NC += inp_sC, out_ptr_NCHW += out_sC) {
 131:           opmath_t coefficients[4];
 132: 
 133:           #pragma unroll 4
 134:           for (index_t i = 0; i < 4; ++i) {
 135:             coefficients[i] = cubic_interp1d(
 136:               get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw - 1, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
 137:               get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw + 0, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
 138:               get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw + 1, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
 139:               get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw + 2, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
 140:               tx);
 141:           }
 142: 
 143:           *out_ptr_NCHW = cubic_interp1d(
 144:             coefficients[0],
 145:             coefficients[1],
 146:             coefficients[2],
 147:             coefficients[3],
 148:             ty);
 149:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 150-152
```cpp
 150:       }
 151:     }
 152:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 154-175
```cpp
 154:   template <typename scalar_t, typename index_t>
 155:   C10_LAUNCH_BOUNDS_1(512)
 156:   __global__ void grid_sampler_3d_kernel(
 157:       const index_t nthreads,
 158:       TensorInfo<const scalar_t, index_t> input,
 159:       TensorInfo<const scalar_t, index_t> grid,
 160:       TensorInfo<scalar_t, index_t> output,
 161:       const GridSamplerInterpolation interpolation_mode,
 162:       const GridSamplerPadding padding_mode,
 163:       bool align_corners) {
 164: 
 165:     using opmath_t = at::opmath_type<scalar_t>;
 166:     index_t C = input.sizes[1];
 167:     index_t inp_D = input.sizes[2];
 168:     index_t inp_H = input.sizes[3];
 169:     index_t inp_W = input.sizes[4];
 170:     index_t out_D = grid.sizes[1];
 171:     index_t out_H = grid.sizes[2];
 172:     index_t out_W = grid.sizes[3];
 173:     index_t inp_sN = input.strides[0];
 174:     index_t inp_sC = input.strides[1];
 175:     index_t inp_sD = input.strides[2];
```
- EN: This block defines GPU kernel entry point(s) `grid_sampler_3d_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `grid_sampler_3d_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 176-187
```cpp
 176:     index_t inp_sH = input.strides[3];
 177:     index_t inp_sW = input.strides[4];
 178:     index_t grid_sN = grid.strides[0];
 179:     index_t grid_sD = grid.strides[1];
 180:     index_t grid_sH = grid.strides[2];
 181:     index_t grid_sW = grid.strides[3];
 182:     index_t grid_sCoor = grid.strides[4];
 183:     index_t out_sN = output.strides[0];
 184:     index_t out_sC = output.strides[1];
 185:     index_t out_sD = output.strides[2];
 186:     index_t out_sH = output.strides[3];
 187:     index_t out_sW = output.strides[4];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 189-210
```cpp
 189:     CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
 190:       const index_t w = index % out_W;
 191:       const index_t h = (index / out_W) % out_H;
 192:       const index_t d = (index / (out_H * out_W)) % out_D;
 193:       const index_t n = index / (out_D * out_H * out_W);
 194:       const index_t grid_offset = n * grid_sN + d * grid_sD + h * grid_sH + w * grid_sW;
 195: 
 196:       // get the corresponding input x, y, z coordinates from grid
 197:       opmath_t x = grid.data[grid_offset];
 198:       opmath_t y = grid.data[grid_offset + grid_sCoor];
 199:       opmath_t z = grid.data[grid_offset + 2 * grid_sCoor];
 200: 
 201:       opmath_t ix = grid_sampler_compute_source_index(x, inp_W, padding_mode, align_corners);
 202:       opmath_t iy = grid_sampler_compute_source_index(y, inp_H, padding_mode, align_corners);
 203:       opmath_t iz = grid_sampler_compute_source_index(z, inp_D, padding_mode, align_corners);
 204: 
 205:       if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
 206:         // get corner pixel values from (x, y, z)
 207:         // for 4d, we used north-east-south-west
 208:         // for 5d, we add top-bottom
 209:         index_t ix_tnw = static_cast<index_t>(::floor(ix));
 210:         index_t iy_tnw = static_cast<index_t>(::floor(iy));
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 211-211
```cpp
 211:         index_t iz_tnw = static_cast<index_t>(::floor(iz));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 213-215
```cpp
 213:         index_t ix_tne = ix_tnw + 1;
 214:         index_t iy_tne = iy_tnw;
 215:         index_t iz_tne = iz_tnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 217-219
```cpp
 217:         index_t ix_tsw = ix_tnw;
 218:         index_t iy_tsw = iy_tnw + 1;
 219:         index_t iz_tsw = iz_tnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 221-223
```cpp
 221:         index_t ix_tse = ix_tnw + 1;
 222:         index_t iy_tse = iy_tnw + 1;
 223:         index_t iz_tse = iz_tnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 225-227
```cpp
 225:         index_t ix_bnw = ix_tnw;
 226:         index_t iy_bnw = iy_tnw;
 227:         index_t iz_bnw = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 229-231
```cpp
 229:         index_t ix_bne = ix_tnw + 1;
 230:         index_t iy_bne = iy_tnw;
 231:         index_t iz_bne = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 233-235
```cpp
 233:         index_t ix_bsw = ix_tnw;
 234:         index_t iy_bsw = iy_tnw + 1;
 235:         index_t iz_bsw = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 237-239
```cpp
 237:         index_t ix_bse = ix_tnw + 1;
 238:         index_t iy_bse = iy_tnw + 1;
 239:         index_t iz_bse = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 241-241
```cpp
 241:         // get surfaces to each neighbor:
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 242-249
```cpp
 242:         opmath_t tnw = (ix_bse - ix)    * (iy_bse - iy)    * (iz_bse - iz);
 243:         opmath_t tne = (ix    - ix_bsw) * (iy_bsw - iy)    * (iz_bsw - iz);
 244:         opmath_t tsw = (ix_bne - ix)    * (iy    - iy_bne) * (iz_bne - iz);
 245:         opmath_t tse = (ix    - ix_bnw) * (iy    - iy_bnw) * (iz_bnw - iz);
 246:         opmath_t bnw = (ix_tse - ix)    * (iy_tse - iy)    * (iz - iz_tse);
 247:         opmath_t bne = (ix    - ix_tsw) * (iy_tsw - iy)    * (iz - iz_tsw);
 248:         opmath_t bsw = (ix_tne - ix)    * (iy    - iy_tne) * (iz - iz_tne);
 249:         opmath_t bse = (ix    - ix_tnw) * (iy    - iy_tnw) * (iz - iz_tnw);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 251-272
```cpp
 251:         auto inp_ptr_NC = input.data + n * inp_sN;
 252:         auto out_ptr_NCDHW = output.data + n * out_sN + d * out_sD + h * out_sH + w * out_sW;
 253:         for (index_t c = 0; c < C; ++c, inp_ptr_NC += inp_sC, out_ptr_NCDHW += out_sC) {
 254:           //   (c, iz_tnw, iy_tnw, ix_tnw) * tnw + (c, iz_tne, iy_tne, ix_tne) * tne
 255:           // + (c, iz_tsw, iy_tsw, ix_tsw) * tsw + (c, iz_tse, iy_tse, ix_tse) * tse
 256:           // + (c, iz_bnw, iy_bnw, ix_bnw) * bnw + (c, iz_bne, iy_bne, ix_bne) * bne
 257:           // + (c, iz_bsw, iy_bsw, ix_bsw) * bsw + (c, iz_bse, iy_bse, ix_bse) * bse
 258:           opmath_t out_acc = 0;
 259:           if (within_bounds_3d(iz_tnw, iy_tnw, ix_tnw, inp_D, inp_H, inp_W)) {
 260:             out_acc += inp_ptr_NC[iz_tnw * inp_sD + iy_tnw * inp_sH + ix_tnw * inp_sW] * tnw;
 261:           }
 262:           if (within_bounds_3d(iz_tne, iy_tne, ix_tne, inp_D, inp_H, inp_W)) {
 263:             out_acc += inp_ptr_NC[iz_tne * inp_sD + iy_tne * inp_sH + ix_tne * inp_sW] * tne;
 264:           }
 265:           if (within_bounds_3d(iz_tsw, iy_tsw, ix_tsw, inp_D, inp_H, inp_W)) {
 266:             out_acc += inp_ptr_NC[iz_tsw * inp_sD + iy_tsw * inp_sH + ix_tsw * inp_sW] * tsw;
 267:           }
 268:           if (within_bounds_3d(iz_tse, iy_tse, ix_tse, inp_D, inp_H, inp_W)) {
 269:             out_acc += inp_ptr_NC[iz_tse * inp_sD + iy_tse * inp_sH + ix_tse * inp_sW] * tse;
 270:           }
 271:           if (within_bounds_3d(iz_bnw, iy_bnw, ix_bnw, inp_D, inp_H, inp_W)) {
 272:             out_acc += inp_ptr_NC[iz_bnw * inp_sD + iy_bnw * inp_sH + ix_bnw * inp_sW] * bnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 273-286
```cpp
 273:           }
 274:           if (within_bounds_3d(iz_bne, iy_bne, ix_bne, inp_D, inp_H, inp_W)) {
 275:             out_acc += inp_ptr_NC[iz_bne * inp_sD + iy_bne * inp_sH + ix_bne * inp_sW] * bne;
 276:           }
 277:           if (within_bounds_3d(iz_bsw, iy_bsw, ix_bsw, inp_D, inp_H, inp_W)) {
 278:             out_acc += inp_ptr_NC[iz_bsw * inp_sD + iy_bsw * inp_sH + ix_bsw * inp_sW] * bsw;
 279:           }
 280:           if (within_bounds_3d(iz_bse, iy_bse, ix_bse, inp_D, inp_H, inp_W)) {
 281:             out_acc += inp_ptr_NC[iz_bse * inp_sD + iy_bse * inp_sH + ix_bse * inp_sW] * bse;
 282:           }
 283:           *out_ptr_NCDHW = out_acc;
 284:         }
 285:       } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
 286:         index_t ix_nearest = static_cast<index_t>(std::nearbyint(ix));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 287-288
```cpp
 287:         index_t iy_nearest = static_cast<index_t>(std::nearbyint(iy));
 288:         index_t iz_nearest = static_cast<index_t>(std::nearbyint(iz));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 290-290
```cpp
 290:         // assign nearest neighbour pixel value to output pixel
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 291-302
```cpp
 291:         auto inp_ptr_NC = input.data + n * inp_sN;
 292:         auto out_ptr_NCDHW = output.data + n * out_sN + d * out_sD + h * out_sH + w * out_sW;
 293:         for (index_t c = 0; c < C; ++c, inp_ptr_NC += inp_sC, out_ptr_NCDHW += out_sC) {
 294:           if (within_bounds_3d(iz_nearest, iy_nearest, ix_nearest, inp_D, inp_H, inp_W)) {
 295:             *out_ptr_NCDHW = inp_ptr_NC[iz_nearest * inp_sD + iy_nearest * inp_sH + ix_nearest * inp_sW];
 296:           } else {
 297:             *out_ptr_NCDHW = static_cast<scalar_t>(0);
 298:           }
 299:         }
 300:       }
 301:     }
 302:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 304-308
```cpp
 304: // Note [Passing pointer and offset to fastAtomicAdd]
 305: // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 306: // For its internal bounds checking, fastAtomicAdd needs to know where the destination address
 307: // lies relative to the entire tensor, so we pass the base grad_input.data and full offset information,
 308: // including batch * channel offset (NC_offset).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 310-331
```cpp
 310:   template <typename scalar_t, typename index_t>
 311:   C10_LAUNCH_BOUNDS_1(256)
 312:   __global__ void grid_sampler_2d_backward_kernel(
 313:       const index_t nthreads,
 314:       TensorInfo<const scalar_t, index_t> grad_output,
 315:       TensorInfo<const scalar_t, index_t> input,
 316:       TensorInfo<const scalar_t, index_t> grid,
 317:       TensorInfo<scalar_t, index_t> grad_input,  // initialized to zeros (or unused if input_requires_grad is false)
 318:       TensorInfo<scalar_t, index_t> grad_grid,   // initialized to empty
 319:       const GridSamplerInterpolation interpolation_mode,
 320:       const GridSamplerPadding padding_mode,
 321:       bool align_corners,
 322:       const index_t grad_input_memory_span,
 323:       const bool input_requires_grad) {
 324: 
 325:     index_t C = input.sizes[1];
 326:     index_t inp_H = input.sizes[2];
 327:     index_t inp_W = input.sizes[3];
 328:     index_t out_H = grid.sizes[1];
 329:     index_t out_W = grid.sizes[2];
 330:     index_t inp_sN = input.strides[0];
 331:     index_t inp_sC = input.strides[1];
```
- EN: This block defines GPU kernel entry point(s) `grid_sampler_2d_backward_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `grid_sampler_2d_backward_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 332-345
```cpp
 332:     index_t inp_sH = input.strides[2];
 333:     index_t inp_sW = input.strides[3];
 334:     index_t grid_sN = grid.strides[0];
 335:     index_t grid_sH = grid.strides[1];
 336:     index_t grid_sW = grid.strides[2];
 337:     index_t grid_sCoor = grid.strides[3];
 338:     index_t gOut_sN = grad_output.strides[0];
 339:     index_t gOut_sC = grad_output.strides[1];
 340:     index_t gOut_sH = grad_output.strides[2];
 341:     index_t gOut_sW = grad_output.strides[3];
 342:     // gInp_* (and NC_offset below) are not really needed if input_requires_grad is false.
 343:     index_t gInp_sN;
 344:     index_t gInp_sC;
 345:     index_t gInp_sH;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 346-353
```cpp
 346:     index_t gInp_sW;
 347:     if (input_requires_grad) {
 348:       gInp_sN = grad_input.strides[0];
 349:       gInp_sC = grad_input.strides[1];
 350:       gInp_sH = grad_input.strides[2];
 351:       gInp_sW = grad_input.strides[3];
 352:     }
 353:     index_t gGrid_sW = grad_grid.strides[2];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 355-376
```cpp
 355:     CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
 356:       const index_t w = index % out_W;
 357:       const index_t h = (index / out_W) % out_H;
 358:       const index_t n = index / (out_H * out_W);
 359:       const auto grid_offset = n * grid_sN + h * grid_sH + w * grid_sW;
 360: 
 361:       // get the corresponding input x, y coordinates from grid
 362:       scalar_t x = grid.data[grid_offset];
 363:       scalar_t y = grid.data[grid_offset + grid_sCoor];
 364: 
 365:       // multipliers for gradients on ix and iy
 366:       scalar_t gix_mult, giy_mult;
 367:       scalar_t ix = grid_sampler_compute_source_index_set_grad(x, inp_W, padding_mode, align_corners, &gix_mult);
 368:       scalar_t iy = grid_sampler_compute_source_index_set_grad(y, inp_H, padding_mode, align_corners, &giy_mult);
 369: 
 370:       if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
 371:         // get NE, NW, SE, SW pixel values from (x, y)
 372:         index_t ix_nw = static_cast<index_t>(std::floor(ix));
 373:         index_t iy_nw = static_cast<index_t>(std::floor(iy));
 374:         index_t ix_ne = ix_nw + 1;
 375:         index_t iy_ne = iy_nw;
 376:         index_t ix_sw = ix_nw;
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 377-379
```cpp
 377:         index_t iy_sw = iy_nw + 1;
 378:         index_t ix_se = ix_nw + 1;
 379:         index_t iy_se = iy_nw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 381-381
```cpp
 381:         // get surfaces to each neighbor:
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 382-385
```cpp
 382:         scalar_t nw = (ix_se - ix)    * (iy_se - iy);
 383:         scalar_t ne = (ix    - ix_sw) * (iy_sw - iy);
 384:         scalar_t sw = (ix_ne - ix)    * (iy    - iy_ne);
 385:         scalar_t se = (ix    - ix_nw) * (iy    - iy_nw);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 387-408
```cpp
 387:         scalar_t gix = static_cast<scalar_t>(0), giy = static_cast<scalar_t>(0);
 388:         const scalar_t *gOut_ptr_NCHW = grad_output.data + n * gOut_sN + h * gOut_sH + w * gOut_sW;
 389:         index_t NC_offset = n * gInp_sN;
 390:         const scalar_t *inp_ptr_NC = input.data + n * inp_sN;
 391:         for (index_t c = 0; c < C; ++c, inp_ptr_NC += inp_sC, NC_offset += gInp_sC, gOut_ptr_NCHW += gOut_sC) {
 392:           const scalar_t gOut = *gOut_ptr_NCHW;
 393: 
 394:           if (input_requires_grad) {
 395:             // calculate and set grad_input. See Note [Passing pointer and offset to fastAtomicAdd].
 396:             safe_add_2d(grad_input.data, iy_nw, ix_nw, gInp_sH, gInp_sW, inp_H, inp_W, nw * gOut, NC_offset, grad_input_memory_span);
 397:             safe_add_2d(grad_input.data, iy_ne, ix_ne, gInp_sH, gInp_sW, inp_H, inp_W, ne * gOut, NC_offset, grad_input_memory_span);
 398:             safe_add_2d(grad_input.data, iy_sw, ix_sw, gInp_sH, gInp_sW, inp_H, inp_W, sw * gOut, NC_offset, grad_input_memory_span);
 399:             safe_add_2d(grad_input.data, iy_se, ix_se, gInp_sH, gInp_sW, inp_H, inp_W, se * gOut, NC_offset, grad_input_memory_span);
 400:           }
 401: 
 402:           // calculate grad_grid
 403:           if (within_bounds_2d(iy_nw, ix_nw, inp_H, inp_W)) {
 404:             scalar_t nw_val = inp_ptr_NC[iy_nw * inp_sH + ix_nw * inp_sW];
 405:             gix -= nw_val * (iy_se - iy) * gOut;
 406:             giy -= nw_val * (ix_se - ix) * gOut;
 407:           }
 408:           if (within_bounds_2d(iy_ne, ix_ne, inp_H, inp_W)) {
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 409-422
```cpp
 409:             scalar_t ne_val = inp_ptr_NC[iy_ne * inp_sH + ix_ne * inp_sW];
 410:             gix += ne_val * (iy_sw - iy) * gOut;
 411:             giy -= ne_val * (ix - ix_sw) * gOut;
 412:           }
 413:           if (within_bounds_2d(iy_sw, ix_sw, inp_H, inp_W)) {
 414:             scalar_t sw_val = inp_ptr_NC[iy_sw * inp_sH + ix_sw * inp_sW];
 415:             gix -= sw_val * (iy - iy_ne) * gOut;
 416:             giy += sw_val * (ix_ne - ix) * gOut;
 417:           }
 418:           if (within_bounds_2d(iy_se, ix_se, inp_H, inp_W)) {
 419:             scalar_t se_val = inp_ptr_NC[iy_se * inp_sH + ix_se * inp_sW];
 420:             gix += se_val * (iy - iy_nw) * gOut;
 421:             giy += se_val * (ix - ix_nw) * gOut;
 422:           }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 423-423
```cpp
 423:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 425-428
```cpp
 425:         // assuming grad_grid is contiguous
 426:         // thus we can
 427:         //   1. use index with gGrid_sW to directly compute gGrid_ptr_NHW
 428:         //   2. directly assign to gGrid_ptr_NHW[0], gGrid_ptr_NHW[1]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 429-444
```cpp
 429:         scalar_t *gGrid_ptr_NHW = grad_grid.data + index * gGrid_sW;
 430:         gGrid_ptr_NHW[0] = gix_mult * gix;
 431:         gGrid_ptr_NHW[1] = giy_mult * giy;
 432:       } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
 433:         if (input_requires_grad) {
 434:           index_t ix_nearest = static_cast<index_t>(std::nearbyint(ix));
 435:           index_t iy_nearest = static_cast<index_t>(std::nearbyint(iy));
 436: 
 437:           // assign nearest neighbour pixel value to output pixel
 438:           const scalar_t *gOut_ptr_NCHW = grad_output.data + n * gOut_sN + h * gOut_sH + w * gOut_sW;
 439:           index_t NC_offset = n * gInp_sN;
 440:           for (index_t c = 0; c < C; ++c, NC_offset += gInp_sC, gOut_ptr_NCHW += gOut_sC) {
 441:             // calculate and set grad_input. See Note [Passing pointer and offset to fastAtomicAdd].
 442:             safe_add_2d(grad_input.data, iy_nearest, ix_nearest, gInp_sH, gInp_sW, inp_H, inp_W, *gOut_ptr_NCHW, NC_offset, grad_input_memory_span);
 443:           }
 444:         }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 446-449
```cpp
 446:         // assuming grad_grid is contiguous
 447:         // thus we can
 448:         //   1. use index with gGrid_sW to directly compute gGrid_ptr_NHW
 449:         //   2. directly assign to gGrid_ptr_NHW[0], gGrid_ptr_NHW[1]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 450-453
```cpp
 450:         scalar_t *gGrid_ptr_NHW = grad_grid.data + index * gGrid_sW;
 451:         gGrid_ptr_NHW[0] = static_cast<scalar_t>(0);
 452:         gGrid_ptr_NHW[1] = static_cast<scalar_t>(0);
 453:       } else if (interpolation_mode == GridSamplerInterpolation::Bicubic) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 455-456
```cpp
 455:         ix = grid_sampler_unnormalize_set_grad(x, inp_W, align_corners, &gix_mult);
 456:         iy = grid_sampler_unnormalize_set_grad(y, inp_H, align_corners, &giy_mult);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 458-459
```cpp
 458:         scalar_t ix_nw = std::floor(ix);
 459:         scalar_t iy_nw = std::floor(iy);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 461-462
```cpp
 461:         const scalar_t tx = ix - ix_nw;
 462:         const scalar_t ty = iy - iy_nw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 464-467
```cpp
 464:         scalar_t x_coeffs[4];
 465:         scalar_t y_coeffs[4];
 466:         scalar_t x_coeffs_grad[4];
 467:         scalar_t y_coeffs_grad[4];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 469-472
```cpp
 469:         get_cubic_upsampling_coefficients<scalar_t>(x_coeffs, tx);
 470:         get_cubic_upsampling_coefficients<scalar_t>(y_coeffs, ty);
 471:         get_cubic_coefficients_grad<scalar_t>(x_coeffs_grad, tx);
 472:         get_cubic_coefficients_grad<scalar_t>(y_coeffs_grad, ty);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 474-475
```cpp
 474:         scalar_t gix = static_cast<scalar_t>(0);
 475:         scalar_t giy = static_cast<scalar_t>(0);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 477-479
```cpp
 477:         const scalar_t *gOut_ptr_NCHW = grad_output.data + n * gOut_sN + h * gOut_sH + w * gOut_sW;
 478:         index_t NC_offset = n * gInp_sN;
 479:         const scalar_t *inp_ptr_NC = input.data + n * inp_sN;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 481-501
```cpp
 481:         for (index_t c = 0; c < C; ++c, gOut_ptr_NCHW += gOut_sC, NC_offset += gInp_sC, inp_ptr_NC+= inp_sC) {
 482:           const scalar_t gOut = *gOut_ptr_NCHW;
 483: 
 484:           #pragma unroll 4
 485:           for (index_t i = 0; i < 4; ++i) {
 486:             #pragma unroll 4
 487:             for (index_t j = 0; j < 4; ++j) {
 488: 
 489:               if (input_requires_grad) {
 490:                 // set input gradient. See Note [Passing pointer and offset to fastAtomicAdd].
 491:                 add_value_bounded<scalar_t>(grad_input.data, ix_nw - 1 + i, iy_nw - 1 + j, inp_W, inp_H, gInp_sW, gInp_sH,
 492:                   gOut * x_coeffs[i] * y_coeffs[j],
 493:                   padding_mode,
 494:                   align_corners,
 495:                   NC_offset,
 496:                   grad_input_memory_span);
 497:               }
 498: 
 499:               // set grid gradient
 500:               scalar_t val = get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw - 1 + i, iy_nw - 1 + j,
 501:                 inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 503-507
```cpp
 503:               gix -= val * x_coeffs_grad[i] * y_coeffs[j] * gOut;
 504:               giy -= val * y_coeffs_grad[j] * x_coeffs[i] * gOut;
 505:             }
 506:           }
 507:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 509-514
```cpp
 509:         scalar_t *gGrid_ptr_NHW = grad_grid.data + index * gGrid_sW;
 510:         gGrid_ptr_NHW[0] = gix_mult * gix;
 511:         gGrid_ptr_NHW[1] = giy_mult * giy;
 512:       }
 513:     }
 514:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 516-537
```cpp
 516:   template <typename scalar_t, typename index_t>
 517:   C10_LAUNCH_BOUNDS_1(256)
 518:   __global__ void grid_sampler_3d_backward_kernel(
 519:       const index_t nthreads,
 520:       TensorInfo<const scalar_t, index_t> grad_output,
 521:       TensorInfo<const scalar_t, index_t> input,
 522:       TensorInfo<const scalar_t, index_t> grid,
 523:       TensorInfo<scalar_t, index_t> grad_input,  // initialized to zeros (or unused if input_requires_grad is false)
 524:       TensorInfo<scalar_t, index_t> grad_grid,   // initialized to empty
 525:       const GridSamplerInterpolation interpolation_mode,
 526:       const GridSamplerPadding padding_mode,
 527:       bool align_corners,
 528:       const index_t grad_input_memory_span,
 529:       const bool input_requires_grad) {
 530: 
 531:     index_t C = input.sizes[1];
 532:     index_t inp_D = input.sizes[2];
 533:     index_t inp_H = input.sizes[3];
 534:     index_t inp_W = input.sizes[4];
 535:     index_t out_D = grid.sizes[1];
 536:     index_t out_H = grid.sizes[2];
 537:     index_t out_W = grid.sizes[3];
```
- EN: This block defines GPU kernel entry point(s) `grid_sampler_3d_backward_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `grid_sampler_3d_backward_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 538-551
```cpp
 538:     index_t inp_sN = input.strides[0];
 539:     index_t inp_sC = input.strides[1];
 540:     index_t inp_sD = input.strides[2];
 541:     index_t inp_sH = input.strides[3];
 542:     index_t inp_sW = input.strides[4];
 543:     index_t grid_sN = grid.strides[0];
 544:     index_t grid_sD = grid.strides[1];
 545:     index_t grid_sH = grid.strides[2];
 546:     index_t grid_sW = grid.strides[3];
 547:     index_t grid_sCoor = grid.strides[4];
 548:     index_t gOut_sN = grad_output.strides[0];
 549:     index_t gOut_sC = grad_output.strides[1];
 550:     index_t gOut_sD = grad_output.strides[2];
 551:     index_t gOut_sH = grad_output.strides[3];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 552-565
```cpp
 552:     index_t gOut_sW = grad_output.strides[4];
 553:     // gInp_* (and NC_offset below) are not really needed if input_requires_grad is false.
 554:     int64_t gInp_sN = 0;
 555:     int64_t gInp_sC = 0;
 556:     int64_t gInp_sD = 0;
 557:     int64_t gInp_sH = 0;
 558:     int64_t gInp_sW = 0;
 559:     if (input_requires_grad) {
 560:       gInp_sN = grad_input.strides[0];
 561:       gInp_sC = grad_input.strides[1];
 562:       gInp_sD = grad_input.strides[2];
 563:       gInp_sH = grad_input.strides[3];
 564:       gInp_sW = grad_input.strides[4];
 565:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 566-566
```cpp
 566:     index_t gGrid_sW = grad_grid.strides[3];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 568-589
```cpp
 568:     CUDA_KERNEL_LOOP_TYPE(index, nthreads, index_t) {
 569:       const index_t w = index % out_W;
 570:       const index_t h = (index / out_W) % out_H;
 571:       const index_t d = (index / (out_H * out_W)) % out_D;
 572:       const index_t n = index / (out_D * out_H * out_W);
 573:       const auto grid_offset = n * grid_sN + d * grid_sD + h * grid_sH + w * grid_sW;
 574: 
 575:       // get the corresponding input x, y, z coordinates from grid
 576:       scalar_t ix = grid.data[grid_offset];
 577:       scalar_t iy = grid.data[grid_offset + grid_sCoor];
 578:       scalar_t iz = grid.data[grid_offset + 2 * grid_sCoor];
 579: 
 580:       // multipliers for gradients on ix, iy, and iz
 581:       scalar_t gix_mult, giy_mult, giz_mult;
 582:       ix = grid_sampler_compute_source_index_set_grad(ix, inp_W, padding_mode, align_corners, &gix_mult);
 583:       iy = grid_sampler_compute_source_index_set_grad(iy, inp_H, padding_mode, align_corners, &giy_mult);
 584:       iz = grid_sampler_compute_source_index_set_grad(iz, inp_D, padding_mode, align_corners, &giz_mult);
 585: 
 586:       if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
 587:         // get corner pixel values from (x, y, z)
 588:         // for 4d, we used north-east-south-west
 589:         // for 5d, we add top-bottom
```
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 590-592
```cpp
 590:         index_t ix_tnw = static_cast<index_t>(std::floor(ix));
 591:         index_t iy_tnw = static_cast<index_t>(std::floor(iy));
 592:         index_t iz_tnw = static_cast<index_t>(std::floor(iz));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 594-596
```cpp
 594:         index_t ix_tne = ix_tnw + 1;
 595:         index_t iy_tne = iy_tnw;
 596:         index_t iz_tne = iz_tnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 598-600
```cpp
 598:         index_t ix_tsw = ix_tnw;
 599:         index_t iy_tsw = iy_tnw + 1;
 600:         index_t iz_tsw = iz_tnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 602-604
```cpp
 602:         index_t ix_tse = ix_tnw + 1;
 603:         index_t iy_tse = iy_tnw + 1;
 604:         index_t iz_tse = iz_tnw;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 606-608
```cpp
 606:         index_t ix_bnw = ix_tnw;
 607:         index_t iy_bnw = iy_tnw;
 608:         index_t iz_bnw = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 610-612
```cpp
 610:         index_t ix_bne = ix_tnw + 1;
 611:         index_t iy_bne = iy_tnw;
 612:         index_t iz_bne = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 614-616
```cpp
 614:         index_t ix_bsw = ix_tnw;
 615:         index_t iy_bsw = iy_tnw + 1;
 616:         index_t iz_bsw = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 618-620
```cpp
 618:         index_t ix_bse = ix_tnw + 1;
 619:         index_t iy_bse = iy_tnw + 1;
 620:         index_t iz_bse = iz_tnw + 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 622-622
```cpp
 622:         // get surfaces to each neighbor:
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 623-630
```cpp
 623:         scalar_t tnw = (ix_bse - ix)    * (iy_bse - iy)    * (iz_bse - iz);
 624:         scalar_t tne = (ix    - ix_bsw) * (iy_bsw - iy)    * (iz_bsw - iz);
 625:         scalar_t tsw = (ix_bne - ix)    * (iy    - iy_bne) * (iz_bne - iz);
 626:         scalar_t tse = (ix    - ix_bnw) * (iy    - iy_bnw) * (iz_bnw - iz);
 627:         scalar_t bnw = (ix_tse - ix)    * (iy_tse - iy)    * (iz - iz_tse);
 628:         scalar_t bne = (ix    - ix_tsw) * (iy_tsw - iy)    * (iz - iz_tsw);
 629:         scalar_t bsw = (ix_tne - ix)    * (iy    - iy_tne) * (iz - iz_tne);
 630:         scalar_t bse = (ix    - ix_tnw) * (iy    - iy_tnw) * (iz - iz_tnw);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 632-653
```cpp
 632:         scalar_t gix = static_cast<scalar_t>(0), giy = static_cast<scalar_t>(0), giz = static_cast<scalar_t>(0);
 633:         const scalar_t *gOut_ptr_NCDHW = grad_output.data + n * gOut_sN + d * gOut_sD + h * gOut_sH + w * gOut_sW;
 634:         index_t NC_offset;
 635:         if (input_requires_grad) {
 636:           NC_offset = n * gInp_sN;
 637:         }
 638:         const scalar_t *inp_ptr_NC = input.data + n * inp_sN;
 639:         // calculate bilinear weighted pixel value and set output pixel
 640:         for (index_t c = 0; c < C; ++c, gOut_ptr_NCDHW += gOut_sC, NC_offset += gInp_sC, inp_ptr_NC += inp_sC) {
 641:           scalar_t gOut = *gOut_ptr_NCDHW;
 642: 
 643:           // calculate and set grad_input. See Note [Passing pointer and offset to fastAtomicAdd].
 644:           if (input_requires_grad) {
 645:             safe_add_3d(grad_input.data, iz_tnw, iy_tnw, ix_tnw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tnw * gOut,
 646:                         NC_offset, grad_input_memory_span);
 647:             safe_add_3d(grad_input.data, iz_tne, iy_tne, ix_tne, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tne * gOut,
 648:                         NC_offset, grad_input_memory_span);
 649:             safe_add_3d(grad_input.data, iz_tsw, iy_tsw, ix_tsw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tsw * gOut,
 650:                         NC_offset, grad_input_memory_span);
 651:             safe_add_3d(grad_input.data, iz_tse, iy_tse, ix_tse, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tse * gOut,
 652:                         NC_offset, grad_input_memory_span);
 653:             safe_add_3d(grad_input.data, iz_bnw, iy_bnw, ix_bnw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bnw * gOut,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 654-667
```cpp
 654:                         NC_offset, grad_input_memory_span);
 655:             safe_add_3d(grad_input.data, iz_bne, iy_bne, ix_bne, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bne * gOut,
 656:                         NC_offset, grad_input_memory_span);
 657:             safe_add_3d(grad_input.data, iz_bsw, iy_bsw, ix_bsw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bsw * gOut,
 658:                         NC_offset, grad_input_memory_span);
 659:             safe_add_3d(grad_input.data, iz_bse, iy_bse, ix_bse, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bse * gOut,
 660:                         NC_offset, grad_input_memory_span);
 661:           }
 662:           // calculate grad_grid
 663:           if (within_bounds_3d(iz_tnw, iy_tnw, ix_tnw, inp_D, inp_H, inp_W)) {
 664:             scalar_t tnw_val = inp_ptr_NC[iz_tnw * inp_sD + iy_tnw * inp_sH + ix_tnw * inp_sW];
 665:             gix -= tnw_val * (iy_bse - iy)    * (iz_bse - iz)    * gOut;
 666:             giy -= tnw_val * (ix_bse - ix)    * (iz_bse - iz)    * gOut;
 667:             giz -= tnw_val * (ix_bse - ix)    * (iy_bse - iy)    * gOut;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 668-681
```cpp
 668:           }
 669:           if (within_bounds_3d(iz_tne, iy_tne, ix_tne, inp_D, inp_H, inp_W)) {
 670:             scalar_t tne_val = inp_ptr_NC[iz_tne * inp_sD + iy_tne * inp_sH + ix_tne * inp_sW];
 671:             gix += tne_val * (iy_bsw - iy)    * (iz_bsw - iz)    * gOut;
 672:             giy -= tne_val * (ix    - ix_bsw) * (iz_bsw - iz)    * gOut;
 673:             giz -= tne_val * (ix    - ix_bsw) * (iy_bsw - iy)    * gOut;
 674:           }
 675:           if (within_bounds_3d(iz_tsw, iy_tsw, ix_tsw, inp_D, inp_H, inp_W)) {
 676:             scalar_t tsw_val = inp_ptr_NC[iz_tsw * inp_sD + iy_tsw * inp_sH + ix_tsw * inp_sW];
 677:             gix -= tsw_val * (iy - iy_bne)    * (iz_bne - iz)    * gOut;
 678:             giy += tsw_val * (ix_bne - ix)    * (iz_bne - iz)    * gOut;
 679:             giz -= tsw_val * (ix_bne - ix)    * (iy    - iy_bne) * gOut;
 680:           }
 681:           if (within_bounds_3d(iz_tse, iy_tse, ix_tse, inp_D, inp_H, inp_W)) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 682-695
```cpp
 682:             scalar_t tse_val = inp_ptr_NC[iz_tse * inp_sD + iy_tse * inp_sH + ix_tse * inp_sW];
 683:             gix += tse_val * (iy - iy_bnw)    * (iz_bnw - iz)    * gOut;
 684:             giy += tse_val * (ix    - ix_bnw) * (iz_bnw - iz)    * gOut;
 685:             giz -= tse_val * (ix    - ix_bnw) * (iy    - iy_bnw) * gOut;
 686:           }
 687:           if (within_bounds_3d(iz_bnw, iy_bnw, ix_bnw, inp_D, inp_H, inp_W)) {
 688:             scalar_t bnw_val = inp_ptr_NC[iz_bnw * inp_sD + iy_bnw * inp_sH + ix_bnw * inp_sW];
 689:             gix -= bnw_val * (iy_tse - iy)    * (iz - iz_tse)    * gOut;
 690:             giy -= bnw_val * (ix_tse - ix)    * (iz - iz_tse)    * gOut;
 691:             giz += bnw_val * (ix_tse - ix)    * (iy_tse - iy)    * gOut;
 692:           }
 693:           if (within_bounds_3d(iz_bne, iy_bne, ix_bne, inp_D, inp_H, inp_W)) {
 694:             scalar_t bne_val = inp_ptr_NC[iz_bne * inp_sD + iy_bne * inp_sH + ix_bne * inp_sW];
 695:             gix += bne_val * (iy_tsw - iy)    * (iz - iz_tsw)    * gOut;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 696-709
```cpp
 696:             giy -= bne_val * (ix    - ix_tsw) * (iz - iz_tsw)    * gOut;
 697:             giz += bne_val * (ix    - ix_tsw) * (iy_tsw - iy)    * gOut;
 698:           }
 699:           if (within_bounds_3d(iz_bsw, iy_bsw, ix_bsw, inp_D, inp_H, inp_W)) {
 700:             scalar_t bsw_val = inp_ptr_NC[iz_bsw * inp_sD + iy_bsw * inp_sH + ix_bsw * inp_sW];
 701:             gix -= bsw_val * (iy - iy_tne)    * (iz - iz_tne)    * gOut;
 702:             giy += bsw_val * (ix_tne - ix)    * (iz - iz_tne)    * gOut;
 703:             giz += bsw_val * (ix_tne - ix)    * (iy    - iy_tne) * gOut;
 704:           }
 705:           if (within_bounds_3d(iz_bse, iy_bse, ix_bse, inp_D, inp_H, inp_W)) {
 706:             scalar_t bse_val = inp_ptr_NC[iz_bse * inp_sD + iy_bse * inp_sH + ix_bse * inp_sW];
 707:             gix += bse_val * (iy - iy_tnw)    * (iz - iz_tnw)    * gOut;
 708:             giy += bse_val * (ix    - ix_tnw) * (iz - iz_tnw)    * gOut;
 709:             giz += bse_val * (ix    - ix_tnw) * (iy    - iy_tnw) * gOut;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 710-711
```cpp
 710:           }
 711:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 713-716
```cpp
 713:         // assuming grad_grid is contiguous
 714:         // thus we can
 715:         //   1. use index with gGrid_sW to directly compute gGrid_ptr_NDHW
 716:         //   2. directly assign to gGrid_ptr_NDHW[0], gGrid_ptr_NDHW[1], gGrid_ptr_NDHW[2]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 717-736
```cpp
 717:         scalar_t *gGrid_ptr_NDHW = grad_grid.data + index * gGrid_sW;
 718:         gGrid_ptr_NDHW[0] = gix_mult * gix;
 719:         gGrid_ptr_NDHW[1] = giy_mult * giy;
 720:         gGrid_ptr_NDHW[2] = giz_mult * giz;
 721:       } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
 722:         if (input_requires_grad) {
 723:           auto ix_nearest = static_cast<index_t>(std::nearbyint(ix));
 724:           auto iy_nearest = static_cast<index_t>(std::nearbyint(iy));
 725:           auto iz_nearest = static_cast<index_t>(std::nearbyint(iz));
 726: 
 727:           // assign nearest neighbour pixel value to output pixel
 728:           const scalar_t *gOut_ptr_NCDHW = grad_output.data + n * gOut_sN + d * gOut_sD + h * gOut_sH + w * gOut_sW;
 729:           index_t NC_offset = n * gInp_sN;
 730:           for (index_t c = 0; c < C; ++c, gOut_ptr_NCDHW += gOut_sC, NC_offset += gInp_sC) {
 731:             // calculate and set grad_input. See Note [Passing pointer and offset to fastAtomicAdd].
 732:             safe_add_3d(grad_input.data, iz_nearest, iy_nearest, ix_nearest,
 733:                         gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, *gOut_ptr_NCDHW,
 734:                         NC_offset, grad_input_memory_span);
 735:           }
 736:         }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 737-740
```cpp
 737:         // assuming grad_grid is contiguous
 738:         // thus we can
 739:         //   1. use index with gGrid_sW to directly compute gGrid_ptr_NDHW
 740:         //   2. directly assign to gGrid_ptr_NDHW[0], gGrid_ptr_NDHW[1], gGrid_ptr_NDHW[2]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 741-748
```cpp
 741:         scalar_t *gGrid_ptr_NDHW = grad_grid.data + index * gGrid_sW;
 742:         gGrid_ptr_NDHW[0] = static_cast<scalar_t>(0);
 743:         gGrid_ptr_NDHW[1] = static_cast<scalar_t>(0);
 744:         gGrid_ptr_NDHW[2] = static_cast<scalar_t>(0);
 745:       }
 746:     }
 747:   }
 748: }  // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 750-771
```cpp
 750: void launch_grid_sampler_2d_forward_kernel(
 751:     const TensorBase &output, const TensorBase &input, const TensorBase &grid,
 752:     int64_t interpolation_mode, int64_t padding_mode, bool align_corners) {
 753:   // See NOTE [ grid_sampler Native Functions ].
 754:   // Add checks here in case this is called instead of grid_sampler.
 755:   check_grid_sampler_common(input, grid);
 756:   check_grid_sampler_2d(input, grid);
 757: 
 758:   auto N = input.size(0);
 759:   auto H = grid.size(1);
 760:   auto W = grid.size(2);
 761:   int64_t count = N * H * W;
 762:   if (count > 0) {
 763:     AT_DISPATCH_FLOATING_TYPES_AND2(
 764:       ScalarType::Half, ScalarType::BFloat16,
 765:       input.scalar_type(), "grid_sampler_2d_cuda", [&] {
 766:       if (canUse32BitIndexMath(input) && canUse32BitIndexMath(grid) &&
 767:           canUse32BitIndexMath(output)) {
 768:         grid_sampler_2d_kernel<scalar_t>
 769:           <<<GET_BLOCKS(count, 256), 256, 0, at::cuda::getCurrentCUDAStream()>>>(
 770:             static_cast<int>(count),
 771:             getTensorInfo<const scalar_t, int>(input),
```
- EN: This block defines or continues the implementation of `launch_grid_sampler_2d_forward_kernel`.
- CN: 该代码块定义或继续实现 `launch_grid_sampler_2d_forward_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 772-785
```cpp
 772:             getTensorInfo<const scalar_t, int>(grid),
 773:             getTensorInfo<scalar_t, int>(output),
 774:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 775:             static_cast<GridSamplerPadding>(padding_mode),
 776:             align_corners);
 777:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 778:       } else {
 779:         grid_sampler_2d_kernel<scalar_t>
 780:           <<<GET_BLOCKS(count, 256), 256, 0, at::cuda::getCurrentCUDAStream()>>>(
 781:             count,
 782:             getTensorInfo<const scalar_t, int64_t>(input),
 783:             getTensorInfo<const scalar_t, int64_t>(grid),
 784:             getTensorInfo<scalar_t, int64_t>(output),
 785:             static_cast<GridSamplerInterpolation>(interpolation_mode),
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 786-792
```cpp
 786:             static_cast<GridSamplerPadding>(padding_mode),
 787:             align_corners);
 788:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 789:       }
 790:     });
 791:   }
 792: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 794-815
```cpp
 794: void launch_grid_sampler_3d_forward_kernel(
 795:     const TensorBase &output, const TensorBase &input, const TensorBase &grid,
 796:     int64_t interpolation_mode, int64_t padding_mode, bool align_corners) {
 797:   // See NOTE [ grid_sampler Native Functions ].
 798:   // Add checks here in case this is called instead of grid_sampler.
 799:   check_grid_sampler_common(input, grid);
 800:   check_grid_sampler_3d(input, grid, interpolation_mode);
 801: 
 802:   auto N = input.size(0);
 803:   auto D = grid.size(1);
 804:   auto H = grid.size(2);
 805:   auto W = grid.size(3);
 806:   int64_t count = N * D * H * W;
 807:   if (count > 0) {
 808:     AT_DISPATCH_FLOATING_TYPES_AND2(
 809:       ScalarType::Half, ScalarType::BFloat16,
 810:       input.scalar_type(), "grid_sampler_3d_cuda", [&] {
 811:       if (canUse32BitIndexMath(input) && canUse32BitIndexMath(grid) &&
 812:           canUse32BitIndexMath(output)) {
 813:         grid_sampler_3d_kernel<scalar_t>
 814:           <<<GET_BLOCKS(count, 512), 512, 0, at::cuda::getCurrentCUDAStream()>>>(
 815:             static_cast<int>(count),
```
- EN: This block defines or continues the implementation of `launch_grid_sampler_3d_forward_kernel`.
- CN: 该代码块定义或继续实现 `launch_grid_sampler_3d_forward_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 816-829
```cpp
 816:             getTensorInfo<const scalar_t, int>(input),
 817:             getTensorInfo<const scalar_t, int>(grid),
 818:             getTensorInfo<scalar_t, int>(output),
 819:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 820:             static_cast<GridSamplerPadding>(padding_mode),
 821:             align_corners);
 822:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 823:       } else {
 824:         grid_sampler_3d_kernel<scalar_t>
 825:           <<<GET_BLOCKS(count, 512), 512, 0, at::cuda::getCurrentCUDAStream()>>>(
 826:             count,
 827:             getTensorInfo<const scalar_t, int64_t>(input),
 828:             getTensorInfo<const scalar_t, int64_t>(grid),
 829:             getTensorInfo<scalar_t, int64_t>(output),
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 830-837
```cpp
 830:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 831:             static_cast<GridSamplerPadding>(padding_mode),
 832:             align_corners);
 833:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 834:       }
 835:     });
 836:   }
 837: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 839-859
```cpp
 839: void launch_grid_sampler_2d_backward_kernel(
 840:     const TensorBase &grad_input, const TensorBase &grad_grid,
 841:     const TensorBase &grad_output, const TensorBase &input,
 842:     const TensorBase &grid, int64_t interpolation_mode, int64_t padding_mode,
 843:     bool align_corners, std::array<bool,2> output_mask) {
 844:   // See NOTE [ grid_sampler Native Functions ].
 845:   // Add checks here in case this is called instead of grid_sampler.
 846:   check_grid_sampler_common(input, grid);
 847:   check_grid_sampler_2d(input, grid);
 848: 
 849:   // See Note [Writing Nondeterministic Operations]
 850:   // Nondeterministic because of atomicAdd usage
 851:   globalContext().alertNotDeterministic("grid_sampler_2d_backward_cuda");
 852:   auto N = input.size(0);
 853:   auto H = grid.size(1);
 854:   auto W = grid.size(2);
 855: 
 856:   // If `input` gradient is not required, we skip computing it -- not needing to create
 857:   // the tensor to hold the gradient can markedly increase performance. (`grid` gradient
 858:   // is always computed.)
 859:   auto input_requires_grad = output_mask[0];
```
- EN: This block defines or continues the implementation of `launch_grid_sampler_2d_backward_kernel`.
- CN: 该代码块定义或继续实现 `launch_grid_sampler_2d_backward_kernel`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 861-882
```cpp
 861:   int64_t count = N * H * W;
 862:   if (count > 0) {
 863:     AT_DISPATCH_FLOATING_TYPES_AND2(
 864:       ScalarType::Half, ScalarType::BFloat16,
 865:       input.scalar_type(), "grid_sampler_2d_backward_cuda", [&] {
 866:       if (canUse32BitIndexMath(input) && canUse32BitIndexMath(grid) &&
 867:           canUse32BitIndexMath(grad_output)) {
 868:         grid_sampler_2d_backward_kernel<scalar_t>
 869:           <<<GET_BLOCKS(count, 256), 256, 0, at::cuda::getCurrentCUDAStream()>>>(
 870:             static_cast<int>(count),
 871:             getTensorInfo<const scalar_t, int>(grad_output),
 872:             getTensorInfo<const scalar_t, int>(input),
 873:             getTensorInfo<const scalar_t, int>(grid),
 874:             input_requires_grad ? getTensorInfo<scalar_t, int>(grad_input) : TensorInfo<scalar_t, int>(),
 875:             getTensorInfo<scalar_t, int>(grad_grid),
 876:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 877:             static_cast<GridSamplerPadding>(padding_mode),
 878:             align_corners,
 879:             /*grad_input_memory_span =*/input_requires_grad ? static_cast<int>(grad_input.numel()) : 0,
 880:             input_requires_grad);
 881:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 882:       } else {
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 883-896
```cpp
 883:         grid_sampler_2d_backward_kernel<scalar_t>
 884:           <<<GET_BLOCKS(count, 256), 256, 0, at::cuda::getCurrentCUDAStream()>>>(
 885:             count,
 886:             getTensorInfo<const scalar_t, int64_t>(grad_output),
 887:             getTensorInfo<const scalar_t, int64_t>(input),
 888:             getTensorInfo<const scalar_t, int64_t>(grid),
 889:             input_requires_grad ? getTensorInfo<scalar_t, int64_t>(grad_input) : TensorInfo<scalar_t, int64_t>(),
 890:             getTensorInfo<scalar_t, int64_t>(grad_grid),
 891:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 892:             static_cast<GridSamplerPadding>(padding_mode),
 893:             align_corners,
 894:             /*grad_input_memory_span =*/input_requires_grad ? grad_input.numel() : 0,
 895:             input_requires_grad);
 896:         C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 897-900
```cpp
 897:       }
 898:     });
 899:   }
 900: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 902-923
```cpp
 902: void launch_grid_sampler_3d_backward_kernel(
 903:     const TensorBase &grad_input, const TensorBase &grad_grid,
 904:     const TensorBase& grad_output, const TensorBase& input,
 905:     const TensorBase& grid, int64_t interpolation_mode, int64_t padding_mode,
 906:     bool align_corners, std::array<bool,2> output_mask) {
 907:   // See NOTE [ grid_sampler Native Functions ].
 908:   // Add checks here in case this is called instead of grid_sampler.
 909:   check_grid_sampler_common(input, grid);
 910:   check_grid_sampler_3d(input, grid, interpolation_mode);
 911: 
 912:   // See Note [Writing Nondeterministic Operations]
 913:   // Nondeterministic because of atomicAdd usage
 914:   globalContext().alertNotDeterministic("grid_sampler_3d_backward_cuda");
 915:   auto N = input.size(0);
 916:   auto D = grid.size(1);
 917:   auto H = grid.size(2);
 918:   auto W = grid.size(3);
 919:   int64_t count = N * D * H * W;
 920:   auto input_requires_grad = output_mask[0];
 921:   if (count > 0) {
 922:     AT_DISPATCH_FLOATING_TYPES_AND2(
 923:       ScalarType::Half, ScalarType::BFloat16,
```
- EN: This block defines or continues the implementation of `launch_grid_sampler_3d_backward_kernel`.
- CN: 该代码块定义或继续实现 `launch_grid_sampler_3d_backward_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 924-945
```cpp
 924:       input.scalar_type(), "grid_sampler_3d_backward_cuda", [&] {
 925:       if (canUse32BitIndexMath(input) && canUse32BitIndexMath(grid) &&
 926:           canUse32BitIndexMath(grad_output)) {
 927:         grid_sampler_3d_backward_kernel<scalar_t>
 928:           <<<GET_BLOCKS(count, 256), 256, 0, at::cuda::getCurrentCUDAStream()>>>(
 929:             static_cast<int>(count),
 930:             getTensorInfo<const scalar_t, int>(grad_output),
 931:             getTensorInfo<const scalar_t, int>(input),
 932:             getTensorInfo<const scalar_t, int>(grid),
 933:             input_requires_grad ? getTensorInfo<scalar_t, int>(grad_input) : TensorInfo<scalar_t, int>(),
 934:             getTensorInfo<scalar_t, int>(grad_grid),
 935:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 936:             static_cast<GridSamplerPadding>(padding_mode),
 937:             align_corners,
 938:             /*grad_input_memory_span =*/input_requires_grad ? static_cast<int>(grad_input.numel()) : 0,
 939:             input_requires_grad);
 940:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 941:       } else {
 942:         grid_sampler_3d_backward_kernel<scalar_t>
 943:           <<<GET_BLOCKS(count, 256), 256, 0, at::cuda::getCurrentCUDAStream()>>>(
 944:             count,
 945:             getTensorInfo<const scalar_t, int64_t>(grad_output),
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 946-959
```cpp
 946:             getTensorInfo<const scalar_t, int64_t>(input),
 947:             getTensorInfo<const scalar_t, int64_t>(grid),
 948:             input_requires_grad ? getTensorInfo<scalar_t, int64_t>(grad_input) : TensorInfo<scalar_t, int64_t>(),
 949:             getTensorInfo<scalar_t, int64_t>(grad_grid),
 950:             static_cast<GridSamplerInterpolation>(interpolation_mode),
 951:             static_cast<GridSamplerPadding>(padding_mode),
 952:             align_corners,
 953:             /*grad_input_memory_span =*/input_requires_grad ? grad_input.numel() : 0,
 954:             input_requires_grad);
 955:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 956:       }
 957:     });
 958:   }
 959: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 961-961
```cpp
 961: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/OpMathType.h>`
  - `<ATen/native/cuda/GridSampler.h>`
  - `<ATen/native/GridSamplerUtils.h>`
  - `<ATen/native/cuda/GridSampler.cuh>`
  - `<ATen/native/cuda/UpSample.cuh>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/detail/TensorInfo.cuh>`
  - `<ATen/cuda/detail/IndexUtils.cuh>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/core/TensorBase.h>`
  - `<ATen/Dispatch.h>`
  - `<c10/macros/Macros.h>`
- Runtime symbols / 运行时符号:
  - `launch_grid_sampler_2d_forward_kernel`
  - `launch_grid_sampler_3d_forward_kernel`
  - `launch_grid_sampler_2d_backward_kernel`
  - `launch_grid_sampler_3d_backward_kernel`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `at::cuda::detail`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。

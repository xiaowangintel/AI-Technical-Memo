# GridSampler.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/GridSampler.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `grid_sampler_unnormalize`, `grid_sampler_unnormalize_set_grad`, `clip_coordinates`, `clip_coordinates_set_grad`.
- 用途（中文）: 声明或定义与 `grid_sampler_unnormalize`, `grid_sampler_unnormalize_set_grad`, `clip_coordinates`, `clip_coordinates_set_grad` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```cpp
   1: #pragma once
   2: #include <ATen/native/cuda/KernelUtils.cuh>
   3: #include <ATen/native/GridSamplerUtils.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/KernelUtils.cuh>`, `<ATen/native/GridSamplerUtils.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/KernelUtils.cuh>`, `<ATen/native/GridSamplerUtils.h>`。

### Lines 5-26
```cpp
   5: namespace at::native {
   6: 
   7: using detail::GridSamplerInterpolation;
   8: using detail::GridSamplerPadding;
   9: 
  10: // Unnormalizes a coordinate from the -1 to +1 scale to its pixel index value,
  11: // where we view each pixel as an area between (idx - 0.5) and (idx + 0.5).
  12: // if align_corners: -1 and +1 get sent to the centers of the corner pixels
  13: //     -1 --> 0
  14: //     +1 --> (size - 1)
  15: //     scale_factor = (size - 1) / 2
  16: // if not align_corners: -1 and +1 get sent to the image edges
  17: //     -1 --> -0.5
  18: //     +1 --> (size - 1) + 0.5 == size - 0.5
  19: //     scale_factor = size / 2
  20: template <typename scalar_t>
  21: __forceinline__ __device__
  22: scalar_t grid_sampler_unnormalize(scalar_t coord, int size, bool align_corners) {
  23:   if (align_corners) {
  24:     // unnormalize coord from [-1, 1] to [0, size - 1]
  25:     return ((coord + 1.f) / 2) * (size - 1);
  26:   } else {
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `between`.
- CN: 该代码块定义或继续实现 `between`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 27-27
```cpp
  27:     // unnormalize coord from [-1, 1] to [-0.5, size - 0.5]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 28-30
```cpp
  28:     return ((coord + 1.f) * size - 1) / 2;
  29:   }
  30: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 32-35
```cpp
  32: // grid_sampler_unnormalize_set_grad works the same as grid_sampler_unnormalize
  33: // except that it also returns the `d output / d input` via pointer argument
  34: // `grad_in`.
  35: // This is useful in the backward pass of grid_sampler.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 36-49
```cpp
  36: template <typename scalar_t>
  37: __forceinline__ __device__
  38: scalar_t grid_sampler_unnormalize_set_grad(scalar_t coord, int size,
  39:                                            bool align_corners, scalar_t *grad_in) {
  40:   if (align_corners) {
  41:     // unnormalize coord from [-1, 1] to [0, size - 1]
  42:     *grad_in = static_cast<scalar_t>(size - 1) / 2;
  43:     return ((coord + 1.f) / 2) * (size - 1);
  44:   } else {
  45:     // unnormalize coord from [-1, 1] to [-0.5, size - 0.5]
  46:     *grad_in = static_cast<scalar_t>(size) / 2;
  47:     return ((coord + 1.f) * size - 1) / 2;
  48:   }
  49: }
```
- EN: This block defines or continues the implementation of `grid_sampler_unnormalize_set_grad`.
- CN: 该代码块定义或继续实现 `grid_sampler_unnormalize_set_grad`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 51-51
```cpp
  51: // Clips coordinates to between 0 and clip_limit - 1
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 52-56
```cpp
  52: template <typename scalar_t>
  53: __forceinline__ __device__
  54: scalar_t clip_coordinates(scalar_t in, int clip_limit) {
  55:   return ::min(static_cast<scalar_t>(clip_limit - 1), ::max(in, static_cast<scalar_t>(0)));
  56: }
```
- EN: This block defines or continues the implementation of `clip_coordinates`.
- CN: 该代码块定义或继续实现 `clip_coordinates`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 58-60
```cpp
  58: // clip_coordinates_set_grad works similarly to clip_coordinates except that
  59: // it also returns the `d output / d input` via pointer argument `grad_in`.
  60: // This is useful in the backward pass of grid_sampler.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 61-79
```cpp
  61: template <typename scalar_t>
  62: __forceinline__ __device__
  63: scalar_t clip_coordinates_set_grad(scalar_t in, int clip_limit, scalar_t *grad_in) {
  64:   // Note that it is important for the gradient calculation that borders
  65:   // are considered out of bounds.
  66:   if (in <= static_cast<scalar_t>(0)) {
  67:     *grad_in = static_cast<scalar_t>(0);
  68:     return static_cast<scalar_t>(0);
  69:   } else {
  70:     scalar_t max = static_cast<scalar_t>(clip_limit - 1);
  71:     if (in >= max) {
  72:       *grad_in = static_cast<scalar_t>(0);
  73:       return max;
  74:     } else {
  75:       *grad_in = static_cast<scalar_t>(1);
  76:       return in;
  77:     }
  78:   }
  79: }
```
- EN: This block defines or continues the implementation of `clip_coordinates_set_grad`.
- CN: 该代码块定义或继续实现 `clip_coordinates_set_grad`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 81-83
```cpp
  81: // Reflects coordinates until they fall between low and high (inclusive).
  82: // The bounds are passed as twice their value so that half-integer values
  83: // can be represented as ints.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 84-101
```cpp
  84: template <typename scalar_t>
  85: __forceinline__ __device__
  86: scalar_t reflect_coordinates(scalar_t in, int twice_low, int twice_high) {
  87:   if (twice_low == twice_high) {
  88:     return static_cast<scalar_t>(0);
  89:   }
  90:   scalar_t min = static_cast<scalar_t>(twice_low) / 2;
  91:   scalar_t span = static_cast<scalar_t>(twice_high - twice_low) / 2;
  92:   in = ::fabs(in - min);
  93:   // `fmod` returns same sign as `in`, which is positive after the `fabs` above.
  94:   scalar_t extra = ::fmod(in, span);
  95:   int flips = static_cast<int>(::floor(in / span));
  96:   if (flips % 2 == 0) {
  97:     return extra + min;
  98:   } else {
  99:     return span - extra + min;
 100:   }
 101: }
```
- EN: This block defines or continues the implementation of `reflect_coordinates`.
- CN: 该代码块定义或继续实现 `reflect_coordinates`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 103-106
```cpp
 103: // reflect_coordinates_set_grad works similarly to reflect_coordinates except
 104: // that it also returns the `d output / d input` via pointer argument
 105: // `grad_in`.
 106: // This is useful in the backward pass of grid_sampler.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 107-128
```cpp
 107: template <typename scalar_t>
 108: __forceinline__ __device__
 109: scalar_t reflect_coordinates_set_grad(scalar_t in, int twice_low, int twice_high,
 110:                                       scalar_t *grad_in) {
 111:   if (twice_low == twice_high) {
 112:     *grad_in = static_cast<scalar_t>(0);
 113:     return static_cast<scalar_t>(0);
 114:   }
 115:   int grad_in_mult_;
 116:   scalar_t min = static_cast<scalar_t>(twice_low) / 2;
 117:   scalar_t span = static_cast<scalar_t>(twice_high - twice_low) / 2;
 118:   in = in - min;
 119:   if (in < static_cast<scalar_t>(0)) {
 120:     grad_in_mult_ = -1;
 121:     in = -in;
 122:   } else {
 123:     grad_in_mult_ = 1;
 124:   }
 125:   // `fmod` returns same sign as `in`, which is positive after the `if` above.
 126:   scalar_t extra = ::fmod(in, span);
 127:   int flips = static_cast<int>(::floor(in / span));
 128:   if (flips % 2 == 0) {
```
- EN: This block defines or continues the implementation of `reflect_coordinates_set_grad`.
- CN: 该代码块定义或继续实现 `reflect_coordinates_set_grad`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 129-129
```cpp
 129:     *grad_in = static_cast<scalar_t>(grad_in_mult_);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 130-135
```cpp
 130:     return extra + min;
 131:   } else {
 132:     *grad_in = static_cast<scalar_t>(-grad_in_mult_);
 133:     return span - extra + min;
 134:   }
 135: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 137-146
```cpp
 137: template<typename scalar_t>
 138: __forceinline__ __device__
 139: scalar_t safe_downgrade_to_int_range(scalar_t x){
 140:   // -100.0 does not have special meaning. This is just to make sure
 141:   // it's not within_bounds_2d or within_bounds_3d, and does not cause
 142:   // undefined behavior. See #35506.
 143:   if (x > INT_MAX-1 || x < INT_MIN || !::isfinite(static_cast<double>(x)))
 144:     return static_cast<scalar_t>(-100.0);
 145:   return x;
 146: }
```
- EN: This block defines or continues the implementation of `safe_downgrade_to_int_range`.
- CN: 该代码块定义或继续实现 `safe_downgrade_to_int_range`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 148-169
```cpp
 148: template<typename scalar_t>
 149: __forceinline__ __device__
 150: scalar_t compute_coordinates(scalar_t coord, int size,
 151:                              GridSamplerPadding padding_mode,
 152:                              bool align_corners) {
 153:   if (padding_mode == GridSamplerPadding::Border) {
 154:     // clip coordinates to image borders
 155:     coord = clip_coordinates(coord, size);
 156:   } else if (padding_mode == GridSamplerPadding::Reflection) {
 157:     // reflect coordinates by image borders
 158:     if (align_corners) {
 159:       coord = reflect_coordinates(coord, 0, 2*(size - 1));
 160:     } else {
 161:       coord = reflect_coordinates(coord, -1, 2*size - 1);
 162:     }
 163:     // clip coordinates to image borders
 164:     coord = clip_coordinates(coord, size);
 165:   }
 166: 
 167:   coord = safe_downgrade_to_int_range(coord);
 168:   return coord;
 169: }
```
- EN: This block defines or continues the implementation of `compute_coordinates`.
- CN: 该代码块定义或继续实现 `compute_coordinates`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 171-171
```cpp
 171: // Computes the pixel source index value for a grid coordinate
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 172-182
```cpp
 172: template <typename scalar_t>
 173: __forceinline__ __device__
 174: scalar_t grid_sampler_compute_source_index(
 175:     scalar_t coord,
 176:     int size,
 177:     GridSamplerPadding padding_mode,
 178:     bool align_corners) {
 179:   coord = grid_sampler_unnormalize(coord, size, align_corners);
 180:   coord = compute_coordinates(coord, size, padding_mode, align_corners);
 181:   return coord;
 182: }
```
- EN: This block defines or continues the implementation of `grid_sampler_compute_source_index`.
- CN: 该代码块定义或继续实现 `grid_sampler_compute_source_index`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 184-187
```cpp
 184: // grid_sampler_compute_source_index_set_grad works similarly to
 185: // grid_sampler_compute_source_index except that it also returns the
 186: // `d output / d input` via pointer argument `grad_in`.
 187: // This is useful in the backward pass of grid_sampler.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 188-209
```cpp
 188: template <typename scalar_t>
 189: __forceinline__ __device__
 190: scalar_t grid_sampler_compute_source_index_set_grad(
 191:     scalar_t coord,
 192:     int size,
 193:     GridSamplerPadding padding_mode,
 194:     bool align_corners,
 195:     scalar_t *grad_in) {
 196:   scalar_t grad_clip, grad_refl;
 197:   coord = grid_sampler_unnormalize_set_grad(coord, size, align_corners, grad_in);
 198:   if (padding_mode == GridSamplerPadding::Border) {
 199:     // clip coordinates to image borders
 200:     coord = clip_coordinates_set_grad(coord, size, &grad_clip);
 201:     *grad_in = (*grad_in) * grad_clip;
 202:   } else if (padding_mode == GridSamplerPadding::Reflection) {
 203:     // reflect coordinates by image borders
 204:     if (align_corners) {
 205:       coord = reflect_coordinates_set_grad(coord, 0, 2*(size - 1), &grad_refl);
 206:     } else {
 207:       coord = reflect_coordinates_set_grad(coord, -1, 2*size - 1, &grad_refl);
 208:     }
 209:     // clip coordinates to image borders
```
- EN: This block defines or continues the implementation of `grid_sampler_compute_source_index_set_grad`.
- CN: 该代码块定义或继续实现 `grid_sampler_compute_source_index_set_grad`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 210-212
```cpp
 210:     coord = clip_coordinates_set_grad(coord, size, &grad_clip);
 211:     *grad_in = (*grad_in) * grad_refl * grad_clip;
 212:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 214-216
```cpp
 214:   coord = safe_downgrade_to_int_range(coord);
 215:   return coord;
 216: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 218-221
```cpp
 218: __forceinline__ __device__
 219: bool within_bounds_2d(int h, int w, int H, int W) {
 220:   return h >= 0 && h < H && w >= 0 && w < W;
 221: }
```
- EN: This block defines or continues the implementation of `within_bounds_2d`.
- CN: 该代码块定义或继续实现 `within_bounds_2d`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 223-226
```cpp
 223: __forceinline__ __device__
 224: bool within_bounds_3d(int d, int h, int w, int D, int H, int W) {
 225:   return d >= 0 && d < D && h >= 0 && h < H && w >= 0 && w < W;
 226: }
```
- EN: This block defines or continues the implementation of `within_bounds_3d`.
- CN: 该代码块定义或继续实现 `within_bounds_3d`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 228-245
```cpp
 228: template<typename scalar_t>
 229: __forceinline__ __device__
 230: scalar_t get_value_bounded(
 231:     const scalar_t *data, scalar_t x, scalar_t y, int W, int H, int sW, int sH,
 232:     GridSamplerPadding padding_mode,
 233:     bool align_corners) {
 234: 
 235:   x = compute_coordinates(x, W, padding_mode, align_corners);
 236:   y = compute_coordinates(y, H, padding_mode, align_corners);
 237: 
 238:   int ix = static_cast<int>(x);
 239:   int iy = static_cast<int>(y);
 240: 
 241:   if (within_bounds_2d(iy, ix, H, W)) {
 242:     return data[iy * sH + ix * sW];
 243:   }
 244:   return static_cast<scalar_t>(0);
 245: }
```
- EN: This block defines or continues the implementation of `get_value_bounded`.
- CN: 该代码块定义或继续实现 `get_value_bounded`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 247-261
```cpp
 247: template<typename scalar_t, typename index_t>
 248: __forceinline__ __device__
 249: void safe_add_2d(scalar_t *data, int h, int w,
 250:                  int sH, int sW, int H, int W,
 251:                  scalar_t delta,
 252:                  const index_t NC_offset,
 253:                  const index_t memory_span) {
 254:   if (within_bounds_2d(h, w, H, W)) {
 255:     fastAtomicAdd(data,
 256:                   NC_offset + h * sH + w * sW,
 257:                   memory_span,
 258:                   delta,
 259:                   true);
 260:   }
 261: }
```
- EN: This block defines or continues the implementation of `safe_add_2d`.
- CN: 该代码块定义或继续实现 `safe_add_2d`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 263-277
```cpp
 263: template<typename scalar_t, typename index_t>
 264: __forceinline__ __device__
 265: void safe_add_3d(scalar_t *data, int d, int h, int w,
 266:                  int sD, int sH, int sW, int D, int H, int W,
 267:                  scalar_t delta,
 268:                  const index_t NC_offset,
 269:                  const index_t memory_span) {
 270:   if (within_bounds_3d(d, h, w, D, H, W)) {
 271:     fastAtomicAdd(data,
 272:                   NC_offset + d * sD + h * sH + w * sW,
 273:                   memory_span,
 274:                   delta,
 275:                   true);
 276:   }
 277: }
```
- EN: This block defines or continues the implementation of `safe_add_3d`.
- CN: 该代码块定义或继续实现 `safe_add_3d`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 279-296
```cpp
 279: template<typename scalar_t, typename index_t>
 280: __forceinline__ __device__
 281: void add_value_bounded(
 282:     scalar_t* data, scalar_t x, scalar_t y, int W, int H, int sW, int sH,
 283:     scalar_t delta,
 284:     GridSamplerPadding padding_mode,
 285:     bool align_corners,
 286:     const index_t NC_offset,
 287:     const index_t memory_span) {
 288: 
 289:   x = compute_coordinates(x, W, padding_mode, align_corners);
 290:   y = compute_coordinates(y, H, padding_mode, align_corners);
 291: 
 292:   int ix = static_cast<int>(x);
 293:   int iy = static_cast<int>(y);
 294: 
 295:   safe_add_2d(data, iy, ix, sH, sW, H, W, delta, NC_offset, memory_span);
 296: }
```
- EN: This block defines or continues the implementation of `add_value_bounded`.
- CN: 该代码块定义或继续实现 `add_value_bounded`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 298-298
```cpp
 298: // Calculate the differential of the cubic convolution, i.e. `d coeff / d x`
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 299-318
```cpp
 299: template<typename scalar_t>
 300: __forceinline__ __device__
 301: void get_cubic_coefficients_grad(
 302:     scalar_t coeffs[4],
 303:     scalar_t t) {
 304: 
 305:   // Must be the same as forward calculation in
 306:   // aten/src/ATen/native/cuda/UpSample.cuh:get_cubic_upsample_coefficients
 307:   scalar_t A = -0.75;
 308: 
 309:   scalar_t x;
 310:   x = -1 - t;  // 1 < x = |-1 - tx| < 2
 311:   coeffs[0] = (-3 * A * x - 10 * A ) * x - 8 * A;
 312:   x = -t;     // x = |0 - tx| <= 1
 313:   coeffs[1] = (-3 * (A + 2) * x - 2 * (A + 3)) * x;
 314:   x = 1 - t;  // x = |1 - tx| <= 1
 315:   coeffs[2] = (3 * (A + 2) * x - 2 * (A + 3)) * x;
 316:   x = 2 - t;  // 1 < x = |2 - tx| < 2
 317:   coeffs[3] = (3 * A * x - 10 * A) * x + 8 * A;
 318: }
```
- EN: This block defines or continues the implementation of `get_cubic_coefficients_grad`.
- CN: 该代码块定义或继续实现 `get_cubic_coefficients_grad`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 321-321
```cpp
 321: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/cuda/KernelUtils.cuh>`
  - `<ATen/native/GridSamplerUtils.h>`
- Runtime symbols / 运行时符号: no obvious helper symbols were extracted; dependencies are mostly local or implicit / 未提取到明显辅助符号，依赖主要是局部实现或隐式机制。

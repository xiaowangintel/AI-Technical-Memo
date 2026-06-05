# GridSampler.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/GridSampler.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Grid Sampler. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 grid、sampler 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <algorithm>
0004: #include <cmath>
0005: #include <cstdint>
0006: #include <utility>
0007: 
0008: #include <ATen/native/GridSamplerUtils.h>
0009: 
0010: namespace at::native {
0011: 
0012: using detail::GridSamplerInterpolation;
0013: using detail::GridSamplerPadding;
0014: 
0015: // Unnormalizes a coordinate from the -1 to +1 scale to its pixel index value,
0016: // where we view each pixel as an area between (idx - 0.5) and (idx + 0.5).
0017: // if align_corners: -1 and +1 get sent to the centers of the corner pixels
0018: //     -1 --> 0
0019: //     +1 --> (size - 1)
0020: //     scale_factor = (size - 1) / 2
0021: // if not align_corners: -1 and +1 get sent to the image edges
0022: //     -1 --> -0.5
0023: //     +1 --> (size - 1) + 0.5 == size - 0.5
0024: //     scale_factor = size / 2
0025: template <typename scalar_t>
0026: static inline scalar_t grid_sampler_unnormalize(scalar_t coord, int64_t size,
0027:                                                 bool align_corners) {
0028:   if (align_corners) {
0029:     // unnormalize coord from [-1, 1] to [0, size - 1]
0030:     return ((coord + 1) / 2) * (size - 1);
```
- **EN**: Lines 1-30 mainly cover comments/documentation, header inclusion, state/variable declarations. Notable symbols: between, and, grid_sampler_unnormalize.
- **CN**: 第 1-30 行主要涉及注释或说明、头文件包含、变量/别名声明。 值得关注的符号包括：between, and, grid_sampler_unnormalize。

### Lines 31-60 / 第 31-60 行
```cpp
0031:   } else {
0032:     // unnormalize coord from [-1, 1] to [-0.5, size - 0.5]
0033:     return ((coord + 1) * size - 1) / 2;
0034:   }
0035: }
0036: 
0037: // grid_sampler_unnormalize_set_grad works the same as grid_sampler_unnormalize
0038: // except that it also returns the `d output / d input` via pointer argument
0039: // `grad_in`.
0040: // This is useful in the backward pass of grid_sampler.
0041: template <typename scalar_t>
0042: static inline scalar_t grid_sampler_unnormalize_set_grad(scalar_t coord, int64_t size,
0043:                                                          bool align_corners, scalar_t *grad_in) {
0044:   if (align_corners) {
0045:     // unnormalize coord from [-1, 1] to [0, size - 1]
0046:     *grad_in = static_cast<scalar_t>(size - 1) / 2;
0047:     return ((coord + 1) / 2) * (size - 1);
0048:   } else {
0049:     // unnormalize coord from [-1, 1] to [-0.5, size - 0.5]
0050:     *grad_in = static_cast<scalar_t>(size) / 2;
0051:     return ((coord + 1) * size - 1) / 2;
0052:   }
0053: }
0054: 
0055: // Clips coordinates to between 0 and clip_limit - 1
0056: template<typename scalar_t>
0057: static inline scalar_t clip_coordinates(scalar_t in, int64_t clip_limit) {
0058:   return std::min(static_cast<scalar_t>(clip_limit - 1), std::max(in, static_cast<scalar_t>(0)));
0059: }
0060: 
```
- **EN**: Lines 31-60 mainly cover comments/documentation, expressions/calls, return paths. Notable symbols: grid_sampler_unnormalize_set_grad, clip_coordinates, min, max.
- **CN**: 第 31-60 行主要涉及注释或说明、表达式或调用、返回路径。 值得关注的符号包括：grid_sampler_unnormalize_set_grad, clip_coordinates, min, max。

### Lines 61-90 / 第 61-90 行
```cpp
0061: // clip_coordinates_set_grad works similarly to clip_coordinates except that
0062: // it also returns the `d output / d input` via pointer argument `grad_in`.
0063: // This is useful in the backward pass of grid_sampler.
0064: template<typename scalar_t>
0065: static inline scalar_t clip_coordinates_set_grad(scalar_t in, int64_t clip_limit,
0066:                                                  scalar_t *grad_in) {
0067:   // Note that it is important for the gradient calculation that borders
0068:   // are considered out of bounds.
0069:   if (in <= static_cast<scalar_t>(0)) {
0070:     *grad_in = static_cast<scalar_t>(0);
0071:     return static_cast<scalar_t>(0);
0072:   } else {
0073:     scalar_t max = static_cast<scalar_t>(clip_limit - 1);
0074:     if (in >= max) {
0075:       *grad_in = static_cast<scalar_t>(0);
0076:       return max;
0077:     } else {
0078:       *grad_in = static_cast<scalar_t>(1);
0079:       return in;
0080:     }
0081:   }
0082: }
0083: 
0084: // Reflects coordinates until they fall between low and high (inclusive).
0085: // The bounds are passed as twice their value so that half-integer values
0086: // can be represented as ints.
0087: template<typename scalar_t>
0088: static inline scalar_t reflect_coordinates(scalar_t in, int64_t twice_low,
0089:                                            int64_t twice_high) {
0090:   if (twice_low == twice_high) {
```
- **EN**: Lines 61-90 mainly cover comments/documentation, function signatures/definitions, state/variable declarations. Notable symbols: clip_coordinates_set_grad, high, reflect_coordinates.
- **CN**: 第 61-90 行主要涉及注释或说明、函数签名或实现、变量/别名声明。 值得关注的符号包括：clip_coordinates_set_grad, high, reflect_coordinates。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     return static_cast<scalar_t>(0);
0092:   }
0093:   scalar_t min = static_cast<scalar_t>(twice_low) / 2;
0094:   scalar_t span = static_cast<scalar_t>(twice_high - twice_low) / 2;
0095:   in = std::fabs(in - min);
0096:   // `fmod` returns same sign as `in`, which is positive after the `fabs` above.
0097:   scalar_t extra = std::fmod(in, span);
0098:   int flips = static_cast<int>(std::floor(in / span));
0099:   if (flips % 2 == 0) {
0100:     return extra + min;
0101:   } else {
0102:     return span - extra + min;
0103:   }
0104: }
0105: 
0106: // reflect_coordinates_set_grad works similarly to reflect_coordinates except
0107: // that it also returns the `d output / d input` via pointer argument
0108: // `grad_in`.
0109: // This is useful in the backward pass of grid_sampler.
0110: template<typename scalar_t>
0111: static inline scalar_t reflect_coordinates_set_grad(scalar_t in, int64_t twice_low,
0112:                                                     int64_t twice_high, scalar_t *grad_in) {
0113:   if (twice_low == twice_high) {
0114:     *grad_in = static_cast<scalar_t>(0);
0115:     return static_cast<scalar_t>(0);
0116:   }
0117:   int grad_in_mult_;
0118:   scalar_t min = static_cast<scalar_t>(twice_low) / 2;
0119:   scalar_t span = static_cast<scalar_t>(twice_high - twice_low) / 2;
0120:   in = in - min;
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, return paths. Notable symbols: fabs, fmod, floor, reflect_coordinates_set_grad.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、返回路径。 值得关注的符号包括：fabs, fmod, floor, reflect_coordinates_set_grad。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   if (in < static_cast<scalar_t>(0)) {
0122:     grad_in_mult_ = -1;
0123:     in = -in;
0124:   } else {
0125:     grad_in_mult_ = 1;
0126:   }
0127:   // `fmod` returns same sign as `in`, which is positive after the `if` above.
0128:   scalar_t extra = std::fmod(in, span);
0129:   int flips = static_cast<int>(std::floor(in / span));
0130:   if (flips % 2 == 0) {
0131:     *grad_in = static_cast<scalar_t>(grad_in_mult_);
0132:     return extra + min;
0133:   } else {
0134:     *grad_in = static_cast<scalar_t>(-grad_in_mult_);
0135:     return span - extra + min;
0136:   }
0137: }
0138: 
0139: // Mapping the out-of-boundary points back into boundary
0140: // This would only affect padding_mode=border or reflection
0141: template<typename scalar_t>
0142: static inline scalar_t compute_coordinates(scalar_t coord, int64_t size,
0143:                                            GridSamplerPadding padding_mode,
0144:                                            bool align_corners) {
0145:   if (padding_mode == GridSamplerPadding::Border) {
0146:     // clip coordinates to image borders
0147:     coord = clip_coordinates(coord, size);
0148:   } else if (padding_mode == GridSamplerPadding::Reflection) {
0149:     // reflect coordinates by image borders
0150:     if (align_corners) {
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: fmod, floor, compute_coordinates, clip_coordinates.
- **CN**: 第 121-150 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：fmod, floor, compute_coordinates, clip_coordinates。

### Lines 151-180 / 第 151-180 行
```cpp
0151:       coord = reflect_coordinates(coord, 0, 2*(size - 1));
0152:     } else {
0153:       coord = reflect_coordinates(coord, -1, 2*size - 1);
0154:     }
0155:     // clip coordinates to image borders
0156:     coord = clip_coordinates(coord, size);
0157:   }
0158:   return coord;
0159: }
0160: 
0161: // Computes the pixel source index value for a grid coordinate
0162: template <typename scalar_t>
0163: static inline scalar_t grid_sampler_compute_source_index(
0164:     scalar_t coord,
0165:     int64_t size,
0166:     GridSamplerPadding padding_mode,
0167:     bool align_corners) {
0168:   coord = grid_sampler_unnormalize(coord, size, align_corners);
0169:   coord = compute_coordinates(coord, size, padding_mode, align_corners);
0170:   return coord;
0171: }
0172: 
0173: // grid_sampler_compute_source_index_set_grad works similarly to
0174: // grid_sampler_compute_source_index except that it also returns the
0175: // `d output / d input` via pointer argument `grad_in`.
0176: // This is useful in the backward pass of grid_sampler.
0177: template <typename scalar_t>
0178: static inline scalar_t grid_sampler_compute_source_index_set_grad(
0179:     scalar_t coord,
0180:     int64_t size,
```
- **EN**: Lines 151-180 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: reflect_coordinates, clip_coordinates, grid_sampler_compute_source_index, grid_sampler_unnormalize.
- **CN**: 第 151-180 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：reflect_coordinates, clip_coordinates, grid_sampler_compute_source_index, grid_sampler_unnormalize。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     GridSamplerPadding padding_mode,
0182:     bool align_corners,
0183:     scalar_t *grad_in) {
0184:   scalar_t grad_clip, grad_refl;
0185:   coord = grid_sampler_unnormalize_set_grad(coord, size, align_corners, grad_in);
0186:   if (padding_mode == GridSamplerPadding::Border) {
0187:     // clip coordinates to image borders
0188:     coord = clip_coordinates_set_grad(coord, size, &grad_clip);
0189:     *grad_in = (*grad_in) * grad_clip;
0190:   } else if (padding_mode == GridSamplerPadding::Reflection) {
0191:     // reflect coordinates by image borders
0192:     if (align_corners) {
0193:       coord = reflect_coordinates_set_grad(coord, 0, 2*(size - 1), &grad_refl);
0194:     } else {
0195:       coord = reflect_coordinates_set_grad(coord, -1, 2*size - 1, &grad_refl);
0196:     }
0197:     // clip coordinates to image borders
0198:     coord = clip_coordinates_set_grad(coord, size, &grad_clip);
0199:     *grad_in = (*grad_in) * grad_refl * grad_clip;
0200:   }
0201:   return coord;
0202: }
0203: 
0204: static inline bool within_bounds_2d(int64_t h, int64_t w, int64_t H, int64_t W) {
0205:   return h >= 0 && h < H && w >= 0 && w < W;
0206: }
0207: 
0208: static inline bool within_bounds_3d(int64_t d, int64_t h, int64_t w, int64_t D, int64_t H, int64_t W) {
0209:   return d >= 0 && d < D && h >= 0 && h < H && w >= 0 && w < W;
0210: }
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: grid_sampler_unnormalize_set_grad, clip_coordinates_set_grad, reflect_coordinates_set_grad, within_bounds_2d.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：grid_sampler_unnormalize_set_grad, clip_coordinates_set_grad, reflect_coordinates_set_grad, within_bounds_2d。

### Lines 211-240 / 第 211-240 行
```cpp
0211: 
0212: template<typename scalar_t>
0213: static inline scalar_t get_value_bounded(
0214:     const scalar_t* data,
0215:     scalar_t x,
0216:     scalar_t y,
0217:     int64_t W,
0218:     int64_t H,
0219:     int64_t sW,
0220:     int64_t sH,
0221:     GridSamplerPadding padding_mode,
0222:     bool align_corners) {
0223: 
0224:   x = compute_coordinates(x, W, padding_mode, align_corners);
0225:   y = compute_coordinates(y, H, padding_mode, align_corners);
0226: 
0227:   int64_t ix = static_cast<int64_t>(x);
0228:   int64_t iy = static_cast<int64_t>(y);
0229: 
0230:   if (within_bounds_2d(iy, ix, H, W)) {
0231:     return data[iy * sH + ix * sW];
0232:   }
0233:   return static_cast<scalar_t>(0);
0234: }
0235: 
0236: template<typename scalar_t>
0237: static inline void safe_add_2d(scalar_t *data, int64_t h, int64_t w,
0238:                                int64_t sH, int64_t sW, int64_t H, int64_t W,
0239:                                scalar_t delta) {
0240:   if (within_bounds_2d(h, w, H, W)) {
```
- **EN**: Lines 211-240 mainly cover expressions/calls, state/variable declarations, template setup. Notable symbols: get_value_bounded, compute_coordinates, within_bounds_2d, safe_add_2d.
- **CN**: 第 211-240 行主要涉及表达式或调用、变量/别名声明、模板声明。 值得关注的符号包括：get_value_bounded, compute_coordinates, within_bounds_2d, safe_add_2d。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     data[h * sH + w * sW] += delta;
0242:   }
0243: }
0244: 
0245: template<typename scalar_t>
0246: static inline void safe_add_3d(scalar_t *data, int64_t d, int64_t h, int64_t w,
0247:                                int64_t sD, int64_t sH, int64_t sW,
0248:                                int64_t D, int64_t H, int64_t W,
0249:                                scalar_t delta) {
0250:   if (within_bounds_3d(d, h, w, D, H, W)) {
0251:     data[d * sD + h * sH + w * sW] += delta;
0252:   }
0253: }
0254: 
0255: template<typename scalar_t>
0256: static inline void add_value_bounded(
0257:     scalar_t* data,
0258:     scalar_t x,
0259:     scalar_t y,
0260:     int64_t W,
0261:     int64_t H,
0262:     int64_t sW,
0263:     int64_t sH,
0264:     scalar_t delta,
0265:     GridSamplerPadding padding_mode,
0266:     bool align_corners) {
0267: 
0268:   x = compute_coordinates(x, W, padding_mode, align_corners);
0269:   y = compute_coordinates(y, H, padding_mode, align_corners);
0270: 
```
- **EN**: Lines 241-270 mainly cover expressions/calls, state/variable declarations, template setup. Notable symbols: safe_add_3d, within_bounds_3d, add_value_bounded, compute_coordinates.
- **CN**: 第 241-270 行主要涉及表达式或调用、变量/别名声明、模板声明。 值得关注的符号包括：safe_add_3d, within_bounds_3d, add_value_bounded, compute_coordinates。

### Lines 271-298 / 第 271-298 行
```cpp
0271:   int64_t ix = static_cast<int64_t>(x);
0272:   int64_t iy = static_cast<int64_t>(y);
0273: 
0274:   safe_add_2d(data, iy, ix, sH, sW, H, W, delta);
0275: }
0276: 
0277: // Calculate the differential of the cubic convolution, i.e. `d coeff / d x`
0278: template<typename scalar_t>
0279: static inline void get_cubic_coefficients_grad(
0280:     scalar_t coeffs[4],
0281:     scalar_t t) {
0282: 
0283:   // Must be the same as forward calculation in
0284:   // aten/src/ATen/native/UpSample.h:get_cubic_upsample_coefficients
0285:   scalar_t A = -0.75;
0286: 
0287:   scalar_t x;
0288:   x = -1 - t; // 1 < x = |-1 - tx| < 2
0289:   coeffs[0] = (-3 * A * x - 10 * A ) * x - 8 * A;
0290:   x = -t;     // x = |0 - tx| <= 1
0291:   coeffs[1] = (-3 * (A + 2) * x - 2 * (A + 3)) * x;
0292:   x = 1 - t;  // x = |1 - tx| <= 1
0293:   coeffs[2] = (3 * (A + 2) * x - 2 * (A + 3)) * x;
0294:   x = 2 - t;  // 1 < x = |2 - tx| < 2
0295:   coeffs[3] = (3 * A * x - 10 * A) * x + 8 * A;
0296: }
0297: 
0298: }  // namespace at::native
```
- **EN**: Lines 271-298 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: safe_add_2d, get_cubic_coefficients_grad.
- **CN**: 第 271-298 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：safe_add_2d, get_cubic_coefficients_grad。

## Key Concepts / 关键概念
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<algorithm>`, `<cmath>`, `<cstdint>`, `<utility>`, `<ATen/native/GridSamplerUtils.h>`
- **Macros / 宏**: None highlighted / 无特别标注
- **Namespaces / 命名空间**: `at::native`, `at::`, `std::`

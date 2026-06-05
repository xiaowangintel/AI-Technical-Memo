# GridSampler.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/GridSampler.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Grid Sampler. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 grid、sampler 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/native/GridSampler.h>
0003: #include <ATen/native/GridSamplerUtils.h>
0004: #include <ATen/core/Tensor.h>
0005: #include <ATen/Dispatch.h>
0006: #include <ATen/Parallel.h>
0007: #include <ATen/cpu/vec/vec.h>
0008: #include <ATen/native/UpSample.h>
0009: #include <ATen/native/cpu/GridSamplerKernel.h>
0010: #include <c10/util/Exception.h>
0011: #include <c10/util/irange.h>
0012: 
0013: #ifndef AT_PER_OPERATOR_HEADERS
0014: #include <ATen/Functions.h>
0015: #include <ATen/NativeFunctions.h>
0016: #else
0017: #include <ATen/ops/_empty_affine_quantized.h>
0018: #include <ATen/ops/_grid_sampler_2d_cpu_fallback_backward_native.h>
0019: #include <ATen/ops/_grid_sampler_2d_cpu_fallback_native.h>
0020: #include <ATen/ops/cudnn_grid_sampler.h>
0021: #include <ATen/ops/empty.h>
0022: #include <ATen/ops/empty_like.h>
0023: #include <ATen/ops/grid_sampler_2d.h>
0024: #include <ATen/ops/grid_sampler_2d_backward_native.h>
0025: #include <ATen/ops/grid_sampler_2d_native.h>
0026: #include <ATen/ops/grid_sampler_3d.h>
0027: #include <ATen/ops/grid_sampler_3d_backward_native.h>
0028: #include <ATen/ops/grid_sampler_3d_native.h>
0029: #include <ATen/ops/grid_sampler_native.h>
0030: #include <ATen/ops/zeros_like.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: 
0032: #include <algorithm>
0033: #endif
0034: 
0035: namespace at::native {
0036: 
0037: using at::native::detail::GridSamplerInterpolation;
0038: using at::native::detail::GridSamplerPadding;
0039: 
0040: namespace {
0041: 
0042:   template<typename scalar_t>
0043:   Tensor grid_sampler_3d_cpu_impl(const Tensor& input, const Tensor& grid,
0044:                                   GridSamplerInterpolation interpolation_mode,
0045:                                   GridSamplerPadding padding_mode,
0046:                                   bool align_corners) {
0047:     // See NOTE [ grid_sampler Native Functions ].
0048:     // Add checks here in case this is called instead of grid_sampler.
0049:     check_grid_sampler_common(input, grid);
0050:     check_grid_sampler_3d(
0051:       input, grid, static_cast<int64_t>(interpolation_mode));
0052: 
0053:     int64_t N = input.size(0);
0054:     int64_t C = input.size(1);
0055:     int64_t inp_D = input.size(2);
0056:     int64_t inp_H = input.size(3);
0057:     int64_t inp_W = input.size(4);
0058:     int64_t out_D = grid.size(1);
0059:     int64_t out_H = grid.size(2);
0060:     int64_t out_W = grid.size(3);
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, namespace structuring. Notable symbols: grid_sampler_3d_cpu_impl, check_grid_sampler_common, check_grid_sampler_3d, size.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、命名空间组织。 值得关注的符号包括：grid_sampler_3d_cpu_impl, check_grid_sampler_common, check_grid_sampler_3d, size。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     auto output = at::empty({N, C, out_D, out_H, out_W}, input.options());
0062:     if (output.numel() == 0) {
0063:         return output;
0064:     }
0065:     int64_t inp_sN = input.stride(0);
0066:     int64_t inp_sC = input.stride(1);
0067:     int64_t inp_sD = input.stride(2);
0068:     int64_t inp_sH = input.stride(3);
0069:     int64_t inp_sW = input.stride(4);
0070:     int64_t grid_sN = grid.stride(0);
0071:     int64_t grid_sD = grid.stride(1);
0072:     int64_t grid_sH = grid.stride(2);
0073:     int64_t grid_sW = grid.stride(3);
0074:     int64_t grid_sCoor = grid.stride(4);
0075:     int64_t out_sN = output.stride(0);
0076:     int64_t out_sC = output.stride(1);
0077:     int64_t out_sD = output.stride(2);
0078:     int64_t out_sH = output.stride(3);
0079:     int64_t out_sW = output.stride(4);
0080:     const scalar_t *inp_ptr = input.const_data_ptr<scalar_t>();
0081:     scalar_t *out_ptr = output.data_ptr<scalar_t>();
0082:     const scalar_t *grid_ptr = grid.const_data_ptr<scalar_t>();
0083:     // loop over each output pixel
0084:     at::parallel_for(0, N, 0, [&](int64_t start, int64_t end) {
0085:       for (const auto n : c10::irange(start, end)) {
0086:         const scalar_t *grid_ptr_N = grid_ptr + n * grid_sN;
0087:         const scalar_t *inp_ptr_N = inp_ptr + n * inp_sN;
0088:         for (const auto d : c10::irange(out_D)) {
0089:           for (const auto h : c10::irange(out_H)) {
0090:             for (const auto w : c10::irange(out_W)) {
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, control-flow checks, return paths. Notable symbols: empty, options, numel, stride.
- **CN**: 第 61-90 行主要涉及变量/别名声明、控制流逻辑、返回路径。 值得关注的符号包括：empty, options, numel, stride。

### Lines 91-120 / 第 91-120 行
```cpp
0091:               // get the corresponding input x, y, z coordinates from grid
0092:               const scalar_t *grid_ptr_NDHW = grid_ptr_N + d * grid_sD + h * grid_sH + w * grid_sW;
0093:               scalar_t ix = *grid_ptr_NDHW;
0094:               scalar_t iy = grid_ptr_NDHW[grid_sCoor];
0095:               scalar_t iz = grid_ptr_NDHW[2 * grid_sCoor];
0096: 
0097:               ix = grid_sampler_compute_source_index(ix, inp_W, padding_mode, align_corners);
0098:               iy = grid_sampler_compute_source_index(iy, inp_H, padding_mode, align_corners);
0099:               iz = grid_sampler_compute_source_index(iz, inp_D, padding_mode, align_corners);
0100: 
0101:               if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
0102:                 // get corner pixel values from (x, y, z)
0103:                 // for 4d, we used north-east-south-west
0104:                 // for 5d, we add top-bottom
0105:                 int64_t ix_tnw = static_cast<int64_t>(std::floor(ix));
0106:                 int64_t iy_tnw = static_cast<int64_t>(std::floor(iy));
0107:                 int64_t iz_tnw = static_cast<int64_t>(std::floor(iz));
0108: 
0109:                 int64_t ix_tne = ix_tnw + 1;
0110:                 int64_t iy_tne = iy_tnw;
0111:                 int64_t iz_tne = iz_tnw;
0112: 
0113:                 int64_t ix_tsw = ix_tnw;
0114:                 int64_t iy_tsw = iy_tnw + 1;
0115:                 int64_t iz_tsw = iz_tnw;
0116: 
0117:                 int64_t ix_tse = ix_tnw + 1;
0118:                 int64_t iy_tse = iy_tnw + 1;
0119:                 int64_t iz_tse = iz_tnw;
0120: 
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: grid_sampler_compute_source_index, from, floor.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：grid_sampler_compute_source_index, from, floor。

### Lines 121-150 / 第 121-150 行
```cpp
0121:                 int64_t ix_bnw = ix_tnw;
0122:                 int64_t iy_bnw = iy_tnw;
0123:                 int64_t iz_bnw = iz_tnw + 1;
0124: 
0125:                 int64_t ix_bne = ix_tnw + 1;
0126:                 int64_t iy_bne = iy_tnw;
0127:                 int64_t iz_bne = iz_tnw + 1;
0128: 
0129:                 int64_t ix_bsw = ix_tnw;
0130:                 int64_t iy_bsw = iy_tnw + 1;
0131:                 int64_t iz_bsw = iz_tnw + 1;
0132: 
0133:                 int64_t ix_bse = ix_tnw + 1;
0134:                 int64_t iy_bse = iy_tnw + 1;
0135:                 int64_t iz_bse = iz_tnw + 1;
0136: 
0137:                 // get surfaces to each neighbor:
0138:                 scalar_t tnw = (ix_bse - ix)    * (iy_bse - iy)    * (iz_bse - iz);
0139:                 scalar_t tne = (ix    - ix_bsw) * (iy_bsw - iy)    * (iz_bsw - iz);
0140:                 scalar_t tsw = (ix_bne - ix)    * (iy    - iy_bne) * (iz_bne - iz);
0141:                 scalar_t tse = (ix    - ix_bnw) * (iy    - iy_bnw) * (iz_bnw - iz);
0142:                 scalar_t bnw = (ix_tse - ix)    * (iy_tse - iy)    * (iz - iz_tse);
0143:                 scalar_t bne = (ix    - ix_tsw) * (iy_tsw - iy)    * (iz - iz_tsw);
0144:                 scalar_t bsw = (ix_tne - ix)    * (iy    - iy_tne) * (iz - iz_tne);
0145:                 scalar_t bse = (ix    - ix_tnw) * (iy    - iy_tnw) * (iz - iz_tnw);
0146: 
0147:                 // calculate bilinear weighted pixel value and set output pixel
0148:                 scalar_t *out_ptr_NCDHW = out_ptr + n * out_sN + d * out_sD + h * out_sH + w * out_sW;
0149:                 const scalar_t *inp_ptr_NC = inp_ptr_N;
0150:                 for (int64_t c = 0; c < C; ++c, out_ptr_NCDHW += out_sC, inp_ptr_NC += inp_sC) {
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, comments/documentation, control-flow checks.
- **CN**: 第 121-150 行主要涉及变量/别名声明、注释或说明、控制流逻辑。

### Lines 151-180 / 第 151-180 行
```cpp
0151:                   //   (c, iz_tnw, iy_tnw, ix_tnw) * tnw + (c, iz_tne, iy_tne, ix_tne) * tne
0152:                   // + (c, iz_tsw, iy_tsw, ix_tsw) * tsw + (c, iz_tse, iy_tse, ix_tse) * tse
0153:                   // + (c, iz_bnw, iy_bnw, ix_bnw) * bnw + (c, iz_bne, iy_bne, ix_bne) * bne
0154:                   // + (c, iz_bsw, iy_bsw, ix_bsw) * bsw + (c, iz_bse, iy_bse, ix_bse) * bse
0155:                   *out_ptr_NCDHW = static_cast<scalar_t>(0);
0156:                   if (within_bounds_3d(iz_tnw, iy_tnw, ix_tnw, inp_D, inp_H, inp_W)) {
0157:                     *out_ptr_NCDHW += inp_ptr_NC[iz_tnw * inp_sD + iy_tnw * inp_sH + ix_tnw * inp_sW] * tnw;
0158:                   }
0159:                   if (within_bounds_3d(iz_tne, iy_tne, ix_tne, inp_D, inp_H, inp_W)) {
0160:                     *out_ptr_NCDHW += inp_ptr_NC[iz_tne * inp_sD + iy_tne * inp_sH + ix_tne * inp_sW] * tne;
0161:                   }
0162:                   if (within_bounds_3d(iz_tsw, iy_tsw, ix_tsw, inp_D, inp_H, inp_W)) {
0163:                     *out_ptr_NCDHW += inp_ptr_NC[iz_tsw * inp_sD + iy_tsw * inp_sH + ix_tsw * inp_sW] * tsw;
0164:                   }
0165:                   if (within_bounds_3d(iz_tse, iy_tse, ix_tse, inp_D, inp_H, inp_W)) {
0166:                     *out_ptr_NCDHW += inp_ptr_NC[iz_tse * inp_sD + iy_tse * inp_sH + ix_tse * inp_sW] * tse;
0167:                   }
0168:                   if (within_bounds_3d(iz_bnw, iy_bnw, ix_bnw, inp_D, inp_H, inp_W)) {
0169:                     *out_ptr_NCDHW += inp_ptr_NC[iz_bnw * inp_sD + iy_bnw * inp_sH + ix_bnw * inp_sW] * bnw;
0170:                   }
0171:                   if (within_bounds_3d(iz_bne, iy_bne, ix_bne, inp_D, inp_H, inp_W)) {
0172:                     *out_ptr_NCDHW += inp_ptr_NC[iz_bne * inp_sD + iy_bne * inp_sH + ix_bne * inp_sW] * bne;
0173:                   }
0174:                   if (within_bounds_3d(iz_bsw, iy_bsw, ix_bsw, inp_D, inp_H, inp_W)) {
0175:                     *out_ptr_NCDHW += inp_ptr_NC[iz_bsw * inp_sD + iy_bsw * inp_sH + ix_bsw * inp_sW] * bsw;
0176:                   }
0177:                   if (within_bounds_3d(iz_bse, iy_bse, ix_bse, inp_D, inp_H, inp_W)) {
0178:                     *out_ptr_NCDHW += inp_ptr_NC[iz_bse * inp_sD + iy_bse * inp_sH + ix_bse * inp_sW] * bse;
0179:                   }
0180:                 }
```
- **EN**: Lines 151-180 mainly cover comments/documentation, expressions/calls, control-flow checks. Notable symbols: within_bounds_3d.
- **CN**: 第 151-180 行主要涉及注释或说明、表达式或调用、控制流逻辑。 值得关注的符号包括：within_bounds_3d。

### Lines 181-210 / 第 181-210 行
```cpp
0181:               } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
0182:                 int64_t ix_nearest = static_cast<int64_t>(std::nearbyint(ix));
0183:                 int64_t iy_nearest = static_cast<int64_t>(std::nearbyint(iy));
0184:                 int64_t iz_nearest = static_cast<int64_t>(std::nearbyint(iz));
0185: 
0186:                 // assign nearest neighbour pixel value to output pixel
0187:                 scalar_t *out_ptr_NCDHW = out_ptr + n * out_sN + d * out_sD + h * out_sH + w * out_sW;
0188:                 const scalar_t *inp_ptr_NC = inp_ptr_N;
0189:                 for (int64_t c = 0; c < C; ++c, out_ptr_NCDHW += out_sC, inp_ptr_NC += inp_sC) {
0190:                   if (within_bounds_3d(iz_nearest, iy_nearest, ix_nearest, inp_D, inp_H, inp_W)) {
0191:                     *out_ptr_NCDHW = inp_ptr_NC[iz_nearest * inp_sD + iy_nearest * inp_sH + ix_nearest * inp_sW];
0192:                   } else {
0193:                     *out_ptr_NCDHW = static_cast<scalar_t>(0);
0194:                   }
0195:                 }
0196:               }
0197:             }
0198:           }
0199:         }
0200:       }
0201:     });
0202:     return output;
0203:   }
0204: 
0205:   template<typename scalar_t>
0206:   std::tuple<Tensor, Tensor>
0207:   grid_sampler_3d_backward_cpu_impl(const Tensor& grad_output,
0208:                                     const Tensor& input, const Tensor& grid,
0209:                                     GridSamplerInterpolation interpolation_mode,
0210:                                     GridSamplerPadding padding_mode,
```
- **EN**: Lines 181-210 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: nearbyint, within_bounds_3d, grid_sampler_3d_backward_cpu_impl.
- **CN**: 第 181-210 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：nearbyint, within_bounds_3d, grid_sampler_3d_backward_cpu_impl。

### Lines 211-240 / 第 211-240 行
```cpp
0211:                                     bool align_corners, std::array<bool,2> output_mask) {
0212:     // See NOTE [ grid_sampler Native Functions ].
0213:     // Add checks here in case this is called instead of grid_sampler.
0214:     check_grid_sampler_common(input, grid);
0215:     check_grid_sampler_3d(
0216:       input, grid, static_cast<int64_t>(interpolation_mode));
0217: 
0218:     auto input_requires_grad = output_mask[0];
0219:     Tensor grad_input = ([&]() {
0220:       if (input_requires_grad) {
0221:         return at::zeros_like(input, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0222:       } else {
0223:         return Tensor();
0224:       }
0225:     })();
0226:     auto grad_grid = at::empty_like(grid, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0227:     if (grid.numel() == 0 || input.numel() == 0) {
0228:       grad_grid.zero_();
0229:       return std::make_tuple(grad_input, grad_grid);
0230:     }
0231:     // If interpolation mode is Nearest, then grad_grid is not filled in the
0232:     // loop below.
0233:     if (interpolation_mode == GridSamplerInterpolation::Nearest) {
0234:       grad_grid.zero_();
0235:     }
0236:     int64_t N = input.size(0);
0237:     int64_t C = input.size(1);
0238:     int64_t inp_D = input.size(2);
0239:     int64_t inp_H = input.size(3);
0240:     int64_t inp_W = input.size(4);
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: check_grid_sampler_common, check_grid_sampler_3d, zeros_like, Tensor.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：check_grid_sampler_common, check_grid_sampler_3d, zeros_like, Tensor。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     int64_t out_D = grid.size(1);
0242:     int64_t out_H = grid.size(2);
0243:     int64_t out_W = grid.size(3);
0244:     int64_t inp_sN = input.stride(0);
0245:     int64_t inp_sC = input.stride(1);
0246:     int64_t inp_sD = input.stride(2);
0247:     int64_t inp_sH = input.stride(3);
0248:     int64_t inp_sW = input.stride(4);
0249:     int64_t grid_sN = grid.stride(0);
0250:     int64_t grid_sD = grid.stride(1);
0251:     int64_t grid_sH = grid.stride(2);
0252:     int64_t grid_sW = grid.stride(3);
0253:     int64_t grid_sCoor = grid.stride(4);
0254:     int64_t gOut_sN = grad_output.stride(0);
0255:     int64_t gOut_sC = grad_output.stride(1);
0256:     int64_t gOut_sD = grad_output.stride(2);
0257:     int64_t gOut_sH = grad_output.stride(3);
0258:     int64_t gOut_sW = grad_output.stride(4);
0259:     int64_t gInp_sN = 0;
0260:     int64_t gInp_sC = 0;
0261:     int64_t gInp_sD = 0;
0262:     int64_t gInp_sH = 0;
0263:     int64_t gInp_sW = 0;
0264:     if (input_requires_grad) {
0265:       gInp_sN = grad_input.stride(0);
0266:       gInp_sC = grad_input.stride(1);
0267:       gInp_sD = grad_input.stride(2);
0268:       gInp_sH = grad_input.stride(3);
0269:       gInp_sW = grad_input.stride(4);
0270:     }
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: size, stride.
- **CN**: 第 241-270 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：size, stride。

### Lines 271-300 / 第 271-300 行
```cpp
0271:     int64_t gGrid_sN = grad_grid.stride(0);
0272:     int64_t gGrid_sW = grad_grid.stride(3);
0273:     const scalar_t *inp_ptr = input.const_data_ptr<scalar_t>();
0274:     const scalar_t *grid_ptr = grid.const_data_ptr<scalar_t>();
0275:     const scalar_t *gOut_ptr = grad_output.const_data_ptr<scalar_t>();
0276:     scalar_t *gInp_ptr = nullptr;
0277:     if (input_requires_grad) {
0278:       gInp_ptr = grad_input.mutable_data_ptr<scalar_t>();
0279:     }
0280:     scalar_t *gGrid_ptr = grad_grid.data_ptr<scalar_t>();
0281:     // loop over each output pixel
0282:     at::parallel_for(0, N, 0, [&](int64_t start, int64_t end) {
0283:       for (const auto n : c10::irange(start, end)) {
0284:         const scalar_t *grid_ptr_N = grid_ptr + n * grid_sN;
0285:         const scalar_t *inp_ptr_N = inp_ptr + n * inp_sN;
0286:         scalar_t *gGrid_ptr_NDHW = gGrid_ptr + n * gGrid_sN;
0287:         for (const auto d : c10::irange(out_D)) {
0288:           for (const auto h : c10::irange(out_H)) {
0289:             for (int64_t w = 0; w < out_W; ++w, gGrid_ptr_NDHW += gGrid_sW /* grad_grid is contiguous */ ) {
0290:               // get the corresponding input x, y, z coordinates from grid
0291:               const scalar_t *grid_ptr_NDHW = grid_ptr_N + d * grid_sD + h * grid_sH + w * grid_sW;
0292:               scalar_t ix = *grid_ptr_NDHW;
0293:               scalar_t iy = grid_ptr_NDHW[grid_sCoor];
0294:               scalar_t iz = grid_ptr_NDHW[2 * grid_sCoor];
0295: 
0296:               // multipliers for gradients on ix, iy, and iz
0297:               scalar_t gix_mult, giy_mult, giz_mult;
0298:               ix = grid_sampler_compute_source_index_set_grad(ix, inp_W, padding_mode, align_corners, &gix_mult);
0299:               iy = grid_sampler_compute_source_index_set_grad(iy, inp_H, padding_mode, align_corners, &giy_mult);
0300:               iz = grid_sampler_compute_source_index_set_grad(iz, inp_D, padding_mode, align_corners, &giz_mult);
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: stride, parallel_for, irange, grid_sampler_compute_source_index_set_grad.
- **CN**: 第 271-300 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：stride, parallel_for, irange, grid_sampler_compute_source_index_set_grad。

### Lines 301-330 / 第 301-330 行
```cpp
0301: 
0302:               if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
0303:                 // get corner pixel values from (x, y, z)
0304:                 // for 4d, we used north-east-south-west
0305:                 // for 5d, we add top-bottom
0306:                 int64_t ix_tnw = static_cast<int64_t>(std::floor(ix));
0307:                 int64_t iy_tnw = static_cast<int64_t>(std::floor(iy));
0308:                 int64_t iz_tnw = static_cast<int64_t>(std::floor(iz));
0309: 
0310:                 int64_t ix_tne = ix_tnw + 1;
0311:                 int64_t iy_tne = iy_tnw;
0312:                 int64_t iz_tne = iz_tnw;
0313: 
0314:                 int64_t ix_tsw = ix_tnw;
0315:                 int64_t iy_tsw = iy_tnw + 1;
0316:                 int64_t iz_tsw = iz_tnw;
0317: 
0318:                 int64_t ix_tse = ix_tnw + 1;
0319:                 int64_t iy_tse = iy_tnw + 1;
0320:                 int64_t iz_tse = iz_tnw;
0321: 
0322:                 int64_t ix_bnw = ix_tnw;
0323:                 int64_t iy_bnw = iy_tnw;
0324:                 int64_t iz_bnw = iz_tnw + 1;
0325: 
0326:                 int64_t ix_bne = ix_tnw + 1;
0327:                 int64_t iy_bne = iy_tnw;
0328:                 int64_t iz_bne = iz_tnw + 1;
0329: 
0330:                 int64_t ix_bsw = ix_tnw;
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: from, floor.
- **CN**: 第 301-330 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：from, floor。

### Lines 331-360 / 第 331-360 行
```cpp
0331:                 int64_t iy_bsw = iy_tnw + 1;
0332:                 int64_t iz_bsw = iz_tnw + 1;
0333: 
0334:                 int64_t ix_bse = ix_tnw + 1;
0335:                 int64_t iy_bse = iy_tnw + 1;
0336:                 int64_t iz_bse = iz_tnw + 1;
0337: 
0338:                 // get surfaces to each neighbor:
0339:                 scalar_t tnw = (ix_bse - ix)    * (iy_bse - iy)    * (iz_bse - iz);
0340:                 scalar_t tne = (ix    - ix_bsw) * (iy_bsw - iy)    * (iz_bsw - iz);
0341:                 scalar_t tsw = (ix_bne - ix)    * (iy    - iy_bne) * (iz_bne - iz);
0342:                 scalar_t tse = (ix    - ix_bnw) * (iy    - iy_bnw) * (iz_bnw - iz);
0343:                 scalar_t bnw = (ix_tse - ix)    * (iy_tse - iy)    * (iz - iz_tse);
0344:                 scalar_t bne = (ix    - ix_tsw) * (iy_tsw - iy)    * (iz - iz_tsw);
0345:                 scalar_t bsw = (ix_tne - ix)    * (iy    - iy_tne) * (iz - iz_tne);
0346:                 scalar_t bse = (ix    - ix_tnw) * (iy    - iy_tnw) * (iz - iz_tnw);
0347: 
0348:                 scalar_t gix = static_cast<scalar_t>(0), giy = static_cast<scalar_t>(0), giz = static_cast<scalar_t>(0);
0349:                 const scalar_t *gOut_ptr_NCDHW = gOut_ptr + n * gOut_sN + d * gOut_sD + h * gOut_sH + w * gOut_sW;
0350:                 const scalar_t *inp_ptr_NC = inp_ptr_N;
0351:                 scalar_t *gInp_ptr_NC = gInp_ptr + n * gInp_sN;
0352:                 // calculate bilinear weighted pixel value and set output pixel
0353:                 for (int64_t c = 0; c < C; ++c, gOut_ptr_NCDHW += gOut_sC, gInp_ptr_NC += gInp_sC, inp_ptr_NC += inp_sC) {
0354:                   scalar_t gOut = *gOut_ptr_NCDHW;
0355: 
0356:                   // calculate and set grad_input
0357:                   if (input_requires_grad) {
0358:                     safe_add_3d(gInp_ptr_NC, iz_tnw, iy_tnw, ix_tnw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tnw * gOut);
0359:                     safe_add_3d(gInp_ptr_NC, iz_tne, iy_tne, ix_tne, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tne * gOut);
0360:                     safe_add_3d(gInp_ptr_NC, iz_tsw, iy_tsw, ix_tsw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tsw * gOut);
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: safe_add_3d.
- **CN**: 第 331-360 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：safe_add_3d。

### Lines 361-390 / 第 361-390 行
```cpp
0361:                     safe_add_3d(gInp_ptr_NC, iz_tse, iy_tse, ix_tse, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, tse * gOut);
0362:                     safe_add_3d(gInp_ptr_NC, iz_bnw, iy_bnw, ix_bnw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bnw * gOut);
0363:                     safe_add_3d(gInp_ptr_NC, iz_bne, iy_bne, ix_bne, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bne * gOut);
0364:                     safe_add_3d(gInp_ptr_NC, iz_bsw, iy_bsw, ix_bsw, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bsw * gOut);
0365:                     safe_add_3d(gInp_ptr_NC, iz_bse, iy_bse, ix_bse, gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, bse * gOut);
0366:                   }
0367:                   // calculate grad_grid
0368:                   if (within_bounds_3d(iz_tnw, iy_tnw, ix_tnw, inp_D, inp_H, inp_W)) {
0369:                     scalar_t tnw_val = inp_ptr_NC[iz_tnw * inp_sD + iy_tnw * inp_sH + ix_tnw * inp_sW];
0370:                     gix -= tnw_val * (iy_bse - iy)    * (iz_bse - iz)    * gOut;
0371:                     giy -= tnw_val * (ix_bse - ix)    * (iz_bse - iz)    * gOut;
0372:                     giz -= tnw_val * (ix_bse - ix)    * (iy_bse - iy)    * gOut;
0373:                   }
0374:                   if (within_bounds_3d(iz_tne, iy_tne, ix_tne, inp_D, inp_H, inp_W)) {
0375:                     scalar_t tne_val = inp_ptr_NC[iz_tne * inp_sD + iy_tne * inp_sH + ix_tne * inp_sW];
0376:                     gix += tne_val * (iy_bsw - iy)    * (iz_bsw - iz)    * gOut;
0377:                     giy -= tne_val * (ix    - ix_bsw) * (iz_bsw - iz)    * gOut;
0378:                     giz -= tne_val * (ix    - ix_bsw) * (iy_bsw - iy)    * gOut;
0379:                   }
0380:                   if (within_bounds_3d(iz_tsw, iy_tsw, ix_tsw, inp_D, inp_H, inp_W)) {
0381:                     scalar_t tsw_val = inp_ptr_NC[iz_tsw * inp_sD + iy_tsw * inp_sH + ix_tsw * inp_sW];
0382:                     gix -= tsw_val * (iy - iy_bne)    * (iz_bne - iz)    * gOut;
0383:                     giy += tsw_val * (ix_bne - ix)    * (iz_bne - iz)    * gOut;
0384:                     giz -= tsw_val * (ix_bne - ix)    * (iy    - iy_bne) * gOut;
0385:                   }
0386:                   if (within_bounds_3d(iz_tse, iy_tse, ix_tse, inp_D, inp_H, inp_W)) {
0387:                     scalar_t tse_val = inp_ptr_NC[iz_tse * inp_sD + iy_tse * inp_sH + ix_tse * inp_sW];
0388:                     gix += tse_val * (iy - iy_bnw)    * (iz_bnw - iz)    * gOut;
0389:                     giy += tse_val * (ix    - ix_bnw) * (iz_bnw - iz)    * gOut;
0390:                     giz -= tse_val * (ix    - ix_bnw) * (iy    - iy_bnw) * gOut;
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: safe_add_3d, within_bounds_3d.
- **CN**: 第 361-390 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：safe_add_3d, within_bounds_3d。

### Lines 391-420 / 第 391-420 行
```cpp
0391:                   }
0392:                   if (within_bounds_3d(iz_bnw, iy_bnw, ix_bnw, inp_D, inp_H, inp_W)) {
0393:                     scalar_t bnw_val = inp_ptr_NC[iz_bnw * inp_sD + iy_bnw * inp_sH + ix_bnw * inp_sW];
0394:                     gix -= bnw_val * (iy_tse - iy)    * (iz - iz_tse)    * gOut;
0395:                     giy -= bnw_val * (ix_tse - ix)    * (iz - iz_tse)    * gOut;
0396:                     giz += bnw_val * (ix_tse - ix)    * (iy_tse - iy)    * gOut;
0397:                   }
0398:                   if (within_bounds_3d(iz_bne, iy_bne, ix_bne, inp_D, inp_H, inp_W)) {
0399:                     scalar_t bne_val = inp_ptr_NC[iz_bne * inp_sD + iy_bne * inp_sH + ix_bne * inp_sW];
0400:                     gix += bne_val * (iy_tsw - iy)    * (iz - iz_tsw)    * gOut;
0401:                     giy -= bne_val * (ix    - ix_tsw) * (iz - iz_tsw)    * gOut;
0402:                     giz += bne_val * (ix    - ix_tsw) * (iy_tsw - iy)    * gOut;
0403:                   }
0404:                   if (within_bounds_3d(iz_bsw, iy_bsw, ix_bsw, inp_D, inp_H, inp_W)) {
0405:                     scalar_t bsw_val = inp_ptr_NC[iz_bsw * inp_sD + iy_bsw * inp_sH + ix_bsw * inp_sW];
0406:                     gix -= bsw_val * (iy - iy_tne)    * (iz - iz_tne)    * gOut;
0407:                     giy += bsw_val * (ix_tne - ix)    * (iz - iz_tne)    * gOut;
0408:                     giz += bsw_val * (ix_tne - ix)    * (iy    - iy_tne) * gOut;
0409:                   }
0410:                   if (within_bounds_3d(iz_bse, iy_bse, ix_bse, inp_D, inp_H, inp_W)) {
0411:                     scalar_t bse_val = inp_ptr_NC[iz_bse * inp_sD + iy_bse * inp_sH + ix_bse * inp_sW];
0412:                     gix += bse_val * (iy - iy_tnw)    * (iz - iz_tnw)    * gOut;
0413:                     giy += bse_val * (ix    - ix_tnw) * (iz - iz_tnw)    * gOut;
0414:                     giz += bse_val * (ix    - ix_tnw) * (iy    - iy_tnw) * gOut;
0415:                   }
0416:                 }
0417: 
0418:                 // assuming grad_grid is contiguous
0419:                 gGrid_ptr_NDHW[0] = gix_mult * gix;
0420:                 gGrid_ptr_NDHW[1] = giy_mult * giy;
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: within_bounds_3d.
- **CN**: 第 391-420 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：within_bounds_3d。

### Lines 421-450 / 第 421-450 行
```cpp
0421:                 gGrid_ptr_NDHW[2] = giz_mult * giz;
0422:               } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
0423:                 int64_t ix_nearest = static_cast<int64_t>(std::nearbyint(ix));
0424:                 int64_t iy_nearest = static_cast<int64_t>(std::nearbyint(iy));
0425:                 int64_t iz_nearest = static_cast<int64_t>(std::nearbyint(iz));
0426: 
0427:                 // assign nearest neighbour pixel value to output pixel
0428:                 const scalar_t *gOut_ptr_NCDHW = gOut_ptr + n * gOut_sN + d * gOut_sD + h * gOut_sH + w * gOut_sW;
0429:                 if (input_requires_grad) {
0430:                   scalar_t *gInp_ptr_NC = gInp_ptr + n * gInp_sN;
0431:                   for (int64_t c = 0; c < C; ++c, gOut_ptr_NCDHW += gOut_sC, gInp_ptr_NC += gInp_sC) {
0432:                     // calculate and set grad_input
0433:                     safe_add_3d(gInp_ptr_NC, iz_nearest, iy_nearest, ix_nearest,
0434:                                 gInp_sD, gInp_sH, gInp_sW, inp_D, inp_H, inp_W, *gOut_ptr_NCDHW);
0435:                   }
0436:                 }
0437:               }
0438:             }
0439:           }
0440:         }
0441:       }
0442:     });
0443:     return std::make_tuple(grad_input, grad_grid);
0444:   }
0445: 
0446: }  // namespace
0447: 
0448: static Tensor _grid_sampler_2d_cpu_quantized(
0449:     const Tensor& input,
0450:     const Tensor& grid,
```
- **EN**: Lines 421-450 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: nearbyint, safe_add_3d, make_tuple, _grid_sampler_2d_cpu_quantized.
- **CN**: 第 421-450 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：nearbyint, safe_add_3d, make_tuple, _grid_sampler_2d_cpu_quantized。

### Lines 451-480 / 第 451-480 行
```cpp
0451:     int64_t interpolation_mode_,
0452:     int64_t padding_mode_,
0453:     bool align_corners) {
0454:   // See NOTE [ grid_sampler Native Functions ].
0455:   // Add checks here in case this is called instead of grid_sampler.
0456:   check_grid_sampler_common(input, grid);
0457:   check_grid_sampler_2d(input, grid);
0458: 
0459:   auto interpolation_mode =
0460:       static_cast<GridSamplerInterpolation>(interpolation_mode_);
0461:   /* Bilinear interpolation is supported using the fact that we can perform
0462:    * linear interpolations on quantized values without rescaling. */
0463:   TORCH_CHECK(
0464:       interpolation_mode == GridSamplerInterpolation::Bilinear,
0465:       "_grid_sampler_2d_cpu_quantized(): only bilinear interpolation supported")
0466:   auto padding_mode = static_cast<GridSamplerPadding>(padding_mode_);
0467: 
0468:   int64_t N = input.size(0);
0469:   int64_t C = input.size(1);
0470:   int64_t inp_H = input.size(2);
0471:   int64_t inp_W = input.size(3);
0472:   int64_t out_H = grid.size(1);
0473:   int64_t out_W = grid.size(2);
0474:   uint8_t zero_point = input.q_zero_point();
0475:   auto output = at::_empty_affine_quantized(
0476:       {N, C, out_H, out_W},
0477:       at::device(c10::kCPU).dtype(c10::kQUInt8),
0478:       input.q_scale(),
0479:       zero_point);
0480:   int64_t inp_sN = input.stride(0);
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: check_grid_sampler_common, check_grid_sampler_2d, TORCH_CHECK, _grid_sampler_2d_cpu_quantized.
- **CN**: 第 451-480 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：check_grid_sampler_common, check_grid_sampler_2d, TORCH_CHECK, _grid_sampler_2d_cpu_quantized。

### Lines 481-510 / 第 481-510 行
```cpp
0481:   int64_t inp_sC = input.stride(1);
0482:   int64_t inp_sH = input.stride(2);
0483:   int64_t inp_sW = input.stride(3);
0484:   int64_t grid_sN = grid.stride(0);
0485:   int64_t grid_sH = grid.stride(1);
0486:   int64_t grid_sW = grid.stride(2);
0487:   int64_t grid_sCoor = grid.stride(3);
0488:   int64_t out_sN = output.stride(0);
0489:   int64_t out_sC = output.stride(1);
0490:   int64_t out_sH = output.stride(2);
0491:   int64_t out_sW = output.stride(3);
0492:   const uint8_t* inp_ptr = input.const_data_ptr<uint8_t>();
0493:   uint8_t* out_ptr = output.data_ptr<uint8_t>();
0494:   const float* grid_ptr = grid.const_data_ptr<float>();
0495:   at::parallel_for(0, N, 0, [&](int64_t start, int64_t end) {
0496:     for (const auto n : c10::irange(start, end)) {
0497:       const float* grid_ptr_N = grid_ptr + n * grid_sN;
0498:       const uint8_t* inp_ptr_N = inp_ptr + n * inp_sN;
0499:       for (const auto h : c10::irange(out_H)) {
0500:         for (const auto w : c10::irange(out_W)) {
0501:           // get the corresponding input x, y, z coordinates from grid
0502:           const float* grid_ptr_NHW = grid_ptr_N + h * grid_sH + w * grid_sW;
0503:           float x = *grid_ptr_NHW;
0504:           float y = grid_ptr_NHW[grid_sCoor];
0505: 
0506:           float ix = grid_sampler_compute_source_index(
0507:               x, inp_W, padding_mode, align_corners);
0508:           float iy = grid_sampler_compute_source_index(
0509:               y, inp_H, padding_mode, align_corners);
0510: 
```
- **EN**: Lines 481-510 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: stride, parallel_for, irange, grid_sampler_compute_source_index.
- **CN**: 第 481-510 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：stride, parallel_for, irange, grid_sampler_compute_source_index。

### Lines 511-540 / 第 511-540 行
```cpp
0511:           // get corner pixel values from (x, y)
0512:           // for 4d, we use north-east-south-west
0513:           int64_t ix_nw = static_cast<int64_t>(std::floor(ix));
0514:           int64_t iy_nw = static_cast<int64_t>(std::floor(iy));
0515: 
0516:           int64_t ix_ne = ix_nw + 1;
0517:           int64_t iy_ne = iy_nw;
0518: 
0519:           int64_t ix_sw = ix_nw;
0520:           int64_t iy_sw = iy_nw + 1;
0521: 
0522:           int64_t ix_se = ix_nw + 1;
0523:           int64_t iy_se = iy_nw + 1;
0524: 
0525:           // get surfaces to each neighbor:
0526:           float nw = (ix_se - ix) * (iy_se - iy);
0527:           float ne = (ix - ix_sw) * (iy_sw - iy);
0528:           float sw = (ix_ne - ix) * (iy - iy_ne);
0529:           float se = (ix - ix_nw) * (iy - iy_nw);
0530: 
0531:           // calculate bilinear weighted pixel value and set output pixel
0532:           const uint8_t* inp_ptr_NC = inp_ptr_N;
0533:           uint8_t* out_ptr_NCHW =
0534:               out_ptr + n * out_sN + h * out_sH + w * out_sW;
0535:           for (int64_t c = 0; c < C;
0536:                ++c, out_ptr_NCHW += out_sC, inp_ptr_NC += inp_sC) {
0537:             float res = 0;
0538:             res += within_bounds_2d(iy_nw, ix_nw, inp_H, inp_W)
0539:                 ? inp_ptr_NC[iy_nw * inp_sH + ix_nw * inp_sW] * nw
0540:                 : zero_point * nw;
```
- **EN**: Lines 511-540 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: from, floor, within_bounds_2d.
- **CN**: 第 511-540 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：from, floor, within_bounds_2d。

### Lines 541-570 / 第 541-570 行
```cpp
0541:             res += within_bounds_2d(iy_ne, ix_ne, inp_H, inp_W)
0542:                 ? inp_ptr_NC[iy_ne * inp_sH + ix_ne * inp_sW] * ne
0543:                 : zero_point * ne;
0544:             res += within_bounds_2d(iy_sw, ix_sw, inp_H, inp_W)
0545:                 ? inp_ptr_NC[iy_sw * inp_sH + ix_sw * inp_sW] * sw
0546:                 : zero_point * sw;
0547:             res += within_bounds_2d(iy_se, ix_se, inp_H, inp_W)
0548:                 ? inp_ptr_NC[iy_se * inp_sH + ix_se * inp_sW] * se
0549:                 : zero_point * se;
0550:             *out_ptr_NCHW = std::nearbyint(res);
0551:           }
0552:         }
0553:       }
0554:     }
0555:   });
0556:   return output;
0557: }
0558: 
0559: Tensor _grid_sampler_2d_cpu_fallback(const Tensor& input, const Tensor& grid,
0560:                                      int64_t interpolation_mode_,
0561:                                      int64_t padding_mode_,
0562:                                      bool align_corners) {
0563:   // See NOTE [ grid_sampler Native Functions ].
0564:   // Add checks here in case this is called instead of grid_sampler.
0565:   check_grid_sampler_common(input, grid);
0566:   check_grid_sampler_2d(input, grid);
0567: 
0568:   auto interpolation_mode = static_cast<GridSamplerInterpolation>(interpolation_mode_);
0569:   auto padding_mode = static_cast<GridSamplerPadding>(padding_mode_);
0570:   using scalar_t = float;
```
- **EN**: Lines 541-570 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: within_bounds_2d, nearbyint, _grid_sampler_2d_cpu_fallback, check_grid_sampler_common.
- **CN**: 第 541-570 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：within_bounds_2d, nearbyint, _grid_sampler_2d_cpu_fallback, check_grid_sampler_common。

### Lines 571-600 / 第 571-600 行
```cpp
0571: 
0572:   int64_t N = input.size(0);
0573:   int64_t C = input.size(1);
0574:   int64_t inp_H = input.size(2);
0575:   int64_t inp_W = input.size(3);
0576:   int64_t out_H = grid.size(1);
0577:   int64_t out_W = grid.size(2);
0578:   auto output = at::empty({N, C, out_H, out_W}, input.options());
0579:   if (output.numel() == 0) {
0580:       return output;
0581:   }
0582:   int64_t inp_sN = input.stride(0);
0583:   int64_t inp_sC = input.stride(1);
0584:   int64_t inp_sH = input.stride(2);
0585:   int64_t inp_sW = input.stride(3);
0586:   int64_t grid_sN = grid.stride(0);
0587:   int64_t grid_sH = grid.stride(1);
0588:   int64_t grid_sW = grid.stride(2);
0589:   int64_t grid_sCoor = grid.stride(3);
0590:   int64_t out_sN = output.stride(0);
0591:   int64_t out_sC = output.stride(1);
0592:   int64_t out_sH = output.stride(2);
0593:   int64_t out_sW = output.stride(3);
0594:   const scalar_t *inp_ptr = input.const_data_ptr<scalar_t>();
0595:   scalar_t *out_ptr = output.data_ptr<scalar_t>();
0596:   const scalar_t *grid_ptr = grid.const_data_ptr<scalar_t>();
0597:   // loop over each output pixel
0598:   at::parallel_for(0, N, 0, [&](int64_t start, int64_t end) {
0599:     for (const auto n : c10::irange(start, end)) {
0600:       const scalar_t *grid_ptr_N = grid_ptr + n * grid_sN;
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, control-flow checks, return paths. Notable symbols: size, empty, options, numel.
- **CN**: 第 571-600 行主要涉及变量/别名声明、控制流逻辑、返回路径。 值得关注的符号包括：size, empty, options, numel。

### Lines 601-630 / 第 601-630 行
```cpp
0601:       const scalar_t *inp_ptr_N = inp_ptr + n * inp_sN;
0602:       for (const auto h : c10::irange(out_H)) {
0603:         for (const auto w : c10::irange(out_W)) {
0604:           // get the corresponding input x, y, z coordinates from grid
0605:           const scalar_t *grid_ptr_NHW = grid_ptr_N + h * grid_sH + w * grid_sW;
0606:           scalar_t x = *grid_ptr_NHW;
0607:           scalar_t y = grid_ptr_NHW[grid_sCoor];
0608: 
0609:           scalar_t ix = grid_sampler_compute_source_index(x, inp_W, padding_mode, align_corners);
0610:           scalar_t iy = grid_sampler_compute_source_index(y, inp_H, padding_mode, align_corners);
0611: 
0612:           if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
0613:             // get corner pixel values from (x, y)
0614:             // for 4d, we use north-east-south-west
0615:             int64_t ix_nw = static_cast<int64_t>(std::floor(ix));
0616:             int64_t iy_nw = static_cast<int64_t>(std::floor(iy));
0617: 
0618:             int64_t ix_ne = ix_nw + 1;
0619:             int64_t iy_ne = iy_nw;
0620: 
0621:             int64_t ix_sw = ix_nw;
0622:             int64_t iy_sw = iy_nw + 1;
0623: 
0624:             int64_t ix_se = ix_nw + 1;
0625:             int64_t iy_se = iy_nw + 1;
0626: 
0627: 
0628:             // get surfaces to each neighbor:
0629:             scalar_t nw = (ix_se - ix)    * (iy_se - iy);
0630:             scalar_t ne = (ix    - ix_sw) * (iy_sw - iy);
```
- **EN**: Lines 601-630 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: irange, grid_sampler_compute_source_index, from, floor.
- **CN**: 第 601-630 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：irange, grid_sampler_compute_source_index, from, floor。

### Lines 631-660 / 第 631-660 行
```cpp
0631:             scalar_t sw = (ix_ne - ix)    * (iy    - iy_ne);
0632:             scalar_t se = (ix    - ix_nw) * (iy    - iy_nw);
0633: 
0634:             // calculate bilinear weighted pixel value and set output pixel
0635:             const scalar_t *inp_ptr_NC = inp_ptr_N;
0636:             scalar_t *out_ptr_NCHW = out_ptr + n * out_sN + h * out_sH + w * out_sW;
0637:             for (int64_t c = 0; c < C; ++c, out_ptr_NCHW += out_sC, inp_ptr_NC += inp_sC) {
0638:               auto res = static_cast<scalar_t>(0);
0639:               if (within_bounds_2d(iy_nw, ix_nw, inp_H, inp_W)) {
0640:                 res += inp_ptr_NC[iy_nw * inp_sH + ix_nw * inp_sW] * nw;
0641:               }
0642:               if (within_bounds_2d(iy_ne, ix_ne, inp_H, inp_W)) {
0643:                 res += inp_ptr_NC[iy_ne * inp_sH + ix_ne * inp_sW] * ne;
0644:               }
0645:               if (within_bounds_2d(iy_sw, ix_sw, inp_H, inp_W)) {
0646:                 res += inp_ptr_NC[iy_sw * inp_sH + ix_sw * inp_sW] * sw;
0647:               }
0648:               if (within_bounds_2d(iy_se, ix_se, inp_H, inp_W)) {
0649:                 res += inp_ptr_NC[iy_se * inp_sH + ix_se * inp_sW] * se;
0650:               }
0651:               *out_ptr_NCHW = res;
0652:             }
0653:           } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
0654:             int64_t ix_nearest = static_cast<int64_t>(std::nearbyint(ix));
0655:             int64_t iy_nearest = static_cast<int64_t>(std::nearbyint(iy));
0656: 
0657:             // assign nearest neighbour pixel value to output pixel
0658:             scalar_t *out_ptr_NCHW = out_ptr + n * out_sN + h * out_sH + w * out_sW;
0659:             const scalar_t *inp_ptr_NC = inp_ptr_N;
0660:             for (int64_t c = 0; c < C; ++c, out_ptr_NCHW += out_sC, inp_ptr_NC += inp_sC) {
```
- **EN**: Lines 631-660 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: within_bounds_2d, nearbyint.
- **CN**: 第 631-660 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：within_bounds_2d, nearbyint。

### Lines 661-690 / 第 661-690 行
```cpp
0661:               if (within_bounds_2d(iy_nearest, ix_nearest, inp_H, inp_W)) {
0662:                 *out_ptr_NCHW = inp_ptr_NC[iy_nearest * inp_sH + ix_nearest * inp_sW];
0663:               } else {
0664:                 *out_ptr_NCHW = static_cast<scalar_t>(0);
0665:               }
0666:             }
0667:           } else if (interpolation_mode == GridSamplerInterpolation::Bicubic) {
0668:             // grid_sampler_compute_source_index will "clip the value" of idx depends on the padding,
0669:             // which would cause calculation to be wrong,
0670:             // for example x = -0.1 -> ix = 0 for zero padding, but in bicubic ix = floor(x) = -1
0671:             // There would be more problem in reflection padding, since the -1 and +1 direction is not fixed in boundary condition
0672:             ix = grid_sampler_unnormalize(x, inp_W, align_corners);
0673:             iy = grid_sampler_unnormalize(y, inp_H, align_corners);
0674: 
0675:             scalar_t ix_nw = std::floor(ix);
0676:             scalar_t iy_nw = std::floor(iy);
0677: 
0678:             const scalar_t tx = ix - ix_nw;
0679:             const scalar_t ty = iy - iy_nw;
0680: 
0681:             const scalar_t *inp_ptr_NC = inp_ptr_N;
0682:             scalar_t *out_ptr_NCHW = out_ptr + n * out_sN + h * out_sH + w * out_sW;
0683:             for (int64_t c = 0; c < C; ++c, out_ptr_NCHW += out_sC, inp_ptr_NC += inp_sC) {
0684:               // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
0685:               scalar_t coefficients[4];
0686: 
0687:               // Interpolate 4 values in the x direction
0688:               for (const auto i : c10::irange(4)) {
0689:                 coefficients[i] = cubic_interp1d<scalar_t>(
0690:                   get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw - 1, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
```
- **EN**: Lines 661-690 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: within_bounds_2d, floor, grid_sampler_unnormalize, NOLINTNEXTLINE.
- **CN**: 第 661-690 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：within_bounds_2d, floor, grid_sampler_unnormalize, NOLINTNEXTLINE。

### Lines 691-720 / 第 691-720 行
```cpp
0691:                   get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw + 0, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
0692:                   get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw + 1, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
0693:                   get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw + 2, iy_nw - 1 + i, inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners),
0694:                   tx);
0695:               }
0696: 
0697:               // Interpolate in the y direction
0698:               *out_ptr_NCHW = cubic_interp1d<scalar_t>(
0699:                 coefficients[0],
0700:                 coefficients[1],
0701:                 coefficients[2],
0702:                 coefficients[3],
0703:                 ty);
0704:             }
0705:           }
0706:         }
0707:       }
0708:     }
0709:   });
0710:   return output;
0711: }
0712: 
0713: std::tuple<Tensor, Tensor>
0714: _grid_sampler_2d_cpu_fallback_backward(const Tensor& grad_output,
0715:                                        const Tensor& input, const Tensor& grid,
0716:                                        int64_t interpolation_mode_,
0717:                                        int64_t padding_mode_,
0718:                                        bool align_corners) {
0719:   // See NOTE [ grid_sampler Native Functions ].
0720:   // Add checks here in case this is called instead of grid_sampler.
```
- **EN**: Lines 691-720 mainly cover expressions/calls, function signatures/definitions, comments/documentation. Notable symbols: _grid_sampler_2d_cpu_fallback_backward.
- **CN**: 第 691-720 行主要涉及表达式或调用、函数签名或实现、注释或说明。 值得关注的符号包括：_grid_sampler_2d_cpu_fallback_backward。

### Lines 721-750 / 第 721-750 行
```cpp
0721:   check_grid_sampler_common(input, grid);
0722:   check_grid_sampler_2d(input, grid);
0723: 
0724:   const auto interpolation_mode = static_cast<GridSamplerInterpolation>(interpolation_mode_);
0725:   const auto padding_mode = static_cast<GridSamplerPadding>(padding_mode_);
0726:   using scalar_t = float;
0727: 
0728:   auto grad_input = at::zeros_like(input, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0729:   auto grad_grid = at::empty_like(grid, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0730:   if (grid.numel() == 0 || input.numel() == 0) {
0731:     grad_grid.zero_();
0732:     return std::make_tuple(grad_input, grad_grid);
0733:   }
0734:   // If interpolation mode is Nearest, then grad_grid is not filled in the
0735:   // loop below.
0736:   if (interpolation_mode == GridSamplerInterpolation::Nearest) {
0737:     grad_grid.zero_();
0738:   }
0739:   int64_t N = input.size(0);
0740:   int64_t C = input.size(1);
0741:   int64_t inp_H = input.size(2);
0742:   int64_t inp_W = input.size(3);
0743:   int64_t out_H = grid.size(1);
0744:   int64_t out_W = grid.size(2);
0745:   int64_t inp_sN = input.stride(0);
0746:   int64_t inp_sC = input.stride(1);
0747:   int64_t inp_sH = input.stride(2);
0748:   int64_t inp_sW = input.stride(3);
0749:   int64_t grid_sN = grid.stride(0);
0750:   int64_t grid_sH = grid.stride(1);
```
- **EN**: Lines 721-750 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: check_grid_sampler_common, check_grid_sampler_2d, zeros_like, empty_like.
- **CN**: 第 721-750 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：check_grid_sampler_common, check_grid_sampler_2d, zeros_like, empty_like。

### Lines 751-780 / 第 751-780 行
```cpp
0751:   int64_t grid_sW = grid.stride(2);
0752:   int64_t grid_sCoor = grid.stride(3);
0753:   int64_t gOut_sN = grad_output.stride(0);
0754:   int64_t gOut_sC = grad_output.stride(1);
0755:   int64_t gOut_sH = grad_output.stride(2);
0756:   int64_t gOut_sW = grad_output.stride(3);
0757:   int64_t gInp_sN = grad_input.stride(0);
0758:   int64_t gInp_sC = grad_input.stride(1);
0759:   int64_t gInp_sH = grad_input.stride(2);
0760:   int64_t gInp_sW = grad_input.stride(3);
0761:   int64_t gGrid_sN = grad_grid.stride(0);
0762:   int64_t gGrid_sW = grad_grid.stride(2);
0763:   const scalar_t *inp_ptr = input.const_data_ptr<scalar_t>();
0764:   const scalar_t *grid_ptr = grid.const_data_ptr<scalar_t>();
0765:   const scalar_t *gOut_ptr = grad_output.const_data_ptr<scalar_t>();
0766:   scalar_t *gInp_ptr = grad_input.mutable_data_ptr<scalar_t>();
0767:   scalar_t *gGrid_ptr = grad_grid.data_ptr<scalar_t>();
0768:   // loop over each output pixel
0769:   at::parallel_for(0, N, 0, [&](int64_t start, int64_t end) {
0770:     for (const auto n : c10::irange(start, end)) {
0771:       const scalar_t *grid_ptr_N = grid_ptr + n * grid_sN;
0772:       const scalar_t *inp_ptr_N = inp_ptr + n * inp_sN;
0773:       scalar_t *gGrid_ptr_NHW = gGrid_ptr + n * gGrid_sN;
0774:       for (const auto h : c10::irange(out_H)) {
0775:         for (int64_t w = 0; w < out_W; ++w, gGrid_ptr_NHW += gGrid_sW /* grad_grid is contiguous */ ) {
0776:           // get the corresponding input x, y coordinates from grid
0777:           const scalar_t *grid_ptr_NHW = grid_ptr_N + h * grid_sH + w * grid_sW;
0778:           scalar_t x = *grid_ptr_NHW;
0779:           scalar_t y = grid_ptr_NHW[grid_sCoor];
0780: 
```
- **EN**: Lines 751-780 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: stride, parallel_for, irange.
- **CN**: 第 751-780 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：stride, parallel_for, irange。

### Lines 781-810 / 第 781-810 行
```cpp
0781:           // multipliers for gradients on ix, iy
0782:           scalar_t gix_mult{}, giy_mult{};
0783:           scalar_t ix = grid_sampler_compute_source_index_set_grad(x, inp_W, padding_mode, align_corners, &gix_mult);
0784:           scalar_t iy = grid_sampler_compute_source_index_set_grad(y, inp_H, padding_mode, align_corners, &giy_mult);
0785: 
0786:           if (interpolation_mode == GridSamplerInterpolation::Bilinear) {
0787:             // get corner pixel values from (x, y)
0788:             // for 4d, we use north-east-south-west
0789:             int64_t ix_nw = static_cast<int64_t>(std::floor(ix));
0790:             int64_t iy_nw = static_cast<int64_t>(std::floor(iy));
0791: 
0792:             int64_t ix_ne = ix_nw + 1;
0793:             int64_t iy_ne = iy_nw;
0794: 
0795:             int64_t ix_sw = ix_nw;
0796:             int64_t iy_sw = iy_nw + 1;
0797: 
0798:             int64_t ix_se = ix_nw + 1;
0799:             int64_t iy_se = iy_nw + 1;
0800: 
0801:             // get surfaces to each neighbor:
0802:             scalar_t nw = (ix_se - ix)    * (iy_se - iy);
0803:             scalar_t ne = (ix    - ix_sw) * (iy_sw - iy);
0804:             scalar_t sw = (ix_ne - ix)    * (iy    - iy_ne);
0805:             scalar_t se = (ix    - ix_nw) * (iy    - iy_nw);
0806: 
0807:             scalar_t gix = static_cast<scalar_t>(0), giy = static_cast<scalar_t>(0);
0808:             const scalar_t *gOut_ptr_NCHW = gOut_ptr + n * gOut_sN + h * gOut_sH + w * gOut_sW;
0809:             scalar_t *gInp_ptr_NC = gInp_ptr + n * gInp_sN;
0810:             const scalar_t *inp_ptr_NC = inp_ptr_N;
```
- **EN**: Lines 781-810 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: grid_sampler_compute_source_index_set_grad, from, floor.
- **CN**: 第 781-810 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：grid_sampler_compute_source_index_set_grad, from, floor。

### Lines 811-840 / 第 811-840 行
```cpp
0811:             // calculate bilinear weighted pixel value and set output pixel
0812:             for (int64_t c = 0; c < C; ++c, gOut_ptr_NCHW += gOut_sC, gInp_ptr_NC += gInp_sC, inp_ptr_NC += inp_sC) {
0813:               scalar_t gOut = *gOut_ptr_NCHW;
0814: 
0815:               // calculate and set grad_input
0816:               safe_add_2d(gInp_ptr_NC, iy_nw, ix_nw, gInp_sH, gInp_sW, inp_H, inp_W, nw * gOut);
0817:               safe_add_2d(gInp_ptr_NC, iy_ne, ix_ne, gInp_sH, gInp_sW, inp_H, inp_W, ne * gOut);
0818:               safe_add_2d(gInp_ptr_NC, iy_sw, ix_sw, gInp_sH, gInp_sW, inp_H, inp_W, sw * gOut);
0819:               safe_add_2d(gInp_ptr_NC, iy_se, ix_se, gInp_sH, gInp_sW, inp_H, inp_W, se * gOut);
0820: 
0821:               // calculate grad_grid
0822:               if (within_bounds_2d(iy_nw, ix_nw, inp_H, inp_W)) {
0823:                 scalar_t nw_val = inp_ptr_NC[iy_nw * inp_sH + ix_nw * inp_sW];
0824:                 gix -= nw_val * (iy_se - iy) * gOut;
0825:                 giy -= nw_val * (ix_se - ix) * gOut;
0826:               }
0827:               if (within_bounds_2d(iy_ne, ix_ne, inp_H, inp_W)) {
0828:                 scalar_t ne_val = inp_ptr_NC[iy_ne * inp_sH + ix_ne * inp_sW];
0829:                 gix += ne_val * (iy_sw - iy) * gOut;
0830:                 giy -= ne_val * (ix - ix_sw) * gOut;
0831:               }
0832:               if (within_bounds_2d(iy_sw, ix_sw, inp_H, inp_W)) {
0833:                 scalar_t sw_val = inp_ptr_NC[iy_sw * inp_sH + ix_sw * inp_sW];
0834:                 gix -= sw_val * (iy - iy_ne) * gOut;
0835:                 giy += sw_val * (ix_ne - ix) * gOut;
0836:               }
0837:               if (within_bounds_2d(iy_se, ix_se, inp_H, inp_W)) {
0838:                 scalar_t se_val = inp_ptr_NC[iy_se * inp_sH + ix_se * inp_sW];
0839:                 gix += se_val * (iy - iy_nw) * gOut;
0840:                 giy += se_val * (ix - ix_nw) * gOut;
```
- **EN**: Lines 811-840 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: safe_add_2d, within_bounds_2d.
- **CN**: 第 811-840 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：safe_add_2d, within_bounds_2d。

### Lines 841-870 / 第 841-870 行
```cpp
0841:               }
0842:             }
0843: 
0844:             // assuming grad_grid is contiguous
0845:             gGrid_ptr_NHW[0] = gix_mult * gix;
0846:             gGrid_ptr_NHW[1] = giy_mult * giy;
0847:           } else if (interpolation_mode == GridSamplerInterpolation::Nearest) {
0848:             int64_t ix_nearest = static_cast<int64_t>(std::nearbyint(ix));
0849:             int64_t iy_nearest = static_cast<int64_t>(std::nearbyint(iy));
0850: 
0851:             // assign nearest neighbour pixel value to output pixel
0852:             const scalar_t *gOut_ptr_NCHW = gOut_ptr + n * gOut_sN + h * gOut_sH + w * gOut_sW;
0853:             scalar_t *gInp_ptr_NC = gInp_ptr + n * gInp_sN;
0854:             for (int64_t c = 0; c < C; ++c, gOut_ptr_NCHW += gOut_sC, gInp_ptr_NC += gInp_sC) {
0855:               // calculate and set grad_input
0856:               safe_add_2d(gInp_ptr_NC, iy_nearest, ix_nearest, gInp_sH, gInp_sW,
0857:                           inp_H, inp_W, *gOut_ptr_NCHW);
0858:             }
0859:           } else if (interpolation_mode == GridSamplerInterpolation::Bicubic) {
0860: 
0861:             ix = grid_sampler_unnormalize_set_grad(x, inp_W, align_corners, &gix_mult);
0862:             iy = grid_sampler_unnormalize_set_grad(y, inp_H, align_corners, &giy_mult);
0863: 
0864:             scalar_t ix_nw = std::floor(ix);
0865:             scalar_t iy_nw = std::floor(iy);
0866: 
0867:             const scalar_t tx = ix - ix_nw;
0868:             const scalar_t ty = iy - iy_nw;
0869: 
0870:             // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
```
- **EN**: Lines 841-870 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: nearbyint, safe_add_2d, grid_sampler_unnormalize_set_grad, floor.
- **CN**: 第 841-870 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：nearbyint, safe_add_2d, grid_sampler_unnormalize_set_grad, floor。

### Lines 871-900 / 第 871-900 行
```cpp
0871:             scalar_t x_coeffs[4];
0872:             // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
0873:             scalar_t y_coeffs[4];
0874:             // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
0875:             scalar_t x_coeffs_grad[4];
0876:             // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
0877:             scalar_t y_coeffs_grad[4];
0878: 
0879:             get_cubic_upsample_coefficients<scalar_t>(x_coeffs, tx);
0880:             get_cubic_upsample_coefficients<scalar_t>(y_coeffs, ty);
0881:             get_cubic_coefficients_grad<scalar_t>(x_coeffs_grad, tx);
0882:             get_cubic_coefficients_grad<scalar_t>(y_coeffs_grad, ty);
0883: 
0884:             scalar_t gix = static_cast<scalar_t>(0);
0885:             scalar_t giy = static_cast<scalar_t>(0);
0886: 
0887:             const scalar_t *gOut_ptr_NCHW = gOut_ptr + n * gOut_sN + h * gOut_sH + w * gOut_sW;
0888:             scalar_t *gInp_ptr_NC = gInp_ptr + n * gInp_sN;
0889:             const scalar_t *inp_ptr_NC = inp_ptr_N;
0890: 
0891:             for (int64_t c = 0; c < C; ++c, gOut_ptr_NCHW += gOut_sC, gInp_ptr_NC += gInp_sC, inp_ptr_NC+= inp_sC) {
0892:               scalar_t gOut = *gOut_ptr_NCHW;
0893: 
0894:               for (const auto i : c10::irange(4)) {
0895:                 for (const auto j : c10::irange(4)) {
0896: 
0897:                   // set input gradient
0898:                   add_value_bounded<scalar_t>(gInp_ptr_NC, ix_nw - 1 + i, iy_nw - 1 + j,
0899:                     inp_W, inp_H, gInp_sW, gInp_sH, gOut * x_coeffs[i] * y_coeffs[j], padding_mode, align_corners);
0900: 
```
- **EN**: Lines 871-900 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: NOLINTNEXTLINE, irange.
- **CN**: 第 871-900 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：NOLINTNEXTLINE, irange。

### Lines 901-930 / 第 901-930 行
```cpp
0901:                   // set grid gradient
0902:                   scalar_t val = get_value_bounded<scalar_t>(inp_ptr_NC, ix_nw - 1 + i, iy_nw - 1 + j,
0903:                     inp_W, inp_H, inp_sW, inp_sH, padding_mode, align_corners);
0904: 
0905:                   gix -= val * x_coeffs_grad[i] * y_coeffs[j] * gOut;
0906:                   giy -= val * y_coeffs_grad[j] * x_coeffs[i] * gOut;
0907:                 }
0908:               }
0909:             }
0910:             gGrid_ptr_NHW[0] = gix_mult * gix;
0911:             gGrid_ptr_NHW[1] = giy_mult * giy;
0912:           }
0913:         }
0914:       }
0915:     }
0916:   });
0917:   return std::make_tuple(grad_input, grad_grid);
0918: }
0919: 
0920: Tensor grid_sampler_2d_cpu(const Tensor& input, const Tensor& grid,
0921:                            int64_t interpolation_mode, int64_t padding_mode,
0922:                            bool align_corners) {
0923:   // See NOTE [ grid_sampler Native Functions ].
0924:   // Add checks here in case this is called instead of grid_sampler.
0925:   check_grid_sampler_common(input, grid);
0926:   check_grid_sampler_2d(input, grid);
0927: 
0928:   if (input.scalar_type() == kQUInt8) {
0929:     return native::_grid_sampler_2d_cpu_quantized(
0930:         input, grid, interpolation_mode, padding_mode, align_corners);
```
- **EN**: Lines 901-930 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: make_tuple, grid_sampler_2d_cpu, check_grid_sampler_common, check_grid_sampler_2d.
- **CN**: 第 901-930 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：make_tuple, grid_sampler_2d_cpu, check_grid_sampler_common, check_grid_sampler_2d。

### Lines 931-960 / 第 931-960 行
```cpp
0931:   }
0932:   // AVX gather instructions use signed 32-bit offsets to gather float values.
0933:   // Check for possible overflow and fallback to scalar implementation
0934:   if (input.scalar_type() == kFloat) {
0935:     auto sizes = input.sizes();
0936:     auto strides = input.strides();
0937:     const auto grid_sW = grid.strides()[2];
0938:     // NOTE: Gather offsets are only used for the input H, W dimensions
0939:     //       or only for strided access to the grid tensor
0940:     auto max_gather_offset = std::max(
0941:       (sizes[2] - 1) * strides[2] + (sizes[3] - 1) * strides[3],
0942:       grid_sW * (vec::Vectorized<float>::size() - 1));
0943: 
0944:     if (max_gather_offset > std::numeric_limits<int32_t>::max()) {
0945:       return native::_grid_sampler_2d_cpu_fallback(
0946:         input, grid, interpolation_mode, padding_mode, align_corners);
0947:     }
0948:   }
0949: 
0950:   auto in_size = input.sizes();
0951:   auto grid_size = grid.sizes();
0952:   auto output = at::empty(
0953:       {in_size[0], in_size[1], grid_size[1], grid_size[2]}, input.options());
0954:   grid_sampler_2d_cpu_kernel(
0955:       kCPU, output, input, grid, interpolation_mode, padding_mode, align_corners);
0956:   return output;
0957: }
0958: 
0959: DEFINE_DISPATCH(grid_sampler_2d_cpu_kernel);
0960: 
```
- **EN**: Lines 931-960 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: scalar_type, sizes, strides, max.
- **CN**: 第 931-960 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：scalar_type, sizes, strides, max。

### Lines 961-990 / 第 961-990 行
```cpp
0961: 
0962: Tensor grid_sampler_3d_cpu(const Tensor& input, const Tensor& grid,
0963:                            int64_t interpolation_mode, int64_t padding_mode,
0964:                            bool align_corners) {
0965:   // See NOTE [ grid_sampler Native Functions ].
0966:   // Add checks here in case this is called instead of grid_sampler.
0967:   check_grid_sampler_common(input, grid);
0968:   check_grid_sampler_3d(input, grid, interpolation_mode);
0969: 
0970:   return AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(), "grid_sampler3d_cpu", [&] {
0971:     return grid_sampler_3d_cpu_impl<scalar_t>(
0972:       input, grid, static_cast<GridSamplerInterpolation>(interpolation_mode),
0973:       static_cast<GridSamplerPadding>(padding_mode), align_corners);
0974:   });
0975: }
0976: 
0977: std::tuple<Tensor, Tensor>
0978: grid_sampler_2d_backward_cpu(const Tensor& grad_output, const Tensor& input, const Tensor& grid,
0979:                              int64_t interpolation_mode, int64_t padding_mode, bool align_corners,
0980:                              std::array<bool,2> output_mask) {
0981:   // See NOTE [ grid_sampler Native Functions ].
0982:   // Add checks here in case this is called instead of grid_sampler.
0983:   check_grid_sampler_common(input, grid);
0984:   check_grid_sampler_2d(input, grid);
0985: 
0986:   // AVX gather instructions use signed 32-bit offsets to gather float values.
0987:   // Check for possible overflow and fallback to scalar implementation
0988:   if (input.scalar_type() == kFloat) {
0989:     auto isizes = input.sizes();
0990:     auto istrides = input.strides();
```
- **EN**: Lines 961-990 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: grid_sampler_3d_cpu, check_grid_sampler_common, check_grid_sampler_3d, AT_DISPATCH_FLOATING_TYPES_AND2.
- **CN**: 第 961-990 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：grid_sampler_3d_cpu, check_grid_sampler_common, check_grid_sampler_3d, AT_DISPATCH_FLOATING_TYPES_AND2。

### Lines 991-1020 / 第 991-1020 行
```cpp
0991:     auto gsizes = grad_output.sizes();
0992:     auto gstrides = grad_output.strides();
0993:     const auto grid_sW = grid.strides()[2];
0994:     // NOTE: Gather offsets are only used for the height and width dimensions
0995:     auto max_gather_offset = std::max(
0996:       {
0997:         (isizes[2] - 1) * istrides[2] + (isizes[3] - 1) * istrides[3],
0998:         (gsizes[2] - 1) * gstrides[2] + (gsizes[3] - 1) * gstrides[3],
0999:       grid_sW * (vec::Vectorized<float>::size() - 1)});
1000: 
1001:     if (max_gather_offset > std::numeric_limits<int32_t>::max()) {
1002:       return native::_grid_sampler_2d_cpu_fallback_backward(
1003:         grad_output, input, grid, interpolation_mode, padding_mode, align_corners);
1004:     }
1005:   }
1006: 
1007:   auto input_requires_grad = output_mask[0];
1008:   Tensor grad_input = ([&]() {
1009:     if (input_requires_grad) {
1010:       return at::zeros_like(input, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
1011:     } else {
1012:       return Tensor();
1013:     }
1014:   })();
1015:   auto grad_grid = at::empty_like(grid, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
1016:   grid_sampler_2d_backward_cpu_kernel(
1017:       kCPU, grad_input, grad_grid, grad_output, input, grid,
1018:       interpolation_mode, padding_mode, align_corners, output_mask);
1019:   return std::make_tuple(std::move(grad_input), std::move(grad_grid));
1020: }
```
- **EN**: Lines 991-1020 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: sizes, strides, max, size.
- **CN**: 第 991-1020 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：sizes, strides, max, size。

### Lines 1021-1050 / 第 1021-1050 行
```cpp
1021: 
1022: DEFINE_DISPATCH(grid_sampler_2d_backward_cpu_kernel);
1023: 
1024: std::tuple<Tensor, Tensor>
1025: grid_sampler_3d_backward_cpu(const Tensor& grad_output, const Tensor& input, const Tensor& grid,
1026:                              int64_t interpolation_mode, int64_t padding_mode, bool align_corners,
1027:                              std::array<bool,2> output_mask) {
1028:   // See NOTE [ grid_sampler Native Functions ].
1029:   // Add checks here in case this is called instead of grid_sampler.
1030:   check_grid_sampler_common(input, grid);
1031:   check_grid_sampler_3d(input, grid, interpolation_mode);
1032: 
1033:   return AT_DISPATCH_FLOATING_TYPES_AND2(kHalf, kBFloat16, input.scalar_type(), "grid_sampler_3d_backward_cpu", [&] {
1034:     return grid_sampler_3d_backward_cpu_impl<scalar_t>(
1035:       grad_output, input, grid,
1036:       static_cast<GridSamplerInterpolation>(interpolation_mode),
1037:       static_cast<GridSamplerPadding>(padding_mode),
1038:       align_corners, output_mask);
1039:   });
1040: }
1041: 
1042: // See NOTE [ grid_sampler Native Functions ].
1043: Tensor grid_sampler(
1044:   const Tensor& input,
1045:   const Tensor& grid,
1046:   int64_t interpolation_mode,
1047:   int64_t padding_mode,
1048:   bool align_corners
1049: ) {
1050:   if (cond_cudnn_grid_sampler(input, grid) &&
```
- **EN**: Lines 1021-1050 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: DEFINE_DISPATCH, grid_sampler_3d_backward_cpu, check_grid_sampler_common, check_grid_sampler_3d.
- **CN**: 第 1021-1050 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：DEFINE_DISPATCH, grid_sampler_3d_backward_cpu, check_grid_sampler_common, check_grid_sampler_3d。

### Lines 1051-1068 / 第 1051-1068 行
```cpp
1051:       static_cast<GridSamplerInterpolation>(interpolation_mode) ==
1052:         GridSamplerInterpolation::Bilinear &&
1053:       static_cast<GridSamplerPadding>(padding_mode) ==
1054:         GridSamplerPadding::Zeros &&
1055:       align_corners) {
1056:     return cudnn_grid_sampler(input, grid);
1057:   }
1058: 
1059:   if (input.dim() == 4) {
1060:     return at::grid_sampler_2d(
1061:       input, grid, interpolation_mode, padding_mode, align_corners);
1062:   } else {
1063:     return at::grid_sampler_3d(
1064:       input, grid, interpolation_mode, padding_mode, align_corners);
1065:   }
1066: }
1067: 
1068: }  // namespace at::native
```
- **EN**: Lines 1051-1068 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: cudnn_grid_sampler, dim, grid_sampler_2d, grid_sampler_3d.
- **CN**: 第 1051-1068 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：cudnn_grid_sampler, dim, grid_sampler_2d, grid_sampler_3d。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: Parallel loop scheduling  
  **CN**: 并行循环调度
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/native/GridSampler.h>`, `<ATen/native/GridSamplerUtils.h>`, `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/cpu/vec/vec.h>`, `<ATen/native/UpSample.h>`, `<ATen/native/cpu/GridSamplerKernel.h>`, `<c10/util/Exception.h>`, `<c10/util/irange.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`

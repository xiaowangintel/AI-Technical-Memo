# UpSampleBilinear2d.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/UpSampleBilinear2d.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU resampling/interpolation kernels and coordinate mapping logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 重采样/插值 kernel 与坐标映射逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/Parallel.h>
 5: #include <ATen/native/UpSample.h>
 6: #include <ATen/native/quantized/AffineQuantizer.h>
 7: #include <ATen/native/quantized/cpu/QuantizedOps.h>
 8: #include <ATen/native/cpu/utils.h>
 9: #include <c10/util/accumulate.h>
10: #include <c10/util/irange.h>
11:
12: #ifndef AT_PER_OPERATOR_HEADERS
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 13-28
```cpp
13: #include <ATen/Functions.h>
14: #include <ATen/NativeFunctions.h>
15: #else
16: #include <ATen/ops/_empty_affine_quantized.h>
17: #include <ATen/ops/upsample_bilinear2d_native.h>
18: #endif
19:
20: #include <cstring>
21:
22: namespace at::native {
23: namespace {
24:
25: // pre calculate interpolation params on width
26: struct UpsampleBilinearParamW {
27:   int64_t w1, w1p;
28:   float w0lambda, w1lambda;
```
- EN: This range pulls in required headers, including `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 30-40
```cpp
30:   UpsampleBilinearParamW(int64_t w1, int64_t w1p, float w0lambda, float w1lambda)
31:     : w1(w1)
32:     , w1p(w1p)
33:     , w0lambda(w0lambda)
34:     , w1lambda(w1lambda) {}
35: };
36:
37: // at::native functions for the native_functions.yaml
38: template <typename scalar_t>
39: void upsample_bilinear2d_out_frame(
40:     Tensor& output,
```
- EN: The main symbol in this range is `UpsampleBilinearParamW`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `UpsampleBilinearParamW`，它们直接构成本文件的算子逻辑。

### Lines 41-52
```cpp
41:     const Tensor& input,
42:     int64_t input_height,
43:     int64_t input_width,
44:     int64_t output_height,
45:     int64_t output_width,
46:     int64_t nbatch,
47:     int64_t channels,
48:     bool align_corners,
49:     std::optional<double> scales_h,
50:     std::optional<double> scales_w) {
51:   auto* idata = static_cast<const scalar_t*>(input.const_data_ptr());
52:   auto* odata = static_cast<scalar_t*>(output.data_ptr());
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 54-69
```cpp
54:   channels = channels * nbatch;
55:   if (channels == 0 || output_height == 0 || output_width == 0) {
56:     return;
57:   }
58:   auto* i_p = reinterpret_cast<const typename scalar_t::underlying*>(idata);
59:   auto* o_p = reinterpret_cast<typename scalar_t::underlying*>(odata);
60:
61:   // special case: just copy
62:   if (input_height == output_height && input_width == output_width) {
63:     std::memcpy(
64:         o_p,
65:         i_p,
66:         channels * input_height * input_width *
67:             sizeof(typename scalar_t::underlying));
68:     return;
69:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 71-86
```cpp
71:   const auto rheight = area_pixel_compute_scale<float>(
72:       input_height, output_height, align_corners, scales_h);
73:
74:   const auto rwidth = area_pixel_compute_scale<float>(
75:       input_width, output_width, align_corners, scales_w);
76:
77:   float output_scale = static_cast<float>(output.q_scale() / input.q_scale());
78:
79:   const int64_t input_q_zero_point = input.q_zero_point();
80:   const int64_t output_q_zero_point = output.q_zero_point();
81:
82:   std::vector<UpsampleBilinearParamW> params_w;
83:   params_w.reserve(output_width);
84:   for (const auto w2 : c10::irange(output_width)) {
85:     const auto w1r = area_pixel_compute_source_index<float>(
86:         rwidth, w2, align_corners, /*cubic=*/false);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 88-102
```cpp
 88:     const int64_t w1 = w1r;
 89:     const int64_t w1p = (w1 < input_width - 1) ? 1 : 0;
 90:
 91:     const float w1lambda = w1r - w1;
 92:     const float w0lambda = static_cast<float>(1.) - w1lambda;
 93:
 94:     params_w.emplace_back(w1, w1p, w0lambda, w1lambda);
 95:   }
 96:
 97:   // compared to 'nearest', each requires 4 points and takes additional * and +
 98:   // set the scale to be 16.
 99:   int64_t grain_size = internal::GRAIN_SIZE / std::max(int64_t{1}, output_width) / 16;
100:   at::parallel_for(0, channels * output_height, grain_size, [&](int64_t begin, int64_t end) {
101:     int64_t nc{0}, h2{0};
102:     data_index_init(begin, nc, channels, h2, output_height);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 104-115
```cpp
104:     for (const auto i : c10::irange(begin, end)) {
105:       const auto h1r = area_pixel_compute_source_index<float>(
106:           rheight, h2, align_corners, /*cubic=*/false);
107:
108:       const int64_t h1 = h1r;
109:       const int64_t h1p = (h1 < input_height - 1) ? 1 : 0;
110:
111:       const float h1lambda = h1r - h1;
112:       const float h0lambda = static_cast<float>(1.) - h1lambda;
113:
114:       const auto* i_ptr = &i_p[nc * input_height * input_width];
115:       auto* pos2 = &o_p[i * output_width];
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 117-127
```cpp
117:       for (const auto w2 : c10::irange(output_width)) {
118:         const auto& param_w = params_w[w2];
119:         const int64_t w1 = param_w.w1;
120:         const int64_t w1p = param_w.w1p;
121:         const float w0lambda = param_w.w0lambda;
122:         const float w1lambda = param_w.w1lambda;
123:
124:         const auto* pos1 = i_ptr + h1 * input_width + w1;
125:
126:         const float result = h0lambda * (w0lambda * pos1[0] + w1lambda * pos1[w1p]) +
127:             h1lambda *
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 128-140
```cpp
128:                 (w0lambda * pos1[h1p * input_width] +
129:                  w1lambda * pos1[h1p * input_width + w1p]) - input_q_zero_point;
130:         // requantization
131:         pos2[w2] = at::native::quantize_val<scalar_t>(
132:                       output_scale, output_q_zero_point, result)
133:                       .val_;
134:       }
135:
136:       data_index_step(nc, channels, h2, output_height);
137:     }
138:   });
139:
140: }
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 142-155
```cpp
142: } // namespace
143:
144: Tensor upsample_bilinear2d_quantized_cpu(
145:     const Tensor& input,
146:     IntArrayRef output_size,
147:     bool align_corners,
148:     std::optional<double> scales_h,
149:     std::optional<double> scales_w) {
150:   auto full_output_size = native::upsample_2d_common_check(input.sizes(), output_size);
151:
152:   TORCH_CHECK(
153:       input.numel() != 0 || c10::multiply_integers(input.sizes().begin() + 1, input.sizes().end()),
154:       "Non-empty 4D data tensor expected but got a tensor with sizes ",
155:       input.sizes());
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `upsample_bilinear2d_quantized_cpu`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `upsample_bilinear2d_quantized_cpu`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 157-170
```cpp
157:   int64_t nbatch = full_output_size[0];
158:   int64_t channels = full_output_size[1];
159:   int64_t output_height = full_output_size[2];
160:   int64_t output_width = full_output_size[3];
161:   int64_t input_height = input.size(2);
162:   int64_t input_width = input.size(3);
163:
164:   if (input.is_contiguous(c10::MemoryFormat::ChannelsLast)) {
165:     Tensor output = at::_empty_affine_quantized(
166:         {nbatch, channels, output_height, output_width},
167:         input.options().memory_format(input.suggest_memory_format()),
168:         input.q_scale(),
169:         input.q_zero_point(),
170:         std::nullopt);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 172-182
```cpp
172:     qupsample_bilinear2d_nhwc_stub(
173:         input.device().type(),
174:         output,
175:         input,
176:         input_height,
177:         input_width,
178:         output_height,
179:         output_width,
180:         nbatch,
181:         channels,
182:         align_corners,
```
- EN: The main symbol in this range is `qupsample_bilinear2d_nhwc_stub`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `qupsample_bilinear2d_nhwc_stub`，它们直接构成本文件的算子逻辑。

### Lines 183-194
```cpp
183:         scales_h,
184:         scales_w);
185:     return output;
186:   } else {
187:     Tensor output = at::_empty_affine_quantized(
188:         {nbatch, channels, output_height, output_width},
189:         input.options(),
190:         input.q_scale(),
191:         input.q_zero_point());
192:
193:     auto input_contig = input.contiguous();
194:     AT_DISPATCH_QINT_TYPES(
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 195-211
```cpp
195:         input_contig.scalar_type(), "upsample_bilinear2d", [&] {
196:           upsample_bilinear2d_out_frame<scalar_t>(
197:               output,
198:               input_contig,
199:               input_height,
200:               input_width,
201:               output_height,
202:               output_width,
203:               nbatch,
204:               channels,
205:               align_corners,
206:               scales_h,
207:               scales_w);
208:         });
209:     return output;
210:   }
211: }
```
- EN: The block also assembles or returns the result expected by the surrounding operator code.
- CN: 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 213-214
```cpp
213: DEFINE_DISPATCH(qupsample_bilinear2d_nhwc_stub);
214: } // namespace at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- CPU parallelism / CPU 并行
- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Low-level memory access / 底层内存访问
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/native/UpSample.h`, `ATen/native/quantized/AffineQuantizer.h`, `ATen/native/quantized/cpu/QuantizedOps.h`, `ATen/native/cpu/utils.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`
- c10 headers / c10 头文件: `c10/util/accumulate.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `cstring`
- Key helper symbols / 关键辅助符号: `parallel_for`

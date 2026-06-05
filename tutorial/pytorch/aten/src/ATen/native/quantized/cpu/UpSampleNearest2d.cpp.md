# UpSampleNearest2d.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/UpSampleNearest2d.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU resampling/interpolation kernels and coordinate mapping logic in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 重采样/插值 kernel 与坐标映射逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: #include <ATen/core/Tensor.h>
 3: #include <ATen/Dispatch.h>
 4: #include <ATen/Parallel.h>
 5: #include <ATen/native/UpSample.h>
 6: #include <ATen/native/cpu/utils.h>
 7:
 8: #ifndef AT_PER_OPERATOR_HEADERS
 9: #include <ATen/Functions.h>
10: #include <ATen/NativeFunctions.h>
11: #else
12: #include <ATen/ops/_empty_affine_quantized.h>
13: #include <ATen/ops/_upsample_nearest_exact2d_native.h>
14: #include <ATen/ops/upsample_nearest2d_native.h>
15: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 17-27
```cpp
17: #include <c10/util/accumulate.h>
18: #include <c10/util/irange.h>
19:
20: #include <cstring>
21: #include <vector>
22:
23:
24: namespace at::native {
25:
26: // Define a typedef to dispatch to nearest_idx or nearest_exact_idx
27: typedef int64_t (*nn_compute_source_index_fn_t)(const float, int64_t, int64_t);
```
- EN: This range pulls in required headers, including `c10/util/accumulate.h`, `c10/util/irange.h`, `cstring`. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `c10/util/accumulate.h`, `c10/util/irange.h`, `cstring`。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 29-43
```cpp
29: // at::native functions for the native_functions.yaml
30: template <typename scalar_t, nn_compute_source_index_fn_t nn_compute_source_index_fn>
31: static void upsample_nearest2d_out_frame(
32:     scalar_t* odata,
33:     scalar_t* idata,
34:     int64_t input_height,
35:     int64_t input_width,
36:     int64_t output_height,
37:     int64_t output_width,
38:     int64_t nbatch,
39:     int64_t channels,
40:     std::optional<double> scales_h,
41:     std::optional<double> scales_w) {
42:   float height_scale = compute_scales_value<float>(scales_h, input_height, output_height);
43:   float width_scale = compute_scales_value<float>(scales_w, input_width, output_width);
```
- EN: The main symbol in this range is `upsample_nearest2d_out_frame`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `upsample_nearest2d_out_frame`，它们直接构成本文件的算子逻辑。

### Lines 45-56
```cpp
45:   channels = channels * nbatch;
46:   if (channels == 0 || output_height == 0 || output_width == 0) {
47:     return;
48:   }
49:   auto* i_p = reinterpret_cast<typename scalar_t::underlying*>(idata);
50:   auto* o_p = reinterpret_cast<typename scalar_t::underlying*>(odata);
51:
52:   // special case: just copy
53:   if (input_height == output_height && input_width == output_width) {
54:     std::memcpy(o_p, i_p, channels * input_height * input_width * sizeof(typename scalar_t::underlying));
55:     return;
56:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 58-69
```cpp
58:   std::vector<int64_t> input_offset_arr(output_width);
59:   int64_t* input_offset = input_offset_arr.data();
60:
61:   for (const auto w2 : c10::irange(output_width)) {
62:     const int64_t w1 = nn_compute_source_index_fn(width_scale, w2, input_width);
63:     input_offset[w2] = w1;
64:   }
65:
66:   int64_t grain_size = internal::GRAIN_SIZE / std::max(int64_t{1}, output_width);
67:   at::parallel_for(0, channels * output_height, grain_size, [&](int64_t begin, int64_t end) {
68:     int64_t nc{0}, h2{0};
69:     data_index_init(begin, nc, channels, h2, output_height);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 71-84
```cpp
71:     for (const auto i : c10::irange(begin, end)) {
72:       const int64_t h1 = nn_compute_source_index_fn(height_scale, h2, input_height);
73:       const auto* pos1 = &i_p[nc * input_height * input_width + h1 * input_width];
74:       auto* pos2 = &o_p[i * output_width];
75:
76:       for (const auto w2 : c10::irange(output_width)) {
77:         const int64_t w1 = input_offset[w2];
78:         pos2[w2] = pos1[w1];
79:       }
80:
81:       data_index_step(nc, channels, h2, output_height);
82:     }
83:   });
84: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 86-99
```cpp
86: template <typename scalar_t, nn_compute_source_index_fn_t nn_compute_source_index_fn>
87: static void upsample_nearest2d_out_frame_nhwc(
88:     scalar_t* odata,
89:     scalar_t* idata,
90:     int64_t input_height,
91:     int64_t input_width,
92:     int64_t output_height,
93:     int64_t output_width,
94:     int64_t nbatch,
95:     int64_t channels,
96:     std::optional<double> scales_h,
97:     std::optional<double> scales_w) {
98:   float height_scale = compute_scales_value<float>(scales_h, input_height, output_height);
99:   float width_scale = compute_scales_value<float>(scales_w, input_width, output_width);
```
- EN: The main symbol in this range is `upsample_nearest2d_out_frame_nhwc`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `upsample_nearest2d_out_frame_nhwc`，它们直接构成本文件的算子逻辑。

### Lines 101-114
```cpp
101:   at::parallel_for(0, nbatch * output_height * output_width, 0, [&](int64_t begin, int64_t end) {
102:     int64_t b{0}, h2{0}, w2{0};
103:     data_index_init(begin, b, nbatch, h2, output_height, w2, output_width);
104:
105:     for (const auto i : c10::irange(begin, end)) {
106:       auto* i_p = reinterpret_cast<typename scalar_t::underlying*>(idata + b * input_height * input_width * channels);
107:       auto* o_p = reinterpret_cast<typename scalar_t::underlying*>(odata + i * channels);
108:
109:       const int64_t h1 = nn_compute_source_index_fn(height_scale, h2, input_height);
110:       const int64_t w1 = nn_compute_source_index_fn(width_scale, w2, input_width);
111:
112:       const auto* pos1 = &i_p[(h1 * input_width + w1)*channels];
113:       auto* pos2 = &o_p[0];
114:       std::memcpy(pos2, pos1, channels * sizeof(typename scalar_t::underlying));
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 116-127
```cpp
116:       data_index_step(b, nbatch, h2, output_height, w2, output_width);
117:     }
118:   });
119: }
120:
121: template <nn_compute_source_index_fn_t nn_compute_source_index_fn>
122: static Tensor _upsample_nearest2d_quantized_cpu(
123:     const Tensor& input,
124:     IntArrayRef output_size,
125:     std::optional<double> scales_h,
126:     std::optional<double> scales_w) {
127:   auto full_output_size = native::upsample_2d_common_check(input.sizes(), output_size);
```
- EN: The main symbol in this range is `_upsample_nearest2d_quantized_cpu`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_upsample_nearest2d_quantized_cpu`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 129-140
```cpp
129:   // Allow for empty batch size but not other dimensions
130:   TORCH_CHECK(
131:       input.numel() != 0 || c10::multiply_integers(input.sizes().begin() + 1, input.sizes().end()),
132:       "Non-empty 4D data tensor expected but got a tensor with sizes ",
133:       input.sizes());
134:
135:   int64_t nbatch = full_output_size[0];
136:   int64_t channels = full_output_size[1];
137:   int64_t output_height = full_output_size[2];
138:   int64_t output_width = full_output_size[3];
139:   int64_t input_height = input.size(2);
140:   int64_t input_width = input.size(3);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 142-154
```cpp
142:   if (input.is_contiguous(c10::MemoryFormat::ChannelsLast)) {
143:     Tensor output = at::_empty_affine_quantized(
144:         {nbatch, channels, output_height, output_width},
145:         input.options().memory_format(input.suggest_memory_format()),
146:         input.q_scale(),
147:         input.q_zero_point(),
148:         std::nullopt);
149:
150:     // special case: just copy
151:     if (input_height == output_height && input_width == output_width) {
152:       output.copy_(input);
153:       return output;
154:     }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 156-166
```cpp
156:     AT_DISPATCH_QINT_TYPES(input.scalar_type(), "upsample_nearest2d", [&] {
157:       auto* idata = static_cast<scalar_t*>(input.data_ptr());
158:       auto* odata = static_cast<scalar_t*>(output.data_ptr());
159:       upsample_nearest2d_out_frame_nhwc<scalar_t, nn_compute_source_index_fn>(
160:           odata,
161:           idata,
162:           input_height,
163:           input_width,
164:           output_height,
165:           output_width,
166:           nbatch,
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 167-179
```cpp
167:           channels,
168:           scales_h,
169:           scales_w);
170:     });
171:     return output;
172:   } else {
173:     Tensor output = at::_empty_affine_quantized(
174:         {nbatch, channels, output_height, output_width},
175:         input.options(),
176:         input.q_scale(),
177:         input.q_zero_point());
178:
179:     auto input_contig = input.contiguous();
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 181-191
```cpp
181:     AT_DISPATCH_QINT_TYPES(input_contig.scalar_type(), "upsample_nearest2d", [&] {
182:       auto* idata = static_cast<scalar_t*>(input_contig.data_ptr());
183:       auto* odata = static_cast<scalar_t*>(output.data_ptr());
184:       upsample_nearest2d_out_frame<scalar_t, nn_compute_source_index_fn>(
185:           odata,
186:           idata,
187:           input_height,
188:           input_width,
189:           output_height,
190:           output_width,
191:           nbatch,
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 192-203
```cpp
192:           channels,
193:           scales_h,
194:           scales_w);
195:     });
196:     return output;
197:   }
198: }
199:
200: using at::native::upsample::compute_output_size;
201: using at::native::upsample::get_scale_value;
202:
203: Tensor upsample_nearest2d_quantized_cpu(
```
- EN: The main symbol in this range is `upsample_nearest2d_quantized_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `upsample_nearest2d_quantized_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 204-217
```cpp
204:     const Tensor& input,
205:     IntArrayRef osize,
206:     std::optional<double> scale_h,
207:     std::optional<double> scale_w) {
208:   return _upsample_nearest2d_quantized_cpu<nearest_neighbor_compute_source_index>(input, osize, scale_h, scale_w);
209: }
210:
211: Tensor _upsample_nearest_exact2d_quantized_cpu(
212:     const Tensor& input,
213:     IntArrayRef osize,
214:     std::optional<double> scale_h,
215:     std::optional<double> scale_w) {
216:   return _upsample_nearest2d_quantized_cpu<nearest_neighbor_exact_compute_source_index>(input, osize, scale_h, scale_w);
217: }
```
- EN: The main symbol in this range is `_upsample_nearest_exact2d_quantized_cpu`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `_upsample_nearest_exact2d_quantized_cpu`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 219-219
```cpp
219: } // namespace at::native
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

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/native/UpSample.h`, `ATen/native/cpu/utils.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/_empty_affine_quantized.h`, `ATen/ops/_upsample_nearest_exact2d_native.h`, `ATen/ops/upsample_nearest2d_native.h`
- c10 headers / c10 头文件: `c10/util/accumulate.h`, `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `cstring`, `vector`
- Key helper symbols / 关键辅助符号: `parallel_for`

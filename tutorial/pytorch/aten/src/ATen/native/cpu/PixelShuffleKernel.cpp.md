# PixelShuffleKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/PixelShuffleKernel.cpp`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares support for Pixel Shuffle Kernel in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了 Pixel Shuffle Kernel 的相关支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
 1: #define TORCH_ASSERT_NO_OPERATORS
 2: #include <ATen/native/cpu/PixelShuffleKernel.h>
 3:
 4: #include <ATen/core/TensorBase.h>
 5: #include <ATen/Dispatch.h>
 6: #include <ATen/Parallel.h>
 7: #include <ATen/native/cpu/utils.h>
 8: #include <ATen/cpu/vec/vec.h>
 9: #include <c10/util/irange.h>
10:
11: namespace at::native {
12:
13: namespace {
```
- EN: This range pulls in required headers, including `ATen/native/cpu/PixelShuffleKernel.h`, `ATen/core/TensorBase.h`, `ATen/Dispatch.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 这一段引入了所需头文件，例如 `ATen/native/cpu/PixelShuffleKernel.h`, `ATen/core/TensorBase.h`, `ATen/Dispatch.h`。 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

### Lines 15-30
```cpp
15: template <typename scalar_t>
16: void cpu_pixel_shuffle(
17:     TensorBase& output,
18:     const TensorBase& input,
19:     int64_t upscale_factor) {
20:   auto input_data = input.const_data_ptr<scalar_t>();
21:   auto output_data = output.data_ptr<scalar_t>();
22:
23:   // [(B1...Bn), C, H, W] => [N, C, H, W]
24:   int64_t channels = input.size(-3);
25:   int64_t height = input.size(-2);
26:   int64_t width = input.size(-1);
27:   int64_t sub_channels = channels / (upscale_factor * upscale_factor);
28:   int64_t numel = input.numel();
29:   int64_t nbatch = numel / (channels * height * width);
30:   int64_t S = upscale_factor;
```
- EN: The main symbol in this range is `cpu_pixel_shuffle`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `cpu_pixel_shuffle`，它们直接构成本文件的算子逻辑。

### Lines 32-43
```cpp
32:   // input strides
33:   int64_t stride_n = channels * height * width;
34:   int64_t stride_c = S * S * height * width;
35:   int64_t stride_s1 = S * height * width;
36:   int64_t stride_s2 = height * width;
37:   int64_t stride_h = width;
38:
39:   // input tensor shape of [n, c, s1, s2, h, w]
40:   // output tensor shape of [n, c, h, s1, w, s2]
41:   at::parallel_for(0, numel, 0, [&](int64_t begin, int64_t end) {
42:     int64_t n{0}, c{0}, h{0}, s1{0}, w{0}, s2{0};
43:     data_index_init(begin, n, nbatch, c, sub_channels, h, height, s1, S, w, width, s2, S);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 45-55
```cpp
45:     for (const auto i : c10::irange(begin, end)) {
46:       int64_t input_offset = n * stride_n + c * stride_c + s1 * stride_s1 +
47:           s2 * stride_s2 + h * stride_h + w;
48:       output_data[i] = c10::load(&input_data[input_offset]);
49:
50:       data_index_step(n, nbatch, c, sub_channels, h, height, s1, S, w, width, s2, S);
51:     }
52:   });
53: }
54:
55: template <typename scalar_t>
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 56-70
```cpp
56: void cpu_pixel_shuffle_channels_last(
57:     TensorBase& output,
58:     const TensorBase& input,
59:     int64_t upscale_factor) {
60:   TORCH_CHECK(input.ndimension() == 4,
61:               "pixel shuffle with channels last format supports tensors with 4 dims");
62:   auto input_data = input.const_data_ptr<scalar_t>();
63:   auto output_data = output.data_ptr<scalar_t>();
64:
65:   int64_t nbatch = input.size(0);
66:   int64_t channels = input.size(1);
67:   int64_t height = input.size(2);
68:   int64_t width = input.size(3);
69:   int64_t sub_channels = channels / (upscale_factor * upscale_factor);
70:   int64_t S = upscale_factor;
```
- EN: The main symbol in this range is `cpu_pixel_shuffle_channels_last`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `cpu_pixel_shuffle_channels_last`，它们直接构成本文件的算子逻辑。

### Lines 72-84
```cpp
72:   // input tensor shape of [n, h, w, c, s1, s2]
73:   // output tensor shape of [n, h, s1, w, s2, c]
74:   using Vec = vec::Vectorized<scalar_t>;
75:   at::parallel_for(0, nbatch * height, 0, [&](int64_t begin, int64_t end) {
76:     // temp buffer holding each channel lane
77:     auto buffer = std::make_unique<scalar_t []>(channels);
78:     scalar_t* buffer_ptr = buffer.get();
79:
80:     int64_t n{0}, h{0};
81:     data_index_init(begin, n, nbatch, h, height);
82:     for (const auto i : c10::irange(begin, end)) {
83:       for (const auto w : c10::irange(width)) {
84:         const scalar_t* input_ptr = input_data + n * height * width * channels + h * width * channels + w * channels;
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors. Vectorized/SIMD execution is used to process multiple values per iteration. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。 这里使用向量化/SIMD 执行，在一次迭代中处理多个数据。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 86-96
```cpp
86:         // step 1: transpose each channel lane
87:         //   from: [c, s1*s2]
88:         //   to:   [s1*s2, c]
89:         utils::transpose(sub_channels, S * S, input_ptr, S * S, buffer_ptr, sub_channels);
90:
91:         // step 2: copy from temp buffer to output
92:         for (const auto s1 : c10::irange(S)) {
93:           scalar_t* x_ptr = buffer_ptr + s1 * S * sub_channels;
94:           scalar_t* y_ptr = output_data + i * width * channels + s1 * width * S * sub_channels + w * S * sub_channels;
95:
96:           int64_t size = S * sub_channels;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 97-111
```cpp
 97:           int64_t d = 0;
 98:           for (; d < size - (size % Vec::size()); d += Vec::size()) {
 99:             Vec data_vec = Vec::loadu(x_ptr + d);
100:             data_vec.store(y_ptr + d);
101:           }
102:           for (; d < size; d++) {
103:             y_ptr[d] = x_ptr[d];
104:           }
105:         }
106:       }
107:
108:       data_index_step(n, nbatch, h, height);
109:     }
110:   });
111: }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 113-128
```cpp
113: template <typename scalar_t>
114: void cpu_pixel_unshuffle(
115:     TensorBase& output,
116:     const TensorBase& input,
117:     int64_t downscale_factor) {
118:   auto input_data = input.const_data_ptr<scalar_t>();
119:   auto output_data = output.data_ptr<scalar_t>();
120:
121:   // [(B1...Bn), C, H, W] => [N, C, H, W]
122:   int64_t sub_channels = input.size(-3);
123:   int64_t height = input.size(-2) / downscale_factor;
124:   int64_t width = input.size(-1) / downscale_factor;
125:   int64_t channels = sub_channels * downscale_factor * downscale_factor;
126:   int64_t numel = input.numel();
127:   int64_t nbatch = numel / (channels * height * width);
128:   int64_t S = downscale_factor;
```
- EN: The main symbol in this range is `cpu_pixel_unshuffle`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `cpu_pixel_unshuffle`，它们直接构成本文件的算子逻辑。

### Lines 130-142
```cpp
130:   // input strides
131:   int64_t stride_n = channels * height * width;
132:   int64_t stride_c = height * S * width * S;
133:   int64_t stride_h = S * width * S;
134:   int64_t stride_s1 = width * S;
135:   int64_t stride_w = S;
136:   int64_t stride_s2 = 1;
137:
138:   // input tensor shape of [n, c, h, s1, w, s2]
139:   // output tensor shape of [n, c, s1, s2, h, w]
140:   at::parallel_for(0, numel, 0, [&](int64_t begin, int64_t end) {
141:     int64_t n{0}, c{0}, s1{0}, s2{0}, h{0}, w{0};
142:     data_index_init(begin, n, nbatch, c, sub_channels, s1, S, s2, S, h, height, w, width);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 144-154
```cpp
144:     for (const auto i : c10::irange(begin, end)) {
145:       int64_t input_offset = n * stride_n + c * stride_c + h * stride_h +
146:           s1 * stride_s1 + w * stride_w + s2 * stride_s2;
147:       output_data[i] = c10::load(&input_data[input_offset]);
148:
149:       data_index_step(n, nbatch, c, sub_channels, s1, S, s2, S, h, height, w, width);
150:     }
151:   });
152: }
153:
154: template <typename scalar_t>
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 155-170
```cpp
155: void cpu_pixel_unshuffle_channels_last(
156:     TensorBase& output,
157:     const TensorBase& input,
158:     int64_t downscale_factor) {
159:   TORCH_CHECK(input.ndimension() == 4,
160:               "pixel unshuffle with channels last format supports tensors with 4 dims");
161:   auto input_data = input.const_data_ptr<scalar_t>();
162:   auto output_data = output.data_ptr<scalar_t>();
163:
164:   int64_t nbatch = input.size(0);
165:   int64_t sub_channels = input.size(1);
166:   int64_t height = input.size(2) / downscale_factor;
167:   int64_t width = input.size(3) / downscale_factor;
168:   int64_t channels = sub_channels * downscale_factor * downscale_factor;
169:   int64_t numel = input.numel();
170:   int64_t S = downscale_factor;
```
- EN: The main symbol in this range is `cpu_pixel_unshuffle_channels_last`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `cpu_pixel_unshuffle_channels_last`，它们直接构成本文件的算子逻辑。

### Lines 172-184
```cpp
172:   // input strides
173:   int64_t stride_n = height * width * channels;
174:   int64_t stride_h = S * width * S * sub_channels;
175:   int64_t stride_s1 = width * S * sub_channels;
176:   int64_t stride_w = S * sub_channels;
177:   int64_t stride_s2 = sub_channels;
178:   int64_t stride_c = 1;
179:
180:   // input tensor shape of [n, h, s1, w, s2, c]
181:   // output tensor shape of [n, h, w, c, s1, s2]
182:   at::parallel_for(0, numel, 0, [&](int64_t begin, int64_t end) {
183:     int64_t n{0}, h{0}, w{0}, c{0}, s1{0}, s2{0};
184:     data_index_init(begin, n, nbatch, h, height, w, width, c, sub_channels, s1, S, s2, S);
```
- EN: Work is split across CPU threads with `parallel_for` to improve throughput on large tensors.
- CN: 通过 `parallel_for` 将工作划分到多个 CPU 线程，以提升大张量场景下的吞吐量。

### Lines 186-196
```cpp
186:     for (const auto i : c10::irange(begin, end)) {
187:       int64_t input_offset = n * stride_n + h * stride_h + s1 * stride_s1 +
188:           w * stride_w + s2 * stride_s2 + c * stride_c;
189:       output_data[i] = c10::load(&input_data[input_offset]);
190:
191:       data_index_step(n, nbatch, h, height, w, width, c, sub_channels, s1, S, s2, S);
192:     }
193:   });
194: }
195:
196: void pixel_shuffle_kernel_impl(
```
- EN: The main symbol in this range is `pixel_shuffle_kernel_impl`, which contributes directly to this file's operator logic. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 这一段的主要符号是 `pixel_shuffle_kernel_impl`，它们直接构成本文件的算子逻辑。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 197-208
```cpp
197:     TensorBase& output,
198:     const TensorBase& input,
199:     int64_t upscale_factor) {
200:   switch (input.suggest_memory_format()) {
201:     case at::MemoryFormat::Contiguous: {
202:       AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(ScalarType::Bool, ScalarType::BFloat16, ScalarType::Half,
203:           input.scalar_type(), "pixel_shuffle", [&] {
204:         cpu_pixel_shuffle<scalar_t>(output, input, upscale_factor);
205:       });
206:       break;
207:     }
208:     case at::MemoryFormat::ChannelsLast: {
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 209-220
```cpp
209:       AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(ScalarType::Bool, ScalarType::BFloat16, ScalarType::Half,
210:           input.scalar_type(), "pixel_shuffle_channels_last", [&] {
211:         cpu_pixel_shuffle_channels_last<scalar_t>(output, input, upscale_factor);
212:       });
213:       break;
214:     }
215:     default:
216:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
217:   }
218: }
219:
220: void pixel_unshuffle_kernel_impl(
```
- EN: The main symbol in this range is `pixel_unshuffle_kernel_impl`, which contributes directly to this file's operator logic. Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 这一段的主要符号是 `pixel_unshuffle_kernel_impl`，它们直接构成本文件的算子逻辑。 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 221-232
```cpp
221:     TensorBase& output,
222:     const TensorBase& input,
223:     int64_t downscale_factor) {
224:   switch (input.suggest_memory_format()) {
225:     case at::MemoryFormat::Contiguous: {
226:       // input tensor shape of [N, C, Hr, Wr]
227:       // output tensor shape of [N, Crr, H, W]
228:       AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(ScalarType::Bool, ScalarType::BFloat16, ScalarType::Half,
229:           input.scalar_type(), "pixel_unshuffle", [&] {
230:         cpu_pixel_unshuffle<scalar_t>(output, input, downscale_factor);
231:       });
232:       break;
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 233-246
```cpp
233:     }
234:     case at::MemoryFormat::ChannelsLast: {
235:       // input tensor shape of [N, Hr, Wr, C]
236:       // output tensor shape of [N, H, W, Crr]
237:       AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND3(ScalarType::Bool, ScalarType::BFloat16, ScalarType::Half,
238:           input.scalar_type(), "pixel_unshuffle_channels_last", [&] {
239:         cpu_pixel_unshuffle_channels_last<scalar_t>(output, input, downscale_factor);
240:       });
241:       break;
242:     }
243:     default:
244:       TORCH_CHECK(false, "Unsupported memory format. Supports only ChannelsLast, Contiguous");
245:   }
246: }
```
- EN: Runtime dispatch chooses specialized implementations based on dtype or quantized scalar kind.
- CN: 运行时分派会根据数据类型或量化标量类别选择专用实现。

### Lines 248-253
```cpp
248: } // anonymous namespace
249:
250: REGISTER_DISPATCH(pixel_shuffle_kernel, &pixel_shuffle_kernel_impl)
251: REGISTER_DISPATCH(pixel_unshuffle_kernel, &pixel_unshuffle_kernel_impl)
252:
253: } // at::native
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Runtime dtype dispatch / 运行时数据类型分派
- SIMD vectorization / SIMD 向量化
- CPU parallelism / CPU 并行
- Dispatcher registration / 调度器注册
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/native/cpu/PixelShuffleKernel.h`, `ATen/core/TensorBase.h`, `ATen/Dispatch.h`, `ATen/Parallel.h`, `ATen/native/cpu/utils.h`, `ATen/cpu/vec/vec.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Key helper symbols / 关键辅助符号: `TensorBase`, `Vectorized`, `parallel_for`, `REGISTER_DISPATCH`, `AT_DISPATCH_ALL_TYPES`, `Scalar`, `ScalarType`

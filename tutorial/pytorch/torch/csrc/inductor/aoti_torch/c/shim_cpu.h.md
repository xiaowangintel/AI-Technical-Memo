# shim_cpu.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/inductor/aoti_torch/c/shim_cpu.h`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements AOTInductor torch-side bridge code for converting handles, dispatching calls, and surfacing runtime services.
- 目的 (CN): 实现 AOTInductor 的 torch 侧桥接代码，用于句柄转换、调用分发与运行时服务暴露。
- Lines: 267
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
1: #ifndef AOTI_TORCH_SHIM_CPU
2: #define AOTI_TORCH_SHIM_CPU
3: 
4: #include <ATen/Config.h>
5: #include <torch/csrc/inductor/aoti_torch/c/shim.h>
6: 
7: #ifdef __cplusplus
8: extern "C" {
```

- EN: These lines pull in dependencies such as `ATen/Config.h`, `torch/csrc/inductor/aoti_torch/c/shim.h`, establishing the headers needed by the implementation. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/Config.h`, `torch/csrc/inductor/aoti_torch/c/shim.h`，为后续实现建立所需的头文件基础。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 9-16

```cpp
 9: #endif
10: 
11: #if AT_MKLDNN_ENABLED()
12: 
13: AOTI_TORCH_EXPORT AOTITorchError
14: aoti_torch_cpu_mkldnn__convolution_pointwise_binary(
15:     AtenTensorHandle X,
16:     AtenTensorHandle other,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_mkldnn__convolution_pointwise_binary`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_mkldnn__convolution_pointwise_binary` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 17-24

```cpp
17:     AtenTensorHandle W,
18:     AtenTensorHandle* B,
19:     const int64_t* padding,
20:     int64_t padding_len_,
21:     const int64_t* stride,
22:     int64_t stride_len_,
23:     const int64_t* dilation,
24:     int64_t dilation_len_,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 25-32

```cpp
25:     int64_t groups,
26:     const char* binary_attr,
27:     double* alpha,
28:     const char** unary_attr,
29:     const double** unary_scalars,
30:     int64_t unary_scalars_len_,
31:     const char** unary_algorithm,
32:     AtenTensorHandle* ret0);
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 33-40

```cpp
33: 
34: AOTI_TORCH_EXPORT AOTITorchError
35: aoti_torch_cpu_mkldnn__convolution_pointwise_binary_(
36:     AtenTensorHandle other,
37:     AtenTensorHandle X,
38:     AtenTensorHandle W,
39:     AtenTensorHandle* B,
40:     const int64_t* padding,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_mkldnn__convolution_pointwise_binary_`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_mkldnn__convolution_pointwise_binary_` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 41-48

```cpp
41:     int64_t padding_len_,
42:     const int64_t* stride,
43:     int64_t stride_len_,
44:     const int64_t* dilation,
45:     int64_t dilation_len_,
46:     int64_t groups,
47:     const char* binary_attr,
48:     double* alpha,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 49-56

```cpp
49:     const char** unary_attr,
50:     const double** unary_scalars,
51:     int64_t unary_scalars_len_,
52:     const char** unary_algorithm,
53:     AtenTensorHandle* ret0);
54: 
55: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu_mkldnn__convolution_pointwise(
56:     AtenTensorHandle X,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_mkldnn__convolution_pointwise`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_mkldnn__convolution_pointwise` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 57-64

```cpp
57:     AtenTensorHandle W,
58:     AtenTensorHandle* B,
59:     const int64_t* padding,
60:     int64_t padding_len_,
61:     const int64_t* stride,
62:     int64_t stride_len_,
63:     const int64_t* dilation,
64:     int64_t dilation_len_,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 65-72

```cpp
65:     int64_t groups,
66:     const char* attr,
67:     const double** scalars,
68:     int64_t scalars_len_,
69:     const char** algorithm,
70:     AtenTensorHandle* ret0);
71: 
72: AOTI_TORCH_EXPORT AOTITorchError
```

- EN: The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 73-80

```cpp
73: aoti_torch_cpu_mkldnn__convolution_transpose_pointwise(
74:     AtenTensorHandle X,
75:     AtenTensorHandle W,
76:     AtenTensorHandle* B,
77:     const int64_t* padding,
78:     int64_t padding_len_,
79:     const int64_t* output_padding,
80:     int64_t output_padding_len_,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_mkldnn__convolution_transpose_pointwise`.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_mkldnn__convolution_transpose_pointwise` 等函数/方法承载。
### Lines 81-88

```cpp
81:     const int64_t* stride,
82:     int64_t stride_len_,
83:     const int64_t* dilation,
84:     int64_t dilation_len_,
85:     int64_t groups,
86:     const char* attr,
87:     const double** scalars,
88:     int64_t scalars_len_,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 89-96

```cpp
89:     const char** algorithm,
90:     AtenTensorHandle* ret0);
91: 
92: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu_mkldnn_rnn_layer(
93:     AtenTensorHandle input,
94:     AtenTensorHandle weight0,
95:     AtenTensorHandle weight1,
96:     AtenTensorHandle weight2,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_mkldnn_rnn_layer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_mkldnn_rnn_layer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 97-104

```cpp
 97:     AtenTensorHandle weight3,
 98:     AtenTensorHandle hx_,
 99:     AtenTensorHandle cx_,
100:     int32_t reverse,
101:     const int64_t* batch_sizes,
102:     int64_t batch_sizes_len_,
103:     int64_t mode,
104:     int64_t hidden_size,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 105-112

```cpp
105:     int64_t num_layers,
106:     int32_t has_biases,
107:     int32_t bidirectional,
108:     int32_t batch_first,
109:     int32_t train,
110:     AtenTensorHandle* ret0,
111:     AtenTensorHandle* ret1,
112:     AtenTensorHandle* ret2,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 113-120

```cpp
113:     AtenTensorHandle* ret3);
114: 
115: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__linear_pointwise(
116:     AtenTensorHandle X,
117:     AtenTensorHandle W,
118:     AtenTensorHandle* B,
119:     const char* attr,
120:     const double** scalars,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__linear_pointwise`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__linear_pointwise` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 121-128

```cpp
121:     int64_t scalars_len_,
122:     const char** algorithm,
123:     AtenTensorHandle* ret0);
124: 
125: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__linear_pointwise_binary(
126:     AtenTensorHandle X,
127:     AtenTensorHandle other,
128:     AtenTensorHandle W,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__linear_pointwise_binary`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__linear_pointwise_binary` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 129-136

```cpp
129:     AtenTensorHandle* B,
130:     const char* attr,
131:     AtenTensorHandle* ret0);
132: 
133: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__qlinear_pointwise_tensor(
134:     AtenTensorHandle X,
135:     AtenTensorHandle act_scale,
136:     AtenTensorHandle act_zero_point,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__qlinear_pointwise_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__qlinear_pointwise_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 137-144

```cpp
137:     AtenTensorHandle onednn_weight,
138:     AtenTensorHandle weight_scales,
139:     AtenTensorHandle weight_zero_points,
140:     AtenTensorHandle* B,
141:     double output_scale,
142:     int64_t output_zero_point,
143:     const int32_t* output_dtype,
144:     const char* post_op_name,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 145-152

```cpp
145:     const double** post_op_args,
146:     int64_t post_op_args_len_,
147:     const char* post_op_algorithm,
148:     AtenTensorHandle* ret0);
149: 
150: AOTI_TORCH_EXPORT AOTITorchError
151: aoti_torch_cpu__qlinear_pointwise_binary_tensor(
152:     AtenTensorHandle X,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__qlinear_pointwise_binary_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__qlinear_pointwise_binary_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 153-160

```cpp
153:     AtenTensorHandle act_scale,
154:     AtenTensorHandle act_zero_point,
155:     AtenTensorHandle onednn_weight,
156:     AtenTensorHandle weight_scales,
157:     AtenTensorHandle weight_zero_points,
158:     AtenTensorHandle* other,
159:     AtenTensorHandle* B,
160:     double output_scale,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 161-168

```cpp
161:     int64_t output_zero_point,
162:     const int32_t* output_dtype,
163:     double other_scale,
164:     int64_t other_zero_point,
165:     const char* binary_post_op,
166:     double binary_alpha,
167:     const char* unary_post_op,
168:     const double** unary_post_op_args,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 169-176

```cpp
169:     int64_t unary_post_op_args_len_,
170:     const char* unary_post_op_algorithm,
171:     AtenTensorHandle* ret0);
172: 
173: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__qconv_pointwise_tensor(
174:     AtenTensorHandle X,
175:     AtenTensorHandle act_scale,
176:     AtenTensorHandle act_zero_point,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__qconv_pointwise_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__qconv_pointwise_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 177-184

```cpp
177:     AtenTensorHandle onednn_weight,
178:     AtenTensorHandle weight_scales,
179:     AtenTensorHandle weight_zero_points,
180:     AtenTensorHandle* B,
181:     const int64_t* stride,
182:     int64_t stride_len_,
183:     const int64_t* padding,
184:     int64_t padding_len_,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 185-192

```cpp
185:     const int64_t* dilation,
186:     int64_t dilation_len_,
187:     int64_t groups,
188:     double output_scale,
189:     int64_t output_zero_point,
190:     const int32_t* output_dtype,
191:     const char* attr,
192:     const double** post_op_args,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 193-200

```cpp
193:     int64_t post_op_args_len_,
194:     const char** algorithm,
195:     AtenTensorHandle* ret0);
196: 
197: AOTI_TORCH_EXPORT AOTITorchError
198: aoti_torch_cpu__qconv2d_pointwise_binary_tensor(
199:     AtenTensorHandle X,
200:     AtenTensorHandle act_scale,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__qconv2d_pointwise_binary_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__qconv2d_pointwise_binary_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 201-208

```cpp
201:     AtenTensorHandle act_zero_point,
202:     AtenTensorHandle onednn_weight,
203:     AtenTensorHandle weight_scales,
204:     AtenTensorHandle weight_zero_points,
205:     AtenTensorHandle accum,
206:     AtenTensorHandle* B,
207:     const int64_t* stride_args,
208:     int64_t stride_len_,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 209-216

```cpp
209:     const int64_t* padding_args,
210:     int64_t padding_len_,
211:     const int64_t* dilation_args,
212:     int64_t dilation_len_,
213:     int64_t groups,
214:     double output_scale,
215:     int64_t output_zero_point,
216:     const int32_t* output_dtype,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 217-224

```cpp
217:     double accum_scale,
218:     int64_t accum_zero_point,
219:     const char* binary_attr,
220:     double* alpha,
221:     const char** unary_attr,
222:     const double** unary_scalars,
223:     int64_t unary_scalars_len_,
224:     const char** unary_algorithm,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 225-232

```cpp
225:     AtenTensorHandle* ret0);
226: 
227: #if AT_MKL_ENABLED()
228: 
229: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__mkl_linear(
230:     AtenTensorHandle X,
231:     AtenTensorHandle W,
232:     AtenTensorHandle origin_W,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__mkl_linear`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__mkl_linear` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 233-240

```cpp
233:     AtenTensorHandle* B,
234:     int64_t prepack_batch_size,
235:     AtenTensorHandle* ret0);
236: 
237: #endif // AT_MKL_ENABLED
238: 
239: #endif // AT_MKLDNN_ENABLED()
240: 
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 241-248

```cpp
241: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__weight_int4pack_mm_cpu_tensor(
242:     AtenTensorHandle X,
243:     AtenTensorHandle w,
244:     AtenTensorHandle qGroupSize,
245:     AtenTensorHandle qScaleAndZeros,
246:     AtenTensorHandle* ret0);
247: 
248: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__c10d_functional_all_reduce_(
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__weight_int4pack_mm_cpu_tensor`, `aoti_torch_cpu__c10d_functional_all_reduce_`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__weight_int4pack_mm_cpu_tensor`, `aoti_torch_cpu__c10d_functional_all_reduce_` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 249-256

```cpp
249:     AtenTensorHandle inp,
250:     const char* reduce_op,
251:     const char* group_name,
252:     AtenTensorHandle* ret0);
253: 
254: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__c10d_functional_all_reduce(
255:     AtenTensorHandle inp,
256:     const char* reduce_op,
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__c10d_functional_all_reduce`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__c10d_functional_all_reduce` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 257-264

```cpp
257:     const char* group_name,
258:     AtenTensorHandle* ret0);
259: 
260: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_cpu__c10d_functional_wait_tensor(
261:     AtenTensorHandle inp,
262:     AtenTensorHandle* ret0);
263: 
264: #ifdef __cplusplus
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu__c10d_functional_wait_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu__c10d_functional_wait_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 265-267

```cpp
265: } // extern "C"
266: #endif
267: #endif // AOTI_TORCH_SHIM_CPU
```

- EN: The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。

## Key Concepts / 关键概念
- Inductor / AOTInductor runtime integration / Inductor / AOTInductor 运行时集成
- Tensor/value representation management / 张量/数值表示管理
- Stable ABI / C interface exposure / 稳定 ABI / C 接口暴露
- Primary symbol `aoti_torch_cpu_mkldnn__convolution_pointwise_binary` / 核心符号 `aoti_torch_cpu_mkldnn__convolution_pointwise_binary`
- Primary symbol `aoti_torch_cpu_mkldnn__convolution_pointwise_binary_` / 核心符号 `aoti_torch_cpu_mkldnn__convolution_pointwise_binary_`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `ATen/Config.h`, `torch/csrc/inductor/aoti_torch/c/shim.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `aoti_torch_cpu_mkldnn__convolution_pointwise_binary`, `aoti_torch_cpu_mkldnn__convolution_pointwise_binary_`, `aoti_torch_cpu_mkldnn__convolution_pointwise`, `aoti_torch_cpu_mkldnn__convolution_transpose_pointwise`, `aoti_torch_cpu_mkldnn_rnn_layer`, `aoti_torch_cpu__linear_pointwise`, `aoti_torch_cpu__linear_pointwise_binary`, `aoti_torch_cpu__qlinear_pointwise_tensor`, `aoti_torch_cpu__qlinear_pointwise_binary_tensor`, `aoti_torch_cpu__qconv_pointwise_tensor`
- Related subsystems / 相关子系统: ATen operator layer / ATen 算子层, Inductor runtime / Inductor 运行时

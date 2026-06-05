# shim_xpu.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/inductor/aoti_torch/c/shim_xpu.h`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements AOTInductor torch-side bridge code for converting handles, dispatching calls, and surfacing runtime services.
- 目的 (CN): 实现 AOTInductor 的 torch 侧桥接代码，用于句柄转换、调用分发与运行时服务暴露。
- Lines: 210
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
1: #ifndef AOTI_TORCH_SHIM_XPU
2: #define AOTI_TORCH_SHIM_XPU
3: 
4: #include <ATen/Config.h>
5: #include <torch/csrc/inductor/aoti_torch/c/shim.h>
6: 
7: #ifdef USE_XPU
8: #ifdef __cplusplus
```

- EN: These lines pull in dependencies such as `ATen/Config.h`, `torch/csrc/inductor/aoti_torch/c/shim.h`, establishing the headers needed by the implementation. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/Config.h`, `torch/csrc/inductor/aoti_torch/c/shim.h`，为后续实现建立所需的头文件基础。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 9-16

```cpp
 9: extern "C" {
10: #endif
11: 
12: struct XPUGuardOpaque;
13: using XPUGuardHandle = XPUGuardOpaque*;
14: 
15: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_create_xpu_guard(
16:     int32_t device_index,
```

- EN: This range declares or shapes types such as `XPUGuardOpaque`. The main execution path in this span is carried by `aoti_torch_create_xpu_guard`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段声明或塑造了 ``XPUGuardOpaque`` 等类型。 这一段的主要执行路径由 `aoti_torch_create_xpu_guard` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 17-24

```cpp
17:     XPUGuardHandle* ret_guard // returns new reference
18: );
19: 
20: AOTI_TORCH_EXPORT AOTITorchError
21: aoti_torch_delete_xpu_guard(XPUGuardHandle guard);
22: 
23: AOTI_TORCH_EXPORT AOTITorchError
24: aoti_torch_xpu_guard_set_index(XPUGuardHandle guard, int32_t device_index);
```

- EN: The main execution path in this span is carried by `aoti_torch_delete_xpu_guard`, `aoti_torch_xpu_guard_set_index`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_delete_xpu_guard`, `aoti_torch_xpu_guard_set_index` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 25-32

```cpp
25: 
26: struct XPUStreamGuardOpaque;
27: using XPUStreamGuardHandle = XPUStreamGuardOpaque*;
28: 
29: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_create_xpu_stream_guard(
30:     void* stream,
31:     int32_t device_index,
32:     XPUStreamGuardHandle* ret_guard // returns new reference
```

- EN: This range declares or shapes types such as `XPUStreamGuardOpaque`. The main execution path in this span is carried by `aoti_torch_create_xpu_stream_guard`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段声明或塑造了 ``XPUStreamGuardOpaque`` 等类型。 这一段的主要执行路径由 `aoti_torch_create_xpu_stream_guard` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 33-40

```cpp
33: );
34: 
35: AOTI_TORCH_EXPORT AOTITorchError
36: aoti_torch_delete_xpu_stream_guard(XPUStreamGuardHandle guard);
37: 
38: AOTI_TORCH_EXPORT AOTITorchError
39: aoti_torch_get_current_xpu_stream(int32_t device_index, void** ret_stream);
40: 
```

- EN: The main execution path in this span is carried by `aoti_torch_delete_xpu_stream_guard`, `aoti_torch_get_current_xpu_stream`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_delete_xpu_stream_guard`, `aoti_torch_get_current_xpu_stream` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 41-48

```cpp
41: AOTI_TORCH_EXPORT AOTITorchError
42: aoti_torch_get_current_xpu_device(int32_t* device_index);
43: 
44: AOTI_TORCH_EXPORT AOTITorchError
45: aoti_torch_set_current_xpu_device(const int32_t& device_index);
46: 
47: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_get_current_sycl_queue(void** ret);
48: 
```

- EN: The main execution path in this span is carried by `aoti_torch_get_current_xpu_device`, `aoti_torch_set_current_xpu_device`, `aoti_torch_get_current_sycl_queue`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_get_current_xpu_device`, `aoti_torch_set_current_xpu_device`, `aoti_torch_get_current_sycl_queue` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 49-56

```cpp
49: #if AT_MKLDNN_ENABLED()
50: 
51: AOTI_TORCH_EXPORT AOTITorchError
52: aoti_torch_xpu_mkldnn__convolution_pointwise_binary(
53:     AtenTensorHandle X,
54:     AtenTensorHandle other,
55:     AtenTensorHandle W,
56:     AtenTensorHandle* B,
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu_mkldnn__convolution_pointwise_binary`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu_mkldnn__convolution_pointwise_binary` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 57-64

```cpp
57:     const int64_t* padding,
58:     int64_t padding_len_,
59:     const int64_t* stride,
60:     int64_t stride_len_,
61:     const int64_t* dilation,
62:     int64_t dilation_len_,
63:     int64_t groups,
64:     const char* binary_attr,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 65-72

```cpp
65:     double* alpha,
66:     const char** unary_attr,
67:     const double** unary_scalars,
68:     int64_t unary_scalars_len_,
69:     const char** unary_algorithm,
70:     AtenTensorHandle* ret0);
71: 
72: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_xpu_mkldnn__convolution_pointwise(
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu_mkldnn__convolution_pointwise`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu_mkldnn__convolution_pointwise` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 73-80

```cpp
73:     AtenTensorHandle X,
74:     AtenTensorHandle W,
75:     AtenTensorHandle* B,
76:     const int64_t* padding,
77:     int64_t padding_len_,
78:     const int64_t* stride,
79:     int64_t stride_len_,
80:     const int64_t* dilation,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 81-88

```cpp
81:     int64_t dilation_len_,
82:     int64_t groups,
83:     const char* attr,
84:     const double** scalars,
85:     int64_t scalars_len_,
86:     const char** algorithm,
87:     AtenTensorHandle* ret0);
88: 
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 89-96

```cpp
89: AOTI_TORCH_EXPORT AOTITorchError
90: aoti_torch_xpu_mkldnn__convolution_pointwise_binary_(
91:     AtenTensorHandle other,
92:     AtenTensorHandle X,
93:     AtenTensorHandle W,
94:     AtenTensorHandle* B,
95:     const int64_t* padding,
96:     int64_t padding_len_,
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu_mkldnn__convolution_pointwise_binary_`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu_mkldnn__convolution_pointwise_binary_` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 97-104

```cpp
 97:     const int64_t* stride,
 98:     int64_t stride_len_,
 99:     const int64_t* dilation,
100:     int64_t dilation_len_,
101:     int64_t groups,
102:     const char* binary_attr,
103:     double* alpha,
104:     const char** unary_attr,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 105-112

```cpp
105:     const double** unary_scalars,
106:     int64_t unary_scalars_len_,
107:     const char** unary_algorithm,
108:     AtenTensorHandle* ret0);
109: 
110: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_xpu__qlinear_pointwise_tensor(
111:     AtenTensorHandle X,
112:     AtenTensorHandle act_scale,
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu__qlinear_pointwise_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu__qlinear_pointwise_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 113-120

```cpp
113:     AtenTensorHandle act_zero_point,
114:     AtenTensorHandle onednn_weight,
115:     AtenTensorHandle weight_scales,
116:     AtenTensorHandle weight_zero_points,
117:     AtenTensorHandle* B,
118:     double output_scale,
119:     int64_t output_zero_point,
120:     const int32_t* output_dtype,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 121-128

```cpp
121:     const char* post_op_name,
122:     const double** post_op_args,
123:     int64_t post_op_args_len_,
124:     const char* post_op_algorithm,
125:     AtenTensorHandle* ret0);
126: 
127: AOTI_TORCH_EXPORT AOTITorchError
128: aoti_torch_xpu__qlinear_pointwise_binary_tensor(
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu__qlinear_pointwise_binary_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu__qlinear_pointwise_binary_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 129-136

```cpp
129:     AtenTensorHandle X,
130:     AtenTensorHandle act_scale,
131:     AtenTensorHandle act_zero_point,
132:     AtenTensorHandle onednn_weight,
133:     AtenTensorHandle weight_scales,
134:     AtenTensorHandle weight_zero_points,
135:     AtenTensorHandle* other,
136:     AtenTensorHandle* B,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 137-144

```cpp
137:     double output_scale,
138:     int64_t output_zero_point,
139:     const int32_t* output_dtype,
140:     double other_scale,
141:     int64_t other_zero_point,
142:     const char* binary_post_op,
143:     double binary_alpha,
144:     const char* unary_post_op,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 145-152

```cpp
145:     const double** unary_post_op_args,
146:     int64_t unary_post_op_args_len_,
147:     const char* unary_post_op_algorithm,
148:     AtenTensorHandle* ret0);
149: 
150: AOTI_TORCH_EXPORT AOTITorchError aoti_torch_xpu__qconv_pointwise_tensor(
151:     AtenTensorHandle X,
152:     AtenTensorHandle act_scale,
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu__qconv_pointwise_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu__qconv_pointwise_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 153-160

```cpp
153:     AtenTensorHandle act_zero_point,
154:     AtenTensorHandle onednn_weight,
155:     AtenTensorHandle weight_scales,
156:     AtenTensorHandle weight_zero_points,
157:     AtenTensorHandle* B,
158:     const int64_t* stride,
159:     int64_t stride_len_,
160:     const int64_t* padding,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 161-168

```cpp
161:     int64_t padding_len_,
162:     const int64_t* dilation,
163:     int64_t dilation_len_,
164:     int64_t groups,
165:     double output_scale,
166:     int64_t output_zero_point,
167:     const int32_t* output_dtype,
168:     const char* attr,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 169-176

```cpp
169:     const double** post_op_args,
170:     int64_t post_op_args_len_,
171:     const char** algorithm,
172:     AtenTensorHandle* ret0);
173: 
174: AOTI_TORCH_EXPORT AOTITorchError
175: aoti_torch_xpu__qconv2d_pointwise_binary_tensor(
176:     AtenTensorHandle X,
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu__qconv2d_pointwise_binary_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu__qconv2d_pointwise_binary_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 177-184

```cpp
177:     AtenTensorHandle act_scale,
178:     AtenTensorHandle act_zero_point,
179:     AtenTensorHandle onednn_weight,
180:     AtenTensorHandle weight_scales,
181:     AtenTensorHandle weight_zero_points,
182:     AtenTensorHandle accum,
183:     AtenTensorHandle* B,
184:     const int64_t* stride_args,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 185-192

```cpp
185:     int64_t stride_len_,
186:     const int64_t* padding_args,
187:     int64_t padding_len_,
188:     const int64_t* dilation_args,
189:     int64_t dilation_len_,
190:     int64_t groups,
191:     double output_scale,
192:     int64_t output_zero_point,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 193-200

```cpp
193:     const int32_t* output_dtype,
194:     double accum_scale,
195:     int64_t accum_zero_point,
196:     const char* binary_attr,
197:     double* alpha,
198:     const char** unary_attr,
199:     const double** unary_scalars,
200:     int64_t unary_scalars_len_,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 201-208

```cpp
201:     const char** unary_algorithm,
202:     AtenTensorHandle* ret0);
203: 
204: #endif // AT_MKLDNN_ENABLED()
205: #ifdef __cplusplus
206: } // extern "C"
207: #endif
208: 
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 209-210

```cpp
209: #endif // USE_XPU
210: #endif // AOTI_TORCH_SHIM_XPU
```

- EN: The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。

## Key Concepts / 关键概念
- Inductor / AOTInductor runtime integration / Inductor / AOTInductor 运行时集成
- Tensor/value representation management / 张量/数值表示管理
- Stable ABI / C interface exposure / 稳定 ABI / C 接口暴露
- Primary symbol `XPUGuardOpaque` / 核心符号 `XPUGuardOpaque`
- Primary symbol `XPUStreamGuardOpaque` / 核心符号 `XPUStreamGuardOpaque`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `ATen/Config.h`, `torch/csrc/inductor/aoti_torch/c/shim.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `XPUGuardOpaque`, `XPUStreamGuardOpaque`, `aoti_torch_create_xpu_guard`, `aoti_torch_delete_xpu_guard`, `aoti_torch_xpu_guard_set_index`, `aoti_torch_create_xpu_stream_guard`, `aoti_torch_delete_xpu_stream_guard`, `aoti_torch_get_current_xpu_stream`, `aoti_torch_get_current_xpu_device`, `aoti_torch_set_current_xpu_device`
- Related subsystems / 相关子系统: ATen operator layer / ATen 算子层, Inductor runtime / Inductor 运行时

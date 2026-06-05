# shim_xpu.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/inductor/aoti_torch/shim_xpu.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements AOTInductor torch-side bridge code for converting handles, dispatching calls, and surfacing runtime services.
- 目的 (CN): 实现 AOTInductor 的 torch 侧桥接代码，用于句柄转换、调用分发与运行时服务暴露。
- Lines: 432
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
 1: 
 2: #include <torch/csrc/inductor/aoti_torch/c/shim_xpu.h>
 3: #include <torch/csrc/inductor/aoti_torch/utils.h>
 4: 
 5: #include <c10/core/DeviceGuard.h>
 6: #include <c10/core/DeviceType.h>
 7: #include <c10/core/StreamGuard.h>
 8: #include <c10/xpu/XPUStream.h>
 9: 
10: using namespace torch::aot_inductor;
11: 
12: AOTITorchError aoti_torch_create_xpu_guard(
13:     int32_t device_index,
14:     XPUGuardHandle* ret_guard // returns new reference
15: ) {
16:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
```

- EN: These lines pull in dependencies such as `torch/csrc/inductor/aoti_torch/c/shim_xpu.h`, `torch/csrc/inductor/aoti_torch/utils.h`, `c10/core/DeviceGuard.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `aoti_torch_create_xpu_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `torch/csrc/inductor/aoti_torch/c/shim_xpu.h`, `torch/csrc/inductor/aoti_torch/utils.h`, `c10/core/DeviceGuard.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `aoti_torch_create_xpu_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 17-32

```cpp
17:     at::DeviceGuard* guard =
18:         new at::DeviceGuard(at::Device(at::DeviceType::XPU, device_index));
19:     *ret_guard = reinterpret_cast<XPUGuardHandle>(guard);
20:   });
21: }
22: 
23: AOTITorchError aoti_torch_delete_xpu_guard(XPUGuardHandle guard) {
24:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
25:       { delete reinterpret_cast<at::DeviceGuard*>(guard); });
26: }
27: 
28: AOTITorchError aoti_torch_xpu_guard_set_index(
29:     XPUGuardHandle guard,
30:     int32_t device_index) {
31:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
32:       { reinterpret_cast<at::DeviceGuard*>(guard)->set_index(device_index); });
```

- EN: The main execution path in this span is carried by `DeviceGuard`, `aoti_torch_delete_xpu_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `DeviceGuard`, `aoti_torch_delete_xpu_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 33-48

```cpp
33: }
34: 
35: AOTITorchError aoti_torch_create_xpu_stream_guard(
36:     void* stream,
37:     int32_t device_index,
38:     XPUStreamGuardHandle* ret_guard) {
39:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
40:     assert(stream);
41:     at::StreamGuard* guard =
42:         new at::StreamGuard(at::xpu::getStreamFromExternal(
43:                                 static_cast<sycl::queue*>(stream), device_index)
44:                                 .unwrap());
45:     *ret_guard = reinterpret_cast<XPUStreamGuardHandle>(guard);
46:   });
47: }
48: 
```

- EN: The main execution path in this span is carried by `aoti_torch_create_xpu_stream_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `assert`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_create_xpu_stream_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `assert` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 49-64

```cpp
49: AOTITorchError aoti_torch_delete_xpu_stream_guard(XPUStreamGuardHandle guard) {
50:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
51:       { delete reinterpret_cast<at::StreamGuard*>(guard); });
52: }
53: 
54: AOTITorchError aoti_torch_get_current_xpu_stream(
55:     int32_t device_index,
56:     void** ret_stream) {
57:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
58:       { *ret_stream = &(at::xpu::getCurrentXPUStream(device_index).queue()); });
59: }
60: 
61: AOTITorchError aoti_torch_get_current_xpu_device(int32_t* device_index) {
62:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
63:     *device_index =
64:         static_cast<int32_t>(static_cast<uint16_t>(c10::xpu::current_device()));
```

- EN: The main execution path in this span is carried by `aoti_torch_delete_xpu_stream_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `aoti_torch_get_current_xpu_stream`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_delete_xpu_stream_guard`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `aoti_torch_get_current_xpu_stream` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 65-80

```cpp
65:   });
66: }
67: 
68: AOTITorchError aoti_torch_set_current_xpu_device(const int32_t& device_index) {
69:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
70:       { c10::xpu::set_device(static_cast<int8_t>(device_index)); });
71: }
72: 
73: AOTITorchError aoti_torch_get_current_sycl_queue(void** ret) {
74:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
75:     int32_t device_index =
76:         static_cast<int32_t>(static_cast<uint16_t>(c10::xpu::current_device()));
77:     *ret = &(at::xpu::getCurrentXPUStream(device_index).queue());
78:   });
79: }
80: 
```

- EN: The main execution path in this span is carried by `aoti_torch_set_current_xpu_device`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `set_device`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_set_current_xpu_device`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `set_device` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 81-96

```cpp
81: #if AT_MKLDNN_ENABLED()
82: #include <ATen/native/mkldnn/xpu/Conv.h>
83: #include <ATen/native/mkldnn/xpu/qconv.h>
84: #include <ATen/native/mkldnn/xpu/qlinear.h>
85: 
86: AOTITorchError aoti_torch_xpu_mkldnn__convolution_pointwise_binary(
87:     AtenTensorHandle X,
88:     AtenTensorHandle other,
89:     AtenTensorHandle W,
90:     AtenTensorHandle* B,
91:     const int64_t* padding,
92:     int64_t padding_len_,
93:     const int64_t* stride,
94:     int64_t stride_len_,
95:     const int64_t* dilation,
96:     int64_t dilation_len_,
```

- EN: These lines pull in dependencies such as `ATen/native/mkldnn/xpu/Conv.h`, `ATen/native/mkldnn/xpu/qconv.h`, `ATen/native/mkldnn/xpu/qlinear.h`, establishing the headers needed by the implementation. The main execution path in this span is carried by `aoti_torch_xpu_mkldnn__convolution_pointwise_binary`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/native/mkldnn/xpu/Conv.h`, `ATen/native/mkldnn/xpu/qconv.h`, `ATen/native/mkldnn/xpu/qlinear.h`，为后续实现建立所需的头文件基础。 这一段的主要执行路径由 `aoti_torch_xpu_mkldnn__convolution_pointwise_binary` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 97-112

```cpp
 97:     int64_t groups,
 98:     const char* binary_attr,
 99:     double* alpha,
100:     const char** unary_attr,
101:     const double** unary_scalars,
102:     int64_t unary_scalars_len_,
103:     const char** unary_algorithm,
104:     AtenTensorHandle* ret0) {
105:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
106:     c10::List<std::optional<c10::Scalar>> unary_scalars_list;
107:     unary_scalars_list.reserve(unary_scalars_len_);
108:     for (int64_t i = 0; i < unary_scalars_len_; i++) {
109:       unary_scalars_list.emplace_back(pointer_to_optional(unary_scalars[i]));
110:     }
111:     auto tmp_result = at::native::xpu::convolution_pointwise_binary(
112:         *tensor_handle_to_tensor_pointer(X),
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `convolution_pointwise_binary`, `tensor_handle_to_tensor_pointer`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `convolution_pointwise_binary`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 113-128

```cpp
113:         *tensor_handle_to_tensor_pointer(other),
114:         *tensor_handle_to_tensor_pointer(W),
115:         pointer_to_optional<at::Tensor>(B),
116:         pointer_to_list<int64_t>(padding, padding_len_),
117:         pointer_to_list<int64_t>(stride, stride_len_),
118:         pointer_to_list<int64_t>(dilation, dilation_len_),
119:         groups,
120:         binary_attr,
121:         pointer_to_optional<c10::Scalar>(alpha),
122:         pointer_to_optional<std::string_view>(unary_attr),
123:         unary_scalars_list,
124:         pointer_to_optional<std::string_view>(unary_algorithm));
125:     *ret0 = new_tensor_handle(std::move(tmp_result));
126:   });
127: }
128: 
```

- EN: The main execution path in this span is carried by `tensor_handle_to_tensor_pointer`, `new_tensor_handle`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `tensor_handle_to_tensor_pointer`, `new_tensor_handle` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 129-144

```cpp
129: AOTITorchError aoti_torch_xpu_mkldnn__convolution_pointwise_binary_(
130:     AtenTensorHandle other,
131:     AtenTensorHandle X,
132:     AtenTensorHandle W,
133:     AtenTensorHandle* B,
134:     const int64_t* padding,
135:     int64_t padding_len_,
136:     const int64_t* stride,
137:     int64_t stride_len_,
138:     const int64_t* dilation,
139:     int64_t dilation_len_,
140:     int64_t groups,
141:     const char* binary_attr,
142:     double* alpha,
143:     const char** unary_attr,
144:     const double** unary_scalars,
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu_mkldnn__convolution_pointwise_binary_`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu_mkldnn__convolution_pointwise_binary_` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 145-160

```cpp
145:     int64_t unary_scalars_len_,
146:     const char** unary_algorithm,
147:     AtenTensorHandle* ret0) {
148:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
149:     c10::List<std::optional<c10::Scalar>> unary_scalars_list;
150:     unary_scalars_list.reserve(unary_scalars_len_);
151:     for (int64_t i = 0; i < unary_scalars_len_; i++) {
152:       unary_scalars_list.emplace_back(pointer_to_optional(unary_scalars[i]));
153:     }
154:     auto tmp_result = at::native::xpu::convolution_pointwise_binary_(
155:         *tensor_handle_to_tensor_pointer(other),
156:         *tensor_handle_to_tensor_pointer(X),
157:         *tensor_handle_to_tensor_pointer(W),
158:         pointer_to_optional<at::Tensor>(B),
159:         pointer_to_list<int64_t>(padding, padding_len_),
160:         pointer_to_list<int64_t>(stride, stride_len_),
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `convolution_pointwise_binary_`, `tensor_handle_to_tensor_pointer`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `convolution_pointwise_binary_`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 161-176

```cpp
161:         pointer_to_list<int64_t>(dilation, dilation_len_),
162:         groups,
163:         binary_attr,
164:         pointer_to_optional<c10::Scalar>(alpha),
165:         pointer_to_optional<std::string_view>(unary_attr),
166:         unary_scalars_list,
167:         pointer_to_optional<std::string_view>(unary_algorithm));
168:     *ret0 = new_tensor_handle(std::move(tmp_result));
169:   });
170: }
171: 
172: AOTITorchError aoti_torch_xpu_mkldnn__convolution_pointwise(
173:     AtenTensorHandle X,
174:     AtenTensorHandle W,
175:     AtenTensorHandle* B,
176:     const int64_t* padding,
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_xpu_mkldnn__convolution_pointwise`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_xpu_mkldnn__convolution_pointwise` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 177-192

```cpp
177:     int64_t padding_len_,
178:     const int64_t* stride,
179:     int64_t stride_len_,
180:     const int64_t* dilation,
181:     int64_t dilation_len_,
182:     int64_t groups,
183:     const char* attr,
184:     const double** scalars,
185:     int64_t scalars_len_,
186:     const char** algorithm,
187:     AtenTensorHandle* ret0) {
188:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
189:     c10::List<std::optional<c10::Scalar>> scalars_list;
190:     scalars_list.reserve(scalars_len_);
191:     for (int64_t i = 0; i < scalars_len_; i++) {
192:       scalars_list.emplace_back(pointer_to_optional(scalars[i]));
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 193-208

```cpp
193:     }
194:     auto tmp_result = at::native::xpu::convolution_pointwise(
195:         *tensor_handle_to_tensor_pointer(X),
196:         *tensor_handle_to_tensor_pointer(W),
197:         pointer_to_optional<at::Tensor>(B),
198:         pointer_to_list<int64_t>(padding, padding_len_),
199:         pointer_to_list<int64_t>(stride, stride_len_),
200:         pointer_to_list<int64_t>(dilation, dilation_len_),
201:         groups,
202:         attr,
203:         scalars_list,
204:         pointer_to_optional<std::string_view>(algorithm));
205:     *ret0 = new_tensor_handle(std::move(tmp_result));
206:   });
207: }
208: 
```

- EN: The main execution path in this span is carried by `convolution_pointwise`, `tensor_handle_to_tensor_pointer`, `new_tensor_handle`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `convolution_pointwise`, `tensor_handle_to_tensor_pointer`, `new_tensor_handle` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 209-224

```cpp
209: AOTITorchError aoti_torch_xpu__qlinear_pointwise_tensor(
210:     AtenTensorHandle X,
211:     AtenTensorHandle act_scale,
212:     AtenTensorHandle act_zero_point,
213:     AtenTensorHandle onednn_weight,
214:     AtenTensorHandle weight_scales,
215:     AtenTensorHandle weight_zero_points,
216:     AtenTensorHandle* B,
217:     double output_scale,
218:     int64_t output_zero_point,
219:     const int32_t* output_dtype,
220:     const char* post_op_name,
221:     const double** post_op_args,
222:     int64_t post_op_args_len_,
223:     const char* post_op_algorithm,
224:     AtenTensorHandle* ret0) {
```

- EN: The main execution path in this span is carried by `aoti_torch_xpu__qlinear_pointwise_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_xpu__qlinear_pointwise_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 225-240

```cpp
225:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
226:     c10::List<std::optional<c10::Scalar>> scalars_list;
227:     scalars_list.reserve(post_op_args_len_);
228:     for (int64_t i = 0; i < post_op_args_len_; i++) {
229:       scalars_list.emplace_back(pointer_to_optional(post_op_args[i]));
230:     }
231: 
232:     auto tmp_result =
233:         at::native::xpu::QLinearOnednnXPU::q_linear_pointwise_tensor(
234:             *tensor_handle_to_tensor_pointer(X),
235:             *tensor_handle_to_tensor_pointer(act_scale),
236:             *tensor_handle_to_tensor_pointer(act_zero_point),
237:             *tensor_handle_to_tensor_pointer(onednn_weight),
238:             *tensor_handle_to_tensor_pointer(weight_scales),
239:             *tensor_handle_to_tensor_pointer(weight_zero_points),
240:             pointer_to_optional<at::Tensor>(B),
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `q_linear_pointwise_tensor`, `tensor_handle_to_tensor_pointer`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `q_linear_pointwise_tensor`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 241-256

```cpp
241:             output_scale,
242:             output_zero_point,
243:             pointer_to_optional<at::ScalarType>(output_dtype),
244:             post_op_name,
245:             scalars_list,
246:             post_op_algorithm);
247:     *ret0 = new_tensor_handle(std::move(tmp_result));
248:   });
249: }
250: 
251: AOTITorchError aoti_torch_xpu__qlinear_pointwise_binary_tensor(
252:     AtenTensorHandle X,
253:     AtenTensorHandle act_scale,
254:     AtenTensorHandle act_zero_point,
255:     AtenTensorHandle onednn_weight,
256:     AtenTensorHandle weight_scales,
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_xpu__qlinear_pointwise_binary_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_xpu__qlinear_pointwise_binary_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 257-272

```cpp
257:     AtenTensorHandle weight_zero_points,
258:     AtenTensorHandle* other,
259:     AtenTensorHandle* B,
260:     double output_scale,
261:     int64_t output_zero_point,
262:     const int32_t* output_dtype,
263:     double other_scale,
264:     int64_t other_zero_point,
265:     const char* binary_post_op,
266:     double binary_alpha,
267:     const char* unary_post_op,
268:     const double** unary_post_op_args,
269:     int64_t unary_post_op_args_len_,
270:     const char* unary_post_op_algorithm,
271:     AtenTensorHandle* ret0) {
272:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 273-288

```cpp
273:     c10::List<std::optional<c10::Scalar>> scalars_list;
274:     scalars_list.reserve(unary_post_op_args_len_);
275:     for (int64_t i = 0; i < unary_post_op_args_len_; i++) {
276:       scalars_list.emplace_back(pointer_to_optional(unary_post_op_args[i]));
277:     }
278: 
279:     auto tmp_result =
280:         at::native::xpu::QLinearOnednnXPU::q_linear_pointwise_binary_tensor(
281:             *tensor_handle_to_tensor_pointer(X),
282:             *tensor_handle_to_tensor_pointer(act_scale),
283:             *tensor_handle_to_tensor_pointer(act_zero_point),
284:             *tensor_handle_to_tensor_pointer(onednn_weight),
285:             *tensor_handle_to_tensor_pointer(weight_scales),
286:             *tensor_handle_to_tensor_pointer(weight_zero_points),
287:             pointer_to_optional<at::Tensor>(other),
288:             pointer_to_optional<at::Tensor>(B),
```

- EN: The main execution path in this span is carried by `q_linear_pointwise_binary_tensor`, `tensor_handle_to_tensor_pointer`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `q_linear_pointwise_binary_tensor`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 289-304

```cpp
289:             output_scale,
290:             output_zero_point,
291:             pointer_to_optional<at::ScalarType>(output_dtype),
292:             other_scale,
293:             other_zero_point,
294:             binary_post_op,
295:             binary_alpha,
296:             unary_post_op,
297:             scalars_list,
298:             unary_post_op_algorithm);
299:     *ret0 = new_tensor_handle(std::move(tmp_result));
300:   });
301: }
302: 
303: AOTITorchError aoti_torch_xpu__qconv_pointwise_tensor(
304:     AtenTensorHandle X,
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_xpu__qconv_pointwise_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_xpu__qconv_pointwise_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 305-320

```cpp
305:     AtenTensorHandle act_scale,
306:     AtenTensorHandle act_zero_point,
307:     AtenTensorHandle onednn_weight,
308:     AtenTensorHandle weight_scales,
309:     AtenTensorHandle weight_zero_points,
310:     AtenTensorHandle* B,
311:     const int64_t* stride_args,
312:     int64_t stride_len_,
313:     const int64_t* padding_args,
314:     int64_t padding_len_,
315:     const int64_t* dilation_args,
316:     int64_t dilation_len_,
317:     int64_t groups,
318:     double output_scale,
319:     int64_t output_zero_point,
320:     const int32_t* output_dtype,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 321-336

```cpp
321:     const char* attr,
322:     const double** post_op_args,
323:     int64_t post_op_args_len_,
324:     const char** algorithm,
325:     AtenTensorHandle* ret0) {
326:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
327:     c10::List<std::optional<c10::Scalar>> scalars_list;
328:     scalars_list.reserve(post_op_args_len_);
329:     for (int64_t i = 0; i < post_op_args_len_; i++) {
330:       scalars_list.emplace_back(pointer_to_optional(post_op_args[i]));
331:     }
332: 
333:     c10::List<int64_t> stride_list =
334:         convert_to_c10_List<int64_t>(stride_args, stride_len_);
335:     c10::List<int64_t> padding_list =
336:         convert_to_c10_List<int64_t>(padding_args, padding_len_);
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 337-352

```cpp
337:     c10::List<int64_t> dilation_list =
338:         convert_to_c10_List<int64_t>(dilation_args, dilation_len_);
339: 
340:     auto tmp_result = at::native::xpu::QConvoneDNNXPU::run_pointwise_tensor(
341:         *tensor_handle_to_tensor_pointer(X),
342:         *tensor_handle_to_tensor_pointer(act_scale),
343:         *tensor_handle_to_tensor_pointer(act_zero_point),
344:         *tensor_handle_to_tensor_pointer(onednn_weight),
345:         *tensor_handle_to_tensor_pointer(weight_scales),
346:         *tensor_handle_to_tensor_pointer(weight_zero_points),
347:         pointer_to_optional<at::Tensor>(B),
348:         stride_list,
349:         padding_list,
350:         dilation_list,
351:         groups,
352:         output_scale,
```

- EN: The main execution path in this span is carried by `run_pointwise_tensor`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `run_pointwise_tensor`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 353-368

```cpp
353:         output_zero_point,
354:         pointer_to_optional<at::ScalarType>(output_dtype),
355:         attr,
356:         scalars_list,
357:         pointer_to_optional<std::string_view>(algorithm));
358:     *ret0 = new_tensor_handle(std::move(tmp_result));
359:   });
360: }
361: 
362: AOTITorchError aoti_torch_xpu__qconv2d_pointwise_binary_tensor(
363:     AtenTensorHandle X,
364:     AtenTensorHandle act_scale,
365:     AtenTensorHandle act_zero_point,
366:     AtenTensorHandle onednn_weight,
367:     AtenTensorHandle weight_scales,
368:     AtenTensorHandle weight_zero_points,
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_xpu__qconv2d_pointwise_binary_tensor`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_xpu__qconv2d_pointwise_binary_tensor` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 369-384

```cpp
369:     AtenTensorHandle accum,
370:     AtenTensorHandle* B,
371:     const int64_t* stride_args,
372:     int64_t stride_len_,
373:     const int64_t* padding_args,
374:     int64_t padding_len_,
375:     const int64_t* dilation_args,
376:     int64_t dilation_len_,
377:     int64_t groups,
378:     double output_scale,
379:     int64_t output_zero_point,
380:     const int32_t* output_dtype,
381:     double accum_scale,
382:     int64_t accum_zero_point,
383:     const char* binary_attr,
384:     double* alpha,
```

- EN: This range continues the surrounding implementation through local state updates, helper invocations, and data movement.
- CN: 这一段通过局部状态更新、辅助函数调用与数据传递来延续周围实现。
### Lines 385-400

```cpp
385:     const char** unary_attr,
386:     const double** unary_scalars,
387:     int64_t unary_scalars_len_,
388:     const char** unary_algorithm,
389:     AtenTensorHandle* ret0) {
390:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
391:     c10::List<std::optional<c10::Scalar>> unary_scalars_list;
392:     unary_scalars_list.reserve(unary_scalars_len_);
393:     for (int64_t i = 0; i < unary_scalars_len_; i++) {
394:       unary_scalars_list.emplace_back(pointer_to_optional(unary_scalars[i]));
395:     }
396: 
397:     c10::List<int64_t> stride_list =
398:         convert_to_c10_List<int64_t>(stride_args, stride_len_);
399:     c10::List<int64_t> padding_list =
400:         convert_to_c10_List<int64_t>(padding_args, padding_len_);
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 401-416

```cpp
401:     c10::List<int64_t> dilation_list =
402:         convert_to_c10_List<int64_t>(dilation_args, dilation_len_);
403: 
404:     auto tmp_result =
405:         at::native::xpu::QConvoneDNNXPU::run_pointwise_binary_tensor(
406:             *tensor_handle_to_tensor_pointer(X),
407:             *tensor_handle_to_tensor_pointer(act_scale),
408:             *tensor_handle_to_tensor_pointer(act_zero_point),
409:             *tensor_handle_to_tensor_pointer(onednn_weight),
410:             *tensor_handle_to_tensor_pointer(weight_scales),
411:             *tensor_handle_to_tensor_pointer(weight_zero_points),
412:             *tensor_handle_to_tensor_pointer(accum),
413:             pointer_to_optional<at::Tensor>(B),
414:             stride_list,
415:             padding_list,
416:             dilation_list,
```

- EN: The main execution path in this span is carried by `run_pointwise_binary_tensor`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `run_pointwise_binary_tensor`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 417-432

```cpp
417:             groups,
418:             output_scale,
419:             output_zero_point,
420:             pointer_to_optional<at::ScalarType>(output_dtype),
421:             accum_scale,
422:             accum_zero_point,
423:             binary_attr,
424:             pointer_to_optional<c10::Scalar>(alpha),
425:             pointer_to_optional<std::string_view>(unary_attr),
426:             unary_scalars_list,
427:             pointer_to_optional<std::string_view>(unary_algorithm));
428:     *ret0 = new_tensor_handle(std::move(tmp_result));
429:   });
430: }
431: 
432: #endif // AT_MKLDNN_ENABLED()
```

- EN: The main execution path in this span is carried by `new_tensor_handle`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。

## Key Concepts / 关键概念
- Inductor / AOTInductor runtime integration / Inductor / AOTInductor 运行时集成
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `aoti_torch_create_xpu_guard` / 核心符号 `aoti_torch_create_xpu_guard`
- Primary symbol `aoti_torch_delete_xpu_guard` / 核心符号 `aoti_torch_delete_xpu_guard`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `torch/csrc/inductor/aoti_torch/c/shim_xpu.h`, `torch/csrc/inductor/aoti_torch/utils.h`, `c10/core/DeviceGuard.h`, `c10/core/DeviceType.h`, `c10/core/StreamGuard.h`, `c10/xpu/XPUStream.h`, `ATen/native/mkldnn/xpu/Conv.h`, `ATen/native/mkldnn/xpu/qconv.h`, `ATen/native/mkldnn/xpu/qlinear.h`
- Include roots / 头文件根模块: `ATen`, `c10`, `torch`
- Key symbols / 关键符号: `aoti_torch_create_xpu_guard`, `aoti_torch_delete_xpu_guard`, `aoti_torch_xpu_guard_set_index`, `aoti_torch_create_xpu_stream_guard`, `aoti_torch_delete_xpu_stream_guard`, `aoti_torch_get_current_xpu_stream`, `aoti_torch_get_current_xpu_device`, `aoti_torch_set_current_xpu_device`, `aoti_torch_get_current_sycl_queue`, `aoti_torch_xpu_mkldnn__convolution_pointwise_binary`
- Related subsystems / 相关子系统: ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Inductor runtime / Inductor 运行时

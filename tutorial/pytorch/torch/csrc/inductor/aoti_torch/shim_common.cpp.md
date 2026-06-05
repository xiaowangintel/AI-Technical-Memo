# shim_common.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/inductor/aoti_torch/shim_common.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Implements AOTInductor torch-side bridge code for converting handles, dispatching calls, and surfacing runtime services.
- 目的 (CN): 实现 AOTInductor 的 torch 侧桥接代码，用于句柄转换、调用分发与运行时服务暴露。
- Lines: 1525
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

```cpp
 1: #include <ATen/native/quantized/cpu/qlinear.h>
 2: #include <ATen/record_function.h>
 3: #include <c10/core/DeviceType.h>
 4: #include <c10/core/DispatchKey.h>
 5: #include <c10/core/GradMode.h>
 6: #include <c10/core/Layout.h>
 7: #include <c10/core/MemoryFormat.h>
 8: #include <c10/core/ScalarType.h>
 9: #include <c10/util/Exception.h>
10: #include <torch/csrc/inductor/aoti_runtime/utils.h>
11: #include <torch/csrc/inductor/aoti_torch/c/shim.h>
12: #include <torch/csrc/inductor/aoti_torch/mkldnn_tensor.h>
13: #include <torch/csrc/inductor/aoti_torch/oss_proxy_executor.h>
14: #include <torch/csrc/inductor/aoti_torch/proxy_executor.h>
15: #include <torch/csrc/inductor/aoti_torch/tensor_converter.h>
16: #include <torch/csrc/inductor/aoti_torch/utils.h>
17: #include <torch/csrc/inductor/inductor_ops.h>
18: #include <torch/csrc/jit/serialization/pickle.h>
19: #include <torch/csrc/stable/library.h>
20: #include <torch/library.h>
21: #include <cstdint>
22: #include <cstdio>
23: #include <cstring>
24: #include <fstream>
25: #include <iostream>
26: #include <vector>
27: 
28: #include <c10/core/Device.h>
29: #include <c10/core/DeviceGuard.h>
30: #include <c10/core/Stream.h>
31: #include <c10/util/FileSystem.h>
32: #include <torch/headeronly/version.h>
```

- EN: These lines pull in dependencies such as `ATen/native/quantized/cpu/qlinear.h`, `ATen/record_function.h`, `c10/core/DeviceType.h`, establishing the headers needed by the implementation. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/native/quantized/cpu/qlinear.h`, `ATen/record_function.h`, `c10/core/DeviceType.h`，为后续实现建立所需的头文件基础。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 33-64

```cpp
33: 
34: #ifndef AT_PER_OPERATOR_HEADERS
35: #include <ATen/Functions.h>
36: #else
37: 
38: #include <ATen/ops/_addmm_activation.h>
39: #include <ATen/ops/_embedding_bag.h>
40: #include <ATen/ops/_fft_c2c.h>
41: #include <ATen/ops/_scaled_dot_product_efficient_attention.h>
42: #include <ATen/ops/_scaled_dot_product_flash_attention.h>
43: #include <ATen/ops/_scaled_mm.h>
44: #include <ATen/ops/_wrapped_linear_prepack.h>
45: #include <ATen/ops/_wrapped_quantized_linear_prepacked.h>
46: #include <ATen/ops/addmm.h>
47: #include <ATen/ops/as_strided.h>
48: #include <ATen/ops/bmm.h>
49: #include <ATen/ops/convolution.h>
50: #include <ATen/ops/empty_strided.h>
51: #include <ATen/ops/fbgemm_linear_fp16_weight_fp32_activation.h>
52: #include <ATen/ops/fbgemm_pack_gemm_matrix_fp16.h>
53: #include <ATen/ops/from_blob.h>
54: #include <ATen/ops/index_put.h>
55: #include <ATen/ops/mm.h>
56: #include <ATen/ops/nonzero.h>
57: #include <ATen/ops/scalar_tensor.h>
58: #include <ATen/ops/scatter.h>
59: #include <ATen/ops/scatter_reduce.h>
60: #include <ATen/ops/view_as_real_ops.h>
61: #include <ATen/ops/view_ops.h>
62: #endif
63: 
64: using namespace torch::aot_inductor;
```

- EN: These lines pull in dependencies such as `ATen/Functions.h`, `ATen/ops/_addmm_activation.h`, `ATen/ops/_embedding_bag.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/Functions.h`, `ATen/ops/_addmm_activation.h`, `ATen/ops/_embedding_bag.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 65-96

```cpp
65: 
66: namespace {
67: static c10::Device c10_device(int32_t device_type, int32_t device_index) {
68:   if (device_type == aoti_torch_device_type_cpu()) {
69:     return c10::Device(static_cast<c10::DeviceType>(device_type));
70:   } else {
71:     return c10::Device(
72:         static_cast<c10::DeviceType>(device_type),
73:         static_cast<c10::DeviceIndex>(device_index));
74:   }
75: }
76: } // namespace
77: 
78: const int AOTI_TORCH_MAX_NUMEL_TO_PRINT = 64;
79: 
80: #define AOTI_TORCH_DEVICE_TYPE_IMPL(device_str, device_type) \
81:   int32_t aoti_torch_device_type_##device_str() {            \
82:     return (int32_t)c10::DeviceType::device_type;            \
83:   }
84: 
85: AOTI_TORCH_DEVICE_TYPE_IMPL(cpu, CPU)
86: AOTI_TORCH_DEVICE_TYPE_IMPL(cuda, CUDA)
87: AOTI_TORCH_DEVICE_TYPE_IMPL(meta, Meta)
88: AOTI_TORCH_DEVICE_TYPE_IMPL(xpu, XPU)
89: AOTI_TORCH_DEVICE_TYPE_IMPL(mps, MPS)
90: AOTI_TORCH_DEVICE_TYPE_IMPL(privateuse1, PrivateUse1)
91: #undef AOTI_TORCH_DEVICE_TYPE_IMPL
92: 
93: #define AOTI_TORCH_DTYPE_IMPL(dtype, stype) \
94:   int32_t aoti_torch_dtype_##dtype() {      \
95:     return (int32_t)c10::ScalarType::stype; \
96:   }
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `c10_device`, `Device`, `AOTI_TORCH_DEVICE_TYPE_IMPL`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `c10_device`, `Device`, `AOTI_TORCH_DEVICE_TYPE_IMPL` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 97-128

```cpp
 97: 
 98: AOTI_TORCH_DTYPE_IMPL(float8_e5m2, Float8_e5m2)
 99: AOTI_TORCH_DTYPE_IMPL(float8_e4m3fn, Float8_e4m3fn)
100: AOTI_TORCH_DTYPE_IMPL(float8_e5m2fnuz, Float8_e5m2fnuz)
101: AOTI_TORCH_DTYPE_IMPL(float8_e4m3fnuz, Float8_e4m3fnuz)
102: AOTI_TORCH_DTYPE_IMPL(float8_e8m0fnu, Float8_e8m0fnu)
103: AOTI_TORCH_DTYPE_IMPL(float4_e2m1fn_x2, Float4_e2m1fn_x2)
104: AOTI_TORCH_DTYPE_IMPL(bfloat16, BFloat16)
105: AOTI_TORCH_DTYPE_IMPL(float16, Half)
106: AOTI_TORCH_DTYPE_IMPL(float32, Float)
107: AOTI_TORCH_DTYPE_IMPL(float64, Double)
108: AOTI_TORCH_DTYPE_IMPL(uint8, Byte)
109: AOTI_TORCH_DTYPE_IMPL(uint16, UInt16)
110: AOTI_TORCH_DTYPE_IMPL(uint32, UInt32)
111: AOTI_TORCH_DTYPE_IMPL(uint64, UInt64)
112: AOTI_TORCH_DTYPE_IMPL(int8, Char)
113: AOTI_TORCH_DTYPE_IMPL(int16, Short)
114: AOTI_TORCH_DTYPE_IMPL(int32, Int)
115: AOTI_TORCH_DTYPE_IMPL(int64, Long)
116: AOTI_TORCH_DTYPE_IMPL(bool, Bool)
117: AOTI_TORCH_DTYPE_IMPL(complex32, ComplexHalf)
118: AOTI_TORCH_DTYPE_IMPL(complex64, ComplexFloat)
119: AOTI_TORCH_DTYPE_IMPL(complex128, ComplexDouble)
120: #undef AOTI_TORCH_DTYPE_IMPL
121: 
122: #define AOTI_TORCH_LAYOUT_IMPL(name, enum) \
123:   int32_t aoti_torch_layout_##name() {     \
124:     return (int32_t)at::Layout::enum;      \
125:   }
126: 
127: AOTI_TORCH_LAYOUT_IMPL(strided, Strided)
128: AOTI_TORCH_LAYOUT_IMPL(sparse_coo, Sparse)
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_DTYPE_IMPL`, `AOTI_TORCH_LAYOUT_IMPL`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_DTYPE_IMPL`, `AOTI_TORCH_LAYOUT_IMPL` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 129-160

```cpp
129: AOTI_TORCH_LAYOUT_IMPL(sparse_csr, SparseCsr)
130: AOTI_TORCH_LAYOUT_IMPL(sparse_csc, SparseCsc)
131: AOTI_TORCH_LAYOUT_IMPL(sparse_bsr, SparseBsr)
132: AOTI_TORCH_LAYOUT_IMPL(sparse_bsc, SparseBsc)
133: AOTI_TORCH_LAYOUT_IMPL(_mkldnn, Mkldnn)
134: AOTI_TORCH_LAYOUT_IMPL(jagged, Jagged)
135: #undef AOTI_TORCH_LAYOUT_IMPL
136: 
137: #define AOTI_TORCH_MEMORY_FORMAT_IMPL(name, enum) \
138:   int32_t aoti_torch_memory_format_##name() {     \
139:     return (int32_t)at::MemoryFormat::enum;       \
140:   }
141: 
142: AOTI_TORCH_MEMORY_FORMAT_IMPL(contiguous_format, Contiguous)
143: AOTI_TORCH_MEMORY_FORMAT_IMPL(channels_last, ChannelsLast)
144: AOTI_TORCH_MEMORY_FORMAT_IMPL(channels_last_3d, ChannelsLast3d)
145: AOTI_TORCH_MEMORY_FORMAT_IMPL(preserve_format, Preserve)
146: #undef AOTI_TORCH_MEMORY_FORMAT_IMPL
147: 
148: #define AOTI_TORCH_ITEM_IMPL(dtype, ctype)                     \
149:   AOTITorchError aoti_torch_item_##dtype(                      \
150:       AtenTensorHandle tensor, ctype* ret_value) {             \
151:     AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({               \
152:       at::Tensor* t = tensor_handle_to_tensor_pointer(tensor); \
153:       *ret_value = t->item().to<ctype>();                      \
154:     });                                                        \
155:   }
156: 
157: AOTI_TORCH_ITEM_IMPL(float16, c10::Half)
158: AOTI_TORCH_ITEM_IMPL(float32, float)
159: AOTI_TORCH_ITEM_IMPL(float64, double)
160: AOTI_TORCH_ITEM_IMPL(uint8, uint8_t)
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_LAYOUT_IMPL`, `AOTI_TORCH_MEMORY_FORMAT_IMPL`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_LAYOUT_IMPL`, `AOTI_TORCH_MEMORY_FORMAT_IMPL`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 161-192

```cpp
161: AOTI_TORCH_ITEM_IMPL(uint16, uint16_t)
162: AOTI_TORCH_ITEM_IMPL(uint32, uint32_t)
163: AOTI_TORCH_ITEM_IMPL(uint64, uint64_t)
164: AOTI_TORCH_ITEM_IMPL(int8, int8_t)
165: AOTI_TORCH_ITEM_IMPL(int16, int16_t)
166: AOTI_TORCH_ITEM_IMPL(int32, int32_t)
167: AOTI_TORCH_ITEM_IMPL(int64, int64_t)
168: AOTI_TORCH_ITEM_IMPL(bool, bool)
169: AOTI_TORCH_ITEM_IMPL(bfloat16, c10::BFloat16)
170: AOTI_TORCH_ITEM_IMPL(complex64, c10::complex<float>)
171: #undef AOTI_TORCH_ITEM_IMPL
172: 
173: #define AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(dtype, ctype, ttype)                  \
174:   AOTITorchError aoti_torch_scalar_to_tensor_##dtype(                          \
175:       ctype value, AtenTensorHandle* ret_new_tensor) {                         \
176:     AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({                               \
177:       *ret_new_tensor =                                                        \
178:           new_tensor_handle(at::scalar_tensor(value, c10::ScalarType::ttype)); \
179:     });                                                                        \
180:   }
181: 
182: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(float32, float, Float)
183: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(float64, double, Double)
184: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(uint8, uint8_t, Byte)
185: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(uint16, uint16_t, UInt16)
186: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(uint32, uint32_t, UInt32)
187: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(uint64, uint64_t, UInt64)
188: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(int8, int8_t, Char)
189: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(int16, int16_t, Short)
190: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(int32, int32_t, Int)
191: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(int64, int64_t, Long)
192: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(bool, bool, Bool)
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_ITEM_IMPL`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `new_tensor_handle`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_ITEM_IMPL`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `new_tensor_handle` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 193-224

```cpp
193: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(complex64, c10::complex<float>, ComplexFloat)
194: AOTI_TORCH_SCALAR_TO_TENSOR_IMPL(
195:     complex128,
196:     c10::complex<double>,
197:     ComplexDouble)
198: #undef AOTI_TORCH_SCALAR_TO_TENSOR_IMPL
199: 
200: uint64_t aoti_torch_abi_version() {
201:   return TORCH_ABI_VERSION;
202: }
203: 
204: bool aoti_torch_grad_mode_is_enabled() {
205:   return c10::GradMode::is_enabled();
206: }
207: 
208: void aoti_torch_grad_mode_set_enabled(bool enabled) {
209:   return c10::GradMode::set_enabled(enabled);
210: }
211: 
212: size_t aoti_torch_dtype_element_size(int32_t dtype) {
213:   auto scalar_type = static_cast<at::ScalarType>(dtype);
214:   return c10::elementSize(scalar_type);
215: }
216: 
217: AOTITorchError aoti_torch_delete_tensor_object(AtenTensorHandle tensor) {
218:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
219:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
220:     delete t;
221:   });
222: }
223: 
224: AOTITorchError aoti_torch_delete_c10_value_object(C10IValueHandle handle) {
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_SCALAR_TO_TENSOR_IMPL`, `aoti_torch_abi_version`, `aoti_torch_grad_mode_is_enabled`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_SCALAR_TO_TENSOR_IMPL`, `aoti_torch_abi_version`, `aoti_torch_grad_mode_is_enabled` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 225-256

```cpp
225:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
226:     c10::IValue* t = reinterpret_cast<c10::IValue*>(handle);
227:     delete t;
228:   });
229: }
230: 
231: AOTITorchError aoti_torch_int64_to_ivalue(
232:     int64_t val,
233:     C10IValueHandle* ivalue) {
234:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
235:     c10::IValue* t = new c10::IValue(val);
236:     *ivalue = reinterpret_cast<C10IValueHandle>(t);
237:   });
238: }
239: 
240: AOTITorchError aoti_torch_str_to_ivalue(
241:     const char* val,
242:     C10IValueHandle* ivalue) {
243:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
244:     c10::IValue* t = new c10::IValue(val);
245:     *ivalue = reinterpret_cast<C10IValueHandle>(t);
246:   });
247: }
248: 
249: AOTITorchError aoti_torch_strlist_to_ivalue(
250:     const char** val,
251:     int64_t len,
252:     C10IValueHandle* ivalue) {
253:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
254:     c10::List<std::string> vec;
255:     for (int64_t i = 0; i < len; i++) {
256:       vec.push_back(std::string(val[i]));
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `aoti_torch_int64_to_ivalue`, `IValue`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `aoti_torch_int64_to_ivalue`, `IValue` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 257-288

```cpp
257:     }
258:     c10::IValue* t = new c10::IValue(vec);
259:     *ivalue = reinterpret_cast<C10IValueHandle>(t);
260:   });
261: }
262: 
263: AOTITorchError aoti_torch_tensor_to_ivalue(
264:     AtenTensorHandle tensor,
265:     C10IValueHandle* ivalue) {
266:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
267:     at::Tensor* tmp_tensor = tensor_handle_to_tensor_pointer(tensor);
268:     c10::IValue* tmp_ivalue = new c10::IValue(*tmp_tensor);
269:     *ivalue = reinterpret_cast<C10IValueHandle>(tmp_ivalue);
270:   });
271: }
272: 
273: AOTITorchError aoti_torch_get_data_ptr(
274:     AtenTensorHandle tensor,
275:     void** ret_data_ptr) {
276:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
277:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
278:     if (t->is_mkldnn()) {
279:       *ret_data_ptr = data_ptr_from_mkldnn(t);
280:     } else {
281:       *ret_data_ptr = t->data_ptr();
282:     }
283:   });
284: }
285: 
286: AOTITorchError aoti_torch_get_storage_size(
287:     AtenTensorHandle tensor,
288:     int64_t* ret_size) {
```

- EN: The main execution path in this span is carried by `IValue`, `aoti_torch_tensor_to_ivalue`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `IValue`, `aoti_torch_tensor_to_ivalue`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 289-320

```cpp
289:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
290:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
291:     *ret_size = t->storage().nbytes();
292:   });
293: }
294: 
295: AOTITorchError aoti_torch_get_dim(AtenTensorHandle tensor, int64_t* ret_dim) {
296:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
297:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
298:     *ret_dim = t->dim();
299:   });
300: }
301: 
302: AOTITorchError aoti_torch_get_numel(
303:     AtenTensorHandle tensor,
304:     int64_t* ret_numel) {
305:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
306:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
307:     *ret_numel = t->numel();
308:   });
309: }
310: 
311: AOTITorchError aoti_torch_get_storage_numel(
312:     AtenTensorHandle tensor,
313:     int64_t* ret_numel) {
314:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
315:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
316:     TORCH_INTERNAL_ASSERT(t->has_storage());
317:     auto dtype_size = t->dtype().itemsize();
318:     size_t nbytes = t->storage().nbytes();
319:     TORCH_INTERNAL_ASSERT(nbytes % dtype_size == 0);
320:     auto numel = nbytes / dtype_size;
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `aoti_torch_get_dim`. The logic emits runtime diagnostics or assertions to guard assumptions. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `aoti_torch_get_dim` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 321-352

```cpp
321:     *ret_numel = numel;
322:   });
323: }
324: 
325: AOTITorchError aoti_torch_get_sizes(
326:     AtenTensorHandle tensor,
327:     int64_t** ret_sizes) {
328:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
329:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
330:     // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
331:     *ret_sizes = const_cast<int64_t*>(t->sizes().data());
332:   });
333: }
334: 
335: AOTITorchError aoti_torch_get_size(
336:     AtenTensorHandle tensor,
337:     int64_t d,
338:     int64_t* ret_size) {
339:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
340:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
341:     *ret_size = t->size(d);
342:   });
343: }
344: 
345: AOTITorchError aoti_torch_get_strides(
346:     AtenTensorHandle tensor,
347:     int64_t** ret_strides) {
348:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
349:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
350:     // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
351:     *ret_strides = const_cast<int64_t*>(t->strides().data());
352:   });
```

- EN: The main execution path in this span is carried by `aoti_torch_get_sizes`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_get_sizes`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 353-384

```cpp
353: }
354: 
355: AOTITorchError aoti_torch_get_stride(
356:     AtenTensorHandle tensor,
357:     int64_t d,
358:     int64_t* ret_stride) {
359:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
360:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
361:     *ret_stride = t->stride(d);
362:   });
363: }
364: 
365: AOTITorchError aoti_torch_get_dtype(
366:     AtenTensorHandle tensor,
367:     int32_t* ret_dtype) {
368:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
369:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
370:     *ret_dtype = static_cast<int32_t>(t->scalar_type());
371:   });
372: }
373: 
374: AOTITorchError aoti_torch_get_device_type(
375:     AtenTensorHandle tensor,
376:     int32_t* ret_device_type) {
377:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
378:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
379:     *ret_device_type = static_cast<int32_t>(t->device().type());
380:   });
381: }
382: 
383: AOTITorchError aoti_torch_get_device_index(
384:     AtenTensorHandle tensor,
```

- EN: The main execution path in this span is carried by `aoti_torch_get_stride`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_get_stride`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 385-416

```cpp
385:     int32_t* ret_device_index) {
386:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
387:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
388:     *ret_device_index = static_cast<int16_t>(t->device().index());
389:   });
390: }
391: 
392: AOTITorchError aoti_torch_get_layout(
393:     AtenTensorHandle tensor,
394:     int32_t* ret_layout) {
395:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
396:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
397:     *ret_layout = static_cast<int32_t>(t->layout());
398:   });
399: }
400: 
401: AOTITorchError aoti_torch_get_storage_offset(
402:     AtenTensorHandle tensor,
403:     int64_t* ret_storage_offset) {
404:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
405:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
406:     *ret_storage_offset = t->storage_offset();
407:   });
408: }
409: 
410: AOTITorchError aoti_torch_is_contiguous(
411:     AtenTensorHandle tensor,
412:     bool* ret_is_contiguous) {
413:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
414:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
415:     *ret_is_contiguous = t->is_contiguous();
416:   });
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `aoti_torch_get_layout`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `aoti_torch_get_layout` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 417-448

```cpp
417: }
418: 
419: AOTITorchError aoti_torch_is_defined(
420:     AtenTensorHandle tensor,
421:     bool* ret_is_defined) {
422:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
423:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
424:     *ret_is_defined = t->defined();
425:   });
426: }
427: 
428: AOTITorchError aoti_torch_new_tensor_handle(
429:     AtenTensorHandle orig_handle,
430:     AtenTensorHandle* new_handle) {
431:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
432:     at::Tensor* t = tensor_handle_to_tensor_pointer(orig_handle);
433:     *new_handle = new_tensor_handle(at::Tensor(*t));
434:   });
435: }
436: 
437: AOTITorchError aoti_torch__reinterpret_tensor(
438:     AtenTensorHandle self,
439:     int64_t ndim,
440:     const int64_t* sizes_ptr,
441:     const int64_t* strides_ptr,
442:     int64_t offset_increment,
443:     AtenTensorHandle* ret_new_tensor) {
444:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
445:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
446:     c10::IntArrayRef sizes(sizes_ptr, ndim);
447:     c10::IntArrayRef strides(strides_ptr, ndim);
448:     *ret_new_tensor = new_tensor_handle(torch::inductor::_reinterpret_tensor(
```

- EN: The main execution path in this span is carried by `aoti_torch_is_defined`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_is_defined`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 449-480

```cpp
449:         *self_tensor, sizes, strides, offset_increment));
450:   });
451: }
452: 
453: // TODO: implement a more efficient version instead of calling into aten
454: AOTITorchError aoti_torch_empty_strided(
455:     int64_t ndim,
456:     const int64_t* sizes_ptr,
457:     const int64_t* strides_ptr,
458:     int32_t dtype,
459:     int32_t device_type,
460:     int32_t device_index,
461:     AtenTensorHandle* ret_new_tensor) {
462:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
463:     c10::IntArrayRef sizes(sizes_ptr, ndim);
464:     c10::IntArrayRef strides(strides_ptr, ndim);
465:     if (c10::DeviceType(device_type) == c10::DeviceType::CPU) {
466:       *ret_new_tensor = new_tensor_handle(at::detail::empty_strided_cpu(
467:           sizes, strides, static_cast<c10::ScalarType>(dtype)));
468:     } else {
469:       c10::Device device = c10_device(device_type, device_index);
470:       c10::TensorOptions options = c10::TensorOptions().device(device).dtype(
471:           static_cast<c10::ScalarType>(dtype));
472:       *ret_new_tensor =
473:           new_tensor_handle(at::empty_strided(sizes, strides, options));
474:     }
475:   });
476: }
477: 
478: AOTITorchError aoti_torch_empty_strided_pinned(
479:     int64_t ndim,
480:     const int64_t* sizes_ptr,
```

- EN: The main execution path in this span is carried by `aoti_torch_empty_strided`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `sizes`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_empty_strided`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `sizes` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 481-512

```cpp
481:     const int64_t* strides_ptr,
482:     int32_t dtype,
483:     int32_t device_type,
484:     int32_t device_index,
485:     AtenTensorHandle* ret_new_tensor) {
486:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
487:     c10::IntArrayRef sizes(sizes_ptr, ndim);
488:     c10::IntArrayRef strides(strides_ptr, ndim);
489:     TORCH_CHECK(
490:         c10::DeviceType(device_type) == c10::DeviceType::CPU,
491:         "only CPU tensors can be pinned");
492:     *ret_new_tensor = new_tensor_handle(at::detail::empty_strided_cpu(
493:         sizes,
494:         strides,
495:         static_cast<c10::ScalarType>(dtype),
496:         /*is_pinned=*/true));
497:   });
498: }
499: 
500: AOTITorchError aoti_torch_create_tensor_from_blob(
501:     void* data,
502:     int64_t ndim,
503:     const int64_t* sizes_ptr,
504:     const int64_t* strides_ptr,
505:     int64_t storage_offset,
506:     int32_t dtype,
507:     int32_t device_type,
508:     int32_t device_index,
509:     AtenTensorHandle* ret_new_tensor) {
510:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
511:     c10::IntArrayRef sizes(sizes_ptr, ndim);
512:     c10::IntArrayRef strides(strides_ptr, ndim);
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `sizes`, `strides`. The logic emits runtime diagnostics or assertions to guard assumptions. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `sizes`, `strides` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 513-544

```cpp
513:     c10::Device device = c10_device(device_type, device_index);
514:     c10::TensorOptions options = c10::TensorOptions().device(device).dtype(
515:         static_cast<c10::ScalarType>(dtype));
516:     *ret_new_tensor = new_tensor_handle(
517:         // data == nullptr can happen for a 0-size tensor
518:         (data != nullptr) ? at::for_blob(data, sizes)
519:                                 .strides(strides)
520:                                 .storage_offset(storage_offset)
521:                                 .options(options)
522:                                 .make_tensor()
523:                           : at::empty_strided(sizes, strides, options));
524:   });
525: }
526: 
527: AOTITorchError aoti_torch_create_tensor_from_blob_v2(
528:     void* data,
529:     int64_t ndim,
530:     const int64_t* sizes_ptr,
531:     const int64_t* strides_ptr,
532:     int64_t storage_offset,
533:     int32_t dtype,
534:     int32_t device_type,
535:     int32_t device_index,
536:     AtenTensorHandle* ret_new_tensor,
537:     int32_t layout,
538:     const uint8_t* opaque_metadata,
539:     int64_t opaque_metadata_size) {
540:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
541:     if (layout == static_cast<int32_t>(at::kMkldnn)) {
542:       c10::IntArrayRef sizes(sizes_ptr, ndim);
543:       c10::IntArrayRef strides(strides_ptr, ndim);
544:       c10::Device device = c10_device(device_type, device_index);
```

- EN: The main execution path in this span is carried by `c10_device`, `TensorOptions`, `new_tensor_handle`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `c10_device`, `TensorOptions`, `new_tensor_handle` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 545-576

```cpp
545:       // get a mkldnn tensor wrapped by a torch Tensor(OpaqueTensorImpl),
546:       // which used by later mkldnn op.
547:       *ret_new_tensor = new_tensor_handle(mkldnn_tensor_from_data_ptr(
548:           data,
549:           sizes,
550:           static_cast<c10::ScalarType>(dtype),
551:           device,
552:           opaque_metadata,
553:           opaque_metadata_size));
554:     } else {
555:       aoti_torch_create_tensor_from_blob(
556:           data,
557:           ndim,
558:           sizes_ptr,
559:           strides_ptr,
560:           storage_offset,
561:           dtype,
562:           device_type,
563:           device_index,
564:           ret_new_tensor);
565:     }
566:   });
567: }
568: 
569: AOTITorchError aoti_torch__embedding_bag(
570:     AtenTensorHandle weight,
571:     AtenTensorHandle indices,
572:     AtenTensorHandle offsets,
573:     int32_t scale_grad_by_freq,
574:     int32_t mode,
575:     int32_t sparse,
576:     AtenTensorHandle per_sample_weights, // optional argument
```

- EN: The main execution path in this span is carried by `Tensor`, `new_tensor_handle`, `aoti_torch_create_tensor_from_blob`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `Tensor`, `new_tensor_handle`, `aoti_torch_create_tensor_from_blob` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 577-608

```cpp
577:     int32_t include_last_offset,
578:     int32_t padding_idx,
579:     AtenTensorHandle* ret0, // returns new reference
580:     AtenTensorHandle* ret1, // returns new reference
581:     AtenTensorHandle* ret2, // returns new reference
582:     AtenTensorHandle* ret3 // returns new reference
583: ) {
584:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
585:     auto [r0, r1, r2, r3] = at::_embedding_bag(
586:         *tensor_handle_to_tensor_pointer(weight),
587:         *tensor_handle_to_tensor_pointer(indices),
588:         *tensor_handle_to_tensor_pointer(offsets),
589:         scale_grad_by_freq,
590:         mode,
591:         sparse,
592:         pointer_to_optional(
593:             tensor_handle_to_tensor_pointer(per_sample_weights)),
594:         include_last_offset,
595:         padding_idx);
596: 
597:     *ret0 = new_tensor_handle(std::move(r0));
598:     *ret1 = new_tensor_handle(std::move(r1));
599:     *ret2 = new_tensor_handle(std::move(r2));
600:     *ret3 = new_tensor_handle(std::move(r3));
601:   });
602: }
603: 
604: AOTITorchError aoti_torch__fft_c2c(
605:     AtenTensorHandle self,
606:     const int64_t* dim_ptr,
607:     int64_t dim_size,
608:     int64_t normalization,
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `_embedding_bag`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `_embedding_bag`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 609-640

```cpp
609:     int32_t forward,
610:     AtenTensorHandle* ret // returns new reference
611: ) {
612:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
613:     auto dim = c10::IntArrayRef(dim_ptr, dim_size);
614:     *ret = new_tensor_handle(at::_fft_c2c(
615:         *tensor_handle_to_tensor_pointer(self), dim, normalization, forward));
616:   });
617: }
618: 
619: AOTITorchError aoti_torch__scaled_dot_product_flash_attention_v2(
620:     AtenTensorHandle query,
621:     AtenTensorHandle key,
622:     AtenTensorHandle value,
623:     double dropout_p,
624:     int is_causal,
625:     int return_debug_mask,
626:     double* scale, // optional argument
627:     AtenTensorHandle* ret0, // returns new reference
628:     AtenTensorHandle* ret1, // returns new reference
629:     AtenTensorHandle* ret2, // returns new reference
630:     AtenTensorHandle* ret3, // returns new reference
631:     int64_t* ret4,
632:     int64_t* ret5,
633:     AtenTensorHandle* ret6, // returns new reference
634:     AtenTensorHandle* ret7, // returns new reference
635:     AtenTensorHandle* ret8 // returns new reference
636: ) {
637:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
638:     at::Tensor* query_tensor = tensor_handle_to_tensor_pointer(query);
639:     at::Tensor* key_tensor = tensor_handle_to_tensor_pointer(key);
640:     at::Tensor* value_tensor = tensor_handle_to_tensor_pointer(value);
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `IntArrayRef`, `new_tensor_handle`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `IntArrayRef`, `new_tensor_handle` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 641-672

```cpp
641:     auto optional_scale = pointer_to_optional(scale);
642:     auto [r0, r1, r2, r3, r4, r5, r6, r7, r8] =
643:         at::_scaled_dot_product_flash_attention(
644:             *query_tensor,
645:             *key_tensor,
646:             *value_tensor,
647:             dropout_p,
648:             is_causal,
649:             return_debug_mask,
650:             optional_scale);
651: 
652:     *ret0 = new_tensor_handle(std::move(r0));
653:     *ret1 = new_tensor_handle(std::move(r1));
654:     // ret2 and ret3 may be null
655:     if (ret2) {
656:       *ret2 = new_tensor_handle(std::move(r2));
657:     }
658:     if (ret3) {
659:       *ret3 = new_tensor_handle(std::move(r3));
660:     }
661:     *ret4 = r4.expect_int();
662:     *ret5 = r5.expect_int();
663:     *ret6 = new_tensor_handle(std::move(r6));
664:     *ret7 = new_tensor_handle(std::move(r7));
665:     *ret8 = new_tensor_handle(std::move(r8));
666:   });
667: }
668: 
669: AOTITorchError aoti_torch__scaled_dot_product_flash_attention(
670:     AtenTensorHandle query,
671:     AtenTensorHandle key,
672:     AtenTensorHandle value,
```

- EN: The main execution path in this span is carried by `pointer_to_optional`, `_scaled_dot_product_flash_attention`, `new_tensor_handle`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `pointer_to_optional`, `_scaled_dot_product_flash_attention`, `new_tensor_handle` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 673-704

```cpp
673:     double dropout_p,
674:     bool is_causal,
675:     bool return_debug_mask,
676:     double scale,
677:     AtenTensorHandle* ret0, // returns new reference
678:     AtenTensorHandle* ret1, // returns new reference
679:     AtenTensorHandle* ret2, // returns new reference
680:     AtenTensorHandle* ret3, // returns new reference
681:     int64_t* ret4,
682:     int64_t* ret5,
683:     AtenTensorHandle* ret6, // returns new reference
684:     AtenTensorHandle* ret7, // returns new reference
685:     AtenTensorHandle* ret8 // returns new reference
686: ) {
687:   return aoti_torch__scaled_dot_product_flash_attention_v2(
688:       query,
689:       key,
690:       value,
691:       dropout_p,
692:       is_causal,
693:       return_debug_mask,
694:       &scale,
695:       ret0,
696:       ret1,
697:       ret2,
698:       ret3,
699:       ret4,
700:       ret5,
701:       ret6,
702:       ret7,
703:       ret8);
704: }
```

- EN: The main execution path in this span is carried by `aoti_torch__scaled_dot_product_flash_attention_v2`. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `aoti_torch__scaled_dot_product_flash_attention_v2` 等函数/方法承载。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 705-736

```cpp
705: 
706: AOTITorchError aoti_torch__scaled_dot_product_efficient_attention(
707:     AtenTensorHandle query,
708:     AtenTensorHandle key,
709:     AtenTensorHandle value,
710:     AtenTensorHandle attn_bias, // optional argument
711:     int compute_log_sumexp,
712:     double dropout_p,
713:     int is_causal,
714:     double* scale, // optional argument
715:     AtenTensorHandle* ret0, // returns new reference
716:     AtenTensorHandle* ret1, // returns new reference
717:     AtenTensorHandle* ret2, // returns new reference
718:     AtenTensorHandle* ret3 // returns new reference
719: ) {
720:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
721:     at::Tensor* query_tensor = tensor_handle_to_tensor_pointer(query);
722:     at::Tensor* key_tensor = tensor_handle_to_tensor_pointer(key);
723:     at::Tensor* value_tensor = tensor_handle_to_tensor_pointer(value);
724:     auto optional_attn_bias =
725:         pointer_to_optional(tensor_handle_to_tensor_pointer(attn_bias));
726:     auto optional_scale = pointer_to_optional(scale);
727:     auto [r0, r1, r2, r3] = at::_scaled_dot_product_efficient_attention(
728:         *query_tensor,
729:         *key_tensor,
730:         *value_tensor,
731:         optional_attn_bias,
732:         compute_log_sumexp,
733:         dropout_p,
734:         is_causal,
735:         optional_scale);
736:     *ret0 = new_tensor_handle(std::move(r0));
```

- EN: The main execution path in this span is carried by `aoti_torch__scaled_dot_product_efficient_attention`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch__scaled_dot_product_efficient_attention`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 737-768

```cpp
737:     *ret1 = new_tensor_handle(std::move(r1));
738:     *ret2 = new_tensor_handle(std::move(r2));
739:     *ret3 = new_tensor_handle(std::move(r3));
740:   });
741: }
742: 
743: AOTITorchError aoti_torch_convolution(
744:     AtenTensorHandle input,
745:     AtenTensorHandle weight,
746:     AtenTensorHandle bias, // optional argument
747:     const int64_t* stride_ptr,
748:     int64_t stride_size,
749:     const int64_t* padding_ptr,
750:     int64_t padding_size,
751:     const int64_t* dilation_ptr,
752:     int64_t dilation_size,
753:     int transposed,
754:     const int64_t* output_padding_ptr,
755:     int64_t output_padding_size,
756:     int64_t groups,
757:     AtenTensorHandle* out // returns new reference
758: ) {
759:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
760:     at::Tensor* input_tensor = tensor_handle_to_tensor_pointer(input);
761:     at::Tensor* weight_tensor = tensor_handle_to_tensor_pointer(weight);
762:     at::Tensor* bias_tensor = tensor_handle_to_tensor_pointer(bias);
763:     auto optional_bias = pointer_to_optional(bias_tensor);
764:     c10::IntArrayRef stride(stride_ptr, stride_size);
765:     c10::IntArrayRef padding(padding_ptr, padding_size);
766:     c10::IntArrayRef dilation(dilation_ptr, dilation_size);
767:     c10::IntArrayRef output_padding(output_padding_ptr, output_padding_size);
768: 
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_convolution`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_convolution`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 769-800

```cpp
769:     *out = new_tensor_handle(at::convolution(
770:         *input_tensor,
771:         *weight_tensor,
772:         optional_bias,
773:         stride,
774:         padding,
775:         dilation,
776:         static_cast<bool>(transposed),
777:         output_padding,
778:         groups));
779:   });
780: }
781: 
782: AOTITorchError aoti_torch_new_uninitialized_tensor(AtenTensorHandle* ret) {
783:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
784:     at::Tensor* out_tensor = new at::Tensor();
785:     *ret = tensor_pointer_to_tensor_handle(out_tensor);
786:   });
787: }
788: 
789: AOTITorchError aoti_torch__scaled_mm(
790:     AtenTensorHandle self,
791:     AtenTensorHandle mat2,
792:     AtenTensorHandle bias,
793:     int32_t* out_dtype,
794:     AtenTensorHandle scale_a,
795:     AtenTensorHandle scale_b,
796:     AtenTensorHandle scale_result,
797:     int8_t use_fast_accum,
798:     AtenTensorHandle* ret0,
799:     AtenTensorHandle* ret1) {
800:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_new_uninitialized_tensor`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_new_uninitialized_tensor`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 801-832

```cpp
801:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
802:     at::Tensor* mat2_tensor = tensor_handle_to_tensor_pointer(mat2);
803:     at::Tensor* bias_tensor = tensor_handle_to_tensor_pointer(bias);
804:     at::Tensor* scale_a_tensor = tensor_handle_to_tensor_pointer(scale_a);
805:     at::Tensor* scale_b_tensor = tensor_handle_to_tensor_pointer(scale_b);
806:     at::Tensor* scale_result_tensor =
807:         tensor_handle_to_tensor_pointer(scale_result);
808:     auto r0 = at::_scaled_mm(
809:         *self_tensor,
810:         *mat2_tensor,
811:         *scale_a_tensor,
812:         *scale_b_tensor,
813:         pointer_to_optional(bias_tensor),
814:         pointer_to_optional(scale_result_tensor),
815:         pointer_to_optional<c10::ScalarType>(out_dtype),
816:         use_fast_accum);
817:     *ret0 = new_tensor_handle(std::move(r0));
818:   });
819: }
820: 
821: AOTITorchError aoti_torch__scaled_mm_v2(
822:     AtenTensorHandle self,
823:     AtenTensorHandle mat2,
824:     AtenTensorHandle scale_a,
825:     AtenTensorHandle scale_b,
826:     AtenTensorHandle bias,
827:     AtenTensorHandle scale_result,
828:     int32_t* out_dtype,
829:     int8_t use_fast_accum,
830:     AtenTensorHandle* ret0) {
831:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
832:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
```

- EN: The main execution path in this span is carried by `tensor_handle_to_tensor_pointer`, `_scaled_mm`, `pointer_to_optional`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `tensor_handle_to_tensor_pointer`, `_scaled_mm`, `pointer_to_optional` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 833-864

```cpp
833:     at::Tensor* mat2_tensor = tensor_handle_to_tensor_pointer(mat2);
834:     at::Tensor* bias_tensor = tensor_handle_to_tensor_pointer(bias);
835:     at::Tensor* scale_a_tensor = tensor_handle_to_tensor_pointer(scale_a);
836:     at::Tensor* scale_b_tensor = tensor_handle_to_tensor_pointer(scale_b);
837:     at::Tensor* scale_result_tensor =
838:         tensor_handle_to_tensor_pointer(scale_result);
839:     auto r0 = at::_scaled_mm(
840:         *self_tensor,
841:         *mat2_tensor,
842:         *scale_a_tensor,
843:         *scale_b_tensor,
844:         pointer_to_optional(bias_tensor),
845:         pointer_to_optional(scale_result_tensor),
846:         pointer_to_optional<c10::ScalarType>(out_dtype),
847:         use_fast_accum);
848:     *ret0 = new_tensor_handle(std::move(r0));
849:   });
850: }
851: 
852: // TODO: implement a more efficient version instead of calling into aten
853: AOTITorchError aoti_torch_tensor_copy_(
854:     AtenTensorHandle src,
855:     AtenTensorHandle dst) {
856:   return aoti_torch_copy_(dst, src, /*non_blocking=*/0);
857: }
858: 
859: AOTITorchError aoti_torch_assign_tensors(
860:     AtenTensorHandle src,
861:     AtenTensorHandle dst) {
862:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
863:     at::Tensor* src_tensor = tensor_handle_to_tensor_pointer(src);
864:     at::Tensor* dst_tensor = tensor_handle_to_tensor_pointer(dst);
```

- EN: The main execution path in this span is carried by `tensor_handle_to_tensor_pointer`, `_scaled_mm`, `pointer_to_optional`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `tensor_handle_to_tensor_pointer`, `_scaled_mm`, `pointer_to_optional` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 865-896

```cpp
865:     *dst_tensor = *src_tensor;
866:   });
867: }
868: 
869: AOTITorchError aoti_torch_assign_tensors_out(
870:     AtenTensorHandle src,
871:     AtenTensorHandle* ret_dst) {
872:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
873:     at::Tensor* src_tensor_ptr = tensor_handle_to_tensor_pointer(src);
874:     at::Tensor dst_tensor = *src_tensor_ptr;
875:     *ret_dst = new_tensor_handle(std::move(dst_tensor));
876:   });
877: }
878: 
879: AOTITorchError aoti_torch_clone(AtenTensorHandle self, AtenTensorHandle* ret) {
880:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
881:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
882:     *ret = new_tensor_handle(self_tensor->clone());
883:   });
884: }
885: 
886: AOTITorchError aoti_torch_as_strided(
887:     AtenTensorHandle self,
888:     const int64_t* sizes_ptr,
889:     const int64_t* strides_ptr,
890:     AtenTensorHandle* ret) {
891:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
892:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
893:     int64_t ndim = self_tensor->dim();
894:     c10::IntArrayRef sizes(sizes_ptr, ndim);
895:     c10::IntArrayRef strides(strides_ptr, ndim);
896:     at::Tensor ret_tensor = self_tensor->as_strided(sizes, strides);
```

- EN: The main execution path in this span is carried by `aoti_torch_assign_tensors_out`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_assign_tensors_out`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 897-928

```cpp
897:     *ret = new_tensor_handle(std::move(ret_tensor));
898:   });
899: }
900: 
901: AOTITorchError aoti_torch_clone_preserve_strides(
902:     AtenTensorHandle self,
903:     AtenTensorHandle* ret) {
904:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
905:     // To mimic clone_preserve_strides which is used in copy_misaligned_inputs
906:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
907:     int64_t needed_size = 1;
908:     for (int i = 0; i < self_tensor->dim(); i++) {
909:       if (self_tensor->size(i) == 0) {
910:         needed_size = 0;
911:         break;
912:       }
913:       needed_size += (self_tensor->size(i) - 1) * self_tensor->stride(i);
914:     }
915:     at::Tensor ret_tensor =
916:         self_tensor->as_strided({needed_size}, {1})
917:             .clone()
918:             .as_strided(self_tensor->sizes(), self_tensor->strides());
919:     *ret = new_tensor_handle(std::move(ret_tensor));
920:   });
921: }
922: 
923: // TODO: implement a more efficient version instead of calling into aten
924: AOTITorchError aoti_torch_addmm_out(
925:     AtenTensorHandle out,
926:     AtenTensorHandle self,
927:     AtenTensorHandle mat1,
928:     AtenTensorHandle mat2,
```

- EN: The main execution path in this span is carried by `new_tensor_handle`, `aoti_torch_clone_preserve_strides`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `new_tensor_handle`, `aoti_torch_clone_preserve_strides`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 929-960

```cpp
929:     float beta,
930:     float alpha) {
931:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
932:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
933:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
934:     at::Tensor* mat1_tensor = tensor_handle_to_tensor_pointer(mat1);
935:     at::Tensor* mat2_tensor = tensor_handle_to_tensor_pointer(mat2);
936:     at::addmm_out(
937:         *out_tensor, *self_tensor, *mat1_tensor, *mat2_tensor, beta, alpha);
938:   });
939: }
940: 
941: // TODO: implement a more efficient version instead of calling into aten
942: AOTITorchError aoti_torch_bmm_out(
943:     AtenTensorHandle out,
944:     AtenTensorHandle self,
945:     AtenTensorHandle mat2) {
946:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
947:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
948:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
949:     at::Tensor* mat2_tensor = tensor_handle_to_tensor_pointer(mat2);
950:     at::bmm_out(*out_tensor, *self_tensor, *mat2_tensor);
951:   });
952: }
953: 
954: AOTITorchError aoti_torch_copy_(
955:     AtenTensorHandle self,
956:     AtenTensorHandle src,
957:     int32_t non_blocking) {
958:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
959:     tensor_handle_to_tensor_pointer(self)->copy_(
960:         *tensor_handle_to_tensor_pointer(src), non_blocking);
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `addmm_out`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `addmm_out` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 961-992

```cpp
961:   });
962: }
963: 
964: // TODO: implement a more efficient version instead of calling into aten
965: AOTITorchError aoti_torch_mm_out(
966:     AtenTensorHandle out,
967:     AtenTensorHandle self,
968:     AtenTensorHandle mat2) {
969:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
970:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
971:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
972:     at::Tensor* mat2_tensor = tensor_handle_to_tensor_pointer(mat2);
973:     at::mm_out(*out_tensor, *self_tensor, *mat2_tensor);
974:   });
975: }
976: 
977: AOTITorchError aoti_torch__mm_plus_mm_out(
978:     AtenTensorHandle out,
979:     AtenTensorHandle a,
980:     AtenTensorHandle b,
981:     AtenTensorHandle c,
982:     AtenTensorHandle d) {
983:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
984:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
985:     at::Tensor* a_tensor = tensor_handle_to_tensor_pointer(a);
986:     at::Tensor* b_tensor = tensor_handle_to_tensor_pointer(b);
987:     at::Tensor* c_tensor = tensor_handle_to_tensor_pointer(c);
988:     at::Tensor* d_tensor = tensor_handle_to_tensor_pointer(d);
989:     torch::inductor::_mm_plus_mm_out(
990:         *out_tensor, *a_tensor, *b_tensor, *c_tensor, *d_tensor);
991:   });
992: }
```

- EN: The main execution path in this span is carried by `aoti_torch_mm_out`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_mm_out`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 993-1024

```cpp
 993: 
 994: AOTITorchError aoti_torch_cpu_wrapped_fbgemm_pack_gemm_matrix_fp16(
 995:     AtenTensorHandle weight,
 996:     AtenTensorHandle* out) {
 997:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
 998:     at::Tensor* weight_tensor = tensor_handle_to_tensor_pointer(weight);
 999: 
1000:     *out = new_tensor_handle(at::fbgemm_pack_gemm_matrix_fp16(*weight_tensor));
1001:   });
1002: }
1003: 
1004: AOTITorchError aoti_torch_cpu__wrapped_linear_prepack(
1005:     AtenTensorHandle weight,
1006:     AtenTensorHandle weight_scale,
1007:     AtenTensorHandle weight_zero_point,
1008:     AtenTensorHandle bias,
1009:     AtenTensorHandle* out) {
1010:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1011:     at::Tensor* weight_tensor = tensor_handle_to_tensor_pointer(weight);
1012:     at::Tensor* weight_scale_tensor =
1013:         tensor_handle_to_tensor_pointer(weight_scale);
1014:     at::Tensor* weight_zero_point_tensor =
1015:         tensor_handle_to_tensor_pointer(weight_zero_point);
1016:     at::Tensor* bias_tensor = tensor_handle_to_tensor_pointer(bias);
1017: 
1018:     *out = new_tensor_handle(at::_wrapped_linear_prepack(
1019:         *weight_tensor,
1020:         *weight_scale_tensor,
1021:         *weight_zero_point_tensor,
1022:         *bias_tensor));
1023:   });
1024: }
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_wrapped_fbgemm_pack_gemm_matrix_fp16`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_wrapped_fbgemm_pack_gemm_matrix_fp16`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1025-1056

```cpp
1025: 
1026: AOTITorchError aoti_torch_cpu_wrapped_fbgemm_linear_fp16_weight(
1027:     AtenTensorHandle input,
1028:     AtenTensorHandle weight,
1029:     AtenTensorHandle bias, // optional argument
1030:     int64_t out_channel,
1031:     AtenTensorHandle* out) {
1032:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1033:     at::Tensor* input_tensor = tensor_handle_to_tensor_pointer(input);
1034:     at::Tensor* weight_tensor = tensor_handle_to_tensor_pointer(weight);
1035:     auto optional_bias_tensor =
1036:         pointer_to_optional(tensor_handle_to_tensor_pointer(bias));
1037: 
1038:     *out = new_tensor_handle(at::fbgemm_linear_fp16_weight_fp32_activation(
1039:         *input_tensor, *weight_tensor, optional_bias_tensor));
1040:   });
1041: }
1042: 
1043: AOTITorchError aoti_torch_cpu__wrapped_quantized_linear_prepacked(
1044:     AtenTensorHandle input,
1045:     AtenTensorHandle input_scale,
1046:     AtenTensorHandle input_zero_point,
1047:     AtenTensorHandle weight,
1048:     AtenTensorHandle out_scale,
1049:     AtenTensorHandle out_zeropoint,
1050:     int64_t out_channel,
1051:     AtenTensorHandle* out) {
1052:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1053:     at::Tensor* input_tensor = tensor_handle_to_tensor_pointer(input);
1054:     at::Tensor* input_scale_tensor =
1055:         tensor_handle_to_tensor_pointer(input_scale);
1056:     at::Tensor* input_zero_point_tensor =
```

- EN: The main execution path in this span is carried by `aoti_torch_cpu_wrapped_fbgemm_linear_fp16_weight`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_cpu_wrapped_fbgemm_linear_fp16_weight`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1057-1088

```cpp
1057:         tensor_handle_to_tensor_pointer(input_zero_point);
1058:     at::Tensor* weight_tensor = tensor_handle_to_tensor_pointer(weight);
1059:     at::Tensor* out_scale_tensor = tensor_handle_to_tensor_pointer(out_scale);
1060:     at::Tensor* out_zeropoint_tensor =
1061:         tensor_handle_to_tensor_pointer(out_zeropoint);
1062:     *out = new_tensor_handle(at::_wrapped_quantized_linear_prepacked(
1063:         *input_tensor,
1064:         *input_scale_tensor,
1065:         *input_zero_point_tensor,
1066:         *weight_tensor,
1067:         *out_scale_tensor,
1068:         *out_zeropoint_tensor,
1069:         out_channel));
1070:   });
1071: }
1072: 
1073: AOTITorchError aoti_torch_nonzero(
1074:     AtenTensorHandle self,
1075:     AtenTensorHandle* out) {
1076:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1077:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
1078:     *out = new_tensor_handle(at::nonzero(*self_tensor));
1079:   });
1080: }
1081: 
1082: AOTITorchError aoti_torch_repeat_interleave_Tensor(
1083:     AtenTensorHandle repeats,
1084:     int64_t* output_size,
1085:     AtenTensorHandle* out) {
1086:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1087:     at::Tensor* repeats_tensor = tensor_handle_to_tensor_pointer(repeats);
1088:     *out = new_tensor_handle(at::_ops::repeat_interleave_Tensor::call(
```

- EN: The main execution path in this span is carried by `tensor_handle_to_tensor_pointer`, `new_tensor_handle`, `aoti_torch_nonzero`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `tensor_handle_to_tensor_pointer`, `new_tensor_handle`, `aoti_torch_nonzero` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1089-1120

```cpp
1089:         *repeats_tensor, pointer_to_optional<c10::SymInt>(output_size)));
1090:   });
1091: }
1092: 
1093: // Function to check existence of inf and NaN
1094: AOTITorchError aoti_torch_check_inf_and_nan(
1095:     const char* tensor_name,
1096:     AtenTensorHandle tensor) {
1097:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1098:     at::Tensor* check_tensor = tensor_handle_to_tensor_pointer(tensor);
1099: 
1100:     assert_inf_and_nan(tensor_name, *check_tensor);
1101:   });
1102: }
1103: 
1104: AOTITorchError aoti_record_function_start(
1105:     const char* name,
1106:     IValueMapHandle kwargs,
1107:     const C10IValueHandle* inputs,
1108:     const uint64_t n_inputs,
1109:     AtenRecordFunctionHandle* guard) {
1110:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1111:     at::RecordFunction* newGuard =
1112:         new at::RecordFunction(at::RecordScope::FUNCTION);
1113:     std::unordered_map<std::string, c10::IValue> recordKwargs;
1114: 
1115:     if (kwargs != nullptr) {
1116:       auto wrappedKwargs =
1117:           reinterpret_cast<std::unordered_map<std::string, C10IValueHandle>*>(
1118:               kwargs);
1119:       for (const auto& pair : *wrappedKwargs) {
1120:         recordKwargs.emplace(
```

- EN: The main execution path in this span is carried by `aoti_torch_check_inf_and_nan`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_check_inf_and_nan`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1121-1152

```cpp
1121:             pair.first, *(reinterpret_cast<c10::IValue*>(pair.second)));
1122:       }
1123:     }
1124: 
1125:     std::vector<c10::IValue> recordInputs(n_inputs);
1126:     for (size_t i = 0; i < n_inputs; i++) {
1127:       recordInputs[i] = *reinterpret_cast<c10::IValue*>(inputs[i]);
1128:     }
1129: 
1130:     newGuard->before(name, &recordInputs, &recordKwargs);
1131:     *guard = reinterpret_cast<AtenRecordFunctionHandle>(newGuard);
1132:   });
1133: }
1134: 
1135: AOTITorchError aoti_record_function_end(AtenRecordFunctionHandle guard) {
1136:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1137:     at::RecordFunction* t = reinterpret_cast<at::RecordFunction*>(guard);
1138: 
1139:     delete t;
1140:   });
1141: }
1142: 
1143: AOTITorchError aoti_torch_scatter_out(
1144:     AtenTensorHandle out,
1145:     AtenTensorHandle self,
1146:     int64_t dim,
1147:     AtenTensorHandle index,
1148:     AtenTensorHandle src) {
1149:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1150:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
1151:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
1152:     at::Tensor* index_tensor = tensor_handle_to_tensor_pointer(index);
```

- EN: The main execution path in this span is carried by `recordInputs`, `aoti_record_function_end`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `recordInputs`, `aoti_record_function_end`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1153-1184

```cpp
1153:     at::Tensor* src_tensor = tensor_handle_to_tensor_pointer(src);
1154:     at::scatter_out(*out_tensor, *self_tensor, dim, *index_tensor, *src_tensor);
1155:   });
1156: }
1157: 
1158: AOTITorchError aoti_torch_scatter_reduce_out(
1159:     AtenTensorHandle out,
1160:     AtenTensorHandle self,
1161:     int64_t dim,
1162:     AtenTensorHandle index,
1163:     AtenTensorHandle src,
1164:     const char* reduce,
1165:     int32_t include_self) {
1166:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1167:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
1168:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
1169:     at::Tensor* index_tensor = tensor_handle_to_tensor_pointer(index);
1170:     at::Tensor* src_tensor = tensor_handle_to_tensor_pointer(src);
1171:     at::scatter_reduce_out(
1172:         *out_tensor,
1173:         *self_tensor,
1174:         dim,
1175:         *index_tensor,
1176:         *src_tensor,
1177:         reduce,
1178:         (bool)include_self);
1179:   });
1180: }
1181: 
1182: AOTITorchError aoti_torch_index_put_out(
1183:     AtenTensorHandle out,
1184:     AtenTensorHandle self,
```

- EN: The main execution path in this span is carried by `tensor_handle_to_tensor_pointer`, `scatter_out`, `aoti_torch_scatter_reduce_out`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `tensor_handle_to_tensor_pointer`, `scatter_out`, `aoti_torch_scatter_reduce_out` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1185-1216

```cpp
1185:     const AtenTensorHandle* indices,
1186:     const uint32_t num_indices,
1187:     // NOLINTNEXTLINE(misc-misplaced-const)
1188:     const AtenTensorHandle values,
1189:     bool accumulate) {
1190:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1191:     c10::List<std::optional<at::Tensor>> indices_;
1192:     indices_.reserve(num_indices);
1193:     for (size_t i = 0; i < num_indices; i++) {
1194:       indices_.emplace_back(
1195:           pointer_to_optional(tensor_handle_to_tensor_pointer(indices[i])));
1196:     }
1197:     at::Tensor* out_tensor = tensor_handle_to_tensor_pointer(out);
1198:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
1199:     at::Tensor* values_tensor = tensor_handle_to_tensor_pointer(values);
1200:     at::index_put_out(
1201:         *out_tensor, *self_tensor, indices_, *values_tensor, accumulate);
1202:   });
1203: }
1204: 
1205: AOTITorchError aoti_torch_view_as_real(
1206:     AtenTensorHandle self,
1207:     AtenTensorHandle* ret // returns new reference
1208: ) {
1209:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1210:     *ret = new_tensor_handle(
1211:         at::_ops::view_as_real::call(*tensor_handle_to_tensor_pointer(self)));
1212:   });
1213: }
1214: 
1215: AOTITorchError aoti_torch_view_dtype(
1216:     AtenTensorHandle self,
```

- EN: The main execution path in this span is carried by `NOLINTNEXTLINE`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `pointer_to_optional`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `NOLINTNEXTLINE`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `pointer_to_optional` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1217-1248

```cpp
1217:     int32_t dtype,
1218:     AtenTensorHandle* ret // returns new reference
1219: ) {
1220:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1221:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
1222:     *ret = new_tensor_handle(at::_ops::view_dtype::call(
1223:         *self_tensor, static_cast<c10::ScalarType>(dtype)));
1224:   });
1225: }
1226: 
1227: void aoti_torch_save_tensor_handle(
1228:     AtenTensorHandle self,
1229:     const char* tensor_name,
1230:     const char* launch_prefix,
1231:     const char* kernel_name) {
1232:   at::Tensor* t = tensor_handle_to_tensor_pointer(self);
1233: #ifndef C10_MOBILE
1234:   // Save tensor to tmp .pt file for tensors and can be torch.load'ed later
1235:   auto cwd = c10::filesystem::current_path();
1236:   auto tmp_folder = cwd / "tmp" / "aoti_torch";
1237:   if (!c10::filesystem::exists(tmp_folder)) {
1238:     std::cout
1239:         << "aoti_torch_save_tensor_handle: Path does not exist, creating it..."
1240:         << tmp_folder << '\n';
1241: 
1242:     std::error_code ec{};
1243:     if (!c10::filesystem::create_directories(tmp_folder, ec)) {
1244:       std::cout << "aoti_torch_save_tensor_handle: Error creating directory: "
1245:                 << tmp_folder << " error:" << ec.message() << '\n';
1246:       return;
1247:     }
1248:   }
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `new_tensor_handle`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `tensor_handle_to_tensor_pointer`, `new_tensor_handle` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1249-1280

```cpp
1249:   std::string tensor_filepath_to_save = tmp_folder.string() + launch_prefix +
1250:       "_" + kernel_name + "_" + tensor_name + "_" + t->device().str() + ".pt";
1251: 
1252:   auto bytes = torch::jit::pickle_save(c10::IValue(*t));
1253:   std::ofstream fout(tensor_filepath_to_save, std::ios::out | std::ios::binary);
1254:   fout.write(bytes.data(), static_cast<std::streamsize>(bytes.size()));
1255:   fout.close();
1256: 
1257:   std::cout << "aoti_torch_save_tensor_handle: Saved tensor to "
1258:             << tensor_filepath_to_save << '\n';
1259: #endif // !defined(C10_MOBILE)
1260: }
1261: 
1262: void aoti_torch_print_tensor_handle(AtenTensorHandle self, const char* msg) {
1263:   at::Tensor* t = tensor_handle_to_tensor_pointer(self);
1264: 
1265:   // Display message
1266:   std::cout << '[';
1267:   if (msg) {
1268:     std::cout << "  " << msg;
1269:   }
1270:   std::cout << "  " << "]:" << '\n';
1271: 
1272:   // Print exact tensor values for small size tensors
1273:   const int64_t numel = t->numel();
1274:   if (numel <= AOTI_TORCH_MAX_NUMEL_TO_PRINT) {
1275:     std::cout << *t << '\n';
1276:   }
1277: 
1278:   // Print summary stats of the tensor
1279:   std::cout << "Number of elements: " << numel << '\n';
1280: 
```

- EN: The main execution path in this span is carried by `pickle_save`, `fout`, `aoti_torch_print_tensor_handle`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `pickle_save`, `fout`, `aoti_torch_print_tensor_handle` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1281-1312

```cpp
1281:   // Print dtypes and for float types, print exact precision
1282:   auto scalarType = t->scalar_type();
1283:   if (scalarType == at::ScalarType::Float) {
1284:     std::cout << "Dtype: float32" << std::endl;
1285:   } else if (scalarType == at::ScalarType::Half) {
1286:     std::cout << "Dtype: float16" << std::endl;
1287:   } else if (scalarType == at::ScalarType::BFloat16) {
1288:     std::cout << "Dtype: bfloat16" << std::endl;
1289:   } else {
1290:     std::cout << "Dtype: " << t->dtype() << '\n';
1291:   }
1292: 
1293:   if (numel > 0) {
1294:     // torch/aten `mean()` function only supports float and complex dtypes
1295:     // See:
1296:     // https://github.com/pytorch/pytorch/blob/a0e062c6f1a03ec93e87413e42c4d0b336518131/aten/src/ATen/native/ReduceOps.cpp#L304-L309
1297:     auto mean_value = [t](at::ScalarType dtype) {
1298:       return t->to(dtype).mean().item();
1299:     };
1300:     bool is_complex_type =
1301:         at::isComplexType(at::typeMetaToScalarType(t->dtype()));
1302:     at::ScalarType float_dtype =
1303:         is_complex_type ? at::kComplexFloat : at::kFloat;
1304:     std::cout << "Mean value: " << mean_value(float_dtype) << '\n';
1305:     if (!is_complex_type) {
1306:       // "min_all_cuda" function is not implemented for 'ComplexFloat' type.
1307:       // (similar for max) Skip printing min/max value for complex type tensors
1308:       // here if encountered complex (rare occasions), suggest to print
1309:       // out the whole value of the tensor.
1310:       std::cout << "Min value: " << t->to(float_dtype).min().item() << '\n';
1311:       std::cout << "Max value: " << t->to(float_dtype).max().item() << '\n';
1312:     } else {
```

- EN: The main execution path in this span is carried by `isComplexType`, `mean_value`, `complex`. Control-flow branches in this range handle alternative runtime cases or iterative work. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `isComplexType`, `mean_value`, `complex` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1313-1344

```cpp
1313:       // Set the numel threshold to print as 256 to avoid printing out too much
1314:       // More info for aten native cuda kernel for "min_all_cuda" implementation
1315:       // source:
1316:       // https://github.com/pytorch/pytorch/blob/4b3983241263b03abd25ae381ae4743ac49b648e/aten/src/ATen/native/cuda/ReduceMinValuesKernel.cu#L51
1317:       if (numel <= 256) {
1318:         std::cout
1319:             << "[INFO] Aten built-in function `min_all_cuda/max_all_cuda` not implemented for current dtype: "
1320:             << t->dtype() << ". Printing out the whole value:\n"
1321:             << *t << '\n';
1322:       }
1323:     }
1324:   }
1325:   std::cout << "Device: " << t->device() << '\n';
1326:   std::cout << "Size: " << t->sizes() << '\n';
1327:   std::cout << "Stride: " << t->strides() << '\n';
1328:   std::cout << "Layout: " << t->layout() << '\n';
1329:   std::cout << "Is contiguous: " << t->is_contiguous() << '\n';
1330:   std::cout << "Requires grad: " << t->requires_grad() << '\n';
1331: 
1332:   std::cout << '\n';
1333: }
1334: 
1335: // ProxyExecutor
1336: AOTITorchError aoti_torch_proxy_executor_call_function(
1337:     AOTIProxyExecutorHandle proxy_executor,
1338:     int extern_node_index,
1339:     int num_ints,
1340:     int64_t* flatten_int_args,
1341:     int num_tensors,
1342:     AtenTensorHandle* flatten_tensor_args) {
1343:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1344:     if (!proxy_executor) {
```

- EN: The main execution path in this span is carried by `aoti_torch_proxy_executor_call_function`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_proxy_executor_call_function`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1345-1376

```cpp
1345:       throw std::runtime_error(
1346:           "Unable to find a proxy executor to run custom ops. Please check if "
1347:           "there is a json file generated in the same directory as the so, or use "
1348:           "torch._inductor.aoti_compile_and_package to package everything into a "
1349:           "PT2 artifact.");
1350:     }
1351:     ProxyExecutor* executor = reinterpret_cast<ProxyExecutor*>(proxy_executor);
1352:     executor->call_function(
1353:         extern_node_index,
1354:         num_ints,
1355:         flatten_int_args,
1356:         num_tensors,
1357:         flatten_tensor_args);
1358:   });
1359: }
1360: 
1361: void aoti_torch_check(
1362:     bool cond,
1363:     const char* func,
1364:     const char* file,
1365:     uint32_t line,
1366:     const char* msg) {
1367:   if (C10_UNLIKELY_OR_CONST(!cond)) {
1368:     ::c10::detail::torchCheckFail(func, file, line, msg);
1369:   }
1370: }
1371: 
1372: void aoti_torch_warn(
1373:     const char* func,
1374:     const char* file,
1375:     uint32_t line,
1376:     const char* msg) {
```

- EN: The main execution path in this span is carried by `runtime_error`, `aoti_torch_check`, `torchCheckFail`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `runtime_error`, `aoti_torch_check`, `torchCheckFail` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1377-1408

```cpp
1377:   ::c10::warn(
1378:       ::c10::Warning(::c10::UserWarning(), {func, file, line}, msg, false));
1379: }
1380: 
1381: AOTITorchError aoti_torch__alloc_from_pool(
1382:     AtenTensorHandle self,
1383:     int64_t offset_bytes,
1384:     int32_t dtype,
1385:     int64_t ndim,
1386:     const int64_t* sizes_ptr,
1387:     const int64_t* strides_ptr,
1388:     AtenTensorHandle* ret_new_tensor) {
1389:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1390:     at::Tensor* self_tensor = tensor_handle_to_tensor_pointer(self);
1391:     c10::IntArrayRef sizes(sizes_ptr, ndim);
1392:     c10::IntArrayRef strides(strides_ptr, ndim);
1393:     *ret_new_tensor = new_tensor_handle(torch::inductor::_alloc_from_pool(
1394:         *self_tensor,
1395:         offset_bytes,
1396:         static_cast<c10::ScalarType>(dtype),
1397:         sizes,
1398:         strides));
1399:   });
1400: }
1401: 
1402: AOTITorchError aoti_torch_zero_(AtenTensorHandle tensor) {
1403:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1404:     at::Tensor* t = tensor_handle_to_tensor_pointer(tensor);
1405:     t->zero_();
1406:   });
1407: }
1408: 
```

- EN: The main execution path in this span is carried by `warn`, `Warning`, `aoti_torch__alloc_from_pool`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `warn`, `Warning`, `aoti_torch__alloc_from_pool` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1409-1440

```cpp
1409: AOTITorchError aoti_torch_library_init_impl(
1410:     const char* ns,
1411:     const char* k,
1412:     const char* file,
1413:     uint32_t line,
1414:     TorchLibraryHandle* ret_new_torch_lib) {
1415:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1416:     *ret_new_torch_lib =
1417:         reinterpret_cast<TorchLibraryOpaque*>(new torch::Library(
1418:             torch::Library::Kind::IMPL,
1419:             std::string(ns),
1420:             c10::parseDispatchKey(std::string(k)),
1421:             file,
1422:             line));
1423:   });
1424: }
1425: 
1426: AOTITorchError aoti_torch_library_init_def(
1427:     const char* ns,
1428:     const char* file,
1429:     uint32_t line,
1430:     TorchLibraryHandle* ret_new_torch_lib) {
1431:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1432:     *ret_new_torch_lib =
1433:         reinterpret_cast<TorchLibraryOpaque*>(new torch::Library(
1434:             torch::Library::Kind::DEF,
1435:             std::string(ns),
1436:             std::nullopt,
1437:             file,
1438:             line));
1439:   });
1440: }
```

- EN: The main execution path in this span is carried by `aoti_torch_library_init_impl`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `string`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_library_init_impl`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `string` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1441-1472

```cpp
1441: 
1442: AOTITorchError aoti_torch_library_init_fragment(
1443:     const char* ns,
1444:     const char* file,
1445:     uint32_t line,
1446:     TorchLibraryHandle* ret_new_torch_lib) {
1447:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1448:     *ret_new_torch_lib =
1449:         reinterpret_cast<TorchLibraryOpaque*>(new torch::Library(
1450:             torch::Library::Kind::FRAGMENT,
1451:             std::string(ns),
1452:             std::nullopt,
1453:             file,
1454:             line));
1455:   });
1456: }
1457: 
1458: AOTI_TORCH_EXPORT AOTITorchError
1459: aoti_torch_library_def(TorchLibraryHandle self, const char* name) {
1460:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
1461:       { reinterpret_cast<torch::Library*>(self)->def(name); });
1462: }
1463: 
1464: AOTI_TORCH_EXPORT AOTITorchError
1465: aoti_torch_delete_library_object(TorchLibraryHandle tlh) {
1466:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
1467:       { delete reinterpret_cast<torch::Library*>(tlh); });
1468: }
1469: 
1470: AOTITorchError aoti_torch_create_device_guard(
1471:     int32_t device_index,
1472:     DeviceGuardHandle* ret_guard // returns new reference
```

- EN: The main execution path in this span is carried by `aoti_torch_library_init_fragment`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `string`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_library_init_fragment`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `string` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1473-1504

```cpp
1473: ) {
1474:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1475:     // checked=true will fail if no accelerator is available
1476:     const auto device_type =
1477:         at::accelerator::getAccelerator(/*checked=*/true).value();
1478:     c10::Device device(device_type, device_index);
1479:     c10::DeviceGuard* guard = new c10::DeviceGuard(device);
1480:     *ret_guard = reinterpret_cast<DeviceGuardHandle>(guard);
1481:   });
1482: }
1483: 
1484: AOTITorchError aoti_torch_delete_device_guard(DeviceGuardHandle guard) {
1485:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
1486:       { delete reinterpret_cast<c10::DeviceGuard*>(guard); });
1487: }
1488: 
1489: AOTITorchError aoti_torch_device_guard_set_index(
1490:     DeviceGuardHandle guard,
1491:     int32_t device_index) {
1492:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
1493:       { reinterpret_cast<c10::DeviceGuard*>(guard)->set_index(device_index); });
1494: }
1495: 
1496: AOTITorchError aoti_torch_delete_stream(StreamHandle stream) {
1497:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
1498:       { delete reinterpret_cast<c10::Stream*>(stream); });
1499: }
1500: 
1501: AOTITorchError aoti_torch_stream_id(
1502:     StreamHandle stream,
1503:     int64_t* ret_stream_id) {
1504:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
```

- EN: The main execution path in this span is carried by `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `getAccelerator`, `device`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `getAccelerator`, `device` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1505-1525

```cpp
1505:     c10::Stream* stream_ptr = reinterpret_cast<c10::Stream*>(stream);
1506:     *ret_stream_id = stream_ptr->id();
1507:   });
1508: }
1509: 
1510: // This function creates a new Stream object and makes StreamHandle point to it.
1511: // The caller is responsible for managing the object's lifecycle.
1512: AOTITorchError aoti_torch_get_current_stream(
1513:     int32_t device_index,
1514:     StreamHandle* ret_stream) {
1515:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE({
1516:     c10::Stream stream = at::accelerator::getCurrentStream(device_index);
1517:     c10::Stream* stream_ptr = new c10::Stream(stream);
1518:     *ret_stream = reinterpret_cast<StreamHandle>(stream_ptr);
1519:   });
1520: }
1521: 
1522: AOTITorchError aoti_torch_get_current_device_index(int32_t* ret_device_index) {
1523:   AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE(
1524:       { *ret_device_index = at::accelerator::getDeviceIndex(); });
1525: }
```

- EN: The main execution path in this span is carried by `aoti_torch_get_current_stream`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `getCurrentStream`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `aoti_torch_get_current_stream`, `AOTI_TORCH_CONVERT_EXCEPTION_TO_ERROR_CODE`, `getCurrentStream` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。

## Key Concepts / 关键概念
- Inductor / AOTInductor runtime integration / Inductor / AOTInductor 运行时集成
- Dispatch-key routing and redispatch / DispatchKey 路由与再次分发
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Tensor/value representation management / 张量/数值表示管理
- Runtime validation and diagnostics / 运行时校验与诊断
- Stable ABI / C interface exposure / 稳定 ABI / C 接口暴露

## Dependencies / 依赖关系
- Direct includes / 直接包含: `ATen/native/quantized/cpu/qlinear.h`, `ATen/record_function.h`, `c10/core/DeviceType.h`, `c10/core/DispatchKey.h`, `c10/core/GradMode.h`, `c10/core/Layout.h`, `c10/core/MemoryFormat.h`, `c10/core/ScalarType.h`, `c10/util/Exception.h`, `torch/csrc/inductor/aoti_runtime/utils.h`
- Include roots / 头文件根模块: `ATen`, `c10`, `torch`
- Key symbols / 关键符号: `c10_device`, `aoti_torch_abi_version`, `aoti_torch_grad_mode_is_enabled`, `aoti_torch_grad_mode_set_enabled`, `aoti_torch_dtype_element_size`, `aoti_torch_delete_tensor_object`, `aoti_torch_delete_c10_value_object`, `aoti_torch_int64_to_ivalue`, `aoti_torch_str_to_ivalue`, `aoti_torch_strlist_to_ivalue`
- Related subsystems / 相关子系统: ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Inductor runtime / Inductor 运行时, JIT/tracing integration / JIT 与追踪集成

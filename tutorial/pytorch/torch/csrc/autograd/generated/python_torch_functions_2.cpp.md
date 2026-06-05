# python_torch_functions_2.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_torch_functions_2.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 11986
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

- Note / 说明: This file is generated or mechanically expanded, so the analysis groups contiguous line ranges rather than paraphrasing every repeated wrapper individually. / 该文件为生成代码或机械展开代码，因此分析按连续行范围组织，而不是逐个重复改写每个封装模板。
### Lines 1-80

```cpp
 1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
 2: // @generated from ../tools/autograd/templates/python_torch_functions.cpp
 3: 
 4: // Python bindings for torch.* functions implemented through ATen.
 5: //
 6: // The functions are bound as static methods on a class
 7: // torch._C._VariableFunctions which is also aliased as Variable._torch
 8: // and also copied into 'torch' module.
 9: 
10: #include <Python.h>
11: 
12: // Undefine the copysign macro so that at::copysign works as intended with MSVC
13: // https://github.com/python/cpython/blob/c60394c7fc9cc09b16e9675a3eeb5844b6d8523f/PC/pyconfig.h#L196
14: #ifdef _MSC_VER
15: #undef copysign
16: #endif // _MSC_VER
17: 
18: #include "torch/csrc/autograd/python_torch_functions.h"
19: #include "torch/csrc/autograd/python_variable.h"
20: #include "torch/csrc/autograd/utils/wrap_outputs.h"
21: #include "torch/csrc/Dtype.h"
22: #include "torch/csrc/DynamicTypes.h"
23: #include "torch/csrc/Exceptions.h"
24: #include "torch/csrc/utils/out_types.h"
25: #include "torch/csrc/utils/pybind.h"
26: #include "torch/csrc/utils/pycfunction_helpers.h"
27: #include "torch/csrc/utils/python_arg_parser.h"
28: #include "torch/csrc/utils/tensor_layouts.h"
29: #include "torch/csrc/utils/tensor_new.h"
30: #include "torch/csrc/utils/tensor_numpy.h"
31: #include "torch/csrc/jit/frontend/tracer.h"
32: #include "torch/csrc/autograd/generated/variable_factories.h"
33: #include "torch/csrc/utils/structseq.h"
34: #include "torch/csrc/utils/device_lazy_init.h"
35: #include "torch/csrc/autograd/generated/python_return_types.h"
36: 
37: #include <ATen/core/Tensor.h>
38: 
39: #ifndef AT_PER_OPERATOR_HEADERS
40: #include <ATen/Functions.h>
41: #else
42: #include <ATen/ops/_cast_Double.h>
43: #include <ATen/ops/_cast_Int.h>
44: #include <ATen/ops/align_tensors.h>
45: #include <ATen/ops/_print.h>
46: #include <ATen/ops/_make_dep_token.h>
47: #include <ATen/ops/_use_cudnn_rnn_flatten_weight.h>
48: #include <ATen/ops/_cudnn_rnn.h>
49: #include <ATen/ops/_fused_dropout.h>
50: #include <ATen/ops/_sobol_engine_ff.h>
51: #include <ATen/ops/_sobol_engine_scramble.h>
52: #include <ATen/ops/_shape_as_tensor.h>
53: #include <ATen/ops/dropout.h>
54: #include <ATen/ops/dropout.h>
55: #include <ATen/ops/_conj.h>
56: #include <ATen/ops/conj_physical.h>
57: #include <ATen/ops/conj_physical.h>
58: #include <ATen/ops/avg_pool1d.h>
59: #include <ATen/ops/add.h>
60: #include <ATen/ops/_add_relu.h>
61: #include <ATen/ops/_add_relu.h>
62: #include <ATen/ops/addr.h>
63: #include <ATen/ops/_test_functorch_fallback.h>
64: #include <ATen/ops/all.h>
65: #include <ATen/ops/allclose.h>
66: #include <ATen/ops/argmax.h>
67: #include <ATen/ops/acosh.h>
68: #include <ATen/ops/acosh.h>
69: #include <ATen/ops/arccosh.h>
70: #include <ATen/ops/arccosh.h>
71: #include <ATen/ops/arctanh.h>
72: #include <ATen/ops/arctanh.h>
73: #include <ATen/ops/arcsin.h>
74: #include <ATen/ops/arcsin.h>
75: #include <ATen/ops/atan.h>
76: #include <ATen/ops/atan.h>
77: #include <ATen/ops/baddbmm.h>
78: #include <ATen/ops/bartlett_window.h>
79: #include <ATen/ops/batch_norm.h>
80: #include <ATen/ops/copysign.h>
```

- EN: These lines pull in dependencies such as `Python.h`, `torch/csrc/autograd/python_torch_functions.h`, `torch/csrc/autograd/python_variable.h`, establishing the headers needed by the implementation. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `Python.h`, `torch/csrc/autograd/python_torch_functions.h`, `torch/csrc/autograd/python_variable.h`，为后续实现建立所需的头文件基础。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 81-160

```cpp
 81: #include <ATen/ops/_lazy_clone.h>
 82: #include <ATen/ops/logical_not.h>
 83: #include <ATen/ops/logical_xor.h>
 84: #include <ATen/ops/bmm.h>
 85: #include <ATen/ops/broadcast_tensors.h>
 86: #include <ATen/ops/broadcast_to.h>
 87: #include <ATen/ops/_sparse_broadcast_to.h>
 88: #include <ATen/ops/cat.h>
 89: #include <ATen/ops/chain_matmul.h>
 90: #include <ATen/ops/chunk.h>
 91: #include <ATen/ops/tensor_split.h>
 92: #include <ATen/ops/clamp_min.h>
 93: #include <ATen/ops/clamp_min.h>
 94: #include <ATen/ops/clip.h>
 95: #include <ATen/ops/clip.h>
 96: #include <ATen/ops/complex.h>
 97: #include <ATen/ops/polar.h>
 98: #include <ATen/ops/_convolution.h>
 99: #include <ATen/ops/_convolution_mode.h>
100: #include <ATen/ops/conv1d.h>
101: #include <ATen/ops/conv3d.h>
102: #include <ATen/ops/cos.h>
103: #include <ATen/ops/cos.h>
104: #include <ATen/ops/count_nonzero.h>
105: #include <ATen/ops/corrcoef.h>
106: #include <ATen/ops/cudnn_convolution_transpose.h>
107: #include <ATen/ops/cudnn_convolution_relu.h>
108: #include <ATen/ops/ctc_loss.h>
109: #include <ATen/ops/embedding_bag.h>
110: #include <ATen/ops/_embedding_bag.h>
111: #include <ATen/ops/empty.h>
112: #include <ATen/ops/empty_like.h>
113: #include <ATen/ops/erf.h>
114: #include <ATen/ops/erf.h>
115: #include <ATen/ops/erfc.h>
116: #include <ATen/ops/erfc.h>
117: #include <ATen/ops/exp.h>
118: #include <ATen/ops/exp.h>
119: #include <ATen/ops/unflatten.h>
120: #include <ATen/ops/floor.h>
121: #include <ATen/ops/floor.h>
122: #include <ATen/ops/floor_divide.h>
123: #include <ATen/ops/frac.h>
124: #include <ATen/ops/frac.h>
125: #include <ATen/ops/full_like.h>
126: #include <ATen/ops/grid_sampler.h>
127: #include <ATen/ops/_grid_sampler_2d_cpu_fallback.h>
128: #include <ATen/ops/grid_sampler_3d.h>
129: #include <ATen/ops/_fft_c2r.h>
130: #include <ATen/ops/_cufft_get_plan_cache_max_size.h>
131: #include <ATen/ops/_cufft_clear_plan_cache.h>
132: #include <ATen/ops/_unsafe_masked_index_put_accumulate.h>
133: #include <ATen/ops/_index_put_impl.h>
134: #include <ATen/ops/isin.h>
135: #include <ATen/ops/isnan.h>
136: #include <ATen/ops/is_conj.h>
137: #include <ATen/ops/is_inference.h>
138: #include <ATen/ops/layer_norm.h>
139: #include <ATen/ops/_fused_rms_norm.h>
140: #include <ATen/ops/nan_to_num.h>
141: #include <ATen/ops/nan_to_num.h>
142: #include <ATen/ops/mkldnn_linear_backward_weights.h>
143: #include <ATen/ops/_cslt_sparse_mm.h>
144: #include <ATen/ops/_sparse_semi_structured_mm.h>
145: #include <ATen/ops/_sparse_semi_structured_addmm.h>
146: #include <ATen/ops/_mixed_dtypes_linear.h>
147: #include <ATen/ops/fbgemm_pack_gemm_matrix_fp16.h>
148: #include <ATen/ops/_wrapped_quantized_linear_prepacked.h>
149: #include <ATen/ops/fbgemm_linear_fp16_weight_fp32_activation.h>
150: #include <ATen/ops/linspace.h>
151: #include <ATen/ops/log.h>
152: #include <ATen/ops/log.h>
153: #include <ATen/ops/log1p.h>
154: #include <ATen/ops/log1p.h>
155: #include <ATen/ops/logaddexp.h>
156: #include <ATen/ops/xlogy.h>
157: #include <ATen/ops/xlogy.h>
158: #include <ATen/ops/logsumexp.h>
159: #include <ATen/ops/matrix_power.h>
160: #include <ATen/ops/_aminmax.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/_lazy_clone.h`, `ATen/ops/logical_not.h`, `ATen/ops/logical_xor.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/_lazy_clone.h`, `ATen/ops/logical_not.h`, `ATen/ops/logical_xor.h`，为后续实现建立所需的头文件基础。
### Lines 161-240

```cpp
161: #include <ATen/ops/max.h>
162: #include <ATen/ops/quantized_max_pool1d.h>
163: #include <ATen/ops/quantized_max_pool2d.h>
164: #include <ATen/ops/mean.h>
165: #include <ATen/ops/nanmean.h>
166: #include <ATen/ops/nanmedian.h>
167: #include <ATen/ops/min.h>
168: #include <ATen/ops/amin.h>
169: #include <ATen/ops/mkldnn_convolution.h>
170: #include <ATen/ops/miopen_batch_norm.h>
171: #include <ATen/ops/miopen_depthwise_convolution.h>
172: #include <ATen/ops/_weight_int8pack_mm.h>
173: #include <ATen/ops/mv.h>
174: #include <ATen/ops/narrow.h>
175: #include <ATen/ops/batch_norm_stats.h>
176: #include <ATen/ops/batch_norm_update_stats.h>
177: #include <ATen/ops/_nnpack_available.h>
178: #include <ATen/ops/_nnpack_spatial_convolution.h>
179: #include <ATen/ops/_euclidean_dist.h>
180: #include <ATen/ops/permute.h>
181: #include <ATen/ops/pixel_shuffle.h>
182: #include <ATen/ops/native_channel_shuffle.h>
183: #include <ATen/ops/_pin_memory.h>
184: #include <ATen/ops/rad2deg.h>
185: #include <ATen/ops/rad2deg.h>
186: #include <ATen/ops/rand_like.h>
187: #include <ATen/ops/randint.h>
188: #include <ATen/ops/randint_like.h>
189: #include <ATen/ops/randn.h>
190: #include <ATen/ops/randperm.h>
191: #include <ATen/ops/reciprocal.h>
192: #include <ATen/ops/reciprocal.h>
193: #include <ATen/ops/negative.h>
194: #include <ATen/ops/negative.h>
195: #include <ATen/ops/reshape.h>
196: #include <ATen/ops/_mkldnn_reshape.h>
197: #include <ATen/ops/relu.h>
198: #include <ATen/ops/relu.h>
199: #include <ATen/ops/prelu.h>
200: #include <ATen/ops/select.h>
201: #include <ATen/ops/selu.h>
202: #include <ATen/ops/selu.h>
203: #include <ATen/ops/logit.h>
204: #include <ATen/ops/logit.h>
205: #include <ATen/ops/sin.h>
206: #include <ATen/ops/sin.h>
207: #include <ATen/ops/slice_inverse.h>
208: #include <ATen/ops/softmax.h>
209: #include <ATen/ops/unsafe_split_with_sizes.h>
210: #include <ATen/ops/split_with_sizes.h>
211: #include <ATen/ops/vsplit.h>
212: #include <ATen/ops/_chunk_cat.h>
213: #include <ATen/ops/hstack.h>
214: #include <ATen/ops/vstack.h>
215: #include <ATen/ops/dstack.h>
216: #include <ATen/ops/sum.h>
217: #include <ATen/ops/tan.h>
218: #include <ATen/ops/tan.h>
219: #include <ATen/ops/trapz.h>
220: #include <ATen/ops/_nested_tensor_from_mask_left_aligned.h>
221: #include <ATen/ops/_nested_from_padded_and_nested_example.h>
222: #include <ATen/ops/_nested_view_from_buffer.h>
223: #include <ATen/ops/_nested_get_values_copy.h>
224: #include <ATen/ops/_nested_get_lengths.h>
225: #include <ATen/ops/trunc.h>
226: #include <ATen/ops/trunc.h>
227: #include <ATen/ops/unique_dim.h>
228: #include <ATen/ops/unique_consecutive.h>
229: #include <ATen/ops/where.h>
230: #include <ATen/ops/_standard_gamma.h>
231: #include <ATen/ops/_philox_key_fold_in.h>
232: #include <ATen/ops/_philox_uniform.h>
233: #include <ATen/ops/_dirichlet_grad.h>
234: #include <ATen/ops/binomial.h>
235: #include <ATen/ops/_sparse_csr_sum.h>
236: #include <ATen/ops/_sparse_log_softmax_backward_data.h>
237: #include <ATen/ops/nuclear_norm.h>
238: #include <ATen/ops/clone.h>
239: #include <ATen/ops/heaviside.h>
240: #include <ATen/ops/_scaled_mm.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/max.h`, `ATen/ops/quantized_max_pool1d.h`, `ATen/ops/quantized_max_pool2d.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/max.h`, `ATen/ops/quantized_max_pool1d.h`, `ATen/ops/quantized_max_pool2d.h`，为后续实现建立所需的头文件基础。
### Lines 241-320

```cpp
241: #include <ATen/ops/_scaled_grouped_mm_v2.h>
242: #include <ATen/ops/_validate_sparse_compressed_tensor_args.h>
243: #include <ATen/ops/_validate_sparse_csr_tensor_args.h>
244: #include <ATen/ops/_to_cpu.h>
245: #include <ATen/ops/hspmm.h>
246: #include <ATen/ops/_to_sparse_semi_structured.h>
247: #include <ATen/ops/q_per_channel_scales.h>
248: #include <ATen/ops/q_per_channel_zero_points.h>
249: #include <ATen/ops/q_per_channel_axis.h>
250: #include <ATen/ops/_fake_quantize_per_tensor_affine_cachemask_tensor_qparams.h>
251: #include <ATen/ops/_fake_quantize_learnable_per_tensor_affine.h>
252: #include <ATen/ops/fused_moving_avg_obs_fake_quant.h>
253: #include <ATen/ops/_saturate_weight_to_fp16.h>
254: #include <ATen/ops/choose_qparams_optimized.h>
255: #include <ATen/ops/combinations.h>
256: #include <ATen/ops/result_type.h>
257: #include <ATen/ops/can_cast.h>
258: #include <ATen/ops/_lstm_mps.h>
259: #include <ATen/ops/lstm.h>
260: #include <ATen/ops/lstm_cell.h>
261: #include <ATen/ops/quantized_rnn_relu_cell.h>
262: #include <ATen/ops/quantized_rnn_tanh_cell.h>
263: #include <ATen/ops/index_add.h>
264: #include <ATen/ops/index_reduce.h>
265: #include <ATen/ops/scatter_add.h>
266: #include <ATen/ops/scatter_reduce.h>
267: #include <ATen/ops/bitwise_and.h>
268: #include <ATen/ops/bitwise_xor.h>
269: #include <ATen/ops/bitwise_left_shift.h>
270: #include <ATen/ops/rshift.h>
271: #include <ATen/ops/addbmm.h>
272: #include <ATen/ops/diag.h>
273: #include <ATen/ops/cross.h>
274: #include <ATen/ops/triu_indices.h>
275: #include <ATen/ops/trace.h>
276: #include <ATen/ops/ne.h>
277: #include <ATen/ops/eq.h>
278: #include <ATen/ops/greater.h>
279: #include <ATen/ops/take.h>
280: #include <ATen/ops/_linalg_check_errors.h>
281: #include <ATen/ops/svd.h>
282: #include <ATen/ops/cholesky.h>
283: #include <ATen/ops/_lu_with_info.h>
284: #include <ATen/ops/lu_unpack.h>
285: #include <ATen/ops/multinomial.h>
286: #include <ATen/ops/i0.h>
287: #include <ATen/ops/i0.h>
288: #include <ATen/ops/atan2.h>
289: #include <ATen/ops/hypot.h>
290: #include <ATen/ops/igamma.h>
291: #include <ATen/ops/nextafter.h>
292: #include <ATen/ops/fmin.h>
293: #include <ATen/ops/sort.h>
294: #include <ATen/ops/renorm.h>
295: #include <ATen/ops/_amp_foreach_non_finite_check_and_unscale.h>
296: #include <ATen/ops/_amp_update_scale.h>
297: #include <ATen/ops/_foreach_mul.h>
298: #include <ATen/ops/_foreach_mul.h>
299: #include <ATen/ops/_foreach_maximum.h>
300: #include <ATen/ops/_foreach_maximum.h>
301: #include <ATen/ops/_foreach_minimum.h>
302: #include <ATen/ops/_foreach_minimum.h>
303: #include <ATen/ops/_foreach_addcdiv.h>
304: #include <ATen/ops/_foreach_addcdiv.h>
305: #include <ATen/ops/_foreach_abs.h>
306: #include <ATen/ops/_foreach_abs.h>
307: #include <ATen/ops/_foreach_erf.h>
308: #include <ATen/ops/_foreach_erf.h>
309: #include <ATen/ops/_foreach_frac.h>
310: #include <ATen/ops/_foreach_frac.h>
311: #include <ATen/ops/_foreach_lgamma.h>
312: #include <ATen/ops/_foreach_lgamma.h>
313: #include <ATen/ops/_foreach_powsum.h>
314: #include <ATen/ops/_foreach_pow.h>
315: #include <ATen/ops/_foreach_pow.h>
316: #include <ATen/ops/_foreach_reciprocal.h>
317: #include <ATen/ops/_foreach_reciprocal.h>
318: #include <ATen/ops/_foreach_round.h>
319: #include <ATen/ops/_foreach_round.h>
320: #include <ATen/ops/_foreach_sinh.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/_scaled_grouped_mm_v2.h`, `ATen/ops/_validate_sparse_compressed_tensor_args.h`, `ATen/ops/_validate_sparse_csr_tensor_args.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/_scaled_grouped_mm_v2.h`, `ATen/ops/_validate_sparse_compressed_tensor_args.h`, `ATen/ops/_validate_sparse_csr_tensor_args.h`，为后续实现建立所需的头文件基础。
### Lines 321-400

```cpp
321: #include <ATen/ops/_foreach_sinh.h>
322: #include <ATen/ops/_foreach_tan.h>
323: #include <ATen/ops/_foreach_tan.h>
324: #include <ATen/ops/_foreach_copy.h>
325: #include <ATen/ops/bucketize.h>
326: #include <ATen/ops/searchsorted.h>
327: #include <ATen/ops/_convert_indices_from_coo_to_csr.h>
328: #include <ATen/ops/isfinite.h>
329: #include <ATen/ops/_remove_batch_dim.h>
330: #include <ATen/ops/_linalg_eigh.h>
331: #include <ATen/ops/outer.h>
332: #include <ATen/ops/_test_serialization_subcmul.h>
333: #include <ATen/ops/_test_autograd_multiple_dispatch_view.h>
334: #include <ATen/ops/view_as_real_copy.h>
335: #include <ATen/ops/permute_copy.h>
336: #include <ATen/ops/detach_copy.h>
337: #include <ATen/ops/split_copy.h>
338: #include <ATen/ops/unsqueeze_copy.h>
339: #include <ATen/ops/_indices_copy.h>
340: #include <ATen/ops/_values_copy.h>
341: #include <ATen/ops/unbind_copy.h>
342: #include <ATen/ops/alias_copy.h>
343: #include <ATen/ops/_nested_tensor_softmax_with_shape.h>
344: #include <ATen/ops/_safe_softmax.h>
345: #include <ATen/ops/_native_multi_head_attention.h>
346: #include <ATen/ops/_scaled_dot_product_attention_math_for_mps.h>
347: #include <ATen/ops/_scaled_dot_product_flash_attention.h>
348: #include <ATen/ops/_scaled_dot_product_flash_attention_for_cpu.h>
349: #include <ATen/ops/_triton_scaled_dot_attention.h>
350: #include <ATen/ops/_fused_adamw.h>
351: #endif
352: 
353: #include <functional>
354: #include <initializer_list>
355: #include <stdexcept>
356: #include <utility>
357: 
358: using at::Tensor;
359: using at::Device;
360: using at::Layout;
361: using at::Scalar;
362: using at::ScalarType;
363: using at::Backend;
364: using at::OptionalDeviceGuard;
365: using at::DeviceGuard;
366: using at::TensorOptions;
367: using at::IntArrayRef;
368: using at::Generator;
369: using at::TensorList;
370: using at::Dimname;
371: using at::DimnameList;
372: using at::ArrayRef;
373: 
374: using torch::utils::check_out_type_matches;
375: using namespace torch::autograd::utils;
376: 
377: // NOTE: See [Sharded File] comment in VariableType
378: 
379: namespace torch::autograd {
380: 
381: // generated forward declarations start here
382: 
383: static PyObject * THPVariable__cast_Double(PyObject* self_, PyObject* args, PyObject* kwargs);
384: static PyObject * THPVariable__cast_Int(PyObject* self_, PyObject* args, PyObject* kwargs);
385: static PyObject * THPVariable_align_tensors(PyObject* self_, PyObject* args, PyObject* kwargs);
386: static PyObject * THPVariable__print(PyObject* self_, PyObject* args, PyObject* kwargs);
387: static PyObject * THPVariable__make_dep_token(PyObject* self_, PyObject* args, PyObject* kwargs);
388: static PyObject * THPVariable__use_cudnn_rnn_flatten_weight(PyObject* self_, PyObject* args);
389: static PyObject * THPVariable__cudnn_rnn(PyObject* self_, PyObject* args, PyObject* kwargs);
390: static PyObject * THPVariable__fused_dropout(PyObject* self_, PyObject* args, PyObject* kwargs);
391: static PyObject * THPVariable__sobol_engine_ff_(PyObject* self_, PyObject* args, PyObject* kwargs);
392: static PyObject * THPVariable__sobol_engine_scramble_(PyObject* self_, PyObject* args, PyObject* kwargs);
393: static PyObject * THPVariable__shape_as_tensor(PyObject* self_, PyObject* args, PyObject* kwargs);
394: static PyObject * THPVariable_dropout(PyObject* self_, PyObject* args, PyObject* kwargs);
395: static PyObject * THPVariable_dropout_(PyObject* self_, PyObject* args, PyObject* kwargs);
396: static PyObject * THPVariable__conj(PyObject* self_, PyObject* args, PyObject* kwargs);
397: static PyObject * THPVariable_conj_physical(PyObject* self_, PyObject* args, PyObject* kwargs);
398: static PyObject * THPVariable_conj_physical_(PyObject* self_, PyObject* args, PyObject* kwargs);
399: static PyObject * THPVariable_avg_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs);
400: static PyObject * THPVariable_add(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: These lines pull in dependencies such as `ATen/ops/_foreach_sinh.h`, `ATen/ops/_foreach_tan.h`, `ATen/ops/_foreach_copy.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable__cast_Double`, `THPVariable__cast_Int`, `THPVariable_align_tensors`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/ops/_foreach_sinh.h`, `ATen/ops/_foreach_tan.h`, `ATen/ops/_foreach_copy.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable__cast_Double`, `THPVariable__cast_Int`, `THPVariable_align_tensors` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 401-480

```cpp
401: static PyObject * THPVariable__add_relu(PyObject* self_, PyObject* args, PyObject* kwargs);
402: static PyObject * THPVariable__add_relu_(PyObject* self_, PyObject* args, PyObject* kwargs);
403: static PyObject * THPVariable_addr(PyObject* self_, PyObject* args, PyObject* kwargs);
404: static PyObject * THPVariable__test_functorch_fallback(PyObject* self_, PyObject* args, PyObject* kwargs);
405: static PyObject * THPVariable_all(PyObject* self_, PyObject* args, PyObject* kwargs);
406: static PyObject * THPVariable_allclose(PyObject* self_, PyObject* args, PyObject* kwargs);
407: static PyObject * THPVariable_argmax(PyObject* self_, PyObject* args, PyObject* kwargs);
408: static PyObject * THPVariable_acosh(PyObject* self_, PyObject* args, PyObject* kwargs);
409: static PyObject * THPVariable_acosh_(PyObject* self_, PyObject* args, PyObject* kwargs);
410: static PyObject * THPVariable_arccosh(PyObject* self_, PyObject* args, PyObject* kwargs);
411: static PyObject * THPVariable_arccosh_(PyObject* self_, PyObject* args, PyObject* kwargs);
412: static PyObject * THPVariable_arctanh(PyObject* self_, PyObject* args, PyObject* kwargs);
413: static PyObject * THPVariable_arctanh_(PyObject* self_, PyObject* args, PyObject* kwargs);
414: static PyObject * THPVariable_arcsin(PyObject* self_, PyObject* args, PyObject* kwargs);
415: static PyObject * THPVariable_arcsin_(PyObject* self_, PyObject* args, PyObject* kwargs);
416: static PyObject * THPVariable_atan(PyObject* self_, PyObject* args, PyObject* kwargs);
417: static PyObject * THPVariable_atan_(PyObject* self_, PyObject* args, PyObject* kwargs);
418: static PyObject * THPVariable_baddbmm(PyObject* self_, PyObject* args, PyObject* kwargs);
419: static PyObject * THPVariable_bartlett_window(PyObject* self_, PyObject* args, PyObject* kwargs);
420: static PyObject * THPVariable_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
421: static PyObject * THPVariable_copysign(PyObject* self_, PyObject* args, PyObject* kwargs);
422: static PyObject * THPVariable__lazy_clone(PyObject* self_, PyObject* args, PyObject* kwargs);
423: static PyObject * THPVariable_logical_not(PyObject* self_, PyObject* args, PyObject* kwargs);
424: static PyObject * THPVariable_logical_xor(PyObject* self_, PyObject* args, PyObject* kwargs);
425: static PyObject * THPVariable_bmm(PyObject* self_, PyObject* args, PyObject* kwargs);
426: static PyObject * THPVariable_broadcast_tensors(PyObject* self_, PyObject* args, PyObject* kwargs);
427: static PyObject * THPVariable_broadcast_to(PyObject* self_, PyObject* args, PyObject* kwargs);
428: static PyObject * THPVariable__sparse_broadcast_to(PyObject* self_, PyObject* args, PyObject* kwargs);
429: static PyObject * THPVariable_cat(PyObject* self_, PyObject* args, PyObject* kwargs);
430: static PyObject * THPVariable_chain_matmul(PyObject* self_, PyObject* args, PyObject* kwargs);
431: static PyObject * THPVariable_chunk(PyObject* self_, PyObject* args, PyObject* kwargs);
432: static PyObject * THPVariable_tensor_split(PyObject* self_, PyObject* args, PyObject* kwargs);
433: static PyObject * THPVariable_clamp_min(PyObject* self_, PyObject* args, PyObject* kwargs);
434: static PyObject * THPVariable_clamp_min_(PyObject* self_, PyObject* args, PyObject* kwargs);
435: static PyObject * THPVariable_clip(PyObject* self_, PyObject* args, PyObject* kwargs);
436: static PyObject * THPVariable_clip_(PyObject* self_, PyObject* args, PyObject* kwargs);
437: static PyObject * THPVariable_complex(PyObject* self_, PyObject* args, PyObject* kwargs);
438: static PyObject * THPVariable_polar(PyObject* self_, PyObject* args, PyObject* kwargs);
439: static PyObject * THPVariable__convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
440: static PyObject * THPVariable__convolution_mode(PyObject* self_, PyObject* args, PyObject* kwargs);
441: static PyObject * THPVariable_conv1d(PyObject* self_, PyObject* args, PyObject* kwargs);
442: static PyObject * THPVariable_conv3d(PyObject* self_, PyObject* args, PyObject* kwargs);
443: static PyObject * THPVariable_cos(PyObject* self_, PyObject* args, PyObject* kwargs);
444: static PyObject * THPVariable_cos_(PyObject* self_, PyObject* args, PyObject* kwargs);
445: static PyObject * THPVariable_count_nonzero(PyObject* self_, PyObject* args, PyObject* kwargs);
446: static PyObject * THPVariable_corrcoef(PyObject* self_, PyObject* args, PyObject* kwargs);
447: static PyObject * THPVariable_cudnn_convolution_transpose(PyObject* self_, PyObject* args, PyObject* kwargs);
448: static PyObject * THPVariable_cudnn_convolution_relu(PyObject* self_, PyObject* args, PyObject* kwargs);
449: static PyObject * THPVariable_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
450: static PyObject * THPVariable_embedding_bag(PyObject* self_, PyObject* args, PyObject* kwargs);
451: static PyObject * THPVariable__embedding_bag(PyObject* self_, PyObject* args, PyObject* kwargs);
452: static PyObject * THPVariable_empty(PyObject* self_, PyObject* args, PyObject* kwargs);
453: static PyObject * THPVariable_empty_like(PyObject* self_, PyObject* args, PyObject* kwargs);
454: static PyObject * THPVariable_erf(PyObject* self_, PyObject* args, PyObject* kwargs);
455: static PyObject * THPVariable_erf_(PyObject* self_, PyObject* args, PyObject* kwargs);
456: static PyObject * THPVariable_erfc(PyObject* self_, PyObject* args, PyObject* kwargs);
457: static PyObject * THPVariable_erfc_(PyObject* self_, PyObject* args, PyObject* kwargs);
458: static PyObject * THPVariable_exp(PyObject* self_, PyObject* args, PyObject* kwargs);
459: static PyObject * THPVariable_exp_(PyObject* self_, PyObject* args, PyObject* kwargs);
460: static PyObject * THPVariable_unflatten(PyObject* self_, PyObject* args, PyObject* kwargs);
461: static PyObject * THPVariable_floor(PyObject* self_, PyObject* args, PyObject* kwargs);
462: static PyObject * THPVariable_floor_(PyObject* self_, PyObject* args, PyObject* kwargs);
463: static PyObject * THPVariable_floor_divide(PyObject* self_, PyObject* args, PyObject* kwargs);
464: static PyObject * THPVariable_frac(PyObject* self_, PyObject* args, PyObject* kwargs);
465: static PyObject * THPVariable_frac_(PyObject* self_, PyObject* args, PyObject* kwargs);
466: static PyObject * THPVariable_full_like(PyObject* self_, PyObject* args, PyObject* kwargs);
467: static PyObject * THPVariable_grid_sampler(PyObject* self_, PyObject* args, PyObject* kwargs);
468: static PyObject * THPVariable__grid_sampler_2d_cpu_fallback(PyObject* self_, PyObject* args, PyObject* kwargs);
469: static PyObject * THPVariable_grid_sampler_3d(PyObject* self_, PyObject* args, PyObject* kwargs);
470: static PyObject * THPVariable__fft_c2r(PyObject* self_, PyObject* args, PyObject* kwargs);
471: static PyObject * THPVariable__cufft_get_plan_cache_max_size(PyObject* self_, PyObject* args, PyObject* kwargs);
472: static PyObject * THPVariable__cufft_clear_plan_cache(PyObject* self_, PyObject* args, PyObject* kwargs);
473: static PyObject * THPVariable__unsafe_masked_index_put_accumulate(PyObject* self_, PyObject* args, PyObject* kwargs);
474: static PyObject * THPVariable__index_put_impl_(PyObject* self_, PyObject* args, PyObject* kwargs);
475: static PyObject * THPVariable_isin(PyObject* self_, PyObject* args, PyObject* kwargs);
476: static PyObject * THPVariable_isnan(PyObject* self_, PyObject* args, PyObject* kwargs);
477: static PyObject * THPVariable_is_conj(PyObject* self_, PyObject* args, PyObject* kwargs);
478: static PyObject * THPVariable_is_inference(PyObject* self_, PyObject* args, PyObject* kwargs);
479: static PyObject * THPVariable_layer_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
480: static PyObject * THPVariable__fused_rms_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable__add_relu`, `THPVariable__add_relu_`, `THPVariable_addr`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__add_relu`, `THPVariable__add_relu_`, `THPVariable_addr` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 481-560

```cpp
481: static PyObject * THPVariable_nan_to_num(PyObject* self_, PyObject* args, PyObject* kwargs);
482: static PyObject * THPVariable_nan_to_num_(PyObject* self_, PyObject* args, PyObject* kwargs);
483: static PyObject * THPVariable_mkldnn_linear_backward_weights(PyObject* self_, PyObject* args, PyObject* kwargs);
484: static PyObject * THPVariable__cslt_sparse_mm(PyObject* self_, PyObject* args, PyObject* kwargs);
485: static PyObject * THPVariable__sparse_semi_structured_mm(PyObject* self_, PyObject* args, PyObject* kwargs);
486: static PyObject * THPVariable__sparse_semi_structured_addmm(PyObject* self_, PyObject* args, PyObject* kwargs);
487: static PyObject * THPVariable__mixed_dtypes_linear(PyObject* self_, PyObject* args, PyObject* kwargs);
488: static PyObject * THPVariable_fbgemm_pack_gemm_matrix_fp16(PyObject* self_, PyObject* args, PyObject* kwargs);
489: static PyObject * THPVariable__wrapped_quantized_linear_prepacked(PyObject* self_, PyObject* args, PyObject* kwargs);
490: static PyObject * THPVariable_fbgemm_linear_fp16_weight_fp32_activation(PyObject* self_, PyObject* args, PyObject* kwargs);
491: static PyObject * THPVariable_linspace(PyObject* self_, PyObject* args, PyObject* kwargs);
492: static PyObject * THPVariable_log(PyObject* self_, PyObject* args, PyObject* kwargs);
493: static PyObject * THPVariable_log_(PyObject* self_, PyObject* args, PyObject* kwargs);
494: static PyObject * THPVariable_log1p(PyObject* self_, PyObject* args, PyObject* kwargs);
495: static PyObject * THPVariable_log1p_(PyObject* self_, PyObject* args, PyObject* kwargs);
496: static PyObject * THPVariable_logaddexp(PyObject* self_, PyObject* args, PyObject* kwargs);
497: static PyObject * THPVariable_xlogy(PyObject* self_, PyObject* args, PyObject* kwargs);
498: static PyObject * THPVariable_xlogy_(PyObject* self_, PyObject* args, PyObject* kwargs);
499: static PyObject * THPVariable_logsumexp(PyObject* self_, PyObject* args, PyObject* kwargs);
500: static PyObject * THPVariable_matrix_power(PyObject* self_, PyObject* args, PyObject* kwargs);
501: static PyObject * THPVariable__aminmax(PyObject* self_, PyObject* args, PyObject* kwargs);
502: static PyObject * THPVariable_max(PyObject* self_, PyObject* args, PyObject* kwargs);
503: static PyObject * THPVariable_quantized_max_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs);
504: static PyObject * THPVariable_quantized_max_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
505: static PyObject * THPVariable_mean(PyObject* self_, PyObject* args, PyObject* kwargs);
506: static PyObject * THPVariable_nanmean(PyObject* self_, PyObject* args, PyObject* kwargs);
507: static PyObject * THPVariable_nanmedian(PyObject* self_, PyObject* args, PyObject* kwargs);
508: static PyObject * THPVariable_min(PyObject* self_, PyObject* args, PyObject* kwargs);
509: static PyObject * THPVariable_amin(PyObject* self_, PyObject* args, PyObject* kwargs);
510: static PyObject * THPVariable_mkldnn_convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
511: static PyObject * THPVariable_miopen_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
512: static PyObject * THPVariable_miopen_depthwise_convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
513: static PyObject * THPVariable__weight_int8pack_mm(PyObject* self_, PyObject* args, PyObject* kwargs);
514: static PyObject * THPVariable_mv(PyObject* self_, PyObject* args, PyObject* kwargs);
515: static PyObject * THPVariable_narrow(PyObject* self_, PyObject* args, PyObject* kwargs);
516: static PyObject * THPVariable_batch_norm_stats(PyObject* self_, PyObject* args, PyObject* kwargs);
517: static PyObject * THPVariable_batch_norm_update_stats(PyObject* self_, PyObject* args, PyObject* kwargs);
518: static PyObject * THPVariable__nnpack_available(PyObject* self_, PyObject* args);
519: static PyObject * THPVariable__nnpack_spatial_convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
520: static PyObject * THPVariable__euclidean_dist(PyObject* self_, PyObject* args, PyObject* kwargs);
521: static PyObject * THPVariable_permute(PyObject* self_, PyObject* args, PyObject* kwargs);
522: static PyObject * THPVariable_pixel_shuffle(PyObject* self_, PyObject* args, PyObject* kwargs);
523: static PyObject * THPVariable_native_channel_shuffle(PyObject* self_, PyObject* args, PyObject* kwargs);
524: static PyObject * THPVariable__pin_memory(PyObject* self_, PyObject* args, PyObject* kwargs);
525: static PyObject * THPVariable_rad2deg(PyObject* self_, PyObject* args, PyObject* kwargs);
526: static PyObject * THPVariable_rad2deg_(PyObject* self_, PyObject* args, PyObject* kwargs);
527: static PyObject * THPVariable_rand_like(PyObject* self_, PyObject* args, PyObject* kwargs);
528: static PyObject * THPVariable_randint(PyObject* self_, PyObject* args, PyObject* kwargs);
529: static PyObject * THPVariable_randint_like(PyObject* self_, PyObject* args, PyObject* kwargs);
530: static PyObject * THPVariable_randn(PyObject* self_, PyObject* args, PyObject* kwargs);
531: static PyObject * THPVariable_randperm(PyObject* self_, PyObject* args, PyObject* kwargs);
532: static PyObject * THPVariable_reciprocal(PyObject* self_, PyObject* args, PyObject* kwargs);
533: static PyObject * THPVariable_reciprocal_(PyObject* self_, PyObject* args, PyObject* kwargs);
534: static PyObject * THPVariable_negative(PyObject* self_, PyObject* args, PyObject* kwargs);
535: static PyObject * THPVariable_negative_(PyObject* self_, PyObject* args, PyObject* kwargs);
536: static PyObject * THPVariable_reshape(PyObject* self_, PyObject* args, PyObject* kwargs);
537: static PyObject * THPVariable__mkldnn_reshape(PyObject* self_, PyObject* args, PyObject* kwargs);
538: static PyObject * THPVariable_relu(PyObject* self_, PyObject* args, PyObject* kwargs);
539: static PyObject * THPVariable_relu_(PyObject* self_, PyObject* args, PyObject* kwargs);
540: static PyObject * THPVariable_prelu(PyObject* self_, PyObject* args, PyObject* kwargs);
541: static PyObject * THPVariable_select(PyObject* self_, PyObject* args, PyObject* kwargs);
542: static PyObject * THPVariable_selu(PyObject* self_, PyObject* args, PyObject* kwargs);
543: static PyObject * THPVariable_selu_(PyObject* self_, PyObject* args, PyObject* kwargs);
544: static PyObject * THPVariable_logit(PyObject* self_, PyObject* args, PyObject* kwargs);
545: static PyObject * THPVariable_logit_(PyObject* self_, PyObject* args, PyObject* kwargs);
546: static PyObject * THPVariable_sin(PyObject* self_, PyObject* args, PyObject* kwargs);
547: static PyObject * THPVariable_sin_(PyObject* self_, PyObject* args, PyObject* kwargs);
548: static PyObject * THPVariable_slice_inverse(PyObject* self_, PyObject* args, PyObject* kwargs);
549: static PyObject * THPVariable_softmax(PyObject* self_, PyObject* args, PyObject* kwargs);
550: static PyObject * THPVariable_unsafe_split_with_sizes(PyObject* self_, PyObject* args, PyObject* kwargs);
551: static PyObject * THPVariable_split_with_sizes(PyObject* self_, PyObject* args, PyObject* kwargs);
552: static PyObject * THPVariable_vsplit(PyObject* self_, PyObject* args, PyObject* kwargs);
553: static PyObject * THPVariable__chunk_cat(PyObject* self_, PyObject* args, PyObject* kwargs);
554: static PyObject * THPVariable_hstack(PyObject* self_, PyObject* args, PyObject* kwargs);
555: static PyObject * THPVariable_vstack(PyObject* self_, PyObject* args, PyObject* kwargs);
556: static PyObject * THPVariable_dstack(PyObject* self_, PyObject* args, PyObject* kwargs);
557: static PyObject * THPVariable_sum(PyObject* self_, PyObject* args, PyObject* kwargs);
558: static PyObject * THPVariable_tan(PyObject* self_, PyObject* args, PyObject* kwargs);
559: static PyObject * THPVariable_tan_(PyObject* self_, PyObject* args, PyObject* kwargs);
560: static PyObject * THPVariable_trapz(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable_nan_to_num`, `THPVariable_nan_to_num_`, `THPVariable_mkldnn_linear_backward_weights`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_nan_to_num`, `THPVariable_nan_to_num_`, `THPVariable_mkldnn_linear_backward_weights` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 561-640

```cpp
561: static PyObject * THPVariable__nested_tensor_from_mask_left_aligned(PyObject* self_, PyObject* args, PyObject* kwargs);
562: static PyObject * THPVariable__nested_from_padded_and_nested_example(PyObject* self_, PyObject* args, PyObject* kwargs);
563: static PyObject * THPVariable__nested_view_from_buffer(PyObject* self_, PyObject* args, PyObject* kwargs);
564: static PyObject * THPVariable__nested_get_values_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
565: static PyObject * THPVariable__nested_get_lengths(PyObject* self_, PyObject* args, PyObject* kwargs);
566: static PyObject * THPVariable_trunc(PyObject* self_, PyObject* args, PyObject* kwargs);
567: static PyObject * THPVariable_trunc_(PyObject* self_, PyObject* args, PyObject* kwargs);
568: static PyObject * THPVariable_unique_dim(PyObject* self_, PyObject* args, PyObject* kwargs);
569: static PyObject * THPVariable_unique_consecutive(PyObject* self_, PyObject* args, PyObject* kwargs);
570: static PyObject * THPVariable_where(PyObject* self_, PyObject* args, PyObject* kwargs);
571: static PyObject * THPVariable__standard_gamma(PyObject* self_, PyObject* args, PyObject* kwargs);
572: static PyObject * THPVariable__philox_key_fold_in(PyObject* self_, PyObject* args, PyObject* kwargs);
573: static PyObject * THPVariable__philox_uniform_(PyObject* self_, PyObject* args, PyObject* kwargs);
574: static PyObject * THPVariable__dirichlet_grad(PyObject* self_, PyObject* args, PyObject* kwargs);
575: static PyObject * THPVariable_binomial(PyObject* self_, PyObject* args, PyObject* kwargs);
576: static PyObject * THPVariable__sparse_csr_sum(PyObject* self_, PyObject* args, PyObject* kwargs);
577: static PyObject * THPVariable__sparse_log_softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs);
578: static PyObject * THPVariable_nuclear_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
579: static PyObject * THPVariable_clone(PyObject* self_, PyObject* args, PyObject* kwargs);
580: static PyObject * THPVariable_heaviside(PyObject* self_, PyObject* args, PyObject* kwargs);
581: static PyObject * THPVariable__scaled_mm(PyObject* self_, PyObject* args, PyObject* kwargs);
582: static PyObject * THPVariable__scaled_grouped_mm_v2(PyObject* self_, PyObject* args, PyObject* kwargs);
583: static PyObject * THPVariable__validate_sparse_compressed_tensor_args(PyObject* self_, PyObject* args, PyObject* kwargs);
584: static PyObject * THPVariable__validate_sparse_csr_tensor_args(PyObject* self_, PyObject* args, PyObject* kwargs);
585: static PyObject * THPVariable__to_cpu(PyObject* self_, PyObject* args, PyObject* kwargs);
586: static PyObject * THPVariable_hspmm(PyObject* self_, PyObject* args, PyObject* kwargs);
587: static PyObject * THPVariable__to_sparse_semi_structured(PyObject* self_, PyObject* args, PyObject* kwargs);
588: static PyObject * THPVariable_q_per_channel_scales(PyObject* self_, PyObject* args, PyObject* kwargs);
589: static PyObject * THPVariable_q_per_channel_zero_points(PyObject* self_, PyObject* args, PyObject* kwargs);
590: static PyObject * THPVariable_q_per_channel_axis(PyObject* self_, PyObject* args, PyObject* kwargs);
591: static PyObject * THPVariable__fake_quantize_per_tensor_affine_cachemask_tensor_qparams(PyObject* self_, PyObject* args, PyObject* kwargs);
592: static PyObject * THPVariable__fake_quantize_learnable_per_tensor_affine(PyObject* self_, PyObject* args, PyObject* kwargs);
593: static PyObject * THPVariable_fused_moving_avg_obs_fake_quant(PyObject* self_, PyObject* args, PyObject* kwargs);
594: static PyObject * THPVariable__saturate_weight_to_fp16(PyObject* self_, PyObject* args, PyObject* kwargs);
595: static PyObject * THPVariable_choose_qparams_optimized(PyObject* self_, PyObject* args, PyObject* kwargs);
596: static PyObject * THPVariable_combinations(PyObject* self_, PyObject* args, PyObject* kwargs);
597: static PyObject * THPVariable_result_type(PyObject* self_, PyObject* args, PyObject* kwargs);
598: static PyObject * THPVariable_can_cast(PyObject* self_, PyObject* args, PyObject* kwargs);
599: static PyObject * THPVariable__lstm_mps(PyObject* self_, PyObject* args, PyObject* kwargs);
600: static PyObject * THPVariable_lstm(PyObject* self_, PyObject* args, PyObject* kwargs);
601: static PyObject * THPVariable_lstm_cell(PyObject* self_, PyObject* args, PyObject* kwargs);
602: static PyObject * THPVariable_quantized_rnn_relu_cell(PyObject* self_, PyObject* args, PyObject* kwargs);
603: static PyObject * THPVariable_quantized_rnn_tanh_cell(PyObject* self_, PyObject* args, PyObject* kwargs);
604: static PyObject * THPVariable_index_add(PyObject* self_, PyObject* args, PyObject* kwargs);
605: static PyObject * THPVariable_index_reduce(PyObject* self_, PyObject* args, PyObject* kwargs);
606: static PyObject * THPVariable_scatter_add(PyObject* self_, PyObject* args, PyObject* kwargs);
607: static PyObject * THPVariable_scatter_reduce(PyObject* self_, PyObject* args, PyObject* kwargs);
608: static PyObject * THPVariable_bitwise_and(PyObject* self_, PyObject* args, PyObject* kwargs);
609: static PyObject * THPVariable_bitwise_xor(PyObject* self_, PyObject* args, PyObject* kwargs);
610: static PyObject * THPVariable_bitwise_left_shift(PyObject* self_, PyObject* args, PyObject* kwargs);
611: static PyObject * THPVariable___rshift__(PyObject* self_, PyObject* args, PyObject* kwargs);
612: static PyObject * THPVariable_addbmm(PyObject* self_, PyObject* args, PyObject* kwargs);
613: static PyObject * THPVariable_diag(PyObject* self_, PyObject* args, PyObject* kwargs);
614: static PyObject * THPVariable_cross(PyObject* self_, PyObject* args, PyObject* kwargs);
615: static PyObject * THPVariable_triu_indices(PyObject* self_, PyObject* args, PyObject* kwargs);
616: static PyObject * THPVariable_trace(PyObject* self_, PyObject* args, PyObject* kwargs);
617: static PyObject * THPVariable_ne(PyObject* self_, PyObject* args, PyObject* kwargs);
618: static PyObject * THPVariable_eq(PyObject* self_, PyObject* args, PyObject* kwargs);
619: static PyObject * THPVariable_greater(PyObject* self_, PyObject* args, PyObject* kwargs);
620: static PyObject * THPVariable_take(PyObject* self_, PyObject* args, PyObject* kwargs);
621: static PyObject * THPVariable__linalg_check_errors(PyObject* self_, PyObject* args, PyObject* kwargs);
622: static PyObject * THPVariable_svd(PyObject* self_, PyObject* args, PyObject* kwargs);
623: static PyObject * THPVariable_cholesky(PyObject* self_, PyObject* args, PyObject* kwargs);
624: static PyObject * THPVariable__lu_with_info(PyObject* self_, PyObject* args, PyObject* kwargs);
625: static PyObject * THPVariable_lu_unpack(PyObject* self_, PyObject* args, PyObject* kwargs);
626: static PyObject * THPVariable_multinomial(PyObject* self_, PyObject* args, PyObject* kwargs);
627: static PyObject * THPVariable_i0(PyObject* self_, PyObject* args, PyObject* kwargs);
628: static PyObject * THPVariable_i0_(PyObject* self_, PyObject* args, PyObject* kwargs);
629: static PyObject * THPVariable_atan2(PyObject* self_, PyObject* args, PyObject* kwargs);
630: static PyObject * THPVariable_hypot(PyObject* self_, PyObject* args, PyObject* kwargs);
631: static PyObject * THPVariable_igamma(PyObject* self_, PyObject* args, PyObject* kwargs);
632: static PyObject * THPVariable_nextafter(PyObject* self_, PyObject* args, PyObject* kwargs);
633: static PyObject * THPVariable_fmin(PyObject* self_, PyObject* args, PyObject* kwargs);
634: static PyObject * THPVariable_sort(PyObject* self_, PyObject* args, PyObject* kwargs);
635: static PyObject * THPVariable_renorm(PyObject* self_, PyObject* args, PyObject* kwargs);
636: static PyObject * THPVariable__amp_foreach_non_finite_check_and_unscale_(PyObject* self_, PyObject* args, PyObject* kwargs);
637: static PyObject * THPVariable__amp_update_scale_(PyObject* self_, PyObject* args, PyObject* kwargs);
638: static PyObject * THPVariable__foreach_mul(PyObject* self_, PyObject* args, PyObject* kwargs);
639: static PyObject * THPVariable__foreach_mul_(PyObject* self_, PyObject* args, PyObject* kwargs);
640: static PyObject * THPVariable__foreach_maximum(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable__nested_tensor_from_mask_left_aligned`, `THPVariable__nested_from_padded_and_nested_example`, `THPVariable__nested_view_from_buffer`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__nested_tensor_from_mask_left_aligned`, `THPVariable__nested_from_padded_and_nested_example`, `THPVariable__nested_view_from_buffer` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 641-720

```cpp
641: static PyObject * THPVariable__foreach_maximum_(PyObject* self_, PyObject* args, PyObject* kwargs);
642: static PyObject * THPVariable__foreach_minimum(PyObject* self_, PyObject* args, PyObject* kwargs);
643: static PyObject * THPVariable__foreach_minimum_(PyObject* self_, PyObject* args, PyObject* kwargs);
644: static PyObject * THPVariable__foreach_addcdiv(PyObject* self_, PyObject* args, PyObject* kwargs);
645: static PyObject * THPVariable__foreach_addcdiv_(PyObject* self_, PyObject* args, PyObject* kwargs);
646: static PyObject * THPVariable__foreach_abs(PyObject* self_, PyObject* args, PyObject* kwargs);
647: static PyObject * THPVariable__foreach_abs_(PyObject* self_, PyObject* args, PyObject* kwargs);
648: static PyObject * THPVariable__foreach_erf(PyObject* self_, PyObject* args, PyObject* kwargs);
649: static PyObject * THPVariable__foreach_erf_(PyObject* self_, PyObject* args, PyObject* kwargs);
650: static PyObject * THPVariable__foreach_frac(PyObject* self_, PyObject* args, PyObject* kwargs);
651: static PyObject * THPVariable__foreach_frac_(PyObject* self_, PyObject* args, PyObject* kwargs);
652: static PyObject * THPVariable__foreach_lgamma(PyObject* self_, PyObject* args, PyObject* kwargs);
653: static PyObject * THPVariable__foreach_lgamma_(PyObject* self_, PyObject* args, PyObject* kwargs);
654: static PyObject * THPVariable__foreach_powsum(PyObject* self_, PyObject* args, PyObject* kwargs);
655: static PyObject * THPVariable__foreach_pow(PyObject* self_, PyObject* args, PyObject* kwargs);
656: static PyObject * THPVariable__foreach_pow_(PyObject* self_, PyObject* args, PyObject* kwargs);
657: static PyObject * THPVariable__foreach_reciprocal(PyObject* self_, PyObject* args, PyObject* kwargs);
658: static PyObject * THPVariable__foreach_reciprocal_(PyObject* self_, PyObject* args, PyObject* kwargs);
659: static PyObject * THPVariable__foreach_round(PyObject* self_, PyObject* args, PyObject* kwargs);
660: static PyObject * THPVariable__foreach_round_(PyObject* self_, PyObject* args, PyObject* kwargs);
661: static PyObject * THPVariable__foreach_sinh(PyObject* self_, PyObject* args, PyObject* kwargs);
662: static PyObject * THPVariable__foreach_sinh_(PyObject* self_, PyObject* args, PyObject* kwargs);
663: static PyObject * THPVariable__foreach_tan(PyObject* self_, PyObject* args, PyObject* kwargs);
664: static PyObject * THPVariable__foreach_tan_(PyObject* self_, PyObject* args, PyObject* kwargs);
665: static PyObject * THPVariable__foreach_copy_(PyObject* self_, PyObject* args, PyObject* kwargs);
666: static PyObject * THPVariable_bucketize(PyObject* self_, PyObject* args, PyObject* kwargs);
667: static PyObject * THPVariable_searchsorted(PyObject* self_, PyObject* args, PyObject* kwargs);
668: static PyObject * THPVariable__convert_indices_from_coo_to_csr(PyObject* self_, PyObject* args, PyObject* kwargs);
669: static PyObject * THPVariable_isfinite(PyObject* self_, PyObject* args, PyObject* kwargs);
670: static PyObject * THPVariable__remove_batch_dim(PyObject* self_, PyObject* args, PyObject* kwargs);
671: static PyObject * THPVariable__linalg_eigh(PyObject* self_, PyObject* args, PyObject* kwargs);
672: static PyObject * THPVariable_outer(PyObject* self_, PyObject* args, PyObject* kwargs);
673: static PyObject * THPVariable__test_serialization_subcmul(PyObject* self_, PyObject* args, PyObject* kwargs);
674: static PyObject * THPVariable__test_autograd_multiple_dispatch_view(PyObject* self_, PyObject* args, PyObject* kwargs);
675: static PyObject * THPVariable_view_as_real_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
676: static PyObject * THPVariable_permute_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
677: static PyObject * THPVariable_detach_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
678: static PyObject * THPVariable_split_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
679: static PyObject * THPVariable_unsqueeze_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
680: static PyObject * THPVariable__indices_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
681: static PyObject * THPVariable__values_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
682: static PyObject * THPVariable_unbind_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
683: static PyObject * THPVariable_alias_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
684: static PyObject * THPVariable__nested_tensor_softmax_with_shape(PyObject* self_, PyObject* args, PyObject* kwargs);
685: static PyObject * THPVariable__safe_softmax(PyObject* self_, PyObject* args, PyObject* kwargs);
686: static PyObject * THPVariable__native_multi_head_attention(PyObject* self_, PyObject* args, PyObject* kwargs);
687: static PyObject * THPVariable__scaled_dot_product_attention_math_for_mps(PyObject* self_, PyObject* args, PyObject* kwargs);
688: static PyObject * THPVariable__scaled_dot_product_flash_attention(PyObject* self_, PyObject* args, PyObject* kwargs);
689: static PyObject * THPVariable__scaled_dot_product_flash_attention_for_cpu(PyObject* self_, PyObject* args, PyObject* kwargs);
690: static PyObject * THPVariable__triton_scaled_dot_attention(PyObject* self_, PyObject* args, PyObject* kwargs);
691: static PyObject * THPVariable__fused_adamw_(PyObject* self_, PyObject* args, PyObject* kwargs);
692: 
693: static PyMethodDef torch_functions_shard[] = {
694:   {"_cast_Double", castPyCFunctionWithKeywords(THPVariable__cast_Double), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
695:   {"_cast_Int", castPyCFunctionWithKeywords(THPVariable__cast_Int), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
696:   {"align_tensors", castPyCFunctionWithKeywords(THPVariable_align_tensors), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
697:   {"_print", castPyCFunctionWithKeywords(THPVariable__print), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
698:   {"_make_dep_token", castPyCFunctionWithKeywords(THPVariable__make_dep_token), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
699:   {"_use_cudnn_rnn_flatten_weight", THPVariable__use_cudnn_rnn_flatten_weight, METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
700:   {"_cudnn_rnn", castPyCFunctionWithKeywords(THPVariable__cudnn_rnn), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
701:   {"_fused_dropout", castPyCFunctionWithKeywords(THPVariable__fused_dropout), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
702:   {"_sobol_engine_ff_", castPyCFunctionWithKeywords(THPVariable__sobol_engine_ff_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
703:   {"_sobol_engine_scramble_", castPyCFunctionWithKeywords(THPVariable__sobol_engine_scramble_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
704:   {"_shape_as_tensor", castPyCFunctionWithKeywords(THPVariable__shape_as_tensor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
705:   {"dropout", castPyCFunctionWithKeywords(THPVariable_dropout), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
706:   {"dropout_", castPyCFunctionWithKeywords(THPVariable_dropout_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
707:   {"_conj", castPyCFunctionWithKeywords(THPVariable__conj), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
708:   {"conj_physical", castPyCFunctionWithKeywords(THPVariable_conj_physical), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
709:   {"conj_physical_", castPyCFunctionWithKeywords(THPVariable_conj_physical_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
710:   {"avg_pool1d", castPyCFunctionWithKeywords(THPVariable_avg_pool1d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
711:   {"add", castPyCFunctionWithKeywords(THPVariable_add), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
712:   {"_add_relu", castPyCFunctionWithKeywords(THPVariable__add_relu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
713:   {"_add_relu_", castPyCFunctionWithKeywords(THPVariable__add_relu_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
714:   {"addr", castPyCFunctionWithKeywords(THPVariable_addr), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
715:   {"_test_functorch_fallback", castPyCFunctionWithKeywords(THPVariable__test_functorch_fallback), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
716:   {"all", castPyCFunctionWithKeywords(THPVariable_all), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
717:   {"allclose", castPyCFunctionWithKeywords(THPVariable_allclose), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
718:   {"argmax", castPyCFunctionWithKeywords(THPVariable_argmax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
719:   {"acosh", castPyCFunctionWithKeywords(THPVariable_acosh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
720:   {"acosh_", castPyCFunctionWithKeywords(THPVariable_acosh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_maximum_`, `THPVariable__foreach_minimum`, `THPVariable__foreach_minimum_`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_maximum_`, `THPVariable__foreach_minimum`, `THPVariable__foreach_minimum_` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 721-800

```cpp
721:   {"arccosh", castPyCFunctionWithKeywords(THPVariable_arccosh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
722:   {"arccosh_", castPyCFunctionWithKeywords(THPVariable_arccosh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
723:   {"arctanh", castPyCFunctionWithKeywords(THPVariable_arctanh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
724:   {"arctanh_", castPyCFunctionWithKeywords(THPVariable_arctanh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
725:   {"arcsin", castPyCFunctionWithKeywords(THPVariable_arcsin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
726:   {"arcsin_", castPyCFunctionWithKeywords(THPVariable_arcsin_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
727:   {"atan", castPyCFunctionWithKeywords(THPVariable_atan), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
728:   {"atan_", castPyCFunctionWithKeywords(THPVariable_atan_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
729:   {"baddbmm", castPyCFunctionWithKeywords(THPVariable_baddbmm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
730:   {"bartlett_window", castPyCFunctionWithKeywords(THPVariable_bartlett_window), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
731:   {"batch_norm", castPyCFunctionWithKeywords(THPVariable_batch_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
732:   {"copysign", castPyCFunctionWithKeywords(THPVariable_copysign), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
733:   {"_lazy_clone", castPyCFunctionWithKeywords(THPVariable__lazy_clone), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
734:   {"logical_not", castPyCFunctionWithKeywords(THPVariable_logical_not), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
735:   {"logical_xor", castPyCFunctionWithKeywords(THPVariable_logical_xor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
736:   {"bmm", castPyCFunctionWithKeywords(THPVariable_bmm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
737:   {"broadcast_tensors", castPyCFunctionWithKeywords(THPVariable_broadcast_tensors), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
738:   {"broadcast_to", castPyCFunctionWithKeywords(THPVariable_broadcast_to), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
739:   {"_sparse_broadcast_to", castPyCFunctionWithKeywords(THPVariable__sparse_broadcast_to), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
740:   {"cat", castPyCFunctionWithKeywords(THPVariable_cat), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
741:   {"chain_matmul", castPyCFunctionWithKeywords(THPVariable_chain_matmul), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
742:   {"chunk", castPyCFunctionWithKeywords(THPVariable_chunk), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
743:   {"tensor_split", castPyCFunctionWithKeywords(THPVariable_tensor_split), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
744:   {"clamp_min", castPyCFunctionWithKeywords(THPVariable_clamp_min), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
745:   {"clamp_min_", castPyCFunctionWithKeywords(THPVariable_clamp_min_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
746:   {"clip", castPyCFunctionWithKeywords(THPVariable_clip), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
747:   {"clip_", castPyCFunctionWithKeywords(THPVariable_clip_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
748:   {"complex", castPyCFunctionWithKeywords(THPVariable_complex), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
749:   {"polar", castPyCFunctionWithKeywords(THPVariable_polar), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
750:   {"_convolution", castPyCFunctionWithKeywords(THPVariable__convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
751:   {"_convolution_mode", castPyCFunctionWithKeywords(THPVariable__convolution_mode), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
752:   {"conv1d", castPyCFunctionWithKeywords(THPVariable_conv1d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
753:   {"conv3d", castPyCFunctionWithKeywords(THPVariable_conv3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
754:   {"cos", castPyCFunctionWithKeywords(THPVariable_cos), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
755:   {"cos_", castPyCFunctionWithKeywords(THPVariable_cos_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
756:   {"count_nonzero", castPyCFunctionWithKeywords(THPVariable_count_nonzero), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
757:   {"corrcoef", castPyCFunctionWithKeywords(THPVariable_corrcoef), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
758:   {"cudnn_convolution_transpose", castPyCFunctionWithKeywords(THPVariable_cudnn_convolution_transpose), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
759:   {"cudnn_convolution_relu", castPyCFunctionWithKeywords(THPVariable_cudnn_convolution_relu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
760:   {"ctc_loss", castPyCFunctionWithKeywords(THPVariable_ctc_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
761:   {"embedding_bag", castPyCFunctionWithKeywords(THPVariable_embedding_bag), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
762:   {"_embedding_bag", castPyCFunctionWithKeywords(THPVariable__embedding_bag), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
763:   {"empty", castPyCFunctionWithKeywords(THPVariable_empty), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
764:   {"empty_like", castPyCFunctionWithKeywords(THPVariable_empty_like), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
765:   {"erf", castPyCFunctionWithKeywords(THPVariable_erf), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
766:   {"erf_", castPyCFunctionWithKeywords(THPVariable_erf_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
767:   {"erfc", castPyCFunctionWithKeywords(THPVariable_erfc), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
768:   {"erfc_", castPyCFunctionWithKeywords(THPVariable_erfc_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
769:   {"exp", castPyCFunctionWithKeywords(THPVariable_exp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
770:   {"exp_", castPyCFunctionWithKeywords(THPVariable_exp_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
771:   {"unflatten", castPyCFunctionWithKeywords(THPVariable_unflatten), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
772:   {"floor", castPyCFunctionWithKeywords(THPVariable_floor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
773:   {"floor_", castPyCFunctionWithKeywords(THPVariable_floor_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
774:   {"floor_divide", castPyCFunctionWithKeywords(THPVariable_floor_divide), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
775:   {"frac", castPyCFunctionWithKeywords(THPVariable_frac), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
776:   {"frac_", castPyCFunctionWithKeywords(THPVariable_frac_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
777:   {"full_like", castPyCFunctionWithKeywords(THPVariable_full_like), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
778:   {"grid_sampler", castPyCFunctionWithKeywords(THPVariable_grid_sampler), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
779:   {"_grid_sampler_2d_cpu_fallback", castPyCFunctionWithKeywords(THPVariable__grid_sampler_2d_cpu_fallback), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
780:   {"grid_sampler_3d", castPyCFunctionWithKeywords(THPVariable_grid_sampler_3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
781:   {"_fft_c2r", castPyCFunctionWithKeywords(THPVariable__fft_c2r), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
782:   {"_cufft_get_plan_cache_max_size", castPyCFunctionWithKeywords(THPVariable__cufft_get_plan_cache_max_size), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
783:   {"_cufft_clear_plan_cache", castPyCFunctionWithKeywords(THPVariable__cufft_clear_plan_cache), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
784:   {"_unsafe_masked_index_put_accumulate", castPyCFunctionWithKeywords(THPVariable__unsafe_masked_index_put_accumulate), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
785:   {"_index_put_impl_", castPyCFunctionWithKeywords(THPVariable__index_put_impl_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
786:   {"isin", castPyCFunctionWithKeywords(THPVariable_isin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
787:   {"isnan", castPyCFunctionWithKeywords(THPVariable_isnan), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
788:   {"is_conj", castPyCFunctionWithKeywords(THPVariable_is_conj), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
789:   {"is_inference", castPyCFunctionWithKeywords(THPVariable_is_inference), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
790:   {"layer_norm", castPyCFunctionWithKeywords(THPVariable_layer_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
791:   {"_fused_rms_norm", castPyCFunctionWithKeywords(THPVariable__fused_rms_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
792:   {"nan_to_num", castPyCFunctionWithKeywords(THPVariable_nan_to_num), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
793:   {"nan_to_num_", castPyCFunctionWithKeywords(THPVariable_nan_to_num_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
794:   {"mkldnn_linear_backward_weights", castPyCFunctionWithKeywords(THPVariable_mkldnn_linear_backward_weights), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
795:   {"_cslt_sparse_mm", castPyCFunctionWithKeywords(THPVariable__cslt_sparse_mm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
796:   {"_sparse_semi_structured_mm", castPyCFunctionWithKeywords(THPVariable__sparse_semi_structured_mm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
797:   {"_sparse_semi_structured_addmm", castPyCFunctionWithKeywords(THPVariable__sparse_semi_structured_addmm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
798:   {"_mixed_dtypes_linear", castPyCFunctionWithKeywords(THPVariable__mixed_dtypes_linear), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
799:   {"fbgemm_pack_gemm_matrix_fp16", castPyCFunctionWithKeywords(THPVariable_fbgemm_pack_gemm_matrix_fp16), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
800:   {"_wrapped_quantized_linear_prepacked", castPyCFunctionWithKeywords(THPVariable__wrapped_quantized_linear_prepacked), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 801-880

```cpp
801:   {"fbgemm_linear_fp16_weight_fp32_activation", castPyCFunctionWithKeywords(THPVariable_fbgemm_linear_fp16_weight_fp32_activation), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
802:   {"linspace", castPyCFunctionWithKeywords(THPVariable_linspace), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
803:   {"log", castPyCFunctionWithKeywords(THPVariable_log), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
804:   {"log_", castPyCFunctionWithKeywords(THPVariable_log_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
805:   {"log1p", castPyCFunctionWithKeywords(THPVariable_log1p), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
806:   {"log1p_", castPyCFunctionWithKeywords(THPVariable_log1p_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
807:   {"logaddexp", castPyCFunctionWithKeywords(THPVariable_logaddexp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
808:   {"xlogy", castPyCFunctionWithKeywords(THPVariable_xlogy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
809:   {"xlogy_", castPyCFunctionWithKeywords(THPVariable_xlogy_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
810:   {"logsumexp", castPyCFunctionWithKeywords(THPVariable_logsumexp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
811:   {"matrix_power", castPyCFunctionWithKeywords(THPVariable_matrix_power), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
812:   {"_aminmax", castPyCFunctionWithKeywords(THPVariable__aminmax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
813:   {"max", castPyCFunctionWithKeywords(THPVariable_max), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
814:   {"quantized_max_pool1d", castPyCFunctionWithKeywords(THPVariable_quantized_max_pool1d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
815:   {"quantized_max_pool2d", castPyCFunctionWithKeywords(THPVariable_quantized_max_pool2d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
816:   {"mean", castPyCFunctionWithKeywords(THPVariable_mean), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
817:   {"nanmean", castPyCFunctionWithKeywords(THPVariable_nanmean), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
818:   {"nanmedian", castPyCFunctionWithKeywords(THPVariable_nanmedian), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
819:   {"min", castPyCFunctionWithKeywords(THPVariable_min), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
820:   {"amin", castPyCFunctionWithKeywords(THPVariable_amin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
821:   {"mkldnn_convolution", castPyCFunctionWithKeywords(THPVariable_mkldnn_convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
822:   {"miopen_batch_norm", castPyCFunctionWithKeywords(THPVariable_miopen_batch_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
823:   {"miopen_depthwise_convolution", castPyCFunctionWithKeywords(THPVariable_miopen_depthwise_convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
824:   {"_weight_int8pack_mm", castPyCFunctionWithKeywords(THPVariable__weight_int8pack_mm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
825:   {"mv", castPyCFunctionWithKeywords(THPVariable_mv), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
826:   {"narrow", castPyCFunctionWithKeywords(THPVariable_narrow), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
827:   {"batch_norm_stats", castPyCFunctionWithKeywords(THPVariable_batch_norm_stats), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
828:   {"batch_norm_update_stats", castPyCFunctionWithKeywords(THPVariable_batch_norm_update_stats), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
829:   {"_nnpack_available", THPVariable__nnpack_available, METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
830:   {"_nnpack_spatial_convolution", castPyCFunctionWithKeywords(THPVariable__nnpack_spatial_convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
831:   {"_euclidean_dist", castPyCFunctionWithKeywords(THPVariable__euclidean_dist), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
832:   {"permute", castPyCFunctionWithKeywords(THPVariable_permute), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
833:   {"pixel_shuffle", castPyCFunctionWithKeywords(THPVariable_pixel_shuffle), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
834:   {"native_channel_shuffle", castPyCFunctionWithKeywords(THPVariable_native_channel_shuffle), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
835:   {"_pin_memory", castPyCFunctionWithKeywords(THPVariable__pin_memory), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
836:   {"rad2deg", castPyCFunctionWithKeywords(THPVariable_rad2deg), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
837:   {"rad2deg_", castPyCFunctionWithKeywords(THPVariable_rad2deg_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
838:   {"rand_like", castPyCFunctionWithKeywords(THPVariable_rand_like), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
839:   {"randint", castPyCFunctionWithKeywords(THPVariable_randint), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
840:   {"randint_like", castPyCFunctionWithKeywords(THPVariable_randint_like), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
841:   {"randn", castPyCFunctionWithKeywords(THPVariable_randn), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
842:   {"randperm", castPyCFunctionWithKeywords(THPVariable_randperm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
843:   {"reciprocal", castPyCFunctionWithKeywords(THPVariable_reciprocal), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
844:   {"reciprocal_", castPyCFunctionWithKeywords(THPVariable_reciprocal_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
845:   {"negative", castPyCFunctionWithKeywords(THPVariable_negative), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
846:   {"negative_", castPyCFunctionWithKeywords(THPVariable_negative_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
847:   {"reshape", castPyCFunctionWithKeywords(THPVariable_reshape), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
848:   {"_mkldnn_reshape", castPyCFunctionWithKeywords(THPVariable__mkldnn_reshape), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
849:   {"relu", castPyCFunctionWithKeywords(THPVariable_relu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
850:   {"relu_", castPyCFunctionWithKeywords(THPVariable_relu_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
851:   {"prelu", castPyCFunctionWithKeywords(THPVariable_prelu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
852:   {"select", castPyCFunctionWithKeywords(THPVariable_select), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
853:   {"selu", castPyCFunctionWithKeywords(THPVariable_selu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
854:   {"selu_", castPyCFunctionWithKeywords(THPVariable_selu_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
855:   {"logit", castPyCFunctionWithKeywords(THPVariable_logit), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
856:   {"logit_", castPyCFunctionWithKeywords(THPVariable_logit_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
857:   {"sin", castPyCFunctionWithKeywords(THPVariable_sin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
858:   {"sin_", castPyCFunctionWithKeywords(THPVariable_sin_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
859:   {"slice_inverse", castPyCFunctionWithKeywords(THPVariable_slice_inverse), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
860:   {"softmax", castPyCFunctionWithKeywords(THPVariable_softmax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
861:   {"unsafe_split_with_sizes", castPyCFunctionWithKeywords(THPVariable_unsafe_split_with_sizes), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
862:   {"split_with_sizes", castPyCFunctionWithKeywords(THPVariable_split_with_sizes), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
863:   {"vsplit", castPyCFunctionWithKeywords(THPVariable_vsplit), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
864:   {"_chunk_cat", castPyCFunctionWithKeywords(THPVariable__chunk_cat), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
865:   {"hstack", castPyCFunctionWithKeywords(THPVariable_hstack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
866:   {"vstack", castPyCFunctionWithKeywords(THPVariable_vstack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
867:   {"dstack", castPyCFunctionWithKeywords(THPVariable_dstack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
868:   {"sum", castPyCFunctionWithKeywords(THPVariable_sum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
869:   {"tan", castPyCFunctionWithKeywords(THPVariable_tan), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
870:   {"tan_", castPyCFunctionWithKeywords(THPVariable_tan_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
871:   {"trapz", castPyCFunctionWithKeywords(THPVariable_trapz), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
872:   {"_nested_tensor_from_mask_left_aligned", castPyCFunctionWithKeywords(THPVariable__nested_tensor_from_mask_left_aligned), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
873:   {"_nested_from_padded_and_nested_example", castPyCFunctionWithKeywords(THPVariable__nested_from_padded_and_nested_example), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
874:   {"_nested_view_from_buffer", castPyCFunctionWithKeywords(THPVariable__nested_view_from_buffer), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
875:   {"_nested_get_values_copy", castPyCFunctionWithKeywords(THPVariable__nested_get_values_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
876:   {"_nested_get_lengths", castPyCFunctionWithKeywords(THPVariable__nested_get_lengths), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
877:   {"trunc", castPyCFunctionWithKeywords(THPVariable_trunc), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
878:   {"trunc_", castPyCFunctionWithKeywords(THPVariable_trunc_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
879:   {"unique_dim", castPyCFunctionWithKeywords(THPVariable_unique_dim), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
880:   {"unique_consecutive", castPyCFunctionWithKeywords(THPVariable_unique_consecutive), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 881-960

```cpp
881:   {"where", castPyCFunctionWithKeywords(THPVariable_where), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
882:   {"_standard_gamma", castPyCFunctionWithKeywords(THPVariable__standard_gamma), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
883:   {"_philox_key_fold_in", castPyCFunctionWithKeywords(THPVariable__philox_key_fold_in), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
884:   {"_philox_uniform_", castPyCFunctionWithKeywords(THPVariable__philox_uniform_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
885:   {"_dirichlet_grad", castPyCFunctionWithKeywords(THPVariable__dirichlet_grad), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
886:   {"binomial", castPyCFunctionWithKeywords(THPVariable_binomial), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
887:   {"_sparse_csr_sum", castPyCFunctionWithKeywords(THPVariable__sparse_csr_sum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
888:   {"_sparse_log_softmax_backward_data", castPyCFunctionWithKeywords(THPVariable__sparse_log_softmax_backward_data), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
889:   {"nuclear_norm", castPyCFunctionWithKeywords(THPVariable_nuclear_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
890:   {"clone", castPyCFunctionWithKeywords(THPVariable_clone), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
891:   {"heaviside", castPyCFunctionWithKeywords(THPVariable_heaviside), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
892:   {"_scaled_mm", castPyCFunctionWithKeywords(THPVariable__scaled_mm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
893:   {"_scaled_grouped_mm_v2", castPyCFunctionWithKeywords(THPVariable__scaled_grouped_mm_v2), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
894:   {"_validate_sparse_compressed_tensor_args", castPyCFunctionWithKeywords(THPVariable__validate_sparse_compressed_tensor_args), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
895:   {"_validate_sparse_csr_tensor_args", castPyCFunctionWithKeywords(THPVariable__validate_sparse_csr_tensor_args), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
896:   {"_to_cpu", castPyCFunctionWithKeywords(THPVariable__to_cpu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
897:   {"hspmm", castPyCFunctionWithKeywords(THPVariable_hspmm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
898:   {"_to_sparse_semi_structured", castPyCFunctionWithKeywords(THPVariable__to_sparse_semi_structured), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
899:   {"q_per_channel_scales", castPyCFunctionWithKeywords(THPVariable_q_per_channel_scales), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
900:   {"q_per_channel_zero_points", castPyCFunctionWithKeywords(THPVariable_q_per_channel_zero_points), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
901:   {"q_per_channel_axis", castPyCFunctionWithKeywords(THPVariable_q_per_channel_axis), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
902:   {"_fake_quantize_per_tensor_affine_cachemask_tensor_qparams", castPyCFunctionWithKeywords(THPVariable__fake_quantize_per_tensor_affine_cachemask_tensor_qparams), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
903:   {"_fake_quantize_learnable_per_tensor_affine", castPyCFunctionWithKeywords(THPVariable__fake_quantize_learnable_per_tensor_affine), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
904:   {"fused_moving_avg_obs_fake_quant", castPyCFunctionWithKeywords(THPVariable_fused_moving_avg_obs_fake_quant), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
905:   {"_saturate_weight_to_fp16", castPyCFunctionWithKeywords(THPVariable__saturate_weight_to_fp16), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
906:   {"choose_qparams_optimized", castPyCFunctionWithKeywords(THPVariable_choose_qparams_optimized), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
907:   {"combinations", castPyCFunctionWithKeywords(THPVariable_combinations), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
908:   {"result_type", castPyCFunctionWithKeywords(THPVariable_result_type), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
909:   {"can_cast", castPyCFunctionWithKeywords(THPVariable_can_cast), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
910:   {"_lstm_mps", castPyCFunctionWithKeywords(THPVariable__lstm_mps), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
911:   {"lstm", castPyCFunctionWithKeywords(THPVariable_lstm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
912:   {"lstm_cell", castPyCFunctionWithKeywords(THPVariable_lstm_cell), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
913:   {"quantized_rnn_relu_cell", castPyCFunctionWithKeywords(THPVariable_quantized_rnn_relu_cell), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
914:   {"quantized_rnn_tanh_cell", castPyCFunctionWithKeywords(THPVariable_quantized_rnn_tanh_cell), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
915:   {"index_add", castPyCFunctionWithKeywords(THPVariable_index_add), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
916:   {"index_reduce", castPyCFunctionWithKeywords(THPVariable_index_reduce), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
917:   {"scatter_add", castPyCFunctionWithKeywords(THPVariable_scatter_add), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
918:   {"scatter_reduce", castPyCFunctionWithKeywords(THPVariable_scatter_reduce), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
919:   {"bitwise_and", castPyCFunctionWithKeywords(THPVariable_bitwise_and), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
920:   {"bitwise_xor", castPyCFunctionWithKeywords(THPVariable_bitwise_xor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
921:   {"bitwise_left_shift", castPyCFunctionWithKeywords(THPVariable_bitwise_left_shift), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
922:   {"__rshift__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___rshift__>), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
923:   {"addbmm", castPyCFunctionWithKeywords(THPVariable_addbmm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
924:   {"diag", castPyCFunctionWithKeywords(THPVariable_diag), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
925:   {"cross", castPyCFunctionWithKeywords(THPVariable_cross), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
926:   {"triu_indices", castPyCFunctionWithKeywords(THPVariable_triu_indices), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
927:   {"trace", castPyCFunctionWithKeywords(THPVariable_trace), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
928:   {"ne", castPyCFunctionWithKeywords(THPVariable_ne), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
929:   {"eq", castPyCFunctionWithKeywords(THPVariable_eq), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
930:   {"greater", castPyCFunctionWithKeywords(THPVariable_greater), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
931:   {"take", castPyCFunctionWithKeywords(THPVariable_take), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
932:   {"_linalg_check_errors", castPyCFunctionWithKeywords(THPVariable__linalg_check_errors), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
933:   {"svd", castPyCFunctionWithKeywords(THPVariable_svd), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
934:   {"cholesky", castPyCFunctionWithKeywords(THPVariable_cholesky), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
935:   {"_lu_with_info", castPyCFunctionWithKeywords(THPVariable__lu_with_info), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
936:   {"lu_unpack", castPyCFunctionWithKeywords(THPVariable_lu_unpack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
937:   {"multinomial", castPyCFunctionWithKeywords(THPVariable_multinomial), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
938:   {"i0", castPyCFunctionWithKeywords(THPVariable_i0), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
939:   {"i0_", castPyCFunctionWithKeywords(THPVariable_i0_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
940:   {"atan2", castPyCFunctionWithKeywords(THPVariable_atan2), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
941:   {"hypot", castPyCFunctionWithKeywords(THPVariable_hypot), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
942:   {"igamma", castPyCFunctionWithKeywords(THPVariable_igamma), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
943:   {"nextafter", castPyCFunctionWithKeywords(THPVariable_nextafter), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
944:   {"fmin", castPyCFunctionWithKeywords(THPVariable_fmin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
945:   {"sort", castPyCFunctionWithKeywords(THPVariable_sort), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
946:   {"renorm", castPyCFunctionWithKeywords(THPVariable_renorm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
947:   {"_amp_foreach_non_finite_check_and_unscale_", castPyCFunctionWithKeywords(THPVariable__amp_foreach_non_finite_check_and_unscale_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
948:   {"_amp_update_scale_", castPyCFunctionWithKeywords(THPVariable__amp_update_scale_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
949:   {"_foreach_mul", castPyCFunctionWithKeywords(THPVariable__foreach_mul), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
950:   {"_foreach_mul_", castPyCFunctionWithKeywords(THPVariable__foreach_mul_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
951:   {"_foreach_maximum", castPyCFunctionWithKeywords(THPVariable__foreach_maximum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
952:   {"_foreach_maximum_", castPyCFunctionWithKeywords(THPVariable__foreach_maximum_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
953:   {"_foreach_minimum", castPyCFunctionWithKeywords(THPVariable__foreach_minimum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
954:   {"_foreach_minimum_", castPyCFunctionWithKeywords(THPVariable__foreach_minimum_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
955:   {"_foreach_addcdiv", castPyCFunctionWithKeywords(THPVariable__foreach_addcdiv), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
956:   {"_foreach_addcdiv_", castPyCFunctionWithKeywords(THPVariable__foreach_addcdiv_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
957:   {"_foreach_abs", castPyCFunctionWithKeywords(THPVariable__foreach_abs), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
958:   {"_foreach_abs_", castPyCFunctionWithKeywords(THPVariable__foreach_abs_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
959:   {"_foreach_erf", castPyCFunctionWithKeywords(THPVariable__foreach_erf), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
960:   {"_foreach_erf_", castPyCFunctionWithKeywords(THPVariable__foreach_erf_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 961-1040

```cpp
 961:   {"_foreach_frac", castPyCFunctionWithKeywords(THPVariable__foreach_frac), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 962:   {"_foreach_frac_", castPyCFunctionWithKeywords(THPVariable__foreach_frac_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 963:   {"_foreach_lgamma", castPyCFunctionWithKeywords(THPVariable__foreach_lgamma), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 964:   {"_foreach_lgamma_", castPyCFunctionWithKeywords(THPVariable__foreach_lgamma_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 965:   {"_foreach_powsum", castPyCFunctionWithKeywords(THPVariable__foreach_powsum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 966:   {"_foreach_pow", castPyCFunctionWithKeywords(THPVariable__foreach_pow), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 967:   {"_foreach_pow_", castPyCFunctionWithKeywords(THPVariable__foreach_pow_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 968:   {"_foreach_reciprocal", castPyCFunctionWithKeywords(THPVariable__foreach_reciprocal), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 969:   {"_foreach_reciprocal_", castPyCFunctionWithKeywords(THPVariable__foreach_reciprocal_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 970:   {"_foreach_round", castPyCFunctionWithKeywords(THPVariable__foreach_round), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 971:   {"_foreach_round_", castPyCFunctionWithKeywords(THPVariable__foreach_round_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 972:   {"_foreach_sinh", castPyCFunctionWithKeywords(THPVariable__foreach_sinh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 973:   {"_foreach_sinh_", castPyCFunctionWithKeywords(THPVariable__foreach_sinh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 974:   {"_foreach_tan", castPyCFunctionWithKeywords(THPVariable__foreach_tan), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 975:   {"_foreach_tan_", castPyCFunctionWithKeywords(THPVariable__foreach_tan_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 976:   {"_foreach_copy_", castPyCFunctionWithKeywords(THPVariable__foreach_copy_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 977:   {"bucketize", castPyCFunctionWithKeywords(THPVariable_bucketize), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 978:   {"searchsorted", castPyCFunctionWithKeywords(THPVariable_searchsorted), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 979:   {"_convert_indices_from_coo_to_csr", castPyCFunctionWithKeywords(THPVariable__convert_indices_from_coo_to_csr), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 980:   {"isfinite", castPyCFunctionWithKeywords(THPVariable_isfinite), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 981:   {"_remove_batch_dim", castPyCFunctionWithKeywords(THPVariable__remove_batch_dim), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 982:   {"_linalg_eigh", castPyCFunctionWithKeywords(THPVariable__linalg_eigh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 983:   {"outer", castPyCFunctionWithKeywords(THPVariable_outer), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 984:   {"_test_serialization_subcmul", castPyCFunctionWithKeywords(THPVariable__test_serialization_subcmul), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 985:   {"_test_autograd_multiple_dispatch_view", castPyCFunctionWithKeywords(THPVariable__test_autograd_multiple_dispatch_view), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 986:   {"view_as_real_copy", castPyCFunctionWithKeywords(THPVariable_view_as_real_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 987:   {"permute_copy", castPyCFunctionWithKeywords(THPVariable_permute_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 988:   {"detach_copy", castPyCFunctionWithKeywords(THPVariable_detach_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 989:   {"split_copy", castPyCFunctionWithKeywords(THPVariable_split_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 990:   {"unsqueeze_copy", castPyCFunctionWithKeywords(THPVariable_unsqueeze_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 991:   {"_indices_copy", castPyCFunctionWithKeywords(THPVariable__indices_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 992:   {"_values_copy", castPyCFunctionWithKeywords(THPVariable__values_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 993:   {"unbind_copy", castPyCFunctionWithKeywords(THPVariable_unbind_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 994:   {"alias_copy", castPyCFunctionWithKeywords(THPVariable_alias_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 995:   {"_nested_tensor_softmax_with_shape", castPyCFunctionWithKeywords(THPVariable__nested_tensor_softmax_with_shape), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 996:   {"_safe_softmax", castPyCFunctionWithKeywords(THPVariable__safe_softmax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 997:   {"_native_multi_head_attention", castPyCFunctionWithKeywords(THPVariable__native_multi_head_attention), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 998:   {"_scaled_dot_product_attention_math_for_mps", castPyCFunctionWithKeywords(THPVariable__scaled_dot_product_attention_math_for_mps), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 999:   {"_scaled_dot_product_flash_attention", castPyCFunctionWithKeywords(THPVariable__scaled_dot_product_flash_attention), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1000:   {"_scaled_dot_product_flash_attention_for_cpu", castPyCFunctionWithKeywords(THPVariable__scaled_dot_product_flash_attention_for_cpu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1001:   {"_triton_scaled_dot_attention", castPyCFunctionWithKeywords(THPVariable__triton_scaled_dot_attention), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1002:   {"_fused_adamw_", castPyCFunctionWithKeywords(THPVariable__fused_adamw_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1003: };
1004: 
1005: void gatherTorchFunctions_2(std::vector<PyMethodDef> &torch_functions) {
1006:   constexpr size_t num_functions = sizeof(torch_functions_shard) / sizeof(torch_functions_shard[0]);
1007:   torch_functions.insert(
1008:     torch_functions.end(),
1009:     torch_functions_shard,
1010:     torch_functions_shard + num_functions);
1011: }
1012: 
1013: // generated methods start here
1014: 
1015: // _cast_Double
1016: static PyObject * THPVariable__cast_Double(PyObject* self_, PyObject* args, PyObject* kwargs)
1017: {
1018:   HANDLE_TH_ERRORS
1019:   static PythonArgParser parser({
1020:     "_cast_Double(Tensor input, bool non_blocking=False)",
1021:   }, /*traceable=*/true);
1022: 
1023:   ParsedArgs<2> parsed_args;
1024:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1025:   if(_r.has_torch_function()) {
1026:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1027:   }
1028:   // aten::_cast_Double(Tensor self, bool non_blocking=False) -> Tensor
1029: 
1030:   auto dispatch__cast_Double = [](const at::Tensor & self, bool non_blocking) -> at::Tensor {
1031:     pybind11::gil_scoped_release no_gil;
1032:     return at::_cast_Double(self, non_blocking);
1033:   };
1034:   return wrap(dispatch__cast_Double(_r.tensor(0), _r.toBool(1)));
1035:   Py_RETURN_NONE;
1036:   END_HANDLE_TH_ERRORS
1037: }
1038: 
1039: // _cast_Int
1040: static PyObject * THPVariable__cast_Int(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`, `gatherTorchFunctions_2`, `THPVariable__cast_Double`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords`, `gatherTorchFunctions_2`, `THPVariable__cast_Double` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1041-1120

```cpp
1041: {
1042:   HANDLE_TH_ERRORS
1043:   static PythonArgParser parser({
1044:     "_cast_Int(Tensor input, bool non_blocking=False)",
1045:   }, /*traceable=*/true);
1046: 
1047:   ParsedArgs<2> parsed_args;
1048:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1049:   if(_r.has_torch_function()) {
1050:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1051:   }
1052:   // aten::_cast_Int(Tensor self, bool non_blocking=False) -> Tensor
1053: 
1054:   auto dispatch__cast_Int = [](const at::Tensor & self, bool non_blocking) -> at::Tensor {
1055:     pybind11::gil_scoped_release no_gil;
1056:     return at::_cast_Int(self, non_blocking);
1057:   };
1058:   return wrap(dispatch__cast_Int(_r.tensor(0), _r.toBool(1)));
1059:   Py_RETURN_NONE;
1060:   END_HANDLE_TH_ERRORS
1061: }
1062: 
1063: // align_tensors
1064: static PyObject * THPVariable_align_tensors(PyObject* self_, PyObject* args, PyObject* kwargs)
1065: {
1066:   HANDLE_TH_ERRORS
1067:   static PythonArgParser parser({
1068:     "align_tensors(TensorList tensors)",
1069:   }, /*traceable=*/true);
1070: 
1071:   ParsedArgs<1> parsed_args;
1072:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1073:   if(_r.has_torch_function()) {
1074:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1075:   }
1076:   // aten::align_tensors(Tensor[] tensors) -> Tensor[]
1077: 
1078:   auto dispatch_align_tensors = [](at::TensorList tensors) -> ::std::vector<at::Tensor> {
1079:     pybind11::gil_scoped_release no_gil;
1080:     return at::align_tensors(tensors);
1081:   };
1082:   return wrap(dispatch_align_tensors(_r.tensorlist(0)));
1083:   Py_RETURN_NONE;
1084:   END_HANDLE_TH_ERRORS
1085: }
1086: 
1087: // _print
1088: static PyObject * THPVariable__print(PyObject* self_, PyObject* args, PyObject* kwargs)
1089: {
1090:   HANDLE_TH_ERRORS
1091:   static PythonArgParser parser({
1092:     "_print(c10::string_view s)",
1093:   }, /*traceable=*/false);
1094: 
1095:   ParsedArgs<1> parsed_args;
1096:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1097:   if(_r.has_torch_function()) {
1098:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1099:   }
1100:   // aten::_print(str s) -> ()
1101: 
1102:   auto dispatch__print = [](c10::string_view s) -> void {
1103:     pybind11::gil_scoped_release no_gil;
1104:     at::_print(s);
1105:   };
1106:   dispatch__print(_r.stringView(0));
1107:   Py_RETURN_NONE;
1108:   Py_RETURN_NONE;
1109:   END_HANDLE_TH_ERRORS
1110: }
1111: 
1112: // _make_dep_token
1113: static PyObject * THPVariable__make_dep_token(PyObject* self_, PyObject* args, PyObject* kwargs)
1114: {
1115:   HANDLE_TH_ERRORS
1116:   static PythonArgParser parser({
1117:     "_make_dep_token(*, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
1118:   }, /*traceable=*/true);
1119: 
1120:   ParsedArgs<6> parsed_args;
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_cast_Int`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_cast_Int` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1121-1200

```cpp
1121:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1122:   if(_r.has_torch_function()) {
1123:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1124:   }
1125:   // aten::_make_dep_token(*, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
1126:   const auto options = TensorOptions()
1127:       .dtype(_r.scalartypeOptional(1))
1128:       .device(_r.deviceWithDefault(3, torch::tensors::get_default_device()))
1129:       .layout(_r.layoutOptional(2))
1130:       .requires_grad(_r.toBool(5))
1131:       .pinned_memory(_r.toBool(4));
1132:   torch::utils::maybe_initialize_device(options);
1133: 
1134:   auto dispatch__make_dep_token = [](at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
1135:     pybind11::gil_scoped_release no_gil;
1136:     return torch::_make_dep_token(options, memory_format);
1137:   };
1138:   return wrap(dispatch__make_dep_token(options, _r.memoryformatOptional(0)));
1139:   Py_RETURN_NONE;
1140:   END_HANDLE_TH_ERRORS
1141: }
1142: 
1143: // _use_cudnn_rnn_flatten_weight
1144: static PyObject * THPVariable__use_cudnn_rnn_flatten_weight(PyObject* self_, PyObject* args)
1145: {
1146:   HANDLE_TH_ERRORS
1147: 
1148:   // aten::_use_cudnn_rnn_flatten_weight() -> bool
1149: 
1150:   auto dispatch__use_cudnn_rnn_flatten_weight = []() -> bool {
1151:     pybind11::gil_scoped_release no_gil;
1152:     return at::_use_cudnn_rnn_flatten_weight();
1153:   };
1154:   return wrap(dispatch__use_cudnn_rnn_flatten_weight());
1155:   END_HANDLE_TH_ERRORS
1156: }
1157: 
1158: // _cudnn_rnn
1159: static PyObject * THPVariable__cudnn_rnn(PyObject* self_, PyObject* args, PyObject* kwargs)
1160: {
1161:   HANDLE_TH_ERRORS
1162:   static PythonArgParser parser({
1163:     "_cudnn_rnn(Tensor input, TensorList weight, int64_t weight_stride0, Tensor? weight_buf, Tensor hx, Tensor? cx, int64_t mode, SymInt hidden_size, SymInt proj_size, int64_t num_layers, bool batch_first, double dropout, bool train, bool bidirectional, SymIntArrayRef batch_sizes, Tensor? dropout_state)",
1164:   }, /*traceable=*/true);
1165: 
1166:   ParsedArgs<16> parsed_args;
1167:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1168:   if(_r.has_torch_function()) {
1169:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1170:   }
1171:   // aten::_cudnn_rnn(Tensor input, Tensor[] weight, int weight_stride0, Tensor? weight_buf, Tensor hx, Tensor? cx, int mode, SymInt hidden_size, SymInt proj_size, int num_layers, bool batch_first, float dropout, bool train, bool bidirectional, SymInt[] batch_sizes, Tensor? dropout_state) -> (Tensor, Tensor, Tensor, Tensor, Tensor)
1172: 
1173:   auto dispatch__cudnn_rnn = [](const at::Tensor & input, at::TensorList weight, int64_t weight_stride0, const ::std::optional<at::Tensor> & weight_buf, const at::Tensor & hx, const ::std::optional<at::Tensor> & cx, int64_t mode, c10::SymInt hidden_size, c10::SymInt proj_size, int64_t num_layers, bool batch_first, double dropout, bool train, bool bidirectional, c10::SymIntArrayRef batch_sizes, const ::std::optional<at::Tensor> & dropout_state) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
1174:     pybind11::gil_scoped_release no_gil;
1175:     return at::_cudnn_rnn_symint(input, weight, weight_stride0, weight_buf, hx, cx, mode, hidden_size, proj_size, num_layers, batch_first, dropout, train, bidirectional, batch_sizes, dropout_state);
1176:   };
1177:   return wrap(dispatch__cudnn_rnn(_r.tensor(0), _r.tensorlist(1), _r.toInt64(2), _r.optionalTensor(3), _r.tensor(4), _r.optionalTensor(5), _r.toInt64(6), _r.toSymInt(7), _r.toSymInt(8), _r.toInt64(9), _r.toBool(10), _r.toDouble(11), _r.toBool(12), _r.toBool(13), _r.symintlist(14), _r.optionalTensor(15)));
1178:   Py_RETURN_NONE;
1179:   END_HANDLE_TH_ERRORS
1180: }
1181: 
1182: // _fused_dropout
1183: static PyObject * THPVariable__fused_dropout(PyObject* self_, PyObject* args, PyObject* kwargs)
1184: {
1185:   HANDLE_TH_ERRORS
1186:   static PythonArgParser parser({
1187:     "_fused_dropout(Tensor input, double p, Generator? generator=None)",
1188:   }, /*traceable=*/true);
1189: 
1190:   ParsedArgs<3> parsed_args;
1191:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1192:   if(_r.has_torch_function()) {
1193:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1194:   }
1195:   // aten::_fused_dropout(Tensor self, float p, Generator? generator=None) -> (Tensor, Tensor)
1196: 
1197:   auto dispatch__fused_dropout = [](const at::Tensor & self, double p, ::std::optional<at::Generator> generator) -> ::std::tuple<at::Tensor,at::Tensor> {
1198:     pybind11::gil_scoped_release no_gil;
1199:     return at::_fused_dropout(self, p, generator);
1200:   };
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_make_dep_token`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_make_dep_token`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1201-1280

```cpp
1201:   return wrap(dispatch__fused_dropout(_r.tensor(0), _r.toDouble(1), _r.generator(2)));
1202:   Py_RETURN_NONE;
1203:   END_HANDLE_TH_ERRORS
1204: }
1205: 
1206: // _sobol_engine_ff_
1207: static PyObject * THPVariable__sobol_engine_ff_(PyObject* self_, PyObject* args, PyObject* kwargs)
1208: {
1209:   HANDLE_TH_ERRORS
1210:   static PythonArgParser parser({
1211:     "_sobol_engine_ff_(Tensor input, int64_t n, Tensor sobolstate, int64_t dimension, int64_t num_generated)",
1212:   }, /*traceable=*/true);
1213: 
1214:   ParsedArgs<5> parsed_args;
1215:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1216:   if(_r.has_torch_function()) {
1217:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1218:   }
1219:   // aten::_sobol_engine_ff_(Tensor(a!) self, int n, Tensor sobolstate, int dimension, int num_generated) -> Tensor(a!)
1220: 
1221:   auto dispatch__sobol_engine_ff_ = [](at::Tensor self, int64_t n, const at::Tensor & sobolstate, int64_t dimension, int64_t num_generated) -> at::Tensor {
1222:     pybind11::gil_scoped_release no_gil;
1223:     return at::_sobol_engine_ff_(self, n, sobolstate, dimension, num_generated);
1224:   };
1225:   return wrap(dispatch__sobol_engine_ff_(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.toInt64(3), _r.toInt64(4)));
1226:   Py_RETURN_NONE;
1227:   END_HANDLE_TH_ERRORS
1228: }
1229: 
1230: // _sobol_engine_scramble_
1231: static PyObject * THPVariable__sobol_engine_scramble_(PyObject* self_, PyObject* args, PyObject* kwargs)
1232: {
1233:   HANDLE_TH_ERRORS
1234:   static PythonArgParser parser({
1235:     "_sobol_engine_scramble_(Tensor input, Tensor ltm, int64_t dimension)",
1236:   }, /*traceable=*/true);
1237: 
1238:   ParsedArgs<3> parsed_args;
1239:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1240:   if(_r.has_torch_function()) {
1241:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1242:   }
1243:   // aten::_sobol_engine_scramble_(Tensor(a!) self, Tensor ltm, int dimension) -> Tensor(a!)
1244: 
1245:   auto dispatch__sobol_engine_scramble_ = [](at::Tensor self, const at::Tensor & ltm, int64_t dimension) -> at::Tensor {
1246:     pybind11::gil_scoped_release no_gil;
1247:     return at::_sobol_engine_scramble_(self, ltm, dimension);
1248:   };
1249:   return wrap(dispatch__sobol_engine_scramble_(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
1250:   Py_RETURN_NONE;
1251:   END_HANDLE_TH_ERRORS
1252: }
1253: 
1254: // _shape_as_tensor
1255: static PyObject * THPVariable__shape_as_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
1256: {
1257:   HANDLE_TH_ERRORS
1258:   static PythonArgParser parser({
1259:     "_shape_as_tensor(Tensor input)",
1260:   }, /*traceable=*/true);
1261: 
1262:   ParsedArgs<1> parsed_args;
1263:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1264:   if(_r.has_torch_function()) {
1265:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1266:   }
1267:   // aten::_shape_as_tensor(Tensor self) -> Tensor
1268: 
1269:   auto dispatch__shape_as_tensor = [](const at::Tensor & self) -> at::Tensor {
1270:     pybind11::gil_scoped_release no_gil;
1271:     return at::_shape_as_tensor(self);
1272:   };
1273:   return wrap(dispatch__shape_as_tensor(_r.tensor(0)));
1274:   Py_RETURN_NONE;
1275:   END_HANDLE_TH_ERRORS
1276: }
1277: 
1278: // dropout
1279: static PyObject * THPVariable_dropout(PyObject* self_, PyObject* args, PyObject* kwargs)
1280: {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__sobol_engine_ff_`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__sobol_engine_ff_`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1281-1360

```cpp
1281:   HANDLE_TH_ERRORS
1282:   static PythonArgParser parser({
1283:     "dropout(Tensor input, double p, bool train)",
1284:   }, /*traceable=*/true);
1285: 
1286:   ParsedArgs<3> parsed_args;
1287:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1288:   if(_r.has_torch_function()) {
1289:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1290:   }
1291:   // aten::dropout(Tensor input, float p, bool train) -> Tensor
1292: 
1293:   auto dispatch_dropout = [](const at::Tensor & input, double p, bool train) -> at::Tensor {
1294:     pybind11::gil_scoped_release no_gil;
1295:     return at::dropout(input, p, train);
1296:   };
1297:   return wrap(dispatch_dropout(_r.tensor(0), _r.toDouble(1), _r.toBool(2)));
1298:   Py_RETURN_NONE;
1299:   END_HANDLE_TH_ERRORS
1300: }
1301: 
1302: // dropout_
1303: static PyObject * THPVariable_dropout_(PyObject* self_, PyObject* args, PyObject* kwargs)
1304: {
1305:   HANDLE_TH_ERRORS
1306:   static PythonArgParser parser({
1307:     "dropout_(Tensor input, double p, bool train)",
1308:   }, /*traceable=*/true);
1309: 
1310:   ParsedArgs<3> parsed_args;
1311:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1312:   if(_r.has_torch_function()) {
1313:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1314:   }
1315:   // aten::dropout_(Tensor(a!) self, float p, bool train) -> Tensor(a!)
1316: 
1317:   auto dispatch_dropout_ = [](at::Tensor self, double p, bool train) -> at::Tensor {
1318:     pybind11::gil_scoped_release no_gil;
1319:     return at::dropout_(self, p, train);
1320:   };
1321:   return wrap(dispatch_dropout_(_r.tensor(0), _r.toDouble(1), _r.toBool(2)));
1322:   Py_RETURN_NONE;
1323:   END_HANDLE_TH_ERRORS
1324: }
1325: 
1326: // _conj
1327: static PyObject * THPVariable__conj(PyObject* self_, PyObject* args, PyObject* kwargs)
1328: {
1329:   HANDLE_TH_ERRORS
1330:   static PythonArgParser parser({
1331:     "_conj(Tensor input)",
1332:   }, /*traceable=*/true);
1333: 
1334:   ParsedArgs<1> parsed_args;
1335:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1336:   if(_r.has_torch_function()) {
1337:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1338:   }
1339:   // aten::_conj(Tensor(a) self) -> Tensor(a)
1340: 
1341:   auto dispatch__conj = [](const at::Tensor & self) -> at::Tensor {
1342:     pybind11::gil_scoped_release no_gil;
1343:     return self._conj();
1344:   };
1345:   return wrap(dispatch__conj(_r.tensor(0)));
1346:   Py_RETURN_NONE;
1347:   END_HANDLE_TH_ERRORS
1348: }
1349: 
1350: // conj_physical
1351: static PyObject * THPVariable_conj_physical(PyObject* self_, PyObject* args, PyObject* kwargs)
1352: {
1353:   HANDLE_TH_ERRORS
1354:   static PythonArgParser parser({
1355:     "conj_physical(Tensor input, *, Tensor out=None)",
1356:   }, /*traceable=*/true);
1357: 
1358:   ParsedArgs<2> parsed_args;
1359:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1360:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `dropout`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `dropout` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1361-1440

```cpp
1361:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1362:   }
1363:   if (_r.isNone(1)) {
1364:     // aten::conj_physical(Tensor self) -> Tensor
1365: 
1366:     auto dispatch_conj_physical = [](const at::Tensor & self) -> at::Tensor {
1367:       pybind11::gil_scoped_release no_gil;
1368:       return self.conj_physical();
1369:     };
1370:     return wrap(dispatch_conj_physical(_r.tensor(0)));
1371:   } else {
1372:     // aten::conj_physical.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1373: 
1374:     auto dispatch_conj_physical_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1375:       pybind11::gil_scoped_release no_gil;
1376:       return at::conj_physical_out(out, self);
1377:     };
1378:     return wrap(dispatch_conj_physical_out(_r.tensor(1), _r.tensor(0)));
1379:   }
1380:   Py_RETURN_NONE;
1381:   END_HANDLE_TH_ERRORS
1382: }
1383: 
1384: // conj_physical_
1385: static PyObject * THPVariable_conj_physical_(PyObject* self_, PyObject* args, PyObject* kwargs)
1386: {
1387:   HANDLE_TH_ERRORS
1388:   static PythonArgParser parser({
1389:     "conj_physical_(Tensor input)",
1390:   }, /*traceable=*/true);
1391: 
1392:   ParsedArgs<1> parsed_args;
1393:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1394:   if(_r.has_torch_function()) {
1395:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1396:   }
1397:   // aten::conj_physical_(Tensor(a!) self) -> Tensor(a!)
1398: 
1399:   auto dispatch_conj_physical_ = [](at::Tensor self) -> at::Tensor {
1400:     pybind11::gil_scoped_release no_gil;
1401:     return self.conj_physical_();
1402:   };
1403:   return wrap(dispatch_conj_physical_(_r.tensor(0)));
1404:   Py_RETURN_NONE;
1405:   END_HANDLE_TH_ERRORS
1406: }
1407: 
1408: // avg_pool1d
1409: static PyObject * THPVariable_avg_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs)
1410: {
1411:   HANDLE_TH_ERRORS
1412:   static PythonArgParser parser({
1413:     "avg_pool1d(Tensor input, IntArrayRef[1] kernel_size, IntArrayRef[1] stride=None, IntArrayRef[1] padding=0, bool ceil_mode=False, bool count_include_pad=True)",
1414:   }, /*traceable=*/true);
1415: 
1416:   ParsedArgs<6> parsed_args;
1417:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1418:   if(_r.has_torch_function()) {
1419:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1420:   }
1421:   // aten::avg_pool1d(Tensor self, int[1] kernel_size, int[1] stride=[], int[1] padding=0, bool ceil_mode=False, bool count_include_pad=True) -> Tensor
1422: 
1423:   auto dispatch_avg_pool1d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, bool ceil_mode, bool count_include_pad) -> at::Tensor {
1424:     pybind11::gil_scoped_release no_gil;
1425:     return at::avg_pool1d(self, kernel_size, stride, padding, ceil_mode, count_include_pad);
1426:   };
1427:   return wrap(dispatch_avg_pool1d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.toBool(4), _r.toBool(5)));
1428:   Py_RETURN_NONE;
1429:   END_HANDLE_TH_ERRORS
1430: }
1431: 
1432: \
1433: // add
1434: static PyObject * THPVariable_add(PyObject* self_, PyObject* args, PyObject* kwargs)
1435: {
1436:   HANDLE_TH_ERRORS
1437:   static PythonArgParser parser({
1438:     "add(Tensor input, Scalar alpha, Tensor other, *, Tensor out=None)|deprecated",
1439:     "add(Tensor input, Tensor other, *, Scalar alpha=1, Tensor out=None)",
1440:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `conj_physical`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `conj_physical`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1441-1520

```cpp
1441: 
1442:   ParsedArgs<4> parsed_args;
1443:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1444:   if(_r.has_torch_function()) {
1445:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1446:   }
1447:   switch (_r.idx) {
1448:     case 0: {
1449:       if (_r.isNone(3)) {
1450:         // [deprecated] aten::add(Tensor self, Scalar alpha, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
1451: 
1452:         auto dispatch_add = [](at::Tensor out, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
1453:           pybind11::gil_scoped_release no_gil;
1454:           return self.add(other, alpha);
1455:         };
1456:         return wrap(dispatch_add(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.tensor(2)));
1457:       } else {
1458:         // [deprecated] aten::add(Tensor self, Scalar alpha, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
1459: 
1460:         auto dispatch_add_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
1461:           pybind11::gil_scoped_release no_gil;
1462:           return at::add_out(out, self, other, alpha);
1463:         };
1464:         return wrap(dispatch_add_out(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.tensor(2)));
1465:       }
1466:     }
1467:     case 1: {
1468:       if (_r.isNone(3)) {
1469:         // aten::add.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
1470: 
1471:         auto dispatch_add = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
1472:           pybind11::gil_scoped_release no_gil;
1473:           return self.add(other, alpha);
1474:         };
1475:         return wrap(dispatch_add(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
1476:       } else {
1477:         // aten::add.out(Tensor self, Tensor other, *, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
1478: 
1479:         auto dispatch_add_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
1480:           pybind11::gil_scoped_release no_gil;
1481:           return at::add_out(out, self, other, alpha);
1482:         };
1483:         return wrap(dispatch_add_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.scalar(2)));
1484:       }
1485:     }
1486:   }
1487:   Py_RETURN_NONE;
1488:   END_HANDLE_TH_ERRORS
1489: }
1490: 
1491: \
1492: // _add_relu
1493: static PyObject * THPVariable__add_relu(PyObject* self_, PyObject* args, PyObject* kwargs)
1494: {
1495:   HANDLE_TH_ERRORS
1496:   static PythonArgParser parser({
1497:     "_add_relu(Tensor input, Tensor other, *, Scalar alpha=1, Tensor out=None)",
1498:     "_add_relu(Tensor input, Scalar other, Scalar alpha=1)",
1499:   }, /*traceable=*/true);
1500: 
1501:   ParsedArgs<4> parsed_args;
1502:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1503:   if(_r.has_torch_function()) {
1504:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1505:   }
1506:   switch (_r.idx) {
1507:     case 0: {
1508:       if (_r.isNone(3)) {
1509:         // aten::_add_relu.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
1510: 
1511:         auto dispatch__add_relu = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
1512:           pybind11::gil_scoped_release no_gil;
1513:           return at::_add_relu(self, other, alpha);
1514:         };
1515:         return wrap(dispatch__add_relu(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
1516:       } else {
1517:         // aten::_add_relu.out(Tensor self, Tensor other, *, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
1518: 
1519:         auto dispatch__add_relu_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
1520:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `add`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `add`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521:           return at::_add_relu_out(out, self, other, alpha);
1522:         };
1523:         return wrap(dispatch__add_relu_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.scalar(2)));
1524:       }
1525:     }
1526:     case 1: {
1527:       // aten::_add_relu.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> Tensor
1528: 
1529:       auto dispatch__add_relu = [](const at::Tensor & self, const at::Scalar & other, const at::Scalar & alpha) -> at::Tensor {
1530:         pybind11::gil_scoped_release no_gil;
1531:         return at::_add_relu(self, other, alpha);
1532:       };
1533:       return wrap(dispatch__add_relu(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
1534:     }
1535:   }
1536:   Py_RETURN_NONE;
1537:   END_HANDLE_TH_ERRORS
1538: }
1539: 
1540: \
1541: // _add_relu_
1542: static PyObject * THPVariable__add_relu_(PyObject* self_, PyObject* args, PyObject* kwargs)
1543: {
1544:   HANDLE_TH_ERRORS
1545:   static PythonArgParser parser({
1546:     "_add_relu_(Tensor input, Tensor other, *, Scalar alpha=1)",
1547:     "_add_relu_(Tensor input, Scalar other, Scalar alpha=1)",
1548:   }, /*traceable=*/true);
1549: 
1550:   ParsedArgs<3> parsed_args;
1551:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1552:   if(_r.has_torch_function()) {
1553:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1554:   }
1555:   switch (_r.idx) {
1556:     case 0: {
1557:       // aten::_add_relu_.Tensor(Tensor(a!) self, Tensor other, *, Scalar alpha=1) -> Tensor(a!)
1558: 
1559:       auto dispatch__add_relu_ = [](at::Tensor self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
1560:         pybind11::gil_scoped_release no_gil;
1561:         return at::_add_relu_(self, other, alpha);
1562:       };
1563:       return wrap(dispatch__add_relu_(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
1564:     }
1565:     case 1: {
1566:       // aten::_add_relu_.Scalar(Tensor(a!) self, Scalar other, Scalar alpha=1) -> Tensor(a!)
1567: 
1568:       auto dispatch__add_relu_ = [](at::Tensor self, const at::Scalar & other, const at::Scalar & alpha) -> at::Tensor {
1569:         pybind11::gil_scoped_release no_gil;
1570:         return at::_add_relu_(self, other, alpha);
1571:       };
1572:       return wrap(dispatch__add_relu_(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
1573:     }
1574:   }
1575:   Py_RETURN_NONE;
1576:   END_HANDLE_TH_ERRORS
1577: }
1578: 
1579: \
1580: // addr
1581: static PyObject * THPVariable_addr(PyObject* self_, PyObject* args, PyObject* kwargs)
1582: {
1583:   HANDLE_TH_ERRORS
1584:   static PythonArgParser parser({
1585:     "addr(Scalar beta, Tensor input, Scalar alpha, Tensor vec1, Tensor vec2, *, Tensor out=None)|deprecated",
1586:     "addr(Scalar beta, Tensor input, Tensor vec1, Tensor vec2, *, Tensor out=None)|deprecated",
1587:     "addr(Tensor input, Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1, Tensor out=None)",
1588:   }, /*traceable=*/true);
1589: 
1590:   ParsedArgs<6> parsed_args;
1591:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1592:   if(_r.has_torch_function()) {
1593:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1594:   }
1595:   switch (_r.idx) {
1596:     case 0: {
1597:       if (_r.isNone(5)) {
1598:         // [deprecated] aten::addr(Scalar beta, Tensor self, Scalar alpha, Tensor vec1, Tensor vec2, *, Tensor(a!) out) -> Tensor(a!)
1599: 
1600:         auto dispatch_addr = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `_add_relu_out`, `wrap`, `_add_relu`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_add_relu_out`, `wrap`, `_add_relu` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601:           pybind11::gil_scoped_release no_gil;
1602:           return self.addr(vec1, vec2, beta, alpha);
1603:         };
1604:         return wrap(dispatch_addr(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
1605:       } else {
1606:         // [deprecated] aten::addr(Scalar beta, Tensor self, Scalar alpha, Tensor vec1, Tensor vec2, *, Tensor(a!) out) -> Tensor(a!)
1607: 
1608:         auto dispatch_addr_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
1609:           pybind11::gil_scoped_release no_gil;
1610:           return at::addr_out(out, self, vec1, vec2, beta, alpha);
1611:         };
1612:         return wrap(dispatch_addr_out(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
1613:       }
1614:     }
1615:     case 1: {
1616:       if (_r.isNone(4)) {
1617:         // [deprecated] aten::addr(Scalar beta, Tensor self, Tensor vec1, Tensor vec2, *, Tensor(a!) out) -> Tensor(a!)
1618: 
1619:         auto dispatch_addr = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
1620:           pybind11::gil_scoped_release no_gil;
1621:           return self.addr(vec1, vec2, beta, 1);
1622:         };
1623:         return wrap(dispatch_addr(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
1624:       } else {
1625:         // [deprecated] aten::addr(Scalar beta, Tensor self, Tensor vec1, Tensor vec2, *, Tensor(a!) out) -> Tensor(a!)
1626: 
1627:         auto dispatch_addr_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
1628:           pybind11::gil_scoped_release no_gil;
1629:           return at::addr_out(out, self, vec1, vec2, beta, 1);
1630:         };
1631:         return wrap(dispatch_addr_out(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
1632:       }
1633:     }
1634:     case 2: {
1635:       if (_r.isNone(5)) {
1636:         // aten::addr(Tensor self, Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
1637: 
1638:         auto dispatch_addr = [](const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
1639:           pybind11::gil_scoped_release no_gil;
1640:           return self.addr(vec1, vec2, beta, alpha);
1641:         };
1642:         return wrap(dispatch_addr(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
1643:       } else {
1644:         // aten::addr.out(Tensor self, Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
1645: 
1646:         auto dispatch_addr_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
1647:           pybind11::gil_scoped_release no_gil;
1648:           return at::addr_out(out, self, vec1, vec2, beta, alpha);
1649:         };
1650:         return wrap(dispatch_addr_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
1651:       }
1652:     }
1653:   }
1654:   Py_RETURN_NONE;
1655:   END_HANDLE_TH_ERRORS
1656: }
1657: 
1658: // _test_functorch_fallback
1659: static PyObject * THPVariable__test_functorch_fallback(PyObject* self_, PyObject* args, PyObject* kwargs)
1660: {
1661:   HANDLE_TH_ERRORS
1662:   static PythonArgParser parser({
1663:     "_test_functorch_fallback(Tensor input, Tensor other)",
1664:   }, /*traceable=*/true);
1665: 
1666:   ParsedArgs<2> parsed_args;
1667:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1668:   if(_r.has_torch_function()) {
1669:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1670:   }
1671:   // aten::_test_functorch_fallback(Tensor self, Tensor other) -> Tensor
1672: 
1673:   auto dispatch__test_functorch_fallback = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1674:     pybind11::gil_scoped_release no_gil;
1675:     return at::_test_functorch_fallback(self, other);
1676:   };
1677:   return wrap(dispatch__test_functorch_fallback(_r.tensor(0), _r.tensor(1)));
1678:   Py_RETURN_NONE;
1679:   END_HANDLE_TH_ERRORS
1680: }
```

- EN: The main execution path in this span is carried by `wrap`, `addr`, `addr_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `addr`, `addr_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681: 
1682: \
1683: // all
1684: static PyObject * THPVariable_all(PyObject* self_, PyObject* args, PyObject* kwargs)
1685: {
1686:   HANDLE_TH_ERRORS
1687:   static PythonArgParser parser({
1688:     "all(Tensor input, *, Tensor out=None)",
1689:     "all(Tensor input, IntArrayRef? dim=None, bool keepdim=False, *, Tensor out=None)",
1690:     "all(Tensor input, int64_t dim, bool keepdim=False, *, Tensor out=None)",
1691:     "all(Tensor input, Dimname dim, bool keepdim=False, *, Tensor out=None)",
1692:   }, /*traceable=*/true);
1693: 
1694:   ParsedArgs<4> parsed_args;
1695:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1696:   if(_r.has_torch_function()) {
1697:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1698:   }
1699:   switch (_r.idx) {
1700:     case 0: {
1701:       if (_r.isNone(1)) {
1702:         // aten::all(Tensor self) -> Tensor
1703: 
1704:         auto dispatch_all = [](const at::Tensor & self) -> at::Tensor {
1705:           pybind11::gil_scoped_release no_gil;
1706:           return self.all();
1707:         };
1708:         return wrap(dispatch_all(_r.tensor(0)));
1709:       } else {
1710:         // aten::all.all_out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1711: 
1712:         auto dispatch_all_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1713:           pybind11::gil_scoped_release no_gil;
1714:           return at::all_out(out, self);
1715:         };
1716:         return wrap(dispatch_all_out(_r.tensor(1), _r.tensor(0)));
1717:       }
1718:     }
1719:     case 1: {
1720:       if (_r.isNone(3)) {
1721:         // aten::all.dims(Tensor self, int[]? dim=None, bool keepdim=False) -> Tensor
1722: 
1723:         auto dispatch_all = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim) -> at::Tensor {
1724:           pybind11::gil_scoped_release no_gil;
1725:           return self.all(dim, keepdim);
1726:         };
1727:         return wrap(dispatch_all(_r.tensor(0), _r.intlistOptional(1), _r.toBool(2)));
1728:       } else {
1729:         // aten::all.dims_out(Tensor self, int[]? dim=None, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
1730: 
1731:         auto dispatch_all_out = [](at::Tensor out, const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim) -> at::Tensor {
1732:           pybind11::gil_scoped_release no_gil;
1733:           return at::all_out(out, self, dim, keepdim);
1734:         };
1735:         return wrap(dispatch_all_out(_r.tensor(3), _r.tensor(0), _r.intlistOptional(1), _r.toBool(2)));
1736:       }
1737:     }
1738:     case 2: {
1739:       if (_r.isNone(3)) {
1740:         // aten::all.dim(Tensor self, int dim, bool keepdim=False) -> Tensor
1741: 
1742:         auto dispatch_all = [](const at::Tensor & self, int64_t dim, bool keepdim) -> at::Tensor {
1743:           pybind11::gil_scoped_release no_gil;
1744:           return self.all(dim, keepdim);
1745:         };
1746:         return wrap(dispatch_all(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
1747:       } else {
1748:         // aten::all.out(Tensor self, int dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
1749: 
1750:         auto dispatch_all_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, bool keepdim) -> at::Tensor {
1751:           pybind11::gil_scoped_release no_gil;
1752:           return at::all_out(out, self, dim, keepdim);
1753:         };
1754:         return wrap(dispatch_all_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
1755:       }
1756:     }
1757:     case 3: {
1758:       if (_r.isNone(3)) {
1759:         // aten::all.dimname(Tensor self, Dimname dim, bool keepdim=False) -> Tensor
1760: 
```

- EN: The main execution path in this span is carried by `THPVariable_all`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_all`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1761-1840

```cpp
1761:         auto dispatch_all = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> at::Tensor {
1762:           pybind11::gil_scoped_release no_gil;
1763:           return self.all(dim, keepdim);
1764:         };
1765:         return wrap(dispatch_all(_r.tensor(0), _r.dimname(1), _r.toBool(2)));
1766:       } else {
1767:         // aten::all.dimname_out(Tensor self, Dimname dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
1768: 
1769:         auto dispatch_all_out = [](at::Tensor out, const at::Tensor & self, at::Dimname dim, bool keepdim) -> at::Tensor {
1770:           pybind11::gil_scoped_release no_gil;
1771:           return at::all_out(out, self, dim, keepdim);
1772:         };
1773:         return wrap(dispatch_all_out(_r.tensor(3), _r.tensor(0), _r.dimname(1), _r.toBool(2)));
1774:       }
1775:     }
1776:   }
1777:   Py_RETURN_NONE;
1778:   END_HANDLE_TH_ERRORS
1779: }
1780: 
1781: // allclose
1782: static PyObject * THPVariable_allclose(PyObject* self_, PyObject* args, PyObject* kwargs)
1783: {
1784:   HANDLE_TH_ERRORS
1785:   static PythonArgParser parser({
1786:     "allclose(Tensor input, Tensor other, double rtol=1e-05, double atol=1e-08, bool equal_nan=False)",
1787:   }, /*traceable=*/false);
1788: 
1789:   ParsedArgs<5> parsed_args;
1790:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1791:   if(_r.has_torch_function()) {
1792:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1793:   }
1794:   // aten::allclose(Tensor self, Tensor other, float rtol=1e-05, float atol=1e-08, bool equal_nan=False) -> bool
1795: 
1796:   auto dispatch_allclose = [](const at::Tensor & self, const at::Tensor & other, double rtol, double atol, bool equal_nan) -> bool {
1797:     pybind11::gil_scoped_release no_gil;
1798:     return self.allclose(other, rtol, atol, equal_nan);
1799:   };
1800:   return wrap(dispatch_allclose(_r.tensor(0), _r.tensor(1), _r.toDouble(2), _r.toDouble(3), _r.toBool(4)));
1801:   Py_RETURN_NONE;
1802:   END_HANDLE_TH_ERRORS
1803: }
1804: 
1805: // argmax
1806: static PyObject * THPVariable_argmax(PyObject* self_, PyObject* args, PyObject* kwargs)
1807: {
1808:   HANDLE_TH_ERRORS
1809:   static PythonArgParser parser({
1810:     "argmax(Tensor input, int64_t? dim=None, bool keepdim=False, *, Tensor out=None)",
1811:   }, /*traceable=*/true);
1812: 
1813:   ParsedArgs<4> parsed_args;
1814:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1815:   if(_r.has_torch_function()) {
1816:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1817:   }
1818:   if (_r.isNone(3)) {
1819:     // aten::argmax(Tensor self, int? dim=None, bool keepdim=False) -> Tensor
1820: 
1821:     auto dispatch_argmax = [](const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> at::Tensor {
1822:       pybind11::gil_scoped_release no_gil;
1823:       return self.argmax(dim, keepdim);
1824:     };
1825:     return wrap(dispatch_argmax(_r.tensor(0), _r.toInt64Optional(1), _r.toBool(2)));
1826:   } else {
1827:     // aten::argmax.out(Tensor self, int? dim=None, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
1828: 
1829:     auto dispatch_argmax_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> at::Tensor {
1830:       pybind11::gil_scoped_release no_gil;
1831:       return at::argmax_out(out, self, dim, keepdim);
1832:     };
1833:     return wrap(dispatch_argmax_out(_r.tensor(3), _r.tensor(0), _r.toInt64Optional(1), _r.toBool(2)));
1834:   }
1835:   Py_RETURN_NONE;
1836:   END_HANDLE_TH_ERRORS
1837: }
1838: 
1839: // acosh
1840: static PyObject * THPVariable_acosh(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `wrap`, `all_out`, `THPVariable_allclose`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `all_out`, `THPVariable_allclose` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1841-1920

```cpp
1841: {
1842:   HANDLE_TH_ERRORS
1843:   static PythonArgParser parser({
1844:     "acosh(Tensor input, *, Tensor out=None)",
1845:   }, /*traceable=*/true);
1846: 
1847:   ParsedArgs<2> parsed_args;
1848:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1849:   if(_r.has_torch_function()) {
1850:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1851:   }
1852:   if (_r.isNone(1)) {
1853:     // aten::acosh(Tensor self) -> Tensor
1854: 
1855:     auto dispatch_acosh = [](const at::Tensor & self) -> at::Tensor {
1856:       pybind11::gil_scoped_release no_gil;
1857:       return self.acosh();
1858:     };
1859:     return wrap(dispatch_acosh(_r.tensor(0)));
1860:   } else {
1861:     // aten::acosh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1862: 
1863:     auto dispatch_acosh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1864:       pybind11::gil_scoped_release no_gil;
1865:       return at::acosh_out(out, self);
1866:     };
1867:     return wrap(dispatch_acosh_out(_r.tensor(1), _r.tensor(0)));
1868:   }
1869:   Py_RETURN_NONE;
1870:   END_HANDLE_TH_ERRORS
1871: }
1872: 
1873: // acosh_
1874: static PyObject * THPVariable_acosh_(PyObject* self_, PyObject* args, PyObject* kwargs)
1875: {
1876:   HANDLE_TH_ERRORS
1877:   static PythonArgParser parser({
1878:     "acosh_(Tensor input)",
1879:   }, /*traceable=*/true);
1880: 
1881:   ParsedArgs<1> parsed_args;
1882:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1883:   if(_r.has_torch_function()) {
1884:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1885:   }
1886:   // aten::acosh_(Tensor(a!) self) -> Tensor(a!)
1887: 
1888:   auto dispatch_acosh_ = [](at::Tensor self) -> at::Tensor {
1889:     pybind11::gil_scoped_release no_gil;
1890:     return self.acosh_();
1891:   };
1892:   return wrap(dispatch_acosh_(_r.tensor(0)));
1893:   Py_RETURN_NONE;
1894:   END_HANDLE_TH_ERRORS
1895: }
1896: 
1897: // arccosh
1898: static PyObject * THPVariable_arccosh(PyObject* self_, PyObject* args, PyObject* kwargs)
1899: {
1900:   HANDLE_TH_ERRORS
1901:   static PythonArgParser parser({
1902:     "arccosh(Tensor input, *, Tensor out=None)",
1903:   }, /*traceable=*/true);
1904: 
1905:   ParsedArgs<2> parsed_args;
1906:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1907:   if(_r.has_torch_function()) {
1908:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1909:   }
1910:   if (_r.isNone(1)) {
1911:     // aten::arccosh(Tensor self) -> Tensor
1912: 
1913:     auto dispatch_arccosh = [](const at::Tensor & self) -> at::Tensor {
1914:       pybind11::gil_scoped_release no_gil;
1915:       return self.arccosh();
1916:     };
1917:     return wrap(dispatch_arccosh(_r.tensor(0)));
1918:   } else {
1919:     // aten::arccosh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1920: 
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `acosh`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `acosh` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:     auto dispatch_arccosh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1922:       pybind11::gil_scoped_release no_gil;
1923:       return at::arccosh_out(out, self);
1924:     };
1925:     return wrap(dispatch_arccosh_out(_r.tensor(1), _r.tensor(0)));
1926:   }
1927:   Py_RETURN_NONE;
1928:   END_HANDLE_TH_ERRORS
1929: }
1930: 
1931: // arccosh_
1932: static PyObject * THPVariable_arccosh_(PyObject* self_, PyObject* args, PyObject* kwargs)
1933: {
1934:   HANDLE_TH_ERRORS
1935:   static PythonArgParser parser({
1936:     "arccosh_(Tensor input)",
1937:   }, /*traceable=*/true);
1938: 
1939:   ParsedArgs<1> parsed_args;
1940:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1941:   if(_r.has_torch_function()) {
1942:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1943:   }
1944:   // aten::arccosh_(Tensor(a!) self) -> Tensor(a!)
1945: 
1946:   auto dispatch_arccosh_ = [](at::Tensor self) -> at::Tensor {
1947:     pybind11::gil_scoped_release no_gil;
1948:     return self.arccosh_();
1949:   };
1950:   return wrap(dispatch_arccosh_(_r.tensor(0)));
1951:   Py_RETURN_NONE;
1952:   END_HANDLE_TH_ERRORS
1953: }
1954: 
1955: // arctanh
1956: static PyObject * THPVariable_arctanh(PyObject* self_, PyObject* args, PyObject* kwargs)
1957: {
1958:   HANDLE_TH_ERRORS
1959:   static PythonArgParser parser({
1960:     "arctanh(Tensor input, *, Tensor out=None)",
1961:   }, /*traceable=*/true);
1962: 
1963:   ParsedArgs<2> parsed_args;
1964:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1965:   if(_r.has_torch_function()) {
1966:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1967:   }
1968:   if (_r.isNone(1)) {
1969:     // aten::arctanh(Tensor self) -> Tensor
1970: 
1971:     auto dispatch_arctanh = [](const at::Tensor & self) -> at::Tensor {
1972:       pybind11::gil_scoped_release no_gil;
1973:       return self.arctanh();
1974:     };
1975:     return wrap(dispatch_arctanh(_r.tensor(0)));
1976:   } else {
1977:     // aten::arctanh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1978: 
1979:     auto dispatch_arctanh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1980:       pybind11::gil_scoped_release no_gil;
1981:       return at::arctanh_out(out, self);
1982:     };
1983:     return wrap(dispatch_arctanh_out(_r.tensor(1), _r.tensor(0)));
1984:   }
1985:   Py_RETURN_NONE;
1986:   END_HANDLE_TH_ERRORS
1987: }
1988: 
1989: // arctanh_
1990: static PyObject * THPVariable_arctanh_(PyObject* self_, PyObject* args, PyObject* kwargs)
1991: {
1992:   HANDLE_TH_ERRORS
1993:   static PythonArgParser parser({
1994:     "arctanh_(Tensor input)",
1995:   }, /*traceable=*/true);
1996: 
1997:   ParsedArgs<1> parsed_args;
1998:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1999:   if(_r.has_torch_function()) {
2000:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `arccosh_out`, `wrap`, `THPVariable_arccosh_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `arccosh_out`, `wrap`, `THPVariable_arccosh_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001:   }
2002:   // aten::arctanh_(Tensor(a!) self) -> Tensor(a!)
2003: 
2004:   auto dispatch_arctanh_ = [](at::Tensor self) -> at::Tensor {
2005:     pybind11::gil_scoped_release no_gil;
2006:     return self.arctanh_();
2007:   };
2008:   return wrap(dispatch_arctanh_(_r.tensor(0)));
2009:   Py_RETURN_NONE;
2010:   END_HANDLE_TH_ERRORS
2011: }
2012: 
2013: // arcsin
2014: static PyObject * THPVariable_arcsin(PyObject* self_, PyObject* args, PyObject* kwargs)
2015: {
2016:   HANDLE_TH_ERRORS
2017:   static PythonArgParser parser({
2018:     "arcsin(Tensor input, *, Tensor out=None)",
2019:   }, /*traceable=*/true);
2020: 
2021:   ParsedArgs<2> parsed_args;
2022:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2023:   if(_r.has_torch_function()) {
2024:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2025:   }
2026:   if (_r.isNone(1)) {
2027:     // aten::arcsin(Tensor self) -> Tensor
2028: 
2029:     auto dispatch_arcsin = [](const at::Tensor & self) -> at::Tensor {
2030:       pybind11::gil_scoped_release no_gil;
2031:       return self.arcsin();
2032:     };
2033:     return wrap(dispatch_arcsin(_r.tensor(0)));
2034:   } else {
2035:     // aten::arcsin.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2036: 
2037:     auto dispatch_arcsin_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2038:       pybind11::gil_scoped_release no_gil;
2039:       return at::arcsin_out(out, self);
2040:     };
2041:     return wrap(dispatch_arcsin_out(_r.tensor(1), _r.tensor(0)));
2042:   }
2043:   Py_RETURN_NONE;
2044:   END_HANDLE_TH_ERRORS
2045: }
2046: 
2047: // arcsin_
2048: static PyObject * THPVariable_arcsin_(PyObject* self_, PyObject* args, PyObject* kwargs)
2049: {
2050:   HANDLE_TH_ERRORS
2051:   static PythonArgParser parser({
2052:     "arcsin_(Tensor input)",
2053:   }, /*traceable=*/true);
2054: 
2055:   ParsedArgs<1> parsed_args;
2056:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2057:   if(_r.has_torch_function()) {
2058:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2059:   }
2060:   // aten::arcsin_(Tensor(a!) self) -> Tensor(a!)
2061: 
2062:   auto dispatch_arcsin_ = [](at::Tensor self) -> at::Tensor {
2063:     pybind11::gil_scoped_release no_gil;
2064:     return self.arcsin_();
2065:   };
2066:   return wrap(dispatch_arcsin_(_r.tensor(0)));
2067:   Py_RETURN_NONE;
2068:   END_HANDLE_TH_ERRORS
2069: }
2070: 
2071: // atan
2072: static PyObject * THPVariable_atan(PyObject* self_, PyObject* args, PyObject* kwargs)
2073: {
2074:   HANDLE_TH_ERRORS
2075:   static PythonArgParser parser({
2076:     "atan(Tensor input, *, Tensor out=None)",
2077:   }, /*traceable=*/true);
2078: 
2079:   ParsedArgs<2> parsed_args;
2080:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `arctanh_`, `wrap`, `THPVariable_arcsin`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `arctanh_`, `wrap`, `THPVariable_arcsin` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081:   if(_r.has_torch_function()) {
2082:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2083:   }
2084:   if (_r.isNone(1)) {
2085:     // aten::atan(Tensor self) -> Tensor
2086: 
2087:     auto dispatch_atan = [](const at::Tensor & self) -> at::Tensor {
2088:       pybind11::gil_scoped_release no_gil;
2089:       return self.atan();
2090:     };
2091:     return wrap(dispatch_atan(_r.tensor(0)));
2092:   } else {
2093:     // aten::atan.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2094: 
2095:     auto dispatch_atan_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2096:       pybind11::gil_scoped_release no_gil;
2097:       return at::atan_out(out, self);
2098:     };
2099:     return wrap(dispatch_atan_out(_r.tensor(1), _r.tensor(0)));
2100:   }
2101:   Py_RETURN_NONE;
2102:   END_HANDLE_TH_ERRORS
2103: }
2104: 
2105: // atan_
2106: static PyObject * THPVariable_atan_(PyObject* self_, PyObject* args, PyObject* kwargs)
2107: {
2108:   HANDLE_TH_ERRORS
2109:   static PythonArgParser parser({
2110:     "atan_(Tensor input)",
2111:   }, /*traceable=*/true);
2112: 
2113:   ParsedArgs<1> parsed_args;
2114:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2115:   if(_r.has_torch_function()) {
2116:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2117:   }
2118:   // aten::atan_(Tensor(a!) self) -> Tensor(a!)
2119: 
2120:   auto dispatch_atan_ = [](at::Tensor self) -> at::Tensor {
2121:     pybind11::gil_scoped_release no_gil;
2122:     return self.atan_();
2123:   };
2124:   return wrap(dispatch_atan_(_r.tensor(0)));
2125:   Py_RETURN_NONE;
2126:   END_HANDLE_TH_ERRORS
2127: }
2128: 
2129: \
2130: // baddbmm
2131: static PyObject * THPVariable_baddbmm(PyObject* self_, PyObject* args, PyObject* kwargs)
2132: {
2133:   HANDLE_TH_ERRORS
2134:   static PythonArgParser parser({
2135:     "baddbmm(Scalar beta, Tensor input, Scalar alpha, Tensor batch1, Tensor batch2, *, Tensor out=None)|deprecated",
2136:     "baddbmm(Scalar beta, Tensor input, Tensor batch1, Tensor batch2, *, Tensor out=None)|deprecated",
2137:     "baddbmm(Tensor input, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1, Tensor out=None)",
2138:     "baddbmm(Tensor input, Tensor batch1, Tensor batch2, ScalarType out_dtype, *, Scalar beta=1, Scalar alpha=1, Tensor out=None)",
2139:   }, /*traceable=*/true);
2140: 
2141:   ParsedArgs<7> parsed_args;
2142:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2143:   if(_r.has_torch_function()) {
2144:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2145:   }
2146:   switch (_r.idx) {
2147:     case 0: {
2148:       if (_r.isNone(5)) {
2149:         // [deprecated] aten::baddbmm(Scalar beta, Tensor self, Scalar alpha, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
2150: 
2151:         auto dispatch_baddbmm = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2152:           pybind11::gil_scoped_release no_gil;
2153:           return self.baddbmm(batch1, batch2, beta, alpha);
2154:         };
2155:         return wrap(dispatch_baddbmm(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
2156:       } else {
2157:         // [deprecated] aten::baddbmm(Scalar beta, Tensor self, Scalar alpha, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
2158: 
2159:         auto dispatch_baddbmm_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2160:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `atan`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `atan`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2240

```cpp
2161:           return at::baddbmm_out(out, self, batch1, batch2, beta, alpha);
2162:         };
2163:         return wrap(dispatch_baddbmm_out(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
2164:       }
2165:     }
2166:     case 1: {
2167:       if (_r.isNone(4)) {
2168:         // [deprecated] aten::baddbmm(Scalar beta, Tensor self, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
2169: 
2170:         auto dispatch_baddbmm = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2171:           pybind11::gil_scoped_release no_gil;
2172:           return self.baddbmm(batch1, batch2, beta, 1);
2173:         };
2174:         return wrap(dispatch_baddbmm(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
2175:       } else {
2176:         // [deprecated] aten::baddbmm(Scalar beta, Tensor self, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
2177: 
2178:         auto dispatch_baddbmm_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2179:           pybind11::gil_scoped_release no_gil;
2180:           return at::baddbmm_out(out, self, batch1, batch2, beta, 1);
2181:         };
2182:         return wrap(dispatch_baddbmm_out(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
2183:       }
2184:     }
2185:     case 2: {
2186:       if (_r.isNone(5)) {
2187:         // aten::baddbmm(Tensor self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
2188: 
2189:         auto dispatch_baddbmm = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
2190:           pybind11::gil_scoped_release no_gil;
2191:           return self.baddbmm(batch1, batch2, beta, alpha);
2192:         };
2193:         return wrap(dispatch_baddbmm(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
2194:       } else {
2195:         // aten::baddbmm.out(Tensor self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
2196: 
2197:         auto dispatch_baddbmm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
2198:           pybind11::gil_scoped_release no_gil;
2199:           return at::baddbmm_out(out, self, batch1, batch2, beta, alpha);
2200:         };
2201:         return wrap(dispatch_baddbmm_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
2202:       }
2203:     }
2204:     case 3: {
2205:       if (_r.isNone(6)) {
2206:         // aten::baddbmm.dtype(Tensor self, Tensor batch1, Tensor batch2, ScalarType out_dtype, *, Scalar beta=1, Scalar alpha=1) -> Tensor
2207: 
2208:         auto dispatch_baddbmm = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, at::ScalarType out_dtype, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
2209:           pybind11::gil_scoped_release no_gil;
2210:           return at::baddbmm(self, batch1, batch2, out_dtype, beta, alpha);
2211:         };
2212:         return wrap(dispatch_baddbmm(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalartype(3), _r.scalar(4), _r.scalar(5)));
2213:       } else {
2214:         // aten::baddbmm.dtype_out(Tensor self, Tensor batch1, Tensor batch2, ScalarType out_dtype, *, Scalar beta=1, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
2215: 
2216:         auto dispatch_baddbmm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, at::ScalarType out_dtype, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
2217:           pybind11::gil_scoped_release no_gil;
2218:           return at::baddbmm_out(out, self, batch1, batch2, out_dtype, beta, alpha);
2219:         };
2220:         return wrap(dispatch_baddbmm_out(_r.tensor(6), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalartype(3), _r.scalar(4), _r.scalar(5)));
2221:       }
2222:     }
2223:   }
2224:   Py_RETURN_NONE;
2225:   END_HANDLE_TH_ERRORS
2226: }
2227: 
2228: \
2229: // bartlett_window
2230: static PyObject * THPVariable_bartlett_window(PyObject* self_, PyObject* args, PyObject* kwargs)
2231: {
2232:   HANDLE_TH_ERRORS
2233:   static PythonArgParser parser({
2234:     "bartlett_window(int64_t window_length, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
2235:     "bartlett_window(int64_t window_length, bool periodic, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
2236:   }, /*traceable=*/true);
2237: 
2238:   ParsedArgs<7> parsed_args;
2239:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2240:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `baddbmm_out`, `wrap`, `baddbmm`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `baddbmm_out`, `wrap`, `baddbmm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2241-2320

```cpp
2241:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2242:   }
2243:   switch (_r.idx) {
2244:     case 0: {
2245:       // aten::bartlett_window(int window_length, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
2246:       const auto options = TensorOptions()
2247:           .dtype(_r.scalartypeOptional(1))
2248:           .device(_r.deviceWithDefault(3, torch::tensors::get_default_device()))
2249:           .layout(_r.layoutOptional(2))
2250:           .requires_grad(_r.toBool(5))
2251:           .pinned_memory(_r.toBool(4));
2252:       torch::utils::maybe_initialize_device(options);
2253: 
2254:       auto dispatch_bartlett_window = [](int64_t window_length, at::TensorOptions options) -> at::Tensor {
2255:         pybind11::gil_scoped_release no_gil;
2256:         return torch::bartlett_window(window_length, options);
2257:       };
2258:       return wrap(dispatch_bartlett_window(_r.toInt64(0), options));
2259:     }
2260:     case 1: {
2261:       // aten::bartlett_window.periodic(int window_length, bool periodic, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
2262:       const auto options = TensorOptions()
2263:           .dtype(_r.scalartypeOptional(2))
2264:           .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
2265:           .layout(_r.layoutOptional(3))
2266:           .requires_grad(_r.toBool(6))
2267:           .pinned_memory(_r.toBool(5));
2268:       torch::utils::maybe_initialize_device(options);
2269: 
2270:       auto dispatch_bartlett_window = [](int64_t window_length, bool periodic, at::TensorOptions options) -> at::Tensor {
2271:         pybind11::gil_scoped_release no_gil;
2272:         return torch::bartlett_window(window_length, periodic, options);
2273:       };
2274:       return wrap(dispatch_bartlett_window(_r.toInt64(0), _r.toBool(1), options));
2275:     }
2276:   }
2277:   Py_RETURN_NONE;
2278:   END_HANDLE_TH_ERRORS
2279: }
2280: 
2281: // batch_norm
2282: static PyObject * THPVariable_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
2283: {
2284:   HANDLE_TH_ERRORS
2285:   static PythonArgParser parser({
2286:     "batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, double momentum, double eps, bool cudnn_enabled)",
2287:   }, /*traceable=*/true);
2288: 
2289:   ParsedArgs<9> parsed_args;
2290:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2291:   if(_r.has_torch_function()) {
2292:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2293:   }
2294:   // aten::batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float momentum, float eps, bool cudnn_enabled) -> Tensor
2295: 
2296:   auto dispatch_batch_norm = [](const at::Tensor & input, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, bool training, double momentum, double eps, bool cudnn_enabled) -> at::Tensor {
2297:     pybind11::gil_scoped_release no_gil;
2298:     return at::batch_norm(input, weight, bias, running_mean, running_var, training, momentum, eps, cudnn_enabled);
2299:   };
2300:   return wrap(dispatch_batch_norm(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toBool(5), _r.toDouble(6), _r.toDouble(7), _r.toBool(8)));
2301:   Py_RETURN_NONE;
2302:   END_HANDLE_TH_ERRORS
2303: }
2304: 
2305: \
2306: // copysign
2307: static PyObject * THPVariable_copysign(PyObject* self_, PyObject* args, PyObject* kwargs)
2308: {
2309:   HANDLE_TH_ERRORS
2310:   static PythonArgParser parser({
2311:     "copysign(Tensor input, Tensor other, *, Tensor out=None)",
2312:     "copysign(Tensor input, Scalar other, *, Tensor out=None)",
2313:   }, /*traceable=*/true);
2314: 
2315:   ParsedArgs<3> parsed_args;
2316:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2317:   if(_r.has_torch_function()) {
2318:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2319:   }
2320:   switch (_r.idx) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `bartlett_window`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `bartlett_window`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2321-2400

```cpp
2321:     case 0: {
2322:       if (_r.isNone(2)) {
2323:         // aten::copysign.Tensor(Tensor self, Tensor other) -> Tensor
2324: 
2325:         auto dispatch_copysign = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2326:           pybind11::gil_scoped_release no_gil;
2327:           return self.copysign(other);
2328:         };
2329:         return wrap(dispatch_copysign(_r.tensor(0), _r.tensor(1)));
2330:       } else {
2331:         // aten::copysign.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2332: 
2333:         auto dispatch_copysign_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2334:           pybind11::gil_scoped_release no_gil;
2335:           return at::copysign_out(out, self, other);
2336:         };
2337:         return wrap(dispatch_copysign_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2338:       }
2339:     }
2340:     case 1: {
2341:       if (_r.isNone(2)) {
2342:         // aten::copysign.Scalar(Tensor self, Scalar other) -> Tensor
2343: 
2344:         auto dispatch_copysign = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2345:           pybind11::gil_scoped_release no_gil;
2346:           return self.copysign(other);
2347:         };
2348:         return wrap(dispatch_copysign(_r.tensor(0), _r.scalar(1)));
2349:       } else {
2350:         // aten::copysign.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
2351: 
2352:         auto dispatch_copysign_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2353:           pybind11::gil_scoped_release no_gil;
2354:           return at::copysign_out(out, self, other);
2355:         };
2356:         return wrap(dispatch_copysign_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2357:       }
2358:     }
2359:   }
2360:   Py_RETURN_NONE;
2361:   END_HANDLE_TH_ERRORS
2362: }
2363: 
2364: // _lazy_clone
2365: static PyObject * THPVariable__lazy_clone(PyObject* self_, PyObject* args, PyObject* kwargs)
2366: {
2367:   HANDLE_TH_ERRORS
2368:   static PythonArgParser parser({
2369:     "_lazy_clone(Tensor input)",
2370:   }, /*traceable=*/true);
2371: 
2372:   ParsedArgs<1> parsed_args;
2373:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2374:   if(_r.has_torch_function()) {
2375:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2376:   }
2377:   // aten::_lazy_clone(Tensor self) -> Tensor
2378: 
2379:   auto dispatch__lazy_clone = [](const at::Tensor & self) -> at::Tensor {
2380:     pybind11::gil_scoped_release no_gil;
2381:     return self._lazy_clone();
2382:   };
2383:   return wrap(dispatch__lazy_clone(_r.tensor(0)));
2384:   Py_RETURN_NONE;
2385:   END_HANDLE_TH_ERRORS
2386: }
2387: 
2388: // logical_not
2389: static PyObject * THPVariable_logical_not(PyObject* self_, PyObject* args, PyObject* kwargs)
2390: {
2391:   HANDLE_TH_ERRORS
2392:   static PythonArgParser parser({
2393:     "logical_not(Tensor input, *, Tensor out=None)",
2394:   }, /*traceable=*/true);
2395: 
2396:   ParsedArgs<2> parsed_args;
2397:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2398:   if(_r.has_torch_function()) {
2399:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2400:   }
```

- EN: The main execution path in this span is carried by `wrap`, `copysign_out`, `THPVariable__lazy_clone`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `copysign_out`, `THPVariable__lazy_clone` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401:   if (_r.isNone(1)) {
2402:     // aten::logical_not(Tensor self) -> Tensor
2403: 
2404:     auto dispatch_logical_not = [](const at::Tensor & self) -> at::Tensor {
2405:       pybind11::gil_scoped_release no_gil;
2406:       return self.logical_not();
2407:     };
2408:     return wrap(dispatch_logical_not(_r.tensor(0)));
2409:   } else {
2410:     // aten::logical_not.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2411: 
2412:     auto dispatch_logical_not_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2413:       pybind11::gil_scoped_release no_gil;
2414:       return at::logical_not_out(out, self);
2415:     };
2416:     return wrap(dispatch_logical_not_out(_r.tensor(1), _r.tensor(0)));
2417:   }
2418:   Py_RETURN_NONE;
2419:   END_HANDLE_TH_ERRORS
2420: }
2421: 
2422: // logical_xor
2423: static PyObject * THPVariable_logical_xor(PyObject* self_, PyObject* args, PyObject* kwargs)
2424: {
2425:   HANDLE_TH_ERRORS
2426:   static PythonArgParser parser({
2427:     "logical_xor(Tensor input, Tensor other, *, Tensor out=None)",
2428:   }, /*traceable=*/true);
2429: 
2430:   ParsedArgs<3> parsed_args;
2431:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2432:   if(_r.has_torch_function()) {
2433:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2434:   }
2435:   if (_r.isNone(2)) {
2436:     // aten::logical_xor(Tensor self, Tensor other) -> Tensor
2437: 
2438:     auto dispatch_logical_xor = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2439:       pybind11::gil_scoped_release no_gil;
2440:       return self.logical_xor(other);
2441:     };
2442:     return wrap(dispatch_logical_xor(_r.tensor(0), _r.tensor(1)));
2443:   } else {
2444:     // aten::logical_xor.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2445: 
2446:     auto dispatch_logical_xor_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2447:       pybind11::gil_scoped_release no_gil;
2448:       return at::logical_xor_out(out, self, other);
2449:     };
2450:     return wrap(dispatch_logical_xor_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2451:   }
2452:   Py_RETURN_NONE;
2453:   END_HANDLE_TH_ERRORS
2454: }
2455: 
2456: \
2457: // bmm
2458: static PyObject * THPVariable_bmm(PyObject* self_, PyObject* args, PyObject* kwargs)
2459: {
2460:   HANDLE_TH_ERRORS
2461:   static PythonArgParser parser({
2462:     "bmm(Tensor input, Tensor mat2, *, Tensor out=None)",
2463:     "bmm(Tensor input, Tensor mat2, ScalarType out_dtype, *, Tensor out=None)",
2464:   }, /*traceable=*/true);
2465: 
2466:   ParsedArgs<4> parsed_args;
2467:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2468:   if(_r.has_torch_function()) {
2469:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2470:   }
2471:   switch (_r.idx) {
2472:     case 0: {
2473:       if (_r.isNone(2)) {
2474:         // aten::bmm(Tensor self, Tensor mat2) -> Tensor
2475: 
2476:         auto dispatch_bmm = [](const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
2477:           pybind11::gil_scoped_release no_gil;
2478:           return self.bmm(mat2);
2479:         };
2480:         return wrap(dispatch_bmm(_r.tensor(0), _r.tensor(1)));
```

- EN: The main execution path in this span is carried by `logical_not`, `wrap`, `logical_not_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `logical_not`, `wrap`, `logical_not_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481:       } else {
2482:         // aten::bmm.out(Tensor self, Tensor mat2, *, Tensor(a!) out) -> Tensor(a!)
2483: 
2484:         auto dispatch_bmm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
2485:           pybind11::gil_scoped_release no_gil;
2486:           return at::bmm_out(out, self, mat2);
2487:         };
2488:         return wrap(dispatch_bmm_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2489:       }
2490:     }
2491:     case 1: {
2492:       if (_r.isNone(3)) {
2493:         // aten::bmm.dtype(Tensor self, Tensor mat2, ScalarType out_dtype) -> Tensor
2494: 
2495:         auto dispatch_bmm = [](const at::Tensor & self, const at::Tensor & mat2, at::ScalarType out_dtype) -> at::Tensor {
2496:           pybind11::gil_scoped_release no_gil;
2497:           return at::bmm(self, mat2, out_dtype);
2498:         };
2499:         return wrap(dispatch_bmm(_r.tensor(0), _r.tensor(1), _r.scalartype(2)));
2500:       } else {
2501:         // aten::bmm.dtype_out(Tensor self, Tensor mat2, ScalarType out_dtype, *, Tensor(a!) out) -> Tensor(a!)
2502: 
2503:         auto dispatch_bmm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat2, at::ScalarType out_dtype) -> at::Tensor {
2504:           pybind11::gil_scoped_release no_gil;
2505:           return at::bmm_out(out, self, mat2, out_dtype);
2506:         };
2507:         return wrap(dispatch_bmm_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.scalartype(2)));
2508:       }
2509:     }
2510:   }
2511:   Py_RETURN_NONE;
2512:   END_HANDLE_TH_ERRORS
2513: }
2514: 
2515: // broadcast_tensors
2516: static PyObject * THPVariable_broadcast_tensors(PyObject* self_, PyObject* args, PyObject* kwargs)
2517: {
2518:   HANDLE_TH_ERRORS
2519:   static PythonArgParser parser({
2520:     "broadcast_tensors(TensorList tensors)",
2521:   }, /*traceable=*/true);
2522: 
2523:   ParsedArgs<1> parsed_args;
2524:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2525:   if(_r.has_torch_function()) {
2526:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2527:   }
2528:   // aten::broadcast_tensors(Tensor[] tensors) -> Tensor[]
2529: 
2530:   auto dispatch_broadcast_tensors = [](at::TensorList tensors) -> ::std::vector<at::Tensor> {
2531:     pybind11::gil_scoped_release no_gil;
2532:     return at::broadcast_tensors(tensors);
2533:   };
2534:   return wrap(dispatch_broadcast_tensors(_r.tensorlist(0)));
2535:   Py_RETURN_NONE;
2536:   END_HANDLE_TH_ERRORS
2537: }
2538: 
2539: // broadcast_to
2540: static PyObject * THPVariable_broadcast_to(PyObject* self_, PyObject* args, PyObject* kwargs)
2541: {
2542:   HANDLE_TH_ERRORS
2543:   static PythonArgParser parser({
2544:     "broadcast_to(Tensor input, SymIntArrayRef size)",
2545:   }, /*traceable=*/true);
2546: 
2547:   ParsedArgs<2> parsed_args;
2548:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2549:   if(_r.has_torch_function()) {
2550:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2551:   }
2552:   // aten::broadcast_to(Tensor(a) self, SymInt[] size) -> Tensor(a)
2553: 
2554:   auto dispatch_broadcast_to = [](const at::Tensor & self, c10::SymIntArrayRef size) -> at::Tensor {
2555:     pybind11::gil_scoped_release no_gil;
2556:     return self.broadcast_to_symint(size);
2557:   };
2558:   return wrap(dispatch_broadcast_to(_r.tensor(0), _r.symintlist(1)));
2559:   Py_RETURN_NONE;
2560:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `bmm_out`, `wrap`, `bmm`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `bmm_out`, `wrap`, `bmm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2561-2640

```cpp
2561: }
2562: 
2563: // _sparse_broadcast_to
2564: static PyObject * THPVariable__sparse_broadcast_to(PyObject* self_, PyObject* args, PyObject* kwargs)
2565: {
2566:   HANDLE_TH_ERRORS
2567:   static PythonArgParser parser({
2568:     "_sparse_broadcast_to(Tensor input, IntArrayRef size)",
2569:   }, /*traceable=*/true);
2570: 
2571:   ParsedArgs<2> parsed_args;
2572:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2573:   if(_r.has_torch_function()) {
2574:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2575:   }
2576:   // aten::_sparse_broadcast_to(Tensor(a) self, int[] size) -> Tensor(a)
2577: 
2578:   auto dispatch__sparse_broadcast_to = [](const at::Tensor & self, at::IntArrayRef size) -> at::Tensor {
2579:     pybind11::gil_scoped_release no_gil;
2580:     return at::_sparse_broadcast_to(self, size);
2581:   };
2582:   return wrap(dispatch__sparse_broadcast_to(_r.tensor(0), _r.intlist(1)));
2583:   Py_RETURN_NONE;
2584:   END_HANDLE_TH_ERRORS
2585: }
2586: 
2587: \
2588: // cat
2589: static PyObject * THPVariable_cat(PyObject* self_, PyObject* args, PyObject* kwargs)
2590: {
2591:   HANDLE_TH_ERRORS
2592:   static PythonArgParser parser({
2593:     "cat(TensorList tensors, int64_t dim=0, *, Tensor out=None)",
2594:     "cat(TensorList tensors, Dimname dim, *, Tensor out=None)",
2595:   }, /*traceable=*/true);
2596: 
2597:   ParsedArgs<3> parsed_args;
2598:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2599:   if(_r.has_torch_function()) {
2600:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2601:   }
2602:   switch (_r.idx) {
2603:     case 0: {
2604:       if (_r.isNone(2)) {
2605:         // aten::cat(Tensor[] tensors, int dim=0) -> Tensor
2606: 
2607:         auto dispatch_cat = [](const at::ITensorListRef & tensors, int64_t dim) -> at::Tensor {
2608:           pybind11::gil_scoped_release no_gil;
2609:           return at::cat(tensors, dim);
2610:         };
2611:         return wrap(dispatch_cat(_r.tensorlist(0), _r.toInt64(1)));
2612:       } else {
2613:         // aten::cat.out(Tensor[] tensors, int dim=0, *, Tensor(a!) out) -> Tensor(a!)
2614: 
2615:         auto dispatch_cat_out = [](at::Tensor out, const at::ITensorListRef & tensors, int64_t dim) -> at::Tensor {
2616:           pybind11::gil_scoped_release no_gil;
2617:           return at::cat_out(out, tensors, dim);
2618:         };
2619:         return wrap(dispatch_cat_out(_r.tensor(2), _r.tensorlist(0), _r.toInt64(1)));
2620:       }
2621:     }
2622:     case 1: {
2623:       if (_r.isNone(2)) {
2624:         // aten::cat.names(Tensor[] tensors, Dimname dim) -> Tensor
2625: 
2626:         auto dispatch_cat = [](at::TensorList tensors, at::Dimname dim) -> at::Tensor {
2627:           pybind11::gil_scoped_release no_gil;
2628:           return at::cat(tensors, dim);
2629:         };
2630:         return wrap(dispatch_cat(_r.tensorlist(0), _r.dimname(1)));
2631:       } else {
2632:         // aten::cat.names_out(Tensor[] tensors, Dimname dim, *, Tensor(a!) out) -> Tensor(a!)
2633: 
2634:         auto dispatch_cat_out = [](at::Tensor out, at::TensorList tensors, at::Dimname dim) -> at::Tensor {
2635:           pybind11::gil_scoped_release no_gil;
2636:           return at::cat_out(out, tensors, dim);
2637:         };
2638:         return wrap(dispatch_cat_out(_r.tensor(2), _r.tensorlist(0), _r.dimname(1)));
2639:       }
2640:     }
```

- EN: The main execution path in this span is carried by `THPVariable__sparse_broadcast_to`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__sparse_broadcast_to`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2641-2720

```cpp
2641:   }
2642:   Py_RETURN_NONE;
2643:   END_HANDLE_TH_ERRORS
2644: }
2645: 
2646: // chain_matmul
2647: static PyObject * THPVariable_chain_matmul(PyObject* self_, PyObject* args, PyObject* kwargs)
2648: {
2649:   HANDLE_TH_ERRORS
2650:   static PythonArgParser parser({
2651:     "chain_matmul(TensorList matrices, *, Tensor out=None)",
2652:   }, /*traceable=*/true);
2653: 
2654:   ParsedArgs<2> parsed_args;
2655:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2656:   if(_r.has_torch_function()) {
2657:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2658:   }
2659:   if (_r.isNone(1)) {
2660:     // aten::chain_matmul(Tensor[] matrices) -> Tensor
2661: 
2662:     auto dispatch_chain_matmul = [](at::TensorList matrices) -> at::Tensor {
2663:       pybind11::gil_scoped_release no_gil;
2664:       return at::chain_matmul(matrices);
2665:     };
2666:     return wrap(dispatch_chain_matmul(_r.tensorlist(0)));
2667:   } else {
2668:     // aten::chain_matmul.out(Tensor[] matrices, *, Tensor(a!) out) -> Tensor(a!)
2669: 
2670:     auto dispatch_chain_matmul_out = [](at::Tensor out, at::TensorList matrices) -> at::Tensor {
2671:       pybind11::gil_scoped_release no_gil;
2672:       return at::chain_matmul_out(out, matrices);
2673:     };
2674:     return wrap(dispatch_chain_matmul_out(_r.tensor(1), _r.tensorlist(0)));
2675:   }
2676:   Py_RETURN_NONE;
2677:   END_HANDLE_TH_ERRORS
2678: }
2679: 
2680: // chunk
2681: static PyObject * THPVariable_chunk(PyObject* self_, PyObject* args, PyObject* kwargs)
2682: {
2683:   HANDLE_TH_ERRORS
2684:   static PythonArgParser parser({
2685:     "chunk(Tensor input, int64_t chunks, int64_t dim=0)",
2686:   }, /*traceable=*/true);
2687: 
2688:   ParsedArgs<3> parsed_args;
2689:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2690:   if(_r.has_torch_function()) {
2691:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2692:   }
2693:   // aten::chunk(Tensor(a -> *) self, int chunks, int dim=0) -> Tensor(a)[]
2694: 
2695:   auto dispatch_chunk = [](const at::Tensor & self, int64_t chunks, int64_t dim) -> ::std::vector<at::Tensor> {
2696:     pybind11::gil_scoped_release no_gil;
2697:     return self.chunk(chunks, dim);
2698:   };
2699:   return wrap(dispatch_chunk(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
2700:   Py_RETURN_NONE;
2701:   END_HANDLE_TH_ERRORS
2702: }
2703: 
2704: \
2705: // tensor_split
2706: static PyObject * THPVariable_tensor_split(PyObject* self_, PyObject* args, PyObject* kwargs)
2707: {
2708:   HANDLE_TH_ERRORS
2709:   static PythonArgParser parser({
2710:     "tensor_split(Tensor input, SymIntArrayRef indices, int64_t dim=0)",
2711:     "tensor_split(Tensor input, Tensor tensor_indices_or_sections, int64_t dim=0)",
2712:     "tensor_split(Tensor input, SymInt sections, int64_t dim=0)",
2713:   }, /*traceable=*/true);
2714: 
2715:   ParsedArgs<3> parsed_args;
2716:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2717:   if(_r.has_torch_function()) {
2718:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2719:   }
2720:   switch (_r.idx) {
```

- EN: The main execution path in this span is carried by `THPVariable_chain_matmul`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_chain_matmul`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2721-2800

```cpp
2721:     case 0: {
2722:       // aten::tensor_split.indices(Tensor(a -> *) self, SymInt[] indices, int dim=0) -> Tensor(a)[]
2723: 
2724:       auto dispatch_tensor_split = [](const at::Tensor & self, c10::SymIntArrayRef indices, int64_t dim) -> ::std::vector<at::Tensor> {
2725:         pybind11::gil_scoped_release no_gil;
2726:         return self.tensor_split_symint(indices, dim);
2727:       };
2728:       return wrap(dispatch_tensor_split(_r.tensor(0), _r.symintlist(1), _r.toInt64(2)));
2729:     }
2730:     case 1: {
2731:       // aten::tensor_split.tensor_indices_or_sections(Tensor(a -> *) self, Tensor tensor_indices_or_sections, int dim=0) -> Tensor(a)[]
2732: 
2733:       auto dispatch_tensor_split = [](const at::Tensor & self, const at::Tensor & tensor_indices_or_sections, int64_t dim) -> ::std::vector<at::Tensor> {
2734:         pybind11::gil_scoped_release no_gil;
2735:         return self.tensor_split(tensor_indices_or_sections, dim);
2736:       };
2737:       return wrap(dispatch_tensor_split(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
2738:     }
2739:     case 2: {
2740:       // aten::tensor_split.sections(Tensor(a -> *) self, SymInt sections, int dim=0) -> Tensor(a)[]
2741: 
2742:       auto dispatch_tensor_split = [](const at::Tensor & self, c10::SymInt sections, int64_t dim) -> ::std::vector<at::Tensor> {
2743:         pybind11::gil_scoped_release no_gil;
2744:         return self.tensor_split_symint(sections, dim);
2745:       };
2746:       return wrap(dispatch_tensor_split(_r.tensor(0), _r.toSymInt(1), _r.toInt64(2)));
2747:     }
2748:   }
2749:   Py_RETURN_NONE;
2750:   END_HANDLE_TH_ERRORS
2751: }
2752: 
2753: \
2754: // clamp_min
2755: static PyObject * THPVariable_clamp_min(PyObject* self_, PyObject* args, PyObject* kwargs)
2756: {
2757:   HANDLE_TH_ERRORS
2758:   static PythonArgParser parser({
2759:     "clamp_min(Tensor input, Tensor min, *, Tensor out=None)",
2760:     "clamp_min(Tensor input, Scalar min, *, Tensor out=None)",
2761:   }, /*traceable=*/true);
2762: 
2763:   ParsedArgs<3> parsed_args;
2764:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2765:   if(_r.has_torch_function()) {
2766:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2767:   }
2768:   switch (_r.idx) {
2769:     case 0: {
2770:       if (_r.isNone(2)) {
2771:         // aten::clamp_min.Tensor(Tensor self, Tensor min) -> Tensor
2772: 
2773:         auto dispatch_clamp_min = [](const at::Tensor & self, const at::Tensor & min) -> at::Tensor {
2774:           pybind11::gil_scoped_release no_gil;
2775:           return self.clamp_min(min);
2776:         };
2777:         return wrap(dispatch_clamp_min(_r.tensor(0), _r.tensor(1)));
2778:       } else {
2779:         // aten::clamp_min.Tensor_out(Tensor self, Tensor min, *, Tensor(a!) out) -> Tensor(a!)
2780: 
2781:         auto dispatch_clamp_min_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & min) -> at::Tensor {
2782:           pybind11::gil_scoped_release no_gil;
2783:           return at::clamp_min_out(out, self, min);
2784:         };
2785:         return wrap(dispatch_clamp_min_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2786:       }
2787:     }
2788:     case 1: {
2789:       if (_r.isNone(2)) {
2790:         // aten::clamp_min(Tensor self, Scalar min) -> Tensor
2791: 
2792:         auto dispatch_clamp_min = [](const at::Tensor & self, const at::Scalar & min) -> at::Tensor {
2793:           pybind11::gil_scoped_release no_gil;
2794:           return self.clamp_min(min);
2795:         };
2796:         return wrap(dispatch_clamp_min(_r.tensor(0), _r.scalar(1)));
2797:       } else {
2798:         // aten::clamp_min.out(Tensor self, Scalar min, *, Tensor(a!) out) -> Tensor(a!)
2799: 
2800:         auto dispatch_clamp_min_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & min) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_clamp_min`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_clamp_min`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2801-2880

```cpp
2801:           pybind11::gil_scoped_release no_gil;
2802:           return at::clamp_min_out(out, self, min);
2803:         };
2804:         return wrap(dispatch_clamp_min_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2805:       }
2806:     }
2807:   }
2808:   Py_RETURN_NONE;
2809:   END_HANDLE_TH_ERRORS
2810: }
2811: 
2812: \
2813: // clamp_min_
2814: static PyObject * THPVariable_clamp_min_(PyObject* self_, PyObject* args, PyObject* kwargs)
2815: {
2816:   HANDLE_TH_ERRORS
2817:   static PythonArgParser parser({
2818:     "clamp_min_(Tensor input, Tensor min)",
2819:     "clamp_min_(Tensor input, Scalar min)",
2820:   }, /*traceable=*/true);
2821: 
2822:   ParsedArgs<2> parsed_args;
2823:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2824:   if(_r.has_torch_function()) {
2825:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2826:   }
2827:   switch (_r.idx) {
2828:     case 0: {
2829:       // aten::clamp_min_.Tensor(Tensor(a!) self, Tensor min) -> Tensor(a!)
2830: 
2831:       auto dispatch_clamp_min_ = [](at::Tensor self, const at::Tensor & min) -> at::Tensor {
2832:         pybind11::gil_scoped_release no_gil;
2833:         return self.clamp_min_(min);
2834:       };
2835:       return wrap(dispatch_clamp_min_(_r.tensor(0), _r.tensor(1)));
2836:     }
2837:     case 1: {
2838:       // aten::clamp_min_(Tensor(a!) self, Scalar min) -> Tensor(a!)
2839: 
2840:       auto dispatch_clamp_min_ = [](at::Tensor self, const at::Scalar & min) -> at::Tensor {
2841:         pybind11::gil_scoped_release no_gil;
2842:         return self.clamp_min_(min);
2843:       };
2844:       return wrap(dispatch_clamp_min_(_r.tensor(0), _r.scalar(1)));
2845:     }
2846:   }
2847:   Py_RETURN_NONE;
2848:   END_HANDLE_TH_ERRORS
2849: }
2850: 
2851: \
2852: // clip
2853: static PyObject * THPVariable_clip(PyObject* self_, PyObject* args, PyObject* kwargs)
2854: {
2855:   HANDLE_TH_ERRORS
2856:   static PythonArgParser parser({
2857:     "clip(Tensor input, Tensor? min=None, Tensor? max=None, *, Tensor out=None)",
2858:     "clip(Tensor input, Scalar? min=None, Scalar? max=None, *, Tensor out=None)",
2859:   }, /*traceable=*/true);
2860: 
2861:   ParsedArgs<4> parsed_args;
2862:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2863:   if(_r.has_torch_function()) {
2864:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2865:   }
2866:   switch (_r.idx) {
2867:     case 0: {
2868:       if (_r.isNone(3)) {
2869:         // aten::clip.Tensor(Tensor self, Tensor? min=None, Tensor? max=None) -> Tensor
2870: 
2871:         auto dispatch_clip = [](const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
2872:           pybind11::gil_scoped_release no_gil;
2873:           return self.clip(min, max);
2874:         };
2875:         return wrap(dispatch_clip(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2)));
2876:       } else {
2877:         // aten::clip.Tensor_out(Tensor self, Tensor? min=None, Tensor? max=None, *, Tensor(a!) out) -> Tensor(a!)
2878: 
2879:         auto dispatch_clip_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
2880:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `clamp_min_out`, `wrap`, `THPVariable_clamp_min_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `clamp_min_out`, `wrap`, `THPVariable_clamp_min_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-2960

```cpp
2881:           return at::clip_out(out, self, min, max);
2882:         };
2883:         return wrap(dispatch_clip_out(_r.tensor(3), _r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2)));
2884:       }
2885:     }
2886:     case 1: {
2887:       if (_r.isNone(3)) {
2888:         // aten::clip(Tensor self, Scalar? min=None, Scalar? max=None) -> Tensor
2889: 
2890:         auto dispatch_clip = [](const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
2891:           pybind11::gil_scoped_release no_gil;
2892:           return self.clip(min, max);
2893:         };
2894:         return wrap(dispatch_clip(_r.tensor(0), _r.scalarOptional(1), _r.scalarOptional(2)));
2895:       } else {
2896:         // aten::clip.out(Tensor self, Scalar? min=None, Scalar? max=None, *, Tensor(a!) out) -> Tensor(a!)
2897: 
2898:         auto dispatch_clip_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
2899:           pybind11::gil_scoped_release no_gil;
2900:           return at::clip_out(out, self, min, max);
2901:         };
2902:         return wrap(dispatch_clip_out(_r.tensor(3), _r.tensor(0), _r.scalarOptional(1), _r.scalarOptional(2)));
2903:       }
2904:     }
2905:   }
2906:   Py_RETURN_NONE;
2907:   END_HANDLE_TH_ERRORS
2908: }
2909: 
2910: \
2911: // clip_
2912: static PyObject * THPVariable_clip_(PyObject* self_, PyObject* args, PyObject* kwargs)
2913: {
2914:   HANDLE_TH_ERRORS
2915:   static PythonArgParser parser({
2916:     "clip_(Tensor input, Tensor? min=None, Tensor? max=None)",
2917:     "clip_(Tensor input, Scalar? min=None, Scalar? max=None)",
2918:   }, /*traceable=*/true);
2919: 
2920:   ParsedArgs<3> parsed_args;
2921:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2922:   if(_r.has_torch_function()) {
2923:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2924:   }
2925:   switch (_r.idx) {
2926:     case 0: {
2927:       // aten::clip_.Tensor(Tensor(a!) self, Tensor? min=None, Tensor? max=None) -> Tensor(a!)
2928: 
2929:       auto dispatch_clip_ = [](at::Tensor self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
2930:         pybind11::gil_scoped_release no_gil;
2931:         return self.clip_(min, max);
2932:       };
2933:       return wrap(dispatch_clip_(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2)));
2934:     }
2935:     case 1: {
2936:       // aten::clip_(Tensor(a!) self, Scalar? min=None, Scalar? max=None) -> Tensor(a!)
2937: 
2938:       auto dispatch_clip_ = [](at::Tensor self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
2939:         pybind11::gil_scoped_release no_gil;
2940:         return self.clip_(min, max);
2941:       };
2942:       return wrap(dispatch_clip_(_r.tensor(0), _r.scalarOptional(1), _r.scalarOptional(2)));
2943:     }
2944:   }
2945:   Py_RETURN_NONE;
2946:   END_HANDLE_TH_ERRORS
2947: }
2948: 
2949: // complex
2950: static PyObject * THPVariable_complex(PyObject* self_, PyObject* args, PyObject* kwargs)
2951: {
2952:   HANDLE_TH_ERRORS
2953:   static PythonArgParser parser({
2954:     "complex(Tensor real, Tensor imag, *, Tensor out=None)",
2955:   }, /*traceable=*/true);
2956: 
2957:   ParsedArgs<3> parsed_args;
2958:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2959:   if(_r.has_torch_function()) {
2960:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `clip_out`, `wrap`, `clip`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `clip_out`, `wrap`, `clip` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2961-3040

```cpp
2961:   }
2962:   if (_r.isNone(2)) {
2963:     // aten::complex(Tensor real, Tensor imag) -> Tensor
2964: 
2965:     auto dispatch_complex = [](const at::Tensor & real, const at::Tensor & imag) -> at::Tensor {
2966:       pybind11::gil_scoped_release no_gil;
2967:       return at::complex(real, imag);
2968:     };
2969:     return wrap(dispatch_complex(_r.tensor(0), _r.tensor(1)));
2970:   } else {
2971:     // aten::complex.out(Tensor real, Tensor imag, *, Tensor(a!) out) -> Tensor(a!)
2972: 
2973:     auto dispatch_complex_out = [](at::Tensor out, const at::Tensor & real, const at::Tensor & imag) -> at::Tensor {
2974:       pybind11::gil_scoped_release no_gil;
2975:       return at::complex_out(out, real, imag);
2976:     };
2977:     return wrap(dispatch_complex_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2978:   }
2979:   Py_RETURN_NONE;
2980:   END_HANDLE_TH_ERRORS
2981: }
2982: 
2983: // polar
2984: static PyObject * THPVariable_polar(PyObject* self_, PyObject* args, PyObject* kwargs)
2985: {
2986:   HANDLE_TH_ERRORS
2987:   static PythonArgParser parser({
2988:     "polar(Tensor abs, Tensor angle, *, Tensor out=None)",
2989:   }, /*traceable=*/true);
2990: 
2991:   ParsedArgs<3> parsed_args;
2992:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2993:   if(_r.has_torch_function()) {
2994:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2995:   }
2996:   if (_r.isNone(2)) {
2997:     // aten::polar(Tensor abs, Tensor angle) -> Tensor
2998: 
2999:     auto dispatch_polar = [](const at::Tensor & abs, const at::Tensor & angle) -> at::Tensor {
3000:       pybind11::gil_scoped_release no_gil;
3001:       return at::polar(abs, angle);
3002:     };
3003:     return wrap(dispatch_polar(_r.tensor(0), _r.tensor(1)));
3004:   } else {
3005:     // aten::polar.out(Tensor abs, Tensor angle, *, Tensor(a!) out) -> Tensor(a!)
3006: 
3007:     auto dispatch_polar_out = [](at::Tensor out, const at::Tensor & abs, const at::Tensor & angle) -> at::Tensor {
3008:       pybind11::gil_scoped_release no_gil;
3009:       return at::polar_out(out, abs, angle);
3010:     };
3011:     return wrap(dispatch_polar_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
3012:   }
3013:   Py_RETURN_NONE;
3014:   END_HANDLE_TH_ERRORS
3015: }
3016: 
3017: \
3018: // _convolution
3019: static PyObject * THPVariable__convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
3020: {
3021:   HANDLE_TH_ERRORS
3022:   static PythonArgParser parser({
3023:     "_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation, bool transposed, IntArrayRef output_padding, SymInt groups, bool benchmark, bool deterministic, bool cudnn_enabled)",
3024:     "_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation, bool transposed, SymIntArrayRef output_padding, SymInt groups, bool benchmark, bool deterministic, bool cudnn_enabled, bool allow_tf32)",
3025:   }, /*traceable=*/true);
3026: 
3027:   ParsedArgs<13> parsed_args;
3028:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3029:   if(_r.has_torch_function()) {
3030:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3031:   }
3032:   switch (_r.idx) {
3033:     case 0: {
3034:       // aten::_convolution.deprecated(Tensor input, Tensor weight, Tensor? bias, SymInt[] stride, SymInt[] padding, SymInt[] dilation, bool transposed, int[] output_padding, SymInt groups, bool benchmark, bool deterministic, bool cudnn_enabled) -> Tensor
3035: 
3036:       auto dispatch__convolution = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed, at::IntArrayRef output_padding, c10::SymInt groups, bool benchmark, bool deterministic, bool cudnn_enabled) -> at::Tensor {
3037:         pybind11::gil_scoped_release no_gil;
3038:         return at::_convolution_symint(input, weight, bias, stride, padding, dilation, transposed, output_padding, groups, benchmark, deterministic, cudnn_enabled);
3039:       };
3040:       return wrap(dispatch__convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toBool(6), _r.intlist(7), _r.toSymInt(8), _r.toBool(9), _r.toBool(10), _r.toBool(11)));
```

- EN: The main execution path in this span is carried by `complex`, `wrap`, `complex_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `complex`, `wrap`, `complex_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3041-3120

```cpp
3041:     }
3042:     case 1: {
3043:       // aten::_convolution(Tensor input, Tensor weight, Tensor? bias, SymInt[] stride, SymInt[] padding, SymInt[] dilation, bool transposed, SymInt[] output_padding, SymInt groups, bool benchmark, bool deterministic, bool cudnn_enabled, bool allow_tf32) -> Tensor
3044: 
3045:       auto dispatch__convolution = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed, c10::SymIntArrayRef output_padding, c10::SymInt groups, bool benchmark, bool deterministic, bool cudnn_enabled, bool allow_tf32) -> at::Tensor {
3046:         pybind11::gil_scoped_release no_gil;
3047:         return at::_convolution_symint(input, weight, bias, stride, padding, dilation, transposed, output_padding, groups, benchmark, deterministic, cudnn_enabled, allow_tf32);
3048:       };
3049:       return wrap(dispatch__convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toBool(6), _r.symintlist(7), _r.toSymInt(8), _r.toBool(9), _r.toBool(10), _r.toBool(11), _r.toBool(12)));
3050:     }
3051:   }
3052:   Py_RETURN_NONE;
3053:   END_HANDLE_TH_ERRORS
3054: }
3055: 
3056: // _convolution_mode
3057: static PyObject * THPVariable__convolution_mode(PyObject* self_, PyObject* args, PyObject* kwargs)
3058: {
3059:   HANDLE_TH_ERRORS
3060:   static PythonArgParser parser({
3061:     "_convolution_mode(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef stride, c10::string_view padding, SymIntArrayRef dilation, SymInt groups)",
3062:   }, /*traceable=*/true);
3063: 
3064:   ParsedArgs<7> parsed_args;
3065:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3066:   if(_r.has_torch_function()) {
3067:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3068:   }
3069:   // aten::_convolution_mode(Tensor input, Tensor weight, Tensor? bias, SymInt[] stride, str padding, SymInt[] dilation, SymInt groups) -> Tensor
3070: 
3071:   auto dispatch__convolution_mode = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::string_view padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
3072:     pybind11::gil_scoped_release no_gil;
3073:     return at::_convolution_mode_symint(input, weight, bias, stride, padding, dilation, groups);
3074:   };
3075:   return wrap(dispatch__convolution_mode(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.stringView(4), _r.symintlist(5), _r.toSymInt(6)));
3076:   Py_RETURN_NONE;
3077:   END_HANDLE_TH_ERRORS
3078: }
3079: 
3080: \
3081: // conv1d
3082: static PyObject * THPVariable_conv1d(PyObject* self_, PyObject* args, PyObject* kwargs)
3083: {
3084:   HANDLE_TH_ERRORS
3085:   static PythonArgParser parser({
3086:     "conv1d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[1] stride=1, SymIntArrayRef[1] padding=0, SymIntArrayRef[1] dilation=1, SymInt groups=1)",
3087:     "conv1d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[1] stride=1, c10::string_view padding=\"valid\", SymIntArrayRef[1] dilation=1, SymInt groups=1)",
3088:   }, /*traceable=*/false);
3089: 
3090:   ParsedArgs<7> parsed_args;
3091:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3092:   if(_r.has_torch_function()) {
3093:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3094:   }
3095:   switch (_r.idx) {
3096:     case 0: {
3097:       // aten::conv1d(Tensor input, Tensor weight, Tensor? bias=None, SymInt[1] stride=1, SymInt[1] padding=0, SymInt[1] dilation=1, SymInt groups=1) -> Tensor
3098: 
3099:       auto dispatch_conv1d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
3100:         pybind11::gil_scoped_release no_gil;
3101:         return at::conv1d_symint(input, weight, bias, stride, padding, dilation, groups);
3102:       };
3103:       return wrap(dispatch_conv1d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
3104:     }
3105:     case 1: {
3106:       // aten::conv1d.padding(Tensor input, Tensor weight, Tensor? bias=None, SymInt[1] stride=1, str padding="valid", SymInt[1] dilation=1, SymInt groups=1) -> Tensor
3107: 
3108:       auto dispatch_conv1d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::string_view padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
3109:         pybind11::gil_scoped_release no_gil;
3110:         return at::conv1d_symint(input, weight, bias, stride, padding, dilation, groups);
3111:       };
3112:       return wrap(dispatch_conv1d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.stringView(4), _r.symintlist(5), _r.toSymInt(6)));
3113:     }
3114:   }
3115:   Py_RETURN_NONE;
3116:   END_HANDLE_TH_ERRORS
3117: }
3118: 
3119: \
3120: // conv3d
```

- EN: The main execution path in this span is carried by `_convolution`, `_convolution_symint`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_convolution`, `_convolution_symint`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3121-3200

```cpp
3121: static PyObject * THPVariable_conv3d(PyObject* self_, PyObject* args, PyObject* kwargs)
3122: {
3123:   HANDLE_TH_ERRORS
3124:   static PythonArgParser parser({
3125:     "conv3d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[3] stride=1, SymIntArrayRef[3] padding=0, SymIntArrayRef[3] dilation=1, SymInt groups=1)",
3126:     "conv3d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[3] stride=1, c10::string_view padding=\"valid\", SymIntArrayRef[3] dilation=1, SymInt groups=1)",
3127:   }, /*traceable=*/false);
3128: 
3129:   ParsedArgs<7> parsed_args;
3130:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3131:   if(_r.has_torch_function()) {
3132:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3133:   }
3134:   switch (_r.idx) {
3135:     case 0: {
3136:       // aten::conv3d(Tensor input, Tensor weight, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0, SymInt[3] dilation=1, SymInt groups=1) -> Tensor
3137: 
3138:       auto dispatch_conv3d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
3139:         pybind11::gil_scoped_release no_gil;
3140:         return at::conv3d_symint(input, weight, bias, stride, padding, dilation, groups);
3141:       };
3142:       return wrap(dispatch_conv3d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
3143:     }
3144:     case 1: {
3145:       // aten::conv3d.padding(Tensor input, Tensor weight, Tensor? bias=None, SymInt[3] stride=1, str padding="valid", SymInt[3] dilation=1, SymInt groups=1) -> Tensor
3146: 
3147:       auto dispatch_conv3d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::string_view padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
3148:         pybind11::gil_scoped_release no_gil;
3149:         return at::conv3d_symint(input, weight, bias, stride, padding, dilation, groups);
3150:       };
3151:       return wrap(dispatch_conv3d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.stringView(4), _r.symintlist(5), _r.toSymInt(6)));
3152:     }
3153:   }
3154:   Py_RETURN_NONE;
3155:   END_HANDLE_TH_ERRORS
3156: }
3157: 
3158: // cos
3159: static PyObject * THPVariable_cos(PyObject* self_, PyObject* args, PyObject* kwargs)
3160: {
3161:   HANDLE_TH_ERRORS
3162:   static PythonArgParser parser({
3163:     "cos(Tensor input, *, Tensor out=None)",
3164:   }, /*traceable=*/true);
3165: 
3166:   ParsedArgs<2> parsed_args;
3167:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3168:   if(_r.has_torch_function()) {
3169:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3170:   }
3171:   if (_r.isNone(1)) {
3172:     // aten::cos(Tensor self) -> Tensor
3173: 
3174:     auto dispatch_cos = [](const at::Tensor & self) -> at::Tensor {
3175:       pybind11::gil_scoped_release no_gil;
3176:       return self.cos();
3177:     };
3178:     return wrap(dispatch_cos(_r.tensor(0)));
3179:   } else {
3180:     // aten::cos.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
3181: 
3182:     auto dispatch_cos_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
3183:       pybind11::gil_scoped_release no_gil;
3184:       return at::cos_out(out, self);
3185:     };
3186:     return wrap(dispatch_cos_out(_r.tensor(1), _r.tensor(0)));
3187:   }
3188:   Py_RETURN_NONE;
3189:   END_HANDLE_TH_ERRORS
3190: }
3191: 
3192: // cos_
3193: static PyObject * THPVariable_cos_(PyObject* self_, PyObject* args, PyObject* kwargs)
3194: {
3195:   HANDLE_TH_ERRORS
3196:   static PythonArgParser parser({
3197:     "cos_(Tensor input)",
3198:   }, /*traceable=*/true);
3199: 
3200:   ParsedArgs<1> parsed_args;
```

- EN: The main execution path in this span is carried by `THPVariable_conv3d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_conv3d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3201-3280

```cpp
3201:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3202:   if(_r.has_torch_function()) {
3203:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3204:   }
3205:   // aten::cos_(Tensor(a!) self) -> Tensor(a!)
3206: 
3207:   auto dispatch_cos_ = [](at::Tensor self) -> at::Tensor {
3208:     pybind11::gil_scoped_release no_gil;
3209:     return self.cos_();
3210:   };
3211:   return wrap(dispatch_cos_(_r.tensor(0)));
3212:   Py_RETURN_NONE;
3213:   END_HANDLE_TH_ERRORS
3214: }
3215: 
3216: \
3217: // count_nonzero
3218: static PyObject * THPVariable_count_nonzero(PyObject* self_, PyObject* args, PyObject* kwargs)
3219: {
3220:   HANDLE_TH_ERRORS
3221:   static PythonArgParser parser({
3222:     "count_nonzero(Tensor input, int64_t? dim=None)",
3223:     "count_nonzero(Tensor input, IntArrayRef dim)",
3224:   }, /*traceable=*/true);
3225: 
3226:   ParsedArgs<2> parsed_args;
3227:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3228:   if(_r.has_torch_function()) {
3229:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3230:   }
3231:   switch (_r.idx) {
3232:     case 0: {
3233:       // aten::count_nonzero(Tensor self, int? dim=None) -> Tensor
3234: 
3235:       auto dispatch_count_nonzero = [](const at::Tensor & self, ::std::optional<int64_t> dim) -> at::Tensor {
3236:         pybind11::gil_scoped_release no_gil;
3237:         return self.count_nonzero(dim);
3238:       };
3239:       return wrap(dispatch_count_nonzero(_r.tensor(0), _r.toInt64Optional(1)));
3240:     }
3241:     case 1: {
3242:       // aten::count_nonzero.dim_IntList(Tensor self, int[] dim) -> Tensor
3243: 
3244:       auto dispatch_count_nonzero = [](const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
3245:         pybind11::gil_scoped_release no_gil;
3246:         return self.count_nonzero(dim);
3247:       };
3248:       return wrap(dispatch_count_nonzero(_r.tensor(0), _r.intlist(1)));
3249:     }
3250:   }
3251:   Py_RETURN_NONE;
3252:   END_HANDLE_TH_ERRORS
3253: }
3254: 
3255: // corrcoef
3256: static PyObject * THPVariable_corrcoef(PyObject* self_, PyObject* args, PyObject* kwargs)
3257: {
3258:   HANDLE_TH_ERRORS
3259:   static PythonArgParser parser({
3260:     "corrcoef(Tensor input)",
3261:   }, /*traceable=*/true);
3262: 
3263:   ParsedArgs<1> parsed_args;
3264:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3265:   if(_r.has_torch_function()) {
3266:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3267:   }
3268:   // aten::corrcoef(Tensor self) -> Tensor
3269: 
3270:   auto dispatch_corrcoef = [](const at::Tensor & self) -> at::Tensor {
3271:     pybind11::gil_scoped_release no_gil;
3272:     return self.corrcoef();
3273:   };
3274:   return wrap(dispatch_corrcoef(_r.tensor(0)));
3275:   Py_RETURN_NONE;
3276:   END_HANDLE_TH_ERRORS
3277: }
3278: 
3279: // cudnn_convolution_transpose
3280: static PyObject * THPVariable_cudnn_convolution_transpose(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `cos_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `cos_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3281-3360

```cpp
3281: {
3282:   HANDLE_TH_ERRORS
3283:   static PythonArgParser parser({
3284:     "cudnn_convolution_transpose(Tensor input, Tensor weight, SymIntArrayRef padding, SymIntArrayRef output_padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups, bool benchmark, bool deterministic, bool allow_tf32)",
3285:   }, /*traceable=*/true);
3286: 
3287:   ParsedArgs<10> parsed_args;
3288:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3289:   if(_r.has_torch_function()) {
3290:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3291:   }
3292:   // aten::cudnn_convolution_transpose(Tensor self, Tensor weight, SymInt[] padding, SymInt[] output_padding, SymInt[] stride, SymInt[] dilation, SymInt groups, bool benchmark, bool deterministic, bool allow_tf32) -> Tensor
3293: 
3294:   auto dispatch_cudnn_convolution_transpose = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups, bool benchmark, bool deterministic, bool allow_tf32) -> at::Tensor {
3295:     pybind11::gil_scoped_release no_gil;
3296:     return at::cudnn_convolution_transpose_symint(self, weight, padding, output_padding, stride, dilation, groups, benchmark, deterministic, allow_tf32);
3297:   };
3298:   return wrap(dispatch_cudnn_convolution_transpose(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6), _r.toBool(7), _r.toBool(8), _r.toBool(9)));
3299:   Py_RETURN_NONE;
3300:   END_HANDLE_TH_ERRORS
3301: }
3302: 
3303: // cudnn_convolution_relu
3304: static PyObject * THPVariable_cudnn_convolution_relu(PyObject* self_, PyObject* args, PyObject* kwargs)
3305: {
3306:   HANDLE_TH_ERRORS
3307:   static PythonArgParser parser({
3308:     "cudnn_convolution_relu(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation, SymInt groups)",
3309:   }, /*traceable=*/true);
3310: 
3311:   ParsedArgs<7> parsed_args;
3312:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3313:   if(_r.has_torch_function()) {
3314:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3315:   }
3316:   // aten::cudnn_convolution_relu(Tensor self, Tensor weight, Tensor? bias, SymInt[] stride, SymInt[] padding, SymInt[] dilation, SymInt groups) -> Tensor
3317: 
3318:   auto dispatch_cudnn_convolution_relu = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
3319:     pybind11::gil_scoped_release no_gil;
3320:     return at::cudnn_convolution_relu_symint(self, weight, bias, stride, padding, dilation, groups);
3321:   };
3322:   return wrap(dispatch_cudnn_convolution_relu(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
3323:   Py_RETURN_NONE;
3324:   END_HANDLE_TH_ERRORS
3325: }
3326: 
3327: \
3328: // ctc_loss
3329: static PyObject * THPVariable_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
3330: {
3331:   HANDLE_TH_ERRORS
3332:   static PythonArgParser parser({
3333:     "ctc_loss(Tensor log_probs, Tensor targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t blank=0, int64_t reduction=at::Reduction::Mean, bool zero_infinity=False)",
3334:     "ctc_loss(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int64_t blank=0, int64_t reduction=at::Reduction::Mean, bool zero_infinity=False)",
3335:   }, /*traceable=*/true);
3336: 
3337:   ParsedArgs<7> parsed_args;
3338:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3339:   if(_r.has_torch_function()) {
3340:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3341:   }
3342:   switch (_r.idx) {
3343:     case 0: {
3344:       // aten::ctc_loss.IntList(Tensor log_probs, Tensor targets, int[] input_lengths, int[] target_lengths, int blank=0, int reduction=Mean, bool zero_infinity=False) -> Tensor
3345: 
3346:       auto dispatch_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, at::IntArrayRef input_lengths, at::IntArrayRef target_lengths, int64_t blank, int64_t reduction, bool zero_infinity) -> at::Tensor {
3347:         pybind11::gil_scoped_release no_gil;
3348:         return at::ctc_loss(log_probs, targets, input_lengths, target_lengths, blank, reduction, zero_infinity);
3349:       };
3350:       return wrap(dispatch_ctc_loss(_r.tensor(0), _r.tensor(1), _r.intlist(2), _r.intlist(3), _r.toInt64(4), _r.toInt64(5), _r.toBool(6)));
3351:     }
3352:     case 1: {
3353:       // aten::ctc_loss.Tensor(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int blank=0, int reduction=Mean, bool zero_infinity=False) -> Tensor
3354: 
3355:       auto dispatch_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, const at::Tensor & input_lengths, const at::Tensor & target_lengths, int64_t blank, int64_t reduction, bool zero_infinity) -> at::Tensor {
3356:         pybind11::gil_scoped_release no_gil;
3357:         return at::ctc_loss(log_probs, targets, input_lengths, target_lengths, blank, reduction, zero_infinity);
3358:       };
3359:       return wrap(dispatch_ctc_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.toInt64(4), _r.toInt64(5), _r.toBool(6)));
3360:     }
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `cudnn_convolution_transpose`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `cudnn_convolution_transpose` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3361-3440

```cpp
3361:   }
3362:   Py_RETURN_NONE;
3363:   END_HANDLE_TH_ERRORS
3364: }
3365: 
3366: \
3367: // embedding_bag
3368: static PyObject * THPVariable_embedding_bag(PyObject* self_, PyObject* args, PyObject* kwargs)
3369: {
3370:   HANDLE_TH_ERRORS
3371:   static PythonArgParser parser({
3372:     "embedding_bag(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq, int64_t mode, bool sparse, Tensor? per_sample_weights, bool include_last_offset, int64_t? padding_idx)",
3373:     "embedding_bag(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq=False, int64_t mode=0, bool sparse=False, Tensor? per_sample_weights=None, bool include_last_offset=False)",
3374:   }, /*traceable=*/true);
3375: 
3376:   ParsedArgs<9> parsed_args;
3377:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3378:   if(_r.has_torch_function()) {
3379:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3380:   }
3381:   switch (_r.idx) {
3382:     case 0: {
3383:       // aten::embedding_bag.padding_idx(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq, int mode, bool sparse, Tensor? per_sample_weights, bool include_last_offset, int? padding_idx) -> (Tensor, Tensor, Tensor, Tensor)
3384: 
3385:       auto dispatch_embedding_bag = [](const at::Tensor & weight, const at::Tensor & indices, const at::Tensor & offsets, bool scale_grad_by_freq, int64_t mode, bool sparse, const ::std::optional<at::Tensor> & per_sample_weights, bool include_last_offset, ::std::optional<int64_t> padding_idx) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
3386:         pybind11::gil_scoped_release no_gil;
3387:         return at::embedding_bag(weight, indices, offsets, scale_grad_by_freq, mode, sparse, per_sample_weights, include_last_offset, padding_idx);
3388:       };
3389:       return wrap(dispatch_embedding_bag(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3), _r.toInt64(4), _r.toBool(5), _r.optionalTensor(6), _r.toBool(7), _r.toInt64Optional(8)));
3390:     }
3391:     case 1: {
3392:       // aten::embedding_bag(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq=False, int mode=0, bool sparse=False, Tensor? per_sample_weights=None, bool include_last_offset=False) -> (Tensor, Tensor, Tensor, Tensor)
3393: 
3394:       auto dispatch_embedding_bag = [](const at::Tensor & weight, const at::Tensor & indices, const at::Tensor & offsets, bool scale_grad_by_freq, int64_t mode, bool sparse, const ::std::optional<at::Tensor> & per_sample_weights, bool include_last_offset) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
3395:         pybind11::gil_scoped_release no_gil;
3396:         return at::embedding_bag(weight, indices, offsets, scale_grad_by_freq, mode, sparse, per_sample_weights, include_last_offset);
3397:       };
3398:       return wrap(dispatch_embedding_bag(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3), _r.toInt64(4), _r.toBool(5), _r.optionalTensor(6), _r.toBool(7)));
3399:     }
3400:   }
3401:   Py_RETURN_NONE;
3402:   END_HANDLE_TH_ERRORS
3403: }
3404: 
3405: // _embedding_bag
3406: static PyObject * THPVariable__embedding_bag(PyObject* self_, PyObject* args, PyObject* kwargs)
3407: {
3408:   HANDLE_TH_ERRORS
3409:   static PythonArgParser parser({
3410:     "_embedding_bag(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq=False, int64_t mode=0, bool sparse=False, Tensor? per_sample_weights=None, bool include_last_offset=False, int64_t padding_idx=-1)",
3411:   }, /*traceable=*/true);
3412: 
3413:   ParsedArgs<9> parsed_args;
3414:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3415:   if(_r.has_torch_function()) {
3416:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3417:   }
3418:   // aten::_embedding_bag(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq=False, int mode=0, bool sparse=False, Tensor? per_sample_weights=None, bool include_last_offset=False, int padding_idx=-1) -> (Tensor, Tensor, Tensor, Tensor)
3419: 
3420:   auto dispatch__embedding_bag = [](const at::Tensor & weight, const at::Tensor & indices, const at::Tensor & offsets, bool scale_grad_by_freq, int64_t mode, bool sparse, const ::std::optional<at::Tensor> & per_sample_weights, bool include_last_offset, int64_t padding_idx) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
3421:     pybind11::gil_scoped_release no_gil;
3422:     return at::_embedding_bag(weight, indices, offsets, scale_grad_by_freq, mode, sparse, per_sample_weights, include_last_offset, padding_idx);
3423:   };
3424:   return wrap(dispatch__embedding_bag(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3), _r.toInt64(4), _r.toBool(5), _r.optionalTensor(6), _r.toBool(7), _r.toInt64(8)));
3425:   Py_RETURN_NONE;
3426:   END_HANDLE_TH_ERRORS
3427: }
3428: 
3429: \
3430: // empty
3431: static PyObject * THPVariable_empty(PyObject* self_, PyObject* args, PyObject* kwargs)
3432: {
3433:   HANDLE_TH_ERRORS
3434:   static PythonArgParser parser({
3435:     "empty(IntArrayRef size, *, DimnameList? names, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3436:     "empty(SymIntArrayRef size, *, MemoryFormat? memory_format=None, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3437:   }, /*traceable=*/true);
3438: 
3439:   ParsedArgs<8> parsed_args;
3440:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `THPVariable_embedding_bag`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `THPVariable_embedding_bag`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 3441-3520

```cpp
3441:   if(_r.has_torch_function()) {
3442:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3443:   }
3444:   switch (_r.idx) {
3445:     case 0: {
3446:       // aten::empty.names(int[] size, *, Dimname[]? names, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
3447:       auto __names = _r.toDimnameListOptional(1);
3448:       ::std::optional<DimnameList> names = __names ? ::std::make_optional(DimnameList(__names.value())) : ::std::nullopt;
3449:       const auto options = TensorOptions()
3450:           .dtype(_r.scalartypeOptional(3))
3451:           .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
3452:           .layout(_r.layoutOptional(4))
3453:           .requires_grad(_r.toBool(7))
3454:           .pinned_memory(_r.toBool(6));
3455:       torch::utils::maybe_initialize_device(options);
3456: 
3457:       auto dispatch_empty = [](at::IntArrayRef size, ::std::optional<at::DimnameList> names, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
3458:         pybind11::gil_scoped_release no_gil;
3459:         return torch::empty(size, names, options, memory_format);
3460:       };
3461:       return wrap(dispatch_empty(_r.intlist(0), names, options, _r.memoryformatOptional(2)));
3462:     }
3463:     case 1: {
3464:       if (_r.isNone(2)) {
3465:         // aten::empty.memory_format(SymInt[] size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
3466:         const auto options = TensorOptions()
3467:             .dtype(_r.scalartypeOptional(3))
3468:             .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
3469:             .layout(_r.layoutOptional(4))
3470:             .requires_grad(_r.toBool(7))
3471:             .pinned_memory(_r.toBool(6));
3472:         torch::utils::maybe_initialize_device(options);
3473: 
3474:         auto dispatch_empty = [](c10::SymIntArrayRef size, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
3475:           pybind11::gil_scoped_release no_gil;
3476:           return torch::empty_symint(size, options, memory_format);
3477:         };
3478:         return wrap(dispatch_empty(_r.symintlist(0), options, _r.memoryformatOptional(1)));
3479:       } else {
3480:         // aten::empty.out(SymInt[] size, *, MemoryFormat? memory_format=None, Tensor(a!) out) -> Tensor(a!)
3481:         check_out_type_matches(_r.tensor(2), _r.scalartypeOptional(3),
3482:                                _r.isNone(3), _r.layoutOptional(4),
3483:                                _r.deviceWithDefault(5, torch::tensors::get_default_device()), _r.isNone(5));
3484: 
3485:         auto dispatch_empty_out = [](at::Tensor out, c10::SymIntArrayRef size, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
3486:           pybind11::gil_scoped_release no_gil;
3487:           return at::empty_symint_out(out, size, memory_format);
3488:         };
3489:         return wrap(dispatch_empty_out(_r.tensor(2), _r.symintlist(0), _r.memoryformatOptional(1)).set_requires_grad(_r.toBool(7)));
3490:       }
3491:     }
3492:   }
3493:   Py_RETURN_NONE;
3494:   END_HANDLE_TH_ERRORS
3495: }
3496: 
3497: // empty_like
3498: static PyObject * THPVariable_empty_like(PyObject* self_, PyObject* args, PyObject* kwargs)
3499: {
3500:   HANDLE_TH_ERRORS
3501:   static PythonArgParser parser({
3502:     "empty_like(Tensor input, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3503:   }, /*traceable=*/true);
3504: 
3505:   ParsedArgs<7> parsed_args;
3506:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3507:   if(_r.has_torch_function()) {
3508:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3509:   }
3510:   // aten::empty_like(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
3511:   auto self = _r.tensor(0);
3512:   const auto options = TensorOptions()
3513:       .dtype(_r.scalartypeOptional(2))
3514:       .device(_r.deviceOptional(4))
3515:       .layout(_r.layoutOptional(3))
3516:       .requires_grad(_r.toBool(6))
3517:       .pinned_memory(_r.toBool(5));
3518:   torch::utils::maybe_initialize_device(options);
3519: 
3520:   auto dispatch_empty_like = [](const at::Tensor & self, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `make_optional`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `make_optional`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 3521-3600

```cpp
3521:     pybind11::gil_scoped_release no_gil;
3522:     return torch::empty_like(self, options, memory_format);
3523:   };
3524:   return wrap(dispatch_empty_like(self, options, _r.memoryformatOptional(1)));
3525:   Py_RETURN_NONE;
3526:   END_HANDLE_TH_ERRORS
3527: }
3528: 
3529: // erf
3530: static PyObject * THPVariable_erf(PyObject* self_, PyObject* args, PyObject* kwargs)
3531: {
3532:   HANDLE_TH_ERRORS
3533:   static PythonArgParser parser({
3534:     "erf(Tensor input, *, Tensor out=None)",
3535:   }, /*traceable=*/true);
3536: 
3537:   ParsedArgs<2> parsed_args;
3538:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3539:   if(_r.has_torch_function()) {
3540:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3541:   }
3542:   if (_r.isNone(1)) {
3543:     // aten::erf(Tensor self) -> Tensor
3544: 
3545:     auto dispatch_erf = [](const at::Tensor & self) -> at::Tensor {
3546:       pybind11::gil_scoped_release no_gil;
3547:       return self.erf();
3548:     };
3549:     return wrap(dispatch_erf(_r.tensor(0)));
3550:   } else {
3551:     // aten::erf.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
3552: 
3553:     auto dispatch_erf_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
3554:       pybind11::gil_scoped_release no_gil;
3555:       return at::erf_out(out, self);
3556:     };
3557:     return wrap(dispatch_erf_out(_r.tensor(1), _r.tensor(0)));
3558:   }
3559:   Py_RETURN_NONE;
3560:   END_HANDLE_TH_ERRORS
3561: }
3562: 
3563: // erf_
3564: static PyObject * THPVariable_erf_(PyObject* self_, PyObject* args, PyObject* kwargs)
3565: {
3566:   HANDLE_TH_ERRORS
3567:   static PythonArgParser parser({
3568:     "erf_(Tensor input)",
3569:   }, /*traceable=*/true);
3570: 
3571:   ParsedArgs<1> parsed_args;
3572:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3573:   if(_r.has_torch_function()) {
3574:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3575:   }
3576:   // aten::erf_(Tensor(a!) self) -> Tensor(a!)
3577: 
3578:   auto dispatch_erf_ = [](at::Tensor self) -> at::Tensor {
3579:     pybind11::gil_scoped_release no_gil;
3580:     return self.erf_();
3581:   };
3582:   return wrap(dispatch_erf_(_r.tensor(0)));
3583:   Py_RETURN_NONE;
3584:   END_HANDLE_TH_ERRORS
3585: }
3586: 
3587: // erfc
3588: static PyObject * THPVariable_erfc(PyObject* self_, PyObject* args, PyObject* kwargs)
3589: {
3590:   HANDLE_TH_ERRORS
3591:   static PythonArgParser parser({
3592:     "erfc(Tensor input, *, Tensor out=None)",
3593:   }, /*traceable=*/true);
3594: 
3595:   ParsedArgs<2> parsed_args;
3596:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3597:   if(_r.has_torch_function()) {
3598:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3599:   }
3600:   if (_r.isNone(1)) {
```

- EN: The main execution path in this span is carried by `empty_like`, `wrap`, `THPVariable_erf`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `empty_like`, `wrap`, `THPVariable_erf` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601:     // aten::erfc(Tensor self) -> Tensor
3602: 
3603:     auto dispatch_erfc = [](const at::Tensor & self) -> at::Tensor {
3604:       pybind11::gil_scoped_release no_gil;
3605:       return self.erfc();
3606:     };
3607:     return wrap(dispatch_erfc(_r.tensor(0)));
3608:   } else {
3609:     // aten::erfc.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
3610: 
3611:     auto dispatch_erfc_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
3612:       pybind11::gil_scoped_release no_gil;
3613:       return at::erfc_out(out, self);
3614:     };
3615:     return wrap(dispatch_erfc_out(_r.tensor(1), _r.tensor(0)));
3616:   }
3617:   Py_RETURN_NONE;
3618:   END_HANDLE_TH_ERRORS
3619: }
3620: 
3621: // erfc_
3622: static PyObject * THPVariable_erfc_(PyObject* self_, PyObject* args, PyObject* kwargs)
3623: {
3624:   HANDLE_TH_ERRORS
3625:   static PythonArgParser parser({
3626:     "erfc_(Tensor input)",
3627:   }, /*traceable=*/true);
3628: 
3629:   ParsedArgs<1> parsed_args;
3630:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3631:   if(_r.has_torch_function()) {
3632:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3633:   }
3634:   // aten::erfc_(Tensor(a!) self) -> Tensor(a!)
3635: 
3636:   auto dispatch_erfc_ = [](at::Tensor self) -> at::Tensor {
3637:     pybind11::gil_scoped_release no_gil;
3638:     return self.erfc_();
3639:   };
3640:   return wrap(dispatch_erfc_(_r.tensor(0)));
3641:   Py_RETURN_NONE;
3642:   END_HANDLE_TH_ERRORS
3643: }
3644: 
3645: // exp
3646: static PyObject * THPVariable_exp(PyObject* self_, PyObject* args, PyObject* kwargs)
3647: {
3648:   HANDLE_TH_ERRORS
3649:   static PythonArgParser parser({
3650:     "exp(Tensor input, *, Tensor out=None)",
3651:   }, /*traceable=*/true);
3652: 
3653:   ParsedArgs<2> parsed_args;
3654:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3655:   if(_r.has_torch_function()) {
3656:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3657:   }
3658:   if (_r.isNone(1)) {
3659:     // aten::exp(Tensor self) -> Tensor
3660: 
3661:     auto dispatch_exp = [](const at::Tensor & self) -> at::Tensor {
3662:       pybind11::gil_scoped_release no_gil;
3663:       return self.exp();
3664:     };
3665:     return wrap(dispatch_exp(_r.tensor(0)));
3666:   } else {
3667:     // aten::exp.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
3668: 
3669:     auto dispatch_exp_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
3670:       pybind11::gil_scoped_release no_gil;
3671:       return at::exp_out(out, self);
3672:     };
3673:     return wrap(dispatch_exp_out(_r.tensor(1), _r.tensor(0)));
3674:   }
3675:   Py_RETURN_NONE;
3676:   END_HANDLE_TH_ERRORS
3677: }
3678: 
3679: // exp_
3680: static PyObject * THPVariable_exp_(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `erfc`, `wrap`, `erfc_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `erfc`, `wrap`, `erfc_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3681-3760

```cpp
3681: {
3682:   HANDLE_TH_ERRORS
3683:   static PythonArgParser parser({
3684:     "exp_(Tensor input)",
3685:   }, /*traceable=*/true);
3686: 
3687:   ParsedArgs<1> parsed_args;
3688:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3689:   if(_r.has_torch_function()) {
3690:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3691:   }
3692:   // aten::exp_(Tensor(a!) self) -> Tensor(a!)
3693: 
3694:   auto dispatch_exp_ = [](at::Tensor self) -> at::Tensor {
3695:     pybind11::gil_scoped_release no_gil;
3696:     return self.exp_();
3697:   };
3698:   return wrap(dispatch_exp_(_r.tensor(0)));
3699:   Py_RETURN_NONE;
3700:   END_HANDLE_TH_ERRORS
3701: }
3702: 
3703: \
3704: // unflatten
3705: static PyObject * THPVariable_unflatten(PyObject* self_, PyObject* args, PyObject* kwargs)
3706: {
3707:   HANDLE_TH_ERRORS
3708:   static PythonArgParser parser({
3709:     "unflatten(Tensor input, Dimname dim, SymIntArrayRef sizes, DimnameList names)",
3710:     "unflatten(Tensor input, int64_t dim, SymIntArrayRef sizes)",
3711:   }, /*traceable=*/true);
3712: 
3713:   ParsedArgs<4> parsed_args;
3714:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3715:   if(_r.has_torch_function()) {
3716:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3717:   }
3718:   switch (_r.idx) {
3719:     case 0: {
3720:       // aten::unflatten.Dimname(Tensor(a) self, Dimname dim, SymInt[] sizes, Dimname[] names) -> Tensor(a)
3721: 
3722:       auto dispatch_unflatten = [](const at::Tensor & self, at::Dimname dim, c10::SymIntArrayRef sizes, at::DimnameList names) -> at::Tensor {
3723:         pybind11::gil_scoped_release no_gil;
3724:         return self.unflatten_symint(dim, sizes, names);
3725:       };
3726:       return wrap(dispatch_unflatten(_r.tensor(0), _r.dimname(1), _r.symintlist(2), _r.dimnamelist(3)));
3727:     }
3728:     case 1: {
3729:       // aten::unflatten.int(Tensor(a) self, int dim, SymInt[] sizes) -> Tensor(a)
3730: 
3731:       auto dispatch_unflatten = [](const at::Tensor & self, int64_t dim, c10::SymIntArrayRef sizes) -> at::Tensor {
3732:         pybind11::gil_scoped_release no_gil;
3733:         return self.unflatten_symint(dim, sizes);
3734:       };
3735:       return wrap(dispatch_unflatten(_r.tensor(0), _r.toInt64(1), _r.symintlist(2)));
3736:     }
3737:   }
3738:   Py_RETURN_NONE;
3739:   END_HANDLE_TH_ERRORS
3740: }
3741: 
3742: // floor
3743: static PyObject * THPVariable_floor(PyObject* self_, PyObject* args, PyObject* kwargs)
3744: {
3745:   HANDLE_TH_ERRORS
3746:   static PythonArgParser parser({
3747:     "floor(Tensor input, *, Tensor out=None)",
3748:   }, /*traceable=*/true);
3749: 
3750:   ParsedArgs<2> parsed_args;
3751:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3752:   if(_r.has_torch_function()) {
3753:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3754:   }
3755:   if (_r.isNone(1)) {
3756:     // aten::floor(Tensor self) -> Tensor
3757: 
3758:     auto dispatch_floor = [](const at::Tensor & self) -> at::Tensor {
3759:       pybind11::gil_scoped_release no_gil;
3760:       return self.floor();
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `exp_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `exp_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3761-3840

```cpp
3761:     };
3762:     return wrap(dispatch_floor(_r.tensor(0)));
3763:   } else {
3764:     // aten::floor.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
3765: 
3766:     auto dispatch_floor_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
3767:       pybind11::gil_scoped_release no_gil;
3768:       return at::floor_out(out, self);
3769:     };
3770:     return wrap(dispatch_floor_out(_r.tensor(1), _r.tensor(0)));
3771:   }
3772:   Py_RETURN_NONE;
3773:   END_HANDLE_TH_ERRORS
3774: }
3775: 
3776: // floor_
3777: static PyObject * THPVariable_floor_(PyObject* self_, PyObject* args, PyObject* kwargs)
3778: {
3779:   HANDLE_TH_ERRORS
3780:   static PythonArgParser parser({
3781:     "floor_(Tensor input)",
3782:   }, /*traceable=*/true);
3783: 
3784:   ParsedArgs<1> parsed_args;
3785:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3786:   if(_r.has_torch_function()) {
3787:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3788:   }
3789:   // aten::floor_(Tensor(a!) self) -> Tensor(a!)
3790: 
3791:   auto dispatch_floor_ = [](at::Tensor self) -> at::Tensor {
3792:     pybind11::gil_scoped_release no_gil;
3793:     return self.floor_();
3794:   };
3795:   return wrap(dispatch_floor_(_r.tensor(0)));
3796:   Py_RETURN_NONE;
3797:   END_HANDLE_TH_ERRORS
3798: }
3799: 
3800: \
3801: // floor_divide
3802: static PyObject * THPVariable_floor_divide(PyObject* self_, PyObject* args, PyObject* kwargs)
3803: {
3804:   HANDLE_TH_ERRORS
3805:   static PythonArgParser parser({
3806:     "floor_divide(Tensor input, Tensor other, *, Tensor out=None)",
3807:     "floor_divide(Tensor input, Scalar other)",
3808:   }, /*traceable=*/true);
3809: 
3810:   ParsedArgs<3> parsed_args;
3811:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3812:   if(_r.has_torch_function()) {
3813:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3814:   }
3815:   switch (_r.idx) {
3816:     case 0: {
3817:       if (_r.isNone(2)) {
3818:         // aten::floor_divide(Tensor self, Tensor other) -> Tensor
3819: 
3820:         auto dispatch_floor_divide = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3821:           pybind11::gil_scoped_release no_gil;
3822:           return self.floor_divide(other);
3823:         };
3824:         return wrap(dispatch_floor_divide(_r.tensor(0), _r.tensor(1)));
3825:       } else {
3826:         // aten::floor_divide.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
3827: 
3828:         auto dispatch_floor_divide_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3829:           pybind11::gil_scoped_release no_gil;
3830:           return at::floor_divide_out(out, self, other);
3831:         };
3832:         return wrap(dispatch_floor_divide_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
3833:       }
3834:     }
3835:     case 1: {
3836:       // aten::floor_divide.Scalar(Tensor self, Scalar other) -> Tensor
3837: 
3838:       auto dispatch_floor_divide = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
3839:         pybind11::gil_scoped_release no_gil;
3840:         return self.floor_divide(other);
```

- EN: The main execution path in this span is carried by `wrap`, `floor_out`, `THPVariable_floor_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `floor_out`, `THPVariable_floor_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841:       };
3842:       return wrap(dispatch_floor_divide(_r.tensor(0), _r.scalar(1)));
3843:     }
3844:   }
3845:   Py_RETURN_NONE;
3846:   END_HANDLE_TH_ERRORS
3847: }
3848: 
3849: // frac
3850: static PyObject * THPVariable_frac(PyObject* self_, PyObject* args, PyObject* kwargs)
3851: {
3852:   HANDLE_TH_ERRORS
3853:   static PythonArgParser parser({
3854:     "frac(Tensor input, *, Tensor out=None)",
3855:   }, /*traceable=*/true);
3856: 
3857:   ParsedArgs<2> parsed_args;
3858:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3859:   if(_r.has_torch_function()) {
3860:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3861:   }
3862:   if (_r.isNone(1)) {
3863:     // aten::frac(Tensor self) -> Tensor
3864: 
3865:     auto dispatch_frac = [](const at::Tensor & self) -> at::Tensor {
3866:       pybind11::gil_scoped_release no_gil;
3867:       return self.frac();
3868:     };
3869:     return wrap(dispatch_frac(_r.tensor(0)));
3870:   } else {
3871:     // aten::frac.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
3872: 
3873:     auto dispatch_frac_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
3874:       pybind11::gil_scoped_release no_gil;
3875:       return at::frac_out(out, self);
3876:     };
3877:     return wrap(dispatch_frac_out(_r.tensor(1), _r.tensor(0)));
3878:   }
3879:   Py_RETURN_NONE;
3880:   END_HANDLE_TH_ERRORS
3881: }
3882: 
3883: // frac_
3884: static PyObject * THPVariable_frac_(PyObject* self_, PyObject* args, PyObject* kwargs)
3885: {
3886:   HANDLE_TH_ERRORS
3887:   static PythonArgParser parser({
3888:     "frac_(Tensor input)",
3889:   }, /*traceable=*/true);
3890: 
3891:   ParsedArgs<1> parsed_args;
3892:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3893:   if(_r.has_torch_function()) {
3894:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3895:   }
3896:   // aten::frac_(Tensor(a!) self) -> Tensor(a!)
3897: 
3898:   auto dispatch_frac_ = [](at::Tensor self) -> at::Tensor {
3899:     pybind11::gil_scoped_release no_gil;
3900:     return self.frac_();
3901:   };
3902:   return wrap(dispatch_frac_(_r.tensor(0)));
3903:   Py_RETURN_NONE;
3904:   END_HANDLE_TH_ERRORS
3905: }
3906: 
3907: // full_like
3908: static PyObject * THPVariable_full_like(PyObject* self_, PyObject* args, PyObject* kwargs)
3909: {
3910:   HANDLE_TH_ERRORS
3911:   static PythonArgParser parser({
3912:     "full_like(Tensor input, Scalar fill_value, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3913:   }, /*traceable=*/true);
3914: 
3915:   ParsedArgs<8> parsed_args;
3916:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3917:   if(_r.has_torch_function()) {
3918:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3919:   }
3920:   // aten::full_like(Tensor self, Scalar fill_value, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_frac`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_frac`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3921-4000

```cpp
3921:   auto self = _r.tensor(0);
3922:   const auto options = TensorOptions()
3923:       .dtype(_r.scalartypeOptional(3))
3924:       .device(_r.deviceOptional(5))
3925:       .layout(_r.layoutOptional(4))
3926:       .requires_grad(_r.toBool(7))
3927:       .pinned_memory(_r.toBool(6));
3928:   torch::utils::maybe_initialize_device(options);
3929: 
3930:   auto dispatch_full_like = [](const at::Tensor & self, const at::Scalar & fill_value, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
3931:     pybind11::gil_scoped_release no_gil;
3932:     return torch::full_like(self, fill_value, options, memory_format);
3933:   };
3934:   return wrap(dispatch_full_like(self, _r.scalar(1), options, _r.memoryformatOptional(2)));
3935:   Py_RETURN_NONE;
3936:   END_HANDLE_TH_ERRORS
3937: }
3938: 
3939: // grid_sampler
3940: static PyObject * THPVariable_grid_sampler(PyObject* self_, PyObject* args, PyObject* kwargs)
3941: {
3942:   HANDLE_TH_ERRORS
3943:   static PythonArgParser parser({
3944:     "grid_sampler(Tensor input, Tensor grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners)",
3945:   }, /*traceable=*/true);
3946: 
3947:   ParsedArgs<5> parsed_args;
3948:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3949:   if(_r.has_torch_function()) {
3950:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3951:   }
3952:   // aten::grid_sampler(Tensor input, Tensor grid, int interpolation_mode, int padding_mode, bool align_corners) -> Tensor
3953: 
3954:   auto dispatch_grid_sampler = [](const at::Tensor & input, const at::Tensor & grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners) -> at::Tensor {
3955:     pybind11::gil_scoped_release no_gil;
3956:     return at::grid_sampler(input, grid, interpolation_mode, padding_mode, align_corners);
3957:   };
3958:   return wrap(dispatch_grid_sampler(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toInt64(3), _r.toBool(4)));
3959:   Py_RETURN_NONE;
3960:   END_HANDLE_TH_ERRORS
3961: }
3962: 
3963: // _grid_sampler_2d_cpu_fallback
3964: static PyObject * THPVariable__grid_sampler_2d_cpu_fallback(PyObject* self_, PyObject* args, PyObject* kwargs)
3965: {
3966:   HANDLE_TH_ERRORS
3967:   static PythonArgParser parser({
3968:     "_grid_sampler_2d_cpu_fallback(Tensor input, Tensor grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners)",
3969:   }, /*traceable=*/true);
3970: 
3971:   ParsedArgs<5> parsed_args;
3972:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3973:   if(_r.has_torch_function()) {
3974:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3975:   }
3976:   // aten::_grid_sampler_2d_cpu_fallback(Tensor input, Tensor grid, int interpolation_mode, int padding_mode, bool align_corners) -> Tensor
3977: 
3978:   auto dispatch__grid_sampler_2d_cpu_fallback = [](const at::Tensor & input, const at::Tensor & grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners) -> at::Tensor {
3979:     pybind11::gil_scoped_release no_gil;
3980:     return at::_grid_sampler_2d_cpu_fallback(input, grid, interpolation_mode, padding_mode, align_corners);
3981:   };
3982:   return wrap(dispatch__grid_sampler_2d_cpu_fallback(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toInt64(3), _r.toBool(4)));
3983:   Py_RETURN_NONE;
3984:   END_HANDLE_TH_ERRORS
3985: }
3986: 
3987: // grid_sampler_3d
3988: static PyObject * THPVariable_grid_sampler_3d(PyObject* self_, PyObject* args, PyObject* kwargs)
3989: {
3990:   HANDLE_TH_ERRORS
3991:   static PythonArgParser parser({
3992:     "grid_sampler_3d(Tensor input, Tensor grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners)",
3993:   }, /*traceable=*/true);
3994: 
3995:   ParsedArgs<5> parsed_args;
3996:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3997:   if(_r.has_torch_function()) {
3998:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3999:   }
4000:   // aten::grid_sampler_3d(Tensor input, Tensor grid, int interpolation_mode, int padding_mode, bool align_corners) -> Tensor
```

- EN: The main execution path in this span is carried by `TensorOptions`, `maybe_initialize_device`, `full_like`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TensorOptions`, `maybe_initialize_device`, `full_like` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4001-4080

```cpp
4001: 
4002:   auto dispatch_grid_sampler_3d = [](const at::Tensor & input, const at::Tensor & grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners) -> at::Tensor {
4003:     pybind11::gil_scoped_release no_gil;
4004:     return at::grid_sampler_3d(input, grid, interpolation_mode, padding_mode, align_corners);
4005:   };
4006:   return wrap(dispatch_grid_sampler_3d(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toInt64(3), _r.toBool(4)));
4007:   Py_RETURN_NONE;
4008:   END_HANDLE_TH_ERRORS
4009: }
4010: 
4011: // _fft_c2r
4012: static PyObject * THPVariable__fft_c2r(PyObject* self_, PyObject* args, PyObject* kwargs)
4013: {
4014:   HANDLE_TH_ERRORS
4015:   static PythonArgParser parser({
4016:     "_fft_c2r(Tensor input, IntArrayRef dim, int64_t normalization, SymInt last_dim_size, *, Tensor out=None)",
4017:   }, /*traceable=*/true);
4018: 
4019:   ParsedArgs<5> parsed_args;
4020:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4021:   if(_r.has_torch_function()) {
4022:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4023:   }
4024:   if (_r.isNone(4)) {
4025:     // aten::_fft_c2r(Tensor self, int[] dim, int normalization, SymInt last_dim_size) -> Tensor
4026: 
4027:     auto dispatch__fft_c2r = [](const at::Tensor & self, at::IntArrayRef dim, int64_t normalization, c10::SymInt last_dim_size) -> at::Tensor {
4028:       pybind11::gil_scoped_release no_gil;
4029:       return at::_fft_c2r_symint(self, dim, normalization, last_dim_size);
4030:     };
4031:     return wrap(dispatch__fft_c2r(_r.tensor(0), _r.intlist(1), _r.toInt64(2), _r.toSymInt(3)));
4032:   } else {
4033:     // aten::_fft_c2r.out(Tensor self, int[] dim, int normalization, SymInt last_dim_size, *, Tensor(a!) out) -> Tensor(a!)
4034: 
4035:     auto dispatch__fft_c2r_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, int64_t normalization, c10::SymInt last_dim_size) -> at::Tensor {
4036:       pybind11::gil_scoped_release no_gil;
4037:       return at::_fft_c2r_symint_out(out, self, dim, normalization, last_dim_size);
4038:     };
4039:     return wrap(dispatch__fft_c2r_out(_r.tensor(4), _r.tensor(0), _r.intlist(1), _r.toInt64(2), _r.toSymInt(3)));
4040:   }
4041:   Py_RETURN_NONE;
4042:   END_HANDLE_TH_ERRORS
4043: }
4044: 
4045: // _cufft_get_plan_cache_max_size
4046: static PyObject * THPVariable__cufft_get_plan_cache_max_size(PyObject* self_, PyObject* args, PyObject* kwargs)
4047: {
4048:   HANDLE_TH_ERRORS
4049:   static PythonArgParser parser({
4050:     "_cufft_get_plan_cache_max_size(DeviceIndex device_index)",
4051:   }, /*traceable=*/false);
4052: 
4053:   ParsedArgs<1> parsed_args;
4054:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4055:   if(_r.has_torch_function()) {
4056:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4057:   }
4058:   // aten::_cufft_get_plan_cache_max_size(DeviceIndex device_index) -> int
4059: 
4060:   auto dispatch__cufft_get_plan_cache_max_size = [](at::DeviceIndex device_index) -> int64_t {
4061:     pybind11::gil_scoped_release no_gil;
4062:     return at::_cufft_get_plan_cache_max_size(device_index);
4063:   };
4064:   return wrap(dispatch__cufft_get_plan_cache_max_size(_r.toInt64(0)));
4065:   Py_RETURN_NONE;
4066:   END_HANDLE_TH_ERRORS
4067: }
4068: 
4069: // _cufft_clear_plan_cache
4070: static PyObject * THPVariable__cufft_clear_plan_cache(PyObject* self_, PyObject* args, PyObject* kwargs)
4071: {
4072:   HANDLE_TH_ERRORS
4073:   static PythonArgParser parser({
4074:     "_cufft_clear_plan_cache(DeviceIndex device_index)",
4075:   }, /*traceable=*/false);
4076: 
4077:   ParsedArgs<1> parsed_args;
4078:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4079:   if(_r.has_torch_function()) {
4080:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `grid_sampler_3d`, `wrap`, `THPVariable__fft_c2r`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `grid_sampler_3d`, `wrap`, `THPVariable__fft_c2r` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4081-4160

```cpp
4081:   }
4082:   // aten::_cufft_clear_plan_cache(DeviceIndex device_index) -> ()
4083: 
4084:   auto dispatch__cufft_clear_plan_cache = [](at::DeviceIndex device_index) -> void {
4085:     pybind11::gil_scoped_release no_gil;
4086:     at::_cufft_clear_plan_cache(device_index);
4087:   };
4088:   dispatch__cufft_clear_plan_cache(_r.toInt64(0));
4089:   Py_RETURN_NONE;
4090:   Py_RETURN_NONE;
4091:   END_HANDLE_TH_ERRORS
4092: }
4093: 
4094: // _unsafe_masked_index_put_accumulate
4095: static PyObject * THPVariable__unsafe_masked_index_put_accumulate(PyObject* self_, PyObject* args, PyObject* kwargs)
4096: {
4097:   HANDLE_TH_ERRORS
4098:   static PythonArgParser parser({
4099:     "_unsafe_masked_index_put_accumulate(Tensor input, Tensor mask, c10::List<::std::optional<Tensor>> indices, Tensor values)",
4100:   }, /*traceable=*/true);
4101: 
4102:   ParsedArgs<4> parsed_args;
4103:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4104:   if(_r.has_torch_function()) {
4105:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4106:   }
4107:   // aten::_unsafe_masked_index_put_accumulate(Tensor self, Tensor mask, Tensor?[] indices, Tensor values) -> Tensor
4108: 
4109:   auto dispatch__unsafe_masked_index_put_accumulate = [](const at::Tensor & self, const at::Tensor & mask, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values) -> at::Tensor {
4110:     pybind11::gil_scoped_release no_gil;
4111:     return at::_unsafe_masked_index_put_accumulate(self, mask, indices, values);
4112:   };
4113:   return wrap(dispatch__unsafe_masked_index_put_accumulate(_r.tensor(0), _r.tensor(1), _r.list_of_optional_tensors(2), _r.tensor(3)));
4114:   Py_RETURN_NONE;
4115:   END_HANDLE_TH_ERRORS
4116: }
4117: 
4118: // _index_put_impl_
4119: static PyObject * THPVariable__index_put_impl_(PyObject* self_, PyObject* args, PyObject* kwargs)
4120: {
4121:   HANDLE_TH_ERRORS
4122:   static PythonArgParser parser({
4123:     "_index_put_impl_(Tensor input, c10::List<::std::optional<Tensor>> indices, Tensor values, bool accumulate=False, bool unsafe=False)",
4124:   }, /*traceable=*/true);
4125: 
4126:   ParsedArgs<5> parsed_args;
4127:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4128:   if(_r.has_torch_function()) {
4129:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4130:   }
4131:   // aten::_index_put_impl_(Tensor(a!) self, Tensor?[] indices, Tensor values, bool accumulate=False, bool unsafe=False) -> Tensor(a!)
4132: 
4133:   auto dispatch__index_put_impl_ = [](at::Tensor self, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values, bool accumulate, bool unsafe) -> at::Tensor {
4134:     pybind11::gil_scoped_release no_gil;
4135:     return at::_index_put_impl_(self, indices, values, accumulate, unsafe);
4136:   };
4137:   return wrap(dispatch__index_put_impl_(_r.tensor(0), _r.list_of_optional_tensors(1), _r.tensor(2), _r.toBool(3), _r.toBool(4)));
4138:   Py_RETURN_NONE;
4139:   END_HANDLE_TH_ERRORS
4140: }
4141: 
4142: \
4143: // isin
4144: static PyObject * THPVariable_isin(PyObject* self_, PyObject* args, PyObject* kwargs)
4145: {
4146:   HANDLE_TH_ERRORS
4147:   static PythonArgParser parser({
4148:     "isin(Tensor elements, Tensor test_elements, *, bool assume_unique=False, bool invert=False, Tensor out=None)",
4149:     "isin(Scalar element, Tensor test_elements, *, bool assume_unique=False, bool invert=False, Tensor out=None)",
4150:     "isin(Tensor elements, Scalar test_element, *, bool assume_unique=False, bool invert=False, Tensor out=None)",
4151:   }, /*traceable=*/true);
4152: 
4153:   ParsedArgs<5> parsed_args;
4154:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4155:   if(_r.has_torch_function()) {
4156:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4157:   }
4158:   switch (_r.idx) {
4159:     case 0: {
4160:       if (_r.isNone(4)) {
```

- EN: The main execution path in this span is carried by `_cufft_clear_plan_cache`, `dispatch__cufft_clear_plan_cache`, `THPVariable__unsafe_masked_index_put_accumulate`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_cufft_clear_plan_cache`, `dispatch__cufft_clear_plan_cache`, `THPVariable__unsafe_masked_index_put_accumulate` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4161-4240

```cpp
4161:         // aten::isin.Tensor_Tensor(Tensor elements, Tensor test_elements, *, bool assume_unique=False, bool invert=False) -> Tensor
4162: 
4163:         auto dispatch_isin = [](const at::Tensor & elements, const at::Tensor & test_elements, bool assume_unique, bool invert) -> at::Tensor {
4164:           pybind11::gil_scoped_release no_gil;
4165:           return at::isin(elements, test_elements, assume_unique, invert);
4166:         };
4167:         return wrap(dispatch_isin(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
4168:       } else {
4169:         // aten::isin.Tensor_Tensor_out(Tensor elements, Tensor test_elements, *, bool assume_unique=False, bool invert=False, Tensor(a!) out) -> Tensor(a!)
4170: 
4171:         auto dispatch_isin_out = [](at::Tensor out, const at::Tensor & elements, const at::Tensor & test_elements, bool assume_unique, bool invert) -> at::Tensor {
4172:           pybind11::gil_scoped_release no_gil;
4173:           return at::isin_out(out, elements, test_elements, assume_unique, invert);
4174:         };
4175:         return wrap(dispatch_isin_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
4176:       }
4177:     }
4178:     case 1: {
4179:       if (_r.isNone(4)) {
4180:         // aten::isin.Scalar_Tensor(Scalar element, Tensor test_elements, *, bool assume_unique=False, bool invert=False) -> Tensor
4181: 
4182:         auto dispatch_isin = [](const at::Scalar & element, const at::Tensor & test_elements, bool assume_unique, bool invert) -> at::Tensor {
4183:           pybind11::gil_scoped_release no_gil;
4184:           return at::isin(element, test_elements, assume_unique, invert);
4185:         };
4186:         return wrap(dispatch_isin(_r.scalar(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
4187:       } else {
4188:         // aten::isin.Scalar_Tensor_out(Scalar element, Tensor test_elements, *, bool assume_unique=False, bool invert=False, Tensor(a!) out) -> Tensor(a!)
4189: 
4190:         auto dispatch_isin_out = [](at::Tensor out, const at::Scalar & element, const at::Tensor & test_elements, bool assume_unique, bool invert) -> at::Tensor {
4191:           pybind11::gil_scoped_release no_gil;
4192:           return at::isin_out(out, element, test_elements, assume_unique, invert);
4193:         };
4194:         return wrap(dispatch_isin_out(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
4195:       }
4196:     }
4197:     case 2: {
4198:       if (_r.isNone(4)) {
4199:         // aten::isin.Tensor_Scalar(Tensor elements, Scalar test_element, *, bool assume_unique=False, bool invert=False) -> Tensor
4200: 
4201:         auto dispatch_isin = [](const at::Tensor & elements, const at::Scalar & test_element, bool assume_unique, bool invert) -> at::Tensor {
4202:           pybind11::gil_scoped_release no_gil;
4203:           return at::isin(elements, test_element, assume_unique, invert);
4204:         };
4205:         return wrap(dispatch_isin(_r.tensor(0), _r.scalar(1), _r.toBool(2), _r.toBool(3)));
4206:       } else {
4207:         // aten::isin.Tensor_Scalar_out(Tensor elements, Scalar test_element, *, bool assume_unique=False, bool invert=False, Tensor(a!) out) -> Tensor(a!)
4208: 
4209:         auto dispatch_isin_out = [](at::Tensor out, const at::Tensor & elements, const at::Scalar & test_element, bool assume_unique, bool invert) -> at::Tensor {
4210:           pybind11::gil_scoped_release no_gil;
4211:           return at::isin_out(out, elements, test_element, assume_unique, invert);
4212:         };
4213:         return wrap(dispatch_isin_out(_r.tensor(4), _r.tensor(0), _r.scalar(1), _r.toBool(2), _r.toBool(3)));
4214:       }
4215:     }
4216:   }
4217:   Py_RETURN_NONE;
4218:   END_HANDLE_TH_ERRORS
4219: }
4220: 
4221: // isnan
4222: static PyObject * THPVariable_isnan(PyObject* self_, PyObject* args, PyObject* kwargs)
4223: {
4224:   HANDLE_TH_ERRORS
4225:   static PythonArgParser parser({
4226:     "isnan(Tensor input)",
4227:   }, /*traceable=*/true);
4228: 
4229:   ParsedArgs<1> parsed_args;
4230:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4231:   if(_r.has_torch_function()) {
4232:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4233:   }
4234:   // aten::isnan(Tensor self) -> Tensor
4235: 
4236:   auto dispatch_isnan = [](const at::Tensor & self) -> at::Tensor {
4237:     pybind11::gil_scoped_release no_gil;
4238:     return self.isnan();
4239:   };
4240:   return wrap(dispatch_isnan(_r.tensor(0)));
```

- EN: The main execution path in this span is carried by `isin`, `wrap`, `isin_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `isin`, `wrap`, `isin_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4241-4320

```cpp
4241:   Py_RETURN_NONE;
4242:   END_HANDLE_TH_ERRORS
4243: }
4244: 
4245: // is_conj
4246: static PyObject * THPVariable_is_conj(PyObject* self_, PyObject* args, PyObject* kwargs)
4247: {
4248:   HANDLE_TH_ERRORS
4249:   static PythonArgParser parser({
4250:     "is_conj(Tensor input)",
4251:   }, /*traceable=*/false);
4252: 
4253:   ParsedArgs<1> parsed_args;
4254:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4255:   if(_r.has_torch_function()) {
4256:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4257:   }
4258:   // aten::is_conj(Tensor self) -> bool
4259: 
4260:   auto dispatch_is_conj = [](const at::Tensor & self) -> bool {
4261:     pybind11::gil_scoped_release no_gil;
4262:     return self.is_conj();
4263:   };
4264:   return wrap(dispatch_is_conj(_r.tensor(0)));
4265:   Py_RETURN_NONE;
4266:   END_HANDLE_TH_ERRORS
4267: }
4268: 
4269: // is_inference
4270: static PyObject * THPVariable_is_inference(PyObject* self_, PyObject* args, PyObject* kwargs)
4271: {
4272:   HANDLE_TH_ERRORS
4273:   static PythonArgParser parser({
4274:     "is_inference(Tensor input)",
4275:   }, /*traceable=*/false);
4276: 
4277:   ParsedArgs<1> parsed_args;
4278:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4279:   if(_r.has_torch_function()) {
4280:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4281:   }
4282:   // aten::is_inference(Tensor self) -> bool
4283: 
4284:   auto dispatch_is_inference = [](const at::Tensor & self) -> bool {
4285:     pybind11::gil_scoped_release no_gil;
4286:     return self.is_inference();
4287:   };
4288:   return wrap(dispatch_is_inference(_r.tensor(0)));
4289:   Py_RETURN_NONE;
4290:   END_HANDLE_TH_ERRORS
4291: }
4292: 
4293: // layer_norm
4294: static PyObject * THPVariable_layer_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
4295: {
4296:   HANDLE_TH_ERRORS
4297:   static PythonArgParser parser({
4298:     "layer_norm(Tensor input, SymIntArrayRef normalized_shape, Tensor? weight=None, Tensor? bias=None, double eps=1e-05, bool cudnn_enable=True)",
4299:   }, /*traceable=*/true);
4300: 
4301:   ParsedArgs<6> parsed_args;
4302:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4303:   if(_r.has_torch_function()) {
4304:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4305:   }
4306:   // aten::layer_norm(Tensor input, SymInt[] normalized_shape, Tensor? weight=None, Tensor? bias=None, float eps=1e-05, bool cudnn_enable=True) -> Tensor
4307: 
4308:   auto dispatch_layer_norm = [](const at::Tensor & input, c10::SymIntArrayRef normalized_shape, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, double eps, bool cudnn_enable) -> at::Tensor {
4309:     pybind11::gil_scoped_release no_gil;
4310:     return at::layer_norm_symint(input, normalized_shape, weight, bias, eps, cudnn_enable);
4311:   };
4312:   return wrap(dispatch_layer_norm(_r.tensor(0), _r.symintlist(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.toDouble(4), _r.toBool(5)));
4313:   Py_RETURN_NONE;
4314:   END_HANDLE_TH_ERRORS
4315: }
4316: 
4317: // _fused_rms_norm
4318: static PyObject * THPVariable__fused_rms_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
4319: {
4320:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_is_conj`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_is_conj`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4321-4400

```cpp
4321:   static PythonArgParser parser({
4322:     "_fused_rms_norm(Tensor input, IntArrayRef normalized_shape, Tensor? weight, double? eps)",
4323:   }, /*traceable=*/true);
4324: 
4325:   ParsedArgs<4> parsed_args;
4326:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4327:   if(_r.has_torch_function()) {
4328:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4329:   }
4330:   // aten::_fused_rms_norm(Tensor input, int[] normalized_shape, Tensor? weight, float? eps) -> (Tensor, Tensor)
4331: 
4332:   auto dispatch__fused_rms_norm = [](const at::Tensor & input, at::IntArrayRef normalized_shape, const ::std::optional<at::Tensor> & weight, ::std::optional<double> eps) -> ::std::tuple<at::Tensor,at::Tensor> {
4333:     pybind11::gil_scoped_release no_gil;
4334:     return at::_fused_rms_norm(input, normalized_shape, weight, eps);
4335:   };
4336:   return wrap(dispatch__fused_rms_norm(_r.tensor(0), _r.intlist(1), _r.optionalTensor(2), _r.toDoubleOptional(3)));
4337:   Py_RETURN_NONE;
4338:   END_HANDLE_TH_ERRORS
4339: }
4340: 
4341: // nan_to_num
4342: static PyObject * THPVariable_nan_to_num(PyObject* self_, PyObject* args, PyObject* kwargs)
4343: {
4344:   HANDLE_TH_ERRORS
4345:   static PythonArgParser parser({
4346:     "nan_to_num(Tensor input, double? nan=None, double? posinf=None, double? neginf=None, *, Tensor out=None)",
4347:   }, /*traceable=*/true);
4348: 
4349:   ParsedArgs<5> parsed_args;
4350:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4351:   if(_r.has_torch_function()) {
4352:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4353:   }
4354:   if (_r.isNone(4)) {
4355:     // aten::nan_to_num(Tensor self, float? nan=None, float? posinf=None, float? neginf=None) -> Tensor
4356: 
4357:     auto dispatch_nan_to_num = [](const at::Tensor & self, ::std::optional<double> nan, ::std::optional<double> posinf, ::std::optional<double> neginf) -> at::Tensor {
4358:       pybind11::gil_scoped_release no_gil;
4359:       return self.nan_to_num(nan, posinf, neginf);
4360:     };
4361:     return wrap(dispatch_nan_to_num(_r.tensor(0), _r.toDoubleOptional(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
4362:   } else {
4363:     // aten::nan_to_num.out(Tensor self, float? nan=None, float? posinf=None, float? neginf=None, *, Tensor(a!) out) -> Tensor(a!)
4364: 
4365:     auto dispatch_nan_to_num_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<double> nan, ::std::optional<double> posinf, ::std::optional<double> neginf) -> at::Tensor {
4366:       pybind11::gil_scoped_release no_gil;
4367:       return at::nan_to_num_out(out, self, nan, posinf, neginf);
4368:     };
4369:     return wrap(dispatch_nan_to_num_out(_r.tensor(4), _r.tensor(0), _r.toDoubleOptional(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
4370:   }
4371:   Py_RETURN_NONE;
4372:   END_HANDLE_TH_ERRORS
4373: }
4374: 
4375: // nan_to_num_
4376: static PyObject * THPVariable_nan_to_num_(PyObject* self_, PyObject* args, PyObject* kwargs)
4377: {
4378:   HANDLE_TH_ERRORS
4379:   static PythonArgParser parser({
4380:     "nan_to_num_(Tensor input, double? nan=None, double? posinf=None, double? neginf=None)",
4381:   }, /*traceable=*/true);
4382: 
4383:   ParsedArgs<4> parsed_args;
4384:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4385:   if(_r.has_torch_function()) {
4386:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4387:   }
4388:   // aten::nan_to_num_(Tensor(a!) self, float? nan=None, float? posinf=None, float? neginf=None) -> Tensor(a!)
4389: 
4390:   auto dispatch_nan_to_num_ = [](at::Tensor self, ::std::optional<double> nan, ::std::optional<double> posinf, ::std::optional<double> neginf) -> at::Tensor {
4391:     pybind11::gil_scoped_release no_gil;
4392:     return self.nan_to_num_(nan, posinf, neginf);
4393:   };
4394:   return wrap(dispatch_nan_to_num_(_r.tensor(0), _r.toDoubleOptional(1), _r.toDoubleOptional(2), _r.toDoubleOptional(3)));
4395:   Py_RETURN_NONE;
4396:   END_HANDLE_TH_ERRORS
4397: }
4398: 
4399: // mkldnn_linear_backward_weights
4400: static PyObject * THPVariable_mkldnn_linear_backward_weights(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_fused_rms_norm`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_fused_rms_norm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4401-4480

```cpp
4401: {
4402:   HANDLE_TH_ERRORS
4403:   static PythonArgParser parser({
4404:     "mkldnn_linear_backward_weights(Tensor grad_output, Tensor input, Tensor weight, bool bias_defined)",
4405:   }, /*traceable=*/true);
4406: 
4407:   ParsedArgs<4> parsed_args;
4408:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4409:   if(_r.has_torch_function()) {
4410:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4411:   }
4412:   // aten::mkldnn_linear_backward_weights(Tensor grad_output, Tensor input, Tensor weight, bool bias_defined) -> (Tensor, Tensor)
4413: 
4414:   auto dispatch_mkldnn_linear_backward_weights = [](const at::Tensor & grad_output, const at::Tensor & input, const at::Tensor & weight, bool bias_defined) -> ::std::tuple<at::Tensor,at::Tensor> {
4415:     pybind11::gil_scoped_release no_gil;
4416:     return at::mkldnn_linear_backward_weights(grad_output, input, weight, bias_defined);
4417:   };
4418:   return wrap(dispatch_mkldnn_linear_backward_weights(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3)));
4419:   Py_RETURN_NONE;
4420:   END_HANDLE_TH_ERRORS
4421: }
4422: 
4423: // _cslt_sparse_mm
4424: static PyObject * THPVariable__cslt_sparse_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
4425: {
4426:   HANDLE_TH_ERRORS
4427:   static PythonArgParser parser({
4428:     "_cslt_sparse_mm(Tensor compressed_A, Tensor dense_B, Tensor? bias=None, Tensor? alpha=None, ScalarType? out_dtype=None, bool transpose_result=False, int64_t alg_id=0, int64_t split_k=1, int64_t split_k_mode=-1)",
4429:   }, /*traceable=*/true);
4430: 
4431:   ParsedArgs<9> parsed_args;
4432:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4433:   if(_r.has_torch_function()) {
4434:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4435:   }
4436:   // aten::_cslt_sparse_mm(Tensor compressed_A, Tensor dense_B, Tensor? bias=None, Tensor? alpha=None, ScalarType? out_dtype=None, bool transpose_result=False, int alg_id=0, int split_k=1, int split_k_mode=-1) -> Tensor
4437: 
4438:   auto dispatch__cslt_sparse_mm = [](const at::Tensor & compressed_A, const at::Tensor & dense_B, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & alpha, ::std::optional<at::ScalarType> out_dtype, bool transpose_result, int64_t alg_id, int64_t split_k, int64_t split_k_mode) -> at::Tensor {
4439:     pybind11::gil_scoped_release no_gil;
4440:     return at::_cslt_sparse_mm(compressed_A, dense_B, bias, alpha, out_dtype, transpose_result, alg_id, split_k, split_k_mode);
4441:   };
4442:   return wrap(dispatch__cslt_sparse_mm(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.scalartypeOptional(4), _r.toBool(5), _r.toInt64(6), _r.toInt64(7), _r.toInt64(8)));
4443:   Py_RETURN_NONE;
4444:   END_HANDLE_TH_ERRORS
4445: }
4446: 
4447: // _sparse_semi_structured_mm
4448: static PyObject * THPVariable__sparse_semi_structured_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
4449: {
4450:   HANDLE_TH_ERRORS
4451:   static PythonArgParser parser({
4452:     "_sparse_semi_structured_mm(Tensor mat1, Tensor mat1_meta, Tensor mat2, *, ScalarType? out_dtype=None)",
4453:   }, /*traceable=*/true);
4454: 
4455:   ParsedArgs<4> parsed_args;
4456:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4457:   if(_r.has_torch_function()) {
4458:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4459:   }
4460:   // aten::_sparse_semi_structured_mm(Tensor mat1, Tensor mat1_meta, Tensor mat2, *, ScalarType? out_dtype=None) -> Tensor
4461: 
4462:   auto dispatch__sparse_semi_structured_mm = [](const at::Tensor & mat1, const at::Tensor & mat1_meta, const at::Tensor & mat2, ::std::optional<at::ScalarType> out_dtype) -> at::Tensor {
4463:     pybind11::gil_scoped_release no_gil;
4464:     return at::_sparse_semi_structured_mm(mat1, mat1_meta, mat2, out_dtype);
4465:   };
4466:   return wrap(dispatch__sparse_semi_structured_mm(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalartypeOptional(3)));
4467:   Py_RETURN_NONE;
4468:   END_HANDLE_TH_ERRORS
4469: }
4470: 
4471: // _sparse_semi_structured_addmm
4472: static PyObject * THPVariable__sparse_semi_structured_addmm(PyObject* self_, PyObject* args, PyObject* kwargs)
4473: {
4474:   HANDLE_TH_ERRORS
4475:   static PythonArgParser parser({
4476:     "_sparse_semi_structured_addmm(Tensor input, Tensor mat1, Tensor mat1_meta, Tensor mat2, *, Scalar alpha=1, Scalar beta=1, ScalarType? out_dtype=None)",
4477:   }, /*traceable=*/true);
4478: 
4479:   ParsedArgs<7> parsed_args;
4480:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `mkldnn_linear_backward_weights`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `mkldnn_linear_backward_weights` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4481-4560

```cpp
4481:   if(_r.has_torch_function()) {
4482:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4483:   }
4484:   // aten::_sparse_semi_structured_addmm(Tensor input, Tensor mat1, Tensor mat1_meta, Tensor mat2, *, Scalar alpha=1, Scalar beta=1, ScalarType? out_dtype=None) -> Tensor
4485: 
4486:   auto dispatch__sparse_semi_structured_addmm = [](const at::Tensor & input, const at::Tensor & mat1, const at::Tensor & mat1_meta, const at::Tensor & mat2, const at::Scalar & alpha, const at::Scalar & beta, ::std::optional<at::ScalarType> out_dtype) -> at::Tensor {
4487:     pybind11::gil_scoped_release no_gil;
4488:     return at::_sparse_semi_structured_addmm(input, mat1, mat1_meta, mat2, alpha, beta, out_dtype);
4489:   };
4490:   return wrap(dispatch__sparse_semi_structured_addmm(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.scalar(4), _r.scalar(5), _r.scalartypeOptional(6)));
4491:   Py_RETURN_NONE;
4492:   END_HANDLE_TH_ERRORS
4493: }
4494: 
4495: // _mixed_dtypes_linear
4496: static PyObject * THPVariable__mixed_dtypes_linear(PyObject* self_, PyObject* args, PyObject* kwargs)
4497: {
4498:   HANDLE_TH_ERRORS
4499:   static PythonArgParser parser({
4500:     "_mixed_dtypes_linear(Tensor input, Tensor weight, Tensor scale, *, Tensor? bias=None, c10::string_view? activation=None)",
4501:   }, /*traceable=*/true);
4502: 
4503:   ParsedArgs<5> parsed_args;
4504:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4505:   if(_r.has_torch_function()) {
4506:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4507:   }
4508:   // aten::_mixed_dtypes_linear(Tensor input, Tensor weight, Tensor scale, *, Tensor? bias=None, str? activation=None) -> Tensor
4509: 
4510:   auto dispatch__mixed_dtypes_linear = [](const at::Tensor & input, const at::Tensor & weight, const at::Tensor & scale, const ::std::optional<at::Tensor> & bias, ::std::optional<c10::string_view> activation) -> at::Tensor {
4511:     pybind11::gil_scoped_release no_gil;
4512:     return at::_mixed_dtypes_linear(input, weight, scale, bias, activation);
4513:   };
4514:   return wrap(dispatch__mixed_dtypes_linear(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.stringViewOptional(4)));
4515:   Py_RETURN_NONE;
4516:   END_HANDLE_TH_ERRORS
4517: }
4518: 
4519: // fbgemm_pack_gemm_matrix_fp16
4520: static PyObject * THPVariable_fbgemm_pack_gemm_matrix_fp16(PyObject* self_, PyObject* args, PyObject* kwargs)
4521: {
4522:   HANDLE_TH_ERRORS
4523:   static PythonArgParser parser({
4524:     "fbgemm_pack_gemm_matrix_fp16(Tensor input)",
4525:   }, /*traceable=*/true);
4526: 
4527:   ParsedArgs<1> parsed_args;
4528:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4529:   if(_r.has_torch_function()) {
4530:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4531:   }
4532:   // aten::fbgemm_pack_gemm_matrix_fp16(Tensor input) -> Tensor
4533: 
4534:   auto dispatch_fbgemm_pack_gemm_matrix_fp16 = [](const at::Tensor & input) -> at::Tensor {
4535:     pybind11::gil_scoped_release no_gil;
4536:     return at::fbgemm_pack_gemm_matrix_fp16(input);
4537:   };
4538:   return wrap(dispatch_fbgemm_pack_gemm_matrix_fp16(_r.tensor(0)));
4539:   Py_RETURN_NONE;
4540:   END_HANDLE_TH_ERRORS
4541: }
4542: 
4543: // _wrapped_quantized_linear_prepacked
4544: static PyObject * THPVariable__wrapped_quantized_linear_prepacked(PyObject* self_, PyObject* args, PyObject* kwargs)
4545: {
4546:   HANDLE_TH_ERRORS
4547:   static PythonArgParser parser({
4548:     "_wrapped_quantized_linear_prepacked(Tensor input, Tensor input_scale, Tensor input_zero_point, Tensor packed_weight, Tensor output_scale, Tensor output_zero_point, int64_t out_channel)",
4549:   }, /*traceable=*/true);
4550: 
4551:   ParsedArgs<7> parsed_args;
4552:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4553:   if(_r.has_torch_function()) {
4554:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4555:   }
4556:   // aten::_wrapped_quantized_linear_prepacked(Tensor input, Tensor input_scale, Tensor input_zero_point, Tensor packed_weight, Tensor output_scale, Tensor output_zero_point, int out_channel) -> Tensor
4557: 
4558:   auto dispatch__wrapped_quantized_linear_prepacked = [](const at::Tensor & input, const at::Tensor & input_scale, const at::Tensor & input_zero_point, const at::Tensor & packed_weight, const at::Tensor & output_scale, const at::Tensor & output_zero_point, int64_t out_channel) -> at::Tensor {
4559:     pybind11::gil_scoped_release no_gil;
4560:     return at::_wrapped_quantized_linear_prepacked(input, input_scale, input_zero_point, packed_weight, output_scale, output_zero_point, out_channel);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_sparse_semi_structured_addmm`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_sparse_semi_structured_addmm`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4640

```cpp
4561:   };
4562:   return wrap(dispatch__wrapped_quantized_linear_prepacked(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.tensor(4), _r.tensor(5), _r.toInt64(6)));
4563:   Py_RETURN_NONE;
4564:   END_HANDLE_TH_ERRORS
4565: }
4566: 
4567: \
4568: // fbgemm_linear_fp16_weight_fp32_activation
4569: static PyObject * THPVariable_fbgemm_linear_fp16_weight_fp32_activation(PyObject* self_, PyObject* args, PyObject* kwargs)
4570: {
4571:   HANDLE_TH_ERRORS
4572:   static PythonArgParser parser({
4573:     "fbgemm_linear_fp16_weight_fp32_activation(Tensor input, Tensor packed_weight, Tensor? bias)",
4574:     "fbgemm_linear_fp16_weight_fp32_activation(Tensor input, Tensor packed_weight, Tensor? bias, Tensor output)",
4575:   }, /*traceable=*/true);
4576: 
4577:   ParsedArgs<4> parsed_args;
4578:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4579:   if(_r.has_torch_function()) {
4580:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4581:   }
4582:   switch (_r.idx) {
4583:     case 0: {
4584:       // aten::fbgemm_linear_fp16_weight_fp32_activation(Tensor input, Tensor packed_weight, Tensor? bias) -> Tensor
4585: 
4586:       auto dispatch_fbgemm_linear_fp16_weight_fp32_activation = [](const at::Tensor & input, const at::Tensor & packed_weight, const ::std::optional<at::Tensor> & bias) -> at::Tensor {
4587:         pybind11::gil_scoped_release no_gil;
4588:         return at::fbgemm_linear_fp16_weight_fp32_activation(input, packed_weight, bias);
4589:       };
4590:       return wrap(dispatch_fbgemm_linear_fp16_weight_fp32_activation(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2)));
4591:     }
4592:     case 1: {
4593:       // aten::fbgemm_linear_fp16_weight_fp32_activation.out(Tensor input, Tensor packed_weight, Tensor? bias, Tensor(a!) output) -> Tensor
4594: 
4595:       auto dispatch_fbgemm_linear_fp16_weight_fp32_activation = [](const at::Tensor & input, const at::Tensor & packed_weight, const ::std::optional<at::Tensor> & bias, at::Tensor output) -> at::Tensor {
4596:         pybind11::gil_scoped_release no_gil;
4597:         return at::fbgemm_linear_fp16_weight_fp32_activation(input, packed_weight, bias, output);
4598:       };
4599:       return wrap(dispatch_fbgemm_linear_fp16_weight_fp32_activation(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.tensor(3)));
4600:     }
4601:   }
4602:   Py_RETURN_NONE;
4603:   END_HANDLE_TH_ERRORS
4604: }
4605: 
4606: \
4607: // linspace
4608: static PyObject * THPVariable_linspace(PyObject* self_, PyObject* args, PyObject* kwargs)
4609: {
4610:   HANDLE_TH_ERRORS
4611:   static PythonArgParser parser({
4612:     "linspace(Tensor start, Tensor end, int64_t steps, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
4613:     "linspace(Scalar start, Tensor end, int64_t steps, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
4614:     "linspace(Tensor start, Scalar end, int64_t steps, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
4615:     "linspace(Scalar start, Scalar end, int64_t steps, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
4616:   }, /*traceable=*/true);
4617: 
4618:   ParsedArgs<9> parsed_args;
4619:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4620:   if(_r.has_torch_function()) {
4621:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4622:   }
4623:   switch (_r.idx) {
4624:     case 0: {
4625:       if (_r.isNone(3)) {
4626:         // aten::linspace.Tensor_Tensor(Tensor start, Tensor end, int steps, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
4627:         const auto options = TensorOptions()
4628:             .dtype(_r.scalartypeOptional(4))
4629:             .device(_r.deviceWithDefault(6, torch::tensors::get_default_device()))
4630:             .layout(_r.layoutOptional(5))
4631:             .requires_grad(_r.toBool(8))
4632:             .pinned_memory(_r.toBool(7));
4633:         torch::utils::maybe_initialize_device(options);
4634: 
4635:         auto dispatch_linspace = [](const at::Tensor & start, const at::Tensor & end, int64_t steps, at::TensorOptions options) -> at::Tensor {
4636:           pybind11::gil_scoped_release no_gil;
4637:           return torch::linspace(start, end, steps, options);
4638:         };
4639:         return wrap(dispatch_linspace(_r.tensor(0), _r.tensor(1), _r.toInt64(2), options));
4640:       } else {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_fbgemm_linear_fp16_weight_fp32_activation`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_fbgemm_linear_fp16_weight_fp32_activation`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4641-4720

```cpp
4641:         // aten::linspace.Tensor_Tensor_out(Tensor start, Tensor end, int steps, *, Tensor(a!) out) -> Tensor(a!)
4642:         check_out_type_matches(_r.tensor(3), _r.scalartypeOptional(4),
4643:                                _r.isNone(4), _r.layoutOptional(5),
4644:                                _r.deviceWithDefault(6, torch::tensors::get_default_device()), _r.isNone(6));
4645: 
4646:         auto dispatch_linspace_out = [](at::Tensor out, const at::Tensor & start, const at::Tensor & end, int64_t steps) -> at::Tensor {
4647:           pybind11::gil_scoped_release no_gil;
4648:           return at::linspace_out(out, start, end, steps);
4649:         };
4650:         return wrap(dispatch_linspace_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.toInt64(2)).set_requires_grad(_r.toBool(8)));
4651:       }
4652:     }
4653:     case 1: {
4654:       if (_r.isNone(3)) {
4655:         // aten::linspace.Scalar_Tensor(Scalar start, Tensor end, int steps, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
4656:         const auto options = TensorOptions()
4657:             .dtype(_r.scalartypeOptional(4))
4658:             .device(_r.deviceWithDefault(6, torch::tensors::get_default_device()))
4659:             .layout(_r.layoutOptional(5))
4660:             .requires_grad(_r.toBool(8))
4661:             .pinned_memory(_r.toBool(7));
4662:         torch::utils::maybe_initialize_device(options);
4663: 
4664:         auto dispatch_linspace = [](const at::Scalar & start, const at::Tensor & end, int64_t steps, at::TensorOptions options) -> at::Tensor {
4665:           pybind11::gil_scoped_release no_gil;
4666:           return torch::linspace(start, end, steps, options);
4667:         };
4668:         return wrap(dispatch_linspace(_r.scalar(0), _r.tensor(1), _r.toInt64(2), options));
4669:       } else {
4670:         // aten::linspace.Scalar_Tensor_out(Scalar start, Tensor end, int steps, *, Tensor(a!) out) -> Tensor(a!)
4671:         check_out_type_matches(_r.tensor(3), _r.scalartypeOptional(4),
4672:                                _r.isNone(4), _r.layoutOptional(5),
4673:                                _r.deviceWithDefault(6, torch::tensors::get_default_device()), _r.isNone(6));
4674: 
4675:         auto dispatch_linspace_out = [](at::Tensor out, const at::Scalar & start, const at::Tensor & end, int64_t steps) -> at::Tensor {
4676:           pybind11::gil_scoped_release no_gil;
4677:           return at::linspace_out(out, start, end, steps);
4678:         };
4679:         return wrap(dispatch_linspace_out(_r.tensor(3), _r.scalar(0), _r.tensor(1), _r.toInt64(2)).set_requires_grad(_r.toBool(8)));
4680:       }
4681:     }
4682:     case 2: {
4683:       if (_r.isNone(3)) {
4684:         // aten::linspace.Tensor_Scalar(Tensor start, Scalar end, int steps, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
4685:         const auto options = TensorOptions()
4686:             .dtype(_r.scalartypeOptional(4))
4687:             .device(_r.deviceWithDefault(6, torch::tensors::get_default_device()))
4688:             .layout(_r.layoutOptional(5))
4689:             .requires_grad(_r.toBool(8))
4690:             .pinned_memory(_r.toBool(7));
4691:         torch::utils::maybe_initialize_device(options);
4692: 
4693:         auto dispatch_linspace = [](const at::Tensor & start, const at::Scalar & end, int64_t steps, at::TensorOptions options) -> at::Tensor {
4694:           pybind11::gil_scoped_release no_gil;
4695:           return torch::linspace(start, end, steps, options);
4696:         };
4697:         return wrap(dispatch_linspace(_r.tensor(0), _r.scalar(1), _r.toInt64(2), options));
4698:       } else {
4699:         // aten::linspace.Tensor_Scalar_out(Tensor start, Scalar end, int steps, *, Tensor(a!) out) -> Tensor(a!)
4700:         check_out_type_matches(_r.tensor(3), _r.scalartypeOptional(4),
4701:                                _r.isNone(4), _r.layoutOptional(5),
4702:                                _r.deviceWithDefault(6, torch::tensors::get_default_device()), _r.isNone(6));
4703: 
4704:         auto dispatch_linspace_out = [](at::Tensor out, const at::Tensor & start, const at::Scalar & end, int64_t steps) -> at::Tensor {
4705:           pybind11::gil_scoped_release no_gil;
4706:           return at::linspace_out(out, start, end, steps);
4707:         };
4708:         return wrap(dispatch_linspace_out(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.toInt64(2)).set_requires_grad(_r.toBool(8)));
4709:       }
4710:     }
4711:     case 3: {
4712:       if (_r.isNone(3)) {
4713:         // aten::linspace(Scalar start, Scalar end, int steps, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
4714:         const auto options = TensorOptions()
4715:             .dtype(_r.scalartypeOptional(4))
4716:             .device(_r.deviceWithDefault(6, torch::tensors::get_default_device()))
4717:             .layout(_r.layoutOptional(5))
4718:             .requires_grad(_r.toBool(8))
4719:             .pinned_memory(_r.toBool(7));
4720:         torch::utils::maybe_initialize_device(options);
```

- EN: The main execution path in this span is carried by `check_out_type_matches`, `linspace_out`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `check_out_type_matches`, `linspace_out`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4721-4800

```cpp
4721: 
4722:         auto dispatch_linspace = [](const at::Scalar & start, const at::Scalar & end, int64_t steps, at::TensorOptions options) -> at::Tensor {
4723:           pybind11::gil_scoped_release no_gil;
4724:           return torch::linspace(start, end, steps, options);
4725:         };
4726:         return wrap(dispatch_linspace(_r.scalar(0), _r.scalar(1), _r.toInt64(2), options));
4727:       } else {
4728:         // aten::linspace.out(Scalar start, Scalar end, int steps, *, Tensor(a!) out) -> Tensor(a!)
4729:         check_out_type_matches(_r.tensor(3), _r.scalartypeOptional(4),
4730:                                _r.isNone(4), _r.layoutOptional(5),
4731:                                _r.deviceWithDefault(6, torch::tensors::get_default_device()), _r.isNone(6));
4732: 
4733:         auto dispatch_linspace_out = [](at::Tensor out, const at::Scalar & start, const at::Scalar & end, int64_t steps) -> at::Tensor {
4734:           pybind11::gil_scoped_release no_gil;
4735:           return at::linspace_out(out, start, end, steps);
4736:         };
4737:         return wrap(dispatch_linspace_out(_r.tensor(3), _r.scalar(0), _r.scalar(1), _r.toInt64(2)).set_requires_grad(_r.toBool(8)));
4738:       }
4739:     }
4740:   }
4741:   Py_RETURN_NONE;
4742:   END_HANDLE_TH_ERRORS
4743: }
4744: 
4745: // log
4746: static PyObject * THPVariable_log(PyObject* self_, PyObject* args, PyObject* kwargs)
4747: {
4748:   HANDLE_TH_ERRORS
4749:   static PythonArgParser parser({
4750:     "log(Tensor input, *, Tensor out=None)",
4751:   }, /*traceable=*/true);
4752: 
4753:   ParsedArgs<2> parsed_args;
4754:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4755:   if(_r.has_torch_function()) {
4756:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4757:   }
4758:   if (_r.isNone(1)) {
4759:     // aten::log(Tensor self) -> Tensor
4760: 
4761:     auto dispatch_log = [](const at::Tensor & self) -> at::Tensor {
4762:       pybind11::gil_scoped_release no_gil;
4763:       return self.log();
4764:     };
4765:     return wrap(dispatch_log(_r.tensor(0)));
4766:   } else {
4767:     // aten::log.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
4768: 
4769:     auto dispatch_log_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
4770:       pybind11::gil_scoped_release no_gil;
4771:       return at::log_out(out, self);
4772:     };
4773:     return wrap(dispatch_log_out(_r.tensor(1), _r.tensor(0)));
4774:   }
4775:   Py_RETURN_NONE;
4776:   END_HANDLE_TH_ERRORS
4777: }
4778: 
4779: // log_
4780: static PyObject * THPVariable_log_(PyObject* self_, PyObject* args, PyObject* kwargs)
4781: {
4782:   HANDLE_TH_ERRORS
4783:   static PythonArgParser parser({
4784:     "log_(Tensor input)",
4785:   }, /*traceable=*/true);
4786: 
4787:   ParsedArgs<1> parsed_args;
4788:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4789:   if(_r.has_torch_function()) {
4790:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4791:   }
4792:   // aten::log_(Tensor(a!) self) -> Tensor(a!)
4793: 
4794:   auto dispatch_log_ = [](at::Tensor self) -> at::Tensor {
4795:     pybind11::gil_scoped_release no_gil;
4796:     return self.log_();
4797:   };
4798:   return wrap(dispatch_log_(_r.tensor(0)));
4799:   Py_RETURN_NONE;
4800:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `linspace`, `wrap`, `check_out_type_matches`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `linspace`, `wrap`, `check_out_type_matches` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4801-4880

```cpp
4801: }
4802: 
4803: // log1p
4804: static PyObject * THPVariable_log1p(PyObject* self_, PyObject* args, PyObject* kwargs)
4805: {
4806:   HANDLE_TH_ERRORS
4807:   static PythonArgParser parser({
4808:     "log1p(Tensor input, *, Tensor out=None)",
4809:   }, /*traceable=*/true);
4810: 
4811:   ParsedArgs<2> parsed_args;
4812:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4813:   if(_r.has_torch_function()) {
4814:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4815:   }
4816:   if (_r.isNone(1)) {
4817:     // aten::log1p(Tensor self) -> Tensor
4818: 
4819:     auto dispatch_log1p = [](const at::Tensor & self) -> at::Tensor {
4820:       pybind11::gil_scoped_release no_gil;
4821:       return self.log1p();
4822:     };
4823:     return wrap(dispatch_log1p(_r.tensor(0)));
4824:   } else {
4825:     // aten::log1p.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
4826: 
4827:     auto dispatch_log1p_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
4828:       pybind11::gil_scoped_release no_gil;
4829:       return at::log1p_out(out, self);
4830:     };
4831:     return wrap(dispatch_log1p_out(_r.tensor(1), _r.tensor(0)));
4832:   }
4833:   Py_RETURN_NONE;
4834:   END_HANDLE_TH_ERRORS
4835: }
4836: 
4837: // log1p_
4838: static PyObject * THPVariable_log1p_(PyObject* self_, PyObject* args, PyObject* kwargs)
4839: {
4840:   HANDLE_TH_ERRORS
4841:   static PythonArgParser parser({
4842:     "log1p_(Tensor input)",
4843:   }, /*traceable=*/true);
4844: 
4845:   ParsedArgs<1> parsed_args;
4846:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4847:   if(_r.has_torch_function()) {
4848:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4849:   }
4850:   // aten::log1p_(Tensor(a!) self) -> Tensor(a!)
4851: 
4852:   auto dispatch_log1p_ = [](at::Tensor self) -> at::Tensor {
4853:     pybind11::gil_scoped_release no_gil;
4854:     return self.log1p_();
4855:   };
4856:   return wrap(dispatch_log1p_(_r.tensor(0)));
4857:   Py_RETURN_NONE;
4858:   END_HANDLE_TH_ERRORS
4859: }
4860: 
4861: // logaddexp
4862: static PyObject * THPVariable_logaddexp(PyObject* self_, PyObject* args, PyObject* kwargs)
4863: {
4864:   HANDLE_TH_ERRORS
4865:   static PythonArgParser parser({
4866:     "logaddexp(Tensor input, Tensor other, *, Tensor out=None)",
4867:   }, /*traceable=*/true);
4868: 
4869:   ParsedArgs<3> parsed_args;
4870:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4871:   if(_r.has_torch_function()) {
4872:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4873:   }
4874:   if (_r.isNone(2)) {
4875:     // aten::logaddexp(Tensor self, Tensor other) -> Tensor
4876: 
4877:     auto dispatch_logaddexp = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4878:       pybind11::gil_scoped_release no_gil;
4879:       return self.logaddexp(other);
4880:     };
```

- EN: The main execution path in this span is carried by `THPVariable_log1p`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_log1p`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881:     return wrap(dispatch_logaddexp(_r.tensor(0), _r.tensor(1)));
4882:   } else {
4883:     // aten::logaddexp.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
4884: 
4885:     auto dispatch_logaddexp_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4886:       pybind11::gil_scoped_release no_gil;
4887:       return at::logaddexp_out(out, self, other);
4888:     };
4889:     return wrap(dispatch_logaddexp_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4890:   }
4891:   Py_RETURN_NONE;
4892:   END_HANDLE_TH_ERRORS
4893: }
4894: 
4895: \
4896: // xlogy
4897: static PyObject * THPVariable_xlogy(PyObject* self_, PyObject* args, PyObject* kwargs)
4898: {
4899:   HANDLE_TH_ERRORS
4900:   static PythonArgParser parser({
4901:     "xlogy(Tensor input, Tensor other, *, Tensor out=None)",
4902:     "xlogy(Scalar self, Tensor other, *, Tensor out=None)",
4903:     "xlogy(Tensor input, Scalar other, *, Tensor out=None)",
4904:   }, /*traceable=*/true);
4905: 
4906:   ParsedArgs<3> parsed_args;
4907:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4908:   if(_r.has_torch_function()) {
4909:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4910:   }
4911:   switch (_r.idx) {
4912:     case 0: {
4913:       if (_r.isNone(2)) {
4914:         // aten::xlogy.Tensor(Tensor self, Tensor other) -> Tensor
4915: 
4916:         auto dispatch_xlogy = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4917:           pybind11::gil_scoped_release no_gil;
4918:           return self.xlogy(other);
4919:         };
4920:         return wrap(dispatch_xlogy(_r.tensor(0), _r.tensor(1)));
4921:       } else {
4922:         // aten::xlogy.OutTensor(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
4923: 
4924:         auto dispatch_xlogy_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4925:           pybind11::gil_scoped_release no_gil;
4926:           return at::xlogy_out(out, self, other);
4927:         };
4928:         return wrap(dispatch_xlogy_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4929:       }
4930:     }
4931:     case 1: {
4932:       if (_r.isNone(2)) {
4933:         // aten::xlogy.Scalar_Self(Scalar self, Tensor other) -> Tensor
4934: 
4935:         auto dispatch_xlogy = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
4936:           pybind11::gil_scoped_release no_gil;
4937:           return at::xlogy(self, other);
4938:         };
4939:         return wrap(dispatch_xlogy(_r.scalar(0), _r.tensor(1)));
4940:       } else {
4941:         // aten::xlogy.OutScalar_Self(Scalar self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
4942: 
4943:         auto dispatch_xlogy_out = [](at::Tensor out, const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
4944:           pybind11::gil_scoped_release no_gil;
4945:           return at::xlogy_out(out, self, other);
4946:         };
4947:         return wrap(dispatch_xlogy_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
4948:       }
4949:     }
4950:     case 2: {
4951:       if (_r.isNone(2)) {
4952:         // aten::xlogy.Scalar_Other(Tensor self, Scalar other) -> Tensor
4953: 
4954:         auto dispatch_xlogy = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4955:           pybind11::gil_scoped_release no_gil;
4956:           return self.xlogy(other);
4957:         };
4958:         return wrap(dispatch_xlogy(_r.tensor(0), _r.scalar(1)));
4959:       } else {
4960:         // aten::xlogy.OutScalar_Other(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `wrap`, `logaddexp_out`, `THPVariable_xlogy`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `logaddexp_out`, `THPVariable_xlogy` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961: 
4962:         auto dispatch_xlogy_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4963:           pybind11::gil_scoped_release no_gil;
4964:           return at::xlogy_out(out, self, other);
4965:         };
4966:         return wrap(dispatch_xlogy_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
4967:       }
4968:     }
4969:   }
4970:   Py_RETURN_NONE;
4971:   END_HANDLE_TH_ERRORS
4972: }
4973: 
4974: \
4975: // xlogy_
4976: static PyObject * THPVariable_xlogy_(PyObject* self_, PyObject* args, PyObject* kwargs)
4977: {
4978:   HANDLE_TH_ERRORS
4979:   static PythonArgParser parser({
4980:     "xlogy_(Tensor input, Tensor other)",
4981:     "xlogy_(Tensor input, Scalar other)",
4982:   }, /*traceable=*/true);
4983: 
4984:   ParsedArgs<2> parsed_args;
4985:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4986:   if(_r.has_torch_function()) {
4987:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4988:   }
4989:   switch (_r.idx) {
4990:     case 0: {
4991:       // aten::xlogy_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
4992: 
4993:       auto dispatch_xlogy_ = [](at::Tensor self, const at::Tensor & other) -> at::Tensor {
4994:         pybind11::gil_scoped_release no_gil;
4995:         return self.xlogy_(other);
4996:       };
4997:       return wrap(dispatch_xlogy_(_r.tensor(0), _r.tensor(1)));
4998:     }
4999:     case 1: {
5000:       // aten::xlogy_.Scalar_Other(Tensor(a!) self, Scalar other) -> Tensor(a!)
5001: 
5002:       auto dispatch_xlogy_ = [](at::Tensor self, const at::Scalar & other) -> at::Tensor {
5003:         pybind11::gil_scoped_release no_gil;
5004:         return self.xlogy_(other);
5005:       };
5006:       return wrap(dispatch_xlogy_(_r.tensor(0), _r.scalar(1)));
5007:     }
5008:   }
5009:   Py_RETURN_NONE;
5010:   END_HANDLE_TH_ERRORS
5011: }
5012: 
5013: \
5014: // logsumexp
5015: static PyObject * THPVariable_logsumexp(PyObject* self_, PyObject* args, PyObject* kwargs)
5016: {
5017:   HANDLE_TH_ERRORS
5018:   static PythonArgParser parser({
5019:     "logsumexp(Tensor input, IntArrayRef[1] dim, bool keepdim=False, *, Tensor out=None)",
5020:     "logsumexp(Tensor input, DimnameList[1] dim, bool keepdim=False, *, Tensor out=None)",
5021:   }, /*traceable=*/true);
5022: 
5023:   ParsedArgs<4> parsed_args;
5024:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5025:   if(_r.has_torch_function()) {
5026:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5027:   }
5028:   switch (_r.idx) {
5029:     case 0: {
5030:       if (_r.isNone(3)) {
5031:         // aten::logsumexp(Tensor self, int[1] dim, bool keepdim=False) -> Tensor
5032: 
5033:         auto dispatch_logsumexp = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
5034:           pybind11::gil_scoped_release no_gil;
5035:           return self.logsumexp(dim, keepdim);
5036:         };
5037:         return wrap(dispatch_logsumexp(_r.tensor(0), _r.intlist(1), _r.toBool(2)));
5038:       } else {
5039:         // aten::logsumexp.out(Tensor self, int[1] dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
5040: 
```

- EN: The main execution path in this span is carried by `xlogy_out`, `wrap`, `THPVariable_xlogy_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `xlogy_out`, `wrap`, `THPVariable_xlogy_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5041-5120

```cpp
5041:         auto dispatch_logsumexp_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
5042:           pybind11::gil_scoped_release no_gil;
5043:           return at::logsumexp_out(out, self, dim, keepdim);
5044:         };
5045:         return wrap(dispatch_logsumexp_out(_r.tensor(3), _r.tensor(0), _r.intlist(1), _r.toBool(2)));
5046:       }
5047:     }
5048:     case 1: {
5049:       if (_r.isNone(3)) {
5050:         // aten::logsumexp.names(Tensor self, Dimname[1] dim, bool keepdim=False) -> Tensor
5051: 
5052:         auto dispatch_logsumexp = [](const at::Tensor & self, at::DimnameList dim, bool keepdim) -> at::Tensor {
5053:           pybind11::gil_scoped_release no_gil;
5054:           return self.logsumexp(dim, keepdim);
5055:         };
5056:         return wrap(dispatch_logsumexp(_r.tensor(0), _r.dimnamelist(1), _r.toBool(2)));
5057:       } else {
5058:         // aten::logsumexp.names_out(Tensor self, Dimname[1] dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
5059: 
5060:         auto dispatch_logsumexp_out = [](at::Tensor out, const at::Tensor & self, at::DimnameList dim, bool keepdim) -> at::Tensor {
5061:           pybind11::gil_scoped_release no_gil;
5062:           return at::logsumexp_out(out, self, dim, keepdim);
5063:         };
5064:         return wrap(dispatch_logsumexp_out(_r.tensor(3), _r.tensor(0), _r.dimnamelist(1), _r.toBool(2)));
5065:       }
5066:     }
5067:   }
5068:   Py_RETURN_NONE;
5069:   END_HANDLE_TH_ERRORS
5070: }
5071: 
5072: // matrix_power
5073: static PyObject * THPVariable_matrix_power(PyObject* self_, PyObject* args, PyObject* kwargs)
5074: {
5075:   HANDLE_TH_ERRORS
5076:   static PythonArgParser parser({
5077:     "matrix_power(Tensor input, int64_t n, *, Tensor out=None)",
5078:   }, /*traceable=*/true);
5079: 
5080:   ParsedArgs<3> parsed_args;
5081:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5082:   if(_r.has_torch_function()) {
5083:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5084:   }
5085:   if (_r.isNone(2)) {
5086:     // aten::matrix_power(Tensor self, int n) -> Tensor
5087: 
5088:     auto dispatch_matrix_power = [](const at::Tensor & self, int64_t n) -> at::Tensor {
5089:       pybind11::gil_scoped_release no_gil;
5090:       return self.matrix_power(n);
5091:     };
5092:     return wrap(dispatch_matrix_power(_r.tensor(0), _r.toInt64(1)));
5093:   } else {
5094:     // aten::matrix_power.out(Tensor self, int n, *, Tensor(a!) out) -> Tensor(a!)
5095: 
5096:     auto dispatch_matrix_power_out = [](at::Tensor out, const at::Tensor & self, int64_t n) -> at::Tensor {
5097:       pybind11::gil_scoped_release no_gil;
5098:       return at::matrix_power_out(out, self, n);
5099:     };
5100:     return wrap(dispatch_matrix_power_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
5101:   }
5102:   Py_RETURN_NONE;
5103:   END_HANDLE_TH_ERRORS
5104: }
5105: 
5106: \
5107: // _aminmax
5108: static PyObject * THPVariable__aminmax(PyObject* self_, PyObject* args, PyObject* kwargs)
5109: {
5110:   HANDLE_TH_ERRORS
5111:   static PythonArgParser parser({
5112:     "_aminmax(Tensor input)",
5113:     "_aminmax(Tensor input, int64_t dim, bool keepdim=False)",
5114:   }, /*traceable=*/true);
5115: 
5116:   ParsedArgs<3> parsed_args;
5117:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5118:   if(_r.has_torch_function()) {
5119:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5120:   }
```

- EN: The main execution path in this span is carried by `logsumexp_out`, `wrap`, `THPVariable_matrix_power`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `logsumexp_out`, `wrap`, `THPVariable_matrix_power` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5121-5200

```cpp
5121:   switch (_r.idx) {
5122:     case 0: {
5123:       // aten::_aminmax(Tensor self) -> (Tensor, Tensor)
5124: 
5125:       auto dispatch__aminmax = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
5126:         pybind11::gil_scoped_release no_gil;
5127:         return at::_aminmax(self);
5128:       };
5129:       return wrap(dispatch__aminmax(_r.tensor(0)));
5130:     }
5131:     case 1: {
5132:       // aten::_aminmax.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor, Tensor)
5133: 
5134:       auto dispatch__aminmax = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5135:         pybind11::gil_scoped_release no_gil;
5136:         return at::_aminmax(self, dim, keepdim);
5137:       };
5138:       return wrap(dispatch__aminmax(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5139:     }
5140:   }
5141:   Py_RETURN_NONE;
5142:   END_HANDLE_TH_ERRORS
5143: }
5144: 
5145: \
5146: // max
5147: static PyObject * THPVariable_max(PyObject* self_, PyObject* args, PyObject* kwargs)
5148: {
5149:   HANDLE_TH_ERRORS
5150:   static PyTypeObject* NamedTuple = generated::get_max_structseq();
5151:   static PyTypeObject* NamedTuple1 = generated::get_max_out_structseq();
5152:   static PythonArgParser parser({
5153:     "max(Tensor input, *, Tensor out=None)",
5154:     "max(Tensor input, Tensor other, *, Tensor out=None)",
5155:     "max(Tensor input, int64_t dim, bool keepdim=False, *, TensorList[2] out=None)",
5156:     "max(Tensor input, Dimname dim, bool keepdim=False, *, TensorList[2] out=None)",
5157:   }, /*traceable=*/true);
5158: 
5159:   ParsedArgs<4> parsed_args;
5160:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5161:   if(_r.has_torch_function()) {
5162:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5163:   }
5164:   switch (_r.idx) {
5165:     case 0: {
5166:       if (_r.isNone(1)) {
5167:         // aten::max(Tensor self) -> Tensor
5168: 
5169:         auto dispatch_max = [](const at::Tensor & self) -> at::Tensor {
5170:           pybind11::gil_scoped_release no_gil;
5171:           return self.max();
5172:         };
5173:         return wrap(dispatch_max(_r.tensor(0)));
5174:       } else {
5175:         // aten::max.unary_out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
5176: 
5177:         auto dispatch_max_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
5178:           pybind11::gil_scoped_release no_gil;
5179:           return at::max_out(out, self);
5180:         };
5181:         return wrap(dispatch_max_out(_r.tensor(1), _r.tensor(0)));
5182:       }
5183:     }
5184:     case 1: {
5185:       if (_r.isNone(2)) {
5186:         // aten::max.other(Tensor self, Tensor other) -> Tensor
5187: 
5188:         auto dispatch_max = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5189:           pybind11::gil_scoped_release no_gil;
5190:           return self.max(other);
5191:         };
5192:         return wrap(dispatch_max(_r.tensor(0), _r.tensor(1)));
5193:       } else {
5194:         // aten::max.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
5195: 
5196:         auto dispatch_max_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5197:           pybind11::gil_scoped_release no_gil;
5198:           return at::max_out(out, self, other);
5199:         };
5200:         return wrap(dispatch_max_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
```

- EN: The main execution path in this span is carried by `_aminmax`, `wrap`, `THPVariable_max`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_aminmax`, `wrap`, `THPVariable_max` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5201-5280

```cpp
5201:       }
5202:     }
5203:     case 2: {
5204:       if (_r.isNone(3)) {
5205:         // aten::max.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
5206: 
5207:         auto dispatch_max = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5208:           pybind11::gil_scoped_release no_gil;
5209:           return self.max(dim, keepdim);
5210:         };
5211:         return wrap(NamedTuple, dispatch_max(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5212:       } else {
5213:         // aten::max.dim_max(Tensor self, int dim, bool keepdim=False, *, Tensor(a!) max, Tensor(b!) max_values) -> (Tensor(a!) values, Tensor(b!) indices)
5214:         auto out = _r.tensorlist_n<2>(3);
5215:         auto dispatch_max_out = [](at::Tensor & max, at::Tensor & max_values, const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5216:           pybind11::gil_scoped_release no_gil;
5217:           return at::max_out(max, max_values, self, dim, keepdim);
5218:         };
5219:         return wrap(NamedTuple1, dispatch_max_out(out[0], out[1], _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5220:       }
5221:     }
5222:     case 3: {
5223:       if (_r.isNone(3)) {
5224:         // aten::max.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
5225: 
5226:         auto dispatch_max = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5227:           pybind11::gil_scoped_release no_gil;
5228:           return self.max(dim, keepdim);
5229:         };
5230:         return wrap(NamedTuple, dispatch_max(_r.tensor(0), _r.dimname(1), _r.toBool(2)));
5231:       } else {
5232:         // aten::max.names_dim_max(Tensor self, Dimname dim, bool keepdim=False, *, Tensor(a!) max, Tensor(b!) max_values) -> (Tensor(a!) values, Tensor(b!) indices)
5233:         auto out = _r.tensorlist_n<2>(3);
5234:         auto dispatch_max_out = [](at::Tensor & max, at::Tensor & max_values, const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5235:           pybind11::gil_scoped_release no_gil;
5236:           return at::max_out(max, max_values, self, dim, keepdim);
5237:         };
5238:         return wrap(NamedTuple1, dispatch_max_out(out[0], out[1], _r.tensor(0), _r.dimname(1), _r.toBool(2)));
5239:       }
5240:     }
5241:   }
5242:   Py_RETURN_NONE;
5243:   END_HANDLE_TH_ERRORS
5244: }
5245: 
5246: // quantized_max_pool1d
5247: static PyObject * THPVariable_quantized_max_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs)
5248: {
5249:   HANDLE_TH_ERRORS
5250:   static PythonArgParser parser({
5251:     "quantized_max_pool1d(Tensor input, IntArrayRef[1] kernel_size, IntArrayRef[1] stride=None, IntArrayRef[1] padding=0, IntArrayRef[1] dilation=1, bool ceil_mode=False)",
5252:   }, /*traceable=*/true);
5253: 
5254:   ParsedArgs<6> parsed_args;
5255:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5256:   if(_r.has_torch_function()) {
5257:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5258:   }
5259:   // aten::quantized_max_pool1d(Tensor self, int[1] kernel_size, int[1] stride=[], int[1] padding=0, int[1] dilation=1, bool ceil_mode=False) -> Tensor
5260: 
5261:   auto dispatch_quantized_max_pool1d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> at::Tensor {
5262:     pybind11::gil_scoped_release no_gil;
5263:     return at::quantized_max_pool1d(self, kernel_size, stride, padding, dilation, ceil_mode);
5264:   };
5265:   return wrap(dispatch_quantized_max_pool1d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
5266:   Py_RETURN_NONE;
5267:   END_HANDLE_TH_ERRORS
5268: }
5269: 
5270: // quantized_max_pool2d
5271: static PyObject * THPVariable_quantized_max_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
5272: {
5273:   HANDLE_TH_ERRORS
5274:   static PythonArgParser parser({
5275:     "quantized_max_pool2d(Tensor input, IntArrayRef[2] kernel_size, IntArrayRef[2] stride=None, IntArrayRef[2] padding=0, IntArrayRef[2] dilation=1, bool ceil_mode=False)",
5276:   }, /*traceable=*/true);
5277: 
5278:   ParsedArgs<6> parsed_args;
5279:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5280:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `wrap`, `max_out`, `THPVariable_quantized_max_pool1d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `max_out`, `THPVariable_quantized_max_pool1d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5281-5360

```cpp
5281:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5282:   }
5283:   // aten::quantized_max_pool2d(Tensor self, int[2] kernel_size, int[2] stride=[], int[2] padding=0, int[2] dilation=1, bool ceil_mode=False) -> Tensor
5284: 
5285:   auto dispatch_quantized_max_pool2d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> at::Tensor {
5286:     pybind11::gil_scoped_release no_gil;
5287:     return at::quantized_max_pool2d(self, kernel_size, stride, padding, dilation, ceil_mode);
5288:   };
5289:   return wrap(dispatch_quantized_max_pool2d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
5290:   Py_RETURN_NONE;
5291:   END_HANDLE_TH_ERRORS
5292: }
5293: 
5294: \
5295: // mean
5296: static PyObject * THPVariable_mean(PyObject* self_, PyObject* args, PyObject* kwargs)
5297: {
5298:   HANDLE_TH_ERRORS
5299:   static PythonArgParser parser({
5300:     "mean(Tensor input, *, ScalarType? dtype=None, Tensor out=None)",
5301:     "mean(Tensor input, IntArrayRef[1]? dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor out=None)",
5302:     "mean(Tensor input, DimnameList[1] dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor out=None)",
5303:   }, /*traceable=*/true);
5304: 
5305:   ParsedArgs<5> parsed_args;
5306:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5307:   if(_r.has_torch_function()) {
5308:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5309:   }
5310:   switch (_r.idx) {
5311:     case 0: {
5312:       if (_r.isNone(2)) {
5313:         // aten::mean(Tensor self, *, ScalarType? dtype=None) -> Tensor
5314: 
5315:         auto dispatch_mean = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5316:           pybind11::gil_scoped_release no_gil;
5317:           return self.mean(dtype);
5318:         };
5319:         return wrap(dispatch_mean(_r.tensor(0), _r.scalartypeOptional(1)));
5320:       } else {
5321:         // aten::mean.dtype_out(Tensor self, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
5322: 
5323:         auto dispatch_mean_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5324:           pybind11::gil_scoped_release no_gil;
5325:           return at::mean_out(out, self, dtype);
5326:         };
5327:         return wrap(dispatch_mean_out(_r.tensor(2), _r.tensor(0), _r.scalartypeOptional(1)));
5328:       }
5329:     }
5330:     case 1: {
5331:       if (_r.isNone(4)) {
5332:         // aten::mean.dim(Tensor self, int[1]? dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
5333: 
5334:         auto dispatch_mean = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5335:           pybind11::gil_scoped_release no_gil;
5336:           return self.mean(dim, keepdim, dtype);
5337:         };
5338:         return wrap(dispatch_mean(_r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
5339:       } else {
5340:         // aten::mean.out(Tensor self, int[1]? dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
5341: 
5342:         auto dispatch_mean_out = [](at::Tensor out, const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5343:           pybind11::gil_scoped_release no_gil;
5344:           return at::mean_out(out, self, dim, keepdim, dtype);
5345:         };
5346:         return wrap(dispatch_mean_out(_r.tensor(4), _r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
5347:       }
5348:     }
5349:     case 2: {
5350:       if (_r.isNone(4)) {
5351:         // aten::mean.names_dim(Tensor self, Dimname[1] dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
5352: 
5353:         auto dispatch_mean = [](const at::Tensor & self, at::DimnameList dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5354:           pybind11::gil_scoped_release no_gil;
5355:           return self.mean(dim, keepdim, dtype);
5356:         };
5357:         return wrap(dispatch_mean(_r.tensor(0), _r.dimnamelist(1), _r.toBool(2), _r.scalartypeOptional(3)));
5358:       } else {
5359:         // aten::mean.names_out(Tensor self, Dimname[1] dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
5360: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `quantized_max_pool2d`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `quantized_max_pool2d`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5361-5440

```cpp
5361:         auto dispatch_mean_out = [](at::Tensor out, const at::Tensor & self, at::DimnameList dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5362:           pybind11::gil_scoped_release no_gil;
5363:           return at::mean_out(out, self, dim, keepdim, dtype);
5364:         };
5365:         return wrap(dispatch_mean_out(_r.tensor(4), _r.tensor(0), _r.dimnamelist(1), _r.toBool(2), _r.scalartypeOptional(3)));
5366:       }
5367:     }
5368:   }
5369:   Py_RETURN_NONE;
5370:   END_HANDLE_TH_ERRORS
5371: }
5372: 
5373: // nanmean
5374: static PyObject * THPVariable_nanmean(PyObject* self_, PyObject* args, PyObject* kwargs)
5375: {
5376:   HANDLE_TH_ERRORS
5377:   static PythonArgParser parser({
5378:     "nanmean(Tensor input, IntArrayRef[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None, Tensor out=None)",
5379:   }, /*traceable=*/true);
5380: 
5381:   ParsedArgs<5> parsed_args;
5382:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5383:   if(_r.has_torch_function()) {
5384:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5385:   }
5386:   if (_r.isNone(4)) {
5387:     // aten::nanmean(Tensor self, int[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
5388: 
5389:     auto dispatch_nanmean = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5390:       pybind11::gil_scoped_release no_gil;
5391:       return self.nanmean(dim, keepdim, dtype);
5392:     };
5393:     return wrap(dispatch_nanmean(_r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
5394:   } else {
5395:     // aten::nanmean.out(Tensor self, int[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
5396: 
5397:     auto dispatch_nanmean_out = [](at::Tensor out, const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
5398:       pybind11::gil_scoped_release no_gil;
5399:       return at::nanmean_out(out, self, dim, keepdim, dtype);
5400:     };
5401:     return wrap(dispatch_nanmean_out(_r.tensor(4), _r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
5402:   }
5403:   Py_RETURN_NONE;
5404:   END_HANDLE_TH_ERRORS
5405: }
5406: 
5407: \
5408: // nanmedian
5409: static PyObject * THPVariable_nanmedian(PyObject* self_, PyObject* args, PyObject* kwargs)
5410: {
5411:   HANDLE_TH_ERRORS
5412:   static PyTypeObject* NamedTuple = generated::get_nanmedian_structseq();
5413:   static PyTypeObject* NamedTuple1 = generated::get_nanmedian_out_structseq();
5414:   static PythonArgParser parser({
5415:     "nanmedian(Tensor input)",
5416:     "nanmedian(Tensor input, int64_t dim, bool keepdim=False, *, TensorList[2] out=None)",
5417:     "nanmedian(Tensor input, Dimname dim, bool keepdim=False, *, TensorList[2] out=None)",
5418:   }, /*traceable=*/true);
5419: 
5420:   ParsedArgs<4> parsed_args;
5421:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5422:   if(_r.has_torch_function()) {
5423:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5424:   }
5425:   switch (_r.idx) {
5426:     case 0: {
5427:       // aten::nanmedian(Tensor self) -> Tensor
5428: 
5429:       auto dispatch_nanmedian = [](const at::Tensor & self) -> at::Tensor {
5430:         pybind11::gil_scoped_release no_gil;
5431:         return self.nanmedian();
5432:       };
5433:       return wrap(dispatch_nanmedian(_r.tensor(0)));
5434:     }
5435:     case 1: {
5436:       if (_r.isNone(3)) {
5437:         // aten::nanmedian.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
5438: 
5439:         auto dispatch_nanmedian = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5440:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `mean_out`, `wrap`, `THPVariable_nanmean`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `mean_out`, `wrap`, `THPVariable_nanmean` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5441-5520

```cpp
5441:           return self.nanmedian(dim, keepdim);
5442:         };
5443:         return wrap(NamedTuple, dispatch_nanmedian(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5444:       } else {
5445:         // aten::nanmedian.dim_values(Tensor self, int dim, bool keepdim=False, *, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
5446:         auto out = _r.tensorlist_n<2>(3);
5447:         auto dispatch_nanmedian_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5448:           pybind11::gil_scoped_release no_gil;
5449:           return at::nanmedian_out(values, indices, self, dim, keepdim);
5450:         };
5451:         return wrap(NamedTuple1, dispatch_nanmedian_out(out[0], out[1], _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5452:       }
5453:     }
5454:     case 2: {
5455:       if (_r.isNone(3)) {
5456:         // aten::nanmedian.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
5457: 
5458:         auto dispatch_nanmedian = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5459:           pybind11::gil_scoped_release no_gil;
5460:           return self.nanmedian(dim, keepdim);
5461:         };
5462:         return wrap(NamedTuple, dispatch_nanmedian(_r.tensor(0), _r.dimname(1), _r.toBool(2)));
5463:       } else {
5464:         // aten::nanmedian.names_dim_values(Tensor self, Dimname dim, bool keepdim=False, *, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
5465:         auto out = _r.tensorlist_n<2>(3);
5466:         auto dispatch_nanmedian_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5467:           pybind11::gil_scoped_release no_gil;
5468:           return at::nanmedian_out(values, indices, self, dim, keepdim);
5469:         };
5470:         return wrap(NamedTuple1, dispatch_nanmedian_out(out[0], out[1], _r.tensor(0), _r.dimname(1), _r.toBool(2)));
5471:       }
5472:     }
5473:   }
5474:   Py_RETURN_NONE;
5475:   END_HANDLE_TH_ERRORS
5476: }
5477: 
5478: \
5479: // min
5480: static PyObject * THPVariable_min(PyObject* self_, PyObject* args, PyObject* kwargs)
5481: {
5482:   HANDLE_TH_ERRORS
5483:   static PyTypeObject* NamedTuple = generated::get_min_structseq();
5484:   static PyTypeObject* NamedTuple1 = generated::get_min_out_structseq();
5485:   static PythonArgParser parser({
5486:     "min(Tensor input, *, Tensor out=None)",
5487:     "min(Tensor input, Tensor other, *, Tensor out=None)",
5488:     "min(Tensor input, int64_t dim, bool keepdim=False, *, TensorList[2] out=None)",
5489:     "min(Tensor input, Dimname dim, bool keepdim=False, *, TensorList[2] out=None)",
5490:   }, /*traceable=*/true);
5491: 
5492:   ParsedArgs<4> parsed_args;
5493:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5494:   if(_r.has_torch_function()) {
5495:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5496:   }
5497:   switch (_r.idx) {
5498:     case 0: {
5499:       if (_r.isNone(1)) {
5500:         // aten::min(Tensor self) -> Tensor
5501: 
5502:         auto dispatch_min = [](const at::Tensor & self) -> at::Tensor {
5503:           pybind11::gil_scoped_release no_gil;
5504:           return self.min();
5505:         };
5506:         return wrap(dispatch_min(_r.tensor(0)));
5507:       } else {
5508:         // aten::min.unary_out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
5509: 
5510:         auto dispatch_min_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
5511:           pybind11::gil_scoped_release no_gil;
5512:           return at::min_out(out, self);
5513:         };
5514:         return wrap(dispatch_min_out(_r.tensor(1), _r.tensor(0)));
5515:       }
5516:     }
5517:     case 1: {
5518:       if (_r.isNone(2)) {
5519:         // aten::min.other(Tensor self, Tensor other) -> Tensor
5520: 
```

- EN: The main execution path in this span is carried by `wrap`, `nanmedian_out`, `THPVariable_min`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `nanmedian_out`, `THPVariable_min` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5600

```cpp
5521:         auto dispatch_min = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5522:           pybind11::gil_scoped_release no_gil;
5523:           return self.min(other);
5524:         };
5525:         return wrap(dispatch_min(_r.tensor(0), _r.tensor(1)));
5526:       } else {
5527:         // aten::min.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
5528: 
5529:         auto dispatch_min_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5530:           pybind11::gil_scoped_release no_gil;
5531:           return at::min_out(out, self, other);
5532:         };
5533:         return wrap(dispatch_min_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
5534:       }
5535:     }
5536:     case 2: {
5537:       if (_r.isNone(3)) {
5538:         // aten::min.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
5539: 
5540:         auto dispatch_min = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5541:           pybind11::gil_scoped_release no_gil;
5542:           return self.min(dim, keepdim);
5543:         };
5544:         return wrap(NamedTuple, dispatch_min(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5545:       } else {
5546:         // aten::min.dim_min(Tensor self, int dim, bool keepdim=False, *, Tensor(a!) min, Tensor(b!) min_indices) -> (Tensor(a!) values, Tensor(b!) indices)
5547:         auto out = _r.tensorlist_n<2>(3);
5548:         auto dispatch_min_out = [](at::Tensor & min, at::Tensor & min_indices, const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5549:           pybind11::gil_scoped_release no_gil;
5550:           return at::min_out(min, min_indices, self, dim, keepdim);
5551:         };
5552:         return wrap(NamedTuple1, dispatch_min_out(out[0], out[1], _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5553:       }
5554:     }
5555:     case 3: {
5556:       if (_r.isNone(3)) {
5557:         // aten::min.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
5558: 
5559:         auto dispatch_min = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5560:           pybind11::gil_scoped_release no_gil;
5561:           return self.min(dim, keepdim);
5562:         };
5563:         return wrap(NamedTuple, dispatch_min(_r.tensor(0), _r.dimname(1), _r.toBool(2)));
5564:       } else {
5565:         // aten::min.names_dim_min(Tensor self, Dimname dim, bool keepdim=False, *, Tensor(a!) min, Tensor(b!) min_indices) -> (Tensor(a!) values, Tensor(b!) indices)
5566:         auto out = _r.tensorlist_n<2>(3);
5567:         auto dispatch_min_out = [](at::Tensor & min, at::Tensor & min_indices, const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
5568:           pybind11::gil_scoped_release no_gil;
5569:           return at::min_out(min, min_indices, self, dim, keepdim);
5570:         };
5571:         return wrap(NamedTuple1, dispatch_min_out(out[0], out[1], _r.tensor(0), _r.dimname(1), _r.toBool(2)));
5572:       }
5573:     }
5574:   }
5575:   Py_RETURN_NONE;
5576:   END_HANDLE_TH_ERRORS
5577: }
5578: 
5579: // amin
5580: static PyObject * THPVariable_amin(PyObject* self_, PyObject* args, PyObject* kwargs)
5581: {
5582:   HANDLE_TH_ERRORS
5583:   static PythonArgParser parser({
5584:     "amin(Tensor input, IntArrayRef[1] dim=None, bool keepdim=False, *, Tensor out=None)",
5585:   }, /*traceable=*/true);
5586: 
5587:   ParsedArgs<4> parsed_args;
5588:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5589:   if(_r.has_torch_function()) {
5590:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5591:   }
5592:   if (_r.isNone(3)) {
5593:     // aten::amin(Tensor self, int[1] dim=[], bool keepdim=False) -> Tensor
5594: 
5595:     auto dispatch_amin = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
5596:       pybind11::gil_scoped_release no_gil;
5597:       return self.amin(dim, keepdim);
5598:     };
5599:     return wrap(dispatch_amin(_r.tensor(0), _r.intlist(1), _r.toBool(2)));
5600:   } else {
```

- EN: The main execution path in this span is carried by `wrap`, `min_out`, `THPVariable_amin`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `min_out`, `THPVariable_amin` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5601-5680

```cpp
5601:     // aten::amin.out(Tensor self, int[1] dim=[], bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
5602: 
5603:     auto dispatch_amin_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
5604:       pybind11::gil_scoped_release no_gil;
5605:       return at::amin_out(out, self, dim, keepdim);
5606:     };
5607:     return wrap(dispatch_amin_out(_r.tensor(3), _r.tensor(0), _r.intlist(1), _r.toBool(2)));
5608:   }
5609:   Py_RETURN_NONE;
5610:   END_HANDLE_TH_ERRORS
5611: }
5612: 
5613: // mkldnn_convolution
5614: static PyObject * THPVariable_mkldnn_convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
5615: {
5616:   HANDLE_TH_ERRORS
5617:   static PythonArgParser parser({
5618:     "mkldnn_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups)",
5619:   }, /*traceable=*/true);
5620: 
5621:   ParsedArgs<7> parsed_args;
5622:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5623:   if(_r.has_torch_function()) {
5624:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5625:   }
5626:   // aten::mkldnn_convolution(Tensor self, Tensor weight, Tensor? bias, SymInt[] padding, SymInt[] stride, SymInt[] dilation, SymInt groups) -> Tensor
5627: 
5628:   auto dispatch_mkldnn_convolution = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
5629:     pybind11::gil_scoped_release no_gil;
5630:     return at::mkldnn_convolution_symint(self, weight, bias, padding, stride, dilation, groups);
5631:   };
5632:   return wrap(dispatch_mkldnn_convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
5633:   Py_RETURN_NONE;
5634:   END_HANDLE_TH_ERRORS
5635: }
5636: 
5637: // miopen_batch_norm
5638: static PyObject * THPVariable_miopen_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
5639: {
5640:   HANDLE_TH_ERRORS
5641:   static PythonArgParser parser({
5642:     "miopen_batch_norm(Tensor input, Tensor weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, double exponential_average_factor, double epsilon)",
5643:   }, /*traceable=*/true);
5644: 
5645:   ParsedArgs<8> parsed_args;
5646:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5647:   if(_r.has_torch_function()) {
5648:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5649:   }
5650:   // aten::miopen_batch_norm(Tensor input, Tensor weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float exponential_average_factor, float epsilon) -> (Tensor, Tensor, Tensor)
5651: 
5652:   auto dispatch_miopen_batch_norm = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, bool training, double exponential_average_factor, double epsilon) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
5653:     pybind11::gil_scoped_release no_gil;
5654:     return at::miopen_batch_norm(input, weight, bias, running_mean, running_var, training, exponential_average_factor, epsilon);
5655:   };
5656:   return wrap(dispatch_miopen_batch_norm(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toBool(5), _r.toDouble(6), _r.toDouble(7)));
5657:   Py_RETURN_NONE;
5658:   END_HANDLE_TH_ERRORS
5659: }
5660: 
5661: // miopen_depthwise_convolution
5662: static PyObject * THPVariable_miopen_depthwise_convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
5663: {
5664:   HANDLE_TH_ERRORS
5665:   static PythonArgParser parser({
5666:     "miopen_depthwise_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups, bool benchmark, bool deterministic)",
5667:   }, /*traceable=*/true);
5668: 
5669:   ParsedArgs<9> parsed_args;
5670:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5671:   if(_r.has_torch_function()) {
5672:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5673:   }
5674:   // aten::miopen_depthwise_convolution(Tensor self, Tensor weight, Tensor? bias, SymInt[] padding, SymInt[] stride, SymInt[] dilation, SymInt groups, bool benchmark, bool deterministic) -> Tensor
5675: 
5676:   auto dispatch_miopen_depthwise_convolution = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups, bool benchmark, bool deterministic) -> at::Tensor {
5677:     pybind11::gil_scoped_release no_gil;
5678:     return at::miopen_depthwise_convolution_symint(self, weight, bias, padding, stride, dilation, groups, benchmark, deterministic);
5679:   };
5680:   return wrap(dispatch_miopen_depthwise_convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6), _r.toBool(7), _r.toBool(8)));
```

- EN: The main execution path in this span is carried by `amin_out`, `wrap`, `THPVariable_mkldnn_convolution`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `amin_out`, `wrap`, `THPVariable_mkldnn_convolution` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5681-5760

```cpp
5681:   Py_RETURN_NONE;
5682:   END_HANDLE_TH_ERRORS
5683: }
5684: 
5685: // _weight_int8pack_mm
5686: static PyObject * THPVariable__weight_int8pack_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
5687: {
5688:   HANDLE_TH_ERRORS
5689:   static PythonArgParser parser({
5690:     "_weight_int8pack_mm(Tensor input, Tensor mat2, Tensor scales)",
5691:   }, /*traceable=*/true);
5692: 
5693:   ParsedArgs<3> parsed_args;
5694:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5695:   if(_r.has_torch_function()) {
5696:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5697:   }
5698:   // aten::_weight_int8pack_mm(Tensor self, Tensor mat2, Tensor scales) -> Tensor
5699: 
5700:   auto dispatch__weight_int8pack_mm = [](const at::Tensor & self, const at::Tensor & mat2, const at::Tensor & scales) -> at::Tensor {
5701:     pybind11::gil_scoped_release no_gil;
5702:     return at::_weight_int8pack_mm(self, mat2, scales);
5703:   };
5704:   return wrap(dispatch__weight_int8pack_mm(_r.tensor(0), _r.tensor(1), _r.tensor(2)));
5705:   Py_RETURN_NONE;
5706:   END_HANDLE_TH_ERRORS
5707: }
5708: 
5709: // mv
5710: static PyObject * THPVariable_mv(PyObject* self_, PyObject* args, PyObject* kwargs)
5711: {
5712:   HANDLE_TH_ERRORS
5713:   static PythonArgParser parser({
5714:     "mv(Tensor input, Tensor vec, *, Tensor out=None)",
5715:   }, /*traceable=*/true);
5716: 
5717:   ParsedArgs<3> parsed_args;
5718:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5719:   if(_r.has_torch_function()) {
5720:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5721:   }
5722:   if (_r.isNone(2)) {
5723:     // aten::mv(Tensor self, Tensor vec) -> Tensor
5724: 
5725:     auto dispatch_mv = [](const at::Tensor & self, const at::Tensor & vec) -> at::Tensor {
5726:       pybind11::gil_scoped_release no_gil;
5727:       return self.mv(vec);
5728:     };
5729:     return wrap(dispatch_mv(_r.tensor(0), _r.tensor(1)));
5730:   } else {
5731:     // aten::mv.out(Tensor self, Tensor vec, *, Tensor(a!) out) -> Tensor(a!)
5732: 
5733:     auto dispatch_mv_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & vec) -> at::Tensor {
5734:       pybind11::gil_scoped_release no_gil;
5735:       return at::mv_out(out, self, vec);
5736:     };
5737:     return wrap(dispatch_mv_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
5738:   }
5739:   Py_RETURN_NONE;
5740:   END_HANDLE_TH_ERRORS
5741: }
5742: 
5743: \
5744: // narrow
5745: static PyObject * THPVariable_narrow(PyObject* self_, PyObject* args, PyObject* kwargs)
5746: {
5747:   HANDLE_TH_ERRORS
5748:   static PythonArgParser parser({
5749:     "narrow(Tensor input, int64_t dim, Tensor start, SymInt length)",
5750:     "narrow(Tensor input, int64_t dim, SymInt start, SymInt length)",
5751:   }, /*traceable=*/true);
5752: 
5753:   ParsedArgs<4> parsed_args;
5754:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5755:   if(_r.has_torch_function()) {
5756:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5757:   }
5758:   switch (_r.idx) {
5759:     case 0: {
5760:       // aten::narrow.Tensor(Tensor(a) self, int dim, Tensor start, SymInt length) -> Tensor(a)
```

- EN: The main execution path in this span is carried by `THPVariable__weight_int8pack_mm`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__weight_int8pack_mm`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5840

```cpp
5761: 
5762:       auto dispatch_narrow = [](const at::Tensor & self, int64_t dim, const at::Tensor & start, c10::SymInt length) -> at::Tensor {
5763:         pybind11::gil_scoped_release no_gil;
5764:         return self.narrow_symint(dim, start, length);
5765:       };
5766:       return wrap(dispatch_narrow(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.toSymInt(3)));
5767:     }
5768:     case 1: {
5769:       // aten::narrow(Tensor(a) self, int dim, SymInt start, SymInt length) -> Tensor(a)
5770: 
5771:       auto dispatch_narrow = [](const at::Tensor & self, int64_t dim, c10::SymInt start, c10::SymInt length) -> at::Tensor {
5772:         pybind11::gil_scoped_release no_gil;
5773:         return self.narrow_symint(dim, start, length);
5774:       };
5775:       return wrap(dispatch_narrow(_r.tensor(0), _r.toInt64(1), _r.toSymInt(2), _r.toSymInt(3)));
5776:     }
5777:   }
5778:   Py_RETURN_NONE;
5779:   END_HANDLE_TH_ERRORS
5780: }
5781: 
5782: // batch_norm_stats
5783: static PyObject * THPVariable_batch_norm_stats(PyObject* self_, PyObject* args, PyObject* kwargs)
5784: {
5785:   HANDLE_TH_ERRORS
5786:   static PythonArgParser parser({
5787:     "batch_norm_stats(Tensor input, double eps)",
5788:   }, /*traceable=*/true);
5789: 
5790:   ParsedArgs<2> parsed_args;
5791:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5792:   if(_r.has_torch_function()) {
5793:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5794:   }
5795:   // aten::batch_norm_stats(Tensor input, float eps) -> (Tensor, Tensor)
5796: 
5797:   auto dispatch_batch_norm_stats = [](const at::Tensor & input, double eps) -> ::std::tuple<at::Tensor,at::Tensor> {
5798:     pybind11::gil_scoped_release no_gil;
5799:     return at::batch_norm_stats(input, eps);
5800:   };
5801:   return wrap(dispatch_batch_norm_stats(_r.tensor(0), _r.toDouble(1)));
5802:   Py_RETURN_NONE;
5803:   END_HANDLE_TH_ERRORS
5804: }
5805: 
5806: // batch_norm_update_stats
5807: static PyObject * THPVariable_batch_norm_update_stats(PyObject* self_, PyObject* args, PyObject* kwargs)
5808: {
5809:   HANDLE_TH_ERRORS
5810:   static PythonArgParser parser({
5811:     "batch_norm_update_stats(Tensor input, Tensor? running_mean, Tensor? running_var, double momentum)",
5812:   }, /*traceable=*/true);
5813: 
5814:   ParsedArgs<4> parsed_args;
5815:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5816:   if(_r.has_torch_function()) {
5817:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5818:   }
5819:   // aten::batch_norm_update_stats(Tensor input, Tensor? running_mean, Tensor? running_var, float momentum) -> (Tensor, Tensor)
5820: 
5821:   auto dispatch_batch_norm_update_stats = [](const at::Tensor & input, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, double momentum) -> ::std::tuple<at::Tensor,at::Tensor> {
5822:     pybind11::gil_scoped_release no_gil;
5823:     return at::batch_norm_update_stats(input, running_mean, running_var, momentum);
5824:   };
5825:   return wrap(dispatch_batch_norm_update_stats(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.toDouble(3)));
5826:   Py_RETURN_NONE;
5827:   END_HANDLE_TH_ERRORS
5828: }
5829: 
5830: // _nnpack_available
5831: static PyObject * THPVariable__nnpack_available(PyObject* self_, PyObject* args)
5832: {
5833:   HANDLE_TH_ERRORS
5834: 
5835:   // aten::_nnpack_available() -> bool
5836: 
5837:   auto dispatch__nnpack_available = []() -> bool {
5838:     pybind11::gil_scoped_release no_gil;
5839:     return at::_nnpack_available();
5840:   };
```

- EN: The main execution path in this span is carried by `wrap`, `narrow`, `THPVariable_batch_norm_stats`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `narrow`, `THPVariable_batch_norm_stats` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5841-5920

```cpp
5841:   return wrap(dispatch__nnpack_available());
5842:   END_HANDLE_TH_ERRORS
5843: }
5844: 
5845: // _nnpack_spatial_convolution
5846: static PyObject * THPVariable__nnpack_spatial_convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
5847: {
5848:   HANDLE_TH_ERRORS
5849:   static PythonArgParser parser({
5850:     "_nnpack_spatial_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef[2] padding, SymIntArrayRef[2] stride=1)",
5851:   }, /*traceable=*/true);
5852: 
5853:   ParsedArgs<5> parsed_args;
5854:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5855:   if(_r.has_torch_function()) {
5856:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5857:   }
5858:   // aten::_nnpack_spatial_convolution(Tensor input, Tensor weight, Tensor? bias, SymInt[2] padding, SymInt[2] stride=1) -> Tensor
5859: 
5860:   auto dispatch__nnpack_spatial_convolution = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride) -> at::Tensor {
5861:     pybind11::gil_scoped_release no_gil;
5862:     return at::_nnpack_spatial_convolution_symint(input, weight, bias, padding, stride);
5863:   };
5864:   return wrap(dispatch__nnpack_spatial_convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4)));
5865:   Py_RETURN_NONE;
5866:   END_HANDLE_TH_ERRORS
5867: }
5868: 
5869: // _euclidean_dist
5870: static PyObject * THPVariable__euclidean_dist(PyObject* self_, PyObject* args, PyObject* kwargs)
5871: {
5872:   HANDLE_TH_ERRORS
5873:   static PythonArgParser parser({
5874:     "_euclidean_dist(Tensor x1, Tensor x2)",
5875:   }, /*traceable=*/true);
5876: 
5877:   ParsedArgs<2> parsed_args;
5878:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5879:   if(_r.has_torch_function()) {
5880:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5881:   }
5882:   // aten::_euclidean_dist(Tensor x1, Tensor x2) -> Tensor
5883: 
5884:   auto dispatch__euclidean_dist = [](const at::Tensor & x1, const at::Tensor & x2) -> at::Tensor {
5885:     pybind11::gil_scoped_release no_gil;
5886:     return at::_euclidean_dist(x1, x2);
5887:   };
5888:   return wrap(dispatch__euclidean_dist(_r.tensor(0), _r.tensor(1)));
5889:   Py_RETURN_NONE;
5890:   END_HANDLE_TH_ERRORS
5891: }
5892: 
5893: // permute
5894: static PyObject * THPVariable_permute(PyObject* self_, PyObject* args, PyObject* kwargs)
5895: {
5896:   HANDLE_TH_ERRORS
5897:   static PythonArgParser parser({
5898:     "permute(Tensor input, IntArrayRef dims)",
5899:   }, /*traceable=*/true);
5900: 
5901:   ParsedArgs<2> parsed_args;
5902:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5903:   if(_r.has_torch_function()) {
5904:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5905:   }
5906:   // aten::permute(Tensor(a) self, int[] dims) -> Tensor(a)
5907: 
5908:   auto dispatch_permute = [](const at::Tensor & self, at::IntArrayRef dims) -> at::Tensor {
5909:     pybind11::gil_scoped_release no_gil;
5910:     return self.permute(dims);
5911:   };
5912:   return wrap(dispatch_permute(_r.tensor(0), _r.intlist(1)));
5913:   Py_RETURN_NONE;
5914:   END_HANDLE_TH_ERRORS
5915: }
5916: 
5917: // pixel_shuffle
5918: static PyObject * THPVariable_pixel_shuffle(PyObject* self_, PyObject* args, PyObject* kwargs)
5919: {
5920:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__nnpack_spatial_convolution`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__nnpack_spatial_convolution`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5921-6000

```cpp
5921:   static PythonArgParser parser({
5922:     "pixel_shuffle(Tensor input, int64_t upscale_factor)",
5923:   }, /*traceable=*/true);
5924: 
5925:   ParsedArgs<2> parsed_args;
5926:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5927:   if(_r.has_torch_function()) {
5928:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5929:   }
5930:   // aten::pixel_shuffle(Tensor self, int upscale_factor) -> Tensor
5931: 
5932:   auto dispatch_pixel_shuffle = [](const at::Tensor & self, int64_t upscale_factor) -> at::Tensor {
5933:     pybind11::gil_scoped_release no_gil;
5934:     return at::pixel_shuffle(self, upscale_factor);
5935:   };
5936:   return wrap(dispatch_pixel_shuffle(_r.tensor(0), _r.toInt64(1)));
5937:   Py_RETURN_NONE;
5938:   END_HANDLE_TH_ERRORS
5939: }
5940: 
5941: // native_channel_shuffle
5942: static PyObject * THPVariable_native_channel_shuffle(PyObject* self_, PyObject* args, PyObject* kwargs)
5943: {
5944:   HANDLE_TH_ERRORS
5945:   static PythonArgParser parser({
5946:     "native_channel_shuffle(Tensor input, SymInt groups)",
5947:   }, /*traceable=*/true);
5948: 
5949:   ParsedArgs<2> parsed_args;
5950:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5951:   if(_r.has_torch_function()) {
5952:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5953:   }
5954:   // aten::native_channel_shuffle(Tensor self, SymInt groups) -> Tensor
5955: 
5956:   auto dispatch_native_channel_shuffle = [](const at::Tensor & self, c10::SymInt groups) -> at::Tensor {
5957:     pybind11::gil_scoped_release no_gil;
5958:     return at::native_channel_shuffle_symint(self, groups);
5959:   };
5960:   return wrap(dispatch_native_channel_shuffle(_r.tensor(0), _r.toSymInt(1)));
5961:   Py_RETURN_NONE;
5962:   END_HANDLE_TH_ERRORS
5963: }
5964: 
5965: // _pin_memory
5966: static PyObject * THPVariable__pin_memory(PyObject* self_, PyObject* args, PyObject* kwargs)
5967: {
5968:   HANDLE_TH_ERRORS
5969:   static PythonArgParser parser({
5970:     "_pin_memory(Tensor input, Device? device=None)",
5971:   }, /*traceable=*/true);
5972: 
5973:   ParsedArgs<2> parsed_args;
5974:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5975:   if(_r.has_torch_function()) {
5976:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5977:   }
5978:   // aten::_pin_memory(Tensor self, Device? device=None) -> Tensor
5979: 
5980:   auto dispatch__pin_memory = [](const at::Tensor & self, ::std::optional<at::Device> device) -> at::Tensor {
5981:     pybind11::gil_scoped_release no_gil;
5982:     return at::_pin_memory(self, device);
5983:   };
5984:   return wrap(dispatch__pin_memory(_r.tensor(0), _r.deviceOptional(1)));
5985:   Py_RETURN_NONE;
5986:   END_HANDLE_TH_ERRORS
5987: }
5988: 
5989: // rad2deg
5990: static PyObject * THPVariable_rad2deg(PyObject* self_, PyObject* args, PyObject* kwargs)
5991: {
5992:   HANDLE_TH_ERRORS
5993:   static PythonArgParser parser({
5994:     "rad2deg(Tensor input, *, Tensor out=None)",
5995:   }, /*traceable=*/true);
5996: 
5997:   ParsedArgs<2> parsed_args;
5998:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5999:   if(_r.has_torch_function()) {
6000:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `pixel_shuffle`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `pixel_shuffle` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6001-6080

```cpp
6001:   }
6002:   if (_r.isNone(1)) {
6003:     // aten::rad2deg(Tensor self) -> Tensor
6004: 
6005:     auto dispatch_rad2deg = [](const at::Tensor & self) -> at::Tensor {
6006:       pybind11::gil_scoped_release no_gil;
6007:       return self.rad2deg();
6008:     };
6009:     return wrap(dispatch_rad2deg(_r.tensor(0)));
6010:   } else {
6011:     // aten::rad2deg.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6012: 
6013:     auto dispatch_rad2deg_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6014:       pybind11::gil_scoped_release no_gil;
6015:       return at::rad2deg_out(out, self);
6016:     };
6017:     return wrap(dispatch_rad2deg_out(_r.tensor(1), _r.tensor(0)));
6018:   }
6019:   Py_RETURN_NONE;
6020:   END_HANDLE_TH_ERRORS
6021: }
6022: 
6023: // rad2deg_
6024: static PyObject * THPVariable_rad2deg_(PyObject* self_, PyObject* args, PyObject* kwargs)
6025: {
6026:   HANDLE_TH_ERRORS
6027:   static PythonArgParser parser({
6028:     "rad2deg_(Tensor input)",
6029:   }, /*traceable=*/true);
6030: 
6031:   ParsedArgs<1> parsed_args;
6032:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6033:   if(_r.has_torch_function()) {
6034:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6035:   }
6036:   // aten::rad2deg_(Tensor(a!) self) -> Tensor(a!)
6037: 
6038:   auto dispatch_rad2deg_ = [](at::Tensor self) -> at::Tensor {
6039:     pybind11::gil_scoped_release no_gil;
6040:     return self.rad2deg_();
6041:   };
6042:   return wrap(dispatch_rad2deg_(_r.tensor(0)));
6043:   Py_RETURN_NONE;
6044:   END_HANDLE_TH_ERRORS
6045: }
6046: 
6047: \
6048: // rand_like
6049: static PyObject * THPVariable_rand_like(PyObject* self_, PyObject* args, PyObject* kwargs)
6050: {
6051:   HANDLE_TH_ERRORS
6052:   static PythonArgParser parser({
6053:     "rand_like(Tensor input, *, Generator? generator, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6054:     "rand_like(Tensor input, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6055:   }, /*traceable=*/true);
6056: 
6057:   ParsedArgs<8> parsed_args;
6058:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6059:   if(_r.has_torch_function()) {
6060:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6061:   }
6062:   switch (_r.idx) {
6063:     case 0: {
6064:       // aten::rand_like.generator(Tensor self, *, Generator? generator, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6065:       auto self = _r.tensor(0);
6066:       const auto options = TensorOptions()
6067:           .dtype(_r.scalartypeOptional(3))
6068:           .device(_r.deviceOptional(5))
6069:           .layout(_r.layoutOptional(4))
6070:           .requires_grad(_r.toBool(7))
6071:           .pinned_memory(_r.toBool(6));
6072:       torch::utils::maybe_initialize_device(options);
6073: 
6074:       auto dispatch_rand_like = [](const at::Tensor & self, ::std::optional<at::Generator> generator, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6075:         pybind11::gil_scoped_release no_gil;
6076:         return torch::rand_like(self, generator, options, memory_format);
6077:       };
6078:       return wrap(dispatch_rand_like(self, _r.generator(1), options, _r.memoryformatOptional(2)));
6079:     }
6080:     case 1: {
```

- EN: The main execution path in this span is carried by `rad2deg`, `wrap`, `rad2deg_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `rad2deg`, `wrap`, `rad2deg_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6081-6160

```cpp
6081:       // aten::rand_like(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6082:       auto self = _r.tensor(0);
6083:       const auto options = TensorOptions()
6084:           .dtype(_r.scalartypeOptional(2))
6085:           .device(_r.deviceOptional(4))
6086:           .layout(_r.layoutOptional(3))
6087:           .requires_grad(_r.toBool(6))
6088:           .pinned_memory(_r.toBool(5));
6089:       torch::utils::maybe_initialize_device(options);
6090: 
6091:       auto dispatch_rand_like = [](const at::Tensor & self, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6092:         pybind11::gil_scoped_release no_gil;
6093:         return torch::rand_like(self, options, memory_format);
6094:       };
6095:       return wrap(dispatch_rand_like(self, options, _r.memoryformatOptional(1)));
6096:     }
6097:   }
6098:   Py_RETURN_NONE;
6099:   END_HANDLE_TH_ERRORS
6100: }
6101: 
6102: \
6103: // randint
6104: static PyObject * THPVariable_randint(PyObject* self_, PyObject* args, PyObject* kwargs)
6105: {
6106:   HANDLE_TH_ERRORS
6107:   static PythonArgParser parser({
6108:     "randint(SymInt high, SymIntArrayRef size, *, Generator? generator, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6109:     "randint(SymInt high, SymIntArrayRef size, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6110:     "randint(SymInt low, SymInt high, SymIntArrayRef size, *, Generator? generator, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6111:     "randint(SymInt low, SymInt high, SymIntArrayRef size, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6112:   }, /*traceable=*/true);
6113: 
6114:   ParsedArgs<10> parsed_args;
6115:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6116:   if(_r.has_torch_function()) {
6117:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6118:   }
6119:   switch (_r.idx) {
6120:     case 0: {
6121:       if (_r.isNone(3)) {
6122:         // aten::randint.generator(SymInt high, SymInt[] size, *, Generator? generator, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6123:         const auto options = TensorOptions()
6124:             .dtype(_r.scalartypeWithDefault(4, at::kLong))
6125:             .device(_r.deviceWithDefault(6, torch::tensors::get_default_device()))
6126:             .layout(_r.layoutOptional(5))
6127:             .requires_grad(_r.toBool(8))
6128:             .pinned_memory(_r.toBool(7));
6129:         torch::utils::maybe_initialize_device(options);
6130: 
6131:         auto dispatch_randint = [](c10::SymInt high, c10::SymIntArrayRef size, ::std::optional<at::Generator> generator, at::TensorOptions options) -> at::Tensor {
6132:           pybind11::gil_scoped_release no_gil;
6133:           return torch::randint_symint(high, size, generator, options);
6134:         };
6135:         return wrap(dispatch_randint(_r.toSymInt(0), _r.symintlist(1), _r.generator(2), options));
6136:       } else {
6137:         // aten::randint.generator_out(SymInt high, SymInt[] size, *, Generator? generator, Tensor(a!) out) -> Tensor(a!)
6138:         check_out_type_matches(_r.tensor(3), _r.scalartypeWithDefault(4, at::kLong),
6139:                                _r.isNone(4), _r.layoutOptional(5),
6140:                                _r.deviceWithDefault(6, torch::tensors::get_default_device()), _r.isNone(6));
6141: 
6142:         auto dispatch_randint_out = [](at::Tensor out, c10::SymInt high, c10::SymIntArrayRef size, ::std::optional<at::Generator> generator) -> at::Tensor {
6143:           pybind11::gil_scoped_release no_gil;
6144:           return at::randint_symint_out(out, high, size, generator);
6145:         };
6146:         return wrap(dispatch_randint_out(_r.tensor(3), _r.toSymInt(0), _r.symintlist(1), _r.generator(2)).set_requires_grad(_r.toBool(8)));
6147:       }
6148:     }
6149:     case 1: {
6150:       if (_r.isNone(2)) {
6151:         // aten::randint(SymInt high, SymInt[] size, *, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6152:         const auto options = TensorOptions()
6153:             .dtype(_r.scalartypeWithDefault(3, at::kLong))
6154:             .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
6155:             .layout(_r.layoutOptional(4))
6156:             .requires_grad(_r.toBool(7))
6157:             .pinned_memory(_r.toBool(6));
6158:         torch::utils::maybe_initialize_device(options);
6159: 
6160:         auto dispatch_randint = [](c10::SymInt high, c10::SymIntArrayRef size, at::TensorOptions options) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `rand_like`, `TensorOptions`, `maybe_initialize_device`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `rand_like`, `TensorOptions`, `maybe_initialize_device` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 6161-6240

```cpp
6161:           pybind11::gil_scoped_release no_gil;
6162:           return torch::randint_symint(high, size, options);
6163:         };
6164:         return wrap(dispatch_randint(_r.toSymInt(0), _r.symintlist(1), options));
6165:       } else {
6166:         // aten::randint.out(SymInt high, SymInt[] size, *, Tensor(a!) out) -> Tensor(a!)
6167:         check_out_type_matches(_r.tensor(2), _r.scalartypeWithDefault(3, at::kLong),
6168:                                _r.isNone(3), _r.layoutOptional(4),
6169:                                _r.deviceWithDefault(5, torch::tensors::get_default_device()), _r.isNone(5));
6170: 
6171:         auto dispatch_randint_out = [](at::Tensor out, c10::SymInt high, c10::SymIntArrayRef size) -> at::Tensor {
6172:           pybind11::gil_scoped_release no_gil;
6173:           return at::randint_symint_out(out, high, size);
6174:         };
6175:         return wrap(dispatch_randint_out(_r.tensor(2), _r.toSymInt(0), _r.symintlist(1)).set_requires_grad(_r.toBool(7)));
6176:       }
6177:     }
6178:     case 2: {
6179:       if (_r.isNone(4)) {
6180:         // aten::randint.low_generator(SymInt low, SymInt high, SymInt[] size, *, Generator? generator, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6181:         const auto options = TensorOptions()
6182:             .dtype(_r.scalartypeWithDefault(5, at::kLong))
6183:             .device(_r.deviceWithDefault(7, torch::tensors::get_default_device()))
6184:             .layout(_r.layoutOptional(6))
6185:             .requires_grad(_r.toBool(9))
6186:             .pinned_memory(_r.toBool(8));
6187:         torch::utils::maybe_initialize_device(options);
6188: 
6189:         auto dispatch_randint = [](c10::SymInt low, c10::SymInt high, c10::SymIntArrayRef size, ::std::optional<at::Generator> generator, at::TensorOptions options) -> at::Tensor {
6190:           pybind11::gil_scoped_release no_gil;
6191:           return torch::randint_symint(low, high, size, generator, options);
6192:         };
6193:         return wrap(dispatch_randint(_r.toSymInt(0), _r.toSymInt(1), _r.symintlist(2), _r.generator(3), options));
6194:       } else {
6195:         // aten::randint.low_generator_out(SymInt low, SymInt high, SymInt[] size, *, Generator? generator, Tensor(a!) out) -> Tensor(a!)
6196:         check_out_type_matches(_r.tensor(4), _r.scalartypeWithDefault(5, at::kLong),
6197:                                _r.isNone(5), _r.layoutOptional(6),
6198:                                _r.deviceWithDefault(7, torch::tensors::get_default_device()), _r.isNone(7));
6199: 
6200:         auto dispatch_randint_out = [](at::Tensor out, c10::SymInt low, c10::SymInt high, c10::SymIntArrayRef size, ::std::optional<at::Generator> generator) -> at::Tensor {
6201:           pybind11::gil_scoped_release no_gil;
6202:           return at::randint_symint_out(out, low, high, size, generator);
6203:         };
6204:         return wrap(dispatch_randint_out(_r.tensor(4), _r.toSymInt(0), _r.toSymInt(1), _r.symintlist(2), _r.generator(3)).set_requires_grad(_r.toBool(9)));
6205:       }
6206:     }
6207:     case 3: {
6208:       if (_r.isNone(3)) {
6209:         // aten::randint.low(SymInt low, SymInt high, SymInt[] size, *, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6210:         const auto options = TensorOptions()
6211:             .dtype(_r.scalartypeWithDefault(4, at::kLong))
6212:             .device(_r.deviceWithDefault(6, torch::tensors::get_default_device()))
6213:             .layout(_r.layoutOptional(5))
6214:             .requires_grad(_r.toBool(8))
6215:             .pinned_memory(_r.toBool(7));
6216:         torch::utils::maybe_initialize_device(options);
6217: 
6218:         auto dispatch_randint = [](c10::SymInt low, c10::SymInt high, c10::SymIntArrayRef size, at::TensorOptions options) -> at::Tensor {
6219:           pybind11::gil_scoped_release no_gil;
6220:           return torch::randint_symint(low, high, size, options);
6221:         };
6222:         return wrap(dispatch_randint(_r.toSymInt(0), _r.toSymInt(1), _r.symintlist(2), options));
6223:       } else {
6224:         // aten::randint.low_out(SymInt low, SymInt high, SymInt[] size, *, Tensor(a!) out) -> Tensor(a!)
6225:         check_out_type_matches(_r.tensor(3), _r.scalartypeWithDefault(4, at::kLong),
6226:                                _r.isNone(4), _r.layoutOptional(5),
6227:                                _r.deviceWithDefault(6, torch::tensors::get_default_device()), _r.isNone(6));
6228: 
6229:         auto dispatch_randint_out = [](at::Tensor out, c10::SymInt low, c10::SymInt high, c10::SymIntArrayRef size) -> at::Tensor {
6230:           pybind11::gil_scoped_release no_gil;
6231:           return at::randint_symint_out(out, low, high, size);
6232:         };
6233:         return wrap(dispatch_randint_out(_r.tensor(3), _r.toSymInt(0), _r.toSymInt(1), _r.symintlist(2)).set_requires_grad(_r.toBool(8)));
6234:       }
6235:     }
6236:   }
6237:   Py_RETURN_NONE;
6238:   END_HANDLE_TH_ERRORS
6239: }
6240: 
```

- EN: The main execution path in this span is carried by `randint_symint`, `wrap`, `check_out_type_matches`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `randint_symint`, `wrap`, `check_out_type_matches` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 6241-6320

```cpp
6241: \
6242: // randint_like
6243: static PyObject * THPVariable_randint_like(PyObject* self_, PyObject* args, PyObject* kwargs)
6244: {
6245:   HANDLE_TH_ERRORS
6246:   static PythonArgParser parser({
6247:     "randint_like(Tensor input, SymInt low, SymInt high, *, Generator? generator, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6248:     "randint_like(Tensor input, SymInt low, SymInt high, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6249:     "randint_like(Tensor input, Tensor high, *, Generator? generator, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6250:     "randint_like(Tensor input, Tensor high, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6251:     "randint_like(Tensor input, SymInt high, *, Generator? generator, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6252:     "randint_like(Tensor input, SymInt high, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6253:   }, /*traceable=*/true);
6254: 
6255:   ParsedArgs<10> parsed_args;
6256:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6257:   if(_r.has_torch_function()) {
6258:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6259:   }
6260:   switch (_r.idx) {
6261:     case 0: {
6262:       // aten::randint_like.low_generator_dtype(Tensor self, SymInt low, SymInt high, *, Generator? generator, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6263:       auto self = _r.tensor(0);
6264:       const auto options = TensorOptions()
6265:           .dtype(_r.scalartypeOptional(5))
6266:           .device(_r.deviceOptional(7))
6267:           .layout(_r.layoutOptional(6))
6268:           .requires_grad(_r.toBool(9))
6269:           .pinned_memory(_r.toBool(8));
6270:       torch::utils::maybe_initialize_device(options);
6271: 
6272:       auto dispatch_randint_like = [](const at::Tensor & self, c10::SymInt low, c10::SymInt high, ::std::optional<at::Generator> generator, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6273:         pybind11::gil_scoped_release no_gil;
6274:         return torch::randint_like_symint(self, low, high, generator, options, memory_format);
6275:       };
6276:       return wrap(dispatch_randint_like(self, _r.toSymInt(1), _r.toSymInt(2), _r.generator(3), options, _r.memoryformatOptional(4)));
6277:     }
6278:     case 1: {
6279:       // aten::randint_like.low_dtype(Tensor self, SymInt low, SymInt high, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6280:       auto self = _r.tensor(0);
6281:       const auto options = TensorOptions()
6282:           .dtype(_r.scalartypeOptional(4))
6283:           .device(_r.deviceOptional(6))
6284:           .layout(_r.layoutOptional(5))
6285:           .requires_grad(_r.toBool(8))
6286:           .pinned_memory(_r.toBool(7));
6287:       torch::utils::maybe_initialize_device(options);
6288: 
6289:       auto dispatch_randint_like = [](const at::Tensor & self, c10::SymInt low, c10::SymInt high, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6290:         pybind11::gil_scoped_release no_gil;
6291:         return torch::randint_like_symint(self, low, high, options, memory_format);
6292:       };
6293:       return wrap(dispatch_randint_like(self, _r.toSymInt(1), _r.toSymInt(2), options, _r.memoryformatOptional(3)));
6294:     }
6295:     case 2: {
6296:       // aten::randint_like.Tensor_generator(Tensor self, Tensor high, *, Generator? generator, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6297:       auto self = _r.tensor(0);
6298:       const auto options = TensorOptions()
6299:           .dtype(_r.scalartypeOptional(4))
6300:           .device(_r.deviceOptional(6))
6301:           .layout(_r.layoutOptional(5))
6302:           .requires_grad(_r.toBool(8))
6303:           .pinned_memory(_r.toBool(7));
6304:       torch::utils::maybe_initialize_device(options);
6305: 
6306:       auto dispatch_randint_like = [](const at::Tensor & self, const at::Tensor & high, ::std::optional<at::Generator> generator, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6307:         pybind11::gil_scoped_release no_gil;
6308:         return torch::randint_like(self, high, generator, options, memory_format);
6309:       };
6310:       return wrap(dispatch_randint_like(self, _r.tensor(1), _r.generator(2), options, _r.memoryformatOptional(3)));
6311:     }
6312:     case 3: {
6313:       // aten::randint_like.Tensor(Tensor self, Tensor high, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6314:       auto self = _r.tensor(0);
6315:       const auto options = TensorOptions()
6316:           .dtype(_r.scalartypeOptional(3))
6317:           .device(_r.deviceOptional(5))
6318:           .layout(_r.layoutOptional(4))
6319:           .requires_grad(_r.toBool(7))
6320:           .pinned_memory(_r.toBool(6));
```

- EN: The main execution path in this span is carried by `THPVariable_randint_like`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_randint_like`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6321-6400

```cpp
6321:       torch::utils::maybe_initialize_device(options);
6322: 
6323:       auto dispatch_randint_like = [](const at::Tensor & self, const at::Tensor & high, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6324:         pybind11::gil_scoped_release no_gil;
6325:         return torch::randint_like(self, high, options, memory_format);
6326:       };
6327:       return wrap(dispatch_randint_like(self, _r.tensor(1), options, _r.memoryformatOptional(2)));
6328:     }
6329:     case 4: {
6330:       // aten::randint_like.generator(Tensor self, SymInt high, *, Generator? generator, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6331:       auto self = _r.tensor(0);
6332:       const auto options = TensorOptions()
6333:           .dtype(_r.scalartypeOptional(4))
6334:           .device(_r.deviceOptional(6))
6335:           .layout(_r.layoutOptional(5))
6336:           .requires_grad(_r.toBool(8))
6337:           .pinned_memory(_r.toBool(7));
6338:       torch::utils::maybe_initialize_device(options);
6339: 
6340:       auto dispatch_randint_like = [](const at::Tensor & self, c10::SymInt high, ::std::optional<at::Generator> generator, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6341:         pybind11::gil_scoped_release no_gil;
6342:         return torch::randint_like_symint(self, high, generator, options, memory_format);
6343:       };
6344:       return wrap(dispatch_randint_like(self, _r.toSymInt(1), _r.generator(2), options, _r.memoryformatOptional(3)));
6345:     }
6346:     case 5: {
6347:       // aten::randint_like(Tensor self, SymInt high, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
6348:       auto self = _r.tensor(0);
6349:       const auto options = TensorOptions()
6350:           .dtype(_r.scalartypeOptional(3))
6351:           .device(_r.deviceOptional(5))
6352:           .layout(_r.layoutOptional(4))
6353:           .requires_grad(_r.toBool(7))
6354:           .pinned_memory(_r.toBool(6));
6355:       torch::utils::maybe_initialize_device(options);
6356: 
6357:       auto dispatch_randint_like = [](const at::Tensor & self, c10::SymInt high, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
6358:         pybind11::gil_scoped_release no_gil;
6359:         return torch::randint_like_symint(self, high, options, memory_format);
6360:       };
6361:       return wrap(dispatch_randint_like(self, _r.toSymInt(1), options, _r.memoryformatOptional(2)));
6362:     }
6363:   }
6364:   Py_RETURN_NONE;
6365:   END_HANDLE_TH_ERRORS
6366: }
6367: 
6368: \
6369: // randn
6370: static PyObject * THPVariable_randn(PyObject* self_, PyObject* args, PyObject* kwargs)
6371: {
6372:   HANDLE_TH_ERRORS
6373:   static PythonArgParser parser({
6374:     "randn(SymIntArrayRef size, *, Generator? generator, DimnameList? names, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6375:     "randn(SymIntArrayRef size, *, Generator? generator, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6376:     "randn(SymIntArrayRef size, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6377:     "randn(SymIntArrayRef size, *, DimnameList? names, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6378:   }, /*traceable=*/true);
6379: 
6380:   ParsedArgs<8> parsed_args;
6381:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6382:   if(_r.has_torch_function()) {
6383:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6384:   }
6385:   switch (_r.idx) {
6386:     case 0: {
6387:       // aten::randn.generator_with_names(SymInt[] size, *, Generator? generator, Dimname[]? names, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6388:       auto __names = _r.toDimnameListOptional(2);
6389:       ::std::optional<DimnameList> names = __names ? ::std::make_optional(DimnameList(__names.value())) : ::std::nullopt;
6390:       const auto options = TensorOptions()
6391:           .dtype(_r.scalartypeOptional(3))
6392:           .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
6393:           .layout(_r.layoutOptional(4))
6394:           .requires_grad(_r.toBool(7))
6395:           .pinned_memory(_r.toBool(6));
6396:       torch::utils::maybe_initialize_device(options);
6397: 
6398:       auto dispatch_randn = [](c10::SymIntArrayRef size, ::std::optional<at::Generator> generator, ::std::optional<at::DimnameList> names, at::TensorOptions options) -> at::Tensor {
6399:         pybind11::gil_scoped_release no_gil;
6400:         return torch::randn_symint(size, generator, names, options);
```

- EN: The main execution path in this span is carried by `maybe_initialize_device`, `randint_like`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `maybe_initialize_device`, `randint_like`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 6401-6480

```cpp
6401:       };
6402:       return wrap(dispatch_randn(_r.symintlist(0), _r.generator(1), names, options));
6403:     }
6404:     case 1: {
6405:       if (_r.isNone(2)) {
6406:         // aten::randn.generator(SymInt[] size, *, Generator? generator, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6407:         const auto options = TensorOptions()
6408:             .dtype(_r.scalartypeOptional(3))
6409:             .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
6410:             .layout(_r.layoutOptional(4))
6411:             .requires_grad(_r.toBool(7))
6412:             .pinned_memory(_r.toBool(6));
6413:         torch::utils::maybe_initialize_device(options);
6414: 
6415:         auto dispatch_randn = [](c10::SymIntArrayRef size, ::std::optional<at::Generator> generator, at::TensorOptions options) -> at::Tensor {
6416:           pybind11::gil_scoped_release no_gil;
6417:           return torch::randn_symint(size, generator, options);
6418:         };
6419:         return wrap(dispatch_randn(_r.symintlist(0), _r.generator(1), options));
6420:       } else {
6421:         // aten::randn.generator_out(SymInt[] size, *, Generator? generator, Tensor(a!) out) -> Tensor(a!)
6422:         check_out_type_matches(_r.tensor(2), _r.scalartypeOptional(3),
6423:                                _r.isNone(3), _r.layoutOptional(4),
6424:                                _r.deviceWithDefault(5, torch::tensors::get_default_device()), _r.isNone(5));
6425: 
6426:         auto dispatch_randn_out = [](at::Tensor out, c10::SymIntArrayRef size, ::std::optional<at::Generator> generator) -> at::Tensor {
6427:           pybind11::gil_scoped_release no_gil;
6428:           return at::randn_symint_out(out, size, generator);
6429:         };
6430:         return wrap(dispatch_randn_out(_r.tensor(2), _r.symintlist(0), _r.generator(1)).set_requires_grad(_r.toBool(7)));
6431:       }
6432:     }
6433:     case 2: {
6434:       if (_r.isNone(1)) {
6435:         // aten::randn(SymInt[] size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6436:         const auto options = TensorOptions()
6437:             .dtype(_r.scalartypeOptional(2))
6438:             .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
6439:             .layout(_r.layoutOptional(3))
6440:             .requires_grad(_r.toBool(6))
6441:             .pinned_memory(_r.toBool(5));
6442:         torch::utils::maybe_initialize_device(options);
6443: 
6444:         auto dispatch_randn = [](c10::SymIntArrayRef size, at::TensorOptions options) -> at::Tensor {
6445:           pybind11::gil_scoped_release no_gil;
6446:           return torch::randn_symint(size, options);
6447:         };
6448:         return wrap(dispatch_randn(_r.symintlist(0), options));
6449:       } else {
6450:         // aten::randn.out(SymInt[] size, *, Tensor(a!) out) -> Tensor(a!)
6451:         check_out_type_matches(_r.tensor(1), _r.scalartypeOptional(2),
6452:                                _r.isNone(2), _r.layoutOptional(3),
6453:                                _r.deviceWithDefault(4, torch::tensors::get_default_device()), _r.isNone(4));
6454: 
6455:         auto dispatch_randn_out = [](at::Tensor out, c10::SymIntArrayRef size) -> at::Tensor {
6456:           pybind11::gil_scoped_release no_gil;
6457:           return at::randn_symint_out(out, size);
6458:         };
6459:         return wrap(dispatch_randn_out(_r.tensor(1), _r.symintlist(0)).set_requires_grad(_r.toBool(6)));
6460:       }
6461:     }
6462:     case 3: {
6463:       // aten::randn.names(SymInt[] size, *, Dimname[]? names, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6464:       auto __names = _r.toDimnameListOptional(1);
6465:       ::std::optional<DimnameList> names = __names ? ::std::make_optional(DimnameList(__names.value())) : ::std::nullopt;
6466:       const auto options = TensorOptions()
6467:           .dtype(_r.scalartypeOptional(2))
6468:           .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
6469:           .layout(_r.layoutOptional(3))
6470:           .requires_grad(_r.toBool(6))
6471:           .pinned_memory(_r.toBool(5));
6472:       torch::utils::maybe_initialize_device(options);
6473: 
6474:       auto dispatch_randn = [](c10::SymIntArrayRef size, ::std::optional<at::DimnameList> names, at::TensorOptions options) -> at::Tensor {
6475:         pybind11::gil_scoped_release no_gil;
6476:         return torch::randn_symint(size, names, options);
6477:       };
6478:       return wrap(dispatch_randn(_r.symintlist(0), names, options));
6479:     }
6480:   }
```

- EN: The main execution path in this span is carried by `wrap`, `TensorOptions`, `maybe_initialize_device`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `wrap`, `TensorOptions`, `maybe_initialize_device` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 6481-6560

```cpp
6481:   Py_RETURN_NONE;
6482:   END_HANDLE_TH_ERRORS
6483: }
6484: 
6485: \
6486: // randperm
6487: static PyObject * THPVariable_randperm(PyObject* self_, PyObject* args, PyObject* kwargs)
6488: {
6489:   HANDLE_TH_ERRORS
6490:   static PythonArgParser parser({
6491:     "randperm(SymInt n, *, Generator? generator, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6492:     "randperm(SymInt n, *, Tensor out=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
6493:   }, /*traceable=*/true);
6494: 
6495:   ParsedArgs<8> parsed_args;
6496:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6497:   if(_r.has_torch_function()) {
6498:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6499:   }
6500:   switch (_r.idx) {
6501:     case 0: {
6502:       if (_r.isNone(2)) {
6503:         // aten::randperm.generator(SymInt n, *, Generator? generator, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6504:         const auto options = TensorOptions()
6505:             .dtype(_r.scalartypeWithDefault(3, at::kLong))
6506:             .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
6507:             .layout(_r.layoutOptional(4))
6508:             .requires_grad(_r.toBool(7))
6509:             .pinned_memory(_r.toBool(6));
6510:         torch::utils::maybe_initialize_device(options);
6511: 
6512:         auto dispatch_randperm = [](c10::SymInt n, ::std::optional<at::Generator> generator, at::TensorOptions options) -> at::Tensor {
6513:           pybind11::gil_scoped_release no_gil;
6514:           return torch::randperm_symint(n, generator, options);
6515:         };
6516:         return wrap(dispatch_randperm(_r.toSymInt(0), _r.generator(1), options));
6517:       } else {
6518:         // aten::randperm.generator_out(SymInt n, *, Generator? generator, Tensor(a!) out) -> Tensor(a!)
6519:         check_out_type_matches(_r.tensor(2), _r.scalartypeWithDefault(3, at::kLong),
6520:                                _r.isNone(3), _r.layoutOptional(4),
6521:                                _r.deviceWithDefault(5, torch::tensors::get_default_device()), _r.isNone(5));
6522: 
6523:         auto dispatch_randperm_out = [](at::Tensor out, c10::SymInt n, ::std::optional<at::Generator> generator) -> at::Tensor {
6524:           pybind11::gil_scoped_release no_gil;
6525:           return at::randperm_symint_out(out, n, generator);
6526:         };
6527:         return wrap(dispatch_randperm_out(_r.tensor(2), _r.toSymInt(0), _r.generator(1)).set_requires_grad(_r.toBool(7)));
6528:       }
6529:     }
6530:     case 1: {
6531:       if (_r.isNone(1)) {
6532:         // aten::randperm(SymInt n, *, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
6533:         const auto options = TensorOptions()
6534:             .dtype(_r.scalartypeWithDefault(2, at::kLong))
6535:             .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
6536:             .layout(_r.layoutOptional(3))
6537:             .requires_grad(_r.toBool(6))
6538:             .pinned_memory(_r.toBool(5));
6539:         torch::utils::maybe_initialize_device(options);
6540: 
6541:         auto dispatch_randperm = [](c10::SymInt n, at::TensorOptions options) -> at::Tensor {
6542:           pybind11::gil_scoped_release no_gil;
6543:           return torch::randperm_symint(n, options);
6544:         };
6545:         return wrap(dispatch_randperm(_r.toSymInt(0), options));
6546:       } else {
6547:         // aten::randperm.out(SymInt n, *, Tensor(a!) out) -> Tensor(a!)
6548:         check_out_type_matches(_r.tensor(1), _r.scalartypeWithDefault(2, at::kLong),
6549:                                _r.isNone(2), _r.layoutOptional(3),
6550:                                _r.deviceWithDefault(4, torch::tensors::get_default_device()), _r.isNone(4));
6551: 
6552:         auto dispatch_randperm_out = [](at::Tensor out, c10::SymInt n) -> at::Tensor {
6553:           pybind11::gil_scoped_release no_gil;
6554:           return at::randperm_symint_out(out, n);
6555:         };
6556:         return wrap(dispatch_randperm_out(_r.tensor(1), _r.toSymInt(0)).set_requires_grad(_r.toBool(6)));
6557:       }
6558:     }
6559:   }
6560:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `THPVariable_randperm`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_randperm`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6561-6640

```cpp
6561:   END_HANDLE_TH_ERRORS
6562: }
6563: 
6564: // reciprocal
6565: static PyObject * THPVariable_reciprocal(PyObject* self_, PyObject* args, PyObject* kwargs)
6566: {
6567:   HANDLE_TH_ERRORS
6568:   static PythonArgParser parser({
6569:     "reciprocal(Tensor input, *, Tensor out=None)",
6570:   }, /*traceable=*/true);
6571: 
6572:   ParsedArgs<2> parsed_args;
6573:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6574:   if(_r.has_torch_function()) {
6575:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6576:   }
6577:   if (_r.isNone(1)) {
6578:     // aten::reciprocal(Tensor self) -> Tensor
6579: 
6580:     auto dispatch_reciprocal = [](const at::Tensor & self) -> at::Tensor {
6581:       pybind11::gil_scoped_release no_gil;
6582:       return self.reciprocal();
6583:     };
6584:     return wrap(dispatch_reciprocal(_r.tensor(0)));
6585:   } else {
6586:     // aten::reciprocal.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6587: 
6588:     auto dispatch_reciprocal_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6589:       pybind11::gil_scoped_release no_gil;
6590:       return at::reciprocal_out(out, self);
6591:     };
6592:     return wrap(dispatch_reciprocal_out(_r.tensor(1), _r.tensor(0)));
6593:   }
6594:   Py_RETURN_NONE;
6595:   END_HANDLE_TH_ERRORS
6596: }
6597: 
6598: // reciprocal_
6599: static PyObject * THPVariable_reciprocal_(PyObject* self_, PyObject* args, PyObject* kwargs)
6600: {
6601:   HANDLE_TH_ERRORS
6602:   static PythonArgParser parser({
6603:     "reciprocal_(Tensor input)",
6604:   }, /*traceable=*/true);
6605: 
6606:   ParsedArgs<1> parsed_args;
6607:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6608:   if(_r.has_torch_function()) {
6609:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6610:   }
6611:   // aten::reciprocal_(Tensor(a!) self) -> Tensor(a!)
6612: 
6613:   auto dispatch_reciprocal_ = [](at::Tensor self) -> at::Tensor {
6614:     pybind11::gil_scoped_release no_gil;
6615:     return self.reciprocal_();
6616:   };
6617:   return wrap(dispatch_reciprocal_(_r.tensor(0)));
6618:   Py_RETURN_NONE;
6619:   END_HANDLE_TH_ERRORS
6620: }
6621: 
6622: // negative
6623: static PyObject * THPVariable_negative(PyObject* self_, PyObject* args, PyObject* kwargs)
6624: {
6625:   HANDLE_TH_ERRORS
6626:   static PythonArgParser parser({
6627:     "negative(Tensor input, *, Tensor out=None)",
6628:   }, /*traceable=*/true);
6629: 
6630:   ParsedArgs<2> parsed_args;
6631:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6632:   if(_r.has_torch_function()) {
6633:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6634:   }
6635:   if (_r.isNone(1)) {
6636:     // aten::negative(Tensor self) -> Tensor
6637: 
6638:     auto dispatch_negative = [](const at::Tensor & self) -> at::Tensor {
6639:       pybind11::gil_scoped_release no_gil;
6640:       return self.negative();
```

- EN: The main execution path in this span is carried by `THPVariable_reciprocal`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_reciprocal`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6641-6720

```cpp
6641:     };
6642:     return wrap(dispatch_negative(_r.tensor(0)));
6643:   } else {
6644:     // aten::negative.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6645: 
6646:     auto dispatch_negative_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6647:       pybind11::gil_scoped_release no_gil;
6648:       return at::negative_out(out, self);
6649:     };
6650:     return wrap(dispatch_negative_out(_r.tensor(1), _r.tensor(0)));
6651:   }
6652:   Py_RETURN_NONE;
6653:   END_HANDLE_TH_ERRORS
6654: }
6655: 
6656: // negative_
6657: static PyObject * THPVariable_negative_(PyObject* self_, PyObject* args, PyObject* kwargs)
6658: {
6659:   HANDLE_TH_ERRORS
6660:   static PythonArgParser parser({
6661:     "negative_(Tensor input)",
6662:   }, /*traceable=*/true);
6663: 
6664:   ParsedArgs<1> parsed_args;
6665:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6666:   if(_r.has_torch_function()) {
6667:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6668:   }
6669:   // aten::negative_(Tensor(a!) self) -> Tensor(a!)
6670: 
6671:   auto dispatch_negative_ = [](at::Tensor self) -> at::Tensor {
6672:     pybind11::gil_scoped_release no_gil;
6673:     return self.negative_();
6674:   };
6675:   return wrap(dispatch_negative_(_r.tensor(0)));
6676:   Py_RETURN_NONE;
6677:   END_HANDLE_TH_ERRORS
6678: }
6679: 
6680: // reshape
6681: static PyObject * THPVariable_reshape(PyObject* self_, PyObject* args, PyObject* kwargs)
6682: {
6683:   HANDLE_TH_ERRORS
6684:   static PythonArgParser parser({
6685:     "reshape(Tensor input, SymIntArrayRef shape)",
6686:   }, /*traceable=*/true);
6687: 
6688:   ParsedArgs<2> parsed_args;
6689:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6690:   if(_r.has_torch_function()) {
6691:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6692:   }
6693:   // aten::reshape(Tensor(a) self, SymInt[] shape) -> Tensor(a)
6694: 
6695:   auto dispatch_reshape = [](const at::Tensor & self, c10::SymIntArrayRef shape) -> at::Tensor {
6696:     pybind11::gil_scoped_release no_gil;
6697:     return self.reshape_symint(shape);
6698:   };
6699:   return wrap(dispatch_reshape(_r.tensor(0), _r.symintlist(1)));
6700:   Py_RETURN_NONE;
6701:   END_HANDLE_TH_ERRORS
6702: }
6703: 
6704: // _mkldnn_reshape
6705: static PyObject * THPVariable__mkldnn_reshape(PyObject* self_, PyObject* args, PyObject* kwargs)
6706: {
6707:   HANDLE_TH_ERRORS
6708:   static PythonArgParser parser({
6709:     "_mkldnn_reshape(Tensor input, IntArrayRef shape)",
6710:   }, /*traceable=*/true);
6711: 
6712:   ParsedArgs<2> parsed_args;
6713:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6714:   if(_r.has_torch_function()) {
6715:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6716:   }
6717:   // aten::_mkldnn_reshape(Tensor self, int[] shape) -> Tensor
6718: 
6719:   auto dispatch__mkldnn_reshape = [](const at::Tensor & self, at::IntArrayRef shape) -> at::Tensor {
6720:     pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `wrap`, `negative_out`, `THPVariable_negative_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `negative_out`, `THPVariable_negative_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6721-6800

```cpp
6721:     return at::_mkldnn_reshape(self, shape);
6722:   };
6723:   return wrap(dispatch__mkldnn_reshape(_r.tensor(0), _r.intlist(1)));
6724:   Py_RETURN_NONE;
6725:   END_HANDLE_TH_ERRORS
6726: }
6727: 
6728: // relu
6729: static PyObject * THPVariable_relu(PyObject* self_, PyObject* args, PyObject* kwargs)
6730: {
6731:   HANDLE_TH_ERRORS
6732:   static PythonArgParser parser({
6733:     "relu(Tensor input)",
6734:   }, /*traceable=*/true);
6735: 
6736:   ParsedArgs<1> parsed_args;
6737:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6738:   if(_r.has_torch_function()) {
6739:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6740:   }
6741:   // aten::relu(Tensor self) -> Tensor
6742: 
6743:   auto dispatch_relu = [](const at::Tensor & self) -> at::Tensor {
6744:     pybind11::gil_scoped_release no_gil;
6745:     return self.relu();
6746:   };
6747:   return wrap(dispatch_relu(_r.tensor(0)));
6748:   Py_RETURN_NONE;
6749:   END_HANDLE_TH_ERRORS
6750: }
6751: 
6752: // relu_
6753: static PyObject * THPVariable_relu_(PyObject* self_, PyObject* args, PyObject* kwargs)
6754: {
6755:   HANDLE_TH_ERRORS
6756:   static PythonArgParser parser({
6757:     "relu_(Tensor input)",
6758:   }, /*traceable=*/true);
6759: 
6760:   ParsedArgs<1> parsed_args;
6761:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6762:   if(_r.has_torch_function()) {
6763:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6764:   }
6765:   // aten::relu_(Tensor(a!) self) -> Tensor(a!)
6766: 
6767:   auto dispatch_relu_ = [](at::Tensor self) -> at::Tensor {
6768:     pybind11::gil_scoped_release no_gil;
6769:     return self.relu_();
6770:   };
6771:   return wrap(dispatch_relu_(_r.tensor(0)));
6772:   Py_RETURN_NONE;
6773:   END_HANDLE_TH_ERRORS
6774: }
6775: 
6776: // prelu
6777: static PyObject * THPVariable_prelu(PyObject* self_, PyObject* args, PyObject* kwargs)
6778: {
6779:   HANDLE_TH_ERRORS
6780:   static PythonArgParser parser({
6781:     "prelu(Tensor input, Tensor weight)",
6782:   }, /*traceable=*/true);
6783: 
6784:   ParsedArgs<2> parsed_args;
6785:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6786:   if(_r.has_torch_function()) {
6787:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6788:   }
6789:   // aten::prelu(Tensor self, Tensor weight) -> Tensor
6790: 
6791:   auto dispatch_prelu = [](const at::Tensor & self, const at::Tensor & weight) -> at::Tensor {
6792:     pybind11::gil_scoped_release no_gil;
6793:     return self.prelu(weight);
6794:   };
6795:   return wrap(dispatch_prelu(_r.tensor(0), _r.tensor(1)));
6796:   Py_RETURN_NONE;
6797:   END_HANDLE_TH_ERRORS
6798: }
6799: 
6800: \
```

- EN: The main execution path in this span is carried by `_mkldnn_reshape`, `wrap`, `THPVariable_relu`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_mkldnn_reshape`, `wrap`, `THPVariable_relu` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6801-6880

```cpp
6801: // select
6802: static PyObject * THPVariable_select(PyObject* self_, PyObject* args, PyObject* kwargs)
6803: {
6804:   HANDLE_TH_ERRORS
6805:   static PythonArgParser parser({
6806:     "select(Tensor input, Dimname dim, int64_t index)",
6807:     "select(Tensor input, int64_t dim, SymInt index)",
6808:   }, /*traceable=*/true);
6809: 
6810:   ParsedArgs<3> parsed_args;
6811:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6812:   if(_r.has_torch_function()) {
6813:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6814:   }
6815:   switch (_r.idx) {
6816:     case 0: {
6817:       // aten::select.Dimname(Tensor(a) self, Dimname dim, int index) -> Tensor(a)
6818: 
6819:       auto dispatch_select = [](const at::Tensor & self, at::Dimname dim, int64_t index) -> at::Tensor {
6820:         pybind11::gil_scoped_release no_gil;
6821:         return self.select(dim, index);
6822:       };
6823:       return wrap(dispatch_select(_r.tensor(0), _r.dimname(1), _r.toInt64(2)));
6824:     }
6825:     case 1: {
6826:       // aten::select.int(Tensor(a) self, int dim, SymInt index) -> Tensor(a)
6827: 
6828:       auto dispatch_select = [](const at::Tensor & self, int64_t dim, c10::SymInt index) -> at::Tensor {
6829:         pybind11::gil_scoped_release no_gil;
6830:         return self.select_symint(dim, index);
6831:       };
6832:       return wrap(dispatch_select(_r.tensor(0), _r.toInt64(1), _r.toSymInt(2)));
6833:     }
6834:   }
6835:   Py_RETURN_NONE;
6836:   END_HANDLE_TH_ERRORS
6837: }
6838: 
6839: // selu
6840: static PyObject * THPVariable_selu(PyObject* self_, PyObject* args, PyObject* kwargs)
6841: {
6842:   HANDLE_TH_ERRORS
6843:   static PythonArgParser parser({
6844:     "selu(Tensor input)",
6845:   }, /*traceable=*/true);
6846: 
6847:   ParsedArgs<1> parsed_args;
6848:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6849:   if(_r.has_torch_function()) {
6850:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6851:   }
6852:   // aten::selu(Tensor self) -> Tensor
6853: 
6854:   auto dispatch_selu = [](const at::Tensor & self) -> at::Tensor {
6855:     pybind11::gil_scoped_release no_gil;
6856:     return at::selu(self);
6857:   };
6858:   return wrap(dispatch_selu(_r.tensor(0)));
6859:   Py_RETURN_NONE;
6860:   END_HANDLE_TH_ERRORS
6861: }
6862: 
6863: // selu_
6864: static PyObject * THPVariable_selu_(PyObject* self_, PyObject* args, PyObject* kwargs)
6865: {
6866:   HANDLE_TH_ERRORS
6867:   static PythonArgParser parser({
6868:     "selu_(Tensor input)",
6869:   }, /*traceable=*/true);
6870: 
6871:   ParsedArgs<1> parsed_args;
6872:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6873:   if(_r.has_torch_function()) {
6874:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6875:   }
6876:   // aten::selu_(Tensor(a!) self) -> Tensor(a!)
6877: 
6878:   auto dispatch_selu_ = [](at::Tensor self) -> at::Tensor {
6879:     pybind11::gil_scoped_release no_gil;
6880:     return at::selu_(self);
```

- EN: The main execution path in this span is carried by `THPVariable_select`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_select`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6881-6960

```cpp
6881:   };
6882:   return wrap(dispatch_selu_(_r.tensor(0)));
6883:   Py_RETURN_NONE;
6884:   END_HANDLE_TH_ERRORS
6885: }
6886: 
6887: // logit
6888: static PyObject * THPVariable_logit(PyObject* self_, PyObject* args, PyObject* kwargs)
6889: {
6890:   HANDLE_TH_ERRORS
6891:   static PythonArgParser parser({
6892:     "logit(Tensor input, double? eps=None, *, Tensor out=None)",
6893:   }, /*traceable=*/true);
6894: 
6895:   ParsedArgs<3> parsed_args;
6896:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6897:   if(_r.has_torch_function()) {
6898:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6899:   }
6900:   if (_r.isNone(2)) {
6901:     // aten::logit(Tensor self, float? eps=None) -> Tensor
6902: 
6903:     auto dispatch_logit = [](const at::Tensor & self, ::std::optional<double> eps) -> at::Tensor {
6904:       pybind11::gil_scoped_release no_gil;
6905:       return self.logit(eps);
6906:     };
6907:     return wrap(dispatch_logit(_r.tensor(0), _r.toDoubleOptional(1)));
6908:   } else {
6909:     // aten::logit.out(Tensor self, float? eps=None, *, Tensor(a!) out) -> Tensor(a!)
6910: 
6911:     auto dispatch_logit_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<double> eps) -> at::Tensor {
6912:       pybind11::gil_scoped_release no_gil;
6913:       return at::logit_out(out, self, eps);
6914:     };
6915:     return wrap(dispatch_logit_out(_r.tensor(2), _r.tensor(0), _r.toDoubleOptional(1)));
6916:   }
6917:   Py_RETURN_NONE;
6918:   END_HANDLE_TH_ERRORS
6919: }
6920: 
6921: // logit_
6922: static PyObject * THPVariable_logit_(PyObject* self_, PyObject* args, PyObject* kwargs)
6923: {
6924:   HANDLE_TH_ERRORS
6925:   static PythonArgParser parser({
6926:     "logit_(Tensor input, double? eps=None)",
6927:   }, /*traceable=*/true);
6928: 
6929:   ParsedArgs<2> parsed_args;
6930:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6931:   if(_r.has_torch_function()) {
6932:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6933:   }
6934:   // aten::logit_(Tensor(a!) self, float? eps=None) -> Tensor(a!)
6935: 
6936:   auto dispatch_logit_ = [](at::Tensor self, ::std::optional<double> eps) -> at::Tensor {
6937:     pybind11::gil_scoped_release no_gil;
6938:     return self.logit_(eps);
6939:   };
6940:   return wrap(dispatch_logit_(_r.tensor(0), _r.toDoubleOptional(1)));
6941:   Py_RETURN_NONE;
6942:   END_HANDLE_TH_ERRORS
6943: }
6944: 
6945: // sin
6946: static PyObject * THPVariable_sin(PyObject* self_, PyObject* args, PyObject* kwargs)
6947: {
6948:   HANDLE_TH_ERRORS
6949:   static PythonArgParser parser({
6950:     "sin(Tensor input, *, Tensor out=None)",
6951:   }, /*traceable=*/true);
6952: 
6953:   ParsedArgs<2> parsed_args;
6954:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6955:   if(_r.has_torch_function()) {
6956:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6957:   }
6958:   if (_r.isNone(1)) {
6959:     // aten::sin(Tensor self) -> Tensor
6960: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_logit`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_logit`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6961-7040

```cpp
6961:     auto dispatch_sin = [](const at::Tensor & self) -> at::Tensor {
6962:       pybind11::gil_scoped_release no_gil;
6963:       return self.sin();
6964:     };
6965:     return wrap(dispatch_sin(_r.tensor(0)));
6966:   } else {
6967:     // aten::sin.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6968: 
6969:     auto dispatch_sin_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6970:       pybind11::gil_scoped_release no_gil;
6971:       return at::sin_out(out, self);
6972:     };
6973:     return wrap(dispatch_sin_out(_r.tensor(1), _r.tensor(0)));
6974:   }
6975:   Py_RETURN_NONE;
6976:   END_HANDLE_TH_ERRORS
6977: }
6978: 
6979: // sin_
6980: static PyObject * THPVariable_sin_(PyObject* self_, PyObject* args, PyObject* kwargs)
6981: {
6982:   HANDLE_TH_ERRORS
6983:   static PythonArgParser parser({
6984:     "sin_(Tensor input)",
6985:   }, /*traceable=*/true);
6986: 
6987:   ParsedArgs<1> parsed_args;
6988:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6989:   if(_r.has_torch_function()) {
6990:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6991:   }
6992:   // aten::sin_(Tensor(a!) self) -> Tensor(a!)
6993: 
6994:   auto dispatch_sin_ = [](at::Tensor self) -> at::Tensor {
6995:     pybind11::gil_scoped_release no_gil;
6996:     return self.sin_();
6997:   };
6998:   return wrap(dispatch_sin_(_r.tensor(0)));
6999:   Py_RETURN_NONE;
7000:   END_HANDLE_TH_ERRORS
7001: }
7002: 
7003: // slice_inverse
7004: static PyObject * THPVariable_slice_inverse(PyObject* self_, PyObject* args, PyObject* kwargs)
7005: {
7006:   HANDLE_TH_ERRORS
7007:   static PythonArgParser parser({
7008:     "slice_inverse(Tensor input, Tensor src, int64_t dim=0, SymInt? start=None, SymInt? end=None, SymInt step=1)",
7009:   }, /*traceable=*/true);
7010: 
7011:   ParsedArgs<6> parsed_args;
7012:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7013:   if(_r.has_torch_function()) {
7014:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7015:   }
7016:   // aten::slice_inverse(Tensor(a) self, Tensor src, int dim=0, SymInt? start=None, SymInt? end=None, SymInt step=1) -> Tensor(a)
7017: 
7018:   auto dispatch_slice_inverse = [](const at::Tensor & self, const at::Tensor & src, int64_t dim, ::std::optional<c10::SymInt> start, ::std::optional<c10::SymInt> end, c10::SymInt step) -> at::Tensor {
7019:     pybind11::gil_scoped_release no_gil;
7020:     return self.slice_inverse_symint(src, dim, start, end, step);
7021:   };
7022:   return wrap(dispatch_slice_inverse(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toSymIntOptional(3), _r.toSymIntOptional(4), _r.toSymInt(5)));
7023:   Py_RETURN_NONE;
7024:   END_HANDLE_TH_ERRORS
7025: }
7026: 
7027: \
7028: // softmax
7029: static PyObject * THPVariable_softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
7030: {
7031:   HANDLE_TH_ERRORS
7032:   static PythonArgParser parser({
7033:     "softmax(Tensor input, int64_t dim, ScalarType? dtype=None, *, Tensor out=None)",
7034:     "softmax(Tensor input, Dimname dim, *, ScalarType? dtype=None)",
7035:   }, /*traceable=*/true);
7036: 
7037:   ParsedArgs<4> parsed_args;
7038:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7039:   if(_r.has_torch_function()) {
7040:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `wrap`, `sin_out`, `THPVariable_sin_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `sin_out`, `THPVariable_sin_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7041-7120

```cpp
7041:   }
7042:   switch (_r.idx) {
7043:     case 0: {
7044:       if (_r.isNone(3)) {
7045:         // aten::softmax.int(Tensor self, int dim, ScalarType? dtype=None) -> Tensor
7046: 
7047:         auto dispatch_softmax = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7048:           pybind11::gil_scoped_release no_gil;
7049:           return self.softmax(dim, dtype);
7050:         };
7051:         return wrap(dispatch_softmax(_r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
7052:       } else {
7053:         // aten::softmax.int_out(Tensor self, int dim, ScalarType? dtype=None, *, Tensor(a!) out) -> Tensor(a!)
7054: 
7055:         auto dispatch_softmax_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7056:           pybind11::gil_scoped_release no_gil;
7057:           return at::softmax_out(out, self, dim, dtype);
7058:         };
7059:         return wrap(dispatch_softmax_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
7060:       }
7061:     }
7062:     case 1: {
7063:       // aten::softmax.Dimname(Tensor self, Dimname dim, *, ScalarType? dtype=None) -> Tensor
7064: 
7065:       auto dispatch_softmax = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7066:         pybind11::gil_scoped_release no_gil;
7067:         return self.softmax(dim, dtype);
7068:       };
7069:       return wrap(dispatch_softmax(_r.tensor(0), _r.dimname(1), _r.scalartypeOptional(2)));
7070:     }
7071:   }
7072:   Py_RETURN_NONE;
7073:   END_HANDLE_TH_ERRORS
7074: }
7075: 
7076: // unsafe_split_with_sizes
7077: static PyObject * THPVariable_unsafe_split_with_sizes(PyObject* self_, PyObject* args, PyObject* kwargs)
7078: {
7079:   HANDLE_TH_ERRORS
7080:   static PythonArgParser parser({
7081:     "unsafe_split_with_sizes(Tensor input, SymIntArrayRef split_sizes, int64_t dim=0)",
7082:   }, /*traceable=*/true);
7083: 
7084:   ParsedArgs<3> parsed_args;
7085:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7086:   if(_r.has_torch_function()) {
7087:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7088:   }
7089:   // aten::unsafe_split_with_sizes(Tensor self, SymInt[] split_sizes, int dim=0) -> Tensor[]
7090: 
7091:   auto dispatch_unsafe_split_with_sizes = [](const at::Tensor & self, c10::SymIntArrayRef split_sizes, int64_t dim) -> ::std::vector<at::Tensor> {
7092:     pybind11::gil_scoped_release no_gil;
7093:     return self.unsafe_split_with_sizes_symint(split_sizes, dim);
7094:   };
7095:   return wrap(dispatch_unsafe_split_with_sizes(_r.tensor(0), _r.symintlist(1), _r.toInt64(2)));
7096:   Py_RETURN_NONE;
7097:   END_HANDLE_TH_ERRORS
7098: }
7099: 
7100: // split_with_sizes
7101: static PyObject * THPVariable_split_with_sizes(PyObject* self_, PyObject* args, PyObject* kwargs)
7102: {
7103:   HANDLE_TH_ERRORS
7104:   static PythonArgParser parser({
7105:     "split_with_sizes(Tensor input, SymIntArrayRef split_sizes, int64_t dim=0)",
7106:   }, /*traceable=*/true);
7107: 
7108:   ParsedArgs<3> parsed_args;
7109:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7110:   if(_r.has_torch_function()) {
7111:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7112:   }
7113:   // aten::split_with_sizes(Tensor(a -> *) self, SymInt[] split_sizes, int dim=0) -> Tensor(a)[]
7114: 
7115:   auto dispatch_split_with_sizes = [](const at::Tensor & self, c10::SymIntArrayRef split_sizes, int64_t dim) -> ::std::vector<at::Tensor> {
7116:     pybind11::gil_scoped_release no_gil;
7117:     return self.split_with_sizes_symint(split_sizes, dim);
7118:   };
7119:   return wrap(dispatch_split_with_sizes(_r.tensor(0), _r.symintlist(1), _r.toInt64(2)));
7120:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `wrap`, `softmax_out`, `THPVariable_unsafe_split_with_sizes`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `softmax_out`, `THPVariable_unsafe_split_with_sizes` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7121-7200

```cpp
7121:   END_HANDLE_TH_ERRORS
7122: }
7123: 
7124: \
7125: // vsplit
7126: static PyObject * THPVariable_vsplit(PyObject* self_, PyObject* args, PyObject* kwargs)
7127: {
7128:   HANDLE_TH_ERRORS
7129:   static PythonArgParser parser({
7130:     "vsplit(Tensor input, int64_t sections)",
7131:     "vsplit(Tensor input, IntArrayRef indices)",
7132:   }, /*traceable=*/true);
7133: 
7134:   ParsedArgs<2> parsed_args;
7135:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7136:   if(_r.has_torch_function()) {
7137:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7138:   }
7139:   switch (_r.idx) {
7140:     case 0: {
7141:       // aten::vsplit.int(Tensor(a -> *) self, int sections) -> Tensor(a)[]
7142: 
7143:       auto dispatch_vsplit = [](const at::Tensor & self, int64_t sections) -> ::std::vector<at::Tensor> {
7144:         pybind11::gil_scoped_release no_gil;
7145:         return self.vsplit(sections);
7146:       };
7147:       return wrap(dispatch_vsplit(_r.tensor(0), _r.toInt64(1)));
7148:     }
7149:     case 1: {
7150:       // aten::vsplit.array(Tensor(a -> *) self, int[] indices) -> Tensor(a)[]
7151: 
7152:       auto dispatch_vsplit = [](const at::Tensor & self, at::IntArrayRef indices) -> ::std::vector<at::Tensor> {
7153:         pybind11::gil_scoped_release no_gil;
7154:         return self.vsplit(indices);
7155:       };
7156:       return wrap(dispatch_vsplit(_r.tensor(0), _r.intlist(1)));
7157:     }
7158:   }
7159:   Py_RETURN_NONE;
7160:   END_HANDLE_TH_ERRORS
7161: }
7162: 
7163: // _chunk_cat
7164: static PyObject * THPVariable__chunk_cat(PyObject* self_, PyObject* args, PyObject* kwargs)
7165: {
7166:   HANDLE_TH_ERRORS
7167:   static PythonArgParser parser({
7168:     "_chunk_cat(TensorList tensors, int64_t dim, int64_t num_chunks, *, Tensor out=None)",
7169:   }, /*traceable=*/true);
7170: 
7171:   ParsedArgs<4> parsed_args;
7172:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7173:   if(_r.has_torch_function()) {
7174:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7175:   }
7176:   if (_r.isNone(3)) {
7177:     // aten::_chunk_cat(Tensor[] tensors, int dim, int num_chunks) -> Tensor
7178: 
7179:     auto dispatch__chunk_cat = [](at::TensorList tensors, int64_t dim, int64_t num_chunks) -> at::Tensor {
7180:       pybind11::gil_scoped_release no_gil;
7181:       return at::_chunk_cat(tensors, dim, num_chunks);
7182:     };
7183:     return wrap(dispatch__chunk_cat(_r.tensorlist(0), _r.toInt64(1), _r.toInt64(2)));
7184:   } else {
7185:     // aten::_chunk_cat.out(Tensor[] tensors, int dim, int num_chunks, *, Tensor(a!) out) -> Tensor(a!)
7186: 
7187:     auto dispatch__chunk_cat_out = [](at::Tensor out, at::TensorList tensors, int64_t dim, int64_t num_chunks) -> at::Tensor {
7188:       pybind11::gil_scoped_release no_gil;
7189:       return at::_chunk_cat_out(out, tensors, dim, num_chunks);
7190:     };
7191:     return wrap(dispatch__chunk_cat_out(_r.tensor(3), _r.tensorlist(0), _r.toInt64(1), _r.toInt64(2)));
7192:   }
7193:   Py_RETURN_NONE;
7194:   END_HANDLE_TH_ERRORS
7195: }
7196: 
7197: // hstack
7198: static PyObject * THPVariable_hstack(PyObject* self_, PyObject* args, PyObject* kwargs)
7199: {
7200:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_vsplit`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_vsplit`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7201-7280

```cpp
7201:   static PythonArgParser parser({
7202:     "hstack(TensorList tensors, *, Tensor out=None)",
7203:   }, /*traceable=*/true);
7204: 
7205:   ParsedArgs<2> parsed_args;
7206:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7207:   if(_r.has_torch_function()) {
7208:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7209:   }
7210:   if (_r.isNone(1)) {
7211:     // aten::hstack(Tensor[] tensors) -> Tensor
7212: 
7213:     auto dispatch_hstack = [](at::TensorList tensors) -> at::Tensor {
7214:       pybind11::gil_scoped_release no_gil;
7215:       return at::hstack(tensors);
7216:     };
7217:     return wrap(dispatch_hstack(_r.tensorlist(0)));
7218:   } else {
7219:     // aten::hstack.out(Tensor[] tensors, *, Tensor(a!) out) -> Tensor(a!)
7220: 
7221:     auto dispatch_hstack_out = [](at::Tensor out, at::TensorList tensors) -> at::Tensor {
7222:       pybind11::gil_scoped_release no_gil;
7223:       return at::hstack_out(out, tensors);
7224:     };
7225:     return wrap(dispatch_hstack_out(_r.tensor(1), _r.tensorlist(0)));
7226:   }
7227:   Py_RETURN_NONE;
7228:   END_HANDLE_TH_ERRORS
7229: }
7230: 
7231: // vstack
7232: static PyObject * THPVariable_vstack(PyObject* self_, PyObject* args, PyObject* kwargs)
7233: {
7234:   HANDLE_TH_ERRORS
7235:   static PythonArgParser parser({
7236:     "vstack(TensorList tensors, *, Tensor out=None)",
7237:   }, /*traceable=*/true);
7238: 
7239:   ParsedArgs<2> parsed_args;
7240:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7241:   if(_r.has_torch_function()) {
7242:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7243:   }
7244:   if (_r.isNone(1)) {
7245:     // aten::vstack(Tensor[] tensors) -> Tensor
7246: 
7247:     auto dispatch_vstack = [](at::TensorList tensors) -> at::Tensor {
7248:       pybind11::gil_scoped_release no_gil;
7249:       return at::vstack(tensors);
7250:     };
7251:     return wrap(dispatch_vstack(_r.tensorlist(0)));
7252:   } else {
7253:     // aten::vstack.out(Tensor[] tensors, *, Tensor(a!) out) -> Tensor(a!)
7254: 
7255:     auto dispatch_vstack_out = [](at::Tensor out, at::TensorList tensors) -> at::Tensor {
7256:       pybind11::gil_scoped_release no_gil;
7257:       return at::vstack_out(out, tensors);
7258:     };
7259:     return wrap(dispatch_vstack_out(_r.tensor(1), _r.tensorlist(0)));
7260:   }
7261:   Py_RETURN_NONE;
7262:   END_HANDLE_TH_ERRORS
7263: }
7264: 
7265: // dstack
7266: static PyObject * THPVariable_dstack(PyObject* self_, PyObject* args, PyObject* kwargs)
7267: {
7268:   HANDLE_TH_ERRORS
7269:   static PythonArgParser parser({
7270:     "dstack(TensorList tensors, *, Tensor out=None)",
7271:   }, /*traceable=*/true);
7272: 
7273:   ParsedArgs<2> parsed_args;
7274:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7275:   if(_r.has_torch_function()) {
7276:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7277:   }
7278:   if (_r.isNone(1)) {
7279:     // aten::dstack(Tensor[] tensors) -> Tensor
7280: 
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `hstack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `hstack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7281-7360

```cpp
7281:     auto dispatch_dstack = [](at::TensorList tensors) -> at::Tensor {
7282:       pybind11::gil_scoped_release no_gil;
7283:       return at::dstack(tensors);
7284:     };
7285:     return wrap(dispatch_dstack(_r.tensorlist(0)));
7286:   } else {
7287:     // aten::dstack.out(Tensor[] tensors, *, Tensor(a!) out) -> Tensor(a!)
7288: 
7289:     auto dispatch_dstack_out = [](at::Tensor out, at::TensorList tensors) -> at::Tensor {
7290:       pybind11::gil_scoped_release no_gil;
7291:       return at::dstack_out(out, tensors);
7292:     };
7293:     return wrap(dispatch_dstack_out(_r.tensor(1), _r.tensorlist(0)));
7294:   }
7295:   Py_RETURN_NONE;
7296:   END_HANDLE_TH_ERRORS
7297: }
7298: 
7299: \
7300: // sum
7301: static PyObject * THPVariable_sum(PyObject* self_, PyObject* args, PyObject* kwargs)
7302: {
7303:   HANDLE_TH_ERRORS
7304:   static PythonArgParser parser({
7305:     "sum(Tensor input, *, ScalarType? dtype=None)",
7306:     "sum(Tensor input, IntArrayRef[1]? dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor out=None)",
7307:     "sum(Tensor input, DimnameList[1] dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor out=None)",
7308:   }, /*traceable=*/true);
7309: 
7310:   ParsedArgs<5> parsed_args;
7311:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7312:   if(_r.has_torch_function()) {
7313:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7314:   }
7315:   switch (_r.idx) {
7316:     case 0: {
7317:       // aten::sum(Tensor self, *, ScalarType? dtype=None) -> Tensor
7318: 
7319:       auto dispatch_sum = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7320:         pybind11::gil_scoped_release no_gil;
7321:         return self.sum(dtype);
7322:       };
7323:       return wrap(dispatch_sum(_r.tensor(0), _r.scalartypeOptional(1)));
7324:     }
7325:     case 1: {
7326:       if (_r.isNone(4)) {
7327:         // aten::sum.dim_IntList(Tensor self, int[1]? dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
7328: 
7329:         auto dispatch_sum = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7330:           pybind11::gil_scoped_release no_gil;
7331:           return self.sum(dim, keepdim, dtype);
7332:         };
7333:         return wrap(dispatch_sum(_r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
7334:       } else {
7335:         // aten::sum.IntList_out(Tensor self, int[1]? dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
7336: 
7337:         auto dispatch_sum_out = [](at::Tensor out, const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7338:           pybind11::gil_scoped_release no_gil;
7339:           return at::sum_out(out, self, dim, keepdim, dtype);
7340:         };
7341:         return wrap(dispatch_sum_out(_r.tensor(4), _r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
7342:       }
7343:     }
7344:     case 2: {
7345:       if (_r.isNone(4)) {
7346:         // aten::sum.dim_DimnameList(Tensor self, Dimname[1] dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
7347: 
7348:         auto dispatch_sum = [](const at::Tensor & self, at::DimnameList dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7349:           pybind11::gil_scoped_release no_gil;
7350:           return self.sum(dim, keepdim, dtype);
7351:         };
7352:         return wrap(dispatch_sum(_r.tensor(0), _r.dimnamelist(1), _r.toBool(2), _r.scalartypeOptional(3)));
7353:       } else {
7354:         // aten::sum.DimnameList_out(Tensor self, Dimname[1] dim, bool keepdim=False, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
7355: 
7356:         auto dispatch_sum_out = [](at::Tensor out, const at::Tensor & self, at::DimnameList dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7357:           pybind11::gil_scoped_release no_gil;
7358:           return at::sum_out(out, self, dim, keepdim, dtype);
7359:         };
7360:         return wrap(dispatch_sum_out(_r.tensor(4), _r.tensor(0), _r.dimnamelist(1), _r.toBool(2), _r.scalartypeOptional(3)));
```

- EN: The main execution path in this span is carried by `dstack`, `wrap`, `dstack_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `dstack`, `wrap`, `dstack_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7361-7440

```cpp
7361:       }
7362:     }
7363:   }
7364:   Py_RETURN_NONE;
7365:   END_HANDLE_TH_ERRORS
7366: }
7367: 
7368: // tan
7369: static PyObject * THPVariable_tan(PyObject* self_, PyObject* args, PyObject* kwargs)
7370: {
7371:   HANDLE_TH_ERRORS
7372:   static PythonArgParser parser({
7373:     "tan(Tensor input, *, Tensor out=None)",
7374:   }, /*traceable=*/true);
7375: 
7376:   ParsedArgs<2> parsed_args;
7377:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7378:   if(_r.has_torch_function()) {
7379:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7380:   }
7381:   if (_r.isNone(1)) {
7382:     // aten::tan(Tensor self) -> Tensor
7383: 
7384:     auto dispatch_tan = [](const at::Tensor & self) -> at::Tensor {
7385:       pybind11::gil_scoped_release no_gil;
7386:       return self.tan();
7387:     };
7388:     return wrap(dispatch_tan(_r.tensor(0)));
7389:   } else {
7390:     // aten::tan.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
7391: 
7392:     auto dispatch_tan_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
7393:       pybind11::gil_scoped_release no_gil;
7394:       return at::tan_out(out, self);
7395:     };
7396:     return wrap(dispatch_tan_out(_r.tensor(1), _r.tensor(0)));
7397:   }
7398:   Py_RETURN_NONE;
7399:   END_HANDLE_TH_ERRORS
7400: }
7401: 
7402: // tan_
7403: static PyObject * THPVariable_tan_(PyObject* self_, PyObject* args, PyObject* kwargs)
7404: {
7405:   HANDLE_TH_ERRORS
7406:   static PythonArgParser parser({
7407:     "tan_(Tensor input)",
7408:   }, /*traceable=*/true);
7409: 
7410:   ParsedArgs<1> parsed_args;
7411:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7412:   if(_r.has_torch_function()) {
7413:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7414:   }
7415:   // aten::tan_(Tensor(a!) self) -> Tensor(a!)
7416: 
7417:   auto dispatch_tan_ = [](at::Tensor self) -> at::Tensor {
7418:     pybind11::gil_scoped_release no_gil;
7419:     return self.tan_();
7420:   };
7421:   return wrap(dispatch_tan_(_r.tensor(0)));
7422:   Py_RETURN_NONE;
7423:   END_HANDLE_TH_ERRORS
7424: }
7425: 
7426: \
7427: // trapz
7428: static PyObject * THPVariable_trapz(PyObject* self_, PyObject* args, PyObject* kwargs)
7429: {
7430:   HANDLE_TH_ERRORS
7431:   static PythonArgParser parser({
7432:     "trapz(Tensor y, *, double dx=1, int64_t dim=-1)",
7433:     "trapz(Tensor y, Tensor x, *, int64_t dim=-1)",
7434:   }, /*traceable=*/true);
7435: 
7436:   ParsedArgs<3> parsed_args;
7437:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7438:   if(_r.has_torch_function()) {
7439:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7440:   }
```

- EN: The main execution path in this span is carried by `THPVariable_tan`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_tan`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7441-7520

```cpp
7441:   switch (_r.idx) {
7442:     case 0: {
7443:       // aten::trapz.dx(Tensor y, *, float dx=1, int dim=-1) -> Tensor
7444: 
7445:       auto dispatch_trapz = [](const at::Tensor & y, double dx, int64_t dim) -> at::Tensor {
7446:         pybind11::gil_scoped_release no_gil;
7447:         return at::trapz(y, dx, dim);
7448:       };
7449:       return wrap(dispatch_trapz(_r.tensor(0), _r.toDouble(1), _r.toInt64(2)));
7450:     }
7451:     case 1: {
7452:       // aten::trapz.x(Tensor y, Tensor x, *, int dim=-1) -> Tensor
7453: 
7454:       auto dispatch_trapz = [](const at::Tensor & y, const at::Tensor & x, int64_t dim) -> at::Tensor {
7455:         pybind11::gil_scoped_release no_gil;
7456:         return at::trapz(y, x, dim);
7457:       };
7458:       return wrap(dispatch_trapz(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
7459:     }
7460:   }
7461:   Py_RETURN_NONE;
7462:   END_HANDLE_TH_ERRORS
7463: }
7464: 
7465: // _nested_tensor_from_mask_left_aligned
7466: static PyObject * THPVariable__nested_tensor_from_mask_left_aligned(PyObject* self_, PyObject* args, PyObject* kwargs)
7467: {
7468:   HANDLE_TH_ERRORS
7469:   static PythonArgParser parser({
7470:     "_nested_tensor_from_mask_left_aligned(Tensor t, Tensor mask)",
7471:   }, /*traceable=*/false);
7472: 
7473:   ParsedArgs<2> parsed_args;
7474:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7475:   if(_r.has_torch_function()) {
7476:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7477:   }
7478:   // aten::_nested_tensor_from_mask_left_aligned(Tensor t, Tensor mask) -> bool
7479: 
7480:   auto dispatch__nested_tensor_from_mask_left_aligned = [](const at::Tensor & t, const at::Tensor & mask) -> bool {
7481:     pybind11::gil_scoped_release no_gil;
7482:     return at::_nested_tensor_from_mask_left_aligned(t, mask);
7483:   };
7484:   return wrap(dispatch__nested_tensor_from_mask_left_aligned(_r.tensor(0), _r.tensor(1)));
7485:   Py_RETURN_NONE;
7486:   END_HANDLE_TH_ERRORS
7487: }
7488: 
7489: // _nested_from_padded_and_nested_example
7490: static PyObject * THPVariable__nested_from_padded_and_nested_example(PyObject* self_, PyObject* args, PyObject* kwargs)
7491: {
7492:   HANDLE_TH_ERRORS
7493:   static PythonArgParser parser({
7494:     "_nested_from_padded_and_nested_example(Tensor padded, Tensor nt_example)",
7495:   }, /*traceable=*/true);
7496: 
7497:   ParsedArgs<2> parsed_args;
7498:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7499:   if(_r.has_torch_function()) {
7500:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7501:   }
7502:   // aten::_nested_from_padded_and_nested_example(Tensor padded, Tensor nt_example) -> Tensor
7503: 
7504:   auto dispatch__nested_from_padded_and_nested_example = [](const at::Tensor & padded, const at::Tensor & nt_example) -> at::Tensor {
7505:     pybind11::gil_scoped_release no_gil;
7506:     return at::_nested_from_padded_and_nested_example(padded, nt_example);
7507:   };
7508:   return wrap(dispatch__nested_from_padded_and_nested_example(_r.tensor(0), _r.tensor(1)));
7509:   Py_RETURN_NONE;
7510:   END_HANDLE_TH_ERRORS
7511: }
7512: 
7513: // _nested_view_from_buffer
7514: static PyObject * THPVariable__nested_view_from_buffer(PyObject* self_, PyObject* args, PyObject* kwargs)
7515: {
7516:   HANDLE_TH_ERRORS
7517:   static PythonArgParser parser({
7518:     "_nested_view_from_buffer(Tensor input, Tensor nested_size, Tensor nested_strides, Tensor offsets)",
7519:   }, /*traceable=*/true);
7520: 
```

- EN: The main execution path in this span is carried by `trapz`, `wrap`, `THPVariable__nested_tensor_from_mask_left_aligned`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `trapz`, `wrap`, `THPVariable__nested_tensor_from_mask_left_aligned` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7521-7600

```cpp
7521:   ParsedArgs<4> parsed_args;
7522:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7523:   if(_r.has_torch_function()) {
7524:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7525:   }
7526:   // aten::_nested_view_from_buffer(Tensor(a) self, Tensor nested_size, Tensor nested_strides, Tensor offsets) -> Tensor(a)
7527: 
7528:   auto dispatch__nested_view_from_buffer = [](const at::Tensor & self, const at::Tensor & nested_size, const at::Tensor & nested_strides, const at::Tensor & offsets) -> at::Tensor {
7529:     pybind11::gil_scoped_release no_gil;
7530:     return at::_nested_view_from_buffer(self, nested_size, nested_strides, offsets);
7531:   };
7532:   return wrap(dispatch__nested_view_from_buffer(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
7533:   Py_RETURN_NONE;
7534:   END_HANDLE_TH_ERRORS
7535: }
7536: 
7537: // _nested_get_values_copy
7538: static PyObject * THPVariable__nested_get_values_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
7539: {
7540:   HANDLE_TH_ERRORS
7541:   static PythonArgParser parser({
7542:     "_nested_get_values_copy(Tensor input, *, Tensor out=None)",
7543:   }, /*traceable=*/true);
7544: 
7545:   ParsedArgs<2> parsed_args;
7546:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7547:   if(_r.has_torch_function()) {
7548:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7549:   }
7550:   if (_r.isNone(1)) {
7551:     // aten::_nested_get_values_copy(Tensor self) -> Tensor
7552: 
7553:     auto dispatch__nested_get_values_copy = [](const at::Tensor & self) -> at::Tensor {
7554:       pybind11::gil_scoped_release no_gil;
7555:       return at::_nested_get_values_copy(self);
7556:     };
7557:     return wrap(dispatch__nested_get_values_copy(_r.tensor(0)));
7558:   } else {
7559:     // aten::_nested_get_values_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
7560: 
7561:     auto dispatch__nested_get_values_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
7562:       pybind11::gil_scoped_release no_gil;
7563:       return at::_nested_get_values_copy_out(out, self);
7564:     };
7565:     return wrap(dispatch__nested_get_values_copy_out(_r.tensor(1), _r.tensor(0)));
7566:   }
7567:   Py_RETURN_NONE;
7568:   END_HANDLE_TH_ERRORS
7569: }
7570: 
7571: // _nested_get_lengths
7572: static PyObject * THPVariable__nested_get_lengths(PyObject* self_, PyObject* args, PyObject* kwargs)
7573: {
7574:   HANDLE_TH_ERRORS
7575:   static PythonArgParser parser({
7576:     "_nested_get_lengths(Tensor input)",
7577:   }, /*traceable=*/true);
7578: 
7579:   ParsedArgs<1> parsed_args;
7580:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7581:   if(_r.has_torch_function()) {
7582:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7583:   }
7584:   // aten::_nested_get_lengths(Tensor self) -> Tensor
7585: 
7586:   auto dispatch__nested_get_lengths = [](const at::Tensor & self) -> at::Tensor {
7587:     pybind11::gil_scoped_release no_gil;
7588:     return at::_nested_get_lengths(self);
7589:   };
7590:   return wrap(dispatch__nested_get_lengths(_r.tensor(0)));
7591:   Py_RETURN_NONE;
7592:   END_HANDLE_TH_ERRORS
7593: }
7594: 
7595: // trunc
7596: static PyObject * THPVariable_trunc(PyObject* self_, PyObject* args, PyObject* kwargs)
7597: {
7598:   HANDLE_TH_ERRORS
7599:   static PythonArgParser parser({
7600:     "trunc(Tensor input, *, Tensor out=None)",
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_nested_view_from_buffer`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_nested_view_from_buffer`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7601-7680

```cpp
7601:   }, /*traceable=*/true);
7602: 
7603:   ParsedArgs<2> parsed_args;
7604:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7605:   if(_r.has_torch_function()) {
7606:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7607:   }
7608:   if (_r.isNone(1)) {
7609:     // aten::trunc(Tensor self) -> Tensor
7610: 
7611:     auto dispatch_trunc = [](const at::Tensor & self) -> at::Tensor {
7612:       pybind11::gil_scoped_release no_gil;
7613:       return self.trunc();
7614:     };
7615:     return wrap(dispatch_trunc(_r.tensor(0)));
7616:   } else {
7617:     // aten::trunc.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
7618: 
7619:     auto dispatch_trunc_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
7620:       pybind11::gil_scoped_release no_gil;
7621:       return at::trunc_out(out, self);
7622:     };
7623:     return wrap(dispatch_trunc_out(_r.tensor(1), _r.tensor(0)));
7624:   }
7625:   Py_RETURN_NONE;
7626:   END_HANDLE_TH_ERRORS
7627: }
7628: 
7629: // trunc_
7630: static PyObject * THPVariable_trunc_(PyObject* self_, PyObject* args, PyObject* kwargs)
7631: {
7632:   HANDLE_TH_ERRORS
7633:   static PythonArgParser parser({
7634:     "trunc_(Tensor input)",
7635:   }, /*traceable=*/true);
7636: 
7637:   ParsedArgs<1> parsed_args;
7638:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7639:   if(_r.has_torch_function()) {
7640:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7641:   }
7642:   // aten::trunc_(Tensor(a!) self) -> Tensor(a!)
7643: 
7644:   auto dispatch_trunc_ = [](at::Tensor self) -> at::Tensor {
7645:     pybind11::gil_scoped_release no_gil;
7646:     return self.trunc_();
7647:   };
7648:   return wrap(dispatch_trunc_(_r.tensor(0)));
7649:   Py_RETURN_NONE;
7650:   END_HANDLE_TH_ERRORS
7651: }
7652: 
7653: // unique_dim
7654: static PyObject * THPVariable_unique_dim(PyObject* self_, PyObject* args, PyObject* kwargs)
7655: {
7656:   HANDLE_TH_ERRORS
7657:   static PythonArgParser parser({
7658:     "unique_dim(Tensor input, int64_t dim, bool sorted=True, bool return_inverse=False, bool return_counts=False)",
7659:   }, /*traceable=*/true);
7660: 
7661:   ParsedArgs<5> parsed_args;
7662:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7663:   if(_r.has_torch_function()) {
7664:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7665:   }
7666:   // aten::unique_dim(Tensor self, int dim, bool sorted=True, bool return_inverse=False, bool return_counts=False) -> (Tensor, Tensor, Tensor)
7667: 
7668:   auto dispatch_unique_dim = [](const at::Tensor & self, int64_t dim, bool sorted, bool return_inverse, bool return_counts) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
7669:     pybind11::gil_scoped_release no_gil;
7670:     return at::unique_dim(self, dim, sorted, return_inverse, return_counts);
7671:   };
7672:   return wrap(dispatch_unique_dim(_r.tensor(0), _r.toInt64(1), _r.toBool(2), _r.toBool(3), _r.toBool(4)));
7673:   Py_RETURN_NONE;
7674:   END_HANDLE_TH_ERRORS
7675: }
7676: 
7677: // unique_consecutive
7678: static PyObject * THPVariable_unique_consecutive(PyObject* self_, PyObject* args, PyObject* kwargs)
7679: {
7680:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `trunc`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `trunc`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7681-7760

```cpp
7681:   static PythonArgParser parser({
7682:     "unique_consecutive(Tensor input, bool return_inverse=False, bool return_counts=False, int64_t? dim=None)",
7683:   }, /*traceable=*/true);
7684: 
7685:   ParsedArgs<4> parsed_args;
7686:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7687:   if(_r.has_torch_function()) {
7688:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7689:   }
7690:   // aten::unique_consecutive(Tensor self, bool return_inverse=False, bool return_counts=False, int? dim=None) -> (Tensor, Tensor, Tensor)
7691: 
7692:   auto dispatch_unique_consecutive = [](const at::Tensor & self, bool return_inverse, bool return_counts, ::std::optional<int64_t> dim) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
7693:     pybind11::gil_scoped_release no_gil;
7694:     return at::unique_consecutive(self, return_inverse, return_counts, dim);
7695:   };
7696:   return wrap(dispatch_unique_consecutive(_r.tensor(0), _r.toBool(1), _r.toBool(2), _r.toInt64Optional(3)));
7697:   Py_RETURN_NONE;
7698:   END_HANDLE_TH_ERRORS
7699: }
7700: 
7701: \
7702: // where
7703: static PyObject * THPVariable_where(PyObject* self_, PyObject* args, PyObject* kwargs)
7704: {
7705:   HANDLE_TH_ERRORS
7706:   static PythonArgParser parser({
7707:     "where(Tensor condition)",
7708:     "where(Tensor condition, Tensor input, Tensor other, *, Tensor out=None)",
7709:     "where(Tensor condition, Scalar self, Tensor other)",
7710:     "where(Tensor condition, Tensor input, Scalar other)",
7711:     "where(Tensor condition, Scalar self, Scalar other)",
7712:   }, /*traceable=*/true);
7713: 
7714:   ParsedArgs<4> parsed_args;
7715:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7716:   if(_r.has_torch_function()) {
7717:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7718:   }
7719:   switch (_r.idx) {
7720:     case 0: {
7721:       // aten::where(Tensor condition) -> Tensor[]
7722: 
7723:       auto dispatch_where = [](const at::Tensor & condition) -> ::std::vector<at::Tensor> {
7724:         pybind11::gil_scoped_release no_gil;
7725:         return at::where(condition);
7726:       };
7727:       return wrap(dispatch_where(_r.tensor(0)));
7728:     }
7729:     case 1: {
7730:       if (_r.isNone(3)) {
7731:         // aten::where.self(Tensor condition, Tensor self, Tensor other) -> Tensor
7732: 
7733:         auto dispatch_where = [](const at::Tensor & condition, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7734:           pybind11::gil_scoped_release no_gil;
7735:           return self.where(condition, other);
7736:         };
7737:         return wrap(dispatch_where(_r.tensor(0), _r.tensor(1), _r.tensor(2)));
7738:       } else {
7739:         // aten::where.self_out(Tensor condition, Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
7740: 
7741:         auto dispatch_where_out = [](at::Tensor out, const at::Tensor & condition, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7742:           pybind11::gil_scoped_release no_gil;
7743:           return at::where_out(out, condition, self, other);
7744:         };
7745:         return wrap(dispatch_where_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.tensor(2)));
7746:       }
7747:     }
7748:     case 2: {
7749:       // aten::where.ScalarSelf(Tensor condition, Scalar self, Tensor other) -> Tensor
7750: 
7751:       auto dispatch_where = [](const at::Tensor & condition, const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
7752:         pybind11::gil_scoped_release no_gil;
7753:         return at::where(condition, self, other);
7754:       };
7755:       return wrap(dispatch_where(_r.tensor(0), _r.scalar(1), _r.tensor(2)));
7756:     }
7757:     case 3: {
7758:       // aten::where.ScalarOther(Tensor condition, Tensor self, Scalar other) -> Tensor
7759: 
7760:       auto dispatch_where = [](const at::Tensor & condition, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `unique_consecutive`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `unique_consecutive` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7761-7840

```cpp
7761:         pybind11::gil_scoped_release no_gil;
7762:         return self.where(condition, other);
7763:       };
7764:       return wrap(dispatch_where(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
7765:     }
7766:     case 4: {
7767:       // aten::where.Scalar(Tensor condition, Scalar self, Scalar other) -> Tensor
7768: 
7769:       auto dispatch_where = [](const at::Tensor & condition, const at::Scalar & self, const at::Scalar & other) -> at::Tensor {
7770:         pybind11::gil_scoped_release no_gil;
7771:         return at::where(condition, self, other);
7772:       };
7773:       return wrap(dispatch_where(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
7774:     }
7775:   }
7776:   Py_RETURN_NONE;
7777:   END_HANDLE_TH_ERRORS
7778: }
7779: 
7780: // _standard_gamma
7781: static PyObject * THPVariable__standard_gamma(PyObject* self_, PyObject* args, PyObject* kwargs)
7782: {
7783:   HANDLE_TH_ERRORS
7784:   static PythonArgParser parser({
7785:     "_standard_gamma(Tensor input, Generator? generator=None)",
7786:   }, /*traceable=*/true);
7787: 
7788:   ParsedArgs<2> parsed_args;
7789:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7790:   if(_r.has_torch_function()) {
7791:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7792:   }
7793:   // aten::_standard_gamma(Tensor self, Generator? generator=None) -> Tensor
7794: 
7795:   auto dispatch__standard_gamma = [](const at::Tensor & self, ::std::optional<at::Generator> generator) -> at::Tensor {
7796:     pybind11::gil_scoped_release no_gil;
7797:     return at::_standard_gamma(self, generator);
7798:   };
7799:   return wrap(dispatch__standard_gamma(_r.tensor(0), _r.generator(1)));
7800:   Py_RETURN_NONE;
7801:   END_HANDLE_TH_ERRORS
7802: }
7803: 
7804: // _philox_key_fold_in
7805: static PyObject * THPVariable__philox_key_fold_in(PyObject* self_, PyObject* args, PyObject* kwargs)
7806: {
7807:   HANDLE_TH_ERRORS
7808:   static PythonArgParser parser({
7809:     "_philox_key_fold_in(Tensor key, int64_t data)",
7810:   }, /*traceable=*/true);
7811: 
7812:   ParsedArgs<2> parsed_args;
7813:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7814:   if(_r.has_torch_function()) {
7815:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7816:   }
7817:   // aten::_philox_key_fold_in(Tensor key, int data) -> Tensor
7818: 
7819:   auto dispatch__philox_key_fold_in = [](const at::Tensor & key, int64_t data) -> at::Tensor {
7820:     pybind11::gil_scoped_release no_gil;
7821:     return at::_philox_key_fold_in(key, data);
7822:   };
7823:   return wrap(dispatch__philox_key_fold_in(_r.tensor(0), _r.toInt64(1)));
7824:   Py_RETURN_NONE;
7825:   END_HANDLE_TH_ERRORS
7826: }
7827: 
7828: // _philox_uniform_
7829: static PyObject * THPVariable__philox_uniform_(PyObject* self_, PyObject* args, PyObject* kwargs)
7830: {
7831:   HANDLE_TH_ERRORS
7832:   static PythonArgParser parser({
7833:     "_philox_uniform_(Tensor input, Tensor key, double low=0, double high=1)",
7834:   }, /*traceable=*/true);
7835: 
7836:   ParsedArgs<4> parsed_args;
7837:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7838:   if(_r.has_torch_function()) {
7839:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7840:   }
```

- EN: The main execution path in this span is carried by `wrap`, `where`, `THPVariable__standard_gamma`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `where`, `THPVariable__standard_gamma` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7841-7920

```cpp
7841:   // aten::_philox_uniform_(Tensor(a!) self, Tensor key, float low=0, float high=1) -> Tensor(a!)
7842: 
7843:   auto dispatch__philox_uniform_ = [](at::Tensor self, const at::Tensor & key, double low, double high) -> at::Tensor {
7844:     pybind11::gil_scoped_release no_gil;
7845:     return self._philox_uniform_(key, low, high);
7846:   };
7847:   return wrap(dispatch__philox_uniform_(_r.tensor(0), _r.tensor(1), _r.toDouble(2), _r.toDouble(3)));
7848:   Py_RETURN_NONE;
7849:   END_HANDLE_TH_ERRORS
7850: }
7851: 
7852: // _dirichlet_grad
7853: static PyObject * THPVariable__dirichlet_grad(PyObject* self_, PyObject* args, PyObject* kwargs)
7854: {
7855:   HANDLE_TH_ERRORS
7856:   static PythonArgParser parser({
7857:     "_dirichlet_grad(Tensor x, Tensor alpha, Tensor total)",
7858:   }, /*traceable=*/true);
7859: 
7860:   ParsedArgs<3> parsed_args;
7861:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7862:   if(_r.has_torch_function()) {
7863:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7864:   }
7865:   // aten::_dirichlet_grad(Tensor x, Tensor alpha, Tensor total) -> Tensor
7866: 
7867:   auto dispatch__dirichlet_grad = [](const at::Tensor & x, const at::Tensor & alpha, const at::Tensor & total) -> at::Tensor {
7868:     pybind11::gil_scoped_release no_gil;
7869:     return at::_dirichlet_grad(x, alpha, total);
7870:   };
7871:   return wrap(dispatch__dirichlet_grad(_r.tensor(0), _r.tensor(1), _r.tensor(2)));
7872:   Py_RETURN_NONE;
7873:   END_HANDLE_TH_ERRORS
7874: }
7875: 
7876: // binomial
7877: static PyObject * THPVariable_binomial(PyObject* self_, PyObject* args, PyObject* kwargs)
7878: {
7879:   HANDLE_TH_ERRORS
7880:   static PythonArgParser parser({
7881:     "binomial(Tensor count, Tensor prob, Generator? generator=None)",
7882:   }, /*traceable=*/true);
7883: 
7884:   ParsedArgs<3> parsed_args;
7885:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7886:   if(_r.has_torch_function()) {
7887:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7888:   }
7889:   // aten::binomial(Tensor count, Tensor prob, Generator? generator=None) -> Tensor
7890: 
7891:   auto dispatch_binomial = [](const at::Tensor & count, const at::Tensor & prob, ::std::optional<at::Generator> generator) -> at::Tensor {
7892:     pybind11::gil_scoped_release no_gil;
7893:     return at::binomial(count, prob, generator);
7894:   };
7895:   return wrap(dispatch_binomial(_r.tensor(0), _r.tensor(1), _r.generator(2)));
7896:   Py_RETURN_NONE;
7897:   END_HANDLE_TH_ERRORS
7898: }
7899: 
7900: // _sparse_csr_sum
7901: static PyObject * THPVariable__sparse_csr_sum(PyObject* self_, PyObject* args, PyObject* kwargs)
7902: {
7903:   HANDLE_TH_ERRORS
7904:   static PythonArgParser parser({
7905:     "_sparse_csr_sum(Tensor input, IntArrayRef[1] dim, bool keepdim=False, *, ScalarType? dtype=None)",
7906:   }, /*traceable=*/true);
7907: 
7908:   ParsedArgs<4> parsed_args;
7909:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7910:   if(_r.has_torch_function()) {
7911:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7912:   }
7913:   // aten::_sparse_csr_sum.dim_dtype(Tensor self, int[1] dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
7914: 
7915:   auto dispatch__sparse_csr_sum = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7916:     pybind11::gil_scoped_release no_gil;
7917:     return at::_sparse_csr_sum(self, dim, keepdim, dtype);
7918:   };
7919:   return wrap(dispatch__sparse_csr_sum(_r.tensor(0), _r.intlist(1), _r.toBool(2), _r.scalartypeOptional(3)));
7920:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `_philox_uniform_`, `wrap`, `THPVariable__dirichlet_grad`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_philox_uniform_`, `wrap`, `THPVariable__dirichlet_grad` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7921-8000

```cpp
7921:   END_HANDLE_TH_ERRORS
7922: }
7923: 
7924: // _sparse_log_softmax_backward_data
7925: static PyObject * THPVariable__sparse_log_softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs)
7926: {
7927:   HANDLE_TH_ERRORS
7928:   static PythonArgParser parser({
7929:     "_sparse_log_softmax_backward_data(Tensor grad_output, Tensor output, int64_t dim, Tensor input)",
7930:   }, /*traceable=*/true);
7931: 
7932:   ParsedArgs<4> parsed_args;
7933:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7934:   if(_r.has_torch_function()) {
7935:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7936:   }
7937:   // aten::_sparse_log_softmax_backward_data(Tensor grad_output, Tensor output, int dim, Tensor self) -> Tensor
7938: 
7939:   auto dispatch__sparse_log_softmax_backward_data = [](const at::Tensor & grad_output, const at::Tensor & output, int64_t dim, const at::Tensor & self) -> at::Tensor {
7940:     pybind11::gil_scoped_release no_gil;
7941:     return at::_sparse_log_softmax_backward_data(grad_output, output, dim, self);
7942:   };
7943:   return wrap(dispatch__sparse_log_softmax_backward_data(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.tensor(3)));
7944:   Py_RETURN_NONE;
7945:   END_HANDLE_TH_ERRORS
7946: }
7947: 
7948: \
7949: // nuclear_norm
7950: static PyObject * THPVariable_nuclear_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
7951: {
7952:   HANDLE_TH_ERRORS
7953:   static PythonArgParser parser({
7954:     "nuclear_norm(Tensor input, IntArrayRef[2] dim, bool keepdim=False, *, Tensor out=None)",
7955:     "nuclear_norm(Tensor input, bool keepdim=False, *, Tensor out=None)",
7956:   }, /*traceable=*/true);
7957: 
7958:   ParsedArgs<4> parsed_args;
7959:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7960:   if(_r.has_torch_function()) {
7961:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7962:   }
7963:   switch (_r.idx) {
7964:     case 0: {
7965:       if (_r.isNone(3)) {
7966:         // aten::nuclear_norm.dim(Tensor self, int[2] dim, bool keepdim=False) -> Tensor
7967: 
7968:         auto dispatch_nuclear_norm = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
7969:           pybind11::gil_scoped_release no_gil;
7970:           return at::nuclear_norm(self, dim, keepdim);
7971:         };
7972:         return wrap(dispatch_nuclear_norm(_r.tensor(0), _r.intlist(1), _r.toBool(2)));
7973:       } else {
7974:         // aten::nuclear_norm.dim_out(Tensor self, int[2] dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
7975: 
7976:         auto dispatch_nuclear_norm_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
7977:           pybind11::gil_scoped_release no_gil;
7978:           return at::nuclear_norm_out(out, self, dim, keepdim);
7979:         };
7980:         return wrap(dispatch_nuclear_norm_out(_r.tensor(3), _r.tensor(0), _r.intlist(1), _r.toBool(2)));
7981:       }
7982:     }
7983:     case 1: {
7984:       if (_r.isNone(2)) {
7985:         // aten::nuclear_norm(Tensor self, bool keepdim=False) -> Tensor
7986: 
7987:         auto dispatch_nuclear_norm = [](const at::Tensor & self, bool keepdim) -> at::Tensor {
7988:           pybind11::gil_scoped_release no_gil;
7989:           return at::nuclear_norm(self, keepdim);
7990:         };
7991:         return wrap(dispatch_nuclear_norm(_r.tensor(0), _r.toBool(1)));
7992:       } else {
7993:         // aten::nuclear_norm.out(Tensor self, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
7994: 
7995:         auto dispatch_nuclear_norm_out = [](at::Tensor out, const at::Tensor & self, bool keepdim) -> at::Tensor {
7996:           pybind11::gil_scoped_release no_gil;
7997:           return at::nuclear_norm_out(out, self, keepdim);
7998:         };
7999:         return wrap(dispatch_nuclear_norm_out(_r.tensor(2), _r.tensor(0), _r.toBool(1)));
8000:       }
```

- EN: The main execution path in this span is carried by `THPVariable__sparse_log_softmax_backward_data`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__sparse_log_softmax_backward_data`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 8001-8080

```cpp
8001:     }
8002:   }
8003:   Py_RETURN_NONE;
8004:   END_HANDLE_TH_ERRORS
8005: }
8006: 
8007: // clone
8008: static PyObject * THPVariable_clone(PyObject* self_, PyObject* args, PyObject* kwargs)
8009: {
8010:   HANDLE_TH_ERRORS
8011:   static PythonArgParser parser({
8012:     "clone(Tensor input, *, MemoryFormat? memory_format=None)",
8013:   }, /*traceable=*/true);
8014: 
8015:   ParsedArgs<2> parsed_args;
8016:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8017:   if(_r.has_torch_function()) {
8018:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8019:   }
8020:   // aten::clone(Tensor self, *, MemoryFormat? memory_format=None) -> Tensor
8021: 
8022:   auto dispatch_clone = [](const at::Tensor & self, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
8023:     pybind11::gil_scoped_release no_gil;
8024:     return self.clone(memory_format);
8025:   };
8026:   return wrap(dispatch_clone(_r.tensor(0), _r.memoryformatOptional(1)));
8027:   Py_RETURN_NONE;
8028:   END_HANDLE_TH_ERRORS
8029: }
8030: 
8031: // heaviside
8032: static PyObject * THPVariable_heaviside(PyObject* self_, PyObject* args, PyObject* kwargs)
8033: {
8034:   HANDLE_TH_ERRORS
8035:   static PythonArgParser parser({
8036:     "heaviside(Tensor input, Tensor values, *, Tensor out=None)",
8037:   }, /*traceable=*/true);
8038: 
8039:   ParsedArgs<3> parsed_args;
8040:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8041:   if(_r.has_torch_function()) {
8042:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8043:   }
8044:   if (_r.isNone(2)) {
8045:     // aten::heaviside(Tensor self, Tensor values) -> Tensor
8046: 
8047:     auto dispatch_heaviside = [](const at::Tensor & self, const at::Tensor & values) -> at::Tensor {
8048:       pybind11::gil_scoped_release no_gil;
8049:       return self.heaviside(values);
8050:     };
8051:     return wrap(dispatch_heaviside(_r.tensor(0), _r.tensor(1)));
8052:   } else {
8053:     // aten::heaviside.out(Tensor self, Tensor values, *, Tensor(a!) out) -> Tensor(a!)
8054: 
8055:     auto dispatch_heaviside_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & values) -> at::Tensor {
8056:       pybind11::gil_scoped_release no_gil;
8057:       return at::heaviside_out(out, self, values);
8058:     };
8059:     return wrap(dispatch_heaviside_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8060:   }
8061:   Py_RETURN_NONE;
8062:   END_HANDLE_TH_ERRORS
8063: }
8064: 
8065: // _scaled_mm
8066: static PyObject * THPVariable__scaled_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
8067: {
8068:   HANDLE_TH_ERRORS
8069:   static PythonArgParser parser({
8070:     "_scaled_mm(Tensor input, Tensor mat2, Tensor scale_a, Tensor scale_b, Tensor? bias=None, Tensor? scale_result=None, ScalarType? out_dtype=None, bool use_fast_accum=False, *, Tensor out=None)",
8071:   }, /*traceable=*/true);
8072: 
8073:   ParsedArgs<9> parsed_args;
8074:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8075:   if(_r.has_torch_function()) {
8076:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8077:   }
8078:   if (_r.isNone(8)) {
8079:     // aten::_scaled_mm(Tensor self, Tensor mat2, Tensor scale_a, Tensor scale_b, Tensor? bias=None, Tensor? scale_result=None, ScalarType? out_dtype=None, bool use_fast_accum=False) -> Tensor
8080: 
```

- EN: The main execution path in this span is carried by `THPVariable_clone`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_clone`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8081-8160

```cpp
8081:     auto dispatch__scaled_mm = [](const at::Tensor & self, const at::Tensor & mat2, const at::Tensor & scale_a, const at::Tensor & scale_b, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & scale_result, ::std::optional<at::ScalarType> out_dtype, bool use_fast_accum) -> at::Tensor {
8082:       pybind11::gil_scoped_release no_gil;
8083:       return at::_scaled_mm(self, mat2, scale_a, scale_b, bias, scale_result, out_dtype, use_fast_accum);
8084:     };
8085:     return wrap(dispatch__scaled_mm(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.optionalTensor(4), _r.optionalTensor(5), _r.scalartypeOptional(6), _r.toBool(7)));
8086:   } else {
8087:     // aten::_scaled_mm.out(Tensor self, Tensor mat2, Tensor scale_a, Tensor scale_b, Tensor? bias=None, Tensor? scale_result=None, ScalarType? out_dtype=None, bool use_fast_accum=False, *, Tensor(a!) out) -> Tensor(a!)
8088: 
8089:     auto dispatch__scaled_mm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat2, const at::Tensor & scale_a, const at::Tensor & scale_b, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & scale_result, ::std::optional<at::ScalarType> out_dtype, bool use_fast_accum) -> at::Tensor {
8090:       pybind11::gil_scoped_release no_gil;
8091:       return at::_scaled_mm_out(out, self, mat2, scale_a, scale_b, bias, scale_result, out_dtype, use_fast_accum);
8092:     };
8093:     return wrap(dispatch__scaled_mm_out(_r.tensor(8), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.optionalTensor(4), _r.optionalTensor(5), _r.scalartypeOptional(6), _r.toBool(7)));
8094:   }
8095:   Py_RETURN_NONE;
8096:   END_HANDLE_TH_ERRORS
8097: }
8098: 
8099: // _scaled_grouped_mm_v2
8100: static PyObject * THPVariable__scaled_grouped_mm_v2(PyObject* self_, PyObject* args, PyObject* kwargs)
8101: {
8102:   HANDLE_TH_ERRORS
8103:   static PythonArgParser parser({
8104:     "_scaled_grouped_mm_v2(Tensor input, Tensor mat2, TensorList scale_a, IntArrayRef recipe_a, IntArrayRef swizzle_a, TensorList scale_b, IntArrayRef recipe_b, IntArrayRef swizzle_b, Tensor? offs=None, Tensor? bias=None, ScalarType? out_dtype=None, IntArrayRef contraction_dim=None, bool use_fast_accum=False)",
8105:   }, /*traceable=*/true);
8106: 
8107:   ParsedArgs<13> parsed_args;
8108:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8109:   if(_r.has_torch_function()) {
8110:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8111:   }
8112:   // aten::_scaled_grouped_mm_v2(Tensor self, Tensor mat2, Tensor[] scale_a, int[] recipe_a, int[] swizzle_a, Tensor[] scale_b, int[] recipe_b, int[] swizzle_b, Tensor? offs=None, Tensor? bias=None, ScalarType? out_dtype=None, int[] contraction_dim=[], bool use_fast_accum=False) -> Tensor
8113: 
8114:   auto dispatch__scaled_grouped_mm_v2 = [](const at::Tensor & self, const at::Tensor & mat2, at::TensorList scale_a, at::IntArrayRef recipe_a, at::IntArrayRef swizzle_a, at::TensorList scale_b, at::IntArrayRef recipe_b, at::IntArrayRef swizzle_b, const ::std::optional<at::Tensor> & offs, const ::std::optional<at::Tensor> & bias, ::std::optional<at::ScalarType> out_dtype, at::IntArrayRef contraction_dim, bool use_fast_accum) -> at::Tensor {
8115:     pybind11::gil_scoped_release no_gil;
8116:     return at::_scaled_grouped_mm_v2(self, mat2, scale_a, recipe_a, swizzle_a, scale_b, recipe_b, swizzle_b, offs, bias, out_dtype, contraction_dim, use_fast_accum);
8117:   };
8118:   return wrap(dispatch__scaled_grouped_mm_v2(_r.tensor(0), _r.tensor(1), _r.tensorlist(2), _r.intlist(3), _r.intlist(4), _r.tensorlist(5), _r.intlist(6), _r.intlist(7), _r.optionalTensor(8), _r.optionalTensor(9), _r.scalartypeOptional(10), _r.intlist(11), _r.toBool(12)));
8119:   Py_RETURN_NONE;
8120:   END_HANDLE_TH_ERRORS
8121: }
8122: 
8123: // _validate_sparse_compressed_tensor_args
8124: static PyObject * THPVariable__validate_sparse_compressed_tensor_args(PyObject* self_, PyObject* args, PyObject* kwargs)
8125: {
8126:   HANDLE_TH_ERRORS
8127:   static PythonArgParser parser({
8128:     "_validate_sparse_compressed_tensor_args(Tensor compressed_indices, Tensor plain_indices, Tensor values, IntArrayRef size, Layout layout, bool? check_pinning=None)",
8129:   }, /*traceable=*/false);
8130: 
8131:   ParsedArgs<6> parsed_args;
8132:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8133:   if(_r.has_torch_function()) {
8134:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8135:   }
8136:   // aten::_validate_sparse_compressed_tensor_args(Tensor compressed_indices, Tensor plain_indices, Tensor values, int[] size, Layout layout, bool? check_pinning=None) -> ()
8137: 
8138:   auto dispatch__validate_sparse_compressed_tensor_args = [](const at::Tensor & compressed_indices, const at::Tensor & plain_indices, const at::Tensor & values, at::IntArrayRef size, at::Layout layout, ::std::optional<bool> check_pinning) -> void {
8139:     pybind11::gil_scoped_release no_gil;
8140:     at::_validate_sparse_compressed_tensor_args(compressed_indices, plain_indices, values, size, layout, check_pinning);
8141:   };
8142:   dispatch__validate_sparse_compressed_tensor_args(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.intlist(3), _r.layout(4), _r.toBoolOptional(5));
8143:   Py_RETURN_NONE;
8144:   Py_RETURN_NONE;
8145:   END_HANDLE_TH_ERRORS
8146: }
8147: 
8148: // _validate_sparse_csr_tensor_args
8149: static PyObject * THPVariable__validate_sparse_csr_tensor_args(PyObject* self_, PyObject* args, PyObject* kwargs)
8150: {
8151:   HANDLE_TH_ERRORS
8152:   static PythonArgParser parser({
8153:     "_validate_sparse_csr_tensor_args(Tensor crow_indices, Tensor col_indices, Tensor values, IntArrayRef size, bool? check_pinning=None)",
8154:   }, /*traceable=*/false);
8155: 
8156:   ParsedArgs<5> parsed_args;
8157:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8158:   if(_r.has_torch_function()) {
8159:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8160:   }
```

- EN: The main execution path in this span is carried by `_scaled_mm`, `wrap`, `_scaled_mm_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_scaled_mm`, `wrap`, `_scaled_mm_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 8161-8240

```cpp
8161:   // aten::_validate_sparse_csr_tensor_args(Tensor crow_indices, Tensor col_indices, Tensor values, int[] size, bool? check_pinning=None) -> ()
8162: 
8163:   auto dispatch__validate_sparse_csr_tensor_args = [](const at::Tensor & crow_indices, const at::Tensor & col_indices, const at::Tensor & values, at::IntArrayRef size, ::std::optional<bool> check_pinning) -> void {
8164:     pybind11::gil_scoped_release no_gil;
8165:     at::_validate_sparse_csr_tensor_args(crow_indices, col_indices, values, size, check_pinning);
8166:   };
8167:   dispatch__validate_sparse_csr_tensor_args(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.intlist(3), _r.toBoolOptional(4));
8168:   Py_RETURN_NONE;
8169:   Py_RETURN_NONE;
8170:   END_HANDLE_TH_ERRORS
8171: }
8172: 
8173: // _to_cpu
8174: static PyObject * THPVariable__to_cpu(PyObject* self_, PyObject* args, PyObject* kwargs)
8175: {
8176:   HANDLE_TH_ERRORS
8177:   static PythonArgParser parser({
8178:     "_to_cpu(TensorList tensors)",
8179:   }, /*traceable=*/true);
8180: 
8181:   ParsedArgs<1> parsed_args;
8182:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8183:   if(_r.has_torch_function()) {
8184:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8185:   }
8186:   // aten::_to_cpu(Tensor[] tensors) -> Tensor[]
8187: 
8188:   auto dispatch__to_cpu = [](at::TensorList tensors) -> ::std::vector<at::Tensor> {
8189:     pybind11::gil_scoped_release no_gil;
8190:     return at::_to_cpu(tensors);
8191:   };
8192:   return wrap(dispatch__to_cpu(_r.tensorlist(0)));
8193:   Py_RETURN_NONE;
8194:   END_HANDLE_TH_ERRORS
8195: }
8196: 
8197: // hspmm
8198: static PyObject * THPVariable_hspmm(PyObject* self_, PyObject* args, PyObject* kwargs)
8199: {
8200:   HANDLE_TH_ERRORS
8201:   static PythonArgParser parser({
8202:     "hspmm(Tensor mat1, Tensor mat2, *, Tensor out=None)",
8203:   }, /*traceable=*/true);
8204: 
8205:   ParsedArgs<3> parsed_args;
8206:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8207:   if(_r.has_torch_function()) {
8208:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8209:   }
8210:   if (_r.isNone(2)) {
8211:     // aten::hspmm(Tensor mat1, Tensor mat2) -> Tensor
8212: 
8213:     auto dispatch_hspmm = [](const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
8214:       pybind11::gil_scoped_release no_gil;
8215:       return at::hspmm(mat1, mat2);
8216:     };
8217:     return wrap(dispatch_hspmm(_r.tensor(0), _r.tensor(1)));
8218:   } else {
8219:     // aten::hspmm.out(Tensor mat1, Tensor mat2, *, Tensor(a!) out) -> Tensor(a!)
8220: 
8221:     auto dispatch_hspmm_out = [](at::Tensor out, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
8222:       pybind11::gil_scoped_release no_gil;
8223:       return at::hspmm_out(out, mat1, mat2);
8224:     };
8225:     return wrap(dispatch_hspmm_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8226:   }
8227:   Py_RETURN_NONE;
8228:   END_HANDLE_TH_ERRORS
8229: }
8230: 
8231: // _to_sparse_semi_structured
8232: static PyObject * THPVariable__to_sparse_semi_structured(PyObject* self_, PyObject* args, PyObject* kwargs)
8233: {
8234:   HANDLE_TH_ERRORS
8235:   static PythonArgParser parser({
8236:     "_to_sparse_semi_structured(Tensor dense)",
8237:   }, /*traceable=*/true);
8238: 
8239:   ParsedArgs<1> parsed_args;
8240:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `_validate_sparse_csr_tensor_args`, `dispatch__validate_sparse_csr_tensor_args`, `THPVariable__to_cpu`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_validate_sparse_csr_tensor_args`, `dispatch__validate_sparse_csr_tensor_args`, `THPVariable__to_cpu` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 8241-8320

```cpp
8241:   if(_r.has_torch_function()) {
8242:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8243:   }
8244:   // aten::_to_sparse_semi_structured(Tensor dense) -> (Tensor, Tensor)
8245: 
8246:   auto dispatch__to_sparse_semi_structured = [](const at::Tensor & dense) -> ::std::tuple<at::Tensor,at::Tensor> {
8247:     pybind11::gil_scoped_release no_gil;
8248:     return at::_to_sparse_semi_structured(dense);
8249:   };
8250:   return wrap(dispatch__to_sparse_semi_structured(_r.tensor(0)));
8251:   Py_RETURN_NONE;
8252:   END_HANDLE_TH_ERRORS
8253: }
8254: 
8255: // q_per_channel_scales
8256: static PyObject * THPVariable_q_per_channel_scales(PyObject* self_, PyObject* args, PyObject* kwargs)
8257: {
8258:   HANDLE_TH_ERRORS
8259:   static PythonArgParser parser({
8260:     "q_per_channel_scales(Tensor input)",
8261:   }, /*traceable=*/true);
8262: 
8263:   ParsedArgs<1> parsed_args;
8264:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8265:   if(_r.has_torch_function()) {
8266:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8267:   }
8268:   // aten::q_per_channel_scales(Tensor self) -> Tensor
8269: 
8270:   auto dispatch_q_per_channel_scales = [](const at::Tensor & self) -> at::Tensor {
8271:     pybind11::gil_scoped_release no_gil;
8272:     return self.q_per_channel_scales();
8273:   };
8274:   return wrap(dispatch_q_per_channel_scales(_r.tensor(0)));
8275:   Py_RETURN_NONE;
8276:   END_HANDLE_TH_ERRORS
8277: }
8278: 
8279: // q_per_channel_zero_points
8280: static PyObject * THPVariable_q_per_channel_zero_points(PyObject* self_, PyObject* args, PyObject* kwargs)
8281: {
8282:   HANDLE_TH_ERRORS
8283:   static PythonArgParser parser({
8284:     "q_per_channel_zero_points(Tensor input)",
8285:   }, /*traceable=*/true);
8286: 
8287:   ParsedArgs<1> parsed_args;
8288:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8289:   if(_r.has_torch_function()) {
8290:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8291:   }
8292:   // aten::q_per_channel_zero_points(Tensor self) -> Tensor
8293: 
8294:   auto dispatch_q_per_channel_zero_points = [](const at::Tensor & self) -> at::Tensor {
8295:     pybind11::gil_scoped_release no_gil;
8296:     return self.q_per_channel_zero_points();
8297:   };
8298:   return wrap(dispatch_q_per_channel_zero_points(_r.tensor(0)));
8299:   Py_RETURN_NONE;
8300:   END_HANDLE_TH_ERRORS
8301: }
8302: 
8303: // q_per_channel_axis
8304: static PyObject * THPVariable_q_per_channel_axis(PyObject* self_, PyObject* args, PyObject* kwargs)
8305: {
8306:   HANDLE_TH_ERRORS
8307:   static PythonArgParser parser({
8308:     "q_per_channel_axis(Tensor input)",
8309:   }, /*traceable=*/false);
8310: 
8311:   ParsedArgs<1> parsed_args;
8312:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8313:   if(_r.has_torch_function()) {
8314:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8315:   }
8316:   // aten::q_per_channel_axis(Tensor self) -> int
8317: 
8318:   auto dispatch_q_per_channel_axis = [](const at::Tensor & self) -> int64_t {
8319:     pybind11::gil_scoped_release no_gil;
8320:     return self.q_per_channel_axis();
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_to_sparse_semi_structured`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_to_sparse_semi_structured`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8321-8400

```cpp
8321:   };
8322:   return wrap(dispatch_q_per_channel_axis(_r.tensor(0)));
8323:   Py_RETURN_NONE;
8324:   END_HANDLE_TH_ERRORS
8325: }
8326: 
8327: // _fake_quantize_per_tensor_affine_cachemask_tensor_qparams
8328: static PyObject * THPVariable__fake_quantize_per_tensor_affine_cachemask_tensor_qparams(PyObject* self_, PyObject* args, PyObject* kwargs)
8329: {
8330:   HANDLE_TH_ERRORS
8331:   static PyTypeObject* NamedTuple = generated::get__fake_quantize_per_tensor_affine_cachemask_tensor_qparams_structseq();
8332:   static PythonArgParser parser({
8333:     "_fake_quantize_per_tensor_affine_cachemask_tensor_qparams(Tensor input, Tensor scale, Tensor zero_point, Tensor fake_quant_enabled, int64_t quant_min, int64_t quant_max)",
8334:   }, /*traceable=*/true);
8335: 
8336:   ParsedArgs<6> parsed_args;
8337:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8338:   if(_r.has_torch_function()) {
8339:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8340:   }
8341:   // aten::_fake_quantize_per_tensor_affine_cachemask_tensor_qparams(Tensor self, Tensor scale, Tensor zero_point, Tensor fake_quant_enabled, int quant_min, int quant_max) -> (Tensor output, Tensor mask)
8342: 
8343:   auto dispatch__fake_quantize_per_tensor_affine_cachemask_tensor_qparams = [](const at::Tensor & self, const at::Tensor & scale, const at::Tensor & zero_point, const at::Tensor & fake_quant_enabled, int64_t quant_min, int64_t quant_max) -> ::std::tuple<at::Tensor,at::Tensor> {
8344:     pybind11::gil_scoped_release no_gil;
8345:     return at::_fake_quantize_per_tensor_affine_cachemask_tensor_qparams(self, scale, zero_point, fake_quant_enabled, quant_min, quant_max);
8346:   };
8347:   return wrap(NamedTuple, dispatch__fake_quantize_per_tensor_affine_cachemask_tensor_qparams(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.toInt64(4), _r.toInt64(5)));
8348:   Py_RETURN_NONE;
8349:   END_HANDLE_TH_ERRORS
8350: }
8351: 
8352: // _fake_quantize_learnable_per_tensor_affine
8353: static PyObject * THPVariable__fake_quantize_learnable_per_tensor_affine(PyObject* self_, PyObject* args, PyObject* kwargs)
8354: {
8355:   HANDLE_TH_ERRORS
8356:   static PythonArgParser parser({
8357:     "_fake_quantize_learnable_per_tensor_affine(Tensor input, Tensor scale, Tensor zero_point, int64_t quant_min, int64_t quant_max, double grad_factor=1.0)",
8358:   }, /*traceable=*/true);
8359: 
8360:   ParsedArgs<6> parsed_args;
8361:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8362:   if(_r.has_torch_function()) {
8363:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8364:   }
8365:   // aten::_fake_quantize_learnable_per_tensor_affine(Tensor self, Tensor scale, Tensor zero_point, int quant_min, int quant_max, float grad_factor=1.0) -> Tensor
8366: 
8367:   auto dispatch__fake_quantize_learnable_per_tensor_affine = [](const at::Tensor & self, const at::Tensor & scale, const at::Tensor & zero_point, int64_t quant_min, int64_t quant_max, double grad_factor) -> at::Tensor {
8368:     pybind11::gil_scoped_release no_gil;
8369:     return at::_fake_quantize_learnable_per_tensor_affine(self, scale, zero_point, quant_min, quant_max, grad_factor);
8370:   };
8371:   return wrap(dispatch__fake_quantize_learnable_per_tensor_affine(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3), _r.toInt64(4), _r.toDouble(5)));
8372:   Py_RETURN_NONE;
8373:   END_HANDLE_TH_ERRORS
8374: }
8375: 
8376: // fused_moving_avg_obs_fake_quant
8377: static PyObject * THPVariable_fused_moving_avg_obs_fake_quant(PyObject* self_, PyObject* args, PyObject* kwargs)
8378: {
8379:   HANDLE_TH_ERRORS
8380:   static PythonArgParser parser({
8381:     "fused_moving_avg_obs_fake_quant(Tensor input, Tensor observer_on, Tensor fake_quant_on, Tensor running_min, Tensor running_max, Tensor scale, Tensor zero_point, double averaging_const, int64_t quant_min, int64_t quant_max, int64_t ch_axis, bool per_row_fake_quant=False, bool symmetric_quant=False)",
8382:   }, /*traceable=*/true);
8383: 
8384:   ParsedArgs<13> parsed_args;
8385:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8386:   if(_r.has_torch_function()) {
8387:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8388:   }
8389:   // aten::fused_moving_avg_obs_fake_quant(Tensor self, Tensor observer_on, Tensor fake_quant_on, Tensor(a!) running_min, Tensor(b!) running_max, Tensor(c!) scale, Tensor(d!) zero_point, float averaging_const, int quant_min, int quant_max, int ch_axis, bool per_row_fake_quant=False, bool symmetric_quant=False) -> Tensor
8390: 
8391:   auto dispatch_fused_moving_avg_obs_fake_quant = [](const at::Tensor & self, const at::Tensor & observer_on, const at::Tensor & fake_quant_on, at::Tensor running_min, at::Tensor running_max, at::Tensor scale, at::Tensor zero_point, double averaging_const, int64_t quant_min, int64_t quant_max, int64_t ch_axis, bool per_row_fake_quant, bool symmetric_quant) -> at::Tensor {
8392:     pybind11::gil_scoped_release no_gil;
8393:     return at::fused_moving_avg_obs_fake_quant(self, observer_on, fake_quant_on, running_min, running_max, scale, zero_point, averaging_const, quant_min, quant_max, ch_axis, per_row_fake_quant, symmetric_quant);
8394:   };
8395:   return wrap(dispatch_fused_moving_avg_obs_fake_quant(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.tensor(4), _r.tensor(5), _r.tensor(6), _r.toDouble(7), _r.toInt64(8), _r.toInt64(9), _r.toInt64(10), _r.toBool(11), _r.toBool(12)));
8396:   Py_RETURN_NONE;
8397:   END_HANDLE_TH_ERRORS
8398: }
8399: 
8400: // _saturate_weight_to_fp16
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__fake_quantize_per_tensor_affine_cachemask_tensor_qparams`, `get__fake_quantize_per_tensor_affine_cachemask_tensor_qparams_structseq`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__fake_quantize_per_tensor_affine_cachemask_tensor_qparams`, `get__fake_quantize_per_tensor_affine_cachemask_tensor_qparams_structseq` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8401-8480

```cpp
8401: static PyObject * THPVariable__saturate_weight_to_fp16(PyObject* self_, PyObject* args, PyObject* kwargs)
8402: {
8403:   HANDLE_TH_ERRORS
8404:   static PythonArgParser parser({
8405:     "_saturate_weight_to_fp16(Tensor weight)",
8406:   }, /*traceable=*/true);
8407: 
8408:   ParsedArgs<1> parsed_args;
8409:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8410:   if(_r.has_torch_function()) {
8411:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8412:   }
8413:   // aten::_saturate_weight_to_fp16(Tensor weight) -> Tensor
8414: 
8415:   auto dispatch__saturate_weight_to_fp16 = [](const at::Tensor & weight) -> at::Tensor {
8416:     pybind11::gil_scoped_release no_gil;
8417:     return at::_saturate_weight_to_fp16(weight);
8418:   };
8419:   return wrap(dispatch__saturate_weight_to_fp16(_r.tensor(0)));
8420:   Py_RETURN_NONE;
8421:   END_HANDLE_TH_ERRORS
8422: }
8423: 
8424: // choose_qparams_optimized
8425: static PyObject * THPVariable_choose_qparams_optimized(PyObject* self_, PyObject* args, PyObject* kwargs)
8426: {
8427:   HANDLE_TH_ERRORS
8428:   static PythonArgParser parser({
8429:     "choose_qparams_optimized(Tensor input, int64_t numel, int64_t n_bins, double ratio, int64_t bit_width)",
8430:   }, /*traceable=*/true);
8431: 
8432:   ParsedArgs<5> parsed_args;
8433:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8434:   if(_r.has_torch_function()) {
8435:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8436:   }
8437:   // aten::choose_qparams_optimized(Tensor input, int numel, int n_bins, float ratio, int bit_width) -> (Tensor, Tensor)
8438: 
8439:   auto dispatch_choose_qparams_optimized = [](const at::Tensor & input, int64_t numel, int64_t n_bins, double ratio, int64_t bit_width) -> ::std::tuple<at::Tensor,at::Tensor> {
8440:     pybind11::gil_scoped_release no_gil;
8441:     return at::choose_qparams_optimized(input, numel, n_bins, ratio, bit_width);
8442:   };
8443:   return wrap(dispatch_choose_qparams_optimized(_r.tensor(0), _r.toInt64(1), _r.toInt64(2), _r.toDouble(3), _r.toInt64(4)));
8444:   Py_RETURN_NONE;
8445:   END_HANDLE_TH_ERRORS
8446: }
8447: 
8448: // combinations
8449: static PyObject * THPVariable_combinations(PyObject* self_, PyObject* args, PyObject* kwargs)
8450: {
8451:   HANDLE_TH_ERRORS
8452:   static PythonArgParser parser({
8453:     "combinations(Tensor input, int64_t r=2, bool with_replacement=False)",
8454:   }, /*traceable=*/true);
8455: 
8456:   ParsedArgs<3> parsed_args;
8457:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8458:   if(_r.has_torch_function()) {
8459:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8460:   }
8461:   // aten::combinations(Tensor self, int r=2, bool with_replacement=False) -> Tensor
8462: 
8463:   auto dispatch_combinations = [](const at::Tensor & self, int64_t r, bool with_replacement) -> at::Tensor {
8464:     pybind11::gil_scoped_release no_gil;
8465:     return at::combinations(self, r, with_replacement);
8466:   };
8467:   return wrap(dispatch_combinations(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
8468:   Py_RETURN_NONE;
8469:   END_HANDLE_TH_ERRORS
8470: }
8471: 
8472: \
8473: // result_type
8474: static PyObject * THPVariable_result_type(PyObject* self_, PyObject* args, PyObject* kwargs)
8475: {
8476:   HANDLE_TH_ERRORS
8477:   static PythonArgParser parser({
8478:     "result_type(Tensor tensor, Tensor other)",
8479:     "result_type(Scalar scalar, Tensor tensor)",
8480:     "result_type(Tensor tensor, Scalar other)",
```

- EN: The main execution path in this span is carried by `THPVariable__saturate_weight_to_fp16`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__saturate_weight_to_fp16`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8481-8560

```cpp
8481:     "result_type(Scalar scalar1, Scalar scalar2)",
8482:   }, /*traceable=*/false);
8483: 
8484:   ParsedArgs<2> parsed_args;
8485:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8486:   if(_r.has_torch_function()) {
8487:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8488:   }
8489:   switch (_r.idx) {
8490:     case 0: {
8491:       // aten::result_type.Tensor(Tensor tensor, Tensor other) -> ScalarType
8492: 
8493:       auto dispatch_result_type = [](const at::Tensor & tensor, const at::Tensor & other) -> at::ScalarType {
8494:         pybind11::gil_scoped_release no_gil;
8495:         return at::result_type(tensor, other);
8496:       };
8497:       return wrap(dispatch_result_type(_r.tensor(0), _r.tensor(1)));
8498:     }
8499:     case 1: {
8500:       // aten::result_type.Scalar_Tensor(Scalar scalar, Tensor tensor) -> ScalarType
8501: 
8502:       auto dispatch_result_type = [](const at::Scalar & scalar, const at::Tensor & tensor) -> at::ScalarType {
8503:         pybind11::gil_scoped_release no_gil;
8504:         return at::result_type(scalar, tensor);
8505:       };
8506:       return wrap(dispatch_result_type(_r.scalar(0), _r.tensor(1)));
8507:     }
8508:     case 2: {
8509:       // aten::result_type.Scalar(Tensor tensor, Scalar other) -> ScalarType
8510: 
8511:       auto dispatch_result_type = [](const at::Tensor & tensor, const at::Scalar & other) -> at::ScalarType {
8512:         pybind11::gil_scoped_release no_gil;
8513:         return at::result_type(tensor, other);
8514:       };
8515:       return wrap(dispatch_result_type(_r.tensor(0), _r.scalar(1)));
8516:     }
8517:     case 3: {
8518:       // aten::result_type.Scalar_Scalar(Scalar scalar1, Scalar scalar2) -> ScalarType
8519: 
8520:       auto dispatch_result_type = [](const at::Scalar & scalar1, const at::Scalar & scalar2) -> at::ScalarType {
8521:         pybind11::gil_scoped_release no_gil;
8522:         return at::result_type(scalar1, scalar2);
8523:       };
8524:       return wrap(dispatch_result_type(_r.scalar(0), _r.scalar(1)));
8525:     }
8526:   }
8527:   Py_RETURN_NONE;
8528:   END_HANDLE_TH_ERRORS
8529: }
8530: 
8531: // can_cast
8532: static PyObject * THPVariable_can_cast(PyObject* self_, PyObject* args, PyObject* kwargs)
8533: {
8534:   HANDLE_TH_ERRORS
8535:   static PythonArgParser parser({
8536:     "can_cast(ScalarType from_, ScalarType to)",
8537:   }, /*traceable=*/false);
8538: 
8539:   ParsedArgs<2> parsed_args;
8540:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8541:   if(_r.has_torch_function()) {
8542:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8543:   }
8544:   // aten::can_cast(ScalarType from_, ScalarType to) -> bool
8545: 
8546:   auto dispatch_can_cast = [](at::ScalarType from_, at::ScalarType to) -> bool {
8547:     pybind11::gil_scoped_release no_gil;
8548:     return at::can_cast(from_, to);
8549:   };
8550:   return wrap(dispatch_can_cast(_r.scalartype(0), _r.scalartype(1)));
8551:   Py_RETURN_NONE;
8552:   END_HANDLE_TH_ERRORS
8553: }
8554: 
8555: // _lstm_mps
8556: static PyObject * THPVariable__lstm_mps(PyObject* self_, PyObject* args, PyObject* kwargs)
8557: {
8558:   HANDLE_TH_ERRORS
8559:   static PythonArgParser parser({
8560:     "_lstm_mps(Tensor input, TensorList hx, TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional, bool batch_first)",
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `result_type`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `result_type`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8561-8640

```cpp
8561:   }, /*traceable=*/true);
8562: 
8563:   ParsedArgs<9> parsed_args;
8564:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8565:   if(_r.has_torch_function()) {
8566:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8567:   }
8568:   // aten::_lstm_mps(Tensor input, Tensor[] hx, Tensor[] params, bool has_biases, int num_layers, float dropout, bool train, bool bidirectional, bool batch_first) -> (Tensor, Tensor, Tensor, Tensor, Tensor, Tensor)
8569: 
8570:   auto dispatch__lstm_mps = [](const at::Tensor & input, at::TensorList hx, at::TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional, bool batch_first) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
8571:     pybind11::gil_scoped_release no_gil;
8572:     return at::_lstm_mps(input, hx, params, has_biases, num_layers, dropout, train, bidirectional, batch_first);
8573:   };
8574:   return wrap(dispatch__lstm_mps(_r.tensor(0), _r.tensorlist(1), _r.tensorlist(2), _r.toBool(3), _r.toInt64(4), _r.toDouble(5), _r.toBool(6), _r.toBool(7), _r.toBool(8)));
8575:   Py_RETURN_NONE;
8576:   END_HANDLE_TH_ERRORS
8577: }
8578: 
8579: \
8580: // lstm
8581: static PyObject * THPVariable_lstm(PyObject* self_, PyObject* args, PyObject* kwargs)
8582: {
8583:   HANDLE_TH_ERRORS
8584:   static PythonArgParser parser({
8585:     "lstm(Tensor data, Tensor batch_sizes, TensorList hx, TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional)",
8586:     "lstm(Tensor input, TensorList hx, TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional, bool batch_first)",
8587:   }, /*traceable=*/true);
8588: 
8589:   ParsedArgs<9> parsed_args;
8590:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8591:   if(_r.has_torch_function()) {
8592:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8593:   }
8594:   switch (_r.idx) {
8595:     case 0: {
8596:       // aten::lstm.data(Tensor data, Tensor batch_sizes, Tensor[] hx, Tensor[] params, bool has_biases, int num_layers, float dropout, bool train, bool bidirectional) -> (Tensor, Tensor, Tensor)
8597: 
8598:       auto dispatch_lstm = [](const at::Tensor & data, const at::Tensor & batch_sizes, at::TensorList hx, at::TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
8599:         pybind11::gil_scoped_release no_gil;
8600:         return at::lstm(data, batch_sizes, hx, params, has_biases, num_layers, dropout, train, bidirectional);
8601:       };
8602:       return wrap(dispatch_lstm(_r.tensor(0), _r.tensor(1), _r.tensorlist(2), _r.tensorlist(3), _r.toBool(4), _r.toInt64(5), _r.toDouble(6), _r.toBool(7), _r.toBool(8)));
8603:     }
8604:     case 1: {
8605:       // aten::lstm.input(Tensor input, Tensor[] hx, Tensor[] params, bool has_biases, int num_layers, float dropout, bool train, bool bidirectional, bool batch_first) -> (Tensor, Tensor, Tensor)
8606: 
8607:       auto dispatch_lstm = [](const at::Tensor & input, at::TensorList hx, at::TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional, bool batch_first) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
8608:         pybind11::gil_scoped_release no_gil;
8609:         return at::lstm(input, hx, params, has_biases, num_layers, dropout, train, bidirectional, batch_first);
8610:       };
8611:       return wrap(dispatch_lstm(_r.tensor(0), _r.tensorlist(1), _r.tensorlist(2), _r.toBool(3), _r.toInt64(4), _r.toDouble(5), _r.toBool(6), _r.toBool(7), _r.toBool(8)));
8612:     }
8613:   }
8614:   Py_RETURN_NONE;
8615:   END_HANDLE_TH_ERRORS
8616: }
8617: 
8618: // lstm_cell
8619: static PyObject * THPVariable_lstm_cell(PyObject* self_, PyObject* args, PyObject* kwargs)
8620: {
8621:   HANDLE_TH_ERRORS
8622:   static PythonArgParser parser({
8623:     "lstm_cell(Tensor input, TensorList hx, Tensor w_ih, Tensor w_hh, Tensor? b_ih=None, Tensor? b_hh=None)",
8624:   }, /*traceable=*/false);
8625: 
8626:   ParsedArgs<6> parsed_args;
8627:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8628:   if(_r.has_torch_function()) {
8629:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8630:   }
8631:   // aten::lstm_cell(Tensor input, Tensor[] hx, Tensor w_ih, Tensor w_hh, Tensor? b_ih=None, Tensor? b_hh=None) -> (Tensor, Tensor)
8632: 
8633:   auto dispatch_lstm_cell = [](const at::Tensor & input, at::TensorList hx, const at::Tensor & w_ih, const at::Tensor & w_hh, const ::std::optional<at::Tensor> & b_ih, const ::std::optional<at::Tensor> & b_hh) -> ::std::tuple<at::Tensor,at::Tensor> {
8634:     pybind11::gil_scoped_release no_gil;
8635:     return at::lstm_cell(input, hx, w_ih, w_hh, b_ih, b_hh);
8636:   };
8637:   return wrap(dispatch_lstm_cell(_r.tensor(0), _r.tensorlist(1), _r.tensor(2), _r.tensor(3), _r.optionalTensor(4), _r.optionalTensor(5)));
8638:   Py_RETURN_NONE;
8639:   END_HANDLE_TH_ERRORS
8640: }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_lstm_mps`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_lstm_mps`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8641-8720

```cpp
8641: 
8642: // quantized_rnn_relu_cell
8643: static PyObject * THPVariable_quantized_rnn_relu_cell(PyObject* self_, PyObject* args, PyObject* kwargs)
8644: {
8645:   HANDLE_TH_ERRORS
8646:   static PythonArgParser parser({
8647:     "quantized_rnn_relu_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor b_ih, Tensor b_hh, Tensor packed_ih, Tensor packed_hh, Tensor col_offsets_ih, Tensor col_offsets_hh, Scalar scale_ih, Scalar scale_hh, Scalar zero_point_ih, Scalar zero_point_hh)",
8648:   }, /*traceable=*/true);
8649: 
8650:   ParsedArgs<14> parsed_args;
8651:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8652:   if(_r.has_torch_function()) {
8653:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8654:   }
8655:   // aten::quantized_rnn_relu_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor b_ih, Tensor b_hh, Tensor packed_ih, Tensor packed_hh, Tensor col_offsets_ih, Tensor col_offsets_hh, Scalar scale_ih, Scalar scale_hh, Scalar zero_point_ih, Scalar zero_point_hh) -> Tensor
8656: 
8657:   auto dispatch_quantized_rnn_relu_cell = [](const at::Tensor & input, const at::Tensor & hx, const at::Tensor & w_ih, const at::Tensor & w_hh, const at::Tensor & b_ih, const at::Tensor & b_hh, const at::Tensor & packed_ih, const at::Tensor & packed_hh, const at::Tensor & col_offsets_ih, const at::Tensor & col_offsets_hh, const at::Scalar & scale_ih, const at::Scalar & scale_hh, const at::Scalar & zero_point_ih, const at::Scalar & zero_point_hh) -> at::Tensor {
8658:     pybind11::gil_scoped_release no_gil;
8659:     return at::quantized_rnn_relu_cell(input, hx, w_ih, w_hh, b_ih, b_hh, packed_ih, packed_hh, col_offsets_ih, col_offsets_hh, scale_ih, scale_hh, zero_point_ih, zero_point_hh);
8660:   };
8661:   return wrap(dispatch_quantized_rnn_relu_cell(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.tensor(4), _r.tensor(5), _r.tensor(6), _r.tensor(7), _r.tensor(8), _r.tensor(9), _r.scalar(10), _r.scalar(11), _r.scalar(12), _r.scalar(13)));
8662:   Py_RETURN_NONE;
8663:   END_HANDLE_TH_ERRORS
8664: }
8665: 
8666: // quantized_rnn_tanh_cell
8667: static PyObject * THPVariable_quantized_rnn_tanh_cell(PyObject* self_, PyObject* args, PyObject* kwargs)
8668: {
8669:   HANDLE_TH_ERRORS
8670:   static PythonArgParser parser({
8671:     "quantized_rnn_tanh_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor b_ih, Tensor b_hh, Tensor packed_ih, Tensor packed_hh, Tensor col_offsets_ih, Tensor col_offsets_hh, Scalar scale_ih, Scalar scale_hh, Scalar zero_point_ih, Scalar zero_point_hh)",
8672:   }, /*traceable=*/true);
8673: 
8674:   ParsedArgs<14> parsed_args;
8675:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8676:   if(_r.has_torch_function()) {
8677:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8678:   }
8679:   // aten::quantized_rnn_tanh_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor b_ih, Tensor b_hh, Tensor packed_ih, Tensor packed_hh, Tensor col_offsets_ih, Tensor col_offsets_hh, Scalar scale_ih, Scalar scale_hh, Scalar zero_point_ih, Scalar zero_point_hh) -> Tensor
8680: 
8681:   auto dispatch_quantized_rnn_tanh_cell = [](const at::Tensor & input, const at::Tensor & hx, const at::Tensor & w_ih, const at::Tensor & w_hh, const at::Tensor & b_ih, const at::Tensor & b_hh, const at::Tensor & packed_ih, const at::Tensor & packed_hh, const at::Tensor & col_offsets_ih, const at::Tensor & col_offsets_hh, const at::Scalar & scale_ih, const at::Scalar & scale_hh, const at::Scalar & zero_point_ih, const at::Scalar & zero_point_hh) -> at::Tensor {
8682:     pybind11::gil_scoped_release no_gil;
8683:     return at::quantized_rnn_tanh_cell(input, hx, w_ih, w_hh, b_ih, b_hh, packed_ih, packed_hh, col_offsets_ih, col_offsets_hh, scale_ih, scale_hh, zero_point_ih, zero_point_hh);
8684:   };
8685:   return wrap(dispatch_quantized_rnn_tanh_cell(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.tensor(4), _r.tensor(5), _r.tensor(6), _r.tensor(7), _r.tensor(8), _r.tensor(9), _r.scalar(10), _r.scalar(11), _r.scalar(12), _r.scalar(13)));
8686:   Py_RETURN_NONE;
8687:   END_HANDLE_TH_ERRORS
8688: }
8689: 
8690: \
8691: // index_add
8692: static PyObject * THPVariable_index_add(PyObject* self_, PyObject* args, PyObject* kwargs)
8693: {
8694:   HANDLE_TH_ERRORS
8695:   static PythonArgParser parser({
8696:     "index_add(Tensor input, int64_t dim, Tensor index, Tensor source, *, Scalar alpha=1, Tensor out=None)",
8697:     "index_add(Tensor input, Dimname dim, Tensor index, Tensor source, *, Scalar alpha=1)",
8698:   }, /*traceable=*/true);
8699: 
8700:   ParsedArgs<6> parsed_args;
8701:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8702:   if(_r.has_torch_function()) {
8703:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8704:   }
8705:   switch (_r.idx) {
8706:     case 0: {
8707:       if (_r.isNone(5)) {
8708:         // aten::index_add(Tensor self, int dim, Tensor index, Tensor source, *, Scalar alpha=1) -> Tensor
8709: 
8710:         auto dispatch_index_add = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, const at::Scalar & alpha) -> at::Tensor {
8711:           pybind11::gil_scoped_release no_gil;
8712:           return self.index_add(dim, index, source, alpha);
8713:         };
8714:         return wrap(dispatch_index_add(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3), _r.scalar(4)));
8715:       } else {
8716:         // aten::index_add.out(Tensor self, int dim, Tensor index, Tensor source, *, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
8717: 
8718:         auto dispatch_index_add_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, const at::Scalar & alpha) -> at::Tensor {
8719:           pybind11::gil_scoped_release no_gil;
8720:           return at::index_add_out(out, self, dim, index, source, alpha);
```

- EN: The main execution path in this span is carried by `THPVariable_quantized_rnn_relu_cell`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_quantized_rnn_relu_cell`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8721-8800

```cpp
8721:         };
8722:         return wrap(dispatch_index_add_out(_r.tensor(5), _r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3), _r.scalar(4)));
8723:       }
8724:     }
8725:     case 1: {
8726:       // aten::index_add.dimname(Tensor self, Dimname dim, Tensor index, Tensor source, *, Scalar alpha=1) -> Tensor
8727: 
8728:       auto dispatch_index_add = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & source, const at::Scalar & alpha) -> at::Tensor {
8729:         pybind11::gil_scoped_release no_gil;
8730:         return self.index_add(dim, index, source, alpha);
8731:       };
8732:       return wrap(dispatch_index_add(_r.tensor(0), _r.dimname(1), _r.tensor(2), _r.tensor(3), _r.scalar(4)));
8733:     }
8734:   }
8735:   Py_RETURN_NONE;
8736:   END_HANDLE_TH_ERRORS
8737: }
8738: 
8739: // index_reduce
8740: static PyObject * THPVariable_index_reduce(PyObject* self_, PyObject* args, PyObject* kwargs)
8741: {
8742:   HANDLE_TH_ERRORS
8743:   static PythonArgParser parser({
8744:     "index_reduce(Tensor input, int64_t dim, Tensor index, Tensor source, c10::string_view reduce, *, bool include_self=True, Tensor out=None)",
8745:   }, /*traceable=*/true);
8746: 
8747:   ParsedArgs<7> parsed_args;
8748:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8749:   if(_r.has_torch_function()) {
8750:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8751:   }
8752:   if (_r.isNone(6)) {
8753:     // aten::index_reduce(Tensor self, int dim, Tensor index, Tensor source, str reduce, *, bool include_self=True) -> Tensor
8754: 
8755:     auto dispatch_index_reduce = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, c10::string_view reduce, bool include_self) -> at::Tensor {
8756:       pybind11::gil_scoped_release no_gil;
8757:       return self.index_reduce(dim, index, source, reduce, include_self);
8758:     };
8759:     return wrap(dispatch_index_reduce(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3), _r.stringView(4), _r.toBool(5)));
8760:   } else {
8761:     // aten::index_reduce.out(Tensor self, int dim, Tensor index, Tensor source, str reduce, *, bool include_self=True, Tensor(a!) out) -> Tensor(a!)
8762: 
8763:     auto dispatch_index_reduce_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, c10::string_view reduce, bool include_self) -> at::Tensor {
8764:       pybind11::gil_scoped_release no_gil;
8765:       return at::index_reduce_out(out, self, dim, index, source, reduce, include_self);
8766:     };
8767:     return wrap(dispatch_index_reduce_out(_r.tensor(6), _r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3), _r.stringView(4), _r.toBool(5)));
8768:   }
8769:   Py_RETURN_NONE;
8770:   END_HANDLE_TH_ERRORS
8771: }
8772: 
8773: \
8774: // scatter_add
8775: static PyObject * THPVariable_scatter_add(PyObject* self_, PyObject* args, PyObject* kwargs)
8776: {
8777:   HANDLE_TH_ERRORS
8778:   static PythonArgParser parser({
8779:     "scatter_add(Tensor input, int64_t dim, Tensor index, Tensor src, *, Tensor out=None)",
8780:     "scatter_add(Tensor input, Dimname dim, Tensor index, Tensor src)",
8781:   }, /*traceable=*/true);
8782: 
8783:   ParsedArgs<5> parsed_args;
8784:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8785:   if(_r.has_torch_function()) {
8786:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8787:   }
8788:   switch (_r.idx) {
8789:     case 0: {
8790:       if (_r.isNone(4)) {
8791:         // aten::scatter_add(Tensor self, int dim, Tensor index, Tensor src) -> Tensor
8792: 
8793:         auto dispatch_scatter_add = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
8794:           pybind11::gil_scoped_release no_gil;
8795:           return self.scatter_add(dim, index, src);
8796:         };
8797:         return wrap(dispatch_scatter_add(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3)));
8798:       } else {
8799:         // aten::scatter_add.out(Tensor self, int dim, Tensor index, Tensor src, *, Tensor(a!) out) -> Tensor(a!)
8800: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_index_reduce`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_index_reduce`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8801-8880

```cpp
8801:         auto dispatch_scatter_add_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
8802:           pybind11::gil_scoped_release no_gil;
8803:           return at::scatter_add_out(out, self, dim, index, src);
8804:         };
8805:         return wrap(dispatch_scatter_add_out(_r.tensor(4), _r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3)));
8806:       }
8807:     }
8808:     case 1: {
8809:       // aten::scatter_add.dimname(Tensor self, Dimname dim, Tensor index, Tensor src) -> Tensor
8810: 
8811:       auto dispatch_scatter_add = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
8812:         pybind11::gil_scoped_release no_gil;
8813:         return self.scatter_add(dim, index, src);
8814:       };
8815:       return wrap(dispatch_scatter_add(_r.tensor(0), _r.dimname(1), _r.tensor(2), _r.tensor(3)));
8816:     }
8817:   }
8818:   Py_RETURN_NONE;
8819:   END_HANDLE_TH_ERRORS
8820: }
8821: 
8822: // scatter_reduce
8823: static PyObject * THPVariable_scatter_reduce(PyObject* self_, PyObject* args, PyObject* kwargs)
8824: {
8825:   HANDLE_TH_ERRORS
8826:   static PythonArgParser parser({
8827:     "scatter_reduce(Tensor input, int64_t dim, Tensor index, Tensor src, c10::string_view reduce, *, bool include_self=True, Tensor out=None)",
8828:   }, /*traceable=*/true);
8829: 
8830:   ParsedArgs<7> parsed_args;
8831:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8832:   if(_r.has_torch_function()) {
8833:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8834:   }
8835:   if (_r.isNone(6)) {
8836:     // aten::scatter_reduce.two(Tensor self, int dim, Tensor index, Tensor src, str reduce, *, bool include_self=True) -> Tensor
8837: 
8838:     auto dispatch_scatter_reduce = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src, c10::string_view reduce, bool include_self) -> at::Tensor {
8839:       pybind11::gil_scoped_release no_gil;
8840:       return self.scatter_reduce(dim, index, src, reduce, include_self);
8841:     };
8842:     return wrap(dispatch_scatter_reduce(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3), _r.stringView(4), _r.toBool(5)));
8843:   } else {
8844:     // aten::scatter_reduce.two_out(Tensor self, int dim, Tensor index, Tensor src, str reduce, *, bool include_self=True, Tensor(a!) out) -> Tensor(a!)
8845: 
8846:     auto dispatch_scatter_reduce_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src, c10::string_view reduce, bool include_self) -> at::Tensor {
8847:       pybind11::gil_scoped_release no_gil;
8848:       return at::scatter_reduce_out(out, self, dim, index, src, reduce, include_self);
8849:     };
8850:     return wrap(dispatch_scatter_reduce_out(_r.tensor(6), _r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.tensor(3), _r.stringView(4), _r.toBool(5)));
8851:   }
8852:   Py_RETURN_NONE;
8853:   END_HANDLE_TH_ERRORS
8854: }
8855: 
8856: \
8857: // bitwise_and
8858: static PyObject * THPVariable_bitwise_and(PyObject* self_, PyObject* args, PyObject* kwargs)
8859: {
8860:   HANDLE_TH_ERRORS
8861:   static PythonArgParser parser({
8862:     "bitwise_and(Tensor input, Tensor other, *, Tensor out=None)",
8863:     "bitwise_and(Scalar self, Tensor other)",
8864:     "bitwise_and(Tensor input, Scalar other, *, Tensor out=None)",
8865:   }, /*traceable=*/true);
8866: 
8867:   ParsedArgs<3> parsed_args;
8868:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8869:   if(_r.has_torch_function()) {
8870:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8871:   }
8872:   switch (_r.idx) {
8873:     case 0: {
8874:       if (_r.isNone(2)) {
8875:         // aten::bitwise_and.Tensor(Tensor self, Tensor other) -> Tensor
8876: 
8877:         auto dispatch_bitwise_and = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8878:           pybind11::gil_scoped_release no_gil;
8879:           return self.bitwise_and(other);
8880:         };
```

- EN: The main execution path in this span is carried by `scatter_add_out`, `wrap`, `THPVariable_scatter_reduce`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `scatter_add_out`, `wrap`, `THPVariable_scatter_reduce` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8881-8960

```cpp
8881:         return wrap(dispatch_bitwise_and(_r.tensor(0), _r.tensor(1)));
8882:       } else {
8883:         // aten::bitwise_and.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
8884: 
8885:         auto dispatch_bitwise_and_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8886:           pybind11::gil_scoped_release no_gil;
8887:           return at::bitwise_and_out(out, self, other);
8888:         };
8889:         return wrap(dispatch_bitwise_and_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8890:       }
8891:     }
8892:     case 1: {
8893:       // aten::bitwise_and.Scalar_Tensor(Scalar self, Tensor other) -> Tensor
8894: 
8895:       auto dispatch_bitwise_and = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
8896:         pybind11::gil_scoped_release no_gil;
8897:         return at::bitwise_and(self, other);
8898:       };
8899:       return wrap(dispatch_bitwise_and(_r.scalar(0), _r.tensor(1)));
8900:     }
8901:     case 2: {
8902:       if (_r.isNone(2)) {
8903:         // aten::bitwise_and.Scalar(Tensor self, Scalar other) -> Tensor
8904: 
8905:         auto dispatch_bitwise_and = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8906:           pybind11::gil_scoped_release no_gil;
8907:           return self.bitwise_and(other);
8908:         };
8909:         return wrap(dispatch_bitwise_and(_r.tensor(0), _r.scalar(1)));
8910:       } else {
8911:         // aten::bitwise_and.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
8912: 
8913:         auto dispatch_bitwise_and_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8914:           pybind11::gil_scoped_release no_gil;
8915:           return at::bitwise_and_out(out, self, other);
8916:         };
8917:         return wrap(dispatch_bitwise_and_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
8918:       }
8919:     }
8920:   }
8921:   Py_RETURN_NONE;
8922:   END_HANDLE_TH_ERRORS
8923: }
8924: 
8925: \
8926: // bitwise_xor
8927: static PyObject * THPVariable_bitwise_xor(PyObject* self_, PyObject* args, PyObject* kwargs)
8928: {
8929:   HANDLE_TH_ERRORS
8930:   static PythonArgParser parser({
8931:     "bitwise_xor(Tensor input, Tensor other, *, Tensor out=None)",
8932:     "bitwise_xor(Scalar self, Tensor other)",
8933:     "bitwise_xor(Tensor input, Scalar other, *, Tensor out=None)",
8934:   }, /*traceable=*/true);
8935: 
8936:   ParsedArgs<3> parsed_args;
8937:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8938:   if(_r.has_torch_function()) {
8939:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8940:   }
8941:   switch (_r.idx) {
8942:     case 0: {
8943:       if (_r.isNone(2)) {
8944:         // aten::bitwise_xor.Tensor(Tensor self, Tensor other) -> Tensor
8945: 
8946:         auto dispatch_bitwise_xor = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8947:           pybind11::gil_scoped_release no_gil;
8948:           return self.bitwise_xor(other);
8949:         };
8950:         return wrap(dispatch_bitwise_xor(_r.tensor(0), _r.tensor(1)));
8951:       } else {
8952:         // aten::bitwise_xor.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
8953: 
8954:         auto dispatch_bitwise_xor_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8955:           pybind11::gil_scoped_release no_gil;
8956:           return at::bitwise_xor_out(out, self, other);
8957:         };
8958:         return wrap(dispatch_bitwise_xor_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8959:       }
8960:     }
```

- EN: The main execution path in this span is carried by `wrap`, `bitwise_and_out`, `bitwise_and`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `bitwise_and_out`, `bitwise_and` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8961-9040

```cpp
8961:     case 1: {
8962:       // aten::bitwise_xor.Scalar_Tensor(Scalar self, Tensor other) -> Tensor
8963: 
8964:       auto dispatch_bitwise_xor = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
8965:         pybind11::gil_scoped_release no_gil;
8966:         return at::bitwise_xor(self, other);
8967:       };
8968:       return wrap(dispatch_bitwise_xor(_r.scalar(0), _r.tensor(1)));
8969:     }
8970:     case 2: {
8971:       if (_r.isNone(2)) {
8972:         // aten::bitwise_xor.Scalar(Tensor self, Scalar other) -> Tensor
8973: 
8974:         auto dispatch_bitwise_xor = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8975:           pybind11::gil_scoped_release no_gil;
8976:           return self.bitwise_xor(other);
8977:         };
8978:         return wrap(dispatch_bitwise_xor(_r.tensor(0), _r.scalar(1)));
8979:       } else {
8980:         // aten::bitwise_xor.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
8981: 
8982:         auto dispatch_bitwise_xor_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8983:           pybind11::gil_scoped_release no_gil;
8984:           return at::bitwise_xor_out(out, self, other);
8985:         };
8986:         return wrap(dispatch_bitwise_xor_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
8987:       }
8988:     }
8989:   }
8990:   Py_RETURN_NONE;
8991:   END_HANDLE_TH_ERRORS
8992: }
8993: 
8994: \
8995: // bitwise_left_shift
8996: static PyObject * THPVariable_bitwise_left_shift(PyObject* self_, PyObject* args, PyObject* kwargs)
8997: {
8998:   HANDLE_TH_ERRORS
8999:   static PythonArgParser parser({
9000:     "bitwise_left_shift(Tensor input, Tensor other, *, Tensor out=None)",
9001:     "bitwise_left_shift(Scalar self, Tensor other)",
9002:     "bitwise_left_shift(Tensor input, Scalar other, *, Tensor out=None)",
9003:   }, /*traceable=*/true);
9004: 
9005:   ParsedArgs<3> parsed_args;
9006:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9007:   if(_r.has_torch_function()) {
9008:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9009:   }
9010:   switch (_r.idx) {
9011:     case 0: {
9012:       if (_r.isNone(2)) {
9013:         // aten::bitwise_left_shift.Tensor(Tensor self, Tensor other) -> Tensor
9014: 
9015:         auto dispatch_bitwise_left_shift = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9016:           pybind11::gil_scoped_release no_gil;
9017:           return self.bitwise_left_shift(other);
9018:         };
9019:         return wrap(dispatch_bitwise_left_shift(_r.tensor(0), _r.tensor(1)));
9020:       } else {
9021:         // aten::bitwise_left_shift.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9022: 
9023:         auto dispatch_bitwise_left_shift_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9024:           pybind11::gil_scoped_release no_gil;
9025:           return at::bitwise_left_shift_out(out, self, other);
9026:         };
9027:         return wrap(dispatch_bitwise_left_shift_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9028:       }
9029:     }
9030:     case 1: {
9031:       // aten::bitwise_left_shift.Scalar_Tensor(Scalar self, Tensor other) -> Tensor
9032: 
9033:       auto dispatch_bitwise_left_shift = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
9034:         pybind11::gil_scoped_release no_gil;
9035:         return at::bitwise_left_shift(self, other);
9036:       };
9037:       return wrap(dispatch_bitwise_left_shift(_r.scalar(0), _r.tensor(1)));
9038:     }
9039:     case 2: {
9040:       if (_r.isNone(2)) {
```

- EN: The main execution path in this span is carried by `bitwise_xor`, `wrap`, `bitwise_xor_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `bitwise_xor`, `wrap`, `bitwise_xor_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9041-9120

```cpp
9041:         // aten::bitwise_left_shift.Tensor_Scalar(Tensor self, Scalar other) -> Tensor
9042: 
9043:         auto dispatch_bitwise_left_shift = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9044:           pybind11::gil_scoped_release no_gil;
9045:           return self.bitwise_left_shift(other);
9046:         };
9047:         return wrap(dispatch_bitwise_left_shift(_r.tensor(0), _r.scalar(1)));
9048:       } else {
9049:         // aten::bitwise_left_shift.Tensor_Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
9050: 
9051:         auto dispatch_bitwise_left_shift_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9052:           pybind11::gil_scoped_release no_gil;
9053:           return at::bitwise_left_shift_out(out, self, other);
9054:         };
9055:         return wrap(dispatch_bitwise_left_shift_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
9056:       }
9057:     }
9058:   }
9059:   Py_RETURN_NONE;
9060:   END_HANDLE_TH_ERRORS
9061: }
9062: 
9063: \
9064: // __rshift__
9065: static PyObject * THPVariable___rshift__(PyObject* self_, PyObject* args, PyObject* kwargs)
9066: {
9067:   HANDLE_TH_ERRORS
9068:   static PythonArgParser parser({
9069:     "__rshift__(Tensor input, Tensor other)",
9070:     "__rshift__(Tensor input, Scalar other)",
9071:   }, /*traceable=*/true);
9072: 
9073:   ParsedArgs<2> parsed_args;
9074:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9075:   if(_r.has_torch_function()) {
9076:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9077:   }
9078:   switch (_r.idx) {
9079:     case 0: {
9080:       // aten::__rshift__.Tensor(Tensor self, Tensor other) -> Tensor
9081: 
9082:       auto dispatch___rshift__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9083:         pybind11::gil_scoped_release no_gil;
9084:         return self.__rshift__(other);
9085:       };
9086:       return wrap(dispatch___rshift__(_r.tensor(0), _r.tensor(1)));
9087:     }
9088:     case 1: {
9089:       // aten::__rshift__.Scalar(Tensor self, Scalar other) -> Tensor
9090: 
9091:       auto dispatch___rshift__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9092:         pybind11::gil_scoped_release no_gil;
9093:         return self.__rshift__(other);
9094:       };
9095:       return wrap(dispatch___rshift__(_r.tensor(0), _r.scalar(1)));
9096:     }
9097:   }
9098:   Py_RETURN_NONE;
9099:   END_HANDLE_TH_ERRORS
9100: }
9101: 
9102: \
9103: // addbmm
9104: static PyObject * THPVariable_addbmm(PyObject* self_, PyObject* args, PyObject* kwargs)
9105: {
9106:   HANDLE_TH_ERRORS
9107:   static PythonArgParser parser({
9108:     "addbmm(Scalar beta, Tensor input, Scalar alpha, Tensor batch1, Tensor batch2, *, Tensor out=None)|deprecated",
9109:     "addbmm(Scalar beta, Tensor input, Tensor batch1, Tensor batch2, *, Tensor out=None)|deprecated",
9110:     "addbmm(Tensor input, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1, Tensor out=None)",
9111:   }, /*traceable=*/true);
9112: 
9113:   ParsedArgs<6> parsed_args;
9114:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9115:   if(_r.has_torch_function()) {
9116:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9117:   }
9118:   switch (_r.idx) {
9119:     case 0: {
9120:       if (_r.isNone(5)) {
```

- EN: The main execution path in this span is carried by `wrap`, `bitwise_left_shift_out`, `THPVariable___rshift__`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `bitwise_left_shift_out`, `THPVariable___rshift__` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9121-9200

```cpp
9121:         // [deprecated] aten::addbmm(Scalar beta, Tensor self, Scalar alpha, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
9122: 
9123:         auto dispatch_addbmm = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
9124:           pybind11::gil_scoped_release no_gil;
9125:           return self.addbmm(batch1, batch2, beta, alpha);
9126:         };
9127:         return wrap(dispatch_addbmm(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
9128:       } else {
9129:         // [deprecated] aten::addbmm(Scalar beta, Tensor self, Scalar alpha, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
9130: 
9131:         auto dispatch_addbmm_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
9132:           pybind11::gil_scoped_release no_gil;
9133:           return at::addbmm_out(out, self, batch1, batch2, beta, alpha);
9134:         };
9135:         return wrap(dispatch_addbmm_out(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
9136:       }
9137:     }
9138:     case 1: {
9139:       if (_r.isNone(4)) {
9140:         // [deprecated] aten::addbmm(Scalar beta, Tensor self, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
9141: 
9142:         auto dispatch_addbmm = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
9143:           pybind11::gil_scoped_release no_gil;
9144:           return self.addbmm(batch1, batch2, beta, 1);
9145:         };
9146:         return wrap(dispatch_addbmm(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
9147:       } else {
9148:         // [deprecated] aten::addbmm(Scalar beta, Tensor self, Tensor batch1, Tensor batch2, *, Tensor(a!) out) -> Tensor(a!)
9149: 
9150:         auto dispatch_addbmm_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
9151:           pybind11::gil_scoped_release no_gil;
9152:           return at::addbmm_out(out, self, batch1, batch2, beta, 1);
9153:         };
9154:         return wrap(dispatch_addbmm_out(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
9155:       }
9156:     }
9157:     case 2: {
9158:       if (_r.isNone(5)) {
9159:         // aten::addbmm(Tensor self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
9160: 
9161:         auto dispatch_addbmm = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
9162:           pybind11::gil_scoped_release no_gil;
9163:           return self.addbmm(batch1, batch2, beta, alpha);
9164:         };
9165:         return wrap(dispatch_addbmm(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
9166:       } else {
9167:         // aten::addbmm.out(Tensor self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
9168: 
9169:         auto dispatch_addbmm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
9170:           pybind11::gil_scoped_release no_gil;
9171:           return at::addbmm_out(out, self, batch1, batch2, beta, alpha);
9172:         };
9173:         return wrap(dispatch_addbmm_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
9174:       }
9175:     }
9176:   }
9177:   Py_RETURN_NONE;
9178:   END_HANDLE_TH_ERRORS
9179: }
9180: 
9181: // diag
9182: static PyObject * THPVariable_diag(PyObject* self_, PyObject* args, PyObject* kwargs)
9183: {
9184:   HANDLE_TH_ERRORS
9185:   static PythonArgParser parser({
9186:     "diag(Tensor input, int64_t diagonal=0, *, Tensor out=None)",
9187:   }, /*traceable=*/true);
9188: 
9189:   ParsedArgs<3> parsed_args;
9190:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9191:   if(_r.has_torch_function()) {
9192:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9193:   }
9194:   if (_r.isNone(2)) {
9195:     // aten::diag(Tensor self, int diagonal=0) -> Tensor
9196: 
9197:     auto dispatch_diag = [](const at::Tensor & self, int64_t diagonal) -> at::Tensor {
9198:       pybind11::gil_scoped_release no_gil;
9199:       return self.diag(diagonal);
9200:     };
```

- EN: The main execution path in this span is carried by `addbmm`, `wrap`, `addbmm_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `addbmm`, `wrap`, `addbmm_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9201-9280

```cpp
9201:     return wrap(dispatch_diag(_r.tensor(0), _r.toInt64(1)));
9202:   } else {
9203:     // aten::diag.out(Tensor self, int diagonal=0, *, Tensor(a!) out) -> Tensor(a!)
9204: 
9205:     auto dispatch_diag_out = [](at::Tensor out, const at::Tensor & self, int64_t diagonal) -> at::Tensor {
9206:       pybind11::gil_scoped_release no_gil;
9207:       return at::diag_out(out, self, diagonal);
9208:     };
9209:     return wrap(dispatch_diag_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
9210:   }
9211:   Py_RETURN_NONE;
9212:   END_HANDLE_TH_ERRORS
9213: }
9214: 
9215: // cross
9216: static PyObject * THPVariable_cross(PyObject* self_, PyObject* args, PyObject* kwargs)
9217: {
9218:   HANDLE_TH_ERRORS
9219:   static PythonArgParser parser({
9220:     "cross(Tensor input, Tensor other, int64_t? dim=None, *, Tensor out=None)",
9221:   }, /*traceable=*/true);
9222: 
9223:   ParsedArgs<4> parsed_args;
9224:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9225:   if(_r.has_torch_function()) {
9226:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9227:   }
9228:   if (_r.isNone(3)) {
9229:     // aten::cross(Tensor self, Tensor other, int? dim=None) -> Tensor
9230: 
9231:     auto dispatch_cross = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<int64_t> dim) -> at::Tensor {
9232:       pybind11::gil_scoped_release no_gil;
9233:       return self.cross(other, dim);
9234:     };
9235:     return wrap(dispatch_cross(_r.tensor(0), _r.tensor(1), _r.toInt64Optional(2)));
9236:   } else {
9237:     // aten::cross.out(Tensor self, Tensor other, int? dim=None, *, Tensor(a!) out) -> Tensor(a!)
9238: 
9239:     auto dispatch_cross_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, ::std::optional<int64_t> dim) -> at::Tensor {
9240:       pybind11::gil_scoped_release no_gil;
9241:       return at::cross_out(out, self, other, dim);
9242:     };
9243:     return wrap(dispatch_cross_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.toInt64Optional(2)));
9244:   }
9245:   Py_RETURN_NONE;
9246:   END_HANDLE_TH_ERRORS
9247: }
9248: 
9249: // triu_indices
9250: static PyObject * THPVariable_triu_indices(PyObject* self_, PyObject* args, PyObject* kwargs)
9251: {
9252:   HANDLE_TH_ERRORS
9253:   static PythonArgParser parser({
9254:     "triu_indices(int64_t row, int64_t col, int64_t offset=0, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
9255:   }, /*traceable=*/true);
9256: 
9257:   ParsedArgs<8> parsed_args;
9258:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9259:   if(_r.has_torch_function()) {
9260:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9261:   }
9262:   // aten::triu_indices(int row, int col, int offset=0, *, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
9263:   const auto options = TensorOptions()
9264:       .dtype(_r.scalartypeWithDefault(3, at::kLong))
9265:       .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
9266:       .layout(_r.layoutOptional(4))
9267:       .requires_grad(_r.toBool(7))
9268:       .pinned_memory(_r.toBool(6));
9269:   torch::utils::maybe_initialize_device(options);
9270: 
9271:   auto dispatch_triu_indices = [](int64_t row, int64_t col, int64_t offset, at::TensorOptions options) -> at::Tensor {
9272:     pybind11::gil_scoped_release no_gil;
9273:     return torch::triu_indices(row, col, offset, options);
9274:   };
9275:   return wrap(dispatch_triu_indices(_r.toInt64(0), _r.toInt64(1), _r.toInt64(2), options));
9276:   Py_RETURN_NONE;
9277:   END_HANDLE_TH_ERRORS
9278: }
9279: 
9280: // trace
```

- EN: The main execution path in this span is carried by `wrap`, `diag_out`, `THPVariable_cross`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `diag_out`, `THPVariable_cross` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9281-9360

```cpp
9281: static PyObject * THPVariable_trace(PyObject* self_, PyObject* args, PyObject* kwargs)
9282: {
9283:   HANDLE_TH_ERRORS
9284:   static PythonArgParser parser({
9285:     "trace(Tensor input)",
9286:   }, /*traceable=*/true);
9287: 
9288:   ParsedArgs<1> parsed_args;
9289:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9290:   if(_r.has_torch_function()) {
9291:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9292:   }
9293:   // aten::trace(Tensor self) -> Tensor
9294: 
9295:   auto dispatch_trace = [](const at::Tensor & self) -> at::Tensor {
9296:     pybind11::gil_scoped_release no_gil;
9297:     return self.trace();
9298:   };
9299:   return wrap(dispatch_trace(_r.tensor(0)));
9300:   Py_RETURN_NONE;
9301:   END_HANDLE_TH_ERRORS
9302: }
9303: 
9304: \
9305: // ne
9306: static PyObject * THPVariable_ne(PyObject* self_, PyObject* args, PyObject* kwargs)
9307: {
9308:   HANDLE_TH_ERRORS
9309:   static PythonArgParser parser({
9310:     "ne(Tensor input, Tensor other, *, Tensor out=None)",
9311:     "ne(Tensor input, Scalar other, *, Tensor out=None)",
9312:   }, /*traceable=*/true);
9313: 
9314:   ParsedArgs<3> parsed_args;
9315:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9316:   if(_r.has_torch_function()) {
9317:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9318:   }
9319:   switch (_r.idx) {
9320:     case 0: {
9321:       if (_r.isNone(2)) {
9322:         // aten::ne.Tensor(Tensor self, Tensor other) -> Tensor
9323: 
9324:         auto dispatch_ne = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9325:           pybind11::gil_scoped_release no_gil;
9326:           return self.ne(other);
9327:         };
9328:         return wrap(dispatch_ne(_r.tensor(0), _r.tensor(1)));
9329:       } else {
9330:         // aten::ne.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9331: 
9332:         auto dispatch_ne_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9333:           pybind11::gil_scoped_release no_gil;
9334:           return at::ne_out(out, self, other);
9335:         };
9336:         return wrap(dispatch_ne_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9337:       }
9338:     }
9339:     case 1: {
9340:       if (_r.isNone(2)) {
9341:         // aten::ne.Scalar(Tensor self, Scalar other) -> Tensor
9342: 
9343:         auto dispatch_ne = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9344:           pybind11::gil_scoped_release no_gil;
9345:           return self.ne(other);
9346:         };
9347:         return wrap(dispatch_ne(_r.tensor(0), _r.scalar(1)));
9348:       } else {
9349:         // aten::ne.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
9350: 
9351:         auto dispatch_ne_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9352:           pybind11::gil_scoped_release no_gil;
9353:           return at::ne_out(out, self, other);
9354:         };
9355:         return wrap(dispatch_ne_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
9356:       }
9357:     }
9358:   }
9359:   Py_RETURN_NONE;
9360:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_trace`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_trace`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9361-9440

```cpp
9361: }
9362: 
9363: \
9364: // eq
9365: static PyObject * THPVariable_eq(PyObject* self_, PyObject* args, PyObject* kwargs)
9366: {
9367:   HANDLE_TH_ERRORS
9368:   static PythonArgParser parser({
9369:     "eq(Tensor input, Tensor other, *, Tensor out=None)",
9370:     "eq(Tensor input, Scalar other, *, Tensor out=None)",
9371:   }, /*traceable=*/true);
9372: 
9373:   ParsedArgs<3> parsed_args;
9374:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9375:   if(_r.has_torch_function()) {
9376:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9377:   }
9378:   switch (_r.idx) {
9379:     case 0: {
9380:       if (_r.isNone(2)) {
9381:         // aten::eq.Tensor(Tensor self, Tensor other) -> Tensor
9382: 
9383:         auto dispatch_eq = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9384:           pybind11::gil_scoped_release no_gil;
9385:           return self.eq(other);
9386:         };
9387:         return wrap(dispatch_eq(_r.tensor(0), _r.tensor(1)));
9388:       } else {
9389:         // aten::eq.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9390: 
9391:         auto dispatch_eq_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9392:           pybind11::gil_scoped_release no_gil;
9393:           return at::eq_out(out, self, other);
9394:         };
9395:         return wrap(dispatch_eq_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9396:       }
9397:     }
9398:     case 1: {
9399:       if (_r.isNone(2)) {
9400:         // aten::eq.Scalar(Tensor self, Scalar other) -> Tensor
9401: 
9402:         auto dispatch_eq = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9403:           pybind11::gil_scoped_release no_gil;
9404:           return self.eq(other);
9405:         };
9406:         return wrap(dispatch_eq(_r.tensor(0), _r.scalar(1)));
9407:       } else {
9408:         // aten::eq.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
9409: 
9410:         auto dispatch_eq_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9411:           pybind11::gil_scoped_release no_gil;
9412:           return at::eq_out(out, self, other);
9413:         };
9414:         return wrap(dispatch_eq_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
9415:       }
9416:     }
9417:   }
9418:   Py_RETURN_NONE;
9419:   END_HANDLE_TH_ERRORS
9420: }
9421: 
9422: \
9423: // greater
9424: static PyObject * THPVariable_greater(PyObject* self_, PyObject* args, PyObject* kwargs)
9425: {
9426:   HANDLE_TH_ERRORS
9427:   static PythonArgParser parser({
9428:     "greater(Tensor input, Tensor other, *, Tensor out=None)",
9429:     "greater(Tensor input, Scalar other, *, Tensor out=None)",
9430:   }, /*traceable=*/true);
9431: 
9432:   ParsedArgs<3> parsed_args;
9433:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9434:   if(_r.has_torch_function()) {
9435:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9436:   }
9437:   switch (_r.idx) {
9438:     case 0: {
9439:       if (_r.isNone(2)) {
9440:         // aten::greater.Tensor(Tensor self, Tensor other) -> Tensor
```

- EN: The main execution path in this span is carried by `THPVariable_eq`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_eq`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9441-9520

```cpp
9441: 
9442:         auto dispatch_greater = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9443:           pybind11::gil_scoped_release no_gil;
9444:           return self.greater(other);
9445:         };
9446:         return wrap(dispatch_greater(_r.tensor(0), _r.tensor(1)));
9447:       } else {
9448:         // aten::greater.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9449: 
9450:         auto dispatch_greater_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9451:           pybind11::gil_scoped_release no_gil;
9452:           return at::greater_out(out, self, other);
9453:         };
9454:         return wrap(dispatch_greater_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9455:       }
9456:     }
9457:     case 1: {
9458:       if (_r.isNone(2)) {
9459:         // aten::greater.Scalar(Tensor self, Scalar other) -> Tensor
9460: 
9461:         auto dispatch_greater = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9462:           pybind11::gil_scoped_release no_gil;
9463:           return self.greater(other);
9464:         };
9465:         return wrap(dispatch_greater(_r.tensor(0), _r.scalar(1)));
9466:       } else {
9467:         // aten::greater.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
9468: 
9469:         auto dispatch_greater_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9470:           pybind11::gil_scoped_release no_gil;
9471:           return at::greater_out(out, self, other);
9472:         };
9473:         return wrap(dispatch_greater_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
9474:       }
9475:     }
9476:   }
9477:   Py_RETURN_NONE;
9478:   END_HANDLE_TH_ERRORS
9479: }
9480: 
9481: // take
9482: static PyObject * THPVariable_take(PyObject* self_, PyObject* args, PyObject* kwargs)
9483: {
9484:   HANDLE_TH_ERRORS
9485:   static PythonArgParser parser({
9486:     "take(Tensor input, Tensor index, *, Tensor out=None)",
9487:   }, /*traceable=*/true);
9488: 
9489:   ParsedArgs<3> parsed_args;
9490:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9491:   if(_r.has_torch_function()) {
9492:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9493:   }
9494:   if (_r.isNone(2)) {
9495:     // aten::take(Tensor self, Tensor index) -> Tensor
9496: 
9497:     auto dispatch_take = [](const at::Tensor & self, const at::Tensor & index) -> at::Tensor {
9498:       pybind11::gil_scoped_release no_gil;
9499:       return self.take(index);
9500:     };
9501:     return wrap(dispatch_take(_r.tensor(0), _r.tensor(1)));
9502:   } else {
9503:     // aten::take.out(Tensor self, Tensor index, *, Tensor(a!) out) -> Tensor(a!)
9504: 
9505:     auto dispatch_take_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & index) -> at::Tensor {
9506:       pybind11::gil_scoped_release no_gil;
9507:       return at::take_out(out, self, index);
9508:     };
9509:     return wrap(dispatch_take_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9510:   }
9511:   Py_RETURN_NONE;
9512:   END_HANDLE_TH_ERRORS
9513: }
9514: 
9515: // _linalg_check_errors
9516: static PyObject * THPVariable__linalg_check_errors(PyObject* self_, PyObject* args, PyObject* kwargs)
9517: {
9518:   HANDLE_TH_ERRORS
9519:   static PythonArgParser parser({
9520:     "_linalg_check_errors(Tensor info, c10::string_view api_name, *, bool is_matrix)",
```

- EN: The main execution path in this span is carried by `wrap`, `greater_out`, `THPVariable_take`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `greater_out`, `THPVariable_take` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9521-9600

```cpp
9521:   }, /*traceable=*/false);
9522: 
9523:   ParsedArgs<3> parsed_args;
9524:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9525:   if(_r.has_torch_function()) {
9526:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9527:   }
9528:   // aten::_linalg_check_errors(Tensor info, str api_name, *, bool is_matrix) -> ()
9529: 
9530:   auto dispatch__linalg_check_errors = [](const at::Tensor & info, c10::string_view api_name, bool is_matrix) -> void {
9531:     pybind11::gil_scoped_release no_gil;
9532:     at::_linalg_check_errors(info, api_name, is_matrix);
9533:   };
9534:   dispatch__linalg_check_errors(_r.tensor(0), _r.stringView(1), _r.toBool(2));
9535:   Py_RETURN_NONE;
9536:   Py_RETURN_NONE;
9537:   END_HANDLE_TH_ERRORS
9538: }
9539: 
9540: // svd
9541: static PyObject * THPVariable_svd(PyObject* self_, PyObject* args, PyObject* kwargs)
9542: {
9543:   HANDLE_TH_ERRORS
9544:   static PyTypeObject* NamedTuple = generated::get_svd_out_structseq();
9545:   static PyTypeObject* NamedTuple1 = generated::get_svd_structseq();
9546:   static PythonArgParser parser({
9547:     "svd(Tensor input, bool some=True, bool compute_uv=True, *, TensorList[3] out=None)",
9548:   }, /*traceable=*/true);
9549: 
9550:   ParsedArgs<4> parsed_args;
9551:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9552:   if(_r.has_torch_function()) {
9553:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9554:   }
9555:   if (_r.isNone(3)) {
9556:     // aten::svd(Tensor self, bool some=True, bool compute_uv=True) -> (Tensor U, Tensor S, Tensor V)
9557: 
9558:     auto dispatch_svd = [](const at::Tensor & self, bool some, bool compute_uv) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
9559:       pybind11::gil_scoped_release no_gil;
9560:       return self.svd(some, compute_uv);
9561:     };
9562:     return wrap(NamedTuple1, dispatch_svd(_r.tensor(0), _r.toBool(1), _r.toBool(2)));
9563:   } else {
9564:     // aten::svd.U(Tensor self, bool some=True, bool compute_uv=True, *, Tensor(a!) U, Tensor(b!) S, Tensor(c!) V) -> (Tensor(a!) U, Tensor(b!) S, Tensor(c!) V)
9565:     auto out = _r.tensorlist_n<3>(3);
9566:     auto dispatch_svd_out = [](at::Tensor & U, at::Tensor & S, at::Tensor & V, const at::Tensor & self, bool some, bool compute_uv) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
9567:       pybind11::gil_scoped_release no_gil;
9568:       return at::svd_out(U, S, V, self, some, compute_uv);
9569:     };
9570:     return wrap(NamedTuple, dispatch_svd_out(out[0], out[1], out[2], _r.tensor(0), _r.toBool(1), _r.toBool(2)));
9571:   }
9572:   Py_RETURN_NONE;
9573:   END_HANDLE_TH_ERRORS
9574: }
9575: 
9576: // cholesky
9577: static PyObject * THPVariable_cholesky(PyObject* self_, PyObject* args, PyObject* kwargs)
9578: {
9579:   HANDLE_TH_ERRORS
9580:   static PythonArgParser parser({
9581:     "cholesky(Tensor input, bool upper=False, *, Tensor out=None)",
9582:   }, /*traceable=*/true);
9583: 
9584:   ParsedArgs<3> parsed_args;
9585:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9586:   if(_r.has_torch_function()) {
9587:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9588:   }
9589:   if (_r.isNone(2)) {
9590:     // aten::cholesky(Tensor self, bool upper=False) -> Tensor
9591: 
9592:     auto dispatch_cholesky = [](const at::Tensor & self, bool upper) -> at::Tensor {
9593:       pybind11::gil_scoped_release no_gil;
9594:       return self.cholesky(upper);
9595:     };
9596:     return wrap(dispatch_cholesky(_r.tensor(0), _r.toBool(1)));
9597:   } else {
9598:     // aten::cholesky.out(Tensor self, bool upper=False, *, Tensor(a!) out) -> Tensor(a!)
9599: 
9600:     auto dispatch_cholesky_out = [](at::Tensor out, const at::Tensor & self, bool upper) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_linalg_check_errors`, `dispatch__linalg_check_errors`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_linalg_check_errors`, `dispatch__linalg_check_errors` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9601-9680

```cpp
9601:       pybind11::gil_scoped_release no_gil;
9602:       return at::cholesky_out(out, self, upper);
9603:     };
9604:     return wrap(dispatch_cholesky_out(_r.tensor(2), _r.tensor(0), _r.toBool(1)));
9605:   }
9606:   Py_RETURN_NONE;
9607:   END_HANDLE_TH_ERRORS
9608: }
9609: 
9610: // _lu_with_info
9611: static PyObject * THPVariable__lu_with_info(PyObject* self_, PyObject* args, PyObject* kwargs)
9612: {
9613:   HANDLE_TH_ERRORS
9614:   static PyTypeObject* NamedTuple = generated::get__lu_with_info_structseq();
9615:   static PythonArgParser parser({
9616:     "_lu_with_info(Tensor input, bool pivot=True, bool check_errors=True)",
9617:   }, /*traceable=*/true);
9618: 
9619:   ParsedArgs<3> parsed_args;
9620:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9621:   if(_r.has_torch_function()) {
9622:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9623:   }
9624:   // aten::_lu_with_info(Tensor self, bool pivot=True, bool check_errors=True) -> (Tensor LU, Tensor pivots, Tensor info)
9625: 
9626:   auto dispatch__lu_with_info = [](const at::Tensor & self, bool pivot, bool check_errors) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
9627:     pybind11::gil_scoped_release no_gil;
9628:     return at::_lu_with_info(self, pivot, check_errors);
9629:   };
9630:   return wrap(NamedTuple, dispatch__lu_with_info(_r.tensor(0), _r.toBool(1), _r.toBool(2)));
9631:   Py_RETURN_NONE;
9632:   END_HANDLE_TH_ERRORS
9633: }
9634: 
9635: // lu_unpack
9636: static PyObject * THPVariable_lu_unpack(PyObject* self_, PyObject* args, PyObject* kwargs)
9637: {
9638:   HANDLE_TH_ERRORS
9639:   static PyTypeObject* NamedTuple = generated::get_lu_unpack_structseq();
9640:   static PyTypeObject* NamedTuple1 = generated::get_lu_unpack_out_structseq();
9641:   static PythonArgParser parser({
9642:     "lu_unpack(Tensor LU_data, Tensor LU_pivots, bool unpack_data=True, bool unpack_pivots=True, *, TensorList[3] out=None)",
9643:   }, /*traceable=*/true);
9644: 
9645:   ParsedArgs<5> parsed_args;
9646:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9647:   if(_r.has_torch_function()) {
9648:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9649:   }
9650:   if (_r.isNone(4)) {
9651:     // aten::lu_unpack(Tensor LU_data, Tensor LU_pivots, bool unpack_data=True, bool unpack_pivots=True) -> (Tensor P, Tensor L, Tensor U)
9652: 
9653:     auto dispatch_lu_unpack = [](const at::Tensor & LU_data, const at::Tensor & LU_pivots, bool unpack_data, bool unpack_pivots) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
9654:       pybind11::gil_scoped_release no_gil;
9655:       return at::lu_unpack(LU_data, LU_pivots, unpack_data, unpack_pivots);
9656:     };
9657:     return wrap(NamedTuple, dispatch_lu_unpack(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
9658:   } else {
9659:     // aten::lu_unpack.out(Tensor LU_data, Tensor LU_pivots, bool unpack_data=True, bool unpack_pivots=True, *, Tensor(a!) P, Tensor(b!) L, Tensor(c!) U) -> (Tensor(a!) P, Tensor(b!) L, Tensor(c!) U)
9660:     auto out = _r.tensorlist_n<3>(4);
9661:     auto dispatch_lu_unpack_out = [](at::Tensor & P, at::Tensor & L, at::Tensor & U, const at::Tensor & LU_data, const at::Tensor & LU_pivots, bool unpack_data, bool unpack_pivots) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
9662:       pybind11::gil_scoped_release no_gil;
9663:       return at::lu_unpack_out(P, L, U, LU_data, LU_pivots, unpack_data, unpack_pivots);
9664:     };
9665:     return wrap(NamedTuple1, dispatch_lu_unpack_out(out[0], out[1], out[2], _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
9666:   }
9667:   Py_RETURN_NONE;
9668:   END_HANDLE_TH_ERRORS
9669: }
9670: 
9671: // multinomial
9672: static PyObject * THPVariable_multinomial(PyObject* self_, PyObject* args, PyObject* kwargs)
9673: {
9674:   HANDLE_TH_ERRORS
9675:   static PythonArgParser parser({
9676:     "multinomial(Tensor input, SymInt num_samples, bool replacement=False, *, Generator? generator=None, Tensor out=None)",
9677:   }, /*traceable=*/true);
9678: 
9679:   ParsedArgs<5> parsed_args;
9680:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `cholesky_out`, `wrap`, `THPVariable__lu_with_info`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cholesky_out`, `wrap`, `THPVariable__lu_with_info` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9681-9760

```cpp
9681:   if(_r.has_torch_function()) {
9682:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9683:   }
9684:   if (_r.isNone(4)) {
9685:     // aten::multinomial(Tensor self, SymInt num_samples, bool replacement=False, *, Generator? generator=None) -> Tensor
9686: 
9687:     auto dispatch_multinomial = [](const at::Tensor & self, c10::SymInt num_samples, bool replacement, ::std::optional<at::Generator> generator) -> at::Tensor {
9688:       pybind11::gil_scoped_release no_gil;
9689:       return self.multinomial_symint(num_samples, replacement, generator);
9690:     };
9691:     return wrap(dispatch_multinomial(_r.tensor(0), _r.toSymInt(1), _r.toBool(2), _r.generator(3)));
9692:   } else {
9693:     // aten::multinomial.out(Tensor self, SymInt num_samples, bool replacement=False, *, Generator? generator=None, Tensor(a!) out) -> Tensor(a!)
9694: 
9695:     auto dispatch_multinomial_out = [](at::Tensor out, const at::Tensor & self, c10::SymInt num_samples, bool replacement, ::std::optional<at::Generator> generator) -> at::Tensor {
9696:       pybind11::gil_scoped_release no_gil;
9697:       return at::multinomial_symint_out(out, self, num_samples, replacement, generator);
9698:     };
9699:     return wrap(dispatch_multinomial_out(_r.tensor(4), _r.tensor(0), _r.toSymInt(1), _r.toBool(2), _r.generator(3)));
9700:   }
9701:   Py_RETURN_NONE;
9702:   END_HANDLE_TH_ERRORS
9703: }
9704: 
9705: // i0
9706: static PyObject * THPVariable_i0(PyObject* self_, PyObject* args, PyObject* kwargs)
9707: {
9708:   HANDLE_TH_ERRORS
9709:   static PythonArgParser parser({
9710:     "i0(Tensor input, *, Tensor out=None)",
9711:   }, /*traceable=*/true);
9712: 
9713:   ParsedArgs<2> parsed_args;
9714:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9715:   if(_r.has_torch_function()) {
9716:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9717:   }
9718:   if (_r.isNone(1)) {
9719:     // aten::i0(Tensor self) -> Tensor
9720: 
9721:     auto dispatch_i0 = [](const at::Tensor & self) -> at::Tensor {
9722:       pybind11::gil_scoped_release no_gil;
9723:       return self.i0();
9724:     };
9725:     return wrap(dispatch_i0(_r.tensor(0)));
9726:   } else {
9727:     // aten::i0.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
9728: 
9729:     auto dispatch_i0_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
9730:       pybind11::gil_scoped_release no_gil;
9731:       return at::i0_out(out, self);
9732:     };
9733:     return wrap(dispatch_i0_out(_r.tensor(1), _r.tensor(0)));
9734:   }
9735:   Py_RETURN_NONE;
9736:   END_HANDLE_TH_ERRORS
9737: }
9738: 
9739: // i0_
9740: static PyObject * THPVariable_i0_(PyObject* self_, PyObject* args, PyObject* kwargs)
9741: {
9742:   HANDLE_TH_ERRORS
9743:   static PythonArgParser parser({
9744:     "i0_(Tensor input)",
9745:   }, /*traceable=*/true);
9746: 
9747:   ParsedArgs<1> parsed_args;
9748:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9749:   if(_r.has_torch_function()) {
9750:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9751:   }
9752:   // aten::i0_(Tensor(a!) self) -> Tensor(a!)
9753: 
9754:   auto dispatch_i0_ = [](at::Tensor self) -> at::Tensor {
9755:     pybind11::gil_scoped_release no_gil;
9756:     return self.i0_();
9757:   };
9758:   return wrap(dispatch_i0_(_r.tensor(0)));
9759:   Py_RETURN_NONE;
9760:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `multinomial`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `multinomial`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9761-9840

```cpp
9761: }
9762: 
9763: // atan2
9764: static PyObject * THPVariable_atan2(PyObject* self_, PyObject* args, PyObject* kwargs)
9765: {
9766:   HANDLE_TH_ERRORS
9767:   static PythonArgParser parser({
9768:     "atan2(Tensor input, Tensor other, *, Tensor out=None)",
9769:   }, /*traceable=*/true);
9770: 
9771:   ParsedArgs<3> parsed_args;
9772:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9773:   if(_r.has_torch_function()) {
9774:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9775:   }
9776:   if (_r.isNone(2)) {
9777:     // aten::atan2(Tensor self, Tensor other) -> Tensor
9778: 
9779:     auto dispatch_atan2 = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9780:       pybind11::gil_scoped_release no_gil;
9781:       return self.atan2(other);
9782:     };
9783:     return wrap(dispatch_atan2(_r.tensor(0), _r.tensor(1)));
9784:   } else {
9785:     // aten::atan2.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9786: 
9787:     auto dispatch_atan2_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9788:       pybind11::gil_scoped_release no_gil;
9789:       return at::atan2_out(out, self, other);
9790:     };
9791:     return wrap(dispatch_atan2_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9792:   }
9793:   Py_RETURN_NONE;
9794:   END_HANDLE_TH_ERRORS
9795: }
9796: 
9797: // hypot
9798: static PyObject * THPVariable_hypot(PyObject* self_, PyObject* args, PyObject* kwargs)
9799: {
9800:   HANDLE_TH_ERRORS
9801:   static PythonArgParser parser({
9802:     "hypot(Tensor input, Tensor other, *, Tensor out=None)",
9803:   }, /*traceable=*/true);
9804: 
9805:   ParsedArgs<3> parsed_args;
9806:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9807:   if(_r.has_torch_function()) {
9808:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9809:   }
9810:   if (_r.isNone(2)) {
9811:     // aten::hypot(Tensor self, Tensor other) -> Tensor
9812: 
9813:     auto dispatch_hypot = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9814:       pybind11::gil_scoped_release no_gil;
9815:       return self.hypot(other);
9816:     };
9817:     return wrap(dispatch_hypot(_r.tensor(0), _r.tensor(1)));
9818:   } else {
9819:     // aten::hypot.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9820: 
9821:     auto dispatch_hypot_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9822:       pybind11::gil_scoped_release no_gil;
9823:       return at::hypot_out(out, self, other);
9824:     };
9825:     return wrap(dispatch_hypot_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9826:   }
9827:   Py_RETURN_NONE;
9828:   END_HANDLE_TH_ERRORS
9829: }
9830: 
9831: // igamma
9832: static PyObject * THPVariable_igamma(PyObject* self_, PyObject* args, PyObject* kwargs)
9833: {
9834:   HANDLE_TH_ERRORS
9835:   static PythonArgParser parser({
9836:     "igamma(Tensor input, Tensor other, *, Tensor out=None)",
9837:   }, /*traceable=*/true);
9838: 
9839:   ParsedArgs<3> parsed_args;
9840:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `THPVariable_atan2`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_atan2`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9841-9920

```cpp
9841:   if(_r.has_torch_function()) {
9842:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9843:   }
9844:   if (_r.isNone(2)) {
9845:     // aten::igamma(Tensor self, Tensor other) -> Tensor
9846: 
9847:     auto dispatch_igamma = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9848:       pybind11::gil_scoped_release no_gil;
9849:       return self.igamma(other);
9850:     };
9851:     return wrap(dispatch_igamma(_r.tensor(0), _r.tensor(1)));
9852:   } else {
9853:     // aten::igamma.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9854: 
9855:     auto dispatch_igamma_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9856:       pybind11::gil_scoped_release no_gil;
9857:       return at::igamma_out(out, self, other);
9858:     };
9859:     return wrap(dispatch_igamma_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9860:   }
9861:   Py_RETURN_NONE;
9862:   END_HANDLE_TH_ERRORS
9863: }
9864: 
9865: // nextafter
9866: static PyObject * THPVariable_nextafter(PyObject* self_, PyObject* args, PyObject* kwargs)
9867: {
9868:   HANDLE_TH_ERRORS
9869:   static PythonArgParser parser({
9870:     "nextafter(Tensor input, Tensor other, *, Tensor out=None)",
9871:   }, /*traceable=*/true);
9872: 
9873:   ParsedArgs<3> parsed_args;
9874:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9875:   if(_r.has_torch_function()) {
9876:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9877:   }
9878:   if (_r.isNone(2)) {
9879:     // aten::nextafter(Tensor self, Tensor other) -> Tensor
9880: 
9881:     auto dispatch_nextafter = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9882:       pybind11::gil_scoped_release no_gil;
9883:       return self.nextafter(other);
9884:     };
9885:     return wrap(dispatch_nextafter(_r.tensor(0), _r.tensor(1)));
9886:   } else {
9887:     // aten::nextafter.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9888: 
9889:     auto dispatch_nextafter_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9890:       pybind11::gil_scoped_release no_gil;
9891:       return at::nextafter_out(out, self, other);
9892:     };
9893:     return wrap(dispatch_nextafter_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9894:   }
9895:   Py_RETURN_NONE;
9896:   END_HANDLE_TH_ERRORS
9897: }
9898: 
9899: // fmin
9900: static PyObject * THPVariable_fmin(PyObject* self_, PyObject* args, PyObject* kwargs)
9901: {
9902:   HANDLE_TH_ERRORS
9903:   static PythonArgParser parser({
9904:     "fmin(Tensor input, Tensor other, *, Tensor out=None)",
9905:   }, /*traceable=*/true);
9906: 
9907:   ParsedArgs<3> parsed_args;
9908:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9909:   if(_r.has_torch_function()) {
9910:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9911:   }
9912:   if (_r.isNone(2)) {
9913:     // aten::fmin(Tensor self, Tensor other) -> Tensor
9914: 
9915:     auto dispatch_fmin = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9916:       pybind11::gil_scoped_release no_gil;
9917:       return self.fmin(other);
9918:     };
9919:     return wrap(dispatch_fmin(_r.tensor(0), _r.tensor(1)));
9920:   } else {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `igamma`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `igamma`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9921-10000

```cpp
 9921:     // aten::fmin.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
 9922: 
 9923:     auto dispatch_fmin_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
 9924:       pybind11::gil_scoped_release no_gil;
 9925:       return at::fmin_out(out, self, other);
 9926:     };
 9927:     return wrap(dispatch_fmin_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
 9928:   }
 9929:   Py_RETURN_NONE;
 9930:   END_HANDLE_TH_ERRORS
 9931: }
 9932: 
 9933: \
 9934: // sort
 9935: static PyObject * THPVariable_sort(PyObject* self_, PyObject* args, PyObject* kwargs)
 9936: {
 9937:   HANDLE_TH_ERRORS
 9938:   static PyTypeObject* NamedTuple = generated::get_sort_out_structseq();
 9939:   static PyTypeObject* NamedTuple1 = generated::get_sort_structseq();
 9940:   static PythonArgParser parser({
 9941:     "sort(Tensor input, *, bool? stable, int64_t dim=-1, bool descending=False, TensorList[2] out=None)",
 9942:     "sort(Tensor input, int64_t dim=-1, bool descending=False, *, TensorList[2] out=None)",
 9943:     "sort(Tensor input, *, bool? stable, Dimname dim, bool descending=False, TensorList[2] out=None)",
 9944:     "sort(Tensor input, Dimname dim, bool descending=False, *, TensorList[2] out=None)",
 9945:   }, /*traceable=*/true);
 9946: 
 9947:   ParsedArgs<5> parsed_args;
 9948:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
 9949:   if(_r.has_torch_function()) {
 9950:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
 9951:   }
 9952:   switch (_r.idx) {
 9953:     case 0: {
 9954:       if (_r.isNone(4)) {
 9955:         // aten::sort.stable(Tensor self, *, bool? stable, int dim=-1, bool descending=False) -> (Tensor values, Tensor indices)
 9956: 
 9957:         auto dispatch_sort = [](const at::Tensor & self, ::std::optional<bool> stable, int64_t dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
 9958:           pybind11::gil_scoped_release no_gil;
 9959:           return self.sort(stable, dim, descending);
 9960:         };
 9961:         return wrap(NamedTuple1, dispatch_sort(_r.tensor(0), _r.toBoolOptional(1), _r.toInt64(2), _r.toBool(3)));
 9962:       } else {
 9963:         // aten::sort.values_stable(Tensor self, *, bool? stable, int dim=-1, bool descending=False, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
 9964:         auto out = _r.tensorlist_n<2>(4);
 9965:         auto dispatch_sort_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, ::std::optional<bool> stable, int64_t dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
 9966:           pybind11::gil_scoped_release no_gil;
 9967:           return at::sort_out(values, indices, self, stable, dim, descending);
 9968:         };
 9969:         return wrap(NamedTuple, dispatch_sort_out(out[0], out[1], _r.tensor(0), _r.toBoolOptional(1), _r.toInt64(2), _r.toBool(3)));
 9970:       }
 9971:     }
 9972:     case 1: {
 9973:       if (_r.isNone(3)) {
 9974:         // aten::sort(Tensor self, int dim=-1, bool descending=False) -> (Tensor values, Tensor indices)
 9975: 
 9976:         auto dispatch_sort = [](const at::Tensor & self, int64_t dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
 9977:           pybind11::gil_scoped_release no_gil;
 9978:           return self.sort(dim, descending);
 9979:         };
 9980:         return wrap(NamedTuple1, dispatch_sort(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
 9981:       } else {
 9982:         // aten::sort.values(Tensor self, int dim=-1, bool descending=False, *, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
 9983:         auto out = _r.tensorlist_n<2>(3);
 9984:         auto dispatch_sort_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, int64_t dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
 9985:           pybind11::gil_scoped_release no_gil;
 9986:           return at::sort_out(values, indices, self, dim, descending);
 9987:         };
 9988:         return wrap(NamedTuple, dispatch_sort_out(out[0], out[1], _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
 9989:       }
 9990:     }
 9991:     case 2: {
 9992:       if (_r.isNone(4)) {
 9993:         // aten::sort.dimname_stable(Tensor self, *, bool? stable, Dimname dim, bool descending=False) -> (Tensor values, Tensor indices)
 9994: 
 9995:         auto dispatch_sort = [](const at::Tensor & self, ::std::optional<bool> stable, at::Dimname dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
 9996:           pybind11::gil_scoped_release no_gil;
 9997:           return self.sort(stable, dim, descending);
 9998:         };
 9999:         return wrap(NamedTuple1, dispatch_sort(_r.tensor(0), _r.toBoolOptional(1), _r.dimname(2), _r.toBool(3)));
10000:       } else {
```

- EN: The main execution path in this span is carried by `fmin_out`, `wrap`, `THPVariable_sort`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `fmin_out`, `wrap`, `THPVariable_sort` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10001-10080

```cpp
10001:         // aten::sort.dimname_values_stable(Tensor self, *, bool? stable, Dimname dim, bool descending=False, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
10002:         auto out = _r.tensorlist_n<2>(4);
10003:         auto dispatch_sort_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, ::std::optional<bool> stable, at::Dimname dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
10004:           pybind11::gil_scoped_release no_gil;
10005:           return at::sort_out(values, indices, self, stable, dim, descending);
10006:         };
10007:         return wrap(NamedTuple, dispatch_sort_out(out[0], out[1], _r.tensor(0), _r.toBoolOptional(1), _r.dimname(2), _r.toBool(3)));
10008:       }
10009:     }
10010:     case 3: {
10011:       if (_r.isNone(3)) {
10012:         // aten::sort.dimname(Tensor self, Dimname dim, bool descending=False) -> (Tensor values, Tensor indices)
10013: 
10014:         auto dispatch_sort = [](const at::Tensor & self, at::Dimname dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
10015:           pybind11::gil_scoped_release no_gil;
10016:           return self.sort(dim, descending);
10017:         };
10018:         return wrap(NamedTuple1, dispatch_sort(_r.tensor(0), _r.dimname(1), _r.toBool(2)));
10019:       } else {
10020:         // aten::sort.dimname_values(Tensor self, Dimname dim, bool descending=False, *, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
10021:         auto out = _r.tensorlist_n<2>(3);
10022:         auto dispatch_sort_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, at::Dimname dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
10023:           pybind11::gil_scoped_release no_gil;
10024:           return at::sort_out(values, indices, self, dim, descending);
10025:         };
10026:         return wrap(NamedTuple, dispatch_sort_out(out[0], out[1], _r.tensor(0), _r.dimname(1), _r.toBool(2)));
10027:       }
10028:     }
10029:   }
10030:   Py_RETURN_NONE;
10031:   END_HANDLE_TH_ERRORS
10032: }
10033: 
10034: // renorm
10035: static PyObject * THPVariable_renorm(PyObject* self_, PyObject* args, PyObject* kwargs)
10036: {
10037:   HANDLE_TH_ERRORS
10038:   static PythonArgParser parser({
10039:     "renorm(Tensor input, Scalar p, int64_t dim, Scalar maxnorm, *, Tensor out=None)",
10040:   }, /*traceable=*/true);
10041: 
10042:   ParsedArgs<5> parsed_args;
10043:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10044:   if(_r.has_torch_function()) {
10045:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10046:   }
10047:   if (_r.isNone(4)) {
10048:     // aten::renorm(Tensor self, Scalar p, int dim, Scalar maxnorm) -> Tensor
10049: 
10050:     auto dispatch_renorm = [](const at::Tensor & self, const at::Scalar & p, int64_t dim, const at::Scalar & maxnorm) -> at::Tensor {
10051:       pybind11::gil_scoped_release no_gil;
10052:       return self.renorm(p, dim, maxnorm);
10053:     };
10054:     return wrap(dispatch_renorm(_r.tensor(0), _r.scalar(1), _r.toInt64(2), _r.scalar(3)));
10055:   } else {
10056:     // aten::renorm.out(Tensor self, Scalar p, int dim, Scalar maxnorm, *, Tensor(a!) out) -> Tensor(a!)
10057: 
10058:     auto dispatch_renorm_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & p, int64_t dim, const at::Scalar & maxnorm) -> at::Tensor {
10059:       pybind11::gil_scoped_release no_gil;
10060:       return at::renorm_out(out, self, p, dim, maxnorm);
10061:     };
10062:     return wrap(dispatch_renorm_out(_r.tensor(4), _r.tensor(0), _r.scalar(1), _r.toInt64(2), _r.scalar(3)));
10063:   }
10064:   Py_RETURN_NONE;
10065:   END_HANDLE_TH_ERRORS
10066: }
10067: 
10068: // _amp_foreach_non_finite_check_and_unscale_
10069: static PyObject * THPVariable__amp_foreach_non_finite_check_and_unscale_(PyObject* self_, PyObject* args, PyObject* kwargs)
10070: {
10071:   HANDLE_TH_ERRORS
10072:   static PythonArgParser parser({
10073:     "_amp_foreach_non_finite_check_and_unscale_(TensorList self, Tensor found_inf, Tensor inv_scale)",
10074:   }, /*traceable=*/false);
10075: 
10076:   ParsedArgs<3> parsed_args;
10077:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10078:   if(_r.has_torch_function()) {
10079:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10080:   }
```

- EN: The main execution path in this span is carried by `sort_out`, `wrap`, `THPVariable_renorm`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `sort_out`, `wrap`, `THPVariable_renorm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10081-10160

```cpp
10081:   // aten::_amp_foreach_non_finite_check_and_unscale_(Tensor(a!)[] self, Tensor(b!) found_inf, Tensor inv_scale) -> ()
10082: 
10083:   auto dispatch__amp_foreach_non_finite_check_and_unscale_ = [](at::TensorList self, at::Tensor found_inf, const at::Tensor & inv_scale) -> void {
10084:     pybind11::gil_scoped_release no_gil;
10085:     at::_amp_foreach_non_finite_check_and_unscale_(self, found_inf, inv_scale);
10086:   };
10087:   dispatch__amp_foreach_non_finite_check_and_unscale_(_r.tensorlist(0), _r.tensor(1), _r.tensor(2));
10088:   Py_RETURN_NONE;
10089:   Py_RETURN_NONE;
10090:   END_HANDLE_TH_ERRORS
10091: }
10092: 
10093: // _amp_update_scale_
10094: static PyObject * THPVariable__amp_update_scale_(PyObject* self_, PyObject* args, PyObject* kwargs)
10095: {
10096:   HANDLE_TH_ERRORS
10097:   static PythonArgParser parser({
10098:     "_amp_update_scale_(Tensor input, Tensor growth_tracker, Tensor found_inf, double scale_growth_factor, double scale_backoff_factor, int64_t growth_interval)",
10099:   }, /*traceable=*/true);
10100: 
10101:   ParsedArgs<6> parsed_args;
10102:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10103:   if(_r.has_torch_function()) {
10104:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10105:   }
10106:   // aten::_amp_update_scale_(Tensor(a!) self, Tensor(b!) growth_tracker, Tensor found_inf, float scale_growth_factor, float scale_backoff_factor, int growth_interval) -> Tensor(a!)
10107: 
10108:   auto dispatch__amp_update_scale_ = [](at::Tensor self, at::Tensor growth_tracker, const at::Tensor & found_inf, double scale_growth_factor, double scale_backoff_factor, int64_t growth_interval) -> at::Tensor {
10109:     pybind11::gil_scoped_release no_gil;
10110:     return at::_amp_update_scale_(self, growth_tracker, found_inf, scale_growth_factor, scale_backoff_factor, growth_interval);
10111:   };
10112:   return wrap(dispatch__amp_update_scale_(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3), _r.toDouble(4), _r.toInt64(5)));
10113:   Py_RETURN_NONE;
10114:   END_HANDLE_TH_ERRORS
10115: }
10116: 
10117: \
10118: // _foreach_mul
10119: static PyObject * THPVariable__foreach_mul(PyObject* self_, PyObject* args, PyObject* kwargs)
10120: {
10121:   HANDLE_TH_ERRORS
10122:   static PythonArgParser parser({
10123:     "_foreach_mul(TensorList self, ScalarList scalars)",
10124:     "_foreach_mul(TensorList self, Tensor other)",
10125:     "_foreach_mul(TensorList self, TensorList other)",
10126:     "_foreach_mul(TensorList self, Scalar scalar)",
10127:   }, /*traceable=*/true);
10128: 
10129:   ParsedArgs<2> parsed_args;
10130:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10131:   if(_r.has_torch_function()) {
10132:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10133:   }
10134:   switch (_r.idx) {
10135:     case 0: {
10136:       // aten::_foreach_mul.ScalarList(Tensor[] self, Scalar[] scalars) -> Tensor[]
10137: 
10138:       auto dispatch__foreach_mul = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
10139:         pybind11::gil_scoped_release no_gil;
10140:         return at::_foreach_mul(self, scalars);
10141:       };
10142:       return wrap(dispatch__foreach_mul(_r.tensorlist(0), _r.scalarlist(1)));
10143:     }
10144:     case 1: {
10145:       // aten::_foreach_mul.Tensor(Tensor[] self, Tensor other) -> Tensor[]
10146: 
10147:       auto dispatch__foreach_mul = [](at::TensorList self, const at::Tensor & other) -> ::std::vector<at::Tensor> {
10148:         pybind11::gil_scoped_release no_gil;
10149:         return at::_foreach_mul(self, other);
10150:       };
10151:       return wrap(dispatch__foreach_mul(_r.tensorlist(0), _r.tensor(1)));
10152:     }
10153:     case 2: {
10154:       // aten::_foreach_mul.List(Tensor[] self, Tensor[] other) -> Tensor[]
10155: 
10156:       auto dispatch__foreach_mul = [](at::TensorList self, at::TensorList other) -> ::std::vector<at::Tensor> {
10157:         pybind11::gil_scoped_release no_gil;
10158:         return at::_foreach_mul(self, other);
10159:       };
10160:       return wrap(dispatch__foreach_mul(_r.tensorlist(0), _r.tensorlist(1)));
```

- EN: The main execution path in this span is carried by `_amp_foreach_non_finite_check_and_unscale_`, `dispatch__amp_foreach_non_finite_check_and_unscale_`, `THPVariable__amp_update_scale_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_amp_foreach_non_finite_check_and_unscale_`, `dispatch__amp_foreach_non_finite_check_and_unscale_`, `THPVariable__amp_update_scale_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10161-10240

```cpp
10161:     }
10162:     case 3: {
10163:       // aten::_foreach_mul.Scalar(Tensor[] self, Scalar scalar) -> Tensor[]
10164: 
10165:       auto dispatch__foreach_mul = [](at::TensorList self, const at::Scalar & scalar) -> ::std::vector<at::Tensor> {
10166:         pybind11::gil_scoped_release no_gil;
10167:         return at::_foreach_mul(self, scalar);
10168:       };
10169:       return wrap(dispatch__foreach_mul(_r.tensorlist(0), _r.scalar(1)));
10170:     }
10171:   }
10172:   Py_RETURN_NONE;
10173:   END_HANDLE_TH_ERRORS
10174: }
10175: 
10176: \
10177: // _foreach_mul_
10178: static PyObject * THPVariable__foreach_mul_(PyObject* self_, PyObject* args, PyObject* kwargs)
10179: {
10180:   HANDLE_TH_ERRORS
10181:   static PythonArgParser parser({
10182:     "_foreach_mul_(TensorList self, ScalarList scalars)",
10183:     "_foreach_mul_(TensorList self, Tensor other)",
10184:     "_foreach_mul_(TensorList self, TensorList other)",
10185:     "_foreach_mul_(TensorList self, Scalar scalar)",
10186:   }, /*traceable=*/false);
10187: 
10188:   ParsedArgs<2> parsed_args;
10189:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10190:   if(_r.has_torch_function()) {
10191:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10192:   }
10193:   switch (_r.idx) {
10194:     case 0: {
10195:       // aten::_foreach_mul_.ScalarList(Tensor(a!)[] self, Scalar[] scalars) -> ()
10196: 
10197:       auto dispatch__foreach_mul_ = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> void {
10198:         pybind11::gil_scoped_release no_gil;
10199:         at::_foreach_mul_(self, scalars);
10200:       };
10201:       dispatch__foreach_mul_(_r.tensorlist(0), _r.scalarlist(1));
10202:       PyObject* self_tensorlist = _r.args[0];
10203:       Py_INCREF(self_tensorlist);
10204:       return self_tensorlist;
10205:     }
10206:     case 1: {
10207:       // aten::_foreach_mul_.Tensor(Tensor(a!)[] self, Tensor other) -> ()
10208: 
10209:       auto dispatch__foreach_mul_ = [](at::TensorList self, const at::Tensor & other) -> void {
10210:         pybind11::gil_scoped_release no_gil;
10211:         at::_foreach_mul_(self, other);
10212:       };
10213:       dispatch__foreach_mul_(_r.tensorlist(0), _r.tensor(1));
10214:       PyObject* self_tensorlist = _r.args[0];
10215:       Py_INCREF(self_tensorlist);
10216:       return self_tensorlist;
10217:     }
10218:     case 2: {
10219:       // aten::_foreach_mul_.List(Tensor(a!)[] self, Tensor[] other) -> ()
10220: 
10221:       auto dispatch__foreach_mul_ = [](at::TensorList self, at::TensorList other) -> void {
10222:         pybind11::gil_scoped_release no_gil;
10223:         at::_foreach_mul_(self, other);
10224:       };
10225:       dispatch__foreach_mul_(_r.tensorlist(0), _r.tensorlist(1));
10226:       PyObject* self_tensorlist = _r.args[0];
10227:       Py_INCREF(self_tensorlist);
10228:       return self_tensorlist;
10229:     }
10230:     case 3: {
10231:       // aten::_foreach_mul_.Scalar(Tensor(a!)[] self, Scalar scalar) -> ()
10232: 
10233:       auto dispatch__foreach_mul_ = [](at::TensorList self, const at::Scalar & scalar) -> void {
10234:         pybind11::gil_scoped_release no_gil;
10235:         at::_foreach_mul_(self, scalar);
10236:       };
10237:       dispatch__foreach_mul_(_r.tensorlist(0), _r.scalar(1));
10238:       PyObject* self_tensorlist = _r.args[0];
10239:       Py_INCREF(self_tensorlist);
10240:       return self_tensorlist;
```

- EN: The main execution path in this span is carried by `_foreach_mul`, `wrap`, `THPVariable__foreach_mul_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_foreach_mul`, `wrap`, `THPVariable__foreach_mul_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10241-10320

```cpp
10241:     }
10242:   }
10243:   Py_RETURN_NONE;
10244:   END_HANDLE_TH_ERRORS
10245: }
10246: 
10247: \
10248: // _foreach_maximum
10249: static PyObject * THPVariable__foreach_maximum(PyObject* self_, PyObject* args, PyObject* kwargs)
10250: {
10251:   HANDLE_TH_ERRORS
10252:   static PythonArgParser parser({
10253:     "_foreach_maximum(TensorList self, Scalar scalar)",
10254:     "_foreach_maximum(TensorList self, ScalarList scalars)",
10255:     "_foreach_maximum(TensorList self, TensorList other)",
10256:   }, /*traceable=*/true);
10257: 
10258:   ParsedArgs<2> parsed_args;
10259:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10260:   if(_r.has_torch_function()) {
10261:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10262:   }
10263:   switch (_r.idx) {
10264:     case 0: {
10265:       // aten::_foreach_maximum.Scalar(Tensor[] self, Scalar scalar) -> Tensor[]
10266: 
10267:       auto dispatch__foreach_maximum = [](at::TensorList self, const at::Scalar & scalar) -> ::std::vector<at::Tensor> {
10268:         pybind11::gil_scoped_release no_gil;
10269:         return at::_foreach_maximum(self, scalar);
10270:       };
10271:       return wrap(dispatch__foreach_maximum(_r.tensorlist(0), _r.scalar(1)));
10272:     }
10273:     case 1: {
10274:       // aten::_foreach_maximum.ScalarList(Tensor[] self, Scalar[] scalars) -> Tensor[]
10275: 
10276:       auto dispatch__foreach_maximum = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
10277:         pybind11::gil_scoped_release no_gil;
10278:         return at::_foreach_maximum(self, scalars);
10279:       };
10280:       return wrap(dispatch__foreach_maximum(_r.tensorlist(0), _r.scalarlist(1)));
10281:     }
10282:     case 2: {
10283:       // aten::_foreach_maximum.List(Tensor[] self, Tensor[] other) -> Tensor[]
10284: 
10285:       auto dispatch__foreach_maximum = [](at::TensorList self, at::TensorList other) -> ::std::vector<at::Tensor> {
10286:         pybind11::gil_scoped_release no_gil;
10287:         return at::_foreach_maximum(self, other);
10288:       };
10289:       return wrap(dispatch__foreach_maximum(_r.tensorlist(0), _r.tensorlist(1)));
10290:     }
10291:   }
10292:   Py_RETURN_NONE;
10293:   END_HANDLE_TH_ERRORS
10294: }
10295: 
10296: \
10297: // _foreach_maximum_
10298: static PyObject * THPVariable__foreach_maximum_(PyObject* self_, PyObject* args, PyObject* kwargs)
10299: {
10300:   HANDLE_TH_ERRORS
10301:   static PythonArgParser parser({
10302:     "_foreach_maximum_(TensorList self, Scalar scalar)",
10303:     "_foreach_maximum_(TensorList self, ScalarList scalars)",
10304:     "_foreach_maximum_(TensorList self, TensorList other)",
10305:   }, /*traceable=*/false);
10306: 
10307:   ParsedArgs<2> parsed_args;
10308:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10309:   if(_r.has_torch_function()) {
10310:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10311:   }
10312:   switch (_r.idx) {
10313:     case 0: {
10314:       // aten::_foreach_maximum_.Scalar(Tensor(a!)[] self, Scalar scalar) -> ()
10315: 
10316:       auto dispatch__foreach_maximum_ = [](at::TensorList self, const at::Scalar & scalar) -> void {
10317:         pybind11::gil_scoped_release no_gil;
10318:         at::_foreach_maximum_(self, scalar);
10319:       };
10320:       dispatch__foreach_maximum_(_r.tensorlist(0), _r.scalar(1));
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_maximum`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_maximum`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10321-10400

```cpp
10321:       PyObject* self_tensorlist = _r.args[0];
10322:       Py_INCREF(self_tensorlist);
10323:       return self_tensorlist;
10324:     }
10325:     case 1: {
10326:       // aten::_foreach_maximum_.ScalarList(Tensor(a!)[] self, Scalar[] scalars) -> ()
10327: 
10328:       auto dispatch__foreach_maximum_ = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> void {
10329:         pybind11::gil_scoped_release no_gil;
10330:         at::_foreach_maximum_(self, scalars);
10331:       };
10332:       dispatch__foreach_maximum_(_r.tensorlist(0), _r.scalarlist(1));
10333:       PyObject* self_tensorlist = _r.args[0];
10334:       Py_INCREF(self_tensorlist);
10335:       return self_tensorlist;
10336:     }
10337:     case 2: {
10338:       // aten::_foreach_maximum_.List(Tensor(a!)[] self, Tensor[] other) -> ()
10339: 
10340:       auto dispatch__foreach_maximum_ = [](at::TensorList self, at::TensorList other) -> void {
10341:         pybind11::gil_scoped_release no_gil;
10342:         at::_foreach_maximum_(self, other);
10343:       };
10344:       dispatch__foreach_maximum_(_r.tensorlist(0), _r.tensorlist(1));
10345:       PyObject* self_tensorlist = _r.args[0];
10346:       Py_INCREF(self_tensorlist);
10347:       return self_tensorlist;
10348:     }
10349:   }
10350:   Py_RETURN_NONE;
10351:   END_HANDLE_TH_ERRORS
10352: }
10353: 
10354: \
10355: // _foreach_minimum
10356: static PyObject * THPVariable__foreach_minimum(PyObject* self_, PyObject* args, PyObject* kwargs)
10357: {
10358:   HANDLE_TH_ERRORS
10359:   static PythonArgParser parser({
10360:     "_foreach_minimum(TensorList self, Scalar scalar)",
10361:     "_foreach_minimum(TensorList self, ScalarList scalars)",
10362:     "_foreach_minimum(TensorList self, TensorList other)",
10363:   }, /*traceable=*/true);
10364: 
10365:   ParsedArgs<2> parsed_args;
10366:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10367:   if(_r.has_torch_function()) {
10368:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10369:   }
10370:   switch (_r.idx) {
10371:     case 0: {
10372:       // aten::_foreach_minimum.Scalar(Tensor[] self, Scalar scalar) -> Tensor[]
10373: 
10374:       auto dispatch__foreach_minimum = [](at::TensorList self, const at::Scalar & scalar) -> ::std::vector<at::Tensor> {
10375:         pybind11::gil_scoped_release no_gil;
10376:         return at::_foreach_minimum(self, scalar);
10377:       };
10378:       return wrap(dispatch__foreach_minimum(_r.tensorlist(0), _r.scalar(1)));
10379:     }
10380:     case 1: {
10381:       // aten::_foreach_minimum.ScalarList(Tensor[] self, Scalar[] scalars) -> Tensor[]
10382: 
10383:       auto dispatch__foreach_minimum = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
10384:         pybind11::gil_scoped_release no_gil;
10385:         return at::_foreach_minimum(self, scalars);
10386:       };
10387:       return wrap(dispatch__foreach_minimum(_r.tensorlist(0), _r.scalarlist(1)));
10388:     }
10389:     case 2: {
10390:       // aten::_foreach_minimum.List(Tensor[] self, Tensor[] other) -> Tensor[]
10391: 
10392:       auto dispatch__foreach_minimum = [](at::TensorList self, at::TensorList other) -> ::std::vector<at::Tensor> {
10393:         pybind11::gil_scoped_release no_gil;
10394:         return at::_foreach_minimum(self, other);
10395:       };
10396:       return wrap(dispatch__foreach_minimum(_r.tensorlist(0), _r.tensorlist(1)));
10397:     }
10398:   }
10399:   Py_RETURN_NONE;
10400:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `Py_INCREF`, `_foreach_maximum_`, `dispatch__foreach_maximum_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `Py_INCREF`, `_foreach_maximum_`, `dispatch__foreach_maximum_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10401-10480

```cpp
10401: }
10402: 
10403: \
10404: // _foreach_minimum_
10405: static PyObject * THPVariable__foreach_minimum_(PyObject* self_, PyObject* args, PyObject* kwargs)
10406: {
10407:   HANDLE_TH_ERRORS
10408:   static PythonArgParser parser({
10409:     "_foreach_minimum_(TensorList self, Scalar scalar)",
10410:     "_foreach_minimum_(TensorList self, ScalarList scalars)",
10411:     "_foreach_minimum_(TensorList self, TensorList other)",
10412:   }, /*traceable=*/false);
10413: 
10414:   ParsedArgs<2> parsed_args;
10415:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10416:   if(_r.has_torch_function()) {
10417:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10418:   }
10419:   switch (_r.idx) {
10420:     case 0: {
10421:       // aten::_foreach_minimum_.Scalar(Tensor(a!)[] self, Scalar scalar) -> ()
10422: 
10423:       auto dispatch__foreach_minimum_ = [](at::TensorList self, const at::Scalar & scalar) -> void {
10424:         pybind11::gil_scoped_release no_gil;
10425:         at::_foreach_minimum_(self, scalar);
10426:       };
10427:       dispatch__foreach_minimum_(_r.tensorlist(0), _r.scalar(1));
10428:       PyObject* self_tensorlist = _r.args[0];
10429:       Py_INCREF(self_tensorlist);
10430:       return self_tensorlist;
10431:     }
10432:     case 1: {
10433:       // aten::_foreach_minimum_.ScalarList(Tensor(a!)[] self, Scalar[] scalars) -> ()
10434: 
10435:       auto dispatch__foreach_minimum_ = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> void {
10436:         pybind11::gil_scoped_release no_gil;
10437:         at::_foreach_minimum_(self, scalars);
10438:       };
10439:       dispatch__foreach_minimum_(_r.tensorlist(0), _r.scalarlist(1));
10440:       PyObject* self_tensorlist = _r.args[0];
10441:       Py_INCREF(self_tensorlist);
10442:       return self_tensorlist;
10443:     }
10444:     case 2: {
10445:       // aten::_foreach_minimum_.List(Tensor(a!)[] self, Tensor[] other) -> ()
10446: 
10447:       auto dispatch__foreach_minimum_ = [](at::TensorList self, at::TensorList other) -> void {
10448:         pybind11::gil_scoped_release no_gil;
10449:         at::_foreach_minimum_(self, other);
10450:       };
10451:       dispatch__foreach_minimum_(_r.tensorlist(0), _r.tensorlist(1));
10452:       PyObject* self_tensorlist = _r.args[0];
10453:       Py_INCREF(self_tensorlist);
10454:       return self_tensorlist;
10455:     }
10456:   }
10457:   Py_RETURN_NONE;
10458:   END_HANDLE_TH_ERRORS
10459: }
10460: 
10461: \
10462: // _foreach_addcdiv
10463: static PyObject * THPVariable__foreach_addcdiv(PyObject* self_, PyObject* args, PyObject* kwargs)
10464: {
10465:   HANDLE_TH_ERRORS
10466:   static PythonArgParser parser({
10467:     "_foreach_addcdiv(TensorList self, TensorList tensor1, TensorList tensor2, ScalarList scalars)",
10468:     "_foreach_addcdiv(TensorList self, TensorList tensor1, TensorList tensor2, Tensor scalars)",
10469:     "_foreach_addcdiv(TensorList self, TensorList tensor1, TensorList tensor2, Scalar value=1)",
10470:   }, /*traceable=*/true);
10471: 
10472:   ParsedArgs<4> parsed_args;
10473:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10474:   if(_r.has_torch_function()) {
10475:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10476:   }
10477:   switch (_r.idx) {
10478:     case 0: {
10479:       // aten::_foreach_addcdiv.ScalarList(Tensor[] self, Tensor[] tensor1, Tensor[] tensor2, Scalar[] scalars) -> Tensor[]
10480: 
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_minimum_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_minimum_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10481-10560

```cpp
10481:       auto dispatch__foreach_addcdiv = [](at::TensorList self, at::TensorList tensor1, at::TensorList tensor2, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
10482:         pybind11::gil_scoped_release no_gil;
10483:         return at::_foreach_addcdiv(self, tensor1, tensor2, scalars);
10484:       };
10485:       return wrap(dispatch__foreach_addcdiv(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.scalarlist(3)));
10486:     }
10487:     case 1: {
10488:       // aten::_foreach_addcdiv.Tensor(Tensor[] self, Tensor[] tensor1, Tensor[] tensor2, Tensor scalars) -> Tensor[]
10489: 
10490:       auto dispatch__foreach_addcdiv = [](at::TensorList self, at::TensorList tensor1, at::TensorList tensor2, const at::Tensor & scalars) -> ::std::vector<at::Tensor> {
10491:         pybind11::gil_scoped_release no_gil;
10492:         return at::_foreach_addcdiv(self, tensor1, tensor2, scalars);
10493:       };
10494:       return wrap(dispatch__foreach_addcdiv(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.tensor(3)));
10495:     }
10496:     case 2: {
10497:       // aten::_foreach_addcdiv.Scalar(Tensor[] self, Tensor[] tensor1, Tensor[] tensor2, Scalar value=1) -> Tensor[]
10498: 
10499:       auto dispatch__foreach_addcdiv = [](at::TensorList self, at::TensorList tensor1, at::TensorList tensor2, const at::Scalar & value) -> ::std::vector<at::Tensor> {
10500:         pybind11::gil_scoped_release no_gil;
10501:         return at::_foreach_addcdiv(self, tensor1, tensor2, value);
10502:       };
10503:       return wrap(dispatch__foreach_addcdiv(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.scalar(3)));
10504:     }
10505:   }
10506:   Py_RETURN_NONE;
10507:   END_HANDLE_TH_ERRORS
10508: }
10509: 
10510: \
10511: // _foreach_addcdiv_
10512: static PyObject * THPVariable__foreach_addcdiv_(PyObject* self_, PyObject* args, PyObject* kwargs)
10513: {
10514:   HANDLE_TH_ERRORS
10515:   static PythonArgParser parser({
10516:     "_foreach_addcdiv_(TensorList self, TensorList tensor1, TensorList tensor2, ScalarList scalars)",
10517:     "_foreach_addcdiv_(TensorList self, TensorList tensor1, TensorList tensor2, Tensor scalars)",
10518:     "_foreach_addcdiv_(TensorList self, TensorList tensor1, TensorList tensor2, Scalar value=1)",
10519:   }, /*traceable=*/false);
10520: 
10521:   ParsedArgs<4> parsed_args;
10522:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10523:   if(_r.has_torch_function()) {
10524:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10525:   }
10526:   switch (_r.idx) {
10527:     case 0: {
10528:       // aten::_foreach_addcdiv_.ScalarList(Tensor(a!)[] self, Tensor[] tensor1, Tensor[] tensor2, Scalar[] scalars) -> ()
10529: 
10530:       auto dispatch__foreach_addcdiv_ = [](at::TensorList self, at::TensorList tensor1, at::TensorList tensor2, at::ArrayRef<at::Scalar> scalars) -> void {
10531:         pybind11::gil_scoped_release no_gil;
10532:         at::_foreach_addcdiv_(self, tensor1, tensor2, scalars);
10533:       };
10534:       dispatch__foreach_addcdiv_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.scalarlist(3));
10535:       PyObject* self_tensorlist = _r.args[0];
10536:       Py_INCREF(self_tensorlist);
10537:       return self_tensorlist;
10538:     }
10539:     case 1: {
10540:       // aten::_foreach_addcdiv_.Tensor(Tensor(a!)[] self, Tensor[] tensor1, Tensor[] tensor2, Tensor scalars) -> ()
10541: 
10542:       auto dispatch__foreach_addcdiv_ = [](at::TensorList self, at::TensorList tensor1, at::TensorList tensor2, const at::Tensor & scalars) -> void {
10543:         pybind11::gil_scoped_release no_gil;
10544:         at::_foreach_addcdiv_(self, tensor1, tensor2, scalars);
10545:       };
10546:       dispatch__foreach_addcdiv_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.tensor(3));
10547:       PyObject* self_tensorlist = _r.args[0];
10548:       Py_INCREF(self_tensorlist);
10549:       return self_tensorlist;
10550:     }
10551:     case 2: {
10552:       // aten::_foreach_addcdiv_.Scalar(Tensor(a!)[] self, Tensor[] tensor1, Tensor[] tensor2, Scalar value=1) -> ()
10553: 
10554:       auto dispatch__foreach_addcdiv_ = [](at::TensorList self, at::TensorList tensor1, at::TensorList tensor2, const at::Scalar & value) -> void {
10555:         pybind11::gil_scoped_release no_gil;
10556:         at::_foreach_addcdiv_(self, tensor1, tensor2, value);
10557:       };
10558:       dispatch__foreach_addcdiv_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.scalar(3));
10559:       PyObject* self_tensorlist = _r.args[0];
10560:       Py_INCREF(self_tensorlist);
```

- EN: The main execution path in this span is carried by `_foreach_addcdiv`, `wrap`, `THPVariable__foreach_addcdiv_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_foreach_addcdiv`, `wrap`, `THPVariable__foreach_addcdiv_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10561-10640

```cpp
10561:       return self_tensorlist;
10562:     }
10563:   }
10564:   Py_RETURN_NONE;
10565:   END_HANDLE_TH_ERRORS
10566: }
10567: 
10568: // _foreach_abs
10569: static PyObject * THPVariable__foreach_abs(PyObject* self_, PyObject* args, PyObject* kwargs)
10570: {
10571:   HANDLE_TH_ERRORS
10572:   static PythonArgParser parser({
10573:     "_foreach_abs(TensorList self)",
10574:   }, /*traceable=*/true);
10575: 
10576:   ParsedArgs<1> parsed_args;
10577:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10578:   if(_r.has_torch_function()) {
10579:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10580:   }
10581:   // aten::_foreach_abs(Tensor[] self) -> Tensor[]
10582: 
10583:   auto dispatch__foreach_abs = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10584:     pybind11::gil_scoped_release no_gil;
10585:     return at::_foreach_abs(self);
10586:   };
10587:   return wrap(dispatch__foreach_abs(_r.tensorlist(0)));
10588:   Py_RETURN_NONE;
10589:   END_HANDLE_TH_ERRORS
10590: }
10591: 
10592: // _foreach_abs_
10593: static PyObject * THPVariable__foreach_abs_(PyObject* self_, PyObject* args, PyObject* kwargs)
10594: {
10595:   HANDLE_TH_ERRORS
10596:   static PythonArgParser parser({
10597:     "_foreach_abs_(TensorList self)",
10598:   }, /*traceable=*/false);
10599: 
10600:   ParsedArgs<1> parsed_args;
10601:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10602:   if(_r.has_torch_function()) {
10603:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10604:   }
10605:   // aten::_foreach_abs_(Tensor(a!)[] self) -> ()
10606: 
10607:   auto dispatch__foreach_abs_ = [](at::TensorList self) -> void {
10608:     pybind11::gil_scoped_release no_gil;
10609:     at::_foreach_abs_(self);
10610:   };
10611:   dispatch__foreach_abs_(_r.tensorlist(0));
10612:   PyObject* self_tensorlist = _r.args[0];
10613:   Py_INCREF(self_tensorlist);
10614:   return self_tensorlist;
10615:   Py_RETURN_NONE;
10616:   END_HANDLE_TH_ERRORS
10617: }
10618: 
10619: // _foreach_erf
10620: static PyObject * THPVariable__foreach_erf(PyObject* self_, PyObject* args, PyObject* kwargs)
10621: {
10622:   HANDLE_TH_ERRORS
10623:   static PythonArgParser parser({
10624:     "_foreach_erf(TensorList self)",
10625:   }, /*traceable=*/true);
10626: 
10627:   ParsedArgs<1> parsed_args;
10628:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10629:   if(_r.has_torch_function()) {
10630:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10631:   }
10632:   // aten::_foreach_erf(Tensor[] self) -> Tensor[]
10633: 
10634:   auto dispatch__foreach_erf = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10635:     pybind11::gil_scoped_release no_gil;
10636:     return at::_foreach_erf(self);
10637:   };
10638:   return wrap(dispatch__foreach_erf(_r.tensorlist(0)));
10639:   Py_RETURN_NONE;
10640:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_abs`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_abs`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10641-10720

```cpp
10641: }
10642: 
10643: // _foreach_erf_
10644: static PyObject * THPVariable__foreach_erf_(PyObject* self_, PyObject* args, PyObject* kwargs)
10645: {
10646:   HANDLE_TH_ERRORS
10647:   static PythonArgParser parser({
10648:     "_foreach_erf_(TensorList self)",
10649:   }, /*traceable=*/false);
10650: 
10651:   ParsedArgs<1> parsed_args;
10652:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10653:   if(_r.has_torch_function()) {
10654:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10655:   }
10656:   // aten::_foreach_erf_(Tensor(a!)[] self) -> ()
10657: 
10658:   auto dispatch__foreach_erf_ = [](at::TensorList self) -> void {
10659:     pybind11::gil_scoped_release no_gil;
10660:     at::_foreach_erf_(self);
10661:   };
10662:   dispatch__foreach_erf_(_r.tensorlist(0));
10663:   PyObject* self_tensorlist = _r.args[0];
10664:   Py_INCREF(self_tensorlist);
10665:   return self_tensorlist;
10666:   Py_RETURN_NONE;
10667:   END_HANDLE_TH_ERRORS
10668: }
10669: 
10670: // _foreach_frac
10671: static PyObject * THPVariable__foreach_frac(PyObject* self_, PyObject* args, PyObject* kwargs)
10672: {
10673:   HANDLE_TH_ERRORS
10674:   static PythonArgParser parser({
10675:     "_foreach_frac(TensorList self)",
10676:   }, /*traceable=*/true);
10677: 
10678:   ParsedArgs<1> parsed_args;
10679:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10680:   if(_r.has_torch_function()) {
10681:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10682:   }
10683:   // aten::_foreach_frac(Tensor[] self) -> Tensor[]
10684: 
10685:   auto dispatch__foreach_frac = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10686:     pybind11::gil_scoped_release no_gil;
10687:     return at::_foreach_frac(self);
10688:   };
10689:   return wrap(dispatch__foreach_frac(_r.tensorlist(0)));
10690:   Py_RETURN_NONE;
10691:   END_HANDLE_TH_ERRORS
10692: }
10693: 
10694: // _foreach_frac_
10695: static PyObject * THPVariable__foreach_frac_(PyObject* self_, PyObject* args, PyObject* kwargs)
10696: {
10697:   HANDLE_TH_ERRORS
10698:   static PythonArgParser parser({
10699:     "_foreach_frac_(TensorList self)",
10700:   }, /*traceable=*/false);
10701: 
10702:   ParsedArgs<1> parsed_args;
10703:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10704:   if(_r.has_torch_function()) {
10705:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10706:   }
10707:   // aten::_foreach_frac_(Tensor(a!)[] self) -> ()
10708: 
10709:   auto dispatch__foreach_frac_ = [](at::TensorList self) -> void {
10710:     pybind11::gil_scoped_release no_gil;
10711:     at::_foreach_frac_(self);
10712:   };
10713:   dispatch__foreach_frac_(_r.tensorlist(0));
10714:   PyObject* self_tensorlist = _r.args[0];
10715:   Py_INCREF(self_tensorlist);
10716:   return self_tensorlist;
10717:   Py_RETURN_NONE;
10718:   END_HANDLE_TH_ERRORS
10719: }
10720: 
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_erf_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_erf_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10721-10800

```cpp
10721: // _foreach_lgamma
10722: static PyObject * THPVariable__foreach_lgamma(PyObject* self_, PyObject* args, PyObject* kwargs)
10723: {
10724:   HANDLE_TH_ERRORS
10725:   static PythonArgParser parser({
10726:     "_foreach_lgamma(TensorList self)",
10727:   }, /*traceable=*/true);
10728: 
10729:   ParsedArgs<1> parsed_args;
10730:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10731:   if(_r.has_torch_function()) {
10732:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10733:   }
10734:   // aten::_foreach_lgamma(Tensor[] self) -> Tensor[]
10735: 
10736:   auto dispatch__foreach_lgamma = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10737:     pybind11::gil_scoped_release no_gil;
10738:     return at::_foreach_lgamma(self);
10739:   };
10740:   return wrap(dispatch__foreach_lgamma(_r.tensorlist(0)));
10741:   Py_RETURN_NONE;
10742:   END_HANDLE_TH_ERRORS
10743: }
10744: 
10745: // _foreach_lgamma_
10746: static PyObject * THPVariable__foreach_lgamma_(PyObject* self_, PyObject* args, PyObject* kwargs)
10747: {
10748:   HANDLE_TH_ERRORS
10749:   static PythonArgParser parser({
10750:     "_foreach_lgamma_(TensorList self)",
10751:   }, /*traceable=*/false);
10752: 
10753:   ParsedArgs<1> parsed_args;
10754:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10755:   if(_r.has_torch_function()) {
10756:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10757:   }
10758:   // aten::_foreach_lgamma_(Tensor(a!)[] self) -> ()
10759: 
10760:   auto dispatch__foreach_lgamma_ = [](at::TensorList self) -> void {
10761:     pybind11::gil_scoped_release no_gil;
10762:     at::_foreach_lgamma_(self);
10763:   };
10764:   dispatch__foreach_lgamma_(_r.tensorlist(0));
10765:   PyObject* self_tensorlist = _r.args[0];
10766:   Py_INCREF(self_tensorlist);
10767:   return self_tensorlist;
10768:   Py_RETURN_NONE;
10769:   END_HANDLE_TH_ERRORS
10770: }
10771: 
10772: // _foreach_powsum
10773: static PyObject * THPVariable__foreach_powsum(PyObject* self_, PyObject* args, PyObject* kwargs)
10774: {
10775:   HANDLE_TH_ERRORS
10776:   static PythonArgParser parser({
10777:     "_foreach_powsum(TensorList self, Scalar ord=2, ScalarType? dtype=None)",
10778:   }, /*traceable=*/true);
10779: 
10780:   ParsedArgs<3> parsed_args;
10781:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10782:   if(_r.has_torch_function()) {
10783:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10784:   }
10785:   // aten::_foreach_powsum.Scalar(Tensor[] self, Scalar ord=2, ScalarType? dtype=None) -> Tensor[]
10786: 
10787:   auto dispatch__foreach_powsum = [](at::TensorList self, const at::Scalar & ord, ::std::optional<at::ScalarType> dtype) -> ::std::vector<at::Tensor> {
10788:     pybind11::gil_scoped_release no_gil;
10789:     return at::_foreach_powsum(self, ord, dtype);
10790:   };
10791:   return wrap(dispatch__foreach_powsum(_r.tensorlist(0), _r.scalar(1), _r.scalartypeOptional(2)));
10792:   Py_RETURN_NONE;
10793:   END_HANDLE_TH_ERRORS
10794: }
10795: 
10796: \
10797: // _foreach_pow
10798: static PyObject * THPVariable__foreach_pow(PyObject* self_, PyObject* args, PyObject* kwargs)
10799: {
10800:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_lgamma`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_lgamma`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10801-10880

```cpp
10801:   static PythonArgParser parser({
10802:     "_foreach_pow(Scalar self, TensorList exponent)",
10803:     "_foreach_pow(TensorList self, Scalar exponent)",
10804:     "_foreach_pow(TensorList self, ScalarList exponent)",
10805:     "_foreach_pow(TensorList self, TensorList exponent)",
10806:   }, /*traceable=*/true);
10807: 
10808:   ParsedArgs<2> parsed_args;
10809:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10810:   if(_r.has_torch_function()) {
10811:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10812:   }
10813:   switch (_r.idx) {
10814:     case 0: {
10815:       // aten::_foreach_pow.ScalarAndTensor(Scalar self, Tensor[] exponent) -> Tensor[]
10816: 
10817:       auto dispatch__foreach_pow = [](const at::Scalar & self, at::TensorList exponent) -> ::std::vector<at::Tensor> {
10818:         pybind11::gil_scoped_release no_gil;
10819:         return at::_foreach_pow(self, exponent);
10820:       };
10821:       return wrap(dispatch__foreach_pow(_r.scalar(0), _r.tensorlist(1)));
10822:     }
10823:     case 1: {
10824:       // aten::_foreach_pow.Scalar(Tensor[] self, Scalar exponent) -> Tensor[]
10825: 
10826:       auto dispatch__foreach_pow = [](at::TensorList self, const at::Scalar & exponent) -> ::std::vector<at::Tensor> {
10827:         pybind11::gil_scoped_release no_gil;
10828:         return at::_foreach_pow(self, exponent);
10829:       };
10830:       return wrap(dispatch__foreach_pow(_r.tensorlist(0), _r.scalar(1)));
10831:     }
10832:     case 2: {
10833:       // aten::_foreach_pow.ScalarList(Tensor[] self, Scalar[] exponent) -> Tensor[]
10834: 
10835:       auto dispatch__foreach_pow = [](at::TensorList self, at::ArrayRef<at::Scalar> exponent) -> ::std::vector<at::Tensor> {
10836:         pybind11::gil_scoped_release no_gil;
10837:         return at::_foreach_pow(self, exponent);
10838:       };
10839:       return wrap(dispatch__foreach_pow(_r.tensorlist(0), _r.scalarlist(1)));
10840:     }
10841:     case 3: {
10842:       // aten::_foreach_pow.List(Tensor[] self, Tensor[] exponent) -> Tensor[]
10843: 
10844:       auto dispatch__foreach_pow = [](at::TensorList self, at::TensorList exponent) -> ::std::vector<at::Tensor> {
10845:         pybind11::gil_scoped_release no_gil;
10846:         return at::_foreach_pow(self, exponent);
10847:       };
10848:       return wrap(dispatch__foreach_pow(_r.tensorlist(0), _r.tensorlist(1)));
10849:     }
10850:   }
10851:   Py_RETURN_NONE;
10852:   END_HANDLE_TH_ERRORS
10853: }
10854: 
10855: \
10856: // _foreach_pow_
10857: static PyObject * THPVariable__foreach_pow_(PyObject* self_, PyObject* args, PyObject* kwargs)
10858: {
10859:   HANDLE_TH_ERRORS
10860:   static PythonArgParser parser({
10861:     "_foreach_pow_(TensorList self, Scalar exponent)",
10862:     "_foreach_pow_(TensorList self, ScalarList exponent)",
10863:     "_foreach_pow_(TensorList self, TensorList exponent)",
10864:   }, /*traceable=*/false);
10865: 
10866:   ParsedArgs<2> parsed_args;
10867:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10868:   if(_r.has_torch_function()) {
10869:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10870:   }
10871:   switch (_r.idx) {
10872:     case 0: {
10873:       // aten::_foreach_pow_.Scalar(Tensor(a!)[] self, Scalar exponent) -> ()
10874: 
10875:       auto dispatch__foreach_pow_ = [](at::TensorList self, const at::Scalar & exponent) -> void {
10876:         pybind11::gil_scoped_release no_gil;
10877:         at::_foreach_pow_(self, exponent);
10878:       };
10879:       dispatch__foreach_pow_(_r.tensorlist(0), _r.scalar(1));
10880:       PyObject* self_tensorlist = _r.args[0];
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_foreach_pow`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_foreach_pow` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10881-10960

```cpp
10881:       Py_INCREF(self_tensorlist);
10882:       return self_tensorlist;
10883:     }
10884:     case 1: {
10885:       // aten::_foreach_pow_.ScalarList(Tensor(a!)[] self, Scalar[] exponent) -> ()
10886: 
10887:       auto dispatch__foreach_pow_ = [](at::TensorList self, at::ArrayRef<at::Scalar> exponent) -> void {
10888:         pybind11::gil_scoped_release no_gil;
10889:         at::_foreach_pow_(self, exponent);
10890:       };
10891:       dispatch__foreach_pow_(_r.tensorlist(0), _r.scalarlist(1));
10892:       PyObject* self_tensorlist = _r.args[0];
10893:       Py_INCREF(self_tensorlist);
10894:       return self_tensorlist;
10895:     }
10896:     case 2: {
10897:       // aten::_foreach_pow_.List(Tensor(a!)[] self, Tensor[] exponent) -> ()
10898: 
10899:       auto dispatch__foreach_pow_ = [](at::TensorList self, at::TensorList exponent) -> void {
10900:         pybind11::gil_scoped_release no_gil;
10901:         at::_foreach_pow_(self, exponent);
10902:       };
10903:       dispatch__foreach_pow_(_r.tensorlist(0), _r.tensorlist(1));
10904:       PyObject* self_tensorlist = _r.args[0];
10905:       Py_INCREF(self_tensorlist);
10906:       return self_tensorlist;
10907:     }
10908:   }
10909:   Py_RETURN_NONE;
10910:   END_HANDLE_TH_ERRORS
10911: }
10912: 
10913: // _foreach_reciprocal
10914: static PyObject * THPVariable__foreach_reciprocal(PyObject* self_, PyObject* args, PyObject* kwargs)
10915: {
10916:   HANDLE_TH_ERRORS
10917:   static PythonArgParser parser({
10918:     "_foreach_reciprocal(TensorList self)",
10919:   }, /*traceable=*/true);
10920: 
10921:   ParsedArgs<1> parsed_args;
10922:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10923:   if(_r.has_torch_function()) {
10924:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10925:   }
10926:   // aten::_foreach_reciprocal(Tensor[] self) -> Tensor[]
10927: 
10928:   auto dispatch__foreach_reciprocal = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10929:     pybind11::gil_scoped_release no_gil;
10930:     return at::_foreach_reciprocal(self);
10931:   };
10932:   return wrap(dispatch__foreach_reciprocal(_r.tensorlist(0)));
10933:   Py_RETURN_NONE;
10934:   END_HANDLE_TH_ERRORS
10935: }
10936: 
10937: // _foreach_reciprocal_
10938: static PyObject * THPVariable__foreach_reciprocal_(PyObject* self_, PyObject* args, PyObject* kwargs)
10939: {
10940:   HANDLE_TH_ERRORS
10941:   static PythonArgParser parser({
10942:     "_foreach_reciprocal_(TensorList self)",
10943:   }, /*traceable=*/false);
10944: 
10945:   ParsedArgs<1> parsed_args;
10946:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10947:   if(_r.has_torch_function()) {
10948:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10949:   }
10950:   // aten::_foreach_reciprocal_(Tensor(a!)[] self) -> ()
10951: 
10952:   auto dispatch__foreach_reciprocal_ = [](at::TensorList self) -> void {
10953:     pybind11::gil_scoped_release no_gil;
10954:     at::_foreach_reciprocal_(self);
10955:   };
10956:   dispatch__foreach_reciprocal_(_r.tensorlist(0));
10957:   PyObject* self_tensorlist = _r.args[0];
10958:   Py_INCREF(self_tensorlist);
10959:   return self_tensorlist;
10960:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `Py_INCREF`, `_foreach_pow_`, `dispatch__foreach_pow_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `Py_INCREF`, `_foreach_pow_`, `dispatch__foreach_pow_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10961-11040

```cpp
10961:   END_HANDLE_TH_ERRORS
10962: }
10963: 
10964: // _foreach_round
10965: static PyObject * THPVariable__foreach_round(PyObject* self_, PyObject* args, PyObject* kwargs)
10966: {
10967:   HANDLE_TH_ERRORS
10968:   static PythonArgParser parser({
10969:     "_foreach_round(TensorList self)",
10970:   }, /*traceable=*/true);
10971: 
10972:   ParsedArgs<1> parsed_args;
10973:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10974:   if(_r.has_torch_function()) {
10975:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10976:   }
10977:   // aten::_foreach_round(Tensor[] self) -> Tensor[]
10978: 
10979:   auto dispatch__foreach_round = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10980:     pybind11::gil_scoped_release no_gil;
10981:     return at::_foreach_round(self);
10982:   };
10983:   return wrap(dispatch__foreach_round(_r.tensorlist(0)));
10984:   Py_RETURN_NONE;
10985:   END_HANDLE_TH_ERRORS
10986: }
10987: 
10988: // _foreach_round_
10989: static PyObject * THPVariable__foreach_round_(PyObject* self_, PyObject* args, PyObject* kwargs)
10990: {
10991:   HANDLE_TH_ERRORS
10992:   static PythonArgParser parser({
10993:     "_foreach_round_(TensorList self)",
10994:   }, /*traceable=*/false);
10995: 
10996:   ParsedArgs<1> parsed_args;
10997:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10998:   if(_r.has_torch_function()) {
10999:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11000:   }
11001:   // aten::_foreach_round_(Tensor(a!)[] self) -> ()
11002: 
11003:   auto dispatch__foreach_round_ = [](at::TensorList self) -> void {
11004:     pybind11::gil_scoped_release no_gil;
11005:     at::_foreach_round_(self);
11006:   };
11007:   dispatch__foreach_round_(_r.tensorlist(0));
11008:   PyObject* self_tensorlist = _r.args[0];
11009:   Py_INCREF(self_tensorlist);
11010:   return self_tensorlist;
11011:   Py_RETURN_NONE;
11012:   END_HANDLE_TH_ERRORS
11013: }
11014: 
11015: // _foreach_sinh
11016: static PyObject * THPVariable__foreach_sinh(PyObject* self_, PyObject* args, PyObject* kwargs)
11017: {
11018:   HANDLE_TH_ERRORS
11019:   static PythonArgParser parser({
11020:     "_foreach_sinh(TensorList self)",
11021:   }, /*traceable=*/true);
11022: 
11023:   ParsedArgs<1> parsed_args;
11024:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11025:   if(_r.has_torch_function()) {
11026:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11027:   }
11028:   // aten::_foreach_sinh(Tensor[] self) -> Tensor[]
11029: 
11030:   auto dispatch__foreach_sinh = [](at::TensorList self) -> ::std::vector<at::Tensor> {
11031:     pybind11::gil_scoped_release no_gil;
11032:     return at::_foreach_sinh(self);
11033:   };
11034:   return wrap(dispatch__foreach_sinh(_r.tensorlist(0)));
11035:   Py_RETURN_NONE;
11036:   END_HANDLE_TH_ERRORS
11037: }
11038: 
11039: // _foreach_sinh_
11040: static PyObject * THPVariable__foreach_sinh_(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_round`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_round`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11041-11120

```cpp
11041: {
11042:   HANDLE_TH_ERRORS
11043:   static PythonArgParser parser({
11044:     "_foreach_sinh_(TensorList self)",
11045:   }, /*traceable=*/false);
11046: 
11047:   ParsedArgs<1> parsed_args;
11048:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11049:   if(_r.has_torch_function()) {
11050:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11051:   }
11052:   // aten::_foreach_sinh_(Tensor(a!)[] self) -> ()
11053: 
11054:   auto dispatch__foreach_sinh_ = [](at::TensorList self) -> void {
11055:     pybind11::gil_scoped_release no_gil;
11056:     at::_foreach_sinh_(self);
11057:   };
11058:   dispatch__foreach_sinh_(_r.tensorlist(0));
11059:   PyObject* self_tensorlist = _r.args[0];
11060:   Py_INCREF(self_tensorlist);
11061:   return self_tensorlist;
11062:   Py_RETURN_NONE;
11063:   END_HANDLE_TH_ERRORS
11064: }
11065: 
11066: // _foreach_tan
11067: static PyObject * THPVariable__foreach_tan(PyObject* self_, PyObject* args, PyObject* kwargs)
11068: {
11069:   HANDLE_TH_ERRORS
11070:   static PythonArgParser parser({
11071:     "_foreach_tan(TensorList self)",
11072:   }, /*traceable=*/true);
11073: 
11074:   ParsedArgs<1> parsed_args;
11075:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11076:   if(_r.has_torch_function()) {
11077:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11078:   }
11079:   // aten::_foreach_tan(Tensor[] self) -> Tensor[]
11080: 
11081:   auto dispatch__foreach_tan = [](at::TensorList self) -> ::std::vector<at::Tensor> {
11082:     pybind11::gil_scoped_release no_gil;
11083:     return at::_foreach_tan(self);
11084:   };
11085:   return wrap(dispatch__foreach_tan(_r.tensorlist(0)));
11086:   Py_RETURN_NONE;
11087:   END_HANDLE_TH_ERRORS
11088: }
11089: 
11090: // _foreach_tan_
11091: static PyObject * THPVariable__foreach_tan_(PyObject* self_, PyObject* args, PyObject* kwargs)
11092: {
11093:   HANDLE_TH_ERRORS
11094:   static PythonArgParser parser({
11095:     "_foreach_tan_(TensorList self)",
11096:   }, /*traceable=*/false);
11097: 
11098:   ParsedArgs<1> parsed_args;
11099:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11100:   if(_r.has_torch_function()) {
11101:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11102:   }
11103:   // aten::_foreach_tan_(Tensor(a!)[] self) -> ()
11104: 
11105:   auto dispatch__foreach_tan_ = [](at::TensorList self) -> void {
11106:     pybind11::gil_scoped_release no_gil;
11107:     at::_foreach_tan_(self);
11108:   };
11109:   dispatch__foreach_tan_(_r.tensorlist(0));
11110:   PyObject* self_tensorlist = _r.args[0];
11111:   Py_INCREF(self_tensorlist);
11112:   return self_tensorlist;
11113:   Py_RETURN_NONE;
11114:   END_HANDLE_TH_ERRORS
11115: }
11116: 
11117: // _foreach_copy_
11118: static PyObject * THPVariable__foreach_copy_(PyObject* self_, PyObject* args, PyObject* kwargs)
11119: {
11120:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_foreach_sinh_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_foreach_sinh_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11121-11200

```cpp
11121:   static PythonArgParser parser({
11122:     "_foreach_copy_(TensorList self, TensorList src, bool non_blocking=False)",
11123:   }, /*traceable=*/false);
11124: 
11125:   ParsedArgs<3> parsed_args;
11126:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11127:   if(_r.has_torch_function()) {
11128:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11129:   }
11130:   // aten::_foreach_copy_(Tensor(a!)[] self, Tensor[] src, bool non_blocking=False) -> ()
11131: 
11132:   auto dispatch__foreach_copy_ = [](at::TensorList self, at::TensorList src, bool non_blocking) -> void {
11133:     pybind11::gil_scoped_release no_gil;
11134:     at::_foreach_copy_(self, src, non_blocking);
11135:   };
11136:   dispatch__foreach_copy_(_r.tensorlist(0), _r.tensorlist(1), _r.toBool(2));
11137:   PyObject* self_tensorlist = _r.args[0];
11138:   Py_INCREF(self_tensorlist);
11139:   return self_tensorlist;
11140:   Py_RETURN_NONE;
11141:   END_HANDLE_TH_ERRORS
11142: }
11143: 
11144: \
11145: // bucketize
11146: static PyObject * THPVariable_bucketize(PyObject* self_, PyObject* args, PyObject* kwargs)
11147: {
11148:   HANDLE_TH_ERRORS
11149:   static PythonArgParser parser({
11150:     "bucketize(Tensor input, Tensor boundaries, *, bool out_int32=False, bool right=False, Tensor out=None)",
11151:     "bucketize(Scalar self, Tensor boundaries, *, bool out_int32=False, bool right=False)",
11152:   }, /*traceable=*/true);
11153: 
11154:   ParsedArgs<5> parsed_args;
11155:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11156:   if(_r.has_torch_function()) {
11157:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11158:   }
11159:   switch (_r.idx) {
11160:     case 0: {
11161:       if (_r.isNone(4)) {
11162:         // aten::bucketize.Tensor(Tensor self, Tensor boundaries, *, bool out_int32=False, bool right=False) -> Tensor
11163: 
11164:         auto dispatch_bucketize = [](const at::Tensor & self, const at::Tensor & boundaries, bool out_int32, bool right) -> at::Tensor {
11165:           pybind11::gil_scoped_release no_gil;
11166:           return at::bucketize(self, boundaries, out_int32, right);
11167:         };
11168:         return wrap(dispatch_bucketize(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
11169:       } else {
11170:         // aten::bucketize.Tensor_out(Tensor self, Tensor boundaries, *, bool out_int32=False, bool right=False, Tensor(a!) out) -> Tensor(a!)
11171: 
11172:         auto dispatch_bucketize_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & boundaries, bool out_int32, bool right) -> at::Tensor {
11173:           pybind11::gil_scoped_release no_gil;
11174:           return at::bucketize_out(out, self, boundaries, out_int32, right);
11175:         };
11176:         return wrap(dispatch_bucketize_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
11177:       }
11178:     }
11179:     case 1: {
11180:       // aten::bucketize.Scalar(Scalar self, Tensor boundaries, *, bool out_int32=False, bool right=False) -> Tensor
11181: 
11182:       auto dispatch_bucketize = [](const at::Scalar & self, const at::Tensor & boundaries, bool out_int32, bool right) -> at::Tensor {
11183:         pybind11::gil_scoped_release no_gil;
11184:         return at::bucketize(self, boundaries, out_int32, right);
11185:       };
11186:       return wrap(dispatch_bucketize(_r.scalar(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
11187:     }
11188:   }
11189:   Py_RETURN_NONE;
11190:   END_HANDLE_TH_ERRORS
11191: }
11192: 
11193: \
11194: // searchsorted
11195: static PyObject * THPVariable_searchsorted(PyObject* self_, PyObject* args, PyObject* kwargs)
11196: {
11197:   HANDLE_TH_ERRORS
11198:   static PythonArgParser parser({
11199:     "searchsorted(Tensor sorted_sequence, Tensor input, *, bool out_int32=False, bool right=False, c10::string_view? side=None, Tensor? sorter=None, Tensor out=None)",
11200:     "searchsorted(Tensor sorted_sequence, Scalar self, *, bool out_int32=False, bool right=False, c10::string_view? side=None, Tensor? sorter=None, Tensor out=None)",
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_foreach_copy_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_foreach_copy_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11201-11280

```cpp
11201:   }, /*traceable=*/true);
11202: 
11203:   ParsedArgs<7> parsed_args;
11204:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11205:   if(_r.has_torch_function()) {
11206:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11207:   }
11208:   switch (_r.idx) {
11209:     case 0: {
11210:       if (_r.isNone(6)) {
11211:         // aten::searchsorted.Tensor(Tensor sorted_sequence, Tensor self, *, bool out_int32=False, bool right=False, str? side=None, Tensor? sorter=None) -> Tensor
11212: 
11213:         auto dispatch_searchsorted = [](const at::Tensor & sorted_sequence, const at::Tensor & self, bool out_int32, bool right, ::std::optional<c10::string_view> side, const ::std::optional<at::Tensor> & sorter) -> at::Tensor {
11214:           pybind11::gil_scoped_release no_gil;
11215:           return at::searchsorted(sorted_sequence, self, out_int32, right, side, sorter);
11216:         };
11217:         return wrap(dispatch_searchsorted(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3), _r.stringViewOptional(4), _r.optionalTensor(5)));
11218:       } else {
11219:         // aten::searchsorted.Tensor_out(Tensor sorted_sequence, Tensor self, *, bool out_int32=False, bool right=False, str? side=None, Tensor? sorter=None, Tensor(a!) out) -> Tensor(a!)
11220: 
11221:         auto dispatch_searchsorted_out = [](at::Tensor out, const at::Tensor & sorted_sequence, const at::Tensor & self, bool out_int32, bool right, ::std::optional<c10::string_view> side, const ::std::optional<at::Tensor> & sorter) -> at::Tensor {
11222:           pybind11::gil_scoped_release no_gil;
11223:           return at::searchsorted_out(out, sorted_sequence, self, out_int32, right, side, sorter);
11224:         };
11225:         return wrap(dispatch_searchsorted_out(_r.tensor(6), _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3), _r.stringViewOptional(4), _r.optionalTensor(5)));
11226:       }
11227:     }
11228:     case 1: {
11229:       if (_r.isNone(6)) {
11230:         // aten::searchsorted.Scalar(Tensor sorted_sequence, Scalar self, *, bool out_int32=False, bool right=False, str? side=None, Tensor? sorter=None) -> Tensor
11231: 
11232:         auto dispatch_searchsorted = [](const at::Tensor & sorted_sequence, const at::Scalar & self, bool out_int32, bool right, ::std::optional<c10::string_view> side, const ::std::optional<at::Tensor> & sorter) -> at::Tensor {
11233:           pybind11::gil_scoped_release no_gil;
11234:           return at::searchsorted(sorted_sequence, self, out_int32, right, side, sorter);
11235:         };
11236:         return wrap(dispatch_searchsorted(_r.tensor(0), _r.scalar(1), _r.toBool(2), _r.toBool(3), _r.stringViewOptional(4), _r.optionalTensor(5)));
11237:       } else {
11238:         // aten::searchsorted.Scalar_out(Tensor sorted_sequence, Scalar self, *, bool out_int32=False, bool right=False, str? side=None, Tensor? sorter=None, Tensor(a!) out) -> Tensor(a!)
11239: 
11240:         auto dispatch_searchsorted_out = [](at::Tensor out, const at::Tensor & sorted_sequence, const at::Scalar & self, bool out_int32, bool right, ::std::optional<c10::string_view> side, const ::std::optional<at::Tensor> & sorter) -> at::Tensor {
11241:           pybind11::gil_scoped_release no_gil;
11242:           return at::searchsorted_out(out, sorted_sequence, self, out_int32, right, side, sorter);
11243:         };
11244:         return wrap(dispatch_searchsorted_out(_r.tensor(6), _r.tensor(0), _r.scalar(1), _r.toBool(2), _r.toBool(3), _r.stringViewOptional(4), _r.optionalTensor(5)));
11245:       }
11246:     }
11247:   }
11248:   Py_RETURN_NONE;
11249:   END_HANDLE_TH_ERRORS
11250: }
11251: 
11252: // _convert_indices_from_coo_to_csr
11253: static PyObject * THPVariable__convert_indices_from_coo_to_csr(PyObject* self_, PyObject* args, PyObject* kwargs)
11254: {
11255:   HANDLE_TH_ERRORS
11256:   static PythonArgParser parser({
11257:     "_convert_indices_from_coo_to_csr(Tensor input, int64_t size, *, bool out_int32=False, Tensor out=None)",
11258:   }, /*traceable=*/true);
11259: 
11260:   ParsedArgs<4> parsed_args;
11261:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11262:   if(_r.has_torch_function()) {
11263:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11264:   }
11265:   if (_r.isNone(3)) {
11266:     // aten::_convert_indices_from_coo_to_csr(Tensor self, int size, *, bool out_int32=False) -> Tensor
11267: 
11268:     auto dispatch__convert_indices_from_coo_to_csr = [](const at::Tensor & self, int64_t size, bool out_int32) -> at::Tensor {
11269:       pybind11::gil_scoped_release no_gil;
11270:       return at::_convert_indices_from_coo_to_csr(self, size, out_int32);
11271:     };
11272:     return wrap(dispatch__convert_indices_from_coo_to_csr(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
11273:   } else {
11274:     // aten::_convert_indices_from_coo_to_csr.out(Tensor self, int size, *, bool out_int32=False, Tensor(a!) out) -> Tensor(a!)
11275: 
11276:     auto dispatch__convert_indices_from_coo_to_csr_out = [](at::Tensor out, const at::Tensor & self, int64_t size, bool out_int32) -> at::Tensor {
11277:       pybind11::gil_scoped_release no_gil;
11278:       return at::_convert_indices_from_coo_to_csr_out(out, self, size, out_int32);
11279:     };
11280:     return wrap(dispatch__convert_indices_from_coo_to_csr_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `searchsorted`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `searchsorted`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11281-11360

```cpp
11281:   }
11282:   Py_RETURN_NONE;
11283:   END_HANDLE_TH_ERRORS
11284: }
11285: 
11286: // isfinite
11287: static PyObject * THPVariable_isfinite(PyObject* self_, PyObject* args, PyObject* kwargs)
11288: {
11289:   HANDLE_TH_ERRORS
11290:   static PythonArgParser parser({
11291:     "isfinite(Tensor input)",
11292:   }, /*traceable=*/true);
11293: 
11294:   ParsedArgs<1> parsed_args;
11295:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11296:   if(_r.has_torch_function()) {
11297:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11298:   }
11299:   // aten::isfinite(Tensor self) -> Tensor
11300: 
11301:   auto dispatch_isfinite = [](const at::Tensor & self) -> at::Tensor {
11302:     pybind11::gil_scoped_release no_gil;
11303:     return self.isfinite();
11304:   };
11305:   return wrap(dispatch_isfinite(_r.tensor(0)));
11306:   Py_RETURN_NONE;
11307:   END_HANDLE_TH_ERRORS
11308: }
11309: 
11310: // _remove_batch_dim
11311: static PyObject * THPVariable__remove_batch_dim(PyObject* self_, PyObject* args, PyObject* kwargs)
11312: {
11313:   HANDLE_TH_ERRORS
11314:   static PythonArgParser parser({
11315:     "_remove_batch_dim(Tensor input, int64_t level, SymInt batch_size, int64_t out_dim)",
11316:   }, /*traceable=*/true);
11317: 
11318:   ParsedArgs<4> parsed_args;
11319:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11320:   if(_r.has_torch_function()) {
11321:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11322:   }
11323:   // aten::_remove_batch_dim(Tensor self, int level, SymInt batch_size, int out_dim) -> Tensor
11324: 
11325:   auto dispatch__remove_batch_dim = [](const at::Tensor & self, int64_t level, c10::SymInt batch_size, int64_t out_dim) -> at::Tensor {
11326:     pybind11::gil_scoped_release no_gil;
11327:     return at::_remove_batch_dim_symint(self, level, batch_size, out_dim);
11328:   };
11329:   return wrap(dispatch__remove_batch_dim(_r.tensor(0), _r.toInt64(1), _r.toSymInt(2), _r.toInt64(3)));
11330:   Py_RETURN_NONE;
11331:   END_HANDLE_TH_ERRORS
11332: }
11333: 
11334: // _linalg_eigh
11335: static PyObject * THPVariable__linalg_eigh(PyObject* self_, PyObject* args, PyObject* kwargs)
11336: {
11337:   HANDLE_TH_ERRORS
11338:   static PyTypeObject* NamedTuple = generated::get__linalg_eigh_structseq();
11339:   static PyTypeObject* NamedTuple1 = generated::get__linalg_eigh_out_structseq();
11340:   static PythonArgParser parser({
11341:     "_linalg_eigh(Tensor A, c10::string_view UPLO=\"L\", bool compute_v=True, *, TensorList[2] out=None)",
11342:   }, /*traceable=*/true);
11343: 
11344:   ParsedArgs<4> parsed_args;
11345:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11346:   if(_r.has_torch_function()) {
11347:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11348:   }
11349:   if (_r.isNone(3)) {
11350:     // aten::_linalg_eigh(Tensor A, str UPLO="L", bool compute_v=True) -> (Tensor eigenvalues, Tensor eigenvectors)
11351: 
11352:     auto dispatch__linalg_eigh = [](const at::Tensor & A, c10::string_view UPLO, bool compute_v) -> ::std::tuple<at::Tensor,at::Tensor> {
11353:       pybind11::gil_scoped_release no_gil;
11354:       return at::_linalg_eigh(A, UPLO, compute_v);
11355:     };
11356:     return wrap(NamedTuple, dispatch__linalg_eigh(_r.tensor(0), _r.stringView(1), _r.toBool(2)));
11357:   } else {
11358:     // aten::_linalg_eigh.eigenvalues(Tensor A, str UPLO="L", bool compute_v=True, *, Tensor(a!) eigenvalues, Tensor(b!) eigenvectors) -> (Tensor(a!) eigenvalues, Tensor(b!) eigenvectors)
11359:     auto out = _r.tensorlist_n<2>(3);
11360:     auto dispatch__linalg_eigh_out = [](at::Tensor & eigenvalues, at::Tensor & eigenvectors, const at::Tensor & A, c10::string_view UPLO, bool compute_v) -> ::std::tuple<at::Tensor,at::Tensor> {
```

- EN: The main execution path in this span is carried by `THPVariable_isfinite`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_isfinite`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11361-11440

```cpp
11361:       pybind11::gil_scoped_release no_gil;
11362:       return at::_linalg_eigh_out(eigenvalues, eigenvectors, A, UPLO, compute_v);
11363:     };
11364:     return wrap(NamedTuple1, dispatch__linalg_eigh_out(out[0], out[1], _r.tensor(0), _r.stringView(1), _r.toBool(2)));
11365:   }
11366:   Py_RETURN_NONE;
11367:   END_HANDLE_TH_ERRORS
11368: }
11369: 
11370: // outer
11371: static PyObject * THPVariable_outer(PyObject* self_, PyObject* args, PyObject* kwargs)
11372: {
11373:   HANDLE_TH_ERRORS
11374:   static PythonArgParser parser({
11375:     "outer(Tensor input, Tensor vec2, *, Tensor out=None)",
11376:   }, /*traceable=*/true);
11377: 
11378:   ParsedArgs<3> parsed_args;
11379:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11380:   if(_r.has_torch_function()) {
11381:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11382:   }
11383:   if (_r.isNone(2)) {
11384:     // aten::outer(Tensor self, Tensor vec2) -> Tensor
11385: 
11386:     auto dispatch_outer = [](const at::Tensor & self, const at::Tensor & vec2) -> at::Tensor {
11387:       pybind11::gil_scoped_release no_gil;
11388:       return self.outer(vec2);
11389:     };
11390:     return wrap(dispatch_outer(_r.tensor(0), _r.tensor(1)));
11391:   } else {
11392:     // aten::outer.out(Tensor self, Tensor vec2, *, Tensor(a!) out) -> Tensor(a!)
11393: 
11394:     auto dispatch_outer_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & vec2) -> at::Tensor {
11395:       pybind11::gil_scoped_release no_gil;
11396:       return at::outer_out(out, self, vec2);
11397:     };
11398:     return wrap(dispatch_outer_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
11399:   }
11400:   Py_RETURN_NONE;
11401:   END_HANDLE_TH_ERRORS
11402: }
11403: 
11404: // _test_serialization_subcmul
11405: static PyObject * THPVariable__test_serialization_subcmul(PyObject* self_, PyObject* args, PyObject* kwargs)
11406: {
11407:   HANDLE_TH_ERRORS
11408:   static PythonArgParser parser({
11409:     "_test_serialization_subcmul(Tensor input, Tensor other, Scalar alpha=1)",
11410:   }, /*traceable=*/true);
11411: 
11412:   ParsedArgs<3> parsed_args;
11413:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11414:   if(_r.has_torch_function()) {
11415:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11416:   }
11417:   // aten::_test_serialization_subcmul(Tensor self, Tensor other, Scalar alpha=1) -> Tensor
11418: 
11419:   auto dispatch__test_serialization_subcmul = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
11420:     pybind11::gil_scoped_release no_gil;
11421:     return at::_test_serialization_subcmul(self, other, alpha);
11422:   };
11423:   return wrap(dispatch__test_serialization_subcmul(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
11424:   Py_RETURN_NONE;
11425:   END_HANDLE_TH_ERRORS
11426: }
11427: 
11428: // _test_autograd_multiple_dispatch_view
11429: static PyObject * THPVariable__test_autograd_multiple_dispatch_view(PyObject* self_, PyObject* args, PyObject* kwargs)
11430: {
11431:   HANDLE_TH_ERRORS
11432:   static PythonArgParser parser({
11433:     "_test_autograd_multiple_dispatch_view(Tensor input)",
11434:   }, /*traceable=*/true);
11435: 
11436:   ParsedArgs<1> parsed_args;
11437:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11438:   if(_r.has_torch_function()) {
11439:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11440:   }
```

- EN: The main execution path in this span is carried by `_linalg_eigh_out`, `wrap`, `THPVariable_outer`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_linalg_eigh_out`, `wrap`, `THPVariable_outer` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11441-11520

```cpp
11441:   // aten::_test_autograd_multiple_dispatch_view(Tensor(a) self) -> Tensor(a)
11442: 
11443:   auto dispatch__test_autograd_multiple_dispatch_view = [](const at::Tensor & self) -> at::Tensor {
11444:     pybind11::gil_scoped_release no_gil;
11445:     return at::_test_autograd_multiple_dispatch_view(self);
11446:   };
11447:   return wrap(dispatch__test_autograd_multiple_dispatch_view(_r.tensor(0)));
11448:   Py_RETURN_NONE;
11449:   END_HANDLE_TH_ERRORS
11450: }
11451: 
11452: // view_as_real_copy
11453: static PyObject * THPVariable_view_as_real_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11454: {
11455:   HANDLE_TH_ERRORS
11456:   static PythonArgParser parser({
11457:     "view_as_real_copy(Tensor input, *, Tensor out=None)",
11458:   }, /*traceable=*/true);
11459: 
11460:   ParsedArgs<2> parsed_args;
11461:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11462:   if(_r.has_torch_function()) {
11463:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11464:   }
11465:   if (_r.isNone(1)) {
11466:     // aten::view_as_real_copy(Tensor self) -> Tensor
11467: 
11468:     auto dispatch_view_as_real_copy = [](const at::Tensor & self) -> at::Tensor {
11469:       pybind11::gil_scoped_release no_gil;
11470:       return at::view_as_real_copy(self);
11471:     };
11472:     return wrap(dispatch_view_as_real_copy(_r.tensor(0)));
11473:   } else {
11474:     // aten::view_as_real_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11475: 
11476:     auto dispatch_view_as_real_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11477:       pybind11::gil_scoped_release no_gil;
11478:       return at::view_as_real_copy_out(out, self);
11479:     };
11480:     return wrap(dispatch_view_as_real_copy_out(_r.tensor(1), _r.tensor(0)));
11481:   }
11482:   Py_RETURN_NONE;
11483:   END_HANDLE_TH_ERRORS
11484: }
11485: 
11486: // permute_copy
11487: static PyObject * THPVariable_permute_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11488: {
11489:   HANDLE_TH_ERRORS
11490:   static PythonArgParser parser({
11491:     "permute_copy(Tensor input, IntArrayRef dims, *, Tensor out=None)",
11492:   }, /*traceable=*/true);
11493: 
11494:   ParsedArgs<3> parsed_args;
11495:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11496:   if(_r.has_torch_function()) {
11497:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11498:   }
11499:   if (_r.isNone(2)) {
11500:     // aten::permute_copy(Tensor self, int[] dims) -> Tensor
11501: 
11502:     auto dispatch_permute_copy = [](const at::Tensor & self, at::IntArrayRef dims) -> at::Tensor {
11503:       pybind11::gil_scoped_release no_gil;
11504:       return at::permute_copy(self, dims);
11505:     };
11506:     return wrap(dispatch_permute_copy(_r.tensor(0), _r.intlist(1)));
11507:   } else {
11508:     // aten::permute_copy.out(Tensor self, int[] dims, *, Tensor(a!) out) -> Tensor(a!)
11509: 
11510:     auto dispatch_permute_copy_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dims) -> at::Tensor {
11511:       pybind11::gil_scoped_release no_gil;
11512:       return at::permute_copy_out(out, self, dims);
11513:     };
11514:     return wrap(dispatch_permute_copy_out(_r.tensor(2), _r.tensor(0), _r.intlist(1)));
11515:   }
11516:   Py_RETURN_NONE;
11517:   END_HANDLE_TH_ERRORS
11518: }
11519: 
11520: // detach_copy
```

- EN: The main execution path in this span is carried by `_test_autograd_multiple_dispatch_view`, `wrap`, `THPVariable_view_as_real_copy`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_test_autograd_multiple_dispatch_view`, `wrap`, `THPVariable_view_as_real_copy` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11521-11600

```cpp
11521: static PyObject * THPVariable_detach_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11522: {
11523:   HANDLE_TH_ERRORS
11524:   static PythonArgParser parser({
11525:     "detach_copy(Tensor input, *, Tensor out=None)",
11526:   }, /*traceable=*/true);
11527: 
11528:   ParsedArgs<2> parsed_args;
11529:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11530:   if(_r.has_torch_function()) {
11531:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11532:   }
11533:   if (_r.isNone(1)) {
11534:     // aten::detach_copy(Tensor self) -> Tensor
11535: 
11536:     auto dispatch_detach_copy = [](const at::Tensor & self) -> at::Tensor {
11537:       pybind11::gil_scoped_release no_gil;
11538:       return at::detach_copy(self);
11539:     };
11540:     return wrap(dispatch_detach_copy(_r.tensor(0)));
11541:   } else {
11542:     // aten::detach_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11543: 
11544:     auto dispatch_detach_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11545:       pybind11::gil_scoped_release no_gil;
11546:       return at::detach_copy_out(out, self);
11547:     };
11548:     return wrap(dispatch_detach_copy_out(_r.tensor(1), _r.tensor(0)));
11549:   }
11550:   Py_RETURN_NONE;
11551:   END_HANDLE_TH_ERRORS
11552: }
11553: 
11554: // split_copy
11555: static PyObject * THPVariable_split_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11556: {
11557:   HANDLE_TH_ERRORS
11558:   static PythonArgParser parser({
11559:     "split_copy(Tensor input, SymInt split_size, int64_t dim=0, *, TensorList out=None)",
11560:   }, /*traceable=*/false);
11561: 
11562:   ParsedArgs<4> parsed_args;
11563:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11564:   if(_r.has_torch_function()) {
11565:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11566:   }
11567:   if (_r.isNone(3)) {
11568:     // aten::split_copy.Tensor(Tensor self, SymInt split_size, int dim=0) -> Tensor[]
11569: 
11570:     auto dispatch_split_copy = [](const at::Tensor & self, c10::SymInt split_size, int64_t dim) -> ::std::vector<at::Tensor> {
11571:       pybind11::gil_scoped_release no_gil;
11572:       return at::split_copy_symint(self, split_size, dim);
11573:     };
11574:     return wrap(dispatch_split_copy(_r.tensor(0), _r.toSymInt(1), _r.toInt64(2)));
11575:   } else {
11576:     // aten::split_copy.Tensor_out(Tensor self, SymInt split_size, int dim=0, *, Tensor(a!)[] out) -> ()
11577: 
11578:     auto dispatch_split_copy_out = [](at::TensorList out, const at::Tensor & self, c10::SymInt split_size, int64_t dim) -> void {
11579:       pybind11::gil_scoped_release no_gil;
11580:       at::split_copy_symint_out(out, self, split_size, dim);
11581:     };
11582:     dispatch_split_copy_out(_r.tensorlist(3), _r.tensor(0), _r.toSymInt(1), _r.toInt64(2));
11583:     Py_RETURN_NONE;
11584:   }
11585:   Py_RETURN_NONE;
11586:   END_HANDLE_TH_ERRORS
11587: }
11588: 
11589: // unsqueeze_copy
11590: static PyObject * THPVariable_unsqueeze_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11591: {
11592:   HANDLE_TH_ERRORS
11593:   static PythonArgParser parser({
11594:     "unsqueeze_copy(Tensor input, int64_t dim, *, Tensor out=None)",
11595:   }, /*traceable=*/true);
11596: 
11597:   ParsedArgs<3> parsed_args;
11598:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11599:   if(_r.has_torch_function()) {
11600:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `THPVariable_detach_copy`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_detach_copy`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11601-11680

```cpp
11601:   }
11602:   if (_r.isNone(2)) {
11603:     // aten::unsqueeze_copy(Tensor self, int dim) -> Tensor
11604: 
11605:     auto dispatch_unsqueeze_copy = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
11606:       pybind11::gil_scoped_release no_gil;
11607:       return at::unsqueeze_copy(self, dim);
11608:     };
11609:     return wrap(dispatch_unsqueeze_copy(_r.tensor(0), _r.toInt64(1)));
11610:   } else {
11611:     // aten::unsqueeze_copy.out(Tensor self, int dim, *, Tensor(a!) out) -> Tensor(a!)
11612: 
11613:     auto dispatch_unsqueeze_copy_out = [](at::Tensor out, const at::Tensor & self, int64_t dim) -> at::Tensor {
11614:       pybind11::gil_scoped_release no_gil;
11615:       return at::unsqueeze_copy_out(out, self, dim);
11616:     };
11617:     return wrap(dispatch_unsqueeze_copy_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
11618:   }
11619:   Py_RETURN_NONE;
11620:   END_HANDLE_TH_ERRORS
11621: }
11622: 
11623: // _indices_copy
11624: static PyObject * THPVariable__indices_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11625: {
11626:   HANDLE_TH_ERRORS
11627:   static PythonArgParser parser({
11628:     "_indices_copy(Tensor input, *, Tensor out=None)",
11629:   }, /*traceable=*/true);
11630: 
11631:   ParsedArgs<2> parsed_args;
11632:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11633:   if(_r.has_torch_function()) {
11634:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11635:   }
11636:   if (_r.isNone(1)) {
11637:     // aten::_indices_copy(Tensor self) -> Tensor
11638: 
11639:     auto dispatch__indices_copy = [](const at::Tensor & self) -> at::Tensor {
11640:       pybind11::gil_scoped_release no_gil;
11641:       return at::_indices_copy(self);
11642:     };
11643:     return wrap(dispatch__indices_copy(_r.tensor(0)));
11644:   } else {
11645:     // aten::_indices_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11646: 
11647:     auto dispatch__indices_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11648:       pybind11::gil_scoped_release no_gil;
11649:       return at::_indices_copy_out(out, self);
11650:     };
11651:     return wrap(dispatch__indices_copy_out(_r.tensor(1), _r.tensor(0)));
11652:   }
11653:   Py_RETURN_NONE;
11654:   END_HANDLE_TH_ERRORS
11655: }
11656: 
11657: // _values_copy
11658: static PyObject * THPVariable__values_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11659: {
11660:   HANDLE_TH_ERRORS
11661:   static PythonArgParser parser({
11662:     "_values_copy(Tensor input, *, Tensor out=None)",
11663:   }, /*traceable=*/true);
11664: 
11665:   ParsedArgs<2> parsed_args;
11666:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11667:   if(_r.has_torch_function()) {
11668:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11669:   }
11670:   if (_r.isNone(1)) {
11671:     // aten::_values_copy(Tensor self) -> Tensor
11672: 
11673:     auto dispatch__values_copy = [](const at::Tensor & self) -> at::Tensor {
11674:       pybind11::gil_scoped_release no_gil;
11675:       return at::_values_copy(self);
11676:     };
11677:     return wrap(dispatch__values_copy(_r.tensor(0)));
11678:   } else {
11679:     // aten::_values_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11680: 
```

- EN: The main execution path in this span is carried by `unsqueeze_copy`, `wrap`, `unsqueeze_copy_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `unsqueeze_copy`, `wrap`, `unsqueeze_copy_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11681-11760

```cpp
11681:     auto dispatch__values_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11682:       pybind11::gil_scoped_release no_gil;
11683:       return at::_values_copy_out(out, self);
11684:     };
11685:     return wrap(dispatch__values_copy_out(_r.tensor(1), _r.tensor(0)));
11686:   }
11687:   Py_RETURN_NONE;
11688:   END_HANDLE_TH_ERRORS
11689: }
11690: 
11691: // unbind_copy
11692: static PyObject * THPVariable_unbind_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11693: {
11694:   HANDLE_TH_ERRORS
11695:   static PythonArgParser parser({
11696:     "unbind_copy(Tensor input, int64_t dim=0, *, TensorList out=None)",
11697:   }, /*traceable=*/false);
11698: 
11699:   ParsedArgs<3> parsed_args;
11700:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11701:   if(_r.has_torch_function()) {
11702:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11703:   }
11704:   if (_r.isNone(2)) {
11705:     // aten::unbind_copy.int(Tensor self, int dim=0) -> Tensor[]
11706: 
11707:     auto dispatch_unbind_copy = [](const at::Tensor & self, int64_t dim) -> ::std::vector<at::Tensor> {
11708:       pybind11::gil_scoped_release no_gil;
11709:       return at::unbind_copy(self, dim);
11710:     };
11711:     return wrap(dispatch_unbind_copy(_r.tensor(0), _r.toInt64(1)));
11712:   } else {
11713:     // aten::unbind_copy.int_out(Tensor self, int dim=0, *, Tensor(a!)[] out) -> ()
11714: 
11715:     auto dispatch_unbind_copy_out = [](at::TensorList out, const at::Tensor & self, int64_t dim) -> void {
11716:       pybind11::gil_scoped_release no_gil;
11717:       at::unbind_copy_out(out, self, dim);
11718:     };
11719:     dispatch_unbind_copy_out(_r.tensorlist(2), _r.tensor(0), _r.toInt64(1));
11720:     Py_RETURN_NONE;
11721:   }
11722:   Py_RETURN_NONE;
11723:   END_HANDLE_TH_ERRORS
11724: }
11725: 
11726: // alias_copy
11727: static PyObject * THPVariable_alias_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11728: {
11729:   HANDLE_TH_ERRORS
11730:   static PythonArgParser parser({
11731:     "alias_copy(Tensor input, *, Tensor out=None)",
11732:   }, /*traceable=*/true);
11733: 
11734:   ParsedArgs<2> parsed_args;
11735:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11736:   if(_r.has_torch_function()) {
11737:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11738:   }
11739:   if (_r.isNone(1)) {
11740:     // aten::alias_copy(Tensor self) -> Tensor
11741: 
11742:     auto dispatch_alias_copy = [](const at::Tensor & self) -> at::Tensor {
11743:       pybind11::gil_scoped_release no_gil;
11744:       return at::alias_copy(self);
11745:     };
11746:     return wrap(dispatch_alias_copy(_r.tensor(0)));
11747:   } else {
11748:     // aten::alias_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11749: 
11750:     auto dispatch_alias_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11751:       pybind11::gil_scoped_release no_gil;
11752:       return at::alias_copy_out(out, self);
11753:     };
11754:     return wrap(dispatch_alias_copy_out(_r.tensor(1), _r.tensor(0)));
11755:   }
11756:   Py_RETURN_NONE;
11757:   END_HANDLE_TH_ERRORS
11758: }
11759: 
11760: // _nested_tensor_softmax_with_shape
```

- EN: The main execution path in this span is carried by `_values_copy_out`, `wrap`, `THPVariable_unbind_copy`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_values_copy_out`, `wrap`, `THPVariable_unbind_copy` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11761-11840

```cpp
11761: static PyObject * THPVariable__nested_tensor_softmax_with_shape(PyObject* self_, PyObject* args, PyObject* kwargs)
11762: {
11763:   HANDLE_TH_ERRORS
11764:   static PythonArgParser parser({
11765:     "_nested_tensor_softmax_with_shape(Tensor input, Tensor query)",
11766:   }, /*traceable=*/true);
11767: 
11768:   ParsedArgs<2> parsed_args;
11769:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11770:   if(_r.has_torch_function()) {
11771:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11772:   }
11773:   // aten::_nested_tensor_softmax_with_shape(Tensor self, Tensor query) -> Tensor
11774: 
11775:   auto dispatch__nested_tensor_softmax_with_shape = [](const at::Tensor & self, const at::Tensor & query) -> at::Tensor {
11776:     pybind11::gil_scoped_release no_gil;
11777:     return at::_nested_tensor_softmax_with_shape(self, query);
11778:   };
11779:   return wrap(dispatch__nested_tensor_softmax_with_shape(_r.tensor(0), _r.tensor(1)));
11780:   Py_RETURN_NONE;
11781:   END_HANDLE_TH_ERRORS
11782: }
11783: 
11784: // _safe_softmax
11785: static PyObject * THPVariable__safe_softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
11786: {
11787:   HANDLE_TH_ERRORS
11788:   static PythonArgParser parser({
11789:     "_safe_softmax(Tensor input, int64_t dim, ScalarType? dtype=None)",
11790:   }, /*traceable=*/true);
11791: 
11792:   ParsedArgs<3> parsed_args;
11793:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11794:   if(_r.has_torch_function()) {
11795:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11796:   }
11797:   // aten::_safe_softmax(Tensor self, int dim, ScalarType? dtype=None) -> Tensor
11798: 
11799:   auto dispatch__safe_softmax = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
11800:     pybind11::gil_scoped_release no_gil;
11801:     return at::_safe_softmax(self, dim, dtype);
11802:   };
11803:   return wrap(dispatch__safe_softmax(_r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
11804:   Py_RETURN_NONE;
11805:   END_HANDLE_TH_ERRORS
11806: }
11807: 
11808: // _native_multi_head_attention
11809: static PyObject * THPVariable__native_multi_head_attention(PyObject* self_, PyObject* args, PyObject* kwargs)
11810: {
11811:   HANDLE_TH_ERRORS
11812:   static PythonArgParser parser({
11813:     "_native_multi_head_attention(Tensor query, Tensor key, Tensor value, int64_t embed_dim, int64_t num_head, Tensor qkv_weight, Tensor qkv_bias, Tensor proj_weight, Tensor proj_bias, Tensor? mask=None, bool need_weights=True, bool average_attn_weights=True, int64_t? mask_type=None)",
11814:   }, /*traceable=*/true);
11815: 
11816:   ParsedArgs<13> parsed_args;
11817:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11818:   if(_r.has_torch_function()) {
11819:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11820:   }
11821:   // aten::_native_multi_head_attention(Tensor query, Tensor key, Tensor value, int embed_dim, int num_head, Tensor qkv_weight, Tensor qkv_bias, Tensor proj_weight, Tensor proj_bias, Tensor? mask=None, bool need_weights=True, bool average_attn_weights=True, int? mask_type=None) -> (Tensor, Tensor)
11822: 
11823:   auto dispatch__native_multi_head_attention = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, int64_t embed_dim, int64_t num_head, const at::Tensor & qkv_weight, const at::Tensor & qkv_bias, const at::Tensor & proj_weight, const at::Tensor & proj_bias, const ::std::optional<at::Tensor> & mask, bool need_weights, bool average_attn_weights, ::std::optional<int64_t> mask_type) -> ::std::tuple<at::Tensor,at::Tensor> {
11824:     pybind11::gil_scoped_release no_gil;
11825:     return at::_native_multi_head_attention(query, key, value, embed_dim, num_head, qkv_weight, qkv_bias, proj_weight, proj_bias, mask, need_weights, average_attn_weights, mask_type);
11826:   };
11827:   return wrap(dispatch__native_multi_head_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3), _r.toInt64(4), _r.tensor(5), _r.tensor(6), _r.tensor(7), _r.tensor(8), _r.optionalTensor(9), _r.toBool(10), _r.toBool(11), _r.toInt64Optional(12)));
11828:   Py_RETURN_NONE;
11829:   END_HANDLE_TH_ERRORS
11830: }
11831: 
11832: // _scaled_dot_product_attention_math_for_mps
11833: static PyObject * THPVariable__scaled_dot_product_attention_math_for_mps(PyObject* self_, PyObject* args, PyObject* kwargs)
11834: {
11835:   HANDLE_TH_ERRORS
11836:   static PythonArgParser parser({
11837:     "_scaled_dot_product_attention_math_for_mps(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, double dropout_p=0.0, bool is_causal=False, Tensor? dropout_mask=None, *, double? scale=None, bool enable_gqa=False)",
11838:   }, /*traceable=*/true);
11839: 
11840:   ParsedArgs<9> parsed_args;
```

- EN: The main execution path in this span is carried by `THPVariable__nested_tensor_softmax_with_shape`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__nested_tensor_softmax_with_shape`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11841-11920

```cpp
11841:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11842:   if(_r.has_torch_function()) {
11843:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11844:   }
11845:   // aten::_scaled_dot_product_attention_math_for_mps(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, float dropout_p=0.0, bool is_causal=False, Tensor? dropout_mask=None, *, float? scale=None, bool enable_gqa=False) -> (Tensor, Tensor)
11846: 
11847:   auto dispatch__scaled_dot_product_attention_math_for_mps = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & attn_mask, double dropout_p, bool is_causal, const ::std::optional<at::Tensor> & dropout_mask, ::std::optional<double> scale, bool enable_gqa) -> ::std::tuple<at::Tensor,at::Tensor> {
11848:     pybind11::gil_scoped_release no_gil;
11849:     return at::_scaled_dot_product_attention_math_for_mps(query, key, value, attn_mask, dropout_p, is_causal, dropout_mask, scale, enable_gqa);
11850:   };
11851:   return wrap(dispatch__scaled_dot_product_attention_math_for_mps(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.toDouble(4), _r.toBool(5), _r.optionalTensor(6), _r.toDoubleOptional(7), _r.toBool(8)));
11852:   Py_RETURN_NONE;
11853:   END_HANDLE_TH_ERRORS
11854: }
11855: 
11856: \
11857: // _scaled_dot_product_flash_attention
11858: static PyObject * THPVariable__scaled_dot_product_flash_attention(PyObject* self_, PyObject* args, PyObject* kwargs)
11859: {
11860:   HANDLE_TH_ERRORS
11861:   static PyTypeObject* NamedTuple = generated::get__scaled_dot_product_flash_attention_structseq();
11862:   static PythonArgParser parser({
11863:     "_scaled_dot_product_flash_attention(Tensor query, Tensor key, Tensor value, Tensor? q_descale, Tensor? k_descale, Tensor? v_descale, double dropout_p=0.0, bool is_causal=False, bool return_debug_mask=False, *, double? scale=None)",
11864:     "_scaled_dot_product_flash_attention(Tensor query, Tensor key, Tensor value, double dropout_p=0.0, bool is_causal=False, bool return_debug_mask=False, *, double? scale=None)",
11865:   }, /*traceable=*/true);
11866: 
11867:   ParsedArgs<10> parsed_args;
11868:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11869:   if(_r.has_torch_function()) {
11870:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11871:   }
11872:   switch (_r.idx) {
11873:     case 0: {
11874:       // aten::_scaled_dot_product_flash_attention.quantized(Tensor query, Tensor key, Tensor value, Tensor? q_descale, Tensor? k_descale, Tensor? v_descale, float dropout_p=0.0, bool is_causal=False, bool return_debug_mask=False, *, float? scale=None) -> (Tensor output, Tensor logsumexp, Tensor cum_seq_q, Tensor cum_seq_k, SymInt max_q, SymInt max_k, Tensor rng_state, Tensor unused, Tensor debug_attn_mask)
11875: 
11876:       auto dispatch__scaled_dot_product_flash_attention = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & q_descale, const ::std::optional<at::Tensor> & k_descale, const ::std::optional<at::Tensor> & v_descale, double dropout_p, bool is_causal, bool return_debug_mask, ::std::optional<double> scale) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,c10::SymInt,c10::SymInt,at::Tensor,at::Tensor,at::Tensor> {
11877:         pybind11::gil_scoped_release no_gil;
11878:         return at::_scaled_dot_product_flash_attention(query, key, value, q_descale, k_descale, v_descale, dropout_p, is_causal, return_debug_mask, scale);
11879:       };
11880:       return wrap(NamedTuple, dispatch__scaled_dot_product_flash_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.optionalTensor(5), _r.toDouble(6), _r.toBool(7), _r.toBool(8), _r.toDoubleOptional(9)));
11881:     }
11882:     case 1: {
11883:       // aten::_scaled_dot_product_flash_attention(Tensor query, Tensor key, Tensor value, float dropout_p=0.0, bool is_causal=False, bool return_debug_mask=False, *, float? scale=None) -> (Tensor output, Tensor logsumexp, Tensor cum_seq_q, Tensor cum_seq_k, SymInt max_q, SymInt max_k, Tensor rng_state, Tensor unused, Tensor debug_attn_mask)
11884: 
11885:       auto dispatch__scaled_dot_product_flash_attention = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, double dropout_p, bool is_causal, bool return_debug_mask, ::std::optional<double> scale) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,c10::SymInt,c10::SymInt,at::Tensor,at::Tensor,at::Tensor> {
11886:         pybind11::gil_scoped_release no_gil;
11887:         return at::_scaled_dot_product_flash_attention(query, key, value, dropout_p, is_causal, return_debug_mask, scale);
11888:       };
11889:       return wrap(NamedTuple, dispatch__scaled_dot_product_flash_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3), _r.toBool(4), _r.toBool(5), _r.toDoubleOptional(6)));
11890:     }
11891:   }
11892:   Py_RETURN_NONE;
11893:   END_HANDLE_TH_ERRORS
11894: }
11895: 
11896: // _scaled_dot_product_flash_attention_for_cpu
11897: static PyObject * THPVariable__scaled_dot_product_flash_attention_for_cpu(PyObject* self_, PyObject* args, PyObject* kwargs)
11898: {
11899:   HANDLE_TH_ERRORS
11900:   static PyTypeObject* NamedTuple = generated::get__scaled_dot_product_flash_attention_for_cpu_structseq();
11901:   static PythonArgParser parser({
11902:     "_scaled_dot_product_flash_attention_for_cpu(Tensor query, Tensor key, Tensor value, double dropout_p=0.0, bool is_causal=False, *, Tensor? attn_mask=None, double? scale=None)",
11903:   }, /*traceable=*/true);
11904: 
11905:   ParsedArgs<7> parsed_args;
11906:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11907:   if(_r.has_torch_function()) {
11908:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11909:   }
11910:   // aten::_scaled_dot_product_flash_attention_for_cpu(Tensor query, Tensor key, Tensor value, float dropout_p=0.0, bool is_causal=False, *, Tensor? attn_mask=None, float? scale=None) -> (Tensor output, Tensor logsumexp)
11911: 
11912:   auto dispatch__scaled_dot_product_flash_attention_for_cpu = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, double dropout_p, bool is_causal, const ::std::optional<at::Tensor> & attn_mask, ::std::optional<double> scale) -> ::std::tuple<at::Tensor,at::Tensor> {
11913:     pybind11::gil_scoped_release no_gil;
11914:     return at::_scaled_dot_product_flash_attention_for_cpu(query, key, value, dropout_p, is_causal, attn_mask, scale);
11915:   };
11916:   return wrap(NamedTuple, dispatch__scaled_dot_product_flash_attention_for_cpu(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3), _r.toBool(4), _r.optionalTensor(5), _r.toDoubleOptional(6)));
11917:   Py_RETURN_NONE;
11918:   END_HANDLE_TH_ERRORS
11919: }
11920: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_scaled_dot_product_attention_math_for_mps`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_scaled_dot_product_attention_math_for_mps`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11921-11986

```cpp
11921: // _triton_scaled_dot_attention
11922: static PyObject * THPVariable__triton_scaled_dot_attention(PyObject* self_, PyObject* args, PyObject* kwargs)
11923: {
11924:   HANDLE_TH_ERRORS
11925:   static PythonArgParser parser({
11926:     "_triton_scaled_dot_attention(Tensor q, Tensor k, Tensor v, double dropout_p=0.0)",
11927:   }, /*traceable=*/true);
11928: 
11929:   ParsedArgs<4> parsed_args;
11930:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11931:   if(_r.has_torch_function()) {
11932:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11933:   }
11934:   // aten::_triton_scaled_dot_attention(Tensor q, Tensor k, Tensor v, float dropout_p=0.0) -> Tensor
11935: 
11936:   auto dispatch__triton_scaled_dot_attention = [](const at::Tensor & q, const at::Tensor & k, const at::Tensor & v, double dropout_p) -> at::Tensor {
11937:     pybind11::gil_scoped_release no_gil;
11938:     return at::_triton_scaled_dot_attention(q, k, v, dropout_p);
11939:   };
11940:   return wrap(dispatch__triton_scaled_dot_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3)));
11941:   Py_RETURN_NONE;
11942:   END_HANDLE_TH_ERRORS
11943: }
11944: 
11945: \
11946: // _fused_adamw_
11947: static PyObject * THPVariable__fused_adamw_(PyObject* self_, PyObject* args, PyObject* kwargs)
11948: {
11949:   HANDLE_TH_ERRORS
11950:   static PythonArgParser parser({
11951:     "_fused_adamw_(TensorList self, TensorList grads, TensorList exp_avgs, TensorList exp_avg_sqs, TensorList max_exp_avg_sqs, TensorList state_steps, *, Tensor lr, double beta1, double beta2, double weight_decay, double eps, bool amsgrad, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None)",
11952:     "_fused_adamw_(TensorList self, TensorList grads, TensorList exp_avgs, TensorList exp_avg_sqs, TensorList max_exp_avg_sqs, TensorList state_steps, *, double lr, double beta1, double beta2, double weight_decay, double eps, bool amsgrad, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None)",
11953:   }, /*traceable=*/false);
11954: 
11955:   ParsedArgs<15> parsed_args;
11956:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11957:   if(_r.has_torch_function()) {
11958:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11959:   }
11960:   switch (_r.idx) {
11961:     case 0: {
11962:       // aten::_fused_adamw_.tensor_lr(Tensor(a!)[] self, Tensor(b!)[] grads, Tensor(c!)[] exp_avgs, Tensor(d!)[] exp_avg_sqs, Tensor(e!)[] max_exp_avg_sqs, Tensor[] state_steps, *, Tensor lr, float beta1, float beta2, float weight_decay, float eps, bool amsgrad, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None) -> ()
11963: 
11964:       auto dispatch__fused_adamw_ = [](at::TensorList self, at::TensorList grads, at::TensorList exp_avgs, at::TensorList exp_avg_sqs, at::TensorList max_exp_avg_sqs, at::TensorList state_steps, const at::Tensor & lr, double beta1, double beta2, double weight_decay, double eps, bool amsgrad, bool maximize, const ::std::optional<at::Tensor> & grad_scale, const ::std::optional<at::Tensor> & found_inf) -> void {
11965:         pybind11::gil_scoped_release no_gil;
11966:         at::_fused_adamw_(self, grads, exp_avgs, exp_avg_sqs, max_exp_avg_sqs, state_steps, lr, beta1, beta2, weight_decay, eps, amsgrad, maximize, grad_scale, found_inf);
11967:       };
11968:       dispatch__fused_adamw_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.tensorlist(3), _r.tensorlist(4), _r.tensorlist(5), _r.tensor(6), _r.toDouble(7), _r.toDouble(8), _r.toDouble(9), _r.toDouble(10), _r.toBool(11), _r.toBool(12), _r.optionalTensor(13), _r.optionalTensor(14));
11969:       Py_RETURN_NONE;
11970:     }
11971:     case 1: {
11972:       // aten::_fused_adamw_(Tensor(a!)[] self, Tensor(b!)[] grads, Tensor(c!)[] exp_avgs, Tensor(d!)[] exp_avg_sqs, Tensor(e!)[] max_exp_avg_sqs, Tensor[] state_steps, *, float lr, float beta1, float beta2, float weight_decay, float eps, bool amsgrad, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None) -> ()
11973: 
11974:       auto dispatch__fused_adamw_ = [](at::TensorList self, at::TensorList grads, at::TensorList exp_avgs, at::TensorList exp_avg_sqs, at::TensorList max_exp_avg_sqs, at::TensorList state_steps, double lr, double beta1, double beta2, double weight_decay, double eps, bool amsgrad, bool maximize, const ::std::optional<at::Tensor> & grad_scale, const ::std::optional<at::Tensor> & found_inf) -> void {
11975:         pybind11::gil_scoped_release no_gil;
11976:         at::_fused_adamw_(self, grads, exp_avgs, exp_avg_sqs, max_exp_avg_sqs, state_steps, lr, beta1, beta2, weight_decay, eps, amsgrad, maximize, grad_scale, found_inf);
11977:       };
11978:       dispatch__fused_adamw_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.tensorlist(3), _r.tensorlist(4), _r.tensorlist(5), _r.toDouble(6), _r.toDouble(7), _r.toDouble(8), _r.toDouble(9), _r.toDouble(10), _r.toBool(11), _r.toBool(12), _r.optionalTensor(13), _r.optionalTensor(14));
11979:       Py_RETURN_NONE;
11980:     }
11981:   }
11982:   Py_RETURN_NONE;
11983:   END_HANDLE_TH_ERRORS
11984: }
11985: 
11986: } // namespace torch::autograd
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable__triton_scaled_dot_attention`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable__triton_scaled_dot_attention`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `THPVariable__cast_Double` / 核心符号 `THPVariable__cast_Double`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `Python.h`, `torch/csrc/autograd/python_torch_functions.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/utils/wrap_outputs.h`, `torch/csrc/Dtype.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, `torch/csrc/utils/out_types.h`, `torch/csrc/utils/pybind.h`, `torch/csrc/utils/pycfunction_helpers.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `THPVariable__cast_Double`, `THPVariable__cast_Int`, `THPVariable_align_tensors`, `THPVariable__print`, `THPVariable__make_dep_token`, `THPVariable__use_cudnn_rnn_flatten_weight`, `THPVariable__cudnn_rnn`, `THPVariable__fused_dropout`, `THPVariable__sobol_engine_ff_`, `THPVariable__sobol_engine_scramble_`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层

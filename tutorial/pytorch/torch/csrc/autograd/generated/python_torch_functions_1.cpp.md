# python_torch_functions_1.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_torch_functions_1.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Defines generated backward-function classes and helpers used by the autograd engine.
- 目的 (CN): 定义自动求导引擎使用的生成式反向函数类与辅助逻辑。
- Lines: 11549
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
42: #include <ATen/ops/_cast_Char.h>
43: #include <ATen/ops/_cast_Short.h>
44: #include <ATen/ops/_cast_Half.h>
45: #include <ATen/ops/_make_dual.h>
46: #include <ATen/ops/_unpack_dual.h>
47: #include <ATen/ops/_functional_assert_scalar.h>
48: #include <ATen/ops/_functional_assert_async.h>
49: #include <ATen/ops/sym_constrain_range_for_size.h>
50: #include <ATen/ops/_functional_sym_constrain_range.h>
51: #include <ATen/ops/_cudnn_ctc_loss.h>
52: #include <ATen/ops/_cudnn_init_dropout_state.h>
53: #include <ATen/ops/_debug_has_internal_overlap.h>
54: #include <ATen/ops/_masked_scale.h>
55: #include <ATen/ops/_sobol_engine_initialize_state.h>
56: #include <ATen/ops/feature_alpha_dropout.h>
57: #include <ATen/ops/feature_alpha_dropout.h>
58: #include <ATen/ops/abs.h>
59: #include <ATen/ops/abs.h>
60: #include <ATen/ops/absolute.h>
61: #include <ATen/ops/view_as_real.h>
62: #include <ATen/ops/real.h>
63: #include <ATen/ops/_conj_physical.h>
64: #include <ATen/ops/resolve_conj.h>
65: #include <ATen/ops/_neg_view.h>
66: #include <ATen/ops/acos.h>
67: #include <ATen/ops/acos.h>
68: #include <ATen/ops/arccos.h>
69: #include <ATen/ops/arccos.h>
70: #include <ATen/ops/adaptive_avg_pool1d.h>
71: #include <ATen/ops/addmv.h>
72: #include <ATen/ops/addmv.h>
73: #include <ATen/ops/_is_all_true.h>
74: #include <ATen/ops/_test_check_tensor.h>
75: #include <ATen/ops/argmin.h>
76: #include <ATen/ops/asinh.h>
77: #include <ATen/ops/asinh.h>
78: #include <ATen/ops/arcsinh.h>
79: #include <ATen/ops/arcsinh.h>
80: #include <ATen/ops/asin.h>
```

- EN: These lines pull in dependencies such as `Python.h`, `torch/csrc/autograd/python_torch_functions.h`, `torch/csrc/autograd/python_variable.h`, establishing the headers needed by the implementation. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `Python.h`, `torch/csrc/autograd/python_torch_functions.h`, `torch/csrc/autograd/python_variable.h`，为后续实现建立所需的头文件基础。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 81-160

```cpp
 81: #include <ATen/ops/asin.h>
 82: #include <ATen/ops/arctan.h>
 83: #include <ATen/ops/arctan.h>
 84: #include <ATen/ops/atleast_2d.h>
 85: #include <ATen/ops/atleast_3d.h>
 86: #include <ATen/ops/quantized_batch_norm.h>
 87: #include <ATen/ops/_batch_norm_impl_index.h>
 88: #include <ATen/ops/bincount.h>
 89: #include <ATen/ops/bitwise_not.h>
 90: #include <ATen/ops/logical_or.h>
 91: #include <ATen/ops/unsafe_chunk.h>
 92: #include <ATen/ops/clamp.h>
 93: #include <ATen/ops/clamp.h>
 94: #include <ATen/ops/clamp_max.h>
 95: #include <ATen/ops/clamp_max.h>
 96: #include <ATen/ops/constant_pad_nd.h>
 97: #include <ATen/ops/convolution.h>
 98: #include <ATen/ops/conv2d.h>
 99: #include <ATen/ops/conv_tbc.h>
100: #include <ATen/ops/conv_transpose1d.h>
101: #include <ATen/ops/conv_transpose3d.h>
102: #include <ATen/ops/cosine_embedding_loss.h>
103: #include <ATen/ops/cudnn_affine_grid_generator.h>
104: #include <ATen/ops/cudnn_batch_norm.h>
105: #include <ATen/ops/_mps_convolution_transpose.h>
106: #include <ATen/ops/cudnn_convolution_add_relu.h>
107: #include <ATen/ops/cudnn_grid_sampler.h>
108: #include <ATen/ops/_cummax_helper.h>
109: #include <ATen/ops/cumprod.h>
110: #include <ATen/ops/gradient.h>
111: #include <ATen/ops/div.h>
112: #include <ATen/ops/true_divide.h>
113: #include <ATen/ops/_embedding_bag_forward_only.h>
114: #include <ATen/ops/row_stack.h>
115: #include <ATen/ops/_empty_per_channel_affine_quantized.h>
116: #include <ATen/ops/_resize_output.h>
117: #include <ATen/ops/empty_strided.h>
118: #include <ATen/ops/flatten.h>
119: #include <ATen/ops/fill.h>
120: #include <ATen/ops/fill.h>
121: #include <ATen/ops/grid_sampler_2d.h>
122: #include <ATen/ops/hann_window.h>
123: #include <ATen/ops/kaiser_window.h>
124: #include <ATen/ops/_fft_r2c.h>
125: #include <ATen/ops/_validate_compressed_sparse_indices.h>
126: #include <ATen/ops/_cufft_get_plan_cache_size.h>
127: #include <ATen/ops/_unsafe_masked_index.h>
128: #include <ATen/ops/index_put.h>
129: #include <ATen/ops/index_put.h>
130: #include <ATen/ops/_unsafe_index_put.h>
131: #include <ATen/ops/is_distributed.h>
132: #include <ATen/ops/is_complex.h>
133: #include <ATen/ops/isreal.h>
134: #include <ATen/ops/kron.h>
135: #include <ATen/ops/native_layer_norm.h>
136: #include <ATen/ops/_cslt_compress.h>
137: #include <ATen/ops/_sparse_semi_structured_tile.h>
138: #include <ATen/ops/_sparse_semi_structured_linear.h>
139: #include <ATen/ops/fbgemm_linear_fp16_weight.h>
140: #include <ATen/ops/ldexp.h>
141: #include <ATen/ops/ldexp.h>
142: #include <ATen/ops/log10.h>
143: #include <ATen/ops/log10.h>
144: #include <ATen/ops/logaddexp2.h>
145: #include <ATen/ops/_log_softmax_backward_data.h>
146: #include <ATen/ops/logcumsumexp.h>
147: #include <ATen/ops/margin_ranking_loss.h>
148: #include <ATen/ops/matrix_exp.h>
149: #include <ATen/ops/amax.h>
150: #include <ATen/ops/max_pool1d.h>
151: #include <ATen/ops/mkldnn_max_pool3d.h>
152: #include <ATen/ops/quantized_max_pool3d.h>
153: #include <ATen/ops/max_pool3d.h>
154: #include <ATen/ops/median.h>
155: #include <ATen/ops/_mps_convolution.h>
156: #include <ATen/ops/miopen_convolution.h>
157: #include <ATen/ops/miopen_convolution_transpose.h>
158: #include <ATen/ops/miopen_convolution_add_relu.h>
159: #include <ATen/ops/_use_miopen_ctc_loss.h>
160: #include <ATen/ops/miopen_ctc_loss.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/asin.h`, `ATen/ops/arctan.h`, `ATen/ops/atleast_2d.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/asin.h`, `ATen/ops/arctan.h`, `ATen/ops/atleast_2d.h`，为后续实现建立所需的头文件基础。
### Lines 161-240

```cpp
161: #include <ATen/ops/mm.h>
162: #include <ATen/ops/_int_mm.h>
163: #include <ATen/ops/_weight_int4pack_mm_with_scales_and_zeros.h>
164: #include <ATen/ops/_convert_weight_to_int4pack_for_cpu.h>
165: #include <ATen/ops/multiply.h>
166: #include <ATen/ops/mvlgamma.h>
167: #include <ATen/ops/narrow_copy.h>
168: #include <ATen/ops/_native_batch_norm_legit_no_training.h>
169: #include <ATen/ops/batch_norm_elemt.h>
170: #include <ATen/ops/batch_norm_gather_stats_with_counts.h>
171: #include <ATen/ops/batch_norm_backward_elemt.h>
172: #include <ATen/ops/ones_like.h>
173: #include <ATen/ops/pairwise_distance.h>
174: #include <ATen/ops/cdist.h>
175: #include <ATen/ops/cosine_similarity.h>
176: #include <ATen/ops/movedim.h>
177: #include <ATen/ops/adjoint.h>
178: #include <ATen/ops/channel_shuffle.h>
179: #include <ATen/ops/pinverse.h>
180: #include <ATen/ops/poisson_nll_loss.h>
181: #include <ATen/ops/deg2rad.h>
182: #include <ATen/ops/deg2rad.h>
183: #include <ATen/ops/randn_like.h>
184: #include <ATen/ops/repeat_interleave.h>
185: #include <ATen/ops/_prelu_kernel.h>
186: #include <ATen/ops/hardshrink.h>
187: #include <ATen/ops/rsqrt.h>
188: #include <ATen/ops/rsqrt.h>
189: #include <ATen/ops/celu.h>
190: #include <ATen/ops/celu.h>
191: #include <ATen/ops/sinc.h>
192: #include <ATen/ops/sinc.h>
193: #include <ATen/ops/sinh.h>
194: #include <ATen/ops/sinh.h>
195: #include <ATen/ops/detach.h>
196: #include <ATen/ops/detach.h>
197: #include <ATen/ops/_softmax.h>
198: #include <ATen/ops/_softmax_backward_data.h>
199: #include <ATen/ops/unsafe_split.h>
200: #include <ATen/ops/split.h>
201: #include <ATen/ops/hsplit.h>
202: #include <ATen/ops/squeeze.h>
203: #include <ATen/ops/stack.h>
204: #include <ATen/ops/_stack.h>
205: #include <ATen/ops/istft.h>
206: #include <ATen/ops/nansum.h>
207: #include <ATen/ops/sqrt.h>
208: #include <ATen/ops/sqrt.h>
209: #include <ATen/ops/square.h>
210: #include <ATen/ops/square.h>
211: #include <ATen/ops/t.h>
212: #include <ATen/ops/tanh.h>
213: #include <ATen/ops/tanh.h>
214: #include <ATen/ops/tensordot.h>
215: #include <ATen/ops/tile.h>
216: #include <ATen/ops/_mkldnn_transpose.h>
217: #include <ATen/ops/_mkldnn_transpose.h>
218: #include <ATen/ops/rot90.h>
219: #include <ATen/ops/_nested_from_padded.h>
220: #include <ATen/ops/_nested_get_values.h>
221: #include <ATen/ops/_nested_get_offsets.h>
222: #include <ATen/ops/_nested_get_jagged_dummy.h>
223: #include <ATen/ops/triplet_margin_loss.h>
224: #include <ATen/ops/fix.h>
225: #include <ATen/ops/fix.h>
226: #include <ATen/ops/_has_compatible_shallow_copy_type.h>
227: #include <ATen/ops/_unique.h>
228: #include <ATen/ops/vander.h>
229: #include <ATen/ops/var_mean.h>
230: #include <ATen/ops/_weight_norm.h>
231: #include <ATen/ops/_standard_gamma_grad.h>
232: #include <ATen/ops/_philox_key_split.h>
233: #include <ATen/ops/poisson.h>
234: #include <ATen/ops/native_norm.h>
235: #include <ATen/ops/_sparse_softmax_backward_data.h>
236: #include <ATen/ops/norm.h>
237: #include <ATen/ops/frexp.h>
238: #include <ATen/ops/positive.h>
239: #include <ATen/ops/resize_as.h>
240: #include <ATen/ops/zero.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/mm.h`, `ATen/ops/_int_mm.h`, `ATen/ops/_weight_int4pack_mm_with_scales_and_zeros.h`, establishing the headers needed by the implementation. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/ops/mm.h`, `ATen/ops/_int_mm.h`, `ATen/ops/_weight_int4pack_mm_with_scales_and_zeros.h`，为后续实现建立所需的头文件基础。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 241-320

```cpp
241: #include <ATen/ops/sub.h>
242: #include <ATen/ops/subtract.h>
243: #include <ATen/ops/rsub.h>
244: #include <ATen/ops/_validate_sparse_csc_tensor_args.h>
245: #include <ATen/ops/_coalesce.h>
246: #include <ATen/ops/unbind.h>
247: #include <ATen/ops/quantize_per_tensor_dynamic.h>
248: #include <ATen/ops/quantize_per_tensor.h>
249: #include <ATen/ops/quantize_per_channel.h>
250: #include <ATen/ops/q_zero_point.h>
251: #include <ATen/ops/int_repr.h>
252: #include <ATen/ops/_make_per_tensor_quantized_tensor.h>
253: #include <ATen/ops/_make_per_channel_quantized_tensor.h>
254: #include <ATen/ops/fake_quantize_per_channel_affine.h>
255: #include <ATen/ops/_fused_moving_avg_obs_fq_helper.h>
256: #include <ATen/ops/_choose_qparams_per_tensor.h>
257: #include <ATen/ops/meshgrid.h>
258: #include <ATen/ops/promote_types.h>
259: #include <ATen/ops/gru.h>
260: #include <ATen/ops/rnn_tanh_cell.h>
261: #include <ATen/ops/quantized_gru_cell.h>
262: #include <ATen/ops/masked_scatter.h>
263: #include <ATen/ops/put.h>
264: #include <ATen/ops/or.h>
265: #include <ATen/ops/xor.h>
266: #include <ATen/ops/lshift.h>
267: #include <ATen/ops/bitwise_right_shift.h>
268: #include <ATen/ops/tril.h>
269: #include <ATen/ops/tril_indices.h>
270: #include <ATen/ops/not_equal.h>
271: #include <ATen/ops/greater_equal.h>
272: #include <ATen/ops/less.h>
273: #include <ATen/ops/index_select.h>
274: #include <ATen/ops/gather.h>
275: #include <ATen/ops/addcmul.h>
276: #include <ATen/ops/triangular_solve.h>
277: #include <ATen/ops/swapaxes.h>
278: #include <ATen/ops/cholesky_inverse.h>
279: #include <ATen/ops/geqrf.h>
280: #include <ATen/ops/orgqr.h>
281: #include <ATen/ops/ormqr.h>
282: #include <ATen/ops/lu_solve.h>
283: #include <ATen/ops/lgamma.h>
284: #include <ATen/ops/digamma.h>
285: #include <ATen/ops/erfinv.h>
286: #include <ATen/ops/sign.h>
287: #include <ATen/ops/dist.h>
288: #include <ATen/ops/arctan2.h>
289: #include <ATen/ops/histc.h>
290: #include <ATen/ops/histogram.h>
291: #include <ATen/ops/_histogramdd_bin_edges.h>
292: #include <ATen/ops/histogramdd.h>
293: #include <ATen/ops/fmax.h>
294: #include <ATen/ops/maximum.h>
295: #include <ATen/ops/minimum.h>
296: #include <ATen/ops/nanquantile.h>
297: #include <ATen/ops/msort.h>
298: #include <ATen/ops/pow.h>
299: #include <ATen/ops/float_power.h>
300: #include <ATen/ops/_foreach_add.h>
301: #include <ATen/ops/_foreach_add.h>
302: #include <ATen/ops/_foreach_sub.h>
303: #include <ATen/ops/_foreach_sub.h>
304: #include <ATen/ops/_foreach_clamp_max.h>
305: #include <ATen/ops/_foreach_clamp_max.h>
306: #include <ATen/ops/_foreach_acos.h>
307: #include <ATen/ops/_foreach_acos.h>
308: #include <ATen/ops/_foreach_asin.h>
309: #include <ATen/ops/_foreach_asin.h>
310: #include <ATen/ops/_foreach_cos.h>
311: #include <ATen/ops/_foreach_cos.h>
312: #include <ATen/ops/_foreach_cosh.h>
313: #include <ATen/ops/_foreach_cosh.h>
314: #include <ATen/ops/_foreach_exp.h>
315: #include <ATen/ops/_foreach_exp.h>
316: #include <ATen/ops/_foreach_floor.h>
317: #include <ATen/ops/_foreach_floor.h>
318: #include <ATen/ops/_foreach_log1p.h>
319: #include <ATen/ops/_foreach_log1p.h>
320: #include <ATen/ops/_foreach_log2.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/sub.h`, `ATen/ops/subtract.h`, `ATen/ops/rsub.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/sub.h`, `ATen/ops/subtract.h`, `ATen/ops/rsub.h`，为后续实现建立所需的头文件基础。
### Lines 321-400

```cpp
321: #include <ATen/ops/_foreach_log2.h>
322: #include <ATen/ops/_foreach_neg.h>
323: #include <ATen/ops/_foreach_neg.h>
324: #include <ATen/ops/_foreach_norm.h>
325: #include <ATen/ops/_foreach_sigmoid.h>
326: #include <ATen/ops/_foreach_sigmoid.h>
327: #include <ATen/ops/_foreach_sign.h>
328: #include <ATen/ops/_foreach_sign.h>
329: #include <ATen/ops/_foreach_sqrt.h>
330: #include <ATen/ops/_foreach_sqrt.h>
331: #include <ATen/ops/_foreach_tanh.h>
332: #include <ATen/ops/_foreach_tanh.h>
333: #include <ATen/ops/_foreach_clone.h>
334: #include <ATen/ops/_convert_indices_from_csr_to_coo.h>
335: #include <ATen/ops/mkldnn_adaptive_avg_pool2d.h>
336: #include <ATen/ops/column_stack.h>
337: #include <ATen/ops/isinf.h>
338: #include <ATen/ops/isneginf.h>
339: #include <ATen/ops/_linalg_slogdet.h>
340: #include <ATen/ops/slogdet.h>
341: #include <ATen/ops/inverse.h>
342: #include <ATen/ops/inner.h>
343: #include <ATen/ops/_linalg_solve_ex.h>
344: #include <ATen/ops/_test_parallel_materialize.h>
345: #include <ATen/ops/_test_autograd_multiple_dispatch.h>
346: #include <ATen/ops/_test_autograd_multiple_dispatch_view_copy.h>
347: #include <ATen/ops/segment_reduce.h>
348: #include <ATen/ops/_conj_copy.h>
349: #include <ATen/ops/_sparse_broadcast_to_copy.h>
350: #include <ATen/ops/select_copy.h>
351: #include <ATen/ops/squeeze_copy.h>
352: #include <ATen/ops/transpose_copy.h>
353: #include <ATen/ops/values_copy.h>
354: #include <ATen/ops/unfold_copy.h>
355: #include <ATen/ops/_fused_sdp_choice.h>
356: #include <ATen/ops/_scaled_dot_product_efficient_attention.h>
357: #include <ATen/ops/_scaled_dot_product_cudnn_attention.h>
358: #include <ATen/ops/_flash_attention_forward_no_dropout_inplace.h>
359: #include <ATen/ops/_fill_mem_eff_dropout_mask.h>
360: #include <ATen/ops/_foobar.h>
361: #include <ATen/ops/_fused_adagrad.h>
362: #include <ATen/ops/_propagate_xla_data.h>
363: #endif
364: 
365: #include <functional>
366: #include <initializer_list>
367: #include <stdexcept>
368: #include <utility>
369: 
370: using at::Tensor;
371: using at::Device;
372: using at::Layout;
373: using at::Scalar;
374: using at::ScalarType;
375: using at::Backend;
376: using at::OptionalDeviceGuard;
377: using at::DeviceGuard;
378: using at::TensorOptions;
379: using at::IntArrayRef;
380: using at::Generator;
381: using at::TensorList;
382: using at::Dimname;
383: using at::DimnameList;
384: using at::ArrayRef;
385: 
386: using torch::utils::check_out_type_matches;
387: using namespace torch::autograd::utils;
388: 
389: // NOTE: See [Sharded File] comment in VariableType
390: 
391: namespace torch::autograd {
392: 
393: // generated forward declarations start here
394: 
395: static PyObject * THPVariable__cast_Char(PyObject* self_, PyObject* args, PyObject* kwargs);
396: static PyObject * THPVariable__cast_Short(PyObject* self_, PyObject* args, PyObject* kwargs);
397: static PyObject * THPVariable__cast_Half(PyObject* self_, PyObject* args, PyObject* kwargs);
398: static PyObject * THPVariable__make_dual(PyObject* self_, PyObject* args, PyObject* kwargs);
399: static PyObject * THPVariable__unpack_dual(PyObject* self_, PyObject* args, PyObject* kwargs);
400: static PyObject * THPVariable__functional_assert_scalar(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: These lines pull in dependencies such as `ATen/ops/_foreach_log2.h`, `ATen/ops/_foreach_neg.h`, `ATen/ops/_foreach_norm.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable__cast_Char`, `THPVariable__cast_Short`, `THPVariable__cast_Half`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这些行引入了依赖，例如 `ATen/ops/_foreach_log2.h`, `ATen/ops/_foreach_neg.h`, `ATen/ops/_foreach_norm.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable__cast_Char`, `THPVariable__cast_Short`, `THPVariable__cast_Half` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 401-480

```cpp
401: static PyObject * THPVariable__functional_assert_async(PyObject* self_, PyObject* args, PyObject* kwargs);
402: static PyObject * THPVariable_sym_constrain_range_for_size(PyObject* self_, PyObject* args, PyObject* kwargs);
403: static PyObject * THPVariable__functional_sym_constrain_range(PyObject* self_, PyObject* args, PyObject* kwargs);
404: static PyObject * THPVariable__cudnn_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
405: static PyObject * THPVariable__cudnn_init_dropout_state(PyObject* self_, PyObject* args, PyObject* kwargs);
406: static PyObject * THPVariable__debug_has_internal_overlap(PyObject* self_, PyObject* args, PyObject* kwargs);
407: static PyObject * THPVariable__masked_scale(PyObject* self_, PyObject* args, PyObject* kwargs);
408: static PyObject * THPVariable__sobol_engine_initialize_state_(PyObject* self_, PyObject* args, PyObject* kwargs);
409: static PyObject * THPVariable_feature_alpha_dropout(PyObject* self_, PyObject* args, PyObject* kwargs);
410: static PyObject * THPVariable_feature_alpha_dropout_(PyObject* self_, PyObject* args, PyObject* kwargs);
411: static PyObject * THPVariable_abs(PyObject* self_, PyObject* args, PyObject* kwargs);
412: static PyObject * THPVariable_abs_(PyObject* self_, PyObject* args, PyObject* kwargs);
413: static PyObject * THPVariable_absolute(PyObject* self_, PyObject* args, PyObject* kwargs);
414: static PyObject * THPVariable_view_as_real(PyObject* self_, PyObject* args, PyObject* kwargs);
415: static PyObject * THPVariable_real(PyObject* self_, PyObject* args, PyObject* kwargs);
416: static PyObject * THPVariable__conj_physical(PyObject* self_, PyObject* args, PyObject* kwargs);
417: static PyObject * THPVariable_resolve_conj(PyObject* self_, PyObject* args, PyObject* kwargs);
418: static PyObject * THPVariable__neg_view(PyObject* self_, PyObject* args, PyObject* kwargs);
419: static PyObject * THPVariable_acos(PyObject* self_, PyObject* args, PyObject* kwargs);
420: static PyObject * THPVariable_acos_(PyObject* self_, PyObject* args, PyObject* kwargs);
421: static PyObject * THPVariable_arccos(PyObject* self_, PyObject* args, PyObject* kwargs);
422: static PyObject * THPVariable_arccos_(PyObject* self_, PyObject* args, PyObject* kwargs);
423: static PyObject * THPVariable_adaptive_avg_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs);
424: static PyObject * THPVariable_addmv(PyObject* self_, PyObject* args, PyObject* kwargs);
425: static PyObject * THPVariable_addmv_(PyObject* self_, PyObject* args, PyObject* kwargs);
426: static PyObject * THPVariable__is_all_true(PyObject* self_, PyObject* args, PyObject* kwargs);
427: static PyObject * THPVariable__test_check_tensor(PyObject* self_, PyObject* args, PyObject* kwargs);
428: static PyObject * THPVariable_argmin(PyObject* self_, PyObject* args, PyObject* kwargs);
429: static PyObject * THPVariable_asinh(PyObject* self_, PyObject* args, PyObject* kwargs);
430: static PyObject * THPVariable_asinh_(PyObject* self_, PyObject* args, PyObject* kwargs);
431: static PyObject * THPVariable_arcsinh(PyObject* self_, PyObject* args, PyObject* kwargs);
432: static PyObject * THPVariable_arcsinh_(PyObject* self_, PyObject* args, PyObject* kwargs);
433: static PyObject * THPVariable_asin(PyObject* self_, PyObject* args, PyObject* kwargs);
434: static PyObject * THPVariable_asin_(PyObject* self_, PyObject* args, PyObject* kwargs);
435: static PyObject * THPVariable_arctan(PyObject* self_, PyObject* args, PyObject* kwargs);
436: static PyObject * THPVariable_arctan_(PyObject* self_, PyObject* args, PyObject* kwargs);
437: static PyObject * THPVariable_atleast_2d(PyObject* self_, PyObject* args, PyObject* kwargs);
438: static PyObject * THPVariable_atleast_3d(PyObject* self_, PyObject* args, PyObject* kwargs);
439: static PyObject * THPVariable_quantized_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
440: static PyObject * THPVariable__batch_norm_impl_index(PyObject* self_, PyObject* args, PyObject* kwargs);
441: static PyObject * THPVariable_bincount(PyObject* self_, PyObject* args, PyObject* kwargs);
442: static PyObject * THPVariable_bitwise_not(PyObject* self_, PyObject* args, PyObject* kwargs);
443: static PyObject * THPVariable_logical_or(PyObject* self_, PyObject* args, PyObject* kwargs);
444: static PyObject * THPVariable_unsafe_chunk(PyObject* self_, PyObject* args, PyObject* kwargs);
445: static PyObject * THPVariable_clamp(PyObject* self_, PyObject* args, PyObject* kwargs);
446: static PyObject * THPVariable_clamp_(PyObject* self_, PyObject* args, PyObject* kwargs);
447: static PyObject * THPVariable_clamp_max(PyObject* self_, PyObject* args, PyObject* kwargs);
448: static PyObject * THPVariable_clamp_max_(PyObject* self_, PyObject* args, PyObject* kwargs);
449: static PyObject * THPVariable_constant_pad_nd(PyObject* self_, PyObject* args, PyObject* kwargs);
450: static PyObject * THPVariable_convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
451: static PyObject * THPVariable_conv2d(PyObject* self_, PyObject* args, PyObject* kwargs);
452: static PyObject * THPVariable_conv_tbc(PyObject* self_, PyObject* args, PyObject* kwargs);
453: static PyObject * THPVariable_conv_transpose1d(PyObject* self_, PyObject* args, PyObject* kwargs);
454: static PyObject * THPVariable_conv_transpose3d(PyObject* self_, PyObject* args, PyObject* kwargs);
455: static PyObject * THPVariable_cosine_embedding_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
456: static PyObject * THPVariable_cudnn_affine_grid_generator(PyObject* self_, PyObject* args, PyObject* kwargs);
457: static PyObject * THPVariable_cudnn_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
458: static PyObject * THPVariable__mps_convolution_transpose(PyObject* self_, PyObject* args, PyObject* kwargs);
459: static PyObject * THPVariable_cudnn_convolution_add_relu(PyObject* self_, PyObject* args, PyObject* kwargs);
460: static PyObject * THPVariable_cudnn_grid_sampler(PyObject* self_, PyObject* args, PyObject* kwargs);
461: static PyObject * THPVariable__cummax_helper(PyObject* self_, PyObject* args, PyObject* kwargs);
462: static PyObject * THPVariable_cumprod(PyObject* self_, PyObject* args, PyObject* kwargs);
463: static PyObject * THPVariable_gradient(PyObject* self_, PyObject* args, PyObject* kwargs);
464: static PyObject * THPVariable_div(PyObject* self_, PyObject* args, PyObject* kwargs);
465: static PyObject * THPVariable_true_divide(PyObject* self_, PyObject* args, PyObject* kwargs);
466: static PyObject * THPVariable__embedding_bag_forward_only(PyObject* self_, PyObject* args, PyObject* kwargs);
467: static PyObject * THPVariable_row_stack(PyObject* self_, PyObject* args, PyObject* kwargs);
468: static PyObject * THPVariable__empty_per_channel_affine_quantized(PyObject* self_, PyObject* args, PyObject* kwargs);
469: static PyObject * THPVariable__resize_output_(PyObject* self_, PyObject* args, PyObject* kwargs);
470: static PyObject * THPVariable_empty_strided(PyObject* self_, PyObject* args, PyObject* kwargs);
471: static PyObject * THPVariable_flatten(PyObject* self_, PyObject* args, PyObject* kwargs);
472: static PyObject * THPVariable_fill(PyObject* self_, PyObject* args, PyObject* kwargs);
473: static PyObject * THPVariable_fill_(PyObject* self_, PyObject* args, PyObject* kwargs);
474: static PyObject * THPVariable_grid_sampler_2d(PyObject* self_, PyObject* args, PyObject* kwargs);
475: static PyObject * THPVariable_hann_window(PyObject* self_, PyObject* args, PyObject* kwargs);
476: static PyObject * THPVariable_kaiser_window(PyObject* self_, PyObject* args, PyObject* kwargs);
477: static PyObject * THPVariable__fft_r2c(PyObject* self_, PyObject* args, PyObject* kwargs);
478: static PyObject * THPVariable__validate_compressed_sparse_indices(PyObject* self_, PyObject* args, PyObject* kwargs);
479: static PyObject * THPVariable__cufft_get_plan_cache_size(PyObject* self_, PyObject* args, PyObject* kwargs);
480: static PyObject * THPVariable__unsafe_masked_index(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable__functional_assert_async`, `THPVariable_sym_constrain_range_for_size`, `THPVariable__functional_sym_constrain_range`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__functional_assert_async`, `THPVariable_sym_constrain_range_for_size`, `THPVariable__functional_sym_constrain_range` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 481-560

```cpp
481: static PyObject * THPVariable_index_put_(PyObject* self_, PyObject* args, PyObject* kwargs);
482: static PyObject * THPVariable_index_put(PyObject* self_, PyObject* args, PyObject* kwargs);
483: static PyObject * THPVariable__unsafe_index_put(PyObject* self_, PyObject* args, PyObject* kwargs);
484: static PyObject * THPVariable_is_distributed(PyObject* self_, PyObject* args, PyObject* kwargs);
485: static PyObject * THPVariable_is_complex(PyObject* self_, PyObject* args, PyObject* kwargs);
486: static PyObject * THPVariable_isreal(PyObject* self_, PyObject* args, PyObject* kwargs);
487: static PyObject * THPVariable_kron(PyObject* self_, PyObject* args, PyObject* kwargs);
488: static PyObject * THPVariable_native_layer_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
489: static PyObject * THPVariable__cslt_compress(PyObject* self_, PyObject* args, PyObject* kwargs);
490: static PyObject * THPVariable__sparse_semi_structured_tile(PyObject* self_, PyObject* args, PyObject* kwargs);
491: static PyObject * THPVariable__sparse_semi_structured_linear(PyObject* self_, PyObject* args, PyObject* kwargs);
492: static PyObject * THPVariable_fbgemm_linear_fp16_weight(PyObject* self_, PyObject* args, PyObject* kwargs);
493: static PyObject * THPVariable_ldexp(PyObject* self_, PyObject* args, PyObject* kwargs);
494: static PyObject * THPVariable_ldexp_(PyObject* self_, PyObject* args, PyObject* kwargs);
495: static PyObject * THPVariable_log10(PyObject* self_, PyObject* args, PyObject* kwargs);
496: static PyObject * THPVariable_log10_(PyObject* self_, PyObject* args, PyObject* kwargs);
497: static PyObject * THPVariable_logaddexp2(PyObject* self_, PyObject* args, PyObject* kwargs);
498: static PyObject * THPVariable__log_softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs);
499: static PyObject * THPVariable_logcumsumexp(PyObject* self_, PyObject* args, PyObject* kwargs);
500: static PyObject * THPVariable_margin_ranking_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
501: static PyObject * THPVariable_matrix_exp(PyObject* self_, PyObject* args, PyObject* kwargs);
502: static PyObject * THPVariable_amax(PyObject* self_, PyObject* args, PyObject* kwargs);
503: static PyObject * THPVariable_max_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs);
504: static PyObject * THPVariable_mkldnn_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
505: static PyObject * THPVariable_quantized_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
506: static PyObject * THPVariable_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs);
507: static PyObject * THPVariable_median(PyObject* self_, PyObject* args, PyObject* kwargs);
508: static PyObject * THPVariable__mps_convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
509: static PyObject * THPVariable_miopen_convolution(PyObject* self_, PyObject* args, PyObject* kwargs);
510: static PyObject * THPVariable_miopen_convolution_transpose(PyObject* self_, PyObject* args, PyObject* kwargs);
511: static PyObject * THPVariable_miopen_convolution_add_relu(PyObject* self_, PyObject* args, PyObject* kwargs);
512: static PyObject * THPVariable__use_miopen_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
513: static PyObject * THPVariable_miopen_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
514: static PyObject * THPVariable_mm(PyObject* self_, PyObject* args, PyObject* kwargs);
515: static PyObject * THPVariable__int_mm(PyObject* self_, PyObject* args, PyObject* kwargs);
516: static PyObject * THPVariable__weight_int4pack_mm_with_scales_and_zeros(PyObject* self_, PyObject* args, PyObject* kwargs);
517: static PyObject * THPVariable__convert_weight_to_int4pack_for_cpu(PyObject* self_, PyObject* args, PyObject* kwargs);
518: static PyObject * THPVariable_multiply(PyObject* self_, PyObject* args, PyObject* kwargs);
519: static PyObject * THPVariable_mvlgamma(PyObject* self_, PyObject* args, PyObject* kwargs);
520: static PyObject * THPVariable_narrow_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
521: static PyObject * THPVariable__native_batch_norm_legit_no_training(PyObject* self_, PyObject* args, PyObject* kwargs);
522: static PyObject * THPVariable_batch_norm_elemt(PyObject* self_, PyObject* args, PyObject* kwargs);
523: static PyObject * THPVariable_batch_norm_gather_stats_with_counts(PyObject* self_, PyObject* args, PyObject* kwargs);
524: static PyObject * THPVariable_batch_norm_backward_elemt(PyObject* self_, PyObject* args, PyObject* kwargs);
525: static PyObject * THPVariable_ones_like(PyObject* self_, PyObject* args, PyObject* kwargs);
526: static PyObject * THPVariable_pairwise_distance(PyObject* self_, PyObject* args, PyObject* kwargs);
527: static PyObject * THPVariable_cdist(PyObject* self_, PyObject* args, PyObject* kwargs);
528: static PyObject * THPVariable_cosine_similarity(PyObject* self_, PyObject* args, PyObject* kwargs);
529: static PyObject * THPVariable_movedim(PyObject* self_, PyObject* args, PyObject* kwargs);
530: static PyObject * THPVariable_adjoint(PyObject* self_, PyObject* args, PyObject* kwargs);
531: static PyObject * THPVariable_channel_shuffle(PyObject* self_, PyObject* args, PyObject* kwargs);
532: static PyObject * THPVariable_pinverse(PyObject* self_, PyObject* args, PyObject* kwargs);
533: static PyObject * THPVariable_poisson_nll_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
534: static PyObject * THPVariable_deg2rad(PyObject* self_, PyObject* args, PyObject* kwargs);
535: static PyObject * THPVariable_deg2rad_(PyObject* self_, PyObject* args, PyObject* kwargs);
536: static PyObject * THPVariable_randn_like(PyObject* self_, PyObject* args, PyObject* kwargs);
537: static PyObject * THPVariable_repeat_interleave(PyObject* self_, PyObject* args, PyObject* kwargs);
538: static PyObject * THPVariable__prelu_kernel(PyObject* self_, PyObject* args, PyObject* kwargs);
539: static PyObject * THPVariable_hardshrink(PyObject* self_, PyObject* args, PyObject* kwargs);
540: static PyObject * THPVariable_rsqrt(PyObject* self_, PyObject* args, PyObject* kwargs);
541: static PyObject * THPVariable_rsqrt_(PyObject* self_, PyObject* args, PyObject* kwargs);
542: static PyObject * THPVariable_celu(PyObject* self_, PyObject* args, PyObject* kwargs);
543: static PyObject * THPVariable_celu_(PyObject* self_, PyObject* args, PyObject* kwargs);
544: static PyObject * THPVariable_sinc(PyObject* self_, PyObject* args, PyObject* kwargs);
545: static PyObject * THPVariable_sinc_(PyObject* self_, PyObject* args, PyObject* kwargs);
546: static PyObject * THPVariable_sinh(PyObject* self_, PyObject* args, PyObject* kwargs);
547: static PyObject * THPVariable_sinh_(PyObject* self_, PyObject* args, PyObject* kwargs);
548: static PyObject * THPVariable_detach(PyObject* self_, PyObject* args, PyObject* kwargs);
549: static PyObject * THPVariable_detach_(PyObject* self_, PyObject* args, PyObject* kwargs);
550: static PyObject * THPVariable__softmax(PyObject* self_, PyObject* args, PyObject* kwargs);
551: static PyObject * THPVariable__softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs);
552: static PyObject * THPVariable_unsafe_split(PyObject* self_, PyObject* args, PyObject* kwargs);
553: static PyObject * THPVariable_split(PyObject* self_, PyObject* args, PyObject* kwargs);
554: static PyObject * THPVariable_hsplit(PyObject* self_, PyObject* args, PyObject* kwargs);
555: static PyObject * THPVariable_squeeze(PyObject* self_, PyObject* args, PyObject* kwargs);
556: static PyObject * THPVariable_stack(PyObject* self_, PyObject* args, PyObject* kwargs);
557: static PyObject * THPVariable__stack(PyObject* self_, PyObject* args, PyObject* kwargs);
558: static PyObject * THPVariable_istft(PyObject* self_, PyObject* args, PyObject* kwargs);
559: static PyObject * THPVariable_nansum(PyObject* self_, PyObject* args, PyObject* kwargs);
560: static PyObject * THPVariable_sqrt(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable_index_put_`, `THPVariable_index_put`, `THPVariable__unsafe_index_put`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_index_put_`, `THPVariable_index_put`, `THPVariable__unsafe_index_put` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 561-640

```cpp
561: static PyObject * THPVariable_sqrt_(PyObject* self_, PyObject* args, PyObject* kwargs);
562: static PyObject * THPVariable_square(PyObject* self_, PyObject* args, PyObject* kwargs);
563: static PyObject * THPVariable_square_(PyObject* self_, PyObject* args, PyObject* kwargs);
564: static PyObject * THPVariable_t(PyObject* self_, PyObject* args, PyObject* kwargs);
565: static PyObject * THPVariable_tanh(PyObject* self_, PyObject* args, PyObject* kwargs);
566: static PyObject * THPVariable_tanh_(PyObject* self_, PyObject* args, PyObject* kwargs);
567: static PyObject * THPVariable_tensordot(PyObject* self_, PyObject* args, PyObject* kwargs);
568: static PyObject * THPVariable_tile(PyObject* self_, PyObject* args, PyObject* kwargs);
569: static PyObject * THPVariable__mkldnn_transpose(PyObject* self_, PyObject* args, PyObject* kwargs);
570: static PyObject * THPVariable__mkldnn_transpose_(PyObject* self_, PyObject* args, PyObject* kwargs);
571: static PyObject * THPVariable_rot90(PyObject* self_, PyObject* args, PyObject* kwargs);
572: static PyObject * THPVariable__nested_from_padded(PyObject* self_, PyObject* args, PyObject* kwargs);
573: static PyObject * THPVariable__nested_get_values(PyObject* self_, PyObject* args, PyObject* kwargs);
574: static PyObject * THPVariable__nested_get_offsets(PyObject* self_, PyObject* args, PyObject* kwargs);
575: static PyObject * THPVariable__nested_get_jagged_dummy(PyObject* self_, PyObject* args, PyObject* kwargs);
576: static PyObject * THPVariable_triplet_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs);
577: static PyObject * THPVariable_fix(PyObject* self_, PyObject* args, PyObject* kwargs);
578: static PyObject * THPVariable_fix_(PyObject* self_, PyObject* args, PyObject* kwargs);
579: static PyObject * THPVariable__has_compatible_shallow_copy_type(PyObject* self_, PyObject* args, PyObject* kwargs);
580: static PyObject * THPVariable__unique(PyObject* self_, PyObject* args, PyObject* kwargs);
581: static PyObject * THPVariable_vander(PyObject* self_, PyObject* args, PyObject* kwargs);
582: static PyObject * THPVariable_var_mean(PyObject* self_, PyObject* args, PyObject* kwargs);
583: static PyObject * THPVariable__weight_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
584: static PyObject * THPVariable__standard_gamma_grad(PyObject* self_, PyObject* args, PyObject* kwargs);
585: static PyObject * THPVariable__philox_key_split(PyObject* self_, PyObject* args, PyObject* kwargs);
586: static PyObject * THPVariable_poisson(PyObject* self_, PyObject* args, PyObject* kwargs);
587: static PyObject * THPVariable_native_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
588: static PyObject * THPVariable__sparse_softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs);
589: static PyObject * THPVariable_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
590: static PyObject * THPVariable_frexp(PyObject* self_, PyObject* args, PyObject* kwargs);
591: static PyObject * THPVariable_positive(PyObject* self_, PyObject* args, PyObject* kwargs);
592: static PyObject * THPVariable_resize_as_(PyObject* self_, PyObject* args, PyObject* kwargs);
593: static PyObject * THPVariable_zero_(PyObject* self_, PyObject* args, PyObject* kwargs);
594: static PyObject * THPVariable_sub(PyObject* self_, PyObject* args, PyObject* kwargs);
595: static PyObject * THPVariable_subtract(PyObject* self_, PyObject* args, PyObject* kwargs);
596: static PyObject * THPVariable_rsub(PyObject* self_, PyObject* args, PyObject* kwargs);
597: static PyObject * THPVariable__validate_sparse_csc_tensor_args(PyObject* self_, PyObject* args, PyObject* kwargs);
598: static PyObject * THPVariable__coalesce(PyObject* self_, PyObject* args, PyObject* kwargs);
599: static PyObject * THPVariable_unbind(PyObject* self_, PyObject* args, PyObject* kwargs);
600: static PyObject * THPVariable_quantize_per_tensor_dynamic(PyObject* self_, PyObject* args, PyObject* kwargs);
601: static PyObject * THPVariable_quantize_per_tensor(PyObject* self_, PyObject* args, PyObject* kwargs);
602: static PyObject * THPVariable_quantize_per_channel(PyObject* self_, PyObject* args, PyObject* kwargs);
603: static PyObject * THPVariable_q_zero_point(PyObject* self_, PyObject* args, PyObject* kwargs);
604: static PyObject * THPVariable_int_repr(PyObject* self_, PyObject* args, PyObject* kwargs);
605: static PyObject * THPVariable__make_per_tensor_quantized_tensor(PyObject* self_, PyObject* args, PyObject* kwargs);
606: static PyObject * THPVariable__make_per_channel_quantized_tensor(PyObject* self_, PyObject* args, PyObject* kwargs);
607: static PyObject * THPVariable_fake_quantize_per_channel_affine(PyObject* self_, PyObject* args, PyObject* kwargs);
608: static PyObject * THPVariable__fused_moving_avg_obs_fq_helper(PyObject* self_, PyObject* args, PyObject* kwargs);
609: static PyObject * THPVariable__choose_qparams_per_tensor(PyObject* self_, PyObject* args, PyObject* kwargs);
610: static PyObject * THPVariable_meshgrid(PyObject* self_, PyObject* args, PyObject* kwargs);
611: static PyObject * THPVariable_promote_types(PyObject* self_, PyObject* args, PyObject* kwargs);
612: static PyObject * THPVariable_gru(PyObject* self_, PyObject* args, PyObject* kwargs);
613: static PyObject * THPVariable_rnn_tanh_cell(PyObject* self_, PyObject* args, PyObject* kwargs);
614: static PyObject * THPVariable_quantized_gru_cell(PyObject* self_, PyObject* args, PyObject* kwargs);
615: static PyObject * THPVariable_masked_scatter(PyObject* self_, PyObject* args, PyObject* kwargs);
616: static PyObject * THPVariable_put(PyObject* self_, PyObject* args, PyObject* kwargs);
617: static PyObject * THPVariable___or__(PyObject* self_, PyObject* args, PyObject* kwargs);
618: static PyObject * THPVariable___xor__(PyObject* self_, PyObject* args, PyObject* kwargs);
619: static PyObject * THPVariable___lshift__(PyObject* self_, PyObject* args, PyObject* kwargs);
620: static PyObject * THPVariable_bitwise_right_shift(PyObject* self_, PyObject* args, PyObject* kwargs);
621: static PyObject * THPVariable_tril(PyObject* self_, PyObject* args, PyObject* kwargs);
622: static PyObject * THPVariable_tril_indices(PyObject* self_, PyObject* args, PyObject* kwargs);
623: static PyObject * THPVariable_not_equal(PyObject* self_, PyObject* args, PyObject* kwargs);
624: static PyObject * THPVariable_greater_equal(PyObject* self_, PyObject* args, PyObject* kwargs);
625: static PyObject * THPVariable_less(PyObject* self_, PyObject* args, PyObject* kwargs);
626: static PyObject * THPVariable_index_select(PyObject* self_, PyObject* args, PyObject* kwargs);
627: static PyObject * THPVariable_gather(PyObject* self_, PyObject* args, PyObject* kwargs);
628: static PyObject * THPVariable_addcmul(PyObject* self_, PyObject* args, PyObject* kwargs);
629: static PyObject * THPVariable_triangular_solve(PyObject* self_, PyObject* args, PyObject* kwargs);
630: static PyObject * THPVariable_swapaxes(PyObject* self_, PyObject* args, PyObject* kwargs);
631: static PyObject * THPVariable_cholesky_inverse(PyObject* self_, PyObject* args, PyObject* kwargs);
632: static PyObject * THPVariable_geqrf(PyObject* self_, PyObject* args, PyObject* kwargs);
633: static PyObject * THPVariable_orgqr(PyObject* self_, PyObject* args, PyObject* kwargs);
634: static PyObject * THPVariable_ormqr(PyObject* self_, PyObject* args, PyObject* kwargs);
635: static PyObject * THPVariable_lu_solve(PyObject* self_, PyObject* args, PyObject* kwargs);
636: static PyObject * THPVariable_lgamma(PyObject* self_, PyObject* args, PyObject* kwargs);
637: static PyObject * THPVariable_digamma(PyObject* self_, PyObject* args, PyObject* kwargs);
638: static PyObject * THPVariable_erfinv(PyObject* self_, PyObject* args, PyObject* kwargs);
639: static PyObject * THPVariable_sign(PyObject* self_, PyObject* args, PyObject* kwargs);
640: static PyObject * THPVariable_dist(PyObject* self_, PyObject* args, PyObject* kwargs);
```

- EN: The main execution path in this span is carried by `THPVariable_sqrt_`, `THPVariable_square`, `THPVariable_square_`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_sqrt_`, `THPVariable_square`, `THPVariable_square_` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 641-720

```cpp
641: static PyObject * THPVariable_arctan2(PyObject* self_, PyObject* args, PyObject* kwargs);
642: static PyObject * THPVariable_histc(PyObject* self_, PyObject* args, PyObject* kwargs);
643: static PyObject * THPVariable_histogram(PyObject* self_, PyObject* args, PyObject* kwargs);
644: static PyObject * THPVariable__histogramdd_bin_edges(PyObject* self_, PyObject* args, PyObject* kwargs);
645: static PyObject * THPVariable_histogramdd(PyObject* self_, PyObject* args, PyObject* kwargs);
646: static PyObject * THPVariable_fmax(PyObject* self_, PyObject* args, PyObject* kwargs);
647: static PyObject * THPVariable_maximum(PyObject* self_, PyObject* args, PyObject* kwargs);
648: static PyObject * THPVariable_minimum(PyObject* self_, PyObject* args, PyObject* kwargs);
649: static PyObject * THPVariable_nanquantile(PyObject* self_, PyObject* args, PyObject* kwargs);
650: static PyObject * THPVariable_msort(PyObject* self_, PyObject* args, PyObject* kwargs);
651: static PyObject * THPVariable_pow(PyObject* self_, PyObject* args, PyObject* kwargs);
652: static PyObject * THPVariable_float_power(PyObject* self_, PyObject* args, PyObject* kwargs);
653: static PyObject * THPVariable__foreach_add(PyObject* self_, PyObject* args, PyObject* kwargs);
654: static PyObject * THPVariable__foreach_add_(PyObject* self_, PyObject* args, PyObject* kwargs);
655: static PyObject * THPVariable__foreach_sub(PyObject* self_, PyObject* args, PyObject* kwargs);
656: static PyObject * THPVariable__foreach_sub_(PyObject* self_, PyObject* args, PyObject* kwargs);
657: static PyObject * THPVariable__foreach_clamp_max(PyObject* self_, PyObject* args, PyObject* kwargs);
658: static PyObject * THPVariable__foreach_clamp_max_(PyObject* self_, PyObject* args, PyObject* kwargs);
659: static PyObject * THPVariable__foreach_acos(PyObject* self_, PyObject* args, PyObject* kwargs);
660: static PyObject * THPVariable__foreach_acos_(PyObject* self_, PyObject* args, PyObject* kwargs);
661: static PyObject * THPVariable__foreach_asin(PyObject* self_, PyObject* args, PyObject* kwargs);
662: static PyObject * THPVariable__foreach_asin_(PyObject* self_, PyObject* args, PyObject* kwargs);
663: static PyObject * THPVariable__foreach_cos(PyObject* self_, PyObject* args, PyObject* kwargs);
664: static PyObject * THPVariable__foreach_cos_(PyObject* self_, PyObject* args, PyObject* kwargs);
665: static PyObject * THPVariable__foreach_cosh(PyObject* self_, PyObject* args, PyObject* kwargs);
666: static PyObject * THPVariable__foreach_cosh_(PyObject* self_, PyObject* args, PyObject* kwargs);
667: static PyObject * THPVariable__foreach_exp(PyObject* self_, PyObject* args, PyObject* kwargs);
668: static PyObject * THPVariable__foreach_exp_(PyObject* self_, PyObject* args, PyObject* kwargs);
669: static PyObject * THPVariable__foreach_floor(PyObject* self_, PyObject* args, PyObject* kwargs);
670: static PyObject * THPVariable__foreach_floor_(PyObject* self_, PyObject* args, PyObject* kwargs);
671: static PyObject * THPVariable__foreach_log1p(PyObject* self_, PyObject* args, PyObject* kwargs);
672: static PyObject * THPVariable__foreach_log1p_(PyObject* self_, PyObject* args, PyObject* kwargs);
673: static PyObject * THPVariable__foreach_log2(PyObject* self_, PyObject* args, PyObject* kwargs);
674: static PyObject * THPVariable__foreach_log2_(PyObject* self_, PyObject* args, PyObject* kwargs);
675: static PyObject * THPVariable__foreach_neg(PyObject* self_, PyObject* args, PyObject* kwargs);
676: static PyObject * THPVariable__foreach_neg_(PyObject* self_, PyObject* args, PyObject* kwargs);
677: static PyObject * THPVariable__foreach_norm(PyObject* self_, PyObject* args, PyObject* kwargs);
678: static PyObject * THPVariable__foreach_sigmoid(PyObject* self_, PyObject* args, PyObject* kwargs);
679: static PyObject * THPVariable__foreach_sigmoid_(PyObject* self_, PyObject* args, PyObject* kwargs);
680: static PyObject * THPVariable__foreach_sign(PyObject* self_, PyObject* args, PyObject* kwargs);
681: static PyObject * THPVariable__foreach_sign_(PyObject* self_, PyObject* args, PyObject* kwargs);
682: static PyObject * THPVariable__foreach_sqrt(PyObject* self_, PyObject* args, PyObject* kwargs);
683: static PyObject * THPVariable__foreach_sqrt_(PyObject* self_, PyObject* args, PyObject* kwargs);
684: static PyObject * THPVariable__foreach_tanh(PyObject* self_, PyObject* args, PyObject* kwargs);
685: static PyObject * THPVariable__foreach_tanh_(PyObject* self_, PyObject* args, PyObject* kwargs);
686: static PyObject * THPVariable__foreach_clone(PyObject* self_, PyObject* args, PyObject* kwargs);
687: static PyObject * THPVariable__convert_indices_from_csr_to_coo(PyObject* self_, PyObject* args, PyObject* kwargs);
688: static PyObject * THPVariable_mkldnn_adaptive_avg_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs);
689: static PyObject * THPVariable_column_stack(PyObject* self_, PyObject* args, PyObject* kwargs);
690: static PyObject * THPVariable_isinf(PyObject* self_, PyObject* args, PyObject* kwargs);
691: static PyObject * THPVariable_isneginf(PyObject* self_, PyObject* args, PyObject* kwargs);
692: static PyObject * THPVariable__linalg_slogdet(PyObject* self_, PyObject* args, PyObject* kwargs);
693: static PyObject * THPVariable_slogdet(PyObject* self_, PyObject* args, PyObject* kwargs);
694: static PyObject * THPVariable_inverse(PyObject* self_, PyObject* args, PyObject* kwargs);
695: static PyObject * THPVariable_inner(PyObject* self_, PyObject* args, PyObject* kwargs);
696: static PyObject * THPVariable__linalg_solve_ex(PyObject* self_, PyObject* args, PyObject* kwargs);
697: static PyObject * THPVariable__test_parallel_materialize(PyObject* self_, PyObject* args, PyObject* kwargs);
698: static PyObject * THPVariable__test_autograd_multiple_dispatch(PyObject* self_, PyObject* args, PyObject* kwargs);
699: static PyObject * THPVariable__test_autograd_multiple_dispatch_view_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
700: static PyObject * THPVariable_segment_reduce(PyObject* self_, PyObject* args, PyObject* kwargs);
701: static PyObject * THPVariable__conj_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
702: static PyObject * THPVariable__sparse_broadcast_to_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
703: static PyObject * THPVariable_select_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
704: static PyObject * THPVariable_squeeze_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
705: static PyObject * THPVariable_transpose_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
706: static PyObject * THPVariable_values_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
707: static PyObject * THPVariable_unfold_copy(PyObject* self_, PyObject* args, PyObject* kwargs);
708: static PyObject * THPVariable__fused_sdp_choice(PyObject* self_, PyObject* args, PyObject* kwargs);
709: static PyObject * THPVariable__scaled_dot_product_efficient_attention(PyObject* self_, PyObject* args, PyObject* kwargs);
710: static PyObject * THPVariable__scaled_dot_product_cudnn_attention(PyObject* self_, PyObject* args, PyObject* kwargs);
711: static PyObject * THPVariable__flash_attention_forward_no_dropout_inplace(PyObject* self_, PyObject* args, PyObject* kwargs);
712: static PyObject * THPVariable__fill_mem_eff_dropout_mask_(PyObject* self_, PyObject* args, PyObject* kwargs);
713: static PyObject * THPVariable__foobar(PyObject* self_, PyObject* args, PyObject* kwargs);
714: static PyObject * THPVariable__fused_adagrad_(PyObject* self_, PyObject* args, PyObject* kwargs);
715: static PyObject * THPVariable__propagate_xla_data(PyObject* self_, PyObject* args, PyObject* kwargs);
716: 
717: static PyMethodDef torch_functions_shard[] = {
718:   {"_cast_Char", castPyCFunctionWithKeywords(THPVariable__cast_Char), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
719:   {"_cast_Short", castPyCFunctionWithKeywords(THPVariable__cast_Short), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
720:   {"_cast_Half", castPyCFunctionWithKeywords(THPVariable__cast_Half), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `THPVariable_arctan2`, `THPVariable_histc`, `THPVariable_histogram`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_arctan2`, `THPVariable_histc`, `THPVariable_histogram` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 721-800

```cpp
721:   {"_make_dual", castPyCFunctionWithKeywords(THPVariable__make_dual), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
722:   {"_unpack_dual", castPyCFunctionWithKeywords(THPVariable__unpack_dual), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
723:   {"_functional_assert_scalar", castPyCFunctionWithKeywords(THPVariable__functional_assert_scalar), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
724:   {"_functional_assert_async", castPyCFunctionWithKeywords(THPVariable__functional_assert_async), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
725:   {"sym_constrain_range_for_size", castPyCFunctionWithKeywords(THPVariable_sym_constrain_range_for_size), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
726:   {"_functional_sym_constrain_range", castPyCFunctionWithKeywords(THPVariable__functional_sym_constrain_range), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
727:   {"_cudnn_ctc_loss", castPyCFunctionWithKeywords(THPVariable__cudnn_ctc_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
728:   {"_cudnn_init_dropout_state", castPyCFunctionWithKeywords(THPVariable__cudnn_init_dropout_state), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
729:   {"_debug_has_internal_overlap", castPyCFunctionWithKeywords(THPVariable__debug_has_internal_overlap), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
730:   {"_masked_scale", castPyCFunctionWithKeywords(THPVariable__masked_scale), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
731:   {"_sobol_engine_initialize_state_", castPyCFunctionWithKeywords(THPVariable__sobol_engine_initialize_state_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
732:   {"feature_alpha_dropout", castPyCFunctionWithKeywords(THPVariable_feature_alpha_dropout), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
733:   {"feature_alpha_dropout_", castPyCFunctionWithKeywords(THPVariable_feature_alpha_dropout_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
734:   {"abs", castPyCFunctionWithKeywords(THPVariable_abs), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
735:   {"abs_", castPyCFunctionWithKeywords(THPVariable_abs_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
736:   {"absolute", castPyCFunctionWithKeywords(THPVariable_absolute), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
737:   {"view_as_real", castPyCFunctionWithKeywords(THPVariable_view_as_real), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
738:   {"real", castPyCFunctionWithKeywords(THPVariable_real), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
739:   {"_conj_physical", castPyCFunctionWithKeywords(THPVariable__conj_physical), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
740:   {"resolve_conj", castPyCFunctionWithKeywords(THPVariable_resolve_conj), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
741:   {"_neg_view", castPyCFunctionWithKeywords(THPVariable__neg_view), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
742:   {"acos", castPyCFunctionWithKeywords(THPVariable_acos), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
743:   {"acos_", castPyCFunctionWithKeywords(THPVariable_acos_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
744:   {"arccos", castPyCFunctionWithKeywords(THPVariable_arccos), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
745:   {"arccos_", castPyCFunctionWithKeywords(THPVariable_arccos_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
746:   {"adaptive_avg_pool1d", castPyCFunctionWithKeywords(THPVariable_adaptive_avg_pool1d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
747:   {"addmv", castPyCFunctionWithKeywords(THPVariable_addmv), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
748:   {"addmv_", castPyCFunctionWithKeywords(THPVariable_addmv_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
749:   {"_is_all_true", castPyCFunctionWithKeywords(THPVariable__is_all_true), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
750:   {"_test_check_tensor", castPyCFunctionWithKeywords(THPVariable__test_check_tensor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
751:   {"argmin", castPyCFunctionWithKeywords(THPVariable_argmin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
752:   {"asinh", castPyCFunctionWithKeywords(THPVariable_asinh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
753:   {"asinh_", castPyCFunctionWithKeywords(THPVariable_asinh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
754:   {"arcsinh", castPyCFunctionWithKeywords(THPVariable_arcsinh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
755:   {"arcsinh_", castPyCFunctionWithKeywords(THPVariable_arcsinh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
756:   {"asin", castPyCFunctionWithKeywords(THPVariable_asin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
757:   {"asin_", castPyCFunctionWithKeywords(THPVariable_asin_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
758:   {"arctan", castPyCFunctionWithKeywords(THPVariable_arctan), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
759:   {"arctan_", castPyCFunctionWithKeywords(THPVariable_arctan_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
760:   {"atleast_2d", castPyCFunctionWithKeywords(THPVariable_atleast_2d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
761:   {"atleast_3d", castPyCFunctionWithKeywords(THPVariable_atleast_3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
762:   {"quantized_batch_norm", castPyCFunctionWithKeywords(THPVariable_quantized_batch_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
763:   {"_batch_norm_impl_index", castPyCFunctionWithKeywords(THPVariable__batch_norm_impl_index), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
764:   {"bincount", castPyCFunctionWithKeywords(THPVariable_bincount), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
765:   {"bitwise_not", castPyCFunctionWithKeywords(THPVariable_bitwise_not), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
766:   {"logical_or", castPyCFunctionWithKeywords(THPVariable_logical_or), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
767:   {"unsafe_chunk", castPyCFunctionWithKeywords(THPVariable_unsafe_chunk), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
768:   {"clamp", castPyCFunctionWithKeywords(THPVariable_clamp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
769:   {"clamp_", castPyCFunctionWithKeywords(THPVariable_clamp_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
770:   {"clamp_max", castPyCFunctionWithKeywords(THPVariable_clamp_max), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
771:   {"clamp_max_", castPyCFunctionWithKeywords(THPVariable_clamp_max_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
772:   {"constant_pad_nd", castPyCFunctionWithKeywords(THPVariable_constant_pad_nd), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
773:   {"convolution", castPyCFunctionWithKeywords(THPVariable_convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
774:   {"conv2d", castPyCFunctionWithKeywords(THPVariable_conv2d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
775:   {"conv_tbc", castPyCFunctionWithKeywords(THPVariable_conv_tbc), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
776:   {"conv_transpose1d", castPyCFunctionWithKeywords(THPVariable_conv_transpose1d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
777:   {"conv_transpose3d", castPyCFunctionWithKeywords(THPVariable_conv_transpose3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
778:   {"cosine_embedding_loss", castPyCFunctionWithKeywords(THPVariable_cosine_embedding_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
779:   {"cudnn_affine_grid_generator", castPyCFunctionWithKeywords(THPVariable_cudnn_affine_grid_generator), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
780:   {"cudnn_batch_norm", castPyCFunctionWithKeywords(THPVariable_cudnn_batch_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
781:   {"_mps_convolution_transpose", castPyCFunctionWithKeywords(THPVariable__mps_convolution_transpose), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
782:   {"cudnn_convolution_add_relu", castPyCFunctionWithKeywords(THPVariable_cudnn_convolution_add_relu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
783:   {"cudnn_grid_sampler", castPyCFunctionWithKeywords(THPVariable_cudnn_grid_sampler), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
784:   {"_cummax_helper", castPyCFunctionWithKeywords(THPVariable__cummax_helper), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
785:   {"cumprod", castPyCFunctionWithKeywords(THPVariable_cumprod), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
786:   {"gradient", castPyCFunctionWithKeywords(THPVariable_gradient), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
787:   {"div", castPyCFunctionWithKeywords(THPVariable_div), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
788:   {"true_divide", castPyCFunctionWithKeywords(THPVariable_true_divide), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
789:   {"_embedding_bag_forward_only", castPyCFunctionWithKeywords(THPVariable__embedding_bag_forward_only), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
790:   {"row_stack", castPyCFunctionWithKeywords(THPVariable_row_stack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
791:   {"_empty_per_channel_affine_quantized", castPyCFunctionWithKeywords(THPVariable__empty_per_channel_affine_quantized), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
792:   {"_resize_output_", castPyCFunctionWithKeywords(THPVariable__resize_output_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
793:   {"empty_strided", castPyCFunctionWithKeywords(THPVariable_empty_strided), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
794:   {"flatten", castPyCFunctionWithKeywords(THPVariable_flatten), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
795:   {"fill", castPyCFunctionWithKeywords(THPVariable_fill), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
796:   {"fill_", castPyCFunctionWithKeywords(THPVariable_fill_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
797:   {"grid_sampler_2d", castPyCFunctionWithKeywords(THPVariable_grid_sampler_2d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
798:   {"hann_window", castPyCFunctionWithKeywords(THPVariable_hann_window), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
799:   {"kaiser_window", castPyCFunctionWithKeywords(THPVariable_kaiser_window), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
800:   {"_fft_r2c", castPyCFunctionWithKeywords(THPVariable__fft_r2c), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 801-880

```cpp
801:   {"_validate_compressed_sparse_indices", castPyCFunctionWithKeywords(THPVariable__validate_compressed_sparse_indices), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
802:   {"_cufft_get_plan_cache_size", castPyCFunctionWithKeywords(THPVariable__cufft_get_plan_cache_size), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
803:   {"_unsafe_masked_index", castPyCFunctionWithKeywords(THPVariable__unsafe_masked_index), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
804:   {"index_put_", castPyCFunctionWithKeywords(THPVariable_index_put_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
805:   {"index_put", castPyCFunctionWithKeywords(THPVariable_index_put), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
806:   {"_unsafe_index_put", castPyCFunctionWithKeywords(THPVariable__unsafe_index_put), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
807:   {"is_distributed", castPyCFunctionWithKeywords(THPVariable_is_distributed), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
808:   {"is_complex", castPyCFunctionWithKeywords(THPVariable_is_complex), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
809:   {"isreal", castPyCFunctionWithKeywords(THPVariable_isreal), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
810:   {"kron", castPyCFunctionWithKeywords(THPVariable_kron), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
811:   {"native_layer_norm", castPyCFunctionWithKeywords(THPVariable_native_layer_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
812:   {"_cslt_compress", castPyCFunctionWithKeywords(THPVariable__cslt_compress), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
813:   {"_sparse_semi_structured_tile", castPyCFunctionWithKeywords(THPVariable__sparse_semi_structured_tile), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
814:   {"_sparse_semi_structured_linear", castPyCFunctionWithKeywords(THPVariable__sparse_semi_structured_linear), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
815:   {"fbgemm_linear_fp16_weight", castPyCFunctionWithKeywords(THPVariable_fbgemm_linear_fp16_weight), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
816:   {"ldexp", castPyCFunctionWithKeywords(THPVariable_ldexp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
817:   {"ldexp_", castPyCFunctionWithKeywords(THPVariable_ldexp_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
818:   {"log10", castPyCFunctionWithKeywords(THPVariable_log10), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
819:   {"log10_", castPyCFunctionWithKeywords(THPVariable_log10_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
820:   {"logaddexp2", castPyCFunctionWithKeywords(THPVariable_logaddexp2), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
821:   {"_log_softmax_backward_data", castPyCFunctionWithKeywords(THPVariable__log_softmax_backward_data), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
822:   {"logcumsumexp", castPyCFunctionWithKeywords(THPVariable_logcumsumexp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
823:   {"margin_ranking_loss", castPyCFunctionWithKeywords(THPVariable_margin_ranking_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
824:   {"matrix_exp", castPyCFunctionWithKeywords(THPVariable_matrix_exp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
825:   {"amax", castPyCFunctionWithKeywords(THPVariable_amax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
826:   {"max_pool1d", castPyCFunctionWithKeywords(THPVariable_max_pool1d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
827:   {"mkldnn_max_pool3d", castPyCFunctionWithKeywords(THPVariable_mkldnn_max_pool3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
828:   {"quantized_max_pool3d", castPyCFunctionWithKeywords(THPVariable_quantized_max_pool3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
829:   {"max_pool3d", castPyCFunctionWithKeywords(THPVariable_max_pool3d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
830:   {"median", castPyCFunctionWithKeywords(THPVariable_median), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
831:   {"_mps_convolution", castPyCFunctionWithKeywords(THPVariable__mps_convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
832:   {"miopen_convolution", castPyCFunctionWithKeywords(THPVariable_miopen_convolution), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
833:   {"miopen_convolution_transpose", castPyCFunctionWithKeywords(THPVariable_miopen_convolution_transpose), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
834:   {"miopen_convolution_add_relu", castPyCFunctionWithKeywords(THPVariable_miopen_convolution_add_relu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
835:   {"_use_miopen_ctc_loss", castPyCFunctionWithKeywords(THPVariable__use_miopen_ctc_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
836:   {"miopen_ctc_loss", castPyCFunctionWithKeywords(THPVariable_miopen_ctc_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
837:   {"mm", castPyCFunctionWithKeywords(THPVariable_mm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
838:   {"_int_mm", castPyCFunctionWithKeywords(THPVariable__int_mm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
839:   {"_weight_int4pack_mm_with_scales_and_zeros", castPyCFunctionWithKeywords(THPVariable__weight_int4pack_mm_with_scales_and_zeros), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
840:   {"_convert_weight_to_int4pack_for_cpu", castPyCFunctionWithKeywords(THPVariable__convert_weight_to_int4pack_for_cpu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
841:   {"multiply", castPyCFunctionWithKeywords(THPVariable_multiply), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
842:   {"mvlgamma", castPyCFunctionWithKeywords(THPVariable_mvlgamma), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
843:   {"narrow_copy", castPyCFunctionWithKeywords(THPVariable_narrow_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
844:   {"_native_batch_norm_legit_no_training", castPyCFunctionWithKeywords(THPVariable__native_batch_norm_legit_no_training), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
845:   {"batch_norm_elemt", castPyCFunctionWithKeywords(THPVariable_batch_norm_elemt), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
846:   {"batch_norm_gather_stats_with_counts", castPyCFunctionWithKeywords(THPVariable_batch_norm_gather_stats_with_counts), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
847:   {"batch_norm_backward_elemt", castPyCFunctionWithKeywords(THPVariable_batch_norm_backward_elemt), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
848:   {"ones_like", castPyCFunctionWithKeywords(THPVariable_ones_like), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
849:   {"pairwise_distance", castPyCFunctionWithKeywords(THPVariable_pairwise_distance), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
850:   {"cdist", castPyCFunctionWithKeywords(THPVariable_cdist), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
851:   {"cosine_similarity", castPyCFunctionWithKeywords(THPVariable_cosine_similarity), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
852:   {"movedim", castPyCFunctionWithKeywords(THPVariable_movedim), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
853:   {"adjoint", castPyCFunctionWithKeywords(THPVariable_adjoint), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
854:   {"channel_shuffle", castPyCFunctionWithKeywords(THPVariable_channel_shuffle), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
855:   {"pinverse", castPyCFunctionWithKeywords(THPVariable_pinverse), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
856:   {"poisson_nll_loss", castPyCFunctionWithKeywords(THPVariable_poisson_nll_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
857:   {"deg2rad", castPyCFunctionWithKeywords(THPVariable_deg2rad), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
858:   {"deg2rad_", castPyCFunctionWithKeywords(THPVariable_deg2rad_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
859:   {"randn_like", castPyCFunctionWithKeywords(THPVariable_randn_like), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
860:   {"repeat_interleave", castPyCFunctionWithKeywords(THPVariable_repeat_interleave), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
861:   {"_prelu_kernel", castPyCFunctionWithKeywords(THPVariable__prelu_kernel), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
862:   {"hardshrink", castPyCFunctionWithKeywords(THPVariable_hardshrink), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
863:   {"rsqrt", castPyCFunctionWithKeywords(THPVariable_rsqrt), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
864:   {"rsqrt_", castPyCFunctionWithKeywords(THPVariable_rsqrt_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
865:   {"celu", castPyCFunctionWithKeywords(THPVariable_celu), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
866:   {"celu_", castPyCFunctionWithKeywords(THPVariable_celu_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
867:   {"sinc", castPyCFunctionWithKeywords(THPVariable_sinc), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
868:   {"sinc_", castPyCFunctionWithKeywords(THPVariable_sinc_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
869:   {"sinh", castPyCFunctionWithKeywords(THPVariable_sinh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
870:   {"sinh_", castPyCFunctionWithKeywords(THPVariable_sinh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
871:   {"detach", castPyCFunctionWithKeywords(THPVariable_detach), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
872:   {"detach_", castPyCFunctionWithKeywords(THPVariable_detach_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
873:   {"_softmax", castPyCFunctionWithKeywords(THPVariable__softmax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
874:   {"_softmax_backward_data", castPyCFunctionWithKeywords(THPVariable__softmax_backward_data), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
875:   {"unsafe_split", castPyCFunctionWithKeywords(THPVariable_unsafe_split), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
876:   {"split", castPyCFunctionWithKeywords(THPVariable_split), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
877:   {"hsplit", castPyCFunctionWithKeywords(THPVariable_hsplit), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
878:   {"squeeze", castPyCFunctionWithKeywords(THPVariable_squeeze), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
879:   {"stack", castPyCFunctionWithKeywords(THPVariable_stack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
880:   {"_stack", castPyCFunctionWithKeywords(THPVariable__stack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 881-960

```cpp
881:   {"istft", castPyCFunctionWithKeywords(THPVariable_istft), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
882:   {"nansum", castPyCFunctionWithKeywords(THPVariable_nansum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
883:   {"sqrt", castPyCFunctionWithKeywords(THPVariable_sqrt), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
884:   {"sqrt_", castPyCFunctionWithKeywords(THPVariable_sqrt_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
885:   {"square", castPyCFunctionWithKeywords(THPVariable_square), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
886:   {"square_", castPyCFunctionWithKeywords(THPVariable_square_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
887:   {"t", castPyCFunctionWithKeywords(THPVariable_t), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
888:   {"tanh", castPyCFunctionWithKeywords(THPVariable_tanh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
889:   {"tanh_", castPyCFunctionWithKeywords(THPVariable_tanh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
890:   {"tensordot", castPyCFunctionWithKeywords(THPVariable_tensordot), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
891:   {"tile", castPyCFunctionWithKeywords(THPVariable_tile), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
892:   {"_mkldnn_transpose", castPyCFunctionWithKeywords(THPVariable__mkldnn_transpose), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
893:   {"_mkldnn_transpose_", castPyCFunctionWithKeywords(THPVariable__mkldnn_transpose_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
894:   {"rot90", castPyCFunctionWithKeywords(THPVariable_rot90), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
895:   {"_nested_from_padded", castPyCFunctionWithKeywords(THPVariable__nested_from_padded), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
896:   {"_nested_get_values", castPyCFunctionWithKeywords(THPVariable__nested_get_values), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
897:   {"_nested_get_offsets", castPyCFunctionWithKeywords(THPVariable__nested_get_offsets), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
898:   {"_nested_get_jagged_dummy", castPyCFunctionWithKeywords(THPVariable__nested_get_jagged_dummy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
899:   {"triplet_margin_loss", castPyCFunctionWithKeywords(THPVariable_triplet_margin_loss), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
900:   {"fix", castPyCFunctionWithKeywords(THPVariable_fix), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
901:   {"fix_", castPyCFunctionWithKeywords(THPVariable_fix_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
902:   {"_has_compatible_shallow_copy_type", castPyCFunctionWithKeywords(THPVariable__has_compatible_shallow_copy_type), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
903:   {"_unique", castPyCFunctionWithKeywords(THPVariable__unique), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
904:   {"vander", castPyCFunctionWithKeywords(THPVariable_vander), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
905:   {"var_mean", castPyCFunctionWithKeywords(THPVariable_var_mean), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
906:   {"_weight_norm", castPyCFunctionWithKeywords(THPVariable__weight_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
907:   {"_standard_gamma_grad", castPyCFunctionWithKeywords(THPVariable__standard_gamma_grad), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
908:   {"_philox_key_split", castPyCFunctionWithKeywords(THPVariable__philox_key_split), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
909:   {"poisson", castPyCFunctionWithKeywords(THPVariable_poisson), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
910:   {"native_norm", castPyCFunctionWithKeywords(THPVariable_native_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
911:   {"_sparse_softmax_backward_data", castPyCFunctionWithKeywords(THPVariable__sparse_softmax_backward_data), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
912:   {"norm", castPyCFunctionWithKeywords(THPVariable_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
913:   {"frexp", castPyCFunctionWithKeywords(THPVariable_frexp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
914:   {"positive", castPyCFunctionWithKeywords(THPVariable_positive), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
915:   {"resize_as_", castPyCFunctionWithKeywords(THPVariable_resize_as_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
916:   {"zero_", castPyCFunctionWithKeywords(THPVariable_zero_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
917:   {"sub", castPyCFunctionWithKeywords(THPVariable_sub), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
918:   {"subtract", castPyCFunctionWithKeywords(THPVariable_subtract), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
919:   {"rsub", castPyCFunctionWithKeywords(THPVariable_rsub), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
920:   {"_validate_sparse_csc_tensor_args", castPyCFunctionWithKeywords(THPVariable__validate_sparse_csc_tensor_args), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
921:   {"_coalesce", castPyCFunctionWithKeywords(THPVariable__coalesce), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
922:   {"unbind", castPyCFunctionWithKeywords(THPVariable_unbind), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
923:   {"quantize_per_tensor_dynamic", castPyCFunctionWithKeywords(THPVariable_quantize_per_tensor_dynamic), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
924:   {"quantize_per_tensor", castPyCFunctionWithKeywords(THPVariable_quantize_per_tensor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
925:   {"quantize_per_channel", castPyCFunctionWithKeywords(THPVariable_quantize_per_channel), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
926:   {"q_zero_point", castPyCFunctionWithKeywords(THPVariable_q_zero_point), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
927:   {"int_repr", castPyCFunctionWithKeywords(THPVariable_int_repr), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
928:   {"_make_per_tensor_quantized_tensor", castPyCFunctionWithKeywords(THPVariable__make_per_tensor_quantized_tensor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
929:   {"_make_per_channel_quantized_tensor", castPyCFunctionWithKeywords(THPVariable__make_per_channel_quantized_tensor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
930:   {"fake_quantize_per_channel_affine", castPyCFunctionWithKeywords(THPVariable_fake_quantize_per_channel_affine), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
931:   {"_fused_moving_avg_obs_fq_helper", castPyCFunctionWithKeywords(THPVariable__fused_moving_avg_obs_fq_helper), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
932:   {"_choose_qparams_per_tensor", castPyCFunctionWithKeywords(THPVariable__choose_qparams_per_tensor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
933:   {"meshgrid", castPyCFunctionWithKeywords(THPVariable_meshgrid), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
934:   {"promote_types", castPyCFunctionWithKeywords(THPVariable_promote_types), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
935:   {"gru", castPyCFunctionWithKeywords(THPVariable_gru), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
936:   {"rnn_tanh_cell", castPyCFunctionWithKeywords(THPVariable_rnn_tanh_cell), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
937:   {"quantized_gru_cell", castPyCFunctionWithKeywords(THPVariable_quantized_gru_cell), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
938:   {"masked_scatter", castPyCFunctionWithKeywords(THPVariable_masked_scatter), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
939:   {"put", castPyCFunctionWithKeywords(THPVariable_put), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
940:   {"__or__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___or__>), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
941:   {"__xor__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___xor__>), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
942:   {"__lshift__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___lshift__>), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
943:   {"bitwise_right_shift", castPyCFunctionWithKeywords(THPVariable_bitwise_right_shift), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
944:   {"tril", castPyCFunctionWithKeywords(THPVariable_tril), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
945:   {"tril_indices", castPyCFunctionWithKeywords(THPVariable_tril_indices), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
946:   {"not_equal", castPyCFunctionWithKeywords(THPVariable_not_equal), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
947:   {"greater_equal", castPyCFunctionWithKeywords(THPVariable_greater_equal), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
948:   {"less", castPyCFunctionWithKeywords(THPVariable_less), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
949:   {"index_select", castPyCFunctionWithKeywords(THPVariable_index_select), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
950:   {"gather", castPyCFunctionWithKeywords(THPVariable_gather), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
951:   {"addcmul", castPyCFunctionWithKeywords(THPVariable_addcmul), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
952:   {"triangular_solve", castPyCFunctionWithKeywords(THPVariable_triangular_solve), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
953:   {"swapaxes", castPyCFunctionWithKeywords(THPVariable_swapaxes), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
954:   {"cholesky_inverse", castPyCFunctionWithKeywords(THPVariable_cholesky_inverse), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
955:   {"geqrf", castPyCFunctionWithKeywords(THPVariable_geqrf), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
956:   {"orgqr", castPyCFunctionWithKeywords(THPVariable_orgqr), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
957:   {"ormqr", castPyCFunctionWithKeywords(THPVariable_ormqr), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
958:   {"lu_solve", castPyCFunctionWithKeywords(THPVariable_lu_solve), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
959:   {"lgamma", castPyCFunctionWithKeywords(THPVariable_lgamma), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
960:   {"digamma", castPyCFunctionWithKeywords(THPVariable_digamma), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 961-1040

```cpp
 961:   {"erfinv", castPyCFunctionWithKeywords(THPVariable_erfinv), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 962:   {"sign", castPyCFunctionWithKeywords(THPVariable_sign), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 963:   {"dist", castPyCFunctionWithKeywords(THPVariable_dist), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 964:   {"arctan2", castPyCFunctionWithKeywords(THPVariable_arctan2), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 965:   {"histc", castPyCFunctionWithKeywords(THPVariable_histc), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 966:   {"histogram", castPyCFunctionWithKeywords(THPVariable_histogram), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 967:   {"_histogramdd_bin_edges", castPyCFunctionWithKeywords(THPVariable__histogramdd_bin_edges), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 968:   {"histogramdd", castPyCFunctionWithKeywords(THPVariable_histogramdd), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 969:   {"fmax", castPyCFunctionWithKeywords(THPVariable_fmax), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 970:   {"maximum", castPyCFunctionWithKeywords(THPVariable_maximum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 971:   {"minimum", castPyCFunctionWithKeywords(THPVariable_minimum), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 972:   {"nanquantile", castPyCFunctionWithKeywords(THPVariable_nanquantile), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 973:   {"msort", castPyCFunctionWithKeywords(THPVariable_msort), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 974:   {"pow", castPyCFunctionWithKeywords(THPVariable_pow), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 975:   {"float_power", castPyCFunctionWithKeywords(THPVariable_float_power), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 976:   {"_foreach_add", castPyCFunctionWithKeywords(THPVariable__foreach_add), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 977:   {"_foreach_add_", castPyCFunctionWithKeywords(THPVariable__foreach_add_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 978:   {"_foreach_sub", castPyCFunctionWithKeywords(THPVariable__foreach_sub), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 979:   {"_foreach_sub_", castPyCFunctionWithKeywords(THPVariable__foreach_sub_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 980:   {"_foreach_clamp_max", castPyCFunctionWithKeywords(THPVariable__foreach_clamp_max), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 981:   {"_foreach_clamp_max_", castPyCFunctionWithKeywords(THPVariable__foreach_clamp_max_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 982:   {"_foreach_acos", castPyCFunctionWithKeywords(THPVariable__foreach_acos), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 983:   {"_foreach_acos_", castPyCFunctionWithKeywords(THPVariable__foreach_acos_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 984:   {"_foreach_asin", castPyCFunctionWithKeywords(THPVariable__foreach_asin), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 985:   {"_foreach_asin_", castPyCFunctionWithKeywords(THPVariable__foreach_asin_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 986:   {"_foreach_cos", castPyCFunctionWithKeywords(THPVariable__foreach_cos), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 987:   {"_foreach_cos_", castPyCFunctionWithKeywords(THPVariable__foreach_cos_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 988:   {"_foreach_cosh", castPyCFunctionWithKeywords(THPVariable__foreach_cosh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 989:   {"_foreach_cosh_", castPyCFunctionWithKeywords(THPVariable__foreach_cosh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 990:   {"_foreach_exp", castPyCFunctionWithKeywords(THPVariable__foreach_exp), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 991:   {"_foreach_exp_", castPyCFunctionWithKeywords(THPVariable__foreach_exp_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 992:   {"_foreach_floor", castPyCFunctionWithKeywords(THPVariable__foreach_floor), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 993:   {"_foreach_floor_", castPyCFunctionWithKeywords(THPVariable__foreach_floor_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 994:   {"_foreach_log1p", castPyCFunctionWithKeywords(THPVariable__foreach_log1p), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 995:   {"_foreach_log1p_", castPyCFunctionWithKeywords(THPVariable__foreach_log1p_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 996:   {"_foreach_log2", castPyCFunctionWithKeywords(THPVariable__foreach_log2), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 997:   {"_foreach_log2_", castPyCFunctionWithKeywords(THPVariable__foreach_log2_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 998:   {"_foreach_neg", castPyCFunctionWithKeywords(THPVariable__foreach_neg), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
 999:   {"_foreach_neg_", castPyCFunctionWithKeywords(THPVariable__foreach_neg_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1000:   {"_foreach_norm", castPyCFunctionWithKeywords(THPVariable__foreach_norm), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1001:   {"_foreach_sigmoid", castPyCFunctionWithKeywords(THPVariable__foreach_sigmoid), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1002:   {"_foreach_sigmoid_", castPyCFunctionWithKeywords(THPVariable__foreach_sigmoid_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1003:   {"_foreach_sign", castPyCFunctionWithKeywords(THPVariable__foreach_sign), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1004:   {"_foreach_sign_", castPyCFunctionWithKeywords(THPVariable__foreach_sign_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1005:   {"_foreach_sqrt", castPyCFunctionWithKeywords(THPVariable__foreach_sqrt), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1006:   {"_foreach_sqrt_", castPyCFunctionWithKeywords(THPVariable__foreach_sqrt_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1007:   {"_foreach_tanh", castPyCFunctionWithKeywords(THPVariable__foreach_tanh), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1008:   {"_foreach_tanh_", castPyCFunctionWithKeywords(THPVariable__foreach_tanh_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1009:   {"_foreach_clone", castPyCFunctionWithKeywords(THPVariable__foreach_clone), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1010:   {"_convert_indices_from_csr_to_coo", castPyCFunctionWithKeywords(THPVariable__convert_indices_from_csr_to_coo), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1011:   {"mkldnn_adaptive_avg_pool2d", castPyCFunctionWithKeywords(THPVariable_mkldnn_adaptive_avg_pool2d), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1012:   {"column_stack", castPyCFunctionWithKeywords(THPVariable_column_stack), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1013:   {"isinf", castPyCFunctionWithKeywords(THPVariable_isinf), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1014:   {"isneginf", castPyCFunctionWithKeywords(THPVariable_isneginf), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1015:   {"_linalg_slogdet", castPyCFunctionWithKeywords(THPVariable__linalg_slogdet), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1016:   {"slogdet", castPyCFunctionWithKeywords(THPVariable_slogdet), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1017:   {"inverse", castPyCFunctionWithKeywords(THPVariable_inverse), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1018:   {"inner", castPyCFunctionWithKeywords(THPVariable_inner), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1019:   {"_linalg_solve_ex", castPyCFunctionWithKeywords(THPVariable__linalg_solve_ex), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1020:   {"_test_parallel_materialize", castPyCFunctionWithKeywords(THPVariable__test_parallel_materialize), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1021:   {"_test_autograd_multiple_dispatch", castPyCFunctionWithKeywords(THPVariable__test_autograd_multiple_dispatch), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1022:   {"_test_autograd_multiple_dispatch_view_copy", castPyCFunctionWithKeywords(THPVariable__test_autograd_multiple_dispatch_view_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1023:   {"segment_reduce", castPyCFunctionWithKeywords(THPVariable_segment_reduce), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1024:   {"_conj_copy", castPyCFunctionWithKeywords(THPVariable__conj_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1025:   {"_sparse_broadcast_to_copy", castPyCFunctionWithKeywords(THPVariable__sparse_broadcast_to_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1026:   {"select_copy", castPyCFunctionWithKeywords(THPVariable_select_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1027:   {"squeeze_copy", castPyCFunctionWithKeywords(THPVariable_squeeze_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1028:   {"transpose_copy", castPyCFunctionWithKeywords(THPVariable_transpose_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1029:   {"values_copy", castPyCFunctionWithKeywords(THPVariable_values_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1030:   {"unfold_copy", castPyCFunctionWithKeywords(THPVariable_unfold_copy), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1031:   {"_fused_sdp_choice", castPyCFunctionWithKeywords(THPVariable__fused_sdp_choice), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1032:   {"_scaled_dot_product_efficient_attention", castPyCFunctionWithKeywords(THPVariable__scaled_dot_product_efficient_attention), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1033:   {"_scaled_dot_product_cudnn_attention", castPyCFunctionWithKeywords(THPVariable__scaled_dot_product_cudnn_attention), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1034:   {"_flash_attention_forward_no_dropout_inplace", castPyCFunctionWithKeywords(THPVariable__flash_attention_forward_no_dropout_inplace), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1035:   {"_fill_mem_eff_dropout_mask_", castPyCFunctionWithKeywords(THPVariable__fill_mem_eff_dropout_mask_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1036:   {"_foobar", castPyCFunctionWithKeywords(THPVariable__foobar), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1037:   {"_fused_adagrad_", castPyCFunctionWithKeywords(THPVariable__fused_adagrad_), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1038:   {"_propagate_xla_data", castPyCFunctionWithKeywords(THPVariable__propagate_xla_data), METH_VARARGS | METH_KEYWORDS | METH_STATIC, nullptr},
1039: };
1040: 
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1041-1120

```cpp
1041: void gatherTorchFunctions_1(std::vector<PyMethodDef> &torch_functions) {
1042:   constexpr size_t num_functions = sizeof(torch_functions_shard) / sizeof(torch_functions_shard[0]);
1043:   torch_functions.insert(
1044:     torch_functions.end(),
1045:     torch_functions_shard,
1046:     torch_functions_shard + num_functions);
1047: }
1048: 
1049: // generated methods start here
1050: 
1051: // _cast_Char
1052: static PyObject * THPVariable__cast_Char(PyObject* self_, PyObject* args, PyObject* kwargs)
1053: {
1054:   HANDLE_TH_ERRORS
1055:   static PythonArgParser parser({
1056:     "_cast_Char(Tensor input, bool non_blocking=False)",
1057:   }, /*traceable=*/true);
1058: 
1059:   ParsedArgs<2> parsed_args;
1060:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1061:   if(_r.has_torch_function()) {
1062:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1063:   }
1064:   // aten::_cast_Char(Tensor self, bool non_blocking=False) -> Tensor
1065: 
1066:   auto dispatch__cast_Char = [](const at::Tensor & self, bool non_blocking) -> at::Tensor {
1067:     pybind11::gil_scoped_release no_gil;
1068:     return at::_cast_Char(self, non_blocking);
1069:   };
1070:   return wrap(dispatch__cast_Char(_r.tensor(0), _r.toBool(1)));
1071:   Py_RETURN_NONE;
1072:   END_HANDLE_TH_ERRORS
1073: }
1074: 
1075: // _cast_Short
1076: static PyObject * THPVariable__cast_Short(PyObject* self_, PyObject* args, PyObject* kwargs)
1077: {
1078:   HANDLE_TH_ERRORS
1079:   static PythonArgParser parser({
1080:     "_cast_Short(Tensor input, bool non_blocking=False)",
1081:   }, /*traceable=*/true);
1082: 
1083:   ParsedArgs<2> parsed_args;
1084:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1085:   if(_r.has_torch_function()) {
1086:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1087:   }
1088:   // aten::_cast_Short(Tensor self, bool non_blocking=False) -> Tensor
1089: 
1090:   auto dispatch__cast_Short = [](const at::Tensor & self, bool non_blocking) -> at::Tensor {
1091:     pybind11::gil_scoped_release no_gil;
1092:     return at::_cast_Short(self, non_blocking);
1093:   };
1094:   return wrap(dispatch__cast_Short(_r.tensor(0), _r.toBool(1)));
1095:   Py_RETURN_NONE;
1096:   END_HANDLE_TH_ERRORS
1097: }
1098: 
1099: // _cast_Half
1100: static PyObject * THPVariable__cast_Half(PyObject* self_, PyObject* args, PyObject* kwargs)
1101: {
1102:   HANDLE_TH_ERRORS
1103:   static PythonArgParser parser({
1104:     "_cast_Half(Tensor input, bool non_blocking=False)",
1105:   }, /*traceable=*/true);
1106: 
1107:   ParsedArgs<2> parsed_args;
1108:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1109:   if(_r.has_torch_function()) {
1110:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1111:   }
1112:   // aten::_cast_Half(Tensor self, bool non_blocking=False) -> Tensor
1113: 
1114:   auto dispatch__cast_Half = [](const at::Tensor & self, bool non_blocking) -> at::Tensor {
1115:     pybind11::gil_scoped_release no_gil;
1116:     return at::_cast_Half(self, non_blocking);
1117:   };
1118:   return wrap(dispatch__cast_Half(_r.tensor(0), _r.toBool(1)));
1119:   Py_RETURN_NONE;
1120:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `gatherTorchFunctions_1`, `THPVariable__cast_Char`, `parser`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `gatherTorchFunctions_1`, `THPVariable__cast_Char`, `parser` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1121-1200

```cpp
1121: }
1122: 
1123: // _make_dual
1124: static PyObject * THPVariable__make_dual(PyObject* self_, PyObject* args, PyObject* kwargs)
1125: {
1126:   HANDLE_TH_ERRORS
1127:   static PythonArgParser parser({
1128:     "_make_dual(Tensor primal, Tensor tangent, int64_t level)",
1129:   }, /*traceable=*/true);
1130: 
1131:   ParsedArgs<3> parsed_args;
1132:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1133:   if(_r.has_torch_function()) {
1134:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1135:   }
1136:   // aten::_make_dual(Tensor(a) primal, Tensor tangent, int level) -> Tensor(a)
1137: 
1138:   auto dispatch__make_dual = [](const at::Tensor & primal, const at::Tensor & tangent, int64_t level) -> at::Tensor {
1139:     pybind11::gil_scoped_release no_gil;
1140:     return at::_make_dual(primal, tangent, level);
1141:   };
1142:   return wrap(dispatch__make_dual(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
1143:   Py_RETURN_NONE;
1144:   END_HANDLE_TH_ERRORS
1145: }
1146: 
1147: // _unpack_dual
1148: static PyObject * THPVariable__unpack_dual(PyObject* self_, PyObject* args, PyObject* kwargs)
1149: {
1150:   HANDLE_TH_ERRORS
1151:   static PyTypeObject* NamedTuple = generated::get__unpack_dual_structseq();
1152:   static PythonArgParser parser({
1153:     "_unpack_dual(Tensor dual, int64_t level)",
1154:   }, /*traceable=*/true);
1155: 
1156:   ParsedArgs<2> parsed_args;
1157:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1158:   if(_r.has_torch_function()) {
1159:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1160:   }
1161:   // aten::_unpack_dual(Tensor(a) dual, int level) -> (Tensor(a) primal, Tensor tangent)
1162: 
1163:   auto dispatch__unpack_dual = [](const at::Tensor & dual, int64_t level) -> ::std::tuple<at::Tensor,at::Tensor> {
1164:     pybind11::gil_scoped_release no_gil;
1165:     return at::_unpack_dual(dual, level);
1166:   };
1167:   return wrap(NamedTuple, dispatch__unpack_dual(_r.tensor(0), _r.toInt64(1)));
1168:   Py_RETURN_NONE;
1169:   END_HANDLE_TH_ERRORS
1170: }
1171: 
1172: // _functional_assert_scalar
1173: static PyObject * THPVariable__functional_assert_scalar(PyObject* self_, PyObject* args, PyObject* kwargs)
1174: {
1175:   HANDLE_TH_ERRORS
1176:   static PythonArgParser parser({
1177:     "_functional_assert_scalar(Scalar self, c10::string_view assert_msg, Tensor dep_token)",
1178:   }, /*traceable=*/true);
1179: 
1180:   ParsedArgs<3> parsed_args;
1181:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1182:   if(_r.has_torch_function()) {
1183:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1184:   }
1185:   // aten::_functional_assert_scalar(Scalar self, str assert_msg, Tensor dep_token) -> Tensor
1186: 
1187:   auto dispatch__functional_assert_scalar = [](const at::Scalar & self, c10::string_view assert_msg, const at::Tensor & dep_token) -> at::Tensor {
1188:     pybind11::gil_scoped_release no_gil;
1189:     return at::_functional_assert_scalar(self, assert_msg, dep_token);
1190:   };
1191:   return wrap(dispatch__functional_assert_scalar(_r.scalar(0), _r.stringView(1), _r.tensor(2)));
1192:   Py_RETURN_NONE;
1193:   END_HANDLE_TH_ERRORS
1194: }
1195: 
1196: // _functional_assert_async
1197: static PyObject * THPVariable__functional_assert_async(PyObject* self_, PyObject* args, PyObject* kwargs)
1198: {
1199:   HANDLE_TH_ERRORS
1200:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable__make_dual`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__make_dual`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1201-1280

```cpp
1201:     "_functional_assert_async(Tensor input, c10::string_view assert_msg, Tensor dep_token)",
1202:   }, /*traceable=*/true);
1203: 
1204:   ParsedArgs<3> parsed_args;
1205:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1206:   if(_r.has_torch_function()) {
1207:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1208:   }
1209:   // aten::_functional_assert_async.msg(Tensor self, str assert_msg, Tensor dep_token) -> Tensor
1210: 
1211:   auto dispatch__functional_assert_async = [](const at::Tensor & self, c10::string_view assert_msg, const at::Tensor & dep_token) -> at::Tensor {
1212:     pybind11::gil_scoped_release no_gil;
1213:     return at::_functional_assert_async(self, assert_msg, dep_token);
1214:   };
1215:   return wrap(dispatch__functional_assert_async(_r.tensor(0), _r.stringView(1), _r.tensor(2)));
1216:   Py_RETURN_NONE;
1217:   END_HANDLE_TH_ERRORS
1218: }
1219: 
1220: // sym_constrain_range_for_size
1221: static PyObject * THPVariable_sym_constrain_range_for_size(PyObject* self_, PyObject* args, PyObject* kwargs)
1222: {
1223:   HANDLE_TH_ERRORS
1224:   static PythonArgParser parser({
1225:     "sym_constrain_range_for_size(Scalar size, *, int64_t? min=None, int64_t? max=None)",
1226:   }, /*traceable=*/false);
1227: 
1228:   ParsedArgs<3> parsed_args;
1229:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1230:   if(_r.has_torch_function()) {
1231:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1232:   }
1233:   // aten::sym_constrain_range_for_size(Scalar size, *, int? min=None, int? max=None) -> ()
1234: 
1235:   auto dispatch_sym_constrain_range_for_size = [](const at::Scalar & size, ::std::optional<int64_t> min, ::std::optional<int64_t> max) -> void {
1236:     pybind11::gil_scoped_release no_gil;
1237:     at::sym_constrain_range_for_size(size, min, max);
1238:   };
1239:   dispatch_sym_constrain_range_for_size(_r.scalar(0), _r.toInt64Optional(1), _r.toInt64Optional(2));
1240:   Py_RETURN_NONE;
1241:   Py_RETURN_NONE;
1242:   END_HANDLE_TH_ERRORS
1243: }
1244: 
1245: // _functional_sym_constrain_range
1246: static PyObject * THPVariable__functional_sym_constrain_range(PyObject* self_, PyObject* args, PyObject* kwargs)
1247: {
1248:   HANDLE_TH_ERRORS
1249:   static PythonArgParser parser({
1250:     "_functional_sym_constrain_range(Scalar size, int64_t? min, int64_t? max, Tensor dep_token)",
1251:   }, /*traceable=*/true);
1252: 
1253:   ParsedArgs<4> parsed_args;
1254:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1255:   if(_r.has_torch_function()) {
1256:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1257:   }
1258:   // aten::_functional_sym_constrain_range(Scalar size, int? min, int? max, Tensor dep_token) -> Tensor
1259: 
1260:   auto dispatch__functional_sym_constrain_range = [](const at::Scalar & size, ::std::optional<int64_t> min, ::std::optional<int64_t> max, const at::Tensor & dep_token) -> at::Tensor {
1261:     pybind11::gil_scoped_release no_gil;
1262:     return at::_functional_sym_constrain_range(size, min, max, dep_token);
1263:   };
1264:   return wrap(dispatch__functional_sym_constrain_range(_r.scalar(0), _r.toInt64Optional(1), _r.toInt64Optional(2), _r.tensor(3)));
1265:   Py_RETURN_NONE;
1266:   END_HANDLE_TH_ERRORS
1267: }
1268: 
1269: \
1270: // _cudnn_ctc_loss
1271: static PyObject * THPVariable__cudnn_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
1272: {
1273:   HANDLE_TH_ERRORS
1274:   static PythonArgParser parser({
1275:     "_cudnn_ctc_loss(Tensor log_probs, Tensor targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t blank, bool deterministic, bool zero_infinity)",
1276:     "_cudnn_ctc_loss(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int64_t blank, bool deterministic, bool zero_infinity)",
1277:   }, /*traceable=*/true);
1278: 
1279:   ParsedArgs<7> parsed_args;
1280:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_functional_assert_async`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_functional_assert_async`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1281-1360

```cpp
1281:   if(_r.has_torch_function()) {
1282:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1283:   }
1284:   switch (_r.idx) {
1285:     case 0: {
1286:       // aten::_cudnn_ctc_loss(Tensor log_probs, Tensor targets, int[] input_lengths, int[] target_lengths, int blank, bool deterministic, bool zero_infinity) -> (Tensor, Tensor)
1287: 
1288:       auto dispatch__cudnn_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, at::IntArrayRef input_lengths, at::IntArrayRef target_lengths, int64_t blank, bool deterministic, bool zero_infinity) -> ::std::tuple<at::Tensor,at::Tensor> {
1289:         pybind11::gil_scoped_release no_gil;
1290:         return at::_cudnn_ctc_loss(log_probs, targets, input_lengths, target_lengths, blank, deterministic, zero_infinity);
1291:       };
1292:       return wrap(dispatch__cudnn_ctc_loss(_r.tensor(0), _r.tensor(1), _r.intlist(2), _r.intlist(3), _r.toInt64(4), _r.toBool(5), _r.toBool(6)));
1293:     }
1294:     case 1: {
1295:       // aten::_cudnn_ctc_loss.Tensor(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int blank, bool deterministic, bool zero_infinity) -> (Tensor, Tensor)
1296: 
1297:       auto dispatch__cudnn_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, const at::Tensor & input_lengths, const at::Tensor & target_lengths, int64_t blank, bool deterministic, bool zero_infinity) -> ::std::tuple<at::Tensor,at::Tensor> {
1298:         pybind11::gil_scoped_release no_gil;
1299:         return at::_cudnn_ctc_loss(log_probs, targets, input_lengths, target_lengths, blank, deterministic, zero_infinity);
1300:       };
1301:       return wrap(dispatch__cudnn_ctc_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.toInt64(4), _r.toBool(5), _r.toBool(6)));
1302:     }
1303:   }
1304:   Py_RETURN_NONE;
1305:   END_HANDLE_TH_ERRORS
1306: }
1307: 
1308: // _cudnn_init_dropout_state
1309: static PyObject * THPVariable__cudnn_init_dropout_state(PyObject* self_, PyObject* args, PyObject* kwargs)
1310: {
1311:   HANDLE_TH_ERRORS
1312:   static PythonArgParser parser({
1313:     "_cudnn_init_dropout_state(double dropout, bool train, int64_t dropout_seed, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
1314:   }, /*traceable=*/true);
1315: 
1316:   ParsedArgs<8> parsed_args;
1317:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1318:   if(_r.has_torch_function()) {
1319:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1320:   }
1321:   // aten::_cudnn_init_dropout_state(float dropout, bool train, int dropout_seed, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False) -> Tensor
1322:   const auto options = TensorOptions()
1323:       .dtype(_r.scalartypeOptional(3))
1324:       .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
1325:       .layout(_r.layoutOptional(4))
1326:       .requires_grad(_r.toBool(7))
1327:       .pinned_memory(_r.toBool(6));
1328:   torch::utils::maybe_initialize_device(options);
1329: 
1330:   auto dispatch__cudnn_init_dropout_state = [](double dropout, bool train, int64_t dropout_seed, at::TensorOptions options) -> at::Tensor {
1331:     pybind11::gil_scoped_release no_gil;
1332:     return torch::_cudnn_init_dropout_state(dropout, train, dropout_seed, options);
1333:   };
1334:   return wrap(dispatch__cudnn_init_dropout_state(_r.toDouble(0), _r.toBool(1), _r.toInt64(2), options));
1335:   Py_RETURN_NONE;
1336:   END_HANDLE_TH_ERRORS
1337: }
1338: 
1339: // _debug_has_internal_overlap
1340: static PyObject * THPVariable__debug_has_internal_overlap(PyObject* self_, PyObject* args, PyObject* kwargs)
1341: {
1342:   HANDLE_TH_ERRORS
1343:   static PythonArgParser parser({
1344:     "_debug_has_internal_overlap(Tensor input)",
1345:   }, /*traceable=*/false);
1346: 
1347:   ParsedArgs<1> parsed_args;
1348:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1349:   if(_r.has_torch_function()) {
1350:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1351:   }
1352:   // aten::_debug_has_internal_overlap(Tensor self) -> int
1353: 
1354:   auto dispatch__debug_has_internal_overlap = [](const at::Tensor & self) -> int64_t {
1355:     pybind11::gil_scoped_release no_gil;
1356:     return at::_debug_has_internal_overlap(self);
1357:   };
1358:   return wrap(dispatch__debug_has_internal_overlap(_r.tensor(0)));
1359:   Py_RETURN_NONE;
1360:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_cudnn_ctc_loss`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_cudnn_ctc_loss`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 1361-1440

```cpp
1361: }
1362: 
1363: // _masked_scale
1364: static PyObject * THPVariable__masked_scale(PyObject* self_, PyObject* args, PyObject* kwargs)
1365: {
1366:   HANDLE_TH_ERRORS
1367:   static PythonArgParser parser({
1368:     "_masked_scale(Tensor input, Tensor mask, double scale)",
1369:   }, /*traceable=*/true);
1370: 
1371:   ParsedArgs<3> parsed_args;
1372:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1373:   if(_r.has_torch_function()) {
1374:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1375:   }
1376:   // aten::_masked_scale(Tensor self, Tensor mask, float scale) -> Tensor
1377: 
1378:   auto dispatch__masked_scale = [](const at::Tensor & self, const at::Tensor & mask, double scale) -> at::Tensor {
1379:     pybind11::gil_scoped_release no_gil;
1380:     return at::_masked_scale(self, mask, scale);
1381:   };
1382:   return wrap(dispatch__masked_scale(_r.tensor(0), _r.tensor(1), _r.toDouble(2)));
1383:   Py_RETURN_NONE;
1384:   END_HANDLE_TH_ERRORS
1385: }
1386: 
1387: // _sobol_engine_initialize_state_
1388: static PyObject * THPVariable__sobol_engine_initialize_state_(PyObject* self_, PyObject* args, PyObject* kwargs)
1389: {
1390:   HANDLE_TH_ERRORS
1391:   static PythonArgParser parser({
1392:     "_sobol_engine_initialize_state_(Tensor input, int64_t dimension)",
1393:   }, /*traceable=*/true);
1394: 
1395:   ParsedArgs<2> parsed_args;
1396:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1397:   if(_r.has_torch_function()) {
1398:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1399:   }
1400:   // aten::_sobol_engine_initialize_state_(Tensor(a!) self, int dimension) -> Tensor(a!)
1401: 
1402:   auto dispatch__sobol_engine_initialize_state_ = [](at::Tensor self, int64_t dimension) -> at::Tensor {
1403:     pybind11::gil_scoped_release no_gil;
1404:     return at::_sobol_engine_initialize_state_(self, dimension);
1405:   };
1406:   return wrap(dispatch__sobol_engine_initialize_state_(_r.tensor(0), _r.toInt64(1)));
1407:   Py_RETURN_NONE;
1408:   END_HANDLE_TH_ERRORS
1409: }
1410: 
1411: // feature_alpha_dropout
1412: static PyObject * THPVariable_feature_alpha_dropout(PyObject* self_, PyObject* args, PyObject* kwargs)
1413: {
1414:   HANDLE_TH_ERRORS
1415:   static PythonArgParser parser({
1416:     "feature_alpha_dropout(Tensor input, double p, bool train)",
1417:   }, /*traceable=*/true);
1418: 
1419:   ParsedArgs<3> parsed_args;
1420:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1421:   if(_r.has_torch_function()) {
1422:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1423:   }
1424:   // aten::feature_alpha_dropout(Tensor input, float p, bool train) -> Tensor
1425: 
1426:   auto dispatch_feature_alpha_dropout = [](const at::Tensor & input, double p, bool train) -> at::Tensor {
1427:     pybind11::gil_scoped_release no_gil;
1428:     return at::feature_alpha_dropout(input, p, train);
1429:   };
1430:   return wrap(dispatch_feature_alpha_dropout(_r.tensor(0), _r.toDouble(1), _r.toBool(2)));
1431:   Py_RETURN_NONE;
1432:   END_HANDLE_TH_ERRORS
1433: }
1434: 
1435: // feature_alpha_dropout_
1436: static PyObject * THPVariable_feature_alpha_dropout_(PyObject* self_, PyObject* args, PyObject* kwargs)
1437: {
1438:   HANDLE_TH_ERRORS
1439:   static PythonArgParser parser({
1440:     "feature_alpha_dropout_(Tensor input, double p, bool train)",
```

- EN: The main execution path in this span is carried by `THPVariable__masked_scale`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__masked_scale`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1441-1520

```cpp
1441:   }, /*traceable=*/true);
1442: 
1443:   ParsedArgs<3> parsed_args;
1444:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1445:   if(_r.has_torch_function()) {
1446:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1447:   }
1448:   // aten::feature_alpha_dropout_(Tensor(a!) self, float p, bool train) -> Tensor(a!)
1449: 
1450:   auto dispatch_feature_alpha_dropout_ = [](at::Tensor self, double p, bool train) -> at::Tensor {
1451:     pybind11::gil_scoped_release no_gil;
1452:     return at::feature_alpha_dropout_(self, p, train);
1453:   };
1454:   return wrap(dispatch_feature_alpha_dropout_(_r.tensor(0), _r.toDouble(1), _r.toBool(2)));
1455:   Py_RETURN_NONE;
1456:   END_HANDLE_TH_ERRORS
1457: }
1458: 
1459: // abs
1460: static PyObject * THPVariable_abs(PyObject* self_, PyObject* args, PyObject* kwargs)
1461: {
1462:   HANDLE_TH_ERRORS
1463:   static PythonArgParser parser({
1464:     "abs(Tensor input, *, Tensor out=None)",
1465:   }, /*traceable=*/true);
1466: 
1467:   ParsedArgs<2> parsed_args;
1468:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1469:   if(_r.has_torch_function()) {
1470:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1471:   }
1472:   if (_r.isNone(1)) {
1473:     // aten::abs(Tensor self) -> Tensor
1474: 
1475:     auto dispatch_abs = [](const at::Tensor & self) -> at::Tensor {
1476:       pybind11::gil_scoped_release no_gil;
1477:       return self.abs();
1478:     };
1479:     return wrap(dispatch_abs(_r.tensor(0)));
1480:   } else {
1481:     // aten::abs.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1482: 
1483:     auto dispatch_abs_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1484:       pybind11::gil_scoped_release no_gil;
1485:       return at::abs_out(out, self);
1486:     };
1487:     return wrap(dispatch_abs_out(_r.tensor(1), _r.tensor(0)));
1488:   }
1489:   Py_RETURN_NONE;
1490:   END_HANDLE_TH_ERRORS
1491: }
1492: 
1493: // abs_
1494: static PyObject * THPVariable_abs_(PyObject* self_, PyObject* args, PyObject* kwargs)
1495: {
1496:   HANDLE_TH_ERRORS
1497:   static PythonArgParser parser({
1498:     "abs_(Tensor input)",
1499:   }, /*traceable=*/true);
1500: 
1501:   ParsedArgs<1> parsed_args;
1502:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1503:   if(_r.has_torch_function()) {
1504:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1505:   }
1506:   // aten::abs_(Tensor(a!) self) -> Tensor(a!)
1507: 
1508:   auto dispatch_abs_ = [](at::Tensor self) -> at::Tensor {
1509:     pybind11::gil_scoped_release no_gil;
1510:     return self.abs_();
1511:   };
1512:   return wrap(dispatch_abs_(_r.tensor(0)));
1513:   Py_RETURN_NONE;
1514:   END_HANDLE_TH_ERRORS
1515: }
1516: 
1517: // absolute
1518: static PyObject * THPVariable_absolute(PyObject* self_, PyObject* args, PyObject* kwargs)
1519: {
1520:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `feature_alpha_dropout_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `feature_alpha_dropout_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1521-1600

```cpp
1521:   static PythonArgParser parser({
1522:     "absolute(Tensor input, *, Tensor out=None)",
1523:   }, /*traceable=*/true);
1524: 
1525:   ParsedArgs<2> parsed_args;
1526:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1527:   if(_r.has_torch_function()) {
1528:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1529:   }
1530:   if (_r.isNone(1)) {
1531:     // aten::absolute(Tensor self) -> Tensor
1532: 
1533:     auto dispatch_absolute = [](const at::Tensor & self) -> at::Tensor {
1534:       pybind11::gil_scoped_release no_gil;
1535:       return self.absolute();
1536:     };
1537:     return wrap(dispatch_absolute(_r.tensor(0)));
1538:   } else {
1539:     // aten::absolute.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1540: 
1541:     auto dispatch_absolute_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1542:       pybind11::gil_scoped_release no_gil;
1543:       return at::absolute_out(out, self);
1544:     };
1545:     return wrap(dispatch_absolute_out(_r.tensor(1), _r.tensor(0)));
1546:   }
1547:   Py_RETURN_NONE;
1548:   END_HANDLE_TH_ERRORS
1549: }
1550: 
1551: // view_as_real
1552: static PyObject * THPVariable_view_as_real(PyObject* self_, PyObject* args, PyObject* kwargs)
1553: {
1554:   HANDLE_TH_ERRORS
1555:   static PythonArgParser parser({
1556:     "view_as_real(Tensor input)",
1557:   }, /*traceable=*/true);
1558: 
1559:   ParsedArgs<1> parsed_args;
1560:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1561:   if(_r.has_torch_function()) {
1562:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1563:   }
1564:   // aten::view_as_real(Tensor(a) self) -> Tensor(a)
1565: 
1566:   auto dispatch_view_as_real = [](const at::Tensor & self) -> at::Tensor {
1567:     pybind11::gil_scoped_release no_gil;
1568:     return at::view_as_real(self);
1569:   };
1570:   return wrap(dispatch_view_as_real(_r.tensor(0)));
1571:   Py_RETURN_NONE;
1572:   END_HANDLE_TH_ERRORS
1573: }
1574: 
1575: // real
1576: static PyObject * THPVariable_real(PyObject* self_, PyObject* args, PyObject* kwargs)
1577: {
1578:   HANDLE_TH_ERRORS
1579:   static PythonArgParser parser({
1580:     "real(Tensor input)",
1581:   }, /*traceable=*/true);
1582: 
1583:   ParsedArgs<1> parsed_args;
1584:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1585:   if(_r.has_torch_function()) {
1586:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1587:   }
1588:   // aten::real(Tensor(a) self) -> Tensor(a)
1589: 
1590:   auto dispatch_real = [](const at::Tensor & self) -> at::Tensor {
1591:     pybind11::gil_scoped_release no_gil;
1592:     return at::real(self);
1593:   };
1594:   return wrap(dispatch_real(_r.tensor(0)));
1595:   Py_RETURN_NONE;
1596:   END_HANDLE_TH_ERRORS
1597: }
1598: 
1599: // _conj_physical
1600: static PyObject * THPVariable__conj_physical(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `absolute`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `absolute` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1601-1680

```cpp
1601: {
1602:   HANDLE_TH_ERRORS
1603:   static PythonArgParser parser({
1604:     "_conj_physical(Tensor input)",
1605:   }, /*traceable=*/true);
1606: 
1607:   ParsedArgs<1> parsed_args;
1608:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1609:   if(_r.has_torch_function()) {
1610:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1611:   }
1612:   // aten::_conj_physical(Tensor self) -> Tensor
1613: 
1614:   auto dispatch__conj_physical = [](const at::Tensor & self) -> at::Tensor {
1615:     pybind11::gil_scoped_release no_gil;
1616:     return self._conj_physical();
1617:   };
1618:   return wrap(dispatch__conj_physical(_r.tensor(0)));
1619:   Py_RETURN_NONE;
1620:   END_HANDLE_TH_ERRORS
1621: }
1622: 
1623: // resolve_conj
1624: static PyObject * THPVariable_resolve_conj(PyObject* self_, PyObject* args, PyObject* kwargs)
1625: {
1626:   HANDLE_TH_ERRORS
1627:   static PythonArgParser parser({
1628:     "resolve_conj(Tensor input)",
1629:   }, /*traceable=*/true);
1630: 
1631:   ParsedArgs<1> parsed_args;
1632:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1633:   if(_r.has_torch_function()) {
1634:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1635:   }
1636:   // aten::resolve_conj(Tensor(a) self) -> Tensor(a)
1637: 
1638:   auto dispatch_resolve_conj = [](const at::Tensor & self) -> at::Tensor {
1639:     pybind11::gil_scoped_release no_gil;
1640:     return self.resolve_conj();
1641:   };
1642:   return wrap(dispatch_resolve_conj(_r.tensor(0)));
1643:   Py_RETURN_NONE;
1644:   END_HANDLE_TH_ERRORS
1645: }
1646: 
1647: // _neg_view
1648: static PyObject * THPVariable__neg_view(PyObject* self_, PyObject* args, PyObject* kwargs)
1649: {
1650:   HANDLE_TH_ERRORS
1651:   static PythonArgParser parser({
1652:     "_neg_view(Tensor input)",
1653:   }, /*traceable=*/true);
1654: 
1655:   ParsedArgs<1> parsed_args;
1656:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1657:   if(_r.has_torch_function()) {
1658:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1659:   }
1660:   // aten::_neg_view(Tensor(a) self) -> Tensor(a)
1661: 
1662:   auto dispatch__neg_view = [](const at::Tensor & self) -> at::Tensor {
1663:     pybind11::gil_scoped_release no_gil;
1664:     return self._neg_view();
1665:   };
1666:   return wrap(dispatch__neg_view(_r.tensor(0)));
1667:   Py_RETURN_NONE;
1668:   END_HANDLE_TH_ERRORS
1669: }
1670: 
1671: // acos
1672: static PyObject * THPVariable_acos(PyObject* self_, PyObject* args, PyObject* kwargs)
1673: {
1674:   HANDLE_TH_ERRORS
1675:   static PythonArgParser parser({
1676:     "acos(Tensor input, *, Tensor out=None)",
1677:   }, /*traceable=*/true);
1678: 
1679:   ParsedArgs<2> parsed_args;
1680:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_conj_physical`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_conj_physical` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1760

```cpp
1681:   if(_r.has_torch_function()) {
1682:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1683:   }
1684:   if (_r.isNone(1)) {
1685:     // aten::acos(Tensor self) -> Tensor
1686: 
1687:     auto dispatch_acos = [](const at::Tensor & self) -> at::Tensor {
1688:       pybind11::gil_scoped_release no_gil;
1689:       return self.acos();
1690:     };
1691:     return wrap(dispatch_acos(_r.tensor(0)));
1692:   } else {
1693:     // aten::acos.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1694: 
1695:     auto dispatch_acos_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1696:       pybind11::gil_scoped_release no_gil;
1697:       return at::acos_out(out, self);
1698:     };
1699:     return wrap(dispatch_acos_out(_r.tensor(1), _r.tensor(0)));
1700:   }
1701:   Py_RETURN_NONE;
1702:   END_HANDLE_TH_ERRORS
1703: }
1704: 
1705: // acos_
1706: static PyObject * THPVariable_acos_(PyObject* self_, PyObject* args, PyObject* kwargs)
1707: {
1708:   HANDLE_TH_ERRORS
1709:   static PythonArgParser parser({
1710:     "acos_(Tensor input)",
1711:   }, /*traceable=*/true);
1712: 
1713:   ParsedArgs<1> parsed_args;
1714:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1715:   if(_r.has_torch_function()) {
1716:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1717:   }
1718:   // aten::acos_(Tensor(a!) self) -> Tensor(a!)
1719: 
1720:   auto dispatch_acos_ = [](at::Tensor self) -> at::Tensor {
1721:     pybind11::gil_scoped_release no_gil;
1722:     return self.acos_();
1723:   };
1724:   return wrap(dispatch_acos_(_r.tensor(0)));
1725:   Py_RETURN_NONE;
1726:   END_HANDLE_TH_ERRORS
1727: }
1728: 
1729: // arccos
1730: static PyObject * THPVariable_arccos(PyObject* self_, PyObject* args, PyObject* kwargs)
1731: {
1732:   HANDLE_TH_ERRORS
1733:   static PythonArgParser parser({
1734:     "arccos(Tensor input, *, Tensor out=None)",
1735:   }, /*traceable=*/true);
1736: 
1737:   ParsedArgs<2> parsed_args;
1738:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1739:   if(_r.has_torch_function()) {
1740:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1741:   }
1742:   if (_r.isNone(1)) {
1743:     // aten::arccos(Tensor self) -> Tensor
1744: 
1745:     auto dispatch_arccos = [](const at::Tensor & self) -> at::Tensor {
1746:       pybind11::gil_scoped_release no_gil;
1747:       return self.arccos();
1748:     };
1749:     return wrap(dispatch_arccos(_r.tensor(0)));
1750:   } else {
1751:     // aten::arccos.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
1752: 
1753:     auto dispatch_arccos_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
1754:       pybind11::gil_scoped_release no_gil;
1755:       return at::arccos_out(out, self);
1756:     };
1757:     return wrap(dispatch_arccos_out(_r.tensor(1), _r.tensor(0)));
1758:   }
1759:   Py_RETURN_NONE;
1760:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `acos`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `acos`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1761-1840

```cpp
1761: }
1762: 
1763: // arccos_
1764: static PyObject * THPVariable_arccos_(PyObject* self_, PyObject* args, PyObject* kwargs)
1765: {
1766:   HANDLE_TH_ERRORS
1767:   static PythonArgParser parser({
1768:     "arccos_(Tensor input)",
1769:   }, /*traceable=*/true);
1770: 
1771:   ParsedArgs<1> parsed_args;
1772:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1773:   if(_r.has_torch_function()) {
1774:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1775:   }
1776:   // aten::arccos_(Tensor(a!) self) -> Tensor(a!)
1777: 
1778:   auto dispatch_arccos_ = [](at::Tensor self) -> at::Tensor {
1779:     pybind11::gil_scoped_release no_gil;
1780:     return self.arccos_();
1781:   };
1782:   return wrap(dispatch_arccos_(_r.tensor(0)));
1783:   Py_RETURN_NONE;
1784:   END_HANDLE_TH_ERRORS
1785: }
1786: 
1787: // adaptive_avg_pool1d
1788: static PyObject * THPVariable_adaptive_avg_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs)
1789: {
1790:   HANDLE_TH_ERRORS
1791:   static PythonArgParser parser({
1792:     "adaptive_avg_pool1d(Tensor input, IntArrayRef[1] output_size)",
1793:   }, /*traceable=*/true);
1794: 
1795:   ParsedArgs<2> parsed_args;
1796:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1797:   if(_r.has_torch_function()) {
1798:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1799:   }
1800:   // aten::adaptive_avg_pool1d(Tensor self, int[1] output_size) -> Tensor
1801: 
1802:   auto dispatch_adaptive_avg_pool1d = [](const at::Tensor & self, at::IntArrayRef output_size) -> at::Tensor {
1803:     pybind11::gil_scoped_release no_gil;
1804:     return at::adaptive_avg_pool1d(self, output_size);
1805:   };
1806:   return wrap(dispatch_adaptive_avg_pool1d(_r.tensor(0), _r.intlist(1)));
1807:   Py_RETURN_NONE;
1808:   END_HANDLE_TH_ERRORS
1809: }
1810: 
1811: \
1812: // addmv
1813: static PyObject * THPVariable_addmv(PyObject* self_, PyObject* args, PyObject* kwargs)
1814: {
1815:   HANDLE_TH_ERRORS
1816:   static PythonArgParser parser({
1817:     "addmv(Scalar beta, Tensor input, Scalar alpha, Tensor mat, Tensor vec, *, Tensor out=None)|deprecated",
1818:     "addmv(Scalar beta, Tensor input, Tensor mat, Tensor vec, *, Tensor out=None)|deprecated",
1819:     "addmv(Tensor input, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1, Tensor out=None)",
1820:   }, /*traceable=*/true);
1821: 
1822:   ParsedArgs<6> parsed_args;
1823:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1824:   if(_r.has_torch_function()) {
1825:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1826:   }
1827:   switch (_r.idx) {
1828:     case 0: {
1829:       if (_r.isNone(5)) {
1830:         // [deprecated] aten::addmv(Scalar beta, Tensor self, Scalar alpha, Tensor mat, Tensor vec, *, Tensor(a!) out) -> Tensor(a!)
1831: 
1832:         auto dispatch_addmv = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
1833:           pybind11::gil_scoped_release no_gil;
1834:           return self.addmv(mat, vec, beta, alpha);
1835:         };
1836:         return wrap(dispatch_addmv(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
1837:       } else {
1838:         // [deprecated] aten::addmv(Scalar beta, Tensor self, Scalar alpha, Tensor mat, Tensor vec, *, Tensor(a!) out) -> Tensor(a!)
1839: 
1840:         auto dispatch_addmv_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable_arccos_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_arccos_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1841-1920

```cpp
1841:           pybind11::gil_scoped_release no_gil;
1842:           return at::addmv_out(out, self, mat, vec, beta, alpha);
1843:         };
1844:         return wrap(dispatch_addmv_out(_r.tensor(5), _r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
1845:       }
1846:     }
1847:     case 1: {
1848:       if (_r.isNone(4)) {
1849:         // [deprecated] aten::addmv(Scalar beta, Tensor self, Tensor mat, Tensor vec, *, Tensor(a!) out) -> Tensor(a!)
1850: 
1851:         auto dispatch_addmv = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
1852:           pybind11::gil_scoped_release no_gil;
1853:           return self.addmv(mat, vec, beta, 1);
1854:         };
1855:         return wrap(dispatch_addmv(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
1856:       } else {
1857:         // [deprecated] aten::addmv(Scalar beta, Tensor self, Tensor mat, Tensor vec, *, Tensor(a!) out) -> Tensor(a!)
1858: 
1859:         auto dispatch_addmv_out = [](at::Tensor out, const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
1860:           pybind11::gil_scoped_release no_gil;
1861:           return at::addmv_out(out, self, mat, vec, beta, 1);
1862:         };
1863:         return wrap(dispatch_addmv_out(_r.tensor(4), _r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
1864:       }
1865:     }
1866:     case 2: {
1867:       if (_r.isNone(5)) {
1868:         // aten::addmv(Tensor self, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1) -> Tensor
1869: 
1870:         auto dispatch_addmv = [](const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
1871:           pybind11::gil_scoped_release no_gil;
1872:           return self.addmv(mat, vec, beta, alpha);
1873:         };
1874:         return wrap(dispatch_addmv(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
1875:       } else {
1876:         // aten::addmv.out(Tensor self, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
1877: 
1878:         auto dispatch_addmv_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
1879:           pybind11::gil_scoped_release no_gil;
1880:           return at::addmv_out(out, self, mat, vec, beta, alpha);
1881:         };
1882:         return wrap(dispatch_addmv_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
1883:       }
1884:     }
1885:   }
1886:   Py_RETURN_NONE;
1887:   END_HANDLE_TH_ERRORS
1888: }
1889: 
1890: \
1891: // addmv_
1892: static PyObject * THPVariable_addmv_(PyObject* self_, PyObject* args, PyObject* kwargs)
1893: {
1894:   HANDLE_TH_ERRORS
1895:   static PythonArgParser parser({
1896:     "addmv_(Scalar beta, Tensor input, Scalar alpha, Tensor mat, Tensor vec)|deprecated",
1897:     "addmv_(Scalar beta, Tensor input, Tensor mat, Tensor vec)|deprecated",
1898:     "addmv_(Tensor input, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1)",
1899:   }, /*traceable=*/true);
1900: 
1901:   ParsedArgs<5> parsed_args;
1902:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1903:   if(_r.has_torch_function()) {
1904:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1905:   }
1906:   switch (_r.idx) {
1907:     case 0: {
1908:       // [deprecated] aten::addmv_(Scalar beta, Tensor(a!) self, Scalar alpha, Tensor mat, Tensor vec) -> Tensor(a!)
1909: 
1910:       auto dispatch_addmv_ = [](const at::Scalar & beta, at::Tensor self, const at::Scalar & alpha, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
1911:         pybind11::gil_scoped_release no_gil;
1912:         return self.addmv_(mat, vec, beta, alpha);
1913:       };
1914:       return wrap(dispatch_addmv_(_r.scalar(0), _r.tensor(1), _r.scalar(2), _r.tensor(3), _r.tensor(4)));
1915:     }
1916:     case 1: {
1917:       // [deprecated] aten::addmv_(Scalar beta, Tensor(a!) self, Tensor mat, Tensor vec) -> Tensor(a!)
1918: 
1919:       auto dispatch_addmv_ = [](const at::Scalar & beta, at::Tensor self, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
1920:         pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `addmv_out`, `wrap`, `addmv`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `addmv_out`, `wrap`, `addmv` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2000

```cpp
1921:         return self.addmv_(mat, vec, beta, 1);
1922:       };
1923:       return wrap(dispatch_addmv_(_r.scalar(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
1924:     }
1925:     case 2: {
1926:       // aten::addmv_(Tensor(a!) self, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1) -> Tensor(a!)
1927: 
1928:       auto dispatch_addmv_ = [](at::Tensor self, const at::Tensor & mat, const at::Tensor & vec, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
1929:         pybind11::gil_scoped_release no_gil;
1930:         return self.addmv_(mat, vec, beta, alpha);
1931:       };
1932:       return wrap(dispatch_addmv_(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3), _r.scalar(4)));
1933:     }
1934:   }
1935:   Py_RETURN_NONE;
1936:   END_HANDLE_TH_ERRORS
1937: }
1938: 
1939: // _is_all_true
1940: static PyObject * THPVariable__is_all_true(PyObject* self_, PyObject* args, PyObject* kwargs)
1941: {
1942:   HANDLE_TH_ERRORS
1943:   static PythonArgParser parser({
1944:     "_is_all_true(Tensor input)",
1945:   }, /*traceable=*/true);
1946: 
1947:   ParsedArgs<1> parsed_args;
1948:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1949:   if(_r.has_torch_function()) {
1950:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1951:   }
1952:   // aten::_is_all_true(Tensor self) -> Tensor
1953: 
1954:   auto dispatch__is_all_true = [](const at::Tensor & self) -> at::Tensor {
1955:     pybind11::gil_scoped_release no_gil;
1956:     return self._is_all_true();
1957:   };
1958:   return wrap(dispatch__is_all_true(_r.tensor(0)));
1959:   Py_RETURN_NONE;
1960:   END_HANDLE_TH_ERRORS
1961: }
1962: 
1963: // _test_check_tensor
1964: static PyObject * THPVariable__test_check_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
1965: {
1966:   HANDLE_TH_ERRORS
1967:   static PythonArgParser parser({
1968:     "_test_check_tensor(Tensor input)",
1969:   }, /*traceable=*/true);
1970: 
1971:   ParsedArgs<1> parsed_args;
1972:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1973:   if(_r.has_torch_function()) {
1974:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1975:   }
1976:   // aten::_test_check_tensor(Tensor self) -> Tensor
1977: 
1978:   auto dispatch__test_check_tensor = [](const at::Tensor & self) -> at::Tensor {
1979:     pybind11::gil_scoped_release no_gil;
1980:     return at::_test_check_tensor(self);
1981:   };
1982:   return wrap(dispatch__test_check_tensor(_r.tensor(0)));
1983:   Py_RETURN_NONE;
1984:   END_HANDLE_TH_ERRORS
1985: }
1986: 
1987: // argmin
1988: static PyObject * THPVariable_argmin(PyObject* self_, PyObject* args, PyObject* kwargs)
1989: {
1990:   HANDLE_TH_ERRORS
1991:   static PythonArgParser parser({
1992:     "argmin(Tensor input, int64_t? dim=None, bool keepdim=False, *, Tensor out=None)",
1993:   }, /*traceable=*/true);
1994: 
1995:   ParsedArgs<4> parsed_args;
1996:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
1997:   if(_r.has_torch_function()) {
1998:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
1999:   }
2000:   if (_r.isNone(3)) {
```

- EN: The main execution path in this span is carried by `wrap`, `addmv_`, `THPVariable__is_all_true`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `addmv_`, `THPVariable__is_all_true` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2001-2080

```cpp
2001:     // aten::argmin(Tensor self, int? dim=None, bool keepdim=False) -> Tensor
2002: 
2003:     auto dispatch_argmin = [](const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> at::Tensor {
2004:       pybind11::gil_scoped_release no_gil;
2005:       return self.argmin(dim, keepdim);
2006:     };
2007:     return wrap(dispatch_argmin(_r.tensor(0), _r.toInt64Optional(1), _r.toBool(2)));
2008:   } else {
2009:     // aten::argmin.out(Tensor self, int? dim=None, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
2010: 
2011:     auto dispatch_argmin_out = [](at::Tensor out, const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> at::Tensor {
2012:       pybind11::gil_scoped_release no_gil;
2013:       return at::argmin_out(out, self, dim, keepdim);
2014:     };
2015:     return wrap(dispatch_argmin_out(_r.tensor(3), _r.tensor(0), _r.toInt64Optional(1), _r.toBool(2)));
2016:   }
2017:   Py_RETURN_NONE;
2018:   END_HANDLE_TH_ERRORS
2019: }
2020: 
2021: // asinh
2022: static PyObject * THPVariable_asinh(PyObject* self_, PyObject* args, PyObject* kwargs)
2023: {
2024:   HANDLE_TH_ERRORS
2025:   static PythonArgParser parser({
2026:     "asinh(Tensor input, *, Tensor out=None)",
2027:   }, /*traceable=*/true);
2028: 
2029:   ParsedArgs<2> parsed_args;
2030:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2031:   if(_r.has_torch_function()) {
2032:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2033:   }
2034:   if (_r.isNone(1)) {
2035:     // aten::asinh(Tensor self) -> Tensor
2036: 
2037:     auto dispatch_asinh = [](const at::Tensor & self) -> at::Tensor {
2038:       pybind11::gil_scoped_release no_gil;
2039:       return self.asinh();
2040:     };
2041:     return wrap(dispatch_asinh(_r.tensor(0)));
2042:   } else {
2043:     // aten::asinh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2044: 
2045:     auto dispatch_asinh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2046:       pybind11::gil_scoped_release no_gil;
2047:       return at::asinh_out(out, self);
2048:     };
2049:     return wrap(dispatch_asinh_out(_r.tensor(1), _r.tensor(0)));
2050:   }
2051:   Py_RETURN_NONE;
2052:   END_HANDLE_TH_ERRORS
2053: }
2054: 
2055: // asinh_
2056: static PyObject * THPVariable_asinh_(PyObject* self_, PyObject* args, PyObject* kwargs)
2057: {
2058:   HANDLE_TH_ERRORS
2059:   static PythonArgParser parser({
2060:     "asinh_(Tensor input)",
2061:   }, /*traceable=*/true);
2062: 
2063:   ParsedArgs<1> parsed_args;
2064:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2065:   if(_r.has_torch_function()) {
2066:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2067:   }
2068:   // aten::asinh_(Tensor(a!) self) -> Tensor(a!)
2069: 
2070:   auto dispatch_asinh_ = [](at::Tensor self) -> at::Tensor {
2071:     pybind11::gil_scoped_release no_gil;
2072:     return self.asinh_();
2073:   };
2074:   return wrap(dispatch_asinh_(_r.tensor(0)));
2075:   Py_RETURN_NONE;
2076:   END_HANDLE_TH_ERRORS
2077: }
2078: 
2079: // arcsinh
2080: static PyObject * THPVariable_arcsinh(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `argmin`, `wrap`, `argmin_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `argmin`, `wrap`, `argmin_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2081-2160

```cpp
2081: {
2082:   HANDLE_TH_ERRORS
2083:   static PythonArgParser parser({
2084:     "arcsinh(Tensor input, *, Tensor out=None)",
2085:   }, /*traceable=*/true);
2086: 
2087:   ParsedArgs<2> parsed_args;
2088:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2089:   if(_r.has_torch_function()) {
2090:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2091:   }
2092:   if (_r.isNone(1)) {
2093:     // aten::arcsinh(Tensor self) -> Tensor
2094: 
2095:     auto dispatch_arcsinh = [](const at::Tensor & self) -> at::Tensor {
2096:       pybind11::gil_scoped_release no_gil;
2097:       return self.arcsinh();
2098:     };
2099:     return wrap(dispatch_arcsinh(_r.tensor(0)));
2100:   } else {
2101:     // aten::arcsinh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2102: 
2103:     auto dispatch_arcsinh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2104:       pybind11::gil_scoped_release no_gil;
2105:       return at::arcsinh_out(out, self);
2106:     };
2107:     return wrap(dispatch_arcsinh_out(_r.tensor(1), _r.tensor(0)));
2108:   }
2109:   Py_RETURN_NONE;
2110:   END_HANDLE_TH_ERRORS
2111: }
2112: 
2113: // arcsinh_
2114: static PyObject * THPVariable_arcsinh_(PyObject* self_, PyObject* args, PyObject* kwargs)
2115: {
2116:   HANDLE_TH_ERRORS
2117:   static PythonArgParser parser({
2118:     "arcsinh_(Tensor input)",
2119:   }, /*traceable=*/true);
2120: 
2121:   ParsedArgs<1> parsed_args;
2122:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2123:   if(_r.has_torch_function()) {
2124:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2125:   }
2126:   // aten::arcsinh_(Tensor(a!) self) -> Tensor(a!)
2127: 
2128:   auto dispatch_arcsinh_ = [](at::Tensor self) -> at::Tensor {
2129:     pybind11::gil_scoped_release no_gil;
2130:     return self.arcsinh_();
2131:   };
2132:   return wrap(dispatch_arcsinh_(_r.tensor(0)));
2133:   Py_RETURN_NONE;
2134:   END_HANDLE_TH_ERRORS
2135: }
2136: 
2137: // asin
2138: static PyObject * THPVariable_asin(PyObject* self_, PyObject* args, PyObject* kwargs)
2139: {
2140:   HANDLE_TH_ERRORS
2141:   static PythonArgParser parser({
2142:     "asin(Tensor input, *, Tensor out=None)",
2143:   }, /*traceable=*/true);
2144: 
2145:   ParsedArgs<2> parsed_args;
2146:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2147:   if(_r.has_torch_function()) {
2148:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2149:   }
2150:   if (_r.isNone(1)) {
2151:     // aten::asin(Tensor self) -> Tensor
2152: 
2153:     auto dispatch_asin = [](const at::Tensor & self) -> at::Tensor {
2154:       pybind11::gil_scoped_release no_gil;
2155:       return self.asin();
2156:     };
2157:     return wrap(dispatch_asin(_r.tensor(0)));
2158:   } else {
2159:     // aten::asin.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2160: 
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `arcsinh`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `arcsinh` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2240

```cpp
2161:     auto dispatch_asin_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2162:       pybind11::gil_scoped_release no_gil;
2163:       return at::asin_out(out, self);
2164:     };
2165:     return wrap(dispatch_asin_out(_r.tensor(1), _r.tensor(0)));
2166:   }
2167:   Py_RETURN_NONE;
2168:   END_HANDLE_TH_ERRORS
2169: }
2170: 
2171: // asin_
2172: static PyObject * THPVariable_asin_(PyObject* self_, PyObject* args, PyObject* kwargs)
2173: {
2174:   HANDLE_TH_ERRORS
2175:   static PythonArgParser parser({
2176:     "asin_(Tensor input)",
2177:   }, /*traceable=*/true);
2178: 
2179:   ParsedArgs<1> parsed_args;
2180:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2181:   if(_r.has_torch_function()) {
2182:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2183:   }
2184:   // aten::asin_(Tensor(a!) self) -> Tensor(a!)
2185: 
2186:   auto dispatch_asin_ = [](at::Tensor self) -> at::Tensor {
2187:     pybind11::gil_scoped_release no_gil;
2188:     return self.asin_();
2189:   };
2190:   return wrap(dispatch_asin_(_r.tensor(0)));
2191:   Py_RETURN_NONE;
2192:   END_HANDLE_TH_ERRORS
2193: }
2194: 
2195: // arctan
2196: static PyObject * THPVariable_arctan(PyObject* self_, PyObject* args, PyObject* kwargs)
2197: {
2198:   HANDLE_TH_ERRORS
2199:   static PythonArgParser parser({
2200:     "arctan(Tensor input, *, Tensor out=None)",
2201:   }, /*traceable=*/true);
2202: 
2203:   ParsedArgs<2> parsed_args;
2204:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2205:   if(_r.has_torch_function()) {
2206:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2207:   }
2208:   if (_r.isNone(1)) {
2209:     // aten::arctan(Tensor self) -> Tensor
2210: 
2211:     auto dispatch_arctan = [](const at::Tensor & self) -> at::Tensor {
2212:       pybind11::gil_scoped_release no_gil;
2213:       return self.arctan();
2214:     };
2215:     return wrap(dispatch_arctan(_r.tensor(0)));
2216:   } else {
2217:     // aten::arctan.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2218: 
2219:     auto dispatch_arctan_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2220:       pybind11::gil_scoped_release no_gil;
2221:       return at::arctan_out(out, self);
2222:     };
2223:     return wrap(dispatch_arctan_out(_r.tensor(1), _r.tensor(0)));
2224:   }
2225:   Py_RETURN_NONE;
2226:   END_HANDLE_TH_ERRORS
2227: }
2228: 
2229: // arctan_
2230: static PyObject * THPVariable_arctan_(PyObject* self_, PyObject* args, PyObject* kwargs)
2231: {
2232:   HANDLE_TH_ERRORS
2233:   static PythonArgParser parser({
2234:     "arctan_(Tensor input)",
2235:   }, /*traceable=*/true);
2236: 
2237:   ParsedArgs<1> parsed_args;
2238:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2239:   if(_r.has_torch_function()) {
2240:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `asin_out`, `wrap`, `THPVariable_asin_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `asin_out`, `wrap`, `THPVariable_asin_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2241-2320

```cpp
2241:   }
2242:   // aten::arctan_(Tensor(a!) self) -> Tensor(a!)
2243: 
2244:   auto dispatch_arctan_ = [](at::Tensor self) -> at::Tensor {
2245:     pybind11::gil_scoped_release no_gil;
2246:     return self.arctan_();
2247:   };
2248:   return wrap(dispatch_arctan_(_r.tensor(0)));
2249:   Py_RETURN_NONE;
2250:   END_HANDLE_TH_ERRORS
2251: }
2252: 
2253: \
2254: // atleast_2d
2255: static PyObject * THPVariable_atleast_2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2256: {
2257:   HANDLE_TH_ERRORS
2258:   static PythonArgParser parser({
2259:     "atleast_2d(Tensor input)",
2260:     "atleast_2d(TensorList tensors)",
2261:   }, /*traceable=*/true);
2262: 
2263:   ParsedArgs<1> parsed_args;
2264:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2265:   if(_r.has_torch_function()) {
2266:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2267:   }
2268:   switch (_r.idx) {
2269:     case 0: {
2270:       // aten::atleast_2d(Tensor self) -> Tensor
2271: 
2272:       auto dispatch_atleast_2d = [](const at::Tensor & self) -> at::Tensor {
2273:         pybind11::gil_scoped_release no_gil;
2274:         return at::atleast_2d(self);
2275:       };
2276:       return wrap(dispatch_atleast_2d(_r.tensor(0)));
2277:     }
2278:     case 1: {
2279:       // aten::atleast_2d.Sequence(Tensor[] tensors) -> Tensor[]
2280: 
2281:       auto dispatch_atleast_2d = [](at::TensorList tensors) -> ::std::vector<at::Tensor> {
2282:         pybind11::gil_scoped_release no_gil;
2283:         return at::atleast_2d(tensors);
2284:       };
2285:       return wrap(dispatch_atleast_2d(_r.tensorlist(0)));
2286:     }
2287:   }
2288:   Py_RETURN_NONE;
2289:   END_HANDLE_TH_ERRORS
2290: }
2291: 
2292: \
2293: // atleast_3d
2294: static PyObject * THPVariable_atleast_3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2295: {
2296:   HANDLE_TH_ERRORS
2297:   static PythonArgParser parser({
2298:     "atleast_3d(Tensor input)",
2299:     "atleast_3d(TensorList tensors)",
2300:   }, /*traceable=*/true);
2301: 
2302:   ParsedArgs<1> parsed_args;
2303:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2304:   if(_r.has_torch_function()) {
2305:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2306:   }
2307:   switch (_r.idx) {
2308:     case 0: {
2309:       // aten::atleast_3d(Tensor self) -> Tensor
2310: 
2311:       auto dispatch_atleast_3d = [](const at::Tensor & self) -> at::Tensor {
2312:         pybind11::gil_scoped_release no_gil;
2313:         return at::atleast_3d(self);
2314:       };
2315:       return wrap(dispatch_atleast_3d(_r.tensor(0)));
2316:     }
2317:     case 1: {
2318:       // aten::atleast_3d.Sequence(Tensor[] tensors) -> Tensor[]
2319: 
2320:       auto dispatch_atleast_3d = [](at::TensorList tensors) -> ::std::vector<at::Tensor> {
```

- EN: The main execution path in this span is carried by `arctan_`, `wrap`, `THPVariable_atleast_2d`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `arctan_`, `wrap`, `THPVariable_atleast_2d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2321-2400

```cpp
2321:         pybind11::gil_scoped_release no_gil;
2322:         return at::atleast_3d(tensors);
2323:       };
2324:       return wrap(dispatch_atleast_3d(_r.tensorlist(0)));
2325:     }
2326:   }
2327:   Py_RETURN_NONE;
2328:   END_HANDLE_TH_ERRORS
2329: }
2330: 
2331: // quantized_batch_norm
2332: static PyObject * THPVariable_quantized_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
2333: {
2334:   HANDLE_TH_ERRORS
2335:   static PythonArgParser parser({
2336:     "quantized_batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor mean, Tensor var, double eps, double output_scale, int64_t output_zero_point)",
2337:   }, /*traceable=*/true);
2338: 
2339:   ParsedArgs<8> parsed_args;
2340:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2341:   if(_r.has_torch_function()) {
2342:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2343:   }
2344:   // aten::quantized_batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor mean, Tensor var, float eps, float output_scale, int output_zero_point) -> Tensor
2345: 
2346:   auto dispatch_quantized_batch_norm = [](const at::Tensor & input, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, const at::Tensor & mean, const at::Tensor & var, double eps, double output_scale, int64_t output_zero_point) -> at::Tensor {
2347:     pybind11::gil_scoped_release no_gil;
2348:     return at::quantized_batch_norm(input, weight, bias, mean, var, eps, output_scale, output_zero_point);
2349:   };
2350:   return wrap(dispatch_quantized_batch_norm(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.tensor(3), _r.tensor(4), _r.toDouble(5), _r.toDouble(6), _r.toInt64(7)));
2351:   Py_RETURN_NONE;
2352:   END_HANDLE_TH_ERRORS
2353: }
2354: 
2355: // _batch_norm_impl_index
2356: static PyObject * THPVariable__batch_norm_impl_index(PyObject* self_, PyObject* args, PyObject* kwargs)
2357: {
2358:   HANDLE_TH_ERRORS
2359:   static PythonArgParser parser({
2360:     "_batch_norm_impl_index(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, double momentum, double eps, bool cudnn_enabled)",
2361:   }, /*traceable=*/true);
2362: 
2363:   ParsedArgs<9> parsed_args;
2364:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2365:   if(_r.has_torch_function()) {
2366:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2367:   }
2368:   // aten::_batch_norm_impl_index(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float momentum, float eps, bool cudnn_enabled) -> (Tensor, Tensor, Tensor, Tensor, int)
2369: 
2370:   auto dispatch__batch_norm_impl_index = [](const at::Tensor & input, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, bool training, double momentum, double eps, bool cudnn_enabled) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,int64_t> {
2371:     pybind11::gil_scoped_release no_gil;
2372:     return at::_batch_norm_impl_index(input, weight, bias, running_mean, running_var, training, momentum, eps, cudnn_enabled);
2373:   };
2374:   return wrap(dispatch__batch_norm_impl_index(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toBool(5), _r.toDouble(6), _r.toDouble(7), _r.toBool(8)));
2375:   Py_RETURN_NONE;
2376:   END_HANDLE_TH_ERRORS
2377: }
2378: 
2379: // bincount
2380: static PyObject * THPVariable_bincount(PyObject* self_, PyObject* args, PyObject* kwargs)
2381: {
2382:   HANDLE_TH_ERRORS
2383:   static PythonArgParser parser({
2384:     "bincount(Tensor input, Tensor? weights=None, SymInt minlength=0)",
2385:   }, /*traceable=*/true);
2386: 
2387:   ParsedArgs<3> parsed_args;
2388:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2389:   if(_r.has_torch_function()) {
2390:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2391:   }
2392:   // aten::bincount(Tensor self, Tensor? weights=None, SymInt minlength=0) -> Tensor
2393: 
2394:   auto dispatch_bincount = [](const at::Tensor & self, const ::std::optional<at::Tensor> & weights, c10::SymInt minlength) -> at::Tensor {
2395:     pybind11::gil_scoped_release no_gil;
2396:     return self.bincount_symint(weights, minlength);
2397:   };
2398:   return wrap(dispatch_bincount(_r.tensor(0), _r.optionalTensor(1), _r.toSymInt(2)));
2399:   Py_RETURN_NONE;
2400:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `atleast_3d`, `wrap`, `THPVariable_quantized_batch_norm`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `atleast_3d`, `wrap`, `THPVariable_quantized_batch_norm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2480

```cpp
2401: }
2402: 
2403: // bitwise_not
2404: static PyObject * THPVariable_bitwise_not(PyObject* self_, PyObject* args, PyObject* kwargs)
2405: {
2406:   HANDLE_TH_ERRORS
2407:   static PythonArgParser parser({
2408:     "bitwise_not(Tensor input, *, Tensor out=None)",
2409:   }, /*traceable=*/true);
2410: 
2411:   ParsedArgs<2> parsed_args;
2412:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2413:   if(_r.has_torch_function()) {
2414:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2415:   }
2416:   if (_r.isNone(1)) {
2417:     // aten::bitwise_not(Tensor self) -> Tensor
2418: 
2419:     auto dispatch_bitwise_not = [](const at::Tensor & self) -> at::Tensor {
2420:       pybind11::gil_scoped_release no_gil;
2421:       return self.bitwise_not();
2422:     };
2423:     return wrap(dispatch_bitwise_not(_r.tensor(0)));
2424:   } else {
2425:     // aten::bitwise_not.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
2426: 
2427:     auto dispatch_bitwise_not_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
2428:       pybind11::gil_scoped_release no_gil;
2429:       return at::bitwise_not_out(out, self);
2430:     };
2431:     return wrap(dispatch_bitwise_not_out(_r.tensor(1), _r.tensor(0)));
2432:   }
2433:   Py_RETURN_NONE;
2434:   END_HANDLE_TH_ERRORS
2435: }
2436: 
2437: // logical_or
2438: static PyObject * THPVariable_logical_or(PyObject* self_, PyObject* args, PyObject* kwargs)
2439: {
2440:   HANDLE_TH_ERRORS
2441:   static PythonArgParser parser({
2442:     "logical_or(Tensor input, Tensor other, *, Tensor out=None)",
2443:   }, /*traceable=*/true);
2444: 
2445:   ParsedArgs<3> parsed_args;
2446:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2447:   if(_r.has_torch_function()) {
2448:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2449:   }
2450:   if (_r.isNone(2)) {
2451:     // aten::logical_or(Tensor self, Tensor other) -> Tensor
2452: 
2453:     auto dispatch_logical_or = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2454:       pybind11::gil_scoped_release no_gil;
2455:       return self.logical_or(other);
2456:     };
2457:     return wrap(dispatch_logical_or(_r.tensor(0), _r.tensor(1)));
2458:   } else {
2459:     // aten::logical_or.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
2460: 
2461:     auto dispatch_logical_or_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2462:       pybind11::gil_scoped_release no_gil;
2463:       return at::logical_or_out(out, self, other);
2464:     };
2465:     return wrap(dispatch_logical_or_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2466:   }
2467:   Py_RETURN_NONE;
2468:   END_HANDLE_TH_ERRORS
2469: }
2470: 
2471: // unsafe_chunk
2472: static PyObject * THPVariable_unsafe_chunk(PyObject* self_, PyObject* args, PyObject* kwargs)
2473: {
2474:   HANDLE_TH_ERRORS
2475:   static PythonArgParser parser({
2476:     "unsafe_chunk(Tensor input, int64_t chunks, int64_t dim=0)",
2477:   }, /*traceable=*/true);
2478: 
2479:   ParsedArgs<3> parsed_args;
2480:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `THPVariable_bitwise_not`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_bitwise_not`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2481-2560

```cpp
2481:   if(_r.has_torch_function()) {
2482:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2483:   }
2484:   // aten::unsafe_chunk(Tensor self, int chunks, int dim=0) -> Tensor[]
2485: 
2486:   auto dispatch_unsafe_chunk = [](const at::Tensor & self, int64_t chunks, int64_t dim) -> ::std::vector<at::Tensor> {
2487:     pybind11::gil_scoped_release no_gil;
2488:     return self.unsafe_chunk(chunks, dim);
2489:   };
2490:   return wrap(dispatch_unsafe_chunk(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
2491:   Py_RETURN_NONE;
2492:   END_HANDLE_TH_ERRORS
2493: }
2494: 
2495: \
2496: // clamp
2497: static PyObject * THPVariable_clamp(PyObject* self_, PyObject* args, PyObject* kwargs)
2498: {
2499:   HANDLE_TH_ERRORS
2500:   static PythonArgParser parser({
2501:     "clamp(Tensor input, Tensor? min=None, Tensor? max=None, *, Tensor out=None)",
2502:     "clamp(Tensor input, Scalar? min=None, Scalar? max=None, *, Tensor out=None)",
2503:   }, /*traceable=*/true);
2504: 
2505:   ParsedArgs<4> parsed_args;
2506:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2507:   if(_r.has_torch_function()) {
2508:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2509:   }
2510:   switch (_r.idx) {
2511:     case 0: {
2512:       if (_r.isNone(3)) {
2513:         // aten::clamp.Tensor(Tensor self, Tensor? min=None, Tensor? max=None) -> Tensor
2514: 
2515:         auto dispatch_clamp = [](const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
2516:           pybind11::gil_scoped_release no_gil;
2517:           return self.clamp(min, max);
2518:         };
2519:         return wrap(dispatch_clamp(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2)));
2520:       } else {
2521:         // aten::clamp.Tensor_out(Tensor self, Tensor? min=None, Tensor? max=None, *, Tensor(a!) out) -> Tensor(a!)
2522: 
2523:         auto dispatch_clamp_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
2524:           pybind11::gil_scoped_release no_gil;
2525:           return at::clamp_out(out, self, min, max);
2526:         };
2527:         return wrap(dispatch_clamp_out(_r.tensor(3), _r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2)));
2528:       }
2529:     }
2530:     case 1: {
2531:       if (_r.isNone(3)) {
2532:         // aten::clamp(Tensor self, Scalar? min=None, Scalar? max=None) -> Tensor
2533: 
2534:         auto dispatch_clamp = [](const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
2535:           pybind11::gil_scoped_release no_gil;
2536:           return self.clamp(min, max);
2537:         };
2538:         return wrap(dispatch_clamp(_r.tensor(0), _r.scalarOptional(1), _r.scalarOptional(2)));
2539:       } else {
2540:         // aten::clamp.out(Tensor self, Scalar? min=None, Scalar? max=None, *, Tensor(a!) out) -> Tensor(a!)
2541: 
2542:         auto dispatch_clamp_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
2543:           pybind11::gil_scoped_release no_gil;
2544:           return at::clamp_out(out, self, min, max);
2545:         };
2546:         return wrap(dispatch_clamp_out(_r.tensor(3), _r.tensor(0), _r.scalarOptional(1), _r.scalarOptional(2)));
2547:       }
2548:     }
2549:   }
2550:   Py_RETURN_NONE;
2551:   END_HANDLE_TH_ERRORS
2552: }
2553: 
2554: \
2555: // clamp_
2556: static PyObject * THPVariable_clamp_(PyObject* self_, PyObject* args, PyObject* kwargs)
2557: {
2558:   HANDLE_TH_ERRORS
2559:   static PythonArgParser parser({
2560:     "clamp_(Tensor input, Tensor? min=None, Tensor? max=None)",
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `unsafe_chunk`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `unsafe_chunk`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2561-2640

```cpp
2561:     "clamp_(Tensor input, Scalar? min=None, Scalar? max=None)",
2562:   }, /*traceable=*/true);
2563: 
2564:   ParsedArgs<3> parsed_args;
2565:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2566:   if(_r.has_torch_function()) {
2567:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2568:   }
2569:   switch (_r.idx) {
2570:     case 0: {
2571:       // aten::clamp_.Tensor(Tensor(a!) self, Tensor? min=None, Tensor? max=None) -> Tensor(a!)
2572: 
2573:       auto dispatch_clamp_ = [](at::Tensor self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
2574:         pybind11::gil_scoped_release no_gil;
2575:         return self.clamp_(min, max);
2576:       };
2577:       return wrap(dispatch_clamp_(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2)));
2578:     }
2579:     case 1: {
2580:       // aten::clamp_(Tensor(a!) self, Scalar? min=None, Scalar? max=None) -> Tensor(a!)
2581: 
2582:       auto dispatch_clamp_ = [](at::Tensor self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
2583:         pybind11::gil_scoped_release no_gil;
2584:         return self.clamp_(min, max);
2585:       };
2586:       return wrap(dispatch_clamp_(_r.tensor(0), _r.scalarOptional(1), _r.scalarOptional(2)));
2587:     }
2588:   }
2589:   Py_RETURN_NONE;
2590:   END_HANDLE_TH_ERRORS
2591: }
2592: 
2593: \
2594: // clamp_max
2595: static PyObject * THPVariable_clamp_max(PyObject* self_, PyObject* args, PyObject* kwargs)
2596: {
2597:   HANDLE_TH_ERRORS
2598:   static PythonArgParser parser({
2599:     "clamp_max(Tensor input, Tensor max, *, Tensor out=None)",
2600:     "clamp_max(Tensor input, Scalar max, *, Tensor out=None)",
2601:   }, /*traceable=*/true);
2602: 
2603:   ParsedArgs<3> parsed_args;
2604:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2605:   if(_r.has_torch_function()) {
2606:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2607:   }
2608:   switch (_r.idx) {
2609:     case 0: {
2610:       if (_r.isNone(2)) {
2611:         // aten::clamp_max.Tensor(Tensor self, Tensor max) -> Tensor
2612: 
2613:         auto dispatch_clamp_max = [](const at::Tensor & self, const at::Tensor & max) -> at::Tensor {
2614:           pybind11::gil_scoped_release no_gil;
2615:           return self.clamp_max(max);
2616:         };
2617:         return wrap(dispatch_clamp_max(_r.tensor(0), _r.tensor(1)));
2618:       } else {
2619:         // aten::clamp_max.Tensor_out(Tensor self, Tensor max, *, Tensor(a!) out) -> Tensor(a!)
2620: 
2621:         auto dispatch_clamp_max_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & max) -> at::Tensor {
2622:           pybind11::gil_scoped_release no_gil;
2623:           return at::clamp_max_out(out, self, max);
2624:         };
2625:         return wrap(dispatch_clamp_max_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
2626:       }
2627:     }
2628:     case 1: {
2629:       if (_r.isNone(2)) {
2630:         // aten::clamp_max(Tensor self, Scalar max) -> Tensor
2631: 
2632:         auto dispatch_clamp_max = [](const at::Tensor & self, const at::Scalar & max) -> at::Tensor {
2633:           pybind11::gil_scoped_release no_gil;
2634:           return self.clamp_max(max);
2635:         };
2636:         return wrap(dispatch_clamp_max(_r.tensor(0), _r.scalar(1)));
2637:       } else {
2638:         // aten::clamp_max.out(Tensor self, Scalar max, *, Tensor(a!) out) -> Tensor(a!)
2639: 
2640:         auto dispatch_clamp_max_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & max) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `clamp_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `clamp_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2641-2720

```cpp
2641:           pybind11::gil_scoped_release no_gil;
2642:           return at::clamp_max_out(out, self, max);
2643:         };
2644:         return wrap(dispatch_clamp_max_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
2645:       }
2646:     }
2647:   }
2648:   Py_RETURN_NONE;
2649:   END_HANDLE_TH_ERRORS
2650: }
2651: 
2652: \
2653: // clamp_max_
2654: static PyObject * THPVariable_clamp_max_(PyObject* self_, PyObject* args, PyObject* kwargs)
2655: {
2656:   HANDLE_TH_ERRORS
2657:   static PythonArgParser parser({
2658:     "clamp_max_(Tensor input, Tensor max)",
2659:     "clamp_max_(Tensor input, Scalar max)",
2660:   }, /*traceable=*/true);
2661: 
2662:   ParsedArgs<2> parsed_args;
2663:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2664:   if(_r.has_torch_function()) {
2665:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2666:   }
2667:   switch (_r.idx) {
2668:     case 0: {
2669:       // aten::clamp_max_.Tensor(Tensor(a!) self, Tensor max) -> Tensor(a!)
2670: 
2671:       auto dispatch_clamp_max_ = [](at::Tensor self, const at::Tensor & max) -> at::Tensor {
2672:         pybind11::gil_scoped_release no_gil;
2673:         return self.clamp_max_(max);
2674:       };
2675:       return wrap(dispatch_clamp_max_(_r.tensor(0), _r.tensor(1)));
2676:     }
2677:     case 1: {
2678:       // aten::clamp_max_(Tensor(a!) self, Scalar max) -> Tensor(a!)
2679: 
2680:       auto dispatch_clamp_max_ = [](at::Tensor self, const at::Scalar & max) -> at::Tensor {
2681:         pybind11::gil_scoped_release no_gil;
2682:         return self.clamp_max_(max);
2683:       };
2684:       return wrap(dispatch_clamp_max_(_r.tensor(0), _r.scalar(1)));
2685:     }
2686:   }
2687:   Py_RETURN_NONE;
2688:   END_HANDLE_TH_ERRORS
2689: }
2690: 
2691: // constant_pad_nd
2692: static PyObject * THPVariable_constant_pad_nd(PyObject* self_, PyObject* args, PyObject* kwargs)
2693: {
2694:   HANDLE_TH_ERRORS
2695:   static PythonArgParser parser({
2696:     "constant_pad_nd(Tensor input, SymIntArrayRef pad, Scalar value=0)",
2697:   }, /*traceable=*/true);
2698: 
2699:   ParsedArgs<3> parsed_args;
2700:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2701:   if(_r.has_torch_function()) {
2702:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2703:   }
2704:   // aten::constant_pad_nd(Tensor self, SymInt[] pad, Scalar value=0) -> Tensor
2705: 
2706:   auto dispatch_constant_pad_nd = [](const at::Tensor & self, c10::SymIntArrayRef pad, const at::Scalar & value) -> at::Tensor {
2707:     pybind11::gil_scoped_release no_gil;
2708:     return at::constant_pad_nd_symint(self, pad, value);
2709:   };
2710:   return wrap(dispatch_constant_pad_nd(_r.tensor(0), _r.symintlist(1), _r.scalar(2)));
2711:   Py_RETURN_NONE;
2712:   END_HANDLE_TH_ERRORS
2713: }
2714: 
2715: // convolution
2716: static PyObject * THPVariable_convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
2717: {
2718:   HANDLE_TH_ERRORS
2719:   static PythonArgParser parser({
2720:     "convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation, bool transposed, SymIntArrayRef output_padding, SymInt groups)",
```

- EN: The main execution path in this span is carried by `clamp_max_out`, `wrap`, `THPVariable_clamp_max_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `clamp_max_out`, `wrap`, `THPVariable_clamp_max_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2721-2800

```cpp
2721:   }, /*traceable=*/false);
2722: 
2723:   ParsedArgs<9> parsed_args;
2724:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2725:   if(_r.has_torch_function()) {
2726:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2727:   }
2728:   // aten::convolution(Tensor input, Tensor weight, Tensor? bias, SymInt[] stride, SymInt[] padding, SymInt[] dilation, bool transposed, SymInt[] output_padding, SymInt groups) -> Tensor
2729: 
2730:   auto dispatch_convolution = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed, c10::SymIntArrayRef output_padding, c10::SymInt groups) -> at::Tensor {
2731:     pybind11::gil_scoped_release no_gil;
2732:     return at::convolution_symint(input, weight, bias, stride, padding, dilation, transposed, output_padding, groups);
2733:   };
2734:   return wrap(dispatch_convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toBool(6), _r.symintlist(7), _r.toSymInt(8)));
2735:   Py_RETURN_NONE;
2736:   END_HANDLE_TH_ERRORS
2737: }
2738: 
2739: \
2740: // conv2d
2741: static PyObject * THPVariable_conv2d(PyObject* self_, PyObject* args, PyObject* kwargs)
2742: {
2743:   HANDLE_TH_ERRORS
2744:   static PythonArgParser parser({
2745:     "conv2d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[2] stride=1, SymIntArrayRef[2] padding=0, SymIntArrayRef[2] dilation=1, SymInt groups=1)",
2746:     "conv2d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[2] stride=1, c10::string_view padding=\"valid\", SymIntArrayRef[2] dilation=1, SymInt groups=1)",
2747:   }, /*traceable=*/false);
2748: 
2749:   ParsedArgs<7> parsed_args;
2750:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2751:   if(_r.has_torch_function()) {
2752:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2753:   }
2754:   switch (_r.idx) {
2755:     case 0: {
2756:       // aten::conv2d(Tensor input, Tensor weight, Tensor? bias=None, SymInt[2] stride=1, SymInt[2] padding=0, SymInt[2] dilation=1, SymInt groups=1) -> Tensor
2757: 
2758:       auto dispatch_conv2d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
2759:         pybind11::gil_scoped_release no_gil;
2760:         return at::conv2d_symint(input, weight, bias, stride, padding, dilation, groups);
2761:       };
2762:       return wrap(dispatch_conv2d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
2763:     }
2764:     case 1: {
2765:       // aten::conv2d.padding(Tensor input, Tensor weight, Tensor? bias=None, SymInt[2] stride=1, str padding="valid", SymInt[2] dilation=1, SymInt groups=1) -> Tensor
2766: 
2767:       auto dispatch_conv2d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::string_view padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
2768:         pybind11::gil_scoped_release no_gil;
2769:         return at::conv2d_symint(input, weight, bias, stride, padding, dilation, groups);
2770:       };
2771:       return wrap(dispatch_conv2d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.stringView(4), _r.symintlist(5), _r.toSymInt(6)));
2772:     }
2773:   }
2774:   Py_RETURN_NONE;
2775:   END_HANDLE_TH_ERRORS
2776: }
2777: 
2778: // conv_tbc
2779: static PyObject * THPVariable_conv_tbc(PyObject* self_, PyObject* args, PyObject* kwargs)
2780: {
2781:   HANDLE_TH_ERRORS
2782:   static PythonArgParser parser({
2783:     "conv_tbc(Tensor input, Tensor weight, Tensor bias, int64_t pad=0)",
2784:   }, /*traceable=*/true);
2785: 
2786:   ParsedArgs<4> parsed_args;
2787:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2788:   if(_r.has_torch_function()) {
2789:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2790:   }
2791:   // aten::conv_tbc(Tensor self, Tensor weight, Tensor bias, int pad=0) -> Tensor
2792: 
2793:   auto dispatch_conv_tbc = [](const at::Tensor & self, const at::Tensor & weight, const at::Tensor & bias, int64_t pad) -> at::Tensor {
2794:     pybind11::gil_scoped_release no_gil;
2795:     return at::conv_tbc(self, weight, bias, pad);
2796:   };
2797:   return wrap(dispatch_conv_tbc(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3)));
2798:   Py_RETURN_NONE;
2799:   END_HANDLE_TH_ERRORS
2800: }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `convolution`, `convolution_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `convolution`, `convolution_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2801-2880

```cpp
2801: 
2802: // conv_transpose1d
2803: static PyObject * THPVariable_conv_transpose1d(PyObject* self_, PyObject* args, PyObject* kwargs)
2804: {
2805:   HANDLE_TH_ERRORS
2806:   static PythonArgParser parser({
2807:     "conv_transpose1d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[1] stride=1, SymIntArrayRef[1] padding=0, SymIntArrayRef[1] output_padding=0, SymInt groups=1, SymIntArrayRef[1] dilation=1)",
2808:   }, /*traceable=*/false);
2809: 
2810:   ParsedArgs<8> parsed_args;
2811:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2812:   if(_r.has_torch_function()) {
2813:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2814:   }
2815:   // aten::conv_transpose1d(Tensor input, Tensor weight, Tensor? bias=None, SymInt[1] stride=1, SymInt[1] padding=0, SymInt[1] output_padding=0, SymInt groups=1, SymInt[1] dilation=1) -> Tensor
2816: 
2817:   auto dispatch_conv_transpose1d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymInt groups, c10::SymIntArrayRef dilation) -> at::Tensor {
2818:     pybind11::gil_scoped_release no_gil;
2819:     return at::conv_transpose1d_symint(input, weight, bias, stride, padding, output_padding, groups, dilation);
2820:   };
2821:   return wrap(dispatch_conv_transpose1d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6), _r.symintlist(7)));
2822:   Py_RETURN_NONE;
2823:   END_HANDLE_TH_ERRORS
2824: }
2825: 
2826: // conv_transpose3d
2827: static PyObject * THPVariable_conv_transpose3d(PyObject* self_, PyObject* args, PyObject* kwargs)
2828: {
2829:   HANDLE_TH_ERRORS
2830:   static PythonArgParser parser({
2831:     "conv_transpose3d(Tensor input, Tensor weight, Tensor? bias=None, SymIntArrayRef[3] stride=1, SymIntArrayRef[3] padding=0, SymIntArrayRef[3] output_padding=0, SymInt groups=1, SymIntArrayRef[3] dilation=1)",
2832:   }, /*traceable=*/false);
2833: 
2834:   ParsedArgs<8> parsed_args;
2835:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2836:   if(_r.has_torch_function()) {
2837:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2838:   }
2839:   // aten::conv_transpose3d.input(Tensor input, Tensor weight, Tensor? bias=None, SymInt[3] stride=1, SymInt[3] padding=0, SymInt[3] output_padding=0, SymInt groups=1, SymInt[3] dilation=1) -> Tensor
2840: 
2841:   auto dispatch_conv_transpose3d = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymInt groups, c10::SymIntArrayRef dilation) -> at::Tensor {
2842:     pybind11::gil_scoped_release no_gil;
2843:     return at::conv_transpose3d_symint(input, weight, bias, stride, padding, output_padding, groups, dilation);
2844:   };
2845:   return wrap(dispatch_conv_transpose3d(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6), _r.symintlist(7)));
2846:   Py_RETURN_NONE;
2847:   END_HANDLE_TH_ERRORS
2848: }
2849: 
2850: // cosine_embedding_loss
2851: static PyObject * THPVariable_cosine_embedding_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
2852: {
2853:   HANDLE_TH_ERRORS
2854:   static PythonArgParser parser({
2855:     "cosine_embedding_loss(Tensor input1, Tensor input2, Tensor target, double margin=0.0, int64_t reduction=at::Reduction::Mean)",
2856:   }, /*traceable=*/true);
2857: 
2858:   ParsedArgs<5> parsed_args;
2859:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2860:   if(_r.has_torch_function()) {
2861:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2862:   }
2863:   // aten::cosine_embedding_loss(Tensor input1, Tensor input2, Tensor target, float margin=0.0, int reduction=Mean) -> Tensor
2864: 
2865:   auto dispatch_cosine_embedding_loss = [](const at::Tensor & input1, const at::Tensor & input2, const at::Tensor & target, double margin, int64_t reduction) -> at::Tensor {
2866:     pybind11::gil_scoped_release no_gil;
2867:     return at::cosine_embedding_loss(input1, input2, target, margin, reduction);
2868:   };
2869:   return wrap(dispatch_cosine_embedding_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3), _r.toInt64(4)));
2870:   Py_RETURN_NONE;
2871:   END_HANDLE_TH_ERRORS
2872: }
2873: 
2874: // cudnn_affine_grid_generator
2875: static PyObject * THPVariable_cudnn_affine_grid_generator(PyObject* self_, PyObject* args, PyObject* kwargs)
2876: {
2877:   HANDLE_TH_ERRORS
2878:   static PythonArgParser parser({
2879:     "cudnn_affine_grid_generator(Tensor theta, int64_t N, int64_t C, int64_t H, int64_t W)",
2880:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `THPVariable_conv_transpose1d`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_conv_transpose1d`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2881-2960

```cpp
2881: 
2882:   ParsedArgs<5> parsed_args;
2883:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2884:   if(_r.has_torch_function()) {
2885:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2886:   }
2887:   // aten::cudnn_affine_grid_generator(Tensor theta, int N, int C, int H, int W) -> Tensor grid
2888: 
2889:   auto dispatch_cudnn_affine_grid_generator = [](const at::Tensor & theta, int64_t N, int64_t C, int64_t H, int64_t W) -> at::Tensor {
2890:     pybind11::gil_scoped_release no_gil;
2891:     return at::cudnn_affine_grid_generator(theta, N, C, H, W);
2892:   };
2893:   return wrap(dispatch_cudnn_affine_grid_generator(_r.tensor(0), _r.toInt64(1), _r.toInt64(2), _r.toInt64(3), _r.toInt64(4)));
2894:   Py_RETURN_NONE;
2895:   END_HANDLE_TH_ERRORS
2896: }
2897: 
2898: // cudnn_batch_norm
2899: static PyObject * THPVariable_cudnn_batch_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
2900: {
2901:   HANDLE_TH_ERRORS
2902:   static PythonArgParser parser({
2903:     "cudnn_batch_norm(Tensor input, Tensor weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, double exponential_average_factor, double epsilon, *, TensorList[4] out=None)",
2904:   }, /*traceable=*/true);
2905: 
2906:   ParsedArgs<9> parsed_args;
2907:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2908:   if(_r.has_torch_function()) {
2909:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2910:   }
2911:   if (_r.isNone(8)) {
2912:     // aten::cudnn_batch_norm(Tensor input, Tensor weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float exponential_average_factor, float epsilon) -> (Tensor, Tensor, Tensor, Tensor)
2913: 
2914:     auto dispatch_cudnn_batch_norm = [](const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, bool training, double exponential_average_factor, double epsilon) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
2915:       pybind11::gil_scoped_release no_gil;
2916:       return at::cudnn_batch_norm(input, weight, bias, running_mean, running_var, training, exponential_average_factor, epsilon);
2917:     };
2918:     return wrap(dispatch_cudnn_batch_norm(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toBool(5), _r.toDouble(6), _r.toDouble(7)));
2919:   } else {
2920:     // aten::cudnn_batch_norm.out(Tensor input, Tensor weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float exponential_average_factor, float epsilon, *, Tensor(a!) out0, Tensor(b!) out1, Tensor(c!) out2, Tensor(d!) out3) -> (Tensor(a!), Tensor(b!), Tensor(c!), Tensor(d!))
2921:     auto out = _r.tensorlist_n<4>(8);
2922:     auto dispatch_cudnn_batch_norm_out = [](at::Tensor & out0, at::Tensor & out1, at::Tensor & out2, at::Tensor & out3, const at::Tensor & input, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, bool training, double exponential_average_factor, double epsilon) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
2923:       pybind11::gil_scoped_release no_gil;
2924:       return at::cudnn_batch_norm_out(out0, out1, out2, out3, input, weight, bias, running_mean, running_var, training, exponential_average_factor, epsilon);
2925:     };
2926:     return wrap(dispatch_cudnn_batch_norm_out(out[0], out[1], out[2], out[3], _r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toBool(5), _r.toDouble(6), _r.toDouble(7)));
2927:   }
2928:   Py_RETURN_NONE;
2929:   END_HANDLE_TH_ERRORS
2930: }
2931: 
2932: // _mps_convolution_transpose
2933: static PyObject * THPVariable__mps_convolution_transpose(PyObject* self_, PyObject* args, PyObject* kwargs)
2934: {
2935:   HANDLE_TH_ERRORS
2936:   static PythonArgParser parser({
2937:     "_mps_convolution_transpose(Tensor input, Tensor weight, SymIntArrayRef padding, SymIntArrayRef output_padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups)",
2938:   }, /*traceable=*/true);
2939: 
2940:   ParsedArgs<7> parsed_args;
2941:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2942:   if(_r.has_torch_function()) {
2943:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2944:   }
2945:   // aten::_mps_convolution_transpose(Tensor self, Tensor weight, SymInt[] padding, SymInt[] output_padding, SymInt[] stride, SymInt[] dilation, SymInt groups) -> Tensor
2946: 
2947:   auto dispatch__mps_convolution_transpose = [](const at::Tensor & self, const at::Tensor & weight, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
2948:     pybind11::gil_scoped_release no_gil;
2949:     return at::_mps_convolution_transpose_symint(self, weight, padding, output_padding, stride, dilation, groups);
2950:   };
2951:   return wrap(dispatch__mps_convolution_transpose(_r.tensor(0), _r.tensor(1), _r.symintlist(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
2952:   Py_RETURN_NONE;
2953:   END_HANDLE_TH_ERRORS
2954: }
2955: 
2956: // cudnn_convolution_add_relu
2957: static PyObject * THPVariable_cudnn_convolution_add_relu(PyObject* self_, PyObject* args, PyObject* kwargs)
2958: {
2959:   HANDLE_TH_ERRORS
2960:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `cudnn_affine_grid_generator`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `cudnn_affine_grid_generator`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2961-3040

```cpp
2961:     "cudnn_convolution_add_relu(Tensor input, Tensor weight, Tensor z, Scalar? alpha, Tensor? bias, SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation, SymInt groups)",
2962:   }, /*traceable=*/true);
2963: 
2964:   ParsedArgs<9> parsed_args;
2965:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2966:   if(_r.has_torch_function()) {
2967:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2968:   }
2969:   // aten::cudnn_convolution_add_relu(Tensor self, Tensor weight, Tensor z, Scalar? alpha, Tensor? bias, SymInt[] stride, SymInt[] padding, SymInt[] dilation, SymInt groups) -> Tensor
2970: 
2971:   auto dispatch_cudnn_convolution_add_relu = [](const at::Tensor & self, const at::Tensor & weight, const at::Tensor & z, const ::std::optional<at::Scalar> & alpha, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
2972:     pybind11::gil_scoped_release no_gil;
2973:     return at::cudnn_convolution_add_relu_symint(self, weight, z, alpha, bias, stride, padding, dilation, groups);
2974:   };
2975:   return wrap(dispatch_cudnn_convolution_add_relu(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalarOptional(3), _r.optionalTensor(4), _r.symintlist(5), _r.symintlist(6), _r.symintlist(7), _r.toSymInt(8)));
2976:   Py_RETURN_NONE;
2977:   END_HANDLE_TH_ERRORS
2978: }
2979: 
2980: // cudnn_grid_sampler
2981: static PyObject * THPVariable_cudnn_grid_sampler(PyObject* self_, PyObject* args, PyObject* kwargs)
2982: {
2983:   HANDLE_TH_ERRORS
2984:   static PythonArgParser parser({
2985:     "cudnn_grid_sampler(Tensor input, Tensor grid)",
2986:   }, /*traceable=*/true);
2987: 
2988:   ParsedArgs<2> parsed_args;
2989:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
2990:   if(_r.has_torch_function()) {
2991:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
2992:   }
2993:   // aten::cudnn_grid_sampler(Tensor self, Tensor grid) -> Tensor output
2994: 
2995:   auto dispatch_cudnn_grid_sampler = [](const at::Tensor & self, const at::Tensor & grid) -> at::Tensor {
2996:     pybind11::gil_scoped_release no_gil;
2997:     return at::cudnn_grid_sampler(self, grid);
2998:   };
2999:   return wrap(dispatch_cudnn_grid_sampler(_r.tensor(0), _r.tensor(1)));
3000:   Py_RETURN_NONE;
3001:   END_HANDLE_TH_ERRORS
3002: }
3003: 
3004: // _cummax_helper
3005: static PyObject * THPVariable__cummax_helper(PyObject* self_, PyObject* args, PyObject* kwargs)
3006: {
3007:   HANDLE_TH_ERRORS
3008:   static PythonArgParser parser({
3009:     "_cummax_helper(Tensor input, Tensor values, Tensor indices, int64_t dim)",
3010:   }, /*traceable=*/false);
3011: 
3012:   ParsedArgs<4> parsed_args;
3013:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3014:   if(_r.has_torch_function()) {
3015:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3016:   }
3017:   // aten::_cummax_helper(Tensor self, Tensor(a!) values, Tensor(b!) indices, int dim) -> ()
3018: 
3019:   auto dispatch__cummax_helper = [](const at::Tensor & self, at::Tensor values, at::Tensor indices, int64_t dim) -> void {
3020:     pybind11::gil_scoped_release no_gil;
3021:     at::_cummax_helper(self, values, indices, dim);
3022:   };
3023:   dispatch__cummax_helper(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3));
3024:   Py_RETURN_NONE;
3025:   Py_RETURN_NONE;
3026:   END_HANDLE_TH_ERRORS
3027: }
3028: 
3029: \
3030: // cumprod
3031: static PyObject * THPVariable_cumprod(PyObject* self_, PyObject* args, PyObject* kwargs)
3032: {
3033:   HANDLE_TH_ERRORS
3034:   static PythonArgParser parser({
3035:     "cumprod(Tensor input, int64_t dim, *, ScalarType? dtype=None, Tensor out=None)",
3036:     "cumprod(Tensor input, Dimname dim, *, ScalarType? dtype=None, Tensor out=None)",
3037:   }, /*traceable=*/true);
3038: 
3039:   ParsedArgs<4> parsed_args;
3040:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `cudnn_convolution_add_relu`, `cudnn_convolution_add_relu_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `cudnn_convolution_add_relu`, `cudnn_convolution_add_relu_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3041-3120

```cpp
3041:   if(_r.has_torch_function()) {
3042:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3043:   }
3044:   switch (_r.idx) {
3045:     case 0: {
3046:       if (_r.isNone(3)) {
3047:         // aten::cumprod(Tensor self, int dim, *, ScalarType? dtype=None) -> Tensor
3048: 
3049:         auto dispatch_cumprod = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
3050:           pybind11::gil_scoped_release no_gil;
3051:           return self.cumprod(dim, dtype);
3052:         };
3053:         return wrap(dispatch_cumprod(_r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
3054:       } else {
3055:         // aten::cumprod.out(Tensor self, int dim, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
3056: 
3057:         auto dispatch_cumprod_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
3058:           pybind11::gil_scoped_release no_gil;
3059:           return at::cumprod_out(out, self, dim, dtype);
3060:         };
3061:         return wrap(dispatch_cumprod_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.scalartypeOptional(2)));
3062:       }
3063:     }
3064:     case 1: {
3065:       if (_r.isNone(3)) {
3066:         // aten::cumprod.dimname(Tensor self, Dimname dim, *, ScalarType? dtype=None) -> Tensor
3067: 
3068:         auto dispatch_cumprod = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
3069:           pybind11::gil_scoped_release no_gil;
3070:           return self.cumprod(dim, dtype);
3071:         };
3072:         return wrap(dispatch_cumprod(_r.tensor(0), _r.dimname(1), _r.scalartypeOptional(2)));
3073:       } else {
3074:         // aten::cumprod.dimname_out(Tensor self, Dimname dim, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
3075: 
3076:         auto dispatch_cumprod_out = [](at::Tensor out, const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
3077:           pybind11::gil_scoped_release no_gil;
3078:           return at::cumprod_out(out, self, dim, dtype);
3079:         };
3080:         return wrap(dispatch_cumprod_out(_r.tensor(3), _r.tensor(0), _r.dimname(1), _r.scalartypeOptional(2)));
3081:       }
3082:     }
3083:   }
3084:   Py_RETURN_NONE;
3085:   END_HANDLE_TH_ERRORS
3086: }
3087: 
3088: \
3089: // gradient
3090: static PyObject * THPVariable_gradient(PyObject* self_, PyObject* args, PyObject* kwargs)
3091: {
3092:   HANDLE_TH_ERRORS
3093:   static PythonArgParser parser({
3094:     "gradient(Tensor input, *, IntArrayRef dim, int64_t edge_order=1)",
3095:     "gradient(Tensor input, *, Scalar spacing, IntArrayRef dim, int64_t edge_order=1)",
3096:     "gradient(Tensor input, *, Scalar? spacing=None, int64_t? dim=None, int64_t edge_order=1)",
3097:     "gradient(Tensor input, *, ScalarList spacing, int64_t? dim=None, int64_t edge_order=1)",
3098:     "gradient(Tensor input, *, ScalarList spacing, IntArrayRef dim, int64_t edge_order=1)",
3099:     "gradient(Tensor input, *, TensorList spacing, int64_t? dim=None, int64_t edge_order=1)",
3100:     "gradient(Tensor input, *, TensorList spacing, IntArrayRef dim, int64_t edge_order=1)",
3101:   }, /*traceable=*/true);
3102: 
3103:   ParsedArgs<4> parsed_args;
3104:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3105:   if(_r.has_torch_function()) {
3106:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3107:   }
3108:   switch (_r.idx) {
3109:     case 0: {
3110:       // aten::gradient.array(Tensor self, *, int[] dim, int edge_order=1) -> Tensor[]
3111: 
3112:       auto dispatch_gradient = [](const at::Tensor & self, at::IntArrayRef dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3113:         pybind11::gil_scoped_release no_gil;
3114:         return at::gradient(self, dim, edge_order);
3115:       };
3116:       return wrap(dispatch_gradient(_r.tensor(0), _r.intlist(1), _r.toInt64(2)));
3117:     }
3118:     case 1: {
3119:       // aten::gradient.scalararray(Tensor self, *, Scalar spacing, int[] dim, int edge_order=1) -> Tensor[]
3120: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `cumprod`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `cumprod`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3121-3200

```cpp
3121:       auto dispatch_gradient = [](const at::Tensor & self, const at::Scalar & spacing, at::IntArrayRef dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3122:         pybind11::gil_scoped_release no_gil;
3123:         return at::gradient(self, spacing, dim, edge_order);
3124:       };
3125:       return wrap(dispatch_gradient(_r.tensor(0), _r.scalar(1), _r.intlist(2), _r.toInt64(3)));
3126:     }
3127:     case 2: {
3128:       // aten::gradient.scalarint(Tensor self, *, Scalar? spacing=None, int? dim=None, int edge_order=1) -> Tensor[]
3129: 
3130:       auto dispatch_gradient = [](const at::Tensor & self, const ::std::optional<at::Scalar> & spacing, ::std::optional<int64_t> dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3131:         pybind11::gil_scoped_release no_gil;
3132:         return at::gradient(self, spacing, dim, edge_order);
3133:       };
3134:       return wrap(dispatch_gradient(_r.tensor(0), _r.scalarOptional(1), _r.toInt64Optional(2), _r.toInt64(3)));
3135:     }
3136:     case 3: {
3137:       // aten::gradient.scalarrayint(Tensor self, *, Scalar[] spacing, int? dim=None, int edge_order=1) -> Tensor[]
3138: 
3139:       auto dispatch_gradient = [](const at::Tensor & self, at::ArrayRef<at::Scalar> spacing, ::std::optional<int64_t> dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3140:         pybind11::gil_scoped_release no_gil;
3141:         return at::gradient(self, spacing, dim, edge_order);
3142:       };
3143:       return wrap(dispatch_gradient(_r.tensor(0), _r.scalarlist(1), _r.toInt64Optional(2), _r.toInt64(3)));
3144:     }
3145:     case 4: {
3146:       // aten::gradient.scalarrayarray(Tensor self, *, Scalar[] spacing, int[] dim, int edge_order=1) -> Tensor[]
3147: 
3148:       auto dispatch_gradient = [](const at::Tensor & self, at::ArrayRef<at::Scalar> spacing, at::IntArrayRef dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3149:         pybind11::gil_scoped_release no_gil;
3150:         return at::gradient(self, spacing, dim, edge_order);
3151:       };
3152:       return wrap(dispatch_gradient(_r.tensor(0), _r.scalarlist(1), _r.intlist(2), _r.toInt64(3)));
3153:     }
3154:     case 5: {
3155:       // aten::gradient.tensorarrayint(Tensor self, *, Tensor[] spacing, int? dim=None, int edge_order=1) -> Tensor[]
3156: 
3157:       auto dispatch_gradient = [](const at::Tensor & self, at::TensorList spacing, ::std::optional<int64_t> dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3158:         pybind11::gil_scoped_release no_gil;
3159:         return at::gradient(self, spacing, dim, edge_order);
3160:       };
3161:       return wrap(dispatch_gradient(_r.tensor(0), _r.tensorlist(1), _r.toInt64Optional(2), _r.toInt64(3)));
3162:     }
3163:     case 6: {
3164:       // aten::gradient.tensorarray(Tensor self, *, Tensor[] spacing, int[] dim, int edge_order=1) -> Tensor[]
3165: 
3166:       auto dispatch_gradient = [](const at::Tensor & self, at::TensorList spacing, at::IntArrayRef dim, int64_t edge_order) -> ::std::vector<at::Tensor> {
3167:         pybind11::gil_scoped_release no_gil;
3168:         return at::gradient(self, spacing, dim, edge_order);
3169:       };
3170:       return wrap(dispatch_gradient(_r.tensor(0), _r.tensorlist(1), _r.intlist(2), _r.toInt64(3)));
3171:     }
3172:   }
3173:   Py_RETURN_NONE;
3174:   END_HANDLE_TH_ERRORS
3175: }
3176: 
3177: \
3178: // div
3179: static PyObject * THPVariable_div(PyObject* self_, PyObject* args, PyObject* kwargs)
3180: {
3181:   HANDLE_TH_ERRORS
3182:   static PythonArgParser parser({
3183:     "div(Tensor input, Tensor other, *, Tensor out=None)",
3184:     "div(Tensor input, Tensor other, *, c10::string_view? rounding_mode, Tensor out=None)",
3185:     "div(Tensor input, Scalar other, *, c10::string_view? rounding_mode)",
3186:   }, /*traceable=*/true);
3187: 
3188:   ParsedArgs<4> parsed_args;
3189:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3190:   if(_r.has_torch_function()) {
3191:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3192:   }
3193:   switch (_r.idx) {
3194:     case 0: {
3195:       if (_r.isNone(2)) {
3196:         // aten::div.Tensor(Tensor self, Tensor other) -> Tensor
3197: 
3198:         auto dispatch_div = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3199:           pybind11::gil_scoped_release no_gil;
3200:           return self.div(other);
```

- EN: The main execution path in this span is carried by `gradient`, `wrap`, `THPVariable_div`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `gradient`, `wrap`, `THPVariable_div` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3201-3280

```cpp
3201:         };
3202:         return wrap(dispatch_div(_r.tensor(0), _r.tensor(1)));
3203:       } else {
3204:         // aten::div.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
3205: 
3206:         auto dispatch_div_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3207:           pybind11::gil_scoped_release no_gil;
3208:           return at::div_out(out, self, other);
3209:         };
3210:         return wrap(dispatch_div_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
3211:       }
3212:     }
3213:     case 1: {
3214:       if (_r.isNone(3)) {
3215:         // aten::div.Tensor_mode(Tensor self, Tensor other, *, str? rounding_mode) -> Tensor
3216: 
3217:         auto dispatch_div = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
3218:           pybind11::gil_scoped_release no_gil;
3219:           return self.div(other, rounding_mode);
3220:         };
3221:         return wrap(dispatch_div(_r.tensor(0), _r.tensor(1), _r.stringViewOptional(2)));
3222:       } else {
3223:         // aten::div.out_mode(Tensor self, Tensor other, *, str? rounding_mode, Tensor(a!) out) -> Tensor(a!)
3224: 
3225:         auto dispatch_div_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
3226:           pybind11::gil_scoped_release no_gil;
3227:           return at::div_out(out, self, other, rounding_mode);
3228:         };
3229:         return wrap(dispatch_div_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.stringViewOptional(2)));
3230:       }
3231:     }
3232:     case 2: {
3233:       // aten::div.Scalar_mode(Tensor self, Scalar other, *, str? rounding_mode) -> Tensor
3234: 
3235:       auto dispatch_div = [](const at::Tensor & self, const at::Scalar & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
3236:         pybind11::gil_scoped_release no_gil;
3237:         return self.div(other, rounding_mode);
3238:       };
3239:       return wrap(dispatch_div(_r.tensor(0), _r.scalar(1), _r.stringViewOptional(2)));
3240:     }
3241:   }
3242:   Py_RETURN_NONE;
3243:   END_HANDLE_TH_ERRORS
3244: }
3245: 
3246: \
3247: // true_divide
3248: static PyObject * THPVariable_true_divide(PyObject* self_, PyObject* args, PyObject* kwargs)
3249: {
3250:   HANDLE_TH_ERRORS
3251:   static PythonArgParser parser({
3252:     "true_divide(Tensor input, Tensor other, *, Tensor out=None)",
3253:     "true_divide(Tensor input, Scalar other)",
3254:   }, /*traceable=*/true);
3255: 
3256:   ParsedArgs<3> parsed_args;
3257:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3258:   if(_r.has_torch_function()) {
3259:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3260:   }
3261:   switch (_r.idx) {
3262:     case 0: {
3263:       if (_r.isNone(2)) {
3264:         // aten::true_divide.Tensor(Tensor self, Tensor other) -> Tensor
3265: 
3266:         auto dispatch_true_divide = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3267:           pybind11::gil_scoped_release no_gil;
3268:           return self.true_divide(other);
3269:         };
3270:         return wrap(dispatch_true_divide(_r.tensor(0), _r.tensor(1)));
3271:       } else {
3272:         // aten::true_divide.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
3273: 
3274:         auto dispatch_true_divide_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3275:           pybind11::gil_scoped_release no_gil;
3276:           return at::true_divide_out(out, self, other);
3277:         };
3278:         return wrap(dispatch_true_divide_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
3279:       }
3280:     }
```

- EN: The main execution path in this span is carried by `wrap`, `div_out`, `THPVariable_true_divide`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `div_out`, `THPVariable_true_divide` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3281-3360

```cpp
3281:     case 1: {
3282:       // aten::true_divide.Scalar(Tensor self, Scalar other) -> Tensor
3283: 
3284:       auto dispatch_true_divide = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
3285:         pybind11::gil_scoped_release no_gil;
3286:         return self.true_divide(other);
3287:       };
3288:       return wrap(dispatch_true_divide(_r.tensor(0), _r.scalar(1)));
3289:     }
3290:   }
3291:   Py_RETURN_NONE;
3292:   END_HANDLE_TH_ERRORS
3293: }
3294: 
3295: // _embedding_bag_forward_only
3296: static PyObject * THPVariable__embedding_bag_forward_only(PyObject* self_, PyObject* args, PyObject* kwargs)
3297: {
3298:   HANDLE_TH_ERRORS
3299:   static PythonArgParser parser({
3300:     "_embedding_bag_forward_only(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq=False, int64_t mode=0, bool sparse=False, Tensor? per_sample_weights=None, bool include_last_offset=False, int64_t padding_idx=-1)",
3301:   }, /*traceable=*/true);
3302: 
3303:   ParsedArgs<9> parsed_args;
3304:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3305:   if(_r.has_torch_function()) {
3306:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3307:   }
3308:   // aten::_embedding_bag_forward_only(Tensor weight, Tensor indices, Tensor offsets, bool scale_grad_by_freq=False, int mode=0, bool sparse=False, Tensor? per_sample_weights=None, bool include_last_offset=False, int padding_idx=-1) -> (Tensor, Tensor, Tensor, Tensor)
3309: 
3310:   auto dispatch__embedding_bag_forward_only = [](const at::Tensor & weight, const at::Tensor & indices, const at::Tensor & offsets, bool scale_grad_by_freq, int64_t mode, bool sparse, const ::std::optional<at::Tensor> & per_sample_weights, bool include_last_offset, int64_t padding_idx) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
3311:     pybind11::gil_scoped_release no_gil;
3312:     return at::_embedding_bag_forward_only(weight, indices, offsets, scale_grad_by_freq, mode, sparse, per_sample_weights, include_last_offset, padding_idx);
3313:   };
3314:   return wrap(dispatch__embedding_bag_forward_only(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3), _r.toInt64(4), _r.toBool(5), _r.optionalTensor(6), _r.toBool(7), _r.toInt64(8)));
3315:   Py_RETURN_NONE;
3316:   END_HANDLE_TH_ERRORS
3317: }
3318: 
3319: // row_stack
3320: static PyObject * THPVariable_row_stack(PyObject* self_, PyObject* args, PyObject* kwargs)
3321: {
3322:   HANDLE_TH_ERRORS
3323:   static PythonArgParser parser({
3324:     "row_stack(TensorList tensors, *, Tensor out=None)",
3325:   }, /*traceable=*/true);
3326: 
3327:   ParsedArgs<2> parsed_args;
3328:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3329:   if(_r.has_torch_function()) {
3330:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3331:   }
3332:   if (_r.isNone(1)) {
3333:     // aten::row_stack(Tensor[] tensors) -> Tensor
3334: 
3335:     auto dispatch_row_stack = [](at::TensorList tensors) -> at::Tensor {
3336:       pybind11::gil_scoped_release no_gil;
3337:       return at::row_stack(tensors);
3338:     };
3339:     return wrap(dispatch_row_stack(_r.tensorlist(0)));
3340:   } else {
3341:     // aten::row_stack.out(Tensor[] tensors, *, Tensor(a!) out) -> Tensor(a!)
3342: 
3343:     auto dispatch_row_stack_out = [](at::Tensor out, at::TensorList tensors) -> at::Tensor {
3344:       pybind11::gil_scoped_release no_gil;
3345:       return at::row_stack_out(out, tensors);
3346:     };
3347:     return wrap(dispatch_row_stack_out(_r.tensor(1), _r.tensorlist(0)));
3348:   }
3349:   Py_RETURN_NONE;
3350:   END_HANDLE_TH_ERRORS
3351: }
3352: 
3353: // _empty_per_channel_affine_quantized
3354: static PyObject * THPVariable__empty_per_channel_affine_quantized(PyObject* self_, PyObject* args, PyObject* kwargs)
3355: {
3356:   HANDLE_TH_ERRORS
3357:   static PythonArgParser parser({
3358:     "_empty_per_channel_affine_quantized(SymIntArrayRef size, *, Tensor scales, Tensor zero_points, int64_t axis, MemoryFormat? memory_format=c10::MemoryFormat::Contiguous, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3359:   }, /*traceable=*/true);
3360: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__embedding_bag_forward_only`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__embedding_bag_forward_only`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 3361-3440

```cpp
3361:   ParsedArgs<10> parsed_args;
3362:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3363:   if(_r.has_torch_function()) {
3364:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3365:   }
3366:   // aten::_empty_per_channel_affine_quantized(SymInt[] size, *, Tensor scales, Tensor zero_points, int axis, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=contiguous_format) -> Tensor
3367:   const auto options = TensorOptions()
3368:       .dtype(_r.scalartypeOptional(5))
3369:       .device(_r.deviceWithDefault(7, torch::tensors::get_default_device()))
3370:       .layout(_r.layoutOptional(6))
3371:       .requires_grad(_r.toBool(9))
3372:       .pinned_memory(_r.toBool(8));
3373:   torch::utils::maybe_initialize_device(options);
3374: 
3375:   auto dispatch__empty_per_channel_affine_quantized = [](c10::SymIntArrayRef size, const at::Tensor & scales, const at::Tensor & zero_points, int64_t axis, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
3376:     pybind11::gil_scoped_release no_gil;
3377:     return torch::_empty_per_channel_affine_quantized_symint(size, scales, zero_points, axis, options, memory_format);
3378:   };
3379:   return wrap(dispatch__empty_per_channel_affine_quantized(_r.symintlist(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3), options, _r.memoryformat(4)));
3380:   Py_RETURN_NONE;
3381:   END_HANDLE_TH_ERRORS
3382: }
3383: 
3384: // _resize_output_
3385: static PyObject * THPVariable__resize_output_(PyObject* self_, PyObject* args, PyObject* kwargs)
3386: {
3387:   HANDLE_TH_ERRORS
3388:   static PythonArgParser parser({
3389:     "_resize_output_(Tensor input, SymIntArrayRef size, Device device)",
3390:   }, /*traceable=*/true);
3391: 
3392:   ParsedArgs<3> parsed_args;
3393:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3394:   if(_r.has_torch_function()) {
3395:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3396:   }
3397:   // aten::_resize_output_(Tensor(a!) self, SymInt[] size, Device device) -> Tensor(a!)
3398: 
3399:   auto dispatch__resize_output_ = [](const at::Tensor & self, c10::SymIntArrayRef size, at::Device device) -> at::Tensor {
3400:     pybind11::gil_scoped_release no_gil;
3401:     return at::_resize_output__symint(self, size, device);
3402:   };
3403:   return wrap(dispatch__resize_output_(_r.tensor(0), _r.symintlist(1), _r.device(2)));
3404:   Py_RETURN_NONE;
3405:   END_HANDLE_TH_ERRORS
3406: }
3407: 
3408: // empty_strided
3409: static PyObject * THPVariable_empty_strided(PyObject* self_, PyObject* args, PyObject* kwargs)
3410: {
3411:   HANDLE_TH_ERRORS
3412:   static PythonArgParser parser({
3413:     "empty_strided(SymIntArrayRef size, SymIntArrayRef stride, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3414:   }, /*traceable=*/true);
3415: 
3416:   ParsedArgs<7> parsed_args;
3417:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3418:   if(_r.has_torch_function()) {
3419:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3420:   }
3421:   // aten::empty_strided(SymInt[] size, SymInt[] stride, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
3422:   const auto options = TensorOptions()
3423:       .dtype(_r.scalartypeOptional(2))
3424:       .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
3425:       .layout(_r.layoutOptional(3))
3426:       .requires_grad(_r.toBool(6))
3427:       .pinned_memory(_r.toBool(5));
3428:   torch::utils::maybe_initialize_device(options);
3429: 
3430:   auto dispatch_empty_strided = [](c10::SymIntArrayRef size, c10::SymIntArrayRef stride, at::TensorOptions options) -> at::Tensor {
3431:     pybind11::gil_scoped_release no_gil;
3432:     return torch::empty_strided_symint(size, stride, options);
3433:   };
3434:   return wrap(dispatch_empty_strided(_r.symintlist(0), _r.symintlist(1), options));
3435:   Py_RETURN_NONE;
3436:   END_HANDLE_TH_ERRORS
3437: }
3438: 
3439: \
3440: // flatten
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_empty_per_channel_affine_quantized`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_empty_per_channel_affine_quantized`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 3441-3520

```cpp
3441: static PyObject * THPVariable_flatten(PyObject* self_, PyObject* args, PyObject* kwargs)
3442: {
3443:   HANDLE_TH_ERRORS
3444:   static PythonArgParser parser({
3445:     "flatten(Tensor input, int64_t start_dim, int64_t end_dim, Dimname out_dim)",
3446:     "flatten(Tensor input, int64_t start_dim=0, int64_t end_dim=-1)",
3447:     "flatten(Tensor input, Dimname start_dim, Dimname end_dim, Dimname out_dim)",
3448:     "flatten(Tensor input, DimnameList dims, Dimname out_dim)",
3449:   }, /*traceable=*/true);
3450: 
3451:   ParsedArgs<4> parsed_args;
3452:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3453:   if(_r.has_torch_function()) {
3454:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3455:   }
3456:   switch (_r.idx) {
3457:     case 0: {
3458:       // aten::flatten.named_out_dim(Tensor(a) self, int start_dim, int end_dim, Dimname out_dim) -> Tensor(a)
3459: 
3460:       auto dispatch_flatten = [](const at::Tensor & self, int64_t start_dim, int64_t end_dim, at::Dimname out_dim) -> at::Tensor {
3461:         pybind11::gil_scoped_release no_gil;
3462:         return self.flatten(start_dim, end_dim, out_dim);
3463:       };
3464:       return wrap(dispatch_flatten(_r.tensor(0), _r.toInt64(1), _r.toInt64(2), _r.dimname(3)));
3465:     }
3466:     case 1: {
3467:       // aten::flatten.using_ints(Tensor(a) self, int start_dim=0, int end_dim=-1) -> Tensor(a)
3468: 
3469:       auto dispatch_flatten = [](const at::Tensor & self, int64_t start_dim, int64_t end_dim) -> at::Tensor {
3470:         pybind11::gil_scoped_release no_gil;
3471:         return self.flatten(start_dim, end_dim);
3472:       };
3473:       return wrap(dispatch_flatten(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
3474:     }
3475:     case 2: {
3476:       // aten::flatten.using_names(Tensor(a) self, Dimname start_dim, Dimname end_dim, Dimname out_dim) -> Tensor(a)
3477: 
3478:       auto dispatch_flatten = [](const at::Tensor & self, at::Dimname start_dim, at::Dimname end_dim, at::Dimname out_dim) -> at::Tensor {
3479:         pybind11::gil_scoped_release no_gil;
3480:         return self.flatten(start_dim, end_dim, out_dim);
3481:       };
3482:       return wrap(dispatch_flatten(_r.tensor(0), _r.dimname(1), _r.dimname(2), _r.dimname(3)));
3483:     }
3484:     case 3: {
3485:       // aten::flatten.DimnameList(Tensor(a) self, Dimname[] dims, Dimname out_dim) -> Tensor(a)
3486: 
3487:       auto dispatch_flatten = [](const at::Tensor & self, at::DimnameList dims, at::Dimname out_dim) -> at::Tensor {
3488:         pybind11::gil_scoped_release no_gil;
3489:         return self.flatten(dims, out_dim);
3490:       };
3491:       return wrap(dispatch_flatten(_r.tensor(0), _r.dimnamelist(1), _r.dimname(2)));
3492:     }
3493:   }
3494:   Py_RETURN_NONE;
3495:   END_HANDLE_TH_ERRORS
3496: }
3497: 
3498: \
3499: // fill
3500: static PyObject * THPVariable_fill(PyObject* self_, PyObject* args, PyObject* kwargs)
3501: {
3502:   HANDLE_TH_ERRORS
3503:   static PythonArgParser parser({
3504:     "fill(Tensor input, Tensor value)",
3505:     "fill(Tensor input, Scalar value)",
3506:   }, /*traceable=*/true);
3507: 
3508:   ParsedArgs<2> parsed_args;
3509:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3510:   if(_r.has_torch_function()) {
3511:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3512:   }
3513:   switch (_r.idx) {
3514:     case 0: {
3515:       // aten::fill.Tensor(Tensor self, Tensor value) -> Tensor
3516: 
3517:       auto dispatch_fill = [](const at::Tensor & self, const at::Tensor & value) -> at::Tensor {
3518:         pybind11::gil_scoped_release no_gil;
3519:         return at::fill(self, value);
3520:       };
```

- EN: The main execution path in this span is carried by `THPVariable_flatten`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_flatten`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3521-3600

```cpp
3521:       return wrap(dispatch_fill(_r.tensor(0), _r.tensor(1)));
3522:     }
3523:     case 1: {
3524:       // aten::fill.Scalar(Tensor self, Scalar value) -> Tensor
3525: 
3526:       auto dispatch_fill = [](const at::Tensor & self, const at::Scalar & value) -> at::Tensor {
3527:         pybind11::gil_scoped_release no_gil;
3528:         return at::fill(self, value);
3529:       };
3530:       return wrap(dispatch_fill(_r.tensor(0), _r.scalar(1)));
3531:     }
3532:   }
3533:   Py_RETURN_NONE;
3534:   END_HANDLE_TH_ERRORS
3535: }
3536: 
3537: \
3538: // fill_
3539: static PyObject * THPVariable_fill_(PyObject* self_, PyObject* args, PyObject* kwargs)
3540: {
3541:   HANDLE_TH_ERRORS
3542:   static PythonArgParser parser({
3543:     "fill_(Tensor input, Tensor value)",
3544:     "fill_(Tensor input, Scalar value)",
3545:   }, /*traceable=*/true);
3546: 
3547:   ParsedArgs<2> parsed_args;
3548:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3549:   if(_r.has_torch_function()) {
3550:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3551:   }
3552:   switch (_r.idx) {
3553:     case 0: {
3554:       // aten::fill_.Tensor(Tensor(a!) self, Tensor value) -> Tensor(a!)
3555: 
3556:       auto dispatch_fill_ = [](at::Tensor self, const at::Tensor & value) -> at::Tensor {
3557:         pybind11::gil_scoped_release no_gil;
3558:         return self.fill_(value);
3559:       };
3560:       return wrap(dispatch_fill_(_r.tensor(0), _r.tensor(1)));
3561:     }
3562:     case 1: {
3563:       // aten::fill_.Scalar(Tensor(a!) self, Scalar value) -> Tensor(a!)
3564: 
3565:       auto dispatch_fill_ = [](at::Tensor self, const at::Scalar & value) -> at::Tensor {
3566:         pybind11::gil_scoped_release no_gil;
3567:         return self.fill_(value);
3568:       };
3569:       return wrap(dispatch_fill_(_r.tensor(0), _r.scalar(1)));
3570:     }
3571:   }
3572:   Py_RETURN_NONE;
3573:   END_HANDLE_TH_ERRORS
3574: }
3575: 
3576: // grid_sampler_2d
3577: static PyObject * THPVariable_grid_sampler_2d(PyObject* self_, PyObject* args, PyObject* kwargs)
3578: {
3579:   HANDLE_TH_ERRORS
3580:   static PythonArgParser parser({
3581:     "grid_sampler_2d(Tensor input, Tensor grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners)",
3582:   }, /*traceable=*/true);
3583: 
3584:   ParsedArgs<5> parsed_args;
3585:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3586:   if(_r.has_torch_function()) {
3587:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3588:   }
3589:   // aten::grid_sampler_2d(Tensor input, Tensor grid, int interpolation_mode, int padding_mode, bool align_corners) -> Tensor
3590: 
3591:   auto dispatch_grid_sampler_2d = [](const at::Tensor & input, const at::Tensor & grid, int64_t interpolation_mode, int64_t padding_mode, bool align_corners) -> at::Tensor {
3592:     pybind11::gil_scoped_release no_gil;
3593:     return at::grid_sampler_2d(input, grid, interpolation_mode, padding_mode, align_corners);
3594:   };
3595:   return wrap(dispatch_grid_sampler_2d(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toInt64(3), _r.toBool(4)));
3596:   Py_RETURN_NONE;
3597:   END_HANDLE_TH_ERRORS
3598: }
3599: 
3600: \
```

- EN: The main execution path in this span is carried by `wrap`, `fill`, `THPVariable_fill_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `fill`, `THPVariable_fill_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3601-3680

```cpp
3601: // hann_window
3602: static PyObject * THPVariable_hann_window(PyObject* self_, PyObject* args, PyObject* kwargs)
3603: {
3604:   HANDLE_TH_ERRORS
3605:   static PythonArgParser parser({
3606:     "hann_window(int64_t window_length, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3607:     "hann_window(int64_t window_length, bool periodic, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3608:   }, /*traceable=*/true);
3609: 
3610:   ParsedArgs<7> parsed_args;
3611:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3612:   if(_r.has_torch_function()) {
3613:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3614:   }
3615:   switch (_r.idx) {
3616:     case 0: {
3617:       // aten::hann_window(int window_length, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
3618:       const auto options = TensorOptions()
3619:           .dtype(_r.scalartypeOptional(1))
3620:           .device(_r.deviceWithDefault(3, torch::tensors::get_default_device()))
3621:           .layout(_r.layoutOptional(2))
3622:           .requires_grad(_r.toBool(5))
3623:           .pinned_memory(_r.toBool(4));
3624:       torch::utils::maybe_initialize_device(options);
3625: 
3626:       auto dispatch_hann_window = [](int64_t window_length, at::TensorOptions options) -> at::Tensor {
3627:         pybind11::gil_scoped_release no_gil;
3628:         return torch::hann_window(window_length, options);
3629:       };
3630:       return wrap(dispatch_hann_window(_r.toInt64(0), options));
3631:     }
3632:     case 1: {
3633:       // aten::hann_window.periodic(int window_length, bool periodic, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
3634:       const auto options = TensorOptions()
3635:           .dtype(_r.scalartypeOptional(2))
3636:           .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
3637:           .layout(_r.layoutOptional(3))
3638:           .requires_grad(_r.toBool(6))
3639:           .pinned_memory(_r.toBool(5));
3640:       torch::utils::maybe_initialize_device(options);
3641: 
3642:       auto dispatch_hann_window = [](int64_t window_length, bool periodic, at::TensorOptions options) -> at::Tensor {
3643:         pybind11::gil_scoped_release no_gil;
3644:         return torch::hann_window(window_length, periodic, options);
3645:       };
3646:       return wrap(dispatch_hann_window(_r.toInt64(0), _r.toBool(1), options));
3647:     }
3648:   }
3649:   Py_RETURN_NONE;
3650:   END_HANDLE_TH_ERRORS
3651: }
3652: 
3653: \
3654: // kaiser_window
3655: static PyObject * THPVariable_kaiser_window(PyObject* self_, PyObject* args, PyObject* kwargs)
3656: {
3657:   HANDLE_TH_ERRORS
3658:   static PythonArgParser parser({
3659:     "kaiser_window(int64_t window_length, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3660:     "kaiser_window(int64_t window_length, bool periodic, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3661:     "kaiser_window(int64_t window_length, bool periodic, double beta, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
3662:   }, /*traceable=*/true);
3663: 
3664:   ParsedArgs<8> parsed_args;
3665:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3666:   if(_r.has_torch_function()) {
3667:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3668:   }
3669:   switch (_r.idx) {
3670:     case 0: {
3671:       // aten::kaiser_window(int window_length, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
3672:       const auto options = TensorOptions()
3673:           .dtype(_r.scalartypeOptional(1))
3674:           .device(_r.deviceWithDefault(3, torch::tensors::get_default_device()))
3675:           .layout(_r.layoutOptional(2))
3676:           .requires_grad(_r.toBool(5))
3677:           .pinned_memory(_r.toBool(4));
3678:       torch::utils::maybe_initialize_device(options);
3679: 
3680:       auto dispatch_kaiser_window = [](int64_t window_length, at::TensorOptions options) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable_hann_window`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_hann_window`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3681-3760

```cpp
3681:         pybind11::gil_scoped_release no_gil;
3682:         return torch::kaiser_window(window_length, options);
3683:       };
3684:       return wrap(dispatch_kaiser_window(_r.toInt64(0), options));
3685:     }
3686:     case 1: {
3687:       // aten::kaiser_window.periodic(int window_length, bool periodic, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
3688:       const auto options = TensorOptions()
3689:           .dtype(_r.scalartypeOptional(2))
3690:           .device(_r.deviceWithDefault(4, torch::tensors::get_default_device()))
3691:           .layout(_r.layoutOptional(3))
3692:           .requires_grad(_r.toBool(6))
3693:           .pinned_memory(_r.toBool(5));
3694:       torch::utils::maybe_initialize_device(options);
3695: 
3696:       auto dispatch_kaiser_window = [](int64_t window_length, bool periodic, at::TensorOptions options) -> at::Tensor {
3697:         pybind11::gil_scoped_release no_gil;
3698:         return torch::kaiser_window(window_length, periodic, options);
3699:       };
3700:       return wrap(dispatch_kaiser_window(_r.toInt64(0), _r.toBool(1), options));
3701:     }
3702:     case 2: {
3703:       // aten::kaiser_window.beta(int window_length, bool periodic, float beta, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
3704:       const auto options = TensorOptions()
3705:           .dtype(_r.scalartypeOptional(3))
3706:           .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
3707:           .layout(_r.layoutOptional(4))
3708:           .requires_grad(_r.toBool(7))
3709:           .pinned_memory(_r.toBool(6));
3710:       torch::utils::maybe_initialize_device(options);
3711: 
3712:       auto dispatch_kaiser_window = [](int64_t window_length, bool periodic, double beta, at::TensorOptions options) -> at::Tensor {
3713:         pybind11::gil_scoped_release no_gil;
3714:         return torch::kaiser_window(window_length, periodic, beta, options);
3715:       };
3716:       return wrap(dispatch_kaiser_window(_r.toInt64(0), _r.toBool(1), _r.toDouble(2), options));
3717:     }
3718:   }
3719:   Py_RETURN_NONE;
3720:   END_HANDLE_TH_ERRORS
3721: }
3722: 
3723: // _fft_r2c
3724: static PyObject * THPVariable__fft_r2c(PyObject* self_, PyObject* args, PyObject* kwargs)
3725: {
3726:   HANDLE_TH_ERRORS
3727:   static PythonArgParser parser({
3728:     "_fft_r2c(Tensor input, IntArrayRef dim, int64_t normalization, bool onesided, *, Tensor out=None)",
3729:   }, /*traceable=*/true);
3730: 
3731:   ParsedArgs<5> parsed_args;
3732:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3733:   if(_r.has_torch_function()) {
3734:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3735:   }
3736:   if (_r.isNone(4)) {
3737:     // aten::_fft_r2c(Tensor self, int[] dim, int normalization, bool onesided) -> Tensor
3738: 
3739:     auto dispatch__fft_r2c = [](const at::Tensor & self, at::IntArrayRef dim, int64_t normalization, bool onesided) -> at::Tensor {
3740:       pybind11::gil_scoped_release no_gil;
3741:       return at::_fft_r2c(self, dim, normalization, onesided);
3742:     };
3743:     return wrap(dispatch__fft_r2c(_r.tensor(0), _r.intlist(1), _r.toInt64(2), _r.toBool(3)));
3744:   } else {
3745:     // aten::_fft_r2c.out(Tensor self, int[] dim, int normalization, bool onesided, *, Tensor(a!) out) -> Tensor(a!)
3746: 
3747:     auto dispatch__fft_r2c_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, int64_t normalization, bool onesided) -> at::Tensor {
3748:       pybind11::gil_scoped_release no_gil;
3749:       return at::_fft_r2c_out(out, self, dim, normalization, onesided);
3750:     };
3751:     return wrap(dispatch__fft_r2c_out(_r.tensor(4), _r.tensor(0), _r.intlist(1), _r.toInt64(2), _r.toBool(3)));
3752:   }
3753:   Py_RETURN_NONE;
3754:   END_HANDLE_TH_ERRORS
3755: }
3756: 
3757: // _validate_compressed_sparse_indices
3758: static PyObject * THPVariable__validate_compressed_sparse_indices(PyObject* self_, PyObject* args, PyObject* kwargs)
3759: {
3760:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `kaiser_window`, `wrap`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `kaiser_window`, `wrap`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 3761-3840

```cpp
3761:   static PythonArgParser parser({
3762:     "_validate_compressed_sparse_indices(bool is_crow, Tensor compressed_idx, Tensor plain_idx, int64_t cdim, int64_t dim, int64_t nnz)",
3763:   }, /*traceable=*/false);
3764: 
3765:   ParsedArgs<6> parsed_args;
3766:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3767:   if(_r.has_torch_function()) {
3768:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3769:   }
3770:   // aten::_validate_compressed_sparse_indices(bool is_crow, Tensor compressed_idx, Tensor plain_idx, int cdim, int dim, int nnz) -> ()
3771: 
3772:   auto dispatch__validate_compressed_sparse_indices = [](bool is_crow, const at::Tensor & compressed_idx, const at::Tensor & plain_idx, int64_t cdim, int64_t dim, int64_t nnz) -> void {
3773:     pybind11::gil_scoped_release no_gil;
3774:     at::_validate_compressed_sparse_indices(is_crow, compressed_idx, plain_idx, cdim, dim, nnz);
3775:   };
3776:   dispatch__validate_compressed_sparse_indices(_r.toBool(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3), _r.toInt64(4), _r.toInt64(5));
3777:   Py_RETURN_NONE;
3778:   Py_RETURN_NONE;
3779:   END_HANDLE_TH_ERRORS
3780: }
3781: 
3782: // _cufft_get_plan_cache_size
3783: static PyObject * THPVariable__cufft_get_plan_cache_size(PyObject* self_, PyObject* args, PyObject* kwargs)
3784: {
3785:   HANDLE_TH_ERRORS
3786:   static PythonArgParser parser({
3787:     "_cufft_get_plan_cache_size(DeviceIndex device_index)",
3788:   }, /*traceable=*/false);
3789: 
3790:   ParsedArgs<1> parsed_args;
3791:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3792:   if(_r.has_torch_function()) {
3793:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3794:   }
3795:   // aten::_cufft_get_plan_cache_size(DeviceIndex device_index) -> int
3796: 
3797:   auto dispatch__cufft_get_plan_cache_size = [](at::DeviceIndex device_index) -> int64_t {
3798:     pybind11::gil_scoped_release no_gil;
3799:     return at::_cufft_get_plan_cache_size(device_index);
3800:   };
3801:   return wrap(dispatch__cufft_get_plan_cache_size(_r.toInt64(0)));
3802:   Py_RETURN_NONE;
3803:   END_HANDLE_TH_ERRORS
3804: }
3805: 
3806: // _unsafe_masked_index
3807: static PyObject * THPVariable__unsafe_masked_index(PyObject* self_, PyObject* args, PyObject* kwargs)
3808: {
3809:   HANDLE_TH_ERRORS
3810:   static PythonArgParser parser({
3811:     "_unsafe_masked_index(Tensor input, Tensor mask, c10::List<::std::optional<Tensor>> indices, Scalar fill)",
3812:   }, /*traceable=*/true);
3813: 
3814:   ParsedArgs<4> parsed_args;
3815:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3816:   if(_r.has_torch_function()) {
3817:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3818:   }
3819:   // aten::_unsafe_masked_index(Tensor self, Tensor mask, Tensor?[] indices, Scalar fill) -> Tensor
3820: 
3821:   auto dispatch__unsafe_masked_index = [](const at::Tensor & self, const at::Tensor & mask, const c10::List<::std::optional<at::Tensor>> & indices, const at::Scalar & fill) -> at::Tensor {
3822:     pybind11::gil_scoped_release no_gil;
3823:     return at::_unsafe_masked_index(self, mask, indices, fill);
3824:   };
3825:   return wrap(dispatch__unsafe_masked_index(_r.tensor(0), _r.tensor(1), _r.list_of_optional_tensors(2), _r.scalar(3)));
3826:   Py_RETURN_NONE;
3827:   END_HANDLE_TH_ERRORS
3828: }
3829: 
3830: // index_put_
3831: static PyObject * THPVariable_index_put_(PyObject* self_, PyObject* args, PyObject* kwargs)
3832: {
3833:   HANDLE_TH_ERRORS
3834:   static PythonArgParser parser({
3835:     "index_put_(Tensor input, c10::List<::std::optional<Tensor>> indices, Tensor values, bool accumulate=False)",
3836:   }, /*traceable=*/true);
3837: 
3838:   ParsedArgs<4> parsed_args;
3839:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3840:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_validate_compressed_sparse_indices`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_validate_compressed_sparse_indices` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3841-3920

```cpp
3841:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3842:   }
3843:   // aten::index_put_(Tensor(a!) self, Tensor?[] indices, Tensor values, bool accumulate=False) -> Tensor(a!)
3844: 
3845:   auto dispatch_index_put_ = [](at::Tensor self, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values, bool accumulate) -> at::Tensor {
3846:     pybind11::gil_scoped_release no_gil;
3847:     return self.index_put_(indices, values, accumulate);
3848:   };
3849:   return wrap(dispatch_index_put_(_r.tensor(0), _r.list_of_optional_tensors(1), _r.tensor(2), _r.toBool(3)));
3850:   Py_RETURN_NONE;
3851:   END_HANDLE_TH_ERRORS
3852: }
3853: 
3854: // index_put
3855: static PyObject * THPVariable_index_put(PyObject* self_, PyObject* args, PyObject* kwargs)
3856: {
3857:   HANDLE_TH_ERRORS
3858:   static PythonArgParser parser({
3859:     "index_put(Tensor input, c10::List<::std::optional<Tensor>> indices, Tensor values, bool accumulate=False)",
3860:   }, /*traceable=*/true);
3861: 
3862:   ParsedArgs<4> parsed_args;
3863:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3864:   if(_r.has_torch_function()) {
3865:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3866:   }
3867:   // aten::index_put(Tensor self, Tensor?[] indices, Tensor values, bool accumulate=False) -> Tensor
3868: 
3869:   auto dispatch_index_put = [](const at::Tensor & self, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values, bool accumulate) -> at::Tensor {
3870:     pybind11::gil_scoped_release no_gil;
3871:     return self.index_put(indices, values, accumulate);
3872:   };
3873:   return wrap(dispatch_index_put(_r.tensor(0), _r.list_of_optional_tensors(1), _r.tensor(2), _r.toBool(3)));
3874:   Py_RETURN_NONE;
3875:   END_HANDLE_TH_ERRORS
3876: }
3877: 
3878: // _unsafe_index_put
3879: static PyObject * THPVariable__unsafe_index_put(PyObject* self_, PyObject* args, PyObject* kwargs)
3880: {
3881:   HANDLE_TH_ERRORS
3882:   static PythonArgParser parser({
3883:     "_unsafe_index_put(Tensor input, c10::List<::std::optional<Tensor>> indices, Tensor values, bool accumulate=False)",
3884:   }, /*traceable=*/true);
3885: 
3886:   ParsedArgs<4> parsed_args;
3887:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3888:   if(_r.has_torch_function()) {
3889:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3890:   }
3891:   // aten::_unsafe_index_put(Tensor self, Tensor?[] indices, Tensor values, bool accumulate=False) -> Tensor
3892: 
3893:   auto dispatch__unsafe_index_put = [](const at::Tensor & self, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values, bool accumulate) -> at::Tensor {
3894:     pybind11::gil_scoped_release no_gil;
3895:     return at::_unsafe_index_put(self, indices, values, accumulate);
3896:   };
3897:   return wrap(dispatch__unsafe_index_put(_r.tensor(0), _r.list_of_optional_tensors(1), _r.tensor(2), _r.toBool(3)));
3898:   Py_RETURN_NONE;
3899:   END_HANDLE_TH_ERRORS
3900: }
3901: 
3902: // is_distributed
3903: static PyObject * THPVariable_is_distributed(PyObject* self_, PyObject* args, PyObject* kwargs)
3904: {
3905:   HANDLE_TH_ERRORS
3906:   static PythonArgParser parser({
3907:     "is_distributed(Tensor input)",
3908:   }, /*traceable=*/false);
3909: 
3910:   ParsedArgs<1> parsed_args;
3911:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3912:   if(_r.has_torch_function()) {
3913:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3914:   }
3915:   // aten::is_distributed(Tensor self) -> bool
3916: 
3917:   auto dispatch_is_distributed = [](const at::Tensor & self) -> bool {
3918:     pybind11::gil_scoped_release no_gil;
3919:     return self.is_distributed();
3920:   };
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `index_put_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `index_put_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3921-4000

```cpp
3921:   return wrap(dispatch_is_distributed(_r.tensor(0)));
3922:   Py_RETURN_NONE;
3923:   END_HANDLE_TH_ERRORS
3924: }
3925: 
3926: // is_complex
3927: static PyObject * THPVariable_is_complex(PyObject* self_, PyObject* args, PyObject* kwargs)
3928: {
3929:   HANDLE_TH_ERRORS
3930:   static PythonArgParser parser({
3931:     "is_complex(Tensor input)",
3932:   }, /*traceable=*/false);
3933: 
3934:   ParsedArgs<1> parsed_args;
3935:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3936:   if(_r.has_torch_function()) {
3937:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3938:   }
3939:   // aten::is_complex(Tensor self) -> bool
3940: 
3941:   auto dispatch_is_complex = [](const at::Tensor & self) -> bool {
3942:     pybind11::gil_scoped_release no_gil;
3943:     return self.is_complex();
3944:   };
3945:   return wrap(dispatch_is_complex(_r.tensor(0)));
3946:   Py_RETURN_NONE;
3947:   END_HANDLE_TH_ERRORS
3948: }
3949: 
3950: // isreal
3951: static PyObject * THPVariable_isreal(PyObject* self_, PyObject* args, PyObject* kwargs)
3952: {
3953:   HANDLE_TH_ERRORS
3954:   static PythonArgParser parser({
3955:     "isreal(Tensor input)",
3956:   }, /*traceable=*/true);
3957: 
3958:   ParsedArgs<1> parsed_args;
3959:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3960:   if(_r.has_torch_function()) {
3961:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3962:   }
3963:   // aten::isreal(Tensor self) -> Tensor
3964: 
3965:   auto dispatch_isreal = [](const at::Tensor & self) -> at::Tensor {
3966:     pybind11::gil_scoped_release no_gil;
3967:     return self.isreal();
3968:   };
3969:   return wrap(dispatch_isreal(_r.tensor(0)));
3970:   Py_RETURN_NONE;
3971:   END_HANDLE_TH_ERRORS
3972: }
3973: 
3974: // kron
3975: static PyObject * THPVariable_kron(PyObject* self_, PyObject* args, PyObject* kwargs)
3976: {
3977:   HANDLE_TH_ERRORS
3978:   static PythonArgParser parser({
3979:     "kron(Tensor input, Tensor other, *, Tensor out=None)",
3980:   }, /*traceable=*/true);
3981: 
3982:   ParsedArgs<3> parsed_args;
3983:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
3984:   if(_r.has_torch_function()) {
3985:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
3986:   }
3987:   if (_r.isNone(2)) {
3988:     // aten::kron(Tensor self, Tensor other) -> Tensor
3989: 
3990:     auto dispatch_kron = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3991:       pybind11::gil_scoped_release no_gil;
3992:       return self.kron(other);
3993:     };
3994:     return wrap(dispatch_kron(_r.tensor(0), _r.tensor(1)));
3995:   } else {
3996:     // aten::kron.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
3997: 
3998:     auto dispatch_kron_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3999:       pybind11::gil_scoped_release no_gil;
4000:       return at::kron_out(out, self, other);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_is_complex`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_is_complex`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4001-4080

```cpp
4001:     };
4002:     return wrap(dispatch_kron_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4003:   }
4004:   Py_RETURN_NONE;
4005:   END_HANDLE_TH_ERRORS
4006: }
4007: 
4008: // native_layer_norm
4009: static PyObject * THPVariable_native_layer_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
4010: {
4011:   HANDLE_TH_ERRORS
4012:   static PythonArgParser parser({
4013:     "native_layer_norm(Tensor input, SymIntArrayRef normalized_shape, Tensor? weight, Tensor? bias, double eps)",
4014:   }, /*traceable=*/true);
4015: 
4016:   ParsedArgs<5> parsed_args;
4017:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4018:   if(_r.has_torch_function()) {
4019:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4020:   }
4021:   // aten::native_layer_norm(Tensor input, SymInt[] normalized_shape, Tensor? weight, Tensor? bias, float eps) -> (Tensor, Tensor, Tensor)
4022: 
4023:   auto dispatch_native_layer_norm = [](const at::Tensor & input, c10::SymIntArrayRef normalized_shape, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, double eps) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
4024:     pybind11::gil_scoped_release no_gil;
4025:     return at::native_layer_norm_symint(input, normalized_shape, weight, bias, eps);
4026:   };
4027:   return wrap(dispatch_native_layer_norm(_r.tensor(0), _r.symintlist(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.toDouble(4)));
4028:   Py_RETURN_NONE;
4029:   END_HANDLE_TH_ERRORS
4030: }
4031: 
4032: // _cslt_compress
4033: static PyObject * THPVariable__cslt_compress(PyObject* self_, PyObject* args, PyObject* kwargs)
4034: {
4035:   HANDLE_TH_ERRORS
4036:   static PythonArgParser parser({
4037:     "_cslt_compress(Tensor input)",
4038:   }, /*traceable=*/true);
4039: 
4040:   ParsedArgs<1> parsed_args;
4041:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4042:   if(_r.has_torch_function()) {
4043:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4044:   }
4045:   // aten::_cslt_compress(Tensor input) -> Tensor
4046: 
4047:   auto dispatch__cslt_compress = [](const at::Tensor & input) -> at::Tensor {
4048:     pybind11::gil_scoped_release no_gil;
4049:     return at::_cslt_compress(input);
4050:   };
4051:   return wrap(dispatch__cslt_compress(_r.tensor(0)));
4052:   Py_RETURN_NONE;
4053:   END_HANDLE_TH_ERRORS
4054: }
4055: 
4056: // _sparse_semi_structured_tile
4057: static PyObject * THPVariable__sparse_semi_structured_tile(PyObject* self_, PyObject* args, PyObject* kwargs)
4058: {
4059:   HANDLE_TH_ERRORS
4060:   static PythonArgParser parser({
4061:     "_sparse_semi_structured_tile(Tensor input, c10::string_view algorithm=\"\", bool use_cutlass=True)",
4062:   }, /*traceable=*/true);
4063: 
4064:   ParsedArgs<3> parsed_args;
4065:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4066:   if(_r.has_torch_function()) {
4067:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4068:   }
4069:   // aten::_sparse_semi_structured_tile(Tensor input, str algorithm="", bool use_cutlass=True) -> (Tensor, Tensor, Tensor, Tensor, Tensor)
4070: 
4071:   auto dispatch__sparse_semi_structured_tile = [](const at::Tensor & input, c10::string_view algorithm, bool use_cutlass) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
4072:     pybind11::gil_scoped_release no_gil;
4073:     return at::_sparse_semi_structured_tile(input, algorithm, use_cutlass);
4074:   };
4075:   return wrap(dispatch__sparse_semi_structured_tile(_r.tensor(0), _r.stringView(1), _r.toBool(2)));
4076:   Py_RETURN_NONE;
4077:   END_HANDLE_TH_ERRORS
4078: }
4079: 
4080: // _sparse_semi_structured_linear
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_native_layer_norm`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_native_layer_norm`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4081-4160

```cpp
4081: static PyObject * THPVariable__sparse_semi_structured_linear(PyObject* self_, PyObject* args, PyObject* kwargs)
4082: {
4083:   HANDLE_TH_ERRORS
4084:   static PythonArgParser parser({
4085:     "_sparse_semi_structured_linear(Tensor input, Tensor weight, Tensor meta, *, Tensor? bias=None, c10::string_view? activation=None, ScalarType? out_dtype=None)",
4086:   }, /*traceable=*/true);
4087: 
4088:   ParsedArgs<6> parsed_args;
4089:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4090:   if(_r.has_torch_function()) {
4091:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4092:   }
4093:   // aten::_sparse_semi_structured_linear(Tensor input, Tensor weight, Tensor meta, *, Tensor? bias=None, str? activation=None, ScalarType? out_dtype=None) -> Tensor
4094: 
4095:   auto dispatch__sparse_semi_structured_linear = [](const at::Tensor & input, const at::Tensor & weight, const at::Tensor & meta, const ::std::optional<at::Tensor> & bias, ::std::optional<c10::string_view> activation, ::std::optional<at::ScalarType> out_dtype) -> at::Tensor {
4096:     pybind11::gil_scoped_release no_gil;
4097:     return at::_sparse_semi_structured_linear(input, weight, meta, bias, activation, out_dtype);
4098:   };
4099:   return wrap(dispatch__sparse_semi_structured_linear(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.stringViewOptional(4), _r.scalartypeOptional(5)));
4100:   Py_RETURN_NONE;
4101:   END_HANDLE_TH_ERRORS
4102: }
4103: 
4104: \
4105: // fbgemm_linear_fp16_weight
4106: static PyObject * THPVariable_fbgemm_linear_fp16_weight(PyObject* self_, PyObject* args, PyObject* kwargs)
4107: {
4108:   HANDLE_TH_ERRORS
4109:   static PythonArgParser parser({
4110:     "fbgemm_linear_fp16_weight(Tensor input, Tensor packed_weight, Tensor bias)",
4111:     "fbgemm_linear_fp16_weight(Tensor input, Tensor packed_weight, Tensor bias, Tensor output)",
4112:   }, /*traceable=*/true);
4113: 
4114:   ParsedArgs<4> parsed_args;
4115:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4116:   if(_r.has_torch_function()) {
4117:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4118:   }
4119:   switch (_r.idx) {
4120:     case 0: {
4121:       // aten::fbgemm_linear_fp16_weight(Tensor input, Tensor packed_weight, Tensor bias) -> Tensor
4122: 
4123:       auto dispatch_fbgemm_linear_fp16_weight = [](const at::Tensor & input, const at::Tensor & packed_weight, const at::Tensor & bias) -> at::Tensor {
4124:         pybind11::gil_scoped_release no_gil;
4125:         return at::fbgemm_linear_fp16_weight(input, packed_weight, bias);
4126:       };
4127:       return wrap(dispatch_fbgemm_linear_fp16_weight(_r.tensor(0), _r.tensor(1), _r.tensor(2)));
4128:     }
4129:     case 1: {
4130:       // aten::fbgemm_linear_fp16_weight.out(Tensor input, Tensor packed_weight, Tensor bias, Tensor(a!) output) -> Tensor
4131: 
4132:       auto dispatch_fbgemm_linear_fp16_weight = [](const at::Tensor & input, const at::Tensor & packed_weight, const at::Tensor & bias, at::Tensor output) -> at::Tensor {
4133:         pybind11::gil_scoped_release no_gil;
4134:         return at::fbgemm_linear_fp16_weight(input, packed_weight, bias, output);
4135:       };
4136:       return wrap(dispatch_fbgemm_linear_fp16_weight(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3)));
4137:     }
4138:   }
4139:   Py_RETURN_NONE;
4140:   END_HANDLE_TH_ERRORS
4141: }
4142: 
4143: // ldexp
4144: static PyObject * THPVariable_ldexp(PyObject* self_, PyObject* args, PyObject* kwargs)
4145: {
4146:   HANDLE_TH_ERRORS
4147:   static PythonArgParser parser({
4148:     "ldexp(Tensor input, Tensor other, *, Tensor out=None)",
4149:   }, /*traceable=*/true);
4150: 
4151:   ParsedArgs<3> parsed_args;
4152:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4153:   if(_r.has_torch_function()) {
4154:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4155:   }
4156:   if (_r.isNone(2)) {
4157:     // aten::ldexp.Tensor(Tensor self, Tensor other) -> Tensor
4158: 
4159:     auto dispatch_ldexp = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4160:       pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable__sparse_semi_structured_linear`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__sparse_semi_structured_linear`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4161-4240

```cpp
4161:       return self.ldexp(other);
4162:     };
4163:     return wrap(dispatch_ldexp(_r.tensor(0), _r.tensor(1)));
4164:   } else {
4165:     // aten::ldexp.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
4166: 
4167:     auto dispatch_ldexp_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4168:       pybind11::gil_scoped_release no_gil;
4169:       return at::ldexp_out(out, self, other);
4170:     };
4171:     return wrap(dispatch_ldexp_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4172:   }
4173:   Py_RETURN_NONE;
4174:   END_HANDLE_TH_ERRORS
4175: }
4176: 
4177: // ldexp_
4178: static PyObject * THPVariable_ldexp_(PyObject* self_, PyObject* args, PyObject* kwargs)
4179: {
4180:   HANDLE_TH_ERRORS
4181:   static PythonArgParser parser({
4182:     "ldexp_(Tensor input, Tensor other)",
4183:   }, /*traceable=*/true);
4184: 
4185:   ParsedArgs<2> parsed_args;
4186:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4187:   if(_r.has_torch_function()) {
4188:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4189:   }
4190:   // aten::ldexp_(Tensor(a!) self, Tensor other) -> Tensor(a!)
4191: 
4192:   auto dispatch_ldexp_ = [](at::Tensor self, const at::Tensor & other) -> at::Tensor {
4193:     pybind11::gil_scoped_release no_gil;
4194:     return self.ldexp_(other);
4195:   };
4196:   return wrap(dispatch_ldexp_(_r.tensor(0), _r.tensor(1)));
4197:   Py_RETURN_NONE;
4198:   END_HANDLE_TH_ERRORS
4199: }
4200: 
4201: // log10
4202: static PyObject * THPVariable_log10(PyObject* self_, PyObject* args, PyObject* kwargs)
4203: {
4204:   HANDLE_TH_ERRORS
4205:   static PythonArgParser parser({
4206:     "log10(Tensor input, *, Tensor out=None)",
4207:   }, /*traceable=*/true);
4208: 
4209:   ParsedArgs<2> parsed_args;
4210:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4211:   if(_r.has_torch_function()) {
4212:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4213:   }
4214:   if (_r.isNone(1)) {
4215:     // aten::log10(Tensor self) -> Tensor
4216: 
4217:     auto dispatch_log10 = [](const at::Tensor & self) -> at::Tensor {
4218:       pybind11::gil_scoped_release no_gil;
4219:       return self.log10();
4220:     };
4221:     return wrap(dispatch_log10(_r.tensor(0)));
4222:   } else {
4223:     // aten::log10.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
4224: 
4225:     auto dispatch_log10_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
4226:       pybind11::gil_scoped_release no_gil;
4227:       return at::log10_out(out, self);
4228:     };
4229:     return wrap(dispatch_log10_out(_r.tensor(1), _r.tensor(0)));
4230:   }
4231:   Py_RETURN_NONE;
4232:   END_HANDLE_TH_ERRORS
4233: }
4234: 
4235: // log10_
4236: static PyObject * THPVariable_log10_(PyObject* self_, PyObject* args, PyObject* kwargs)
4237: {
4238:   HANDLE_TH_ERRORS
4239:   static PythonArgParser parser({
4240:     "log10_(Tensor input)",
```

- EN: The main execution path in this span is carried by `wrap`, `ldexp_out`, `THPVariable_ldexp_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `ldexp_out`, `THPVariable_ldexp_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4241-4320

```cpp
4241:   }, /*traceable=*/true);
4242: 
4243:   ParsedArgs<1> parsed_args;
4244:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4245:   if(_r.has_torch_function()) {
4246:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4247:   }
4248:   // aten::log10_(Tensor(a!) self) -> Tensor(a!)
4249: 
4250:   auto dispatch_log10_ = [](at::Tensor self) -> at::Tensor {
4251:     pybind11::gil_scoped_release no_gil;
4252:     return self.log10_();
4253:   };
4254:   return wrap(dispatch_log10_(_r.tensor(0)));
4255:   Py_RETURN_NONE;
4256:   END_HANDLE_TH_ERRORS
4257: }
4258: 
4259: // logaddexp2
4260: static PyObject * THPVariable_logaddexp2(PyObject* self_, PyObject* args, PyObject* kwargs)
4261: {
4262:   HANDLE_TH_ERRORS
4263:   static PythonArgParser parser({
4264:     "logaddexp2(Tensor input, Tensor other, *, Tensor out=None)",
4265:   }, /*traceable=*/true);
4266: 
4267:   ParsedArgs<3> parsed_args;
4268:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4269:   if(_r.has_torch_function()) {
4270:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4271:   }
4272:   if (_r.isNone(2)) {
4273:     // aten::logaddexp2(Tensor self, Tensor other) -> Tensor
4274: 
4275:     auto dispatch_logaddexp2 = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4276:       pybind11::gil_scoped_release no_gil;
4277:       return self.logaddexp2(other);
4278:     };
4279:     return wrap(dispatch_logaddexp2(_r.tensor(0), _r.tensor(1)));
4280:   } else {
4281:     // aten::logaddexp2.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
4282: 
4283:     auto dispatch_logaddexp2_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4284:       pybind11::gil_scoped_release no_gil;
4285:       return at::logaddexp2_out(out, self, other);
4286:     };
4287:     return wrap(dispatch_logaddexp2_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4288:   }
4289:   Py_RETURN_NONE;
4290:   END_HANDLE_TH_ERRORS
4291: }
4292: 
4293: // _log_softmax_backward_data
4294: static PyObject * THPVariable__log_softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs)
4295: {
4296:   HANDLE_TH_ERRORS
4297:   static PythonArgParser parser({
4298:     "_log_softmax_backward_data(Tensor grad_output, Tensor output, int64_t dim, ScalarType input_dtype, *, Tensor out=None)",
4299:   }, /*traceable=*/true);
4300: 
4301:   ParsedArgs<5> parsed_args;
4302:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4303:   if(_r.has_torch_function()) {
4304:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4305:   }
4306:   if (_r.isNone(4)) {
4307:     // aten::_log_softmax_backward_data(Tensor grad_output, Tensor output, int dim, ScalarType input_dtype) -> Tensor
4308: 
4309:     auto dispatch__log_softmax_backward_data = [](const at::Tensor & grad_output, const at::Tensor & output, int64_t dim, at::ScalarType input_dtype) -> at::Tensor {
4310:       pybind11::gil_scoped_release no_gil;
4311:       return at::_log_softmax_backward_data(grad_output, output, dim, input_dtype);
4312:     };
4313:     return wrap(dispatch__log_softmax_backward_data(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.scalartype(3)));
4314:   } else {
4315:     // aten::_log_softmax_backward_data.out(Tensor grad_output, Tensor output, int dim, ScalarType input_dtype, *, Tensor(a!) out) -> Tensor(a!)
4316: 
4317:     auto dispatch__log_softmax_backward_data_out = [](at::Tensor out, const at::Tensor & grad_output, const at::Tensor & output, int64_t dim, at::ScalarType input_dtype) -> at::Tensor {
4318:       pybind11::gil_scoped_release no_gil;
4319:       return at::_log_softmax_backward_data_out(out, grad_output, output, dim, input_dtype);
4320:     };
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `log10_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `log10_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4321-4400

```cpp
4321:     return wrap(dispatch__log_softmax_backward_data_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.scalartype(3)));
4322:   }
4323:   Py_RETURN_NONE;
4324:   END_HANDLE_TH_ERRORS
4325: }
4326: 
4327: \
4328: // logcumsumexp
4329: static PyObject * THPVariable_logcumsumexp(PyObject* self_, PyObject* args, PyObject* kwargs)
4330: {
4331:   HANDLE_TH_ERRORS
4332:   static PythonArgParser parser({
4333:     "logcumsumexp(Tensor input, int64_t dim, *, Tensor out=None)",
4334:     "logcumsumexp(Tensor input, Dimname dim, *, Tensor out=None)",
4335:   }, /*traceable=*/true);
4336: 
4337:   ParsedArgs<3> parsed_args;
4338:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4339:   if(_r.has_torch_function()) {
4340:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4341:   }
4342:   switch (_r.idx) {
4343:     case 0: {
4344:       if (_r.isNone(2)) {
4345:         // aten::logcumsumexp(Tensor self, int dim) -> Tensor
4346: 
4347:         auto dispatch_logcumsumexp = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
4348:           pybind11::gil_scoped_release no_gil;
4349:           return self.logcumsumexp(dim);
4350:         };
4351:         return wrap(dispatch_logcumsumexp(_r.tensor(0), _r.toInt64(1)));
4352:       } else {
4353:         // aten::logcumsumexp.out(Tensor self, int dim, *, Tensor(a!) out) -> Tensor(a!)
4354: 
4355:         auto dispatch_logcumsumexp_out = [](at::Tensor out, const at::Tensor & self, int64_t dim) -> at::Tensor {
4356:           pybind11::gil_scoped_release no_gil;
4357:           return at::logcumsumexp_out(out, self, dim);
4358:         };
4359:         return wrap(dispatch_logcumsumexp_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
4360:       }
4361:     }
4362:     case 1: {
4363:       if (_r.isNone(2)) {
4364:         // aten::logcumsumexp.dimname(Tensor self, Dimname dim) -> Tensor
4365: 
4366:         auto dispatch_logcumsumexp = [](const at::Tensor & self, at::Dimname dim) -> at::Tensor {
4367:           pybind11::gil_scoped_release no_gil;
4368:           return self.logcumsumexp(dim);
4369:         };
4370:         return wrap(dispatch_logcumsumexp(_r.tensor(0), _r.dimname(1)));
4371:       } else {
4372:         // aten::logcumsumexp.dimname_out(Tensor self, Dimname dim, *, Tensor(a!) out) -> Tensor(a!)
4373: 
4374:         auto dispatch_logcumsumexp_out = [](at::Tensor out, const at::Tensor & self, at::Dimname dim) -> at::Tensor {
4375:           pybind11::gil_scoped_release no_gil;
4376:           return at::logcumsumexp_out(out, self, dim);
4377:         };
4378:         return wrap(dispatch_logcumsumexp_out(_r.tensor(2), _r.tensor(0), _r.dimname(1)));
4379:       }
4380:     }
4381:   }
4382:   Py_RETURN_NONE;
4383:   END_HANDLE_TH_ERRORS
4384: }
4385: 
4386: // margin_ranking_loss
4387: static PyObject * THPVariable_margin_ranking_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
4388: {
4389:   HANDLE_TH_ERRORS
4390:   static PythonArgParser parser({
4391:     "margin_ranking_loss(Tensor input1, Tensor input2, Tensor target, double margin=0.0, int64_t reduction=at::Reduction::Mean)",
4392:   }, /*traceable=*/true);
4393: 
4394:   ParsedArgs<5> parsed_args;
4395:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4396:   if(_r.has_torch_function()) {
4397:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4398:   }
4399:   // aten::margin_ranking_loss(Tensor input1, Tensor input2, Tensor target, float margin=0.0, int reduction=Mean) -> Tensor
4400: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_logcumsumexp`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_logcumsumexp`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4401-4480

```cpp
4401:   auto dispatch_margin_ranking_loss = [](const at::Tensor & input1, const at::Tensor & input2, const at::Tensor & target, double margin, int64_t reduction) -> at::Tensor {
4402:     pybind11::gil_scoped_release no_gil;
4403:     return at::margin_ranking_loss(input1, input2, target, margin, reduction);
4404:   };
4405:   return wrap(dispatch_margin_ranking_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3), _r.toInt64(4)));
4406:   Py_RETURN_NONE;
4407:   END_HANDLE_TH_ERRORS
4408: }
4409: 
4410: // matrix_exp
4411: static PyObject * THPVariable_matrix_exp(PyObject* self_, PyObject* args, PyObject* kwargs)
4412: {
4413:   HANDLE_TH_ERRORS
4414:   static PythonArgParser parser({
4415:     "matrix_exp(Tensor input)",
4416:   }, /*traceable=*/true);
4417: 
4418:   ParsedArgs<1> parsed_args;
4419:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4420:   if(_r.has_torch_function()) {
4421:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4422:   }
4423:   // aten::matrix_exp(Tensor self) -> Tensor
4424: 
4425:   auto dispatch_matrix_exp = [](const at::Tensor & self) -> at::Tensor {
4426:     pybind11::gil_scoped_release no_gil;
4427:     return self.matrix_exp();
4428:   };
4429:   return wrap(dispatch_matrix_exp(_r.tensor(0)));
4430:   Py_RETURN_NONE;
4431:   END_HANDLE_TH_ERRORS
4432: }
4433: 
4434: // amax
4435: static PyObject * THPVariable_amax(PyObject* self_, PyObject* args, PyObject* kwargs)
4436: {
4437:   HANDLE_TH_ERRORS
4438:   static PythonArgParser parser({
4439:     "amax(Tensor input, IntArrayRef[1] dim=None, bool keepdim=False, *, Tensor out=None)",
4440:   }, /*traceable=*/true);
4441: 
4442:   ParsedArgs<4> parsed_args;
4443:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4444:   if(_r.has_torch_function()) {
4445:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4446:   }
4447:   if (_r.isNone(3)) {
4448:     // aten::amax(Tensor self, int[1] dim=[], bool keepdim=False) -> Tensor
4449: 
4450:     auto dispatch_amax = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
4451:       pybind11::gil_scoped_release no_gil;
4452:       return self.amax(dim, keepdim);
4453:     };
4454:     return wrap(dispatch_amax(_r.tensor(0), _r.intlist(1), _r.toBool(2)));
4455:   } else {
4456:     // aten::amax.out(Tensor self, int[1] dim=[], bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
4457: 
4458:     auto dispatch_amax_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
4459:       pybind11::gil_scoped_release no_gil;
4460:       return at::amax_out(out, self, dim, keepdim);
4461:     };
4462:     return wrap(dispatch_amax_out(_r.tensor(3), _r.tensor(0), _r.intlist(1), _r.toBool(2)));
4463:   }
4464:   Py_RETURN_NONE;
4465:   END_HANDLE_TH_ERRORS
4466: }
4467: 
4468: // max_pool1d
4469: static PyObject * THPVariable_max_pool1d(PyObject* self_, PyObject* args, PyObject* kwargs)
4470: {
4471:   HANDLE_TH_ERRORS
4472:   static PythonArgParser parser({
4473:     "max_pool1d(Tensor input, IntArrayRef[1] kernel_size, IntArrayRef[1] stride=None, IntArrayRef[1] padding=0, IntArrayRef[1] dilation=1, bool ceil_mode=False)",
4474:   }, /*traceable=*/true);
4475: 
4476:   ParsedArgs<6> parsed_args;
4477:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4478:   if(_r.has_torch_function()) {
4479:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4480:   }
```

- EN: The main execution path in this span is carried by `margin_ranking_loss`, `wrap`, `THPVariable_matrix_exp`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `margin_ranking_loss`, `wrap`, `THPVariable_matrix_exp` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4481-4560

```cpp
4481:   // aten::max_pool1d(Tensor self, int[1] kernel_size, int[1] stride=[], int[1] padding=0, int[1] dilation=1, bool ceil_mode=False) -> Tensor
4482: 
4483:   auto dispatch_max_pool1d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> at::Tensor {
4484:     pybind11::gil_scoped_release no_gil;
4485:     return at::max_pool1d(self, kernel_size, stride, padding, dilation, ceil_mode);
4486:   };
4487:   return wrap(dispatch_max_pool1d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
4488:   Py_RETURN_NONE;
4489:   END_HANDLE_TH_ERRORS
4490: }
4491: 
4492: // mkldnn_max_pool3d
4493: static PyObject * THPVariable_mkldnn_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
4494: {
4495:   HANDLE_TH_ERRORS
4496:   static PythonArgParser parser({
4497:     "mkldnn_max_pool3d(Tensor input, IntArrayRef[3] kernel_size, IntArrayRef[3] stride=None, IntArrayRef[3] padding=0, IntArrayRef[3] dilation=1, bool ceil_mode=False)",
4498:   }, /*traceable=*/true);
4499: 
4500:   ParsedArgs<6> parsed_args;
4501:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4502:   if(_r.has_torch_function()) {
4503:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4504:   }
4505:   // aten::mkldnn_max_pool3d(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, int[3] dilation=1, bool ceil_mode=False) -> Tensor
4506: 
4507:   auto dispatch_mkldnn_max_pool3d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> at::Tensor {
4508:     pybind11::gil_scoped_release no_gil;
4509:     return at::mkldnn_max_pool3d(self, kernel_size, stride, padding, dilation, ceil_mode);
4510:   };
4511:   return wrap(dispatch_mkldnn_max_pool3d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
4512:   Py_RETURN_NONE;
4513:   END_HANDLE_TH_ERRORS
4514: }
4515: 
4516: // quantized_max_pool3d
4517: static PyObject * THPVariable_quantized_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
4518: {
4519:   HANDLE_TH_ERRORS
4520:   static PythonArgParser parser({
4521:     "quantized_max_pool3d(Tensor input, IntArrayRef[3] kernel_size, IntArrayRef[3] stride=None, IntArrayRef[3] padding=0, IntArrayRef[3] dilation=1, bool ceil_mode=False)",
4522:   }, /*traceable=*/true);
4523: 
4524:   ParsedArgs<6> parsed_args;
4525:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4526:   if(_r.has_torch_function()) {
4527:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4528:   }
4529:   // aten::quantized_max_pool3d(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, int[3] dilation=1, bool ceil_mode=False) -> Tensor
4530: 
4531:   auto dispatch_quantized_max_pool3d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> at::Tensor {
4532:     pybind11::gil_scoped_release no_gil;
4533:     return at::quantized_max_pool3d(self, kernel_size, stride, padding, dilation, ceil_mode);
4534:   };
4535:   return wrap(dispatch_quantized_max_pool3d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
4536:   Py_RETURN_NONE;
4537:   END_HANDLE_TH_ERRORS
4538: }
4539: 
4540: // max_pool3d
4541: static PyObject * THPVariable_max_pool3d(PyObject* self_, PyObject* args, PyObject* kwargs)
4542: {
4543:   HANDLE_TH_ERRORS
4544:   static PythonArgParser parser({
4545:     "max_pool3d(Tensor input, IntArrayRef[3] kernel_size, IntArrayRef[3] stride=None, IntArrayRef[3] padding=0, IntArrayRef[3] dilation=1, bool ceil_mode=False)",
4546:   }, /*traceable=*/true);
4547: 
4548:   ParsedArgs<6> parsed_args;
4549:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4550:   if(_r.has_torch_function()) {
4551:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4552:   }
4553:   // aten::max_pool3d(Tensor self, int[3] kernel_size, int[3] stride=[], int[3] padding=0, int[3] dilation=1, bool ceil_mode=False) -> Tensor
4554: 
4555:   auto dispatch_max_pool3d = [](const at::Tensor & self, at::IntArrayRef kernel_size, at::IntArrayRef stride, at::IntArrayRef padding, at::IntArrayRef dilation, bool ceil_mode) -> at::Tensor {
4556:     pybind11::gil_scoped_release no_gil;
4557:     return at::max_pool3d(self, kernel_size, stride, padding, dilation, ceil_mode);
4558:   };
4559:   return wrap(dispatch_max_pool3d(_r.tensor(0), _r.intlist(1), _r.intlist(2), _r.intlist(3), _r.intlist(4), _r.toBool(5)));
4560:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `max_pool1d`, `wrap`, `THPVariable_mkldnn_max_pool3d`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `max_pool1d`, `wrap`, `THPVariable_mkldnn_max_pool3d` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4561-4640

```cpp
4561:   END_HANDLE_TH_ERRORS
4562: }
4563: 
4564: \
4565: // median
4566: static PyObject * THPVariable_median(PyObject* self_, PyObject* args, PyObject* kwargs)
4567: {
4568:   HANDLE_TH_ERRORS
4569:   static PyTypeObject* NamedTuple = generated::get_median_structseq();
4570:   static PyTypeObject* NamedTuple1 = generated::get_median_out_structseq();
4571:   static PythonArgParser parser({
4572:     "median(Tensor input)",
4573:     "median(Tensor input, int64_t dim, bool keepdim=False, *, TensorList[2] out=None)",
4574:     "median(Tensor input, Dimname dim, bool keepdim=False, *, TensorList[2] out=None)",
4575:   }, /*traceable=*/true);
4576: 
4577:   ParsedArgs<4> parsed_args;
4578:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4579:   if(_r.has_torch_function()) {
4580:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4581:   }
4582:   switch (_r.idx) {
4583:     case 0: {
4584:       // aten::median(Tensor self) -> Tensor
4585: 
4586:       auto dispatch_median = [](const at::Tensor & self) -> at::Tensor {
4587:         pybind11::gil_scoped_release no_gil;
4588:         return self.median();
4589:       };
4590:       return wrap(dispatch_median(_r.tensor(0)));
4591:     }
4592:     case 1: {
4593:       if (_r.isNone(3)) {
4594:         // aten::median.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
4595: 
4596:         auto dispatch_median = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
4597:           pybind11::gil_scoped_release no_gil;
4598:           return self.median(dim, keepdim);
4599:         };
4600:         return wrap(NamedTuple, dispatch_median(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
4601:       } else {
4602:         // aten::median.dim_values(Tensor self, int dim, bool keepdim=False, *, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
4603:         auto out = _r.tensorlist_n<2>(3);
4604:         auto dispatch_median_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
4605:           pybind11::gil_scoped_release no_gil;
4606:           return at::median_out(values, indices, self, dim, keepdim);
4607:         };
4608:         return wrap(NamedTuple1, dispatch_median_out(out[0], out[1], _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
4609:       }
4610:     }
4611:     case 2: {
4612:       if (_r.isNone(3)) {
4613:         // aten::median.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
4614: 
4615:         auto dispatch_median = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
4616:           pybind11::gil_scoped_release no_gil;
4617:           return self.median(dim, keepdim);
4618:         };
4619:         return wrap(NamedTuple, dispatch_median(_r.tensor(0), _r.dimname(1), _r.toBool(2)));
4620:       } else {
4621:         // aten::median.names_dim_values(Tensor self, Dimname dim, bool keepdim=False, *, Tensor(a!) values, Tensor(b!) indices) -> (Tensor(a!) values, Tensor(b!) indices)
4622:         auto out = _r.tensorlist_n<2>(3);
4623:         auto dispatch_median_out = [](at::Tensor & values, at::Tensor & indices, const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
4624:           pybind11::gil_scoped_release no_gil;
4625:           return at::median_out(values, indices, self, dim, keepdim);
4626:         };
4627:         return wrap(NamedTuple1, dispatch_median_out(out[0], out[1], _r.tensor(0), _r.dimname(1), _r.toBool(2)));
4628:       }
4629:     }
4630:   }
4631:   Py_RETURN_NONE;
4632:   END_HANDLE_TH_ERRORS
4633: }
4634: 
4635: // _mps_convolution
4636: static PyObject * THPVariable__mps_convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
4637: {
4638:   HANDLE_TH_ERRORS
4639:   static PythonArgParser parser({
4640:     "_mps_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups)",
```

- EN: The main execution path in this span is carried by `THPVariable_median`, `get_median_structseq`, `get_median_out_structseq`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_median`, `get_median_structseq`, `get_median_out_structseq` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4641-4720

```cpp
4641:   }, /*traceable=*/true);
4642: 
4643:   ParsedArgs<7> parsed_args;
4644:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4645:   if(_r.has_torch_function()) {
4646:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4647:   }
4648:   // aten::_mps_convolution(Tensor self, Tensor weight, Tensor? bias, SymInt[] padding, SymInt[] stride, SymInt[] dilation, SymInt groups) -> Tensor
4649: 
4650:   auto dispatch__mps_convolution = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
4651:     pybind11::gil_scoped_release no_gil;
4652:     return at::_mps_convolution_symint(self, weight, bias, padding, stride, dilation, groups);
4653:   };
4654:   return wrap(dispatch__mps_convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6)));
4655:   Py_RETURN_NONE;
4656:   END_HANDLE_TH_ERRORS
4657: }
4658: 
4659: // miopen_convolution
4660: static PyObject * THPVariable_miopen_convolution(PyObject* self_, PyObject* args, PyObject* kwargs)
4661: {
4662:   HANDLE_TH_ERRORS
4663:   static PythonArgParser parser({
4664:     "miopen_convolution(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups, bool benchmark, bool deterministic)",
4665:   }, /*traceable=*/true);
4666: 
4667:   ParsedArgs<9> parsed_args;
4668:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4669:   if(_r.has_torch_function()) {
4670:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4671:   }
4672:   // aten::miopen_convolution(Tensor self, Tensor weight, Tensor? bias, SymInt[] padding, SymInt[] stride, SymInt[] dilation, SymInt groups, bool benchmark, bool deterministic) -> Tensor
4673: 
4674:   auto dispatch_miopen_convolution = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups, bool benchmark, bool deterministic) -> at::Tensor {
4675:     pybind11::gil_scoped_release no_gil;
4676:     return at::miopen_convolution_symint(self, weight, bias, padding, stride, dilation, groups, benchmark, deterministic);
4677:   };
4678:   return wrap(dispatch_miopen_convolution(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.toSymInt(6), _r.toBool(7), _r.toBool(8)));
4679:   Py_RETURN_NONE;
4680:   END_HANDLE_TH_ERRORS
4681: }
4682: 
4683: // miopen_convolution_transpose
4684: static PyObject * THPVariable_miopen_convolution_transpose(PyObject* self_, PyObject* args, PyObject* kwargs)
4685: {
4686:   HANDLE_TH_ERRORS
4687:   static PythonArgParser parser({
4688:     "miopen_convolution_transpose(Tensor input, Tensor weight, Tensor? bias, SymIntArrayRef padding, SymIntArrayRef output_padding, SymIntArrayRef stride, SymIntArrayRef dilation, SymInt groups, bool benchmark, bool deterministic)",
4689:   }, /*traceable=*/true);
4690: 
4691:   ParsedArgs<10> parsed_args;
4692:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4693:   if(_r.has_torch_function()) {
4694:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4695:   }
4696:   // aten::miopen_convolution_transpose(Tensor self, Tensor weight, Tensor? bias, SymInt[] padding, SymInt[] output_padding, SymInt[] stride, SymInt[] dilation, SymInt groups, bool benchmark, bool deterministic) -> Tensor
4697: 
4698:   auto dispatch_miopen_convolution_transpose = [](const at::Tensor & self, const at::Tensor & weight, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef padding, c10::SymIntArrayRef output_padding, c10::SymIntArrayRef stride, c10::SymIntArrayRef dilation, c10::SymInt groups, bool benchmark, bool deterministic) -> at::Tensor {
4699:     pybind11::gil_scoped_release no_gil;
4700:     return at::miopen_convolution_transpose_symint(self, weight, bias, padding, output_padding, stride, dilation, groups, benchmark, deterministic);
4701:   };
4702:   return wrap(dispatch_miopen_convolution_transpose(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.symintlist(3), _r.symintlist(4), _r.symintlist(5), _r.symintlist(6), _r.toSymInt(7), _r.toBool(8), _r.toBool(9)));
4703:   Py_RETURN_NONE;
4704:   END_HANDLE_TH_ERRORS
4705: }
4706: 
4707: // miopen_convolution_add_relu
4708: static PyObject * THPVariable_miopen_convolution_add_relu(PyObject* self_, PyObject* args, PyObject* kwargs)
4709: {
4710:   HANDLE_TH_ERRORS
4711:   static PythonArgParser parser({
4712:     "miopen_convolution_add_relu(Tensor input, Tensor weight, Tensor z, Scalar? alpha, Tensor? bias, SymIntArrayRef stride, SymIntArrayRef padding, SymIntArrayRef dilation, SymInt groups)",
4713:   }, /*traceable=*/true);
4714: 
4715:   ParsedArgs<9> parsed_args;
4716:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4717:   if(_r.has_torch_function()) {
4718:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4719:   }
4720:   // aten::miopen_convolution_add_relu(Tensor self, Tensor weight, Tensor z, Scalar? alpha, Tensor? bias, SymInt[] stride, SymInt[] padding, SymInt[] dilation, SymInt groups) -> Tensor
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_mps_convolution`, `_mps_convolution_symint`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_mps_convolution`, `_mps_convolution_symint` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4721-4800

```cpp
4721: 
4722:   auto dispatch_miopen_convolution_add_relu = [](const at::Tensor & self, const at::Tensor & weight, const at::Tensor & z, const ::std::optional<at::Scalar> & alpha, const ::std::optional<at::Tensor> & bias, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, c10::SymInt groups) -> at::Tensor {
4723:     pybind11::gil_scoped_release no_gil;
4724:     return at::miopen_convolution_add_relu_symint(self, weight, z, alpha, bias, stride, padding, dilation, groups);
4725:   };
4726:   return wrap(dispatch_miopen_convolution_add_relu(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalarOptional(3), _r.optionalTensor(4), _r.symintlist(5), _r.symintlist(6), _r.symintlist(7), _r.toSymInt(8)));
4727:   Py_RETURN_NONE;
4728:   END_HANDLE_TH_ERRORS
4729: }
4730: 
4731: \
4732: // _use_miopen_ctc_loss
4733: static PyObject * THPVariable__use_miopen_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
4734: {
4735:   HANDLE_TH_ERRORS
4736:   static PythonArgParser parser({
4737:     "_use_miopen_ctc_loss(Tensor log_probs, Tensor targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t blank)",
4738:     "_use_miopen_ctc_loss(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int64_t blank)",
4739:   }, /*traceable=*/false);
4740: 
4741:   ParsedArgs<5> parsed_args;
4742:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4743:   if(_r.has_torch_function()) {
4744:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4745:   }
4746:   switch (_r.idx) {
4747:     case 0: {
4748:       // aten::_use_miopen_ctc_loss(Tensor log_probs, Tensor targets, int[] input_lengths, int[] target_lengths, int blank) -> bool
4749: 
4750:       auto dispatch__use_miopen_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, at::IntArrayRef input_lengths, at::IntArrayRef target_lengths, int64_t blank) -> bool {
4751:         pybind11::gil_scoped_release no_gil;
4752:         return at::_use_miopen_ctc_loss(log_probs, targets, input_lengths, target_lengths, blank);
4753:       };
4754:       return wrap(dispatch__use_miopen_ctc_loss(_r.tensor(0), _r.tensor(1), _r.intlist(2), _r.intlist(3), _r.toInt64(4)));
4755:     }
4756:     case 1: {
4757:       // aten::_use_miopen_ctc_loss.Tensor(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int blank) -> bool
4758: 
4759:       auto dispatch__use_miopen_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, const at::Tensor & input_lengths, const at::Tensor & target_lengths, int64_t blank) -> bool {
4760:         pybind11::gil_scoped_release no_gil;
4761:         return at::_use_miopen_ctc_loss(log_probs, targets, input_lengths, target_lengths, blank);
4762:       };
4763:       return wrap(dispatch__use_miopen_ctc_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.toInt64(4)));
4764:     }
4765:   }
4766:   Py_RETURN_NONE;
4767:   END_HANDLE_TH_ERRORS
4768: }
4769: 
4770: \
4771: // miopen_ctc_loss
4772: static PyObject * THPVariable_miopen_ctc_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
4773: {
4774:   HANDLE_TH_ERRORS
4775:   static PythonArgParser parser({
4776:     "miopen_ctc_loss(Tensor log_probs, Tensor targets, IntArrayRef input_lengths, IntArrayRef target_lengths, int64_t blank, bool deterministic, bool zero_infinity)",
4777:     "miopen_ctc_loss(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int64_t blank, bool deterministic, bool zero_infinity)",
4778:   }, /*traceable=*/true);
4779: 
4780:   ParsedArgs<7> parsed_args;
4781:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4782:   if(_r.has_torch_function()) {
4783:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4784:   }
4785:   switch (_r.idx) {
4786:     case 0: {
4787:       // aten::miopen_ctc_loss(Tensor log_probs, Tensor targets, int[] input_lengths, int[] target_lengths, int blank, bool deterministic, bool zero_infinity) -> (Tensor, Tensor)
4788: 
4789:       auto dispatch_miopen_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, at::IntArrayRef input_lengths, at::IntArrayRef target_lengths, int64_t blank, bool deterministic, bool zero_infinity) -> ::std::tuple<at::Tensor,at::Tensor> {
4790:         pybind11::gil_scoped_release no_gil;
4791:         return at::miopen_ctc_loss(log_probs, targets, input_lengths, target_lengths, blank, deterministic, zero_infinity);
4792:       };
4793:       return wrap(dispatch_miopen_ctc_loss(_r.tensor(0), _r.tensor(1), _r.intlist(2), _r.intlist(3), _r.toInt64(4), _r.toBool(5), _r.toBool(6)));
4794:     }
4795:     case 1: {
4796:       // aten::miopen_ctc_loss.Tensor(Tensor log_probs, Tensor targets, Tensor input_lengths, Tensor target_lengths, int blank, bool deterministic, bool zero_infinity) -> (Tensor, Tensor)
4797: 
4798:       auto dispatch_miopen_ctc_loss = [](const at::Tensor & log_probs, const at::Tensor & targets, const at::Tensor & input_lengths, const at::Tensor & target_lengths, int64_t blank, bool deterministic, bool zero_infinity) -> ::std::tuple<at::Tensor,at::Tensor> {
4799:         pybind11::gil_scoped_release no_gil;
4800:         return at::miopen_ctc_loss(log_probs, targets, input_lengths, target_lengths, blank, deterministic, zero_infinity);
```

- EN: The main execution path in this span is carried by `miopen_convolution_add_relu_symint`, `wrap`, `THPVariable__use_miopen_ctc_loss`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `miopen_convolution_add_relu_symint`, `wrap`, `THPVariable__use_miopen_ctc_loss` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4801-4880

```cpp
4801:       };
4802:       return wrap(dispatch_miopen_ctc_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.toInt64(4), _r.toBool(5), _r.toBool(6)));
4803:     }
4804:   }
4805:   Py_RETURN_NONE;
4806:   END_HANDLE_TH_ERRORS
4807: }
4808: 
4809: \
4810: // mm
4811: static PyObject * THPVariable_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
4812: {
4813:   HANDLE_TH_ERRORS
4814:   static PythonArgParser parser({
4815:     "mm(Tensor input, Tensor mat2, *, Tensor out=None)",
4816:     "mm(Tensor input, Tensor mat2, ScalarType out_dtype, *, Tensor out=None)",
4817:   }, /*traceable=*/true);
4818: 
4819:   ParsedArgs<4> parsed_args;
4820:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4821:   if(_r.has_torch_function()) {
4822:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4823:   }
4824:   switch (_r.idx) {
4825:     case 0: {
4826:       if (_r.isNone(2)) {
4827:         // aten::mm(Tensor self, Tensor mat2) -> Tensor
4828: 
4829:         auto dispatch_mm = [](const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
4830:           pybind11::gil_scoped_release no_gil;
4831:           return self.mm(mat2);
4832:         };
4833:         return wrap(dispatch_mm(_r.tensor(0), _r.tensor(1)));
4834:       } else {
4835:         // aten::mm.out(Tensor self, Tensor mat2, *, Tensor(a!) out) -> Tensor(a!)
4836: 
4837:         auto dispatch_mm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
4838:           pybind11::gil_scoped_release no_gil;
4839:           return at::mm_out(out, self, mat2);
4840:         };
4841:         return wrap(dispatch_mm_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4842:       }
4843:     }
4844:     case 1: {
4845:       if (_r.isNone(3)) {
4846:         // aten::mm.dtype(Tensor self, Tensor mat2, ScalarType out_dtype) -> Tensor
4847: 
4848:         auto dispatch_mm = [](const at::Tensor & self, const at::Tensor & mat2, at::ScalarType out_dtype) -> at::Tensor {
4849:           pybind11::gil_scoped_release no_gil;
4850:           return at::mm(self, mat2, out_dtype);
4851:         };
4852:         return wrap(dispatch_mm(_r.tensor(0), _r.tensor(1), _r.scalartype(2)));
4853:       } else {
4854:         // aten::mm.dtype_out(Tensor self, Tensor mat2, ScalarType out_dtype, *, Tensor(a!) out) -> Tensor(a!)
4855: 
4856:         auto dispatch_mm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat2, at::ScalarType out_dtype) -> at::Tensor {
4857:           pybind11::gil_scoped_release no_gil;
4858:           return at::mm_out(out, self, mat2, out_dtype);
4859:         };
4860:         return wrap(dispatch_mm_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.scalartype(2)));
4861:       }
4862:     }
4863:   }
4864:   Py_RETURN_NONE;
4865:   END_HANDLE_TH_ERRORS
4866: }
4867: 
4868: // _int_mm
4869: static PyObject * THPVariable__int_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
4870: {
4871:   HANDLE_TH_ERRORS
4872:   static PythonArgParser parser({
4873:     "_int_mm(Tensor input, Tensor mat2, *, Tensor out=None)",
4874:   }, /*traceable=*/true);
4875: 
4876:   ParsedArgs<3> parsed_args;
4877:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4878:   if(_r.has_torch_function()) {
4879:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4880:   }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_mm`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_mm`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4881-4960

```cpp
4881:   if (_r.isNone(2)) {
4882:     // aten::_int_mm(Tensor self, Tensor mat2) -> Tensor
4883: 
4884:     auto dispatch__int_mm = [](const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
4885:       pybind11::gil_scoped_release no_gil;
4886:       return at::_int_mm(self, mat2);
4887:     };
4888:     return wrap(dispatch__int_mm(_r.tensor(0), _r.tensor(1)));
4889:   } else {
4890:     // aten::_int_mm.out(Tensor self, Tensor mat2, *, Tensor(a!) out) -> Tensor(a!)
4891: 
4892:     auto dispatch__int_mm_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
4893:       pybind11::gil_scoped_release no_gil;
4894:       return at::_int_mm_out(out, self, mat2);
4895:     };
4896:     return wrap(dispatch__int_mm_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4897:   }
4898:   Py_RETURN_NONE;
4899:   END_HANDLE_TH_ERRORS
4900: }
4901: 
4902: // _weight_int4pack_mm_with_scales_and_zeros
4903: static PyObject * THPVariable__weight_int4pack_mm_with_scales_and_zeros(PyObject* self_, PyObject* args, PyObject* kwargs)
4904: {
4905:   HANDLE_TH_ERRORS
4906:   static PythonArgParser parser({
4907:     "_weight_int4pack_mm_with_scales_and_zeros(Tensor input, Tensor mat2, int64_t qGroupSize, Tensor qScale, Tensor qZeros)",
4908:   }, /*traceable=*/true);
4909: 
4910:   ParsedArgs<5> parsed_args;
4911:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4912:   if(_r.has_torch_function()) {
4913:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4914:   }
4915:   // aten::_weight_int4pack_mm_with_scales_and_zeros(Tensor self, Tensor mat2, int qGroupSize, Tensor qScale, Tensor qZeros) -> Tensor
4916: 
4917:   auto dispatch__weight_int4pack_mm_with_scales_and_zeros = [](const at::Tensor & self, const at::Tensor & mat2, int64_t qGroupSize, const at::Tensor & qScale, const at::Tensor & qZeros) -> at::Tensor {
4918:     pybind11::gil_scoped_release no_gil;
4919:     return at::_weight_int4pack_mm_with_scales_and_zeros(self, mat2, qGroupSize, qScale, qZeros);
4920:   };
4921:   return wrap(dispatch__weight_int4pack_mm_with_scales_and_zeros(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.tensor(3), _r.tensor(4)));
4922:   Py_RETURN_NONE;
4923:   END_HANDLE_TH_ERRORS
4924: }
4925: 
4926: // _convert_weight_to_int4pack_for_cpu
4927: static PyObject * THPVariable__convert_weight_to_int4pack_for_cpu(PyObject* self_, PyObject* args, PyObject* kwargs)
4928: {
4929:   HANDLE_TH_ERRORS
4930:   static PythonArgParser parser({
4931:     "_convert_weight_to_int4pack_for_cpu(Tensor input, int64_t innerKTiles)",
4932:   }, /*traceable=*/true);
4933: 
4934:   ParsedArgs<2> parsed_args;
4935:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4936:   if(_r.has_torch_function()) {
4937:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4938:   }
4939:   // aten::_convert_weight_to_int4pack_for_cpu(Tensor self, int innerKTiles) -> Tensor
4940: 
4941:   auto dispatch__convert_weight_to_int4pack_for_cpu = [](const at::Tensor & self, int64_t innerKTiles) -> at::Tensor {
4942:     pybind11::gil_scoped_release no_gil;
4943:     return at::_convert_weight_to_int4pack_for_cpu(self, innerKTiles);
4944:   };
4945:   return wrap(dispatch__convert_weight_to_int4pack_for_cpu(_r.tensor(0), _r.toInt64(1)));
4946:   Py_RETURN_NONE;
4947:   END_HANDLE_TH_ERRORS
4948: }
4949: 
4950: \
4951: // multiply
4952: static PyObject * THPVariable_multiply(PyObject* self_, PyObject* args, PyObject* kwargs)
4953: {
4954:   HANDLE_TH_ERRORS
4955:   static PythonArgParser parser({
4956:     "multiply(Tensor input, Tensor other, *, Tensor out=None)",
4957:     "multiply(Tensor input, Scalar other)",
4958:   }, /*traceable=*/true);
4959: 
4960:   ParsedArgs<3> parsed_args;
```

- EN: The main execution path in this span is carried by `_int_mm`, `wrap`, `_int_mm_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_int_mm`, `wrap`, `_int_mm_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4961-5040

```cpp
4961:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
4962:   if(_r.has_torch_function()) {
4963:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
4964:   }
4965:   switch (_r.idx) {
4966:     case 0: {
4967:       if (_r.isNone(2)) {
4968:         // aten::multiply.Tensor(Tensor self, Tensor other) -> Tensor
4969: 
4970:         auto dispatch_multiply = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4971:           pybind11::gil_scoped_release no_gil;
4972:           return self.multiply(other);
4973:         };
4974:         return wrap(dispatch_multiply(_r.tensor(0), _r.tensor(1)));
4975:       } else {
4976:         // aten::multiply.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
4977: 
4978:         auto dispatch_multiply_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4979:           pybind11::gil_scoped_release no_gil;
4980:           return at::multiply_out(out, self, other);
4981:         };
4982:         return wrap(dispatch_multiply_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
4983:       }
4984:     }
4985:     case 1: {
4986:       // aten::multiply.Scalar(Tensor self, Scalar other) -> Tensor
4987: 
4988:       auto dispatch_multiply = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4989:         pybind11::gil_scoped_release no_gil;
4990:         return self.multiply(other);
4991:       };
4992:       return wrap(dispatch_multiply(_r.tensor(0), _r.scalar(1)));
4993:     }
4994:   }
4995:   Py_RETURN_NONE;
4996:   END_HANDLE_TH_ERRORS
4997: }
4998: 
4999: // mvlgamma
5000: static PyObject * THPVariable_mvlgamma(PyObject* self_, PyObject* args, PyObject* kwargs)
5001: {
5002:   HANDLE_TH_ERRORS
5003:   static PythonArgParser parser({
5004:     "mvlgamma(Tensor input, int64_t p, *, Tensor out=None)",
5005:   }, /*traceable=*/true);
5006: 
5007:   ParsedArgs<3> parsed_args;
5008:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5009:   if(_r.has_torch_function()) {
5010:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5011:   }
5012:   if (_r.isNone(2)) {
5013:     // aten::mvlgamma(Tensor self, int p) -> Tensor
5014: 
5015:     auto dispatch_mvlgamma = [](const at::Tensor & self, int64_t p) -> at::Tensor {
5016:       pybind11::gil_scoped_release no_gil;
5017:       return self.mvlgamma(p);
5018:     };
5019:     return wrap(dispatch_mvlgamma(_r.tensor(0), _r.toInt64(1)));
5020:   } else {
5021:     // aten::mvlgamma.out(Tensor self, int p, *, Tensor(a!) out) -> Tensor(a!)
5022: 
5023:     auto dispatch_mvlgamma_out = [](at::Tensor out, const at::Tensor & self, int64_t p) -> at::Tensor {
5024:       pybind11::gil_scoped_release no_gil;
5025:       return at::mvlgamma_out(out, self, p);
5026:     };
5027:     return wrap(dispatch_mvlgamma_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
5028:   }
5029:   Py_RETURN_NONE;
5030:   END_HANDLE_TH_ERRORS
5031: }
5032: 
5033: // narrow_copy
5034: static PyObject * THPVariable_narrow_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
5035: {
5036:   HANDLE_TH_ERRORS
5037:   static PythonArgParser parser({
5038:     "narrow_copy(Tensor input, int64_t dim, SymInt start, SymInt length, *, Tensor out=None)",
5039:   }, /*traceable=*/true);
5040: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `multiply_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `multiply_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5120

```cpp
5041:   ParsedArgs<5> parsed_args;
5042:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5043:   if(_r.has_torch_function()) {
5044:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5045:   }
5046:   if (_r.isNone(4)) {
5047:     // aten::narrow_copy(Tensor self, int dim, SymInt start, SymInt length) -> Tensor
5048: 
5049:     auto dispatch_narrow_copy = [](const at::Tensor & self, int64_t dim, c10::SymInt start, c10::SymInt length) -> at::Tensor {
5050:       pybind11::gil_scoped_release no_gil;
5051:       return self.narrow_copy_symint(dim, start, length);
5052:     };
5053:     return wrap(dispatch_narrow_copy(_r.tensor(0), _r.toInt64(1), _r.toSymInt(2), _r.toSymInt(3)));
5054:   } else {
5055:     // aten::narrow_copy.out(Tensor self, int dim, SymInt start, SymInt length, *, Tensor(a!) out) -> Tensor(a!)
5056: 
5057:     auto dispatch_narrow_copy_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, c10::SymInt start, c10::SymInt length) -> at::Tensor {
5058:       pybind11::gil_scoped_release no_gil;
5059:       return at::narrow_copy_symint_out(out, self, dim, start, length);
5060:     };
5061:     return wrap(dispatch_narrow_copy_out(_r.tensor(4), _r.tensor(0), _r.toInt64(1), _r.toSymInt(2), _r.toSymInt(3)));
5062:   }
5063:   Py_RETURN_NONE;
5064:   END_HANDLE_TH_ERRORS
5065: }
5066: 
5067: // _native_batch_norm_legit_no_training
5068: static PyObject * THPVariable__native_batch_norm_legit_no_training(PyObject* self_, PyObject* args, PyObject* kwargs)
5069: {
5070:   HANDLE_TH_ERRORS
5071:   static PythonArgParser parser({
5072:     "_native_batch_norm_legit_no_training(Tensor input, Tensor? weight, Tensor? bias, Tensor running_mean, Tensor running_var, double momentum, double eps)",
5073:   }, /*traceable=*/true);
5074: 
5075:   ParsedArgs<7> parsed_args;
5076:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5077:   if(_r.has_torch_function()) {
5078:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5079:   }
5080:   // aten::_native_batch_norm_legit_no_training(Tensor input, Tensor? weight, Tensor? bias, Tensor running_mean, Tensor running_var, float momentum, float eps) -> (Tensor, Tensor, Tensor)
5081: 
5082:   auto dispatch__native_batch_norm_legit_no_training = [](const at::Tensor & input, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, const at::Tensor & running_mean, const at::Tensor & running_var, double momentum, double eps) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
5083:     pybind11::gil_scoped_release no_gil;
5084:     return at::_native_batch_norm_legit_no_training(input, weight, bias, running_mean, running_var, momentum, eps);
5085:   };
5086:   return wrap(dispatch__native_batch_norm_legit_no_training(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.tensor(3), _r.tensor(4), _r.toDouble(5), _r.toDouble(6)));
5087:   Py_RETURN_NONE;
5088:   END_HANDLE_TH_ERRORS
5089: }
5090: 
5091: // batch_norm_elemt
5092: static PyObject * THPVariable_batch_norm_elemt(PyObject* self_, PyObject* args, PyObject* kwargs)
5093: {
5094:   HANDLE_TH_ERRORS
5095:   static PythonArgParser parser({
5096:     "batch_norm_elemt(Tensor input, Tensor? weight, Tensor? bias, Tensor mean, Tensor invstd, double eps, *, Tensor out=None)",
5097:   }, /*traceable=*/true);
5098: 
5099:   ParsedArgs<7> parsed_args;
5100:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5101:   if(_r.has_torch_function()) {
5102:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5103:   }
5104:   if (_r.isNone(6)) {
5105:     // aten::batch_norm_elemt(Tensor input, Tensor? weight, Tensor? bias, Tensor mean, Tensor invstd, float eps) -> Tensor
5106: 
5107:     auto dispatch_batch_norm_elemt = [](const at::Tensor & input, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, const at::Tensor & mean, const at::Tensor & invstd, double eps) -> at::Tensor {
5108:       pybind11::gil_scoped_release no_gil;
5109:       return at::batch_norm_elemt(input, weight, bias, mean, invstd, eps);
5110:     };
5111:     return wrap(dispatch_batch_norm_elemt(_r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.tensor(3), _r.tensor(4), _r.toDouble(5)));
5112:   } else {
5113:     // aten::batch_norm_elemt.out(Tensor input, Tensor? weight, Tensor? bias, Tensor mean, Tensor invstd, float eps, *, Tensor(a!) out) -> Tensor(a!)
5114: 
5115:     auto dispatch_batch_norm_elemt_out = [](at::Tensor out, const at::Tensor & input, const ::std::optional<at::Tensor> & weight, const ::std::optional<at::Tensor> & bias, const at::Tensor & mean, const at::Tensor & invstd, double eps) -> at::Tensor {
5116:       pybind11::gil_scoped_release no_gil;
5117:       return at::batch_norm_elemt_out(out, input, weight, bias, mean, invstd, eps);
5118:     };
5119:     return wrap(dispatch_batch_norm_elemt_out(_r.tensor(6), _r.tensor(0), _r.optionalTensor(1), _r.optionalTensor(2), _r.tensor(3), _r.tensor(4), _r.toDouble(5)));
5120:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `narrow_copy`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `narrow_copy`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5121-5200

```cpp
5121:   Py_RETURN_NONE;
5122:   END_HANDLE_TH_ERRORS
5123: }
5124: 
5125: // batch_norm_gather_stats_with_counts
5126: static PyObject * THPVariable_batch_norm_gather_stats_with_counts(PyObject* self_, PyObject* args, PyObject* kwargs)
5127: {
5128:   HANDLE_TH_ERRORS
5129:   static PythonArgParser parser({
5130:     "batch_norm_gather_stats_with_counts(Tensor input, Tensor mean, Tensor invstd, Tensor? running_mean, Tensor? running_var, double momentum, double eps, Tensor counts)",
5131:   }, /*traceable=*/true);
5132: 
5133:   ParsedArgs<8> parsed_args;
5134:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5135:   if(_r.has_torch_function()) {
5136:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5137:   }
5138:   // aten::batch_norm_gather_stats_with_counts(Tensor input, Tensor mean, Tensor invstd, Tensor? running_mean, Tensor? running_var, float momentum, float eps, Tensor counts) -> (Tensor, Tensor)
5139: 
5140:   auto dispatch_batch_norm_gather_stats_with_counts = [](const at::Tensor & input, const at::Tensor & mean, const at::Tensor & invstd, const ::std::optional<at::Tensor> & running_mean, const ::std::optional<at::Tensor> & running_var, double momentum, double eps, const at::Tensor & counts) -> ::std::tuple<at::Tensor,at::Tensor> {
5141:     pybind11::gil_scoped_release no_gil;
5142:     return at::batch_norm_gather_stats_with_counts(input, mean, invstd, running_mean, running_var, momentum, eps, counts);
5143:   };
5144:   return wrap(dispatch_batch_norm_gather_stats_with_counts(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toDouble(5), _r.toDouble(6), _r.tensor(7)));
5145:   Py_RETURN_NONE;
5146:   END_HANDLE_TH_ERRORS
5147: }
5148: 
5149: // batch_norm_backward_elemt
5150: static PyObject * THPVariable_batch_norm_backward_elemt(PyObject* self_, PyObject* args, PyObject* kwargs)
5151: {
5152:   HANDLE_TH_ERRORS
5153:   static PythonArgParser parser({
5154:     "batch_norm_backward_elemt(Tensor grad_out, Tensor input, Tensor mean, Tensor invstd, Tensor? weight, Tensor sum_dy, Tensor sum_dy_xmu, Tensor count)",
5155:   }, /*traceable=*/true);
5156: 
5157:   ParsedArgs<8> parsed_args;
5158:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5159:   if(_r.has_torch_function()) {
5160:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5161:   }
5162:   // aten::batch_norm_backward_elemt(Tensor grad_out, Tensor input, Tensor mean, Tensor invstd, Tensor? weight, Tensor sum_dy, Tensor sum_dy_xmu, Tensor count) -> Tensor
5163: 
5164:   auto dispatch_batch_norm_backward_elemt = [](const at::Tensor & grad_out, const at::Tensor & input, const at::Tensor & mean, const at::Tensor & invstd, const ::std::optional<at::Tensor> & weight, const at::Tensor & sum_dy, const at::Tensor & sum_dy_xmu, const at::Tensor & count) -> at::Tensor {
5165:     pybind11::gil_scoped_release no_gil;
5166:     return at::batch_norm_backward_elemt(grad_out, input, mean, invstd, weight, sum_dy, sum_dy_xmu, count);
5167:   };
5168:   return wrap(dispatch_batch_norm_backward_elemt(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.optionalTensor(4), _r.tensor(5), _r.tensor(6), _r.tensor(7)));
5169:   Py_RETURN_NONE;
5170:   END_HANDLE_TH_ERRORS
5171: }
5172: 
5173: // ones_like
5174: static PyObject * THPVariable_ones_like(PyObject* self_, PyObject* args, PyObject* kwargs)
5175: {
5176:   HANDLE_TH_ERRORS
5177:   static PythonArgParser parser({
5178:     "ones_like(Tensor input, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
5179:   }, /*traceable=*/true);
5180: 
5181:   ParsedArgs<7> parsed_args;
5182:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5183:   if(_r.has_torch_function()) {
5184:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5185:   }
5186:   // aten::ones_like(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
5187:   auto self = _r.tensor(0);
5188:   const auto options = TensorOptions()
5189:       .dtype(_r.scalartypeOptional(2))
5190:       .device(_r.deviceOptional(4))
5191:       .layout(_r.layoutOptional(3))
5192:       .requires_grad(_r.toBool(6))
5193:       .pinned_memory(_r.toBool(5));
5194:   torch::utils::maybe_initialize_device(options);
5195: 
5196:   auto dispatch_ones_like = [](const at::Tensor & self, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
5197:     pybind11::gil_scoped_release no_gil;
5198:     return torch::ones_like(self, options, memory_format);
5199:   };
5200:   return wrap(dispatch_ones_like(self, options, _r.memoryformatOptional(1)));
```

- EN: The main execution path in this span is carried by `THPVariable_batch_norm_gather_stats_with_counts`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_batch_norm_gather_stats_with_counts`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5201-5280

```cpp
5201:   Py_RETURN_NONE;
5202:   END_HANDLE_TH_ERRORS
5203: }
5204: 
5205: // pairwise_distance
5206: static PyObject * THPVariable_pairwise_distance(PyObject* self_, PyObject* args, PyObject* kwargs)
5207: {
5208:   HANDLE_TH_ERRORS
5209:   static PythonArgParser parser({
5210:     "pairwise_distance(Tensor x1, Tensor x2, double p=2, double eps=1e-06, bool keepdim=False)",
5211:   }, /*traceable=*/true);
5212: 
5213:   ParsedArgs<5> parsed_args;
5214:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5215:   if(_r.has_torch_function()) {
5216:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5217:   }
5218:   // aten::pairwise_distance(Tensor x1, Tensor x2, float p=2, float eps=1e-06, bool keepdim=False) -> Tensor
5219: 
5220:   auto dispatch_pairwise_distance = [](const at::Tensor & x1, const at::Tensor & x2, double p, double eps, bool keepdim) -> at::Tensor {
5221:     pybind11::gil_scoped_release no_gil;
5222:     return at::pairwise_distance(x1, x2, p, eps, keepdim);
5223:   };
5224:   return wrap(dispatch_pairwise_distance(_r.tensor(0), _r.tensor(1), _r.toDouble(2), _r.toDouble(3), _r.toBool(4)));
5225:   Py_RETURN_NONE;
5226:   END_HANDLE_TH_ERRORS
5227: }
5228: 
5229: // cdist
5230: static PyObject * THPVariable_cdist(PyObject* self_, PyObject* args, PyObject* kwargs)
5231: {
5232:   HANDLE_TH_ERRORS
5233:   static PythonArgParser parser({
5234:     "cdist(Tensor x1, Tensor x2, double p=2, int64_t? compute_mode=None)",
5235:   }, /*traceable=*/true);
5236: 
5237:   ParsedArgs<4> parsed_args;
5238:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5239:   if(_r.has_torch_function()) {
5240:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5241:   }
5242:   // aten::cdist(Tensor x1, Tensor x2, float p=2, int? compute_mode=None) -> Tensor
5243: 
5244:   auto dispatch_cdist = [](const at::Tensor & x1, const at::Tensor & x2, double p, ::std::optional<int64_t> compute_mode) -> at::Tensor {
5245:     pybind11::gil_scoped_release no_gil;
5246:     return at::cdist(x1, x2, p, compute_mode);
5247:   };
5248:   return wrap(dispatch_cdist(_r.tensor(0), _r.tensor(1), _r.toDouble(2), _r.toInt64Optional(3)));
5249:   Py_RETURN_NONE;
5250:   END_HANDLE_TH_ERRORS
5251: }
5252: 
5253: // cosine_similarity
5254: static PyObject * THPVariable_cosine_similarity(PyObject* self_, PyObject* args, PyObject* kwargs)
5255: {
5256:   HANDLE_TH_ERRORS
5257:   static PythonArgParser parser({
5258:     "cosine_similarity(Tensor x1, Tensor x2, int64_t dim=1, double eps=1e-08)",
5259:   }, /*traceable=*/true);
5260: 
5261:   ParsedArgs<4> parsed_args;
5262:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5263:   if(_r.has_torch_function()) {
5264:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5265:   }
5266:   // aten::cosine_similarity(Tensor x1, Tensor x2, int dim=1, float eps=1e-08) -> Tensor
5267: 
5268:   auto dispatch_cosine_similarity = [](const at::Tensor & x1, const at::Tensor & x2, int64_t dim, double eps) -> at::Tensor {
5269:     pybind11::gil_scoped_release no_gil;
5270:     return at::cosine_similarity(x1, x2, dim, eps);
5271:   };
5272:   return wrap(dispatch_cosine_similarity(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.toDouble(3)));
5273:   Py_RETURN_NONE;
5274:   END_HANDLE_TH_ERRORS
5275: }
5276: 
5277: \
5278: // movedim
5279: static PyObject * THPVariable_movedim(PyObject* self_, PyObject* args, PyObject* kwargs)
5280: {
```

- EN: The main execution path in this span is carried by `THPVariable_pairwise_distance`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_pairwise_distance`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5360

```cpp
5281:   HANDLE_TH_ERRORS
5282:   static PythonArgParser parser({
5283:     "movedim(Tensor input, int64_t source, int64_t destination)",
5284:     "movedim(Tensor input, IntArrayRef source, IntArrayRef destination)",
5285:   }, /*traceable=*/true);
5286: 
5287:   ParsedArgs<3> parsed_args;
5288:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5289:   if(_r.has_torch_function()) {
5290:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5291:   }
5292:   switch (_r.idx) {
5293:     case 0: {
5294:       // aten::movedim.int(Tensor(a) self, int source, int destination) -> Tensor(a)
5295: 
5296:       auto dispatch_movedim = [](const at::Tensor & self, int64_t source, int64_t destination) -> at::Tensor {
5297:         pybind11::gil_scoped_release no_gil;
5298:         return self.movedim(source, destination);
5299:       };
5300:       return wrap(dispatch_movedim(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
5301:     }
5302:     case 1: {
5303:       // aten::movedim.intlist(Tensor(a) self, int[] source, int[] destination) -> Tensor(a)
5304: 
5305:       auto dispatch_movedim = [](const at::Tensor & self, at::IntArrayRef source, at::IntArrayRef destination) -> at::Tensor {
5306:         pybind11::gil_scoped_release no_gil;
5307:         return self.movedim(source, destination);
5308:       };
5309:       return wrap(dispatch_movedim(_r.tensor(0), _r.intlist(1), _r.intlist(2)));
5310:     }
5311:   }
5312:   Py_RETURN_NONE;
5313:   END_HANDLE_TH_ERRORS
5314: }
5315: 
5316: // adjoint
5317: static PyObject * THPVariable_adjoint(PyObject* self_, PyObject* args, PyObject* kwargs)
5318: {
5319:   HANDLE_TH_ERRORS
5320:   static PythonArgParser parser({
5321:     "adjoint(Tensor input)",
5322:   }, /*traceable=*/true);
5323: 
5324:   ParsedArgs<1> parsed_args;
5325:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5326:   if(_r.has_torch_function()) {
5327:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5328:   }
5329:   // aten::adjoint(Tensor(a) self) -> Tensor(a)
5330: 
5331:   auto dispatch_adjoint = [](const at::Tensor & self) -> at::Tensor {
5332:     pybind11::gil_scoped_release no_gil;
5333:     return self.adjoint();
5334:   };
5335:   return wrap(dispatch_adjoint(_r.tensor(0)));
5336:   Py_RETURN_NONE;
5337:   END_HANDLE_TH_ERRORS
5338: }
5339: 
5340: // channel_shuffle
5341: static PyObject * THPVariable_channel_shuffle(PyObject* self_, PyObject* args, PyObject* kwargs)
5342: {
5343:   HANDLE_TH_ERRORS
5344:   static PythonArgParser parser({
5345:     "channel_shuffle(Tensor input, SymInt groups)",
5346:   }, /*traceable=*/true);
5347: 
5348:   ParsedArgs<2> parsed_args;
5349:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5350:   if(_r.has_torch_function()) {
5351:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5352:   }
5353:   // aten::channel_shuffle(Tensor self, SymInt groups) -> Tensor
5354: 
5355:   auto dispatch_channel_shuffle = [](const at::Tensor & self, c10::SymInt groups) -> at::Tensor {
5356:     pybind11::gil_scoped_release no_gil;
5357:     return at::channel_shuffle_symint(self, groups);
5358:   };
5359:   return wrap(dispatch_channel_shuffle(_r.tensor(0), _r.toSymInt(1)));
5360:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5361-5440

```cpp
5361:   END_HANDLE_TH_ERRORS
5362: }
5363: 
5364: // pinverse
5365: static PyObject * THPVariable_pinverse(PyObject* self_, PyObject* args, PyObject* kwargs)
5366: {
5367:   HANDLE_TH_ERRORS
5368:   static PythonArgParser parser({
5369:     "pinverse(Tensor input, double rcond=1e-15)",
5370:   }, /*traceable=*/true);
5371: 
5372:   ParsedArgs<2> parsed_args;
5373:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5374:   if(_r.has_torch_function()) {
5375:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5376:   }
5377:   // aten::pinverse(Tensor self, float rcond=1e-15) -> Tensor
5378: 
5379:   auto dispatch_pinverse = [](const at::Tensor & self, double rcond) -> at::Tensor {
5380:     pybind11::gil_scoped_release no_gil;
5381:     return self.pinverse(rcond);
5382:   };
5383:   return wrap(dispatch_pinverse(_r.tensor(0), _r.toDouble(1)));
5384:   Py_RETURN_NONE;
5385:   END_HANDLE_TH_ERRORS
5386: }
5387: 
5388: // poisson_nll_loss
5389: static PyObject * THPVariable_poisson_nll_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
5390: {
5391:   HANDLE_TH_ERRORS
5392:   static PythonArgParser parser({
5393:     "poisson_nll_loss(Tensor input, Tensor target, bool log_input, bool full, double eps, int64_t reduction)",
5394:   }, /*traceable=*/true);
5395: 
5396:   ParsedArgs<6> parsed_args;
5397:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5398:   if(_r.has_torch_function()) {
5399:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5400:   }
5401:   // aten::poisson_nll_loss(Tensor input, Tensor target, bool log_input, bool full, float eps, int reduction) -> Tensor
5402: 
5403:   auto dispatch_poisson_nll_loss = [](const at::Tensor & input, const at::Tensor & target, bool log_input, bool full, double eps, int64_t reduction) -> at::Tensor {
5404:     pybind11::gil_scoped_release no_gil;
5405:     return at::poisson_nll_loss(input, target, log_input, full, eps, reduction);
5406:   };
5407:   return wrap(dispatch_poisson_nll_loss(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3), _r.toDouble(4), _r.toInt64(5)));
5408:   Py_RETURN_NONE;
5409:   END_HANDLE_TH_ERRORS
5410: }
5411: 
5412: // deg2rad
5413: static PyObject * THPVariable_deg2rad(PyObject* self_, PyObject* args, PyObject* kwargs)
5414: {
5415:   HANDLE_TH_ERRORS
5416:   static PythonArgParser parser({
5417:     "deg2rad(Tensor input, *, Tensor out=None)",
5418:   }, /*traceable=*/true);
5419: 
5420:   ParsedArgs<2> parsed_args;
5421:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5422:   if(_r.has_torch_function()) {
5423:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5424:   }
5425:   if (_r.isNone(1)) {
5426:     // aten::deg2rad(Tensor self) -> Tensor
5427: 
5428:     auto dispatch_deg2rad = [](const at::Tensor & self) -> at::Tensor {
5429:       pybind11::gil_scoped_release no_gil;
5430:       return self.deg2rad();
5431:     };
5432:     return wrap(dispatch_deg2rad(_r.tensor(0)));
5433:   } else {
5434:     // aten::deg2rad.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
5435: 
5436:     auto dispatch_deg2rad_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
5437:       pybind11::gil_scoped_release no_gil;
5438:       return at::deg2rad_out(out, self);
5439:     };
5440:     return wrap(dispatch_deg2rad_out(_r.tensor(1), _r.tensor(0)));
```

- EN: The main execution path in this span is carried by `THPVariable_pinverse`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_pinverse`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5441-5520

```cpp
5441:   }
5442:   Py_RETURN_NONE;
5443:   END_HANDLE_TH_ERRORS
5444: }
5445: 
5446: // deg2rad_
5447: static PyObject * THPVariable_deg2rad_(PyObject* self_, PyObject* args, PyObject* kwargs)
5448: {
5449:   HANDLE_TH_ERRORS
5450:   static PythonArgParser parser({
5451:     "deg2rad_(Tensor input)",
5452:   }, /*traceable=*/true);
5453: 
5454:   ParsedArgs<1> parsed_args;
5455:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5456:   if(_r.has_torch_function()) {
5457:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5458:   }
5459:   // aten::deg2rad_(Tensor(a!) self) -> Tensor(a!)
5460: 
5461:   auto dispatch_deg2rad_ = [](at::Tensor self) -> at::Tensor {
5462:     pybind11::gil_scoped_release no_gil;
5463:     return self.deg2rad_();
5464:   };
5465:   return wrap(dispatch_deg2rad_(_r.tensor(0)));
5466:   Py_RETURN_NONE;
5467:   END_HANDLE_TH_ERRORS
5468: }
5469: 
5470: \
5471: // randn_like
5472: static PyObject * THPVariable_randn_like(PyObject* self_, PyObject* args, PyObject* kwargs)
5473: {
5474:   HANDLE_TH_ERRORS
5475:   static PythonArgParser parser({
5476:     "randn_like(Tensor input, *, Generator? generator, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
5477:     "randn_like(Tensor input, *, MemoryFormat? memory_format=None, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
5478:   }, /*traceable=*/true);
5479: 
5480:   ParsedArgs<8> parsed_args;
5481:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5482:   if(_r.has_torch_function()) {
5483:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5484:   }
5485:   switch (_r.idx) {
5486:     case 0: {
5487:       // aten::randn_like.generator(Tensor self, *, Generator? generator, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
5488:       auto self = _r.tensor(0);
5489:       const auto options = TensorOptions()
5490:           .dtype(_r.scalartypeOptional(3))
5491:           .device(_r.deviceOptional(5))
5492:           .layout(_r.layoutOptional(4))
5493:           .requires_grad(_r.toBool(7))
5494:           .pinned_memory(_r.toBool(6));
5495:       torch::utils::maybe_initialize_device(options);
5496: 
5497:       auto dispatch_randn_like = [](const at::Tensor & self, ::std::optional<at::Generator> generator, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
5498:         pybind11::gil_scoped_release no_gil;
5499:         return torch::randn_like(self, generator, options, memory_format);
5500:       };
5501:       return wrap(dispatch_randn_like(self, _r.generator(1), options, _r.memoryformatOptional(2)));
5502:     }
5503:     case 1: {
5504:       // aten::randn_like(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor
5505:       auto self = _r.tensor(0);
5506:       const auto options = TensorOptions()
5507:           .dtype(_r.scalartypeOptional(2))
5508:           .device(_r.deviceOptional(4))
5509:           .layout(_r.layoutOptional(3))
5510:           .requires_grad(_r.toBool(6))
5511:           .pinned_memory(_r.toBool(5));
5512:       torch::utils::maybe_initialize_device(options);
5513: 
5514:       auto dispatch_randn_like = [](const at::Tensor & self, at::TensorOptions options, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
5515:         pybind11::gil_scoped_release no_gil;
5516:         return torch::randn_like(self, options, memory_format);
5517:       };
5518:       return wrap(dispatch_randn_like(self, options, _r.memoryformatOptional(1)));
5519:     }
5520:   }
```

- EN: The main execution path in this span is carried by `THPVariable_deg2rad_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_deg2rad_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5521-5600

```cpp
5521:   Py_RETURN_NONE;
5522:   END_HANDLE_TH_ERRORS
5523: }
5524: 
5525: \
5526: // repeat_interleave
5527: static PyObject * THPVariable_repeat_interleave(PyObject* self_, PyObject* args, PyObject* kwargs)
5528: {
5529:   HANDLE_TH_ERRORS
5530:   static PythonArgParser parser({
5531:     "repeat_interleave(Tensor input, Tensor repeats, int64_t? dim=None, *, SymInt? output_size=None)",
5532:     "repeat_interleave(Tensor repeats, *, SymInt? output_size=None)",
5533:     "repeat_interleave(Tensor input, SymInt repeats, int64_t? dim=None, *, SymInt? output_size=None)",
5534:   }, /*traceable=*/true);
5535: 
5536:   ParsedArgs<4> parsed_args;
5537:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5538:   if(_r.has_torch_function()) {
5539:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5540:   }
5541:   switch (_r.idx) {
5542:     case 0: {
5543:       // aten::repeat_interleave.self_Tensor(Tensor self, Tensor repeats, int? dim=None, *, SymInt? output_size=None) -> Tensor
5544: 
5545:       auto dispatch_repeat_interleave = [](const at::Tensor & self, const at::Tensor & repeats, ::std::optional<int64_t> dim, ::std::optional<c10::SymInt> output_size) -> at::Tensor {
5546:         pybind11::gil_scoped_release no_gil;
5547:         return self.repeat_interleave_symint(repeats, dim, output_size);
5548:       };
5549:       return wrap(dispatch_repeat_interleave(_r.tensor(0), _r.tensor(1), _r.toInt64Optional(2), _r.toSymIntOptional(3)));
5550:     }
5551:     case 1: {
5552:       // aten::repeat_interleave.Tensor(Tensor repeats, *, SymInt? output_size=None) -> Tensor
5553: 
5554:       auto dispatch_repeat_interleave = [](const at::Tensor & repeats, ::std::optional<c10::SymInt> output_size) -> at::Tensor {
5555:         pybind11::gil_scoped_release no_gil;
5556:         return at::repeat_interleave_symint(repeats, output_size);
5557:       };
5558:       return wrap(dispatch_repeat_interleave(_r.tensor(0), _r.toSymIntOptional(1)));
5559:     }
5560:     case 2: {
5561:       // aten::repeat_interleave.self_int(Tensor self, SymInt repeats, int? dim=None, *, SymInt? output_size=None) -> Tensor
5562: 
5563:       auto dispatch_repeat_interleave = [](const at::Tensor & self, c10::SymInt repeats, ::std::optional<int64_t> dim, ::std::optional<c10::SymInt> output_size) -> at::Tensor {
5564:         pybind11::gil_scoped_release no_gil;
5565:         return self.repeat_interleave_symint(repeats, dim, output_size);
5566:       };
5567:       return wrap(dispatch_repeat_interleave(_r.tensor(0), _r.toSymInt(1), _r.toInt64Optional(2), _r.toSymIntOptional(3)));
5568:     }
5569:   }
5570:   Py_RETURN_NONE;
5571:   END_HANDLE_TH_ERRORS
5572: }
5573: 
5574: // _prelu_kernel
5575: static PyObject * THPVariable__prelu_kernel(PyObject* self_, PyObject* args, PyObject* kwargs)
5576: {
5577:   HANDLE_TH_ERRORS
5578:   static PythonArgParser parser({
5579:     "_prelu_kernel(Tensor input, Tensor weight)",
5580:   }, /*traceable=*/true);
5581: 
5582:   ParsedArgs<2> parsed_args;
5583:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5584:   if(_r.has_torch_function()) {
5585:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5586:   }
5587:   // aten::_prelu_kernel(Tensor self, Tensor weight) -> Tensor
5588: 
5589:   auto dispatch__prelu_kernel = [](const at::Tensor & self, const at::Tensor & weight) -> at::Tensor {
5590:     pybind11::gil_scoped_release no_gil;
5591:     return at::_prelu_kernel(self, weight);
5592:   };
5593:   return wrap(dispatch__prelu_kernel(_r.tensor(0), _r.tensor(1)));
5594:   Py_RETURN_NONE;
5595:   END_HANDLE_TH_ERRORS
5596: }
5597: 
5598: // hardshrink
5599: static PyObject * THPVariable_hardshrink(PyObject* self_, PyObject* args, PyObject* kwargs)
5600: {
```

- EN: The main execution path in this span is carried by `THPVariable_repeat_interleave`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_repeat_interleave`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5601-5680

```cpp
5601:   HANDLE_TH_ERRORS
5602:   static PythonArgParser parser({
5603:     "hardshrink(Tensor input, Scalar lambd=0.5, *, Tensor out=None)",
5604:   }, /*traceable=*/true);
5605: 
5606:   ParsedArgs<3> parsed_args;
5607:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5608:   if(_r.has_torch_function()) {
5609:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5610:   }
5611:   if (_r.isNone(2)) {
5612:     // aten::hardshrink(Tensor self, Scalar lambd=0.5) -> Tensor
5613: 
5614:     auto dispatch_hardshrink = [](const at::Tensor & self, const at::Scalar & lambd) -> at::Tensor {
5615:       pybind11::gil_scoped_release no_gil;
5616:       return self.hardshrink(lambd);
5617:     };
5618:     return wrap(dispatch_hardshrink(_r.tensor(0), _r.scalar(1)));
5619:   } else {
5620:     // aten::hardshrink.out(Tensor self, Scalar lambd=0.5, *, Tensor(a!) out) -> Tensor(a!)
5621: 
5622:     auto dispatch_hardshrink_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & lambd) -> at::Tensor {
5623:       pybind11::gil_scoped_release no_gil;
5624:       return at::hardshrink_out(out, self, lambd);
5625:     };
5626:     return wrap(dispatch_hardshrink_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
5627:   }
5628:   Py_RETURN_NONE;
5629:   END_HANDLE_TH_ERRORS
5630: }
5631: 
5632: // rsqrt
5633: static PyObject * THPVariable_rsqrt(PyObject* self_, PyObject* args, PyObject* kwargs)
5634: {
5635:   HANDLE_TH_ERRORS
5636:   static PythonArgParser parser({
5637:     "rsqrt(Tensor input, *, Tensor out=None)",
5638:   }, /*traceable=*/true);
5639: 
5640:   ParsedArgs<2> parsed_args;
5641:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5642:   if(_r.has_torch_function()) {
5643:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5644:   }
5645:   if (_r.isNone(1)) {
5646:     // aten::rsqrt(Tensor self) -> Tensor
5647: 
5648:     auto dispatch_rsqrt = [](const at::Tensor & self) -> at::Tensor {
5649:       pybind11::gil_scoped_release no_gil;
5650:       return self.rsqrt();
5651:     };
5652:     return wrap(dispatch_rsqrt(_r.tensor(0)));
5653:   } else {
5654:     // aten::rsqrt.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
5655: 
5656:     auto dispatch_rsqrt_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
5657:       pybind11::gil_scoped_release no_gil;
5658:       return at::rsqrt_out(out, self);
5659:     };
5660:     return wrap(dispatch_rsqrt_out(_r.tensor(1), _r.tensor(0)));
5661:   }
5662:   Py_RETURN_NONE;
5663:   END_HANDLE_TH_ERRORS
5664: }
5665: 
5666: // rsqrt_
5667: static PyObject * THPVariable_rsqrt_(PyObject* self_, PyObject* args, PyObject* kwargs)
5668: {
5669:   HANDLE_TH_ERRORS
5670:   static PythonArgParser parser({
5671:     "rsqrt_(Tensor input)",
5672:   }, /*traceable=*/true);
5673: 
5674:   ParsedArgs<1> parsed_args;
5675:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5676:   if(_r.has_torch_function()) {
5677:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5678:   }
5679:   // aten::rsqrt_(Tensor(a!) self) -> Tensor(a!)
5680: 
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `hardshrink`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `hardshrink` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5681-5760

```cpp
5681:   auto dispatch_rsqrt_ = [](at::Tensor self) -> at::Tensor {
5682:     pybind11::gil_scoped_release no_gil;
5683:     return self.rsqrt_();
5684:   };
5685:   return wrap(dispatch_rsqrt_(_r.tensor(0)));
5686:   Py_RETURN_NONE;
5687:   END_HANDLE_TH_ERRORS
5688: }
5689: 
5690: // celu
5691: static PyObject * THPVariable_celu(PyObject* self_, PyObject* args, PyObject* kwargs)
5692: {
5693:   HANDLE_TH_ERRORS
5694:   static PythonArgParser parser({
5695:     "celu(Tensor input, Scalar alpha=1.0)",
5696:   }, /*traceable=*/true);
5697: 
5698:   ParsedArgs<2> parsed_args;
5699:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5700:   if(_r.has_torch_function()) {
5701:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5702:   }
5703:   // aten::celu(Tensor self, Scalar alpha=1.0) -> Tensor
5704: 
5705:   auto dispatch_celu = [](const at::Tensor & self, const at::Scalar & alpha) -> at::Tensor {
5706:     pybind11::gil_scoped_release no_gil;
5707:     return at::celu(self, alpha);
5708:   };
5709:   return wrap(dispatch_celu(_r.tensor(0), _r.scalar(1)));
5710:   Py_RETURN_NONE;
5711:   END_HANDLE_TH_ERRORS
5712: }
5713: 
5714: // celu_
5715: static PyObject * THPVariable_celu_(PyObject* self_, PyObject* args, PyObject* kwargs)
5716: {
5717:   HANDLE_TH_ERRORS
5718:   static PythonArgParser parser({
5719:     "celu_(Tensor input, Scalar alpha=1.0)",
5720:   }, /*traceable=*/true);
5721: 
5722:   ParsedArgs<2> parsed_args;
5723:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5724:   if(_r.has_torch_function()) {
5725:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5726:   }
5727:   // aten::celu_(Tensor(a!) self, Scalar alpha=1.0) -> Tensor(a!)
5728: 
5729:   auto dispatch_celu_ = [](at::Tensor self, const at::Scalar & alpha) -> at::Tensor {
5730:     pybind11::gil_scoped_release no_gil;
5731:     return at::celu_(self, alpha);
5732:   };
5733:   return wrap(dispatch_celu_(_r.tensor(0), _r.scalar(1)));
5734:   Py_RETURN_NONE;
5735:   END_HANDLE_TH_ERRORS
5736: }
5737: 
5738: // sinc
5739: static PyObject * THPVariable_sinc(PyObject* self_, PyObject* args, PyObject* kwargs)
5740: {
5741:   HANDLE_TH_ERRORS
5742:   static PythonArgParser parser({
5743:     "sinc(Tensor input, *, Tensor out=None)",
5744:   }, /*traceable=*/true);
5745: 
5746:   ParsedArgs<2> parsed_args;
5747:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5748:   if(_r.has_torch_function()) {
5749:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5750:   }
5751:   if (_r.isNone(1)) {
5752:     // aten::sinc(Tensor self) -> Tensor
5753: 
5754:     auto dispatch_sinc = [](const at::Tensor & self) -> at::Tensor {
5755:       pybind11::gil_scoped_release no_gil;
5756:       return self.sinc();
5757:     };
5758:     return wrap(dispatch_sinc(_r.tensor(0)));
5759:   } else {
5760:     // aten::sinc.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_celu`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_celu`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5840

```cpp
5761: 
5762:     auto dispatch_sinc_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
5763:       pybind11::gil_scoped_release no_gil;
5764:       return at::sinc_out(out, self);
5765:     };
5766:     return wrap(dispatch_sinc_out(_r.tensor(1), _r.tensor(0)));
5767:   }
5768:   Py_RETURN_NONE;
5769:   END_HANDLE_TH_ERRORS
5770: }
5771: 
5772: // sinc_
5773: static PyObject * THPVariable_sinc_(PyObject* self_, PyObject* args, PyObject* kwargs)
5774: {
5775:   HANDLE_TH_ERRORS
5776:   static PythonArgParser parser({
5777:     "sinc_(Tensor input)",
5778:   }, /*traceable=*/true);
5779: 
5780:   ParsedArgs<1> parsed_args;
5781:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5782:   if(_r.has_torch_function()) {
5783:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5784:   }
5785:   // aten::sinc_(Tensor(a!) self) -> Tensor(a!)
5786: 
5787:   auto dispatch_sinc_ = [](at::Tensor self) -> at::Tensor {
5788:     pybind11::gil_scoped_release no_gil;
5789:     return self.sinc_();
5790:   };
5791:   return wrap(dispatch_sinc_(_r.tensor(0)));
5792:   Py_RETURN_NONE;
5793:   END_HANDLE_TH_ERRORS
5794: }
5795: 
5796: // sinh
5797: static PyObject * THPVariable_sinh(PyObject* self_, PyObject* args, PyObject* kwargs)
5798: {
5799:   HANDLE_TH_ERRORS
5800:   static PythonArgParser parser({
5801:     "sinh(Tensor input, *, Tensor out=None)",
5802:   }, /*traceable=*/true);
5803: 
5804:   ParsedArgs<2> parsed_args;
5805:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5806:   if(_r.has_torch_function()) {
5807:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5808:   }
5809:   if (_r.isNone(1)) {
5810:     // aten::sinh(Tensor self) -> Tensor
5811: 
5812:     auto dispatch_sinh = [](const at::Tensor & self) -> at::Tensor {
5813:       pybind11::gil_scoped_release no_gil;
5814:       return self.sinh();
5815:     };
5816:     return wrap(dispatch_sinh(_r.tensor(0)));
5817:   } else {
5818:     // aten::sinh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
5819: 
5820:     auto dispatch_sinh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
5821:       pybind11::gil_scoped_release no_gil;
5822:       return at::sinh_out(out, self);
5823:     };
5824:     return wrap(dispatch_sinh_out(_r.tensor(1), _r.tensor(0)));
5825:   }
5826:   Py_RETURN_NONE;
5827:   END_HANDLE_TH_ERRORS
5828: }
5829: 
5830: // sinh_
5831: static PyObject * THPVariable_sinh_(PyObject* self_, PyObject* args, PyObject* kwargs)
5832: {
5833:   HANDLE_TH_ERRORS
5834:   static PythonArgParser parser({
5835:     "sinh_(Tensor input)",
5836:   }, /*traceable=*/true);
5837: 
5838:   ParsedArgs<1> parsed_args;
5839:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5840:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `sinc_out`, `wrap`, `THPVariable_sinc_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `sinc_out`, `wrap`, `THPVariable_sinc_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5841-5920

```cpp
5841:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5842:   }
5843:   // aten::sinh_(Tensor(a!) self) -> Tensor(a!)
5844: 
5845:   auto dispatch_sinh_ = [](at::Tensor self) -> at::Tensor {
5846:     pybind11::gil_scoped_release no_gil;
5847:     return self.sinh_();
5848:   };
5849:   return wrap(dispatch_sinh_(_r.tensor(0)));
5850:   Py_RETURN_NONE;
5851:   END_HANDLE_TH_ERRORS
5852: }
5853: 
5854: // detach
5855: static PyObject * THPVariable_detach(PyObject* self_, PyObject* args, PyObject* kwargs)
5856: {
5857:   HANDLE_TH_ERRORS
5858:   static PythonArgParser parser({
5859:     "detach(Tensor input)",
5860:   }, /*traceable=*/true);
5861: 
5862:   ParsedArgs<1> parsed_args;
5863:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5864:   if(_r.has_torch_function()) {
5865:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5866:   }
5867:   // aten::detach(Tensor(a) self) -> Tensor(a)
5868: 
5869:   auto dispatch_detach = [](const at::Tensor & self) -> at::Tensor {
5870:     pybind11::gil_scoped_release no_gil;
5871:     return self.detach();
5872:   };
5873:   return wrap(dispatch_detach(_r.tensor(0)));
5874:   Py_RETURN_NONE;
5875:   END_HANDLE_TH_ERRORS
5876: }
5877: 
5878: // detach_
5879: static PyObject * THPVariable_detach_(PyObject* self_, PyObject* args, PyObject* kwargs)
5880: {
5881:   HANDLE_TH_ERRORS
5882:   static PythonArgParser parser({
5883:     "detach_(Tensor input)",
5884:   }, /*traceable=*/true);
5885: 
5886:   ParsedArgs<1> parsed_args;
5887:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5888:   if(_r.has_torch_function()) {
5889:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5890:   }
5891:   // aten::detach_(Tensor(a!) self) -> Tensor(a!)
5892: 
5893:   auto dispatch_detach_ = [](at::Tensor self) -> at::Tensor {
5894:     pybind11::gil_scoped_release no_gil;
5895:     return self.detach_();
5896:   };
5897:   return wrap(dispatch_detach_(_r.tensor(0)));
5898:   Py_RETURN_NONE;
5899:   END_HANDLE_TH_ERRORS
5900: }
5901: 
5902: // _softmax
5903: static PyObject * THPVariable__softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
5904: {
5905:   HANDLE_TH_ERRORS
5906:   static PythonArgParser parser({
5907:     "_softmax(Tensor input, int64_t dim, bool half_to_float, *, Tensor out=None)",
5908:   }, /*traceable=*/true);
5909: 
5910:   ParsedArgs<4> parsed_args;
5911:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5912:   if(_r.has_torch_function()) {
5913:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5914:   }
5915:   if (_r.isNone(3)) {
5916:     // aten::_softmax(Tensor self, int dim, bool half_to_float) -> Tensor
5917: 
5918:     auto dispatch__softmax = [](const at::Tensor & self, int64_t dim, bool half_to_float) -> at::Tensor {
5919:       pybind11::gil_scoped_release no_gil;
5920:       return at::_softmax(self, dim, half_to_float);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `sinh_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `sinh_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5921-6000

```cpp
5921:     };
5922:     return wrap(dispatch__softmax(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5923:   } else {
5924:     // aten::_softmax.out(Tensor self, int dim, bool half_to_float, *, Tensor(a!) out) -> Tensor(a!)
5925: 
5926:     auto dispatch__softmax_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, bool half_to_float) -> at::Tensor {
5927:       pybind11::gil_scoped_release no_gil;
5928:       return at::_softmax_out(out, self, dim, half_to_float);
5929:     };
5930:     return wrap(dispatch__softmax_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.toBool(2)));
5931:   }
5932:   Py_RETURN_NONE;
5933:   END_HANDLE_TH_ERRORS
5934: }
5935: 
5936: // _softmax_backward_data
5937: static PyObject * THPVariable__softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs)
5938: {
5939:   HANDLE_TH_ERRORS
5940:   static PythonArgParser parser({
5941:     "_softmax_backward_data(Tensor grad_output, Tensor output, int64_t dim, ScalarType input_dtype, *, Tensor grad_input=None)",
5942:   }, /*traceable=*/true);
5943: 
5944:   ParsedArgs<5> parsed_args;
5945:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5946:   if(_r.has_torch_function()) {
5947:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5948:   }
5949:   if (_r.isNone(4)) {
5950:     // aten::_softmax_backward_data(Tensor grad_output, Tensor output, int dim, ScalarType input_dtype) -> Tensor
5951: 
5952:     auto dispatch__softmax_backward_data = [](const at::Tensor & grad_output, const at::Tensor & output, int64_t dim, at::ScalarType input_dtype) -> at::Tensor {
5953:       pybind11::gil_scoped_release no_gil;
5954:       return at::_softmax_backward_data(grad_output, output, dim, input_dtype);
5955:     };
5956:     return wrap(dispatch__softmax_backward_data(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.scalartype(3)));
5957:   } else {
5958:     // aten::_softmax_backward_data.out(Tensor grad_output, Tensor output, int dim, ScalarType input_dtype, *, Tensor(a!) grad_input) -> Tensor(a!)
5959: 
5960:     auto dispatch__softmax_backward_data_out = [](at::Tensor grad_input, const at::Tensor & grad_output, const at::Tensor & output, int64_t dim, at::ScalarType input_dtype) -> at::Tensor {
5961:       pybind11::gil_scoped_release no_gil;
5962:       return at::_softmax_backward_data_out(grad_input, grad_output, output, dim, input_dtype);
5963:     };
5964:     return wrap(dispatch__softmax_backward_data_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.scalartype(3)));
5965:   }
5966:   Py_RETURN_NONE;
5967:   END_HANDLE_TH_ERRORS
5968: }
5969: 
5970: // unsafe_split
5971: static PyObject * THPVariable_unsafe_split(PyObject* self_, PyObject* args, PyObject* kwargs)
5972: {
5973:   HANDLE_TH_ERRORS
5974:   static PythonArgParser parser({
5975:     "unsafe_split(Tensor input, SymInt split_size, int64_t dim=0)",
5976:   }, /*traceable=*/true);
5977: 
5978:   ParsedArgs<3> parsed_args;
5979:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
5980:   if(_r.has_torch_function()) {
5981:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
5982:   }
5983:   // aten::unsafe_split.Tensor(Tensor self, SymInt split_size, int dim=0) -> Tensor[]
5984: 
5985:   auto dispatch_unsafe_split = [](const at::Tensor & self, c10::SymInt split_size, int64_t dim) -> ::std::vector<at::Tensor> {
5986:     pybind11::gil_scoped_release no_gil;
5987:     return self.unsafe_split_symint(split_size, dim);
5988:   };
5989:   return wrap(dispatch_unsafe_split(_r.tensor(0), _r.toSymInt(1), _r.toInt64(2)));
5990:   Py_RETURN_NONE;
5991:   END_HANDLE_TH_ERRORS
5992: }
5993: 
5994: \
5995: // split
5996: static PyObject * THPVariable_split(PyObject* self_, PyObject* args, PyObject* kwargs)
5997: {
5998:   HANDLE_TH_ERRORS
5999:   static PythonArgParser parser({
6000:     "split(Tensor input, SymInt split_size, int64_t dim=0)",
```

- EN: The main execution path in this span is carried by `wrap`, `_softmax_out`, `THPVariable__softmax_backward_data`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `_softmax_out`, `THPVariable__softmax_backward_data` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6001-6080

```cpp
6001:     "split(Tensor input, SymIntArrayRef split_size, int64_t dim=0)",
6002:   }, /*traceable=*/true);
6003: 
6004:   ParsedArgs<3> parsed_args;
6005:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6006:   if(_r.has_torch_function()) {
6007:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6008:   }
6009:   switch (_r.idx) {
6010:     case 0: {
6011:       // aten::split.Tensor(Tensor(a -> *) self, SymInt split_size, int dim=0) -> Tensor(a)[]
6012: 
6013:       auto dispatch_split = [](const at::Tensor & self, c10::SymInt split_size, int64_t dim) -> ::std::vector<at::Tensor> {
6014:         pybind11::gil_scoped_release no_gil;
6015:         return self.split_symint(split_size, dim);
6016:       };
6017:       return wrap(dispatch_split(_r.tensor(0), _r.toSymInt(1), _r.toInt64(2)));
6018:     }
6019:     case 1: {
6020:       // aten::split.sizes(Tensor(a -> *) self, SymInt[] split_size, int dim=0) -> Tensor(a)[]
6021: 
6022:       auto dispatch_split = [](const at::Tensor & self, c10::SymIntArrayRef split_size, int64_t dim) -> ::std::vector<at::Tensor> {
6023:         pybind11::gil_scoped_release no_gil;
6024:         return self.split_symint(split_size, dim);
6025:       };
6026:       return wrap(dispatch_split(_r.tensor(0), _r.symintlist(1), _r.toInt64(2)));
6027:     }
6028:   }
6029:   Py_RETURN_NONE;
6030:   END_HANDLE_TH_ERRORS
6031: }
6032: 
6033: \
6034: // hsplit
6035: static PyObject * THPVariable_hsplit(PyObject* self_, PyObject* args, PyObject* kwargs)
6036: {
6037:   HANDLE_TH_ERRORS
6038:   static PythonArgParser parser({
6039:     "hsplit(Tensor input, int64_t sections)",
6040:     "hsplit(Tensor input, IntArrayRef indices)",
6041:   }, /*traceable=*/true);
6042: 
6043:   ParsedArgs<2> parsed_args;
6044:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6045:   if(_r.has_torch_function()) {
6046:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6047:   }
6048:   switch (_r.idx) {
6049:     case 0: {
6050:       // aten::hsplit.int(Tensor(a -> *) self, int sections) -> Tensor(a)[]
6051: 
6052:       auto dispatch_hsplit = [](const at::Tensor & self, int64_t sections) -> ::std::vector<at::Tensor> {
6053:         pybind11::gil_scoped_release no_gil;
6054:         return self.hsplit(sections);
6055:       };
6056:       return wrap(dispatch_hsplit(_r.tensor(0), _r.toInt64(1)));
6057:     }
6058:     case 1: {
6059:       // aten::hsplit.array(Tensor(a -> *) self, int[] indices) -> Tensor(a)[]
6060: 
6061:       auto dispatch_hsplit = [](const at::Tensor & self, at::IntArrayRef indices) -> ::std::vector<at::Tensor> {
6062:         pybind11::gil_scoped_release no_gil;
6063:         return self.hsplit(indices);
6064:       };
6065:       return wrap(dispatch_hsplit(_r.tensor(0), _r.intlist(1)));
6066:     }
6067:   }
6068:   Py_RETURN_NONE;
6069:   END_HANDLE_TH_ERRORS
6070: }
6071: 
6072: \
6073: // squeeze
6074: static PyObject * THPVariable_squeeze(PyObject* self_, PyObject* args, PyObject* kwargs)
6075: {
6076:   HANDLE_TH_ERRORS
6077:   static PythonArgParser parser({
6078:     "squeeze(Tensor input)",
6079:     "squeeze(Tensor input, int64_t dim)",
6080:     "squeeze(Tensor input, IntArrayRef dim)",
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable_hsplit`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable_hsplit` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6081-6160

```cpp
6081:     "squeeze(Tensor input, Dimname dim)",
6082:   }, /*traceable=*/true);
6083: 
6084:   ParsedArgs<2> parsed_args;
6085:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6086:   if(_r.has_torch_function()) {
6087:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6088:   }
6089:   switch (_r.idx) {
6090:     case 0: {
6091:       // aten::squeeze(Tensor(a) self) -> Tensor(a)
6092: 
6093:       auto dispatch_squeeze = [](const at::Tensor & self) -> at::Tensor {
6094:         pybind11::gil_scoped_release no_gil;
6095:         return self.squeeze();
6096:       };
6097:       return wrap(dispatch_squeeze(_r.tensor(0)));
6098:     }
6099:     case 1: {
6100:       // aten::squeeze.dim(Tensor(a) self, int dim) -> Tensor(a)
6101: 
6102:       auto dispatch_squeeze = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
6103:         pybind11::gil_scoped_release no_gil;
6104:         return self.squeeze(dim);
6105:       };
6106:       return wrap(dispatch_squeeze(_r.tensor(0), _r.toInt64(1)));
6107:     }
6108:     case 2: {
6109:       // aten::squeeze.dims(Tensor(a) self, int[] dim) -> Tensor(a)
6110: 
6111:       auto dispatch_squeeze = [](const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
6112:         pybind11::gil_scoped_release no_gil;
6113:         return self.squeeze(dim);
6114:       };
6115:       return wrap(dispatch_squeeze(_r.tensor(0), _r.intlist(1)));
6116:     }
6117:     case 3: {
6118:       // aten::squeeze.dimname(Tensor(a) self, Dimname dim) -> Tensor(a)
6119: 
6120:       auto dispatch_squeeze = [](const at::Tensor & self, at::Dimname dim) -> at::Tensor {
6121:         pybind11::gil_scoped_release no_gil;
6122:         return self.squeeze(dim);
6123:       };
6124:       return wrap(dispatch_squeeze(_r.tensor(0), _r.dimname(1)));
6125:     }
6126:   }
6127:   Py_RETURN_NONE;
6128:   END_HANDLE_TH_ERRORS
6129: }
6130: 
6131: // stack
6132: static PyObject * THPVariable_stack(PyObject* self_, PyObject* args, PyObject* kwargs)
6133: {
6134:   HANDLE_TH_ERRORS
6135:   static PythonArgParser parser({
6136:     "stack(TensorList tensors, int64_t dim=0, *, Tensor out=None)",
6137:   }, /*traceable=*/true);
6138: 
6139:   ParsedArgs<3> parsed_args;
6140:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6141:   if(_r.has_torch_function()) {
6142:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6143:   }
6144:   if (_r.isNone(2)) {
6145:     // aten::stack(Tensor[] tensors, int dim=0) -> Tensor
6146: 
6147:     auto dispatch_stack = [](at::TensorList tensors, int64_t dim) -> at::Tensor {
6148:       pybind11::gil_scoped_release no_gil;
6149:       return at::stack(tensors, dim);
6150:     };
6151:     return wrap(dispatch_stack(_r.tensorlist(0), _r.toInt64(1)));
6152:   } else {
6153:     // aten::stack.out(Tensor[] tensors, int dim=0, *, Tensor(a!) out) -> Tensor(a!)
6154: 
6155:     auto dispatch_stack_out = [](at::Tensor out, at::TensorList tensors, int64_t dim) -> at::Tensor {
6156:       pybind11::gil_scoped_release no_gil;
6157:       return at::stack_out(out, tensors, dim);
6158:     };
6159:     return wrap(dispatch_stack_out(_r.tensor(2), _r.tensorlist(0), _r.toInt64(1)));
6160:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `squeeze`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `squeeze`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6161-6240

```cpp
6161:   Py_RETURN_NONE;
6162:   END_HANDLE_TH_ERRORS
6163: }
6164: 
6165: // _stack
6166: static PyObject * THPVariable__stack(PyObject* self_, PyObject* args, PyObject* kwargs)
6167: {
6168:   HANDLE_TH_ERRORS
6169:   static PythonArgParser parser({
6170:     "_stack(TensorList tensors, int64_t dim=0, *, Tensor out=None)",
6171:   }, /*traceable=*/true);
6172: 
6173:   ParsedArgs<3> parsed_args;
6174:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6175:   if(_r.has_torch_function()) {
6176:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6177:   }
6178:   if (_r.isNone(2)) {
6179:     // aten::_stack(Tensor[] tensors, int dim=0) -> Tensor
6180: 
6181:     auto dispatch__stack = [](at::TensorList tensors, int64_t dim) -> at::Tensor {
6182:       pybind11::gil_scoped_release no_gil;
6183:       return at::_stack(tensors, dim);
6184:     };
6185:     return wrap(dispatch__stack(_r.tensorlist(0), _r.toInt64(1)));
6186:   } else {
6187:     // aten::_stack.out(Tensor[] tensors, int dim=0, *, Tensor(a!) out) -> Tensor(a!)
6188: 
6189:     auto dispatch__stack_out = [](at::Tensor out, at::TensorList tensors, int64_t dim) -> at::Tensor {
6190:       pybind11::gil_scoped_release no_gil;
6191:       return at::_stack_out(out, tensors, dim);
6192:     };
6193:     return wrap(dispatch__stack_out(_r.tensor(2), _r.tensorlist(0), _r.toInt64(1)));
6194:   }
6195:   Py_RETURN_NONE;
6196:   END_HANDLE_TH_ERRORS
6197: }
6198: 
6199: // istft
6200: static PyObject * THPVariable_istft(PyObject* self_, PyObject* args, PyObject* kwargs)
6201: {
6202:   HANDLE_TH_ERRORS
6203:   static PythonArgParser parser({
6204:     "istft(Tensor input, int64_t n_fft, int64_t? hop_length=None, int64_t? win_length=None, Tensor? window=None, bool center=True, bool normalized=False, bool? onesided=None, int64_t? length=None, bool return_complex=False)",
6205:   }, /*traceable=*/true);
6206: 
6207:   ParsedArgs<10> parsed_args;
6208:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6209:   if(_r.has_torch_function()) {
6210:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6211:   }
6212:   // aten::istft(Tensor self, int n_fft, int? hop_length=None, int? win_length=None, Tensor? window=None, bool center=True, bool normalized=False, bool? onesided=None, int? length=None, bool return_complex=False) -> Tensor
6213: 
6214:   auto dispatch_istft = [](const at::Tensor & self, int64_t n_fft, ::std::optional<int64_t> hop_length, ::std::optional<int64_t> win_length, const ::std::optional<at::Tensor> & window, bool center, bool normalized, ::std::optional<bool> onesided, ::std::optional<int64_t> length, bool return_complex) -> at::Tensor {
6215:     pybind11::gil_scoped_release no_gil;
6216:     return self.istft(n_fft, hop_length, win_length, window, center, normalized, onesided, length, return_complex);
6217:   };
6218:   return wrap(dispatch_istft(_r.tensor(0), _r.toInt64(1), _r.toInt64Optional(2), _r.toInt64Optional(3), _r.optionalTensor(4), _r.toBool(5), _r.toBool(6), _r.toBoolOptional(7), _r.toInt64Optional(8), _r.toBool(9)));
6219:   Py_RETURN_NONE;
6220:   END_HANDLE_TH_ERRORS
6221: }
6222: 
6223: // nansum
6224: static PyObject * THPVariable_nansum(PyObject* self_, PyObject* args, PyObject* kwargs)
6225: {
6226:   HANDLE_TH_ERRORS
6227:   static PythonArgParser parser({
6228:     "nansum(Tensor input, IntArrayRef[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None, Tensor out=None)",
6229:   }, /*traceable=*/true);
6230: 
6231:   ParsedArgs<5> parsed_args;
6232:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6233:   if(_r.has_torch_function()) {
6234:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6235:   }
6236:   if (_r.isNone(4)) {
6237:     // aten::nansum(Tensor self, int[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
6238: 
6239:     auto dispatch_nansum = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6240:       pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable__stack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__stack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6241-6320

```cpp
6241:       return self.nansum(dim, keepdim, dtype);
6242:     };
6243:     return wrap(dispatch_nansum(_r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
6244:   } else {
6245:     // aten::nansum.out(Tensor self, int[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None, Tensor(a!) out) -> Tensor(a!)
6246: 
6247:     auto dispatch_nansum_out = [](at::Tensor out, const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6248:       pybind11::gil_scoped_release no_gil;
6249:       return at::nansum_out(out, self, dim, keepdim, dtype);
6250:     };
6251:     return wrap(dispatch_nansum_out(_r.tensor(4), _r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.scalartypeOptional(3)));
6252:   }
6253:   Py_RETURN_NONE;
6254:   END_HANDLE_TH_ERRORS
6255: }
6256: 
6257: // sqrt
6258: static PyObject * THPVariable_sqrt(PyObject* self_, PyObject* args, PyObject* kwargs)
6259: {
6260:   HANDLE_TH_ERRORS
6261:   static PythonArgParser parser({
6262:     "sqrt(Tensor input, *, Tensor out=None)",
6263:   }, /*traceable=*/true);
6264: 
6265:   ParsedArgs<2> parsed_args;
6266:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6267:   if(_r.has_torch_function()) {
6268:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6269:   }
6270:   if (_r.isNone(1)) {
6271:     // aten::sqrt(Tensor self) -> Tensor
6272: 
6273:     auto dispatch_sqrt = [](const at::Tensor & self) -> at::Tensor {
6274:       pybind11::gil_scoped_release no_gil;
6275:       return self.sqrt();
6276:     };
6277:     return wrap(dispatch_sqrt(_r.tensor(0)));
6278:   } else {
6279:     // aten::sqrt.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6280: 
6281:     auto dispatch_sqrt_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6282:       pybind11::gil_scoped_release no_gil;
6283:       return at::sqrt_out(out, self);
6284:     };
6285:     return wrap(dispatch_sqrt_out(_r.tensor(1), _r.tensor(0)));
6286:   }
6287:   Py_RETURN_NONE;
6288:   END_HANDLE_TH_ERRORS
6289: }
6290: 
6291: // sqrt_
6292: static PyObject * THPVariable_sqrt_(PyObject* self_, PyObject* args, PyObject* kwargs)
6293: {
6294:   HANDLE_TH_ERRORS
6295:   static PythonArgParser parser({
6296:     "sqrt_(Tensor input)",
6297:   }, /*traceable=*/true);
6298: 
6299:   ParsedArgs<1> parsed_args;
6300:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6301:   if(_r.has_torch_function()) {
6302:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6303:   }
6304:   // aten::sqrt_(Tensor(a!) self) -> Tensor(a!)
6305: 
6306:   auto dispatch_sqrt_ = [](at::Tensor self) -> at::Tensor {
6307:     pybind11::gil_scoped_release no_gil;
6308:     return self.sqrt_();
6309:   };
6310:   return wrap(dispatch_sqrt_(_r.tensor(0)));
6311:   Py_RETURN_NONE;
6312:   END_HANDLE_TH_ERRORS
6313: }
6314: 
6315: // square
6316: static PyObject * THPVariable_square(PyObject* self_, PyObject* args, PyObject* kwargs)
6317: {
6318:   HANDLE_TH_ERRORS
6319:   static PythonArgParser parser({
6320:     "square(Tensor input, *, Tensor out=None)",
```

- EN: The main execution path in this span is carried by `wrap`, `nansum_out`, `THPVariable_sqrt`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `nansum_out`, `THPVariable_sqrt` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6321-6400

```cpp
6321:   }, /*traceable=*/true);
6322: 
6323:   ParsedArgs<2> parsed_args;
6324:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6325:   if(_r.has_torch_function()) {
6326:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6327:   }
6328:   if (_r.isNone(1)) {
6329:     // aten::square(Tensor self) -> Tensor
6330: 
6331:     auto dispatch_square = [](const at::Tensor & self) -> at::Tensor {
6332:       pybind11::gil_scoped_release no_gil;
6333:       return self.square();
6334:     };
6335:     return wrap(dispatch_square(_r.tensor(0)));
6336:   } else {
6337:     // aten::square.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6338: 
6339:     auto dispatch_square_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6340:       pybind11::gil_scoped_release no_gil;
6341:       return at::square_out(out, self);
6342:     };
6343:     return wrap(dispatch_square_out(_r.tensor(1), _r.tensor(0)));
6344:   }
6345:   Py_RETURN_NONE;
6346:   END_HANDLE_TH_ERRORS
6347: }
6348: 
6349: // square_
6350: static PyObject * THPVariable_square_(PyObject* self_, PyObject* args, PyObject* kwargs)
6351: {
6352:   HANDLE_TH_ERRORS
6353:   static PythonArgParser parser({
6354:     "square_(Tensor input)",
6355:   }, /*traceable=*/true);
6356: 
6357:   ParsedArgs<1> parsed_args;
6358:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6359:   if(_r.has_torch_function()) {
6360:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6361:   }
6362:   // aten::square_(Tensor(a!) self) -> Tensor(a!)
6363: 
6364:   auto dispatch_square_ = [](at::Tensor self) -> at::Tensor {
6365:     pybind11::gil_scoped_release no_gil;
6366:     return self.square_();
6367:   };
6368:   return wrap(dispatch_square_(_r.tensor(0)));
6369:   Py_RETURN_NONE;
6370:   END_HANDLE_TH_ERRORS
6371: }
6372: 
6373: // t
6374: static PyObject * THPVariable_t(PyObject* self_, PyObject* args, PyObject* kwargs)
6375: {
6376:   HANDLE_TH_ERRORS
6377:   static PythonArgParser parser({
6378:     "t(Tensor input)",
6379:   }, /*traceable=*/true);
6380: 
6381:   ParsedArgs<1> parsed_args;
6382:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6383:   if(_r.has_torch_function()) {
6384:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6385:   }
6386:   // aten::t(Tensor(a) self) -> Tensor(a)
6387: 
6388:   auto dispatch_t = [](const at::Tensor & self) -> at::Tensor {
6389:     pybind11::gil_scoped_release no_gil;
6390:     return self.t();
6391:   };
6392:   return wrap(dispatch_t(_r.tensor(0)));
6393:   Py_RETURN_NONE;
6394:   END_HANDLE_TH_ERRORS
6395: }
6396: 
6397: // tanh
6398: static PyObject * THPVariable_tanh(PyObject* self_, PyObject* args, PyObject* kwargs)
6399: {
6400:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `square`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `square`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6401-6480

```cpp
6401:   static PythonArgParser parser({
6402:     "tanh(Tensor input, *, Tensor out=None)",
6403:   }, /*traceable=*/true);
6404: 
6405:   ParsedArgs<2> parsed_args;
6406:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6407:   if(_r.has_torch_function()) {
6408:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6409:   }
6410:   if (_r.isNone(1)) {
6411:     // aten::tanh(Tensor self) -> Tensor
6412: 
6413:     auto dispatch_tanh = [](const at::Tensor & self) -> at::Tensor {
6414:       pybind11::gil_scoped_release no_gil;
6415:       return self.tanh();
6416:     };
6417:     return wrap(dispatch_tanh(_r.tensor(0)));
6418:   } else {
6419:     // aten::tanh.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6420: 
6421:     auto dispatch_tanh_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6422:       pybind11::gil_scoped_release no_gil;
6423:       return at::tanh_out(out, self);
6424:     };
6425:     return wrap(dispatch_tanh_out(_r.tensor(1), _r.tensor(0)));
6426:   }
6427:   Py_RETURN_NONE;
6428:   END_HANDLE_TH_ERRORS
6429: }
6430: 
6431: // tanh_
6432: static PyObject * THPVariable_tanh_(PyObject* self_, PyObject* args, PyObject* kwargs)
6433: {
6434:   HANDLE_TH_ERRORS
6435:   static PythonArgParser parser({
6436:     "tanh_(Tensor input)",
6437:   }, /*traceable=*/true);
6438: 
6439:   ParsedArgs<1> parsed_args;
6440:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6441:   if(_r.has_torch_function()) {
6442:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6443:   }
6444:   // aten::tanh_(Tensor(a!) self) -> Tensor(a!)
6445: 
6446:   auto dispatch_tanh_ = [](at::Tensor self) -> at::Tensor {
6447:     pybind11::gil_scoped_release no_gil;
6448:     return self.tanh_();
6449:   };
6450:   return wrap(dispatch_tanh_(_r.tensor(0)));
6451:   Py_RETURN_NONE;
6452:   END_HANDLE_TH_ERRORS
6453: }
6454: 
6455: // tensordot
6456: static PyObject * THPVariable_tensordot(PyObject* self_, PyObject* args, PyObject* kwargs)
6457: {
6458:   HANDLE_TH_ERRORS
6459:   static PythonArgParser parser({
6460:     "tensordot(Tensor input, Tensor other, IntArrayRef dims_self, IntArrayRef dims_other, *, Tensor out=None)",
6461:   }, /*traceable=*/true);
6462: 
6463:   ParsedArgs<5> parsed_args;
6464:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6465:   if(_r.has_torch_function()) {
6466:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6467:   }
6468:   if (_r.isNone(4)) {
6469:     // aten::tensordot(Tensor self, Tensor other, int[] dims_self, int[] dims_other) -> Tensor
6470: 
6471:     auto dispatch_tensordot = [](const at::Tensor & self, const at::Tensor & other, at::IntArrayRef dims_self, at::IntArrayRef dims_other) -> at::Tensor {
6472:       pybind11::gil_scoped_release no_gil;
6473:       return at::tensordot(self, other, dims_self, dims_other);
6474:     };
6475:     return wrap(dispatch_tensordot(_r.tensor(0), _r.tensor(1), _r.intlist(2), _r.intlist(3)));
6476:   } else {
6477:     // aten::tensordot.out(Tensor self, Tensor other, int[] dims_self, int[] dims_other, *, Tensor(a!) out) -> Tensor(a!)
6478: 
6479:     auto dispatch_tensordot_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, at::IntArrayRef dims_self, at::IntArrayRef dims_other) -> at::Tensor {
6480:       pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `tanh`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `tanh` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6481-6560

```cpp
6481:       return at::tensordot_out(out, self, other, dims_self, dims_other);
6482:     };
6483:     return wrap(dispatch_tensordot_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.intlist(2), _r.intlist(3)));
6484:   }
6485:   Py_RETURN_NONE;
6486:   END_HANDLE_TH_ERRORS
6487: }
6488: 
6489: // tile
6490: static PyObject * THPVariable_tile(PyObject* self_, PyObject* args, PyObject* kwargs)
6491: {
6492:   HANDLE_TH_ERRORS
6493:   static PythonArgParser parser({
6494:     "tile(Tensor input, SymIntArrayRef dims)",
6495:   }, /*traceable=*/true);
6496: 
6497:   ParsedArgs<2> parsed_args;
6498:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6499:   if(_r.has_torch_function()) {
6500:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6501:   }
6502:   // aten::tile(Tensor self, SymInt[] dims) -> Tensor
6503: 
6504:   auto dispatch_tile = [](const at::Tensor & self, c10::SymIntArrayRef dims) -> at::Tensor {
6505:     pybind11::gil_scoped_release no_gil;
6506:     return self.tile_symint(dims);
6507:   };
6508:   return wrap(dispatch_tile(_r.tensor(0), _r.symintlist(1)));
6509:   Py_RETURN_NONE;
6510:   END_HANDLE_TH_ERRORS
6511: }
6512: 
6513: // _mkldnn_transpose
6514: static PyObject * THPVariable__mkldnn_transpose(PyObject* self_, PyObject* args, PyObject* kwargs)
6515: {
6516:   HANDLE_TH_ERRORS
6517:   static PythonArgParser parser({
6518:     "_mkldnn_transpose(Tensor input, int64_t dim0, int64_t dim1)",
6519:   }, /*traceable=*/true);
6520: 
6521:   ParsedArgs<3> parsed_args;
6522:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6523:   if(_r.has_torch_function()) {
6524:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6525:   }
6526:   // aten::_mkldnn_transpose(Tensor self, int dim0, int dim1) -> Tensor
6527: 
6528:   auto dispatch__mkldnn_transpose = [](const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
6529:     pybind11::gil_scoped_release no_gil;
6530:     return at::_mkldnn_transpose(self, dim0, dim1);
6531:   };
6532:   return wrap(dispatch__mkldnn_transpose(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
6533:   Py_RETURN_NONE;
6534:   END_HANDLE_TH_ERRORS
6535: }
6536: 
6537: // _mkldnn_transpose_
6538: static PyObject * THPVariable__mkldnn_transpose_(PyObject* self_, PyObject* args, PyObject* kwargs)
6539: {
6540:   HANDLE_TH_ERRORS
6541:   static PythonArgParser parser({
6542:     "_mkldnn_transpose_(Tensor input, int64_t dim0, int64_t dim1)",
6543:   }, /*traceable=*/true);
6544: 
6545:   ParsedArgs<3> parsed_args;
6546:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6547:   if(_r.has_torch_function()) {
6548:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6549:   }
6550:   // aten::_mkldnn_transpose_(Tensor(a!) self, int dim0, int dim1) -> Tensor(a!)
6551: 
6552:   auto dispatch__mkldnn_transpose_ = [](at::Tensor self, int64_t dim0, int64_t dim1) -> at::Tensor {
6553:     pybind11::gil_scoped_release no_gil;
6554:     return at::_mkldnn_transpose_(self, dim0, dim1);
6555:   };
6556:   return wrap(dispatch__mkldnn_transpose_(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
6557:   Py_RETURN_NONE;
6558:   END_HANDLE_TH_ERRORS
6559: }
6560: 
```

- EN: The main execution path in this span is carried by `tensordot_out`, `wrap`, `THPVariable_tile`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `tensordot_out`, `wrap`, `THPVariable_tile` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6561-6640

```cpp
6561: // rot90
6562: static PyObject * THPVariable_rot90(PyObject* self_, PyObject* args, PyObject* kwargs)
6563: {
6564:   HANDLE_TH_ERRORS
6565:   static PythonArgParser parser({
6566:     "rot90(Tensor input, int64_t k=1, IntArrayRef dims={0,1})",
6567:   }, /*traceable=*/true);
6568: 
6569:   ParsedArgs<3> parsed_args;
6570:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6571:   if(_r.has_torch_function()) {
6572:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6573:   }
6574:   // aten::rot90(Tensor self, int k=1, int[] dims=[0,1]) -> Tensor
6575: 
6576:   auto dispatch_rot90 = [](const at::Tensor & self, int64_t k, at::IntArrayRef dims) -> at::Tensor {
6577:     pybind11::gil_scoped_release no_gil;
6578:     return self.rot90(k, dims);
6579:   };
6580:   return wrap(dispatch_rot90(_r.tensor(0), _r.toInt64(1), _r.intlist(2)));
6581:   Py_RETURN_NONE;
6582:   END_HANDLE_TH_ERRORS
6583: }
6584: 
6585: // _nested_from_padded
6586: static PyObject * THPVariable__nested_from_padded(PyObject* self_, PyObject* args, PyObject* kwargs)
6587: {
6588:   HANDLE_TH_ERRORS
6589:   static PythonArgParser parser({
6590:     "_nested_from_padded(Tensor padded, Tensor cpu_nested_shape_example, bool fuse_transform_0213=False)",
6591:   }, /*traceable=*/true);
6592: 
6593:   ParsedArgs<3> parsed_args;
6594:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6595:   if(_r.has_torch_function()) {
6596:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6597:   }
6598:   // aten::_nested_from_padded(Tensor padded, Tensor cpu_nested_shape_example, bool fuse_transform_0213=False) -> Tensor
6599: 
6600:   auto dispatch__nested_from_padded = [](const at::Tensor & padded, const at::Tensor & cpu_nested_shape_example, bool fuse_transform_0213) -> at::Tensor {
6601:     pybind11::gil_scoped_release no_gil;
6602:     return at::_nested_from_padded(padded, cpu_nested_shape_example, fuse_transform_0213);
6603:   };
6604:   return wrap(dispatch__nested_from_padded(_r.tensor(0), _r.tensor(1), _r.toBool(2)));
6605:   Py_RETURN_NONE;
6606:   END_HANDLE_TH_ERRORS
6607: }
6608: 
6609: // _nested_get_values
6610: static PyObject * THPVariable__nested_get_values(PyObject* self_, PyObject* args, PyObject* kwargs)
6611: {
6612:   HANDLE_TH_ERRORS
6613:   static PythonArgParser parser({
6614:     "_nested_get_values(Tensor input)",
6615:   }, /*traceable=*/true);
6616: 
6617:   ParsedArgs<1> parsed_args;
6618:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6619:   if(_r.has_torch_function()) {
6620:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6621:   }
6622:   // aten::_nested_get_values(Tensor(a) self) -> Tensor(a)
6623: 
6624:   auto dispatch__nested_get_values = [](const at::Tensor & self) -> at::Tensor {
6625:     pybind11::gil_scoped_release no_gil;
6626:     return at::_nested_get_values(self);
6627:   };
6628:   return wrap(dispatch__nested_get_values(_r.tensor(0)));
6629:   Py_RETURN_NONE;
6630:   END_HANDLE_TH_ERRORS
6631: }
6632: 
6633: // _nested_get_offsets
6634: static PyObject * THPVariable__nested_get_offsets(PyObject* self_, PyObject* args, PyObject* kwargs)
6635: {
6636:   HANDLE_TH_ERRORS
6637:   static PythonArgParser parser({
6638:     "_nested_get_offsets(Tensor input)",
6639:   }, /*traceable=*/true);
6640: 
```

- EN: The main execution path in this span is carried by `THPVariable_rot90`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_rot90`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6641-6720

```cpp
6641:   ParsedArgs<1> parsed_args;
6642:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6643:   if(_r.has_torch_function()) {
6644:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6645:   }
6646:   // aten::_nested_get_offsets(Tensor self) -> Tensor
6647: 
6648:   auto dispatch__nested_get_offsets = [](const at::Tensor & self) -> at::Tensor {
6649:     pybind11::gil_scoped_release no_gil;
6650:     return at::_nested_get_offsets(self);
6651:   };
6652:   return wrap(dispatch__nested_get_offsets(_r.tensor(0)));
6653:   Py_RETURN_NONE;
6654:   END_HANDLE_TH_ERRORS
6655: }
6656: 
6657: // _nested_get_jagged_dummy
6658: static PyObject * THPVariable__nested_get_jagged_dummy(PyObject* self_, PyObject* args, PyObject* kwargs)
6659: {
6660:   HANDLE_TH_ERRORS
6661:   static PythonArgParser parser({
6662:     "_nested_get_jagged_dummy(Tensor any)",
6663:   }, /*traceable=*/true);
6664: 
6665:   ParsedArgs<1> parsed_args;
6666:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6667:   if(_r.has_torch_function()) {
6668:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6669:   }
6670:   // aten::_nested_get_jagged_dummy(Tensor any) -> Tensor
6671: 
6672:   auto dispatch__nested_get_jagged_dummy = [](const at::Tensor & any) -> at::Tensor {
6673:     pybind11::gil_scoped_release no_gil;
6674:     return at::_nested_get_jagged_dummy(any);
6675:   };
6676:   return wrap(dispatch__nested_get_jagged_dummy(_r.tensor(0)));
6677:   Py_RETURN_NONE;
6678:   END_HANDLE_TH_ERRORS
6679: }
6680: 
6681: // triplet_margin_loss
6682: static PyObject * THPVariable_triplet_margin_loss(PyObject* self_, PyObject* args, PyObject* kwargs)
6683: {
6684:   HANDLE_TH_ERRORS
6685:   static PythonArgParser parser({
6686:     "triplet_margin_loss(Tensor anchor, Tensor positive, Tensor negative, double margin=1.0, double p=2, double eps=1e-06, bool swap=False, int64_t reduction=at::Reduction::Mean)",
6687:   }, /*traceable=*/true);
6688: 
6689:   ParsedArgs<8> parsed_args;
6690:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6691:   if(_r.has_torch_function()) {
6692:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6693:   }
6694:   // aten::triplet_margin_loss(Tensor anchor, Tensor positive, Tensor negative, float margin=1.0, float p=2, float eps=1e-06, bool swap=False, int reduction=Mean) -> Tensor
6695: 
6696:   auto dispatch_triplet_margin_loss = [](const at::Tensor & anchor, const at::Tensor & positive, const at::Tensor & negative, double margin, double p, double eps, bool swap, int64_t reduction) -> at::Tensor {
6697:     pybind11::gil_scoped_release no_gil;
6698:     return at::triplet_margin_loss(anchor, positive, negative, margin, p, eps, swap, reduction);
6699:   };
6700:   return wrap(dispatch_triplet_margin_loss(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toDouble(3), _r.toDouble(4), _r.toDouble(5), _r.toBool(6), _r.toInt64(7)));
6701:   Py_RETURN_NONE;
6702:   END_HANDLE_TH_ERRORS
6703: }
6704: 
6705: // fix
6706: static PyObject * THPVariable_fix(PyObject* self_, PyObject* args, PyObject* kwargs)
6707: {
6708:   HANDLE_TH_ERRORS
6709:   static PythonArgParser parser({
6710:     "fix(Tensor input, *, Tensor out=None)",
6711:   }, /*traceable=*/true);
6712: 
6713:   ParsedArgs<2> parsed_args;
6714:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6715:   if(_r.has_torch_function()) {
6716:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6717:   }
6718:   if (_r.isNone(1)) {
6719:     // aten::fix(Tensor self) -> Tensor
6720: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_nested_get_offsets`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_nested_get_offsets`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6721-6800

```cpp
6721:     auto dispatch_fix = [](const at::Tensor & self) -> at::Tensor {
6722:       pybind11::gil_scoped_release no_gil;
6723:       return self.fix();
6724:     };
6725:     return wrap(dispatch_fix(_r.tensor(0)));
6726:   } else {
6727:     // aten::fix.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
6728: 
6729:     auto dispatch_fix_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
6730:       pybind11::gil_scoped_release no_gil;
6731:       return at::fix_out(out, self);
6732:     };
6733:     return wrap(dispatch_fix_out(_r.tensor(1), _r.tensor(0)));
6734:   }
6735:   Py_RETURN_NONE;
6736:   END_HANDLE_TH_ERRORS
6737: }
6738: 
6739: // fix_
6740: static PyObject * THPVariable_fix_(PyObject* self_, PyObject* args, PyObject* kwargs)
6741: {
6742:   HANDLE_TH_ERRORS
6743:   static PythonArgParser parser({
6744:     "fix_(Tensor input)",
6745:   }, /*traceable=*/true);
6746: 
6747:   ParsedArgs<1> parsed_args;
6748:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6749:   if(_r.has_torch_function()) {
6750:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6751:   }
6752:   // aten::fix_(Tensor(a!) self) -> Tensor(a!)
6753: 
6754:   auto dispatch_fix_ = [](at::Tensor self) -> at::Tensor {
6755:     pybind11::gil_scoped_release no_gil;
6756:     return self.fix_();
6757:   };
6758:   return wrap(dispatch_fix_(_r.tensor(0)));
6759:   Py_RETURN_NONE;
6760:   END_HANDLE_TH_ERRORS
6761: }
6762: 
6763: // _has_compatible_shallow_copy_type
6764: static PyObject * THPVariable__has_compatible_shallow_copy_type(PyObject* self_, PyObject* args, PyObject* kwargs)
6765: {
6766:   HANDLE_TH_ERRORS
6767:   static PythonArgParser parser({
6768:     "_has_compatible_shallow_copy_type(Tensor input, Tensor from)",
6769:   }, /*traceable=*/false);
6770: 
6771:   ParsedArgs<2> parsed_args;
6772:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6773:   if(_r.has_torch_function()) {
6774:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6775:   }
6776:   // aten::_has_compatible_shallow_copy_type(Tensor self, Tensor from) -> bool
6777: 
6778:   auto dispatch__has_compatible_shallow_copy_type = [](const at::Tensor & self, const at::Tensor & from) -> bool {
6779:     pybind11::gil_scoped_release no_gil;
6780:     return at::_has_compatible_shallow_copy_type(self, from);
6781:   };
6782:   return wrap(dispatch__has_compatible_shallow_copy_type(_r.tensor(0), _r.tensor(1)));
6783:   Py_RETURN_NONE;
6784:   END_HANDLE_TH_ERRORS
6785: }
6786: 
6787: // _unique
6788: static PyObject * THPVariable__unique(PyObject* self_, PyObject* args, PyObject* kwargs)
6789: {
6790:   HANDLE_TH_ERRORS
6791:   static PythonArgParser parser({
6792:     "_unique(Tensor input, bool sorted=True, bool return_inverse=False)",
6793:   }, /*traceable=*/true);
6794: 
6795:   ParsedArgs<3> parsed_args;
6796:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6797:   if(_r.has_torch_function()) {
6798:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6799:   }
6800:   // aten::_unique(Tensor self, bool sorted=True, bool return_inverse=False) -> (Tensor, Tensor)
```

- EN: The main execution path in this span is carried by `wrap`, `fix_out`, `THPVariable_fix_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `fix_out`, `THPVariable_fix_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6801-6880

```cpp
6801: 
6802:   auto dispatch__unique = [](const at::Tensor & self, bool sorted, bool return_inverse) -> ::std::tuple<at::Tensor,at::Tensor> {
6803:     pybind11::gil_scoped_release no_gil;
6804:     return at::_unique(self, sorted, return_inverse);
6805:   };
6806:   return wrap(dispatch__unique(_r.tensor(0), _r.toBool(1), _r.toBool(2)));
6807:   Py_RETURN_NONE;
6808:   END_HANDLE_TH_ERRORS
6809: }
6810: 
6811: // vander
6812: static PyObject * THPVariable_vander(PyObject* self_, PyObject* args, PyObject* kwargs)
6813: {
6814:   HANDLE_TH_ERRORS
6815:   static PythonArgParser parser({
6816:     "vander(Tensor x, int64_t? N=None, bool increasing=False)",
6817:   }, /*traceable=*/true);
6818: 
6819:   ParsedArgs<3> parsed_args;
6820:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6821:   if(_r.has_torch_function()) {
6822:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6823:   }
6824:   // aten::vander(Tensor x, int? N=None, bool increasing=False) -> Tensor
6825: 
6826:   auto dispatch_vander = [](const at::Tensor & x, ::std::optional<int64_t> N, bool increasing) -> at::Tensor {
6827:     pybind11::gil_scoped_release no_gil;
6828:     return at::vander(x, N, increasing);
6829:   };
6830:   return wrap(dispatch_vander(_r.tensor(0), _r.toInt64Optional(1), _r.toBool(2)));
6831:   Py_RETURN_NONE;
6832:   END_HANDLE_TH_ERRORS
6833: }
6834: 
6835: \
6836: // var_mean
6837: static PyObject * THPVariable_var_mean(PyObject* self_, PyObject* args, PyObject* kwargs)
6838: {
6839:   HANDLE_TH_ERRORS
6840:   static PythonArgParser parser({
6841:     "var_mean(Tensor input, IntArrayRef[1]? dim, bool unbiased=True, bool keepdim=False)",
6842:     "var_mean(Tensor input, IntArrayRef[1]? dim=None, *, Scalar? correction=None, bool keepdim=False)",
6843:     "var_mean(Tensor input, bool unbiased=True)",
6844:     "var_mean(Tensor input, DimnameList[1] dim, bool unbiased=True, bool keepdim=False)",
6845:     "var_mean(Tensor input, DimnameList[1] dim, *, Scalar? correction=None, bool keepdim=False)",
6846:   }, /*traceable=*/true);
6847: 
6848:   ParsedArgs<4> parsed_args;
6849:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6850:   if(_r.has_torch_function()) {
6851:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6852:   }
6853:   switch (_r.idx) {
6854:     case 0: {
6855:       // aten::var_mean.dim(Tensor self, int[1]? dim, bool unbiased=True, bool keepdim=False) -> (Tensor, Tensor)
6856: 
6857:       auto dispatch_var_mean = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool unbiased, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
6858:         pybind11::gil_scoped_release no_gil;
6859:         return at::var_mean(self, dim, unbiased, keepdim);
6860:       };
6861:       return wrap(dispatch_var_mean(_r.tensor(0), _r.intlistOptional(1), _r.toBool(2), _r.toBool(3)));
6862:     }
6863:     case 1: {
6864:       // aten::var_mean.correction(Tensor self, int[1]? dim=None, *, Scalar? correction=None, bool keepdim=False) -> (Tensor, Tensor)
6865: 
6866:       auto dispatch_var_mean = [](const at::Tensor & self, at::OptionalIntArrayRef dim, const ::std::optional<at::Scalar> & correction, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
6867:         pybind11::gil_scoped_release no_gil;
6868:         return at::var_mean(self, dim, correction, keepdim);
6869:       };
6870:       return wrap(dispatch_var_mean(_r.tensor(0), _r.intlistOptional(1), _r.scalarOptional(2), _r.toBool(3)));
6871:     }
6872:     case 2: {
6873:       // aten::var_mean(Tensor self, bool unbiased=True) -> (Tensor, Tensor)
6874: 
6875:       auto dispatch_var_mean = [](const at::Tensor & self, bool unbiased) -> ::std::tuple<at::Tensor,at::Tensor> {
6876:         pybind11::gil_scoped_release no_gil;
6877:         return at::var_mean(self, unbiased);
6878:       };
6879:       return wrap(dispatch_var_mean(_r.tensor(0), _r.toBool(1)));
6880:     }
```

- EN: The main execution path in this span is carried by `_unique`, `wrap`, `THPVariable_vander`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_unique`, `wrap`, `THPVariable_vander` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6881-6960

```cpp
6881:     case 3: {
6882:       // aten::var_mean.names_dim(Tensor self, Dimname[1] dim, bool unbiased=True, bool keepdim=False) -> (Tensor, Tensor)
6883: 
6884:       auto dispatch_var_mean = [](const at::Tensor & self, at::DimnameList dim, bool unbiased, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
6885:         pybind11::gil_scoped_release no_gil;
6886:         return at::var_mean(self, dim, unbiased, keepdim);
6887:       };
6888:       return wrap(dispatch_var_mean(_r.tensor(0), _r.dimnamelist(1), _r.toBool(2), _r.toBool(3)));
6889:     }
6890:     case 4: {
6891:       // aten::var_mean.correction_names(Tensor self, Dimname[1] dim, *, Scalar? correction=None, bool keepdim=False) -> (Tensor, Tensor)
6892: 
6893:       auto dispatch_var_mean = [](const at::Tensor & self, at::DimnameList dim, const ::std::optional<at::Scalar> & correction, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
6894:         pybind11::gil_scoped_release no_gil;
6895:         return at::var_mean(self, dim, correction, keepdim);
6896:       };
6897:       return wrap(dispatch_var_mean(_r.tensor(0), _r.dimnamelist(1), _r.scalarOptional(2), _r.toBool(3)));
6898:     }
6899:   }
6900:   Py_RETURN_NONE;
6901:   END_HANDLE_TH_ERRORS
6902: }
6903: 
6904: // _weight_norm
6905: static PyObject * THPVariable__weight_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
6906: {
6907:   HANDLE_TH_ERRORS
6908:   static PythonArgParser parser({
6909:     "_weight_norm(Tensor v, Tensor g, int64_t dim=0)",
6910:   }, /*traceable=*/true);
6911: 
6912:   ParsedArgs<3> parsed_args;
6913:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6914:   if(_r.has_torch_function()) {
6915:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6916:   }
6917:   // aten::_weight_norm(Tensor v, Tensor g, int dim=0) -> Tensor
6918: 
6919:   auto dispatch__weight_norm = [](const at::Tensor & v, const at::Tensor & g, int64_t dim) -> at::Tensor {
6920:     pybind11::gil_scoped_release no_gil;
6921:     return at::_weight_norm(v, g, dim);
6922:   };
6923:   return wrap(dispatch__weight_norm(_r.tensor(0), _r.tensor(1), _r.toInt64(2)));
6924:   Py_RETURN_NONE;
6925:   END_HANDLE_TH_ERRORS
6926: }
6927: 
6928: // _standard_gamma_grad
6929: static PyObject * THPVariable__standard_gamma_grad(PyObject* self_, PyObject* args, PyObject* kwargs)
6930: {
6931:   HANDLE_TH_ERRORS
6932:   static PythonArgParser parser({
6933:     "_standard_gamma_grad(Tensor input, Tensor output)",
6934:   }, /*traceable=*/true);
6935: 
6936:   ParsedArgs<2> parsed_args;
6937:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6938:   if(_r.has_torch_function()) {
6939:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6940:   }
6941:   // aten::_standard_gamma_grad(Tensor self, Tensor output) -> Tensor
6942: 
6943:   auto dispatch__standard_gamma_grad = [](const at::Tensor & self, const at::Tensor & output) -> at::Tensor {
6944:     pybind11::gil_scoped_release no_gil;
6945:     return at::_standard_gamma_grad(self, output);
6946:   };
6947:   return wrap(dispatch__standard_gamma_grad(_r.tensor(0), _r.tensor(1)));
6948:   Py_RETURN_NONE;
6949:   END_HANDLE_TH_ERRORS
6950: }
6951: 
6952: // _philox_key_split
6953: static PyObject * THPVariable__philox_key_split(PyObject* self_, PyObject* args, PyObject* kwargs)
6954: {
6955:   HANDLE_TH_ERRORS
6956:   static PythonArgParser parser({
6957:     "_philox_key_split(Tensor key, int64_t num_splits)",
6958:   }, /*traceable=*/true);
6959: 
6960:   ParsedArgs<2> parsed_args;
```

- EN: The main execution path in this span is carried by `var_mean`, `wrap`, `THPVariable__weight_norm`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `var_mean`, `wrap`, `THPVariable__weight_norm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6961-7040

```cpp
6961:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6962:   if(_r.has_torch_function()) {
6963:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6964:   }
6965:   // aten::_philox_key_split(Tensor key, int num_splits) -> Tensor
6966: 
6967:   auto dispatch__philox_key_split = [](const at::Tensor & key, int64_t num_splits) -> at::Tensor {
6968:     pybind11::gil_scoped_release no_gil;
6969:     return at::_philox_key_split(key, num_splits);
6970:   };
6971:   return wrap(dispatch__philox_key_split(_r.tensor(0), _r.toInt64(1)));
6972:   Py_RETURN_NONE;
6973:   END_HANDLE_TH_ERRORS
6974: }
6975: 
6976: // poisson
6977: static PyObject * THPVariable_poisson(PyObject* self_, PyObject* args, PyObject* kwargs)
6978: {
6979:   HANDLE_TH_ERRORS
6980:   static PythonArgParser parser({
6981:     "poisson(Tensor input, Generator? generator=None)",
6982:   }, /*traceable=*/true);
6983: 
6984:   ParsedArgs<2> parsed_args;
6985:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
6986:   if(_r.has_torch_function()) {
6987:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
6988:   }
6989:   // aten::poisson(Tensor self, Generator? generator=None) -> Tensor
6990: 
6991:   auto dispatch_poisson = [](const at::Tensor & self, ::std::optional<at::Generator> generator) -> at::Tensor {
6992:     pybind11::gil_scoped_release no_gil;
6993:     return at::poisson(self, generator);
6994:   };
6995:   return wrap(dispatch_poisson(_r.tensor(0), _r.generator(1)));
6996:   Py_RETURN_NONE;
6997:   END_HANDLE_TH_ERRORS
6998: }
6999: 
7000: \
7001: // native_norm
7002: static PyObject * THPVariable_native_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
7003: {
7004:   HANDLE_TH_ERRORS
7005:   static PythonArgParser parser({
7006:     "native_norm(Tensor input, Scalar p=2)",
7007:     "native_norm(Tensor input, Scalar? p, IntArrayRef[1] dim, bool keepdim, ScalarType? dtype)",
7008:   }, /*traceable=*/true);
7009: 
7010:   ParsedArgs<5> parsed_args;
7011:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7012:   if(_r.has_torch_function()) {
7013:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7014:   }
7015:   switch (_r.idx) {
7016:     case 0: {
7017:       // aten::native_norm(Tensor self, Scalar p=2) -> Tensor
7018: 
7019:       auto dispatch_native_norm = [](const at::Tensor & self, const at::Scalar & p) -> at::Tensor {
7020:         pybind11::gil_scoped_release no_gil;
7021:         return at::native_norm(self, p);
7022:       };
7023:       return wrap(dispatch_native_norm(_r.tensor(0), _r.scalar(1)));
7024:     }
7025:     case 1: {
7026:       // aten::native_norm.ScalarOpt_dim_dtype(Tensor self, Scalar? p, int[1] dim, bool keepdim, ScalarType? dtype) -> Tensor
7027: 
7028:       auto dispatch_native_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
7029:         pybind11::gil_scoped_release no_gil;
7030:         return at::native_norm(self, p, dim, keepdim, dtype);
7031:       };
7032:       return wrap(dispatch_native_norm(_r.tensor(0), _r.scalarOptional(1), _r.intlist(2), _r.toBool(3), _r.scalartypeOptional(4)));
7033:     }
7034:   }
7035:   Py_RETURN_NONE;
7036:   END_HANDLE_TH_ERRORS
7037: }
7038: 
7039: // _sparse_softmax_backward_data
7040: static PyObject * THPVariable__sparse_softmax_backward_data(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_philox_key_split`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_philox_key_split`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7041-7120

```cpp
7041: {
7042:   HANDLE_TH_ERRORS
7043:   static PythonArgParser parser({
7044:     "_sparse_softmax_backward_data(Tensor grad_output, Tensor output, int64_t dim, Tensor input)",
7045:   }, /*traceable=*/true);
7046: 
7047:   ParsedArgs<4> parsed_args;
7048:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7049:   if(_r.has_torch_function()) {
7050:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7051:   }
7052:   // aten::_sparse_softmax_backward_data(Tensor grad_output, Tensor output, int dim, Tensor self) -> Tensor
7053: 
7054:   auto dispatch__sparse_softmax_backward_data = [](const at::Tensor & grad_output, const at::Tensor & output, int64_t dim, const at::Tensor & self) -> at::Tensor {
7055:     pybind11::gil_scoped_release no_gil;
7056:     return at::_sparse_softmax_backward_data(grad_output, output, dim, self);
7057:   };
7058:   return wrap(dispatch__sparse_softmax_backward_data(_r.tensor(0), _r.tensor(1), _r.toInt64(2), _r.tensor(3)));
7059:   Py_RETURN_NONE;
7060:   END_HANDLE_TH_ERRORS
7061: }
7062: 
7063: \
7064: // norm
7065: static PyObject * THPVariable_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
7066: {
7067:   HANDLE_TH_ERRORS
7068:   static PythonArgParser parser({
7069:     "norm(Tensor input, Scalar p=2)",
7070:     "norm(Tensor input, Scalar? p, *, ScalarType dtype)",
7071:     "norm(Tensor input, Scalar? p, IntArrayRef[1] dim, bool keepdim, *, ScalarType dtype, Tensor out=None)",
7072:     "norm(Tensor input, Scalar? p, IntArrayRef[1] dim, bool keepdim=False, *, Tensor out=None)",
7073:     "norm(Tensor input, Scalar? p, DimnameList[1] dim, bool keepdim, *, ScalarType dtype, Tensor out=None)",
7074:     "norm(Tensor input, Scalar? p, DimnameList[1] dim, bool keepdim=False, *, Tensor out=None)",
7075:   }, /*traceable=*/true);
7076: 
7077:   ParsedArgs<6> parsed_args;
7078:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7079:   if(_r.has_torch_function()) {
7080:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7081:   }
7082:   switch (_r.idx) {
7083:     case 0: {
7084:       // aten::norm.Scalar(Tensor self, Scalar p=2) -> Tensor
7085: 
7086:       auto dispatch_norm = [](const at::Tensor & self, const at::Scalar & p) -> at::Tensor {
7087:         pybind11::gil_scoped_release no_gil;
7088:         return self.norm(p);
7089:       };
7090:       return wrap(dispatch_norm(_r.tensor(0), _r.scalar(1)));
7091:     }
7092:     case 1: {
7093:       // aten::norm.ScalarOpt_dtype(Tensor self, Scalar? p, *, ScalarType dtype) -> Tensor
7094: 
7095:       auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::ScalarType dtype) -> at::Tensor {
7096:         pybind11::gil_scoped_release no_gil;
7097:         return self.norm(p, dtype);
7098:       };
7099:       return wrap(dispatch_norm(_r.tensor(0), _r.scalarOptional(1), _r.scalartype(2)));
7100:     }
7101:     case 2: {
7102:       if (_r.isNone(5)) {
7103:         // aten::norm.ScalarOpt_dim_dtype(Tensor self, Scalar? p, int[1] dim, bool keepdim, *, ScalarType dtype) -> Tensor
7104: 
7105:         auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim, at::ScalarType dtype) -> at::Tensor {
7106:           pybind11::gil_scoped_release no_gil;
7107:           return self.norm(p, dim, keepdim, dtype);
7108:         };
7109:         return wrap(dispatch_norm(_r.tensor(0), _r.scalarOptional(1), _r.intlist(2), _r.toBool(3), _r.scalartype(4)));
7110:       } else {
7111:         // aten::norm.dtype_out(Tensor self, Scalar? p, int[1] dim, bool keepdim, *, ScalarType dtype, Tensor(a!) out) -> Tensor(a!)
7112: 
7113:         auto dispatch_norm_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim, at::ScalarType dtype) -> at::Tensor {
7114:           pybind11::gil_scoped_release no_gil;
7115:           return at::norm_out(out, self, p, dim, keepdim, dtype);
7116:         };
7117:         return wrap(dispatch_norm_out(_r.tensor(5), _r.tensor(0), _r.scalarOptional(1), _r.intlist(2), _r.toBool(3), _r.scalartype(4)));
7118:       }
7119:     }
7120:     case 3: {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_sparse_softmax_backward_data`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_sparse_softmax_backward_data` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7121-7200

```cpp
7121:       if (_r.isNone(4)) {
7122:         // aten::norm.ScalarOpt_dim(Tensor self, Scalar? p, int[1] dim, bool keepdim=False) -> Tensor
7123: 
7124:         auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
7125:           pybind11::gil_scoped_release no_gil;
7126:           return self.norm(p, dim, keepdim);
7127:         };
7128:         return wrap(dispatch_norm(_r.tensor(0), _r.scalarOptional(1), _r.intlist(2), _r.toBool(3)));
7129:       } else {
7130:         // aten::norm.out(Tensor self, Scalar? p, int[1] dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
7131: 
7132:         auto dispatch_norm_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
7133:           pybind11::gil_scoped_release no_gil;
7134:           return at::norm_out(out, self, p, dim, keepdim);
7135:         };
7136:         return wrap(dispatch_norm_out(_r.tensor(4), _r.tensor(0), _r.scalarOptional(1), _r.intlist(2), _r.toBool(3)));
7137:       }
7138:     }
7139:     case 4: {
7140:       if (_r.isNone(5)) {
7141:         // aten::norm.names_ScalarOpt_dim_dtype(Tensor self, Scalar? p, Dimname[1] dim, bool keepdim, *, ScalarType dtype) -> Tensor
7142: 
7143:         auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::DimnameList dim, bool keepdim, at::ScalarType dtype) -> at::Tensor {
7144:           pybind11::gil_scoped_release no_gil;
7145:           return self.norm(p, dim, keepdim, dtype);
7146:         };
7147:         return wrap(dispatch_norm(_r.tensor(0), _r.scalarOptional(1), _r.dimnamelist(2), _r.toBool(3), _r.scalartype(4)));
7148:       } else {
7149:         // aten::norm.names_dtype_out(Tensor self, Scalar? p, Dimname[1] dim, bool keepdim, *, ScalarType dtype, Tensor(a!) out) -> Tensor(a!)
7150: 
7151:         auto dispatch_norm_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::DimnameList dim, bool keepdim, at::ScalarType dtype) -> at::Tensor {
7152:           pybind11::gil_scoped_release no_gil;
7153:           return at::norm_out(out, self, p, dim, keepdim, dtype);
7154:         };
7155:         return wrap(dispatch_norm_out(_r.tensor(5), _r.tensor(0), _r.scalarOptional(1), _r.dimnamelist(2), _r.toBool(3), _r.scalartype(4)));
7156:       }
7157:     }
7158:     case 5: {
7159:       if (_r.isNone(4)) {
7160:         // aten::norm.names_ScalarOpt_dim(Tensor self, Scalar? p, Dimname[1] dim, bool keepdim=False) -> Tensor
7161: 
7162:         auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::DimnameList dim, bool keepdim) -> at::Tensor {
7163:           pybind11::gil_scoped_release no_gil;
7164:           return self.norm(p, dim, keepdim);
7165:         };
7166:         return wrap(dispatch_norm(_r.tensor(0), _r.scalarOptional(1), _r.dimnamelist(2), _r.toBool(3)));
7167:       } else {
7168:         // aten::norm.names_out(Tensor self, Scalar? p, Dimname[1] dim, bool keepdim=False, *, Tensor(a!) out) -> Tensor(a!)
7169: 
7170:         auto dispatch_norm_out = [](at::Tensor out, const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::DimnameList dim, bool keepdim) -> at::Tensor {
7171:           pybind11::gil_scoped_release no_gil;
7172:           return at::norm_out(out, self, p, dim, keepdim);
7173:         };
7174:         return wrap(dispatch_norm_out(_r.tensor(4), _r.tensor(0), _r.scalarOptional(1), _r.dimnamelist(2), _r.toBool(3)));
7175:       }
7176:     }
7177:   }
7178:   Py_RETURN_NONE;
7179:   END_HANDLE_TH_ERRORS
7180: }
7181: 
7182: // frexp
7183: static PyObject * THPVariable_frexp(PyObject* self_, PyObject* args, PyObject* kwargs)
7184: {
7185:   HANDLE_TH_ERRORS
7186:   static PyTypeObject* NamedTuple = generated::get_frexp_structseq();
7187:   static PyTypeObject* NamedTuple1 = generated::get_frexp_out_structseq();
7188:   static PythonArgParser parser({
7189:     "frexp(Tensor input, *, TensorList[2] out=None)",
7190:   }, /*traceable=*/true);
7191: 
7192:   ParsedArgs<2> parsed_args;
7193:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7194:   if(_r.has_torch_function()) {
7195:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7196:   }
7197:   if (_r.isNone(1)) {
7198:     // aten::frexp.Tensor(Tensor self) -> (Tensor mantissa, Tensor exponent)
7199: 
7200:     auto dispatch_frexp = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
```

- EN: The main execution path in this span is carried by `wrap`, `norm_out`, `THPVariable_frexp`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `norm_out`, `THPVariable_frexp` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7201-7280

```cpp
7201:       pybind11::gil_scoped_release no_gil;
7202:       return self.frexp();
7203:     };
7204:     return wrap(NamedTuple, dispatch_frexp(_r.tensor(0)));
7205:   } else {
7206:     // aten::frexp.Tensor_out(Tensor self, *, Tensor(a!) mantissa, Tensor(b!) exponent) -> (Tensor(a!) mantissa, Tensor(b!) exponent)
7207:     auto out = _r.tensorlist_n<2>(1);
7208:     auto dispatch_frexp_out = [](at::Tensor & mantissa, at::Tensor & exponent, const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
7209:       pybind11::gil_scoped_release no_gil;
7210:       return at::frexp_out(mantissa, exponent, self);
7211:     };
7212:     return wrap(NamedTuple1, dispatch_frexp_out(out[0], out[1], _r.tensor(0)));
7213:   }
7214:   Py_RETURN_NONE;
7215:   END_HANDLE_TH_ERRORS
7216: }
7217: 
7218: // positive
7219: static PyObject * THPVariable_positive(PyObject* self_, PyObject* args, PyObject* kwargs)
7220: {
7221:   HANDLE_TH_ERRORS
7222:   static PythonArgParser parser({
7223:     "positive(Tensor input)",
7224:   }, /*traceable=*/true);
7225: 
7226:   ParsedArgs<1> parsed_args;
7227:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7228:   if(_r.has_torch_function()) {
7229:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7230:   }
7231:   // aten::positive(Tensor(a) self) -> Tensor(a)
7232: 
7233:   auto dispatch_positive = [](const at::Tensor & self) -> at::Tensor {
7234:     pybind11::gil_scoped_release no_gil;
7235:     return self.positive();
7236:   };
7237:   return wrap(dispatch_positive(_r.tensor(0)));
7238:   Py_RETURN_NONE;
7239:   END_HANDLE_TH_ERRORS
7240: }
7241: 
7242: // resize_as_
7243: static PyObject * THPVariable_resize_as_(PyObject* self_, PyObject* args, PyObject* kwargs)
7244: {
7245:   HANDLE_TH_ERRORS
7246:   static PythonArgParser parser({
7247:     "resize_as_(Tensor input, Tensor the_template, *, MemoryFormat? memory_format=None)",
7248:   }, /*traceable=*/true);
7249: 
7250:   ParsedArgs<3> parsed_args;
7251:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7252:   if(_r.has_torch_function()) {
7253:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7254:   }
7255:   // aten::resize_as_(Tensor(a!) self, Tensor the_template, *, MemoryFormat? memory_format=None) -> Tensor(a!)
7256: 
7257:   auto dispatch_resize_as_ = [](const at::Tensor & self, const at::Tensor & the_template, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
7258:     pybind11::gil_scoped_release no_gil;
7259:     return self.resize_as_(the_template, memory_format);
7260:   };
7261:   return wrap(dispatch_resize_as_(_r.tensor(0), _r.tensor(1), _r.memoryformatOptional(2)));
7262:   Py_RETURN_NONE;
7263:   END_HANDLE_TH_ERRORS
7264: }
7265: 
7266: // zero_
7267: static PyObject * THPVariable_zero_(PyObject* self_, PyObject* args, PyObject* kwargs)
7268: {
7269:   HANDLE_TH_ERRORS
7270:   static PythonArgParser parser({
7271:     "zero_(Tensor input)",
7272:   }, /*traceable=*/true);
7273: 
7274:   ParsedArgs<1> parsed_args;
7275:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7276:   if(_r.has_torch_function()) {
7277:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7278:   }
7279:   // aten::zero_(Tensor(a!) self) -> Tensor(a!)
7280: 
```

- EN: The main execution path in this span is carried by `wrap`, `frexp_out`, `THPVariable_positive`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `frexp_out`, `THPVariable_positive` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7281-7360

```cpp
7281:   auto dispatch_zero_ = [](at::Tensor self) -> at::Tensor {
7282:     pybind11::gil_scoped_release no_gil;
7283:     return self.zero_();
7284:   };
7285:   return wrap(dispatch_zero_(_r.tensor(0)));
7286:   Py_RETURN_NONE;
7287:   END_HANDLE_TH_ERRORS
7288: }
7289: 
7290: \
7291: // sub
7292: static PyObject * THPVariable_sub(PyObject* self_, PyObject* args, PyObject* kwargs)
7293: {
7294:   HANDLE_TH_ERRORS
7295:   static PythonArgParser parser({
7296:     "sub(Tensor input, Scalar alpha, Tensor other, *, Tensor out=None)|deprecated",
7297:     "sub(Tensor input, Tensor other, *, Scalar alpha=1, Tensor out=None)",
7298:   }, /*traceable=*/true);
7299: 
7300:   ParsedArgs<4> parsed_args;
7301:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7302:   if(_r.has_torch_function()) {
7303:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7304:   }
7305:   switch (_r.idx) {
7306:     case 0: {
7307:       if (_r.isNone(3)) {
7308:         // [deprecated] aten::sub(Tensor self, Scalar alpha, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
7309: 
7310:         auto dispatch_sub = [](at::Tensor out, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
7311:           pybind11::gil_scoped_release no_gil;
7312:           return self.sub(other, alpha);
7313:         };
7314:         return wrap(dispatch_sub(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.tensor(2)));
7315:       } else {
7316:         // [deprecated] aten::sub(Tensor self, Scalar alpha, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
7317: 
7318:         auto dispatch_sub_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
7319:           pybind11::gil_scoped_release no_gil;
7320:           return at::sub_out(out, self, other, alpha);
7321:         };
7322:         return wrap(dispatch_sub_out(_r.tensor(3), _r.tensor(0), _r.scalar(1), _r.tensor(2)));
7323:       }
7324:     }
7325:     case 1: {
7326:       if (_r.isNone(3)) {
7327:         // aten::sub.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
7328: 
7329:         auto dispatch_sub = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
7330:           pybind11::gil_scoped_release no_gil;
7331:           return self.sub(other, alpha);
7332:         };
7333:         return wrap(dispatch_sub(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
7334:       } else {
7335:         // aten::sub.out(Tensor self, Tensor other, *, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
7336: 
7337:         auto dispatch_sub_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
7338:           pybind11::gil_scoped_release no_gil;
7339:           return at::sub_out(out, self, other, alpha);
7340:         };
7341:         return wrap(dispatch_sub_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.scalar(2)));
7342:       }
7343:     }
7344:   }
7345:   Py_RETURN_NONE;
7346:   END_HANDLE_TH_ERRORS
7347: }
7348: 
7349: \
7350: // subtract
7351: static PyObject * THPVariable_subtract(PyObject* self_, PyObject* args, PyObject* kwargs)
7352: {
7353:   HANDLE_TH_ERRORS
7354:   static PythonArgParser parser({
7355:     "subtract(Tensor input, Tensor other, *, Scalar alpha=1, Tensor out=None)",
7356:     "subtract(Tensor input, Scalar other, Scalar alpha=1)",
7357:   }, /*traceable=*/true);
7358: 
7359:   ParsedArgs<4> parsed_args;
7360:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_sub`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_sub`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7361-7440

```cpp
7361:   if(_r.has_torch_function()) {
7362:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7363:   }
7364:   switch (_r.idx) {
7365:     case 0: {
7366:       if (_r.isNone(3)) {
7367:         // aten::subtract.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
7368: 
7369:         auto dispatch_subtract = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
7370:           pybind11::gil_scoped_release no_gil;
7371:           return self.subtract(other, alpha);
7372:         };
7373:         return wrap(dispatch_subtract(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
7374:       } else {
7375:         // aten::subtract.out(Tensor self, Tensor other, *, Scalar alpha=1, Tensor(a!) out) -> Tensor(a!)
7376: 
7377:         auto dispatch_subtract_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
7378:           pybind11::gil_scoped_release no_gil;
7379:           return at::subtract_out(out, self, other, alpha);
7380:         };
7381:         return wrap(dispatch_subtract_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.scalar(2)));
7382:       }
7383:     }
7384:     case 1: {
7385:       // aten::subtract.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> Tensor
7386: 
7387:       auto dispatch_subtract = [](const at::Tensor & self, const at::Scalar & other, const at::Scalar & alpha) -> at::Tensor {
7388:         pybind11::gil_scoped_release no_gil;
7389:         return self.subtract(other, alpha);
7390:       };
7391:       return wrap(dispatch_subtract(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
7392:     }
7393:   }
7394:   Py_RETURN_NONE;
7395:   END_HANDLE_TH_ERRORS
7396: }
7397: 
7398: \
7399: // rsub
7400: static PyObject * THPVariable_rsub(PyObject* self_, PyObject* args, PyObject* kwargs)
7401: {
7402:   HANDLE_TH_ERRORS
7403:   static PythonArgParser parser({
7404:     "rsub(Tensor input, Tensor other, *, Scalar alpha=1)",
7405:     "rsub(Tensor input, Scalar other, Scalar alpha=1)",
7406:   }, /*traceable=*/true);
7407: 
7408:   ParsedArgs<3> parsed_args;
7409:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7410:   if(_r.has_torch_function()) {
7411:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7412:   }
7413:   switch (_r.idx) {
7414:     case 0: {
7415:       // aten::rsub.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
7416: 
7417:       auto dispatch_rsub = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
7418:         pybind11::gil_scoped_release no_gil;
7419:         return at::rsub(self, other, alpha);
7420:       };
7421:       return wrap(dispatch_rsub(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
7422:     }
7423:     case 1: {
7424:       // aten::rsub.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> Tensor
7425: 
7426:       auto dispatch_rsub = [](const at::Tensor & self, const at::Scalar & other, const at::Scalar & alpha) -> at::Tensor {
7427:         pybind11::gil_scoped_release no_gil;
7428:         return at::rsub(self, other, alpha);
7429:       };
7430:       return wrap(dispatch_rsub(_r.tensor(0), _r.scalar(1), _r.scalar(2)));
7431:     }
7432:   }
7433:   Py_RETURN_NONE;
7434:   END_HANDLE_TH_ERRORS
7435: }
7436: 
7437: // _validate_sparse_csc_tensor_args
7438: static PyObject * THPVariable__validate_sparse_csc_tensor_args(PyObject* self_, PyObject* args, PyObject* kwargs)
7439: {
7440:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `subtract_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `subtract_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7441-7520

```cpp
7441:   static PythonArgParser parser({
7442:     "_validate_sparse_csc_tensor_args(Tensor ccol_indices, Tensor row_indices, Tensor values, IntArrayRef size, bool? check_pinning=None)",
7443:   }, /*traceable=*/false);
7444: 
7445:   ParsedArgs<5> parsed_args;
7446:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7447:   if(_r.has_torch_function()) {
7448:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7449:   }
7450:   // aten::_validate_sparse_csc_tensor_args(Tensor ccol_indices, Tensor row_indices, Tensor values, int[] size, bool? check_pinning=None) -> ()
7451: 
7452:   auto dispatch__validate_sparse_csc_tensor_args = [](const at::Tensor & ccol_indices, const at::Tensor & row_indices, const at::Tensor & values, at::IntArrayRef size, ::std::optional<bool> check_pinning) -> void {
7453:     pybind11::gil_scoped_release no_gil;
7454:     at::_validate_sparse_csc_tensor_args(ccol_indices, row_indices, values, size, check_pinning);
7455:   };
7456:   dispatch__validate_sparse_csc_tensor_args(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.intlist(3), _r.toBoolOptional(4));
7457:   Py_RETURN_NONE;
7458:   Py_RETURN_NONE;
7459:   END_HANDLE_TH_ERRORS
7460: }
7461: 
7462: // _coalesce
7463: static PyObject * THPVariable__coalesce(PyObject* self_, PyObject* args, PyObject* kwargs)
7464: {
7465:   HANDLE_TH_ERRORS
7466:   static PythonArgParser parser({
7467:     "_coalesce(Tensor input)",
7468:   }, /*traceable=*/true);
7469: 
7470:   ParsedArgs<1> parsed_args;
7471:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7472:   if(_r.has_torch_function()) {
7473:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7474:   }
7475:   // aten::_coalesce(Tensor self) -> Tensor
7476: 
7477:   auto dispatch__coalesce = [](const at::Tensor & self) -> at::Tensor {
7478:     pybind11::gil_scoped_release no_gil;
7479:     return at::_coalesce(self);
7480:   };
7481:   return wrap(dispatch__coalesce(_r.tensor(0)));
7482:   Py_RETURN_NONE;
7483:   END_HANDLE_TH_ERRORS
7484: }
7485: 
7486: \
7487: // unbind
7488: static PyObject * THPVariable_unbind(PyObject* self_, PyObject* args, PyObject* kwargs)
7489: {
7490:   HANDLE_TH_ERRORS
7491:   static PythonArgParser parser({
7492:     "unbind(Tensor input, int64_t dim=0)",
7493:     "unbind(Tensor input, Dimname dim)",
7494:   }, /*traceable=*/true);
7495: 
7496:   ParsedArgs<2> parsed_args;
7497:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7498:   if(_r.has_torch_function()) {
7499:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7500:   }
7501:   switch (_r.idx) {
7502:     case 0: {
7503:       // aten::unbind.int(Tensor(a -> *) self, int dim=0) -> Tensor(a)[]
7504: 
7505:       auto dispatch_unbind = [](const at::Tensor & self, int64_t dim) -> ::std::vector<at::Tensor> {
7506:         pybind11::gil_scoped_release no_gil;
7507:         return self.unbind(dim);
7508:       };
7509:       return wrap(dispatch_unbind(_r.tensor(0), _r.toInt64(1)));
7510:     }
7511:     case 1: {
7512:       // aten::unbind.Dimname(Tensor(a -> *) self, Dimname dim) -> Tensor(a)[]
7513: 
7514:       auto dispatch_unbind = [](const at::Tensor & self, at::Dimname dim) -> ::std::vector<at::Tensor> {
7515:         pybind11::gil_scoped_release no_gil;
7516:         return self.unbind(dim);
7517:       };
7518:       return wrap(dispatch_unbind(_r.tensor(0), _r.dimname(1)));
7519:     }
7520:   }
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_validate_sparse_csc_tensor_args`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_validate_sparse_csc_tensor_args` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7521-7600

```cpp
7521:   Py_RETURN_NONE;
7522:   END_HANDLE_TH_ERRORS
7523: }
7524: 
7525: // quantize_per_tensor_dynamic
7526: static PyObject * THPVariable_quantize_per_tensor_dynamic(PyObject* self_, PyObject* args, PyObject* kwargs)
7527: {
7528:   HANDLE_TH_ERRORS
7529:   static PythonArgParser parser({
7530:     "quantize_per_tensor_dynamic(Tensor input, ScalarType dtype, bool reduce_range)",
7531:   }, /*traceable=*/true);
7532: 
7533:   ParsedArgs<3> parsed_args;
7534:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7535:   if(_r.has_torch_function()) {
7536:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7537:   }
7538:   // aten::quantize_per_tensor_dynamic(Tensor self, ScalarType dtype, bool reduce_range) -> Tensor
7539: 
7540:   auto dispatch_quantize_per_tensor_dynamic = [](const at::Tensor & self, at::ScalarType dtype, bool reduce_range) -> at::Tensor {
7541:     pybind11::gil_scoped_release no_gil;
7542:     return at::quantize_per_tensor_dynamic(self, dtype, reduce_range);
7543:   };
7544:   return wrap(dispatch_quantize_per_tensor_dynamic(_r.tensor(0), _r.scalartype(1), _r.toBool(2)));
7545:   Py_RETURN_NONE;
7546:   END_HANDLE_TH_ERRORS
7547: }
7548: 
7549: \
7550: // quantize_per_tensor
7551: static PyObject * THPVariable_quantize_per_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
7552: {
7553:   HANDLE_TH_ERRORS
7554:   static PythonArgParser parser({
7555:     "quantize_per_tensor(Tensor input, Tensor scale, Tensor zero_point, ScalarType dtype)",
7556:     "quantize_per_tensor(Tensor input, double scale, int64_t zero_point, ScalarType dtype)",
7557:     "quantize_per_tensor(TensorList tensors, Tensor scales, Tensor zero_points, ScalarType dtype)",
7558:   }, /*traceable=*/true);
7559: 
7560:   ParsedArgs<4> parsed_args;
7561:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7562:   if(_r.has_torch_function()) {
7563:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7564:   }
7565:   switch (_r.idx) {
7566:     case 0: {
7567:       // aten::quantize_per_tensor.tensor_qparams(Tensor self, Tensor scale, Tensor zero_point, ScalarType dtype) -> Tensor
7568: 
7569:       auto dispatch_quantize_per_tensor = [](const at::Tensor & self, const at::Tensor & scale, const at::Tensor & zero_point, at::ScalarType dtype) -> at::Tensor {
7570:         pybind11::gil_scoped_release no_gil;
7571:         return at::quantize_per_tensor(self, scale, zero_point, dtype);
7572:       };
7573:       return wrap(dispatch_quantize_per_tensor(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalartype(3)));
7574:     }
7575:     case 1: {
7576:       // aten::quantize_per_tensor(Tensor self, float scale, int zero_point, ScalarType dtype) -> Tensor
7577: 
7578:       auto dispatch_quantize_per_tensor = [](const at::Tensor & self, double scale, int64_t zero_point, at::ScalarType dtype) -> at::Tensor {
7579:         pybind11::gil_scoped_release no_gil;
7580:         return at::quantize_per_tensor(self, scale, zero_point, dtype);
7581:       };
7582:       return wrap(dispatch_quantize_per_tensor(_r.tensor(0), _r.toDouble(1), _r.toInt64(2), _r.scalartype(3)));
7583:     }
7584:     case 2: {
7585:       // aten::quantize_per_tensor.tensors(Tensor[] tensors, Tensor scales, Tensor zero_points, ScalarType dtype) -> Tensor[]
7586: 
7587:       auto dispatch_quantize_per_tensor = [](at::TensorList tensors, const at::Tensor & scales, const at::Tensor & zero_points, at::ScalarType dtype) -> ::std::vector<at::Tensor> {
7588:         pybind11::gil_scoped_release no_gil;
7589:         return at::quantize_per_tensor(tensors, scales, zero_points, dtype);
7590:       };
7591:       return wrap(dispatch_quantize_per_tensor(_r.tensorlist(0), _r.tensor(1), _r.tensor(2), _r.scalartype(3)));
7592:     }
7593:   }
7594:   Py_RETURN_NONE;
7595:   END_HANDLE_TH_ERRORS
7596: }
7597: 
7598: // quantize_per_channel
7599: static PyObject * THPVariable_quantize_per_channel(PyObject* self_, PyObject* args, PyObject* kwargs)
7600: {
```

- EN: The main execution path in this span is carried by `THPVariable_quantize_per_tensor_dynamic`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_quantize_per_tensor_dynamic`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7601-7680

```cpp
7601:   HANDLE_TH_ERRORS
7602:   static PythonArgParser parser({
7603:     "quantize_per_channel(Tensor input, Tensor scales, Tensor zero_points, int64_t axis, ScalarType dtype)",
7604:   }, /*traceable=*/true);
7605: 
7606:   ParsedArgs<5> parsed_args;
7607:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7608:   if(_r.has_torch_function()) {
7609:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7610:   }
7611:   // aten::quantize_per_channel(Tensor self, Tensor scales, Tensor zero_points, int axis, ScalarType dtype) -> Tensor
7612: 
7613:   auto dispatch_quantize_per_channel = [](const at::Tensor & self, const at::Tensor & scales, const at::Tensor & zero_points, int64_t axis, at::ScalarType dtype) -> at::Tensor {
7614:     pybind11::gil_scoped_release no_gil;
7615:     return at::quantize_per_channel(self, scales, zero_points, axis, dtype);
7616:   };
7617:   return wrap(dispatch_quantize_per_channel(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3), _r.scalartype(4)));
7618:   Py_RETURN_NONE;
7619:   END_HANDLE_TH_ERRORS
7620: }
7621: 
7622: // q_zero_point
7623: static PyObject * THPVariable_q_zero_point(PyObject* self_, PyObject* args, PyObject* kwargs)
7624: {
7625:   HANDLE_TH_ERRORS
7626:   static PythonArgParser parser({
7627:     "q_zero_point(Tensor input)",
7628:   }, /*traceable=*/false);
7629: 
7630:   ParsedArgs<1> parsed_args;
7631:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7632:   if(_r.has_torch_function()) {
7633:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7634:   }
7635:   // aten::q_zero_point(Tensor self) -> int
7636: 
7637:   auto dispatch_q_zero_point = [](const at::Tensor & self) -> int64_t {
7638:     pybind11::gil_scoped_release no_gil;
7639:     return self.q_zero_point();
7640:   };
7641:   return wrap(dispatch_q_zero_point(_r.tensor(0)));
7642:   Py_RETURN_NONE;
7643:   END_HANDLE_TH_ERRORS
7644: }
7645: 
7646: // int_repr
7647: static PyObject * THPVariable_int_repr(PyObject* self_, PyObject* args, PyObject* kwargs)
7648: {
7649:   HANDLE_TH_ERRORS
7650:   static PythonArgParser parser({
7651:     "int_repr(Tensor input)",
7652:   }, /*traceable=*/true);
7653: 
7654:   ParsedArgs<1> parsed_args;
7655:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7656:   if(_r.has_torch_function()) {
7657:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7658:   }
7659:   // aten::int_repr(Tensor self) -> Tensor
7660: 
7661:   auto dispatch_int_repr = [](const at::Tensor & self) -> at::Tensor {
7662:     pybind11::gil_scoped_release no_gil;
7663:     return self.int_repr();
7664:   };
7665:   return wrap(dispatch_int_repr(_r.tensor(0)));
7666:   Py_RETURN_NONE;
7667:   END_HANDLE_TH_ERRORS
7668: }
7669: 
7670: // _make_per_tensor_quantized_tensor
7671: static PyObject * THPVariable__make_per_tensor_quantized_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
7672: {
7673:   HANDLE_TH_ERRORS
7674:   static PythonArgParser parser({
7675:     "_make_per_tensor_quantized_tensor(Tensor input, double scale, int64_t zero_point)",
7676:   }, /*traceable=*/true);
7677: 
7678:   ParsedArgs<3> parsed_args;
7679:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7680:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `quantize_per_channel`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `quantize_per_channel` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7681-7760

```cpp
7681:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7682:   }
7683:   // aten::_make_per_tensor_quantized_tensor(Tensor self, float scale, int zero_point) -> Tensor
7684: 
7685:   auto dispatch__make_per_tensor_quantized_tensor = [](const at::Tensor & self, double scale, int64_t zero_point) -> at::Tensor {
7686:     pybind11::gil_scoped_release no_gil;
7687:     return at::_make_per_tensor_quantized_tensor(self, scale, zero_point);
7688:   };
7689:   return wrap(dispatch__make_per_tensor_quantized_tensor(_r.tensor(0), _r.toDouble(1), _r.toInt64(2)));
7690:   Py_RETURN_NONE;
7691:   END_HANDLE_TH_ERRORS
7692: }
7693: 
7694: // _make_per_channel_quantized_tensor
7695: static PyObject * THPVariable__make_per_channel_quantized_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
7696: {
7697:   HANDLE_TH_ERRORS
7698:   static PythonArgParser parser({
7699:     "_make_per_channel_quantized_tensor(Tensor input, Tensor scale, Tensor zero_point, int64_t axis)",
7700:   }, /*traceable=*/true);
7701: 
7702:   ParsedArgs<4> parsed_args;
7703:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7704:   if(_r.has_torch_function()) {
7705:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7706:   }
7707:   // aten::_make_per_channel_quantized_tensor(Tensor self, Tensor scale, Tensor zero_point, int axis) -> Tensor
7708: 
7709:   auto dispatch__make_per_channel_quantized_tensor = [](const at::Tensor & self, const at::Tensor & scale, const at::Tensor & zero_point, int64_t axis) -> at::Tensor {
7710:     pybind11::gil_scoped_release no_gil;
7711:     return at::_make_per_channel_quantized_tensor(self, scale, zero_point, axis);
7712:   };
7713:   return wrap(dispatch__make_per_channel_quantized_tensor(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3)));
7714:   Py_RETURN_NONE;
7715:   END_HANDLE_TH_ERRORS
7716: }
7717: 
7718: // fake_quantize_per_channel_affine
7719: static PyObject * THPVariable_fake_quantize_per_channel_affine(PyObject* self_, PyObject* args, PyObject* kwargs)
7720: {
7721:   HANDLE_TH_ERRORS
7722:   static PythonArgParser parser({
7723:     "fake_quantize_per_channel_affine(Tensor input, Tensor scale, Tensor zero_point, int64_t axis, int64_t quant_min, int64_t quant_max)",
7724:   }, /*traceable=*/true);
7725: 
7726:   ParsedArgs<6> parsed_args;
7727:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7728:   if(_r.has_torch_function()) {
7729:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7730:   }
7731:   // aten::fake_quantize_per_channel_affine(Tensor self, Tensor scale, Tensor zero_point, int axis, int quant_min, int quant_max) -> Tensor
7732: 
7733:   auto dispatch_fake_quantize_per_channel_affine = [](const at::Tensor & self, const at::Tensor & scale, const at::Tensor & zero_point, int64_t axis, int64_t quant_min, int64_t quant_max) -> at::Tensor {
7734:     pybind11::gil_scoped_release no_gil;
7735:     return at::fake_quantize_per_channel_affine(self, scale, zero_point, axis, quant_min, quant_max);
7736:   };
7737:   return wrap(dispatch_fake_quantize_per_channel_affine(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toInt64(3), _r.toInt64(4), _r.toInt64(5)));
7738:   Py_RETURN_NONE;
7739:   END_HANDLE_TH_ERRORS
7740: }
7741: 
7742: // _fused_moving_avg_obs_fq_helper
7743: static PyObject * THPVariable__fused_moving_avg_obs_fq_helper(PyObject* self_, PyObject* args, PyObject* kwargs)
7744: {
7745:   HANDLE_TH_ERRORS
7746:   static PyTypeObject* NamedTuple = generated::get__fused_moving_avg_obs_fq_helper_structseq();
7747:   static PythonArgParser parser({
7748:     "_fused_moving_avg_obs_fq_helper(Tensor input, Tensor observer_on, Tensor fake_quant_on, Tensor running_min, Tensor running_max, Tensor scale, Tensor zero_point, double averaging_const, int64_t quant_min, int64_t quant_max, int64_t ch_axis, bool per_row_fake_quant=False, bool symmetric_quant=False)",
7749:   }, /*traceable=*/true);
7750: 
7751:   ParsedArgs<13> parsed_args;
7752:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7753:   if(_r.has_torch_function()) {
7754:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7755:   }
7756:   // aten::_fused_moving_avg_obs_fq_helper(Tensor self, Tensor observer_on, Tensor fake_quant_on, Tensor(a!) running_min, Tensor(b!) running_max, Tensor(c!) scale, Tensor(d!) zero_point, float averaging_const, int quant_min, int quant_max, int ch_axis, bool per_row_fake_quant=False, bool symmetric_quant=False) -> (Tensor output, Tensor mask)
7757: 
7758:   auto dispatch__fused_moving_avg_obs_fq_helper = [](const at::Tensor & self, const at::Tensor & observer_on, const at::Tensor & fake_quant_on, at::Tensor running_min, at::Tensor running_max, at::Tensor scale, at::Tensor zero_point, double averaging_const, int64_t quant_min, int64_t quant_max, int64_t ch_axis, bool per_row_fake_quant, bool symmetric_quant) -> ::std::tuple<at::Tensor,at::Tensor> {
7759:     pybind11::gil_scoped_release no_gil;
7760:     return at::_fused_moving_avg_obs_fq_helper(self, observer_on, fake_quant_on, running_min, running_max, scale, zero_point, averaging_const, quant_min, quant_max, ch_axis, per_row_fake_quant, symmetric_quant);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_make_per_tensor_quantized_tensor`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_make_per_tensor_quantized_tensor`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7761-7840

```cpp
7761:   };
7762:   return wrap(NamedTuple, dispatch__fused_moving_avg_obs_fq_helper(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.tensor(4), _r.tensor(5), _r.tensor(6), _r.toDouble(7), _r.toInt64(8), _r.toInt64(9), _r.toInt64(10), _r.toBool(11), _r.toBool(12)));
7763:   Py_RETURN_NONE;
7764:   END_HANDLE_TH_ERRORS
7765: }
7766: 
7767: // _choose_qparams_per_tensor
7768: static PyObject * THPVariable__choose_qparams_per_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
7769: {
7770:   HANDLE_TH_ERRORS
7771:   static PythonArgParser parser({
7772:     "_choose_qparams_per_tensor(Tensor input, bool reduce_range=False)",
7773:   }, /*traceable=*/false);
7774: 
7775:   ParsedArgs<2> parsed_args;
7776:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7777:   if(_r.has_torch_function()) {
7778:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7779:   }
7780:   // aten::_choose_qparams_per_tensor(Tensor self, bool reduce_range=False) -> (float, int)
7781: 
7782:   auto dispatch__choose_qparams_per_tensor = [](const at::Tensor & self, bool reduce_range) -> ::std::tuple<double,int64_t> {
7783:     pybind11::gil_scoped_release no_gil;
7784:     return at::_choose_qparams_per_tensor(self, reduce_range);
7785:   };
7786:   return wrap(dispatch__choose_qparams_per_tensor(_r.tensor(0), _r.toBool(1)));
7787:   Py_RETURN_NONE;
7788:   END_HANDLE_TH_ERRORS
7789: }
7790: 
7791: \
7792: // meshgrid
7793: static PyObject * THPVariable_meshgrid(PyObject* self_, PyObject* args, PyObject* kwargs)
7794: {
7795:   HANDLE_TH_ERRORS
7796:   static PythonArgParser parser({
7797:     "meshgrid(TensorList tensors)",
7798:     "meshgrid(TensorList tensors, *, c10::string_view indexing)",
7799:   }, /*traceable=*/true);
7800: 
7801:   ParsedArgs<2> parsed_args;
7802:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7803:   if(_r.has_torch_function()) {
7804:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7805:   }
7806:   switch (_r.idx) {
7807:     case 0: {
7808:       // aten::meshgrid(Tensor[] tensors) -> Tensor[]
7809: 
7810:       auto dispatch_meshgrid = [](at::TensorList tensors) -> ::std::vector<at::Tensor> {
7811:         pybind11::gil_scoped_release no_gil;
7812:         return at::meshgrid(tensors);
7813:       };
7814:       return wrap(dispatch_meshgrid(_r.tensorlist(0)));
7815:     }
7816:     case 1: {
7817:       // aten::meshgrid.indexing(Tensor[] tensors, *, str indexing) -> Tensor[]
7818: 
7819:       auto dispatch_meshgrid = [](at::TensorList tensors, c10::string_view indexing) -> ::std::vector<at::Tensor> {
7820:         pybind11::gil_scoped_release no_gil;
7821:         return at::meshgrid(tensors, indexing);
7822:       };
7823:       return wrap(dispatch_meshgrid(_r.tensorlist(0), _r.stringView(1)));
7824:     }
7825:   }
7826:   Py_RETURN_NONE;
7827:   END_HANDLE_TH_ERRORS
7828: }
7829: 
7830: // promote_types
7831: static PyObject * THPVariable_promote_types(PyObject* self_, PyObject* args, PyObject* kwargs)
7832: {
7833:   HANDLE_TH_ERRORS
7834:   static PythonArgParser parser({
7835:     "promote_types(ScalarType type1, ScalarType type2)",
7836:   }, /*traceable=*/false);
7837: 
7838:   ParsedArgs<2> parsed_args;
7839:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7840:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__choose_qparams_per_tensor`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__choose_qparams_per_tensor`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7841-7920

```cpp
7841:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7842:   }
7843:   // aten::promote_types(ScalarType type1, ScalarType type2) -> ScalarType
7844: 
7845:   auto dispatch_promote_types = [](at::ScalarType type1, at::ScalarType type2) -> at::ScalarType {
7846:     pybind11::gil_scoped_release no_gil;
7847:     return at::promote_types(type1, type2);
7848:   };
7849:   return wrap(dispatch_promote_types(_r.scalartype(0), _r.scalartype(1)));
7850:   Py_RETURN_NONE;
7851:   END_HANDLE_TH_ERRORS
7852: }
7853: 
7854: \
7855: // gru
7856: static PyObject * THPVariable_gru(PyObject* self_, PyObject* args, PyObject* kwargs)
7857: {
7858:   HANDLE_TH_ERRORS
7859:   static PythonArgParser parser({
7860:     "gru(Tensor data, Tensor batch_sizes, Tensor hx, TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional)",
7861:     "gru(Tensor input, Tensor hx, TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional, bool batch_first)",
7862:   }, /*traceable=*/true);
7863: 
7864:   ParsedArgs<9> parsed_args;
7865:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7866:   if(_r.has_torch_function()) {
7867:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7868:   }
7869:   switch (_r.idx) {
7870:     case 0: {
7871:       // aten::gru.data(Tensor data, Tensor batch_sizes, Tensor hx, Tensor[] params, bool has_biases, int num_layers, float dropout, bool train, bool bidirectional) -> (Tensor, Tensor)
7872: 
7873:       auto dispatch_gru = [](const at::Tensor & data, const at::Tensor & batch_sizes, const at::Tensor & hx, at::TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional) -> ::std::tuple<at::Tensor,at::Tensor> {
7874:         pybind11::gil_scoped_release no_gil;
7875:         return at::gru(data, batch_sizes, hx, params, has_biases, num_layers, dropout, train, bidirectional);
7876:       };
7877:       return wrap(dispatch_gru(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensorlist(3), _r.toBool(4), _r.toInt64(5), _r.toDouble(6), _r.toBool(7), _r.toBool(8)));
7878:     }
7879:     case 1: {
7880:       // aten::gru.input(Tensor input, Tensor hx, Tensor[] params, bool has_biases, int num_layers, float dropout, bool train, bool bidirectional, bool batch_first) -> (Tensor, Tensor)
7881: 
7882:       auto dispatch_gru = [](const at::Tensor & input, const at::Tensor & hx, at::TensorList params, bool has_biases, int64_t num_layers, double dropout, bool train, bool bidirectional, bool batch_first) -> ::std::tuple<at::Tensor,at::Tensor> {
7883:         pybind11::gil_scoped_release no_gil;
7884:         return at::gru(input, hx, params, has_biases, num_layers, dropout, train, bidirectional, batch_first);
7885:       };
7886:       return wrap(dispatch_gru(_r.tensor(0), _r.tensor(1), _r.tensorlist(2), _r.toBool(3), _r.toInt64(4), _r.toDouble(5), _r.toBool(6), _r.toBool(7), _r.toBool(8)));
7887:     }
7888:   }
7889:   Py_RETURN_NONE;
7890:   END_HANDLE_TH_ERRORS
7891: }
7892: 
7893: // rnn_tanh_cell
7894: static PyObject * THPVariable_rnn_tanh_cell(PyObject* self_, PyObject* args, PyObject* kwargs)
7895: {
7896:   HANDLE_TH_ERRORS
7897:   static PythonArgParser parser({
7898:     "rnn_tanh_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor? b_ih=None, Tensor? b_hh=None)",
7899:   }, /*traceable=*/false);
7900: 
7901:   ParsedArgs<6> parsed_args;
7902:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7903:   if(_r.has_torch_function()) {
7904:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7905:   }
7906:   // aten::rnn_tanh_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor? b_ih=None, Tensor? b_hh=None) -> Tensor
7907: 
7908:   auto dispatch_rnn_tanh_cell = [](const at::Tensor & input, const at::Tensor & hx, const at::Tensor & w_ih, const at::Tensor & w_hh, const ::std::optional<at::Tensor> & b_ih, const ::std::optional<at::Tensor> & b_hh) -> at::Tensor {
7909:     pybind11::gil_scoped_release no_gil;
7910:     return at::rnn_tanh_cell(input, hx, w_ih, w_hh, b_ih, b_hh);
7911:   };
7912:   return wrap(dispatch_rnn_tanh_cell(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.optionalTensor(4), _r.optionalTensor(5)));
7913:   Py_RETURN_NONE;
7914:   END_HANDLE_TH_ERRORS
7915: }
7916: 
7917: // quantized_gru_cell
7918: static PyObject * THPVariable_quantized_gru_cell(PyObject* self_, PyObject* args, PyObject* kwargs)
7919: {
7920:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `promote_types`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `promote_types`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7921-8000

```cpp
7921:   static PythonArgParser parser({
7922:     "quantized_gru_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor b_ih, Tensor b_hh, Tensor packed_ih, Tensor packed_hh, Tensor col_offsets_ih, Tensor col_offsets_hh, Scalar scale_ih, Scalar scale_hh, Scalar zero_point_ih, Scalar zero_point_hh)",
7923:   }, /*traceable=*/true);
7924: 
7925:   ParsedArgs<14> parsed_args;
7926:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7927:   if(_r.has_torch_function()) {
7928:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7929:   }
7930:   // aten::quantized_gru_cell(Tensor input, Tensor hx, Tensor w_ih, Tensor w_hh, Tensor b_ih, Tensor b_hh, Tensor packed_ih, Tensor packed_hh, Tensor col_offsets_ih, Tensor col_offsets_hh, Scalar scale_ih, Scalar scale_hh, Scalar zero_point_ih, Scalar zero_point_hh) -> Tensor
7931: 
7932:   auto dispatch_quantized_gru_cell = [](const at::Tensor & input, const at::Tensor & hx, const at::Tensor & w_ih, const at::Tensor & w_hh, const at::Tensor & b_ih, const at::Tensor & b_hh, const at::Tensor & packed_ih, const at::Tensor & packed_hh, const at::Tensor & col_offsets_ih, const at::Tensor & col_offsets_hh, const at::Scalar & scale_ih, const at::Scalar & scale_hh, const at::Scalar & zero_point_ih, const at::Scalar & zero_point_hh) -> at::Tensor {
7933:     pybind11::gil_scoped_release no_gil;
7934:     return at::quantized_gru_cell(input, hx, w_ih, w_hh, b_ih, b_hh, packed_ih, packed_hh, col_offsets_ih, col_offsets_hh, scale_ih, scale_hh, zero_point_ih, zero_point_hh);
7935:   };
7936:   return wrap(dispatch_quantized_gru_cell(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.tensor(4), _r.tensor(5), _r.tensor(6), _r.tensor(7), _r.tensor(8), _r.tensor(9), _r.scalar(10), _r.scalar(11), _r.scalar(12), _r.scalar(13)));
7937:   Py_RETURN_NONE;
7938:   END_HANDLE_TH_ERRORS
7939: }
7940: 
7941: // masked_scatter
7942: static PyObject * THPVariable_masked_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
7943: {
7944:   HANDLE_TH_ERRORS
7945:   static PythonArgParser parser({
7946:     "masked_scatter(Tensor input, Tensor mask, Tensor source)",
7947:   }, /*traceable=*/true);
7948: 
7949:   ParsedArgs<3> parsed_args;
7950:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7951:   if(_r.has_torch_function()) {
7952:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7953:   }
7954:   // aten::masked_scatter(Tensor self, Tensor mask, Tensor source) -> Tensor
7955: 
7956:   auto dispatch_masked_scatter = [](const at::Tensor & self, const at::Tensor & mask, const at::Tensor & source) -> at::Tensor {
7957:     pybind11::gil_scoped_release no_gil;
7958:     return self.masked_scatter(mask, source);
7959:   };
7960:   return wrap(dispatch_masked_scatter(_r.tensor(0), _r.tensor(1), _r.tensor(2)));
7961:   Py_RETURN_NONE;
7962:   END_HANDLE_TH_ERRORS
7963: }
7964: 
7965: // put
7966: static PyObject * THPVariable_put(PyObject* self_, PyObject* args, PyObject* kwargs)
7967: {
7968:   HANDLE_TH_ERRORS
7969:   static PythonArgParser parser({
7970:     "put(Tensor input, Tensor index, Tensor source, bool accumulate=False)",
7971:   }, /*traceable=*/true);
7972: 
7973:   ParsedArgs<4> parsed_args;
7974:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
7975:   if(_r.has_torch_function()) {
7976:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
7977:   }
7978:   // aten::put(Tensor self, Tensor index, Tensor source, bool accumulate=False) -> Tensor
7979: 
7980:   auto dispatch_put = [](const at::Tensor & self, const at::Tensor & index, const at::Tensor & source, bool accumulate) -> at::Tensor {
7981:     pybind11::gil_scoped_release no_gil;
7982:     return self.put(index, source, accumulate);
7983:   };
7984:   return wrap(dispatch_put(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3)));
7985:   Py_RETURN_NONE;
7986:   END_HANDLE_TH_ERRORS
7987: }
7988: 
7989: \
7990: // __or__
7991: static PyObject * THPVariable___or__(PyObject* self_, PyObject* args, PyObject* kwargs)
7992: {
7993:   HANDLE_TH_ERRORS
7994:   static PythonArgParser parser({
7995:     "__or__(Tensor input, Tensor other)",
7996:     "__or__(Tensor input, Scalar other)",
7997:   }, /*traceable=*/true);
7998: 
7999:   ParsedArgs<2> parsed_args;
8000:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `quantized_gru_cell`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `quantized_gru_cell` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8001-8080

```cpp
8001:   if(_r.has_torch_function()) {
8002:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8003:   }
8004:   switch (_r.idx) {
8005:     case 0: {
8006:       // aten::__or__.Tensor(Tensor self, Tensor other) -> Tensor
8007: 
8008:       auto dispatch___or__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8009:         pybind11::gil_scoped_release no_gil;
8010:         return self.__or__(other);
8011:       };
8012:       return wrap(dispatch___or__(_r.tensor(0), _r.tensor(1)));
8013:     }
8014:     case 1: {
8015:       // aten::__or__.Scalar(Tensor self, Scalar other) -> Tensor
8016: 
8017:       auto dispatch___or__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8018:         pybind11::gil_scoped_release no_gil;
8019:         return self.__or__(other);
8020:       };
8021:       return wrap(dispatch___or__(_r.tensor(0), _r.scalar(1)));
8022:     }
8023:   }
8024:   Py_RETURN_NONE;
8025:   END_HANDLE_TH_ERRORS
8026: }
8027: 
8028: \
8029: // __xor__
8030: static PyObject * THPVariable___xor__(PyObject* self_, PyObject* args, PyObject* kwargs)
8031: {
8032:   HANDLE_TH_ERRORS
8033:   static PythonArgParser parser({
8034:     "__xor__(Tensor input, Tensor other)",
8035:     "__xor__(Tensor input, Scalar other)",
8036:   }, /*traceable=*/true);
8037: 
8038:   ParsedArgs<2> parsed_args;
8039:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8040:   if(_r.has_torch_function()) {
8041:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8042:   }
8043:   switch (_r.idx) {
8044:     case 0: {
8045:       // aten::__xor__.Tensor(Tensor self, Tensor other) -> Tensor
8046: 
8047:       auto dispatch___xor__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8048:         pybind11::gil_scoped_release no_gil;
8049:         return self.__xor__(other);
8050:       };
8051:       return wrap(dispatch___xor__(_r.tensor(0), _r.tensor(1)));
8052:     }
8053:     case 1: {
8054:       // aten::__xor__.Scalar(Tensor self, Scalar other) -> Tensor
8055: 
8056:       auto dispatch___xor__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8057:         pybind11::gil_scoped_release no_gil;
8058:         return self.__xor__(other);
8059:       };
8060:       return wrap(dispatch___xor__(_r.tensor(0), _r.scalar(1)));
8061:     }
8062:   }
8063:   Py_RETURN_NONE;
8064:   END_HANDLE_TH_ERRORS
8065: }
8066: 
8067: \
8068: // __lshift__
8069: static PyObject * THPVariable___lshift__(PyObject* self_, PyObject* args, PyObject* kwargs)
8070: {
8071:   HANDLE_TH_ERRORS
8072:   static PythonArgParser parser({
8073:     "__lshift__(Tensor input, Tensor other)",
8074:     "__lshift__(Tensor input, Scalar other)",
8075:   }, /*traceable=*/true);
8076: 
8077:   ParsedArgs<2> parsed_args;
8078:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8079:   if(_r.has_torch_function()) {
8080:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable___xor__`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable___xor__` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8081-8160

```cpp
8081:   }
8082:   switch (_r.idx) {
8083:     case 0: {
8084:       // aten::__lshift__.Tensor(Tensor self, Tensor other) -> Tensor
8085: 
8086:       auto dispatch___lshift__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8087:         pybind11::gil_scoped_release no_gil;
8088:         return self.__lshift__(other);
8089:       };
8090:       return wrap(dispatch___lshift__(_r.tensor(0), _r.tensor(1)));
8091:     }
8092:     case 1: {
8093:       // aten::__lshift__.Scalar(Tensor self, Scalar other) -> Tensor
8094: 
8095:       auto dispatch___lshift__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8096:         pybind11::gil_scoped_release no_gil;
8097:         return self.__lshift__(other);
8098:       };
8099:       return wrap(dispatch___lshift__(_r.tensor(0), _r.scalar(1)));
8100:     }
8101:   }
8102:   Py_RETURN_NONE;
8103:   END_HANDLE_TH_ERRORS
8104: }
8105: 
8106: \
8107: // bitwise_right_shift
8108: static PyObject * THPVariable_bitwise_right_shift(PyObject* self_, PyObject* args, PyObject* kwargs)
8109: {
8110:   HANDLE_TH_ERRORS
8111:   static PythonArgParser parser({
8112:     "bitwise_right_shift(Tensor input, Tensor other, *, Tensor out=None)",
8113:     "bitwise_right_shift(Scalar self, Tensor other)",
8114:     "bitwise_right_shift(Tensor input, Scalar other, *, Tensor out=None)",
8115:   }, /*traceable=*/true);
8116: 
8117:   ParsedArgs<3> parsed_args;
8118:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8119:   if(_r.has_torch_function()) {
8120:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8121:   }
8122:   switch (_r.idx) {
8123:     case 0: {
8124:       if (_r.isNone(2)) {
8125:         // aten::bitwise_right_shift.Tensor(Tensor self, Tensor other) -> Tensor
8126: 
8127:         auto dispatch_bitwise_right_shift = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8128:           pybind11::gil_scoped_release no_gil;
8129:           return self.bitwise_right_shift(other);
8130:         };
8131:         return wrap(dispatch_bitwise_right_shift(_r.tensor(0), _r.tensor(1)));
8132:       } else {
8133:         // aten::bitwise_right_shift.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
8134: 
8135:         auto dispatch_bitwise_right_shift_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8136:           pybind11::gil_scoped_release no_gil;
8137:           return at::bitwise_right_shift_out(out, self, other);
8138:         };
8139:         return wrap(dispatch_bitwise_right_shift_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8140:       }
8141:     }
8142:     case 1: {
8143:       // aten::bitwise_right_shift.Scalar_Tensor(Scalar self, Tensor other) -> Tensor
8144: 
8145:       auto dispatch_bitwise_right_shift = [](const at::Scalar & self, const at::Tensor & other) -> at::Tensor {
8146:         pybind11::gil_scoped_release no_gil;
8147:         return at::bitwise_right_shift(self, other);
8148:       };
8149:       return wrap(dispatch_bitwise_right_shift(_r.scalar(0), _r.tensor(1)));
8150:     }
8151:     case 2: {
8152:       if (_r.isNone(2)) {
8153:         // aten::bitwise_right_shift.Tensor_Scalar(Tensor self, Scalar other) -> Tensor
8154: 
8155:         auto dispatch_bitwise_right_shift = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8156:           pybind11::gil_scoped_release no_gil;
8157:           return self.bitwise_right_shift(other);
8158:         };
8159:         return wrap(dispatch_bitwise_right_shift(_r.tensor(0), _r.scalar(1)));
8160:       } else {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_bitwise_right_shift`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_bitwise_right_shift`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8161-8240

```cpp
8161:         // aten::bitwise_right_shift.Tensor_Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
8162: 
8163:         auto dispatch_bitwise_right_shift_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8164:           pybind11::gil_scoped_release no_gil;
8165:           return at::bitwise_right_shift_out(out, self, other);
8166:         };
8167:         return wrap(dispatch_bitwise_right_shift_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
8168:       }
8169:     }
8170:   }
8171:   Py_RETURN_NONE;
8172:   END_HANDLE_TH_ERRORS
8173: }
8174: 
8175: // tril
8176: static PyObject * THPVariable_tril(PyObject* self_, PyObject* args, PyObject* kwargs)
8177: {
8178:   HANDLE_TH_ERRORS
8179:   static PythonArgParser parser({
8180:     "tril(Tensor input, SymInt diagonal=0, *, Tensor out=None)",
8181:   }, /*traceable=*/true);
8182: 
8183:   ParsedArgs<3> parsed_args;
8184:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8185:   if(_r.has_torch_function()) {
8186:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8187:   }
8188:   if (_r.isNone(2)) {
8189:     // aten::tril(Tensor self, SymInt diagonal=0) -> Tensor
8190: 
8191:     auto dispatch_tril = [](const at::Tensor & self, c10::SymInt diagonal) -> at::Tensor {
8192:       pybind11::gil_scoped_release no_gil;
8193:       return self.tril_symint(diagonal);
8194:     };
8195:     return wrap(dispatch_tril(_r.tensor(0), _r.toSymInt(1)));
8196:   } else {
8197:     // aten::tril.out(Tensor self, SymInt diagonal=0, *, Tensor(a!) out) -> Tensor(a!)
8198: 
8199:     auto dispatch_tril_out = [](at::Tensor out, const at::Tensor & self, c10::SymInt diagonal) -> at::Tensor {
8200:       pybind11::gil_scoped_release no_gil;
8201:       return at::tril_symint_out(out, self, diagonal);
8202:     };
8203:     return wrap(dispatch_tril_out(_r.tensor(2), _r.tensor(0), _r.toSymInt(1)));
8204:   }
8205:   Py_RETURN_NONE;
8206:   END_HANDLE_TH_ERRORS
8207: }
8208: 
8209: // tril_indices
8210: static PyObject * THPVariable_tril_indices(PyObject* self_, PyObject* args, PyObject* kwargs)
8211: {
8212:   HANDLE_TH_ERRORS
8213:   static PythonArgParser parser({
8214:     "tril_indices(int64_t row, int64_t col, int64_t offset=0, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
8215:   }, /*traceable=*/true);
8216: 
8217:   ParsedArgs<8> parsed_args;
8218:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8219:   if(_r.has_torch_function()) {
8220:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8221:   }
8222:   // aten::tril_indices(int row, int col, int offset=0, *, ScalarType? dtype=long, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
8223:   const auto options = TensorOptions()
8224:       .dtype(_r.scalartypeWithDefault(3, at::kLong))
8225:       .device(_r.deviceWithDefault(5, torch::tensors::get_default_device()))
8226:       .layout(_r.layoutOptional(4))
8227:       .requires_grad(_r.toBool(7))
8228:       .pinned_memory(_r.toBool(6));
8229:   torch::utils::maybe_initialize_device(options);
8230: 
8231:   auto dispatch_tril_indices = [](int64_t row, int64_t col, int64_t offset, at::TensorOptions options) -> at::Tensor {
8232:     pybind11::gil_scoped_release no_gil;
8233:     return torch::tril_indices(row, col, offset, options);
8234:   };
8235:   return wrap(dispatch_tril_indices(_r.toInt64(0), _r.toInt64(1), _r.toInt64(2), options));
8236:   Py_RETURN_NONE;
8237:   END_HANDLE_TH_ERRORS
8238: }
8239: 
8240: \
```

- EN: The main execution path in this span is carried by `bitwise_right_shift_out`, `wrap`, `THPVariable_tril`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `bitwise_right_shift_out`, `wrap`, `THPVariable_tril` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 8241-8320

```cpp
8241: // not_equal
8242: static PyObject * THPVariable_not_equal(PyObject* self_, PyObject* args, PyObject* kwargs)
8243: {
8244:   HANDLE_TH_ERRORS
8245:   static PythonArgParser parser({
8246:     "not_equal(Tensor input, Tensor other, *, Tensor out=None)",
8247:     "not_equal(Tensor input, Scalar other, *, Tensor out=None)",
8248:   }, /*traceable=*/true);
8249: 
8250:   ParsedArgs<3> parsed_args;
8251:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8252:   if(_r.has_torch_function()) {
8253:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8254:   }
8255:   switch (_r.idx) {
8256:     case 0: {
8257:       if (_r.isNone(2)) {
8258:         // aten::not_equal.Tensor(Tensor self, Tensor other) -> Tensor
8259: 
8260:         auto dispatch_not_equal = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8261:           pybind11::gil_scoped_release no_gil;
8262:           return self.not_equal(other);
8263:         };
8264:         return wrap(dispatch_not_equal(_r.tensor(0), _r.tensor(1)));
8265:       } else {
8266:         // aten::not_equal.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
8267: 
8268:         auto dispatch_not_equal_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8269:           pybind11::gil_scoped_release no_gil;
8270:           return at::not_equal_out(out, self, other);
8271:         };
8272:         return wrap(dispatch_not_equal_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8273:       }
8274:     }
8275:     case 1: {
8276:       if (_r.isNone(2)) {
8277:         // aten::not_equal.Scalar(Tensor self, Scalar other) -> Tensor
8278: 
8279:         auto dispatch_not_equal = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8280:           pybind11::gil_scoped_release no_gil;
8281:           return self.not_equal(other);
8282:         };
8283:         return wrap(dispatch_not_equal(_r.tensor(0), _r.scalar(1)));
8284:       } else {
8285:         // aten::not_equal.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
8286: 
8287:         auto dispatch_not_equal_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8288:           pybind11::gil_scoped_release no_gil;
8289:           return at::not_equal_out(out, self, other);
8290:         };
8291:         return wrap(dispatch_not_equal_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
8292:       }
8293:     }
8294:   }
8295:   Py_RETURN_NONE;
8296:   END_HANDLE_TH_ERRORS
8297: }
8298: 
8299: \
8300: // greater_equal
8301: static PyObject * THPVariable_greater_equal(PyObject* self_, PyObject* args, PyObject* kwargs)
8302: {
8303:   HANDLE_TH_ERRORS
8304:   static PythonArgParser parser({
8305:     "greater_equal(Tensor input, Tensor other, *, Tensor out=None)",
8306:     "greater_equal(Tensor input, Scalar other, *, Tensor out=None)",
8307:   }, /*traceable=*/true);
8308: 
8309:   ParsedArgs<3> parsed_args;
8310:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8311:   if(_r.has_torch_function()) {
8312:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8313:   }
8314:   switch (_r.idx) {
8315:     case 0: {
8316:       if (_r.isNone(2)) {
8317:         // aten::greater_equal.Tensor(Tensor self, Tensor other) -> Tensor
8318: 
8319:         auto dispatch_greater_equal = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8320:           pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable_not_equal`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_not_equal`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8321-8400

```cpp
8321:           return self.greater_equal(other);
8322:         };
8323:         return wrap(dispatch_greater_equal(_r.tensor(0), _r.tensor(1)));
8324:       } else {
8325:         // aten::greater_equal.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
8326: 
8327:         auto dispatch_greater_equal_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8328:           pybind11::gil_scoped_release no_gil;
8329:           return at::greater_equal_out(out, self, other);
8330:         };
8331:         return wrap(dispatch_greater_equal_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8332:       }
8333:     }
8334:     case 1: {
8335:       if (_r.isNone(2)) {
8336:         // aten::greater_equal.Scalar(Tensor self, Scalar other) -> Tensor
8337: 
8338:         auto dispatch_greater_equal = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8339:           pybind11::gil_scoped_release no_gil;
8340:           return self.greater_equal(other);
8341:         };
8342:         return wrap(dispatch_greater_equal(_r.tensor(0), _r.scalar(1)));
8343:       } else {
8344:         // aten::greater_equal.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
8345: 
8346:         auto dispatch_greater_equal_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8347:           pybind11::gil_scoped_release no_gil;
8348:           return at::greater_equal_out(out, self, other);
8349:         };
8350:         return wrap(dispatch_greater_equal_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
8351:       }
8352:     }
8353:   }
8354:   Py_RETURN_NONE;
8355:   END_HANDLE_TH_ERRORS
8356: }
8357: 
8358: \
8359: // less
8360: static PyObject * THPVariable_less(PyObject* self_, PyObject* args, PyObject* kwargs)
8361: {
8362:   HANDLE_TH_ERRORS
8363:   static PythonArgParser parser({
8364:     "less(Tensor input, Tensor other, *, Tensor out=None)",
8365:     "less(Tensor input, Scalar other, *, Tensor out=None)",
8366:   }, /*traceable=*/true);
8367: 
8368:   ParsedArgs<3> parsed_args;
8369:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8370:   if(_r.has_torch_function()) {
8371:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8372:   }
8373:   switch (_r.idx) {
8374:     case 0: {
8375:       if (_r.isNone(2)) {
8376:         // aten::less.Tensor(Tensor self, Tensor other) -> Tensor
8377: 
8378:         auto dispatch_less = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8379:           pybind11::gil_scoped_release no_gil;
8380:           return self.less(other);
8381:         };
8382:         return wrap(dispatch_less(_r.tensor(0), _r.tensor(1)));
8383:       } else {
8384:         // aten::less.Tensor_out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
8385: 
8386:         auto dispatch_less_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8387:           pybind11::gil_scoped_release no_gil;
8388:           return at::less_out(out, self, other);
8389:         };
8390:         return wrap(dispatch_less_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8391:       }
8392:     }
8393:     case 1: {
8394:       if (_r.isNone(2)) {
8395:         // aten::less.Scalar(Tensor self, Scalar other) -> Tensor
8396: 
8397:         auto dispatch_less = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8398:           pybind11::gil_scoped_release no_gil;
8399:           return self.less(other);
8400:         };
```

- EN: The main execution path in this span is carried by `wrap`, `greater_equal_out`, `THPVariable_less`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `greater_equal_out`, `THPVariable_less` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8401-8480

```cpp
8401:         return wrap(dispatch_less(_r.tensor(0), _r.scalar(1)));
8402:       } else {
8403:         // aten::less.Scalar_out(Tensor self, Scalar other, *, Tensor(a!) out) -> Tensor(a!)
8404: 
8405:         auto dispatch_less_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8406:           pybind11::gil_scoped_release no_gil;
8407:           return at::less_out(out, self, other);
8408:         };
8409:         return wrap(dispatch_less_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
8410:       }
8411:     }
8412:   }
8413:   Py_RETURN_NONE;
8414:   END_HANDLE_TH_ERRORS
8415: }
8416: 
8417: \
8418: // index_select
8419: static PyObject * THPVariable_index_select(PyObject* self_, PyObject* args, PyObject* kwargs)
8420: {
8421:   HANDLE_TH_ERRORS
8422:   static PythonArgParser parser({
8423:     "index_select(Tensor input, int64_t dim, Tensor index, *, Tensor out=None)",
8424:     "index_select(Tensor input, Dimname dim, Tensor index, *, Tensor out=None)",
8425:   }, /*traceable=*/true);
8426: 
8427:   ParsedArgs<4> parsed_args;
8428:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8429:   if(_r.has_torch_function()) {
8430:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8431:   }
8432:   switch (_r.idx) {
8433:     case 0: {
8434:       if (_r.isNone(3)) {
8435:         // aten::index_select(Tensor self, int dim, Tensor index) -> Tensor
8436: 
8437:         auto dispatch_index_select = [](const at::Tensor & self, int64_t dim, const at::Tensor & index) -> at::Tensor {
8438:           pybind11::gil_scoped_release no_gil;
8439:           return self.index_select(dim, index);
8440:         };
8441:         return wrap(dispatch_index_select(_r.tensor(0), _r.toInt64(1), _r.tensor(2)));
8442:       } else {
8443:         // aten::index_select.out(Tensor self, int dim, Tensor index, *, Tensor(a!) out) -> Tensor(a!)
8444: 
8445:         auto dispatch_index_select_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, const at::Tensor & index) -> at::Tensor {
8446:           pybind11::gil_scoped_release no_gil;
8447:           return at::index_select_out(out, self, dim, index);
8448:         };
8449:         return wrap(dispatch_index_select_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.tensor(2)));
8450:       }
8451:     }
8452:     case 1: {
8453:       if (_r.isNone(3)) {
8454:         // aten::index_select.dimname(Tensor self, Dimname dim, Tensor index) -> Tensor
8455: 
8456:         auto dispatch_index_select = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index) -> at::Tensor {
8457:           pybind11::gil_scoped_release no_gil;
8458:           return self.index_select(dim, index);
8459:         };
8460:         return wrap(dispatch_index_select(_r.tensor(0), _r.dimname(1), _r.tensor(2)));
8461:       } else {
8462:         // aten::index_select.dimname_out(Tensor self, Dimname dim, Tensor index, *, Tensor(a!) out) -> Tensor(a!)
8463: 
8464:         auto dispatch_index_select_out = [](at::Tensor out, const at::Tensor & self, at::Dimname dim, const at::Tensor & index) -> at::Tensor {
8465:           pybind11::gil_scoped_release no_gil;
8466:           return at::index_select_out(out, self, dim, index);
8467:         };
8468:         return wrap(dispatch_index_select_out(_r.tensor(3), _r.tensor(0), _r.dimname(1), _r.tensor(2)));
8469:       }
8470:     }
8471:   }
8472:   Py_RETURN_NONE;
8473:   END_HANDLE_TH_ERRORS
8474: }
8475: 
8476: \
8477: // gather
8478: static PyObject * THPVariable_gather(PyObject* self_, PyObject* args, PyObject* kwargs)
8479: {
8480:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `wrap`, `less_out`, `THPVariable_index_select`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `less_out`, `THPVariable_index_select` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8481-8560

```cpp
8481:   static PythonArgParser parser({
8482:     "gather(Tensor input, int64_t dim, Tensor index, *, bool sparse_grad=False, Tensor out=None)",
8483:     "gather(Tensor input, Dimname dim, Tensor index, *, bool sparse_grad=False, Tensor out=None)",
8484:   }, /*traceable=*/true);
8485: 
8486:   ParsedArgs<5> parsed_args;
8487:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8488:   if(_r.has_torch_function()) {
8489:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8490:   }
8491:   switch (_r.idx) {
8492:     case 0: {
8493:       if (_r.isNone(4)) {
8494:         // aten::gather(Tensor self, int dim, Tensor index, *, bool sparse_grad=False) -> Tensor
8495: 
8496:         auto dispatch_gather = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, bool sparse_grad) -> at::Tensor {
8497:           pybind11::gil_scoped_release no_gil;
8498:           return self.gather(dim, index, sparse_grad);
8499:         };
8500:         return wrap(dispatch_gather(_r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.toBool(3)));
8501:       } else {
8502:         // aten::gather.out(Tensor self, int dim, Tensor index, *, bool sparse_grad=False, Tensor(a!) out) -> Tensor(a!)
8503: 
8504:         auto dispatch_gather_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, const at::Tensor & index, bool sparse_grad) -> at::Tensor {
8505:           pybind11::gil_scoped_release no_gil;
8506:           return at::gather_out(out, self, dim, index, sparse_grad);
8507:         };
8508:         return wrap(dispatch_gather_out(_r.tensor(4), _r.tensor(0), _r.toInt64(1), _r.tensor(2), _r.toBool(3)));
8509:       }
8510:     }
8511:     case 1: {
8512:       if (_r.isNone(4)) {
8513:         // aten::gather.dimname(Tensor self, Dimname dim, Tensor index, *, bool sparse_grad=False) -> Tensor
8514: 
8515:         auto dispatch_gather = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, bool sparse_grad) -> at::Tensor {
8516:           pybind11::gil_scoped_release no_gil;
8517:           return self.gather(dim, index, sparse_grad);
8518:         };
8519:         return wrap(dispatch_gather(_r.tensor(0), _r.dimname(1), _r.tensor(2), _r.toBool(3)));
8520:       } else {
8521:         // aten::gather.dimname_out(Tensor self, Dimname dim, Tensor index, *, bool sparse_grad=False, Tensor(a!) out) -> Tensor(a!)
8522: 
8523:         auto dispatch_gather_out = [](at::Tensor out, const at::Tensor & self, at::Dimname dim, const at::Tensor & index, bool sparse_grad) -> at::Tensor {
8524:           pybind11::gil_scoped_release no_gil;
8525:           return at::gather_out(out, self, dim, index, sparse_grad);
8526:         };
8527:         return wrap(dispatch_gather_out(_r.tensor(4), _r.tensor(0), _r.dimname(1), _r.tensor(2), _r.toBool(3)));
8528:       }
8529:     }
8530:   }
8531:   Py_RETURN_NONE;
8532:   END_HANDLE_TH_ERRORS
8533: }
8534: 
8535: \
8536: // addcmul
8537: static PyObject * THPVariable_addcmul(PyObject* self_, PyObject* args, PyObject* kwargs)
8538: {
8539:   HANDLE_TH_ERRORS
8540:   static PythonArgParser parser({
8541:     "addcmul(Tensor input, Scalar value, Tensor tensor1, Tensor tensor2, *, Tensor out=None)|deprecated",
8542:     "addcmul(Tensor input, Tensor tensor1, Tensor tensor2, *, Scalar value=1, Tensor out=None)",
8543:   }, /*traceable=*/true);
8544: 
8545:   ParsedArgs<5> parsed_args;
8546:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8547:   if(_r.has_torch_function()) {
8548:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8549:   }
8550:   switch (_r.idx) {
8551:     case 0: {
8552:       if (_r.isNone(4)) {
8553:         // [deprecated] aten::addcmul(Tensor self, Scalar value, Tensor tensor1, Tensor tensor2, *, Tensor(a!) out) -> Tensor(a!)
8554: 
8555:         auto dispatch_addcmul = [](at::Tensor out, const at::Tensor & self, const at::Scalar & value, const at::Tensor & tensor1, const at::Tensor & tensor2) -> at::Tensor {
8556:           pybind11::gil_scoped_release no_gil;
8557:           return self.addcmul(tensor1, tensor2, value);
8558:         };
8559:         return wrap(dispatch_addcmul(_r.tensor(4), _r.tensor(0), _r.scalar(1), _r.tensor(2), _r.tensor(3)));
8560:       } else {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `gather`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `gather` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8561-8640

```cpp
8561:         // [deprecated] aten::addcmul(Tensor self, Scalar value, Tensor tensor1, Tensor tensor2, *, Tensor(a!) out) -> Tensor(a!)
8562: 
8563:         auto dispatch_addcmul_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & value, const at::Tensor & tensor1, const at::Tensor & tensor2) -> at::Tensor {
8564:           pybind11::gil_scoped_release no_gil;
8565:           return at::addcmul_out(out, self, tensor1, tensor2, value);
8566:         };
8567:         return wrap(dispatch_addcmul_out(_r.tensor(4), _r.tensor(0), _r.scalar(1), _r.tensor(2), _r.tensor(3)));
8568:       }
8569:     }
8570:     case 1: {
8571:       if (_r.isNone(4)) {
8572:         // aten::addcmul(Tensor self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> Tensor
8573: 
8574:         auto dispatch_addcmul = [](const at::Tensor & self, const at::Tensor & tensor1, const at::Tensor & tensor2, const at::Scalar & value) -> at::Tensor {
8575:           pybind11::gil_scoped_release no_gil;
8576:           return self.addcmul(tensor1, tensor2, value);
8577:         };
8578:         return wrap(dispatch_addcmul(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3)));
8579:       } else {
8580:         // aten::addcmul.out(Tensor self, Tensor tensor1, Tensor tensor2, *, Scalar value=1, Tensor(a!) out) -> Tensor(a!)
8581: 
8582:         auto dispatch_addcmul_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & tensor1, const at::Tensor & tensor2, const at::Scalar & value) -> at::Tensor {
8583:           pybind11::gil_scoped_release no_gil;
8584:           return at::addcmul_out(out, self, tensor1, tensor2, value);
8585:         };
8586:         return wrap(dispatch_addcmul_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.scalar(3)));
8587:       }
8588:     }
8589:   }
8590:   Py_RETURN_NONE;
8591:   END_HANDLE_TH_ERRORS
8592: }
8593: 
8594: // triangular_solve
8595: static PyObject * THPVariable_triangular_solve(PyObject* self_, PyObject* args, PyObject* kwargs)
8596: {
8597:   HANDLE_TH_ERRORS
8598:   static PyTypeObject* NamedTuple = generated::get_triangular_solve_out_structseq();
8599:   static PyTypeObject* NamedTuple1 = generated::get_triangular_solve_structseq();
8600:   static PythonArgParser parser({
8601:     "triangular_solve(Tensor input, Tensor A, bool upper=True, bool transpose=False, bool unitriangular=False, *, TensorList[2] out=None)",
8602:   }, /*traceable=*/true);
8603: 
8604:   ParsedArgs<6> parsed_args;
8605:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8606:   if(_r.has_torch_function()) {
8607:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8608:   }
8609:   if (_r.isNone(5)) {
8610:     // aten::triangular_solve(Tensor self, Tensor A, bool upper=True, bool transpose=False, bool unitriangular=False) -> (Tensor solution, Tensor cloned_coefficient)
8611: 
8612:     auto dispatch_triangular_solve = [](const at::Tensor & self, const at::Tensor & A, bool upper, bool transpose, bool unitriangular) -> ::std::tuple<at::Tensor,at::Tensor> {
8613:       pybind11::gil_scoped_release no_gil;
8614:       return self.triangular_solve(A, upper, transpose, unitriangular);
8615:     };
8616:     return wrap(NamedTuple1, dispatch_triangular_solve(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3), _r.toBool(4)));
8617:   } else {
8618:     // aten::triangular_solve.X(Tensor self, Tensor A, bool upper=True, bool transpose=False, bool unitriangular=False, *, Tensor(a!) X, Tensor(b!) M) -> (Tensor(a!) solution, Tensor(b!) cloned_coefficient)
8619:     auto out = _r.tensorlist_n<2>(5);
8620:     auto dispatch_triangular_solve_out = [](at::Tensor & X, at::Tensor & M, const at::Tensor & self, const at::Tensor & A, bool upper, bool transpose, bool unitriangular) -> ::std::tuple<at::Tensor,at::Tensor> {
8621:       pybind11::gil_scoped_release no_gil;
8622:       return at::triangular_solve_out(X, M, self, A, upper, transpose, unitriangular);
8623:     };
8624:     return wrap(NamedTuple, dispatch_triangular_solve_out(out[0], out[1], _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3), _r.toBool(4)));
8625:   }
8626:   Py_RETURN_NONE;
8627:   END_HANDLE_TH_ERRORS
8628: }
8629: 
8630: // swapaxes
8631: static PyObject * THPVariable_swapaxes(PyObject* self_, PyObject* args, PyObject* kwargs)
8632: {
8633:   HANDLE_TH_ERRORS
8634:   static PythonArgParser parser({
8635:     "swapaxes(Tensor input, int64_t axis0, int64_t axis1)",
8636:   }, /*traceable=*/true);
8637: 
8638:   ParsedArgs<3> parsed_args;
8639:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8640:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `addcmul`, `addcmul_out`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `addcmul`, `addcmul_out`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8641-8720

```cpp
8641:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8642:   }
8643:   // aten::swapaxes(Tensor(a) self, int axis0, int axis1) -> Tensor(a)
8644: 
8645:   auto dispatch_swapaxes = [](const at::Tensor & self, int64_t axis0, int64_t axis1) -> at::Tensor {
8646:     pybind11::gil_scoped_release no_gil;
8647:     return self.swapaxes(axis0, axis1);
8648:   };
8649:   return wrap(dispatch_swapaxes(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
8650:   Py_RETURN_NONE;
8651:   END_HANDLE_TH_ERRORS
8652: }
8653: 
8654: // cholesky_inverse
8655: static PyObject * THPVariable_cholesky_inverse(PyObject* self_, PyObject* args, PyObject* kwargs)
8656: {
8657:   HANDLE_TH_ERRORS
8658:   static PythonArgParser parser({
8659:     "cholesky_inverse(Tensor input, bool upper=False, *, Tensor out=None)",
8660:   }, /*traceable=*/true);
8661: 
8662:   ParsedArgs<3> parsed_args;
8663:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8664:   if(_r.has_torch_function()) {
8665:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8666:   }
8667:   if (_r.isNone(2)) {
8668:     // aten::cholesky_inverse(Tensor self, bool upper=False) -> Tensor
8669: 
8670:     auto dispatch_cholesky_inverse = [](const at::Tensor & self, bool upper) -> at::Tensor {
8671:       pybind11::gil_scoped_release no_gil;
8672:       return self.cholesky_inverse(upper);
8673:     };
8674:     return wrap(dispatch_cholesky_inverse(_r.tensor(0), _r.toBool(1)));
8675:   } else {
8676:     // aten::cholesky_inverse.out(Tensor self, bool upper=False, *, Tensor(a!) out) -> Tensor(a!)
8677: 
8678:     auto dispatch_cholesky_inverse_out = [](at::Tensor out, const at::Tensor & self, bool upper) -> at::Tensor {
8679:       pybind11::gil_scoped_release no_gil;
8680:       return at::cholesky_inverse_out(out, self, upper);
8681:     };
8682:     return wrap(dispatch_cholesky_inverse_out(_r.tensor(2), _r.tensor(0), _r.toBool(1)));
8683:   }
8684:   Py_RETURN_NONE;
8685:   END_HANDLE_TH_ERRORS
8686: }
8687: 
8688: // geqrf
8689: static PyObject * THPVariable_geqrf(PyObject* self_, PyObject* args, PyObject* kwargs)
8690: {
8691:   HANDLE_TH_ERRORS
8692:   static PyTypeObject* NamedTuple = generated::get_geqrf_out_structseq();
8693:   static PyTypeObject* NamedTuple1 = generated::get_geqrf_structseq();
8694:   static PythonArgParser parser({
8695:     "geqrf(Tensor input, *, TensorList[2] out=None)",
8696:   }, /*traceable=*/true);
8697: 
8698:   ParsedArgs<2> parsed_args;
8699:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8700:   if(_r.has_torch_function()) {
8701:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8702:   }
8703:   if (_r.isNone(1)) {
8704:     // aten::geqrf(Tensor self) -> (Tensor a, Tensor tau)
8705: 
8706:     auto dispatch_geqrf = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
8707:       pybind11::gil_scoped_release no_gil;
8708:       return self.geqrf();
8709:     };
8710:     return wrap(NamedTuple1, dispatch_geqrf(_r.tensor(0)));
8711:   } else {
8712:     // aten::geqrf.a(Tensor self, *, Tensor(a!) a, Tensor(b!) tau) -> (Tensor(a!) a, Tensor(b!) tau)
8713:     auto out = _r.tensorlist_n<2>(1);
8714:     auto dispatch_geqrf_out = [](at::Tensor & a, at::Tensor & tau, const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
8715:       pybind11::gil_scoped_release no_gil;
8716:       return at::geqrf_out(a, tau, self);
8717:     };
8718:     return wrap(NamedTuple, dispatch_geqrf_out(out[0], out[1], _r.tensor(0)));
8719:   }
8720:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `swapaxes`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `swapaxes`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8721-8800

```cpp
8721:   END_HANDLE_TH_ERRORS
8722: }
8723: 
8724: // orgqr
8725: static PyObject * THPVariable_orgqr(PyObject* self_, PyObject* args, PyObject* kwargs)
8726: {
8727:   HANDLE_TH_ERRORS
8728:   static PythonArgParser parser({
8729:     "orgqr(Tensor input, Tensor input2, *, Tensor out=None)",
8730:   }, /*traceable=*/true);
8731: 
8732:   ParsedArgs<3> parsed_args;
8733:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8734:   if(_r.has_torch_function()) {
8735:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8736:   }
8737:   if (_r.isNone(2)) {
8738:     // aten::orgqr(Tensor self, Tensor input2) -> Tensor
8739: 
8740:     auto dispatch_orgqr = [](const at::Tensor & self, const at::Tensor & input2) -> at::Tensor {
8741:       pybind11::gil_scoped_release no_gil;
8742:       return self.orgqr(input2);
8743:     };
8744:     return wrap(dispatch_orgqr(_r.tensor(0), _r.tensor(1)));
8745:   } else {
8746:     // aten::orgqr.out(Tensor self, Tensor input2, *, Tensor(a!) out) -> Tensor(a!)
8747: 
8748:     auto dispatch_orgqr_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & input2) -> at::Tensor {
8749:       pybind11::gil_scoped_release no_gil;
8750:       return at::orgqr_out(out, self, input2);
8751:     };
8752:     return wrap(dispatch_orgqr_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
8753:   }
8754:   Py_RETURN_NONE;
8755:   END_HANDLE_TH_ERRORS
8756: }
8757: 
8758: // ormqr
8759: static PyObject * THPVariable_ormqr(PyObject* self_, PyObject* args, PyObject* kwargs)
8760: {
8761:   HANDLE_TH_ERRORS
8762:   static PythonArgParser parser({
8763:     "ormqr(Tensor input, Tensor input2, Tensor input3, bool left=True, bool transpose=False, *, Tensor out=None)",
8764:   }, /*traceable=*/true);
8765: 
8766:   ParsedArgs<6> parsed_args;
8767:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8768:   if(_r.has_torch_function()) {
8769:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8770:   }
8771:   if (_r.isNone(5)) {
8772:     // aten::ormqr(Tensor self, Tensor input2, Tensor input3, bool left=True, bool transpose=False) -> Tensor
8773: 
8774:     auto dispatch_ormqr = [](const at::Tensor & self, const at::Tensor & input2, const at::Tensor & input3, bool left, bool transpose) -> at::Tensor {
8775:       pybind11::gil_scoped_release no_gil;
8776:       return self.ormqr(input2, input3, left, transpose);
8777:     };
8778:     return wrap(dispatch_ormqr(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3), _r.toBool(4)));
8779:   } else {
8780:     // aten::ormqr.out(Tensor self, Tensor input2, Tensor input3, bool left=True, bool transpose=False, *, Tensor(a!) out) -> Tensor(a!)
8781: 
8782:     auto dispatch_ormqr_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & input2, const at::Tensor & input3, bool left, bool transpose) -> at::Tensor {
8783:       pybind11::gil_scoped_release no_gil;
8784:       return at::ormqr_out(out, self, input2, input3, left, transpose);
8785:     };
8786:     return wrap(dispatch_ormqr_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.tensor(2), _r.toBool(3), _r.toBool(4)));
8787:   }
8788:   Py_RETURN_NONE;
8789:   END_HANDLE_TH_ERRORS
8790: }
8791: 
8792: // lu_solve
8793: static PyObject * THPVariable_lu_solve(PyObject* self_, PyObject* args, PyObject* kwargs)
8794: {
8795:   HANDLE_TH_ERRORS
8796:   static PythonArgParser parser({
8797:     "lu_solve(Tensor input, Tensor LU_data, Tensor LU_pivots, *, Tensor out=None)",
8798:   }, /*traceable=*/true);
8799: 
8800:   ParsedArgs<4> parsed_args;
```

- EN: The main execution path in this span is carried by `THPVariable_orgqr`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_orgqr`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8801-8880

```cpp
8801:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8802:   if(_r.has_torch_function()) {
8803:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8804:   }
8805:   if (_r.isNone(3)) {
8806:     // aten::lu_solve(Tensor self, Tensor LU_data, Tensor LU_pivots) -> Tensor
8807: 
8808:     auto dispatch_lu_solve = [](const at::Tensor & self, const at::Tensor & LU_data, const at::Tensor & LU_pivots) -> at::Tensor {
8809:       pybind11::gil_scoped_release no_gil;
8810:       return self.lu_solve(LU_data, LU_pivots);
8811:     };
8812:     return wrap(dispatch_lu_solve(_r.tensor(0), _r.tensor(1), _r.tensor(2)));
8813:   } else {
8814:     // aten::lu_solve.out(Tensor self, Tensor LU_data, Tensor LU_pivots, *, Tensor(a!) out) -> Tensor(a!)
8815: 
8816:     auto dispatch_lu_solve_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & LU_data, const at::Tensor & LU_pivots) -> at::Tensor {
8817:       pybind11::gil_scoped_release no_gil;
8818:       return at::lu_solve_out(out, self, LU_data, LU_pivots);
8819:     };
8820:     return wrap(dispatch_lu_solve_out(_r.tensor(3), _r.tensor(0), _r.tensor(1), _r.tensor(2)));
8821:   }
8822:   Py_RETURN_NONE;
8823:   END_HANDLE_TH_ERRORS
8824: }
8825: 
8826: // lgamma
8827: static PyObject * THPVariable_lgamma(PyObject* self_, PyObject* args, PyObject* kwargs)
8828: {
8829:   HANDLE_TH_ERRORS
8830:   static PythonArgParser parser({
8831:     "lgamma(Tensor input, *, Tensor out=None)",
8832:   }, /*traceable=*/true);
8833: 
8834:   ParsedArgs<2> parsed_args;
8835:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8836:   if(_r.has_torch_function()) {
8837:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8838:   }
8839:   if (_r.isNone(1)) {
8840:     // aten::lgamma(Tensor self) -> Tensor
8841: 
8842:     auto dispatch_lgamma = [](const at::Tensor & self) -> at::Tensor {
8843:       pybind11::gil_scoped_release no_gil;
8844:       return self.lgamma();
8845:     };
8846:     return wrap(dispatch_lgamma(_r.tensor(0)));
8847:   } else {
8848:     // aten::lgamma.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
8849: 
8850:     auto dispatch_lgamma_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
8851:       pybind11::gil_scoped_release no_gil;
8852:       return at::lgamma_out(out, self);
8853:     };
8854:     return wrap(dispatch_lgamma_out(_r.tensor(1), _r.tensor(0)));
8855:   }
8856:   Py_RETURN_NONE;
8857:   END_HANDLE_TH_ERRORS
8858: }
8859: 
8860: // digamma
8861: static PyObject * THPVariable_digamma(PyObject* self_, PyObject* args, PyObject* kwargs)
8862: {
8863:   HANDLE_TH_ERRORS
8864:   static PythonArgParser parser({
8865:     "digamma(Tensor input, *, Tensor out=None)",
8866:   }, /*traceable=*/true);
8867: 
8868:   ParsedArgs<2> parsed_args;
8869:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8870:   if(_r.has_torch_function()) {
8871:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8872:   }
8873:   if (_r.isNone(1)) {
8874:     // aten::digamma(Tensor self) -> Tensor
8875: 
8876:     auto dispatch_digamma = [](const at::Tensor & self) -> at::Tensor {
8877:       pybind11::gil_scoped_release no_gil;
8878:       return self.digamma();
8879:     };
8880:     return wrap(dispatch_digamma(_r.tensor(0)));
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `lu_solve`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `lu_solve`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8881-8960

```cpp
8881:   } else {
8882:     // aten::digamma.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
8883: 
8884:     auto dispatch_digamma_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
8885:       pybind11::gil_scoped_release no_gil;
8886:       return at::digamma_out(out, self);
8887:     };
8888:     return wrap(dispatch_digamma_out(_r.tensor(1), _r.tensor(0)));
8889:   }
8890:   Py_RETURN_NONE;
8891:   END_HANDLE_TH_ERRORS
8892: }
8893: 
8894: // erfinv
8895: static PyObject * THPVariable_erfinv(PyObject* self_, PyObject* args, PyObject* kwargs)
8896: {
8897:   HANDLE_TH_ERRORS
8898:   static PythonArgParser parser({
8899:     "erfinv(Tensor input, *, Tensor out=None)",
8900:   }, /*traceable=*/true);
8901: 
8902:   ParsedArgs<2> parsed_args;
8903:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8904:   if(_r.has_torch_function()) {
8905:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8906:   }
8907:   if (_r.isNone(1)) {
8908:     // aten::erfinv(Tensor self) -> Tensor
8909: 
8910:     auto dispatch_erfinv = [](const at::Tensor & self) -> at::Tensor {
8911:       pybind11::gil_scoped_release no_gil;
8912:       return self.erfinv();
8913:     };
8914:     return wrap(dispatch_erfinv(_r.tensor(0)));
8915:   } else {
8916:     // aten::erfinv.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
8917: 
8918:     auto dispatch_erfinv_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
8919:       pybind11::gil_scoped_release no_gil;
8920:       return at::erfinv_out(out, self);
8921:     };
8922:     return wrap(dispatch_erfinv_out(_r.tensor(1), _r.tensor(0)));
8923:   }
8924:   Py_RETURN_NONE;
8925:   END_HANDLE_TH_ERRORS
8926: }
8927: 
8928: // sign
8929: static PyObject * THPVariable_sign(PyObject* self_, PyObject* args, PyObject* kwargs)
8930: {
8931:   HANDLE_TH_ERRORS
8932:   static PythonArgParser parser({
8933:     "sign(Tensor input, *, Tensor out=None)",
8934:   }, /*traceable=*/true);
8935: 
8936:   ParsedArgs<2> parsed_args;
8937:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8938:   if(_r.has_torch_function()) {
8939:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8940:   }
8941:   if (_r.isNone(1)) {
8942:     // aten::sign(Tensor self) -> Tensor
8943: 
8944:     auto dispatch_sign = [](const at::Tensor & self) -> at::Tensor {
8945:       pybind11::gil_scoped_release no_gil;
8946:       return self.sign();
8947:     };
8948:     return wrap(dispatch_sign(_r.tensor(0)));
8949:   } else {
8950:     // aten::sign.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
8951: 
8952:     auto dispatch_sign_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
8953:       pybind11::gil_scoped_release no_gil;
8954:       return at::sign_out(out, self);
8955:     };
8956:     return wrap(dispatch_sign_out(_r.tensor(1), _r.tensor(0)));
8957:   }
8958:   Py_RETURN_NONE;
8959:   END_HANDLE_TH_ERRORS
8960: }
```

- EN: The main execution path in this span is carried by `digamma_out`, `wrap`, `THPVariable_erfinv`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `digamma_out`, `wrap`, `THPVariable_erfinv` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8961-9040

```cpp
8961: 
8962: // dist
8963: static PyObject * THPVariable_dist(PyObject* self_, PyObject* args, PyObject* kwargs)
8964: {
8965:   HANDLE_TH_ERRORS
8966:   static PythonArgParser parser({
8967:     "dist(Tensor input, Tensor other, Scalar p=2)",
8968:   }, /*traceable=*/true);
8969: 
8970:   ParsedArgs<3> parsed_args;
8971:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8972:   if(_r.has_torch_function()) {
8973:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8974:   }
8975:   // aten::dist(Tensor self, Tensor other, Scalar p=2) -> Tensor
8976: 
8977:   auto dispatch_dist = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & p) -> at::Tensor {
8978:     pybind11::gil_scoped_release no_gil;
8979:     return self.dist(other, p);
8980:   };
8981:   return wrap(dispatch_dist(_r.tensor(0), _r.tensor(1), _r.scalar(2)));
8982:   Py_RETURN_NONE;
8983:   END_HANDLE_TH_ERRORS
8984: }
8985: 
8986: // arctan2
8987: static PyObject * THPVariable_arctan2(PyObject* self_, PyObject* args, PyObject* kwargs)
8988: {
8989:   HANDLE_TH_ERRORS
8990:   static PythonArgParser parser({
8991:     "arctan2(Tensor input, Tensor other, *, Tensor out=None)",
8992:   }, /*traceable=*/true);
8993: 
8994:   ParsedArgs<3> parsed_args;
8995:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
8996:   if(_r.has_torch_function()) {
8997:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
8998:   }
8999:   if (_r.isNone(2)) {
9000:     // aten::arctan2(Tensor self, Tensor other) -> Tensor
9001: 
9002:     auto dispatch_arctan2 = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9003:       pybind11::gil_scoped_release no_gil;
9004:       return self.arctan2(other);
9005:     };
9006:     return wrap(dispatch_arctan2(_r.tensor(0), _r.tensor(1)));
9007:   } else {
9008:     // aten::arctan2.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9009: 
9010:     auto dispatch_arctan2_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9011:       pybind11::gil_scoped_release no_gil;
9012:       return at::arctan2_out(out, self, other);
9013:     };
9014:     return wrap(dispatch_arctan2_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9015:   }
9016:   Py_RETURN_NONE;
9017:   END_HANDLE_TH_ERRORS
9018: }
9019: 
9020: // histc
9021: static PyObject * THPVariable_histc(PyObject* self_, PyObject* args, PyObject* kwargs)
9022: {
9023:   HANDLE_TH_ERRORS
9024:   static PythonArgParser parser({
9025:     "histc(Tensor input, int64_t bins=100, Scalar min=0, Scalar max=0, *, Tensor out=None)",
9026:   }, /*traceable=*/true);
9027: 
9028:   ParsedArgs<5> parsed_args;
9029:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9030:   if(_r.has_torch_function()) {
9031:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9032:   }
9033:   if (_r.isNone(4)) {
9034:     // aten::histc(Tensor self, int bins=100, Scalar min=0, Scalar max=0) -> Tensor
9035: 
9036:     auto dispatch_histc = [](const at::Tensor & self, int64_t bins, const at::Scalar & min, const at::Scalar & max) -> at::Tensor {
9037:       pybind11::gil_scoped_release no_gil;
9038:       return self.histc(bins, min, max);
9039:     };
9040:     return wrap(dispatch_histc(_r.tensor(0), _r.toInt64(1), _r.scalar(2), _r.scalar(3)));
```

- EN: The main execution path in this span is carried by `THPVariable_dist`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_dist`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9041-9120

```cpp
9041:   } else {
9042:     // aten::histc.out(Tensor self, int bins=100, Scalar min=0, Scalar max=0, *, Tensor(a!) out) -> Tensor(a!)
9043: 
9044:     auto dispatch_histc_out = [](at::Tensor out, const at::Tensor & self, int64_t bins, const at::Scalar & min, const at::Scalar & max) -> at::Tensor {
9045:       pybind11::gil_scoped_release no_gil;
9046:       return at::histc_out(out, self, bins, min, max);
9047:     };
9048:     return wrap(dispatch_histc_out(_r.tensor(4), _r.tensor(0), _r.toInt64(1), _r.scalar(2), _r.scalar(3)));
9049:   }
9050:   Py_RETURN_NONE;
9051:   END_HANDLE_TH_ERRORS
9052: }
9053: 
9054: \
9055: // histogram
9056: static PyObject * THPVariable_histogram(PyObject* self_, PyObject* args, PyObject* kwargs)
9057: {
9058:   HANDLE_TH_ERRORS
9059:   static PyTypeObject* NamedTuple = generated::get_histogram_out_structseq();
9060:   static PyTypeObject* NamedTuple1 = generated::get_histogram_structseq();
9061:   static PythonArgParser parser({
9062:     "histogram(Tensor input, Tensor bins, *, Tensor? weight=None, bool density=False, TensorList[2] out=None)",
9063:     "histogram(Tensor input, int64_t bins=100, *, ArrayRef<double>? range=None, Tensor? weight=None, bool density=False, TensorList[2] out=None)",
9064:   }, /*traceable=*/true);
9065: 
9066:   ParsedArgs<6> parsed_args;
9067:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9068:   if(_r.has_torch_function()) {
9069:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9070:   }
9071:   switch (_r.idx) {
9072:     case 0: {
9073:       if (_r.isNone(4)) {
9074:         // aten::histogram.bins_tensor(Tensor self, Tensor bins, *, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor bin_edges)
9075: 
9076:         auto dispatch_histogram = [](const at::Tensor & self, const at::Tensor & bins, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,at::Tensor> {
9077:           pybind11::gil_scoped_release no_gil;
9078:           return self.histogram(bins, weight, density);
9079:         };
9080:         return wrap(NamedTuple1, dispatch_histogram(_r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toBool(3)));
9081:       } else {
9082:         // aten::histogram.bins_tensor_out(Tensor self, Tensor bins, *, Tensor? weight=None, bool density=False, Tensor(a!) hist, Tensor(b!) bin_edges) -> (Tensor(a!) hist, Tensor(b!) bin_edges)
9083:         auto out = _r.tensorlist_n<2>(4);
9084:         auto dispatch_histogram_out = [](at::Tensor & hist, at::Tensor & bin_edges, const at::Tensor & self, const at::Tensor & bins, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,at::Tensor> {
9085:           pybind11::gil_scoped_release no_gil;
9086:           return at::histogram_out(hist, bin_edges, self, bins, weight, density);
9087:         };
9088:         return wrap(NamedTuple, dispatch_histogram_out(out[0], out[1], _r.tensor(0), _r.tensor(1), _r.optionalTensor(2), _r.toBool(3)));
9089:       }
9090:     }
9091:     case 1: {
9092:       if (_r.isNone(5)) {
9093:         // aten::histogram.bin_ct(Tensor self, int bins=100, *, float[]? range=None, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor bin_edges)
9094: 
9095:         auto dispatch_histogram = [](const at::Tensor & self, int64_t bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,at::Tensor> {
9096:           pybind11::gil_scoped_release no_gil;
9097:           return self.histogram(bins, range, weight, density);
9098:         };
9099:         return wrap(NamedTuple1, dispatch_histogram(_r.tensor(0), _r.toInt64(1), _r.doublelistOptional(2), _r.optionalTensor(3), _r.toBool(4)));
9100:       } else {
9101:         // aten::histogram.bin_ct_out(Tensor self, int bins=100, *, float[]? range=None, Tensor? weight=None, bool density=False, Tensor(a!) hist, Tensor(b!) bin_edges) -> (Tensor(a!) hist, Tensor(b!) bin_edges)
9102:         auto out = _r.tensorlist_n<2>(5);
9103:         auto dispatch_histogram_out = [](at::Tensor & hist, at::Tensor & bin_edges, const at::Tensor & self, int64_t bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,at::Tensor> {
9104:           pybind11::gil_scoped_release no_gil;
9105:           return at::histogram_out(hist, bin_edges, self, bins, range, weight, density);
9106:         };
9107:         return wrap(NamedTuple, dispatch_histogram_out(out[0], out[1], _r.tensor(0), _r.toInt64(1), _r.doublelistOptional(2), _r.optionalTensor(3), _r.toBool(4)));
9108:       }
9109:     }
9110:   }
9111:   Py_RETURN_NONE;
9112:   END_HANDLE_TH_ERRORS
9113: }
9114: 
9115: // _histogramdd_bin_edges
9116: static PyObject * THPVariable__histogramdd_bin_edges(PyObject* self_, PyObject* args, PyObject* kwargs)
9117: {
9118:   HANDLE_TH_ERRORS
9119:   static PythonArgParser parser({
9120:     "_histogramdd_bin_edges(Tensor input, IntArrayRef bins, *, ArrayRef<double>? range=None, Tensor? weight=None, bool density=False)",
```

- EN: The main execution path in this span is carried by `histc_out`, `wrap`, `THPVariable_histogram`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `histc_out`, `wrap`, `THPVariable_histogram` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9121-9200

```cpp
9121:   }, /*traceable=*/true);
9122: 
9123:   ParsedArgs<5> parsed_args;
9124:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9125:   if(_r.has_torch_function()) {
9126:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9127:   }
9128:   // aten::_histogramdd_bin_edges(Tensor self, int[] bins, *, float[]? range=None, Tensor? weight=None, bool density=False) -> Tensor[]
9129: 
9130:   auto dispatch__histogramdd_bin_edges = [](const at::Tensor & self, at::IntArrayRef bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::vector<at::Tensor> {
9131:     pybind11::gil_scoped_release no_gil;
9132:     return at::_histogramdd_bin_edges(self, bins, range, weight, density);
9133:   };
9134:   return wrap(dispatch__histogramdd_bin_edges(_r.tensor(0), _r.intlist(1), _r.doublelistOptional(2), _r.optionalTensor(3), _r.toBool(4)));
9135:   Py_RETURN_NONE;
9136:   END_HANDLE_TH_ERRORS
9137: }
9138: 
9139: \
9140: // histogramdd
9141: static PyObject * THPVariable_histogramdd(PyObject* self_, PyObject* args, PyObject* kwargs)
9142: {
9143:   HANDLE_TH_ERRORS
9144:   static PyTypeObject* NamedTuple = generated::get_histogramdd_structseq();
9145:   static PythonArgParser parser({
9146:     "histogramdd(Tensor input, int64_t bins, ArrayRef<double>? range=None, Tensor? weight=None, bool density=False)",
9147:     "histogramdd(Tensor input, IntArrayRef bins, ArrayRef<double>? range=None, Tensor? weight=None, bool density=False)",
9148:     "histogramdd(Tensor input, TensorList bins, ArrayRef<double>? range=None, Tensor? weight=None, bool density=False)",
9149:   }, /*traceable=*/true);
9150: 
9151:   ParsedArgs<5> parsed_args;
9152:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9153:   if(_r.has_torch_function()) {
9154:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9155:   }
9156:   switch (_r.idx) {
9157:     case 0: {
9158:       // aten::histogramdd.int_bins(Tensor self, int bins, float[]? range=None, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor[] bin_edges)
9159: 
9160:       auto dispatch_histogramdd = [](const at::Tensor & self, int64_t bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,::std::vector<at::Tensor>> {
9161:         pybind11::gil_scoped_release no_gil;
9162:         return at::histogramdd(self, bins, range, weight, density);
9163:       };
9164:       return wrap(NamedTuple, dispatch_histogramdd(_r.tensor(0), _r.toInt64(1), _r.doublelistOptional(2), _r.optionalTensor(3), _r.toBool(4)));
9165:     }
9166:     case 1: {
9167:       // aten::histogramdd(Tensor self, int[] bins, float[]? range=None, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor[] bin_edges)
9168: 
9169:       auto dispatch_histogramdd = [](const at::Tensor & self, at::IntArrayRef bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,::std::vector<at::Tensor>> {
9170:         pybind11::gil_scoped_release no_gil;
9171:         return at::histogramdd(self, bins, range, weight, density);
9172:       };
9173:       return wrap(NamedTuple, dispatch_histogramdd(_r.tensor(0), _r.intlist(1), _r.doublelistOptional(2), _r.optionalTensor(3), _r.toBool(4)));
9174:     }
9175:     case 2: {
9176:       // aten::histogramdd.TensorList_bins(Tensor self, Tensor[] bins, float[]? range=None, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor[] bin_edges)
9177: 
9178:       auto dispatch_histogramdd = [](const at::Tensor & self, at::TensorList bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,::std::vector<at::Tensor>> {
9179:         pybind11::gil_scoped_release no_gil;
9180:         return at::histogramdd(self, bins, range, weight, density);
9181:       };
9182:       return wrap(NamedTuple, dispatch_histogramdd(_r.tensor(0), _r.tensorlist(1), _r.doublelistOptional(2), _r.optionalTensor(3), _r.toBool(4)));
9183:     }
9184:   }
9185:   Py_RETURN_NONE;
9186:   END_HANDLE_TH_ERRORS
9187: }
9188: 
9189: // fmax
9190: static PyObject * THPVariable_fmax(PyObject* self_, PyObject* args, PyObject* kwargs)
9191: {
9192:   HANDLE_TH_ERRORS
9193:   static PythonArgParser parser({
9194:     "fmax(Tensor input, Tensor other, *, Tensor out=None)",
9195:   }, /*traceable=*/true);
9196: 
9197:   ParsedArgs<3> parsed_args;
9198:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9199:   if(_r.has_torch_function()) {
9200:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_histogramdd_bin_edges`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_histogramdd_bin_edges`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9201-9280

```cpp
9201:   }
9202:   if (_r.isNone(2)) {
9203:     // aten::fmax(Tensor self, Tensor other) -> Tensor
9204: 
9205:     auto dispatch_fmax = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9206:       pybind11::gil_scoped_release no_gil;
9207:       return self.fmax(other);
9208:     };
9209:     return wrap(dispatch_fmax(_r.tensor(0), _r.tensor(1)));
9210:   } else {
9211:     // aten::fmax.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9212: 
9213:     auto dispatch_fmax_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9214:       pybind11::gil_scoped_release no_gil;
9215:       return at::fmax_out(out, self, other);
9216:     };
9217:     return wrap(dispatch_fmax_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9218:   }
9219:   Py_RETURN_NONE;
9220:   END_HANDLE_TH_ERRORS
9221: }
9222: 
9223: // maximum
9224: static PyObject * THPVariable_maximum(PyObject* self_, PyObject* args, PyObject* kwargs)
9225: {
9226:   HANDLE_TH_ERRORS
9227:   static PythonArgParser parser({
9228:     "maximum(Tensor input, Tensor other, *, Tensor out=None)",
9229:   }, /*traceable=*/true);
9230: 
9231:   ParsedArgs<3> parsed_args;
9232:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9233:   if(_r.has_torch_function()) {
9234:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9235:   }
9236:   if (_r.isNone(2)) {
9237:     // aten::maximum(Tensor self, Tensor other) -> Tensor
9238: 
9239:     auto dispatch_maximum = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9240:       pybind11::gil_scoped_release no_gil;
9241:       return self.maximum(other);
9242:     };
9243:     return wrap(dispatch_maximum(_r.tensor(0), _r.tensor(1)));
9244:   } else {
9245:     // aten::maximum.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9246: 
9247:     auto dispatch_maximum_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9248:       pybind11::gil_scoped_release no_gil;
9249:       return at::maximum_out(out, self, other);
9250:     };
9251:     return wrap(dispatch_maximum_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9252:   }
9253:   Py_RETURN_NONE;
9254:   END_HANDLE_TH_ERRORS
9255: }
9256: 
9257: // minimum
9258: static PyObject * THPVariable_minimum(PyObject* self_, PyObject* args, PyObject* kwargs)
9259: {
9260:   HANDLE_TH_ERRORS
9261:   static PythonArgParser parser({
9262:     "minimum(Tensor input, Tensor other, *, Tensor out=None)",
9263:   }, /*traceable=*/true);
9264: 
9265:   ParsedArgs<3> parsed_args;
9266:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9267:   if(_r.has_torch_function()) {
9268:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9269:   }
9270:   if (_r.isNone(2)) {
9271:     // aten::minimum(Tensor self, Tensor other) -> Tensor
9272: 
9273:     auto dispatch_minimum = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9274:       pybind11::gil_scoped_release no_gil;
9275:       return self.minimum(other);
9276:     };
9277:     return wrap(dispatch_minimum(_r.tensor(0), _r.tensor(1)));
9278:   } else {
9279:     // aten::minimum.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
9280: 
```

- EN: The main execution path in this span is carried by `fmax`, `wrap`, `fmax_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `fmax`, `wrap`, `fmax_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9281-9360

```cpp
9281:     auto dispatch_minimum_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9282:       pybind11::gil_scoped_release no_gil;
9283:       return at::minimum_out(out, self, other);
9284:     };
9285:     return wrap(dispatch_minimum_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9286:   }
9287:   Py_RETURN_NONE;
9288:   END_HANDLE_TH_ERRORS
9289: }
9290: 
9291: \
9292: // nanquantile
9293: static PyObject * THPVariable_nanquantile(PyObject* self_, PyObject* args, PyObject* kwargs)
9294: {
9295:   HANDLE_TH_ERRORS
9296:   static PythonArgParser parser({
9297:     "nanquantile(Tensor input, Tensor q, int64_t? dim=None, bool keepdim=False, *, c10::string_view interpolation=\"linear\", Tensor out=None)",
9298:     "nanquantile(Tensor input, double q, int64_t? dim=None, bool keepdim=False, *, c10::string_view interpolation=\"linear\", Tensor out=None)",
9299:   }, /*traceable=*/true);
9300: 
9301:   ParsedArgs<6> parsed_args;
9302:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9303:   if(_r.has_torch_function()) {
9304:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9305:   }
9306:   switch (_r.idx) {
9307:     case 0: {
9308:       if (_r.isNone(5)) {
9309:         // aten::nanquantile(Tensor self, Tensor q, int? dim=None, bool keepdim=False, *, str interpolation='linear') -> Tensor
9310: 
9311:         auto dispatch_nanquantile = [](const at::Tensor & self, const at::Tensor & q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
9312:           pybind11::gil_scoped_release no_gil;
9313:           return self.nanquantile(q, dim, keepdim, interpolation);
9314:         };
9315:         return wrap(dispatch_nanquantile(_r.tensor(0), _r.tensor(1), _r.toInt64Optional(2), _r.toBool(3), _r.stringView(4)));
9316:       } else {
9317:         // aten::nanquantile.out(Tensor self, Tensor q, int? dim=None, bool keepdim=False, *, str interpolation='linear', Tensor(a!) out) -> Tensor(a!)
9318: 
9319:         auto dispatch_nanquantile_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
9320:           pybind11::gil_scoped_release no_gil;
9321:           return at::nanquantile_out(out, self, q, dim, keepdim, interpolation);
9322:         };
9323:         return wrap(dispatch_nanquantile_out(_r.tensor(5), _r.tensor(0), _r.tensor(1), _r.toInt64Optional(2), _r.toBool(3), _r.stringView(4)));
9324:       }
9325:     }
9326:     case 1: {
9327:       if (_r.isNone(5)) {
9328:         // aten::nanquantile.scalar(Tensor self, float q, int? dim=None, bool keepdim=False, *, str interpolation='linear') -> Tensor
9329: 
9330:         auto dispatch_nanquantile = [](const at::Tensor & self, double q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
9331:           pybind11::gil_scoped_release no_gil;
9332:           return self.nanquantile(q, dim, keepdim, interpolation);
9333:         };
9334:         return wrap(dispatch_nanquantile(_r.tensor(0), _r.toDouble(1), _r.toInt64Optional(2), _r.toBool(3), _r.stringView(4)));
9335:       } else {
9336:         // aten::nanquantile.scalar_out(Tensor self, float q, int? dim=None, bool keepdim=False, *, str interpolation='linear', Tensor(a!) out) -> Tensor(a!)
9337: 
9338:         auto dispatch_nanquantile_out = [](at::Tensor out, const at::Tensor & self, double q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
9339:           pybind11::gil_scoped_release no_gil;
9340:           return at::nanquantile_out(out, self, q, dim, keepdim, interpolation);
9341:         };
9342:         return wrap(dispatch_nanquantile_out(_r.tensor(5), _r.tensor(0), _r.toDouble(1), _r.toInt64Optional(2), _r.toBool(3), _r.stringView(4)));
9343:       }
9344:     }
9345:   }
9346:   Py_RETURN_NONE;
9347:   END_HANDLE_TH_ERRORS
9348: }
9349: 
9350: // msort
9351: static PyObject * THPVariable_msort(PyObject* self_, PyObject* args, PyObject* kwargs)
9352: {
9353:   HANDLE_TH_ERRORS
9354:   static PythonArgParser parser({
9355:     "msort(Tensor input, *, Tensor out=None)",
9356:   }, /*traceable=*/true);
9357: 
9358:   ParsedArgs<2> parsed_args;
9359:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9360:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `minimum_out`, `wrap`, `THPVariable_nanquantile`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `minimum_out`, `wrap`, `THPVariable_nanquantile` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9361-9440

```cpp
9361:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9362:   }
9363:   if (_r.isNone(1)) {
9364:     // aten::msort(Tensor self) -> Tensor
9365: 
9366:     auto dispatch_msort = [](const at::Tensor & self) -> at::Tensor {
9367:       pybind11::gil_scoped_release no_gil;
9368:       return self.msort();
9369:     };
9370:     return wrap(dispatch_msort(_r.tensor(0)));
9371:   } else {
9372:     // aten::msort.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
9373: 
9374:     auto dispatch_msort_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
9375:       pybind11::gil_scoped_release no_gil;
9376:       return at::msort_out(out, self);
9377:     };
9378:     return wrap(dispatch_msort_out(_r.tensor(1), _r.tensor(0)));
9379:   }
9380:   Py_RETURN_NONE;
9381:   END_HANDLE_TH_ERRORS
9382: }
9383: 
9384: \
9385: // pow
9386: static PyObject * THPVariable_pow(PyObject* self_, PyObject* args, PyObject* kwargs)
9387: {
9388:   HANDLE_TH_ERRORS
9389:   static PythonArgParser parser({
9390:     "pow(Tensor input, Tensor exponent, *, Tensor out=None)",
9391:     "pow(Scalar self, Tensor exponent, *, Tensor out=None)",
9392:     "pow(Tensor input, Scalar exponent, *, Tensor out=None)",
9393:   }, /*traceable=*/true);
9394: 
9395:   ParsedArgs<3> parsed_args;
9396:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9397:   if(_r.has_torch_function()) {
9398:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9399:   }
9400:   switch (_r.idx) {
9401:     case 0: {
9402:       if (_r.isNone(2)) {
9403:         // aten::pow.Tensor_Tensor(Tensor self, Tensor exponent) -> Tensor
9404: 
9405:         auto dispatch_pow = [](const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
9406:           pybind11::gil_scoped_release no_gil;
9407:           return self.pow(exponent);
9408:         };
9409:         return wrap(dispatch_pow(_r.tensor(0), _r.tensor(1)));
9410:       } else {
9411:         // aten::pow.Tensor_Tensor_out(Tensor self, Tensor exponent, *, Tensor(a!) out) -> Tensor(a!)
9412: 
9413:         auto dispatch_pow_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
9414:           pybind11::gil_scoped_release no_gil;
9415:           return at::pow_out(out, self, exponent);
9416:         };
9417:         return wrap(dispatch_pow_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9418:       }
9419:     }
9420:     case 1: {
9421:       if (_r.isNone(2)) {
9422:         // aten::pow.Scalar(Scalar self, Tensor exponent) -> Tensor
9423: 
9424:         auto dispatch_pow = [](const at::Scalar & self, const at::Tensor & exponent) -> at::Tensor {
9425:           pybind11::gil_scoped_release no_gil;
9426:           return at::pow(self, exponent);
9427:         };
9428:         return wrap(dispatch_pow(_r.scalar(0), _r.tensor(1)));
9429:       } else {
9430:         // aten::pow.Scalar_out(Scalar self, Tensor exponent, *, Tensor(a!) out) -> Tensor(a!)
9431: 
9432:         auto dispatch_pow_out = [](at::Tensor out, const at::Scalar & self, const at::Tensor & exponent) -> at::Tensor {
9433:           pybind11::gil_scoped_release no_gil;
9434:           return at::pow_out(out, self, exponent);
9435:         };
9436:         return wrap(dispatch_pow_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
9437:       }
9438:     }
9439:     case 2: {
9440:       if (_r.isNone(2)) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `msort`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `msort`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9441-9520

```cpp
9441:         // aten::pow.Tensor_Scalar(Tensor self, Scalar exponent) -> Tensor
9442: 
9443:         auto dispatch_pow = [](const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
9444:           pybind11::gil_scoped_release no_gil;
9445:           return self.pow(exponent);
9446:         };
9447:         return wrap(dispatch_pow(_r.tensor(0), _r.scalar(1)));
9448:       } else {
9449:         // aten::pow.Tensor_Scalar_out(Tensor self, Scalar exponent, *, Tensor(a!) out) -> Tensor(a!)
9450: 
9451:         auto dispatch_pow_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
9452:           pybind11::gil_scoped_release no_gil;
9453:           return at::pow_out(out, self, exponent);
9454:         };
9455:         return wrap(dispatch_pow_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
9456:       }
9457:     }
9458:   }
9459:   Py_RETURN_NONE;
9460:   END_HANDLE_TH_ERRORS
9461: }
9462: 
9463: \
9464: // float_power
9465: static PyObject * THPVariable_float_power(PyObject* self_, PyObject* args, PyObject* kwargs)
9466: {
9467:   HANDLE_TH_ERRORS
9468:   static PythonArgParser parser({
9469:     "float_power(Tensor input, Tensor exponent, *, Tensor out=None)",
9470:     "float_power(Scalar self, Tensor exponent, *, Tensor out=None)",
9471:     "float_power(Tensor input, Scalar exponent, *, Tensor out=None)",
9472:   }, /*traceable=*/true);
9473: 
9474:   ParsedArgs<3> parsed_args;
9475:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9476:   if(_r.has_torch_function()) {
9477:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9478:   }
9479:   switch (_r.idx) {
9480:     case 0: {
9481:       if (_r.isNone(2)) {
9482:         // aten::float_power.Tensor_Tensor(Tensor self, Tensor exponent) -> Tensor
9483: 
9484:         auto dispatch_float_power = [](const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
9485:           pybind11::gil_scoped_release no_gil;
9486:           return self.float_power(exponent);
9487:         };
9488:         return wrap(dispatch_float_power(_r.tensor(0), _r.tensor(1)));
9489:       } else {
9490:         // aten::float_power.Tensor_Tensor_out(Tensor self, Tensor exponent, *, Tensor(a!) out) -> Tensor(a!)
9491: 
9492:         auto dispatch_float_power_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
9493:           pybind11::gil_scoped_release no_gil;
9494:           return at::float_power_out(out, self, exponent);
9495:         };
9496:         return wrap(dispatch_float_power_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
9497:       }
9498:     }
9499:     case 1: {
9500:       if (_r.isNone(2)) {
9501:         // aten::float_power.Scalar(Scalar self, Tensor exponent) -> Tensor
9502: 
9503:         auto dispatch_float_power = [](const at::Scalar & self, const at::Tensor & exponent) -> at::Tensor {
9504:           pybind11::gil_scoped_release no_gil;
9505:           return at::float_power(self, exponent);
9506:         };
9507:         return wrap(dispatch_float_power(_r.scalar(0), _r.tensor(1)));
9508:       } else {
9509:         // aten::float_power.Scalar_out(Scalar self, Tensor exponent, *, Tensor(a!) out) -> Tensor(a!)
9510: 
9511:         auto dispatch_float_power_out = [](at::Tensor out, const at::Scalar & self, const at::Tensor & exponent) -> at::Tensor {
9512:           pybind11::gil_scoped_release no_gil;
9513:           return at::float_power_out(out, self, exponent);
9514:         };
9515:         return wrap(dispatch_float_power_out(_r.tensor(2), _r.scalar(0), _r.tensor(1)));
9516:       }
9517:     }
9518:     case 2: {
9519:       if (_r.isNone(2)) {
9520:         // aten::float_power.Tensor_Scalar(Tensor self, Scalar exponent) -> Tensor
```

- EN: The main execution path in this span is carried by `wrap`, `pow_out`, `THPVariable_float_power`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `pow_out`, `THPVariable_float_power` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9521-9600

```cpp
9521: 
9522:         auto dispatch_float_power = [](const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
9523:           pybind11::gil_scoped_release no_gil;
9524:           return self.float_power(exponent);
9525:         };
9526:         return wrap(dispatch_float_power(_r.tensor(0), _r.scalar(1)));
9527:       } else {
9528:         // aten::float_power.Tensor_Scalar_out(Tensor self, Scalar exponent, *, Tensor(a!) out) -> Tensor(a!)
9529: 
9530:         auto dispatch_float_power_out = [](at::Tensor out, const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
9531:           pybind11::gil_scoped_release no_gil;
9532:           return at::float_power_out(out, self, exponent);
9533:         };
9534:         return wrap(dispatch_float_power_out(_r.tensor(2), _r.tensor(0), _r.scalar(1)));
9535:       }
9536:     }
9537:   }
9538:   Py_RETURN_NONE;
9539:   END_HANDLE_TH_ERRORS
9540: }
9541: 
9542: \
9543: // _foreach_add
9544: static PyObject * THPVariable__foreach_add(PyObject* self_, PyObject* args, PyObject* kwargs)
9545: {
9546:   HANDLE_TH_ERRORS
9547:   static PythonArgParser parser({
9548:     "_foreach_add(TensorList self, Scalar scalar)",
9549:     "_foreach_add(TensorList self, ScalarList scalars)",
9550:     "_foreach_add(TensorList self, Tensor other, *, Scalar alpha=1)",
9551:     "_foreach_add(TensorList self, TensorList other, *, Scalar alpha=1)",
9552:   }, /*traceable=*/true);
9553: 
9554:   ParsedArgs<3> parsed_args;
9555:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9556:   if(_r.has_torch_function()) {
9557:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9558:   }
9559:   switch (_r.idx) {
9560:     case 0: {
9561:       // aten::_foreach_add.Scalar(Tensor[] self, Scalar scalar) -> Tensor[]
9562: 
9563:       auto dispatch__foreach_add = [](at::TensorList self, const at::Scalar & scalar) -> ::std::vector<at::Tensor> {
9564:         pybind11::gil_scoped_release no_gil;
9565:         return at::_foreach_add(self, scalar);
9566:       };
9567:       return wrap(dispatch__foreach_add(_r.tensorlist(0), _r.scalar(1)));
9568:     }
9569:     case 1: {
9570:       // aten::_foreach_add.ScalarList(Tensor[] self, Scalar[] scalars) -> Tensor[]
9571: 
9572:       auto dispatch__foreach_add = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
9573:         pybind11::gil_scoped_release no_gil;
9574:         return at::_foreach_add(self, scalars);
9575:       };
9576:       return wrap(dispatch__foreach_add(_r.tensorlist(0), _r.scalarlist(1)));
9577:     }
9578:     case 2: {
9579:       // aten::_foreach_add.Tensor(Tensor[] self, Tensor other, *, Scalar alpha=1) -> Tensor[]
9580: 
9581:       auto dispatch__foreach_add = [](at::TensorList self, const at::Tensor & other, const at::Scalar & alpha) -> ::std::vector<at::Tensor> {
9582:         pybind11::gil_scoped_release no_gil;
9583:         return at::_foreach_add(self, other, alpha);
9584:       };
9585:       return wrap(dispatch__foreach_add(_r.tensorlist(0), _r.tensor(1), _r.scalar(2)));
9586:     }
9587:     case 3: {
9588:       // aten::_foreach_add.List(Tensor[] self, Tensor[] other, *, Scalar alpha=1) -> Tensor[]
9589: 
9590:       auto dispatch__foreach_add = [](at::TensorList self, at::TensorList other, const at::Scalar & alpha) -> ::std::vector<at::Tensor> {
9591:         pybind11::gil_scoped_release no_gil;
9592:         return at::_foreach_add(self, other, alpha);
9593:       };
9594:       return wrap(dispatch__foreach_add(_r.tensorlist(0), _r.tensorlist(1), _r.scalar(2)));
9595:     }
9596:   }
9597:   Py_RETURN_NONE;
9598:   END_HANDLE_TH_ERRORS
9599: }
9600: 
```

- EN: The main execution path in this span is carried by `wrap`, `float_power_out`, `THPVariable__foreach_add`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `float_power_out`, `THPVariable__foreach_add` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9601-9680

```cpp
9601: \
9602: // _foreach_add_
9603: static PyObject * THPVariable__foreach_add_(PyObject* self_, PyObject* args, PyObject* kwargs)
9604: {
9605:   HANDLE_TH_ERRORS
9606:   static PythonArgParser parser({
9607:     "_foreach_add_(TensorList self, Scalar scalar)",
9608:     "_foreach_add_(TensorList self, ScalarList scalars)",
9609:     "_foreach_add_(TensorList self, Tensor other, *, Scalar alpha=1)",
9610:     "_foreach_add_(TensorList self, TensorList other, *, Scalar alpha=1)",
9611:   }, /*traceable=*/false);
9612: 
9613:   ParsedArgs<3> parsed_args;
9614:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9615:   if(_r.has_torch_function()) {
9616:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9617:   }
9618:   switch (_r.idx) {
9619:     case 0: {
9620:       // aten::_foreach_add_.Scalar(Tensor(a!)[] self, Scalar scalar) -> ()
9621: 
9622:       auto dispatch__foreach_add_ = [](at::TensorList self, const at::Scalar & scalar) -> void {
9623:         pybind11::gil_scoped_release no_gil;
9624:         at::_foreach_add_(self, scalar);
9625:       };
9626:       dispatch__foreach_add_(_r.tensorlist(0), _r.scalar(1));
9627:       PyObject* self_tensorlist = _r.args[0];
9628:       Py_INCREF(self_tensorlist);
9629:       return self_tensorlist;
9630:     }
9631:     case 1: {
9632:       // aten::_foreach_add_.ScalarList(Tensor(a!)[] self, Scalar[] scalars) -> ()
9633: 
9634:       auto dispatch__foreach_add_ = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> void {
9635:         pybind11::gil_scoped_release no_gil;
9636:         at::_foreach_add_(self, scalars);
9637:       };
9638:       dispatch__foreach_add_(_r.tensorlist(0), _r.scalarlist(1));
9639:       PyObject* self_tensorlist = _r.args[0];
9640:       Py_INCREF(self_tensorlist);
9641:       return self_tensorlist;
9642:     }
9643:     case 2: {
9644:       // aten::_foreach_add_.Tensor(Tensor(a!)[] self, Tensor other, *, Scalar alpha=1) -> ()
9645: 
9646:       auto dispatch__foreach_add_ = [](at::TensorList self, const at::Tensor & other, const at::Scalar & alpha) -> void {
9647:         pybind11::gil_scoped_release no_gil;
9648:         at::_foreach_add_(self, other, alpha);
9649:       };
9650:       dispatch__foreach_add_(_r.tensorlist(0), _r.tensor(1), _r.scalar(2));
9651:       PyObject* self_tensorlist = _r.args[0];
9652:       Py_INCREF(self_tensorlist);
9653:       return self_tensorlist;
9654:     }
9655:     case 3: {
9656:       // aten::_foreach_add_.List(Tensor(a!)[] self, Tensor[] other, *, Scalar alpha=1) -> ()
9657: 
9658:       auto dispatch__foreach_add_ = [](at::TensorList self, at::TensorList other, const at::Scalar & alpha) -> void {
9659:         pybind11::gil_scoped_release no_gil;
9660:         at::_foreach_add_(self, other, alpha);
9661:       };
9662:       dispatch__foreach_add_(_r.tensorlist(0), _r.tensorlist(1), _r.scalar(2));
9663:       PyObject* self_tensorlist = _r.args[0];
9664:       Py_INCREF(self_tensorlist);
9665:       return self_tensorlist;
9666:     }
9667:   }
9668:   Py_RETURN_NONE;
9669:   END_HANDLE_TH_ERRORS
9670: }
9671: 
9672: \
9673: // _foreach_sub
9674: static PyObject * THPVariable__foreach_sub(PyObject* self_, PyObject* args, PyObject* kwargs)
9675: {
9676:   HANDLE_TH_ERRORS
9677:   static PythonArgParser parser({
9678:     "_foreach_sub(TensorList self, Scalar scalar)",
9679:     "_foreach_sub(TensorList self, ScalarList scalars)",
9680:     "_foreach_sub(TensorList self, TensorList other, *, Scalar alpha=1)",
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_add_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_add_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9681-9760

```cpp
9681:   }, /*traceable=*/true);
9682: 
9683:   ParsedArgs<3> parsed_args;
9684:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9685:   if(_r.has_torch_function()) {
9686:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9687:   }
9688:   switch (_r.idx) {
9689:     case 0: {
9690:       // aten::_foreach_sub.Scalar(Tensor[] self, Scalar scalar) -> Tensor[]
9691: 
9692:       auto dispatch__foreach_sub = [](at::TensorList self, const at::Scalar & scalar) -> ::std::vector<at::Tensor> {
9693:         pybind11::gil_scoped_release no_gil;
9694:         return at::_foreach_sub(self, scalar);
9695:       };
9696:       return wrap(dispatch__foreach_sub(_r.tensorlist(0), _r.scalar(1)));
9697:     }
9698:     case 1: {
9699:       // aten::_foreach_sub.ScalarList(Tensor[] self, Scalar[] scalars) -> Tensor[]
9700: 
9701:       auto dispatch__foreach_sub = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
9702:         pybind11::gil_scoped_release no_gil;
9703:         return at::_foreach_sub(self, scalars);
9704:       };
9705:       return wrap(dispatch__foreach_sub(_r.tensorlist(0), _r.scalarlist(1)));
9706:     }
9707:     case 2: {
9708:       // aten::_foreach_sub.List(Tensor[] self, Tensor[] other, *, Scalar alpha=1) -> Tensor[]
9709: 
9710:       auto dispatch__foreach_sub = [](at::TensorList self, at::TensorList other, const at::Scalar & alpha) -> ::std::vector<at::Tensor> {
9711:         pybind11::gil_scoped_release no_gil;
9712:         return at::_foreach_sub(self, other, alpha);
9713:       };
9714:       return wrap(dispatch__foreach_sub(_r.tensorlist(0), _r.tensorlist(1), _r.scalar(2)));
9715:     }
9716:   }
9717:   Py_RETURN_NONE;
9718:   END_HANDLE_TH_ERRORS
9719: }
9720: 
9721: \
9722: // _foreach_sub_
9723: static PyObject * THPVariable__foreach_sub_(PyObject* self_, PyObject* args, PyObject* kwargs)
9724: {
9725:   HANDLE_TH_ERRORS
9726:   static PythonArgParser parser({
9727:     "_foreach_sub_(TensorList self, Scalar scalar)",
9728:     "_foreach_sub_(TensorList self, ScalarList scalars)",
9729:     "_foreach_sub_(TensorList self, TensorList other, *, Scalar alpha=1)",
9730:   }, /*traceable=*/false);
9731: 
9732:   ParsedArgs<3> parsed_args;
9733:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9734:   if(_r.has_torch_function()) {
9735:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9736:   }
9737:   switch (_r.idx) {
9738:     case 0: {
9739:       // aten::_foreach_sub_.Scalar(Tensor(a!)[] self, Scalar scalar) -> ()
9740: 
9741:       auto dispatch__foreach_sub_ = [](at::TensorList self, const at::Scalar & scalar) -> void {
9742:         pybind11::gil_scoped_release no_gil;
9743:         at::_foreach_sub_(self, scalar);
9744:       };
9745:       dispatch__foreach_sub_(_r.tensorlist(0), _r.scalar(1));
9746:       PyObject* self_tensorlist = _r.args[0];
9747:       Py_INCREF(self_tensorlist);
9748:       return self_tensorlist;
9749:     }
9750:     case 1: {
9751:       // aten::_foreach_sub_.ScalarList(Tensor(a!)[] self, Scalar[] scalars) -> ()
9752: 
9753:       auto dispatch__foreach_sub_ = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> void {
9754:         pybind11::gil_scoped_release no_gil;
9755:         at::_foreach_sub_(self, scalars);
9756:       };
9757:       dispatch__foreach_sub_(_r.tensorlist(0), _r.scalarlist(1));
9758:       PyObject* self_tensorlist = _r.args[0];
9759:       Py_INCREF(self_tensorlist);
9760:       return self_tensorlist;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_foreach_sub`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_foreach_sub`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9761-9840

```cpp
9761:     }
9762:     case 2: {
9763:       // aten::_foreach_sub_.List(Tensor(a!)[] self, Tensor[] other, *, Scalar alpha=1) -> ()
9764: 
9765:       auto dispatch__foreach_sub_ = [](at::TensorList self, at::TensorList other, const at::Scalar & alpha) -> void {
9766:         pybind11::gil_scoped_release no_gil;
9767:         at::_foreach_sub_(self, other, alpha);
9768:       };
9769:       dispatch__foreach_sub_(_r.tensorlist(0), _r.tensorlist(1), _r.scalar(2));
9770:       PyObject* self_tensorlist = _r.args[0];
9771:       Py_INCREF(self_tensorlist);
9772:       return self_tensorlist;
9773:     }
9774:   }
9775:   Py_RETURN_NONE;
9776:   END_HANDLE_TH_ERRORS
9777: }
9778: 
9779: \
9780: // _foreach_clamp_max
9781: static PyObject * THPVariable__foreach_clamp_max(PyObject* self_, PyObject* args, PyObject* kwargs)
9782: {
9783:   HANDLE_TH_ERRORS
9784:   static PythonArgParser parser({
9785:     "_foreach_clamp_max(TensorList self, Scalar scalar)",
9786:     "_foreach_clamp_max(TensorList self, ScalarList scalars)",
9787:     "_foreach_clamp_max(TensorList self, TensorList other)",
9788:   }, /*traceable=*/true);
9789: 
9790:   ParsedArgs<2> parsed_args;
9791:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9792:   if(_r.has_torch_function()) {
9793:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9794:   }
9795:   switch (_r.idx) {
9796:     case 0: {
9797:       // aten::_foreach_clamp_max.Scalar(Tensor[] self, Scalar scalar) -> Tensor[]
9798: 
9799:       auto dispatch__foreach_clamp_max = [](at::TensorList self, const at::Scalar & scalar) -> ::std::vector<at::Tensor> {
9800:         pybind11::gil_scoped_release no_gil;
9801:         return at::_foreach_clamp_max(self, scalar);
9802:       };
9803:       return wrap(dispatch__foreach_clamp_max(_r.tensorlist(0), _r.scalar(1)));
9804:     }
9805:     case 1: {
9806:       // aten::_foreach_clamp_max.ScalarList(Tensor[] self, Scalar[] scalars) -> Tensor[]
9807: 
9808:       auto dispatch__foreach_clamp_max = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> ::std::vector<at::Tensor> {
9809:         pybind11::gil_scoped_release no_gil;
9810:         return at::_foreach_clamp_max(self, scalars);
9811:       };
9812:       return wrap(dispatch__foreach_clamp_max(_r.tensorlist(0), _r.scalarlist(1)));
9813:     }
9814:     case 2: {
9815:       // aten::_foreach_clamp_max.List(Tensor[] self, Tensor[] other) -> Tensor[]
9816: 
9817:       auto dispatch__foreach_clamp_max = [](at::TensorList self, at::TensorList other) -> ::std::vector<at::Tensor> {
9818:         pybind11::gil_scoped_release no_gil;
9819:         return at::_foreach_clamp_max(self, other);
9820:       };
9821:       return wrap(dispatch__foreach_clamp_max(_r.tensorlist(0), _r.tensorlist(1)));
9822:     }
9823:   }
9824:   Py_RETURN_NONE;
9825:   END_HANDLE_TH_ERRORS
9826: }
9827: 
9828: \
9829: // _foreach_clamp_max_
9830: static PyObject * THPVariable__foreach_clamp_max_(PyObject* self_, PyObject* args, PyObject* kwargs)
9831: {
9832:   HANDLE_TH_ERRORS
9833:   static PythonArgParser parser({
9834:     "_foreach_clamp_max_(TensorList self, Scalar scalar)",
9835:     "_foreach_clamp_max_(TensorList self, ScalarList scalars)",
9836:     "_foreach_clamp_max_(TensorList self, TensorList other)",
9837:   }, /*traceable=*/false);
9838: 
9839:   ParsedArgs<2> parsed_args;
9840:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `_foreach_sub_`, `dispatch__foreach_sub_`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_foreach_sub_`, `dispatch__foreach_sub_`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9841-9920

```cpp
9841:   if(_r.has_torch_function()) {
9842:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9843:   }
9844:   switch (_r.idx) {
9845:     case 0: {
9846:       // aten::_foreach_clamp_max_.Scalar(Tensor(a!)[] self, Scalar scalar) -> ()
9847: 
9848:       auto dispatch__foreach_clamp_max_ = [](at::TensorList self, const at::Scalar & scalar) -> void {
9849:         pybind11::gil_scoped_release no_gil;
9850:         at::_foreach_clamp_max_(self, scalar);
9851:       };
9852:       dispatch__foreach_clamp_max_(_r.tensorlist(0), _r.scalar(1));
9853:       PyObject* self_tensorlist = _r.args[0];
9854:       Py_INCREF(self_tensorlist);
9855:       return self_tensorlist;
9856:     }
9857:     case 1: {
9858:       // aten::_foreach_clamp_max_.ScalarList(Tensor(a!)[] self, Scalar[] scalars) -> ()
9859: 
9860:       auto dispatch__foreach_clamp_max_ = [](at::TensorList self, at::ArrayRef<at::Scalar> scalars) -> void {
9861:         pybind11::gil_scoped_release no_gil;
9862:         at::_foreach_clamp_max_(self, scalars);
9863:       };
9864:       dispatch__foreach_clamp_max_(_r.tensorlist(0), _r.scalarlist(1));
9865:       PyObject* self_tensorlist = _r.args[0];
9866:       Py_INCREF(self_tensorlist);
9867:       return self_tensorlist;
9868:     }
9869:     case 2: {
9870:       // aten::_foreach_clamp_max_.List(Tensor(a!)[] self, Tensor[] other) -> ()
9871: 
9872:       auto dispatch__foreach_clamp_max_ = [](at::TensorList self, at::TensorList other) -> void {
9873:         pybind11::gil_scoped_release no_gil;
9874:         at::_foreach_clamp_max_(self, other);
9875:       };
9876:       dispatch__foreach_clamp_max_(_r.tensorlist(0), _r.tensorlist(1));
9877:       PyObject* self_tensorlist = _r.args[0];
9878:       Py_INCREF(self_tensorlist);
9879:       return self_tensorlist;
9880:     }
9881:   }
9882:   Py_RETURN_NONE;
9883:   END_HANDLE_TH_ERRORS
9884: }
9885: 
9886: // _foreach_acos
9887: static PyObject * THPVariable__foreach_acos(PyObject* self_, PyObject* args, PyObject* kwargs)
9888: {
9889:   HANDLE_TH_ERRORS
9890:   static PythonArgParser parser({
9891:     "_foreach_acos(TensorList self)",
9892:   }, /*traceable=*/true);
9893: 
9894:   ParsedArgs<1> parsed_args;
9895:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9896:   if(_r.has_torch_function()) {
9897:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
9898:   }
9899:   // aten::_foreach_acos(Tensor[] self) -> Tensor[]
9900: 
9901:   auto dispatch__foreach_acos = [](at::TensorList self) -> ::std::vector<at::Tensor> {
9902:     pybind11::gil_scoped_release no_gil;
9903:     return at::_foreach_acos(self);
9904:   };
9905:   return wrap(dispatch__foreach_acos(_r.tensorlist(0)));
9906:   Py_RETURN_NONE;
9907:   END_HANDLE_TH_ERRORS
9908: }
9909: 
9910: // _foreach_acos_
9911: static PyObject * THPVariable__foreach_acos_(PyObject* self_, PyObject* args, PyObject* kwargs)
9912: {
9913:   HANDLE_TH_ERRORS
9914:   static PythonArgParser parser({
9915:     "_foreach_acos_(TensorList self)",
9916:   }, /*traceable=*/false);
9917: 
9918:   ParsedArgs<1> parsed_args;
9919:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
9920:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_foreach_clamp_max_`, `dispatch__foreach_clamp_max_`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_foreach_clamp_max_`, `dispatch__foreach_clamp_max_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 9921-10000

```cpp
 9921:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
 9922:   }
 9923:   // aten::_foreach_acos_(Tensor(a!)[] self) -> ()
 9924: 
 9925:   auto dispatch__foreach_acos_ = [](at::TensorList self) -> void {
 9926:     pybind11::gil_scoped_release no_gil;
 9927:     at::_foreach_acos_(self);
 9928:   };
 9929:   dispatch__foreach_acos_(_r.tensorlist(0));
 9930:   PyObject* self_tensorlist = _r.args[0];
 9931:   Py_INCREF(self_tensorlist);
 9932:   return self_tensorlist;
 9933:   Py_RETURN_NONE;
 9934:   END_HANDLE_TH_ERRORS
 9935: }
 9936: 
 9937: // _foreach_asin
 9938: static PyObject * THPVariable__foreach_asin(PyObject* self_, PyObject* args, PyObject* kwargs)
 9939: {
 9940:   HANDLE_TH_ERRORS
 9941:   static PythonArgParser parser({
 9942:     "_foreach_asin(TensorList self)",
 9943:   }, /*traceable=*/true);
 9944: 
 9945:   ParsedArgs<1> parsed_args;
 9946:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
 9947:   if(_r.has_torch_function()) {
 9948:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
 9949:   }
 9950:   // aten::_foreach_asin(Tensor[] self) -> Tensor[]
 9951: 
 9952:   auto dispatch__foreach_asin = [](at::TensorList self) -> ::std::vector<at::Tensor> {
 9953:     pybind11::gil_scoped_release no_gil;
 9954:     return at::_foreach_asin(self);
 9955:   };
 9956:   return wrap(dispatch__foreach_asin(_r.tensorlist(0)));
 9957:   Py_RETURN_NONE;
 9958:   END_HANDLE_TH_ERRORS
 9959: }
 9960: 
 9961: // _foreach_asin_
 9962: static PyObject * THPVariable__foreach_asin_(PyObject* self_, PyObject* args, PyObject* kwargs)
 9963: {
 9964:   HANDLE_TH_ERRORS
 9965:   static PythonArgParser parser({
 9966:     "_foreach_asin_(TensorList self)",
 9967:   }, /*traceable=*/false);
 9968: 
 9969:   ParsedArgs<1> parsed_args;
 9970:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
 9971:   if(_r.has_torch_function()) {
 9972:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
 9973:   }
 9974:   // aten::_foreach_asin_(Tensor(a!)[] self) -> ()
 9975: 
 9976:   auto dispatch__foreach_asin_ = [](at::TensorList self) -> void {
 9977:     pybind11::gil_scoped_release no_gil;
 9978:     at::_foreach_asin_(self);
 9979:   };
 9980:   dispatch__foreach_asin_(_r.tensorlist(0));
 9981:   PyObject* self_tensorlist = _r.args[0];
 9982:   Py_INCREF(self_tensorlist);
 9983:   return self_tensorlist;
 9984:   Py_RETURN_NONE;
 9985:   END_HANDLE_TH_ERRORS
 9986: }
 9987: 
 9988: // _foreach_cos
 9989: static PyObject * THPVariable__foreach_cos(PyObject* self_, PyObject* args, PyObject* kwargs)
 9990: {
 9991:   HANDLE_TH_ERRORS
 9992:   static PythonArgParser parser({
 9993:     "_foreach_cos(TensorList self)",
 9994:   }, /*traceable=*/true);
 9995: 
 9996:   ParsedArgs<1> parsed_args;
 9997:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
 9998:   if(_r.has_torch_function()) {
 9999:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10000:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_foreach_acos_`, `dispatch__foreach_acos_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_foreach_acos_`, `dispatch__foreach_acos_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10001-10080

```cpp
10001:   // aten::_foreach_cos(Tensor[] self) -> Tensor[]
10002: 
10003:   auto dispatch__foreach_cos = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10004:     pybind11::gil_scoped_release no_gil;
10005:     return at::_foreach_cos(self);
10006:   };
10007:   return wrap(dispatch__foreach_cos(_r.tensorlist(0)));
10008:   Py_RETURN_NONE;
10009:   END_HANDLE_TH_ERRORS
10010: }
10011: 
10012: // _foreach_cos_
10013: static PyObject * THPVariable__foreach_cos_(PyObject* self_, PyObject* args, PyObject* kwargs)
10014: {
10015:   HANDLE_TH_ERRORS
10016:   static PythonArgParser parser({
10017:     "_foreach_cos_(TensorList self)",
10018:   }, /*traceable=*/false);
10019: 
10020:   ParsedArgs<1> parsed_args;
10021:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10022:   if(_r.has_torch_function()) {
10023:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10024:   }
10025:   // aten::_foreach_cos_(Tensor(a!)[] self) -> ()
10026: 
10027:   auto dispatch__foreach_cos_ = [](at::TensorList self) -> void {
10028:     pybind11::gil_scoped_release no_gil;
10029:     at::_foreach_cos_(self);
10030:   };
10031:   dispatch__foreach_cos_(_r.tensorlist(0));
10032:   PyObject* self_tensorlist = _r.args[0];
10033:   Py_INCREF(self_tensorlist);
10034:   return self_tensorlist;
10035:   Py_RETURN_NONE;
10036:   END_HANDLE_TH_ERRORS
10037: }
10038: 
10039: // _foreach_cosh
10040: static PyObject * THPVariable__foreach_cosh(PyObject* self_, PyObject* args, PyObject* kwargs)
10041: {
10042:   HANDLE_TH_ERRORS
10043:   static PythonArgParser parser({
10044:     "_foreach_cosh(TensorList self)",
10045:   }, /*traceable=*/true);
10046: 
10047:   ParsedArgs<1> parsed_args;
10048:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10049:   if(_r.has_torch_function()) {
10050:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10051:   }
10052:   // aten::_foreach_cosh(Tensor[] self) -> Tensor[]
10053: 
10054:   auto dispatch__foreach_cosh = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10055:     pybind11::gil_scoped_release no_gil;
10056:     return at::_foreach_cosh(self);
10057:   };
10058:   return wrap(dispatch__foreach_cosh(_r.tensorlist(0)));
10059:   Py_RETURN_NONE;
10060:   END_HANDLE_TH_ERRORS
10061: }
10062: 
10063: // _foreach_cosh_
10064: static PyObject * THPVariable__foreach_cosh_(PyObject* self_, PyObject* args, PyObject* kwargs)
10065: {
10066:   HANDLE_TH_ERRORS
10067:   static PythonArgParser parser({
10068:     "_foreach_cosh_(TensorList self)",
10069:   }, /*traceable=*/false);
10070: 
10071:   ParsedArgs<1> parsed_args;
10072:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10073:   if(_r.has_torch_function()) {
10074:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10075:   }
10076:   // aten::_foreach_cosh_(Tensor(a!)[] self) -> ()
10077: 
10078:   auto dispatch__foreach_cosh_ = [](at::TensorList self) -> void {
10079:     pybind11::gil_scoped_release no_gil;
10080:     at::_foreach_cosh_(self);
```

- EN: The main execution path in this span is carried by `_foreach_cos`, `wrap`, `THPVariable__foreach_cos_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_foreach_cos`, `wrap`, `THPVariable__foreach_cos_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10081-10160

```cpp
10081:   };
10082:   dispatch__foreach_cosh_(_r.tensorlist(0));
10083:   PyObject* self_tensorlist = _r.args[0];
10084:   Py_INCREF(self_tensorlist);
10085:   return self_tensorlist;
10086:   Py_RETURN_NONE;
10087:   END_HANDLE_TH_ERRORS
10088: }
10089: 
10090: // _foreach_exp
10091: static PyObject * THPVariable__foreach_exp(PyObject* self_, PyObject* args, PyObject* kwargs)
10092: {
10093:   HANDLE_TH_ERRORS
10094:   static PythonArgParser parser({
10095:     "_foreach_exp(TensorList self)",
10096:   }, /*traceable=*/true);
10097: 
10098:   ParsedArgs<1> parsed_args;
10099:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10100:   if(_r.has_torch_function()) {
10101:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10102:   }
10103:   // aten::_foreach_exp(Tensor[] self) -> Tensor[]
10104: 
10105:   auto dispatch__foreach_exp = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10106:     pybind11::gil_scoped_release no_gil;
10107:     return at::_foreach_exp(self);
10108:   };
10109:   return wrap(dispatch__foreach_exp(_r.tensorlist(0)));
10110:   Py_RETURN_NONE;
10111:   END_HANDLE_TH_ERRORS
10112: }
10113: 
10114: // _foreach_exp_
10115: static PyObject * THPVariable__foreach_exp_(PyObject* self_, PyObject* args, PyObject* kwargs)
10116: {
10117:   HANDLE_TH_ERRORS
10118:   static PythonArgParser parser({
10119:     "_foreach_exp_(TensorList self)",
10120:   }, /*traceable=*/false);
10121: 
10122:   ParsedArgs<1> parsed_args;
10123:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10124:   if(_r.has_torch_function()) {
10125:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10126:   }
10127:   // aten::_foreach_exp_(Tensor(a!)[] self) -> ()
10128: 
10129:   auto dispatch__foreach_exp_ = [](at::TensorList self) -> void {
10130:     pybind11::gil_scoped_release no_gil;
10131:     at::_foreach_exp_(self);
10132:   };
10133:   dispatch__foreach_exp_(_r.tensorlist(0));
10134:   PyObject* self_tensorlist = _r.args[0];
10135:   Py_INCREF(self_tensorlist);
10136:   return self_tensorlist;
10137:   Py_RETURN_NONE;
10138:   END_HANDLE_TH_ERRORS
10139: }
10140: 
10141: // _foreach_floor
10142: static PyObject * THPVariable__foreach_floor(PyObject* self_, PyObject* args, PyObject* kwargs)
10143: {
10144:   HANDLE_TH_ERRORS
10145:   static PythonArgParser parser({
10146:     "_foreach_floor(TensorList self)",
10147:   }, /*traceable=*/true);
10148: 
10149:   ParsedArgs<1> parsed_args;
10150:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10151:   if(_r.has_torch_function()) {
10152:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10153:   }
10154:   // aten::_foreach_floor(Tensor[] self) -> Tensor[]
10155: 
10156:   auto dispatch__foreach_floor = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10157:     pybind11::gil_scoped_release no_gil;
10158:     return at::_foreach_floor(self);
10159:   };
10160:   return wrap(dispatch__foreach_floor(_r.tensorlist(0)));
```

- EN: The main execution path in this span is carried by `dispatch__foreach_cosh_`, `Py_INCREF`, `THPVariable__foreach_exp`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `dispatch__foreach_cosh_`, `Py_INCREF`, `THPVariable__foreach_exp` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10161-10240

```cpp
10161:   Py_RETURN_NONE;
10162:   END_HANDLE_TH_ERRORS
10163: }
10164: 
10165: // _foreach_floor_
10166: static PyObject * THPVariable__foreach_floor_(PyObject* self_, PyObject* args, PyObject* kwargs)
10167: {
10168:   HANDLE_TH_ERRORS
10169:   static PythonArgParser parser({
10170:     "_foreach_floor_(TensorList self)",
10171:   }, /*traceable=*/false);
10172: 
10173:   ParsedArgs<1> parsed_args;
10174:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10175:   if(_r.has_torch_function()) {
10176:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10177:   }
10178:   // aten::_foreach_floor_(Tensor(a!)[] self) -> ()
10179: 
10180:   auto dispatch__foreach_floor_ = [](at::TensorList self) -> void {
10181:     pybind11::gil_scoped_release no_gil;
10182:     at::_foreach_floor_(self);
10183:   };
10184:   dispatch__foreach_floor_(_r.tensorlist(0));
10185:   PyObject* self_tensorlist = _r.args[0];
10186:   Py_INCREF(self_tensorlist);
10187:   return self_tensorlist;
10188:   Py_RETURN_NONE;
10189:   END_HANDLE_TH_ERRORS
10190: }
10191: 
10192: // _foreach_log1p
10193: static PyObject * THPVariable__foreach_log1p(PyObject* self_, PyObject* args, PyObject* kwargs)
10194: {
10195:   HANDLE_TH_ERRORS
10196:   static PythonArgParser parser({
10197:     "_foreach_log1p(TensorList self)",
10198:   }, /*traceable=*/true);
10199: 
10200:   ParsedArgs<1> parsed_args;
10201:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10202:   if(_r.has_torch_function()) {
10203:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10204:   }
10205:   // aten::_foreach_log1p(Tensor[] self) -> Tensor[]
10206: 
10207:   auto dispatch__foreach_log1p = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10208:     pybind11::gil_scoped_release no_gil;
10209:     return at::_foreach_log1p(self);
10210:   };
10211:   return wrap(dispatch__foreach_log1p(_r.tensorlist(0)));
10212:   Py_RETURN_NONE;
10213:   END_HANDLE_TH_ERRORS
10214: }
10215: 
10216: // _foreach_log1p_
10217: static PyObject * THPVariable__foreach_log1p_(PyObject* self_, PyObject* args, PyObject* kwargs)
10218: {
10219:   HANDLE_TH_ERRORS
10220:   static PythonArgParser parser({
10221:     "_foreach_log1p_(TensorList self)",
10222:   }, /*traceable=*/false);
10223: 
10224:   ParsedArgs<1> parsed_args;
10225:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10226:   if(_r.has_torch_function()) {
10227:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10228:   }
10229:   // aten::_foreach_log1p_(Tensor(a!)[] self) -> ()
10230: 
10231:   auto dispatch__foreach_log1p_ = [](at::TensorList self) -> void {
10232:     pybind11::gil_scoped_release no_gil;
10233:     at::_foreach_log1p_(self);
10234:   };
10235:   dispatch__foreach_log1p_(_r.tensorlist(0));
10236:   PyObject* self_tensorlist = _r.args[0];
10237:   Py_INCREF(self_tensorlist);
10238:   return self_tensorlist;
10239:   Py_RETURN_NONE;
10240:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_floor_`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_floor_`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10241-10320

```cpp
10241: }
10242: 
10243: // _foreach_log2
10244: static PyObject * THPVariable__foreach_log2(PyObject* self_, PyObject* args, PyObject* kwargs)
10245: {
10246:   HANDLE_TH_ERRORS
10247:   static PythonArgParser parser({
10248:     "_foreach_log2(TensorList self)",
10249:   }, /*traceable=*/true);
10250: 
10251:   ParsedArgs<1> parsed_args;
10252:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10253:   if(_r.has_torch_function()) {
10254:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10255:   }
10256:   // aten::_foreach_log2(Tensor[] self) -> Tensor[]
10257: 
10258:   auto dispatch__foreach_log2 = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10259:     pybind11::gil_scoped_release no_gil;
10260:     return at::_foreach_log2(self);
10261:   };
10262:   return wrap(dispatch__foreach_log2(_r.tensorlist(0)));
10263:   Py_RETURN_NONE;
10264:   END_HANDLE_TH_ERRORS
10265: }
10266: 
10267: // _foreach_log2_
10268: static PyObject * THPVariable__foreach_log2_(PyObject* self_, PyObject* args, PyObject* kwargs)
10269: {
10270:   HANDLE_TH_ERRORS
10271:   static PythonArgParser parser({
10272:     "_foreach_log2_(TensorList self)",
10273:   }, /*traceable=*/false);
10274: 
10275:   ParsedArgs<1> parsed_args;
10276:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10277:   if(_r.has_torch_function()) {
10278:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10279:   }
10280:   // aten::_foreach_log2_(Tensor(a!)[] self) -> ()
10281: 
10282:   auto dispatch__foreach_log2_ = [](at::TensorList self) -> void {
10283:     pybind11::gil_scoped_release no_gil;
10284:     at::_foreach_log2_(self);
10285:   };
10286:   dispatch__foreach_log2_(_r.tensorlist(0));
10287:   PyObject* self_tensorlist = _r.args[0];
10288:   Py_INCREF(self_tensorlist);
10289:   return self_tensorlist;
10290:   Py_RETURN_NONE;
10291:   END_HANDLE_TH_ERRORS
10292: }
10293: 
10294: // _foreach_neg
10295: static PyObject * THPVariable__foreach_neg(PyObject* self_, PyObject* args, PyObject* kwargs)
10296: {
10297:   HANDLE_TH_ERRORS
10298:   static PythonArgParser parser({
10299:     "_foreach_neg(TensorList self)",
10300:   }, /*traceable=*/true);
10301: 
10302:   ParsedArgs<1> parsed_args;
10303:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10304:   if(_r.has_torch_function()) {
10305:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10306:   }
10307:   // aten::_foreach_neg(Tensor[] self) -> Tensor[]
10308: 
10309:   auto dispatch__foreach_neg = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10310:     pybind11::gil_scoped_release no_gil;
10311:     return at::_foreach_neg(self);
10312:   };
10313:   return wrap(dispatch__foreach_neg(_r.tensorlist(0)));
10314:   Py_RETURN_NONE;
10315:   END_HANDLE_TH_ERRORS
10316: }
10317: 
10318: // _foreach_neg_
10319: static PyObject * THPVariable__foreach_neg_(PyObject* self_, PyObject* args, PyObject* kwargs)
10320: {
```

- EN: The main execution path in this span is carried by `THPVariable__foreach_log2`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__foreach_log2`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10321-10400

```cpp
10321:   HANDLE_TH_ERRORS
10322:   static PythonArgParser parser({
10323:     "_foreach_neg_(TensorList self)",
10324:   }, /*traceable=*/false);
10325: 
10326:   ParsedArgs<1> parsed_args;
10327:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10328:   if(_r.has_torch_function()) {
10329:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10330:   }
10331:   // aten::_foreach_neg_(Tensor(a!)[] self) -> ()
10332: 
10333:   auto dispatch__foreach_neg_ = [](at::TensorList self) -> void {
10334:     pybind11::gil_scoped_release no_gil;
10335:     at::_foreach_neg_(self);
10336:   };
10337:   dispatch__foreach_neg_(_r.tensorlist(0));
10338:   PyObject* self_tensorlist = _r.args[0];
10339:   Py_INCREF(self_tensorlist);
10340:   return self_tensorlist;
10341:   Py_RETURN_NONE;
10342:   END_HANDLE_TH_ERRORS
10343: }
10344: 
10345: // _foreach_norm
10346: static PyObject * THPVariable__foreach_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
10347: {
10348:   HANDLE_TH_ERRORS
10349:   static PythonArgParser parser({
10350:     "_foreach_norm(TensorList self, Scalar ord=2, ScalarType? dtype=None)",
10351:   }, /*traceable=*/true);
10352: 
10353:   ParsedArgs<3> parsed_args;
10354:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10355:   if(_r.has_torch_function()) {
10356:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10357:   }
10358:   // aten::_foreach_norm.Scalar(Tensor[] self, Scalar ord=2, ScalarType? dtype=None) -> Tensor[]
10359: 
10360:   auto dispatch__foreach_norm = [](at::TensorList self, const at::Scalar & ord, ::std::optional<at::ScalarType> dtype) -> ::std::vector<at::Tensor> {
10361:     pybind11::gil_scoped_release no_gil;
10362:     return at::_foreach_norm(self, ord, dtype);
10363:   };
10364:   return wrap(dispatch__foreach_norm(_r.tensorlist(0), _r.scalar(1), _r.scalartypeOptional(2)));
10365:   Py_RETURN_NONE;
10366:   END_HANDLE_TH_ERRORS
10367: }
10368: 
10369: // _foreach_sigmoid
10370: static PyObject * THPVariable__foreach_sigmoid(PyObject* self_, PyObject* args, PyObject* kwargs)
10371: {
10372:   HANDLE_TH_ERRORS
10373:   static PythonArgParser parser({
10374:     "_foreach_sigmoid(TensorList self)",
10375:   }, /*traceable=*/true);
10376: 
10377:   ParsedArgs<1> parsed_args;
10378:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10379:   if(_r.has_torch_function()) {
10380:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10381:   }
10382:   // aten::_foreach_sigmoid(Tensor[] self) -> Tensor[]
10383: 
10384:   auto dispatch__foreach_sigmoid = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10385:     pybind11::gil_scoped_release no_gil;
10386:     return at::_foreach_sigmoid(self);
10387:   };
10388:   return wrap(dispatch__foreach_sigmoid(_r.tensorlist(0)));
10389:   Py_RETURN_NONE;
10390:   END_HANDLE_TH_ERRORS
10391: }
10392: 
10393: // _foreach_sigmoid_
10394: static PyObject * THPVariable__foreach_sigmoid_(PyObject* self_, PyObject* args, PyObject* kwargs)
10395: {
10396:   HANDLE_TH_ERRORS
10397:   static PythonArgParser parser({
10398:     "_foreach_sigmoid_(TensorList self)",
10399:   }, /*traceable=*/false);
10400: 
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_foreach_neg_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_foreach_neg_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10401-10480

```cpp
10401:   ParsedArgs<1> parsed_args;
10402:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10403:   if(_r.has_torch_function()) {
10404:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10405:   }
10406:   // aten::_foreach_sigmoid_(Tensor(a!)[] self) -> ()
10407: 
10408:   auto dispatch__foreach_sigmoid_ = [](at::TensorList self) -> void {
10409:     pybind11::gil_scoped_release no_gil;
10410:     at::_foreach_sigmoid_(self);
10411:   };
10412:   dispatch__foreach_sigmoid_(_r.tensorlist(0));
10413:   PyObject* self_tensorlist = _r.args[0];
10414:   Py_INCREF(self_tensorlist);
10415:   return self_tensorlist;
10416:   Py_RETURN_NONE;
10417:   END_HANDLE_TH_ERRORS
10418: }
10419: 
10420: // _foreach_sign
10421: static PyObject * THPVariable__foreach_sign(PyObject* self_, PyObject* args, PyObject* kwargs)
10422: {
10423:   HANDLE_TH_ERRORS
10424:   static PythonArgParser parser({
10425:     "_foreach_sign(TensorList self)",
10426:   }, /*traceable=*/true);
10427: 
10428:   ParsedArgs<1> parsed_args;
10429:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10430:   if(_r.has_torch_function()) {
10431:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10432:   }
10433:   // aten::_foreach_sign(Tensor[] self) -> Tensor[]
10434: 
10435:   auto dispatch__foreach_sign = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10436:     pybind11::gil_scoped_release no_gil;
10437:     return at::_foreach_sign(self);
10438:   };
10439:   return wrap(dispatch__foreach_sign(_r.tensorlist(0)));
10440:   Py_RETURN_NONE;
10441:   END_HANDLE_TH_ERRORS
10442: }
10443: 
10444: // _foreach_sign_
10445: static PyObject * THPVariable__foreach_sign_(PyObject* self_, PyObject* args, PyObject* kwargs)
10446: {
10447:   HANDLE_TH_ERRORS
10448:   static PythonArgParser parser({
10449:     "_foreach_sign_(TensorList self)",
10450:   }, /*traceable=*/false);
10451: 
10452:   ParsedArgs<1> parsed_args;
10453:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10454:   if(_r.has_torch_function()) {
10455:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10456:   }
10457:   // aten::_foreach_sign_(Tensor(a!)[] self) -> ()
10458: 
10459:   auto dispatch__foreach_sign_ = [](at::TensorList self) -> void {
10460:     pybind11::gil_scoped_release no_gil;
10461:     at::_foreach_sign_(self);
10462:   };
10463:   dispatch__foreach_sign_(_r.tensorlist(0));
10464:   PyObject* self_tensorlist = _r.args[0];
10465:   Py_INCREF(self_tensorlist);
10466:   return self_tensorlist;
10467:   Py_RETURN_NONE;
10468:   END_HANDLE_TH_ERRORS
10469: }
10470: 
10471: // _foreach_sqrt
10472: static PyObject * THPVariable__foreach_sqrt(PyObject* self_, PyObject* args, PyObject* kwargs)
10473: {
10474:   HANDLE_TH_ERRORS
10475:   static PythonArgParser parser({
10476:     "_foreach_sqrt(TensorList self)",
10477:   }, /*traceable=*/true);
10478: 
10479:   ParsedArgs<1> parsed_args;
10480:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_foreach_sigmoid_`, `dispatch__foreach_sigmoid_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_foreach_sigmoid_`, `dispatch__foreach_sigmoid_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10481-10560

```cpp
10481:   if(_r.has_torch_function()) {
10482:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10483:   }
10484:   // aten::_foreach_sqrt(Tensor[] self) -> Tensor[]
10485: 
10486:   auto dispatch__foreach_sqrt = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10487:     pybind11::gil_scoped_release no_gil;
10488:     return at::_foreach_sqrt(self);
10489:   };
10490:   return wrap(dispatch__foreach_sqrt(_r.tensorlist(0)));
10491:   Py_RETURN_NONE;
10492:   END_HANDLE_TH_ERRORS
10493: }
10494: 
10495: // _foreach_sqrt_
10496: static PyObject * THPVariable__foreach_sqrt_(PyObject* self_, PyObject* args, PyObject* kwargs)
10497: {
10498:   HANDLE_TH_ERRORS
10499:   static PythonArgParser parser({
10500:     "_foreach_sqrt_(TensorList self)",
10501:   }, /*traceable=*/false);
10502: 
10503:   ParsedArgs<1> parsed_args;
10504:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10505:   if(_r.has_torch_function()) {
10506:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10507:   }
10508:   // aten::_foreach_sqrt_(Tensor(a!)[] self) -> ()
10509: 
10510:   auto dispatch__foreach_sqrt_ = [](at::TensorList self) -> void {
10511:     pybind11::gil_scoped_release no_gil;
10512:     at::_foreach_sqrt_(self);
10513:   };
10514:   dispatch__foreach_sqrt_(_r.tensorlist(0));
10515:   PyObject* self_tensorlist = _r.args[0];
10516:   Py_INCREF(self_tensorlist);
10517:   return self_tensorlist;
10518:   Py_RETURN_NONE;
10519:   END_HANDLE_TH_ERRORS
10520: }
10521: 
10522: // _foreach_tanh
10523: static PyObject * THPVariable__foreach_tanh(PyObject* self_, PyObject* args, PyObject* kwargs)
10524: {
10525:   HANDLE_TH_ERRORS
10526:   static PythonArgParser parser({
10527:     "_foreach_tanh(TensorList self)",
10528:   }, /*traceable=*/true);
10529: 
10530:   ParsedArgs<1> parsed_args;
10531:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10532:   if(_r.has_torch_function()) {
10533:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10534:   }
10535:   // aten::_foreach_tanh(Tensor[] self) -> Tensor[]
10536: 
10537:   auto dispatch__foreach_tanh = [](at::TensorList self) -> ::std::vector<at::Tensor> {
10538:     pybind11::gil_scoped_release no_gil;
10539:     return at::_foreach_tanh(self);
10540:   };
10541:   return wrap(dispatch__foreach_tanh(_r.tensorlist(0)));
10542:   Py_RETURN_NONE;
10543:   END_HANDLE_TH_ERRORS
10544: }
10545: 
10546: // _foreach_tanh_
10547: static PyObject * THPVariable__foreach_tanh_(PyObject* self_, PyObject* args, PyObject* kwargs)
10548: {
10549:   HANDLE_TH_ERRORS
10550:   static PythonArgParser parser({
10551:     "_foreach_tanh_(TensorList self)",
10552:   }, /*traceable=*/false);
10553: 
10554:   ParsedArgs<1> parsed_args;
10555:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10556:   if(_r.has_torch_function()) {
10557:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10558:   }
10559:   // aten::_foreach_tanh_(Tensor(a!)[] self) -> ()
10560: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_foreach_sqrt`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_foreach_sqrt`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10561-10640

```cpp
10561:   auto dispatch__foreach_tanh_ = [](at::TensorList self) -> void {
10562:     pybind11::gil_scoped_release no_gil;
10563:     at::_foreach_tanh_(self);
10564:   };
10565:   dispatch__foreach_tanh_(_r.tensorlist(0));
10566:   PyObject* self_tensorlist = _r.args[0];
10567:   Py_INCREF(self_tensorlist);
10568:   return self_tensorlist;
10569:   Py_RETURN_NONE;
10570:   END_HANDLE_TH_ERRORS
10571: }
10572: 
10573: // _foreach_clone
10574: static PyObject * THPVariable__foreach_clone(PyObject* self_, PyObject* args, PyObject* kwargs)
10575: {
10576:   HANDLE_TH_ERRORS
10577:   static PythonArgParser parser({
10578:     "_foreach_clone(TensorList self, *, MemoryFormat? memory_format=None)",
10579:   }, /*traceable=*/true);
10580: 
10581:   ParsedArgs<2> parsed_args;
10582:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10583:   if(_r.has_torch_function()) {
10584:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10585:   }
10586:   // aten::_foreach_clone(Tensor[] self, *, MemoryFormat? memory_format=None) -> Tensor[]
10587: 
10588:   auto dispatch__foreach_clone = [](at::TensorList self, ::std::optional<at::MemoryFormat> memory_format) -> ::std::vector<at::Tensor> {
10589:     pybind11::gil_scoped_release no_gil;
10590:     return at::_foreach_clone(self, memory_format);
10591:   };
10592:   return wrap(dispatch__foreach_clone(_r.tensorlist(0), _r.memoryformatOptional(1)));
10593:   Py_RETURN_NONE;
10594:   END_HANDLE_TH_ERRORS
10595: }
10596: 
10597: // _convert_indices_from_csr_to_coo
10598: static PyObject * THPVariable__convert_indices_from_csr_to_coo(PyObject* self_, PyObject* args, PyObject* kwargs)
10599: {
10600:   HANDLE_TH_ERRORS
10601:   static PythonArgParser parser({
10602:     "_convert_indices_from_csr_to_coo(Tensor crow_indices, Tensor col_indices, *, bool out_int32=False, bool transpose=False, Tensor out=None)",
10603:   }, /*traceable=*/true);
10604: 
10605:   ParsedArgs<5> parsed_args;
10606:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10607:   if(_r.has_torch_function()) {
10608:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10609:   }
10610:   if (_r.isNone(4)) {
10611:     // aten::_convert_indices_from_csr_to_coo(Tensor crow_indices, Tensor col_indices, *, bool out_int32=False, bool transpose=False) -> Tensor
10612: 
10613:     auto dispatch__convert_indices_from_csr_to_coo = [](const at::Tensor & crow_indices, const at::Tensor & col_indices, bool out_int32, bool transpose) -> at::Tensor {
10614:       pybind11::gil_scoped_release no_gil;
10615:       return at::_convert_indices_from_csr_to_coo(crow_indices, col_indices, out_int32, transpose);
10616:     };
10617:     return wrap(dispatch__convert_indices_from_csr_to_coo(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
10618:   } else {
10619:     // aten::_convert_indices_from_csr_to_coo.out(Tensor crow_indices, Tensor col_indices, *, bool out_int32=False, bool transpose=False, Tensor(a!) out) -> Tensor(a!)
10620: 
10621:     auto dispatch__convert_indices_from_csr_to_coo_out = [](at::Tensor out, const at::Tensor & crow_indices, const at::Tensor & col_indices, bool out_int32, bool transpose) -> at::Tensor {
10622:       pybind11::gil_scoped_release no_gil;
10623:       return at::_convert_indices_from_csr_to_coo_out(out, crow_indices, col_indices, out_int32, transpose);
10624:     };
10625:     return wrap(dispatch__convert_indices_from_csr_to_coo_out(_r.tensor(4), _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
10626:   }
10627:   Py_RETURN_NONE;
10628:   END_HANDLE_TH_ERRORS
10629: }
10630: 
10631: // mkldnn_adaptive_avg_pool2d
10632: static PyObject * THPVariable_mkldnn_adaptive_avg_pool2d(PyObject* self_, PyObject* args, PyObject* kwargs)
10633: {
10634:   HANDLE_TH_ERRORS
10635:   static PythonArgParser parser({
10636:     "mkldnn_adaptive_avg_pool2d(Tensor input, IntArrayRef[2] output_size, *, Tensor out=None)",
10637:   }, /*traceable=*/true);
10638: 
10639:   ParsedArgs<3> parsed_args;
10640:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `_foreach_tanh_`, `dispatch__foreach_tanh_`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `_foreach_tanh_`, `dispatch__foreach_tanh_`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10641-10720

```cpp
10641:   if(_r.has_torch_function()) {
10642:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10643:   }
10644:   if (_r.isNone(2)) {
10645:     // aten::mkldnn_adaptive_avg_pool2d(Tensor self, int[2] output_size) -> Tensor
10646: 
10647:     auto dispatch_mkldnn_adaptive_avg_pool2d = [](const at::Tensor & self, at::IntArrayRef output_size) -> at::Tensor {
10648:       pybind11::gil_scoped_release no_gil;
10649:       return at::mkldnn_adaptive_avg_pool2d(self, output_size);
10650:     };
10651:     return wrap(dispatch_mkldnn_adaptive_avg_pool2d(_r.tensor(0), _r.intlist(1)));
10652:   } else {
10653:     // aten::mkldnn_adaptive_avg_pool2d.out(Tensor self, int[2] output_size, *, Tensor(a!) out) -> Tensor(a!)
10654: 
10655:     auto dispatch_mkldnn_adaptive_avg_pool2d_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef output_size) -> at::Tensor {
10656:       pybind11::gil_scoped_release no_gil;
10657:       return at::mkldnn_adaptive_avg_pool2d_out(out, self, output_size);
10658:     };
10659:     return wrap(dispatch_mkldnn_adaptive_avg_pool2d_out(_r.tensor(2), _r.tensor(0), _r.intlist(1)));
10660:   }
10661:   Py_RETURN_NONE;
10662:   END_HANDLE_TH_ERRORS
10663: }
10664: 
10665: // column_stack
10666: static PyObject * THPVariable_column_stack(PyObject* self_, PyObject* args, PyObject* kwargs)
10667: {
10668:   HANDLE_TH_ERRORS
10669:   static PythonArgParser parser({
10670:     "column_stack(TensorList tensors, *, Tensor out=None)",
10671:   }, /*traceable=*/true);
10672: 
10673:   ParsedArgs<2> parsed_args;
10674:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10675:   if(_r.has_torch_function()) {
10676:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10677:   }
10678:   if (_r.isNone(1)) {
10679:     // aten::column_stack(Tensor[] tensors) -> Tensor
10680: 
10681:     auto dispatch_column_stack = [](at::TensorList tensors) -> at::Tensor {
10682:       pybind11::gil_scoped_release no_gil;
10683:       return at::column_stack(tensors);
10684:     };
10685:     return wrap(dispatch_column_stack(_r.tensorlist(0)));
10686:   } else {
10687:     // aten::column_stack.out(Tensor[] tensors, *, Tensor(a!) out) -> Tensor(a!)
10688: 
10689:     auto dispatch_column_stack_out = [](at::Tensor out, at::TensorList tensors) -> at::Tensor {
10690:       pybind11::gil_scoped_release no_gil;
10691:       return at::column_stack_out(out, tensors);
10692:     };
10693:     return wrap(dispatch_column_stack_out(_r.tensor(1), _r.tensorlist(0)));
10694:   }
10695:   Py_RETURN_NONE;
10696:   END_HANDLE_TH_ERRORS
10697: }
10698: 
10699: // isinf
10700: static PyObject * THPVariable_isinf(PyObject* self_, PyObject* args, PyObject* kwargs)
10701: {
10702:   HANDLE_TH_ERRORS
10703:   static PythonArgParser parser({
10704:     "isinf(Tensor input)",
10705:   }, /*traceable=*/true);
10706: 
10707:   ParsedArgs<1> parsed_args;
10708:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10709:   if(_r.has_torch_function()) {
10710:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10711:   }
10712:   // aten::isinf(Tensor self) -> Tensor
10713: 
10714:   auto dispatch_isinf = [](const at::Tensor & self) -> at::Tensor {
10715:     pybind11::gil_scoped_release no_gil;
10716:     return self.isinf();
10717:   };
10718:   return wrap(dispatch_isinf(_r.tensor(0)));
10719:   Py_RETURN_NONE;
10720:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `mkldnn_adaptive_avg_pool2d`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `mkldnn_adaptive_avg_pool2d`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10721-10800

```cpp
10721: }
10722: 
10723: // isneginf
10724: static PyObject * THPVariable_isneginf(PyObject* self_, PyObject* args, PyObject* kwargs)
10725: {
10726:   HANDLE_TH_ERRORS
10727:   static PythonArgParser parser({
10728:     "isneginf(Tensor input, *, Tensor out=None)",
10729:   }, /*traceable=*/true);
10730: 
10731:   ParsedArgs<2> parsed_args;
10732:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10733:   if(_r.has_torch_function()) {
10734:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10735:   }
10736:   if (_r.isNone(1)) {
10737:     // aten::isneginf(Tensor self) -> Tensor
10738: 
10739:     auto dispatch_isneginf = [](const at::Tensor & self) -> at::Tensor {
10740:       pybind11::gil_scoped_release no_gil;
10741:       return self.isneginf();
10742:     };
10743:     return wrap(dispatch_isneginf(_r.tensor(0)));
10744:   } else {
10745:     // aten::isneginf.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
10746: 
10747:     auto dispatch_isneginf_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
10748:       pybind11::gil_scoped_release no_gil;
10749:       return at::isneginf_out(out, self);
10750:     };
10751:     return wrap(dispatch_isneginf_out(_r.tensor(1), _r.tensor(0)));
10752:   }
10753:   Py_RETURN_NONE;
10754:   END_HANDLE_TH_ERRORS
10755: }
10756: 
10757: // _linalg_slogdet
10758: static PyObject * THPVariable__linalg_slogdet(PyObject* self_, PyObject* args, PyObject* kwargs)
10759: {
10760:   HANDLE_TH_ERRORS
10761:   static PyTypeObject* NamedTuple = generated::get__linalg_slogdet_structseq();
10762:   static PyTypeObject* NamedTuple1 = generated::get__linalg_slogdet_out_structseq();
10763:   static PythonArgParser parser({
10764:     "_linalg_slogdet(Tensor A, *, TensorList[4] out=None)",
10765:   }, /*traceable=*/true);
10766: 
10767:   ParsedArgs<2> parsed_args;
10768:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10769:   if(_r.has_torch_function()) {
10770:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10771:   }
10772:   if (_r.isNone(1)) {
10773:     // aten::_linalg_slogdet(Tensor A) -> (Tensor sign, Tensor logabsdet, Tensor LU, Tensor pivots)
10774: 
10775:     auto dispatch__linalg_slogdet = [](const at::Tensor & A) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
10776:       pybind11::gil_scoped_release no_gil;
10777:       return at::_linalg_slogdet(A);
10778:     };
10779:     return wrap(NamedTuple, dispatch__linalg_slogdet(_r.tensor(0)));
10780:   } else {
10781:     // aten::_linalg_slogdet.sign(Tensor A, *, Tensor(a!) sign, Tensor(b!) logabsdet, Tensor(c!) LU, Tensor(d!) pivots) -> (Tensor(a!) sign, Tensor(b!) logabsdet, Tensor(c!) LU, Tensor(d!) pivots)
10782:     auto out = _r.tensorlist_n<4>(1);
10783:     auto dispatch__linalg_slogdet_out = [](at::Tensor & sign, at::Tensor & logabsdet, at::Tensor & LU, at::Tensor & pivots, const at::Tensor & A) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
10784:       pybind11::gil_scoped_release no_gil;
10785:       return at::_linalg_slogdet_out(sign, logabsdet, LU, pivots, A);
10786:     };
10787:     return wrap(NamedTuple1, dispatch__linalg_slogdet_out(out[0], out[1], out[2], out[3], _r.tensor(0)));
10788:   }
10789:   Py_RETURN_NONE;
10790:   END_HANDLE_TH_ERRORS
10791: }
10792: 
10793: // slogdet
10794: static PyObject * THPVariable_slogdet(PyObject* self_, PyObject* args, PyObject* kwargs)
10795: {
10796:   HANDLE_TH_ERRORS
10797:   static PyTypeObject* NamedTuple = generated::get_slogdet_structseq();
10798:   static PyTypeObject* NamedTuple1 = generated::get_slogdet_out_structseq();
10799:   static PythonArgParser parser({
10800:     "slogdet(Tensor input, *, TensorList[2] out=None)",
```

- EN: The main execution path in this span is carried by `THPVariable_isneginf`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_isneginf`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10801-10880

```cpp
10801:   }, /*traceable=*/true);
10802: 
10803:   ParsedArgs<2> parsed_args;
10804:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10805:   if(_r.has_torch_function()) {
10806:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10807:   }
10808:   if (_r.isNone(1)) {
10809:     // aten::slogdet(Tensor self) -> (Tensor sign, Tensor logabsdet)
10810: 
10811:     auto dispatch_slogdet = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
10812:       pybind11::gil_scoped_release no_gil;
10813:       return self.slogdet();
10814:     };
10815:     return wrap(NamedTuple, dispatch_slogdet(_r.tensor(0)));
10816:   } else {
10817:     // aten::slogdet.out(Tensor self, *, Tensor(a!) sign, Tensor(b!) logabsdet) -> (Tensor(a!) sign, Tensor(b!) logabsdet)
10818:     auto out = _r.tensorlist_n<2>(1);
10819:     auto dispatch_slogdet_out = [](at::Tensor & sign, at::Tensor & logabsdet, const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
10820:       pybind11::gil_scoped_release no_gil;
10821:       return at::slogdet_out(sign, logabsdet, self);
10822:     };
10823:     return wrap(NamedTuple1, dispatch_slogdet_out(out[0], out[1], _r.tensor(0)));
10824:   }
10825:   Py_RETURN_NONE;
10826:   END_HANDLE_TH_ERRORS
10827: }
10828: 
10829: // inverse
10830: static PyObject * THPVariable_inverse(PyObject* self_, PyObject* args, PyObject* kwargs)
10831: {
10832:   HANDLE_TH_ERRORS
10833:   static PythonArgParser parser({
10834:     "inverse(Tensor input, *, Tensor out=None)",
10835:   }, /*traceable=*/true);
10836: 
10837:   ParsedArgs<2> parsed_args;
10838:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10839:   if(_r.has_torch_function()) {
10840:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10841:   }
10842:   if (_r.isNone(1)) {
10843:     // aten::inverse(Tensor self) -> Tensor
10844: 
10845:     auto dispatch_inverse = [](const at::Tensor & self) -> at::Tensor {
10846:       pybind11::gil_scoped_release no_gil;
10847:       return self.inverse();
10848:     };
10849:     return wrap(dispatch_inverse(_r.tensor(0)));
10850:   } else {
10851:     // aten::inverse.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
10852: 
10853:     auto dispatch_inverse_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
10854:       pybind11::gil_scoped_release no_gil;
10855:       return at::inverse_out(out, self);
10856:     };
10857:     return wrap(dispatch_inverse_out(_r.tensor(1), _r.tensor(0)));
10858:   }
10859:   Py_RETURN_NONE;
10860:   END_HANDLE_TH_ERRORS
10861: }
10862: 
10863: // inner
10864: static PyObject * THPVariable_inner(PyObject* self_, PyObject* args, PyObject* kwargs)
10865: {
10866:   HANDLE_TH_ERRORS
10867:   static PythonArgParser parser({
10868:     "inner(Tensor input, Tensor other, *, Tensor out=None)",
10869:   }, /*traceable=*/true);
10870: 
10871:   ParsedArgs<3> parsed_args;
10872:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10873:   if(_r.has_torch_function()) {
10874:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10875:   }
10876:   if (_r.isNone(2)) {
10877:     // aten::inner(Tensor self, Tensor other) -> Tensor
10878: 
10879:     auto dispatch_inner = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10880:       pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `slogdet`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `slogdet`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10881-10960

```cpp
10881:       return self.inner(other);
10882:     };
10883:     return wrap(dispatch_inner(_r.tensor(0), _r.tensor(1)));
10884:   } else {
10885:     // aten::inner.out(Tensor self, Tensor other, *, Tensor(a!) out) -> Tensor(a!)
10886: 
10887:     auto dispatch_inner_out = [](at::Tensor out, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10888:       pybind11::gil_scoped_release no_gil;
10889:       return at::inner_out(out, self, other);
10890:     };
10891:     return wrap(dispatch_inner_out(_r.tensor(2), _r.tensor(0), _r.tensor(1)));
10892:   }
10893:   Py_RETURN_NONE;
10894:   END_HANDLE_TH_ERRORS
10895: }
10896: 
10897: // _linalg_solve_ex
10898: static PyObject * THPVariable__linalg_solve_ex(PyObject* self_, PyObject* args, PyObject* kwargs)
10899: {
10900:   HANDLE_TH_ERRORS
10901:   static PyTypeObject* NamedTuple = generated::get__linalg_solve_ex_structseq();
10902:   static PyTypeObject* NamedTuple1 = generated::get__linalg_solve_ex_out_structseq();
10903:   static PythonArgParser parser({
10904:     "_linalg_solve_ex(Tensor A, Tensor B, *, bool left=True, bool check_errors=False, TensorList[4] out=None)",
10905:   }, /*traceable=*/true);
10906: 
10907:   ParsedArgs<5> parsed_args;
10908:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10909:   if(_r.has_torch_function()) {
10910:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10911:   }
10912:   if (_r.isNone(4)) {
10913:     // aten::_linalg_solve_ex(Tensor A, Tensor B, *, bool left=True, bool check_errors=False) -> (Tensor result, Tensor LU, Tensor pivots, Tensor info)
10914: 
10915:     auto dispatch__linalg_solve_ex = [](const at::Tensor & A, const at::Tensor & B, bool left, bool check_errors) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
10916:       pybind11::gil_scoped_release no_gil;
10917:       return at::_linalg_solve_ex(A, B, left, check_errors);
10918:     };
10919:     return wrap(NamedTuple, dispatch__linalg_solve_ex(_r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
10920:   } else {
10921:     // aten::_linalg_solve_ex.result(Tensor A, Tensor B, *, bool left=True, bool check_errors=False, Tensor(a!) result, Tensor(b!) LU, Tensor(c!) pivots, Tensor(d!) info) -> (Tensor(a!) result, Tensor(b!) LU, Tensor(c!) pivots, Tensor(d!) info)
10922:     auto out = _r.tensorlist_n<4>(4);
10923:     auto dispatch__linalg_solve_ex_out = [](at::Tensor & result, at::Tensor & LU, at::Tensor & pivots, at::Tensor & info, const at::Tensor & A, const at::Tensor & B, bool left, bool check_errors) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
10924:       pybind11::gil_scoped_release no_gil;
10925:       return at::_linalg_solve_ex_out(result, LU, pivots, info, A, B, left, check_errors);
10926:     };
10927:     return wrap(NamedTuple1, dispatch__linalg_solve_ex_out(out[0], out[1], out[2], out[3], _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
10928:   }
10929:   Py_RETURN_NONE;
10930:   END_HANDLE_TH_ERRORS
10931: }
10932: 
10933: // _test_parallel_materialize
10934: static PyObject * THPVariable__test_parallel_materialize(PyObject* self_, PyObject* args, PyObject* kwargs)
10935: {
10936:   HANDLE_TH_ERRORS
10937:   static PythonArgParser parser({
10938:     "_test_parallel_materialize(Tensor input, int64_t num_parallel, bool skip_first=False)",
10939:   }, /*traceable=*/true);
10940: 
10941:   ParsedArgs<3> parsed_args;
10942:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10943:   if(_r.has_torch_function()) {
10944:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10945:   }
10946:   // aten::_test_parallel_materialize(Tensor self, int num_parallel, bool skip_first=False) -> Tensor
10947: 
10948:   auto dispatch__test_parallel_materialize = [](const at::Tensor & self, int64_t num_parallel, bool skip_first) -> at::Tensor {
10949:     pybind11::gil_scoped_release no_gil;
10950:     return at::_test_parallel_materialize(self, num_parallel, skip_first);
10951:   };
10952:   return wrap(dispatch__test_parallel_materialize(_r.tensor(0), _r.toInt64(1), _r.toBool(2)));
10953:   Py_RETURN_NONE;
10954:   END_HANDLE_TH_ERRORS
10955: }
10956: 
10957: \
10958: // _test_autograd_multiple_dispatch
10959: static PyObject * THPVariable__test_autograd_multiple_dispatch(PyObject* self_, PyObject* args, PyObject* kwargs)
10960: {
```

- EN: The main execution path in this span is carried by `wrap`, `inner_out`, `THPVariable__linalg_solve_ex`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `inner_out`, `THPVariable__linalg_solve_ex` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10961-11040

```cpp
10961:   HANDLE_TH_ERRORS
10962:   static PythonArgParser parser({
10963:     "_test_autograd_multiple_dispatch(Tensor input)",
10964:     "_test_autograd_multiple_dispatch(Tensor input, bool b)",
10965:   }, /*traceable=*/true);
10966: 
10967:   ParsedArgs<2> parsed_args;
10968:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
10969:   if(_r.has_torch_function()) {
10970:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
10971:   }
10972:   switch (_r.idx) {
10973:     case 0: {
10974:       // aten::_test_autograd_multiple_dispatch.fullcoverage(Tensor self) -> Tensor
10975: 
10976:       auto dispatch__test_autograd_multiple_dispatch = [](const at::Tensor & self) -> at::Tensor {
10977:         pybind11::gil_scoped_release no_gil;
10978:         return at::_test_autograd_multiple_dispatch(self);
10979:       };
10980:       return wrap(dispatch__test_autograd_multiple_dispatch(_r.tensor(0)));
10981:     }
10982:     case 1: {
10983:       // aten::_test_autograd_multiple_dispatch.ntonly(Tensor self, bool b) -> Tensor
10984: 
10985:       auto dispatch__test_autograd_multiple_dispatch = [](const at::Tensor & self, bool b) -> at::Tensor {
10986:         pybind11::gil_scoped_release no_gil;
10987:         return at::_test_autograd_multiple_dispatch(self, b);
10988:       };
10989:       return wrap(dispatch__test_autograd_multiple_dispatch(_r.tensor(0), _r.toBool(1)));
10990:     }
10991:   }
10992:   Py_RETURN_NONE;
10993:   END_HANDLE_TH_ERRORS
10994: }
10995: 
10996: // _test_autograd_multiple_dispatch_view_copy
10997: static PyObject * THPVariable__test_autograd_multiple_dispatch_view_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
10998: {
10999:   HANDLE_TH_ERRORS
11000:   static PythonArgParser parser({
11001:     "_test_autograd_multiple_dispatch_view_copy(Tensor input, *, Tensor out=None)",
11002:   }, /*traceable=*/true);
11003: 
11004:   ParsedArgs<2> parsed_args;
11005:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11006:   if(_r.has_torch_function()) {
11007:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11008:   }
11009:   if (_r.isNone(1)) {
11010:     // aten::_test_autograd_multiple_dispatch_view_copy(Tensor self) -> Tensor
11011: 
11012:     auto dispatch__test_autograd_multiple_dispatch_view_copy = [](const at::Tensor & self) -> at::Tensor {
11013:       pybind11::gil_scoped_release no_gil;
11014:       return at::_test_autograd_multiple_dispatch_view_copy(self);
11015:     };
11016:     return wrap(dispatch__test_autograd_multiple_dispatch_view_copy(_r.tensor(0)));
11017:   } else {
11018:     // aten::_test_autograd_multiple_dispatch_view_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11019: 
11020:     auto dispatch__test_autograd_multiple_dispatch_view_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11021:       pybind11::gil_scoped_release no_gil;
11022:       return at::_test_autograd_multiple_dispatch_view_copy_out(out, self);
11023:     };
11024:     return wrap(dispatch__test_autograd_multiple_dispatch_view_copy_out(_r.tensor(1), _r.tensor(0)));
11025:   }
11026:   Py_RETURN_NONE;
11027:   END_HANDLE_TH_ERRORS
11028: }
11029: 
11030: // segment_reduce
11031: static PyObject * THPVariable_segment_reduce(PyObject* self_, PyObject* args, PyObject* kwargs)
11032: {
11033:   HANDLE_TH_ERRORS
11034:   static PythonArgParser parser({
11035:     "segment_reduce(Tensor data, c10::string_view reduce, *, Tensor? lengths=None, Tensor? indices=None, Tensor? offsets=None, int64_t axis=0, bool unsafe=False, Scalar? initial=None)",
11036:   }, /*traceable=*/true);
11037: 
11038:   ParsedArgs<8> parsed_args;
11039:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11040:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `_test_autograd_multiple_dispatch`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `_test_autograd_multiple_dispatch` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11041-11120

```cpp
11041:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11042:   }
11043:   // aten::segment_reduce(Tensor data, str reduce, *, Tensor? lengths=None, Tensor? indices=None, Tensor? offsets=None, int axis=0, bool unsafe=False, Scalar? initial=None) -> Tensor
11044: 
11045:   auto dispatch_segment_reduce = [](const at::Tensor & data, c10::string_view reduce, const ::std::optional<at::Tensor> & lengths, const ::std::optional<at::Tensor> & indices, const ::std::optional<at::Tensor> & offsets, int64_t axis, bool unsafe, const ::std::optional<at::Scalar> & initial) -> at::Tensor {
11046:     pybind11::gil_scoped_release no_gil;
11047:     return at::segment_reduce(data, reduce, lengths, indices, offsets, axis, unsafe, initial);
11048:   };
11049:   return wrap(dispatch_segment_reduce(_r.tensor(0), _r.stringView(1), _r.optionalTensor(2), _r.optionalTensor(3), _r.optionalTensor(4), _r.toInt64(5), _r.toBool(6), _r.scalarOptional(7)));
11050:   Py_RETURN_NONE;
11051:   END_HANDLE_TH_ERRORS
11052: }
11053: 
11054: // _conj_copy
11055: static PyObject * THPVariable__conj_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11056: {
11057:   HANDLE_TH_ERRORS
11058:   static PythonArgParser parser({
11059:     "_conj_copy(Tensor input, *, Tensor out=None)",
11060:   }, /*traceable=*/true);
11061: 
11062:   ParsedArgs<2> parsed_args;
11063:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11064:   if(_r.has_torch_function()) {
11065:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11066:   }
11067:   if (_r.isNone(1)) {
11068:     // aten::_conj_copy(Tensor self) -> Tensor
11069: 
11070:     auto dispatch__conj_copy = [](const at::Tensor & self) -> at::Tensor {
11071:       pybind11::gil_scoped_release no_gil;
11072:       return at::_conj_copy(self);
11073:     };
11074:     return wrap(dispatch__conj_copy(_r.tensor(0)));
11075:   } else {
11076:     // aten::_conj_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11077: 
11078:     auto dispatch__conj_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11079:       pybind11::gil_scoped_release no_gil;
11080:       return at::_conj_copy_out(out, self);
11081:     };
11082:     return wrap(dispatch__conj_copy_out(_r.tensor(1), _r.tensor(0)));
11083:   }
11084:   Py_RETURN_NONE;
11085:   END_HANDLE_TH_ERRORS
11086: }
11087: 
11088: // _sparse_broadcast_to_copy
11089: static PyObject * THPVariable__sparse_broadcast_to_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11090: {
11091:   HANDLE_TH_ERRORS
11092:   static PythonArgParser parser({
11093:     "_sparse_broadcast_to_copy(Tensor input, IntArrayRef size, *, Tensor out=None)",
11094:   }, /*traceable=*/true);
11095: 
11096:   ParsedArgs<3> parsed_args;
11097:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11098:   if(_r.has_torch_function()) {
11099:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11100:   }
11101:   if (_r.isNone(2)) {
11102:     // aten::_sparse_broadcast_to_copy(Tensor self, int[] size) -> Tensor
11103: 
11104:     auto dispatch__sparse_broadcast_to_copy = [](const at::Tensor & self, at::IntArrayRef size) -> at::Tensor {
11105:       pybind11::gil_scoped_release no_gil;
11106:       return at::_sparse_broadcast_to_copy(self, size);
11107:     };
11108:     return wrap(dispatch__sparse_broadcast_to_copy(_r.tensor(0), _r.intlist(1)));
11109:   } else {
11110:     // aten::_sparse_broadcast_to_copy.out(Tensor self, int[] size, *, Tensor(a!) out) -> Tensor(a!)
11111: 
11112:     auto dispatch__sparse_broadcast_to_copy_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef size) -> at::Tensor {
11113:       pybind11::gil_scoped_release no_gil;
11114:       return at::_sparse_broadcast_to_copy_out(out, self, size);
11115:     };
11116:     return wrap(dispatch__sparse_broadcast_to_copy_out(_r.tensor(2), _r.tensor(0), _r.intlist(1)));
11117:   }
11118:   Py_RETURN_NONE;
11119:   END_HANDLE_TH_ERRORS
11120: }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `segment_reduce`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `segment_reduce`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11121-11200

```cpp
11121: 
11122: // select_copy
11123: static PyObject * THPVariable_select_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11124: {
11125:   HANDLE_TH_ERRORS
11126:   static PythonArgParser parser({
11127:     "select_copy(Tensor input, int64_t dim, SymInt index, *, Tensor out=None)",
11128:   }, /*traceable=*/true);
11129: 
11130:   ParsedArgs<4> parsed_args;
11131:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11132:   if(_r.has_torch_function()) {
11133:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11134:   }
11135:   if (_r.isNone(3)) {
11136:     // aten::select_copy.int(Tensor self, int dim, SymInt index) -> Tensor
11137: 
11138:     auto dispatch_select_copy = [](const at::Tensor & self, int64_t dim, c10::SymInt index) -> at::Tensor {
11139:       pybind11::gil_scoped_release no_gil;
11140:       return at::select_copy_symint(self, dim, index);
11141:     };
11142:     return wrap(dispatch_select_copy(_r.tensor(0), _r.toInt64(1), _r.toSymInt(2)));
11143:   } else {
11144:     // aten::select_copy.int_out(Tensor self, int dim, SymInt index, *, Tensor(a!) out) -> Tensor(a!)
11145: 
11146:     auto dispatch_select_copy_out = [](at::Tensor out, const at::Tensor & self, int64_t dim, c10::SymInt index) -> at::Tensor {
11147:       pybind11::gil_scoped_release no_gil;
11148:       return at::select_copy_symint_out(out, self, dim, index);
11149:     };
11150:     return wrap(dispatch_select_copy_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.toSymInt(2)));
11151:   }
11152:   Py_RETURN_NONE;
11153:   END_HANDLE_TH_ERRORS
11154: }
11155: 
11156: \
11157: // squeeze_copy
11158: static PyObject * THPVariable_squeeze_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11159: {
11160:   HANDLE_TH_ERRORS
11161:   static PythonArgParser parser({
11162:     "squeeze_copy(Tensor input, *, Tensor out=None)",
11163:     "squeeze_copy(Tensor input, int64_t dim, *, Tensor out=None)",
11164:     "squeeze_copy(Tensor input, IntArrayRef dim, *, Tensor out=None)",
11165:   }, /*traceable=*/true);
11166: 
11167:   ParsedArgs<3> parsed_args;
11168:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11169:   if(_r.has_torch_function()) {
11170:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11171:   }
11172:   switch (_r.idx) {
11173:     case 0: {
11174:       if (_r.isNone(1)) {
11175:         // aten::squeeze_copy(Tensor self) -> Tensor
11176: 
11177:         auto dispatch_squeeze_copy = [](const at::Tensor & self) -> at::Tensor {
11178:           pybind11::gil_scoped_release no_gil;
11179:           return at::squeeze_copy(self);
11180:         };
11181:         return wrap(dispatch_squeeze_copy(_r.tensor(0)));
11182:       } else {
11183:         // aten::squeeze_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11184: 
11185:         auto dispatch_squeeze_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11186:           pybind11::gil_scoped_release no_gil;
11187:           return at::squeeze_copy_out(out, self);
11188:         };
11189:         return wrap(dispatch_squeeze_copy_out(_r.tensor(1), _r.tensor(0)));
11190:       }
11191:     }
11192:     case 1: {
11193:       if (_r.isNone(2)) {
11194:         // aten::squeeze_copy.dim(Tensor self, int dim) -> Tensor
11195: 
11196:         auto dispatch_squeeze_copy = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
11197:           pybind11::gil_scoped_release no_gil;
11198:           return at::squeeze_copy(self, dim);
11199:         };
11200:         return wrap(dispatch_squeeze_copy(_r.tensor(0), _r.toInt64(1)));
```

- EN: The main execution path in this span is carried by `THPVariable_select_copy`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_select_copy`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11201-11280

```cpp
11201:       } else {
11202:         // aten::squeeze_copy.dim_out(Tensor self, int dim, *, Tensor(a!) out) -> Tensor(a!)
11203: 
11204:         auto dispatch_squeeze_copy_out = [](at::Tensor out, const at::Tensor & self, int64_t dim) -> at::Tensor {
11205:           pybind11::gil_scoped_release no_gil;
11206:           return at::squeeze_copy_out(out, self, dim);
11207:         };
11208:         return wrap(dispatch_squeeze_copy_out(_r.tensor(2), _r.tensor(0), _r.toInt64(1)));
11209:       }
11210:     }
11211:     case 2: {
11212:       if (_r.isNone(2)) {
11213:         // aten::squeeze_copy.dims(Tensor self, int[] dim) -> Tensor
11214: 
11215:         auto dispatch_squeeze_copy = [](const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
11216:           pybind11::gil_scoped_release no_gil;
11217:           return at::squeeze_copy(self, dim);
11218:         };
11219:         return wrap(dispatch_squeeze_copy(_r.tensor(0), _r.intlist(1)));
11220:       } else {
11221:         // aten::squeeze_copy.dims_out(Tensor self, int[] dim, *, Tensor(a!) out) -> Tensor(a!)
11222: 
11223:         auto dispatch_squeeze_copy_out = [](at::Tensor out, const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
11224:           pybind11::gil_scoped_release no_gil;
11225:           return at::squeeze_copy_out(out, self, dim);
11226:         };
11227:         return wrap(dispatch_squeeze_copy_out(_r.tensor(2), _r.tensor(0), _r.intlist(1)));
11228:       }
11229:     }
11230:   }
11231:   Py_RETURN_NONE;
11232:   END_HANDLE_TH_ERRORS
11233: }
11234: 
11235: // transpose_copy
11236: static PyObject * THPVariable_transpose_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11237: {
11238:   HANDLE_TH_ERRORS
11239:   static PythonArgParser parser({
11240:     "transpose_copy(Tensor input, int64_t dim0, int64_t dim1, *, Tensor out=None)",
11241:   }, /*traceable=*/true);
11242: 
11243:   ParsedArgs<4> parsed_args;
11244:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11245:   if(_r.has_torch_function()) {
11246:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11247:   }
11248:   if (_r.isNone(3)) {
11249:     // aten::transpose_copy.int(Tensor self, int dim0, int dim1) -> Tensor
11250: 
11251:     auto dispatch_transpose_copy = [](const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
11252:       pybind11::gil_scoped_release no_gil;
11253:       return at::transpose_copy(self, dim0, dim1);
11254:     };
11255:     return wrap(dispatch_transpose_copy(_r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
11256:   } else {
11257:     // aten::transpose_copy.int_out(Tensor self, int dim0, int dim1, *, Tensor(a!) out) -> Tensor(a!)
11258: 
11259:     auto dispatch_transpose_copy_out = [](at::Tensor out, const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
11260:       pybind11::gil_scoped_release no_gil;
11261:       return at::transpose_copy_out(out, self, dim0, dim1);
11262:     };
11263:     return wrap(dispatch_transpose_copy_out(_r.tensor(3), _r.tensor(0), _r.toInt64(1), _r.toInt64(2)));
11264:   }
11265:   Py_RETURN_NONE;
11266:   END_HANDLE_TH_ERRORS
11267: }
11268: 
11269: // values_copy
11270: static PyObject * THPVariable_values_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11271: {
11272:   HANDLE_TH_ERRORS
11273:   static PythonArgParser parser({
11274:     "values_copy(Tensor input, *, Tensor out=None)",
11275:   }, /*traceable=*/true);
11276: 
11277:   ParsedArgs<2> parsed_args;
11278:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11279:   if(_r.has_torch_function()) {
11280:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
```

- EN: The main execution path in this span is carried by `squeeze_copy_out`, `wrap`, `squeeze_copy`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `squeeze_copy_out`, `wrap`, `squeeze_copy` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11281-11360

```cpp
11281:   }
11282:   if (_r.isNone(1)) {
11283:     // aten::values_copy(Tensor self) -> Tensor
11284: 
11285:     auto dispatch_values_copy = [](const at::Tensor & self) -> at::Tensor {
11286:       pybind11::gil_scoped_release no_gil;
11287:       return at::values_copy(self);
11288:     };
11289:     return wrap(dispatch_values_copy(_r.tensor(0)));
11290:   } else {
11291:     // aten::values_copy.out(Tensor self, *, Tensor(a!) out) -> Tensor(a!)
11292: 
11293:     auto dispatch_values_copy_out = [](at::Tensor out, const at::Tensor & self) -> at::Tensor {
11294:       pybind11::gil_scoped_release no_gil;
11295:       return at::values_copy_out(out, self);
11296:     };
11297:     return wrap(dispatch_values_copy_out(_r.tensor(1), _r.tensor(0)));
11298:   }
11299:   Py_RETURN_NONE;
11300:   END_HANDLE_TH_ERRORS
11301: }
11302: 
11303: // unfold_copy
11304: static PyObject * THPVariable_unfold_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11305: {
11306:   HANDLE_TH_ERRORS
11307:   static PythonArgParser parser({
11308:     "unfold_copy(Tensor input, int64_t dimension, int64_t size, int64_t step, *, Tensor out=None)",
11309:   }, /*traceable=*/true);
11310: 
11311:   ParsedArgs<5> parsed_args;
11312:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11313:   if(_r.has_torch_function()) {
11314:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11315:   }
11316:   if (_r.isNone(4)) {
11317:     // aten::unfold_copy(Tensor self, int dimension, int size, int step) -> Tensor
11318: 
11319:     auto dispatch_unfold_copy = [](const at::Tensor & self, int64_t dimension, int64_t size, int64_t step) -> at::Tensor {
11320:       pybind11::gil_scoped_release no_gil;
11321:       return at::unfold_copy(self, dimension, size, step);
11322:     };
11323:     return wrap(dispatch_unfold_copy(_r.tensor(0), _r.toInt64(1), _r.toInt64(2), _r.toInt64(3)));
11324:   } else {
11325:     // aten::unfold_copy.out(Tensor self, int dimension, int size, int step, *, Tensor(a!) out) -> Tensor(a!)
11326: 
11327:     auto dispatch_unfold_copy_out = [](at::Tensor out, const at::Tensor & self, int64_t dimension, int64_t size, int64_t step) -> at::Tensor {
11328:       pybind11::gil_scoped_release no_gil;
11329:       return at::unfold_copy_out(out, self, dimension, size, step);
11330:     };
11331:     return wrap(dispatch_unfold_copy_out(_r.tensor(4), _r.tensor(0), _r.toInt64(1), _r.toInt64(2), _r.toInt64(3)));
11332:   }
11333:   Py_RETURN_NONE;
11334:   END_HANDLE_TH_ERRORS
11335: }
11336: 
11337: // _fused_sdp_choice
11338: static PyObject * THPVariable__fused_sdp_choice(PyObject* self_, PyObject* args, PyObject* kwargs)
11339: {
11340:   HANDLE_TH_ERRORS
11341:   static PythonArgParser parser({
11342:     "_fused_sdp_choice(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, double dropout_p=0.0, bool is_causal=False, *, double? scale=None, bool enable_gqa=False)",
11343:   }, /*traceable=*/false);
11344: 
11345:   ParsedArgs<8> parsed_args;
11346:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11347:   if(_r.has_torch_function()) {
11348:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11349:   }
11350:   // aten::_fused_sdp_choice(Tensor query, Tensor key, Tensor value, Tensor? attn_mask=None, float dropout_p=0.0, bool is_causal=False, *, float? scale=None, bool enable_gqa=False) -> int
11351: 
11352:   auto dispatch__fused_sdp_choice = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & attn_mask, double dropout_p, bool is_causal, ::std::optional<double> scale, bool enable_gqa) -> int64_t {
11353:     pybind11::gil_scoped_release no_gil;
11354:     return at::_fused_sdp_choice(query, key, value, attn_mask, dropout_p, is_causal, scale, enable_gqa);
11355:   };
11356:   return wrap(dispatch__fused_sdp_choice(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.toDouble(4), _r.toBool(5), _r.toDoubleOptional(6), _r.toBool(7)));
11357:   Py_RETURN_NONE;
11358:   END_HANDLE_TH_ERRORS
11359: }
11360: 
```

- EN: The main execution path in this span is carried by `values_copy`, `wrap`, `values_copy_out`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `values_copy`, `wrap`, `values_copy_out` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11361-11440

```cpp
11361: // _scaled_dot_product_efficient_attention
11362: static PyObject * THPVariable__scaled_dot_product_efficient_attention(PyObject* self_, PyObject* args, PyObject* kwargs)
11363: {
11364:   HANDLE_TH_ERRORS
11365:   static PyTypeObject* NamedTuple = generated::get__scaled_dot_product_efficient_attention_structseq();
11366:   static PythonArgParser parser({
11367:     "_scaled_dot_product_efficient_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_bias, bool compute_log_sumexp, double dropout_p=0.0, bool is_causal=False, *, double? scale=None)",
11368:   }, /*traceable=*/true);
11369: 
11370:   ParsedArgs<8> parsed_args;
11371:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11372:   if(_r.has_torch_function()) {
11373:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11374:   }
11375:   // aten::_scaled_dot_product_efficient_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_bias, bool compute_log_sumexp, float dropout_p=0.0, bool is_causal=False, *, float? scale=None) -> (Tensor output, Tensor log_sumexp, Tensor philox_seed, Tensor philox_offset)
11376: 
11377:   auto dispatch__scaled_dot_product_efficient_attention = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & attn_bias, bool compute_log_sumexp, double dropout_p, bool is_causal, ::std::optional<double> scale) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor> {
11378:     pybind11::gil_scoped_release no_gil;
11379:     return at::_scaled_dot_product_efficient_attention(query, key, value, attn_bias, compute_log_sumexp, dropout_p, is_causal, scale);
11380:   };
11381:   return wrap(NamedTuple, dispatch__scaled_dot_product_efficient_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.toBool(4), _r.toDouble(5), _r.toBool(6), _r.toDoubleOptional(7)));
11382:   Py_RETURN_NONE;
11383:   END_HANDLE_TH_ERRORS
11384: }
11385: 
11386: // _scaled_dot_product_cudnn_attention
11387: static PyObject * THPVariable__scaled_dot_product_cudnn_attention(PyObject* self_, PyObject* args, PyObject* kwargs)
11388: {
11389:   HANDLE_TH_ERRORS
11390:   static PyTypeObject* NamedTuple = generated::get__scaled_dot_product_cudnn_attention_structseq();
11391:   static PythonArgParser parser({
11392:     "_scaled_dot_product_cudnn_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_bias, bool compute_log_sumexp, double dropout_p=0.0, bool is_causal=False, bool return_debug_mask=False, *, double? scale=None)",
11393:   }, /*traceable=*/true);
11394: 
11395:   ParsedArgs<9> parsed_args;
11396:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11397:   if(_r.has_torch_function()) {
11398:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11399:   }
11400:   // aten::_scaled_dot_product_cudnn_attention(Tensor query, Tensor key, Tensor value, Tensor? attn_bias, bool compute_log_sumexp, float dropout_p=0.0, bool is_causal=False, bool return_debug_mask=False, *, float? scale=None) -> (Tensor output, Tensor logsumexp, Tensor cum_seq_q, Tensor cum_seq_k, SymInt max_q, SymInt max_k, Tensor philox_seed, Tensor philox_offset, Tensor debug_attn_mask)
11401: 
11402:   auto dispatch__scaled_dot_product_cudnn_attention = [](const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & attn_bias, bool compute_log_sumexp, double dropout_p, bool is_causal, bool return_debug_mask, ::std::optional<double> scale) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor,at::Tensor,c10::SymInt,c10::SymInt,at::Tensor,at::Tensor,at::Tensor> {
11403:     pybind11::gil_scoped_release no_gil;
11404:     return at::_scaled_dot_product_cudnn_attention(query, key, value, attn_bias, compute_log_sumexp, dropout_p, is_causal, return_debug_mask, scale);
11405:   };
11406:   return wrap(NamedTuple, dispatch__scaled_dot_product_cudnn_attention(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.optionalTensor(3), _r.toBool(4), _r.toDouble(5), _r.toBool(6), _r.toBool(7), _r.toDoubleOptional(8)));
11407:   Py_RETURN_NONE;
11408:   END_HANDLE_TH_ERRORS
11409: }
11410: 
11411: // _flash_attention_forward_no_dropout_inplace
11412: static PyObject * THPVariable__flash_attention_forward_no_dropout_inplace(PyObject* self_, PyObject* args, PyObject* kwargs)
11413: {
11414:   HANDLE_TH_ERRORS
11415:   static PythonArgParser parser({
11416:     "_flash_attention_forward_no_dropout_inplace(Tensor out, Tensor query, Tensor key, Tensor value, Tensor? cum_seq_q, Tensor? cum_seq_k, SymInt max_q, SymInt max_k, double dropout_p, bool is_causal, bool return_debug_mask, *, double? scale=None, SymInt? window_size_left=None, SymInt? window_size_right=None, Tensor? seqused_k=None, Tensor? alibi_slopes=None, Tensor? block_table=None, int64_t? num_splits=None)",
11417:   }, /*traceable=*/true);
11418: 
11419:   ParsedArgs<18> parsed_args;
11420:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11421:   if(_r.has_torch_function()) {
11422:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11423:   }
11424:   // aten::_flash_attention_forward_no_dropout_inplace(Tensor(a!) out, Tensor query, Tensor key, Tensor value, Tensor? cum_seq_q, Tensor? cum_seq_k, SymInt max_q, SymInt max_k, float dropout_p, bool is_causal, bool return_debug_mask, *, float? scale=None, SymInt? window_size_left=None, SymInt? window_size_right=None, Tensor? seqused_k=None, Tensor? alibi_slopes=None, Tensor? block_table=None, int? num_splits=None) -> Tensor softmax_logsumexp
11425: 
11426:   auto dispatch__flash_attention_forward_no_dropout_inplace = [](at::Tensor out, const at::Tensor & query, const at::Tensor & key, const at::Tensor & value, const ::std::optional<at::Tensor> & cum_seq_q, const ::std::optional<at::Tensor> & cum_seq_k, c10::SymInt max_q, c10::SymInt max_k, double dropout_p, bool is_causal, bool return_debug_mask, ::std::optional<double> scale, ::std::optional<c10::SymInt> window_size_left, ::std::optional<c10::SymInt> window_size_right, const ::std::optional<at::Tensor> & seqused_k, const ::std::optional<at::Tensor> & alibi_slopes, const ::std::optional<at::Tensor> & block_table, ::std::optional<int64_t> num_splits) -> at::Tensor {
11427:     pybind11::gil_scoped_release no_gil;
11428:     return at::_flash_attention_forward_no_dropout_inplace_symint(out, query, key, value, cum_seq_q, cum_seq_k, max_q, max_k, dropout_p, is_causal, return_debug_mask, scale, window_size_left, window_size_right, seqused_k, alibi_slopes, block_table, num_splits);
11429:   };
11430:   return wrap(dispatch__flash_attention_forward_no_dropout_inplace(_r.tensor(0), _r.tensor(1), _r.tensor(2), _r.tensor(3), _r.optionalTensor(4), _r.optionalTensor(5), _r.toSymInt(6), _r.toSymInt(7), _r.toDouble(8), _r.toBool(9), _r.toBool(10), _r.toDoubleOptional(11), _r.toSymIntOptional(12), _r.toSymIntOptional(13), _r.optionalTensor(14), _r.optionalTensor(15), _r.optionalTensor(16), _r.toInt64Optional(17)));
11431:   Py_RETURN_NONE;
11432:   END_HANDLE_TH_ERRORS
11433: }
11434: 
11435: // _fill_mem_eff_dropout_mask_
11436: static PyObject * THPVariable__fill_mem_eff_dropout_mask_(PyObject* self_, PyObject* args, PyObject* kwargs)
11437: {
11438:   HANDLE_TH_ERRORS
11439:   static PythonArgParser parser({
11440:     "_fill_mem_eff_dropout_mask_(Tensor input, double dropout_p, int64_t seed, int64_t offset)",
```

- EN: The main execution path in this span is carried by `THPVariable__scaled_dot_product_efficient_attention`, `get__scaled_dot_product_efficient_attention_structseq`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__scaled_dot_product_efficient_attention`, `get__scaled_dot_product_efficient_attention_structseq`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11441-11520

```cpp
11441:   }, /*traceable=*/true);
11442: 
11443:   ParsedArgs<4> parsed_args;
11444:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11445:   if(_r.has_torch_function()) {
11446:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11447:   }
11448:   // aten::_fill_mem_eff_dropout_mask_(Tensor(a!) self, float dropout_p, int seed, int offset) -> Tensor(a!)
11449: 
11450:   auto dispatch__fill_mem_eff_dropout_mask_ = [](at::Tensor self, double dropout_p, int64_t seed, int64_t offset) -> at::Tensor {
11451:     pybind11::gil_scoped_release no_gil;
11452:     return at::_fill_mem_eff_dropout_mask_(self, dropout_p, seed, offset);
11453:   };
11454:   return wrap(dispatch__fill_mem_eff_dropout_mask_(_r.tensor(0), _r.toDouble(1), _r.toInt64(2), _r.toInt64(3)));
11455:   Py_RETURN_NONE;
11456:   END_HANDLE_TH_ERRORS
11457: }
11458: 
11459: // _foobar
11460: static PyObject * THPVariable__foobar(PyObject* self_, PyObject* args, PyObject* kwargs)
11461: {
11462:   HANDLE_TH_ERRORS
11463:   static PythonArgParser parser({
11464:     "_foobar(Tensor input, bool arg1=True, bool arg2=True, *, bool arg3=True)",
11465:   }, /*traceable=*/true);
11466: 
11467:   ParsedArgs<4> parsed_args;
11468:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11469:   if(_r.has_torch_function()) {
11470:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11471:   }
11472:   // aten::_foobar(Tensor self, bool arg1=True, bool arg2=True, *, bool arg3=True) -> Tensor
11473: 
11474:   auto dispatch__foobar = [](const at::Tensor & self, bool arg1, bool arg2, bool arg3) -> at::Tensor {
11475:     pybind11::gil_scoped_release no_gil;
11476:     return at::_foobar(self, arg1, arg2, arg3);
11477:   };
11478:   return wrap(dispatch__foobar(_r.tensor(0), _r.toBool(1), _r.toBool(2), _r.toBool(3)));
11479:   Py_RETURN_NONE;
11480:   END_HANDLE_TH_ERRORS
11481: }
11482: 
11483: \
11484: // _fused_adagrad_
11485: static PyObject * THPVariable__fused_adagrad_(PyObject* self_, PyObject* args, PyObject* kwargs)
11486: {
11487:   HANDLE_TH_ERRORS
11488:   static PythonArgParser parser({
11489:     "_fused_adagrad_(TensorList self, TensorList grads, TensorList state_sums, TensorList state_steps, *, Tensor lr, double lr_decay, double weight_decay, double eps, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None)",
11490:     "_fused_adagrad_(TensorList self, TensorList grads, TensorList state_sums, TensorList state_steps, *, double lr, double lr_decay, double weight_decay, double eps, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None)",
11491:   }, /*traceable=*/false);
11492: 
11493:   ParsedArgs<11> parsed_args;
11494:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11495:   if(_r.has_torch_function()) {
11496:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11497:   }
11498:   switch (_r.idx) {
11499:     case 0: {
11500:       // aten::_fused_adagrad_.tensor_lr(Tensor(a!)[] self, Tensor(b!)[] grads, Tensor(c!)[] state_sums, Tensor[] state_steps, *, Tensor lr, float lr_decay, float weight_decay, float eps, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None) -> ()
11501: 
11502:       auto dispatch__fused_adagrad_ = [](at::TensorList self, at::TensorList grads, at::TensorList state_sums, at::TensorList state_steps, const at::Tensor & lr, double lr_decay, double weight_decay, double eps, bool maximize, const ::std::optional<at::Tensor> & grad_scale, const ::std::optional<at::Tensor> & found_inf) -> void {
11503:         pybind11::gil_scoped_release no_gil;
11504:         at::_fused_adagrad_(self, grads, state_sums, state_steps, lr, lr_decay, weight_decay, eps, maximize, grad_scale, found_inf);
11505:       };
11506:       dispatch__fused_adagrad_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.tensorlist(3), _r.tensor(4), _r.toDouble(5), _r.toDouble(6), _r.toDouble(7), _r.toBool(8), _r.optionalTensor(9), _r.optionalTensor(10));
11507:       Py_RETURN_NONE;
11508:     }
11509:     case 1: {
11510:       // aten::_fused_adagrad_(Tensor(a!)[] self, Tensor(b!)[] grads, Tensor(c!)[] state_sums, Tensor(d!)[] state_steps, *, float lr, float lr_decay, float weight_decay, float eps, bool maximize, Tensor? grad_scale=None, Tensor? found_inf=None) -> ()
11511: 
11512:       auto dispatch__fused_adagrad_ = [](at::TensorList self, at::TensorList grads, at::TensorList state_sums, at::TensorList state_steps, double lr, double lr_decay, double weight_decay, double eps, bool maximize, const ::std::optional<at::Tensor> & grad_scale, const ::std::optional<at::Tensor> & found_inf) -> void {
11513:         pybind11::gil_scoped_release no_gil;
11514:         at::_fused_adagrad_(self, grads, state_sums, state_steps, lr, lr_decay, weight_decay, eps, maximize, grad_scale, found_inf);
11515:       };
11516:       dispatch__fused_adagrad_(_r.tensorlist(0), _r.tensorlist(1), _r.tensorlist(2), _r.tensorlist(3), _r.toDouble(4), _r.toDouble(5), _r.toDouble(6), _r.toDouble(7), _r.toBool(8), _r.optionalTensor(9), _r.optionalTensor(10));
11517:       Py_RETURN_NONE;
11518:     }
11519:   }
11520:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `_fill_mem_eff_dropout_mask_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `_fill_mem_eff_dropout_mask_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11521-11549

```cpp
11521:   END_HANDLE_TH_ERRORS
11522: }
11523: 
11524: // _propagate_xla_data
11525: static PyObject * THPVariable__propagate_xla_data(PyObject* self_, PyObject* args, PyObject* kwargs)
11526: {
11527:   HANDLE_TH_ERRORS
11528:   static PythonArgParser parser({
11529:     "_propagate_xla_data(Tensor input, Tensor output)",
11530:   }, /*traceable=*/false);
11531: 
11532:   ParsedArgs<2> parsed_args;
11533:   auto _r = parser.parse(nullptr, args, kwargs, parsed_args);
11534:   if(_r.has_torch_function()) {
11535:     return handle_torch_function(_r, nullptr, args, kwargs, THPVariableFunctionsModule, "torch");
11536:   }
11537:   // aten::_propagate_xla_data(Tensor input, Tensor output) -> ()
11538: 
11539:   auto dispatch__propagate_xla_data = [](const at::Tensor & input, const at::Tensor & output) -> void {
11540:     pybind11::gil_scoped_release no_gil;
11541:     at::_propagate_xla_data(input, output);
11542:   };
11543:   dispatch__propagate_xla_data(_r.tensor(0), _r.tensor(1));
11544:   Py_RETURN_NONE;
11545:   Py_RETURN_NONE;
11546:   END_HANDLE_TH_ERRORS
11547: }
11548: 
11549: } // namespace torch::autograd
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable__propagate_xla_data`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable__propagate_xla_data`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理
- Primary symbol `THPVariable__cast_Char` / 核心符号 `THPVariable__cast_Char`

## Dependencies / 依赖关系
- Direct includes / 直接包含: `Python.h`, `torch/csrc/autograd/python_torch_functions.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/utils/wrap_outputs.h`, `torch/csrc/Dtype.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, `torch/csrc/utils/out_types.h`, `torch/csrc/utils/pybind.h`, `torch/csrc/utils/pycfunction_helpers.h`
- Include roots / 头文件根模块: `ATen`, `torch`
- Key symbols / 关键符号: `THPVariable__cast_Char`, `THPVariable__cast_Short`, `THPVariable__cast_Half`, `THPVariable__make_dual`, `THPVariable__unpack_dual`, `THPVariable__functional_assert_scalar`, `THPVariable__functional_assert_async`, `THPVariable_sym_constrain_range_for_size`, `THPVariable__functional_sym_constrain_range`, `THPVariable__cudnn_ctc_loss`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层

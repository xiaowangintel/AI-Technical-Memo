# python_variable_methods.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/autograd/generated/python_variable_methods.cpp`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Provides generated Python bindings that expose autograd- and operator-related APIs to CPython.
- 目的 (CN): 提供生成的 Python 绑定，把自动求导与算子相关 API 暴露给 CPython。
- Lines: 17866
- Language: C++ / C++

## Line-by-Line Analysis / 逐行分析

- Note / 说明: This file is generated or mechanically expanded, so the analysis groups contiguous line ranges rather than paraphrasing every repeated wrapper individually. / 该文件为生成代码或机械展开代码，因此分析按连续行范围组织，而不是逐个重复改写每个封装模板。
### Lines 1-120

```cpp
  1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
  2: // @generated from ../tools/autograd/templates/python_variable_methods.cpp
  3: 
  4: #include <Python.h>
  5: 
  6: // Undefine the copysign macro so that at::copysign works as intended with MSVC
  7: // https://github.com/python/cpython/blob/c60394c7fc9cc09b16e9675a3eeb5844b6d8523f/PC/pyconfig.h#L196
  8: #ifdef _MSC_VER
  9: #undef copysign
 10: #endif // _MSC_VER
 11: 
 12: #include "torch/csrc/DynamicTypes.h"
 13: #include "torch/csrc/Exceptions.h"
 14: #include "torch/csrc/Size.h"
 15: #include "torch/csrc/autograd/generated/VariableType.h"
 16: #include "torch/csrc/autograd/python_variable.h"
 17: #include "torch/csrc/autograd/utils/python_arg_parsing.h"
 18: #include "torch/csrc/autograd/utils/error_messages.h"
 19: #include "torch/csrc/autograd/utils/wrap_outputs.h"
 20: #include "torch/csrc/jit/frontend/tracer.h"
 21: #ifdef USE_CUDA
 22: #include "torch/csrc/cuda/Event.h"
 23: #endif
 24: #include "torch/csrc/utils/device_lazy_init.h"
 25: #include <torch/csrc/utils/numpy_stub.h>
 26: #include "torch/csrc/utils/object_ptr.h"
 27: #include "torch/csrc/utils/pycfunction_helpers.h"
 28: #include "torch/csrc/utils/python_arg_parser.h"
 29: #include "torch/csrc/utils/python_numbers.h"
 30: #include "torch/csrc/utils/python_strings.h"
 31: #include "torch/csrc/utils/tensor_apply.h"
 32: #include "torch/csrc/utils/tensor_list.h"
 33: #include "torch/csrc/utils/tensor_new.h"
 34: #include "torch/csrc/utils/tensor_numpy.h"
 35: #include "torch/csrc/utils/tensor_types.h"
 36: #include "torch/csrc/autograd/generated/python_return_types.h"
 37: 
 38: #include <ATen/core/Tensor.h>
 39: #include <ATen/core/grad_mode.h>
 40: #include <ATen/FuncTorchTLS.h>
 41: #include "c10/core/Stream.h"
 42: 
 43: #include <optional>
 44: #include <stdexcept>
 45: 
 46: #ifndef AT_PER_OPERATOR_HEADERS
 47: #include <ATen/Functions.h>
 48: #else
 49: #include <ATen/ops/and.h>
 50: #include <ATen/ops/and.h>
 51: #include <ATen/ops/lshift.h>
 52: #include <ATen/ops/or.h>
 53: #include <ATen/ops/rshift.h>
 54: #include <ATen/ops/xor.h>
 55: #include <ATen/ops/lshift.h>
 56: #include <ATen/ops/or.h>
 57: #include <ATen/ops/rshift.h>
 58: #include <ATen/ops/xor.h>
 59: #include <ATen/ops/_addmm_activation.h>
 60: #include <ATen/ops/_autocast_to_full_precision.h>
 61: #include <ATen/ops/_autocast_to_reduced_precision.h>
 62: #include <ATen/ops/_coalesced.h>
 63: #include <ATen/ops/_conj.h>
 64: #include <ATen/ops/_conj_physical.h>
 65: #include <ATen/ops/_dimI.h>
 66: #include <ATen/ops/_dimV.h>
 67: #include <ATen/ops/_indices.h>
 68: #include <ATen/ops/_is_all_true.h>
 69: #include <ATen/ops/_is_any_true.h>
 70: #include <ATen/ops/_is_zerotensor.h>
 71: #include <ATen/ops/_lazy_clone.h>
 72: #include <ATen/ops/_neg_view.h>
 73: #include <ATen/ops/_nested_tensor_size.h>
 74: #include <ATen/ops/_nested_tensor_storage_offsets.h>
 75: #include <ATen/ops/_nested_tensor_strides.h>
 76: #include <ATen/ops/_nnz.h>
 77: #include <ATen/ops/_philox_normal.h>
 78: #include <ATen/ops/_philox_uniform.h>
 79: #include <ATen/ops/_sparse_mask_projection.h>
 80: #include <ATen/ops/_to_dense.h>
 81: #include <ATen/ops/_to_sparse.h>
 82: #include <ATen/ops/_to_sparse_bsc.h>
 83: #include <ATen/ops/_to_sparse_bsr.h>
 84: #include <ATen/ops/_to_sparse_csc.h>
 85: #include <ATen/ops/_to_sparse_csr.h>
 86: #include <ATen/ops/_values.h>
 87: #include <ATen/ops/abs.h>
 88: #include <ATen/ops/abs.h>
 89: #include <ATen/ops/absolute.h>
 90: #include <ATen/ops/absolute.h>
 91: #include <ATen/ops/acos.h>
 92: #include <ATen/ops/acos.h>
 93: #include <ATen/ops/acosh.h>
 94: #include <ATen/ops/acosh.h>
 95: #include <ATen/ops/add.h>
 96: #include <ATen/ops/add.h>
 97: #include <ATen/ops/addbmm.h>
 98: #include <ATen/ops/addbmm.h>
 99: #include <ATen/ops/addcdiv.h>
100: #include <ATen/ops/addcdiv.h>
101: #include <ATen/ops/addcmul.h>
102: #include <ATen/ops/addcmul.h>
103: #include <ATen/ops/addmm.h>
104: #include <ATen/ops/addmm.h>
105: #include <ATen/ops/addmv.h>
106: #include <ATen/ops/addmv.h>
107: #include <ATen/ops/addr.h>
108: #include <ATen/ops/addr.h>
109: #include <ATen/ops/adjoint.h>
110: #include <ATen/ops/align_as.h>
111: #include <ATen/ops/align_to.h>
112: #include <ATen/ops/all.h>
113: #include <ATen/ops/allclose.h>
114: #include <ATen/ops/amax.h>
115: #include <ATen/ops/amin.h>
116: #include <ATen/ops/aminmax.h>
117: #include <ATen/ops/angle.h>
118: #include <ATen/ops/any.h>
119: #include <ATen/ops/arccos.h>
120: #include <ATen/ops/arccos.h>
```

- EN: These lines pull in dependencies such as `Python.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, establishing the headers needed by the implementation. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这些行引入了依赖，例如 `Python.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`，为后续实现建立所需的头文件基础。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 121-240

```cpp
121: #include <ATen/ops/arccosh.h>
122: #include <ATen/ops/arccosh.h>
123: #include <ATen/ops/arcsin.h>
124: #include <ATen/ops/arcsin.h>
125: #include <ATen/ops/arcsinh.h>
126: #include <ATen/ops/arcsinh.h>
127: #include <ATen/ops/arctan.h>
128: #include <ATen/ops/arctan2.h>
129: #include <ATen/ops/arctan2.h>
130: #include <ATen/ops/arctan.h>
131: #include <ATen/ops/arctanh.h>
132: #include <ATen/ops/arctanh.h>
133: #include <ATen/ops/argmax.h>
134: #include <ATen/ops/argmin.h>
135: #include <ATen/ops/argsort.h>
136: #include <ATen/ops/argwhere.h>
137: #include <ATen/ops/as_strided.h>
138: #include <ATen/ops/as_strided.h>
139: #include <ATen/ops/as_strided_scatter.h>
140: #include <ATen/ops/asin.h>
141: #include <ATen/ops/asin.h>
142: #include <ATen/ops/asinh.h>
143: #include <ATen/ops/asinh.h>
144: #include <ATen/ops/atan.h>
145: #include <ATen/ops/atan2.h>
146: #include <ATen/ops/atan2.h>
147: #include <ATen/ops/atan.h>
148: #include <ATen/ops/atanh.h>
149: #include <ATen/ops/atanh.h>
150: #include <ATen/ops/baddbmm.h>
151: #include <ATen/ops/baddbmm.h>
152: #include <ATen/ops/bernoulli.h>
153: #include <ATen/ops/bernoulli.h>
154: #include <ATen/ops/bincount.h>
155: #include <ATen/ops/bitwise_and.h>
156: #include <ATen/ops/bitwise_and.h>
157: #include <ATen/ops/bitwise_left_shift.h>
158: #include <ATen/ops/bitwise_left_shift.h>
159: #include <ATen/ops/bitwise_not.h>
160: #include <ATen/ops/bitwise_not.h>
161: #include <ATen/ops/bitwise_or.h>
162: #include <ATen/ops/bitwise_or.h>
163: #include <ATen/ops/bitwise_right_shift.h>
164: #include <ATen/ops/bitwise_right_shift.h>
165: #include <ATen/ops/bitwise_xor.h>
166: #include <ATen/ops/bitwise_xor.h>
167: #include <ATen/ops/bmm.h>
168: #include <ATen/ops/broadcast_to.h>
169: #include <ATen/ops/cauchy.h>
170: #include <ATen/ops/ccol_indices.h>
171: #include <ATen/ops/ceil.h>
172: #include <ATen/ops/ceil.h>
173: #include <ATen/ops/chalf.h>
174: #include <ATen/ops/cholesky.h>
175: #include <ATen/ops/cholesky_inverse.h>
176: #include <ATen/ops/cholesky_solve.h>
177: #include <ATen/ops/chunk.h>
178: #include <ATen/ops/clamp.h>
179: #include <ATen/ops/clamp.h>
180: #include <ATen/ops/clamp_max.h>
181: #include <ATen/ops/clamp_max.h>
182: #include <ATen/ops/clamp_min.h>
183: #include <ATen/ops/clamp_min.h>
184: #include <ATen/ops/clip.h>
185: #include <ATen/ops/clip.h>
186: #include <ATen/ops/clone.h>
187: #include <ATen/ops/coalesce.h>
188: #include <ATen/ops/col_indices.h>
189: #include <ATen/ops/conj.h>
190: #include <ATen/ops/conj_physical.h>
191: #include <ATen/ops/conj_physical.h>
192: #include <ATen/ops/copysign.h>
193: #include <ATen/ops/copysign.h>
194: #include <ATen/ops/corrcoef.h>
195: #include <ATen/ops/cos.h>
196: #include <ATen/ops/cos.h>
197: #include <ATen/ops/cosh.h>
198: #include <ATen/ops/cosh.h>
199: #include <ATen/ops/count_nonzero.h>
200: #include <ATen/ops/cov.h>
201: #include <ATen/ops/cross.h>
202: #include <ATen/ops/crow_indices.h>
203: #include <ATen/ops/cummax.h>
204: #include <ATen/ops/cummin.h>
205: #include <ATen/ops/cumprod.h>
206: #include <ATen/ops/cumprod.h>
207: #include <ATen/ops/cumsum.h>
208: #include <ATen/ops/cumsum.h>
209: #include <ATen/ops/deg2rad.h>
210: #include <ATen/ops/deg2rad.h>
211: #include <ATen/ops/dense_dim.h>
212: #include <ATen/ops/dequantize.h>
213: #include <ATen/ops/det.h>
214: #include <ATen/ops/detach.h>
215: #include <ATen/ops/detach.h>
216: #include <ATen/ops/diag.h>
217: #include <ATen/ops/diag_embed.h>
218: #include <ATen/ops/diagflat.h>
219: #include <ATen/ops/diagonal.h>
220: #include <ATen/ops/diagonal_scatter.h>
221: #include <ATen/ops/diff.h>
222: #include <ATen/ops/digamma.h>
223: #include <ATen/ops/digamma.h>
224: #include <ATen/ops/dist.h>
225: #include <ATen/ops/div.h>
226: #include <ATen/ops/div.h>
227: #include <ATen/ops/divide.h>
228: #include <ATen/ops/divide.h>
229: #include <ATen/ops/dot.h>
230: #include <ATen/ops/dsplit.h>
231: #include <ATen/ops/eq.h>
232: #include <ATen/ops/eq.h>
233: #include <ATen/ops/equal.h>
234: #include <ATen/ops/erf.h>
235: #include <ATen/ops/erf.h>
236: #include <ATen/ops/erfc.h>
237: #include <ATen/ops/erfc.h>
238: #include <ATen/ops/erfinv.h>
239: #include <ATen/ops/erfinv.h>
240: #include <ATen/ops/exp.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/arccosh.h`, `ATen/ops/arcsin.h`, `ATen/ops/arcsinh.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/arccosh.h`, `ATen/ops/arcsin.h`, `ATen/ops/arcsinh.h`，为后续实现建立所需的头文件基础。
### Lines 241-360

```cpp
241: #include <ATen/ops/exp2.h>
242: #include <ATen/ops/exp2.h>
243: #include <ATen/ops/exp.h>
244: #include <ATen/ops/expand.h>
245: #include <ATen/ops/expand_as.h>
246: #include <ATen/ops/expm1.h>
247: #include <ATen/ops/expm1.h>
248: #include <ATen/ops/exponential.h>
249: #include <ATen/ops/fill.h>
250: #include <ATen/ops/fill_diagonal.h>
251: #include <ATen/ops/fix.h>
252: #include <ATen/ops/fix.h>
253: #include <ATen/ops/flatten.h>
254: #include <ATen/ops/flip.h>
255: #include <ATen/ops/fliplr.h>
256: #include <ATen/ops/flipud.h>
257: #include <ATen/ops/float_power.h>
258: #include <ATen/ops/float_power.h>
259: #include <ATen/ops/floor.h>
260: #include <ATen/ops/floor.h>
261: #include <ATen/ops/floor_divide.h>
262: #include <ATen/ops/floor_divide.h>
263: #include <ATen/ops/fmax.h>
264: #include <ATen/ops/fmin.h>
265: #include <ATen/ops/fmod.h>
266: #include <ATen/ops/fmod.h>
267: #include <ATen/ops/frac.h>
268: #include <ATen/ops/frac.h>
269: #include <ATen/ops/frexp.h>
270: #include <ATen/ops/gather.h>
271: #include <ATen/ops/gcd.h>
272: #include <ATen/ops/gcd.h>
273: #include <ATen/ops/ge.h>
274: #include <ATen/ops/ge.h>
275: #include <ATen/ops/geometric.h>
276: #include <ATen/ops/geqrf.h>
277: #include <ATen/ops/ger.h>
278: #include <ATen/ops/greater.h>
279: #include <ATen/ops/greater.h>
280: #include <ATen/ops/greater_equal.h>
281: #include <ATen/ops/greater_equal.h>
282: #include <ATen/ops/gt.h>
283: #include <ATen/ops/gt.h>
284: #include <ATen/ops/hardshrink.h>
285: #include <ATen/ops/hash_tensor.h>
286: #include <ATen/ops/heaviside.h>
287: #include <ATen/ops/heaviside.h>
288: #include <ATen/ops/histc.h>
289: #include <ATen/ops/histogram.h>
290: #include <ATen/ops/hsplit.h>
291: #include <ATen/ops/hypot.h>
292: #include <ATen/ops/hypot.h>
293: #include <ATen/ops/i0.h>
294: #include <ATen/ops/i0.h>
295: #include <ATen/ops/igamma.h>
296: #include <ATen/ops/igamma.h>
297: #include <ATen/ops/igammac.h>
298: #include <ATen/ops/igammac.h>
299: #include <ATen/ops/index_add.h>
300: #include <ATen/ops/index_add.h>
301: #include <ATen/ops/index_copy.h>
302: #include <ATen/ops/index_copy.h>
303: #include <ATen/ops/index_fill.h>
304: #include <ATen/ops/index_fill.h>
305: #include <ATen/ops/index_put.h>
306: #include <ATen/ops/index_put.h>
307: #include <ATen/ops/index_reduce.h>
308: #include <ATen/ops/index_reduce.h>
309: #include <ATen/ops/index_select.h>
310: #include <ATen/ops/indices.h>
311: #include <ATen/ops/inner.h>
312: #include <ATen/ops/int_repr.h>
313: #include <ATen/ops/inverse.h>
314: #include <ATen/ops/is_coalesced.h>
315: #include <ATen/ops/is_complex.h>
316: #include <ATen/ops/is_conj.h>
317: #include <ATen/ops/is_distributed.h>
318: #include <ATen/ops/is_floating_point.h>
319: #include <ATen/ops/is_inference.h>
320: #include <ATen/ops/is_neg.h>
321: #include <ATen/ops/is_nonzero.h>
322: #include <ATen/ops/is_pinned.h>
323: #include <ATen/ops/is_same_size.h>
324: #include <ATen/ops/is_set_to.h>
325: #include <ATen/ops/is_signed.h>
326: #include <ATen/ops/isclose.h>
327: #include <ATen/ops/isfinite.h>
328: #include <ATen/ops/isinf.h>
329: #include <ATen/ops/isnan.h>
330: #include <ATen/ops/isneginf.h>
331: #include <ATen/ops/isposinf.h>
332: #include <ATen/ops/isreal.h>
333: #include <ATen/ops/istft.h>
334: #include <ATen/ops/kron.h>
335: #include <ATen/ops/kthvalue.h>
336: #include <ATen/ops/lcm.h>
337: #include <ATen/ops/lcm.h>
338: #include <ATen/ops/ldexp.h>
339: #include <ATen/ops/ldexp.h>
340: #include <ATen/ops/le.h>
341: #include <ATen/ops/le.h>
342: #include <ATen/ops/lerp.h>
343: #include <ATen/ops/lerp.h>
344: #include <ATen/ops/less.h>
345: #include <ATen/ops/less.h>
346: #include <ATen/ops/less_equal.h>
347: #include <ATen/ops/less_equal.h>
348: #include <ATen/ops/lgamma.h>
349: #include <ATen/ops/lgamma.h>
350: #include <ATen/ops/log.h>
351: #include <ATen/ops/log10.h>
352: #include <ATen/ops/log10.h>
353: #include <ATen/ops/log1p.h>
354: #include <ATen/ops/log1p.h>
355: #include <ATen/ops/log2.h>
356: #include <ATen/ops/log2.h>
357: #include <ATen/ops/log.h>
358: #include <ATen/ops/log_normal.h>
359: #include <ATen/ops/log_softmax.h>
360: #include <ATen/ops/logaddexp.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/exp2.h`, `ATen/ops/exp.h`, `ATen/ops/expand.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/exp2.h`, `ATen/ops/exp.h`, `ATen/ops/expand.h`，为后续实现建立所需的头文件基础。
### Lines 361-480

```cpp
361: #include <ATen/ops/logaddexp2.h>
362: #include <ATen/ops/logcumsumexp.h>
363: #include <ATen/ops/logdet.h>
364: #include <ATen/ops/logical_and.h>
365: #include <ATen/ops/logical_and.h>
366: #include <ATen/ops/logical_not.h>
367: #include <ATen/ops/logical_not.h>
368: #include <ATen/ops/logical_or.h>
369: #include <ATen/ops/logical_or.h>
370: #include <ATen/ops/logical_xor.h>
371: #include <ATen/ops/logical_xor.h>
372: #include <ATen/ops/logit.h>
373: #include <ATen/ops/logit.h>
374: #include <ATen/ops/logsumexp.h>
375: #include <ATen/ops/lt.h>
376: #include <ATen/ops/lt.h>
377: #include <ATen/ops/lu_solve.h>
378: #include <ATen/ops/masked_fill.h>
379: #include <ATen/ops/masked_fill.h>
380: #include <ATen/ops/masked_scatter.h>
381: #include <ATen/ops/masked_scatter.h>
382: #include <ATen/ops/masked_select.h>
383: #include <ATen/ops/matmul.h>
384: #include <ATen/ops/matrix_exp.h>
385: #include <ATen/ops/matrix_power.h>
386: #include <ATen/ops/max.h>
387: #include <ATen/ops/maximum.h>
388: #include <ATen/ops/mean.h>
389: #include <ATen/ops/median.h>
390: #include <ATen/ops/min.h>
391: #include <ATen/ops/minimum.h>
392: #include <ATen/ops/mm.h>
393: #include <ATen/ops/mode.h>
394: #include <ATen/ops/moveaxis.h>
395: #include <ATen/ops/movedim.h>
396: #include <ATen/ops/msort.h>
397: #include <ATen/ops/mul.h>
398: #include <ATen/ops/mul.h>
399: #include <ATen/ops/multinomial.h>
400: #include <ATen/ops/multiply.h>
401: #include <ATen/ops/multiply.h>
402: #include <ATen/ops/mv.h>
403: #include <ATen/ops/mvlgamma.h>
404: #include <ATen/ops/mvlgamma.h>
405: #include <ATen/ops/nan_to_num.h>
406: #include <ATen/ops/nan_to_num.h>
407: #include <ATen/ops/nanmean.h>
408: #include <ATen/ops/nanmedian.h>
409: #include <ATen/ops/nanquantile.h>
410: #include <ATen/ops/nansum.h>
411: #include <ATen/ops/narrow.h>
412: #include <ATen/ops/narrow_copy.h>
413: #include <ATen/ops/ne.h>
414: #include <ATen/ops/ne.h>
415: #include <ATen/ops/neg.h>
416: #include <ATen/ops/neg.h>
417: #include <ATen/ops/negative.h>
418: #include <ATen/ops/negative.h>
419: #include <ATen/ops/new_empty.h>
420: #include <ATen/ops/new_empty_strided.h>
421: #include <ATen/ops/new_full.h>
422: #include <ATen/ops/new_ones.h>
423: #include <ATen/ops/new_zeros.h>
424: #include <ATen/ops/nextafter.h>
425: #include <ATen/ops/nextafter.h>
426: #include <ATen/ops/nonzero_static.h>
427: #include <ATen/ops/norm.h>
428: #include <ATen/ops/normal.h>
429: #include <ATen/ops/not_equal.h>
430: #include <ATen/ops/not_equal.h>
431: #include <ATen/ops/orgqr.h>
432: #include <ATen/ops/ormqr.h>
433: #include <ATen/ops/outer.h>
434: #include <ATen/ops/permute.h>
435: #include <ATen/ops/pin_memory.h>
436: #include <ATen/ops/pinverse.h>
437: #include <ATen/ops/polygamma.h>
438: #include <ATen/ops/polygamma.h>
439: #include <ATen/ops/positive.h>
440: #include <ATen/ops/pow.h>
441: #include <ATen/ops/pow.h>
442: #include <ATen/ops/prelu.h>
443: #include <ATen/ops/prod.h>
444: #include <ATen/ops/put.h>
445: #include <ATen/ops/put.h>
446: #include <ATen/ops/q_per_channel_axis.h>
447: #include <ATen/ops/q_per_channel_scales.h>
448: #include <ATen/ops/q_per_channel_zero_points.h>
449: #include <ATen/ops/q_scale.h>
450: #include <ATen/ops/q_zero_point.h>
451: #include <ATen/ops/qr.h>
452: #include <ATen/ops/qscheme.h>
453: #include <ATen/ops/quantile.h>
454: #include <ATen/ops/rad2deg.h>
455: #include <ATen/ops/rad2deg.h>
456: #include <ATen/ops/random.h>
457: #include <ATen/ops/ravel.h>
458: #include <ATen/ops/reciprocal.h>
459: #include <ATen/ops/reciprocal.h>
460: #include <ATen/ops/record_stream.h>
461: #include <ATen/ops/refine_names.h>
462: #include <ATen/ops/relu.h>
463: #include <ATen/ops/relu.h>
464: #include <ATen/ops/remainder.h>
465: #include <ATen/ops/remainder.h>
466: #include <ATen/ops/rename.h>
467: #include <ATen/ops/rename.h>
468: #include <ATen/ops/renorm.h>
469: #include <ATen/ops/renorm.h>
470: #include <ATen/ops/repeat.h>
471: #include <ATen/ops/repeat_interleave.h>
472: #include <ATen/ops/reshape.h>
473: #include <ATen/ops/reshape_as.h>
474: #include <ATen/ops/resize.h>
475: #include <ATen/ops/resize_as.h>
476: #include <ATen/ops/resize_as_sparse.h>
477: #include <ATen/ops/resolve_conj.h>
478: #include <ATen/ops/resolve_neg.h>
479: #include <ATen/ops/retain_grad.h>
480: #include <ATen/ops/roll.h>
```

- EN: These lines pull in dependencies such as `ATen/ops/logaddexp2.h`, `ATen/ops/logcumsumexp.h`, `ATen/ops/logdet.h`, establishing the headers needed by the implementation.
- CN: 这些行引入了依赖，例如 `ATen/ops/logaddexp2.h`, `ATen/ops/logcumsumexp.h`, `ATen/ops/logdet.h`，为后续实现建立所需的头文件基础。
### Lines 481-600

```cpp
481: #include <ATen/ops/rot90.h>
482: #include <ATen/ops/round.h>
483: #include <ATen/ops/round.h>
484: #include <ATen/ops/row_indices.h>
485: #include <ATen/ops/rsqrt.h>
486: #include <ATen/ops/rsqrt.h>
487: #include <ATen/ops/scatter.h>
488: #include <ATen/ops/scatter.h>
489: #include <ATen/ops/scatter_add.h>
490: #include <ATen/ops/scatter_add.h>
491: #include <ATen/ops/scatter_reduce.h>
492: #include <ATen/ops/scatter_reduce.h>
493: #include <ATen/ops/select.h>
494: #include <ATen/ops/select_scatter.h>
495: #include <ATen/ops/sgn.h>
496: #include <ATen/ops/sgn.h>
497: #include <ATen/ops/sigmoid.h>
498: #include <ATen/ops/sigmoid.h>
499: #include <ATen/ops/sign.h>
500: #include <ATen/ops/sign.h>
501: #include <ATen/ops/signbit.h>
502: #include <ATen/ops/sin.h>
503: #include <ATen/ops/sin.h>
504: #include <ATen/ops/sinc.h>
505: #include <ATen/ops/sinc.h>
506: #include <ATen/ops/sinh.h>
507: #include <ATen/ops/sinh.h>
508: #include <ATen/ops/slice_inverse.h>
509: #include <ATen/ops/slice_scatter.h>
510: #include <ATen/ops/slogdet.h>
511: #include <ATen/ops/smm.h>
512: #include <ATen/ops/softmax.h>
513: #include <ATen/ops/sort.h>
514: #include <ATen/ops/sparse_dim.h>
515: #include <ATen/ops/sparse_mask.h>
516: #include <ATen/ops/sparse_resize.h>
517: #include <ATen/ops/sparse_resize_and_clear.h>
518: #include <ATen/ops/split.h>
519: #include <ATen/ops/split_with_sizes.h>
520: #include <ATen/ops/sqrt.h>
521: #include <ATen/ops/sqrt.h>
522: #include <ATen/ops/square.h>
523: #include <ATen/ops/square.h>
524: #include <ATen/ops/squeeze.h>
525: #include <ATen/ops/squeeze.h>
526: #include <ATen/ops/sspaddmm.h>
527: #include <ATen/ops/std.h>
528: #include <ATen/ops/stft.h>
529: #include <ATen/ops/sub.h>
530: #include <ATen/ops/sub.h>
531: #include <ATen/ops/subtract.h>
532: #include <ATen/ops/subtract.h>
533: #include <ATen/ops/sum.h>
534: #include <ATen/ops/sum_to_size.h>
535: #include <ATen/ops/svd.h>
536: #include <ATen/ops/swapaxes.h>
537: #include <ATen/ops/swapaxes.h>
538: #include <ATen/ops/swapdims.h>
539: #include <ATen/ops/swapdims.h>
540: #include <ATen/ops/t.h>
541: #include <ATen/ops/t.h>
542: #include <ATen/ops/take.h>
543: #include <ATen/ops/take_along_dim.h>
544: #include <ATen/ops/tan.h>
545: #include <ATen/ops/tan.h>
546: #include <ATen/ops/tanh.h>
547: #include <ATen/ops/tanh.h>
548: #include <ATen/ops/tensor_split.h>
549: #include <ATen/ops/tile.h>
550: #include <ATen/ops/to_dense.h>
551: #include <ATen/ops/to_mkldnn.h>
552: #include <ATen/ops/to_padded_tensor.h>
553: #include <ATen/ops/to_sparse.h>
554: #include <ATen/ops/to_sparse_bsc.h>
555: #include <ATen/ops/to_sparse_bsr.h>
556: #include <ATen/ops/to_sparse_csc.h>
557: #include <ATen/ops/to_sparse_csr.h>
558: #include <ATen/ops/topk.h>
559: #include <ATen/ops/trace.h>
560: #include <ATen/ops/transpose.h>
561: #include <ATen/ops/transpose.h>
562: #include <ATen/ops/triangular_solve.h>
563: #include <ATen/ops/tril.h>
564: #include <ATen/ops/tril.h>
565: #include <ATen/ops/triu.h>
566: #include <ATen/ops/triu.h>
567: #include <ATen/ops/true_divide.h>
568: #include <ATen/ops/true_divide.h>
569: #include <ATen/ops/trunc.h>
570: #include <ATen/ops/trunc.h>
571: #include <ATen/ops/type_as.h>
572: #include <ATen/ops/unbind.h>
573: #include <ATen/ops/unflatten.h>
574: #include <ATen/ops/unfold.h>
575: #include <ATen/ops/uniform.h>
576: #include <ATen/ops/unsafe_chunk.h>
577: #include <ATen/ops/unsafe_split.h>
578: #include <ATen/ops/unsafe_split_with_sizes.h>
579: #include <ATen/ops/unsqueeze.h>
580: #include <ATen/ops/unsqueeze.h>
581: #include <ATen/ops/values.h>
582: #include <ATen/ops/var.h>
583: #include <ATen/ops/vdot.h>
584: #include <ATen/ops/view.h>
585: #include <ATen/ops/view_as.h>
586: #include <ATen/ops/vsplit.h>
587: #include <ATen/ops/where.h>
588: #include <ATen/ops/xlogy.h>
589: #include <ATen/ops/xlogy.h>
590: #include <ATen/ops/zero.h>
591: #include <ATen/ops/_local_scalar_dense.h>
592: #endif
593: 
594: using at::device_of;
595: using at::OptionalDeviceGuard;
596: using at::Scalar;
597: using at::ScalarType;
598: using at::Tensor;
599: using c10::Stream;
600: using namespace torch::autograd::utils;
```

- EN: These lines pull in dependencies such as `ATen/ops/rot90.h`, `ATen/ops/round.h`, `ATen/ops/row_indices.h`, establishing the headers needed by the implementation. They also open, refine, or close the relevant C++ namespace scope.
- CN: 这些行引入了依赖，例如 `ATen/ops/rot90.h`, `ATen/ops/round.h`, `ATen/ops/row_indices.h`，为后续实现建立所需的头文件基础。 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。
### Lines 601-720

```cpp
601: 
602: namespace torch::autograd {
603: 
604: static PyObject * THPVariable__is_view(PyObject *self, PyObject* args)
605: {
606:   HANDLE_TH_ERRORS
607:   if (has_torch_function(self)) {
608:     return handle_torch_function(self, "_is_view", args);
609:   }
610:   auto& self_ = THPVariable_Unpack(self);
611:   if (self_.is_view()) {
612:     Py_RETURN_TRUE;
613:   } else {
614:     Py_RETURN_FALSE;
615:   }
616:   END_HANDLE_TH_ERRORS
617: }
618: 
619: // implemented on the python object bc no support for first-class functions in native_functions.yaml
620: // See: ATen/native/README.md for more context
621: static PyObject * THPVariable_apply_(PyObject* self, PyObject* arg)
622: {
623:   HANDLE_TH_ERRORS
624:   if (has_torch_function(self)) {
625:     auto args = py::make_tuple(py::handle(arg));
626:     return handle_torch_function(self, "apply_", args.ptr());
627:   }
628:   auto& self_ = THPVariable_Unpack(self);
629:   if (self_.requires_grad()) {
630:     throw std::runtime_error(
631:         "Can't call apply_() on Variable that requires grad. Use "
632:         "var.detach().apply_() instead.");
633:   }
634:   return THPVariable_Wrap(torch::utils::apply_(self_, arg));
635:   END_HANDLE_TH_ERRORS
636: }
637: 
638: static PyObject * THPVariable_size(PyObject* self, PyObject* args, PyObject* kwargs)
639: {
640:   HANDLE_TH_ERRORS
641:   static PythonArgParser parser({
642:     "size(int64_t? dim=None)",
643:     "size(Dimname dim)",
644:   });
645:   auto& self_ = THPVariable_Unpack(self);
646:   ParsedArgs<3> parsed_args;
647:   auto r = parser.parse(self, args, kwargs, parsed_args);
648: 
649:   if(r.has_torch_function()){
650:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
651:   }
652:   if (r.idx == 0) {
653:     if (!r.toInt64Optional(0).has_value()) {
654:       return THPSize_NewFromSymSizes(self_);
655:     }
656:     if (jit::tracer::isTracing()) {
657:       // will error out if a tensor has symints
658:       return wrap(jit::tracer::getSizeOf(self_, r.toInt64(0)));
659:     } else {
660:       return torch::toPyObject(self_.sym_size(r.toInt64(0)));
661:     }
662:   } else if (r.idx == 1) {
663:     if (jit::tracer::isTracing()) {
664:       TORCH_INTERNAL_ASSERT(false, "NYI: Named tensors w/ JIT");
665:     }
666:     return wrap(self_.size(r.dimname(0)));
667:   }
668:   Py_RETURN_NONE;
669:   END_HANDLE_TH_ERRORS
670: }
671: 
672: static PyObject * THPVariable_stride(PyObject* self, PyObject* args, PyObject* kwargs)
673: {
674:   HANDLE_TH_ERRORS
675:   static PythonArgParser parser({
676:     "stride(int64_t? dim=None)",
677:     "stride(Dimname dim)",
678:   });
679:   auto& self_ = THPVariable_Unpack(self);
680:   ParsedArgs<3> parsed_args;
681:   auto r = parser.parse(self, args, kwargs, parsed_args);
682: 
683:   if(r.has_torch_function()){
684:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
685:   }
686: 
687:   if (r.idx == 0) {
688:     if (r.toInt64Optional(0).has_value()) {
689:       return torch::toPyObject(self_.sym_stride(r.toInt64(0)));
690:     }
691:     // yes, this is called strides in ATen.
692:     at::SymIntArrayRef strides = self_.sym_strides();
693:     // we can't do the normal wrapping here because IntArrayRef maps to both
694:     // torch.Size and tuple in python
695:     // TODO: consider factoring this out
696:     THPObjectPtr tuple(PyTuple_New(static_cast<Py_ssize_t>(strides.size())));
697:     if (!tuple) throw python_error();
698:     for (size_t i = 0; i != strides.size(); i++) {
699:       PyObject* s = torch::toPyObject(strides[i]);
700:       if (!s) throw python_error();
701:       PyTuple_SET_ITEM(tuple.get(), i, s);
702:     }
703:     return tuple.release();
704:   } else if (r.idx == 1) {
705:     return wrap(self_.stride(r.dimname(0)));
706:   }
707:   Py_RETURN_NONE;
708:   END_HANDLE_TH_ERRORS
709: }
710: 
711: // implemented on the python object to avoid dispatch overhead
712: static PyObject * THPVariable_get_device(PyObject* self_, PyObject* args)
713: {
714:   HANDLE_TH_ERRORS
715:   if (has_torch_function(self_)) {
716:     return handle_torch_function(self_, "get_device", args, nullptr);
717:   }
718:   auto& self = THPVariable_Unpack(self_);
719:   return wrap(self.get_device());
720:   END_HANDLE_TH_ERRORS
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `THPVariable__is_view`, `handle_torch_function`, `THPVariable_Unpack`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `THPVariable__is_view`, `handle_torch_function`, `THPVariable_Unpack` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 721-840

```cpp
721: }
722: 
723: static PyObject * THPVariable_has_names(PyObject* self_, PyObject* args)
724: {
725:   HANDLE_TH_ERRORS
726:   if (has_torch_function(self_)) {
727:     return handle_torch_function(self_, "has_names", args);
728:   }
729:   auto& self = THPVariable_Unpack(self_);
730:   return wrap(self.has_names());
731:   END_HANDLE_TH_ERRORS
732: }
733: 
734: // implemented on the python object to avoid dispatch overhead
735: static PyObject * THPVariable_data_ptr(PyObject* self_, PyObject* args)
736: {
737:   HANDLE_TH_ERRORS
738:   if (has_torch_function(self_)) {
739:     return handle_torch_function(self_, "data_ptr", args);
740:   }
741:   auto& self = THPVariable_Unpack(self_);
742:   return wrap(self.data_ptr());
743:   END_HANDLE_TH_ERRORS
744: }
745: 
746: // implemented on the python object to avoid dispatch overhead
747: // Unlike data_ptr(), this is a read-only access that does not trigger
748: // copy-on-write materialization.
749: static PyObject * THPVariable_const_data_ptr(PyObject* self_, PyObject* args)
750: {
751:   HANDLE_TH_ERRORS
752:   if (check_has_torch_function(self_)) {
753:     return handle_torch_function(self_, "const_data_ptr", args);
754:   }
755:   auto& self = THPVariable_Unpack(self_);
756:   return wrap(const_cast<void*>(self.const_data_ptr()));
757:   END_HANDLE_TH_ERRORS
758: }
759: 
760: // implemented on the python object to avoid dispatch overhead
761: static PyObject * THPVariable_storage_offset(PyObject* self_, PyObject* args)
762: {
763:   HANDLE_TH_ERRORS
764:   if (has_torch_function(self_)) {
765:     return handle_torch_function(self_, "storage_offset");
766:   }
767:   auto& self = THPVariable_Unpack(self_);
768:   return py::cast(self.sym_storage_offset()).release().ptr();
769:   END_HANDLE_TH_ERRORS
770: }
771: 
772: // implemented on the python object to avoid dispatch overhead
773: static PyObject * THPVariable_dim(PyObject* self, PyObject* args)
774: {
775:    HANDLE_TH_ERRORS
776:    if (has_torch_function(self)) {
777:      return handle_torch_function(self, "dim", args);
778:    }
779:    auto& self_ = THPVariable_Unpack(self);
780:    return THPUtils_packInt64(self_.dim());
781:    END_HANDLE_TH_ERRORS
782: }
783: 
784: // implemented on the python object to avoid dispatch overhead
785: static PyObject * THPVariable_numel(PyObject* self, PyObject* args)
786: {
787:    HANDLE_TH_ERRORS
788:    if (has_torch_function(self)) {
789:      return handle_torch_function(self, "numel", args);
790:    }
791:    auto& self_ = THPVariable_Unpack(self);
792:    if (jit::tracer::isTracing()) {
793:      return wrap(jit::tracer::getNumelOf(self_));
794:    } else {
795:      return py::cast(self_.sym_numel()).release().ptr();
796:    }
797:    END_HANDLE_TH_ERRORS
798: }
799: 
800: static Tensor dispatch_contiguous(const Tensor & self, at::MemoryFormat memory_format) {
801:   pybind11::gil_scoped_release no_gil;
802:   OptionalDeviceGuard device_guard(device_of(self));
803:   return self.contiguous(memory_format);
804: }
805: 
806: static PyObject * THPVariable_contiguous(PyObject* self, PyObject* args, PyObject* kwargs)
807: {
808:   HANDLE_TH_ERRORS
809:   static PythonArgParser parser({
810:     "contiguous(*, MemoryFormat memory_format=contiguous_format)",
811:   });
812:   ParsedArgs<1> parsed_args;
813:   auto r = parser.parse(self, args, kwargs, parsed_args);
814: 
815:   if(r.has_torch_function()){
816:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
817:   }
818: 
819:   auto& self_ = THPVariable_Unpack(self);
820:   auto memory_format = r.memoryformat(0);
821:   // avoids touching the GIL or current device if self is already contiguous
822:   if (self_.is_contiguous_or_false(memory_format)) {
823:     // NOTE: this logic is duplicated from VariableType.cpp. Since we need to
824:     // record this call to contiguous() in the trace regardless of whether
825:     // we actually call contiguous here, we need to record this information
826:     // manually.
827:     if (jit::tracer::isTracing()) {
828:       const auto& tracer_state = jit::tracer::getTracingState();
829:       auto op_name = c10::Symbol::fromQualString("aten::contiguous");
830:       auto node = tracer_state->createNode(op_name, /*num_outputs=*/0);
831:       jit::tracer::recordSourceLocation(node);
832:       jit::tracer::addInputs(node, "self", self_);
833:       jit::tracer::addInputs(node, "memory_format", memory_format);
834:       tracer_state->insertNode(node);
835:       jit::tracer::addOutput(node, self_);
836:     }
837:     Py_INCREF(self);
838:     return self;
839:   }
840:   return THPVariable_Wrap(dispatch_contiguous(self_, memory_format));
```

- EN: The main execution path in this span is carried by `THPVariable_has_names`, `handle_torch_function`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_has_names`, `handle_torch_function`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 841-960

```cpp
841:   END_HANDLE_TH_ERRORS
842: }
843: 
844: static Tensor dispatch_copy_(const Tensor & self, const Tensor & other, bool non_blocking) {
845:   pybind11::gil_scoped_release no_gil;
846:   OptionalDeviceGuard device_guard(device_of(self));
847:   return self.copy_(other, non_blocking);
848: }
849: 
850: static void maybe_warn_requires_grad(const Tensor & self) {
851:   if (at::GradMode::is_enabled() && self.requires_grad()) {
852:     TORCH_WARN_ONCE("Converting a tensor with requires_grad=True to a scalar may lead to unexpected behavior.\n"
853:                     "Consider using tensor.detach() first.");
854:   }
855: }
856: 
857:  static PyObject * THPVariable_copy_(PyObject* self, PyObject* args, PyObject* kwargs)
858: {
859:   HANDLE_TH_ERRORS
860:   static PythonArgParser parser({
861:     "copy_(Tensor other, bool non_blocking=False)",
862:     "copy_(Tensor other, bool async=False)|deprecated"
863:   });
864:   auto& self_ = THPVariable_Unpack(self);
865:   ParsedArgs<2> parsed_args;
866:   auto r = parser.parse(self, args, kwargs, parsed_args);
867: 
868:   if(r.has_torch_function()){
869:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
870:   }
871: 
872:   return THPVariable_Wrap(dispatch_copy_(self_, r.tensor(0), r.toBool(1)));
873:   END_HANDLE_TH_ERRORS
874: }
875: 
876: template<typename T>
877: static T dispatch_to(const Tensor & self) {
878:   pybind11::gil_scoped_release no_gil;
879:   OptionalDeviceGuard device_guard(device_of(self));
880:   TORCH_CHECK_VALUE(self.sym_numel() == 1, "only one element tensors can be converted to Python scalars");
881:   return self.template item<T>();
882: }
883: 
884: static PyObject * THPVariable_float_scalar(PyObject* self, PyObject* args) {
885:   HANDLE_TH_ERRORS
886:   if (has_torch_function(self)) {
887:     return handle_torch_function(self, "__float__", args);
888:   }
889:   jit::tracer::warn("Converting a tensor to a Python float", jit::tracer::WARN_PYTHON_DATAFLOW);
890:   auto& self_ = THPVariable_Unpack(self);
891:   maybe_warn_requires_grad(self_);
892:   return wrap(dispatch_to<double>(self_));
893:   END_HANDLE_TH_ERRORS
894: }
895: 
896: static PyObject * THPVariable_complex_scalar(PyObject* self, PyObject* args) {
897:   HANDLE_TH_ERRORS
898:   if (has_torch_function(self)) {
899:     return handle_torch_function(self, "__complex__", args);
900:   }
901:   jit::tracer::warn("Converting a tensor to a Python complex", jit::tracer::WARN_PYTHON_DATAFLOW);
902:   auto& self_ = THPVariable_Unpack(self);
903:   maybe_warn_requires_grad(self_);
904:   return wrap(dispatch_to<c10::complex<double>>(self_));
905:   END_HANDLE_TH_ERRORS
906: }
907: 
908: static PyObject * THPVariable_integral_scalar(PyObject* self, PyObject* args) {
909:   HANDLE_TH_ERRORS
910:   if (has_torch_function(self)) {
911:     return handle_torch_function(self, "__int__", args);
912:   }
913:   jit::tracer::warn("Converting a tensor to a Python integer", jit::tracer::WARN_PYTHON_DATAFLOW);
914:   auto& self_ = THPVariable_Unpack(self);
915:   if (isFloatingType(self_.scalar_type())) {
916:     // we can't dispatch to item<int64_t> here because we want to avoid ATen overflow checks;
917:     // the python integral type (long in python2) can't overflow.
918:     return THPUtils_packDoubleAsInt(dispatch_to<double>(self_));
919:   } else {
920:     return wrap(dispatch_to<int64_t>(self_));
921:   }
922:   END_HANDLE_TH_ERRORS
923: }
924: 
925: // This is the __index__ function in Python which is similar to __int__, but
926: // called when used as a slice.
927: static PyObject * THPVariable_index_scalar(PyObject* self, PyObject* args) {
928:   HANDLE_TH_ERRORS
929:   if (has_torch_function(self)) {
930:     return handle_torch_function(self, "__index__", args);
931:   }
932:   auto& self_ = THPVariable_Unpack(self);
933:   // TODO: change the condition to `self_.dim() != 0` once we expose scalars
934:   // in PyTorch.
935:   if (!isIntegralType(self_.scalar_type(), /*includeBool=*/true) || self_.sym_numel() != 1) {
936:     throw TypeError("only integer tensors of a single element can be converted to an index");
937:   }
938:   return wrap(dispatch_to<int64_t>(self_));
939:   END_HANDLE_TH_ERRORS
940: }
941: 
942: static Tensor dispatch_invert(const Tensor & self) {
943:   pybind11::gil_scoped_release no_gil;
944:   OptionalDeviceGuard device_guard(device_of(self));
945:   return self.bitwise_not();
946: }
947: 
948: static PyObject * THPVariable_invert(PyObject* self, PyObject* args) {
949:   HANDLE_TH_ERRORS
950:   if (has_torch_function(self)) {
951:     return handle_torch_function(self, "__invert__", args);
952:   }
953:   auto& self_ = THPVariable_Unpack(self);
954:   if (!isIntegralType(self_.scalar_type(), /*includeBool=*/true)) {
955:     throw TypeError("~ (operator.invert) is only implemented on integer and Boolean-type tensors");
956:   }
957:   return THPVariable_Wrap(dispatch_invert(self_));
958:   END_HANDLE_TH_ERRORS
959: }
960: 
```

- EN: The main execution path in this span is carried by `dispatch_copy_`, `device_guard`, `maybe_warn_requires_grad`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `dispatch_copy_`, `device_guard`, `maybe_warn_requires_grad` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 961-1080

```cpp
 961: static Tensor dispatch_to(const Tensor & self, Device device, bool non_blocking, bool copy, std::optional<c10::MemoryFormat> optional_memory_format) {
 962:   pybind11::gil_scoped_release no_gil;
 963:   // NOTE: this is where we record aten::to in the graph during tracing. However, the behavior of aten::to
 964:   // is different with respect to TensorOptions fields that are not present: aten::to inherits fields that
 965:   // are missing from the self argument while the tracer assumes that they should be populated with the
 966:   // default values (eg. float for scalar type). By explicitly copying over the tensor options here we fully
 967:   // specify all tensor options and thus record the proper trace
 968:   return self.to(self.options().device(device).memory_format(optional_memory_format), non_blocking, copy);
 969: }
 970: 
 971: static Tensor dispatch_to(const Tensor & self, bool non_blocking, bool copy, std::optional<c10::MemoryFormat> optional_memory_format) {
 972:   pybind11::gil_scoped_release no_gil;
 973:   return self.to(self.options().memory_format(optional_memory_format), non_blocking, copy);
 974: }
 975: 
 976: static Tensor dispatch_to(const Tensor & self, ScalarType dtype, bool non_blocking, bool copy, std::optional<c10::MemoryFormat> optional_memory_format) {
 977:   pybind11::gil_scoped_release no_gil;
 978:   // TODO: Make this call the TensorOptions version, maybe?
 979:   return self.to(dtype, non_blocking, copy, optional_memory_format);
 980: }
 981: 
 982: static Tensor dispatch_to(const Tensor & self, Device device, ScalarType dtype, bool non_blocking, bool copy, std::optional<c10::MemoryFormat> optional_memory_format) {
 983:   pybind11::gil_scoped_release no_gil;
 984:   // TODO: Make this call the TensorOptions version, maybe?
 985:   return self.to(device, dtype, non_blocking, copy, optional_memory_format);
 986: }
 987: 
 988: static PyObject * THPVariable_cpu(PyObject* self, PyObject* args, PyObject* kwargs)
 989: {
 990:    HANDLE_TH_ERRORS
 991:    static PythonArgParser parser({
 992:      "cpu(*, MemoryFormat? memory_format=None)"
 993:    });
 994:    auto& self_ = THPVariable_Unpack(self);
 995:    ParsedArgs<1> parsed_args;
 996:    auto r = parser.parse(self, args, kwargs, parsed_args);
 997: 
 998:    if(r.has_torch_function()){
 999:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1000:     }
1001: 
1002:    auto opt_memory_format = r.memoryformatOptional(0);
1003:    return THPVariable_Wrap(dispatch_to(self_, at::Device(at::DeviceType::CPU), false, false, opt_memory_format));
1004:    END_HANDLE_TH_ERRORS
1005: }
1006: 
1007: static Tensor dispatch_nonzero(const Tensor & self) {
1008:   pybind11::gil_scoped_release no_gil;
1009:   OptionalDeviceGuard device_guard(device_of(self));
1010:   return self.nonzero();
1011: }
1012: 
1013: static std::vector<Tensor> dispatch_nonzero_numpy(const Tensor & self) {
1014:   pybind11::gil_scoped_release no_gil;
1015:   OptionalDeviceGuard device_guard(device_of(self));
1016:   return self.nonzero_numpy();
1017: }
1018: 
1019: static PyObject * THPVariable_nonzero(PyObject* self, PyObject* args, PyObject* kwargs)
1020: {
1021:   HANDLE_TH_ERRORS
1022:   static PythonArgParser parser({
1023:     "nonzero()",
1024:     "nonzero(*, bool as_tuple)",
1025:   });
1026:   auto& self_ = THPVariable_Unpack(self);
1027:   ParsedArgs<2> parsed_args;
1028:   auto r = parser.parse(self, args, kwargs, parsed_args);
1029: 
1030:   if(r.has_torch_function()){
1031:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1032:   }
1033: 
1034:   if (r.idx == 0 || (r.idx == 1 && !r.toBool(0))) {
1035:     return wrap(dispatch_nonzero(self_));
1036:   } else {
1037:     return wrap(dispatch_nonzero_numpy(self_));
1038:   }
1039:   END_HANDLE_TH_ERRORS
1040: }
1041: 
1042: static PyObject * THPVariable_cuda(PyObject* self, PyObject* args, PyObject* kwargs)
1043: {
1044:   HANDLE_TH_ERRORS
1045:   static PythonArgParser parser({
1046:     "cuda(Device? device=None, bool non_blocking=False, *, MemoryFormat? memory_format=None)",
1047:     "cuda(Device? device=None, bool async=False, *, MemoryFormat? memory_format=None)|deprecated"
1048:   });
1049:   auto& self_ = THPVariable_Unpack(self);
1050:   ParsedArgs<3> parsed_args;
1051:   auto r = parser.parse(self, args, kwargs, parsed_args);
1052: 
1053:   if(r.has_torch_function()){
1054:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1055:   }
1056: 
1057:   auto device = r.isNone(0) ? at::Device(at::DeviceType::CUDA) : r.device(0);
1058:   auto opt_memory_format = r.memoryformatOptional(2);
1059:   TORCH_CHECK(device.is_cuda(), "Invalid device, must be cuda device");
1060:   torch::utils::device_lazy_init(at::kCUDA);
1061:   return THPVariable_Wrap(dispatch_to(self_, device, r.toBool(1), false, opt_memory_format));
1062:   END_HANDLE_TH_ERRORS
1063: }
1064: 
1065: static PyObject * THPVariable_mtia(PyObject* self, PyObject* args, PyObject* kwargs)
1066: {
1067:   HANDLE_TH_ERRORS
1068:   static PythonArgParser parser({
1069:     "mtia(Device? device=None, bool non_blocking=False, *, MemoryFormat? memory_format=None)",
1070:     "mtia(Device? device=None, bool async=False, *, MemoryFormat? memory_format=None)|deprecated"
1071:   });
1072:   auto& self_ = THPVariable_Unpack(self);
1073:   ParsedArgs<3> parsed_args;
1074:   auto r = parser.parse(self, args, kwargs, parsed_args);
1075: 
1076:   if (r.has_torch_function()) {
1077:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1078:   }
1079: 
1080:   auto device = r.isNone(0) ? at::Device(at::DeviceType::MTIA) : r.device(0);
```

- EN: The main execution path in this span is carried by `dispatch_to`, `values`, `THPVariable_cpu`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `dispatch_to`, `values`, `THPVariable_cpu` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1081-1200

```cpp
1081:   auto opt_memory_format = r.memoryformatOptional(2);
1082:   TORCH_CHECK(device.is_mtia(), "Invalid device, must be MTIA device");
1083:   torch::utils::device_lazy_init(at::kMTIA);
1084:   return THPVariable_Wrap(dispatch_to(self_, device, r.toBool(1), false, opt_memory_format));
1085:   END_HANDLE_TH_ERRORS
1086: }
1087: 
1088: static PyObject * THPVariable_xpu(PyObject* self, PyObject* args, PyObject* kwargs)
1089: {
1090:   HANDLE_TH_ERRORS
1091:   static PythonArgParser parser({
1092:     "xpu(Device? device=None, bool non_blocking=False, *, MemoryFormat? memory_format=None)",
1093:     "xpu(Device? device=None, bool async=False, *, MemoryFormat? memory_format=None)|deprecated"
1094:   });
1095:   auto& self_ = THPVariable_Unpack(self);
1096:   ParsedArgs<3> parsed_args;
1097:   auto r = parser.parse(self, args, kwargs, parsed_args);
1098: 
1099:   if (r.has_torch_function()) {
1100:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1101:   }
1102: 
1103:   auto device = r.isNone(0) ? at::Device(at::DeviceType::XPU) : r.device(0);
1104:   auto opt_memory_format = r.memoryformatOptional(2);
1105:   TORCH_CHECK(device.is_xpu(), "Invalid device, must be xpu device");
1106:   torch::utils::device_lazy_init(at::kXPU);
1107:   return THPVariable_Wrap(dispatch_to(self_, device, r.toBool(1), false, opt_memory_format));
1108:   END_HANDLE_TH_ERRORS
1109: }
1110: 
1111: static PyObject * THPVariable_ipu(PyObject* self, PyObject* args, PyObject* kwargs)
1112: {
1113:   HANDLE_TH_ERRORS
1114:   static PythonArgParser parser({
1115:     "ipu(Device? device=None, bool non_blocking=False, *, MemoryFormat? memory_format=None)",
1116:     "ipu(Device? device=None, bool async=False, *, MemoryFormat? memory_format=None)|deprecated"
1117:   });
1118:   auto& self_ = THPVariable_Unpack(self);
1119:   ParsedArgs<3> parsed_args;
1120:   auto r = parser.parse(self, args, kwargs, parsed_args);
1121: 
1122:   if (r.has_torch_function()) {
1123:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1124:   }
1125: 
1126:   auto device = r.isNone(0) ? at::Device(at::DeviceType::IPU) : r.device(0);
1127:   auto opt_memory_format = r.memoryformatOptional(2);
1128:   TORCH_CHECK(device.is_ipu(), "Invalid device, must be ipu device");
1129:   return THPVariable_Wrap(dispatch_to(self_, device, r.toBool(1), false, opt_memory_format));
1130:   END_HANDLE_TH_ERRORS
1131: }
1132: 
1133: static PyObject * THPVariable_to_type(PyObject* self, ScalarType scalarType, std::optional<c10::MemoryFormat> optional_memory_format) {
1134:   HANDLE_TH_ERRORS
1135:   auto& self_ = THPVariable_Unpack(self);
1136:   return THPVariable_Wrap(dispatch_to(self_, scalarType, false, false, optional_memory_format));
1137:   END_HANDLE_TH_ERRORS
1138: }
1139: 
1140: static PyObject * THPVariable_byte(PyObject* self, PyObject* args, PyObject* kwargs)  {
1141:   HANDLE_TH_ERRORS
1142:   static PythonArgParser parser({
1143:     "byte(*, MemoryFormat? memory_format=None)"
1144:   });
1145:   ParsedArgs<1> parsed_args;
1146:   auto r = parser.parse(self, args, kwargs, parsed_args);
1147: 
1148:   if(r.has_torch_function()){
1149:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1150:   }
1151: 
1152:   auto opt_memory_format = r.memoryformatOptional(0);
1153:   return THPVariable_to_type(self, ScalarType::Byte, opt_memory_format);
1154:   END_HANDLE_TH_ERRORS
1155: }
1156: 
1157: static PyObject * THPVariable_char(PyObject* self, PyObject* args, PyObject* kwargs)  {
1158:   HANDLE_TH_ERRORS
1159:   static PythonArgParser parser({
1160:     "char(*, MemoryFormat? memory_format=None)"
1161:   });
1162:   ParsedArgs<1> parsed_args;
1163:   auto r = parser.parse(self, args, kwargs, parsed_args);
1164: 
1165:   if(r.has_torch_function()){
1166:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1167:   }
1168: 
1169:   auto opt_memory_format = r.memoryformatOptional(0);
1170:   return THPVariable_to_type(self, ScalarType::Char, opt_memory_format);
1171:   END_HANDLE_TH_ERRORS
1172: }
1173: 
1174: static PyObject * THPVariable_double(PyObject* self, PyObject* args, PyObject* kwargs) {
1175:   HANDLE_TH_ERRORS
1176:   static PythonArgParser parser({
1177:     "double(*, MemoryFormat? memory_format=None)"
1178:   });
1179:   ParsedArgs<1> parsed_args;
1180:   auto r = parser.parse(self, args, kwargs, parsed_args);
1181: 
1182:   if(r.has_torch_function()){
1183:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1184:   }
1185: 
1186:   auto opt_memory_format = r.memoryformatOptional(0);
1187:   return THPVariable_to_type(self, ScalarType::Double, opt_memory_format);
1188:   END_HANDLE_TH_ERRORS
1189: }
1190: 
1191: static PyObject * THPVariable_float(PyObject* self, PyObject* args, PyObject* kwargs) {
1192:   HANDLE_TH_ERRORS
1193:   static PythonArgParser parser({
1194:     "float(*, MemoryFormat? memory_format=None)"
1195:   });
1196:   ParsedArgs<1> parsed_args;
1197:   auto r = parser.parse(self, args, kwargs, parsed_args);
1198: 
1199:   if(r.has_torch_function()){
1200:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `TORCH_CHECK`, `device_lazy_init`, `THPVariable_Wrap`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `TORCH_CHECK`, `device_lazy_init`, `THPVariable_Wrap` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1201-1320

```cpp
1201:   }
1202: 
1203:   auto opt_memory_format = r.memoryformatOptional(0);
1204:   return THPVariable_to_type(self, ScalarType::Float, opt_memory_format);
1205:   END_HANDLE_TH_ERRORS
1206: }
1207: 
1208: static PyObject * THPVariable_cdouble(PyObject* self, PyObject* args, PyObject* kwargs) {
1209:   HANDLE_TH_ERRORS
1210:   static PythonArgParser parser({
1211:     "cdouble(*, MemoryFormat? memory_format=None)"
1212:   });
1213:   ParsedArgs<1> parsed_args;
1214:   auto r = parser.parse(self, args, kwargs, parsed_args);
1215: 
1216:   if(r.has_torch_function()){
1217:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1218:   }
1219: 
1220:   auto opt_memory_format = r.memoryformatOptional(0);
1221:   return THPVariable_to_type(self, ScalarType::ComplexDouble, opt_memory_format);
1222:   END_HANDLE_TH_ERRORS
1223: }
1224: 
1225: static PyObject * THPVariable_cfloat(PyObject* self, PyObject* args, PyObject* kwargs) {
1226:   HANDLE_TH_ERRORS
1227:   static PythonArgParser parser({
1228:     "cfloat(*, MemoryFormat? memory_format=None)"
1229:   });
1230:   ParsedArgs<1> parsed_args;
1231:   auto r = parser.parse(self, args, kwargs, parsed_args);
1232: 
1233:   if(r.has_torch_function()){
1234:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1235:   }
1236: 
1237:   auto opt_memory_format = r.memoryformatOptional(0);
1238:   return THPVariable_to_type(self, ScalarType::ComplexFloat, opt_memory_format);
1239:   END_HANDLE_TH_ERRORS
1240: }
1241: 
1242: static PyObject * THPVariable_half(PyObject* self, PyObject* args, PyObject* kwargs) {
1243:   HANDLE_TH_ERRORS
1244:   static PythonArgParser parser({
1245:     "half(*, MemoryFormat? memory_format=None)"
1246:   });
1247:   ParsedArgs<1> parsed_args;
1248:   auto r = parser.parse(self, args, kwargs, parsed_args);
1249: 
1250:   if(r.has_torch_function()){
1251:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1252:   }
1253: 
1254:   auto opt_memory_format = r.memoryformatOptional(0);
1255:   return THPVariable_to_type(self, ScalarType::Half, opt_memory_format);
1256:   END_HANDLE_TH_ERRORS
1257: }
1258: 
1259: static PyObject * THPVariable_int(PyObject* self, PyObject* args, PyObject* kwargs) {
1260:   HANDLE_TH_ERRORS
1261:   static PythonArgParser parser({
1262:     "int(*, MemoryFormat? memory_format=None)"
1263:   });
1264:   ParsedArgs<1> parsed_args;
1265:   auto r = parser.parse(self, args, kwargs, parsed_args);
1266: 
1267:   if(r.has_torch_function()){
1268:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1269:   }
1270: 
1271:   auto opt_memory_format = r.memoryformatOptional(0);
1272:   return THPVariable_to_type(self, ScalarType::Int, opt_memory_format);
1273:   END_HANDLE_TH_ERRORS
1274: }
1275: 
1276: static PyObject * THPVariable_long(PyObject* self, PyObject* args, PyObject* kwargs) {
1277:   HANDLE_TH_ERRORS
1278:   static PythonArgParser parser({
1279:     "long(*, MemoryFormat? memory_format=None)"
1280:   });
1281:   ParsedArgs<1> parsed_args;
1282:   auto r = parser.parse(self, args, kwargs, parsed_args);
1283: 
1284:   if(r.has_torch_function()){
1285:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1286:   }
1287: 
1288:   auto opt_memory_format = r.memoryformatOptional(0);
1289:   return THPVariable_to_type(self, ScalarType::Long, opt_memory_format);
1290:   END_HANDLE_TH_ERRORS
1291: }
1292: 
1293: static PyObject * THPVariable_short(PyObject* self, PyObject* args, PyObject* kwargs) {
1294:   HANDLE_TH_ERRORS
1295:   static PythonArgParser parser({
1296:     "short(*, MemoryFormat? memory_format=None)"
1297:   });
1298:   ParsedArgs<1> parsed_args;
1299:   auto r = parser.parse(self, args, kwargs, parsed_args);
1300: 
1301:   if(r.has_torch_function()){
1302:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1303:   }
1304: 
1305:   auto opt_memory_format = r.memoryformatOptional(0);
1306:   return THPVariable_to_type(self, ScalarType::Short, opt_memory_format);
1307:   END_HANDLE_TH_ERRORS
1308: }
1309: 
1310: static PyObject * THPVariable_bool(PyObject* self, PyObject* args, PyObject* kwargs) {
1311:   HANDLE_TH_ERRORS
1312:   static PythonArgParser parser({
1313:     "bool(*, MemoryFormat? memory_format=None)"
1314:   });
1315:   ParsedArgs<1> parsed_args;
1316:   auto r = parser.parse(self, args, kwargs, parsed_args);
1317: 
1318:   if(r.has_torch_function()){
1319:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1320:   }
```

- EN: The main execution path in this span is carried by `THPVariable_to_type`, `THPVariable_cdouble`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_to_type`, `THPVariable_cdouble`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1321-1440

```cpp
1321: 
1322:   auto opt_memory_format = r.memoryformatOptional(0);
1323:   return THPVariable_to_type(self, ScalarType::Bool, opt_memory_format);
1324:   END_HANDLE_TH_ERRORS
1325: }
1326: 
1327: static PyObject * THPVariable_bfloat16(PyObject* self, PyObject* args, PyObject* kwargs) {
1328:   HANDLE_TH_ERRORS
1329:   static PythonArgParser parser({
1330:     "bfloat16(*, MemoryFormat? memory_format=None)"
1331:   });
1332:   ParsedArgs<1> parsed_args;
1333:   auto r = parser.parse(self, args, kwargs, parsed_args);
1334: 
1335:   if(r.has_torch_function()){
1336:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1337:   }
1338: 
1339:   auto opt_memory_format = r.memoryformatOptional(0);
1340:   return THPVariable_to_type(self, ScalarType::BFloat16, opt_memory_format);
1341:   END_HANDLE_TH_ERRORS
1342: }
1343: 
1344: static PyObject * THPVariable_element_size(PyObject* self, PyObject* args)
1345: {
1346:   HANDLE_TH_ERRORS
1347:   if (has_torch_function(self)) {
1348:     return handle_torch_function(self, "element_size", args);
1349:   }
1350:   auto& self_ = THPVariable_Unpack(self);
1351:   return THPUtils_packInt64(self_.element_size());
1352:   END_HANDLE_TH_ERRORS
1353: }
1354: 
1355: // implemented on the python object bc PyObjects not declarable in native_functions.yaml
1356: // See: ATen/native/README.md for more context
1357: static PyObject * THPVariable_numpy(PyObject* self, PyObject* args, PyObject* kwargs)
1358: {
1359:   HANDLE_TH_ERRORS
1360:   static PythonArgParser parser({
1361:     "numpy(*, bool force=False)"
1362:   });
1363:   auto& self_ = THPVariable_Unpack(self);
1364:   ParsedArgs<1> parsed_args;
1365:   auto r = parser.parse(self, args, kwargs, parsed_args);
1366: 
1367:   if (r.has_torch_function()) {
1368:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1369:   }
1370: 
1371:   jit::tracer::warn("Converting a tensor to a NumPy array", jit::tracer::WARN_PYTHON_DATAFLOW);
1372:   return torch::utils::tensor_to_numpy(self_, r.toBool(0));
1373:   END_HANDLE_TH_ERRORS
1374: }
1375: 
1376: static PyObject * THPVariable_requires_grad_(PyObject* self, PyObject* args, PyObject* kwargs)
1377: {
1378:   HANDLE_TH_ERRORS
1379:   static PythonArgParser parser({
1380:     "requires_grad_(bool requires_grad=True)",
1381:   });
1382:   auto& self_ = THPVariable_Unpack(self);
1383:   ParsedArgs<1> parsed_args;
1384:   auto r = parser.parse(self, args, kwargs, parsed_args);
1385: 
1386:   if(r.has_torch_function()){
1387:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1388:   }
1389: 
1390:   // temporary hack to improve functorch UX.
1391:   const auto& functorch_tls = at::functorch::functorchTLSAccessor();
1392:   if (functorch_tls) {
1393:     functorch_tls->checkSupportsInplaceRequiresGrad();
1394:   }
1395: 
1396:   auto requires_grad = r.toBool(0);
1397:   // should we throw if requires_grad is true?  var.requires_grad = True throws here
1398:   // but it's nice to let this be a no-op.
1399:   if (!self_.is_leaf() && !requires_grad) {
1400:     throw std::runtime_error(autograd::utils::requires_grad_leaf_error(requires_grad));
1401:   }
1402:   if (requires_grad && ! isDifferentiableType(at::typeMetaToScalarType(self_.dtype()))) {
1403:     throw std::runtime_error("only Tensors of floating point dtype can require gradients");
1404:   }
1405:   self_.set_requires_grad(requires_grad);
1406:   return THPVariable_Wrap(self_);
1407:   END_HANDLE_TH_ERRORS
1408: }
1409: 
1410: static inline bool dispatch_is_contiguous(const Tensor & self, MemoryFormat memory_format) {
1411:   return self.is_contiguous(memory_format);
1412: }
1413: 
1414: // implemented on the python object to avoid dispatch overhead
1415: static PyObject * THPVariable_is_contiguous(PyObject* self_, PyObject* args, PyObject* kwargs)
1416: {
1417:   HANDLE_TH_ERRORS
1418:   static PythonArgParser parser({
1419:     "is_contiguous(*, MemoryFormat memory_format=contiguous_format)",
1420:   });
1421:   ParsedArgs<1> parsed_args;
1422:   auto r = parser.parse(self_, args, kwargs, parsed_args);
1423: 
1424:   if(r.has_torch_function()){
1425:     return handle_torch_function(r, self_, args, kwargs, reinterpret_cast<PyObject*>(Py_TYPE(self_)), "torch.Tensor");
1426:   }
1427: 
1428:   auto memory_format = r.memoryformat(0);
1429:   auto& self = THPVariable_Unpack(self_);
1430:   return wrap(dispatch_is_contiguous(self, memory_format));
1431:   END_HANDLE_TH_ERRORS
1432: }
1433: 
1434: // implemented on the python object to avoid dispatch overhead
1435: static PyObject * THPVariable_item(PyObject* self, PyObject* args)
1436: {
1437:   HANDLE_TH_ERRORS
1438:   if (has_torch_function(self)) {
1439:     return handle_torch_function(self, "item", args);
1440:   }
```

- EN: The main execution path in this span is carried by `THPVariable_to_type`, `THPVariable_bfloat16`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_to_type`, `THPVariable_bfloat16`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 1441-1560

```cpp
1441:   jit::tracer::warn("Converting a tensor to a Python number", jit::tracer::WARN_PYTHON_DATAFLOW);
1442:   auto& self_ = THPVariable_Unpack(self);
1443:   auto dispatch_item_ = [](const Tensor& self) -> at::Scalar {
1444:     pybind11::gil_scoped_release no_gil;
1445:     return self.item();
1446:   };
1447:   return py::cast(dispatch_item_(self_)).release().ptr();
1448:   END_HANDLE_TH_ERRORS
1449: }
1450: 
1451: // implemented on the python object bc no support for first class functions in native_functions.yaml
1452: // See: ATen/native/README.md for more context
1453: static PyObject * THPVariable_map_(PyObject* self, PyObject* args, PyObject* kwargs)
1454: {
1455:   HANDLE_TH_ERRORS
1456:   static PythonArgParser parser({ "map_(Tensor other, PyObject* callable)" });
1457:   auto& self_ = THPVariable_Unpack(self);
1458:   ParsedArgs<2> parsed_args;
1459:   auto r = parser.parse(self, args, kwargs, parsed_args);
1460: 
1461:   if(r.has_torch_function()){
1462:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1463:   }
1464: 
1465:   Variable other = r.tensor(0);
1466:   if (self_.requires_grad() || other.requires_grad()) {
1467:     throw std::runtime_error(
1468:         "Can't call map_() on Variable that requires grad. Use "
1469:         "var.detach().map_() instead.");
1470:   }
1471:   TORCH_CHECK(
1472:       !self_.unsafeGetTensorImpl()->is_python_dispatch() && !other.unsafeGetTensorImpl()->is_python_dispatch(),
1473:       ".map_ is not supported for tensor subclasses.");
1474: 
1475:   return THPVariable_Wrap(torch::utils::map_(self_, other, r.pyobject(1)));
1476:   END_HANDLE_TH_ERRORS
1477: }
1478: 
1479: // implemented on the python object bc no support for first class functions in native_functions.yaml
1480: // See: ATen/native/README.md for more context
1481: static PyObject * THPVariable_map2_(PyObject* self, PyObject* args, PyObject* kwargs)
1482: {
1483:   HANDLE_TH_ERRORS
1484:   static PythonArgParser parser({ "map2_(Tensor x, Tensor y, PyObject* callable)" });
1485:   auto& self_ = THPVariable_Unpack(self);
1486:   ParsedArgs<3> parsed_args;
1487:   auto r = parser.parse(self, args, kwargs, parsed_args);
1488: 
1489:   if(r.has_torch_function()){
1490:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1491:   }
1492: 
1493:   Variable x = r.tensor(0);
1494:   Variable y = r.tensor(1);
1495:   if (self_.requires_grad() || x.requires_grad() || y.requires_grad()) {
1496:     throw std::runtime_error(
1497:         "Can't call map2_() on Variable that requires grad. Use "
1498:         "var.detach().map2_() instead.");
1499:   }
1500:   TORCH_CHECK(
1501:       !x.unsafeGetTensorImpl()->is_python_dispatch() && !y.unsafeGetTensorImpl()->is_python_dispatch(),
1502:       ".map2_ is not supported for tensor subclasses.");
1503:   return THPVariable_Wrap(torch::utils::map2_(self_, x, y, r.pyobject(2)));
1504:   END_HANDLE_TH_ERRORS
1505: }
1506: 
1507: static PyObject * THPVariable_new(PyObject* self, PyObject* args, PyObject* kwargs)
1508: {
1509:   HANDLE_TH_ERRORS
1510:   if (has_torch_function(self)) {
1511:     return handle_torch_function(self, "new", args, kwargs);
1512:   }
1513:   auto& self_ = THPVariable_Unpack(self);
1514:   OptionalDeviceGuard device_guard(device_of(self_));
1515:   return THPVariable_Wrap(torch::utils::legacy_tensor_new(legacyExtractDispatchKey(self_), self_.scalar_type(), args, kwargs));
1516:   END_HANDLE_TH_ERRORS
1517: }
1518: 
1519: static PyObject * THPVariable_new_tensor(PyObject* self, PyObject* args, PyObject* kwargs)
1520: {
1521:   HANDLE_TH_ERRORS
1522:   if (has_torch_function(self)) {
1523:     return handle_torch_function(self, "new_tensor", args, kwargs);
1524:   }
1525:   auto& self_ = THPVariable_Unpack(self);
1526:   OptionalDeviceGuard device_guard(device_of(self_));
1527:   return THPVariable_Wrap(torch::utils::new_tensor(legacyExtractDispatchKey(self_), self_.scalar_type(), args, kwargs));
1528:   END_HANDLE_TH_ERRORS
1529: }
1530: 
1531: static PyObject * THPVariable_storage(PyObject* self, PyObject* arg)
1532: {
1533:   HANDLE_TH_ERRORS
1534:   if (has_torch_function(self)) {
1535:     return handle_torch_function(self, "untyped_storage");
1536:   }
1537:   auto& self_ = THPVariable_Unpack(self);
1538:   return createPyObject(self_.storage());
1539:   END_HANDLE_TH_ERRORS
1540: }
1541: 
1542: static PyObject * THPVariable_to(PyObject* self, PyObject* args, PyObject* kwargs)
1543: {
1544:   HANDLE_TH_ERRORS
1545:   static PythonArgParser parser({
1546:     "to(Device device=None, ScalarType dtype=None, bool non_blocking=False, bool copy=False, *, MemoryFormat? memory_format=None)",
1547:     "to(ScalarType dtype, bool non_blocking=False, bool copy=False, *, MemoryFormat? memory_format=None)",
1548:     "to(Tensor tensor, bool non_blocking=False, bool copy=False, *, MemoryFormat? memory_format=None)",
1549:   });
1550:   ParsedArgs<5> parsed_args;
1551:   auto r = parser.parse(self, args, kwargs, parsed_args);
1552:   if (r.has_torch_function()) {
1553:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1554:   }
1555:   auto parsed = parse_to_conversion(r, /*allow_copy*/ true);
1556:   auto& device = std::get<0>(parsed);
1557:   auto& scalarType = std::get<1>(parsed);
1558:   auto non_blocking = std::get<2>(parsed);
1559:   auto copy = std::get<3>(parsed);
1560:   auto opt_memory_format = std::get<4>(parsed);
```

- EN: The main execution path in this span is carried by `warn`, `THPVariable_Unpack`, `cast`. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here.
- CN: 这一段的主要执行路径由 `warn`, `THPVariable_Unpack`, `cast` 等函数/方法承载。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。
### Lines 1561-1680

```cpp
1561:   auto& self_ = THPVariable_Unpack(self);
1562:   torch::utils::maybe_initialize_device(device);
1563:   if (!device && !scalarType && !copy && !opt_memory_format.has_value()) {
1564:     Py_INCREF(self);
1565:     return self;
1566:   } else if (!device && !scalarType) {
1567:     return THPVariable_Wrap(
1568:         dispatch_to(self_, non_blocking, copy, opt_memory_format));
1569:   } else if (!device) {
1570:     return THPVariable_Wrap(dispatch_to(self_, *scalarType, non_blocking, copy, opt_memory_format));
1571:   } else if (!scalarType) {
1572:     return THPVariable_Wrap(dispatch_to(self_, *device, non_blocking, copy, opt_memory_format));
1573:   } else {
1574:     return THPVariable_Wrap(dispatch_to(self_, *device, *scalarType, non_blocking, copy, opt_memory_format));
1575:   }
1576:   Py_RETURN_NONE;
1577:   END_HANDLE_TH_ERRORS
1578: }
1579: 
1580: // implemented on the python object b/c arbitrarily nested list not declarable in native_functions.yaml
1581: // See: ATen/native/README.md for more context
1582: static PyObject * THPVariable_tolist(PyObject* self, PyObject* args)
1583: {
1584:   HANDLE_TH_ERRORS
1585:   if (has_torch_function(self)) {
1586:     return handle_torch_function(self, "tolist", args);
1587:   }
1588:   jit::tracer::warn("Converting a tensor to a Python list", jit::tracer::WARN_PYTHON_DATAFLOW);
1589:   auto self_ = THPVariable_Unpack(self);
1590:   return torch::utils::tensor_to_list(self_);
1591:   END_HANDLE_TH_ERRORS
1592: }
1593: 
1594: static PyObject * THPVariable_type(PyObject* self, PyObject* args, PyObject* kwargs)
1595: {
1596:   HANDLE_TH_ERRORS
1597:   static PythonArgParser parser({
1598:     "type(PyObject* dtype=None, bool non_blocking=False, *, MemoryFormat? memory_format=None)",
1599:     "type(PyObject* dtype=None, bool async=False, *, MemoryFormat? memory_format=None)|deprecated"
1600:   });
1601:   auto& self_ = THPVariable_Unpack(self);
1602:   ParsedArgs<3> parsed_args;
1603:   auto r = parser.parse(self, args, kwargs, parsed_args);
1604: 
1605:   if(r.has_torch_function()){
1606:     return handle_torch_function(r, self, args, kwargs, THPVariableClass, "torch.Tensor");
1607:   }
1608: 
1609:   if (r.isNone(0)) {
1610:     return THPUtils_packString(torch::utils::options_to_string(self_.options()));
1611:   }
1612:   auto obj = r.pyobject(0);
1613:   auto opt_memory_format = r.memoryformatOptional(2);
1614:   std::string type_name;
1615:   bool is_dtype = false;
1616:   if (PyType_Check(obj)) {
1617:     if (obj == THPVariableClass) {
1618:       type_name = "torch.Tensor";
1619:     } else {
1620:       type_name = ((PyTypeObject*)obj)->tp_name;
1621:     }
1622:   } else if (THPUtils_checkString(obj)) {
1623:     type_name = THPUtils_unpackString(obj);
1624:   } else if (THPDtype_Check(obj)) {
1625:     is_dtype = true;
1626:   } else {
1627:     throw TypeError("dtype must be a type, str, or dtype object");
1628:   }
1629:   Device device = self_.device();
1630:   if (is_dtype) {
1631:     auto scalar_type = r.scalartype(0);
1632:     return THPVariable_Wrap(dispatch_to(self_, scalar_type, /*non_blocking=*/ r.toBool(1), /*copy=*/ false, opt_memory_format));
1633:   }
1634:   at::TensorOptions options = torch::utils::options_from_string(type_name);
1635:   auto scalar_type = at::typeMetaToScalarType(options.dtype());
1636:   auto device_type = options.device().type();
1637:   if (device_type != device.type()) {
1638:     device = at::Device(device_type);
1639:   }
1640:   torch::utils::maybe_initialize_device(device);
1641:   return THPVariable_Wrap(dispatch_to(self_, device, scalar_type, /*non_blocking=*/ r.toBool(1), /*copy=*/ false, opt_memory_format));
1642:   END_HANDLE_TH_ERRORS
1643: }
1644: 
1645: // generated methods start here
1646: 
1647: \
1648: // __and__
1649: static PyObject * THPVariable___and__(PyObject* self_, PyObject* args, PyObject* kwargs)
1650: {
1651:   HANDLE_TH_ERRORS
1652:   const Tensor& self = THPVariable_Unpack(self_);
1653:   static PythonArgParser parser({
1654:     "__and__(Tensor other)",
1655:     "__and__(Scalar other)",
1656:   }, /*traceable=*/true);
1657: 
1658:   ParsedArgs<1> parsed_args;
1659:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1660:   if(_r.has_torch_function()) {
1661:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1662:   }
1663:   switch (_r.idx) {
1664:     case 0: {
1665:       // aten::__and__.Tensor(Tensor self, Tensor other) -> Tensor
1666: 
1667:       auto dispatch___and__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1668:         pybind11::gil_scoped_release no_gil;
1669:         return self.__and__(other);
1670:       };
1671:       return wrap(dispatch___and__(self, _r.tensor(0)));
1672:     }
1673:     case 1: {
1674:       // aten::__and__.Scalar(Tensor self, Scalar other) -> Tensor
1675: 
1676:       auto dispatch___and__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1677:         pybind11::gil_scoped_release no_gil;
1678:         return self.__and__(other);
1679:       };
1680:       return wrap(dispatch___and__(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `maybe_initialize_device`, `Py_INCREF`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `maybe_initialize_device`, `Py_INCREF` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1681-1800

```cpp
1681:     }
1682:   }
1683:   Py_RETURN_NONE;
1684:   END_HANDLE_TH_ERRORS
1685: }
1686: 
1687: \
1688: // __iand__
1689: static PyObject * THPVariable___iand__(PyObject* self_, PyObject* args, PyObject* kwargs)
1690: {
1691:   HANDLE_TH_ERRORS
1692:   const Tensor& self = THPVariable_Unpack(self_);
1693:   static PythonArgParser parser({
1694:     "__iand__(Tensor other)",
1695:     "__iand__(Scalar other)",
1696:   }, /*traceable=*/true);
1697: 
1698:   ParsedArgs<1> parsed_args;
1699:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1700:   if(_r.has_torch_function()) {
1701:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1702:   }
1703:   switch (_r.idx) {
1704:     case 0: {
1705:       // aten::__iand__.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
1706: 
1707:       auto dispatch___iand__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1708:         pybind11::gil_scoped_release no_gil;
1709:         return self.__iand__(other);
1710:       };
1711:       return wrap(dispatch___iand__(self, _r.tensor(0)));
1712:     }
1713:     case 1: {
1714:       // aten::__iand__.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
1715: 
1716:       auto dispatch___iand__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1717:         pybind11::gil_scoped_release no_gil;
1718:         return self.__iand__(other);
1719:       };
1720:       return wrap(dispatch___iand__(self, _r.scalar(0)));
1721:     }
1722:   }
1723:   Py_RETURN_NONE;
1724:   END_HANDLE_TH_ERRORS
1725: }
1726: 
1727: \
1728: // __ilshift__
1729: static PyObject * THPVariable___ilshift__(PyObject* self_, PyObject* args, PyObject* kwargs)
1730: {
1731:   HANDLE_TH_ERRORS
1732:   const Tensor& self = THPVariable_Unpack(self_);
1733:   static PythonArgParser parser({
1734:     "__ilshift__(Tensor other)",
1735:     "__ilshift__(Scalar other)",
1736:   }, /*traceable=*/true);
1737: 
1738:   ParsedArgs<1> parsed_args;
1739:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1740:   if(_r.has_torch_function()) {
1741:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1742:   }
1743:   switch (_r.idx) {
1744:     case 0: {
1745:       // aten::__ilshift__.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
1746: 
1747:       auto dispatch___ilshift__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1748:         pybind11::gil_scoped_release no_gil;
1749:         return self.__ilshift__(other);
1750:       };
1751:       return wrap(dispatch___ilshift__(self, _r.tensor(0)));
1752:     }
1753:     case 1: {
1754:       // aten::__ilshift__.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
1755: 
1756:       auto dispatch___ilshift__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1757:         pybind11::gil_scoped_release no_gil;
1758:         return self.__ilshift__(other);
1759:       };
1760:       return wrap(dispatch___ilshift__(self, _r.scalar(0)));
1761:     }
1762:   }
1763:   Py_RETURN_NONE;
1764:   END_HANDLE_TH_ERRORS
1765: }
1766: 
1767: \
1768: // __ior__
1769: static PyObject * THPVariable___ior__(PyObject* self_, PyObject* args, PyObject* kwargs)
1770: {
1771:   HANDLE_TH_ERRORS
1772:   const Tensor& self = THPVariable_Unpack(self_);
1773:   static PythonArgParser parser({
1774:     "__ior__(Tensor other)",
1775:     "__ior__(Scalar other)",
1776:   }, /*traceable=*/true);
1777: 
1778:   ParsedArgs<1> parsed_args;
1779:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1780:   if(_r.has_torch_function()) {
1781:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1782:   }
1783:   switch (_r.idx) {
1784:     case 0: {
1785:       // aten::__ior__.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
1786: 
1787:       auto dispatch___ior__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1788:         pybind11::gil_scoped_release no_gil;
1789:         return self.__ior__(other);
1790:       };
1791:       return wrap(dispatch___ior__(self, _r.tensor(0)));
1792:     }
1793:     case 1: {
1794:       // aten::__ior__.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
1795: 
1796:       auto dispatch___ior__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1797:         pybind11::gil_scoped_release no_gil;
1798:         return self.__ior__(other);
1799:       };
1800:       return wrap(dispatch___ior__(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `THPVariable___iand__`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable___iand__`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1801-1920

```cpp
1801:     }
1802:   }
1803:   Py_RETURN_NONE;
1804:   END_HANDLE_TH_ERRORS
1805: }
1806: 
1807: \
1808: // __irshift__
1809: static PyObject * THPVariable___irshift__(PyObject* self_, PyObject* args, PyObject* kwargs)
1810: {
1811:   HANDLE_TH_ERRORS
1812:   const Tensor& self = THPVariable_Unpack(self_);
1813:   static PythonArgParser parser({
1814:     "__irshift__(Tensor other)",
1815:     "__irshift__(Scalar other)",
1816:   }, /*traceable=*/true);
1817: 
1818:   ParsedArgs<1> parsed_args;
1819:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1820:   if(_r.has_torch_function()) {
1821:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1822:   }
1823:   switch (_r.idx) {
1824:     case 0: {
1825:       // aten::__irshift__.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
1826: 
1827:       auto dispatch___irshift__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1828:         pybind11::gil_scoped_release no_gil;
1829:         return self.__irshift__(other);
1830:       };
1831:       return wrap(dispatch___irshift__(self, _r.tensor(0)));
1832:     }
1833:     case 1: {
1834:       // aten::__irshift__.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
1835: 
1836:       auto dispatch___irshift__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1837:         pybind11::gil_scoped_release no_gil;
1838:         return self.__irshift__(other);
1839:       };
1840:       return wrap(dispatch___irshift__(self, _r.scalar(0)));
1841:     }
1842:   }
1843:   Py_RETURN_NONE;
1844:   END_HANDLE_TH_ERRORS
1845: }
1846: 
1847: \
1848: // __ixor__
1849: static PyObject * THPVariable___ixor__(PyObject* self_, PyObject* args, PyObject* kwargs)
1850: {
1851:   HANDLE_TH_ERRORS
1852:   const Tensor& self = THPVariable_Unpack(self_);
1853:   static PythonArgParser parser({
1854:     "__ixor__(Tensor other)",
1855:     "__ixor__(Scalar other)",
1856:   }, /*traceable=*/true);
1857: 
1858:   ParsedArgs<1> parsed_args;
1859:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1860:   if(_r.has_torch_function()) {
1861:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1862:   }
1863:   switch (_r.idx) {
1864:     case 0: {
1865:       // aten::__ixor__.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
1866: 
1867:       auto dispatch___ixor__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1868:         pybind11::gil_scoped_release no_gil;
1869:         return self.__ixor__(other);
1870:       };
1871:       return wrap(dispatch___ixor__(self, _r.tensor(0)));
1872:     }
1873:     case 1: {
1874:       // aten::__ixor__.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
1875: 
1876:       auto dispatch___ixor__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1877:         pybind11::gil_scoped_release no_gil;
1878:         return self.__ixor__(other);
1879:       };
1880:       return wrap(dispatch___ixor__(self, _r.scalar(0)));
1881:     }
1882:   }
1883:   Py_RETURN_NONE;
1884:   END_HANDLE_TH_ERRORS
1885: }
1886: 
1887: \
1888: // __lshift__
1889: static PyObject * THPVariable___lshift__(PyObject* self_, PyObject* args, PyObject* kwargs)
1890: {
1891:   HANDLE_TH_ERRORS
1892:   const Tensor& self = THPVariable_Unpack(self_);
1893:   static PythonArgParser parser({
1894:     "__lshift__(Tensor other)",
1895:     "__lshift__(Scalar other)",
1896:   }, /*traceable=*/true);
1897: 
1898:   ParsedArgs<1> parsed_args;
1899:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1900:   if(_r.has_torch_function()) {
1901:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1902:   }
1903:   switch (_r.idx) {
1904:     case 0: {
1905:       // aten::__lshift__.Tensor(Tensor self, Tensor other) -> Tensor
1906: 
1907:       auto dispatch___lshift__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1908:         pybind11::gil_scoped_release no_gil;
1909:         return self.__lshift__(other);
1910:       };
1911:       return wrap(dispatch___lshift__(self, _r.tensor(0)));
1912:     }
1913:     case 1: {
1914:       // aten::__lshift__.Scalar(Tensor self, Scalar other) -> Tensor
1915: 
1916:       auto dispatch___lshift__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1917:         pybind11::gil_scoped_release no_gil;
1918:         return self.__lshift__(other);
1919:       };
1920:       return wrap(dispatch___lshift__(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `THPVariable___irshift__`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable___irshift__`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 1921-2040

```cpp
1921:     }
1922:   }
1923:   Py_RETURN_NONE;
1924:   END_HANDLE_TH_ERRORS
1925: }
1926: 
1927: \
1928: // __or__
1929: static PyObject * THPVariable___or__(PyObject* self_, PyObject* args, PyObject* kwargs)
1930: {
1931:   HANDLE_TH_ERRORS
1932:   const Tensor& self = THPVariable_Unpack(self_);
1933:   static PythonArgParser parser({
1934:     "__or__(Tensor other)",
1935:     "__or__(Scalar other)",
1936:   }, /*traceable=*/true);
1937: 
1938:   ParsedArgs<1> parsed_args;
1939:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1940:   if(_r.has_torch_function()) {
1941:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1942:   }
1943:   switch (_r.idx) {
1944:     case 0: {
1945:       // aten::__or__.Tensor(Tensor self, Tensor other) -> Tensor
1946: 
1947:       auto dispatch___or__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1948:         pybind11::gil_scoped_release no_gil;
1949:         return self.__or__(other);
1950:       };
1951:       return wrap(dispatch___or__(self, _r.tensor(0)));
1952:     }
1953:     case 1: {
1954:       // aten::__or__.Scalar(Tensor self, Scalar other) -> Tensor
1955: 
1956:       auto dispatch___or__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1957:         pybind11::gil_scoped_release no_gil;
1958:         return self.__or__(other);
1959:       };
1960:       return wrap(dispatch___or__(self, _r.scalar(0)));
1961:     }
1962:   }
1963:   Py_RETURN_NONE;
1964:   END_HANDLE_TH_ERRORS
1965: }
1966: 
1967: \
1968: // __rshift__
1969: static PyObject * THPVariable___rshift__(PyObject* self_, PyObject* args, PyObject* kwargs)
1970: {
1971:   HANDLE_TH_ERRORS
1972:   const Tensor& self = THPVariable_Unpack(self_);
1973:   static PythonArgParser parser({
1974:     "__rshift__(Tensor other)",
1975:     "__rshift__(Scalar other)",
1976:   }, /*traceable=*/true);
1977: 
1978:   ParsedArgs<1> parsed_args;
1979:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
1980:   if(_r.has_torch_function()) {
1981:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
1982:   }
1983:   switch (_r.idx) {
1984:     case 0: {
1985:       // aten::__rshift__.Tensor(Tensor self, Tensor other) -> Tensor
1986: 
1987:       auto dispatch___rshift__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
1988:         pybind11::gil_scoped_release no_gil;
1989:         return self.__rshift__(other);
1990:       };
1991:       return wrap(dispatch___rshift__(self, _r.tensor(0)));
1992:     }
1993:     case 1: {
1994:       // aten::__rshift__.Scalar(Tensor self, Scalar other) -> Tensor
1995: 
1996:       auto dispatch___rshift__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
1997:         pybind11::gil_scoped_release no_gil;
1998:         return self.__rshift__(other);
1999:       };
2000:       return wrap(dispatch___rshift__(self, _r.scalar(0)));
2001:     }
2002:   }
2003:   Py_RETURN_NONE;
2004:   END_HANDLE_TH_ERRORS
2005: }
2006: 
2007: \
2008: // __xor__
2009: static PyObject * THPVariable___xor__(PyObject* self_, PyObject* args, PyObject* kwargs)
2010: {
2011:   HANDLE_TH_ERRORS
2012:   const Tensor& self = THPVariable_Unpack(self_);
2013:   static PythonArgParser parser({
2014:     "__xor__(Tensor other)",
2015:     "__xor__(Scalar other)",
2016:   }, /*traceable=*/true);
2017: 
2018:   ParsedArgs<1> parsed_args;
2019:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2020:   if(_r.has_torch_function()) {
2021:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2022:   }
2023:   switch (_r.idx) {
2024:     case 0: {
2025:       // aten::__xor__.Tensor(Tensor self, Tensor other) -> Tensor
2026: 
2027:       auto dispatch___xor__ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
2028:         pybind11::gil_scoped_release no_gil;
2029:         return self.__xor__(other);
2030:       };
2031:       return wrap(dispatch___xor__(self, _r.tensor(0)));
2032:     }
2033:     case 1: {
2034:       // aten::__xor__.Scalar(Tensor self, Scalar other) -> Tensor
2035: 
2036:       auto dispatch___xor__ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
2037:         pybind11::gil_scoped_release no_gil;
2038:         return self.__xor__(other);
2039:       };
2040:       return wrap(dispatch___xor__(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `THPVariable___or__`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable___or__`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2041-2160

```cpp
2041:     }
2042:   }
2043:   Py_RETURN_NONE;
2044:   END_HANDLE_TH_ERRORS
2045: }
2046: 
2047: // _addmm_activation
2048: static PyObject * THPVariable__addmm_activation(PyObject* self_, PyObject* args, PyObject* kwargs)
2049: {
2050:   HANDLE_TH_ERRORS
2051:   const Tensor& self = THPVariable_Unpack(self_);
2052:   static PythonArgParser parser({
2053:     "_addmm_activation(Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1, bool use_gelu=False)",
2054:   }, /*traceable=*/true);
2055: 
2056:   ParsedArgs<5> parsed_args;
2057:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2058:   if(_r.has_torch_function()) {
2059:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2060:   }
2061:   // aten::_addmm_activation(Tensor self, Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1, bool use_gelu=False) -> Tensor
2062: 
2063:   auto dispatch__addmm_activation = [](const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2, const at::Scalar & beta, const at::Scalar & alpha, bool use_gelu) -> at::Tensor {
2064:     pybind11::gil_scoped_release no_gil;
2065:     return self._addmm_activation(mat1, mat2, beta, alpha, use_gelu);
2066:   };
2067:   return wrap(dispatch__addmm_activation(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3), _r.toBool(4)));
2068:   Py_RETURN_NONE;
2069:   END_HANDLE_TH_ERRORS
2070: }
2071: 
2072: // _autocast_to_full_precision
2073: static PyObject * THPVariable__autocast_to_full_precision(PyObject* self_, PyObject* args, PyObject* kwargs)
2074: {
2075:   HANDLE_TH_ERRORS
2076:   const Tensor& self = THPVariable_Unpack(self_);
2077:   static PythonArgParser parser({
2078:     "_autocast_to_full_precision(bool cuda_enabled, bool cpu_enabled)",
2079:   }, /*traceable=*/true);
2080: 
2081:   ParsedArgs<2> parsed_args;
2082:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2083:   if(_r.has_torch_function()) {
2084:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2085:   }
2086:   // aten::_autocast_to_full_precision(Tensor(a) self, bool cuda_enabled, bool cpu_enabled) -> Tensor(a)
2087: 
2088:   auto dispatch__autocast_to_full_precision = [](const at::Tensor & self, bool cuda_enabled, bool cpu_enabled) -> at::Tensor {
2089:     pybind11::gil_scoped_release no_gil;
2090:     return self._autocast_to_full_precision(cuda_enabled, cpu_enabled);
2091:   };
2092:   return wrap(dispatch__autocast_to_full_precision(self, _r.toBool(0), _r.toBool(1)));
2093:   Py_RETURN_NONE;
2094:   END_HANDLE_TH_ERRORS
2095: }
2096: 
2097: // _autocast_to_reduced_precision
2098: static PyObject * THPVariable__autocast_to_reduced_precision(PyObject* self_, PyObject* args, PyObject* kwargs)
2099: {
2100:   HANDLE_TH_ERRORS
2101:   const Tensor& self = THPVariable_Unpack(self_);
2102:   static PythonArgParser parser({
2103:     "_autocast_to_reduced_precision(bool cuda_enabled, bool cpu_enabled, ScalarType cuda_dtype, ScalarType cpu_dtype)",
2104:   }, /*traceable=*/true);
2105: 
2106:   ParsedArgs<4> parsed_args;
2107:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2108:   if(_r.has_torch_function()) {
2109:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2110:   }
2111:   // aten::_autocast_to_reduced_precision(Tensor(a) self, bool cuda_enabled, bool cpu_enabled, ScalarType cuda_dtype, ScalarType cpu_dtype) -> Tensor(a)
2112: 
2113:   auto dispatch__autocast_to_reduced_precision = [](const at::Tensor & self, bool cuda_enabled, bool cpu_enabled, at::ScalarType cuda_dtype, at::ScalarType cpu_dtype) -> at::Tensor {
2114:     pybind11::gil_scoped_release no_gil;
2115:     return self._autocast_to_reduced_precision(cuda_enabled, cpu_enabled, cuda_dtype, cpu_dtype);
2116:   };
2117:   return wrap(dispatch__autocast_to_reduced_precision(self, _r.toBool(0), _r.toBool(1), _r.scalartype(2), _r.scalartype(3)));
2118:   Py_RETURN_NONE;
2119:   END_HANDLE_TH_ERRORS
2120: }
2121: 
2122: // _coalesced_
2123: static PyObject * THPVariable__coalesced_(PyObject* self_, PyObject* args, PyObject* kwargs)
2124: {
2125:   HANDLE_TH_ERRORS
2126:   const Tensor& self = THPVariable_Unpack(self_);
2127:   static PythonArgParser parser({
2128:     "_coalesced_(bool coalesced)",
2129:   }, /*traceable=*/false);
2130: 
2131:   ParsedArgs<1> parsed_args;
2132:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2133:   if(_r.has_torch_function()) {
2134:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2135:   }
2136:   // aten::_coalesced_(Tensor(a!) self, bool coalesced) -> Tensor(a!)
2137: 
2138:   auto dispatch__coalesced_ = [](const at::Tensor & self, bool coalesced) -> at::Tensor {
2139:     pybind11::gil_scoped_release no_gil;
2140:     return self._coalesced_(coalesced);
2141:   };
2142:   return wrap(dispatch__coalesced_(self, _r.toBool(0)));
2143:   Py_RETURN_NONE;
2144:   END_HANDLE_TH_ERRORS
2145: }
2146: 
2147: // _conj
2148: static PyObject * THPVariable__conj(PyObject* self_, PyObject* args)
2149: {
2150:   HANDLE_TH_ERRORS
2151:   const Tensor& self = THPVariable_Unpack(self_);
2152:   if (has_torch_function(self_)) {
2153:     return handle_torch_function(self_, "_conj");
2154:   }
2155:   // aten::_conj(Tensor(a) self) -> Tensor(a)
2156: 
2157:   auto dispatch__conj = [](const at::Tensor & self) -> at::Tensor {
2158:     pybind11::gil_scoped_release no_gil;
2159:     return self._conj();
2160:   };
```

- EN: The main execution path in this span is carried by `THPVariable__addmm_activation`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable__addmm_activation`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2161-2280

```cpp
2161:   return wrap(dispatch__conj(self));
2162:   END_HANDLE_TH_ERRORS
2163: }
2164: 
2165: // _conj_physical
2166: static PyObject * THPVariable__conj_physical(PyObject* self_, PyObject* args)
2167: {
2168:   HANDLE_TH_ERRORS
2169:   const Tensor& self = THPVariable_Unpack(self_);
2170:   if (has_torch_function(self_)) {
2171:     return handle_torch_function(self_, "_conj_physical");
2172:   }
2173:   // aten::_conj_physical(Tensor self) -> Tensor
2174: 
2175:   auto dispatch__conj_physical = [](const at::Tensor & self) -> at::Tensor {
2176:     pybind11::gil_scoped_release no_gil;
2177:     return self._conj_physical();
2178:   };
2179:   return wrap(dispatch__conj_physical(self));
2180:   END_HANDLE_TH_ERRORS
2181: }
2182: 
2183: // _dimI
2184: static PyObject * THPVariable__dimI(PyObject* self_, PyObject* args)
2185: {
2186:   HANDLE_TH_ERRORS
2187:   const Tensor& self = THPVariable_Unpack(self_);
2188:   if (has_torch_function(self_)) {
2189:     return handle_torch_function(self_, "_dimI");
2190:   }
2191:   // aten::_dimI(Tensor self) -> int
2192: 
2193:   auto dispatch__dimI = [](const at::Tensor & self) -> int64_t {
2194:     pybind11::gil_scoped_release no_gil;
2195:     return self._dimI();
2196:   };
2197:   return wrap(dispatch__dimI(self));
2198:   END_HANDLE_TH_ERRORS
2199: }
2200: 
2201: // _dimV
2202: static PyObject * THPVariable__dimV(PyObject* self_, PyObject* args)
2203: {
2204:   HANDLE_TH_ERRORS
2205:   const Tensor& self = THPVariable_Unpack(self_);
2206:   if (has_torch_function(self_)) {
2207:     return handle_torch_function(self_, "_dimV");
2208:   }
2209:   // aten::_dimV(Tensor self) -> int
2210: 
2211:   auto dispatch__dimV = [](const at::Tensor & self) -> int64_t {
2212:     pybind11::gil_scoped_release no_gil;
2213:     return self._dimV();
2214:   };
2215:   return wrap(dispatch__dimV(self));
2216:   END_HANDLE_TH_ERRORS
2217: }
2218: 
2219: // _indices
2220: static PyObject * THPVariable__indices(PyObject* self_, PyObject* args)
2221: {
2222:   HANDLE_TH_ERRORS
2223:   const Tensor& self = THPVariable_Unpack(self_);
2224:   if (has_torch_function(self_)) {
2225:     return handle_torch_function(self_, "_indices");
2226:   }
2227:   // aten::_indices(Tensor(a) self) -> Tensor(a)
2228: 
2229:   auto dispatch__indices = [](const at::Tensor & self) -> at::Tensor {
2230:     pybind11::gil_scoped_release no_gil;
2231:     return self._indices();
2232:   };
2233:   return wrap(dispatch__indices(self));
2234:   END_HANDLE_TH_ERRORS
2235: }
2236: 
2237: // _is_all_true
2238: static PyObject * THPVariable__is_all_true(PyObject* self_, PyObject* args)
2239: {
2240:   HANDLE_TH_ERRORS
2241:   const Tensor& self = THPVariable_Unpack(self_);
2242:   if (has_torch_function(self_)) {
2243:     return handle_torch_function(self_, "_is_all_true");
2244:   }
2245:   // aten::_is_all_true(Tensor self) -> Tensor
2246: 
2247:   auto dispatch__is_all_true = [](const at::Tensor & self) -> at::Tensor {
2248:     pybind11::gil_scoped_release no_gil;
2249:     return self._is_all_true();
2250:   };
2251:   return wrap(dispatch__is_all_true(self));
2252:   END_HANDLE_TH_ERRORS
2253: }
2254: 
2255: // _is_any_true
2256: static PyObject * THPVariable__is_any_true(PyObject* self_, PyObject* args)
2257: {
2258:   HANDLE_TH_ERRORS
2259:   const Tensor& self = THPVariable_Unpack(self_);
2260:   if (has_torch_function(self_)) {
2261:     return handle_torch_function(self_, "_is_any_true");
2262:   }
2263:   // aten::_is_any_true(Tensor self) -> Tensor
2264: 
2265:   auto dispatch__is_any_true = [](const at::Tensor & self) -> at::Tensor {
2266:     pybind11::gil_scoped_release no_gil;
2267:     return self._is_any_true();
2268:   };
2269:   return wrap(dispatch__is_any_true(self));
2270:   END_HANDLE_TH_ERRORS
2271: }
2272: 
2273: // _is_zerotensor
2274: static PyObject * THPVariable__is_zerotensor(PyObject* self_, PyObject* args)
2275: {
2276:   HANDLE_TH_ERRORS
2277:   const Tensor& self = THPVariable_Unpack(self_);
2278:   if (has_torch_function(self_)) {
2279:     return handle_torch_function(self_, "_is_zerotensor");
2280:   }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__conj_physical`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__conj_physical`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2281-2400

```cpp
2281:   // aten::_is_zerotensor(Tensor self) -> bool
2282: 
2283:   auto dispatch__is_zerotensor = [](const at::Tensor & self) -> bool {
2284:     pybind11::gil_scoped_release no_gil;
2285:     return self._is_zerotensor();
2286:   };
2287:   return wrap(dispatch__is_zerotensor(self));
2288:   END_HANDLE_TH_ERRORS
2289: }
2290: 
2291: // _lazy_clone
2292: static PyObject * THPVariable__lazy_clone(PyObject* self_, PyObject* args)
2293: {
2294:   HANDLE_TH_ERRORS
2295:   const Tensor& self = THPVariable_Unpack(self_);
2296:   if (has_torch_function(self_)) {
2297:     return handle_torch_function(self_, "_lazy_clone");
2298:   }
2299:   // aten::_lazy_clone(Tensor self) -> Tensor
2300: 
2301:   auto dispatch__lazy_clone = [](const at::Tensor & self) -> at::Tensor {
2302:     pybind11::gil_scoped_release no_gil;
2303:     return self._lazy_clone();
2304:   };
2305:   return wrap(dispatch__lazy_clone(self));
2306:   END_HANDLE_TH_ERRORS
2307: }
2308: 
2309: // _neg_view
2310: static PyObject * THPVariable__neg_view(PyObject* self_, PyObject* args)
2311: {
2312:   HANDLE_TH_ERRORS
2313:   const Tensor& self = THPVariable_Unpack(self_);
2314:   if (has_torch_function(self_)) {
2315:     return handle_torch_function(self_, "_neg_view");
2316:   }
2317:   // aten::_neg_view(Tensor(a) self) -> Tensor(a)
2318: 
2319:   auto dispatch__neg_view = [](const at::Tensor & self) -> at::Tensor {
2320:     pybind11::gil_scoped_release no_gil;
2321:     return self._neg_view();
2322:   };
2323:   return wrap(dispatch__neg_view(self));
2324:   END_HANDLE_TH_ERRORS
2325: }
2326: 
2327: // _nested_tensor_size
2328: static PyObject * THPVariable__nested_tensor_size(PyObject* self_, PyObject* args)
2329: {
2330:   HANDLE_TH_ERRORS
2331:   const Tensor& self = THPVariable_Unpack(self_);
2332:   if (has_torch_function(self_)) {
2333:     return handle_torch_function(self_, "_nested_tensor_size");
2334:   }
2335:   // aten::_nested_tensor_size(Tensor self) -> Tensor
2336: 
2337:   auto dispatch__nested_tensor_size = [](const at::Tensor & self) -> at::Tensor {
2338:     pybind11::gil_scoped_release no_gil;
2339:     return self._nested_tensor_size();
2340:   };
2341:   return wrap(dispatch__nested_tensor_size(self));
2342:   END_HANDLE_TH_ERRORS
2343: }
2344: 
2345: // _nested_tensor_storage_offsets
2346: static PyObject * THPVariable__nested_tensor_storage_offsets(PyObject* self_, PyObject* args)
2347: {
2348:   HANDLE_TH_ERRORS
2349:   const Tensor& self = THPVariable_Unpack(self_);
2350:   if (has_torch_function(self_)) {
2351:     return handle_torch_function(self_, "_nested_tensor_storage_offsets");
2352:   }
2353:   // aten::_nested_tensor_storage_offsets(Tensor self) -> Tensor
2354: 
2355:   auto dispatch__nested_tensor_storage_offsets = [](const at::Tensor & self) -> at::Tensor {
2356:     pybind11::gil_scoped_release no_gil;
2357:     return self._nested_tensor_storage_offsets();
2358:   };
2359:   return wrap(dispatch__nested_tensor_storage_offsets(self));
2360:   END_HANDLE_TH_ERRORS
2361: }
2362: 
2363: // _nested_tensor_strides
2364: static PyObject * THPVariable__nested_tensor_strides(PyObject* self_, PyObject* args)
2365: {
2366:   HANDLE_TH_ERRORS
2367:   const Tensor& self = THPVariable_Unpack(self_);
2368:   if (has_torch_function(self_)) {
2369:     return handle_torch_function(self_, "_nested_tensor_strides");
2370:   }
2371:   // aten::_nested_tensor_strides(Tensor self) -> Tensor
2372: 
2373:   auto dispatch__nested_tensor_strides = [](const at::Tensor & self) -> at::Tensor {
2374:     pybind11::gil_scoped_release no_gil;
2375:     return self._nested_tensor_strides();
2376:   };
2377:   return wrap(dispatch__nested_tensor_strides(self));
2378:   END_HANDLE_TH_ERRORS
2379: }
2380: 
2381: // _nnz
2382: static PyObject * THPVariable__nnz(PyObject* self_, PyObject* args)
2383: {
2384:   HANDLE_TH_ERRORS
2385:   const Tensor& self = THPVariable_Unpack(self_);
2386:   if (has_torch_function(self_)) {
2387:     return handle_torch_function(self_, "_nnz");
2388:   }
2389:   // aten::_nnz(Tensor self) -> int
2390: 
2391:   auto dispatch__nnz = [](const at::Tensor & self) -> int64_t {
2392:     pybind11::gil_scoped_release no_gil;
2393:     return self._nnz();
2394:   };
2395:   return wrap(dispatch__nnz(self));
2396:   END_HANDLE_TH_ERRORS
2397: }
2398: 
2399: // _philox_normal_
2400: static PyObject * THPVariable__philox_normal_(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `_is_zerotensor`, `wrap`, `THPVariable__lazy_clone`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `_is_zerotensor`, `wrap`, `THPVariable__lazy_clone` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2401-2520

```cpp
2401: {
2402:   HANDLE_TH_ERRORS
2403:   const Tensor& self = THPVariable_Unpack(self_);
2404:   static PythonArgParser parser({
2405:     "_philox_normal_(Tensor key, double mean=0, double std=1)",
2406:   }, /*traceable=*/true);
2407: 
2408:   ParsedArgs<3> parsed_args;
2409:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2410:   if(_r.has_torch_function()) {
2411:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2412:   }
2413:   // aten::_philox_normal_(Tensor(a!) self, Tensor key, float mean=0, float std=1) -> Tensor(a!)
2414: 
2415:   auto dispatch__philox_normal_ = [](const at::Tensor & self, const at::Tensor & key, double mean, double std) -> at::Tensor {
2416:     pybind11::gil_scoped_release no_gil;
2417:     return self._philox_normal_(key, mean, std);
2418:   };
2419:   return wrap(dispatch__philox_normal_(self, _r.tensor(0), _r.toDouble(1), _r.toDouble(2)));
2420:   Py_RETURN_NONE;
2421:   END_HANDLE_TH_ERRORS
2422: }
2423: 
2424: // _philox_uniform_
2425: static PyObject * THPVariable__philox_uniform_(PyObject* self_, PyObject* args, PyObject* kwargs)
2426: {
2427:   HANDLE_TH_ERRORS
2428:   const Tensor& self = THPVariable_Unpack(self_);
2429:   static PythonArgParser parser({
2430:     "_philox_uniform_(Tensor key, double low=0, double high=1)",
2431:   }, /*traceable=*/true);
2432: 
2433:   ParsedArgs<3> parsed_args;
2434:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2435:   if(_r.has_torch_function()) {
2436:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2437:   }
2438:   // aten::_philox_uniform_(Tensor(a!) self, Tensor key, float low=0, float high=1) -> Tensor(a!)
2439: 
2440:   auto dispatch__philox_uniform_ = [](const at::Tensor & self, const at::Tensor & key, double low, double high) -> at::Tensor {
2441:     pybind11::gil_scoped_release no_gil;
2442:     return self._philox_uniform_(key, low, high);
2443:   };
2444:   return wrap(dispatch__philox_uniform_(self, _r.tensor(0), _r.toDouble(1), _r.toDouble(2)));
2445:   Py_RETURN_NONE;
2446:   END_HANDLE_TH_ERRORS
2447: }
2448: 
2449: // _sparse_mask_projection
2450: static PyObject * THPVariable__sparse_mask_projection(PyObject* self_, PyObject* args, PyObject* kwargs)
2451: {
2452:   HANDLE_TH_ERRORS
2453:   const Tensor& self = THPVariable_Unpack(self_);
2454:   static PythonArgParser parser({
2455:     "_sparse_mask_projection(Tensor mask, bool accumulate_matches=False)",
2456:   }, /*traceable=*/true);
2457: 
2458:   ParsedArgs<2> parsed_args;
2459:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2460:   if(_r.has_torch_function()) {
2461:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2462:   }
2463:   // aten::_sparse_mask_projection(Tensor self, Tensor mask, bool accumulate_matches=False) -> Tensor
2464: 
2465:   auto dispatch__sparse_mask_projection = [](const at::Tensor & self, const at::Tensor & mask, bool accumulate_matches) -> at::Tensor {
2466:     pybind11::gil_scoped_release no_gil;
2467:     return self._sparse_mask_projection(mask, accumulate_matches);
2468:   };
2469:   return wrap(dispatch__sparse_mask_projection(self, _r.tensor(0), _r.toBool(1)));
2470:   Py_RETURN_NONE;
2471:   END_HANDLE_TH_ERRORS
2472: }
2473: 
2474: // _to_dense
2475: static PyObject * THPVariable__to_dense(PyObject* self_, PyObject* args, PyObject* kwargs)
2476: {
2477:   HANDLE_TH_ERRORS
2478:   const Tensor& self = THPVariable_Unpack(self_);
2479:   static PythonArgParser parser({
2480:     "_to_dense(ScalarType? dtype=None, bool? masked_grad=None)",
2481:   }, /*traceable=*/true);
2482: 
2483:   ParsedArgs<2> parsed_args;
2484:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2485:   if(_r.has_torch_function()) {
2486:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2487:   }
2488:   // aten::_to_dense(Tensor self, ScalarType? dtype=None, bool? masked_grad=None) -> Tensor
2489: 
2490:   auto dispatch__to_dense = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype, ::std::optional<bool> masked_grad) -> at::Tensor {
2491:     pybind11::gil_scoped_release no_gil;
2492:     return self._to_dense(dtype, masked_grad);
2493:   };
2494:   return wrap(dispatch__to_dense(self, _r.scalartypeOptional(0), _r.toBoolOptional(1)));
2495:   Py_RETURN_NONE;
2496:   END_HANDLE_TH_ERRORS
2497: }
2498: 
2499: \
2500: // _to_sparse
2501: static PyObject * THPVariable__to_sparse(PyObject* self_, PyObject* args, PyObject* kwargs)
2502: {
2503:   HANDLE_TH_ERRORS
2504:   const Tensor& self = THPVariable_Unpack(self_);
2505:   static PythonArgParser parser({
2506:     "_to_sparse(*, Layout? layout=None, IntArrayRef[2]? blocksize=None, int64_t? dense_dim=None)",
2507:     "_to_sparse(int64_t sparse_dim)",
2508:   }, /*traceable=*/true);
2509: 
2510:   ParsedArgs<3> parsed_args;
2511:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2512:   if(_r.has_torch_function()) {
2513:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2514:   }
2515:   switch (_r.idx) {
2516:     case 0: {
2517:       // aten::_to_sparse(Tensor self, *, Layout? layout=None, int[2]? blocksize=None, int? dense_dim=None) -> Tensor
2518: 
2519:       auto dispatch__to_sparse = [](const at::Tensor & self, ::std::optional<at::Layout> layout, at::OptionalIntArrayRef blocksize, ::std::optional<int64_t> dense_dim) -> at::Tensor {
2520:         pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2521-2640

```cpp
2521:         return self._to_sparse(layout, blocksize, dense_dim);
2522:       };
2523:       return wrap(dispatch__to_sparse(self, _r.layoutOptional(0), _r.intlistOptional(1), _r.toInt64Optional(2)));
2524:     }
2525:     case 1: {
2526:       // aten::_to_sparse.sparse_dim(Tensor self, int sparse_dim) -> Tensor
2527: 
2528:       auto dispatch__to_sparse = [](const at::Tensor & self, int64_t sparse_dim) -> at::Tensor {
2529:         pybind11::gil_scoped_release no_gil;
2530:         return self._to_sparse(sparse_dim);
2531:       };
2532:       return wrap(dispatch__to_sparse(self, _r.toInt64(0)));
2533:     }
2534:   }
2535:   Py_RETURN_NONE;
2536:   END_HANDLE_TH_ERRORS
2537: }
2538: 
2539: // _to_sparse_bsc
2540: static PyObject * THPVariable__to_sparse_bsc(PyObject* self_, PyObject* args, PyObject* kwargs)
2541: {
2542:   HANDLE_TH_ERRORS
2543:   const Tensor& self = THPVariable_Unpack(self_);
2544:   static PythonArgParser parser({
2545:     "_to_sparse_bsc(IntArrayRef[2] blocksize, int64_t? dense_dim=None)",
2546:   }, /*traceable=*/true);
2547: 
2548:   ParsedArgs<2> parsed_args;
2549:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2550:   if(_r.has_torch_function()) {
2551:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2552:   }
2553:   // aten::_to_sparse_bsc(Tensor self, int[2] blocksize, int? dense_dim=None) -> Tensor
2554: 
2555:   auto dispatch__to_sparse_bsc = [](const at::Tensor & self, at::IntArrayRef blocksize, ::std::optional<int64_t> dense_dim) -> at::Tensor {
2556:     pybind11::gil_scoped_release no_gil;
2557:     return self._to_sparse_bsc(blocksize, dense_dim);
2558:   };
2559:   return wrap(dispatch__to_sparse_bsc(self, _r.intlist(0), _r.toInt64Optional(1)));
2560:   Py_RETURN_NONE;
2561:   END_HANDLE_TH_ERRORS
2562: }
2563: 
2564: // _to_sparse_bsr
2565: static PyObject * THPVariable__to_sparse_bsr(PyObject* self_, PyObject* args, PyObject* kwargs)
2566: {
2567:   HANDLE_TH_ERRORS
2568:   const Tensor& self = THPVariable_Unpack(self_);
2569:   static PythonArgParser parser({
2570:     "_to_sparse_bsr(IntArrayRef[2] blocksize, int64_t? dense_dim=None)",
2571:   }, /*traceable=*/true);
2572: 
2573:   ParsedArgs<2> parsed_args;
2574:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2575:   if(_r.has_torch_function()) {
2576:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2577:   }
2578:   // aten::_to_sparse_bsr(Tensor self, int[2] blocksize, int? dense_dim=None) -> Tensor
2579: 
2580:   auto dispatch__to_sparse_bsr = [](const at::Tensor & self, at::IntArrayRef blocksize, ::std::optional<int64_t> dense_dim) -> at::Tensor {
2581:     pybind11::gil_scoped_release no_gil;
2582:     return self._to_sparse_bsr(blocksize, dense_dim);
2583:   };
2584:   return wrap(dispatch__to_sparse_bsr(self, _r.intlist(0), _r.toInt64Optional(1)));
2585:   Py_RETURN_NONE;
2586:   END_HANDLE_TH_ERRORS
2587: }
2588: 
2589: // _to_sparse_csc
2590: static PyObject * THPVariable__to_sparse_csc(PyObject* self_, PyObject* args, PyObject* kwargs)
2591: {
2592:   HANDLE_TH_ERRORS
2593:   const Tensor& self = THPVariable_Unpack(self_);
2594:   static PythonArgParser parser({
2595:     "_to_sparse_csc(int64_t? dense_dim=None)",
2596:   }, /*traceable=*/true);
2597: 
2598:   ParsedArgs<1> parsed_args;
2599:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2600:   if(_r.has_torch_function()) {
2601:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2602:   }
2603:   // aten::_to_sparse_csc(Tensor self, int? dense_dim=None) -> Tensor
2604: 
2605:   auto dispatch__to_sparse_csc = [](const at::Tensor & self, ::std::optional<int64_t> dense_dim) -> at::Tensor {
2606:     pybind11::gil_scoped_release no_gil;
2607:     return self._to_sparse_csc(dense_dim);
2608:   };
2609:   return wrap(dispatch__to_sparse_csc(self, _r.toInt64Optional(0)));
2610:   Py_RETURN_NONE;
2611:   END_HANDLE_TH_ERRORS
2612: }
2613: 
2614: // _to_sparse_csr
2615: static PyObject * THPVariable__to_sparse_csr(PyObject* self_, PyObject* args, PyObject* kwargs)
2616: {
2617:   HANDLE_TH_ERRORS
2618:   const Tensor& self = THPVariable_Unpack(self_);
2619:   static PythonArgParser parser({
2620:     "_to_sparse_csr(int64_t? dense_dim=None)",
2621:   }, /*traceable=*/true);
2622: 
2623:   ParsedArgs<1> parsed_args;
2624:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2625:   if(_r.has_torch_function()) {
2626:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2627:   }
2628:   // aten::_to_sparse_csr(Tensor self, int? dense_dim=None) -> Tensor
2629: 
2630:   auto dispatch__to_sparse_csr = [](const at::Tensor & self, ::std::optional<int64_t> dense_dim) -> at::Tensor {
2631:     pybind11::gil_scoped_release no_gil;
2632:     return self._to_sparse_csr(dense_dim);
2633:   };
2634:   return wrap(dispatch__to_sparse_csr(self, _r.toInt64Optional(0)));
2635:   Py_RETURN_NONE;
2636:   END_HANDLE_TH_ERRORS
2637: }
2638: 
2639: // _values
2640: static PyObject * THPVariable__values(PyObject* self_, PyObject* args)
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable__to_sparse_bsc`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable__to_sparse_bsc`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 2641-2760

```cpp
2641: {
2642:   HANDLE_TH_ERRORS
2643:   const Tensor& self = THPVariable_Unpack(self_);
2644:   if (has_torch_function(self_)) {
2645:     return handle_torch_function(self_, "_values");
2646:   }
2647:   // aten::_values(Tensor(a) self) -> Tensor(a)
2648: 
2649:   auto dispatch__values = [](const at::Tensor & self) -> at::Tensor {
2650:     pybind11::gil_scoped_release no_gil;
2651:     return self._values();
2652:   };
2653:   return wrap(dispatch__values(self));
2654:   END_HANDLE_TH_ERRORS
2655: }
2656: 
2657: // abs
2658: static PyObject * THPVariable_abs(PyObject* self_, PyObject* args)
2659: {
2660:   HANDLE_TH_ERRORS
2661:   const Tensor& self = THPVariable_Unpack(self_);
2662:   if (has_torch_function(self_)) {
2663:     return handle_torch_function(self_, "abs");
2664:   }
2665:   // aten::abs(Tensor self) -> Tensor
2666: 
2667:   auto dispatch_abs = [](const at::Tensor & self) -> at::Tensor {
2668:     pybind11::gil_scoped_release no_gil;
2669:     return self.abs();
2670:   };
2671:   return wrap(dispatch_abs(self));
2672:   END_HANDLE_TH_ERRORS
2673: }
2674: 
2675: // abs_
2676: static PyObject * THPVariable_abs_(PyObject* self_, PyObject* args)
2677: {
2678:   HANDLE_TH_ERRORS
2679:   const Tensor& self = THPVariable_Unpack(self_);
2680:   if (has_torch_function(self_)) {
2681:     return handle_torch_function(self_, "abs_");
2682:   }
2683:   // aten::abs_(Tensor(a!) self) -> Tensor(a!)
2684: 
2685:   auto dispatch_abs_ = [](const at::Tensor & self) -> at::Tensor {
2686:     pybind11::gil_scoped_release no_gil;
2687:     return self.abs_();
2688:   };
2689:   return wrap(dispatch_abs_(self));
2690:   END_HANDLE_TH_ERRORS
2691: }
2692: 
2693: // absolute
2694: static PyObject * THPVariable_absolute(PyObject* self_, PyObject* args)
2695: {
2696:   HANDLE_TH_ERRORS
2697:   const Tensor& self = THPVariable_Unpack(self_);
2698:   if (has_torch_function(self_)) {
2699:     return handle_torch_function(self_, "absolute");
2700:   }
2701:   // aten::absolute(Tensor self) -> Tensor
2702: 
2703:   auto dispatch_absolute = [](const at::Tensor & self) -> at::Tensor {
2704:     pybind11::gil_scoped_release no_gil;
2705:     return self.absolute();
2706:   };
2707:   return wrap(dispatch_absolute(self));
2708:   END_HANDLE_TH_ERRORS
2709: }
2710: 
2711: // absolute_
2712: static PyObject * THPVariable_absolute_(PyObject* self_, PyObject* args)
2713: {
2714:   HANDLE_TH_ERRORS
2715:   const Tensor& self = THPVariable_Unpack(self_);
2716:   if (has_torch_function(self_)) {
2717:     return handle_torch_function(self_, "absolute_");
2718:   }
2719:   // aten::absolute_(Tensor(a!) self) -> Tensor(a!)
2720: 
2721:   auto dispatch_absolute_ = [](const at::Tensor & self) -> at::Tensor {
2722:     pybind11::gil_scoped_release no_gil;
2723:     return self.absolute_();
2724:   };
2725:   return wrap(dispatch_absolute_(self));
2726:   END_HANDLE_TH_ERRORS
2727: }
2728: 
2729: // acos
2730: static PyObject * THPVariable_acos(PyObject* self_, PyObject* args)
2731: {
2732:   HANDLE_TH_ERRORS
2733:   const Tensor& self = THPVariable_Unpack(self_);
2734:   if (has_torch_function(self_)) {
2735:     return handle_torch_function(self_, "acos");
2736:   }
2737:   // aten::acos(Tensor self) -> Tensor
2738: 
2739:   auto dispatch_acos = [](const at::Tensor & self) -> at::Tensor {
2740:     pybind11::gil_scoped_release no_gil;
2741:     return self.acos();
2742:   };
2743:   return wrap(dispatch_acos(self));
2744:   END_HANDLE_TH_ERRORS
2745: }
2746: 
2747: // acos_
2748: static PyObject * THPVariable_acos_(PyObject* self_, PyObject* args)
2749: {
2750:   HANDLE_TH_ERRORS
2751:   const Tensor& self = THPVariable_Unpack(self_);
2752:   if (has_torch_function(self_)) {
2753:     return handle_torch_function(self_, "acos_");
2754:   }
2755:   // aten::acos_(Tensor(a!) self) -> Tensor(a!)
2756: 
2757:   auto dispatch_acos_ = [](const at::Tensor & self) -> at::Tensor {
2758:     pybind11::gil_scoped_release no_gil;
2759:     return self.acos_();
2760:   };
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `handle_torch_function`, `_values`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `handle_torch_function`, `_values` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2761-2880

```cpp
2761:   return wrap(dispatch_acos_(self));
2762:   END_HANDLE_TH_ERRORS
2763: }
2764: 
2765: // acosh
2766: static PyObject * THPVariable_acosh(PyObject* self_, PyObject* args)
2767: {
2768:   HANDLE_TH_ERRORS
2769:   const Tensor& self = THPVariable_Unpack(self_);
2770:   if (has_torch_function(self_)) {
2771:     return handle_torch_function(self_, "acosh");
2772:   }
2773:   // aten::acosh(Tensor self) -> Tensor
2774: 
2775:   auto dispatch_acosh = [](const at::Tensor & self) -> at::Tensor {
2776:     pybind11::gil_scoped_release no_gil;
2777:     return self.acosh();
2778:   };
2779:   return wrap(dispatch_acosh(self));
2780:   END_HANDLE_TH_ERRORS
2781: }
2782: 
2783: // acosh_
2784: static PyObject * THPVariable_acosh_(PyObject* self_, PyObject* args)
2785: {
2786:   HANDLE_TH_ERRORS
2787:   const Tensor& self = THPVariable_Unpack(self_);
2788:   if (has_torch_function(self_)) {
2789:     return handle_torch_function(self_, "acosh_");
2790:   }
2791:   // aten::acosh_(Tensor(a!) self) -> Tensor(a!)
2792: 
2793:   auto dispatch_acosh_ = [](const at::Tensor & self) -> at::Tensor {
2794:     pybind11::gil_scoped_release no_gil;
2795:     return self.acosh_();
2796:   };
2797:   return wrap(dispatch_acosh_(self));
2798:   END_HANDLE_TH_ERRORS
2799: }
2800: 
2801: \
2802: // add
2803: static PyObject * THPVariable_add(PyObject* self_, PyObject* args, PyObject* kwargs)
2804: {
2805:   HANDLE_TH_ERRORS
2806:   const Tensor& self = THPVariable_Unpack(self_);
2807:   static PythonArgParser parser({
2808:     "add(Scalar alpha, Tensor other)|deprecated",
2809:     "add(Tensor other, *, Scalar alpha=1)",
2810:   }, /*traceable=*/true);
2811: 
2812:   ParsedArgs<2> parsed_args;
2813:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2814:   if(_r.has_torch_function()) {
2815:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2816:   }
2817:   switch (_r.idx) {
2818:     case 0: {
2819:       // [deprecated] aten::add(Tensor self, Scalar alpha, Tensor other) -> Tensor
2820: 
2821:       auto dispatch_add = [](const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
2822:         pybind11::gil_scoped_release no_gil;
2823:         return self.add(other, alpha);
2824:       };
2825:       return wrap(dispatch_add(self, _r.scalar(0), _r.tensor(1)));
2826:     }
2827:     case 1: {
2828:       // aten::add.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
2829: 
2830:       auto dispatch_add = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
2831:         pybind11::gil_scoped_release no_gil;
2832:         return self.add(other, alpha);
2833:       };
2834:       return wrap(dispatch_add(self, _r.tensor(0), _r.scalar(1)));
2835:     }
2836:   }
2837:   Py_RETURN_NONE;
2838:   END_HANDLE_TH_ERRORS
2839: }
2840: 
2841: \
2842: // add_
2843: static PyObject * THPVariable_add_(PyObject* self_, PyObject* args, PyObject* kwargs)
2844: {
2845:   HANDLE_TH_ERRORS
2846:   const Tensor& self = THPVariable_Unpack(self_);
2847:   static PythonArgParser parser({
2848:     "add_(Scalar alpha, Tensor other)|deprecated",
2849:     "add_(Tensor other, *, Scalar alpha=1)",
2850:   }, /*traceable=*/true);
2851: 
2852:   ParsedArgs<2> parsed_args;
2853:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2854:   if(_r.has_torch_function()) {
2855:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2856:   }
2857:   switch (_r.idx) {
2858:     case 0: {
2859:       // [deprecated] aten::add_(Tensor(a!) self, Scalar alpha, Tensor other) -> Tensor(a!)
2860: 
2861:       auto dispatch_add_ = [](const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
2862:         pybind11::gil_scoped_release no_gil;
2863:         return self.add_(other, alpha);
2864:       };
2865:       return wrap(dispatch_add_(self, _r.scalar(0), _r.tensor(1)));
2866:     }
2867:     case 1: {
2868:       // aten::add_.Tensor(Tensor(a!) self, Tensor other, *, Scalar alpha=1) -> Tensor(a!)
2869: 
2870:       auto dispatch_add_ = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
2871:         pybind11::gil_scoped_release no_gil;
2872:         return self.add_(other, alpha);
2873:       };
2874:       return wrap(dispatch_add_(self, _r.tensor(0), _r.scalar(1)));
2875:     }
2876:   }
2877:   Py_RETURN_NONE;
2878:   END_HANDLE_TH_ERRORS
2879: }
2880: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_acosh`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_acosh`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 2881-3000

```cpp
2881: \
2882: // addbmm
2883: static PyObject * THPVariable_addbmm(PyObject* self_, PyObject* args, PyObject* kwargs)
2884: {
2885:   HANDLE_TH_ERRORS
2886:   const Tensor& self = THPVariable_Unpack(self_);
2887:   static PythonArgParser parser({
2888:     "addbmm(Scalar beta, Scalar alpha, Tensor batch1, Tensor batch2)|deprecated",
2889:     "addbmm(Scalar beta, Tensor batch1, Tensor batch2)|deprecated",
2890:     "addbmm(Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1)",
2891:   }, /*traceable=*/true);
2892: 
2893:   ParsedArgs<4> parsed_args;
2894:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2895:   if(_r.has_torch_function()) {
2896:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2897:   }
2898:   switch (_r.idx) {
2899:     case 0: {
2900:       // [deprecated] aten::addbmm(Scalar beta, Tensor self, Scalar alpha, Tensor batch1, Tensor batch2) -> Tensor
2901: 
2902:       auto dispatch_addbmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2903:         pybind11::gil_scoped_release no_gil;
2904:         return self.addbmm(batch1, batch2, beta, alpha);
2905:       };
2906:       return wrap(dispatch_addbmm(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
2907:     }
2908:     case 1: {
2909:       // [deprecated] aten::addbmm(Scalar beta, Tensor self, Tensor batch1, Tensor batch2) -> Tensor
2910: 
2911:       auto dispatch_addbmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2912:         pybind11::gil_scoped_release no_gil;
2913:         return self.addbmm(batch1, batch2, beta, 1);
2914:       };
2915:       return wrap(dispatch_addbmm(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
2916:     }
2917:     case 2: {
2918:       // aten::addbmm(Tensor self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
2919: 
2920:       auto dispatch_addbmm = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
2921:         pybind11::gil_scoped_release no_gil;
2922:         return self.addbmm(batch1, batch2, beta, alpha);
2923:       };
2924:       return wrap(dispatch_addbmm(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
2925:     }
2926:   }
2927:   Py_RETURN_NONE;
2928:   END_HANDLE_TH_ERRORS
2929: }
2930: 
2931: \
2932: // addbmm_
2933: static PyObject * THPVariable_addbmm_(PyObject* self_, PyObject* args, PyObject* kwargs)
2934: {
2935:   HANDLE_TH_ERRORS
2936:   const Tensor& self = THPVariable_Unpack(self_);
2937:   static PythonArgParser parser({
2938:     "addbmm_(Scalar beta, Scalar alpha, Tensor batch1, Tensor batch2)|deprecated",
2939:     "addbmm_(Scalar beta, Tensor batch1, Tensor batch2)|deprecated",
2940:     "addbmm_(Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1)",
2941:   }, /*traceable=*/true);
2942: 
2943:   ParsedArgs<4> parsed_args;
2944:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2945:   if(_r.has_torch_function()) {
2946:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2947:   }
2948:   switch (_r.idx) {
2949:     case 0: {
2950:       // [deprecated] aten::addbmm_(Scalar beta, Tensor(a!) self, Scalar alpha, Tensor batch1, Tensor batch2) -> Tensor(a!)
2951: 
2952:       auto dispatch_addbmm_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2953:         pybind11::gil_scoped_release no_gil;
2954:         return self.addbmm_(batch1, batch2, beta, alpha);
2955:       };
2956:       return wrap(dispatch_addbmm_(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
2957:     }
2958:     case 1: {
2959:       // [deprecated] aten::addbmm_(Scalar beta, Tensor(a!) self, Tensor batch1, Tensor batch2) -> Tensor(a!)
2960: 
2961:       auto dispatch_addbmm_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
2962:         pybind11::gil_scoped_release no_gil;
2963:         return self.addbmm_(batch1, batch2, beta, 1);
2964:       };
2965:       return wrap(dispatch_addbmm_(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
2966:     }
2967:     case 2: {
2968:       // aten::addbmm_(Tensor(a!) self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1) -> Tensor(a!)
2969: 
2970:       auto dispatch_addbmm_ = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
2971:         pybind11::gil_scoped_release no_gil;
2972:         return self.addbmm_(batch1, batch2, beta, alpha);
2973:       };
2974:       return wrap(dispatch_addbmm_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
2975:     }
2976:   }
2977:   Py_RETURN_NONE;
2978:   END_HANDLE_TH_ERRORS
2979: }
2980: 
2981: \
2982: // addcdiv
2983: static PyObject * THPVariable_addcdiv(PyObject* self_, PyObject* args, PyObject* kwargs)
2984: {
2985:   HANDLE_TH_ERRORS
2986:   const Tensor& self = THPVariable_Unpack(self_);
2987:   static PythonArgParser parser({
2988:     "addcdiv(Scalar value, Tensor tensor1, Tensor tensor2)|deprecated",
2989:     "addcdiv(Tensor tensor1, Tensor tensor2, *, Scalar value=1)",
2990:   }, /*traceable=*/true);
2991: 
2992:   ParsedArgs<3> parsed_args;
2993:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
2994:   if(_r.has_torch_function()) {
2995:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
2996:   }
2997:   switch (_r.idx) {
2998:     case 0: {
2999:       // [deprecated] aten::addcdiv(Tensor self, Scalar value, Tensor tensor1, Tensor tensor2) -> Tensor
3000: 
```

- EN: The main execution path in this span is carried by `THPVariable_addbmm`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_addbmm`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3001-3120

```cpp
3001:       auto dispatch_addcdiv = [](const at::Tensor & self, const at::Scalar & value, const at::Tensor & tensor1, const at::Tensor & tensor2) -> at::Tensor {
3002:         pybind11::gil_scoped_release no_gil;
3003:         return self.addcdiv(tensor1, tensor2, value);
3004:       };
3005:       return wrap(dispatch_addcdiv(self, _r.scalar(0), _r.tensor(1), _r.tensor(2)));
3006:     }
3007:     case 1: {
3008:       // aten::addcdiv(Tensor self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> Tensor
3009: 
3010:       auto dispatch_addcdiv = [](const at::Tensor & self, const at::Tensor & tensor1, const at::Tensor & tensor2, const at::Scalar & value) -> at::Tensor {
3011:         pybind11::gil_scoped_release no_gil;
3012:         return self.addcdiv(tensor1, tensor2, value);
3013:       };
3014:       return wrap(dispatch_addcdiv(self, _r.tensor(0), _r.tensor(1), _r.scalar(2)));
3015:     }
3016:   }
3017:   Py_RETURN_NONE;
3018:   END_HANDLE_TH_ERRORS
3019: }
3020: 
3021: \
3022: // addcdiv_
3023: static PyObject * THPVariable_addcdiv_(PyObject* self_, PyObject* args, PyObject* kwargs)
3024: {
3025:   HANDLE_TH_ERRORS
3026:   const Tensor& self = THPVariable_Unpack(self_);
3027:   static PythonArgParser parser({
3028:     "addcdiv_(Scalar value, Tensor tensor1, Tensor tensor2)|deprecated",
3029:     "addcdiv_(Tensor tensor1, Tensor tensor2, *, Scalar value=1)",
3030:   }, /*traceable=*/true);
3031: 
3032:   ParsedArgs<3> parsed_args;
3033:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3034:   if(_r.has_torch_function()) {
3035:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3036:   }
3037:   switch (_r.idx) {
3038:     case 0: {
3039:       // [deprecated] aten::addcdiv_(Tensor(a!) self, Scalar value, Tensor tensor1, Tensor tensor2) -> Tensor(a!)
3040: 
3041:       auto dispatch_addcdiv_ = [](const at::Tensor & self, const at::Scalar & value, const at::Tensor & tensor1, const at::Tensor & tensor2) -> at::Tensor {
3042:         pybind11::gil_scoped_release no_gil;
3043:         return self.addcdiv_(tensor1, tensor2, value);
3044:       };
3045:       return wrap(dispatch_addcdiv_(self, _r.scalar(0), _r.tensor(1), _r.tensor(2)));
3046:     }
3047:     case 1: {
3048:       // aten::addcdiv_(Tensor(a!) self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> Tensor(a!)
3049: 
3050:       auto dispatch_addcdiv_ = [](const at::Tensor & self, const at::Tensor & tensor1, const at::Tensor & tensor2, const at::Scalar & value) -> at::Tensor {
3051:         pybind11::gil_scoped_release no_gil;
3052:         return self.addcdiv_(tensor1, tensor2, value);
3053:       };
3054:       return wrap(dispatch_addcdiv_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2)));
3055:     }
3056:   }
3057:   Py_RETURN_NONE;
3058:   END_HANDLE_TH_ERRORS
3059: }
3060: 
3061: \
3062: // addcmul
3063: static PyObject * THPVariable_addcmul(PyObject* self_, PyObject* args, PyObject* kwargs)
3064: {
3065:   HANDLE_TH_ERRORS
3066:   const Tensor& self = THPVariable_Unpack(self_);
3067:   static PythonArgParser parser({
3068:     "addcmul(Scalar value, Tensor tensor1, Tensor tensor2)|deprecated",
3069:     "addcmul(Tensor tensor1, Tensor tensor2, *, Scalar value=1)",
3070:   }, /*traceable=*/true);
3071: 
3072:   ParsedArgs<3> parsed_args;
3073:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3074:   if(_r.has_torch_function()) {
3075:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3076:   }
3077:   switch (_r.idx) {
3078:     case 0: {
3079:       // [deprecated] aten::addcmul(Tensor self, Scalar value, Tensor tensor1, Tensor tensor2) -> Tensor
3080: 
3081:       auto dispatch_addcmul = [](const at::Tensor & self, const at::Scalar & value, const at::Tensor & tensor1, const at::Tensor & tensor2) -> at::Tensor {
3082:         pybind11::gil_scoped_release no_gil;
3083:         return self.addcmul(tensor1, tensor2, value);
3084:       };
3085:       return wrap(dispatch_addcmul(self, _r.scalar(0), _r.tensor(1), _r.tensor(2)));
3086:     }
3087:     case 1: {
3088:       // aten::addcmul(Tensor self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> Tensor
3089: 
3090:       auto dispatch_addcmul = [](const at::Tensor & self, const at::Tensor & tensor1, const at::Tensor & tensor2, const at::Scalar & value) -> at::Tensor {
3091:         pybind11::gil_scoped_release no_gil;
3092:         return self.addcmul(tensor1, tensor2, value);
3093:       };
3094:       return wrap(dispatch_addcmul(self, _r.tensor(0), _r.tensor(1), _r.scalar(2)));
3095:     }
3096:   }
3097:   Py_RETURN_NONE;
3098:   END_HANDLE_TH_ERRORS
3099: }
3100: 
3101: \
3102: // addcmul_
3103: static PyObject * THPVariable_addcmul_(PyObject* self_, PyObject* args, PyObject* kwargs)
3104: {
3105:   HANDLE_TH_ERRORS
3106:   const Tensor& self = THPVariable_Unpack(self_);
3107:   static PythonArgParser parser({
3108:     "addcmul_(Scalar value, Tensor tensor1, Tensor tensor2)|deprecated",
3109:     "addcmul_(Tensor tensor1, Tensor tensor2, *, Scalar value=1)",
3110:   }, /*traceable=*/true);
3111: 
3112:   ParsedArgs<3> parsed_args;
3113:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3114:   if(_r.has_torch_function()) {
3115:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3116:   }
3117:   switch (_r.idx) {
3118:     case 0: {
3119:       // [deprecated] aten::addcmul_(Tensor(a!) self, Scalar value, Tensor tensor1, Tensor tensor2) -> Tensor(a!)
3120: 
```

- EN: The main execution path in this span is carried by `wrap`, `addcdiv`, `THPVariable_addcdiv_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `addcdiv`, `THPVariable_addcdiv_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3121-3240

```cpp
3121:       auto dispatch_addcmul_ = [](const at::Tensor & self, const at::Scalar & value, const at::Tensor & tensor1, const at::Tensor & tensor2) -> at::Tensor {
3122:         pybind11::gil_scoped_release no_gil;
3123:         return self.addcmul_(tensor1, tensor2, value);
3124:       };
3125:       return wrap(dispatch_addcmul_(self, _r.scalar(0), _r.tensor(1), _r.tensor(2)));
3126:     }
3127:     case 1: {
3128:       // aten::addcmul_(Tensor(a!) self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> Tensor(a!)
3129: 
3130:       auto dispatch_addcmul_ = [](const at::Tensor & self, const at::Tensor & tensor1, const at::Tensor & tensor2, const at::Scalar & value) -> at::Tensor {
3131:         pybind11::gil_scoped_release no_gil;
3132:         return self.addcmul_(tensor1, tensor2, value);
3133:       };
3134:       return wrap(dispatch_addcmul_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2)));
3135:     }
3136:   }
3137:   Py_RETURN_NONE;
3138:   END_HANDLE_TH_ERRORS
3139: }
3140: 
3141: \
3142: // addmm
3143: static PyObject * THPVariable_addmm(PyObject* self_, PyObject* args, PyObject* kwargs)
3144: {
3145:   HANDLE_TH_ERRORS
3146:   const Tensor& self = THPVariable_Unpack(self_);
3147:   static PythonArgParser parser({
3148:     "addmm(Scalar beta, Scalar alpha, Tensor mat1, Tensor mat2)|deprecated",
3149:     "addmm(Scalar beta, Tensor mat1, Tensor mat2)|deprecated",
3150:     "addmm(Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1)",
3151:   }, /*traceable=*/true);
3152: 
3153:   ParsedArgs<4> parsed_args;
3154:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3155:   if(_r.has_torch_function()) {
3156:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3157:   }
3158:   switch (_r.idx) {
3159:     case 0: {
3160:       // [deprecated] aten::addmm(Scalar beta, Tensor self, Scalar alpha, Tensor mat1, Tensor mat2) -> Tensor
3161: 
3162:       auto dispatch_addmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
3163:         pybind11::gil_scoped_release no_gil;
3164:         return self.addmm(mat1, mat2, beta, alpha);
3165:       };
3166:       return wrap(dispatch_addmm(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
3167:     }
3168:     case 1: {
3169:       // [deprecated] aten::addmm(Scalar beta, Tensor self, Tensor mat1, Tensor mat2) -> Tensor
3170: 
3171:       auto dispatch_addmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
3172:         pybind11::gil_scoped_release no_gil;
3173:         return self.addmm(mat1, mat2, beta, 1);
3174:       };
3175:       return wrap(dispatch_addmm(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
3176:     }
3177:     case 2: {
3178:       // aten::addmm(Tensor self, Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
3179: 
3180:       auto dispatch_addmm = [](const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
3181:         pybind11::gil_scoped_release no_gil;
3182:         return self.addmm(mat1, mat2, beta, alpha);
3183:       };
3184:       return wrap(dispatch_addmm(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
3185:     }
3186:   }
3187:   Py_RETURN_NONE;
3188:   END_HANDLE_TH_ERRORS
3189: }
3190: 
3191: \
3192: // addmm_
3193: static PyObject * THPVariable_addmm_(PyObject* self_, PyObject* args, PyObject* kwargs)
3194: {
3195:   HANDLE_TH_ERRORS
3196:   const Tensor& self = THPVariable_Unpack(self_);
3197:   static PythonArgParser parser({
3198:     "addmm_(Scalar beta, Scalar alpha, Tensor mat1, Tensor mat2)|deprecated",
3199:     "addmm_(Scalar beta, Tensor mat1, Tensor mat2)|deprecated",
3200:     "addmm_(Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1)",
3201:   }, /*traceable=*/true);
3202: 
3203:   ParsedArgs<4> parsed_args;
3204:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3205:   if(_r.has_torch_function()) {
3206:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3207:   }
3208:   switch (_r.idx) {
3209:     case 0: {
3210:       // [deprecated] aten::addmm_(Scalar beta, Tensor(a!) self, Scalar alpha, Tensor mat1, Tensor mat2) -> Tensor(a!)
3211: 
3212:       auto dispatch_addmm_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
3213:         pybind11::gil_scoped_release no_gil;
3214:         return self.addmm_(mat1, mat2, beta, alpha);
3215:       };
3216:       return wrap(dispatch_addmm_(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
3217:     }
3218:     case 1: {
3219:       // [deprecated] aten::addmm_(Scalar beta, Tensor(a!) self, Tensor mat1, Tensor mat2) -> Tensor(a!)
3220: 
3221:       auto dispatch_addmm_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
3222:         pybind11::gil_scoped_release no_gil;
3223:         return self.addmm_(mat1, mat2, beta, 1);
3224:       };
3225:       return wrap(dispatch_addmm_(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
3226:     }
3227:     case 2: {
3228:       // aten::addmm_(Tensor(a!) self, Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1) -> Tensor(a!)
3229: 
3230:       auto dispatch_addmm_ = [](const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
3231:         pybind11::gil_scoped_release no_gil;
3232:         return self.addmm_(mat1, mat2, beta, alpha);
3233:       };
3234:       return wrap(dispatch_addmm_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
3235:     }
3236:   }
3237:   Py_RETURN_NONE;
3238:   END_HANDLE_TH_ERRORS
3239: }
3240: 
```

- EN: The main execution path in this span is carried by `wrap`, `addcmul_`, `THPVariable_addmm`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `addcmul_`, `THPVariable_addmm` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3241-3360

```cpp
3241: \
3242: // addmv
3243: static PyObject * THPVariable_addmv(PyObject* self_, PyObject* args, PyObject* kwargs)
3244: {
3245:   HANDLE_TH_ERRORS
3246:   const Tensor& self = THPVariable_Unpack(self_);
3247:   static PythonArgParser parser({
3248:     "addmv(Scalar beta, Scalar alpha, Tensor mat, Tensor vec)|deprecated",
3249:     "addmv(Scalar beta, Tensor mat, Tensor vec)|deprecated",
3250:     "addmv(Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1)",
3251:   }, /*traceable=*/true);
3252: 
3253:   ParsedArgs<4> parsed_args;
3254:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3255:   if(_r.has_torch_function()) {
3256:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3257:   }
3258:   switch (_r.idx) {
3259:     case 0: {
3260:       // [deprecated] aten::addmv(Scalar beta, Tensor self, Scalar alpha, Tensor mat, Tensor vec) -> Tensor
3261: 
3262:       auto dispatch_addmv = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
3263:         pybind11::gil_scoped_release no_gil;
3264:         return self.addmv(mat, vec, beta, alpha);
3265:       };
3266:       return wrap(dispatch_addmv(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
3267:     }
3268:     case 1: {
3269:       // [deprecated] aten::addmv(Scalar beta, Tensor self, Tensor mat, Tensor vec) -> Tensor
3270: 
3271:       auto dispatch_addmv = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
3272:         pybind11::gil_scoped_release no_gil;
3273:         return self.addmv(mat, vec, beta, 1);
3274:       };
3275:       return wrap(dispatch_addmv(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
3276:     }
3277:     case 2: {
3278:       // aten::addmv(Tensor self, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1) -> Tensor
3279: 
3280:       auto dispatch_addmv = [](const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
3281:         pybind11::gil_scoped_release no_gil;
3282:         return self.addmv(mat, vec, beta, alpha);
3283:       };
3284:       return wrap(dispatch_addmv(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
3285:     }
3286:   }
3287:   Py_RETURN_NONE;
3288:   END_HANDLE_TH_ERRORS
3289: }
3290: 
3291: \
3292: // addmv_
3293: static PyObject * THPVariable_addmv_(PyObject* self_, PyObject* args, PyObject* kwargs)
3294: {
3295:   HANDLE_TH_ERRORS
3296:   const Tensor& self = THPVariable_Unpack(self_);
3297:   static PythonArgParser parser({
3298:     "addmv_(Scalar beta, Scalar alpha, Tensor mat, Tensor vec)|deprecated",
3299:     "addmv_(Scalar beta, Tensor mat, Tensor vec)|deprecated",
3300:     "addmv_(Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1)",
3301:   }, /*traceable=*/true);
3302: 
3303:   ParsedArgs<4> parsed_args;
3304:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3305:   if(_r.has_torch_function()) {
3306:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3307:   }
3308:   switch (_r.idx) {
3309:     case 0: {
3310:       // [deprecated] aten::addmv_(Scalar beta, Tensor(a!) self, Scalar alpha, Tensor mat, Tensor vec) -> Tensor(a!)
3311: 
3312:       auto dispatch_addmv_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
3313:         pybind11::gil_scoped_release no_gil;
3314:         return self.addmv_(mat, vec, beta, alpha);
3315:       };
3316:       return wrap(dispatch_addmv_(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
3317:     }
3318:     case 1: {
3319:       // [deprecated] aten::addmv_(Scalar beta, Tensor(a!) self, Tensor mat, Tensor vec) -> Tensor(a!)
3320: 
3321:       auto dispatch_addmv_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec) -> at::Tensor {
3322:         pybind11::gil_scoped_release no_gil;
3323:         return self.addmv_(mat, vec, beta, 1);
3324:       };
3325:       return wrap(dispatch_addmv_(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
3326:     }
3327:     case 2: {
3328:       // aten::addmv_(Tensor(a!) self, Tensor mat, Tensor vec, *, Scalar beta=1, Scalar alpha=1) -> Tensor(a!)
3329: 
3330:       auto dispatch_addmv_ = [](const at::Tensor & self, const at::Tensor & mat, const at::Tensor & vec, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
3331:         pybind11::gil_scoped_release no_gil;
3332:         return self.addmv_(mat, vec, beta, alpha);
3333:       };
3334:       return wrap(dispatch_addmv_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
3335:     }
3336:   }
3337:   Py_RETURN_NONE;
3338:   END_HANDLE_TH_ERRORS
3339: }
3340: 
3341: \
3342: // addr
3343: static PyObject * THPVariable_addr(PyObject* self_, PyObject* args, PyObject* kwargs)
3344: {
3345:   HANDLE_TH_ERRORS
3346:   const Tensor& self = THPVariable_Unpack(self_);
3347:   static PythonArgParser parser({
3348:     "addr(Scalar beta, Scalar alpha, Tensor vec1, Tensor vec2)|deprecated",
3349:     "addr(Scalar beta, Tensor vec1, Tensor vec2)|deprecated",
3350:     "addr(Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1)",
3351:   }, /*traceable=*/true);
3352: 
3353:   ParsedArgs<4> parsed_args;
3354:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3355:   if(_r.has_torch_function()) {
3356:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3357:   }
3358:   switch (_r.idx) {
3359:     case 0: {
3360:       // [deprecated] aten::addr(Scalar beta, Tensor self, Scalar alpha, Tensor vec1, Tensor vec2) -> Tensor
```

- EN: The main execution path in this span is carried by `THPVariable_addmv`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_addmv`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3361-3480

```cpp
3361: 
3362:       auto dispatch_addr = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
3363:         pybind11::gil_scoped_release no_gil;
3364:         return self.addr(vec1, vec2, beta, alpha);
3365:       };
3366:       return wrap(dispatch_addr(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
3367:     }
3368:     case 1: {
3369:       // [deprecated] aten::addr(Scalar beta, Tensor self, Tensor vec1, Tensor vec2) -> Tensor
3370: 
3371:       auto dispatch_addr = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
3372:         pybind11::gil_scoped_release no_gil;
3373:         return self.addr(vec1, vec2, beta, 1);
3374:       };
3375:       return wrap(dispatch_addr(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
3376:     }
3377:     case 2: {
3378:       // aten::addr(Tensor self, Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
3379: 
3380:       auto dispatch_addr = [](const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
3381:         pybind11::gil_scoped_release no_gil;
3382:         return self.addr(vec1, vec2, beta, alpha);
3383:       };
3384:       return wrap(dispatch_addr(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
3385:     }
3386:   }
3387:   Py_RETURN_NONE;
3388:   END_HANDLE_TH_ERRORS
3389: }
3390: 
3391: \
3392: // addr_
3393: static PyObject * THPVariable_addr_(PyObject* self_, PyObject* args, PyObject* kwargs)
3394: {
3395:   HANDLE_TH_ERRORS
3396:   const Tensor& self = THPVariable_Unpack(self_);
3397:   static PythonArgParser parser({
3398:     "addr_(Scalar beta, Scalar alpha, Tensor vec1, Tensor vec2)|deprecated",
3399:     "addr_(Scalar beta, Tensor vec1, Tensor vec2)|deprecated",
3400:     "addr_(Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1)",
3401:   }, /*traceable=*/true);
3402: 
3403:   ParsedArgs<4> parsed_args;
3404:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3405:   if(_r.has_torch_function()) {
3406:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3407:   }
3408:   switch (_r.idx) {
3409:     case 0: {
3410:       // [deprecated] aten::addr_(Scalar beta, Tensor(a!) self, Scalar alpha, Tensor vec1, Tensor vec2) -> Tensor(a!)
3411: 
3412:       auto dispatch_addr_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
3413:         pybind11::gil_scoped_release no_gil;
3414:         return self.addr_(vec1, vec2, beta, alpha);
3415:       };
3416:       return wrap(dispatch_addr_(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
3417:     }
3418:     case 1: {
3419:       // [deprecated] aten::addr_(Scalar beta, Tensor(a!) self, Tensor vec1, Tensor vec2) -> Tensor(a!)
3420: 
3421:       auto dispatch_addr_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2) -> at::Tensor {
3422:         pybind11::gil_scoped_release no_gil;
3423:         return self.addr_(vec1, vec2, beta, 1);
3424:       };
3425:       return wrap(dispatch_addr_(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
3426:     }
3427:     case 2: {
3428:       // aten::addr_(Tensor(a!) self, Tensor vec1, Tensor vec2, *, Scalar beta=1, Scalar alpha=1) -> Tensor(a!)
3429: 
3430:       auto dispatch_addr_ = [](const at::Tensor & self, const at::Tensor & vec1, const at::Tensor & vec2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
3431:         pybind11::gil_scoped_release no_gil;
3432:         return self.addr_(vec1, vec2, beta, alpha);
3433:       };
3434:       return wrap(dispatch_addr_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
3435:     }
3436:   }
3437:   Py_RETURN_NONE;
3438:   END_HANDLE_TH_ERRORS
3439: }
3440: 
3441: // adjoint
3442: static PyObject * THPVariable_adjoint(PyObject* self_, PyObject* args)
3443: {
3444:   HANDLE_TH_ERRORS
3445:   const Tensor& self = THPVariable_Unpack(self_);
3446:   if (has_torch_function(self_)) {
3447:     return handle_torch_function(self_, "adjoint");
3448:   }
3449:   // aten::adjoint(Tensor(a) self) -> Tensor(a)
3450: 
3451:   auto dispatch_adjoint = [](const at::Tensor & self) -> at::Tensor {
3452:     pybind11::gil_scoped_release no_gil;
3453:     return self.adjoint();
3454:   };
3455:   return wrap(dispatch_adjoint(self));
3456:   END_HANDLE_TH_ERRORS
3457: }
3458: 
3459: // align_as
3460: static PyObject * THPVariable_align_as(PyObject* self_, PyObject* args, PyObject* kwargs)
3461: {
3462:   HANDLE_TH_ERRORS
3463:   const Tensor& self = THPVariable_Unpack(self_);
3464:   static PythonArgParser parser({
3465:     "align_as(Tensor other)",
3466:   }, /*traceable=*/true);
3467: 
3468:   ParsedArgs<1> parsed_args;
3469:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3470:   if(_r.has_torch_function()) {
3471:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3472:   }
3473:   // aten::align_as(Tensor self, Tensor other) -> Tensor
3474: 
3475:   auto dispatch_align_as = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3476:     pybind11::gil_scoped_release no_gil;
3477:     return self.align_as(other);
3478:   };
3479:   return wrap(dispatch_align_as(self, _r.tensor(0)));
3480:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `wrap`, `addr`, `THPVariable_addr_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `addr`, `THPVariable_addr_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3481-3600

```cpp
3481:   END_HANDLE_TH_ERRORS
3482: }
3483: 
3484: \
3485: // align_to
3486: static PyObject * THPVariable_align_to(PyObject* self_, PyObject* args, PyObject* kwargs)
3487: {
3488:   HANDLE_TH_ERRORS
3489:   const Tensor& self = THPVariable_Unpack(self_);
3490:   static PythonArgParser parser({
3491:     "align_to(DimnameList names)",
3492:     "align_to(DimnameList order, int64_t ellipsis_idx)",
3493:   }, /*traceable=*/true);
3494: 
3495:   ParsedArgs<2> parsed_args;
3496:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3497:   if(_r.has_torch_function()) {
3498:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3499:   }
3500:   switch (_r.idx) {
3501:     case 0: {
3502:       // aten::align_to(Tensor(a) self, Dimname[] names) -> Tensor(a)
3503: 
3504:       auto dispatch_align_to = [](const at::Tensor & self, at::DimnameList names) -> at::Tensor {
3505:         pybind11::gil_scoped_release no_gil;
3506:         return self.align_to(names);
3507:       };
3508:       return wrap(dispatch_align_to(self, _r.dimnamelist(0)));
3509:     }
3510:     case 1: {
3511:       // aten::align_to.ellipsis_idx(Tensor(a) self, Dimname[] order, int ellipsis_idx) -> Tensor(a)
3512: 
3513:       auto dispatch_align_to = [](const at::Tensor & self, at::DimnameList order, int64_t ellipsis_idx) -> at::Tensor {
3514:         pybind11::gil_scoped_release no_gil;
3515:         return self.align_to(order, ellipsis_idx);
3516:       };
3517:       return wrap(dispatch_align_to(self, _r.dimnamelist(0), _r.toInt64(1)));
3518:     }
3519:   }
3520:   Py_RETURN_NONE;
3521:   END_HANDLE_TH_ERRORS
3522: }
3523: 
3524: \
3525: // all
3526: static PyObject * THPVariable_all(PyObject* self_, PyObject* args, PyObject* kwargs)
3527: {
3528:   HANDLE_TH_ERRORS
3529:   const Tensor& self = THPVariable_Unpack(self_);
3530:   static PythonArgParser parser({
3531:     "all()",
3532:     "all(IntArrayRef? dim=None, bool keepdim=False)",
3533:     "all(int64_t dim, bool keepdim=False)",
3534:     "all(Dimname dim, bool keepdim=False)",
3535:   }, /*traceable=*/true);
3536: 
3537:   ParsedArgs<2> parsed_args;
3538:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3539:   if(_r.has_torch_function()) {
3540:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3541:   }
3542:   switch (_r.idx) {
3543:     case 0: {
3544:       // aten::all(Tensor self) -> Tensor
3545: 
3546:       auto dispatch_all = [](const at::Tensor & self) -> at::Tensor {
3547:         pybind11::gil_scoped_release no_gil;
3548:         return self.all();
3549:       };
3550:       return wrap(dispatch_all(self));
3551:     }
3552:     case 1: {
3553:       // aten::all.dims(Tensor self, int[]? dim=None, bool keepdim=False) -> Tensor
3554: 
3555:       auto dispatch_all = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim) -> at::Tensor {
3556:         pybind11::gil_scoped_release no_gil;
3557:         return self.all(dim, keepdim);
3558:       };
3559:       return wrap(dispatch_all(self, _r.intlistOptional(0), _r.toBool(1)));
3560:     }
3561:     case 2: {
3562:       // aten::all.dim(Tensor self, int dim, bool keepdim=False) -> Tensor
3563: 
3564:       auto dispatch_all = [](const at::Tensor & self, int64_t dim, bool keepdim) -> at::Tensor {
3565:         pybind11::gil_scoped_release no_gil;
3566:         return self.all(dim, keepdim);
3567:       };
3568:       return wrap(dispatch_all(self, _r.toInt64(0), _r.toBool(1)));
3569:     }
3570:     case 3: {
3571:       // aten::all.dimname(Tensor self, Dimname dim, bool keepdim=False) -> Tensor
3572: 
3573:       auto dispatch_all = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> at::Tensor {
3574:         pybind11::gil_scoped_release no_gil;
3575:         return self.all(dim, keepdim);
3576:       };
3577:       return wrap(dispatch_all(self, _r.dimname(0), _r.toBool(1)));
3578:     }
3579:   }
3580:   Py_RETURN_NONE;
3581:   END_HANDLE_TH_ERRORS
3582: }
3583: 
3584: // allclose
3585: static PyObject * THPVariable_allclose(PyObject* self_, PyObject* args, PyObject* kwargs)
3586: {
3587:   HANDLE_TH_ERRORS
3588:   const Tensor& self = THPVariable_Unpack(self_);
3589:   static PythonArgParser parser({
3590:     "allclose(Tensor other, double rtol=1e-05, double atol=1e-08, bool equal_nan=False)",
3591:   }, /*traceable=*/false);
3592: 
3593:   ParsedArgs<4> parsed_args;
3594:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3595:   if(_r.has_torch_function()) {
3596:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3597:   }
3598:   // aten::allclose(Tensor self, Tensor other, float rtol=1e-05, float atol=1e-08, bool equal_nan=False) -> bool
3599: 
3600:   auto dispatch_allclose = [](const at::Tensor & self, const at::Tensor & other, double rtol, double atol, bool equal_nan) -> bool {
```

- EN: The main execution path in this span is carried by `THPVariable_align_to`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_align_to`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3601-3720

```cpp
3601:     pybind11::gil_scoped_release no_gil;
3602:     return self.allclose(other, rtol, atol, equal_nan);
3603:   };
3604:   return wrap(dispatch_allclose(self, _r.tensor(0), _r.toDouble(1), _r.toDouble(2), _r.toBool(3)));
3605:   Py_RETURN_NONE;
3606:   END_HANDLE_TH_ERRORS
3607: }
3608: 
3609: // amax
3610: static PyObject * THPVariable_amax(PyObject* self_, PyObject* args, PyObject* kwargs)
3611: {
3612:   HANDLE_TH_ERRORS
3613:   const Tensor& self = THPVariable_Unpack(self_);
3614:   static PythonArgParser parser({
3615:     "amax(IntArrayRef[1] dim=None, bool keepdim=False)",
3616:   }, /*traceable=*/true);
3617: 
3618:   ParsedArgs<2> parsed_args;
3619:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3620:   if(_r.has_torch_function()) {
3621:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3622:   }
3623:   // aten::amax(Tensor self, int[1] dim=[], bool keepdim=False) -> Tensor
3624: 
3625:   auto dispatch_amax = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
3626:     pybind11::gil_scoped_release no_gil;
3627:     return self.amax(dim, keepdim);
3628:   };
3629:   return wrap(dispatch_amax(self, _r.intlist(0), _r.toBool(1)));
3630:   Py_RETURN_NONE;
3631:   END_HANDLE_TH_ERRORS
3632: }
3633: 
3634: // amin
3635: static PyObject * THPVariable_amin(PyObject* self_, PyObject* args, PyObject* kwargs)
3636: {
3637:   HANDLE_TH_ERRORS
3638:   const Tensor& self = THPVariable_Unpack(self_);
3639:   static PythonArgParser parser({
3640:     "amin(IntArrayRef[1] dim=None, bool keepdim=False)",
3641:   }, /*traceable=*/true);
3642: 
3643:   ParsedArgs<2> parsed_args;
3644:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3645:   if(_r.has_torch_function()) {
3646:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3647:   }
3648:   // aten::amin(Tensor self, int[1] dim=[], bool keepdim=False) -> Tensor
3649: 
3650:   auto dispatch_amin = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
3651:     pybind11::gil_scoped_release no_gil;
3652:     return self.amin(dim, keepdim);
3653:   };
3654:   return wrap(dispatch_amin(self, _r.intlist(0), _r.toBool(1)));
3655:   Py_RETURN_NONE;
3656:   END_HANDLE_TH_ERRORS
3657: }
3658: 
3659: // aminmax
3660: static PyObject * THPVariable_aminmax(PyObject* self_, PyObject* args, PyObject* kwargs)
3661: {
3662:   HANDLE_TH_ERRORS
3663:   static PyTypeObject* NamedTuple = generated::get_aminmax_structseq();
3664:   const Tensor& self = THPVariable_Unpack(self_);
3665:   static PythonArgParser parser({
3666:     "aminmax(*, int64_t? dim=None, bool keepdim=False)",
3667:   }, /*traceable=*/true);
3668: 
3669:   ParsedArgs<2> parsed_args;
3670:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3671:   if(_r.has_torch_function()) {
3672:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3673:   }
3674:   // aten::aminmax(Tensor self, *, int? dim=None, bool keepdim=False) -> (Tensor min, Tensor max)
3675: 
3676:   auto dispatch_aminmax = [](const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
3677:     pybind11::gil_scoped_release no_gil;
3678:     return self.aminmax(dim, keepdim);
3679:   };
3680:   return wrap(NamedTuple, dispatch_aminmax(self, _r.toInt64Optional(0), _r.toBool(1)));
3681:   Py_RETURN_NONE;
3682:   END_HANDLE_TH_ERRORS
3683: }
3684: 
3685: // angle
3686: static PyObject * THPVariable_angle(PyObject* self_, PyObject* args)
3687: {
3688:   HANDLE_TH_ERRORS
3689:   const Tensor& self = THPVariable_Unpack(self_);
3690:   if (has_torch_function(self_)) {
3691:     return handle_torch_function(self_, "angle");
3692:   }
3693:   // aten::angle(Tensor self) -> Tensor
3694: 
3695:   auto dispatch_angle = [](const at::Tensor & self) -> at::Tensor {
3696:     pybind11::gil_scoped_release no_gil;
3697:     return self.angle();
3698:   };
3699:   return wrap(dispatch_angle(self));
3700:   END_HANDLE_TH_ERRORS
3701: }
3702: 
3703: \
3704: // any
3705: static PyObject * THPVariable_any(PyObject* self_, PyObject* args, PyObject* kwargs)
3706: {
3707:   HANDLE_TH_ERRORS
3708:   const Tensor& self = THPVariable_Unpack(self_);
3709:   static PythonArgParser parser({
3710:     "any()",
3711:     "any(IntArrayRef? dim=None, bool keepdim=False)",
3712:     "any(int64_t dim, bool keepdim=False)",
3713:     "any(Dimname dim, bool keepdim=False)",
3714:   }, /*traceable=*/true);
3715: 
3716:   ParsedArgs<2> parsed_args;
3717:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3718:   if(_r.has_torch_function()) {
3719:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3720:   }
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_amax`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_amax`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3721-3840

```cpp
3721:   switch (_r.idx) {
3722:     case 0: {
3723:       // aten::any(Tensor self) -> Tensor
3724: 
3725:       auto dispatch_any = [](const at::Tensor & self) -> at::Tensor {
3726:         pybind11::gil_scoped_release no_gil;
3727:         return self.any();
3728:       };
3729:       return wrap(dispatch_any(self));
3730:     }
3731:     case 1: {
3732:       // aten::any.dims(Tensor self, int[]? dim=None, bool keepdim=False) -> Tensor
3733: 
3734:       auto dispatch_any = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim) -> at::Tensor {
3735:         pybind11::gil_scoped_release no_gil;
3736:         return self.any(dim, keepdim);
3737:       };
3738:       return wrap(dispatch_any(self, _r.intlistOptional(0), _r.toBool(1)));
3739:     }
3740:     case 2: {
3741:       // aten::any.dim(Tensor self, int dim, bool keepdim=False) -> Tensor
3742: 
3743:       auto dispatch_any = [](const at::Tensor & self, int64_t dim, bool keepdim) -> at::Tensor {
3744:         pybind11::gil_scoped_release no_gil;
3745:         return self.any(dim, keepdim);
3746:       };
3747:       return wrap(dispatch_any(self, _r.toInt64(0), _r.toBool(1)));
3748:     }
3749:     case 3: {
3750:       // aten::any.dimname(Tensor self, Dimname dim, bool keepdim=False) -> Tensor
3751: 
3752:       auto dispatch_any = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> at::Tensor {
3753:         pybind11::gil_scoped_release no_gil;
3754:         return self.any(dim, keepdim);
3755:       };
3756:       return wrap(dispatch_any(self, _r.dimname(0), _r.toBool(1)));
3757:     }
3758:   }
3759:   Py_RETURN_NONE;
3760:   END_HANDLE_TH_ERRORS
3761: }
3762: 
3763: // arccos
3764: static PyObject * THPVariable_arccos(PyObject* self_, PyObject* args)
3765: {
3766:   HANDLE_TH_ERRORS
3767:   const Tensor& self = THPVariable_Unpack(self_);
3768:   if (has_torch_function(self_)) {
3769:     return handle_torch_function(self_, "arccos");
3770:   }
3771:   // aten::arccos(Tensor self) -> Tensor
3772: 
3773:   auto dispatch_arccos = [](const at::Tensor & self) -> at::Tensor {
3774:     pybind11::gil_scoped_release no_gil;
3775:     return self.arccos();
3776:   };
3777:   return wrap(dispatch_arccos(self));
3778:   END_HANDLE_TH_ERRORS
3779: }
3780: 
3781: // arccos_
3782: static PyObject * THPVariable_arccos_(PyObject* self_, PyObject* args)
3783: {
3784:   HANDLE_TH_ERRORS
3785:   const Tensor& self = THPVariable_Unpack(self_);
3786:   if (has_torch_function(self_)) {
3787:     return handle_torch_function(self_, "arccos_");
3788:   }
3789:   // aten::arccos_(Tensor(a!) self) -> Tensor(a!)
3790: 
3791:   auto dispatch_arccos_ = [](const at::Tensor & self) -> at::Tensor {
3792:     pybind11::gil_scoped_release no_gil;
3793:     return self.arccos_();
3794:   };
3795:   return wrap(dispatch_arccos_(self));
3796:   END_HANDLE_TH_ERRORS
3797: }
3798: 
3799: // arccosh
3800: static PyObject * THPVariable_arccosh(PyObject* self_, PyObject* args)
3801: {
3802:   HANDLE_TH_ERRORS
3803:   const Tensor& self = THPVariable_Unpack(self_);
3804:   if (has_torch_function(self_)) {
3805:     return handle_torch_function(self_, "arccosh");
3806:   }
3807:   // aten::arccosh(Tensor self) -> Tensor
3808: 
3809:   auto dispatch_arccosh = [](const at::Tensor & self) -> at::Tensor {
3810:     pybind11::gil_scoped_release no_gil;
3811:     return self.arccosh();
3812:   };
3813:   return wrap(dispatch_arccosh(self));
3814:   END_HANDLE_TH_ERRORS
3815: }
3816: 
3817: // arccosh_
3818: static PyObject * THPVariable_arccosh_(PyObject* self_, PyObject* args)
3819: {
3820:   HANDLE_TH_ERRORS
3821:   const Tensor& self = THPVariable_Unpack(self_);
3822:   if (has_torch_function(self_)) {
3823:     return handle_torch_function(self_, "arccosh_");
3824:   }
3825:   // aten::arccosh_(Tensor(a!) self) -> Tensor(a!)
3826: 
3827:   auto dispatch_arccosh_ = [](const at::Tensor & self) -> at::Tensor {
3828:     pybind11::gil_scoped_release no_gil;
3829:     return self.arccosh_();
3830:   };
3831:   return wrap(dispatch_arccosh_(self));
3832:   END_HANDLE_TH_ERRORS
3833: }
3834: 
3835: // arcsin
3836: static PyObject * THPVariable_arcsin(PyObject* self_, PyObject* args)
3837: {
3838:   HANDLE_TH_ERRORS
3839:   const Tensor& self = THPVariable_Unpack(self_);
3840:   if (has_torch_function(self_)) {
```

- EN: The main execution path in this span is carried by `any`, `wrap`, `THPVariable_arccos`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `any`, `wrap`, `THPVariable_arccos` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 3841-3960

```cpp
3841:     return handle_torch_function(self_, "arcsin");
3842:   }
3843:   // aten::arcsin(Tensor self) -> Tensor
3844: 
3845:   auto dispatch_arcsin = [](const at::Tensor & self) -> at::Tensor {
3846:     pybind11::gil_scoped_release no_gil;
3847:     return self.arcsin();
3848:   };
3849:   return wrap(dispatch_arcsin(self));
3850:   END_HANDLE_TH_ERRORS
3851: }
3852: 
3853: // arcsin_
3854: static PyObject * THPVariable_arcsin_(PyObject* self_, PyObject* args)
3855: {
3856:   HANDLE_TH_ERRORS
3857:   const Tensor& self = THPVariable_Unpack(self_);
3858:   if (has_torch_function(self_)) {
3859:     return handle_torch_function(self_, "arcsin_");
3860:   }
3861:   // aten::arcsin_(Tensor(a!) self) -> Tensor(a!)
3862: 
3863:   auto dispatch_arcsin_ = [](const at::Tensor & self) -> at::Tensor {
3864:     pybind11::gil_scoped_release no_gil;
3865:     return self.arcsin_();
3866:   };
3867:   return wrap(dispatch_arcsin_(self));
3868:   END_HANDLE_TH_ERRORS
3869: }
3870: 
3871: // arcsinh
3872: static PyObject * THPVariable_arcsinh(PyObject* self_, PyObject* args)
3873: {
3874:   HANDLE_TH_ERRORS
3875:   const Tensor& self = THPVariable_Unpack(self_);
3876:   if (has_torch_function(self_)) {
3877:     return handle_torch_function(self_, "arcsinh");
3878:   }
3879:   // aten::arcsinh(Tensor self) -> Tensor
3880: 
3881:   auto dispatch_arcsinh = [](const at::Tensor & self) -> at::Tensor {
3882:     pybind11::gil_scoped_release no_gil;
3883:     return self.arcsinh();
3884:   };
3885:   return wrap(dispatch_arcsinh(self));
3886:   END_HANDLE_TH_ERRORS
3887: }
3888: 
3889: // arcsinh_
3890: static PyObject * THPVariable_arcsinh_(PyObject* self_, PyObject* args)
3891: {
3892:   HANDLE_TH_ERRORS
3893:   const Tensor& self = THPVariable_Unpack(self_);
3894:   if (has_torch_function(self_)) {
3895:     return handle_torch_function(self_, "arcsinh_");
3896:   }
3897:   // aten::arcsinh_(Tensor(a!) self) -> Tensor(a!)
3898: 
3899:   auto dispatch_arcsinh_ = [](const at::Tensor & self) -> at::Tensor {
3900:     pybind11::gil_scoped_release no_gil;
3901:     return self.arcsinh_();
3902:   };
3903:   return wrap(dispatch_arcsinh_(self));
3904:   END_HANDLE_TH_ERRORS
3905: }
3906: 
3907: // arctan
3908: static PyObject * THPVariable_arctan(PyObject* self_, PyObject* args)
3909: {
3910:   HANDLE_TH_ERRORS
3911:   const Tensor& self = THPVariable_Unpack(self_);
3912:   if (has_torch_function(self_)) {
3913:     return handle_torch_function(self_, "arctan");
3914:   }
3915:   // aten::arctan(Tensor self) -> Tensor
3916: 
3917:   auto dispatch_arctan = [](const at::Tensor & self) -> at::Tensor {
3918:     pybind11::gil_scoped_release no_gil;
3919:     return self.arctan();
3920:   };
3921:   return wrap(dispatch_arctan(self));
3922:   END_HANDLE_TH_ERRORS
3923: }
3924: 
3925: // arctan2
3926: static PyObject * THPVariable_arctan2(PyObject* self_, PyObject* args, PyObject* kwargs)
3927: {
3928:   HANDLE_TH_ERRORS
3929:   const Tensor& self = THPVariable_Unpack(self_);
3930:   static PythonArgParser parser({
3931:     "arctan2(Tensor other)",
3932:   }, /*traceable=*/true);
3933: 
3934:   ParsedArgs<1> parsed_args;
3935:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
3936:   if(_r.has_torch_function()) {
3937:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3938:   }
3939:   // aten::arctan2(Tensor self, Tensor other) -> Tensor
3940: 
3941:   auto dispatch_arctan2 = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3942:     pybind11::gil_scoped_release no_gil;
3943:     return self.arctan2(other);
3944:   };
3945:   return wrap(dispatch_arctan2(self, _r.tensor(0)));
3946:   Py_RETURN_NONE;
3947:   END_HANDLE_TH_ERRORS
3948: }
3949: 
3950: // arctan2_
3951: static PyObject * THPVariable_arctan2_(PyObject* self_, PyObject* args, PyObject* kwargs)
3952: {
3953:   HANDLE_TH_ERRORS
3954:   const Tensor& self = THPVariable_Unpack(self_);
3955:   static PythonArgParser parser({
3956:     "arctan2_(Tensor other)",
3957:   }, /*traceable=*/true);
3958: 
3959:   ParsedArgs<1> parsed_args;
3960:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `arcsin`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `arcsin`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 3961-4080

```cpp
3961:   if(_r.has_torch_function()) {
3962:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
3963:   }
3964:   // aten::arctan2_(Tensor(a!) self, Tensor other) -> Tensor(a!)
3965: 
3966:   auto dispatch_arctan2_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
3967:     pybind11::gil_scoped_release no_gil;
3968:     return self.arctan2_(other);
3969:   };
3970:   return wrap(dispatch_arctan2_(self, _r.tensor(0)));
3971:   Py_RETURN_NONE;
3972:   END_HANDLE_TH_ERRORS
3973: }
3974: 
3975: // arctan_
3976: static PyObject * THPVariable_arctan_(PyObject* self_, PyObject* args)
3977: {
3978:   HANDLE_TH_ERRORS
3979:   const Tensor& self = THPVariable_Unpack(self_);
3980:   if (has_torch_function(self_)) {
3981:     return handle_torch_function(self_, "arctan_");
3982:   }
3983:   // aten::arctan_(Tensor(a!) self) -> Tensor(a!)
3984: 
3985:   auto dispatch_arctan_ = [](const at::Tensor & self) -> at::Tensor {
3986:     pybind11::gil_scoped_release no_gil;
3987:     return self.arctan_();
3988:   };
3989:   return wrap(dispatch_arctan_(self));
3990:   END_HANDLE_TH_ERRORS
3991: }
3992: 
3993: // arctanh
3994: static PyObject * THPVariable_arctanh(PyObject* self_, PyObject* args)
3995: {
3996:   HANDLE_TH_ERRORS
3997:   const Tensor& self = THPVariable_Unpack(self_);
3998:   if (has_torch_function(self_)) {
3999:     return handle_torch_function(self_, "arctanh");
4000:   }
4001:   // aten::arctanh(Tensor self) -> Tensor
4002: 
4003:   auto dispatch_arctanh = [](const at::Tensor & self) -> at::Tensor {
4004:     pybind11::gil_scoped_release no_gil;
4005:     return self.arctanh();
4006:   };
4007:   return wrap(dispatch_arctanh(self));
4008:   END_HANDLE_TH_ERRORS
4009: }
4010: 
4011: // arctanh_
4012: static PyObject * THPVariable_arctanh_(PyObject* self_, PyObject* args)
4013: {
4014:   HANDLE_TH_ERRORS
4015:   const Tensor& self = THPVariable_Unpack(self_);
4016:   if (has_torch_function(self_)) {
4017:     return handle_torch_function(self_, "arctanh_");
4018:   }
4019:   // aten::arctanh_(Tensor(a!) self) -> Tensor(a!)
4020: 
4021:   auto dispatch_arctanh_ = [](const at::Tensor & self) -> at::Tensor {
4022:     pybind11::gil_scoped_release no_gil;
4023:     return self.arctanh_();
4024:   };
4025:   return wrap(dispatch_arctanh_(self));
4026:   END_HANDLE_TH_ERRORS
4027: }
4028: 
4029: // argmax
4030: static PyObject * THPVariable_argmax(PyObject* self_, PyObject* args, PyObject* kwargs)
4031: {
4032:   HANDLE_TH_ERRORS
4033:   const Tensor& self = THPVariable_Unpack(self_);
4034:   static PythonArgParser parser({
4035:     "argmax(int64_t? dim=None, bool keepdim=False)",
4036:   }, /*traceable=*/true);
4037: 
4038:   ParsedArgs<2> parsed_args;
4039:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4040:   if(_r.has_torch_function()) {
4041:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4042:   }
4043:   // aten::argmax(Tensor self, int? dim=None, bool keepdim=False) -> Tensor
4044: 
4045:   auto dispatch_argmax = [](const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> at::Tensor {
4046:     pybind11::gil_scoped_release no_gil;
4047:     return self.argmax(dim, keepdim);
4048:   };
4049:   return wrap(dispatch_argmax(self, _r.toInt64Optional(0), _r.toBool(1)));
4050:   Py_RETURN_NONE;
4051:   END_HANDLE_TH_ERRORS
4052: }
4053: 
4054: // argmin
4055: static PyObject * THPVariable_argmin(PyObject* self_, PyObject* args, PyObject* kwargs)
4056: {
4057:   HANDLE_TH_ERRORS
4058:   const Tensor& self = THPVariable_Unpack(self_);
4059:   static PythonArgParser parser({
4060:     "argmin(int64_t? dim=None, bool keepdim=False)",
4061:   }, /*traceable=*/true);
4062: 
4063:   ParsedArgs<2> parsed_args;
4064:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4065:   if(_r.has_torch_function()) {
4066:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4067:   }
4068:   // aten::argmin(Tensor self, int? dim=None, bool keepdim=False) -> Tensor
4069: 
4070:   auto dispatch_argmin = [](const at::Tensor & self, ::std::optional<int64_t> dim, bool keepdim) -> at::Tensor {
4071:     pybind11::gil_scoped_release no_gil;
4072:     return self.argmin(dim, keepdim);
4073:   };
4074:   return wrap(dispatch_argmin(self, _r.toInt64Optional(0), _r.toBool(1)));
4075:   Py_RETURN_NONE;
4076:   END_HANDLE_TH_ERRORS
4077: }
4078: 
4079: \
4080: // argsort
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `arctan2_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `arctan2_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4081-4200

```cpp
4081: static PyObject * THPVariable_argsort(PyObject* self_, PyObject* args, PyObject* kwargs)
4082: {
4083:   HANDLE_TH_ERRORS
4084:   const Tensor& self = THPVariable_Unpack(self_);
4085:   static PythonArgParser parser({
4086:     "argsort(*, bool stable, int64_t dim=-1, bool descending=False)",
4087:     "argsort(int64_t dim=-1, bool descending=False)",
4088:     "argsort(Dimname dim, bool descending=False)",
4089:   }, /*traceable=*/true);
4090: 
4091:   ParsedArgs<3> parsed_args;
4092:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4093:   if(_r.has_torch_function()) {
4094:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4095:   }
4096:   switch (_r.idx) {
4097:     case 0: {
4098:       // aten::argsort.stable(Tensor self, *, bool stable, int dim=-1, bool descending=False) -> Tensor
4099: 
4100:       auto dispatch_argsort = [](const at::Tensor & self, bool stable, int64_t dim, bool descending) -> at::Tensor {
4101:         pybind11::gil_scoped_release no_gil;
4102:         return self.argsort(stable, dim, descending);
4103:       };
4104:       return wrap(dispatch_argsort(self, _r.toBool(0), _r.toInt64(1), _r.toBool(2)));
4105:     }
4106:     case 1: {
4107:       // aten::argsort(Tensor self, int dim=-1, bool descending=False) -> Tensor
4108: 
4109:       auto dispatch_argsort = [](const at::Tensor & self, int64_t dim, bool descending) -> at::Tensor {
4110:         pybind11::gil_scoped_release no_gil;
4111:         return self.argsort(dim, descending);
4112:       };
4113:       return wrap(dispatch_argsort(self, _r.toInt64(0), _r.toBool(1)));
4114:     }
4115:     case 2: {
4116:       // aten::argsort.dimname(Tensor self, Dimname dim, bool descending=False) -> Tensor
4117: 
4118:       auto dispatch_argsort = [](const at::Tensor & self, at::Dimname dim, bool descending) -> at::Tensor {
4119:         pybind11::gil_scoped_release no_gil;
4120:         return self.argsort(dim, descending);
4121:       };
4122:       return wrap(dispatch_argsort(self, _r.dimname(0), _r.toBool(1)));
4123:     }
4124:   }
4125:   Py_RETURN_NONE;
4126:   END_HANDLE_TH_ERRORS
4127: }
4128: 
4129: // argwhere
4130: static PyObject * THPVariable_argwhere(PyObject* self_, PyObject* args)
4131: {
4132:   HANDLE_TH_ERRORS
4133:   const Tensor& self = THPVariable_Unpack(self_);
4134:   if (has_torch_function(self_)) {
4135:     return handle_torch_function(self_, "argwhere");
4136:   }
4137:   // aten::argwhere(Tensor self) -> Tensor
4138: 
4139:   auto dispatch_argwhere = [](const at::Tensor & self) -> at::Tensor {
4140:     pybind11::gil_scoped_release no_gil;
4141:     return self.argwhere();
4142:   };
4143:   return wrap(dispatch_argwhere(self));
4144:   END_HANDLE_TH_ERRORS
4145: }
4146: 
4147: // as_strided
4148: static PyObject * THPVariable_as_strided(PyObject* self_, PyObject* args, PyObject* kwargs)
4149: {
4150:   HANDLE_TH_ERRORS
4151:   const Tensor& self = THPVariable_Unpack(self_);
4152:   static PythonArgParser parser({
4153:     "as_strided(SymIntArrayRef size, SymIntArrayRef stride, SymInt? storage_offset=None)",
4154:   }, /*traceable=*/true);
4155: 
4156:   ParsedArgs<3> parsed_args;
4157:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4158:   if(_r.has_torch_function()) {
4159:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4160:   }
4161:   // aten::as_strided(Tensor(a) self, SymInt[] size, SymInt[] stride, SymInt? storage_offset=None) -> Tensor(a)
4162: 
4163:   auto dispatch_as_strided = [](const at::Tensor & self, c10::SymIntArrayRef size, c10::SymIntArrayRef stride, ::std::optional<c10::SymInt> storage_offset) -> at::Tensor {
4164:     pybind11::gil_scoped_release no_gil;
4165:     return self.as_strided_symint(size, stride, storage_offset);
4166:   };
4167:   return wrap(dispatch_as_strided(self, _r.symintlist(0), _r.symintlist(1), _r.toSymIntOptional(2)));
4168:   Py_RETURN_NONE;
4169:   END_HANDLE_TH_ERRORS
4170: }
4171: 
4172: // as_strided_
4173: static PyObject * THPVariable_as_strided_(PyObject* self_, PyObject* args, PyObject* kwargs)
4174: {
4175:   HANDLE_TH_ERRORS
4176:   const Tensor& self = THPVariable_Unpack(self_);
4177:   static PythonArgParser parser({
4178:     "as_strided_(SymIntArrayRef size, SymIntArrayRef stride, SymInt? storage_offset=None)",
4179:   }, /*traceable=*/true);
4180: 
4181:   ParsedArgs<3> parsed_args;
4182:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4183:   if(_r.has_torch_function()) {
4184:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4185:   }
4186:   // aten::as_strided_(Tensor(a!) self, SymInt[] size, SymInt[] stride, SymInt? storage_offset=None) -> Tensor(a!)
4187: 
4188:   auto dispatch_as_strided_ = [](const at::Tensor & self, c10::SymIntArrayRef size, c10::SymIntArrayRef stride, ::std::optional<c10::SymInt> storage_offset) -> at::Tensor {
4189:     pybind11::gil_scoped_release no_gil;
4190:     return self.as_strided__symint(size, stride, storage_offset);
4191:   };
4192:   return wrap(dispatch_as_strided_(self, _r.symintlist(0), _r.symintlist(1), _r.toSymIntOptional(2)));
4193:   Py_RETURN_NONE;
4194:   END_HANDLE_TH_ERRORS
4195: }
4196: 
4197: // as_strided_scatter
4198: static PyObject * THPVariable_as_strided_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
4199: {
4200:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_argsort`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_argsort`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4201-4320

```cpp
4201:   const Tensor& self = THPVariable_Unpack(self_);
4202:   static PythonArgParser parser({
4203:     "as_strided_scatter(Tensor src, SymIntArrayRef size, SymIntArrayRef stride, SymInt? storage_offset=None)",
4204:   }, /*traceable=*/true);
4205: 
4206:   ParsedArgs<4> parsed_args;
4207:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4208:   if(_r.has_torch_function()) {
4209:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4210:   }
4211:   // aten::as_strided_scatter(Tensor self, Tensor src, SymInt[] size, SymInt[] stride, SymInt? storage_offset=None) -> Tensor
4212: 
4213:   auto dispatch_as_strided_scatter = [](const at::Tensor & self, const at::Tensor & src, c10::SymIntArrayRef size, c10::SymIntArrayRef stride, ::std::optional<c10::SymInt> storage_offset) -> at::Tensor {
4214:     pybind11::gil_scoped_release no_gil;
4215:     return self.as_strided_scatter_symint(src, size, stride, storage_offset);
4216:   };
4217:   return wrap(dispatch_as_strided_scatter(self, _r.tensor(0), _r.symintlist(1), _r.symintlist(2), _r.toSymIntOptional(3)));
4218:   Py_RETURN_NONE;
4219:   END_HANDLE_TH_ERRORS
4220: }
4221: 
4222: // asin
4223: static PyObject * THPVariable_asin(PyObject* self_, PyObject* args)
4224: {
4225:   HANDLE_TH_ERRORS
4226:   const Tensor& self = THPVariable_Unpack(self_);
4227:   if (has_torch_function(self_)) {
4228:     return handle_torch_function(self_, "asin");
4229:   }
4230:   // aten::asin(Tensor self) -> Tensor
4231: 
4232:   auto dispatch_asin = [](const at::Tensor & self) -> at::Tensor {
4233:     pybind11::gil_scoped_release no_gil;
4234:     return self.asin();
4235:   };
4236:   return wrap(dispatch_asin(self));
4237:   END_HANDLE_TH_ERRORS
4238: }
4239: 
4240: // asin_
4241: static PyObject * THPVariable_asin_(PyObject* self_, PyObject* args)
4242: {
4243:   HANDLE_TH_ERRORS
4244:   const Tensor& self = THPVariable_Unpack(self_);
4245:   if (has_torch_function(self_)) {
4246:     return handle_torch_function(self_, "asin_");
4247:   }
4248:   // aten::asin_(Tensor(a!) self) -> Tensor(a!)
4249: 
4250:   auto dispatch_asin_ = [](const at::Tensor & self) -> at::Tensor {
4251:     pybind11::gil_scoped_release no_gil;
4252:     return self.asin_();
4253:   };
4254:   return wrap(dispatch_asin_(self));
4255:   END_HANDLE_TH_ERRORS
4256: }
4257: 
4258: // asinh
4259: static PyObject * THPVariable_asinh(PyObject* self_, PyObject* args)
4260: {
4261:   HANDLE_TH_ERRORS
4262:   const Tensor& self = THPVariable_Unpack(self_);
4263:   if (has_torch_function(self_)) {
4264:     return handle_torch_function(self_, "asinh");
4265:   }
4266:   // aten::asinh(Tensor self) -> Tensor
4267: 
4268:   auto dispatch_asinh = [](const at::Tensor & self) -> at::Tensor {
4269:     pybind11::gil_scoped_release no_gil;
4270:     return self.asinh();
4271:   };
4272:   return wrap(dispatch_asinh(self));
4273:   END_HANDLE_TH_ERRORS
4274: }
4275: 
4276: // asinh_
4277: static PyObject * THPVariable_asinh_(PyObject* self_, PyObject* args)
4278: {
4279:   HANDLE_TH_ERRORS
4280:   const Tensor& self = THPVariable_Unpack(self_);
4281:   if (has_torch_function(self_)) {
4282:     return handle_torch_function(self_, "asinh_");
4283:   }
4284:   // aten::asinh_(Tensor(a!) self) -> Tensor(a!)
4285: 
4286:   auto dispatch_asinh_ = [](const at::Tensor & self) -> at::Tensor {
4287:     pybind11::gil_scoped_release no_gil;
4288:     return self.asinh_();
4289:   };
4290:   return wrap(dispatch_asinh_(self));
4291:   END_HANDLE_TH_ERRORS
4292: }
4293: 
4294: // atan
4295: static PyObject * THPVariable_atan(PyObject* self_, PyObject* args)
4296: {
4297:   HANDLE_TH_ERRORS
4298:   const Tensor& self = THPVariable_Unpack(self_);
4299:   if (has_torch_function(self_)) {
4300:     return handle_torch_function(self_, "atan");
4301:   }
4302:   // aten::atan(Tensor self) -> Tensor
4303: 
4304:   auto dispatch_atan = [](const at::Tensor & self) -> at::Tensor {
4305:     pybind11::gil_scoped_release no_gil;
4306:     return self.atan();
4307:   };
4308:   return wrap(dispatch_atan(self));
4309:   END_HANDLE_TH_ERRORS
4310: }
4311: 
4312: // atan2
4313: static PyObject * THPVariable_atan2(PyObject* self_, PyObject* args, PyObject* kwargs)
4314: {
4315:   HANDLE_TH_ERRORS
4316:   const Tensor& self = THPVariable_Unpack(self_);
4317:   static PythonArgParser parser({
4318:     "atan2(Tensor other)",
4319:   }, /*traceable=*/true);
4320: 
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 4321-4440

```cpp
4321:   ParsedArgs<1> parsed_args;
4322:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4323:   if(_r.has_torch_function()) {
4324:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4325:   }
4326:   // aten::atan2(Tensor self, Tensor other) -> Tensor
4327: 
4328:   auto dispatch_atan2 = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4329:     pybind11::gil_scoped_release no_gil;
4330:     return self.atan2(other);
4331:   };
4332:   return wrap(dispatch_atan2(self, _r.tensor(0)));
4333:   Py_RETURN_NONE;
4334:   END_HANDLE_TH_ERRORS
4335: }
4336: 
4337: // atan2_
4338: static PyObject * THPVariable_atan2_(PyObject* self_, PyObject* args, PyObject* kwargs)
4339: {
4340:   HANDLE_TH_ERRORS
4341:   const Tensor& self = THPVariable_Unpack(self_);
4342:   static PythonArgParser parser({
4343:     "atan2_(Tensor other)",
4344:   }, /*traceable=*/true);
4345: 
4346:   ParsedArgs<1> parsed_args;
4347:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4348:   if(_r.has_torch_function()) {
4349:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4350:   }
4351:   // aten::atan2_(Tensor(a!) self, Tensor other) -> Tensor(a!)
4352: 
4353:   auto dispatch_atan2_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4354:     pybind11::gil_scoped_release no_gil;
4355:     return self.atan2_(other);
4356:   };
4357:   return wrap(dispatch_atan2_(self, _r.tensor(0)));
4358:   Py_RETURN_NONE;
4359:   END_HANDLE_TH_ERRORS
4360: }
4361: 
4362: // atan_
4363: static PyObject * THPVariable_atan_(PyObject* self_, PyObject* args)
4364: {
4365:   HANDLE_TH_ERRORS
4366:   const Tensor& self = THPVariable_Unpack(self_);
4367:   if (has_torch_function(self_)) {
4368:     return handle_torch_function(self_, "atan_");
4369:   }
4370:   // aten::atan_(Tensor(a!) self) -> Tensor(a!)
4371: 
4372:   auto dispatch_atan_ = [](const at::Tensor & self) -> at::Tensor {
4373:     pybind11::gil_scoped_release no_gil;
4374:     return self.atan_();
4375:   };
4376:   return wrap(dispatch_atan_(self));
4377:   END_HANDLE_TH_ERRORS
4378: }
4379: 
4380: // atanh
4381: static PyObject * THPVariable_atanh(PyObject* self_, PyObject* args)
4382: {
4383:   HANDLE_TH_ERRORS
4384:   const Tensor& self = THPVariable_Unpack(self_);
4385:   if (has_torch_function(self_)) {
4386:     return handle_torch_function(self_, "atanh");
4387:   }
4388:   // aten::atanh(Tensor self) -> Tensor
4389: 
4390:   auto dispatch_atanh = [](const at::Tensor & self) -> at::Tensor {
4391:     pybind11::gil_scoped_release no_gil;
4392:     return self.atanh();
4393:   };
4394:   return wrap(dispatch_atanh(self));
4395:   END_HANDLE_TH_ERRORS
4396: }
4397: 
4398: // atanh_
4399: static PyObject * THPVariable_atanh_(PyObject* self_, PyObject* args)
4400: {
4401:   HANDLE_TH_ERRORS
4402:   const Tensor& self = THPVariable_Unpack(self_);
4403:   if (has_torch_function(self_)) {
4404:     return handle_torch_function(self_, "atanh_");
4405:   }
4406:   // aten::atanh_(Tensor(a!) self) -> Tensor(a!)
4407: 
4408:   auto dispatch_atanh_ = [](const at::Tensor & self) -> at::Tensor {
4409:     pybind11::gil_scoped_release no_gil;
4410:     return self.atanh_();
4411:   };
4412:   return wrap(dispatch_atanh_(self));
4413:   END_HANDLE_TH_ERRORS
4414: }
4415: 
4416: \
4417: // baddbmm
4418: static PyObject * THPVariable_baddbmm(PyObject* self_, PyObject* args, PyObject* kwargs)
4419: {
4420:   HANDLE_TH_ERRORS
4421:   const Tensor& self = THPVariable_Unpack(self_);
4422:   static PythonArgParser parser({
4423:     "baddbmm(Scalar beta, Scalar alpha, Tensor batch1, Tensor batch2)|deprecated",
4424:     "baddbmm(Scalar beta, Tensor batch1, Tensor batch2)|deprecated",
4425:     "baddbmm(Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1)",
4426:   }, /*traceable=*/true);
4427: 
4428:   ParsedArgs<4> parsed_args;
4429:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4430:   if(_r.has_torch_function()) {
4431:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4432:   }
4433:   switch (_r.idx) {
4434:     case 0: {
4435:       // [deprecated] aten::baddbmm(Scalar beta, Tensor self, Scalar alpha, Tensor batch1, Tensor batch2) -> Tensor
4436: 
4437:       auto dispatch_baddbmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
4438:         pybind11::gil_scoped_release no_gil;
4439:         return self.baddbmm(batch1, batch2, beta, alpha);
4440:       };
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `atan2`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `atan2`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4441-4560

```cpp
4441:       return wrap(dispatch_baddbmm(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
4442:     }
4443:     case 1: {
4444:       // [deprecated] aten::baddbmm(Scalar beta, Tensor self, Tensor batch1, Tensor batch2) -> Tensor
4445: 
4446:       auto dispatch_baddbmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
4447:         pybind11::gil_scoped_release no_gil;
4448:         return self.baddbmm(batch1, batch2, beta, 1);
4449:       };
4450:       return wrap(dispatch_baddbmm(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
4451:     }
4452:     case 2: {
4453:       // aten::baddbmm(Tensor self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
4454: 
4455:       auto dispatch_baddbmm = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
4456:         pybind11::gil_scoped_release no_gil;
4457:         return self.baddbmm(batch1, batch2, beta, alpha);
4458:       };
4459:       return wrap(dispatch_baddbmm(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
4460:     }
4461:   }
4462:   Py_RETURN_NONE;
4463:   END_HANDLE_TH_ERRORS
4464: }
4465: 
4466: \
4467: // baddbmm_
4468: static PyObject * THPVariable_baddbmm_(PyObject* self_, PyObject* args, PyObject* kwargs)
4469: {
4470:   HANDLE_TH_ERRORS
4471:   const Tensor& self = THPVariable_Unpack(self_);
4472:   static PythonArgParser parser({
4473:     "baddbmm_(Scalar beta, Scalar alpha, Tensor batch1, Tensor batch2)|deprecated",
4474:     "baddbmm_(Scalar beta, Tensor batch1, Tensor batch2)|deprecated",
4475:     "baddbmm_(Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1)",
4476:   }, /*traceable=*/true);
4477: 
4478:   ParsedArgs<4> parsed_args;
4479:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4480:   if(_r.has_torch_function()) {
4481:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4482:   }
4483:   switch (_r.idx) {
4484:     case 0: {
4485:       // [deprecated] aten::baddbmm_(Scalar beta, Tensor(a!) self, Scalar alpha, Tensor batch1, Tensor batch2) -> Tensor(a!)
4486: 
4487:       auto dispatch_baddbmm_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
4488:         pybind11::gil_scoped_release no_gil;
4489:         return self.baddbmm_(batch1, batch2, beta, alpha);
4490:       };
4491:       return wrap(dispatch_baddbmm_(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
4492:     }
4493:     case 1: {
4494:       // [deprecated] aten::baddbmm_(Scalar beta, Tensor(a!) self, Tensor batch1, Tensor batch2) -> Tensor(a!)
4495: 
4496:       auto dispatch_baddbmm_ = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2) -> at::Tensor {
4497:         pybind11::gil_scoped_release no_gil;
4498:         return self.baddbmm_(batch1, batch2, beta, 1);
4499:       };
4500:       return wrap(dispatch_baddbmm_(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
4501:     }
4502:     case 2: {
4503:       // aten::baddbmm_(Tensor(a!) self, Tensor batch1, Tensor batch2, *, Scalar beta=1, Scalar alpha=1) -> Tensor(a!)
4504: 
4505:       auto dispatch_baddbmm_ = [](const at::Tensor & self, const at::Tensor & batch1, const at::Tensor & batch2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
4506:         pybind11::gil_scoped_release no_gil;
4507:         return self.baddbmm_(batch1, batch2, beta, alpha);
4508:       };
4509:       return wrap(dispatch_baddbmm_(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
4510:     }
4511:   }
4512:   Py_RETURN_NONE;
4513:   END_HANDLE_TH_ERRORS
4514: }
4515: 
4516: \
4517: // bernoulli
4518: static PyObject * THPVariable_bernoulli(PyObject* self_, PyObject* args, PyObject* kwargs)
4519: {
4520:   HANDLE_TH_ERRORS
4521:   const Tensor& self = THPVariable_Unpack(self_);
4522:   static PythonArgParser parser({
4523:     "bernoulli(*, Generator? generator=None)",
4524:     "bernoulli(double p, *, Generator? generator=None)",
4525:   }, /*traceable=*/true);
4526: 
4527:   ParsedArgs<2> parsed_args;
4528:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4529:   if(_r.has_torch_function()) {
4530:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4531:   }
4532:   switch (_r.idx) {
4533:     case 0: {
4534:       // aten::bernoulli(Tensor self, *, Generator? generator=None) -> Tensor
4535: 
4536:       auto dispatch_bernoulli = [](const at::Tensor & self, ::std::optional<at::Generator> generator) -> at::Tensor {
4537:         pybind11::gil_scoped_release no_gil;
4538:         return self.bernoulli(generator);
4539:       };
4540:       return wrap(dispatch_bernoulli(self, _r.generator(0)));
4541:     }
4542:     case 1: {
4543:       // aten::bernoulli.p(Tensor self, float p, *, Generator? generator=None) -> Tensor
4544: 
4545:       auto dispatch_bernoulli = [](const at::Tensor & self, double p, ::std::optional<at::Generator> generator) -> at::Tensor {
4546:         pybind11::gil_scoped_release no_gil;
4547:         return self.bernoulli(p, generator);
4548:       };
4549:       return wrap(dispatch_bernoulli(self, _r.toDouble(0), _r.generator(1)));
4550:     }
4551:   }
4552:   Py_RETURN_NONE;
4553:   END_HANDLE_TH_ERRORS
4554: }
4555: 
4556: \
4557: // bernoulli_
4558: static PyObject * THPVariable_bernoulli_(PyObject* self_, PyObject* args, PyObject* kwargs)
4559: {
4560:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `wrap`, `baddbmm`, `THPVariable_baddbmm_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `baddbmm`, `THPVariable_baddbmm_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4561-4680

```cpp
4561:   const Tensor& self = THPVariable_Unpack(self_);
4562:   static PythonArgParser parser({
4563:     "bernoulli_(Tensor p, *, Generator? generator=None)",
4564:     "bernoulli_(double p=0.5, *, Generator? generator=None)",
4565:   }, /*traceable=*/true);
4566: 
4567:   ParsedArgs<2> parsed_args;
4568:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4569:   if(_r.has_torch_function()) {
4570:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4571:   }
4572:   switch (_r.idx) {
4573:     case 0: {
4574:       // aten::bernoulli_.Tensor(Tensor(a!) self, Tensor p, *, Generator? generator=None) -> Tensor(a!)
4575: 
4576:       auto dispatch_bernoulli_ = [](const at::Tensor & self, const at::Tensor & p, ::std::optional<at::Generator> generator) -> at::Tensor {
4577:         pybind11::gil_scoped_release no_gil;
4578:         return self.bernoulli_(p, generator);
4579:       };
4580:       return wrap(dispatch_bernoulli_(self, _r.tensor(0), _r.generator(1)));
4581:     }
4582:     case 1: {
4583:       // aten::bernoulli_.float(Tensor(a!) self, float p=0.5, *, Generator? generator=None) -> Tensor(a!)
4584: 
4585:       auto dispatch_bernoulli_ = [](const at::Tensor & self, double p, ::std::optional<at::Generator> generator) -> at::Tensor {
4586:         pybind11::gil_scoped_release no_gil;
4587:         return self.bernoulli_(p, generator);
4588:       };
4589:       return wrap(dispatch_bernoulli_(self, _r.toDouble(0), _r.generator(1)));
4590:     }
4591:   }
4592:   Py_RETURN_NONE;
4593:   END_HANDLE_TH_ERRORS
4594: }
4595: 
4596: // bincount
4597: static PyObject * THPVariable_bincount(PyObject* self_, PyObject* args, PyObject* kwargs)
4598: {
4599:   HANDLE_TH_ERRORS
4600:   const Tensor& self = THPVariable_Unpack(self_);
4601:   static PythonArgParser parser({
4602:     "bincount(Tensor? weights=None, SymInt minlength=0)",
4603:   }, /*traceable=*/true);
4604: 
4605:   ParsedArgs<2> parsed_args;
4606:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4607:   if(_r.has_torch_function()) {
4608:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4609:   }
4610:   // aten::bincount(Tensor self, Tensor? weights=None, SymInt minlength=0) -> Tensor
4611: 
4612:   auto dispatch_bincount = [](const at::Tensor & self, const ::std::optional<at::Tensor> & weights, c10::SymInt minlength) -> at::Tensor {
4613:     pybind11::gil_scoped_release no_gil;
4614:     return self.bincount_symint(weights, minlength);
4615:   };
4616:   return wrap(dispatch_bincount(self, _r.optionalTensor(0), _r.toSymInt(1)));
4617:   Py_RETURN_NONE;
4618:   END_HANDLE_TH_ERRORS
4619: }
4620: 
4621: \
4622: // bitwise_and
4623: static PyObject * THPVariable_bitwise_and(PyObject* self_, PyObject* args, PyObject* kwargs)
4624: {
4625:   HANDLE_TH_ERRORS
4626:   const Tensor& self = THPVariable_Unpack(self_);
4627:   static PythonArgParser parser({
4628:     "bitwise_and(Tensor other)",
4629:     "bitwise_and(Scalar other)",
4630:   }, /*traceable=*/true);
4631: 
4632:   ParsedArgs<1> parsed_args;
4633:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4634:   if(_r.has_torch_function()) {
4635:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4636:   }
4637:   switch (_r.idx) {
4638:     case 0: {
4639:       // aten::bitwise_and.Tensor(Tensor self, Tensor other) -> Tensor
4640: 
4641:       auto dispatch_bitwise_and = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4642:         pybind11::gil_scoped_release no_gil;
4643:         return self.bitwise_and(other);
4644:       };
4645:       return wrap(dispatch_bitwise_and(self, _r.tensor(0)));
4646:     }
4647:     case 1: {
4648:       // aten::bitwise_and.Scalar(Tensor self, Scalar other) -> Tensor
4649: 
4650:       auto dispatch_bitwise_and = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4651:         pybind11::gil_scoped_release no_gil;
4652:         return self.bitwise_and(other);
4653:       };
4654:       return wrap(dispatch_bitwise_and(self, _r.scalar(0)));
4655:     }
4656:   }
4657:   Py_RETURN_NONE;
4658:   END_HANDLE_TH_ERRORS
4659: }
4660: 
4661: \
4662: // bitwise_and_
4663: static PyObject * THPVariable_bitwise_and_(PyObject* self_, PyObject* args, PyObject* kwargs)
4664: {
4665:   HANDLE_TH_ERRORS
4666:   const Tensor& self = THPVariable_Unpack(self_);
4667:   static PythonArgParser parser({
4668:     "bitwise_and_(Tensor other)",
4669:     "bitwise_and_(Scalar other)",
4670:   }, /*traceable=*/true);
4671: 
4672:   ParsedArgs<1> parsed_args;
4673:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4674:   if(_r.has_torch_function()) {
4675:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4676:   }
4677:   switch (_r.idx) {
4678:     case 0: {
4679:       // aten::bitwise_and_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
4680: 
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4681-4800

```cpp
4681:       auto dispatch_bitwise_and_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4682:         pybind11::gil_scoped_release no_gil;
4683:         return self.bitwise_and_(other);
4684:       };
4685:       return wrap(dispatch_bitwise_and_(self, _r.tensor(0)));
4686:     }
4687:     case 1: {
4688:       // aten::bitwise_and_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
4689: 
4690:       auto dispatch_bitwise_and_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4691:         pybind11::gil_scoped_release no_gil;
4692:         return self.bitwise_and_(other);
4693:       };
4694:       return wrap(dispatch_bitwise_and_(self, _r.scalar(0)));
4695:     }
4696:   }
4697:   Py_RETURN_NONE;
4698:   END_HANDLE_TH_ERRORS
4699: }
4700: 
4701: \
4702: // bitwise_left_shift
4703: static PyObject * THPVariable_bitwise_left_shift(PyObject* self_, PyObject* args, PyObject* kwargs)
4704: {
4705:   HANDLE_TH_ERRORS
4706:   const Tensor& self = THPVariable_Unpack(self_);
4707:   static PythonArgParser parser({
4708:     "bitwise_left_shift(Tensor other)",
4709:     "bitwise_left_shift(Scalar other)",
4710:   }, /*traceable=*/true);
4711: 
4712:   ParsedArgs<1> parsed_args;
4713:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4714:   if(_r.has_torch_function()) {
4715:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4716:   }
4717:   switch (_r.idx) {
4718:     case 0: {
4719:       // aten::bitwise_left_shift.Tensor(Tensor self, Tensor other) -> Tensor
4720: 
4721:       auto dispatch_bitwise_left_shift = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4722:         pybind11::gil_scoped_release no_gil;
4723:         return self.bitwise_left_shift(other);
4724:       };
4725:       return wrap(dispatch_bitwise_left_shift(self, _r.tensor(0)));
4726:     }
4727:     case 1: {
4728:       // aten::bitwise_left_shift.Tensor_Scalar(Tensor self, Scalar other) -> Tensor
4729: 
4730:       auto dispatch_bitwise_left_shift = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4731:         pybind11::gil_scoped_release no_gil;
4732:         return self.bitwise_left_shift(other);
4733:       };
4734:       return wrap(dispatch_bitwise_left_shift(self, _r.scalar(0)));
4735:     }
4736:   }
4737:   Py_RETURN_NONE;
4738:   END_HANDLE_TH_ERRORS
4739: }
4740: 
4741: \
4742: // bitwise_left_shift_
4743: static PyObject * THPVariable_bitwise_left_shift_(PyObject* self_, PyObject* args, PyObject* kwargs)
4744: {
4745:   HANDLE_TH_ERRORS
4746:   const Tensor& self = THPVariable_Unpack(self_);
4747:   static PythonArgParser parser({
4748:     "bitwise_left_shift_(Tensor other)",
4749:     "bitwise_left_shift_(Scalar other)",
4750:   }, /*traceable=*/true);
4751: 
4752:   ParsedArgs<1> parsed_args;
4753:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4754:   if(_r.has_torch_function()) {
4755:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4756:   }
4757:   switch (_r.idx) {
4758:     case 0: {
4759:       // aten::bitwise_left_shift_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
4760: 
4761:       auto dispatch_bitwise_left_shift_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4762:         pybind11::gil_scoped_release no_gil;
4763:         return self.bitwise_left_shift_(other);
4764:       };
4765:       return wrap(dispatch_bitwise_left_shift_(self, _r.tensor(0)));
4766:     }
4767:     case 1: {
4768:       // aten::bitwise_left_shift_.Tensor_Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
4769: 
4770:       auto dispatch_bitwise_left_shift_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4771:         pybind11::gil_scoped_release no_gil;
4772:         return self.bitwise_left_shift_(other);
4773:       };
4774:       return wrap(dispatch_bitwise_left_shift_(self, _r.scalar(0)));
4775:     }
4776:   }
4777:   Py_RETURN_NONE;
4778:   END_HANDLE_TH_ERRORS
4779: }
4780: 
4781: // bitwise_not
4782: static PyObject * THPVariable_bitwise_not(PyObject* self_, PyObject* args)
4783: {
4784:   HANDLE_TH_ERRORS
4785:   const Tensor& self = THPVariable_Unpack(self_);
4786:   if (has_torch_function(self_)) {
4787:     return handle_torch_function(self_, "bitwise_not");
4788:   }
4789:   // aten::bitwise_not(Tensor self) -> Tensor
4790: 
4791:   auto dispatch_bitwise_not = [](const at::Tensor & self) -> at::Tensor {
4792:     pybind11::gil_scoped_release no_gil;
4793:     return self.bitwise_not();
4794:   };
4795:   return wrap(dispatch_bitwise_not(self));
4796:   END_HANDLE_TH_ERRORS
4797: }
4798: 
4799: // bitwise_not_
4800: static PyObject * THPVariable_bitwise_not_(PyObject* self_, PyObject* args)
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_bitwise_left_shift`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_bitwise_left_shift`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4801-4920

```cpp
4801: {
4802:   HANDLE_TH_ERRORS
4803:   const Tensor& self = THPVariable_Unpack(self_);
4804:   if (has_torch_function(self_)) {
4805:     return handle_torch_function(self_, "bitwise_not_");
4806:   }
4807:   // aten::bitwise_not_(Tensor(a!) self) -> Tensor(a!)
4808: 
4809:   auto dispatch_bitwise_not_ = [](const at::Tensor & self) -> at::Tensor {
4810:     pybind11::gil_scoped_release no_gil;
4811:     return self.bitwise_not_();
4812:   };
4813:   return wrap(dispatch_bitwise_not_(self));
4814:   END_HANDLE_TH_ERRORS
4815: }
4816: 
4817: \
4818: // bitwise_or
4819: static PyObject * THPVariable_bitwise_or(PyObject* self_, PyObject* args, PyObject* kwargs)
4820: {
4821:   HANDLE_TH_ERRORS
4822:   const Tensor& self = THPVariable_Unpack(self_);
4823:   static PythonArgParser parser({
4824:     "bitwise_or(Tensor other)",
4825:     "bitwise_or(Scalar other)",
4826:   }, /*traceable=*/true);
4827: 
4828:   ParsedArgs<1> parsed_args;
4829:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4830:   if(_r.has_torch_function()) {
4831:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4832:   }
4833:   switch (_r.idx) {
4834:     case 0: {
4835:       // aten::bitwise_or.Tensor(Tensor self, Tensor other) -> Tensor
4836: 
4837:       auto dispatch_bitwise_or = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4838:         pybind11::gil_scoped_release no_gil;
4839:         return self.bitwise_or(other);
4840:       };
4841:       return wrap(dispatch_bitwise_or(self, _r.tensor(0)));
4842:     }
4843:     case 1: {
4844:       // aten::bitwise_or.Scalar(Tensor self, Scalar other) -> Tensor
4845: 
4846:       auto dispatch_bitwise_or = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4847:         pybind11::gil_scoped_release no_gil;
4848:         return self.bitwise_or(other);
4849:       };
4850:       return wrap(dispatch_bitwise_or(self, _r.scalar(0)));
4851:     }
4852:   }
4853:   Py_RETURN_NONE;
4854:   END_HANDLE_TH_ERRORS
4855: }
4856: 
4857: \
4858: // bitwise_or_
4859: static PyObject * THPVariable_bitwise_or_(PyObject* self_, PyObject* args, PyObject* kwargs)
4860: {
4861:   HANDLE_TH_ERRORS
4862:   const Tensor& self = THPVariable_Unpack(self_);
4863:   static PythonArgParser parser({
4864:     "bitwise_or_(Tensor other)",
4865:     "bitwise_or_(Scalar other)",
4866:   }, /*traceable=*/true);
4867: 
4868:   ParsedArgs<1> parsed_args;
4869:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4870:   if(_r.has_torch_function()) {
4871:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4872:   }
4873:   switch (_r.idx) {
4874:     case 0: {
4875:       // aten::bitwise_or_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
4876: 
4877:       auto dispatch_bitwise_or_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4878:         pybind11::gil_scoped_release no_gil;
4879:         return self.bitwise_or_(other);
4880:       };
4881:       return wrap(dispatch_bitwise_or_(self, _r.tensor(0)));
4882:     }
4883:     case 1: {
4884:       // aten::bitwise_or_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
4885: 
4886:       auto dispatch_bitwise_or_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4887:         pybind11::gil_scoped_release no_gil;
4888:         return self.bitwise_or_(other);
4889:       };
4890:       return wrap(dispatch_bitwise_or_(self, _r.scalar(0)));
4891:     }
4892:   }
4893:   Py_RETURN_NONE;
4894:   END_HANDLE_TH_ERRORS
4895: }
4896: 
4897: \
4898: // bitwise_right_shift
4899: static PyObject * THPVariable_bitwise_right_shift(PyObject* self_, PyObject* args, PyObject* kwargs)
4900: {
4901:   HANDLE_TH_ERRORS
4902:   const Tensor& self = THPVariable_Unpack(self_);
4903:   static PythonArgParser parser({
4904:     "bitwise_right_shift(Tensor other)",
4905:     "bitwise_right_shift(Scalar other)",
4906:   }, /*traceable=*/true);
4907: 
4908:   ParsedArgs<1> parsed_args;
4909:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4910:   if(_r.has_torch_function()) {
4911:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4912:   }
4913:   switch (_r.idx) {
4914:     case 0: {
4915:       // aten::bitwise_right_shift.Tensor(Tensor self, Tensor other) -> Tensor
4916: 
4917:       auto dispatch_bitwise_right_shift = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4918:         pybind11::gil_scoped_release no_gil;
4919:         return self.bitwise_right_shift(other);
4920:       };
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `handle_torch_function`, `bitwise_not_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `handle_torch_function`, `bitwise_not_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 4921-5040

```cpp
4921:       return wrap(dispatch_bitwise_right_shift(self, _r.tensor(0)));
4922:     }
4923:     case 1: {
4924:       // aten::bitwise_right_shift.Tensor_Scalar(Tensor self, Scalar other) -> Tensor
4925: 
4926:       auto dispatch_bitwise_right_shift = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4927:         pybind11::gil_scoped_release no_gil;
4928:         return self.bitwise_right_shift(other);
4929:       };
4930:       return wrap(dispatch_bitwise_right_shift(self, _r.scalar(0)));
4931:     }
4932:   }
4933:   Py_RETURN_NONE;
4934:   END_HANDLE_TH_ERRORS
4935: }
4936: 
4937: \
4938: // bitwise_right_shift_
4939: static PyObject * THPVariable_bitwise_right_shift_(PyObject* self_, PyObject* args, PyObject* kwargs)
4940: {
4941:   HANDLE_TH_ERRORS
4942:   const Tensor& self = THPVariable_Unpack(self_);
4943:   static PythonArgParser parser({
4944:     "bitwise_right_shift_(Tensor other)",
4945:     "bitwise_right_shift_(Scalar other)",
4946:   }, /*traceable=*/true);
4947: 
4948:   ParsedArgs<1> parsed_args;
4949:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4950:   if(_r.has_torch_function()) {
4951:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4952:   }
4953:   switch (_r.idx) {
4954:     case 0: {
4955:       // aten::bitwise_right_shift_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
4956: 
4957:       auto dispatch_bitwise_right_shift_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4958:         pybind11::gil_scoped_release no_gil;
4959:         return self.bitwise_right_shift_(other);
4960:       };
4961:       return wrap(dispatch_bitwise_right_shift_(self, _r.tensor(0)));
4962:     }
4963:     case 1: {
4964:       // aten::bitwise_right_shift_.Tensor_Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
4965: 
4966:       auto dispatch_bitwise_right_shift_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
4967:         pybind11::gil_scoped_release no_gil;
4968:         return self.bitwise_right_shift_(other);
4969:       };
4970:       return wrap(dispatch_bitwise_right_shift_(self, _r.scalar(0)));
4971:     }
4972:   }
4973:   Py_RETURN_NONE;
4974:   END_HANDLE_TH_ERRORS
4975: }
4976: 
4977: \
4978: // bitwise_xor
4979: static PyObject * THPVariable_bitwise_xor(PyObject* self_, PyObject* args, PyObject* kwargs)
4980: {
4981:   HANDLE_TH_ERRORS
4982:   const Tensor& self = THPVariable_Unpack(self_);
4983:   static PythonArgParser parser({
4984:     "bitwise_xor(Tensor other)",
4985:     "bitwise_xor(Scalar other)",
4986:   }, /*traceable=*/true);
4987: 
4988:   ParsedArgs<1> parsed_args;
4989:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
4990:   if(_r.has_torch_function()) {
4991:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
4992:   }
4993:   switch (_r.idx) {
4994:     case 0: {
4995:       // aten::bitwise_xor.Tensor(Tensor self, Tensor other) -> Tensor
4996: 
4997:       auto dispatch_bitwise_xor = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
4998:         pybind11::gil_scoped_release no_gil;
4999:         return self.bitwise_xor(other);
5000:       };
5001:       return wrap(dispatch_bitwise_xor(self, _r.tensor(0)));
5002:     }
5003:     case 1: {
5004:       // aten::bitwise_xor.Scalar(Tensor self, Scalar other) -> Tensor
5005: 
5006:       auto dispatch_bitwise_xor = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
5007:         pybind11::gil_scoped_release no_gil;
5008:         return self.bitwise_xor(other);
5009:       };
5010:       return wrap(dispatch_bitwise_xor(self, _r.scalar(0)));
5011:     }
5012:   }
5013:   Py_RETURN_NONE;
5014:   END_HANDLE_TH_ERRORS
5015: }
5016: 
5017: \
5018: // bitwise_xor_
5019: static PyObject * THPVariable_bitwise_xor_(PyObject* self_, PyObject* args, PyObject* kwargs)
5020: {
5021:   HANDLE_TH_ERRORS
5022:   const Tensor& self = THPVariable_Unpack(self_);
5023:   static PythonArgParser parser({
5024:     "bitwise_xor_(Tensor other)",
5025:     "bitwise_xor_(Scalar other)",
5026:   }, /*traceable=*/true);
5027: 
5028:   ParsedArgs<1> parsed_args;
5029:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5030:   if(_r.has_torch_function()) {
5031:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5032:   }
5033:   switch (_r.idx) {
5034:     case 0: {
5035:       // aten::bitwise_xor_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
5036: 
5037:       auto dispatch_bitwise_xor_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5038:         pybind11::gil_scoped_release no_gil;
5039:         return self.bitwise_xor_(other);
5040:       };
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_bitwise_right_shift_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_bitwise_right_shift_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5041-5160

```cpp
5041:       return wrap(dispatch_bitwise_xor_(self, _r.tensor(0)));
5042:     }
5043:     case 1: {
5044:       // aten::bitwise_xor_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
5045: 
5046:       auto dispatch_bitwise_xor_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
5047:         pybind11::gil_scoped_release no_gil;
5048:         return self.bitwise_xor_(other);
5049:       };
5050:       return wrap(dispatch_bitwise_xor_(self, _r.scalar(0)));
5051:     }
5052:   }
5053:   Py_RETURN_NONE;
5054:   END_HANDLE_TH_ERRORS
5055: }
5056: 
5057: // bmm
5058: static PyObject * THPVariable_bmm(PyObject* self_, PyObject* args, PyObject* kwargs)
5059: {
5060:   HANDLE_TH_ERRORS
5061:   const Tensor& self = THPVariable_Unpack(self_);
5062:   static PythonArgParser parser({
5063:     "bmm(Tensor mat2)",
5064:   }, /*traceable=*/true);
5065: 
5066:   ParsedArgs<1> parsed_args;
5067:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5068:   if(_r.has_torch_function()) {
5069:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5070:   }
5071:   // aten::bmm(Tensor self, Tensor mat2) -> Tensor
5072: 
5073:   auto dispatch_bmm = [](const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
5074:     pybind11::gil_scoped_release no_gil;
5075:     return self.bmm(mat2);
5076:   };
5077:   return wrap(dispatch_bmm(self, _r.tensor(0)));
5078:   Py_RETURN_NONE;
5079:   END_HANDLE_TH_ERRORS
5080: }
5081: 
5082: // broadcast_to
5083: static PyObject * THPVariable_broadcast_to(PyObject* self_, PyObject* args, PyObject* kwargs)
5084: {
5085:   HANDLE_TH_ERRORS
5086:   const Tensor& self = THPVariable_Unpack(self_);
5087:   static PythonArgParser parser({
5088:     "broadcast_to(SymIntArrayRef size)",
5089:   }, /*traceable=*/true);
5090: 
5091:   ParsedArgs<1> parsed_args;
5092:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5093:   if(_r.has_torch_function()) {
5094:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5095:   }
5096:   // aten::broadcast_to(Tensor(a) self, SymInt[] size) -> Tensor(a)
5097: 
5098:   auto dispatch_broadcast_to = [](const at::Tensor & self, c10::SymIntArrayRef size) -> at::Tensor {
5099:     pybind11::gil_scoped_release no_gil;
5100:     return self.broadcast_to_symint(size);
5101:   };
5102:   return wrap(dispatch_broadcast_to(self, _r.symintlist(0)));
5103:   Py_RETURN_NONE;
5104:   END_HANDLE_TH_ERRORS
5105: }
5106: 
5107: // cauchy_
5108: static PyObject * THPVariable_cauchy_(PyObject* self_, PyObject* args, PyObject* kwargs)
5109: {
5110:   HANDLE_TH_ERRORS
5111:   const Tensor& self = THPVariable_Unpack(self_);
5112:   static PythonArgParser parser({
5113:     "cauchy_(double median=0, double sigma=1, *, Generator? generator=None)",
5114:   }, /*traceable=*/true);
5115: 
5116:   ParsedArgs<3> parsed_args;
5117:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5118:   if(_r.has_torch_function()) {
5119:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5120:   }
5121:   // aten::cauchy_(Tensor(a!) self, float median=0, float sigma=1, *, Generator? generator=None) -> Tensor(a!)
5122: 
5123:   auto dispatch_cauchy_ = [](const at::Tensor & self, double median, double sigma, ::std::optional<at::Generator> generator) -> at::Tensor {
5124:     pybind11::gil_scoped_release no_gil;
5125:     return self.cauchy_(median, sigma, generator);
5126:   };
5127:   return wrap(dispatch_cauchy_(self, _r.toDouble(0), _r.toDouble(1), _r.generator(2)));
5128:   Py_RETURN_NONE;
5129:   END_HANDLE_TH_ERRORS
5130: }
5131: 
5132: // ccol_indices
5133: static PyObject * THPVariable_ccol_indices(PyObject* self_, PyObject* args)
5134: {
5135:   HANDLE_TH_ERRORS
5136:   const Tensor& self = THPVariable_Unpack(self_);
5137:   if (has_torch_function(self_)) {
5138:     return handle_torch_function(self_, "ccol_indices");
5139:   }
5140:   // aten::ccol_indices(Tensor(a) self) -> Tensor(a)
5141: 
5142:   auto dispatch_ccol_indices = [](const at::Tensor & self) -> at::Tensor {
5143:     pybind11::gil_scoped_release no_gil;
5144:     return self.ccol_indices();
5145:   };
5146:   return wrap(dispatch_ccol_indices(self));
5147:   END_HANDLE_TH_ERRORS
5148: }
5149: 
5150: // ceil
5151: static PyObject * THPVariable_ceil(PyObject* self_, PyObject* args)
5152: {
5153:   HANDLE_TH_ERRORS
5154:   const Tensor& self = THPVariable_Unpack(self_);
5155:   if (has_torch_function(self_)) {
5156:     return handle_torch_function(self_, "ceil");
5157:   }
5158:   // aten::ceil(Tensor self) -> Tensor
5159: 
5160:   auto dispatch_ceil = [](const at::Tensor & self) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_bmm`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_bmm`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 5161-5280

```cpp
5161:     pybind11::gil_scoped_release no_gil;
5162:     return self.ceil();
5163:   };
5164:   return wrap(dispatch_ceil(self));
5165:   END_HANDLE_TH_ERRORS
5166: }
5167: 
5168: // ceil_
5169: static PyObject * THPVariable_ceil_(PyObject* self_, PyObject* args)
5170: {
5171:   HANDLE_TH_ERRORS
5172:   const Tensor& self = THPVariable_Unpack(self_);
5173:   if (has_torch_function(self_)) {
5174:     return handle_torch_function(self_, "ceil_");
5175:   }
5176:   // aten::ceil_(Tensor(a!) self) -> Tensor(a!)
5177: 
5178:   auto dispatch_ceil_ = [](const at::Tensor & self) -> at::Tensor {
5179:     pybind11::gil_scoped_release no_gil;
5180:     return self.ceil_();
5181:   };
5182:   return wrap(dispatch_ceil_(self));
5183:   END_HANDLE_TH_ERRORS
5184: }
5185: 
5186: // chalf
5187: static PyObject * THPVariable_chalf(PyObject* self_, PyObject* args, PyObject* kwargs)
5188: {
5189:   HANDLE_TH_ERRORS
5190:   const Tensor& self = THPVariable_Unpack(self_);
5191:   static PythonArgParser parser({
5192:     "chalf(*, MemoryFormat? memory_format=None)",
5193:   }, /*traceable=*/true);
5194: 
5195:   ParsedArgs<1> parsed_args;
5196:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5197:   if(_r.has_torch_function()) {
5198:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5199:   }
5200:   // aten::chalf(Tensor self, *, MemoryFormat? memory_format=None) -> Tensor
5201: 
5202:   auto dispatch_chalf = [](const at::Tensor & self, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
5203:     pybind11::gil_scoped_release no_gil;
5204:     return self.chalf(memory_format);
5205:   };
5206:   return wrap(dispatch_chalf(self, _r.memoryformatOptional(0)));
5207:   Py_RETURN_NONE;
5208:   END_HANDLE_TH_ERRORS
5209: }
5210: 
5211: // cholesky
5212: static PyObject * THPVariable_cholesky(PyObject* self_, PyObject* args, PyObject* kwargs)
5213: {
5214:   HANDLE_TH_ERRORS
5215:   const Tensor& self = THPVariable_Unpack(self_);
5216:   static PythonArgParser parser({
5217:     "cholesky(bool upper=False)",
5218:   }, /*traceable=*/true);
5219: 
5220:   ParsedArgs<1> parsed_args;
5221:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5222:   if(_r.has_torch_function()) {
5223:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5224:   }
5225:   // aten::cholesky(Tensor self, bool upper=False) -> Tensor
5226: 
5227:   auto dispatch_cholesky = [](const at::Tensor & self, bool upper) -> at::Tensor {
5228:     pybind11::gil_scoped_release no_gil;
5229:     return self.cholesky(upper);
5230:   };
5231:   return wrap(dispatch_cholesky(self, _r.toBool(0)));
5232:   Py_RETURN_NONE;
5233:   END_HANDLE_TH_ERRORS
5234: }
5235: 
5236: // cholesky_inverse
5237: static PyObject * THPVariable_cholesky_inverse(PyObject* self_, PyObject* args, PyObject* kwargs)
5238: {
5239:   HANDLE_TH_ERRORS
5240:   const Tensor& self = THPVariable_Unpack(self_);
5241:   static PythonArgParser parser({
5242:     "cholesky_inverse(bool upper=False)",
5243:   }, /*traceable=*/true);
5244: 
5245:   ParsedArgs<1> parsed_args;
5246:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5247:   if(_r.has_torch_function()) {
5248:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5249:   }
5250:   // aten::cholesky_inverse(Tensor self, bool upper=False) -> Tensor
5251: 
5252:   auto dispatch_cholesky_inverse = [](const at::Tensor & self, bool upper) -> at::Tensor {
5253:     pybind11::gil_scoped_release no_gil;
5254:     return self.cholesky_inverse(upper);
5255:   };
5256:   return wrap(dispatch_cholesky_inverse(self, _r.toBool(0)));
5257:   Py_RETURN_NONE;
5258:   END_HANDLE_TH_ERRORS
5259: }
5260: 
5261: // cholesky_solve
5262: static PyObject * THPVariable_cholesky_solve(PyObject* self_, PyObject* args, PyObject* kwargs)
5263: {
5264:   HANDLE_TH_ERRORS
5265:   const Tensor& self = THPVariable_Unpack(self_);
5266:   static PythonArgParser parser({
5267:     "cholesky_solve(Tensor input2, bool upper=False)",
5268:   }, /*traceable=*/true);
5269: 
5270:   ParsedArgs<2> parsed_args;
5271:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5272:   if(_r.has_torch_function()) {
5273:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5274:   }
5275:   // aten::cholesky_solve(Tensor self, Tensor input2, bool upper=False) -> Tensor
5276: 
5277:   auto dispatch_cholesky_solve = [](const at::Tensor & self, const at::Tensor & input2, bool upper) -> at::Tensor {
5278:     pybind11::gil_scoped_release no_gil;
5279:     return self.cholesky_solve(input2, upper);
5280:   };
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_ceil_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_ceil_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5281-5400

```cpp
5281:   return wrap(dispatch_cholesky_solve(self, _r.tensor(0), _r.toBool(1)));
5282:   Py_RETURN_NONE;
5283:   END_HANDLE_TH_ERRORS
5284: }
5285: 
5286: // chunk
5287: static PyObject * THPVariable_chunk(PyObject* self_, PyObject* args, PyObject* kwargs)
5288: {
5289:   HANDLE_TH_ERRORS
5290:   const Tensor& self = THPVariable_Unpack(self_);
5291:   static PythonArgParser parser({
5292:     "chunk(int64_t chunks, int64_t dim=0)",
5293:   }, /*traceable=*/true);
5294: 
5295:   ParsedArgs<2> parsed_args;
5296:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5297:   if(_r.has_torch_function()) {
5298:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5299:   }
5300:   // aten::chunk(Tensor(a -> *) self, int chunks, int dim=0) -> Tensor(a)[]
5301: 
5302:   auto dispatch_chunk = [](const at::Tensor & self, int64_t chunks, int64_t dim) -> ::std::vector<at::Tensor> {
5303:     pybind11::gil_scoped_release no_gil;
5304:     return self.chunk(chunks, dim);
5305:   };
5306:   return wrap(dispatch_chunk(self, _r.toInt64(0), _r.toInt64(1)));
5307:   Py_RETURN_NONE;
5308:   END_HANDLE_TH_ERRORS
5309: }
5310: 
5311: \
5312: // clamp
5313: static PyObject * THPVariable_clamp(PyObject* self_, PyObject* args, PyObject* kwargs)
5314: {
5315:   HANDLE_TH_ERRORS
5316:   const Tensor& self = THPVariable_Unpack(self_);
5317:   static PythonArgParser parser({
5318:     "clamp(Tensor? min=None, Tensor? max=None)",
5319:     "clamp(Scalar? min=None, Scalar? max=None)",
5320:   }, /*traceable=*/true);
5321: 
5322:   ParsedArgs<2> parsed_args;
5323:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5324:   if(_r.has_torch_function()) {
5325:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5326:   }
5327:   switch (_r.idx) {
5328:     case 0: {
5329:       // aten::clamp.Tensor(Tensor self, Tensor? min=None, Tensor? max=None) -> Tensor
5330: 
5331:       auto dispatch_clamp = [](const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
5332:         pybind11::gil_scoped_release no_gil;
5333:         return self.clamp(min, max);
5334:       };
5335:       return wrap(dispatch_clamp(self, _r.optionalTensor(0), _r.optionalTensor(1)));
5336:     }
5337:     case 1: {
5338:       // aten::clamp(Tensor self, Scalar? min=None, Scalar? max=None) -> Tensor
5339: 
5340:       auto dispatch_clamp = [](const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
5341:         pybind11::gil_scoped_release no_gil;
5342:         return self.clamp(min, max);
5343:       };
5344:       return wrap(dispatch_clamp(self, _r.scalarOptional(0), _r.scalarOptional(1)));
5345:     }
5346:   }
5347:   Py_RETURN_NONE;
5348:   END_HANDLE_TH_ERRORS
5349: }
5350: 
5351: \
5352: // clamp_
5353: static PyObject * THPVariable_clamp_(PyObject* self_, PyObject* args, PyObject* kwargs)
5354: {
5355:   HANDLE_TH_ERRORS
5356:   const Tensor& self = THPVariable_Unpack(self_);
5357:   static PythonArgParser parser({
5358:     "clamp_(Tensor? min=None, Tensor? max=None)",
5359:     "clamp_(Scalar? min=None, Scalar? max=None)",
5360:   }, /*traceable=*/true);
5361: 
5362:   ParsedArgs<2> parsed_args;
5363:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5364:   if(_r.has_torch_function()) {
5365:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5366:   }
5367:   switch (_r.idx) {
5368:     case 0: {
5369:       // aten::clamp_.Tensor(Tensor(a!) self, Tensor? min=None, Tensor? max=None) -> Tensor(a!)
5370: 
5371:       auto dispatch_clamp_ = [](const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
5372:         pybind11::gil_scoped_release no_gil;
5373:         return self.clamp_(min, max);
5374:       };
5375:       return wrap(dispatch_clamp_(self, _r.optionalTensor(0), _r.optionalTensor(1)));
5376:     }
5377:     case 1: {
5378:       // aten::clamp_(Tensor(a!) self, Scalar? min=None, Scalar? max=None) -> Tensor(a!)
5379: 
5380:       auto dispatch_clamp_ = [](const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
5381:         pybind11::gil_scoped_release no_gil;
5382:         return self.clamp_(min, max);
5383:       };
5384:       return wrap(dispatch_clamp_(self, _r.scalarOptional(0), _r.scalarOptional(1)));
5385:     }
5386:   }
5387:   Py_RETURN_NONE;
5388:   END_HANDLE_TH_ERRORS
5389: }
5390: 
5391: \
5392: // clamp_max
5393: static PyObject * THPVariable_clamp_max(PyObject* self_, PyObject* args, PyObject* kwargs)
5394: {
5395:   HANDLE_TH_ERRORS
5396:   const Tensor& self = THPVariable_Unpack(self_);
5397:   static PythonArgParser parser({
5398:     "clamp_max(Tensor max)",
5399:     "clamp_max(Scalar max)",
5400:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_chunk`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_chunk`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5401-5520

```cpp
5401: 
5402:   ParsedArgs<1> parsed_args;
5403:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5404:   if(_r.has_torch_function()) {
5405:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5406:   }
5407:   switch (_r.idx) {
5408:     case 0: {
5409:       // aten::clamp_max.Tensor(Tensor self, Tensor max) -> Tensor
5410: 
5411:       auto dispatch_clamp_max = [](const at::Tensor & self, const at::Tensor & max) -> at::Tensor {
5412:         pybind11::gil_scoped_release no_gil;
5413:         return self.clamp_max(max);
5414:       };
5415:       return wrap(dispatch_clamp_max(self, _r.tensor(0)));
5416:     }
5417:     case 1: {
5418:       // aten::clamp_max(Tensor self, Scalar max) -> Tensor
5419: 
5420:       auto dispatch_clamp_max = [](const at::Tensor & self, const at::Scalar & max) -> at::Tensor {
5421:         pybind11::gil_scoped_release no_gil;
5422:         return self.clamp_max(max);
5423:       };
5424:       return wrap(dispatch_clamp_max(self, _r.scalar(0)));
5425:     }
5426:   }
5427:   Py_RETURN_NONE;
5428:   END_HANDLE_TH_ERRORS
5429: }
5430: 
5431: \
5432: // clamp_max_
5433: static PyObject * THPVariable_clamp_max_(PyObject* self_, PyObject* args, PyObject* kwargs)
5434: {
5435:   HANDLE_TH_ERRORS
5436:   const Tensor& self = THPVariable_Unpack(self_);
5437:   static PythonArgParser parser({
5438:     "clamp_max_(Tensor max)",
5439:     "clamp_max_(Scalar max)",
5440:   }, /*traceable=*/true);
5441: 
5442:   ParsedArgs<1> parsed_args;
5443:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5444:   if(_r.has_torch_function()) {
5445:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5446:   }
5447:   switch (_r.idx) {
5448:     case 0: {
5449:       // aten::clamp_max_.Tensor(Tensor(a!) self, Tensor max) -> Tensor(a!)
5450: 
5451:       auto dispatch_clamp_max_ = [](const at::Tensor & self, const at::Tensor & max) -> at::Tensor {
5452:         pybind11::gil_scoped_release no_gil;
5453:         return self.clamp_max_(max);
5454:       };
5455:       return wrap(dispatch_clamp_max_(self, _r.tensor(0)));
5456:     }
5457:     case 1: {
5458:       // aten::clamp_max_(Tensor(a!) self, Scalar max) -> Tensor(a!)
5459: 
5460:       auto dispatch_clamp_max_ = [](const at::Tensor & self, const at::Scalar & max) -> at::Tensor {
5461:         pybind11::gil_scoped_release no_gil;
5462:         return self.clamp_max_(max);
5463:       };
5464:       return wrap(dispatch_clamp_max_(self, _r.scalar(0)));
5465:     }
5466:   }
5467:   Py_RETURN_NONE;
5468:   END_HANDLE_TH_ERRORS
5469: }
5470: 
5471: \
5472: // clamp_min
5473: static PyObject * THPVariable_clamp_min(PyObject* self_, PyObject* args, PyObject* kwargs)
5474: {
5475:   HANDLE_TH_ERRORS
5476:   const Tensor& self = THPVariable_Unpack(self_);
5477:   static PythonArgParser parser({
5478:     "clamp_min(Tensor min)",
5479:     "clamp_min(Scalar min)",
5480:   }, /*traceable=*/true);
5481: 
5482:   ParsedArgs<1> parsed_args;
5483:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5484:   if(_r.has_torch_function()) {
5485:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5486:   }
5487:   switch (_r.idx) {
5488:     case 0: {
5489:       // aten::clamp_min.Tensor(Tensor self, Tensor min) -> Tensor
5490: 
5491:       auto dispatch_clamp_min = [](const at::Tensor & self, const at::Tensor & min) -> at::Tensor {
5492:         pybind11::gil_scoped_release no_gil;
5493:         return self.clamp_min(min);
5494:       };
5495:       return wrap(dispatch_clamp_min(self, _r.tensor(0)));
5496:     }
5497:     case 1: {
5498:       // aten::clamp_min(Tensor self, Scalar min) -> Tensor
5499: 
5500:       auto dispatch_clamp_min = [](const at::Tensor & self, const at::Scalar & min) -> at::Tensor {
5501:         pybind11::gil_scoped_release no_gil;
5502:         return self.clamp_min(min);
5503:       };
5504:       return wrap(dispatch_clamp_min(self, _r.scalar(0)));
5505:     }
5506:   }
5507:   Py_RETURN_NONE;
5508:   END_HANDLE_TH_ERRORS
5509: }
5510: 
5511: \
5512: // clamp_min_
5513: static PyObject * THPVariable_clamp_min_(PyObject* self_, PyObject* args, PyObject* kwargs)
5514: {
5515:   HANDLE_TH_ERRORS
5516:   const Tensor& self = THPVariable_Unpack(self_);
5517:   static PythonArgParser parser({
5518:     "clamp_min_(Tensor min)",
5519:     "clamp_min_(Scalar min)",
5520:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `clamp_max`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `clamp_max` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5521-5640

```cpp
5521: 
5522:   ParsedArgs<1> parsed_args;
5523:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5524:   if(_r.has_torch_function()) {
5525:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5526:   }
5527:   switch (_r.idx) {
5528:     case 0: {
5529:       // aten::clamp_min_.Tensor(Tensor(a!) self, Tensor min) -> Tensor(a!)
5530: 
5531:       auto dispatch_clamp_min_ = [](const at::Tensor & self, const at::Tensor & min) -> at::Tensor {
5532:         pybind11::gil_scoped_release no_gil;
5533:         return self.clamp_min_(min);
5534:       };
5535:       return wrap(dispatch_clamp_min_(self, _r.tensor(0)));
5536:     }
5537:     case 1: {
5538:       // aten::clamp_min_(Tensor(a!) self, Scalar min) -> Tensor(a!)
5539: 
5540:       auto dispatch_clamp_min_ = [](const at::Tensor & self, const at::Scalar & min) -> at::Tensor {
5541:         pybind11::gil_scoped_release no_gil;
5542:         return self.clamp_min_(min);
5543:       };
5544:       return wrap(dispatch_clamp_min_(self, _r.scalar(0)));
5545:     }
5546:   }
5547:   Py_RETURN_NONE;
5548:   END_HANDLE_TH_ERRORS
5549: }
5550: 
5551: \
5552: // clip
5553: static PyObject * THPVariable_clip(PyObject* self_, PyObject* args, PyObject* kwargs)
5554: {
5555:   HANDLE_TH_ERRORS
5556:   const Tensor& self = THPVariable_Unpack(self_);
5557:   static PythonArgParser parser({
5558:     "clip(Tensor? min=None, Tensor? max=None)",
5559:     "clip(Scalar? min=None, Scalar? max=None)",
5560:   }, /*traceable=*/true);
5561: 
5562:   ParsedArgs<2> parsed_args;
5563:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5564:   if(_r.has_torch_function()) {
5565:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5566:   }
5567:   switch (_r.idx) {
5568:     case 0: {
5569:       // aten::clip.Tensor(Tensor self, Tensor? min=None, Tensor? max=None) -> Tensor
5570: 
5571:       auto dispatch_clip = [](const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
5572:         pybind11::gil_scoped_release no_gil;
5573:         return self.clip(min, max);
5574:       };
5575:       return wrap(dispatch_clip(self, _r.optionalTensor(0), _r.optionalTensor(1)));
5576:     }
5577:     case 1: {
5578:       // aten::clip(Tensor self, Scalar? min=None, Scalar? max=None) -> Tensor
5579: 
5580:       auto dispatch_clip = [](const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
5581:         pybind11::gil_scoped_release no_gil;
5582:         return self.clip(min, max);
5583:       };
5584:       return wrap(dispatch_clip(self, _r.scalarOptional(0), _r.scalarOptional(1)));
5585:     }
5586:   }
5587:   Py_RETURN_NONE;
5588:   END_HANDLE_TH_ERRORS
5589: }
5590: 
5591: \
5592: // clip_
5593: static PyObject * THPVariable_clip_(PyObject* self_, PyObject* args, PyObject* kwargs)
5594: {
5595:   HANDLE_TH_ERRORS
5596:   const Tensor& self = THPVariable_Unpack(self_);
5597:   static PythonArgParser parser({
5598:     "clip_(Tensor? min=None, Tensor? max=None)",
5599:     "clip_(Scalar? min=None, Scalar? max=None)",
5600:   }, /*traceable=*/true);
5601: 
5602:   ParsedArgs<2> parsed_args;
5603:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5604:   if(_r.has_torch_function()) {
5605:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5606:   }
5607:   switch (_r.idx) {
5608:     case 0: {
5609:       // aten::clip_.Tensor(Tensor(a!) self, Tensor? min=None, Tensor? max=None) -> Tensor(a!)
5610: 
5611:       auto dispatch_clip_ = [](const at::Tensor & self, const ::std::optional<at::Tensor> & min, const ::std::optional<at::Tensor> & max) -> at::Tensor {
5612:         pybind11::gil_scoped_release no_gil;
5613:         return self.clip_(min, max);
5614:       };
5615:       return wrap(dispatch_clip_(self, _r.optionalTensor(0), _r.optionalTensor(1)));
5616:     }
5617:     case 1: {
5618:       // aten::clip_(Tensor(a!) self, Scalar? min=None, Scalar? max=None) -> Tensor(a!)
5619: 
5620:       auto dispatch_clip_ = [](const at::Tensor & self, const ::std::optional<at::Scalar> & min, const ::std::optional<at::Scalar> & max) -> at::Tensor {
5621:         pybind11::gil_scoped_release no_gil;
5622:         return self.clip_(min, max);
5623:       };
5624:       return wrap(dispatch_clip_(self, _r.scalarOptional(0), _r.scalarOptional(1)));
5625:     }
5626:   }
5627:   Py_RETURN_NONE;
5628:   END_HANDLE_TH_ERRORS
5629: }
5630: 
5631: // clone
5632: static PyObject * THPVariable_clone(PyObject* self_, PyObject* args, PyObject* kwargs)
5633: {
5634:   HANDLE_TH_ERRORS
5635:   const Tensor& self = THPVariable_Unpack(self_);
5636:   static PythonArgParser parser({
5637:     "clone(*, MemoryFormat? memory_format=None)",
5638:   }, /*traceable=*/true);
5639: 
5640:   ParsedArgs<1> parsed_args;
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `clamp_min_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `clamp_min_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5641-5760

```cpp
5641:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5642:   if(_r.has_torch_function()) {
5643:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5644:   }
5645:   // aten::clone(Tensor self, *, MemoryFormat? memory_format=None) -> Tensor
5646: 
5647:   auto dispatch_clone = [](const at::Tensor & self, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
5648:     pybind11::gil_scoped_release no_gil;
5649:     return self.clone(memory_format);
5650:   };
5651:   return wrap(dispatch_clone(self, _r.memoryformatOptional(0)));
5652:   Py_RETURN_NONE;
5653:   END_HANDLE_TH_ERRORS
5654: }
5655: 
5656: // coalesce
5657: static PyObject * THPVariable_coalesce(PyObject* self_, PyObject* args)
5658: {
5659:   HANDLE_TH_ERRORS
5660:   const Tensor& self = THPVariable_Unpack(self_);
5661:   if (has_torch_function(self_)) {
5662:     return handle_torch_function(self_, "coalesce");
5663:   }
5664:   // aten::coalesce(Tensor(a) self) -> Tensor(a)
5665: 
5666:   auto dispatch_coalesce = [](const at::Tensor & self) -> at::Tensor {
5667:     pybind11::gil_scoped_release no_gil;
5668:     return self.coalesce();
5669:   };
5670:   return wrap(dispatch_coalesce(self));
5671:   END_HANDLE_TH_ERRORS
5672: }
5673: 
5674: // col_indices
5675: static PyObject * THPVariable_col_indices(PyObject* self_, PyObject* args)
5676: {
5677:   HANDLE_TH_ERRORS
5678:   const Tensor& self = THPVariable_Unpack(self_);
5679:   if (has_torch_function(self_)) {
5680:     return handle_torch_function(self_, "col_indices");
5681:   }
5682:   // aten::col_indices(Tensor(a) self) -> Tensor(a)
5683: 
5684:   auto dispatch_col_indices = [](const at::Tensor & self) -> at::Tensor {
5685:     pybind11::gil_scoped_release no_gil;
5686:     return self.col_indices();
5687:   };
5688:   return wrap(dispatch_col_indices(self));
5689:   END_HANDLE_TH_ERRORS
5690: }
5691: 
5692: // conj
5693: static PyObject * THPVariable_conj(PyObject* self_, PyObject* args)
5694: {
5695:   HANDLE_TH_ERRORS
5696:   const Tensor& self = THPVariable_Unpack(self_);
5697:   if (has_torch_function(self_)) {
5698:     return handle_torch_function(self_, "conj");
5699:   }
5700:   // aten::conj(Tensor(a) self) -> Tensor(a)
5701: 
5702:   auto dispatch_conj = [](const at::Tensor & self) -> at::Tensor {
5703:     pybind11::gil_scoped_release no_gil;
5704:     return self.conj();
5705:   };
5706:   return wrap(dispatch_conj(self));
5707:   END_HANDLE_TH_ERRORS
5708: }
5709: 
5710: // conj_physical
5711: static PyObject * THPVariable_conj_physical(PyObject* self_, PyObject* args)
5712: {
5713:   HANDLE_TH_ERRORS
5714:   const Tensor& self = THPVariable_Unpack(self_);
5715:   if (has_torch_function(self_)) {
5716:     return handle_torch_function(self_, "conj_physical");
5717:   }
5718:   // aten::conj_physical(Tensor self) -> Tensor
5719: 
5720:   auto dispatch_conj_physical = [](const at::Tensor & self) -> at::Tensor {
5721:     pybind11::gil_scoped_release no_gil;
5722:     return self.conj_physical();
5723:   };
5724:   return wrap(dispatch_conj_physical(self));
5725:   END_HANDLE_TH_ERRORS
5726: }
5727: 
5728: // conj_physical_
5729: static PyObject * THPVariable_conj_physical_(PyObject* self_, PyObject* args)
5730: {
5731:   HANDLE_TH_ERRORS
5732:   const Tensor& self = THPVariable_Unpack(self_);
5733:   if (has_torch_function(self_)) {
5734:     return handle_torch_function(self_, "conj_physical_");
5735:   }
5736:   // aten::conj_physical_(Tensor(a!) self) -> Tensor(a!)
5737: 
5738:   auto dispatch_conj_physical_ = [](const at::Tensor & self) -> at::Tensor {
5739:     pybind11::gil_scoped_release no_gil;
5740:     return self.conj_physical_();
5741:   };
5742:   return wrap(dispatch_conj_physical_(self));
5743:   END_HANDLE_TH_ERRORS
5744: }
5745: 
5746: \
5747: // copysign
5748: static PyObject * THPVariable_copysign(PyObject* self_, PyObject* args, PyObject* kwargs)
5749: {
5750:   HANDLE_TH_ERRORS
5751:   const Tensor& self = THPVariable_Unpack(self_);
5752:   static PythonArgParser parser({
5753:     "copysign(Tensor other)",
5754:     "copysign(Scalar other)",
5755:   }, /*traceable=*/true);
5756: 
5757:   ParsedArgs<1> parsed_args;
5758:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5759:   if(_r.has_torch_function()) {
5760:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `clone`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `clone`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5761-5880

```cpp
5761:   }
5762:   switch (_r.idx) {
5763:     case 0: {
5764:       // aten::copysign.Tensor(Tensor self, Tensor other) -> Tensor
5765: 
5766:       auto dispatch_copysign = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5767:         pybind11::gil_scoped_release no_gil;
5768:         return self.copysign(other);
5769:       };
5770:       return wrap(dispatch_copysign(self, _r.tensor(0)));
5771:     }
5772:     case 1: {
5773:       // aten::copysign.Scalar(Tensor self, Scalar other) -> Tensor
5774: 
5775:       auto dispatch_copysign = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
5776:         pybind11::gil_scoped_release no_gil;
5777:         return self.copysign(other);
5778:       };
5779:       return wrap(dispatch_copysign(self, _r.scalar(0)));
5780:     }
5781:   }
5782:   Py_RETURN_NONE;
5783:   END_HANDLE_TH_ERRORS
5784: }
5785: 
5786: \
5787: // copysign_
5788: static PyObject * THPVariable_copysign_(PyObject* self_, PyObject* args, PyObject* kwargs)
5789: {
5790:   HANDLE_TH_ERRORS
5791:   const Tensor& self = THPVariable_Unpack(self_);
5792:   static PythonArgParser parser({
5793:     "copysign_(Tensor other)",
5794:     "copysign_(Scalar other)",
5795:   }, /*traceable=*/true);
5796: 
5797:   ParsedArgs<1> parsed_args;
5798:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5799:   if(_r.has_torch_function()) {
5800:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5801:   }
5802:   switch (_r.idx) {
5803:     case 0: {
5804:       // aten::copysign_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
5805: 
5806:       auto dispatch_copysign_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
5807:         pybind11::gil_scoped_release no_gil;
5808:         return self.copysign_(other);
5809:       };
5810:       return wrap(dispatch_copysign_(self, _r.tensor(0)));
5811:     }
5812:     case 1: {
5813:       // aten::copysign_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
5814: 
5815:       auto dispatch_copysign_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
5816:         pybind11::gil_scoped_release no_gil;
5817:         return self.copysign_(other);
5818:       };
5819:       return wrap(dispatch_copysign_(self, _r.scalar(0)));
5820:     }
5821:   }
5822:   Py_RETURN_NONE;
5823:   END_HANDLE_TH_ERRORS
5824: }
5825: 
5826: // corrcoef
5827: static PyObject * THPVariable_corrcoef(PyObject* self_, PyObject* args)
5828: {
5829:   HANDLE_TH_ERRORS
5830:   const Tensor& self = THPVariable_Unpack(self_);
5831:   if (has_torch_function(self_)) {
5832:     return handle_torch_function(self_, "corrcoef");
5833:   }
5834:   // aten::corrcoef(Tensor self) -> Tensor
5835: 
5836:   auto dispatch_corrcoef = [](const at::Tensor & self) -> at::Tensor {
5837:     pybind11::gil_scoped_release no_gil;
5838:     return self.corrcoef();
5839:   };
5840:   return wrap(dispatch_corrcoef(self));
5841:   END_HANDLE_TH_ERRORS
5842: }
5843: 
5844: // cos
5845: static PyObject * THPVariable_cos(PyObject* self_, PyObject* args)
5846: {
5847:   HANDLE_TH_ERRORS
5848:   const Tensor& self = THPVariable_Unpack(self_);
5849:   if (has_torch_function(self_)) {
5850:     return handle_torch_function(self_, "cos");
5851:   }
5852:   // aten::cos(Tensor self) -> Tensor
5853: 
5854:   auto dispatch_cos = [](const at::Tensor & self) -> at::Tensor {
5855:     pybind11::gil_scoped_release no_gil;
5856:     return self.cos();
5857:   };
5858:   return wrap(dispatch_cos(self));
5859:   END_HANDLE_TH_ERRORS
5860: }
5861: 
5862: // cos_
5863: static PyObject * THPVariable_cos_(PyObject* self_, PyObject* args)
5864: {
5865:   HANDLE_TH_ERRORS
5866:   const Tensor& self = THPVariable_Unpack(self_);
5867:   if (has_torch_function(self_)) {
5868:     return handle_torch_function(self_, "cos_");
5869:   }
5870:   // aten::cos_(Tensor(a!) self) -> Tensor(a!)
5871: 
5872:   auto dispatch_cos_ = [](const at::Tensor & self) -> at::Tensor {
5873:     pybind11::gil_scoped_release no_gil;
5874:     return self.cos_();
5875:   };
5876:   return wrap(dispatch_cos_(self));
5877:   END_HANDLE_TH_ERRORS
5878: }
5879: 
5880: // cosh
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_copysign_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_copysign_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 5881-6000

```cpp
5881: static PyObject * THPVariable_cosh(PyObject* self_, PyObject* args)
5882: {
5883:   HANDLE_TH_ERRORS
5884:   const Tensor& self = THPVariable_Unpack(self_);
5885:   if (has_torch_function(self_)) {
5886:     return handle_torch_function(self_, "cosh");
5887:   }
5888:   // aten::cosh(Tensor self) -> Tensor
5889: 
5890:   auto dispatch_cosh = [](const at::Tensor & self) -> at::Tensor {
5891:     pybind11::gil_scoped_release no_gil;
5892:     return self.cosh();
5893:   };
5894:   return wrap(dispatch_cosh(self));
5895:   END_HANDLE_TH_ERRORS
5896: }
5897: 
5898: // cosh_
5899: static PyObject * THPVariable_cosh_(PyObject* self_, PyObject* args)
5900: {
5901:   HANDLE_TH_ERRORS
5902:   const Tensor& self = THPVariable_Unpack(self_);
5903:   if (has_torch_function(self_)) {
5904:     return handle_torch_function(self_, "cosh_");
5905:   }
5906:   // aten::cosh_(Tensor(a!) self) -> Tensor(a!)
5907: 
5908:   auto dispatch_cosh_ = [](const at::Tensor & self) -> at::Tensor {
5909:     pybind11::gil_scoped_release no_gil;
5910:     return self.cosh_();
5911:   };
5912:   return wrap(dispatch_cosh_(self));
5913:   END_HANDLE_TH_ERRORS
5914: }
5915: 
5916: \
5917: // count_nonzero
5918: static PyObject * THPVariable_count_nonzero(PyObject* self_, PyObject* args, PyObject* kwargs)
5919: {
5920:   HANDLE_TH_ERRORS
5921:   const Tensor& self = THPVariable_Unpack(self_);
5922:   static PythonArgParser parser({
5923:     "count_nonzero(int64_t? dim=None)",
5924:     "count_nonzero(IntArrayRef dim)",
5925:   }, /*traceable=*/true);
5926: 
5927:   ParsedArgs<1> parsed_args;
5928:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5929:   if(_r.has_torch_function()) {
5930:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5931:   }
5932:   switch (_r.idx) {
5933:     case 0: {
5934:       // aten::count_nonzero(Tensor self, int? dim=None) -> Tensor
5935: 
5936:       auto dispatch_count_nonzero = [](const at::Tensor & self, ::std::optional<int64_t> dim) -> at::Tensor {
5937:         pybind11::gil_scoped_release no_gil;
5938:         return self.count_nonzero(dim);
5939:       };
5940:       return wrap(dispatch_count_nonzero(self, _r.toInt64Optional(0)));
5941:     }
5942:     case 1: {
5943:       // aten::count_nonzero.dim_IntList(Tensor self, int[] dim) -> Tensor
5944: 
5945:       auto dispatch_count_nonzero = [](const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
5946:         pybind11::gil_scoped_release no_gil;
5947:         return self.count_nonzero(dim);
5948:       };
5949:       return wrap(dispatch_count_nonzero(self, _r.intlist(0)));
5950:     }
5951:   }
5952:   Py_RETURN_NONE;
5953:   END_HANDLE_TH_ERRORS
5954: }
5955: 
5956: // cov
5957: static PyObject * THPVariable_cov(PyObject* self_, PyObject* args, PyObject* kwargs)
5958: {
5959:   HANDLE_TH_ERRORS
5960:   const Tensor& self = THPVariable_Unpack(self_);
5961:   static PythonArgParser parser({
5962:     "cov(*, int64_t correction=1, Tensor? fweights=None, Tensor? aweights=None)",
5963:   }, /*traceable=*/true);
5964: 
5965:   ParsedArgs<3> parsed_args;
5966:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5967:   if(_r.has_torch_function()) {
5968:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5969:   }
5970:   // aten::cov(Tensor self, *, int correction=1, Tensor? fweights=None, Tensor? aweights=None) -> Tensor
5971: 
5972:   auto dispatch_cov = [](const at::Tensor & self, int64_t correction, const ::std::optional<at::Tensor> & fweights, const ::std::optional<at::Tensor> & aweights) -> at::Tensor {
5973:     pybind11::gil_scoped_release no_gil;
5974:     return self.cov(correction, fweights, aweights);
5975:   };
5976:   return wrap(dispatch_cov(self, _r.toInt64(0), _r.optionalTensor(1), _r.optionalTensor(2)));
5977:   Py_RETURN_NONE;
5978:   END_HANDLE_TH_ERRORS
5979: }
5980: 
5981: // cross
5982: static PyObject * THPVariable_cross(PyObject* self_, PyObject* args, PyObject* kwargs)
5983: {
5984:   HANDLE_TH_ERRORS
5985:   const Tensor& self = THPVariable_Unpack(self_);
5986:   static PythonArgParser parser({
5987:     "cross(Tensor other, int64_t? dim=None)",
5988:   }, /*traceable=*/true);
5989: 
5990:   ParsedArgs<2> parsed_args;
5991:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
5992:   if(_r.has_torch_function()) {
5993:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
5994:   }
5995:   // aten::cross(Tensor self, Tensor other, int? dim=None) -> Tensor
5996: 
5997:   auto dispatch_cross = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<int64_t> dim) -> at::Tensor {
5998:     pybind11::gil_scoped_release no_gil;
5999:     return self.cross(other, dim);
6000:   };
```

- EN: The main execution path in this span is carried by `THPVariable_cosh`, `THPVariable_Unpack`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_cosh`, `THPVariable_Unpack`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6001-6120

```cpp
6001:   return wrap(dispatch_cross(self, _r.tensor(0), _r.toInt64Optional(1)));
6002:   Py_RETURN_NONE;
6003:   END_HANDLE_TH_ERRORS
6004: }
6005: 
6006: // crow_indices
6007: static PyObject * THPVariable_crow_indices(PyObject* self_, PyObject* args)
6008: {
6009:   HANDLE_TH_ERRORS
6010:   const Tensor& self = THPVariable_Unpack(self_);
6011:   if (has_torch_function(self_)) {
6012:     return handle_torch_function(self_, "crow_indices");
6013:   }
6014:   // aten::crow_indices(Tensor(a) self) -> Tensor(a)
6015: 
6016:   auto dispatch_crow_indices = [](const at::Tensor & self) -> at::Tensor {
6017:     pybind11::gil_scoped_release no_gil;
6018:     return self.crow_indices();
6019:   };
6020:   return wrap(dispatch_crow_indices(self));
6021:   END_HANDLE_TH_ERRORS
6022: }
6023: 
6024: \
6025: // cummax
6026: static PyObject * THPVariable_cummax(PyObject* self_, PyObject* args, PyObject* kwargs)
6027: {
6028:   HANDLE_TH_ERRORS
6029:   static PyTypeObject* NamedTuple = generated::get_cummax_structseq();
6030:   const Tensor& self = THPVariable_Unpack(self_);
6031:   static PythonArgParser parser({
6032:     "cummax(int64_t dim)",
6033:     "cummax(Dimname dim)",
6034:   }, /*traceable=*/true);
6035: 
6036:   ParsedArgs<1> parsed_args;
6037:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6038:   if(_r.has_torch_function()) {
6039:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6040:   }
6041:   switch (_r.idx) {
6042:     case 0: {
6043:       // aten::cummax(Tensor self, int dim) -> (Tensor values, Tensor indices)
6044: 
6045:       auto dispatch_cummax = [](const at::Tensor & self, int64_t dim) -> ::std::tuple<at::Tensor,at::Tensor> {
6046:         pybind11::gil_scoped_release no_gil;
6047:         return self.cummax(dim);
6048:       };
6049:       return wrap(NamedTuple, dispatch_cummax(self, _r.toInt64(0)));
6050:     }
6051:     case 1: {
6052:       // aten::cummax.dimname(Tensor self, Dimname dim) -> (Tensor values, Tensor indices)
6053: 
6054:       auto dispatch_cummax = [](const at::Tensor & self, at::Dimname dim) -> ::std::tuple<at::Tensor,at::Tensor> {
6055:         pybind11::gil_scoped_release no_gil;
6056:         return self.cummax(dim);
6057:       };
6058:       return wrap(NamedTuple, dispatch_cummax(self, _r.dimname(0)));
6059:     }
6060:   }
6061:   Py_RETURN_NONE;
6062:   END_HANDLE_TH_ERRORS
6063: }
6064: 
6065: \
6066: // cummin
6067: static PyObject * THPVariable_cummin(PyObject* self_, PyObject* args, PyObject* kwargs)
6068: {
6069:   HANDLE_TH_ERRORS
6070:   static PyTypeObject* NamedTuple = generated::get_cummin_structseq();
6071:   const Tensor& self = THPVariable_Unpack(self_);
6072:   static PythonArgParser parser({
6073:     "cummin(int64_t dim)",
6074:     "cummin(Dimname dim)",
6075:   }, /*traceable=*/true);
6076: 
6077:   ParsedArgs<1> parsed_args;
6078:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6079:   if(_r.has_torch_function()) {
6080:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6081:   }
6082:   switch (_r.idx) {
6083:     case 0: {
6084:       // aten::cummin(Tensor self, int dim) -> (Tensor values, Tensor indices)
6085: 
6086:       auto dispatch_cummin = [](const at::Tensor & self, int64_t dim) -> ::std::tuple<at::Tensor,at::Tensor> {
6087:         pybind11::gil_scoped_release no_gil;
6088:         return self.cummin(dim);
6089:       };
6090:       return wrap(NamedTuple, dispatch_cummin(self, _r.toInt64(0)));
6091:     }
6092:     case 1: {
6093:       // aten::cummin.dimname(Tensor self, Dimname dim) -> (Tensor values, Tensor indices)
6094: 
6095:       auto dispatch_cummin = [](const at::Tensor & self, at::Dimname dim) -> ::std::tuple<at::Tensor,at::Tensor> {
6096:         pybind11::gil_scoped_release no_gil;
6097:         return self.cummin(dim);
6098:       };
6099:       return wrap(NamedTuple, dispatch_cummin(self, _r.dimname(0)));
6100:     }
6101:   }
6102:   Py_RETURN_NONE;
6103:   END_HANDLE_TH_ERRORS
6104: }
6105: 
6106: \
6107: // cumprod
6108: static PyObject * THPVariable_cumprod(PyObject* self_, PyObject* args, PyObject* kwargs)
6109: {
6110:   HANDLE_TH_ERRORS
6111:   const Tensor& self = THPVariable_Unpack(self_);
6112:   static PythonArgParser parser({
6113:     "cumprod(int64_t dim, *, ScalarType? dtype=None)",
6114:     "cumprod(Dimname dim, *, ScalarType? dtype=None)",
6115:   }, /*traceable=*/true);
6116: 
6117:   ParsedArgs<2> parsed_args;
6118:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6119:   if(_r.has_torch_function()) {
6120:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_crow_indices`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_crow_indices`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6121-6240

```cpp
6121:   }
6122:   switch (_r.idx) {
6123:     case 0: {
6124:       // aten::cumprod(Tensor self, int dim, *, ScalarType? dtype=None) -> Tensor
6125: 
6126:       auto dispatch_cumprod = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6127:         pybind11::gil_scoped_release no_gil;
6128:         return self.cumprod(dim, dtype);
6129:       };
6130:       return wrap(dispatch_cumprod(self, _r.toInt64(0), _r.scalartypeOptional(1)));
6131:     }
6132:     case 1: {
6133:       // aten::cumprod.dimname(Tensor self, Dimname dim, *, ScalarType? dtype=None) -> Tensor
6134: 
6135:       auto dispatch_cumprod = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6136:         pybind11::gil_scoped_release no_gil;
6137:         return self.cumprod(dim, dtype);
6138:       };
6139:       return wrap(dispatch_cumprod(self, _r.dimname(0), _r.scalartypeOptional(1)));
6140:     }
6141:   }
6142:   Py_RETURN_NONE;
6143:   END_HANDLE_TH_ERRORS
6144: }
6145: 
6146: \
6147: // cumprod_
6148: static PyObject * THPVariable_cumprod_(PyObject* self_, PyObject* args, PyObject* kwargs)
6149: {
6150:   HANDLE_TH_ERRORS
6151:   const Tensor& self = THPVariable_Unpack(self_);
6152:   static PythonArgParser parser({
6153:     "cumprod_(int64_t dim, *, ScalarType? dtype=None)",
6154:     "cumprod_(Dimname dim, *, ScalarType? dtype=None)",
6155:   }, /*traceable=*/true);
6156: 
6157:   ParsedArgs<2> parsed_args;
6158:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6159:   if(_r.has_torch_function()) {
6160:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6161:   }
6162:   switch (_r.idx) {
6163:     case 0: {
6164:       // aten::cumprod_(Tensor(a!) self, int dim, *, ScalarType? dtype=None) -> Tensor(a!)
6165: 
6166:       auto dispatch_cumprod_ = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6167:         pybind11::gil_scoped_release no_gil;
6168:         return self.cumprod_(dim, dtype);
6169:       };
6170:       return wrap(dispatch_cumprod_(self, _r.toInt64(0), _r.scalartypeOptional(1)));
6171:     }
6172:     case 1: {
6173:       // aten::cumprod_.dimname(Tensor(a!) self, Dimname dim, *, ScalarType? dtype=None) -> Tensor(a!)
6174: 
6175:       auto dispatch_cumprod_ = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6176:         pybind11::gil_scoped_release no_gil;
6177:         return self.cumprod_(dim, dtype);
6178:       };
6179:       return wrap(dispatch_cumprod_(self, _r.dimname(0), _r.scalartypeOptional(1)));
6180:     }
6181:   }
6182:   Py_RETURN_NONE;
6183:   END_HANDLE_TH_ERRORS
6184: }
6185: 
6186: \
6187: // cumsum
6188: static PyObject * THPVariable_cumsum(PyObject* self_, PyObject* args, PyObject* kwargs)
6189: {
6190:   HANDLE_TH_ERRORS
6191:   const Tensor& self = THPVariable_Unpack(self_);
6192:   static PythonArgParser parser({
6193:     "cumsum(int64_t dim, *, ScalarType? dtype=None)",
6194:     "cumsum(Dimname dim, *, ScalarType? dtype=None)",
6195:   }, /*traceable=*/true);
6196: 
6197:   ParsedArgs<2> parsed_args;
6198:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6199:   if(_r.has_torch_function()) {
6200:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6201:   }
6202:   switch (_r.idx) {
6203:     case 0: {
6204:       // aten::cumsum(Tensor self, int dim, *, ScalarType? dtype=None) -> Tensor
6205: 
6206:       auto dispatch_cumsum = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6207:         pybind11::gil_scoped_release no_gil;
6208:         return self.cumsum(dim, dtype);
6209:       };
6210:       return wrap(dispatch_cumsum(self, _r.toInt64(0), _r.scalartypeOptional(1)));
6211:     }
6212:     case 1: {
6213:       // aten::cumsum.dimname(Tensor self, Dimname dim, *, ScalarType? dtype=None) -> Tensor
6214: 
6215:       auto dispatch_cumsum = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6216:         pybind11::gil_scoped_release no_gil;
6217:         return self.cumsum(dim, dtype);
6218:       };
6219:       return wrap(dispatch_cumsum(self, _r.dimname(0), _r.scalartypeOptional(1)));
6220:     }
6221:   }
6222:   Py_RETURN_NONE;
6223:   END_HANDLE_TH_ERRORS
6224: }
6225: 
6226: \
6227: // cumsum_
6228: static PyObject * THPVariable_cumsum_(PyObject* self_, PyObject* args, PyObject* kwargs)
6229: {
6230:   HANDLE_TH_ERRORS
6231:   const Tensor& self = THPVariable_Unpack(self_);
6232:   static PythonArgParser parser({
6233:     "cumsum_(int64_t dim, *, ScalarType? dtype=None)",
6234:     "cumsum_(Dimname dim, *, ScalarType? dtype=None)",
6235:   }, /*traceable=*/true);
6236: 
6237:   ParsedArgs<2> parsed_args;
6238:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6239:   if(_r.has_torch_function()) {
6240:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `cumprod`, `wrap`, `THPVariable_cumprod_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cumprod`, `wrap`, `THPVariable_cumprod_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6241-6360

```cpp
6241:   }
6242:   switch (_r.idx) {
6243:     case 0: {
6244:       // aten::cumsum_(Tensor(a!) self, int dim, *, ScalarType? dtype=None) -> Tensor(a!)
6245: 
6246:       auto dispatch_cumsum_ = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6247:         pybind11::gil_scoped_release no_gil;
6248:         return self.cumsum_(dim, dtype);
6249:       };
6250:       return wrap(dispatch_cumsum_(self, _r.toInt64(0), _r.scalartypeOptional(1)));
6251:     }
6252:     case 1: {
6253:       // aten::cumsum_.dimname(Tensor(a!) self, Dimname dim, *, ScalarType? dtype=None) -> Tensor(a!)
6254: 
6255:       auto dispatch_cumsum_ = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
6256:         pybind11::gil_scoped_release no_gil;
6257:         return self.cumsum_(dim, dtype);
6258:       };
6259:       return wrap(dispatch_cumsum_(self, _r.dimname(0), _r.scalartypeOptional(1)));
6260:     }
6261:   }
6262:   Py_RETURN_NONE;
6263:   END_HANDLE_TH_ERRORS
6264: }
6265: 
6266: // deg2rad
6267: static PyObject * THPVariable_deg2rad(PyObject* self_, PyObject* args)
6268: {
6269:   HANDLE_TH_ERRORS
6270:   const Tensor& self = THPVariable_Unpack(self_);
6271:   if (has_torch_function(self_)) {
6272:     return handle_torch_function(self_, "deg2rad");
6273:   }
6274:   // aten::deg2rad(Tensor self) -> Tensor
6275: 
6276:   auto dispatch_deg2rad = [](const at::Tensor & self) -> at::Tensor {
6277:     pybind11::gil_scoped_release no_gil;
6278:     return self.deg2rad();
6279:   };
6280:   return wrap(dispatch_deg2rad(self));
6281:   END_HANDLE_TH_ERRORS
6282: }
6283: 
6284: // deg2rad_
6285: static PyObject * THPVariable_deg2rad_(PyObject* self_, PyObject* args)
6286: {
6287:   HANDLE_TH_ERRORS
6288:   const Tensor& self = THPVariable_Unpack(self_);
6289:   if (has_torch_function(self_)) {
6290:     return handle_torch_function(self_, "deg2rad_");
6291:   }
6292:   // aten::deg2rad_(Tensor(a!) self) -> Tensor(a!)
6293: 
6294:   auto dispatch_deg2rad_ = [](const at::Tensor & self) -> at::Tensor {
6295:     pybind11::gil_scoped_release no_gil;
6296:     return self.deg2rad_();
6297:   };
6298:   return wrap(dispatch_deg2rad_(self));
6299:   END_HANDLE_TH_ERRORS
6300: }
6301: 
6302: // dense_dim
6303: static PyObject * THPVariable_dense_dim(PyObject* self_, PyObject* args)
6304: {
6305:   HANDLE_TH_ERRORS
6306:   const Tensor& self = THPVariable_Unpack(self_);
6307:   if (has_torch_function(self_)) {
6308:     return handle_torch_function(self_, "dense_dim");
6309:   }
6310:   // aten::dense_dim(Tensor self) -> int
6311: 
6312:   auto dispatch_dense_dim = [](const at::Tensor & self) -> int64_t {
6313:     pybind11::gil_scoped_release no_gil;
6314:     return self.dense_dim();
6315:   };
6316:   return wrap(dispatch_dense_dim(self));
6317:   END_HANDLE_TH_ERRORS
6318: }
6319: 
6320: // dequantize
6321: static PyObject * THPVariable_dequantize(PyObject* self_, PyObject* args)
6322: {
6323:   HANDLE_TH_ERRORS
6324:   const Tensor& self = THPVariable_Unpack(self_);
6325:   if (has_torch_function(self_)) {
6326:     return handle_torch_function(self_, "dequantize");
6327:   }
6328:   // aten::dequantize.self(Tensor self) -> Tensor
6329: 
6330:   auto dispatch_dequantize = [](const at::Tensor & self) -> at::Tensor {
6331:     pybind11::gil_scoped_release no_gil;
6332:     return self.dequantize();
6333:   };
6334:   return wrap(dispatch_dequantize(self));
6335:   END_HANDLE_TH_ERRORS
6336: }
6337: 
6338: // det
6339: static PyObject * THPVariable_det(PyObject* self_, PyObject* args)
6340: {
6341:   HANDLE_TH_ERRORS
6342:   const Tensor& self = THPVariable_Unpack(self_);
6343:   if (has_torch_function(self_)) {
6344:     return handle_torch_function(self_, "det");
6345:   }
6346:   // aten::det(Tensor self) -> Tensor
6347: 
6348:   auto dispatch_det = [](const at::Tensor & self) -> at::Tensor {
6349:     pybind11::gil_scoped_release no_gil;
6350:     return self.det();
6351:   };
6352:   return wrap(dispatch_det(self));
6353:   END_HANDLE_TH_ERRORS
6354: }
6355: 
6356: // detach
6357: static PyObject * THPVariable_detach(PyObject* self_, PyObject* args)
6358: {
6359:   HANDLE_TH_ERRORS
6360:   const Tensor& self = THPVariable_Unpack(self_);
```

- EN: The main execution path in this span is carried by `cumsum_`, `wrap`, `THPVariable_deg2rad`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `cumsum_`, `wrap`, `THPVariable_deg2rad` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6361-6480

```cpp
6361:   if (has_torch_function(self_)) {
6362:     return handle_torch_function(self_, "detach");
6363:   }
6364:   // aten::detach(Tensor(a) self) -> Tensor(a)
6365: 
6366:   auto dispatch_detach = [](const at::Tensor & self) -> at::Tensor {
6367:     pybind11::gil_scoped_release no_gil;
6368:     return self.detach();
6369:   };
6370:   return wrap(dispatch_detach(self));
6371:   END_HANDLE_TH_ERRORS
6372: }
6373: 
6374: // detach_
6375: static PyObject * THPVariable_detach_(PyObject* self_, PyObject* args)
6376: {
6377:   HANDLE_TH_ERRORS
6378:   const Tensor& self = THPVariable_Unpack(self_);
6379:   if (has_torch_function(self_)) {
6380:     return handle_torch_function(self_, "detach_");
6381:   }
6382:   // aten::detach_(Tensor(a!) self) -> Tensor(a!)
6383: 
6384:   auto dispatch_detach_ = [](const at::Tensor & self) -> at::Tensor {
6385:     pybind11::gil_scoped_release no_gil;
6386:     return self.detach_();
6387:   };
6388:   return wrap(dispatch_detach_(self));
6389:   END_HANDLE_TH_ERRORS
6390: }
6391: 
6392: // diag
6393: static PyObject * THPVariable_diag(PyObject* self_, PyObject* args, PyObject* kwargs)
6394: {
6395:   HANDLE_TH_ERRORS
6396:   const Tensor& self = THPVariable_Unpack(self_);
6397:   static PythonArgParser parser({
6398:     "diag(int64_t diagonal=0)",
6399:   }, /*traceable=*/true);
6400: 
6401:   ParsedArgs<1> parsed_args;
6402:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6403:   if(_r.has_torch_function()) {
6404:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6405:   }
6406:   // aten::diag(Tensor self, int diagonal=0) -> Tensor
6407: 
6408:   auto dispatch_diag = [](const at::Tensor & self, int64_t diagonal) -> at::Tensor {
6409:     pybind11::gil_scoped_release no_gil;
6410:     return self.diag(diagonal);
6411:   };
6412:   return wrap(dispatch_diag(self, _r.toInt64(0)));
6413:   Py_RETURN_NONE;
6414:   END_HANDLE_TH_ERRORS
6415: }
6416: 
6417: // diag_embed
6418: static PyObject * THPVariable_diag_embed(PyObject* self_, PyObject* args, PyObject* kwargs)
6419: {
6420:   HANDLE_TH_ERRORS
6421:   const Tensor& self = THPVariable_Unpack(self_);
6422:   static PythonArgParser parser({
6423:     "diag_embed(int64_t offset=0, int64_t dim1=-2, int64_t dim2=-1)",
6424:   }, /*traceable=*/true);
6425: 
6426:   ParsedArgs<3> parsed_args;
6427:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6428:   if(_r.has_torch_function()) {
6429:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6430:   }
6431:   // aten::diag_embed(Tensor self, int offset=0, int dim1=-2, int dim2=-1) -> Tensor
6432: 
6433:   auto dispatch_diag_embed = [](const at::Tensor & self, int64_t offset, int64_t dim1, int64_t dim2) -> at::Tensor {
6434:     pybind11::gil_scoped_release no_gil;
6435:     return self.diag_embed(offset, dim1, dim2);
6436:   };
6437:   return wrap(dispatch_diag_embed(self, _r.toInt64(0), _r.toInt64(1), _r.toInt64(2)));
6438:   Py_RETURN_NONE;
6439:   END_HANDLE_TH_ERRORS
6440: }
6441: 
6442: // diagflat
6443: static PyObject * THPVariable_diagflat(PyObject* self_, PyObject* args, PyObject* kwargs)
6444: {
6445:   HANDLE_TH_ERRORS
6446:   const Tensor& self = THPVariable_Unpack(self_);
6447:   static PythonArgParser parser({
6448:     "diagflat(int64_t offset=0)",
6449:   }, /*traceable=*/true);
6450: 
6451:   ParsedArgs<1> parsed_args;
6452:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6453:   if(_r.has_torch_function()) {
6454:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6455:   }
6456:   // aten::diagflat(Tensor self, int offset=0) -> Tensor
6457: 
6458:   auto dispatch_diagflat = [](const at::Tensor & self, int64_t offset) -> at::Tensor {
6459:     pybind11::gil_scoped_release no_gil;
6460:     return self.diagflat(offset);
6461:   };
6462:   return wrap(dispatch_diagflat(self, _r.toInt64(0)));
6463:   Py_RETURN_NONE;
6464:   END_HANDLE_TH_ERRORS
6465: }
6466: 
6467: \
6468: // diagonal
6469: static PyObject * THPVariable_diagonal(PyObject* self_, PyObject* args, PyObject* kwargs)
6470: {
6471:   HANDLE_TH_ERRORS
6472:   const Tensor& self = THPVariable_Unpack(self_);
6473:   static PythonArgParser parser({
6474:     "diagonal(*, Dimname outdim, Dimname dim1, Dimname dim2, int64_t offset=0)",
6475:     "diagonal(int64_t offset=0, int64_t dim1=0, int64_t dim2=1)",
6476:   }, /*traceable=*/true);
6477: 
6478:   ParsedArgs<4> parsed_args;
6479:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6480:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `detach`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `detach`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6481-6600

```cpp
6481:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6482:   }
6483:   switch (_r.idx) {
6484:     case 0: {
6485:       // aten::diagonal.Dimname(Tensor(a) self, *, Dimname outdim, Dimname dim1, Dimname dim2, int offset=0) -> Tensor(a)
6486: 
6487:       auto dispatch_diagonal = [](const at::Tensor & self, at::Dimname outdim, at::Dimname dim1, at::Dimname dim2, int64_t offset) -> at::Tensor {
6488:         pybind11::gil_scoped_release no_gil;
6489:         return self.diagonal(outdim, dim1, dim2, offset);
6490:       };
6491:       return wrap(dispatch_diagonal(self, _r.dimname(0), _r.dimname(1), _r.dimname(2), _r.toInt64(3)));
6492:     }
6493:     case 1: {
6494:       // aten::diagonal(Tensor(a) self, int offset=0, int dim1=0, int dim2=1) -> Tensor(a)
6495: 
6496:       auto dispatch_diagonal = [](const at::Tensor & self, int64_t offset, int64_t dim1, int64_t dim2) -> at::Tensor {
6497:         pybind11::gil_scoped_release no_gil;
6498:         return self.diagonal(offset, dim1, dim2);
6499:       };
6500:       return wrap(dispatch_diagonal(self, _r.toInt64(0), _r.toInt64(1), _r.toInt64(2)));
6501:     }
6502:   }
6503:   Py_RETURN_NONE;
6504:   END_HANDLE_TH_ERRORS
6505: }
6506: 
6507: // diagonal_scatter
6508: static PyObject * THPVariable_diagonal_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
6509: {
6510:   HANDLE_TH_ERRORS
6511:   const Tensor& self = THPVariable_Unpack(self_);
6512:   static PythonArgParser parser({
6513:     "diagonal_scatter(Tensor src, int64_t offset=0, int64_t dim1=0, int64_t dim2=1)",
6514:   }, /*traceable=*/true);
6515: 
6516:   ParsedArgs<4> parsed_args;
6517:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6518:   if(_r.has_torch_function()) {
6519:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6520:   }
6521:   // aten::diagonal_scatter(Tensor self, Tensor src, int offset=0, int dim1=0, int dim2=1) -> Tensor
6522: 
6523:   auto dispatch_diagonal_scatter = [](const at::Tensor & self, const at::Tensor & src, int64_t offset, int64_t dim1, int64_t dim2) -> at::Tensor {
6524:     pybind11::gil_scoped_release no_gil;
6525:     return self.diagonal_scatter(src, offset, dim1, dim2);
6526:   };
6527:   return wrap(dispatch_diagonal_scatter(self, _r.tensor(0), _r.toInt64(1), _r.toInt64(2), _r.toInt64(3)));
6528:   Py_RETURN_NONE;
6529:   END_HANDLE_TH_ERRORS
6530: }
6531: 
6532: // diff
6533: static PyObject * THPVariable_diff(PyObject* self_, PyObject* args, PyObject* kwargs)
6534: {
6535:   HANDLE_TH_ERRORS
6536:   const Tensor& self = THPVariable_Unpack(self_);
6537:   static PythonArgParser parser({
6538:     "diff(int64_t n=1, int64_t dim=-1, Tensor? prepend=None, Tensor? append=None)",
6539:   }, /*traceable=*/true);
6540: 
6541:   ParsedArgs<4> parsed_args;
6542:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6543:   if(_r.has_torch_function()) {
6544:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6545:   }
6546:   // aten::diff(Tensor self, int n=1, int dim=-1, Tensor? prepend=None, Tensor? append=None) -> Tensor
6547: 
6548:   auto dispatch_diff = [](const at::Tensor & self, int64_t n, int64_t dim, const ::std::optional<at::Tensor> & prepend, const ::std::optional<at::Tensor> & append) -> at::Tensor {
6549:     pybind11::gil_scoped_release no_gil;
6550:     return self.diff(n, dim, prepend, append);
6551:   };
6552:   return wrap(dispatch_diff(self, _r.toInt64(0), _r.toInt64(1), _r.optionalTensor(2), _r.optionalTensor(3)));
6553:   Py_RETURN_NONE;
6554:   END_HANDLE_TH_ERRORS
6555: }
6556: 
6557: // digamma
6558: static PyObject * THPVariable_digamma(PyObject* self_, PyObject* args)
6559: {
6560:   HANDLE_TH_ERRORS
6561:   const Tensor& self = THPVariable_Unpack(self_);
6562:   if (has_torch_function(self_)) {
6563:     return handle_torch_function(self_, "digamma");
6564:   }
6565:   // aten::digamma(Tensor self) -> Tensor
6566: 
6567:   auto dispatch_digamma = [](const at::Tensor & self) -> at::Tensor {
6568:     pybind11::gil_scoped_release no_gil;
6569:     return self.digamma();
6570:   };
6571:   return wrap(dispatch_digamma(self));
6572:   END_HANDLE_TH_ERRORS
6573: }
6574: 
6575: // digamma_
6576: static PyObject * THPVariable_digamma_(PyObject* self_, PyObject* args)
6577: {
6578:   HANDLE_TH_ERRORS
6579:   const Tensor& self = THPVariable_Unpack(self_);
6580:   if (has_torch_function(self_)) {
6581:     return handle_torch_function(self_, "digamma_");
6582:   }
6583:   // aten::digamma_(Tensor(a!) self) -> Tensor(a!)
6584: 
6585:   auto dispatch_digamma_ = [](const at::Tensor & self) -> at::Tensor {
6586:     pybind11::gil_scoped_release no_gil;
6587:     return self.digamma_();
6588:   };
6589:   return wrap(dispatch_digamma_(self));
6590:   END_HANDLE_TH_ERRORS
6591: }
6592: 
6593: // dist
6594: static PyObject * THPVariable_dist(PyObject* self_, PyObject* args, PyObject* kwargs)
6595: {
6596:   HANDLE_TH_ERRORS
6597:   const Tensor& self = THPVariable_Unpack(self_);
6598:   static PythonArgParser parser({
6599:     "dist(Tensor other, Scalar p=2)",
6600:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `diagonal`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `diagonal` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6601-6720

```cpp
6601: 
6602:   ParsedArgs<2> parsed_args;
6603:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6604:   if(_r.has_torch_function()) {
6605:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6606:   }
6607:   // aten::dist(Tensor self, Tensor other, Scalar p=2) -> Tensor
6608: 
6609:   auto dispatch_dist = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & p) -> at::Tensor {
6610:     pybind11::gil_scoped_release no_gil;
6611:     return self.dist(other, p);
6612:   };
6613:   return wrap(dispatch_dist(self, _r.tensor(0), _r.scalar(1)));
6614:   Py_RETURN_NONE;
6615:   END_HANDLE_TH_ERRORS
6616: }
6617: 
6618: \
6619: // div
6620: static PyObject * THPVariable_div(PyObject* self_, PyObject* args, PyObject* kwargs)
6621: {
6622:   HANDLE_TH_ERRORS
6623:   const Tensor& self = THPVariable_Unpack(self_);
6624:   static PythonArgParser parser({
6625:     "div(Tensor other)",
6626:     "div(Tensor other, *, c10::string_view? rounding_mode)",
6627:     "div(Scalar other, *, c10::string_view? rounding_mode)",
6628:   }, /*traceable=*/true);
6629: 
6630:   ParsedArgs<2> parsed_args;
6631:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6632:   if(_r.has_torch_function()) {
6633:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6634:   }
6635:   switch (_r.idx) {
6636:     case 0: {
6637:       // aten::div.Tensor(Tensor self, Tensor other) -> Tensor
6638: 
6639:       auto dispatch_div = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
6640:         pybind11::gil_scoped_release no_gil;
6641:         return self.div(other);
6642:       };
6643:       return wrap(dispatch_div(self, _r.tensor(0)));
6644:     }
6645:     case 1: {
6646:       // aten::div.Tensor_mode(Tensor self, Tensor other, *, str? rounding_mode) -> Tensor
6647: 
6648:       auto dispatch_div = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6649:         pybind11::gil_scoped_release no_gil;
6650:         return self.div(other, rounding_mode);
6651:       };
6652:       return wrap(dispatch_div(self, _r.tensor(0), _r.stringViewOptional(1)));
6653:     }
6654:     case 2: {
6655:       // aten::div.Scalar_mode(Tensor self, Scalar other, *, str? rounding_mode) -> Tensor
6656: 
6657:       auto dispatch_div = [](const at::Tensor & self, const at::Scalar & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6658:         pybind11::gil_scoped_release no_gil;
6659:         return self.div(other, rounding_mode);
6660:       };
6661:       return wrap(dispatch_div(self, _r.scalar(0), _r.stringViewOptional(1)));
6662:     }
6663:   }
6664:   Py_RETURN_NONE;
6665:   END_HANDLE_TH_ERRORS
6666: }
6667: 
6668: \
6669: // div_
6670: static PyObject * THPVariable_div_(PyObject* self_, PyObject* args, PyObject* kwargs)
6671: {
6672:   HANDLE_TH_ERRORS
6673:   const Tensor& self = THPVariable_Unpack(self_);
6674:   static PythonArgParser parser({
6675:     "div_(Tensor other)",
6676:     "div_(Tensor other, *, c10::string_view? rounding_mode)",
6677:     "div_(Scalar other, *, c10::string_view? rounding_mode)",
6678:   }, /*traceable=*/true);
6679: 
6680:   ParsedArgs<2> parsed_args;
6681:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6682:   if(_r.has_torch_function()) {
6683:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6684:   }
6685:   switch (_r.idx) {
6686:     case 0: {
6687:       // aten::div_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
6688: 
6689:       auto dispatch_div_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
6690:         pybind11::gil_scoped_release no_gil;
6691:         return self.div_(other);
6692:       };
6693:       return wrap(dispatch_div_(self, _r.tensor(0)));
6694:     }
6695:     case 1: {
6696:       // aten::div_.Tensor_mode(Tensor(a!) self, Tensor other, *, str? rounding_mode) -> Tensor(a!)
6697: 
6698:       auto dispatch_div_ = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6699:         pybind11::gil_scoped_release no_gil;
6700:         return self.div_(other, rounding_mode);
6701:       };
6702:       return wrap(dispatch_div_(self, _r.tensor(0), _r.stringViewOptional(1)));
6703:     }
6704:     case 2: {
6705:       // aten::div_.Scalar_mode(Tensor(a!) self, Scalar other, *, str? rounding_mode) -> Tensor(a!)
6706: 
6707:       auto dispatch_div_ = [](const at::Tensor & self, const at::Scalar & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6708:         pybind11::gil_scoped_release no_gil;
6709:         return self.div_(other, rounding_mode);
6710:       };
6711:       return wrap(dispatch_div_(self, _r.scalar(0), _r.stringViewOptional(1)));
6712:     }
6713:   }
6714:   Py_RETURN_NONE;
6715:   END_HANDLE_TH_ERRORS
6716: }
6717: 
6718: \
6719: // divide
6720: static PyObject * THPVariable_divide(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `dist`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `dist`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6721-6840

```cpp
6721: {
6722:   HANDLE_TH_ERRORS
6723:   const Tensor& self = THPVariable_Unpack(self_);
6724:   static PythonArgParser parser({
6725:     "divide(Tensor other)",
6726:     "divide(Tensor other, *, c10::string_view? rounding_mode)",
6727:     "divide(Scalar other)",
6728:     "divide(Scalar other, *, c10::string_view? rounding_mode)",
6729:   }, /*traceable=*/true);
6730: 
6731:   ParsedArgs<2> parsed_args;
6732:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6733:   if(_r.has_torch_function()) {
6734:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6735:   }
6736:   switch (_r.idx) {
6737:     case 0: {
6738:       // aten::divide.Tensor(Tensor self, Tensor other) -> Tensor
6739: 
6740:       auto dispatch_divide = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
6741:         pybind11::gil_scoped_release no_gil;
6742:         return self.divide(other);
6743:       };
6744:       return wrap(dispatch_divide(self, _r.tensor(0)));
6745:     }
6746:     case 1: {
6747:       // aten::divide.Tensor_mode(Tensor self, Tensor other, *, str? rounding_mode) -> Tensor
6748: 
6749:       auto dispatch_divide = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6750:         pybind11::gil_scoped_release no_gil;
6751:         return self.divide(other, rounding_mode);
6752:       };
6753:       return wrap(dispatch_divide(self, _r.tensor(0), _r.stringViewOptional(1)));
6754:     }
6755:     case 2: {
6756:       // aten::divide.Scalar(Tensor self, Scalar other) -> Tensor
6757: 
6758:       auto dispatch_divide = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
6759:         pybind11::gil_scoped_release no_gil;
6760:         return self.divide(other);
6761:       };
6762:       return wrap(dispatch_divide(self, _r.scalar(0)));
6763:     }
6764:     case 3: {
6765:       // aten::divide.Scalar_mode(Tensor self, Scalar other, *, str? rounding_mode) -> Tensor
6766: 
6767:       auto dispatch_divide = [](const at::Tensor & self, const at::Scalar & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6768:         pybind11::gil_scoped_release no_gil;
6769:         return self.divide(other, rounding_mode);
6770:       };
6771:       return wrap(dispatch_divide(self, _r.scalar(0), _r.stringViewOptional(1)));
6772:     }
6773:   }
6774:   Py_RETURN_NONE;
6775:   END_HANDLE_TH_ERRORS
6776: }
6777: 
6778: \
6779: // divide_
6780: static PyObject * THPVariable_divide_(PyObject* self_, PyObject* args, PyObject* kwargs)
6781: {
6782:   HANDLE_TH_ERRORS
6783:   const Tensor& self = THPVariable_Unpack(self_);
6784:   static PythonArgParser parser({
6785:     "divide_(Tensor other)",
6786:     "divide_(Tensor other, *, c10::string_view? rounding_mode)",
6787:     "divide_(Scalar other)",
6788:     "divide_(Scalar other, *, c10::string_view? rounding_mode)",
6789:   }, /*traceable=*/true);
6790: 
6791:   ParsedArgs<2> parsed_args;
6792:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6793:   if(_r.has_torch_function()) {
6794:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6795:   }
6796:   switch (_r.idx) {
6797:     case 0: {
6798:       // aten::divide_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
6799: 
6800:       auto dispatch_divide_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
6801:         pybind11::gil_scoped_release no_gil;
6802:         return self.divide_(other);
6803:       };
6804:       return wrap(dispatch_divide_(self, _r.tensor(0)));
6805:     }
6806:     case 1: {
6807:       // aten::divide_.Tensor_mode(Tensor(a!) self, Tensor other, *, str? rounding_mode) -> Tensor(a!)
6808: 
6809:       auto dispatch_divide_ = [](const at::Tensor & self, const at::Tensor & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6810:         pybind11::gil_scoped_release no_gil;
6811:         return self.divide_(other, rounding_mode);
6812:       };
6813:       return wrap(dispatch_divide_(self, _r.tensor(0), _r.stringViewOptional(1)));
6814:     }
6815:     case 2: {
6816:       // aten::divide_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
6817: 
6818:       auto dispatch_divide_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
6819:         pybind11::gil_scoped_release no_gil;
6820:         return self.divide_(other);
6821:       };
6822:       return wrap(dispatch_divide_(self, _r.scalar(0)));
6823:     }
6824:     case 3: {
6825:       // aten::divide_.Scalar_mode(Tensor(a!) self, Scalar other, *, str? rounding_mode) -> Tensor(a!)
6826: 
6827:       auto dispatch_divide_ = [](const at::Tensor & self, const at::Scalar & other, ::std::optional<c10::string_view> rounding_mode) -> at::Tensor {
6828:         pybind11::gil_scoped_release no_gil;
6829:         return self.divide_(other, rounding_mode);
6830:       };
6831:       return wrap(dispatch_divide_(self, _r.scalar(0), _r.stringViewOptional(1)));
6832:     }
6833:   }
6834:   Py_RETURN_NONE;
6835:   END_HANDLE_TH_ERRORS
6836: }
6837: 
6838: // dot
6839: static PyObject * THPVariable_dot(PyObject* self_, PyObject* args, PyObject* kwargs)
6840: {
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 6841-6960

```cpp
6841:   HANDLE_TH_ERRORS
6842:   const Tensor& self = THPVariable_Unpack(self_);
6843:   static PythonArgParser parser({
6844:     "dot(Tensor tensor)",
6845:   }, /*traceable=*/true);
6846: 
6847:   ParsedArgs<1> parsed_args;
6848:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6849:   if(_r.has_torch_function()) {
6850:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6851:   }
6852:   // aten::dot(Tensor self, Tensor tensor) -> Tensor
6853: 
6854:   auto dispatch_dot = [](const at::Tensor & self, const at::Tensor & tensor) -> at::Tensor {
6855:     pybind11::gil_scoped_release no_gil;
6856:     return self.dot(tensor);
6857:   };
6858:   return wrap(dispatch_dot(self, _r.tensor(0)));
6859:   Py_RETURN_NONE;
6860:   END_HANDLE_TH_ERRORS
6861: }
6862: 
6863: \
6864: // dsplit
6865: static PyObject * THPVariable_dsplit(PyObject* self_, PyObject* args, PyObject* kwargs)
6866: {
6867:   HANDLE_TH_ERRORS
6868:   const Tensor& self = THPVariable_Unpack(self_);
6869:   static PythonArgParser parser({
6870:     "dsplit(int64_t sections)",
6871:     "dsplit(IntArrayRef indices)",
6872:   }, /*traceable=*/true);
6873: 
6874:   ParsedArgs<1> parsed_args;
6875:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6876:   if(_r.has_torch_function()) {
6877:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6878:   }
6879:   switch (_r.idx) {
6880:     case 0: {
6881:       // aten::dsplit.int(Tensor(a -> *) self, int sections) -> Tensor(a)[]
6882: 
6883:       auto dispatch_dsplit = [](const at::Tensor & self, int64_t sections) -> ::std::vector<at::Tensor> {
6884:         pybind11::gil_scoped_release no_gil;
6885:         return self.dsplit(sections);
6886:       };
6887:       return wrap(dispatch_dsplit(self, _r.toInt64(0)));
6888:     }
6889:     case 1: {
6890:       // aten::dsplit.array(Tensor(a -> *) self, int[] indices) -> Tensor(a)[]
6891: 
6892:       auto dispatch_dsplit = [](const at::Tensor & self, at::IntArrayRef indices) -> ::std::vector<at::Tensor> {
6893:         pybind11::gil_scoped_release no_gil;
6894:         return self.dsplit(indices);
6895:       };
6896:       return wrap(dispatch_dsplit(self, _r.intlist(0)));
6897:     }
6898:   }
6899:   Py_RETURN_NONE;
6900:   END_HANDLE_TH_ERRORS
6901: }
6902: 
6903: \
6904: // eq
6905: static PyObject * THPVariable_eq(PyObject* self_, PyObject* args, PyObject* kwargs)
6906: {
6907:   HANDLE_TH_ERRORS
6908:   const Tensor& self = THPVariable_Unpack(self_);
6909:   static PythonArgParser parser({
6910:     "eq(Tensor other)",
6911:     "eq(Scalar other)",
6912:   }, /*traceable=*/true);
6913: 
6914:   ParsedArgs<1> parsed_args;
6915:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6916:   if(_r.has_torch_function()) {
6917:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6918:   }
6919:   switch (_r.idx) {
6920:     case 0: {
6921:       // aten::eq.Tensor(Tensor self, Tensor other) -> Tensor
6922: 
6923:       auto dispatch_eq = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
6924:         pybind11::gil_scoped_release no_gil;
6925:         return self.eq(other);
6926:       };
6927:       return wrap(dispatch_eq(self, _r.tensor(0)));
6928:     }
6929:     case 1: {
6930:       // aten::eq.Scalar(Tensor self, Scalar other) -> Tensor
6931: 
6932:       auto dispatch_eq = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
6933:         pybind11::gil_scoped_release no_gil;
6934:         return self.eq(other);
6935:       };
6936:       return wrap(dispatch_eq(self, _r.scalar(0)));
6937:     }
6938:   }
6939:   Py_RETURN_NONE;
6940:   END_HANDLE_TH_ERRORS
6941: }
6942: 
6943: \
6944: // eq_
6945: static PyObject * THPVariable_eq_(PyObject* self_, PyObject* args, PyObject* kwargs)
6946: {
6947:   HANDLE_TH_ERRORS
6948:   const Tensor& self = THPVariable_Unpack(self_);
6949:   static PythonArgParser parser({
6950:     "eq_(Tensor other)",
6951:     "eq_(Scalar other)",
6952:   }, /*traceable=*/true);
6953: 
6954:   ParsedArgs<1> parsed_args;
6955:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6956:   if(_r.has_torch_function()) {
6957:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6958:   }
6959:   switch (_r.idx) {
6960:     case 0: {
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 6961-7080

```cpp
6961:       // aten::eq_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
6962: 
6963:       auto dispatch_eq_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
6964:         pybind11::gil_scoped_release no_gil;
6965:         return self.eq_(other);
6966:       };
6967:       return wrap(dispatch_eq_(self, _r.tensor(0)));
6968:     }
6969:     case 1: {
6970:       // aten::eq_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
6971: 
6972:       auto dispatch_eq_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
6973:         pybind11::gil_scoped_release no_gil;
6974:         return self.eq_(other);
6975:       };
6976:       return wrap(dispatch_eq_(self, _r.scalar(0)));
6977:     }
6978:   }
6979:   Py_RETURN_NONE;
6980:   END_HANDLE_TH_ERRORS
6981: }
6982: 
6983: // equal
6984: static PyObject * THPVariable_equal(PyObject* self_, PyObject* args, PyObject* kwargs)
6985: {
6986:   HANDLE_TH_ERRORS
6987:   const Tensor& self = THPVariable_Unpack(self_);
6988:   static PythonArgParser parser({
6989:     "equal(Tensor other)",
6990:   }, /*traceable=*/false);
6991: 
6992:   ParsedArgs<1> parsed_args;
6993:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
6994:   if(_r.has_torch_function()) {
6995:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
6996:   }
6997:   // aten::equal(Tensor self, Tensor other) -> bool
6998: 
6999:   auto dispatch_equal = [](const at::Tensor & self, const at::Tensor & other) -> bool {
7000:     pybind11::gil_scoped_release no_gil;
7001:     return self.equal(other);
7002:   };
7003:   return wrap(dispatch_equal(self, _r.tensor(0)));
7004:   Py_RETURN_NONE;
7005:   END_HANDLE_TH_ERRORS
7006: }
7007: 
7008: // erf
7009: static PyObject * THPVariable_erf(PyObject* self_, PyObject* args)
7010: {
7011:   HANDLE_TH_ERRORS
7012:   const Tensor& self = THPVariable_Unpack(self_);
7013:   if (has_torch_function(self_)) {
7014:     return handle_torch_function(self_, "erf");
7015:   }
7016:   // aten::erf(Tensor self) -> Tensor
7017: 
7018:   auto dispatch_erf = [](const at::Tensor & self) -> at::Tensor {
7019:     pybind11::gil_scoped_release no_gil;
7020:     return self.erf();
7021:   };
7022:   return wrap(dispatch_erf(self));
7023:   END_HANDLE_TH_ERRORS
7024: }
7025: 
7026: // erf_
7027: static PyObject * THPVariable_erf_(PyObject* self_, PyObject* args)
7028: {
7029:   HANDLE_TH_ERRORS
7030:   const Tensor& self = THPVariable_Unpack(self_);
7031:   if (has_torch_function(self_)) {
7032:     return handle_torch_function(self_, "erf_");
7033:   }
7034:   // aten::erf_(Tensor(a!) self) -> Tensor(a!)
7035: 
7036:   auto dispatch_erf_ = [](const at::Tensor & self) -> at::Tensor {
7037:     pybind11::gil_scoped_release no_gil;
7038:     return self.erf_();
7039:   };
7040:   return wrap(dispatch_erf_(self));
7041:   END_HANDLE_TH_ERRORS
7042: }
7043: 
7044: // erfc
7045: static PyObject * THPVariable_erfc(PyObject* self_, PyObject* args)
7046: {
7047:   HANDLE_TH_ERRORS
7048:   const Tensor& self = THPVariable_Unpack(self_);
7049:   if (has_torch_function(self_)) {
7050:     return handle_torch_function(self_, "erfc");
7051:   }
7052:   // aten::erfc(Tensor self) -> Tensor
7053: 
7054:   auto dispatch_erfc = [](const at::Tensor & self) -> at::Tensor {
7055:     pybind11::gil_scoped_release no_gil;
7056:     return self.erfc();
7057:   };
7058:   return wrap(dispatch_erfc(self));
7059:   END_HANDLE_TH_ERRORS
7060: }
7061: 
7062: // erfc_
7063: static PyObject * THPVariable_erfc_(PyObject* self_, PyObject* args)
7064: {
7065:   HANDLE_TH_ERRORS
7066:   const Tensor& self = THPVariable_Unpack(self_);
7067:   if (has_torch_function(self_)) {
7068:     return handle_torch_function(self_, "erfc_");
7069:   }
7070:   // aten::erfc_(Tensor(a!) self) -> Tensor(a!)
7071: 
7072:   auto dispatch_erfc_ = [](const at::Tensor & self) -> at::Tensor {
7073:     pybind11::gil_scoped_release no_gil;
7074:     return self.erfc_();
7075:   };
7076:   return wrap(dispatch_erfc_(self));
7077:   END_HANDLE_TH_ERRORS
7078: }
7079: 
7080: // erfinv
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_equal`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_equal`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7081-7200

```cpp
7081: static PyObject * THPVariable_erfinv(PyObject* self_, PyObject* args)
7082: {
7083:   HANDLE_TH_ERRORS
7084:   const Tensor& self = THPVariable_Unpack(self_);
7085:   if (has_torch_function(self_)) {
7086:     return handle_torch_function(self_, "erfinv");
7087:   }
7088:   // aten::erfinv(Tensor self) -> Tensor
7089: 
7090:   auto dispatch_erfinv = [](const at::Tensor & self) -> at::Tensor {
7091:     pybind11::gil_scoped_release no_gil;
7092:     return self.erfinv();
7093:   };
7094:   return wrap(dispatch_erfinv(self));
7095:   END_HANDLE_TH_ERRORS
7096: }
7097: 
7098: // erfinv_
7099: static PyObject * THPVariable_erfinv_(PyObject* self_, PyObject* args)
7100: {
7101:   HANDLE_TH_ERRORS
7102:   const Tensor& self = THPVariable_Unpack(self_);
7103:   if (has_torch_function(self_)) {
7104:     return handle_torch_function(self_, "erfinv_");
7105:   }
7106:   // aten::erfinv_(Tensor(a!) self) -> Tensor(a!)
7107: 
7108:   auto dispatch_erfinv_ = [](const at::Tensor & self) -> at::Tensor {
7109:     pybind11::gil_scoped_release no_gil;
7110:     return self.erfinv_();
7111:   };
7112:   return wrap(dispatch_erfinv_(self));
7113:   END_HANDLE_TH_ERRORS
7114: }
7115: 
7116: // exp
7117: static PyObject * THPVariable_exp(PyObject* self_, PyObject* args)
7118: {
7119:   HANDLE_TH_ERRORS
7120:   const Tensor& self = THPVariable_Unpack(self_);
7121:   if (has_torch_function(self_)) {
7122:     return handle_torch_function(self_, "exp");
7123:   }
7124:   // aten::exp(Tensor self) -> Tensor
7125: 
7126:   auto dispatch_exp = [](const at::Tensor & self) -> at::Tensor {
7127:     pybind11::gil_scoped_release no_gil;
7128:     return self.exp();
7129:   };
7130:   return wrap(dispatch_exp(self));
7131:   END_HANDLE_TH_ERRORS
7132: }
7133: 
7134: // exp2
7135: static PyObject * THPVariable_exp2(PyObject* self_, PyObject* args)
7136: {
7137:   HANDLE_TH_ERRORS
7138:   const Tensor& self = THPVariable_Unpack(self_);
7139:   if (has_torch_function(self_)) {
7140:     return handle_torch_function(self_, "exp2");
7141:   }
7142:   // aten::exp2(Tensor self) -> Tensor
7143: 
7144:   auto dispatch_exp2 = [](const at::Tensor & self) -> at::Tensor {
7145:     pybind11::gil_scoped_release no_gil;
7146:     return self.exp2();
7147:   };
7148:   return wrap(dispatch_exp2(self));
7149:   END_HANDLE_TH_ERRORS
7150: }
7151: 
7152: // exp2_
7153: static PyObject * THPVariable_exp2_(PyObject* self_, PyObject* args)
7154: {
7155:   HANDLE_TH_ERRORS
7156:   const Tensor& self = THPVariable_Unpack(self_);
7157:   if (has_torch_function(self_)) {
7158:     return handle_torch_function(self_, "exp2_");
7159:   }
7160:   // aten::exp2_(Tensor(a!) self) -> Tensor(a!)
7161: 
7162:   auto dispatch_exp2_ = [](const at::Tensor & self) -> at::Tensor {
7163:     pybind11::gil_scoped_release no_gil;
7164:     return self.exp2_();
7165:   };
7166:   return wrap(dispatch_exp2_(self));
7167:   END_HANDLE_TH_ERRORS
7168: }
7169: 
7170: // exp_
7171: static PyObject * THPVariable_exp_(PyObject* self_, PyObject* args)
7172: {
7173:   HANDLE_TH_ERRORS
7174:   const Tensor& self = THPVariable_Unpack(self_);
7175:   if (has_torch_function(self_)) {
7176:     return handle_torch_function(self_, "exp_");
7177:   }
7178:   // aten::exp_(Tensor(a!) self) -> Tensor(a!)
7179: 
7180:   auto dispatch_exp_ = [](const at::Tensor & self) -> at::Tensor {
7181:     pybind11::gil_scoped_release no_gil;
7182:     return self.exp_();
7183:   };
7184:   return wrap(dispatch_exp_(self));
7185:   END_HANDLE_TH_ERRORS
7186: }
7187: 
7188: // expand
7189: static PyObject * THPVariable_expand(PyObject* self_, PyObject* args, PyObject* kwargs)
7190: {
7191:   HANDLE_TH_ERRORS
7192:   const Tensor& self = THPVariable_Unpack(self_);
7193:   static PythonArgParser parser({
7194:     "expand(SymIntArrayRef size, *, bool implicit=False)",
7195:   }, /*traceable=*/true);
7196: 
7197:   ParsedArgs<2> parsed_args;
7198:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7199:   if(_r.has_torch_function()) {
7200:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `THPVariable_erfinv`, `THPVariable_Unpack`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_erfinv`, `THPVariable_Unpack`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7201-7320

```cpp
7201:   }
7202:   // aten::expand(Tensor(a) self, SymInt[] size, *, bool implicit=False) -> Tensor(a)
7203: 
7204:   auto dispatch_expand = [](const at::Tensor & self, c10::SymIntArrayRef size, bool implicit) -> at::Tensor {
7205:     pybind11::gil_scoped_release no_gil;
7206:     return self.expand_symint(size, implicit);
7207:   };
7208:   return wrap(dispatch_expand(self, _r.symintlist(0), _r.toBool(1)));
7209:   Py_RETURN_NONE;
7210:   END_HANDLE_TH_ERRORS
7211: }
7212: 
7213: // expand_as
7214: static PyObject * THPVariable_expand_as(PyObject* self_, PyObject* args, PyObject* kwargs)
7215: {
7216:   HANDLE_TH_ERRORS
7217:   const Tensor& self = THPVariable_Unpack(self_);
7218:   static PythonArgParser parser({
7219:     "expand_as(Tensor other)",
7220:   }, /*traceable=*/true);
7221: 
7222:   ParsedArgs<1> parsed_args;
7223:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7224:   if(_r.has_torch_function()) {
7225:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7226:   }
7227:   // aten::expand_as(Tensor(a) self, Tensor other) -> Tensor(a)
7228: 
7229:   auto dispatch_expand_as = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7230:     pybind11::gil_scoped_release no_gil;
7231:     return self.expand_as(other);
7232:   };
7233:   return wrap(dispatch_expand_as(self, _r.tensor(0)));
7234:   Py_RETURN_NONE;
7235:   END_HANDLE_TH_ERRORS
7236: }
7237: 
7238: // expm1
7239: static PyObject * THPVariable_expm1(PyObject* self_, PyObject* args)
7240: {
7241:   HANDLE_TH_ERRORS
7242:   const Tensor& self = THPVariable_Unpack(self_);
7243:   if (has_torch_function(self_)) {
7244:     return handle_torch_function(self_, "expm1");
7245:   }
7246:   // aten::expm1(Tensor self) -> Tensor
7247: 
7248:   auto dispatch_expm1 = [](const at::Tensor & self) -> at::Tensor {
7249:     pybind11::gil_scoped_release no_gil;
7250:     return self.expm1();
7251:   };
7252:   return wrap(dispatch_expm1(self));
7253:   END_HANDLE_TH_ERRORS
7254: }
7255: 
7256: // expm1_
7257: static PyObject * THPVariable_expm1_(PyObject* self_, PyObject* args)
7258: {
7259:   HANDLE_TH_ERRORS
7260:   const Tensor& self = THPVariable_Unpack(self_);
7261:   if (has_torch_function(self_)) {
7262:     return handle_torch_function(self_, "expm1_");
7263:   }
7264:   // aten::expm1_(Tensor(a!) self) -> Tensor(a!)
7265: 
7266:   auto dispatch_expm1_ = [](const at::Tensor & self) -> at::Tensor {
7267:     pybind11::gil_scoped_release no_gil;
7268:     return self.expm1_();
7269:   };
7270:   return wrap(dispatch_expm1_(self));
7271:   END_HANDLE_TH_ERRORS
7272: }
7273: 
7274: // exponential_
7275: static PyObject * THPVariable_exponential_(PyObject* self_, PyObject* args, PyObject* kwargs)
7276: {
7277:   HANDLE_TH_ERRORS
7278:   const Tensor& self = THPVariable_Unpack(self_);
7279:   static PythonArgParser parser({
7280:     "exponential_(double lambd=1, *, Generator? generator=None)",
7281:   }, /*traceable=*/true);
7282: 
7283:   ParsedArgs<2> parsed_args;
7284:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7285:   if(_r.has_torch_function()) {
7286:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7287:   }
7288:   // aten::exponential_(Tensor(a!) self, float lambd=1, *, Generator? generator=None) -> Tensor(a!)
7289: 
7290:   auto dispatch_exponential_ = [](const at::Tensor & self, double lambd, ::std::optional<at::Generator> generator) -> at::Tensor {
7291:     pybind11::gil_scoped_release no_gil;
7292:     return self.exponential_(lambd, generator);
7293:   };
7294:   return wrap(dispatch_exponential_(self, _r.toDouble(0), _r.generator(1)));
7295:   Py_RETURN_NONE;
7296:   END_HANDLE_TH_ERRORS
7297: }
7298: 
7299: \
7300: // fill_
7301: static PyObject * THPVariable_fill_(PyObject* self_, PyObject* args, PyObject* kwargs)
7302: {
7303:   HANDLE_TH_ERRORS
7304:   const Tensor& self = THPVariable_Unpack(self_);
7305:   static PythonArgParser parser({
7306:     "fill_(Tensor value)",
7307:     "fill_(Scalar value)",
7308:   }, /*traceable=*/true);
7309: 
7310:   ParsedArgs<1> parsed_args;
7311:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7312:   if(_r.has_torch_function()) {
7313:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7314:   }
7315:   switch (_r.idx) {
7316:     case 0: {
7317:       // aten::fill_.Tensor(Tensor(a!) self, Tensor value) -> Tensor(a!)
7318: 
7319:       auto dispatch_fill_ = [](const at::Tensor & self, const at::Tensor & value) -> at::Tensor {
7320:         pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `expand`, `wrap`, `THPVariable_expand_as`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `expand`, `wrap`, `THPVariable_expand_as` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7321-7440

```cpp
7321:         return self.fill_(value);
7322:       };
7323:       return wrap(dispatch_fill_(self, _r.tensor(0)));
7324:     }
7325:     case 1: {
7326:       // aten::fill_.Scalar(Tensor(a!) self, Scalar value) -> Tensor(a!)
7327: 
7328:       auto dispatch_fill_ = [](const at::Tensor & self, const at::Scalar & value) -> at::Tensor {
7329:         pybind11::gil_scoped_release no_gil;
7330:         return self.fill_(value);
7331:       };
7332:       return wrap(dispatch_fill_(self, _r.scalar(0)));
7333:     }
7334:   }
7335:   Py_RETURN_NONE;
7336:   END_HANDLE_TH_ERRORS
7337: }
7338: 
7339: // fill_diagonal_
7340: static PyObject * THPVariable_fill_diagonal_(PyObject* self_, PyObject* args, PyObject* kwargs)
7341: {
7342:   HANDLE_TH_ERRORS
7343:   const Tensor& self = THPVariable_Unpack(self_);
7344:   static PythonArgParser parser({
7345:     "fill_diagonal_(Scalar fill_value, bool wrap=False)",
7346:   }, /*traceable=*/true);
7347: 
7348:   ParsedArgs<2> parsed_args;
7349:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7350:   if(_r.has_torch_function()) {
7351:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7352:   }
7353:   // aten::fill_diagonal_(Tensor(a!) self, Scalar fill_value, bool wrap=False) -> Tensor(a!)
7354: 
7355:   auto dispatch_fill_diagonal_ = [](const at::Tensor & self, const at::Scalar & fill_value, bool wrap) -> at::Tensor {
7356:     pybind11::gil_scoped_release no_gil;
7357:     return self.fill_diagonal_(fill_value, wrap);
7358:   };
7359:   return wrap(dispatch_fill_diagonal_(self, _r.scalar(0), _r.toBool(1)));
7360:   Py_RETURN_NONE;
7361:   END_HANDLE_TH_ERRORS
7362: }
7363: 
7364: // fix
7365: static PyObject * THPVariable_fix(PyObject* self_, PyObject* args)
7366: {
7367:   HANDLE_TH_ERRORS
7368:   const Tensor& self = THPVariable_Unpack(self_);
7369:   if (has_torch_function(self_)) {
7370:     return handle_torch_function(self_, "fix");
7371:   }
7372:   // aten::fix(Tensor self) -> Tensor
7373: 
7374:   auto dispatch_fix = [](const at::Tensor & self) -> at::Tensor {
7375:     pybind11::gil_scoped_release no_gil;
7376:     return self.fix();
7377:   };
7378:   return wrap(dispatch_fix(self));
7379:   END_HANDLE_TH_ERRORS
7380: }
7381: 
7382: // fix_
7383: static PyObject * THPVariable_fix_(PyObject* self_, PyObject* args)
7384: {
7385:   HANDLE_TH_ERRORS
7386:   const Tensor& self = THPVariable_Unpack(self_);
7387:   if (has_torch_function(self_)) {
7388:     return handle_torch_function(self_, "fix_");
7389:   }
7390:   // aten::fix_(Tensor(a!) self) -> Tensor(a!)
7391: 
7392:   auto dispatch_fix_ = [](const at::Tensor & self) -> at::Tensor {
7393:     pybind11::gil_scoped_release no_gil;
7394:     return self.fix_();
7395:   };
7396:   return wrap(dispatch_fix_(self));
7397:   END_HANDLE_TH_ERRORS
7398: }
7399: 
7400: \
7401: // flatten
7402: static PyObject * THPVariable_flatten(PyObject* self_, PyObject* args, PyObject* kwargs)
7403: {
7404:   HANDLE_TH_ERRORS
7405:   const Tensor& self = THPVariable_Unpack(self_);
7406:   static PythonArgParser parser({
7407:     "flatten(int64_t start_dim, int64_t end_dim, Dimname out_dim)",
7408:     "flatten(int64_t start_dim=0, int64_t end_dim=-1)",
7409:     "flatten(Dimname start_dim, Dimname end_dim, Dimname out_dim)",
7410:     "flatten(DimnameList dims, Dimname out_dim)",
7411:   }, /*traceable=*/true);
7412: 
7413:   ParsedArgs<3> parsed_args;
7414:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7415:   if(_r.has_torch_function()) {
7416:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7417:   }
7418:   switch (_r.idx) {
7419:     case 0: {
7420:       // aten::flatten.named_out_dim(Tensor(a) self, int start_dim, int end_dim, Dimname out_dim) -> Tensor(a)
7421: 
7422:       auto dispatch_flatten = [](const at::Tensor & self, int64_t start_dim, int64_t end_dim, at::Dimname out_dim) -> at::Tensor {
7423:         pybind11::gil_scoped_release no_gil;
7424:         return self.flatten(start_dim, end_dim, out_dim);
7425:       };
7426:       return wrap(dispatch_flatten(self, _r.toInt64(0), _r.toInt64(1), _r.dimname(2)));
7427:     }
7428:     case 1: {
7429:       // aten::flatten.using_ints(Tensor(a) self, int start_dim=0, int end_dim=-1) -> Tensor(a)
7430: 
7431:       auto dispatch_flatten = [](const at::Tensor & self, int64_t start_dim, int64_t end_dim) -> at::Tensor {
7432:         pybind11::gil_scoped_release no_gil;
7433:         return self.flatten(start_dim, end_dim);
7434:       };
7435:       return wrap(dispatch_flatten(self, _r.toInt64(0), _r.toInt64(1)));
7436:     }
7437:     case 2: {
7438:       // aten::flatten.using_names(Tensor(a) self, Dimname start_dim, Dimname end_dim, Dimname out_dim) -> Tensor(a)
7439: 
7440:       auto dispatch_flatten = [](const at::Tensor & self, at::Dimname start_dim, at::Dimname end_dim, at::Dimname out_dim) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_fill_diagonal_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_fill_diagonal_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7441-7560

```cpp
7441:         pybind11::gil_scoped_release no_gil;
7442:         return self.flatten(start_dim, end_dim, out_dim);
7443:       };
7444:       return wrap(dispatch_flatten(self, _r.dimname(0), _r.dimname(1), _r.dimname(2)));
7445:     }
7446:     case 3: {
7447:       // aten::flatten.DimnameList(Tensor(a) self, Dimname[] dims, Dimname out_dim) -> Tensor(a)
7448: 
7449:       auto dispatch_flatten = [](const at::Tensor & self, at::DimnameList dims, at::Dimname out_dim) -> at::Tensor {
7450:         pybind11::gil_scoped_release no_gil;
7451:         return self.flatten(dims, out_dim);
7452:       };
7453:       return wrap(dispatch_flatten(self, _r.dimnamelist(0), _r.dimname(1)));
7454:     }
7455:   }
7456:   Py_RETURN_NONE;
7457:   END_HANDLE_TH_ERRORS
7458: }
7459: 
7460: // flip
7461: static PyObject * THPVariable_flip(PyObject* self_, PyObject* args, PyObject* kwargs)
7462: {
7463:   HANDLE_TH_ERRORS
7464:   const Tensor& self = THPVariable_Unpack(self_);
7465:   static PythonArgParser parser({
7466:     "flip(IntArrayRef dims)",
7467:   }, /*traceable=*/true);
7468: 
7469:   ParsedArgs<1> parsed_args;
7470:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7471:   if(_r.has_torch_function()) {
7472:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7473:   }
7474:   // aten::flip(Tensor self, int[] dims) -> Tensor
7475: 
7476:   auto dispatch_flip = [](const at::Tensor & self, at::IntArrayRef dims) -> at::Tensor {
7477:     pybind11::gil_scoped_release no_gil;
7478:     return self.flip(dims);
7479:   };
7480:   return wrap(dispatch_flip(self, _r.intlist(0)));
7481:   Py_RETURN_NONE;
7482:   END_HANDLE_TH_ERRORS
7483: }
7484: 
7485: // fliplr
7486: static PyObject * THPVariable_fliplr(PyObject* self_, PyObject* args)
7487: {
7488:   HANDLE_TH_ERRORS
7489:   const Tensor& self = THPVariable_Unpack(self_);
7490:   if (has_torch_function(self_)) {
7491:     return handle_torch_function(self_, "fliplr");
7492:   }
7493:   // aten::fliplr(Tensor self) -> Tensor
7494: 
7495:   auto dispatch_fliplr = [](const at::Tensor & self) -> at::Tensor {
7496:     pybind11::gil_scoped_release no_gil;
7497:     return self.fliplr();
7498:   };
7499:   return wrap(dispatch_fliplr(self));
7500:   END_HANDLE_TH_ERRORS
7501: }
7502: 
7503: // flipud
7504: static PyObject * THPVariable_flipud(PyObject* self_, PyObject* args)
7505: {
7506:   HANDLE_TH_ERRORS
7507:   const Tensor& self = THPVariable_Unpack(self_);
7508:   if (has_torch_function(self_)) {
7509:     return handle_torch_function(self_, "flipud");
7510:   }
7511:   // aten::flipud(Tensor self) -> Tensor
7512: 
7513:   auto dispatch_flipud = [](const at::Tensor & self) -> at::Tensor {
7514:     pybind11::gil_scoped_release no_gil;
7515:     return self.flipud();
7516:   };
7517:   return wrap(dispatch_flipud(self));
7518:   END_HANDLE_TH_ERRORS
7519: }
7520: 
7521: \
7522: // float_power
7523: static PyObject * THPVariable_float_power(PyObject* self_, PyObject* args, PyObject* kwargs)
7524: {
7525:   HANDLE_TH_ERRORS
7526:   const Tensor& self = THPVariable_Unpack(self_);
7527:   static PythonArgParser parser({
7528:     "float_power(Tensor exponent)",
7529:     "float_power(Scalar exponent)",
7530:   }, /*traceable=*/true);
7531: 
7532:   ParsedArgs<1> parsed_args;
7533:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7534:   if(_r.has_torch_function()) {
7535:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7536:   }
7537:   switch (_r.idx) {
7538:     case 0: {
7539:       // aten::float_power.Tensor_Tensor(Tensor self, Tensor exponent) -> Tensor
7540: 
7541:       auto dispatch_float_power = [](const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
7542:         pybind11::gil_scoped_release no_gil;
7543:         return self.float_power(exponent);
7544:       };
7545:       return wrap(dispatch_float_power(self, _r.tensor(0)));
7546:     }
7547:     case 1: {
7548:       // aten::float_power.Tensor_Scalar(Tensor self, Scalar exponent) -> Tensor
7549: 
7550:       auto dispatch_float_power = [](const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
7551:         pybind11::gil_scoped_release no_gil;
7552:         return self.float_power(exponent);
7553:       };
7554:       return wrap(dispatch_float_power(self, _r.scalar(0)));
7555:     }
7556:   }
7557:   Py_RETURN_NONE;
7558:   END_HANDLE_TH_ERRORS
7559: }
7560: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_flip`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_flip`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 7561-7680

```cpp
7561: \
7562: // float_power_
7563: static PyObject * THPVariable_float_power_(PyObject* self_, PyObject* args, PyObject* kwargs)
7564: {
7565:   HANDLE_TH_ERRORS
7566:   const Tensor& self = THPVariable_Unpack(self_);
7567:   static PythonArgParser parser({
7568:     "float_power_(Tensor exponent)",
7569:     "float_power_(Scalar exponent)",
7570:   }, /*traceable=*/true);
7571: 
7572:   ParsedArgs<1> parsed_args;
7573:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7574:   if(_r.has_torch_function()) {
7575:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7576:   }
7577:   switch (_r.idx) {
7578:     case 0: {
7579:       // aten::float_power_.Tensor(Tensor(a!) self, Tensor exponent) -> Tensor(a!)
7580: 
7581:       auto dispatch_float_power_ = [](const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
7582:         pybind11::gil_scoped_release no_gil;
7583:         return self.float_power_(exponent);
7584:       };
7585:       return wrap(dispatch_float_power_(self, _r.tensor(0)));
7586:     }
7587:     case 1: {
7588:       // aten::float_power_.Scalar(Tensor(a!) self, Scalar exponent) -> Tensor(a!)
7589: 
7590:       auto dispatch_float_power_ = [](const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
7591:         pybind11::gil_scoped_release no_gil;
7592:         return self.float_power_(exponent);
7593:       };
7594:       return wrap(dispatch_float_power_(self, _r.scalar(0)));
7595:     }
7596:   }
7597:   Py_RETURN_NONE;
7598:   END_HANDLE_TH_ERRORS
7599: }
7600: 
7601: // floor
7602: static PyObject * THPVariable_floor(PyObject* self_, PyObject* args)
7603: {
7604:   HANDLE_TH_ERRORS
7605:   const Tensor& self = THPVariable_Unpack(self_);
7606:   if (has_torch_function(self_)) {
7607:     return handle_torch_function(self_, "floor");
7608:   }
7609:   // aten::floor(Tensor self) -> Tensor
7610: 
7611:   auto dispatch_floor = [](const at::Tensor & self) -> at::Tensor {
7612:     pybind11::gil_scoped_release no_gil;
7613:     return self.floor();
7614:   };
7615:   return wrap(dispatch_floor(self));
7616:   END_HANDLE_TH_ERRORS
7617: }
7618: 
7619: // floor_
7620: static PyObject * THPVariable_floor_(PyObject* self_, PyObject* args)
7621: {
7622:   HANDLE_TH_ERRORS
7623:   const Tensor& self = THPVariable_Unpack(self_);
7624:   if (has_torch_function(self_)) {
7625:     return handle_torch_function(self_, "floor_");
7626:   }
7627:   // aten::floor_(Tensor(a!) self) -> Tensor(a!)
7628: 
7629:   auto dispatch_floor_ = [](const at::Tensor & self) -> at::Tensor {
7630:     pybind11::gil_scoped_release no_gil;
7631:     return self.floor_();
7632:   };
7633:   return wrap(dispatch_floor_(self));
7634:   END_HANDLE_TH_ERRORS
7635: }
7636: 
7637: \
7638: // floor_divide
7639: static PyObject * THPVariable_floor_divide(PyObject* self_, PyObject* args, PyObject* kwargs)
7640: {
7641:   HANDLE_TH_ERRORS
7642:   const Tensor& self = THPVariable_Unpack(self_);
7643:   static PythonArgParser parser({
7644:     "floor_divide(Tensor other)",
7645:     "floor_divide(Scalar other)",
7646:   }, /*traceable=*/true);
7647: 
7648:   ParsedArgs<1> parsed_args;
7649:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7650:   if(_r.has_torch_function()) {
7651:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7652:   }
7653:   switch (_r.idx) {
7654:     case 0: {
7655:       // aten::floor_divide(Tensor self, Tensor other) -> Tensor
7656: 
7657:       auto dispatch_floor_divide = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7658:         pybind11::gil_scoped_release no_gil;
7659:         return self.floor_divide(other);
7660:       };
7661:       return wrap(dispatch_floor_divide(self, _r.tensor(0)));
7662:     }
7663:     case 1: {
7664:       // aten::floor_divide.Scalar(Tensor self, Scalar other) -> Tensor
7665: 
7666:       auto dispatch_floor_divide = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
7667:         pybind11::gil_scoped_release no_gil;
7668:         return self.floor_divide(other);
7669:       };
7670:       return wrap(dispatch_floor_divide(self, _r.scalar(0)));
7671:     }
7672:   }
7673:   Py_RETURN_NONE;
7674:   END_HANDLE_TH_ERRORS
7675: }
7676: 
7677: \
7678: // floor_divide_
7679: static PyObject * THPVariable_floor_divide_(PyObject* self_, PyObject* args, PyObject* kwargs)
7680: {
```

- EN: The main execution path in this span is carried by `THPVariable_float_power_`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_float_power_`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7681-7800

```cpp
7681:   HANDLE_TH_ERRORS
7682:   const Tensor& self = THPVariable_Unpack(self_);
7683:   static PythonArgParser parser({
7684:     "floor_divide_(Tensor other)",
7685:     "floor_divide_(Scalar other)",
7686:   }, /*traceable=*/true);
7687: 
7688:   ParsedArgs<1> parsed_args;
7689:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7690:   if(_r.has_torch_function()) {
7691:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7692:   }
7693:   switch (_r.idx) {
7694:     case 0: {
7695:       // aten::floor_divide_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
7696: 
7697:       auto dispatch_floor_divide_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7698:         pybind11::gil_scoped_release no_gil;
7699:         return self.floor_divide_(other);
7700:       };
7701:       return wrap(dispatch_floor_divide_(self, _r.tensor(0)));
7702:     }
7703:     case 1: {
7704:       // aten::floor_divide_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
7705: 
7706:       auto dispatch_floor_divide_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
7707:         pybind11::gil_scoped_release no_gil;
7708:         return self.floor_divide_(other);
7709:       };
7710:       return wrap(dispatch_floor_divide_(self, _r.scalar(0)));
7711:     }
7712:   }
7713:   Py_RETURN_NONE;
7714:   END_HANDLE_TH_ERRORS
7715: }
7716: 
7717: // fmax
7718: static PyObject * THPVariable_fmax(PyObject* self_, PyObject* args, PyObject* kwargs)
7719: {
7720:   HANDLE_TH_ERRORS
7721:   const Tensor& self = THPVariable_Unpack(self_);
7722:   static PythonArgParser parser({
7723:     "fmax(Tensor other)",
7724:   }, /*traceable=*/true);
7725: 
7726:   ParsedArgs<1> parsed_args;
7727:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7728:   if(_r.has_torch_function()) {
7729:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7730:   }
7731:   // aten::fmax(Tensor self, Tensor other) -> Tensor
7732: 
7733:   auto dispatch_fmax = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7734:     pybind11::gil_scoped_release no_gil;
7735:     return self.fmax(other);
7736:   };
7737:   return wrap(dispatch_fmax(self, _r.tensor(0)));
7738:   Py_RETURN_NONE;
7739:   END_HANDLE_TH_ERRORS
7740: }
7741: 
7742: // fmin
7743: static PyObject * THPVariable_fmin(PyObject* self_, PyObject* args, PyObject* kwargs)
7744: {
7745:   HANDLE_TH_ERRORS
7746:   const Tensor& self = THPVariable_Unpack(self_);
7747:   static PythonArgParser parser({
7748:     "fmin(Tensor other)",
7749:   }, /*traceable=*/true);
7750: 
7751:   ParsedArgs<1> parsed_args;
7752:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7753:   if(_r.has_torch_function()) {
7754:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7755:   }
7756:   // aten::fmin(Tensor self, Tensor other) -> Tensor
7757: 
7758:   auto dispatch_fmin = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7759:     pybind11::gil_scoped_release no_gil;
7760:     return self.fmin(other);
7761:   };
7762:   return wrap(dispatch_fmin(self, _r.tensor(0)));
7763:   Py_RETURN_NONE;
7764:   END_HANDLE_TH_ERRORS
7765: }
7766: 
7767: \
7768: // fmod
7769: static PyObject * THPVariable_fmod(PyObject* self_, PyObject* args, PyObject* kwargs)
7770: {
7771:   HANDLE_TH_ERRORS
7772:   const Tensor& self = THPVariable_Unpack(self_);
7773:   static PythonArgParser parser({
7774:     "fmod(Tensor other)",
7775:     "fmod(Scalar other)",
7776:   }, /*traceable=*/true);
7777: 
7778:   ParsedArgs<1> parsed_args;
7779:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7780:   if(_r.has_torch_function()) {
7781:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7782:   }
7783:   switch (_r.idx) {
7784:     case 0: {
7785:       // aten::fmod.Tensor(Tensor self, Tensor other) -> Tensor
7786: 
7787:       auto dispatch_fmod = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7788:         pybind11::gil_scoped_release no_gil;
7789:         return self.fmod(other);
7790:       };
7791:       return wrap(dispatch_fmod(self, _r.tensor(0)));
7792:     }
7793:     case 1: {
7794:       // aten::fmod.Scalar(Tensor self, Scalar other) -> Tensor
7795: 
7796:       auto dispatch_fmod = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
7797:         pybind11::gil_scoped_release no_gil;
7798:         return self.fmod(other);
7799:       };
7800:       return wrap(dispatch_fmod(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7801-7920

```cpp
7801:     }
7802:   }
7803:   Py_RETURN_NONE;
7804:   END_HANDLE_TH_ERRORS
7805: }
7806: 
7807: \
7808: // fmod_
7809: static PyObject * THPVariable_fmod_(PyObject* self_, PyObject* args, PyObject* kwargs)
7810: {
7811:   HANDLE_TH_ERRORS
7812:   const Tensor& self = THPVariable_Unpack(self_);
7813:   static PythonArgParser parser({
7814:     "fmod_(Tensor other)",
7815:     "fmod_(Scalar other)",
7816:   }, /*traceable=*/true);
7817: 
7818:   ParsedArgs<1> parsed_args;
7819:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7820:   if(_r.has_torch_function()) {
7821:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7822:   }
7823:   switch (_r.idx) {
7824:     case 0: {
7825:       // aten::fmod_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
7826: 
7827:       auto dispatch_fmod_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7828:         pybind11::gil_scoped_release no_gil;
7829:         return self.fmod_(other);
7830:       };
7831:       return wrap(dispatch_fmod_(self, _r.tensor(0)));
7832:     }
7833:     case 1: {
7834:       // aten::fmod_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
7835: 
7836:       auto dispatch_fmod_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
7837:         pybind11::gil_scoped_release no_gil;
7838:         return self.fmod_(other);
7839:       };
7840:       return wrap(dispatch_fmod_(self, _r.scalar(0)));
7841:     }
7842:   }
7843:   Py_RETURN_NONE;
7844:   END_HANDLE_TH_ERRORS
7845: }
7846: 
7847: // frac
7848: static PyObject * THPVariable_frac(PyObject* self_, PyObject* args)
7849: {
7850:   HANDLE_TH_ERRORS
7851:   const Tensor& self = THPVariable_Unpack(self_);
7852:   if (has_torch_function(self_)) {
7853:     return handle_torch_function(self_, "frac");
7854:   }
7855:   // aten::frac(Tensor self) -> Tensor
7856: 
7857:   auto dispatch_frac = [](const at::Tensor & self) -> at::Tensor {
7858:     pybind11::gil_scoped_release no_gil;
7859:     return self.frac();
7860:   };
7861:   return wrap(dispatch_frac(self));
7862:   END_HANDLE_TH_ERRORS
7863: }
7864: 
7865: // frac_
7866: static PyObject * THPVariable_frac_(PyObject* self_, PyObject* args)
7867: {
7868:   HANDLE_TH_ERRORS
7869:   const Tensor& self = THPVariable_Unpack(self_);
7870:   if (has_torch_function(self_)) {
7871:     return handle_torch_function(self_, "frac_");
7872:   }
7873:   // aten::frac_(Tensor(a!) self) -> Tensor(a!)
7874: 
7875:   auto dispatch_frac_ = [](const at::Tensor & self) -> at::Tensor {
7876:     pybind11::gil_scoped_release no_gil;
7877:     return self.frac_();
7878:   };
7879:   return wrap(dispatch_frac_(self));
7880:   END_HANDLE_TH_ERRORS
7881: }
7882: 
7883: // frexp
7884: static PyObject * THPVariable_frexp(PyObject* self_, PyObject* args)
7885: {
7886:   HANDLE_TH_ERRORS
7887:   static PyTypeObject* NamedTuple = generated::get_frexp_structseq();
7888:   const Tensor& self = THPVariable_Unpack(self_);
7889:   if (has_torch_function(self_)) {
7890:     return handle_torch_function(self_, "frexp");
7891:   }
7892:   // aten::frexp.Tensor(Tensor self) -> (Tensor mantissa, Tensor exponent)
7893: 
7894:   auto dispatch_frexp = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
7895:     pybind11::gil_scoped_release no_gil;
7896:     return self.frexp();
7897:   };
7898:   return wrap(NamedTuple, dispatch_frexp(self));
7899:   END_HANDLE_TH_ERRORS
7900: }
7901: 
7902: \
7903: // gather
7904: static PyObject * THPVariable_gather(PyObject* self_, PyObject* args, PyObject* kwargs)
7905: {
7906:   HANDLE_TH_ERRORS
7907:   const Tensor& self = THPVariable_Unpack(self_);
7908:   static PythonArgParser parser({
7909:     "gather(int64_t dim, Tensor index, *, bool sparse_grad=False)",
7910:     "gather(Dimname dim, Tensor index, *, bool sparse_grad=False)",
7911:   }, /*traceable=*/true);
7912: 
7913:   ParsedArgs<3> parsed_args;
7914:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7915:   if(_r.has_torch_function()) {
7916:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7917:   }
7918:   switch (_r.idx) {
7919:     case 0: {
7920:       // aten::gather(Tensor self, int dim, Tensor index, *, bool sparse_grad=False) -> Tensor
```

- EN: The main execution path in this span is carried by `THPVariable_fmod_`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_fmod_`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 7921-8040

```cpp
7921: 
7922:       auto dispatch_gather = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, bool sparse_grad) -> at::Tensor {
7923:         pybind11::gil_scoped_release no_gil;
7924:         return self.gather(dim, index, sparse_grad);
7925:       };
7926:       return wrap(dispatch_gather(self, _r.toInt64(0), _r.tensor(1), _r.toBool(2)));
7927:     }
7928:     case 1: {
7929:       // aten::gather.dimname(Tensor self, Dimname dim, Tensor index, *, bool sparse_grad=False) -> Tensor
7930: 
7931:       auto dispatch_gather = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, bool sparse_grad) -> at::Tensor {
7932:         pybind11::gil_scoped_release no_gil;
7933:         return self.gather(dim, index, sparse_grad);
7934:       };
7935:       return wrap(dispatch_gather(self, _r.dimname(0), _r.tensor(1), _r.toBool(2)));
7936:     }
7937:   }
7938:   Py_RETURN_NONE;
7939:   END_HANDLE_TH_ERRORS
7940: }
7941: 
7942: // gcd
7943: static PyObject * THPVariable_gcd(PyObject* self_, PyObject* args, PyObject* kwargs)
7944: {
7945:   HANDLE_TH_ERRORS
7946:   const Tensor& self = THPVariable_Unpack(self_);
7947:   static PythonArgParser parser({
7948:     "gcd(Tensor other)",
7949:   }, /*traceable=*/true);
7950: 
7951:   ParsedArgs<1> parsed_args;
7952:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7953:   if(_r.has_torch_function()) {
7954:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7955:   }
7956:   // aten::gcd(Tensor self, Tensor other) -> Tensor
7957: 
7958:   auto dispatch_gcd = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7959:     pybind11::gil_scoped_release no_gil;
7960:     return self.gcd(other);
7961:   };
7962:   return wrap(dispatch_gcd(self, _r.tensor(0)));
7963:   Py_RETURN_NONE;
7964:   END_HANDLE_TH_ERRORS
7965: }
7966: 
7967: // gcd_
7968: static PyObject * THPVariable_gcd_(PyObject* self_, PyObject* args, PyObject* kwargs)
7969: {
7970:   HANDLE_TH_ERRORS
7971:   const Tensor& self = THPVariable_Unpack(self_);
7972:   static PythonArgParser parser({
7973:     "gcd_(Tensor other)",
7974:   }, /*traceable=*/true);
7975: 
7976:   ParsedArgs<1> parsed_args;
7977:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
7978:   if(_r.has_torch_function()) {
7979:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
7980:   }
7981:   // aten::gcd_(Tensor(a!) self, Tensor other) -> Tensor(a!)
7982: 
7983:   auto dispatch_gcd_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
7984:     pybind11::gil_scoped_release no_gil;
7985:     return self.gcd_(other);
7986:   };
7987:   return wrap(dispatch_gcd_(self, _r.tensor(0)));
7988:   Py_RETURN_NONE;
7989:   END_HANDLE_TH_ERRORS
7990: }
7991: 
7992: \
7993: // ge
7994: static PyObject * THPVariable_ge(PyObject* self_, PyObject* args, PyObject* kwargs)
7995: {
7996:   HANDLE_TH_ERRORS
7997:   const Tensor& self = THPVariable_Unpack(self_);
7998:   static PythonArgParser parser({
7999:     "ge(Tensor other)",
8000:     "ge(Scalar other)",
8001:   }, /*traceable=*/true);
8002: 
8003:   ParsedArgs<1> parsed_args;
8004:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8005:   if(_r.has_torch_function()) {
8006:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8007:   }
8008:   switch (_r.idx) {
8009:     case 0: {
8010:       // aten::ge.Tensor(Tensor self, Tensor other) -> Tensor
8011: 
8012:       auto dispatch_ge = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8013:         pybind11::gil_scoped_release no_gil;
8014:         return self.ge(other);
8015:       };
8016:       return wrap(dispatch_ge(self, _r.tensor(0)));
8017:     }
8018:     case 1: {
8019:       // aten::ge.Scalar(Tensor self, Scalar other) -> Tensor
8020: 
8021:       auto dispatch_ge = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8022:         pybind11::gil_scoped_release no_gil;
8023:         return self.ge(other);
8024:       };
8025:       return wrap(dispatch_ge(self, _r.scalar(0)));
8026:     }
8027:   }
8028:   Py_RETURN_NONE;
8029:   END_HANDLE_TH_ERRORS
8030: }
8031: 
8032: \
8033: // ge_
8034: static PyObject * THPVariable_ge_(PyObject* self_, PyObject* args, PyObject* kwargs)
8035: {
8036:   HANDLE_TH_ERRORS
8037:   const Tensor& self = THPVariable_Unpack(self_);
8038:   static PythonArgParser parser({
8039:     "ge_(Tensor other)",
8040:     "ge_(Scalar other)",
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_gcd`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_gcd`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8041-8160

```cpp
8041:   }, /*traceable=*/true);
8042: 
8043:   ParsedArgs<1> parsed_args;
8044:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8045:   if(_r.has_torch_function()) {
8046:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8047:   }
8048:   switch (_r.idx) {
8049:     case 0: {
8050:       // aten::ge_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
8051: 
8052:       auto dispatch_ge_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8053:         pybind11::gil_scoped_release no_gil;
8054:         return self.ge_(other);
8055:       };
8056:       return wrap(dispatch_ge_(self, _r.tensor(0)));
8057:     }
8058:     case 1: {
8059:       // aten::ge_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
8060: 
8061:       auto dispatch_ge_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8062:         pybind11::gil_scoped_release no_gil;
8063:         return self.ge_(other);
8064:       };
8065:       return wrap(dispatch_ge_(self, _r.scalar(0)));
8066:     }
8067:   }
8068:   Py_RETURN_NONE;
8069:   END_HANDLE_TH_ERRORS
8070: }
8071: 
8072: // geometric_
8073: static PyObject * THPVariable_geometric_(PyObject* self_, PyObject* args, PyObject* kwargs)
8074: {
8075:   HANDLE_TH_ERRORS
8076:   const Tensor& self = THPVariable_Unpack(self_);
8077:   static PythonArgParser parser({
8078:     "geometric_(double p, *, Generator? generator=None)",
8079:   }, /*traceable=*/true);
8080: 
8081:   ParsedArgs<2> parsed_args;
8082:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8083:   if(_r.has_torch_function()) {
8084:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8085:   }
8086:   // aten::geometric_(Tensor(a!) self, float p, *, Generator? generator=None) -> Tensor(a!)
8087: 
8088:   auto dispatch_geometric_ = [](const at::Tensor & self, double p, ::std::optional<at::Generator> generator) -> at::Tensor {
8089:     pybind11::gil_scoped_release no_gil;
8090:     return self.geometric_(p, generator);
8091:   };
8092:   return wrap(dispatch_geometric_(self, _r.toDouble(0), _r.generator(1)));
8093:   Py_RETURN_NONE;
8094:   END_HANDLE_TH_ERRORS
8095: }
8096: 
8097: // geqrf
8098: static PyObject * THPVariable_geqrf(PyObject* self_, PyObject* args)
8099: {
8100:   HANDLE_TH_ERRORS
8101:   static PyTypeObject* NamedTuple = generated::get_geqrf_structseq();
8102:   const Tensor& self = THPVariable_Unpack(self_);
8103:   if (has_torch_function(self_)) {
8104:     return handle_torch_function(self_, "geqrf");
8105:   }
8106:   // aten::geqrf(Tensor self) -> (Tensor a, Tensor tau)
8107: 
8108:   auto dispatch_geqrf = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
8109:     pybind11::gil_scoped_release no_gil;
8110:     return self.geqrf();
8111:   };
8112:   return wrap(NamedTuple, dispatch_geqrf(self));
8113:   END_HANDLE_TH_ERRORS
8114: }
8115: 
8116: // ger
8117: static PyObject * THPVariable_ger(PyObject* self_, PyObject* args, PyObject* kwargs)
8118: {
8119:   HANDLE_TH_ERRORS
8120:   const Tensor& self = THPVariable_Unpack(self_);
8121:   static PythonArgParser parser({
8122:     "ger(Tensor vec2)",
8123:   }, /*traceable=*/true);
8124: 
8125:   ParsedArgs<1> parsed_args;
8126:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8127:   if(_r.has_torch_function()) {
8128:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8129:   }
8130:   // aten::ger(Tensor self, Tensor vec2) -> Tensor
8131: 
8132:   auto dispatch_ger = [](const at::Tensor & self, const at::Tensor & vec2) -> at::Tensor {
8133:     pybind11::gil_scoped_release no_gil;
8134:     return self.ger(vec2);
8135:   };
8136:   return wrap(dispatch_ger(self, _r.tensor(0)));
8137:   Py_RETURN_NONE;
8138:   END_HANDLE_TH_ERRORS
8139: }
8140: 
8141: \
8142: // greater
8143: static PyObject * THPVariable_greater(PyObject* self_, PyObject* args, PyObject* kwargs)
8144: {
8145:   HANDLE_TH_ERRORS
8146:   const Tensor& self = THPVariable_Unpack(self_);
8147:   static PythonArgParser parser({
8148:     "greater(Tensor other)",
8149:     "greater(Scalar other)",
8150:   }, /*traceable=*/true);
8151: 
8152:   ParsedArgs<1> parsed_args;
8153:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8154:   if(_r.has_torch_function()) {
8155:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8156:   }
8157:   switch (_r.idx) {
8158:     case 0: {
8159:       // aten::greater.Tensor(Tensor self, Tensor other) -> Tensor
8160: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable_geometric_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable_geometric_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8161-8280

```cpp
8161:       auto dispatch_greater = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8162:         pybind11::gil_scoped_release no_gil;
8163:         return self.greater(other);
8164:       };
8165:       return wrap(dispatch_greater(self, _r.tensor(0)));
8166:     }
8167:     case 1: {
8168:       // aten::greater.Scalar(Tensor self, Scalar other) -> Tensor
8169: 
8170:       auto dispatch_greater = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8171:         pybind11::gil_scoped_release no_gil;
8172:         return self.greater(other);
8173:       };
8174:       return wrap(dispatch_greater(self, _r.scalar(0)));
8175:     }
8176:   }
8177:   Py_RETURN_NONE;
8178:   END_HANDLE_TH_ERRORS
8179: }
8180: 
8181: \
8182: // greater_
8183: static PyObject * THPVariable_greater_(PyObject* self_, PyObject* args, PyObject* kwargs)
8184: {
8185:   HANDLE_TH_ERRORS
8186:   const Tensor& self = THPVariable_Unpack(self_);
8187:   static PythonArgParser parser({
8188:     "greater_(Tensor other)",
8189:     "greater_(Scalar other)",
8190:   }, /*traceable=*/true);
8191: 
8192:   ParsedArgs<1> parsed_args;
8193:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8194:   if(_r.has_torch_function()) {
8195:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8196:   }
8197:   switch (_r.idx) {
8198:     case 0: {
8199:       // aten::greater_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
8200: 
8201:       auto dispatch_greater_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8202:         pybind11::gil_scoped_release no_gil;
8203:         return self.greater_(other);
8204:       };
8205:       return wrap(dispatch_greater_(self, _r.tensor(0)));
8206:     }
8207:     case 1: {
8208:       // aten::greater_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
8209: 
8210:       auto dispatch_greater_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8211:         pybind11::gil_scoped_release no_gil;
8212:         return self.greater_(other);
8213:       };
8214:       return wrap(dispatch_greater_(self, _r.scalar(0)));
8215:     }
8216:   }
8217:   Py_RETURN_NONE;
8218:   END_HANDLE_TH_ERRORS
8219: }
8220: 
8221: \
8222: // greater_equal
8223: static PyObject * THPVariable_greater_equal(PyObject* self_, PyObject* args, PyObject* kwargs)
8224: {
8225:   HANDLE_TH_ERRORS
8226:   const Tensor& self = THPVariable_Unpack(self_);
8227:   static PythonArgParser parser({
8228:     "greater_equal(Tensor other)",
8229:     "greater_equal(Scalar other)",
8230:   }, /*traceable=*/true);
8231: 
8232:   ParsedArgs<1> parsed_args;
8233:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8234:   if(_r.has_torch_function()) {
8235:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8236:   }
8237:   switch (_r.idx) {
8238:     case 0: {
8239:       // aten::greater_equal.Tensor(Tensor self, Tensor other) -> Tensor
8240: 
8241:       auto dispatch_greater_equal = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8242:         pybind11::gil_scoped_release no_gil;
8243:         return self.greater_equal(other);
8244:       };
8245:       return wrap(dispatch_greater_equal(self, _r.tensor(0)));
8246:     }
8247:     case 1: {
8248:       // aten::greater_equal.Scalar(Tensor self, Scalar other) -> Tensor
8249: 
8250:       auto dispatch_greater_equal = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8251:         pybind11::gil_scoped_release no_gil;
8252:         return self.greater_equal(other);
8253:       };
8254:       return wrap(dispatch_greater_equal(self, _r.scalar(0)));
8255:     }
8256:   }
8257:   Py_RETURN_NONE;
8258:   END_HANDLE_TH_ERRORS
8259: }
8260: 
8261: \
8262: // greater_equal_
8263: static PyObject * THPVariable_greater_equal_(PyObject* self_, PyObject* args, PyObject* kwargs)
8264: {
8265:   HANDLE_TH_ERRORS
8266:   const Tensor& self = THPVariable_Unpack(self_);
8267:   static PythonArgParser parser({
8268:     "greater_equal_(Tensor other)",
8269:     "greater_equal_(Scalar other)",
8270:   }, /*traceable=*/true);
8271: 
8272:   ParsedArgs<1> parsed_args;
8273:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8274:   if(_r.has_torch_function()) {
8275:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8276:   }
8277:   switch (_r.idx) {
8278:     case 0: {
8279:       // aten::greater_equal_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
8280: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_greater_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_greater_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8281-8400

```cpp
8281:       auto dispatch_greater_equal_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8282:         pybind11::gil_scoped_release no_gil;
8283:         return self.greater_equal_(other);
8284:       };
8285:       return wrap(dispatch_greater_equal_(self, _r.tensor(0)));
8286:     }
8287:     case 1: {
8288:       // aten::greater_equal_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
8289: 
8290:       auto dispatch_greater_equal_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8291:         pybind11::gil_scoped_release no_gil;
8292:         return self.greater_equal_(other);
8293:       };
8294:       return wrap(dispatch_greater_equal_(self, _r.scalar(0)));
8295:     }
8296:   }
8297:   Py_RETURN_NONE;
8298:   END_HANDLE_TH_ERRORS
8299: }
8300: 
8301: \
8302: // gt
8303: static PyObject * THPVariable_gt(PyObject* self_, PyObject* args, PyObject* kwargs)
8304: {
8305:   HANDLE_TH_ERRORS
8306:   const Tensor& self = THPVariable_Unpack(self_);
8307:   static PythonArgParser parser({
8308:     "gt(Tensor other)",
8309:     "gt(Scalar other)",
8310:   }, /*traceable=*/true);
8311: 
8312:   ParsedArgs<1> parsed_args;
8313:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8314:   if(_r.has_torch_function()) {
8315:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8316:   }
8317:   switch (_r.idx) {
8318:     case 0: {
8319:       // aten::gt.Tensor(Tensor self, Tensor other) -> Tensor
8320: 
8321:       auto dispatch_gt = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8322:         pybind11::gil_scoped_release no_gil;
8323:         return self.gt(other);
8324:       };
8325:       return wrap(dispatch_gt(self, _r.tensor(0)));
8326:     }
8327:     case 1: {
8328:       // aten::gt.Scalar(Tensor self, Scalar other) -> Tensor
8329: 
8330:       auto dispatch_gt = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8331:         pybind11::gil_scoped_release no_gil;
8332:         return self.gt(other);
8333:       };
8334:       return wrap(dispatch_gt(self, _r.scalar(0)));
8335:     }
8336:   }
8337:   Py_RETURN_NONE;
8338:   END_HANDLE_TH_ERRORS
8339: }
8340: 
8341: \
8342: // gt_
8343: static PyObject * THPVariable_gt_(PyObject* self_, PyObject* args, PyObject* kwargs)
8344: {
8345:   HANDLE_TH_ERRORS
8346:   const Tensor& self = THPVariable_Unpack(self_);
8347:   static PythonArgParser parser({
8348:     "gt_(Tensor other)",
8349:     "gt_(Scalar other)",
8350:   }, /*traceable=*/true);
8351: 
8352:   ParsedArgs<1> parsed_args;
8353:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8354:   if(_r.has_torch_function()) {
8355:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8356:   }
8357:   switch (_r.idx) {
8358:     case 0: {
8359:       // aten::gt_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
8360: 
8361:       auto dispatch_gt_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8362:         pybind11::gil_scoped_release no_gil;
8363:         return self.gt_(other);
8364:       };
8365:       return wrap(dispatch_gt_(self, _r.tensor(0)));
8366:     }
8367:     case 1: {
8368:       // aten::gt_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
8369: 
8370:       auto dispatch_gt_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
8371:         pybind11::gil_scoped_release no_gil;
8372:         return self.gt_(other);
8373:       };
8374:       return wrap(dispatch_gt_(self, _r.scalar(0)));
8375:     }
8376:   }
8377:   Py_RETURN_NONE;
8378:   END_HANDLE_TH_ERRORS
8379: }
8380: 
8381: // hardshrink
8382: static PyObject * THPVariable_hardshrink(PyObject* self_, PyObject* args, PyObject* kwargs)
8383: {
8384:   HANDLE_TH_ERRORS
8385:   const Tensor& self = THPVariable_Unpack(self_);
8386:   static PythonArgParser parser({
8387:     "hardshrink(Scalar lambd=0.5)",
8388:   }, /*traceable=*/true);
8389: 
8390:   ParsedArgs<1> parsed_args;
8391:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8392:   if(_r.has_torch_function()) {
8393:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8394:   }
8395:   // aten::hardshrink(Tensor self, Scalar lambd=0.5) -> Tensor
8396: 
8397:   auto dispatch_hardshrink = [](const at::Tensor & self, const at::Scalar & lambd) -> at::Tensor {
8398:     pybind11::gil_scoped_release no_gil;
8399:     return self.hardshrink(lambd);
8400:   };
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_gt`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_gt`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8401-8520

```cpp
8401:   return wrap(dispatch_hardshrink(self, _r.scalar(0)));
8402:   Py_RETURN_NONE;
8403:   END_HANDLE_TH_ERRORS
8404: }
8405: 
8406: // hash_tensor
8407: static PyObject * THPVariable_hash_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
8408: {
8409:   HANDLE_TH_ERRORS
8410:   const Tensor& self = THPVariable_Unpack(self_);
8411:   static PythonArgParser parser({
8412:     "hash_tensor(IntArrayRef[1] dim=None, *, bool keepdim=False, int64_t mode=0)",
8413:   }, /*traceable=*/true);
8414: 
8415:   ParsedArgs<3> parsed_args;
8416:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8417:   if(_r.has_torch_function()) {
8418:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8419:   }
8420:   // aten::hash_tensor(Tensor self, int[1] dim=[], *, bool keepdim=False, int mode=0) -> Tensor
8421: 
8422:   auto dispatch_hash_tensor = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim, int64_t mode) -> at::Tensor {
8423:     pybind11::gil_scoped_release no_gil;
8424:     return self.hash_tensor(dim, keepdim, mode);
8425:   };
8426:   return wrap(dispatch_hash_tensor(self, _r.intlist(0), _r.toBool(1), _r.toInt64(2)));
8427:   Py_RETURN_NONE;
8428:   END_HANDLE_TH_ERRORS
8429: }
8430: 
8431: // heaviside
8432: static PyObject * THPVariable_heaviside(PyObject* self_, PyObject* args, PyObject* kwargs)
8433: {
8434:   HANDLE_TH_ERRORS
8435:   const Tensor& self = THPVariable_Unpack(self_);
8436:   static PythonArgParser parser({
8437:     "heaviside(Tensor values)",
8438:   }, /*traceable=*/true);
8439: 
8440:   ParsedArgs<1> parsed_args;
8441:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8442:   if(_r.has_torch_function()) {
8443:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8444:   }
8445:   // aten::heaviside(Tensor self, Tensor values) -> Tensor
8446: 
8447:   auto dispatch_heaviside = [](const at::Tensor & self, const at::Tensor & values) -> at::Tensor {
8448:     pybind11::gil_scoped_release no_gil;
8449:     return self.heaviside(values);
8450:   };
8451:   return wrap(dispatch_heaviside(self, _r.tensor(0)));
8452:   Py_RETURN_NONE;
8453:   END_HANDLE_TH_ERRORS
8454: }
8455: 
8456: // heaviside_
8457: static PyObject * THPVariable_heaviside_(PyObject* self_, PyObject* args, PyObject* kwargs)
8458: {
8459:   HANDLE_TH_ERRORS
8460:   const Tensor& self = THPVariable_Unpack(self_);
8461:   static PythonArgParser parser({
8462:     "heaviside_(Tensor values)",
8463:   }, /*traceable=*/true);
8464: 
8465:   ParsedArgs<1> parsed_args;
8466:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8467:   if(_r.has_torch_function()) {
8468:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8469:   }
8470:   // aten::heaviside_(Tensor(a!) self, Tensor values) -> Tensor(a!)
8471: 
8472:   auto dispatch_heaviside_ = [](const at::Tensor & self, const at::Tensor & values) -> at::Tensor {
8473:     pybind11::gil_scoped_release no_gil;
8474:     return self.heaviside_(values);
8475:   };
8476:   return wrap(dispatch_heaviside_(self, _r.tensor(0)));
8477:   Py_RETURN_NONE;
8478:   END_HANDLE_TH_ERRORS
8479: }
8480: 
8481: // histc
8482: static PyObject * THPVariable_histc(PyObject* self_, PyObject* args, PyObject* kwargs)
8483: {
8484:   HANDLE_TH_ERRORS
8485:   const Tensor& self = THPVariable_Unpack(self_);
8486:   static PythonArgParser parser({
8487:     "histc(int64_t bins=100, Scalar min=0, Scalar max=0)",
8488:   }, /*traceable=*/true);
8489: 
8490:   ParsedArgs<3> parsed_args;
8491:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8492:   if(_r.has_torch_function()) {
8493:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8494:   }
8495:   // aten::histc(Tensor self, int bins=100, Scalar min=0, Scalar max=0) -> Tensor
8496: 
8497:   auto dispatch_histc = [](const at::Tensor & self, int64_t bins, const at::Scalar & min, const at::Scalar & max) -> at::Tensor {
8498:     pybind11::gil_scoped_release no_gil;
8499:     return self.histc(bins, min, max);
8500:   };
8501:   return wrap(dispatch_histc(self, _r.toInt64(0), _r.scalar(1), _r.scalar(2)));
8502:   Py_RETURN_NONE;
8503:   END_HANDLE_TH_ERRORS
8504: }
8505: 
8506: \
8507: // histogram
8508: static PyObject * THPVariable_histogram(PyObject* self_, PyObject* args, PyObject* kwargs)
8509: {
8510:   HANDLE_TH_ERRORS
8511:   static PyTypeObject* NamedTuple = generated::get_histogram_structseq();
8512:   const Tensor& self = THPVariable_Unpack(self_);
8513:   static PythonArgParser parser({
8514:     "histogram(Tensor bins, *, Tensor? weight=None, bool density=False)",
8515:     "histogram(int64_t bins=100, *, ArrayRef<double>? range=None, Tensor? weight=None, bool density=False)",
8516:   }, /*traceable=*/true);
8517: 
8518:   ParsedArgs<4> parsed_args;
8519:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8520:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_hash_tensor`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_hash_tensor`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 8521-8640

```cpp
8521:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8522:   }
8523:   switch (_r.idx) {
8524:     case 0: {
8525:       // aten::histogram.bins_tensor(Tensor self, Tensor bins, *, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor bin_edges)
8526: 
8527:       auto dispatch_histogram = [](const at::Tensor & self, const at::Tensor & bins, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,at::Tensor> {
8528:         pybind11::gil_scoped_release no_gil;
8529:         return self.histogram(bins, weight, density);
8530:       };
8531:       return wrap(NamedTuple, dispatch_histogram(self, _r.tensor(0), _r.optionalTensor(1), _r.toBool(2)));
8532:     }
8533:     case 1: {
8534:       // aten::histogram.bin_ct(Tensor self, int bins=100, *, float[]? range=None, Tensor? weight=None, bool density=False) -> (Tensor hist, Tensor bin_edges)
8535: 
8536:       auto dispatch_histogram = [](const at::Tensor & self, int64_t bins, ::std::optional<at::ArrayRef<double>> range, const ::std::optional<at::Tensor> & weight, bool density) -> ::std::tuple<at::Tensor,at::Tensor> {
8537:         pybind11::gil_scoped_release no_gil;
8538:         return self.histogram(bins, range, weight, density);
8539:       };
8540:       return wrap(NamedTuple, dispatch_histogram(self, _r.toInt64(0), _r.doublelistOptional(1), _r.optionalTensor(2), _r.toBool(3)));
8541:     }
8542:   }
8543:   Py_RETURN_NONE;
8544:   END_HANDLE_TH_ERRORS
8545: }
8546: 
8547: \
8548: // hsplit
8549: static PyObject * THPVariable_hsplit(PyObject* self_, PyObject* args, PyObject* kwargs)
8550: {
8551:   HANDLE_TH_ERRORS
8552:   const Tensor& self = THPVariable_Unpack(self_);
8553:   static PythonArgParser parser({
8554:     "hsplit(int64_t sections)",
8555:     "hsplit(IntArrayRef indices)",
8556:   }, /*traceable=*/true);
8557: 
8558:   ParsedArgs<1> parsed_args;
8559:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8560:   if(_r.has_torch_function()) {
8561:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8562:   }
8563:   switch (_r.idx) {
8564:     case 0: {
8565:       // aten::hsplit.int(Tensor(a -> *) self, int sections) -> Tensor(a)[]
8566: 
8567:       auto dispatch_hsplit = [](const at::Tensor & self, int64_t sections) -> ::std::vector<at::Tensor> {
8568:         pybind11::gil_scoped_release no_gil;
8569:         return self.hsplit(sections);
8570:       };
8571:       return wrap(dispatch_hsplit(self, _r.toInt64(0)));
8572:     }
8573:     case 1: {
8574:       // aten::hsplit.array(Tensor(a -> *) self, int[] indices) -> Tensor(a)[]
8575: 
8576:       auto dispatch_hsplit = [](const at::Tensor & self, at::IntArrayRef indices) -> ::std::vector<at::Tensor> {
8577:         pybind11::gil_scoped_release no_gil;
8578:         return self.hsplit(indices);
8579:       };
8580:       return wrap(dispatch_hsplit(self, _r.intlist(0)));
8581:     }
8582:   }
8583:   Py_RETURN_NONE;
8584:   END_HANDLE_TH_ERRORS
8585: }
8586: 
8587: // hypot
8588: static PyObject * THPVariable_hypot(PyObject* self_, PyObject* args, PyObject* kwargs)
8589: {
8590:   HANDLE_TH_ERRORS
8591:   const Tensor& self = THPVariable_Unpack(self_);
8592:   static PythonArgParser parser({
8593:     "hypot(Tensor other)",
8594:   }, /*traceable=*/true);
8595: 
8596:   ParsedArgs<1> parsed_args;
8597:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8598:   if(_r.has_torch_function()) {
8599:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8600:   }
8601:   // aten::hypot(Tensor self, Tensor other) -> Tensor
8602: 
8603:   auto dispatch_hypot = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8604:     pybind11::gil_scoped_release no_gil;
8605:     return self.hypot(other);
8606:   };
8607:   return wrap(dispatch_hypot(self, _r.tensor(0)));
8608:   Py_RETURN_NONE;
8609:   END_HANDLE_TH_ERRORS
8610: }
8611: 
8612: // hypot_
8613: static PyObject * THPVariable_hypot_(PyObject* self_, PyObject* args, PyObject* kwargs)
8614: {
8615:   HANDLE_TH_ERRORS
8616:   const Tensor& self = THPVariable_Unpack(self_);
8617:   static PythonArgParser parser({
8618:     "hypot_(Tensor other)",
8619:   }, /*traceable=*/true);
8620: 
8621:   ParsedArgs<1> parsed_args;
8622:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8623:   if(_r.has_torch_function()) {
8624:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8625:   }
8626:   // aten::hypot_(Tensor(a!) self, Tensor other) -> Tensor(a!)
8627: 
8628:   auto dispatch_hypot_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8629:     pybind11::gil_scoped_release no_gil;
8630:     return self.hypot_(other);
8631:   };
8632:   return wrap(dispatch_hypot_(self, _r.tensor(0)));
8633:   Py_RETURN_NONE;
8634:   END_HANDLE_TH_ERRORS
8635: }
8636: 
8637: // i0
8638: static PyObject * THPVariable_i0(PyObject* self_, PyObject* args)
8639: {
8640:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable_hsplit`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable_hsplit` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 8641-8760

```cpp
8641:   const Tensor& self = THPVariable_Unpack(self_);
8642:   if (has_torch_function(self_)) {
8643:     return handle_torch_function(self_, "i0");
8644:   }
8645:   // aten::i0(Tensor self) -> Tensor
8646: 
8647:   auto dispatch_i0 = [](const at::Tensor & self) -> at::Tensor {
8648:     pybind11::gil_scoped_release no_gil;
8649:     return self.i0();
8650:   };
8651:   return wrap(dispatch_i0(self));
8652:   END_HANDLE_TH_ERRORS
8653: }
8654: 
8655: // i0_
8656: static PyObject * THPVariable_i0_(PyObject* self_, PyObject* args)
8657: {
8658:   HANDLE_TH_ERRORS
8659:   const Tensor& self = THPVariable_Unpack(self_);
8660:   if (has_torch_function(self_)) {
8661:     return handle_torch_function(self_, "i0_");
8662:   }
8663:   // aten::i0_(Tensor(a!) self) -> Tensor(a!)
8664: 
8665:   auto dispatch_i0_ = [](const at::Tensor & self) -> at::Tensor {
8666:     pybind11::gil_scoped_release no_gil;
8667:     return self.i0_();
8668:   };
8669:   return wrap(dispatch_i0_(self));
8670:   END_HANDLE_TH_ERRORS
8671: }
8672: 
8673: // igamma
8674: static PyObject * THPVariable_igamma(PyObject* self_, PyObject* args, PyObject* kwargs)
8675: {
8676:   HANDLE_TH_ERRORS
8677:   const Tensor& self = THPVariable_Unpack(self_);
8678:   static PythonArgParser parser({
8679:     "igamma(Tensor other)",
8680:   }, /*traceable=*/true);
8681: 
8682:   ParsedArgs<1> parsed_args;
8683:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8684:   if(_r.has_torch_function()) {
8685:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8686:   }
8687:   // aten::igamma(Tensor self, Tensor other) -> Tensor
8688: 
8689:   auto dispatch_igamma = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8690:     pybind11::gil_scoped_release no_gil;
8691:     return self.igamma(other);
8692:   };
8693:   return wrap(dispatch_igamma(self, _r.tensor(0)));
8694:   Py_RETURN_NONE;
8695:   END_HANDLE_TH_ERRORS
8696: }
8697: 
8698: // igamma_
8699: static PyObject * THPVariable_igamma_(PyObject* self_, PyObject* args, PyObject* kwargs)
8700: {
8701:   HANDLE_TH_ERRORS
8702:   const Tensor& self = THPVariable_Unpack(self_);
8703:   static PythonArgParser parser({
8704:     "igamma_(Tensor other)",
8705:   }, /*traceable=*/true);
8706: 
8707:   ParsedArgs<1> parsed_args;
8708:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8709:   if(_r.has_torch_function()) {
8710:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8711:   }
8712:   // aten::igamma_(Tensor(a!) self, Tensor other) -> Tensor(a!)
8713: 
8714:   auto dispatch_igamma_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8715:     pybind11::gil_scoped_release no_gil;
8716:     return self.igamma_(other);
8717:   };
8718:   return wrap(dispatch_igamma_(self, _r.tensor(0)));
8719:   Py_RETURN_NONE;
8720:   END_HANDLE_TH_ERRORS
8721: }
8722: 
8723: // igammac
8724: static PyObject * THPVariable_igammac(PyObject* self_, PyObject* args, PyObject* kwargs)
8725: {
8726:   HANDLE_TH_ERRORS
8727:   const Tensor& self = THPVariable_Unpack(self_);
8728:   static PythonArgParser parser({
8729:     "igammac(Tensor other)",
8730:   }, /*traceable=*/true);
8731: 
8732:   ParsedArgs<1> parsed_args;
8733:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8734:   if(_r.has_torch_function()) {
8735:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8736:   }
8737:   // aten::igammac(Tensor self, Tensor other) -> Tensor
8738: 
8739:   auto dispatch_igammac = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8740:     pybind11::gil_scoped_release no_gil;
8741:     return self.igammac(other);
8742:   };
8743:   return wrap(dispatch_igammac(self, _r.tensor(0)));
8744:   Py_RETURN_NONE;
8745:   END_HANDLE_TH_ERRORS
8746: }
8747: 
8748: // igammac_
8749: static PyObject * THPVariable_igammac_(PyObject* self_, PyObject* args, PyObject* kwargs)
8750: {
8751:   HANDLE_TH_ERRORS
8752:   const Tensor& self = THPVariable_Unpack(self_);
8753:   static PythonArgParser parser({
8754:     "igammac_(Tensor other)",
8755:   }, /*traceable=*/true);
8756: 
8757:   ParsedArgs<1> parsed_args;
8758:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8759:   if(_r.has_torch_function()) {
8760:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `handle_torch_function`, `i0`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `handle_torch_function`, `i0` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8761-8880

```cpp
8761:   }
8762:   // aten::igammac_(Tensor(a!) self, Tensor other) -> Tensor(a!)
8763: 
8764:   auto dispatch_igammac_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
8765:     pybind11::gil_scoped_release no_gil;
8766:     return self.igammac_(other);
8767:   };
8768:   return wrap(dispatch_igammac_(self, _r.tensor(0)));
8769:   Py_RETURN_NONE;
8770:   END_HANDLE_TH_ERRORS
8771: }
8772: 
8773: \
8774: // index_add
8775: static PyObject * THPVariable_index_add(PyObject* self_, PyObject* args, PyObject* kwargs)
8776: {
8777:   HANDLE_TH_ERRORS
8778:   const Tensor& self = THPVariable_Unpack(self_);
8779:   static PythonArgParser parser({
8780:     "index_add(int64_t dim, Tensor index, Tensor source, *, Scalar alpha=1)",
8781:     "index_add(Dimname dim, Tensor index, Tensor source, *, Scalar alpha=1)",
8782:   }, /*traceable=*/true);
8783: 
8784:   ParsedArgs<4> parsed_args;
8785:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8786:   if(_r.has_torch_function()) {
8787:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8788:   }
8789:   switch (_r.idx) {
8790:     case 0: {
8791:       // aten::index_add(Tensor self, int dim, Tensor index, Tensor source, *, Scalar alpha=1) -> Tensor
8792: 
8793:       auto dispatch_index_add = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, const at::Scalar & alpha) -> at::Tensor {
8794:         pybind11::gil_scoped_release no_gil;
8795:         return self.index_add(dim, index, source, alpha);
8796:       };
8797:       return wrap(dispatch_index_add(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.scalar(3)));
8798:     }
8799:     case 1: {
8800:       // aten::index_add.dimname(Tensor self, Dimname dim, Tensor index, Tensor source, *, Scalar alpha=1) -> Tensor
8801: 
8802:       auto dispatch_index_add = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & source, const at::Scalar & alpha) -> at::Tensor {
8803:         pybind11::gil_scoped_release no_gil;
8804:         return self.index_add(dim, index, source, alpha);
8805:       };
8806:       return wrap(dispatch_index_add(self, _r.dimname(0), _r.tensor(1), _r.tensor(2), _r.scalar(3)));
8807:     }
8808:   }
8809:   Py_RETURN_NONE;
8810:   END_HANDLE_TH_ERRORS
8811: }
8812: 
8813: // index_add_
8814: static PyObject * THPVariable_index_add_(PyObject* self_, PyObject* args, PyObject* kwargs)
8815: {
8816:   HANDLE_TH_ERRORS
8817:   const Tensor& self = THPVariable_Unpack(self_);
8818:   static PythonArgParser parser({
8819:     "index_add_(int64_t dim, Tensor index, Tensor source, *, Scalar alpha=1)",
8820:   }, /*traceable=*/true);
8821: 
8822:   ParsedArgs<4> parsed_args;
8823:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8824:   if(_r.has_torch_function()) {
8825:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8826:   }
8827:   // aten::index_add_(Tensor(a!) self, int dim, Tensor index, Tensor source, *, Scalar alpha=1) -> Tensor(a!)
8828: 
8829:   auto dispatch_index_add_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, const at::Scalar & alpha) -> at::Tensor {
8830:     pybind11::gil_scoped_release no_gil;
8831:     return self.index_add_(dim, index, source, alpha);
8832:   };
8833:   return wrap(dispatch_index_add_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.scalar(3)));
8834:   Py_RETURN_NONE;
8835:   END_HANDLE_TH_ERRORS
8836: }
8837: 
8838: \
8839: // index_copy
8840: static PyObject * THPVariable_index_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
8841: {
8842:   HANDLE_TH_ERRORS
8843:   const Tensor& self = THPVariable_Unpack(self_);
8844:   static PythonArgParser parser({
8845:     "index_copy(int64_t dim, Tensor index, Tensor source)",
8846:     "index_copy(Dimname dim, Tensor index, Tensor source)",
8847:   }, /*traceable=*/true);
8848: 
8849:   ParsedArgs<3> parsed_args;
8850:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8851:   if(_r.has_torch_function()) {
8852:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8853:   }
8854:   switch (_r.idx) {
8855:     case 0: {
8856:       // aten::index_copy(Tensor self, int dim, Tensor index, Tensor source) -> Tensor
8857: 
8858:       auto dispatch_index_copy = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source) -> at::Tensor {
8859:         pybind11::gil_scoped_release no_gil;
8860:         return self.index_copy(dim, index, source);
8861:       };
8862:       return wrap(dispatch_index_copy(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
8863:     }
8864:     case 1: {
8865:       // aten::index_copy.dimname(Tensor self, Dimname dim, Tensor index, Tensor source) -> Tensor
8866: 
8867:       auto dispatch_index_copy = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & source) -> at::Tensor {
8868:         pybind11::gil_scoped_release no_gil;
8869:         return self.index_copy(dim, index, source);
8870:       };
8871:       return wrap(dispatch_index_copy(self, _r.dimname(0), _r.tensor(1), _r.tensor(2)));
8872:     }
8873:   }
8874:   Py_RETURN_NONE;
8875:   END_HANDLE_TH_ERRORS
8876: }
8877: 
8878: \
8879: // index_copy_
8880: static PyObject * THPVariable_index_copy_(PyObject* self_, PyObject* args, PyObject* kwargs)
```

- EN: The main execution path in this span is carried by `igammac_`, `wrap`, `THPVariable_index_add`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `igammac_`, `wrap`, `THPVariable_index_add` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 8881-9000

```cpp
8881: {
8882:   HANDLE_TH_ERRORS
8883:   const Tensor& self = THPVariable_Unpack(self_);
8884:   static PythonArgParser parser({
8885:     "index_copy_(int64_t dim, Tensor index, Tensor source)",
8886:     "index_copy_(Dimname dim, Tensor index, Tensor source)",
8887:   }, /*traceable=*/true);
8888: 
8889:   ParsedArgs<3> parsed_args;
8890:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8891:   if(_r.has_torch_function()) {
8892:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8893:   }
8894:   switch (_r.idx) {
8895:     case 0: {
8896:       // aten::index_copy_(Tensor(a!) self, int dim, Tensor index, Tensor source) -> Tensor(a!)
8897: 
8898:       auto dispatch_index_copy_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source) -> at::Tensor {
8899:         pybind11::gil_scoped_release no_gil;
8900:         return self.index_copy_(dim, index, source);
8901:       };
8902:       return wrap(dispatch_index_copy_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
8903:     }
8904:     case 1: {
8905:       // aten::index_copy_.dimname(Tensor(a!) self, Dimname dim, Tensor index, Tensor source) -> Tensor(a!)
8906: 
8907:       auto dispatch_index_copy_ = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & source) -> at::Tensor {
8908:         pybind11::gil_scoped_release no_gil;
8909:         return self.index_copy_(dim, index, source);
8910:       };
8911:       return wrap(dispatch_index_copy_(self, _r.dimname(0), _r.tensor(1), _r.tensor(2)));
8912:     }
8913:   }
8914:   Py_RETURN_NONE;
8915:   END_HANDLE_TH_ERRORS
8916: }
8917: 
8918: \
8919: // index_fill
8920: static PyObject * THPVariable_index_fill(PyObject* self_, PyObject* args, PyObject* kwargs)
8921: {
8922:   HANDLE_TH_ERRORS
8923:   const Tensor& self = THPVariable_Unpack(self_);
8924:   static PythonArgParser parser({
8925:     "index_fill(int64_t dim, Tensor index, Tensor value)",
8926:     "index_fill(Dimname dim, Tensor index, Tensor value)",
8927:     "index_fill(int64_t dim, Tensor index, Scalar value)",
8928:     "index_fill(Dimname dim, Tensor index, Scalar value)",
8929:   }, /*traceable=*/true);
8930: 
8931:   ParsedArgs<3> parsed_args;
8932:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8933:   if(_r.has_torch_function()) {
8934:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8935:   }
8936:   switch (_r.idx) {
8937:     case 0: {
8938:       // aten::index_fill.int_Tensor(Tensor self, int dim, Tensor index, Tensor value) -> Tensor
8939: 
8940:       auto dispatch_index_fill = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & value) -> at::Tensor {
8941:         pybind11::gil_scoped_release no_gil;
8942:         return self.index_fill(dim, index, value);
8943:       };
8944:       return wrap(dispatch_index_fill(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
8945:     }
8946:     case 1: {
8947:       // aten::index_fill.Dimname_Tensor(Tensor self, Dimname dim, Tensor index, Tensor value) -> Tensor
8948: 
8949:       auto dispatch_index_fill = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & value) -> at::Tensor {
8950:         pybind11::gil_scoped_release no_gil;
8951:         return self.index_fill(dim, index, value);
8952:       };
8953:       return wrap(dispatch_index_fill(self, _r.dimname(0), _r.tensor(1), _r.tensor(2)));
8954:     }
8955:     case 2: {
8956:       // aten::index_fill.int_Scalar(Tensor self, int dim, Tensor index, Scalar value) -> Tensor
8957: 
8958:       auto dispatch_index_fill = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
8959:         pybind11::gil_scoped_release no_gil;
8960:         return self.index_fill(dim, index, value);
8961:       };
8962:       return wrap(dispatch_index_fill(self, _r.toInt64(0), _r.tensor(1), _r.scalar(2)));
8963:     }
8964:     case 3: {
8965:       // aten::index_fill.Dimname_Scalar(Tensor self, Dimname dim, Tensor index, Scalar value) -> Tensor
8966: 
8967:       auto dispatch_index_fill = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
8968:         pybind11::gil_scoped_release no_gil;
8969:         return self.index_fill(dim, index, value);
8970:       };
8971:       return wrap(dispatch_index_fill(self, _r.dimname(0), _r.tensor(1), _r.scalar(2)));
8972:     }
8973:   }
8974:   Py_RETURN_NONE;
8975:   END_HANDLE_TH_ERRORS
8976: }
8977: 
8978: \
8979: // index_fill_
8980: static PyObject * THPVariable_index_fill_(PyObject* self_, PyObject* args, PyObject* kwargs)
8981: {
8982:   HANDLE_TH_ERRORS
8983:   const Tensor& self = THPVariable_Unpack(self_);
8984:   static PythonArgParser parser({
8985:     "index_fill_(int64_t dim, Tensor index, Tensor value)",
8986:     "index_fill_(Dimname dim, Tensor index, Tensor value)",
8987:     "index_fill_(int64_t dim, Tensor index, Scalar value)",
8988:     "index_fill_(Dimname dim, Tensor index, Scalar value)",
8989:   }, /*traceable=*/true);
8990: 
8991:   ParsedArgs<3> parsed_args;
8992:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
8993:   if(_r.has_torch_function()) {
8994:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
8995:   }
8996:   switch (_r.idx) {
8997:     case 0: {
8998:       // aten::index_fill_.int_Tensor(Tensor(a!) self, int dim, Tensor index, Tensor value) -> Tensor(a!)
8999: 
9000:       auto dispatch_index_fill_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & value) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9001-9120

```cpp
9001:         pybind11::gil_scoped_release no_gil;
9002:         return self.index_fill_(dim, index, value);
9003:       };
9004:       return wrap(dispatch_index_fill_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
9005:     }
9006:     case 1: {
9007:       // aten::index_fill_.Dimname_Tensor(Tensor(a!) self, Dimname dim, Tensor index, Tensor value) -> Tensor(a!)
9008: 
9009:       auto dispatch_index_fill_ = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & value) -> at::Tensor {
9010:         pybind11::gil_scoped_release no_gil;
9011:         return self.index_fill_(dim, index, value);
9012:       };
9013:       return wrap(dispatch_index_fill_(self, _r.dimname(0), _r.tensor(1), _r.tensor(2)));
9014:     }
9015:     case 2: {
9016:       // aten::index_fill_.int_Scalar(Tensor(a!) self, int dim, Tensor index, Scalar value) -> Tensor(a!)
9017: 
9018:       auto dispatch_index_fill_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
9019:         pybind11::gil_scoped_release no_gil;
9020:         return self.index_fill_(dim, index, value);
9021:       };
9022:       return wrap(dispatch_index_fill_(self, _r.toInt64(0), _r.tensor(1), _r.scalar(2)));
9023:     }
9024:     case 3: {
9025:       // aten::index_fill_.Dimname_Scalar(Tensor(a!) self, Dimname dim, Tensor index, Scalar value) -> Tensor(a!)
9026: 
9027:       auto dispatch_index_fill_ = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
9028:         pybind11::gil_scoped_release no_gil;
9029:         return self.index_fill_(dim, index, value);
9030:       };
9031:       return wrap(dispatch_index_fill_(self, _r.dimname(0), _r.tensor(1), _r.scalar(2)));
9032:     }
9033:   }
9034:   Py_RETURN_NONE;
9035:   END_HANDLE_TH_ERRORS
9036: }
9037: 
9038: // index_put
9039: static PyObject * THPVariable_index_put(PyObject* self_, PyObject* args, PyObject* kwargs)
9040: {
9041:   HANDLE_TH_ERRORS
9042:   const Tensor& self = THPVariable_Unpack(self_);
9043:   static PythonArgParser parser({
9044:     "index_put(c10::List<::std::optional<Tensor>> indices, Tensor values, bool accumulate=False)",
9045:   }, /*traceable=*/true);
9046: 
9047:   ParsedArgs<3> parsed_args;
9048:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9049:   if(_r.has_torch_function()) {
9050:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9051:   }
9052:   // aten::index_put(Tensor self, Tensor?[] indices, Tensor values, bool accumulate=False) -> Tensor
9053: 
9054:   auto dispatch_index_put = [](const at::Tensor & self, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values, bool accumulate) -> at::Tensor {
9055:     pybind11::gil_scoped_release no_gil;
9056:     return self.index_put(indices, values, accumulate);
9057:   };
9058:   return wrap(dispatch_index_put(self, _r.list_of_optional_tensors(0), _r.tensor(1), _r.toBool(2)));
9059:   Py_RETURN_NONE;
9060:   END_HANDLE_TH_ERRORS
9061: }
9062: 
9063: // index_put_
9064: static PyObject * THPVariable_index_put_(PyObject* self_, PyObject* args, PyObject* kwargs)
9065: {
9066:   HANDLE_TH_ERRORS
9067:   const Tensor& self = THPVariable_Unpack(self_);
9068:   static PythonArgParser parser({
9069:     "index_put_(c10::List<::std::optional<Tensor>> indices, Tensor values, bool accumulate=False)",
9070:   }, /*traceable=*/true);
9071: 
9072:   ParsedArgs<3> parsed_args;
9073:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9074:   if(_r.has_torch_function()) {
9075:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9076:   }
9077:   // aten::index_put_(Tensor(a!) self, Tensor?[] indices, Tensor values, bool accumulate=False) -> Tensor(a!)
9078: 
9079:   auto dispatch_index_put_ = [](const at::Tensor & self, const c10::List<::std::optional<at::Tensor>> & indices, const at::Tensor & values, bool accumulate) -> at::Tensor {
9080:     pybind11::gil_scoped_release no_gil;
9081:     return self.index_put_(indices, values, accumulate);
9082:   };
9083:   return wrap(dispatch_index_put_(self, _r.list_of_optional_tensors(0), _r.tensor(1), _r.toBool(2)));
9084:   Py_RETURN_NONE;
9085:   END_HANDLE_TH_ERRORS
9086: }
9087: 
9088: // index_reduce
9089: static PyObject * THPVariable_index_reduce(PyObject* self_, PyObject* args, PyObject* kwargs)
9090: {
9091:   HANDLE_TH_ERRORS
9092:   const Tensor& self = THPVariable_Unpack(self_);
9093:   static PythonArgParser parser({
9094:     "index_reduce(int64_t dim, Tensor index, Tensor source, c10::string_view reduce, *, bool include_self=True)",
9095:   }, /*traceable=*/true);
9096: 
9097:   ParsedArgs<5> parsed_args;
9098:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9099:   if(_r.has_torch_function()) {
9100:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9101:   }
9102:   // aten::index_reduce(Tensor self, int dim, Tensor index, Tensor source, str reduce, *, bool include_self=True) -> Tensor
9103: 
9104:   auto dispatch_index_reduce = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, c10::string_view reduce, bool include_self) -> at::Tensor {
9105:     pybind11::gil_scoped_release no_gil;
9106:     return self.index_reduce(dim, index, source, reduce, include_self);
9107:   };
9108:   return wrap(dispatch_index_reduce(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.stringView(3), _r.toBool(4)));
9109:   Py_RETURN_NONE;
9110:   END_HANDLE_TH_ERRORS
9111: }
9112: 
9113: // index_reduce_
9114: static PyObject * THPVariable_index_reduce_(PyObject* self_, PyObject* args, PyObject* kwargs)
9115: {
9116:   HANDLE_TH_ERRORS
9117:   const Tensor& self = THPVariable_Unpack(self_);
9118:   static PythonArgParser parser({
9119:     "index_reduce_(int64_t dim, Tensor index, Tensor source, c10::string_view reduce, *, bool include_self=True)",
9120:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_index_put`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_index_put`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9121-9240

```cpp
9121: 
9122:   ParsedArgs<5> parsed_args;
9123:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9124:   if(_r.has_torch_function()) {
9125:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9126:   }
9127:   // aten::index_reduce_(Tensor(a!) self, int dim, Tensor index, Tensor source, str reduce, *, bool include_self=True) -> Tensor(a!)
9128: 
9129:   auto dispatch_index_reduce_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & source, c10::string_view reduce, bool include_self) -> at::Tensor {
9130:     pybind11::gil_scoped_release no_gil;
9131:     return self.index_reduce_(dim, index, source, reduce, include_self);
9132:   };
9133:   return wrap(dispatch_index_reduce_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.stringView(3), _r.toBool(4)));
9134:   Py_RETURN_NONE;
9135:   END_HANDLE_TH_ERRORS
9136: }
9137: 
9138: \
9139: // index_select
9140: static PyObject * THPVariable_index_select(PyObject* self_, PyObject* args, PyObject* kwargs)
9141: {
9142:   HANDLE_TH_ERRORS
9143:   const Tensor& self = THPVariable_Unpack(self_);
9144:   static PythonArgParser parser({
9145:     "index_select(int64_t dim, Tensor index)",
9146:     "index_select(Dimname dim, Tensor index)",
9147:   }, /*traceable=*/true);
9148: 
9149:   ParsedArgs<2> parsed_args;
9150:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9151:   if(_r.has_torch_function()) {
9152:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9153:   }
9154:   switch (_r.idx) {
9155:     case 0: {
9156:       // aten::index_select(Tensor self, int dim, Tensor index) -> Tensor
9157: 
9158:       auto dispatch_index_select = [](const at::Tensor & self, int64_t dim, const at::Tensor & index) -> at::Tensor {
9159:         pybind11::gil_scoped_release no_gil;
9160:         return self.index_select(dim, index);
9161:       };
9162:       return wrap(dispatch_index_select(self, _r.toInt64(0), _r.tensor(1)));
9163:     }
9164:     case 1: {
9165:       // aten::index_select.dimname(Tensor self, Dimname dim, Tensor index) -> Tensor
9166: 
9167:       auto dispatch_index_select = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index) -> at::Tensor {
9168:         pybind11::gil_scoped_release no_gil;
9169:         return self.index_select(dim, index);
9170:       };
9171:       return wrap(dispatch_index_select(self, _r.dimname(0), _r.tensor(1)));
9172:     }
9173:   }
9174:   Py_RETURN_NONE;
9175:   END_HANDLE_TH_ERRORS
9176: }
9177: 
9178: // indices
9179: static PyObject * THPVariable_indices(PyObject* self_, PyObject* args)
9180: {
9181:   HANDLE_TH_ERRORS
9182:   const Tensor& self = THPVariable_Unpack(self_);
9183:   if (has_torch_function(self_)) {
9184:     return handle_torch_function(self_, "indices");
9185:   }
9186:   // aten::indices(Tensor(a) self) -> Tensor(a)
9187: 
9188:   auto dispatch_indices = [](const at::Tensor & self) -> at::Tensor {
9189:     pybind11::gil_scoped_release no_gil;
9190:     return self.indices();
9191:   };
9192:   return wrap(dispatch_indices(self));
9193:   END_HANDLE_TH_ERRORS
9194: }
9195: 
9196: // inner
9197: static PyObject * THPVariable_inner(PyObject* self_, PyObject* args, PyObject* kwargs)
9198: {
9199:   HANDLE_TH_ERRORS
9200:   const Tensor& self = THPVariable_Unpack(self_);
9201:   static PythonArgParser parser({
9202:     "inner(Tensor other)",
9203:   }, /*traceable=*/true);
9204: 
9205:   ParsedArgs<1> parsed_args;
9206:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9207:   if(_r.has_torch_function()) {
9208:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9209:   }
9210:   // aten::inner(Tensor self, Tensor other) -> Tensor
9211: 
9212:   auto dispatch_inner = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9213:     pybind11::gil_scoped_release no_gil;
9214:     return self.inner(other);
9215:   };
9216:   return wrap(dispatch_inner(self, _r.tensor(0)));
9217:   Py_RETURN_NONE;
9218:   END_HANDLE_TH_ERRORS
9219: }
9220: 
9221: // int_repr
9222: static PyObject * THPVariable_int_repr(PyObject* self_, PyObject* args)
9223: {
9224:   HANDLE_TH_ERRORS
9225:   const Tensor& self = THPVariable_Unpack(self_);
9226:   if (has_torch_function(self_)) {
9227:     return handle_torch_function(self_, "int_repr");
9228:   }
9229:   // aten::int_repr(Tensor self) -> Tensor
9230: 
9231:   auto dispatch_int_repr = [](const at::Tensor & self) -> at::Tensor {
9232:     pybind11::gil_scoped_release no_gil;
9233:     return self.int_repr();
9234:   };
9235:   return wrap(dispatch_int_repr(self));
9236:   END_HANDLE_TH_ERRORS
9237: }
9238: 
9239: // inverse
9240: static PyObject * THPVariable_inverse(PyObject* self_, PyObject* args)
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `index_reduce_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `index_reduce_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9241-9360

```cpp
9241: {
9242:   HANDLE_TH_ERRORS
9243:   const Tensor& self = THPVariable_Unpack(self_);
9244:   if (has_torch_function(self_)) {
9245:     return handle_torch_function(self_, "inverse");
9246:   }
9247:   // aten::inverse(Tensor self) -> Tensor
9248: 
9249:   auto dispatch_inverse = [](const at::Tensor & self) -> at::Tensor {
9250:     pybind11::gil_scoped_release no_gil;
9251:     return self.inverse();
9252:   };
9253:   return wrap(dispatch_inverse(self));
9254:   END_HANDLE_TH_ERRORS
9255: }
9256: 
9257: // is_coalesced
9258: static PyObject * THPVariable_is_coalesced(PyObject* self_, PyObject* args)
9259: {
9260:   HANDLE_TH_ERRORS
9261:   const Tensor& self = THPVariable_Unpack(self_);
9262:   if (has_torch_function(self_)) {
9263:     return handle_torch_function(self_, "is_coalesced");
9264:   }
9265:   // aten::is_coalesced(Tensor self) -> bool
9266: 
9267:   auto dispatch_is_coalesced = [](const at::Tensor & self) -> bool {
9268:     pybind11::gil_scoped_release no_gil;
9269:     return self.is_coalesced();
9270:   };
9271:   return wrap(dispatch_is_coalesced(self));
9272:   END_HANDLE_TH_ERRORS
9273: }
9274: 
9275: // is_complex
9276: static PyObject * THPVariable_is_complex(PyObject* self_, PyObject* args)
9277: {
9278:   HANDLE_TH_ERRORS
9279:   const Tensor& self = THPVariable_Unpack(self_);
9280:   if (has_torch_function(self_)) {
9281:     return handle_torch_function(self_, "is_complex");
9282:   }
9283:   // aten::is_complex(Tensor self) -> bool
9284: 
9285:   auto dispatch_is_complex = [](const at::Tensor & self) -> bool {
9286:     pybind11::gil_scoped_release no_gil;
9287:     return self.is_complex();
9288:   };
9289:   return wrap(dispatch_is_complex(self));
9290:   END_HANDLE_TH_ERRORS
9291: }
9292: 
9293: // is_conj
9294: static PyObject * THPVariable_is_conj(PyObject* self_, PyObject* args)
9295: {
9296:   HANDLE_TH_ERRORS
9297:   const Tensor& self = THPVariable_Unpack(self_);
9298:   if (has_torch_function(self_)) {
9299:     return handle_torch_function(self_, "is_conj");
9300:   }
9301:   // aten::is_conj(Tensor self) -> bool
9302: 
9303:   auto dispatch_is_conj = [](const at::Tensor & self) -> bool {
9304:     pybind11::gil_scoped_release no_gil;
9305:     return self.is_conj();
9306:   };
9307:   return wrap(dispatch_is_conj(self));
9308:   END_HANDLE_TH_ERRORS
9309: }
9310: 
9311: // is_distributed
9312: static PyObject * THPVariable_is_distributed(PyObject* self_, PyObject* args)
9313: {
9314:   HANDLE_TH_ERRORS
9315:   const Tensor& self = THPVariable_Unpack(self_);
9316:   if (has_torch_function(self_)) {
9317:     return handle_torch_function(self_, "is_distributed");
9318:   }
9319:   // aten::is_distributed(Tensor self) -> bool
9320: 
9321:   auto dispatch_is_distributed = [](const at::Tensor & self) -> bool {
9322:     pybind11::gil_scoped_release no_gil;
9323:     return self.is_distributed();
9324:   };
9325:   return wrap(dispatch_is_distributed(self));
9326:   END_HANDLE_TH_ERRORS
9327: }
9328: 
9329: // is_floating_point
9330: static PyObject * THPVariable_is_floating_point(PyObject* self_, PyObject* args)
9331: {
9332:   HANDLE_TH_ERRORS
9333:   const Tensor& self = THPVariable_Unpack(self_);
9334:   if (has_torch_function(self_)) {
9335:     return handle_torch_function(self_, "is_floating_point");
9336:   }
9337:   // aten::is_floating_point(Tensor self) -> bool
9338: 
9339:   auto dispatch_is_floating_point = [](const at::Tensor & self) -> bool {
9340:     pybind11::gil_scoped_release no_gil;
9341:     return self.is_floating_point();
9342:   };
9343:   return wrap(dispatch_is_floating_point(self));
9344:   END_HANDLE_TH_ERRORS
9345: }
9346: 
9347: // is_inference
9348: static PyObject * THPVariable_is_inference(PyObject* self_, PyObject* args)
9349: {
9350:   HANDLE_TH_ERRORS
9351:   const Tensor& self = THPVariable_Unpack(self_);
9352:   if (has_torch_function(self_)) {
9353:     return handle_torch_function(self_, "is_inference");
9354:   }
9355:   // aten::is_inference(Tensor self) -> bool
9356: 
9357:   auto dispatch_is_inference = [](const at::Tensor & self) -> bool {
9358:     pybind11::gil_scoped_release no_gil;
9359:     return self.is_inference();
9360:   };
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `handle_torch_function`, `inverse`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `handle_torch_function`, `inverse` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9361-9480

```cpp
9361:   return wrap(dispatch_is_inference(self));
9362:   END_HANDLE_TH_ERRORS
9363: }
9364: 
9365: // is_neg
9366: static PyObject * THPVariable_is_neg(PyObject* self_, PyObject* args)
9367: {
9368:   HANDLE_TH_ERRORS
9369:   const Tensor& self = THPVariable_Unpack(self_);
9370:   if (has_torch_function(self_)) {
9371:     return handle_torch_function(self_, "is_neg");
9372:   }
9373:   // aten::is_neg(Tensor self) -> bool
9374: 
9375:   auto dispatch_is_neg = [](const at::Tensor & self) -> bool {
9376:     pybind11::gil_scoped_release no_gil;
9377:     return self.is_neg();
9378:   };
9379:   return wrap(dispatch_is_neg(self));
9380:   END_HANDLE_TH_ERRORS
9381: }
9382: 
9383: // is_nonzero
9384: static PyObject * THPVariable_is_nonzero(PyObject* self_, PyObject* args)
9385: {
9386:   HANDLE_TH_ERRORS
9387:   const Tensor& self = THPVariable_Unpack(self_);
9388:   if (has_torch_function(self_)) {
9389:     return handle_torch_function(self_, "is_nonzero");
9390:   }
9391:   // aten::is_nonzero(Tensor self) -> bool
9392: 
9393:   auto dispatch_is_nonzero = [](const at::Tensor & self) -> bool {
9394:     pybind11::gil_scoped_release no_gil;
9395:     return self.is_nonzero();
9396:   };
9397:   return wrap(dispatch_is_nonzero(self));
9398:   END_HANDLE_TH_ERRORS
9399: }
9400: 
9401: // is_pinned
9402: static PyObject * THPVariable_is_pinned(PyObject* self_, PyObject* args, PyObject* kwargs)
9403: {
9404:   HANDLE_TH_ERRORS
9405:   const Tensor& self = THPVariable_Unpack(self_);
9406:   static PythonArgParser parser({
9407:     "is_pinned(Device? device=None)",
9408:   }, /*traceable=*/false);
9409: 
9410:   ParsedArgs<1> parsed_args;
9411:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9412:   if(_r.has_torch_function()) {
9413:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9414:   }
9415:   // aten::is_pinned(Tensor self, Device? device=None) -> bool
9416: 
9417:   auto dispatch_is_pinned = [](const at::Tensor & self, ::std::optional<at::Device> device) -> bool {
9418:     pybind11::gil_scoped_release no_gil;
9419:     return self.is_pinned(device);
9420:   };
9421:   return wrap(dispatch_is_pinned(self, _r.deviceOptional(0)));
9422:   Py_RETURN_NONE;
9423:   END_HANDLE_TH_ERRORS
9424: }
9425: 
9426: // is_same_size
9427: static PyObject * THPVariable_is_same_size(PyObject* self_, PyObject* args, PyObject* kwargs)
9428: {
9429:   HANDLE_TH_ERRORS
9430:   const Tensor& self = THPVariable_Unpack(self_);
9431:   static PythonArgParser parser({
9432:     "is_same_size(Tensor other)",
9433:   }, /*traceable=*/false);
9434: 
9435:   ParsedArgs<1> parsed_args;
9436:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9437:   if(_r.has_torch_function()) {
9438:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9439:   }
9440:   // aten::is_same_size(Tensor self, Tensor other) -> bool
9441: 
9442:   auto dispatch_is_same_size = [](const at::Tensor & self, const at::Tensor & other) -> bool {
9443:     pybind11::gil_scoped_release no_gil;
9444:     return self.is_same_size(other);
9445:   };
9446:   return wrap(dispatch_is_same_size(self, _r.tensor(0)));
9447:   Py_RETURN_NONE;
9448:   END_HANDLE_TH_ERRORS
9449: }
9450: 
9451: // is_set_to
9452: static PyObject * THPVariable_is_set_to(PyObject* self_, PyObject* args, PyObject* kwargs)
9453: {
9454:   HANDLE_TH_ERRORS
9455:   const Tensor& self = THPVariable_Unpack(self_);
9456:   static PythonArgParser parser({
9457:     "is_set_to(Tensor tensor)",
9458:   }, /*traceable=*/false);
9459: 
9460:   ParsedArgs<1> parsed_args;
9461:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9462:   if(_r.has_torch_function()) {
9463:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9464:   }
9465:   // aten::is_set_to(Tensor self, Tensor tensor) -> bool
9466: 
9467:   auto dispatch_is_set_to = [](const at::Tensor & self, const at::Tensor & tensor) -> bool {
9468:     pybind11::gil_scoped_release no_gil;
9469:     return self.is_set_to(tensor);
9470:   };
9471:   return wrap(dispatch_is_set_to(self, _r.tensor(0)));
9472:   Py_RETURN_NONE;
9473:   END_HANDLE_TH_ERRORS
9474: }
9475: 
9476: // is_signed
9477: static PyObject * THPVariable_is_signed(PyObject* self_, PyObject* args)
9478: {
9479:   HANDLE_TH_ERRORS
9480:   const Tensor& self = THPVariable_Unpack(self_);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_is_neg`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_is_neg`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9481-9600

```cpp
9481:   if (has_torch_function(self_)) {
9482:     return handle_torch_function(self_, "is_signed");
9483:   }
9484:   // aten::is_signed(Tensor self) -> bool
9485: 
9486:   auto dispatch_is_signed = [](const at::Tensor & self) -> bool {
9487:     pybind11::gil_scoped_release no_gil;
9488:     return self.is_signed();
9489:   };
9490:   return wrap(dispatch_is_signed(self));
9491:   END_HANDLE_TH_ERRORS
9492: }
9493: 
9494: // isclose
9495: static PyObject * THPVariable_isclose(PyObject* self_, PyObject* args, PyObject* kwargs)
9496: {
9497:   HANDLE_TH_ERRORS
9498:   const Tensor& self = THPVariable_Unpack(self_);
9499:   static PythonArgParser parser({
9500:     "isclose(Tensor other, double rtol=1e-05, double atol=1e-08, bool equal_nan=False)",
9501:   }, /*traceable=*/true);
9502: 
9503:   ParsedArgs<4> parsed_args;
9504:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9505:   if(_r.has_torch_function()) {
9506:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9507:   }
9508:   // aten::isclose(Tensor self, Tensor other, float rtol=1e-05, float atol=1e-08, bool equal_nan=False) -> Tensor
9509: 
9510:   auto dispatch_isclose = [](const at::Tensor & self, const at::Tensor & other, double rtol, double atol, bool equal_nan) -> at::Tensor {
9511:     pybind11::gil_scoped_release no_gil;
9512:     return self.isclose(other, rtol, atol, equal_nan);
9513:   };
9514:   return wrap(dispatch_isclose(self, _r.tensor(0), _r.toDouble(1), _r.toDouble(2), _r.toBool(3)));
9515:   Py_RETURN_NONE;
9516:   END_HANDLE_TH_ERRORS
9517: }
9518: 
9519: // isfinite
9520: static PyObject * THPVariable_isfinite(PyObject* self_, PyObject* args)
9521: {
9522:   HANDLE_TH_ERRORS
9523:   const Tensor& self = THPVariable_Unpack(self_);
9524:   if (has_torch_function(self_)) {
9525:     return handle_torch_function(self_, "isfinite");
9526:   }
9527:   // aten::isfinite(Tensor self) -> Tensor
9528: 
9529:   auto dispatch_isfinite = [](const at::Tensor & self) -> at::Tensor {
9530:     pybind11::gil_scoped_release no_gil;
9531:     return self.isfinite();
9532:   };
9533:   return wrap(dispatch_isfinite(self));
9534:   END_HANDLE_TH_ERRORS
9535: }
9536: 
9537: // isinf
9538: static PyObject * THPVariable_isinf(PyObject* self_, PyObject* args)
9539: {
9540:   HANDLE_TH_ERRORS
9541:   const Tensor& self = THPVariable_Unpack(self_);
9542:   if (has_torch_function(self_)) {
9543:     return handle_torch_function(self_, "isinf");
9544:   }
9545:   // aten::isinf(Tensor self) -> Tensor
9546: 
9547:   auto dispatch_isinf = [](const at::Tensor & self) -> at::Tensor {
9548:     pybind11::gil_scoped_release no_gil;
9549:     return self.isinf();
9550:   };
9551:   return wrap(dispatch_isinf(self));
9552:   END_HANDLE_TH_ERRORS
9553: }
9554: 
9555: // isnan
9556: static PyObject * THPVariable_isnan(PyObject* self_, PyObject* args)
9557: {
9558:   HANDLE_TH_ERRORS
9559:   const Tensor& self = THPVariable_Unpack(self_);
9560:   if (has_torch_function(self_)) {
9561:     return handle_torch_function(self_, "isnan");
9562:   }
9563:   // aten::isnan(Tensor self) -> Tensor
9564: 
9565:   auto dispatch_isnan = [](const at::Tensor & self) -> at::Tensor {
9566:     pybind11::gil_scoped_release no_gil;
9567:     return self.isnan();
9568:   };
9569:   return wrap(dispatch_isnan(self));
9570:   END_HANDLE_TH_ERRORS
9571: }
9572: 
9573: // isneginf
9574: static PyObject * THPVariable_isneginf(PyObject* self_, PyObject* args)
9575: {
9576:   HANDLE_TH_ERRORS
9577:   const Tensor& self = THPVariable_Unpack(self_);
9578:   if (has_torch_function(self_)) {
9579:     return handle_torch_function(self_, "isneginf");
9580:   }
9581:   // aten::isneginf(Tensor self) -> Tensor
9582: 
9583:   auto dispatch_isneginf = [](const at::Tensor & self) -> at::Tensor {
9584:     pybind11::gil_scoped_release no_gil;
9585:     return self.isneginf();
9586:   };
9587:   return wrap(dispatch_isneginf(self));
9588:   END_HANDLE_TH_ERRORS
9589: }
9590: 
9591: // isposinf
9592: static PyObject * THPVariable_isposinf(PyObject* self_, PyObject* args)
9593: {
9594:   HANDLE_TH_ERRORS
9595:   const Tensor& self = THPVariable_Unpack(self_);
9596:   if (has_torch_function(self_)) {
9597:     return handle_torch_function(self_, "isposinf");
9598:   }
9599:   // aten::isposinf(Tensor self) -> Tensor
9600: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `is_signed`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `is_signed`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9601-9720

```cpp
9601:   auto dispatch_isposinf = [](const at::Tensor & self) -> at::Tensor {
9602:     pybind11::gil_scoped_release no_gil;
9603:     return self.isposinf();
9604:   };
9605:   return wrap(dispatch_isposinf(self));
9606:   END_HANDLE_TH_ERRORS
9607: }
9608: 
9609: // isreal
9610: static PyObject * THPVariable_isreal(PyObject* self_, PyObject* args)
9611: {
9612:   HANDLE_TH_ERRORS
9613:   const Tensor& self = THPVariable_Unpack(self_);
9614:   if (has_torch_function(self_)) {
9615:     return handle_torch_function(self_, "isreal");
9616:   }
9617:   // aten::isreal(Tensor self) -> Tensor
9618: 
9619:   auto dispatch_isreal = [](const at::Tensor & self) -> at::Tensor {
9620:     pybind11::gil_scoped_release no_gil;
9621:     return self.isreal();
9622:   };
9623:   return wrap(dispatch_isreal(self));
9624:   END_HANDLE_TH_ERRORS
9625: }
9626: 
9627: // istft
9628: static PyObject * THPVariable_istft(PyObject* self_, PyObject* args, PyObject* kwargs)
9629: {
9630:   HANDLE_TH_ERRORS
9631:   const Tensor& self = THPVariable_Unpack(self_);
9632:   static PythonArgParser parser({
9633:     "istft(int64_t n_fft, int64_t? hop_length=None, int64_t? win_length=None, Tensor? window=None, bool center=True, bool normalized=False, bool? onesided=None, int64_t? length=None, bool return_complex=False)",
9634:   }, /*traceable=*/true);
9635: 
9636:   ParsedArgs<9> parsed_args;
9637:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9638:   if(_r.has_torch_function()) {
9639:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9640:   }
9641:   // aten::istft(Tensor self, int n_fft, int? hop_length=None, int? win_length=None, Tensor? window=None, bool center=True, bool normalized=False, bool? onesided=None, int? length=None, bool return_complex=False) -> Tensor
9642: 
9643:   auto dispatch_istft = [](const at::Tensor & self, int64_t n_fft, ::std::optional<int64_t> hop_length, ::std::optional<int64_t> win_length, const ::std::optional<at::Tensor> & window, bool center, bool normalized, ::std::optional<bool> onesided, ::std::optional<int64_t> length, bool return_complex) -> at::Tensor {
9644:     pybind11::gil_scoped_release no_gil;
9645:     return self.istft(n_fft, hop_length, win_length, window, center, normalized, onesided, length, return_complex);
9646:   };
9647:   return wrap(dispatch_istft(self, _r.toInt64(0), _r.toInt64Optional(1), _r.toInt64Optional(2), _r.optionalTensor(3), _r.toBool(4), _r.toBool(5), _r.toBoolOptional(6), _r.toInt64Optional(7), _r.toBool(8)));
9648:   Py_RETURN_NONE;
9649:   END_HANDLE_TH_ERRORS
9650: }
9651: 
9652: // kron
9653: static PyObject * THPVariable_kron(PyObject* self_, PyObject* args, PyObject* kwargs)
9654: {
9655:   HANDLE_TH_ERRORS
9656:   const Tensor& self = THPVariable_Unpack(self_);
9657:   static PythonArgParser parser({
9658:     "kron(Tensor other)",
9659:   }, /*traceable=*/true);
9660: 
9661:   ParsedArgs<1> parsed_args;
9662:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9663:   if(_r.has_torch_function()) {
9664:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9665:   }
9666:   // aten::kron(Tensor self, Tensor other) -> Tensor
9667: 
9668:   auto dispatch_kron = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9669:     pybind11::gil_scoped_release no_gil;
9670:     return self.kron(other);
9671:   };
9672:   return wrap(dispatch_kron(self, _r.tensor(0)));
9673:   Py_RETURN_NONE;
9674:   END_HANDLE_TH_ERRORS
9675: }
9676: 
9677: \
9678: // kthvalue
9679: static PyObject * THPVariable_kthvalue(PyObject* self_, PyObject* args, PyObject* kwargs)
9680: {
9681:   HANDLE_TH_ERRORS
9682:   static PyTypeObject* NamedTuple = generated::get_kthvalue_structseq();
9683:   const Tensor& self = THPVariable_Unpack(self_);
9684:   static PythonArgParser parser({
9685:     "kthvalue(SymInt k, int64_t dim=-1, bool keepdim=False)",
9686:     "kthvalue(SymInt k, Dimname dim, bool keepdim=False)",
9687:   }, /*traceable=*/true);
9688: 
9689:   ParsedArgs<3> parsed_args;
9690:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9691:   if(_r.has_torch_function()) {
9692:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9693:   }
9694:   switch (_r.idx) {
9695:     case 0: {
9696:       // aten::kthvalue(Tensor self, SymInt k, int dim=-1, bool keepdim=False) -> (Tensor values, Tensor indices)
9697: 
9698:       auto dispatch_kthvalue = [](const at::Tensor & self, c10::SymInt k, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
9699:         pybind11::gil_scoped_release no_gil;
9700:         return self.kthvalue_symint(k, dim, keepdim);
9701:       };
9702:       return wrap(NamedTuple, dispatch_kthvalue(self, _r.toSymInt(0), _r.toInt64(1), _r.toBool(2)));
9703:     }
9704:     case 1: {
9705:       // aten::kthvalue.dimname(Tensor self, SymInt k, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
9706: 
9707:       auto dispatch_kthvalue = [](const at::Tensor & self, c10::SymInt k, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
9708:         pybind11::gil_scoped_release no_gil;
9709:         return self.kthvalue_symint(k, dim, keepdim);
9710:       };
9711:       return wrap(NamedTuple, dispatch_kthvalue(self, _r.toSymInt(0), _r.dimname(1), _r.toBool(2)));
9712:     }
9713:   }
9714:   Py_RETURN_NONE;
9715:   END_HANDLE_TH_ERRORS
9716: }
9717: 
9718: // lcm
9719: static PyObject * THPVariable_lcm(PyObject* self_, PyObject* args, PyObject* kwargs)
9720: {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_isreal`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_isreal`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9721-9840

```cpp
9721:   HANDLE_TH_ERRORS
9722:   const Tensor& self = THPVariable_Unpack(self_);
9723:   static PythonArgParser parser({
9724:     "lcm(Tensor other)",
9725:   }, /*traceable=*/true);
9726: 
9727:   ParsedArgs<1> parsed_args;
9728:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9729:   if(_r.has_torch_function()) {
9730:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9731:   }
9732:   // aten::lcm(Tensor self, Tensor other) -> Tensor
9733: 
9734:   auto dispatch_lcm = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9735:     pybind11::gil_scoped_release no_gil;
9736:     return self.lcm(other);
9737:   };
9738:   return wrap(dispatch_lcm(self, _r.tensor(0)));
9739:   Py_RETURN_NONE;
9740:   END_HANDLE_TH_ERRORS
9741: }
9742: 
9743: // lcm_
9744: static PyObject * THPVariable_lcm_(PyObject* self_, PyObject* args, PyObject* kwargs)
9745: {
9746:   HANDLE_TH_ERRORS
9747:   const Tensor& self = THPVariable_Unpack(self_);
9748:   static PythonArgParser parser({
9749:     "lcm_(Tensor other)",
9750:   }, /*traceable=*/true);
9751: 
9752:   ParsedArgs<1> parsed_args;
9753:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9754:   if(_r.has_torch_function()) {
9755:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9756:   }
9757:   // aten::lcm_(Tensor(a!) self, Tensor other) -> Tensor(a!)
9758: 
9759:   auto dispatch_lcm_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9760:     pybind11::gil_scoped_release no_gil;
9761:     return self.lcm_(other);
9762:   };
9763:   return wrap(dispatch_lcm_(self, _r.tensor(0)));
9764:   Py_RETURN_NONE;
9765:   END_HANDLE_TH_ERRORS
9766: }
9767: 
9768: // ldexp
9769: static PyObject * THPVariable_ldexp(PyObject* self_, PyObject* args, PyObject* kwargs)
9770: {
9771:   HANDLE_TH_ERRORS
9772:   const Tensor& self = THPVariable_Unpack(self_);
9773:   static PythonArgParser parser({
9774:     "ldexp(Tensor other)",
9775:   }, /*traceable=*/true);
9776: 
9777:   ParsedArgs<1> parsed_args;
9778:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9779:   if(_r.has_torch_function()) {
9780:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9781:   }
9782:   // aten::ldexp.Tensor(Tensor self, Tensor other) -> Tensor
9783: 
9784:   auto dispatch_ldexp = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9785:     pybind11::gil_scoped_release no_gil;
9786:     return self.ldexp(other);
9787:   };
9788:   return wrap(dispatch_ldexp(self, _r.tensor(0)));
9789:   Py_RETURN_NONE;
9790:   END_HANDLE_TH_ERRORS
9791: }
9792: 
9793: // ldexp_
9794: static PyObject * THPVariable_ldexp_(PyObject* self_, PyObject* args, PyObject* kwargs)
9795: {
9796:   HANDLE_TH_ERRORS
9797:   const Tensor& self = THPVariable_Unpack(self_);
9798:   static PythonArgParser parser({
9799:     "ldexp_(Tensor other)",
9800:   }, /*traceable=*/true);
9801: 
9802:   ParsedArgs<1> parsed_args;
9803:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9804:   if(_r.has_torch_function()) {
9805:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9806:   }
9807:   // aten::ldexp_(Tensor(a!) self, Tensor other) -> Tensor(a!)
9808: 
9809:   auto dispatch_ldexp_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9810:     pybind11::gil_scoped_release no_gil;
9811:     return self.ldexp_(other);
9812:   };
9813:   return wrap(dispatch_ldexp_(self, _r.tensor(0)));
9814:   Py_RETURN_NONE;
9815:   END_HANDLE_TH_ERRORS
9816: }
9817: 
9818: \
9819: // le
9820: static PyObject * THPVariable_le(PyObject* self_, PyObject* args, PyObject* kwargs)
9821: {
9822:   HANDLE_TH_ERRORS
9823:   const Tensor& self = THPVariable_Unpack(self_);
9824:   static PythonArgParser parser({
9825:     "le(Tensor other)",
9826:     "le(Scalar other)",
9827:   }, /*traceable=*/true);
9828: 
9829:   ParsedArgs<1> parsed_args;
9830:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9831:   if(_r.has_torch_function()) {
9832:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9833:   }
9834:   switch (_r.idx) {
9835:     case 0: {
9836:       // aten::le.Tensor(Tensor self, Tensor other) -> Tensor
9837: 
9838:       auto dispatch_le = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9839:         pybind11::gil_scoped_release no_gil;
9840:         return self.le(other);
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9841-9960

```cpp
9841:       };
9842:       return wrap(dispatch_le(self, _r.tensor(0)));
9843:     }
9844:     case 1: {
9845:       // aten::le.Scalar(Tensor self, Scalar other) -> Tensor
9846: 
9847:       auto dispatch_le = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9848:         pybind11::gil_scoped_release no_gil;
9849:         return self.le(other);
9850:       };
9851:       return wrap(dispatch_le(self, _r.scalar(0)));
9852:     }
9853:   }
9854:   Py_RETURN_NONE;
9855:   END_HANDLE_TH_ERRORS
9856: }
9857: 
9858: \
9859: // le_
9860: static PyObject * THPVariable_le_(PyObject* self_, PyObject* args, PyObject* kwargs)
9861: {
9862:   HANDLE_TH_ERRORS
9863:   const Tensor& self = THPVariable_Unpack(self_);
9864:   static PythonArgParser parser({
9865:     "le_(Tensor other)",
9866:     "le_(Scalar other)",
9867:   }, /*traceable=*/true);
9868: 
9869:   ParsedArgs<1> parsed_args;
9870:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9871:   if(_r.has_torch_function()) {
9872:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9873:   }
9874:   switch (_r.idx) {
9875:     case 0: {
9876:       // aten::le_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
9877: 
9878:       auto dispatch_le_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
9879:         pybind11::gil_scoped_release no_gil;
9880:         return self.le_(other);
9881:       };
9882:       return wrap(dispatch_le_(self, _r.tensor(0)));
9883:     }
9884:     case 1: {
9885:       // aten::le_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
9886: 
9887:       auto dispatch_le_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
9888:         pybind11::gil_scoped_release no_gil;
9889:         return self.le_(other);
9890:       };
9891:       return wrap(dispatch_le_(self, _r.scalar(0)));
9892:     }
9893:   }
9894:   Py_RETURN_NONE;
9895:   END_HANDLE_TH_ERRORS
9896: }
9897: 
9898: \
9899: // lerp
9900: static PyObject * THPVariable_lerp(PyObject* self_, PyObject* args, PyObject* kwargs)
9901: {
9902:   HANDLE_TH_ERRORS
9903:   const Tensor& self = THPVariable_Unpack(self_);
9904:   static PythonArgParser parser({
9905:     "lerp(Tensor end, Tensor weight)",
9906:     "lerp(Tensor end, Scalar weight)",
9907:   }, /*traceable=*/true);
9908: 
9909:   ParsedArgs<2> parsed_args;
9910:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9911:   if(_r.has_torch_function()) {
9912:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9913:   }
9914:   switch (_r.idx) {
9915:     case 0: {
9916:       // aten::lerp.Tensor(Tensor self, Tensor end, Tensor weight) -> Tensor
9917: 
9918:       auto dispatch_lerp = [](const at::Tensor & self, const at::Tensor & end, const at::Tensor & weight) -> at::Tensor {
9919:         pybind11::gil_scoped_release no_gil;
9920:         return self.lerp(end, weight);
9921:       };
9922:       return wrap(dispatch_lerp(self, _r.tensor(0), _r.tensor(1)));
9923:     }
9924:     case 1: {
9925:       // aten::lerp.Scalar(Tensor self, Tensor end, Scalar weight) -> Tensor
9926: 
9927:       auto dispatch_lerp = [](const at::Tensor & self, const at::Tensor & end, const at::Scalar & weight) -> at::Tensor {
9928:         pybind11::gil_scoped_release no_gil;
9929:         return self.lerp(end, weight);
9930:       };
9931:       return wrap(dispatch_lerp(self, _r.tensor(0), _r.scalar(1)));
9932:     }
9933:   }
9934:   Py_RETURN_NONE;
9935:   END_HANDLE_TH_ERRORS
9936: }
9937: 
9938: \
9939: // lerp_
9940: static PyObject * THPVariable_lerp_(PyObject* self_, PyObject* args, PyObject* kwargs)
9941: {
9942:   HANDLE_TH_ERRORS
9943:   const Tensor& self = THPVariable_Unpack(self_);
9944:   static PythonArgParser parser({
9945:     "lerp_(Tensor end, Tensor weight)",
9946:     "lerp_(Tensor end, Scalar weight)",
9947:   }, /*traceable=*/true);
9948: 
9949:   ParsedArgs<2> parsed_args;
9950:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
9951:   if(_r.has_torch_function()) {
9952:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
9953:   }
9954:   switch (_r.idx) {
9955:     case 0: {
9956:       // aten::lerp_.Tensor(Tensor(a!) self, Tensor end, Tensor weight) -> Tensor(a!)
9957: 
9958:       auto dispatch_lerp_ = [](const at::Tensor & self, const at::Tensor & end, const at::Tensor & weight) -> at::Tensor {
9959:         pybind11::gil_scoped_release no_gil;
9960:         return self.lerp_(end, weight);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_le_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_le_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 9961-10080

```cpp
 9961:       };
 9962:       return wrap(dispatch_lerp_(self, _r.tensor(0), _r.tensor(1)));
 9963:     }
 9964:     case 1: {
 9965:       // aten::lerp_.Scalar(Tensor(a!) self, Tensor end, Scalar weight) -> Tensor(a!)
 9966: 
 9967:       auto dispatch_lerp_ = [](const at::Tensor & self, const at::Tensor & end, const at::Scalar & weight) -> at::Tensor {
 9968:         pybind11::gil_scoped_release no_gil;
 9969:         return self.lerp_(end, weight);
 9970:       };
 9971:       return wrap(dispatch_lerp_(self, _r.tensor(0), _r.scalar(1)));
 9972:     }
 9973:   }
 9974:   Py_RETURN_NONE;
 9975:   END_HANDLE_TH_ERRORS
 9976: }
 9977: 
 9978: \
 9979: // less
 9980: static PyObject * THPVariable_less(PyObject* self_, PyObject* args, PyObject* kwargs)
 9981: {
 9982:   HANDLE_TH_ERRORS
 9983:   const Tensor& self = THPVariable_Unpack(self_);
 9984:   static PythonArgParser parser({
 9985:     "less(Tensor other)",
 9986:     "less(Scalar other)",
 9987:   }, /*traceable=*/true);
 9988: 
 9989:   ParsedArgs<1> parsed_args;
 9990:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
 9991:   if(_r.has_torch_function()) {
 9992:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
 9993:   }
 9994:   switch (_r.idx) {
 9995:     case 0: {
 9996:       // aten::less.Tensor(Tensor self, Tensor other) -> Tensor
 9997: 
 9998:       auto dispatch_less = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
 9999:         pybind11::gil_scoped_release no_gil;
10000:         return self.less(other);
10001:       };
10002:       return wrap(dispatch_less(self, _r.tensor(0)));
10003:     }
10004:     case 1: {
10005:       // aten::less.Scalar(Tensor self, Scalar other) -> Tensor
10006: 
10007:       auto dispatch_less = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
10008:         pybind11::gil_scoped_release no_gil;
10009:         return self.less(other);
10010:       };
10011:       return wrap(dispatch_less(self, _r.scalar(0)));
10012:     }
10013:   }
10014:   Py_RETURN_NONE;
10015:   END_HANDLE_TH_ERRORS
10016: }
10017: 
10018: \
10019: // less_
10020: static PyObject * THPVariable_less_(PyObject* self_, PyObject* args, PyObject* kwargs)
10021: {
10022:   HANDLE_TH_ERRORS
10023:   const Tensor& self = THPVariable_Unpack(self_);
10024:   static PythonArgParser parser({
10025:     "less_(Tensor other)",
10026:     "less_(Scalar other)",
10027:   }, /*traceable=*/true);
10028: 
10029:   ParsedArgs<1> parsed_args;
10030:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10031:   if(_r.has_torch_function()) {
10032:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10033:   }
10034:   switch (_r.idx) {
10035:     case 0: {
10036:       // aten::less_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
10037: 
10038:       auto dispatch_less_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10039:         pybind11::gil_scoped_release no_gil;
10040:         return self.less_(other);
10041:       };
10042:       return wrap(dispatch_less_(self, _r.tensor(0)));
10043:     }
10044:     case 1: {
10045:       // aten::less_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
10046: 
10047:       auto dispatch_less_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
10048:         pybind11::gil_scoped_release no_gil;
10049:         return self.less_(other);
10050:       };
10051:       return wrap(dispatch_less_(self, _r.scalar(0)));
10052:     }
10053:   }
10054:   Py_RETURN_NONE;
10055:   END_HANDLE_TH_ERRORS
10056: }
10057: 
10058: \
10059: // less_equal
10060: static PyObject * THPVariable_less_equal(PyObject* self_, PyObject* args, PyObject* kwargs)
10061: {
10062:   HANDLE_TH_ERRORS
10063:   const Tensor& self = THPVariable_Unpack(self_);
10064:   static PythonArgParser parser({
10065:     "less_equal(Tensor other)",
10066:     "less_equal(Scalar other)",
10067:   }, /*traceable=*/true);
10068: 
10069:   ParsedArgs<1> parsed_args;
10070:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10071:   if(_r.has_torch_function()) {
10072:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10073:   }
10074:   switch (_r.idx) {
10075:     case 0: {
10076:       // aten::less_equal.Tensor(Tensor self, Tensor other) -> Tensor
10077: 
10078:       auto dispatch_less_equal = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10079:         pybind11::gil_scoped_release no_gil;
10080:         return self.less_equal(other);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_less`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_less`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10081-10200

```cpp
10081:       };
10082:       return wrap(dispatch_less_equal(self, _r.tensor(0)));
10083:     }
10084:     case 1: {
10085:       // aten::less_equal.Scalar(Tensor self, Scalar other) -> Tensor
10086: 
10087:       auto dispatch_less_equal = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
10088:         pybind11::gil_scoped_release no_gil;
10089:         return self.less_equal(other);
10090:       };
10091:       return wrap(dispatch_less_equal(self, _r.scalar(0)));
10092:     }
10093:   }
10094:   Py_RETURN_NONE;
10095:   END_HANDLE_TH_ERRORS
10096: }
10097: 
10098: \
10099: // less_equal_
10100: static PyObject * THPVariable_less_equal_(PyObject* self_, PyObject* args, PyObject* kwargs)
10101: {
10102:   HANDLE_TH_ERRORS
10103:   const Tensor& self = THPVariable_Unpack(self_);
10104:   static PythonArgParser parser({
10105:     "less_equal_(Tensor other)",
10106:     "less_equal_(Scalar other)",
10107:   }, /*traceable=*/true);
10108: 
10109:   ParsedArgs<1> parsed_args;
10110:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10111:   if(_r.has_torch_function()) {
10112:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10113:   }
10114:   switch (_r.idx) {
10115:     case 0: {
10116:       // aten::less_equal_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
10117: 
10118:       auto dispatch_less_equal_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10119:         pybind11::gil_scoped_release no_gil;
10120:         return self.less_equal_(other);
10121:       };
10122:       return wrap(dispatch_less_equal_(self, _r.tensor(0)));
10123:     }
10124:     case 1: {
10125:       // aten::less_equal_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
10126: 
10127:       auto dispatch_less_equal_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
10128:         pybind11::gil_scoped_release no_gil;
10129:         return self.less_equal_(other);
10130:       };
10131:       return wrap(dispatch_less_equal_(self, _r.scalar(0)));
10132:     }
10133:   }
10134:   Py_RETURN_NONE;
10135:   END_HANDLE_TH_ERRORS
10136: }
10137: 
10138: // lgamma
10139: static PyObject * THPVariable_lgamma(PyObject* self_, PyObject* args)
10140: {
10141:   HANDLE_TH_ERRORS
10142:   const Tensor& self = THPVariable_Unpack(self_);
10143:   if (has_torch_function(self_)) {
10144:     return handle_torch_function(self_, "lgamma");
10145:   }
10146:   // aten::lgamma(Tensor self) -> Tensor
10147: 
10148:   auto dispatch_lgamma = [](const at::Tensor & self) -> at::Tensor {
10149:     pybind11::gil_scoped_release no_gil;
10150:     return self.lgamma();
10151:   };
10152:   return wrap(dispatch_lgamma(self));
10153:   END_HANDLE_TH_ERRORS
10154: }
10155: 
10156: // lgamma_
10157: static PyObject * THPVariable_lgamma_(PyObject* self_, PyObject* args)
10158: {
10159:   HANDLE_TH_ERRORS
10160:   const Tensor& self = THPVariable_Unpack(self_);
10161:   if (has_torch_function(self_)) {
10162:     return handle_torch_function(self_, "lgamma_");
10163:   }
10164:   // aten::lgamma_(Tensor(a!) self) -> Tensor(a!)
10165: 
10166:   auto dispatch_lgamma_ = [](const at::Tensor & self) -> at::Tensor {
10167:     pybind11::gil_scoped_release no_gil;
10168:     return self.lgamma_();
10169:   };
10170:   return wrap(dispatch_lgamma_(self));
10171:   END_HANDLE_TH_ERRORS
10172: }
10173: 
10174: // log
10175: static PyObject * THPVariable_log(PyObject* self_, PyObject* args)
10176: {
10177:   HANDLE_TH_ERRORS
10178:   const Tensor& self = THPVariable_Unpack(self_);
10179:   if (has_torch_function(self_)) {
10180:     return handle_torch_function(self_, "log");
10181:   }
10182:   // aten::log(Tensor self) -> Tensor
10183: 
10184:   auto dispatch_log = [](const at::Tensor & self) -> at::Tensor {
10185:     pybind11::gil_scoped_release no_gil;
10186:     return self.log();
10187:   };
10188:   return wrap(dispatch_log(self));
10189:   END_HANDLE_TH_ERRORS
10190: }
10191: 
10192: // log10
10193: static PyObject * THPVariable_log10(PyObject* self_, PyObject* args)
10194: {
10195:   HANDLE_TH_ERRORS
10196:   const Tensor& self = THPVariable_Unpack(self_);
10197:   if (has_torch_function(self_)) {
10198:     return handle_torch_function(self_, "log10");
10199:   }
10200:   // aten::log10(Tensor self) -> Tensor
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_less_equal_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_less_equal_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10201-10320

```cpp
10201: 
10202:   auto dispatch_log10 = [](const at::Tensor & self) -> at::Tensor {
10203:     pybind11::gil_scoped_release no_gil;
10204:     return self.log10();
10205:   };
10206:   return wrap(dispatch_log10(self));
10207:   END_HANDLE_TH_ERRORS
10208: }
10209: 
10210: // log10_
10211: static PyObject * THPVariable_log10_(PyObject* self_, PyObject* args)
10212: {
10213:   HANDLE_TH_ERRORS
10214:   const Tensor& self = THPVariable_Unpack(self_);
10215:   if (has_torch_function(self_)) {
10216:     return handle_torch_function(self_, "log10_");
10217:   }
10218:   // aten::log10_(Tensor(a!) self) -> Tensor(a!)
10219: 
10220:   auto dispatch_log10_ = [](const at::Tensor & self) -> at::Tensor {
10221:     pybind11::gil_scoped_release no_gil;
10222:     return self.log10_();
10223:   };
10224:   return wrap(dispatch_log10_(self));
10225:   END_HANDLE_TH_ERRORS
10226: }
10227: 
10228: // log1p
10229: static PyObject * THPVariable_log1p(PyObject* self_, PyObject* args)
10230: {
10231:   HANDLE_TH_ERRORS
10232:   const Tensor& self = THPVariable_Unpack(self_);
10233:   if (has_torch_function(self_)) {
10234:     return handle_torch_function(self_, "log1p");
10235:   }
10236:   // aten::log1p(Tensor self) -> Tensor
10237: 
10238:   auto dispatch_log1p = [](const at::Tensor & self) -> at::Tensor {
10239:     pybind11::gil_scoped_release no_gil;
10240:     return self.log1p();
10241:   };
10242:   return wrap(dispatch_log1p(self));
10243:   END_HANDLE_TH_ERRORS
10244: }
10245: 
10246: // log1p_
10247: static PyObject * THPVariable_log1p_(PyObject* self_, PyObject* args)
10248: {
10249:   HANDLE_TH_ERRORS
10250:   const Tensor& self = THPVariable_Unpack(self_);
10251:   if (has_torch_function(self_)) {
10252:     return handle_torch_function(self_, "log1p_");
10253:   }
10254:   // aten::log1p_(Tensor(a!) self) -> Tensor(a!)
10255: 
10256:   auto dispatch_log1p_ = [](const at::Tensor & self) -> at::Tensor {
10257:     pybind11::gil_scoped_release no_gil;
10258:     return self.log1p_();
10259:   };
10260:   return wrap(dispatch_log1p_(self));
10261:   END_HANDLE_TH_ERRORS
10262: }
10263: 
10264: // log2
10265: static PyObject * THPVariable_log2(PyObject* self_, PyObject* args)
10266: {
10267:   HANDLE_TH_ERRORS
10268:   const Tensor& self = THPVariable_Unpack(self_);
10269:   if (has_torch_function(self_)) {
10270:     return handle_torch_function(self_, "log2");
10271:   }
10272:   // aten::log2(Tensor self) -> Tensor
10273: 
10274:   auto dispatch_log2 = [](const at::Tensor & self) -> at::Tensor {
10275:     pybind11::gil_scoped_release no_gil;
10276:     return self.log2();
10277:   };
10278:   return wrap(dispatch_log2(self));
10279:   END_HANDLE_TH_ERRORS
10280: }
10281: 
10282: // log2_
10283: static PyObject * THPVariable_log2_(PyObject* self_, PyObject* args)
10284: {
10285:   HANDLE_TH_ERRORS
10286:   const Tensor& self = THPVariable_Unpack(self_);
10287:   if (has_torch_function(self_)) {
10288:     return handle_torch_function(self_, "log2_");
10289:   }
10290:   // aten::log2_(Tensor(a!) self) -> Tensor(a!)
10291: 
10292:   auto dispatch_log2_ = [](const at::Tensor & self) -> at::Tensor {
10293:     pybind11::gil_scoped_release no_gil;
10294:     return self.log2_();
10295:   };
10296:   return wrap(dispatch_log2_(self));
10297:   END_HANDLE_TH_ERRORS
10298: }
10299: 
10300: // log_
10301: static PyObject * THPVariable_log_(PyObject* self_, PyObject* args)
10302: {
10303:   HANDLE_TH_ERRORS
10304:   const Tensor& self = THPVariable_Unpack(self_);
10305:   if (has_torch_function(self_)) {
10306:     return handle_torch_function(self_, "log_");
10307:   }
10308:   // aten::log_(Tensor(a!) self) -> Tensor(a!)
10309: 
10310:   auto dispatch_log_ = [](const at::Tensor & self) -> at::Tensor {
10311:     pybind11::gil_scoped_release no_gil;
10312:     return self.log_();
10313:   };
10314:   return wrap(dispatch_log_(self));
10315:   END_HANDLE_TH_ERRORS
10316: }
10317: 
10318: // log_normal_
10319: static PyObject * THPVariable_log_normal_(PyObject* self_, PyObject* args, PyObject* kwargs)
10320: {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_log10_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_log10_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10321-10440

```cpp
10321:   HANDLE_TH_ERRORS
10322:   const Tensor& self = THPVariable_Unpack(self_);
10323:   static PythonArgParser parser({
10324:     "log_normal_(double mean=1, double std=2, *, Generator? generator=None)",
10325:   }, /*traceable=*/true);
10326: 
10327:   ParsedArgs<3> parsed_args;
10328:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10329:   if(_r.has_torch_function()) {
10330:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10331:   }
10332:   // aten::log_normal_(Tensor(a!) self, float mean=1, float std=2, *, Generator? generator=None) -> Tensor(a!)
10333: 
10334:   auto dispatch_log_normal_ = [](const at::Tensor & self, double mean, double std, ::std::optional<at::Generator> generator) -> at::Tensor {
10335:     pybind11::gil_scoped_release no_gil;
10336:     return self.log_normal_(mean, std, generator);
10337:   };
10338:   return wrap(dispatch_log_normal_(self, _r.toDouble(0), _r.toDouble(1), _r.generator(2)));
10339:   Py_RETURN_NONE;
10340:   END_HANDLE_TH_ERRORS
10341: }
10342: 
10343: \
10344: // log_softmax
10345: static PyObject * THPVariable_log_softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
10346: {
10347:   HANDLE_TH_ERRORS
10348:   const Tensor& self = THPVariable_Unpack(self_);
10349:   static PythonArgParser parser({
10350:     "log_softmax(int64_t dim, ScalarType? dtype=None)",
10351:     "log_softmax(Dimname dim, *, ScalarType? dtype=None)",
10352:   }, /*traceable=*/true);
10353: 
10354:   ParsedArgs<2> parsed_args;
10355:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10356:   if(_r.has_torch_function()) {
10357:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10358:   }
10359:   switch (_r.idx) {
10360:     case 0: {
10361:       // aten::log_softmax.int(Tensor self, int dim, ScalarType? dtype=None) -> Tensor
10362: 
10363:       auto dispatch_log_softmax = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
10364:         pybind11::gil_scoped_release no_gil;
10365:         return self.log_softmax(dim, dtype);
10366:       };
10367:       return wrap(dispatch_log_softmax(self, _r.toInt64(0), _r.scalartypeOptional(1)));
10368:     }
10369:     case 1: {
10370:       // aten::log_softmax.Dimname(Tensor self, Dimname dim, *, ScalarType? dtype=None) -> Tensor
10371: 
10372:       auto dispatch_log_softmax = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
10373:         pybind11::gil_scoped_release no_gil;
10374:         return self.log_softmax(dim, dtype);
10375:       };
10376:       return wrap(dispatch_log_softmax(self, _r.dimname(0), _r.scalartypeOptional(1)));
10377:     }
10378:   }
10379:   Py_RETURN_NONE;
10380:   END_HANDLE_TH_ERRORS
10381: }
10382: 
10383: // logaddexp
10384: static PyObject * THPVariable_logaddexp(PyObject* self_, PyObject* args, PyObject* kwargs)
10385: {
10386:   HANDLE_TH_ERRORS
10387:   const Tensor& self = THPVariable_Unpack(self_);
10388:   static PythonArgParser parser({
10389:     "logaddexp(Tensor other)",
10390:   }, /*traceable=*/true);
10391: 
10392:   ParsedArgs<1> parsed_args;
10393:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10394:   if(_r.has_torch_function()) {
10395:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10396:   }
10397:   // aten::logaddexp(Tensor self, Tensor other) -> Tensor
10398: 
10399:   auto dispatch_logaddexp = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10400:     pybind11::gil_scoped_release no_gil;
10401:     return self.logaddexp(other);
10402:   };
10403:   return wrap(dispatch_logaddexp(self, _r.tensor(0)));
10404:   Py_RETURN_NONE;
10405:   END_HANDLE_TH_ERRORS
10406: }
10407: 
10408: // logaddexp2
10409: static PyObject * THPVariable_logaddexp2(PyObject* self_, PyObject* args, PyObject* kwargs)
10410: {
10411:   HANDLE_TH_ERRORS
10412:   const Tensor& self = THPVariable_Unpack(self_);
10413:   static PythonArgParser parser({
10414:     "logaddexp2(Tensor other)",
10415:   }, /*traceable=*/true);
10416: 
10417:   ParsedArgs<1> parsed_args;
10418:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10419:   if(_r.has_torch_function()) {
10420:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10421:   }
10422:   // aten::logaddexp2(Tensor self, Tensor other) -> Tensor
10423: 
10424:   auto dispatch_logaddexp2 = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10425:     pybind11::gil_scoped_release no_gil;
10426:     return self.logaddexp2(other);
10427:   };
10428:   return wrap(dispatch_logaddexp2(self, _r.tensor(0)));
10429:   Py_RETURN_NONE;
10430:   END_HANDLE_TH_ERRORS
10431: }
10432: 
10433: \
10434: // logcumsumexp
10435: static PyObject * THPVariable_logcumsumexp(PyObject* self_, PyObject* args, PyObject* kwargs)
10436: {
10437:   HANDLE_TH_ERRORS
10438:   const Tensor& self = THPVariable_Unpack(self_);
10439:   static PythonArgParser parser({
10440:     "logcumsumexp(int64_t dim)",
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10441-10560

```cpp
10441:     "logcumsumexp(Dimname dim)",
10442:   }, /*traceable=*/true);
10443: 
10444:   ParsedArgs<1> parsed_args;
10445:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10446:   if(_r.has_torch_function()) {
10447:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10448:   }
10449:   switch (_r.idx) {
10450:     case 0: {
10451:       // aten::logcumsumexp(Tensor self, int dim) -> Tensor
10452: 
10453:       auto dispatch_logcumsumexp = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
10454:         pybind11::gil_scoped_release no_gil;
10455:         return self.logcumsumexp(dim);
10456:       };
10457:       return wrap(dispatch_logcumsumexp(self, _r.toInt64(0)));
10458:     }
10459:     case 1: {
10460:       // aten::logcumsumexp.dimname(Tensor self, Dimname dim) -> Tensor
10461: 
10462:       auto dispatch_logcumsumexp = [](const at::Tensor & self, at::Dimname dim) -> at::Tensor {
10463:         pybind11::gil_scoped_release no_gil;
10464:         return self.logcumsumexp(dim);
10465:       };
10466:       return wrap(dispatch_logcumsumexp(self, _r.dimname(0)));
10467:     }
10468:   }
10469:   Py_RETURN_NONE;
10470:   END_HANDLE_TH_ERRORS
10471: }
10472: 
10473: // logdet
10474: static PyObject * THPVariable_logdet(PyObject* self_, PyObject* args)
10475: {
10476:   HANDLE_TH_ERRORS
10477:   const Tensor& self = THPVariable_Unpack(self_);
10478:   if (has_torch_function(self_)) {
10479:     return handle_torch_function(self_, "logdet");
10480:   }
10481:   // aten::logdet(Tensor self) -> Tensor
10482: 
10483:   auto dispatch_logdet = [](const at::Tensor & self) -> at::Tensor {
10484:     pybind11::gil_scoped_release no_gil;
10485:     return self.logdet();
10486:   };
10487:   return wrap(dispatch_logdet(self));
10488:   END_HANDLE_TH_ERRORS
10489: }
10490: 
10491: // logical_and
10492: static PyObject * THPVariable_logical_and(PyObject* self_, PyObject* args, PyObject* kwargs)
10493: {
10494:   HANDLE_TH_ERRORS
10495:   const Tensor& self = THPVariable_Unpack(self_);
10496:   static PythonArgParser parser({
10497:     "logical_and(Tensor other)",
10498:   }, /*traceable=*/true);
10499: 
10500:   ParsedArgs<1> parsed_args;
10501:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10502:   if(_r.has_torch_function()) {
10503:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10504:   }
10505:   // aten::logical_and(Tensor self, Tensor other) -> Tensor
10506: 
10507:   auto dispatch_logical_and = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10508:     pybind11::gil_scoped_release no_gil;
10509:     return self.logical_and(other);
10510:   };
10511:   return wrap(dispatch_logical_and(self, _r.tensor(0)));
10512:   Py_RETURN_NONE;
10513:   END_HANDLE_TH_ERRORS
10514: }
10515: 
10516: // logical_and_
10517: static PyObject * THPVariable_logical_and_(PyObject* self_, PyObject* args, PyObject* kwargs)
10518: {
10519:   HANDLE_TH_ERRORS
10520:   const Tensor& self = THPVariable_Unpack(self_);
10521:   static PythonArgParser parser({
10522:     "logical_and_(Tensor other)",
10523:   }, /*traceable=*/true);
10524: 
10525:   ParsedArgs<1> parsed_args;
10526:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10527:   if(_r.has_torch_function()) {
10528:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10529:   }
10530:   // aten::logical_and_(Tensor(a!) self, Tensor other) -> Tensor(a!)
10531: 
10532:   auto dispatch_logical_and_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10533:     pybind11::gil_scoped_release no_gil;
10534:     return self.logical_and_(other);
10535:   };
10536:   return wrap(dispatch_logical_and_(self, _r.tensor(0)));
10537:   Py_RETURN_NONE;
10538:   END_HANDLE_TH_ERRORS
10539: }
10540: 
10541: // logical_not
10542: static PyObject * THPVariable_logical_not(PyObject* self_, PyObject* args)
10543: {
10544:   HANDLE_TH_ERRORS
10545:   const Tensor& self = THPVariable_Unpack(self_);
10546:   if (has_torch_function(self_)) {
10547:     return handle_torch_function(self_, "logical_not");
10548:   }
10549:   // aten::logical_not(Tensor self) -> Tensor
10550: 
10551:   auto dispatch_logical_not = [](const at::Tensor & self) -> at::Tensor {
10552:     pybind11::gil_scoped_release no_gil;
10553:     return self.logical_not();
10554:   };
10555:   return wrap(dispatch_logical_not(self));
10556:   END_HANDLE_TH_ERRORS
10557: }
10558: 
10559: // logical_not_
10560: static PyObject * THPVariable_logical_not_(PyObject* self_, PyObject* args)
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `logcumsumexp`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `logcumsumexp`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10561-10680

```cpp
10561: {
10562:   HANDLE_TH_ERRORS
10563:   const Tensor& self = THPVariable_Unpack(self_);
10564:   if (has_torch_function(self_)) {
10565:     return handle_torch_function(self_, "logical_not_");
10566:   }
10567:   // aten::logical_not_(Tensor(a!) self) -> Tensor(a!)
10568: 
10569:   auto dispatch_logical_not_ = [](const at::Tensor & self) -> at::Tensor {
10570:     pybind11::gil_scoped_release no_gil;
10571:     return self.logical_not_();
10572:   };
10573:   return wrap(dispatch_logical_not_(self));
10574:   END_HANDLE_TH_ERRORS
10575: }
10576: 
10577: // logical_or
10578: static PyObject * THPVariable_logical_or(PyObject* self_, PyObject* args, PyObject* kwargs)
10579: {
10580:   HANDLE_TH_ERRORS
10581:   const Tensor& self = THPVariable_Unpack(self_);
10582:   static PythonArgParser parser({
10583:     "logical_or(Tensor other)",
10584:   }, /*traceable=*/true);
10585: 
10586:   ParsedArgs<1> parsed_args;
10587:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10588:   if(_r.has_torch_function()) {
10589:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10590:   }
10591:   // aten::logical_or(Tensor self, Tensor other) -> Tensor
10592: 
10593:   auto dispatch_logical_or = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10594:     pybind11::gil_scoped_release no_gil;
10595:     return self.logical_or(other);
10596:   };
10597:   return wrap(dispatch_logical_or(self, _r.tensor(0)));
10598:   Py_RETURN_NONE;
10599:   END_HANDLE_TH_ERRORS
10600: }
10601: 
10602: // logical_or_
10603: static PyObject * THPVariable_logical_or_(PyObject* self_, PyObject* args, PyObject* kwargs)
10604: {
10605:   HANDLE_TH_ERRORS
10606:   const Tensor& self = THPVariable_Unpack(self_);
10607:   static PythonArgParser parser({
10608:     "logical_or_(Tensor other)",
10609:   }, /*traceable=*/true);
10610: 
10611:   ParsedArgs<1> parsed_args;
10612:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10613:   if(_r.has_torch_function()) {
10614:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10615:   }
10616:   // aten::logical_or_(Tensor(a!) self, Tensor other) -> Tensor(a!)
10617: 
10618:   auto dispatch_logical_or_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10619:     pybind11::gil_scoped_release no_gil;
10620:     return self.logical_or_(other);
10621:   };
10622:   return wrap(dispatch_logical_or_(self, _r.tensor(0)));
10623:   Py_RETURN_NONE;
10624:   END_HANDLE_TH_ERRORS
10625: }
10626: 
10627: // logical_xor
10628: static PyObject * THPVariable_logical_xor(PyObject* self_, PyObject* args, PyObject* kwargs)
10629: {
10630:   HANDLE_TH_ERRORS
10631:   const Tensor& self = THPVariable_Unpack(self_);
10632:   static PythonArgParser parser({
10633:     "logical_xor(Tensor other)",
10634:   }, /*traceable=*/true);
10635: 
10636:   ParsedArgs<1> parsed_args;
10637:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10638:   if(_r.has_torch_function()) {
10639:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10640:   }
10641:   // aten::logical_xor(Tensor self, Tensor other) -> Tensor
10642: 
10643:   auto dispatch_logical_xor = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10644:     pybind11::gil_scoped_release no_gil;
10645:     return self.logical_xor(other);
10646:   };
10647:   return wrap(dispatch_logical_xor(self, _r.tensor(0)));
10648:   Py_RETURN_NONE;
10649:   END_HANDLE_TH_ERRORS
10650: }
10651: 
10652: // logical_xor_
10653: static PyObject * THPVariable_logical_xor_(PyObject* self_, PyObject* args, PyObject* kwargs)
10654: {
10655:   HANDLE_TH_ERRORS
10656:   const Tensor& self = THPVariable_Unpack(self_);
10657:   static PythonArgParser parser({
10658:     "logical_xor_(Tensor other)",
10659:   }, /*traceable=*/true);
10660: 
10661:   ParsedArgs<1> parsed_args;
10662:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10663:   if(_r.has_torch_function()) {
10664:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10665:   }
10666:   // aten::logical_xor_(Tensor(a!) self, Tensor other) -> Tensor(a!)
10667: 
10668:   auto dispatch_logical_xor_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10669:     pybind11::gil_scoped_release no_gil;
10670:     return self.logical_xor_(other);
10671:   };
10672:   return wrap(dispatch_logical_xor_(self, _r.tensor(0)));
10673:   Py_RETURN_NONE;
10674:   END_HANDLE_TH_ERRORS
10675: }
10676: 
10677: // logit
10678: static PyObject * THPVariable_logit(PyObject* self_, PyObject* args, PyObject* kwargs)
10679: {
10680:   HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `handle_torch_function`, `logical_not_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `handle_torch_function`, `logical_not_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10681-10800

```cpp
10681:   const Tensor& self = THPVariable_Unpack(self_);
10682:   static PythonArgParser parser({
10683:     "logit(double? eps=None)",
10684:   }, /*traceable=*/true);
10685: 
10686:   ParsedArgs<1> parsed_args;
10687:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10688:   if(_r.has_torch_function()) {
10689:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10690:   }
10691:   // aten::logit(Tensor self, float? eps=None) -> Tensor
10692: 
10693:   auto dispatch_logit = [](const at::Tensor & self, ::std::optional<double> eps) -> at::Tensor {
10694:     pybind11::gil_scoped_release no_gil;
10695:     return self.logit(eps);
10696:   };
10697:   return wrap(dispatch_logit(self, _r.toDoubleOptional(0)));
10698:   Py_RETURN_NONE;
10699:   END_HANDLE_TH_ERRORS
10700: }
10701: 
10702: // logit_
10703: static PyObject * THPVariable_logit_(PyObject* self_, PyObject* args, PyObject* kwargs)
10704: {
10705:   HANDLE_TH_ERRORS
10706:   const Tensor& self = THPVariable_Unpack(self_);
10707:   static PythonArgParser parser({
10708:     "logit_(double? eps=None)",
10709:   }, /*traceable=*/true);
10710: 
10711:   ParsedArgs<1> parsed_args;
10712:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10713:   if(_r.has_torch_function()) {
10714:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10715:   }
10716:   // aten::logit_(Tensor(a!) self, float? eps=None) -> Tensor(a!)
10717: 
10718:   auto dispatch_logit_ = [](const at::Tensor & self, ::std::optional<double> eps) -> at::Tensor {
10719:     pybind11::gil_scoped_release no_gil;
10720:     return self.logit_(eps);
10721:   };
10722:   return wrap(dispatch_logit_(self, _r.toDoubleOptional(0)));
10723:   Py_RETURN_NONE;
10724:   END_HANDLE_TH_ERRORS
10725: }
10726: 
10727: \
10728: // logsumexp
10729: static PyObject * THPVariable_logsumexp(PyObject* self_, PyObject* args, PyObject* kwargs)
10730: {
10731:   HANDLE_TH_ERRORS
10732:   const Tensor& self = THPVariable_Unpack(self_);
10733:   static PythonArgParser parser({
10734:     "logsumexp(IntArrayRef[1] dim, bool keepdim=False)",
10735:     "logsumexp(DimnameList[1] dim, bool keepdim=False)",
10736:   }, /*traceable=*/true);
10737: 
10738:   ParsedArgs<2> parsed_args;
10739:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10740:   if(_r.has_torch_function()) {
10741:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10742:   }
10743:   switch (_r.idx) {
10744:     case 0: {
10745:       // aten::logsumexp(Tensor self, int[1] dim, bool keepdim=False) -> Tensor
10746: 
10747:       auto dispatch_logsumexp = [](const at::Tensor & self, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
10748:         pybind11::gil_scoped_release no_gil;
10749:         return self.logsumexp(dim, keepdim);
10750:       };
10751:       return wrap(dispatch_logsumexp(self, _r.intlist(0), _r.toBool(1)));
10752:     }
10753:     case 1: {
10754:       // aten::logsumexp.names(Tensor self, Dimname[1] dim, bool keepdim=False) -> Tensor
10755: 
10756:       auto dispatch_logsumexp = [](const at::Tensor & self, at::DimnameList dim, bool keepdim) -> at::Tensor {
10757:         pybind11::gil_scoped_release no_gil;
10758:         return self.logsumexp(dim, keepdim);
10759:       };
10760:       return wrap(dispatch_logsumexp(self, _r.dimnamelist(0), _r.toBool(1)));
10761:     }
10762:   }
10763:   Py_RETURN_NONE;
10764:   END_HANDLE_TH_ERRORS
10765: }
10766: 
10767: \
10768: // lt
10769: static PyObject * THPVariable_lt(PyObject* self_, PyObject* args, PyObject* kwargs)
10770: {
10771:   HANDLE_TH_ERRORS
10772:   const Tensor& self = THPVariable_Unpack(self_);
10773:   static PythonArgParser parser({
10774:     "lt(Tensor other)",
10775:     "lt(Scalar other)",
10776:   }, /*traceable=*/true);
10777: 
10778:   ParsedArgs<1> parsed_args;
10779:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10780:   if(_r.has_torch_function()) {
10781:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10782:   }
10783:   switch (_r.idx) {
10784:     case 0: {
10785:       // aten::lt.Tensor(Tensor self, Tensor other) -> Tensor
10786: 
10787:       auto dispatch_lt = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10788:         pybind11::gil_scoped_release no_gil;
10789:         return self.lt(other);
10790:       };
10791:       return wrap(dispatch_lt(self, _r.tensor(0)));
10792:     }
10793:     case 1: {
10794:       // aten::lt.Scalar(Tensor self, Scalar other) -> Tensor
10795: 
10796:       auto dispatch_lt = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
10797:         pybind11::gil_scoped_release no_gil;
10798:         return self.lt(other);
10799:       };
10800:       return wrap(dispatch_lt(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `parser`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `parser`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 10801-10920

```cpp
10801:     }
10802:   }
10803:   Py_RETURN_NONE;
10804:   END_HANDLE_TH_ERRORS
10805: }
10806: 
10807: \
10808: // lt_
10809: static PyObject * THPVariable_lt_(PyObject* self_, PyObject* args, PyObject* kwargs)
10810: {
10811:   HANDLE_TH_ERRORS
10812:   const Tensor& self = THPVariable_Unpack(self_);
10813:   static PythonArgParser parser({
10814:     "lt_(Tensor other)",
10815:     "lt_(Scalar other)",
10816:   }, /*traceable=*/true);
10817: 
10818:   ParsedArgs<1> parsed_args;
10819:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10820:   if(_r.has_torch_function()) {
10821:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10822:   }
10823:   switch (_r.idx) {
10824:     case 0: {
10825:       // aten::lt_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
10826: 
10827:       auto dispatch_lt_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
10828:         pybind11::gil_scoped_release no_gil;
10829:         return self.lt_(other);
10830:       };
10831:       return wrap(dispatch_lt_(self, _r.tensor(0)));
10832:     }
10833:     case 1: {
10834:       // aten::lt_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
10835: 
10836:       auto dispatch_lt_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
10837:         pybind11::gil_scoped_release no_gil;
10838:         return self.lt_(other);
10839:       };
10840:       return wrap(dispatch_lt_(self, _r.scalar(0)));
10841:     }
10842:   }
10843:   Py_RETURN_NONE;
10844:   END_HANDLE_TH_ERRORS
10845: }
10846: 
10847: // lu_solve
10848: static PyObject * THPVariable_lu_solve(PyObject* self_, PyObject* args, PyObject* kwargs)
10849: {
10850:   HANDLE_TH_ERRORS
10851:   const Tensor& self = THPVariable_Unpack(self_);
10852:   static PythonArgParser parser({
10853:     "lu_solve(Tensor LU_data, Tensor LU_pivots)",
10854:   }, /*traceable=*/true);
10855: 
10856:   ParsedArgs<2> parsed_args;
10857:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10858:   if(_r.has_torch_function()) {
10859:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10860:   }
10861:   // aten::lu_solve(Tensor self, Tensor LU_data, Tensor LU_pivots) -> Tensor
10862: 
10863:   auto dispatch_lu_solve = [](const at::Tensor & self, const at::Tensor & LU_data, const at::Tensor & LU_pivots) -> at::Tensor {
10864:     pybind11::gil_scoped_release no_gil;
10865:     return self.lu_solve(LU_data, LU_pivots);
10866:   };
10867:   return wrap(dispatch_lu_solve(self, _r.tensor(0), _r.tensor(1)));
10868:   Py_RETURN_NONE;
10869:   END_HANDLE_TH_ERRORS
10870: }
10871: 
10872: \
10873: // masked_fill
10874: static PyObject * THPVariable_masked_fill(PyObject* self_, PyObject* args, PyObject* kwargs)
10875: {
10876:   HANDLE_TH_ERRORS
10877:   const Tensor& self = THPVariable_Unpack(self_);
10878:   static PythonArgParser parser({
10879:     "masked_fill(Tensor mask, Tensor value)",
10880:     "masked_fill(Tensor mask, Scalar value)",
10881:   }, /*traceable=*/true);
10882: 
10883:   ParsedArgs<2> parsed_args;
10884:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10885:   if(_r.has_torch_function()) {
10886:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10887:   }
10888:   switch (_r.idx) {
10889:     case 0: {
10890:       // aten::masked_fill.Tensor(Tensor self, Tensor mask, Tensor value) -> Tensor
10891: 
10892:       auto dispatch_masked_fill = [](const at::Tensor & self, const at::Tensor & mask, const at::Tensor & value) -> at::Tensor {
10893:         pybind11::gil_scoped_release no_gil;
10894:         return self.masked_fill(mask, value);
10895:       };
10896:       return wrap(dispatch_masked_fill(self, _r.tensor(0), _r.tensor(1)));
10897:     }
10898:     case 1: {
10899:       // aten::masked_fill.Scalar(Tensor self, Tensor mask, Scalar value) -> Tensor
10900: 
10901:       auto dispatch_masked_fill = [](const at::Tensor & self, const at::Tensor & mask, const at::Scalar & value) -> at::Tensor {
10902:         pybind11::gil_scoped_release no_gil;
10903:         return self.masked_fill(mask, value);
10904:       };
10905:       return wrap(dispatch_masked_fill(self, _r.tensor(0), _r.scalar(1)));
10906:     }
10907:   }
10908:   Py_RETURN_NONE;
10909:   END_HANDLE_TH_ERRORS
10910: }
10911: 
10912: \
10913: // masked_fill_
10914: static PyObject * THPVariable_masked_fill_(PyObject* self_, PyObject* args, PyObject* kwargs)
10915: {
10916:   HANDLE_TH_ERRORS
10917:   const Tensor& self = THPVariable_Unpack(self_);
10918:   static PythonArgParser parser({
10919:     "masked_fill_(Tensor mask, Tensor value)",
10920:     "masked_fill_(Tensor mask, Scalar value)",
```

- EN: The main execution path in this span is carried by `THPVariable_lt_`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_lt_`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 10921-11040

```cpp
10921:   }, /*traceable=*/true);
10922: 
10923:   ParsedArgs<2> parsed_args;
10924:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10925:   if(_r.has_torch_function()) {
10926:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10927:   }
10928:   switch (_r.idx) {
10929:     case 0: {
10930:       // aten::masked_fill_.Tensor(Tensor(a!) self, Tensor mask, Tensor value) -> Tensor(a!)
10931: 
10932:       auto dispatch_masked_fill_ = [](const at::Tensor & self, const at::Tensor & mask, const at::Tensor & value) -> at::Tensor {
10933:         pybind11::gil_scoped_release no_gil;
10934:         return self.masked_fill_(mask, value);
10935:       };
10936:       return wrap(dispatch_masked_fill_(self, _r.tensor(0), _r.tensor(1)));
10937:     }
10938:     case 1: {
10939:       // aten::masked_fill_.Scalar(Tensor(a!) self, Tensor mask, Scalar value) -> Tensor(a!)
10940: 
10941:       auto dispatch_masked_fill_ = [](const at::Tensor & self, const at::Tensor & mask, const at::Scalar & value) -> at::Tensor {
10942:         pybind11::gil_scoped_release no_gil;
10943:         return self.masked_fill_(mask, value);
10944:       };
10945:       return wrap(dispatch_masked_fill_(self, _r.tensor(0), _r.scalar(1)));
10946:     }
10947:   }
10948:   Py_RETURN_NONE;
10949:   END_HANDLE_TH_ERRORS
10950: }
10951: 
10952: // masked_scatter
10953: static PyObject * THPVariable_masked_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
10954: {
10955:   HANDLE_TH_ERRORS
10956:   const Tensor& self = THPVariable_Unpack(self_);
10957:   static PythonArgParser parser({
10958:     "masked_scatter(Tensor mask, Tensor source)",
10959:   }, /*traceable=*/true);
10960: 
10961:   ParsedArgs<2> parsed_args;
10962:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10963:   if(_r.has_torch_function()) {
10964:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10965:   }
10966:   // aten::masked_scatter(Tensor self, Tensor mask, Tensor source) -> Tensor
10967: 
10968:   auto dispatch_masked_scatter = [](const at::Tensor & self, const at::Tensor & mask, const at::Tensor & source) -> at::Tensor {
10969:     pybind11::gil_scoped_release no_gil;
10970:     return self.masked_scatter(mask, source);
10971:   };
10972:   return wrap(dispatch_masked_scatter(self, _r.tensor(0), _r.tensor(1)));
10973:   Py_RETURN_NONE;
10974:   END_HANDLE_TH_ERRORS
10975: }
10976: 
10977: // masked_scatter_
10978: static PyObject * THPVariable_masked_scatter_(PyObject* self_, PyObject* args, PyObject* kwargs)
10979: {
10980:   HANDLE_TH_ERRORS
10981:   const Tensor& self = THPVariable_Unpack(self_);
10982:   static PythonArgParser parser({
10983:     "masked_scatter_(Tensor mask, Tensor source)",
10984:   }, /*traceable=*/true);
10985: 
10986:   ParsedArgs<2> parsed_args;
10987:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
10988:   if(_r.has_torch_function()) {
10989:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
10990:   }
10991:   // aten::masked_scatter_(Tensor(a!) self, Tensor mask, Tensor source) -> Tensor(a!)
10992: 
10993:   auto dispatch_masked_scatter_ = [](const at::Tensor & self, const at::Tensor & mask, const at::Tensor & source) -> at::Tensor {
10994:     pybind11::gil_scoped_release no_gil;
10995:     return self.masked_scatter_(mask, source);
10996:   };
10997:   return wrap(dispatch_masked_scatter_(self, _r.tensor(0), _r.tensor(1)));
10998:   Py_RETURN_NONE;
10999:   END_HANDLE_TH_ERRORS
11000: }
11001: 
11002: // masked_select
11003: static PyObject * THPVariable_masked_select(PyObject* self_, PyObject* args, PyObject* kwargs)
11004: {
11005:   HANDLE_TH_ERRORS
11006:   const Tensor& self = THPVariable_Unpack(self_);
11007:   static PythonArgParser parser({
11008:     "masked_select(Tensor mask)",
11009:   }, /*traceable=*/true);
11010: 
11011:   ParsedArgs<1> parsed_args;
11012:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11013:   if(_r.has_torch_function()) {
11014:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11015:   }
11016:   // aten::masked_select(Tensor self, Tensor mask) -> Tensor
11017: 
11018:   auto dispatch_masked_select = [](const at::Tensor & self, const at::Tensor & mask) -> at::Tensor {
11019:     pybind11::gil_scoped_release no_gil;
11020:     return self.masked_select(mask);
11021:   };
11022:   return wrap(dispatch_masked_select(self, _r.tensor(0)));
11023:   Py_RETURN_NONE;
11024:   END_HANDLE_TH_ERRORS
11025: }
11026: 
11027: // matmul
11028: static PyObject * THPVariable_matmul(PyObject* self_, PyObject* args, PyObject* kwargs)
11029: {
11030:   HANDLE_TH_ERRORS
11031:   const Tensor& self = THPVariable_Unpack(self_);
11032:   static PythonArgParser parser({
11033:     "matmul(Tensor other)",
11034:   }, /*traceable=*/true);
11035: 
11036:   ParsedArgs<1> parsed_args;
11037:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11038:   if(_r.has_torch_function()) {
11039:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11040:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable_masked_scatter`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable_masked_scatter` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11041-11160

```cpp
11041:   // aten::matmul(Tensor self, Tensor other) -> Tensor
11042: 
11043:   auto dispatch_matmul = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11044:     pybind11::gil_scoped_release no_gil;
11045:     return self.matmul(other);
11046:   };
11047:   return wrap(dispatch_matmul(self, _r.tensor(0)));
11048:   Py_RETURN_NONE;
11049:   END_HANDLE_TH_ERRORS
11050: }
11051: 
11052: // matrix_exp
11053: static PyObject * THPVariable_matrix_exp(PyObject* self_, PyObject* args)
11054: {
11055:   HANDLE_TH_ERRORS
11056:   const Tensor& self = THPVariable_Unpack(self_);
11057:   if (has_torch_function(self_)) {
11058:     return handle_torch_function(self_, "matrix_exp");
11059:   }
11060:   // aten::matrix_exp(Tensor self) -> Tensor
11061: 
11062:   auto dispatch_matrix_exp = [](const at::Tensor & self) -> at::Tensor {
11063:     pybind11::gil_scoped_release no_gil;
11064:     return self.matrix_exp();
11065:   };
11066:   return wrap(dispatch_matrix_exp(self));
11067:   END_HANDLE_TH_ERRORS
11068: }
11069: 
11070: // matrix_power
11071: static PyObject * THPVariable_matrix_power(PyObject* self_, PyObject* args, PyObject* kwargs)
11072: {
11073:   HANDLE_TH_ERRORS
11074:   const Tensor& self = THPVariable_Unpack(self_);
11075:   static PythonArgParser parser({
11076:     "matrix_power(int64_t n)",
11077:   }, /*traceable=*/true);
11078: 
11079:   ParsedArgs<1> parsed_args;
11080:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11081:   if(_r.has_torch_function()) {
11082:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11083:   }
11084:   // aten::matrix_power(Tensor self, int n) -> Tensor
11085: 
11086:   auto dispatch_matrix_power = [](const at::Tensor & self, int64_t n) -> at::Tensor {
11087:     pybind11::gil_scoped_release no_gil;
11088:     return self.matrix_power(n);
11089:   };
11090:   return wrap(dispatch_matrix_power(self, _r.toInt64(0)));
11091:   Py_RETURN_NONE;
11092:   END_HANDLE_TH_ERRORS
11093: }
11094: 
11095: \
11096: // max
11097: static PyObject * THPVariable_max(PyObject* self_, PyObject* args, PyObject* kwargs)
11098: {
11099:   HANDLE_TH_ERRORS
11100:   static PyTypeObject* NamedTuple = generated::get_max_structseq();
11101:   const Tensor& self = THPVariable_Unpack(self_);
11102:   static PythonArgParser parser({
11103:     "max()",
11104:     "max(Tensor other)",
11105:     "max(int64_t dim, bool keepdim=False)",
11106:     "max(Dimname dim, bool keepdim=False)",
11107:   }, /*traceable=*/true);
11108: 
11109:   ParsedArgs<2> parsed_args;
11110:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11111:   if(_r.has_torch_function()) {
11112:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11113:   }
11114:   switch (_r.idx) {
11115:     case 0: {
11116:       // aten::max(Tensor self) -> Tensor
11117: 
11118:       auto dispatch_max = [](const at::Tensor & self) -> at::Tensor {
11119:         pybind11::gil_scoped_release no_gil;
11120:         return self.max();
11121:       };
11122:       return wrap(dispatch_max(self));
11123:     }
11124:     case 1: {
11125:       // aten::max.other(Tensor self, Tensor other) -> Tensor
11126: 
11127:       auto dispatch_max = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11128:         pybind11::gil_scoped_release no_gil;
11129:         return self.max(other);
11130:       };
11131:       return wrap(dispatch_max(self, _r.tensor(0)));
11132:     }
11133:     case 2: {
11134:       // aten::max.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11135: 
11136:       auto dispatch_max = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11137:         pybind11::gil_scoped_release no_gil;
11138:         return self.max(dim, keepdim);
11139:       };
11140:       return wrap(NamedTuple, dispatch_max(self, _r.toInt64(0), _r.toBool(1)));
11141:     }
11142:     case 3: {
11143:       // aten::max.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11144: 
11145:       auto dispatch_max = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11146:         pybind11::gil_scoped_release no_gil;
11147:         return self.max(dim, keepdim);
11148:       };
11149:       return wrap(NamedTuple, dispatch_max(self, _r.dimname(0), _r.toBool(1)));
11150:     }
11151:   }
11152:   Py_RETURN_NONE;
11153:   END_HANDLE_TH_ERRORS
11154: }
11155: 
11156: // maximum
11157: static PyObject * THPVariable_maximum(PyObject* self_, PyObject* args, PyObject* kwargs)
11158: {
11159:   HANDLE_TH_ERRORS
11160:   const Tensor& self = THPVariable_Unpack(self_);
```

- EN: The main execution path in this span is carried by `matmul`, `wrap`, `THPVariable_matrix_exp`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `matmul`, `wrap`, `THPVariable_matrix_exp` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11161-11280

```cpp
11161:   static PythonArgParser parser({
11162:     "maximum(Tensor other)",
11163:   }, /*traceable=*/true);
11164: 
11165:   ParsedArgs<1> parsed_args;
11166:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11167:   if(_r.has_torch_function()) {
11168:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11169:   }
11170:   // aten::maximum(Tensor self, Tensor other) -> Tensor
11171: 
11172:   auto dispatch_maximum = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11173:     pybind11::gil_scoped_release no_gil;
11174:     return self.maximum(other);
11175:   };
11176:   return wrap(dispatch_maximum(self, _r.tensor(0)));
11177:   Py_RETURN_NONE;
11178:   END_HANDLE_TH_ERRORS
11179: }
11180: 
11181: \
11182: // mean
11183: static PyObject * THPVariable_mean(PyObject* self_, PyObject* args, PyObject* kwargs)
11184: {
11185:   HANDLE_TH_ERRORS
11186:   const Tensor& self = THPVariable_Unpack(self_);
11187:   static PythonArgParser parser({
11188:     "mean(*, ScalarType? dtype=None)",
11189:     "mean(IntArrayRef[1]? dim, bool keepdim=False, *, ScalarType? dtype=None)",
11190:     "mean(DimnameList[1] dim, bool keepdim=False, *, ScalarType? dtype=None)",
11191:   }, /*traceable=*/true);
11192: 
11193:   ParsedArgs<3> parsed_args;
11194:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11195:   if(_r.has_torch_function()) {
11196:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11197:   }
11198:   switch (_r.idx) {
11199:     case 0: {
11200:       // aten::mean(Tensor self, *, ScalarType? dtype=None) -> Tensor
11201: 
11202:       auto dispatch_mean = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
11203:         pybind11::gil_scoped_release no_gil;
11204:         return self.mean(dtype);
11205:       };
11206:       return wrap(dispatch_mean(self, _r.scalartypeOptional(0)));
11207:     }
11208:     case 1: {
11209:       // aten::mean.dim(Tensor self, int[1]? dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
11210: 
11211:       auto dispatch_mean = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
11212:         pybind11::gil_scoped_release no_gil;
11213:         return self.mean(dim, keepdim, dtype);
11214:       };
11215:       return wrap(dispatch_mean(self, _r.intlistOptional(0), _r.toBool(1), _r.scalartypeOptional(2)));
11216:     }
11217:     case 2: {
11218:       // aten::mean.names_dim(Tensor self, Dimname[1] dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
11219: 
11220:       auto dispatch_mean = [](const at::Tensor & self, at::DimnameList dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
11221:         pybind11::gil_scoped_release no_gil;
11222:         return self.mean(dim, keepdim, dtype);
11223:       };
11224:       return wrap(dispatch_mean(self, _r.dimnamelist(0), _r.toBool(1), _r.scalartypeOptional(2)));
11225:     }
11226:   }
11227:   Py_RETURN_NONE;
11228:   END_HANDLE_TH_ERRORS
11229: }
11230: 
11231: \
11232: // median
11233: static PyObject * THPVariable_median(PyObject* self_, PyObject* args, PyObject* kwargs)
11234: {
11235:   HANDLE_TH_ERRORS
11236:   static PyTypeObject* NamedTuple = generated::get_median_structseq();
11237:   const Tensor& self = THPVariable_Unpack(self_);
11238:   static PythonArgParser parser({
11239:     "median()",
11240:     "median(int64_t dim, bool keepdim=False)",
11241:     "median(Dimname dim, bool keepdim=False)",
11242:   }, /*traceable=*/true);
11243: 
11244:   ParsedArgs<2> parsed_args;
11245:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11246:   if(_r.has_torch_function()) {
11247:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11248:   }
11249:   switch (_r.idx) {
11250:     case 0: {
11251:       // aten::median(Tensor self) -> Tensor
11252: 
11253:       auto dispatch_median = [](const at::Tensor & self) -> at::Tensor {
11254:         pybind11::gil_scoped_release no_gil;
11255:         return self.median();
11256:       };
11257:       return wrap(dispatch_median(self));
11258:     }
11259:     case 1: {
11260:       // aten::median.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11261: 
11262:       auto dispatch_median = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11263:         pybind11::gil_scoped_release no_gil;
11264:         return self.median(dim, keepdim);
11265:       };
11266:       return wrap(NamedTuple, dispatch_median(self, _r.toInt64(0), _r.toBool(1)));
11267:     }
11268:     case 2: {
11269:       // aten::median.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11270: 
11271:       auto dispatch_median = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11272:         pybind11::gil_scoped_release no_gil;
11273:         return self.median(dim, keepdim);
11274:       };
11275:       return wrap(NamedTuple, dispatch_median(self, _r.dimname(0), _r.toBool(1)));
11276:     }
11277:   }
11278:   Py_RETURN_NONE;
11279:   END_HANDLE_TH_ERRORS
11280: }
```

- EN: The main execution path in this span is carried by `parser`, `handle_torch_function`, `maximum`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `parser`, `handle_torch_function`, `maximum` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11281-11400

```cpp
11281: 
11282: \
11283: // min
11284: static PyObject * THPVariable_min(PyObject* self_, PyObject* args, PyObject* kwargs)
11285: {
11286:   HANDLE_TH_ERRORS
11287:   static PyTypeObject* NamedTuple = generated::get_min_structseq();
11288:   const Tensor& self = THPVariable_Unpack(self_);
11289:   static PythonArgParser parser({
11290:     "min()",
11291:     "min(Tensor other)",
11292:     "min(int64_t dim, bool keepdim=False)",
11293:     "min(Dimname dim, bool keepdim=False)",
11294:   }, /*traceable=*/true);
11295: 
11296:   ParsedArgs<2> parsed_args;
11297:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11298:   if(_r.has_torch_function()) {
11299:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11300:   }
11301:   switch (_r.idx) {
11302:     case 0: {
11303:       // aten::min(Tensor self) -> Tensor
11304: 
11305:       auto dispatch_min = [](const at::Tensor & self) -> at::Tensor {
11306:         pybind11::gil_scoped_release no_gil;
11307:         return self.min();
11308:       };
11309:       return wrap(dispatch_min(self));
11310:     }
11311:     case 1: {
11312:       // aten::min.other(Tensor self, Tensor other) -> Tensor
11313: 
11314:       auto dispatch_min = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11315:         pybind11::gil_scoped_release no_gil;
11316:         return self.min(other);
11317:       };
11318:       return wrap(dispatch_min(self, _r.tensor(0)));
11319:     }
11320:     case 2: {
11321:       // aten::min.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11322: 
11323:       auto dispatch_min = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11324:         pybind11::gil_scoped_release no_gil;
11325:         return self.min(dim, keepdim);
11326:       };
11327:       return wrap(NamedTuple, dispatch_min(self, _r.toInt64(0), _r.toBool(1)));
11328:     }
11329:     case 3: {
11330:       // aten::min.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11331: 
11332:       auto dispatch_min = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11333:         pybind11::gil_scoped_release no_gil;
11334:         return self.min(dim, keepdim);
11335:       };
11336:       return wrap(NamedTuple, dispatch_min(self, _r.dimname(0), _r.toBool(1)));
11337:     }
11338:   }
11339:   Py_RETURN_NONE;
11340:   END_HANDLE_TH_ERRORS
11341: }
11342: 
11343: // minimum
11344: static PyObject * THPVariable_minimum(PyObject* self_, PyObject* args, PyObject* kwargs)
11345: {
11346:   HANDLE_TH_ERRORS
11347:   const Tensor& self = THPVariable_Unpack(self_);
11348:   static PythonArgParser parser({
11349:     "minimum(Tensor other)",
11350:   }, /*traceable=*/true);
11351: 
11352:   ParsedArgs<1> parsed_args;
11353:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11354:   if(_r.has_torch_function()) {
11355:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11356:   }
11357:   // aten::minimum(Tensor self, Tensor other) -> Tensor
11358: 
11359:   auto dispatch_minimum = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11360:     pybind11::gil_scoped_release no_gil;
11361:     return self.minimum(other);
11362:   };
11363:   return wrap(dispatch_minimum(self, _r.tensor(0)));
11364:   Py_RETURN_NONE;
11365:   END_HANDLE_TH_ERRORS
11366: }
11367: 
11368: // mm
11369: static PyObject * THPVariable_mm(PyObject* self_, PyObject* args, PyObject* kwargs)
11370: {
11371:   HANDLE_TH_ERRORS
11372:   const Tensor& self = THPVariable_Unpack(self_);
11373:   static PythonArgParser parser({
11374:     "mm(Tensor mat2)",
11375:   }, /*traceable=*/true);
11376: 
11377:   ParsedArgs<1> parsed_args;
11378:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11379:   if(_r.has_torch_function()) {
11380:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11381:   }
11382:   // aten::mm(Tensor self, Tensor mat2) -> Tensor
11383: 
11384:   auto dispatch_mm = [](const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
11385:     pybind11::gil_scoped_release no_gil;
11386:     return self.mm(mat2);
11387:   };
11388:   return wrap(dispatch_mm(self, _r.tensor(0)));
11389:   Py_RETURN_NONE;
11390:   END_HANDLE_TH_ERRORS
11391: }
11392: 
11393: \
11394: // mode
11395: static PyObject * THPVariable_mode(PyObject* self_, PyObject* args, PyObject* kwargs)
11396: {
11397:   HANDLE_TH_ERRORS
11398:   static PyTypeObject* NamedTuple = generated::get_mode_structseq();
11399:   const Tensor& self = THPVariable_Unpack(self_);
11400:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `THPVariable_min`, `get_min_structseq`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_min`, `get_min_structseq`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11401-11520

```cpp
11401:     "mode(int64_t dim=-1, bool keepdim=False)",
11402:     "mode(Dimname dim, bool keepdim=False)",
11403:   }, /*traceable=*/true);
11404: 
11405:   ParsedArgs<2> parsed_args;
11406:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11407:   if(_r.has_torch_function()) {
11408:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11409:   }
11410:   switch (_r.idx) {
11411:     case 0: {
11412:       // aten::mode(Tensor self, int dim=-1, bool keepdim=False) -> (Tensor values, Tensor indices)
11413: 
11414:       auto dispatch_mode = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11415:         pybind11::gil_scoped_release no_gil;
11416:         return self.mode(dim, keepdim);
11417:       };
11418:       return wrap(NamedTuple, dispatch_mode(self, _r.toInt64(0), _r.toBool(1)));
11419:     }
11420:     case 1: {
11421:       // aten::mode.dimname(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11422: 
11423:       auto dispatch_mode = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11424:         pybind11::gil_scoped_release no_gil;
11425:         return self.mode(dim, keepdim);
11426:       };
11427:       return wrap(NamedTuple, dispatch_mode(self, _r.dimname(0), _r.toBool(1)));
11428:     }
11429:   }
11430:   Py_RETURN_NONE;
11431:   END_HANDLE_TH_ERRORS
11432: }
11433: 
11434: \
11435: // moveaxis
11436: static PyObject * THPVariable_moveaxis(PyObject* self_, PyObject* args, PyObject* kwargs)
11437: {
11438:   HANDLE_TH_ERRORS
11439:   const Tensor& self = THPVariable_Unpack(self_);
11440:   static PythonArgParser parser({
11441:     "moveaxis(int64_t source, int64_t destination)",
11442:     "moveaxis(IntArrayRef source, IntArrayRef destination)",
11443:   }, /*traceable=*/true);
11444: 
11445:   ParsedArgs<2> parsed_args;
11446:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11447:   if(_r.has_torch_function()) {
11448:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11449:   }
11450:   switch (_r.idx) {
11451:     case 0: {
11452:       // aten::moveaxis.int(Tensor(a) self, int source, int destination) -> Tensor(a)
11453: 
11454:       auto dispatch_moveaxis = [](const at::Tensor & self, int64_t source, int64_t destination) -> at::Tensor {
11455:         pybind11::gil_scoped_release no_gil;
11456:         return self.moveaxis(source, destination);
11457:       };
11458:       return wrap(dispatch_moveaxis(self, _r.toInt64(0), _r.toInt64(1)));
11459:     }
11460:     case 1: {
11461:       // aten::moveaxis.intlist(Tensor(a) self, int[] source, int[] destination) -> Tensor(a)
11462: 
11463:       auto dispatch_moveaxis = [](const at::Tensor & self, at::IntArrayRef source, at::IntArrayRef destination) -> at::Tensor {
11464:         pybind11::gil_scoped_release no_gil;
11465:         return self.moveaxis(source, destination);
11466:       };
11467:       return wrap(dispatch_moveaxis(self, _r.intlist(0), _r.intlist(1)));
11468:     }
11469:   }
11470:   Py_RETURN_NONE;
11471:   END_HANDLE_TH_ERRORS
11472: }
11473: 
11474: \
11475: // movedim
11476: static PyObject * THPVariable_movedim(PyObject* self_, PyObject* args, PyObject* kwargs)
11477: {
11478:   HANDLE_TH_ERRORS
11479:   const Tensor& self = THPVariable_Unpack(self_);
11480:   static PythonArgParser parser({
11481:     "movedim(int64_t source, int64_t destination)",
11482:     "movedim(IntArrayRef source, IntArrayRef destination)",
11483:   }, /*traceable=*/true);
11484: 
11485:   ParsedArgs<2> parsed_args;
11486:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11487:   if(_r.has_torch_function()) {
11488:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11489:   }
11490:   switch (_r.idx) {
11491:     case 0: {
11492:       // aten::movedim.int(Tensor(a) self, int source, int destination) -> Tensor(a)
11493: 
11494:       auto dispatch_movedim = [](const at::Tensor & self, int64_t source, int64_t destination) -> at::Tensor {
11495:         pybind11::gil_scoped_release no_gil;
11496:         return self.movedim(source, destination);
11497:       };
11498:       return wrap(dispatch_movedim(self, _r.toInt64(0), _r.toInt64(1)));
11499:     }
11500:     case 1: {
11501:       // aten::movedim.intlist(Tensor(a) self, int[] source, int[] destination) -> Tensor(a)
11502: 
11503:       auto dispatch_movedim = [](const at::Tensor & self, at::IntArrayRef source, at::IntArrayRef destination) -> at::Tensor {
11504:         pybind11::gil_scoped_release no_gil;
11505:         return self.movedim(source, destination);
11506:       };
11507:       return wrap(dispatch_movedim(self, _r.intlist(0), _r.intlist(1)));
11508:     }
11509:   }
11510:   Py_RETURN_NONE;
11511:   END_HANDLE_TH_ERRORS
11512: }
11513: 
11514: // msort
11515: static PyObject * THPVariable_msort(PyObject* self_, PyObject* args)
11516: {
11517:   HANDLE_TH_ERRORS
11518:   const Tensor& self = THPVariable_Unpack(self_);
11519:   if (has_torch_function(self_)) {
11520:     return handle_torch_function(self_, "msort");
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `mode`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `mode`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11521-11640

```cpp
11521:   }
11522:   // aten::msort(Tensor self) -> Tensor
11523: 
11524:   auto dispatch_msort = [](const at::Tensor & self) -> at::Tensor {
11525:     pybind11::gil_scoped_release no_gil;
11526:     return self.msort();
11527:   };
11528:   return wrap(dispatch_msort(self));
11529:   END_HANDLE_TH_ERRORS
11530: }
11531: 
11532: // mul
11533: static PyObject * THPVariable_mul(PyObject* self_, PyObject* args, PyObject* kwargs)
11534: {
11535:   HANDLE_TH_ERRORS
11536:   const Tensor& self = THPVariable_Unpack(self_);
11537:   static PythonArgParser parser({
11538:     "mul(Tensor other)",
11539:   }, /*traceable=*/true);
11540: 
11541:   ParsedArgs<1> parsed_args;
11542:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11543:   if(_r.has_torch_function()) {
11544:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11545:   }
11546:   // aten::mul.Tensor(Tensor self, Tensor other) -> Tensor
11547: 
11548:   auto dispatch_mul = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11549:     pybind11::gil_scoped_release no_gil;
11550:     return self.mul(other);
11551:   };
11552:   return wrap(dispatch_mul(self, _r.tensor(0)));
11553:   Py_RETURN_NONE;
11554:   END_HANDLE_TH_ERRORS
11555: }
11556: 
11557: // mul_
11558: static PyObject * THPVariable_mul_(PyObject* self_, PyObject* args, PyObject* kwargs)
11559: {
11560:   HANDLE_TH_ERRORS
11561:   const Tensor& self = THPVariable_Unpack(self_);
11562:   static PythonArgParser parser({
11563:     "mul_(Tensor other)",
11564:   }, /*traceable=*/true);
11565: 
11566:   ParsedArgs<1> parsed_args;
11567:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11568:   if(_r.has_torch_function()) {
11569:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11570:   }
11571:   // aten::mul_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
11572: 
11573:   auto dispatch_mul_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11574:     pybind11::gil_scoped_release no_gil;
11575:     return self.mul_(other);
11576:   };
11577:   return wrap(dispatch_mul_(self, _r.tensor(0)));
11578:   Py_RETURN_NONE;
11579:   END_HANDLE_TH_ERRORS
11580: }
11581: 
11582: // multinomial
11583: static PyObject * THPVariable_multinomial(PyObject* self_, PyObject* args, PyObject* kwargs)
11584: {
11585:   HANDLE_TH_ERRORS
11586:   const Tensor& self = THPVariable_Unpack(self_);
11587:   static PythonArgParser parser({
11588:     "multinomial(SymInt num_samples, bool replacement=False, *, Generator? generator=None)",
11589:   }, /*traceable=*/true);
11590: 
11591:   ParsedArgs<3> parsed_args;
11592:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11593:   if(_r.has_torch_function()) {
11594:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11595:   }
11596:   // aten::multinomial(Tensor self, SymInt num_samples, bool replacement=False, *, Generator? generator=None) -> Tensor
11597: 
11598:   auto dispatch_multinomial = [](const at::Tensor & self, c10::SymInt num_samples, bool replacement, ::std::optional<at::Generator> generator) -> at::Tensor {
11599:     pybind11::gil_scoped_release no_gil;
11600:     return self.multinomial_symint(num_samples, replacement, generator);
11601:   };
11602:   return wrap(dispatch_multinomial(self, _r.toSymInt(0), _r.toBool(1), _r.generator(2)));
11603:   Py_RETURN_NONE;
11604:   END_HANDLE_TH_ERRORS
11605: }
11606: 
11607: \
11608: // multiply
11609: static PyObject * THPVariable_multiply(PyObject* self_, PyObject* args, PyObject* kwargs)
11610: {
11611:   HANDLE_TH_ERRORS
11612:   const Tensor& self = THPVariable_Unpack(self_);
11613:   static PythonArgParser parser({
11614:     "multiply(Tensor other)",
11615:     "multiply(Scalar other)",
11616:   }, /*traceable=*/true);
11617: 
11618:   ParsedArgs<1> parsed_args;
11619:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11620:   if(_r.has_torch_function()) {
11621:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11622:   }
11623:   switch (_r.idx) {
11624:     case 0: {
11625:       // aten::multiply.Tensor(Tensor self, Tensor other) -> Tensor
11626: 
11627:       auto dispatch_multiply = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11628:         pybind11::gil_scoped_release no_gil;
11629:         return self.multiply(other);
11630:       };
11631:       return wrap(dispatch_multiply(self, _r.tensor(0)));
11632:     }
11633:     case 1: {
11634:       // aten::multiply.Scalar(Tensor self, Scalar other) -> Tensor
11635: 
11636:       auto dispatch_multiply = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
11637:         pybind11::gil_scoped_release no_gil;
11638:         return self.multiply(other);
11639:       };
11640:       return wrap(dispatch_multiply(self, _r.scalar(0)));
```

- EN: The main execution path in this span is carried by `msort`, `wrap`, `THPVariable_mul`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `msort`, `wrap`, `THPVariable_mul` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11641-11760

```cpp
11641:     }
11642:   }
11643:   Py_RETURN_NONE;
11644:   END_HANDLE_TH_ERRORS
11645: }
11646: 
11647: \
11648: // multiply_
11649: static PyObject * THPVariable_multiply_(PyObject* self_, PyObject* args, PyObject* kwargs)
11650: {
11651:   HANDLE_TH_ERRORS
11652:   const Tensor& self = THPVariable_Unpack(self_);
11653:   static PythonArgParser parser({
11654:     "multiply_(Tensor other)",
11655:     "multiply_(Scalar other)",
11656:   }, /*traceable=*/true);
11657: 
11658:   ParsedArgs<1> parsed_args;
11659:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11660:   if(_r.has_torch_function()) {
11661:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11662:   }
11663:   switch (_r.idx) {
11664:     case 0: {
11665:       // aten::multiply_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
11666: 
11667:       auto dispatch_multiply_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
11668:         pybind11::gil_scoped_release no_gil;
11669:         return self.multiply_(other);
11670:       };
11671:       return wrap(dispatch_multiply_(self, _r.tensor(0)));
11672:     }
11673:     case 1: {
11674:       // aten::multiply_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
11675: 
11676:       auto dispatch_multiply_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
11677:         pybind11::gil_scoped_release no_gil;
11678:         return self.multiply_(other);
11679:       };
11680:       return wrap(dispatch_multiply_(self, _r.scalar(0)));
11681:     }
11682:   }
11683:   Py_RETURN_NONE;
11684:   END_HANDLE_TH_ERRORS
11685: }
11686: 
11687: // mv
11688: static PyObject * THPVariable_mv(PyObject* self_, PyObject* args, PyObject* kwargs)
11689: {
11690:   HANDLE_TH_ERRORS
11691:   const Tensor& self = THPVariable_Unpack(self_);
11692:   static PythonArgParser parser({
11693:     "mv(Tensor vec)",
11694:   }, /*traceable=*/true);
11695: 
11696:   ParsedArgs<1> parsed_args;
11697:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11698:   if(_r.has_torch_function()) {
11699:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11700:   }
11701:   // aten::mv(Tensor self, Tensor vec) -> Tensor
11702: 
11703:   auto dispatch_mv = [](const at::Tensor & self, const at::Tensor & vec) -> at::Tensor {
11704:     pybind11::gil_scoped_release no_gil;
11705:     return self.mv(vec);
11706:   };
11707:   return wrap(dispatch_mv(self, _r.tensor(0)));
11708:   Py_RETURN_NONE;
11709:   END_HANDLE_TH_ERRORS
11710: }
11711: 
11712: // mvlgamma
11713: static PyObject * THPVariable_mvlgamma(PyObject* self_, PyObject* args, PyObject* kwargs)
11714: {
11715:   HANDLE_TH_ERRORS
11716:   const Tensor& self = THPVariable_Unpack(self_);
11717:   static PythonArgParser parser({
11718:     "mvlgamma(int64_t p)",
11719:   }, /*traceable=*/true);
11720: 
11721:   ParsedArgs<1> parsed_args;
11722:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11723:   if(_r.has_torch_function()) {
11724:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11725:   }
11726:   // aten::mvlgamma(Tensor self, int p) -> Tensor
11727: 
11728:   auto dispatch_mvlgamma = [](const at::Tensor & self, int64_t p) -> at::Tensor {
11729:     pybind11::gil_scoped_release no_gil;
11730:     return self.mvlgamma(p);
11731:   };
11732:   return wrap(dispatch_mvlgamma(self, _r.toInt64(0)));
11733:   Py_RETURN_NONE;
11734:   END_HANDLE_TH_ERRORS
11735: }
11736: 
11737: // mvlgamma_
11738: static PyObject * THPVariable_mvlgamma_(PyObject* self_, PyObject* args, PyObject* kwargs)
11739: {
11740:   HANDLE_TH_ERRORS
11741:   const Tensor& self = THPVariable_Unpack(self_);
11742:   static PythonArgParser parser({
11743:     "mvlgamma_(int64_t p)",
11744:   }, /*traceable=*/true);
11745: 
11746:   ParsedArgs<1> parsed_args;
11747:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11748:   if(_r.has_torch_function()) {
11749:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11750:   }
11751:   // aten::mvlgamma_(Tensor(a!) self, int p) -> Tensor(a!)
11752: 
11753:   auto dispatch_mvlgamma_ = [](const at::Tensor & self, int64_t p) -> at::Tensor {
11754:     pybind11::gil_scoped_release no_gil;
11755:     return self.mvlgamma_(p);
11756:   };
11757:   return wrap(dispatch_mvlgamma_(self, _r.toInt64(0)));
11758:   Py_RETURN_NONE;
11759:   END_HANDLE_TH_ERRORS
11760: }
```

- EN: The main execution path in this span is carried by `THPVariable_multiply_`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_multiply_`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 11761-11880

```cpp
11761: 
11762: // nan_to_num
11763: static PyObject * THPVariable_nan_to_num(PyObject* self_, PyObject* args, PyObject* kwargs)
11764: {
11765:   HANDLE_TH_ERRORS
11766:   const Tensor& self = THPVariable_Unpack(self_);
11767:   static PythonArgParser parser({
11768:     "nan_to_num(double? nan=None, double? posinf=None, double? neginf=None)",
11769:   }, /*traceable=*/true);
11770: 
11771:   ParsedArgs<3> parsed_args;
11772:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11773:   if(_r.has_torch_function()) {
11774:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11775:   }
11776:   // aten::nan_to_num(Tensor self, float? nan=None, float? posinf=None, float? neginf=None) -> Tensor
11777: 
11778:   auto dispatch_nan_to_num = [](const at::Tensor & self, ::std::optional<double> nan, ::std::optional<double> posinf, ::std::optional<double> neginf) -> at::Tensor {
11779:     pybind11::gil_scoped_release no_gil;
11780:     return self.nan_to_num(nan, posinf, neginf);
11781:   };
11782:   return wrap(dispatch_nan_to_num(self, _r.toDoubleOptional(0), _r.toDoubleOptional(1), _r.toDoubleOptional(2)));
11783:   Py_RETURN_NONE;
11784:   END_HANDLE_TH_ERRORS
11785: }
11786: 
11787: // nan_to_num_
11788: static PyObject * THPVariable_nan_to_num_(PyObject* self_, PyObject* args, PyObject* kwargs)
11789: {
11790:   HANDLE_TH_ERRORS
11791:   const Tensor& self = THPVariable_Unpack(self_);
11792:   static PythonArgParser parser({
11793:     "nan_to_num_(double? nan=None, double? posinf=None, double? neginf=None)",
11794:   }, /*traceable=*/true);
11795: 
11796:   ParsedArgs<3> parsed_args;
11797:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11798:   if(_r.has_torch_function()) {
11799:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11800:   }
11801:   // aten::nan_to_num_(Tensor(a!) self, float? nan=None, float? posinf=None, float? neginf=None) -> Tensor(a!)
11802: 
11803:   auto dispatch_nan_to_num_ = [](const at::Tensor & self, ::std::optional<double> nan, ::std::optional<double> posinf, ::std::optional<double> neginf) -> at::Tensor {
11804:     pybind11::gil_scoped_release no_gil;
11805:     return self.nan_to_num_(nan, posinf, neginf);
11806:   };
11807:   return wrap(dispatch_nan_to_num_(self, _r.toDoubleOptional(0), _r.toDoubleOptional(1), _r.toDoubleOptional(2)));
11808:   Py_RETURN_NONE;
11809:   END_HANDLE_TH_ERRORS
11810: }
11811: 
11812: // nanmean
11813: static PyObject * THPVariable_nanmean(PyObject* self_, PyObject* args, PyObject* kwargs)
11814: {
11815:   HANDLE_TH_ERRORS
11816:   const Tensor& self = THPVariable_Unpack(self_);
11817:   static PythonArgParser parser({
11818:     "nanmean(IntArrayRef[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None)",
11819:   }, /*traceable=*/true);
11820: 
11821:   ParsedArgs<3> parsed_args;
11822:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11823:   if(_r.has_torch_function()) {
11824:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11825:   }
11826:   // aten::nanmean(Tensor self, int[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
11827: 
11828:   auto dispatch_nanmean = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
11829:     pybind11::gil_scoped_release no_gil;
11830:     return self.nanmean(dim, keepdim, dtype);
11831:   };
11832:   return wrap(dispatch_nanmean(self, _r.intlistOptional(0), _r.toBool(1), _r.scalartypeOptional(2)));
11833:   Py_RETURN_NONE;
11834:   END_HANDLE_TH_ERRORS
11835: }
11836: 
11837: \
11838: // nanmedian
11839: static PyObject * THPVariable_nanmedian(PyObject* self_, PyObject* args, PyObject* kwargs)
11840: {
11841:   HANDLE_TH_ERRORS
11842:   static PyTypeObject* NamedTuple = generated::get_nanmedian_structseq();
11843:   const Tensor& self = THPVariable_Unpack(self_);
11844:   static PythonArgParser parser({
11845:     "nanmedian()",
11846:     "nanmedian(int64_t dim, bool keepdim=False)",
11847:     "nanmedian(Dimname dim, bool keepdim=False)",
11848:   }, /*traceable=*/true);
11849: 
11850:   ParsedArgs<2> parsed_args;
11851:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11852:   if(_r.has_torch_function()) {
11853:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11854:   }
11855:   switch (_r.idx) {
11856:     case 0: {
11857:       // aten::nanmedian(Tensor self) -> Tensor
11858: 
11859:       auto dispatch_nanmedian = [](const at::Tensor & self) -> at::Tensor {
11860:         pybind11::gil_scoped_release no_gil;
11861:         return self.nanmedian();
11862:       };
11863:       return wrap(dispatch_nanmedian(self));
11864:     }
11865:     case 1: {
11866:       // aten::nanmedian.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11867: 
11868:       auto dispatch_nanmedian = [](const at::Tensor & self, int64_t dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11869:         pybind11::gil_scoped_release no_gil;
11870:         return self.nanmedian(dim, keepdim);
11871:       };
11872:       return wrap(NamedTuple, dispatch_nanmedian(self, _r.toInt64(0), _r.toBool(1)));
11873:     }
11874:     case 2: {
11875:       // aten::nanmedian.names_dim(Tensor self, Dimname dim, bool keepdim=False) -> (Tensor values, Tensor indices)
11876: 
11877:       auto dispatch_nanmedian = [](const at::Tensor & self, at::Dimname dim, bool keepdim) -> ::std::tuple<at::Tensor,at::Tensor> {
11878:         pybind11::gil_scoped_release no_gil;
11879:         return self.nanmedian(dim, keepdim);
11880:       };
```

- EN: The main execution path in this span is carried by `THPVariable_nan_to_num`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_nan_to_num`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 11881-12000

```cpp
11881:       return wrap(NamedTuple, dispatch_nanmedian(self, _r.dimname(0), _r.toBool(1)));
11882:     }
11883:   }
11884:   Py_RETURN_NONE;
11885:   END_HANDLE_TH_ERRORS
11886: }
11887: 
11888: \
11889: // nanquantile
11890: static PyObject * THPVariable_nanquantile(PyObject* self_, PyObject* args, PyObject* kwargs)
11891: {
11892:   HANDLE_TH_ERRORS
11893:   const Tensor& self = THPVariable_Unpack(self_);
11894:   static PythonArgParser parser({
11895:     "nanquantile(Tensor q, int64_t? dim=None, bool keepdim=False, *, c10::string_view interpolation=\"linear\")",
11896:     "nanquantile(double q, int64_t? dim=None, bool keepdim=False, *, c10::string_view interpolation=\"linear\")",
11897:   }, /*traceable=*/true);
11898: 
11899:   ParsedArgs<4> parsed_args;
11900:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11901:   if(_r.has_torch_function()) {
11902:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11903:   }
11904:   switch (_r.idx) {
11905:     case 0: {
11906:       // aten::nanquantile(Tensor self, Tensor q, int? dim=None, bool keepdim=False, *, str interpolation='linear') -> Tensor
11907: 
11908:       auto dispatch_nanquantile = [](const at::Tensor & self, const at::Tensor & q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
11909:         pybind11::gil_scoped_release no_gil;
11910:         return self.nanquantile(q, dim, keepdim, interpolation);
11911:       };
11912:       return wrap(dispatch_nanquantile(self, _r.tensor(0), _r.toInt64Optional(1), _r.toBool(2), _r.stringView(3)));
11913:     }
11914:     case 1: {
11915:       // aten::nanquantile.scalar(Tensor self, float q, int? dim=None, bool keepdim=False, *, str interpolation='linear') -> Tensor
11916: 
11917:       auto dispatch_nanquantile = [](const at::Tensor & self, double q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
11918:         pybind11::gil_scoped_release no_gil;
11919:         return self.nanquantile(q, dim, keepdim, interpolation);
11920:       };
11921:       return wrap(dispatch_nanquantile(self, _r.toDouble(0), _r.toInt64Optional(1), _r.toBool(2), _r.stringView(3)));
11922:     }
11923:   }
11924:   Py_RETURN_NONE;
11925:   END_HANDLE_TH_ERRORS
11926: }
11927: 
11928: // nansum
11929: static PyObject * THPVariable_nansum(PyObject* self_, PyObject* args, PyObject* kwargs)
11930: {
11931:   HANDLE_TH_ERRORS
11932:   const Tensor& self = THPVariable_Unpack(self_);
11933:   static PythonArgParser parser({
11934:     "nansum(IntArrayRef[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None)",
11935:   }, /*traceable=*/true);
11936: 
11937:   ParsedArgs<3> parsed_args;
11938:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11939:   if(_r.has_torch_function()) {
11940:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11941:   }
11942:   // aten::nansum(Tensor self, int[1]? dim=None, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
11943: 
11944:   auto dispatch_nansum = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
11945:     pybind11::gil_scoped_release no_gil;
11946:     return self.nansum(dim, keepdim, dtype);
11947:   };
11948:   return wrap(dispatch_nansum(self, _r.intlistOptional(0), _r.toBool(1), _r.scalartypeOptional(2)));
11949:   Py_RETURN_NONE;
11950:   END_HANDLE_TH_ERRORS
11951: }
11952: 
11953: \
11954: // narrow
11955: static PyObject * THPVariable_narrow(PyObject* self_, PyObject* args, PyObject* kwargs)
11956: {
11957:   HANDLE_TH_ERRORS
11958:   const Tensor& self = THPVariable_Unpack(self_);
11959:   static PythonArgParser parser({
11960:     "narrow(int64_t dim, Tensor start, SymInt length)",
11961:     "narrow(int64_t dim, SymInt start, SymInt length)",
11962:   }, /*traceable=*/true);
11963: 
11964:   ParsedArgs<3> parsed_args;
11965:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
11966:   if(_r.has_torch_function()) {
11967:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
11968:   }
11969:   switch (_r.idx) {
11970:     case 0: {
11971:       // aten::narrow.Tensor(Tensor(a) self, int dim, Tensor start, SymInt length) -> Tensor(a)
11972: 
11973:       auto dispatch_narrow = [](const at::Tensor & self, int64_t dim, const at::Tensor & start, c10::SymInt length) -> at::Tensor {
11974:         pybind11::gil_scoped_release no_gil;
11975:         return self.narrow_symint(dim, start, length);
11976:       };
11977:       return wrap(dispatch_narrow(self, _r.toInt64(0), _r.tensor(1), _r.toSymInt(2)));
11978:     }
11979:     case 1: {
11980:       // aten::narrow(Tensor(a) self, int dim, SymInt start, SymInt length) -> Tensor(a)
11981: 
11982:       auto dispatch_narrow = [](const at::Tensor & self, int64_t dim, c10::SymInt start, c10::SymInt length) -> at::Tensor {
11983:         pybind11::gil_scoped_release no_gil;
11984:         return self.narrow_symint(dim, start, length);
11985:       };
11986:       return wrap(dispatch_narrow(self, _r.toInt64(0), _r.toSymInt(1), _r.toSymInt(2)));
11987:     }
11988:   }
11989:   Py_RETURN_NONE;
11990:   END_HANDLE_TH_ERRORS
11991: }
11992: 
11993: // narrow_copy
11994: static PyObject * THPVariable_narrow_copy(PyObject* self_, PyObject* args, PyObject* kwargs)
11995: {
11996:   HANDLE_TH_ERRORS
11997:   const Tensor& self = THPVariable_Unpack(self_);
11998:   static PythonArgParser parser({
11999:     "narrow_copy(int64_t dim, SymInt start, SymInt length)",
12000:   }, /*traceable=*/true);
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_nanquantile`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_nanquantile`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 12001-12120

```cpp
12001: 
12002:   ParsedArgs<3> parsed_args;
12003:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12004:   if(_r.has_torch_function()) {
12005:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12006:   }
12007:   // aten::narrow_copy(Tensor self, int dim, SymInt start, SymInt length) -> Tensor
12008: 
12009:   auto dispatch_narrow_copy = [](const at::Tensor & self, int64_t dim, c10::SymInt start, c10::SymInt length) -> at::Tensor {
12010:     pybind11::gil_scoped_release no_gil;
12011:     return self.narrow_copy_symint(dim, start, length);
12012:   };
12013:   return wrap(dispatch_narrow_copy(self, _r.toInt64(0), _r.toSymInt(1), _r.toSymInt(2)));
12014:   Py_RETURN_NONE;
12015:   END_HANDLE_TH_ERRORS
12016: }
12017: 
12018: \
12019: // ne
12020: static PyObject * THPVariable_ne(PyObject* self_, PyObject* args, PyObject* kwargs)
12021: {
12022:   HANDLE_TH_ERRORS
12023:   const Tensor& self = THPVariable_Unpack(self_);
12024:   static PythonArgParser parser({
12025:     "ne(Tensor other)",
12026:     "ne(Scalar other)",
12027:   }, /*traceable=*/true);
12028: 
12029:   ParsedArgs<1> parsed_args;
12030:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12031:   if(_r.has_torch_function()) {
12032:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12033:   }
12034:   switch (_r.idx) {
12035:     case 0: {
12036:       // aten::ne.Tensor(Tensor self, Tensor other) -> Tensor
12037: 
12038:       auto dispatch_ne = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
12039:         pybind11::gil_scoped_release no_gil;
12040:         return self.ne(other);
12041:       };
12042:       return wrap(dispatch_ne(self, _r.tensor(0)));
12043:     }
12044:     case 1: {
12045:       // aten::ne.Scalar(Tensor self, Scalar other) -> Tensor
12046: 
12047:       auto dispatch_ne = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
12048:         pybind11::gil_scoped_release no_gil;
12049:         return self.ne(other);
12050:       };
12051:       return wrap(dispatch_ne(self, _r.scalar(0)));
12052:     }
12053:   }
12054:   Py_RETURN_NONE;
12055:   END_HANDLE_TH_ERRORS
12056: }
12057: 
12058: \
12059: // ne_
12060: static PyObject * THPVariable_ne_(PyObject* self_, PyObject* args, PyObject* kwargs)
12061: {
12062:   HANDLE_TH_ERRORS
12063:   const Tensor& self = THPVariable_Unpack(self_);
12064:   static PythonArgParser parser({
12065:     "ne_(Tensor other)",
12066:     "ne_(Scalar other)",
12067:   }, /*traceable=*/true);
12068: 
12069:   ParsedArgs<1> parsed_args;
12070:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12071:   if(_r.has_torch_function()) {
12072:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12073:   }
12074:   switch (_r.idx) {
12075:     case 0: {
12076:       // aten::ne_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
12077: 
12078:       auto dispatch_ne_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
12079:         pybind11::gil_scoped_release no_gil;
12080:         return self.ne_(other);
12081:       };
12082:       return wrap(dispatch_ne_(self, _r.tensor(0)));
12083:     }
12084:     case 1: {
12085:       // aten::ne_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
12086: 
12087:       auto dispatch_ne_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
12088:         pybind11::gil_scoped_release no_gil;
12089:         return self.ne_(other);
12090:       };
12091:       return wrap(dispatch_ne_(self, _r.scalar(0)));
12092:     }
12093:   }
12094:   Py_RETURN_NONE;
12095:   END_HANDLE_TH_ERRORS
12096: }
12097: 
12098: // neg
12099: static PyObject * THPVariable_neg(PyObject* self_, PyObject* args)
12100: {
12101:   HANDLE_TH_ERRORS
12102:   const Tensor& self = THPVariable_Unpack(self_);
12103:   if (has_torch_function(self_)) {
12104:     return handle_torch_function(self_, "neg");
12105:   }
12106:   // aten::neg(Tensor self) -> Tensor
12107: 
12108:   auto dispatch_neg = [](const at::Tensor & self) -> at::Tensor {
12109:     pybind11::gil_scoped_release no_gil;
12110:     return self.neg();
12111:   };
12112:   return wrap(dispatch_neg(self));
12113:   END_HANDLE_TH_ERRORS
12114: }
12115: 
12116: // neg_
12117: static PyObject * THPVariable_neg_(PyObject* self_, PyObject* args)
12118: {
12119:   HANDLE_TH_ERRORS
12120:   const Tensor& self = THPVariable_Unpack(self_);
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `narrow_copy`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `narrow_copy`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 12121-12240

```cpp
12121:   if (has_torch_function(self_)) {
12122:     return handle_torch_function(self_, "neg_");
12123:   }
12124:   // aten::neg_(Tensor(a!) self) -> Tensor(a!)
12125: 
12126:   auto dispatch_neg_ = [](const at::Tensor & self) -> at::Tensor {
12127:     pybind11::gil_scoped_release no_gil;
12128:     return self.neg_();
12129:   };
12130:   return wrap(dispatch_neg_(self));
12131:   END_HANDLE_TH_ERRORS
12132: }
12133: 
12134: // negative
12135: static PyObject * THPVariable_negative(PyObject* self_, PyObject* args)
12136: {
12137:   HANDLE_TH_ERRORS
12138:   const Tensor& self = THPVariable_Unpack(self_);
12139:   if (has_torch_function(self_)) {
12140:     return handle_torch_function(self_, "negative");
12141:   }
12142:   // aten::negative(Tensor self) -> Tensor
12143: 
12144:   auto dispatch_negative = [](const at::Tensor & self) -> at::Tensor {
12145:     pybind11::gil_scoped_release no_gil;
12146:     return self.negative();
12147:   };
12148:   return wrap(dispatch_negative(self));
12149:   END_HANDLE_TH_ERRORS
12150: }
12151: 
12152: // negative_
12153: static PyObject * THPVariable_negative_(PyObject* self_, PyObject* args)
12154: {
12155:   HANDLE_TH_ERRORS
12156:   const Tensor& self = THPVariable_Unpack(self_);
12157:   if (has_torch_function(self_)) {
12158:     return handle_torch_function(self_, "negative_");
12159:   }
12160:   // aten::negative_(Tensor(a!) self) -> Tensor(a!)
12161: 
12162:   auto dispatch_negative_ = [](const at::Tensor & self) -> at::Tensor {
12163:     pybind11::gil_scoped_release no_gil;
12164:     return self.negative_();
12165:   };
12166:   return wrap(dispatch_negative_(self));
12167:   END_HANDLE_TH_ERRORS
12168: }
12169: 
12170: // new_empty
12171: static PyObject * THPVariable_new_empty(PyObject* self_, PyObject* args, PyObject* kwargs)
12172: {
12173:   HANDLE_TH_ERRORS
12174:   const Tensor& self = THPVariable_Unpack(self_);
12175:   static PythonArgParser parser({
12176:     "new_empty(SymIntArrayRef size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
12177:   }, /*traceable=*/true);
12178: 
12179:   ParsedArgs<6> parsed_args;
12180:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12181:   if(_r.has_torch_function()) {
12182:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12183:   }
12184:   // aten::new_empty(Tensor self, SymInt[] size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
12185:   const auto options = TensorOptions()
12186:       .dtype(_r.scalartypeOptional(1))
12187:       .device(_r.deviceOptional(3))
12188:       .layout(_r.layoutOptional(2))
12189:       .requires_grad(_r.toBool(5))
12190:       .pinned_memory(_r.toBool(4));
12191:   torch::utils::maybe_initialize_device(options);
12192: 
12193:   auto dispatch_new_empty = [](const at::Tensor & self, c10::SymIntArrayRef size, at::TensorOptions options) -> at::Tensor {
12194:     pybind11::gil_scoped_release no_gil;
12195:     return self.new_empty_symint(size, options);
12196:   };
12197:   return wrap(dispatch_new_empty(self, _r.symintlist(0), options).set_requires_grad(_r.toBool(5)));
12198:   Py_RETURN_NONE;
12199:   END_HANDLE_TH_ERRORS
12200: }
12201: 
12202: // new_empty_strided
12203: static PyObject * THPVariable_new_empty_strided(PyObject* self_, PyObject* args, PyObject* kwargs)
12204: {
12205:   HANDLE_TH_ERRORS
12206:   const Tensor& self = THPVariable_Unpack(self_);
12207:   static PythonArgParser parser({
12208:     "new_empty_strided(SymIntArrayRef size, SymIntArrayRef stride, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
12209:   }, /*traceable=*/true);
12210: 
12211:   ParsedArgs<7> parsed_args;
12212:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12213:   if(_r.has_torch_function()) {
12214:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12215:   }
12216:   // aten::new_empty_strided(Tensor self, SymInt[] size, SymInt[] stride, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
12217:   const auto options = TensorOptions()
12218:       .dtype(_r.scalartypeOptional(2))
12219:       .device(_r.deviceOptional(4))
12220:       .layout(_r.layoutOptional(3))
12221:       .requires_grad(_r.toBool(6))
12222:       .pinned_memory(_r.toBool(5));
12223:   torch::utils::maybe_initialize_device(options);
12224: 
12225:   auto dispatch_new_empty_strided = [](const at::Tensor & self, c10::SymIntArrayRef size, c10::SymIntArrayRef stride, at::TensorOptions options) -> at::Tensor {
12226:     pybind11::gil_scoped_release no_gil;
12227:     return self.new_empty_strided_symint(size, stride, options);
12228:   };
12229:   return wrap(dispatch_new_empty_strided(self, _r.symintlist(0), _r.symintlist(1), options).set_requires_grad(_r.toBool(6)));
12230:   Py_RETURN_NONE;
12231:   END_HANDLE_TH_ERRORS
12232: }
12233: 
12234: // new_full
12235: static PyObject * THPVariable_new_full(PyObject* self_, PyObject* args, PyObject* kwargs)
12236: {
12237:   HANDLE_TH_ERRORS
12238:   const Tensor& self = THPVariable_Unpack(self_);
12239:   static PythonArgParser parser({
12240:     "new_full(SymIntArrayRef size, Scalar fill_value, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `neg_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `neg_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 12241-12360

```cpp
12241:   }, /*traceable=*/true);
12242: 
12243:   ParsedArgs<7> parsed_args;
12244:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12245:   if(_r.has_torch_function()) {
12246:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12247:   }
12248:   // aten::new_full(Tensor self, SymInt[] size, Scalar fill_value, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
12249:   const auto options = TensorOptions()
12250:       .dtype(_r.scalartypeOptional(2))
12251:       .device(_r.deviceOptional(4))
12252:       .layout(_r.layoutOptional(3))
12253:       .requires_grad(_r.toBool(6))
12254:       .pinned_memory(_r.toBool(5));
12255:   torch::utils::maybe_initialize_device(options);
12256: 
12257:   auto dispatch_new_full = [](const at::Tensor & self, c10::SymIntArrayRef size, const at::Scalar & fill_value, at::TensorOptions options) -> at::Tensor {
12258:     pybind11::gil_scoped_release no_gil;
12259:     return self.new_full_symint(size, fill_value, options);
12260:   };
12261:   return wrap(dispatch_new_full(self, _r.symintlist(0), _r.scalar(1), options).set_requires_grad(_r.toBool(6)));
12262:   Py_RETURN_NONE;
12263:   END_HANDLE_TH_ERRORS
12264: }
12265: 
12266: // new_ones
12267: static PyObject * THPVariable_new_ones(PyObject* self_, PyObject* args, PyObject* kwargs)
12268: {
12269:   HANDLE_TH_ERRORS
12270:   const Tensor& self = THPVariable_Unpack(self_);
12271:   static PythonArgParser parser({
12272:     "new_ones(SymIntArrayRef size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
12273:   }, /*traceable=*/true);
12274: 
12275:   ParsedArgs<6> parsed_args;
12276:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12277:   if(_r.has_torch_function()) {
12278:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12279:   }
12280:   // aten::new_ones(Tensor self, SymInt[] size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
12281:   const auto options = TensorOptions()
12282:       .dtype(_r.scalartypeOptional(1))
12283:       .device(_r.deviceOptional(3))
12284:       .layout(_r.layoutOptional(2))
12285:       .requires_grad(_r.toBool(5))
12286:       .pinned_memory(_r.toBool(4));
12287:   torch::utils::maybe_initialize_device(options);
12288: 
12289:   auto dispatch_new_ones = [](const at::Tensor & self, c10::SymIntArrayRef size, at::TensorOptions options) -> at::Tensor {
12290:     pybind11::gil_scoped_release no_gil;
12291:     return self.new_ones_symint(size, options);
12292:   };
12293:   return wrap(dispatch_new_ones(self, _r.symintlist(0), options).set_requires_grad(_r.toBool(5)));
12294:   Py_RETURN_NONE;
12295:   END_HANDLE_TH_ERRORS
12296: }
12297: 
12298: // new_zeros
12299: static PyObject * THPVariable_new_zeros(PyObject* self_, PyObject* args, PyObject* kwargs)
12300: {
12301:   HANDLE_TH_ERRORS
12302:   const Tensor& self = THPVariable_Unpack(self_);
12303:   static PythonArgParser parser({
12304:     "new_zeros(SymIntArrayRef size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=False, bool? requires_grad=False)",
12305:   }, /*traceable=*/true);
12306: 
12307:   ParsedArgs<6> parsed_args;
12308:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12309:   if(_r.has_torch_function()) {
12310:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12311:   }
12312:   // aten::new_zeros(Tensor self, SymInt[] size, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None) -> Tensor
12313:   const auto options = TensorOptions()
12314:       .dtype(_r.scalartypeOptional(1))
12315:       .device(_r.deviceOptional(3))
12316:       .layout(_r.layoutOptional(2))
12317:       .requires_grad(_r.toBool(5))
12318:       .pinned_memory(_r.toBool(4));
12319:   torch::utils::maybe_initialize_device(options);
12320: 
12321:   auto dispatch_new_zeros = [](const at::Tensor & self, c10::SymIntArrayRef size, at::TensorOptions options) -> at::Tensor {
12322:     pybind11::gil_scoped_release no_gil;
12323:     return self.new_zeros_symint(size, options);
12324:   };
12325:   return wrap(dispatch_new_zeros(self, _r.symintlist(0), options).set_requires_grad(_r.toBool(5)));
12326:   Py_RETURN_NONE;
12327:   END_HANDLE_TH_ERRORS
12328: }
12329: 
12330: // nextafter
12331: static PyObject * THPVariable_nextafter(PyObject* self_, PyObject* args, PyObject* kwargs)
12332: {
12333:   HANDLE_TH_ERRORS
12334:   const Tensor& self = THPVariable_Unpack(self_);
12335:   static PythonArgParser parser({
12336:     "nextafter(Tensor other)",
12337:   }, /*traceable=*/true);
12338: 
12339:   ParsedArgs<1> parsed_args;
12340:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12341:   if(_r.has_torch_function()) {
12342:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12343:   }
12344:   // aten::nextafter(Tensor self, Tensor other) -> Tensor
12345: 
12346:   auto dispatch_nextafter = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
12347:     pybind11::gil_scoped_release no_gil;
12348:     return self.nextafter(other);
12349:   };
12350:   return wrap(dispatch_nextafter(self, _r.tensor(0)));
12351:   Py_RETURN_NONE;
12352:   END_HANDLE_TH_ERRORS
12353: }
12354: 
12355: // nextafter_
12356: static PyObject * THPVariable_nextafter_(PyObject* self_, PyObject* args, PyObject* kwargs)
12357: {
12358:   HANDLE_TH_ERRORS
12359:   const Tensor& self = THPVariable_Unpack(self_);
12360:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `new_full`, `TensorOptions`. Control-flow branches in this range handle alternative runtime cases or iterative work. Autograd-related metadata or graph connectivity is updated here. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `new_full`, `TensorOptions` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这里会更新与自动求导相关的元数据或计算图连接关系。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。
### Lines 12361-12480

```cpp
12361:     "nextafter_(Tensor other)",
12362:   }, /*traceable=*/true);
12363: 
12364:   ParsedArgs<1> parsed_args;
12365:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12366:   if(_r.has_torch_function()) {
12367:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12368:   }
12369:   // aten::nextafter_(Tensor(a!) self, Tensor other) -> Tensor(a!)
12370: 
12371:   auto dispatch_nextafter_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
12372:     pybind11::gil_scoped_release no_gil;
12373:     return self.nextafter_(other);
12374:   };
12375:   return wrap(dispatch_nextafter_(self, _r.tensor(0)));
12376:   Py_RETURN_NONE;
12377:   END_HANDLE_TH_ERRORS
12378: }
12379: 
12380: // nonzero_static
12381: static PyObject * THPVariable_nonzero_static(PyObject* self_, PyObject* args, PyObject* kwargs)
12382: {
12383:   HANDLE_TH_ERRORS
12384:   const Tensor& self = THPVariable_Unpack(self_);
12385:   static PythonArgParser parser({
12386:     "nonzero_static(*, SymInt size, int64_t fill_value=-1)",
12387:   }, /*traceable=*/true);
12388: 
12389:   ParsedArgs<2> parsed_args;
12390:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12391:   if(_r.has_torch_function()) {
12392:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12393:   }
12394:   // aten::nonzero_static(Tensor self, *, SymInt size, int fill_value=-1) -> Tensor
12395: 
12396:   auto dispatch_nonzero_static = [](const at::Tensor & self, c10::SymInt size, int64_t fill_value) -> at::Tensor {
12397:     pybind11::gil_scoped_release no_gil;
12398:     return self.nonzero_static_symint(size, fill_value);
12399:   };
12400:   return wrap(dispatch_nonzero_static(self, _r.toSymInt(0), _r.toInt64(1)));
12401:   Py_RETURN_NONE;
12402:   END_HANDLE_TH_ERRORS
12403: }
12404: 
12405: \
12406: // norm
12407: static PyObject * THPVariable_norm(PyObject* self_, PyObject* args, PyObject* kwargs)
12408: {
12409:   HANDLE_TH_ERRORS
12410:   const Tensor& self = THPVariable_Unpack(self_);
12411:   static PythonArgParser parser({
12412:     "norm(Scalar p=2)",
12413:     "norm(Scalar? p, *, ScalarType dtype)",
12414:     "norm(Scalar? p, IntArrayRef[1] dim, bool keepdim, *, ScalarType dtype)",
12415:     "norm(Scalar? p, IntArrayRef[1] dim, bool keepdim=False)",
12416:     "norm(Scalar? p, DimnameList[1] dim, bool keepdim, *, ScalarType dtype)",
12417:     "norm(Scalar? p, DimnameList[1] dim, bool keepdim=False)",
12418:   }, /*traceable=*/true);
12419: 
12420:   ParsedArgs<4> parsed_args;
12421:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12422:   if(_r.has_torch_function()) {
12423:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12424:   }
12425:   switch (_r.idx) {
12426:     case 0: {
12427:       // aten::norm.Scalar(Tensor self, Scalar p=2) -> Tensor
12428: 
12429:       auto dispatch_norm = [](const at::Tensor & self, const at::Scalar & p) -> at::Tensor {
12430:         pybind11::gil_scoped_release no_gil;
12431:         return self.norm(p);
12432:       };
12433:       return wrap(dispatch_norm(self, _r.scalar(0)));
12434:     }
12435:     case 1: {
12436:       // aten::norm.ScalarOpt_dtype(Tensor self, Scalar? p, *, ScalarType dtype) -> Tensor
12437: 
12438:       auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::ScalarType dtype) -> at::Tensor {
12439:         pybind11::gil_scoped_release no_gil;
12440:         return self.norm(p, dtype);
12441:       };
12442:       return wrap(dispatch_norm(self, _r.scalarOptional(0), _r.scalartype(1)));
12443:     }
12444:     case 2: {
12445:       // aten::norm.ScalarOpt_dim_dtype(Tensor self, Scalar? p, int[1] dim, bool keepdim, *, ScalarType dtype) -> Tensor
12446: 
12447:       auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim, at::ScalarType dtype) -> at::Tensor {
12448:         pybind11::gil_scoped_release no_gil;
12449:         return self.norm(p, dim, keepdim, dtype);
12450:       };
12451:       return wrap(dispatch_norm(self, _r.scalarOptional(0), _r.intlist(1), _r.toBool(2), _r.scalartype(3)));
12452:     }
12453:     case 3: {
12454:       // aten::norm.ScalarOpt_dim(Tensor self, Scalar? p, int[1] dim, bool keepdim=False) -> Tensor
12455: 
12456:       auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::IntArrayRef dim, bool keepdim) -> at::Tensor {
12457:         pybind11::gil_scoped_release no_gil;
12458:         return self.norm(p, dim, keepdim);
12459:       };
12460:       return wrap(dispatch_norm(self, _r.scalarOptional(0), _r.intlist(1), _r.toBool(2)));
12461:     }
12462:     case 4: {
12463:       // aten::norm.names_ScalarOpt_dim_dtype(Tensor self, Scalar? p, Dimname[1] dim, bool keepdim, *, ScalarType dtype) -> Tensor
12464: 
12465:       auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::DimnameList dim, bool keepdim, at::ScalarType dtype) -> at::Tensor {
12466:         pybind11::gil_scoped_release no_gil;
12467:         return self.norm(p, dim, keepdim, dtype);
12468:       };
12469:       return wrap(dispatch_norm(self, _r.scalarOptional(0), _r.dimnamelist(1), _r.toBool(2), _r.scalartype(3)));
12470:     }
12471:     case 5: {
12472:       // aten::norm.names_ScalarOpt_dim(Tensor self, Scalar? p, Dimname[1] dim, bool keepdim=False) -> Tensor
12473: 
12474:       auto dispatch_norm = [](const at::Tensor & self, const ::std::optional<at::Scalar> & p, at::DimnameList dim, bool keepdim) -> at::Tensor {
12475:         pybind11::gil_scoped_release no_gil;
12476:         return self.norm(p, dim, keepdim);
12477:       };
12478:       return wrap(dispatch_norm(self, _r.scalarOptional(0), _r.dimnamelist(1), _r.toBool(2)));
12479:     }
12480:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `nextafter_`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `nextafter_`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 12481-12600

```cpp
12481:   Py_RETURN_NONE;
12482:   END_HANDLE_TH_ERRORS
12483: }
12484: 
12485: // normal_
12486: static PyObject * THPVariable_normal_(PyObject* self_, PyObject* args, PyObject* kwargs)
12487: {
12488:   HANDLE_TH_ERRORS
12489:   const Tensor& self = THPVariable_Unpack(self_);
12490:   static PythonArgParser parser({
12491:     "normal_(double mean=0, double std=1, *, Generator? generator=None)",
12492:   }, /*traceable=*/true);
12493: 
12494:   ParsedArgs<3> parsed_args;
12495:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12496:   if(_r.has_torch_function()) {
12497:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12498:   }
12499:   // aten::normal_(Tensor(a!) self, float mean=0, float std=1, *, Generator? generator=None) -> Tensor(a!)
12500: 
12501:   auto dispatch_normal_ = [](const at::Tensor & self, double mean, double std, ::std::optional<at::Generator> generator) -> at::Tensor {
12502:     pybind11::gil_scoped_release no_gil;
12503:     return self.normal_(mean, std, generator);
12504:   };
12505:   return wrap(dispatch_normal_(self, _r.toDouble(0), _r.toDouble(1), _r.generator(2)));
12506:   Py_RETURN_NONE;
12507:   END_HANDLE_TH_ERRORS
12508: }
12509: 
12510: \
12511: // not_equal
12512: static PyObject * THPVariable_not_equal(PyObject* self_, PyObject* args, PyObject* kwargs)
12513: {
12514:   HANDLE_TH_ERRORS
12515:   const Tensor& self = THPVariable_Unpack(self_);
12516:   static PythonArgParser parser({
12517:     "not_equal(Tensor other)",
12518:     "not_equal(Scalar other)",
12519:   }, /*traceable=*/true);
12520: 
12521:   ParsedArgs<1> parsed_args;
12522:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12523:   if(_r.has_torch_function()) {
12524:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12525:   }
12526:   switch (_r.idx) {
12527:     case 0: {
12528:       // aten::not_equal.Tensor(Tensor self, Tensor other) -> Tensor
12529: 
12530:       auto dispatch_not_equal = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
12531:         pybind11::gil_scoped_release no_gil;
12532:         return self.not_equal(other);
12533:       };
12534:       return wrap(dispatch_not_equal(self, _r.tensor(0)));
12535:     }
12536:     case 1: {
12537:       // aten::not_equal.Scalar(Tensor self, Scalar other) -> Tensor
12538: 
12539:       auto dispatch_not_equal = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
12540:         pybind11::gil_scoped_release no_gil;
12541:         return self.not_equal(other);
12542:       };
12543:       return wrap(dispatch_not_equal(self, _r.scalar(0)));
12544:     }
12545:   }
12546:   Py_RETURN_NONE;
12547:   END_HANDLE_TH_ERRORS
12548: }
12549: 
12550: \
12551: // not_equal_
12552: static PyObject * THPVariable_not_equal_(PyObject* self_, PyObject* args, PyObject* kwargs)
12553: {
12554:   HANDLE_TH_ERRORS
12555:   const Tensor& self = THPVariable_Unpack(self_);
12556:   static PythonArgParser parser({
12557:     "not_equal_(Tensor other)",
12558:     "not_equal_(Scalar other)",
12559:   }, /*traceable=*/true);
12560: 
12561:   ParsedArgs<1> parsed_args;
12562:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12563:   if(_r.has_torch_function()) {
12564:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12565:   }
12566:   switch (_r.idx) {
12567:     case 0: {
12568:       // aten::not_equal_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
12569: 
12570:       auto dispatch_not_equal_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
12571:         pybind11::gil_scoped_release no_gil;
12572:         return self.not_equal_(other);
12573:       };
12574:       return wrap(dispatch_not_equal_(self, _r.tensor(0)));
12575:     }
12576:     case 1: {
12577:       // aten::not_equal_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
12578: 
12579:       auto dispatch_not_equal_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
12580:         pybind11::gil_scoped_release no_gil;
12581:         return self.not_equal_(other);
12582:       };
12583:       return wrap(dispatch_not_equal_(self, _r.scalar(0)));
12584:     }
12585:   }
12586:   Py_RETURN_NONE;
12587:   END_HANDLE_TH_ERRORS
12588: }
12589: 
12590: // orgqr
12591: static PyObject * THPVariable_orgqr(PyObject* self_, PyObject* args, PyObject* kwargs)
12592: {
12593:   HANDLE_TH_ERRORS
12594:   const Tensor& self = THPVariable_Unpack(self_);
12595:   static PythonArgParser parser({
12596:     "orgqr(Tensor input2)",
12597:   }, /*traceable=*/true);
12598: 
12599:   ParsedArgs<1> parsed_args;
12600:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
```

- EN: The main execution path in this span is carried by `THPVariable_normal_`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_normal_`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 12601-12720

```cpp
12601:   if(_r.has_torch_function()) {
12602:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12603:   }
12604:   // aten::orgqr(Tensor self, Tensor input2) -> Tensor
12605: 
12606:   auto dispatch_orgqr = [](const at::Tensor & self, const at::Tensor & input2) -> at::Tensor {
12607:     pybind11::gil_scoped_release no_gil;
12608:     return self.orgqr(input2);
12609:   };
12610:   return wrap(dispatch_orgqr(self, _r.tensor(0)));
12611:   Py_RETURN_NONE;
12612:   END_HANDLE_TH_ERRORS
12613: }
12614: 
12615: // ormqr
12616: static PyObject * THPVariable_ormqr(PyObject* self_, PyObject* args, PyObject* kwargs)
12617: {
12618:   HANDLE_TH_ERRORS
12619:   const Tensor& self = THPVariable_Unpack(self_);
12620:   static PythonArgParser parser({
12621:     "ormqr(Tensor input2, Tensor input3, bool left=True, bool transpose=False)",
12622:   }, /*traceable=*/true);
12623: 
12624:   ParsedArgs<4> parsed_args;
12625:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12626:   if(_r.has_torch_function()) {
12627:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12628:   }
12629:   // aten::ormqr(Tensor self, Tensor input2, Tensor input3, bool left=True, bool transpose=False) -> Tensor
12630: 
12631:   auto dispatch_ormqr = [](const at::Tensor & self, const at::Tensor & input2, const at::Tensor & input3, bool left, bool transpose) -> at::Tensor {
12632:     pybind11::gil_scoped_release no_gil;
12633:     return self.ormqr(input2, input3, left, transpose);
12634:   };
12635:   return wrap(dispatch_ormqr(self, _r.tensor(0), _r.tensor(1), _r.toBool(2), _r.toBool(3)));
12636:   Py_RETURN_NONE;
12637:   END_HANDLE_TH_ERRORS
12638: }
12639: 
12640: // outer
12641: static PyObject * THPVariable_outer(PyObject* self_, PyObject* args, PyObject* kwargs)
12642: {
12643:   HANDLE_TH_ERRORS
12644:   const Tensor& self = THPVariable_Unpack(self_);
12645:   static PythonArgParser parser({
12646:     "outer(Tensor vec2)",
12647:   }, /*traceable=*/true);
12648: 
12649:   ParsedArgs<1> parsed_args;
12650:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12651:   if(_r.has_torch_function()) {
12652:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12653:   }
12654:   // aten::outer(Tensor self, Tensor vec2) -> Tensor
12655: 
12656:   auto dispatch_outer = [](const at::Tensor & self, const at::Tensor & vec2) -> at::Tensor {
12657:     pybind11::gil_scoped_release no_gil;
12658:     return self.outer(vec2);
12659:   };
12660:   return wrap(dispatch_outer(self, _r.tensor(0)));
12661:   Py_RETURN_NONE;
12662:   END_HANDLE_TH_ERRORS
12663: }
12664: 
12665: // permute
12666: static PyObject * THPVariable_permute(PyObject* self_, PyObject* args, PyObject* kwargs)
12667: {
12668:   HANDLE_TH_ERRORS
12669:   const Tensor& self = THPVariable_Unpack(self_);
12670:   static PythonArgParser parser({
12671:     "permute(IntArrayRef dims)",
12672:   }, /*traceable=*/true);
12673: 
12674:   ParsedArgs<1> parsed_args;
12675:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12676:   if(_r.has_torch_function()) {
12677:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12678:   }
12679:   // aten::permute(Tensor(a) self, int[] dims) -> Tensor(a)
12680: 
12681:   auto dispatch_permute = [](const at::Tensor & self, at::IntArrayRef dims) -> at::Tensor {
12682:     pybind11::gil_scoped_release no_gil;
12683:     return self.permute(dims);
12684:   };
12685:   return wrap(dispatch_permute(self, _r.intlist(0)));
12686:   Py_RETURN_NONE;
12687:   END_HANDLE_TH_ERRORS
12688: }
12689: 
12690: // pin_memory
12691: static PyObject * THPVariable_pin_memory(PyObject* self_, PyObject* args, PyObject* kwargs)
12692: {
12693:   HANDLE_TH_ERRORS
12694:   const Tensor& self = THPVariable_Unpack(self_);
12695:   static PythonArgParser parser({
12696:     "pin_memory(Device? device=None)",
12697:   }, /*traceable=*/true);
12698: 
12699:   ParsedArgs<1> parsed_args;
12700:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12701:   if(_r.has_torch_function()) {
12702:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12703:   }
12704:   // aten::pin_memory(Tensor(a) self, Device? device=None) -> Tensor(a)
12705: 
12706:   auto dispatch_pin_memory = [](const at::Tensor & self, ::std::optional<at::Device> device) -> at::Tensor {
12707:     pybind11::gil_scoped_release no_gil;
12708:     return self.pin_memory(device);
12709:   };
12710:   return wrap(dispatch_pin_memory(self, _r.deviceOptional(0)));
12711:   Py_RETURN_NONE;
12712:   END_HANDLE_TH_ERRORS
12713: }
12714: 
12715: // pinverse
12716: static PyObject * THPVariable_pinverse(PyObject* self_, PyObject* args, PyObject* kwargs)
12717: {
12718:   HANDLE_TH_ERRORS
12719:   const Tensor& self = THPVariable_Unpack(self_);
12720:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `orgqr`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `orgqr`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 12721-12840

```cpp
12721:     "pinverse(double rcond=1e-15)",
12722:   }, /*traceable=*/true);
12723: 
12724:   ParsedArgs<1> parsed_args;
12725:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12726:   if(_r.has_torch_function()) {
12727:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12728:   }
12729:   // aten::pinverse(Tensor self, float rcond=1e-15) -> Tensor
12730: 
12731:   auto dispatch_pinverse = [](const at::Tensor & self, double rcond) -> at::Tensor {
12732:     pybind11::gil_scoped_release no_gil;
12733:     return self.pinverse(rcond);
12734:   };
12735:   return wrap(dispatch_pinverse(self, _r.toDouble(0)));
12736:   Py_RETURN_NONE;
12737:   END_HANDLE_TH_ERRORS
12738: }
12739: 
12740: // polygamma
12741: static PyObject * THPVariable_polygamma(PyObject* self_, PyObject* args, PyObject* kwargs)
12742: {
12743:   HANDLE_TH_ERRORS
12744:   const Tensor& self = THPVariable_Unpack(self_);
12745:   static PythonArgParser parser({
12746:     "polygamma(int64_t n)",
12747:   }, /*traceable=*/true);
12748: 
12749:   ParsedArgs<1> parsed_args;
12750:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12751:   if(_r.has_torch_function()) {
12752:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12753:   }
12754:   // aten::polygamma(int n, Tensor self) -> Tensor
12755: 
12756:   auto dispatch_polygamma = [](int64_t n, const at::Tensor & self) -> at::Tensor {
12757:     pybind11::gil_scoped_release no_gil;
12758:     return self.polygamma(n);
12759:   };
12760:   return wrap(dispatch_polygamma(_r.toInt64(0), self));
12761:   Py_RETURN_NONE;
12762:   END_HANDLE_TH_ERRORS
12763: }
12764: 
12765: // polygamma_
12766: static PyObject * THPVariable_polygamma_(PyObject* self_, PyObject* args, PyObject* kwargs)
12767: {
12768:   HANDLE_TH_ERRORS
12769:   const Tensor& self = THPVariable_Unpack(self_);
12770:   static PythonArgParser parser({
12771:     "polygamma_(int64_t n)",
12772:   }, /*traceable=*/true);
12773: 
12774:   ParsedArgs<1> parsed_args;
12775:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12776:   if(_r.has_torch_function()) {
12777:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12778:   }
12779:   // aten::polygamma_(Tensor(a!) self, int n) -> Tensor(a!)
12780: 
12781:   auto dispatch_polygamma_ = [](const at::Tensor & self, int64_t n) -> at::Tensor {
12782:     pybind11::gil_scoped_release no_gil;
12783:     return self.polygamma_(n);
12784:   };
12785:   return wrap(dispatch_polygamma_(self, _r.toInt64(0)));
12786:   Py_RETURN_NONE;
12787:   END_HANDLE_TH_ERRORS
12788: }
12789: 
12790: // positive
12791: static PyObject * THPVariable_positive(PyObject* self_, PyObject* args)
12792: {
12793:   HANDLE_TH_ERRORS
12794:   const Tensor& self = THPVariable_Unpack(self_);
12795:   if (has_torch_function(self_)) {
12796:     return handle_torch_function(self_, "positive");
12797:   }
12798:   // aten::positive(Tensor(a) self) -> Tensor(a)
12799: 
12800:   auto dispatch_positive = [](const at::Tensor & self) -> at::Tensor {
12801:     pybind11::gil_scoped_release no_gil;
12802:     return self.positive();
12803:   };
12804:   return wrap(dispatch_positive(self));
12805:   END_HANDLE_TH_ERRORS
12806: }
12807: 
12808: \
12809: // pow
12810: static PyObject * THPVariable_pow(PyObject* self_, PyObject* args, PyObject* kwargs)
12811: {
12812:   HANDLE_TH_ERRORS
12813:   const Tensor& self = THPVariable_Unpack(self_);
12814:   static PythonArgParser parser({
12815:     "pow(Tensor exponent)",
12816:     "pow(Scalar exponent)",
12817:   }, /*traceable=*/true);
12818: 
12819:   ParsedArgs<1> parsed_args;
12820:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12821:   if(_r.has_torch_function()) {
12822:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12823:   }
12824:   switch (_r.idx) {
12825:     case 0: {
12826:       // aten::pow.Tensor_Tensor(Tensor self, Tensor exponent) -> Tensor
12827: 
12828:       auto dispatch_pow = [](const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
12829:         pybind11::gil_scoped_release no_gil;
12830:         return self.pow(exponent);
12831:       };
12832:       return wrap(dispatch_pow(self, _r.tensor(0)));
12833:     }
12834:     case 1: {
12835:       // aten::pow.Tensor_Scalar(Tensor self, Scalar exponent) -> Tensor
12836: 
12837:       auto dispatch_pow = [](const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
12838:         pybind11::gil_scoped_release no_gil;
12839:         return self.pow(exponent);
12840:       };
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `pinverse`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `pinverse`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 12841-12960

```cpp
12841:       return wrap(dispatch_pow(self, _r.scalar(0)));
12842:     }
12843:   }
12844:   Py_RETURN_NONE;
12845:   END_HANDLE_TH_ERRORS
12846: }
12847: 
12848: \
12849: // pow_
12850: static PyObject * THPVariable_pow_(PyObject* self_, PyObject* args, PyObject* kwargs)
12851: {
12852:   HANDLE_TH_ERRORS
12853:   const Tensor& self = THPVariable_Unpack(self_);
12854:   static PythonArgParser parser({
12855:     "pow_(Tensor exponent)",
12856:     "pow_(Scalar exponent)",
12857:   }, /*traceable=*/true);
12858: 
12859:   ParsedArgs<1> parsed_args;
12860:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12861:   if(_r.has_torch_function()) {
12862:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12863:   }
12864:   switch (_r.idx) {
12865:     case 0: {
12866:       // aten::pow_.Tensor(Tensor(a!) self, Tensor exponent) -> Tensor(a!)
12867: 
12868:       auto dispatch_pow_ = [](const at::Tensor & self, const at::Tensor & exponent) -> at::Tensor {
12869:         pybind11::gil_scoped_release no_gil;
12870:         return self.pow_(exponent);
12871:       };
12872:       return wrap(dispatch_pow_(self, _r.tensor(0)));
12873:     }
12874:     case 1: {
12875:       // aten::pow_.Scalar(Tensor(a!) self, Scalar exponent) -> Tensor(a!)
12876: 
12877:       auto dispatch_pow_ = [](const at::Tensor & self, const at::Scalar & exponent) -> at::Tensor {
12878:         pybind11::gil_scoped_release no_gil;
12879:         return self.pow_(exponent);
12880:       };
12881:       return wrap(dispatch_pow_(self, _r.scalar(0)));
12882:     }
12883:   }
12884:   Py_RETURN_NONE;
12885:   END_HANDLE_TH_ERRORS
12886: }
12887: 
12888: // prelu
12889: static PyObject * THPVariable_prelu(PyObject* self_, PyObject* args, PyObject* kwargs)
12890: {
12891:   HANDLE_TH_ERRORS
12892:   const Tensor& self = THPVariable_Unpack(self_);
12893:   static PythonArgParser parser({
12894:     "prelu(Tensor weight)",
12895:   }, /*traceable=*/true);
12896: 
12897:   ParsedArgs<1> parsed_args;
12898:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12899:   if(_r.has_torch_function()) {
12900:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12901:   }
12902:   // aten::prelu(Tensor self, Tensor weight) -> Tensor
12903: 
12904:   auto dispatch_prelu = [](const at::Tensor & self, const at::Tensor & weight) -> at::Tensor {
12905:     pybind11::gil_scoped_release no_gil;
12906:     return self.prelu(weight);
12907:   };
12908:   return wrap(dispatch_prelu(self, _r.tensor(0)));
12909:   Py_RETURN_NONE;
12910:   END_HANDLE_TH_ERRORS
12911: }
12912: 
12913: \
12914: // prod
12915: static PyObject * THPVariable_prod(PyObject* self_, PyObject* args, PyObject* kwargs)
12916: {
12917:   HANDLE_TH_ERRORS
12918:   const Tensor& self = THPVariable_Unpack(self_);
12919:   static PythonArgParser parser({
12920:     "prod(*, ScalarType? dtype=None)",
12921:     "prod(int64_t dim, bool keepdim=False, *, ScalarType? dtype=None)",
12922:     "prod(Dimname dim, bool keepdim=False, *, ScalarType? dtype=None)",
12923:   }, /*traceable=*/true);
12924: 
12925:   ParsedArgs<3> parsed_args;
12926:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12927:   if(_r.has_torch_function()) {
12928:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12929:   }
12930:   switch (_r.idx) {
12931:     case 0: {
12932:       // aten::prod(Tensor self, *, ScalarType? dtype=None) -> Tensor
12933: 
12934:       auto dispatch_prod = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
12935:         pybind11::gil_scoped_release no_gil;
12936:         return self.prod(dtype);
12937:       };
12938:       return wrap(dispatch_prod(self, _r.scalartypeOptional(0)));
12939:     }
12940:     case 1: {
12941:       // aten::prod.dim_int(Tensor self, int dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
12942: 
12943:       auto dispatch_prod = [](const at::Tensor & self, int64_t dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
12944:         pybind11::gil_scoped_release no_gil;
12945:         return self.prod(dim, keepdim, dtype);
12946:       };
12947:       return wrap(dispatch_prod(self, _r.toInt64(0), _r.toBool(1), _r.scalartypeOptional(2)));
12948:     }
12949:     case 2: {
12950:       // aten::prod.dim_Dimname(Tensor self, Dimname dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
12951: 
12952:       auto dispatch_prod = [](const at::Tensor & self, at::Dimname dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
12953:         pybind11::gil_scoped_release no_gil;
12954:         return self.prod(dim, keepdim, dtype);
12955:       };
12956:       return wrap(dispatch_prod(self, _r.dimname(0), _r.toBool(1), _r.scalartypeOptional(2)));
12957:     }
12958:   }
12959:   Py_RETURN_NONE;
12960:   END_HANDLE_TH_ERRORS
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_pow_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_pow_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 12961-13080

```cpp
12961: }
12962: 
12963: // put
12964: static PyObject * THPVariable_put(PyObject* self_, PyObject* args, PyObject* kwargs)
12965: {
12966:   HANDLE_TH_ERRORS
12967:   const Tensor& self = THPVariable_Unpack(self_);
12968:   static PythonArgParser parser({
12969:     "put(Tensor index, Tensor source, bool accumulate=False)",
12970:   }, /*traceable=*/true);
12971: 
12972:   ParsedArgs<3> parsed_args;
12973:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12974:   if(_r.has_torch_function()) {
12975:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
12976:   }
12977:   // aten::put(Tensor self, Tensor index, Tensor source, bool accumulate=False) -> Tensor
12978: 
12979:   auto dispatch_put = [](const at::Tensor & self, const at::Tensor & index, const at::Tensor & source, bool accumulate) -> at::Tensor {
12980:     pybind11::gil_scoped_release no_gil;
12981:     return self.put(index, source, accumulate);
12982:   };
12983:   return wrap(dispatch_put(self, _r.tensor(0), _r.tensor(1), _r.toBool(2)));
12984:   Py_RETURN_NONE;
12985:   END_HANDLE_TH_ERRORS
12986: }
12987: 
12988: // put_
12989: static PyObject * THPVariable_put_(PyObject* self_, PyObject* args, PyObject* kwargs)
12990: {
12991:   HANDLE_TH_ERRORS
12992:   const Tensor& self = THPVariable_Unpack(self_);
12993:   static PythonArgParser parser({
12994:     "put_(Tensor index, Tensor source, bool accumulate=False)",
12995:   }, /*traceable=*/true);
12996: 
12997:   ParsedArgs<3> parsed_args;
12998:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
12999:   if(_r.has_torch_function()) {
13000:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13001:   }
13002:   // aten::put_(Tensor(a!) self, Tensor index, Tensor source, bool accumulate=False) -> Tensor(a!)
13003: 
13004:   auto dispatch_put_ = [](const at::Tensor & self, const at::Tensor & index, const at::Tensor & source, bool accumulate) -> at::Tensor {
13005:     pybind11::gil_scoped_release no_gil;
13006:     return self.put_(index, source, accumulate);
13007:   };
13008:   return wrap(dispatch_put_(self, _r.tensor(0), _r.tensor(1), _r.toBool(2)));
13009:   Py_RETURN_NONE;
13010:   END_HANDLE_TH_ERRORS
13011: }
13012: 
13013: // q_per_channel_axis
13014: static PyObject * THPVariable_q_per_channel_axis(PyObject* self_, PyObject* args)
13015: {
13016:   HANDLE_TH_ERRORS
13017:   const Tensor& self = THPVariable_Unpack(self_);
13018:   if (has_torch_function(self_)) {
13019:     return handle_torch_function(self_, "q_per_channel_axis");
13020:   }
13021:   // aten::q_per_channel_axis(Tensor self) -> int
13022: 
13023:   auto dispatch_q_per_channel_axis = [](const at::Tensor & self) -> int64_t {
13024:     pybind11::gil_scoped_release no_gil;
13025:     return self.q_per_channel_axis();
13026:   };
13027:   return wrap(dispatch_q_per_channel_axis(self));
13028:   END_HANDLE_TH_ERRORS
13029: }
13030: 
13031: // q_per_channel_scales
13032: static PyObject * THPVariable_q_per_channel_scales(PyObject* self_, PyObject* args)
13033: {
13034:   HANDLE_TH_ERRORS
13035:   const Tensor& self = THPVariable_Unpack(self_);
13036:   if (has_torch_function(self_)) {
13037:     return handle_torch_function(self_, "q_per_channel_scales");
13038:   }
13039:   // aten::q_per_channel_scales(Tensor self) -> Tensor
13040: 
13041:   auto dispatch_q_per_channel_scales = [](const at::Tensor & self) -> at::Tensor {
13042:     pybind11::gil_scoped_release no_gil;
13043:     return self.q_per_channel_scales();
13044:   };
13045:   return wrap(dispatch_q_per_channel_scales(self));
13046:   END_HANDLE_TH_ERRORS
13047: }
13048: 
13049: // q_per_channel_zero_points
13050: static PyObject * THPVariable_q_per_channel_zero_points(PyObject* self_, PyObject* args)
13051: {
13052:   HANDLE_TH_ERRORS
13053:   const Tensor& self = THPVariable_Unpack(self_);
13054:   if (has_torch_function(self_)) {
13055:     return handle_torch_function(self_, "q_per_channel_zero_points");
13056:   }
13057:   // aten::q_per_channel_zero_points(Tensor self) -> Tensor
13058: 
13059:   auto dispatch_q_per_channel_zero_points = [](const at::Tensor & self) -> at::Tensor {
13060:     pybind11::gil_scoped_release no_gil;
13061:     return self.q_per_channel_zero_points();
13062:   };
13063:   return wrap(dispatch_q_per_channel_zero_points(self));
13064:   END_HANDLE_TH_ERRORS
13065: }
13066: 
13067: // q_scale
13068: static PyObject * THPVariable_q_scale(PyObject* self_, PyObject* args)
13069: {
13070:   HANDLE_TH_ERRORS
13071:   const Tensor& self = THPVariable_Unpack(self_);
13072:   if (has_torch_function(self_)) {
13073:     return handle_torch_function(self_, "q_scale");
13074:   }
13075:   // aten::q_scale(Tensor self) -> float
13076: 
13077:   auto dispatch_q_scale = [](const at::Tensor & self) -> double {
13078:     pybind11::gil_scoped_release no_gil;
13079:     return self.q_scale();
13080:   };
```

- EN: The main execution path in this span is carried by `THPVariable_put`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_put`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 13081-13200

```cpp
13081:   return wrap(dispatch_q_scale(self));
13082:   END_HANDLE_TH_ERRORS
13083: }
13084: 
13085: // q_zero_point
13086: static PyObject * THPVariable_q_zero_point(PyObject* self_, PyObject* args)
13087: {
13088:   HANDLE_TH_ERRORS
13089:   const Tensor& self = THPVariable_Unpack(self_);
13090:   if (has_torch_function(self_)) {
13091:     return handle_torch_function(self_, "q_zero_point");
13092:   }
13093:   // aten::q_zero_point(Tensor self) -> int
13094: 
13095:   auto dispatch_q_zero_point = [](const at::Tensor & self) -> int64_t {
13096:     pybind11::gil_scoped_release no_gil;
13097:     return self.q_zero_point();
13098:   };
13099:   return wrap(dispatch_q_zero_point(self));
13100:   END_HANDLE_TH_ERRORS
13101: }
13102: 
13103: // qr
13104: static PyObject * THPVariable_qr(PyObject* self_, PyObject* args, PyObject* kwargs)
13105: {
13106:   HANDLE_TH_ERRORS
13107:   static PyTypeObject* NamedTuple = generated::get_qr_structseq();
13108:   const Tensor& self = THPVariable_Unpack(self_);
13109:   static PythonArgParser parser({
13110:     "qr(bool some=True)",
13111:   }, /*traceable=*/true);
13112: 
13113:   ParsedArgs<1> parsed_args;
13114:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13115:   if(_r.has_torch_function()) {
13116:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13117:   }
13118:   // aten::qr(Tensor self, bool some=True) -> (Tensor Q, Tensor R)
13119: 
13120:   auto dispatch_qr = [](const at::Tensor & self, bool some) -> ::std::tuple<at::Tensor,at::Tensor> {
13121:     pybind11::gil_scoped_release no_gil;
13122:     return self.qr(some);
13123:   };
13124:   return wrap(NamedTuple, dispatch_qr(self, _r.toBool(0)));
13125:   Py_RETURN_NONE;
13126:   END_HANDLE_TH_ERRORS
13127: }
13128: 
13129: // qscheme
13130: static PyObject * THPVariable_qscheme(PyObject* self_, PyObject* args)
13131: {
13132:   HANDLE_TH_ERRORS
13133:   const Tensor& self = THPVariable_Unpack(self_);
13134:   if (has_torch_function(self_)) {
13135:     return handle_torch_function(self_, "qscheme");
13136:   }
13137:   // aten::qscheme(Tensor self) -> QScheme
13138: 
13139:   auto dispatch_qscheme = [](const at::Tensor & self) -> at::QScheme {
13140:     pybind11::gil_scoped_release no_gil;
13141:     return self.qscheme();
13142:   };
13143:   return wrap(dispatch_qscheme(self));
13144:   END_HANDLE_TH_ERRORS
13145: }
13146: 
13147: \
13148: // quantile
13149: static PyObject * THPVariable_quantile(PyObject* self_, PyObject* args, PyObject* kwargs)
13150: {
13151:   HANDLE_TH_ERRORS
13152:   const Tensor& self = THPVariable_Unpack(self_);
13153:   static PythonArgParser parser({
13154:     "quantile(Tensor q, int64_t? dim=None, bool keepdim=False, *, c10::string_view interpolation=\"linear\")",
13155:     "quantile(double q, int64_t? dim=None, bool keepdim=False, *, c10::string_view interpolation=\"linear\")",
13156:   }, /*traceable=*/true);
13157: 
13158:   ParsedArgs<4> parsed_args;
13159:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13160:   if(_r.has_torch_function()) {
13161:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13162:   }
13163:   switch (_r.idx) {
13164:     case 0: {
13165:       // aten::quantile(Tensor self, Tensor q, int? dim=None, bool keepdim=False, *, str interpolation='linear') -> Tensor
13166: 
13167:       auto dispatch_quantile = [](const at::Tensor & self, const at::Tensor & q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
13168:         pybind11::gil_scoped_release no_gil;
13169:         return self.quantile(q, dim, keepdim, interpolation);
13170:       };
13171:       return wrap(dispatch_quantile(self, _r.tensor(0), _r.toInt64Optional(1), _r.toBool(2), _r.stringView(3)));
13172:     }
13173:     case 1: {
13174:       // aten::quantile.scalar(Tensor self, float q, int? dim=None, bool keepdim=False, *, str interpolation='linear') -> Tensor
13175: 
13176:       auto dispatch_quantile = [](const at::Tensor & self, double q, ::std::optional<int64_t> dim, bool keepdim, c10::string_view interpolation) -> at::Tensor {
13177:         pybind11::gil_scoped_release no_gil;
13178:         return self.quantile(q, dim, keepdim, interpolation);
13179:       };
13180:       return wrap(dispatch_quantile(self, _r.toDouble(0), _r.toInt64Optional(1), _r.toBool(2), _r.stringView(3)));
13181:     }
13182:   }
13183:   Py_RETURN_NONE;
13184:   END_HANDLE_TH_ERRORS
13185: }
13186: 
13187: // rad2deg
13188: static PyObject * THPVariable_rad2deg(PyObject* self_, PyObject* args)
13189: {
13190:   HANDLE_TH_ERRORS
13191:   const Tensor& self = THPVariable_Unpack(self_);
13192:   if (has_torch_function(self_)) {
13193:     return handle_torch_function(self_, "rad2deg");
13194:   }
13195:   // aten::rad2deg(Tensor self) -> Tensor
13196: 
13197:   auto dispatch_rad2deg = [](const at::Tensor & self) -> at::Tensor {
13198:     pybind11::gil_scoped_release no_gil;
13199:     return self.rad2deg();
13200:   };
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_q_zero_point`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_q_zero_point`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 13201-13320

```cpp
13201:   return wrap(dispatch_rad2deg(self));
13202:   END_HANDLE_TH_ERRORS
13203: }
13204: 
13205: // rad2deg_
13206: static PyObject * THPVariable_rad2deg_(PyObject* self_, PyObject* args)
13207: {
13208:   HANDLE_TH_ERRORS
13209:   const Tensor& self = THPVariable_Unpack(self_);
13210:   if (has_torch_function(self_)) {
13211:     return handle_torch_function(self_, "rad2deg_");
13212:   }
13213:   // aten::rad2deg_(Tensor(a!) self) -> Tensor(a!)
13214: 
13215:   auto dispatch_rad2deg_ = [](const at::Tensor & self) -> at::Tensor {
13216:     pybind11::gil_scoped_release no_gil;
13217:     return self.rad2deg_();
13218:   };
13219:   return wrap(dispatch_rad2deg_(self));
13220:   END_HANDLE_TH_ERRORS
13221: }
13222: 
13223: \
13224: // random_
13225: static PyObject * THPVariable_random_(PyObject* self_, PyObject* args, PyObject* kwargs)
13226: {
13227:   HANDLE_TH_ERRORS
13228:   const Tensor& self = THPVariable_Unpack(self_);
13229:   static PythonArgParser parser({
13230:     "random_(*, Generator? generator=None)",
13231:     "random_(int64_t from, int64_t? to, *, Generator? generator=None)",
13232:     "random_(int64_t to, *, Generator? generator=None)",
13233:   }, /*traceable=*/true);
13234: 
13235:   ParsedArgs<3> parsed_args;
13236:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13237:   if(_r.has_torch_function()) {
13238:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13239:   }
13240:   switch (_r.idx) {
13241:     case 0: {
13242:       // aten::random_(Tensor(a!) self, *, Generator? generator=None) -> Tensor(a!)
13243: 
13244:       auto dispatch_random_ = [](const at::Tensor & self, ::std::optional<at::Generator> generator) -> at::Tensor {
13245:         pybind11::gil_scoped_release no_gil;
13246:         return self.random_(generator);
13247:       };
13248:       return wrap(dispatch_random_(self, _r.generator(0)));
13249:     }
13250:     case 1: {
13251:       // aten::random_.from(Tensor(a!) self, int from, int? to, *, Generator? generator=None) -> Tensor(a!)
13252: 
13253:       auto dispatch_random_ = [](const at::Tensor & self, int64_t from, ::std::optional<int64_t> to, ::std::optional<at::Generator> generator) -> at::Tensor {
13254:         pybind11::gil_scoped_release no_gil;
13255:         return self.random_(from, to, generator);
13256:       };
13257:       return wrap(dispatch_random_(self, _r.toInt64(0), _r.toInt64Optional(1), _r.generator(2)));
13258:     }
13259:     case 2: {
13260:       // aten::random_.to(Tensor(a!) self, int to, *, Generator? generator=None) -> Tensor(a!)
13261: 
13262:       auto dispatch_random_ = [](const at::Tensor & self, int64_t to, ::std::optional<at::Generator> generator) -> at::Tensor {
13263:         pybind11::gil_scoped_release no_gil;
13264:         return self.random_(to, generator);
13265:       };
13266:       return wrap(dispatch_random_(self, _r.toInt64(0), _r.generator(1)));
13267:     }
13268:   }
13269:   Py_RETURN_NONE;
13270:   END_HANDLE_TH_ERRORS
13271: }
13272: 
13273: // ravel
13274: static PyObject * THPVariable_ravel(PyObject* self_, PyObject* args)
13275: {
13276:   HANDLE_TH_ERRORS
13277:   const Tensor& self = THPVariable_Unpack(self_);
13278:   if (has_torch_function(self_)) {
13279:     return handle_torch_function(self_, "ravel");
13280:   }
13281:   // aten::ravel(Tensor(a) self) -> Tensor(a)
13282: 
13283:   auto dispatch_ravel = [](const at::Tensor & self) -> at::Tensor {
13284:     pybind11::gil_scoped_release no_gil;
13285:     return self.ravel();
13286:   };
13287:   return wrap(dispatch_ravel(self));
13288:   END_HANDLE_TH_ERRORS
13289: }
13290: 
13291: // reciprocal
13292: static PyObject * THPVariable_reciprocal(PyObject* self_, PyObject* args)
13293: {
13294:   HANDLE_TH_ERRORS
13295:   const Tensor& self = THPVariable_Unpack(self_);
13296:   if (has_torch_function(self_)) {
13297:     return handle_torch_function(self_, "reciprocal");
13298:   }
13299:   // aten::reciprocal(Tensor self) -> Tensor
13300: 
13301:   auto dispatch_reciprocal = [](const at::Tensor & self) -> at::Tensor {
13302:     pybind11::gil_scoped_release no_gil;
13303:     return self.reciprocal();
13304:   };
13305:   return wrap(dispatch_reciprocal(self));
13306:   END_HANDLE_TH_ERRORS
13307: }
13308: 
13309: // reciprocal_
13310: static PyObject * THPVariable_reciprocal_(PyObject* self_, PyObject* args)
13311: {
13312:   HANDLE_TH_ERRORS
13313:   const Tensor& self = THPVariable_Unpack(self_);
13314:   if (has_torch_function(self_)) {
13315:     return handle_torch_function(self_, "reciprocal_");
13316:   }
13317:   // aten::reciprocal_(Tensor(a!) self) -> Tensor(a!)
13318: 
13319:   auto dispatch_reciprocal_ = [](const at::Tensor & self) -> at::Tensor {
13320:     pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_rad2deg_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_rad2deg_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 13321-13440

```cpp
13321:     return self.reciprocal_();
13322:   };
13323:   return wrap(dispatch_reciprocal_(self));
13324:   END_HANDLE_TH_ERRORS
13325: }
13326: 
13327: // record_stream
13328: static PyObject * THPVariable_record_stream(PyObject* self_, PyObject* args, PyObject* kwargs)
13329: {
13330:   HANDLE_TH_ERRORS
13331:   const Tensor& self = THPVariable_Unpack(self_);
13332:   static PythonArgParser parser({
13333:     "record_stream(Stream s)",
13334:   }, /*traceable=*/false);
13335: 
13336:   ParsedArgs<1> parsed_args;
13337:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13338:   if(_r.has_torch_function()) {
13339:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13340:   }
13341:   // aten::record_stream(Tensor(a!) self, Stream s) -> ()
13342: 
13343:   auto dispatch_record_stream = [](const at::Tensor & self, at::Stream s) -> void {
13344:     pybind11::gil_scoped_release no_gil;
13345:     self.record_stream(s);
13346:   };
13347:   dispatch_record_stream(self, _r.stream(0));
13348:   Py_RETURN_NONE;
13349:   Py_RETURN_NONE;
13350:   END_HANDLE_TH_ERRORS
13351: }
13352: 
13353: // refine_names
13354: static PyObject * THPVariable_refine_names(PyObject* self_, PyObject* args, PyObject* kwargs)
13355: {
13356:   HANDLE_TH_ERRORS
13357:   const Tensor& self = THPVariable_Unpack(self_);
13358:   static PythonArgParser parser({
13359:     "refine_names(DimnameList names)",
13360:   }, /*traceable=*/true);
13361: 
13362:   ParsedArgs<1> parsed_args;
13363:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13364:   if(_r.has_torch_function()) {
13365:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13366:   }
13367:   // aten::refine_names(Tensor(a) self, Dimname[] names) -> Tensor(a)
13368: 
13369:   auto dispatch_refine_names = [](const at::Tensor & self, at::DimnameList names) -> at::Tensor {
13370:     pybind11::gil_scoped_release no_gil;
13371:     return self.refine_names(names);
13372:   };
13373:   return wrap(dispatch_refine_names(self, _r.dimnamelist(0)));
13374:   Py_RETURN_NONE;
13375:   END_HANDLE_TH_ERRORS
13376: }
13377: 
13378: // relu
13379: static PyObject * THPVariable_relu(PyObject* self_, PyObject* args)
13380: {
13381:   HANDLE_TH_ERRORS
13382:   const Tensor& self = THPVariable_Unpack(self_);
13383:   if (has_torch_function(self_)) {
13384:     return handle_torch_function(self_, "relu");
13385:   }
13386:   // aten::relu(Tensor self) -> Tensor
13387: 
13388:   auto dispatch_relu = [](const at::Tensor & self) -> at::Tensor {
13389:     pybind11::gil_scoped_release no_gil;
13390:     return self.relu();
13391:   };
13392:   return wrap(dispatch_relu(self));
13393:   END_HANDLE_TH_ERRORS
13394: }
13395: 
13396: // relu_
13397: static PyObject * THPVariable_relu_(PyObject* self_, PyObject* args)
13398: {
13399:   HANDLE_TH_ERRORS
13400:   const Tensor& self = THPVariable_Unpack(self_);
13401:   if (has_torch_function(self_)) {
13402:     return handle_torch_function(self_, "relu_");
13403:   }
13404:   // aten::relu_(Tensor(a!) self) -> Tensor(a!)
13405: 
13406:   auto dispatch_relu_ = [](const at::Tensor & self) -> at::Tensor {
13407:     pybind11::gil_scoped_release no_gil;
13408:     return self.relu_();
13409:   };
13410:   return wrap(dispatch_relu_(self));
13411:   END_HANDLE_TH_ERRORS
13412: }
13413: 
13414: \
13415: // remainder
13416: static PyObject * THPVariable_remainder(PyObject* self_, PyObject* args, PyObject* kwargs)
13417: {
13418:   HANDLE_TH_ERRORS
13419:   const Tensor& self = THPVariable_Unpack(self_);
13420:   static PythonArgParser parser({
13421:     "remainder(Tensor other)",
13422:     "remainder(Scalar other)",
13423:   }, /*traceable=*/true);
13424: 
13425:   ParsedArgs<1> parsed_args;
13426:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13427:   if(_r.has_torch_function()) {
13428:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13429:   }
13430:   switch (_r.idx) {
13431:     case 0: {
13432:       // aten::remainder.Tensor(Tensor self, Tensor other) -> Tensor
13433: 
13434:       auto dispatch_remainder = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
13435:         pybind11::gil_scoped_release no_gil;
13436:         return self.remainder(other);
13437:       };
13438:       return wrap(dispatch_remainder(self, _r.tensor(0)));
13439:     }
13440:     case 1: {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_record_stream`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_record_stream`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 13441-13560

```cpp
13441:       // aten::remainder.Scalar(Tensor self, Scalar other) -> Tensor
13442: 
13443:       auto dispatch_remainder = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
13444:         pybind11::gil_scoped_release no_gil;
13445:         return self.remainder(other);
13446:       };
13447:       return wrap(dispatch_remainder(self, _r.scalar(0)));
13448:     }
13449:   }
13450:   Py_RETURN_NONE;
13451:   END_HANDLE_TH_ERRORS
13452: }
13453: 
13454: \
13455: // remainder_
13456: static PyObject * THPVariable_remainder_(PyObject* self_, PyObject* args, PyObject* kwargs)
13457: {
13458:   HANDLE_TH_ERRORS
13459:   const Tensor& self = THPVariable_Unpack(self_);
13460:   static PythonArgParser parser({
13461:     "remainder_(Tensor other)",
13462:     "remainder_(Scalar other)",
13463:   }, /*traceable=*/true);
13464: 
13465:   ParsedArgs<1> parsed_args;
13466:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13467:   if(_r.has_torch_function()) {
13468:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13469:   }
13470:   switch (_r.idx) {
13471:     case 0: {
13472:       // aten::remainder_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
13473: 
13474:       auto dispatch_remainder_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
13475:         pybind11::gil_scoped_release no_gil;
13476:         return self.remainder_(other);
13477:       };
13478:       return wrap(dispatch_remainder_(self, _r.tensor(0)));
13479:     }
13480:     case 1: {
13481:       // aten::remainder_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
13482: 
13483:       auto dispatch_remainder_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
13484:         pybind11::gil_scoped_release no_gil;
13485:         return self.remainder_(other);
13486:       };
13487:       return wrap(dispatch_remainder_(self, _r.scalar(0)));
13488:     }
13489:   }
13490:   Py_RETURN_NONE;
13491:   END_HANDLE_TH_ERRORS
13492: }
13493: 
13494: // rename
13495: static PyObject * THPVariable_rename(PyObject* self_, PyObject* args, PyObject* kwargs)
13496: {
13497:   HANDLE_TH_ERRORS
13498:   const Tensor& self = THPVariable_Unpack(self_);
13499:   static PythonArgParser parser({
13500:     "rename(DimnameList? names)",
13501:   }, /*traceable=*/true);
13502: 
13503:   ParsedArgs<1> parsed_args;
13504:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13505:   if(_r.has_torch_function()) {
13506:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13507:   }
13508:   // aten::rename(Tensor(a) self, Dimname[]? names) -> Tensor(a)
13509:   auto __names = _r.toDimnameListOptional(0);
13510:   ::std::optional<DimnameList> names = __names ? ::std::make_optional(DimnameList(__names.value())) : ::std::nullopt;
13511:   auto dispatch_rename = [](const at::Tensor & self, ::std::optional<at::DimnameList> names) -> at::Tensor {
13512:     pybind11::gil_scoped_release no_gil;
13513:     return self.rename(names);
13514:   };
13515:   return wrap(dispatch_rename(self, names));
13516:   Py_RETURN_NONE;
13517:   END_HANDLE_TH_ERRORS
13518: }
13519: 
13520: // rename_
13521: static PyObject * THPVariable_rename_(PyObject* self_, PyObject* args, PyObject* kwargs)
13522: {
13523:   HANDLE_TH_ERRORS
13524:   const Tensor& self = THPVariable_Unpack(self_);
13525:   static PythonArgParser parser({
13526:     "rename_(DimnameList? names)",
13527:   }, /*traceable=*/true);
13528: 
13529:   ParsedArgs<1> parsed_args;
13530:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13531:   if(_r.has_torch_function()) {
13532:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13533:   }
13534:   // aten::rename_(Tensor(a!) self, Dimname[]? names) -> Tensor(a!)
13535:   auto __names = _r.toDimnameListOptional(0);
13536:   ::std::optional<DimnameList> names = __names ? ::std::make_optional(DimnameList(__names.value())) : ::std::nullopt;
13537:   auto dispatch_rename_ = [](const at::Tensor & self, ::std::optional<at::DimnameList> names) -> at::Tensor {
13538:     pybind11::gil_scoped_release no_gil;
13539:     return self.rename_(names);
13540:   };
13541:   return wrap(dispatch_rename_(self, names));
13542:   Py_RETURN_NONE;
13543:   END_HANDLE_TH_ERRORS
13544: }
13545: 
13546: // renorm
13547: static PyObject * THPVariable_renorm(PyObject* self_, PyObject* args, PyObject* kwargs)
13548: {
13549:   HANDLE_TH_ERRORS
13550:   const Tensor& self = THPVariable_Unpack(self_);
13551:   static PythonArgParser parser({
13552:     "renorm(Scalar p, int64_t dim, Scalar maxnorm)",
13553:   }, /*traceable=*/true);
13554: 
13555:   ParsedArgs<3> parsed_args;
13556:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13557:   if(_r.has_torch_function()) {
13558:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13559:   }
13560:   // aten::renorm(Tensor self, Scalar p, int dim, Scalar maxnorm) -> Tensor
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_remainder_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_remainder_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 13561-13680

```cpp
13561: 
13562:   auto dispatch_renorm = [](const at::Tensor & self, const at::Scalar & p, int64_t dim, const at::Scalar & maxnorm) -> at::Tensor {
13563:     pybind11::gil_scoped_release no_gil;
13564:     return self.renorm(p, dim, maxnorm);
13565:   };
13566:   return wrap(dispatch_renorm(self, _r.scalar(0), _r.toInt64(1), _r.scalar(2)));
13567:   Py_RETURN_NONE;
13568:   END_HANDLE_TH_ERRORS
13569: }
13570: 
13571: // renorm_
13572: static PyObject * THPVariable_renorm_(PyObject* self_, PyObject* args, PyObject* kwargs)
13573: {
13574:   HANDLE_TH_ERRORS
13575:   const Tensor& self = THPVariable_Unpack(self_);
13576:   static PythonArgParser parser({
13577:     "renorm_(Scalar p, int64_t dim, Scalar maxnorm)",
13578:   }, /*traceable=*/true);
13579: 
13580:   ParsedArgs<3> parsed_args;
13581:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13582:   if(_r.has_torch_function()) {
13583:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13584:   }
13585:   // aten::renorm_(Tensor(a!) self, Scalar p, int dim, Scalar maxnorm) -> Tensor(a!)
13586: 
13587:   auto dispatch_renorm_ = [](const at::Tensor & self, const at::Scalar & p, int64_t dim, const at::Scalar & maxnorm) -> at::Tensor {
13588:     pybind11::gil_scoped_release no_gil;
13589:     return self.renorm_(p, dim, maxnorm);
13590:   };
13591:   return wrap(dispatch_renorm_(self, _r.scalar(0), _r.toInt64(1), _r.scalar(2)));
13592:   Py_RETURN_NONE;
13593:   END_HANDLE_TH_ERRORS
13594: }
13595: 
13596: // repeat
13597: static PyObject * THPVariable_repeat(PyObject* self_, PyObject* args, PyObject* kwargs)
13598: {
13599:   HANDLE_TH_ERRORS
13600:   const Tensor& self = THPVariable_Unpack(self_);
13601:   static PythonArgParser parser({
13602:     "repeat(SymIntArrayRef repeats)",
13603:   }, /*traceable=*/true);
13604: 
13605:   ParsedArgs<1> parsed_args;
13606:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13607:   if(_r.has_torch_function()) {
13608:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13609:   }
13610:   // aten::repeat(Tensor self, SymInt[] repeats) -> Tensor
13611: 
13612:   auto dispatch_repeat = [](const at::Tensor & self, c10::SymIntArrayRef repeats) -> at::Tensor {
13613:     pybind11::gil_scoped_release no_gil;
13614:     return self.repeat_symint(repeats);
13615:   };
13616:   return wrap(dispatch_repeat(self, _r.symintlist(0)));
13617:   Py_RETURN_NONE;
13618:   END_HANDLE_TH_ERRORS
13619: }
13620: 
13621: \
13622: // repeat_interleave
13623: static PyObject * THPVariable_repeat_interleave(PyObject* self_, PyObject* args, PyObject* kwargs)
13624: {
13625:   HANDLE_TH_ERRORS
13626:   const Tensor& self = THPVariable_Unpack(self_);
13627:   static PythonArgParser parser({
13628:     "repeat_interleave(Tensor repeats, int64_t? dim=None, *, SymInt? output_size=None)",
13629:     "repeat_interleave(SymInt repeats, int64_t? dim=None, *, SymInt? output_size=None)",
13630:   }, /*traceable=*/true);
13631: 
13632:   ParsedArgs<3> parsed_args;
13633:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13634:   if(_r.has_torch_function()) {
13635:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13636:   }
13637:   switch (_r.idx) {
13638:     case 0: {
13639:       // aten::repeat_interleave.self_Tensor(Tensor self, Tensor repeats, int? dim=None, *, SymInt? output_size=None) -> Tensor
13640: 
13641:       auto dispatch_repeat_interleave = [](const at::Tensor & self, const at::Tensor & repeats, ::std::optional<int64_t> dim, ::std::optional<c10::SymInt> output_size) -> at::Tensor {
13642:         pybind11::gil_scoped_release no_gil;
13643:         return self.repeat_interleave_symint(repeats, dim, output_size);
13644:       };
13645:       return wrap(dispatch_repeat_interleave(self, _r.tensor(0), _r.toInt64Optional(1), _r.toSymIntOptional(2)));
13646:     }
13647:     case 1: {
13648:       // aten::repeat_interleave.self_int(Tensor self, SymInt repeats, int? dim=None, *, SymInt? output_size=None) -> Tensor
13649: 
13650:       auto dispatch_repeat_interleave = [](const at::Tensor & self, c10::SymInt repeats, ::std::optional<int64_t> dim, ::std::optional<c10::SymInt> output_size) -> at::Tensor {
13651:         pybind11::gil_scoped_release no_gil;
13652:         return self.repeat_interleave_symint(repeats, dim, output_size);
13653:       };
13654:       return wrap(dispatch_repeat_interleave(self, _r.toSymInt(0), _r.toInt64Optional(1), _r.toSymIntOptional(2)));
13655:     }
13656:   }
13657:   Py_RETURN_NONE;
13658:   END_HANDLE_TH_ERRORS
13659: }
13660: 
13661: // reshape
13662: static PyObject * THPVariable_reshape(PyObject* self_, PyObject* args, PyObject* kwargs)
13663: {
13664:   HANDLE_TH_ERRORS
13665:   const Tensor& self = THPVariable_Unpack(self_);
13666:   static PythonArgParser parser({
13667:     "reshape(SymIntArrayRef shape)",
13668:   }, /*traceable=*/true);
13669: 
13670:   ParsedArgs<1> parsed_args;
13671:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13672:   if(_r.has_torch_function()) {
13673:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13674:   }
13675:   // aten::reshape(Tensor(a) self, SymInt[] shape) -> Tensor(a)
13676: 
13677:   auto dispatch_reshape = [](const at::Tensor & self, c10::SymIntArrayRef shape) -> at::Tensor {
13678:     pybind11::gil_scoped_release no_gil;
13679:     return self.reshape_symint(shape);
13680:   };
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_renorm_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_renorm_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 13681-13800

```cpp
13681:   return wrap(dispatch_reshape(self, _r.symintlist(0)));
13682:   Py_RETURN_NONE;
13683:   END_HANDLE_TH_ERRORS
13684: }
13685: 
13686: // reshape_as
13687: static PyObject * THPVariable_reshape_as(PyObject* self_, PyObject* args, PyObject* kwargs)
13688: {
13689:   HANDLE_TH_ERRORS
13690:   const Tensor& self = THPVariable_Unpack(self_);
13691:   static PythonArgParser parser({
13692:     "reshape_as(Tensor other)",
13693:   }, /*traceable=*/true);
13694: 
13695:   ParsedArgs<1> parsed_args;
13696:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13697:   if(_r.has_torch_function()) {
13698:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13699:   }
13700:   // aten::reshape_as(Tensor(a) self, Tensor other) -> Tensor(a)
13701: 
13702:   auto dispatch_reshape_as = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
13703:     pybind11::gil_scoped_release no_gil;
13704:     return self.reshape_as(other);
13705:   };
13706:   return wrap(dispatch_reshape_as(self, _r.tensor(0)));
13707:   Py_RETURN_NONE;
13708:   END_HANDLE_TH_ERRORS
13709: }
13710: 
13711: // resize_
13712: static PyObject * THPVariable_resize_(PyObject* self_, PyObject* args, PyObject* kwargs)
13713: {
13714:   HANDLE_TH_ERRORS
13715:   const Tensor& self = THPVariable_Unpack(self_);
13716:   static PythonArgParser parser({
13717:     "resize_(SymIntArrayRef size, *, MemoryFormat? memory_format=None)",
13718:   }, /*traceable=*/true);
13719: 
13720:   ParsedArgs<2> parsed_args;
13721:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13722:   if(_r.has_torch_function()) {
13723:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13724:   }
13725:   // aten::resize_(Tensor(a!) self, SymInt[] size, *, MemoryFormat? memory_format=None) -> Tensor(a!)
13726: 
13727:   auto dispatch_resize_ = [](const at::Tensor & self, c10::SymIntArrayRef size, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
13728:     pybind11::gil_scoped_release no_gil;
13729:     return self.resize__symint(size, memory_format);
13730:   };
13731:   return wrap(dispatch_resize_(self, _r.symintlist(0), _r.memoryformatOptional(1)));
13732:   Py_RETURN_NONE;
13733:   END_HANDLE_TH_ERRORS
13734: }
13735: 
13736: // resize_as_
13737: static PyObject * THPVariable_resize_as_(PyObject* self_, PyObject* args, PyObject* kwargs)
13738: {
13739:   HANDLE_TH_ERRORS
13740:   const Tensor& self = THPVariable_Unpack(self_);
13741:   static PythonArgParser parser({
13742:     "resize_as_(Tensor the_template, *, MemoryFormat? memory_format=None)",
13743:   }, /*traceable=*/true);
13744: 
13745:   ParsedArgs<2> parsed_args;
13746:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13747:   if(_r.has_torch_function()) {
13748:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13749:   }
13750:   // aten::resize_as_(Tensor(a!) self, Tensor the_template, *, MemoryFormat? memory_format=None) -> Tensor(a!)
13751: 
13752:   auto dispatch_resize_as_ = [](const at::Tensor & self, const at::Tensor & the_template, ::std::optional<at::MemoryFormat> memory_format) -> at::Tensor {
13753:     pybind11::gil_scoped_release no_gil;
13754:     return self.resize_as_(the_template, memory_format);
13755:   };
13756:   return wrap(dispatch_resize_as_(self, _r.tensor(0), _r.memoryformatOptional(1)));
13757:   Py_RETURN_NONE;
13758:   END_HANDLE_TH_ERRORS
13759: }
13760: 
13761: // resize_as_sparse_
13762: static PyObject * THPVariable_resize_as_sparse_(PyObject* self_, PyObject* args, PyObject* kwargs)
13763: {
13764:   HANDLE_TH_ERRORS
13765:   const Tensor& self = THPVariable_Unpack(self_);
13766:   static PythonArgParser parser({
13767:     "resize_as_sparse_(Tensor the_template)",
13768:   }, /*traceable=*/true);
13769: 
13770:   ParsedArgs<1> parsed_args;
13771:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13772:   if(_r.has_torch_function()) {
13773:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13774:   }
13775:   // aten::resize_as_sparse_(Tensor(a!) self, Tensor the_template) -> Tensor(a!)
13776: 
13777:   auto dispatch_resize_as_sparse_ = [](const at::Tensor & self, const at::Tensor & the_template) -> at::Tensor {
13778:     pybind11::gil_scoped_release no_gil;
13779:     return self.resize_as_sparse_(the_template);
13780:   };
13781:   return wrap(dispatch_resize_as_sparse_(self, _r.tensor(0)));
13782:   Py_RETURN_NONE;
13783:   END_HANDLE_TH_ERRORS
13784: }
13785: 
13786: // resolve_conj
13787: static PyObject * THPVariable_resolve_conj(PyObject* self_, PyObject* args)
13788: {
13789:   HANDLE_TH_ERRORS
13790:   const Tensor& self = THPVariable_Unpack(self_);
13791:   if (has_torch_function(self_)) {
13792:     return handle_torch_function(self_, "resolve_conj");
13793:   }
13794:   // aten::resolve_conj(Tensor(a) self) -> Tensor(a)
13795: 
13796:   auto dispatch_resolve_conj = [](const at::Tensor & self) -> at::Tensor {
13797:     pybind11::gil_scoped_release no_gil;
13798:     return self.resolve_conj();
13799:   };
13800:   return wrap(dispatch_resolve_conj(self));
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_reshape_as`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_reshape_as`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 13801-13920

```cpp
13801:   END_HANDLE_TH_ERRORS
13802: }
13803: 
13804: // resolve_neg
13805: static PyObject * THPVariable_resolve_neg(PyObject* self_, PyObject* args)
13806: {
13807:   HANDLE_TH_ERRORS
13808:   const Tensor& self = THPVariable_Unpack(self_);
13809:   if (has_torch_function(self_)) {
13810:     return handle_torch_function(self_, "resolve_neg");
13811:   }
13812:   // aten::resolve_neg(Tensor(a) self) -> Tensor(a)
13813: 
13814:   auto dispatch_resolve_neg = [](const at::Tensor & self) -> at::Tensor {
13815:     pybind11::gil_scoped_release no_gil;
13816:     return self.resolve_neg();
13817:   };
13818:   return wrap(dispatch_resolve_neg(self));
13819:   END_HANDLE_TH_ERRORS
13820: }
13821: 
13822: // retain_grad
13823: static PyObject * THPVariable_retain_grad(PyObject* self_, PyObject* args)
13824: {
13825:   HANDLE_TH_ERRORS
13826:   const Tensor& self = THPVariable_Unpack(self_);
13827:   if (has_torch_function(self_)) {
13828:     return handle_torch_function(self_, "retain_grad");
13829:   }
13830:   // aten::retain_grad(Tensor(a!) self) -> ()
13831: 
13832:   auto dispatch_retain_grad = [](const at::Tensor & self) -> void {
13833:     pybind11::gil_scoped_release no_gil;
13834:     self.retain_grad();
13835:   };
13836:   dispatch_retain_grad(self);
13837:   Py_RETURN_NONE;
13838:   END_HANDLE_TH_ERRORS
13839: }
13840: 
13841: // roll
13842: static PyObject * THPVariable_roll(PyObject* self_, PyObject* args, PyObject* kwargs)
13843: {
13844:   HANDLE_TH_ERRORS
13845:   const Tensor& self = THPVariable_Unpack(self_);
13846:   static PythonArgParser parser({
13847:     "roll(SymIntArrayRef[1] shifts, IntArrayRef[1] dims=None)",
13848:   }, /*traceable=*/true);
13849: 
13850:   ParsedArgs<2> parsed_args;
13851:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13852:   if(_r.has_torch_function()) {
13853:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13854:   }
13855:   // aten::roll(Tensor self, SymInt[1] shifts, int[1] dims=[]) -> Tensor
13856: 
13857:   auto dispatch_roll = [](const at::Tensor & self, c10::SymIntArrayRef shifts, at::IntArrayRef dims) -> at::Tensor {
13858:     pybind11::gil_scoped_release no_gil;
13859:     return self.roll_symint(shifts, dims);
13860:   };
13861:   return wrap(dispatch_roll(self, _r.symintlist(0), _r.intlist(1)));
13862:   Py_RETURN_NONE;
13863:   END_HANDLE_TH_ERRORS
13864: }
13865: 
13866: // rot90
13867: static PyObject * THPVariable_rot90(PyObject* self_, PyObject* args, PyObject* kwargs)
13868: {
13869:   HANDLE_TH_ERRORS
13870:   const Tensor& self = THPVariable_Unpack(self_);
13871:   static PythonArgParser parser({
13872:     "rot90(int64_t k=1, IntArrayRef dims={0,1})",
13873:   }, /*traceable=*/true);
13874: 
13875:   ParsedArgs<2> parsed_args;
13876:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13877:   if(_r.has_torch_function()) {
13878:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13879:   }
13880:   // aten::rot90(Tensor self, int k=1, int[] dims=[0,1]) -> Tensor
13881: 
13882:   auto dispatch_rot90 = [](const at::Tensor & self, int64_t k, at::IntArrayRef dims) -> at::Tensor {
13883:     pybind11::gil_scoped_release no_gil;
13884:     return self.rot90(k, dims);
13885:   };
13886:   return wrap(dispatch_rot90(self, _r.toInt64(0), _r.intlist(1)));
13887:   Py_RETURN_NONE;
13888:   END_HANDLE_TH_ERRORS
13889: }
13890: 
13891: \
13892: // round
13893: static PyObject * THPVariable_round(PyObject* self_, PyObject* args, PyObject* kwargs)
13894: {
13895:   HANDLE_TH_ERRORS
13896:   const Tensor& self = THPVariable_Unpack(self_);
13897:   static PythonArgParser parser({
13898:     "round()",
13899:     "round(*, int64_t decimals)",
13900:   }, /*traceable=*/true);
13901: 
13902:   ParsedArgs<1> parsed_args;
13903:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13904:   if(_r.has_torch_function()) {
13905:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13906:   }
13907:   switch (_r.idx) {
13908:     case 0: {
13909:       // aten::round(Tensor self) -> Tensor
13910: 
13911:       auto dispatch_round = [](const at::Tensor & self) -> at::Tensor {
13912:         pybind11::gil_scoped_release no_gil;
13913:         return self.round();
13914:       };
13915:       return wrap(dispatch_round(self));
13916:     }
13917:     case 1: {
13918:       // aten::round.decimals(Tensor self, *, int decimals) -> Tensor
13919: 
13920:       auto dispatch_round = [](const at::Tensor & self, int64_t decimals) -> at::Tensor {
```

- EN: The main execution path in this span is carried by `THPVariable_resolve_neg`, `THPVariable_Unpack`, `handle_torch_function`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_resolve_neg`, `THPVariable_Unpack`, `handle_torch_function` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 13921-14040

```cpp
13921:         pybind11::gil_scoped_release no_gil;
13922:         return self.round(decimals);
13923:       };
13924:       return wrap(dispatch_round(self, _r.toInt64(0)));
13925:     }
13926:   }
13927:   Py_RETURN_NONE;
13928:   END_HANDLE_TH_ERRORS
13929: }
13930: 
13931: \
13932: // round_
13933: static PyObject * THPVariable_round_(PyObject* self_, PyObject* args, PyObject* kwargs)
13934: {
13935:   HANDLE_TH_ERRORS
13936:   const Tensor& self = THPVariable_Unpack(self_);
13937:   static PythonArgParser parser({
13938:     "round_()",
13939:     "round_(*, int64_t decimals)",
13940:   }, /*traceable=*/true);
13941: 
13942:   ParsedArgs<1> parsed_args;
13943:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
13944:   if(_r.has_torch_function()) {
13945:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
13946:   }
13947:   switch (_r.idx) {
13948:     case 0: {
13949:       // aten::round_(Tensor(a!) self) -> Tensor(a!)
13950: 
13951:       auto dispatch_round_ = [](const at::Tensor & self) -> at::Tensor {
13952:         pybind11::gil_scoped_release no_gil;
13953:         return self.round_();
13954:       };
13955:       return wrap(dispatch_round_(self));
13956:     }
13957:     case 1: {
13958:       // aten::round_.decimals(Tensor(a!) self, *, int decimals) -> Tensor(a!)
13959: 
13960:       auto dispatch_round_ = [](const at::Tensor & self, int64_t decimals) -> at::Tensor {
13961:         pybind11::gil_scoped_release no_gil;
13962:         return self.round_(decimals);
13963:       };
13964:       return wrap(dispatch_round_(self, _r.toInt64(0)));
13965:     }
13966:   }
13967:   Py_RETURN_NONE;
13968:   END_HANDLE_TH_ERRORS
13969: }
13970: 
13971: // row_indices
13972: static PyObject * THPVariable_row_indices(PyObject* self_, PyObject* args)
13973: {
13974:   HANDLE_TH_ERRORS
13975:   const Tensor& self = THPVariable_Unpack(self_);
13976:   if (has_torch_function(self_)) {
13977:     return handle_torch_function(self_, "row_indices");
13978:   }
13979:   // aten::row_indices(Tensor(a) self) -> Tensor(a)
13980: 
13981:   auto dispatch_row_indices = [](const at::Tensor & self) -> at::Tensor {
13982:     pybind11::gil_scoped_release no_gil;
13983:     return self.row_indices();
13984:   };
13985:   return wrap(dispatch_row_indices(self));
13986:   END_HANDLE_TH_ERRORS
13987: }
13988: 
13989: // rsqrt
13990: static PyObject * THPVariable_rsqrt(PyObject* self_, PyObject* args)
13991: {
13992:   HANDLE_TH_ERRORS
13993:   const Tensor& self = THPVariable_Unpack(self_);
13994:   if (has_torch_function(self_)) {
13995:     return handle_torch_function(self_, "rsqrt");
13996:   }
13997:   // aten::rsqrt(Tensor self) -> Tensor
13998: 
13999:   auto dispatch_rsqrt = [](const at::Tensor & self) -> at::Tensor {
14000:     pybind11::gil_scoped_release no_gil;
14001:     return self.rsqrt();
14002:   };
14003:   return wrap(dispatch_rsqrt(self));
14004:   END_HANDLE_TH_ERRORS
14005: }
14006: 
14007: // rsqrt_
14008: static PyObject * THPVariable_rsqrt_(PyObject* self_, PyObject* args)
14009: {
14010:   HANDLE_TH_ERRORS
14011:   const Tensor& self = THPVariable_Unpack(self_);
14012:   if (has_torch_function(self_)) {
14013:     return handle_torch_function(self_, "rsqrt_");
14014:   }
14015:   // aten::rsqrt_(Tensor(a!) self) -> Tensor(a!)
14016: 
14017:   auto dispatch_rsqrt_ = [](const at::Tensor & self) -> at::Tensor {
14018:     pybind11::gil_scoped_release no_gil;
14019:     return self.rsqrt_();
14020:   };
14021:   return wrap(dispatch_rsqrt_(self));
14022:   END_HANDLE_TH_ERRORS
14023: }
14024: 
14025: \
14026: // scatter
14027: static PyObject * THPVariable_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
14028: {
14029:   HANDLE_TH_ERRORS
14030:   const Tensor& self = THPVariable_Unpack(self_);
14031:   static PythonArgParser parser({
14032:     "scatter(int64_t dim, Tensor index, Tensor src)",
14033:     "scatter(int64_t dim, Tensor index, Tensor src, *, c10::string_view reduce)",
14034:     "scatter(Dimname dim, Tensor index, Tensor src)",
14035:     "scatter(int64_t dim, Tensor index, Scalar value)",
14036:     "scatter(int64_t dim, Tensor index, Scalar value, *, c10::string_view reduce)",
14037:     "scatter(Dimname dim, Tensor index, Scalar value)",
14038:   }, /*traceable=*/true);
14039: 
14040:   ParsedArgs<4> parsed_args;
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_round_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_round_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14041-14160

```cpp
14041:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14042:   if(_r.has_torch_function()) {
14043:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14044:   }
14045:   switch (_r.idx) {
14046:     case 0: {
14047:       // aten::scatter.src(Tensor self, int dim, Tensor index, Tensor src) -> Tensor
14048: 
14049:       auto dispatch_scatter = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
14050:         pybind11::gil_scoped_release no_gil;
14051:         return self.scatter(dim, index, src);
14052:       };
14053:       return wrap(dispatch_scatter(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
14054:     }
14055:     case 1: {
14056:       // aten::scatter.reduce(Tensor self, int dim, Tensor index, Tensor src, *, str reduce) -> Tensor
14057: 
14058:       auto dispatch_scatter = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src, c10::string_view reduce) -> at::Tensor {
14059:         pybind11::gil_scoped_release no_gil;
14060:         return self.scatter(dim, index, src, reduce);
14061:       };
14062:       return wrap(dispatch_scatter(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.stringView(3)));
14063:     }
14064:     case 2: {
14065:       // aten::scatter.dimname_src(Tensor self, Dimname dim, Tensor index, Tensor src) -> Tensor
14066: 
14067:       auto dispatch_scatter = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
14068:         pybind11::gil_scoped_release no_gil;
14069:         return self.scatter(dim, index, src);
14070:       };
14071:       return wrap(dispatch_scatter(self, _r.dimname(0), _r.tensor(1), _r.tensor(2)));
14072:     }
14073:     case 3: {
14074:       // aten::scatter.value(Tensor self, int dim, Tensor index, Scalar value) -> Tensor
14075: 
14076:       auto dispatch_scatter = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
14077:         pybind11::gil_scoped_release no_gil;
14078:         return self.scatter(dim, index, value);
14079:       };
14080:       return wrap(dispatch_scatter(self, _r.toInt64(0), _r.tensor(1), _r.scalar(2)));
14081:     }
14082:     case 4: {
14083:       // aten::scatter.value_reduce(Tensor self, int dim, Tensor index, Scalar value, *, str reduce) -> Tensor
14084: 
14085:       auto dispatch_scatter = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Scalar & value, c10::string_view reduce) -> at::Tensor {
14086:         pybind11::gil_scoped_release no_gil;
14087:         return self.scatter(dim, index, value, reduce);
14088:       };
14089:       return wrap(dispatch_scatter(self, _r.toInt64(0), _r.tensor(1), _r.scalar(2), _r.stringView(3)));
14090:     }
14091:     case 5: {
14092:       // aten::scatter.dimname_value(Tensor self, Dimname dim, Tensor index, Scalar value) -> Tensor
14093: 
14094:       auto dispatch_scatter = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
14095:         pybind11::gil_scoped_release no_gil;
14096:         return self.scatter(dim, index, value);
14097:       };
14098:       return wrap(dispatch_scatter(self, _r.dimname(0), _r.tensor(1), _r.scalar(2)));
14099:     }
14100:   }
14101:   Py_RETURN_NONE;
14102:   END_HANDLE_TH_ERRORS
14103: }
14104: 
14105: \
14106: // scatter_
14107: static PyObject * THPVariable_scatter_(PyObject* self_, PyObject* args, PyObject* kwargs)
14108: {
14109:   HANDLE_TH_ERRORS
14110:   const Tensor& self = THPVariable_Unpack(self_);
14111:   static PythonArgParser parser({
14112:     "scatter_(int64_t dim, Tensor index, Tensor src)",
14113:     "scatter_(int64_t dim, Tensor index, Tensor src, *, c10::string_view reduce)",
14114:     "scatter_(int64_t dim, Tensor index, Scalar value)",
14115:     "scatter_(int64_t dim, Tensor index, Scalar value, *, c10::string_view reduce)",
14116:   }, /*traceable=*/true);
14117: 
14118:   ParsedArgs<4> parsed_args;
14119:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14120:   if(_r.has_torch_function()) {
14121:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14122:   }
14123:   switch (_r.idx) {
14124:     case 0: {
14125:       // aten::scatter_.src(Tensor(a!) self, int dim, Tensor index, Tensor src) -> Tensor(a!)
14126: 
14127:       auto dispatch_scatter_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
14128:         pybind11::gil_scoped_release no_gil;
14129:         return self.scatter_(dim, index, src);
14130:       };
14131:       return wrap(dispatch_scatter_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
14132:     }
14133:     case 1: {
14134:       // aten::scatter_.reduce(Tensor(a!) self, int dim, Tensor index, Tensor src, *, str reduce) -> Tensor(a!)
14135: 
14136:       auto dispatch_scatter_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src, c10::string_view reduce) -> at::Tensor {
14137:         pybind11::gil_scoped_release no_gil;
14138:         return self.scatter_(dim, index, src, reduce);
14139:       };
14140:       return wrap(dispatch_scatter_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.stringView(3)));
14141:     }
14142:     case 2: {
14143:       // aten::scatter_.value(Tensor(a!) self, int dim, Tensor index, Scalar value) -> Tensor(a!)
14144: 
14145:       auto dispatch_scatter_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Scalar & value) -> at::Tensor {
14146:         pybind11::gil_scoped_release no_gil;
14147:         return self.scatter_(dim, index, value);
14148:       };
14149:       return wrap(dispatch_scatter_(self, _r.toInt64(0), _r.tensor(1), _r.scalar(2)));
14150:     }
14151:     case 3: {
14152:       // aten::scatter_.value_reduce(Tensor(a!) self, int dim, Tensor index, Scalar value, *, str reduce) -> Tensor(a!)
14153: 
14154:       auto dispatch_scatter_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Scalar & value, c10::string_view reduce) -> at::Tensor {
14155:         pybind11::gil_scoped_release no_gil;
14156:         return self.scatter_(dim, index, value, reduce);
14157:       };
14158:       return wrap(dispatch_scatter_(self, _r.toInt64(0), _r.tensor(1), _r.scalar(2), _r.stringView(3)));
14159:     }
14160:   }
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable_scatter_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable_scatter_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14161-14280

```cpp
14161:   Py_RETURN_NONE;
14162:   END_HANDLE_TH_ERRORS
14163: }
14164: 
14165: \
14166: // scatter_add
14167: static PyObject * THPVariable_scatter_add(PyObject* self_, PyObject* args, PyObject* kwargs)
14168: {
14169:   HANDLE_TH_ERRORS
14170:   const Tensor& self = THPVariable_Unpack(self_);
14171:   static PythonArgParser parser({
14172:     "scatter_add(int64_t dim, Tensor index, Tensor src)",
14173:     "scatter_add(Dimname dim, Tensor index, Tensor src)",
14174:   }, /*traceable=*/true);
14175: 
14176:   ParsedArgs<3> parsed_args;
14177:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14178:   if(_r.has_torch_function()) {
14179:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14180:   }
14181:   switch (_r.idx) {
14182:     case 0: {
14183:       // aten::scatter_add(Tensor self, int dim, Tensor index, Tensor src) -> Tensor
14184: 
14185:       auto dispatch_scatter_add = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
14186:         pybind11::gil_scoped_release no_gil;
14187:         return self.scatter_add(dim, index, src);
14188:       };
14189:       return wrap(dispatch_scatter_add(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
14190:     }
14191:     case 1: {
14192:       // aten::scatter_add.dimname(Tensor self, Dimname dim, Tensor index, Tensor src) -> Tensor
14193: 
14194:       auto dispatch_scatter_add = [](const at::Tensor & self, at::Dimname dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
14195:         pybind11::gil_scoped_release no_gil;
14196:         return self.scatter_add(dim, index, src);
14197:       };
14198:       return wrap(dispatch_scatter_add(self, _r.dimname(0), _r.tensor(1), _r.tensor(2)));
14199:     }
14200:   }
14201:   Py_RETURN_NONE;
14202:   END_HANDLE_TH_ERRORS
14203: }
14204: 
14205: // scatter_add_
14206: static PyObject * THPVariable_scatter_add_(PyObject* self_, PyObject* args, PyObject* kwargs)
14207: {
14208:   HANDLE_TH_ERRORS
14209:   const Tensor& self = THPVariable_Unpack(self_);
14210:   static PythonArgParser parser({
14211:     "scatter_add_(int64_t dim, Tensor index, Tensor src)",
14212:   }, /*traceable=*/true);
14213: 
14214:   ParsedArgs<3> parsed_args;
14215:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14216:   if(_r.has_torch_function()) {
14217:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14218:   }
14219:   // aten::scatter_add_(Tensor(a!) self, int dim, Tensor index, Tensor src) -> Tensor(a!)
14220: 
14221:   auto dispatch_scatter_add_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src) -> at::Tensor {
14222:     pybind11::gil_scoped_release no_gil;
14223:     return self.scatter_add_(dim, index, src);
14224:   };
14225:   return wrap(dispatch_scatter_add_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2)));
14226:   Py_RETURN_NONE;
14227:   END_HANDLE_TH_ERRORS
14228: }
14229: 
14230: // scatter_reduce
14231: static PyObject * THPVariable_scatter_reduce(PyObject* self_, PyObject* args, PyObject* kwargs)
14232: {
14233:   HANDLE_TH_ERRORS
14234:   const Tensor& self = THPVariable_Unpack(self_);
14235:   static PythonArgParser parser({
14236:     "scatter_reduce(int64_t dim, Tensor index, Tensor src, c10::string_view reduce, *, bool include_self=True)",
14237:   }, /*traceable=*/true);
14238: 
14239:   ParsedArgs<5> parsed_args;
14240:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14241:   if(_r.has_torch_function()) {
14242:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14243:   }
14244:   // aten::scatter_reduce.two(Tensor self, int dim, Tensor index, Tensor src, str reduce, *, bool include_self=True) -> Tensor
14245: 
14246:   auto dispatch_scatter_reduce = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src, c10::string_view reduce, bool include_self) -> at::Tensor {
14247:     pybind11::gil_scoped_release no_gil;
14248:     return self.scatter_reduce(dim, index, src, reduce, include_self);
14249:   };
14250:   return wrap(dispatch_scatter_reduce(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.stringView(3), _r.toBool(4)));
14251:   Py_RETURN_NONE;
14252:   END_HANDLE_TH_ERRORS
14253: }
14254: 
14255: // scatter_reduce_
14256: static PyObject * THPVariable_scatter_reduce_(PyObject* self_, PyObject* args, PyObject* kwargs)
14257: {
14258:   HANDLE_TH_ERRORS
14259:   const Tensor& self = THPVariable_Unpack(self_);
14260:   static PythonArgParser parser({
14261:     "scatter_reduce_(int64_t dim, Tensor index, Tensor src, c10::string_view reduce, *, bool include_self=True)",
14262:   }, /*traceable=*/true);
14263: 
14264:   ParsedArgs<5> parsed_args;
14265:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14266:   if(_r.has_torch_function()) {
14267:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14268:   }
14269:   // aten::scatter_reduce_.two(Tensor(a!) self, int dim, Tensor index, Tensor src, str reduce, *, bool include_self=True) -> Tensor(a!)
14270: 
14271:   auto dispatch_scatter_reduce_ = [](const at::Tensor & self, int64_t dim, const at::Tensor & index, const at::Tensor & src, c10::string_view reduce, bool include_self) -> at::Tensor {
14272:     pybind11::gil_scoped_release no_gil;
14273:     return self.scatter_reduce_(dim, index, src, reduce, include_self);
14274:   };
14275:   return wrap(dispatch_scatter_reduce_(self, _r.toInt64(0), _r.tensor(1), _r.tensor(2), _r.stringView(3), _r.toBool(4)));
14276:   Py_RETURN_NONE;
14277:   END_HANDLE_TH_ERRORS
14278: }
14279: 
14280: \
```

- EN: The main execution path in this span is carried by `THPVariable_scatter_add`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_scatter_add`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14281-14400

```cpp
14281: // select
14282: static PyObject * THPVariable_select(PyObject* self_, PyObject* args, PyObject* kwargs)
14283: {
14284:   HANDLE_TH_ERRORS
14285:   const Tensor& self = THPVariable_Unpack(self_);
14286:   static PythonArgParser parser({
14287:     "select(Dimname dim, int64_t index)",
14288:     "select(int64_t dim, SymInt index)",
14289:   }, /*traceable=*/true);
14290: 
14291:   ParsedArgs<2> parsed_args;
14292:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14293:   if(_r.has_torch_function()) {
14294:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14295:   }
14296:   switch (_r.idx) {
14297:     case 0: {
14298:       // aten::select.Dimname(Tensor(a) self, Dimname dim, int index) -> Tensor(a)
14299: 
14300:       auto dispatch_select = [](const at::Tensor & self, at::Dimname dim, int64_t index) -> at::Tensor {
14301:         pybind11::gil_scoped_release no_gil;
14302:         return self.select(dim, index);
14303:       };
14304:       return wrap(dispatch_select(self, _r.dimname(0), _r.toInt64(1)));
14305:     }
14306:     case 1: {
14307:       // aten::select.int(Tensor(a) self, int dim, SymInt index) -> Tensor(a)
14308: 
14309:       auto dispatch_select = [](const at::Tensor & self, int64_t dim, c10::SymInt index) -> at::Tensor {
14310:         pybind11::gil_scoped_release no_gil;
14311:         return self.select_symint(dim, index);
14312:       };
14313:       return wrap(dispatch_select(self, _r.toInt64(0), _r.toSymInt(1)));
14314:     }
14315:   }
14316:   Py_RETURN_NONE;
14317:   END_HANDLE_TH_ERRORS
14318: }
14319: 
14320: // select_scatter
14321: static PyObject * THPVariable_select_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
14322: {
14323:   HANDLE_TH_ERRORS
14324:   const Tensor& self = THPVariable_Unpack(self_);
14325:   static PythonArgParser parser({
14326:     "select_scatter(Tensor src, int64_t dim, SymInt index)",
14327:   }, /*traceable=*/true);
14328: 
14329:   ParsedArgs<3> parsed_args;
14330:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14331:   if(_r.has_torch_function()) {
14332:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14333:   }
14334:   // aten::select_scatter(Tensor self, Tensor src, int dim, SymInt index) -> Tensor
14335: 
14336:   auto dispatch_select_scatter = [](const at::Tensor & self, const at::Tensor & src, int64_t dim, c10::SymInt index) -> at::Tensor {
14337:     pybind11::gil_scoped_release no_gil;
14338:     return self.select_scatter_symint(src, dim, index);
14339:   };
14340:   return wrap(dispatch_select_scatter(self, _r.tensor(0), _r.toInt64(1), _r.toSymInt(2)));
14341:   Py_RETURN_NONE;
14342:   END_HANDLE_TH_ERRORS
14343: }
14344: 
14345: // sgn
14346: static PyObject * THPVariable_sgn(PyObject* self_, PyObject* args)
14347: {
14348:   HANDLE_TH_ERRORS
14349:   const Tensor& self = THPVariable_Unpack(self_);
14350:   if (has_torch_function(self_)) {
14351:     return handle_torch_function(self_, "sgn");
14352:   }
14353:   // aten::sgn(Tensor self) -> Tensor
14354: 
14355:   auto dispatch_sgn = [](const at::Tensor & self) -> at::Tensor {
14356:     pybind11::gil_scoped_release no_gil;
14357:     return self.sgn();
14358:   };
14359:   return wrap(dispatch_sgn(self));
14360:   END_HANDLE_TH_ERRORS
14361: }
14362: 
14363: // sgn_
14364: static PyObject * THPVariable_sgn_(PyObject* self_, PyObject* args)
14365: {
14366:   HANDLE_TH_ERRORS
14367:   const Tensor& self = THPVariable_Unpack(self_);
14368:   if (has_torch_function(self_)) {
14369:     return handle_torch_function(self_, "sgn_");
14370:   }
14371:   // aten::sgn_(Tensor(a!) self) -> Tensor(a!)
14372: 
14373:   auto dispatch_sgn_ = [](const at::Tensor & self) -> at::Tensor {
14374:     pybind11::gil_scoped_release no_gil;
14375:     return self.sgn_();
14376:   };
14377:   return wrap(dispatch_sgn_(self));
14378:   END_HANDLE_TH_ERRORS
14379: }
14380: 
14381: // sigmoid
14382: static PyObject * THPVariable_sigmoid(PyObject* self_, PyObject* args)
14383: {
14384:   HANDLE_TH_ERRORS
14385:   const Tensor& self = THPVariable_Unpack(self_);
14386:   if (has_torch_function(self_)) {
14387:     return handle_torch_function(self_, "sigmoid");
14388:   }
14389:   // aten::sigmoid(Tensor self) -> Tensor
14390: 
14391:   auto dispatch_sigmoid = [](const at::Tensor & self) -> at::Tensor {
14392:     pybind11::gil_scoped_release no_gil;
14393:     return self.sigmoid();
14394:   };
14395:   return wrap(dispatch_sigmoid(self));
14396:   END_HANDLE_TH_ERRORS
14397: }
14398: 
14399: // sigmoid_
14400: static PyObject * THPVariable_sigmoid_(PyObject* self_, PyObject* args)
```

- EN: The main execution path in this span is carried by `THPVariable_select`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_select`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14401-14520

```cpp
14401: {
14402:   HANDLE_TH_ERRORS
14403:   const Tensor& self = THPVariable_Unpack(self_);
14404:   if (has_torch_function(self_)) {
14405:     return handle_torch_function(self_, "sigmoid_");
14406:   }
14407:   // aten::sigmoid_(Tensor(a!) self) -> Tensor(a!)
14408: 
14409:   auto dispatch_sigmoid_ = [](const at::Tensor & self) -> at::Tensor {
14410:     pybind11::gil_scoped_release no_gil;
14411:     return self.sigmoid_();
14412:   };
14413:   return wrap(dispatch_sigmoid_(self));
14414:   END_HANDLE_TH_ERRORS
14415: }
14416: 
14417: // sign
14418: static PyObject * THPVariable_sign(PyObject* self_, PyObject* args)
14419: {
14420:   HANDLE_TH_ERRORS
14421:   const Tensor& self = THPVariable_Unpack(self_);
14422:   if (has_torch_function(self_)) {
14423:     return handle_torch_function(self_, "sign");
14424:   }
14425:   // aten::sign(Tensor self) -> Tensor
14426: 
14427:   auto dispatch_sign = [](const at::Tensor & self) -> at::Tensor {
14428:     pybind11::gil_scoped_release no_gil;
14429:     return self.sign();
14430:   };
14431:   return wrap(dispatch_sign(self));
14432:   END_HANDLE_TH_ERRORS
14433: }
14434: 
14435: // sign_
14436: static PyObject * THPVariable_sign_(PyObject* self_, PyObject* args)
14437: {
14438:   HANDLE_TH_ERRORS
14439:   const Tensor& self = THPVariable_Unpack(self_);
14440:   if (has_torch_function(self_)) {
14441:     return handle_torch_function(self_, "sign_");
14442:   }
14443:   // aten::sign_(Tensor(a!) self) -> Tensor(a!)
14444: 
14445:   auto dispatch_sign_ = [](const at::Tensor & self) -> at::Tensor {
14446:     pybind11::gil_scoped_release no_gil;
14447:     return self.sign_();
14448:   };
14449:   return wrap(dispatch_sign_(self));
14450:   END_HANDLE_TH_ERRORS
14451: }
14452: 
14453: // signbit
14454: static PyObject * THPVariable_signbit(PyObject* self_, PyObject* args)
14455: {
14456:   HANDLE_TH_ERRORS
14457:   const Tensor& self = THPVariable_Unpack(self_);
14458:   if (has_torch_function(self_)) {
14459:     return handle_torch_function(self_, "signbit");
14460:   }
14461:   // aten::signbit(Tensor self) -> Tensor
14462: 
14463:   auto dispatch_signbit = [](const at::Tensor & self) -> at::Tensor {
14464:     pybind11::gil_scoped_release no_gil;
14465:     return self.signbit();
14466:   };
14467:   return wrap(dispatch_signbit(self));
14468:   END_HANDLE_TH_ERRORS
14469: }
14470: 
14471: // sin
14472: static PyObject * THPVariable_sin(PyObject* self_, PyObject* args)
14473: {
14474:   HANDLE_TH_ERRORS
14475:   const Tensor& self = THPVariable_Unpack(self_);
14476:   if (has_torch_function(self_)) {
14477:     return handle_torch_function(self_, "sin");
14478:   }
14479:   // aten::sin(Tensor self) -> Tensor
14480: 
14481:   auto dispatch_sin = [](const at::Tensor & self) -> at::Tensor {
14482:     pybind11::gil_scoped_release no_gil;
14483:     return self.sin();
14484:   };
14485:   return wrap(dispatch_sin(self));
14486:   END_HANDLE_TH_ERRORS
14487: }
14488: 
14489: // sin_
14490: static PyObject * THPVariable_sin_(PyObject* self_, PyObject* args)
14491: {
14492:   HANDLE_TH_ERRORS
14493:   const Tensor& self = THPVariable_Unpack(self_);
14494:   if (has_torch_function(self_)) {
14495:     return handle_torch_function(self_, "sin_");
14496:   }
14497:   // aten::sin_(Tensor(a!) self) -> Tensor(a!)
14498: 
14499:   auto dispatch_sin_ = [](const at::Tensor & self) -> at::Tensor {
14500:     pybind11::gil_scoped_release no_gil;
14501:     return self.sin_();
14502:   };
14503:   return wrap(dispatch_sin_(self));
14504:   END_HANDLE_TH_ERRORS
14505: }
14506: 
14507: // sinc
14508: static PyObject * THPVariable_sinc(PyObject* self_, PyObject* args)
14509: {
14510:   HANDLE_TH_ERRORS
14511:   const Tensor& self = THPVariable_Unpack(self_);
14512:   if (has_torch_function(self_)) {
14513:     return handle_torch_function(self_, "sinc");
14514:   }
14515:   // aten::sinc(Tensor self) -> Tensor
14516: 
14517:   auto dispatch_sinc = [](const at::Tensor & self) -> at::Tensor {
14518:     pybind11::gil_scoped_release no_gil;
14519:     return self.sinc();
14520:   };
```

- EN: The main execution path in this span is carried by `THPVariable_Unpack`, `handle_torch_function`, `sigmoid_`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_Unpack`, `handle_torch_function`, `sigmoid_` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14521-14640

```cpp
14521:   return wrap(dispatch_sinc(self));
14522:   END_HANDLE_TH_ERRORS
14523: }
14524: 
14525: // sinc_
14526: static PyObject * THPVariable_sinc_(PyObject* self_, PyObject* args)
14527: {
14528:   HANDLE_TH_ERRORS
14529:   const Tensor& self = THPVariable_Unpack(self_);
14530:   if (has_torch_function(self_)) {
14531:     return handle_torch_function(self_, "sinc_");
14532:   }
14533:   // aten::sinc_(Tensor(a!) self) -> Tensor(a!)
14534: 
14535:   auto dispatch_sinc_ = [](const at::Tensor & self) -> at::Tensor {
14536:     pybind11::gil_scoped_release no_gil;
14537:     return self.sinc_();
14538:   };
14539:   return wrap(dispatch_sinc_(self));
14540:   END_HANDLE_TH_ERRORS
14541: }
14542: 
14543: // sinh
14544: static PyObject * THPVariable_sinh(PyObject* self_, PyObject* args)
14545: {
14546:   HANDLE_TH_ERRORS
14547:   const Tensor& self = THPVariable_Unpack(self_);
14548:   if (has_torch_function(self_)) {
14549:     return handle_torch_function(self_, "sinh");
14550:   }
14551:   // aten::sinh(Tensor self) -> Tensor
14552: 
14553:   auto dispatch_sinh = [](const at::Tensor & self) -> at::Tensor {
14554:     pybind11::gil_scoped_release no_gil;
14555:     return self.sinh();
14556:   };
14557:   return wrap(dispatch_sinh(self));
14558:   END_HANDLE_TH_ERRORS
14559: }
14560: 
14561: // sinh_
14562: static PyObject * THPVariable_sinh_(PyObject* self_, PyObject* args)
14563: {
14564:   HANDLE_TH_ERRORS
14565:   const Tensor& self = THPVariable_Unpack(self_);
14566:   if (has_torch_function(self_)) {
14567:     return handle_torch_function(self_, "sinh_");
14568:   }
14569:   // aten::sinh_(Tensor(a!) self) -> Tensor(a!)
14570: 
14571:   auto dispatch_sinh_ = [](const at::Tensor & self) -> at::Tensor {
14572:     pybind11::gil_scoped_release no_gil;
14573:     return self.sinh_();
14574:   };
14575:   return wrap(dispatch_sinh_(self));
14576:   END_HANDLE_TH_ERRORS
14577: }
14578: 
14579: // slice_inverse
14580: static PyObject * THPVariable_slice_inverse(PyObject* self_, PyObject* args, PyObject* kwargs)
14581: {
14582:   HANDLE_TH_ERRORS
14583:   const Tensor& self = THPVariable_Unpack(self_);
14584:   static PythonArgParser parser({
14585:     "slice_inverse(Tensor src, int64_t dim=0, SymInt? start=None, SymInt? end=None, SymInt step=1)",
14586:   }, /*traceable=*/true);
14587: 
14588:   ParsedArgs<5> parsed_args;
14589:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14590:   if(_r.has_torch_function()) {
14591:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14592:   }
14593:   // aten::slice_inverse(Tensor(a) self, Tensor src, int dim=0, SymInt? start=None, SymInt? end=None, SymInt step=1) -> Tensor(a)
14594: 
14595:   auto dispatch_slice_inverse = [](const at::Tensor & self, const at::Tensor & src, int64_t dim, ::std::optional<c10::SymInt> start, ::std::optional<c10::SymInt> end, c10::SymInt step) -> at::Tensor {
14596:     pybind11::gil_scoped_release no_gil;
14597:     return self.slice_inverse_symint(src, dim, start, end, step);
14598:   };
14599:   return wrap(dispatch_slice_inverse(self, _r.tensor(0), _r.toInt64(1), _r.toSymIntOptional(2), _r.toSymIntOptional(3), _r.toSymInt(4)));
14600:   Py_RETURN_NONE;
14601:   END_HANDLE_TH_ERRORS
14602: }
14603: 
14604: // slice_scatter
14605: static PyObject * THPVariable_slice_scatter(PyObject* self_, PyObject* args, PyObject* kwargs)
14606: {
14607:   HANDLE_TH_ERRORS
14608:   const Tensor& self = THPVariable_Unpack(self_);
14609:   static PythonArgParser parser({
14610:     "slice_scatter(Tensor src, int64_t dim=0, SymInt? start=None, SymInt? end=None, SymInt step=1)",
14611:   }, /*traceable=*/true);
14612: 
14613:   ParsedArgs<5> parsed_args;
14614:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14615:   if(_r.has_torch_function()) {
14616:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14617:   }
14618:   // aten::slice_scatter(Tensor self, Tensor src, int dim=0, SymInt? start=None, SymInt? end=None, SymInt step=1) -> Tensor
14619: 
14620:   auto dispatch_slice_scatter = [](const at::Tensor & self, const at::Tensor & src, int64_t dim, ::std::optional<c10::SymInt> start, ::std::optional<c10::SymInt> end, c10::SymInt step) -> at::Tensor {
14621:     pybind11::gil_scoped_release no_gil;
14622:     return self.slice_scatter_symint(src, dim, start, end, step);
14623:   };
14624:   return wrap(dispatch_slice_scatter(self, _r.tensor(0), _r.toInt64(1), _r.toSymIntOptional(2), _r.toSymIntOptional(3), _r.toSymInt(4)));
14625:   Py_RETURN_NONE;
14626:   END_HANDLE_TH_ERRORS
14627: }
14628: 
14629: // slogdet
14630: static PyObject * THPVariable_slogdet(PyObject* self_, PyObject* args)
14631: {
14632:   HANDLE_TH_ERRORS
14633:   static PyTypeObject* NamedTuple = generated::get_slogdet_structseq();
14634:   const Tensor& self = THPVariable_Unpack(self_);
14635:   if (has_torch_function(self_)) {
14636:     return handle_torch_function(self_, "slogdet");
14637:   }
14638:   // aten::slogdet(Tensor self) -> (Tensor sign, Tensor logabsdet)
14639: 
14640:   auto dispatch_slogdet = [](const at::Tensor & self) -> ::std::tuple<at::Tensor,at::Tensor> {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_sinc_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_sinc_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14641-14760

```cpp
14641:     pybind11::gil_scoped_release no_gil;
14642:     return self.slogdet();
14643:   };
14644:   return wrap(NamedTuple, dispatch_slogdet(self));
14645:   END_HANDLE_TH_ERRORS
14646: }
14647: 
14648: // smm
14649: static PyObject * THPVariable_smm(PyObject* self_, PyObject* args, PyObject* kwargs)
14650: {
14651:   HANDLE_TH_ERRORS
14652:   const Tensor& self = THPVariable_Unpack(self_);
14653:   static PythonArgParser parser({
14654:     "smm(Tensor mat2)",
14655:   }, /*traceable=*/true);
14656: 
14657:   ParsedArgs<1> parsed_args;
14658:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14659:   if(_r.has_torch_function()) {
14660:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14661:   }
14662:   // aten::smm(Tensor self, Tensor mat2) -> Tensor
14663: 
14664:   auto dispatch_smm = [](const at::Tensor & self, const at::Tensor & mat2) -> at::Tensor {
14665:     pybind11::gil_scoped_release no_gil;
14666:     return self.smm(mat2);
14667:   };
14668:   return wrap(dispatch_smm(self, _r.tensor(0)));
14669:   Py_RETURN_NONE;
14670:   END_HANDLE_TH_ERRORS
14671: }
14672: 
14673: \
14674: // softmax
14675: static PyObject * THPVariable_softmax(PyObject* self_, PyObject* args, PyObject* kwargs)
14676: {
14677:   HANDLE_TH_ERRORS
14678:   const Tensor& self = THPVariable_Unpack(self_);
14679:   static PythonArgParser parser({
14680:     "softmax(int64_t dim, ScalarType? dtype=None)",
14681:     "softmax(Dimname dim, *, ScalarType? dtype=None)",
14682:   }, /*traceable=*/true);
14683: 
14684:   ParsedArgs<2> parsed_args;
14685:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14686:   if(_r.has_torch_function()) {
14687:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14688:   }
14689:   switch (_r.idx) {
14690:     case 0: {
14691:       // aten::softmax.int(Tensor self, int dim, ScalarType? dtype=None) -> Tensor
14692: 
14693:       auto dispatch_softmax = [](const at::Tensor & self, int64_t dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
14694:         pybind11::gil_scoped_release no_gil;
14695:         return self.softmax(dim, dtype);
14696:       };
14697:       return wrap(dispatch_softmax(self, _r.toInt64(0), _r.scalartypeOptional(1)));
14698:     }
14699:     case 1: {
14700:       // aten::softmax.Dimname(Tensor self, Dimname dim, *, ScalarType? dtype=None) -> Tensor
14701: 
14702:       auto dispatch_softmax = [](const at::Tensor & self, at::Dimname dim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
14703:         pybind11::gil_scoped_release no_gil;
14704:         return self.softmax(dim, dtype);
14705:       };
14706:       return wrap(dispatch_softmax(self, _r.dimname(0), _r.scalartypeOptional(1)));
14707:     }
14708:   }
14709:   Py_RETURN_NONE;
14710:   END_HANDLE_TH_ERRORS
14711: }
14712: 
14713: \
14714: // sort
14715: static PyObject * THPVariable_sort(PyObject* self_, PyObject* args, PyObject* kwargs)
14716: {
14717:   HANDLE_TH_ERRORS
14718:   static PyTypeObject* NamedTuple = generated::get_sort_structseq();
14719:   const Tensor& self = THPVariable_Unpack(self_);
14720:   static PythonArgParser parser({
14721:     "sort(*, bool? stable, int64_t dim=-1, bool descending=False)",
14722:     "sort(int64_t dim=-1, bool descending=False)",
14723:     "sort(*, bool? stable, Dimname dim, bool descending=False)",
14724:     "sort(Dimname dim, bool descending=False)",
14725:   }, /*traceable=*/true);
14726: 
14727:   ParsedArgs<3> parsed_args;
14728:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14729:   if(_r.has_torch_function()) {
14730:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14731:   }
14732:   switch (_r.idx) {
14733:     case 0: {
14734:       // aten::sort.stable(Tensor self, *, bool? stable, int dim=-1, bool descending=False) -> (Tensor values, Tensor indices)
14735: 
14736:       auto dispatch_sort = [](const at::Tensor & self, ::std::optional<bool> stable, int64_t dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
14737:         pybind11::gil_scoped_release no_gil;
14738:         return self.sort(stable, dim, descending);
14739:       };
14740:       return wrap(NamedTuple, dispatch_sort(self, _r.toBoolOptional(0), _r.toInt64(1), _r.toBool(2)));
14741:     }
14742:     case 1: {
14743:       // aten::sort(Tensor self, int dim=-1, bool descending=False) -> (Tensor values, Tensor indices)
14744: 
14745:       auto dispatch_sort = [](const at::Tensor & self, int64_t dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
14746:         pybind11::gil_scoped_release no_gil;
14747:         return self.sort(dim, descending);
14748:       };
14749:       return wrap(NamedTuple, dispatch_sort(self, _r.toInt64(0), _r.toBool(1)));
14750:     }
14751:     case 2: {
14752:       // aten::sort.dimname_stable(Tensor self, *, bool? stable, Dimname dim, bool descending=False) -> (Tensor values, Tensor indices)
14753: 
14754:       auto dispatch_sort = [](const at::Tensor & self, ::std::optional<bool> stable, at::Dimname dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
14755:         pybind11::gil_scoped_release no_gil;
14756:         return self.sort(stable, dim, descending);
14757:       };
14758:       return wrap(NamedTuple, dispatch_sort(self, _r.toBoolOptional(0), _r.dimname(1), _r.toBool(2)));
14759:     }
14760:     case 3: {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_smm`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_smm`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 14761-14880

```cpp
14761:       // aten::sort.dimname(Tensor self, Dimname dim, bool descending=False) -> (Tensor values, Tensor indices)
14762: 
14763:       auto dispatch_sort = [](const at::Tensor & self, at::Dimname dim, bool descending) -> ::std::tuple<at::Tensor,at::Tensor> {
14764:         pybind11::gil_scoped_release no_gil;
14765:         return self.sort(dim, descending);
14766:       };
14767:       return wrap(NamedTuple, dispatch_sort(self, _r.dimname(0), _r.toBool(1)));
14768:     }
14769:   }
14770:   Py_RETURN_NONE;
14771:   END_HANDLE_TH_ERRORS
14772: }
14773: 
14774: // sparse_dim
14775: static PyObject * THPVariable_sparse_dim(PyObject* self_, PyObject* args)
14776: {
14777:   HANDLE_TH_ERRORS
14778:   const Tensor& self = THPVariable_Unpack(self_);
14779:   if (has_torch_function(self_)) {
14780:     return handle_torch_function(self_, "sparse_dim");
14781:   }
14782:   // aten::sparse_dim(Tensor self) -> int
14783: 
14784:   auto dispatch_sparse_dim = [](const at::Tensor & self) -> int64_t {
14785:     pybind11::gil_scoped_release no_gil;
14786:     return self.sparse_dim();
14787:   };
14788:   return wrap(dispatch_sparse_dim(self));
14789:   END_HANDLE_TH_ERRORS
14790: }
14791: 
14792: // sparse_mask
14793: static PyObject * THPVariable_sparse_mask(PyObject* self_, PyObject* args, PyObject* kwargs)
14794: {
14795:   HANDLE_TH_ERRORS
14796:   const Tensor& self = THPVariable_Unpack(self_);
14797:   static PythonArgParser parser({
14798:     "sparse_mask(Tensor mask)",
14799:   }, /*traceable=*/true);
14800: 
14801:   ParsedArgs<1> parsed_args;
14802:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14803:   if(_r.has_torch_function()) {
14804:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14805:   }
14806:   // aten::sparse_mask(Tensor self, Tensor mask) -> Tensor
14807: 
14808:   auto dispatch_sparse_mask = [](const at::Tensor & self, const at::Tensor & mask) -> at::Tensor {
14809:     pybind11::gil_scoped_release no_gil;
14810:     return self.sparse_mask(mask);
14811:   };
14812:   return wrap(dispatch_sparse_mask(self, _r.tensor(0)));
14813:   Py_RETURN_NONE;
14814:   END_HANDLE_TH_ERRORS
14815: }
14816: 
14817: // sparse_resize_
14818: static PyObject * THPVariable_sparse_resize_(PyObject* self_, PyObject* args, PyObject* kwargs)
14819: {
14820:   HANDLE_TH_ERRORS
14821:   const Tensor& self = THPVariable_Unpack(self_);
14822:   static PythonArgParser parser({
14823:     "sparse_resize_(IntArrayRef size, int64_t sparse_dim, int64_t dense_dim)",
14824:   }, /*traceable=*/true);
14825: 
14826:   ParsedArgs<3> parsed_args;
14827:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14828:   if(_r.has_torch_function()) {
14829:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14830:   }
14831:   // aten::sparse_resize_(Tensor(a!) self, int[] size, int sparse_dim, int dense_dim) -> Tensor(a!)
14832: 
14833:   auto dispatch_sparse_resize_ = [](const at::Tensor & self, at::IntArrayRef size, int64_t sparse_dim, int64_t dense_dim) -> at::Tensor {
14834:     pybind11::gil_scoped_release no_gil;
14835:     return self.sparse_resize_(size, sparse_dim, dense_dim);
14836:   };
14837:   return wrap(dispatch_sparse_resize_(self, _r.intlist(0), _r.toInt64(1), _r.toInt64(2)));
14838:   Py_RETURN_NONE;
14839:   END_HANDLE_TH_ERRORS
14840: }
14841: 
14842: // sparse_resize_and_clear_
14843: static PyObject * THPVariable_sparse_resize_and_clear_(PyObject* self_, PyObject* args, PyObject* kwargs)
14844: {
14845:   HANDLE_TH_ERRORS
14846:   const Tensor& self = THPVariable_Unpack(self_);
14847:   static PythonArgParser parser({
14848:     "sparse_resize_and_clear_(IntArrayRef size, int64_t sparse_dim, int64_t dense_dim)",
14849:   }, /*traceable=*/true);
14850: 
14851:   ParsedArgs<3> parsed_args;
14852:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14853:   if(_r.has_torch_function()) {
14854:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14855:   }
14856:   // aten::sparse_resize_and_clear_(Tensor(a!) self, int[] size, int sparse_dim, int dense_dim) -> Tensor(a!)
14857: 
14858:   auto dispatch_sparse_resize_and_clear_ = [](const at::Tensor & self, at::IntArrayRef size, int64_t sparse_dim, int64_t dense_dim) -> at::Tensor {
14859:     pybind11::gil_scoped_release no_gil;
14860:     return self.sparse_resize_and_clear_(size, sparse_dim, dense_dim);
14861:   };
14862:   return wrap(dispatch_sparse_resize_and_clear_(self, _r.intlist(0), _r.toInt64(1), _r.toInt64(2)));
14863:   Py_RETURN_NONE;
14864:   END_HANDLE_TH_ERRORS
14865: }
14866: 
14867: \
14868: // split
14869: static PyObject * THPVariable_split(PyObject* self_, PyObject* args, PyObject* kwargs)
14870: {
14871:   HANDLE_TH_ERRORS
14872:   const Tensor& self = THPVariable_Unpack(self_);
14873:   static PythonArgParser parser({
14874:     "split(SymInt split_size, int64_t dim=0)",
14875:     "split(SymIntArrayRef split_size, int64_t dim=0)",
14876:   }, /*traceable=*/true);
14877: 
14878:   ParsedArgs<2> parsed_args;
14879:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14880:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_sparse_dim`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_sparse_dim`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 14881-15000

```cpp
14881:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14882:   }
14883:   switch (_r.idx) {
14884:     case 0: {
14885:       // aten::split.Tensor(Tensor(a -> *) self, SymInt split_size, int dim=0) -> Tensor(a)[]
14886: 
14887:       auto dispatch_split = [](const at::Tensor & self, c10::SymInt split_size, int64_t dim) -> ::std::vector<at::Tensor> {
14888:         pybind11::gil_scoped_release no_gil;
14889:         return self.split_symint(split_size, dim);
14890:       };
14891:       return wrap(dispatch_split(self, _r.toSymInt(0), _r.toInt64(1)));
14892:     }
14893:     case 1: {
14894:       // aten::split.sizes(Tensor(a -> *) self, SymInt[] split_size, int dim=0) -> Tensor(a)[]
14895: 
14896:       auto dispatch_split = [](const at::Tensor & self, c10::SymIntArrayRef split_size, int64_t dim) -> ::std::vector<at::Tensor> {
14897:         pybind11::gil_scoped_release no_gil;
14898:         return self.split_symint(split_size, dim);
14899:       };
14900:       return wrap(dispatch_split(self, _r.symintlist(0), _r.toInt64(1)));
14901:     }
14902:   }
14903:   Py_RETURN_NONE;
14904:   END_HANDLE_TH_ERRORS
14905: }
14906: 
14907: // split_with_sizes
14908: static PyObject * THPVariable_split_with_sizes(PyObject* self_, PyObject* args, PyObject* kwargs)
14909: {
14910:   HANDLE_TH_ERRORS
14911:   const Tensor& self = THPVariable_Unpack(self_);
14912:   static PythonArgParser parser({
14913:     "split_with_sizes(SymIntArrayRef split_sizes, int64_t dim=0)",
14914:   }, /*traceable=*/true);
14915: 
14916:   ParsedArgs<2> parsed_args;
14917:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
14918:   if(_r.has_torch_function()) {
14919:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
14920:   }
14921:   // aten::split_with_sizes(Tensor(a -> *) self, SymInt[] split_sizes, int dim=0) -> Tensor(a)[]
14922: 
14923:   auto dispatch_split_with_sizes = [](const at::Tensor & self, c10::SymIntArrayRef split_sizes, int64_t dim) -> ::std::vector<at::Tensor> {
14924:     pybind11::gil_scoped_release no_gil;
14925:     return self.split_with_sizes_symint(split_sizes, dim);
14926:   };
14927:   return wrap(dispatch_split_with_sizes(self, _r.symintlist(0), _r.toInt64(1)));
14928:   Py_RETURN_NONE;
14929:   END_HANDLE_TH_ERRORS
14930: }
14931: 
14932: // sqrt
14933: static PyObject * THPVariable_sqrt(PyObject* self_, PyObject* args)
14934: {
14935:   HANDLE_TH_ERRORS
14936:   const Tensor& self = THPVariable_Unpack(self_);
14937:   if (has_torch_function(self_)) {
14938:     return handle_torch_function(self_, "sqrt");
14939:   }
14940:   // aten::sqrt(Tensor self) -> Tensor
14941: 
14942:   auto dispatch_sqrt = [](const at::Tensor & self) -> at::Tensor {
14943:     pybind11::gil_scoped_release no_gil;
14944:     return self.sqrt();
14945:   };
14946:   return wrap(dispatch_sqrt(self));
14947:   END_HANDLE_TH_ERRORS
14948: }
14949: 
14950: // sqrt_
14951: static PyObject * THPVariable_sqrt_(PyObject* self_, PyObject* args)
14952: {
14953:   HANDLE_TH_ERRORS
14954:   const Tensor& self = THPVariable_Unpack(self_);
14955:   if (has_torch_function(self_)) {
14956:     return handle_torch_function(self_, "sqrt_");
14957:   }
14958:   // aten::sqrt_(Tensor(a!) self) -> Tensor(a!)
14959: 
14960:   auto dispatch_sqrt_ = [](const at::Tensor & self) -> at::Tensor {
14961:     pybind11::gil_scoped_release no_gil;
14962:     return self.sqrt_();
14963:   };
14964:   return wrap(dispatch_sqrt_(self));
14965:   END_HANDLE_TH_ERRORS
14966: }
14967: 
14968: // square
14969: static PyObject * THPVariable_square(PyObject* self_, PyObject* args)
14970: {
14971:   HANDLE_TH_ERRORS
14972:   const Tensor& self = THPVariable_Unpack(self_);
14973:   if (has_torch_function(self_)) {
14974:     return handle_torch_function(self_, "square");
14975:   }
14976:   // aten::square(Tensor self) -> Tensor
14977: 
14978:   auto dispatch_square = [](const at::Tensor & self) -> at::Tensor {
14979:     pybind11::gil_scoped_release no_gil;
14980:     return self.square();
14981:   };
14982:   return wrap(dispatch_square(self));
14983:   END_HANDLE_TH_ERRORS
14984: }
14985: 
14986: // square_
14987: static PyObject * THPVariable_square_(PyObject* self_, PyObject* args)
14988: {
14989:   HANDLE_TH_ERRORS
14990:   const Tensor& self = THPVariable_Unpack(self_);
14991:   if (has_torch_function(self_)) {
14992:     return handle_torch_function(self_, "square_");
14993:   }
14994:   // aten::square_(Tensor(a!) self) -> Tensor(a!)
14995: 
14996:   auto dispatch_square_ = [](const at::Tensor & self) -> at::Tensor {
14997:     pybind11::gil_scoped_release no_gil;
14998:     return self.square_();
14999:   };
15000:   return wrap(dispatch_square_(self));
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `wrap`, `THPVariable_split_with_sizes`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `wrap`, `THPVariable_split_with_sizes` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15001-15120

```cpp
15001:   END_HANDLE_TH_ERRORS
15002: }
15003: 
15004: \
15005: // squeeze
15006: static PyObject * THPVariable_squeeze(PyObject* self_, PyObject* args, PyObject* kwargs)
15007: {
15008:   HANDLE_TH_ERRORS
15009:   const Tensor& self = THPVariable_Unpack(self_);
15010:   static PythonArgParser parser({
15011:     "squeeze()",
15012:     "squeeze(int64_t dim)",
15013:     "squeeze(IntArrayRef dim)",
15014:     "squeeze(Dimname dim)",
15015:   }, /*traceable=*/true);
15016: 
15017:   ParsedArgs<1> parsed_args;
15018:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15019:   if(_r.has_torch_function()) {
15020:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15021:   }
15022:   switch (_r.idx) {
15023:     case 0: {
15024:       // aten::squeeze(Tensor(a) self) -> Tensor(a)
15025: 
15026:       auto dispatch_squeeze = [](const at::Tensor & self) -> at::Tensor {
15027:         pybind11::gil_scoped_release no_gil;
15028:         return self.squeeze();
15029:       };
15030:       return wrap(dispatch_squeeze(self));
15031:     }
15032:     case 1: {
15033:       // aten::squeeze.dim(Tensor(a) self, int dim) -> Tensor(a)
15034: 
15035:       auto dispatch_squeeze = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
15036:         pybind11::gil_scoped_release no_gil;
15037:         return self.squeeze(dim);
15038:       };
15039:       return wrap(dispatch_squeeze(self, _r.toInt64(0)));
15040:     }
15041:     case 2: {
15042:       // aten::squeeze.dims(Tensor(a) self, int[] dim) -> Tensor(a)
15043: 
15044:       auto dispatch_squeeze = [](const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
15045:         pybind11::gil_scoped_release no_gil;
15046:         return self.squeeze(dim);
15047:       };
15048:       return wrap(dispatch_squeeze(self, _r.intlist(0)));
15049:     }
15050:     case 3: {
15051:       // aten::squeeze.dimname(Tensor(a) self, Dimname dim) -> Tensor(a)
15052: 
15053:       auto dispatch_squeeze = [](const at::Tensor & self, at::Dimname dim) -> at::Tensor {
15054:         pybind11::gil_scoped_release no_gil;
15055:         return self.squeeze(dim);
15056:       };
15057:       return wrap(dispatch_squeeze(self, _r.dimname(0)));
15058:     }
15059:   }
15060:   Py_RETURN_NONE;
15061:   END_HANDLE_TH_ERRORS
15062: }
15063: 
15064: \
15065: // squeeze_
15066: static PyObject * THPVariable_squeeze_(PyObject* self_, PyObject* args, PyObject* kwargs)
15067: {
15068:   HANDLE_TH_ERRORS
15069:   const Tensor& self = THPVariable_Unpack(self_);
15070:   static PythonArgParser parser({
15071:     "squeeze_()",
15072:     "squeeze_(int64_t dim)",
15073:     "squeeze_(IntArrayRef dim)",
15074:     "squeeze_(Dimname dim)",
15075:   }, /*traceable=*/true);
15076: 
15077:   ParsedArgs<1> parsed_args;
15078:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15079:   if(_r.has_torch_function()) {
15080:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15081:   }
15082:   switch (_r.idx) {
15083:     case 0: {
15084:       // aten::squeeze_(Tensor(a!) self) -> Tensor(a!)
15085: 
15086:       auto dispatch_squeeze_ = [](const at::Tensor & self) -> at::Tensor {
15087:         pybind11::gil_scoped_release no_gil;
15088:         return self.squeeze_();
15089:       };
15090:       return wrap(dispatch_squeeze_(self));
15091:     }
15092:     case 1: {
15093:       // aten::squeeze_.dim(Tensor(a!) self, int dim) -> Tensor(a!)
15094: 
15095:       auto dispatch_squeeze_ = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
15096:         pybind11::gil_scoped_release no_gil;
15097:         return self.squeeze_(dim);
15098:       };
15099:       return wrap(dispatch_squeeze_(self, _r.toInt64(0)));
15100:     }
15101:     case 2: {
15102:       // aten::squeeze_.dims(Tensor(a!) self, int[] dim) -> Tensor(a!)
15103: 
15104:       auto dispatch_squeeze_ = [](const at::Tensor & self, at::IntArrayRef dim) -> at::Tensor {
15105:         pybind11::gil_scoped_release no_gil;
15106:         return self.squeeze_(dim);
15107:       };
15108:       return wrap(dispatch_squeeze_(self, _r.intlist(0)));
15109:     }
15110:     case 3: {
15111:       // aten::squeeze_.dimname(Tensor(a!) self, Dimname dim) -> Tensor(a!)
15112: 
15113:       auto dispatch_squeeze_ = [](const at::Tensor & self, at::Dimname dim) -> at::Tensor {
15114:         pybind11::gil_scoped_release no_gil;
15115:         return self.squeeze_(dim);
15116:       };
15117:       return wrap(dispatch_squeeze_(self, _r.dimname(0)));
15118:     }
15119:   }
15120:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `THPVariable_squeeze`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_squeeze`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15121-15240

```cpp
15121:   END_HANDLE_TH_ERRORS
15122: }
15123: 
15124: \
15125: // sspaddmm
15126: static PyObject * THPVariable_sspaddmm(PyObject* self_, PyObject* args, PyObject* kwargs)
15127: {
15128:   HANDLE_TH_ERRORS
15129:   const Tensor& self = THPVariable_Unpack(self_);
15130:   static PythonArgParser parser({
15131:     "sspaddmm(Scalar beta, Scalar alpha, Tensor mat1, Tensor mat2)|deprecated",
15132:     "sspaddmm(Scalar beta, Tensor mat1, Tensor mat2)|deprecated",
15133:     "sspaddmm(Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1)",
15134:   }, /*traceable=*/true);
15135: 
15136:   ParsedArgs<4> parsed_args;
15137:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15138:   if(_r.has_torch_function()) {
15139:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15140:   }
15141:   switch (_r.idx) {
15142:     case 0: {
15143:       // [deprecated] aten::sspaddmm(Scalar beta, Tensor self, Scalar alpha, Tensor mat1, Tensor mat2) -> Tensor
15144: 
15145:       auto dispatch_sspaddmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
15146:         pybind11::gil_scoped_release no_gil;
15147:         return self.sspaddmm(mat1, mat2, beta, alpha);
15148:       };
15149:       return wrap(dispatch_sspaddmm(_r.scalar(0), self, _r.scalar(1), _r.tensor(2), _r.tensor(3)));
15150:     }
15151:     case 1: {
15152:       // [deprecated] aten::sspaddmm(Scalar beta, Tensor self, Tensor mat1, Tensor mat2) -> Tensor
15153: 
15154:       auto dispatch_sspaddmm = [](const at::Scalar & beta, const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2) -> at::Tensor {
15155:         pybind11::gil_scoped_release no_gil;
15156:         return self.sspaddmm(mat1, mat2, beta, 1);
15157:       };
15158:       return wrap(dispatch_sspaddmm(_r.scalar(0), self, _r.tensor(1), _r.tensor(2)));
15159:     }
15160:     case 2: {
15161:       // aten::sspaddmm(Tensor self, Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1) -> Tensor
15162: 
15163:       auto dispatch_sspaddmm = [](const at::Tensor & self, const at::Tensor & mat1, const at::Tensor & mat2, const at::Scalar & beta, const at::Scalar & alpha) -> at::Tensor {
15164:         pybind11::gil_scoped_release no_gil;
15165:         return self.sspaddmm(mat1, mat2, beta, alpha);
15166:       };
15167:       return wrap(dispatch_sspaddmm(self, _r.tensor(0), _r.tensor(1), _r.scalar(2), _r.scalar(3)));
15168:     }
15169:   }
15170:   Py_RETURN_NONE;
15171:   END_HANDLE_TH_ERRORS
15172: }
15173: 
15174: \
15175: // std
15176: static PyObject * THPVariable_std(PyObject* self_, PyObject* args, PyObject* kwargs)
15177: {
15178:   HANDLE_TH_ERRORS
15179:   const Tensor& self = THPVariable_Unpack(self_);
15180:   static PythonArgParser parser({
15181:     "std(IntArrayRef[1]? dim, bool unbiased=True, bool keepdim=False)",
15182:     "std(IntArrayRef[1]? dim=None, *, Scalar? correction=None, bool keepdim=False)",
15183:     "std(bool unbiased=True)",
15184:     "std(DimnameList[1] dim, bool unbiased=True, bool keepdim=False)",
15185:     "std(DimnameList[1] dim, *, Scalar? correction=None, bool keepdim=False)",
15186:   }, /*traceable=*/true);
15187: 
15188:   ParsedArgs<3> parsed_args;
15189:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15190:   if(_r.has_torch_function()) {
15191:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15192:   }
15193:   switch (_r.idx) {
15194:     case 0: {
15195:       // aten::std.dim(Tensor self, int[1]? dim, bool unbiased=True, bool keepdim=False) -> Tensor
15196: 
15197:       auto dispatch_std = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool unbiased, bool keepdim) -> at::Tensor {
15198:         pybind11::gil_scoped_release no_gil;
15199:         return self.std(dim, unbiased, keepdim);
15200:       };
15201:       return wrap(dispatch_std(self, _r.intlistOptional(0), _r.toBool(1), _r.toBool(2)));
15202:     }
15203:     case 1: {
15204:       // aten::std.correction(Tensor self, int[1]? dim=None, *, Scalar? correction=None, bool keepdim=False) -> Tensor
15205: 
15206:       auto dispatch_std = [](const at::Tensor & self, at::OptionalIntArrayRef dim, const ::std::optional<at::Scalar> & correction, bool keepdim) -> at::Tensor {
15207:         pybind11::gil_scoped_release no_gil;
15208:         return self.std(dim, correction, keepdim);
15209:       };
15210:       return wrap(dispatch_std(self, _r.intlistOptional(0), _r.scalarOptional(1), _r.toBool(2)));
15211:     }
15212:     case 2: {
15213:       // aten::std(Tensor self, bool unbiased=True) -> Tensor
15214: 
15215:       auto dispatch_std = [](const at::Tensor & self, bool unbiased) -> at::Tensor {
15216:         pybind11::gil_scoped_release no_gil;
15217:         return self.std(unbiased);
15218:       };
15219:       return wrap(dispatch_std(self, _r.toBool(0)));
15220:     }
15221:     case 3: {
15222:       // aten::std.names_dim(Tensor self, Dimname[1] dim, bool unbiased=True, bool keepdim=False) -> Tensor
15223: 
15224:       auto dispatch_std = [](const at::Tensor & self, at::DimnameList dim, bool unbiased, bool keepdim) -> at::Tensor {
15225:         pybind11::gil_scoped_release no_gil;
15226:         return self.std(dim, unbiased, keepdim);
15227:       };
15228:       return wrap(dispatch_std(self, _r.dimnamelist(0), _r.toBool(1), _r.toBool(2)));
15229:     }
15230:     case 4: {
15231:       // aten::std.correction_names(Tensor self, Dimname[1] dim, *, Scalar? correction=None, bool keepdim=False) -> Tensor
15232: 
15233:       auto dispatch_std = [](const at::Tensor & self, at::DimnameList dim, const ::std::optional<at::Scalar> & correction, bool keepdim) -> at::Tensor {
15234:         pybind11::gil_scoped_release no_gil;
15235:         return self.std(dim, correction, keepdim);
15236:       };
15237:       return wrap(dispatch_std(self, _r.dimnamelist(0), _r.scalarOptional(1), _r.toBool(2)));
15238:     }
15239:   }
15240:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `THPVariable_sspaddmm`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_sspaddmm`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15241-15360

```cpp
15241:   END_HANDLE_TH_ERRORS
15242: }
15243: 
15244: \
15245: // stft
15246: static PyObject * THPVariable_stft(PyObject* self_, PyObject* args, PyObject* kwargs)
15247: {
15248:   HANDLE_TH_ERRORS
15249:   const Tensor& self = THPVariable_Unpack(self_);
15250:   static PythonArgParser parser({
15251:     "stft(int64_t n_fft, int64_t? hop_length=None, int64_t? win_length=None, Tensor? window=None, bool center=True, c10::string_view pad_mode=\"reflect\", bool normalized=False, bool? onesided=None, bool? return_complex=None, bool? align_to_window=None)",
15252:     "stft(int64_t n_fft, int64_t? hop_length=None, int64_t? win_length=None, Tensor? window=None, bool normalized=False, bool? onesided=None, bool? return_complex=None, bool? align_to_window=None)",
15253:   }, /*traceable=*/true);
15254: 
15255:   ParsedArgs<10> parsed_args;
15256:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15257:   if(_r.has_torch_function()) {
15258:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15259:   }
15260:   switch (_r.idx) {
15261:     case 0: {
15262:       // aten::stft.center(Tensor self, int n_fft, int? hop_length=None, int? win_length=None, Tensor? window=None, bool center=True, str pad_mode="reflect", bool normalized=False, bool? onesided=None, bool? return_complex=None, bool? align_to_window=None) -> Tensor
15263: 
15264:       auto dispatch_stft = [](const at::Tensor & self, int64_t n_fft, ::std::optional<int64_t> hop_length, ::std::optional<int64_t> win_length, const ::std::optional<at::Tensor> & window, bool center, c10::string_view pad_mode, bool normalized, ::std::optional<bool> onesided, ::std::optional<bool> return_complex, ::std::optional<bool> align_to_window) -> at::Tensor {
15265:         pybind11::gil_scoped_release no_gil;
15266:         return self.stft(n_fft, hop_length, win_length, window, center, pad_mode, normalized, onesided, return_complex, align_to_window);
15267:       };
15268:       return wrap(dispatch_stft(self, _r.toInt64(0), _r.toInt64Optional(1), _r.toInt64Optional(2), _r.optionalTensor(3), _r.toBool(4), _r.stringView(5), _r.toBool(6), _r.toBoolOptional(7), _r.toBoolOptional(8), _r.toBoolOptional(9)));
15269:     }
15270:     case 1: {
15271:       // aten::stft(Tensor self, int n_fft, int? hop_length=None, int? win_length=None, Tensor? window=None, bool normalized=False, bool? onesided=None, bool? return_complex=None, bool? align_to_window=None) -> Tensor
15272: 
15273:       auto dispatch_stft = [](const at::Tensor & self, int64_t n_fft, ::std::optional<int64_t> hop_length, ::std::optional<int64_t> win_length, const ::std::optional<at::Tensor> & window, bool normalized, ::std::optional<bool> onesided, ::std::optional<bool> return_complex, ::std::optional<bool> align_to_window) -> at::Tensor {
15274:         pybind11::gil_scoped_release no_gil;
15275:         return self.stft(n_fft, hop_length, win_length, window, normalized, onesided, return_complex, align_to_window);
15276:       };
15277:       return wrap(dispatch_stft(self, _r.toInt64(0), _r.toInt64Optional(1), _r.toInt64Optional(2), _r.optionalTensor(3), _r.toBool(4), _r.toBoolOptional(5), _r.toBoolOptional(6), _r.toBoolOptional(7)));
15278:     }
15279:   }
15280:   Py_RETURN_NONE;
15281:   END_HANDLE_TH_ERRORS
15282: }
15283: 
15284: \
15285: // sub
15286: static PyObject * THPVariable_sub(PyObject* self_, PyObject* args, PyObject* kwargs)
15287: {
15288:   HANDLE_TH_ERRORS
15289:   const Tensor& self = THPVariable_Unpack(self_);
15290:   static PythonArgParser parser({
15291:     "sub(Scalar alpha, Tensor other)|deprecated",
15292:     "sub(Tensor other, *, Scalar alpha=1)",
15293:   }, /*traceable=*/true);
15294: 
15295:   ParsedArgs<2> parsed_args;
15296:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15297:   if(_r.has_torch_function()) {
15298:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15299:   }
15300:   switch (_r.idx) {
15301:     case 0: {
15302:       // [deprecated] aten::sub(Tensor self, Scalar alpha, Tensor other) -> Tensor
15303: 
15304:       auto dispatch_sub = [](const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
15305:         pybind11::gil_scoped_release no_gil;
15306:         return self.sub(other, alpha);
15307:       };
15308:       return wrap(dispatch_sub(self, _r.scalar(0), _r.tensor(1)));
15309:     }
15310:     case 1: {
15311:       // aten::sub.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
15312: 
15313:       auto dispatch_sub = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
15314:         pybind11::gil_scoped_release no_gil;
15315:         return self.sub(other, alpha);
15316:       };
15317:       return wrap(dispatch_sub(self, _r.tensor(0), _r.scalar(1)));
15318:     }
15319:   }
15320:   Py_RETURN_NONE;
15321:   END_HANDLE_TH_ERRORS
15322: }
15323: 
15324: \
15325: // sub_
15326: static PyObject * THPVariable_sub_(PyObject* self_, PyObject* args, PyObject* kwargs)
15327: {
15328:   HANDLE_TH_ERRORS
15329:   const Tensor& self = THPVariable_Unpack(self_);
15330:   static PythonArgParser parser({
15331:     "sub_(Scalar alpha, Tensor other)|deprecated",
15332:     "sub_(Tensor other, *, Scalar alpha=1)",
15333:   }, /*traceable=*/true);
15334: 
15335:   ParsedArgs<2> parsed_args;
15336:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15337:   if(_r.has_torch_function()) {
15338:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15339:   }
15340:   switch (_r.idx) {
15341:     case 0: {
15342:       // [deprecated] aten::sub_(Tensor(a!) self, Scalar alpha, Tensor other) -> Tensor(a!)
15343: 
15344:       auto dispatch_sub_ = [](const at::Tensor & self, const at::Scalar & alpha, const at::Tensor & other) -> at::Tensor {
15345:         pybind11::gil_scoped_release no_gil;
15346:         return self.sub_(other, alpha);
15347:       };
15348:       return wrap(dispatch_sub_(self, _r.scalar(0), _r.tensor(1)));
15349:     }
15350:     case 1: {
15351:       // aten::sub_.Tensor(Tensor(a!) self, Tensor other, *, Scalar alpha=1) -> Tensor(a!)
15352: 
15353:       auto dispatch_sub_ = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
15354:         pybind11::gil_scoped_release no_gil;
15355:         return self.sub_(other, alpha);
15356:       };
15357:       return wrap(dispatch_sub_(self, _r.tensor(0), _r.scalar(1)));
15358:     }
15359:   }
15360:   Py_RETURN_NONE;
```

- EN: The main execution path in this span is carried by `THPVariable_stft`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_stft`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 15361-15480

```cpp
15361:   END_HANDLE_TH_ERRORS
15362: }
15363: 
15364: \
15365: // subtract
15366: static PyObject * THPVariable_subtract(PyObject* self_, PyObject* args, PyObject* kwargs)
15367: {
15368:   HANDLE_TH_ERRORS
15369:   const Tensor& self = THPVariable_Unpack(self_);
15370:   static PythonArgParser parser({
15371:     "subtract(Tensor other, *, Scalar alpha=1)",
15372:     "subtract(Scalar other, Scalar alpha=1)",
15373:   }, /*traceable=*/true);
15374: 
15375:   ParsedArgs<2> parsed_args;
15376:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15377:   if(_r.has_torch_function()) {
15378:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15379:   }
15380:   switch (_r.idx) {
15381:     case 0: {
15382:       // aten::subtract.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor
15383: 
15384:       auto dispatch_subtract = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
15385:         pybind11::gil_scoped_release no_gil;
15386:         return self.subtract(other, alpha);
15387:       };
15388:       return wrap(dispatch_subtract(self, _r.tensor(0), _r.scalar(1)));
15389:     }
15390:     case 1: {
15391:       // aten::subtract.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> Tensor
15392: 
15393:       auto dispatch_subtract = [](const at::Tensor & self, const at::Scalar & other, const at::Scalar & alpha) -> at::Tensor {
15394:         pybind11::gil_scoped_release no_gil;
15395:         return self.subtract(other, alpha);
15396:       };
15397:       return wrap(dispatch_subtract(self, _r.scalar(0), _r.scalar(1)));
15398:     }
15399:   }
15400:   Py_RETURN_NONE;
15401:   END_HANDLE_TH_ERRORS
15402: }
15403: 
15404: \
15405: // subtract_
15406: static PyObject * THPVariable_subtract_(PyObject* self_, PyObject* args, PyObject* kwargs)
15407: {
15408:   HANDLE_TH_ERRORS
15409:   const Tensor& self = THPVariable_Unpack(self_);
15410:   static PythonArgParser parser({
15411:     "subtract_(Tensor other, *, Scalar alpha=1)",
15412:     "subtract_(Scalar other, Scalar alpha=1)",
15413:   }, /*traceable=*/true);
15414: 
15415:   ParsedArgs<2> parsed_args;
15416:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15417:   if(_r.has_torch_function()) {
15418:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15419:   }
15420:   switch (_r.idx) {
15421:     case 0: {
15422:       // aten::subtract_.Tensor(Tensor(a!) self, Tensor other, *, Scalar alpha=1) -> Tensor(a!)
15423: 
15424:       auto dispatch_subtract_ = [](const at::Tensor & self, const at::Tensor & other, const at::Scalar & alpha) -> at::Tensor {
15425:         pybind11::gil_scoped_release no_gil;
15426:         return self.subtract_(other, alpha);
15427:       };
15428:       return wrap(dispatch_subtract_(self, _r.tensor(0), _r.scalar(1)));
15429:     }
15430:     case 1: {
15431:       // aten::subtract_.Scalar(Tensor(a!) self, Scalar other, Scalar alpha=1) -> Tensor(a!)
15432: 
15433:       auto dispatch_subtract_ = [](const at::Tensor & self, const at::Scalar & other, const at::Scalar & alpha) -> at::Tensor {
15434:         pybind11::gil_scoped_release no_gil;
15435:         return self.subtract_(other, alpha);
15436:       };
15437:       return wrap(dispatch_subtract_(self, _r.scalar(0), _r.scalar(1)));
15438:     }
15439:   }
15440:   Py_RETURN_NONE;
15441:   END_HANDLE_TH_ERRORS
15442: }
15443: 
15444: \
15445: // sum
15446: static PyObject * THPVariable_sum(PyObject* self_, PyObject* args, PyObject* kwargs)
15447: {
15448:   HANDLE_TH_ERRORS
15449:   const Tensor& self = THPVariable_Unpack(self_);
15450:   static PythonArgParser parser({
15451:     "sum(*, ScalarType? dtype=None)",
15452:     "sum(IntArrayRef[1]? dim, bool keepdim=False, *, ScalarType? dtype=None)",
15453:     "sum(DimnameList[1] dim, bool keepdim=False, *, ScalarType? dtype=None)",
15454:   }, /*traceable=*/true);
15455: 
15456:   ParsedArgs<3> parsed_args;
15457:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15458:   if(_r.has_torch_function()) {
15459:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15460:   }
15461:   switch (_r.idx) {
15462:     case 0: {
15463:       // aten::sum(Tensor self, *, ScalarType? dtype=None) -> Tensor
15464: 
15465:       auto dispatch_sum = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
15466:         pybind11::gil_scoped_release no_gil;
15467:         return self.sum(dtype);
15468:       };
15469:       return wrap(dispatch_sum(self, _r.scalartypeOptional(0)));
15470:     }
15471:     case 1: {
15472:       // aten::sum.dim_IntList(Tensor self, int[1]? dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
15473: 
15474:       auto dispatch_sum = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
15475:         pybind11::gil_scoped_release no_gil;
15476:         return self.sum(dim, keepdim, dtype);
15477:       };
15478:       return wrap(dispatch_sum(self, _r.intlistOptional(0), _r.toBool(1), _r.scalartypeOptional(2)));
15479:     }
15480:     case 2: {
```

- EN: The main execution path in this span is carried by `THPVariable_subtract`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_subtract`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15481-15600

```cpp
15481:       // aten::sum.dim_DimnameList(Tensor self, Dimname[1] dim, bool keepdim=False, *, ScalarType? dtype=None) -> Tensor
15482: 
15483:       auto dispatch_sum = [](const at::Tensor & self, at::DimnameList dim, bool keepdim, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
15484:         pybind11::gil_scoped_release no_gil;
15485:         return self.sum(dim, keepdim, dtype);
15486:       };
15487:       return wrap(dispatch_sum(self, _r.dimnamelist(0), _r.toBool(1), _r.scalartypeOptional(2)));
15488:     }
15489:   }
15490:   Py_RETURN_NONE;
15491:   END_HANDLE_TH_ERRORS
15492: }
15493: 
15494: // sum_to_size
15495: static PyObject * THPVariable_sum_to_size(PyObject* self_, PyObject* args, PyObject* kwargs)
15496: {
15497:   HANDLE_TH_ERRORS
15498:   const Tensor& self = THPVariable_Unpack(self_);
15499:   static PythonArgParser parser({
15500:     "sum_to_size(SymIntArrayRef size)",
15501:   }, /*traceable=*/true);
15502: 
15503:   ParsedArgs<1> parsed_args;
15504:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15505:   if(_r.has_torch_function()) {
15506:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15507:   }
15508:   // aten::sum_to_size(Tensor self, SymInt[] size) -> Tensor
15509: 
15510:   auto dispatch_sum_to_size = [](const at::Tensor & self, c10::SymIntArrayRef size) -> at::Tensor {
15511:     pybind11::gil_scoped_release no_gil;
15512:     return self.sum_to_size_symint(size);
15513:   };
15514:   return wrap(dispatch_sum_to_size(self, _r.symintlist(0)));
15515:   Py_RETURN_NONE;
15516:   END_HANDLE_TH_ERRORS
15517: }
15518: 
15519: // svd
15520: static PyObject * THPVariable_svd(PyObject* self_, PyObject* args, PyObject* kwargs)
15521: {
15522:   HANDLE_TH_ERRORS
15523:   static PyTypeObject* NamedTuple = generated::get_svd_structseq();
15524:   const Tensor& self = THPVariable_Unpack(self_);
15525:   static PythonArgParser parser({
15526:     "svd(bool some=True, bool compute_uv=True)",
15527:   }, /*traceable=*/true);
15528: 
15529:   ParsedArgs<2> parsed_args;
15530:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15531:   if(_r.has_torch_function()) {
15532:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15533:   }
15534:   // aten::svd(Tensor self, bool some=True, bool compute_uv=True) -> (Tensor U, Tensor S, Tensor V)
15535: 
15536:   auto dispatch_svd = [](const at::Tensor & self, bool some, bool compute_uv) -> ::std::tuple<at::Tensor,at::Tensor,at::Tensor> {
15537:     pybind11::gil_scoped_release no_gil;
15538:     return self.svd(some, compute_uv);
15539:   };
15540:   return wrap(NamedTuple, dispatch_svd(self, _r.toBool(0), _r.toBool(1)));
15541:   Py_RETURN_NONE;
15542:   END_HANDLE_TH_ERRORS
15543: }
15544: 
15545: // swapaxes
15546: static PyObject * THPVariable_swapaxes(PyObject* self_, PyObject* args, PyObject* kwargs)
15547: {
15548:   HANDLE_TH_ERRORS
15549:   const Tensor& self = THPVariable_Unpack(self_);
15550:   static PythonArgParser parser({
15551:     "swapaxes(int64_t axis0, int64_t axis1)",
15552:   }, /*traceable=*/true);
15553: 
15554:   ParsedArgs<2> parsed_args;
15555:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15556:   if(_r.has_torch_function()) {
15557:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15558:   }
15559:   // aten::swapaxes(Tensor(a) self, int axis0, int axis1) -> Tensor(a)
15560: 
15561:   auto dispatch_swapaxes = [](const at::Tensor & self, int64_t axis0, int64_t axis1) -> at::Tensor {
15562:     pybind11::gil_scoped_release no_gil;
15563:     return self.swapaxes(axis0, axis1);
15564:   };
15565:   return wrap(dispatch_swapaxes(self, _r.toInt64(0), _r.toInt64(1)));
15566:   Py_RETURN_NONE;
15567:   END_HANDLE_TH_ERRORS
15568: }
15569: 
15570: // swapaxes_
15571: static PyObject * THPVariable_swapaxes_(PyObject* self_, PyObject* args, PyObject* kwargs)
15572: {
15573:   HANDLE_TH_ERRORS
15574:   const Tensor& self = THPVariable_Unpack(self_);
15575:   static PythonArgParser parser({
15576:     "swapaxes_(int64_t axis0, int64_t axis1)",
15577:   }, /*traceable=*/true);
15578: 
15579:   ParsedArgs<2> parsed_args;
15580:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15581:   if(_r.has_torch_function()) {
15582:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15583:   }
15584:   // aten::swapaxes_(Tensor(a!) self, int axis0, int axis1) -> Tensor(a!)
15585: 
15586:   auto dispatch_swapaxes_ = [](const at::Tensor & self, int64_t axis0, int64_t axis1) -> at::Tensor {
15587:     pybind11::gil_scoped_release no_gil;
15588:     return self.swapaxes_(axis0, axis1);
15589:   };
15590:   return wrap(dispatch_swapaxes_(self, _r.toInt64(0), _r.toInt64(1)));
15591:   Py_RETURN_NONE;
15592:   END_HANDLE_TH_ERRORS
15593: }
15594: 
15595: // swapdims
15596: static PyObject * THPVariable_swapdims(PyObject* self_, PyObject* args, PyObject* kwargs)
15597: {
15598:   HANDLE_TH_ERRORS
15599:   const Tensor& self = THPVariable_Unpack(self_);
15600:   static PythonArgParser parser({
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_sum_to_size`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_sum_to_size`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15601-15720

```cpp
15601:     "swapdims(int64_t dim0, int64_t dim1)",
15602:   }, /*traceable=*/true);
15603: 
15604:   ParsedArgs<2> parsed_args;
15605:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15606:   if(_r.has_torch_function()) {
15607:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15608:   }
15609:   // aten::swapdims(Tensor(a) self, int dim0, int dim1) -> Tensor(a)
15610: 
15611:   auto dispatch_swapdims = [](const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
15612:     pybind11::gil_scoped_release no_gil;
15613:     return self.swapdims(dim0, dim1);
15614:   };
15615:   return wrap(dispatch_swapdims(self, _r.toInt64(0), _r.toInt64(1)));
15616:   Py_RETURN_NONE;
15617:   END_HANDLE_TH_ERRORS
15618: }
15619: 
15620: // swapdims_
15621: static PyObject * THPVariable_swapdims_(PyObject* self_, PyObject* args, PyObject* kwargs)
15622: {
15623:   HANDLE_TH_ERRORS
15624:   const Tensor& self = THPVariable_Unpack(self_);
15625:   static PythonArgParser parser({
15626:     "swapdims_(int64_t dim0, int64_t dim1)",
15627:   }, /*traceable=*/true);
15628: 
15629:   ParsedArgs<2> parsed_args;
15630:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15631:   if(_r.has_torch_function()) {
15632:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15633:   }
15634:   // aten::swapdims_(Tensor(a!) self, int dim0, int dim1) -> Tensor(a!)
15635: 
15636:   auto dispatch_swapdims_ = [](const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
15637:     pybind11::gil_scoped_release no_gil;
15638:     return self.swapdims_(dim0, dim1);
15639:   };
15640:   return wrap(dispatch_swapdims_(self, _r.toInt64(0), _r.toInt64(1)));
15641:   Py_RETURN_NONE;
15642:   END_HANDLE_TH_ERRORS
15643: }
15644: 
15645: // t
15646: static PyObject * THPVariable_t(PyObject* self_, PyObject* args)
15647: {
15648:   HANDLE_TH_ERRORS
15649:   const Tensor& self = THPVariable_Unpack(self_);
15650:   if (has_torch_function(self_)) {
15651:     return handle_torch_function(self_, "t");
15652:   }
15653:   // aten::t(Tensor(a) self) -> Tensor(a)
15654: 
15655:   auto dispatch_t = [](const at::Tensor & self) -> at::Tensor {
15656:     pybind11::gil_scoped_release no_gil;
15657:     return self.t();
15658:   };
15659:   return wrap(dispatch_t(self));
15660:   END_HANDLE_TH_ERRORS
15661: }
15662: 
15663: // t_
15664: static PyObject * THPVariable_t_(PyObject* self_, PyObject* args)
15665: {
15666:   HANDLE_TH_ERRORS
15667:   const Tensor& self = THPVariable_Unpack(self_);
15668:   if (has_torch_function(self_)) {
15669:     return handle_torch_function(self_, "t_");
15670:   }
15671:   // aten::t_(Tensor(a!) self) -> Tensor(a!)
15672: 
15673:   auto dispatch_t_ = [](const at::Tensor & self) -> at::Tensor {
15674:     pybind11::gil_scoped_release no_gil;
15675:     return self.t_();
15676:   };
15677:   return wrap(dispatch_t_(self));
15678:   END_HANDLE_TH_ERRORS
15679: }
15680: 
15681: // take
15682: static PyObject * THPVariable_take(PyObject* self_, PyObject* args, PyObject* kwargs)
15683: {
15684:   HANDLE_TH_ERRORS
15685:   const Tensor& self = THPVariable_Unpack(self_);
15686:   static PythonArgParser parser({
15687:     "take(Tensor index)",
15688:   }, /*traceable=*/true);
15689: 
15690:   ParsedArgs<1> parsed_args;
15691:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15692:   if(_r.has_torch_function()) {
15693:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15694:   }
15695:   // aten::take(Tensor self, Tensor index) -> Tensor
15696: 
15697:   auto dispatch_take = [](const at::Tensor & self, const at::Tensor & index) -> at::Tensor {
15698:     pybind11::gil_scoped_release no_gil;
15699:     return self.take(index);
15700:   };
15701:   return wrap(dispatch_take(self, _r.tensor(0)));
15702:   Py_RETURN_NONE;
15703:   END_HANDLE_TH_ERRORS
15704: }
15705: 
15706: // take_along_dim
15707: static PyObject * THPVariable_take_along_dim(PyObject* self_, PyObject* args, PyObject* kwargs)
15708: {
15709:   HANDLE_TH_ERRORS
15710:   const Tensor& self = THPVariable_Unpack(self_);
15711:   static PythonArgParser parser({
15712:     "take_along_dim(Tensor indices, int64_t? dim=None)",
15713:   }, /*traceable=*/true);
15714: 
15715:   ParsedArgs<2> parsed_args;
15716:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15717:   if(_r.has_torch_function()) {
15718:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15719:   }
15720:   // aten::take_along_dim(Tensor self, Tensor indices, int? dim=None) -> Tensor
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `swapdims`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `swapdims`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 15721-15840

```cpp
15721: 
15722:   auto dispatch_take_along_dim = [](const at::Tensor & self, const at::Tensor & indices, ::std::optional<int64_t> dim) -> at::Tensor {
15723:     pybind11::gil_scoped_release no_gil;
15724:     return self.take_along_dim(indices, dim);
15725:   };
15726:   return wrap(dispatch_take_along_dim(self, _r.tensor(0), _r.toInt64Optional(1)));
15727:   Py_RETURN_NONE;
15728:   END_HANDLE_TH_ERRORS
15729: }
15730: 
15731: // tan
15732: static PyObject * THPVariable_tan(PyObject* self_, PyObject* args)
15733: {
15734:   HANDLE_TH_ERRORS
15735:   const Tensor& self = THPVariable_Unpack(self_);
15736:   if (has_torch_function(self_)) {
15737:     return handle_torch_function(self_, "tan");
15738:   }
15739:   // aten::tan(Tensor self) -> Tensor
15740: 
15741:   auto dispatch_tan = [](const at::Tensor & self) -> at::Tensor {
15742:     pybind11::gil_scoped_release no_gil;
15743:     return self.tan();
15744:   };
15745:   return wrap(dispatch_tan(self));
15746:   END_HANDLE_TH_ERRORS
15747: }
15748: 
15749: // tan_
15750: static PyObject * THPVariable_tan_(PyObject* self_, PyObject* args)
15751: {
15752:   HANDLE_TH_ERRORS
15753:   const Tensor& self = THPVariable_Unpack(self_);
15754:   if (has_torch_function(self_)) {
15755:     return handle_torch_function(self_, "tan_");
15756:   }
15757:   // aten::tan_(Tensor(a!) self) -> Tensor(a!)
15758: 
15759:   auto dispatch_tan_ = [](const at::Tensor & self) -> at::Tensor {
15760:     pybind11::gil_scoped_release no_gil;
15761:     return self.tan_();
15762:   };
15763:   return wrap(dispatch_tan_(self));
15764:   END_HANDLE_TH_ERRORS
15765: }
15766: 
15767: // tanh
15768: static PyObject * THPVariable_tanh(PyObject* self_, PyObject* args)
15769: {
15770:   HANDLE_TH_ERRORS
15771:   const Tensor& self = THPVariable_Unpack(self_);
15772:   if (has_torch_function(self_)) {
15773:     return handle_torch_function(self_, "tanh");
15774:   }
15775:   // aten::tanh(Tensor self) -> Tensor
15776: 
15777:   auto dispatch_tanh = [](const at::Tensor & self) -> at::Tensor {
15778:     pybind11::gil_scoped_release no_gil;
15779:     return self.tanh();
15780:   };
15781:   return wrap(dispatch_tanh(self));
15782:   END_HANDLE_TH_ERRORS
15783: }
15784: 
15785: // tanh_
15786: static PyObject * THPVariable_tanh_(PyObject* self_, PyObject* args)
15787: {
15788:   HANDLE_TH_ERRORS
15789:   const Tensor& self = THPVariable_Unpack(self_);
15790:   if (has_torch_function(self_)) {
15791:     return handle_torch_function(self_, "tanh_");
15792:   }
15793:   // aten::tanh_(Tensor(a!) self) -> Tensor(a!)
15794: 
15795:   auto dispatch_tanh_ = [](const at::Tensor & self) -> at::Tensor {
15796:     pybind11::gil_scoped_release no_gil;
15797:     return self.tanh_();
15798:   };
15799:   return wrap(dispatch_tanh_(self));
15800:   END_HANDLE_TH_ERRORS
15801: }
15802: 
15803: \
15804: // tensor_split
15805: static PyObject * THPVariable_tensor_split(PyObject* self_, PyObject* args, PyObject* kwargs)
15806: {
15807:   HANDLE_TH_ERRORS
15808:   const Tensor& self = THPVariable_Unpack(self_);
15809:   static PythonArgParser parser({
15810:     "tensor_split(SymIntArrayRef indices, int64_t dim=0)",
15811:     "tensor_split(Tensor tensor_indices_or_sections, int64_t dim=0)",
15812:     "tensor_split(SymInt sections, int64_t dim=0)",
15813:   }, /*traceable=*/true);
15814: 
15815:   ParsedArgs<2> parsed_args;
15816:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15817:   if(_r.has_torch_function()) {
15818:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15819:   }
15820:   switch (_r.idx) {
15821:     case 0: {
15822:       // aten::tensor_split.indices(Tensor(a -> *) self, SymInt[] indices, int dim=0) -> Tensor(a)[]
15823: 
15824:       auto dispatch_tensor_split = [](const at::Tensor & self, c10::SymIntArrayRef indices, int64_t dim) -> ::std::vector<at::Tensor> {
15825:         pybind11::gil_scoped_release no_gil;
15826:         return self.tensor_split_symint(indices, dim);
15827:       };
15828:       return wrap(dispatch_tensor_split(self, _r.symintlist(0), _r.toInt64(1)));
15829:     }
15830:     case 1: {
15831:       // aten::tensor_split.tensor_indices_or_sections(Tensor(a -> *) self, Tensor tensor_indices_or_sections, int dim=0) -> Tensor(a)[]
15832: 
15833:       auto dispatch_tensor_split = [](const at::Tensor & self, const at::Tensor & tensor_indices_or_sections, int64_t dim) -> ::std::vector<at::Tensor> {
15834:         pybind11::gil_scoped_release no_gil;
15835:         return self.tensor_split(tensor_indices_or_sections, dim);
15836:       };
15837:       return wrap(dispatch_tensor_split(self, _r.tensor(0), _r.toInt64(1)));
15838:     }
15839:     case 2: {
15840:       // aten::tensor_split.sections(Tensor(a -> *) self, SymInt sections, int dim=0) -> Tensor(a)[]
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_tan`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_tan`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15841-15960

```cpp
15841: 
15842:       auto dispatch_tensor_split = [](const at::Tensor & self, c10::SymInt sections, int64_t dim) -> ::std::vector<at::Tensor> {
15843:         pybind11::gil_scoped_release no_gil;
15844:         return self.tensor_split_symint(sections, dim);
15845:       };
15846:       return wrap(dispatch_tensor_split(self, _r.toSymInt(0), _r.toInt64(1)));
15847:     }
15848:   }
15849:   Py_RETURN_NONE;
15850:   END_HANDLE_TH_ERRORS
15851: }
15852: 
15853: // tile
15854: static PyObject * THPVariable_tile(PyObject* self_, PyObject* args, PyObject* kwargs)
15855: {
15856:   HANDLE_TH_ERRORS
15857:   const Tensor& self = THPVariable_Unpack(self_);
15858:   static PythonArgParser parser({
15859:     "tile(SymIntArrayRef dims)",
15860:   }, /*traceable=*/true);
15861: 
15862:   ParsedArgs<1> parsed_args;
15863:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15864:   if(_r.has_torch_function()) {
15865:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15866:   }
15867:   // aten::tile(Tensor self, SymInt[] dims) -> Tensor
15868: 
15869:   auto dispatch_tile = [](const at::Tensor & self, c10::SymIntArrayRef dims) -> at::Tensor {
15870:     pybind11::gil_scoped_release no_gil;
15871:     return self.tile_symint(dims);
15872:   };
15873:   return wrap(dispatch_tile(self, _r.symintlist(0)));
15874:   Py_RETURN_NONE;
15875:   END_HANDLE_TH_ERRORS
15876: }
15877: 
15878: // to_dense
15879: static PyObject * THPVariable_to_dense(PyObject* self_, PyObject* args, PyObject* kwargs)
15880: {
15881:   HANDLE_TH_ERRORS
15882:   const Tensor& self = THPVariable_Unpack(self_);
15883:   static PythonArgParser parser({
15884:     "to_dense(ScalarType? dtype=None, *, bool? masked_grad=None)",
15885:   }, /*traceable=*/true);
15886: 
15887:   ParsedArgs<2> parsed_args;
15888:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15889:   if(_r.has_torch_function()) {
15890:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15891:   }
15892:   // aten::to_dense(Tensor self, ScalarType? dtype=None, *, bool? masked_grad=None) -> Tensor
15893: 
15894:   auto dispatch_to_dense = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype, ::std::optional<bool> masked_grad) -> at::Tensor {
15895:     pybind11::gil_scoped_release no_gil;
15896:     return self.to_dense(dtype, masked_grad);
15897:   };
15898:   return wrap(dispatch_to_dense(self, _r.scalartypeOptional(0), _r.toBoolOptional(1)));
15899:   Py_RETURN_NONE;
15900:   END_HANDLE_TH_ERRORS
15901: }
15902: 
15903: // to_mkldnn
15904: static PyObject * THPVariable_to_mkldnn(PyObject* self_, PyObject* args, PyObject* kwargs)
15905: {
15906:   HANDLE_TH_ERRORS
15907:   const Tensor& self = THPVariable_Unpack(self_);
15908:   static PythonArgParser parser({
15909:     "to_mkldnn(ScalarType? dtype=None)",
15910:   }, /*traceable=*/true);
15911: 
15912:   ParsedArgs<1> parsed_args;
15913:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15914:   if(_r.has_torch_function()) {
15915:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15916:   }
15917:   // aten::to_mkldnn(Tensor self, ScalarType? dtype=None) -> Tensor
15918: 
15919:   auto dispatch_to_mkldnn = [](const at::Tensor & self, ::std::optional<at::ScalarType> dtype) -> at::Tensor {
15920:     pybind11::gil_scoped_release no_gil;
15921:     return self.to_mkldnn(dtype);
15922:   };
15923:   return wrap(dispatch_to_mkldnn(self, _r.scalartypeOptional(0)));
15924:   Py_RETURN_NONE;
15925:   END_HANDLE_TH_ERRORS
15926: }
15927: 
15928: // to_padded_tensor
15929: static PyObject * THPVariable_to_padded_tensor(PyObject* self_, PyObject* args, PyObject* kwargs)
15930: {
15931:   HANDLE_TH_ERRORS
15932:   const Tensor& self = THPVariable_Unpack(self_);
15933:   static PythonArgParser parser({
15934:     "to_padded_tensor(double padding, SymIntArrayRef? output_size=None)",
15935:   }, /*traceable=*/true);
15936: 
15937:   ParsedArgs<2> parsed_args;
15938:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15939:   if(_r.has_torch_function()) {
15940:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15941:   }
15942:   // aten::to_padded_tensor(Tensor self, float padding, SymInt[]? output_size=None) -> Tensor
15943: 
15944:   auto dispatch_to_padded_tensor = [](const at::Tensor & self, double padding, at::OptionalSymIntArrayRef output_size) -> at::Tensor {
15945:     pybind11::gil_scoped_release no_gil;
15946:     return self.to_padded_tensor_symint(padding, output_size);
15947:   };
15948:   return wrap(dispatch_to_padded_tensor(self, _r.toDouble(0), _r.symintlistOptional(1)));
15949:   Py_RETURN_NONE;
15950:   END_HANDLE_TH_ERRORS
15951: }
15952: 
15953: \
15954: // to_sparse
15955: static PyObject * THPVariable_to_sparse(PyObject* self_, PyObject* args, PyObject* kwargs)
15956: {
15957:   HANDLE_TH_ERRORS
15958:   const Tensor& self = THPVariable_Unpack(self_);
15959:   static PythonArgParser parser({
15960:     "to_sparse(*, Layout? layout=None, IntArrayRef[2]? blocksize=None, int64_t? dense_dim=None)",
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_tile`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_tile`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 15961-16080

```cpp
15961:     "to_sparse(int64_t sparse_dim)",
15962:   }, /*traceable=*/true);
15963: 
15964:   ParsedArgs<3> parsed_args;
15965:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
15966:   if(_r.has_torch_function()) {
15967:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
15968:   }
15969:   switch (_r.idx) {
15970:     case 0: {
15971:       // aten::to_sparse(Tensor self, *, Layout? layout=None, int[2]? blocksize=None, int? dense_dim=None) -> Tensor
15972: 
15973:       auto dispatch_to_sparse = [](const at::Tensor & self, ::std::optional<at::Layout> layout, at::OptionalIntArrayRef blocksize, ::std::optional<int64_t> dense_dim) -> at::Tensor {
15974:         pybind11::gil_scoped_release no_gil;
15975:         return self.to_sparse(layout, blocksize, dense_dim);
15976:       };
15977:       return wrap(dispatch_to_sparse(self, _r.layoutOptional(0), _r.intlistOptional(1), _r.toInt64Optional(2)));
15978:     }
15979:     case 1: {
15980:       // aten::to_sparse.sparse_dim(Tensor self, int sparse_dim) -> Tensor
15981: 
15982:       auto dispatch_to_sparse = [](const at::Tensor & self, int64_t sparse_dim) -> at::Tensor {
15983:         pybind11::gil_scoped_release no_gil;
15984:         return self.to_sparse(sparse_dim);
15985:       };
15986:       return wrap(dispatch_to_sparse(self, _r.toInt64(0)));
15987:     }
15988:   }
15989:   Py_RETURN_NONE;
15990:   END_HANDLE_TH_ERRORS
15991: }
15992: 
15993: // to_sparse_bsc
15994: static PyObject * THPVariable_to_sparse_bsc(PyObject* self_, PyObject* args, PyObject* kwargs)
15995: {
15996:   HANDLE_TH_ERRORS
15997:   const Tensor& self = THPVariable_Unpack(self_);
15998:   static PythonArgParser parser({
15999:     "to_sparse_bsc(IntArrayRef[2] blocksize, int64_t? dense_dim=None)",
16000:   }, /*traceable=*/true);
16001: 
16002:   ParsedArgs<2> parsed_args;
16003:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16004:   if(_r.has_torch_function()) {
16005:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16006:   }
16007:   // aten::to_sparse_bsc(Tensor self, int[2] blocksize, int? dense_dim=None) -> Tensor
16008: 
16009:   auto dispatch_to_sparse_bsc = [](const at::Tensor & self, at::IntArrayRef blocksize, ::std::optional<int64_t> dense_dim) -> at::Tensor {
16010:     pybind11::gil_scoped_release no_gil;
16011:     return self.to_sparse_bsc(blocksize, dense_dim);
16012:   };
16013:   return wrap(dispatch_to_sparse_bsc(self, _r.intlist(0), _r.toInt64Optional(1)));
16014:   Py_RETURN_NONE;
16015:   END_HANDLE_TH_ERRORS
16016: }
16017: 
16018: // to_sparse_bsr
16019: static PyObject * THPVariable_to_sparse_bsr(PyObject* self_, PyObject* args, PyObject* kwargs)
16020: {
16021:   HANDLE_TH_ERRORS
16022:   const Tensor& self = THPVariable_Unpack(self_);
16023:   static PythonArgParser parser({
16024:     "to_sparse_bsr(IntArrayRef[2] blocksize, int64_t? dense_dim=None)",
16025:   }, /*traceable=*/true);
16026: 
16027:   ParsedArgs<2> parsed_args;
16028:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16029:   if(_r.has_torch_function()) {
16030:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16031:   }
16032:   // aten::to_sparse_bsr(Tensor self, int[2] blocksize, int? dense_dim=None) -> Tensor
16033: 
16034:   auto dispatch_to_sparse_bsr = [](const at::Tensor & self, at::IntArrayRef blocksize, ::std::optional<int64_t> dense_dim) -> at::Tensor {
16035:     pybind11::gil_scoped_release no_gil;
16036:     return self.to_sparse_bsr(blocksize, dense_dim);
16037:   };
16038:   return wrap(dispatch_to_sparse_bsr(self, _r.intlist(0), _r.toInt64Optional(1)));
16039:   Py_RETURN_NONE;
16040:   END_HANDLE_TH_ERRORS
16041: }
16042: 
16043: // to_sparse_csc
16044: static PyObject * THPVariable_to_sparse_csc(PyObject* self_, PyObject* args, PyObject* kwargs)
16045: {
16046:   HANDLE_TH_ERRORS
16047:   const Tensor& self = THPVariable_Unpack(self_);
16048:   static PythonArgParser parser({
16049:     "to_sparse_csc(int64_t? dense_dim=None)",
16050:   }, /*traceable=*/true);
16051: 
16052:   ParsedArgs<1> parsed_args;
16053:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16054:   if(_r.has_torch_function()) {
16055:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16056:   }
16057:   // aten::to_sparse_csc(Tensor self, int? dense_dim=None) -> Tensor
16058: 
16059:   auto dispatch_to_sparse_csc = [](const at::Tensor & self, ::std::optional<int64_t> dense_dim) -> at::Tensor {
16060:     pybind11::gil_scoped_release no_gil;
16061:     return self.to_sparse_csc(dense_dim);
16062:   };
16063:   return wrap(dispatch_to_sparse_csc(self, _r.toInt64Optional(0)));
16064:   Py_RETURN_NONE;
16065:   END_HANDLE_TH_ERRORS
16066: }
16067: 
16068: // to_sparse_csr
16069: static PyObject * THPVariable_to_sparse_csr(PyObject* self_, PyObject* args, PyObject* kwargs)
16070: {
16071:   HANDLE_TH_ERRORS
16072:   const Tensor& self = THPVariable_Unpack(self_);
16073:   static PythonArgParser parser({
16074:     "to_sparse_csr(int64_t? dense_dim=None)",
16075:   }, /*traceable=*/true);
16076: 
16077:   ParsedArgs<1> parsed_args;
16078:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16079:   if(_r.has_torch_function()) {
16080:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `to_sparse`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `to_sparse`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 16081-16200

```cpp
16081:   }
16082:   // aten::to_sparse_csr(Tensor self, int? dense_dim=None) -> Tensor
16083: 
16084:   auto dispatch_to_sparse_csr = [](const at::Tensor & self, ::std::optional<int64_t> dense_dim) -> at::Tensor {
16085:     pybind11::gil_scoped_release no_gil;
16086:     return self.to_sparse_csr(dense_dim);
16087:   };
16088:   return wrap(dispatch_to_sparse_csr(self, _r.toInt64Optional(0)));
16089:   Py_RETURN_NONE;
16090:   END_HANDLE_TH_ERRORS
16091: }
16092: 
16093: // topk
16094: static PyObject * THPVariable_topk(PyObject* self_, PyObject* args, PyObject* kwargs)
16095: {
16096:   HANDLE_TH_ERRORS
16097:   static PyTypeObject* NamedTuple = generated::get_topk_structseq();
16098:   const Tensor& self = THPVariable_Unpack(self_);
16099:   static PythonArgParser parser({
16100:     "topk(SymInt k, int64_t dim=-1, bool largest=True, bool sorted=True)",
16101:   }, /*traceable=*/true);
16102: 
16103:   ParsedArgs<4> parsed_args;
16104:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16105:   if(_r.has_torch_function()) {
16106:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16107:   }
16108:   // aten::topk(Tensor self, SymInt k, int dim=-1, bool largest=True, bool sorted=True) -> (Tensor values, Tensor indices)
16109: 
16110:   auto dispatch_topk = [](const at::Tensor & self, c10::SymInt k, int64_t dim, bool largest, bool sorted) -> ::std::tuple<at::Tensor,at::Tensor> {
16111:     pybind11::gil_scoped_release no_gil;
16112:     return self.topk_symint(k, dim, largest, sorted);
16113:   };
16114:   return wrap(NamedTuple, dispatch_topk(self, _r.toSymInt(0), _r.toInt64(1), _r.toBool(2), _r.toBool(3)));
16115:   Py_RETURN_NONE;
16116:   END_HANDLE_TH_ERRORS
16117: }
16118: 
16119: // trace
16120: static PyObject * THPVariable_trace(PyObject* self_, PyObject* args)
16121: {
16122:   HANDLE_TH_ERRORS
16123:   const Tensor& self = THPVariable_Unpack(self_);
16124:   if (has_torch_function(self_)) {
16125:     return handle_torch_function(self_, "trace");
16126:   }
16127:   // aten::trace(Tensor self) -> Tensor
16128: 
16129:   auto dispatch_trace = [](const at::Tensor & self) -> at::Tensor {
16130:     pybind11::gil_scoped_release no_gil;
16131:     return self.trace();
16132:   };
16133:   return wrap(dispatch_trace(self));
16134:   END_HANDLE_TH_ERRORS
16135: }
16136: 
16137: \
16138: // transpose
16139: static PyObject * THPVariable_transpose(PyObject* self_, PyObject* args, PyObject* kwargs)
16140: {
16141:   HANDLE_TH_ERRORS
16142:   const Tensor& self = THPVariable_Unpack(self_);
16143:   static PythonArgParser parser({
16144:     "transpose(int64_t dim0, int64_t dim1)",
16145:     "transpose(Dimname dim0, Dimname dim1)",
16146:   }, /*traceable=*/true);
16147: 
16148:   ParsedArgs<2> parsed_args;
16149:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16150:   if(_r.has_torch_function()) {
16151:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16152:   }
16153:   switch (_r.idx) {
16154:     case 0: {
16155:       // aten::transpose.int(Tensor(a) self, int dim0, int dim1) -> Tensor(a)
16156: 
16157:       auto dispatch_transpose = [](const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
16158:         pybind11::gil_scoped_release no_gil;
16159:         return self.transpose(dim0, dim1);
16160:       };
16161:       return wrap(dispatch_transpose(self, _r.toInt64(0), _r.toInt64(1)));
16162:     }
16163:     case 1: {
16164:       // aten::transpose.Dimname(Tensor(a) self, Dimname dim0, Dimname dim1) -> Tensor(a)
16165: 
16166:       auto dispatch_transpose = [](const at::Tensor & self, at::Dimname dim0, at::Dimname dim1) -> at::Tensor {
16167:         pybind11::gil_scoped_release no_gil;
16168:         return self.transpose(dim0, dim1);
16169:       };
16170:       return wrap(dispatch_transpose(self, _r.dimname(0), _r.dimname(1)));
16171:     }
16172:   }
16173:   Py_RETURN_NONE;
16174:   END_HANDLE_TH_ERRORS
16175: }
16176: 
16177: // transpose_
16178: static PyObject * THPVariable_transpose_(PyObject* self_, PyObject* args, PyObject* kwargs)
16179: {
16180:   HANDLE_TH_ERRORS
16181:   const Tensor& self = THPVariable_Unpack(self_);
16182:   static PythonArgParser parser({
16183:     "transpose_(int64_t dim0, int64_t dim1)",
16184:   }, /*traceable=*/true);
16185: 
16186:   ParsedArgs<2> parsed_args;
16187:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16188:   if(_r.has_torch_function()) {
16189:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16190:   }
16191:   // aten::transpose_(Tensor(a!) self, int dim0, int dim1) -> Tensor(a!)
16192: 
16193:   auto dispatch_transpose_ = [](const at::Tensor & self, int64_t dim0, int64_t dim1) -> at::Tensor {
16194:     pybind11::gil_scoped_release no_gil;
16195:     return self.transpose_(dim0, dim1);
16196:   };
16197:   return wrap(dispatch_transpose_(self, _r.toInt64(0), _r.toInt64(1)));
16198:   Py_RETURN_NONE;
16199:   END_HANDLE_TH_ERRORS
16200: }
```

- EN: The main execution path in this span is carried by `to_sparse_csr`, `wrap`, `THPVariable_topk`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `to_sparse_csr`, `wrap`, `THPVariable_topk` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 16201-16320

```cpp
16201: 
16202: // triangular_solve
16203: static PyObject * THPVariable_triangular_solve(PyObject* self_, PyObject* args, PyObject* kwargs)
16204: {
16205:   HANDLE_TH_ERRORS
16206:   static PyTypeObject* NamedTuple = generated::get_triangular_solve_structseq();
16207:   const Tensor& self = THPVariable_Unpack(self_);
16208:   static PythonArgParser parser({
16209:     "triangular_solve(Tensor A, bool upper=True, bool transpose=False, bool unitriangular=False)",
16210:   }, /*traceable=*/true);
16211: 
16212:   ParsedArgs<4> parsed_args;
16213:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16214:   if(_r.has_torch_function()) {
16215:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16216:   }
16217:   // aten::triangular_solve(Tensor self, Tensor A, bool upper=True, bool transpose=False, bool unitriangular=False) -> (Tensor solution, Tensor cloned_coefficient)
16218: 
16219:   auto dispatch_triangular_solve = [](const at::Tensor & self, const at::Tensor & A, bool upper, bool transpose, bool unitriangular) -> ::std::tuple<at::Tensor,at::Tensor> {
16220:     pybind11::gil_scoped_release no_gil;
16221:     return self.triangular_solve(A, upper, transpose, unitriangular);
16222:   };
16223:   return wrap(NamedTuple, dispatch_triangular_solve(self, _r.tensor(0), _r.toBool(1), _r.toBool(2), _r.toBool(3)));
16224:   Py_RETURN_NONE;
16225:   END_HANDLE_TH_ERRORS
16226: }
16227: 
16228: // tril
16229: static PyObject * THPVariable_tril(PyObject* self_, PyObject* args, PyObject* kwargs)
16230: {
16231:   HANDLE_TH_ERRORS
16232:   const Tensor& self = THPVariable_Unpack(self_);
16233:   static PythonArgParser parser({
16234:     "tril(SymInt diagonal=0)",
16235:   }, /*traceable=*/true);
16236: 
16237:   ParsedArgs<1> parsed_args;
16238:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16239:   if(_r.has_torch_function()) {
16240:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16241:   }
16242:   // aten::tril(Tensor self, SymInt diagonal=0) -> Tensor
16243: 
16244:   auto dispatch_tril = [](const at::Tensor & self, c10::SymInt diagonal) -> at::Tensor {
16245:     pybind11::gil_scoped_release no_gil;
16246:     return self.tril_symint(diagonal);
16247:   };
16248:   return wrap(dispatch_tril(self, _r.toSymInt(0)));
16249:   Py_RETURN_NONE;
16250:   END_HANDLE_TH_ERRORS
16251: }
16252: 
16253: // tril_
16254: static PyObject * THPVariable_tril_(PyObject* self_, PyObject* args, PyObject* kwargs)
16255: {
16256:   HANDLE_TH_ERRORS
16257:   const Tensor& self = THPVariable_Unpack(self_);
16258:   static PythonArgParser parser({
16259:     "tril_(SymInt diagonal=0)",
16260:   }, /*traceable=*/true);
16261: 
16262:   ParsedArgs<1> parsed_args;
16263:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16264:   if(_r.has_torch_function()) {
16265:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16266:   }
16267:   // aten::tril_(Tensor(a!) self, SymInt diagonal=0) -> Tensor(a!)
16268: 
16269:   auto dispatch_tril_ = [](const at::Tensor & self, c10::SymInt diagonal) -> at::Tensor {
16270:     pybind11::gil_scoped_release no_gil;
16271:     return self.tril__symint(diagonal);
16272:   };
16273:   return wrap(dispatch_tril_(self, _r.toSymInt(0)));
16274:   Py_RETURN_NONE;
16275:   END_HANDLE_TH_ERRORS
16276: }
16277: 
16278: // triu
16279: static PyObject * THPVariable_triu(PyObject* self_, PyObject* args, PyObject* kwargs)
16280: {
16281:   HANDLE_TH_ERRORS
16282:   const Tensor& self = THPVariable_Unpack(self_);
16283:   static PythonArgParser parser({
16284:     "triu(SymInt diagonal=0)",
16285:   }, /*traceable=*/true);
16286: 
16287:   ParsedArgs<1> parsed_args;
16288:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16289:   if(_r.has_torch_function()) {
16290:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16291:   }
16292:   // aten::triu(Tensor self, SymInt diagonal=0) -> Tensor
16293: 
16294:   auto dispatch_triu = [](const at::Tensor & self, c10::SymInt diagonal) -> at::Tensor {
16295:     pybind11::gil_scoped_release no_gil;
16296:     return self.triu_symint(diagonal);
16297:   };
16298:   return wrap(dispatch_triu(self, _r.toSymInt(0)));
16299:   Py_RETURN_NONE;
16300:   END_HANDLE_TH_ERRORS
16301: }
16302: 
16303: // triu_
16304: static PyObject * THPVariable_triu_(PyObject* self_, PyObject* args, PyObject* kwargs)
16305: {
16306:   HANDLE_TH_ERRORS
16307:   const Tensor& self = THPVariable_Unpack(self_);
16308:   static PythonArgParser parser({
16309:     "triu_(SymInt diagonal=0)",
16310:   }, /*traceable=*/true);
16311: 
16312:   ParsedArgs<1> parsed_args;
16313:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16314:   if(_r.has_torch_function()) {
16315:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16316:   }
16317:   // aten::triu_(Tensor(a!) self, SymInt diagonal=0) -> Tensor(a!)
16318: 
16319:   auto dispatch_triu_ = [](const at::Tensor & self, c10::SymInt diagonal) -> at::Tensor {
16320:     pybind11::gil_scoped_release no_gil;
```

- EN: The main execution path in this span is carried by `THPVariable_triangular_solve`, `get_triangular_solve_structseq`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `THPVariable_triangular_solve`, `get_triangular_solve_structseq`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 16321-16440

```cpp
16321:     return self.triu__symint(diagonal);
16322:   };
16323:   return wrap(dispatch_triu_(self, _r.toSymInt(0)));
16324:   Py_RETURN_NONE;
16325:   END_HANDLE_TH_ERRORS
16326: }
16327: 
16328: \
16329: // true_divide
16330: static PyObject * THPVariable_true_divide(PyObject* self_, PyObject* args, PyObject* kwargs)
16331: {
16332:   HANDLE_TH_ERRORS
16333:   const Tensor& self = THPVariable_Unpack(self_);
16334:   static PythonArgParser parser({
16335:     "true_divide(Tensor other)",
16336:     "true_divide(Scalar other)",
16337:   }, /*traceable=*/true);
16338: 
16339:   ParsedArgs<1> parsed_args;
16340:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16341:   if(_r.has_torch_function()) {
16342:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16343:   }
16344:   switch (_r.idx) {
16345:     case 0: {
16346:       // aten::true_divide.Tensor(Tensor self, Tensor other) -> Tensor
16347: 
16348:       auto dispatch_true_divide = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
16349:         pybind11::gil_scoped_release no_gil;
16350:         return self.true_divide(other);
16351:       };
16352:       return wrap(dispatch_true_divide(self, _r.tensor(0)));
16353:     }
16354:     case 1: {
16355:       // aten::true_divide.Scalar(Tensor self, Scalar other) -> Tensor
16356: 
16357:       auto dispatch_true_divide = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
16358:         pybind11::gil_scoped_release no_gil;
16359:         return self.true_divide(other);
16360:       };
16361:       return wrap(dispatch_true_divide(self, _r.scalar(0)));
16362:     }
16363:   }
16364:   Py_RETURN_NONE;
16365:   END_HANDLE_TH_ERRORS
16366: }
16367: 
16368: \
16369: // true_divide_
16370: static PyObject * THPVariable_true_divide_(PyObject* self_, PyObject* args, PyObject* kwargs)
16371: {
16372:   HANDLE_TH_ERRORS
16373:   const Tensor& self = THPVariable_Unpack(self_);
16374:   static PythonArgParser parser({
16375:     "true_divide_(Tensor other)",
16376:     "true_divide_(Scalar other)",
16377:   }, /*traceable=*/true);
16378: 
16379:   ParsedArgs<1> parsed_args;
16380:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16381:   if(_r.has_torch_function()) {
16382:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16383:   }
16384:   switch (_r.idx) {
16385:     case 0: {
16386:       // aten::true_divide_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
16387: 
16388:       auto dispatch_true_divide_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
16389:         pybind11::gil_scoped_release no_gil;
16390:         return self.true_divide_(other);
16391:       };
16392:       return wrap(dispatch_true_divide_(self, _r.tensor(0)));
16393:     }
16394:     case 1: {
16395:       // aten::true_divide_.Scalar(Tensor(a!) self, Scalar other) -> Tensor(a!)
16396: 
16397:       auto dispatch_true_divide_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
16398:         pybind11::gil_scoped_release no_gil;
16399:         return self.true_divide_(other);
16400:       };
16401:       return wrap(dispatch_true_divide_(self, _r.scalar(0)));
16402:     }
16403:   }
16404:   Py_RETURN_NONE;
16405:   END_HANDLE_TH_ERRORS
16406: }
16407: 
16408: // trunc
16409: static PyObject * THPVariable_trunc(PyObject* self_, PyObject* args)
16410: {
16411:   HANDLE_TH_ERRORS
16412:   const Tensor& self = THPVariable_Unpack(self_);
16413:   if (has_torch_function(self_)) {
16414:     return handle_torch_function(self_, "trunc");
16415:   }
16416:   // aten::trunc(Tensor self) -> Tensor
16417: 
16418:   auto dispatch_trunc = [](const at::Tensor & self) -> at::Tensor {
16419:     pybind11::gil_scoped_release no_gil;
16420:     return self.trunc();
16421:   };
16422:   return wrap(dispatch_trunc(self));
16423:   END_HANDLE_TH_ERRORS
16424: }
16425: 
16426: // trunc_
16427: static PyObject * THPVariable_trunc_(PyObject* self_, PyObject* args)
16428: {
16429:   HANDLE_TH_ERRORS
16430:   const Tensor& self = THPVariable_Unpack(self_);
16431:   if (has_torch_function(self_)) {
16432:     return handle_torch_function(self_, "trunc_");
16433:   }
16434:   // aten::trunc_(Tensor(a!) self) -> Tensor(a!)
16435: 
16436:   auto dispatch_trunc_ = [](const at::Tensor & self) -> at::Tensor {
16437:     pybind11::gil_scoped_release no_gil;
16438:     return self.trunc_();
16439:   };
16440:   return wrap(dispatch_trunc_(self));
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_true_divide`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results. The block finishes by returning a value or delegating work to the next layer.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_true_divide`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。 该代码块最后会返回一个值，或把工作委托给下一层。
### Lines 16441-16560

```cpp
16441:   END_HANDLE_TH_ERRORS
16442: }
16443: 
16444: // type_as
16445: static PyObject * THPVariable_type_as(PyObject* self_, PyObject* args, PyObject* kwargs)
16446: {
16447:   HANDLE_TH_ERRORS
16448:   const Tensor& self = THPVariable_Unpack(self_);
16449:   static PythonArgParser parser({
16450:     "type_as(Tensor other)",
16451:   }, /*traceable=*/true);
16452: 
16453:   ParsedArgs<1> parsed_args;
16454:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16455:   if(_r.has_torch_function()) {
16456:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16457:   }
16458:   // aten::type_as(Tensor self, Tensor other) -> Tensor
16459: 
16460:   auto dispatch_type_as = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
16461:     pybind11::gil_scoped_release no_gil;
16462:     return self.type_as(other);
16463:   };
16464:   return wrap(dispatch_type_as(self, _r.tensor(0)));
16465:   Py_RETURN_NONE;
16466:   END_HANDLE_TH_ERRORS
16467: }
16468: 
16469: \
16470: // unbind
16471: static PyObject * THPVariable_unbind(PyObject* self_, PyObject* args, PyObject* kwargs)
16472: {
16473:   HANDLE_TH_ERRORS
16474:   const Tensor& self = THPVariable_Unpack(self_);
16475:   static PythonArgParser parser({
16476:     "unbind(int64_t dim=0)",
16477:     "unbind(Dimname dim)",
16478:   }, /*traceable=*/true);
16479: 
16480:   ParsedArgs<1> parsed_args;
16481:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16482:   if(_r.has_torch_function()) {
16483:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16484:   }
16485:   switch (_r.idx) {
16486:     case 0: {
16487:       // aten::unbind.int(Tensor(a -> *) self, int dim=0) -> Tensor(a)[]
16488: 
16489:       auto dispatch_unbind = [](const at::Tensor & self, int64_t dim) -> ::std::vector<at::Tensor> {
16490:         pybind11::gil_scoped_release no_gil;
16491:         return self.unbind(dim);
16492:       };
16493:       return wrap(dispatch_unbind(self, _r.toInt64(0)));
16494:     }
16495:     case 1: {
16496:       // aten::unbind.Dimname(Tensor(a -> *) self, Dimname dim) -> Tensor(a)[]
16497: 
16498:       auto dispatch_unbind = [](const at::Tensor & self, at::Dimname dim) -> ::std::vector<at::Tensor> {
16499:         pybind11::gil_scoped_release no_gil;
16500:         return self.unbind(dim);
16501:       };
16502:       return wrap(dispatch_unbind(self, _r.dimname(0)));
16503:     }
16504:   }
16505:   Py_RETURN_NONE;
16506:   END_HANDLE_TH_ERRORS
16507: }
16508: 
16509: \
16510: // unflatten
16511: static PyObject * THPVariable_unflatten(PyObject* self_, PyObject* args, PyObject* kwargs)
16512: {
16513:   HANDLE_TH_ERRORS
16514:   const Tensor& self = THPVariable_Unpack(self_);
16515:   static PythonArgParser parser({
16516:     "unflatten(Dimname dim, SymIntArrayRef sizes, DimnameList names)",
16517:     "unflatten(int64_t dim, SymIntArrayRef sizes)",
16518:   }, /*traceable=*/true);
16519: 
16520:   ParsedArgs<3> parsed_args;
16521:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16522:   if(_r.has_torch_function()) {
16523:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16524:   }
16525:   switch (_r.idx) {
16526:     case 0: {
16527:       // aten::unflatten.Dimname(Tensor(a) self, Dimname dim, SymInt[] sizes, Dimname[] names) -> Tensor(a)
16528: 
16529:       auto dispatch_unflatten = [](const at::Tensor & self, at::Dimname dim, c10::SymIntArrayRef sizes, at::DimnameList names) -> at::Tensor {
16530:         pybind11::gil_scoped_release no_gil;
16531:         return self.unflatten_symint(dim, sizes, names);
16532:       };
16533:       return wrap(dispatch_unflatten(self, _r.dimname(0), _r.symintlist(1), _r.dimnamelist(2)));
16534:     }
16535:     case 1: {
16536:       // aten::unflatten.int(Tensor(a) self, int dim, SymInt[] sizes) -> Tensor(a)
16537: 
16538:       auto dispatch_unflatten = [](const at::Tensor & self, int64_t dim, c10::SymIntArrayRef sizes) -> at::Tensor {
16539:         pybind11::gil_scoped_release no_gil;
16540:         return self.unflatten_symint(dim, sizes);
16541:       };
16542:       return wrap(dispatch_unflatten(self, _r.toInt64(0), _r.symintlist(1)));
16543:     }
16544:   }
16545:   Py_RETURN_NONE;
16546:   END_HANDLE_TH_ERRORS
16547: }
16548: 
16549: // unfold
16550: static PyObject * THPVariable_unfold(PyObject* self_, PyObject* args, PyObject* kwargs)
16551: {
16552:   HANDLE_TH_ERRORS
16553:   const Tensor& self = THPVariable_Unpack(self_);
16554:   static PythonArgParser parser({
16555:     "unfold(int64_t dimension, int64_t size, int64_t step)",
16556:   }, /*traceable=*/true);
16557: 
16558:   ParsedArgs<3> parsed_args;
16559:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16560:   if(_r.has_torch_function()) {
```

- EN: The main execution path in this span is carried by `THPVariable_type_as`, `THPVariable_Unpack`, `parser`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `THPVariable_type_as`, `THPVariable_Unpack`, `parser` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 16561-16680

```cpp
16561:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16562:   }
16563:   // aten::unfold(Tensor(a) self, int dimension, int size, int step) -> Tensor(a)
16564: 
16565:   auto dispatch_unfold = [](const at::Tensor & self, int64_t dimension, int64_t size, int64_t step) -> at::Tensor {
16566:     pybind11::gil_scoped_release no_gil;
16567:     return self.unfold(dimension, size, step);
16568:   };
16569:   return wrap(dispatch_unfold(self, _r.toInt64(0), _r.toInt64(1), _r.toInt64(2)));
16570:   Py_RETURN_NONE;
16571:   END_HANDLE_TH_ERRORS
16572: }
16573: 
16574: // uniform_
16575: static PyObject * THPVariable_uniform_(PyObject* self_, PyObject* args, PyObject* kwargs)
16576: {
16577:   HANDLE_TH_ERRORS
16578:   const Tensor& self = THPVariable_Unpack(self_);
16579:   static PythonArgParser parser({
16580:     "uniform_(double from=0, double to=1, *, Generator? generator=None)",
16581:   }, /*traceable=*/true);
16582: 
16583:   ParsedArgs<3> parsed_args;
16584:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16585:   if(_r.has_torch_function()) {
16586:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16587:   }
16588:   // aten::uniform_(Tensor(a!) self, float from=0, float to=1, *, Generator? generator=None) -> Tensor(a!)
16589: 
16590:   auto dispatch_uniform_ = [](const at::Tensor & self, double from, double to, ::std::optional<at::Generator> generator) -> at::Tensor {
16591:     pybind11::gil_scoped_release no_gil;
16592:     return self.uniform_(from, to, generator);
16593:   };
16594:   return wrap(dispatch_uniform_(self, _r.toDouble(0), _r.toDouble(1), _r.generator(2)));
16595:   Py_RETURN_NONE;
16596:   END_HANDLE_TH_ERRORS
16597: }
16598: 
16599: // unsafe_chunk
16600: static PyObject * THPVariable_unsafe_chunk(PyObject* self_, PyObject* args, PyObject* kwargs)
16601: {
16602:   HANDLE_TH_ERRORS
16603:   const Tensor& self = THPVariable_Unpack(self_);
16604:   static PythonArgParser parser({
16605:     "unsafe_chunk(int64_t chunks, int64_t dim=0)",
16606:   }, /*traceable=*/true);
16607: 
16608:   ParsedArgs<2> parsed_args;
16609:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16610:   if(_r.has_torch_function()) {
16611:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16612:   }
16613:   // aten::unsafe_chunk(Tensor self, int chunks, int dim=0) -> Tensor[]
16614: 
16615:   auto dispatch_unsafe_chunk = [](const at::Tensor & self, int64_t chunks, int64_t dim) -> ::std::vector<at::Tensor> {
16616:     pybind11::gil_scoped_release no_gil;
16617:     return self.unsafe_chunk(chunks, dim);
16618:   };
16619:   return wrap(dispatch_unsafe_chunk(self, _r.toInt64(0), _r.toInt64(1)));
16620:   Py_RETURN_NONE;
16621:   END_HANDLE_TH_ERRORS
16622: }
16623: 
16624: // unsafe_split
16625: static PyObject * THPVariable_unsafe_split(PyObject* self_, PyObject* args, PyObject* kwargs)
16626: {
16627:   HANDLE_TH_ERRORS
16628:   const Tensor& self = THPVariable_Unpack(self_);
16629:   static PythonArgParser parser({
16630:     "unsafe_split(SymInt split_size, int64_t dim=0)",
16631:   }, /*traceable=*/true);
16632: 
16633:   ParsedArgs<2> parsed_args;
16634:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16635:   if(_r.has_torch_function()) {
16636:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16637:   }
16638:   // aten::unsafe_split.Tensor(Tensor self, SymInt split_size, int dim=0) -> Tensor[]
16639: 
16640:   auto dispatch_unsafe_split = [](const at::Tensor & self, c10::SymInt split_size, int64_t dim) -> ::std::vector<at::Tensor> {
16641:     pybind11::gil_scoped_release no_gil;
16642:     return self.unsafe_split_symint(split_size, dim);
16643:   };
16644:   return wrap(dispatch_unsafe_split(self, _r.toSymInt(0), _r.toInt64(1)));
16645:   Py_RETURN_NONE;
16646:   END_HANDLE_TH_ERRORS
16647: }
16648: 
16649: // unsafe_split_with_sizes
16650: static PyObject * THPVariable_unsafe_split_with_sizes(PyObject* self_, PyObject* args, PyObject* kwargs)
16651: {
16652:   HANDLE_TH_ERRORS
16653:   const Tensor& self = THPVariable_Unpack(self_);
16654:   static PythonArgParser parser({
16655:     "unsafe_split_with_sizes(SymIntArrayRef split_sizes, int64_t dim=0)",
16656:   }, /*traceable=*/true);
16657: 
16658:   ParsedArgs<2> parsed_args;
16659:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16660:   if(_r.has_torch_function()) {
16661:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16662:   }
16663:   // aten::unsafe_split_with_sizes(Tensor self, SymInt[] split_sizes, int dim=0) -> Tensor[]
16664: 
16665:   auto dispatch_unsafe_split_with_sizes = [](const at::Tensor & self, c10::SymIntArrayRef split_sizes, int64_t dim) -> ::std::vector<at::Tensor> {
16666:     pybind11::gil_scoped_release no_gil;
16667:     return self.unsafe_split_with_sizes_symint(split_sizes, dim);
16668:   };
16669:   return wrap(dispatch_unsafe_split_with_sizes(self, _r.symintlist(0), _r.toInt64(1)));
16670:   Py_RETURN_NONE;
16671:   END_HANDLE_TH_ERRORS
16672: }
16673: 
16674: // unsqueeze
16675: static PyObject * THPVariable_unsqueeze(PyObject* self_, PyObject* args, PyObject* kwargs)
16676: {
16677:   HANDLE_TH_ERRORS
16678:   const Tensor& self = THPVariable_Unpack(self_);
16679:   static PythonArgParser parser({
16680:     "unsqueeze(int64_t dim)",
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `unfold`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `unfold`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 16681-16800

```cpp
16681:   }, /*traceable=*/true);
16682: 
16683:   ParsedArgs<1> parsed_args;
16684:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16685:   if(_r.has_torch_function()) {
16686:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16687:   }
16688:   // aten::unsqueeze(Tensor(a) self, int dim) -> Tensor(a)
16689: 
16690:   auto dispatch_unsqueeze = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
16691:     pybind11::gil_scoped_release no_gil;
16692:     return self.unsqueeze(dim);
16693:   };
16694:   return wrap(dispatch_unsqueeze(self, _r.toInt64(0)));
16695:   Py_RETURN_NONE;
16696:   END_HANDLE_TH_ERRORS
16697: }
16698: 
16699: // unsqueeze_
16700: static PyObject * THPVariable_unsqueeze_(PyObject* self_, PyObject* args, PyObject* kwargs)
16701: {
16702:   HANDLE_TH_ERRORS
16703:   const Tensor& self = THPVariable_Unpack(self_);
16704:   static PythonArgParser parser({
16705:     "unsqueeze_(int64_t dim)",
16706:   }, /*traceable=*/true);
16707: 
16708:   ParsedArgs<1> parsed_args;
16709:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16710:   if(_r.has_torch_function()) {
16711:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16712:   }
16713:   // aten::unsqueeze_(Tensor(a!) self, int dim) -> Tensor(a!)
16714: 
16715:   auto dispatch_unsqueeze_ = [](const at::Tensor & self, int64_t dim) -> at::Tensor {
16716:     pybind11::gil_scoped_release no_gil;
16717:     return self.unsqueeze_(dim);
16718:   };
16719:   return wrap(dispatch_unsqueeze_(self, _r.toInt64(0)));
16720:   Py_RETURN_NONE;
16721:   END_HANDLE_TH_ERRORS
16722: }
16723: 
16724: // values
16725: static PyObject * THPVariable_values(PyObject* self_, PyObject* args)
16726: {
16727:   HANDLE_TH_ERRORS
16728:   const Tensor& self = THPVariable_Unpack(self_);
16729:   if (has_torch_function(self_)) {
16730:     return handle_torch_function(self_, "values");
16731:   }
16732:   // aten::values(Tensor(a) self) -> Tensor(a)
16733: 
16734:   auto dispatch_values = [](const at::Tensor & self) -> at::Tensor {
16735:     pybind11::gil_scoped_release no_gil;
16736:     return self.values();
16737:   };
16738:   return wrap(dispatch_values(self));
16739:   END_HANDLE_TH_ERRORS
16740: }
16741: 
16742: \
16743: // var
16744: static PyObject * THPVariable_var(PyObject* self_, PyObject* args, PyObject* kwargs)
16745: {
16746:   HANDLE_TH_ERRORS
16747:   const Tensor& self = THPVariable_Unpack(self_);
16748:   static PythonArgParser parser({
16749:     "var(IntArrayRef[1]? dim, bool unbiased=True, bool keepdim=False)",
16750:     "var(IntArrayRef[1]? dim=None, *, Scalar? correction=None, bool keepdim=False)",
16751:     "var(bool unbiased=True)",
16752:     "var(DimnameList[1] dim, bool unbiased=True, bool keepdim=False)",
16753:     "var(DimnameList[1] dim, *, Scalar? correction=None, bool keepdim=False)",
16754:   }, /*traceable=*/true);
16755: 
16756:   ParsedArgs<3> parsed_args;
16757:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16758:   if(_r.has_torch_function()) {
16759:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16760:   }
16761:   switch (_r.idx) {
16762:     case 0: {
16763:       // aten::var.dim(Tensor self, int[1]? dim, bool unbiased=True, bool keepdim=False) -> Tensor
16764: 
16765:       auto dispatch_var = [](const at::Tensor & self, at::OptionalIntArrayRef dim, bool unbiased, bool keepdim) -> at::Tensor {
16766:         pybind11::gil_scoped_release no_gil;
16767:         return self.var(dim, unbiased, keepdim);
16768:       };
16769:       return wrap(dispatch_var(self, _r.intlistOptional(0), _r.toBool(1), _r.toBool(2)));
16770:     }
16771:     case 1: {
16772:       // aten::var.correction(Tensor self, int[1]? dim=None, *, Scalar? correction=None, bool keepdim=False) -> Tensor
16773: 
16774:       auto dispatch_var = [](const at::Tensor & self, at::OptionalIntArrayRef dim, const ::std::optional<at::Scalar> & correction, bool keepdim) -> at::Tensor {
16775:         pybind11::gil_scoped_release no_gil;
16776:         return self.var(dim, correction, keepdim);
16777:       };
16778:       return wrap(dispatch_var(self, _r.intlistOptional(0), _r.scalarOptional(1), _r.toBool(2)));
16779:     }
16780:     case 2: {
16781:       // aten::var(Tensor self, bool unbiased=True) -> Tensor
16782: 
16783:       auto dispatch_var = [](const at::Tensor & self, bool unbiased) -> at::Tensor {
16784:         pybind11::gil_scoped_release no_gil;
16785:         return self.var(unbiased);
16786:       };
16787:       return wrap(dispatch_var(self, _r.toBool(0)));
16788:     }
16789:     case 3: {
16790:       // aten::var.names_dim(Tensor self, Dimname[1] dim, bool unbiased=True, bool keepdim=False) -> Tensor
16791: 
16792:       auto dispatch_var = [](const at::Tensor & self, at::DimnameList dim, bool unbiased, bool keepdim) -> at::Tensor {
16793:         pybind11::gil_scoped_release no_gil;
16794:         return self.var(dim, unbiased, keepdim);
16795:       };
16796:       return wrap(dispatch_var(self, _r.dimnamelist(0), _r.toBool(1), _r.toBool(2)));
16797:     }
16798:     case 4: {
16799:       // aten::var.correction_names(Tensor self, Dimname[1] dim, *, Scalar? correction=None, bool keepdim=False) -> Tensor
16800: 
```

- EN: The main execution path in this span is carried by `handle_torch_function`, `unsqueeze`, `wrap`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `handle_torch_function`, `unsqueeze`, `wrap` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 16801-16920

```cpp
16801:       auto dispatch_var = [](const at::Tensor & self, at::DimnameList dim, const ::std::optional<at::Scalar> & correction, bool keepdim) -> at::Tensor {
16802:         pybind11::gil_scoped_release no_gil;
16803:         return self.var(dim, correction, keepdim);
16804:       };
16805:       return wrap(dispatch_var(self, _r.dimnamelist(0), _r.scalarOptional(1), _r.toBool(2)));
16806:     }
16807:   }
16808:   Py_RETURN_NONE;
16809:   END_HANDLE_TH_ERRORS
16810: }
16811: 
16812: // vdot
16813: static PyObject * THPVariable_vdot(PyObject* self_, PyObject* args, PyObject* kwargs)
16814: {
16815:   HANDLE_TH_ERRORS
16816:   const Tensor& self = THPVariable_Unpack(self_);
16817:   static PythonArgParser parser({
16818:     "vdot(Tensor other)",
16819:   }, /*traceable=*/true);
16820: 
16821:   ParsedArgs<1> parsed_args;
16822:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16823:   if(_r.has_torch_function()) {
16824:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16825:   }
16826:   // aten::vdot(Tensor self, Tensor other) -> Tensor
16827: 
16828:   auto dispatch_vdot = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
16829:     pybind11::gil_scoped_release no_gil;
16830:     return self.vdot(other);
16831:   };
16832:   return wrap(dispatch_vdot(self, _r.tensor(0)));
16833:   Py_RETURN_NONE;
16834:   END_HANDLE_TH_ERRORS
16835: }
16836: 
16837: \
16838: // view
16839: static PyObject * THPVariable_view(PyObject* self_, PyObject* args, PyObject* kwargs)
16840: {
16841:   HANDLE_TH_ERRORS
16842:   const Tensor& self = THPVariable_Unpack(self_);
16843:   static PythonArgParser parser({
16844:     "view(ScalarType dtype)",
16845:     "view(SymIntArrayRef size)",
16846:   }, /*traceable=*/true);
16847: 
16848:   ParsedArgs<1> parsed_args;
16849:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16850:   if(_r.has_torch_function()) {
16851:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16852:   }
16853:   switch (_r.idx) {
16854:     case 0: {
16855:       // aten::view.dtype(Tensor(a) self, ScalarType dtype) -> Tensor(a)
16856: 
16857:       auto dispatch_view = [](const at::Tensor & self, at::ScalarType dtype) -> at::Tensor {
16858:         pybind11::gil_scoped_release no_gil;
16859:         return self.view(dtype);
16860:       };
16861:       return wrap(dispatch_view(self, _r.scalartype(0)));
16862:     }
16863:     case 1: {
16864:       // aten::view(Tensor(a) self, SymInt[] size) -> Tensor(a)
16865: 
16866:       auto dispatch_view = [](const at::Tensor & self, c10::SymIntArrayRef size) -> at::Tensor {
16867:         pybind11::gil_scoped_release no_gil;
16868:         return self.view_symint(size);
16869:       };
16870:       return wrap(dispatch_view(self, _r.symintlist(0)));
16871:     }
16872:   }
16873:   Py_RETURN_NONE;
16874:   END_HANDLE_TH_ERRORS
16875: }
16876: 
16877: // view_as
16878: static PyObject * THPVariable_view_as(PyObject* self_, PyObject* args, PyObject* kwargs)
16879: {
16880:   HANDLE_TH_ERRORS
16881:   const Tensor& self = THPVariable_Unpack(self_);
16882:   static PythonArgParser parser({
16883:     "view_as(Tensor other)",
16884:   }, /*traceable=*/true);
16885: 
16886:   ParsedArgs<1> parsed_args;
16887:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16888:   if(_r.has_torch_function()) {
16889:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16890:   }
16891:   // aten::view_as(Tensor(a) self, Tensor other) -> Tensor(a)
16892: 
16893:   auto dispatch_view_as = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
16894:     pybind11::gil_scoped_release no_gil;
16895:     return self.view_as(other);
16896:   };
16897:   return wrap(dispatch_view_as(self, _r.tensor(0)));
16898:   Py_RETURN_NONE;
16899:   END_HANDLE_TH_ERRORS
16900: }
16901: 
16902: \
16903: // vsplit
16904: static PyObject * THPVariable_vsplit(PyObject* self_, PyObject* args, PyObject* kwargs)
16905: {
16906:   HANDLE_TH_ERRORS
16907:   const Tensor& self = THPVariable_Unpack(self_);
16908:   static PythonArgParser parser({
16909:     "vsplit(int64_t sections)",
16910:     "vsplit(IntArrayRef indices)",
16911:   }, /*traceable=*/true);
16912: 
16913:   ParsedArgs<1> parsed_args;
16914:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16915:   if(_r.has_torch_function()) {
16916:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16917:   }
16918:   switch (_r.idx) {
16919:     case 0: {
16920:       // aten::vsplit.int(Tensor(a -> *) self, int sections) -> Tensor(a)[]
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_vdot`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_vdot`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 16921-17040

```cpp
16921: 
16922:       auto dispatch_vsplit = [](const at::Tensor & self, int64_t sections) -> ::std::vector<at::Tensor> {
16923:         pybind11::gil_scoped_release no_gil;
16924:         return self.vsplit(sections);
16925:       };
16926:       return wrap(dispatch_vsplit(self, _r.toInt64(0)));
16927:     }
16928:     case 1: {
16929:       // aten::vsplit.array(Tensor(a -> *) self, int[] indices) -> Tensor(a)[]
16930: 
16931:       auto dispatch_vsplit = [](const at::Tensor & self, at::IntArrayRef indices) -> ::std::vector<at::Tensor> {
16932:         pybind11::gil_scoped_release no_gil;
16933:         return self.vsplit(indices);
16934:       };
16935:       return wrap(dispatch_vsplit(self, _r.intlist(0)));
16936:     }
16937:   }
16938:   Py_RETURN_NONE;
16939:   END_HANDLE_TH_ERRORS
16940: }
16941: 
16942: \
16943: // where
16944: static PyObject * THPVariable_where(PyObject* self_, PyObject* args, PyObject* kwargs)
16945: {
16946:   HANDLE_TH_ERRORS
16947:   const Tensor& self = THPVariable_Unpack(self_);
16948:   static PythonArgParser parser({
16949:     "where(Tensor condition, Tensor other)",
16950:     "where(Tensor condition, Scalar other)",
16951:   }, /*traceable=*/true);
16952: 
16953:   ParsedArgs<2> parsed_args;
16954:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16955:   if(_r.has_torch_function()) {
16956:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16957:   }
16958:   switch (_r.idx) {
16959:     case 0: {
16960:       // aten::where.self(Tensor condition, Tensor self, Tensor other) -> Tensor
16961: 
16962:       auto dispatch_where = [](const at::Tensor & condition, const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
16963:         pybind11::gil_scoped_release no_gil;
16964:         return self.where(condition, other);
16965:       };
16966:       return wrap(dispatch_where(_r.tensor(0), self, _r.tensor(1)));
16967:     }
16968:     case 1: {
16969:       // aten::where.ScalarOther(Tensor condition, Tensor self, Scalar other) -> Tensor
16970: 
16971:       auto dispatch_where = [](const at::Tensor & condition, const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
16972:         pybind11::gil_scoped_release no_gil;
16973:         return self.where(condition, other);
16974:       };
16975:       return wrap(dispatch_where(_r.tensor(0), self, _r.scalar(1)));
16976:     }
16977:   }
16978:   Py_RETURN_NONE;
16979:   END_HANDLE_TH_ERRORS
16980: }
16981: 
16982: \
16983: // xlogy
16984: static PyObject * THPVariable_xlogy(PyObject* self_, PyObject* args, PyObject* kwargs)
16985: {
16986:   HANDLE_TH_ERRORS
16987:   const Tensor& self = THPVariable_Unpack(self_);
16988:   static PythonArgParser parser({
16989:     "xlogy(Tensor other)",
16990:     "xlogy(Scalar other)",
16991:   }, /*traceable=*/true);
16992: 
16993:   ParsedArgs<1> parsed_args;
16994:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
16995:   if(_r.has_torch_function()) {
16996:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
16997:   }
16998:   switch (_r.idx) {
16999:     case 0: {
17000:       // aten::xlogy.Tensor(Tensor self, Tensor other) -> Tensor
17001: 
17002:       auto dispatch_xlogy = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
17003:         pybind11::gil_scoped_release no_gil;
17004:         return self.xlogy(other);
17005:       };
17006:       return wrap(dispatch_xlogy(self, _r.tensor(0)));
17007:     }
17008:     case 1: {
17009:       // aten::xlogy.Scalar_Other(Tensor self, Scalar other) -> Tensor
17010: 
17011:       auto dispatch_xlogy = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
17012:         pybind11::gil_scoped_release no_gil;
17013:         return self.xlogy(other);
17014:       };
17015:       return wrap(dispatch_xlogy(self, _r.scalar(0)));
17016:     }
17017:   }
17018:   Py_RETURN_NONE;
17019:   END_HANDLE_TH_ERRORS
17020: }
17021: 
17022: \
17023: // xlogy_
17024: static PyObject * THPVariable_xlogy_(PyObject* self_, PyObject* args, PyObject* kwargs)
17025: {
17026:   HANDLE_TH_ERRORS
17027:   const Tensor& self = THPVariable_Unpack(self_);
17028:   static PythonArgParser parser({
17029:     "xlogy_(Tensor other)",
17030:     "xlogy_(Scalar other)",
17031:   }, /*traceable=*/true);
17032: 
17033:   ParsedArgs<1> parsed_args;
17034:   auto _r = parser.parse(self_, args, kwargs, parsed_args);
17035:   if(_r.has_torch_function()) {
17036:     return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
17037:   }
17038:   switch (_r.idx) {
17039:     case 0: {
17040:       // aten::xlogy_.Tensor(Tensor(a!) self, Tensor other) -> Tensor(a!)
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_where`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_where`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 17041-17160

```cpp
17041: 
17042:       auto dispatch_xlogy_ = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
17043:         pybind11::gil_scoped_release no_gil;
17044:         return self.xlogy_(other);
17045:       };
17046:       return wrap(dispatch_xlogy_(self, _r.tensor(0)));
17047:     }
17048:     case 1: {
17049:       // aten::xlogy_.Scalar_Other(Tensor(a!) self, Scalar other) -> Tensor(a!)
17050: 
17051:       auto dispatch_xlogy_ = [](const at::Tensor & self, const at::Scalar & other) -> at::Tensor {
17052:         pybind11::gil_scoped_release no_gil;
17053:         return self.xlogy_(other);
17054:       };
17055:       return wrap(dispatch_xlogy_(self, _r.scalar(0)));
17056:     }
17057:   }
17058:   Py_RETURN_NONE;
17059:   END_HANDLE_TH_ERRORS
17060: }
17061: 
17062: // zero_
17063: static PyObject * THPVariable_zero_(PyObject* self_, PyObject* args)
17064: {
17065:   HANDLE_TH_ERRORS
17066:   const Tensor& self = THPVariable_Unpack(self_);
17067:   if (has_torch_function(self_)) {
17068:     return handle_torch_function(self_, "zero_");
17069:   }
17070:   // aten::zero_(Tensor(a!) self) -> Tensor(a!)
17071: 
17072:   auto dispatch_zero_ = [](const at::Tensor & self) -> at::Tensor {
17073:     pybind11::gil_scoped_release no_gil;
17074:     return self.zero_();
17075:   };
17076:   return wrap(dispatch_zero_(self));
17077:   END_HANDLE_TH_ERRORS
17078: }
17079: 
17080: static PyObject * THPVariable_bool_scalar(PyObject* self, PyObject* args) {
17081:   if (has_torch_function(self)) {
17082:     HANDLE_TH_ERRORS
17083:     return handle_torch_function(self, "__bool__", args);
17084:     END_HANDLE_TH_ERRORS
17085:   }
17086:   jit::tracer::warn("Converting a tensor to a Python boolean", jit::tracer::WARN_PYTHON_DATAFLOW);
17087:   return THPVariable_is_nonzero(self, args);
17088: }
17089: 
17090: static PyObject * THPVariable___eq__(PyObject* self_, PyObject* args, PyObject* kwargs)
17091: {
17092:   HANDLE_TH_ERRORS
17093: #ifdef USE_NUMPY
17094:   if (torch::utils::is_numpy_available()) {
17095:     static PythonArgParser parser({
17096:       "__eq__(PyObject* other)",
17097:     }, /*traceable=*/true);
17098: 
17099:     ParsedArgs<1> parsed_args;
17100:     auto _r = parser.parse(self_, args, kwargs, parsed_args);
17101:     if(_r.has_torch_function()) {
17102:       return handle_torch_function(_r, self_, args, kwargs, THPVariableClass, "torch.Tensor");
17103:     }
17104:     switch (_r.idx) {
17105:       case 0: {
17106:         auto other = _r.pyobject(0);
17107:         if (PyArray_Check(other)) {
17108:           auto other_tensor = torch::utils::tensor_from_numpy(other);
17109:           auto dispatch_eq = [](const at::Tensor & self, const at::Tensor & other) -> at::Tensor {
17110:             pybind11::gil_scoped_release no_gil;
17111:             return self.eq(other);
17112:           };
17113:           const Tensor& self = THPVariable_Unpack(self_);
17114:           return wrap(dispatch_eq(self, other_tensor));
17115:         }
17116:       }
17117:     }
17118:   }
17119: #endif
17120:   return THPVariable_eq(self_, args, kwargs);
17121:   Py_RETURN_NONE;
17122:   END_HANDLE_TH_ERRORS
17123: }
17124: 
17125: // Wrapper converts a raised TypeError into returning NotImplemented
17126: // Used to implement binary arithmetic operators
17127: template <PyObject* (*Func)(PyObject*, PyObject*, PyObject*)>
17128: static PyObject * TypeError_to_NotImplemented_(PyObject* self, PyObject* args, PyObject* kwargs) {
17129: 
17130:   PyObject* ret = Func(self, args, kwargs);
17131:   if (!ret && PyErr_ExceptionMatches(PyExc_TypeError)) {
17132:     PyErr_Clear();
17133:     Py_INCREF(Py_NotImplemented);
17134:     ret = Py_NotImplemented;
17135:   }
17136:   return ret;
17137: }
17138: 
17139: // set_ has to be defined in the template because the c10::Storage object
17140: // does not have a type, and we need to make sure the Python storage object's
17141: // type matches the tensor's type
17142: static PyObject* THPVariable_set_(
17143:     PyObject* self_,
17144:     PyObject* args,
17145:     PyObject* kwargs) {
17146:   HANDLE_TH_ERRORS
17147:   const Tensor& self = THPVariable_Unpack(self_);
17148:   static PythonArgParser parser(
17149:       {
17150:           "set_()",
17151:           "set_(Storage source)",
17152:           "set_(Storage source, SymInt storage_offset, SymIntArrayRef size, SymIntArrayRef stride=None)",
17153:           "set_(Tensor source)",
17154:           "set_(Tensor source, SymInt storage_offset, SymIntArrayRef size, SymIntArrayRef stride=None)",
17155:       },
17156:       /*traceable=*/false);
17157: 
17158:   ParsedArgs<4> parsed_args;
17159:   auto _r = parser.parse(args, kwargs, parsed_args);
17160: 
```

- EN: The main execution path in this span is carried by `wrap`, `THPVariable_zero_`, `THPVariable_Unpack`. Control-flow branches in this range handle alternative runtime cases or iterative work. The code also manipulates Inductor/AOTI runtime state, kernel-facing data, or ABI-oriented wrappers. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `wrap`, `THPVariable_zero_`, `THPVariable_Unpack` 等函数/方法承载。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。 这段代码还会处理 Inductor/AOTI 运行时状态、面向内核的数据或以 ABI 为导向的封装。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 17161-17280

```cpp
17161:   switch (_r.idx) {
17162:     case 0: {
17163:       // aten::set_(Tensor(a!) self) -> Tensor(a!)
17164:       auto dispatch_set_ = [](const Tensor& self) -> Tensor {
17165:         pybind11::gil_scoped_release no_gil;
17166:         return self.set_();
17167:       };
17168:       return wrap(dispatch_set_(self));
17169:     }
17170:     case 1: {
17171:       // aten::set_.source_Storage(Tensor(a!) self, Storage source) ->
17172:       // Tensor(a!)
17173:       at::ScalarType storage_scalar_type{};
17174:       bool is_typed_storage = true;
17175:       at::Storage storage = _r.storage(0, storage_scalar_type, is_typed_storage);
17176:       TORCH_CHECK(storage_scalar_type == self.dtype() || !is_typed_storage,
17177:         "Expected a Storage of type ", self.dtype(),
17178:         " or an UntypedStorage, but got type ", storage_scalar_type,
17179:         " for argument 1 'storage'");
17180:       auto dispatch_set_ = [](const Tensor& self, Storage source) -> Tensor {
17181:         pybind11::gil_scoped_release no_gil;
17182:         return self.set_(std::move(source));
17183:       };
17184:       return wrap(dispatch_set_(self, storage));
17185:     }
17186:     case 2: {
17187:       // aten::set_.source_Storage_storage_offset(Tensor(a!) self, Storage
17188:       // source, int storage_offset, int[] size, int[] stride=[]) -> Tensor(a!)
17189:       at::ScalarType storage_scalar_type{};
17190:       bool is_typed_storage = true;
17191:       at::Storage storage = _r.storage(0, storage_scalar_type, is_typed_storage);
17192:       TORCH_CHECK(storage_scalar_type == self.dtype() || !is_typed_storage,
17193:         "Expected a Storage of type ", self.dtype(),
17194:         " or an UntypedStorage, but got type ", storage_scalar_type,
17195:         " for argument 1 'storage'");
17196:       auto dispatch_set_ = [](const Tensor& self,
17197:                               Storage source,
17198:                               c10::SymInt storage_offset,
17199:                               c10::SymIntArrayRef size,
17200:                               c10::SymIntArrayRef stride) -> Tensor {
17201:         pybind11::gil_scoped_release no_gil;
17202:         return self.set__symint(std::move(source), std::move(storage_offset), size, stride);
17203:       };
17204:       return wrap(dispatch_set_(
17205:           self, storage, _r.toSymInt(1), _r.symintlist(2), _r.symintlist(3)));
17206:     }
17207:     case 3: {
17208:       // aten::set_.source_Tensor(Tensor(a!) self, Tensor source) -> Tensor(a!)
17209:       auto dispatch_set_ = [](const Tensor& self, const Tensor& source) -> Tensor {
17210:         TORCH_CHECK(source.dtype() == self.dtype(), "Could not set tensor of type ", source.dtype(), " to a tensor of type ", self.dtype());
17211:         pybind11::gil_scoped_release no_gil;
17212:         return self.set_(source);
17213:       };
17214:       return wrap(dispatch_set_(self, _r.tensor(0)));
17215:     }
17216:     case 4: {
17217:       // aten::set_.source_Tensor_storage_offset(Tensor(a!) self, Tensor
17218:       // source, int storage_offset, int[] size, int[] stride=[]) -> Tensor(a!)
17219:       at::Tensor storage = _r.tensor(0);
17220:       auto dispatch_set_ = [](const Tensor& self,
17221:                               const Tensor& source,
17222:                               c10::SymInt storage_offset,
17223:                               c10::SymIntArrayRef size,
17224:                               c10::SymIntArrayRef stride) -> Tensor {
17225:         pybind11::gil_scoped_release no_gil;
17226:         return self.set__symint(source, std::move(storage_offset), size, stride);
17227:       };
17228:       return wrap(dispatch_set_(
17229:           self, storage, _r.toSymInt(1), _r.symintlist(2), _r.symintlist(3)));
17230:     }
17231:   }
17232:   Py_RETURN_NONE;
17233:   END_HANDLE_TH_ERRORS
17234: }
17235: 
17236: // XXX: ops that are bound here are not exposed to the C++ api nor the JIT.
17237: // Any new ops added here should be accompanied with a comment why they are not
17238: // being registered through native_functions.yaml, and be tagged cpp / JIT
17239: PyMethodDef variable_methods[] = {
17240:   // These magic methods are all implemented on python object to wrap NotImplementedError
17241:   {"__add__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_add>), METH_VARARGS | METH_KEYWORDS, nullptr},
17242:   {"__radd__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_add>), METH_VARARGS | METH_KEYWORDS, nullptr},
17243:   {"__iadd__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_add_>), METH_VARARGS | METH_KEYWORDS, nullptr},
17244:   {"__rmul__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_mul>), METH_VARARGS | METH_KEYWORDS, nullptr},
17245:   {"__mul__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_mul>), METH_VARARGS | METH_KEYWORDS, nullptr},
17246:   {"__imul__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_mul_>), METH_VARARGS | METH_KEYWORDS, nullptr},
17247:   {"__sub__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_sub>), METH_VARARGS | METH_KEYWORDS, nullptr},
17248:   {"__isub__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_sub_>), METH_VARARGS | METH_KEYWORDS, nullptr},
17249:   {"__div__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_div>), METH_VARARGS | METH_KEYWORDS, nullptr},
17250:   {"__truediv__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_div>), METH_VARARGS | METH_KEYWORDS, nullptr},
17251:   {"__floordiv__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_floor_divide>), METH_VARARGS | METH_KEYWORDS, nullptr},
17252:   {"__idiv__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_div_>), METH_VARARGS | METH_KEYWORDS, nullptr},
17253:   {"__ifloordiv__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_floor_divide_>), METH_VARARGS | METH_KEYWORDS, nullptr},
17254:   {"__mod__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_remainder>), METH_VARARGS | METH_KEYWORDS, nullptr},
17255:   {"__imod__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_remainder_>), METH_VARARGS | METH_KEYWORDS, nullptr},
17256:   {"__eq__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___eq__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17257:   {"__ne__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_ne>), METH_VARARGS | METH_KEYWORDS, nullptr},
17258:   {"__lt__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_lt>), METH_VARARGS | METH_KEYWORDS, nullptr},
17259:   {"__le__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_le>), METH_VARARGS | METH_KEYWORDS, nullptr},
17260:   {"__gt__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_gt>), METH_VARARGS | METH_KEYWORDS, nullptr},
17261:   {"__ge__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_ge>), METH_VARARGS | METH_KEYWORDS, nullptr},
17262:   {"__rand__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_bitwise_and>), METH_VARARGS | METH_KEYWORDS, nullptr},
17263:   {"__ror__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_bitwise_or>), METH_VARARGS | METH_KEYWORDS, nullptr},
17264:   {"__rxor__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_bitwise_xor>), METH_VARARGS | METH_KEYWORDS, nullptr},
17265:   {"__bool__", THPVariable_bool_scalar, METH_NOARGS, nullptr},
17266:   {"__float__", THPVariable_float_scalar, METH_NOARGS, nullptr},
17267:   {"__complex__", THPVariable_complex_scalar, METH_NOARGS, nullptr},
17268:   {"__int__", THPVariable_integral_scalar, METH_NOARGS, nullptr},
17269:   {"__long__", THPVariable_integral_scalar, METH_NOARGS, nullptr},
17270:   {"__index__", THPVariable_index_scalar, METH_NOARGS, nullptr},
17271:   {"__nonzero__", THPVariable_bool_scalar, METH_NOARGS, nullptr},
17272:   {"__invert__", THPVariable_invert, METH_NOARGS, nullptr},
17273:   {"__matmul__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable_matmul>), METH_VARARGS | METH_KEYWORDS, nullptr},
17274:   {"_is_view", THPVariable__is_view, METH_NOARGS, nullptr},
17275:   {"apply_", THPVariable_apply_, METH_O, nullptr},
17276:   {"bfloat16", castPyCFunctionWithKeywords(THPVariable_bfloat16), METH_VARARGS | METH_KEYWORDS, nullptr},
17277:   {"byte", castPyCFunctionWithKeywords(THPVariable_byte), METH_VARARGS | METH_KEYWORDS, nullptr},
17278:   {"char", castPyCFunctionWithKeywords(THPVariable_char), METH_VARARGS | METH_KEYWORDS, nullptr},
17279:   {"contiguous", castPyCFunctionWithKeywords(THPVariable_contiguous), METH_VARARGS | METH_KEYWORDS, nullptr},
17280:   {"copy_", castPyCFunctionWithKeywords(THPVariable_copy_), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `set_`, `wrap`, `Tensor`. Because this is generated binding code, the span mostly registers or forwards APIs into a mechanically produced Python-facing surface. The logic emits runtime diagnostics or assertions to guard assumptions. Control-flow branches in this range handle alternative runtime cases or iterative work.
- CN: 这一段的主要执行路径由 `set_`, `wrap`, `Tensor` 等函数/方法承载。 由于这是生成的绑定代码，这一段主要是在机械化生成的 Python 接口层上完成 API 注册或转发。 这里的逻辑会发出运行时诊断或断言，以保护关键假设。 这一段中的控制流分支用于处理不同的运行时情况或循环工作。
### Lines 17281-17400

```cpp
17281:   {"cpu", castPyCFunctionWithKeywords(THPVariable_cpu), METH_VARARGS | METH_KEYWORDS, nullptr},
17282:   {"cuda", castPyCFunctionWithKeywords(THPVariable_cuda), METH_VARARGS | METH_KEYWORDS, nullptr},
17283:   {"mtia", castPyCFunctionWithKeywords(THPVariable_mtia), METH_VARARGS | METH_KEYWORDS, nullptr},
17284:   {"xpu", castPyCFunctionWithKeywords(THPVariable_xpu), METH_VARARGS | METH_KEYWORDS, nullptr},
17285:   {"ipu", castPyCFunctionWithKeywords(THPVariable_ipu), METH_VARARGS | METH_KEYWORDS, nullptr},
17286:   {"const_data_ptr", THPVariable_const_data_ptr, METH_NOARGS, nullptr},
17287:   {"data_ptr", THPVariable_data_ptr, METH_NOARGS, nullptr},
17288:   {"dim", THPVariable_dim, METH_NOARGS, nullptr},
17289:   {"has_names", THPVariable_has_names, METH_NOARGS, nullptr},
17290:   {"double", castPyCFunctionWithKeywords(THPVariable_double), METH_VARARGS | METH_KEYWORDS, nullptr},
17291:   {"cdouble", castPyCFunctionWithKeywords(THPVariable_cdouble), METH_VARARGS | METH_KEYWORDS, nullptr},
17292:   {"element_size", THPVariable_element_size, METH_NOARGS, nullptr},
17293:   {"float", castPyCFunctionWithKeywords(THPVariable_float), METH_VARARGS | METH_KEYWORDS, nullptr},
17294:   {"cfloat", castPyCFunctionWithKeywords(THPVariable_cfloat), METH_VARARGS | METH_KEYWORDS, nullptr},
17295:   {"get_device", THPVariable_get_device, METH_NOARGS, nullptr},
17296:   {"bool", castPyCFunctionWithKeywords(THPVariable_bool), METH_VARARGS | METH_KEYWORDS, nullptr},
17297:   {"half", castPyCFunctionWithKeywords(THPVariable_half), METH_VARARGS | METH_KEYWORDS, nullptr},
17298:   {"int", castPyCFunctionWithKeywords(THPVariable_int), METH_VARARGS | METH_KEYWORDS, nullptr},
17299:   {"is_contiguous", castPyCFunctionWithKeywords(THPVariable_is_contiguous), METH_VARARGS | METH_KEYWORDS, nullptr},
17300:   {"item", THPVariable_item, METH_NOARGS, nullptr},
17301:   {"long", castPyCFunctionWithKeywords(THPVariable_long), METH_VARARGS | METH_KEYWORDS, nullptr},
17302:   {"map_", castPyCFunctionWithKeywords(THPVariable_map_), METH_VARARGS | METH_KEYWORDS, nullptr},
17303:   {"map2_", castPyCFunctionWithKeywords(THPVariable_map2_), METH_VARARGS | METH_KEYWORDS, nullptr},
17304:   {"ndimension", THPVariable_dim, METH_NOARGS, nullptr},
17305:   {"nelement", THPVariable_numel, METH_NOARGS, nullptr},
17306:   {"new", castPyCFunctionWithKeywords(THPVariable_new), METH_VARARGS | METH_KEYWORDS, nullptr},
17307:   {"new_tensor", castPyCFunctionWithKeywords(THPVariable_new_tensor), METH_VARARGS | METH_KEYWORDS, nullptr},
17308:   {"nonzero", castPyCFunctionWithKeywords(THPVariable_nonzero), METH_VARARGS | METH_KEYWORDS, nullptr},
17309:   {"numel", THPVariable_numel, METH_NOARGS, nullptr},
17310:   {"numpy", castPyCFunctionWithKeywords(THPVariable_numpy), METH_VARARGS | METH_KEYWORDS, nullptr},
17311:   {"requires_grad_", castPyCFunctionWithKeywords(THPVariable_requires_grad_), METH_VARARGS | METH_KEYWORDS, nullptr},
17312:   {"set_", castPyCFunctionWithKeywords(THPVariable_set_), METH_VARARGS | METH_KEYWORDS, nullptr},
17313:   {"short", castPyCFunctionWithKeywords(THPVariable_short), METH_VARARGS | METH_KEYWORDS, nullptr},
17314:   {"size", castPyCFunctionWithKeywords(THPVariable_size), METH_VARARGS | METH_KEYWORDS, nullptr},
17315:   {"untyped_storage", THPVariable_storage, METH_NOARGS, nullptr},
17316:   {"storage_offset", THPVariable_storage_offset, METH_NOARGS, nullptr},
17317:   {"stride", castPyCFunctionWithKeywords(THPVariable_stride), METH_VARARGS | METH_KEYWORDS, nullptr},
17318:   {"to", castPyCFunctionWithKeywords(THPVariable_to), METH_VARARGS | METH_KEYWORDS, nullptr},
17319:   {"tolist", THPVariable_tolist, METH_NOARGS, nullptr},
17320:   {"type", castPyCFunctionWithKeywords(THPVariable_type), METH_VARARGS | METH_KEYWORDS, nullptr},
17321:   {"__and__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___and__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17322:   {"__iand__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___iand__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17323:   {"__ilshift__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___ilshift__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17324:   {"__ior__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___ior__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17325:   {"__irshift__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___irshift__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17326:   {"__ixor__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___ixor__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17327:   {"__lshift__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___lshift__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17328:   {"__or__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___or__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17329:   {"__rshift__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___rshift__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17330:   {"__xor__", castPyCFunctionWithKeywords(TypeError_to_NotImplemented_<THPVariable___xor__>), METH_VARARGS | METH_KEYWORDS, nullptr},
17331:   {"_addmm_activation", castPyCFunctionWithKeywords(THPVariable__addmm_activation), METH_VARARGS | METH_KEYWORDS, nullptr},
17332:   {"_autocast_to_full_precision", castPyCFunctionWithKeywords(THPVariable__autocast_to_full_precision), METH_VARARGS | METH_KEYWORDS, nullptr},
17333:   {"_autocast_to_reduced_precision", castPyCFunctionWithKeywords(THPVariable__autocast_to_reduced_precision), METH_VARARGS | METH_KEYWORDS, nullptr},
17334:   {"_coalesced_", castPyCFunctionWithKeywords(THPVariable__coalesced_), METH_VARARGS | METH_KEYWORDS, nullptr},
17335:   {"_conj", THPVariable__conj, METH_NOARGS, nullptr},
17336:   {"_conj_physical", THPVariable__conj_physical, METH_NOARGS, nullptr},
17337:   {"_dimI", THPVariable__dimI, METH_NOARGS, nullptr},
17338:   {"_dimV", THPVariable__dimV, METH_NOARGS, nullptr},
17339:   {"_indices", THPVariable__indices, METH_NOARGS, nullptr},
17340:   {"_is_all_true", THPVariable__is_all_true, METH_NOARGS, nullptr},
17341:   {"_is_any_true", THPVariable__is_any_true, METH_NOARGS, nullptr},
17342:   {"_is_zerotensor", THPVariable__is_zerotensor, METH_NOARGS, nullptr},
17343:   {"_lazy_clone", THPVariable__lazy_clone, METH_NOARGS, nullptr},
17344:   {"_neg_view", THPVariable__neg_view, METH_NOARGS, nullptr},
17345:   {"_nested_tensor_size", THPVariable__nested_tensor_size, METH_NOARGS, nullptr},
17346:   {"_nested_tensor_storage_offsets", THPVariable__nested_tensor_storage_offsets, METH_NOARGS, nullptr},
17347:   {"_nested_tensor_strides", THPVariable__nested_tensor_strides, METH_NOARGS, nullptr},
17348:   {"_nnz", THPVariable__nnz, METH_NOARGS, nullptr},
17349:   {"_philox_normal_", castPyCFunctionWithKeywords(THPVariable__philox_normal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17350:   {"_philox_uniform_", castPyCFunctionWithKeywords(THPVariable__philox_uniform_), METH_VARARGS | METH_KEYWORDS, nullptr},
17351:   {"_sparse_mask_projection", castPyCFunctionWithKeywords(THPVariable__sparse_mask_projection), METH_VARARGS | METH_KEYWORDS, nullptr},
17352:   {"_to_dense", castPyCFunctionWithKeywords(THPVariable__to_dense), METH_VARARGS | METH_KEYWORDS, nullptr},
17353:   {"_to_sparse", castPyCFunctionWithKeywords(THPVariable__to_sparse), METH_VARARGS | METH_KEYWORDS, nullptr},
17354:   {"_to_sparse_bsc", castPyCFunctionWithKeywords(THPVariable__to_sparse_bsc), METH_VARARGS | METH_KEYWORDS, nullptr},
17355:   {"_to_sparse_bsr", castPyCFunctionWithKeywords(THPVariable__to_sparse_bsr), METH_VARARGS | METH_KEYWORDS, nullptr},
17356:   {"_to_sparse_csc", castPyCFunctionWithKeywords(THPVariable__to_sparse_csc), METH_VARARGS | METH_KEYWORDS, nullptr},
17357:   {"_to_sparse_csr", castPyCFunctionWithKeywords(THPVariable__to_sparse_csr), METH_VARARGS | METH_KEYWORDS, nullptr},
17358:   {"_values", THPVariable__values, METH_NOARGS, nullptr},
17359:   {"abs", THPVariable_abs, METH_NOARGS, nullptr},
17360:   {"abs_", THPVariable_abs_, METH_NOARGS, nullptr},
17361:   {"absolute", THPVariable_absolute, METH_NOARGS, nullptr},
17362:   {"absolute_", THPVariable_absolute_, METH_NOARGS, nullptr},
17363:   {"acos", THPVariable_acos, METH_NOARGS, nullptr},
17364:   {"acos_", THPVariable_acos_, METH_NOARGS, nullptr},
17365:   {"acosh", THPVariable_acosh, METH_NOARGS, nullptr},
17366:   {"acosh_", THPVariable_acosh_, METH_NOARGS, nullptr},
17367:   {"add", castPyCFunctionWithKeywords(THPVariable_add), METH_VARARGS | METH_KEYWORDS, nullptr},
17368:   {"add_", castPyCFunctionWithKeywords(THPVariable_add_), METH_VARARGS | METH_KEYWORDS, nullptr},
17369:   {"addbmm", castPyCFunctionWithKeywords(THPVariable_addbmm), METH_VARARGS | METH_KEYWORDS, nullptr},
17370:   {"addbmm_", castPyCFunctionWithKeywords(THPVariable_addbmm_), METH_VARARGS | METH_KEYWORDS, nullptr},
17371:   {"addcdiv", castPyCFunctionWithKeywords(THPVariable_addcdiv), METH_VARARGS | METH_KEYWORDS, nullptr},
17372:   {"addcdiv_", castPyCFunctionWithKeywords(THPVariable_addcdiv_), METH_VARARGS | METH_KEYWORDS, nullptr},
17373:   {"addcmul", castPyCFunctionWithKeywords(THPVariable_addcmul), METH_VARARGS | METH_KEYWORDS, nullptr},
17374:   {"addcmul_", castPyCFunctionWithKeywords(THPVariable_addcmul_), METH_VARARGS | METH_KEYWORDS, nullptr},
17375:   {"addmm", castPyCFunctionWithKeywords(THPVariable_addmm), METH_VARARGS | METH_KEYWORDS, nullptr},
17376:   {"addmm_", castPyCFunctionWithKeywords(THPVariable_addmm_), METH_VARARGS | METH_KEYWORDS, nullptr},
17377:   {"addmv", castPyCFunctionWithKeywords(THPVariable_addmv), METH_VARARGS | METH_KEYWORDS, nullptr},
17378:   {"addmv_", castPyCFunctionWithKeywords(THPVariable_addmv_), METH_VARARGS | METH_KEYWORDS, nullptr},
17379:   {"addr", castPyCFunctionWithKeywords(THPVariable_addr), METH_VARARGS | METH_KEYWORDS, nullptr},
17380:   {"addr_", castPyCFunctionWithKeywords(THPVariable_addr_), METH_VARARGS | METH_KEYWORDS, nullptr},
17381:   {"adjoint", THPVariable_adjoint, METH_NOARGS, nullptr},
17382:   {"align_as", castPyCFunctionWithKeywords(THPVariable_align_as), METH_VARARGS | METH_KEYWORDS, nullptr},
17383:   {"align_to", castPyCFunctionWithKeywords(THPVariable_align_to), METH_VARARGS | METH_KEYWORDS, nullptr},
17384:   {"all", castPyCFunctionWithKeywords(THPVariable_all), METH_VARARGS | METH_KEYWORDS, nullptr},
17385:   {"allclose", castPyCFunctionWithKeywords(THPVariable_allclose), METH_VARARGS | METH_KEYWORDS, nullptr},
17386:   {"amax", castPyCFunctionWithKeywords(THPVariable_amax), METH_VARARGS | METH_KEYWORDS, nullptr},
17387:   {"amin", castPyCFunctionWithKeywords(THPVariable_amin), METH_VARARGS | METH_KEYWORDS, nullptr},
17388:   {"aminmax", castPyCFunctionWithKeywords(THPVariable_aminmax), METH_VARARGS | METH_KEYWORDS, nullptr},
17389:   {"angle", THPVariable_angle, METH_NOARGS, nullptr},
17390:   {"any", castPyCFunctionWithKeywords(THPVariable_any), METH_VARARGS | METH_KEYWORDS, nullptr},
17391:   {"arccos", THPVariable_arccos, METH_NOARGS, nullptr},
17392:   {"arccos_", THPVariable_arccos_, METH_NOARGS, nullptr},
17393:   {"arccosh", THPVariable_arccosh, METH_NOARGS, nullptr},
17394:   {"arccosh_", THPVariable_arccosh_, METH_NOARGS, nullptr},
17395:   {"arcsin", THPVariable_arcsin, METH_NOARGS, nullptr},
17396:   {"arcsin_", THPVariable_arcsin_, METH_NOARGS, nullptr},
17397:   {"arcsinh", THPVariable_arcsinh, METH_NOARGS, nullptr},
17398:   {"arcsinh_", THPVariable_arcsinh_, METH_NOARGS, nullptr},
17399:   {"arctan", THPVariable_arctan, METH_NOARGS, nullptr},
17400:   {"arctan2", castPyCFunctionWithKeywords(THPVariable_arctan2), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. Autograd-related metadata or graph connectivity is updated here. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这里会更新与自动求导相关的元数据或计算图连接关系。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 17401-17520

```cpp
17401:   {"arctan2_", castPyCFunctionWithKeywords(THPVariable_arctan2_), METH_VARARGS | METH_KEYWORDS, nullptr},
17402:   {"arctan_", THPVariable_arctan_, METH_NOARGS, nullptr},
17403:   {"arctanh", THPVariable_arctanh, METH_NOARGS, nullptr},
17404:   {"arctanh_", THPVariable_arctanh_, METH_NOARGS, nullptr},
17405:   {"argmax", castPyCFunctionWithKeywords(THPVariable_argmax), METH_VARARGS | METH_KEYWORDS, nullptr},
17406:   {"argmin", castPyCFunctionWithKeywords(THPVariable_argmin), METH_VARARGS | METH_KEYWORDS, nullptr},
17407:   {"argsort", castPyCFunctionWithKeywords(THPVariable_argsort), METH_VARARGS | METH_KEYWORDS, nullptr},
17408:   {"argwhere", THPVariable_argwhere, METH_NOARGS, nullptr},
17409:   {"as_strided", castPyCFunctionWithKeywords(THPVariable_as_strided), METH_VARARGS | METH_KEYWORDS, nullptr},
17410:   {"as_strided_", castPyCFunctionWithKeywords(THPVariable_as_strided_), METH_VARARGS | METH_KEYWORDS, nullptr},
17411:   {"as_strided_scatter", castPyCFunctionWithKeywords(THPVariable_as_strided_scatter), METH_VARARGS | METH_KEYWORDS, nullptr},
17412:   {"asin", THPVariable_asin, METH_NOARGS, nullptr},
17413:   {"asin_", THPVariable_asin_, METH_NOARGS, nullptr},
17414:   {"asinh", THPVariable_asinh, METH_NOARGS, nullptr},
17415:   {"asinh_", THPVariable_asinh_, METH_NOARGS, nullptr},
17416:   {"atan", THPVariable_atan, METH_NOARGS, nullptr},
17417:   {"atan2", castPyCFunctionWithKeywords(THPVariable_atan2), METH_VARARGS | METH_KEYWORDS, nullptr},
17418:   {"atan2_", castPyCFunctionWithKeywords(THPVariable_atan2_), METH_VARARGS | METH_KEYWORDS, nullptr},
17419:   {"atan_", THPVariable_atan_, METH_NOARGS, nullptr},
17420:   {"atanh", THPVariable_atanh, METH_NOARGS, nullptr},
17421:   {"atanh_", THPVariable_atanh_, METH_NOARGS, nullptr},
17422:   {"baddbmm", castPyCFunctionWithKeywords(THPVariable_baddbmm), METH_VARARGS | METH_KEYWORDS, nullptr},
17423:   {"baddbmm_", castPyCFunctionWithKeywords(THPVariable_baddbmm_), METH_VARARGS | METH_KEYWORDS, nullptr},
17424:   {"bernoulli", castPyCFunctionWithKeywords(THPVariable_bernoulli), METH_VARARGS | METH_KEYWORDS, nullptr},
17425:   {"bernoulli_", castPyCFunctionWithKeywords(THPVariable_bernoulli_), METH_VARARGS | METH_KEYWORDS, nullptr},
17426:   {"bincount", castPyCFunctionWithKeywords(THPVariable_bincount), METH_VARARGS | METH_KEYWORDS, nullptr},
17427:   {"bitwise_and", castPyCFunctionWithKeywords(THPVariable_bitwise_and), METH_VARARGS | METH_KEYWORDS, nullptr},
17428:   {"bitwise_and_", castPyCFunctionWithKeywords(THPVariable_bitwise_and_), METH_VARARGS | METH_KEYWORDS, nullptr},
17429:   {"bitwise_left_shift", castPyCFunctionWithKeywords(THPVariable_bitwise_left_shift), METH_VARARGS | METH_KEYWORDS, nullptr},
17430:   {"bitwise_left_shift_", castPyCFunctionWithKeywords(THPVariable_bitwise_left_shift_), METH_VARARGS | METH_KEYWORDS, nullptr},
17431:   {"bitwise_not", THPVariable_bitwise_not, METH_NOARGS, nullptr},
17432:   {"bitwise_not_", THPVariable_bitwise_not_, METH_NOARGS, nullptr},
17433:   {"bitwise_or", castPyCFunctionWithKeywords(THPVariable_bitwise_or), METH_VARARGS | METH_KEYWORDS, nullptr},
17434:   {"bitwise_or_", castPyCFunctionWithKeywords(THPVariable_bitwise_or_), METH_VARARGS | METH_KEYWORDS, nullptr},
17435:   {"bitwise_right_shift", castPyCFunctionWithKeywords(THPVariable_bitwise_right_shift), METH_VARARGS | METH_KEYWORDS, nullptr},
17436:   {"bitwise_right_shift_", castPyCFunctionWithKeywords(THPVariable_bitwise_right_shift_), METH_VARARGS | METH_KEYWORDS, nullptr},
17437:   {"bitwise_xor", castPyCFunctionWithKeywords(THPVariable_bitwise_xor), METH_VARARGS | METH_KEYWORDS, nullptr},
17438:   {"bitwise_xor_", castPyCFunctionWithKeywords(THPVariable_bitwise_xor_), METH_VARARGS | METH_KEYWORDS, nullptr},
17439:   {"bmm", castPyCFunctionWithKeywords(THPVariable_bmm), METH_VARARGS | METH_KEYWORDS, nullptr},
17440:   {"broadcast_to", castPyCFunctionWithKeywords(THPVariable_broadcast_to), METH_VARARGS | METH_KEYWORDS, nullptr},
17441:   {"cauchy_", castPyCFunctionWithKeywords(THPVariable_cauchy_), METH_VARARGS | METH_KEYWORDS, nullptr},
17442:   {"ccol_indices", THPVariable_ccol_indices, METH_NOARGS, nullptr},
17443:   {"ceil", THPVariable_ceil, METH_NOARGS, nullptr},
17444:   {"ceil_", THPVariable_ceil_, METH_NOARGS, nullptr},
17445:   {"chalf", castPyCFunctionWithKeywords(THPVariable_chalf), METH_VARARGS | METH_KEYWORDS, nullptr},
17446:   {"cholesky", castPyCFunctionWithKeywords(THPVariable_cholesky), METH_VARARGS | METH_KEYWORDS, nullptr},
17447:   {"cholesky_inverse", castPyCFunctionWithKeywords(THPVariable_cholesky_inverse), METH_VARARGS | METH_KEYWORDS, nullptr},
17448:   {"cholesky_solve", castPyCFunctionWithKeywords(THPVariable_cholesky_solve), METH_VARARGS | METH_KEYWORDS, nullptr},
17449:   {"chunk", castPyCFunctionWithKeywords(THPVariable_chunk), METH_VARARGS | METH_KEYWORDS, nullptr},
17450:   {"clamp", castPyCFunctionWithKeywords(THPVariable_clamp), METH_VARARGS | METH_KEYWORDS, nullptr},
17451:   {"clamp_", castPyCFunctionWithKeywords(THPVariable_clamp_), METH_VARARGS | METH_KEYWORDS, nullptr},
17452:   {"clamp_max", castPyCFunctionWithKeywords(THPVariable_clamp_max), METH_VARARGS | METH_KEYWORDS, nullptr},
17453:   {"clamp_max_", castPyCFunctionWithKeywords(THPVariable_clamp_max_), METH_VARARGS | METH_KEYWORDS, nullptr},
17454:   {"clamp_min", castPyCFunctionWithKeywords(THPVariable_clamp_min), METH_VARARGS | METH_KEYWORDS, nullptr},
17455:   {"clamp_min_", castPyCFunctionWithKeywords(THPVariable_clamp_min_), METH_VARARGS | METH_KEYWORDS, nullptr},
17456:   {"clip", castPyCFunctionWithKeywords(THPVariable_clip), METH_VARARGS | METH_KEYWORDS, nullptr},
17457:   {"clip_", castPyCFunctionWithKeywords(THPVariable_clip_), METH_VARARGS | METH_KEYWORDS, nullptr},
17458:   {"clone", castPyCFunctionWithKeywords(THPVariable_clone), METH_VARARGS | METH_KEYWORDS, nullptr},
17459:   {"coalesce", THPVariable_coalesce, METH_NOARGS, nullptr},
17460:   {"col_indices", THPVariable_col_indices, METH_NOARGS, nullptr},
17461:   {"conj", THPVariable_conj, METH_NOARGS, nullptr},
17462:   {"conj_physical", THPVariable_conj_physical, METH_NOARGS, nullptr},
17463:   {"conj_physical_", THPVariable_conj_physical_, METH_NOARGS, nullptr},
17464:   {"copysign", castPyCFunctionWithKeywords(THPVariable_copysign), METH_VARARGS | METH_KEYWORDS, nullptr},
17465:   {"copysign_", castPyCFunctionWithKeywords(THPVariable_copysign_), METH_VARARGS | METH_KEYWORDS, nullptr},
17466:   {"corrcoef", THPVariable_corrcoef, METH_NOARGS, nullptr},
17467:   {"cos", THPVariable_cos, METH_NOARGS, nullptr},
17468:   {"cos_", THPVariable_cos_, METH_NOARGS, nullptr},
17469:   {"cosh", THPVariable_cosh, METH_NOARGS, nullptr},
17470:   {"cosh_", THPVariable_cosh_, METH_NOARGS, nullptr},
17471:   {"count_nonzero", castPyCFunctionWithKeywords(THPVariable_count_nonzero), METH_VARARGS | METH_KEYWORDS, nullptr},
17472:   {"cov", castPyCFunctionWithKeywords(THPVariable_cov), METH_VARARGS | METH_KEYWORDS, nullptr},
17473:   {"cross", castPyCFunctionWithKeywords(THPVariable_cross), METH_VARARGS | METH_KEYWORDS, nullptr},
17474:   {"crow_indices", THPVariable_crow_indices, METH_NOARGS, nullptr},
17475:   {"cummax", castPyCFunctionWithKeywords(THPVariable_cummax), METH_VARARGS | METH_KEYWORDS, nullptr},
17476:   {"cummin", castPyCFunctionWithKeywords(THPVariable_cummin), METH_VARARGS | METH_KEYWORDS, nullptr},
17477:   {"cumprod", castPyCFunctionWithKeywords(THPVariable_cumprod), METH_VARARGS | METH_KEYWORDS, nullptr},
17478:   {"cumprod_", castPyCFunctionWithKeywords(THPVariable_cumprod_), METH_VARARGS | METH_KEYWORDS, nullptr},
17479:   {"cumsum", castPyCFunctionWithKeywords(THPVariable_cumsum), METH_VARARGS | METH_KEYWORDS, nullptr},
17480:   {"cumsum_", castPyCFunctionWithKeywords(THPVariable_cumsum_), METH_VARARGS | METH_KEYWORDS, nullptr},
17481:   {"deg2rad", THPVariable_deg2rad, METH_NOARGS, nullptr},
17482:   {"deg2rad_", THPVariable_deg2rad_, METH_NOARGS, nullptr},
17483:   {"dense_dim", THPVariable_dense_dim, METH_NOARGS, nullptr},
17484:   {"dequantize", THPVariable_dequantize, METH_NOARGS, nullptr},
17485:   {"det", THPVariable_det, METH_NOARGS, nullptr},
17486:   {"detach", THPVariable_detach, METH_NOARGS, nullptr},
17487:   {"detach_", THPVariable_detach_, METH_NOARGS, nullptr},
17488:   {"diag", castPyCFunctionWithKeywords(THPVariable_diag), METH_VARARGS | METH_KEYWORDS, nullptr},
17489:   {"diag_embed", castPyCFunctionWithKeywords(THPVariable_diag_embed), METH_VARARGS | METH_KEYWORDS, nullptr},
17490:   {"diagflat", castPyCFunctionWithKeywords(THPVariable_diagflat), METH_VARARGS | METH_KEYWORDS, nullptr},
17491:   {"diagonal", castPyCFunctionWithKeywords(THPVariable_diagonal), METH_VARARGS | METH_KEYWORDS, nullptr},
17492:   {"diagonal_scatter", castPyCFunctionWithKeywords(THPVariable_diagonal_scatter), METH_VARARGS | METH_KEYWORDS, nullptr},
17493:   {"diff", castPyCFunctionWithKeywords(THPVariable_diff), METH_VARARGS | METH_KEYWORDS, nullptr},
17494:   {"digamma", THPVariable_digamma, METH_NOARGS, nullptr},
17495:   {"digamma_", THPVariable_digamma_, METH_NOARGS, nullptr},
17496:   {"dist", castPyCFunctionWithKeywords(THPVariable_dist), METH_VARARGS | METH_KEYWORDS, nullptr},
17497:   {"div", castPyCFunctionWithKeywords(THPVariable_div), METH_VARARGS | METH_KEYWORDS, nullptr},
17498:   {"div_", castPyCFunctionWithKeywords(THPVariable_div_), METH_VARARGS | METH_KEYWORDS, nullptr},
17499:   {"divide", castPyCFunctionWithKeywords(THPVariable_divide), METH_VARARGS | METH_KEYWORDS, nullptr},
17500:   {"divide_", castPyCFunctionWithKeywords(THPVariable_divide_), METH_VARARGS | METH_KEYWORDS, nullptr},
17501:   {"dot", castPyCFunctionWithKeywords(THPVariable_dot), METH_VARARGS | METH_KEYWORDS, nullptr},
17502:   {"dsplit", castPyCFunctionWithKeywords(THPVariable_dsplit), METH_VARARGS | METH_KEYWORDS, nullptr},
17503:   {"eq", castPyCFunctionWithKeywords(THPVariable_eq), METH_VARARGS | METH_KEYWORDS, nullptr},
17504:   {"eq_", castPyCFunctionWithKeywords(THPVariable_eq_), METH_VARARGS | METH_KEYWORDS, nullptr},
17505:   {"equal", castPyCFunctionWithKeywords(THPVariable_equal), METH_VARARGS | METH_KEYWORDS, nullptr},
17506:   {"erf", THPVariable_erf, METH_NOARGS, nullptr},
17507:   {"erf_", THPVariable_erf_, METH_NOARGS, nullptr},
17508:   {"erfc", THPVariable_erfc, METH_NOARGS, nullptr},
17509:   {"erfc_", THPVariable_erfc_, METH_NOARGS, nullptr},
17510:   {"erfinv", THPVariable_erfinv, METH_NOARGS, nullptr},
17511:   {"erfinv_", THPVariable_erfinv_, METH_NOARGS, nullptr},
17512:   {"exp", THPVariable_exp, METH_NOARGS, nullptr},
17513:   {"exp2", THPVariable_exp2, METH_NOARGS, nullptr},
17514:   {"exp2_", THPVariable_exp2_, METH_NOARGS, nullptr},
17515:   {"exp_", THPVariable_exp_, METH_NOARGS, nullptr},
17516:   {"expand", castPyCFunctionWithKeywords(THPVariable_expand), METH_VARARGS | METH_KEYWORDS, nullptr},
17517:   {"expand_as", castPyCFunctionWithKeywords(THPVariable_expand_as), METH_VARARGS | METH_KEYWORDS, nullptr},
17518:   {"expm1", THPVariable_expm1, METH_NOARGS, nullptr},
17519:   {"expm1_", THPVariable_expm1_, METH_NOARGS, nullptr},
17520:   {"exponential_", castPyCFunctionWithKeywords(THPVariable_exponential_), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 17521-17640

```cpp
17521:   {"fill_", castPyCFunctionWithKeywords(THPVariable_fill_), METH_VARARGS | METH_KEYWORDS, nullptr},
17522:   {"fill_diagonal_", castPyCFunctionWithKeywords(THPVariable_fill_diagonal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17523:   {"fix", THPVariable_fix, METH_NOARGS, nullptr},
17524:   {"fix_", THPVariable_fix_, METH_NOARGS, nullptr},
17525:   {"flatten", castPyCFunctionWithKeywords(THPVariable_flatten), METH_VARARGS | METH_KEYWORDS, nullptr},
17526:   {"flip", castPyCFunctionWithKeywords(THPVariable_flip), METH_VARARGS | METH_KEYWORDS, nullptr},
17527:   {"fliplr", THPVariable_fliplr, METH_NOARGS, nullptr},
17528:   {"flipud", THPVariable_flipud, METH_NOARGS, nullptr},
17529:   {"float_power", castPyCFunctionWithKeywords(THPVariable_float_power), METH_VARARGS | METH_KEYWORDS, nullptr},
17530:   {"float_power_", castPyCFunctionWithKeywords(THPVariable_float_power_), METH_VARARGS | METH_KEYWORDS, nullptr},
17531:   {"floor", THPVariable_floor, METH_NOARGS, nullptr},
17532:   {"floor_", THPVariable_floor_, METH_NOARGS, nullptr},
17533:   {"floor_divide", castPyCFunctionWithKeywords(THPVariable_floor_divide), METH_VARARGS | METH_KEYWORDS, nullptr},
17534:   {"floor_divide_", castPyCFunctionWithKeywords(THPVariable_floor_divide_), METH_VARARGS | METH_KEYWORDS, nullptr},
17535:   {"fmax", castPyCFunctionWithKeywords(THPVariable_fmax), METH_VARARGS | METH_KEYWORDS, nullptr},
17536:   {"fmin", castPyCFunctionWithKeywords(THPVariable_fmin), METH_VARARGS | METH_KEYWORDS, nullptr},
17537:   {"fmod", castPyCFunctionWithKeywords(THPVariable_fmod), METH_VARARGS | METH_KEYWORDS, nullptr},
17538:   {"fmod_", castPyCFunctionWithKeywords(THPVariable_fmod_), METH_VARARGS | METH_KEYWORDS, nullptr},
17539:   {"frac", THPVariable_frac, METH_NOARGS, nullptr},
17540:   {"frac_", THPVariable_frac_, METH_NOARGS, nullptr},
17541:   {"frexp", THPVariable_frexp, METH_NOARGS, nullptr},
17542:   {"gather", castPyCFunctionWithKeywords(THPVariable_gather), METH_VARARGS | METH_KEYWORDS, nullptr},
17543:   {"gcd", castPyCFunctionWithKeywords(THPVariable_gcd), METH_VARARGS | METH_KEYWORDS, nullptr},
17544:   {"gcd_", castPyCFunctionWithKeywords(THPVariable_gcd_), METH_VARARGS | METH_KEYWORDS, nullptr},
17545:   {"ge", castPyCFunctionWithKeywords(THPVariable_ge), METH_VARARGS | METH_KEYWORDS, nullptr},
17546:   {"ge_", castPyCFunctionWithKeywords(THPVariable_ge_), METH_VARARGS | METH_KEYWORDS, nullptr},
17547:   {"geometric_", castPyCFunctionWithKeywords(THPVariable_geometric_), METH_VARARGS | METH_KEYWORDS, nullptr},
17548:   {"geqrf", THPVariable_geqrf, METH_NOARGS, nullptr},
17549:   {"ger", castPyCFunctionWithKeywords(THPVariable_ger), METH_VARARGS | METH_KEYWORDS, nullptr},
17550:   {"greater", castPyCFunctionWithKeywords(THPVariable_greater), METH_VARARGS | METH_KEYWORDS, nullptr},
17551:   {"greater_", castPyCFunctionWithKeywords(THPVariable_greater_), METH_VARARGS | METH_KEYWORDS, nullptr},
17552:   {"greater_equal", castPyCFunctionWithKeywords(THPVariable_greater_equal), METH_VARARGS | METH_KEYWORDS, nullptr},
17553:   {"greater_equal_", castPyCFunctionWithKeywords(THPVariable_greater_equal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17554:   {"gt", castPyCFunctionWithKeywords(THPVariable_gt), METH_VARARGS | METH_KEYWORDS, nullptr},
17555:   {"gt_", castPyCFunctionWithKeywords(THPVariable_gt_), METH_VARARGS | METH_KEYWORDS, nullptr},
17556:   {"hardshrink", castPyCFunctionWithKeywords(THPVariable_hardshrink), METH_VARARGS | METH_KEYWORDS, nullptr},
17557:   {"hash_tensor", castPyCFunctionWithKeywords(THPVariable_hash_tensor), METH_VARARGS | METH_KEYWORDS, nullptr},
17558:   {"heaviside", castPyCFunctionWithKeywords(THPVariable_heaviside), METH_VARARGS | METH_KEYWORDS, nullptr},
17559:   {"heaviside_", castPyCFunctionWithKeywords(THPVariable_heaviside_), METH_VARARGS | METH_KEYWORDS, nullptr},
17560:   {"histc", castPyCFunctionWithKeywords(THPVariable_histc), METH_VARARGS | METH_KEYWORDS, nullptr},
17561:   {"histogram", castPyCFunctionWithKeywords(THPVariable_histogram), METH_VARARGS | METH_KEYWORDS, nullptr},
17562:   {"hsplit", castPyCFunctionWithKeywords(THPVariable_hsplit), METH_VARARGS | METH_KEYWORDS, nullptr},
17563:   {"hypot", castPyCFunctionWithKeywords(THPVariable_hypot), METH_VARARGS | METH_KEYWORDS, nullptr},
17564:   {"hypot_", castPyCFunctionWithKeywords(THPVariable_hypot_), METH_VARARGS | METH_KEYWORDS, nullptr},
17565:   {"i0", THPVariable_i0, METH_NOARGS, nullptr},
17566:   {"i0_", THPVariable_i0_, METH_NOARGS, nullptr},
17567:   {"igamma", castPyCFunctionWithKeywords(THPVariable_igamma), METH_VARARGS | METH_KEYWORDS, nullptr},
17568:   {"igamma_", castPyCFunctionWithKeywords(THPVariable_igamma_), METH_VARARGS | METH_KEYWORDS, nullptr},
17569:   {"igammac", castPyCFunctionWithKeywords(THPVariable_igammac), METH_VARARGS | METH_KEYWORDS, nullptr},
17570:   {"igammac_", castPyCFunctionWithKeywords(THPVariable_igammac_), METH_VARARGS | METH_KEYWORDS, nullptr},
17571:   {"index_add", castPyCFunctionWithKeywords(THPVariable_index_add), METH_VARARGS | METH_KEYWORDS, nullptr},
17572:   {"index_add_", castPyCFunctionWithKeywords(THPVariable_index_add_), METH_VARARGS | METH_KEYWORDS, nullptr},
17573:   {"index_copy", castPyCFunctionWithKeywords(THPVariable_index_copy), METH_VARARGS | METH_KEYWORDS, nullptr},
17574:   {"index_copy_", castPyCFunctionWithKeywords(THPVariable_index_copy_), METH_VARARGS | METH_KEYWORDS, nullptr},
17575:   {"index_fill", castPyCFunctionWithKeywords(THPVariable_index_fill), METH_VARARGS | METH_KEYWORDS, nullptr},
17576:   {"index_fill_", castPyCFunctionWithKeywords(THPVariable_index_fill_), METH_VARARGS | METH_KEYWORDS, nullptr},
17577:   {"index_put", castPyCFunctionWithKeywords(THPVariable_index_put), METH_VARARGS | METH_KEYWORDS, nullptr},
17578:   {"index_put_", castPyCFunctionWithKeywords(THPVariable_index_put_), METH_VARARGS | METH_KEYWORDS, nullptr},
17579:   {"index_reduce", castPyCFunctionWithKeywords(THPVariable_index_reduce), METH_VARARGS | METH_KEYWORDS, nullptr},
17580:   {"index_reduce_", castPyCFunctionWithKeywords(THPVariable_index_reduce_), METH_VARARGS | METH_KEYWORDS, nullptr},
17581:   {"index_select", castPyCFunctionWithKeywords(THPVariable_index_select), METH_VARARGS | METH_KEYWORDS, nullptr},
17582:   {"indices", THPVariable_indices, METH_NOARGS, nullptr},
17583:   {"inner", castPyCFunctionWithKeywords(THPVariable_inner), METH_VARARGS | METH_KEYWORDS, nullptr},
17584:   {"int_repr", THPVariable_int_repr, METH_NOARGS, nullptr},
17585:   {"inverse", THPVariable_inverse, METH_NOARGS, nullptr},
17586:   {"is_coalesced", THPVariable_is_coalesced, METH_NOARGS, nullptr},
17587:   {"is_complex", THPVariable_is_complex, METH_NOARGS, nullptr},
17588:   {"is_conj", THPVariable_is_conj, METH_NOARGS, nullptr},
17589:   {"is_distributed", THPVariable_is_distributed, METH_NOARGS, nullptr},
17590:   {"is_floating_point", THPVariable_is_floating_point, METH_NOARGS, nullptr},
17591:   {"is_inference", THPVariable_is_inference, METH_NOARGS, nullptr},
17592:   {"is_neg", THPVariable_is_neg, METH_NOARGS, nullptr},
17593:   {"is_nonzero", THPVariable_is_nonzero, METH_NOARGS, nullptr},
17594:   {"is_pinned", castPyCFunctionWithKeywords(THPVariable_is_pinned), METH_VARARGS | METH_KEYWORDS, nullptr},
17595:   {"is_same_size", castPyCFunctionWithKeywords(THPVariable_is_same_size), METH_VARARGS | METH_KEYWORDS, nullptr},
17596:   {"is_set_to", castPyCFunctionWithKeywords(THPVariable_is_set_to), METH_VARARGS | METH_KEYWORDS, nullptr},
17597:   {"is_signed", THPVariable_is_signed, METH_NOARGS, nullptr},
17598:   {"isclose", castPyCFunctionWithKeywords(THPVariable_isclose), METH_VARARGS | METH_KEYWORDS, nullptr},
17599:   {"isfinite", THPVariable_isfinite, METH_NOARGS, nullptr},
17600:   {"isinf", THPVariable_isinf, METH_NOARGS, nullptr},
17601:   {"isnan", THPVariable_isnan, METH_NOARGS, nullptr},
17602:   {"isneginf", THPVariable_isneginf, METH_NOARGS, nullptr},
17603:   {"isposinf", THPVariable_isposinf, METH_NOARGS, nullptr},
17604:   {"isreal", THPVariable_isreal, METH_NOARGS, nullptr},
17605:   {"istft", castPyCFunctionWithKeywords(THPVariable_istft), METH_VARARGS | METH_KEYWORDS, nullptr},
17606:   {"kron", castPyCFunctionWithKeywords(THPVariable_kron), METH_VARARGS | METH_KEYWORDS, nullptr},
17607:   {"kthvalue", castPyCFunctionWithKeywords(THPVariable_kthvalue), METH_VARARGS | METH_KEYWORDS, nullptr},
17608:   {"lcm", castPyCFunctionWithKeywords(THPVariable_lcm), METH_VARARGS | METH_KEYWORDS, nullptr},
17609:   {"lcm_", castPyCFunctionWithKeywords(THPVariable_lcm_), METH_VARARGS | METH_KEYWORDS, nullptr},
17610:   {"ldexp", castPyCFunctionWithKeywords(THPVariable_ldexp), METH_VARARGS | METH_KEYWORDS, nullptr},
17611:   {"ldexp_", castPyCFunctionWithKeywords(THPVariable_ldexp_), METH_VARARGS | METH_KEYWORDS, nullptr},
17612:   {"le", castPyCFunctionWithKeywords(THPVariable_le), METH_VARARGS | METH_KEYWORDS, nullptr},
17613:   {"le_", castPyCFunctionWithKeywords(THPVariable_le_), METH_VARARGS | METH_KEYWORDS, nullptr},
17614:   {"lerp", castPyCFunctionWithKeywords(THPVariable_lerp), METH_VARARGS | METH_KEYWORDS, nullptr},
17615:   {"lerp_", castPyCFunctionWithKeywords(THPVariable_lerp_), METH_VARARGS | METH_KEYWORDS, nullptr},
17616:   {"less", castPyCFunctionWithKeywords(THPVariable_less), METH_VARARGS | METH_KEYWORDS, nullptr},
17617:   {"less_", castPyCFunctionWithKeywords(THPVariable_less_), METH_VARARGS | METH_KEYWORDS, nullptr},
17618:   {"less_equal", castPyCFunctionWithKeywords(THPVariable_less_equal), METH_VARARGS | METH_KEYWORDS, nullptr},
17619:   {"less_equal_", castPyCFunctionWithKeywords(THPVariable_less_equal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17620:   {"lgamma", THPVariable_lgamma, METH_NOARGS, nullptr},
17621:   {"lgamma_", THPVariable_lgamma_, METH_NOARGS, nullptr},
17622:   {"log", THPVariable_log, METH_NOARGS, nullptr},
17623:   {"log10", THPVariable_log10, METH_NOARGS, nullptr},
17624:   {"log10_", THPVariable_log10_, METH_NOARGS, nullptr},
17625:   {"log1p", THPVariable_log1p, METH_NOARGS, nullptr},
17626:   {"log1p_", THPVariable_log1p_, METH_NOARGS, nullptr},
17627:   {"log2", THPVariable_log2, METH_NOARGS, nullptr},
17628:   {"log2_", THPVariable_log2_, METH_NOARGS, nullptr},
17629:   {"log_", THPVariable_log_, METH_NOARGS, nullptr},
17630:   {"log_normal_", castPyCFunctionWithKeywords(THPVariable_log_normal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17631:   {"log_softmax", castPyCFunctionWithKeywords(THPVariable_log_softmax), METH_VARARGS | METH_KEYWORDS, nullptr},
17632:   {"logaddexp", castPyCFunctionWithKeywords(THPVariable_logaddexp), METH_VARARGS | METH_KEYWORDS, nullptr},
17633:   {"logaddexp2", castPyCFunctionWithKeywords(THPVariable_logaddexp2), METH_VARARGS | METH_KEYWORDS, nullptr},
17634:   {"logcumsumexp", castPyCFunctionWithKeywords(THPVariable_logcumsumexp), METH_VARARGS | METH_KEYWORDS, nullptr},
17635:   {"logdet", THPVariable_logdet, METH_NOARGS, nullptr},
17636:   {"logical_and", castPyCFunctionWithKeywords(THPVariable_logical_and), METH_VARARGS | METH_KEYWORDS, nullptr},
17637:   {"logical_and_", castPyCFunctionWithKeywords(THPVariable_logical_and_), METH_VARARGS | METH_KEYWORDS, nullptr},
17638:   {"logical_not", THPVariable_logical_not, METH_NOARGS, nullptr},
17639:   {"logical_not_", THPVariable_logical_not_, METH_NOARGS, nullptr},
17640:   {"logical_or", castPyCFunctionWithKeywords(THPVariable_logical_or), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 17641-17760

```cpp
17641:   {"logical_or_", castPyCFunctionWithKeywords(THPVariable_logical_or_), METH_VARARGS | METH_KEYWORDS, nullptr},
17642:   {"logical_xor", castPyCFunctionWithKeywords(THPVariable_logical_xor), METH_VARARGS | METH_KEYWORDS, nullptr},
17643:   {"logical_xor_", castPyCFunctionWithKeywords(THPVariable_logical_xor_), METH_VARARGS | METH_KEYWORDS, nullptr},
17644:   {"logit", castPyCFunctionWithKeywords(THPVariable_logit), METH_VARARGS | METH_KEYWORDS, nullptr},
17645:   {"logit_", castPyCFunctionWithKeywords(THPVariable_logit_), METH_VARARGS | METH_KEYWORDS, nullptr},
17646:   {"logsumexp", castPyCFunctionWithKeywords(THPVariable_logsumexp), METH_VARARGS | METH_KEYWORDS, nullptr},
17647:   {"lt", castPyCFunctionWithKeywords(THPVariable_lt), METH_VARARGS | METH_KEYWORDS, nullptr},
17648:   {"lt_", castPyCFunctionWithKeywords(THPVariable_lt_), METH_VARARGS | METH_KEYWORDS, nullptr},
17649:   {"lu_solve", castPyCFunctionWithKeywords(THPVariable_lu_solve), METH_VARARGS | METH_KEYWORDS, nullptr},
17650:   {"masked_fill", castPyCFunctionWithKeywords(THPVariable_masked_fill), METH_VARARGS | METH_KEYWORDS, nullptr},
17651:   {"masked_fill_", castPyCFunctionWithKeywords(THPVariable_masked_fill_), METH_VARARGS | METH_KEYWORDS, nullptr},
17652:   {"masked_scatter", castPyCFunctionWithKeywords(THPVariable_masked_scatter), METH_VARARGS | METH_KEYWORDS, nullptr},
17653:   {"masked_scatter_", castPyCFunctionWithKeywords(THPVariable_masked_scatter_), METH_VARARGS | METH_KEYWORDS, nullptr},
17654:   {"masked_select", castPyCFunctionWithKeywords(THPVariable_masked_select), METH_VARARGS | METH_KEYWORDS, nullptr},
17655:   {"matmul", castPyCFunctionWithKeywords(THPVariable_matmul), METH_VARARGS | METH_KEYWORDS, nullptr},
17656:   {"matrix_exp", THPVariable_matrix_exp, METH_NOARGS, nullptr},
17657:   {"matrix_power", castPyCFunctionWithKeywords(THPVariable_matrix_power), METH_VARARGS | METH_KEYWORDS, nullptr},
17658:   {"max", castPyCFunctionWithKeywords(THPVariable_max), METH_VARARGS | METH_KEYWORDS, nullptr},
17659:   {"maximum", castPyCFunctionWithKeywords(THPVariable_maximum), METH_VARARGS | METH_KEYWORDS, nullptr},
17660:   {"mean", castPyCFunctionWithKeywords(THPVariable_mean), METH_VARARGS | METH_KEYWORDS, nullptr},
17661:   {"median", castPyCFunctionWithKeywords(THPVariable_median), METH_VARARGS | METH_KEYWORDS, nullptr},
17662:   {"min", castPyCFunctionWithKeywords(THPVariable_min), METH_VARARGS | METH_KEYWORDS, nullptr},
17663:   {"minimum", castPyCFunctionWithKeywords(THPVariable_minimum), METH_VARARGS | METH_KEYWORDS, nullptr},
17664:   {"mm", castPyCFunctionWithKeywords(THPVariable_mm), METH_VARARGS | METH_KEYWORDS, nullptr},
17665:   {"mode", castPyCFunctionWithKeywords(THPVariable_mode), METH_VARARGS | METH_KEYWORDS, nullptr},
17666:   {"moveaxis", castPyCFunctionWithKeywords(THPVariable_moveaxis), METH_VARARGS | METH_KEYWORDS, nullptr},
17667:   {"movedim", castPyCFunctionWithKeywords(THPVariable_movedim), METH_VARARGS | METH_KEYWORDS, nullptr},
17668:   {"msort", THPVariable_msort, METH_NOARGS, nullptr},
17669:   {"mul", castPyCFunctionWithKeywords(THPVariable_mul), METH_VARARGS | METH_KEYWORDS, nullptr},
17670:   {"mul_", castPyCFunctionWithKeywords(THPVariable_mul_), METH_VARARGS | METH_KEYWORDS, nullptr},
17671:   {"multinomial", castPyCFunctionWithKeywords(THPVariable_multinomial), METH_VARARGS | METH_KEYWORDS, nullptr},
17672:   {"multiply", castPyCFunctionWithKeywords(THPVariable_multiply), METH_VARARGS | METH_KEYWORDS, nullptr},
17673:   {"multiply_", castPyCFunctionWithKeywords(THPVariable_multiply_), METH_VARARGS | METH_KEYWORDS, nullptr},
17674:   {"mv", castPyCFunctionWithKeywords(THPVariable_mv), METH_VARARGS | METH_KEYWORDS, nullptr},
17675:   {"mvlgamma", castPyCFunctionWithKeywords(THPVariable_mvlgamma), METH_VARARGS | METH_KEYWORDS, nullptr},
17676:   {"mvlgamma_", castPyCFunctionWithKeywords(THPVariable_mvlgamma_), METH_VARARGS | METH_KEYWORDS, nullptr},
17677:   {"nan_to_num", castPyCFunctionWithKeywords(THPVariable_nan_to_num), METH_VARARGS | METH_KEYWORDS, nullptr},
17678:   {"nan_to_num_", castPyCFunctionWithKeywords(THPVariable_nan_to_num_), METH_VARARGS | METH_KEYWORDS, nullptr},
17679:   {"nanmean", castPyCFunctionWithKeywords(THPVariable_nanmean), METH_VARARGS | METH_KEYWORDS, nullptr},
17680:   {"nanmedian", castPyCFunctionWithKeywords(THPVariable_nanmedian), METH_VARARGS | METH_KEYWORDS, nullptr},
17681:   {"nanquantile", castPyCFunctionWithKeywords(THPVariable_nanquantile), METH_VARARGS | METH_KEYWORDS, nullptr},
17682:   {"nansum", castPyCFunctionWithKeywords(THPVariable_nansum), METH_VARARGS | METH_KEYWORDS, nullptr},
17683:   {"narrow", castPyCFunctionWithKeywords(THPVariable_narrow), METH_VARARGS | METH_KEYWORDS, nullptr},
17684:   {"narrow_copy", castPyCFunctionWithKeywords(THPVariable_narrow_copy), METH_VARARGS | METH_KEYWORDS, nullptr},
17685:   {"ne", castPyCFunctionWithKeywords(THPVariable_ne), METH_VARARGS | METH_KEYWORDS, nullptr},
17686:   {"ne_", castPyCFunctionWithKeywords(THPVariable_ne_), METH_VARARGS | METH_KEYWORDS, nullptr},
17687:   {"neg", THPVariable_neg, METH_NOARGS, nullptr},
17688:   {"neg_", THPVariable_neg_, METH_NOARGS, nullptr},
17689:   {"negative", THPVariable_negative, METH_NOARGS, nullptr},
17690:   {"negative_", THPVariable_negative_, METH_NOARGS, nullptr},
17691:   {"new_empty", castPyCFunctionWithKeywords(THPVariable_new_empty), METH_VARARGS | METH_KEYWORDS, nullptr},
17692:   {"new_empty_strided", castPyCFunctionWithKeywords(THPVariable_new_empty_strided), METH_VARARGS | METH_KEYWORDS, nullptr},
17693:   {"new_full", castPyCFunctionWithKeywords(THPVariable_new_full), METH_VARARGS | METH_KEYWORDS, nullptr},
17694:   {"new_ones", castPyCFunctionWithKeywords(THPVariable_new_ones), METH_VARARGS | METH_KEYWORDS, nullptr},
17695:   {"new_zeros", castPyCFunctionWithKeywords(THPVariable_new_zeros), METH_VARARGS | METH_KEYWORDS, nullptr},
17696:   {"nextafter", castPyCFunctionWithKeywords(THPVariable_nextafter), METH_VARARGS | METH_KEYWORDS, nullptr},
17697:   {"nextafter_", castPyCFunctionWithKeywords(THPVariable_nextafter_), METH_VARARGS | METH_KEYWORDS, nullptr},
17698:   {"nonzero_static", castPyCFunctionWithKeywords(THPVariable_nonzero_static), METH_VARARGS | METH_KEYWORDS, nullptr},
17699:   {"norm", castPyCFunctionWithKeywords(THPVariable_norm), METH_VARARGS | METH_KEYWORDS, nullptr},
17700:   {"normal_", castPyCFunctionWithKeywords(THPVariable_normal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17701:   {"not_equal", castPyCFunctionWithKeywords(THPVariable_not_equal), METH_VARARGS | METH_KEYWORDS, nullptr},
17702:   {"not_equal_", castPyCFunctionWithKeywords(THPVariable_not_equal_), METH_VARARGS | METH_KEYWORDS, nullptr},
17703:   {"orgqr", castPyCFunctionWithKeywords(THPVariable_orgqr), METH_VARARGS | METH_KEYWORDS, nullptr},
17704:   {"ormqr", castPyCFunctionWithKeywords(THPVariable_ormqr), METH_VARARGS | METH_KEYWORDS, nullptr},
17705:   {"outer", castPyCFunctionWithKeywords(THPVariable_outer), METH_VARARGS | METH_KEYWORDS, nullptr},
17706:   {"permute", castPyCFunctionWithKeywords(THPVariable_permute), METH_VARARGS | METH_KEYWORDS, nullptr},
17707:   {"pin_memory", castPyCFunctionWithKeywords(THPVariable_pin_memory), METH_VARARGS | METH_KEYWORDS, nullptr},
17708:   {"pinverse", castPyCFunctionWithKeywords(THPVariable_pinverse), METH_VARARGS | METH_KEYWORDS, nullptr},
17709:   {"polygamma", castPyCFunctionWithKeywords(THPVariable_polygamma), METH_VARARGS | METH_KEYWORDS, nullptr},
17710:   {"polygamma_", castPyCFunctionWithKeywords(THPVariable_polygamma_), METH_VARARGS | METH_KEYWORDS, nullptr},
17711:   {"positive", THPVariable_positive, METH_NOARGS, nullptr},
17712:   {"pow", castPyCFunctionWithKeywords(THPVariable_pow), METH_VARARGS | METH_KEYWORDS, nullptr},
17713:   {"pow_", castPyCFunctionWithKeywords(THPVariable_pow_), METH_VARARGS | METH_KEYWORDS, nullptr},
17714:   {"prelu", castPyCFunctionWithKeywords(THPVariable_prelu), METH_VARARGS | METH_KEYWORDS, nullptr},
17715:   {"prod", castPyCFunctionWithKeywords(THPVariable_prod), METH_VARARGS | METH_KEYWORDS, nullptr},
17716:   {"put", castPyCFunctionWithKeywords(THPVariable_put), METH_VARARGS | METH_KEYWORDS, nullptr},
17717:   {"put_", castPyCFunctionWithKeywords(THPVariable_put_), METH_VARARGS | METH_KEYWORDS, nullptr},
17718:   {"q_per_channel_axis", THPVariable_q_per_channel_axis, METH_NOARGS, nullptr},
17719:   {"q_per_channel_scales", THPVariable_q_per_channel_scales, METH_NOARGS, nullptr},
17720:   {"q_per_channel_zero_points", THPVariable_q_per_channel_zero_points, METH_NOARGS, nullptr},
17721:   {"q_scale", THPVariable_q_scale, METH_NOARGS, nullptr},
17722:   {"q_zero_point", THPVariable_q_zero_point, METH_NOARGS, nullptr},
17723:   {"qr", castPyCFunctionWithKeywords(THPVariable_qr), METH_VARARGS | METH_KEYWORDS, nullptr},
17724:   {"qscheme", THPVariable_qscheme, METH_NOARGS, nullptr},
17725:   {"quantile", castPyCFunctionWithKeywords(THPVariable_quantile), METH_VARARGS | METH_KEYWORDS, nullptr},
17726:   {"rad2deg", THPVariable_rad2deg, METH_NOARGS, nullptr},
17727:   {"rad2deg_", THPVariable_rad2deg_, METH_NOARGS, nullptr},
17728:   {"random_", castPyCFunctionWithKeywords(THPVariable_random_), METH_VARARGS | METH_KEYWORDS, nullptr},
17729:   {"ravel", THPVariable_ravel, METH_NOARGS, nullptr},
17730:   {"reciprocal", THPVariable_reciprocal, METH_NOARGS, nullptr},
17731:   {"reciprocal_", THPVariable_reciprocal_, METH_NOARGS, nullptr},
17732:   {"record_stream", castPyCFunctionWithKeywords(THPVariable_record_stream), METH_VARARGS | METH_KEYWORDS, nullptr},
17733:   {"refine_names", castPyCFunctionWithKeywords(THPVariable_refine_names), METH_VARARGS | METH_KEYWORDS, nullptr},
17734:   {"relu", THPVariable_relu, METH_NOARGS, nullptr},
17735:   {"relu_", THPVariable_relu_, METH_NOARGS, nullptr},
17736:   {"remainder", castPyCFunctionWithKeywords(THPVariable_remainder), METH_VARARGS | METH_KEYWORDS, nullptr},
17737:   {"remainder_", castPyCFunctionWithKeywords(THPVariable_remainder_), METH_VARARGS | METH_KEYWORDS, nullptr},
17738:   {"rename", castPyCFunctionWithKeywords(THPVariable_rename), METH_VARARGS | METH_KEYWORDS, nullptr},
17739:   {"rename_", castPyCFunctionWithKeywords(THPVariable_rename_), METH_VARARGS | METH_KEYWORDS, nullptr},
17740:   {"renorm", castPyCFunctionWithKeywords(THPVariable_renorm), METH_VARARGS | METH_KEYWORDS, nullptr},
17741:   {"renorm_", castPyCFunctionWithKeywords(THPVariable_renorm_), METH_VARARGS | METH_KEYWORDS, nullptr},
17742:   {"repeat", castPyCFunctionWithKeywords(THPVariable_repeat), METH_VARARGS | METH_KEYWORDS, nullptr},
17743:   {"repeat_interleave", castPyCFunctionWithKeywords(THPVariable_repeat_interleave), METH_VARARGS | METH_KEYWORDS, nullptr},
17744:   {"reshape", castPyCFunctionWithKeywords(THPVariable_reshape), METH_VARARGS | METH_KEYWORDS, nullptr},
17745:   {"reshape_as", castPyCFunctionWithKeywords(THPVariable_reshape_as), METH_VARARGS | METH_KEYWORDS, nullptr},
17746:   {"resize_", castPyCFunctionWithKeywords(THPVariable_resize_), METH_VARARGS | METH_KEYWORDS, nullptr},
17747:   {"resize_as_", castPyCFunctionWithKeywords(THPVariable_resize_as_), METH_VARARGS | METH_KEYWORDS, nullptr},
17748:   {"resize_as_sparse_", castPyCFunctionWithKeywords(THPVariable_resize_as_sparse_), METH_VARARGS | METH_KEYWORDS, nullptr},
17749:   {"resolve_conj", THPVariable_resolve_conj, METH_NOARGS, nullptr},
17750:   {"resolve_neg", THPVariable_resolve_neg, METH_NOARGS, nullptr},
17751:   {"retain_grad", THPVariable_retain_grad, METH_NOARGS, nullptr},
17752:   {"roll", castPyCFunctionWithKeywords(THPVariable_roll), METH_VARARGS | METH_KEYWORDS, nullptr},
17753:   {"rot90", castPyCFunctionWithKeywords(THPVariable_rot90), METH_VARARGS | METH_KEYWORDS, nullptr},
17754:   {"round", castPyCFunctionWithKeywords(THPVariable_round), METH_VARARGS | METH_KEYWORDS, nullptr},
17755:   {"round_", castPyCFunctionWithKeywords(THPVariable_round_), METH_VARARGS | METH_KEYWORDS, nullptr},
17756:   {"row_indices", THPVariable_row_indices, METH_NOARGS, nullptr},
17757:   {"rsqrt", THPVariable_rsqrt, METH_NOARGS, nullptr},
17758:   {"rsqrt_", THPVariable_rsqrt_, METH_NOARGS, nullptr},
17759:   {"scatter", castPyCFunctionWithKeywords(THPVariable_scatter), METH_VARARGS | METH_KEYWORDS, nullptr},
17760:   {"scatter_", castPyCFunctionWithKeywords(THPVariable_scatter_), METH_VARARGS | METH_KEYWORDS, nullptr},
```

- EN: The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。
### Lines 17761-17866

```cpp
17761:   {"scatter_add", castPyCFunctionWithKeywords(THPVariable_scatter_add), METH_VARARGS | METH_KEYWORDS, nullptr},
17762:   {"scatter_add_", castPyCFunctionWithKeywords(THPVariable_scatter_add_), METH_VARARGS | METH_KEYWORDS, nullptr},
17763:   {"scatter_reduce", castPyCFunctionWithKeywords(THPVariable_scatter_reduce), METH_VARARGS | METH_KEYWORDS, nullptr},
17764:   {"scatter_reduce_", castPyCFunctionWithKeywords(THPVariable_scatter_reduce_), METH_VARARGS | METH_KEYWORDS, nullptr},
17765:   {"select", castPyCFunctionWithKeywords(THPVariable_select), METH_VARARGS | METH_KEYWORDS, nullptr},
17766:   {"select_scatter", castPyCFunctionWithKeywords(THPVariable_select_scatter), METH_VARARGS | METH_KEYWORDS, nullptr},
17767:   {"sgn", THPVariable_sgn, METH_NOARGS, nullptr},
17768:   {"sgn_", THPVariable_sgn_, METH_NOARGS, nullptr},
17769:   {"sigmoid", THPVariable_sigmoid, METH_NOARGS, nullptr},
17770:   {"sigmoid_", THPVariable_sigmoid_, METH_NOARGS, nullptr},
17771:   {"sign", THPVariable_sign, METH_NOARGS, nullptr},
17772:   {"sign_", THPVariable_sign_, METH_NOARGS, nullptr},
17773:   {"signbit", THPVariable_signbit, METH_NOARGS, nullptr},
17774:   {"sin", THPVariable_sin, METH_NOARGS, nullptr},
17775:   {"sin_", THPVariable_sin_, METH_NOARGS, nullptr},
17776:   {"sinc", THPVariable_sinc, METH_NOARGS, nullptr},
17777:   {"sinc_", THPVariable_sinc_, METH_NOARGS, nullptr},
17778:   {"sinh", THPVariable_sinh, METH_NOARGS, nullptr},
17779:   {"sinh_", THPVariable_sinh_, METH_NOARGS, nullptr},
17780:   {"slice_inverse", castPyCFunctionWithKeywords(THPVariable_slice_inverse), METH_VARARGS | METH_KEYWORDS, nullptr},
17781:   {"slice_scatter", castPyCFunctionWithKeywords(THPVariable_slice_scatter), METH_VARARGS | METH_KEYWORDS, nullptr},
17782:   {"slogdet", THPVariable_slogdet, METH_NOARGS, nullptr},
17783:   {"smm", castPyCFunctionWithKeywords(THPVariable_smm), METH_VARARGS | METH_KEYWORDS, nullptr},
17784:   {"softmax", castPyCFunctionWithKeywords(THPVariable_softmax), METH_VARARGS | METH_KEYWORDS, nullptr},
17785:   {"sort", castPyCFunctionWithKeywords(THPVariable_sort), METH_VARARGS | METH_KEYWORDS, nullptr},
17786:   {"sparse_dim", THPVariable_sparse_dim, METH_NOARGS, nullptr},
17787:   {"sparse_mask", castPyCFunctionWithKeywords(THPVariable_sparse_mask), METH_VARARGS | METH_KEYWORDS, nullptr},
17788:   {"sparse_resize_", castPyCFunctionWithKeywords(THPVariable_sparse_resize_), METH_VARARGS | METH_KEYWORDS, nullptr},
17789:   {"sparse_resize_and_clear_", castPyCFunctionWithKeywords(THPVariable_sparse_resize_and_clear_), METH_VARARGS | METH_KEYWORDS, nullptr},
17790:   {"split", castPyCFunctionWithKeywords(THPVariable_split), METH_VARARGS | METH_KEYWORDS, nullptr},
17791:   {"split_with_sizes", castPyCFunctionWithKeywords(THPVariable_split_with_sizes), METH_VARARGS | METH_KEYWORDS, nullptr},
17792:   {"sqrt", THPVariable_sqrt, METH_NOARGS, nullptr},
17793:   {"sqrt_", THPVariable_sqrt_, METH_NOARGS, nullptr},
17794:   {"square", THPVariable_square, METH_NOARGS, nullptr},
17795:   {"square_", THPVariable_square_, METH_NOARGS, nullptr},
17796:   {"squeeze", castPyCFunctionWithKeywords(THPVariable_squeeze), METH_VARARGS | METH_KEYWORDS, nullptr},
17797:   {"squeeze_", castPyCFunctionWithKeywords(THPVariable_squeeze_), METH_VARARGS | METH_KEYWORDS, nullptr},
17798:   {"sspaddmm", castPyCFunctionWithKeywords(THPVariable_sspaddmm), METH_VARARGS | METH_KEYWORDS, nullptr},
17799:   {"std", castPyCFunctionWithKeywords(THPVariable_std), METH_VARARGS | METH_KEYWORDS, nullptr},
17800:   {"stft", castPyCFunctionWithKeywords(THPVariable_stft), METH_VARARGS | METH_KEYWORDS, nullptr},
17801:   {"sub", castPyCFunctionWithKeywords(THPVariable_sub), METH_VARARGS | METH_KEYWORDS, nullptr},
17802:   {"sub_", castPyCFunctionWithKeywords(THPVariable_sub_), METH_VARARGS | METH_KEYWORDS, nullptr},
17803:   {"subtract", castPyCFunctionWithKeywords(THPVariable_subtract), METH_VARARGS | METH_KEYWORDS, nullptr},
17804:   {"subtract_", castPyCFunctionWithKeywords(THPVariable_subtract_), METH_VARARGS | METH_KEYWORDS, nullptr},
17805:   {"sum", castPyCFunctionWithKeywords(THPVariable_sum), METH_VARARGS | METH_KEYWORDS, nullptr},
17806:   {"sum_to_size", castPyCFunctionWithKeywords(THPVariable_sum_to_size), METH_VARARGS | METH_KEYWORDS, nullptr},
17807:   {"svd", castPyCFunctionWithKeywords(THPVariable_svd), METH_VARARGS | METH_KEYWORDS, nullptr},
17808:   {"swapaxes", castPyCFunctionWithKeywords(THPVariable_swapaxes), METH_VARARGS | METH_KEYWORDS, nullptr},
17809:   {"swapaxes_", castPyCFunctionWithKeywords(THPVariable_swapaxes_), METH_VARARGS | METH_KEYWORDS, nullptr},
17810:   {"swapdims", castPyCFunctionWithKeywords(THPVariable_swapdims), METH_VARARGS | METH_KEYWORDS, nullptr},
17811:   {"swapdims_", castPyCFunctionWithKeywords(THPVariable_swapdims_), METH_VARARGS | METH_KEYWORDS, nullptr},
17812:   {"t", THPVariable_t, METH_NOARGS, nullptr},
17813:   {"t_", THPVariable_t_, METH_NOARGS, nullptr},
17814:   {"take", castPyCFunctionWithKeywords(THPVariable_take), METH_VARARGS | METH_KEYWORDS, nullptr},
17815:   {"take_along_dim", castPyCFunctionWithKeywords(THPVariable_take_along_dim), METH_VARARGS | METH_KEYWORDS, nullptr},
17816:   {"tan", THPVariable_tan, METH_NOARGS, nullptr},
17817:   {"tan_", THPVariable_tan_, METH_NOARGS, nullptr},
17818:   {"tanh", THPVariable_tanh, METH_NOARGS, nullptr},
17819:   {"tanh_", THPVariable_tanh_, METH_NOARGS, nullptr},
17820:   {"tensor_split", castPyCFunctionWithKeywords(THPVariable_tensor_split), METH_VARARGS | METH_KEYWORDS, nullptr},
17821:   {"tile", castPyCFunctionWithKeywords(THPVariable_tile), METH_VARARGS | METH_KEYWORDS, nullptr},
17822:   {"to_dense", castPyCFunctionWithKeywords(THPVariable_to_dense), METH_VARARGS | METH_KEYWORDS, nullptr},
17823:   {"to_mkldnn", castPyCFunctionWithKeywords(THPVariable_to_mkldnn), METH_VARARGS | METH_KEYWORDS, nullptr},
17824:   {"to_padded_tensor", castPyCFunctionWithKeywords(THPVariable_to_padded_tensor), METH_VARARGS | METH_KEYWORDS, nullptr},
17825:   {"to_sparse", castPyCFunctionWithKeywords(THPVariable_to_sparse), METH_VARARGS | METH_KEYWORDS, nullptr},
17826:   {"to_sparse_bsc", castPyCFunctionWithKeywords(THPVariable_to_sparse_bsc), METH_VARARGS | METH_KEYWORDS, nullptr},
17827:   {"to_sparse_bsr", castPyCFunctionWithKeywords(THPVariable_to_sparse_bsr), METH_VARARGS | METH_KEYWORDS, nullptr},
17828:   {"to_sparse_csc", castPyCFunctionWithKeywords(THPVariable_to_sparse_csc), METH_VARARGS | METH_KEYWORDS, nullptr},
17829:   {"to_sparse_csr", castPyCFunctionWithKeywords(THPVariable_to_sparse_csr), METH_VARARGS | METH_KEYWORDS, nullptr},
17830:   {"topk", castPyCFunctionWithKeywords(THPVariable_topk), METH_VARARGS | METH_KEYWORDS, nullptr},
17831:   {"trace", THPVariable_trace, METH_NOARGS, nullptr},
17832:   {"transpose", castPyCFunctionWithKeywords(THPVariable_transpose), METH_VARARGS | METH_KEYWORDS, nullptr},
17833:   {"transpose_", castPyCFunctionWithKeywords(THPVariable_transpose_), METH_VARARGS | METH_KEYWORDS, nullptr},
17834:   {"triangular_solve", castPyCFunctionWithKeywords(THPVariable_triangular_solve), METH_VARARGS | METH_KEYWORDS, nullptr},
17835:   {"tril", castPyCFunctionWithKeywords(THPVariable_tril), METH_VARARGS | METH_KEYWORDS, nullptr},
17836:   {"tril_", castPyCFunctionWithKeywords(THPVariable_tril_), METH_VARARGS | METH_KEYWORDS, nullptr},
17837:   {"triu", castPyCFunctionWithKeywords(THPVariable_triu), METH_VARARGS | METH_KEYWORDS, nullptr},
17838:   {"triu_", castPyCFunctionWithKeywords(THPVariable_triu_), METH_VARARGS | METH_KEYWORDS, nullptr},
17839:   {"true_divide", castPyCFunctionWithKeywords(THPVariable_true_divide), METH_VARARGS | METH_KEYWORDS, nullptr},
17840:   {"true_divide_", castPyCFunctionWithKeywords(THPVariable_true_divide_), METH_VARARGS | METH_KEYWORDS, nullptr},
17841:   {"trunc", THPVariable_trunc, METH_NOARGS, nullptr},
17842:   {"trunc_", THPVariable_trunc_, METH_NOARGS, nullptr},
17843:   {"type_as", castPyCFunctionWithKeywords(THPVariable_type_as), METH_VARARGS | METH_KEYWORDS, nullptr},
17844:   {"unbind", castPyCFunctionWithKeywords(THPVariable_unbind), METH_VARARGS | METH_KEYWORDS, nullptr},
17845:   {"unflatten", castPyCFunctionWithKeywords(THPVariable_unflatten), METH_VARARGS | METH_KEYWORDS, nullptr},
17846:   {"unfold", castPyCFunctionWithKeywords(THPVariable_unfold), METH_VARARGS | METH_KEYWORDS, nullptr},
17847:   {"uniform_", castPyCFunctionWithKeywords(THPVariable_uniform_), METH_VARARGS | METH_KEYWORDS, nullptr},
17848:   {"unsafe_chunk", castPyCFunctionWithKeywords(THPVariable_unsafe_chunk), METH_VARARGS | METH_KEYWORDS, nullptr},
17849:   {"unsafe_split", castPyCFunctionWithKeywords(THPVariable_unsafe_split), METH_VARARGS | METH_KEYWORDS, nullptr},
17850:   {"unsafe_split_with_sizes", castPyCFunctionWithKeywords(THPVariable_unsafe_split_with_sizes), METH_VARARGS | METH_KEYWORDS, nullptr},
17851:   {"unsqueeze", castPyCFunctionWithKeywords(THPVariable_unsqueeze), METH_VARARGS | METH_KEYWORDS, nullptr},
17852:   {"unsqueeze_", castPyCFunctionWithKeywords(THPVariable_unsqueeze_), METH_VARARGS | METH_KEYWORDS, nullptr},
17853:   {"values", THPVariable_values, METH_NOARGS, nullptr},
17854:   {"var", castPyCFunctionWithKeywords(THPVariable_var), METH_VARARGS | METH_KEYWORDS, nullptr},
17855:   {"vdot", castPyCFunctionWithKeywords(THPVariable_vdot), METH_VARARGS | METH_KEYWORDS, nullptr},
17856:   {"view", castPyCFunctionWithKeywords(THPVariable_view), METH_VARARGS | METH_KEYWORDS, nullptr},
17857:   {"view_as", castPyCFunctionWithKeywords(THPVariable_view_as), METH_VARARGS | METH_KEYWORDS, nullptr},
17858:   {"vsplit", castPyCFunctionWithKeywords(THPVariable_vsplit), METH_VARARGS | METH_KEYWORDS, nullptr},
17859:   {"where", castPyCFunctionWithKeywords(THPVariable_where), METH_VARARGS | METH_KEYWORDS, nullptr},
17860:   {"xlogy", castPyCFunctionWithKeywords(THPVariable_xlogy), METH_VARARGS | METH_KEYWORDS, nullptr},
17861:   {"xlogy_", castPyCFunctionWithKeywords(THPVariable_xlogy_), METH_VARARGS | METH_KEYWORDS, nullptr},
17862:   {"zero_", THPVariable_zero_, METH_NOARGS, nullptr},
17863:   {nullptr}
17864: };
17865: 
17866: } // namespace torch::autograd
```

- EN: They also open, refine, or close the relevant C++ namespace scope. The main execution path in this span is carried by `castPyCFunctionWithKeywords`. This span sits on the Python/C++ boundary and converts or validates Python-visible arguments/results.
- CN: 它们还会打开、细化或关闭相应的 C++ 命名空间作用域。 这一段的主要执行路径由 `castPyCFunctionWithKeywords` 等函数/方法承载。 这一段位于 Python/C++ 边界上，会转换或校验对 Python 可见的参数与结果。

## Key Concepts / 关键概念
- Autograd graph and gradient metadata / 自动求导图与梯度元数据
- Generated code patterns / 生成代码模式
- Dispatch-key routing and redispatch / DispatchKey 路由与再次分发
- Gradient bookkeeping and autograd history / 梯度记录与自动求导历史
- Python/C++ binding boundary / Python/C++ 绑定边界
- Tensor/value representation management / 张量/数值表示管理

## Dependencies / 依赖关系
- Direct includes / 直接包含: `Python.h`, `torch/csrc/DynamicTypes.h`, `torch/csrc/Exceptions.h`, `torch/csrc/Size.h`, `torch/csrc/autograd/generated/VariableType.h`, `torch/csrc/autograd/python_variable.h`, `torch/csrc/autograd/utils/python_arg_parsing.h`, `torch/csrc/autograd/utils/error_messages.h`, `torch/csrc/autograd/utils/wrap_outputs.h`, `torch/csrc/jit/frontend/tracer.h`
- Include roots / 头文件根模块: `ATen`, `c10`, `torch`
- Key symbols / 关键符号: `THPVariable__is_view`, `handle_torch_function`, `THPVariable_apply_`, `THPVariable_Wrap`, `THPVariable_size`, `THPSize_NewFromSymSizes`, `wrap`, `THPVariable_stride`, `tuple`, `THPVariable_get_device`
- Related subsystems / 相关子系统: Autograd / 自动求导, ATen operator layer / ATen 算子层, c10 core utilities / c10 核心工具, Python binding layer / Python 绑定层

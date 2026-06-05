# factory.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/fastrnns/factory.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: from collections import namedtuple
 2: 
 3: import torch
 4: from torch import Tensor
 5: 
 6: from .cells import flat_lstm_cell, lstm_cell, premul_lstm_cell, premul_lstm_cell_no_bias
 7: 
 8: 
 9: # list[list[T]] -> list[T]
10: def flatten_list(lst):
11:     result = []
12:     for inner in lst:
13:         result.extend(inner)
14:     return result
15: 
16: 
17: """
18: Define a creator as a function:
19: (options) -> (inputs, params, forward, backward_setup, backward)
20: inputs: the inputs to the returned 'forward'. One can call
21:     forward(*inputs) directly.
22: params: List[Tensor] all requires_grad=True parameters.
23: forward: function / graph executor / module
24:     One can call rnn(rnn_inputs) using the outputs of the creator.
25: backward_setup: backward_inputs = backward_setup(*outputs)
26:     Then, we pass backward_inputs to backward. If None, then it is assumed to
27:     be the identity function.
28: backward: Given `output = backward_setup(*forward(*inputs))`, performs
29:     backpropagation. If None, then nothing happens.
30: 
31: fastrnns.bench times the forward and backward invocations.
32: """
33: 
34: 
35: ModelDef = namedtuple(
36:     "ModelDef", ["inputs", "params", "forward", "backward_setup", "backward"]
37: )
38: 
39: 
40: def lstm_backward_setup(lstm_outputs, seed=None):
41:     hx, _ = lstm_outputs
42:     return simple_backward_setup(hx, seed)
43: 
44: 
45: def simple_backward_setup(output, seed=None):
46:     if not isinstance(output, torch.Tensor):
47:         raise AssertionError(f"Expected output to be a Tensor, but got {type(output)}")
48:     if seed:
49:         torch.manual_seed(seed)
50:     grad_output = torch.randn_like(output)
51:     return output, grad_output
52: 
53: 
54: def simple_backward(output, grad_output, **kwargs):
55:     return output.backward(grad_output, **kwargs)
56: 
57: 
58: def pytorch_lstm_creator(**kwargs):
59:     input, hidden, _, module = lstm_inputs(return_module=True, **kwargs)
60:     return ModelDef(
````
- EN: Handles module imports such as `collections`, `torch`, `.cells`.
- CN: 处理模块导入，例如 `collections`, `torch`, `.cells`。
- EN: Implements callable logic such as `flatten_list`, `lstm_backward_setup`, `simple_backward_setup`, `simple_backward`.
- CN: 实现可调用逻辑，例如 `flatten_list`, `lstm_backward_setup`, `simple_backward_setup`, `simple_backward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````python
 61:         inputs=[input, hidden],
 62:         params=flatten_list(module.all_weights),
 63:         forward=module,
 64:         backward_setup=lstm_backward_setup,
 65:         backward=simple_backward,
 66:     )
 67: 
 68: 
 69: def lstm_creator(script=True, **kwargs):
 70:     input, hidden, params, _ = lstm_inputs(return_module=False, **kwargs)
 71:     inputs = [input, hidden] + params[0]
 72:     return ModelDef(
 73:         inputs=inputs,
 74:         params=flatten_list(params),
 75:         forward=lstm_factory(lstm_cell, script),
 76:         backward_setup=lstm_backward_setup,
 77:         backward=simple_backward,
 78:     )
 79: 
 80: 
 81: def lnlstm_creator(script=True, decompose_layernorm=False, **kwargs):
 82:     if script is not True:
 83:         raise AssertionError("lnlstm_creator requires script=True")
 84:     from .custom_lstms import script_lnlstm
 85: 
 86:     input_size = kwargs["inputSize"]
 87:     hidden_size = kwargs["hiddenSize"]
 88:     seq_len = kwargs["seqLength"]
 89:     batch_size = kwargs["miniBatch"]
 90:     ge = script_lnlstm(
 91:         input_size, hidden_size, 1, decompose_layernorm=decompose_layernorm
 92:     ).cuda()
 93: 
 94:     input = torch.randn(seq_len, batch_size, input_size, device="cuda")
 95:     states = [
 96:         (
 97:             torch.randn(batch_size, hidden_size, device="cuda"),
 98:             torch.randn(batch_size, hidden_size, device="cuda"),
 99:         )
100:     ]
101: 
102:     return ModelDef(
103:         inputs=[input, states],
104:         params=ge.parameters(),
105:         forward=ge,
106:         backward_setup=lstm_backward_setup,
107:         backward=simple_backward,
108:     )
109: 
110: 
111: def dropoutlstm_creator(script=True, **kwargs):
112:     if script is not True:
113:         raise AssertionError("dropoutlstm_creator requires script=True")
114:     from .custom_lstms import LSTMState, script_lstm
115: 
116:     input_size = kwargs["inputSize"]
117:     hidden_size = kwargs["hiddenSize"]
118:     seq_len = kwargs["seqLength"]
119:     batch_size = kwargs["miniBatch"]
120:     num_layers = kwargs["numLayers"]
````
- EN: Handles module imports such as `.custom_lstms`.
- CN: 处理模块导入，例如 `.custom_lstms`。
- EN: Implements callable logic such as `lstm_creator`, `lnlstm_creator`, `dropoutlstm_creator`.
- CN: 实现可调用逻辑，例如 `lstm_creator`, `lnlstm_creator`, `dropoutlstm_creator`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:     ge = script_lstm(input_size, hidden_size, num_layers, dropout=True).cuda()
122: 
123:     input = torch.randn(seq_len, batch_size, input_size, device="cuda")
124:     states = [
125:         LSTMState(
126:             torch.randn(batch_size, hidden_size, device="cuda"),
127:             torch.randn(batch_size, hidden_size, device="cuda"),
128:         )
129:         for _ in range(num_layers)
130:     ]
131:     return ModelDef(
132:         inputs=[input, states],
133:         params=ge.parameters(),
134:         forward=ge,
135:         backward_setup=lstm_backward_setup,
136:         backward=simple_backward,
137:     )
138: 
139: 
140: def lstm_premul_creator(script=True, **kwargs):
141:     input, hidden, params, _ = lstm_inputs(return_module=False, **kwargs)
142:     inputs = [input, hidden] + params[0]
143:     return ModelDef(
144:         inputs=inputs,
145:         params=flatten_list(params),
146:         forward=lstm_factory_premul(premul_lstm_cell, script),
147:         backward_setup=lstm_backward_setup,
148:         backward=simple_backward,
149:     )
150: 
151: 
152: def lstm_premul_bias_creator(script=True, **kwargs):
153:     input, hidden, params, _ = lstm_inputs(return_module=False, **kwargs)
154:     inputs = [input, hidden] + params[0]
155:     return ModelDef(
156:         inputs=inputs,
157:         params=flatten_list(params),
158:         forward=lstm_factory_premul_bias(premul_lstm_cell_no_bias, script),
159:         backward_setup=lstm_backward_setup,
160:         backward=simple_backward,
161:     )
162: 
163: 
164: def lstm_simple_creator(script=True, **kwargs):
165:     input, hidden, params, _ = lstm_inputs(return_module=False, **kwargs)
166:     inputs = [input] + [h[0] for h in hidden] + params[0]
167:     return ModelDef(
168:         inputs=inputs,
169:         params=flatten_list(params),
170:         forward=lstm_factory_simple(flat_lstm_cell, script),
171:         backward_setup=lstm_backward_setup,
172:         backward=simple_backward,
173:     )
174: 
175: 
176: def lstm_multilayer_creator(script=True, **kwargs):
177:     input, hidden, params, _ = lstm_inputs(return_module=False, **kwargs)
178:     inputs = [input, hidden, flatten_list(params)]
179:     return ModelDef(
180:         inputs=inputs,
````
- EN: Implements callable logic such as `lstm_premul_creator`, `lstm_premul_bias_creator`, `lstm_simple_creator`, `lstm_multilayer_creator`.
- CN: 实现可调用逻辑，例如 `lstm_premul_creator`, `lstm_premul_bias_creator`, `lstm_simple_creator`, `lstm_multilayer_creator`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         params=flatten_list(params),
182:         forward=lstm_factory_multilayer(lstm_cell, script),
183:         backward_setup=lstm_backward_setup,
184:         backward=simple_backward,
185:     )
186: 
187: 
188: def imagenet_cnn_creator(arch, jit=True):
189:     def creator(device="cuda", **kwargs):
190:         model = arch().to(device)
191:         x = torch.randn(32, 3, 224, 224, device=device)
192:         if jit:
193:             model = torch.jit.trace(model, x)
194:         return ModelDef(
195:             inputs=(x,),
196:             params=list(model.parameters()),
197:             forward=model,
198:             backward_setup=simple_backward_setup,
199:             backward=simple_backward,
200:         )
201: 
202:     return creator
203: 
204: 
205: def varlen_lstm_inputs(
206:     minlen=30,
207:     maxlen=100,
208:     numLayers=1,
209:     inputSize=512,
210:     hiddenSize=512,
211:     miniBatch=64,
212:     return_module=False,
213:     device="cuda",
214:     seed=None,
215:     **kwargs,
216: ):
217:     if seed is not None:
218:         torch.manual_seed(seed)
219:     lengths = torch.randint(
220:         low=minlen, high=maxlen, size=[miniBatch], dtype=torch.long, device=device
221:     )
222:     x = [torch.randn(length, inputSize, device=device) for length in lengths]
223:     hx = torch.randn(numLayers, miniBatch, hiddenSize, device=device)
224:     cx = torch.randn(numLayers, miniBatch, hiddenSize, device=device)
225:     lstm = torch.nn.LSTM(inputSize, hiddenSize, numLayers).to(device)
226: 
227:     if return_module:
228:         return x, lengths, (hx, cx), lstm.all_weights, lstm
229:     else:
230:         # NB: lstm.all_weights format:
231:         # w_ih, w_hh, b_ih, b_hh = lstm.all_weights[layer]
232:         return x, lengths, (hx, cx), lstm.all_weights, None
233: 
234: 
235: def varlen_lstm_backward_setup(forward_output, seed=None):
236:     if seed:
237:         torch.manual_seed(seed)
238:     rnn_utils = torch.nn.utils.rnn
239:     sequences = forward_output[0]
240:     padded = rnn_utils.pad_sequence(sequences)
````
- EN: Implements callable logic such as `imagenet_cnn_creator`, `creator`, `varlen_lstm_inputs`, `varlen_lstm_backward_setup`.
- CN: 实现可调用逻辑，例如 `imagenet_cnn_creator`, `creator`, `varlen_lstm_inputs`, `varlen_lstm_backward_setup`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:     grad = torch.randn_like(padded)
242:     return padded, grad
243: 
244: 
245: def varlen_pytorch_lstm_creator(**kwargs):
246:     rnn_utils = torch.nn.utils.rnn
247:     sequences, _, hidden, _, module = varlen_lstm_inputs(return_module=True, **kwargs)
248: 
249:     def forward(sequences, hidden):
250:         packed = rnn_utils.pack_sequence(sequences, enforce_sorted=False)
251:         out, new_hidden = module(packed, hidden)
252:         padded, lengths = rnn_utils.pad_packed_sequence(out)
253:         # XXX: It's more efficient to store the output in its padded form,
254:         # but that might not be conducive to loss computation.
255:         # Un-padding the output also makes the backward pass 2x slower...
256:         # return [padded[:lengths[i], i, :] for i in range(lengths.size(0))]
257:         return padded, new_hidden
258: 
259:     return ModelDef(
260:         inputs=[sequences, hidden],
261:         params=flatten_list(module.all_weights),
262:         forward=forward,
263:         backward_setup=lstm_backward_setup,
264:         backward=simple_backward,
265:     )
266: 
267: 
268: def varlen_lstm_factory(cell, script):
269:     def dynamic_rnn(
270:         sequences: list[Tensor],
271:         hiddens: tuple[Tensor, Tensor],
272:         w_ih: Tensor,
273:         w_hh: Tensor,
274:         b_ih: Tensor,
275:         b_hh: Tensor,
276:     ) -> tuple[list[Tensor], tuple[list[Tensor], list[Tensor]]]:
277:         hx, cx = hiddens
278:         hxs = hx.unbind(1)
279:         cxs = cx.unbind(1)
280:         # List of: (output, hx, cx)
281:         outputs = []
282:         hx_outs = []
283:         cx_outs = []
284: 
285:         for batch in range(len(sequences)):
286:             output = []
287:             hy, cy = hxs[batch], cxs[batch]
288:             inputs = sequences[batch].unbind(0)
289: 
290:             for seq_idx in range(len(inputs)):
291:                 hy, cy = cell(
292:                     inputs[seq_idx].unsqueeze(0), (hy, cy), w_ih, w_hh, b_ih, b_hh
293:                 )
294:                 output += [hy]
295:             outputs += [torch.stack(output)]
296:             hx_outs += [hy.unsqueeze(0)]
297:             cx_outs += [cy.unsqueeze(0)]
298: 
299:         return outputs, (hx_outs, cx_outs)
300: 
````
- EN: Implements callable logic such as `varlen_pytorch_lstm_creator`, `forward`, `varlen_lstm_factory`, `dynamic_rnn`.
- CN: 实现可调用逻辑，例如 `varlen_pytorch_lstm_creator`, `forward`, `varlen_lstm_factory`, `dynamic_rnn`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:     if script:
302:         cell = torch.jit.script(cell)
303:         dynamic_rnn = torch.jit.script(dynamic_rnn)
304: 
305:     return dynamic_rnn
306: 
307: 
308: def varlen_lstm_creator(script=False, **kwargs):
309:     sequences, _, hidden, params, _ = varlen_lstm_inputs(return_module=False, **kwargs)
310:     inputs = [sequences, hidden] + params[0]
311:     return ModelDef(
312:         inputs=inputs,
313:         params=flatten_list(params),
314:         forward=varlen_lstm_factory(lstm_cell, script),
315:         backward_setup=varlen_lstm_backward_setup,
316:         backward=simple_backward,
317:     )
318: 
319: 
320: # cudnn_layernorm_lstm: since cudnn does not have Layernorm LSTM, we cannot benchmark
321: # the lowerbound directly. Instead, we only benchmark the forward pass by mimicking the
322: # computation of a cudnn lstm + seq_len * 3 layernorm computation. This should serve
323: # as a perf lowerbound for the Layernorm LSTM forward pass(given that Layernorm itself
324: # is invariant), the lowerbound of backward pass is hard to get since we lose the
325: # intermediate results, we can still optimize the layernorm implementation to make
326: # a faster forward lowerbound though.
327: def layernorm_pytorch_lstm_creator(**kwargs):
328:     input, hidden, _, module = lstm_inputs(return_module=True, **kwargs)
329:     batch_size = kwargs["miniBatch"]
330:     hidden_size = kwargs["hiddenSize"]
331:     ln_i = torch.nn.LayerNorm(4 * hidden_size).cuda()
332:     ln_h = torch.nn.LayerNorm(4 * hidden_size).cuda()
333:     ln_c = torch.nn.LayerNorm(hidden_size).cuda()
334:     ln_input1 = torch.randn(batch_size, 4 * hidden_size, device="cuda")
335: 
336:     def forward(input, hidden):
337:         out, new_hidden = module(input, hidden)
338:         # plus (seq_len * three laynorm cell computation) to mimic the lower bound of
339:         # Layernorm cudnn LSTM in the forward pass
340:         seq_len = len(input.unbind(0))
341:         hy, cy = new_hidden
342:         for i in range(seq_len):
343:             ln_i(ln_input1)
344:             ln_h(ln_input1)
345:             cy = ln_c(cy)
346: 
347:         return out, (hy, cy)
348: 
349:     return ModelDef(
350:         inputs=[input, hidden],
351:         params=flatten_list(module.all_weights),
352:         forward=forward,
353:         backward_setup=lstm_backward_setup,
354:         backward=None,
355:     )
356: 
357: 
358: # input: lstm.all_weights format (w_ih, w_hh, b_ih, b_hh = lstm.all_weights[layer])
359: # output: packed_weights with format
360: # packed_weights[0] is w_ih with size (layer, 4*hiddenSize, inputSize)
````
- EN: Implements callable logic such as `varlen_lstm_creator`, `layernorm_pytorch_lstm_creator`, `forward`.
- CN: 实现可调用逻辑，例如 `varlen_lstm_creator`, `layernorm_pytorch_lstm_creator`, `forward`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361: # packed_weights[1] is w_hh with size (layer, 4*hiddenSize, hiddenSize)
362: # packed_weights[2] is b_ih with size (layer, 4*hiddenSize)
363: # packed_weights[3] is b_hh with size (layer, 4*hiddenSize)
364: def stack_weights(weights):
365:     def unzip_columns(mat):
366:         if not isinstance(mat, list):
367:             raise AssertionError(f"Expected mat to be a list, but got {type(mat)}")
368:         if not isinstance(mat[0], list):
369:             raise AssertionError(
370:                 f"Expected mat[0] to be a list, but got {type(mat[0])}"
371:             )
372:         layers = len(mat)
373:         columns = len(mat[0])
374:         return [[mat[layer][col] for layer in range(layers)] for col in range(columns)]
375: 
376:     # XXX: script fns have problems indexing multidim lists, so we try to
377:     # avoid them by stacking tensors
378:     all_weights = weights
379:     packed_weights = [torch.stack(param) for param in unzip_columns(all_weights)]
380:     return packed_weights
381: 
382: 
383: # returns: x, (hx, cx), all_weights, lstm module with all_weights as params
384: def lstm_inputs(
385:     seqLength=100,
386:     numLayers=1,
387:     inputSize=512,
388:     hiddenSize=512,
389:     miniBatch=64,
390:     dropout=0.0,
391:     return_module=False,
392:     device="cuda",
393:     seed=None,
394: ):
395:     if seed is not None:
396:         torch.manual_seed(seed)
397:     x = torch.randn(seqLength, miniBatch, inputSize, device=device)
398:     hx = torch.randn(numLayers, miniBatch, hiddenSize, device=device)
399:     cx = torch.randn(numLayers, miniBatch, hiddenSize, device=device)
400:     lstm = torch.nn.LSTM(inputSize, hiddenSize, numLayers, dropout=dropout)
401:     if "cuda" in device:
402:         lstm = lstm.cuda()
403: 
404:     if return_module:
405:         return x, (hx, cx), lstm.all_weights, lstm
406:     else:
407:         # NB: lstm.all_weights format:
408:         # w_ih, w_hh, b_ih, b_hh = lstm.all_weights[layer]
409:         return x, (hx, cx), lstm.all_weights, None
410: 
411: 
412: def lstm_factory(cell, script):
413:     def dynamic_rnn(
414:         input: Tensor,
415:         hidden: tuple[Tensor, Tensor],
416:         w_ih: Tensor,
417:         w_hh: Tensor,
418:         b_ih: Tensor,
419:         b_hh: Tensor,
420:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
````
- EN: Implements callable logic such as `stack_weights`, `unzip_columns`, `lstm_inputs`, `lstm_factory`.
- CN: 实现可调用逻辑，例如 `stack_weights`, `unzip_columns`, `lstm_inputs`, `lstm_factory`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````python
421:         hx, cx = hidden
422:         outputs = []
423:         inputs = input.unbind(0)
424:         hy, cy = hx[0], cx[0]
425:         for seq_idx in range(len(inputs)):
426:             hy, cy = cell(inputs[seq_idx], (hy, cy), w_ih, w_hh, b_ih, b_hh)
427:             outputs += [hy]
428:         return torch.stack(outputs), (hy.unsqueeze(0), cy.unsqueeze(0))
429: 
430:     if script:
431:         cell = torch.jit.script(cell)
432:         dynamic_rnn = torch.jit.script(dynamic_rnn)
433: 
434:     return dynamic_rnn
435: 
436: 
437: # premul: we're going to premultiply the inputs & weights
438: def lstm_factory_premul(premul_cell, script):
439:     def dynamic_rnn(
440:         input: Tensor,
441:         hidden: tuple[Tensor, Tensor],
442:         w_ih: Tensor,
443:         w_hh: Tensor,
444:         b_ih: Tensor,
445:         b_hh: Tensor,
446:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
447:         hx, cx = hidden
448:         outputs = []
449:         inputs = torch.matmul(input, w_ih.t()).unbind(0)
450:         hy, cy = hx[0], cx[0]
451:         for seq_idx in range(len(inputs)):
452:             hy, cy = premul_cell(inputs[seq_idx], (hy, cy), w_hh, b_ih, b_hh)
453:             outputs += [hy]
454:         return torch.stack(outputs), (hy.unsqueeze(0), cy.unsqueeze(0))
455: 
456:     if script:
457:         premul_cell = torch.jit.script(premul_cell)
458:         dynamic_rnn = torch.jit.script(dynamic_rnn)
459: 
460:     return dynamic_rnn
461: 
462: 
463: # premul: we're going to premultiply the inputs & weights, and add bias
464: def lstm_factory_premul_bias(premul_cell, script):
465:     def dynamic_rnn(
466:         input: Tensor,
467:         hidden: tuple[Tensor, Tensor],
468:         w_ih: Tensor,
469:         w_hh: Tensor,
470:         b_ih: Tensor,
471:         b_hh: Tensor,
472:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
473:         hx, cx = hidden
474:         outputs = []
475:         inpSize = input.size()
476:         # add bias for all timesteps instead of going step-by-step, results in a single reduction kernel in the backward
477:         # FIXME matmul(x,y) + bias currently goes through jit AD, and backward formula in AD is not optimized for this
478:         # case. Workaround with mm and views.
479:         inpSize = input.size()
480:         inputs = torch.mm(input.view(-1, inpSize[2]), w_ih.t()) + b_ih
````
- EN: Implements callable logic such as `lstm_factory_premul`, `dynamic_rnn`, `lstm_factory_premul_bias`.
- CN: 实现可调用逻辑，例如 `lstm_factory_premul`, `dynamic_rnn`, `lstm_factory_premul_bias`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-539
````python
481:         inputs = inputs.view(inpSize[0], inpSize[1], -1).unbind(0)
482:         hy, cy = hx[0], cx[0]
483:         for seq_idx in range(len(inputs)):
484:             hy, cy = premul_cell(inputs[seq_idx], (hy, cy), w_hh, b_hh)
485:             outputs += [hy]
486:         return torch.stack(outputs), (hy.unsqueeze(0), cy.unsqueeze(0))
487: 
488:     if script:
489:         premul_cell = torch.jit.script(premul_cell)
490:         dynamic_rnn = torch.jit.script(dynamic_rnn)
491: 
492:     return dynamic_rnn
493: 
494: 
495: # simple: flat inputs (no tuples), no list to accumulate outputs
496: #         useful mostly for benchmarking older JIT versions
497: def lstm_factory_simple(cell, script):
498:     def dynamic_rnn(input, hx, cx, w_ih, w_hh, b_ih, b_hh):
499:         hy = hx  # for scoping
500:         cy = cx  # for scoping
501:         inputs = input.unbind(0)
502:         for seq_idx in range(len(inputs)):
503:             hy, cy = cell(inputs[seq_idx], hy, cy, w_ih, w_hh, b_ih, b_hh)
504:         return hy, cy
505: 
506:     if script:
507:         cell = torch.jit.script(cell)
508:         dynamic_rnn = torch.jit.script(dynamic_rnn)
509: 
510:     return dynamic_rnn
511: 
512: 
513: def lstm_factory_multilayer(cell, script):
514:     def dynamic_rnn(
515:         input: Tensor, hidden: tuple[Tensor, Tensor], params: list[Tensor]
516:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
517:         params_stride = 4  # NB: this assumes that biases are there
518:         hx, cx = hidden
519:         hy, cy = hidden  # for scoping...
520:         inputs, outputs = input.unbind(0), []
521:         for layer in range(hx.size(0)):
522:             hy = hx[layer]
523:             cy = cx[layer]
524:             base_idx = layer * params_stride
525:             w_ih = params[base_idx]
526:             w_hh = params[base_idx + 1]
527:             b_ih = params[base_idx + 2]
528:             b_hh = params[base_idx + 3]
529:             for seq_idx in range(len(inputs)):
530:                 hy, cy = cell(inputs[seq_idx], (hy, cy), w_ih, w_hh, b_ih, b_hh)
531:                 outputs += [hy]
532:             inputs, outputs = outputs, []
533:         return torch.stack(inputs), (hy.unsqueeze(0), cy.unsqueeze(0))
534: 
535:     if script:
536:         cell = torch.jit.script(cell)
537:         dynamic_rnn = torch.jit.script(dynamic_rnn)
538: 
539:     return dynamic_rnn
````
- EN: Implements callable logic such as `lstm_factory_simple`, `dynamic_rnn`, `lstm_factory_multilayer`.
- CN: 实现可调用逻辑，例如 `lstm_factory_simple`, `dynamic_rnn`, `lstm_factory_multilayer`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `flatten_list` / 符号 `flatten_list`
- Symbol `lstm_backward_setup` / 符号 `lstm_backward_setup`
- Symbol `simple_backward_setup` / 符号 `simple_backward_setup`
- Symbol `simple_backward` / 符号 `simple_backward`

## Dependencies / 依赖关系
- Python imports: `collections`, `torch`, `.cells`, `.custom_lstms`
- Python 导入: `collections`, `torch`, `.cells`, `.custom_lstms`

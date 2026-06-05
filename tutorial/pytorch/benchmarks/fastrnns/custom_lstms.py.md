# custom_lstms.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/fastrnns/custom_lstms.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: import numbers
 2: import warnings
 3: from collections import namedtuple
 4: 
 5: import torch
 6: import torch.jit as jit
 7: import torch.nn as nn
 8: from torch import Tensor
 9: from torch.nn import Parameter
10: 
11: 
12: """
13: Some helper classes for writing custom TorchScript LSTMs.
14: 
15: Goals:
16: - Classes are easy to read, use, and extend
17: - Performance of custom LSTMs approach fused-kernel-levels of speed.
18: 
19: A few notes about features we could add to clean up the below code:
20: - Support enumerate with nn.ModuleList:
21:   https://github.com/pytorch/pytorch/issues/14471
22: - Support enumerate/zip with lists:
23:   https://github.com/pytorch/pytorch/issues/15952
24: - Support overriding of class methods:
25:   https://github.com/pytorch/pytorch/issues/10733
26: - Support passing around user-defined namedtuple types for readability
27: - Support slicing w/ range. It enables reversing lists easily.
28:   https://github.com/pytorch/pytorch/issues/10774
29: - Multiline type annotations. List[List[Tuple[Tensor,Tensor]]] is verbose
30:   https://github.com/pytorch/pytorch/pull/14922
31: """
32: 
33: 
34: def script_lstm(
35:     input_size,
36:     hidden_size,
37:     num_layers,
38:     bias=True,
39:     batch_first=False,
40:     dropout=False,
41:     bidirectional=False,
42: ):
43:     """Returns a ScriptModule that mimics a PyTorch native LSTM."""
44: 
45:     # The following are not implemented.
46:     if not bias:
47:         raise AssertionError("bias=False is not implemented")
48:     if batch_first:
49:         raise AssertionError("batch_first=True is not implemented")
50: 
51:     if bidirectional:
52:         stack_type = StackedLSTM2
53:         layer_type = BidirLSTMLayer
54:         dirs = 2
55:     elif dropout:
56:         stack_type = StackedLSTMWithDropout
57:         layer_type = LSTMLayer
58:         dirs = 1
59:     else:
60:         stack_type = StackedLSTM
````
- EN: Handles module imports such as `numbers`, `warnings`, `collections`, `torch`.
- CN: 处理模块导入，例如 `numbers`, `warnings`, `collections`, `torch`。
- EN: Declares or extends types including `methods`.
- CN: 声明或扩展类型，包括 `methods`。
- EN: Implements callable logic such as `script_lstm`.
- CN: 实现可调用逻辑，例如 `script_lstm`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:         layer_type = LSTMLayer
 62:         dirs = 1
 63: 
 64:     return stack_type(
 65:         num_layers,
 66:         layer_type,
 67:         first_layer_args=[LSTMCell, input_size, hidden_size],
 68:         other_layer_args=[LSTMCell, hidden_size * dirs, hidden_size],
 69:     )
 70: 
 71: 
 72: def script_lnlstm(
 73:     input_size,
 74:     hidden_size,
 75:     num_layers,
 76:     bias=True,
 77:     batch_first=False,
 78:     dropout=False,
 79:     bidirectional=False,
 80:     decompose_layernorm=False,
 81: ):
 82:     """Returns a ScriptModule that mimics a PyTorch native LSTM."""
 83: 
 84:     # The following are not implemented.
 85:     if not bias:
 86:         raise AssertionError("bias=False is not implemented")
 87:     if batch_first:
 88:         raise AssertionError("batch_first=True is not implemented")
 89:     if dropout:
 90:         raise AssertionError("dropout=True is not implemented")
 91: 
 92:     if bidirectional:
 93:         stack_type = StackedLSTM2
 94:         layer_type = BidirLSTMLayer
 95:         dirs = 2
 96:     else:
 97:         stack_type = StackedLSTM
 98:         layer_type = LSTMLayer
 99:         dirs = 1
100: 
101:     return stack_type(
102:         num_layers,
103:         layer_type,
104:         first_layer_args=[
105:             LayerNormLSTMCell,
106:             input_size,
107:             hidden_size,
108:             decompose_layernorm,
109:         ],
110:         other_layer_args=[
111:             LayerNormLSTMCell,
112:             hidden_size * dirs,
113:             hidden_size,
114:             decompose_layernorm,
115:         ],
116:     )
117: 
118: 
119: LSTMState = namedtuple("LSTMState", ["hx", "cx"])
120: 
````
- EN: Implements callable logic such as `script_lnlstm`.
- CN: 实现可调用逻辑，例如 `script_lnlstm`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121: 
122: def reverse(lst: list[Tensor]) -> list[Tensor]:
123:     return lst[::-1]
124: 
125: 
126: class LSTMCell(jit.ScriptModule):
127:     def __init__(self, input_size, hidden_size):
128:         super().__init__()
129:         self.input_size = input_size
130:         self.hidden_size = hidden_size
131:         self.weight_ih = Parameter(torch.randn(4 * hidden_size, input_size))
132:         self.weight_hh = Parameter(torch.randn(4 * hidden_size, hidden_size))
133:         self.bias_ih = Parameter(torch.randn(4 * hidden_size))
134:         self.bias_hh = Parameter(torch.randn(4 * hidden_size))
135: 
136:     @jit.script_method
137:     def forward(
138:         self, input: Tensor, state: tuple[Tensor, Tensor]
139:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
140:         hx, cx = state
141:         gates = (
142:             torch.mm(input, self.weight_ih.t())
143:             + self.bias_ih
144:             + torch.mm(hx, self.weight_hh.t())
145:             + self.bias_hh
146:         )
147:         ingate, forgetgate, cellgate, outgate = gates.chunk(4, 1)
148: 
149:         ingate = torch.sigmoid(ingate)
150:         forgetgate = torch.sigmoid(forgetgate)
151:         cellgate = torch.tanh(cellgate)
152:         outgate = torch.sigmoid(outgate)
153: 
154:         cy = (forgetgate * cx) + (ingate * cellgate)
155:         hy = outgate * torch.tanh(cy)
156: 
157:         return hy, (hy, cy)
158: 
159: 
160: class LayerNorm(jit.ScriptModule):
161:     def __init__(self, normalized_shape):
162:         super().__init__()
163:         if isinstance(normalized_shape, numbers.Integral):
164:             normalized_shape = (normalized_shape,)
165:         normalized_shape = torch.Size(normalized_shape)
166: 
167:         # XXX: This is true for our LSTM / NLP use case and helps simplify code
168:         if len(normalized_shape) != 1:
169:             raise AssertionError(
170:                 f"Expected normalized_shape to have length 1, but got {len(normalized_shape)}"
171:             )
172: 
173:         self.weight = Parameter(torch.ones(normalized_shape))
174:         self.bias = Parameter(torch.zeros(normalized_shape))
175:         self.normalized_shape = normalized_shape
176: 
177:     @jit.script_method
178:     def compute_layernorm_stats(self, input):
179:         mu = input.mean(-1, keepdim=True)
180:         sigma = input.std(-1, keepdim=True, unbiased=False)
````
- EN: Declares or extends types including `LSTMCell`, `LayerNorm`.
- CN: 声明或扩展类型，包括 `LSTMCell`, `LayerNorm`。
- EN: Implements callable logic such as `reverse`, `__init__`, `forward`, `compute_layernorm_stats`.
- CN: 实现可调用逻辑，例如 `reverse`, `__init__`, `forward`, `compute_layernorm_stats`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         return mu, sigma
182: 
183:     @jit.script_method
184:     def forward(self, input):
185:         mu, sigma = self.compute_layernorm_stats(input)
186:         return (input - mu) / sigma * self.weight + self.bias
187: 
188: 
189: class LayerNormLSTMCell(jit.ScriptModule):
190:     def __init__(self, input_size, hidden_size, decompose_layernorm=False):
191:         super().__init__()
192:         self.input_size = input_size
193:         self.hidden_size = hidden_size
194:         self.weight_ih = Parameter(torch.randn(4 * hidden_size, input_size))
195:         self.weight_hh = Parameter(torch.randn(4 * hidden_size, hidden_size))
196:         # The layernorms provide learnable biases
197: 
198:         if decompose_layernorm:
199:             ln = LayerNorm
200:         else:
201:             ln = nn.LayerNorm
202: 
203:         self.layernorm_i = ln(4 * hidden_size)
204:         self.layernorm_h = ln(4 * hidden_size)
205:         self.layernorm_c = ln(hidden_size)
206: 
207:     @jit.script_method
208:     def forward(
209:         self, input: Tensor, state: tuple[Tensor, Tensor]
210:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
211:         hx, cx = state
212:         igates = self.layernorm_i(torch.mm(input, self.weight_ih.t()))
213:         hgates = self.layernorm_h(torch.mm(hx, self.weight_hh.t()))
214:         gates = igates + hgates
215:         ingate, forgetgate, cellgate, outgate = gates.chunk(4, 1)
216: 
217:         ingate = torch.sigmoid(ingate)
218:         forgetgate = torch.sigmoid(forgetgate)
219:         cellgate = torch.tanh(cellgate)
220:         outgate = torch.sigmoid(outgate)
221: 
222:         cy = self.layernorm_c((forgetgate * cx) + (ingate * cellgate))
223:         hy = outgate * torch.tanh(cy)
224: 
225:         return hy, (hy, cy)
226: 
227: 
228: class LSTMLayer(jit.ScriptModule):
229:     def __init__(self, cell, *cell_args):
230:         super().__init__()
231:         self.cell = cell(*cell_args)
232: 
233:     @jit.script_method
234:     def forward(
235:         self, input: Tensor, state: tuple[Tensor, Tensor]
236:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
237:         inputs = input.unbind(0)
238:         outputs = torch.jit.annotate(list[Tensor], [])
239:         for i in range(len(inputs)):
240:             out, state = self.cell(inputs[i], state)
````
- EN: Declares or extends types including `LayerNormLSTMCell`, `LSTMLayer`.
- CN: 声明或扩展类型，包括 `LayerNormLSTMCell`, `LSTMLayer`。
- EN: Implements callable logic such as `forward`, `__init__`.
- CN: 实现可调用逻辑，例如 `forward`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241:             outputs += [out]
242:         return torch.stack(outputs), state
243: 
244: 
245: class ReverseLSTMLayer(jit.ScriptModule):
246:     def __init__(self, cell, *cell_args):
247:         super().__init__()
248:         self.cell = cell(*cell_args)
249: 
250:     @jit.script_method
251:     def forward(
252:         self, input: Tensor, state: tuple[Tensor, Tensor]
253:     ) -> tuple[Tensor, tuple[Tensor, Tensor]]:
254:         inputs = reverse(input.unbind(0))
255:         outputs = jit.annotate(list[Tensor], [])
256:         for i in range(len(inputs)):
257:             out, state = self.cell(inputs[i], state)
258:             outputs += [out]
259:         return torch.stack(reverse(outputs)), state
260: 
261: 
262: class BidirLSTMLayer(jit.ScriptModule):
263:     __constants__ = ["directions"]
264: 
265:     def __init__(self, cell, *cell_args):
266:         super().__init__()
267:         self.directions = nn.ModuleList(
268:             [
269:                 LSTMLayer(cell, *cell_args),
270:                 ReverseLSTMLayer(cell, *cell_args),
271:             ]
272:         )
273: 
274:     @jit.script_method
275:     def forward(
276:         self, input: Tensor, states: list[tuple[Tensor, Tensor]]
277:     ) -> tuple[Tensor, list[tuple[Tensor, Tensor]]]:
278:         # List[LSTMState]: [forward LSTMState, backward LSTMState]
279:         outputs = jit.annotate(list[Tensor], [])
280:         output_states = jit.annotate(list[tuple[Tensor, Tensor]], [])
281:         # XXX: enumerate https://github.com/pytorch/pytorch/issues/14471
282:         i = 0
283:         for direction in self.directions:
284:             state = states[i]
285:             out, out_state = direction(input, state)
286:             outputs += [out]
287:             output_states += [out_state]
288:             i += 1  # noqa: SIM113
289:         return torch.cat(outputs, -1), output_states
290: 
291: 
292: def init_stacked_lstm(num_layers, layer, first_layer_args, other_layer_args):
293:     layers = [layer(*first_layer_args)] + [
294:         layer(*other_layer_args) for _ in range(num_layers - 1)
295:     ]
296:     return nn.ModuleList(layers)
297: 
298: 
299: class StackedLSTM(jit.ScriptModule):
300:     __constants__ = ["layers"]  # Necessary for iterating through self.layers
````
- EN: Declares or extends types including `ReverseLSTMLayer`, `BidirLSTMLayer`, `StackedLSTM`.
- CN: 声明或扩展类型，包括 `ReverseLSTMLayer`, `BidirLSTMLayer`, `StackedLSTM`。
- EN: Implements callable logic such as `__init__`, `forward`, `init_stacked_lstm`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`, `init_stacked_lstm`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301: 
302:     def __init__(self, num_layers, layer, first_layer_args, other_layer_args):
303:         super().__init__()
304:         self.layers = init_stacked_lstm(
305:             num_layers, layer, first_layer_args, other_layer_args
306:         )
307: 
308:     @jit.script_method
309:     def forward(
310:         self, input: Tensor, states: list[tuple[Tensor, Tensor]]
311:     ) -> tuple[Tensor, list[tuple[Tensor, Tensor]]]:
312:         # List[LSTMState]: One state per layer
313:         output_states = jit.annotate(list[tuple[Tensor, Tensor]], [])
314:         output = input
315:         # XXX: enumerate https://github.com/pytorch/pytorch/issues/14471
316:         i = 0
317:         for rnn_layer in self.layers:
318:             state = states[i]
319:             output, out_state = rnn_layer(output, state)
320:             output_states += [out_state]
321:             i += 1  # noqa: SIM113
322:         return output, output_states
323: 
324: 
325: # Differs from StackedLSTM in that its forward method takes
326: # List[List[Tuple[Tensor,Tensor]]]. It would be nice to subclass StackedLSTM
327: # except we don't support overriding script methods.
328: # https://github.com/pytorch/pytorch/issues/10733
329: class StackedLSTM2(jit.ScriptModule):
330:     __constants__ = ["layers"]  # Necessary for iterating through self.layers
331: 
332:     def __init__(self, num_layers, layer, first_layer_args, other_layer_args):
333:         super().__init__()
334:         self.layers = init_stacked_lstm(
335:             num_layers, layer, first_layer_args, other_layer_args
336:         )
337: 
338:     @jit.script_method
339:     def forward(
340:         self, input: Tensor, states: list[list[tuple[Tensor, Tensor]]]
341:     ) -> tuple[Tensor, list[list[tuple[Tensor, Tensor]]]]:
342:         # List[List[LSTMState]]: The outer list is for layers,
343:         #                        inner list is for directions.
344:         output_states = jit.annotate(list[list[tuple[Tensor, Tensor]]], [])
345:         output = input
346:         # XXX: enumerate https://github.com/pytorch/pytorch/issues/14471
347:         i = 0
348:         for rnn_layer in self.layers:
349:             state = states[i]
350:             output, out_state = rnn_layer(output, state)
351:             output_states += [out_state]
352:             i += 1  # noqa: SIM113
353:         return output, output_states
354: 
355: 
356: class StackedLSTMWithDropout(jit.ScriptModule):
357:     # Necessary for iterating through self.layers and dropout support
358:     __constants__ = ["layers", "num_layers"]
359: 
360:     def __init__(self, num_layers, layer, first_layer_args, other_layer_args):
````
- EN: Declares or extends types including `StackedLSTM2`, `StackedLSTMWithDropout`.
- CN: 声明或扩展类型，包括 `StackedLSTM2`, `StackedLSTMWithDropout`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361:         super().__init__()
362:         self.layers = init_stacked_lstm(
363:             num_layers, layer, first_layer_args, other_layer_args
364:         )
365:         # Introduces a Dropout layer on the outputs of each LSTM layer except
366:         # the last layer, with dropout probability = 0.4.
367:         self.num_layers = num_layers
368: 
369:         if num_layers == 1:
370:             warnings.warn(
371:                 "dropout lstm adds dropout layers after all but last "
372:                 "recurrent layer, it expects num_layers greater than "
373:                 "1, but got num_layers = 1"
374:             )
375: 
376:         self.dropout_layer = nn.Dropout(0.4)
377: 
378:     @jit.script_method
379:     def forward(
380:         self, input: Tensor, states: list[tuple[Tensor, Tensor]]
381:     ) -> tuple[Tensor, list[tuple[Tensor, Tensor]]]:
382:         # List[LSTMState]: One state per layer
383:         output_states = jit.annotate(list[tuple[Tensor, Tensor]], [])
384:         output = input
385:         # XXX: enumerate https://github.com/pytorch/pytorch/issues/14471
386:         i = 0
387:         for rnn_layer in self.layers:
388:             state = states[i]
389:             output, out_state = rnn_layer(output, state)
390:             # Apply the dropout layer except the last layer
391:             if i < self.num_layers - 1:
392:                 output = self.dropout_layer(output)
393:             output_states += [out_state]
394:             i += 1  # noqa: SIM113
395:         return output, output_states
396: 
397: 
398: def flatten_states(states):
399:     states = list(zip(*states))
400:     if len(states) != 2:
401:         raise AssertionError(f"Expected states to have length 2, but got {len(states)}")
402:     return [torch.stack(state) for state in states]
403: 
404: 
405: def double_flatten_states(states):
406:     # XXX: Can probably write this in a nicer way
407:     states = flatten_states([flatten_states(inner) for inner in states])
408:     return [hidden.view([-1] + list(hidden.shape[2:])) for hidden in states]
409: 
410: 
411: def test_script_rnn_layer(seq_len, batch, input_size, hidden_size):
412:     inp = torch.randn(seq_len, batch, input_size)
413:     state = LSTMState(torch.randn(batch, hidden_size), torch.randn(batch, hidden_size))
414:     rnn = LSTMLayer(LSTMCell, input_size, hidden_size)
415:     out, out_state = rnn(inp, state)
416: 
417:     # Control: pytorch native LSTM
418:     lstm = nn.LSTM(input_size, hidden_size, 1)
419:     lstm_state = LSTMState(state.hx.unsqueeze(0), state.cx.unsqueeze(0))
420:     for lstm_param, custom_param in zip(lstm.all_weights[0], rnn.parameters()):
````
- EN: Implements callable logic such as `forward`, `flatten_states`, `double_flatten_states`, `test_script_rnn_layer`.
- CN: 实现可调用逻辑，例如 `forward`, `flatten_states`, `double_flatten_states`, `test_script_rnn_layer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````python
421:         if lstm_param.shape != custom_param.shape:
422:             raise AssertionError(
423:                 f"Shape mismatch: lstm_param.shape={lstm_param.shape}, custom_param.shape={custom_param.shape}"
424:             )
425:         with torch.no_grad():
426:             lstm_param.copy_(custom_param)
427:     lstm_out, lstm_out_state = lstm(inp, lstm_state)
428: 
429:     if (out - lstm_out).abs().max() >= 1e-5:
430:         raise AssertionError(
431:             f"Output mismatch: max diff={(out - lstm_out).abs().max()}"
432:         )
433:     if (out_state[0] - lstm_out_state[0]).abs().max() >= 1e-5:
434:         raise AssertionError(
435:             f"Hidden state mismatch: max diff={(out_state[0] - lstm_out_state[0]).abs().max()}"
436:         )
437:     if (out_state[1] - lstm_out_state[1]).abs().max() >= 1e-5:
438:         raise AssertionError(
439:             f"Cell state mismatch: max diff={(out_state[1] - lstm_out_state[1]).abs().max()}"
440:         )
441: 
442: 
443: def test_script_stacked_rnn(seq_len, batch, input_size, hidden_size, num_layers):
444:     inp = torch.randn(seq_len, batch, input_size)
445:     states = [
446:         LSTMState(torch.randn(batch, hidden_size), torch.randn(batch, hidden_size))
447:         for _ in range(num_layers)
448:     ]
449:     rnn = script_lstm(input_size, hidden_size, num_layers)
450:     out, out_state = rnn(inp, states)
451:     custom_state = flatten_states(out_state)
452: 
453:     # Control: pytorch native LSTM
454:     lstm = nn.LSTM(input_size, hidden_size, num_layers)
455:     lstm_state = flatten_states(states)
456:     for layer in range(num_layers):
457:         custom_params = list(rnn.parameters())[4 * layer : 4 * (layer + 1)]
458:         for lstm_param, custom_param in zip(lstm.all_weights[layer], custom_params):
459:             if lstm_param.shape != custom_param.shape:
460:                 raise AssertionError(
461:                     f"Shape mismatch at layer {layer}: lstm_param.shape={lstm_param.shape}, custom_param.shape={custom_param.shape}"
462:                 )
463:             with torch.no_grad():
464:                 lstm_param.copy_(custom_param)
465:     lstm_out, lstm_out_state = lstm(inp, lstm_state)
466: 
467:     if (out - lstm_out).abs().max() >= 1e-5:
468:         raise AssertionError(
469:             f"Output mismatch: max diff={(out - lstm_out).abs().max()}"
470:         )
471:     if (custom_state[0] - lstm_out_state[0]).abs().max() >= 1e-5:
472:         raise AssertionError(
473:             f"Hidden state mismatch: max diff={(custom_state[0] - lstm_out_state[0]).abs().max()}"
474:         )
475:     if (custom_state[1] - lstm_out_state[1]).abs().max() >= 1e-5:
476:         raise AssertionError(
477:             f"Cell state mismatch: max diff={(custom_state[1] - lstm_out_state[1]).abs().max()}"
478:         )
479: 
480: 
````
- EN: Implements callable logic such as `test_script_stacked_rnn`.
- CN: 实现可调用逻辑，例如 `test_script_stacked_rnn`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 481-540
````python
481: def test_script_stacked_bidir_rnn(seq_len, batch, input_size, hidden_size, num_layers):
482:     inp = torch.randn(seq_len, batch, input_size)
483:     states = [
484:         [
485:             LSTMState(torch.randn(batch, hidden_size), torch.randn(batch, hidden_size))
486:             for _ in range(2)
487:         ]
488:         for _ in range(num_layers)
489:     ]
490:     rnn = script_lstm(input_size, hidden_size, num_layers, bidirectional=True)
491:     out, out_state = rnn(inp, states)
492:     custom_state = double_flatten_states(out_state)
493: 
494:     # Control: pytorch native LSTM
495:     lstm = nn.LSTM(input_size, hidden_size, num_layers, bidirectional=True)
496:     lstm_state = double_flatten_states(states)
497:     for layer in range(num_layers):
498:         for direct in range(2):
499:             index = 2 * layer + direct
500:             custom_params = list(rnn.parameters())[4 * index : 4 * index + 4]
501:             for lstm_param, custom_param in zip(lstm.all_weights[index], custom_params):
502:                 if lstm_param.shape != custom_param.shape:
503:                     raise AssertionError(
504:                         f"Shape mismatch at layer {layer}, direction {direct}: "
505:                         f"lstm_param.shape={lstm_param.shape}, custom_param.shape={custom_param.shape}"
506:                     )
507:                 with torch.no_grad():
508:                     lstm_param.copy_(custom_param)
509:     lstm_out, lstm_out_state = lstm(inp, lstm_state)
510: 
511:     if (out - lstm_out).abs().max() >= 1e-5:
512:         raise AssertionError(
513:             f"Output mismatch: max diff={(out - lstm_out).abs().max()}"
514:         )
515:     if (custom_state[0] - lstm_out_state[0]).abs().max() >= 1e-5:
516:         raise AssertionError(
517:             f"Hidden state mismatch: max diff={(custom_state[0] - lstm_out_state[0]).abs().max()}"
518:         )
519:     if (custom_state[1] - lstm_out_state[1]).abs().max() >= 1e-5:
520:         raise AssertionError(
521:             f"Cell state mismatch: max diff={(custom_state[1] - lstm_out_state[1]).abs().max()}"
522:         )
523: 
524: 
525: def test_script_stacked_lstm_dropout(
526:     seq_len, batch, input_size, hidden_size, num_layers
527: ):
528:     inp = torch.randn(seq_len, batch, input_size)
529:     states = [
530:         LSTMState(torch.randn(batch, hidden_size), torch.randn(batch, hidden_size))
531:         for _ in range(num_layers)
532:     ]
533:     rnn = script_lstm(input_size, hidden_size, num_layers, dropout=True)
534: 
535:     # just a smoke test
536:     out, out_state = rnn(inp, states)
537: 
538: 
539: def test_script_stacked_lnlstm(seq_len, batch, input_size, hidden_size, num_layers):
540:     inp = torch.randn(seq_len, batch, input_size)
````
- EN: Implements callable logic such as `test_script_stacked_bidir_rnn`, `test_script_stacked_lstm_dropout`, `test_script_stacked_lnlstm`.
- CN: 实现可调用逻辑，例如 `test_script_stacked_bidir_rnn`, `test_script_stacked_lstm_dropout`, `test_script_stacked_lnlstm`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 541-555
````python
541:     states = [
542:         LSTMState(torch.randn(batch, hidden_size), torch.randn(batch, hidden_size))
543:         for _ in range(num_layers)
544:     ]
545:     rnn = script_lnlstm(input_size, hidden_size, num_layers)
546: 
547:     # just a smoke test
548:     out, out_state = rnn(inp, states)
549: 
550: 
551: test_script_rnn_layer(5, 2, 3, 7)
552: test_script_stacked_rnn(5, 2, 3, 7, 4)
553: test_script_stacked_bidir_rnn(5, 2, 3, 7, 4)
554: test_script_stacked_lstm_dropout(5, 2, 3, 7, 4)
555: test_script_stacked_lnlstm(5, 2, 3, 7, 4)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `script_lstm` / 符号 `script_lstm`
- Symbol `script_lnlstm` / 符号 `script_lnlstm`
- Symbol `reverse` / 符号 `reverse`
- Symbol `LSTMCell` / 符号 `LSTMCell`

## Dependencies / 依赖关系
- Python imports: `numbers`, `warnings`, `collections`, `torch`, `torch.jit`, `torch.nn`
- Python 导入: `numbers`, `warnings`, `collections`, `torch`, `torch.jit`, `torch.nn`

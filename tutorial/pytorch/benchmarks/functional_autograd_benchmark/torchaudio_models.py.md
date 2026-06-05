# torchaudio_models.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/functional_autograd_benchmark/torchaudio_models.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````python
 1: # Taken from https://github.com/pytorch/audio/blob/master/torchaudio/models/wav2letter.py
 2: # So that we don't need torchaudio to be installed
 3: 
 4: import math
 5: from collections import OrderedDict
 6: 
 7: import torch
 8: import torch.nn.functional as F
 9: from torch import nn, Tensor
10: 
11: 
12: __all__ = ["Wav2Letter"]
13: 
14: 
15: class Wav2Letter(nn.Module):
16:     r"""Wav2Letter model architecture from the `"Wav2Letter: an End-to-End ConvNet-based Speech Recognition System"
17:      <https://arxiv.org/abs/1609.03193>`_ paper.
18:      :math:`\text{padding} = \frac{\text{ceil}(\text{kernel} - \text{stride})}{2}`
19:     Args:
20:         num_classes (int, optional): Number of classes to be classified. (Default: ``40``)
21:         input_type (str, optional): Wav2Letter can use as input: ``waveform``, ``power_spectrum``
22:          or ``mfcc`` (Default: ``waveform``).
23:         num_features (int, optional): Number of input features that the network will receive (Default: ``1``).
24:     """
25: 
26:     def __init__(
27:         self, num_classes: int = 40, input_type: str = "waveform", num_features: int = 1
28:     ) -> None:
29:         super().__init__()
30: 
31:         acoustic_num_features = 250 if input_type == "waveform" else num_features
32:         acoustic_model = nn.Sequential(
33:             nn.Conv1d(
34:                 in_channels=acoustic_num_features,
35:                 out_channels=250,
36:                 kernel_size=48,
37:                 stride=2,
38:                 padding=23,
39:             ),
40:             nn.ReLU(inplace=True),
41:             nn.Conv1d(
42:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
43:             ),
44:             nn.ReLU(inplace=True),
45:             nn.Conv1d(
46:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
47:             ),
48:             nn.ReLU(inplace=True),
49:             nn.Conv1d(
50:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
51:             ),
52:             nn.ReLU(inplace=True),
53:             nn.Conv1d(
54:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
55:             ),
56:             nn.ReLU(inplace=True),
57:             nn.Conv1d(
58:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
59:             ),
60:             nn.ReLU(inplace=True),
````
- EN: Handles module imports such as `math`, `collections`, `torch`, `torch.nn.functional`.
- CN: 处理模块导入，例如 `math`, `collections`, `torch`, `torch.nn.functional`。
- EN: Declares or extends types including `Wav2Letter`.
- CN: 声明或扩展类型，包括 `Wav2Letter`。
- EN: Implements callable logic such as `__init__`.
- CN: 实现可调用逻辑，例如 `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````python
 61:             nn.Conv1d(
 62:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
 63:             ),
 64:             nn.ReLU(inplace=True),
 65:             nn.Conv1d(
 66:                 in_channels=250, out_channels=250, kernel_size=7, stride=1, padding=3
 67:             ),
 68:             nn.ReLU(inplace=True),
 69:             nn.Conv1d(
 70:                 in_channels=250, out_channels=2000, kernel_size=32, stride=1, padding=16
 71:             ),
 72:             nn.ReLU(inplace=True),
 73:             nn.Conv1d(
 74:                 in_channels=2000, out_channels=2000, kernel_size=1, stride=1, padding=0
 75:             ),
 76:             nn.ReLU(inplace=True),
 77:             nn.Conv1d(
 78:                 in_channels=2000,
 79:                 out_channels=num_classes,
 80:                 kernel_size=1,
 81:                 stride=1,
 82:                 padding=0,
 83:             ),
 84:             nn.ReLU(inplace=True),
 85:         )
 86: 
 87:         if input_type == "waveform":
 88:             waveform_model = nn.Sequential(
 89:                 nn.Conv1d(
 90:                     in_channels=num_features,
 91:                     out_channels=250,
 92:                     kernel_size=250,
 93:                     stride=160,
 94:                     padding=45,
 95:                 ),
 96:                 nn.ReLU(inplace=True),
 97:             )
 98:             self.acoustic_model = nn.Sequential(waveform_model, acoustic_model)
 99: 
100:         if input_type in ["power_spectrum", "mfcc"]:
101:             self.acoustic_model = acoustic_model
102: 
103:     def forward(self, x: Tensor) -> Tensor:
104:         r"""
105:         Args:
106:             x (Tensor): Tensor of dimension (batch_size, num_features, input_length).
107:         Returns:
108:             Tensor: Predictor tensor of dimension (batch_size, number_of_classes, input_length).
109:         """
110: 
111:         x = self.acoustic_model(x)
112:         x = nn.functional.log_softmax(x, dim=1)
113:         return x
114: 
115: 
116: # Taken from  https://github.com/SeanNaren/deepspeech.pytorch with modifications
117: class SequenceWise(nn.Module):
118:     def __init__(self, module):
119:         """
120:         Collapses input of dim T*N*H to (T*N)*H, and applies to a module.
````
- EN: Declares or extends types including `SequenceWise`.
- CN: 声明或扩展类型，包括 `SequenceWise`。
- EN: Implements callable logic such as `forward`, `__init__`.
- CN: 实现可调用逻辑，例如 `forward`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````python
121:         Allows handling of variable sequence lengths and minibatch sizes.
122:         :param module: Module to apply input to.
123:         """
124:         super().__init__()
125:         self.module = module
126: 
127:     def forward(self, x):
128:         t, n = x.size(0), x.size(1)
129:         x = x.view(t * n, -1)
130:         x = self.module(x)
131:         x = x.view(t, n, -1)
132:         return x
133: 
134:     def __repr__(self):
135:         tmpstr = self.__class__.__name__ + " (\n"
136:         tmpstr += self.module.__repr__()
137:         tmpstr += ")"
138:         return tmpstr
139: 
140: 
141: class MaskConv(nn.Module):
142:     def __init__(self, seq_module):
143:         """
144:         Adds padding to the output of the module based on the given lengths. This is to ensure that the
145:         results of the model do not change when batch sizes change during inference.
146:         Input needs to be in the shape of (BxCxDxT)
147:         :param seq_module: The sequential module containing the conv stack.
148:         """
149:         super().__init__()
150:         self.seq_module = seq_module
151: 
152:     def forward(self, x, lengths):
153:         """
154:         :param x: The input of size BxCxDxT
155:         :param lengths: The actual length of each sequence in the batch
156:         :return: Masked output from the module
157:         """
158:         for module in self.seq_module:
159:             x = module(x)
160:             mask = torch.BoolTensor(x.size()).fill_(0)
161:             if x.is_cuda:
162:                 mask = mask.cuda()
163:             for i, length in enumerate(lengths):
164:                 length = length.item()
165:                 if (mask[i].size(2) - length) > 0:
166:                     mask[i].narrow(2, length, mask[i].size(2) - length).fill_(1)
167:             x = x.masked_fill(mask, 0)
168:         return x, lengths
169: 
170: 
171: class InferenceBatchSoftmax(nn.Module):
172:     def forward(self, input_):
173:         if not self.training:
174:             return F.softmax(input_, dim=-1)
175:         else:
176:             return input_
177: 
178: 
179: class BatchRNN(nn.Module):
180:     def __init__(
````
- EN: Declares or extends types including `MaskConv`, `InferenceBatchSoftmax`, `BatchRNN`.
- CN: 声明或扩展类型，包括 `MaskConv`, `InferenceBatchSoftmax`, `BatchRNN`。
- EN: Implements callable logic such as `forward`, `__repr__`, `__init__`.
- CN: 实现可调用逻辑，例如 `forward`, `__repr__`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````python
181:         self,
182:         input_size,
183:         hidden_size,
184:         rnn_type=nn.LSTM,
185:         bidirectional=False,
186:         batch_norm=True,
187:     ):
188:         super().__init__()
189:         self.input_size = input_size
190:         self.hidden_size = hidden_size
191:         self.bidirectional = bidirectional
192:         self.batch_norm = (
193:             SequenceWise(nn.BatchNorm1d(input_size)) if batch_norm else None
194:         )
195:         self.rnn = rnn_type(
196:             input_size=input_size,
197:             hidden_size=hidden_size,
198:             bidirectional=bidirectional,
199:             bias=True,
200:         )
201:         self.num_directions = 2 if bidirectional else 1
202: 
203:     def flatten_parameters(self):
204:         self.rnn.flatten_parameters()
205: 
206:     def forward(self, x, output_lengths):
207:         if self.batch_norm is not None:
208:             x = self.batch_norm(x)
209:         x = nn.utils.rnn.pack_padded_sequence(x, output_lengths, enforce_sorted=False)
210:         x, h = self.rnn(x)
211:         x, _ = nn.utils.rnn.pad_packed_sequence(x)
212:         if self.bidirectional:
213:             x = (
214:                 x.view(x.size(0), x.size(1), 2, -1)
215:                 .sum(2)
216:                 .view(x.size(0), x.size(1), -1)
217:             )  # (TxNxH*2) -> (TxNxH) by sum
218:         return x
219: 
220: 
221: class Lookahead(nn.Module):
222:     # Wang et al., 2016 - Lookahead Convolution Layer for Unidirectional Recurrent Neural Networks
223:     # input shape - sequence, batch, feature - TxNxH
224:     # output shape - same as input
225:     def __init__(self, n_features, context):
226:         super().__init__()
227:         if context <= 0:
228:             raise AssertionError(f"context must be positive, but got {context}")
229:         self.context = context
230:         self.n_features = n_features
231:         self.pad = (0, self.context - 1)
232:         self.conv = nn.Conv1d(
233:             self.n_features,
234:             self.n_features,
235:             kernel_size=self.context,
236:             stride=1,
237:             groups=self.n_features,
238:             padding=0,
239:             bias=None,
240:         )
````
- EN: Declares or extends types including `Lookahead`.
- CN: 声明或扩展类型，包括 `Lookahead`。
- EN: Implements callable logic such as `flatten_parameters`, `forward`, `__init__`.
- CN: 实现可调用逻辑，例如 `flatten_parameters`, `forward`, `__init__`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````python
241: 
242:     def forward(self, x):
243:         x = x.transpose(0, 1).transpose(1, 2)
244:         x = F.pad(x, pad=self.pad, value=0)
245:         x = self.conv(x)
246:         x = x.transpose(1, 2).transpose(0, 1).contiguous()
247:         return x
248: 
249:     def __repr__(self):
250:         return (
251:             self.__class__.__name__
252:             + "("
253:             + "n_features="
254:             + str(self.n_features)
255:             + ", context="
256:             + str(self.context)
257:             + ")"
258:         )
259: 
260: 
261: class DeepSpeech(nn.Module):
262:     def __init__(
263:         self,
264:         rnn_type,
265:         labels,
266:         rnn_hidden_size,
267:         nb_layers,
268:         audio_conf,
269:         bidirectional,
270:         context=20,
271:     ):
272:         super().__init__()
273: 
274:         self.hidden_size = rnn_hidden_size
275:         self.hidden_layers = nb_layers
276:         self.rnn_type = rnn_type
277:         self.audio_conf = audio_conf
278:         self.labels = labels
279:         self.bidirectional = bidirectional
280: 
281:         sample_rate = self.audio_conf["sample_rate"]
282:         window_size = self.audio_conf["window_size"]
283:         num_classes = len(self.labels)
284: 
285:         self.conv = MaskConv(
286:             nn.Sequential(
287:                 nn.Conv2d(1, 32, kernel_size=(41, 11), stride=(2, 2), padding=(20, 5)),
288:                 nn.BatchNorm2d(32),
289:                 nn.Hardtanh(0, 20, inplace=True),
290:                 nn.Conv2d(32, 32, kernel_size=(21, 11), stride=(2, 1), padding=(10, 5)),
291:                 nn.BatchNorm2d(32),
292:                 nn.Hardtanh(0, 20, inplace=True),
293:             )
294:         )
295:         # Based on above convolutions and spectrogram size using conv formula (W - F + 2P)/ S+1
296:         rnn_input_size = int(math.floor((sample_rate * window_size) / 2) + 1)
297:         rnn_input_size = int(math.floor(rnn_input_size + 2 * 20 - 41) / 2 + 1)
298:         rnn_input_size = int(math.floor(rnn_input_size + 2 * 10 - 21) / 2 + 1)
299:         rnn_input_size *= 32
300: 
````
- EN: Declares or extends types including `DeepSpeech`.
- CN: 声明或扩展类型，包括 `DeepSpeech`。
- EN: Implements callable logic such as `forward`, `__repr__`, `__init__`.
- CN: 实现可调用逻辑，例如 `forward`, `__repr__`, `__init__`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````python
301:         rnns = []
302:         rnn = BatchRNN(
303:             input_size=rnn_input_size,
304:             hidden_size=rnn_hidden_size,
305:             rnn_type=rnn_type,
306:             bidirectional=bidirectional,
307:             batch_norm=False,
308:         )
309:         rnns.append(("0", rnn))
310:         for x in range(nb_layers - 1):
311:             rnn = BatchRNN(
312:                 input_size=rnn_hidden_size,
313:                 hidden_size=rnn_hidden_size,
314:                 rnn_type=rnn_type,
315:                 bidirectional=bidirectional,
316:             )
317:             rnns.append((f"{x + 1:d}", rnn))
318:         self.rnns = nn.Sequential(OrderedDict(rnns))
319:         self.lookahead = (
320:             nn.Sequential(
321:                 # consider adding batch norm?
322:                 Lookahead(rnn_hidden_size, context=context),
323:                 nn.Hardtanh(0, 20, inplace=True),
324:             )
325:             if not bidirectional
326:             else None
327:         )
328: 
329:         fully_connected = nn.Sequential(
330:             nn.BatchNorm1d(rnn_hidden_size),
331:             nn.Linear(rnn_hidden_size, num_classes, bias=False),
332:         )
333:         self.fc = nn.Sequential(
334:             SequenceWise(fully_connected),
335:         )
336:         self.inference_softmax = InferenceBatchSoftmax()
337: 
338:     def forward(self, x, lengths):
339:         lengths = lengths.cpu().int()
340:         output_lengths = self.get_seq_lens(lengths)
341:         x, _ = self.conv(x, output_lengths)
342: 
343:         sizes = x.size()
344:         x = x.view(
345:             sizes[0], sizes[1] * sizes[2], sizes[3]
346:         )  # Collapse feature dimension
347:         x = x.transpose(1, 2).transpose(0, 1).contiguous()  # TxNxH
348: 
349:         for rnn in self.rnns:
350:             x = rnn(x, output_lengths)
351: 
352:         if not self.bidirectional:  # no need for lookahead layer in bidirectional
353:             x = self.lookahead(x)
354: 
355:         x = self.fc(x)
356:         x = x.transpose(0, 1)
357:         # identity in training mode, softmax in eval mode
358:         x = self.inference_softmax(x)
359:         return x, output_lengths
360: 
````
- EN: Implements callable logic such as `forward`.
- CN: 实现可调用逻辑，例如 `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 361-420
````python
361:     def get_seq_lens(self, input_length):
362:         """
363:         Given a 1D Tensor or Variable containing integer sequence lengths, return a 1D tensor or variable
364:         containing the size sequences that will be output by the network.
365:         :param input_length: 1D Tensor
366:         :return: 1D Tensor scaled by model
367:         """
368:         seq_len = input_length
369:         for m in self.conv.modules():
370:             if type(m) is nn.modules.conv.Conv2d:
371:                 seq_len = (
372:                     seq_len
373:                     + 2 * m.padding[1]
374:                     - m.dilation[1] * (m.kernel_size[1] - 1)
375:                     - 1
376:                 )
377:                 seq_len = seq_len.true_divide(m.stride[1]) + 1
378:         return seq_len.int()
379: 
380: 
381: # Taken from https://github.com/pytorch/examples/blob/master/word_language_model/model.py#L108-L152
382: class PositionalEncoding(nn.Module):
383:     r"""Inject some information about the relative or absolute position of the tokens
384:         in the sequence. The positional encodings have the same dimension as
385:         the embeddings, so that the two can be summed. Here, we use sine and cosine
386:         functions of different frequencies.
387:     .. math::
388:         \text{PosEncoder}(pos, 2i) = sin(pos/10000^(2i/d_model))
389:         \text{PosEncoder}(pos, 2i+1) = cos(pos/10000^(2i/d_model))
390:         \text{where pos is the word position and i is the embed idx)
391:     Args:
392:         d_model: the embed dim (required).
393:         dropout: the dropout value (default=0.1).
394:         max_len: the max. length of the incoming sequence (default=5000).
395:     Examples:
396:         >>> pos_encoder = PositionalEncoding(d_model)
397:     """
398: 
399:     def __init__(self, d_model, dropout=0.1, max_len=5000):
400:         super().__init__()
401:         self.dropout = nn.Dropout(p=dropout)
402: 
403:         pe = torch.zeros(max_len, d_model)
404:         position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
405:         div_term = torch.exp(
406:             torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model)
407:         )
408:         pe[:, 0::2] = torch.sin(position * div_term)
409:         pe[:, 1::2] = torch.cos(position * div_term)
410:         pe = pe.unsqueeze(0).transpose(0, 1)
411:         self.register_buffer("pe", pe)
412: 
413:     def forward(self, x):
414:         r"""Inputs of forward function
415:         Args:
416:             x: the sequence fed to the positional encoder model (required).
417:         Shape:
418:             x: [sequence length, batch size, embed dim]
419:             output: [sequence length, batch size, embed dim]
420:         Examples:
````
- EN: Declares or extends types including `PositionalEncoding`.
- CN: 声明或扩展类型，包括 `PositionalEncoding`。
- EN: Implements callable logic such as `get_seq_lens`, `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `get_seq_lens`, `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 421-480
````python
421:             >>> output = pos_encoder(x)
422:         """
423: 
424:         x = x + self.pe[: x.size(0), :]
425:         return self.dropout(x)
426: 
427: 
428: class TransformerModel(nn.Module):
429:     """Container module with an encoder, a recurrent or transformer module, and a decoder."""
430: 
431:     def __init__(self, ntoken, ninp, nhead, nhid, nlayers, dropout=0.5):
432:         super().__init__()
433:         try:
434:             from torch.nn import TransformerEncoder, TransformerEncoderLayer
435:         except Exception as e:
436:             raise ImportError(
437:                 "TransformerEncoder module does not exist in PyTorch 1.1 or lower."
438:             ) from e
439:         self.model_type = "Transformer"
440:         self.src_mask = None
441:         self.pos_encoder = PositionalEncoding(ninp, dropout)
442:         encoder_layers = TransformerEncoderLayer(ninp, nhead, nhid, dropout)
443:         self.transformer_encoder = TransformerEncoder(encoder_layers, nlayers)
444:         self.encoder = nn.Embedding(ntoken, ninp)
445:         self.ninp = ninp
446:         self.decoder = nn.Linear(ninp, ntoken)
447: 
448:         self.init_weights()
449: 
450:     def init_weights(self):
451:         initrange = 0.1
452:         nn.init.uniform_(self.encoder.weight, -initrange, initrange)
453:         # Not sure how this works in the original code
454:         # nn.init.zeros_(self.decoder)
455:         nn.init.uniform_(self.decoder.weight, -initrange, initrange)
456: 
457:     def forward(self, src, has_mask=True):
458:         if has_mask:
459:             device = src.device
460:             # This will be created once during warmup
461:             if self.src_mask is None or self.src_mask.size(0) != len(src):
462:                 mask = nn.Transformer.generate_square_subsequent_mask(len(src)).to(
463:                     device
464:                 )
465:                 self.src_mask = mask
466:         else:
467:             self.src_mask = None
468: 
469:         src = self.encoder(src) * math.sqrt(self.ninp)
470:         src = self.pos_encoder(src)
471:         output = self.transformer_encoder(src, self.src_mask)
472:         output = self.decoder(output)
473:         return F.log_softmax(output, dim=-1)
474: 
475: 
476: # From https://github.com/pytorch/text/tree/master/torchtext/nn/modules
477: class MultiheadAttentionContainer(torch.nn.Module):
478:     def __init__(self, nhead, in_proj_container, attention_layer, out_proj):
479:         r"""A multi-head attention container
480:         Args:
````
- EN: Handles module imports such as `torch.nn`.
- CN: 处理模块导入，例如 `torch.nn`。
- EN: Declares or extends types including `TransformerModel`, `MultiheadAttentionContainer`.
- CN: 声明或扩展类型，包括 `TransformerModel`, `MultiheadAttentionContainer`。
- EN: Implements callable logic such as `__init__`, `init_weights`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `init_weights`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 481-540
````python
481:             nhead: the number of heads in the multiheadattention model
482:             in_proj_container: A container of multi-head in-projection linear layers (a.k.a nn.Linear).
483:             attention_layer: The attention layer.
484:             out_proj: The multi-head out-projection layer (a.k.a nn.Linear).
485:         Examples::
486:             >>> import torch
487:             >>> embed_dim, num_heads, bsz = 10, 5, 64
488:             >>> in_proj_container = InProjContainer(torch.nn.Linear(embed_dim, embed_dim),
489:                                                     torch.nn.Linear(embed_dim, embed_dim),
490:                                                     torch.nn.Linear(embed_dim, embed_dim))
491:             >>> MHA = MultiheadAttentionContainer(num_heads,
492:                                                   in_proj_container,
493:                                                   ScaledDotProduct(),
494:                                                   torch.nn.Linear(embed_dim, embed_dim))
495:             >>> query = torch.rand((21, bsz, embed_dim))
496:             >>> key = value = torch.rand((16, bsz, embed_dim))
497:             >>> attn_output, attn_weights = MHA(query, key, value)
498:             >>> print(attn_output.shape)
499:             >>> torch.Size([21, 64, 10])
500:         """
501:         super().__init__()
502:         self.nhead = nhead
503:         self.in_proj_container = in_proj_container
504:         self.attention_layer = attention_layer
505:         self.out_proj = out_proj
506: 
507:     def forward(
508:         self,
509:         query: torch.Tensor,
510:         key: torch.Tensor,
511:         value: torch.Tensor,
512:         attn_mask: torch.Tensor | None = None,
513:         bias_k: torch.Tensor | None = None,
514:         bias_v: torch.Tensor | None = None,
515:     ) -> tuple[torch.Tensor, torch.Tensor]:
516:         r"""
517:         Args:
518:             query, key, value (Tensor): map a query and a set of key-value pairs to an output.
519:                 See "Attention Is All You Need" for more details.
520:             attn_mask, bias_k and bias_v (Tensor, optional): keyword arguments passed to the attention layer.
521:                 See the definitions in the attention.
522:         Shape:
523:             - Inputs:
524:             - query: :math:`(L, N, E)`
525:             - key: :math:`(S, N, E)`
526:             - value: :math:`(S, N, E)`
527:             - attn_mask, bias_k and bias_v: same with the shape of the corresponding args in attention layer.
528:             - Outputs:
529:             - attn_output: :math:`(L, N, E)`
530:             - attn_output_weights: :math:`(N * H, L, S)`
531:             where where L is the target length, S is the sequence length, H is the number of attention heads,
532:                 N is the batch size, and E is the embedding dimension.
533:         """
534:         tgt_len, src_len, bsz, embed_dim = (
535:             query.size(-3),
536:             key.size(-3),
537:             query.size(-2),
538:             query.size(-1),
539:         )
540:         q, k, v = self.in_proj_container(query, key, value)
````
- EN: Implements callable logic such as `forward`.
- CN: 实现可调用逻辑，例如 `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 541-600
````python
541:         if q.size(-1) % self.nhead != 0:
542:             raise AssertionError(
543:                 f"query's embed_dim ({q.size(-1)}) must be divisible by the number of heads ({self.nhead})"
544:             )
545:         head_dim = q.size(-1) // self.nhead
546:         q = q.reshape(tgt_len, bsz * self.nhead, head_dim)
547: 
548:         if k.size(-1) % self.nhead != 0:
549:             raise AssertionError(
550:                 f"key's embed_dim ({k.size(-1)}) must be divisible by the number of heads ({self.nhead})"
551:             )
552:         head_dim = k.size(-1) // self.nhead
553:         k = k.reshape(src_len, bsz * self.nhead, head_dim)
554: 
555:         if v.size(-1) % self.nhead != 0:
556:             raise AssertionError(
557:                 f"value's embed_dim ({v.size(-1)}) must be divisible by the number of heads ({self.nhead})"
558:             )
559:         head_dim = v.size(-1) // self.nhead
560:         v = v.reshape(src_len, bsz * self.nhead, head_dim)
561: 
562:         attn_output, attn_output_weights = self.attention_layer(
563:             q, k, v, attn_mask=attn_mask, bias_k=bias_k, bias_v=bias_v
564:         )
565:         attn_output = attn_output.reshape(tgt_len, bsz, embed_dim)
566:         attn_output = self.out_proj(attn_output)
567:         return attn_output, attn_output_weights
568: 
569: 
570: class ScaledDotProduct(torch.nn.Module):
571:     def __init__(self, dropout=0.0):
572:         r"""Processes a projected query and key-value pair to apply
573:         scaled dot product attention.
574:         Args:
575:             dropout (float): probability of dropping an attention weight.
576:         Examples::
577:             >>> SDP = torchtext.models.ScaledDotProduct(0.1)
578:             >>> q = torch.randn(256, 21, 3)
579:             >>> k = v = torch.randn(256, 21, 3)
580:             >>> attn_output, attn_weights = SDP(q, k, v)
581:             >>> print(attn_output.shape, attn_weights.shape)
582:             torch.Size([256, 21, 3]) torch.Size([256, 21, 21])
583:         """
584:         super().__init__()
585:         self.dropout = dropout
586: 
587:     def forward(
588:         self,
589:         query: torch.Tensor,
590:         key: torch.Tensor,
591:         value: torch.Tensor,
592:         attn_mask: torch.Tensor | None = None,
593:         bias_k: torch.Tensor | None = None,
594:         bias_v: torch.Tensor | None = None,
595:     ) -> tuple[torch.Tensor, torch.Tensor]:
596:         r"""Uses a scaled dot product with the projected key-value pair to update
597:         the projected query.
598:         Args:
599:             query (Tensor): Projected query
600:             key (Tensor): Projected key
````
- EN: Declares or extends types including `ScaledDotProduct`.
- CN: 声明或扩展类型，包括 `ScaledDotProduct`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 601-660
````python
601:             value (Tensor): Projected value
602:             attn_mask (BoolTensor, optional): 3D mask that prevents attention to certain positions.
603:             bias_k and bias_v: (Tensor, optional): one more key and value sequence to be added at
604:                 sequence dim (dim=-3). Those are used for incremental decoding. Users should provide
605:                 non-None to both arguments in order to activate them.
606:         Shape:
607:             - query: :math:`(L, N * H, E / H)`
608:             - key: :math:`(S, N * H, E / H)`
609:             - value: :math:`(S, N * H, E / H)`
610:             - attn_mask: :math:`(N * H, L, S)`, positions with ``True`` are not allowed to attend
611:                 while ``False`` values will be unchanged.
612:             - bias_k and bias_v:bias: :math:`(1, N * H, E / H)`
613:             - Output: :math:`(L, N * H, E / H)`, :math:`(N * H, L, S)`
614:             where L is the target length, S is the source length, H is the number
615:             of attention heads, N is the batch size, and E is the embedding dimension.
616:         """
617:         if bias_k is not None and bias_v is not None:
618:             if not (
619:                 key.size(-1) == bias_k.size(-1)
620:                 and key.size(-2) == bias_k.size(-2)
621:                 and bias_k.size(-3) == 1
622:             ):
623:                 raise AssertionError(
624:                     f"Shape of bias_k is not supported: key.shape={key.shape}, bias_k.shape={bias_k.shape}"
625:                 )
626:             if not (
627:                 value.size(-1) == bias_v.size(-1)
628:                 and value.size(-2) == bias_v.size(-2)
629:                 and bias_v.size(-3) == 1
630:             ):
631:                 raise AssertionError(
632:                     f"Shape of bias_v is not supported: value.shape={value.shape}, bias_v.shape={bias_v.shape}"
633:                 )
634:             key = torch.cat([key, bias_k])
635:             value = torch.cat([value, bias_v])
636:             if attn_mask is not None:
637:                 _attn_mask = attn_mask
638:                 attn_mask = torch.nn.functional.pad(_attn_mask, [0, 1])
639: 
640:         tgt_len, head_dim = query.size(-3), query.size(-1)
641:         if not (query.size(-1) == key.size(-1) == value.size(-1)):
642:             raise AssertionError(
643:                 f"The feature dim of query, key, value must be equal: "
644:                 f"query={query.size(-1)}, key={key.size(-1)}, value={value.size(-1)}"
645:             )
646:         if key.size() != value.size():
647:             raise AssertionError(
648:                 f"Shape of key, value must match: key.shape={key.shape}, value.shape={value.shape}"
649:             )
650:         src_len = key.size(-3)
651:         batch_heads = max(query.size(-2), key.size(-2))
652: 
653:         # Scale query
654:         query, key, value = (
655:             query.transpose(-2, -3),
656:             key.transpose(-2, -3),
657:             value.transpose(-2, -3),
658:         )
659:         query = query * (float(head_dim) ** -0.5)
660:         if attn_mask is not None:
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 661-712
````python
661:             if attn_mask.dim() != 3:
662:                 raise RuntimeError("attn_mask must be a 3D tensor.")
663:             if (
664:                 (attn_mask.size(-1) != src_len)
665:                 or (attn_mask.size(-2) != tgt_len)
666:                 or (attn_mask.size(-3) != 1 and attn_mask.size(-3) != batch_heads)
667:             ):
668:                 raise RuntimeError("The size of the attn_mask is not correct.")
669:             if attn_mask.dtype != torch.bool:
670:                 raise RuntimeError("Only bool tensor is supported for attn_mask")
671: 
672:         # Dot product of q, k
673:         attn_output_weights = torch.matmul(query, key.mT)
674:         if attn_mask is not None:
675:             attn_output_weights.masked_fill_(
676:                 attn_mask,
677:                 -1e8,
678:             )
679:         attn_output_weights = torch.nn.functional.softmax(attn_output_weights, dim=-1)
680:         attn_output_weights = torch.nn.functional.dropout(
681:             attn_output_weights, p=self.dropout, training=self.training
682:         )
683:         attn_output = torch.matmul(attn_output_weights, value)
684:         return attn_output.transpose(-2, -3), attn_output_weights
685: 
686: 
687: class InProjContainer(torch.nn.Module):
688:     def __init__(self, query_proj, key_proj, value_proj):
689:         r"""A in-proj container to process inputs.
690:         Args:
691:             query_proj: a proj layer for query.
692:             key_proj: a proj layer for key.
693:             value_proj: a proj layer for value.
694:         """
695: 
696:         super().__init__()
697:         self.query_proj = query_proj
698:         self.key_proj = key_proj
699:         self.value_proj = value_proj
700: 
701:     def forward(
702:         self, query: torch.Tensor, key: torch.Tensor, value: torch.Tensor
703:     ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
704:         r"""Projects the input sequences using in-proj layers.
705:         Args:
706:             query, key, value (Tensors): sequence to be projected
707:         Shape:
708:             - query, key, value: :math:`(S, N, E)`
709:             - Output: :math:`(S, N, E)`
710:             where S is the sequence length, N is the batch size, and E is the embedding dimension.
711:         """
712:         return self.query_proj(query), self.key_proj(key), self.value_proj(value)
````
- EN: Declares or extends types including `InProjContainer`.
- CN: 声明或扩展类型，包括 `InProjContainer`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `Wav2Letter` / 符号 `Wav2Letter`
- Symbol `__init__` / 符号 `__init__`
- Symbol `forward` / 符号 `forward`
- Symbol `SequenceWise` / 符号 `SequenceWise`

## Dependencies / 依赖关系
- Python imports: `math`, `collections`, `torch`, `torch.nn.functional`, `torch.nn`
- Python 导入: `math`, `collections`, `torch`, `torch.nn.functional`, `torch.nn`

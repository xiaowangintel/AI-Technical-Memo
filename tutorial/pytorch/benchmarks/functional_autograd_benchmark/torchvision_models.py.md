# torchvision_models.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/functional_autograd_benchmark/torchvision_models.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80
````python
 1: # Taken from https://github.com/pytorch/vision
 2: # So that we don't need torchvision to be installed
 3: from collections import OrderedDict
 4: 
 5: import torch
 6: from torch import nn
 7: from torch.jit.annotations import Dict
 8: from torch.nn import functional as F
 9: 
10: 
11: try:
12:     from scipy.optimize import linear_sum_assignment
13: 
14:     scipy_available = True
15: except Exception:
16:     scipy_available = False
17: 
18: 
19: def conv3x3(in_planes, out_planes, stride=1, groups=1, dilation=1):
20:     """3x3 convolution with padding"""
21:     return nn.Conv2d(
22:         in_planes,
23:         out_planes,
24:         kernel_size=3,
25:         stride=stride,
26:         padding=dilation,
27:         groups=groups,
28:         bias=False,
29:         dilation=dilation,
30:     )
31: 
32: 
33: def conv1x1(in_planes, out_planes, stride=1):
34:     """1x1 convolution"""
35:     return nn.Conv2d(in_planes, out_planes, kernel_size=1, stride=stride, bias=False)
36: 
37: 
38: class BasicBlock(nn.Module):
39:     expansion = 1
40: 
41:     def __init__(
42:         self,
43:         inplanes,
44:         planes,
45:         stride=1,
46:         downsample=None,
47:         groups=1,
48:         base_width=64,
49:         dilation=1,
50:         norm_layer=None,
51:     ):
52:         super().__init__()
53:         if norm_layer is None:
54:             norm_layer = nn.BatchNorm2d
55:         if groups != 1 or base_width != 64:
56:             raise ValueError("BasicBlock only supports groups=1 and base_width=64")
57:         if dilation > 1:
58:             raise NotImplementedError("Dilation > 1 not supported in BasicBlock")
59:         # Both self.conv1 and self.downsample layers downsample the input when stride != 1
60:         self.conv1 = conv3x3(inplanes, planes, stride)
61:         self.bn1 = norm_layer(planes)
62:         self.relu = nn.ReLU(inplace=True)
63:         self.conv2 = conv3x3(planes, planes)
64:         self.bn2 = norm_layer(planes)
65:         self.downsample = downsample
66:         self.stride = stride
67: 
68:     def forward(self, x):
69:         identity = x
70: 
71:         out = self.conv1(x)
72:         out = self.bn1(out)
73:         out = self.relu(out)
74: 
75:         out = self.conv2(out)
76:         out = self.bn2(out)
77: 
78:         if self.downsample is not None:
79:             identity = self.downsample(x)
80: 
````
- EN: Handles module imports such as `collections`, `torch`, `torch.jit.annotations`, `torch.nn`.
- CN: 处理模块导入，例如 `collections`, `torch`, `torch.jit.annotations`, `torch.nn`。
- EN: Declares or extends types including `BasicBlock`.
- CN: 声明或扩展类型，包括 `BasicBlock`。
- EN: Implements callable logic such as `conv3x3`, `conv1x1`, `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `conv3x3`, `conv1x1`, `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-160
````python
 81:         out += identity
 82:         out = self.relu(out)
 83: 
 84:         return out
 85: 
 86: 
 87: class Bottleneck(nn.Module):
 88:     # Bottleneck in torchvision places the stride for downsampling at 3x3 convolution(self.conv2)
 89:     # while original implementation places the stride at the first 1x1 convolution(self.conv1)
 90:     # according to "Deep residual learning for image recognition"https://arxiv.org/abs/1512.03385.
 91:     # This variant is also known as ResNet V1.5 and improves accuracy according to
 92:     # https://ngc.nvidia.com/catalog/model-scripts/nvidia:resnet_50_v1_5_for_pytorch.
 93: 
 94:     expansion = 4
 95: 
 96:     def __init__(
 97:         self,
 98:         inplanes,
 99:         planes,
100:         stride=1,
101:         downsample=None,
102:         groups=1,
103:         base_width=64,
104:         dilation=1,
105:         norm_layer=None,
106:     ):
107:         super().__init__()
108:         if norm_layer is None:
109:             norm_layer = nn.BatchNorm2d
110:         width = int(planes * (base_width / 64.0)) * groups
111:         # Both self.conv2 and self.downsample layers downsample the input when stride != 1
112:         self.conv1 = conv1x1(inplanes, width)
113:         self.bn1 = norm_layer(width)
114:         self.conv2 = conv3x3(width, width, stride, groups, dilation)
115:         self.bn2 = norm_layer(width)
116:         self.conv3 = conv1x1(width, planes * self.expansion)
117:         self.bn3 = norm_layer(planes * self.expansion)
118:         self.relu = nn.ReLU(inplace=True)
119:         self.downsample = downsample
120:         self.stride = stride
121: 
122:     def forward(self, x):
123:         identity = x
124: 
125:         out = self.conv1(x)
126:         out = self.bn1(out)
127:         out = self.relu(out)
128: 
129:         out = self.conv2(out)
130:         out = self.bn2(out)
131:         out = self.relu(out)
132: 
133:         out = self.conv3(out)
134:         out = self.bn3(out)
135: 
136:         if self.downsample is not None:
137:             identity = self.downsample(x)
138: 
139:         out += identity
140:         out = self.relu(out)
141: 
142:         return out
143: 
144: 
145: class ResNet(nn.Module):
146:     def __init__(
147:         self,
148:         block,
149:         layers,
150:         num_classes=1000,
151:         zero_init_residual=False,
152:         groups=1,
153:         width_per_group=64,
154:         replace_stride_with_dilation=None,
155:         norm_layer=None,
156:     ):
157:         super().__init__()
158:         if norm_layer is None:
159:             norm_layer = nn.BatchNorm2d
160:         self._norm_layer = norm_layer
````
- EN: Declares or extends types including `Bottleneck`, `ResNet`.
- CN: 声明或扩展类型，包括 `Bottleneck`, `ResNet`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-240
````python
161: 
162:         self.inplanes = 64
163:         self.dilation = 1
164:         if replace_stride_with_dilation is None:
165:             # each element in the tuple indicates if we should replace
166:             # the 2x2 stride with a dilated convolution instead
167:             replace_stride_with_dilation = [False, False, False]
168:         if len(replace_stride_with_dilation) != 3:
169:             raise ValueError(
170:                 "replace_stride_with_dilation should be None "
171:                 f"or a 3-element tuple, got {replace_stride_with_dilation}"
172:             )
173:         self.groups = groups
174:         self.base_width = width_per_group
175:         self.conv1 = nn.Conv2d(
176:             3, self.inplanes, kernel_size=7, stride=2, padding=3, bias=False
177:         )
178:         self.bn1 = norm_layer(self.inplanes)
179:         self.relu = nn.ReLU(inplace=True)
180:         self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
181:         self.layer1 = self._make_layer(block, 64, layers[0])
182:         self.layer2 = self._make_layer(
183:             block, 128, layers[1], stride=2, dilate=replace_stride_with_dilation[0]
184:         )
185:         self.layer3 = self._make_layer(
186:             block, 256, layers[2], stride=2, dilate=replace_stride_with_dilation[1]
187:         )
188:         self.layer4 = self._make_layer(
189:             block, 512, layers[3], stride=2, dilate=replace_stride_with_dilation[2]
190:         )
191:         self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
192:         self.fc = nn.Linear(512 * block.expansion, num_classes)
193: 
194:         for m in self.modules():
195:             if isinstance(m, nn.Conv2d):
196:                 nn.init.kaiming_normal_(m.weight, mode="fan_out", nonlinearity="relu")
197:             elif isinstance(m, (nn.BatchNorm2d, nn.GroupNorm)):
198:                 nn.init.constant_(m.weight, 1)
199:                 nn.init.constant_(m.bias, 0)
200: 
201:         # Zero-initialize the last BN in each residual branch,
202:         # so that the residual branch starts with zeros, and each residual block behaves like an identity.
203:         # This improves the model by 0.2~0.3% according to https://arxiv.org/abs/1706.02677
204:         if zero_init_residual:
205:             for m in self.modules():
206:                 if isinstance(m, Bottleneck):
207:                     nn.init.constant_(m.bn3.weight, 0)
208:                 elif isinstance(m, BasicBlock):
209:                     nn.init.constant_(m.bn2.weight, 0)
210: 
211:     def _make_layer(self, block, planes, blocks, stride=1, dilate=False):
212:         norm_layer = self._norm_layer
213:         downsample = None
214:         previous_dilation = self.dilation
215:         if dilate:
216:             self.dilation *= stride
217:             stride = 1
218:         if stride != 1 or self.inplanes != planes * block.expansion:
219:             downsample = nn.Sequential(
220:                 conv1x1(self.inplanes, planes * block.expansion, stride),
221:                 norm_layer(planes * block.expansion),
222:             )
223: 
224:         layers = []
225:         layers.append(
226:             block(
227:                 self.inplanes,
228:                 planes,
229:                 stride,
230:                 downsample,
231:                 self.groups,
232:                 self.base_width,
233:                 previous_dilation,
234:                 norm_layer,
235:             )
236:         )
237:         self.inplanes = planes * block.expansion
238:         for _ in range(1, blocks):
239:             layers.append(
240:                 block(
````
- EN: Implements callable logic such as `_make_layer`.
- CN: 实现可调用逻辑，例如 `_make_layer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-320
````python
241:                     self.inplanes,
242:                     planes,
243:                     groups=self.groups,
244:                     base_width=self.base_width,
245:                     dilation=self.dilation,
246:                     norm_layer=norm_layer,
247:                 )
248:             )
249: 
250:         return nn.Sequential(*layers)
251: 
252:     def _forward_impl(self, x):
253:         # See note [TorchScript super()]
254:         x = self.conv1(x)
255:         x = self.bn1(x)
256:         x = self.relu(x)
257:         x = self.maxpool(x)
258: 
259:         x = self.layer1(x)
260:         x = self.layer2(x)
261:         x = self.layer3(x)
262:         x = self.layer4(x)
263: 
264:         x = self.avgpool(x)
265:         x = torch.flatten(x, 1)
266:         x = self.fc(x)
267: 
268:         return x
269: 
270:     def forward(self, x):
271:         return self._forward_impl(x)
272: 
273: 
274: def _resnet(arch, block, layers, pretrained, progress, **kwargs):
275:     model = ResNet(block, layers, **kwargs)
276:     # if pretrained:
277:     #     state_dict = load_state_dict_from_url(model_urls[arch],
278:     #                                           progress=progress)
279:     #     model.load_state_dict(state_dict)
280:     return model
281: 
282: 
283: def resnet18(pretrained=False, progress=True, **kwargs):
284:     r"""ResNet-18 model from
285:     `"Deep Residual Learning for Image Recognition" <https://arxiv.org/pdf/1512.03385.pdf>`_
286:     Args:
287:         pretrained (bool): If True, returns a model pre-trained on ImageNet
288:         progress (bool): If True, displays a progress bar of the download to stderr
289:     """
290:     return _resnet("resnet18", BasicBlock, [2, 2, 2, 2], pretrained, progress, **kwargs)
291: 
292: 
293: def resnet50(pretrained=False, progress=True, **kwargs):
294:     r"""ResNet-50 model from
295:     `"Deep Residual Learning for Image Recognition" <https://arxiv.org/pdf/1512.03385.pdf>`_
296:     Args:
297:         pretrained (bool): If True, returns a model pre-trained on ImageNet
298:         progress (bool): If True, displays a progress bar of the download to stderr
299:     """
300:     return _resnet("resnet50", Bottleneck, [3, 4, 6, 3], pretrained, progress, **kwargs)
301: 
302: 
303: class IntermediateLayerGetter(nn.ModuleDict):
304:     """
305:     Module wrapper that returns intermediate layers from a model
306:     It has a strong assumption that the modules have been registered
307:     into the model in the same order as they are used.
308:     This means that one should **not** reuse the same nn.Module
309:     twice in the forward if you want this to work.
310:     Additionally, it is only able to query submodules that are directly
311:     assigned to the model. So if `model` is passed, `model.feature1` can
312:     be returned, but not `model.feature1.layer2`.
313:     Args:
314:         model (nn.Module): model on which we will extract the features
315:         return_layers (Dict[name, new_name]): a dict containing the names
316:             of the modules for which the activations will be returned as
317:             the key of the dict, and the value of the dict is the name
318:             of the returned activation (which the user can specify).
319:     Examples::
320:         >>> m = torchvision.models.resnet18(pretrained=True)
````
- EN: Declares or extends types including `IntermediateLayerGetter`.
- CN: 声明或扩展类型，包括 `IntermediateLayerGetter`。
- EN: Implements callable logic such as `_forward_impl`, `forward`, `_resnet`, `resnet18`.
- CN: 实现可调用逻辑，例如 `_forward_impl`, `forward`, `_resnet`, `resnet18`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 321-400
````python
321:         >>> # extract layer1 and layer3, giving as names `feat1` and feat2`
322:         >>> new_m = torchvision.models._utils.IntermediateLayerGetter(m,
323:         >>>     {'layer1': 'feat1', 'layer3': 'feat2'})
324:         >>> out = new_m(torch.rand(1, 3, 224, 224))
325:         >>> print([(k, v.shape) for k, v in out.items()])
326:         >>>     [('feat1', torch.Size([1, 64, 56, 56])),
327:         >>>      ('feat2', torch.Size([1, 256, 14, 14]))]
328:     """
329: 
330:     _version = 2
331:     __annotations__ = {
332:         "return_layers": Dict[str, str],
333:     }
334: 
335:     def __init__(self, model, return_layers):
336:         if not set(return_layers).issubset(
337:             [name for name, _ in model.named_children()]
338:         ):
339:             raise ValueError("return_layers are not present in model")
340:         orig_return_layers = return_layers
341:         return_layers = {str(k): str(v) for k, v in return_layers.items()}
342:         layers = OrderedDict()
343:         for name, module in model.named_children():
344:             layers[name] = module
345:             if name in return_layers:
346:                 del return_layers[name]
347:             if not return_layers:
348:                 break
349: 
350:         super().__init__(layers)
351:         self.return_layers = orig_return_layers
352: 
353:     def forward(self, x):
354:         out = OrderedDict()
355:         for name, module in self.items():
356:             x = module(x)
357:             if name in self.return_layers:
358:                 out_name = self.return_layers[name]
359:                 out[out_name] = x
360:         return out
361: 
362: 
363: class _SimpleSegmentationModel(nn.Module):
364:     __constants__ = ["aux_classifier"]
365: 
366:     def __init__(self, backbone, classifier, aux_classifier=None):
367:         super().__init__()
368:         self.backbone = backbone
369:         self.classifier = classifier
370:         self.aux_classifier = aux_classifier
371: 
372:     def forward(self, x):
373:         input_shape = x.shape[-2:]
374:         # contract: features is a dict of tensors
375:         features = self.backbone(x)
376: 
377:         result = OrderedDict()
378:         x = features["out"]
379:         x = self.classifier(x)
380:         x = F.interpolate(x, size=input_shape, mode="bilinear", align_corners=False)
381:         result["out"] = x
382: 
383:         if self.aux_classifier is not None:
384:             x = features["aux"]
385:             x = self.aux_classifier(x)
386:             x = F.interpolate(x, size=input_shape, mode="bilinear", align_corners=False)
387:             result["aux"] = x
388: 
389:         return result
390: 
391: 
392: class FCN(_SimpleSegmentationModel):
393:     """
394:     Implements a Fully-Convolutional Network for semantic segmentation.
395:     Args:
396:         backbone (nn.Module): the network used to compute the features for the model.
397:             The backbone should return an OrderedDict[Tensor], with the key being
398:             "out" for the last feature map used, and "aux" if an auxiliary classifier
399:             is used.
400:         classifier (nn.Module): module that takes the "out" element returned from
````
- EN: Declares or extends types including `_SimpleSegmentationModel`, `FCN`.
- CN: 声明或扩展类型，包括 `_SimpleSegmentationModel`, `FCN`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 401-480
````python
401:             the backbone and returns a dense prediction.
402:         aux_classifier (nn.Module, optional): auxiliary classifier used during training
403:     """
404: 
405: 
406: class FCNHead(nn.Sequential):
407:     def __init__(self, in_channels, channels):
408:         inter_channels = in_channels // 4
409:         layers = [
410:             nn.Conv2d(in_channels, inter_channels, 3, padding=1, bias=False),
411:             nn.BatchNorm2d(inter_channels),
412:             nn.ReLU(),
413:             nn.Dropout(0.1),
414:             nn.Conv2d(inter_channels, channels, 1),
415:         ]
416: 
417:         super().__init__(*layers)
418: 
419: 
420: def _segm_resnet(name, backbone_name, num_classes, aux, pretrained_backbone=True):
421:     # backbone = resnet.__dict__[backbone_name](
422:     #     pretrained=pretrained_backbone,
423:     #     replace_stride_with_dilation=[False, True, True])
424:     # Hardcoded resnet 50
425:     if backbone_name != "resnet50":
426:         raise AssertionError(
427:             f"Expected backbone_name='resnet50', but got '{backbone_name}'"
428:         )
429:     backbone = resnet50(
430:         pretrained=pretrained_backbone, replace_stride_with_dilation=[False, True, True]
431:     )
432: 
433:     return_layers = {"layer4": "out"}
434:     if aux:
435:         return_layers["layer3"] = "aux"
436:     backbone = IntermediateLayerGetter(backbone, return_layers=return_layers)
437: 
438:     aux_classifier = None
439:     if aux:
440:         inplanes = 1024
441:         aux_classifier = FCNHead(inplanes, num_classes)
442: 
443:     model_map = {
444:         # 'deeplabv3': (DeepLabHead, DeepLabV3), # Not used
445:         "fcn": (FCNHead, FCN),
446:     }
447:     inplanes = 2048
448:     classifier = model_map[name][0](inplanes, num_classes)
449:     base_model = model_map[name][1]
450: 
451:     model = base_model(backbone, classifier, aux_classifier)
452:     return model
453: 
454: 
455: def _load_model(
456:     arch_type, backbone, pretrained, progress, num_classes, aux_loss, **kwargs
457: ):
458:     if pretrained:
459:         aux_loss = True
460:     model = _segm_resnet(arch_type, backbone, num_classes, aux_loss, **kwargs)
461:     # if pretrained:
462:     #     arch = arch_type + '_' + backbone + '_coco'
463:     #     model_url = model_urls[arch]
464:     #     if model_url is None:
465:     #         raise NotImplementedError('pretrained {} is not supported as of now'.format(arch))
466:     #     else:
467:     #         state_dict = load_state_dict_from_url(model_url, progress=progress)
468:     #         model.load_state_dict(state_dict)
469:     return model
470: 
471: 
472: def fcn_resnet50(
473:     pretrained=False, progress=True, num_classes=21, aux_loss=None, **kwargs
474: ):
475:     """Constructs a Fully-Convolutional Network model with a ResNet-50 backbone.
476:     Args:
477:         pretrained (bool): If True, returns a model pre-trained on COCO train2017 which
478:             contains the same classes as Pascal VOC
479:         progress (bool): If True, displays a progress bar of the download to stderr
480:     """
````
- EN: Declares or extends types including `FCNHead`.
- CN: 声明或扩展类型，包括 `FCNHead`。
- EN: Implements callable logic such as `__init__`, `_segm_resnet`, `_load_model`, `fcn_resnet50`.
- CN: 实现可调用逻辑，例如 `__init__`, `_segm_resnet`, `_load_model`, `fcn_resnet50`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 481-560
````python
481:     return _load_model(
482:         "fcn", "resnet50", pretrained, progress, num_classes, aux_loss, **kwargs
483:     )
484: 
485: 
486: # Taken from @fmassa example slides and https://github.com/facebookresearch/detr
487: class DETR(nn.Module):
488:     """
489:     Demo DETR implementation.
490: 
491:     Demo implementation of DETR in minimal number of lines, with the
492:     following differences wrt DETR in the paper:
493:     * learned positional encoding (instead of sine)
494:     * positional encoding is passed at input (instead of attention)
495:     * fc bbox predictor (instead of MLP)
496:     The model achieves ~40 AP on COCO val5k and runs at ~28 FPS on Tesla V100.
497:     Only batch size 1 supported.
498:     """
499: 
500:     def __init__(
501:         self,
502:         num_classes,
503:         hidden_dim=256,
504:         nheads=8,
505:         num_encoder_layers=6,
506:         num_decoder_layers=6,
507:     ):
508:         super().__init__()
509: 
510:         # create ResNet-50 backbone
511:         self.backbone = resnet50()
512:         del self.backbone.fc
513: 
514:         # create conversion layer
515:         self.conv = nn.Conv2d(2048, hidden_dim, 1)
516: 
517:         # create a default PyTorch transformer
518:         self.transformer = nn.Transformer(
519:             hidden_dim, nheads, num_encoder_layers, num_decoder_layers
520:         )
521: 
522:         # prediction heads, one extra class for predicting non-empty slots
523:         # note that in baseline DETR linear_bbox layer is 3-layer MLP
524:         self.linear_class = nn.Linear(hidden_dim, num_classes + 1)
525:         self.linear_bbox = nn.Linear(hidden_dim, 4)
526: 
527:         # output positional encodings (object queries)
528:         self.query_pos = nn.Parameter(torch.rand(100, hidden_dim))
529: 
530:         # spatial positional encodings
531:         # note that in baseline DETR we use sine positional encodings
532:         self.row_embed = nn.Parameter(torch.rand(50, hidden_dim // 2))
533:         self.col_embed = nn.Parameter(torch.rand(50, hidden_dim // 2))
534: 
535:     def forward(self, inputs):
536:         # propagate inputs through ResNet-50 up to avg-pool layer
537:         x = self.backbone.conv1(inputs)
538:         x = self.backbone.bn1(x)
539:         x = self.backbone.relu(x)
540:         x = self.backbone.maxpool(x)
541: 
542:         x = self.backbone.layer1(x)
543:         x = self.backbone.layer2(x)
544:         x = self.backbone.layer3(x)
545:         x = self.backbone.layer4(x)
546: 
547:         # convert from 2048 to 256 feature planes for the transformer
548:         h = self.conv(x)
549: 
550:         # construct positional encodings
551:         H, W = h.shape[-2:]
552:         pos = (
553:             torch.cat(
554:                 [
555:                     self.col_embed[:W].unsqueeze(0).repeat(H, 1, 1),
556:                     self.row_embed[:H].unsqueeze(1).repeat(1, W, 1),
557:                 ],
558:                 dim=-1,
559:             )
560:             .flatten(0, 1)
````
- EN: Declares or extends types including `DETR`, `for`.
- CN: 声明或扩展类型，包括 `DETR`, `for`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 561-640
````python
561:             .unsqueeze(1)
562:         )
563: 
564:         # propagate through the transformer
565:         # TODO (alband) Why this is not automatically broadcasted? (had to add the repeat)
566:         f = pos + 0.1 * h.flatten(2).permute(2, 0, 1)
567:         s = self.query_pos.unsqueeze(1)
568:         s = s.expand(s.size(0), inputs.size(0), s.size(2))
569:         h = self.transformer(f, s).transpose(0, 1)
570: 
571:         # finally project transformer outputs to class labels and bounding boxes
572:         return {
573:             "pred_logits": self.linear_class(h),
574:             "pred_boxes": self.linear_bbox(h).sigmoid(),
575:         }
576: 
577: 
578: def generalized_box_iou(boxes1, boxes2):
579:     """
580:     Generalized IoU from https://giou.stanford.edu/
581:     The boxes should be in [x0, y0, x1, y1] format
582:     Returns a [N, M] pairwise matrix, where N = len(boxes1)
583:     and M = len(boxes2)
584:     """
585:     # degenerate boxes gives inf / nan results
586:     # so do an early check
587:     if not (boxes1[:, 2:] >= boxes1[:, :2]).all():
588:         raise AssertionError("boxes1 has invalid box coordinates (x1 < x0 or y1 < y0)")
589:     if not (boxes2[:, 2:] >= boxes2[:, :2]).all():
590:         raise AssertionError("boxes2 has invalid box coordinates (x1 < x0 or y1 < y0)")
591:     iou, union = box_iou(boxes1, boxes2)
592: 
593:     lt = torch.min(boxes1[:, None, :2], boxes2[:, :2])
594:     rb = torch.max(boxes1[:, None, 2:], boxes2[:, 2:])
595: 
596:     wh = (rb - lt).clamp(min=0)  # [N,M,2]
597:     area = wh[:, :, 0] * wh[:, :, 1]
598: 
599:     return iou - (area - union) / area
600: 
601: 
602: def box_cxcywh_to_xyxy(x):
603:     x_c, y_c, w, h = x.unbind(-1)
604:     b = [(x_c - 0.5 * w), (y_c - 0.5 * h), (x_c + 0.5 * w), (y_c + 0.5 * h)]
605:     return torch.stack(b, dim=-1)
606: 
607: 
608: def box_area(boxes):
609:     """
610:     Computes the area of a set of bounding boxes, which are specified by its
611:     (x1, y1, x2, y2) coordinates.
612:     Args:
613:         boxes (Tensor[N, 4]): boxes for which the area will be computed. They
614:             are expected to be in (x1, y1, x2, y2) format
615:     Returns:
616:         area (Tensor[N]): area for each box
617:     """
618:     return (boxes[:, 2] - boxes[:, 0]) * (boxes[:, 3] - boxes[:, 1])
619: 
620: 
621: # modified from torchvision to also return the union
622: def box_iou(boxes1, boxes2):
623:     area1 = box_area(boxes1)
624:     area2 = box_area(boxes2)
625: 
626:     lt = torch.max(boxes1[:, None, :2], boxes2[:, :2])  # [N,M,2]
627:     rb = torch.min(boxes1[:, None, 2:], boxes2[:, 2:])  # [N,M,2]
628: 
629:     wh = (rb - lt).clamp(min=0)  # [N,M,2]
630:     inter = wh[:, :, 0] * wh[:, :, 1]  # [N,M]
631: 
632:     union = area1[:, None] + area2 - inter
633: 
634:     iou = inter / union
635:     return iou, union
636: 
637: 
638: def is_dist_avail_and_initialized():
639:     return False
640: 
````
- EN: Declares or extends types including `labels`.
- CN: 声明或扩展类型，包括 `labels`。
- EN: Implements callable logic such as `generalized_box_iou`, `box_cxcywh_to_xyxy`, `box_area`, `box_iou`.
- CN: 实现可调用逻辑，例如 `generalized_box_iou`, `box_cxcywh_to_xyxy`, `box_area`, `box_iou`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 641-720
````python
641: 
642: def get_world_size():
643:     if not is_dist_avail_and_initialized():
644:         return 1
645: 
646: 
647: @torch.no_grad()
648: def accuracy(output, target, topk=(1,)):
649:     """Computes the precision@k for the specified values of k"""
650:     if target.numel() == 0:
651:         return [torch.zeros([], device=output.device)]
652:     maxk = max(topk)
653:     batch_size = target.size(0)
654: 
655:     _, pred = output.topk(maxk, 1, True, True)
656:     pred = pred.t()
657:     correct = pred.eq(target.view(1, -1).expand_as(pred))
658: 
659:     res = []
660:     for k in topk:
661:         correct_k = correct[:k].view(-1).float().sum(0)
662:         res.append(correct_k.mul_(100.0 / batch_size))
663:     return res
664: 
665: 
666: class SetCriterion(nn.Module):
667:     """This class computes the loss for DETR.
668:     The process happens in two steps:
669:         1) we compute hungarian assignment between ground truth boxes and the outputs of the model
670:         2) we supervise each pair of matched ground-truth / prediction (supervise class and box)
671:     """
672: 
673:     def __init__(self, num_classes, matcher, weight_dict, eos_coef, losses):
674:         """Create the criterion.
675:         Parameters:
676:             num_classes: number of object categories, omitting the special no-object category
677:             matcher: module able to compute a matching between targets and proposals
678:             weight_dict: dict containing as key the names of the losses and as values their relative weight.
679:             eos_coef: relative classification weight applied to the no-object category
680:             losses: list of all the losses to be applied. See get_loss for list of available losses.
681:         """
682:         super().__init__()
683:         self.num_classes = num_classes
684:         self.matcher = matcher
685:         self.weight_dict = weight_dict
686:         self.eos_coef = eos_coef
687:         self.losses = losses
688:         empty_weight = torch.ones(self.num_classes + 1)
689:         empty_weight[-1] = self.eos_coef
690:         self.register_buffer("empty_weight", empty_weight)
691: 
692:     def loss_labels(self, outputs, targets, indices, num_boxes, log=True):
693:         """Classification loss (NLL)
694:         targets dicts must contain the key "labels" containing a tensor of dim [nb_target_boxes]
695:         """
696:         if "pred_logits" not in outputs:
697:             raise AssertionError("outputs must contain 'pred_logits' key")
698:         src_logits = outputs["pred_logits"]
699: 
700:         idx = self._get_src_permutation_idx(indices)
701:         target_classes_o = torch.cat(
702:             [t["labels"][J] for t, (_, J) in zip(targets, indices)]
703:         )
704:         target_classes = torch.full(
705:             src_logits.shape[:2],
706:             self.num_classes,
707:             dtype=torch.int64,
708:             device=src_logits.device,
709:         )
710:         target_classes[idx] = target_classes_o
711: 
712:         loss_ce = F.cross_entropy(
713:             src_logits.transpose(1, 2), target_classes, self.empty_weight
714:         )
715:         losses = {"loss_ce": loss_ce}
716: 
717:         if log:
718:             # TODO this should probably be a separate loss, not hacked in this one here
719:             losses["class_error"] = 100 - accuracy(src_logits[idx], target_classes_o)[0]
720:         return losses
````
- EN: Declares or extends types including `SetCriterion`, `computes`, `and`.
- CN: 声明或扩展类型，包括 `SetCriterion`, `computes`, `and`。
- EN: Implements callable logic such as `get_world_size`, `accuracy`, `__init__`, `loss_labels`.
- CN: 实现可调用逻辑，例如 `get_world_size`, `accuracy`, `__init__`, `loss_labels`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 721-800
````python
721: 
722:     @torch.no_grad()
723:     def loss_cardinality(self, outputs, targets, indices, num_boxes):
724:         """Compute the cardinality error, ie the absolute error in the number of predicted non-empty boxes
725:         This is not really a loss, it is intended for logging purposes only. It doesn't propagate gradients
726:         """
727:         pred_logits = outputs["pred_logits"]
728:         device = pred_logits.device
729:         tgt_lengths = torch.as_tensor(
730:             [len(v["labels"]) for v in targets], device=device
731:         )
732:         # Count the number of predictions that are NOT "no-object" (which is the last class)
733:         card_pred = (pred_logits.argmax(-1) != pred_logits.shape[-1] - 1).sum(1)
734:         card_err = F.l1_loss(card_pred.float(), tgt_lengths.float())
735:         losses = {"cardinality_error": card_err}
736:         return losses
737: 
738:     def loss_boxes(self, outputs, targets, indices, num_boxes):
739:         """Compute the losses related to the bounding boxes, the L1 regression loss and the GIoU loss
740:         targets dicts must contain the key "boxes" containing a tensor of dim [nb_target_boxes, 4]
741:         The target boxes are expected in format (center_x, center_y, h, w), normalized by the image size.
742:         """
743:         if "pred_boxes" not in outputs:
744:             raise AssertionError("outputs must contain 'pred_boxes' key")
745:         idx = self._get_src_permutation_idx(indices)
746:         src_boxes = outputs["pred_boxes"][idx]
747:         target_boxes = torch.cat(
748:             [t["boxes"][i] for t, (_, i) in zip(targets, indices)], dim=0
749:         )
750: 
751:         loss_bbox = F.l1_loss(src_boxes, target_boxes, reduction="none")
752: 
753:         losses = {}
754:         losses["loss_bbox"] = loss_bbox.sum() / num_boxes
755: 
756:         loss_giou = 1 - torch.diag(
757:             generalized_box_iou(
758:                 box_cxcywh_to_xyxy(src_boxes), box_cxcywh_to_xyxy(target_boxes)
759:             )
760:         )
761:         losses["loss_giou"] = loss_giou.sum() / num_boxes
762:         return losses
763: 
764:     def loss_masks(self, outputs, targets, indices, num_boxes):
765:         """Compute the losses related to the masks: the focal loss and the dice loss.
766:         targets dicts must contain the key "masks" containing a tensor of dim [nb_target_boxes, h, w]
767:         """
768:         if "pred_masks" not in outputs:
769:             raise AssertionError("outputs must contain 'pred_masks' key")
770: 
771:         src_idx = self._get_src_permutation_idx(indices)
772:         tgt_idx = self._get_tgt_permutation_idx(indices)
773: 
774:         src_masks = outputs["pred_masks"]
775: 
776:         # TODO use valid to mask invalid areas due to padding in loss
777:         target_masks, valid = nested_tensor_from_tensor_list(  # noqa: F821
778:             [t["masks"] for t in targets]
779:         ).decompose()
780:         target_masks = target_masks.to(src_masks)
781: 
782:         src_masks = src_masks[src_idx]
783:         # upsample predictions to the target size
784:         src_masks = interpolate(  # noqa: F821
785:             src_masks[:, None],
786:             size=target_masks.shape[-2:],
787:             mode="bilinear",
788:             align_corners=False,
789:         )
790:         src_masks = src_masks[:, 0].flatten(1)
791: 
792:         target_masks = target_masks[tgt_idx].flatten(1)
793: 
794:         losses = {
795:             "loss_mask": sigmoid_focal_loss(  # noqa: F821
796:                 src_masks, target_masks, num_boxes
797:             ),
798:             "loss_dice": dice_loss(src_masks, target_masks, num_boxes),  # noqa: F821
799:         }
800:         return losses
````
- EN: Implements callable logic such as `loss_cardinality`, `loss_boxes`, `loss_masks`.
- CN: 实现可调用逻辑，例如 `loss_cardinality`, `loss_boxes`, `loss_masks`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 801-880
````python
801: 
802:     def _get_src_permutation_idx(self, indices):
803:         # permute predictions following indices
804:         batch_idx = torch.cat(
805:             [torch.full_like(src, i) for i, (src, _) in enumerate(indices)]
806:         )
807:         src_idx = torch.cat([src for (src, _) in indices])
808:         return batch_idx, src_idx
809: 
810:     def _get_tgt_permutation_idx(self, indices):
811:         # permute targets following indices
812:         batch_idx = torch.cat(
813:             [torch.full_like(tgt, i) for i, (_, tgt) in enumerate(indices)]
814:         )
815:         tgt_idx = torch.cat([tgt for (_, tgt) in indices])
816:         return batch_idx, tgt_idx
817: 
818:     def get_loss(self, loss, outputs, targets, indices, num_boxes, **kwargs):
819:         loss_map = {
820:             "labels": self.loss_labels,
821:             "cardinality": self.loss_cardinality,
822:             "boxes": self.loss_boxes,
823:             "masks": self.loss_masks,
824:         }
825:         if loss not in loss_map:
826:             raise AssertionError(
827:                 f"Unknown loss type '{loss}'. Available: {list(loss_map.keys())}"
828:             )
829:         return loss_map[loss](outputs, targets, indices, num_boxes, **kwargs)
830: 
831:     def forward(self, outputs, targets):
832:         """This performs the loss computation.
833:         Parameters:
834:              outputs: dict of tensors, see the output specification of the model for the format
835:              targets: list of dicts, such that len(targets) == batch_size.
836:                       The expected keys in each dict depends on the losses applied, see each loss' doc
837:         """
838:         outputs_without_aux = {k: v for k, v in outputs.items() if k != "aux_outputs"}
839: 
840:         # Retrieve the matching between the outputs of the last layer and the targets
841:         indices = self.matcher(outputs_without_aux, targets)
842: 
843:         # Compute the average number of target boxes across all nodes, for normalization purposes
844:         num_boxes = sum(len(t["labels"]) for t in targets)
845:         num_boxes = torch.as_tensor(
846:             [num_boxes], dtype=torch.float, device=next(iter(outputs.values())).device
847:         )
848:         if is_dist_avail_and_initialized():
849:             torch.distributed.all_reduce(num_boxes)
850:         num_boxes = torch.clamp(num_boxes / get_world_size(), min=1).item()
851: 
852:         # Compute all the requested losses
853:         losses = {}
854:         for loss in self.losses:
855:             losses.update(self.get_loss(loss, outputs, targets, indices, num_boxes))
856: 
857:         # In case of auxiliary losses, we repeat this process with the output of each intermediate layer.
858:         if "aux_outputs" in outputs:
859:             for i, aux_outputs in enumerate(outputs["aux_outputs"]):
860:                 indices = self.matcher(aux_outputs, targets)
861:                 for loss in self.losses:
862:                     if loss == "masks":
863:                         # Intermediate masks losses are too costly to compute, we ignore them.
864:                         continue
865:                     kwargs = {}
866:                     if loss == "labels":
867:                         # Logging is enabled only for the last layer
868:                         kwargs = {"log": False}
869:                     l_dict = self.get_loss(
870:                         loss, aux_outputs, targets, indices, num_boxes, **kwargs
871:                     )
872:                     l_dict = {k + f"_{i}": v for k, v in l_dict.items()}
873:                     losses.update(l_dict)
874: 
875:         return losses
876: 
877: 
878: class HungarianMatcher(nn.Module):
879:     """This class computes an assignment between the targets and the predictions of the network
880:     For efficiency reasons, the targets don't include the no_object. Because of this, in general,
````
- EN: Declares or extends types including `HungarianMatcher`, `computes`.
- CN: 声明或扩展类型，包括 `HungarianMatcher`, `computes`。
- EN: Implements callable logic such as `_get_src_permutation_idx`, `_get_tgt_permutation_idx`, `get_loss`, `forward`.
- CN: 实现可调用逻辑，例如 `_get_src_permutation_idx`, `_get_tgt_permutation_idx`, `get_loss`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 881-960
````python
881:     there are more predictions than targets. In this case, we do a 1-to-1 matching of the best predictions,
882:     while the others are un-matched (and thus treated as non-objects).
883:     """
884: 
885:     def __init__(
886:         self, cost_class: float = 1, cost_bbox: float = 1, cost_giou: float = 1
887:     ):
888:         """Creates the matcher
889:         Params:
890:             cost_class: This is the relative weight of the classification error in the matching cost
891:             cost_bbox: This is the relative weight of the L1 error of the bounding box coordinates in the matching cost
892:             cost_giou: This is the relative weight of the giou loss of the bounding box in the matching cost
893:         """
894:         super().__init__()
895:         self.cost_class = cost_class
896:         self.cost_bbox = cost_bbox
897:         self.cost_giou = cost_giou
898:         if cost_class == 0 and cost_bbox == 0 and cost_giou == 0:
899:             raise AssertionError(
900:                 "At least one of cost_class, cost_bbox, or cost_giou must be non-zero"
901:             )
902: 
903:     @torch.no_grad()
904:     def forward(self, outputs, targets):
905:         """Performs the matching
906:         Params:
907:             outputs: This is a dict that contains at least these entries:
908:                  "pred_logits": Tensor of dim [batch_size, num_queries, num_classes] with the classification logits
909:                  "pred_boxes": Tensor of dim [batch_size, num_queries, 4] with the predicted box coordinates
910:             targets: This is a list of targets (len(targets) = batch_size), where each target is a dict containing:
911:                  "labels": Tensor of dim [num_target_boxes] (where num_target_boxes is the number of ground-truth
912:                            objects in the target) containing the class labels
913:                  "boxes": Tensor of dim [num_target_boxes, 4] containing the target box coordinates
914:         Returns:
915:             A list of size batch_size, containing tuples of (index_i, index_j) where:
916:                 - index_i is the indices of the selected predictions (in order)
917:                 - index_j is the indices of the corresponding selected targets (in order)
918:             For each batch element, it holds:
919:                 len(index_i) = len(index_j) = min(num_queries, num_target_boxes)
920:         """
921:         bs, num_queries = outputs["pred_logits"].shape[:2]
922: 
923:         # We flatten to compute the cost matrices in a batch
924:         out_prob = (
925:             outputs["pred_logits"].flatten(0, 1).softmax(-1)
926:         )  # [batch_size * num_queries, num_classes]
927:         out_bbox = outputs["pred_boxes"].flatten(0, 1)  # [batch_size * num_queries, 4]
928: 
929:         # Also concat the target labels and boxes
930:         tgt_ids = torch.cat([v["labels"] for v in targets])
931:         tgt_bbox = torch.cat([v["boxes"] for v in targets])
932: 
933:         # Compute the classification cost. Contrary to the loss, we don't use the NLL,
934:         # but approximate it in 1 - proba[target class].
935:         # The 1 is a constant that doesn't change the matching, it can be omitted.
936:         cost_class = -out_prob[:, tgt_ids]
937: 
938:         # Compute the L1 cost between boxes
939:         cost_bbox = torch.cdist(out_bbox, tgt_bbox, p=1)
940: 
941:         # Compute the giou cost between boxes
942:         cost_giou = -generalized_box_iou(
943:             box_cxcywh_to_xyxy(out_bbox), box_cxcywh_to_xyxy(tgt_bbox)
944:         )
945: 
946:         # Final cost matrix
947:         C = (
948:             self.cost_bbox * cost_bbox
949:             + self.cost_class * cost_class
950:             + self.cost_giou * cost_giou
951:         )
952:         C = C.view(bs, num_queries, -1).cpu()
953: 
954:         sizes = [len(v["boxes"]) for v in targets]
955:         if not scipy_available:
956:             raise RuntimeError(
957:                 "The 'detr' model requires scipy to run. Please make sure you have it installed"
958:                 " if you enable the 'detr' model."
959:             )
960:         indices = [
````
- EN: Declares or extends types including `labels`.
- CN: 声明或扩展类型，包括 `labels`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 961-969
````python
961:             linear_sum_assignment(c[i]) for i, c in enumerate(C.split(sizes, -1))
962:         ]
963:         return [
964:             (
965:                 torch.as_tensor(i, dtype=torch.int64),
966:                 torch.as_tensor(j, dtype=torch.int64),
967:             )
968:             for i, j in indices
969:         ]
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `conv3x3` / 符号 `conv3x3`
- Symbol `conv1x1` / 符号 `conv1x1`
- Symbol `BasicBlock` / 符号 `BasicBlock`
- Symbol `__init__` / 符号 `__init__`

## Dependencies / 依赖关系
- Python imports: `collections`, `torch`, `torch.jit.annotations`, `torch.nn`, `scipy.optimize`
- Python 导入: `collections`, `torch`, `torch.jit.annotations`, `torch.nn`, `scipy.optimize`

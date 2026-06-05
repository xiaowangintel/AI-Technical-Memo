# quantize.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/gpt_fast/quantize.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: # flake8: noqa: E266
 2: import torch
 3: import torch.nn as nn
 4: import torch.nn.functional as F
 5: 
 6: 
 7: ##### Quantization Primitives ######
 8: 
 9: 
10: def dynamically_quantize_per_channel(x, quant_min, quant_max, target_dtype):
11:     # assumes symmetric quantization
12:     # assumes axis == 0
13:     # assumes dense memory format
14:     # TODO(future): relax ^ as needed
15: 
16:     # default setup for affine quantization of activations
17:     eps = torch.finfo(torch.float32).eps
18: 
19:     # get min and max
20:     min_val, max_val = torch.aminmax(x, dim=1)
21: 
22:     # calculate scales and zero_points based on min and max
23:     # reference: https://fburl.com/code/srbiybme
24:     min_val_neg = torch.min(min_val, torch.zeros_like(min_val))
25:     max_val_pos = torch.max(max_val, torch.zeros_like(max_val))
26:     device = min_val_neg.device
27: 
28:     # reference: https://fburl.com/code/4wll53rk
29:     max_val_pos = torch.max(-min_val_neg, max_val_pos)
30:     scales = max_val_pos / (float(quant_max - quant_min) / 2)
31:     # ensure scales is the same dtype as the original tensor
32:     scales = torch.clamp(scales, min=eps).to(x.dtype)
33:     zero_points = torch.zeros(min_val_neg.size(), dtype=torch.int64, device=device)
34: 
35:     # quantize based on qmin/qmax/scales/zp
36:     # reference: https://www.internalfb.com/code/fbsource/[8edc275012b1]/fbcode/caffe2/torch/ao/quantization/fx/_decomposed.py?lines=63
37:     x_div = x / scales.unsqueeze(-1)
38:     x_round = torch.round(x_div)
39:     x_zp = x_round + zero_points.unsqueeze(-1)
40:     quant = torch.clamp(x_zp, quant_min, quant_max).to(target_dtype)
````
- EN: Handles module imports such as `torch`, `torch.nn`, `torch.nn.functional`.
- CN: 处理模块导入，例如 `torch`, `torch.nn`, `torch.nn.functional`。
- EN: Implements callable logic such as `dynamically_quantize_per_channel`.
- CN: 实现可调用逻辑，例如 `dynamically_quantize_per_channel`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41: 
42:     return quant, scales, zero_points
43: 
44: 
45: ##### Weight-only int8 per-channel quantized code ######
46: 
47: 
48: def replace_linear_weight_only_int8_per_channel(module):
49:     for name, child in module.named_children():
50:         if isinstance(child, nn.Linear):
51:             setattr(
52:                 module,
53:                 name,
54:                 WeightOnlyInt8Linear(child.in_features, child.out_features),
55:             )
56:         else:
57:             replace_linear_weight_only_int8_per_channel(child)
58: 
59: 
60: class WeightOnlyInt8QuantHandler:
61:     def __init__(self, mod):
62:         self.mod = mod
63: 
64:     @torch.no_grad()
65:     def create_quantized_state_dict(self):
66:         cur_state_dict = self.mod.state_dict()
67:         for fqn, mod in self.mod.named_modules():
68:             if isinstance(mod, torch.nn.Linear):
69:                 int8_weight, scales, _ = dynamically_quantize_per_channel(
70:                     mod.weight.float(), -128, 127, torch.int8
71:                 )
72:                 cur_state_dict[f"{fqn}.weight"] = int8_weight.to("cpu")
73:                 cur_state_dict[f"{fqn}.scales"] = scales.to(mod.weight.dtype).to("cpu")
74: 
75:         return cur_state_dict
76: 
77:     def convert_for_runtime(self):
78:         replace_linear_weight_only_int8_per_channel(self.mod)
79:         return self.mod
80: 
````
- EN: Declares or extends types including `WeightOnlyInt8QuantHandler`.
- CN: 声明或扩展类型，包括 `WeightOnlyInt8QuantHandler`。
- EN: Implements callable logic such as `replace_linear_weight_only_int8_per_channel`, `__init__`, `create_quantized_state_dict`, `convert_for_runtime`.
- CN: 实现可调用逻辑，例如 `replace_linear_weight_only_int8_per_channel`, `__init__`, `create_quantized_state_dict`, `convert_for_runtime`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-105
````python
 81: 
 82: class WeightOnlyInt8Linear(torch.nn.Module):
 83:     __constants__ = ["in_features", "out_features"]
 84:     in_features: int
 85:     out_features: int
 86:     weight: torch.Tensor
 87: 
 88:     def __init__(
 89:         self,
 90:         in_features: int,
 91:         out_features: int,
 92:         bias: bool = True,
 93:         device=None,
 94:         dtype=None,
 95:     ) -> None:
 96:         super().__init__()
 97:         self.in_features = in_features
 98:         self.out_features = out_features
 99:         self.register_buffer(
100:             "weight", torch.empty((out_features, in_features), dtype=torch.int8)
101:         )
102:         self.register_buffer("scales", torch.ones(out_features, dtype=torch.bfloat16))
103: 
104:     def forward(self, input: torch.Tensor) -> torch.Tensor:
105:         return F.linear(input, self.weight.to(dtype=input.dtype)) * self.scales
````
- EN: Declares or extends types including `WeightOnlyInt8Linear`.
- CN: 声明或扩展类型，包括 `WeightOnlyInt8Linear`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `dynamically_quantize_per_channel` / 符号 `dynamically_quantize_per_channel`
- Symbol `replace_linear_weight_only_int8_per_channel` / 符号 `replace_linear_weight_only_int8_per_channel`
- Symbol `WeightOnlyInt8QuantHandler` / 符号 `WeightOnlyInt8QuantHandler`
- Symbol `__init__` / 符号 `__init__`

## Dependencies / 依赖关系
- Python imports: `torch`, `torch.nn`, `torch.nn.functional`
- Python 导入: `torch`, `torch.nn`, `torch.nn.functional`

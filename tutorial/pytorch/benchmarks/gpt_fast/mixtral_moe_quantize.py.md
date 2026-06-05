# mixtral_moe_quantize.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/gpt_fast/mixtral_moe_quantize.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: # flake8: noqa: E266
 2: from mixtral_moe_model import ConditionalFeedForward
 3: 
 4: import torch
 5: import torch.nn as nn
 6: import torch.nn.functional as F
 7: 
 8: 
 9: ##### Quantization Primitives ######
10: 
11: 
12: def dynamically_quantize_per_channel(x, quant_min, quant_max, target_dtype):
13:     # assumes symmetric quantization
14:     # assumes axis == 0
15:     # assumes dense memory format
16:     # TODO(future): relax ^ as needed
17: 
18:     # default setup for affine quantization of activations
19:     eps = torch.finfo(torch.float32).eps
20: 
21:     # get min and max
22:     min_val, max_val = torch.aminmax(x, dim=1)
23: 
24:     # calculate scales and zero_points based on min and max
25:     # reference: https://fburl.com/code/srbiybme
26:     min_val_neg = torch.min(min_val, torch.zeros_like(min_val))
27:     max_val_pos = torch.max(max_val, torch.zeros_like(max_val))
28:     device = min_val_neg.device
29: 
30:     # reference: https://fburl.com/code/4wll53rk
31:     max_val_pos = torch.max(-min_val_neg, max_val_pos)
32:     scales = max_val_pos / (float(quant_max - quant_min) / 2)
33:     # ensure scales is the same dtype as the original tensor
34:     scales = torch.clamp(scales, min=eps).to(x.dtype)
35:     zero_points = torch.zeros(min_val_neg.size(), dtype=torch.int64, device=device)
36: 
37:     # quantize based on qmin/qmax/scales/zp
38:     # reference: https://www.internalfb.com/code/fbsource/[8edc275012b1]/fbcode/caffe2/torch/ao/quantization/fx/_decomposed.py?lines=63
39:     x_div = x / scales.unsqueeze(-1)
40:     x_round = torch.round(x_div)
````
- EN: Handles module imports such as `mixtral_moe_model`, `torch`, `torch.nn`, `torch.nn.functional`.
- CN: 处理模块导入，例如 `mixtral_moe_model`, `torch`, `torch.nn`, `torch.nn.functional`。
- EN: Implements callable logic such as `dynamically_quantize_per_channel`.
- CN: 实现可调用逻辑，例如 `dynamically_quantize_per_channel`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:     x_zp = x_round + zero_points.unsqueeze(-1)
42:     quant = torch.clamp(x_zp, quant_min, quant_max).to(target_dtype)
43: 
44:     return quant, scales, zero_points
45: 
46: 
47: ##### Weight-only int8 per-channel quantized code ######
48: 
49: 
50: def replace_linear_weight_only_int8_per_channel(module):
51:     for name, child in module.named_children():
52:         if isinstance(child, nn.Linear) and name != "gate":
53:             setattr(
54:                 module,
55:                 name,
56:                 WeightOnlyInt8Linear(
57:                     child.in_features, child.out_features, target_dtype=torch.int8
58:                 ),
59:             )
60:         elif isinstance(child, ConditionalFeedForward):
61:             num_experts, intermediate_size, dim = child.w1.shape
62:             setattr(
63:                 module,
64:                 name,
65:                 ConditionalFeedForwardInt8(
66:                     num_experts, intermediate_size, dim, target_dtype=torch.int8
67:                 ),
68:             )
69:         else:
70:             replace_linear_weight_only_int8_per_channel(child)
71: 
72: 
73: class WeightOnlyInt8QuantHandler:
74:     def __init__(self, mod):
75:         self.mod = mod
76: 
77:     @torch.no_grad()
78:     def create_quantized_state_dict(self):
79:         cur_state_dict = self.mod.state_dict()
80:         for fqn, mod in self.mod.named_modules():
````
- EN: Declares or extends types including `WeightOnlyInt8QuantHandler`.
- CN: 声明或扩展类型，包括 `WeightOnlyInt8QuantHandler`。
- EN: Implements callable logic such as `replace_linear_weight_only_int8_per_channel`, `__init__`, `create_quantized_state_dict`.
- CN: 实现可调用逻辑，例如 `replace_linear_weight_only_int8_per_channel`, `__init__`, `create_quantized_state_dict`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:             if isinstance(mod, torch.nn.Linear) and not fqn.endswith(".gate"):
 82:                 int8_weight, scales, _ = dynamically_quantize_per_channel(
 83:                     mod.weight.float(), -128, 127, torch.int8
 84:                 )
 85:                 cur_state_dict[f"{fqn}.weight"] = int8_weight
 86:                 cur_state_dict[f"{fqn}.scales"] = scales.to(mod.weight.dtype)
 87:             elif isinstance(mod, ConditionalFeedForward):
 88:                 for weight_idx in range(3):
 89:                     weight_name = f"w{weight_idx + 1}"
 90:                     scales_name = f"scales{weight_idx + 1}"
 91:                     weight = getattr(mod, weight_name)
 92:                     num_experts, intermediate_size, dim = weight.shape
 93: 
 94:                     bit8_weight_list = []
 95:                     scales_list = []
 96:                     for expert_idx in range(num_experts):
 97:                         bit8_weight, scales, _ = dynamically_quantize_per_channel(
 98:                             weight[expert_idx].float(), -128, 127, torch.int8
 99:                         )
100:                         bit8_weight_list.append(
101:                             bit8_weight.reshape(1, intermediate_size, dim)
102:                         )
103:                         scales_list.append(scales.reshape(1, intermediate_size))
104: 
105:                     cur_state_dict[f"{fqn}.{weight_name}"] = torch.cat(
106:                         bit8_weight_list, dim=0
107:                     )
108:                     cur_state_dict[f"{fqn}.{scales_name}"] = torch.cat(
109:                         scales_list, dim=0
110:                     )
111: 
112:         return cur_state_dict
113: 
114:     def convert_for_runtime(self):
115:         replace_linear_weight_only_int8_per_channel(self.mod)
116:         return self.mod
117: 
118: 
119: class WeightOnlyInt8Linear(torch.nn.Module):
120:     __constants__ = ["in_features", "out_features"]
````
- EN: Declares or extends types including `WeightOnlyInt8Linear`.
- CN: 声明或扩展类型，包括 `WeightOnlyInt8Linear`。
- EN: Implements callable logic such as `convert_for_runtime`.
- CN: 实现可调用逻辑，例如 `convert_for_runtime`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````python
121:     in_features: int
122:     out_features: int
123:     weight: torch.Tensor
124: 
125:     def __init__(
126:         self,
127:         in_features: int,
128:         out_features: int,
129:         bias: bool = True,
130:         device=None,
131:         dtype=None,
132:         target_dtype=None,
133:     ) -> None:
134:         if target_dtype is None:
135:             raise AssertionError("target_dtype must not be None")
136:         super().__init__()
137:         self.in_features = in_features
138:         self.out_features = out_features
139:         self.register_buffer(
140:             "weight", torch.empty((out_features, in_features), dtype=target_dtype)
141:         )
142:         self.register_buffer("scales", torch.ones(out_features, dtype=torch.bfloat16))
143: 
144:     def forward(self, input: torch.Tensor) -> torch.Tensor:
145:         return F.linear(input, self.weight.to(dtype=input.dtype)) * self.scales
146: 
147: 
148: class ConditionalFeedForwardInt8(nn.Module):
149:     def __init__(self, num_experts, intermediate_size, dim, target_dtype):
150:         super().__init__()
151: 
152:         self.target_dtype = target_dtype
153: 
154:         self.register_buffer(
155:             "w1", torch.empty(num_experts, intermediate_size, dim, dtype=target_dtype)
156:         )
157:         self.register_buffer(
158:             "w2", torch.empty(num_experts, dim, intermediate_size, dtype=target_dtype)
159:         )
160:         self.register_buffer(
````
- EN: Declares or extends types including `ConditionalFeedForwardInt8`.
- CN: 声明或扩展类型，包括 `ConditionalFeedForwardInt8`。
- EN: Implements callable logic such as `__init__`, `forward`.
- CN: 实现可调用逻辑，例如 `__init__`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-188
````python
161:             "w3", torch.empty(num_experts, intermediate_size, dim, dtype=target_dtype)
162:         )
163: 
164:         self.register_buffer(
165:             "scales1", torch.empty(num_experts, intermediate_size, dtype=torch.bfloat16)
166:         )
167:         self.register_buffer(
168:             "scales2", torch.empty(num_experts, dim, dtype=torch.bfloat16)
169:         )
170:         self.register_buffer(
171:             "scales3", torch.empty(num_experts, intermediate_size, dtype=torch.bfloat16)
172:         )
173: 
174:     def forward(self, x, expert_indices):
175:         w1_weights = self.w1.to(x.dtype)[expert_indices]  # [T, A, D, D]
176:         w3_weights = self.w3.to(x.dtype)[expert_indices]  # [T, A, D, D]
177:         w2_weights = self.w2.to(x.dtype)[expert_indices]
178:         x1 = F.silu(
179:             torch.einsum("ti,taoi -> tao", x, w1_weights)
180:             * self.scales1[expert_indices].to(x.dtype)
181:         )
182:         x3 = torch.einsum("ti, taoi -> tao", x, w3_weights) * self.scales3[
183:             expert_indices
184:         ].to(x.dtype)
185:         expert_outs = torch.einsum(
186:             "tao, taio -> tai", (x1 * x3), w2_weights
187:         ) * self.scales2[expert_indices].to(x.dtype)  # [T, A, D, D]
188:         return expert_outs
````
- EN: Implements callable logic such as `forward`.
- CN: 实现可调用逻辑，例如 `forward`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `dynamically_quantize_per_channel` / 符号 `dynamically_quantize_per_channel`
- Symbol `replace_linear_weight_only_int8_per_channel` / 符号 `replace_linear_weight_only_int8_per_channel`
- Symbol `WeightOnlyInt8QuantHandler` / 符号 `WeightOnlyInt8QuantHandler`
- Symbol `__init__` / 符号 `__init__`

## Dependencies / 依赖关系
- Python imports: `mixtral_moe_model`, `torch`, `torch.nn`, `torch.nn.functional`
- Python 导入: `mixtral_moe_model`, `torch`, `torch.nn`, `torch.nn.functional`

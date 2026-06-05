# test.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/fastrnns/test.py`
- Repository: `pytorch`
- Purpose (EN): Defines benchmark-oriented test cases and validation scenarios.
- 用途 (CN): 定义面向基准测试的测试用例和校验场景。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: 
 3: import torch
 4: import torch.nn as nn
 5: 
 6: from .factory import pytorch_lstm_creator, varlen_pytorch_lstm_creator
 7: from .runner import get_nn_runners
 8: 
 9: 
10: def barf():
11:     import pdb
12: 
13:     pdb.set_trace()
14: 
15: 
16: def assertEqual(tensor, expected, threshold=0.001):
17:     if isinstance(tensor, (list, tuple)):
18:         for t, e in zip(tensor, expected):
19:             assertEqual(t, e)
20:     else:
21:         if (tensor - expected).abs().max() > threshold:
22:             barf()
23: 
24: 
25: def filter_requires_grad(tensors):
26:     return [t for t in tensors if t.requires_grad]
27: 
28: 
29: def test_rnns(
30:     experim_creator,
31:     control_creator,
32:     check_grad=True,
33:     verbose=False,
34:     seqLength=100,
35:     numLayers=1,
36:     inputSize=512,
37:     hiddenSize=512,
38:     miniBatch=64,
39:     device="cuda",
40:     seed=17,
````
- EN: Handles module imports such as `argparse`, `torch`, `torch.nn`, `.factory`.
- CN: 处理模块导入，例如 `argparse`, `torch`, `torch.nn`, `.factory`。
- EN: Implements callable logic such as `barf`, `assertEqual`, `filter_requires_grad`, `test_rnns`.
- CN: 实现可调用逻辑，例如 `barf`, `assertEqual`, `filter_requires_grad`, `test_rnns`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41: ):
42:     creator_args = dict(
43:         seqLength=seqLength,
44:         numLayers=numLayers,
45:         inputSize=inputSize,
46:         hiddenSize=hiddenSize,
47:         miniBatch=miniBatch,
48:         device=device,
49:         seed=seed,
50:     )
51: 
52:     print("Setting up...")
53:     control = control_creator(**creator_args)
54:     experiment = experim_creator(**creator_args)
55: 
56:     # Precondition
57:     assertEqual(experiment.inputs, control.inputs)
58:     assertEqual(experiment.params, control.params)
59: 
60:     print("Checking outputs...")
61:     control_outputs = control.forward(*control.inputs)
62:     experim_outputs = experiment.forward(*experiment.inputs)
63:     assertEqual(experim_outputs, control_outputs)
64: 
65:     print("Checking grads...")
66:     if control.backward_setup is None:
67:         raise AssertionError("control.backward_setup must not be None")
68:     if experiment.backward_setup is None:
69:         raise AssertionError("experiment.backward_setup must not be None")
70:     if control.backward is None:
71:         raise AssertionError("control.backward must not be None")
72:     if experiment.backward is None:
73:         raise AssertionError("experiment.backward must not be None")
74:     control_backward_inputs = control.backward_setup(control_outputs, seed)
75:     experim_backward_inputs = experiment.backward_setup(experim_outputs, seed)
76: 
77:     control.backward(*control_backward_inputs)
78:     experiment.backward(*experim_backward_inputs)
79: 
80:     control_grads = [p.grad for p in control.params]
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````python
 81:     experim_grads = [p.grad for p in experiment.params]
 82:     assertEqual(experim_grads, control_grads)
 83: 
 84:     if verbose:
 85:         print(experiment.forward.graph_for(*experiment.inputs))
 86:     print()
 87: 
 88: 
 89: def test_vl_py(**test_args):
 90:     # XXX: This compares vl_py with vl_lstm.
 91:     # It's done this way because those two don't give the same outputs so
 92:     # the result isn't an apples-to-apples comparison right now.
 93:     control_creator = varlen_pytorch_lstm_creator
 94:     name, experim_creator, context = get_nn_runners("vl_py")[0]
 95:     with context():
 96:         print(f"testing {name}...")
 97:         creator_keys = [
 98:             "seqLength",
 99:             "numLayers",
100:             "inputSize",
101:             "hiddenSize",
102:             "miniBatch",
103:             "device",
104:             "seed",
105:         ]
106:         creator_args = {key: test_args[key] for key in creator_keys}
107: 
108:         print("Setting up...")
109:         control = control_creator(**creator_args)
110:         experiment = experim_creator(**creator_args)
111: 
112:         # Precondition
113:         assertEqual(experiment.inputs, control.inputs[:2])
114:         assertEqual(experiment.params, control.params)
115: 
116:         print("Checking outputs...")
117:         control_out, control_hiddens = control.forward(*control.inputs)
118:         control_hx, control_cx = control_hiddens
119:         experim_out, experim_hiddens = experiment.forward(*experiment.inputs)
120:         experim_hx, experim_cx = experim_hiddens
````
- EN: Implements callable logic such as `test_vl_py`.
- CN: 实现可调用逻辑，例如 `test_vl_py`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````python
121: 
122:         experim_padded = nn.utils.rnn.pad_sequence(experim_out).squeeze(-2)
123:         assertEqual(experim_padded, control_out)
124:         assertEqual(torch.cat(experim_hx, dim=1), control_hx)
125:         assertEqual(torch.cat(experim_cx, dim=1), control_cx)
126: 
127:         print("Checking grads...")
128:         if control.backward_setup is None:
129:             raise AssertionError("control.backward_setup must not be None")
130:         if experiment.backward_setup is None:
131:             raise AssertionError("experiment.backward_setup must not be None")
132:         if control.backward is None:
133:             raise AssertionError("control.backward must not be None")
134:         if experiment.backward is None:
135:             raise AssertionError("experiment.backward must not be None")
136:         control_backward_inputs = control.backward_setup(
137:             (control_out, control_hiddens), test_args["seed"]
138:         )
139:         experim_backward_inputs = experiment.backward_setup(
140:             (experim_out, experim_hiddens), test_args["seed"]
141:         )
142: 
143:         control.backward(*control_backward_inputs)
144:         experiment.backward(*experim_backward_inputs)
145: 
146:         control_grads = [p.grad for p in control.params]
147:         experim_grads = [p.grad for p in experiment.params]
148:         assertEqual(experim_grads, control_grads)
149: 
150:         if test_args["verbose"]:
151:             print(experiment.forward.graph_for(*experiment.inputs))
152:         print()
153: 
154: 
155: if __name__ == "__main__":
156:     parser = argparse.ArgumentParser(description="Test lstm correctness")
157: 
158:     parser.add_argument("--seqLength", default="100", type=int)
159:     parser.add_argument("--numLayers", default="1", type=int)
160:     parser.add_argument("--inputSize", default="512", type=int)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-193
````python
161:     parser.add_argument("--hiddenSize", default="512", type=int)
162:     parser.add_argument("--miniBatch", default="64", type=int)
163:     parser.add_argument("--device", default="cuda", type=str)
164:     parser.add_argument("--check-grad", "--check_grad", default="True", type=bool)
165:     parser.add_argument("--variable-lstms", "--variable_lstms", action="store_true")
166:     parser.add_argument("--seed", default="17", type=int)
167:     parser.add_argument("--verbose", action="store_true")
168:     parser.add_argument("--rnns", nargs="*", help="What to run. jit_premul, jit, etc")
169:     args = parser.parse_args()
170:     if args.rnns is None:
171:         args.rnns = ["jit_premul", "jit"]
172:     print(args)
173: 
174:     if "cuda" in args.device:
175:         if not torch.cuda.is_available():
176:             raise AssertionError(
177:                 f"CUDA device requested ({args.device}) but CUDA is not available"
178:             )
179: 
180:     rnn_runners = get_nn_runners(*args.rnns)
181: 
182:     should_test_varlen_lstms = args.variable_lstms
183:     test_args = vars(args)
184:     del test_args["rnns"]
185:     del test_args["variable_lstms"]
186: 
187:     if should_test_varlen_lstms:
188:         test_vl_py(**test_args)
189: 
190:     for name, creator, context in rnn_runners:
191:         with context():
192:             print(f"testing {name}...")
193:             test_rnns(creator, pytorch_lstm_creator, **test_args)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Validation and test coverage / 校验与测试覆盖
- Symbol `barf` / 符号 `barf`
- Symbol `assertEqual` / 符号 `assertEqual`
- Symbol `filter_requires_grad` / 符号 `filter_requires_grad`
- Symbol `test_rnns` / 符号 `test_rnns`

## Dependencies / 依赖关系
- Python imports: `argparse`, `torch`, `torch.nn`, `.factory`, `.runner`, `pdb`
- Python 导入: `argparse`, `torch`, `torch.nn`, `.factory`, `.runner`, `pdb`

# distributed.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/distributed.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: import logging
 3: import os
 4: from functools import partial
 5: 
 6: import torch
 7: import torch._dynamo as dynamo
 8: import torch.utils._pytree as pytree
 9: from torch._dynamo.testing import reduce_to_scalar_loss
10: from torch.nn.parallel import DistributedDataParallel as DDP
11: from torch.profiler import profile, ProfilerActivity, record_function
12: 
13: 
14: try:
15:     from .common import timed
16:     from .dist_util import apply_fsdp, cleanup, get_model, model_iter_fn, setup
17: except ImportError:
18:     from common import timed
19:     from dist_util import apply_fsdp, cleanup, get_model, model_iter_fn, setup
20: 
21: log = logging.getLogger(__name__)
22: 
23: 
24: def torchviz_model(args, model, inputs, rank):
25:     from torchviz import make_dot
26: 
27:     outputs = model(*inputs)
28:     loss = reduce_to_scalar_loss(outputs)
29:     parameter_names = dict(model.named_parameters())
30:     dot = make_dot(loss, params=parameter_names, show_attrs=True, show_saved=True)
31:     if rank == 0:
32:         dot.render("torchviz.dot")
33: 
34: 
35: def profile_model(args, model, inputs, rank):
36:     with profile(activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA]) as prof:
37:         for i in range(args.repeat):
38:             with record_function("Forward"):
39:                 outputs = model(*inputs)
40:                 loss = reduce_to_scalar_loss(outputs)
````
- EN: Handles module imports such as `argparse`, `logging`, `os`, `functools`.
- CN: 处理模块导入，例如 `argparse`, `logging`, `os`, `functools`。
- EN: Implements callable logic such as `torchviz_model`, `profile_model`.
- CN: 实现可调用逻辑，例如 `torchviz_model`, `profile_model`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:             with record_function("Backward"):
42:                 loss.backward()
43:     if rank == 0:
44:         prof.export_chrome_trace(args.trace_file)
45: 
46: 
47: def run_model(args, model, inputs, key):
48:     rank = int(os.getenv("RANK", 0))
49:     world_size = int(os.getenv("WORLD_SIZE", 1))
50:     # result_q = []
51: 
52:     setup(rank, world_size)
53:     if args.device == "cuda":
54:         # needed for FSDP
55:         torch.cuda.set_device(rank)
56: 
57:     dev_rank = f"{args.device}:{rank}"
58:     model = model.to(dev_rank)
59: 
60:     def move_tensor(maybe_tensor):
61:         if torch.is_tensor(maybe_tensor):
62:             return maybe_tensor.to(dev_rank)
63:         return maybe_tensor
64: 
65:     inputs = pytree.tree_map(move_tensor, inputs)
66: 
67:     if args.fsdp:
68:         model = apply_fsdp(
69:             args,
70:             model,
71:             use_checkpointing=args.fsdp_checkpoint,
72:             use_wrap_policy=args.fsdp_wrap,
73:         )
74:     elif args.ddp:
75:         model = DDP(model)
76: 
77:     if args.verbose:
78:         print(model)
79: 
80:     if args.dynamo:
````
- EN: Implements callable logic such as `run_model`, `move_tensor`.
- CN: 实现可调用逻辑，例如 `run_model`, `move_tensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:         dynamo.reset()
 82:         if args.verbose:
 83:             dynamo.config.verbose = True
 84:             dynamo.config.log_level = logging.DEBUG
 85:         if args.dynamo_no_optimize_ddp:
 86:             dynamo.config.optimize_ddp = False
 87:         if args.dynamo == "inductor" and args.fsdp:
 88:             torch._inductor.config.triton.cudagraphs = False
 89:             log.warning("disabling inductor cudagraphs for compatibility with FSDP")
 90: 
 91:         def print_compile(gm, ex):
 92:             print(
 93:                 f"print_compile:\n{str(gm.graph)}\n-----------------------------------------"
 94:             )
 95:             return gm
 96: 
 97:         dynamo_ctx = dynamo.optimize(
 98:             print_compile if args.dynamo == "print" else args.dynamo
 99:         )
100:         model = dynamo_ctx(model)
101: 
102:     # warmup
103:     _ = timed(model, model_iter_fn, inputs, times=3, return_result=False)
104:     t_total = timed(
105:         model, model_iter_fn, inputs, times=args.repeat, return_result=False
106:     )
107:     if args.torchviz:
108:         torchviz_model(args, model, inputs, rank)
109:     if args.profile:
110:         profile_model(args, model, inputs, rank)
111: 
112:     cleanup()
113:     return t_total
114: 
115: 
116: if __name__ == "__main__":
117:     parser = argparse.ArgumentParser()
118:     parser.add_argument("--device", default="cuda")
119:     parser.add_argument(
120:         "--dynamo",
````
- EN: Implements callable logic such as `print_compile`.
- CN: 实现可调用逻辑，例如 `print_compile`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````python
121:         default=None,
122:         help="if set to a str, uses dynamo[str] backend. else, eager",
123:     )
124:     parser.add_argument("--verbose", action="store_true")
125:     parser.add_argument("--batch-size", "--batch_size", default=None)
126:     parser.add_argument(
127:         "--torchviz", action="store_true", help="Dump autograd graph with torchviz"
128:     )
129:     parser.add_argument("--profile", action="store_true", help="Run the profiler")
130:     parser.add_argument(
131:         "--trace-file", "--trace_file", default="profile.json", help="Run the profiler"
132:     )
133:     parser.add_argument("--repeat", default=10, help="Repeats for timing run")
134:     parser.add_argument(
135:         "--dynamo-no-optimize-ddp",
136:         "--dynamo_no_optimize_ddp",
137:         action="store_true",
138:         help="Disable dynamo's ddp optimizer (enabled by default)",
139:     )
140:     parser.add_argument(
141:         "--fsdp-checkpoint",
142:         "--fsdp_checkpoint",
143:         action="store_true",
144:         help="Use gradient checkpointing via model-specific policy",
145:     )
146:     parser.add_argument(
147:         "--fsdp-wrap",
148:         "--fsdp_wrap",
149:         action="store_true",
150:         help="Apply fsdp to submodules via model-specific policy",
151:     )
152: 
153:     dist_arg = parser.add_mutually_exclusive_group()
154:     dist_arg.add_argument("--ddp", action="store_true")
155:     dist_arg.add_argument("--fsdp", action="store_true")
156: 
157:     model_arg = parser.add_mutually_exclusive_group(required=True)
158:     model_arg.add_argument(
159:         "--torchbench-model",
160:         "--torchbench_model",
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-177
````python
161:         help="name of torchbench model, e.g. BERT_pytorch",
162:     )
163:     model_arg.add_argument(
164:         "--toy-model", "--toy_model", action="store_true", help="use toy model instead"
165:     )
166:     args = parser.parse_args()
167: 
168:     model_name = args.torchbench_model
169:     if args.toy_model:
170:         model_name = "ToyModel"
171:     model, inputs = get_model(args)
172: 
173:     fn = partial(run_model, args, model, inputs)
174: 
175:     world_size = os.getenv("WORLD_SIZE", 1)
176:     t_total = fn(f"{model_name}_{world_size}")
177:     print(f"mean latency {t_total / args.repeat} across {args.repeat} runs")
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `torchviz_model` / 符号 `torchviz_model`
- Symbol `profile_model` / 符号 `profile_model`
- Symbol `run_model` / 符号 `run_model`
- Symbol `move_tensor` / 符号 `move_tensor`

## Dependencies / 依赖关系
- Python imports: `argparse`, `logging`, `os`, `functools`, `torch`, `torch._dynamo`, `torch.utils._pytree`, `torch._dynamo.testing`, `torch.nn.parallel`, `torch.profiler`
- Python 导入: `argparse`, `logging`, `os`, `functools`, `torch`, `torch._dynamo`, `torch.utils._pytree`, `torch._dynamo.testing`, `torch.nn.parallel`, `torch.profiler`

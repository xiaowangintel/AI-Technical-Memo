# dist_util.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/dist_util.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: import functools
 3: import importlib
 4: import os
 5: 
 6: import torch
 7: import torch.distributed as dist
 8: import torch.nn as nn
 9: from torch._dynamo.testing import reduce_to_scalar_loss
10: from torch.distributed.algorithms._checkpoint.checkpoint_wrapper import (
11:     apply_activation_checkpointing,
12:     checkpoint_wrapper,
13:     CheckpointImpl,
14: )
15: from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
16: from torch.distributed.fsdp.wrap import ModuleWrapPolicy
17: 
18: 
19: try:
20:     from .torchbench import setup_torchbench_cwd
21: except ImportError:
22:     from torchbench import setup_torchbench_cwd
23: 
24: 
25: def setup(rank, world_size):
26:     os.environ["MASTER_ADDR"] = os.getenv("MASTER_ADDR", "localhost")
27:     os.environ["MASTER_PORT"] = os.getenv("MASTER_PORT", "12355")
28:     os.environ["RANK"] = os.getenv("RANK", "0")
29:     os.environ["WORLD_SIZE"] = os.getenv("WORLD_SIZE", "1")
30:     dist.init_process_group("nccl")
31: 
32: 
33: def cleanup():
34:     dist.destroy_process_group()
35: 
36: 
37: class CustomLinear(torch.nn.Module):
38:     def __init__(self, a, b):
39:         super().__init__()
40:         self.weight = nn.Parameter(torch.randn(a, b))
````
- EN: Handles module imports such as `argparse`, `functools`, `importlib`, `os`.
- CN: 处理模块导入，例如 `argparse`, `functools`, `importlib`, `os`。
- EN: Declares or extends types including `CustomLinear`.
- CN: 声明或扩展类型，包括 `CustomLinear`。
- EN: Implements callable logic such as `setup`, `cleanup`, `__init__`.
- CN: 实现可调用逻辑，例如 `setup`, `cleanup`, `__init__`。

### Lines 41-80
````python
41: 
42:     def forward(self, x):
43:         return torch.mm(x, self.weight)
44: 
45: 
46: class MyModule(torch.nn.Module):
47:     def __init__(self, a, b):
48:         super().__init__()
49:         self.net = nn.Sequential(
50:             nn.Linear(a, b),
51:             nn.ReLU(),
52:         )
53: 
54:     def forward(self, x):
55:         return self.net(x)
56: 
57: 
58: class ToyModel(nn.Module):
59:     def __init__(self):
60:         super().__init__()
61:         self.net = nn.Sequential(
62:             *[nn.Linear(10, 10000), nn.ReLU()]
63:             + [nn.Linear(10000, 10000), nn.ReLU()]
64:             + [MyModule(10000, 10000)]
65:             + [MyModule(10000, 1000)]
66:             + [MyModule(1000, 1000)]
67:             + [MyModule(1000, 1000)]
68:             + [MyModule(1000, 1000)]
69:             + [MyModule(1000, 1000)]
70:             + [MyModule(1000, 1000)]
71:             + [MyModule(1000, 1000)]
72:             + [MyModule(1000, 1000)]
73:             + [nn.Linear(1000, 5)]
74:         )
75: 
76:     def forward(self, x):
77:         return self.net(x)
78: 
79: 
80: def model_iter_fn(model, example_inputs, collect_outputs=False):
````
- EN: Declares or extends types including `MyModule`, `ToyModel`.
- CN: 声明或扩展类型，包括 `MyModule`, `ToyModel`。
- EN: Implements callable logic such as `forward`, `__init__`, `model_iter_fn`.
- CN: 实现可调用逻辑，例如 `forward`, `__init__`, `model_iter_fn`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:     outputs = model(*example_inputs)
 82:     loss = reduce_to_scalar_loss(outputs)
 83:     loss.backward()
 84:     if collect_outputs:
 85:         return outputs
 86: 
 87: 
 88: def get_model(args):
 89:     if args.torchbench_model:
 90:         setup_torchbench_cwd()
 91:         module = importlib.import_module(
 92:             f"torchbenchmark.models.{args.torchbench_model}"
 93:         )
 94:         benchmark_cls = getattr(module, "Model", None)
 95:         bm = benchmark_cls(test="train", device=args.device, batch_size=args.batch_size)
 96:         model, inputs = bm.get_module()
 97:     elif args.toy_model:
 98:         model = ToyModel()
 99:         inputs = (torch.randn(20, 10),)
100:     else:
101:         raise argparse.ArgumentError(
102:             args.torchbench_model, message="Must specify a model"
103:         )
104: 
105:     return model, inputs
106: 
107: 
108: def fsdp_checkpointing_base(model, blocks):
109:     """apply activation checkpointing to model
110:     returns None as model is updated directly
111:     """
112:     non_reentrant_wrapper = functools.partial(
113:         checkpoint_wrapper,
114:         offload_to_cpu=False,
115:         checkpoint_impl=CheckpointImpl.NO_REENTRANT,
116:     )
117: 
118:     def check_fn(submodule):
119:         return isinstance(submodule, blocks)
120: 
````
- EN: Implements callable logic such as `get_model`, `fsdp_checkpointing_base`, `check_fn`.
- CN: 实现可调用逻辑，例如 `get_model`, `fsdp_checkpointing_base`, `check_fn`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-142
````python
121:     apply_activation_checkpointing(
122:         model, checkpoint_wrapper_fn=non_reentrant_wrapper, check_fn=check_fn
123:     )
124: 
125: 
126: MODEL_FSDP_WRAP = {
127:     "toy_model": (MyModule,),
128: }
129: 
130: 
131: def apply_fsdp(args, model, use_checkpointing=False, use_wrap_policy=True):
132:     wrap_policy = None
133:     blocks = MODEL_FSDP_WRAP[
134:         "toy_model" if model.__class__ is ToyModel else args.torchbench_model
135:     ]
136:     if use_wrap_policy:
137:         wrap_policy = ModuleWrapPolicy(blocks)
138: 
139:     model = FSDP(model, auto_wrap_policy=wrap_policy, use_orig_params=True)
140:     if use_checkpointing:
141:         fsdp_checkpointing_base(model, blocks)
142:     return model
````
- EN: Implements callable logic such as `apply_fsdp`.
- CN: 实现可调用逻辑，例如 `apply_fsdp`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `setup` / 符号 `setup`
- Symbol `cleanup` / 符号 `cleanup`
- Symbol `CustomLinear` / 符号 `CustomLinear`
- Symbol `__init__` / 符号 `__init__`

## Dependencies / 依赖关系
- Python imports: `argparse`, `functools`, `importlib`, `os`, `torch`, `torch.distributed`, `torch.nn`, `torch._dynamo.testing`, `torch.distributed.algorithms._checkpoint.checkpoint_wrapper`, `torch.distributed.fsdp`
- Python 导入: `argparse`, `functools`, `importlib`, `os`, `torch`, `torch.distributed`, `torch.nn`, `torch._dynamo.testing`, `torch.distributed.algorithms._checkpoint.checkpoint_wrapper`, `torch.distributed.fsdp`

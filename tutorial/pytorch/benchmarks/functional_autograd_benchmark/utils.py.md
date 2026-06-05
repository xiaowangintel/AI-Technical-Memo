# utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/functional_autograd_benchmark/utils.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: from collections import defaultdict
 2: from collections.abc import Callable
 3: 
 4: import torch
 5: from torch import nn, Tensor
 6: 
 7: 
 8: # Type helpers
 9: InputsType = Tensor | tuple[Tensor, ...]
10: # A Getter takes in a device and returns a callable and the inputs to that callable
11: GetterReturnType = tuple[Callable[..., Tensor], InputsType]
12: GetterType = Callable[[torch.device], GetterReturnType]
13: # V here refers to the v in either vjp, jvp, vhp or hvp
14: VType = None | Tensor | tuple[Tensor, ...]
15: # Type used to store timing results. The first key is the model name, the second key
16: # is the task name, the result is a Tuple of: speedup, mean_before, var_before, mean_after, var_after.
17: TimingResultType = dict[str, dict[str, tuple[float, ...]]]
18: 
19: 
20: # Utilities to make nn.Module "functional"
21: # In particular the goal is to be able to provide a function that takes as input
22: # the parameters and evaluate the nn.Module using fixed inputs.
23: def _del_nested_attr(obj: nn.Module, names: list[str]) -> None:
24:     """
25:     Deletes the attribute specified by the given list of names.
26:     For example, to delete the attribute obj.conv.weight,
27:     use _del_nested_attr(obj, ['conv', 'weight'])
28:     """
29:     if len(names) == 1:
30:         delattr(obj, names[0])
31:     else:
32:         _del_nested_attr(getattr(obj, names[0]), names[1:])
33: 
34: 
35: def _set_nested_attr(obj: nn.Module, names: list[str], value: Tensor) -> None:
36:     """
37:     Set the attribute specified by the given list of names to value.
38:     For example, to set the attribute obj.conv.weight,
39:     use _del_nested_attr(obj, ['conv', 'weight'], value)
40:     """
````
- EN: Handles module imports such as `collections`, `collections.abc`, `torch`.
- CN: 处理模块导入，例如 `collections`, `collections.abc`, `torch`。
- EN: Implements callable logic such as `_del_nested_attr`, `_set_nested_attr`.
- CN: 实现可调用逻辑，例如 `_del_nested_attr`, `_set_nested_attr`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-80
````python
41:     if len(names) == 1:
42:         setattr(obj, names[0], value)
43:     else:
44:         _set_nested_attr(getattr(obj, names[0]), names[1:], value)
45: 
46: 
47: def extract_weights(mod: nn.Module) -> tuple[tuple[Tensor, ...], list[str]]:
48:     """
49:     This function removes all the Parameters from the model and
50:     return them as a tuple as well as their original attribute names.
51:     The weights must be re-loaded with `load_weights` before the model
52:     can be used again.
53:     Note that this function modifies the model in place and after this
54:     call, mod.parameters() will be empty.
55:     """
56:     orig_params = tuple(mod.parameters())
57:     # Remove all the parameters in the model
58:     names = []
59:     for name, p in list(mod.named_parameters()):
60:         _del_nested_attr(mod, name.split("."))
61:         names.append(name)
62: 
63:     # Make params regular Tensors instead of nn.Parameter
64:     params = tuple(p.detach().requires_grad_() for p in orig_params)
65:     return params, names
66: 
67: 
68: def load_weights(mod: nn.Module, names: list[str], params: tuple[Tensor, ...]) -> None:
69:     """
70:     Reload a set of weights so that `mod` can be used again to perform a forward pass.
71:     Note that the `params` are regular Tensors (that can have history) and so are left
72:     as Tensors. This means that mod.parameters() will still be empty after this call.
73:     """
74:     for name, p in zip(names, params):
75:         _set_nested_attr(mod, name.split("."), p)
76: 
77: 
78: # Utilities to read/write markdown table-like content.
79: def to_markdown_table(
80:     res: TimingResultType, header: tuple[str, ...] | None = None
````
- EN: Implements callable logic such as `extract_weights`, `load_weights`, `to_markdown_table`.
- CN: 实现可调用逻辑，例如 `extract_weights`, `load_weights`, `to_markdown_table`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81: ) -> str:
 82:     if header is None:
 83:         header = ("model", "task", "mean", "var")
 84:     out = ""
 85: 
 86:     def write_line(*args):
 87:         nonlocal out
 88:         out += f"| {' | '.join(str(a) for a in args)} |\n"
 89: 
 90:     # Make it a markdown table
 91:     write_line(*header)
 92:     write_line(*["--"] * len(header))
 93:     for model, tasks in res.items():
 94:         for task, line in tasks.items():
 95:             write_line(*(model, task) + line)
 96: 
 97:     return out
 98: 
 99: 
100: def from_markdown_table(data: str) -> TimingResultType:
101:     out = data.strip().split("\n")
102:     out = out[2:]  # Ignore the header lines
103: 
104:     res: TimingResultType
105:     res = defaultdict(defaultdict)
106: 
107:     for line in out:
108:         model, task, mean, var = (f.strip() for f in line.strip().split("|") if f)
109:         res[model][task] = (float(mean), float(var))
110: 
111:     return res
112: 
113: 
114: def check_for_functorch():
115:     try:
116:         import functorch  # noqa: F401
117: 
118:         return True
119:     except ImportError:
120:         return False
````
- EN: Handles module imports such as `functorch`.
- CN: 处理模块导入，例如 `functorch`。
- EN: Implements callable logic such as `write_line`, `from_markdown_table`, `check_for_functorch`.
- CN: 实现可调用逻辑，例如 `write_line`, `from_markdown_table`, `check_for_functorch`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `_del_nested_attr` / 符号 `_del_nested_attr`
- Symbol `_set_nested_attr` / 符号 `_set_nested_attr`
- Symbol `extract_weights` / 符号 `extract_weights`
- Symbol `load_weights` / 符号 `load_weights`

## Dependencies / 依赖关系
- Python imports: `collections`, `collections.abc`, `torch`, `functorch`
- Python 导入: `collections`, `collections.abc`, `torch`, `functorch`

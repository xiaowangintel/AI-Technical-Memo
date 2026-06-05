# distributed.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/distributed.py`
- **Purpose / 用途:** Implementation module for distributed; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols ExptAssignment, make_expt_dict_uniform, make_expt_dict_random, make_expt_assignment, _convert_launch_metadata. / 用于 distributed 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 ExptAssignment、make_expt_dict_uniform、make_expt_dict_random、make_expt_assignment、_convert_launch_metadata。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-1 (module)
```python
1| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 2-8 (module)
```python
2| import torch
3| import triton
4| import triton.language as tl
5| import random
6| from dataclasses import dataclass
7| from .distributed_details.mesh import SymmetricMemoryPool
8| 
```
**EN:** This block imports `torch`, `triton`, `triton.language`, `random`, `dataclasses (dataclass)`, `.distributed_details.mesh (SymmetricMemoryPool)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch`, `triton`, `triton.language`, `random`, `dataclasses (dataclass)`, `.distributed_details.mesh (SymmetricMemoryPool)` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 9-12 (ExptAssignment)
```python
 9| @dataclass
10| class ExptAssignment:
11|     # torch.Tensor[n_expt_shard, n_expt_tot // 32]
12|     # (expt_bitmask[i, j//32] >> j%32) & 1 == 1 iff expert j is owned by shard i
```
**EN:** Defines class `ExptAssignment` with decorators `dataclass` to organize related behavior. Key fields include `expt_bitmask`, `expt_boolmask`, `expt_map`, `n_expts_per_shard`.

**CN:** 定义类 `ExptAssignment`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `expt_bitmask`, `expt_boolmask`, `expt_map`, `n_expts_per_shard`.

### Block 4 — Lines 13-15 (ExptAssignment)
```python
13|     expt_bitmask: torch.Tensor
14|     # torch.Tensor[n_expt_shard, n_expt_tot]
15|     # expt_boolmask[i, j] == True iff expert j is owned by shard i
```
**EN:** Annotated assignment stores `expt_bitmask` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `expt_bitmask` and 声明该带注解的字段.

### Block 5 — Lines 16-19 (ExptAssignment)
```python
16|     expt_boolmask: torch.Tensor
17|     # torch.Tensor[n_expt_shard, n_expt_tot]
18|     # expt_map[i, j] is the local expert id of expert j in shard i,
19|     # or -1 if expert j is not owned by shard i
```
**EN:** Annotated assignment stores `expt_boolmask` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `expt_boolmask` and 声明该带注解的字段.

### Block 6 — Lines 20-21 (ExptAssignment)
```python
20|     expt_map: torch.Tensor
21|     # number of experts per shard
```
**EN:** Annotated assignment stores `expt_map` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `expt_map` and 声明该带注解的字段.

### Block 7 — Lines 22-24 (ExptAssignment)
```python
22|     n_expts_per_shard: list[int]
23| 
24| 
```
**EN:** Annotated assignment stores `n_expts_per_shard` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `n_expts_per_shard` and 声明该带注解的字段.

### Block 8 — Lines 25-25 (make_expt_dict_uniform)
```python
25| def make_expt_dict_uniform(n_expt_shard, n_expt_tot):
```
**EN:** Defines function `make_expt_dict_uniform(n_expt_shard, n_expt_tot)` for this module. The body mainly prepares intermediate values; iterates over cases or dimensions; returns the computed result. It uses calls such as `dict`, `range`, `list` to implement its workflow.

**CN:** 定义函数 `make_expt_dict_uniform(n_expt_shard, n_expt_tot)`，供本模块使用. 主体主要准备中间值; 遍历测试用例或维度; 返回计算结果. 其中会调用 `dict`, `range`, `list` 来实现其工作流程.

### Block 9 — Lines 26-29 (make_expt_dict_uniform)
```python
26|     """
27|     create expert assignment dictionary where shard i owns:
28|     [i*(n_expt_tot//n_expt_shard)...(i+1)*(n_expt_tot//n_expt_shard))
29|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 10 — Lines 30-30 (make_expt_dict_uniform)
```python
30|     expt_dict = dict()
```
**EN:** Assigns `expt_dict` and calls `dict`.

**CN:** 将 `expt_dict`，并调用 `dict`.

### Block 11 — Lines 31-34 (make_expt_dict_uniform)
```python
31|     for i in range(n_expt_shard):
32|         start = (n_expt_tot // n_expt_shard) * i
33|         end = (n_expt_tot // n_expt_shard) * (i + 1)
34|         expt_dict[i] = list(range(start, end))
```
**EN:** Loops over `range(n_expt_shard)` with target `i`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `range(n_expt_shard)` ，目标变量为 `i`. 循环体主要准备中间值; 准备中间值.

### Block 12 — Lines 35-37 (make_expt_dict_uniform)
```python
35|     return expt_dict
36| 
37| 
```
**EN:** Returns `expt_dict`.

**CN:** 返回 `expt_dict`.

### Block 13 — Lines 38-38 (make_expt_dict_random)
```python
38| def make_expt_dict_random(n_expt_shard, n_expt_tot):
```
**EN:** Defines function `make_expt_dict_random(n_expt_shard, n_expt_tot)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `dict`, `random.Random`, `list`, `rng.shuffle`, `range` to implement its workflow.

**CN:** 定义函数 `make_expt_dict_random(n_expt_shard, n_expt_tot)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `dict`, `random.Random`, `list`, `rng.shuffle`, `range` 来实现其工作流程.

### Block 14 — Lines 39-42 (make_expt_dict_random)
```python
39|     """
40|     create expert assignment dictionary where each shard owns
41|     a disjoint random subset of experts
42|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 15 — Lines 43-44 (make_expt_dict_random)
```python
43|     expt_dict = dict()
44|     # random permutation of experts
```
**EN:** Assigns `expt_dict` and calls `dict`.

**CN:** 将 `expt_dict`，并调用 `dict`.

### Block 16 — Lines 45-45 (make_expt_dict_random)
```python
45|     rng = random.Random(0)
```
**EN:** Assigns `rng` and calls `random.Random`.

**CN:** 将 `rng`，并调用 `random.Random`.

### Block 17 — Lines 46-46 (make_expt_dict_random)
```python
46|     perm = list(range(n_expt_tot))
```
**EN:** Assigns `perm` and calls `list`.

**CN:** 将 `perm`，并调用 `list`.

### Block 18 — Lines 47-48 (make_expt_dict_random)
```python
47|     rng.shuffle(perm)
48|     # random (distinct) cut points; ensures no empty shard
```
**EN:** Calls `rng.shuffle` for side effects, registration, or validation.

**CN:** 调用 `rng.shuffle` ，用于副作用、注册或校验。

### Block 19 — Lines 49-49 (make_expt_dict_random)
```python
49|     cuts = [0] + sorted(rng.sample(range(1, n_expt_tot), n_expt_shard - 1)) + [n_expt_tot]
```
**EN:** Assigns `cuts` and evaluates `[0] + sorted(rng.sample(range(1, n_expt_tot), n_expt_shard - 1)) + ...`.

**CN:** 将 `cuts` and 计算 `[0] + sorted(rng.sample(range(1, n_expt_tot), n_expt_shard - 1)) + ...`.

### Block 20 — Lines 50-52 (make_expt_dict_random)
```python
50|     for i in range(n_expt_shard):
51|         a, b = cuts[i], cuts[i + 1]
52|         expt_dict[i] = perm[a:b]
```
**EN:** Loops over `range(n_expt_shard)` with target `i`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `range(n_expt_shard)` ，目标变量为 `i`. 循环体主要准备中间值; 准备中间值.

### Block 21 — Lines 53-55 (make_expt_dict_random)
```python
53|     return expt_dict
54| 
55| 
```
**EN:** Returns `expt_dict`.

**CN:** 返回 `expt_dict`.

### Block 22 — Lines 56-56 (make_expt_assignment)
```python
56| def make_expt_assignment(n_expt_shard, n_expt_tot, expt_dict: dict[int, list[int]], device) -> ExptAssignment:
```
**EN:** Defines function `make_expt_assignment(n_expt_shard, n_expt_tot, expt_dict, device)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.zeros`, `expt_dict.items`, `expt_bitmask.to`, `expt_boolmask.to`, `torch.full` to implement its workflow.

**CN:** 定义函数 `make_expt_assignment(n_expt_shard, n_expt_tot, expt_dict, device)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `torch.zeros`, `expt_dict.items`, `expt_bitmask.to`, `expt_boolmask.to`, `torch.full` 来实现其工作流程.

### Block 23 — Lines 57-63 (make_expt_assignment)
```python
57|     """
58|     n_expt_shard: int
59|     n_expt_tot: int
60|     expt_dict: dict[int, list[int]]
61|       expt_dict[i] is the list of expert ids owned by shard i
62|     """
63|     # make expt_bitmask
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 24 — Lines 64-64 (make_expt_assignment)
```python
64|     words = (n_expt_tot + 31) // 32  # safe even if n_expt_tot not multiple of 32
```
**EN:** Assigns `words` and evaluates `(n_expt_tot + 31) // 32`.

**CN:** 将 `words` and 计算 `(n_expt_tot + 31) // 32`.

### Block 25 — Lines 65-65 (make_expt_assignment)
```python
65|     expt_bitmask = torch.zeros((n_expt_shard, words), dtype=torch.int32)
```
**EN:** Assigns `expt_bitmask` and calls `torch.zeros`.

**CN:** 将 `expt_bitmask`，并调用 `torch.zeros`.

### Block 26 — Lines 66-66 (make_expt_assignment)
```python
66|     expt_boolmask = torch.zeros((n_expt_shard, n_expt_tot), dtype=torch.bool)
```
**EN:** Assigns `expt_boolmask` and calls `torch.zeros`.

**CN:** 将 `expt_boolmask`，并调用 `torch.zeros`.

### Block 27 — Lines 67-67 (make_expt_assignment)
```python
67|     counts = {expt_id: 0 for expt_id in range(n_expt_tot)}
```
**EN:** Assigns `counts` and evaluates `{expt_id: 0 for expt_id in range(n_expt_tot)}`.

**CN:** 将 `counts` and 计算 `{expt_id: 0 for expt_id in range(n_expt_tot)}`.

### Block 28 — Lines 68-82 (make_expt_assignment)
```python
68|     for shard, experts in expt_dict.items():
69|         if len(experts) == 0:
70|             raise ValueError(f"shard {shard} has no experts")
71|         if shard < 0 or shard >= n_expt_shard:
72|             raise ValueError(f"shard {shard} out of range [0, {n_expt_shard})")
73|         if not isinstance(experts, (list, tuple)):
74|             raise TypeError(f"expt_dict[{shard}] must be a list/tuple of ints")
75|         for e in experts:
76|             counts[e] += 1
77|             if not (0 <= e < n_expt_tot):
78|                 raise ValueError(f"expert id {e} out of range [0, {n_expt_tot})")
79|             word = e >> 5  # e // 32
80|             bit = e & 31  # e % 32
81|             expt_bitmask[shard, word] |= (1 << bit)
82|             expt_boolmask[shard, e] = True
```
**EN:** Loops over `expt_dict.items()` with target `(shard, experts)`. The loop body mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 遍历 `expt_dict.items()` ，目标变量为 `(shard, experts)`. 循环体主要根据运行时条件分支; 根据运行时条件分支.

### Block 29 — Lines 83-84 (make_expt_assignment)
```python
83|     if not all(counts[e] == 1 for e in range(n_expt_tot)):
84|         raise ValueError("each expert must be owned by exactly one shard")
```
**EN:** Checks `not all((counts[e] == 1 for e in range(n_expt_tot)))`..

**CN:** 检查 `not all((counts[e] == 1 for e in range(n_expt_tot)))`..

### Block 30 — Lines 85-85 (make_expt_assignment)
```python
85|     expt_bitmask = expt_bitmask.to(device)
```
**EN:** Assigns `expt_bitmask` and calls `expt_bitmask.to`.

**CN:** 将 `expt_bitmask`，并调用 `expt_bitmask.to`.

### Block 31 — Lines 86-87 (make_expt_assignment)
```python
86|     expt_boolmask = expt_boolmask.to(device)
87|     # make expt_map
```
**EN:** Assigns `expt_boolmask` and calls `expt_boolmask.to`.

**CN:** 将 `expt_boolmask`，并调用 `expt_boolmask.to`.

### Block 32 — Lines 88-88 (make_expt_assignment)
```python
88|     expt_map = torch.full((n_expt_shard, n_expt_tot), -1, dtype=torch.int32)
```
**EN:** Assigns `expt_map` and calls `torch.full`.

**CN:** 将 `expt_map`，并调用 `torch.full`.

### Block 33 — Lines 89-91 (make_expt_assignment)
```python
89|     for shard, experts in expt_dict.items():
90|         for local_id, global_id in enumerate(sorted(experts)):
91|             expt_map[shard, global_id] = local_id
```
**EN:** Loops over `expt_dict.items()` with target `(shard, experts)`. The loop body mainly iterates over cases or dimensions.

**CN:** 遍历 `expt_dict.items()` ，目标变量为 `(shard, experts)`. 循环体主要遍历测试用例或维度.

### Block 34 — Lines 92-93 (make_expt_assignment)
```python
92|     expt_map = expt_map.to(device)
93|     # number of experts per shard
```
**EN:** Assigns `expt_map` and calls `expt_map.to`.

**CN:** 将 `expt_map`，并调用 `expt_map.to`.

### Block 35 — Lines 94-94 (make_expt_assignment)
```python
94|     n_expts_per_shard = [len(experts) for experts in expt_dict.values()]
```
**EN:** Assigns `n_expts_per_shard` and evaluates `[len(experts) for experts in expt_dict.values()]`.

**CN:** 将 `n_expts_per_shard` and 计算 `[len(experts) for experts in expt_dict.values()]`.

### Block 36 — Lines 95-100 (make_expt_assignment)
```python
 95|     return ExptAssignment(expt_bitmask, expt_boolmask, expt_map, n_expts_per_shard)
 96| 
 97| 
 98| # ------------------------------------------------------------
 99| 
100| 
```
**EN:** Returns `ExptAssignment(expt_bitmask, expt_boolmask, expt_map, n_expts_per_shard)`.

**CN:** 返回 `ExptAssignment(expt_bitmask, expt_boolmask, expt_map, n_expts_per_shard)`.

### Block 37 — Lines 101-101 (_convert_launch_metadata)
```python
101| def _convert_launch_metadata(grid, kernel, args):
```
**EN:** Defines function `_convert_launch_metadata(grid, kernel, args)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `args['expt_indx_ptr'].int`, `src.element_size`, `(src_rank_filter[local_expt_indx // 3...`, `torch.sum`, `(src_rank_filter[expt_indx // 32] >> ...` to implement its workflow.

**CN:** 定义函数 `_convert_launch_metadata(grid, kernel, args)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `args['expt_indx_ptr'].int`, `src.element_size`, `(src_rank_filter[local_expt_indx // 3...`, `torch.sum`, `(src_rank_filter[expt_indx // 32] >> ...` 来实现其工作流程.

### Block 38 — Lines 102-102 (_convert_launch_metadata)
```python
102|     src = args["src_ptr"]
```
**EN:** Assigns `src` and evaluates `args['src_ptr']`.

**CN:** 将 `src` and 计算 `args['src_ptr']`.

### Block 39 — Lines 103-103 (_convert_launch_metadata)
```python
103|     src_rank = args["SRC_RANK"]
```
**EN:** Assigns `src_rank` and evaluates `args['SRC_RANK']`.

**CN:** 将 `src_rank` and 计算 `args['SRC_RANK']`.

### Block 40 — Lines 104-104 (_convert_launch_metadata)
```python
104|     n_tokens_local = args["n_tokens_local"]
```
**EN:** Assigns `n_tokens_local` and evaluates `args['n_tokens_local']`.

**CN:** 将 `n_tokens_local` and 计算 `args['n_tokens_local']`.

### Block 41 — Lines 105-105 (_convert_launch_metadata)
```python
105|     src_row_start = n_tokens_local * src_rank
```
**EN:** Assigns `src_row_start` and evaluates `n_tokens_local * src_rank`.

**CN:** 将 `src_row_start` and 计算 `n_tokens_local * src_rank`.

### Block 42 — Lines 106-106 (_convert_launch_metadata)
```python
106|     expt_filter = args["expt_filter_ptr"]
```
**EN:** Assigns `expt_filter` and evaluates `args['expt_filter_ptr']`.

**CN:** 将 `expt_filter` and 计算 `args['expt_filter_ptr']`.

### Block 43 — Lines 107-107 (_convert_launch_metadata)
```python
107|     expt_indx = args["expt_indx_ptr"].int()
```
**EN:** Assigns `expt_indx` and calls `args['expt_indx_ptr'].int`.

**CN:** 将 `expt_indx`，并调用 `args['expt_indx_ptr'].int`.

### Block 44 — Lines 108-108 (_convert_launch_metadata)
```python
108|     d_model = src.shape[1]
```
**EN:** Assigns `d_model` and evaluates `src.shape[1]`.

**CN:** 将 `d_model` and 计算 `src.shape[1]`.

### Block 45 — Lines 109-109 (_convert_launch_metadata)
```python
109|     elem_bytes = src.element_size()
```
**EN:** Assigns `elem_bytes` and calls `src.element_size`.

**CN:** 将 `elem_bytes`，并调用 `src.element_size`.

### Block 46 — Lines 110-111 (_convert_launch_metadata)
```python
110|     src_bytes = src.numel() * elem_bytes
111|     # Find out number of tokens being dispatched out from this GPU
```
**EN:** Assigns `src_bytes` and evaluates `src.numel() * elem_bytes`.

**CN:** 将 `src_bytes` and 计算 `src.numel() * elem_bytes`.

### Block 47 — Lines 112-112 (_convert_launch_metadata)
```python
112|     local_expt_indx = expt_indx[src_row_start:src_row_start + n_tokens_local]
```
**EN:** Assigns `local_expt_indx` and evaluates `expt_indx[src_row_start:src_row_start + n_tokens_local]`.

**CN:** 将 `local_expt_indx` and 计算 `expt_indx[src_row_start:src_row_start + n_tokens_local]`.

### Block 48 — Lines 113-113 (_convert_launch_metadata)
```python
113|     src_rank_filter = expt_filter[src_rank]
```
**EN:** Assigns `src_rank_filter` and evaluates `expt_filter[src_rank]`.

**CN:** 将 `src_rank_filter` and 计算 `expt_filter[src_rank]`.

### Block 49 — Lines 114-114 (_convert_launch_metadata)
```python
114|     local_filter = ((src_rank_filter[local_expt_indx // 32] >> (local_expt_indx % 32)) & 1).to(torch.int32)
```
**EN:** Assigns `local_filter` and calls `(src_rank_filter[local_expt_indx // 32] >> loca...`.

**CN:** 将 `local_filter`，并调用 `(src_rank_filter[local_expt_indx // 32] >> loca...`.

### Block 50 — Lines 115-115 (_convert_launch_metadata)
```python
115|     dst_local_tokens = torch.sum(local_filter)
```
**EN:** Assigns `dst_local_tokens` and calls `torch.sum`.

**CN:** 将 `dst_local_tokens`，并调用 `torch.sum`.

### Block 51 — Lines 116-116 (_convert_launch_metadata)
```python
116|     dst_output_tokens = local_filter.numel() - dst_local_tokens
```
**EN:** Assigns `dst_output_tokens` and evaluates `local_filter.numel() - dst_local_tokens`.

**CN:** 将 `dst_output_tokens` and 计算 `local_filter.numel() - dst_local_tokens`.

### Block 52 — Lines 117-117 (_convert_launch_metadata)
```python
117|     global_filter = ((src_rank_filter[expt_indx // 32] >> (expt_indx % 32)) & 1).to(torch.int32)
```
**EN:** Assigns `global_filter` and calls `(src_rank_filter[expt_indx // 32] >> expt_indx ...`.

**CN:** 将 `global_filter`，并调用 `(src_rank_filter[expt_indx // 32] >> expt_indx ...`.

### Block 53 — Lines 118-119 (_convert_launch_metadata)
```python
118|     dst_input_tokens = torch.sum(global_filter) - dst_local_tokens
119|     # Calculate the number of bytes transferred out from this GPU
```
**EN:** Assigns `dst_input_tokens` and evaluates `torch.sum(global_filter) - dst_local_tokens`.

**CN:** 将 `dst_input_tokens` and 计算 `torch.sum(global_filter) - dst_local_tokens`.

### Block 54 — Lines 120-120 (_convert_launch_metadata)
```python
120|     dram_bytes = src_bytes + dst_local_tokens * d_model * elem_bytes
```
**EN:** Assigns `dram_bytes` and evaluates `src_bytes + dst_local_tokens * d_model * elem_bytes`.

**CN:** 将 `dram_bytes` and 计算 `src_bytes + dst_local_tokens * d_model * elem_bytes`.

### Block 55 — Lines 121-126 (_convert_launch_metadata)
```python
121|     if "dp_to_ep" in kernel.name:
122|         dram_bytes += dst_input_tokens * d_model * elem_bytes
123|     elif "ep_to_dp" in kernel.name:
124|         dram_bytes += dst_output_tokens * d_model * elem_bytes
125|     else:
126|         raise ValueError(f"unknown kernel name {kernel.name}")
```
**EN:** Checks `'dp_to_ep' in kernel.name`., while the else branch branches on runtime conditions.

**CN:** 检查 `'dp_to_ep' in kernel.name`.；而 else 分支根据运行时条件分支.

### Block 56 — Lines 127-127 (_convert_launch_metadata)
```python
127|     nvlink_bytes = (dst_output_tokens + dst_input_tokens) * d_model * elem_bytes
```
**EN:** Assigns `nvlink_bytes` and evaluates `(dst_output_tokens + dst_input_tokens) * d_model * elem_bytes`.

**CN:** 将 `nvlink_bytes` and 计算 `(dst_output_tokens + dst_input_tokens) * d_model * elem_bytes`.

### Block 57 — Lines 128-134 (_convert_launch_metadata)
```python
128|     return {
129|         "name": f"{kernel.name} [tokens={n_tokens_local}, d_model={d_model}]",
130|         "bytes": dram_bytes,
131|         "nvlink_bytes": nvlink_bytes,
132|     }
133| 
134| 
```
**EN:** Returns `{'name': f'{kernel.name} [tokens={n_tokens_local}, d_model={d_model}]', 'byte...`.

**CN:** 返回 `{'name': f'{kernel.name} [tokens={n_tokens_local}, d_model={d_model}]', 'byte...`.

### Block 58 — Lines 135-147 (_convert_dp_to_ep)
```python
135| @triton.jit(launch_metadata=_convert_launch_metadata)
136| def _convert_dp_to_ep(
137|     peer_dst_ptrs, dst_stride_m, # dst tensors
138|     src_ptr, src_stride_m, src_shape_n,  # src tensor
139|     expt_filter_ptr, expt_filter_stride_m, # expt map
140|     expt_indx_ptr, expt_indx_stride_m, # expt indx
141|     dst_row_indx_ptr, dst_row_indx_stride_m, # gate indx
142|     n_tokens_local,
143|     SRC_RANK: tl.constexpr,
144|     N_EXPT_ACT: tl.constexpr,
145|     N_RANKS: tl.constexpr,
146|     BLOCK: tl.constexpr
147| ):
```
**EN:** Defines function `_convert_dp_to_ep(peer_dst_ptrs, dst_stride_m, src_ptr, src_stride_m, src_shape_n, expt_filter_ptr, expt_filter_stride_m, expt_indx_ptr, expt_indx_stride_m, dst_row_indx_ptr, dst_row_indx_stride_m, n_tokens_local, SRC_RANK, N_EXPT_ACT, N_RANKS, BLOCK)` with decorators `triton.jit(launch_metadata=_convert_l...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `triton.jit`, `tl.program_id`, `tl.arange`, `tl.load`, `tl.sum` to implement its workflow.

**CN:** 定义函数 `_convert_dp_to_ep(peer_dst_ptrs, dst_stride_m, src_ptr, src_stride_m, src_shape_n, expt_filter_ptr, expt_filter_stride_m, expt_indx_ptr, expt_indx_stride_m, dst_row_indx_ptr, dst_row_indx_stride_m, n_tokens_local, SRC_RANK, N_EXPT_ACT, N_RANKS, BLOCK)`，带有装饰器 `triton.jit(launch_metadata=_convert_l...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `triton.jit`, `tl.program_id`, `tl.arange`, `tl.load`, `tl.sum` 来实现其工作流程.

### Block 59 — Lines 148-148 (_convert_dp_to_ep)
```python
148|     pid_m = tl.program_id(0)
```
**EN:** Assigns `pid_m` and calls `tl.program_id`.

**CN:** 将 `pid_m`，并调用 `tl.program_id`.

### Block 60 — Lines 149-149 (_convert_dp_to_ep)
```python
149|     off_m_global = pid_m + n_tokens_local * SRC_RANK
```
**EN:** Assigns `off_m_global` and evaluates `pid_m + n_tokens_local * SRC_RANK`.

**CN:** 将 `off_m_global` and 计算 `pid_m + n_tokens_local * SRC_RANK`.

### Block 61 — Lines 150-150 (_convert_dp_to_ep)
```python
150|     off_m_local = pid_m
```
**EN:** Assigns `off_m_local` and references `pid_m`.

**CN:** 将 `off_m_local` and 引用 `pid_m`.

### Block 62 — Lines 151-151 (_convert_dp_to_ep)
```python
151|     offs_r = tl.arange(0, N_RANKS)
```
**EN:** Assigns `offs_r` and calls `tl.arange`.

**CN:** 将 `offs_r`，并调用 `tl.arange`.

### Block 63 — Lines 152-152 (_convert_dp_to_ep)
```python
152|     offs_e = tl.arange(0, N_EXPT_ACT)
```
**EN:** Assigns `offs_e` and calls `tl.arange`.

**CN:** 将 `offs_e`，并调用 `tl.arange`.

### Block 64 — Lines 153-153 (_convert_dp_to_ep)
```python
153|     offs_n = tl.arange(0, BLOCK)
```
**EN:** Assigns `offs_n` and calls `tl.arange`.

**CN:** 将 `offs_n`，并调用 `tl.arange`.

### Block 65 — Lines 154-154 (_convert_dp_to_ep)
```python
154|     dst_row_indx = tl.load(dst_row_indx_ptr + off_m_global * dst_row_indx_stride_m + offs_e)
```
**EN:** Assigns `dst_row_indx` and calls `tl.load`.

**CN:** 将 `dst_row_indx`，并调用 `tl.load`.

### Block 66 — Lines 155-155 (_convert_dp_to_ep)
```python
155|     expt_indx = tl.load(expt_indx_ptr + off_m_global * expt_indx_stride_m + offs_e)
```
**EN:** Assigns `expt_indx` and calls `tl.load`.

**CN:** 将 `expt_indx`，并调用 `tl.load`.

### Block 67 — Lines 156-156 (_convert_dp_to_ep)
```python
156|     expt_filter_ptr_rows = expt_filter_ptr + offs_r[:, None] * expt_filter_stride_m
```
**EN:** Assigns `expt_filter_ptr_rows` and evaluates `expt_filter_ptr + offs_r[:, None] * expt_filter_stride_m`.

**CN:** 将 `expt_filter_ptr_rows` and 计算 `expt_filter_ptr + offs_r[:, None] * expt_filter_stride_m`.

### Block 68 — Lines 157-157 (_convert_dp_to_ep)
```python
157|     expt_filter = (tl.load(expt_filter_ptr_rows + (expt_indx // 32)[None, :]) >> (expt_indx % 32)) & 1
```
**EN:** Assigns `expt_filter` and evaluates `tl.load(expt_filter_ptr_rows + (expt_indx // 32)[None, :]) >> expt_...`.

**CN:** 将 `expt_filter` and 计算 `tl.load(expt_filter_ptr_rows + (expt_indx // 32)[None, :]) >> expt_...`.

### Block 69 — Lines 158-158 (_convert_dp_to_ep)
```python
158|     expt_ranks = tl.sum(offs_r[:, None] * expt_filter, axis=0)
```
**EN:** Assigns `expt_ranks` and calls `tl.sum`.

**CN:** 将 `expt_ranks`，并调用 `tl.sum`.

### Block 70 — Lines 159-159 (_convert_dp_to_ep)
```python
159|     dst_row_ptrs = tl.zeros((N_EXPT_ACT,), dtype=tl.int64)
```
**EN:** Assigns `dst_row_ptrs` and calls `tl.zeros`.

**CN:** 将 `dst_row_ptrs`，并调用 `tl.zeros`.

### Block 71 — Lines 160-162 (_convert_dp_to_ep)
```python
160|     for dst_rank in tl.static_range(N_RANKS):
161|         peer_dst_ptr = peer_dst_ptrs[dst_rank].to(tl.int64, bitcast=True)
162|         dst_row_ptrs = tl.where(dst_rank == expt_ranks, peer_dst_ptr, dst_row_ptrs)
```
**EN:** Loops over `tl.static_range(N_RANKS)` with target `dst_rank`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `tl.static_range(N_RANKS)` ，目标变量为 `dst_rank`. 循环体主要准备中间值; 准备中间值.

### Block 72 — Lines 163-163 (_convert_dp_to_ep)
```python
163|     dst_row_ptrs = dst_row_ptrs.to(src_ptr.dtype, bitcast=True)
```
**EN:** Assigns `dst_row_ptrs` and calls `dst_row_ptrs.to`.

**CN:** 将 `dst_row_ptrs`，并调用 `dst_row_ptrs.to`.

### Block 73 — Lines 164-164 (_convert_dp_to_ep)
```python
164|     dst_row_ptrs = tl.multiple_of(dst_row_ptrs, 16)
```
**EN:** Assigns `dst_row_ptrs` and calls `tl.multiple_of`.

**CN:** 将 `dst_row_ptrs`，并调用 `tl.multiple_of`.

### Block 74 — Lines 165-165 (_convert_dp_to_ep)
```python
165|     dst_row_ptrs = dst_row_ptrs + dst_row_indx * dst_stride_m
```
**EN:** Assigns `dst_row_ptrs` and evaluates `dst_row_ptrs + dst_row_indx * dst_stride_m`.

**CN:** 将 `dst_row_ptrs` and 计算 `dst_row_ptrs + dst_row_indx * dst_stride_m`.

### Block 75 — Lines 166-166 (_convert_dp_to_ep)
```python
166|     dst_ptrs = dst_row_ptrs[:, None] + offs_n[None, :]
```
**EN:** Assigns `dst_ptrs` and evaluates `dst_row_ptrs[:, None] + offs_n[None, :]`.

**CN:** 将 `dst_ptrs` and 计算 `dst_row_ptrs[:, None] + offs_n[None, :]`.

### Block 76 — Lines 167-167 (_convert_dp_to_ep)
```python
167|     src_ptrs = src_ptr + off_m_local * src_stride_m + offs_n
```
**EN:** Assigns `src_ptrs` and evaluates `src_ptr + off_m_local * src_stride_m + offs_n`.

**CN:** 将 `src_ptrs` and 计算 `src_ptr + off_m_local * src_stride_m + offs_n`.

### Block 77 — Lines 168-175 (_convert_dp_to_ep)
```python
168|     for start_n in range(0, src_shape_n, BLOCK):
169|         mask_n = start_n + offs_n < src_shape_n
170|         src = tl.load(src_ptrs, mask=mask_n, other=0.0)
171|         tl.store(dst_ptrs, src[None, :], mask=mask_n[None, :])
172|         src_ptrs += BLOCK
173|         dst_ptrs += BLOCK
174| 
175| 
```
**EN:** Loops over `range(0, src_shape_n, BLOCK)` with target `start_n`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `range(0, src_shape_n, BLOCK)` ，目标变量为 `start_n`. 循环体主要准备中间值; 准备中间值.

### Block 78 — Lines 176-176 (convert_dp_to_ep)
```python
176| def convert_dp_to_ep(src, expt_assignment, expt_indx, gate_indx, symm_mem_pool: SymmetricMemoryPool):
```
**EN:** Defines function `convert_dp_to_ep(src, expt_assignment, expt_indx, gate_indx, symm_mem_pool)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `all`, `symm_mem_pool.make_empty`, `_convert_dp_to_ep[grid]`, `hdl.barrier`, `expt_bitmask.size` to implement its workflow.

**CN:** 定义函数 `convert_dp_to_ep(src, expt_assignment, expt_indx, gate_indx, symm_mem_pool)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `all`, `symm_mem_pool.make_empty`, `_convert_dp_to_ep[grid]`, `hdl.barrier`, `expt_bitmask.size` 来实现其工作流程.

### Block 79 — Lines 177-178 (convert_dp_to_ep)
```python
177|     expt_bitmask = expt_assignment.expt_bitmask
178|     # extract problem dimensions
```
**EN:** Assigns `expt_bitmask` and references `expt_assignment.expt_bitmask`.

**CN:** 将 `expt_bitmask` and 引用 `expt_assignment.expt_bitmask`.

### Block 80 — Lines 179-179 (convert_dp_to_ep)
```python
179|     device = src.device
```
**EN:** Assigns `device` and references `src.device`.

**CN:** 将 `device` and 引用 `src.device`.

### Block 81 — Lines 180-180 (convert_dp_to_ep)
```python
180|     n_tokens_local, d_model = src.shape
```
**EN:** Assigns `n_tokens_local`, `d_model` and references `src.shape`.

**CN:** 将 `n_tokens_local`, `d_model` and 引用 `src.shape`.

### Block 82 — Lines 181-182 (convert_dp_to_ep)
```python
181|     n_tokens_global, n_expt_act = expt_indx.shape
182|     # validate invariants
```
**EN:** Assigns `n_tokens_global`, `n_expt_act` and references `expt_indx.shape`.

**CN:** 将 `n_tokens_global`, `n_expt_act` and 引用 `expt_indx.shape`.

### Block 83 — Lines 183-183 (convert_dp_to_ep)
```python
183|     assert symm_mem_pool.mesh.world_size == expt_bitmask.size(0)
```
**EN:** Asserts `symm_mem_pool.mesh.world_size == expt_bitmask.size(0)` to enforce invariants.

**CN:** 断言 `symm_mem_pool.mesh.world_size == expt_bitmask.size(0)` 以确保不变量成立。

### Block 84 — Lines 184-184 (convert_dp_to_ep)
```python
184|     assert all(t.device == device for t in [expt_bitmask, expt_indx, gate_indx]), "all tensors must be on the same device"
```
**EN:** Asserts `all((t.device == device for t in [expt_bitmask, expt_indx, gate_indx]))` to enforce invariants.

**CN:** 断言 `all((t.device == device for t in [expt_bitmask, expt_indx, gate_indx]))` 以确保不变量成立。

### Block 85 — Lines 185-185 (convert_dp_to_ep)
```python
185|     assert expt_bitmask.dtype == torch.int32, "expt_bitmask must be int32 bitmask words"
```
**EN:** Asserts `expt_bitmask.dtype == torch.int32` to enforce invariants.

**CN:** 断言 `expt_bitmask.dtype == torch.int32` 以确保不变量成立。

### Block 86 — Lines 186-186 (convert_dp_to_ep)
```python
186|     assert expt_bitmask.stride(-1) == 1 and expt_indx.stride(-1) == 1 and gate_indx.stride(-1) == 1
```
**EN:** Asserts `expt_bitmask.stride(-1) == 1 and expt_indx.stride(-1) == 1 and (gate_indx.stride(-1) == 1)` to enforce invariants.

**CN:** 断言 `expt_bitmask.stride(-1) == 1 and expt_indx.stride(-1) == 1 and (gate_indx.stride(-1) == 1)` 以确保不变量成立。

### Block 87 — Lines 187-187 (convert_dp_to_ep)
```python
187|     assert n_tokens_local * symm_mem_pool.mesh.world_size <= n_tokens_global
```
**EN:** Asserts `n_tokens_local * symm_mem_pool.mesh.world_size <= n_tokens_global` to enforce invariants.

**CN:** 断言 `n_tokens_local * symm_mem_pool.mesh.world_size <= n_tokens_global` 以确保不变量成立。

### Block 88 — Lines 188-192 (convert_dp_to_ep)
```python
188|     peer_bufs = symm_mem_pool.make_empty(
189|         region="dp_to_ep",
190|         shape=(n_tokens_global * n_expt_act, d_model),
191|         dtype=src.dtype,
192|     )
```
**EN:** Assigns `peer_bufs` and calls `symm_mem_pool.make_empty`.

**CN:** 将 `peer_bufs`，并调用 `symm_mem_pool.make_empty`.

### Block 89 — Lines 193-193 (convert_dp_to_ep)
```python
193|     dst_local = peer_bufs[symm_mem_pool.mesh.local_rank]
```
**EN:** Assigns `dst_local` and evaluates `peer_bufs[symm_mem_pool.mesh.local_rank]`.

**CN:** 将 `dst_local` and 计算 `peer_bufs[symm_mem_pool.mesh.local_rank]`.

### Block 90 — Lines 194-195 (convert_dp_to_ep)
```python
194|     hdl = symm_mem_pool.hdl
195|     # launch kernel
```
**EN:** Assigns `hdl` and references `symm_mem_pool.hdl`.

**CN:** 将 `hdl` and 引用 `symm_mem_pool.hdl`.

### Block 91 — Lines 196-196 (convert_dp_to_ep)
```python
196|     BLOCK = 512
```
**EN:** Assigns `BLOCK` and stores constant `512`.

**CN:** 将 `BLOCK` and 保存常量 `512`.

### Block 92 — Lines 197-197 (convert_dp_to_ep)
```python
197|     grid = (n_tokens_local,)
```
**EN:** Assigns `grid` and builds a tuple.

**CN:** 将 `grid` and 构造一个元组.

### Block 93 — Lines 198-209 (convert_dp_to_ep)
```python
198|     _convert_dp_to_ep[grid](
199|         tuple(peer_bufs), dst_local.stride(0),
200|         src, src.stride(0), src.shape[1],
201|         expt_bitmask, expt_bitmask.stride(0),
202|         expt_indx, expt_indx.stride(0),
203|         gate_indx, n_expt_act,
204|         n_tokens_local,
205|         SRC_RANK=symm_mem_pool.mesh.local_rank,
206|         N_EXPT_ACT=n_expt_act,
207|         N_RANKS=symm_mem_pool.mesh.world_size,
208|         BLOCK=BLOCK,
209|     )
```
**EN:** Calls `_convert_dp_to_ep[grid]` for side effects, registration, or validation.

**CN:** 调用 `_convert_dp_to_ep[grid]` ，用于副作用、注册或校验。

### Block 94 — Lines 210-210 (convert_dp_to_ep)
```python
210|     hdl.barrier(channel=0)
```
**EN:** Calls `hdl.barrier` for side effects, registration, or validation.

**CN:** 调用 `hdl.barrier` ，用于副作用、注册或校验。

### Block 95 — Lines 211-215 (convert_dp_to_ep)
```python
211|     return dst_local
212| 
213| 
214| # ------------------------------------------------------------
215| 
```
**EN:** Returns `dst_local`.

**CN:** 返回 `dst_local`.

### Block 96 — Lines 216-228 (_convert_ep_to_dp)
```python
216| @triton.jit(launch_metadata=_convert_launch_metadata)
217| def _convert_ep_to_dp(
218|     peer_dst_ptrs, dst_stride_m, # dst tensors
219|     src_ptr, src_stride_m, src_shape_n, # src tensor
220|     expt_filter_ptr, expt_filter_stride_m, # expt map
221|     expt_indx_ptr,  # expt indx
222|     dst_row_indx_ptr, # topk indx
223|     n_tokens_local,
224|     BLOCK: tl.constexpr,
225|     SRC_RANK: tl.constexpr,
226|     N_RANKS: tl.constexpr
227| ):
228|     # token offset
```
**EN:** Defines function `_convert_ep_to_dp(peer_dst_ptrs, dst_stride_m, src_ptr, src_stride_m, src_shape_n, expt_filter_ptr, expt_filter_stride_m, expt_indx_ptr, dst_row_indx_ptr, n_tokens_local, BLOCK, SRC_RANK, N_RANKS)` with decorators `triton.jit(launch_metadata=_convert_l...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `triton.jit`, `tl.program_id`, `tl.load`, `tl.zeros((1,), dtype=tl.int64).item`, `tl.static_range` to implement its workflow.

**CN:** 定义函数 `_convert_ep_to_dp(peer_dst_ptrs, dst_stride_m, src_ptr, src_stride_m, src_shape_n, expt_filter_ptr, expt_filter_stride_m, expt_indx_ptr, dst_row_indx_ptr, n_tokens_local, BLOCK, SRC_RANK, N_RANKS)`，带有装饰器 `triton.jit(launch_metadata=_convert_l...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `triton.jit`, `tl.program_id`, `tl.load`, `tl.zeros((1,), dtype=tl.int64).item`, `tl.static_range` 来实现其工作流程.

### Block 97 — Lines 229-230 (_convert_ep_to_dp)
```python
229|     pid_m = tl.program_id(0)
230|     # destination base pointer
```
**EN:** Assigns `pid_m` and calls `tl.program_id`.

**CN:** 将 `pid_m`，并调用 `tl.program_id`.

### Block 98 — Lines 231-231 (_convert_ep_to_dp)
```python
231|     dst_indx_global = tl.load(dst_row_indx_ptr + pid_m)
```
**EN:** Assigns `dst_indx_global` and calls `tl.load`.

**CN:** 将 `dst_indx_global`，并调用 `tl.load`.

### Block 99 — Lines 232-232 (_convert_ep_to_dp)
```python
232|     dst_rank = dst_indx_global // n_tokens_local
```
**EN:** Assigns `dst_rank` and evaluates `dst_indx_global // n_tokens_local`.

**CN:** 将 `dst_rank` and 计算 `dst_indx_global // n_tokens_local`.

### Block 100 — Lines 233-233 (_convert_ep_to_dp)
```python
233|     dst_ptr = tl.zeros((1,), dtype=tl.int64).item()
```
**EN:** Assigns `dst_ptr` and calls `tl.zeros((1,), dtype=tl.int64).item`.

**CN:** 将 `dst_ptr`，并调用 `tl.zeros((1,), dtype=tl.int64).item`.

### Block 101 — Lines 234-236 (_convert_ep_to_dp)
```python
234|     for i in tl.static_range(N_RANKS):
235|         if dst_rank == i:
236|             dst_ptr = peer_dst_ptrs[i].to(tl.int64, bitcast=True)
```
**EN:** Loops over `tl.static_range(N_RANKS)` with target `i`. The loop body mainly branches on runtime conditions.

**CN:** 遍历 `tl.static_range(N_RANKS)` ，目标变量为 `i`. 循环体主要根据运行时条件分支.

### Block 102 — Lines 237-238 (_convert_ep_to_dp)
```python
237|     dst_ptr = tl.multiple_of(dst_ptr.to(src_ptr.dtype), 16)
238|     # input / output pointers
```
**EN:** Assigns `dst_ptr` and calls `tl.multiple_of`.

**CN:** 将 `dst_ptr`，并调用 `tl.multiple_of`.

### Block 103 — Lines 239-239 (_convert_ep_to_dp)
```python
239|     dst_expt_indx = tl.load(expt_indx_ptr + dst_indx_global)
```
**EN:** Assigns `dst_expt_indx` and calls `tl.load`.

**CN:** 将 `dst_expt_indx`，并调用 `tl.load`.

### Block 104 — Lines 240-240 (_convert_ep_to_dp)
```python
240|     expt_filter_ptr = expt_filter_ptr + SRC_RANK * expt_filter_stride_m
```
**EN:** Assigns `expt_filter_ptr` and evaluates `expt_filter_ptr + SRC_RANK * expt_filter_stride_m`.

**CN:** 将 `expt_filter_ptr` and 计算 `expt_filter_ptr + SRC_RANK * expt_filter_stride_m`.

### Block 105 — Lines 241-241 (_convert_ep_to_dp)
```python
241|     has_dst_expt = (tl.load(expt_filter_ptr + dst_expt_indx // 32) >> (dst_expt_indx % 32)) & 1
```
**EN:** Assigns `has_dst_expt` and evaluates `tl.load(expt_filter_ptr + dst_expt_indx // 32) >> dst_expt_indx % 3...`.

**CN:** 将 `has_dst_expt` and 计算 `tl.load(expt_filter_ptr + dst_expt_indx // 32) >> dst_expt_indx % 3...`.

### Block 106 — Lines 242-243 (_convert_ep_to_dp)
```python
242|     if not has_dst_expt.to(tl.int1):
243|         return
```
**EN:** Checks `not has_dst_expt.to(tl.int1)`. The true branch mainly returns the computed result.

**CN:** 检查 `not has_dst_expt.to(tl.int1)`. 真分支主要返回计算结果.

### Block 107 — Lines 244-244 (_convert_ep_to_dp)
```python
244|     dst_indx_local = dst_indx_global - dst_rank * n_tokens_local
```
**EN:** Assigns `dst_indx_local` and evaluates `dst_indx_global - dst_rank * n_tokens_local`.

**CN:** 将 `dst_indx_local` and 计算 `dst_indx_global - dst_rank * n_tokens_local`.

### Block 108 — Lines 245-245 (_convert_ep_to_dp)
```python
245|     offs_n = tl.arange(0, BLOCK)
```
**EN:** Assigns `offs_n` and calls `tl.arange`.

**CN:** 将 `offs_n`，并调用 `tl.arange`.

### Block 109 — Lines 246-246 (_convert_ep_to_dp)
```python
246|     dst_ptrs = dst_ptr + dst_indx_local * dst_stride_m + offs_n
```
**EN:** Assigns `dst_ptrs` and evaluates `dst_ptr + dst_indx_local * dst_stride_m + offs_n`.

**CN:** 将 `dst_ptrs` and 计算 `dst_ptr + dst_indx_local * dst_stride_m + offs_n`.

### Block 110 — Lines 247-247 (_convert_ep_to_dp)
```python
247|     src_ptrs = src_ptr + pid_m * src_stride_m + offs_n
```
**EN:** Assigns `src_ptrs` and evaluates `src_ptr + pid_m * src_stride_m + offs_n`.

**CN:** 将 `src_ptrs` and 计算 `src_ptr + pid_m * src_stride_m + offs_n`.

### Block 111 — Lines 248-255 (_convert_ep_to_dp)
```python
248|     for start_n in range(0, src_shape_n, BLOCK):
249|         mask_n = start_n + offs_n < src_shape_n
250|         src = tl.load(src_ptrs, mask=mask_n, other=0.0)
251|         tl.store(dst_ptrs, src, mask=mask_n)
252|         src_ptrs += BLOCK
253|         dst_ptrs += BLOCK
254| 
255| 
```
**EN:** Loops over `range(0, src_shape_n, BLOCK)` with target `start_n`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `range(0, src_shape_n, BLOCK)` ，目标变量为 `start_n`. 循环体主要准备中间值; 准备中间值.

### Block 112 — Lines 256-256 (convert_ep_to_dp)
```python
256| def convert_ep_to_dp(src, expt_assignment, expt_indx, topk_indx, symm_mem_pool: SymmetricMemoryPool):
```
**EN:** Defines function `convert_ep_to_dp(src, expt_assignment, expt_indx, topk_indx, symm_mem_pool)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `symm_mem_pool.make_empty`, `_convert_ep_to_dp[grid]`, `hdl.barrier`, `tuple`, `dst_local.stride` to implement its workflow.

**CN:** 定义函数 `convert_ep_to_dp(src, expt_assignment, expt_indx, topk_indx, symm_mem_pool)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `symm_mem_pool.make_empty`, `_convert_ep_to_dp[grid]`, `hdl.barrier`, `tuple`, `dst_local.stride` 来实现其工作流程.

### Block 113 — Lines 257-258 (convert_ep_to_dp)
```python
257|     expt_bitmask = expt_assignment.expt_bitmask
258|     # extract problem dimensions
```
**EN:** Assigns `expt_bitmask` and references `expt_assignment.expt_bitmask`.

**CN:** 将 `expt_bitmask` and 引用 `expt_assignment.expt_bitmask`.

### Block 114 — Lines 259-259 (convert_ep_to_dp)
```python
259|     n_tokens_global, d_model = src.shape
```
**EN:** Assigns `n_tokens_global`, `d_model` and references `src.shape`.

**CN:** 将 `n_tokens_global`, `d_model` and 引用 `src.shape`.

### Block 115 — Lines 260-260 (convert_ep_to_dp)
```python
260|     n_tokens_local = n_tokens_global // symm_mem_pool.mesh.world_size
```
**EN:** Assigns `n_tokens_local` and evaluates `n_tokens_global // symm_mem_pool.mesh.world_size`.

**CN:** 将 `n_tokens_local` and 计算 `n_tokens_global // symm_mem_pool.mesh.world_size`.

### Block 116 — Lines 261-265 (convert_ep_to_dp)
```python
261|     peer_bufs = symm_mem_pool.make_empty(
262|         region="ep_to_dp",
263|         shape=(n_tokens_local, d_model),
264|         dtype=src.dtype,
265|     )
```
**EN:** Assigns `peer_bufs` and calls `symm_mem_pool.make_empty`.

**CN:** 将 `peer_bufs`，并调用 `symm_mem_pool.make_empty`.

### Block 117 — Lines 266-266 (convert_ep_to_dp)
```python
266|     dst_local = peer_bufs[symm_mem_pool.mesh.local_rank]
```
**EN:** Assigns `dst_local` and evaluates `peer_bufs[symm_mem_pool.mesh.local_rank]`.

**CN:** 将 `dst_local` and 计算 `peer_bufs[symm_mem_pool.mesh.local_rank]`.

### Block 118 — Lines 267-268 (convert_ep_to_dp)
```python
267|     hdl = symm_mem_pool.hdl
268|     # launch kernel
```
**EN:** Assigns `hdl` and references `symm_mem_pool.hdl`.

**CN:** 将 `hdl` and 引用 `symm_mem_pool.hdl`.

### Block 119 — Lines 269-269 (convert_ep_to_dp)
```python
269|     BLOCK = 512
```
**EN:** Assigns `BLOCK` and stores constant `512`.

**CN:** 将 `BLOCK` and 保存常量 `512`.

### Block 120 — Lines 270-270 (convert_ep_to_dp)
```python
270|     grid = (n_tokens_global,)
```
**EN:** Assigns `grid` and builds a tuple.

**CN:** 将 `grid` and 构造一个元组.

### Block 121 — Lines 271-281 (convert_ep_to_dp)
```python
271|     _convert_ep_to_dp[grid](
272|         tuple(peer_bufs), dst_local.stride(0),
273|         src, src.stride(0), src.shape[1],
274|         expt_bitmask, expt_bitmask.stride(0),
275|         expt_indx,
276|         topk_indx,
277|         n_tokens_local,
278|         BLOCK=BLOCK,
279|         SRC_RANK=symm_mem_pool.mesh.local_rank,
280|         N_RANKS=symm_mem_pool.mesh.world_size,
281|     )
```
**EN:** Calls `_convert_ep_to_dp[grid]` for side effects, registration, or validation.

**CN:** 调用 `_convert_ep_to_dp[grid]` ，用于副作用、注册或校验。

### Block 122 — Lines 282-282 (convert_ep_to_dp)
```python
282|     hdl.barrier(channel=0)
```
**EN:** Calls `hdl.barrier` for side effects, registration, or validation.

**CN:** 调用 `hdl.barrier` ，用于副作用、注册或校验。

### Block 123 — Lines 283-283 (convert_ep_to_dp)
```python
283|     return dst_local
```
**EN:** Returns `dst_local`.

**CN:** 返回 `dst_local`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `ExptAssignment`, `make_expt_dict_uniform`, `make_expt_dict_random`, `make_expt_assignment`, `_convert_launch_metadata`, `_convert_dp_to_ep`, `convert_dp_to_ep`, `_convert_ep_to_dp`.
  **CN:** 主要符号：`ExptAssignment`, `make_expt_dict_uniform`, `make_expt_dict_random`, `make_expt_assignment`, `_convert_launch_metadata`, `_convert_dp_to_ep`, `convert_dp_to_ep`, `_convert_ep_to_dp`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Distributed execution concepts such as meshes, ranks, or shard mapping appear in this file.
  **CN:** 该文件涉及 mesh、rank 或分片映射等分布式执行概念。
- **EN:** Top-k selection and, where relevant, gradient propagation are key operations here.
  **CN:** 这里的关键操作是 Top-k 选择以及相关的梯度传播。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`, `triton`, `triton.language`, `random`, `dataclasses (dataclass)`.
  **CN:** 外部模块：`torch`, `triton`, `triton.language`, `random`, `dataclasses (dataclass)`。
- **EN:** Internal modules: `.distributed_details.mesh (SymmetricMemoryPool)`.
  **CN:** 内部模块：`.distributed_details.mesh (SymmetricMemoryPool)`。

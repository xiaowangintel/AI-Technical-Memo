# test_distributed.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_distributed.py`
- **Purpose / 用途:** Pytest coverage for test distributed; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test distributed 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-17 (module)
```python
 1| import contextlib
 2| import os
 3| import socket
 4| 
 5| import torch
 6| import torch.distributed as dist
 7| import torch.multiprocessing as mp
 8| import triton
 9| from triton_kernels.distributed import convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_dict_random, make_expt_assignment, SymmetricMemoryPool
10| from triton_kernels.distributed_details.mesh import Mesh
11| from triton_kernels.reduce import reduce
12| from triton_kernels.topk import topk
13| from triton_kernels.matmul import matmul
14| from triton_kernels.tensor import make_ragged_tensor_metadata, remap_ragged_tensor_metadata
15| import pytest
16| 
17| 
```
**EN:** This block imports `contextlib`, `os`, `socket`, `torch`, `torch.distributed`, `torch.multiprocessing`, `triton`, `triton_kernels.distributed (convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_dict_random)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `contextlib`, `os`, `socket`, `torch`, `torch.distributed`, `torch.multiprocessing`, `triton`, `triton_kernels.distributed (convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_dict_random)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 18-18 (_make_expt_dict_for_mode)
```python
18| def _make_expt_dict_for_mode(n_shards, n_expts_tot, affinity_mode):
```
**EN:** Defines function `_make_expt_dict_for_mode(n_shards, n_expts_tot, affinity_mode)` for this module. The body mainly prepares intermediate values; handles exceptional paths. It uses calls such as `factories[affinity_mode]`, `ValueError` to implement its workflow.

**CN:** 定义函数 `_make_expt_dict_for_mode(n_shards, n_expts_tot, affinity_mode)`，供本模块使用. 主体主要准备中间值; 处理异常路径. 其中会调用 `factories[affinity_mode]`, `ValueError` 来实现其工作流程.

### Block 3 — Lines 19-22 (_make_expt_dict_for_mode)
```python
19|     factories = {
20|         "uniform": make_expt_dict_uniform,
21|         "random": make_expt_dict_random,
22|     }
```
**EN:** Assigns `factories` and builds a dictionary.

**CN:** 将 `factories` and 构造一个字典.

### Block 4 — Lines 23-28 (_make_expt_dict_for_mode)
```python
23|     try:
24|         return factories[affinity_mode](n_shards, n_expts_tot)
25|     except KeyError as exc:
26|         raise ValueError(f"Unknown affinity mode: {affinity_mode}") from exc
27| 
28| 
```
**EN:** Wraps code that mainly returns the computed result with exception/finally handling.

**CN:** 包装代码 that mainly 返回计算结果 以及异常/收尾处理.

### Block 5 — Lines 29-29 (_make_y_indx_for_mode)
```python
29| def _make_y_indx_for_mode(n_tokens_global, n_expts_tot, n_expts_act, n_shards, affinity_mode, dev):
```
**EN:** Defines function `_make_y_indx_for_mode(n_tokens_global, n_expts_tot, n_expts_act, n_shards, affinity_mode, dev)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; returns the computed result. It uses calls such as `torch.arange`, `(shard_order * expts_per_rank + intra...`, `round_robin_indx.unsqueeze(0).expand(...`, `ValueError`, `round_robin_indx.unsqueeze(0).expand` to implement its workflow.

**CN:** 定义函数 `_make_y_indx_for_mode(n_tokens_global, n_expts_tot, n_expts_act, n_shards, affinity_mode, dev)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 返回计算结果. 其中会调用 `torch.arange`, `(shard_order * expts_per_rank + intra...`, `round_robin_indx.unsqueeze(0).expand(...`, `ValueError`, `round_robin_indx.unsqueeze(0).expand` 来实现其工作流程.

### Block 6 — Lines 30-30 (_make_y_indx_for_mode)
```python
30|     y_indx_global = None
```
**EN:** Assigns `y_indx_global` and stores constant `None`.

**CN:** 将 `y_indx_global` and 保存常量 `None`.

### Block 7 — Lines 31-42 (_make_y_indx_for_mode)
```python
31|     if affinity_mode == "uniform":
32|         if n_expts_tot % n_shards != 0:
33|             raise ValueError("uniform affinity requires experts evenly divisible by shards")
34|         expts_per_rank = n_expts_tot // n_shards
35|         rounds = (n_expts_act + n_shards - 1) // n_shards
36|         if rounds > expts_per_rank:
37|             raise ValueError("round-robin selection exceeds experts available per shard")
38|         order = torch.arange(n_expts_act, device=dev, dtype=torch.int32)
39|         shard_order = order % n_shards
40|         intra_shard = order // n_shards
41|         round_robin_indx = (shard_order * expts_per_rank + intra_shard).to(torch.int16)
42|         y_indx_global = round_robin_indx.unsqueeze(0).expand(n_tokens_global, -1).contiguous()
```
**EN:** Checks `affinity_mode == 'uniform'`. The true branch mainly branches on runtime conditions; prepares intermediate values.

**CN:** 检查 `affinity_mode == 'uniform'`. 真分支主要根据运行时条件分支; 准备中间值.

### Block 8 — Lines 43-50 (_make_y_indx_for_mode)
```python
43|     return y_indx_global
44| 
45| 
46| # ------------------------------------------------------------
47| # fixture
48| # ------------------------------------------------------------
49| 
50| 
```
**EN:** Returns `y_indx_global`.

**CN:** 返回 `y_indx_global`.

### Block 9 — Lines 51-51 (_get_free_tcp_port)
```python
51| def _get_free_tcp_port():
```
**EN:** Defines function `_get_free_tcp_port()` for this module. The body mainly uses contextual resources. It uses calls such as `contextlib.closing`, `sock.bind`, `socket.socket`, `sock.getsockname` to implement its workflow.

**CN:** 定义函数 `_get_free_tcp_port()`，供本模块使用. 主体主要使用上下文资源. 其中会调用 `contextlib.closing`, `sock.bind`, `socket.socket`, `sock.getsockname` 来实现其工作流程.

### Block 10 — Lines 52-56 (_get_free_tcp_port)
```python
52|     with contextlib.closing(socket.socket(socket.AF_INET, socket.SOCK_STREAM)) as sock:
53|         sock.bind(("127.0.0.1", 0))
54|         return sock.getsockname()[1]
55| 
56| 
```
**EN:** Uses context manager(s) `contextlib.closing(socket.socket(socket.AF...` around code that mainly invokes `sock.bind`; returns the computed result.

**CN:** 使用上下文管理器 `contextlib.closing(socket.socket(socket.AF...` ，其中代码主要invokes `sock.bind`; 返回计算结果.

### Block 11 — Lines 57-57 (_distributed_worker)
```python
57| def _distributed_worker(rank, fn, world_size, kwargs):
```
**EN:** Defines function `_distributed_worker(rank, fn, world_size, kwargs)` for this module. The body mainly prepares intermediate values; invokes `dist.init_process_group`; invokes `torch.cuda.set_device`. It uses calls such as `dist.init_process_group`, `torch.cuda.set_device`, `fn`, `dist.barrier`, `dist.destroy_process_group` to implement its workflow.

**CN:** 定义函数 `_distributed_worker(rank, fn, world_size, kwargs)`，供本模块使用. 主体主要准备中间值; invokes `dist.init_process_group`; invokes `torch.cuda.set_device`. 其中会调用 `dist.init_process_group`, `torch.cuda.set_device`, `fn`, `dist.barrier`, `dist.destroy_process_group` 来实现其工作流程.

### Block 12 — Lines 58-58 (_distributed_worker)
```python
58|     dev = f"cuda:{rank}"
```
**EN:** Assigns `dev` and evaluates `f'cuda:{rank}'`.

**CN:** 将 `dev` and 计算 `f'cuda:{rank}'`.

### Block 13 — Lines 59-59 (_distributed_worker)
```python
59|     dist.init_process_group(backend="nccl", rank=rank, world_size=world_size, device_id=torch.device(dev))
```
**EN:** Calls `dist.init_process_group` for side effects, registration, or validation.

**CN:** 调用 `dist.init_process_group` ，用于副作用、注册或校验。

### Block 14 — Lines 60-60 (_distributed_worker)
```python
60|     torch.cuda.set_device(dev)
```
**EN:** Calls `torch.cuda.set_device` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.set_device` ，用于副作用、注册或校验。

### Block 15 — Lines 61-67 (_distributed_worker)
```python
61|     try:
62|         fn(rank=rank, world_size=world_size, **kwargs)
63|         dist.barrier()
64|     finally:
65|         dist.destroy_process_group()
66| 
67| 
```
**EN:** Wraps code that mainly invokes `fn`; invokes `dist.barrier` with exception/finally handling.

**CN:** 包装代码 that mainly invokes `fn`; invokes `dist.barrier` 以及异常/收尾处理.

### Block 16 — Lines 68-69 (distributed_launcher)
```python
68| @pytest.fixture
69| def distributed_launcher(request):
```
**EN:** Defines function `distributed_launcher(request)` with decorators `pytest.fixture` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `getattr`, `_get_free_tcp_port`, `str`, `os.environ.setdefault`, `torch.cuda.is_available` to implement its workflow.

**CN:** 定义函数 `distributed_launcher(request)`，带有装饰器 `pytest.fixture`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `getattr`, `_get_free_tcp_port`, `str`, `os.environ.setdefault`, `torch.cuda.is_available` 来实现其工作流程.

### Block 17 — Lines 70-70 (distributed_launcher)
```python
70|     n_gpus = getattr(request, "param", None)
```
**EN:** Assigns `n_gpus` and calls `getattr`.

**CN:** 将 `n_gpus`，并调用 `getattr`.

### Block 18 — Lines 71-72 (distributed_launcher)
```python
71|     if not torch.cuda.is_available():
72|         pytest.skip("CUDA required for distributed GPU test")
```
**EN:** Checks `not torch.cuda.is_available()`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `not torch.cuda.is_available()`. 真分支主要invokes `pytest.skip`.

### Block 19 — Lines 73-75 (distributed_launcher)
```python
73|     if torch.cuda.device_count() < n_gpus:
74|         pytest.skip(f"requires up to {n_gpus} CUDA devices, found {torch.cuda.device_count()}")
75| 
```
**EN:** Checks `torch.cuda.device_count() < n_gpus`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `torch.cuda.device_count() < n_gpus`. 真分支主要invokes `pytest.skip`.

### Block 20 — Lines 76-77 (distributed_launcher)
```python
76|     master_port = _get_free_tcp_port()
77| 
```
**EN:** Assigns `master_port` and calls `_get_free_tcp_port`.

**CN:** 将 `master_port`，并调用 `_get_free_tcp_port`.

### Block 21 — Lines 78-78 (distributed_launcher)
```python
78|     os.environ["WORLD_SIZE"] = str(n_gpus)
```
**EN:** Assigns `os.environ['WORLD_SIZE']` and calls `str`.

**CN:** 将 `os.environ['WORLD_SIZE']`，并调用 `str`.

### Block 22 — Lines 79-79 (distributed_launcher)
```python
79|     os.environ.setdefault("MASTER_ADDR", "127.0.0.1")
```
**EN:** Calls `os.environ.setdefault` for side effects, registration, or validation.

**CN:** 调用 `os.environ.setdefault` ，用于副作用、注册或校验。

### Block 23 — Lines 80-81 (distributed_launcher)
```python
80|     os.environ.setdefault("MASTER_PORT", str(master_port))
81| 
```
**EN:** Calls `os.environ.setdefault` for side effects, registration, or validation.

**CN:** 调用 `os.environ.setdefault` ，用于副作用、注册或校验。

### Block 24 — Lines 82-82 (launch)
```python
82|     def launch(fn, **kwargs):
```
**EN:** Defines function `launch(fn, **kwargs)` for this module. The body mainly invokes `mp.spawn`. It uses calls such as `mp.spawn` to implement its workflow.

**CN:** 定义函数 `launch(fn, **kwargs)`，供本模块使用. 主体主要invokes `mp.spawn`. 其中会调用 `mp.spawn` 来实现其工作流程.

### Block 25 — Lines 83-89 (launch)
```python
83|         mp.spawn(
84|             _distributed_worker,
85|             args=(fn, n_gpus, kwargs),
86|             nprocs=n_gpus,
87|             join=True,
88|         )
89| 
```
**EN:** Calls `mp.spawn` for side effects, registration, or validation.

**CN:** 调用 `mp.spawn` ，用于副作用、注册或校验。

### Block 26 — Lines 90-90 (distributed_launcher)
```python
90|     launch.world_size = n_gpus
```
**EN:** Assigns `launch.world_size` and references `n_gpus`.

**CN:** 将 `launch.world_size` and 引用 `n_gpus`.

### Block 27 — Lines 91-98 (distributed_launcher)
```python
91|     return launch
92| 
93| 
94| # ------------------------------------------------------------
95| # expt assignment
96| # ------------------------------------------------------------
97| 
98| 
```
**EN:** Returns `launch`.

**CN:** 返回 `launch`.

### Block 28 — Lines 99-101 (test_make_expt_assignment)
```python
 99| @pytest.mark.parametrize("n_expts_shard, n_expts_tot", [(8, 512), (16, 64)])
100| @pytest.mark.parametrize("affinity_mode", ["uniform", "random"])
101| def test_make_expt_assignment(n_expts_shard, n_expts_tot, affinity_mode):
```
**EN:** Defines function `test_make_expt_assignment(n_expts_shard, n_expts_tot, affinity_mode)` with decorators `pytest.mark.parametrize('n_expts_shar...`, `pytest.mark.parametrize('affinity_mod...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `_make_expt_dict_for_mode`, `make_expt_assignment`, `range`, `bitmask.T.flatten().nonzero()[:, 0].t...` to implement its workflow.

**CN:** 定义函数 `test_make_expt_assignment(n_expts_shard, n_expts_tot, affinity_mode)`，带有装饰器 `pytest.mark.parametrize('n_expts_shar...`, `pytest.mark.parametrize('affinity_mod...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `_make_expt_dict_for_mode`, `make_expt_assignment`, `range`, `bitmask.T.flatten().nonzero()[:, 0].t...` 来实现其工作流程.

### Block 29 — Lines 102-102 (test_make_expt_assignment)
```python
102|     device = "cuda"
```
**EN:** Assigns `device` and stores a string literal or docstring.

**CN:** 将 `device` and 保存字符串字面量或文档字符串.

### Block 30 — Lines 103-103 (test_make_expt_assignment)
```python
103|     expt_dict = _make_expt_dict_for_mode(n_expts_shard, n_expts_tot, affinity_mode)
```
**EN:** Assigns `expt_dict` and calls `_make_expt_dict_for_mode`.

**CN:** 将 `expt_dict`，并调用 `_make_expt_dict_for_mode`.

### Block 31 — Lines 104-105 (test_make_expt_assignment)
```python
104|     expt_assignment = make_expt_assignment(n_expts_shard, n_expts_tot, expt_dict, device)
105|     # mask correctness & uniqueness: each expert set exactly once, and on the right shard
```
**EN:** Assigns `expt_assignment` and calls `make_expt_assignment`.

**CN:** 将 `expt_assignment`，并调用 `make_expt_assignment`.

### Block 32 — Lines 106-120 (test_make_expt_assignment)
```python
106|     for shard in range(n_expts_shard):
107|         bitmask = expt_assignment.expt_bitmask[shard, :]
108|         bitmask = (bitmask >> torch.arange(32, device=bitmask.device)[:, None]) & 1
109|         experts = bitmask.T.flatten().nonzero()[:, 0].tolist()
110|         assert sorted(expt_dict[shard]) == experts
111|         expt_map = torch.full((n_expts_tot, ), -1, device=device)
112|         expt_map[experts] = torch.arange(len(experts), device=expt_map.device)
113|         assert torch.all(expt_map == expt_assignment.expt_map[shard, :])
114| 
115| 
116| # ------------------------------------------------------------
117| # expert sharding
118| # ------------------------------------------------------------
119| 
120| 
```
**EN:** Loops over `range(n_expts_shard)` with target `shard`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `range(n_expts_shard)` ，目标变量为 `shard`. 循环体主要准备中间值; 准备中间值.

### Block 33 — Lines 121-121 (routing)
```python
121| def routing(logits, n_expts_act, all_gather=False, y_indx=None):
```
**EN:** Defines function `routing(logits, n_expts_act, all_gather, y_indx)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `topk`, `make_ragged_tensor_metadata`, `torch.div` to implement its workflow.

**CN:** 定义函数 `routing(logits, n_expts_act, all_gather, y_indx)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `topk`, `make_ragged_tensor_metadata`, `torch.div` 来实现其工作流程.

### Block 34 — Lines 122-122 (routing)
```python
122|     sparse_logits = topk(logits, n_expts_act, all_gather=all_gather, y_indx=y_indx)
```
**EN:** Assigns `sparse_logits` and calls `topk`.

**CN:** 将 `sparse_logits`，并调用 `topk`.

### Block 35 — Lines 123-123 (routing)
```python
123|     dispatch_indx = sparse_logits.mask_metadata.row_sorted_indx
```
**EN:** Assigns `dispatch_indx` and references `sparse_logits.mask_metadata.row_sorted_indx`.

**CN:** 将 `dispatch_indx` and 引用 `sparse_logits.mask_metadata.row_sorted_indx`.

### Block 36 — Lines 124-124 (routing)
```python
124|     combine_indx = sparse_logits.mask_metadata.col_sorted_indx
```
**EN:** Assigns `combine_indx` and references `sparse_logits.mask_metadata.col_sorted_indx`.

**CN:** 将 `combine_indx` and 引用 `sparse_logits.mask_metadata.col_sorted_indx`.

### Block 37 — Lines 125-125 (routing)
```python
125|     ragged_batch_metadata = make_ragged_tensor_metadata(sparse_logits.mask_metadata.col_sum, dispatch_indx.shape[0])
```
**EN:** Assigns `ragged_batch_metadata` and calls `make_ragged_tensor_metadata`.

**CN:** 将 `ragged_batch_metadata`，并调用 `make_ragged_tensor_metadata`.

### Block 38 — Lines 126-126 (routing)
```python
126|     gather_idx = torch.div(combine_indx, n_expts_act, rounding_mode="trunc")
```
**EN:** Assigns `gather_idx` and calls `torch.div`.

**CN:** 将 `gather_idx`，并调用 `torch.div`.

### Block 39 — Lines 127-127 (routing)
```python
127|     scatter_idx = combine_indx
```
**EN:** Assigns `scatter_idx` and references `combine_indx`.

**CN:** 将 `scatter_idx` and 引用 `combine_indx`.

### Block 40 — Lines 128-130 (routing)
```python
128|     return ragged_batch_metadata, gather_idx, scatter_idx, sparse_logits.indx
129| 
130| 
```
**EN:** Returns `(ragged_batch_metadata, gather_idx, scatter_idx, sparse_logits.indx)`.

**CN:** 返回 `(ragged_batch_metadata, gather_idx, scatter_idx, sparse_logits.indx)`.

### Block 41 — Lines 131-131 (mixture_of_expt_nosharded)
```python
131| def mixture_of_expt_nosharded(x_global, l_global, w_global, b_global, n_expts_act, y_indx=None):
```
**EN:** Defines function `mixture_of_expt_nosharded(x_global, l_global, w_global, b_global, n_expts_act, y_indx)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `routing`, `matmul`, `(dispatch_indx != -1).view`, `y_global.view`, `y_mask.expand_as` to implement its workflow.

**CN:** 定义函数 `mixture_of_expt_nosharded(x_global, l_global, w_global, b_global, n_expts_act, y_indx)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `routing`, `matmul`, `(dispatch_indx != -1).view`, `y_global.view`, `y_mask.expand_as` 来实现其工作流程.

### Block 42 — Lines 132-132 (mixture_of_expt_nosharded)
```python
132|     rdata, combine_indx, dispatch_indx, _ = routing(l_global, n_expts_act, y_indx=y_indx)
```
**EN:** Assigns `rdata`, `combine_indx`, `dispatch_indx`, `_` and calls `routing`.

**CN:** 将 `rdata`, `combine_indx`, `dispatch_indx`, `_`，并调用 `routing`.

### Block 43 — Lines 133-133 (mixture_of_expt_nosharded)
```python
133|     y_global = matmul(x_global, w_global, b_global, rdata, gather_indx=combine_indx, scatter_indx=dispatch_indx)
```
**EN:** Assigns `y_global` and calls `matmul`.

**CN:** 将 `y_global`，并调用 `matmul`.

### Block 44 — Lines 134-134 (mixture_of_expt_nosharded)
```python
134|     y_mask = (dispatch_indx != -1).view(y_global.shape[-2] // n_expts_act, n_expts_act, 1)
```
**EN:** Assigns `y_mask` and calls `(dispatch_indx != -1).view`.

**CN:** 将 `y_mask`，并调用 `(dispatch_indx != -1).view`.

### Block 45 — Lines 135-135 (mixture_of_expt_nosharded)
```python
135|     y_global = y_global.view(y_global.shape[-2] // n_expts_act, n_expts_act, -1)
```
**EN:** Assigns `y_global` and calls `y_global.view`.

**CN:** 将 `y_global`，并调用 `y_global.view`.

### Block 46 — Lines 136-136 (mixture_of_expt_nosharded)
```python
136|     y_mask = y_mask.expand_as(y_global)
```
**EN:** Assigns `y_mask` and calls `y_mask.expand_as`.

**CN:** 将 `y_mask`，并调用 `y_mask.expand_as`.

### Block 47 — Lines 137-137 (mixture_of_expt_nosharded)
```python
137|     y_global, _ = reduce(y_global, dim=1, mask=y_mask)
```
**EN:** Assigns `y_global`, `_` and calls `reduce`.

**CN:** 将 `y_global`, `_`，并调用 `reduce`.

### Block 48 — Lines 138-140 (mixture_of_expt_nosharded)
```python
138|     return y_global
139| 
140| 
```
**EN:** Returns `y_global`.

**CN:** 返回 `y_global`.

### Block 49 — Lines 141-142 (mixture_of_expt_epsharded)
```python
141| def mixture_of_expt_epsharded(x_dp_local, l_dp_local, w_ep_local, b_ep_local, expt_assignment, n_expts_act,
142|                               symm_mem_pool, y_indx=None):
```
**EN:** Defines function `mixture_of_expt_epsharded(x_dp_local, l_dp_local, w_ep_local, b_ep_local, expt_assignment, n_expts_act, symm_mem_pool, y_indx)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `dist.get_rank`, `topk`, `make_ragged_tensor_metadata`, `convert_dp_to_ep`, `remap_ragged_tensor_metadata` to implement its workflow.

**CN:** 定义函数 `mixture_of_expt_epsharded(x_dp_local, l_dp_local, w_ep_local, b_ep_local, expt_assignment, n_expts_act, symm_mem_pool, y_indx)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `dist.get_rank`, `topk`, `make_ragged_tensor_metadata`, `convert_dp_to_ep`, `remap_ragged_tensor_metadata` 来实现其工作流程.

### Block 50 — Lines 143-143 (mixture_of_expt_epsharded)
```python
143|     rank = dist.get_rank()
```
**EN:** Assigns `rank` and calls `dist.get_rank`.

**CN:** 将 `rank`，并调用 `dist.get_rank`.

### Block 51 — Lines 144-145 (mixture_of_expt_epsharded)
```python
144|     expt_map = expt_assignment.expt_map[rank, :]
145|     # active global logits (sparse)
```
**EN:** Assigns `expt_map` and evaluates `expt_assignment.expt_map[rank, :]`.

**CN:** 将 `expt_map` and 计算 `expt_assignment.expt_map[rank, :]`.

### Block 52 — Lines 146-148 (mixture_of_expt_epsharded)
```python
146|     l_global_active = topk(l_dp_local, n_expts_act, apply_softmax=True, all_gather=True, y_indx=y_indx,
147|                            symm_mem_pool=symm_mem_pool)
148|     # expert histogram, dispatch/combine indx
```
**EN:** Assigns `l_global_active` and calls `topk`.

**CN:** 将 `l_global_active`，并调用 `topk`.

### Block 53 — Lines 149-149 (mixture_of_expt_epsharded)
```python
149|     active_indx = l_global_active.indx
```
**EN:** Assigns `active_indx` and references `l_global_active.indx`.

**CN:** 将 `active_indx` and 引用 `l_global_active.indx`.

### Block 54 — Lines 150-150 (mixture_of_expt_epsharded)
```python
150|     expt_sizes = l_global_active.mask_metadata.col_sum
```
**EN:** Assigns `expt_sizes` and references `l_global_active.mask_metadata.col_sum`.

**CN:** 将 `expt_sizes` and 引用 `l_global_active.mask_metadata.col_sum`.

### Block 55 — Lines 151-151 (mixture_of_expt_epsharded)
```python
151|     dispatch_indx = l_global_active.mask_metadata.row_sorted_indx
```
**EN:** Assigns `dispatch_indx` and references `l_global_active.mask_metadata.row_sorted_indx`.

**CN:** 将 `dispatch_indx` and 引用 `l_global_active.mask_metadata.row_sorted_indx`.

### Block 56 — Lines 152-153 (mixture_of_expt_epsharded)
```python
152|     combine_indx = l_global_active.mask_metadata.col_sorted_indx
153|     # ragged tensor metadata
```
**EN:** Assigns `combine_indx` and references `l_global_active.mask_metadata.col_sorted_indx`.

**CN:** 将 `combine_indx` and 引用 `l_global_active.mask_metadata.col_sorted_indx`.

### Block 57 — Lines 154-155 (mixture_of_expt_epsharded)
```python
154|     x_global_metadata = make_ragged_tensor_metadata(expt_sizes, dispatch_indx.shape[0])
155|     # convert x from dp-local to expert-sorted, ep-local
```
**EN:** Assigns `x_global_metadata` and calls `make_ragged_tensor_metadata`.

**CN:** 将 `x_global_metadata`，并调用 `make_ragged_tensor_metadata`.

### Block 58 — Lines 156-156 (mixture_of_expt_epsharded)
```python
156|     y_ep_local = convert_dp_to_ep(x_dp_local, expt_assignment, active_indx, dispatch_indx, symm_mem_pool)
```
**EN:** Assigns `y_ep_local` and calls `convert_dp_to_ep`.

**CN:** 将 `y_ep_local`，并调用 `convert_dp_to_ep`.

### Block 59 — Lines 157-158 (mixture_of_expt_epsharded)
```python
157|     y_ep_local_metadata = remap_ragged_tensor_metadata(x_global_metadata, expt_map)
158|     # matrix multiply
```
**EN:** Assigns `y_ep_local_metadata` and calls `remap_ragged_tensor_metadata`.

**CN:** 将 `y_ep_local_metadata`，并调用 `remap_ragged_tensor_metadata`.

### Block 60 — Lines 159-160 (mixture_of_expt_epsharded)
```python
159|     y_ep_local = matmul(y_ep_local, w_ep_local, b_ep_local, a_ragged_metadata=y_ep_local_metadata)
160|     # convert x from expert-sorted, ep-local to token-sorted, dp-local
```
**EN:** Assigns `y_ep_local` and calls `matmul`.

**CN:** 将 `y_ep_local`，并调用 `matmul`.

### Block 61 — Lines 161-162 (mixture_of_expt_epsharded)
```python
161|     y_dp_local = convert_ep_to_dp(y_ep_local, expt_assignment, active_indx, combine_indx, symm_mem_pool)
162|     # weighted average of the output token from experts
```
**EN:** Assigns `y_dp_local` and calls `convert_ep_to_dp`.

**CN:** 将 `y_dp_local`，并调用 `convert_ep_to_dp`.

### Block 62 — Lines 163-163 (mixture_of_expt_epsharded)
```python
163|     y_dp_local = y_dp_local.view(-1, n_expts_act, y_dp_local.shape[-1])
```
**EN:** Assigns `y_dp_local` and calls `y_dp_local.view`.

**CN:** 将 `y_dp_local`，并调用 `y_dp_local.view`.

### Block 63 — Lines 164-164 (mixture_of_expt_epsharded)
```python
164|     z_dp_local, _ = reduce(y_dp_local, dim=1)
```
**EN:** Assigns `z_dp_local`, `_` and calls `reduce`.

**CN:** 将 `z_dp_local`, `_`，并调用 `reduce`.

### Block 64 — Lines 165-167 (mixture_of_expt_epsharded)
```python
165|     return z_dp_local
166| 
167| 
```
**EN:** Returns `z_dp_local`.

**CN:** 返回 `z_dp_local`.

### Block 65 — Lines 168-168 (_run_expert_sharding)
```python
168| def _run_expert_sharding(rank, world_size, *, n_tokens, d_model, n_expts_tot, n_expts_act, affinity_mode):
```
**EN:** Defines function `_run_expert_sharding(rank, world_size, n_tokens, d_model, n_expts_tot, n_expts_act, affinity_mode)` for this module. The body mainly invokes `torch.manual_seed`; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.manual_seed`, `torch.cuda.current_device`, `_make_expt_dict_for_mode`, `make_expt_assignment`, `torch.randn` to implement its workflow.

**CN:** 定义函数 `_run_expert_sharding(rank, world_size, n_tokens, d_model, n_expts_tot, n_expts_act, affinity_mode)`，供本模块使用. 主体主要invokes `torch.manual_seed`; 准备中间值; 准备中间值. 其中会调用 `torch.manual_seed`, `torch.cuda.current_device`, `_make_expt_dict_for_mode`, `make_expt_assignment`, `torch.randn` 来实现其工作流程.

### Block 66 — Lines 169-170 (_run_expert_sharding)
```python
169|     torch.manual_seed(0)
170| 
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 67 — Lines 171-171 (_run_expert_sharding)
```python
171|     dev = torch.cuda.current_device()
```
**EN:** Assigns `dev` and calls `torch.cuda.current_device`.

**CN:** 将 `dev`，并调用 `torch.cuda.current_device`.

### Block 68 — Lines 172-173 (_run_expert_sharding)
```python
172|     n_shards = world_size
173| 
```
**EN:** Assigns `n_shards` and references `world_size`.

**CN:** 将 `n_shards` and 引用 `world_size`.

### Block 69 — Lines 174-174 (_run_expert_sharding)
```python
174|     expt_dict = _make_expt_dict_for_mode(n_shards, n_expts_tot, affinity_mode)
```
**EN:** Assigns `expt_dict` and calls `_make_expt_dict_for_mode`.

**CN:** 将 `expt_dict`，并调用 `_make_expt_dict_for_mode`.

### Block 70 — Lines 175-176 (_run_expert_sharding)
```python
175|     expt_assignment = make_expt_assignment(n_shards, n_expts_tot, expt_dict, device=dev)
176|     # reference data
```
**EN:** Assigns `expt_assignment` and calls `make_expt_assignment`.

**CN:** 将 `expt_assignment`，并调用 `make_expt_assignment`.

### Block 71 — Lines 177-177 (_run_expert_sharding)
```python
177|     n_tokens_global = n_tokens
```
**EN:** Assigns `n_tokens_global` and references `n_tokens`.

**CN:** 将 `n_tokens_global` and 引用 `n_tokens`.

### Block 72 — Lines 178-178 (_run_expert_sharding)
```python
178|     x_global = torch.randn(n_tokens_global, d_model, device=dev, dtype=torch.bfloat16)
```
**EN:** Assigns `x_global` and calls `torch.randn`.

**CN:** 将 `x_global`，并调用 `torch.randn`.

### Block 73 — Lines 179-179 (_run_expert_sharding)
```python
179|     l_global = torch.rand(n_tokens_global, n_expts_tot, device=dev, dtype=torch.float32)
```
**EN:** Assigns `l_global` and calls `torch.rand`.

**CN:** 将 `l_global`，并调用 `torch.rand`.

### Block 74 — Lines 180-180 (_run_expert_sharding)
```python
180|     w_global = torch.randn((n_expts_tot, d_model, d_model), device=dev, dtype=torch.bfloat16)
```
**EN:** Assigns `w_global` and calls `torch.randn`.

**CN:** 将 `w_global`，并调用 `torch.randn`.

### Block 75 — Lines 181-182 (_run_expert_sharding)
```python
181|     b_global = torch.randn((n_expts_tot, d_model), device=dev, dtype=torch.float32)
182|     # initialize data shard
```
**EN:** Assigns `b_global` and calls `torch.randn`.

**CN:** 将 `b_global`，并调用 `torch.randn`.

### Block 76 — Lines 183-183 (_run_expert_sharding)
```python
183|     n_tokens_local = n_tokens_global // n_shards
```
**EN:** Assigns `n_tokens_local` and evaluates `n_tokens_global // n_shards`.

**CN:** 将 `n_tokens_local` and 计算 `n_tokens_global // n_shards`.

### Block 77 — Lines 184-184 (_run_expert_sharding)
```python
184|     first_token_indx, last_token_indx = rank * n_tokens_local, (rank + 1) * n_tokens_local
```
**EN:** Assigns `first_token_indx`, `last_token_indx` and builds a tuple.

**CN:** 将 `first_token_indx`, `last_token_indx` and 构造一个元组.

### Block 78 — Lines 185-185 (_run_expert_sharding)
```python
185|     w_ep_local = w_global[expt_assignment.expt_boolmask[rank, :], :, :]
```
**EN:** Assigns `w_ep_local` and evaluates `w_global[expt_assignment.expt_boolmask[rank, :], :, :]`.

**CN:** 将 `w_ep_local` and 计算 `w_global[expt_assignment.expt_boolmask[rank, :], :, :]`.

### Block 79 — Lines 186-186 (_run_expert_sharding)
```python
186|     b_ep_local = b_global[expt_assignment.expt_boolmask[rank, :], :]
```
**EN:** Assigns `b_ep_local` and evaluates `b_global[expt_assignment.expt_boolmask[rank, :], :]`.

**CN:** 将 `b_ep_local` and 计算 `b_global[expt_assignment.expt_boolmask[rank, :], :]`.

### Block 80 — Lines 187-187 (_run_expert_sharding)
```python
187|     x_dp_local = x_global[first_token_indx:last_token_indx, :]
```
**EN:** Assigns `x_dp_local` and evaluates `x_global[first_token_indx:last_token_indx, :]`.

**CN:** 将 `x_dp_local` and 计算 `x_global[first_token_indx:last_token_indx, :]`.

### Block 81 — Lines 188-190 (_run_expert_sharding)
```python
188|     l_dp_local = l_global[first_token_indx:last_token_indx, :]
189|     # routing
190|     # test correctness
```
**EN:** Assigns `l_dp_local` and evaluates `l_global[first_token_indx:last_token_indx, :]`.

**CN:** 将 `l_dp_local` and 计算 `l_global[first_token_indx:last_token_indx, :]`.

### Block 82 — Lines 191-191 (_run_expert_sharding)
```python
191|     y_indx_global = _make_y_indx_for_mode(n_tokens_global, n_expts_tot, n_expts_act, n_shards, affinity_mode, dev)
```
**EN:** Assigns `y_indx_global` and calls `_make_y_indx_for_mode`.

**CN:** 将 `y_indx_global`，并调用 `_make_y_indx_for_mode`.

### Block 83 — Lines 192-200 (_run_expert_sharding)
```python
192|     y_global_ref = mixture_of_expt_nosharded(
193|         x_global,
194|         l_global,
195|         w_global,
196|         b_global,
197|         n_expts_act,
198|         y_indx=y_indx_global,
199|     )
200| 
```
**EN:** Assigns `y_global_ref` and calls `mixture_of_expt_nosharded`.

**CN:** 将 `y_global_ref`，并调用 `mixture_of_expt_nosharded`.

### Block 84 — Lines 201-201 (_run_expert_sharding)
```python
201|     symm_mem_pool = SymmetricMemoryPool(Mesh(dist.group.WORLD))
```
**EN:** Assigns `symm_mem_pool` and calls `SymmetricMemoryPool`.

**CN:** 将 `symm_mem_pool`，并调用 `SymmetricMemoryPool`.

### Block 85 — Lines 202-211 (_run_expert_sharding)
```python
202|     symm_mem_pool.initialize_matmul(
203|         n_tokens_global=n_tokens_global,
204|         d_input=d_model,
205|         d_model=d_model,
206|         n_expts_act=n_expts_act,
207|         n_expts_tot=n_expts_tot,
208|         dtype=torch.bfloat16,
209|         device=dev,
210|     )
211| 
```
**EN:** Calls `symm_mem_pool.initialize_matmul` for side effects, registration, or validation.

**CN:** 调用 `symm_mem_pool.initialize_matmul` ，用于副作用、注册或校验。

### Block 86 — Lines 212-212 (run_moe)
```python
212|     def run_moe():
```
**EN:** Defines function `run_moe()` for this module. The body mainly returns the computed result. It uses calls such as `mixture_of_expt_epsharded` to implement its workflow.

**CN:** 定义函数 `run_moe()`，供本模块使用. 主体主要返回计算结果. 其中会调用 `mixture_of_expt_epsharded` 来实现其工作流程.

### Block 87 — Lines 213-223 (run_moe)
```python
213|         return mixture_of_expt_epsharded(
214|             x_dp_local,
215|             l_dp_local,
216|             w_ep_local,
217|             b_ep_local,
218|             expt_assignment,
219|             n_expts_act,
220|             y_indx=y_indx_global,
221|             symm_mem_pool=symm_mem_pool,
222|         )
223| 
```
**EN:** Returns `mixture_of_expt_epsharded(x_dp_local, l_dp_local, w_ep_local, b_ep_local, exp...`.

**CN:** 返回 `mixture_of_expt_epsharded(x_dp_local, l_dp_local, w_ep_local, b_ep_local, exp...`.

### Block 88 — Lines 224-224 (_run_expert_sharding)
```python
224|     y_dp_local_tri = run_moe()
```
**EN:** Assigns `y_dp_local_tri` and calls `run_moe`.

**CN:** 将 `y_dp_local_tri`，并调用 `run_moe`.

### Block 89 — Lines 225-227 (_run_expert_sharding)
```python
225|     y_global_tri = torch.empty_like(y_global_ref)
226| 
227|     # Validate warmup run.
```
**EN:** Assigns `y_global_tri` and calls `torch.empty_like`.

**CN:** 将 `y_global_tri`，并调用 `torch.empty_like`.

### Block 90 — Lines 228-228 (_run_expert_sharding)
```python
228|     dist.all_gather_into_tensor(y_global_tri, y_dp_local_tri)
```
**EN:** Calls `dist.all_gather_into_tensor` for side effects, registration, or validation.

**CN:** 调用 `dist.all_gather_into_tensor` ，用于副作用、注册或校验。

### Block 91 — Lines 229-231 (_run_expert_sharding)
```python
229|     triton.testing.assert_close(y_global_ref, y_global_tri)
230| 
231|     # Validate cuda graph capture + replay.
```
**EN:** Calls `triton.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `triton.testing.assert_close` ，用于副作用、注册或校验。

### Block 92 — Lines 232-232 (_run_expert_sharding)
```python
232|     g = torch.cuda.CUDAGraph()
```
**EN:** Assigns `g` and calls `torch.cuda.CUDAGraph`.

**CN:** 将 `g`，并调用 `torch.cuda.CUDAGraph`.

### Block 93 — Lines 233-233 (_run_expert_sharding)
```python
233|     stream = torch.cuda.Stream()
```
**EN:** Assigns `stream` and calls `torch.cuda.Stream`.

**CN:** 将 `stream`，并调用 `torch.cuda.Stream`.

### Block 94 — Lines 234-237 (_run_expert_sharding)
```python
234|     with torch.cuda.stream(stream):
235|         with torch.cuda.graph(g):
236|             y_dp_local_tri_graph = run_moe()
237| 
```
**EN:** Uses context manager(s) `torch.cuda.stream(stream)` around code that mainly uses contextual resources.

**CN:** 使用上下文管理器 `torch.cuda.stream(stream)` ，其中代码主要使用上下文资源.

### Block 95 — Lines 238-238 (_run_expert_sharding)
```python
238|     g.replay()
```
**EN:** Calls `g.replay` for side effects, registration, or validation.

**CN:** 调用 `g.replay` ，用于副作用、注册或校验。

### Block 96 — Lines 239-239 (_run_expert_sharding)
```python
239|     dist.all_gather_into_tensor(y_global_tri, y_dp_local_tri_graph)
```
**EN:** Calls `dist.all_gather_into_tensor` for side effects, registration, or validation.

**CN:** 调用 `dist.all_gather_into_tensor` ，用于副作用、注册或校验。

### Block 97 — Lines 240-242 (_run_expert_sharding)
```python
240|     triton.testing.assert_close(y_global_ref, y_global_tri)
241| 
242| 
```
**EN:** Calls `triton.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `triton.testing.assert_close` ，用于副作用、注册或校验。

### Block 98 — Lines 243-247 (test_expert_sharding)
```python
243| @pytest.mark.parametrize("distributed_launcher", [2, 4], indirect=True)
244| @pytest.mark.parametrize("n_tokens", [16, 128, 4096])
245| @pytest.mark.parametrize("d_model, n_expts_tot, n_expts_act", [(16, 4, 4), (5760, 128, 4)])
246| @pytest.mark.parametrize("affinity_mode", ["uniform", "random"])
247| def test_expert_sharding(distributed_launcher, n_tokens, d_model, n_expts_tot, n_expts_act, affinity_mode):
```
**EN:** Defines function `test_expert_sharding(distributed_launcher, n_tokens, d_model, n_expts_tot, n_expts_act, affinity_mode)` with decorators `pytest.mark.parametrize('distributed_...`, `pytest.mark.parametrize('n_tokens', [...`, `pytest.mark.parametrize('d_model, n_e...`, `pytest.mark.parametrize('affinity_mod...` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; invokes `distributed_launcher`. It uses calls such as `pytest.mark.parametrize`, `distributed_launcher`, `ValueError` to implement its workflow.

**CN:** 定义函数 `test_expert_sharding(distributed_launcher, n_tokens, d_model, n_expts_tot, n_expts_act, affinity_mode)`，带有装饰器 `pytest.mark.parametrize('distributed_...`, `pytest.mark.parametrize('n_tokens', [...`, `pytest.mark.parametrize('d_model, n_e...`, `pytest.mark.parametrize('affinity_mod...`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; invokes `distributed_launcher`. 其中会调用 `pytest.mark.parametrize`, `distributed_launcher`, `ValueError` 来实现其工作流程.

### Block 99 — Lines 248-249 (test_expert_sharding)
```python
248|     if n_tokens < distributed_launcher.world_size:
249|         raise ValueError("n_tokens must be >= number of gpus")
```
**EN:** Checks `n_tokens < distributed_launcher.world_size`..

**CN:** 检查 `n_tokens < distributed_launcher.world_size`..

### Block 100 — Lines 250-252 (test_expert_sharding)
```python
250|     if n_tokens % distributed_launcher.world_size != 0:
251|         raise ValueError("n_tokens must be divisible by number of gpus")
252| 
```
**EN:** Checks `n_tokens % distributed_launcher.world_size != 0`..

**CN:** 检查 `n_tokens % distributed_launcher.world_size != 0`..

### Block 101 — Lines 253-260 (test_expert_sharding)
```python
253|     distributed_launcher(
254|         _run_expert_sharding,
255|         n_tokens=n_tokens,
256|         d_model=d_model,
257|         n_expts_tot=n_expts_tot,
258|         n_expts_act=n_expts_act,
259|         affinity_mode=affinity_mode,
260|     )
```
**EN:** Calls `distributed_launcher` for side effects, registration, or validation.

**CN:** 调用 `distributed_launcher` ，用于副作用、注册或校验。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `_make_expt_dict_for_mode`, `_make_y_indx_for_mode`, `_get_free_tcp_port`, `_distributed_worker`, `distributed_launcher`, `test_make_expt_assignment`, `routing`, `mixture_of_expt_nosharded`.
  **CN:** 主要符号：`_make_expt_dict_for_mode`, `_make_y_indx_for_mode`, `_get_free_tcp_port`, `_distributed_worker`, `distributed_launcher`, `test_make_expt_assignment`, `routing`, `mixture_of_expt_nosharded`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。
- **EN:** Distributed execution concepts such as meshes, ranks, or shard mapping appear in this file.
  **CN:** 该文件涉及 mesh、rank 或分片映射等分布式执行概念。
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。
- **EN:** Top-k selection and, where relevant, gradient propagation are key operations here.
  **CN:** 这里的关键操作是 Top-k 选择以及相关的梯度传播。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `contextlib`, `os`, `socket`, `torch`, `torch.distributed`, `torch.multiprocessing`, `triton`, `pytest`.
  **CN:** 外部模块：`contextlib`, `os`, `socket`, `torch`, `torch.distributed`, `torch.multiprocessing`, `triton`, `pytest`。
- **EN:** Internal modules: `triton_kernels.distributed (convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_dict_random, make_expt_assignment, SymmetricMemoryPool)`, `triton_kernels.distributed_details.mesh (Mesh)`, `triton_kernels.reduce (reduce)`, `triton_kernels.topk (topk)`, `triton_kernels.matmul (matmul)`, `triton_kernels.tensor (make_ragged_tensor_metadata, remap_ragged_tensor_metadata)`.
  **CN:** 内部模块：`triton_kernels.distributed (convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_dict_random, make_expt_assignment, SymmetricMemoryPool)`, `triton_kernels.distributed_details.mesh (Mesh)`, `triton_kernels.reduce (reduce)`, `triton_kernels.topk (topk)`, `triton_kernels.matmul (matmul)`, `triton_kernels.tensor (make_ragged_tensor_metadata, remap_ragged_tensor_metadata)`。

# testing.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/testing.py`
- **Purpose / 用途:** Implementation module for testing; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols assert_equal, assert_close, ComputeSanitizerTool, compute_sanitizer, compute_actual_scale. / 用于 testing 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 assert_equal、assert_close、ComputeSanitizerTool、compute_sanitizer、compute_actual_scale。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-13 (module)
```python
 1| import enum
 2| import functools
 3| import os
 4| import subprocess
 5| import sys
 6| import torch
 7| from triton_kernels.numerics import MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5
 8| from triton_kernels.tensor import convert_layout, wrap_torch_tensor, FP4, make_ragged_tensor_metadata
 9| from triton_kernels.numerics_details.mxfp import downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE
10| import itertools
11| from dataclasses import replace
12| 
13| 
```
**EN:** This block imports `enum`, `functools`, `os`, `subprocess`, `sys`, `torch`, `triton_kernels.numerics (MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, make_ragged_tensor_metadata)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `enum`, `functools`, `os`, `subprocess`, `sys`, `torch`, `triton_kernels.numerics (MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, make_ragged_tensor_metadata)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 14-14 (assert_equal)
```python
14| def assert_equal(ref, tri):
```
**EN:** Defines function `assert_equal(ref, tri)` for this module. The body mainly branches on runtime conditions. It uses calls such as `isinstance`, `torch.all` to implement its workflow.

**CN:** 定义函数 `assert_equal(ref, tri)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `isinstance`, `torch.all` 来实现其工作流程.

### Block 3 — Lines 15-20 (assert_equal)
```python
15|     if isinstance(ref, torch.Tensor):
16|         assert torch.all(ref == tri)
17|     else:
18|         assert ref == tri
19| 
20| 
```
**EN:** Checks `isinstance(ref, torch.Tensor)`. The true branch mainly checks invariants, while the else branch checks invariants.

**CN:** 检查 `isinstance(ref, torch.Tensor)`. 真分支主要检查不变量；而 else 分支检查不变量.

### Block 4 — Lines 21-21 (assert_close)
```python
21| def assert_close(ref, tri, maxtol=None, rmstol=None, description="--", verbose=True):
```
**EN:** Defines function `assert_close(ref, tri, maxtol, rmstol, description, verbose)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; branches on runtime conditions. It uses calls such as `ref.to(torch.float32).detach`, `tri.to(torch.float32).detach`, `torch.isinf`, `torch.equal`, `torch.where` to implement its workflow.

**CN:** 定义函数 `assert_close(ref, tri, maxtol, rmstol, description, verbose)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `ref.to(torch.float32).detach`, `tri.to(torch.float32).detach`, `torch.isinf`, `torch.equal`, `torch.where` 来实现其工作流程.

### Block 5 — Lines 22-28 (assert_close)
```python
22|     if tri.dtype.itemsize == 1:
23|         ref_as_type = ref.to(tri.dtype)
24|         if ref.dtype == tri.dtype:
25|             assert torch.all(ref_as_type == tri)
26|             return
27|         ref = ref_as_type
28| 
```
**EN:** Checks `tri.dtype.itemsize == 1`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `tri.dtype.itemsize == 1`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 6 — Lines 29-31 (assert_close)
```python
29|     if ref.numel() == 0:
30|         return
31| 
```
**EN:** Checks `ref.numel() == 0`. The true branch mainly returns the computed result.

**CN:** 检查 `ref.numel() == 0`. 真分支主要返回计算结果.

### Block 7 — Lines 32-33 (assert_close)
```python
32|     if maxtol is None:
33|         maxtol = 2e-2
```
**EN:** Checks `maxtol is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `maxtol is None`. 真分支主要准备中间值.

### Block 8 — Lines 34-35 (assert_close)
```python
34|     if rmstol is None:
35|         rmstol = 4e-3
```
**EN:** Checks `rmstol is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `rmstol is None`. 真分支主要准备中间值.

### Block 9 — Lines 36-40 (assert_close)
```python
36|     """
37|     Compare reference values against obtained values.
38|     """
39| 
40|     # cast to float32:
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 10 — Lines 41-41 (assert_close)
```python
41|     ref = ref.to(torch.float32).detach()
```
**EN:** Assigns `ref` and calls `ref.to(torch.float32).detach`.

**CN:** 将 `ref`，并调用 `ref.to(torch.float32).detach`.

### Block 11 — Lines 42-42 (assert_close)
```python
42|     tri = tri.to(torch.float32).detach()
```
**EN:** Assigns `tri` and calls `tri.to(torch.float32).detach`.

**CN:** 将 `tri`，并调用 `tri.to(torch.float32).detach`.

### Block 12 — Lines 43-45 (assert_close)
```python
43|     assert ref.shape == tri.shape, f"Tensors must have same size {ref.shape=} {tri.shape=}"
44| 
45|     # deal with infinite elements:
```
**EN:** Asserts `ref.shape == tri.shape` to enforce invariants.

**CN:** 断言 `ref.shape == tri.shape` 以确保不变量成立。

### Block 13 — Lines 46-46 (assert_close)
```python
46|     inf_mask_ref = torch.isinf(ref)
```
**EN:** Assigns `inf_mask_ref` and calls `torch.isinf`.

**CN:** 将 `inf_mask_ref`，并调用 `torch.isinf`.

### Block 14 — Lines 47-47 (assert_close)
```python
47|     inf_mask_tri = torch.isinf(tri)
```
**EN:** Assigns `inf_mask_tri` and calls `torch.isinf`.

**CN:** 将 `inf_mask_tri`，并调用 `torch.isinf`.

### Block 15 — Lines 48-48 (assert_close)
```python
48|     assert torch.equal(inf_mask_ref, inf_mask_tri), "Tensor must have same infinite elements"
```
**EN:** Asserts `torch.equal(inf_mask_ref, inf_mask_tri)` to enforce invariants.

**CN:** 断言 `torch.equal(inf_mask_ref, inf_mask_tri)` 以确保不变量成立。

### Block 16 — Lines 49-49 (assert_close)
```python
49|     refn = torch.where(inf_mask_ref, 0, ref)
```
**EN:** Assigns `refn` and calls `torch.where`.

**CN:** 将 `refn`，并调用 `torch.where`.

### Block 17 — Lines 50-52 (assert_close)
```python
50|     trin = torch.where(inf_mask_tri, 0, tri)
51| 
52|     # normalise so that RMS calculation doesn't overflow:
```
**EN:** Assigns `trin` and calls `torch.where`.

**CN:** 将 `trin`，并调用 `torch.where`.

### Block 18 — Lines 53-53 (assert_close)
```python
53|     eps = 1.0e-30
```
**EN:** Assigns `eps` and stores constant `1e-30`.

**CN:** 将 `eps` and 保存常量 `1e-30`.

### Block 19 — Lines 54-54 (assert_close)
```python
54|     multiplier = 1.0 / (torch.max(torch.abs(refn)) + eps)
```
**EN:** Assigns `multiplier` and evaluates `1.0 / (torch.max(torch.abs(refn)) + eps)`.

**CN:** 将 `multiplier` and 计算 `1.0 / (torch.max(torch.abs(refn)) + eps)`.

### Block 20 — Lines 55-55 (assert_close)
```python
55|     refn *= multiplier
```
**EN:** Updates `refn` with operator `Mult` using `multiplier`.

**CN:** 更新 `refn`，使用运算符 `Mult`，并使用 `multiplier`.

### Block 21 — Lines 56-57 (assert_close)
```python
56|     trin *= multiplier
57| 
```
**EN:** Updates `trin` with operator `Mult` using `multiplier`.

**CN:** 更新 `trin`，使用运算符 `Mult`，并使用 `multiplier`.

### Block 22 — Lines 58-59 (assert_close)
```python
58|     ref_rms = torch.sqrt(torch.square(refn).mean()) + eps
59| 
```
**EN:** Assigns `ref_rms` and evaluates `torch.sqrt(torch.square(refn).mean()) + eps`.

**CN:** 将 `ref_rms` and 计算 `torch.sqrt(torch.square(refn).mean()) + eps`.

### Block 23 — Lines 60-60 (assert_close)
```python
60|     rel_err = torch.abs(refn - trin) / torch.maximum(ref_rms, torch.abs(refn))
```
**EN:** Assigns `rel_err` and evaluates `torch.abs(refn - trin) / torch.maximum(ref_rms, torch.abs(refn))`.

**CN:** 将 `rel_err` and 计算 `torch.abs(refn - trin) / torch.maximum(ref_rms, torch.abs(refn))`.

### Block 24 — Lines 61-61 (assert_close)
```python
61|     max_err = torch.max(rel_err).item()
```
**EN:** Assigns `max_err` and calls `torch.max(rel_err).item`.

**CN:** 将 `max_err`，并调用 `torch.max(rel_err).item`.

### Block 25 — Lines 62-63 (assert_close)
```python
62|     rms_err = torch.sqrt(torch.square(rel_err).mean()).item()
63| 
```
**EN:** Assigns `rms_err` and calls `torch.sqrt(torch.square(rel_err).mean()).item`.

**CN:** 将 `rms_err`，并调用 `torch.sqrt(torch.square(rel_err).mean()).item`.

### Block 26 — Lines 64-67 (assert_close)
```python
64|     if verbose:
65|         print("%s maximum relative error = %s (threshold = %s)" % (description, max_err, maxtol))
66|         print("%s RMS relative error = %s (threshold = %s)" % (description, rms_err, rmstol))
67| 
```
**EN:** Checks `verbose`. The true branch mainly invokes `print`; invokes `print`.

**CN:** 检查 `verbose`. 真分支主要invokes `print`; invokes `print`.

### Block 27 — Lines 68-78 (assert_close)
```python
68|     if max_err > maxtol:
69|         bad_idxs = torch.nonzero(rel_err > maxtol)
70|         num_nonzero = bad_idxs.size(0)
71|         bad_idxs = bad_idxs[:1000]
72|         print("%d / %d mismatched elements (shape = %s) at coords %s" %
73|               (num_nonzero, rel_err.numel(), tuple(rel_err.shape), bad_idxs.tolist()))
74| 
75|         bad_idxs = bad_idxs.unbind(-1)
76|         print("ref values: ", ref[tuple(bad_idxs)].cpu())
77|         print("tri values: ", tri[tuple(bad_idxs)].cpu())
78| 
```
**EN:** Checks `max_err > maxtol`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `max_err > maxtol`. 真分支主要准备中间值; 准备中间值.

### Block 28 — Lines 79-79 (assert_close)
```python
79|     assert max_err <= maxtol
```
**EN:** Asserts `max_err <= maxtol` to enforce invariants.

**CN:** 断言 `max_err <= maxtol` 以确保不变量成立。

### Block 29 — Lines 80-82 (assert_close)
```python
80|     assert rms_err <= rmstol
81| 
82| 
```
**EN:** Asserts `rms_err <= rmstol` to enforce invariants.

**CN:** 断言 `rms_err <= rmstol` 以确保不变量成立。

### Block 30 — Lines 83-83 (ComputeSanitizerTool)
```python
83| class ComputeSanitizerTool(enum.Enum):
```
**EN:** Defines class `ComputeSanitizerTool` inheriting from `enum.Enum` to organize related behavior. Key fields include `MEMCHECK`, `RACECHECK`, `SYNCCHECK`, `INITCHECK`.

**CN:** 定义类 `ComputeSanitizerTool`，继承自 `enum.Enum`，用于组织相关行为。关键字段包括 `MEMCHECK`, `RACECHECK`, `SYNCCHECK`, `INITCHECK`.

### Block 31 — Lines 84-84 (ComputeSanitizerTool)
```python
84|     MEMCHECK = "memcheck"
```
**EN:** Assigns `MEMCHECK` and stores a string literal or docstring.

**CN:** 将 `MEMCHECK` and 保存字符串字面量或文档字符串.

### Block 32 — Lines 85-85 (ComputeSanitizerTool)
```python
85|     RACECHECK = "racecheck"
```
**EN:** Assigns `RACECHECK` and stores a string literal or docstring.

**CN:** 将 `RACECHECK` and 保存字符串字面量或文档字符串.

### Block 33 — Lines 86-86 (ComputeSanitizerTool)
```python
86|     SYNCCHECK = "synccheck"
```
**EN:** Assigns `SYNCCHECK` and stores a string literal or docstring.

**CN:** 将 `SYNCCHECK` and 保存字符串字面量或文档字符串.

### Block 34 — Lines 87-89 (ComputeSanitizerTool)
```python
87|     INITCHECK = "initcheck"
88| 
89| 
```
**EN:** Assigns `INITCHECK` and stores a string literal or docstring.

**CN:** 将 `INITCHECK` and 保存字符串字面量或文档字符串.

### Block 35 — Lines 90-90 (compute_sanitizer)
```python
90| def compute_sanitizer(**target_kwargs):
```
**EN:** Defines function `compute_sanitizer(**target_kwargs)` for this module. The body mainly returns the computed result. It uses calls such as `functools.wraps`, `target_kwargs.pop`, `isinstance`, `all`, `psutil.Process(os.getppid()).exe` to implement its workflow.

**CN:** 定义函数 `compute_sanitizer(**target_kwargs)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `functools.wraps`, `target_kwargs.pop`, `isinstance`, `all`, `psutil.Process(os.getppid()).exe` 来实现其工作流程.

### Block 36 — Lines 91-99 (compute_sanitizer)
```python
91|     """
92|     Decorator to run a test with compute sanitizer enabled and pytorch caching allocator disabled,
93|     to expose potential memory access errors.
94|     This decorator requires the `request` fixture to be present.
95|     If `run_sanitizer` argument is present and set to False, the sanitizer is not run.
96|     Running tests under compute sanitizer requires launching subprocess and is slow,
97|     so use sparingly
98|     """
99| 
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 37 — Lines 100-101 (decorator)
```python
100|     def decorator(test_fn):
101| 
```
**EN:** Defines function `decorator(test_fn)` for this module. The body mainly returns the computed result. It uses calls such as `functools.wraps`, `target_kwargs.pop`, `isinstance`, `all`, `psutil.Process(os.getppid()).exe` to implement its workflow.

**CN:** 定义函数 `decorator(test_fn)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `functools.wraps`, `target_kwargs.pop`, `isinstance`, `all`, `psutil.Process(os.getppid()).exe` 来实现其工作流程.

### Block 38 — Lines 102-103 (wrapper)
```python
102|         @functools.wraps(test_fn)
103|         def wrapper(*args, **kwargs):
```
**EN:** Defines function `wrapper(*args, **kwargs)` with decorators `functools.wraps(test_fn)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; prepares intermediate values. It uses calls such as `functools.wraps`, `target_kwargs.pop`, `isinstance`, `all`, `psutil.Process(os.getppid()).exe` to implement its workflow.

**CN:** 定义函数 `wrapper(*args, **kwargs)`，带有装饰器 `functools.wraps(test_fn)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 准备中间值. 其中会调用 `functools.wraps`, `target_kwargs.pop`, `isinstance`, `all`, `psutil.Process(os.getppid()).exe` 来实现其工作流程.

### Block 39 — Lines 104-107 (wrapper)
```python
104|             if os.environ.get("SKIP_COMPUTE_SANITIZER") == "1":
105|                 test_fn(*args, **kwargs)
106|                 return
107| 
```
**EN:** Checks `os.environ.get('SKIP_COMPUTE_SANITIZER') == '1'`. The true branch mainly invokes `test_fn`; returns the computed result.

**CN:** 检查 `os.environ.get('SKIP_COMPUTE_SANITIZER') == '1'`. 真分支主要invokes `test_fn`; 返回计算结果.

### Block 40 — Lines 108-109 (wrapper)
```python
108|             import psutil
109| 
```
**EN:** This block imports `psutil` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `psutil` ，为模块提供所需的外部库和内部辅助工具。

### Block 41 — Lines 110-113 (wrapper)
```python
110|             if target_kwargs.pop("clear_torch_cache", False):
111|                 # If we don't pop clear_torch_cache, it won't pass
112|                 # target_kwargs.items() <= kwargs.items() condition below.
113|                 torch.cuda.empty_cache()
```
**EN:** Checks `target_kwargs.pop('clear_torch_cache', False)`. The true branch mainly invokes `torch.cuda.empty_cache`.

**CN:** 检查 `target_kwargs.pop('clear_torch_cache', False)`. 真分支主要invokes `torch.cuda.empty_cache`.

### Block 42 — Lines 114-114 (wrapper)
```python
114|             tools_to_check = target_kwargs.pop("tools_to_check", [ComputeSanitizerTool.MEMCHECK])
```
**EN:** Assigns `tools_to_check` and calls `target_kwargs.pop`.

**CN:** 将 `tools_to_check`，并调用 `target_kwargs.pop`.

### Block 43 — Lines 115-115 (wrapper)
```python
115|             assert isinstance(tools_to_check, list), f"{tools_to_check=}"
```
**EN:** Asserts `isinstance(tools_to_check, list)` to enforce invariants.

**CN:** 断言 `isinstance(tools_to_check, list)` 以确保不变量成立。

### Block 44 — Lines 116-118 (wrapper)
```python
116|             assert all(tool in ComputeSanitizerTool for tool in tools_to_check), (
117|                 f"{(tool for tool in tools_to_check if tool not in ComputeSanitizerTool)=}")
118| 
```
**EN:** Asserts `all((tool in ComputeSanitizerTool for tool in tools_to_check))` to enforce invariants.

**CN:** 断言 `all((tool in ComputeSanitizerTool for tool in tools_to_check))` 以确保不变量成立。

### Block 45 — Lines 119-119 (wrapper)
```python
119|             ppid_name = psutil.Process(os.getppid()).exe()
```
**EN:** Assigns `ppid_name` and calls `psutil.Process(os.getppid()).exe`.

**CN:** 将 `ppid_name`，并调用 `psutil.Process(os.getppid()).exe`.

### Block 46 — Lines 120-120 (wrapper)
```python
120|             run_compute_sanitizer = target_kwargs.items() <= kwargs.items()
```
**EN:** Assigns `run_compute_sanitizer` and evaluates `target_kwargs.items() <= kwargs.items()`.

**CN:** 将 `run_compute_sanitizer` and 计算 `target_kwargs.items() <= kwargs.items()`.

### Block 47 — Lines 121-122 (wrapper)
```python
121|             if "run_sanitizer" in kwargs:
122|                 run_compute_sanitizer &= kwargs["run_sanitizer"]
```
**EN:** Checks `'run_sanitizer' in kwargs`..

**CN:** 检查 `'run_sanitizer' in kwargs`..

### Block 48 — Lines 123-187 (wrapper)
```python
123|             if run_compute_sanitizer and "compute-sanitizer" not in ppid_name:
124|                 for tool in tools_to_check:
125|                     path = os.path.realpath(test_fn.__globals__["__file__"])
126|                     # get path of current file
127|                     env = {
128|                         "PATH": os.environ["PATH"],
129|                         "PYTORCH_NO_CUDA_MEMORY_CACHING": "1",
130|                         "TORCH_SHOW_CPP_STACKTRACES": "1",
131|                         "CUDA_LAUNCH_BLOCKING": "1",
132|                     }
133|                     if "CUDA_VISIBLE_DEVICES" in os.environ:
134|                         env["CUDA_VISIBLE_DEVICES"] = os.environ["CUDA_VISIBLE_DEVICES"]
135|                     assert "request_fixture" in kwargs, (
136|                         "memcheck'ed test must have a (possibly unused) `request` fixture")
137|                     test_id = kwargs["request_fixture"].node.callspec.id
138|                     cmd = f"{path}::{test_fn.__name__}[{test_id}]"
139|                     cmd = [
140|                         "compute-sanitizer",
141|                         "--target-processes=application-only",
142|                         "--destroy-on-device-error=context",
143|                         f"--tool={tool.value}",
144|                         sys.executable,
145|                         "-m",
146|                         "pytest",
147|                         "-vsx",
148|                         cmd,
149|                     ]
150|                     for opt in ["--update_checksum", "--ignore_checksum_error"]:
151|                         if opt in sys.argv:
152|                             cmd.append(opt)
153|                     out = subprocess.run(
154|                         cmd,
155|                         stdout=subprocess.PIPE,
156|                         stderr=subprocess.STDOUT,
157|                         env=env,
158|                     )
159|                     sanitizer_ok = "ERROR SUMMARY: 0 errors" in str(
160|                         out.stdout) or "RACECHECK SUMMARY: 0 hazards displayed" in str(out.stdout)
161|                     test_output = out.stdout
162|                     if type(test_output) is bytes:
163|                         test_output = test_output.decode()
164| 
165|                     fail = False
166|                     if not sanitizer_ok:
167|                         print("compute-sanitizer returned an error")
168|                         fail = True
169|                     elif out.returncode != 0:
170|                         print(
171|                             "The test failed due to some other reason: consider running without compute-sanitizer to verify."
172|                         )
173|                         print(f"{out.returncode=}")
174|                         fail = True
175| 
176|                     if fail:
177|                         print("*****************************************************")
178|                         print("******************** TEST OUTPUT ********************")
179|                         print("*****************************************************")
180|                         print(test_output)
181|                         print("*****************************************************")
182|                         print("****************** TEST OUTPUT END ******************")
183|                         print("*****************************************************")
184|                         assert None
185|             else:
186|                 test_fn(*args, **kwargs)
187| 
```
**EN:** Checks `run_compute_sanitizer and 'compute-sanitizer' not in ppid_name`. The true branch mainly iterates over cases or dimensions, while the else branch invokes `test_fn`.

**CN:** 检查 `run_compute_sanitizer and 'compute-sanitizer' not in ppid_name`. 真分支主要遍历测试用例或维度；而 else 分支invokes `test_fn`.

### Block 49 — Lines 188-189 (decorator)
```python
188|         return wrapper
189| 
```
**EN:** Returns `wrapper`.

**CN:** 返回 `wrapper`.

### Block 50 — Lines 190-192 (compute_sanitizer)
```python
190|     return decorator
191| 
192| 
```
**EN:** Returns `decorator`.

**CN:** 返回 `decorator`.

### Block 51 — Lines 193-193 (compute_actual_scale)
```python
193| def compute_actual_scale(x, dtype, per_batch_scale=False):
```
**EN:** Defines function `compute_actual_scale(x, dtype, per_batch_scale)` for this module. The body mainly prepares intermediate values; prepares intermediate values; returns the computed result. It uses calls such as `x.abs().amax`, `x.abs().max`, `x.abs`, `tuple`, `range` to implement its workflow.

**CN:** 定义函数 `compute_actual_scale(x, dtype, per_batch_scale)`，供本模块使用. 主体主要准备中间值; 准备中间值; 返回计算结果. 其中会调用 `x.abs().amax`, `x.abs().max`, `x.abs`, `tuple`, `range` 来实现其工作流程.

### Block 52 — Lines 194-198 (compute_actual_scale)
```python
194|     max_finite = {
195|         torch.float8_e5m2: MAX_FINITE_FLOAT8E5,
196|         torch.float8_e4m3fn: MAX_FINITE_FLOAT8E4NV,
197|         torch.float8_e4m3fnuz: MAX_FINITE_FLOAT8E4B8,
198|     }[dtype]
```
**EN:** Assigns `max_finite` and evaluates `{torch.float8_e5m2: MAX_FINITE_FLOAT8E5, torch.float8_e4m3fn: MAX_F...`.

**CN:** 将 `max_finite` and 计算 `{torch.float8_e5m2: MAX_FINITE_FLOAT8E5, torch.float8_e4m3fn: MAX_F...`.

### Block 53 — Lines 199-199 (compute_actual_scale)
```python
199|     maxvals = x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_scale else x.abs().max()
```
**EN:** Assigns `maxvals` and uses conditional expression `x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_sc...`.

**CN:** 将 `maxvals` and 使用条件表达式 `x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_sc...`.

### Block 54 — Lines 200-205 (compute_actual_scale)
```python
200|     return maxvals / max_finite
201| 
202| 
203| # --- create tensor ---
204| 
205| 
```
**EN:** Returns `maxvals / max_finite`.

**CN:** 返回 `maxvals / max_finite`.

### Block 55 — Lines 206-206 (normalize_blocks)
```python
206| def normalize_blocks(x, BLOCK_SIZE=None):
```
**EN:** Defines function `normalize_blocks(x, BLOCK_SIZE)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; branches on runtime conditions. It uses calls such as `itertools.product`, `int`, `x.unsqueeze`, `range`, `min` to implement its workflow.

**CN:** 定义函数 `normalize_blocks(x, BLOCK_SIZE)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 根据运行时条件分支. 其中会调用 `itertools.product`, `int`, `x.unsqueeze`, `range`, `min` 来实现其工作流程.

### Block 56 — Lines 207-208 (normalize_blocks)
```python
207|     if BLOCK_SIZE is None:
208|         BLOCK_SIZE = int(MXFP_BLOCK_SIZE)
```
**EN:** Checks `BLOCK_SIZE is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `BLOCK_SIZE is None`. 真分支主要准备中间值.

### Block 57 — Lines 209-209 (normalize_blocks)
```python
209|     x_ndim = x.ndim
```
**EN:** Assigns `x_ndim` and references `x.ndim`.

**CN:** 将 `x_ndim` and 引用 `x.ndim`.

### Block 58 — Lines 210-211 (normalize_blocks)
```python
210|     if x_ndim == 2:
211|         x = x.unsqueeze(0)
```
**EN:** Checks `x_ndim == 2`. The true branch mainly prepares intermediate values.

**CN:** 检查 `x_ndim == 2`. 真分支主要准备中间值.

### Block 59 — Lines 212-226 (normalize_blocks)
```python
212|     for e, i, j in itertools.product(range(x.shape[0]), range(0, x.shape[1], BLOCK_SIZE),
213|                                      range(0, x.shape[2], BLOCK_SIZE)):
214|         i_end = min(i + BLOCK_SIZE, x.shape[1])
215|         j_end = min(j + BLOCK_SIZE, x.shape[2])
216|         block = x[e, i:i_end, j:j_end]
217|         m_abs = block.abs().max()
218|         i_len = i_end - i
219|         j_len = j_end - j
220|         min_len = min(i_len, j_len)
221|         signs = torch.randint(0, 2, (max(i_len, j_len), ), device=x.device) * 2 - 1
222|         block.diagonal(dim1=-2, dim2=-1)[:] = signs[:min_len] * m_abs
223|         if j_len > i_len:
224|             block[i_len - 1, i_len:] = signs[min_len:] * m_abs
225|         elif i_len > j_len:
226|             block[j_len:, j_len - 1] = signs[min_len:] * m_abs
```
**EN:** Loops over `itertools.product(range(x.shape[0]), range(0, x.shape[1],...` with target `(e, i, j)`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `itertools.product(range(x.shape[0]), range(0, x.shape[1],...` ，目标变量为 `(e, i, j)`. 循环体主要准备中间值; 准备中间值.

### Block 60 — Lines 227-228 (normalize_blocks)
```python
227|     if x_ndim == 2:
228|         x = x.squeeze(0)
```
**EN:** Checks `x_ndim == 2`. The true branch mainly prepares intermediate values.

**CN:** 检查 `x_ndim == 2`. 真分支主要准备中间值.

### Block 61 — Lines 229-231 (normalize_blocks)
```python
229|     return x
230| 
231| 
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

### Block 62 — Lines 232-232 (alloc_rand)
```python
232| def alloc_rand(shape, device, dtype, requires_grad=False):
```
**EN:** Defines function `alloc_rand(shape, device, dtype, requires_grad)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.randn`, `normalize_blocks`, `tmp.to(dtype).requires_grad_`, `torch.randint`, `tmp.to` to implement its workflow.

**CN:** 定义函数 `alloc_rand(shape, device, dtype, requires_grad)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `torch.randn`, `normalize_blocks`, `tmp.to(dtype).requires_grad_`, `torch.randint`, `tmp.to` 来实现其工作流程.

### Block 63 — Lines 233-235 (alloc_rand)
```python
233|     if dtype.itemsize == 1:
234|         tmp = 2**-(torch.randint(4, 8, shape, device=device, dtype=torch.float16))
235|         return tmp.to(dtype).requires_grad_(requires_grad)
```
**EN:** Checks `dtype.itemsize == 1`. The true branch mainly prepares intermediate values; returns the computed result.

**CN:** 检查 `dtype.itemsize == 1`. 真分支主要准备中间值; 返回计算结果.

### Block 64 — Lines 236-236 (alloc_rand)
```python
236|     ret = torch.randn(shape, device=device, dtype=dtype, requires_grad=requires_grad)
```
**EN:** Assigns `ret` and calls `torch.randn`.

**CN:** 将 `ret`，并调用 `torch.randn`.

### Block 65 — Lines 237-237 (alloc_rand)
```python
237|     ret = normalize_blocks(ret)
```
**EN:** Assigns `ret` and calls `normalize_blocks`.

**CN:** 将 `ret`，并调用 `normalize_blocks`.

### Block 66 — Lines 238-240 (alloc_rand)
```python
238|     return ret
239| 
240| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 67 — Lines 241-241 (make_slice_sizes)
```python
241| def make_slice_sizes(n_slices, total_size, device="cuda"):
```
**EN:** Defines function `make_slice_sizes(n_slices, total_size, device)` for this module. The body mainly invokes `torch.manual_seed`; prepares intermediate values; branches on runtime conditions. It uses calls such as `torch.manual_seed`, `torch.multinomial`, `torch.bincount(assignments, minlength...`, `ValueError`, `torch.zeros` to implement its workflow.

**CN:** 定义函数 `make_slice_sizes(n_slices, total_size, device)`，供本模块使用. 主体主要invokes `torch.manual_seed`; 准备中间值; 根据运行时条件分支. 其中会调用 `torch.manual_seed`, `torch.multinomial`, `torch.bincount(assignments, minlength...`, `ValueError`, `torch.zeros` 来实现其工作流程.

### Block 68 — Lines 242-242 (make_slice_sizes)
```python
242|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 69 — Lines 243-243 (make_slice_sizes)
```python
243|     dtype = torch.int32
```
**EN:** Assigns `dtype` and references `torch.int32`.

**CN:** 将 `dtype` and 引用 `torch.int32`.

### Block 70 — Lines 244-245 (make_slice_sizes)
```python
244|     if total_size < 0:
245|         raise ValueError("total_size must be non-negative")
```
**EN:** Checks `total_size < 0`..

**CN:** 检查 `total_size < 0`..

### Block 71 — Lines 246-247 (make_slice_sizes)
```python
246|     if n_slices <= 0:
247|         return torch.zeros((0, ), dtype=dtype, device=device)
```
**EN:** Checks `n_slices <= 0`. The true branch mainly returns the computed result.

**CN:** 检查 `n_slices <= 0`. 真分支主要返回计算结果.

### Block 72 — Lines 248-250 (make_slice_sizes)
```python
248|     if total_size == 0:
249|         return torch.zeros((n_slices, ), dtype=dtype, device=device)
250|     # always set one slice size to zero
```
**EN:** Checks `total_size == 0`. The true branch mainly returns the computed result.

**CN:** 检查 `total_size == 0`. 真分支主要返回计算结果.

### Block 73 — Lines 251-251 (make_slice_sizes)
```python
251|     probs = torch.ones(n_slices, device=device) / n_slices
```
**EN:** Assigns `probs` and evaluates `torch.ones(n_slices, device=device) / n_slices`.

**CN:** 将 `probs` and 计算 `torch.ones(n_slices, device=device) / n_slices`.

### Block 74 — Lines 252-254 (make_slice_sizes)
```python
252|     if n_slices > 1:
253|         probs[2] += probs[1]
254|         probs[1] = 0.
```
**EN:** Checks `n_slices > 1`. The true branch mainly prepares intermediate values.

**CN:** 检查 `n_slices > 1`. 真分支主要准备中间值.

### Block 75 — Lines 255-255 (make_slice_sizes)
```python
255|     assignments = torch.multinomial(probs, total_size, replacement=True)
```
**EN:** Assigns `assignments` and calls `torch.multinomial`.

**CN:** 将 `assignments`，并调用 `torch.multinomial`.

### Block 76 — Lines 256-256 (make_slice_sizes)
```python
256|     counts = torch.bincount(assignments, minlength=n_slices).to(dtype)
```
**EN:** Assigns `counts` and calls `torch.bincount(assignments, minlength=n_slices).to`.

**CN:** 将 `counts`，并调用 `torch.bincount(assignments, minlength=n_slices).to`.

### Block 77 — Lines 257-257 (make_slice_sizes)
```python
257|     assert counts.sum().item() == total_size
```
**EN:** Asserts `counts.sum().item() == total_size` to enforce invariants.

**CN:** 断言 `counts.sum().item() == total_size` 以确保不变量成立。

### Block 78 — Lines 258-258 (make_slice_sizes)
```python
258|     assert len(counts) == n_slices
```
**EN:** Asserts `len(counts) == n_slices` to enforce invariants.

**CN:** 断言 `len(counts) == n_slices` 以确保不变量成立。

### Block 79 — Lines 259-261 (make_slice_sizes)
```python
259|     return counts
260| 
261| 
```
**EN:** Returns `counts`.

**CN:** 返回 `counts`.

### Block 80 — Lines 262-262 (pad_rows_to_multiples)
```python
262| def pad_rows_to_multiples(A, indices, multiple=128, pad_value=float('nan')):
```
**EN:** Defines function `pad_rows_to_multiples(A, indices, multiple, pad_value)` for this module. The body mainly prepares intermediate values; prepares intermediate values; iterates over cases or dimensions. It uses calls such as `float`, `A.size`, `zip`, `torch.vstack`, `torch.full` to implement its workflow.

**CN:** 定义函数 `pad_rows_to_multiples(A, indices, multiple, pad_value)`，供本模块使用. 主体主要准备中间值; 准备中间值; 遍历测试用例或维度. 其中会调用 `float`, `A.size`, `zip`, `torch.vstack`, `torch.full` 来实现其工作流程.

### Block 81 — Lines 263-266 (pad_rows_to_multiples)
```python
263|     """
264|     Insert padding so that each row A[i] (for i in indices)
265|     appears at an output row index that is a multiple of `multiple`.
266|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 82 — Lines 267-267 (pad_rows_to_multiples)
```python
267|     D = A.size(1)
```
**EN:** Assigns `D` and calls `A.size`.

**CN:** 将 `D`，并调用 `A.size`.

### Block 83 — Lines 268-268 (pad_rows_to_multiples)
```python
268|     out = []
```
**EN:** Assigns `out` and builds a list.

**CN:** 将 `out` and 构造一个列表.

### Block 84 — Lines 269-274 (pad_rows_to_multiples)
```python
269|     for i_cur, i_next in zip(indices[:-1], indices[1:]):
270|         size = (i_next - i_cur)
271|         size_padded = ((size + multiple - 1) // multiple) * multiple
272|         cur = torch.full((size_padded, D), pad_value, dtype=A.dtype, device=A.device)
273|         cur[:size, :] = A[i_cur:i_next, :]
274|         out.append(cur)
```
**EN:** Loops over `zip(indices[:-1], indices[1:])` with target `(i_cur, i_next)`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `zip(indices[:-1], indices[1:])` ，目标变量为 `(i_cur, i_next)`. 循环体主要准备中间值; 准备中间值.

### Block 85 — Lines 275-277 (pad_rows_to_multiples)
```python
275|     return torch.vstack(out)
276| 
277| 
```
**EN:** Returns `torch.vstack(out)`.

**CN:** 返回 `torch.vstack(out)`.

### Block 86 — Lines 278-278 (pad_ragged_tensor)
```python
278| def pad_ragged_tensor(x, x_ragged_metadata, hbm_swizzling, transpose):
```
**EN:** Defines function `pad_ragged_tensor(x, x_ragged_metadata, hbm_swizzling, transpose)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; prepares intermediate values. It uses calls such as `replace`, `pad_rows_to_multiples(x.T, x_ragged_m...`, `pad_rows_to_multiples(x, x_ragged_met...`, `pad_rows_to_multiples`, `x_ragged_metadata.block_offs` to implement its workflow.

**CN:** 定义函数 `pad_ragged_tensor(x, x_ragged_metadata, hbm_swizzling, transpose)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 准备中间值. 其中会调用 `replace`, `pad_rows_to_multiples(x.T, x_ragged_m...`, `pad_rows_to_multiples(x, x_ragged_met...`, `pad_rows_to_multiples`, `x_ragged_metadata.block_offs` 来实现其工作流程.

### Block 87 — Lines 279-279 (pad_ragged_tensor)
```python
279|     multiple = 128 if hbm_swizzling else 64
```
**EN:** Assigns `multiple` and uses conditional expression `128 if hbm_swizzling else 64`.

**CN:** 将 `multiple` and 使用条件表达式 `128 if hbm_swizzling else 64`.

### Block 88 — Lines 280-284 (pad_ragged_tensor)
```python
280|     if transpose:
281|         y = pad_rows_to_multiples(x.T, x_ragged_metadata.slice_offs, multiple=multiple, pad_value=0).T.contiguous()
282|     else:
283|         y = pad_rows_to_multiples(x, x_ragged_metadata.slice_offs, multiple=multiple, pad_value=0).contiguous()
284| 
```
**EN:** Checks `transpose`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `transpose`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 89 — Lines 285-286 (pad_ragged_tensor)
```python
285|     y_ragged_metadata = replace(x_ragged_metadata, slice_offs=x_ragged_metadata.block_offs(multiple) * multiple,
286|                                 slice_sizes_divisibility=multiple)
```
**EN:** Assigns `y_ragged_metadata` and calls `replace`.

**CN:** 将 `y_ragged_metadata`，并调用 `replace`.

### Block 90 — Lines 287-289 (pad_ragged_tensor)
```python
287|     return y, y_ragged_metadata
288| 
289| 
```
**EN:** Returns `(y, y_ragged_metadata)`.

**CN:** 返回 `(y, y_ragged_metadata)`.

### Block 91 — Lines 290-292 (make_random_tensor)
```python
290| def make_random_tensor(shape, n_slices, ragged_dim, ragged_padding, device, dtype, mxfp_dim, transpose,
291|                        squeeze_batch_dim, is_mx_rowmajor=False, value_hbm_swizzling=None, scale_hbm_swizzling=None):
292|     # allocate buffer
```
**EN:** Defines function `make_random_tensor(shape, n_slices, ragged_dim, ragged_padding, device, dtype, mxfp_dim, transpose, squeeze_batch_dim, is_mx_rowmajor, value_hbm_swizzling, scale_hbm_swizzling)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `alloc_rand`, `buffer.squeeze`, `make_slice_sizes`, `make_ragged_tensor_metadata`, `pad_ragged_tensor` to implement its workflow.

**CN:** 定义函数 `make_random_tensor(shape, n_slices, ragged_dim, ragged_padding, device, dtype, mxfp_dim, transpose, squeeze_batch_dim, is_mx_rowmajor, value_hbm_swizzling, scale_hbm_swizzling)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `alloc_rand`, `buffer.squeeze`, `make_slice_sizes`, `make_ragged_tensor_metadata`, `pad_ragged_tensor` 来实现其工作流程.

### Block 92 — Lines 293-293 (make_random_tensor)
```python
293|     buffer_shape = ((n_slices, ) if ragged_dim is None else tuple()) + shape
```
**EN:** Assigns `buffer_shape` and evaluates `((n_slices,) if ragged_dim is None else tuple()) + shape`.

**CN:** 将 `buffer_shape` and 计算 `((n_slices,) if ragged_dim is None else tuple()) + shape`.

### Block 93 — Lines 294-294 (make_random_tensor)
```python
294|     buffer_dtype = torch.bfloat16 if dtype.has_mx_scale else dtype.torch_dtype
```
**EN:** Assigns `buffer_dtype` and uses conditional expression `torch.bfloat16 if dtype.has_mx_scale else dtype.torch_dtype`.

**CN:** 将 `buffer_dtype` and 使用条件表达式 `torch.bfloat16 if dtype.has_mx_scale else dtype.torch_dtype`.

### Block 94 — Lines 295-295 (make_random_tensor)
```python
295|     buffer = alloc_rand(buffer_shape, device=device, dtype=buffer_dtype)
```
**EN:** Assigns `buffer` and calls `alloc_rand`.

**CN:** 将 `buffer`，并调用 `alloc_rand`.

### Block 95 — Lines 296-298 (make_random_tensor)
```python
296|     if squeeze_batch_dim:
297|         buffer = buffer.squeeze(0)
298|     # handle raggedness
```
**EN:** Checks `squeeze_batch_dim`. The true branch mainly prepares intermediate values.

**CN:** 检查 `squeeze_batch_dim`. 真分支主要准备中间值.

### Block 96 — Lines 299-299 (make_random_tensor)
```python
299|     ragged_metadata = None
```
**EN:** Assigns `ragged_metadata` and stores constant `None`.

**CN:** 将 `ragged_metadata` and 保存常量 `None`.

### Block 97 — Lines 300-302 (make_random_tensor)
```python
300|     if ragged_dim is not None:
301|         slice_sizes = make_slice_sizes(n_slices, shape[ragged_dim], device=device)
302|         ragged_metadata = make_ragged_tensor_metadata(slice_sizes, shape[ragged_dim])
```
**EN:** Checks `ragged_dim is not None`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `ragged_dim is not None`. 真分支主要准备中间值; 准备中间值.

### Block 98 — Lines 303-306 (make_random_tensor)
```python
303|     if ragged_padding:
304|         buffer, ragged_metadata = pad_ragged_tensor(buffer, ragged_metadata, value_hbm_swizzling is not None
305|                                                     or scale_hbm_swizzling is not None, ragged_dim == 1)
306|     # handle transpose
```
**EN:** Checks `ragged_padding`. The true branch mainly prepares intermediate values.

**CN:** 检查 `ragged_padding`. 真分支主要准备中间值.

### Block 99 — Lines 307-309 (make_random_tensor)
```python
307|     if transpose:
308|         buffer = buffer.mT.contiguous().mT
309|     # handle mxfp
```
**EN:** Checks `transpose`. The true branch mainly prepares intermediate values.

**CN:** 检查 `transpose`. 真分支主要准备中间值.

### Block 100 — Lines 310-310 (make_random_tensor)
```python
310|     scales = None
```
**EN:** Assigns `scales` and stores constant `None`.

**CN:** 将 `scales` and 保存常量 `None`.

### Block 101 — Lines 311-354 (make_random_tensor)
```python
311|     if mxfp_dim is not None:
312|         assert dtype.has_mx_scale
313|         is_nvfp4 = getattr(dtype, "is_nvfp4", False)
314|         scale_dtype = torch.float8_e4m3fn if is_nvfp4 else torch.uint8
315|         microblock_size = NVFP_BLOCK_SIZE.value if is_nvfp4 else MXFP_BLOCK_SIZE.value
316|         if dtype.is_mxfloat4:
317|             logical_shape = list(buffer.shape)
318|         if is_mx_rowmajor:
319|             scales = downcast_to_mxfp(
320|                 buffer,
321|                 dtype.torch_dtype,
322|                 axis=mxfp_dim,
323|                 scale_dtype=scale_dtype,
324|                 microblock_size=microblock_size,
325|             )[1]
326|             buffer = downcast_to_mxfp(
327|                 buffer.mT.contiguous(),
328|                 dtype.torch_dtype,
329|                 axis=mxfp_dim,
330|                 scale_dtype=scale_dtype,
331|                 microblock_size=microblock_size,
332|             )[0].mT
333|         else:
334|             buffer, scales = downcast_to_mxfp(
335|                 buffer,
336|                 dtype.torch_dtype,
337|                 axis=mxfp_dim,
338|                 scale_dtype=scale_dtype,
339|                 microblock_size=microblock_size,
340|             )
341|         buffer = wrap_torch_tensor(
342|             buffer,
343|             FP4 if dtype.is_mxfloat4 else None,
344|             shape=logical_shape if dtype.is_mxfloat4 else None,
345|         )
346|         scales = wrap_torch_tensor(scales)
347|         if value_hbm_swizzling is not None:
348|             # convert buffer to swizzled hbm layout
349|             buffer = convert_layout(buffer, value_hbm_swizzling)
350|         if scale_hbm_swizzling is not None:
351|             # hack to avoid circular dependency
352|             if callable(scale_hbm_swizzling):
353|                 scale_hbm_swizzling = scale_hbm_swizzling(ragged_metadata)
354|             scales = convert_layout(scales, scale_hbm_swizzling)
```
**EN:** Checks `mxfp_dim is not None`. The true branch mainly checks invariants; prepares intermediate values.

**CN:** 检查 `mxfp_dim is not None`. 真分支主要检查不变量; 准备中间值.

### Block 102 — Lines 355-355 (make_random_tensor)
```python
355|     return buffer, scales, ragged_metadata
```
**EN:** Returns `(buffer, scales, ragged_metadata)`.

**CN:** 返回 `(buffer, scales, ragged_metadata)`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `assert_equal`, `assert_close`, `ComputeSanitizerTool`, `compute_sanitizer`, `compute_actual_scale`, `normalize_blocks`, `alloc_rand`, `make_slice_sizes`.
  **CN:** 主要符号：`assert_equal`, `assert_close`, `ComputeSanitizerTool`, `compute_sanitizer`, `compute_actual_scale`, `normalize_blocks`, `alloc_rand`, `make_slice_sizes`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `enum`, `functools`, `os`, `subprocess`, `sys`, `torch`, `itertools`, `dataclasses (replace)`.
  **CN:** 外部模块：`enum`, `functools`, `os`, `subprocess`, `sys`, `torch`, `itertools`, `dataclasses (replace)`。
- **EN:** Internal modules: `triton_kernels.numerics (MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, make_ragged_tensor_metadata)`, `triton_kernels.numerics_details.mxfp (downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE)`.
  **CN:** 内部模块：`triton_kernels.numerics (MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, make_ragged_tensor_metadata)`, `triton_kernels.numerics_details.mxfp (downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE)`。

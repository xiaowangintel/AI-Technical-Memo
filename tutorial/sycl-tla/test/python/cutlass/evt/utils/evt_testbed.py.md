# evt_testbed.py — Code Analysis / 代码分析

## Source / 来源
- Path / 路径: `test/python/cutlass/evt/utils/evt_testbed.py`
- **EN:** Defines the shared reference model, launcher, tensor factory, and base testcase utilities used by the EVT test modules.
- **CN:** 定义 EVT 测试模块共用的参考模型、启动器、张量工厂以及基础 testcase 工具。

## Line-by-Line Analysis / 逐行分析

### Lines 33-45
```python
"""
Testbed classes of EVT
"""

import torch
import unittest

import cutlass_cppgen
from cutlass_cppgen import Tensor
import cutlass_cppgen.backend.evt
from cutlass_cppgen.shape import GemmCoord
from cutlass_cppgen.utils.datatypes import torch_type
from cutlass_cppgen.utils.profiler import CUDAEventProfiler
```
**EN:** This opening block establishes the module context. The module docstring states the purpose: Testbed classes of EVT It imports the libraries needed for code generation, testing, and tensor/runtime support.
**CN:** 这一开头代码块建立了模块的上下文。 模块文档字符串直接说明了用途：Testbed classes of EVT 它导入了代码生成、测试以及张量/运行时支持所需的库。

### Line 48
```python
class EVTReferenceModule:
```
**EN:** `EVTReferenceModule` is the main class introduced in this block.
**CN:** `EVTReferenceModule` 是此处引入的主要类。

### Lines 49-53
```python
    def __init__(self, layout_A, layout_B, layout_C, epilogue_visitor):
        self.layout_A = layout_A
        self.layout_B = layout_B
        self.layout_C = layout_C
        self.epilogue_visitor = epilogue_visitor
```
**EN:** `__init__(self, layout_A, layout_B, layout_C, epilogue_visitor)` is a method inside `EVTReferenceModule`. Key steps in this block are `self.layout_A = layout_A; self.layout_B = layout_B; self.layout_C = layout_C; self.epilogue_visitor = epilogue_visitor`.
**CN:** `__init__(self, layout_A, layout_B, layout_C, epilogue_visitor)` 是一个位于 `EVTReferenceModule` 内部的方法。 此代码块中的关键步骤是 `self.layout_A = layout_A; self.layout_B = layout_B; self.layout_C = layout_C; self.epilogue_visitor = epilogue_visitor`。

### Lines 55-81
```python
    def run(self, A, B, C, problem_size, alpha, beta, batch=1):
        if self.layout_A == cutlass_cppgen.LayoutType.RowMajor:
            A_row = A.view((batch, problem_size.m, problem_size.k))
        else:
            A_col = A.view((batch, problem_size.k, problem_size.m))
            A_row = torch.permute(A_col, (0, 2, 1))

        if self.layout_B == cutlass_cppgen.LayoutType.RowMajor:
            B_row = B.view((batch, problem_size.k, problem_size.n))
        else:
            B_col = B.view((batch, problem_size.n, problem_size.k))
            B_row = torch.permute(B_col, (0, 2, 1))

        if self.layout_C == cutlass_cppgen.LayoutType.RowMajor:
            C_row = C.view((batch, problem_size.m, problem_size.n))
        else:
            C_col = C.view((batch, problem_size.n, problem_size.m))
            C_row = torch.permute(C_col, (0, 2, 1))

        out_row = torch.matmul(A_row, B_row) * alpha + C_row * beta

        if self.layout_C == cutlass_cppgen.LayoutType.ColumnMajor:
            out = torch.permute(out_row, (0, 2, 1))
        else:
            out = out_row

        return torch.flatten(out)
```
**EN:** `run(self, A, B, C, problem_size, alpha, beta, batch)` is a method inside `EVTReferenceModule`. Key steps in this block are `out_row = torch.matmul(A_row, B_row) * alpha + C_row * beta; return torch.flatten(out)`. The implementation reshapes/layout-normalizes tensors so PyTorch matmul can serve as the reference GEMM path.
**CN:** `run(self, A, B, C, problem_size, alpha, beta, batch)` 是一个位于 `EVTReferenceModule` 内部的方法。 此代码块中的关键步骤是 `out_row = torch.matmul(A_row, B_row) * alpha + C_row * beta; return torch.flatten(out)`。 这里会先对张量做 reshape/布局归一化，再用 PyTorch matmul 作为参考 GEMM 路径。

### Lines 83-119
```python
    def __call__(self, A, B, C, problem_size, batch=1, epilogue_args=None):
        # Running the mainloop
        accum = self.run(
            A, B, C, problem_size, 1.0, 0.0, batch=batch
        ).reshape(batch, problem_size.m, problem_size.n)
        
        # Running the epilogue with optional mixed-precision computation
        epilogue_args["accum"] = accum
        if accum.dtype in [torch.float16, torch.bfloat16]:
            # Mixed-precision mode: compute in FP32, convert back to original dtype
            # This provides more accurate reference by reducing intermediate rounding errors
            original_dtype = accum.dtype

            # Convert all tensor inputs to FP32
            epilogue_args_fp32 = {}
            for key, value in epilogue_args.items():
                if isinstance(value, torch.Tensor) and value.dtype in [torch.float16, torch.bfloat16]:
                    epilogue_args_fp32[key] = value.to(torch.float32)
                else:
                    epilogue_args_fp32[key] = value

            # Compute reference in FP32
            references = self.epilogue_visitor(**epilogue_args_fp32)

            # Convert results back to original dtype
            if not isinstance(references, tuple):
                references = (references,)
            references = tuple(
                ref.to(original_dtype) if isinstance(ref, torch.Tensor) else ref
                for ref in references
            )
        else:
            # Standard mode: compute in native dtype
            references = self.epilogue_visitor(**epilogue_args)
            if not isinstance(references, tuple):
                references = (references,)
        return references
```
**EN:** `__call__(self, A, B, C, problem_size, batch, epilogue_args)` is a method inside `EVTReferenceModule`. Key steps in this block are `accum = self.run(A, B, C, problem_size, 1.0, 0.0, batch=batch).reshape(batch, problem_size.m, problem_size.n); epilogue_args['accum'] = accum; return references`. The surrounding control flow iterates over `epilogue_args.items()` to cover multiple shapes/configurations.
**CN:** `__call__(self, A, B, C, problem_size, batch, epilogue_args)` 是一个位于 `EVTReferenceModule` 内部的方法。 此代码块中的关键步骤是 `accum = self.run(A, B, C, problem_size, 1.0, 0.0, batch=batch).reshape(batch, problem_size.m, problem_size.n); epilogue_args['accum'] = accum; return references`。 外层控制流会遍历 `epilogue_args.items()`，以覆盖多种形状或配置。

### Lines 122-125
```python
class EVTTestBed:
    """
    Epilogue Visitor Testbed
    """
```
**EN:** `EVTTestBed` is the main class introduced in this block.
**CN:** `EVTTestBed` 是此处引入的主要类。

### Lines 126-149
```python
    def __init__(self, element, evt_fn, example_inputs, profile=False, **kwargs) -> None:
        self.element = element
        layout = cutlass_cppgen.LayoutType.RowMajor
        self.example_inputs = example_inputs
        
        # Create the Gemm plan
        self.plan = cutlass_cppgen.op.Gemm(element=element, layout=layout, element_accumulator=torch.float32)
        
        if "tile_description" in kwargs:
            self.plan.tile_description = kwargs["tile_description"]
        
        if "swizzling_functor" in kwargs:
            self.plan.swizzling_functor = kwargs["swizzling_functor"]
        
        # Compile the epilogue visitor
        epilogue_visitor = cutlass_cppgen.epilogue.trace(evt_fn, example_inputs)
        if "epilogue_stages" in kwargs:
            epilogue_visitor.epilogue_stages = kwargs["epilogue_stages"]
        self.plan.epilogue_visitor = epilogue_visitor
        
        # Reference model
        self.reference_fn = EVTReferenceModule(layout, layout, layout, epilogue_visitor)
        
        self.profile = profile
```
**EN:** `__init__(self, element, evt_fn, example_inputs, profile, **kwargs)` is a method inside `EVTTestBed`. Key steps in this block are `self.element = element; layout = cutlass_cppgen.LayoutType.RowMajor; self.example_inputs = example_inputs; self.plan = cutlass_cppgen.op.Gemm(element=element, layout=layout, element_accumulator=torch.float32)`.
**CN:** `__init__(self, element, evt_fn, example_inputs, profile, **kwargs)` 是一个位于 `EVTTestBed` 内部的方法。 此代码块中的关键步骤是 `self.element = element; layout = cutlass_cppgen.LayoutType.RowMajor; self.example_inputs = example_inputs; self.plan = cutlass_cppgen.op.Gemm(element=element, layout=layout, element_accumulator=torch.float32)`。

### Lines 151-167
```python
    def get_torch_tensor(self, shape, dtype=None, fill=None):
        if dtype is None:
            dtype = self.element
        
        dtype = torch_type(dtype)
        if torch.cuda.is_available():
            device="cuda"
        elif torch.xpu.is_available():
            device="xpu"
        else:
            device="cpu"
        if fill is None:
            return torch.ceil(
                torch.empty(size=shape, dtype=dtype, device=device).uniform_(-4.5, 3.5)
            )
        else:
            return torch.full(shape, fill, dtype=dtype, device=device)
```
**EN:** `get_torch_tensor(self, shape, dtype, fill)` is a method inside `EVTTestBed`. Key steps in this block are `dtype = torch_type(dtype)`. Device selection is dynamic: CUDA is preferred, then XPU, and finally CPU as a fallback.
**CN:** `get_torch_tensor(self, shape, dtype, fill)` 是一个位于 `EVTTestBed` 内部的方法。 此代码块中的关键步骤是 `dtype = torch_type(dtype)`。 设备选择是动态的：优先 CUDA，其次 XPU，最后退回 CPU。

### Lines 169-235
```python
    def verify(self, problem_size, input_keys, result_keys, batch_count=1):
        """
        Verify the results
        """
        problem_size = GemmCoord(*problem_size)

        # Initiate the GEMM arguments
        tensor_A = self.get_torch_tensor((batch_count, problem_size.m, problem_size.k))
        tensor_B = self.get_torch_tensor((batch_count, problem_size.k, problem_size.n))
        
        # Initialize the epilogue args
        epilogue_args = {}
        for key in self.example_inputs.keys():
            if key in input_keys:
                tensor = self.example_inputs[key]
                if isinstance(tensor, Tensor):
                    epilogue_args[key] = self.get_torch_tensor(tensor.shape, tensor.element)
                else:
                    epilogue_args[key] = tensor
            elif key in result_keys:
                tensor = self.example_inputs[key]
                if isinstance(tensor, Tensor):
                    if "max" in key:
                        fill = -1000
                    else:
                        fill = 0
                    epilogue_args[key] = self.get_torch_tensor(tensor.shape, tensor.element, fill=fill)
                else:
                    epilogue_args[key] = tensor
        
        tensor_D = epilogue_args["D"]
        if "C" in epilogue_args:
            tensor_C = epilogue_args["C"]
        else:
            tensor_C = tensor_D
        # Run the device kernel
        self.plan.run(tensor_A, tensor_B, tensor_C, tensor_D, visitor_args=epilogue_args)
        
        # Run the host reference
        evt_args_inputs = {}
        for key in input_keys:
            evt_args_inputs[key] = epilogue_args[key]
        
        reference_results = self.reference_fn(
            tensor_A, tensor_B, tensor_C, problem_size, batch_count, evt_args_inputs)
        
        # Compare the results
        for result, ref in zip(result_keys, reference_results):
            # Use torch.testing.assert_close() instead of torch.equal()
            # because floating-point arithmetic can introduce tiny numerical errors due to limited
            # precision representation. torch.equal() requires exact bit-for-bit equality which often
            # fails for mathematically equivalent results.
            # torch.testing.assert_close() provides configurable relative/absolute tolerances (rtol/atol)
            # to handle these precision issues, automatically sets appropriate default tolerances based
            # on tensor datatypes (e.g., stricter tolerances for float64 vs float32), and gives detailed
            # error messages when assertions fail, making it much more suitable for robust numerical testing.
            torch.testing.assert_close(
                epilogue_args[result].flatten(), 
                ref.masked_fill(torch.isnan(ref), float('inf')).flatten(), check_dtype=False)
        
        # Run profile
        if self.profile:
            profiler = CUDAEventProfiler(
                self.plan, 100, 100, tensor_A, tensor_B, tensor_C, tensor_D,
                visitor_args = epilogue_args
            )
            print(f"Cutlass Python Duration: {profiler()}")
```
**EN:** `verify(self, problem_size, input_keys, result_keys, batch_count)` is a method inside `EVTTestBed`. The docstring describes it as: Verify the results Key steps in this block are `problem_size = GemmCoord(*problem_size); tensor_A = self.get_torch_tensor((batch_count, problem_size.m, problem_size.k)); tensor_B = self.get_torch_tensor((batch_count, problem_size.k, problem_size.n)); epilogue_args = {}`. The surrounding control flow iterates over `self.example_inputs.keys()` to cover multiple shapes/configurations. Validation uses `torch.testing.assert_close` so dtype differences and small numerical noise are tolerated.
**CN:** `verify(self, problem_size, input_keys, result_keys, batch_count)` 是一个位于 `EVTTestBed` 内部的方法。 文档字符串将其描述为：Verify the results 此代码块中的关键步骤是 `problem_size = GemmCoord(*problem_size); tensor_A = self.get_torch_tensor((batch_count, problem_size.m, problem_size.k)); tensor_B = self.get_torch_tensor((batch_count, problem_size.k, problem_size.n)); epilogue_args = {}`。 外层控制流会遍历 `self.example_inputs.keys()`，以覆盖多种形状或配置。 验证使用 `torch.testing.assert_close`，因此可以容忍 dtype 差异和细微数值噪声。

### Lines 238-241
```python
class EVTTestCaseBase(unittest.TestCase):
    """
    Base class for EVT Unittest
    """
```
**EN:** `EVTTestCaseBase` is the main class introduced in this block. It inherits from `unittest.TestCase`, so the class reuses shared helpers or unittest behavior.
**CN:** `EVTTestCaseBase` 是此处引入的主要类。 它继承自 `unittest.TestCase`，因此会复用共享辅助逻辑或 unittest 行为。

### Lines 242-250
```python
    def __init__(self, methodName: str = "runTest", lmnk=(6, 512, 256, 128)) -> None:
        super().__init__(methodName)
        
        self.element = cutlass_cppgen.DataType.f16
        self.l, self.m, self.n, self.k = lmnk
        
        self.problem_size = (self.m, self.n, self.k)
        
        torch.random.manual_seed(42)
```
**EN:** `__init__(self, methodName, lmnk)` is a method inside `EVTTestCaseBase`. Key steps in this block are `super().__init__(methodName); self.element = cutlass_cppgen.DataType.f16; (self.l, self.m, self.n, self.k) = lmnk; self.problem_size = (self.m, self.n, self.k)`.
**CN:** `__init__(self, methodName, lmnk)` 是一个位于 `EVTTestCaseBase` 内部的方法。 此代码块中的关键步骤是 `super().__init__(methodName); self.element = cutlass_cppgen.DataType.f16; (self.l, self.m, self.n, self.k) = lmnk; self.problem_size = (self.m, self.n, self.k)`。

### Lines 252-256
```python
    def fake_tensor(self, element, shape, stride=None):
        if stride is None:
            return Tensor(element=element, shape=shape, layout_tag=cutlass_cppgen.LayoutType.RowMajor)
        else:
            return Tensor(element=element, shape=shape, stride=stride)
```
**EN:** `fake_tensor(self, element, shape, stride)` is a method inside `EVTTestCaseBase`.
**CN:** `fake_tensor(self, element, shape, stride)` 是一个位于 `EVTTestCaseBase` 内部的方法。

### Lines 258-272
```python
    def get_problem_sizes(self, alignment, k=None, batch_count=[3,]):
        k = k if k else self.k
        problem_size_m = [alignment, 512 - 3 * alignment]
        problem_size_n = [alignment, 512 - alignment]
        if alignment % 8 == 0:
            problem_size_m.append(768)
            problem_size_n.append(768)
        problem_size_l = batch_count
        problem_sizes = []
        for m in problem_size_m:
            for n in problem_size_n:
                for l in problem_size_l:
                    problem_sizes.append((m, n, k, l))
        
        return problem_sizes
```
**EN:** `get_problem_sizes(self, alignment, k, batch_count)` is a method inside `EVTTestCaseBase`. Key steps in this block are `k = k if k else self.k; problem_size_m = [alignment, 512 - 3 * alignment]; problem_size_n = [alignment, 512 - alignment]; problem_size_l = batch_count`. The surrounding control flow iterates over `problem_size_m` to cover multiple shapes/configurations.
**CN:** `get_problem_sizes(self, alignment, k, batch_count)` 是一个位于 `EVTTestCaseBase` 内部的方法。 此代码块中的关键步骤是 `k = k if k else self.k; problem_size_m = [alignment, 512 - 3 * alignment]; problem_size_n = [alignment, 512 - alignment]; problem_size_l = batch_count`。 外层控制流会遍历 `problem_size_m`，以覆盖多种形状或配置。

## Key Concepts / 关键概念
- **EN:** Reference execution using PyTorch matmul plus traced epilogue visitors.
  **CN:** 使用 PyTorch matmul 与 trace 后的 epilogue visitor 做参考执行。
- **EN:** CUTLASS GEMM plan construction and invocation.
  **CN:** CUTLASS GEMM plan 的构建与调用。
- **EN:** Tensor allocation helpers that choose CUDA/XPU/CPU at runtime.
  **CN:** 运行时自动选择 CUDA/XPU/CPU 的张量分配辅助逻辑。
- **EN:** Numerical validation with `torch.testing.assert_close`.
  **CN:** 使用 `torch.testing.assert_close` 进行数值验证。

## Dependencies / 依赖关系
- `import torch`
  - **EN:** Supplies tensor allocation, math kernels, and the numerical reference implementation.
  - **CN:** 提供张量分配、数学内核以及数值参考实现。
- `import unittest`
  - **EN:** Supplies the Python unit-test framework and result/runner abstractions.
  - **CN:** 提供 Python 单元测试框架以及结果/运行器抽象。
- `import cutlass_cppgen`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。
- `from cutlass_cppgen import Tensor`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。
- `import cutlass_cppgen.backend.evt`
  - **EN:** Brings in backend helpers such as architecture queries, data types, and convenience symbols used by the tests.
  - **CN:** 引入后端辅助内容，如架构查询、数据类型以及测试中使用的便捷符号。
- `from cutlass_cppgen.shape import GemmCoord`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。
- `from cutlass_cppgen.utils.datatypes import torch_type`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。
- `from cutlass_cppgen.utils.profiler import CUDAEventProfiler`
  - **EN:** Provides the main CUTLASS Python API used to build plans, trace visitors, or emit code.
  - **CN:** 提供主要的 CUTLASS Python API，用于构建 plan、trace visitor 或发射代码。

# benchmark.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/tensorexpr/benchmark.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9 / 第 1-9 行

```python
import contextlib
import json
import os
import time

import numpy as np

import torch

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 10-22 / 第 10-22 行

```python
from . import tensor_engine


class Benchmark:
    def __init__(self, mode, device, dtype):
        self.mode = mode
        self.deterministic = False
        self.device = device
        self.dtype = dtype
        self.output_type = "stdout"
        self.print_ir = False
        self.print_kernel = False
        if mode == "both":
```

- **EN:** Important local symbols in this block include Benchmark, __init__.
- **CN:** 该代码块中的重要局部符号包括 Benchmark、__init__。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 23-32 / 第 23-32 行

```python
            self.requires_grad = True
        elif mode == "fwd":
            self.requires_grad = False
        else:
            raise ValueError(f"invalid mode: {mode}")
        self.result_grad = None
        self.grad_variables = []
        self.engine = tensor_engine.get_engine()
        self.engine.reset(device)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 33-41 / 第 33-41 行

```python
        # forward all member functions in self.engine to self
        for method in dir(self.engine):
            if not callable(getattr(self.engine, method)):
                continue
            # don't forward if this function is overridden here
            if hasattr(self, method):
                continue
            # don't forward if it is a internal function
            if method.startswith("_"):
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 42-49 / 第 42-49 行

```python
                continue
            method_engine = getattr(self.engine, method)
            setattr(self, method, method_engine)

    def forward(self):
        """do one step worth of computation"""
        raise ValueError("this method should be reimplemented by subclass")

```

- **EN:** Important local symbols in this block include forward.
- **CN:** 该代码块中的重要局部符号包括 forward。

### Lines 50-57 / 第 50-57 行

```python
    def check(self):
        if not self.deterministic:
            return
        np.testing.assert_allclose(
            self.reference(), self.numpy(self.compute()), atol=1e-2
        )

    def config(self):
```

- **EN:** Important local symbols in this block include check, config.
- **CN:** 该代码块中的重要局部符号包括 check、config。

### Lines 58-66 / 第 58-66 行

```python
        """returns an array for the current benchmark configs"""
        raise ValueError("this method should be reimplemented by subclass")

    def desc(self):
        """return the description of the current benchmark"""
        config = self.config()
        config_str = "_".join([str(x) for x in config])
        device = self.device
        if "NNC_NUM_THREADS" in os.environ:
```

- **EN:** Important local symbols in this block include desc.
- **CN:** 该代码块中的重要局部符号包括 desc。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 67-74 / 第 67-74 行

```python
            num_threads_str = os.environ["NNC_NUM_THREADS"]
            device += num_threads_str
        return f"{self.engine.mode}: {self.module()}_{self.mode}_{device}_{config_str}"

    @staticmethod
    def module():
        raise ValueError("this method should be reimplemented by subclass")

```

- **EN:** Important local symbols in this block include module.
- **CN:** 该代码块中的重要局部符号包括 module。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 75-83 / 第 75-83 行

```python
    def memory_workload(self):
        raise ValueError("this method should be reimplemented by subclass")

    def compute_workload(self):
        """return the number of scalar operations it takes to finish the tensor op"""
        return None

    @staticmethod
    def input_iterable():
```

- **EN:** Important local symbols in this block include memory_workload, compute_workload, input_iterable.
- **CN:** 该代码块中的重要局部符号包括 memory_workload、compute_workload、input_iterable。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 84-91 / 第 84-91 行

```python
        """A benchmark child class should return true if it utilizes the input iter arg"""
        return False

    def dtype_to_bytes(self):
        return torch.tensor(0, dtype=self.dtype).element_size()

    @staticmethod
    def default_configs():
```

- **EN:** Important local symbols in this block include dtype_to_bytes, default_configs, should.
- **CN:** 该代码块中的重要局部符号包括 dtype_to_bytes、default_configs、should。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 92-102 / 第 92-102 行

```python
        """return a list of default configs for this benchmark"""
        raise ValueError("this method should be reimplemented by subclass")

    def is_supported(self):
        return True

    def rand(self, shape, device=None, dtype=None, requires_grad=False):
        v = self.engine.rand(
            shape, device=device, dtype=dtype, requires_grad=requires_grad
        )
        if requires_grad:
```

- **EN:** Important local symbols in this block include is_supported, rand.
- **CN:** 该代码块中的重要局部符号包括 is_supported、rand。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 103-110 / 第 103-110 行

```python
            self.grad_variables.append(v)
        return v

    def nchw_rand(self, shape, device=None, requires_grad=False):
        v = self.engine.nchw_rand(shape, device=device, requires_grad=requires_grad)
        if requires_grad:
            self.grad_variables.append(v)
        return v
```

- **EN:** Important local symbols in this block include nchw_rand.
- **CN:** 该代码块中的重要局部符号包括 nchw_rand。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 111-118 / 第 111-118 行

```python

    def compute(self):
        if self.bm_jit:
            return self.bm_jit(*self.inputs)
        else:
            return self.forward(*self.inputs)

    def run(self, args):
```

- **EN:** Important local symbols in this block include compute, run.
- **CN:** 该代码块中的重要局部符号包括 compute、run。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 119-132 / 第 119-132 行

```python
        self.print_ir = args.print_ir
        if args.cuda_fuser == "old":
            torch._C._jit_override_can_fuse_on_gpu(True)
            if args.print_kernel:
                os.environ["PYTORCH_FUSION_DEBUG"] = "1"
            return self.run_impl(True)
        elif args.cuda_fuser == "te":
            torch._C._jit_set_texpr_fuser_enabled(True)
            with cuda_pointwise_context(
                args.cuda_pointwise_loop_levels,
                args.cuda_pointwise_block_count,
                args.cuda_pointwise_block_size,
            ):
                return self.run_impl(True)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 133-140 / 第 133-140 行

```python
        elif args.cuda_fuser == "nvf":
            torch._C._jit_set_nvfuser_enabled(True)
            torch._C._jit_set_profiling_executor(True)
            torch._C._jit_set_profiling_mode(True)
            torch._C._jit_override_can_fuse_on_cpu(False)
            torch._C._jit_override_can_fuse_on_gpu(False)
            torch._C._jit_set_bailout_depth(20)
            if args.print_kernel:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 141-148 / 第 141-148 行

```python
                os.environ["PYTORCH_CUDA_FUSER_DEBUG"] = "1"
            return self.run_impl(True)
        else:
            return self.run_impl(False)

    def run_impl(self, use_fuser):
        warmups = 10
        if self.device == "cuda":
```

- **EN:** Important local symbols in this block include run_impl.
- **CN:** 该代码块中的重要局部符号包括 run_impl。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 149-156 / 第 149-156 行

```python
            iters = 1000
        else:
            iters = 10
        engine = tensor_engine.get_engine()

        self.bm_jit = None
        for i in range(warmups + iters):
            if i == warmups:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 157-166 / 第 157-166 行

```python
                if self.device == "cuda":
                    engine.sync_cuda()
                time_start = time.time()

            if i == 0:
                if self.jit_mode == "trace" and use_fuser:
                    self.bm_jit = torch.jit.trace(
                        self.forward, example_inputs=self.inputs, check_trace=False
                    )
                if callable(getattr(self, "reference", None)):
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 167-175 / 第 167-175 行

```python
                    self.check()
                else:
                    print("Warning: no reference result for ", self.module())
            elif i == 1:
                # The fusion graph is visible after the first iter is executed
                if self.jit_mode == "trace" and use_fuser and self.print_ir:
                    print(self.bm_jit.graph_for(*self.inputs))
            z = self.compute()
            if self.mode == "both":
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 176-187 / 第 176-187 行

```python
                if self.result_grad is None:
                    self.result_grad = engine.rand_like(z)
                engine.backward([z], [self.result_grad], self.grad_variables)

        if self.device == "cuda":
            engine.sync_cuda()

        duration = time.time() - time_start
        iter_time = duration / iters
        memory_workload = self.memory_workload()
        compute_workload = self.compute_workload()

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 188-197 / 第 188-197 行

```python
        result_dict = {
            "desc": self.desc(),
            "us": iter_time * 1e6,
            "sol": memory_workload["sol"] * self.dtype_to_bytes() / iter_time / 1e9,
            "algorithmic": memory_workload["algorithmic"]
            * self.dtype_to_bytes()
            / iter_time
            / 1e9,
        }
        if compute_workload:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 198-211 / 第 198-211 行

```python
            result_dict["compute_workload"] = compute_workload / iter_time / 1e9
        self.dump_result(result_dict)

    def dump_result(self, result_dict):
        if self.output_type == "json":
            print(json.dumps(result_dict))
        elif self.output_type == "stdout":
            msg = "{}: {:.2f} us, SOL {:.2f} GB/s, algorithmic {:.2f} GB/s".format(
                result_dict["desc"],
                result_dict["us"],
                result_dict["sol"],
                result_dict["algorithmic"],
            )
            if "compute_workload" in result_dict:
```

- **EN:** Important local symbols in this block include dump_result.
- **CN:** 该代码块中的重要局部符号包括 dump_result。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 212-219 / 第 212-219 行

```python
                msg += f", compute {result_dict['compute_workload']:.2f} Gops/s"
            print(msg)
        else:
            raise Exception("Unknown output_type " + self.output_type)  # noqa: TRY002


@contextlib.contextmanager
def cuda_pointwise_context(loop_levels, block_count, block_size):
```

- **EN:** Important local symbols in this block include cuda_pointwise_context.
- **CN:** 该代码块中的重要局部符号包括 cuda_pointwise_context。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 220-229 / 第 220-229 行

```python
    if loop_levels:
        old_loop_levels = torch._C._jit_get_te_cuda_pointwise_loop_levels()
        torch._C._jit_set_te_cuda_pointwise_loop_levels(loop_levels)
    if block_count:
        old_block_count = torch._C._jit_get_te_cuda_pointwise_block_count()
        torch._C._jit_set_te_cuda_pointwise_block_count(block_count)
    if block_size:
        old_block_size = torch._C._jit_get_te_cuda_pointwise_block_size()
        torch._C._jit_set_te_cuda_pointwise_block_size(block_size)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 230-237 / 第 230-237 行

```python
    try:
        yield
    finally:
        if loop_levels:
            torch._C._jit_set_te_cuda_pointwise_loop_levels(old_loop_levels)
        if block_count:
            torch._C._jit_set_te_cuda_pointwise_block_count(old_block_count)
        if block_size:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 238-245 / 第 238-245 行

```python
            torch._C._jit_set_te_cuda_pointwise_block_size(old_block_size)


# Auxiliary class to facilitate dynamic input shape
class DynamicShape:
    r"""
    An Auxiliary class for dynamic shape benchmarks

```

- **EN:** Important local symbols in this block include DynamicShape, to, for.
- **CN:** 该代码块中的重要局部符号包括 DynamicShape、to、for。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 246-253 / 第 246-253 行

```python
    Pre-computes input with random shapes and also
    modifies the compute method so in each call the
    fuser sees a different input tensor shape
    """

    # Number of random inputs in an instance
    SAMPLE_SIZE = 100

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 254-261 / 第 254-261 行

```python
    def __init__(self, dynamic_range=1.2):
        self._input_samples = []
        self._input_sample_index = 0
        self._dynamic_range = (
            1.0 / dynamic_range if dynamic_range > 1.0 else dynamic_range
        )
        self._enable_dynamic_shapes = True

```

- **EN:** Important local symbols in this block include __init__.
- **CN:** 该代码块中的重要局部符号包括 __init__。

### Lines 262-269 / 第 262-269 行

```python
    # Returns the input test case that current index points to
    @property
    def inputs(self):
        return self._input_samples[self._input_sample_index]

    # An inputs assignment actually adds a test case in the class buffer
    @inputs.setter
    def inputs(self, val):
```

- **EN:** Important local symbols in this block include inputs, buffer.
- **CN:** 该代码块中的重要局部符号包括 inputs、buffer。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 270-280 / 第 270-280 行

```python
        self._input_samples.append(val)

    # Runs normal compute while increment test case index
    def compute(self):
        super().compute()
        self._input_sample_index = (self._input_sample_index + 1) % self.SAMPLE_SIZE

    # Defined by benchmark, the benchmark needs to specify the input
    # tensor construction in this method, essentially the same way
    # a benchmark creates the inputs list in the initializer
    def instantiate_input(self):
```

- **EN:** Important local symbols in this block include compute, instantiate_input.
- **CN:** 该代码块中的重要局部符号包括 compute、instantiate_input。

### Lines 281-290 / 第 281-290 行

```python
        raise NotImplementedError

    # Instantiate random shaped inputs and start the benchmark run
    def run(self, args):
        # force disable dynamic shape from command line
        if args.no_dynamic_shape:
            self._enable_dynamic_shapes = False
        self.load_inputs()
        super().run(args)

```

- **EN:** Important local symbols in this block include run.
- **CN:** 该代码块中的重要局部符号包括 run。

### Lines 291-298 / 第 291-298 行

```python
    # pre-compute inputs so the creations of random tensors
    # do not add to the compute time
    def load_inputs(self):
        for i in range(self.SAMPLE_SIZE - 1):
            self.instantiate_input()

    # returns a randomized shape
    def rand_shape(self, shape):
```

- **EN:** Important local symbols in this block include load_inputs, rand_shape.
- **CN:** 该代码块中的重要局部符号包括 load_inputs、rand_shape。

### Lines 299-307 / 第 299-307 行

```python
        if not self._enable_dynamic_shapes:
            return shape
        ratios = np.random.uniform(self._dynamic_range, 1.0, len(shape))
        dyn_shape = list(np.multiply(shape, ratios).astype(int))
        return dyn_shape


benchmark_classes = []

```

- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 308-310 / 第 308-310 行

```python

def register_benchmark_class(benchmark_cls):
    benchmark_classes.append(benchmark_cls)
```

- **EN:** Important local symbols in this block include register_benchmark_class.
- **CN:** 该代码块中的重要局部符号包括 register_benchmark_class。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: Benchmark, DynamicShape, __init__, forward, check, config, desc, module** — 代表性符号：Benchmark、DynamicShape、__init__、forward、check、config、desc、module

## Dependencies / 依赖关系

- `contextlib`
- `json`
- `os`
- `time`
- `numpy`
- `torch`
- `.`

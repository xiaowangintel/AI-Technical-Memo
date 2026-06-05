# __main__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/tensorexpr/__main__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```python
import argparse
import itertools
import os

# from . import conv
# from . import normalization
# from . import pooling
from . import (  # noqa: F401
    attention,
    benchmark,
    broadcast,
    concat,
    elementwise,
    matmul,
    reduction,
    rnn_eltwise,
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 17-32 / 第 17-32 行

```python
    softmax,
    swish,
    tensor_engine,
)


def main():
    parser = argparse.ArgumentParser(
        formatter_class=argparse.RawDescriptionHelpFormatter,
        description="""Benchmark operators in specific shapes.
Works only with Python3.\n A few examples:
  * benchmark.py: runs all the default configs with all the benchmarks.
  * benchmark.py reduce: runs all the default configs with all benchmark with a prefix 'reduce'
  * benchmark.py layernorm_fwd_cpu_128_32_128_128: run a particular benchmark in that config""",
    )
    parser.add_argument(
```

- **EN:** Important local symbols in this block include main.
- **CN:** 该代码块中的重要局部符号包括 main。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 33-48 / 第 33-48 行

```python
        "benchmark_names",
        type=str,
        default=None,
        nargs="*",
        help="name of the benchmark to run",
    )
    parser.add_argument(
        "--device",
        type=str,
        default="cpu,cuda",
        help="a comma separated list of device names",
    )
    parser.add_argument(
        "--mode",
        type=str,
        default="fwd,both",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 49-64 / 第 49-64 行

```python
        help="a comma separated list of running modes",
    )
    parser.add_argument(
        "--dtype",
        type=str,
        default="float32",
        help="a comma separated list of Data Types: {float32[default], float16}",
    )
    parser.add_argument(
        "--input-iter",
        type=str,
        default=None,
        help="a comma separated list of Tensor dimensions that includes a start, \
              stop, and increment that can be constant or a power of 2 \
              {start:stop:inc,start:stop:pow2}",
    )
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 65-80 / 第 65-80 行

```python
    parser.add_argument(
        "--engine",
        type=str,
        default="pt",
        help="the underlying tensor engine. only pt for now",
    )
    parser.add_argument(
        "--jit-mode",
        "--jit_mode",
        type=str,
        default="trace",
        help="the jit mode to use: one of {trace, none}",
    )
    parser.add_argument(
        "--cuda-pointwise-loop-levels",
        "--cuda_pointwise_loop_levels",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 81-96 / 第 81-96 行

```python
        type=int,
        default=None,
        help="num of loop levesl for Cuda pointwise operations: 2 or 3",
    )
    parser.add_argument(
        "--cuda-pointwise-block-count",
        "--cuda_pointwise_block_count",
        type=int,
        default=None,
        help="num of block for Cuda pointwise operations",
    )
    parser.add_argument(
        "--cuda-pointwise-block-size",
        "--cuda_pointwise_block_size",
        type=int,
        default=None,
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 97-112 / 第 97-112 行

```python
        help="num of blocks for Cuda pointwise operations",
    )
    parser.add_argument(
        "--cuda-fuser",
        "--cuda_fuser",
        type=str,
        default="te",
        help="The Cuda fuser backend to use: one of {te, nvf, old, none}",
    )
    parser.add_argument(
        "--output",
        type=str,
        default="stdout",
        help="The output format of the benchmark run {stdout[default], json}",
    )
    parser.add_argument(
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 113-128 / 第 113-128 行

```python
        "--print-ir",
        action="store_true",
        help="Print the IR graph of the Fusion.",
    )
    parser.add_argument(
        "--print-kernel",
        action="store_true",
        help="Print generated kernel(s).",
    )
    parser.add_argument(
        "--no-dynamic-shape",
        action="store_true",
        help="Disable shape randomization in dynamic benchmarks.",
    )
    parser.add_argument(
        "--cpu-fusion",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 129-141 / 第 129-141 行

```python
        "--cpu_fusion",
        default=False,
        action="store_true",
        help="Enable CPU fusion.",
    )
    parser.add_argument(
        "--cat-wo-conditionals",
        "--cat_wo_conditionals",
        default=False,
        action="store_true",
        help="Enable CAT wo conditionals.",
    )

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 142-153 / 第 142-153 行

```python
    args = parser.parse_args()

    if args.cuda_fuser == "te":
        import torch

        torch._C._jit_set_profiling_executor(True)
        torch._C._jit_set_texpr_fuser_enabled(True)
        torch._C._jit_override_can_fuse_on_gpu(True)
        torch._C._get_graph_executor_optimize(True)
    elif args.cuda_fuser == "old":
        import torch

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 154-166 / 第 154-166 行

```python
        torch._C._jit_set_profiling_executor(False)
        torch._C._jit_set_texpr_fuser_enabled(False)
        torch._C._jit_override_can_fuse_on_gpu(True)
    elif args.cuda_fuser == "nvf":
        import torch

        torch._C._jit_set_profiling_executor(True)
        torch._C._jit_set_texpr_fuser_enabled(False)
        torch._C._jit_set_nvfuser_enabled(True)
        torch._C._get_graph_executor_optimize(True)
    else:
        raise ValueError(f"Undefined fuser: {args.cuda_fuser}")

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 167-175 / 第 167-175 行

```python
    if args.cpu_fusion:
        import torch

        torch._C._jit_override_can_fuse_on_cpu(True)
    else:
        import torch

        torch._C._jit_override_can_fuse_on_cpu(False)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 176-184 / 第 176-184 行

```python
    if args.cat_wo_conditionals:
        import torch

        torch._C._jit_cat_wo_conditionals(True)
    else:
        import torch

        torch._C._jit_cat_wo_conditionals(False)

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 185-195 / 第 185-195 行

```python
    def set_global_threads(num_threads):
        os.environ["OMP_NUM_THREADS"] = str(num_threads)
        os.environ["MKL_NUM_THREADS"] = str(num_threads)
        os.environ["TVM_NUM_THREADS"] = str(num_threads)
        os.environ["NNC_NUM_THREADS"] = str(num_threads)

    devices = args.device.split(",")
    # accept 'gpu' as an alternative as the 'cuda' device
    devices = ["cuda" if device == "gpu" else device for device in devices]
    cpu_count = 0
    for index, device in enumerate(devices):
```

- **EN:** Important local symbols in this block include set_global_threads.
- **CN:** 该代码块中的重要局部符号包括 set_global_threads。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 196-211 / 第 196-211 行

```python
        if device.startswith("cpu"):
            cpu_count += 1
            if cpu_count > 1:
                raise ValueError(
                    f"more than one CPU device is not allowed: {cpu_count:d}"
                )
            if device == "cpu":
                continue
            num_threads_str = device[3:]
            try:
                # see if the device is in 'cpu1' or 'cpu4' format
                num_threads = int(num_threads_str)
                set_global_threads(num_threads)
                devices[index] = "cpu"
            except ValueError:
                continue
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 212-220 / 第 212-220 行

```python

    modes = args.mode.split(",")

    datatypes = args.dtype.split(",")
    for index, dtype in enumerate(datatypes):
        datatypes[index] = getattr(torch, dtype)
        if not datatypes[index]:
            raise AttributeError(f"DataType: {dtype} is not valid!")

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 221-230 / 第 221-230 行

```python
    tensor_engine.set_engine_mode(args.engine)

    def run_default_configs(bench_cls, allow_skip=True):
        for mode, device, dtype, config in itertools.product(
            modes, devices, datatypes, bench_cls.default_configs()
        ):
            bench = bench_cls(mode, device, dtype, *config)
            bench.output_type = args.output
            bench.jit_mode = args.jit_mode
            if not bench.is_supported():
```

- **EN:** Important local symbols in this block include run_default_configs.
- **CN:** 该代码块中的重要局部符号包括 run_default_configs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 231-238 / 第 231-238 行

```python
                if allow_skip:
                    continue
                else:
                    raise ValueError(
                        f"attempted to run an unsupported benchmark: {bench.desc()}"
                    )
            bench.run(args)

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 239-246 / 第 239-246 行

```python
    def run_with_input_iter(bench_cls, input_iter, allow_skip=True):
        tensor_dim_specs = input_iter.split(",")
        tensor_dim_specs = [dim.split(":") for dim in tensor_dim_specs]

        configs = []
        for start, stop, inc in tensor_dim_specs:
            dim_list = []
            if inc == "pow2":
```

- **EN:** Important local symbols in this block include run_with_input_iter.
- **CN:** 该代码块中的重要局部符号包括 run_with_input_iter。

### Lines 247-262 / 第 247-262 行

```python
                curr = int(start)
                while curr <= int(stop):
                    dim_list.append(curr)
                    curr <<= 1
            elif inc == "pow2+1":
                curr = int(start)
                while curr <= int(stop):
                    dim_list.append(curr)
                    curr -= 1
                    curr <<= 1
                    curr += 1
            else:
                dim_list = list(range(int(start), int(stop) + int(inc), int(inc)))
            configs.append(dim_list)
        configs = itertools.product(*configs)

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 263-270 / 第 263-270 行

```python
        for mode, device, dtype, config in itertools.product(
            modes, devices, datatypes, list(configs)
        ):
            bench = bench_cls(mode, device, dtype, *config)
            bench.output_type = args.output
            bench.jit_mode = args.jit_mode
            if not bench.is_supported():
                if allow_skip:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 271-279 / 第 271-279 行

```python
                    continue
                else:
                    raise ValueError(
                        f"attempted to run an unsupported benchmark: {bench.desc()}"
                    )
            bench.run(args)

    benchmark_classes = benchmark.benchmark_classes
    if not args.benchmark_names:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 280-287 / 第 280-287 行

```python
        # by default, run all the benchmarks
        for benchmark_cls in benchmark_classes:
            run_default_configs(benchmark_cls, allow_skip=True)
    else:
        for name in args.benchmark_names:
            # if the name is the prefix of a benchmark class, run all the benchmarks for that class
            match_class_name = False
            for bench_cls in benchmark_classes:
```

- **EN:** Important local symbols in this block include match_class_name.
- **CN:** 该代码块中的重要局部符号包括 match_class_name。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 288-298 / 第 288-298 行

```python
                if name in bench_cls.module():
                    match_class_name = True
                    if (args.input_iter is not None) and bench_cls.input_iterable():
                        run_with_input_iter(bench_cls, args.input_iter, allow_skip=True)
                    else:
                        if args.input_iter is not None:
                            print(
                                f"WARNING: Incompatible benchmark class called with input_iter arg: {name}"
                            )
                        run_default_configs(bench_cls, allow_skip=True)

```

- **EN:** Important local symbols in this block include called.
- **CN:** 该代码块中的重要局部符号包括 called。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 299-306 / 第 299-306 行

```python
            if match_class_name:
                continue

            # if not a class module, parse the config and call it that way
            match_class_name = False
            for bench_cls in benchmark_classes:
                cls_module = bench_cls.module()
                if name.startswith(cls_module):
```

- **EN:** Important local symbols in this block include module.
- **CN:** 该代码块中的重要局部符号包括 module。

### Lines 307-316 / 第 307-316 行

```python
                    match_class_name = True
                    if name[len(cls_module)] != "_":
                        raise ValueError(f"invalid name: {name}")
                    config_str = name[(len(cls_module) + 1) :]
                    config = config_str.split("_")
                    if len(config) < 2:
                        raise ValueError(f"invalid config: {config}")
                    mode, device = config[0:2]
                    # TODO: make sure virtual devices such as 'cpu1' and 'cpu4' are supported.
                    if mode not in ["fwd", "both"]:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 317-329 / 第 317-329 行

```python
                        raise ValueError(f"invalid mode: {mode}")
                    for i, entry in enumerate(config):
                        try:
                            value = int(entry)
                            config[i] = value
                        except ValueError:
                            pass
                    # TODO: output dtype in the config and  parse it back from the str
                    bench = bench_cls(config[0], config[1], torch.float32, *config[2:])
                    bench.jit_mode = args.jit_mode
                    bench.output_type = args.output
                    bench.run(args)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 330-337 / 第 330-337 行

```python
            if not match_class_name:
                available_classes = ", ".join(
                    [bench_cls.module() for bench_cls in benchmark_classes]
                )
                raise ValueError(
                    f"invalid name: {name}\nAvailable benchmark classes:\n{available_classes}"
                )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 338-340 / 第 338-340 行

```python

if __name__ == "__main__":
    main()
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Benchmark workflow** — 记录或实现性能测量场景。
- **Structured metadata** — 存放机器可读的设置或清单式元数据。
- **Representative symbols: main, set_global_threads, run_default_configs, run_with_input_iter** — 代表性符号：main、set_global_threads、run_default_configs、run_with_input_iter

## Dependencies / 依赖关系

- `argparse`
- `itertools`
- `os`
- `.`
- `torch`

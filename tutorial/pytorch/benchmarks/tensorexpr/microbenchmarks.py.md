# microbenchmarks.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/tensorexpr/microbenchmarks.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9 / 第 1-9 行

```python
import argparse
import operator
import time

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 10-17 / 第 10-17 行

```python
import torch
import torch._C._te as te


class kernel_arena_scope:
    def __enter__(self):
        self.scope = te.KernelScope()

```

- **EN:** Important local symbols in this block include kernel_arena_scope, __enter__.
- **CN:** 该代码块中的重要局部符号包括 kernel_arena_scope、__enter__。

### Lines 18-33 / 第 18-33 行

```python
    def __exit__(self, typ, val, traceback):
        self.scope = None


unary_ops = [
    ("sin", torch.sin),
    ("cos", torch.cos),
    ("tan", torch.tan),
    ("asin", torch.asin),
    ("acos", torch.acos),
    ("atan", torch.atan),
    ("sinh", torch.sinh),
    ("cosh", torch.cosh),
    ("tanh", torch.tanh),
    ("sigmoid", torch.sigmoid),
    ("exp", torch.exp),
```

- **EN:** Important local symbols in this block include __exit__.
- **CN:** 该代码块中的重要局部符号包括 __exit__。

### Lines 34-49 / 第 34-49 行

```python
    ("expm1", torch.expm1),
    ("expm1", torch.expm1),
    ("abs", torch.abs),
    ("log", torch.log),
    ("fast_log", torch.log),
    ("log2", torch.log2),
    ("log10", torch.log10),
    ("log1p", torch.log1p),
    ("erf", torch.erf),
    ("erfc", torch.erfc),
    ("sqrt", torch.sqrt),
    ("rsqrt", torch.rsqrt),
    ("ceil", torch.ceil),
    ("floor", torch.floor),
    ("round", torch.round),
    ("trunc", torch.trunc),
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 50-57 / 第 50-57 行

```python
    ("lgamma", torch.lgamma),
    # ("frac", torch.frac), # seems unimplemented
    # ("isnan", torch.isnan), # no out variant
]


def gen_unary_nnc_fun(nnc_name):
    def nnc_fun(A, B):
```

- **EN:** Important local symbols in this block include gen_unary_nnc_fun, nnc_fun.
- **CN:** 该代码块中的重要局部符号包括 gen_unary_nnc_fun、nnc_fun。

### Lines 58-65 / 第 58-65 行

```python
        def compute(i, j):
            return getattr(A.load([i, j]), nnc_name)()

        return compute

    return nnc_fun


```

- **EN:** Important local symbols in this block include compute.
- **CN:** 该代码块中的重要局部符号包括 compute。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 66-73 / 第 66-73 行

```python
def gen_unary_torch_fun(torch_op):
    def torch_fun(a, b, out):
        def fun():
            return torch_op(a, out=out)

        return fun

    return torch_fun
```

- **EN:** Important local symbols in this block include gen_unary_torch_fun, torch_fun, fun.
- **CN:** 该代码块中的重要局部符号包括 gen_unary_torch_fun、torch_fun、fun。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 74-81 / 第 74-81 行

```python


def gen_binary_nnc_fun(fn):
    def nnc_fun(A, B):
        def compute(i, j):
            return fn(A.load([i, j]), B.load([i, j]))

        return compute
```

- **EN:** Important local symbols in this block include gen_binary_nnc_fun, nnc_fun, compute.
- **CN:** 该代码块中的重要局部符号包括 gen_binary_nnc_fun、nnc_fun、compute。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 82-89 / 第 82-89 行

```python

    return nnc_fun


def gen_binary_torch_fun(fn):
    def pt_fun(a, b, out):
        def fun():
            return fn(a, b, out=out)
```

- **EN:** Important local symbols in this block include gen_binary_torch_fun, pt_fun, fun.
- **CN:** 该代码块中的重要局部符号包括 gen_binary_torch_fun、pt_fun、fun。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 90-97 / 第 90-97 行

```python

        return fun

    return pt_fun


def gen_int_comparison_tensors(N, M):
    return (
```

- **EN:** Important local symbols in this block include gen_int_comparison_tensors.
- **CN:** 该代码块中的重要局部符号包括 gen_int_comparison_tensors。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 98-105 / 第 98-105 行

```python
        torch.randint(0, 3, (N, M)),
        torch.randint(0, 3, (N, M)),
        torch.empty((N, M), dtype=torch.bool),
    )


def gen_float_comparison_tensors(N, M):
    return (torch.rand(N, M), torch.rand(N, M), torch.empty((N, M), dtype=torch.bool))
```

- **EN:** Important local symbols in this block include gen_float_comparison_tensors.
- **CN:** 该代码块中的重要局部符号包括 gen_float_comparison_tensors。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 106-121 / 第 106-121 行

```python


te_bool = te.Dtype.Bool
binary_ops = [
    ("add", operator.add, torch.add),
    ("mul", operator.mul, torch.mul),
    ("sub", operator.sub, torch.sub),
    ("div", operator.truediv, torch.div),
    (
        "eq",
        (lambda a, b: te.Cast.make(te_bool, a == b)),
        torch.eq,
        gen_int_comparison_tensors,
    ),
    (
        "gt",
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 122-137 / 第 122-137 行

```python
        (lambda a, b: te.Cast.make(te_bool, a > b)),
        torch.gt,
        gen_float_comparison_tensors,
    ),
    (
        "lt",
        (lambda a, b: te.Cast.make(te_bool, a < b)),
        torch.lt,
        gen_float_comparison_tensors,
    ),
    (
        "gte",
        (lambda a, b: te.Cast.make(te_bool, a >= b)),
        torch.greater_equal,
        gen_float_comparison_tensors,
    ),
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 138-147 / 第 138-147 行

```python
    (
        "lte",
        (lambda a, b: te.Cast.make(te_bool, a <= b)),
        torch.less_equal,
        gen_float_comparison_tensors,
    ),
    # ('neq', (lambda a, b: a != b), None)), # no one-op equivalent
    # ('&', (lambda a, b: a & b), torch.bitwise_and), # requires more work to test
]

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 148-156 / 第 148-156 行

```python

def nnc_relu(A, B):
    def f(i, j):
        return torch._C._te.ifThenElse(
            A.load([i, j]) < torch._C._te.ExprHandle.float(0),
            torch._C._te.ExprHandle.float(0),
            A.load([i, j]),
        )

```

- **EN:** Important local symbols in this block include nnc_relu, f.
- **CN:** 该代码块中的重要局部符号包括 nnc_relu、f。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 157-169 / 第 157-169 行

```python
    return f


def pt_relu(a, b, c):
    return torch.relu(a)


custom_ops = [
    ("relu", nnc_relu, pt_relu),
    # ('nnc_mul_relu', nnc_mul_relu, pt_mul_relu)
    # ('manual_sigmoid', nnc_manual_sigmoid, lambda a, b, c: torch.sigmoid(a, out=c))
]

```

- **EN:** Important local symbols in this block include pt_relu.
- **CN:** 该代码块中的重要局部符号包括 pt_relu。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 170-177 / 第 170-177 行

```python

def gen_custom_torch_fun(fn):
    def pt_fun(a, b, out):
        def fun():
            return fn(a, b, out)

        return fun

```

- **EN:** Important local symbols in this block include gen_custom_torch_fun, pt_fun, fun.
- **CN:** 该代码块中的重要局部符号包括 gen_custom_torch_fun、pt_fun、fun。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 178-189 / 第 178-189 行

```python
    return pt_fun


def normalize_benchmarks(ops):
    return [i + (None,) if len(i) == 3 else i for i in ops]


names = []
nnc_fns = []
pt_fns = []
shape_fns = []

```

- **EN:** Important local symbols in this block include normalize_benchmarks.
- **CN:** 该代码块中的重要局部符号包括 normalize_benchmarks。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 190-201 / 第 190-201 行

```python
for nnc_name, pt_op in unary_ops:
    names.append(nnc_name)
    nnc_fns.append(gen_unary_nnc_fun(nnc_name))
    pt_fns.append(gen_unary_torch_fun(pt_op))
    shape_fns.append(None)

for name, lmbda, pt_fn, shape_fn in normalize_benchmarks(binary_ops):
    names.append(name)
    nnc_fns.append(gen_binary_nnc_fun(lmbda))
    pt_fns.append(gen_binary_torch_fun(pt_fn))
    shape_fns.append(shape_fn)

```

- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 202-209 / 第 202-209 行

```python
for name, lmbda, pt_fn, shape_fn in normalize_benchmarks(custom_ops):
    names.append(name)
    nnc_fns.append(lmbda)
    pt_fns.append(gen_custom_torch_fun(pt_fn))
    shape_fns.append(shape_fn)

benchmarks = list(zip(names, nnc_fns, pt_fns, shape_fns))

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 210-218 / 第 210-218 行

```python

def run_benchmarks(benchmarks, sizes):
    df = pd.DataFrame(columns=["name", "N", "M", "nnc_time", "torch_time", "ratio"])
    with torch.no_grad():
        for name, nnc_fun, torch_fun, shape_fn in benchmarks:
            for N, M in sizes:
                iters = int(1e6 / (N + M))
                with kernel_arena_scope():
                    if shape_fn is None:
```

- **EN:** Important local symbols in this block include run_benchmarks.
- **CN:** 该代码块中的重要局部符号包括 run_benchmarks。

### Lines 219-226 / 第 219-226 行

```python
                        tA = torch.rand(M, N).clamp(0.01, 0.99)
                        tB = torch.rand(M, N).clamp(0.01, 0.99)
                        tX = torch.empty(M, N)
                        tR = torch.empty(M, N)
                    else:
                        tA, tB, tX = shape_fn(M, N)
                        tR = tX.clone()

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 227-234 / 第 227-234 行

```python
                    def get_nnc_type(dtype):
                        if dtype == torch.float:
                            return torch._C._te.Dtype.Float
                        elif dtype == torch.long:
                            return torch._C._te.Dtype.Long

                    dtype = get_nnc_type(tA.dtype)

```

- **EN:** Important local symbols in this block include get_nnc_type.
- **CN:** 该代码块中的重要局部符号包括 get_nnc_type。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 235-244 / 第 235-244 行

```python
                    dM = torch._C._te.ExprHandle.int(M)
                    dN = torch._C._te.ExprHandle.int(N)

                    A = torch._C._te.Placeholder("A", dtype, [dM, dN])
                    B = torch._C._te.Placeholder("B", dtype, [dM, dN])

                    dim_args = [
                        torch._C._te.DimArg(*args) for args in [(dM, "m"), (dN, "n")]
                    ]

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 245-253 / 第 245-253 行

```python
                    compute = nnc_fun(A, B)
                    X = torch._C._te.Compute("X", dim_args, compute)
                    loopnest = torch._C._te.LoopNest([X])
                    loopnest.prepare_for_codegen()
                    stmt = torch._C._te.simplify(loopnest.root_stmt())
                    cg = torch._C._te.construct_codegen(
                        "llvm", stmt, [torch._C._te.BufferArg(x) for x in [A, B, X]]
                    )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 254-261 / 第 254-261 行

```python
                    # warmup
                    for _ in range(10):
                        cg.call([tA, tB, tX])
                    start = time.time()
                    for it in range(iters):
                        cg.call([tA, tB, tX])
                    time1 = time.time() - start

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 262-270 / 第 262-270 行

```python
                    fn = torch_fun(tA, tB, tR)
                    # warmup
                    for _ in range(10):
                        tR = fn()
                    start = time.time()
                    for it in range(iters):
                        tR = fn()
                    time2 = time.time() - start

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 271-283 / 第 271-283 行

```python
                    df = df.append(
                        {
                            "name": name,
                            "N": N,
                            "M": M,
                            "nnc_time": time1,
                            "torch_time": time2,
                            "ratio": time2 / time1,
                        },
                        ignore_index=True,
                    )
                    print(name, N, M)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 284-291 / 第 284-291 行

```python
                    print(time2 / time1, time1, time2)
                    print()

                    def check_correctness(a, b):
                        if not np.allclose(a, b):
                            print(name)
                            raise AssertionError(f"Arrays not close for {name}")

```

- **EN:** Important local symbols in this block include check_correctness.
- **CN:** 该代码块中的重要局部符号包括 check_correctness。

### Lines 292-300 / 第 292-300 行

```python
                    check_correctness(tX, tR)
    return df


def dump_plot(df, sizes):
    keys = []
    vals = []
    indexed = df[df["N"] == df["M"]]
    for index, row in indexed.iterrows():
```

- **EN:** Important local symbols in this block include dump_plot.
- **CN:** 该代码块中的重要局部符号包括 dump_plot。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 301-316 / 第 301-316 行

```python
        keys.append(row["name"])
        vals.append(row["ratio"])

    keys = keys[:: len(sizes)]
    sns.set(rc={"figure.figsize": (5.0, len(keys) * 0.5)})

    cmap = sns.diverging_palette(10, 120, n=9, as_cmap=True)
    np_vals = np.array([vals]).reshape(-1, len(sizes))
    g = sns.heatmap(np_vals, annot=True, cmap=cmap, center=1.0, yticklabels=True)
    plt.yticks(rotation=0)
    plt.title("PyTorch performance divided by NNC performance (single core)")
    plt.xlabel("Size of NxN matrix")
    plt.ylabel("Operation")
    g.set_yticklabels(keys)
    g.set_xticklabels(sizes)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 317-329 / 第 317-329 行

```python
    plt.savefig("nnc.png")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Runs NNC microbenchmarks")
    parser.add_argument(
        "--multi-threaded",
        "--multi_threaded",
        action="store_true",
        help="Run with more than one thread",
    )
    args = parser.parse_args()
    if not args.multi_threaded:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 330-334 / 第 330-334 行

```python
        torch.set_num_threads(1)

    sizes = [1, 4, 16, 64, 256, 1024]
    df = run_benchmarks(benchmarks, [(i, i) for i in sizes])
    dump_plot(df, sizes)
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Representative symbols: kernel_arena_scope, __enter__, __exit__, gen_unary_nnc_fun, nnc_fun, compute, gen_unary_torch_fun, torch_fun** — 代表性符号：kernel_arena_scope、__enter__、__exit__、gen_unary_nnc_fun、nnc_fun、compute、gen_unary_torch_fun、torch_fun

## Dependencies / 依赖关系

- `argparse`
- `operator`
- `time`
- `matplotlib.pyplot`
- `numpy`
- `pandas`
- `seaborn`
- `torch`
- `torch._C._te`

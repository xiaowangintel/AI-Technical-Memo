# utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/sparse/dlmc/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8 / 第 1-8 行

```python
import math
from pathlib import Path

from scipy import sparse

import torch


```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。

### Lines 9-17 / 第 9-17 行

```python
def to_coo_scipy(x):
    indices_1 = x._indices().numpy()
    values_1 = x._values().numpy()
    return sparse.coo_matrix((values_1, (indices_1[0], indices_1[1])), shape=x.shape)


def sparse_grad_output(a, b):
    c = torch.sparse.mm(a, b)
    if c.is_sparse:
```

- **EN:** Important local symbols in this block include to_coo_scipy, sparse_grad_output.
- **CN:** 该代码块中的重要局部符号包括 to_coo_scipy、sparse_grad_output。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 18-28 / 第 18-28 行

```python
        c2 = torch.rand_like(c.to_dense())
        return c2.sparse_mask(c.coalesce())
    else:
        return torch.rand_like(c)


def read_matrix_params(path):
    with open(path) as file:
        line = file.readline()
        nrows, ncols, nnz = (int(el) for el in line.split(", "))
        return (nrows, ncols), nnz
```

- **EN:** Important local symbols in this block include read_matrix_params.
- **CN:** 该代码块中的重要局部符号包括 read_matrix_params。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 29-36 / 第 29-36 行

```python


def csr_to_coo(indices, indptr, shape):
    n_rows, n_cols = shape
    cols = indices
    rows = [0] * len(cols)
    for i in range(n_rows):
        for j in range(indptr[i], indptr[i + 1]):
```

- **EN:** Important local symbols in this block include csr_to_coo.
- **CN:** 该代码块中的重要局部符号包括 csr_to_coo。

### Lines 37-46 / 第 37-46 行

```python
            rows[j] = i
    return torch.tensor([rows, cols], dtype=torch.long)


def load_sparse_matrix(path, device):
    with open(path) as file:
        nrows, ncols, nnz = (int(el) for el in file.readline().split(", "))
        index_pointers = (int(el) for el in file.readline().split())
        indices = (int(el) for el in file.readline().split())

```

- **EN:** Important local symbols in this block include load_sparse_matrix.
- **CN:** 该代码块中的重要局部符号包括 load_sparse_matrix。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 47-54 / 第 47-54 行

```python
    index_pointers = list(index_pointers)
    indices = list(indices)
    data = torch.randn(nnz, dtype=torch.double)
    shape = (nrows, ncols)
    return torch.sparse_coo_tensor(
        csr_to_coo(indices, index_pointers, shape), data, shape, device=device
    )

```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 55-62 / 第 55-62 行

```python

def gen_vector(path, device):
    with open(path) as file:
        nrows, ncols, nnz = (int(el) for el in file.readline().split(", "))
        return torch.randn(nrows, dtype=torch.double, device=device)


def gen_matrix(path, device):
```

- **EN:** Important local symbols in this block include gen_vector, gen_matrix.
- **CN:** 该代码块中的重要局部符号包括 gen_vector、gen_matrix。
- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 63-78 / 第 63-78 行

```python
    with open(path) as file:
        nrows, ncols, nnz = (int(el) for el in file.readline().split(", "))
        return torch.randn(nrows, ncols, dtype=torch.double, device=device)


def load_spmv_dataset(dataset_path, hidden_size, sparsity, device, n_limit=math.inf):
    """load_spmv_dataset loads a DLMC dataset for a sparse matrix-vector multiplication (SPMV) performance test.
    Args:
        dataset_path:
            path of the dataset from DLMC collection.
        hidden_size
            This value allows tensors of varying sizes.
        sparsity:
            This value allows tensors of varying sparsities.
        device:
            Whether to place the Tensor on a GPU or CPU.
```

- **EN:** Important local symbols in this block include load_spmv_dataset.
- **CN:** 该代码块中的重要局部符号包括 load_spmv_dataset。
- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 79-88 / 第 79-88 行

```python
        n_limit:
            This value allows a dataset with some limit size.
    """
    current_folder_path = f"{dataset_path}/{sparsity}"
    path = Path(current_folder_path)
    files = path.glob("**/*.smtx")
    print(dataset_path, hidden_size, sparsity)
    index = 0
    x_files, y_files = [], []
    for f in files:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 89-99 / 第 89-99 行

```python
        if index >= n_limit:
            break
        print(".", end="")
        size, nnz = read_matrix_params(f.as_posix())
        if size[1] == hidden_size:
            x_files.append(f.as_posix())
        if size[0] == hidden_size:
            y_files.append(f.as_posix())
        index += 1  # noqa: SIM113
    print()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 100-115 / 第 100-115 行

```python
    for fx, fy in zip(x_files, y_files):
        x = load_sparse_matrix(fx, device)
        y = gen_vector(fy, device)
        yield (x, y)


def load_spmm_dataset(
    dataset_path, hidden_size, sparsity, spmm_type, device, n_limit=math.inf
):
    """load_spmm_dataset loads a DLMC dataset for a sparse matrix-matrix multiplication (SPMM) performance test.
    Args:
        dataset_path:
            path of the dataset from DLMC collection.
        hidden_size
            This value allows tensors of varying sizes.
        sparsity:
```

- **EN:** Important local symbols in this block include load_spmm_dataset.
- **CN:** 该代码块中的重要局部符号包括 load_spmm_dataset。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 116-130 / 第 116-130 行

```python
            This value allows tensors of varying sparsities.
        spmm_type:
            This value allows tensors for `sparse@sparse` or `sparse@dense` operations.
        device:
            Whether to place the Tensor on a GPU or CPU.
        n_limit:
            This value allows a dataset with some limit size.
    """
    current_folder_path = f"{dataset_path}/{sparsity}"
    path = Path(current_folder_path)
    files = path.glob("**/*.smtx")
    print(dataset_path, hidden_size, sparsity)
    index = 0
    x_files, y_files = [], []
    for f in files:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 131-141 / 第 131-141 行

```python
        if index >= n_limit:
            break
        print(".", end="")
        size, nnz = read_matrix_params(f.as_posix())
        if size[1] == hidden_size:
            x_files.append(f.as_posix())
        if size[0] == hidden_size:
            y_files.append(f.as_posix())
        index += 1  # noqa: SIM113
    print()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 142-150 / 第 142-150 行

```python
    for fx, fy in zip(x_files, y_files):
        x = load_sparse_matrix(fx, device)
        y = (
            gen_matrix(fy, device)
            if spmm_type == "sparse@dense"
            else load_sparse_matrix(fy, device)
        )
        yield (x, y)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 151-166 / 第 151-166 行

```python

def load_dlmc_dataset(
    dataset_path,
    operation,
    hidden_size,
    sparsity,
    device,
    requires_grad,
    n_limit=math.inf,
):
    """load_dlmc_dataset loads a DLMC dataset for a matmul performance test.
    Args:
        dataset_path:
            path of the dataset from DLMC collection.
        operation:
            This value allows tensors for `sparse@sparse`|`sparse@dense`|`sparse@vector` operations.
```

- **EN:** Important local symbols in this block include load_dlmc_dataset.
- **CN:** 该代码块中的重要局部符号包括 load_dlmc_dataset。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 167-178 / 第 167-178 行

```python
        hidden_size
            This value allows tensors of varying sizes.
        sparsity:
            This value allows tensors of varying sparsities.
        device:
            Whether to place the Tensor on a GPU or CPU.
        requires_grad:
            Loads the dataset for backward test.
        n_limit:
            This value allows a dataset with some limit size.
    """
    if operation == "sparse@sparse" or operation == "sparse@dense":
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 179-188 / 第 179-188 行

```python
        collection = load_spmm_dataset(
            dataset_path, hidden_size, sparsity, operation, device, n_limit
        )
    elif operation == "sparse@vector":
        collection = load_spmv_dataset(
            dataset_path, hidden_size, sparsity, device, n_limit
        )
    scipy_vars = {}
    backward_vars = {}
    for x, y in collection:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 189-204 / 第 189-204 行

```python
        if device == "cpu":
            scipy_vars = {
                "sx": to_coo_scipy(x) if x.is_sparse else x.numpy(),
                "sy": to_coo_scipy(y) if y.is_sparse else y.numpy(),
            }
        if not requires_grad:
            dx = x.to_dense() if x.is_sparse else x
            dy = y.to_dense() if y.is_sparse else y
        else:
            c = sparse_grad_output(x, y)
            backward_vars = {
                "sparse_grad_output": c,
                "grad_output": c.to_dense() if c.is_sparse else c,
            }
            x.requires_grad_(True)
            y.requires_grad_(True)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 205-209 / 第 205-209 行

```python
            dx = x.to_dense().detach() if x.is_sparse else x.clone().detach()
            dy = y.to_dense().detach() if y.is_sparse else y.clone().detach()
            dx.requires_grad_(True)
            dy.requires_grad_(True)
        yield {"x": x, "y": y, "dx": dx, "dy": dy, **scipy_vars, **backward_vars}
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Representative symbols: to_coo_scipy, sparse_grad_output, read_matrix_params, csr_to_coo, load_sparse_matrix, gen_vector, gen_matrix, load_spmv_dataset** — 代表性符号：to_coo_scipy、sparse_grad_output、read_matrix_params、csr_to_coo、load_sparse_matrix、gen_vector、gen_matrix、load_spmv_dataset

## Dependencies / 依赖关系

- `math`
- `pathlib`
- `scipy`
- `torch`

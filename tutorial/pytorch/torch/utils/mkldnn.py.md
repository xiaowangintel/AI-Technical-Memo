# mkldnn.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/mkldnn.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `mkldnn.py`. Key abstractions such as `MkldnnLinear, _MkldnnConvNd` shape the public surface of the file.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `mkldnn.py` 展开。 `MkldnnLinear, _MkldnnConvNd` 等关键抽象塑造了该文件的公开接口。
## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行
```python
# mypy: allow-untyped-defs
import torch


class MkldnnLinear(torch.jit.ScriptModule):
    def __init__(self, dense_module, dtype) -> None:
        super().__init__()
        self.register_buffer('weight', dense_module.weight.to_mkldnn(dtype))
        if dense_module.bias is not None:
            # Bias can be fp32 or bf16 for OneDNN bf16 path, but for good accuracy,
            # we use fp32 dtype.
            self.register_buffer('bias', dense_module.bias.to_mkldnn())
        else:
            # TODO: Remove this once ScriptModule supports registering None buffer
            self.register_buffer(
                'bias',
                torch.zeros([dense_module.weight.size(0)], dtype=torch.float).to_mkldnn())

    @torch.jit.script_method
    def __getstate__(self):
        return (self.weight.to_dense(), self.bias.to_dense(), self.training)
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnLinear`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `MkldnnLinear` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 23-43 / 第 23-43 行
```python
    @torch.jit.script_method
    def __setstate__(self, state):
        self.weight = state[0].to_mkldnn()
        self.bias = state[1].to_mkldnn()
        self.training = state[2]

    @torch.jit.script_method
    def forward(self, x):
        x_mkldnn = x if x.is_mkldnn else x.to_mkldnn()
        y_mkldnn = torch._C._nn.mkldnn_linear(x_mkldnn, self.weight, self.bias)
        y = y_mkldnn if x.is_mkldnn else y_mkldnn.to_dense()
        return y


class _MkldnnConvNd(torch.jit.ScriptModule):
    """Common base of MkldnnConv1d and MkldnnConv2d."""

    __constants__ = ['stride', 'padding', 'dilation', 'groups']

    def __init__(self, dense_module) -> None:
        super().__init__()
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnLinear`, `_MkldnnConvNd`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `MkldnnLinear`, `_MkldnnConvNd` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 45-62 / 第 45-62 行
```python
        self.stride = dense_module.stride
        self.padding = dense_module.padding
        self.dilation = dense_module.dilation
        self.groups = dense_module.groups

        if dense_module.bias is not None:
            self.register_buffer('bias', dense_module.bias.to_mkldnn())
        else:
            # Bias can be fp32 or bf16 for OneDNN bf16 path, but for good accuracy,
            # we use fp32 dtype.
            # TODO: Remove this once ScriptModule supports registering None buffer
            self.register_buffer(
                'bias',
                torch.zeros([dense_module.weight.size(0)], dtype=torch.float).to_mkldnn())

    @torch.jit.script_method
    def __getstate__(self):
        return (self.weight.to_dense(), self.bias.to_dense(), self.training)
```
- **EN**: It introduces or extends class-level abstractions such as `_MkldnnConvNd`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_MkldnnConvNd` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 64-86 / 第 64-86 行
```python
    @torch.jit.script_method
    def forward(self, x):
        return torch.mkldnn_convolution(
            x,
            self.weight,
            self.bias,
            self.padding,
            self.stride,
            self.dilation,
            self.groups)


class MkldnnConv1d(_MkldnnConvNd):
    def __init__(self, dense_module, dtype) -> None:
        super().__init__(dense_module)

        self.register_buffer('weight', dense_module.weight.to_mkldnn(dtype))

    @torch.jit.script_method
    def __setstate__(self, state):
        self.weight = state[0].to_mkldnn()
        self.bias = state[1].to_mkldnn()
        self.training = state[2]
```
- **EN**: It introduces or extends class-level abstractions such as `_MkldnnConvNd`, `MkldnnConv1d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_MkldnnConvNd`, `MkldnnConv1d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 89-109 / 第 89-109 行
```python
class MkldnnConv2d(_MkldnnConvNd):
    def __init__(self, dense_module, dtype) -> None:
        super().__init__(dense_module)

        self.register_buffer('weight', torch._C._nn.mkldnn_reorder_conv2d_weight(
            dense_module.weight.to_mkldnn(dtype),
            self.padding,
            self.stride,
            self.dilation,
            self.groups))

    @torch.jit.script_method
    def __setstate__(self, state):
        self.weight = torch._C._nn.mkldnn_reorder_conv2d_weight(
            state[0].to_mkldnn(),
            self.padding,
            self.stride,
            self.dilation,
            self.groups)
        self.bias = state[1].to_mkldnn()
        self.training = state[2]
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnConv2d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions.
- **CN**: 它引入或扩展了 `MkldnnConv2d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。

### Lines 111-131 / 第 111-131 行
```python
class MkldnnConv3d(_MkldnnConvNd):
    def __init__(self, dense_module, dtype) -> None:
        super().__init__(dense_module)

        self.register_buffer('weight', torch._C._nn.mkldnn_reorder_conv3d_weight(
            dense_module.weight.to_mkldnn(dtype),
            self.padding,
            self.stride,
            self.dilation,
            self.groups))

    @torch.jit.script_method
    def __setstate__(self, state):
        self.weight = torch._C._nn.mkldnn_reorder_conv3d_weight(
            state[0].to_mkldnn(),
            self.padding,
            self.stride,
            self.dilation,
            self.groups)
        self.bias = state[1].to_mkldnn()
        self.training = state[2]
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnConv3d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions.
- **CN**: 它引入或扩展了 `MkldnnConv3d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。

### Lines 134-156 / 第 134-156 行
```python
class MkldnnBatchNorm(torch.jit.ScriptModule):
    __constants__ = ['exponential_average_factor', 'eps']

    def __init__(self, dense_module) -> None:
        super().__init__()

        if dense_module.training:
            raise AssertionError("Only support eval mode batchnorm for mkldnn path now")
        if not dense_module.track_running_stats:
            raise AssertionError("Only support track_running_stats=True for mkldnn path now")
        if not dense_module.affine:
            raise AssertionError("Only support affine=True for mkldnn path now")

        if dense_module.momentum is None:
            self.exponential_average_factor = 0.0
        else:
            self.exponential_average_factor = dense_module.momentum
        self.eps = dense_module.eps

        self.register_buffer('weight', dense_module.weight.to_mkldnn())
        self.register_buffer('bias', dense_module.bias.to_mkldnn())
        self.register_buffer('running_mean', dense_module.running_mean.to_mkldnn())
        self.register_buffer('running_var', dense_module.running_var.to_mkldnn())
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnBatchNorm`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `MkldnnBatchNorm` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 158-172 / 第 158-172 行
```python
    @torch.jit.script_method
    def __getstate__(self):
        weight = self.weight.to_dense()
        bias = self.bias.to_dense()
        running_mean = self.running_mean.to_dense()
        running_var = self.running_var.to_dense()
        return (weight, bias, running_mean, running_var, self.training)

    @torch.jit.script_method
    def __setstate__(self, state):
        self.weight = state[0].to_mkldnn()
        self.bias = state[1].to_mkldnn()
        self.running_mean = state[2].to_mkldnn()
        self.running_var = state[3].to_mkldnn()
        self.training = state[4]
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnBatchNorm`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `MkldnnBatchNorm` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 174-195 / 第 174-195 行
```python
    @torch.jit.script_method
    def forward(self, x):
        return torch.batch_norm(
            x,
            self.weight,
            self.bias,
            self.running_mean,
            self.running_var,
            False,  # training
            self.exponential_average_factor,
            self.eps,
            False,  # cuda_enabled
        )

class MkldnnPrelu(torch.jit.ScriptModule):
    def __init__(self, dense_module, dtype) -> None:
        super().__init__()
        self.register_buffer('weight', dense_module.weight.to_mkldnn(dtype))

    @torch.jit.script_method
    def __getstate__(self):
        return (self.weight.to_dense(), self.training)
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnBatchNorm`, `MkldnnPrelu`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `MkldnnBatchNorm`, `MkldnnPrelu` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 197-211 / 第 197-211 行
```python
    @torch.jit.script_method
    def __setstate__(self, state):
        self.weight = state[0].to_mkldnn()
        self.training = state[1]

    @torch.jit.script_method
    def forward(self, x):
        x_mkldnn = x if x.is_mkldnn else x.to_mkldnn()
        y_mkldnn = torch.prelu(x_mkldnn, self.weight)
        y = y_mkldnn if x.is_mkldnn else y_mkldnn.to_dense()
        return y

def to_mkldnn(module, dtype=torch.float):
    if dtype not in (torch.float, torch.bfloat16, torch.half):
        raise AssertionError("MKLDNN only support float, bfloat16, and half path now")
```
- **EN**: It introduces or extends class-level abstractions such as `MkldnnPrelu`, which organize state and behavior for this subsystem. Key callable entry points in this range include `to_mkldnn`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `MkldnnPrelu` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `to_mkldnn`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 214-236 / 第 214-236 行
```python
    def m_fn(m, d):
        if isinstance(m, torch.nn.Linear):
            return MkldnnLinear(m, d)
        elif isinstance(m, torch.nn.Conv1d):
            return MkldnnConv1d(m, d)
        elif isinstance(m, torch.nn.Conv2d):
            return MkldnnConv2d(m, d)
        elif isinstance(m, torch.nn.Conv3d):
            return MkldnnConv3d(m, d)
        elif isinstance(m, (torch.nn.BatchNorm2d, torch.nn.BatchNorm3d)):
            # For batchnorm bf16 path, OneDNN requires weight and bias need fp32 dtype.
            # so it doesn't need dtype argument.
            return MkldnnBatchNorm(m)
        elif isinstance(m, torch.nn.PReLU):
            return MkldnnPrelu(m, d)
        else:
            return m

    def m_fn_rec(m, d):
        new_m = m_fn(m, d)
        for name, sub_m in m.named_children():
            setattr(new_m, name, m_fn_rec(sub_m, d))
        return new_m
```
- **EN**: Key callable entry points in this range include `to_mkldnn`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `to_mkldnn`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 238-238 / 第 238-238 行
```python
    return m_fn_rec(module, dtype)
```
- **EN**: Key callable entry points in this range include `to_mkldnn`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `to_mkldnn`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **MkldnnLinear**
  - EN: `MkldnnLinear` is one of the main classes that structures the file's behavior.
  - CN: `MkldnnLinear` 是组织该文件行为的核心类之一。
- **_MkldnnConvNd**
  - EN: `_MkldnnConvNd` is one of the main classes that structures the file's behavior.
  - CN: `_MkldnnConvNd` 是组织该文件行为的核心类之一。
- **to_mkldnn**
  - EN: `to_mkldnn` is a representative function that exposes or coordinates an important action in this module.
  - CN: `to_mkldnn` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`
- **Primary symbols / 核心符号**: `MkldnnLinear`, `_MkldnnConvNd`, `MkldnnConv1d`, `MkldnnConv2d`, `MkldnnConv3d`, `MkldnnBatchNorm`, `MkldnnPrelu`, `to_mkldnn`

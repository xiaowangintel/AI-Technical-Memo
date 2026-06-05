# _functions.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/modules/_functions.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines reusable nn.Module classes, their state, and their forward-path helpers.
- **Purpose (CN)**: 定义可复用的 nn.Module 类、其状态以及前向路径辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
# mypy: allow-untyped-defs
import torch
import torch.distributed as dist
from torch.autograd.function import Function


class SyncBatchNorm(Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(
        self,
        input,
        weight,
        bias,
        running_mean,
        running_var,
        eps,
        momentum,
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 19-40
```python
        process_group,
        world_size,
    ):
        if not (
            input.is_contiguous(memory_format=torch.channels_last)
            or input.is_contiguous(memory_format=torch.channels_last_3d)
        ):
            input = input.contiguous()
        if weight is not None:
            weight = weight.contiguous()

        size = int(input.numel() // input.size(1))
        if size == 1 and world_size < 2:
            raise ValueError(
                f"Expected more than 1 value per channel when training, got input size {size}"
            )

        num_channels = input.shape[1]
        if input.numel() > 0:
            # calculate mean/invstd for input.
            mean, invstd = torch.batch_norm_stats(input, eps)
```
- **EN**: This block continues `SyncBatchNorm.forward` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.forward`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 41-58
```python
            count = torch.full(
                (1,),
                input.numel() // input.size(1),
                dtype=mean.dtype,
                device=mean.device,
            )

            # C, C, 1 -> (2C + 1)
            combined = torch.cat([mean, invstd, count], dim=0)
        else:
            # for empty input, set stats and the count to zero. The stats with
            # zero count will be filtered out later when computing global mean
            # & invstd, but they still needs to participate the all_gather
            # collective communication to unblock other peer processes.
            combined = torch.zeros(
                2 * num_channels + 1, dtype=input.dtype, device=input.device
            )
```
- **EN**: This block continues `SyncBatchNorm.forward` and works to normalize dtype/device related arguments and behavior.
- **CN**: 该代码块继续实现 `SyncBatchNorm.forward`，用于规范化 dtype/device 相关参数与行为。

### Lines 59-76
```python
        # Use allgather instead of allreduce because count could be different across
        # ranks, simple all reduce op can not give correct results.
        # batch_norm_gather_stats_with_counts calculates global mean & invstd based on
        # all gathered mean, invstd and count.
        # for nccl backend, use the optimized version of all gather.
        # The Gloo backend does not support `all_gather_into_tensor`.
        if process_group._get_backend_name() != "gloo":
            # world_size * (2C + 1)
            combined_size = combined.numel()
            combined_flat = torch.empty(
                1,
                combined_size * world_size,
                dtype=combined.dtype,
                device=combined.device,
            )
            dist.all_gather_into_tensor(
                combined_flat, combined, process_group, async_op=False
            )
```
- **EN**: This block continues `SyncBatchNorm.forward` and works to hand work to a compiler/backend pipeline. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.forward`，用于将工作移交给编译器或后端流水线。 同时它还会根据运行时条件分支处理。

### Lines 77-101
```python
            combined = torch.reshape(combined_flat, (world_size, combined_size))
            # world_size * (2C + 1) -> world_size * C, world_size * C, world_size * 1
            mean_all, invstd_all, count_all = torch.split(combined, num_channels, dim=1)
        else:
            # world_size * (2C + 1)
            combined_list = [torch.empty_like(combined) for _ in range(world_size)]
            dist.all_gather(combined_list, combined, process_group, async_op=False)
            combined = torch.stack(combined_list, dim=0)
            # world_size * (2C + 1) -> world_size * C, world_size * C, world_size * 1
            mean_all, invstd_all, count_all = torch.split(combined, num_channels, dim=1)

        if not (torch.cuda.is_available() and torch.cuda.is_current_stream_capturing()):
            # The lines below force a synchronization between CUDA and CPU, because
            # the shape of the result count_all depends on the values in mask tensor.
            # Such synchronizations break CUDA Graph capturing.
            # See https://github.com/pytorch/pytorch/issues/78549
            # FIXME: https://github.com/pytorch/pytorch/issues/78656 describes
            # a better longer-term solution.

            # remove stats from empty inputs
            mask = count_all.squeeze(-1) >= 1
            count_all = count_all[mask]
            mean_all = mean_all[mask]
            invstd_all = invstd_all[mask]
```
- **EN**: This block continues `SyncBatchNorm.forward` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.forward`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 102-125
```python
        # calculate global mean & invstd
        counts = count_all.view(-1)
        if running_mean is not None and counts.dtype != running_mean.dtype:
            counts = counts.to(running_mean.dtype)
        mean, invstd = torch.batch_norm_gather_stats_with_counts(
            input,
            mean_all,
            invstd_all,
            running_mean,
            running_var,
            momentum,
            eps,
            counts,
        )

        self.save_for_backward(input, weight, mean, invstd, count_all.to(torch.int32))
        self.process_group = process_group

        # apply element-wise normalization
        if input.numel() > 0:
            return torch.batch_norm_elemt(input, weight, bias, mean, invstd, eps)
        else:
            return torch.empty_like(input)
```
- **EN**: This block continues `SyncBatchNorm` and works to connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm`，用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 126-143
```python
    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(self, grad_output):
        if not (
            grad_output.is_contiguous(memory_format=torch.channels_last)
            or grad_output.is_contiguous(memory_format=torch.channels_last_3d)
        ):
            grad_output = grad_output.contiguous()
        saved_input, weight, mean, invstd, count_tensor = self.saved_tensors
        grad_input = grad_weight = grad_bias = None
        process_group = self.process_group

        if saved_input.numel() > 0:
            # calculate local stats as well as grad_weight / grad_bias
            (
                sum_dy,
                sum_dy_xmu,
                grad_weight,
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 144-167
```python
                grad_bias,
            ) = torch.batch_norm_backward_reduce(
                grad_output,
                saved_input,
                mean,
                invstd,
                weight,
                self.needs_input_grad[0],
                self.needs_input_grad[1],
                self.needs_input_grad[2],
            )

            if self.needs_input_grad[0]:
                # synchronizing stats used to calculate input gradient.
                num_channels = sum_dy.shape[0]
                combined = torch.cat([sum_dy, sum_dy_xmu], dim=0)
                torch.distributed.all_reduce(
                    combined,
                    torch.distributed.ReduceOp.SUM,
                    process_group,
                    async_op=False,
                )
                sum_dy, sum_dy_xmu = torch.split(combined, num_channels)
```
- **EN**: This block continues `SyncBatchNorm.backward` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.backward`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 168-185
```python
                # backward pass for gradient calculation
                if weight is not None and weight.dtype != mean.dtype:
                    weight = weight.to(mean.dtype)
                grad_input = torch.batch_norm_backward_elemt(
                    grad_output,
                    saved_input,
                    mean,
                    invstd,
                    weight,
                    sum_dy,
                    sum_dy_xmu,
                    count_tensor,
                )
            # synchronizing of grad_weight / grad_bias is not needed as distributed
            # training would handle all reduce.
            if weight is None or not self.needs_input_grad[1]:
                grad_weight = None
```
- **EN**: This block continues `SyncBatchNorm.backward` and works to connect execution with differentiation-aware logic. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `SyncBatchNorm.backward`，用于将执行过程与可微分逻辑连接起来。 同时它还会根据运行时条件分支处理。

### Lines 186-212
```python
            if weight is None or not self.needs_input_grad[2]:
                grad_bias = None
        else:
            # This process got an empty input tensor in the forward pass.
            # Although this process can directly set grad_input as an empty
            # tensor of zeros, it still needs to participate in the collective
            # communication to unblock its peers, as other peer processes might
            # have received non-empty inputs.
            num_channels = saved_input.shape[1]
            if self.needs_input_grad[0]:
                # launch all_reduce to unblock other peer processes
                combined = torch.zeros(
                    2 * num_channels, dtype=saved_input.dtype, device=saved_input.device
                )
                torch.distributed.all_reduce(
                    combined,
                    torch.distributed.ReduceOp.SUM,
                    process_group,
                    async_op=False,
                )

            # Leave grad_input, grad_weight and grad_bias as None, which will be
            # interpreted by the autograd engine as Tensors full of zeros.

        return grad_input, grad_weight, grad_bias, None, None, None, None, None, None
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 213-238
```python
class CrossMapLRN2d(Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, input, size, alpha=1e-4, beta=0.75, k=1):
        ctx.size = size
        ctx.alpha = alpha
        ctx.beta = beta
        ctx.k = k
        ctx.scale = None

        if input.dim() != 4:
            raise ValueError(
                f"CrossMapLRN2d: Expected input to be 4D, got {input.dim()}D instead."
            )

        ctx.scale = ctx.scale or input.new()
        output = input.new()
        channels = input.size(1)

        output.resize_as_(input)
        ctx.scale.resize_as_(input)

        # use output storage as temporary buffer
        input_square = output
        torch.pow(input, 2, out=input_square)
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 239-263
```python
        pre_pad = int((ctx.size - 1) / 2 + 1)
        pre_pad_crop = min(pre_pad, channels)

        scale_first = ctx.scale.select(1, 0)
        scale_first.zero_()
        # compute first feature map normalization
        for c in range(pre_pad_crop):
            scale_first.add_(input_square.select(1, c))

        # reuse computations for next feature maps normalization
        # by adding the next feature map and removing the previous
        for c in range(1, channels):
            scale_previous = ctx.scale.select(1, c - 1)
            scale_current = ctx.scale.select(1, c)
            scale_current.copy_(scale_previous)
            if c < channels - pre_pad + 1:
                square_next = input_square.select(1, c + pre_pad - 1)
                scale_current.add_(square_next, alpha=1)

            if c > pre_pad:
                square_previous = input_square.select(1, c - pre_pad)
                scale_current.add_(square_previous, alpha=-1)

        ctx.scale.mul_(ctx.alpha / ctx.size).add_(ctx.k)
```
- **EN**: This block continues `CrossMapLRN2d.forward` and works to prepare neural-network operators or module behavior. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CrossMapLRN2d.forward`，用于准备神经网络算子或模块行为。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 264-289
```python
        torch.pow(ctx.scale, -ctx.beta, out=output)
        output.mul_(input)

        ctx.save_for_backward(input, output)
        return output

    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(ctx, grad_output):
        input, output = ctx.saved_tensors
        grad_input = grad_output.new()

        batch_size = input.size(0)
        channels = input.size(1)
        input_height = input.size(2)
        input_width = input.size(3)

        paddded_ratio = input.new(channels + ctx.size - 1, input_height, input_width)
        accum_ratio = input.new(input_height, input_width)

        cache_ratio_value = 2 * ctx.alpha * ctx.beta / ctx.size
        inversePrePad = int(ctx.size - (ctx.size - 1) / 2)

        grad_input.resize_as_(input)
        torch.pow(ctx.scale, -ctx.beta, out=grad_input).mul_(grad_output)
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 290-310
```python
        paddded_ratio.zero_()
        padded_ratio_center = paddded_ratio.narrow(0, inversePrePad, channels)
        for n in range(batch_size):
            torch.mul(grad_output[n], output[n], out=padded_ratio_center)
            padded_ratio_center.div_(ctx.scale[n])
            torch.sum(
                paddded_ratio.narrow(0, 0, ctx.size - 1),
                0,
                keepdim=False,
                out=accum_ratio,
            )
            for c in range(channels):
                accum_ratio.add_(paddded_ratio[c + ctx.size - 1])
                grad_input[n][c].addcmul_(
                    input[n][c], accum_ratio, value=-cache_ratio_value
                )
                accum_ratio.add_(paddded_ratio[c], alpha=-1)

        return grad_input, None, None, None, None
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 311-320
```python
class BackwardHookFunction(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, *args):
        ctx.mark_non_differentiable(*[arg for arg in args if not arg.requires_grad])
        return args

    @staticmethod
    def backward(ctx, *args):
        return args
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.distributed`, `torch.autograd.function`
- **Primary symbols / 核心符号**: `SyncBatchNorm`, `CrossMapLRN2d`, `BackwardHookFunction`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。

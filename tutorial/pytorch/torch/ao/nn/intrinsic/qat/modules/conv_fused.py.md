# conv_fused.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/intrinsic/qat/modules/conv_fused.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `conv_fused.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `conv_fused.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-32 / 第 1-32 行
```python
# mypy: allow-untyped-defs
import math
from typing import ClassVar

import torch
import torch.ao.nn.intrinsic as nni
import torch.ao.nn.qat as nnqat
import torch.nn as nn
import torch.nn.functional as F
from torch.nn import init
from torch.nn.modules.utils import _pair, _single, _triple
from torch.nn.parameter import Parameter
from torch.nn.utils import fuse_conv_bn_weights


__all__ = [
    "ConvBn1d",
    "ConvBnReLU1d",
    "ConvReLU1d",
    "ConvBn2d",
    "ConvBnReLU2d",
    "ConvReLU2d",
    "ConvBn3d",
    "ConvBnReLU3d",
    "ConvReLU3d",
    "update_bn_stats",
    "freeze_bn_stats",
]
_BN_CLASS_MAP = {
    1: nn.BatchNorm1d,
    2: nn.BatchNorm2d,
    3: nn.BatchNorm3d,
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Named constants such as `_BN_CLASS_MAP` centralize shared configuration or sentinel values.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 `_BN_CLASS_MAP` 等具名常量把共享配置或哨兵值集中定义在一起。

### Lines 33-64 / 第 33-64 行
```python
}


class _ConvBnNd(nn.modules.conv._ConvNd, nni._FusedModule):
    _version = 2
    _FLOAT_MODULE: ClassVar[type[nn.modules.conv._ConvNd]]

    def __init__(
        self,
        # ConvNd args
        in_channels,
        out_channels,
        kernel_size,
        stride,
        padding,
        dilation,
        transposed,
        output_padding,
        groups,
        bias,
        padding_mode,
        # BatchNormNd args
        # num_features: out_channels
        eps=1e-05,
        momentum=0.1,
        # affine: True
        # track_running_stats: True
        # Args for this module
        freeze_bn=False,
        qconfig=None,
        dim=2,
    ):
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 65-89 / 第 65-89 行
```python
        nn.modules.conv._ConvNd.__init__(
            self,
            in_channels,
            out_channels,
            kernel_size,
            stride,
            padding,
            dilation,
            transposed,
            output_padding,
            groups,
            False,
            padding_mode,
        )
        if not qconfig:
            raise AssertionError("qconfig must be provided for QAT module")
        self.qconfig = qconfig
        self.freeze_bn = freeze_bn if self.training else True
        self.bn = _BN_CLASS_MAP[dim](out_channels, eps, momentum, True, True)
        self.weight_fake_quant = self.qconfig.weight()
        if bias:
            self.bias = Parameter(torch.empty(out_channels))
        else:
            self.register_parameter("bias", None)
        self.reset_bn_parameters()
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 91-119 / 第 91-119 行
```python
        # this needs to be called after reset_bn_parameters,
        # as they modify the same state
        if self.training:
            if freeze_bn:
                self.freeze_bn_stats()
            else:
                self.update_bn_stats()
        else:
            self.freeze_bn_stats()

        self._enable_slow_path_for_better_numerical_stability = False

    def reset_running_stats(self):
        self.bn.reset_running_stats()

    def reset_bn_parameters(self):
        self.bn.reset_running_stats()
        init.uniform_(self.bn.weight)
        init.zeros_(self.bn.bias)
        # note: below is actually for conv, not BN
        if self.bias is not None:
            fan_in, _ = init._calculate_fan_in_and_fan_out(self.weight)
            bound = 1 / math.sqrt(fan_in)
            init.uniform_(self.bias, -bound, bound)

    def update_bn_stats(self):
        self.freeze_bn = False
        self.bn.training = True
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 121-152 / 第 121-152 行
```python
    def freeze_bn_stats(self):
        self.freeze_bn = True
        self.bn.training = False
        return self

    def _forward(self, input):
        if self._enable_slow_path_for_better_numerical_stability:
            return self._forward_slow(input)
        return self._forward_approximate(input)

    def _forward_approximate(self, input):
        """Approximated method to fuse conv and bn. It requires only one forward pass.
        conv_orig = conv / scale_factor where scale_factor = bn.weight / running_std
        """
        if self.bn.running_var is None:
            raise AssertionError("self.bn.running_var must not be None")
        running_std = torch.sqrt(self.bn.running_var + self.bn.eps)
        scale_factor = self.bn.weight / running_std
        weight_shape = [1] * len(self.weight.shape)
        weight_shape[0] = -1
        bias_shape = [1] * len(self.weight.shape)
        bias_shape[1] = -1
        scaled_weight = self.weight_fake_quant(
            self.weight * scale_factor.reshape(weight_shape)
        )
        # using zero bias here since the bias for original conv
        # will be added later
        if self.bias is not None:
            zero_bias = torch.zeros_like(self.bias, dtype=input.dtype)
        else:
            zero_bias = torch.zeros(
                self.out_channels, device=scaled_weight.device, dtype=input.dtype
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 153-183 / 第 153-183 行
```python
            )
        conv = self._conv_forward(input, scaled_weight, zero_bias)
        conv_orig = conv / scale_factor.reshape(bias_shape)
        if self.bias is not None:
            conv_orig = conv_orig + self.bias.reshape(bias_shape)
        conv = self.bn(conv_orig)
        return conv

    def _forward_slow(self, input):
        """
        A more accurate but slow method to compute conv bn fusion, following https://arxiv.org/pdf/1806.08342.pdf
        It requires two forward passes but handles the case bn.weight == 0

        Conv: Y = WX + B_c
        Conv without bias: Y0 = WX = Y - B_c, Y = Y0 + B_c

        Batch statistics:
          mean_Y = Y.mean()
                 = Y0.mean() + B_c
          var_Y = (Y - mean_Y)^2.mean()
                = (Y0 - Y0.mean())^2.mean()
        BN (r: bn.weight, beta: bn.bias):
          Z = r * (Y - mean_Y) / sqrt(var_Y + eps) + beta
            = r * (Y0 - Y0.mean()) / sqrt(var_Y + eps) + beta

        Fused Conv BN training (std_Y = sqrt(var_Y + eps)):
          Z = (r * W / std_Y) * X + r * (B_c - mean_Y) / std_Y + beta
            = (r * W / std_Y) * X - r * Y0.mean() / std_Y + beta

        Fused Conv BN inference (running_std = sqrt(running_var + eps)):
          Z = (r * W / running_std) * X - r * (running_mean - B_c) / running_std + beta
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Named constants such as `Z`, `Z`, `Z` centralize shared configuration or sentinel values. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 `Z, Z, Z` 等具名常量把共享配置或哨兵值集中定义在一起。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 185-205 / 第 185-205 行
```python
        QAT with fused conv bn:
          Z_train = fake_quant(r * W / running_std) * X * (running_std / std_Y) - r * Y0.mean() / std_Y + beta
                  = conv(X, fake_quant(r * W / running_std)) * (running_std / std_Y) - r * Y0.mean() / std_Y + beta
          Z_inference = conv(X, fake_quant(r * W / running_std)) - r * (running_mean - B_c) / running_std + beta
        """

        if self.bn.running_var is None:
            raise AssertionError("self.bn.running_var must not be None")
        if self.bn.running_mean is None:
            raise AssertionError("self.bn.running_mean must not be None")

        # using zero bias here since the bias for original conv
        # will be added later
        zero_bias = torch.zeros(
            self.out_channels, device=self.weight.device, dtype=input.dtype
        )

        weight_shape = [1] * len(self.weight.shape)
        weight_shape[0] = -1
        bias_shape = [1] * len(self.weight.shape)
        bias_shape[1] = -1
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 207-233 / 第 207-233 行
```python
        if self.bn.training:
            # needed to compute batch mean/std
            conv_out = self._conv_forward(input, self.weight, zero_bias)
            # update bn statistics
            with torch.no_grad():
                conv_out_bias = (
                    conv_out
                    if self.bias is None
                    else conv_out + self.bias.reshape(bias_shape)
                )
                self.bn(conv_out_bias)

            # fused conv + bn without bias using bn running statistics
            running_std = torch.sqrt(self.bn.running_var + self.bn.eps)
            scale_factor = self.bn.weight / running_std
            scaled_weight = self.weight_fake_quant(
                self.weight * scale_factor.reshape(weight_shape)
            )
            # fused conv without bias for inference: (r * W / running_std) * X
            conv_bn = self._conv_forward(input, scaled_weight, zero_bias)

            avg_dims = [0] + list(range(2, len(self.weight.shape)))
            batch_mean = conv_out.mean(avg_dims)
            batch_var = torch.square(conv_out - batch_mean.reshape(bias_shape)).mean(
                avg_dims
            )
            batch_std = torch.sqrt(batch_var + self.bn.eps)
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 235-264 / 第 235-264 行
```python
            # scale to use batch std in training mode
            # conv(X, r * W / std_Y) = conv(X, r * W / running_std) * (running_std / std_Y)
            unscale_factor = running_std / batch_std
            conv_bn *= unscale_factor.reshape(bias_shape)

            fused_mean = batch_mean
            fused_std = batch_std
        else:
            # fused conv + bn without bias using bn running statistics
            running_std = torch.sqrt(self.bn.running_var + self.bn.eps)
            scale_factor = self.bn.weight / running_std
            scaled_weight = self.weight_fake_quant(
                self.weight * scale_factor.reshape(weight_shape)
            )
            # fused conv without bias for inference: (r * W / running_std) * X
            conv_bn = self._conv_forward(input, scaled_weight, zero_bias)

            fused_mean = self.bn.running_mean - (
                self.bias if self.bias is not None else 0
            )
            fused_std = running_std

        # fused bias = beta - r * mean / std
        fused_bias = self.bn.bias - self.bn.weight * fused_mean / fused_std
        conv_bn += fused_bias.reshape(bias_shape)

        # HACK to let conv bias participate in loss to avoid DDP error (parameters
        #   were not used in producing loss)
        if self.bias is not None:
            conv_bn += (self.bias - self.bias).reshape(bias_shape)
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 266-297 / 第 266-297 行
```python
        return conv_bn

    def forward(self, input):
        return self._forward(input)

    def train(self, mode=True):
        """
        Batchnorm's training behavior is using the self.training flag. Prevent
        changing it if BN is frozen. This makes sure that calling `model.train()`
        on a model with a frozen BN will behave properly.
        """
        self.training = mode
        if not self.freeze_bn:
            for module in self.children():
                module.train(mode)
        return self

    # ===== Serialization version history =====
    #
    # Version 1/None
    #   self
    #   |--- weight : Tensor
    #   |--- bias : Tensor
    #   |--- gamma : Tensor
    #   |--- beta : Tensor
    #   |--- running_mean : Tensor
    #   |--- running_var : Tensor
    #   |--- num_batches_tracked : Tensor
    #
    # Version 2
    #   self
    #   |--- weight : Tensor
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 298-329 / 第 298-329 行
```python
    #   |--- bias : Tensor
    #   |--- bn : Module
    #        |--- weight : Tensor (moved from v1.self.gamma)
    #        |--- bias : Tensor (moved from v1.self.beta)
    #        |--- running_mean : Tensor (moved from v1.self.running_mean)
    #        |--- running_var : Tensor (moved from v1.self.running_var)
    #        |--- num_batches_tracked : Tensor (moved from v1.self.num_batches_tracked)
    def _load_from_state_dict(
        self,
        state_dict,
        prefix,
        local_metadata,
        strict,
        missing_keys,
        unexpected_keys,
        error_msgs,
    ):
        version = local_metadata.get("version", None)
        if version is None or version == 1:
            # BN related parameters and buffers were moved into the BN module for v2
            v2_to_v1_names = {
                "bn.weight": "gamma",
                "bn.bias": "beta",
                "bn.running_mean": "running_mean",
                "bn.running_var": "running_var",
                "bn.num_batches_tracked": "num_batches_tracked",
            }
            for v2_name, v1_name in v2_to_v1_names.items():
                if prefix + v1_name in state_dict:
                    state_dict[prefix + v2_name] = state_dict[prefix + v1_name]
                    state_dict.pop(prefix + v1_name)
                elif prefix + v2_name in state_dict:
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 330-354 / 第 330-354 行
```python
                    # there was a brief period where forward compatibility
                    # for this module was broken (between
                    # https://github.com/pytorch/pytorch/pull/38478
                    # and https://github.com/pytorch/pytorch/pull/38820)
                    # and modules emitted the v2 state_dict format while
                    # specifying that version == 1. This patches the forward
                    # compatibility issue by allowing the v2 style entries to
                    # be used.
                    pass
                elif strict:
                    missing_keys.append(prefix + v2_name)

        super()._load_from_state_dict(
            state_dict,
            prefix,
            local_metadata,
            strict,
            missing_keys,
            unexpected_keys,
            error_msgs,
        )

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        r"""Create a qat module from a float module or qparams_dict
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 356-387 / 第 356-387 行
```python
        Args: `mod` a float module, either produced by torch.ao.quantization utilities
        or directly from user
        """
        # The ignore is because _FLOAT_MODULE is a TypeVar here where the bound
        # has no __name__ (code is fine though)
        if type(mod) is not cls._FLOAT_MODULE:
            raise AssertionError(
                "qat."
                + cls.__name__
                + ".from_float only works for "
                + cls._FLOAT_MODULE.__name__
            )
        if not hasattr(mod, "qconfig"):
            raise AssertionError("Input float module must have qconfig defined")
        if not mod.qconfig:
            raise AssertionError("Input float module must have a valid qconfig")
        qconfig = mod.qconfig
        conv, bn = mod[0], mod[1]  # type: ignore[index]
        qat_convbn = cls(
            conv.in_channels,
            conv.out_channels,
            conv.kernel_size,
            conv.stride,
            conv.padding,
            conv.dilation,
            conv.groups,
            conv.bias is not None,
            conv.padding_mode,
            bn.eps,
            bn.momentum,
            False,
            qconfig,
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 388-414 / 第 388-414 行
```python
        )
        qat_convbn.weight = conv.weight
        qat_convbn.bias = conv.bias
        qat_convbn.bn.weight = bn.weight
        qat_convbn.bn.bias = bn.bias
        qat_convbn.bn.running_mean = bn.running_mean
        qat_convbn.bn.running_var = bn.running_var
        # mypy error: Cannot determine type of 'num_batches_tracked'
        qat_convbn.bn.num_batches_tracked = bn.num_batches_tracked
        return qat_convbn

    def to_float(self):
        cls = type(self)
        conv = cls._FLOAT_CONV_MODULE(  # type: ignore[attr-defined]
            self.in_channels,
            self.out_channels,
            self.kernel_size,
            self.stride,
            self.padding,
            self.dilation,
            self.groups,
            self.bias is not None,
            self.padding_mode,
        )
        conv.weight = torch.nn.Parameter(self.weight.detach())
        if self.bias is not None:
            conv.bias = torch.nn.Parameter(self.bias.detach())
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 416-442 / 第 416-442 行
```python
        if cls._FLOAT_BN_MODULE:  # type: ignore[attr-defined]
            # fuse bn into conv
            if self.bn.running_var is None or self.bn.running_mean is None:
                raise AssertionError(
                    "self.bn.running_var and self.bn.running_mean must not be None"
                )
            conv.weight, conv.bias = fuse_conv_bn_weights(
                conv.weight,
                conv.bias,
                self.bn.running_mean,
                self.bn.running_var,
                self.bn.eps,
                self.bn.weight,
                self.bn.bias,
            )

        if cls._FLOAT_RELU_MODULE:  # type: ignore[attr-defined]
            modules = []
            modules.append(conv)
            relu = cls._FLOAT_RELU_MODULE()  # type: ignore[attr-defined]
            modules.append(relu)
            conv_relu = cls._FUSED_FLOAT_MODULE(*modules)  # type: ignore[attr-defined]
            conv_relu.train(self.training)
            return conv_relu
        else:
            conv.train(self.training)
            return conv
```
- **EN**: It introduces or extends class-level abstractions such as `_ConvBnNd`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `_ConvBnNd` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 445-466 / 第 445-466 行
```python
class ConvBn1d(_ConvBnNd, nn.Conv1d):
    r"""
    A ConvBn1d module is a module fused from Conv1d and BatchNorm1d,
    attached with FakeQuantize modules for weight,
    used in quantization aware training.

    We combined the interface of :class:`torch.nn.Conv1d` and
    :class:`torch.nn.BatchNorm1d`.

    Similar to :class:`torch.nn.Conv1d`, with FakeQuantize modules initialized
    to default.

    Attributes:
        freeze_bn:
        weight_fake_quant: fake quant module for weight

    """

    _FLOAT_BN_MODULE: ClassVar[type[nn.BatchNorm1d]] = nn.BatchNorm1d
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = None
    _FLOAT_MODULE: ClassVar[type[nn.Module]] = nni.ConvBn1d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv1d]] = nn.Conv1d
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn1d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBn1d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 468-499 / 第 468-499 行
```python
    def __init__(
        self,
        # Conv1d args
        in_channels,
        out_channels,
        kernel_size,
        stride=1,
        padding=0,
        dilation=1,
        groups=1,
        bias=None,
        padding_mode="zeros",
        # BatchNorm1d args
        # num_features: out_channels
        eps=1e-05,
        momentum=0.1,
        # affine: True
        # track_running_stats: True
        # Args for this module
        freeze_bn=False,
        qconfig=None,
    ):
        kernel_size = _single(kernel_size)
        stride = _single(stride)
        padding = _single(padding)
        dilation = _single(dilation)
        _ConvBnNd.__init__(
            self,
            in_channels,
            out_channels,
            kernel_size,
            stride,
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn1d`, which organize state and behavior for this subsystem. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `ConvBn1d` 等类级抽象，用于组织该子系统的状态与行为。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 500-530 / 第 500-530 行
```python
            padding,
            dilation,
            False,
            _single(0),
            groups,
            bias,
            padding_mode,
            eps,
            momentum,
            freeze_bn,
            qconfig,
            dim=1,
        )


class ConvBnReLU1d(ConvBn1d):
    r"""
    A ConvBnReLU1d module is a module fused from Conv1d, BatchNorm1d and ReLU,
    attached with FakeQuantize modules for weight,
    used in quantization aware training.

    We combined the interface of :class:`torch.nn.Conv1d` and
    :class:`torch.nn.BatchNorm1d` and :class:`torch.nn.ReLU`.

    Similar to `torch.nn.Conv1d`, with FakeQuantize modules initialized to
    default.

    Attributes:
        weight_fake_quant: fake quant module for weight

    """
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn1d`, `ConvBnReLU1d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBn1d`, `ConvBnReLU1d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 532-561 / 第 532-561 行
```python
    # base class defines _FLOAT_MODULE as "ConvBn1d"
    _FLOAT_MODULE: ClassVar[type[nn.Module]] = nni.ConvBnReLU1d
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv1d]] = nn.Conv1d
    _FLOAT_BN_MODULE: ClassVar[type[nn.BatchNorm1d]] = nn.BatchNorm1d
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = nn.ReLU
    # module class after fusing bn into conv
    _FUSED_FLOAT_MODULE: ClassVar[type[nn.Module] | None] = nni.ConvReLU1d

    def forward(self, input):
        r"""Performs forward pass through fused Conv1d, BatchNorm1d, and ReLU."""
        return F.relu(self._forward(input))

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        r"""Creates a QAT module from a floating point module."""
        return super().from_float(mod, use_precomputed_fake_quant)


class ConvReLU1d(nnqat.Conv1d, nni._FusedModule):
    r"""A ConvReLU1d module is a fused module of Conv1d and ReLU, attached with
    FakeQuantize modules for weight for
    quantization aware training.

    We combined the interface of :class:`~torch.nn.Conv1d` and
    :class:`~torch.nn.BatchNorm1d`.

    Attributes:
        weight_fake_quant: fake quant module for weight

    """
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBnReLU1d`, `ConvReLU1d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBnReLU1d`, `ConvReLU1d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 563-594 / 第 563-594 行
```python
    _FLOAT_MODULE: ClassVar[type[nni.ConvReLU1d]] = nni.ConvReLU1d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv1d]] = nn.Conv1d
    _FLOAT_BN_MODULE: ClassVar[type[nn.Module] | None] = None
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = nn.ReLU

    def __init__(
        self,
        in_channels,
        out_channels,
        kernel_size,
        stride=1,
        padding=0,
        dilation=1,
        groups=1,
        bias=True,
        padding_mode="zeros",
        qconfig=None,
    ):
        super().__init__(
            in_channels,
            out_channels,
            kernel_size,
            stride=stride,
            padding=padding,
            dilation=dilation,
            groups=groups,
            bias=bias,
            # pyrefly: ignore [bad-argument-type]
            padding_mode=padding_mode,
            qconfig=qconfig,
        )
        if not qconfig:
```
- **EN**: It introduces or extends class-level abstractions such as `ConvReLU1d`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `ConvReLU1d` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 595-623 / 第 595-623 行
```python
            raise AssertionError("qconfig must be provided for QAT module")
        self.qconfig = qconfig
        self.weight_fake_quant = self.qconfig.weight()

    def forward(self, input):
        r"""Performs forward pass through fused Conv1d and ReLU."""
        return F.relu(
            self._conv_forward(input, self.weight_fake_quant(self.weight), self.bias)
        )

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):  # type: ignore[override]
        r"""Creates a QAT module from a floating point module."""
        return super().from_float(
            mod, use_precomputed_fake_quant=use_precomputed_fake_quant
        )


class ConvBn2d(_ConvBnNd, nn.Conv2d):
    r"""
    A ConvBn2d module is a module fused from Conv2d and BatchNorm2d,
    attached with FakeQuantize modules for weight,
    used in quantization aware training.

    We combined the interface of :class:`torch.nn.Conv2d` and
    :class:`torch.nn.BatchNorm2d`.

    Similar to :class:`torch.nn.Conv2d`, with FakeQuantize modules initialized
    to default.
```
- **EN**: It introduces or extends class-level abstractions such as `ConvReLU1d`, `ConvBn2d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvReLU1d`, `ConvBn2d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 625-656 / 第 625-656 行
```python
    Attributes:
        freeze_bn:
        weight_fake_quant: fake quant module for weight

    """

    _FLOAT_MODULE: ClassVar[type[nni.ConvBn2d]] = nni.ConvBn2d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv2d]] = nn.Conv2d
    _FLOAT_BN_MODULE: ClassVar[type[nn.Module] | None] = nn.BatchNorm2d
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = None

    def __init__(
        self,
        # ConvNd args
        in_channels,
        out_channels,
        kernel_size,
        stride=1,
        padding=0,
        dilation=1,
        groups=1,
        bias=None,
        padding_mode="zeros",
        # BatchNorm2d args
        # num_features: out_channels
        eps=1e-05,
        momentum=0.1,
        # affine: True
        # track_running_stats: True
        # Args for this module
        freeze_bn=False,
        qconfig=None,
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn2d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBn2d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 657-687 / 第 657-687 行
```python
    ):
        kernel_size = _pair(kernel_size)
        stride = _pair(stride)
        padding = _pair(padding)
        dilation = _pair(dilation)
        _ConvBnNd.__init__(
            self,
            in_channels,
            out_channels,
            kernel_size,
            stride,
            padding,
            dilation,
            False,
            _pair(0),
            groups,
            bias,
            padding_mode,
            eps,
            momentum,
            freeze_bn,
            qconfig,
            dim=2,
        )


class ConvBnReLU2d(ConvBn2d):
    r"""
    A ConvBnReLU2d module is a module fused from Conv2d, BatchNorm2d and ReLU,
    attached with FakeQuantize modules for weight,
    used in quantization aware training.
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn2d`, `ConvBnReLU2d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBn2d`, `ConvBnReLU2d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 689-715 / 第 689-715 行
```python
    We combined the interface of :class:`torch.nn.Conv2d` and
    :class:`torch.nn.BatchNorm2d` and :class:`torch.nn.ReLU`.

    Similar to `torch.nn.Conv2d`, with FakeQuantize modules initialized to
    default.

    Attributes:
        weight_fake_quant: fake quant module for weight

    """

    # base class defines _FLOAT_MODULE as "ConvBn2d"
    _FLOAT_MODULE: ClassVar[type[nni.ConvBnReLU2d]] = nni.ConvBnReLU2d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv2d]] = nn.Conv2d
    _FLOAT_BN_MODULE: ClassVar[type[nn.BatchNorm2d]] = nn.BatchNorm2d
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = nn.ReLU
    # module class after fusing bn into conv
    _FUSED_FLOAT_MODULE: ClassVar[type[nni.ConvReLU2d] | None] = nni.ConvReLU2d

    def forward(self, input):
        r"""Performs forward pass through fused Conv2d, BatchNorm2d, and ReLU."""
        return F.relu(self._forward(input))

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        r"""Creates a QAT module from a floating point module."""
        return super().from_float(mod, use_precomputed_fake_quant)
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBnReLU2d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBnReLU2d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 718-749 / 第 718-749 行
```python
class ConvReLU2d(nnqat.Conv2d, nni._FusedModule):
    r"""A ConvReLU2d module is a fused module of Conv2d and ReLU, attached with
    FakeQuantize modules for weight for
    quantization aware training.

    We combined the interface of :class:`~torch.nn.Conv2d` and
    :class:`~torch.nn.BatchNorm2d`.

    Attributes:
        weight_fake_quant: fake quant module for weight

    """

    _FLOAT_MODULE: ClassVar[type[nn.Module]] = nni.ConvReLU2d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv2d]] = nn.Conv2d
    _FLOAT_BN_MODULE: ClassVar[type[nn.Module] | None] = None
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = nn.ReLU

    def __init__(
        self,
        in_channels,
        out_channels,
        kernel_size,
        stride=1,
        padding=0,
        dilation=1,
        groups=1,
        bias=True,
        padding_mode="zeros",
        qconfig=None,
    ):
        super().__init__(
```
- **EN**: It introduces or extends class-level abstractions such as `ConvReLU2d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvReLU2d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 750-778 / 第 750-778 行
```python
            in_channels,
            out_channels,
            kernel_size,
            stride=stride,
            padding=padding,
            dilation=dilation,
            groups=groups,
            bias=bias,
            # pyrefly: ignore [bad-argument-type]
            padding_mode=padding_mode,
            qconfig=qconfig,
        )
        if not qconfig:
            raise AssertionError("qconfig must be provided for QAT module")
        self.qconfig = qconfig
        self.weight_fake_quant = self.qconfig.weight()

    def forward(self, input):
        r"""Performs forward pass through fused Conv2d and ReLU."""
        return F.relu(
            self._conv_forward(input, self.weight_fake_quant(self.weight), self.bias)
        )

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):  # type: ignore[override]
        r"""Creates a QAT module from a floating point module."""
        return super().from_float(
            mod, use_precomputed_fake_quant=use_precomputed_fake_quant
        )
```
- **EN**: It introduces or extends class-level abstractions such as `ConvReLU2d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvReLU2d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 781-802 / 第 781-802 行
```python
class ConvBn3d(_ConvBnNd, nn.Conv3d):
    r"""
    A ConvBn3d module is a module fused from Conv3d and BatchNorm3d,
    attached with FakeQuantize modules for weight,
    used in quantization aware training.

    We combined the interface of :class:`torch.nn.Conv3d` and
    :class:`torch.nn.BatchNorm3d`.

    Similar to :class:`torch.nn.Conv3d`, with FakeQuantize modules initialized
    to default.

    Attributes:
        freeze_bn:
        weight_fake_quant: fake quant module for weight

    """

    _FLOAT_MODULE: ClassVar[type[nni.ConvBn3d]] = nni.ConvBn3d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv3d]] = nn.Conv3d
    _FLOAT_BN_MODULE: ClassVar[type[nn.Module] | None] = nn.BatchNorm3d
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = None
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn3d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBn3d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 804-835 / 第 804-835 行
```python
    def __init__(
        self,
        # ConvNd args
        in_channels,
        out_channels,
        kernel_size,
        stride=1,
        padding=0,
        dilation=1,
        groups=1,
        bias=None,
        padding_mode="zeros",
        # BatchNorm3d args
        # num_features: out_channels
        eps=1e-05,
        momentum=0.1,
        # affine: True
        # track_running_stats: True
        # Args for this module
        freeze_bn=False,
        qconfig=None,
    ):
        kernel_size = _triple(kernel_size)
        stride = _triple(stride)
        padding = _triple(padding)
        dilation = _triple(dilation)
        _ConvBnNd.__init__(
            self,
            in_channels,
            out_channels,
            kernel_size,
            stride,
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn3d`, which organize state and behavior for this subsystem. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `ConvBn3d` 等类级抽象，用于组织该子系统的状态与行为。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 836-866 / 第 836-866 行
```python
            padding,
            dilation,
            False,
            _triple(0),
            groups,
            bias,
            padding_mode,
            eps,
            momentum,
            freeze_bn,
            qconfig,
            dim=3,
        )


class ConvBnReLU3d(ConvBn3d):
    r"""
    A ConvBnReLU3d module is a module fused from Conv3d, BatchNorm3d and ReLU,
    attached with FakeQuantize modules for weight,
    used in quantization aware training.

    We combined the interface of :class:`torch.nn.Conv3d` and
    :class:`torch.nn.BatchNorm3d` and :class:`torch.nn.ReLU`.

    Similar to `torch.nn.Conv3d`, with FakeQuantize modules initialized to
    default.

    Attributes:
        weight_fake_quant: fake quant module for weight

    """
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBn3d`, `ConvBnReLU3d`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBn3d`, `ConvBnReLU3d` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 868-898 / 第 868-898 行
```python
    _FLOAT_MODULE: ClassVar[type[nni.ConvBnReLU3d]] = nni.ConvBnReLU3d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv3d]] = nn.Conv3d
    _FLOAT_BN_MODULE: ClassVar[type[nn.BatchNorm3d]] = nn.BatchNorm3d
    _FLOAT_RELU_MODULE: ClassVar[type[nn.ReLU] | None] = nn.ReLU
    # module class after fusing bn into conv
    _FUSED_FLOAT_MODULE: ClassVar[type[nni.ConvReLU3d] | None] = nni.ConvReLU3d

    def forward(self, input):
        r"""Performs forward pass through fused Conv3d, BatchNorm3d, and ReLU."""
        return F.relu(ConvBn3d._forward(self, input))

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):
        r"""Creates a QAT module from a floating point module."""
        return super().from_float(
            mod, use_precomputed_fake_quant=use_precomputed_fake_quant
        )


class ConvReLU3d(nnqat.Conv3d, nni._FusedModule):
    r"""A ConvReLU3d module is a fused module of Conv3d and ReLU, attached with
    FakeQuantize modules for weight for
    quantization aware training.

    We combined the interface of :class:`~torch.nn.Conv3d` and
    :class:`~torch.nn.BatchNorm3d`.

    Attributes:
        weight_fake_quant: fake quant module for weight

    """
```
- **EN**: It introduces or extends class-level abstractions such as `ConvBnReLU3d`, `ConvReLU3d`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvBnReLU3d`, `ConvReLU3d` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 900-931 / 第 900-931 行
```python
    _FLOAT_MODULE: ClassVar[type[nni.ConvReLU3d]] = nni.ConvReLU3d  # type: ignore[assignment]
    _FLOAT_CONV_MODULE: ClassVar[type[nn.Conv3d]] = nn.Conv3d
    _FLOAT_BN_MODULE: ClassVar[type[nn.Module] | None] = None
    _FLOAT_RELU_MODULE: ClassVar[type[nn.Module] | None] = nn.ReLU

    def __init__(
        self,
        in_channels,
        out_channels,
        kernel_size,
        stride=1,
        padding=0,
        dilation=1,
        groups=1,
        bias=True,
        padding_mode="zeros",
        qconfig=None,
    ):
        super().__init__(
            in_channels,
            out_channels,
            kernel_size,
            stride=stride,
            padding=padding,
            dilation=dilation,
            groups=groups,
            bias=bias,
            # pyrefly: ignore [bad-argument-type]
            padding_mode=padding_mode,
            qconfig=qconfig,
        )
        if not qconfig:
```
- **EN**: It introduces or extends class-level abstractions such as `ConvReLU3d`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `ConvReLU3d` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 932-959 / 第 932-959 行
```python
            raise AssertionError("qconfig must be provided for QAT module")
        self.qconfig = qconfig
        self.weight_fake_quant = self.qconfig.weight()

    def forward(self, input):
        r"""Performs forward pass through fused Conv3d and ReLU."""
        return F.relu(
            self._conv_forward(input, self.weight_fake_quant(self.weight), self.bias)
        )

    @classmethod
    def from_float(cls, mod, use_precomputed_fake_quant=False):  # type: ignore[override]
        r"""Creates a QAT module from a floating point module."""
        return super().from_float(
            mod, use_precomputed_fake_quant=use_precomputed_fake_quant
        )


def update_bn_stats(mod):
    if type(mod) in {
        ConvBnReLU1d,
        ConvBnReLU2d,
        ConvBnReLU3d,
        ConvBn1d,
        ConvBn2d,
        ConvBn3d,
    }:
        mod.update_bn_stats()
```
- **EN**: It introduces or extends class-level abstractions such as `ConvReLU3d`, which organize state and behavior for this subsystem. Key callable entry points in this range include `update_bn_stats`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `ConvReLU3d` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `update_bn_stats`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 962-971 / 第 962-971 行
```python
def freeze_bn_stats(mod):
    if type(mod) in {
        ConvBnReLU1d,
        ConvBnReLU2d,
        ConvBnReLU3d,
        ConvBn1d,
        ConvBn2d,
        ConvBn3d,
    }:
        mod.freeze_bn_stats()
```
- **EN**: Key callable entry points in this range include `freeze_bn_stats`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 这一段的重要可调用入口包括 `freeze_bn_stats`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants.
  - CN: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic`, `torch.ao.nn.qat`, `torch.nn`, `torch.nn.functional`, `torch.nn:init`, `torch.nn.modules.utils:_pair`, `torch.nn.modules.utils:_single`, `torch.nn.modules.utils:_triple`, `torch.nn.parameter:Parameter`, `torch.nn.utils:fuse_conv_bn_weights`
- **Python standard library / Python 标准库**: `math`, `typing:ClassVar`
- **Explicit exports / 显式导出**: `ConvBn1d`, `ConvBnReLU1d`, `ConvReLU1d`, `ConvBn2d`, `ConvBnReLU2d`, `ConvReLU2d`, `ConvBn3d`, `ConvBnReLU3d`, `ConvReLU3d`, `update_bn_stats`, `freeze_bn_stats`
- **Primary symbols / 核心符号**: `_ConvBnNd`, `ConvBn1d`, `ConvBnReLU1d`, `ConvReLU1d`, `ConvBn2d`, `ConvBnReLU2d`, `ConvReLU2d`, `ConvBn3d`, `ConvBnReLU3d`, `ConvReLU3d`, `update_bn_stats`, `freeze_bn_stats`

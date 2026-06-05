# custom_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/custom_op.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines the registration, out-of-tree override, and backend-dispatch framework for vLLM custom operators and pluggable layers. / [CN] 定义 vLLM 自定义算子与可插拔层的注册、树外替换以及按后端分发执行的基础框架。

## Line-by-Line Analysis / 逐行分析

### Registry setup
```python
logger = init_logger(__name__)

# Dictionary of all custom ops (classes, indexed by registered name).
op_registry: dict[str, type["CustomOp"] | type["PluggableLayer"]] = {}
op_registry_oot: dict[str, type["CustomOp"] | type["PluggableLayer"]] = {}


def maybe_get_oot_by_class(class_type: type) -> type:
    class_name = class_type.__name__
    if class_name in op_registry_oot:
        return op_registry_oot[class_name]
    return class_type
```
**EN:** The module keeps two registries: `op_registry` for in-tree named implementations and `op_registry_oot` for out-of-tree replacements. `maybe_get_oot_by_class` lets callers swap a built-in class for a registered external override by class name, which is how extension backends can replace stock implementations without editing core code.
**CN:** 这里维护两个注册表：`op_registry` 保存仓库内的命名实现，`op_registry_oot` 保存树外替换实现。`maybe_get_oot_by_class` 按类名检查是否存在外部替代类，从而让扩展后端无需修改核心代码就能替换内置实现。

### `PluggableLayer`: constructor-time class replacement
```python
class PluggableLayer(nn.Module):
    def __new__(cls, *args, **kwargs):
        try:
            layer_class_name = cls.__name__
        except AttributeError:
            raise TypeError(
                f"Cannot instantiate '{cls.__name__}': its 'name' attribute "
                f"was not set, possibly because it was not decorated with "
                f"@PluggableLayer.register, or it's the PluggableLayer itself."
            ) from None

        if layer_class_name not in op_registry_oot:
            layer_cls_to_instantiate = cls
        else:
            layer_cls_to_instantiate = op_registry_oot[layer_class_name]
            logger.debug(
                "Instantiating pluggable layer: %s using %s",
                layer_class_name,
                str(layer_cls_to_instantiate),
            )
        return super().__new__(layer_cls_to_instantiate)
```
**EN:** `PluggableLayer` changes instantiation behavior, not just method dispatch. If an OOT layer is registered under the same class name, `__new__` constructs that replacement class directly. This matters for layers whose customization changes submodule composition, initialization, or parameter structure rather than a single forward kernel.
**CN:** `PluggableLayer` 改变的是“实例化阶段”的行为，而不只是前向调用。若同名类存在树外实现，`__new__` 会直接实例化替代类。这适用于那种会改变子模块组合、初始化逻辑或参数结构的层，而不仅仅是替换一个前向 kernel。

### `PluggableLayer` decorators
```python
@classmethod
def register(cls, name: str):
    def decorator(op_cls):
        assert name not in op_registry, f"Duplicate op name: {name}"
        op_cls.name = name
        op_registry[name] = op_cls
        return op_cls

    return decorator

@classmethod
def register_oot(cls, _decorated_layer_cls=None, name: str | None = None):
    def decorator(layer_cls):
        reg_name = name if name is not None else cls.__name__
        assert reg_name not in op_registry_oot, f"Duplicate layer name: {reg_name}"
        layer_cls.name = reg_name
        op_registry_oot[reg_name] = layer_cls
        return layer_cls
```
**EN:** `register` records the canonical in-tree layer name, while `register_oot` records override classes for later constructor substitution. `register_oot` supports both decorator forms—with or without parentheses—so backend-specific packages can register replacements ergonomically.
**CN:** `register` 用于登记标准的仓库内层名，`register_oot` 用于登记稍后会在构造阶段被替换进去的外部类。`register_oot` 同时支持带括号和不带括号两种装饰器写法，方便不同后端扩展包接入。

### `CustomOp`: instance creation and forward binding
```python
class CustomOp(nn.Module):
    def __new__(cls, *args, **kwargs):
        try:
            op_name = cls.__name__
        except AttributeError:
            raise TypeError(
                f"Cannot instantiate '{cls.__name__}': its 'name' attribute "
                f"was not set, possibly because it was not decorated with "
                f"@CustomOp.register, or it's the CustomOp base class itself."
            ) from None

        if op_name not in op_registry_oot:
            op_cls_to_instantiate = cls
        else:
            op_cls_to_instantiate = op_registry_oot[op_name]
            logger.debug(
                "Instantiating custom op: %s using %s",
                op_name,
                str(op_cls_to_instantiate),
            )
        return super().__new__(op_cls_to_instantiate)

    def __init__(self, *, enforce_enable: bool = False, compile_native: bool = False):
        super().__init__()
        self._enforce_enable = enforce_enable
        self._forward_method = self.dispatch_forward(compile_native=compile_native)

    def forward(self, *args, **kwargs):
        return self._forward_method(*args, **kwargs)
```
**EN:** `CustomOp` uses the same OOT replacement idea, but once an object is created it binds `self._forward_method` to the chosen implementation. That turns `forward()` into a thin trampoline and avoids checking platform conditions on every invocation.
**CN:** `CustomOp` 也支持树外替换；但对象创建后，会把最终实现绑定到 `self._forward_method`。这样 `forward()` 只需做一次轻量跳转，避免每次调用都重复判断平台分支。

### Backend-specific forward fallbacks
```python
def forward_native(self, *args, **kwargs):
    raise NotImplementedError

def forward_cuda(self, *args, **kwargs):
    raise NotImplementedError

def forward_hip(self, *args, **kwargs):
    return self.forward_cuda(*args, **kwargs)

def forward_xpu(self, *args, **kwargs):
    return self.forward_native(*args, **kwargs)

def forward_cpu(self, *args, **kwargs):
    return self.forward_native(*args, **kwargs)

def forward_tpu(self, *args, **kwargs):
    return self.forward_native(*args, **kwargs)

def forward_oot(self, *args, **kwargs):
    return self.forward_native(*args, **kwargs)
```
**EN:** The base class defines a platform contract: subclasses may implement optimized CUDA/HIP/XPU variants, but the safe fallback is usually `forward_native`. HIP defaults to CUDA because ROCm kernels are often source-compatible; CPU/TPU/XPU default to the native PyTorch path.
**CN:** 基类定义了平台接口约定：子类可以提供 CUDA/HIP/XPU 等优化实现，而常规兜底通常是 `forward_native`。HIP 默认走 CUDA，是因为许多 ROCm kernel 与 CUDA 源级兼容；CPU/TPU/XPU 则默认回退到原生 PyTorch 实现。

### Enabling logic and backend dispatch
```python
def dispatch_forward(self, compile_native: bool):
    compilation_config = get_cached_compilation_config()
    enabled = self._enforce_enable or self.enabled()
    if enabled:
        compilation_config.enabled_custom_ops.update([self.__class__.name])
    else:
        compilation_config.disabled_custom_ops.update([self.__class__.name])

    if not enabled:
        return self.maybe_compile(self.forward_native, enable=compile_native)

    if current_platform.is_rocm():
        return self.forward_hip
    elif current_platform.is_cpu():
        return self.forward_cpu
    elif current_platform.is_tpu():
        return self.forward_tpu
    elif current_platform.is_xpu():
        return self.forward_xpu
    elif current_platform.is_out_of_tree():
        return self.forward_oot
    else:
        return self.forward_cuda
```
**EN:** Dispatch is driven by the cached compilation config plus the runtime platform abstraction. The method also records the final enabled/disabled decision into `compilation_config`, which gives later compilation stages visibility into which custom ops were active. If disabled, vLLM falls back to the native path and may still compile that path for graph capture.
**CN:** 分发逻辑由缓存的编译配置和运行时平台抽象共同决定。此方法还会把最终的启用/禁用结果回写到 `compilation_config`，让后续编译阶段知道哪些自定义算子实际生效。若算子被禁用，则回退到原生实现，并可按需对该原生路径做编译。

### `maybe_compile`: wrapping native paths with `torch.compile`
```python
def maybe_compile(self, fn, *, enable: bool = True):
    from vllm.config.compilation import CompilationMode

    if not enable:
        return fn

    compilation_config = get_cached_compilation_config()
    if compilation_config.mode == CompilationMode.NONE:
        return fn
    if compilation_config.backend == "eager":
        return fn

    compile_options = maybe_disable_graph_partition(
        current_platform.simple_compile_backend
    )
    backend = current_platform.simple_compile_backend

    dynamic_arg_dims = getattr(self.__class__, "_dynamic_arg_dims", None)
    if dynamic_arg_dims is not None:
        compiled_fn = torch.compile(
            fn,
            dynamic=False,
            backend=backend,
            options=compile_options,
        )
```
**EN:** This helper exists for native implementations that would otherwise sit inside opaque custom ops and escape model-level compilation. It refuses to compile when compilation is globally off or when the backend is eager. Otherwise it uses the platform’s simplified compile backend and can alter options through `maybe_disable_graph_partition`.
**CN:** 这个辅助函数用于处理“原生实现被包在不透明自定义算子内部”的场景，否则它们可能绕过模型级编译。若全局编译关闭或后端是 eager，则直接返回原函数；否则使用当前平台指定的简化编译后端，并通过 `maybe_disable_graph_partition` 调整选项。

### Dynamic argument marking for shape-sensitive compilation
```python
        sig = inspect.signature(fn)

        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            for name, dims in dynamic_arg_dims.items():
                arg = bound.arguments.get(name)
                if arg is not None and isinstance(arg, torch.Tensor):
                    dims_list = [dims] if isinstance(dims, int) else dims
                    for d in dims_list:
                        real_d = arg.ndim + d if d < 0 else d
                        torch._dynamo.mark_dynamic(arg, real_d)
            return compiled_fn(*args, **kwargs)

        return wrapper

    return torch.compile(
        fn,
        dynamic=True,
        backend=backend,
        options=compile_options,
    )
```
**EN:** If a custom op class registers `_dynamic_arg_dims`, vLLM compiles with `dynamic=False` but manually marks selected tensor dimensions as dynamic using TorchDynamo. That gives more control than blanket dynamic compilation and supports negative indices for counting dimensions from the end.
**CN:** 如果算子类注册了 `_dynamic_arg_dims`，vLLM 会先用 `dynamic=False` 编译，再通过 TorchDynamo 对指定张量维度手动打上动态标记。这样比完全开启动态编译更可控，并支持用负索引从末尾定位维度。

### Policy for default enablement
```python
@classmethod
def enabled(cls) -> bool:
    compilation_config = get_cached_compilation_config()
    custom_ops = compilation_config.custom_ops
    if not hasattr(cls, "name"):
        logger.warning_once(
            "Custom op %s was not registered, which means it won't appear "
            "in the op registry. It will be enabled/disabled based on the "
            "global settings.",
            cls.__name__,
        )
        return CustomOp.default_on()

    enabled = f"+{cls.name}" in custom_ops
    disabled = f"-{cls.name}" in custom_ops
    assert not (enabled and disabled), f"Cannot enable and disable {cls.name}"

    return (CustomOp.default_on() or enabled) and not disabled
```
**EN:** The enable/disable policy combines global defaults (`all` vs `none`) with per-op overrides (`+name` / `-name`). Unregistered classes still work, but they cannot be tracked in the registry and only follow the global switch.
**CN:** 启用策略把全局默认值（`all` 或 `none`）与单算子覆盖规则（`+name` / `-name`）结合起来。未注册类仍然可以运行，但不会出现在注册表中，因此只能跟随全局开关。

### Registration decorators for custom ops
```python
@classmethod
def register(
    cls,
    name: str,
    dynamic_arg_dims: dict[str, int | list[int]] | None = None,
):
    def decorator(op_cls):
        assert name not in op_registry, f"Duplicate op name: {name}"
        op_cls.name = name
        op_cls._dynamic_arg_dims = dynamic_arg_dims
        op_registry[name] = op_cls
        return op_cls

    return decorator
```
**EN:** `CustomOp.register` stores both the public op name and optional dynamic-dimension metadata. That metadata is later consumed by `maybe_compile`, so registration is not just bookkeeping; it carries compile-time behavior.
**CN:** `CustomOp.register` 不仅登记公开算子名，还可记录动态维度元数据。后者会被 `maybe_compile` 读取，因此注册动作不只是记账，也会影响编译行为。

### Out-of-tree custom-op overrides
```python
@classmethod
def register_oot(cls, _decorated_op_cls=None, name: str | None = None):
    def decorator(op_cls):
        reg_name = name if name is not None else cls.__name__
        assert reg_name not in op_registry_oot, f"Duplicate op name: {reg_name}"
        op_cls.name = reg_name
        op_registry_oot[reg_name] = op_cls
        return op_cls

    if _decorated_op_cls is None:
        return decorator
    elif isinstance(_decorated_op_cls, type):
        return decorator(_decorated_op_cls)
    else:
        raise TypeError("Decorator can only be applied to classes.")
```
**EN:** OOT registration mirrors the layer version: extension packages can inject alternative classes under the same logical op name. Because this is decorator-based and import-time driven, simply importing a backend extension can change which implementation gets instantiated later.
**CN:** 树外注册与层的逻辑一致：扩展包可以在相同逻辑算子名下插入替代类。由于它是基于装饰器、在 import 时生效的机制，因此只要导入对应扩展，就可能改变后续实例化得到的实现。

## Key Concepts / 关键概念
- **CustomOp vs PluggableLayer**: EN: `CustomOp` swaps the execution method; `PluggableLayer` swaps the whole class at construction time. CN: `CustomOp` 主要替换执行方法；`PluggableLayer` 则在构造阶段替换整个类。
- **OOT override**: EN: External backends can register replacements without patching core source. CN: 外部后端可在不修改核心源码的前提下注册替代实现。
- **Compilation-aware dispatch**: EN: Op enablement and `torch.compile` decisions are tied to `CompilationConfig`. CN: 算子启用与 `torch.compile` 决策都与 `CompilationConfig` 紧密绑定。
- **Dynamic shape control**: EN: `_dynamic_arg_dims` provides fine-grained dynamic-dimension marking for compiled native fallbacks. CN: `_dynamic_arg_dims` 为已编译的原生回退路径提供细粒度动态维度标记能力。

## Dependencies / 依赖关系
- **`vllm.config.get_cached_compilation_config`**: EN: Supplies the cached compilation policy and mutable enabled/disabled op sets. CN: 提供缓存的编译策略以及可更新的启用/禁用算子集合。
- **`vllm.model_executor.utils.maybe_disable_graph_partition`**: EN: Tweaks compile options for specific Torch/Inductor combinations. CN: 针对特定 Torch/Inductor 组合调整编译选项。
- **`vllm.platforms.current_platform`**: EN: Abstracts ROCm/CPU/TPU/XPU/OOT backend detection and compile backend choice. CN: 抽象 ROCm/CPU/TPU/XPU/树外平台判断以及编译后端选择。
- **PyTorch (`torch`, `torch.nn`, `torch._dynamo`)**: EN: Provides module base classes, `torch.compile`, and dynamic-shape annotations. CN: 提供模块基类、`torch.compile` 以及动态形状标注能力。

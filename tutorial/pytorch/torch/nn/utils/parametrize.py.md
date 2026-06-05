# parametrize.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/parametrize.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40
```python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs
import collections
import copyreg
from collections.abc import Sequence
from contextlib import contextmanager
from copy import deepcopy

import torch
from torch import Tensor
from torch.__future__ import get_swap_module_params_on_conversion
from torch._library.opaque_object import is_opaque_reference_type
from torch._opaque_base import OpaqueBase
from torch.nn.modules.container import Module, ModuleDict, ModuleList
from torch.nn.parameter import Parameter
from torch.utils._python_dispatch import is_traceable_wrapper_subclass


__all__ = [
    "cached",
    "ParametrizationList",
    "register_parametrization",
    "is_parametrized",
    "remove_parametrizations",
    "type_before_parametrizations",
    "transfer_parametrizations_and_params",
]

_cache_enabled = 0
_cache: dict[tuple[int, str], Tensor | None] = {}


@contextmanager
def cached():
    r"""Context manager that enables the caching system within parametrizations registered with :func:`register_parametrization`.

    The value of the parametrized objects is computed and cached the first time
    they are required when this context manager is active. The cached values are
    discarded when leaving the context manager.
```
- **EN**: These decorators register or transform the following definition so it can trace Python execution into an intermediate graph representation.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将 Python 执行过程跟踪为中间图表示。

### Lines 41-82
```python
    This is useful when using a parametrized parameter more than once in the forward pass.
    An example of this is when parametrizing the recurrent kernel of an RNN or when
    sharing weights.

    The simplest way to activate the cache is by wrapping the forward pass of the neural network

    .. code-block:: python

        import torch.nn.utils.parametrize as P

        ...
        with P.cached():
            output = model(inputs)

    in training and evaluation. One may also wrap the parts of the modules that use
    several times the parametrized tensors. For example, the loop of an RNN with a
    parametrized recurrent kernel:

    .. code-block:: python

        with P.cached():
            for x in xs:
                out_rnn = self.rnn_cell(x, out_rnn)
    """
    global _cache
    global _cache_enabled
    _cache_enabled += 1
    try:
        yield
    finally:
        _cache_enabled -= 1
        if not _cache_enabled:
            _cache = {}


def _register_parameter_or_buffer(module, name, X) -> None:
    if isinstance(X, Parameter):
        module.register_parameter(name, X)
    else:
        module.register_buffer(name, X)
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 83-122
```python
def _maybe_set(dest: Tensor, src: Tensor) -> None:
    should_swap = (
        get_swap_module_params_on_conversion() or is_traceable_wrapper_subclass(dest)
    )
    if should_swap:
        if isinstance(dest, Parameter) and not isinstance(src, Parameter):
            src = Parameter(src, requires_grad=dest.requires_grad)
        torch.utils.swap_tensors(dest, src)
    else:
        dest.set_(src)  # type: ignore[call-overload]


class ParametrizationList(ModuleList):
    r"""A sequential container that holds and manages the original parameters or buffers of a parametrized :class:`torch.nn.Module`.

    It is the type of ``module.parametrizations[tensor_name]`` when ``module[tensor_name]``
    has been parametrized with :func:`register_parametrization`.

    If the first registered parametrization has a ``right_inverse`` that returns one tensor or
    does not have a ``right_inverse`` (in which case we assume that ``right_inverse`` is the identity),
    it will hold the tensor under the name ``original``.
    If it has a ``right_inverse`` that returns more than one tensor, these will be registered as
    ``original0``, ``original1``, ...

    .. warning::
        This class is used internally by :func:`register_parametrization`. It is documented
        here for completeness. It shall not be instantiated by the user.

    Args:
        modules (sequence): sequence of modules representing the parametrizations
        original (Parameter or Tensor): parameter or buffer that is parametrized
        unsafe (bool): a boolean flag that denotes whether the parametrization
            may change the dtype and shape of the tensor. Default: `False`
            Warning: the parametrization is not checked for consistency upon registration.
            Enable this flag at your own risk.
    """

    original: Tensor
    unsafe: bool
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 123-159
```python
    def __init__(
        self,
        modules: Sequence[Module],
        original: Tensor | Parameter,
        unsafe: bool = False,
    ) -> None:
        # We require this because we need to treat differently the first parametrization
        # This should never throw, unless this class is used from the outside
        if len(modules) == 0:
            raise ValueError("ParametrizationList requires one or more modules.")

        super().__init__(modules)
        self.unsafe = unsafe

        # In plain words:
        # module.weight must keep its dtype and shape.
        # Furthermore, if there is no right_inverse or the right_inverse returns a tensor,
        # this should be of the same dtype as the original tensor
        #
        # We check that the following invariants hold:
        #    X = module.weight
        #    Y = param.right_inverse(X)
        #    assert isinstance(Y, Tensor) or
        #           (isinstance(Y, collections.abc.Sequence) and all(isinstance(t, Tensor) for t in Y))
        #    Z = param(Y) if isinstance(Y, Tensor) else param(*Y)
        #    # Consistency checks
        #    assert X.dtype == Z.dtype and X.shape == Z.shape
        #    # If it has one input, this allows to be able to use set_ to be able to
        #    # move data to/from the original tensor without changing its id (which is what the
        #    # optimizer uses to track parameters)
        #    if isinstance(Y, Tensor)
        #      assert X.dtype == Y.dtype
        # Below we use original = X, new = Y

        original_shape = original.shape
        original_dtype = original.dtype
```
- **EN**: Defines the `ParametrizationList.__init__` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`ParametrizationList.__init__` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 160-200
```python
        # Compute new
        with torch.no_grad():
            new = original
            for module in reversed(self):  # type: ignore[call-overload]
                if hasattr(module, "right_inverse"):
                    try:
                        new = module.right_inverse(new)  # type: ignore[operator]
                    except NotImplementedError:
                        pass
                # else, or if it throws, we assume that right_inverse is the identity

        if not isinstance(new, Tensor) and not isinstance(new, Sequence):
            raise ValueError(
                "'right_inverse' must return a Tensor or a Sequence of tensors (list, tuple...). "
                f"Got {type(new).__name__}"
            )

        # Set the number of original tensors
        self.is_tensor = isinstance(new, Tensor)
        self.ntensors = 1 if self.is_tensor else len(new)

        # Register the tensor(s)
        if self.is_tensor:
            # pyrefly: ignore [missing-attribute]
            if original.dtype != new.dtype:
                raise ValueError(
                    "When `right_inverse` outputs one tensor, it may not change the dtype.\n"
                    f"original.dtype: {original.dtype}\n"
                    # pyrefly: ignore [missing-attribute]
                    f"right_inverse(original).dtype: {new.dtype}"
                )

            # pyrefly: ignore [missing-attribute]
            if original.device != new.device:
                raise ValueError(
                    "When `right_inverse` outputs one tensor, it may not change the device.\n"
                    f"original.device: {original.device}\n"
                    # pyrefly: ignore [missing-attribute]
                    f"right_inverse(original).device: {new.device}"
                )
```
- **EN**: This block continues `ParametrizationList.__init__` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `ParametrizationList.__init__`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 201-232
```python
            # Set the original to original so that the user does not need to re-register the parameter
            # manually in the optimiser
            with torch.no_grad():
                # pyrefly: ignore [bad-argument-type]
                _maybe_set(original, new)
            _register_parameter_or_buffer(self, "original", original)
        else:
            for i, originali in enumerate(new):
                match originali:
                    case OpaqueBase():
                        if not is_opaque_reference_type(type(originali)):
                            raise ValueError(
                                f"'right_inverse' must return a Tensor or a reference-type "
                                f"opaque. Got element {i} of the sequence with type "
                                f"{type(originali).__name__}."
                            )
                        setattr(self, f"original{i}", originali)
                    case Tensor():
                        # If the original tensor was a Parameter that required grad, we expect the user to
                        # add the new parameters to the optimizer after registering the parametrization
                        # (this is documented)
                        if isinstance(original, Parameter):
                            originali = Parameter(originali, original.requires_grad)
                        originali.requires_grad_(original.requires_grad)
                        _register_parameter_or_buffer(self, f"original{i}", originali)
                    case _:
                        raise ValueError(
                            "'right_inverse' must return a Tensor or a Sequence of tensors "
                            "(list, tuple...). "
                            f"Got element {i} of the sequence with type {type(originali).__name__}."
                        )
```
- **EN**: This block continues `ParametrizationList.__init__` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `ParametrizationList.__init__`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 233-268
```python
        if not self.unsafe:
            # Consistency checks:
            # Since f : A -> B, right_inverse : B -> A, Z and original should live in B
            # Z = forward(right_inverse(original))
            Z = self()
            if not isinstance(Z, Tensor):
                raise ValueError(
                    f"A parametrization must return a tensor. Got {type(Z).__name__}."
                )
            if Z.dtype != original_dtype:
                raise ValueError(
                    "Registering a parametrization may not change the dtype of the tensor, unless `unsafe` flag is enabled.\n"
                    f"unparametrized dtype: {original_dtype}\n"
                    f"parametrized dtype: {Z.dtype}"
                )
            if Z.shape != original_shape:
                raise ValueError(
                    "Registering a parametrization may not change the shape of the tensor, unless `unsafe` flag is enabled.\n"
                    f"unparametrized shape: {original_shape}\n"
                    f"parametrized shape: {Z.shape}"
                )

    def right_inverse(self, value: Tensor) -> None:
        r"""Call the ``right_inverse`` methods of the parametrizations in the inverse registration order.

        Then, it stores the result in ``self.original`` if ``right_inverse`` outputs one tensor
        or in ``self.original0``, ``self.original1``, ... if it outputs several.

        Args:
            value (Tensor): Value to which initialize the module
        """
        # All the exceptions in this function should almost never throw.
        # They could throw if, for example, right_inverse function returns a different
        # dtype when given a different input, which should most likely be caused by a
        # bug in the user's code
```
- **EN**: Declares `ParametrizationList(ModuleList)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `ParametrizationList(ModuleList)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 269-296
```python
        with torch.no_grad():
            # See https://github.com/pytorch/pytorch/issues/53103
            for module in reversed(self):  # type: ignore[call-overload]
                if hasattr(module, "right_inverse"):
                    value = module.right_inverse(value)  # type: ignore[operator]
                else:
                    raise RuntimeError(
                        f"parametrization {type(module).__name__} does not implement "
                        "right_inverse."
                    )
            if self.is_tensor:
                # These exceptions should only throw when a right_inverse function does not
                # return the same dtype for every input, which should most likely be caused by a bug
                if not isinstance(value, Tensor):
                    raise ValueError(
                        f"`right_inverse` should return a tensor. Got {type(value).__name__}"
                    )
                if value.dtype != self.original.dtype:
                    raise ValueError(
                        f"The tensor returned by `right_inverse` has dtype {value.dtype} "
                        f"while `original` has dtype {self.original.dtype}"
                    )
                # We know that the result is going to have the same dtype
                _maybe_set(self.original, value)
            else:
                if not isinstance(value, collections.abc.Sequence):
                    raise ValueError(
                        "'right_inverse' must return a sequence of tensors. "
```
- **EN**: This block continues `ParametrizationList.right_inverse` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `ParametrizationList.right_inverse`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 297-325
```python
                        f"Got {type(value).__name__}."
                    )
                if len(value) != self.ntensors:
                    raise ValueError(
                        "'right_inverse' must return a sequence of tensors of length "
                        f"{self.ntensors}. Got a sequence of length {len(value)}."
                    )
                for i, tensor in enumerate(value):
                    original_i = getattr(self, f"original{i}")
                    match tensor:
                        case OpaqueBase():
                            if is_opaque_reference_type(type(tensor)):
                                setattr(self, f"original{i}", tensor)
                                continue
                            # Fall-through
                        case Tensor():
                            if original_i.dtype != tensor.dtype:
                                raise ValueError(
                                    f"Tensor {i} returned by `right_inverse` has dtype {tensor.dtype} "
                                    f"while `original{i}` has dtype {original_i.dtype}"
                                )
                            _maybe_set(original_i, tensor)
                            continue
                    raise ValueError(
                        f"'right_inverse' must return a sequence of tensors "
                        f"or reference-type opaques. Got element {i} of type "
                        f"{type(tensor).__name__}."
                    )
```
- **EN**: This block continues `ParametrizationList` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `ParametrizationList`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 326-354
```python
    def forward(self) -> Tensor:
        if torch.jit.is_scripting():
            raise RuntimeError("Parametrization is not working with scripting.")
        # Unpack the originals for the first parametrization
        if self.is_tensor:
            x = self[0](self.original)
        else:
            originals = (getattr(self, f"original{i}") for i in range(self.ntensors))
            x = self[0](*originals)
        # It's not possible to call self[1:] here, so we have to be a bit more cryptic
        # Also we want to skip all non-integer keys
        curr_idx = 1
        while hasattr(self, str(curr_idx)):
            x = self[curr_idx](x)
            curr_idx += 1
        return x


def _inject_new_class(module: Module) -> None:
    r"""Set up a module to be parametrized.

    This works by substituting the class of the module by a class
    that extends it to be able to inject a property

    Args:
        module (nn.Module): module into which to inject the property
    """
    cls = module.__class__
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 355-395
```python
    def default_deepcopy(self, memo):
        # Just emulate a standard deepcopy procedure when __deepcopy__ doesn't exist in the current class.
        obj = memo.get(id(self), None)
        if obj is not None:
            return obj
        replica = self.__new__(self.__class__)
        memo[id(self)] = replica
        replica.__dict__ = deepcopy(self.__dict__, memo)
        # Also save all slots if they exist.
        slots_to_save = copyreg._slotnames(self.__class__)  # type: ignore[attr-defined]
        for slot in slots_to_save:
            if hasattr(self, slot):
                setattr(replica, slot, deepcopy(getattr(self, slot), memo))
        return replica

    def getstate(self):
        raise RuntimeError(
            "Serialization of parametrized modules is only "
            "supported through state_dict(). See:\n"
            "https://pytorch.org/tutorials/beginner/saving_loading_models.html"
            "#saving-loading-a-general-checkpoint-for-inference-and-or-resuming-training"
        )

    dct = {"__getstate__": getstate}
    # We don't allow serialization of parametrized modules but should still allow deepcopying.
    # Default 'deepcopy' function invokes __deepcopy__ method instead of __getstate__ when it exists.
    if not hasattr(cls, "__deepcopy__"):
        dct["__deepcopy__"] = default_deepcopy  # type: ignore[assignment]

    param_cls = type(
        f"Parametrized{cls.__name__}",
        (cls,),
        dct,
    )

    module.__class__ = param_cls


def _inject_property(module: Module, tensor_name: str) -> None:
    r"""Injects a property into module[tensor_name].
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 396-423
```python
    It assumes that the class in the module has already been modified from its
    original one using _inject_new_class and that the tensor under :attr:`tensor_name`
    has already been moved out

    Args:
        module (nn.Module): module into which to inject the property
        tensor_name (str): name of the name of the property to create
    """
    # We check the precondition.
    # This should never fire if register_parametrization is correctly implemented
    if hasattr(module, tensor_name):
        raise AssertionError(f"Module already has an attribute named '{tensor_name}'")

    @torch.jit.unused
    def get_cached_parametrization(parametrization) -> Tensor:
        global _cache
        key = (id(module), tensor_name)
        tensor = _cache.get(key)
        if tensor is None:
            tensor = parametrization()
            _cache[key] = tensor
        return tensor

    def get_parametrized(self) -> Tensor:
        if torch.jit.is_scripting():
            raise RuntimeError("Parametrization is not working with scripting.")
        parametrization = self.parametrizations[tensor_name]
        # pyrefly: ignore [redundant-condition]
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 424-463
```python
        if _cache_enabled:
            if torch.jit.is_scripting():
                # Scripting
                raise RuntimeError(
                    "Caching is not implemented for scripting. "
                    "Either disable caching or avoid scripting."
                )
            elif torch._C._get_tracing_state() is not None:
                # Tracing
                raise RuntimeError(
                    "Cannot trace a model while caching parametrizations."
                )
            else:
                return get_cached_parametrization(parametrization)
        else:
            # If caching is not active, this function just evaluates the parametrization
            return parametrization()

    def set_original(self, value: Tensor) -> None:
        if torch.jit.is_scripting():
            raise RuntimeError("Parametrization is not working with scripting.")
        self.parametrizations[tensor_name].right_inverse(value)

    setattr(module.__class__, tensor_name, property(get_parametrized, set_original))


def register_parametrization(
    module: Module,
    tensor_name: str,
    parametrization: Module,
    *,
    unsafe: bool = False,
) -> Module:
    r"""Register a parametrization to a tensor in a module.

    Assume that ``tensor_name="weight"`` for simplicity. When accessing ``module.weight``,
    the module will return the parametrized version ``parametrization(module.weight)``.
    If the original tensor requires a gradient, the backward pass will differentiate
    through :attr:`parametrization`, and the optimizer will update the tensor accordingly.
```
- **EN**: This module-level block helps trace Python execution into an intermediate graph representation. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将 Python 执行过程跟踪为中间图表示。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 464-503
```python
    The first time that a module registers a parametrization, this function will add an attribute
    ``parametrizations`` to the module of type :class:`~ParametrizationList`.

    The list of parametrizations on the tensor ``weight`` will be accessible under
    ``module.parametrizations.weight``.

    The original tensor will be accessible under
    ``module.parametrizations.weight.original``.

    Parametrizations may be concatenated by registering several parametrizations
    on the same attribute.

    The training mode of a registered parametrization is updated on registration
    to match the training mode of the host module

    Parametrized parameters and buffers have an inbuilt caching system that can be activated
    using the context manager :func:`cached`.

    A :attr:`parametrization` may optionally implement a method with signature

    .. code-block:: python

        def right_inverse(self, X: Tensor) -> Union[Tensor, Sequence[Tensor]]

    This method is called on the unparametrized tensor when the first parametrization
    is registered to compute the initial value of the original tensor.
    If this method is not implemented, the original tensor will be just the unparametrized tensor.

    If all the parametrizations registered on a tensor implement `right_inverse` it is possible
    to initialize a parametrized tensor by assigning to it, as shown in the example below.

    It is possible for the first parametrization to depend on several inputs.
    This may be implemented returning a tuple of tensors from ``right_inverse``
    (see the example implementation of a ``RankOne`` parametrization below).

    In this case, the unconstrained tensors are also located under ``module.parametrizations.weight``
    with names ``original0``, ``original1``,...

    .. note::
```
- **EN**: Defines the `register_parametrization` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`register_parametrization` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 504-537
```python
        If unsafe=False (default) both the forward and right_inverse methods will be called
        once to perform a number of consistency checks.
        If unsafe=True, then right_inverse will be called if the tensor is not parametrized,
        and nothing will be called otherwise.

    .. note::

        In most situations, ``right_inverse`` will be a function such that
        ``forward(right_inverse(X)) == X`` (see
        `right inverse <https://en.wikipedia.org/wiki/Inverse_function#Right_inverses>`_).
        Sometimes, when the parametrization is not surjective, it may be reasonable
        to relax this.

    .. warning::

        If a parametrization depends on several inputs, :func:`~register_parametrization`
        will register a number of new parameters. If such parametrization is registered
        after the optimizer is created, these new parameters will need to be added manually
        to the optimizer. See :meth:`torch.Optimizer.add_param_group`.

    Args:
        module (nn.Module): module on which to register the parametrization
        tensor_name (str): name of the parameter or buffer on which to register
            the parametrization
        parametrization (nn.Module): the parametrization to register
    Keyword args:
        unsafe (bool): a boolean flag that denotes whether the parametrization
            may change the dtype and shape of the tensor. Default: `False`
            Warning: the parametrization is not checked for consistency upon registration.
            Enable this flag at your own risk.

    Raises:
        ValueError: if the module does not have a parameter or a buffer named :attr:`tensor_name`
```
- **EN**: This block continues `register_parametrization` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `register_parametrization`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 538-578
```python
    Examples:
        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_LAPACK)
        >>> import torch
        >>> import torch.nn as nn
        >>> import torch.nn.utils.parametrize as P
        >>>
        >>> class Symmetric(nn.Module):
        >>>     def forward(self, X):
        >>>         return X.triu() + X.triu(1).T  # Return a symmetric matrix
        >>>
        >>>     def right_inverse(self, A):
        >>>         return A.triu()
        >>>
        >>> m = nn.Linear(5, 5)
        >>> P.register_parametrization(m, "weight", Symmetric())
        >>> print(torch.allclose(m.weight, m.weight.T))  # m.weight is now symmetric
        True
        >>> A = torch.rand(5, 5)
        >>> A = A + A.T  # A is now symmetric
        >>> m.weight = A  # Initialize the weight to be the symmetric matrix A
        >>> print(torch.allclose(m.weight, A))
        True

        >>> class RankOne(nn.Module):
        >>>     def forward(self, x, y):
        >>> # Form a rank 1 matrix multiplying two vectors
        >>>         return x.unsqueeze(-1) @ y.unsqueeze(-2)
        >>>
        >>>     def right_inverse(self, Z):
        >>> # Project Z onto the rank 1 matrices
        >>>         U, S, Vh = torch.linalg.svd(Z, full_matrices=False)
        >>> # Return rescaled singular vectors
        >>>         s0_sqrt = S[0].sqrt().unsqueeze(-1)
        >>>         return U[..., :, 0] * s0_sqrt, Vh[..., 0, :] * s0_sqrt
        >>>
        >>> linear_rank_one = P.register_parametrization(
        ...     nn.Linear(4, 4), "weight", RankOne()
        ... )
        >>> print(torch.linalg.matrix_rank(linear_rank_one.weight).item())
        1
```
- **EN**: This block continues `register_parametrization` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `register_parametrization`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 579-606
```python
    """
    parametrization.train(module.training)
    if is_parametrized(module, tensor_name):
        # Correctness checks.
        # If A is the space of tensors with shape and dtype equal to module.weight
        # we check that parametrization.forward and parametrization.right_inverse are
        # functions from A to A
        if not unsafe:
            Y = getattr(module, tensor_name)
            X = parametrization(Y)
            if not isinstance(X, Tensor):
                raise ValueError(
                    f"A parametrization must return a tensor. Got {type(X).__name__}."
                )
            if X.dtype != Y.dtype:
                raise ValueError(
                    "Registering a parametrization may not change the dtype of the tensor, unless the `unsafe` flag is enabled.\n"
                    f"module.{tensor_name}.dtype: {Y.dtype}\n"
                    f"parametrization(module.{tensor_name}).dtype: {X.dtype}"
                )
            if X.shape != Y.shape:
                raise ValueError(
                    "Registering a parametrization may not change the shape of the tensor, unless the `unsafe` flag is enabled.\n"
                    f"module.{tensor_name}.shape: {Y.shape}\n"
                    f"parametrization(module.{tensor_name}).shape: {X.shape}"
                )
            if hasattr(parametrization, "right_inverse"):
                try:
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 607-634
```python
                    Z = parametrization.right_inverse(X)  # type: ignore[operator]
                except NotImplementedError:
                    pass
                else:
                    if not isinstance(Z, Tensor):
                        raise ValueError(
                            f"parametrization.right_inverse must return a tensor. Got: {type(Z).__name__}"
                        )
                    if Z.dtype != Y.dtype:
                        raise ValueError(
                            "The tensor returned by parametrization.right_inverse must have the same dtype "
                            f"as module.{tensor_name}, unless the `unsafe` flag is enabled.\n"
                            f"module.{tensor_name}.dtype: {Y.dtype}\n"
                            f"returned dtype: {Z.dtype}"
                        )
                    if Z.shape != Y.shape:
                        raise ValueError(
                            "The tensor returned by parametrization.right_inverse must have the same shape "
                            f"as module.{tensor_name}, unless the `unsafe` flag is enabled.\n"
                            f"module.{tensor_name}.shape: {Y.shape}\n"
                            f"returned shape: {Z.shape}"
                        )
            # else right_inverse is assumed to be the identity

        # add the new parametrization to the parametrization list
        if not isinstance(module.parametrizations, ModuleDict):
            raise AssertionError(
                f"Expected module.parametrizations to be a ModuleDict, "
```
- **EN**: This block continues `register_parametrization` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `register_parametrization`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 635-676
```python
                f"got {type(module.parametrizations).__name__}"
            )
        module.parametrizations[tensor_name].append(parametrization)  # type: ignore[operator]
        # If unsafe was True in previous parametrization, keep it enabled
        module.parametrizations[tensor_name].unsafe |= unsafe  # type: ignore[index, union-attr, operator]
    elif tensor_name in module._buffers or tensor_name in module._parameters:
        # Set the parametrization mechanism
        # Fetch the original buffer or parameter
        original = getattr(module, tensor_name)
        # We create this early to check for possible errors
        parametrizations = ParametrizationList(
            [parametrization], original, unsafe=unsafe
        )
        # Delete the previous parameter or buffer
        delattr(module, tensor_name)
        # If this is the first parametrization registered on the module,
        # we prepare the module to inject the property
        if not is_parametrized(module):
            # Change the class
            _inject_new_class(module)
            # Inject a ``ModuleDict`` into the instance under module.parametrizations
            module.parametrizations = ModuleDict()
        # Add a property into the class
        _inject_property(module, tensor_name)
        # Add a ParametrizationList
        if not isinstance(module.parametrizations, ModuleDict):
            raise AssertionError(
                f"Expected module.parametrizations to be a ModuleDict, "
                f"got {type(module.parametrizations).__name__}"
            )
        module.parametrizations[tensor_name] = parametrizations
    else:
        raise ValueError(
            f"Module '{module}' does not have a parameter, a buffer, or a "
            f"parametrized element with name '{tensor_name}'"
        )
    return module


def is_parametrized(module: Module, tensor_name: str | None = None) -> bool:
    r"""Determine if a module has a parametrization.
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 677-718
```python
    Args:
        module (nn.Module): module to query
        tensor_name (str, optional): name of the parameter in the module
            Default: ``None``
    Returns:
        ``True`` if :attr:`module` has a parametrization for the parameter named :attr:`tensor_name`,
        or if it has any parametrization when :attr:`tensor_name` is ``None``;
        otherwise ``False``
    """
    parametrizations = getattr(module, "parametrizations", None)
    if parametrizations is None or not isinstance(parametrizations, ModuleDict):
        return False
    if tensor_name is None:
        # Check that there is at least one parametrized buffer or Parameter
        return len(parametrizations) > 0
    else:
        return tensor_name in parametrizations


def remove_parametrizations(
    module: Module,
    tensor_name: str,
    leave_parametrized: bool = True,
) -> Module:
    r"""Remove the parametrizations on a tensor in a module.

    - If ``leave_parametrized=True``, ``module[tensor_name]`` will be set to
      its current output. In this case, the parametrization shall not change the ``dtype``
      of the tensor.
    - If ``leave_parametrized=False``, ``module[tensor_name]`` will be set to
      the unparametrised tensor in ``module.parametrizations[tensor_name].original``.
      This is only possible when the parametrization depends on just one tensor.

    Args:
        module (nn.Module): module from which remove the parametrization
        tensor_name (str): name of the parametrization to be removed
        leave_parametrized (bool, optional): leave the attribute :attr:`tensor_name` parametrized.
            Default: ``True``

    Returns:
        Module: module
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 719-746
```python
    Raises:
        ValueError: if ``module[tensor_name]`` is not parametrized
        ValueError: if ``leave_parametrized=False`` and the parametrization depends on several tensors
    """
    if not is_parametrized(module, tensor_name):
        raise ValueError(
            f"Module {module} does not have a parametrization on {tensor_name}"
        )

    # Fetch the original tensor
    if not isinstance(module.parametrizations, ModuleDict):
        raise AssertionError(
            f"Expected module.parametrizations to be a ModuleDict, "
            f"got {type(module.parametrizations).__name__}"
        )
    parametrizations = module.parametrizations[tensor_name]

    if parametrizations.is_tensor:
        original = parametrizations.original
        if not isinstance(original, torch.Tensor):
            raise AssertionError(
                f"Expected original to be a Tensor (is_tensor promised us a Tensor), "
                f"got {type(original).__name__}"
            )
        if leave_parametrized:
            with torch.no_grad():
                t = getattr(module, tensor_name)
            # We know they have the same dtype because we have checked this when registering the
```
- **EN**: This block continues `remove_parametrizations` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `remove_parametrizations`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 747-788
```python
            # parametrizations. As such, we can use set_
            # We do this so that the parameter does not to change the id()
            # This way the user does not need to update the optimizer
            with torch.no_grad():
                if type(original) is torch.Tensor:
                    _maybe_set(original, t)
                else:
                    try:
                        _maybe_set(original, t)
                    except RuntimeError as e:
                        # TODO: Fix this for tensor subclasses that are parameters:
                        # RuntimeError: set_storage is not allowed on a Tensor created from .data or .detach().
                        raise RuntimeError(
                            "Calling remove_parametrizations() with leave_parametrized=True "
                            "for a parameter that is an instance of a tensor subclass requires "
                            "set_() to be implemented correctly for the tensor subclass."
                            "Alternatively, one can opt into the swap_tensors path"
                            "Either set leave_parametrized=False or provide a working implementation"
                            "for set_() in the tensor subclass or set "
                            "torch.__future__.set_swap_module_params_on_conversion(True)."
                        ) from e
    else:
        if leave_parametrized:
            # We cannot use no_grad because we need to know whether one or more
            # original tensors required grad
            t = getattr(module, tensor_name)
            # We'll have to trust the user to add it to the optimizer
            original = Parameter(t) if t.requires_grad else t
        else:
            raise ValueError(
                "Cannot leave unparametrized (`leave_parametrized=False`) a tensor "
                "that is parametrized in terms of a sequence of tensors."
            )

    # Delete the property that manages the parametrization
    delattr(module.__class__, tensor_name)
    # Delete the ParametrizationList
    del module.parametrizations[tensor_name]

    # Restore the parameter / buffer into the main class
    _register_parameter_or_buffer(module, tensor_name, original)
```
- **EN**: This block continues `remove_parametrizations` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `remove_parametrizations`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 789-826
```python
    # Roll back the parametrized class if no other buffer or parameter
    # is currently parametrized in this class
    if not is_parametrized(module):
        delattr(module, "parametrizations")
        # Restore class
        orig_cls = module.__class__.__bases__[0]
        module.__class__ = orig_cls
    return module


def type_before_parametrizations(module: Module) -> type:
    r"""Return the module type before parametrizations were applied and if not, then it returns the module type.

    Args:
        module (nn.Module): module to get type of
    """
    if is_parametrized(module):
        return module.__class__.__bases__[0]
    else:
        return type(module)


def transfer_parametrizations_and_params(
    from_module: Module,
    to_module: Module,
    tensor_name: str | None = None,
) -> Module:
    r"""Transfer parametrizations and the parameters they parametrize from :attr:`from_module` to :attr:`to_module`.

    If :attr:`tensor_name` is specified, only transfers the specified parameter, otherwise
    transfers all parametrized parameters. If those parameters do not exist in to_module, it will create them.
    Does nothing if from_module is not parametrized.

    Args:
        from_module (nn.Module): module to transfer from
        to_module (nn.Module): module to transfer to
        tensor_name (str, optional): parameter to transfer
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 827-866
```python
    Returns:
        Module: to_module
    """
    if is_parametrized(from_module):
        if not isinstance(from_module.parametrizations, ModuleDict):
            raise AssertionError(
                f"Expected from_module.parametrizations to be a ModuleDict, "
                f"got {type(from_module.parametrizations).__name__}"
            )

        # get list of all params or the single param to transfer
        parameters_to_transfer: list | ModuleDict = (
            from_module.parametrizations if tensor_name is None else [tensor_name]
        )

        if not hasattr(parameters_to_transfer, "__iter__"):
            raise AssertionError(
                f"Expected parameters_to_transfer to be iterable, "
                f"got {type(parameters_to_transfer).__name__}"
            )
        for parameter_name in parameters_to_transfer:
            # initialize the to-be-transferred param in to_module if it doesn't exist already
            if not hasattr(to_module, parameter_name):
                setattr(
                    to_module,
                    parameter_name,
                    Parameter(getattr(from_module, parameter_name)),
                )

            # apply the params's parametrizations to to_module
            for param_func in from_module.parametrizations[  # type: ignore[attr-defined]
                parameter_name
            ]:
                register_parametrization(to_module, parameter_name, param_func)
            if not isinstance(to_module.parametrizations, ModuleDict):
                raise AssertionError(
                    f"Expected to_module.parametrizations to be a ModuleDict, "
                    f"got {type(to_module.parametrizations).__name__}"
                )
```
- **EN**: This block continues `transfer_parametrizations_and_params` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `transfer_parametrizations_and_params`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 867-886
```python
            # make values match, original values can be stored in either original or
            # original0, original1..., need to check both cases
            if hasattr(from_module.parametrizations[parameter_name], "original"):
                to_module.parametrizations[
                    parameter_name
                ].original = from_module.parametrizations[parameter_name].original
            else:
                num = 0
                orig_num = "original" + str(num)
                # loop through each original# until all values have been set
                while hasattr(from_module.parametrizations[parameter_name], orig_num):
                    setattr(
                        to_module.parametrizations[parameter_name],
                        orig_num,
                        getattr(from_module.parametrizations[parameter_name], orig_num),
                    )
                    num = num + 1
                    orig_num = "original" + str(num)

    return to_module
```
- **EN**: This block continues `transfer_parametrizations_and_params` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `transfer_parametrizations_and_params`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.__future__`, `torch._library.opaque_object`, `torch._opaque_base`, `torch.nn.modules.container`, `torch.nn.parameter`, `torch.utils._python_dispatch`
- **Standard library / 标准库**: `collections`, `copyreg`, `collections.abc`, `contextlib`, `copy`
- **Primary symbols / 核心符号**: `__all__`, `cached`, `_register_parameter_or_buffer`, `_maybe_set`, `ParametrizationList`, `_inject_new_class`, `_inject_property`, `register_parametrization`, `is_parametrized`, `remove_parametrizations`, `type_before_parametrizations`, `transfer_parametrizations_and_params`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。

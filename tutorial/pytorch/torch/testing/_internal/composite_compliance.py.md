# composite_compliance.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/composite_compliance.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for composite compliance, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 composite compliance 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
# mypy: ignore-errors

import torch
from torch import Tensor
import itertools

from torch.utils._python_dispatch import TorchDispatchMode
from torch.utils._pytree import tree_map, tree_flatten, tree_unflatten
from torch.utils import _pytree as pytree
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.utils._python_dispatch`, `torch.utils._pytree`, `torch.utils`; external imports: `itertools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.utils._python_dispatch`, `torch.utils._pytree`, `torch.utils`；外部导入：`itertools`。

### Lines 10-21
```python
from functools import partial
from torch.utils._mode_utils import no_dispatch, all_same_mode
import torch.autograd.forward_ad as fwAD
from collections.abc import Callable
import re


def check_attr_consistency(wrapper_tensor, metadata_name, metadata_accessor):
    elem = wrapper_tensor.elem
    metadata_wrapper_tensor = metadata_accessor(wrapper_tensor)
    metadata_elem = metadata_accessor(elem)
    if metadata_wrapper_tensor == metadata_elem:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `check_attr_consistency`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`check_attr_consistency`。

### Lines 22-30
```python
        return
    raise RuntimeError(
        f"This operator is not Composite Compliant: the "
        f"{metadata_name} of the tensor was modified directly without "
        f"going through the PyTorch dispatcher.")

def check_metadata_consistency(wrapper_tensor, CCT):
    # CCT: CompositeCompliantTensor class which is generated using generate_cct
    if not isinstance(wrapper_tensor, CCT):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `check_metadata_consistency`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`check_metadata_consistency`。

### Lines 31-40
```python
        return
    things_to_check = {
        'shape': Tensor.size,
        'dtype': lambda x: x.dtype,
        'device': lambda x: x.device,
        'numel': Tensor.numel,
        'stride': Tensor.stride,
        'storage_offset': Tensor.storage_offset,
    }
    for metadata_name, metadata_accessor in things_to_check.items():
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 41-58
```python
        check_attr_consistency(wrapper_tensor, metadata_name, metadata_accessor)

def is_view_fn(func):
    return func.overloadpacket.__name__ in {
        'as_strided',
        'detach',
        'diagonal',
        'expand',
        'expand_as',
        'movedim',
        'narrow',
        'permute',
        'select',
        'squeeze',
        'transpose',
        't',
        'real',
        'imag',
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_view_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_view_fn`。

### Lines 59-76
```python
        'view_as_real',
        'view_as_complex',
        'unflatten',
        'unfold',
        'unsqueeze',
        'view',
        'view_as',
        'unbind',
        'split',
        'split_with_sizes',
        'vsplit',
        'hsplit',
        'tensor_split',
        'chunk',
        'swapaxes',
        'slice',
        '_reshape_alias',
        '_unsafe_view',
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 77-94
```python
        '_conj',
        'alias',
    }

# manually populated from native_functions that have inplace_view: True.
# In the future we will probably be able to grab that list directly
def is_inplace_view_fn(func):
    return func.overloadpacket.__name__ in {
        'as_strided_',
        'detach_',
        'squeeze_',
        'swapaxes_',
        'swapdims_',
        't_',
        'transpose_',
        'unsqueeze_',
    }

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_inplace_view_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_inplace_view_fn`。

### Lines 95-103
```python

# Introspection please save us
def is_inplace(func):
    name = func.overloadpacket.__name__
    if re.match('__i.+__', name):
        return True
    if re.match('__.+__', name):
        return False
    return name[-1] == '_'
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_inplace`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_inplace`。

### Lines 104-115
```python


def generate_cct_and_mode(autograd_view_consistency=True):
    # This function returns a new class CompositeCompliantTensor
    # The two arguments control the behaviour described below.

    # autograd_view_consistency:
    #   If True, alias result using `set_` if func returns a view
    #   (See Note [Alias Result]).
    #   Since Forward AD doesn't work with `set_`
    #   we disable it by setting alias to False.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `generate_cct_and_mode`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`generate_cct_and_mode`。

### Lines 116-127
```python
    class CompositeCompliantTensor(torch.Tensor):
        elem: torch.Tensor

        __slots__ = ['elem']

        @staticmethod
        def __new__(cls, elem, mode, *args, **kwargs):
            if type(elem) is cls:
                raise AssertionError(
                    "Wrapping a CompositeCompliantTensor in a CompositeCompliantTensor is not supported"
                )

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `CompositeCompliantTensor`, `__new__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`CompositeCompliantTensor`, `__new__`。

### Lines 128-137
```python
            # The storage of CompositeCompliantTensor should never be used directly
            # by a Composite operation; if the Composite
            # operator attempts to read from the storage without dispatching then it'll
            # raise a RuntimeError due to it being a meta storage.
            r = torch.Tensor._make_wrapper_subclass(
                cls, elem.size(),
                dtype=elem.dtype, layout=elem.layout,
                device=elem.device, requires_grad=elem.requires_grad,
                strides=elem.stride(), storage_offset=elem.storage_offset())

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 138-155
```python
            if elem.requires_grad:
                # CompositeCompliantTensor steals the "requires_grad"-ness.
                # Why a new copy of `elem`? Because sometimes OpInfo shares inputs between tests...
                tmp = torch.empty(
                    (),
                    dtype=elem.dtype,
                    device=elem.device,
                    layout=elem.layout,
                    requires_grad=False,
                )
                # Use set_ rather than empty_strided() + copy_ so that we can preserve
                # things like storage_offset.
                tmp.set_(
                    source=elem.untyped_storage().clone(),
                    storage_offset=elem.storage_offset(),
                    size=elem.size(),
                    stride=elem.stride(),
                )
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 156-168
```python
                r.elem = tmp
            else:
                r.elem = elem

            if r.stride() != r.elem.stride():
                raise AssertionError(f"Expected r.stride() == r.elem.stride(), got {r.stride()} != {r.elem.stride()}")

            # Propagate conjugate bits to the wrapper tensor
            # Ref: https://github.com/albanD/subclass_zoo/issues/24
            # Ref: https://github.com/albanD/subclass_zoo/issues/21
            torch._C._set_conj(r, r.elem.is_conj())
            torch._C._set_neg(r, r.elem.is_neg())

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 169-179
```python
            r.mode = mode
            return r

        def __repr__(self):
            return f"CompositeCompliantTensor({self.elem})"

        @classmethod
        def __torch_dispatch__(cls, func, types, args=(), kwargs=None):
            all_args = pytree.arg_tree_leaves(*args, **(kwargs or {}))
            modes = tuple(e.mode for e in all_args if isinstance(e, CompositeCompliantTensor))
            if not all_same_mode(modes):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`, `__torch_dispatch__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`, `__torch_dispatch__`。

### Lines 180-188
```python
                raise RuntimeError("Multiple CompositeCompliantTensorModes NYI")
            with modes[0]:
                return func(*args, **kwargs)

    class CompositeCompliantTensorMode(TorchDispatchMode):
        def __torch_dispatch__(self, func, types, args=(), kwargs=None):
            def unwrap(e):
                return e.elem if isinstance(e, CompositeCompliantTensor) else e

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `CompositeCompliantTensorMode`, `__torch_dispatch__`, `unwrap`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`CompositeCompliantTensorMode`, `__torch_dispatch__`, `unwrap`。

### Lines 189-197
```python
            def wrap(e):
                return CompositeCompliantTensor(e, self) if isinstance(e, torch.Tensor) else e

            if func is torch.ops.aten._local_scalar_dense.default:
                raise RuntimeError(
                    ".item() is not allowed to be called inside of composite "
                    "functions in the PyTorch library because not all backends "
                    "and/or Tensor subclasses (e.g. vmap, ProxyTensor) support them.")

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrap`。

### Lines 198-207
```python
            if func.overloadpacket.__name__ in ('set_', 'resize_'):
                raise RuntimeError(
                    f"{func.__name__} is not allowed to be called inside of "
                    f"Composite operators.")

            if is_inplace(func):
                # NB: We are making an assumption that if the function is in-place,
                # then the first argument is being written to. Introspection please save us!
                mutated_argument = args[0]
                if not isinstance(mutated_argument, CompositeCompliantTensor) and \
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 208-219
```python
                        any(isinstance(a, CompositeCompliantTensor) for a in args[1:]):
                    raise RuntimeError(
                        'Not composite compliant: performing in-place operation '
                        f'{func.__name__} where the Tensor being written to is '
                        'regular Tensor but the other tensors are Tensor Subclasses. '
                        'Please try to avoid this in-place operation.')

            unwrapped_args = tree_map(unwrap, args)
            unwrapped_kwargs = tree_map(unwrap, kwargs)
            unwrapped_rs = func(*unwrapped_args, **unwrapped_kwargs)
            rs = tree_map(wrap, unwrapped_rs)

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 220-237
```python
            if is_view_fn(func) and autograd_view_consistency:
                # Note [Alias Result]
                # Autograd asserts that for B = A.view_fn(...), B and A's storages
                # are the same. Here we try to make B alias A to avoid those asserts.
                # See https://github.com/pytorch/pytorch/issues/65339 for more information
                # about the issue.
                with no_dispatch():
                    # Idea: this is a weird way of getting a storage that aliases the input.
                    # This is a workaround for #65339.
                    # 1. under no_dispatch, all of the wrapper tensors look like regular
                    #    tensors with special storage (the storage is nullptr and
                    #    advertises CPU/CUDA device.
                    # 2. we run func, which ends up running the view operation
                    # 3. All view operations reuse the input's storage and return
                    #    result Tensor(s) with new sizes/strides/offset that alias
                    #    the input.
                    # 4. we set the storage (and sizes/strides/offset) of the wrapper
                    #    tensor results to be that of the tensors that alias the input
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 238-249
```python
                    result = func(*args, **kwargs)
                    if isinstance(result, (tuple, list)):
                        for a, b in zip(rs, result, strict=True):
                            a.set_(b)
                    else:
                        rs.set_(result)

            # Some operations are allowed to in-place modify the metadata of the
            # inputs. The only ones are the "inplace view functions"; when we
            # run into these, we manually modify the metadata of the input.
            with no_dispatch():
                if is_inplace_view_fn(func):
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 250-259
```python
                    func(*args, **kwargs)

            # For each CompositeCompliantTensor t, we check that t and t.elem
            # have consistent metadata. If they don't have consistent metadata,
            # that means the operator did something fishy.
            check = partial(check_metadata_consistency, CCT=CompositeCompliantTensor)
            pytree.tree_map_(check, args)
            pytree.tree_map_(check, kwargs)
            pytree.tree_map_(check, rs)
            return rs
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 260-269
```python

    return CompositeCompliantTensor, CompositeCompliantTensorMode()

def is_tensorlist(lst):
    if not isinstance(lst, list) and not isinstance(lst, tuple):
        return False
    if len(lst) == 0:
        return False
    all_tensors = all(isinstance(elt, torch.Tensor) for elt in lst)
    if all_tensors:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_tensorlist`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_tensorlist`。

### Lines 270-278
```python
        return True
    exists_one_tensor = all(isinstance(elt, torch.Tensor) for elt in lst)
    if exists_one_tensor:
        raise RuntimeError('This test assumes that PyTorch APIs cannot take '
                           'mixed lists of Tensor and other things')
    return False


def maybe_map(fn, should_map, arg):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `maybe_map`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`maybe_map`。

### Lines 279-287
```python
    return fn(arg) if should_map else arg


def wrap(arg, CCT, cct_mode):
    # CCT: CompositeCompliantTensor class which is generated using generate_cct_and_mode
    if isinstance(arg, torch.Tensor):
        return CCT(arg, cct_mode)
    if is_tensorlist(arg):
        return [CCT(a, cct_mode) for a in arg]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrap`。

### Lines 288-301
```python
    raise RuntimeError("wrap assumes that the input can be wrapped")


# Given a list of flat arguments, some of which may be Tensors, return all
# possible ways some of the arguments could be CompositeCompliantTensors (CCT).
# For example, given Tensors A, B, C and flat_args = [A, 1, B],
# We would return the following 4 options:
# [CCT(A), 1, CCT(B)]
# [CCT(A), 1, B]
# [A, 1, CCT(B)]
# [A, 1, B]
# NB: Yes, this is exponential. No, we don't care too much because PyTorch ops
# don't accept that many input Tensors.
def generate_subclass_choices(flat_args, CCT, cct_mode):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `generate_subclass_choices`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`generate_subclass_choices`。

### Lines 302-311
```python
    # CCT: CompositeCompliantTensor class which is generated using generate_cct_and_mode
    is_tensor_likes = [isinstance(arg, torch.Tensor) or is_tensorlist(arg) for arg in flat_args]
    subclass_options = [[False, True] if is_tensor_like else [False] for is_tensor_like in is_tensor_likes]

    for which_args_are_wrapped in itertools.product(*subclass_options):

        result = [maybe_map(partial(wrap, CCT=CCT, cct_mode=cct_mode), should_wrap_arg, arg)
                  for should_wrap_arg, arg in zip(which_args_are_wrapped, flat_args, strict=True)]
        yield result, which_args_are_wrapped

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 312-320
```python

# For an operation f(*args, **kwargs), each Tensor argument may either be
# a regular Tensor or a Tensor Subclass. This iterator iterates through
# all of those options.
def generate_subclass_choices_args_kwargs(args, kwargs, CCT, cct_mode):
    # CCT: CompositeCompliantTensor class which is generated using generate_cct_and_mode
    flat_kwargs, spec = tree_flatten(kwargs)
    flat_args_kwargs = list(args) + list(flat_kwargs)
    for choice, debug_metadata in generate_subclass_choices(flat_args_kwargs, CCT, cct_mode):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `generate_subclass_choices_args_kwargs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`generate_subclass_choices_args_kwargs`。

### Lines 321-338
```python
        new_args = choice[:len(args)]
        new_kwargs = tree_unflatten(choice[len(args):], spec)
        which_args_are_wrapped = debug_metadata[:len(args)]
        which_kwargs_are_wrapped = tree_unflatten(debug_metadata[len(args):], spec)
        yield new_args, new_kwargs, which_args_are_wrapped, which_kwargs_are_wrapped


def raise_composite_compliance_error(err, additional_info=''):
    raise RuntimeError(
        "Composite compliance check failed with "
        "the above error.\n"
        f"{additional_info}"
        "If you are adding an OpInfo of an "
        "existing operator, please feel free to skip this test "
        "because the problem was pre-existing and file an issue. "
        "Otherwise, if you added a new operator, please read "
        "through the Composite Compliance section in "
        "aten/src/ATen/native/README.md for how to resolve this. "
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `raise_composite_compliance_error`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`raise_composite_compliance_error`。

### Lines 339-347
```python
    ) from err


# This test checks ALL possible permutations of calling `op` with arguments
# that are individually either a regular Tensor or a Tensor subclass.
#
# The general strategy is to wrap some Tensor args and kwargs in
# CompositeCompliantTensor wrappers and call the operation.

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 348-365
```python
# If some composite operation does any non-compliant behavior,
# CompositeCompliantTensor will raise an error.
def check_all_permutations(op, args, kwargs, assert_equal_fn):
    CCT, cct_mode = generate_cct_and_mode()
    expected = op(*args, **kwargs)
    for choice in generate_subclass_choices_args_kwargs(args, kwargs, CCT, cct_mode):
        new_args, new_kwargs, which_args_are_wrapped, which_kwargs_are_wrapped = choice

        try:
            actual = op(*new_args, **new_kwargs)
        # NOTE: [What errors are Composite Compliance trying to catch?]
        #
        # There's two things we want to catch:
        # - errors that would raise within the torch_dispatch impl
        # - data_ptr accesses
        # The first is easy to filter for (we could make the error a different
        # error class), the second is always going to be a RuntimeError due to
        # how it is implemented (if you try to access the data_ptr of the
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `check_all_permutations`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`check_all_permutations`。

### Lines 366-377
```python
        # wrapper Tensor, it raises you some internal RuntimeError).
        #
        # So the most general thing to catch here was RuntimeError. If you
        # are here and debugging why your test failed, it's plausible that
        # the operator itself is broken and that there are other tests failing.
        except RuntimeError as err:
            raise_composite_compliance_error(
                err,
                f"- wrapped_args: {which_args_are_wrapped}\n"
                f"- wrapped_kwargs: {which_kwargs_are_wrapped}\n"
            )

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 378-395
```python
        def unwrap(e):
            return e.elem if isinstance(e, CCT) else e

        assert_equal_fn(tree_map(unwrap, actual), expected)

# Checks via the usage of torch dispatch mode certain anti-patterns that
# are not composite compliant.
#
# In particular, the anti-pattern we are trying to prevent is a user
# creating an empty tensor and then resize_-ing it. Torch Dispatch Mode helps
# here because all factory functions will create tensors that are
# CompositeCompliantTensor.
#
# The general strategy is to wrap all Tensor args and kwargs in
# CompositeCompliantTensor wrappers. If an operator that is
# Composite does any non-compliant behavior,
# CompositeCompliantTensor will raise an error.
def check_with_mode(op, args, kwargs, assert_equal_fn):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `check_with_mode`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `check_with_mode`。

### Lines 396-411
```python
    CCT, cct_mode = generate_cct_and_mode()

    def wrap(e):
        return CCT(e, cct_mode) if isinstance(e, torch.Tensor) else e

    expected = op(*args, **kwargs)

    args = tree_map(wrap, args)
    kwargs = tree_map(wrap, kwargs)
    try:
        with cct_mode:
            actual = op(*args, **kwargs)
    # see NOTE: [What errors are Composite Compliance trying to catch?]
    except RuntimeError as err:
        raise_composite_compliance_error(err)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrap`。

### Lines 412-422
```python
    def unwrap(e):
        return e.elem if isinstance(e, CCT) else e

    assert_equal_fn(tree_map(unwrap, actual), expected)

def gather_leaf_tensors(args, kwargs):
    leaf_tensors = []
    args, _args_spec = tree_flatten(args)
    kwargs, _kwargs_spec = tree_flatten(kwargs)
    args = args + kwargs
    for arg in args:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `gather_leaf_tensors`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `gather_leaf_tensors`。

### Lines 423-431
```python
        if not isinstance(arg, torch.Tensor):
            continue
        if arg.requires_grad:
            leaf_tensors.append(arg)
    return leaf_tensors


def compute_expected_grads(op, args, kwargs, output_process_fn_grad=None, gradcheck_wrapper=None):
    if gradcheck_wrapper is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `compute_expected_grads`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`compute_expected_grads`。

### Lines 432-442
```python
        results = op(*args, **kwargs)
    else:
        results = gradcheck_wrapper(op, *args, **kwargs)

    if output_process_fn_grad is not None:
        results = output_process_fn_grad(results)

    flat_results = pytree.tree_leaves(results)
    flat_results = [r for r in flat_results if isinstance(r, torch.Tensor)]
    flat_diff_results = [r for r in flat_results if r.requires_grad]
    if len(flat_diff_results) <= 0:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 443-451
```python
        raise AssertionError("Expected len(flat_diff_results) > 0")

    grads = [torch.ones(r.shape, device=r.device, dtype=r.dtype) for r in flat_diff_results]
    leaf_tensors = gather_leaf_tensors(args, kwargs)
    if len(leaf_tensors) <= 0:
        raise AssertionError("Expected len(leaf_tensors) > 0")
    return torch.autograd.grad(flat_diff_results, leaf_tensors,
                               grads, allow_unused=True, retain_graph=True)

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 452-460
```python

# Checks if the backward formula is composite compliant by testing
# all possible permutations of {inputs, grad_outputs} being
# CompositeCompliantTensor or regular Tensors.
#
# NB: it is important that op is accepted as a Callable and not an OpInfo,
# this means we can apply check_backward_formula to things that aren't OpInfos
# while debugging.
def check_backward_formula(op: Callable, args, kwargs,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `check_backward_formula`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`check_backward_formula`。

### Lines 461-470
```python
                           output_process_fn_grad=None,
                           gradcheck_wrapper=None, assert_equal_fn=None):
    CCT, cct_mode = generate_cct_and_mode()

    expected = compute_expected_grads(op, args, kwargs, output_process_fn_grad, gradcheck_wrapper)

    for choice in generate_subclass_choices_args_kwargs(args, kwargs, CCT, cct_mode):
        new_args, new_kwargs, which_args_are_wrapped, which_kwargs_are_wrapped = choice
        leaf_tensors = gather_leaf_tensors(new_args, new_kwargs)
        if len(leaf_tensors) <= 0:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 471-487
```python
            raise AssertionError("Expected len(leaf_tensors) > 0")

        try:
            if gradcheck_wrapper is None:
                results = op(*new_args, **new_kwargs)
            else:
                results = gradcheck_wrapper(op, *new_args, **new_kwargs)
            if output_process_fn_grad is not None:
                results = output_process_fn_grad(results)
        # see NOTE: [What errors are Composite Compliance trying to catch?]
        except RuntimeError as err:
            raise_composite_compliance_error(
                err,
                f"- wrapped_args: {which_args_are_wrapped}\n"
                f"- wrapped_kwargs: {which_kwargs_are_wrapped}\n"
            )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 488-496
```python
        flat_results = pytree.tree_leaves(results)
        flat_results = [r for r in flat_results if isinstance(r, torch.Tensor)]
        flat_diff_results = [r for r in flat_results if r.requires_grad]
        if len(flat_diff_results) <= 0:
            raise AssertionError("Expected len(flat_diff_results) > 0")

        # NB: ones, not ones_like, so we get a regular Tensor here
        grads = [torch.ones(r.shape, device=r.device, dtype=r.dtype)
                 for r in flat_diff_results]
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 497-509
```python
        for flat_new_grads, which_grad_is_batched in generate_subclass_choices(grads, CCT, cct_mode):
            try:
                actual = torch.autograd.grad(flat_diff_results, leaf_tensors, flat_new_grads,
                                             allow_unused=True, retain_graph=True)
            # see NOTE: [What errors are Composite Compliance trying to catch?]
            except RuntimeError as err:
                raise_composite_compliance_error(
                    err,
                    f"- wrapped_args: {which_args_are_wrapped}\n"
                    f"- wrapped_kwargs: {which_kwargs_are_wrapped}\n"
                    f"- wrapped_grads: {which_grad_is_batched}\n"
                )

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 510-522
```python
            def unwrap(e):
                return e.elem if isinstance(e, CCT) else e

            assert_equal_fn(tuple(map(unwrap, actual)), expected, equal_nan=True)

# Checks if the forward AD formula is composite compliant by testing
# all possible permutations of {primals, tangents} being
# CompositeCompliantTensor or regular Tensors.
#
# NB: it is important that op is accepted as a Callable and not an OpInfo,
# this means we can apply check_forward_ad_formula to things that aren't OpInfos
# while debugging.
def check_forward_ad_formula(op: Callable, args, kwargs, gradcheck_wrapper=None, assert_equal_fn=None):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `check_forward_ad_formula`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `check_forward_ad_formula`。

### Lines 523-531
```python
    CCT, cct_mode = generate_cct_and_mode(autograd_view_consistency=False)

    def maybe_tangent(t):
        if type(t) is CCT:
            raise AssertionError("Expected type(t) is not CCT")
        # Generate `tangent` tensor
        # if given object is a Tensor and requires grad is set.
        if isinstance(t, torch.Tensor) and t.requires_grad:
            return torch.randn_like(t)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `maybe_tangent`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`maybe_tangent`。

### Lines 532-540
```python
        elif is_tensorlist(t):
            return [torch.randn_like(e) if e.requires_grad else None for e in t]
        return None

    tangent_args = tuple(maybe_tangent(arg) for arg in args)
    flat_kwargs, spec = tree_flatten(kwargs)
    flat_tangent_kwargs = tuple(maybe_tangent(arg) for arg in flat_kwargs)
    tangent_kwargs = tree_unflatten(flat_tangent_kwargs, spec)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 541-549
```python
    with fwAD.dual_level():
        def maybe_make_dual(dual):
            # Returns dual tensor if primal is a tensor/tensor subclass
            # with requires_grad set.
            primal, tangent = dual
            if isinstance(primal, torch.Tensor) and primal.requires_grad:
                return fwAD.make_dual(primal.detach(), tangent)
            elif is_tensorlist(primal):
                return tuple(fwAD.make_dual(pri.detach(), tang) if tang is not None else pri
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `maybe_make_dual`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`maybe_make_dual`。

### Lines 550-558
```python
                             for pri, tang in zip(primal, tangent, strict=True))
            return primal

        def compute_expected_grad(args, tangent_args, kwargs, tangent_kwargs):
            op_args = tuple(map(maybe_make_dual, zip(args, tangent_args, strict=True)))
            op_kwargs = {k: maybe_make_dual((v, tangent_kwargs[k])) for k, v in kwargs.items()}

            if gradcheck_wrapper is None:
                return op(*op_args, **op_kwargs)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `compute_expected_grad`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`compute_expected_grad`。

### Lines 559-573
```python
            return gradcheck_wrapper(op, *op_args, **op_kwargs)

        expected = compute_expected_grad(args, tangent_args, kwargs, tangent_kwargs)
        expected = tree_map(fwAD.unpack_dual, expected)
        expected_primals = tree_map(
            lambda x: x.primal,
            expected,
            is_leaf=lambda x: type(x) is fwAD.UnpackedDualTensor,
        )
        expected_tangents = tree_map(
            lambda x: x.tangent,
            expected,
            is_leaf=lambda x: type(x) is fwAD.UnpackedDualTensor,
        )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 574-582
```python
        # Permutations of arg and kwargs in CCT.
        for choice in generate_subclass_choices_args_kwargs(args, kwargs, CCT, cct_mode):
            new_args, new_kwargs, which_args_are_wrapped, which_kwargs_are_wrapped = choice

            # Permutations tangent arg and tangent kwargs in CCT.
            for tang_choice in generate_subclass_choices_args_kwargs(tangent_args, tangent_kwargs, CCT, cct_mode):
                new_tang_args, new_tang_kwargs, \
                    which_tang_args_are_wrapped, which_tang_kwargs_are_wrapped = tang_choice

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 583-600
```python
                op_args = tuple(map(maybe_make_dual, zip(new_args, new_tang_args, strict=True)))
                op_kwargs = {k: maybe_make_dual((v, new_tang_kwargs[k])) for k, v in new_kwargs.items()}

                try:
                    if gradcheck_wrapper is None:
                        actual = op(*op_args, **op_kwargs)
                    else:
                        actual = gradcheck_wrapper(op, *op_args, **op_kwargs)
                # see NOTE: [What errors are Composite Compliance trying to catch?]
                except RuntimeError as err:
                    raise_composite_compliance_error(
                        err,
                        f"- wrapped_args: {which_args_are_wrapped}\n"
                        f"- wrapped_kwargs: {which_kwargs_are_wrapped}\n"
                        f"- wrapped_tangent_args: {which_tang_args_are_wrapped}\n"
                        f"- wrapped_tangent_kwargs: {which_tang_kwargs_are_wrapped}\n"
                    )

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 601-616
```python
                def unwrap(e):
                    return e.elem if isinstance(e, CCT) else e

                actual = tree_map(fwAD.unpack_dual, actual)
                actual_primals = tree_map(
                    lambda x: unwrap(x.primal),
                    actual,
                    is_leaf=lambda x: type(x) is fwAD.UnpackedDualTensor,
                )
                actual_tangents = tree_map(
                    lambda x: unwrap(x.tangent),
                    actual,
                    is_leaf=lambda x: type(x) is fwAD.UnpackedDualTensor,
                )
                assert_equal_fn(actual_primals, expected_primals, equal_nan=True)
                assert_equal_fn(actual_tangents, expected_tangents, equal_nan=True)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.utils._python_dispatch`, `torch.utils._pytree`, `torch.utils`, `torch.utils._mode_utils`, `torch.autograd.forward_ad`
- External imports / 外部导入: `itertools`, `functools`, `collections.abc`, `re`
- Representative symbols / 代表性符号: `check_attr_consistency`, `check_metadata_consistency`, `is_view_fn`, `is_inplace_view_fn`, `is_inplace`, `generate_cct_and_mode`, `is_tensorlist`, `maybe_map`, `wrap`, `generate_subclass_choices`, `...`

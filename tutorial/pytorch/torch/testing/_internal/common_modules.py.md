# common_modules.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/common_modules.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common modules, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common modules 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```python
# mypy: ignore-errors

import torch
import unittest
from copy import deepcopy
from enum import Enum
from functools import wraps, partial
from itertools import chain, product
import itertools
import math
import torch.nn.functional as F
from torch.nn.utils.rnn import pack_padded_sequence
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.nn.functional`, `torch.nn.utils.rnn`; external imports: `unittest`, `copy`, `enum`, `functools`, `itertools`, `math`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.nn.functional`, `torch.nn.utils.rnn`；外部导入：`unittest`, `copy`, `enum`, `functools`, `itertools`, `math`。

### Lines 13-26
```python
from torch.testing import make_tensor
from torch.testing._internal.common_cuda import TEST_CUDNN
from torch.testing._internal.common_dtype import (
    floating_types, floating_and_complex_types_and, get_all_fp_dtypes)
from torch.testing._internal.common_device_type import (
    _TestParametrizer, _update_param_kwargs, expectedFailureMPS, toleranceOverride, tol,
    precisionOverride, skipMeta, skipMPS)
from torch.testing._internal.common_methods_invocations import DecorateInfo
from torch.testing._internal.common_nn import (
    cosineembeddingloss_reference, cross_entropy_loss_reference, ctcloss_reference,
    hingeembeddingloss_reference, huberloss_reference, kldivloss_reference,
    marginrankingloss_reference, multimarginloss_reference, multilabelmarginloss_reference,
    nllloss_reference, nlllossNd_reference, smoothl1loss_reference, softmarginloss_reference, get_reduction)
from torch.testing._internal.common_utils import (
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 27-40
```python
    freeze_rng_state, skipIfMPS, GRADCHECK_NONDET_TOL, TEST_WITH_ROCM, IS_WINDOWS,
    skipIfTorchDynamo, skipIfXpu)
from types import ModuleType
import operator

# List of all namespaces containing modules to test.
MODULE_NAMESPACES: list[ModuleType] = [
    torch.nn.modules,
    torch.ao.nn.qat.modules,
    torch.ao.nn.quantizable.modules,
    torch.ao.nn.quantized.modules,
    torch.ao.nn.quantized.modules,
]

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构。关键符号：无明显局部符号。

### Lines 41-53
```python
# Modules that shouldn't be tested for one reason or another.
MODULES_TO_SKIP: set[type] = {
    torch.nn.Module,  # abstract base class
    torch.nn.Container,  # deprecated
    torch.nn.NLLLoss2d,  # deprecated
    torch.ao.nn.quantized.MaxPool2d,  # aliases to nn.MaxPool2d
    torch.ao.nn.quantized.MaxPool2d,  # aliases to nn.MaxPool2d
}

# List of all module classes to test.
MODULE_CLASSES: list[type] = [*chain.from_iterable([
    [getattr(namespace, module_name) for module_name in namespace.__all__]  # type: ignore[attr-defined]
    for namespace in MODULE_NAMESPACES])]
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 54-65
```python
MODULE_CLASSES = [cls for cls in MODULE_CLASSES if cls not in MODULES_TO_SKIP]

# Dict of module class -> common name. Useful for making test names more intuitive.
# Example: torch.nn.modules.linear.Linear -> "nn.Linear"
MODULE_CLASS_NAMES: dict[type, str] = {}
for namespace in MODULE_NAMESPACES:
    for module_name in namespace.__all__:  # type: ignore[attr-defined]
        module_cls = getattr(namespace, module_name)
        namespace_name = namespace.__name__.replace('torch.', '').replace('.modules', '')

        # Deal with any aliases by preferring earlier names.
        if module_cls not in MODULE_CLASS_NAMES:
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 66-82
```python
            MODULE_CLASS_NAMES[module_cls] = f'{namespace_name}.{module_name}'


# Specifies the modes (i.e. train, eval) to test over.
TrainEvalMode = Enum('TrainEvalMode', ('train_only', 'eval_only', 'train_and_eval'))


class modules(_TestParametrizer):
    """ PROTOTYPE: Decorator for specifying a list of modules over which to run a test. """

    def __init__(self, module_info_iterable, allowed_dtypes=None,
                 train_eval_mode=TrainEvalMode.train_and_eval, skip_if_dynamo=True):
        self.module_info_list = list(module_info_iterable)
        self.allowed_dtypes = set(allowed_dtypes) if allowed_dtypes is not None else None
        self.train_eval_mode = train_eval_mode
        self.skip_if_dynamo = skip_if_dynamo

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `modules`, `__init__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`modules`, `__init__`。

### Lines 83-94
```python
    def _get_training_flags(self, module_info):
        training_flags = []
        if (self.train_eval_mode == TrainEvalMode.train_only or
                self.train_eval_mode == TrainEvalMode.train_and_eval):
            training_flags.append(True)

        if (self.train_eval_mode == TrainEvalMode.eval_only or
                self.train_eval_mode == TrainEvalMode.train_and_eval):
            training_flags.append(False)

        # If train and eval modes don't differ for the module, don't bother using more than one.
        if not module_info.train_and_eval_differ:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_training_flags`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_training_flags`。

### Lines 95-107
```python
            training_flags = training_flags[:1]

        return training_flags

    def _parametrize_test(self, test, generic_cls, device_cls):
        if device_cls is None:
            raise RuntimeError('The @modules decorator is only intended to be used in a device-specific '
                               'context; use it with instantiate_device_type_tests() instead of '
                               'instantiate_parametrized_tests()')

        for module_info in self.module_info_list:
            dtypes = set(module_info.supported_dtypes(device_cls.device_type))
            if self.allowed_dtypes is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_parametrize_test`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_parametrize_test`。

### Lines 108-122
```python
                dtypes = dtypes.intersection(self.allowed_dtypes)

            training_flags = self._get_training_flags(module_info)
            for (training, dtype) in product(training_flags, dtypes):
                # Construct the test name; device / dtype parts are handled outside.
                # See [Note: device and dtype suffix placement]
                test_name = module_info.formatted_name
                if len(training_flags) > 1:
                    test_name += f"_{'train_mode' if training else 'eval_mode'}"

                # Construct parameter kwargs to pass to the test.
                param_kwargs = {'module_info': module_info}
                _update_param_kwargs(param_kwargs, 'dtype', dtype)
                _update_param_kwargs(param_kwargs, 'training', training)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 123-134
```python
                try:

                    @wraps(test)
                    def test_wrapper(*args, **kwargs):
                        return test(*args, **kwargs)

                    if self.skip_if_dynamo and not torch.testing._internal.common_utils.TEST_WITH_TORCHINDUCTOR:
                        test_wrapper = skipIfTorchDynamo("Policy: we don't run ModuleInfo tests w/ Dynamo")(test_wrapper)

                    decorator_fn = partial(module_info.get_decorators, generic_cls.__name__,
                                           test.__name__, device_cls.device_type, dtype)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `test_wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`test_wrapper`。

### Lines 135-147
```python
                    yield (test_wrapper, test_name, param_kwargs, decorator_fn)
                except Exception as ex:
                    # Provides an error message for debugging before rethrowing the exception
                    print(f"Failed to instantiate {test_name} for module {module_info.name}!")
                    raise ex


def get_module_common_name(module_cls):
    if module_cls in MODULE_CLASS_NAMES:
        # Example: "nn.Linear"
        return MODULE_CLASS_NAMES[module_cls]
    else:
        return module_cls.__name__
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_module_common_name`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_module_common_name`。

### Lines 148-159
```python


class FunctionInput:
    """ Contains args and kwargs to pass as input to a function. """
    __slots__ = ['args', 'kwargs']

    def __init__(self, *args, **kwargs):
        self.args = args
        self.kwargs = kwargs


class ModuleInput:
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `FunctionInput`, `__init__`, `ModuleInput`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`FunctionInput`, `__init__`, `ModuleInput`。

### Lines 160-171
```python
    """ Contains args / kwargs for module instantiation + forward pass. """
    __slots__ = ['constructor_input', 'forward_input', 'desc', 'reference_fn']

    def __init__(self, constructor_input, forward_input=None, desc='', reference_fn=None):
        self.constructor_input = constructor_input  # Inputs to pass during construction
        self.forward_input = forward_input  # Inputs to pass to forward()
        self.desc = desc  # Description for this set of inputs
        self.reference_fn = reference_fn  # Reference with signature: reference_fn(module, parameters, *args, **kwargs)

        if reference_fn is not None:

            @wraps(reference_fn)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 172-185
```python
            def copy_reference_fn(m, *args, **kwargs):
                # Copy inputs to avoid undesired side effects from calling the reference.
                args, kwargs = deepcopy(args), deepcopy(kwargs)

                # Note that module parameters are passed in for convenience.
                return reference_fn(m, list(m.parameters()), *args, **kwargs)

            self.reference_fn = copy_reference_fn

class ModuleErrorEnum(Enum):
    """ Enumerates when error is raised when testing modules. """
    CONSTRUCTION_ERROR = 0
    FORWARD_ERROR = 1

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `copy_reference_fn`, `ModuleErrorEnum`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`copy_reference_fn`, `ModuleErrorEnum`。

### Lines 186-204
```python
class ErrorModuleInput:
    """
    A ModuleInput that will cause the operation to throw an error plus information
    about the resulting error.
    """

    __slots__ = ["module_error_input", "error_on", "error_type", "error_regex"]

    def __init__(self,
                 module_error_input,
                 *,
                 error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
                 error_type=RuntimeError,
                 error_regex):
        self.module_error_input = module_error_input
        self.error_on = error_on
        self.error_type = error_type
        self.error_regex = error_regex

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ErrorModuleInput`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ErrorModuleInput`。

### Lines 205-228
```python

class ModuleInfo:
    """ Module information to be used in testing. """

    def __init__(self,
                 module_cls,  # Class object for the module under test
                 *,
                 module_inputs_func,  # Function to generate module inputs
                 skips=(),  # Indicates which tests to skip
                 decorators=None,  # Additional decorators to apply to generated tests
                 dtypes=floating_types(),  # dtypes this function is expected to work with
                 dtypesIfMPS=(torch.float16, torch.float32,),  # dtypes this function is expected to work with on MPS
                 dtypesIfHpu=(torch.bfloat16, torch.float32,),
                 supports_gradgrad=True,  # whether the op supports second order gradients
                 gradcheck_nondet_tol=0.0,  # tolerance for nondeterminism while performing gradcheck
                 module_memformat_affects_out=False,  # whether converting module to channels last will generate
                                                      # channels last output
                 train_and_eval_differ=False,  # whether the module has differing behavior between train and eval
                 module_error_inputs_func=None,  # Function to generate module inputs that error
                 gradcheck_fast_mode=None,  # Whether to use the fast implementation for gradcheck/gradgradcheck.
                                            # When set to None, defers to the default value provided by the wrapper
                                            # function around gradcheck (testing._internal.common_utils.gradcheck)
                 ):
        self.module_cls = module_cls
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ModuleInfo`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ModuleInfo`。

### Lines 229-241
```python
        self.module_inputs_func = module_inputs_func
        self.decorators = (*(decorators if decorators else []), *(skips if skips else []))
        self.dtypes = dtypes
        self.dtypesIfMPS = dtypesIfMPS
        self.dtypesIfHpu = dtypesIfHpu
        self.supports_gradgrad = supports_gradgrad
        self.gradcheck_nondet_tol = gradcheck_nondet_tol
        self.module_memformat_affects_out = module_memformat_affects_out
        self.train_and_eval_differ = train_and_eval_differ
        self.module_error_inputs_func = module_error_inputs_func
        self.gradcheck_fast_mode = gradcheck_fast_mode
        self.is_lazy = issubclass(module_cls, torch.nn.modules.lazy.LazyModuleMixin)

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 242-253
```python
    def get_decorators(self, test_class, test_name, device, dtype, param_kwargs):
        result = []
        for decorator in self.decorators:
            if isinstance(decorator, DecorateInfo):
                if decorator.is_active(test_class, test_name, device, dtype, param_kwargs):
                    result.extend(decorator.decorators)
            else:
                result.append(decorator)
        return result

    def supported_dtypes(self, device_type):
        if device_type == 'mps':
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_decorators`, `supported_dtypes`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_decorators`, `supported_dtypes`。

### Lines 254-265
```python
            return self.dtypesIfMPS
        elif device_type == 'hpu':
            return self.dtypesIfHpu
        else:
            return self.dtypes

    @property
    def name(self):
        return get_module_common_name(self.module_cls)

    @property
    def formatted_name(self):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `name`, `formatted_name`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`name`, `formatted_name`。

### Lines 266-286
```python
        return self.name.replace('.', '_')

# Start of module inputs functions.

def module_inputs_torch_nn_Linear(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    module_inputs = [
        ModuleInput(constructor_input=FunctionInput(10, 8),
                    forward_input=FunctionInput(input=make_input((4, 10))),
                    reference_fn=lambda m, p, input: torch.mm(input, p[0].t()) + p[1].view(1, -1).expand(4, 8)),
        ModuleInput(constructor_input=FunctionInput(10, 8, bias=False),
                    forward_input=FunctionInput(make_input((4, 10))),
                    desc='no_bias',
                    reference_fn=lambda m, p, i: torch.mm(i, p[0].t())),
        ModuleInput(constructor_input=FunctionInput(3, 5),
                    forward_input=FunctionInput(make_input(3)),
                    desc='no_batch_dim',
                    reference_fn=lambda m, p, i: torch.mm(i.view(1, -1), p[0].t()).view(-1) + p[1])
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Linear`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Linear`。

### Lines 287-300
```python
    return module_inputs


def module_inputs_torch_nn_Bilinear(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    def bilinear_reference_fn(m, p, x1, x2, bias=True):
        result = torch.einsum('bn,anm,bm->ba', x1, p[0], x2)
        if bias:
            if x1.shape[0] == 1:
                result = result.view(-1) + p[1]
            else:
                result = result + p[1].view(1, -1).expand(x1.shape[0], p[0].shape[0])
        return result
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Bilinear`, `bilinear_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Bilinear`, `bilinear_reference_fn`。

### Lines 301-315
```python

    module_inputs = [
        ModuleInput(constructor_input=FunctionInput(2, 3, 4),
                    forward_input=FunctionInput(make_input((8, 2)), make_input((8, 3))),
                    reference_fn=bilinear_reference_fn),
        ModuleInput(constructor_input=FunctionInput(2, 3, 4, bias=False),
                    forward_input=FunctionInput(make_input((8, 2)), make_input((8, 3))),
                    desc='no_bias',
                    reference_fn=lambda m, p, x1, x2: bilinear_reference_fn(m, p, x1, x2, bias=False)),
        ModuleInput(constructor_input=FunctionInput(2, 3, 4),
                    forward_input=FunctionInput(make_input(2), make_input(3)),
                    desc='no_batch_dim',
                    reference_fn=lambda m, p, x1, x2: bilinear_reference_fn(m, p, x1.view(1, -1), x2.view(1, -1))),
    ]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 316-329
```python
    return module_inputs


def module_inputs_torch_nn_KLDivLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_batchmean', {'reduction': 'batchmean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('log_target', {'log_target': True})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_KLDivLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_KLDivLoss`。

### Lines 330-343
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return kldivloss_reference(i, t, **constructor_kwargs)

        input = make_input((10, 10)).log()
        target = make_input((10, 10)) if kwargs.get('log_target', False) else make_input((10, 10)).log()
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(input, target),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 344-355
```python
        scalar_input = make_input(()).log()
        # FIXME(rec): scalar_target is unused, perhaps should be argument to FunctionInput?
        scalar_target = (  # noqa: F841
            make_input(()) if kwargs.get('log_target', False) else make_input(()).log()
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(scalar_input, scalar_input),
                        desc='scalar_' + desc,
                        reference_fn=reference_fn)
        )

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 356-374
```python
    return module_inputs


def module_inputs_torch_nn_NLLLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    def make_input(shape, device=device, dtype=dtype, requires_grad=requires_grad):
        return make_tensor(shape, device=device, dtype=dtype,
                           requires_grad=False).log_softmax(dim=1).requires_grad_(requires_grad)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_none', {'reduction': 'none'}),
        ('ignore_index', {'ignore_index': 2}),
        ('weights', {'weight': make_weight(4).abs()}),
        ('weights_ignore_index', {'weight': make_weight(4).abs(), 'ignore_index': 2}),
        ('weights_ignore_index_neg', {'weight': make_weight(4).abs(), 'ignore_index': -1})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_NLLLoss`, `make_input`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_NLLLoss`, `make_input`。

### Lines 375-392
```python
    # TODO: Uncomment when negative weights is supported.
    # negative_weight = make_weight(10)
    # negative_weight[0] = -1
    # cases.append(('weights_negative', {'weight': negative_weight}))
    module_inputs = []
    for desc, constructor_kwargs in cases:

        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return nllloss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((15, 4)),
                                                    torch.empty(15, device=device).uniform_().mul(4).floor().long()),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 393-404
```python
        def nd_reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return nlllossNd_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(
                            make_input((2, 4, 5, 5)),
                            torch.empty(2, 5, 5, device=device).uniform_().mul(4).floor().long()),
                        desc=f"nd_{desc}",
                        reference_fn=nd_reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `nd_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`nd_reference_fn`。

### Lines 405-422
```python
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(
                            make_input((2, 4, 5, 5, 2, 2)),
                            torch.empty(2, 5, 5, 2, 2, device=device).uniform_().mul(4).floor().long()),
                        desc=f"higher_dim_{desc}",
                        reference_fn=nd_reference_fn)
        )

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(
                            make_input((2, 4, 5)),
                            torch.empty(2, 5, device=device).uniform_().mul(4).floor().long()),
                        desc=f"3d_{desc}",
                        reference_fn=nd_reference_fn)
        )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 423-437
```python
    return module_inputs


def module_inputs_torch_nn_GaussianNLLLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('homoscedastic', {'homoscedastic': True}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_GaussianNLLLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_GaussianNLLLoss`。

### Lines 438-450
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        homoscedastic = constructor_kwargs.pop('homoscedastic', False)
        var_input = make_input(1, 3).abs() if homoscedastic else make_input(4, 1).abs()
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input(4, 3),
                                                    make_target(4, 3),
                                                    var_input),
                        desc=desc,
                        reference_fn=no_batch_dim_reference_fn)
        )

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 451-467
```python
    return module_inputs


def module_inputs_torch_nn_PoissonNLLLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('full', {'full': True}),
        ('no_log_input', {'log_input': False}),
        ('full_no_log_input', {'full': True, 'log_input': False}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_PoissonNLLLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_PoissonNLLLoss`。

### Lines 468-480
```python
    def poissonnllloss_reference_fn(i, t, log_input=True, full=False, reduction='mean', eps=1e-8):
        if log_input:
            result = i.exp() - t.mul(i)
        else:
            result = i - t.mul((i + eps).log())

        if full:
            result += (t.mul(t.log()) - t + 0.5 * (2. * math.pi * t).log()).masked_fill(t <= 1, 0)

        if reduction == 'none':
            return result
        elif reduction == 'mean':
            return result.sum() / i.numel()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `poissonnllloss_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`poissonnllloss_reference_fn`。

### Lines 481-498
```python
        else:
            return result.sum()

    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return poissonnllloss_reference_fn(i, t, **constructor_kwargs)

        log_input = constructor_kwargs.get('log_input', True)
        input = make_input((2, 3, 4, 5)) if log_input else make_input((2, 3, 4, 5)).abs().add(0.001)
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(input,
                                                    make_target((2, 3, 4, 5)).floor_().abs_()),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 499-512
```python
    return module_inputs


def module_inputs_torch_nn_MSELoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MSELoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MSELoss`。

### Lines 513-536
```python
    def mse_loss_reference_fn(m, p, i, t, reduction='mean'):
        if reduction == 'none':
            return (i - t).pow(2)
        elif reduction == 'mean':
            return (i - t).pow(2).sum() / i.numel()
        else:
            return (i - t).pow(2).sum()

    module_inputs = []
    for desc, constructor_kwargs in cases:
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((2, 3, 4, 5)),
                                                    make_target((2, 3, 4, 5))),
                        desc=desc,
                        reference_fn=partial(mse_loss_reference_fn, **constructor_kwargs))
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input(()),
                                                    make_target(())),
                        desc=f'{desc}_scalar',
                        reference_fn=partial(mse_loss_reference_fn, **constructor_kwargs))
        )
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `mse_loss_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`mse_loss_reference_fn`。

### Lines 537-548
```python

    return module_inputs


def no_batch_dim_reference_fn(m, p, *args, **kwargs):
    """Reference function for modules supporting no batch dimensions.

    Unbatched inputs are unsqueezed to form a
    single batch input before passing them to the module.
    The output is squeezed to compare with the
    output of unbatched input to the module.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `no_batch_dim_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`no_batch_dim_reference_fn`。

### Lines 549-561
```python
    Currently it only supports modules which return a single Tensor as output.
    You can bind the following kwargs.
    Kwargs:
        batch_first[bool] : If True, all the Tensors in `args` while be unsqueezed at dim `0` .
                        and output will be squeezed at dim `0` else dim `1` for both.
        kwargs_to_batchify[dict] : Dictionary specifying the name of the argument and dimension to unsqueeze.
                               Useful if there are few arguments whose batch dimension are different
                               from the ones selected by `batch_first`.
        is_criterion[bool] : Specify if the module is a criterion and handle the reduction for output accordingly.
    """
    def get_and_pop(key, default):
        v = kwargs.get(key, default)
        if key in kwargs:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_and_pop`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_and_pop`。

### Lines 562-573
```python
            kwargs.pop(key)
        return v

    batch_dim = 0 if get_and_pop('batch_first', True) else 1
    kwargs_to_batchify = get_and_pop('kwargs_to_batchify', None)
    is_criterion = get_and_pop('is_criterion', False)

    if kwargs_to_batchify is not None:
        if not isinstance(kwargs_to_batchify, dict):
            raise AssertionError(f"Expected kwargs_to_batchify to be a dict, got {type(kwargs_to_batchify)}")
        for k, v in kwargs.items():
            if k in kwargs_to_batchify and v is not None:
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 574-585
```python
                bdim = kwargs_to_batchify[k]
                kwargs[k] = v.unsqueeze(bdim)

    single_batch_input_args = [input.unsqueeze(batch_dim) for input in args]
    with freeze_rng_state():
        output = m(*single_batch_input_args, **kwargs).squeeze(batch_dim)

    if is_criterion:
        reduction = get_reduction(m)
        if reduction == 'none':
            return output.squeeze(0)
    return output
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 586-597
```python


def no_batch_dim_reference_mha(m, p, *args, **kwargs):
    """Reference function for MultiheadAttention supporting no batch dimensions.

    Unbatched inputs are unsqueezed to form a
    single batch input before passing them to the module.
    The output is squeezed to compare with the
    output of unbatched input to the module.
    """
    batch_dim = 0 if kwargs.get('batch_first', True) else 1
    if 'batch_first' in kwargs:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `no_batch_dim_reference_mha`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`no_batch_dim_reference_mha`。

### Lines 598-609
```python
        kwargs.pop('batch_first')
    if 'key_padding_mask' in kwargs and kwargs['key_padding_mask'] is not None:
        kwargs['key_padding_mask'] = kwargs['key_padding_mask'].unsqueeze(0)
    single_batch_input_args = [input.unsqueeze(batch_dim) for input in args]
    with freeze_rng_state():
        output = m(*single_batch_input_args, **kwargs)
        return (output[0].squeeze(batch_dim), output[1].squeeze(0))


def no_batch_dim_reference_rnn_gru(m, p, *args, **kwargs):
    """Reference function for RNN and GRU supporting no batch dimensions.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `no_batch_dim_reference_rnn_gru`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`no_batch_dim_reference_rnn_gru`。

### Lines 610-621
```python
    Unbatched inputs are unsqueezed to form a
    single batch input before passing them to the module.
    The output is squeezed to compare with the
    output of unbatched input to the module.
    """
    if len(args) == 1:
        inp, = args
        h = None
    elif len(args) == 2:
        inp, h = args
        h = h.unsqueeze(1)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 622-633
```python
    batch_dim = 0 if kwargs['batch_first'] else 1
    kwargs.pop('batch_first')
    inp = inp.unsqueeze(batch_dim)
    single_batch_input_args = (inp, h)
    with freeze_rng_state():
        output = m(*single_batch_input_args, **kwargs)
        return (output[0].squeeze(batch_dim), output[1].squeeze(1))


def no_batch_dim_reference_lstm(m, p, *args, **kwargs):
    """Reference function for LSTM supporting no batch dimensions.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `no_batch_dim_reference_lstm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`no_batch_dim_reference_lstm`。

### Lines 634-645
```python
    Unbatched inputs are unsqueezed to form a
    single batch input before passing them to the module.
    The output is squeezed to compare with the
    output of unbatched input to the module.
    """
    if len(args) == 1:
        inp, = args
        h = None
    elif len(args) == 2:
        inp, h = args
        h = (h[0].unsqueeze(1), h[1].unsqueeze(1))

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 646-657
```python
    batch_dim = 0 if kwargs['batch_first'] else 1
    kwargs.pop('batch_first')
    inp = inp.unsqueeze(batch_dim)
    single_batch_input_args = (inp, h)
    with freeze_rng_state():
        output = m(*single_batch_input_args, **kwargs)
        return (output[0].squeeze(batch_dim), (output[1][0].squeeze(1), output[1][1].squeeze(1)))


def no_batch_dim_reference_lstmcell(m, p, *args, **kwargs):
    """Reference function for LSTMCell supporting no batch dimensions.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `no_batch_dim_reference_lstmcell`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`no_batch_dim_reference_lstmcell`。

### Lines 658-669
```python
    The module is passed the input and target in batched form with a single item.
    The output is squeezed to compare with the no-batch input.
    """
    inp, (h, c) = args
    single_batch_input_args = (inp.unsqueeze(0), (h.unsqueeze(0), c.unsqueeze(0)))
    with freeze_rng_state():
        output = m(*single_batch_input_args, **kwargs)
        return (output[0].squeeze(0), output[1].squeeze(0))


def generate_regression_criterion_inputs(make_input):
    return [
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `generate_regression_criterion_inputs`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`generate_regression_criterion_inputs`。

### Lines 670-681
```python
        ModuleInput(
            constructor_input=FunctionInput(reduction=reduction),
            forward_input=FunctionInput(make_input((4, )), make_input(4,)),
            reference_fn=partial(no_batch_dim_reference_fn, is_criterion=True),
            desc=f'no_batch_dim_{reduction}'
        ) for reduction in ['none', 'mean', 'sum']]


def module_inputs_torch_nn_AvgPool1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AvgPool1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AvgPool1d`。

### Lines 682-694
```python
        ModuleInput(constructor_input=FunctionInput(kernel_size=2),
                    forward_input=FunctionInput(make_input((3, 6))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn),
        ModuleInput(constructor_input=FunctionInput(2),
                    forward_input=FunctionInput(make_input((2, 3, 6)))),
        ModuleInput(constructor_input=FunctionInput((2,), (2,)),
                    forward_input=FunctionInput(make_input((2, 3, 6))),
                    desc='stride'),
        ModuleInput(constructor_input=FunctionInput(2, 2, 1),
                    forward_input=FunctionInput(make_input((2, 3, 6))),
                    desc='stride_pad')]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 695-718
```python

def module_inputs_torch_nn_AvgPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput((2, 2)),
                    forward_input=FunctionInput(make_input((3, 6, 6))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn),
        ModuleInput(constructor_input=FunctionInput((2, 2)),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6)))),
        ModuleInput(constructor_input=FunctionInput((2, 2), (2, 2)),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='stride'),
        ModuleInput(constructor_input=FunctionInput((2, 2), (2, 2), (1, 1)),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='stride_pad'),
        ModuleInput(constructor_input=FunctionInput((2, 2), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='divisor'),
        ModuleInput(constructor_input=FunctionInput((2, 2), (2, 2), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='divisor_stride'),
        ModuleInput(constructor_input=FunctionInput((2, 2), (2, 2), (1, 1), divisor_override=1),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AvgPool2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AvgPool2d`。

### Lines 719-742
```python
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='divisor_stride_pad')]



def module_inputs_torch_nn_AvgPool3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput((2, 2, 2)),
                    forward_input=FunctionInput(make_input((3, 4, 4, 4))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn),
        ModuleInput(constructor_input=FunctionInput((2, 2, 2)),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4)))),
        ModuleInput(constructor_input=FunctionInput(2, (2, 2, 2)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
                    desc='stride'),
        ModuleInput(constructor_input=FunctionInput(2, 2, (1, 1, 1)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
                    desc='stride_pad'),
        ModuleInput(constructor_input=FunctionInput(4, 2, (1, 2, 1)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
                    desc='stride_pad_gpu_fixedkw_output'),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AvgPool3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AvgPool3d`。

### Lines 743-766
```python
        ModuleInput(constructor_input=FunctionInput((2, 4, 8), 1, (1, 1, 2)),
                    forward_input=FunctionInput(make_input((2, 3, 2, 4, 8))),
                    desc='stride_pad_gpu_general_output'),
        ModuleInput(constructor_input=FunctionInput(3, 1, 0),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='stride1_pad0_gpu_input'),
        ModuleInput(constructor_input=FunctionInput(2, 2, (1, 1, 1)),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='stride_pad_gpu_input_nooverlap'),
        ModuleInput(constructor_input=FunctionInput((2, 2, 2), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='divisor'),
        ModuleInput(constructor_input=FunctionInput(2, (2, 2, 2), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
                    desc='divisor_stride'),
        ModuleInput(constructor_input=FunctionInput(2, 2, (1, 1, 1), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
                    desc='divisor_stride_pad'),
        ModuleInput(constructor_input=FunctionInput(4, 2, (1, 2, 1), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
                    desc='divisor_stride_pad_gpu_fixedkw_output'),
        ModuleInput(constructor_input=FunctionInput((2, 4, 8), 1, (1, 1, 2), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 2, 4, 8))),
                    desc='divisor_stride_pad_gpu_general_output'),
```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 767-778
```python
        ModuleInput(constructor_input=FunctionInput(3, 1, 0, divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='divisor_stride1_pad0_gpu_input'),
        ModuleInput(constructor_input=FunctionInput(2, 2, (1, 1, 1), divisor_override=1),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='divisor_stride_pad_gpu_input_nooverlap')]



def module_inputs_torch_nn_AdaptiveAvgPool1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AdaptiveAvgPool1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AdaptiveAvgPool1d`。

### Lines 779-790
```python
    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((1, 3, 5))),
                    desc='single'),
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((3, 5))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(1,),
                    forward_input=FunctionInput(make_input((1, 3, 5))),
                    desc='one_output')]

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 791-812
```python

def module_inputs_torch_nn_AdaptiveAvgPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='single'),
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((3, 5, 6))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(1,),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='single_1x1output'),
        ModuleInput(constructor_input=FunctionInput((3, 4)),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='tuple'),
        ModuleInput(constructor_input=FunctionInput((3, None)),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='tuple_none')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AdaptiveAvgPool2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AdaptiveAvgPool2d`。

### Lines 813-833
```python
def module_inputs_torch_nn_AdaptiveAvgPool3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((2, 3, 5, 2, 7))),
                    desc='single'),
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((3, 5, 2, 7))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput((3, 4, 5)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 3, 7))),
                    desc='tuple'),
        ModuleInput(constructor_input=FunctionInput((None, 4, 5)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 3, 7))),
                    desc='tuple_none'),
        ModuleInput(constructor_input=FunctionInput((3, 2, 2)),
                    forward_input=FunctionInput(make_input((1, 1, 3, 2, 6))),
                    desc='last_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AdaptiveAvgPool3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AdaptiveAvgPool3d`。

### Lines 834-846
```python

def module_inputs_torch_nn_AdaptiveMaxPool1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((1, 3, 5))),
                    desc='single'),
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((3, 5))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AdaptiveMaxPool1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AdaptiveMaxPool1d`。

### Lines 847-865
```python

def module_inputs_torch_nn_AdaptiveMaxPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='single'),
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((3, 5, 6))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput((3, 4)),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='tuple'),
        ModuleInput(constructor_input=FunctionInput((3, None)),
                    forward_input=FunctionInput(make_input((1, 3, 5, 6))),
                    desc='tuple_none')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AdaptiveMaxPool2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AdaptiveMaxPool2d`。

### Lines 866-889
```python

def module_inputs_torch_nn_AdaptiveMaxPool3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((2, 3, 5, 6, 7))),
                    desc='single'),
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((3, 5, 6, 7))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput((3, 4, 5)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 6, 7))),
                    desc='tuple'),
        ModuleInput(constructor_input=FunctionInput((3, None, 5)),
                    forward_input=FunctionInput(make_input((2, 3, 5, 6, 7))),
                    desc='tuple_none'),
        ModuleInput(constructor_input=FunctionInput(3),
                    forward_input=FunctionInput(make_input((2, 3, 12, 9, 3))),
                    desc='single_nonatomic'),
        ModuleInput(constructor_input=FunctionInput((3, 4, 5)),
                    forward_input=FunctionInput(make_input((2, 3, 6, 4, 10))),
                    desc='tuple_nonatomic')]
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_AdaptiveMaxPool3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_AdaptiveMaxPool3d`。

### Lines 890-913
```python


def module_inputs_torch_nn_BatchNorm1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(10,),
                    forward_input=FunctionInput(make_input((4, 10))),
                    desc='affine'),
        ModuleInput(constructor_input=FunctionInput(5,),
                    forward_input=FunctionInput(make_input((4, 5, 3))),
                    desc='3d_input'),
        ModuleInput(constructor_input=FunctionInput(10, 1e-3, None),
                    forward_input=FunctionInput(make_input((4, 10))),
                    desc='affine_simple_average'),
        ModuleInput(constructor_input=FunctionInput(10, 1e-3, 0.3, False),
                    forward_input=FunctionInput(make_input((4, 10))),
                    desc='not_affine'),
        ModuleInput(constructor_input=FunctionInput(10, 1e-3, 0.3, True, False),
                    forward_input=FunctionInput(make_input((4, 10))),
                    desc='not_tracking_stats'),
        ModuleInput(constructor_input=FunctionInput(5, 1e-3, 0.3, False),
                    forward_input=FunctionInput(make_input((4, 5, 3))),
                    desc='3d_input_not_affine'),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_BatchNorm1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_BatchNorm1d`。

### Lines 914-925
```python
        ModuleInput(constructor_input=FunctionInput(5, 1e-3, 0.3, False),
                    forward_input=FunctionInput(make_input((0, 5, 9))),
                    desc='zero_batch'),
        ModuleInput(constructor_input=FunctionInput(10, bias=False),
                    forward_input=FunctionInput(make_input((4, 10))),
                    desc='affine_not_bias'),]


def module_inputs_torch_nn_BatchNorm2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_BatchNorm2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_BatchNorm2d`。

### Lines 926-946
```python
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6)))),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, None),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='2d_simple_average'),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, 0.8),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='momentum'),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, 0.8, False),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='not_affine'),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, 0.8, True, False),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='not_tracking_stats'),
        ModuleInput(constructor_input=FunctionInput(5, 1e-3, 0.3, False),
                    forward_input=FunctionInput(make_input((0, 5, 2, 2))),
                    desc='zero_batch'),
        ModuleInput(constructor_input=FunctionInput(3, bias=False),
                    forward_input=FunctionInput(make_input((2, 3, 6, 6))),
                    desc='affine_not_bias'),]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 947-970
```python

def module_inputs_torch_nn_BatchNorm3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(3,),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4)))),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, None),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='3d_simple_average'),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, 0.7),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='momentum'),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, 0.7, False),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='not_affine'),
        ModuleInput(constructor_input=FunctionInput(3, 1e-3, 0.7, True, False),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
                    desc='not_tracking_stats'),
        ModuleInput(constructor_input=FunctionInput(5, 1e-3, 0.3, False),
                    forward_input=FunctionInput(make_input((0, 5, 2, 2, 2))),
                    desc='zero_batch'),
        ModuleInput(constructor_input=FunctionInput(3, bias=False),
                    forward_input=FunctionInput(make_input((2, 3, 4, 4, 4))),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_BatchNorm3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_BatchNorm3d`。

### Lines 971-983
```python
                    desc='affine_not_bias'),]


def module_error_inputs_torch_nn_BatchNorm1d_2d_3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    if module_info.module_cls == torch.nn.BatchNorm1d:
        input_shape = (2, 10)
    elif module_info.module_cls == torch.nn.BatchNorm2d:
        input_shape = (2, 10, 5, 5)
    else:
        input_shape = (2, 10, 4, 4, 4)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_BatchNorm1d_2d_3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_BatchNorm1d_2d_3d`。

### Lines 984-1004
```python
    return [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, eps=-1.0),
                forward_input=FunctionInput(make_input(input_shape)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex="eps must be positive"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, eps=0.0),
                forward_input=FunctionInput(make_input(input_shape)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex="eps must be positive"
        ),
    ]

```
- EN: This block reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1005-1022
```python

def module_inputs_torch_nn_ConvNd(module_info, device, dtype, requires_grad, training, **kwargs):
    N = kwargs['N']
    lazy = kwargs.get('lazy', False)
    transposed = kwargs.get('transposed', False)
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    conv_kwargs_list = [{}] if transposed else [{}, {'padding': 'same'}]
    kernel_size, C_in, C_out = 3, 4, 5
    input_no_batch_shape = (C_in,) + tuple(i + 3 for i in range(N))
    input_batch_shape = (2,) + input_no_batch_shape
    return [
        ModuleInput(constructor_input=(FunctionInput(C_out, kernel_size, **conv_kwargs) if lazy else
                                       FunctionInput(C_in, C_out, kernel_size, **conv_kwargs)),
                    forward_input=FunctionInput(make_input(
                        input_batch_shape if with_batch else input_no_batch_shape)),
                    desc=('' if with_batch else 'no_batch_dim'),
                    reference_fn=(None if with_batch else no_batch_dim_reference_fn))
        for with_batch, conv_kwargs in itertools.product([True, False], conv_kwargs_list)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ConvNd`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ConvNd`。

### Lines 1023-1037
```python
    ]


def module_inputs_torch_nn_CosineEmbeddingLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('margin', {'margin': 0.7})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CosineEmbeddingLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CosineEmbeddingLoss`。

### Lines 1038-1050
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i1, i2, t, constructor_kwargs=constructor_kwargs):
            return cosineembeddingloss_reference(i1, i2, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((15, 10)), make_input((15, 10)),
                                                    make_target((15,)).sign()),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 1051-1071
```python
    return module_inputs


def module_inputs_torch_nn_ELU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(alpha=2.),
                    forward_input=FunctionInput(make_input((3, 2, 5))),
                    reference_fn=lambda m, p, i: torch.where(i >= 0, i, 2 * (i.exp() - 1))),
        ModuleInput(constructor_input=FunctionInput(alpha=2.),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3,))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn),
        ModuleInput(constructor_input=FunctionInput(alpha=2.),
                    forward_input=FunctionInput(make_input((2, 3, 2, 5))),
                    desc='4d_input')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ELU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ELU`。

### Lines 1072-1088
```python

def module_inputs_torch_nn_CELU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(alpha=2.),
                    forward_input=FunctionInput(make_input((3, 2, 5))),
                    reference_fn=lambda m, p, i: torch.where(i >= 0, i, 2. * ((.5 * i).exp() - 1))),
        ModuleInput(constructor_input=FunctionInput(alpha=2.),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, i: torch.where(i >= 0, i, 2. * ((.5 * i).exp() - 1)),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(alpha=2.),
                    forward_input=FunctionInput(make_input((3,))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn)]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CELU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CELU`。

### Lines 1089-1103
```python

def module_inputs_torch_nn_GLU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((5, 6)))),
        ModuleInput(constructor_input=FunctionInput(1),
                    forward_input=FunctionInput(make_input((5, 6, 7))),
                    desc='dim'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((4,))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn)]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_GLU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_GLU`。

### Lines 1104-1120
```python

def module_inputs_torch_nn_GELU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput('none'),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, x, *_: x * 0.5 * (1.0 + torch.erf(x / math.sqrt(2.0))),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput('none'),
                    forward_input=FunctionInput(make_input((3, 2, 5))),
                    reference_fn=lambda m, p, x, *_: x * 0.5 * (1.0 + torch.erf(x / math.sqrt(2.0)))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3,))),
                    desc='no_batch_dim',
                    reference_fn=no_batch_dim_reference_fn)]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_GELU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_GELU`。

### Lines 1121-1139
```python

def module_inputs_torch_nn_ReLU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5))),
                    desc='channels_last_mem_format'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 3, 4, 5))),
                    desc='channels_last_3d_mem_format')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReLU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReLU`。

### Lines 1140-1158
```python

def module_inputs_torch_nn_ReLU6(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5))),
                    desc='channels_last_mem_format'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 3, 4, 5))),
                    desc='channels_last_3d_mem_format')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReLU6`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReLU6`。

### Lines 1159-1179
```python

def module_inputs_torch_nn_LeakyReLU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3, 2, 5)))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(0.5),
                    forward_input=FunctionInput(make_input((3, 2, 5))),
                    desc='with_negval'),
        ModuleInput(constructor_input=FunctionInput(0.0),
                    forward_input=FunctionInput(make_input((10, 10))),
                    desc='with_zero_negval'),
        ModuleInput(constructor_input=FunctionInput(0.5),
                    forward_input=FunctionInput(make_input(())),
                    desc='with_negval_scalar')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LeakyReLU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LeakyReLU`。

### Lines 1180-1203
```python

def module_inputs_torch_nn_PReLU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4))),
                    reference_fn=lambda m, p, i: torch.clamp(i, min=0) + torch.clamp(i, max=0) * p[0][0],
                    desc='1d'),
        ModuleInput(constructor_input=FunctionInput(3),
                    forward_input=FunctionInput(make_input((2, 3, 4))),
                    reference_fn=lambda m, p, i: torch.clamp(i, min=0) + torch.clamp(i, max=0) * p[0][0],
                    desc='1d_multiparam'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5))),
                    reference_fn=lambda m, p, i: torch.clamp(i, min=0) + torch.clamp(i, max=0) * p[0][0],
                    desc='2d'),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_PReLU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_PReLU`。

### Lines 1204-1216
```python
        ModuleInput(constructor_input=FunctionInput(3),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5))),
                    reference_fn=lambda m, p, i: torch.clamp(i, min=0) + torch.clamp(i, max=0) * p[0][0],
                    desc='2d_multiparam'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5, 6))),
                    reference_fn=lambda m, p, i: torch.clamp(i, min=0) + torch.clamp(i, max=0) * p[0][0],
                    desc='3d'),
        ModuleInput(constructor_input=FunctionInput(3),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5, 6))),
                    reference_fn=lambda m, p, i: torch.clamp(i, min=0) + torch.clamp(i, max=0) * p[0][0],
                    desc='3d_multiparam')]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1217-1231
```python

def module_inputs_torch_nn_SELU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3, 2, 5)))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_SELU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_SELU`。

### Lines 1232-1248
```python

def module_inputs_torch_nn_SiLU(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, x, *_: x * torch.sigmoid(x),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((5, 6, 7))),
                    reference_fn=lambda m, p, x, *_: x * torch.sigmoid(x))]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_SiLU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_SiLU`。

### Lines 1249-1265
```python

def module_inputs_torch_nn_Softmax(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(1),
                    forward_input=FunctionInput(make_input((10, 20))),
                    reference_fn=lambda m, p, i: torch.exp(i).div(torch.exp(i).sum(1, True).expand(10, 20))),
        ModuleInput(constructor_input=FunctionInput(0),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, i: torch.exp(i).div(torch.exp(i).sum(0, True)),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(-1),
                    forward_input=FunctionInput(make_input((4, 5))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Softmax`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Softmax`。

### Lines 1266-1278
```python

def module_inputs_torch_nn_Softmax2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((1, 3, 10, 20))),
                    reference_fn=lambda m, p, i: torch.exp(i).div(torch.exp(i).sum(1, False))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3, 4, 5))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Softmax2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Softmax2d`。

### Lines 1279-1299
```python

def module_inputs_torch_nn_LogSoftmax(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(1),
                    forward_input=FunctionInput(make_input((10, 20))),
                    reference_fn=lambda m, p, i: torch.exp(i).div_(torch.exp(i).sum(1, True).expand(10, 20)).log_()),
        ModuleInput(constructor_input=FunctionInput(1),
                    forward_input=FunctionInput(make_input((1, 3, 10, 20))),
                    reference_fn=lambda m, p, i: torch.exp(i).div_(torch.exp(i).sum(1, False)).log_(),
                    desc='multiparam'),
        ModuleInput(constructor_input=FunctionInput(0),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, i: torch.exp(i).div_(torch.exp(i).sum(0, False)).log_(),
                    desc='multiparam_scalar'),
        ModuleInput(constructor_input=FunctionInput(-1),
                    forward_input=FunctionInput(make_input((4, 5))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LogSoftmax`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LogSoftmax`。

### Lines 1300-1317
```python

def module_inputs_torch_nn_Softmin(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(1),
                    forward_input=FunctionInput(make_input((10, 20)))),
        ModuleInput(constructor_input=FunctionInput(1),
                    forward_input=FunctionInput(make_input((2, 3, 5, 10))),
                    desc='multidim'),
        ModuleInput(constructor_input=FunctionInput(0),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(-1),
                    forward_input=FunctionInput(make_input((3, 4, 10))),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Softmin`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Softmin`。

### Lines 1318-1341
```python

def module_inputs_torch_nn_Softplus(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((10, 20))),
                    reference_fn=lambda m, p, i: torch.log1p(torch.exp(i))),
        ModuleInput(constructor_input=FunctionInput(2),
                    forward_input=FunctionInput(make_input((10, 20))),
                    reference_fn=lambda m, p, i: 1. / 2. * torch.log1p(torch.exp(2 * i)),
                    desc='beta'),
        ModuleInput(constructor_input=FunctionInput(2, -100),
                    forward_input=FunctionInput(make_input((10, 20))),
                    reference_fn=(
                        lambda m, p, i: ((i * 2) > -100).type_as(i) * i
                        + ((i * 2) <= -100).type_as(i) * 1. / 2. * torch.log1p(torch.exp(2 * i))),
                    desc='beta_threshold'),
        ModuleInput(constructor_input=FunctionInput(2, -100),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=(
                        lambda m, p, i: ((i * 2) > -100).type_as(i) * i
                        + ((i * 2) <= -100).type_as(i) * 1. / 2. * torch.log1p(torch.exp(2 * i))),
                    desc='beta_threshold_scalar'),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Softplus`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Softplus`。

### Lines 1342-1364
```python
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]


def module_inputs_torch_nn_Softshrink(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3, 2, 5)))),
        ModuleInput(constructor_input=FunctionInput(1,),
                    forward_input=FunctionInput(make_input((3, 2, 5))),
                    desc='lambda'),
        ModuleInput(constructor_input=FunctionInput(1,),
                    forward_input=FunctionInput(make_input(())),
                    desc='lambda_scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Softshrink`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Softshrink`。

### Lines 1365-1381
```python

def module_inputs_torch_nn_Softsign(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((3, 2, 5))),
                    reference_fn=lambda m, p, i: i.div(1 + torch.abs(i))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, i: i.div(1 + torch.abs(i)),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Softsign`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Softsign`。

### Lines 1382-1396
```python

def module_inputs_torch_nn_Tanh(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5)))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Tanh`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Tanh`。

### Lines 1397-1412
```python


def module_inputs_torch_nn_Tanhshrink(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5)))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Tanhshrink`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Tanhshrink`。

### Lines 1413-1431
```python

def module_inputs_torch_nn_Threshold(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(2., 1.),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5))),
                    desc='threshold_value'),
        ModuleInput(constructor_input=FunctionInput(2., 10.),
                    forward_input=FunctionInput(make_input((2, 3, 4, 5))),
                    desc='large_value'),
        ModuleInput(constructor_input=FunctionInput(2., 1.),
                    forward_input=FunctionInput(make_input(())),
                    desc='threshold_value_scalar'),
        ModuleInput(constructor_input=FunctionInput(2., 1.),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Threshold`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Threshold`。

### Lines 1432-1448
```python

def module_inputs_torch_nn_Mish(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((5, 6, 7))),
                    reference_fn=lambda m, p, i: i * torch.tanh(F.softplus(i))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(())),
                    reference_fn=lambda m, p, i: i * torch.tanh(F.softplus(i)),
                    desc='scalar'),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(4)),
                    reference_fn=no_batch_dim_reference_fn,
                    desc='no_batch_dim')]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Mish`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Mish`。

### Lines 1449-1463
```python

def module_inputs_torch_nn_L1Loss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input((2, 3, 4)),
                                                make_input((2, 3, 4))),
                    reference_fn=lambda m, p, i, t: 1. / i.numel() * sum((a - b).abs().sum()
                                                                         for a, b in zip(i, t, strict=True))),
        ModuleInput(constructor_input=FunctionInput(),
                    forward_input=FunctionInput(make_input(()), make_input(())),
                    reference_fn=lambda m, p, i, t: 1. / i.numel() * (i - t).abs().sum(),
                    desc='scalar')] + generate_regression_criterion_inputs(make_input)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_L1Loss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_L1Loss`。

### Lines 1464-1475
```python

def module_inputs_torch_nn_SmoothL1Loss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)


    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_SmoothL1Loss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_SmoothL1Loss`。

### Lines 1476-1495
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return smoothl1loss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((5, 10)),
                                                    make_input((5, 10))),
                        desc=desc,
                        reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input(()),
                                                    make_input(())),
                        desc=f'scalar_{desc}',
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 1496-1512
```python
    return module_inputs



def module_inputs_torch_nn_BCELoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('weights', {'weight': make_weight((10,))}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_BCELoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_BCELoss`。

### Lines 1513-1524
```python
    def bce_loss_reference_fn(m, p, i, t, reduction='mean', weight=None):
        result = -(t * i.log() + (1 - t) * (1 - i).log())

        if weight is not None:
            result = result * weight

        if reduction == 'none':
            return result
        elif reduction == 'mean':
            return result.sum() / i.numel()
        else:
            return result.sum()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `bce_loss_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`bce_loss_reference_fn`。

### Lines 1525-1544
```python

    module_inputs = []
    for desc, constructor_kwargs in cases:
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((15, 10), low=1e-2, high=1 - 1e-2),
                                                    make_target((15, 10)).gt(0).to(dtype)),
                        desc=desc,
                        reference_fn=partial(bce_loss_reference_fn, **constructor_kwargs))
        )

    scalar_weight = make_weight(())
    module_inputs.append(
        ModuleInput(constructor_input=FunctionInput(weight=scalar_weight),
                    forward_input=FunctionInput(make_input((), low=1e-2, high=1 - 1e-2),
                                                make_target(()).gt(0).to(dtype)),
                    desc='scalar_weight',
                    reference_fn=partial(bce_loss_reference_fn, weight=scalar_weight))
    )

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1545-1561
```python
    return module_inputs


def module_inputs_torch_nn_BCEWithLogitsLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('weights', {'weight': make_weight((10,))}),
        ('scalar_weights', {'weight': make_weight(())})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_BCEWithLogitsLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_BCEWithLogitsLoss`。

### Lines 1562-1573
```python
    def bce_withlogitsloss_reference_fn(m, p, i, t, reduction='mean', weight=None):
        # TODO: add pos_weight to the definition here and corresponding SampleInputs
        max_val = (-i).clamp(min=0)
        result = (1 - t).mul_(i).add_(max_val).add_((-max_val).exp_().add_((-i - max_val).exp_()).log_())

        if weight is not None:
            result = result * weight

        if reduction == 'none':
            return result
        elif reduction == 'mean':
            return result.sum() / i.numel()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `bce_withlogitsloss_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`bce_withlogitsloss_reference_fn`。

### Lines 1574-1586
```python
        else:
            return result.sum()

    module_inputs = []
    for desc, constructor_kwargs in cases:
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((15, 10), low=1e-2, high=1 - 1e-2),
                                                    make_target((15, 10)).gt(0).to(dtype)),
                        desc=desc,
                        reference_fn=partial(bce_withlogitsloss_reference_fn, **constructor_kwargs))
        )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1587-1603
```python
    return module_inputs


def module_inputs_torch_nn_CrossEntropyLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=torch.long, requires_grad=False)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    reductions: list[str] = ['mean', 'sum', 'none']
    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('weights', {'weight': make_weight((3,))}),
        ('ignore_index', {'ignore_index': 1}),
        ('label_smoothing', {'label_smoothing': 0.15}),
        ('ignore_index_label_smoothing', {'ignore_index': 1, 'label_smoothing': 0.15})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CrossEntropyLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CrossEntropyLoss`。

### Lines 1604-1627
```python
    module_inputs = []
    for reduction, (desc, constructor_kwargs) in product(reductions, cases):
        def reference_fn(m, p, i, t, reduction=reduction, constructor_kwargs=constructor_kwargs):
            return cross_entropy_loss_reference(i, t, reduction=reduction, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                        forward_input=FunctionInput(make_input((2, 3, 5, 5)),
                                                    make_target((2, 5, 5), low=0, high=3)),
                        desc=f"4d_{desc}_{reduction}",
                        reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                        forward_input=FunctionInput(make_input((2, 3, 5)),
                                                    make_target((2, 5), low=0, high=3)),
                        desc=f"3d_{desc}_{reduction}",
                        reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                        forward_input=FunctionInput(make_input((2, 3)),
                                                    make_target((2), low=0, high=3)),
                        desc=f"2d_{desc}_{reduction}",
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 1628-1651
```python
                        reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                        forward_input=FunctionInput(make_input((2, 3, 5, 5, 2, 2)),
                                                    make_target((2, 5, 5, 2, 2), low=0, high=3)),
                        desc=f"higher_dim_{desc}_{reduction}",
                        reference_fn=reference_fn)
        )

        if constructor_kwargs.get('ignore_index', None) is None:
            module_inputs.append(
                ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                            forward_input=FunctionInput(make_input((5, 3, 4, 2)),
                                                        make_input((5, 3, 4, 2)).softmax(dim=1)),
                            desc=f"4d_prob_target_{desc}_{reduction}",
                            reference_fn=reference_fn)
            )
            module_inputs.append(
                ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                            forward_input=FunctionInput(make_input((5, 3, 4)),
                                                        make_input((5, 3, 4)).softmax(dim=1)),
                            desc=f"3d_prob_target_{desc}_{reduction}",
                            reference_fn=reference_fn)
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 1652-1674
```python
            )
            module_inputs.append(
                ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                            forward_input=FunctionInput(make_input((5, 3)),
                                                        make_input((5, 3)).softmax(dim=1)),
                            desc=f"2d_prob_target_{desc}_{reduction}",
                            reference_fn=reference_fn)
            )
            module_inputs.append(
                ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                            forward_input=FunctionInput(make_input((2, 3, 5, 5, 2, 2)),
                                                        make_input((2, 3, 5, 5, 2, 2)).softmax(dim=1)),
                            desc=f"higher_dim_prob_target_{desc}_{reduction}",
                            reference_fn=reference_fn)
            )
            module_inputs.append(
                ModuleInput(constructor_input=FunctionInput(reduction=reduction, **constructor_kwargs),
                            forward_input=FunctionInput(make_input((3,)),
                                                        make_target((), low=0, high=3)),
                            desc=f"no_batch_dim_{desc}_{reduction}",
                            reference_fn=partial(no_batch_dim_reference_fn, is_criterion=True))
            )

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1675-1698
```python
    return module_inputs


def module_error_inputs_torch_nn_CrossEntropyLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    return [
        # Non-floating-point target with same shape as input (soft-target path)
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(),
                forward_input=FunctionInput(
                    make_input((2, 3)),
                    torch.zeros((2, 3), device=device, dtype=torch.long),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="Expected floating point type for target with class probabilities",
        ),
        # Non-long target with different shape from input (class-index path)
        ErrorModuleInput(
            ModuleInput(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_CrossEntropyLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_CrossEntropyLoss`。

### Lines 1699-1722
```python
                constructor_input=FunctionInput(),
                forward_input=FunctionInput(
                    make_input((2, 3)),
                    make_input((2,)),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="expected target dtype to be Long or Byte",
        ),
        # Batch size mismatch
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(),
                forward_input=FunctionInput(
                    make_input((2, 3)),
                    torch.tensor([0, 1, 2], device=device, dtype=torch.long),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex="Expected input batch_size .* to match target batch_size",
        ),
        # Weight wrong shape
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1723-1746
```python
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(weight=make_weight((2,))),
                forward_input=FunctionInput(
                    make_input((2, 3)),
                    torch.tensor([0, 1], device=device, dtype=torch.long),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="weight tensor should be defined either for all .* classes or no classes",
        ),
        # Prob target with ignore_index
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(ignore_index=1),
                forward_input=FunctionInput(
                    make_input((2, 3)),
                    make_input((2, 3)).softmax(dim=1).detach(),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="ignore_index is not supported for floating point target",
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1747-1770
```python
        ),
        # label_smoothing out of range
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(label_smoothing=1.5),
                forward_input=FunctionInput(
                    make_input((2, 3)),
                    torch.tensor([0, 1], device=device, dtype=torch.long),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="label_smoothing must be between 0.0 and 1.0",
        ),
        # Target spatial size mismatch (3D input)
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(),
                forward_input=FunctionInput(
                    make_input((2, 3, 5)),
                    torch.zeros((2, 4), device=device, dtype=torch.long),
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1771-1789
```python
            error_type=RuntimeError,
            error_regex="Expected target size",
        ),
    ]


def module_inputs_torch_nn_CTCLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('blank', {'blank': 14})
    ]
    target_dtypes = [torch.int, torch.long]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CTCLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CTCLoss`。

### Lines 1790-1813
```python
    module_inputs = []
    for target_dtype, (desc, constructor_kwargs) in product(target_dtypes, cases):
        def reference_fn(m, p, i, t, il, tl, constructor_kwargs=constructor_kwargs):
            return ctcloss_reference(i, t, il, tl, **constructor_kwargs)

        blank = constructor_kwargs.get('blank', 0)
        low = 0 if blank == 14 else 1
        high = 14 if blank == 14 else 15

        module_inputs.append(
            ModuleInput(
                constructor_input=FunctionInput(**constructor_kwargs),
                forward_input=FunctionInput(make_input((50, 3, 15)).log_softmax(2),
                                            make_target((3, 30), dtype=target_dtype, low=low, high=high),
                                            (50, 50, 50), (30, 25, 20)),
                desc=f'{desc}_lengths_intlists',
                reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(
                constructor_input=FunctionInput(**constructor_kwargs),
                forward_input=FunctionInput(make_input((50, 3, 15)).log_softmax(2),
                                            make_target((3, 30), dtype=target_dtype, low=low, high=high),
                                            torch.tensor((50, 50, 50), device=device),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 1814-1837
```python
                                            torch.tensor((30, 25, 20), device=device)),
                desc=f'{desc}_lengths_tensors',
                reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(
                constructor_input=FunctionInput(**constructor_kwargs),
                forward_input=FunctionInput(make_input((50, 3, 15)).log_softmax(2),
                                            make_target((30 + 25 + 20,), dtype=target_dtype, low=low, high=high),
                                            (50, 50, 50), (30, 25, 20)),
                desc=f'{desc}_1d_target_lengths_intlists',
                reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(
                constructor_input=FunctionInput(**constructor_kwargs),
                forward_input=FunctionInput(make_input((50, 3, 15)).log_softmax(2),
                                            make_target((30 + 25 + 20,), dtype=target_dtype, low=low, high=high),
                                            torch.tensor((50, 50, 50), device=device),
                                            torch.tensor((30, 25, 20), device=device)),
                desc=f'{desc}_1d_target_lengths_tensors',
                reference_fn=reference_fn)
        )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1838-1861
```python
    return module_inputs


def module_inputs_torch_nn_GroupNorm(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(3, 6, 1e-3),
            forward_input=FunctionInput(make_input((4, 6, 5))),
            desc='1d_affine'),
        ModuleInput(
            constructor_input=FunctionInput(3, 6, 1e-3, bias=False),
            forward_input=FunctionInput(make_input((4, 6, 5))),
            desc='1d_affine_not_bias'),
        ModuleInput(
            constructor_input=FunctionInput(3, 12, 1e-3),
            forward_input=FunctionInput(make_input((4, 12))),
            desc='1d_affine_GN'),
        ModuleInput(
            constructor_input=FunctionInput(1, 6, 1e-3),
            forward_input=FunctionInput(make_input((150, 6))),
            desc='1d_affine_large_batch'),
        ModuleInput(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_GroupNorm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_GroupNorm`。

### Lines 1862-1885
```python
            constructor_input=FunctionInput(5, 5, 1e-3, False),
            forward_input=FunctionInput(make_input((4, 5, 5))),
            desc='1d_no_affine_IN'),
        ModuleInput(
            constructor_input=FunctionInput(1, 10, 1e-3, False),
            forward_input=FunctionInput(make_input((4, 10))),
            desc='1d_no_affine_LN'),
        ModuleInput(
            constructor_input=FunctionInput(3, 6, 1e-3),
            forward_input=FunctionInput(make_input((4, 6, 2, 3))),
            desc='2d_affine'),
        ModuleInput(
            constructor_input=FunctionInput(3, 9, 1e-3, bias=False),
            forward_input=FunctionInput(make_input((4, 9, 2, 3))),
            desc='2d_affine_not_bias'),
        ModuleInput(
            constructor_input=FunctionInput(3, 3, 1e-3, False),
            forward_input=FunctionInput(make_input((4, 3, 2, 3))),
            desc='2d_no_affine_IN'),
        ModuleInput(
            constructor_input=FunctionInput(1, 3, 1e-3, False),
            forward_input=FunctionInput(make_input((4, 3, 2, 3))),
            desc='2d_no_affine_LN'),]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1886-1909
```python

def module_error_inputs_torch_nn_GroupNorm(module_info, device, dtype, requires_grad, training, **kwargs):
    """
    Error inputs for GroupNorm that test error messages include actual values.
    """
    return [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(3, 10),  # num_groups=3, num_channels=10
                forward_input=FunctionInput(),  # Not needed for construction error
            ),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=ValueError,
            error_regex=r"num_channels \(10\) must be divisible by num_groups \(3\)"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(5, 13),  # num_groups=5, num_channels=13
                forward_input=FunctionInput(),  # Not needed for construction error
            ),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=ValueError,
            error_regex=r"num_channels \(13\) must be divisible by num_groups \(5\)"
        ),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_GroupNorm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_GroupNorm`。

### Lines 1910-1933
```python
    ]


def module_inputs_torch_nn_Hardshrink(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(2.),
            forward_input=FunctionInput(make_input((4, 3, 2, 4))),
        ),
        ModuleInput(
            constructor_input=FunctionInput(2.),
            forward_input=FunctionInput(make_input(())),
            desc='scalar',
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(4)),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim',
        )
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Hardshrink`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Hardshrink`。

### Lines 1934-1950
```python

def module_inputs_torch_nn_Hardswish(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(4)),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim',
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input((2, 3, 2, 5))),
            desc='4d_input')
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Hardswish`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Hardswish`。

### Lines 1951-1974
```python

def module_inputs_torch_nn_Hardtanh(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input((3, 2, 5))),
            reference_fn=lambda m, p, i: i.clamp(-1, 1),
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(())),
            reference_fn=lambda m, p, i: i.clamp(-1, 1),
            desc='scalar',
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(4)),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim',
        )
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Hardtanh`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Hardtanh`。

### Lines 1975-1987
```python

def module_inputs_torch_nn_HingeEmbeddingLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('margin', {'margin': 0.5})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_HingeEmbeddingLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_HingeEmbeddingLoss`。

### Lines 1988-2007
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return hingeembeddingloss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((10,)),
                                                    make_target((10,)).gt(0).to(dtype).mul_(2).sub_(1)),
                        desc=desc,
                        reference_fn=reference_fn)
        )
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input(()),
                                                    make_target(()).gt(0).to(dtype).mul_(2).sub_(1)),
                        desc=f'scalar_{desc}',
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 2008-2020
```python
    return module_inputs


def module_inputs_torch_nn_HuberLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_HuberLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_HuberLoss`。

### Lines 2021-2033
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return huberloss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((5, 10)),
                                                    make_input((5, 10))),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 2034-2045
```python
    return module_inputs


def module_inputs_torch_nn_InstanceNormNd(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    lazy = kwargs.get('lazy', False)
    N = kwargs['N']
    num_features, eps, momentum, affine, track_running_stats = 3, 1e-3, 0.3, False, True
    input_no_batch_shape_dict = {1: (3, 15), 2: (3, 6, 6), 3: (3, 4, 4, 4)}
    input_no_batch_shape = input_no_batch_shape_dict[N]
    input_batch_shape = (4,) + input_no_batch_shape

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_InstanceNormNd`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_InstanceNormNd`。

### Lines 2046-2069
```python
    return [
        ModuleInput(
            constructor_input=(
                FunctionInput(eps, momentum) if lazy else FunctionInput(num_features, eps, momentum)
            ),
            forward_input=FunctionInput(make_input(input_batch_shape))),
        ModuleInput(
            constructor_input=(
                FunctionInput(eps, momentum, affine, track_running_stats) if lazy else
                FunctionInput(num_features, eps, momentum, affine, track_running_stats)
            ),
            forward_input=FunctionInput(make_input(input_batch_shape)),
            desc='tracking_stats'),
        ModuleInput(
            constructor_input=(
                FunctionInput(eps, momentum) if lazy else FunctionInput(num_features, eps, momentum)
            ),
            forward_input=FunctionInput(make_input(input_no_batch_shape)),
            reference_fn=no_batch_dim_reference_fn,
            desc='tracking_stats_no_batch_dim'),
        ModuleInput(
            constructor_input=(
                FunctionInput(eps, momentum, affine, track_running_stats) if lazy else
                FunctionInput(num_features, eps, momentum, affine, track_running_stats)
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 2070-2088
```python
            ),
            forward_input=FunctionInput(make_input(input_no_batch_shape)),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim'),
        ModuleInput(
            constructor_input=(
                FunctionInput(eps, momentum, affine=True) if lazy else
                FunctionInput(num_features, eps, momentum, affine=True)
            ),
            forward_input=FunctionInput(make_input(input_batch_shape)),
            desc='affine'),
        ModuleInput(
            constructor_input=(
                FunctionInput(eps, momentum, affine=True, bias=False) if lazy else
                FunctionInput(num_features, eps, momentum, affine=True, bias=False)
            ),
            forward_input=FunctionInput(make_input(input_batch_shape)),
            desc='affine_not_bias'),]

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 2089-2112
```python
def module_inputs_torch_nn_LayerNorm(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3),
            forward_input=FunctionInput(make_input((4, 5, 5))),
            desc='1d_elementwise_affine'),
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3),
            forward_input=FunctionInput(make_input((128, 5, 5))),
            desc='1d_elementwise_affine_large_batch'),
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3, False),
            forward_input=FunctionInput(make_input((4, 5, 5))),
            desc='1d_no_elementwise_affine'),
        ModuleInput(
            constructor_input=FunctionInput([2, 2, 5], 1e-3),
            forward_input=FunctionInput(make_input((4, 2, 2, 5))),
            desc='3d_elementwise_affine'),
        ModuleInput(
            constructor_input=FunctionInput([2, 2, 5], 1e-3, False),
            forward_input=FunctionInput(make_input((4, 2, 2, 5))),
            desc='3d_no_elementwise_affine'),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LayerNorm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LayerNorm`。

### Lines 2113-2125
```python
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3),
            forward_input=FunctionInput(make_input((0, 5))),
            desc='1d_empty_elementwise_affine'),
        ModuleInput(
            constructor_input=FunctionInput([2, 2, 5], 1e-3, elementwise_affine=True, bias=False),
            forward_input=FunctionInput(make_input((4, 2, 2, 5))),
            desc='3d_elementwise_affine_no_bias'),
    ]

def module_inputs_torch_nn_RMSNorm(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_RMSNorm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_RMSNorm`。

### Lines 2126-2138
```python
    def rms_norm_reference_fn(m, p, i):
        eps = m.eps
        if eps is None:
            eps = torch.finfo(i.dtype).eps
        ndim = i.ndim
        normalized_shape = m.normalized_shape
        weight = m.weight
        dims = [ndim - i - 1 for i in range(len(normalized_shape))]
        upcasted_i = i.float()
        result = upcasted_i * torch.rsqrt(upcasted_i.pow(2).mean(dim=dims, keepdim=True) + m.eps)
        if weight is not None:
            result *= weight
        return result.type_as(i)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rms_norm_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rms_norm_reference_fn`。

### Lines 2139-2162
```python

    return [
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3),
            forward_input=FunctionInput(make_input((4, 5, 5))),
            desc='1d_elementwise_affine',
            reference_fn=rms_norm_reference_fn),
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3),
            forward_input=FunctionInput(make_input((128, 5, 5))),
            desc='1d_elementwise_affine_large_batch',
            reference_fn=rms_norm_reference_fn),
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3, False),
            forward_input=FunctionInput(make_input((4, 5, 5))),
            desc='1d_no_elementwise_affine',
            reference_fn=rms_norm_reference_fn),
        ModuleInput(
            constructor_input=FunctionInput([2, 2, 5], 1e-3),
            forward_input=FunctionInput(make_input((4, 2, 2, 5))),
            desc='3d_elementwise_affine',
            reference_fn=rms_norm_reference_fn),
        ModuleInput(
            constructor_input=FunctionInput([2, 2, 5], 1e-3, False),
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 2163-2174
```python
            forward_input=FunctionInput(make_input((4, 2, 2, 5))),
            desc='3d_no_elementwise_affine',
            reference_fn=rms_norm_reference_fn),
        ModuleInput(
            constructor_input=FunctionInput([5], 1e-3),
            forward_input=FunctionInput(make_input((0, 5))),
            desc='1d_empty_elementwise_affine',
            reference_fn=rms_norm_reference_fn),
    ]


def module_inputs_torch_nn_LocalResponseNorm(module_info, device, dtype, requires_grad, training, **kwargs):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LocalResponseNorm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LocalResponseNorm`。

### Lines 2175-2191
```python
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(3,),
            forward_input=FunctionInput(make_input((1, 5, 7))),
            desc='1d'),
        ModuleInput(
            constructor_input=FunctionInput(2,),
            forward_input=FunctionInput(make_input((1, 5, 7, 7))),
            desc='2d_uneven_pad'),
        ModuleInput(
            constructor_input=FunctionInput(1, 1., 0.5, 2.),
            forward_input=FunctionInput(make_input((1, 5, 7, 7, 7))),
            desc='3d_custom_params'),
    ]

```
- EN: This block handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 2192-2210
```python

def module_inputs_torch_nn_LPPool1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1.5, 2),
            forward_input=FunctionInput(make_input((1, 3, 7))),
            desc='norm'),
        ModuleInput(
            constructor_input=FunctionInput(2, 2, 3),
            forward_input=FunctionInput(make_input((1, 3, 7)))),
        ModuleInput(
            constructor_input=FunctionInput(2, 2, 3),
            forward_input=FunctionInput(make_input((3, 7))),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim'),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LPPool1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LPPool1d`。

### Lines 2211-2230
```python


def module_inputs_torch_nn_LPPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(2, 2, 2),
            forward_input=FunctionInput(make_input((1, 3, 7, 7)))),
        ModuleInput(
            constructor_input=FunctionInput(2, 2, 2),
            forward_input=FunctionInput(make_input((3, 7, 7))),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim'),
        ModuleInput(
            constructor_input=FunctionInput(1.5, 2),
            forward_input=FunctionInput(make_input((1, 3, 7, 7))),
            desc='norm'),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LPPool2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LPPool2d`。

### Lines 2231-2249
```python

def module_inputs_torch_nn_LPPool3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(2, 2, 2),
            forward_input=FunctionInput(make_input((1, 3, 7, 7, 7)))),
        ModuleInput(
            constructor_input=FunctionInput(2, 2, 2),
            forward_input=FunctionInput(make_input((3, 7, 7, 7))),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim'),
        ModuleInput(
            constructor_input=FunctionInput(1.5, 2),
            forward_input=FunctionInput(make_input((1, 3, 7, 7, 7))),
            desc='norm'),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LPPool3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LPPool3d`。

### Lines 2250-2268
```python

def module_inputs_torch_nn_MaxPool1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(4),
            forward_input=FunctionInput(make_input((2, 10, 4))),
            desc='3d_input'),
        ModuleInput(
            constructor_input=FunctionInput(4, 4),
            forward_input=FunctionInput(make_input((2, 10, 4))),
            desc='stride'),
        ModuleInput(
            constructor_input=FunctionInput(4, return_indices=True),
            forward_input=FunctionInput(make_input((2, 10, 4))),
            desc='return_indices'),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MaxPool1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MaxPool1d`。

### Lines 2269-2287
```python

def module_inputs_torch_nn_MaxPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput((3, 3), (2, 2), (1, 1)),
            forward_input=FunctionInput(make_input((3, 7, 7))),
            desc='3d_input'),
        ModuleInput(
            constructor_input=FunctionInput((3, 3), (2, 2), (1, 1)),
            forward_input=FunctionInput(make_input((1, 3, 7, 7))),
            desc='4d_input'),
        ModuleInput(
            constructor_input=FunctionInput((3, 3), (2, 2), (1, 1), return_indices=True),
            forward_input=FunctionInput(make_input((1, 3, 7, 7))),
            desc='return_indices'),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MaxPool2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MaxPool2d`。

### Lines 2288-2311
```python

def module_error_inputs_torch_nn_MaxPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    """
    Error inputs for MaxPool2d that test error messages for invalid inputs.
    """
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        # Wrong input dimensions: 2D input instead of 3D/4D
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(2),
                forward_input=FunctionInput(make_input((3, 4))),  # 2D input
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex=r"non-empty 3D or 4D \(batch mode\) tensor expected for input"
        ),
        # Wrong input dimensions: 5D input
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(2),
                forward_input=FunctionInput(make_input((1, 2, 3, 4, 5))),  # 5D input
            ),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_MaxPool2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_MaxPool2d`。

### Lines 2312-2327
```python
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex=r"non-empty 3D or 4D \(batch mode\) tensor expected for input"
        ),
        # Invalid padding: padding > kernel_size / 2
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(3, padding=5),  # kernel=3, pad=5 > 3/2
                forward_input=FunctionInput(make_input((1, 1, 10, 10))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex=r"pad should be at most half of effective kernel size"
        ),
    ]

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 2328-2349
```python

def module_inputs_torch_nn_MaxPool3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput((2, 2, 2)),
            forward_input=FunctionInput(make_input((2, 3, 5, 5, 5)))),
        ModuleInput(
            constructor_input=FunctionInput(2, (2, 2, 2)),
            forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
            desc='stride'),
        ModuleInput(
            constructor_input=FunctionInput(2, 2, (1, 1, 1)),
            forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
            desc='stride_padding'),
        ModuleInput(
            constructor_input=FunctionInput(2, 2, (1, 1, 1), return_indices=True),
            forward_input=FunctionInput(make_input((2, 3, 5, 5, 5))),
            desc='return_indices'),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MaxPool3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MaxPool3d`。

### Lines 2350-2373
```python

def module_inputs_torch_nn_FractionalMaxPool2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    def make_random_samples():
        return torch.empty((1, 3, 2), dtype=torch.double, device=device).uniform_()

    return [
        ModuleInput(
            constructor_input=FunctionInput(2, output_ratio=0.5, _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((1, 3, 5, 7))),
            desc='ratio'),
        ModuleInput(
            constructor_input=FunctionInput((2, 3), output_size=(4, 3), _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((1, 3, 7, 6))),
            desc='size'),
        ModuleInput(
            constructor_input=FunctionInput(
                2, output_ratio=0.5, _random_samples=make_random_samples(), return_indices=True
            ),
            forward_input=FunctionInput(make_input((1, 3, 5, 7))),
            desc='ratio_return_indices'),
        ModuleInput(
            constructor_input=FunctionInput(2, output_ratio=0.5, _random_samples=make_random_samples()),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_FractionalMaxPool2d`, `make_random_samples`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_FractionalMaxPool2d`, `make_random_samples`。

### Lines 2374-2385
```python
            forward_input=FunctionInput(make_input((3, 5, 7))),
            reference_fn=no_batch_dim_reference_fn,
            desc='ratio_no_batch_dim'),
        ModuleInput(
            constructor_input=FunctionInput((2, 3), output_size=(4, 3), _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((3, 7, 6))),
            reference_fn=no_batch_dim_reference_fn,
            desc='size_no_batch_dim'),
    ]


def module_inputs_torch_nn_FractionalMaxPool3d(module_info, device, dtype, requires_grad, training, **kwargs):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_FractionalMaxPool3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_FractionalMaxPool3d`。

### Lines 2386-2409
```python
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    def make_random_samples():
        return torch.empty((2, 4, 3), dtype=torch.double, device=device).uniform_()

    return [
        ModuleInput(
            constructor_input=FunctionInput(2, output_ratio=0.5, _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((2, 4, 5, 5, 5))),
            desc='ratio'),
        ModuleInput(
            constructor_input=FunctionInput((2, 2, 2), output_size=(4, 4, 4), _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((2, 4, 7, 7, 7))),
            desc='size'),
        ModuleInput(
            constructor_input=FunctionInput((4, 2, 3), output_size=(10, 3, 2), _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((2, 4, 16, 7, 5))),
            desc='asymsize'),
        ModuleInput(
            constructor_input=FunctionInput(
                2, output_ratio=0.5, _random_samples=make_random_samples(), return_indices=True
            ),
            forward_input=FunctionInput(make_input((2, 4, 5, 5, 5))),
            desc='ratio_return_indices'),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `make_random_samples`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`make_random_samples`。

### Lines 2410-2421
```python
        ModuleInput(
            constructor_input=FunctionInput(2, output_ratio=0.5, _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((4, 5, 5, 5))),
            reference_fn=no_batch_dim_reference_fn,
            desc='ratio_no_batch_dim'),
        ModuleInput(
            constructor_input=FunctionInput((2, 2, 2), output_size=(4, 4, 4), _random_samples=make_random_samples()),
            forward_input=FunctionInput(make_input((4, 7, 7, 7))),
            reference_fn=no_batch_dim_reference_fn,
            desc='size_no_batch_dim'),
    ]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2422-2445
```python

def module_inputs_torch_nn_Sigmoid(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(())),
            desc='scalar'
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(4)),
            reference_fn=no_batch_dim_reference_fn,
            desc='no_batch_dim',
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input((2, 3, 4, 5))),
            desc='channels_last_mem_format'
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input((2, 3, 3, 4, 5))),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Sigmoid`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Sigmoid`。

### Lines 2446-2469
```python
            desc='channels_last_3d_mem_format'
        )
    ]


def module_inputs_torch_nn_LogSigmoid(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(())),
            reference_fn=lambda m, p, i: i.sigmoid().log(),
            desc='scalar'
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input((2, 3, 4))),
            reference_fn=lambda m, p, i: i.sigmoid().log(),
        ),
        ModuleInput(
            constructor_input=FunctionInput(),
            forward_input=FunctionInput(make_input(4)),
            reference_fn=no_batch_dim_reference_fn,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LogSigmoid`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LogSigmoid`。

### Lines 2470-2486
```python
            desc='no_batch_dim',
        ),
    ]


def module_inputs_torch_nn_MarginRankingLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=torch.long, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('margin', {'margin': 0.5})
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MarginRankingLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MarginRankingLoss`。

### Lines 2487-2499
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i1, i2, t, constructor_kwargs=constructor_kwargs):
            return marginrankingloss_reference(i1, i2, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((50,)), make_input((50,)),
                                                    make_target((50,)).sign()),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 2500-2513
```python
    return module_inputs


def module_inputs_torch_nn_MultiLabelMarginLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=torch.long, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MultiLabelMarginLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MultiLabelMarginLoss`。

### Lines 2514-2526
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return multilabelmarginloss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((10,)),
                                                    make_target((10), low=0, high=10)),
                        desc=f'1d_{desc}',
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 2527-2538
```python
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((5, 10)),
                                                    make_target((5, 10), low=0, high=10)),
                        desc=desc,
                        reference_fn=reference_fn)
        )

    return module_inputs


def module_inputs_torch_nn_MultiMarginLoss(module_info, device, dtype, requires_grad, training, **kwargs):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MultiMarginLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MultiMarginLoss`。

### Lines 2539-2552
```python
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=torch.long, requires_grad=False)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('p', {'p': 2}),
        ('margin', {'margin': 0.5}),
        ('weights', {'weight': make_weight(10)})
    ]

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 2553-2565
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return multimarginloss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((5, 10)),
                                                    make_target((5), low=0, high=10)),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 2566-2577
```python
    return module_inputs


def module_error_inputs_torch_nn_MultiMarginLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    """
    Error inputs for MultiMarginLoss that test the improved error message
    for inconsistent target size.
    Regression test for issue #106251.
    """
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_MultiMarginLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_MultiMarginLoss`。

### Lines 2578-2592
```python
        # Test: target size doesn't match batch size (5 vs 3)
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(),
                forward_input=FunctionInput(
                    make_input((5, 10)),  # input: batch_size=5, num_classes=10
                    torch.tensor([0, 1, 2], device=device, dtype=torch.long),  # target: wrong size (3)
                ),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex=r"target tensor should be 1-D with size equal to.*Expected target size \[5\].*but got \[3\]"
        ),
    ]

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 2593-2606
```python

def module_inputs_torch_nn_MultiLabelSoftMarginLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=torch.long, requires_grad=False)
    make_weight = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
        ('weight', {'weight': make_weight(10)}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MultiLabelSoftMarginLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MultiLabelSoftMarginLoss`。

### Lines 2607-2618
```python
    def multilabelsoftmargin_loss_reference_fn(m, p, i, t, reduction='mean', weight=None):
        result = t * i.sigmoid().log() + (1 - t) * (-i).sigmoid().log()
        if weight is not None:
            result *= weight
        result = (-result).sum(i.dim() - 1) / i.size(-1)

        if reduction == 'none':
            return result
        elif reduction == 'mean':
            return result.mean()
        else:
            return result.sum()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `multilabelsoftmargin_loss_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`multilabelsoftmargin_loss_reference_fn`。

### Lines 2619-2630
```python

    module_inputs = []
    for desc, constructor_kwargs in cases:
        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((5, 10)),
                                                    make_target((5, 10), low=0, high=2)),
                        desc=desc,
                        reference_fn=partial(multilabelsoftmargin_loss_reference_fn, **constructor_kwargs))
        )

    return module_inputs
```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 2631-2643
```python


def module_inputs_torch_nn_SoftMarginLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    make_target = partial(make_tensor, device=device, dtype=dtype, requires_grad=False)

    cases: list[tuple[str, dict]] = [
        ('', {}),
        ('reduction_sum', {'reduction': 'sum'}),
        ('reduction_mean', {'reduction': 'mean'}),
        ('reduction_none', {'reduction': 'none'}),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_SoftMarginLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_SoftMarginLoss`。

### Lines 2644-2656
```python
    module_inputs = []
    for desc, constructor_kwargs in cases:
        def reference_fn(m, p, i, t, constructor_kwargs=constructor_kwargs):
            return softmarginloss_reference(i, t, **constructor_kwargs)

        module_inputs.append(
            ModuleInput(constructor_input=FunctionInput(**constructor_kwargs),
                        forward_input=FunctionInput(make_input((5, 5)),
                                                    make_target((5, 5)).sign()),
                        desc=desc,
                        reference_fn=reference_fn)
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_fn`。

### Lines 2657-2675
```python
    return module_inputs


def module_inputs_torch_nn_TransformerEncoder(module_info, device, dtype, requires_grad, training, **kwargs):
    # Reuse the TransformerEncoderLayer samples since the forward args are nearly the same.
    samples = []
    for layer_module_input in module_inputs_torch_nn_TransformerEncoderLayer(
            None, device, dtype, requires_grad, training):
        # Construct a TransformerEncoderLayer object to pass to TransformerEncoder.
        l_args, l_kwargs = (layer_module_input.constructor_input.args,
                            layer_module_input.constructor_input.kwargs)
        l_kwargs['device'] = device
        l_kwargs['dtype'] = dtype
        encoder_layer = torch.nn.TransformerEncoderLayer(*l_args, **l_kwargs)
        num_layers = 2
        # Note: TransformerEncoderLayer takes a "src_mask" while
        # TransformerEncoder takes a "mask"; rename kwarg appropriately.
        forward_input = layer_module_input.forward_input
        if 'src_mask' in forward_input.kwargs:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_TransformerEncoder`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_TransformerEncoder`。

### Lines 2676-2687
```python
            forward_input.kwargs['mask'] = forward_input.kwargs['src_mask']
            del forward_input.kwargs['src_mask']
        samples.append(ModuleInput(
            constructor_input=FunctionInput(encoder_layer, num_layers),
            forward_input=forward_input,
            desc=layer_module_input.desc
        ))
    return samples

def module_inputs_torch_nn_TransformerEncoderLayer(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_TransformerEncoderLayer`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_TransformerEncoderLayer`。

### Lines 2688-2710
```python
    samples = [
        ModuleInput(
            constructor_input=FunctionInput(4, 2, 16, 0.0),
            forward_input=FunctionInput(
                make_input((2, 3, 4))
            ),
            desc='relu_activation'
        ),
        ModuleInput(
            constructor_input=FunctionInput(4, 2, 8, 0.0, F.gelu),
            forward_input=FunctionInput(
                make_input((2, 3, 4))
            ),
            desc='gelu_activation'
        ),
        ModuleInput(
            constructor_input=FunctionInput(4, 2, 8, 0.0, bias=False),
            forward_input=FunctionInput(
                make_input((2, 3, 4))
            ),
            desc='no_bias'
        ), ]

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2711-2728
```python
    # Samples below are for validating the no-batch-dim support.
    key_padding_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool))
    attn_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool).expand((3, 3)))
    for src_mask, src_key_padding_mask, norm_first, batch_first, bias in \
            itertools.product(attn_masks, key_padding_masks, (True, False), (True, False), (True, False)):
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(d_model=4, nhead=2, dim_feedforward=8,
                                                dropout=0.0, batch_first=batch_first,
                                                norm_first=norm_first, bias=bias),
                forward_input=FunctionInput(
                    make_input((3, 4)), src_mask=src_mask, src_key_padding_mask=src_key_padding_mask
                ),
                reference_fn=partial(no_batch_dim_reference_fn,
                                     batch_first=batch_first, kwargs_to_batchify={'src_key_padding_mask': 0}),
                desc=f'no_batch_dim_batch_first_{batch_first}'
            ))

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 2729-2740
```python
    # Samples below where we pass reference_fn are for validating the fast path,
    # since the fast path requires no_grad mode, we run the fast path in .eval()
    # and no_grad() in the reference_fn and verify that against the results in train mode.
    def fast_path_reference_fn(module, parameters, *args, **kwargs):
        if not module.training:
            raise AssertionError("Expected module.training to be True")
        module.train(False)
        with torch.no_grad():
            output = module(*args, **kwargs)
        module.train(True)
        return output

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `fast_path_reference_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`fast_path_reference_fn`。

### Lines 2741-2756
```python
    if training:
        for norm_first, bias in itertools.product((True, False), (True, False)):
            samples.append(
                ModuleInput(
                    constructor_input=FunctionInput(
                        4, 2, 8, dropout=0.0, batch_first=True, norm_first=norm_first, bias=bias
                    ),
                    forward_input=FunctionInput(
                        make_input((2, 3, 4)),
                    ),
                    # fastpath doesn't run when bias=False
                    reference_fn=fast_path_reference_fn if bias else None,
                    desc=f'fastpath_{bias}_norm_first_{norm_first}'
                )
            )

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 2757-2780
```python
    return samples


def module_inputs_torch_nn_TransformerDecoderLayer(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    samples = [
        ModuleInput(
            constructor_input=FunctionInput(4, 2, 16, 0.0),
            forward_input=FunctionInput(
                make_input((2, 3, 4)), make_input((2, 3, 4))
            ),
            desc='relu_activation'
        ),
        ModuleInput(
            constructor_input=FunctionInput(4, 2, 8, 0.0, F.gelu),
            forward_input=FunctionInput(
                make_input((2, 3, 4)), make_input((2, 3, 4))
            ),
            desc='gelu_activation'
        ),
        ModuleInput(
            constructor_input=FunctionInput(4, 2, 8, 0.0, bias=False),
            forward_input=FunctionInput(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_TransformerDecoderLayer`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_TransformerDecoderLayer`。

### Lines 2781-2804
```python
                make_input((2, 3, 4)), make_input((2, 3, 4))
            ),
            desc='no_bias'
        ), ]

    key_padding_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool))
    attn_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool).expand((3, 3)))
    for tgt_mask, tgt_key_padding_mask, norm_first, bias, batch_first in \
            itertools.product(attn_masks, key_padding_masks, (True, False), (True, False), (True, False)):
        # Using same mask for tgt and memory
        memory_mask = tgt_mask
        memory_key_padding_mask = tgt_key_padding_mask
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(d_model=4, nhead=2, dim_feedforward=8,
                                                dropout=0.0, batch_first=batch_first,
                                                norm_first=norm_first, bias=bias),
                forward_input=FunctionInput(
                    make_input((3, 4)), make_input((3, 4)), tgt_mask=tgt_mask, memory_mask=memory_mask,
                    tgt_key_padding_mask=tgt_key_padding_mask, memory_key_padding_mask=memory_key_padding_mask
                ),
                reference_fn=partial(no_batch_dim_reference_fn,
                                     batch_first=batch_first,
                                     kwargs_to_batchify={'tgt_key_padding_mask': 0, 'memory_key_padding_mask': 0}),
```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 2805-2823
```python
                desc=f'no_batch_dim_batch_first_{batch_first}'
            ))
        src, tgt = make_input((2, 3, 4)), make_input((2, 3, 4))
        if not batch_first:
            src, tgt = src.transpose(0, 1), tgt.transpose(0, 1)
        if tgt_key_padding_mask is not None:
            memory_key_padding_mask, tgt_key_padding_mask = (tgt_key_padding_mask.expand(2, 3),) * 2
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(d_model=4, nhead=2, dim_feedforward=8,
                                                dropout=0.0, batch_first=batch_first,
                                                norm_first=norm_first, bias=bias),
                forward_input=FunctionInput(
                    src, tgt, tgt_mask=tgt_mask, memory_mask=memory_mask,
                    tgt_key_padding_mask=tgt_key_padding_mask, memory_key_padding_mask=memory_key_padding_mask
                ),
                desc=f'norm_first_{norm_first}_batch_first_{batch_first}_bias_{bias}'
            ))

```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 2824-2847
```python
    return samples


def module_inputs_torch_nn_Transformer(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    samples = []
    # Samples below are for validating the no-batch-dim support.
    key_padding_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool))
    attn_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool).expand((3, 3)))
    for mask, key_padding_mask, norm_first, bias, batch_first in \
            itertools.product(attn_masks, key_padding_masks, (True, False), (True, False), (True, False)):
        # Using same mask for tgt and memory
        src_mask , tgt_mask = (mask,) * 2
        src_key_padding_mask, tgt_key_padding_mask = (key_padding_mask,) * 2
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(d_model=4, nhead=2, dim_feedforward=8,
                                                num_encoder_layers=1, num_decoder_layers=1,
                                                dropout=0.0, batch_first=batch_first, norm_first=norm_first, bias=bias),
                forward_input=FunctionInput(
                    make_input((3, 4)), make_input((3, 4)), tgt_mask=tgt_mask, src_mask=src_mask,
                    tgt_key_padding_mask=tgt_key_padding_mask, src_key_padding_mask=src_key_padding_mask
                ),
                reference_fn=partial(no_batch_dim_reference_fn,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Transformer`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Transformer`。

### Lines 2848-2859
```python
                                     batch_first=batch_first,
                                     kwargs_to_batchify={'tgt_key_padding_mask': 0, 'src_key_padding_mask': 0}),
                desc=f'no_batch_dim_batch_first_{batch_first}'
            ))

        src, tgt = make_input((2, 3, 4)), make_input((2, 3, 4))
        if not batch_first:
            src = src.transpose(0, 1)
            tgt = tgt.transpose(0, 1)
        if key_padding_mask is not None:
            src_key_padding_mask, tgt_key_padding_mask = (key_padding_mask.expand(2, 3),) * 2

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 2860-2871
```python
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(d_model=4, nhead=2, dim_feedforward=8,
                                                num_encoder_layers=1, num_decoder_layers=1,
                                                dropout=0.0, batch_first=batch_first, norm_first=norm_first, bias=bias),
                forward_input=FunctionInput(
                    src, tgt, tgt_mask=tgt_mask, src_mask=src_mask,
                    tgt_key_padding_mask=tgt_key_padding_mask, src_key_padding_mask=src_key_padding_mask
                ),
            ))
    return samples

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 2872-2886
```python

def module_inputs_torch_nn_Embedding(module_info, device, dtype, requires_grad, training, **kwargs):
    make_empty = partial(torch.empty, device=device, dtype=torch.long, requires_grad=False)
    return [
        ModuleInput(
            constructor_input=FunctionInput(num_embeddings=4, embedding_dim=3),
            forward_input=FunctionInput(make_empty(2, 3).random_(4))
        ),
        ModuleInput(
            constructor_input=FunctionInput(num_embeddings=4, embedding_dim=3),
            forward_input=FunctionInput(make_empty(1, 512).random_(4).expand(7, 512)),
            desc='discontiguous'
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_Embedding`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_Embedding`。

### Lines 2887-2908
```python

def module_error_inputs_torch_nn_Embedding(module_info, device, dtype, requires_grad, training, **kwargs):
    """
    Error inputs for Embedding that test error messages for invalid inputs.
    """
    samples = []

    # Out of range indices: index exceeds num_embeddings
    # Only test on CPU - CUDA triggers kernel assertion instead of Python exception
    if torch.device(device).type == 'cpu':
        samples.append(
            ErrorModuleInput(
                ModuleInput(
                    constructor_input=FunctionInput(num_embeddings=10, embedding_dim=3),
                    forward_input=FunctionInput(torch.tensor([0, 5, 15], device=device, dtype=torch.long)),
                ),
                error_on=ModuleErrorEnum.FORWARD_ERROR,
                error_type=IndexError,
                error_regex=r"index out of range in self"
            )
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_Embedding`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_Embedding`。

### Lines 2909-2921
```python
    # Float indices: wrong dtype for indices (works on all devices)
    samples.append(
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(num_embeddings=10, embedding_dim=3),
                forward_input=FunctionInput(torch.tensor([1.5, 2.5], device=device, dtype=torch.float32)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex=r"Expected tensor for argument.*indices.*to have.*scalar type.*Long.*Int"
        )
    )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 2922-2934
```python
    # Negative num_embeddings (construction error, device-independent)
    samples.append(
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(num_embeddings=-1, embedding_dim=3),
                forward_input=FunctionInput(),
            ),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=RuntimeError,
            error_regex=r"Trying to create tensor with negative dimension"
        )
    )

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 2935-2947
```python
    return samples



def module_inputs_torch_nn_MultiheadAttention(module_info, device, dtype, requires_grad, training, **kwargs):
    # Currently all samples below are for validating the no-batch-dim support.
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    samples = []
    bool_vals = (True, False)
    key_padding_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool))
    attn_masks = (None, torch.tensor([False, False, True], device=device, dtype=torch.bool).expand((3, 3, 3)))
    products = itertools.product(bool_vals, bool_vals, bool_vals, key_padding_masks, attn_masks)
    for bias, add_bias_kv, add_zero_attn, key_padding_mask, attn_mask in products:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_MultiheadAttention`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_MultiheadAttention`。

### Lines 2948-2966
```python
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(embed_dim=3, num_heads=3, batch_first=True,
                                                bias=bias, add_bias_kv=add_bias_kv, add_zero_attn=add_zero_attn),
                forward_input=FunctionInput(make_input((3, 3)), make_input((3, 3)), make_input((3, 3)),
                                            key_padding_mask=key_padding_mask, attn_mask=attn_mask),
                reference_fn=no_batch_dim_reference_mha,
            )
        )
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(embed_dim=3, num_heads=3, batch_first=False,
                                                bias=bias, add_bias_kv=add_bias_kv, add_zero_attn=add_zero_attn),
                forward_input=FunctionInput(make_input((3, 3)), make_input((3, 3)), make_input((3, 3)),
                                            key_padding_mask=key_padding_mask, attn_mask=attn_mask),
                reference_fn=partial(no_batch_dim_reference_mha, batch_first=False),
            )
        )

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 2967-2985
```python
    return samples


def module_inputs_torch_nn_RNN_GRU_Cell(module_info, device, dtype, requires_grad, training, **kwargs):
    # Currently all samples below are for validating the no-batch-dim support.
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    samples = [
        ModuleInput(
            constructor_input=FunctionInput(5, 10),
            forward_input=FunctionInput(make_input(5), make_input(10)),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput(5, 10, bias=True),
            forward_input=FunctionInput(make_input(5), make_input(10)),
            reference_fn=no_batch_dim_reference_fn,
        )
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_RNN_GRU_Cell`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_RNN_GRU_Cell`。

### Lines 2986-2997
```python
    is_rnn = kwargs.get('is_rnn', False)
    if is_rnn:
        # RNN also supports `nonlinearity` argument.
        # `tanh` is the default, so we check with `relu`
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(5, 10, bias=True, nonlinearity='relu'),
                forward_input=FunctionInput(make_input(5), make_input(10)),
                reference_fn=no_batch_dim_reference_fn,
            )
        )

```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：无明显局部符号。

### Lines 2998-3016
```python
    return samples


def module_inputs_torch_nn_LSTMCell(module_info, device, dtype, requires_grad, training, **kwargs):
    # Currently all samples below are for validating the no-batch-dim support.
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    samples = (
        ModuleInput(
            constructor_input=FunctionInput(5, 10),
            forward_input=FunctionInput(make_input(5), (make_input(10), make_input(10))),
            reference_fn=no_batch_dim_reference_lstmcell,
        ),
        ModuleInput(
            constructor_input=FunctionInput(5, 10, bias=True),
            forward_input=FunctionInput(make_input(5), (make_input(10), make_input(10))),
            reference_fn=no_batch_dim_reference_lstmcell,
        ),
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LSTMCell`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LSTMCell`。

### Lines 3017-3035
```python
    return samples

def make_packed_sequence(inp, batch_sizes):
    required_grad = inp.requires_grad
    inp.requires_grad_(False)  # user won't have access to inp so won't be able to get its grads
    seq = pack_padded_sequence(inp, batch_sizes)
    seq.data.requires_grad_(required_grad)
    return seq


def module_inputs_torch_nn_RNN_GRU(module_info, device, dtype, requires_grad, training, with_packed_sequence=False, **kwargs):
    # Currently all samples below are for validating the no-batch-dim support.
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    is_rnn = kwargs['is_rnn']
    nonlinearity = ('relu', 'tanh')
    bias = (False, True)
    batch_first = (False, True)
    bidirectional = (False, True)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `make_packed_sequence`, `module_inputs_torch_nn_RNN_GRU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`make_packed_sequence`, `module_inputs_torch_nn_RNN_GRU`。

### Lines 3036-3047
```python
    samples = []
    if is_rnn:
        prod_gen = product(nonlinearity, bias, batch_first, bidirectional)
    else:
        prod_gen = product(bias, batch_first, bidirectional)

    for args in prod_gen:
        if is_rnn:
            nl, b, b_f, bidir = args
        else:
            b, b_f, bidir = args

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3048-3070
```python
        cons_args = {'input_size': 2, 'hidden_size': 2, 'num_layers': 2,
                     'batch_first': b_f, 'bias': b, 'bidirectional': bidir}
        cons_args_hidden = {'input_size': 2, 'hidden_size': 3, 'num_layers': 2,
                            'batch_first': b_f, 'bias': b, 'bidirectional': bidir}

        if is_rnn:
            cons_args['nonlinearity'] = nl
            cons_args_hidden['nonlinearity'] = nl
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(**cons_args),
                forward_input=FunctionInput(make_input((3, 2))),
                reference_fn=partial(no_batch_dim_reference_rnn_gru, batch_first=b_f),
            )
        )
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(**cons_args_hidden),
                forward_input=FunctionInput(make_input((3, 2)), make_input((4 if bidir else 2, 3))),
                reference_fn=partial(no_batch_dim_reference_rnn_gru, batch_first=b_f),
            )
        )
        if with_packed_sequence:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 3071-3085
```python
            samples.append(
                ModuleInput(
                    constructor_input=FunctionInput(**cons_args),
                    forward_input=FunctionInput(make_packed_sequence(make_input((5, 2, 2)), torch.tensor([5, 3]))),
                    reference_fn=partial(no_batch_dim_reference_rnn_gru, batch_first=b_f),
                )
            )
            samples.append(
                ModuleInput(
                    constructor_input=FunctionInput(**cons_args),
                    forward_input=FunctionInput(make_packed_sequence(make_input((5, 5, 2)), torch.tensor([5, 3, 3, 2, 2]))),
                    reference_fn=partial(no_batch_dim_reference_rnn_gru, batch_first=b_f),
                )
            )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 3086-3099
```python
    return samples


def module_inputs_torch_nn_LSTM(module_info, device, dtype, requires_grad, training, **kwargs):
    # Currently all samples below are for validating the no-batch-dim support.
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    bias = (False, True)
    batch_first = (False, True)
    bidirectional = (False, True)
    proj_sizes = (0, 2)

    samples = []
    prod_gen = product(bias, batch_first, bidirectional, proj_sizes)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_LSTM`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_LSTM`。

### Lines 3100-3115
```python
    for args in prod_gen:
        b, b_f, bidir, proj_size = args
        hidden_size = 3
        cons_args = {'input_size': 2, 'hidden_size': hidden_size, 'num_layers': 2, 'proj_size': proj_size,
                     'batch_first': b_f, 'bias': b, 'bidirectional': bidir}
        cons_args_hidden = {'input_size': 2, 'hidden_size': hidden_size, 'num_layers': 2, 'proj_size': proj_size,
                            'batch_first': b_f, 'bias': b, 'bidirectional': bidir}

        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(**cons_args),
                forward_input=FunctionInput(make_input((2, 2))),
                reference_fn=partial(no_batch_dim_reference_lstm, batch_first=b_f),
            )
        )

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3116-3127
```python
        h_out = proj_size if proj_size > 0 else hidden_size
        hx = (make_input((4 if bidir else 2, h_out)), make_input((4 if bidir else 2, hidden_size)))
        samples.append(
            ModuleInput(
                constructor_input=FunctionInput(**cons_args_hidden),
                forward_input=FunctionInput(make_input((3, 2)), hx),
                reference_fn=partial(no_batch_dim_reference_lstm, batch_first=b_f),
            )
        )


    return samples
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3128-3145
```python



def module_inputs_torch_nn_ReflectionPad1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((2, 3))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2)),
            forward_input=FunctionInput(make_input((2, 3, 4))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReflectionPad1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReflectionPad1d`。

### Lines 3146-3160
```python
def module_inputs_torch_nn_ReflectionPad2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4, 5))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4)),
            forward_input=FunctionInput(make_input((3, 4, 5, 6))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReflectionPad2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReflectionPad2d`。

### Lines 3161-3175
```python
def module_inputs_torch_nn_ReflectionPad3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((2, 3, 4, 5))),
            reference_fn=no_batch_dim_reference_fn
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 1, 2, 1, 2)),
            forward_input=FunctionInput(make_input((3, 3, 3, 3, 3))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReflectionPad3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReflectionPad3d`。

### Lines 3176-3190
```python
def module_inputs_torch_nn_ReplicationPad1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4))),
            reference_fn=no_batch_dim_reference_fn
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2)),
            forward_input=FunctionInput(make_input((3, 4, 5))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReplicationPad1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReplicationPad1d`。

### Lines 3191-3205
```python
def module_inputs_torch_nn_ReplicationPad2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4, 5))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4)),
            forward_input=FunctionInput(make_input((3, 4, 5, 6))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReplicationPad2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReplicationPad2d`。

### Lines 3206-3220
```python
def module_inputs_torch_nn_ReplicationPad3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4, 5, 6))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4, 5, 6)),
            forward_input=FunctionInput(make_input((3, 4, 5, 6, 7))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ReplicationPad3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ReplicationPad3d`。

### Lines 3221-3235
```python
def module_inputs_torch_nn_ZeroPad1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2)),
            forward_input=FunctionInput(make_input((3, 4, 5))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ZeroPad1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ZeroPad1d`。

### Lines 3236-3250
```python
def module_inputs_torch_nn_ZeroPad2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((1, 2, 3))),
            reference_fn=no_batch_dim_reference_fn
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4)),
            forward_input=FunctionInput(make_input((1, 2, 3, 4))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ZeroPad2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ZeroPad2d`。

### Lines 3251-3265
```python
def module_inputs_torch_nn_ZeroPad3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4, 5, 6))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4, 5, 6)),
            forward_input=FunctionInput(make_input((1, 2, 3, 4, 5))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ZeroPad3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ZeroPad3d`。

### Lines 3266-3280
```python
def module_inputs_torch_nn_ConstantPad1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1, 2),
            forward_input=FunctionInput(make_input((3, 4))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2), 3),
            forward_input=FunctionInput(make_input((3, 4, 5))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ConstantPad1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ConstantPad1d`。

### Lines 3281-3295
```python
def module_inputs_torch_nn_ConstantPad2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1, 3),
            forward_input=FunctionInput(make_input((3, 4, 5))),
            reference_fn=no_batch_dim_reference_fn
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4), 5),
            forward_input=FunctionInput(make_input((1, 2, 3, 4))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ConstantPad2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ConstantPad2d`。

### Lines 3296-3310
```python
def module_inputs_torch_nn_ConstantPad3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    return [
        ModuleInput(
            constructor_input=FunctionInput(1, 3),
            forward_input=FunctionInput(make_input((3, 4, 5, 6))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 3, 4, 5, 6), 7),
            forward_input=FunctionInput(make_input((1, 2, 1, 2, 1))),
        ),
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_ConstantPad3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_ConstantPad3d`。

### Lines 3311-3323
```python
def module_inputs_torch_nn_CircularPad1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    def padding1d_circular_ref(inp, pad):
        r""" input:
                [[[0., 1., 2.],
                  [3., 4., 5.]]]
                pad: (1, 2)
                output:
                    [[[2., 0., 1., 2., 0., 1.],
                      [5., 3., 4., 5., 3., 4.]]]
            """
        return torch.cat([inp[:, :, -pad[0]:], inp, inp[:, :, :pad[1]]], dim=2)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CircularPad1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CircularPad1d`。

### Lines 3324-3347
```python

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4))),
            reference_fn=no_batch_dim_reference_fn
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2)),
            forward_input=FunctionInput(make_input((1, 2, 3))),
            reference_fn=lambda m, p, i: padding1d_circular_ref(i, m.padding),
        ),
        ModuleInput(
            constructor_input=FunctionInput((3, 1)),
            forward_input=FunctionInput(make_input((1, 2, 3))),
            reference_fn=lambda m, p, i: padding1d_circular_ref(i, m.padding),
        ),
        ModuleInput(
            constructor_input=FunctionInput((3, 3)),
            forward_input=FunctionInput(make_input((1, 2, 3))),
            reference_fn=lambda m, p, i: padding1d_circular_ref(i, m.padding),
        ),
    ]

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3348-3364
```python
def module_inputs_torch_nn_CircularPad2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    def padding2d_circular_ref(inp, pad):
        r"""input:
                [[[[0., 1., 2],
                   [3., 4., 5.]]]]
                pad: (1, 2, 2, 1)
        output:
            [[[[2., 0., 1., 2., 0., 1.],
               [5., 3., 4., 5., 3., 4.],
               [2., 0., 1., 2., 0., 1.],
               [5., 3., 4., 5., 3., 4.],
               [2., 0., 1., 2., 0., 1.]]]]
        """
        inp = torch.cat([inp[:, :, -pad[2]:], inp, inp[:, :, :pad[3]]], dim=2)
        return torch.cat([inp[:, :, :, -pad[0]:], inp, inp[:, :, :, :pad[1]]], dim=3)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CircularPad2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CircularPad2d`。

### Lines 3365-3388
```python

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4, 5))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 2, 1)),
            forward_input=FunctionInput(make_input((1, 1, 2, 3))),
            reference_fn=lambda m, p, i: padding2d_circular_ref(i, m.padding),
        ),
        ModuleInput(
            constructor_input=FunctionInput((2, 3, 2, 2)),
            forward_input=FunctionInput(make_input((1, 1, 2, 3))),
            reference_fn=lambda m, p, i: padding2d_circular_ref(i, m.padding),
        ),
        ModuleInput(
            constructor_input=FunctionInput((3, 3, 3, 1)),
            forward_input=FunctionInput(make_input((1, 1, 3, 3))),
            reference_fn=lambda m, p, i: padding2d_circular_ref(i, m.padding),
        ),
    ]

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3389-3405
```python
def module_inputs_torch_nn_CircularPad3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)


    def padding3d_circular_ref(inp, pad):
        r"""input:
                [[[[[ 0.,  1.,  2.],
                    [ 3.,  4.,  5.]],
                   [[ 6.,  7.,  8.],
                    [ 9., 10., 11.]]]]]
            pad: (1, 2, 2, 1, 1, 2)
            output: [[[[[ 8.,  6.,  7.,  8.,  6.,  7.],
                        [11.,  9., 10., 11.,  9., 10.],
                        [ 8.,  6.,  7.,  8.,  6.,  7.],
                        [11.,  9., 10., 11.,  9., 10.],
                        [ 8.,  6.,  7.,  8.,  6.,  7.]],

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_inputs_torch_nn_CircularPad3d`, `padding3d_circular_ref`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_inputs_torch_nn_CircularPad3d`, `padding3d_circular_ref`。

### Lines 3406-3417
```python
                       [[ 2.,  0.,  1.,  2.,  0.,  1.],
                        [ 5.,  3.,  4.,  5.,  3.,  4.],
                        [ 2.,  0.,  1.,  2.,  0.,  1.],
                        [ 5.,  3.,  4.,  5.,  3.,  4.],
                        [ 2.,  0.,  1.,  2.,  0.,  1.]],

                       [[ 8.,  6.,  7.,  8.,  6.,  7.],
                        [11.,  9., 10., 11.,  9., 10.],
                        [ 8.,  6.,  7.,  8.,  6.,  7.],
                        [11.,  9., 10., 11.,  9., 10.],
                        [ 8.,  6.,  7.,  8.,  6.,  7.]],

```
- EN: This block implements local helper logic for common modules. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common modules 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 3418-3432
```python
                       [[ 2.,  0.,  1.,  2.,  0.,  1.],
                        [ 5.,  3.,  4.,  5.,  3.,  4.],
                        [ 2.,  0.,  1.,  2.,  0.,  1.],
                        [ 5.,  3.,  4.,  5.,  3.,  4.],
                        [ 2.,  0.,  1.,  2.,  0.,  1.]],

                       [[ 8.,  6.,  7.,  8.,  6.,  7.],
                        [11.,  9., 10., 11.,  9., 10.],
                        [ 8.,  6.,  7.,  8.,  6.,  7.],
                        [11.,  9., 10., 11.,  9., 10.],
                        [ 8.,  6.,  7.,  8.,  6.,  7.]]]]]
        """
        inp = torch.cat([inp[:, :, -pad[4]:], inp, inp[:, :, :pad[5]]], dim=2)
        inp = torch.cat([inp[:, :, :, -pad[2]:], inp, inp[:, :, :, :pad[3]]], dim=3)
        return torch.cat([inp[:, :, :, :, -pad[0]:], inp, inp[:, :, :, :, :pad[1]]], dim=4)
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3433-3456
```python

    return [
        ModuleInput(
            constructor_input=FunctionInput(1),
            forward_input=FunctionInput(make_input((3, 4, 5, 6))),
            reference_fn=no_batch_dim_reference_fn,
        ),
        ModuleInput(
            constructor_input=FunctionInput((1, 2, 1, 2, 1, 2)),
            forward_input=FunctionInput(make_input((1, 1, 2, 2, 3))),
            reference_fn=lambda m, p, i: padding3d_circular_ref(i, m.padding)
        ),
        ModuleInput(
            constructor_input=FunctionInput((3, 2, 2, 1, 1, 2)),
            forward_input=FunctionInput(make_input((1, 1, 2, 2, 3))),
            reference_fn=lambda m, p, i: padding3d_circular_ref(i, m.padding)
        ),
        ModuleInput(
            constructor_input=FunctionInput((3, 3, 2, 1, 2, 2)),
            forward_input=FunctionInput(make_input((1, 1, 2, 2, 3))),
            reference_fn=lambda m, p, i: padding3d_circular_ref(i, m.padding)
        ),
    ]

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3457-3480
```python

# All these operators share similar issues on cuDNN and MIOpen
rnn_gru_lstm_module_info_decorators = (
    # RuntimeError: Batching rule not implemented for aten::_cudnn_rnn_backward.
    # We could not generate a fallback
    DecorateInfo(
        unittest.expectedFailure, "TestModule", "test_grad",
        active_if=(TEST_CUDNN and not TEST_WITH_ROCM), device_type='cuda'
    ),
    # NotImplementedError: the derivative for '_cudnn_rnn_backward' is not implemented.
    # Double backwards is not supported for CuDNN RNNs due to limitations in the CuDNN API
    DecorateInfo(
        unittest.expectedFailure, "TestModule", "test_gradgrad",
        active_if=(TEST_CUDNN and not TEST_WITH_ROCM), device_type='cuda'
    ),
    # CUDNN GRU doesn't accept non-contiguous hx
    DecorateInfo(
        unittest.expectedFailure, "TestModule", "test_non_contiguous_tensors",
        active_if=(TEST_CUDNN and not TEST_WITH_ROCM), device_type='cuda'
    ),
    # MIOPEN GRU doesn't accept non-contiguous hx (this is dispatched to miopen only for float).
    DecorateInfo(
        unittest.expectedFailure, "TestModule", "test_non_contiguous_tensors",
        active_if=(TEST_CUDNN and TEST_WITH_ROCM), dtypes=(torch.float,), device_type='cuda'
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3481-3495
```python
    )
)

# Start of module error inputs functions.

def module_error_inputs_torch_nn_NLLLoss(module_info, device, dtype, requires_grad, training, **kwargs):
    """
    Error inputs for NLLLoss that test weight dtype must match input dtype.
    Regression test for device parity between CPU and CUDA with empty tensors.
    """
    input_t = torch.tensor([], device=device, dtype=dtype).reshape((0, 0))
    weight_dtype = torch.float32 if dtype == torch.float16 else torch.float16
    weight_t = torch.tensor([], device=device, dtype=weight_dtype)
    target_t = torch.tensor([], device=device, dtype=torch.long)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_NLLLoss`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_NLLLoss`。

### Lines 3496-3507
```python
    return [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(weight=weight_t),
                forward_input=FunctionInput(input_t, target_t),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex=r"expected scalar type \w+ but found \w+"
        ),
    ]

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3508-3531
```python
def module_error_inputs_torch_nn_RNN_GRU_Cell(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    samples = [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
                forward_input=FunctionInput(make_input(3, 11), make_input(3, 20)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="input has inconsistent input_size: got 11 expected 10"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
                forward_input=FunctionInput(make_input(3, 10), make_input(3, 21)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="hidden0 has inconsistent hidden_size: got 21, expected 20"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_RNN_GRU_Cell`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_RNN_GRU_Cell`。

### Lines 3532-3555
```python
                forward_input=FunctionInput(make_input(3, 10), make_input(5, 20)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="Input batch size 3 doesn't match hidden0 batch size 5"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
                forward_input=FunctionInput(make_input(3, 10), make_input(3, 1, 1, 20)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex="Expected hidden to be 1D or 2D, got 4D instead"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20, 'relu'),
                forward_input=FunctionInput(make_input(3, 10), make_input(3, 21)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="hidden0 has inconsistent hidden_size: got 21, expected 20"
        ),
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 3556-3567
```python
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20, 'tanh'),
                forward_input=FunctionInput(make_input(3, 10), make_input(3, 21)),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="hidden0 has inconsistent hidden_size: got 21, expected 20"
        ),
    ]
    return samples

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3568-3591
```python
def module_error_inputs_torch_nn_LSTMCell(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)
    samples = [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
                forward_input=FunctionInput(make_input(3, 11), (make_input(3, 20), make_input(3, 20))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="input has inconsistent input_size: got 11 expected 10"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
                forward_input=FunctionInput(make_input(3, 10), (make_input(3, 21), make_input(3, 21))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="hidden0 has inconsistent hidden_size: got 21, expected 20"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_LSTMCell`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_LSTMCell`。

### Lines 3592-3608
```python
                forward_input=FunctionInput(make_input(3, 10), (make_input(5, 20), make_input(5, 20))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=RuntimeError,
            error_regex="Input batch size 3 doesn't match hidden0 batch size 5"
        ),
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(10, 20),
                forward_input=FunctionInput(make_input(3, 10), (make_input(3, 1, 1, 20), make_input(3, 1, 1, 20))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex="Expected hx\\[0\\] to be 1D or 2D, got 4D instead"
        ),
    ]
    return samples
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 3609-3621
```python


def module_error_inputs_torch_nn_RNN_GRU(module_info, device, dtype, requires_grad, training, **kwargs):
    # use float64 for dtype mismatch test if current dtype is float32, otherwise use float32
    # MPS doesn't support float64, so use float16 instead
    # Extract device type from device string (e.g., 'mps:0' -> 'mps')
    device_type = device.split(':')[0] if isinstance(device, str) else device.type
    if dtype == torch.float32:
        mismatched_dtype = torch.float16 if device_type == 'mps' else torch.float64
    else:
        mismatched_dtype = torch.float32
    make_input = partial(make_tensor, device=device, dtype=mismatched_dtype, requires_grad=requires_grad)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_RNN_GRU`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_RNN_GRU`。

### Lines 3622-3645
```python
    samples = [
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(10, 0, 1)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=ValueError,
            error_regex="hidden_size must be greater than zero"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(10, 10, 0)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=ValueError,
            error_regex="num_layers must be greater than zero"
        ),
        # Test dtype mismatch error message
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(3, 5, dtype=dtype, device=device),
                forward_input=FunctionInput(make_input((2, 4, 3))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex=(r"RNN input dtype .* does not match weight dtype .* "
                         r"Convert input: input\.to\(.*\), or convert model: model\.to\(.*\)")
        ),
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 3646-3669
```python
        # Test bias parameter type validation
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5, bias=0)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="bias should be of type bool, got: int"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5, bias=1)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="bias should be of type bool, got: int"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5, bias="True")),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="bias should be of type bool, got: str"
        ),
        # Test batch_first parameter type validation
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5, batch_first=0)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 3670-3693
```python
            error_regex="batch_first should be of type bool, got: int"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5, batch_first=1)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="batch_first should be of type bool, got: int"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5, batch_first="False")),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="batch_first should be of type bool, got: str"
        ),
        # Test input_size parameter type validation
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3.0, 5)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="input_size should be of type int, got: float"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput("10", 5)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 3694-3717
```python
            error_type=TypeError,
            error_regex="input_size should be of type int, got: str"
        ),
        # Test input_size parameter value validation
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(0, 5)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=ValueError,
            error_regex="input_size must be greater than zero"
        ),
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(-1, 5)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=ValueError,
            error_regex="input_size must be greater than zero"
        ),
        # Test hidden_size parameter type validation
        ErrorModuleInput(
            ModuleInput(constructor_input=FunctionInput(3, 5.0)),
            error_on=ModuleErrorEnum.CONSTRUCTION_ERROR,
            error_type=TypeError,
            error_regex="hidden_size should be of type int, got: float"
        ),
    ]
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 3718-3734
```python
    return samples

def module_error_inputs_torch_nn_Pad1d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    is_constant = kwargs.get('is_constant', False)

    return [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(1, 3) if is_constant else FunctionInput(3),
                forward_input=FunctionInput(make_input((2, 3, 4, 5))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex=r"expected 2D or 3D input \(got 4D input\)",

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_Pad1d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_Pad1d`。

### Lines 3735-3752
```python
        ),
    ]

def module_error_inputs_torch_nn_Pad2d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    is_constant = kwargs.get('is_constant', False)

    return [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(1, 3) if is_constant else FunctionInput(3),
                forward_input=FunctionInput(make_input((2, 3))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex=r"expected 3D or 4D input \(got 2D input\)",

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_Pad2d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_Pad2d`。

### Lines 3753-3770
```python
        ),
    ]

def module_error_inputs_torch_nn_Pad3d(module_info, device, dtype, requires_grad, training, **kwargs):
    make_input = partial(make_tensor, device=device, dtype=dtype, requires_grad=requires_grad)

    is_constant = kwargs.get('is_constant', False)

    return [
        ErrorModuleInput(
            ModuleInput(
                constructor_input=FunctionInput(1, 3) if is_constant else FunctionInput(3),
                forward_input=FunctionInput(make_input((2, 3))),
            ),
            error_on=ModuleErrorEnum.FORWARD_ERROR,
            error_type=ValueError,
            error_regex=r"expected 4D or 5D input \(got 2D input\)",

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `module_error_inputs_torch_nn_Pad3d`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`module_error_inputs_torch_nn_Pad3d`。

### Lines 3771-3794
```python
        ),
    ]


_macos15_or_newer = torch.backends.mps.is_available() and torch.backends.mps.is_macos_or_newer(15, 0)


# Database of ModuleInfo entries in alphabetical order.
module_db: list[ModuleInfo] = [
    ModuleInfo(torch.nn.AdaptiveAvgPool1d,
               module_inputs_func=module_inputs_torch_nn_AdaptiveAvgPool1d,
               skips=(
                   # Fails on MPS backend if input/output sizes are not divisible
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.AdaptiveAvgPool2d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_inputs_func=module_inputs_torch_nn_AdaptiveAvgPool2d,
               skips=(
                   # Fails on MPS backend if input/output sizes are not divisible
                   DecorateInfo(skipMPS),
                   # Fails on backward check if output size is 1x1
                   DecorateInfo(
                       unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; bridges to backend-specific execution artifacts; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；桥接到特定后端的执行产物；处理条件控制流。关键符号：无明显局部符号。

### Lines 3795-3818
```python
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training'),
                   ),)
               ),
    ModuleInfo(torch.nn.AdaptiveAvgPool3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_inputs_func=module_inputs_torch_nn_AdaptiveAvgPool3d,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # not supported on MPS backend
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.AdaptiveMaxPool1d,
               module_inputs_func=module_inputs_torch_nn_AdaptiveMaxPool1d,
               ),
    ModuleInfo(torch.nn.AdaptiveMaxPool2d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_inputs_func=module_inputs_torch_nn_AdaptiveMaxPool2d,
               ),
    ModuleInfo(torch.nn.AdaptiveMaxPool3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_inputs_func=module_inputs_torch_nn_AdaptiveMaxPool3d,
               skips=(
```
- EN: This block checks invariants or expected outcomes; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；桥接到特定后端的执行产物；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 3819-3842
```python
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # not supported on MPS backend
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.AvgPool1d,
               module_inputs_func=module_inputs_torch_nn_AvgPool1d,
               ),
    ModuleInfo(torch.nn.AvgPool2d,
               module_inputs_func=module_inputs_torch_nn_AvgPool2d,
               skips=(
                   # The difference between channels last backward and
                   # channels first backward of AvgPool2d on CUDA is too large
                   # See https://github.com/pytorch/pytorch/issues/107201
                   DecorateInfo(
                       unittest.expectedFailure,
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training'),
                       device_type='cuda',),
               ),),
    ModuleInfo(torch.nn.AvgPool3d,
               module_inputs_func=module_inputs_torch_nn_AvgPool3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 3843-3866
```python
                   # No channels_last support for AvgPool1d as it does not take 4D inputs
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # backward not supported on MPS backend
                   DecorateInfo(skipMPS, 'TestModule', 'test_non_contiguous_tensors'),)
               ),
    ModuleInfo(torch.nn.BatchNorm1d,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_BatchNorm1d,
               module_error_inputs_func=module_error_inputs_torch_nn_BatchNorm1d_2d_3d,
               skips=(
                   # tracking here rather than in the list in test_aotdispatch.py as eval mode passes
                   # RuntimeError: tried to get Double out of SymInt
                   DecorateInfo(
                       unittest.expectedFailure, 'TestEagerFusionModuleInfo',
                       'test_aot_autograd_symbolic_module_exhaustive',
                       active_if=operator.itemgetter('training')
                   ),
                   # torch._subclasses.fake_tensor.DataDependentOutputException: aten._local_scalar_dense.default
                   DecorateInfo(
                       unittest.expectedFailure, 'TestEagerFusionModuleInfo',
                       'test_aot_autograd_module_exhaustive',
                       active_if=operator.itemgetter('training')
                   ))
               ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; bridges to backend-specific execution artifacts; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；桥接到特定后端的执行产物；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3867-3890
```python
    ModuleInfo(torch.nn.BatchNorm2d,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_BatchNorm2d,
               module_error_inputs_func=module_error_inputs_torch_nn_BatchNorm1d_2d_3d,
               skips=(
                   # See https://github.com/pytorch/pytorch/issues/134580
                   DecorateInfo(expectedFailureMPS, 'TestModule', 'test_memory_format', active_if=operator.itemgetter('training')),
                   # tracking here rather than in the list in test_aotdispatch.py as eval mode passes
                   # RuntimeError: tried to get Double out of SymInt
                   DecorateInfo(
                       unittest.expectedFailure, 'TestEagerFusionModuleInfo',
                       'test_aot_autograd_symbolic_module_exhaustive',
                       active_if=operator.itemgetter('training')
                   ),
                   # torch._subclasses.fake_tensor.DataDependentOutputException: aten._local_scalar_dense.default
                   DecorateInfo(
                       unittest.expectedFailure, 'TestEagerFusionModuleInfo',
                       'test_aot_autograd_module_exhaustive',
                       active_if=operator.itemgetter('training')
                   ),)
               ),
    ModuleInfo(torch.nn.BatchNorm3d,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_BatchNorm3d,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; reports or normalizes error conditions; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；报告或规范化错误情况；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 3891-3914
```python
               module_error_inputs_func=module_error_inputs_torch_nn_BatchNorm1d_2d_3d,
               skips=(
                   # not supported on MPS backend
                   DecorateInfo(skipMPS),
                   # tracking here rather than in the list in test_aotdispatch.py as eval mode passes
                   # RuntimeError: tried to get Double out of SymInt
                   DecorateInfo(
                       unittest.expectedFailure, 'TestEagerFusionModuleInfo',
                       'test_aot_autograd_symbolic_module_exhaustive',
                       active_if=operator.itemgetter('training')
                   ),
                   # torch._subclasses.fake_tensor.DataDependentOutputException: aten._local_scalar_dense.default
                   DecorateInfo(
                       unittest.expectedFailure, 'TestEagerFusionModuleInfo',
                       'test_aot_autograd_module_exhaustive',
                       active_if=operator.itemgetter('training')
                   ),)
               ),
    ModuleInfo(torch.nn.CELU,
               module_inputs_func=module_inputs_torch_nn_CELU,
               # not MPS specific, will be xfailed for all devices in next PR
               skips=(
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_check_inplace',
                                device_type='mps', dtypes=[torch.float16]),)
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; advances ONNX export translation; bridges to backend-specific execution artifacts; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；推进 ONNX 导出翻译流程；桥接到特定后端的执行产物；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3915-3938
```python
               ),
    ModuleInfo(torch.nn.Conv1d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=1, lazy=False),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.Conv2d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=2, lazy=False),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format",
                                device_type='cuda', dtypes=[torch.float64]),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.Conv3d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=3, lazy=False),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 3939-3962
```python
               module_memformat_affects_out=True,
               skips=(
                   # Conv3d is not supported on MPS backend
                   DecorateInfo(skipMPS, device_type="mps"),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format"),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.ConvTranspose1d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=1, lazy=False, transposed=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               dtypes=floating_and_complex_types_and(torch.chalf),
               skips=(
                   # Not implemented for chalf on CPU
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_cpu_gpu_parity',
                                dtypes=(torch.chalf,), device_type='cuda'),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
                   DecorateInfo(precisionOverride({torch.chalf: 5e-03}), 'TestModule', 'test_memory_format'),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3963-3986
```python
               )),
    ModuleInfo(torch.nn.ConvTranspose2d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=2, lazy=False, transposed=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               dtypes=floating_and_complex_types_and(torch.chalf),
               skips=(
                   # Fails on backward check because ViewAsRealBackward apply contiguous for grad
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_memory_format',
                                dtypes=(torch.complex32, torch.complex64, torch.complex128)),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format", device_type='cuda',
                                dtypes=[torch.float64, torch.complex128]),
                   # Not implemented for chalf on CPU
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_cpu_gpu_parity',
                                dtypes=(torch.chalf,), device_type='cuda'),
                   DecorateInfo(skipIfXpu, 'TestModule', 'test_cpu_gpu_parity',
                                dtypes=(torch.chalf,), device_type='xpu'),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
                   DecorateInfo(precisionOverride({torch.chalf: 5e-03}), 'TestModule', 'test_memory_format'),
               )),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 3987-4010
```python
    ModuleInfo(torch.nn.ConvTranspose3d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=3, lazy=False, transposed=True),
               dtypes=floating_and_complex_types_and(torch.chalf),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
                   # ConvTranspose3d is not supported on MPS backend
                   DecorateInfo(skipMPS),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format"),
                   # Not implemented for chalf on CPU
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_cpu_gpu_parity',
                                dtypes=(torch.chalf,), device_type='cuda'),
                   DecorateInfo(skipIfXpu, 'TestModule', 'test_cpu_gpu_parity',
                                dtypes=(torch.chalf,), device_type='xpu'),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
                   DecorateInfo(precisionOverride({torch.complex64: 1e-04}), 'TestModule', 'test_cpu_gpu_parity'),
                   DecorateInfo(precisionOverride({torch.chalf: 5e-03}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.CosineEmbeddingLoss,
               module_inputs_func=module_inputs_torch_nn_CosineEmbeddingLoss,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4011-4034
```python
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.ELU,
               module_inputs_func=module_inputs_torch_nn_ELU,
               # not MPS specific, will be xfailed for all devices in next PR
               skips=(
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_check_inplace',
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.FractionalMaxPool2d,
               module_inputs_func=module_inputs_torch_nn_FractionalMaxPool2d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
                   # not supported on MPS backend
                   DecorateInfo(skipMPS),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.FractionalMaxPool3d,
               module_inputs_func=module_inputs_torch_nn_FractionalMaxPool3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
                   # not supported on MPS backend
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4035-4058
```python
                   DecorateInfo(skipMPS),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.L1Loss,
               module_inputs_func=module_inputs_torch_nn_L1Loss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.SmoothL1Loss,
               module_inputs_func=module_inputs_torch_nn_SmoothL1Loss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: input types 'tensor<f32>' and 'tensor<15x10xf16>' are not broadcast compatible
                   # NS: Still fails on MacOS15.1
                   DecorateInfo(skipIfMPS, 'TestModule', 'test_non_contiguous_tensors',
                                dtypes=[torch.float16], device_type='mps'),),
               ),
    ModuleInfo(torch.nn.LazyConv1d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=1, lazy=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4059-4082
```python
                   # Lazy modules don't currently play well with ModuleInfo tests on the meta device.
                   # See https://github.com/pytorch/pytorch/issues/70505 for more info.
                   DecorateInfo(skipMeta),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.LazyConv2d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=2, lazy=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
                   # Lazy modules don't currently play well with ModuleInfo tests on the meta device.
                   # See https://github.com/pytorch/pytorch/issues/70505 for more info.
                   DecorateInfo(skipMeta),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format",
                                device_type='cuda', dtypes=[torch.float64]),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.LazyConv3d,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4083-4106
```python
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=3, lazy=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
                   # Lazy modules don't currently play well with ModuleInfo tests on the meta device.
                   # See https://github.com/pytorch/pytorch/issues/70505 for more info.
                   DecorateInfo(skipMeta),
                   # LazyConv3d is not supported on MPS backend
                   DecorateInfo(skipMPS),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format"),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.LazyConvTranspose1d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=1, lazy=True, transposed=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
                   # Lazy modules don't currently play well with ModuleInfo tests on the meta device.
                   # See https://github.com/pytorch/pytorch/issues/70505 for more info.
                   DecorateInfo(skipMeta),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4107-4130
```python
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.LazyConvTranspose2d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=2, lazy=True, transposed=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
               skips=(
                   # Lazy modules don't currently play well with ModuleInfo tests on the meta device.
                   # See https://github.com/pytorch/pytorch/issues/70505 for more info.
                   DecorateInfo(skipMeta),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format", device_type='cuda',
                                dtypes=[torch.float64]),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.LazyConvTranspose3d,
               module_inputs_func=partial(module_inputs_torch_nn_ConvNd, N=3, lazy=True, transposed=True),
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               module_memformat_affects_out=True,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4131-4154
```python
               skips=(
                   # Lazy modules don't currently play well with ModuleInfo tests on the meta device.
                   # See https://github.com/pytorch/pytorch/issues/70505 for more info.
                   DecorateInfo(skipMeta),
                   # LazyConvTranspose3d is not supported on MPS backend
                   DecorateInfo(skipMPS),
                   # This was wrongly being skipped before and needs investigation.
                   # See https://github.com/pytorch/pytorch/issues/80247
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format"),
               ),
               decorators=(
                   DecorateInfo(precisionOverride({torch.float32: 1e-04}), 'TestModule', 'test_memory_format'),
               )),
    ModuleInfo(torch.nn.Linear,
               module_inputs_func=module_inputs_torch_nn_Linear,
               skips=(
                   # No channels_last support for Linear currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.Bilinear,
               module_inputs_func=module_inputs_torch_nn_Bilinear,
               decorators=[
                   DecorateInfo(
                       toleranceOverride({
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4155-4178
```python
                           torch.float32: tol(atol=1e-4, rtol=1e-4),
                           torch.float64: tol(atol=1e-4, rtol=1e-4)}),
                       'TestModule', 'test_forward', device_type='cpu'),
               ],
               skips=(
                   # No channels_last support for Bilinear currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward",
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.LPPool1d,
               module_inputs_func=module_inputs_torch_nn_LPPool1d,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_grad'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),)
               ),
    ModuleInfo(torch.nn.LPPool2d,
               module_inputs_func=module_inputs_torch_nn_LPPool2d,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_grad'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),
                   # Fails on backward check on MPS
                   # See https://github.com/pytorch/pytorch/issues/107214
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4179-4202
```python
                   DecorateInfo(
                       unittest.expectedFailure,
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training') and not _macos15_or_newer,
                       device_type='mps',
                   ),)
               ),
    ModuleInfo(torch.nn.LPPool3d,
               module_inputs_func=module_inputs_torch_nn_LPPool3d,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_grad'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   DecorateInfo(skipIfMPS, device_type='mps'),)
               ),
    ModuleInfo(torch.nn.MaxPool1d,
               module_inputs_func=module_inputs_torch_nn_MaxPool1d,
               ),
    ModuleInfo(torch.nn.MaxPool2d,
               module_inputs_func=module_inputs_torch_nn_MaxPool2d,
               module_error_inputs_func=module_error_inputs_torch_nn_MaxPool2d,
               ),
    ModuleInfo(torch.nn.MaxPool3d,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 4203-4226
```python
               module_inputs_func=module_inputs_torch_nn_MaxPool3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               ),
    ModuleInfo(torch.nn.KLDivLoss,
               module_inputs_func=module_inputs_torch_nn_KLDivLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # https://github.com/pytorch/pytorch/issues/115588
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_cpu_gpu_parity'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_grad'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),)
               ),
    ModuleInfo(torch.nn.MSELoss,
               module_inputs_func=module_inputs_torch_nn_MSELoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward",
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.MarginRankingLoss,
               module_inputs_func=module_inputs_torch_nn_MarginRankingLoss,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4227-4250
```python
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.MultiLabelMarginLoss,
               module_inputs_func=module_inputs_torch_nn_MultiLabelMarginLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # 'aten::multilabel_margin_loss_forward' is not currently implemented for the MPS device.
                   DecorateInfo(skipIfMPS, 'TestModule', device_type='mps'),
                   # derivative for aten::multilabel_margin_loss_backward is not implemented
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),)
               ),
    ModuleInfo(torch.nn.MultiMarginLoss,
               module_inputs_func=module_inputs_torch_nn_MultiMarginLoss,
               module_error_inputs_func=module_error_inputs_torch_nn_MultiMarginLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # 'aten::multi_margin_loss' is not currently implemented for the MPS device.
                   DecorateInfo(skipIfMPS, 'TestModule', device_type='mps'),
                   # RuntimeError: derivative for aten::multi_margin_loss_backward is not implemented
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),)
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4251-4274
```python
               ),
    ModuleInfo(torch.nn.SoftMarginLoss,
               module_inputs_func=module_inputs_torch_nn_SoftMarginLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward",
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.MultiLabelSoftMarginLoss,
               module_inputs_func=module_inputs_torch_nn_MultiLabelSoftMarginLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.NLLLoss,
               module_inputs_func=module_inputs_torch_nn_NLLLoss,
               module_error_inputs_func=module_error_inputs_torch_nn_NLLLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4275-4298
```python
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.GaussianNLLLoss,
               module_inputs_func=module_inputs_torch_nn_GaussianNLLLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)),
    ModuleInfo(torch.nn.PoissonNLLLoss,
               module_inputs_func=module_inputs_torch_nn_PoissonNLLLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)),
    ModuleInfo(torch.nn.HingeEmbeddingLoss,
               module_inputs_func=module_inputs_torch_nn_HingeEmbeddingLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.HuberLoss,
               module_inputs_func=module_inputs_torch_nn_HuberLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: seemingly incorrect output dtype
```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4299-4322
```python
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward",
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.BCELoss,
               module_inputs_func=module_inputs_torch_nn_BCELoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # error: input types 'tensor<f32>' and 'tensor<15x10xf16>' are not broadcast compatible
                   DecorateInfo(skipIfMPS, 'TestModule', dtypes=[torch.float16], device_type='mps'),)
               ),
    ModuleInfo(torch.nn.BCEWithLogitsLoss,
               module_inputs_func=module_inputs_torch_nn_BCEWithLogitsLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # see #119108: tolerance issue
                   DecorateInfo(skipIfMPS, 'TestModule', dtypes=[torch.float16], device_type='mps'),)
               ),
    ModuleInfo(torch.nn.CrossEntropyLoss,
               module_inputs_func=module_inputs_torch_nn_CrossEntropyLoss,
               module_error_inputs_func=module_error_inputs_torch_nn_CrossEntropyLoss,
               dtypes=get_all_fp_dtypes(include_half=True, include_bfloat16=False),
               decorators=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4323-4346
```python
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_memory_format'),
                   DecorateInfo(toleranceOverride({torch.float16: tol(atol=3e-2, rtol=1e-3)}), "TestModule",
                                "test_forward", dtypes=[torch.float16], device_type='cpu'),
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_cpu_gpu_parity", dtypes=[torch.float16],
                                device_type='cuda'),
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_cpu_gpu_parity", dtypes=[torch.float16],
                                device_type='xpu'),),
               ),
    ModuleInfo(torch.nn.CTCLoss,
               module_inputs_func=module_inputs_torch_nn_CTCLoss,
               skips=(
                   # No channels_last support for loss functions.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # The operator aten::_ctc_loss is not currently implemented for the MPS device.
                   DecorateInfo(skipIfMPS, 'TestModule', device_type='mps',),
                   # derivative for aten::_ctc_loss_backward is not implemented
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_grad'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad'),
                   # https://github.com/pytorch/pytorch/issues/115585
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_non_contiguous_tensors'),)
               ),
    ModuleInfo(torch.nn.GELU,
               module_inputs_func=module_inputs_torch_nn_GELU,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4347-4370
```python
               skips=(
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward",
                                device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.GLU,
               module_inputs_func=module_inputs_torch_nn_GLU,
               ),
    ModuleInfo(torch.nn.GroupNorm,
               module_inputs_func=module_inputs_torch_nn_GroupNorm,
               module_error_inputs_func=module_error_inputs_torch_nn_GroupNorm,
               dtypes=get_all_fp_dtypes(include_bfloat16=True, include_half=True),
               skips=(
                   # Tracking at https://github.com/pytorch/pytorch/issues/98089
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_cpu_gpu_parity'),
                   DecorateInfo(toleranceOverride({torch.float32: tol(atol=1e-4, rtol=1e-4)}),
                                'TestModule', 'test_memory_format', device_type='cpu'),
                   # No channels_last support for GroupNorm currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format', device_type='cuda'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format', device_type='mps'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format', device_type='xpu'),
                   DecorateInfo(unittest.skip("Skipped!"), "TestModule", "test_grad",
                                active_if=TEST_WITH_ROCM, device_type='cuda'),)
               ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4371-4394
```python
    ModuleInfo(torch.nn.Hardshrink,
               module_inputs_func=module_inputs_torch_nn_Hardshrink,
               ),
    ModuleInfo(torch.nn.Hardswish,
               module_inputs_func=module_inputs_torch_nn_Hardswish,
               supports_gradgrad=False),
    ModuleInfo(torch.nn.Hardtanh,
               module_inputs_func=module_inputs_torch_nn_Hardtanh,
               ),
    ModuleInfo(torch.nn.InstanceNorm1d,
               module_inputs_func=partial(module_inputs_torch_nn_InstanceNormNd, N=1),
               train_and_eval_differ=True,
               skips=(
                   # No channels_last support for InstanceNorm1d currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.InstanceNorm2d,
               module_inputs_func=partial(module_inputs_torch_nn_InstanceNormNd, N=2),
               train_and_eval_differ=True,
               skips=(
                   # No channels_last support for InstanceNorm2d currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.InstanceNorm3d,
```
- EN: This block reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4395-4418
```python
               module_inputs_func=partial(module_inputs_torch_nn_InstanceNormNd, N=3),
               train_and_eval_differ=True,
               skips=(
                   # not supported on MPS backend
                   DecorateInfo(expectedFailureMPS, 'TestModuleMPS', 'test_memory_format'),
                   DecorateInfo(expectedFailureMPS, 'TestModuleMPS', 'test_non_contiguous_tensors'),
                   DecorateInfo(expectedFailureMPS, 'TestModuleMPS', 'test_forward'),
                   DecorateInfo(expectedFailureMPS, 'TestModuleMPS', 'test_non_contiguous'),
                   DecorateInfo(expectedFailureMPS, 'TestModuleMPS', 'test_save_load'),
                   # No channels_last support for InstanceNorm3d currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.LocalResponseNorm,
               module_inputs_func=module_inputs_torch_nn_LocalResponseNorm,
               ),
    ModuleInfo(torch.nn.LayerNorm,
               module_inputs_func=module_inputs_torch_nn_LayerNorm,
               skips=(
                   # No channels_last support for LayerNorm currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.RMSNorm,
               module_inputs_func=module_inputs_torch_nn_RMSNorm,
               ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4419-4442
```python
    # TransformerEncoder takes the same inputs as TransformerEncoderLayer
    ModuleInfo(torch.nn.TransformerEncoder,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_TransformerEncoder,
               decorators=[
                   # Not implemented for SDPA backward derivative
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad',
                                device_type='cpu'),
               ],
               skips=(
                   # No channels_last support for TransformerEncoderLayer currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # Doesn't support device / dtype kwargs directly because it is just a
                   # container of TransformerEncoderLayers.
                   DecorateInfo(unittest.expectedFailure, 'TestModule', 'test_factory_kwargs'),)
               ),
    ModuleInfo(torch.nn.TransformerEncoderLayer,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_TransformerEncoderLayer,
               decorators=[
                   DecorateInfo(toleranceOverride({torch.float32: tol(atol=1e-4, rtol=1e-4)}),
                                'TestModule', 'test_non_contiguous_tensors',
                                device_type='cpu', active_if=IS_WINDOWS),
                   DecorateInfo(toleranceOverride({torch.float16: tol(atol=1e-4, rtol=2e-3)}),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4443-4466
```python
                                'TestModule', 'test_forward',
                                device_type='mps'),
                   # Not implemented for SDPA backward derivative
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad',
                                device_type='cpu'),
               ],
               skips=(
                   # No channels_last support for TransformerEncoderLayer currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.TransformerDecoderLayer,
               module_inputs_func=module_inputs_torch_nn_TransformerDecoderLayer,
               decorators=[
                   # Not implemented for SDPA backward derivative
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad',
                                device_type='cpu'),
               ],
               skips=(
                   # No channels_last support for TransformerDecoderLayer currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.Transformer,
               module_inputs_func=module_inputs_torch_nn_Transformer,
               # Inputs are too large to run with slow gradcheck
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4467-4490
```python
               # https://github.com/pytorch/pytorch/issues/117140
               gradcheck_fast_mode=True,
               decorators=[
                   # Not implemented for SDPA backward derivative
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_gradgrad',
                                device_type='cpu'),
               ],
               skips=(
                   # No channels_last support for Transformer currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.MultiheadAttention,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_MultiheadAttention,
               skips=(
                   # No channels_last support for MultiheadAttention currently.
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.Embedding,
               module_inputs_func=module_inputs_torch_nn_Embedding,
               module_error_inputs_func=module_error_inputs_torch_nn_Embedding,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               decorators=[
                   DecorateInfo(toleranceOverride({torch.float32: tol(atol=1e-4, rtol=1e-4)}),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4491-4514
```python
                                'TestModule', 'test_non_contiguous_tensors',
                                device_type='mps')],
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.ReLU,
               module_inputs_func=module_inputs_torch_nn_ReLU,
               skips=None if _macos15_or_newer else (
                   # Fails on backward check on MPS
                   # See https://github.com/pytorch/pytorch/issues/107214
                   DecorateInfo(
                       unittest.expectedFailure,
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training'),
                       device_type='mps',
                   ),)
               ),
    ModuleInfo(torch.nn.LeakyReLU,
               module_inputs_func=module_inputs_torch_nn_LeakyReLU,
               ),
    ModuleInfo(torch.nn.ReLU6,
               module_inputs_func=module_inputs_torch_nn_ReLU6,
               skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 4515-4538
```python
                   # test fails on MPS backend and is being investigated.
                   # See https://github.com/pytorch/pytorch/issues/100914
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.PReLU,
               module_inputs_func=module_inputs_torch_nn_PReLU,
               skips=(
                   # test fails on MPS backend and is being investigated.
                   # See https://github.com/pytorch/pytorch/issues/100914
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.RNNCell,
               module_inputs_func=partial(module_inputs_torch_nn_RNN_GRU_Cell, is_rnn=True),
               module_error_inputs_func=module_error_inputs_torch_nn_RNN_GRU_Cell,
               ),
    ModuleInfo(torch.nn.GRUCell,
               module_inputs_func=module_inputs_torch_nn_RNN_GRU_Cell,
               module_error_inputs_func=module_error_inputs_torch_nn_RNN_GRU_Cell,
               ),
    ModuleInfo(torch.nn.LSTMCell,
               module_inputs_func=module_inputs_torch_nn_LSTMCell,
               module_error_inputs_func=module_error_inputs_torch_nn_LSTMCell,
               ),
    ModuleInfo(torch.nn.Sigmoid,
```
- EN: This block bridges to backend-specific execution artifacts; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 4539-4562
```python
               module_inputs_func=module_inputs_torch_nn_Sigmoid,
               skips=None if _macos15_or_newer else (
                   # Fails on backward check on MPS
                   # See https://github.com/pytorch/pytorch/issues/107214
                   DecorateInfo(
                       unittest.expectedFailure,
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training'),
                       device_type='mps',
                   ),)
               ),
    ModuleInfo(torch.nn.LogSigmoid,
               module_inputs_func=module_inputs_torch_nn_LogSigmoid,
               skips=(
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward", device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.SiLU,
               module_inputs_func=module_inputs_torch_nn_SiLU,
               ),
    ModuleInfo(torch.nn.Softmax,
               module_inputs_func=module_inputs_torch_nn_Softmax,
               ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 4563-4586
```python
    ModuleInfo(torch.nn.Softmax2d,
               module_inputs_func=module_inputs_torch_nn_Softmax2d,
               skips=(
                   # no channels last support for Softmax2d currently
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: tolerance issue
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward", device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.LogSoftmax,
               module_inputs_func=module_inputs_torch_nn_LogSoftmax,
               skips=(
                   # no channels last support for LogSoftmax currently
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),
                   # See #119108: inf nan error
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_forward", device_type='mps', dtypes=[torch.float16]),)
               ),
    ModuleInfo(torch.nn.Softmin,
               module_inputs_func=module_inputs_torch_nn_Softmin,
               skips=(
                   # no channels last support for Softmin currently
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format'),)
               ),
    ModuleInfo(torch.nn.Softplus,
               module_inputs_func=module_inputs_torch_nn_Softplus,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 4587-4610
```python
               skips=(
                   # test fails on MPS backend and is being investigated.
                   # See https://github.com/pytorch/pytorch/issues/100914
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.Softshrink,
               module_inputs_func=module_inputs_torch_nn_Softshrink,
               skips=(
                   # not supported on MPS backend
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.Softsign,
               module_inputs_func=module_inputs_torch_nn_Softsign,
               ),
    ModuleInfo(torch.nn.Tanh,
               module_inputs_func=module_inputs_torch_nn_Tanh,
               skips=None if _macos15_or_newer else (
                   # Fails on backward check on MPS
                   # See https://github.com/pytorch/pytorch/issues/107214
                   DecorateInfo(
                       unittest.expectedFailure,
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training'),
```
- EN: This block checks invariants or expected outcomes; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 4611-4634
```python
                       device_type='mps',
                   ),)
               ),
    ModuleInfo(torch.nn.Tanhshrink,
               module_inputs_func=module_inputs_torch_nn_Tanhshrink,
               skips=None if _macos15_or_newer else (
                   # Fails on backward check on MPS
                   # See https://github.com/pytorch/pytorch/issues/107214
                   DecorateInfo(
                       unittest.expectedFailure,
                       'TestModule',
                       'test_memory_format',
                       active_if=operator.itemgetter('training'),
                       device_type='mps',
                   ),)
               ),
    ModuleInfo(torch.nn.Threshold,
               module_inputs_func=module_inputs_torch_nn_Threshold,
               skips=(
                   # test fails on MPS backend and is being investigated.
                   # See https://github.com/pytorch/pytorch/issues/100914
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.Mish,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 4635-4658
```python
               module_inputs_func=module_inputs_torch_nn_Mish,
               skips=(
                   # not supported on MPS backend
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.RNN,
               train_and_eval_differ=True,
               module_inputs_func=partial(module_inputs_torch_nn_RNN_GRU, is_rnn=True),
               module_error_inputs_func=module_error_inputs_torch_nn_RNN_GRU,
               decorators=rnn_gru_lstm_module_info_decorators
               ),
    ModuleInfo(torch.nn.GRU,
               train_and_eval_differ=True,
               module_inputs_func=partial(module_inputs_torch_nn_RNN_GRU, is_rnn=False),
               module_error_inputs_func=module_error_inputs_torch_nn_RNN_GRU,
               decorators=rnn_gru_lstm_module_info_decorators),
    ModuleInfo(torch.nn.LSTM,
               train_and_eval_differ=True,
               module_inputs_func=module_inputs_torch_nn_LSTM,
               module_error_inputs_func=module_error_inputs_torch_nn_RNN_GRU,
               skips=(
                   # LSTM with projections is not currently supported with MPS
                   DecorateInfo(skipMPS),),
               decorators=rnn_gru_lstm_module_info_decorators),
```
- EN: This block bridges to backend-specific execution artifacts; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 4659-4682
```python
    ModuleInfo(torch.nn.ReflectionPad1d,
               module_inputs_func=module_inputs_torch_nn_ReflectionPad1d,
               ),
    ModuleInfo(torch.nn.ReflectionPad2d,
               module_inputs_func=module_inputs_torch_nn_ReflectionPad2d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='cuda'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='mps'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='xpu'),)
               ),
    ModuleInfo(torch.nn.ReflectionPad3d,
               module_inputs_func=module_inputs_torch_nn_ReflectionPad3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='cuda'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='mps'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='xpu'),)
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 4683-4706
```python
               ),
    ModuleInfo(torch.nn.ReplicationPad1d,
               module_inputs_func=module_inputs_torch_nn_ReplicationPad1d,
               ),
    ModuleInfo(torch.nn.ReplicationPad2d,
               module_inputs_func=module_inputs_torch_nn_ReplicationPad2d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='cuda'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='mps'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='xpu'),)
               ),
    ModuleInfo(torch.nn.ReplicationPad3d,
               module_inputs_func=module_inputs_torch_nn_ReplicationPad3d,
               gradcheck_nondet_tol=GRADCHECK_NONDET_TOL,
               skips=(
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='cuda'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
                                device_type='mps'),
                   DecorateInfo(unittest.skip("Skipped!"), 'TestModule', 'test_memory_format',
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 4707-4730
```python
                                device_type='xpu'),)
               ),
    ModuleInfo(torch.nn.SELU,
               module_inputs_func=module_inputs_torch_nn_SELU,
               skips=(
                   # test fails on MPS backend and is being investigated.
                   # See https://github.com/pytorch/pytorch/issues/100914
                   DecorateInfo(skipMPS),)
               ),
    ModuleInfo(torch.nn.ZeroPad1d,
               module_inputs_func=module_inputs_torch_nn_ZeroPad1d,
               ),
    ModuleInfo(torch.nn.ZeroPad2d,
               module_inputs_func=module_inputs_torch_nn_ZeroPad2d,
               skips=(
                   # Fails with channels last test on MPS backend
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format", device_type='mps'),)
               ),
    ModuleInfo(torch.nn.ZeroPad3d,
               module_inputs_func=module_inputs_torch_nn_ZeroPad3d,
               skips=(
                   # Fails with channels last test on MPS backend
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format", device_type='mps'),)
               ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 4731-4754
```python
    ModuleInfo(torch.nn.CircularPad1d,
               module_inputs_func=module_inputs_torch_nn_CircularPad1d,
               module_error_inputs_func=module_error_inputs_torch_nn_Pad1d,
               ),
    ModuleInfo(torch.nn.CircularPad2d,
               module_inputs_func=module_inputs_torch_nn_CircularPad2d,
               module_error_inputs_func=module_error_inputs_torch_nn_Pad2d,
               ),
    ModuleInfo(torch.nn.CircularPad3d,
               module_inputs_func=module_inputs_torch_nn_CircularPad3d,
               module_error_inputs_func=module_error_inputs_torch_nn_Pad3d,
               skips=(
                   # Fails with channels last test on MPS backend
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format"),)
               ),
    ModuleInfo(torch.nn.ConstantPad1d,
               module_inputs_func=module_inputs_torch_nn_ConstantPad1d,
               ),
    ModuleInfo(torch.nn.ConstantPad2d,
               module_inputs_func=module_inputs_torch_nn_ConstantPad2d,
               skips=(
                   # Fails with channels last test on MPS backend
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format", device_type='mps'),)
               ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reports or normalizes error conditions; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；报告或规范化错误情况；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 4755-4761
```python
    ModuleInfo(torch.nn.ConstantPad3d,
               module_inputs_func=module_inputs_torch_nn_ConstantPad3d,
               skips=(
                   # Fails with channels last test on MPS backend
                   DecorateInfo(unittest.expectedFailure, "TestModule", "test_memory_format", device_type='mps'),)
               )
]
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；复用已计算状态以减少重复工作。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.nn.functional`, `torch.nn.utils.rnn`, `torch.testing`, `torch.testing._internal.common_cuda`, `torch.testing._internal.common_dtype`, `torch.testing._internal.common_device_type`, `torch.testing._internal.common_methods_invocations`, `torch.testing._internal.common_nn`, `torch.testing._internal.common_utils`
- External imports / 外部导入: `unittest`, `copy`, `enum`, `functools`, `itertools`, `math`, `types`, `operator`
- Representative symbols / 代表性符号: `MODULE_CLASSES`, `modules`, `get_module_common_name`, `FunctionInput`, `ModuleInput`, `ModuleErrorEnum`, `ErrorModuleInput`, `ModuleInfo`, `module_inputs_torch_nn_Linear`, `module_inputs_torch_nn_Bilinear`, `...`

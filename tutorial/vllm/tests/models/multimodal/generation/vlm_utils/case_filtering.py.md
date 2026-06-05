# case_filtering.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/case_filtering.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 5 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 5 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L8-L21)
```python
import itertools
from collections import OrderedDict
from collections.abc import Iterable

import pytest

from .types import (
    EMBEDDING_SIZE_FACTORS,
    ExpandableVLMTestArgs,
    ImageSizeWrapper,
    SizeType,
    VLMTestInfo,
    VLMTestType,
)
```
**EN:** Imports standard-library modules such as `collections.OrderedDict`, `collections.abc.Iterable`, `itertools`, third-party packages like `pytest`, project helpers such as `.types.EMBEDDING_SIZE_FACTORS`, `.types.ExpandableVLMTestArgs`, `.types.ImageSizeWrapper`.
**CN:** 导入标准库模块（如 `collections.OrderedDict`、`collections.abc.Iterable`、`itertools`）、第三方包（如 `pytest`）、项目内辅助模块（如 `.types.EMBEDDING_SIZE_FACTORS`、`.types.ExpandableVLMTestArgs`、`.types.ImageSizeWrapper`）。

### Helper / 辅助函数: get_filtered_test_settings (L24-L62)
```python
def get_filtered_test_settings(
    test_settings: dict[str, VLMTestInfo],
    test_type: VLMTestType,
    new_proc_per_test: bool,
) -> dict[str, VLMTestInfo]:
    """Given the dict of potential test settings to run, return a subdict
    of tests who have the current test type enabled with the matching val for
    fork_per_test.
    """

    def matches_test_type(test_info: VLMTestInfo, test_type: VLMTestType):
        return test_info.test_type == test_type or (
            isinstance(test_info.test_type, Iterable)
            and test_type in test_info.test_type
        )

    matching_tests = {}
    for test_name, test_info in test_settings.items():
# ... 13 lines omitted for brevity ...

            # Everything looks okay; keep if this is correct proc handling
            if (
                test_info.distributed_executor_backend is not None
            ) == new_proc_per_test:
                matching_tests[test_name] = test_info

    return matching_tests
```
**EN:** This helper encapsulates reusable logic in `get_filtered_test_settings`. Key inputs are `test_settings`, `test_type`, `new_proc_per_test`. It returns computed state or helper objects back to the caller. The main assertion is `test_info.convert_assets_to_embeddings is not None` and `test_info.custom_test_opts is not None and isinstance(test_info.custom_test_opts, Iterable)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_filtered_test_settings` 中。 关键输入包括 `test_settings`、`test_type`、`new_proc_per_test`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `test_info.convert_assets_to_embeddings is not None` and `test_info.custom_test_opts is not None and isinstance(test_info.custom_test_opts, Iterable)`。

### Helper / 辅助函数: get_model_type_cases (L65-L121)
```python
def get_model_type_cases(
    model_type: str,
    test_info: VLMTestInfo,
    test_type: VLMTestType,
):
    # Ensure that something is wrapped as an iterable it's not already
    ensure_wrapped = lambda e: e if isinstance(e, (list, tuple)) else (e,)

    # This is essentially the same as nesting a bunch of mark.parametrize
    # decorators, but we do it programmatically to allow overrides for on
    # a per-model basis, while still being able to execute each of these
    # as individual test cases in pytest.
    iter_kwargs = OrderedDict(
        [
            ("model", ensure_wrapped(test_info.models)),
            ("max_tokens", ensure_wrapped(test_info.max_tokens)),
            ("num_logprobs", ensure_wrapped(test_info.num_logprobs)),
            ("dtype", ensure_wrapped(test_info.dtype)),
# ... 31 lines omitted for brevity ...
    return [
        pytest.param(
            model_type,
            ExpandableVLMTestArgs(**{k: v for k, v in zip(iter_kwargs.keys(), case)}),
            marks=test_info.marks if test_info.marks is not None else [],
        )
        for case in list(itertools.product(*iter_kwargs.values()))
    ]
```
**EN:** This helper encapsulates reusable logic in `get_model_type_cases`. Key inputs are `model_type`, `test_info`, `test_type`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_model_type_cases` 中。 关键输入包括 `model_type`、`test_info`、`test_type`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_parametrized_options (L124-L145)
```python
def get_parametrized_options(
    test_settings: dict[str, VLMTestInfo],
    test_type: VLMTestType,
    create_new_process_for_each_test: bool,
):
    """Converts all of our VLMTestInfo into an expanded list of parameters.
    This is similar to nesting pytest parametrize calls, but done directly
    through an itertools product so that each test can set things like
    size factors etc, while still running in isolated test cases.
    """
    matching_tests = get_filtered_test_settings(
        test_settings, test_type, create_new_process_for_each_test
    )

    # Get a list per model type, where each entry contains a tuple of all of
    # that model type's cases, then flatten them into the top level so that
    # we can consume them in one mark.parametrize call.
    cases_by_model_type = [
        get_model_type_cases(model_type, test_info, test_type)
        for model_type, test_info in matching_tests.items()
    ]
    return list(itertools.chain(*cases_by_model_type))
```
**EN:** This helper encapsulates reusable logic in `get_parametrized_options`. Key inputs are `test_settings`, `test_type`, `create_new_process_for_each_test`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_parametrized_options` 中。 关键输入包括 `test_settings`、`test_type`、`create_new_process_for_each_test`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_wrapped_test_sizes (L148-L183)
```python
def get_wrapped_test_sizes(
    test_info: VLMTestInfo, test_type: VLMTestType
) -> tuple[ImageSizeWrapper, ...]:
    """Given a test info which may have size factors or fixed sizes, wrap them
    and combine them into an iterable, each of which will be used in parameter
    expansion.

    Args:
        test_info: Test configuration to be expanded.
        test_type: The type of test being filtered for.
    """
    # If it is an embedding test, we always use the EMBEDDING_SIZE_FACTORS
    if test_type == VLMTestType.EMBEDDING:
        return tuple(
            [
                ImageSizeWrapper(type=SizeType.SIZE_FACTOR, data=factor)
                for factor in EMBEDDING_SIZE_FACTORS
            ]
# ... 10 lines omitted for brevity ...
        for factor in size_factors
    ]

    wrapped_sizes = [
        ImageSizeWrapper(type=SizeType.FIXED_SIZE, data=size) for size in fixed_sizes
    ]

    return tuple(wrapped_factors + wrapped_sizes)
```
**EN:** This helper encapsulates reusable logic in `get_wrapped_test_sizes`. Key inputs are `test_info`, `test_type`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_wrapped_test_sizes` 中。 关键输入包括 `test_info`、`test_type`。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.OrderedDict`, `collections.abc.Iterable`, `itertools`
- **Third-party / 第三方**: `pytest`
- **Local relative imports / 本地相对导入**: `.types.EMBEDDING_SIZE_FACTORS`, `.types.ExpandableVLMTestArgs`, `.types.ImageSizeWrapper`, `.types.SizeType`, `.types.VLMTestInfo`, `.types.VLMTestType`

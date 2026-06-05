# choices.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/choices.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements part of SGLang's frontend language layer, such as tracing, intermediate representation, prompting utilities, or chat formatting. / 该文件实现了 SGLang 前端语言层的一部分，例如追踪、中间表示、提示工具或聊天格式化逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
from abc import ABC, abstractmethod
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python
from dataclasses import dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
from typing import Any, Dict, List, Optional
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
import numpy as np
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-8: Module-level supporting statements
```python


@dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: ChoicesDecision class declaration
```python
class ChoicesDecision:
```
**EN:** This block declares the `ChoicesDecision` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ChoicesDecision` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 10-10: Class-level supporting statements
```python
    decision: str
```
**EN:** This block contains supporting statements for the `ChoicesDecision` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChoicesDecision` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Class-level supporting statements
```python
    meta_info: Optional[Dict[str, Any]] = None
```
**EN:** This block contains supporting statements for the `ChoicesDecision` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChoicesDecision` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-13: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-16: ChoicesSamplingMethod class declaration
```python
class ChoicesSamplingMethod(ABC):

    @property
```
**EN:** This block declares the `ChoicesSamplingMethod` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ChoicesSamplingMethod` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 17-18: ChoicesSamplingMethod.requires_unconditional_logprobs method
```python
    def requires_unconditional_logprobs(self) -> bool:
        return False
```
**EN:** This block uses `ChoicesSamplingMethod.requires_unconditional_logprobs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChoicesSamplingMethod.requires_unconditional_logprobs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 19-20: Class-level supporting statements
```python

    @abstractmethod
```
**EN:** This block contains supporting statements for the `ChoicesSamplingMethod` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ChoicesSamplingMethod` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-29: ChoicesSamplingMethod.__call__ method
```python
    def __call__(
        self,
        *,
        choices: List[str],
        normalized_prompt_logprobs: List[float],
        input_token_logprobs: List[List[Any]],
        output_token_logprobs: List[List[Any]],
        unconditional_token_logprobs: Optional[List[List[Any]]] = None,
    ) -> ChoicesDecision: ...
```
**EN:** This block uses `ChoicesSamplingMethod.__call__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChoicesSamplingMethod.__call__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 30-31: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 32-33: TokenLengthNormalized class declaration
```python
class TokenLengthNormalized(ChoicesSamplingMethod):
```
**EN:** This block declares the `TokenLengthNormalized` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `TokenLengthNormalized` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 34-50: TokenLengthNormalized.__call__ method
```python
    def __call__(
        self,
        *,
        choices: List[str],
        normalized_prompt_logprobs: List[float],
        input_token_logprobs: List[List[Any]],
        output_token_logprobs: List[List[Any]],
        unconditional_token_logprobs: Optional[List[List[Any]]] = None,
    ) -> ChoicesDecision:
        """Select the option with the highest token length normalized prompt logprob."""
        best_choice = choices[np.argmax(normalized_prompt_logprobs)]
        meta_info = {
            "normalized_prompt_logprobs": normalized_prompt_logprobs,
            "input_token_logprobs": input_token_logprobs,
            "output_token_logprobs": output_token_logprobs,
        }
        return ChoicesDecision(decision=best_choice, meta_info=meta_info)
```
**EN:** This block uses `TokenLengthNormalized.__call__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TokenLengthNormalized.__call__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 51-52: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 53-53: Module-level supporting statements
```python
token_length_normalized = TokenLengthNormalized()
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 54-55: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-57: GreedyTokenSelection class declaration
```python
class GreedyTokenSelection(ChoicesSamplingMethod):
```
**EN:** This block declares the `GreedyTokenSelection` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `GreedyTokenSelection` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 58-85: GreedyTokenSelection.__call__ method
```python
    def __call__(
        self,
        *,
        choices: List[str],
        normalized_prompt_logprobs: List[float],
        input_token_logprobs: List[List[Any]],
        output_token_logprobs: List[List[Any]],
        unconditional_token_logprobs: Optional[List[List[Any]]] = None,
    ) -> ChoicesDecision:
        """Select the option based on greedy logprob selection. For overlapping options
        where one option is a subset of a longer option, extend the shorter option using
        its average logprob for comparison against the longer option."""

        num_options = len(choices)
        max_tokens = max(len(option) for option in input_token_logprobs)
        logprob_matrix = self._build_logprob_matrix(
            input_token_logprobs, max_tokens, num_options
        )
        remaining = self._greedy_selection(logprob_matrix, num_options, max_tokens)

        best_choice = choices[remaining[0]]
        meta_info = {
            "normalized_prompt_logprobs": normalized_prompt_logprobs,
            "input_token_logprobs": input_token_logprobs,
            "output_token_logprobs": output_token_logprobs,
            "greedy_logprob_matrix": logprob_matrix.tolist(),
        }
        return ChoicesDecision(decision=best_choice, meta_info=meta_info)
```
**EN:** This block uses `GreedyTokenSelection.__call__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GreedyTokenSelection.__call__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 86-86: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GreedyTokenSelection` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GreedyTokenSelection` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 87-95: GreedyTokenSelection._build_logprob_matrix method
```python
    def _build_logprob_matrix(self, input_token_logprobs, max_tokens, num_options):
        logprob_matrix = np.zeros((num_options, max_tokens))
        for i, option in enumerate(input_token_logprobs):
            actual_logprobs = [token[0] for token in option]
            avg_logprob = np.mean(actual_logprobs)
            logprob_matrix[i, : len(option)] = actual_logprobs
            if len(option) < max_tokens:
                logprob_matrix[i, len(option) :] = avg_logprob
        return logprob_matrix
```
**EN:** This block uses `GreedyTokenSelection._build_logprob_matrix` to build helper structures or requests. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GreedyTokenSelection._build_logprob_matrix` 来构建辅助结构或请求。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 96-96: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GreedyTokenSelection` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GreedyTokenSelection` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 97-104: GreedyTokenSelection._greedy_selection method
```python
    def _greedy_selection(self, logprob_matrix, num_options, max_tokens):
        remaining = np.arange(num_options)
        for j in range(max_tokens):
            max_logprob = np.max(logprob_matrix[remaining, j])
            remaining = remaining[logprob_matrix[remaining, j] == max_logprob]
            if len(remaining) == 1:
                break
        return remaining
```
**EN:** This block uses `GreedyTokenSelection._greedy_selection` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GreedyTokenSelection._greedy_selection` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 105-106: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 107-107: Module-level supporting statements
```python
greedy_token_selection = GreedyTokenSelection()
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 108-109: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 110-112: UnconditionalLikelihoodNormalized class declaration
```python
class UnconditionalLikelihoodNormalized(ChoicesSamplingMethod):

    @property
```
**EN:** This block declares the `UnconditionalLikelihoodNormalized` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `UnconditionalLikelihoodNormalized` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 113-114: UnconditionalLikelihoodNormalized.requires_unconditional_logprobs method
```python
    def requires_unconditional_logprobs(self) -> bool:
        return True
```
**EN:** This block uses `UnconditionalLikelihoodNormalized.requires_unconditional_logprobs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `UnconditionalLikelihoodNormalized.requires_unconditional_logprobs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 115-115: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `UnconditionalLikelihoodNormalized` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`UnconditionalLikelihoodNormalized` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 116-148: UnconditionalLikelihoodNormalized.__call__ method
```python
    def __call__(
        self,
        *,
        choices: List[str],
        normalized_prompt_logprobs: List[float],
        input_token_logprobs: List[List[Any]],
        output_token_logprobs: List[List[Any]],
        unconditional_token_logprobs: Optional[List[List[Any]]] = None,
    ) -> ChoicesDecision:
        """Select the option with the highest average token logprob once normalized by
        the unconditional token logprobs.

        The first unconditional token logprob is assumed to be None. If so, it is
        replaced with 0 for the purposes of normalization."""

        if unconditional_token_logprobs is None:
            raise ValueError(
                "Unconditional token logprobs are required for this method."
            )

        normalized_unconditional_prompt_logprobs = self._normalize_logprobs(
            input_token_logprobs, unconditional_token_logprobs
        )

        best_choice = choices[np.argmax(normalized_unconditional_prompt_logprobs)]
        meta_info = {
            "normalized_prompt_logprobs": normalized_prompt_logprobs,
            "input_token_logprobs": input_token_logprobs,
            "output_token_logprobs": output_token_logprobs,
            "unconditional_token_logprobs": unconditional_token_logprobs,
            "normalized_unconditional_prompt_logprobs": normalized_unconditional_prompt_logprobs,
        }
        return ChoicesDecision(decision=best_choice, meta_info=meta_info)
```
**EN:** This block uses `UnconditionalLikelihoodNormalized.__call__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `UnconditionalLikelihoodNormalized.__call__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 149-149: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `UnconditionalLikelihoodNormalized` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`UnconditionalLikelihoodNormalized` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 150-161: UnconditionalLikelihoodNormalized._normalize_logprobs method
```python
    def _normalize_logprobs(self, input_token_logprobs, unconditional_token_logprobs):
        normalized_unconditional_prompt_logprobs = []
        for inputs, unconditionals in zip(
            input_token_logprobs, unconditional_token_logprobs
        ):
            inputs_logprobs = np.array([token[0] for token in inputs])
            unconditionals_logprobs = np.array([token[0] for token in unconditionals])
            unconditionals_logprobs[0] = unconditionals_logprobs[0] or 0
            normalized_unconditional_prompt_logprobs.append(
                float(np.mean(inputs_logprobs - unconditionals_logprobs))
            )
        return normalized_unconditional_prompt_logprobs
```
**EN:** This block uses `UnconditionalLikelihoodNormalized._normalize_logprobs` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `UnconditionalLikelihoodNormalized._normalize_logprobs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 162-163: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 164-164: Module-level supporting statements
```python
unconditional_likelihood_normalized = UnconditionalLikelihoodNormalized()
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Numerical data processing / 数值数据处理

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `numpy`
- `abc` (stdlib)
- `dataclasses` (stdlib)
- `typing` (stdlib)

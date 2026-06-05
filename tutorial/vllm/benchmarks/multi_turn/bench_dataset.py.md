# bench_dataset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/multi_turn/bench_dataset.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements dataset preparation logic, benchmark orchestration, centered around `Distribution`, `UniformDistribution`, `ConstantDistribution`, `ZipfDistribution`. / 实现与数据集准备逻辑、基准测试编排相关的逻辑，核心符号包括 `Distribution`, `UniformDistribution`, `ConstantDistribution`, `ZipfDistribution`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-15)
```python
from abc import ABC, abstractmethod
from statistics import mean
from typing import Any, NamedTuple

import numpy as np  # type: ignore
import pandas as pd  # type: ignore
from bench_utils import (
    TEXT_SEPARATOR,
    Color,
    logger,
)
from tqdm import tqdm
from transformers import AutoTokenizer  # type: ignore
```
**EN:** This block gathers standard-library helpers such as `abc`, `statistics`, `typing`; third-party packages such as `numpy`, `pandas`, `tqdm`, `transformers`; project-local modules such as `bench_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `abc`, `statistics`, `typing`；第三方依赖，如 `numpy`, `pandas`, `tqdm`, `transformers`；项目内部模块，如 `bench_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 18-27)
```python
ConvId = str

# A list of dicts (dicts with keys "id" and "messages")
ShareGptConversations = list[dict[str, Any]]

# A list of dicts (dicts with keys "role" and "content")
MessagesList = list[dict[str, str]]

# Map conversation ID to conversation messages
ConversationsMap = list[ConvId, MessagesList]
```
**EN:** This top-level block prepares shared state such as `ConvId`, `ShareGptConversations`, `MessagesList`, `ConversationsMap`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `ConvId`, `ShareGptConversations`, `MessagesList`, `ConversationsMap`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `Distribution` (lines 30-33)
```python
class Distribution(ABC):
    @abstractmethod
    def sample(self, size: int = 1) -> np.ndarray:
        pass
```
**EN:** Class `Distribution` packages shared state and related operations for this benchmark module. It extends `ABC` and exposes methods such as `sample`.
**CN:** 类 `Distribution` 用于封装该基准模块的共享状态与相关操作。它继承自 `ABC`，并提供 `sample` 等方法。

### Method `Distribution.sample` (lines 32-33)
```python
    def sample(self, size: int = 1) -> np.ndarray:
        pass
```
**EN:** `sample` implements a helper used by `bench_dataset.py`. It mainly works with `size` and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `sample` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `size`，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Class `UniformDistribution` (lines 36-56)
```python
class UniformDistribution(Distribution):
    def __init__(
        self,
        min_val: int | float,
        max_val: int | float,
        is_integer: bool = True,
    ) -> None:
        self.min_val = min_val
        self.max_val = max_val
        self.is_integer = is_integer

    def sample(self, size: int = 1) -> np.ndarray:
        if self.is_integer:
            return np.random.randint(
                int(self.min_val), int(self.max_val + 1), size=size
            )
        else:
            return np.random.uniform(self.min_val, self.max_val, size=size)

    def __repr__(self) -> str:
        return f"UniformDistribution[{self.min_val}, {self.max_val}]"
```
**EN:** Class `UniformDistribution` packages shared state and related operations for this benchmark module. It extends `Distribution` and exposes methods such as `__init__`, `sample`, `__repr__`.
**CN:** 类 `UniformDistribution` 用于封装该基准模块的共享状态与相关操作。它继承自 `Distribution`，并提供 `__init__`, `sample`, `__repr__` 等方法。

### Method `UniformDistribution.__init__` (lines 37-45)
```python
    def __init__(
        self,
        min_val: int | float,
        max_val: int | float,
        is_integer: bool = True,
    ) -> None:
        self.min_val = min_val
        self.max_val = max_val
        self.is_integer = is_integer
```
**EN:** `__init__` implements a helper used by `bench_dataset.py`. It mainly works with `min_val`, `max_val`, `is_integer` and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `min_val`, `max_val`, `is_integer`，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `UniformDistribution.sample` (lines 47-53)
```python
    def sample(self, size: int = 1) -> np.ndarray:
        if self.is_integer:
            return np.random.randint(
                int(self.min_val), int(self.max_val + 1), size=size
            )
        else:
            return np.random.uniform(self.min_val, self.max_val, size=size)
```
**EN:** `sample` implements a helper used by `bench_dataset.py`. It mainly works with `size` and relies on `np.random.randint`, `int`, `np.random.uniform` plus branching to move data through this part of the benchmark pipeline.
**CN:** `sample` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `size`，并结合 `np.random.randint`, `int`, `np.random.uniform` 以及 条件分支 来完成这一段基准测试流程。

### Class `ConstantDistribution` (lines 59-68)
```python
class ConstantDistribution(Distribution):
    def __init__(self, value: int | float) -> None:
        self.value = value
        self.max_val = value

    def sample(self, size: int = 1) -> np.ndarray:
        return np.full(shape=size, fill_value=self.value)

    def __repr__(self) -> str:
        return f"Constant[{self.value}]"
```
**EN:** Class `ConstantDistribution` packages shared state and related operations for this benchmark module. It extends `Distribution` and exposes methods such as `__init__`, `sample`, `__repr__`.
**CN:** 类 `ConstantDistribution` 用于封装该基准模块的共享状态与相关操作。它继承自 `Distribution`，并提供 `__init__`, `sample`, `__repr__` 等方法。

### Method `ConstantDistribution.__init__` (lines 60-62)
```python
    def __init__(self, value: int | float) -> None:
        self.value = value
        self.max_val = value
```
**EN:** `__init__` implements a helper used by `bench_dataset.py`. It mainly works with `value` and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `value`，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `ConstantDistribution.sample` (lines 64-65)
```python
    def sample(self, size: int = 1) -> np.ndarray:
        return np.full(shape=size, fill_value=self.value)
```
**EN:** `sample` implements a helper used by `bench_dataset.py`. It mainly works with `size` and relies on `np.full` plus value production to move data through this part of the benchmark pipeline.
**CN:** `sample` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `size`，并结合 `np.full` 以及 结果返回 来完成这一段基准测试流程。

### Class `ZipfDistribution` (lines 71-83)
```python
class ZipfDistribution(Distribution):
    def __init__(self, alpha: float, max_val: int | None = None) -> None:
        self.alpha = alpha
        self.max_val = max_val

    def sample(self, size: int = 1) -> np.ndarray:
        samples = np.random.zipf(self.alpha, size=size)
        if self.max_val:
            samples = np.minimum(samples, self.max_val)
        return samples

    def __repr__(self) -> str:
        return f"ZipfDistribution[{self.alpha}]"
```
**EN:** Class `ZipfDistribution` packages shared state and related operations for this benchmark module. It extends `Distribution` and exposes methods such as `__init__`, `sample`, `__repr__`.
**CN:** 类 `ZipfDistribution` 用于封装该基准模块的共享状态与相关操作。它继承自 `Distribution`，并提供 `__init__`, `sample`, `__repr__` 等方法。

### Method `ZipfDistribution.__init__` (lines 72-74)
```python
    def __init__(self, alpha: float, max_val: int | None = None) -> None:
        self.alpha = alpha
        self.max_val = max_val
```
**EN:** `__init__` implements a helper used by `bench_dataset.py`. It mainly works with `alpha`, `max_val` and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `alpha`, `max_val`，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `ZipfDistribution.sample` (lines 76-80)
```python
    def sample(self, size: int = 1) -> np.ndarray:
        samples = np.random.zipf(self.alpha, size=size)
        if self.max_val:
            samples = np.minimum(samples, self.max_val)
        return samples
```
**EN:** `sample` implements a helper used by `bench_dataset.py`. It mainly works with `size` and relies on `np.random.zipf`, `np.minimum` plus branching to move data through this part of the benchmark pipeline.
**CN:** `sample` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `size`，并结合 `np.random.zipf`, `np.minimum` 以及 条件分支 来完成这一段基准测试流程。

### Class `PoissonDistribution` (lines 86-98)
```python
class PoissonDistribution(Distribution):
    def __init__(self, alpha: float, max_val: int | None = None) -> None:
        self.alpha = alpha
        self.max_val = max_val

    def sample(self, size: int = 1) -> np.ndarray:
        samples = np.random.poisson(self.alpha, size=size)
        if self.max_val:
            samples = np.minimum(samples, self.max_val)
        return samples

    def __repr__(self) -> str:
        return f"PoissonDistribution[{self.alpha}]"
```
**EN:** Class `PoissonDistribution` packages shared state and related operations for this benchmark module. It extends `Distribution` and exposes methods such as `__init__`, `sample`, `__repr__`.
**CN:** 类 `PoissonDistribution` 用于封装该基准模块的共享状态与相关操作。它继承自 `Distribution`，并提供 `__init__`, `sample`, `__repr__` 等方法。

### Method `PoissonDistribution.__init__` (lines 87-89)
```python
    def __init__(self, alpha: float, max_val: int | None = None) -> None:
        self.alpha = alpha
        self.max_val = max_val
```
**EN:** `__init__` implements a helper used by `bench_dataset.py`. It mainly works with `alpha`, `max_val` and relies on direct statements plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `alpha`, `max_val`，并结合 direct statements 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `PoissonDistribution.sample` (lines 91-95)
```python
    def sample(self, size: int = 1) -> np.ndarray:
        samples = np.random.poisson(self.alpha, size=size)
        if self.max_val:
            samples = np.minimum(samples, self.max_val)
        return samples
```
**EN:** `sample` implements a helper used by `bench_dataset.py`. It mainly works with `size` and relies on `np.random.poisson`, `np.minimum` plus branching to move data through this part of the benchmark pipeline.
**CN:** `sample` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `size`，并结合 `np.random.poisson`, `np.minimum` 以及 条件分支 来完成这一段基准测试流程。

### Class `LognormalDistribution` (lines 101-201)
```python
class LognormalDistribution(Distribution):
    def __init__(
        self,
        mean: float | None = None,
        sigma: float | None = None,
        average: int | None = None,
        median_ratio: float | None = None,
        max_val: int | None = None,
    ) -> None:
        self.average = average
        self.median_ratio = median_ratio
        self.max_val = max_val

        if average is not None:
            if average < 1:
                raise ValueError("Lognormal average must be positive")

            if mean or sigma:
                raise ValueError(
                    "When using lognormal average, you can't provide mean/sigma"
                )

            if self.median_ratio is None:
                # Default value that provides relatively wide range of values
                self.median_ratio = 0.85

            # Calculate mean/sigma of np.random.lognormal based on the average
            mean, sigma = self._generate_lognormal_by_median(
                target_average=self.average, median_ratio=self.median_ratio
            )
        else:
            if mean is None or sigma is None:
                raise ValueError(
                    "Must provide both mean and sigma if average is not used"
    # ... omitted for brevity ...

    def __repr__(self) -> str:
        if self.average:
            return (
                f"LognormalDistribution[{self.average}, "
                f"{self.median_ratio}, {self.max_val}]"
            )
        return f"LognormalDistribution[{self.mean}, {self.sigma}, {self.max_val}]"
```
**EN:** Class `LognormalDistribution` packages shared state and related operations for this benchmark module. It extends `Distribution` and exposes methods such as `__init__`, `_generate_lognormal_by_median`, `sample`, `__repr__`.
**CN:** 类 `LognormalDistribution` 用于封装该基准模块的共享状态与相关操作。它继承自 `Distribution`，并提供 `__init__`, `_generate_lognormal_by_median`, `sample`, `__repr__` 等方法。

### Method `LognormalDistribution.__init__` (lines 102-145)
```python
    def __init__(
        self,
        mean: float | None = None,
        sigma: float | None = None,
        average: int | None = None,
        median_ratio: float | None = None,
        max_val: int | None = None,
    ) -> None:
        self.average = average
        self.median_ratio = median_ratio
        self.max_val = max_val

        if average is not None:
            if average < 1:
                raise ValueError("Lognormal average must be positive")

            if mean or sigma:
                raise ValueError(
                    "When using lognormal average, you can't provide mean/sigma"
                )

            if self.median_ratio is None:
                # Default value that provides relatively wide range of values
                self.median_ratio = 0.85

            # Calculate mean/sigma of np.random.lognormal based on the average
            mean, sigma = self._generate_lognormal_by_median(
                target_average=self.average, median_ratio=self.median_ratio
            )
        else:
            if mean is None or sigma is None:
                raise ValueError(
                    "Must provide both mean and sigma if average is not used"
                )

            if mean <= 0 or sigma < 0:
                raise ValueError(
                    "Lognormal mean must be positive and sigma must be non-negative"
                )

        # Mean and standard deviation of the underlying normal distribution
        # Based on numpy.random.lognormal
        self.mean = mean
        self.sigma = sigma
```
**EN:** `__init__` implements a helper used by `bench_dataset.py`. It mainly works with `mean`, `sigma`, `average`, `median_ratio`, `max_val` and relies on `ValueError`, `self._generate_lognormal_by_median` plus branching to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `mean`, `sigma`, `average`, `median_ratio`, `max_val`，并结合 `ValueError`, `self._generate_lognormal_by_median` 以及 条件分支 来完成这一段基准测试流程。

### Method `LognormalDistribution.sample` (lines 183-193)
```python
    def sample(self, size: int = 1) -> np.ndarray:
        samples = np.random.lognormal(mean=self.mean, sigma=self.sigma, size=size)

        if self.average is not None:
            # Scale to average
            samples *= self.average / samples.mean()

        if self.max_val:
            samples = np.minimum(samples, self.max_val)

        return np.round(samples).astype(int)
```
**EN:** `sample` implements a helper used by `bench_dataset.py`. It mainly works with `size` and relies on `np.random.lognormal`, `samples.mean`, `np.minimum`, `np.round.astype`, `np.round` plus branching to move data through this part of the benchmark pipeline.
**CN:** `sample` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `size`，并结合 `np.random.lognormal`, `samples.mean`, `np.minimum`, `np.round.astype`, `np.round` 以及 条件分支 来完成这一段基准测试流程。

### Class `GenConvArgs` (lines 204-212)
```python
class GenConvArgs(NamedTuple):
    num_conversations: int
    text_files: list[str]
    input_num_turns: Distribution
    input_common_prefix_num_tokens: Distribution
    input_prefix_num_tokens: Distribution
    input_num_tokens: Distribution
    output_num_tokens: Distribution
    print_stats: bool
```
**EN:** Class `GenConvArgs` packages shared state and related operations for this benchmark module. It extends `NamedTuple` and exposes methods such as its methods.
**CN:** 类 `GenConvArgs` 用于封装该基准模块的共享状态与相关操作。它继承自 `NamedTuple`，并提供 its methods 等方法。

### Function `verify_field_exists` (lines 215-221)
```python
def verify_field_exists(
    conf: dict, field_name: str, section: str, subsection: str
) -> None:
    if field_name not in conf:
        raise ValueError(
            f"Missing field '{field_name}' in {section=} and {subsection=}"
        )
```
**EN:** `verify_field_exists` implements a helper used by `bench_dataset.py`. It mainly works with `conf`, `field_name`, `section`, `subsection` and relies on `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `verify_field_exists` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `conf`, `field_name`, `section`, `subsection`，并结合 `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_random_distribution` (lines 224-290)
```python
def get_random_distribution(
    conf: dict, section: str, subsection: str, optional: bool = False
) -> Distribution:
    # section can be "prompt_input" or "prompt_output" (both required)
    conf = conf[section]

    if optional and subsection not in conf:
        # Optional subsection, if not found assume the value is always 0
        return ConstantDistribution(0)

    # subsection can be "num_turns", "num_tokens" or "prefix_num_tokens"
    if subsection not in conf:
        raise ValueError(f"Missing subsection {subsection} in section {section}")

    conf = conf[subsection]

    distribution = conf.get("distribution")
    if distribution is None:
        raise ValueError(
            f"Missing field 'distribution' in {section=} and {subsection=}"
        )

    if distribution == "constant":
        verify_field_exists(conf, "value", section, subsection)
        return ConstantDistribution(conf["value"])

    elif distribution == "zipf":
        verify_field_exists(conf, "alpha", section, subsection)
        max_val = conf.get("max", None)
        return ZipfDistribution(conf["alpha"], max_val=max_val)

    elif distribution == "poisson":
        verify_field_exists(conf, "alpha", section, subsection)
        max_val = conf.get("max", None)
    # ... omitted for brevity ...

        assert min_value > 0
        assert min_value <= max_value

        is_integer = isinstance(min_value, int) and isinstance(max_value, int)
        return UniformDistribution(min_value, max_value, is_integer)
    else:
        raise ValueError(f"Unknown distribution: {distribution}")
```
**EN:** `get_random_distribution` implements a helper used by `bench_dataset.py`. It mainly works with `conf`, `section`, `subsection`, `optional` and relies on `ConstantDistribution`, `ValueError`, `conf.get`, `verify_field_exists`, `ZipfDistribution`, `PoissonDistribution` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_random_distribution` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `conf`, `section`, `subsection`, `optional`，并结合 `ConstantDistribution`, `ValueError`, `conf.get`, `verify_field_exists`, `ZipfDistribution`, `PoissonDistribution` 以及 条件分支 来完成这一段基准测试流程。

### Function `parse_input_json_file` (lines 293-343)
```python
def parse_input_json_file(conf: dict) -> GenConvArgs:
    # Validate the input file
    assert isinstance(conf, dict)
    required_fields = [
        "filetype",
        "num_conversations",
        "text_files",
        "prompt_input",
        "prompt_output",
    ]
    for field in required_fields:
        assert field in conf, f"Missing field {field} in input {conf}"

    assert conf["filetype"] == "generate_conversations"

    assert conf["num_conversations"] > 0, "num_conversations should be larger than zero"

    text_files = conf["text_files"]

    assert isinstance(text_files, list), "Field 'text_files' should be a list"
    assert len(text_files) > 0, (
        "Field 'text_files' should be a list with at least one file"
    )

    # Parse the parameters for the prompt input/output workload
    input_num_turns = get_random_distribution(conf, "prompt_input", "num_turns")
    input_num_tokens = get_random_distribution(conf, "prompt_input", "num_tokens")
    input_common_prefix_num_tokens = get_random_distribution(
        conf, "prompt_input", "common_prefix_num_tokens", optional=True
    )
    input_prefix_num_tokens = get_random_distribution(
        conf, "prompt_input", "prefix_num_tokens"
    )
    output_num_tokens = get_random_distribution(conf, "prompt_output", "num_tokens")
    # ... omitted for brevity ...
        input_num_turns=input_num_turns,
        input_common_prefix_num_tokens=input_common_prefix_num_tokens,
        input_prefix_num_tokens=input_prefix_num_tokens,
        input_num_tokens=input_num_tokens,
        output_num_tokens=output_num_tokens,
        print_stats=print_stats,
    )
    return args
```
**EN:** `parse_input_json_file` parses external input into structured benchmark settings. It mainly works with `conf` and relies on `isinstance`, `len`, `get_random_distribution`, `conf.get`, `GenConvArgs` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `parse_input_json_file` 负责把外部输入解析为结构化的基准配置。 它主要处理 `conf`，并结合 `isinstance`, `len`, `get_random_distribution`, `conf.get`, `GenConvArgs` 以及 循环迭代 来完成这一段基准测试流程。

### Function `print_conv_stats` (lines 346-397)
```python
def print_conv_stats(conversations: ConversationsMap, tokenizer: AutoTokenizer) -> None:
    # Collect statistics
    conv_stats: list[dict[Any, Any]] = []
    req_stats: list[int] = []

    print("\nCollecting statistics...")
    for messages in conversations.values():
        # messages is a list of dicts
        user_tokens: list[int] = []
        assistant_tokens: list[int] = []
        request_tokens: list[int] = []

        req_tokens = 0
        for m in messages:
            content = m["content"]
            num_tokens = len(tokenizer(content).input_ids)

            if m["role"] == "user":
                user_tokens.append(num_tokens)
                # New user prompt including all chat history
                req_tokens += num_tokens
                request_tokens.append(req_tokens)

            elif m["role"] == "assistant":
                assistant_tokens.append(num_tokens)
                # Update assistant answer
                # (will be part of chat history for the next user prompt)
                req_tokens += num_tokens

        item_stats = {
            "conversation_turns": len(messages),
            "user_tokens": mean(user_tokens),
            "assistant_tokens": mean(assistant_tokens),
        }
    # ... omitted for brevity ...
    df = pd.DataFrame(conv_stats)
    print(df.describe(percentiles=percentiles).transpose())
    print(TEXT_SEPARATOR)
    print(f"{Color.YELLOW}Request statistics:{Color.RESET}")
    print(TEXT_SEPARATOR)
    df = pd.DataFrame(req_stats, columns=["request_tokens"])
    print(df.describe(percentiles=percentiles).transpose())
    print(TEXT_SEPARATOR)
```
**EN:** `print_conv_stats` formats results for display or export. It mainly works with `conversations`, `tokenizer` and relies on `print`, `conversations.values`, `len`, `tokenizer`, `user_tokens.append`, `request_tokens.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_conv_stats` 负责格式化结果以便展示或导出。 它主要处理 `conversations`, `tokenizer`，并结合 `print`, `conversations.values`, `len`, `tokenizer`, `user_tokens.append`, `request_tokens.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `generate_conversations` (lines 400-553)
```python
def generate_conversations(
    args: GenConvArgs, tokenizer: AutoTokenizer
) -> ConversationsMap:
    # Text for all user prompts
    # (text from the input text files will be appended to this line)
    base_prompt_text = "Please rewrite the following text and add more content: "
    base_prompt_token_count = len(
        tokenizer.encode(base_prompt_text, add_special_tokens=False)
    )

    logger.info(f"{Color.PURPLE}Generating conversations...{Color.RESET}")
    logger.info(args)

    list_of_tokens = []

    for filename in args.text_files:
        # Load text file that will be used to generate prompts
        with open(filename) as file:
            data = file.read()
            tokens_in_file = tokenizer.encode(data, add_special_tokens=False)
            list_of_tokens.extend(tokens_in_file)
        logger.info(
            f"Loaded {len(tokens_in_file)} tokens from file {filename}, "
            f"total tokens so far: {len(list_of_tokens)}"
        )

    conversations: ConversationsMap = {}
    conv_id = 0

    # Generate number of turns for every conversation
    turn_count: np.ndarray = args.input_num_turns.sample(args.num_conversations)

    # Turn count should be at least 2 (one user prompt and one assistant answer)
    turn_count = np.maximum(turn_count, 2)
    # ... omitted for brevity ...

        # Increase base offset for the next conversation
        base_offset += nturns

    if args.print_stats:
        print_conv_stats(conversations, tokenizer)

    return conversations
```
**EN:** `generate_conversations` implements a helper used by `bench_dataset.py`. It mainly works with `args`, `tokenizer` and relies on `len`, `tokenizer.encode`, `logger.info`, `open`, `file.read`, `list_of_tokens.extend` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `generate_conversations` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `args`, `tokenizer`，并结合 `len`, `tokenizer.encode`, `logger.info`, `open`, `file.read`, `list_of_tokens.extend` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `conversations_list_to_dict` (lines 556-591)
```python
def conversations_list_to_dict(input_list: ShareGptConversations) -> ConversationsMap:
    conversations: ConversationsMap = {}

    for item in input_list:
        conv_id: str = item["id"]
        assert isinstance(conv_id, str)

        assert conv_id not in conversations, (
            f"Conversation ID {conv_id} found more than once in the input"
        )

        messages: MessagesList = item["messages"]
        assert isinstance(messages, list), (
            f"Conversation messages should be a list (ID: {conv_id})"
        )
        assert len(messages) > 0, f"Conversation with no messages (ID: {conv_id})"

        conversations[conv_id] = messages

    logger.info(f"Using {len(conversations)} unique conversations (IDs)")
    assert len(conversations) == len(input_list)

    # Print statistics about the selected conversations
    stats: list[dict[str, Any]] = []
    for conv_data in conversations.values():
        stats.append({"num_turns": len(conv_data)})

    print(TEXT_SEPARATOR)
    print(f"{Color.YELLOW}Conversations statistics:{Color.RESET}")
    print(TEXT_SEPARATOR)
    percentiles = [0.25, 0.5, 0.75, 0.9, 0.99, 0.999, 0.9999]
    conv_stats = pd.DataFrame(stats).describe(percentiles=percentiles)
    print(conv_stats.transpose())
    print(TEXT_SEPARATOR)

    return conversations
```
**EN:** `conversations_list_to_dict` implements a helper used by `bench_dataset.py`. It mainly works with `input_list` and relies on `isinstance`, `len`, `logger.info`, `conversations.values`, `stats.append`, `print` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `conversations_list_to_dict` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `input_list`，并结合 `isinstance`, `len`, `logger.info`, `conversations.values`, `stats.append`, `print` 以及 循环迭代 来完成这一段基准测试流程。

### Function `conversations_dict_to_list` (lines 594-600)
```python
def conversations_dict_to_list(input_dict: ConversationsMap) -> ShareGptConversations:
    output: ShareGptConversations = []
    for conv_id, conv_data in input_dict.items():
        new_item = {"id": conv_id, "messages": conv_data}
        output.append(new_item)

    return output
```
**EN:** `conversations_dict_to_list` implements a helper used by `bench_dataset.py`. It mainly works with `input_dict` and relies on `input_dict.items`, `output.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `conversations_dict_to_list` 负责实现 `bench_dataset.py` 使用的辅助逻辑。 它主要处理 `input_dict`，并结合 `input_dict.items`, `output.append` 以及 循环迭代 来完成这一段基准测试流程。

## Key Concepts / 关键概念
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。

## Dependencies / 依赖关系
- **EN:** Standard library: `abc`, `statistics`, `typing`.
- **CN:** 标准库依赖：`abc`, `statistics`, `typing`。
- **EN:** Third-party packages: `numpy`, `pandas`, `tqdm`, `transformers`.
- **CN:** 第三方依赖：`numpy`, `pandas`, `tqdm`, `transformers`。
- **EN:** Internal modules: `bench_utils`.
- **CN:** 内部模块：`bench_utils`。

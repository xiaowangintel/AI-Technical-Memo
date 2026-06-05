# datasets.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/datasets/datasets.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This module defines a framework for sampling benchmark requests from various datasets. / 该文件的核心目的为：This module defines a framework for sampling benchmark requests from various datasets.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-13)
```python
"""
This module defines a framework for sampling benchmark requests from various
datasets. Each dataset subclass of BenchmarkDataset must implement sample
generation. Supported dataset types include:
  - ShareGPT
  - Random (synthetic)
  - Sonnet
  - BurstGPT
  - HuggingFace
  - VisionArena
"""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 15-50)
```python
import argparse

import ast

import io

import json

import logging

import math

import random

from abc import ABC, abstractmethod

from collections.abc import Callable, Iterator, Mapping

from contextlib import suppress

from dataclasses import dataclass, replace

from functools import cache

from io import BytesIO

from pathlib import Path

from tempfile import NamedTemporaryFile

from typing import Any, cast

import numpy as np

import pybase64 as base64
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 69-3425)
```python
logger = logging.getLogger(__name__)

DEFAULT_NUM_PROMPTS = 1000

lora_tokenizer_cache: dict[int, TokenizerLike] = {}

zeta_prompt = """### Instruction:
You are a code completion assistant and your task is to analyze user edits and then rewrite an excerpt that the user provides, suggesting the appropriate edits within the excerpt, taking into account the cursor location.

### User Edits:

{}

### User Excerpt:

{}

### Response:

"""  # noqa: E501
```
**EN:** This constant/configuration block defines `logger`, `DEFAULT_NUM_PROMPTS`, `lora_tokenizer_cache`, `zeta_prompt`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `DEFAULT_NUM_PROMPTS`, `lora_tokenizer_cache`, `zeta_prompt`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Large-file note
```python
# Focused on representative top-level definitions for a large module.
# The full file contains additional helpers following the same patterns.
```
**EN:** Because the file is large, the analysis below focuses on the most important top-level definitions and leaves repetitive helpers summarized at a higher level.
**CN:** 由于文件较大，下面的分析聚焦最重要的顶层定义；其余重复性辅助逻辑将以更高层次方式概括。

### Class `SampleRequest` (lines 75-85)
```python
class SampleRequest:
    """
    Represents a single inference request for benchmarking.
    """

    prompt: str | list[str] | list[dict]
    prompt_len: int
    expected_output_len: int | None
    multi_modal_data: MultiModalDataDict | dict | list[dict] | None = None
    lora_request: LoRARequest | None = None
    request_id: str | None = None
```
**EN:** Class `SampleRequest` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. The class docstring says: Represents a single inference request for benchmarking.
**CN:** 类 `SampleRequest` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 类文档说明：Represents a single inference request for benchmarking.

### Class `BenchmarkDataset` (lines 93-320)
```python
class BenchmarkDataset(ABC):
    DEFAULT_SEED = 0
    IS_MULTIMODAL = False

    def __init__(
        self,
        dataset_path: str | None = None,
        random_seed: int = DEFAULT_SEED,
        disable_shuffle: bool = False,
        **kwargs,
    ) -> None:
        """
        Initialize the BenchmarkDataset with an optional dataset path and random
        seed.

        Args:
            dataset_path (Optional[str]): Path to the dataset. If None, it
                indicates that a default or random dataset might be used.
            random_seed (int): Seed value for reproducible shuffling or
                sampling. Defaults to DEFAULT_SEED.
        """
        self.dataset_path = dataset_path
        # Set the random seed, ensuring that a None value is replaced with the
    # ... omitted for brevity ...
                "is unique."
            )
```
**EN:** Class `BenchmarkDataset` is a structured building block in this module. It inherits from `ABC`. Key methods include `__init__`, `apply_multimodal_chat_transformation`, `load_data`, `get_random_lora_request`, `get_round_robin_lora_request`, `get_lora_request`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `BenchmarkDataset` 是该模块中的结构化构件，继承自 `ABC`。 关键方法包括 `__init__`, `apply_multimodal_chat_transformation`, `load_data`, `get_random_lora_request`, `get_round_robin_lora_request`, `get_lora_request`，它们共同定义初始化、校验、变换或访问模式。

### Method `BenchmarkDataset.__init__` (lines 97-119)
```python
    def __init__(
        self,
        dataset_path: str | None = None,
        random_seed: int = DEFAULT_SEED,
        disable_shuffle: bool = False,
        **kwargs,
    ) -> None:
        """
        Initialize the BenchmarkDataset with an optional dataset path and random
        seed.

        Args:
            dataset_path (Optional[str]): Path to the dataset. If None, it
                indicates that a default or random dataset might be used.
            random_seed (int): Seed value for reproducible shuffling or
                sampling. Defaults to DEFAULT_SEED.
        """
        self.dataset_path = dataset_path
        # Set the random seed, ensuring that a None value is replaced with the
    # ... omitted for brevity ...
        self.disable_shuffle = disable_shuffle
        self.data: Any | None = None
```
**EN:** Method `BenchmarkDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. The docstring highlights: Initialize the BenchmarkDataset with an optional dataset path and random seed.
**CN:** Method `BenchmarkDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 文档字符串强调：Initialize the BenchmarkDataset with an optional dataset path and random seed.

### Method `BenchmarkDataset.get_random_lora_request` (lines 156-187)
```python
    def get_random_lora_request(
        self,
        max_loras: int | None = None,
        lora_path: str | None = None,
    ) -> LoRARequest | None:
        """
        Optionally select a random LoRA request.

        This method is used when LoRA parameters are provided.  It randomly
        selects a LoRA based on max_loras.

        Args:
            max_loras (Optional[int]): The maximum number of LoRAs available.
                If `None`, LoRA is not used.
            lora_path (Optional[str]): Path to the LoRA parameters on disk.
                If `None`, LoRA is not used.

        Returns:
            A new [`LoRARequest`][vllm.lora.request.LoRARequest]
    # ... omitted for brevity ...
        )
        return lora_request
```
**EN:** Method `BenchmarkDataset.get_random_lora_request` samples, slices, or reshapes data for downstream use. The docstring highlights: Optionally select a random LoRA request. Key calls such as `random.randint`, `LoRARequest`, `str`, `lora_path_on_disk` show the concrete execution path.
**CN:** Method `BenchmarkDataset.get_random_lora_request` 负责为下游流程采样、切片或重组数据。 文档字符串强调：Optionally select a random LoRA request. 像 `random.randint`, `LoRARequest`, `str`, `lora_path_on_disk` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BenchmarkDataset.get_round_robin_lora_request` (lines 189-222)
```python
    def get_round_robin_lora_request(
        self,
        index: int,
        max_loras: int | None = None,
        lora_path: str | None = None,
    ) -> LoRARequest | None:
        """
        Optionally select a LoRA request using deterministic round-robin.

        This method cycles through LoRA IDs in order based on the request
        index, providing reproducible LoRA assignment.

        Args:
            index (int): The request index used for round-robin selection.
            max_loras (Optional[int]): The maximum number of LoRAs available.
                If `None`, LoRA is not used.
            lora_path (Optional[str]): Path to the LoRA parameters on disk.
                If `None`, LoRA is not used.

    # ... omitted for brevity ...
        )
        return lora_request
```
**EN:** Method `BenchmarkDataset.get_round_robin_lora_request` provides a reusable helper around the module's main workflow. The docstring highlights: Optionally select a LoRA request using deterministic round-robin. Key calls such as `LoRARequest`, `str`, `lora_path_on_disk` show the concrete execution path.
**CN:** Method `BenchmarkDataset.get_round_robin_lora_request` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Optionally select a LoRA request using deterministic round-robin. 像 `LoRARequest`, `str`, `lora_path_on_disk` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BenchmarkDataset.get_lora_request` (lines 224-249)
```python
    def get_lora_request(
        self,
        index: int,
        max_loras: int | None = None,
        lora_path: str | None = None,
        lora_assignment: str = "random",
    ) -> LoRARequest | None:
        """
        Select a LoRA request using the specified assignment strategy.

        Args:
            index (int): The request index (used for round-robin).
            max_loras (Optional[int]): The maximum number of LoRAs available.
            lora_path (Optional[str]): Path to the LoRA parameters on disk.
            lora_assignment (str): Strategy for LoRA selection.
                'random' (default) or 'round-robin'.

        Returns:
            A new [`LoRARequest`][vllm.lora.request.LoRARequest]
    # ... omitted for brevity ...
            )
        return self.get_random_lora_request(max_loras=max_loras, lora_path=lora_path)
```
**EN:** Method `BenchmarkDataset.get_lora_request` provides a reusable helper around the module's main workflow. The docstring highlights: Select a LoRA request using the specified assignment strategy. Key calls such as `self.get_round_robin_lora_request`, `self.get_random_lora_request` show the concrete execution path.
**CN:** Method `BenchmarkDataset.get_lora_request` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Select a LoRA request using the specified assignment strategy. 像 `self.get_round_robin_lora_request`, `self.get_random_lora_request` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BenchmarkDataset.sample` (lines 252-276)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        """
        Abstract method to generate sample requests from the dataset.

        Subclasses must override this method to implement dataset-specific logic
        for generating a list of SampleRequest objects.

        Args:
            tokenizer (TokenizerLike): The tokenizer to be used
                for processing the dataset's text.
            num_requests (int): The number of sample requests to generate.
            request_id_prefix (str): The prefix of request_id.
    # ... omitted for brevity ...
        """
        raise NotImplementedError("sample must be implemented in subclasses.")
```
**EN:** Method `BenchmarkDataset.sample` samples, slices, or reshapes data for downstream use. The docstring highlights: Abstract method to generate sample requests from the dataset. Key calls such as `NotImplementedError` show the concrete execution path.
**CN:** Method `BenchmarkDataset.sample` 负责为下游流程采样、切片或重组数据。 文档字符串强调：Abstract method to generate sample requests from the dataset. 像 `NotImplementedError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BenchmarkDataset.maybe_oversample_requests` (lines 278-320)
```python
    def maybe_oversample_requests(
        self,
        requests: list[SampleRequest],
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
    ) -> None:
        """
        Oversamples the list of requests if its size is less than the desired
        number.

        Args:
            requests (List[SampleRequest]): The current list of sampled
                requests.
            num_requests (int): The target number of requests.
            request_id_prefix (str): The prefix applied to generated request
                identifiers.

        """
    # ... omitted for brevity ...
                "is unique."
            )
```
**EN:** Method `BenchmarkDataset.maybe_oversample_requests` samples, slices, or reshapes data for downstream use. The docstring highlights: Oversamples the list of requests if its size is less than the desired number. Key calls such as `logger.info`, `len`, `random.seed`, `range`, `replace` show the concrete execution path.
**CN:** Method `BenchmarkDataset.maybe_oversample_requests` 负责为下游流程采样、切片或重组数据。 文档字符串强调：Oversamples the list of requests if its size is less than the desired number. 像 `logger.info`, `len`, `random.seed`, `range`, `replace` 这样的关键调用展示了该代码块的具体执行路径。

### Function `process_image` (lines 364-409)
```python
def process_image(image: Any) -> Mapping[str, Any]:
    """
    Process a single image input and return a multimedia content dictionary.

    Supports the following input types:

    1. Dictionary with raw image bytes: - Expects a dict with a 'bytes' key
       containing raw image data.  - Loads the bytes as a PIL.Image.Image.

    2. PIL.Image.Image input: - Converts the image to RGB.  - Saves the image as
       a JPEG in memory.  - Encodes the JPEG data as a base64 string.  - Returns
       a dictionary with the image as a base64 data URL.

    3. String input: - Treats the string as a URL, local file path, or base64
       encoded data.  - If string starts with "data:image/", treats as base64.
       - If string starts with "http://", "https://", or "file://", treats as URL.
       - Otherwise treats as local file path and prepends "file://".
       - Returns a dictionary with the image URL or base64 data.

    Raises:
        ValueError: If the input is not a supported type.
    """
    if isinstance(image, dict) and "bytes" in image:
        image = Image.open(BytesIO(image["bytes"]))
    if isinstance(image, Image.Image):
    # ... omitted for brevity ...
        "str (URL, file path, or base64 data URL), or dictionary with raw image bytes."
    )
```
**EN:** Function `process_image` works with modality-specific preprocessing or transport logic. The docstring highlights: Process a single image input and return a multimedia content dictionary. Key calls such as `isinstance`, `Image.open`, `BytesIO`, `convert_image_mode`, `io.BytesIO` show the concrete execution path.
**CN:** Function `process_image` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Process a single image input and return a multimedia content dictionary. 像 `isinstance`, `Image.open`, `BytesIO`, `convert_image_mode`, `io.BytesIO` 这样的关键调用展示了该代码块的具体执行路径。

### Function `process_video` (lines 412-446)
```python
def process_video(video: Any) -> Mapping[str, Any]:
    """
    Process a single video input and return a multimedia content dictionary.

    Supports the following input types:

    1. Dictionary with raw video bytes: - Expects a dict with a 'bytes' key
       containing raw video data.

    2. String input: - Treats the string as a URL or local file path.  -
       Prepends "file://" if the string doesn't start with "http://" or
       "file://".  - Returns a dictionary with the image URL.

    Raises:
        ValueError: If the input is not a supported type.
    """
    if isinstance(video, dict) and "bytes" in video:
        video_bytes = video["bytes"]
        video_base64 = base64.b64encode(video_bytes).decode("utf-8")
        return {
            "type": "video_url",
            "video_url": {"url": f"data:video/mp4;base64,{video_base64}"},
        }

    if isinstance(video, str):
    # ... omitted for brevity ...
        f"Invalid video input {video}. Must be a string of local path/remote url, or a dictionary with raw video bytes in the form of `{{'bytes': raw_video_bytes}}`."  # noqa: E501
    )
```
**EN:** Function `process_video` works with modality-specific preprocessing or transport logic. The docstring highlights: Process a single video input and return a multimedia content dictionary. Key calls such as `isinstance`, `base64.b64encode(video_bytes).decode`, `base64.b64encode`, `video.startswith`, `ValueError` show the concrete execution path.
**CN:** Function `process_video` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Process a single video input and return a multimedia content dictionary. 像 `isinstance`, `base64.b64encode(video_bytes).decode`, `base64.b64encode`, `video.startswith`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `process_audio` (lines 449-467)
```python
def process_audio(audio: Any) -> tuple:
    """
    Process a single audio input and return a (array, sample_rate) tuple.

    Supports:
    1. String: treated as a file path, loaded with soundfile.
    2. Dict with 'array' and 'sampling_rate' keys: HuggingFace audio format.
    3. Tuple (array, sr): passed through directly.
    """
    if isinstance(audio, str):
        return sf.read(audio)
    if isinstance(audio, dict) and "array" in audio and "sampling_rate" in audio:
        return audio["array"], audio["sampling_rate"]
    if isinstance(audio, tuple) and len(audio) == 2:
        return audio
    raise ValueError(
        f"Invalid audio input {audio}. Must be a file path string, "
        "a dict with 'array' and 'sampling_rate', or a (array, sr) tuple."
    )
```
**EN:** Function `process_audio` works with modality-specific preprocessing or transport logic. The docstring highlights: Process a single audio input and return a (array, sample_rate) tuple. Key calls such as `isinstance`, `sf.read`, `len`, `ValueError` show the concrete execution path.
**CN:** Function `process_audio` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Process a single audio input and return a (array, sample_rate) tuple. 像 `isinstance`, `sf.read`, `len`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Class `RandomDataset` (lines 532-746)
```python
class RandomDataset(BenchmarkDataset):
    """
    Synthetic text-only dataset for serving/throughput benchmarks.

    Strategy:
    - Sample input/output token lengths per request from integer-uniform ranges
      around configured means (controlled by range_ratio).
    - Prepend a fixed random prefix of length prefix_len.
    - Generate the remaining tokens as a reproducible sequence:
      (offset + index + arange(input_len)) % vocab_size.
    - Decode then re-encode/truncate to ensure prompt token counts match.
    - Uses numpy.default_rng seeded with random_seed for reproducible sampling.
    """

    # Default values copied from benchmark_serving.py for the random dataset.
    DEFAULT_PREFIX_LEN = 0
    DEFAULT_RANGE_RATIO = 0.0
    DEFAULT_INPUT_LEN = 1024
    DEFAULT_OUTPUT_LEN = 128

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        # Use numpy's default_rng for deterministic sampling
    # ... omitted for brevity ...
        total_input_len = len(adjusted_token_sequence)
        return prompt, total_input_len, token_mismatch
```
**EN:** Class `RandomDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `sample`, `get_prefix`, `generate_token_sequence`, which define initialization, validation, transformation, or access patterns. The class docstring says: Synthetic text-only dataset for serving/throughput benchmarks.
**CN:** 类 `RandomDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `sample`, `get_prefix`, `generate_token_sequence`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Synthetic text-only dataset for serving/throughput benchmarks.

### Method `RandomDataset.__init__` (lines 552-557)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        # Use numpy's default_rng for deterministic sampling
        # Do not use random.seed() or np.random.seed() elsewhere in this class.
        # This ensures that the RNG is isolated from global RNG state.
        self._rng = np.random.default_rng(self.random_seed)
```
**EN:** Method `RandomDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `np.random.default_rng` show the concrete execution path.
**CN:** Method `RandomDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `np.random.default_rng` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomDataset.sample` (lines 559-668)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = DEFAULT_PREFIX_LEN,
        range_ratio: RangeRatio = DEFAULT_RANGE_RATIO,
        input_len: int = DEFAULT_INPUT_LEN,
        output_len: int = DEFAULT_OUTPUT_LEN,
        batchsize: int = 1,
        max_loras: int | None = None,
        lora_path: str | None = None,
        lora_assignment: str = "random",
        **kwargs,
    ) -> list[SampleRequest]:
        resolved_input_rr, _ = _resolve_range_ratios(range_ratio)

        num_special = int(tokenizer.num_special_tokens_to_add())
    # ... omitted for brevity ...

        return requests
```
**EN:** Method `RandomDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `_resolve_range_ratios`, `int`, `tokenizer.num_special_tokens_to_add`, `max`, `math.floor` show the concrete execution path.
**CN:** Method `RandomDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `_resolve_range_ratios`, `int`, `tokenizer.num_special_tokens_to_add`, `max`, `math.floor` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomDataset.get_prefix` (lines 670-701)
```python
    def get_prefix(
        self,
        tokenizer: TokenizerLike,
        allowed_tokens: np.ndarray,
        prefix_len: int,
    ) -> list[int]:
        """
        Get the prefix for the dataset.
        """
        if prefix_len <= 0:
            return []

        prefix_tokens = allowed_tokens[
            self._rng.integers(0, len(allowed_tokens), size=prefix_len)
        ].tolist()
        _, adjusted_tokens, token_mismatch = gen_prompt_decode_to_target_len(
            tokenizer=tokenizer,
            token_sequence=prefix_tokens,
            target_token_len=prefix_len,
    # ... omitted for brevity ...
            )
        return adjusted_tokens
```
**EN:** Method `RandomDataset.get_prefix` provides a reusable helper around the module's main workflow. The docstring highlights: Get the prefix for the dataset. Key calls such as `allowed_tokens[self._rng.integers(0, len(allowed_tokens), size=prefix_len)].tolist`, `self._rng.integers`, `len`, `gen_prompt_decode_to_target_len`, `logger.warning` show the concrete execution path.
**CN:** Method `RandomDataset.get_prefix` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get the prefix for the dataset. 像 `allowed_tokens[self._rng.integers(0, len(allowed_tokens), size=prefix_len)].tolist`, `self._rng.integers`, `len`, `gen_prompt_decode_to_target_len`, `logger.warning` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomDataset.generate_token_sequence` (lines 703-746)
```python
    def generate_token_sequence(
        self,
        *,
        tokenizer: TokenizerLike,
        prefix_token_ids: list[int],
        prefix_len: int,
        vocab_size: int,
        input_len: int,
        offset: int,
        index: int,
        allowed_tokens: np.ndarray,
    ) -> tuple[str, int, int]:
        """
        Returns (prompt, total_input_len).

        NOTE: After decoding the prompt we have to encode and decode it again.
        This is done because in some cases N consecutive tokens
        give a string tokenized into != N number of tokens.
        For example for GPT2Tokenizer:
    # ... omitted for brevity ...
        total_input_len = len(adjusted_token_sequence)
        return prompt, total_input_len, token_mismatch
```
**EN:** Method `RandomDataset.generate_token_sequence` implements an encoding/decoding or token-transformation step. The docstring highlights: Returns (prompt, total_input_len). Key calls such as `allowed_tokens[(offset + index + np.arange(input_len)) % len(allowed_tokens)].tolist`, `np.arange`, `len`, `int`, `gen_prompt_decode_to_target_len` show the concrete execution path.
**CN:** Method `RandomDataset.generate_token_sequence` 实现编码/解码或 Token 变换步骤。 文档字符串强调：Returns (prompt, total_input_len). 像 `allowed_tokens[(offset + index + np.arange(input_len)) % len(allowed_tokens)].tolist`, `np.arange`, `len`, `int`, `gen_prompt_decode_to_target_len` 这样的关键调用展示了该代码块的具体执行路径。

### Class `RandomDatasetForReranking` (lines 754-871)
```python
class RandomDatasetForReranking(RandomDataset):
    """
    Random dataset specialized for the needs of scoring:
    - Batches of inputs
    - Inputs composed of pairs
    """

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = RandomDataset.DEFAULT_PREFIX_LEN,
        range_ratio: RangeRatio = RandomDataset.DEFAULT_RANGE_RATIO,
        input_len: int = RandomDataset.DEFAULT_INPUT_LEN,
        output_len: int = RandomDataset.DEFAULT_OUTPUT_LEN,
        batchsize: int = 1,
        is_reranker: bool = True,
        **kwargs,
    # ... omitted for brevity ...

        return batch_requests
```
**EN:** Class `RandomDatasetForReranking` is a structured building block in this module. It inherits from `RandomDataset`. Key methods include `__init__`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Random dataset specialized for the needs of scoring: - Batches of inputs - Inputs composed of pairs
**CN:** 类 `RandomDatasetForReranking` 是该模块中的结构化构件，继承自 `RandomDataset`。 关键方法包括 `__init__`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Random dataset specialized for the needs of scoring: - Batches of inputs - Inputs composed of pairs

### Method `RandomDatasetForReranking.__init__` (lines 761-762)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
```
**EN:** Method `RandomDatasetForReranking.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super` show the concrete execution path.
**CN:** Method `RandomDatasetForReranking.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomDatasetForReranking.sample` (lines 764-871)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = RandomDataset.DEFAULT_PREFIX_LEN,
        range_ratio: RangeRatio = RandomDataset.DEFAULT_RANGE_RATIO,
        input_len: int = RandomDataset.DEFAULT_INPUT_LEN,
        output_len: int = RandomDataset.DEFAULT_OUTPUT_LEN,
        batchsize: int = 1,
        is_reranker: bool = True,
        **kwargs,
    ) -> list[SampleRequest]:
        n_sep_tokens = int(is_reranker)

        query_len_param = (input_len // 2) - n_sep_tokens if is_reranker else input_len

        query_lens, _, query_offsets = get_sampling_params(
    # ... omitted for brevity ...

        return batch_requests
```
**EN:** Method `RandomDatasetForReranking.sample` samples, slices, or reshapes data for downstream use. Key calls such as `int`, `get_sampling_params`, `np.arange`, `np.array`, `list` show the concrete execution path.
**CN:** Method `RandomDatasetForReranking.sample` 负责为下游流程采样、切片或重组数据。 像 `int`, `get_sampling_params`, `np.arange`, `np.array`, `list` 这样的关键调用展示了该代码块的具体执行路径。

### Class `RandomMultiModalDataset` (lines 879-1304)
```python
class RandomMultiModalDataset(RandomDataset):
    """
    Synthetic multimodal dataset (text + images) that extends RandomDataset.

    Status:
    - Images: supported via synthetic RGB data.
    - Video: supported via synthetic RGB data.
    - Audio: not yet supported.

    Sampling overview:
    1) Number of items per request is sampled uniformly from the integer range
       [floor(n·(1−r)), ceil(n·(1+r))], where n is the base count and r is
       `num_mm_items_range_ratio` in [0, 1]. r=0 keeps it fixed; r=1 allows 0.
       The maximum is further clamped to the sum of per-modality limits.
    2) Each item’s modality and shape is sampled from `bucket_config`, a dict
       mapping (height, width, num_frames) → probability. We treat
       `num_frames`=1 as image and `num_frames` > 1 as video.
       Entries with zero probability are removed and the rest are renormalized
       to sum to 1.
    3) Per-modality hard caps are enforced via `limit_mm_per_prompt`.
       When a modality reaches its cap, all of its buckets are excluded and the
       remaining probabilities are renormalized.

    # ... omitted for brevity ...

        return mm_requests
```
**EN:** Class `RandomMultiModalDataset` is a structured building block in this module. It inherits from `RandomDataset`. Key methods include `__init__`, `generate_synthetic_image`, `generate_synthetic_video`, `map_config_to_modality`, `normalize_bucket_config`, `generate_mm_item`, which define initialization, validation, transformation, or access patterns. The class docstring says: Synthetic multimodal dataset (text + images) that extends RandomDataset.
**CN:** 类 `RandomMultiModalDataset` 是该模块中的结构化构件，继承自 `RandomDataset`。 关键方法包括 `__init__`, `generate_synthetic_image`, `generate_synthetic_video`, `map_config_to_modality`, `normalize_bucket_config`, `generate_mm_item`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Synthetic multimodal dataset (text + images) that extends RandomDataset.

### Method `RandomMultiModalDataset.__init__` (lines 921-922)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
```
**EN:** Method `RandomMultiModalDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super` show the concrete execution path.
**CN:** Method `RandomMultiModalDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomMultiModalDataset.generate_synthetic_video` (lines 940-982)
```python
    def generate_synthetic_video(
        self, width: int, height: int, num_frames: int
    ) -> dict:
        """Generate synthetic video with random values.

        Creates a video with random pixel values, encodes it to MP4 format,
        and returns the content as bytes.
        """
        import cv2

        random_pixels = self._rng.integers(
            0,
            256,
            (num_frames, height, width, 3),
            dtype=np.uint8,
        )

        # Create a temporary video file in memory
        fourcc = cv2.VideoWriter_fourcc(*"mp4v")
    # ... omitted for brevity ...

            return {"bytes": video_content}
```
**EN:** Method `RandomMultiModalDataset.generate_synthetic_video` works with modality-specific preprocessing or transport logic. The docstring highlights: Generate synthetic video with random values. Key calls such as `self._rng.integers`, `cv2.VideoWriter_fourcc`, `NamedTemporaryFile`, `cv2.VideoWriter`, `video_writer.isOpened` show the concrete execution path.
**CN:** Method `RandomMultiModalDataset.generate_synthetic_video` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Generate synthetic video with random values. 像 `self._rng.integers`, `cv2.VideoWriter_fourcc`, `NamedTemporaryFile`, `cv2.VideoWriter`, `video_writer.isOpened` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomMultiModalDataset.generate_mm_item` (lines 1014-1036)
```python
    def generate_mm_item(
        self,
        mm_item_config: tuple[int, int, int],
    ) -> Mapping[str, Any]:
        """
        Create synthetic images and videos and
        apply process_image/process_video respectively.
        This follows the OpenAI API chat completions
        https://github.com/openai/openai-python
        """

        if self.map_config_to_modality(mm_item_config) == "image":
            return process_image(
                self.generate_synthetic_image(mm_item_config[1], mm_item_config[0])
            )
        elif self.map_config_to_modality(mm_item_config) == "video":
            return process_video(
                self.generate_synthetic_video(
                    mm_item_config[1], mm_item_config[0], mm_item_config[2]
    # ... omitted for brevity ...
        else:
            raise ValueError(f"Invalid multimodal item configuration: {mm_item_config}")
```
**EN:** Method `RandomMultiModalDataset.generate_mm_item` provides a reusable helper around the module's main workflow. The docstring highlights: Create synthetic images and videos and apply process_image/process_video respectively. Key calls such as `self.map_config_to_modality`, `process_image`, `self.generate_synthetic_image`, `process_video`, `self.generate_synthetic_video` show the concrete execution path.
**CN:** Method `RandomMultiModalDataset.generate_mm_item` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Create synthetic images and videos and apply process_image/process_video respectively. 像 `self.map_config_to_modality`, `process_image`, `self.generate_synthetic_image`, `process_video`, `self.generate_synthetic_video` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomMultiModalDataset.get_mm_item_sampling_params` (lines 1038-1111)
```python
    def get_mm_item_sampling_params(
        self,
        base_items_per_request: int,
        num_mm_items_range_ratio: float,
        limit_mm_per_prompt: dict[str, int],
        bucket_config: dict[tuple[int, int, int], float],
    ) -> tuple[int, int, dict[str, int], dict[tuple[int, int, int], float]]:
        """
        Get the sampling parameters for the multimodal items.
        """
        # Enforce num_mm_items_range_ratio <= 1
        if not (0.0 <= num_mm_items_range_ratio <= 1.0):
            raise ValueError("num_mm_items_range_ratio must be in [0, 1].")

        # Ensure modalities to sample are in limit_mm_per_prompt
        for k, v in bucket_config.items():
            # get modality from bucket config
            modality = self.map_config_to_modality(k)
            if modality not in limit_mm_per_prompt:
    # ... omitted for brevity ...
            bucket_config,
        )
```
**EN:** Method `RandomMultiModalDataset.get_mm_item_sampling_params` provides a reusable helper around the module's main workflow. The docstring highlights: Get the sampling parameters for the multimodal items. Key calls such as `ValueError`, `bucket_config.items`, `self.map_config_to_modality`, `limit_mm_per_prompt.keys`, `self.normalize_bucket_config` show the concrete execution path.
**CN:** Method `RandomMultiModalDataset.get_mm_item_sampling_params` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get the sampling parameters for the multimodal items. 像 `ValueError`, `bucket_config.items`, `self.map_config_to_modality`, `limit_mm_per_prompt.keys`, `self.normalize_bucket_config` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomMultiModalDataset.get_mm_item_iterator` (lines 1113-1172)
```python
    def get_mm_item_iterator(
        self,
        min_num_mm_items: int,
        max_num_mm_items: int,
        bucket_config: dict[tuple[int, int, int], float],
        limit_mm_per_prompt: dict[str, int],
    ) -> Iterator[tuple[int, int, int]]:
        """
        Iterator over the multimodal items for each request
        whose size is between min_num_mm_items and max_num_mm_items.

        Loop over the bucket config and sample a multimodal item.
        Loop until the number of multimodal items sampled is equal to
        request_num_mm_items or limit of multimodal items per prompt
        for all modalities is reached.

        Note:
        - This function operates on a per-request shallow copy of
          `bucket_config` (tuple->float). The original dict passed to
    # ... omitted for brevity ...
                # Renormalize the bucket config
                bucket_config_copy = self.normalize_bucket_config(bucket_config_copy)
```
**EN:** Method `RandomMultiModalDataset.get_mm_item_iterator` provides a reusable helper around the module's main workflow. The docstring highlights: Iterator over the multimodal items for each request whose size is between min_num_mm_items and max_num_mm_items. Key calls such as `int`, `self._rng.integers`, `self.map_config_to_modality`, `bucket_config.copy`, `sum` show the concrete execution path.
**CN:** Method `RandomMultiModalDataset.get_mm_item_iterator` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Iterator over the multimodal items for each request whose size is between min_num_mm_items and max_num_mm_items. 像 `int`, `self._rng.integers`, `self.map_config_to_modality`, `bucket_config.copy`, `sum` 这样的关键调用展示了该代码块的具体执行路径。

### Method `RandomMultiModalDataset.sample` (lines 1174-1304)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = RandomDataset.DEFAULT_PREFIX_LEN,
        range_ratio: RangeRatio = RandomDataset.DEFAULT_RANGE_RATIO,
        input_len: int = RandomDataset.DEFAULT_INPUT_LEN,
        output_len: int = RandomDataset.DEFAULT_OUTPUT_LEN,
        batchsize: int = 1,
        limit_mm_per_prompt: dict[str, int] = DEFAULT_LIMIT_MM_PER_PROMPT,
        base_items_per_request: int = DEFAULT_BASE_ITEMS_PER_REQUEST,
        num_mm_items_range_ratio: float = DEFAULT_NUM_MM_ITEMS_RANGE_RATIO,
        bucket_config: dict[
            tuple[int, int, int], float
        ] = DEFAULT_MM_ITEM_BUCKET_CONFIG,
        enable_multimodal_chat: bool = DEFAULT_ENABLE_MULTIMODAL_CHAT,
        **kwargs,
    # ... omitted for brevity ...

        return mm_requests
```
**EN:** Method `RandomMultiModalDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `NotImplementedError`, `get_sampling_params`, `self.get_mm_item_sampling_params`, `list`, `tokenizer.added_tokens_decoder.items` show the concrete execution path.
**CN:** Method `RandomMultiModalDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `NotImplementedError`, `get_sampling_params`, `self.get_mm_item_sampling_params`, `list`, `tokenizer.added_tokens_decoder.items` 这样的关键调用展示了该代码块的具体执行路径。

### Class `ShareGPTDataset` (lines 1312-1399)
```python
class ShareGPTDataset(BenchmarkDataset):
    """
    Implements the ShareGPT dataset.  Loads data from a JSON file and generates
    sample requests based on conversation turns.
    """

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.load_data()

    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        with open(self.dataset_path, encoding="utf-8") as f:
            self.data = json.load(f)
        # Filter entries with at least two conversation turns.
        self.data = [
            entry
            for entry in self.data
            if "conversations" in entry and len(entry["conversations"]) >= 2
        ]
        random.seed(self.random_seed)
    # ... omitted for brevity ...
        )
        return samples
```
**EN:** Class `ShareGPTDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `load_data`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Implements the ShareGPT dataset.
**CN:** 类 `ShareGPTDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `load_data`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Implements the ShareGPT dataset.

### Method `ShareGPTDataset.__init__` (lines 1318-1320)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.load_data()
```
**EN:** Method `ShareGPTDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `ShareGPTDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShareGPTDataset.load_data` (lines 1322-1336)
```python
    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        with open(self.dataset_path, encoding="utf-8") as f:
            self.data = json.load(f)
        # Filter entries with at least two conversation turns.
        self.data = [
            entry
            for entry in self.data
            if "conversations" in entry and len(entry["conversations"]) >= 2
        ]
        random.seed(self.random_seed)
        if not getattr(self, "disable_shuffle", False):
            random.shuffle(self.data)
```
**EN:** Method `ShareGPTDataset.load_data` handles loading or retrieval of external/internal data. Key calls such as `ValueError`, `open`, `json.load`, `len`, `random.seed` show the concrete execution path.
**CN:** Method `ShareGPTDataset.load_data` 负责加载或获取外部/内部数据。 像 `ValueError`, `open`, `json.load`, `len`, `random.seed` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShareGPTDataset.sample` (lines 1338-1399)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        lora_path: str | None = None,
        max_loras: int | None = None,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        lora_assignment: str = "random",
        **kwargs,
    ) -> list[SampleRequest]:
        samples: list[SampleRequest] = []
        ind = 0
        for entry in self.data:
            if len(samples) >= num_requests:
                break
            prompt, completion = (
    # ... omitted for brevity ...
        )
        return samples
```
**EN:** Method `ShareGPTDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `len`, `self.get_lora_request`, `tokenizer`, `is_valid_sequence`, `entry.get` show the concrete execution path.
**CN:** Method `ShareGPTDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `len`, `self.get_lora_request`, `tokenizer`, `is_valid_sequence`, `entry.get` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_samples` (lines 1825-2178)
```python
def get_samples(args, tokenizer: TokenizerLike) -> list[SampleRequest]:
    if not hasattr(args, "request_id_prefix"):
        args.request_id_prefix = ""

    if hasattr(args, "random_range_ratio") and isinstance(args.random_range_ratio, str):
        args.random_range_ratio = _parse_range_ratio(args.random_range_ratio)

    if args.dataset_name == "custom":
        dataset = CustomDataset(
            dataset_path=args.dataset_path,
            disable_shuffle=args.disable_shuffle,
            random_seed=args.seed,
        )
        input_requests = dataset.sample(
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            output_len=args.custom_output_len,
            skip_chat_template=args.skip_chat_template,
            request_id_prefix=args.request_id_prefix,
            no_oversample=args.no_oversample,
        )

    elif args.dataset_name in ("custom_image", "custom_mm"):
        if args.dataset_name == "custom_mm":
            logger.warning(
    # ... omitted for brevity ...

    return input_requests
```
**EN:** Function `get_samples` samples, slices, or reshapes data for downstream use. Key calls such as `hasattr`, `isinstance`, `_parse_range_ratio`, `CustomDataset`, `dataset.sample` show the concrete execution path.
**CN:** Function `get_samples` 负责为下游流程采样、切片或重组数据。 像 `hasattr`, `isinstance`, `_parse_range_ratio`, `CustomDataset`, `dataset.sample` 这样的关键调用展示了该代码块的具体执行路径。

### Class `CustomDataset` (lines 2186-2310)
```python
class CustomDataset(BenchmarkDataset):
    """
    Implements the Custom dataset.  Loads data from a JSONL file and generates
    sample requests based on conversation turns. E.g.,
    ` ` `
    {"prompt": "What is the capital of India?", "output_tokens": 10}
    {"prompt": "What is the capital of Iran?", "output_tokens": 1520}
    {"prompt": "What is the capital of China?", "output_tokens": 819}
    ` ` `
    Note that 'output_tokens' column is optional and has to be provided only if
    'custom-output-len' argument is None or -1.
    """

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.load_data()

    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        # self.data will be a list of dictionaries
        # e.g., [{"prompt": "What is the capital of India?"}, ...]
    # ... omitted for brevity ...

        return sampled_requests
```
**EN:** Class `CustomDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `load_data`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Implements the Custom dataset.
**CN:** 类 `CustomDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `load_data`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Implements the Custom dataset.

### Method `CustomDataset.__init__` (lines 2199-2201)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.load_data()
```
**EN:** Method `CustomDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `CustomDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CustomDataset.load_data` (lines 2203-2235)
```python
    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        # self.data will be a list of dictionaries
        # e.g., [{"prompt": "What is the capital of India?"}, ...]
        # This will be the standardized format which load_data()
        # has to convert into depending on the filetype of dataset_path.
        # sample() will assume this standardized format of self.data
        self.data: list[dict] = []

        # Load the JSONL file
        if self.dataset_path.endswith(".jsonl"):
            jsonl_data = pd.read_json(path_or_buf=self.dataset_path, lines=True)

            # check if the JSONL file has a 'prompt' column
            if "prompt" not in jsonl_data.columns:
                raise ValueError("JSONL file must contain a 'prompt' column.")

    # ... omitted for brevity ...
        if not getattr(self, "disable_shuffle", False):
            random.shuffle(self.data)
```
**EN:** Method `CustomDataset.load_data` handles loading or retrieval of external/internal data. Key calls such as `ValueError`, `self.dataset_path.endswith`, `pd.read_json`, `jsonl_data.iterrows`, `self.data.append` show the concrete execution path.
**CN:** Method `CustomDataset.load_data` 负责加载或获取外部/内部数据。 像 `ValueError`, `self.dataset_path.endswith`, `pd.read_json`, `jsonl_data.iterrows`, `self.data.append` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CustomDataset.sample` (lines 2237-2310)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        lora_path: str | None = None,
        max_loras: int | None = None,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # load all data if needed
        self.num_available_samples = len(self.data)
        if num_requests <= 0:
            num_requests = self.num_available_samples
            logger.info(
                "num_requests is set to 0 or negative, "
    # ... omitted for brevity ...

        return sampled_requests
```
**EN:** Method `CustomDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `len`, `logger.info`, `enumerate`, `ValueError`, `int` show the concrete execution path.
**CN:** Method `CustomDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `len`, `logger.info`, `enumerate`, `ValueError`, `int` 这样的关键调用展示了该代码块的具体执行路径。

### Class `CustomImageDataset` (lines 2313-2389)
```python
class CustomImageDataset(CustomDataset):
    """
    Implements the Custom image dataset. Loads data from a JSONL file and generates
    sample requests based on conversation turns. E.g.,
    ` ` `
    {
        "prompt": "How many red blocks in the given images?",
        "image_files": ["path/to/image1.png", "path/to/image2.png"],
    }
    {
        "prompt": "Which country has the most pokemons based on the given graphs?",
        "image_files": ["path/to/image.png"],
    }
    ` ` `

    NOTE: Only the first image file in "image_files" is used for each sample request.

    This is used to benchmark multimodal LLMs on arbitrary datasets.
    """

    IS_MULTIMODAL = True

    def sample(
    # ... omitted for brevity ...

        return sampled_requests
```
**EN:** Class `CustomImageDataset` is a structured building block in this module. It inherits from `CustomDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Implements the Custom image dataset.
**CN:** 类 `CustomImageDataset` 是该模块中的结构化构件，继承自 `CustomDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Implements the Custom image dataset.

### Method `CustomImageDataset.sample` (lines 2335-2389)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # load all data if needed
        self.num_available_samples = len(self.data)
        if num_requests <= 0:
            num_requests = self.num_available_samples
            logger.info(
                "num_requests is set to 0 or negative, "
                "so using all available samples: %d",
                num_requests,
            )
    # ... omitted for brevity ...

        return sampled_requests
```
**EN:** Method `CustomImageDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `len`, `logger.info`, `enumerate`, `tokenizer`, `logger.warning` show the concrete execution path.
**CN:** Method `CustomImageDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `len`, `logger.info`, `enumerate`, `tokenizer`, `logger.warning` 这样的关键调用展示了该代码块的具体执行路径。

### Class `CustomAudioDataset` (lines 2392-2487)
```python
class CustomAudioDataset(CustomDataset):
    """
    Custom dataset for audio benchmarking. Loads data from a JSONL file. E.g.,
    {"prompt": "Transcribe the audio.", "audio": "/path/to/audio.wav"}

    Supports both:
    - Dedicated ASR models (e.g. Whisper) via openai-audio & /v1/audio/transcriptions
    - Chat-based audio models (e.g. Qwen2-Audio) via openai-chat & /v1/chat/completions
    """

    IS_MULTIMODAL = True

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        output_len: int | None = None,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        skip_chat_template: bool = False,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `CustomAudioDataset` is a structured building block in this module. It inherits from `CustomDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Custom dataset for audio benchmarking.
**CN:** 类 `CustomAudioDataset` 是该模块中的结构化构件，继承自 `CustomDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Custom dataset for audio benchmarking.

### Method `CustomAudioDataset.sample` (lines 2404-2487)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        output_len: int | None = None,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        skip_chat_template: bool = False,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        self.num_available_samples = len(self.data)
        if num_requests <= 0:
            num_requests = self.num_available_samples
        sampled_requests = []
        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
                break
            prompt = item.get("prompt", "")
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `CustomAudioDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `len`, `enumerate`, `item.get`, `hasattr`, `process_audio` show the concrete execution path.
**CN:** Method `CustomAudioDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `len`, `enumerate`, `item.get`, `hasattr`, `process_audio` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SpecBench` (lines 2495-2537)
```python
class SpecBench(CustomDataset):
    """
    Implements the SpecBench dataset: https://github.com/hemingkx/Spec-Bench
    Download the dataset using:
    wget https://raw.githubusercontent.com/hemingkx/Spec-Bench/refs/heads/main/data/spec_bench/question.jsonl
    """  # noqa: E501

    def __init__(self, **kwargs) -> None:
        self.category = kwargs.pop("category", None)
        super().__init__(**kwargs)
        self.load_data()

    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        self.data = []

        # Load the JSONL file
        jsonl_data = pd.read_json(path_or_buf=self.dataset_path, lines=True)

        # check if the JSONL file has a 'turns' column
        if "turns" not in jsonl_data.columns:
    # ... omitted for brevity ...
            **kwargs,
        )
```
**EN:** Class `SpecBench` is a structured building block in this module. It inherits from `CustomDataset`. Key methods include `__init__`, `load_data`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Implements the SpecBench dataset: https://github.com/hemingkx/Spec-Bench Download the dataset using: wget https://raw.githubusercontent.com/hemingkx/Spec-Bench/refs/heads/main/d...
**CN:** 类 `SpecBench` 是该模块中的结构化构件，继承自 `CustomDataset`。 关键方法包括 `__init__`, `load_data`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Implements the SpecBench dataset: https://github.com/hemingkx/Spec-Bench Download the dataset using: wget https://raw.githubusercontent.com/hemingkx/Spec-Bench/refs/heads/main/d...

### Method `SpecBench.__init__` (lines 2502-2505)
```python
    def __init__(self, **kwargs) -> None:
        self.category = kwargs.pop("category", None)
        super().__init__(**kwargs)
        self.load_data()
```
**EN:** Method `SpecBench.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `kwargs.pop`, `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `SpecBench.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `kwargs.pop`, `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpecBench.load_data` (lines 2507-2528)
```python
    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        self.data = []

        # Load the JSONL file
        jsonl_data = pd.read_json(path_or_buf=self.dataset_path, lines=True)

        # check if the JSONL file has a 'turns' column
        if "turns" not in jsonl_data.columns:
            raise ValueError("JSONL file must contain a 'turns' column.")

        for _, row in jsonl_data.iterrows():
            # sample only from a specific category if specified
            if (not self.category) or (self.category == row["category"]):
                prompt = row["turns"][0]
                self.data.append({"prompt": prompt})

        random.seed(self.random_seed)
        if not getattr(self, "disable_shuffle", False):
            random.shuffle(self.data)
```
**EN:** Method `SpecBench.load_data` handles loading or retrieval of external/internal data. Key calls such as `ValueError`, `pd.read_json`, `jsonl_data.iterrows`, `self.data.append`, `random.seed` show the concrete execution path.
**CN:** Method `SpecBench.load_data` 负责加载或获取外部/内部数据。 像 `ValueError`, `pd.read_json`, `jsonl_data.iterrows`, `self.data.append`, `random.seed` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpecBench.sample` (lines 2530-2537)
```python
    def sample(
        self,
        **kwargs,
    ) -> list[SampleRequest]:
        # leverage CustomDataset sample
        return super().sample(
            **kwargs,
        )
```
**EN:** Method `SpecBench.sample` samples, slices, or reshapes data for downstream use. Key calls such as `super().sample`, `super` show the concrete execution path.
**CN:** Method `SpecBench.sample` 负责为下游流程采样、切片或重组数据。 像 `super().sample`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SonnetDataset` (lines 2548-2628)
```python
class SonnetDataset(BenchmarkDataset):
    """
    Simplified implementation of the Sonnet dataset.  Loads poem lines from a
    text file and generates sample requests.  Default values here copied from
    `benchmark_serving.py` for the sonnet dataset.
    """

    DEFAULT_PREFIX_LEN = 200
    DEFAULT_INPUT_LEN = 550
    DEFAULT_OUTPUT_LEN = 150

    def __init__(
        self,
        **kwargs,
    ) -> None:
        super().__init__(**kwargs)
        self.load_data()

    def load_data(self) -> None:
        if not self.dataset_path:
            raise ValueError("dataset_path must be provided.")
        with open(self.dataset_path, encoding="utf-8") as f:
            self.data = f.readlines()
    # ... omitted for brevity ...
                ind += 1
        return samples
```
**EN:** Class `SonnetDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `load_data`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Simplified implementation of the Sonnet dataset.
**CN:** 类 `SonnetDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `load_data`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Simplified implementation of the Sonnet dataset.

### Method `SonnetDataset.__init__` (lines 2559-2564)
```python
    def __init__(
        self,
        **kwargs,
    ) -> None:
        super().__init__(**kwargs)
        self.load_data()
```
**EN:** Method `SonnetDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `SonnetDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SonnetDataset.load_data` (lines 2566-2570)
```python
    def load_data(self) -> None:
        if not self.dataset_path:
            raise ValueError("dataset_path must be provided.")
        with open(self.dataset_path, encoding="utf-8") as f:
            self.data = f.readlines()
```
**EN:** Method `SonnetDataset.load_data` handles loading or retrieval of external/internal data. Key calls such as `ValueError`, `open`, `f.readlines` show the concrete execution path.
**CN:** Method `SonnetDataset.load_data` 负责加载或获取外部/内部数据。 像 `ValueError`, `open`, `f.readlines` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SonnetDataset.sample` (lines 2572-2628)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = DEFAULT_PREFIX_LEN,
        input_len: int = DEFAULT_INPUT_LEN,
        output_len: int = DEFAULT_OUTPUT_LEN,
        return_prompt_formatted: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # Calculate average token length for a poem line.
        tokenized_lines = [tokenizer(line).input_ids for line in self.data]
        avg_len = sum(len(tokens) for tokens in tokenized_lines) / len(tokenized_lines)

        # Build the base prompt.
        base_prompt = "Pick as many lines as you can from these poem lines:\n"
        base_msg = [{"role": "user", "content": base_prompt}]
    # ... omitted for brevity ...
                ind += 1
        return samples
```
**EN:** Method `SonnetDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `tokenizer`, `sum`, `len`, `tokenizer.apply_chat_template`, `ValueError` show the concrete execution path.
**CN:** Method `SonnetDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `tokenizer`, `sum`, `len`, `tokenizer.apply_chat_template`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Class `BurstGPTDataset` (lines 2636-2709)
```python
class BurstGPTDataset(BenchmarkDataset):
    """
    Implements the BurstGPT dataset.  Loads data from a CSV file and generates
    sample requests based on synthetic prompt generation. Only rows with Model
    "GPT-4" and positive response tokens are used.
    """

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.load_data()

    def load_data(
        self,
    ):
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        df = pd.read_csv(self.dataset_path)
        # Filter to keep only GPT-4 rows.
        gpt4_df = df[df["Model"] == "GPT-4"]
        # Remove failed requests (where Response tokens is 0 or less).
        gpt4_df = gpt4_df[gpt4_df["Response tokens"] > 0]
        # Sample the desired number of rows.
    # ... omitted for brevity ...
            )
        return samples
```
**EN:** Class `BurstGPTDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `load_data`, `_sample_loaded_data`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Implements the BurstGPT dataset.
**CN:** 类 `BurstGPTDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `load_data`, `_sample_loaded_data`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Implements the BurstGPT dataset.

### Method `BurstGPTDataset.__init__` (lines 2643-2645)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.load_data()
```
**EN:** Method `BurstGPTDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `BurstGPTDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BurstGPTDataset.load_data` (lines 2647-2659)
```python
    def load_data(
        self,
    ):
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        df = pd.read_csv(self.dataset_path)
        # Filter to keep only GPT-4 rows.
        gpt4_df = df[df["Model"] == "GPT-4"]
        # Remove failed requests (where Response tokens is 0 or less).
        gpt4_df = gpt4_df[gpt4_df["Response tokens"] > 0]
        # Sample the desired number of rows.
        self.data = gpt4_df
```
**EN:** Method `BurstGPTDataset.load_data` handles loading or retrieval of external/internal data. Key calls such as `ValueError`, `pd.read_csv` show the concrete execution path.
**CN:** Method `BurstGPTDataset.load_data` 负责加载或获取外部/内部数据。 像 `ValueError`, `pd.read_csv` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BurstGPTDataset._sample_loaded_data` (lines 2661-2671)
```python
    def _sample_loaded_data(self, num_requests: int) -> list:
        if num_requests <= len(self.data):
            data = self.data.sample(n=num_requests, random_state=self.random_seed)
        else:
            data = self.data.sample(
                n=num_requests,
                random_state=self.random_seed,
                replace=True,
            )
        # Convert the dataframe to a list of lists.
        return data.values.tolist()
```
**EN:** Method `BurstGPTDataset._sample_loaded_data` handles loading or retrieval of external/internal data. Key calls such as `len`, `self.data.sample`, `data.values.tolist` show the concrete execution path.
**CN:** Method `BurstGPTDataset._sample_loaded_data` 负责加载或获取外部/内部数据。 像 `len`, `self.data.sample`, `data.values.tolist` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BurstGPTDataset.sample` (lines 2673-2709)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        lora_assignment: str = "random",
        max_loras: int | None = None,
        lora_path: str | None = None,
        **kwargs,
    ) -> list[SampleRequest]:
        samples = []
        data = self._sample_loaded_data(num_requests=num_requests)
        for i in range(num_requests):
            input_len = int(data[i][2])
            output_len = int(data[i][3])
            lora_req = self.get_lora_request(
                index=i,
                max_loras=max_loras,
    # ... omitted for brevity ...
            )
        return samples
```
**EN:** Method `BurstGPTDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `self._sample_loaded_data`, `range`, `int`, `self.get_lora_request`, `tokenizer.decode` show the concrete execution path.
**CN:** Method `BurstGPTDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `self._sample_loaded_data`, `range`, `int`, `self.get_lora_request`, `tokenizer.decode` 这样的关键调用展示了该代码块的具体执行路径。

### Class `HuggingFaceDataset` (lines 2715-2749)
```python
class HuggingFaceDataset(BenchmarkDataset):
    """Base class for datasets hosted on HuggingFace."""

    SUPPORTED_DATASET_PATHS: set[str] | dict[str, Callable] = set()

    def __init__(
        self,
        dataset_path: str,
        dataset_split: str,
        no_stream: bool = False,
        dataset_subset: str | None = None,
        hf_name: str | None = None,
        trust_remote_code: bool = False,
        **kwargs,
    ) -> None:
        super().__init__(dataset_path=dataset_path, **kwargs)

        self.dataset_split = dataset_split
        self.dataset_subset = dataset_subset
        self.load_stream = not no_stream
        self.hf_name = hf_name or dataset_path
        self.trust_remote_code = trust_remote_code
        self.load_data()
    # ... omitted for brevity ...
        if not getattr(self, "disable_shuffle", False):
            self.data = self.data.shuffle(seed=self.random_seed)
```
**EN:** Class `HuggingFaceDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `load_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: Base class for datasets hosted on HuggingFace.
**CN:** 类 `HuggingFaceDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `load_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Base class for datasets hosted on HuggingFace.

### Method `HuggingFaceDataset.__init__` (lines 2720-2737)
```python
    def __init__(
        self,
        dataset_path: str,
        dataset_split: str,
        no_stream: bool = False,
        dataset_subset: str | None = None,
        hf_name: str | None = None,
        trust_remote_code: bool = False,
        **kwargs,
    ) -> None:
        super().__init__(dataset_path=dataset_path, **kwargs)

        self.dataset_split = dataset_split
        self.dataset_subset = dataset_subset
        self.load_stream = not no_stream
        self.hf_name = hf_name or dataset_path
        self.trust_remote_code = trust_remote_code
        self.load_data()
```
**EN:** Method `HuggingFaceDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `HuggingFaceDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `HuggingFaceDataset.load_data` (lines 2739-2749)
```python
    def load_data(self) -> None:
        """Load data from HuggingFace datasets."""
        self.data = load_dataset(
            self.dataset_path,
            name=self.dataset_subset,
            split=self.dataset_split,
            streaming=self.load_stream,
            trust_remote_code=self.trust_remote_code,
        )
        if not getattr(self, "disable_shuffle", False):
            self.data = self.data.shuffle(seed=self.random_seed)
```
**EN:** Method `HuggingFaceDataset.load_data` handles loading or retrieval of external/internal data. The docstring highlights: Load data from HuggingFace datasets. Key calls such as `load_dataset`, `getattr`, `self.data.shuffle` show the concrete execution path.
**CN:** Method `HuggingFaceDataset.load_data` 负责加载或获取外部/内部数据。 文档字符串强调：Load data from HuggingFace datasets. 像 `load_dataset`, `getattr`, `self.data.shuffle` 这样的关键调用展示了该代码块的具体执行路径。

### Class `ConversationDataset` (lines 2757-2814)
```python
class ConversationDataset(HuggingFaceDataset):
    """Dataset for text-only conversation data."""

    SUPPORTED_DATASET_PATHS = {
        "Aeala/ShareGPT_Vicuna_unfiltered",
    }
    IS_MULTIMODAL = False

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # Filter examples with at least 2 conversations
        filtered_data = self.data.filter(lambda x: len(x["conversations"]) >= 2)
        sampled_requests: list[SampleRequest] = []
        ind = 0
        dynamic_output = output_len is None
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `ConversationDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Dataset for text-only conversation data.
**CN:** 类 `ConversationDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Dataset for text-only conversation data.

### Method `ConversationDataset.sample` (lines 2765-2814)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # Filter examples with at least 2 conversations
        filtered_data = self.data.filter(lambda x: len(x["conversations"]) >= 2)
        sampled_requests: list[SampleRequest] = []
        ind = 0
        dynamic_output = output_len is None

        for item in filtered_data:
            if len(sampled_requests) >= num_requests:
                break
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `ConversationDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `self.data.filter`, `len`, `tokenizer`, `isinstance`, `is_valid_sequence` show the concrete execution path.
**CN:** Method `ConversationDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `self.data.filter`, `len`, `tokenizer`, `isinstance`, `is_valid_sequence` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalConversationDataset` (lines 2817-2874)
```python
class MultiModalConversationDataset(HuggingFaceDataset):
    """Dataset for multimodal conversation data."""

    SUPPORTED_DATASET_PATHS = {
        "lmms-lab/LLaVA-OneVision-Data",
    }
    IS_MULTIMODAL = True

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # Filter examples with at least 2 conversations
        filtered_data = self.data.filter(lambda x: len(x["conversations"]) >= 2)
        sampled_requests: list[SampleRequest] = []
        ind = 0
        dynamic_output = output_len is None
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `MultiModalConversationDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Dataset for multimodal conversation data.
**CN:** 类 `MultiModalConversationDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Dataset for multimodal conversation data.

### Method `MultiModalConversationDataset.sample` (lines 2825-2874)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # Filter examples with at least 2 conversations
        filtered_data = self.data.filter(lambda x: len(x["conversations"]) >= 2)
        sampled_requests: list[SampleRequest] = []
        ind = 0
        dynamic_output = output_len is None

        for item in filtered_data:
            if len(sampled_requests) >= num_requests:
                break
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `MultiModalConversationDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `self.data.filter`, `len`, `tokenizer`, `isinstance`, `is_valid_sequence` show the concrete execution path.
**CN:** Method `MultiModalConversationDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `self.data.filter`, `len`, `tokenizer`, `isinstance`, `is_valid_sequence` 这样的关键调用展示了该代码块的具体执行路径。

### Class `VisionArenaDataset` (lines 2882-2937)
```python
class VisionArenaDataset(HuggingFaceDataset):
    """
    Vision Arena Dataset.
    """

    DEFAULT_OUTPUT_LEN = 128
    SUPPORTED_DATASET_PATHS = {
        "lmarena-ai/VisionArena-Chat": lambda x: x["conversation"][0][0]["content"],
        "lmarena-ai/vision-arena-bench-v0.1": lambda x: x["turns"][0][0]["content"],
    }
    IS_MULTIMODAL = True

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        parser_fn = self.SUPPORTED_DATASET_PATHS.get(self.hf_name)
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `VisionArenaDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Vision Arena Dataset.
**CN:** 类 `VisionArenaDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Vision Arena Dataset.

### Method `VisionArenaDataset.sample` (lines 2894-2937)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        parser_fn = self.SUPPORTED_DATASET_PATHS.get(self.hf_name)
        if parser_fn is None:
            raise ValueError(f"Unsupported dataset path: {self.hf_name}")

        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN

        sampled_requests = []
        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `VisionArenaDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `self.SUPPORTED_DATASET_PATHS.get`, `ValueError`, `enumerate`, `len`, `parser_fn` show the concrete execution path.
**CN:** Method `VisionArenaDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `self.SUPPORTED_DATASET_PATHS.get`, `ValueError`, `enumerate`, `len`, `parser_fn` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MMVUDataset` (lines 2940-3008)
```python
class MMVUDataset(HuggingFaceDataset):
    """
    MMVU Dataset.
    https://huggingface.co/datasets/yale-nlp/MMVU
    """

    DEFAULT_OUTPUT_LEN = 128
    SUPPORTED_DATASET_PATHS = {
        "yale-nlp/MMVU": lambda x: (
            x["question"]
            + " "
            + (" ".join(f"{k}.{v}" for k, v in x["choices"].items()))
        ),
    }

    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        self._remote_path_root = (
            f"https://huggingface.co/datasets/{self.hf_name}/resolve/main"
        )
        self._local_path_root = snapshot_download(self.hf_name, repo_type="dataset")

    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `MMVUDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `__init__`, `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: MMVU Dataset.
**CN:** 类 `MMVUDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `__init__`, `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：MMVU Dataset.

### Method `MMVUDataset.__init__` (lines 2955-2961)
```python
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        self._remote_path_root = (
            f"https://huggingface.co/datasets/{self.hf_name}/resolve/main"
        )
        self._local_path_root = snapshot_download(self.hf_name, repo_type="dataset")
```
**EN:** Method `MMVUDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `snapshot_download` show the concrete execution path.
**CN:** Method `MMVUDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `snapshot_download` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MMVUDataset.sample` (lines 2963-3008)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        parser_fn = self.SUPPORTED_DATASET_PATHS.get(self.hf_name)
        if parser_fn is None:
            raise ValueError(f"Unsupported dataset path: {self.hf_name}")

        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN

        sampled_requests = []
        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `MMVUDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `self.SUPPORTED_DATASET_PATHS.get`, `ValueError`, `enumerate`, `len`, `parser_fn` show the concrete execution path.
**CN:** Method `MMVUDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `self.SUPPORTED_DATASET_PATHS.get`, `ValueError`, `enumerate`, `len`, `parser_fn` 这样的关键调用展示了该代码块的具体执行路径。

### Class `InstructCoderDataset` (lines 3016-3073)
```python
class InstructCoderDataset(HuggingFaceDataset):
    """
    InstructCoder Dataset.
    https://huggingface.co/datasets/likaixin/InstructCoder

    InstructCoder is the dataset designed for general code editing.  It consists
    of 114,239 instruction-input-output triplets, and covers multiple distinct
    code editing scenario.
    """

    DEFAULT_OUTPUT_LEN = 200  # this is the average default output length
    SUPPORTED_DATASET_PATHS = {
        "likaixin/InstructCoder",
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
    # ... omitted for brevity ...
            )
            yield prompt
```
**EN:** Class `InstructCoderDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, `sample_prompts`, which define initialization, validation, transformation, or access patterns. The class docstring says: InstructCoder Dataset.
**CN:** 类 `InstructCoderDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`, `sample_prompts`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：InstructCoder Dataset.

### Method `InstructCoderDataset.sample` (lines 3031-3065)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests: list[SampleRequest] = []
        for i, prompt in enumerate(self.sample_prompts(n=num_requests)):
            # apply template
            if not skip_chat_template:
                prompt = tokenizer.apply_chat_template(
                    [{"role": "user", "content": prompt}],
                    add_generation_prompt=True,
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `InstructCoderDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `enumerate`, `self.sample_prompts`, `tokenizer.apply_chat_template`, `len`, `tokenizer` show the concrete execution path.
**CN:** Method `InstructCoderDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `enumerate`, `self.sample_prompts`, `tokenizer.apply_chat_template`, `len`, `tokenizer` 这样的关键调用展示了该代码块的具体执行路径。

### Method `InstructCoderDataset.sample_prompts` (lines 3067-3073)
```python
    def sample_prompts(self, n: int) -> Iterator[str]:
        for item in self.data.take(n):
            prompt = (
                f"{item['input']}\n\n{item['instruction']} Just output "
                "the code, do not include any explanation."
            )
            yield prompt
```
**EN:** Method `InstructCoderDataset.sample_prompts` samples, slices, or reshapes data for downstream use. Key calls such as `self.data.take` show the concrete execution path.
**CN:** Method `InstructCoderDataset.sample_prompts` 负责为下游流程采样、切片或重组数据。 像 `self.data.take` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MTBenchDataset` (lines 3081-3135)
```python
class MTBenchDataset(HuggingFaceDataset):
    """
    MT-Bench Dataset.
    https://huggingface.co/datasets/philschmid/mt-bench

    We create a single turn dataset for MT-Bench.
    This is similar to Spec decoding benchmark setup in vLLM
    https://github.com/vllm-project/vllm/blob/9d98ab5ec/examples/offline_inference/eagle.py#L14-L18
    """  # noqa: E501

    DEFAULT_OUTPUT_LEN = 256  # avg len used in SD bench in vLLM
    SUPPORTED_DATASET_PATHS = {
        "philschmid/mt-bench",
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `MTBenchDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: MT-Bench Dataset.
**CN:** 类 `MTBenchDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：MT-Bench Dataset.

### Method `MTBenchDataset.sample` (lines 3096-3135)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests: list[SampleRequest] = []

        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
                break
            prompt = item["turns"][0]

    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `MTBenchDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` show the concrete execution path.
**CN:** Method `MTBenchDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `HumanEvalDataset` (lines 3143-3195)
```python
class HumanEvalDataset(HuggingFaceDataset):
    """
    HumanEvalDataset Dataset.
    https://huggingface.co/datasets/openai/openai_humaneval

    We create a single turn dataset for HumanEval.
    """

    DEFAULT_OUTPUT_LEN = 256
    SUPPORTED_DATASET_PATHS = {
        "openai/openai_humaneval",
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `HumanEvalDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: HumanEvalDataset Dataset.
**CN:** 类 `HumanEvalDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：HumanEvalDataset Dataset.

### Method `HumanEvalDataset.sample` (lines 3156-3195)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests = []

        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
                break
            prompt = item["prompt"]

    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `HumanEvalDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` show the concrete execution path.
**CN:** Method `HumanEvalDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `GSM8KDataset` (lines 3203-3255)
```python
class GSM8KDataset(HuggingFaceDataset):
    """
    GSM8K Dataset.
    https://huggingface.co/datasets/openai/gsm8k

    We create a single turn dataset for GSM8K.
    """

    DEFAULT_OUTPUT_LEN = 256
    SUPPORTED_DATASET_PATHS = {
        "openai/gsm8k",
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `GSM8KDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: GSM8K Dataset.
**CN:** 类 `GSM8KDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：GSM8K Dataset.

### Method `GSM8KDataset.sample` (lines 3216-3255)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        skip_chat_template: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests = []

        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
                break
            prompt = item["question"]

    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `GSM8KDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` show the concrete execution path.
**CN:** Method `GSM8KDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `BlazeditDataset` (lines 3263-3344)
```python
class BlazeditDataset(HuggingFaceDataset):
    """
    Blazedit Dataset.
    https://github.com/ise-uiuc/blazedit

    5k char version: vdaita/edit_5k_char
    10k char version: vdaita/edit_10k_char
    """  # noqa: E501

    # 5k char version will have output as ~5k chars
    # 10k char version will have output as ~10k chars
    # Assuming 3 char per token, 10k chars will be 3333 tokens
    # We set default to 4000 to be safe
    DEFAULT_OUTPUT_LEN = 4000
    SUPPORTED_DATASET_PATHS = {
        "vdaita/edit_5k_char",
        "vdaita/edit_10k_char",
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
    # ... omitted for brevity ...

        return sampled_requests
```
**EN:** Class `BlazeditDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Blazedit Dataset.
**CN:** 类 `BlazeditDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Blazedit Dataset.

### Method `BlazeditDataset.sample` (lines 3282-3344)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        output_len: int | None = None,
        skip_chat_template: bool = False,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        min_distance: float = 0.0,
        max_distance: float = 1.0,
        **kwargs,
    ) -> list[SampleRequest]:
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests = []

        for i, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
                break
            code = item["code"]
    # ... omitted for brevity ...

        return sampled_requests
```
**EN:** Method `BlazeditDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` show the concrete execution path.
**CN:** Method `BlazeditDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `enumerate`, `len`, `tokenizer.apply_chat_template`, `tokenizer`, `sampled_requests.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `AIMODataset` (lines 3352-3404)
```python
class AIMODataset(HuggingFaceDataset):
    """
    Dataset class for processing a AIMO dataset with reasoning questions.
    """

    SUPPORTED_DATASET_PATHS = {
        "AI-MO/aimo-validation-aime",
        "AI-MO/NuminaMath-1.5",
        "AI-MO/NuminaMath-CoT",
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        **kwargs,
    ) -> list[SampleRequest]:
        sampled_requests: list[SampleRequest] = []
        ind = 0
        dynamic_output = output_len is None
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `AIMODataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Dataset class for processing a AIMO dataset with reasoning questions.
**CN:** 类 `AIMODataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Dataset class for processing a AIMO dataset with reasoning questions.

### Method `AIMODataset.sample` (lines 3363-3404)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        **kwargs,
    ) -> list[SampleRequest]:
        sampled_requests: list[SampleRequest] = []
        ind = 0
        dynamic_output = output_len is None

        for item in self.data:
            if len(sampled_requests) >= num_requests:
                break
            prompt, completion = item["problem"], item["solution"]

            prompt_ids = tokenizer(prompt).input_ids
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `AIMODataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `len`, `tokenizer`, `isinstance`, `is_valid_sequence`, `sampled_requests.append` show the concrete execution path.
**CN:** Method `AIMODataset.sample` 负责为下游流程采样、切片或重组数据。 像 `len`, `tokenizer`, `isinstance`, `is_valid_sequence`, `sampled_requests.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `NextEditPredictionDataset` (lines 3461-3502)
```python
class NextEditPredictionDataset(HuggingFaceDataset):
    """
    Dataset class for processing a Next Edit Prediction dataset.
    """

    SUPPORTED_DATASET_PATHS = {
        "zed-industries/zeta",
    }
    MAPPING_PROMPT_FUNCS = {
        "zed-industries/zeta": _format_zeta_prompt,
    }

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        **kwargs,
    ):
        formatting_prompt_func = self.MAPPING_PROMPT_FUNCS.get(self.hf_name)
        if formatting_prompt_func is None:
            raise ValueError(f"Unsupported dataset path: {self.hf_name}")
    # ... omitted for brevity ...
        )
        return samples
```
**EN:** Class `NextEditPredictionDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Dataset class for processing a Next Edit Prediction dataset.
**CN:** 类 `NextEditPredictionDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Dataset class for processing a Next Edit Prediction dataset.

### Method `NextEditPredictionDataset.sample` (lines 3473-3502)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        **kwargs,
    ):
        formatting_prompt_func = self.MAPPING_PROMPT_FUNCS.get(self.hf_name)
        if formatting_prompt_func is None:
            raise ValueError(f"Unsupported dataset path: {self.hf_name}")
        samples = []
        for i, sample in enumerate(self.data):
            sample = formatting_prompt_func(sample)
            samples.append(
                SampleRequest(
                    prompt=sample["prompt"],
                    prompt_len=len(tokenizer(sample["prompt"]).input_ids),
                    expected_output_len=len(
    # ... omitted for brevity ...
        )
        return samples
```
**EN:** Method `NextEditPredictionDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `self.MAPPING_PROMPT_FUNCS.get`, `ValueError`, `enumerate`, `formatting_prompt_func`, `samples.append` show the concrete execution path.
**CN:** Method `NextEditPredictionDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `self.MAPPING_PROMPT_FUNCS.get`, `ValueError`, `enumerate`, `formatting_prompt_func`, `samples.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `ASRDataset` (lines 3510-3608)
```python
class ASRDataset(HuggingFaceDataset):
    """
    Dataset class for processing a ASR dataset for transcription.
    Tested on the following set:

    +----------------+----------------------------------------+--------------------------+-----------------------------+
    | Dataset        | Domain                                 | Speaking Style           | hf-subset                   |
    +----------------+----------------------------------------+--------------------------+-----------------------------+
    | TED-LIUM       | TED talks                              | Oratory                  | release1, release2, release3|
    |                |                                        |                          | release3-speaker-adaptation |
    | VoxPopuli      | European Parliament                    | Oratory                  | en, de, it, fr,  ...        |
    | LibriSpeech    | Audiobook                              | Narrated                 | "LIUM/tedlium"              |
    | GigaSpeech     | Audiobook, podcast, YouTube            | Narrated, spontaneous    | xs, s, m, l, xl, dev, test  |
    | SPGISpeech     | Financial meetings                     | Oratory, spontaneous     | S, M, L, dev, test          |
    | AMI            | Meetings                               | Spontaneous              | ihm, sdm                    |
    +----------------+----------------------------------------+--------------------------+-----------------------------+

    """  # noqa: E501

    SUPPORTED_DATASET_PATHS = {
        "openslr/librispeech_asr",
        "facebook/voxpopuli",
        "LIUM/tedlium",
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `ASRDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Dataset class for processing a ASR dataset for transcription.
**CN:** 类 `ASRDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Dataset class for processing a ASR dataset for transcription.

### Method `ASRDataset.sample` (lines 3541-3608)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        **kwargs,
    ) -> list[SampleRequest]:
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        if "openai" in getattr(tokenizer, "name_or_path", ""):
            prompt = "<|startoftranscript|><|en|><|transcribe|><|notimestamps|>"
        else:
            prompt = ""
        prompt_len = len(tokenizer(prompt).input_ids)
        sampled_requests: list[SampleRequest] = []
        ind = 0
        skipped = 0
        asr_min_audio_len_sec = kwargs.get("asr_min_audio_len_sec")
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `ASRDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `getattr`, `len`, `tokenizer`, `kwargs.get`, `get_audio_duration` show the concrete execution path.
**CN:** Method `ASRDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `getattr`, `len`, `tokenizer`, `kwargs.get`, `get_audio_duration` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MLPerfDataset` (lines 3616-3694)
```python
class MLPerfDataset(HuggingFaceDataset):
    """
    MLPerf Inference Dataset.

    Dataset on HF:
    https://huggingface.co/datasets/mgoin/mlperf-inference-llama2-data
    https://huggingface.co/datasets/mgoin/mlperf-inference-llama3.1-data

    Each record contains:
      - "system_prompt": system role instruction.
      - "question": user question.
      - "output": reference answer.

    We combine the system prompt and question into a chat-formatted prompt
    (using the tokenizer's chat template) and set the expected output length to
    the tokenized length of the provided reference answer.
    """

    SUPPORTED_DATASET_PATHS = {
        "mgoin/mlperf-inference-llama2-data",
        "mgoin/mlperf-inference-llama3.1-data",
    }

    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `MLPerfDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: MLPerf Inference Dataset.
**CN:** 类 `MLPerfDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：MLPerf Inference Dataset.

### Method `MLPerfDataset.sample` (lines 3639-3694)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        **kwargs,
    ) -> list[SampleRequest]:
        # Force dynamic output length based on reference completion.
        dynamic_output = output_len is None
        sampled_requests: list[SampleRequest] = []
        ind = 0

        for item in self.data:
            if len(sampled_requests) >= num_requests:
                break

            system_prompt = item["system_prompt"]
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `MLPerfDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `len`, `tokenizer.apply_chat_template`, `tokenizer`, `is_valid_sequence`, `sampled_requests.append` show the concrete execution path.
**CN:** Method `MLPerfDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `len`, `tokenizer.apply_chat_template`, `tokenizer`, `is_valid_sequence`, `sampled_requests.append` 这样的关键调用展示了该代码块的具体执行路径。

### Class `PrefixRepetitionRandomDataset` (lines 3702-3786)
```python
class PrefixRepetitionRandomDataset(BenchmarkDataset):
    # Default values copied from benchmark_serving.py for the repeated prefix
    # dataset.
    DEFAULT_PREFIX_LEN = 256
    DEFAULT_SUFFIX_LEN = 256
    DEFAULT_NUM_PREFIXES = 10
    DEFAULT_OUTPUT_LEN = 128

    def __init__(
        self,
        **kwargs,
    ) -> None:
        super().__init__(**kwargs)
        random.seed(self.random_seed)
        np.random.seed(self.random_seed)

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = DEFAULT_PREFIX_LEN,
    # ... omitted for brevity ...
            random.shuffle(requests)
        return requests
```
**EN:** Class `PrefixRepetitionRandomDataset` is a structured building block in this module. It inherits from `BenchmarkDataset`. Key methods include `__init__`, `sample`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `PrefixRepetitionRandomDataset` 是该模块中的结构化构件，继承自 `BenchmarkDataset`。 关键方法包括 `__init__`, `sample`，它们共同定义初始化、校验、变换或访问模式。

### Method `PrefixRepetitionRandomDataset.__init__` (lines 3710-3716)
```python
    def __init__(
        self,
        **kwargs,
    ) -> None:
        super().__init__(**kwargs)
        random.seed(self.random_seed)
        np.random.seed(self.random_seed)
```
**EN:** Method `PrefixRepetitionRandomDataset.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `random.seed`, `np.random.seed` show the concrete execution path.
**CN:** Method `PrefixRepetitionRandomDataset.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `random.seed`, `np.random.seed` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PrefixRepetitionRandomDataset.sample` (lines 3718-3786)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        prefix_len: int = DEFAULT_PREFIX_LEN,
        suffix_len: int = DEFAULT_SUFFIX_LEN,
        num_prefixes: int = DEFAULT_NUM_PREFIXES,
        output_len: int = DEFAULT_OUTPUT_LEN,
        **kwargs,
    ) -> list[SampleRequest]:
        vocab_size = tokenizer.vocab_size
        prompts_per_prefix = num_requests // num_prefixes
        if prompts_per_prefix == 0:
            raise ValueError(
                f"num_requests ({num_requests}) must be greater than or equal "
                f"to num_prefixes ({num_prefixes})"
            )
    # ... omitted for brevity ...
            random.shuffle(requests)
        return requests
```
**EN:** Method `PrefixRepetitionRandomDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `ValueError`, `np.random.randint(0, vocab_size, size=target_length).tolist`, `np.random.randint`, `gen_prompt_decode_to_target_len`, `range` show the concrete execution path.
**CN:** Method `PrefixRepetitionRandomDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `ValueError`, `np.random.randint(0, vocab_size, size=target_length).tolist`, `np.random.randint`, `gen_prompt_decode_to_target_len`, `range` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MMStarDataset` (lines 3794-3860)
```python
class MMStarDataset(HuggingFaceDataset):
    """
    Lin-Chen/MMStar: https://huggingface.co/datasets/Lin-Chen/MMStar
    refer to: https://github.com/sgl-project/SpecForge/pull/106
    """

    DEFAULT_OUTPUT_LEN = 128
    SUPPORTED_DATASET_PATHS = {"Lin-Chen/MMStar"}
    IS_MULTIMODAL = True

    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # If --hf-output-len is not set, use the default output length.
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests: list[SampleRequest] = []
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Class `MMStarDataset` is a structured building block in this module. It inherits from `HuggingFaceDataset`. Key methods include `sample`, which define initialization, validation, transformation, or access patterns. The class docstring says: Lin-Chen/MMStar: https://huggingface.co/datasets/Lin-Chen/MMStar refer to: https://github.com/sgl-project/SpecForge/pull/106
**CN:** 类 `MMStarDataset` 是该模块中的结构化构件，继承自 `HuggingFaceDataset`。 关键方法包括 `sample`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Lin-Chen/MMStar: https://huggingface.co/datasets/Lin-Chen/MMStar refer to: https://github.com/sgl-project/SpecForge/pull/106

### Method `MMStarDataset.sample` (lines 3804-3860)
```python
    def sample(
        self,
        tokenizer: TokenizerLike,
        num_requests: int,
        request_id_prefix: str = "",
        no_oversample: bool = False,
        output_len: int | None = None,
        enable_multimodal_chat: bool = False,
        **kwargs,
    ) -> list[SampleRequest]:
        # If --hf-output-len is not set, use the default output length.
        output_len = output_len if output_len is not None else self.DEFAULT_OUTPUT_LEN
        sampled_requests: list[SampleRequest] = []

        for ind, item in enumerate(self.data):
            if len(sampled_requests) >= num_requests:
                break
            # Split the question text from options
            # (keep only the part before "Options:").
    # ... omitted for brevity ...
        )
        return sampled_requests
```
**EN:** Method `MMStarDataset.sample` samples, slices, or reshapes data for downstream use. Key calls such as `enumerate`, `len`, `item.get`, `full_q.split('Options:', 1)[0].strip`, `full_q.split` show the concrete execution path.
**CN:** Method `MMStarDataset.sample` 负责为下游流程采样、切片或重组数据。 像 `enumerate`, `len`, `item.get`, `full_q.split('Options:', 1)[0].strip`, `full_q.split` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SpeedBench` (lines 3868-3920)
```python
class SpeedBench(CustomDataset):
    """
    SPEED-Bench dataset: https://huggingface.co/datasets/nvidia/SPEED-Bench

    Download the dataset using:

    `curl -LsSf https://raw.githubusercontent.com/NVIDIA-NeMo/Skills/refs/heads/main/nemo_skills/dataset/speed-bench/prepare.py | python3 -`
    """  # noqa: E501

    DOWNLOAD_SCRIPT_URL = "https://raw.githubusercontent.com/NVIDIA-NeMo/Skills/refs/heads/main/nemo_skills/dataset/speed-bench/prepare.py"

    def __init__(self, **kwargs) -> None:
        self.dataset_subset = kwargs.pop("dataset_subset", "qualitative")
        self.category = kwargs.pop("category", None)
        super().__init__(**kwargs)
        self.load_data()

    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        if not Path(self.dataset_path).is_dir():
            raise ValueError(
    # ... omitted for brevity ...
        if not getattr(self, "disable_shuffle", False):
            random.shuffle(self.data)
```
**EN:** Class `SpeedBench` is a structured building block in this module. It inherits from `CustomDataset`. Key methods include `__init__`, `load_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: SPEED-Bench dataset: https://huggingface.co/datasets/nvidia/SPEED-Bench Download the dataset using: `curl -LsSf https://raw.githubusercontent.com/NVIDIA-NeMo/Skills/refs/heads/m...
**CN:** 类 `SpeedBench` 是该模块中的结构化构件，继承自 `CustomDataset`。 关键方法包括 `__init__`, `load_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：SPEED-Bench dataset: https://huggingface.co/datasets/nvidia/SPEED-Bench Download the dataset using: `curl -LsSf https://raw.githubusercontent.com/NVIDIA-NeMo/Skills/refs/heads/m...

### Method `SpeedBench.__init__` (lines 3879-3883)
```python
    def __init__(self, **kwargs) -> None:
        self.dataset_subset = kwargs.pop("dataset_subset", "qualitative")
        self.category = kwargs.pop("category", None)
        super().__init__(**kwargs)
        self.load_data()
```
**EN:** Method `SpeedBench.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `kwargs.pop`, `super().__init__`, `super`, `self.load_data` show the concrete execution path.
**CN:** Method `SpeedBench.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `kwargs.pop`, `super().__init__`, `super`, `self.load_data` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpeedBench.load_data` (lines 3885-3920)
```python
    def load_data(self) -> None:
        if self.dataset_path is None:
            raise ValueError("dataset_path must be provided for loading data.")

        if not Path(self.dataset_path).is_dir():
            raise ValueError(
                f"dataset_path {self.dataset_path} is not a directory. "
                f"Please make sure to download the dataset from HuggingFace using "
                f"`curl -LsSf {self.DOWNLOAD_SCRIPT_URL} | python3 -`"
            )

        self.data = []

        # Load the JSONL file
        jsonl_data = pd.read_json(
            path_or_buf=Path(self.dataset_path) / f"{self.dataset_subset}.jsonl",
            lines=True,
        )

    # ... omitted for brevity ...
        if not getattr(self, "disable_shuffle", False):
            random.shuffle(self.data)
```
**EN:** Method `SpeedBench.load_data` handles loading or retrieval of external/internal data. Key calls such as `ValueError`, `Path(self.dataset_path).is_dir`, `Path`, `pd.read_json`, `jsonl_data.iterrows` show the concrete execution path.
**CN:** Method `SpeedBench.load_data` 负责加载或获取外部/内部数据。 像 `ValueError`, `Path(self.dataset_path).is_dir`, `Path`, `pd.read_json`, `jsonl_data.iterrows` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import ast`, `import io`, `import json`, `import logging`, `import math`, `import random`, `from abc import ABC, abstractmethod`, `from collections.abc import Callable, Iterator, Mapping`, `from contextlib import suppress`, `from dataclasses import dataclass, replace`, `from functools import cache`
- **Third-party / 第三方**: `import numpy as np`, `import pybase64 as base64`, `from huggingface_hub import snapshot_download`, `from PIL import Image`, `from typing_extensions import deprecated`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.datasets.utils import RangeRatio, _resolve_range_ratios, get_sampling_params`, `from vllm.inputs import MultiModalDataDict`, `from vllm.lora.request import LoRARequest`, `from vllm.lora.utils import get_adapter_absolute_path`, `from vllm.multimodal.audio import get_audio_duration`, `from vllm.multimodal.image import convert_image_mode`, `from vllm.tokenizers import TokenizerLike`, `from vllm.utils.argparse_utils import FlexibleArgumentParser`, `from vllm.utils.import_utils import PlaceholderModule`

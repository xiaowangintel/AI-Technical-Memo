# io_struct.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/io_struct.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements io struct logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 I/O struct 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Provide supporting module logic / 提供辅助模块逻辑
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""
The definition of objects transferred between different
processes (TokenizerManager, DetokenizerManager, Scheduler).
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 19-19: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 21-27: Import runtime dependencies / 导入运行时依赖
```python
import copy
import uuid
from abc import ABC
from collections import Counter
from dataclasses import dataclass, field
from enum import Enum
from typing import TYPE_CHECKING, Any, Dict, List, Literal, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 29-29: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 31-50: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.lora.lora_registry import LoRARef
from sglang.srt.managers.embed_types import PositionalEmbeds
from sglang.srt.managers.schedule_batch import BaseFinishReason, Modality
from sglang.srt.multimodal.mm_utils import has_valid_data
from sglang.srt.observability.req_time_stats import (
    APIServerReqTimeStats,
    DPControllerReqTimeStats,
    SchedulerReqTimeStats,
)
from sglang.srt.sampling.sampling_params import SamplingParams
from sglang.srt.utils import ImageData, VideoData

# Handle serialization of Image for pydantic
if TYPE_CHECKING:
    from PIL.Image import Image
else:
    Image = Any


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 51-70: Define class BaseReq / 定义类 BaseReq
```python
class BaseReq(ABC):
    rid: Optional[Union[str, List[str]]] = field(default=None, kw_only=True)
    http_worker_ipc: Optional[str] = field(default=None, kw_only=True)

    def regenerate_rid(self):
        """Generate a new request ID and return it."""
        if isinstance(self.rid, list):
            self.rid = [uuid.uuid4().hex for _ in range(len(self.rid))]
        else:
            self.rid = uuid.uuid4().hex
        return self.rid

    def _validate_rid_uniqueness(self):
        """Validate that request IDs within a batch are unique."""
        if isinstance(self.rid, list) and len(set(self.rid)) != len(self.rid):
            counts = Counter(self.rid)
            duplicates = [rid for rid, count in counts.items() if count > 1]
            raise ValueError(
                f"Duplicate request IDs detected within the request: {duplicates}"
            )
```
**EN:** This block declares the class `BaseReq`. It centers on coordinating io struct behavior, with methods such as regenerate_rid, _validate_rid_uniqueness.
**CN:** 该代码块声明类 `BaseReq`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 regenerate_rid, _validate_rid_uniqueness 等方法组织实现。

### Lines 71-73: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 74-81: Define class BaseBatchReq / 定义类 BaseBatchReq
```python
class BaseBatchReq(ABC):
    rids: Optional[List[str]] = field(default=None, kw_only=True)
    http_worker_ipcs: Optional[List[str]] = field(default=None, kw_only=True)

    def regenerate_rids(self):
        """Generate new request IDs and return them."""
        self.rids = [uuid.uuid4().hex for _ in range(len(self.rids))]
        return self.rids
```
**EN:** This block declares the class `BaseBatchReq`. It centers on coordinating io struct behavior, with methods such as regenerate_rids.
**CN:** 该代码块声明类 `BaseBatchReq`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 regenerate_rids 等方法组织实现。

### Lines 82-84: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 85-104: Define class SpeculativeDecodingMetricsMixin / 定义类 SpeculativeDecodingMetricsMixin
```python
class SpeculativeDecodingMetricsMixin:
    """
    Mixin class containing speculative decoding metrics.

    This class consolidates speculative decoding metrics that are shared across
    batch output types that support speculative decoding to avoid code duplication.
    """

    # Verify count: number of verification forward passes
    spec_verify_ct: List[int]

    # Accepted drafts: Number of accepted draft tokens during speculative decoding
    # (strict drafts-only count, excludes the bonus token).
    spec_num_correct_drafts: List[int]

    # Acceptance histogram: List of lists, where each inner list represents histogram counts.
    # List index = number of accepted tokens in a step, List value = count of steps with that many accepted tokens.
    # Example: histogram[0] = 5 means 5 steps with 0 accepted tokens, histogram[3] = 10 means 10 steps with 3 accepted tokens.
    # Empty list [] when speculative decoding is disabled.
    spec_correct_drafts_histogram: List[List[int]]
```
**EN:** This block declares the class `SpeculativeDecodingMetricsMixin`. It centers on Mixin class containing speculative decoding metrics..
**CN:** 该代码块声明类 `SpeculativeDecodingMetricsMixin`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 105-108: Provide supporting module logic / 提供辅助模块逻辑
```python


# Parameters for a session
@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 109-114: Define class SessionParams / 定义类 SessionParams
```python
class SessionParams:
    id: Optional[str] = None
    rid: Optional[str] = None
    offset: Optional[int] = None
    replace: Optional[bool] = None
    drop_previous_output: Optional[bool] = None
```
**EN:** This block declares the class `SessionParams`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SessionParams`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 115-174: Provide supporting module logic / 提供辅助模块逻辑
```python


# Type definitions for multimodal input data
# Individual data item types for each modality
ImageDataInputItem = Union[Image, str, ImageData, Dict]
AudioDataInputItem = Union[str, Dict]
VideoDataInputItem = Union[str, VideoData, Dict]
# Union type for any multimodal data item
MultimodalDataInputItem = Union[
    ImageDataInputItem, VideoDataInputItem, AudioDataInputItem
]
# Format types supporting single items, lists, or nested lists for batch processing
MultimodalDataInputFormat = Union[
    List[List[MultimodalDataInputItem]],
    List[MultimodalDataInputItem],
    MultimodalDataInputItem,
]


@dataclass
class GenerateReqInput(BaseReq):
    # The input prompt. It can be a single prompt or a batch of prompts.
    text: Optional[Union[List[str], str]] = None
    # The token ids for text; one can specify either text or input_ids
    input_ids: Optional[Union[List[List[int]], List[int]]] = None
    # The embeddings for input_ids; one can specify either text or input_ids or input_embeds.
    input_embeds: Optional[Union[List[List[List[float]]], List[List[float]]]] = None
    # The image input. It can be an image instance, file name, URL, or base64 encoded string.
    # Can be formatted as:
    # - Single image for a single request
    # - List of images (one per request in a batch)
    # - List of lists of images (multiple images per request)
    # See also python/sglang/srt/utils.py:load_image for more details.
    image_data: Optional[MultimodalDataInputFormat] = None
    # The video input. Like image data, it can be a file name, a url, or base64 encoded string.
    video_data: Optional[MultimodalDataInputFormat] = None
    # The audio input. Like image data, it can be a file name, a url, or base64 encoded string.
    audio_data: Optional[MultimodalDataInputFormat] = None
    # Whether to extract and process audio from video inputs.
    use_audio_in_video: bool = False
    # The sampling_params. See descriptions below.
    sampling_params: Optional[Union[List[Dict], Dict]] = None
    # Whether to return logprobs.
    return_logprob: Optional[Union[List[bool], bool]] = None
    # If return logprobs, the start location in the prompt for returning logprobs.
    # By default, this value is "-1", which means it will only return logprobs for output tokens.
    logprob_start_len: Optional[Union[List[int], int]] = None
    # If return logprobs, the number of top logprobs to return at each position.
    top_logprobs_num: Optional[Union[List[int], int]] = None
    # If return logprobs, the token ids to return logprob for.
    token_ids_logprob: Optional[Union[List[List[int]], List[int]]] = None
    # Whether to detokenize tokens in text in the returned logprobs.
    return_text_in_logprobs: bool = False
    # Whether to stream output.
    stream: bool = False
    # Whether to log metrics for this request (e.g. health_generate calls do not log metrics)
    log_metrics: bool = True
    # Whether to return hidden states
    return_hidden_states: Union[List[bool], bool] = False
    # Whether to return captured routed experts
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 175-234: Provide supporting module logic / 提供辅助模块逻辑
```python
    return_routed_experts: bool = False
    return_indexer_topk: bool = False
    # Absolute start position for returned routings; response covers
    # `[routed_experts_start_len, seqlen - 1)`. Must be in [0, prompt_tokens].
    # 0 = full sequence.
    routed_experts_start_len: int = 0

    # The modalities of the image data [image, multi-images, video]
    modalities: Optional[List[str]] = None
    # Session info for continual prompting
    session_params: Optional[Union[List[Dict], Dict]] = None

    # The path to the LoRA adaptors
    lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None
    # The uid of LoRA adaptors, should be initialized by tokenizer manager
    lora_id: Optional[Union[List[Optional[str]], Optional[str]]] = None

    # Custom logit processor for advanced sampling control. Must be a serialized instance
    # of `CustomLogitProcessor` in python/sglang/srt/sampling/custom_logit_processor.py
    # Use the processor's `to_str()` method to generate the serialized string.
    custom_logit_processor: Optional[Union[List[Optional[str]], str]] = None
    # Embedding overrides to place at specific token positions.
    # Runtime type: Optional[Union[PositionalEmbeds, List[Optional[PositionalEmbeds]]]]
    # Typed as Any to avoid Pydantic/FastAPI schema errors (PositionalEmbeds contains torch.Tensor).
    positional_embed_overrides: Any = None

    # For disaggregated inference
    bootstrap_host: Optional[Union[List[str], str]] = None
    bootstrap_port: Optional[Union[List[Optional[int]], int]] = None
    bootstrap_room: Optional[Union[List[int], int]] = None
    bootstrap_pair_key: Optional[Union[List[str], str]] = None
    decode_tp_size: Optional[Union[List[Optional[int]], int]] = None

    # Require reasoning for the request (hybrid reasoning model only)
    require_reasoning: bool = False

    # For DP routing — external router assigns a specific DP worker
    routed_dp_rank: Optional[int] = None
    # For PD disagg — hint telling decode which prefill DP worker has the KV cache
    disagg_prefill_dp_rank: Optional[int] = None
    # Deprecated: use routed_dp_rank instead
    data_parallel_rank: Optional[int] = None

    # For background responses (OpenAI responses API)
    background: bool = False

    # Conversation id used for tracking requests
    conversation_id: Optional[str] = None

    # Priority for the request
    priority: Optional[int] = None

    # Extra key for classifying the request (e.g. cache_salt)
    extra_key: Optional[Union[List[str], str]] = None

    # Routing key for routing-key schedule policy
    routing_key: Optional[str] = None

    # Whether to disallow logging for this request (e.g. due to ZDR)
    no_logs: bool = False
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 235-261: Provide supporting module logic / 提供辅助模块逻辑
```python

    # For custom metric labels
    custom_labels: Optional[Dict[str, str]] = None

    # (Internal) Whether to return bytes for image generation
    return_bytes: bool = False

    # Whether to return entropy
    return_entropy: bool = False

    # Propagates trace context via Engine.generate/async_generate
    external_trace_header: Optional[Dict] = None
    received_time: Optional[float] = None

    # For EPD-disaggregated inference
    need_wait_for_mm_inputs: Optional[bool] = None
    num_items_assigned: Optional[Dict[Modality, List[int]]] = None

    # Multimodal tiling controls (extensions)
    max_dynamic_patch: Optional[int] = None
    min_dynamic_patch: Optional[int] = None
    image_max_dynamic_patch: Optional[int] = None
    video_max_dynamic_patch: Optional[int] = None

    # Pre-computed delimiter indices for multi-item scoring.
    # Batch-level: List[List[int]] (one per request). After __getitem__: List[int].
    multi_item_delimiter_indices: Optional[Union[List[List[int]], List[int]]] = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 263-268: Implement contains mm input / 实现contains mm input
```python
    def contains_mm_input(self) -> bool:
        return (
            has_valid_data(self.image_data)
            or has_valid_data(self.video_data)
            or has_valid_data(self.audio_data)
        )
```
**EN:** This block implements the method `contains_mm_input()` on `GenerateReqInput`. It focuses on handling the io struct responsibilities represented by `contains_mm_input`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `contains_mm_input()`。它围绕 `contains_mm_input` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 270-304: Implement normalize batch and arguments / 实现normalize 批处理 and arguments
```python
    def normalize_batch_and_arguments(self):
        """
        Normalize the batch size and arguments for the request.

        This method resolves various input formats and ensures all parameters
        are properly formatted as either single values or batches depending on the input.
        It also handles parallel sampling expansion and sets default values for
        unspecified parameters.

        Raises:
            ValueError: If inputs are not properly specified (e.g., none or all of
                       text, input_ids, input_embeds are provided)
        """
        if self.data_parallel_rank is not None:
            import warnings

            warnings.warn(
                "'data_parallel_rank' is deprecated, use 'routed_dp_rank' instead.",
                DeprecationWarning,
                stacklevel=2,
            )
            if self.routed_dp_rank is None:
                self.routed_dp_rank = self.data_parallel_rank
            self.data_parallel_rank = None

        self._validate_inputs()
        self._determine_batch_size()
        self._handle_parallel_sampling()

        if self.is_single:
            self._normalize_single_inputs()
        else:
            self._normalize_batch_inputs()

        self._validate_rid_uniqueness()
```
**EN:** This block implements the method `normalize_batch_and_arguments()` on `GenerateReqInput`. It focuses on Normalize the batch size and arguments for the request., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `normalize_batch_and_arguments()`。它围绕 `normalize_batch_and_arguments` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 306-317: Implement validate inputs / 实现validate inputs
```python
    def _validate_inputs(self):
        """Validate that the input configuration is valid."""
        if (
            self.text is None and self.input_ids is None and self.input_embeds is None
        ) or (
            self.text is not None
            and self.input_ids is not None
            and self.input_embeds is not None
        ):
            raise ValueError(
                "Either text, input_ids or input_embeds should be provided."
            )
```
**EN:** This block implements the method `_validate_inputs()` on `GenerateReqInput`. It focuses on Validate that the input configuration is valid., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_validate_inputs()`。它围绕 `_validate_inputs` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 319-345: Implement determine batch size / 实现determine 批处理 size
```python
    def _determine_batch_size(self):
        """Determine if this is a single example or a batch and the batch size."""
        if self.text is not None:
            if isinstance(self.text, str):
                self.is_single = True
                self.batch_size = 1
            else:
                self.is_single = False
                self.batch_size = len(self.text)
            self.input_embeds = None
        elif self.input_ids is not None:
            if len(self.input_ids) == 0:
                raise ValueError("input_ids cannot be empty.")
            if isinstance(self.input_ids[0], int):
                self.is_single = True
                self.batch_size = 1
            else:
                self.is_single = False
                self.batch_size = len(self.input_ids)
            self.input_embeds = None
        else:
            if isinstance(self.input_embeds[0][0], float):
                self.is_single = True
                self.batch_size = 1
            else:
                self.is_single = False
                self.batch_size = len(self.input_embeds)
```
**EN:** This block implements the method `_determine_batch_size()` on `GenerateReqInput`. It focuses on Determine if this is a single example or a batch and the batch size., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_determine_batch_size()`。它围绕 `_determine_batch_size` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 347-371: Implement handle parallel sampling / 实现handle 并行 sampling
```python
    def _handle_parallel_sampling(self):
        """Handle parallel sampling parameters and adjust batch size if needed."""
        # Determine parallel sample count
        if self.sampling_params is None:
            self.parallel_sample_num = 1
            return
        elif isinstance(self.sampling_params, dict):
            self.parallel_sample_num = self.sampling_params.get("n", 1)
        else:  # isinstance(self.sampling_params, list):
            self.parallel_sample_num = self.sampling_params[0].get("n", 1)
            for sampling_params in self.sampling_params:
                if self.parallel_sample_num != sampling_params.get("n", 1):
                    raise ValueError(
                        "The parallel_sample_num should be the same for all samples in sample params."
                    )

        # If using parallel sampling with a single example, convert to batch
        if self.parallel_sample_num > 1 and self.is_single:
            self.is_single = False
            if self.text is not None:
                self.text = [self.text]
            if self.input_ids is not None:
                self.input_ids = [self.input_ids]
            if self.input_embeds is not None:
                self.input_embeds = [self.input_embeds]
```
**EN:** This block implements the method `_handle_parallel_sampling()` on `GenerateReqInput`. It focuses on Handle parallel sampling parameters and adjust batch size if needed., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_handle_parallel_sampling()`。它围绕 `_handle_parallel_sampling` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 373-386: Implement normalize single inputs / 实现normalize single inputs
```python
    def _normalize_single_inputs(self):
        """Normalize inputs for a single example."""
        if self.sampling_params is None:
            self.sampling_params = {}
        if self.rid is None:
            self.rid = uuid.uuid4().hex
        if self.return_logprob is None:
            self.return_logprob = False
        if self.logprob_start_len is None:
            self.logprob_start_len = -1
        if self.top_logprobs_num is None:
            self.top_logprobs_num = 0
        if not self.token_ids_logprob:  # covers both None and []
            self.token_ids_logprob = None
```
**EN:** This block implements the method `_normalize_single_inputs()` on `GenerateReqInput`. It focuses on Normalize inputs for a single example., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_single_inputs()`。它围绕 `_normalize_single_inputs` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 388-407: Implement normalize batch inputs / 实现normalize 批处理 inputs
```python
    def _normalize_batch_inputs(self):
        """Normalize inputs for a batch of examples, including parallel sampling expansion."""
        # Calculate expanded batch size
        if self.parallel_sample_num == 1:
            num = self.batch_size
        else:
            # Expand parallel_sample_num
            num = self.batch_size * self.parallel_sample_num

        # Expand input based on type
        self._expand_inputs(num)
        self._normalize_rid(num)
        self._normalize_lora_paths(num)
        self._normalize_image_data(num)
        self._normalize_video_data(num)
        self._normalize_audio_data(num)
        self._normalize_sampling_params(num)
        self._normalize_logprob_params(num)
        self._normalize_custom_logit_processor(num)
        self._normalize_bootstrap_params(num)
```
**EN:** This block implements the method `_normalize_batch_inputs()` on `GenerateReqInput`. It focuses on Normalize inputs for a batch of examples, including parallel sampling expansion., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_batch_inputs()`。它围绕 `_normalize_batch_inputs` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 409-426: Implement expand inputs / 实现expand inputs
```python
    def _expand_inputs(self, num):
        """Expand the main inputs (text, input_ids, input_embeds) for parallel sampling."""
        if self.text is not None:
            if not isinstance(self.text, list):
                raise ValueError("Text should be a list for batch processing.")
            self.text = self.text * self.parallel_sample_num
        elif self.input_ids is not None:
            if not isinstance(self.input_ids, list) or not isinstance(
                self.input_ids[0], list
            ):
                raise ValueError(
                    "input_ids should be a list of lists for batch processing."
                )
            self.input_ids = self.input_ids * self.parallel_sample_num
        elif self.input_embeds is not None:
            if not isinstance(self.input_embeds, list):
                raise ValueError("input_embeds should be a list for batch processing.")
            self.input_embeds = self.input_embeds * self.parallel_sample_num
```
**EN:** This block implements the method `_expand_inputs(num)` on `GenerateReqInput`. It focuses on Expand the main inputs (text, input_ids, input_embeds) for parallel sampling., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_expand_inputs(num)`。它围绕 `_expand_inputs` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 428-436: Implement normalize lora paths / 实现normalize lora paths
```python
    def _normalize_lora_paths(self, num):
        """Normalize LoRA paths for batch processing."""
        if self.lora_path is not None:
            if isinstance(self.lora_path, str):
                self.lora_path = [self.lora_path] * num
            elif isinstance(self.lora_path, list):
                self.lora_path = self.lora_path * self.parallel_sample_num
            else:
                raise ValueError("lora_path should be a list or a string.")
```
**EN:** This block implements the method `_normalize_lora_paths(num)` on `GenerateReqInput`. It focuses on Normalize LoRA paths for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_lora_paths(num)`。它围绕 `_normalize_lora_paths` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 438-478: Implement normalize image data / 实现normalize 图像 数据
```python
    def _normalize_image_data(self, num):
        """Normalize image data for batch processing."""
        if self.image_data is None:
            self.image_data = [None] * num
        elif not isinstance(self.image_data, list):
            # Single image, convert to list of single-image lists
            self.image_data = [[self.image_data]] * num
            self.modalities = ["image"] * num
        elif isinstance(self.image_data, list):
            # Handle empty list case - treat as no images
            if len(self.image_data) == 0:
                self.image_data = [None] * num
                return

            if len(self.image_data) != self.batch_size:
                raise ValueError(
                    "The length of image_data should be equal to the batch size."
                )

            self.modalities = []
            if len(self.image_data) > 0 and isinstance(self.image_data[0], list):
                # Already a list of lists, keep as is
                for i in range(len(self.image_data)):
                    if self.image_data[i] is None or self.image_data[i] == [None]:
                        self.modalities.append(None)
                    elif len(self.image_data[i]) == 1:
                        self.modalities.append("image")
                    elif len(self.image_data[i]) > 1:
                        self.modalities.append("multi-images")
                    else:
                        # Ensure len(self.modalities) == len(self.image_data)
                        self.modalities.append(None)
                # Expand parallel_sample_num
                self.image_data = self.image_data * self.parallel_sample_num
                self.modalities = self.modalities * self.parallel_sample_num
            else:
                # List of images for a batch, wrap each in a list
                wrapped_images = [[img] for img in self.image_data]
                # Expand for parallel sampling
                self.image_data = wrapped_images * self.parallel_sample_num
                self.modalities = ["image"] * num
```
**EN:** This block implements the method `_normalize_image_data(num)` on `GenerateReqInput`. It focuses on Normalize image data for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_image_data(num)`。它围绕 `_normalize_image_data` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 480-487: Implement normalize video data / 实现normalize video 数据
```python
    def _normalize_video_data(self, num):
        """Normalize video data for batch processing."""
        if self.video_data is None:
            self.video_data = [None] * num
        elif not isinstance(self.video_data, list):
            self.video_data = [self.video_data] * num
        elif isinstance(self.video_data, list):
            self.video_data = self.video_data * self.parallel_sample_num
```
**EN:** This block implements the method `_normalize_video_data(num)` on `GenerateReqInput`. It focuses on Normalize video data for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_video_data(num)`。它围绕 `_normalize_video_data` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 489-496: Implement normalize audio data / 实现normalize audio 数据
```python
    def _normalize_audio_data(self, num):
        """Normalize audio data for batch processing."""
        if self.audio_data is None:
            self.audio_data = [None] * num
        elif not isinstance(self.audio_data, list):
            self.audio_data = [self.audio_data] * num
        elif isinstance(self.audio_data, list):
            self.audio_data = self.audio_data * self.parallel_sample_num
```
**EN:** This block implements the method `_normalize_audio_data(num)` on `GenerateReqInput`. It focuses on Normalize audio data for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_audio_data(num)`。它围绕 `_normalize_audio_data` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 498-505: Implement normalize sampling params / 实现normalize sampling params
```python
    def _normalize_sampling_params(self, num):
        """Normalize sampling parameters for batch processing."""
        if self.sampling_params is None:
            self.sampling_params = [{}] * num
        elif isinstance(self.sampling_params, dict):
            self.sampling_params = [self.sampling_params] * num
        else:  # Already a list
            self.sampling_params = self.sampling_params * self.parallel_sample_num
```
**EN:** This block implements the method `_normalize_sampling_params(num)` on `GenerateReqInput`. It focuses on Normalize sampling parameters for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_sampling_params(num)`。它围绕 `_normalize_sampling_params` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 507-522: Implement normalize rid / 实现normalize rid
```python
    def _normalize_rid(self, num):
        """Normalize request IDs for batch processing."""
        if self.rid is None:
            self.rid = [uuid.uuid4().hex for _ in range(num)]
        elif isinstance(self.rid, str):
            new_rids = [f"{self.rid}_{i}" for i in range(num)]
            self.rid = new_rids
        elif isinstance(self.rid, list):
            # Note: the length of rid shall be the same as the batch_size,
            # as the rid would be expanded for parallel sampling in tokenizer_manager
            if len(self.rid) != self.batch_size:
                raise ValueError(
                    "The specified rids length mismatch with the batch_size for batch processing."
                )
        else:
            raise ValueError("The rid should be a string or a list of strings.")
```
**EN:** This block implements the method `_normalize_rid(num)` on `GenerateReqInput`. It focuses on Normalize request IDs for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_rid(num)`。它围绕 `_normalize_rid` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 524-563: Implement normalize logprob params / 实现normalize logprob params
```python
    def _normalize_logprob_params(self, num):
        """Normalize logprob-related parameters for batch processing."""

        # Helper function to normalize a parameter
        def normalize_param(param, default_value, param_name):
            if param is None:
                return [default_value] * num
            elif not isinstance(param, list):
                return [param] * num
            else:
                if self.parallel_sample_num > 1:
                    raise ValueError(
                        f"Cannot use list {param_name} with parallel_sample_num > 1"
                    )
                return param

        # Normalize each logprob parameter
        self.return_logprob = normalize_param(
            self.return_logprob, False, "return_logprob"
        )
        self.logprob_start_len = normalize_param(
            self.logprob_start_len, -1, "logprob_start_len"
        )
        self.top_logprobs_num = normalize_param(
            self.top_logprobs_num, 0, "top_logprobs_num"
        )

        # Handle token_ids_logprob specially due to its nested structure
        if not self.token_ids_logprob:  # covers both None and []
            self.token_ids_logprob = [None] * num
        elif not isinstance(self.token_ids_logprob, list):
            self.token_ids_logprob = [[self.token_ids_logprob] for _ in range(num)]
        elif not isinstance(self.token_ids_logprob[0], list):
            self.token_ids_logprob = [
                copy.deepcopy(self.token_ids_logprob) for _ in range(num)
            ]
        elif self.parallel_sample_num > 1:
            raise ValueError(
                "Cannot use list token_ids_logprob with parallel_sample_num > 1"
            )
```
**EN:** This block implements the method `_normalize_logprob_params(num)` on `GenerateReqInput`. It focuses on Normalize logprob-related parameters for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_logprob_params(num)`。它围绕 `_normalize_logprob_params` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 565-574: Implement normalize custom logit processor / 实现normalize custom logit 处理器
```python
    def _normalize_custom_logit_processor(self, num):
        """Normalize custom logit processor for batch processing."""
        if self.custom_logit_processor is None:
            self.custom_logit_processor = [None] * num
        elif not isinstance(self.custom_logit_processor, list):
            self.custom_logit_processor = [self.custom_logit_processor] * num
        elif self.parallel_sample_num > 1:
            raise ValueError(
                "Cannot use list custom_logit_processor with parallel_sample_num > 1"
            )
```
**EN:** This block implements the method `_normalize_custom_logit_processor(num)` on `GenerateReqInput`. It focuses on Normalize custom logit processor for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_custom_logit_processor(num)`。它围绕 `_normalize_custom_logit_processor` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 576-608: Implement normalize bootstrap params / 实现normalize bootstrap params
```python
    def _normalize_bootstrap_params(self, num):
        """Normalize bootstrap parameters for batch processing."""
        # Normalize bootstrap_host
        if self.bootstrap_host is None:
            self.bootstrap_host = [None] * num
        elif not isinstance(self.bootstrap_host, list):
            self.bootstrap_host = [self.bootstrap_host] * num
        elif isinstance(self.bootstrap_host, list):
            self.bootstrap_host = self.bootstrap_host * self.parallel_sample_num

        # Normalize bootstrap_port
        if self.bootstrap_port is None:
            self.bootstrap_port = [None] * num
        elif not isinstance(self.bootstrap_port, list):
            self.bootstrap_port = [self.bootstrap_port] * num
        elif isinstance(self.bootstrap_port, list):
            self.bootstrap_port = self.bootstrap_port * self.parallel_sample_num

        # Normalize bootstrap_room
        if self.bootstrap_room is None:
            self.bootstrap_room = [None] * num
        elif not isinstance(self.bootstrap_room, list):
            self.bootstrap_room = [self.bootstrap_room + i for i in range(num)]
        elif isinstance(self.bootstrap_room, list):
            self.bootstrap_room = self.bootstrap_room * self.parallel_sample_num

        # Normalize bootstrap_pair_key
        if self.bootstrap_pair_key is None:
            self.bootstrap_pair_key = [None] * num
        elif not isinstance(self.bootstrap_pair_key, list):
            self.bootstrap_pair_key = [self.bootstrap_pair_key] * num
        elif isinstance(self.bootstrap_pair_key, list):
            self.bootstrap_pair_key = self.bootstrap_pair_key * self.parallel_sample_num
```
**EN:** This block implements the method `_normalize_bootstrap_params(num)` on `GenerateReqInput`. It focuses on Normalize bootstrap parameters for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_normalize_bootstrap_params(num)`。它围绕 `_normalize_bootstrap_params` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 610-616: Implement validate session params / 实现validate 会话 params
```python
    def _validate_session_params(self):
        """Validate that session parameters are properly formatted."""
        if self.session_params is not None:
            if not isinstance(self.session_params, dict) and not isinstance(
                self.session_params[0], dict
            ):
                raise ValueError("Session params must be a dict or a list of dicts.")
```
**EN:** This block implements the method `_validate_session_params()` on `GenerateReqInput`. It focuses on Validate that session parameters are properly formatted., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_validate_session_params()`。它围绕 `_validate_session_params` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 618-626: Implement get positional embed overrides item / 实现get positional embed overrides item
```python
    def _get_positional_embed_overrides_item(
        self, i: int
    ) -> Optional[PositionalEmbeds]:
        """Extract the i-th item from positional_embed_overrides."""
        if self.positional_embed_overrides is None:
            return None
        if isinstance(self.positional_embed_overrides, PositionalEmbeds):
            return self.positional_embed_overrides
        return self.positional_embed_overrides[i]
```
**EN:** This block implements the method `_get_positional_embed_overrides_item(i)` on `GenerateReqInput`. It focuses on Extract the i-th item from positional_embed_overrides., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `_get_positional_embed_overrides_item(i)`。它围绕 `_get_positional_embed_overrides_item` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 628-663: Implement getitem / 实现getitem
```python
    def __getitem__(self, i):
        # Cache sub-objects so that repeated obj[i] calls return the same instance.
        # This avoids subtle bugs where different call sites get divergent objects.
        cache = self.__dict__.setdefault("_sub_obj_cache", {})
        if i in cache:
            return cache[i]
        sub = GenerateReqInput(
            text=self.text[i] if self.text is not None else None,
            input_ids=self.input_ids[i] if self.input_ids is not None else None,
            input_embeds=(
                self.input_embeds[i] if self.input_embeds is not None else None
            ),
            positional_embed_overrides=self._get_positional_embed_overrides_item(i),
            image_data=self.image_data[i],
            video_data=self.video_data[i],
            audio_data=self.audio_data[i],
            sampling_params=self.sampling_params[i],
            rid=self.rid[i],
            return_logprob=self.return_logprob[i],
            logprob_start_len=self.logprob_start_len[i],
            top_logprobs_num=self.top_logprobs_num[i],
            token_ids_logprob=self.token_ids_logprob[i],
            return_text_in_logprobs=self.return_text_in_logprobs,
            stream=self.stream,
            log_metrics=self.log_metrics,
            return_hidden_states=(
                self.return_hidden_states[i]
                if isinstance(self.return_hidden_states, list)
                else self.return_hidden_states
            ),
            return_routed_experts=self.return_routed_experts,
            routed_experts_start_len=self.routed_experts_start_len,
            return_indexer_topk=self.return_indexer_topk,
            modalities=self.modalities[i] if self.modalities else None,
            session_params=self.session_params,
            lora_path=self.lora_path[i] if self.lora_path is not None else None,
```
**EN:** This block implements the method `__getitem__(i)` on `GenerateReqInput`. It focuses on handling the io struct responsibilities represented by `__getitem__`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `__getitem__(i)`。它围绕 `__getitem__` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 664-699: Continue getitem / 继续说明getitem
```python
            lora_id=self.lora_id[i] if self.lora_id is not None else None,
            custom_logit_processor=(
                self.custom_logit_processor[i]
                if self.custom_logit_processor is not None
                else None
            ),
            # if `__getitem__` is called, the bootstrap_host, bootstrap_port, bootstrap_room must be a list
            bootstrap_host=(
                self.bootstrap_host[i] if self.bootstrap_host is not None else None
            ),
            bootstrap_port=(
                self.bootstrap_port[i] if self.bootstrap_port is not None else None
            ),
            bootstrap_room=(
                self.bootstrap_room[i] if self.bootstrap_room is not None else None
            ),
            bootstrap_pair_key=(
                self.bootstrap_pair_key[i]
                if self.bootstrap_pair_key is not None
                else None
            ),
            decode_tp_size=(
                self.decode_tp_size[i] if self.decode_tp_size is not None else None
            ),
            routed_dp_rank=self.routed_dp_rank,
            disagg_prefill_dp_rank=self.disagg_prefill_dp_rank,
            conversation_id=self.conversation_id,
            priority=self.priority,
            extra_key=self.extra_key,
            no_logs=self.no_logs,
            custom_labels=self.custom_labels,
            return_bytes=self.return_bytes,
            return_entropy=self.return_entropy,
            external_trace_header=self.external_trace_header,
            http_worker_ipc=self.http_worker_ipc,
            received_time=self.received_time,
```
**EN:** This block implements the method `__getitem__(i)` on `GenerateReqInput`. It focuses on handling the io struct responsibilities represented by `__getitem__`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `__getitem__(i)`。它围绕 `__getitem__` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 700-707: Continue getitem / 继续说明getitem
```python
            multi_item_delimiter_indices=(
                self.multi_item_delimiter_indices[i]
                if self.multi_item_delimiter_indices is not None
                else None
            ),
        )
        cache[i] = sub
        return sub
```
**EN:** This block implements the method `__getitem__(i)` on `GenerateReqInput`. It focuses on handling the io struct responsibilities represented by `__getitem__`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `GenerateReqInput` 上的方法 `__getitem__(i)`。它围绕 `__getitem__` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 708-710: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 711-800: Define class TokenizedGenerateReqInput / 定义类 TokenizedGenerateReqInput
```python
class TokenizedGenerateReqInput(BaseReq):
    # The input text
    input_text: str
    # The input token ids
    input_ids: List[int]
    # The multimodal inputs
    mm_inputs: object
    # The sampling parameters
    sampling_params: SamplingParams
    # Whether to return the logprobs
    return_logprob: bool
    # If return logprobs, the start location in the prompt for returning logprobs.
    logprob_start_len: int
    # If return logprobs, the number of top logprobs to return at each position.
    top_logprobs_num: int
    # If return logprobs, the token id to return logprob for
    token_ids_logprob: List[int]
    # Whether to stream output
    stream: bool

    # Whether to return hidden states
    return_hidden_states: bool = False

    # Whether to return captured routed experts
    return_routed_experts: bool = False
    # See GenerateReqInput.routed_experts_start_len.
    routed_experts_start_len: int = 0

    return_indexer_topk: bool = False

    # The input embeds
    input_embeds: Optional[Union[List[List[List[float]]], List[List[float]]]] = None

    # Embedding overrides to place at specific token positions.
    positional_embed_overrides: Optional[PositionalEmbeds] = None

    # Session info for continual prompting
    session_params: Optional[SessionParams] = None

    # LoRA related
    lora_id: Optional[str] = None  # None means just use the base model

    # Custom logit processor for advanced sampling control. Must be a serialized instance
    # of `CustomLogitProcessor` in python/sglang/srt/sampling/custom_logit_processor.py
    # Use the processor's `to_str()` method to generate the serialized string.
    custom_logit_processor: Optional[str] = None

    # For disaggregated inference
    bootstrap_host: Optional[str] = None
    bootstrap_port: Optional[int] = None
    bootstrap_room: Optional[int] = None
    bootstrap_pair_key: Optional[str] = None
    decode_tp_size: Optional[int] = None

    # Require reasoning for the request (hybrid reasoning model only)
    require_reasoning: bool = False

    # For DP routing
    routed_dp_rank: Optional[int] = None
    # For PD disagg — hint telling decode which prefill DP worker has the KV cache
    disagg_prefill_dp_rank: Optional[int] = None

    # Priority for the request
    priority: Optional[int] = None

    # Extra key for classifying the request (e.g. cache_salt)
    extra_key: Optional[str] = None

    # Routing key for routing-key schedule policy
    routing_key: Optional[str] = None

    # Whether to disallow logging for this request (e.g. due to ZDR)
    no_logs: bool = False

    # (Internal) Whether to return bytes for image generation
    return_bytes: bool = False

    # Whether to return entropy
    return_entropy: bool = False

    token_type_ids: Optional[List[int]] = None

    need_wait_for_mm_inputs: bool = False
    num_items_assigned: Optional[Dict[Modality, List[int]]] = None

    # Pre-computed delimiter indices for multi-item scoring
    multi_item_delimiter_indices: Optional[List[int]] = None

    # For observability
    time_stats: Optional[Union[APIServerReqTimeStats, DPControllerReqTimeStats]] = None
```
**EN:** This block declares the class `TokenizedGenerateReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `TokenizedGenerateReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 801-803: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 804-815: Define class BatchTokenizedGenerateReqInput / 定义类 BatchTokenizedGenerateReqInput
```python
class BatchTokenizedGenerateReqInput(BaseBatchReq):
    # The batch of tokenized requests
    batch: List[TokenizedGenerateReqInput]

    def __len__(self):
        return len(self.batch)

    def __getitem__(self, i):
        return self.batch[i]

    def __iter__(self):
        return iter(self.batch)
```
**EN:** This block declares the class `BatchTokenizedGenerateReqInput`. It centers on coordinating io struct behavior, with methods such as __len__, __getitem__, __iter__.
**CN:** 该代码块声明类 `BatchTokenizedGenerateReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 __len__, __getitem__, __iter__ 等方法组织实现。

### Lines 816-875: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
class EmbeddingReqInput(BaseReq):
    # The input prompt. It can be a single prompt or a batch of prompts.
    text: Optional[Union[List[List[str]], List[str], str]] = None
    # The image input. It can be an image instance, file name, URL, or base64 encoded string.
    # Can be formatted as:
    # - Single image for a single request
    # - List of images (one per request in a batch)
    # - List of lists of images (multiple images per request)
    # See also python/sglang/srt/utils.py:load_image for more details.
    image_data: Optional[MultimodalDataInputFormat] = None
    # The video input. Like image data, it can be a file name, a url, or base64 encoded string.
    video_data: Optional[MultimodalDataInputFormat] = None
    # The audio input. Like image data, it can be a file name, a url, or base64 encoded string.
    audio_data: Optional[MultimodalDataInputFormat] = None
    # The token ids for text; one can either specify text or input_ids.
    input_ids: Optional[Union[List[List[int]], List[int]]] = None
    # Placeholder token ID used to locate embedding override positions in input token IDs.
    embed_override_token_id: Optional[int] = None
    # Unresolved embedding overrides: per-input list of tensors.
    # Position resolution happens in the tokenizer manager after tokenization.
    # Shape: [num_inputs][num_replacements] where each entry is a torch.Tensor of [hidden_size].
    # Per-input entry may be None when only some inputs in a batch need overrides.
    # Runtime type: Optional[List[Optional[List[torch.Tensor]]]]
    # Typed as Any to avoid Pydantic/FastAPI schema errors (contains torch.Tensor).
    embed_overrides: Any = None
    # Resolved embedding overrides with positions (set by tokenizer manager or score mixin).
    # Runtime type: Optional[Union[PositionalEmbeds, List[Optional[PositionalEmbeds]]]]
    positional_embed_overrides: Any = None
    # Dummy sampling params for compatibility
    sampling_params: Optional[Union[List[Dict], Dict]] = None
    # Dummy input embeds for compatibility
    input_embeds: Optional[Union[List[List[List[float]]], List[List[float]]]] = None
    # Whether to log metrics for this request (e.g. health_generate calls do not log metrics)
    log_metrics: bool = True
    # The modalities of the image data [image, multi-images, video]
    modalities: Optional[List[str]] = None
    # For cross-encoder requests
    is_cross_encoder_request: bool = False
    # Priority for the request
    priority: Optional[int] = None
    # Routing key for routing-key schedule policy
    routing_key: Optional[str] = None

    # For background responses (OpenAI responses API)
    background: bool = False

    # Propagates trace context via Engine.encode/async_encode
    external_trace_header: Optional[Dict] = None
    received_time: Optional[float] = None

    # The number of dimensions the resulting output embeddings should have. It is applicable for Matryoshka Embeddings.
    dimensions: Optional[int] = None

    # The path to the LoRA adaptors
    lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None
    # The uid of LoRA adaptors, should be initialized by tokenizer manager
    lora_id: Optional[Union[List[Optional[str]], Optional[str]]] = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 876-882: Provide supporting module logic / 提供辅助模块逻辑
```python

    # Whether to return pooled hidden states (pre-head transformer output)
    return_pooled_hidden_states: bool = False

    # Pre-computed delimiter indices for multi-item scoring.
    # Batch-level: List[List[int]] (one per request). After __getitem__: List[int].
    multi_item_delimiter_indices: Optional[Union[List[List[int]], List[int]]] = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 884-919: Implement normalize batch and arguments / 实现normalize 批处理 and arguments
```python
    def normalize_batch_and_arguments(self):
        # at least one of text, input_ids, or image should be provided
        if self.text is None and self.input_ids is None and self.image_data is None:
            raise ValueError(
                "At least one of text, input_ids, or image should be provided"
            )

        # text and input_ids cannot be provided at the same time
        if self.text is not None and self.input_ids is not None:
            raise ValueError("text and input_ids cannot be provided at the same time")

        # Derive the batch size
        self.batch_size = 0
        self.is_single = True

        # check the batch size of text
        if self.text is not None:
            if isinstance(self.text, list):
                self.batch_size += len(self.text)
                self.is_single = False
            else:
                self.batch_size += 1

        # check the batch size of input_ids
        if self.input_ids is not None:
            if isinstance(self.input_ids[0], list):
                self.batch_size += len(self.input_ids)
                self.is_single = False
            else:
                self.batch_size += 1

        # Fill in default arguments
        if self.is_single:
            if self.rid is None:
                self.rid = uuid.uuid4().hex
            if self.sampling_params is None:
```
**EN:** This block implements the method `normalize_batch_and_arguments()` on `EmbeddingReqInput`. It focuses on handling the io struct responsibilities represented by `normalize_batch_and_arguments`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `normalize_batch_and_arguments()`。它围绕 `normalize_batch_and_arguments` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 920-937: Continue normalize batch and arguments / 继续说明normalize 批处理 and arguments
```python
                self.sampling_params = {}
            self.sampling_params["max_new_tokens"] = 0
        else:
            if self.rid is None:
                self.rid = [uuid.uuid4().hex for _ in range(self.batch_size)]
            else:
                assert isinstance(self.rid, list), "The rid should be a list."

            if self.sampling_params is None:
                self.sampling_params = [{}] * self.batch_size
            elif isinstance(self.sampling_params, dict):
                self.sampling_params = [self.sampling_params] * self.batch_size
            for i in range(self.batch_size):
                self.sampling_params[i]["max_new_tokens"] = 0

            self._normalize_lora_paths(self.batch_size)

        self._validate_rid_uniqueness()
```
**EN:** This block implements the method `normalize_batch_and_arguments()` on `EmbeddingReqInput`. It focuses on handling the io struct responsibilities represented by `normalize_batch_and_arguments`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `normalize_batch_and_arguments()`。它围绕 `normalize_batch_and_arguments` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 939-950: Implement normalize lora paths / 实现normalize lora paths
```python
    def _normalize_lora_paths(self, num):
        """Normalize LoRA paths for batch processing."""
        if self.lora_path is not None:
            if isinstance(self.lora_path, str):
                self.lora_path = [self.lora_path] * num
            elif isinstance(self.lora_path, list):
                if len(self.lora_path) != num:
                    raise ValueError(
                        f"lora_path list length ({len(self.lora_path)}) must match batch size ({num})"
                    )
            else:
                raise ValueError("lora_path should be a list or a string.")
```
**EN:** This block implements the method `_normalize_lora_paths(num)` on `EmbeddingReqInput`. It focuses on Normalize LoRA paths for batch processing., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `_normalize_lora_paths(num)`。它围绕 `_normalize_lora_paths` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 952-957: Implement contains mm input / 实现contains mm input
```python
    def contains_mm_input(self) -> bool:
        return (
            has_valid_data(self.image_data)
            or has_valid_data(self.video_data)
            or has_valid_data(self.audio_data)
        )
```
**EN:** This block implements the method `contains_mm_input()` on `EmbeddingReqInput`. It focuses on handling the io struct responsibilities represented by `contains_mm_input`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `contains_mm_input()`。它围绕 `contains_mm_input` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 959-967: Implement get positional embed overrides item / 实现get positional embed overrides item
```python
    def _get_positional_embed_overrides_item(
        self, i: int
    ) -> Optional[PositionalEmbeds]:
        """Extract the i-th item from positional_embed_overrides."""
        if self.positional_embed_overrides is None:
            return None
        if isinstance(self.positional_embed_overrides, PositionalEmbeds):
            return self.positional_embed_overrides
        return self.positional_embed_overrides[i]
```
**EN:** This block implements the method `_get_positional_embed_overrides_item(i)` on `EmbeddingReqInput`. It focuses on Extract the i-th item from positional_embed_overrides., so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `_get_positional_embed_overrides_item(i)`。它围绕 `_get_positional_embed_overrides_item` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 969-1004: Implement getitem / 实现getitem
```python
    def __getitem__(self, i):
        # Cache sub-objects so that repeated obj[i] calls return the same instance.
        cache = self.__dict__.setdefault("_sub_obj_cache", {})
        if i in cache:
            return cache[i]

        if self.is_cross_encoder_request:
            sub = EmbeddingReqInput(
                text=[self.text[i]] if self.text is not None else None,
                positional_embed_overrides=self._get_positional_embed_overrides_item(i),
                sampling_params=self.sampling_params[i],
                rid=self.rid[i],
                lora_path=self.lora_path[i] if self.lora_path is not None else None,
                lora_id=self.lora_id[i] if self.lora_id is not None else None,
                is_cross_encoder_request=True,
                http_worker_ipc=self.http_worker_ipc,
                return_pooled_hidden_states=self.return_pooled_hidden_states,
                multi_item_delimiter_indices=(
                    self.multi_item_delimiter_indices[i]
                    if self.multi_item_delimiter_indices is not None
                    else None
                ),
            )
        else:
            sub = EmbeddingReqInput(
                text=self.text[i] if self.text is not None else None,
                input_ids=self.input_ids[i] if self.input_ids is not None else None,
                embed_override_token_id=self.embed_override_token_id,
                embed_overrides=(
                    self.embed_overrides[i]
                    if self.embed_overrides is not None
                    else None
                ),
                positional_embed_overrides=self._get_positional_embed_overrides_item(i),
                image_data=self.image_data[i] if self.image_data is not None else None,
                audio_data=self.audio_data[i] if self.audio_data is not None else None,
```
**EN:** This block implements the method `__getitem__(i)` on `EmbeddingReqInput`. It focuses on handling the io struct responsibilities represented by `__getitem__`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `__getitem__(i)`。它围绕 `__getitem__` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 1005-1022: Continue getitem / 继续说明getitem
```python
                video_data=self.video_data[i] if self.video_data is not None else None,
                sampling_params=self.sampling_params[i],
                rid=self.rid[i],
                lora_path=self.lora_path[i] if self.lora_path is not None else None,
                lora_id=self.lora_id[i] if self.lora_id is not None else None,
                external_trace_header=self.external_trace_header,
                dimensions=self.dimensions,
                http_worker_ipc=self.http_worker_ipc,
                received_time=self.received_time,
                return_pooled_hidden_states=self.return_pooled_hidden_states,
                multi_item_delimiter_indices=(
                    self.multi_item_delimiter_indices[i]
                    if self.multi_item_delimiter_indices is not None
                    else None
                ),
            )
        cache[i] = sub
        return sub
```
**EN:** This block implements the method `__getitem__(i)` on `EmbeddingReqInput`. It focuses on handling the io struct responsibilities represented by `__getitem__`, so the class can advance the io struct workflow in a self-contained way.
**CN:** 该代码块实现 `EmbeddingReqInput` 上的方法 `__getitem__(i)`。它围绕 `__getitem__` 所承担的 I/O struct 相关职责展开，使该类能够独立推进相应流程。

### Lines 1023-1025: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1026-1054: Define class TokenizedEmbeddingReqInput / 定义类 TokenizedEmbeddingReqInput
```python
class TokenizedEmbeddingReqInput(BaseReq):
    # The input text
    input_text: str
    # The input token ids
    input_ids: List[int]
    # The image inputs
    image_inputs: dict
    # The token type ids
    token_type_ids: List[int]
    # Dummy sampling params for compatibility
    sampling_params: SamplingParams
    # Embedding overrides to place at specific token positions.
    positional_embed_overrides: Optional[PositionalEmbeds] = None
    # For DP routing
    routed_dp_rank: Optional[int] = None
    # Priority for the request
    priority: Optional[int] = None
    # The number of dimensions the resulting output embeddings should have. It is applicable for Matryoshka Embeddings.
    dimensions: Optional[int] = None

    # LoRA related
    lora_id: Optional[str] = None  # None means just use the base model
    # Pre-computed delimiter indices for multi-item scoring
    multi_item_delimiter_indices: Optional[List[int]] = None
    # For observability
    time_stats: Optional[Union[APIServerReqTimeStats, DPControllerReqTimeStats]] = None

    # Whether to return pooled hidden states (pre-head transformer output)
    return_pooled_hidden_states: bool = False
```
**EN:** This block declares the class `TokenizedEmbeddingReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `TokenizedEmbeddingReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1055-1057: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1058-1069: Define class BatchTokenizedEmbeddingReqInput / 定义类 BatchTokenizedEmbeddingReqInput
```python
class BatchTokenizedEmbeddingReqInput(BaseBatchReq):
    # The batch of tokenized embedding requests
    batch: List[TokenizedEmbeddingReqInput]

    def __len__(self):
        return len(self.batch)

    def __getitem__(self, i):
        return self.batch[i]

    def __iter__(self):
        return iter(self.batch)
```
**EN:** This block declares the class `BatchTokenizedEmbeddingReqInput`. It centers on coordinating io struct behavior, with methods such as __len__, __getitem__, __iter__.
**CN:** 该代码块声明类 `BatchTokenizedEmbeddingReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 __len__, __getitem__, __iter__ 等方法组织实现。

### Lines 1070-1072: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1073-1141: Define class BatchTokenIDOutput / 定义类 BatchTokenIDOutput
```python
class BatchTokenIDOutput(BaseBatchReq, SpeculativeDecodingMetricsMixin):
    # The finish reason
    finished_reasons: List[BaseFinishReason]
    # For incremental decoding
    decoded_texts: List[str]
    decode_ids: List[int]
    read_offsets: List[int]
    # Only used when `--skip-tokenizer-init` is on
    output_ids: Optional[List[int]]
    # Detokenization configs
    skip_special_tokens: List[bool]
    spaces_between_special_tokens: List[bool]
    no_stop_trim: List[bool]

    # Token counts
    prompt_tokens: List[int]
    reasoning_tokens: List[int]
    completion_tokens: List[int]
    cached_tokens: List[int]

    # Logprobs
    input_token_logprobs_val: List[float]
    input_token_logprobs_idx: List[int]
    output_token_logprobs_val: List[float]
    output_token_logprobs_idx: List[int]
    input_top_logprobs_val: List[List]
    input_top_logprobs_idx: List[List]
    output_top_logprobs_val: List[List]
    output_top_logprobs_idx: List[List]
    input_token_ids_logprobs_val: List[List]
    input_token_ids_logprobs_idx: List[List]
    output_token_ids_logprobs_val: List[List]
    output_token_ids_logprobs_idx: List[List]
    output_token_entropy_val: List[float]

    # Hidden states
    output_hidden_states: List[List[float]]

    # Per-request routed experts (input + output tokens), shape
    # (token, layer, top_k). DetokenizerManager encodes to base64 into
    # BatchStrOutput; on the skip_tokenizer_init path the scheduler sends this
    # straight to TokenizerManager, which encodes on demand.
    routed_experts: List[Optional[torch.Tensor]]

    indexer_topk: List[Optional[torch.Tensor]]

    # The information of placeholder tokens (e.g., image token)
    # idx is the index of the token in the prompt after expansion.
    # val is the length of padded tokens after expansion.
    placeholder_tokens_idx: List[Optional[List[int]]]
    placeholder_tokens_val: List[Optional[List[int]]]

    # Number of times each request was retracted.
    retraction_counts: List[int]

    # The trainer step id. Used to know which step's weights are used for sampling.
    token_steps: List[List[int]] = None

    # Load for DP balance
    load: GetLoadsReqOutput = None
    # Customized info
    customized_info: Optional[Dict[str, List[Any]]] = None
    # Detailed breakdown of cached tokens by source (device/host/storage)
    cached_tokens_details: Optional[List[Optional[Dict[str, Any]]]] = None
    # DP rank of the scheduler that processed each request
    dp_ranks: Optional[List[int]] = None

    # For observability
    time_stats: Optional[List[SchedulerReqTimeStats]] = None
```
**EN:** This block declares the class `BatchTokenIDOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `BatchTokenIDOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1142-1144: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1145-1207: Define class BatchStrOutput / 定义类 BatchStrOutput
```python
class BatchStrOutput(BaseBatchReq, SpeculativeDecodingMetricsMixin):
    # The finish reason
    finished_reasons: List[dict]
    # The output decoded strings
    output_strs: List[str]
    # The token ids
    output_ids: Optional[List[int]]

    # Token counts
    prompt_tokens: List[int]
    completion_tokens: List[int]
    reasoning_tokens: List[int]
    cached_tokens: List[int]

    # Logprobs
    input_token_logprobs_val: List[float]
    input_token_logprobs_idx: List[int]
    output_token_logprobs_val: List[float]
    output_token_logprobs_idx: List[int]
    input_top_logprobs_val: List[List]
    input_top_logprobs_idx: List[List]
    output_top_logprobs_val: List[List]
    output_top_logprobs_idx: List[List]
    input_token_ids_logprobs_val: List[List]
    input_token_ids_logprobs_idx: List[List]
    output_token_ids_logprobs_val: List[List]
    output_token_ids_logprobs_idx: List[List]
    output_token_entropy_val: List[float]

    # Hidden states
    output_hidden_states: List[List[float]]

    # Per-request routed experts, base64-encoded by DetokenizerManager off the
    # tokenizer hot path. Underlying tensor shape is (token, layer, top_k);
    # see BatchTokenIDOutput.routed_experts.
    routed_experts: List[Optional[str]]

    indexer_topk: List[Optional[str]]

    # The information of placeholder tokens (e.g., image token)
    # idx is the index of the token in the prompt after expansion.
    # val is the length of padded tokens after expansion.
    placeholder_tokens_idx: List[Optional[List[int]]]
    placeholder_tokens_val: List[Optional[List[int]]]

    # Number of times each request was retracted.
    retraction_counts: List[int]

    # The trainer step id. Used to know which step's weights are used for sampling.
    token_steps: List[List[int]] = None

    # Load for DP balance
    load: GetLoadsReqOutput = None

    # Customized info
    customized_info: Optional[Dict[str, List[Any]]] = None
    # Detailed breakdown of cached tokens by source (device/host/storage)
    cached_tokens_details: Optional[List[Optional[Dict[str, Any]]]] = None
    # DP rank of the scheduler that processed each request
    dp_ranks: Optional[List[int]] = None

    # For observability
    time_stats: Optional[List[SchedulerReqTimeStats]] = None
```
**EN:** This block declares the class `BatchStrOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `BatchStrOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1208-1210: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1211-1235: Define class BatchEmbeddingOutput / 定义类 BatchEmbeddingOutput
```python
class BatchEmbeddingOutput(BaseBatchReq):
    # The finish reason
    finished_reasons: List[BaseFinishReason]
    # The output embedding
    embeddings: Union[List[List[float]], List[Dict[int, float]]]
    # Token counts
    prompt_tokens: List[int]
    cached_tokens: List[int]
    # Placeholder token info
    placeholder_tokens_idx: List[Optional[List[int]]]
    placeholder_tokens_val: List[Optional[List[int]]]

    # Number of times each request was retracted.
    retraction_counts: List[int]
    # Detailed breakdown of cached tokens by source (device/host/storage)
    cached_tokens_details: Optional[List[Optional[Dict[str, Any]]]] = None

    # For observability
    time_stats: Optional[List[SchedulerReqTimeStats]] = None

    # Optional pooled hidden states (pre-head transformer output).
    # Sent as a single stacked tensor to minimize pickle overhead.
    pooled_hidden_states: Optional[
        Union[List[Optional[torch.Tensor]], torch.Tensor]
    ] = None
```
**EN:** This block declares the class `BatchEmbeddingOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `BatchEmbeddingOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1236-1238: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1239-1240: Define class ClearHiCacheReqInput / 定义类 ClearHiCacheReqInput
```python
class ClearHiCacheReqInput(BaseReq):
    pass
```
**EN:** This block declares the class `ClearHiCacheReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ClearHiCacheReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1241-1243: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1244-1245: Define class ClearHiCacheReqOutput / 定义类 ClearHiCacheReqOutput
```python
class ClearHiCacheReqOutput(BaseReq):
    success: bool
```
**EN:** This block declares the class `ClearHiCacheReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ClearHiCacheReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1246-1248: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1249-1250: Define class FlushCacheReqInput / 定义类 FlushCacheReqInput
```python
class FlushCacheReqInput(BaseReq):
    timeout_s: Optional[float] = None
```
**EN:** This block declares the class `FlushCacheReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `FlushCacheReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1251-1253: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1254-1256: Define class FlushCacheReqOutput / 定义类 FlushCacheReqOutput
```python
class FlushCacheReqOutput(BaseReq):
    success: bool
    message: str = ""
```
**EN:** This block declares the class `FlushCacheReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `FlushCacheReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1257-1259: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1260-1264: Define class AddExternalCorpusReqInput / 定义类 AddExternalCorpusReqInput
```python
class AddExternalCorpusReqInput(BaseReq):
    corpus_id: Optional[str] = None
    file_path: Optional[str] = None
    documents: Optional[List[str]] = None
    token_chunks: Optional[List[List[int]]] = None
```
**EN:** This block declares the class `AddExternalCorpusReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `AddExternalCorpusReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1265-1267: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1268-1272: Define class AddExternalCorpusReqOutput / 定义类 AddExternalCorpusReqOutput
```python
class AddExternalCorpusReqOutput(BaseReq):
    success: bool
    corpus_id: str = ""
    message: str = ""
    loaded_token_count: int = 0
```
**EN:** This block declares the class `AddExternalCorpusReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `AddExternalCorpusReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1273-1275: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1276-1277: Define class RemoveExternalCorpusReqInput / 定义类 RemoveExternalCorpusReqInput
```python
class RemoveExternalCorpusReqInput(BaseReq):
    corpus_id: str
```
**EN:** This block declares the class `RemoveExternalCorpusReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `RemoveExternalCorpusReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1278-1280: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1281-1283: Define class RemoveExternalCorpusReqOutput / 定义类 RemoveExternalCorpusReqOutput
```python
class RemoveExternalCorpusReqOutput(BaseReq):
    success: bool
    message: str = ""
```
**EN:** This block declares the class `RemoveExternalCorpusReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `RemoveExternalCorpusReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1284-1286: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1287-1288: Define class ListExternalCorporaReqInput / 定义类 ListExternalCorporaReqInput
```python
class ListExternalCorporaReqInput(BaseReq):
    pass
```
**EN:** This block declares the class `ListExternalCorporaReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ListExternalCorporaReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1289-1291: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1292-1295: Define class ListExternalCorporaReqOutput / 定义类 ListExternalCorporaReqOutput
```python
class ListExternalCorporaReqOutput(BaseReq):
    success: bool
    corpus_token_counts: Dict[str, int] = field(default_factory=dict)
    message: str = ""
```
**EN:** This block declares the class `ListExternalCorporaReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ListExternalCorporaReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1296-1298: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1299-1330: Define class AttachHiCacheStorageReqInput / 定义类 AttachHiCacheStorageReqInput
```python
class AttachHiCacheStorageReqInput(BaseReq):
    """Dynamically attach (enable) HiCache storage backend at runtime.

    Note: `hicache_storage_backend_extra_config_json` is a JSON string. It may contain both:
    - backend-specific configs (e.g., mooncake master address)
    - prefetch-related knobs (prefetch_threshold, prefetch_timeout_*, hicache_storage_pass_prefix_keys)
    """

    hicache_storage_backend: str
    hicache_storage_backend_extra_config_json: Optional[str] = None
    hicache_storage_prefetch_policy: Optional[str] = None
    hicache_write_policy: Optional[str] = None

    def __post_init__(self):
        if self.hicache_storage_prefetch_policy is None:
            pass
        else:
            allowed = ["best_effort", "wait_complete", "timeout"]
            if self.hicache_storage_prefetch_policy not in allowed:
                raise ValueError(
                    f"Invalid hicache_storage_prefetch_policy: {self.hicache_storage_prefetch_policy!r}. "
                    f"Expected one of {allowed}."
                )

        if self.hicache_write_policy is None:
            return
        allowed = ["write_back", "write_through", "write_through_selective"]
        if self.hicache_write_policy not in allowed:
            raise ValueError(
                f"Invalid hicache_write_policy: {self.hicache_write_policy!r}. "
                f"Expected one of {allowed}."
            )
```
**EN:** This block declares the class `AttachHiCacheStorageReqInput`. It centers on Dynamically attach (enable) HiCache storage backend at runtime., with methods such as __post_init__.
**CN:** 该代码块声明类 `AttachHiCacheStorageReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 __post_init__ 等方法组织实现。

### Lines 1331-1333: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1334-1336: Define class AttachHiCacheStorageReqOutput / 定义类 AttachHiCacheStorageReqOutput
```python
class AttachHiCacheStorageReqOutput(BaseReq):
    success: bool
    message: str = ""
```
**EN:** This block declares the class `AttachHiCacheStorageReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `AttachHiCacheStorageReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1337-1339: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1340-1343: Define class DetachHiCacheStorageReqInput / 定义类 DetachHiCacheStorageReqInput
```python
class DetachHiCacheStorageReqInput(BaseReq):
    """Dynamically detach (disable) HiCache storage backend at runtime."""

    pass
```
**EN:** This block declares the class `DetachHiCacheStorageReqInput`. It centers on Dynamically detach (disable) HiCache storage backend at runtime..
**CN:** 该代码块声明类 `DetachHiCacheStorageReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1344-1346: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1347-1349: Define class DetachHiCacheStorageReqOutput / 定义类 DetachHiCacheStorageReqOutput
```python
class DetachHiCacheStorageReqOutput(BaseReq):
    success: bool
    message: str = ""
```
**EN:** This block declares the class `DetachHiCacheStorageReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `DetachHiCacheStorageReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1350-1352: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1353-1379: Define class PauseGenerationReqInput / 定义类 PauseGenerationReqInput
```python
class PauseGenerationReqInput(BaseReq):
    """
    Note that the PauseGenerationRequests is only supported in SGLang Server.
    abort: Abort and return all requests currently being processed.

    in_place: Pause the scheduler's event_loop from performing inference;
            only non-inference requests (e.g., control commands) will be handled.
            The requests in the engine will be paused and stay in the event_loop,
            then continue generation after continue_generation with the old kv cache.
            Note: In 'inplace' mode, flush_cache will fail if there are any requests
            in the running_batch.

    retract: Pause the scheduler's event loop from performing inference;
            only non-inference requests will be handled, and all currently running
            requests will be retracted back to the waiting_queue.
            Note: The KV cache can be flushed in this mode and will be automatically
            recomputed after continue_generation.
    """

    mode: Literal["abort", "retract", "in_place"] = "abort"

    def __post_init__(self):
        allowed = ["abort", "retract", "in_place"]
        if self.mode not in allowed:
            raise ValueError(
                f"Invalid mode: {self.mode!r}. " f"Expected one of {allowed}."
            )
```
**EN:** This block declares the class `PauseGenerationReqInput`. It centers on Note that the PauseGenerationRequests is only supported in SGLang Server., with methods such as __post_init__.
**CN:** 该代码块声明类 `PauseGenerationReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 __post_init__ 等方法组织实现。

### Lines 1380-1382: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1383-1389: Define class ContinueGenerationReqInput / 定义类 ContinueGenerationReqInput
```python
class ContinueGenerationReqInput(BaseReq):
    # Call torch.cuda.empty_cache() before un-pausing. Returns blocks
    # cached by the PyTorch allocator (left over from transient allocs
    # during post-weight-update processing) back to the driver before
    # inference resumes, with no race against active streams. Set to
    # False to skip the empty_cache call.
    torch_empty_cache: bool = True
```
**EN:** This block declares the class `ContinueGenerationReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ContinueGenerationReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1390-1392: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1393-1396: Define class TokenizerWorkerRegistration / 定义类 TokenizerWorkerRegistration
```python
class TokenizerWorkerRegistration:
    """Sent by each TokenizerWorker on startup to register its IPC name with the router."""

    worker_ipc_name: str
```
**EN:** This block declares the class `TokenizerWorkerRegistration`. It centers on Sent by each TokenizerWorker on startup to register its IPC name with the router..
**CN:** 该代码块声明类 `TokenizerWorkerRegistration`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1397-1399: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1400-1403: Define class PauseContinueBroadcast / 定义类 PauseContinueBroadcast
```python
class PauseContinueBroadcast:
    """Broadcast from router to all workers to set is_pause state."""

    is_pause: bool
```
**EN:** This block declares the class `PauseContinueBroadcast`. It centers on Broadcast from router to all workers to set is_pause state..
**CN:** 该代码块声明类 `PauseContinueBroadcast`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1404-1406: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1407-1429: Define class UpdateWeightFromDiskReqInput / 定义类 UpdateWeightFromDiskReqInput
```python
class UpdateWeightFromDiskReqInput(BaseReq):
    # The model path with the new weights
    model_path: str
    # The format to load the weights
    load_format: Optional[str] = None
    # Whether to abort all requests before updating weights
    abort_all_requests: bool = False
    # Optional: Update weight version along with weights
    weight_version: Optional[str] = None
    # Whether to update weights asynchronously
    is_async: bool = False
    # Whether to call torch.cuda.empty_cache() during flush
    torch_empty_cache: bool = False
    # Whether to keep the scheduler paused after weight update
    keep_pause: bool = False
    # Whether to recapture cuda graph after weight update
    recapture_cuda_graph: bool = False
    # The trainer step id. Used to know which step's weights are used for sampling.
    token_step: int = 0
    # Whether to flush the cache after updating weights
    flush_cache: bool = True
    # Tensor metadata
    manifest: Optional[Dict[str, Any]] = None
```
**EN:** This block declares the class `UpdateWeightFromDiskReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightFromDiskReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1430-1432: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1433-1437: Define class UpdateWeightFromDiskReqOutput / 定义类 UpdateWeightFromDiskReqOutput
```python
class UpdateWeightFromDiskReqOutput(BaseReq):
    success: bool
    message: str
    # Number of paused requests during weight sync.
    num_paused_requests: Optional[int] = 0
```
**EN:** This block declares the class `UpdateWeightFromDiskReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightFromDiskReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1438-1440: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1441-1456: Define class UpdateWeightsFromDistributedReqInput / 定义类 UpdateWeightsFromDistributedReqInput
```python
class UpdateWeightsFromDistributedReqInput(BaseReq):
    names: List[str]
    dtypes: List[str]
    shapes: List[List[int]]
    # The group name
    group_name: str = "weight_update_group"
    # Whether to flush the cache after updating weights
    flush_cache: bool = True
    # Whether to abort all requests before updating weights
    abort_all_requests: bool = False
    # Optional: Update weight version along with weights
    weight_version: Optional[str] = None
    # Optional format specification for loading
    load_format: Optional[str] = None
    # Whether to call torch.cuda.empty_cache() during flush
    torch_empty_cache: bool = False
```
**EN:** This block declares the class `UpdateWeightsFromDistributedReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightsFromDistributedReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1457-1459: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1460-1462: Define class UpdateWeightsFromDistributedReqOutput / 定义类 UpdateWeightsFromDistributedReqOutput
```python
class UpdateWeightsFromDistributedReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `UpdateWeightsFromDistributedReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightsFromDistributedReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1463-1465: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1466-1485: Define class UpdateWeightsFromTensorReqInput / 定义类 UpdateWeightsFromTensorReqInput
```python
class UpdateWeightsFromTensorReqInput(BaseReq):
    """Update model weights from tensor input.

    - Tensors are serialized for transmission
    - Data is structured in JSON for easy transmission over HTTP
    """

    serialized_named_tensors: List[Union[str, bytes]]
    # Optional format specification for loading
    load_format: Optional[str] = None
    # Whether to flush the cache after updating weights
    flush_cache: bool = True
    # Whether to abort all requests before updating weights
    abort_all_requests: bool = False
    # Optional: Update weight version along with weights
    weight_version: Optional[str] = None
    # Optional: Determine whether to disable updating the draft model
    disable_draft_model: Optional[bool] = None
    # Whether to call torch.cuda.empty_cache() during flush
    torch_empty_cache: bool = False
```
**EN:** This block declares the class `UpdateWeightsFromTensorReqInput`. It centers on Update model weights from tensor input..
**CN:** 该代码块声明类 `UpdateWeightsFromTensorReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1486-1488: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1489-1491: Define class UpdateWeightsFromTensorReqOutput / 定义类 UpdateWeightsFromTensorReqOutput
```python
class UpdateWeightsFromTensorReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `UpdateWeightsFromTensorReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightsFromTensorReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1492-1494: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1495-1507: Define class InitWeightsSendGroupForRemoteInstanceReqInput / 定义类 InitWeightsSendGroupForRemoteInstanceReqInput
```python
class InitWeightsSendGroupForRemoteInstanceReqInput(BaseReq):
    # The master address
    master_address: str
    # The ports for each rank's communication group
    ports: str
    # The rank in the communication group
    group_rank: int
    # The world size
    world_size: int
    # The group name
    group_name: str = "weight_send_group"
    # The backend
    backend: str = "nccl"
```
**EN:** This block declares the class `InitWeightsSendGroupForRemoteInstanceReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `InitWeightsSendGroupForRemoteInstanceReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1508-1512: Provide supporting module logic / 提供辅助模块逻辑
```python


# Now UpdateWeightsFromIPCReqInput and UpdateWeightsFromIPCReqOutput
# are only used by Checkpoint Engine (https://github.com/MoonshotAI/checkpoint-engine)
@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1513-1521: Define class UpdateWeightsFromIPCReqInput / 定义类 UpdateWeightsFromIPCReqInput
```python
class UpdateWeightsFromIPCReqInput(BaseReq):
    # ZMQ socket paths for each device UUID
    zmq_handles: Dict[str, str]
    # Whether to flush cache after weight update
    flush_cache: bool = True
    # Optional: Update weight version along with weights
    weight_version: Optional[str] = None
    # Whether to call torch.cuda.empty_cache() during flush
    torch_empty_cache: bool = False
```
**EN:** This block declares the class `UpdateWeightsFromIPCReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightsFromIPCReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1522-1524: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1525-1527: Define class UpdateWeightsFromIPCReqOutput / 定义类 UpdateWeightsFromIPCReqOutput
```python
class UpdateWeightsFromIPCReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `UpdateWeightsFromIPCReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightsFromIPCReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1528-1530: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1531-1533: Define class InitWeightsSendGroupForRemoteInstanceReqOutput / 定义类 InitWeightsSendGroupForRemoteInstanceReqOutput
```python
class InitWeightsSendGroupForRemoteInstanceReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `InitWeightsSendGroupForRemoteInstanceReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `InitWeightsSendGroupForRemoteInstanceReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1534-1536: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1537-1543: Define class SendWeightsToRemoteInstanceReqInput / 定义类 SendWeightsToRemoteInstanceReqInput
```python
class SendWeightsToRemoteInstanceReqInput(BaseReq):
    # The master address
    master_address: str
    # The ports for each rank's communication group
    ports: str
    # The group name
    group_name: str = "weight_send_group"
```
**EN:** This block declares the class `SendWeightsToRemoteInstanceReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SendWeightsToRemoteInstanceReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1544-1546: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1547-1549: Define class SendWeightsToRemoteInstanceReqOutput / 定义类 SendWeightsToRemoteInstanceReqOutput
```python
class SendWeightsToRemoteInstanceReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `SendWeightsToRemoteInstanceReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SendWeightsToRemoteInstanceReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1550-1552: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1553-1554: Define class UpdateExpertBackupReq / 定义类 UpdateExpertBackupReq
```python
class UpdateExpertBackupReq(BaseReq):
    pass
```
**EN:** This block declares the class `UpdateExpertBackupReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateExpertBackupReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1555-1557: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1558-1562: Define class BackupDramReq / 定义类 BackupDramReq
```python
class BackupDramReq(BaseReq):
    rank: int
    weight_pointer_map: Dict[str, Any]
    session_id: str
    buffer_size: int
```
**EN:** This block declares the class `BackupDramReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `BackupDramReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1563-1565: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1566-1578: Define class InitWeightsUpdateGroupReqInput / 定义类 InitWeightsUpdateGroupReqInput
```python
class InitWeightsUpdateGroupReqInput(BaseReq):
    # The master address
    master_address: str
    # The master port
    master_port: int
    # The rank offset
    rank_offset: int
    # The world size
    world_size: int
    # The group name
    group_name: str = "weight_update_group"
    # The backend
    backend: str = "nccl"
```
**EN:** This block declares the class `InitWeightsUpdateGroupReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `InitWeightsUpdateGroupReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1579-1581: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1582-1584: Define class InitWeightsUpdateGroupReqOutput / 定义类 InitWeightsUpdateGroupReqOutput
```python
class InitWeightsUpdateGroupReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `InitWeightsUpdateGroupReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `InitWeightsUpdateGroupReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1585-1587: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1588-1589: Define class DestroyWeightsUpdateGroupReqInput / 定义类 DestroyWeightsUpdateGroupReqInput
```python
class DestroyWeightsUpdateGroupReqInput(BaseReq):
    group_name: str = "weight_update_group"
```
**EN:** This block declares the class `DestroyWeightsUpdateGroupReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `DestroyWeightsUpdateGroupReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1590-1592: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1593-1595: Define class DestroyWeightsUpdateGroupReqOutput / 定义类 DestroyWeightsUpdateGroupReqOutput
```python
class DestroyWeightsUpdateGroupReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `DestroyWeightsUpdateGroupReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `DestroyWeightsUpdateGroupReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1596-1598: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1599-1603: Define class UpdateWeightVersionReqInput / 定义类 UpdateWeightVersionReqInput
```python
class UpdateWeightVersionReqInput(BaseReq):
    # The new weight version
    new_version: str
    # Whether to abort all running requests before updating
    abort_all_requests: bool = True
```
**EN:** This block declares the class `UpdateWeightVersionReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `UpdateWeightVersionReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1604-1606: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1607-1609: Define class GetWeightsByNameReqInput / 定义类 GetWeightsByNameReqInput
```python
class GetWeightsByNameReqInput(BaseReq):
    name: str
    truncate_size: int = 100
```
**EN:** This block declares the class `GetWeightsByNameReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `GetWeightsByNameReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1610-1612: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1613-1614: Define class GetWeightsByNameReqOutput / 定义类 GetWeightsByNameReqOutput
```python
class GetWeightsByNameReqOutput(BaseReq):
    parameter: list
```
**EN:** This block declares the class `GetWeightsByNameReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `GetWeightsByNameReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1615-1617: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1618-1621: Define class ReleaseMemoryOccupationReqInput / 定义类 ReleaseMemoryOccupationReqInput
```python
class ReleaseMemoryOccupationReqInput(BaseReq):
    # Optional tags to identify the memory region, which is primarily used for RL
    # Currently we only support `weights` and `kv_cache`
    tags: Optional[List[str]] = None
```
**EN:** This block declares the class `ReleaseMemoryOccupationReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ReleaseMemoryOccupationReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1622-1624: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1625-1626: Define class ReleaseMemoryOccupationReqOutput / 定义类 ReleaseMemoryOccupationReqOutput
```python
class ReleaseMemoryOccupationReqOutput(BaseReq):
    pass
```
**EN:** This block declares the class `ReleaseMemoryOccupationReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ReleaseMemoryOccupationReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1627-1629: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1630-1633: Define class ResumeMemoryOccupationReqInput / 定义类 ResumeMemoryOccupationReqInput
```python
class ResumeMemoryOccupationReqInput(BaseReq):
    # Optional tags to identify the memory region, which is primarily used for RL
    # Currently we only support `weights` and `kv_cache`
    tags: Optional[List[str]] = None
```
**EN:** This block declares the class `ResumeMemoryOccupationReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ResumeMemoryOccupationReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1634-1636: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1637-1638: Define class ResumeMemoryOccupationReqOutput / 定义类 ResumeMemoryOccupationReqOutput
```python
class ResumeMemoryOccupationReqOutput(BaseReq):
    pass
```
**EN:** This block declares the class `ResumeMemoryOccupationReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ResumeMemoryOccupationReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1639-1641: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1642-1643: Define class CheckWeightsReqInput / 定义类 CheckWeightsReqInput
```python
class CheckWeightsReqInput(BaseReq):
    action: str
```
**EN:** This block declares the class `CheckWeightsReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `CheckWeightsReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1644-1646: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1647-1650: Define class CheckWeightsReqOutput / 定义类 CheckWeightsReqOutput
```python
class CheckWeightsReqOutput(BaseReq):
    success: bool
    message: str
    payload: Optional[Dict] = None
```
**EN:** This block declares the class `CheckWeightsReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `CheckWeightsReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1651-1653: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1654-1655: Define class SlowDownReqInput / 定义类 SlowDownReqInput
```python
class SlowDownReqInput(BaseReq):
    forward_sleep_time: Optional[float]
```
**EN:** This block declares the class `SlowDownReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SlowDownReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1656-1658: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1659-1660: Define class SlowDownReqOutput / 定义类 SlowDownReqOutput
```python
class SlowDownReqOutput(BaseReq):
    pass
```
**EN:** This block declares the class `SlowDownReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SlowDownReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1661-1663: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1664-1674: Define class AbortReq / 定义类 AbortReq
```python
class AbortReq(BaseReq):
    # Whether to abort all requests
    abort_all: bool = False
    # The finished reason data
    finished_reason: Optional[Dict[str, Any]] = None
    abort_message: Optional[str] = None

    def __post_init__(self):
        # FIXME: This is a hack to keep the same with the old code
        if self.rid is None:
            self.rid = ""
```
**EN:** This block declares the class `AbortReq`. It centers on coordinating io struct behavior, with methods such as __post_init__.
**CN:** 该代码块声明类 `AbortReq`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 __post_init__ 等方法组织实现。

### Lines 1675-1677: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1678-1679: Define class ActiveRanksOutput / 定义类 ActiveRanksOutput
```python
class ActiveRanksOutput(BaseReq):
    status: List[bool]
```
**EN:** This block declares the class `ActiveRanksOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ActiveRanksOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1680-1682: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1683-1684: Define class GetInternalStateReq / 定义类 GetInternalStateReq
```python
class GetInternalStateReq(BaseReq):
    pass
```
**EN:** This block declares the class `GetInternalStateReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `GetInternalStateReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1685-1687: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1688-1689: Define class GetInternalStateReqOutput / 定义类 GetInternalStateReqOutput
```python
class GetInternalStateReqOutput(BaseReq):
    internal_state: Dict[Any, Any]
```
**EN:** This block declares the class `GetInternalStateReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `GetInternalStateReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1690-1692: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1693-1694: Define class SetInternalStateReq / 定义类 SetInternalStateReq
```python
class SetInternalStateReq(BaseReq):
    server_args: Dict[str, Any]
```
**EN:** This block declares the class `SetInternalStateReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SetInternalStateReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1695-1697: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1698-1700: Define class SetInternalStateReqOutput / 定义类 SetInternalStateReqOutput
```python
class SetInternalStateReqOutput(BaseReq):
    updated: bool
    server_args: Dict[str, Any]
```
**EN:** This block declares the class `SetInternalStateReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SetInternalStateReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1701-1703: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1704-1726: Define class ProfileReqInput / 定义类 ProfileReqInput
```python
class ProfileReqInput(BaseReq):
    # The output directory
    output_dir: Optional[str] = None
    # Specify the steps to start the profiling
    start_step: Optional[int] = None
    # If set, it profile as many as this number of steps.
    # If it is set, profiling is automatically stopped after this step, and
    # the caller doesn't need to run stop_profile.
    num_steps: Optional[int] = None
    # The activities to record. The choices are ["CPU", "GPU", "MEM", "RPD"]
    activities: Optional[List[str]] = None
    # Whether profile by stages (e.g., prefill and decode) separately
    profile_by_stage: bool = False
    # Whether to record source information (file and line number) for the ops.
    with_stack: Optional[bool] = None
    # Whether to save information about operator’s input shapes.
    record_shapes: Optional[bool] = None
    # Merge profiles from all ranks into a single trace
    merge_profiles: bool = False
    # The prefix of the profile filenames
    profile_prefix: Optional[str] = None
    # Only profile these stages and ignore others
    profile_stages: Optional[List[str]] = None
```
**EN:** This block declares the class `ProfileReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ProfileReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1729-1731: Define class ProfileReqType / 定义类 ProfileReqType
```python
class ProfileReqType(Enum):
    START_PROFILE = 1
    STOP_PROFILE = 2
```
**EN:** This block declares the class `ProfileReqType`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ProfileReqType`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1732-1734: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1735-1747: Define class ProfileReq / 定义类 ProfileReq
```python
class ProfileReq(BaseReq):
    type: ProfileReqType
    output_dir: Optional[str] = None
    start_step: Optional[int] = None
    num_steps: Optional[int] = None
    activities: Optional[List[str]] = None
    profile_by_stage: bool = False
    with_stack: Optional[bool] = None
    record_shapes: Optional[bool] = None
    profile_id: Optional[str] = None
    merge_profiles: bool = False
    profile_prefix: Optional[str] = None
    profile_stages: Optional[List[str]] = None
```
**EN:** This block declares the class `ProfileReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ProfileReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1748-1750: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1751-1753: Define class ProfileReqOutput / 定义类 ProfileReqOutput
```python
class ProfileReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `ProfileReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ProfileReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1754-1756: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1757-1758: Define class FreezeGCReq / 定义类 FreezeGCReq
```python
class FreezeGCReq(BaseReq):
    pass
```
**EN:** This block declares the class `FreezeGCReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `FreezeGCReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1759-1761: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1762-1769: Define class ConfigureLoggingReq / 定义类 ConfigureLoggingReq
```python
class ConfigureLoggingReq(BaseReq):
    log_requests: Optional[bool] = None
    log_requests_level: Optional[int] = None
    log_requests_format: Optional[str] = None
    dump_requests_folder: Optional[str] = None
    dump_requests_threshold: Optional[int] = None
    crash_dump_folder: Optional[str] = None
    dump_requests_exclude_meta_keys: Optional[List[str]] = None
```
**EN:** This block declares the class `ConfigureLoggingReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ConfigureLoggingReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1770-1772: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1773-1777: Define class OpenSessionReqInput / 定义类 OpenSessionReqInput
```python
class OpenSessionReqInput(BaseReq):
    capacity_of_str_len: int
    session_id: Optional[str] = None
    streaming: Optional[bool] = None
    timeout: Optional[float] = None
```
**EN:** This block declares the class `OpenSessionReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `OpenSessionReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1778-1780: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1781-1782: Define class CloseSessionReqInput / 定义类 CloseSessionReqInput
```python
class CloseSessionReqInput(BaseReq):
    session_id: str
```
**EN:** This block declares the class `CloseSessionReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `CloseSessionReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1783-1785: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1786-1788: Define class OpenSessionReqOutput / 定义类 OpenSessionReqOutput
```python
class OpenSessionReqOutput(BaseReq):
    session_id: Optional[str]
    success: bool
```
**EN:** This block declares the class `OpenSessionReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `OpenSessionReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1789-1791: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1792-1793: Define class HealthCheckOutput / 定义类 HealthCheckOutput
```python
class HealthCheckOutput(BaseReq):
    pass
```
**EN:** This block declares the class `HealthCheckOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `HealthCheckOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1796-1799: Define class ExpertDistributionReqType / 定义类 ExpertDistributionReqType
```python
class ExpertDistributionReqType(Enum):
    START_RECORD = 1
    STOP_RECORD = 2
    DUMP_RECORD = 3
```
**EN:** This block declares the class `ExpertDistributionReqType`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ExpertDistributionReqType`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1800-1802: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1803-1804: Define class ExpertDistributionReq / 定义类 ExpertDistributionReq
```python
class ExpertDistributionReq(BaseReq):
    action: ExpertDistributionReqType
```
**EN:** This block declares the class `ExpertDistributionReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ExpertDistributionReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1805-1807: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1808-1809: Define class ExpertDistributionReqOutput / 定义类 ExpertDistributionReqOutput
```python
class ExpertDistributionReqOutput(BaseReq):
    pass
```
**EN:** This block declares the class `ExpertDistributionReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ExpertDistributionReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1810-1812: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1813-1816: Define class Function / 定义类 Function
```python
class Function:
    description: Optional[str] = None
    name: Optional[str] = None
    parameters: Optional[object] = None
```
**EN:** This block declares the class `Function`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `Function`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1817-1819: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1820-1822: Define class Tool / 定义类 Tool
```python
class Tool:
    function: Function
    type: Optional[str] = "function"
```
**EN:** This block declares the class `Tool`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `Tool`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1823-1825: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1826-1833: Define class ParseFunctionCallReq / 定义类 ParseFunctionCallReq
```python
class ParseFunctionCallReq(BaseReq):
    text: str  # The text to parse.
    tools: List[Tool] = field(
        default_factory=list
    )  # A list of available function tools (name, parameters, etc.).
    tool_call_parser: Optional[str] = (
        None  # Specify the parser type, e.g. 'llama3', 'qwen25', or 'mistral'. If not specified, tries all.
    )
```
**EN:** This block declares the class `ParseFunctionCallReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `ParseFunctionCallReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1834-1836: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1837-1839: Define class SeparateReasoningReqInput / 定义类 SeparateReasoningReqInput
```python
class SeparateReasoningReqInput(BaseReq):
    text: str  # The text to parse.
    reasoning_parser: str  # Specify the parser type, e.g., "deepseek-r1".
```
**EN:** This block declares the class `SeparateReasoningReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SeparateReasoningReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1840-1842: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1843-1845: Define class VertexGenerateReqInput / 定义类 VertexGenerateReqInput
```python
class VertexGenerateReqInput(BaseReq):
    instances: List[dict]
    parameters: Optional[dict] = None
```
**EN:** This block declares the class `VertexGenerateReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `VertexGenerateReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1846-1848: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1849-1851: Define class RpcReqInput / 定义类 RpcReqInput
```python
class RpcReqInput(BaseReq):
    method: str
    parameters: Optional[Dict] = None
```
**EN:** This block declares the class `RpcReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `RpcReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1852-1854: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1855-1857: Define class RpcReqOutput / 定义类 RpcReqOutput
```python
class RpcReqOutput(BaseReq):
    success: bool
    message: str
```
**EN:** This block declares the class `RpcReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `RpcReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1858-1860: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1861-1877: Define class LoadLoRAAdapterReqInput / 定义类 LoadLoRAAdapterReqInput
```python
class LoadLoRAAdapterReqInput(BaseReq):
    # The name of the lora module to newly loaded.
    lora_name: str
    # The path of loading.
    lora_path: str
    # Whether to pin the LoRA adapter in memory.
    pinned: bool = False
    # The unique identifier for the LoRA adapter, which automatically generated in the `TokenizerManager`.
    lora_id: Optional[str] = None

    def to_ref(self) -> LoRARef:
        return LoRARef(
            lora_id=self.lora_id,
            lora_name=self.lora_name,
            lora_path=self.lora_path,
            pinned=self.pinned,
        )
```
**EN:** This block declares the class `LoadLoRAAdapterReqInput`. It centers on coordinating io struct behavior, with methods such as to_ref.
**CN:** 该代码块声明类 `LoadLoRAAdapterReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 to_ref 等方法组织实现。

### Lines 1878-1880: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1881-1891: Define class UnloadLoRAAdapterReqInput / 定义类 UnloadLoRAAdapterReqInput
```python
class UnloadLoRAAdapterReqInput(BaseReq):
    # The name of lora module to unload.
    lora_name: str
    # The unique identifier for the LoRA adapter, which automatically generated in the `TokenizerManager`.
    lora_id: Optional[str] = None

    def to_ref(self) -> LoRARef:
        return LoRARef(
            lora_id=self.lora_id,
            lora_name=self.lora_name,
        )
```
**EN:** This block declares the class `UnloadLoRAAdapterReqInput`. It centers on coordinating io struct behavior, with methods such as to_ref.
**CN:** 该代码块声明类 `UnloadLoRAAdapterReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 to_ref 等方法组织实现。

### Lines 1892-1894: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1895-1910: Define class LoadLoRAAdapterFromTensorsReqInput / 定义类 LoadLoRAAdapterFromTensorsReqInput
```python
class LoadLoRAAdapterFromTensorsReqInput(BaseReq):
    lora_name: str
    config_dict: Dict[str, Any]
    serialized_tensors: str
    pinned: bool = False
    added_tokens_config: Optional[Dict[str, Any]] = None
    lora_id: Optional[str] = None
    load_format: Optional[str] = None

    def to_ref(self) -> LoRARef:
        return LoRARef(
            lora_id=self.lora_id,
            lora_name=self.lora_name,
            lora_path="__tensor__",
            pinned=self.pinned,
        )
```
**EN:** This block declares the class `LoadLoRAAdapterFromTensorsReqInput`. It centers on coordinating io struct behavior, with methods such as to_ref.
**CN:** 该代码块声明类 `LoadLoRAAdapterFromTensorsReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 to_ref 等方法组织实现。

### Lines 1911-1913: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1914-1917: Define class LoRAUpdateOutput / 定义类 LoRAUpdateOutput
```python
class LoRAUpdateOutput(BaseReq):
    success: bool
    error_message: Optional[str] = None
    loaded_adapters: Optional[Dict[str, LoRARef]] = None
```
**EN:** This block declares the class `LoRAUpdateOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `LoRAUpdateOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1920-1922: Provide supporting module logic / 提供辅助模块逻辑
```python
LoadLoRAAdapterReqOutput = UnloadLoRAAdapterReqOutput = (
    LoadLoRAAdapterFromTensorsReqOutput
) = LoRAUpdateOutput
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1925-1927: Define class BlockReqType / 定义类 BlockReqType
```python
class BlockReqType(Enum):
    BLOCK = 1
    UNBLOCK = 2
```
**EN:** This block declares the class `BlockReqType`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `BlockReqType`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1928-1930: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1931-1932: Define class BlockReqInput / 定义类 BlockReqInput
```python
class BlockReqInput(BaseReq):
    type: BlockReqType
```
**EN:** This block declares the class `BlockReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `BlockReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1933-1935: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1936-1946: Define class MemoryMetrics / 定义类 MemoryMetrics
```python
class MemoryMetrics:
    """Memory breakdown metrics."""

    weight_gb: float = field(
        metadata={"metric": ("gauge", "Model weight memory in GB")}
    )
    kv_cache_gb: float = field(metadata={"metric": ("gauge", "KV cache memory in GB")})
    graph_gb: float = field(metadata={"metric": ("gauge", "CUDA graph memory in GB")})
    token_capacity: int = field(
        metadata={"metric": ("gauge", "Max tokens in KV cache")}
    )
```
**EN:** This block declares the class `MemoryMetrics`. It centers on Memory breakdown metrics..
**CN:** 该代码块声明类 `MemoryMetrics`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1947-1949: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1950-1963: Define class SpeculativeMetrics / 定义类 SpeculativeMetrics
```python
class SpeculativeMetrics:
    """Speculative decoding metrics."""

    accept_length: float = field(
        metadata={
            "metric": (
                "gauge",
                "Mean acceptance length (accepted drafts + bonus token per forward)",
            )
        }
    )
    accept_rate: float = field(
        metadata={"metric": ("gauge", "Speculative acceptance rate")}
    )
```
**EN:** This block declares the class `SpeculativeMetrics`. It centers on Speculative decoding metrics..
**CN:** 该代码块声明类 `SpeculativeMetrics`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1964-1966: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1967-1974: Define class LoRAMetrics / 定义类 LoRAMetrics
```python
class LoRAMetrics:
    """LoRA adapter pool metrics."""

    slots_used: int = field(metadata={"metric": ("gauge", "LoRA adapter slots in use")})
    slots_total: int = field(metadata={"metric": ("gauge", "Total LoRA adapter slots")})
    utilization: float = field(
        metadata={"metric": ("gauge", "LoRA pool utilization ratio")}
    )
```
**EN:** This block declares the class `LoRAMetrics`. It centers on LoRA adapter pool metrics..
**CN:** 该代码块声明类 `LoRAMetrics`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 1975-1977: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1978-2002: Define class DisaggregationMetrics / 定义类 DisaggregationMetrics
```python
class DisaggregationMetrics:
    """PD disaggregation metrics."""

    mode: str  # "prefill", "decode", or "null" - not a metric
    prefill_bootstrap_queue_reqs: int = field(
        default=0, metadata={"metric": ("gauge", "Prefill bootstrap queue requests")}
    )
    prefill_inflight_queue_reqs: int = field(
        default=0, metadata={"metric": ("gauge", "Prefill inflight queue requests")}
    )
    decode_prealloc_queue_reqs: int = field(
        default=0, metadata={"metric": ("gauge", "Decode prealloc queue requests")}
    )
    decode_transfer_queue_reqs: int = field(
        default=0, metadata={"metric": ("gauge", "Decode transfer queue requests")}
    )
    decode_retracted_queue_reqs: int = field(
        default=0, metadata={"metric": ("gauge", "Decode retracted queue requests")}
    )
    kv_transfer_speed_gb_s: float = field(
        default=0.0, metadata={"metric": ("gauge", "KV transfer speed in GB/s")}
    )
    kv_transfer_latency_ms: float = field(
        default=0.0, metadata={"metric": ("gauge", "KV transfer latency in ms")}
    )
```
**EN:** This block declares the class `DisaggregationMetrics`. It centers on PD disaggregation metrics..
**CN:** 该代码块声明类 `DisaggregationMetrics`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2003-2005: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2006-2016: Define class QueueMetrics / 定义类 QueueMetrics
```python
class QueueMetrics:
    """Detailed queue breakdown."""

    waiting: int = field(metadata={"metric": ("gauge", "Main waiting queue size")})
    grammar: int = field(
        metadata={"metric": ("gauge", "Grammar compilation queue size")}
    )
    paused: int = field(
        metadata={"metric": ("gauge", "Requests paused by weight sync")}
    )
    retracted: int = field(metadata={"metric": ("gauge", "Retracted requests count")})
```
**EN:** This block declares the class `QueueMetrics`. It centers on Detailed queue breakdown..
**CN:** 该代码块声明类 `QueueMetrics`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2017-2019: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2020-2038: Define class GetLoadsReqInput / 定义类 GetLoadsReqInput
```python
class GetLoadsReqInput(BaseReq):
    """Request for /v1/loads endpoint."""

    VALID_SECTIONS = frozenset(
        {"core", "memory", "spec", "lora", "disagg", "queues", "all"}
    )

    include: List[str] = field(default_factory=lambda: ["all"])
    dp_rank: Optional[int] = None

    def __post_init__(self):
        """Validate include sections."""
        if self.include:
            invalid = set(self.include) - self.VALID_SECTIONS
            if invalid:
                raise ValueError(
                    f"Invalid include sections: {invalid}. "
                    f"Valid options: {sorted(self.VALID_SECTIONS)}"
                )
```
**EN:** This block declares the class `GetLoadsReqInput`. It centers on Request for /v1/loads endpoint., with methods such as __post_init__.
**CN:** 该代码块声明类 `GetLoadsReqInput`。它负责承载与 I/O struct 相关的核心状态与行为，并通过 __post_init__ 等方法组织实现。

### Lines 2039-2041: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2042-2085: Define class GetLoadsReqOutput / 定义类 GetLoadsReqOutput
```python
class GetLoadsReqOutput(BaseReq):
    """Per-DP-rank load metrics for /v1/loads endpoint."""

    dp_rank: int
    timestamp: float

    num_running_reqs: int = field(
        metadata={"metric": ("gauge", "Number of running requests")}
    )
    num_waiting_reqs: int = field(
        metadata={"metric": ("gauge", "Number of waiting requests")}
    )
    num_used_tokens: int = field(
        metadata={"metric": ("gauge", "Number of tokens in use")}
    )
    # num_used_tokens + pending prefill tokens (waiting-queue seqlen, incl.
    # disagg bootstrap/prealloc/transfer queues). Used for DP balance.
    num_total_tokens: int = field(
        metadata={"metric": ("gauge", "Used tokens plus pending prefill tokens")}
    )
    max_total_num_tokens: int = field(
        metadata={"metric": ("gauge", "Maximum token capacity")}
    )
    # FIXME: token_usage is actually max usage across all pools (KV, SWA, mamba),
    # not just KV token usage. Rename requires API deprecation.
    token_usage: float = field(metadata={"metric": ("gauge", "Token pool usage ratio")})
    gen_throughput: float = field(
        metadata={"metric": ("gauge", "Generation throughput tokens/sec")}
    )
    cache_hit_rate: float = field(
        metadata={"metric": ("gauge", "Prefix cache hit rate")}
    )
    utilization: float = field(
        metadata={"metric": ("gauge", "Overall utilization ratio")}
    )
    max_running_requests: int = field(
        metadata={"metric": ("gauge", "Maximum running requests capacity")}
    )

    memory: Optional[MemoryMetrics] = None
    speculative: Optional[SpeculativeMetrics] = None
    lora: Optional[LoRAMetrics] = None
    disaggregation: Optional[DisaggregationMetrics] = None
    queues: Optional[QueueMetrics] = None
```
**EN:** This block declares the class `GetLoadsReqOutput`. It centers on Per-DP-rank load metrics for /v1/loads endpoint..
**CN:** 该代码块声明类 `GetLoadsReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2086-2088: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2089-2090: Define class WatchLoadUpdateReq / 定义类 WatchLoadUpdateReq
```python
class WatchLoadUpdateReq(BaseReq):
    loads: List[GetLoadsReqOutput]
```
**EN:** This block declares the class `WatchLoadUpdateReq`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `WatchLoadUpdateReq`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2091-2093: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2094-2095: Define class SetInjectDumpMetadataReqInput / 定义类 SetInjectDumpMetadataReqInput
```python
class SetInjectDumpMetadataReqInput(BaseReq):
    dump_metadata: Dict[str, Any]
```
**EN:** This block declares the class `SetInjectDumpMetadataReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SetInjectDumpMetadataReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2096-2098: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2099-2100: Define class SetInjectDumpMetadataReqOutput / 定义类 SetInjectDumpMetadataReqOutput
```python
class SetInjectDumpMetadataReqOutput(BaseReq):
    success: bool
```
**EN:** This block declares the class `SetInjectDumpMetadataReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `SetInjectDumpMetadataReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2101-2103: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2104-2105: Define class LazyDumpTensorsReqInput / 定义类 LazyDumpTensorsReqInput
```python
class LazyDumpTensorsReqInput(BaseReq):
    pass
```
**EN:** This block declares the class `LazyDumpTensorsReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `LazyDumpTensorsReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2106-2108: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2109-2110: Define class LazyDumpTensorsReqOutput / 定义类 LazyDumpTensorsReqOutput
```python
class LazyDumpTensorsReqOutput(BaseReq):
    success: bool
```
**EN:** This block declares the class `LazyDumpTensorsReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `LazyDumpTensorsReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2111-2113: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2114-2116: Define class DumperControlReqInput / 定义类 DumperControlReqInput
```python
class DumperControlReqInput(BaseReq):
    method: str
    body: Dict[str, Any]
```
**EN:** This block declares the class `DumperControlReqInput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `DumperControlReqInput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2117-2119: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 2120-2123: Define class DumperControlReqOutput / 定义类 DumperControlReqOutput
```python
class DumperControlReqOutput(BaseReq):
    success: bool
    response: List[Dict[str, Any]]
    error: str = ""
```
**EN:** This block declares the class `DumperControlReqOutput`. It centers on coordinating io struct behavior.
**CN:** 该代码块声明类 `DumperControlReqOutput`。它负责承载与 I/O struct 相关的核心状态与行为。

### Lines 2126-2146: Implement check all req types / 实现检查 all req types
```python
def _check_all_req_types():
    """A helper function to check all request types are defined in this file."""
    import inspect
    import sys

    all_classes = inspect.getmembers(sys.modules[__name__], inspect.isclass)
    for class_type in all_classes:
        # check its name
        name = class_type[0]
        is_io_struct = (
            name.endswith("Req") or name.endswith("Input") or name.endswith("Output")
        )
        is_base_req = issubclass(class_type[1], BaseReq) or issubclass(
            class_type[1], BaseBatchReq
        )
        if is_io_struct and not is_base_req:
            raise ValueError(f"{name} is not a subclass of BaseReq or BaseBatchReq.")
        if is_base_req and not is_io_struct:
            raise ValueError(
                f"{name} is a subclass of BaseReq but not follow the naming convention."
            )
```
**EN:** This block implements the function `_check_all_req_types()`. It focuses on A helper function to check all request types are defined in this file., providing reusable behavior for the io struct pipeline.
**CN:** 该代码块实现函数 `_check_all_req_types()`。它围绕 `_check_all_req_types` 所承担的 I/O struct 相关职责展开，为对应处理链路提供可复用能力。

### Lines 2149-2149: Provide supporting module logic / 提供辅助模块逻辑
```python
_check_all_req_types()
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

## Key Concepts / 关键概念
- **Core types / 核心类型**: BaseReq, BaseBatchReq, SpeculativeDecodingMetricsMixin, SessionParams, GenerateReqInput, TokenizedGenerateReqInput, ...
- **Main callables / 主要可调用对象**: _check_all_req_types
- **Domain focus / 领域焦点**: io struct / I/O struct
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: collections, copy, dataclasses, enum, inspect, sys, typing, uuid, warnings
- **Third-party / 第三方库**: PIL.Image, __future__, abc, torch
- **Local Modules / 本地模块**: sglang.srt.lora.lora_registry, sglang.srt.managers.embed_types, sglang.srt.managers.schedule_batch, sglang.srt.multimodal.mm_utils, sglang.srt.observability.req_time_stats, sglang.srt.sampling.sampling_params, sglang.srt.utils

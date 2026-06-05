# throughput.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/throughput.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Benchmark offline inference throughput. / 该文件的核心目的为：Benchmark offline inference throughput.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Benchmark offline inference throughput."""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 5-44)
```python
import argparse

import json

import os

import random

import time

import warnings

from typing import Any

import torch

import uvloop

from tqdm import tqdm

from transformers import AutoModelForCausalLM, PreTrainedTokenizerBase

from vllm.benchmarks.datasets import (
    AIMODataset,
    ASRDataset,
    BurstGPTDataset,
    ConversationDataset,
    InstructCoderDataset,
    MultiModalConversationDataset,
    PrefixRepetitionRandomDataset,
    RandomDataset,
    RandomDatasetForReranking,
    RandomMultiModalDataset,
    SampleRequest,
    ShareGPTDataset,
    SonnetDataset,
# ... omitted for brevity ...
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `run_vllm` (lines 47-127)
```python
def run_vllm(
    requests: list[SampleRequest],
    n: int,
    engine_args: EngineArgs,
    do_profile: bool,
    disable_detokenize: bool = False,
) -> tuple[float, list[RequestOutput] | None]:
    from vllm import LLM, SamplingParams

    llm = LLM.from_engine_args(engine_args)
    assert all(
        llm.llm_engine.model_config.max_model_len
        >= (request.prompt_len + request.expected_output_len)
        for request in requests
    ), (
        "Please ensure that max_model_len is greater than the sum of"
        " prompt_len and expected_output_len for all requests."
    )
    # Add the requests to the engine.
    prompts: list[TextPrompt | TokensPrompt] = []
    sampling_params: list[SamplingParams] = []
    for request in requests:
        prompt = (
            TokensPrompt(prompt_token_ids=request.prompt["prompt_token_ids"])
            if "prompt_token_ids" in request.prompt
    # ... omitted for brevity ...
        end = time.perf_counter()
    return end - start, outputs
```
**EN:** Function `run_vllm` provides a reusable helper around the module's main workflow. Key calls such as `LLM.from_engine_args`, `all`, `TokensPrompt`, `TextPrompt`, `isinstance` show the concrete execution path.
**CN:** Function `run_vllm` 为模块主流程提供可复用的辅助逻辑。 像 `LLM.from_engine_args`, `all`, `TokensPrompt`, `TextPrompt`, `isinstance` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_vllm_chat` (lines 130-176)
```python
def run_vllm_chat(
    requests: list[SampleRequest],
    n: int,
    engine_args: EngineArgs,
    do_profile: bool,
    disable_detokenize: bool = False,
) -> tuple[float, list[RequestOutput]]:
    """
    Run vLLM chat benchmark. This function is recommended ONLY for benchmarking
    multimodal models as it properly handles multimodal inputs and chat
    formatting. For non-multimodal models, use run_vllm() instead.
    """
    from vllm import LLM, SamplingParams

    llm = LLM.from_engine_args(engine_args)

    assert all(
        llm.llm_engine.model_config.max_model_len
        >= (request.prompt_len + request.expected_output_len)
        for request in requests
    ), (
        "Please ensure that max_model_len is greater than the sum of "
        "prompt_len and expected_output_len for all requests."
    )

    # ... omitted for brevity ...
    end = time.perf_counter()
    return end - start, outputs
```
**EN:** Function `run_vllm_chat` provides a reusable helper around the module's main workflow. The docstring highlights: Run vLLM chat benchmark. Key calls such as `LLM.from_engine_args`, `all`, `prompts.append`, `sampling_params.append`, `SamplingParams` show the concrete execution path.
**CN:** Function `run_vllm_chat` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Run vLLM chat benchmark. 像 `LLM.from_engine_args`, `all`, `prompts.append`, `sampling_params.append`, `SamplingParams` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_vllm_async` (lines 179-247)
```python
async def run_vllm_async(
    requests: list[SampleRequest],
    n: int,
    engine_args: AsyncEngineArgs,
    do_profile: bool,
    disable_detokenize: bool = False,
) -> float:
    from vllm import SamplingParams
    from vllm.entrypoints.openai.api_server import (
        build_async_engine_client_from_engine_args,
    )

    async with build_async_engine_client_from_engine_args(
        engine_args,
    ) as llm:
        model_config = llm.model_config
        assert all(
            model_config.max_model_len
            >= (request.prompt_len + request.expected_output_len)
            for request in requests
        ), (
            "Please ensure that max_model_len is greater than the sum of"
            " prompt_len and expected_output_len for all requests."
        )

    # ... omitted for brevity ...
        end = time.perf_counter()
        return end - start
```
**EN:** Function `run_vllm_async` provides a reusable helper around the module's main workflow. Key calls such as `build_async_engine_client_from_engine_args`, `all`, `TokensPrompt`, `TextPrompt`, `isinstance` show the concrete execution path.
**CN:** Function `run_vllm_async` 为模块主流程提供可复用的辅助逻辑。 像 `build_async_engine_client_from_engine_args`, `all`, `TokensPrompt`, `TextPrompt`, `isinstance` 这样的关键调用展示了该代码块的具体执行路径。

### Function `run_hf` (lines 250-319)
```python
def run_hf(
    requests: list[SampleRequest],
    model: str,
    tokenizer: TokenizerLike,
    n: int,
    max_batch_size: int,
    trust_remote_code: bool,
    disable_detokenize: bool = False,
    dtype: torch.dtype | None = torch.float16,
    enable_torch_compile: bool = False,
) -> float:
    assert isinstance(tokenizer, PreTrainedTokenizerBase), (
        "the hf backend only supports HF tokenizers"
    )
    llm = AutoModelForCausalLM.from_pretrained(
        model, dtype=dtype, trust_remote_code=trust_remote_code
    )
    if llm.config.model_type == "llama":
        # To enable padding in the HF backend.
        tokenizer.pad_token = tokenizer.eos_token
    llm = llm.to(current_platform.device_type)
    if enable_torch_compile:
        llm = torch.compile(llm)

    pbar = tqdm(total=len(requests))
    # ... omitted for brevity ...
    end = time.perf_counter()
    return end - start
```
**EN:** Function `run_hf` provides a reusable helper around the module's main workflow. Key calls such as `isinstance`, `AutoModelForCausalLM.from_pretrained`, `llm.to`, `torch.compile`, `tqdm` show the concrete execution path.
**CN:** Function `run_hf` 为模块主流程提供可复用的辅助逻辑。 像 `isinstance`, `AutoModelForCausalLM.from_pretrained`, `llm.to`, `torch.compile`, `tqdm` 这样的关键调用展示了该代码块的具体执行路径。

### Function `save_to_pytorch_benchmark_format` (lines 322-338)
```python
def save_to_pytorch_benchmark_format(
    args: argparse.Namespace, results: dict[str, Any]
) -> None:
    pt_records = convert_to_pytorch_benchmark_format(
        args=args,
        metrics={
            "requests_per_second": [results["requests_per_second"]],
            "tokens_per_second": [results["tokens_per_second"]],
        },
        extra_info={
            k: results[k] for k in ["elapsed_time", "num_requests", "total_num_tokens"]
        },
    )
    if pt_records:
        # Don't use json suffix here as we don't want CI to pick it up
        pt_file = f"{os.path.splitext(args.output_json)[0]}.pytorch.json"
        write_to_json(pt_file, pt_records)
```
**EN:** Function `save_to_pytorch_benchmark_format` serializes data and writes it to a target representation. Key calls such as `convert_to_pytorch_benchmark_format`, `os.path.splitext`, `write_to_json` show the concrete execution path.
**CN:** Function `save_to_pytorch_benchmark_format` 负责序列化数据并写入目标表示。 像 `convert_to_pytorch_benchmark_format`, `os.path.splitext`, `write_to_json` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_requests` (lines 341-505)
```python
def get_requests(args, tokenizer):
    # Common parameters for all dataset types.
    common_kwargs = {
        "dataset_path": args.dataset_path,
        "random_seed": args.seed,
    }
    sample_kwargs = {
        "tokenizer": tokenizer,
        "lora_path": args.lora_path,
        "max_loras": args.max_loras,
        "lora_assignment": getattr(args, "lora_assignment", "random"),
        "num_requests": args.num_prompts,
    }

    if args.dataset_name == "random" or (
        args.dataset_path is None
        and args.dataset_name not in {"prefix_repetition", "random-mm", "random-rerank"}
    ):
        sample_kwargs["range_ratio"] = args.random_range_ratio
        # prefer random_* arguments, fall back to regular arguments
        random_prefix_len = getattr(args, "random_prefix_len", None)
        sample_kwargs["prefix_len"] = (
            random_prefix_len if random_prefix_len is not None else args.prefix_len
        )
        random_input_len = getattr(args, "random_input_len", None)
    # ... omitted for brevity ...
    requests = filter_requests_for_dp(requests, args.data_parallel_size)
    return requests
```
**EN:** Function `get_requests` provides a reusable helper around the module's main workflow. Key calls such as `getattr`, `ValueError`, `sample_kwargs.items`, `dataset_cls(**common_kwargs).sample`, `dataset_cls` show the concrete execution path.
**CN:** Function `get_requests` 为模块主流程提供可复用的辅助逻辑。 像 `getattr`, `ValueError`, `sample_kwargs.items`, `dataset_cls(**common_kwargs).sample`, `dataset_cls` 这样的关键调用展示了该代码块的具体执行路径。

### Function `filter_requests_for_dp` (lines 508-522)
```python
def filter_requests_for_dp(requests, data_parallel_size):
    # Note(zhuohan): The way we get data_parallel_rank is hacky and only
    # works for external launcher mode. Should be cleaned up and deprecated
    # in the future with a better vLLM distributed process design.
    if data_parallel_size == 1:
        return requests

    global_rank = int(os.environ["RANK"])
    world_size = int(os.environ["WORLD_SIZE"])
    data_parallel_rank = global_rank // (world_size // data_parallel_size)
    return [
        r
        for i, r in enumerate(requests)
        if i % data_parallel_size == data_parallel_rank
    ]
```
**EN:** Function `filter_requests_for_dp` provides a reusable helper around the module's main workflow. Key calls such as `int`, `enumerate` show the concrete execution path.
**CN:** Function `filter_requests_for_dp` 为模块主流程提供可复用的辅助逻辑。 像 `int`, `enumerate` 这样的关键调用展示了该代码块的具体执行路径。

### Function `validate_args` (lines 525-711)
```python
def validate_args(args):
    """
    Validate command-line arguments.
    """

    # === Deprecation and Defaulting ===
    if args.dataset is not None:
        warnings.warn(
            "The '--dataset' argument will be deprecated in the next release. "
            "Please use '--dataset-name' and '--dataset-path' instead.",
            stacklevel=2,
        )
        args.dataset_path = args.dataset

    if not getattr(args, "tokenizer", None):
        args.tokenizer = args.model

    # === Backend Validation ===
    valid_backends = {"vllm", "hf", "mii", "vllm-chat"}
    if args.backend not in valid_backends:
        raise ValueError(f"Unsupported backend: {args.backend}")

    # === Dataset Configuration ===
    if (
        not args.dataset
    # ... omitted for brevity ...
            "please use benchmark serving instead"
        )
```
**EN:** Function `validate_args` parses configuration, arguments, or structured metadata. The docstring highlights: Validate command-line arguments. Key calls such as `warnings.warn`, `getattr`, `ValueError`, `print`, `VisionArenaDataset.SUPPORTED_DATASET_PATHS.keys` show the concrete execution path.
**CN:** Function `validate_args` 负责解析配置、参数或结构化元数据。 文档字符串强调：Validate command-line arguments. 像 `warnings.warn`, `getattr`, `ValueError`, `print`, `VisionArenaDataset.SUPPORTED_DATASET_PATHS.keys` 这样的关键调用展示了该代码块的具体执行路径。

### Function `add_cli_args` (lines 714-904)
```python
def add_cli_args(parser: argparse.ArgumentParser):
    parser.add_argument(
        "--backend",
        type=str,
        choices=["vllm", "hf", "mii", "vllm-chat"],
        default="vllm",
    )
    parser.add_argument(
        "--dataset-name",
        type=str,
        choices=[
            "sharegpt",
            "random",
            "sonnet",
            "burstgpt",
            "hf",
            "prefix_repetition",
            "random-mm",
            "random-rerank",
        ],
        help="Name of the dataset to benchmark on.",
        default="sharegpt",
    )
    parser.add_argument(
        "--dataset",
    # ... omitted for brevity ...

    parser = AsyncEngineArgs.add_cli_args(parser)
```
**EN:** Function `add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `parser.add_argument`, `add_random_dataset_base_args`, `add_random_multimodal_dataset_args`, `float`, `AsyncEngineArgs.add_cli_args` show the concrete execution path.
**CN:** Function `add_cli_args` 负责解析配置、参数或结构化元数据。 像 `parser.add_argument`, `add_random_dataset_base_args`, `add_random_multimodal_dataset_args`, `float`, `AsyncEngineArgs.add_cli_args` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 907-1019)
```python
def main(args: argparse.Namespace):
    validate_args(args)
    if args.seed is None:
        args.seed = 0
    random.seed(args.seed)
    # Sample the requests.
    if (
        args.backend == "hf" or args.backend == "mii"
    ) and args.tokenizer_mode == "auto":
        # mistral_common tokenizer is only supported on vllm and vllm-chat backends;
        # for hf and mii backends, we use hf tokenizer
        args.tokenizer_mode = "hf"
    tokenizer = get_tokenizer(
        args.tokenizer,
        tokenizer_mode=args.tokenizer_mode,
        trust_remote_code=args.trust_remote_code,
    )
    requests = get_requests(args, tokenizer)
    is_multi_modal = any(request.multi_modal_data is not None for request in requests)
    request_outputs: list[RequestOutput] | None = None
    if args.backend == "vllm":
        if args.async_engine:
            elapsed_time = uvloop.run(
                run_vllm_async(
                    requests,
    # ... omitted for brevity ...
            json.dump(results, f, indent=4)
        save_to_pytorch_benchmark_format(args, results)
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `validate_args`, `random.seed`, `get_tokenizer`, `get_requests`, `any` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `validate_args`, `random.seed`, `get_tokenizer`, `get_requests`, `any` 这样的关键调用展示了该代码块的具体执行路径。

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
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import json`, `import os`, `import random`, `import time`, `import warnings`, `from typing import Any`
- **Third-party / 第三方**: `import torch`, `import uvloop`, `from tqdm import tqdm`, `from transformers import AutoModelForCausalLM, PreTrainedTokenizerBase`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.datasets import AIMODataset, ASRDataset, BurstGPTDataset, ConversationDataset, InstructCoderDataset, MultiModalConversationDataset, PrefixRepetitionRandomDataset, RandomDataset, RandomDatasetForReranking, RandomMultiModalDataset, SampleRequest, ShareGPTDataset, SonnetDataset, VisionArenaDataset, add_random_dataset_base_args, add_random_multimodal_dataset_args`, `from vllm.benchmarks.lib.utils import convert_to_pytorch_benchmark_format, write_to_json`, `from vllm.engine.arg_utils import AsyncEngineArgs, EngineArgs`, `from vllm.inputs import TextPrompt, TokensPrompt`, `from vllm.lora.request import LoRARequest`, `from vllm.outputs import RequestOutput`, `from vllm.platforms import current_platform`, `from vllm.sampling_params import BeamSearchParams`, `from vllm.tokenizers import TokenizerLike, get_tokenizer`, `from vllm.utils.async_utils import merge_async_iterators`

# data_processing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/data_processing.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache data processing. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 hicache data processing 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: Imports and setup / 导入与初始化
```python
import json
import os
import pickle
import random
from typing import List, Optional, Tuple, Union

import numpy as np
from nextqa import NExTQALoader

# from nextqa.video import , VideoPrompt
from tqdm.asyncio import tqdm
from transformers import PreTrainedTokenizerBase

from sglang.benchmark.datasets.common import (
    SHAREGPT_FILENAME,
    SHAREGPT_REPO_ID,
    gen_prompt,
)
from sglang.benchmark.datasets.generated_shared_prefix import get_gen_prefix_cache_path
from sglang.benchmark.utils import download_and_cache_hf_file
from sglang.lang.chat_template import get_chat_template, get_chat_template_by_model_path
from sglang.srt.entrypoints.openai.protocol import ChatCompletionMessageContentPart
from sglang.utils import encode_video_base64

# type of content fields, can be only prompts or with images/videos
MsgContent = Union[str, List[ChatCompletionMessageContentPart]]

# A list of all the conversations. Each conversation is a list of
# tuples. If multiturn is not enabled, the length of list is 1,
# containing only the first Q&A pair.
# For the shared prefix workload (synthetic, loogle, nextqa), it
# is a list of conversations sharing the same prefix (synthetic,
# doc, video)
SampleOutput = List[List[Tuple[MsgContent, int, int]]]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 37-91: Function `common_filter_chat` / 函数 `common_filter_chat`
```python
def common_filter_chat(
    num_requests: int,
    new_dataset: List,
    tokenizer: PreTrainedTokenizerBase,
    min_prompt_len: Optional[int],
    min_output_len: Optional[int],
    max_prompt_len: Optional[int],
    max_output_len: Optional[int],
    fixed_output_len: Optional[int],
) -> SampleOutput:
    # Filter out sequences that are too long or too short
    filtered_dataset: SampleOutput = []
    l = 0
    input_tokens = 0
    output_tokens = 0
    while l < num_requests:
        for i in range(len(new_dataset)):
            if l == num_requests:
                break
            processed = []
            for j in new_dataset[i]:
                # Tokenize the prompts and completions.
                prompt = j[0]
                prompt_token_ids = tokenizer.encode(prompt)
                prompt_len = len(prompt_token_ids)

                completion = j[1]
                completion_token_ids = tokenizer.encode(completion)
                output_len = (
                    len(completion_token_ids)
                    if fixed_output_len is None
                    else fixed_output_len
                )
                if (
                    min_prompt_len is not None
                    and prompt_len < min_prompt_len
                    or min_output_len is not None
                    and output_len < min_output_len
                    or max_prompt_len is not None
                    and prompt_len > max_prompt_len
                    or max_output_len is not None
                    and output_len > max_output_len
                ):
                    # Prune too short sequences.
                    continue
                input_tokens += prompt_len
                output_tokens += output_len
                processed.append((prompt, prompt_len, output_len))
            if len(processed) != 0:
                filtered_dataset.append(processed)
                l += 1

    print(f"#Input tokens: {input_tokens}")
    print(f"#Output tokens: {output_tokens}")
    return filtered_dataset
```
**EN:** `common_filter_chat` is a function that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. It returns `filtered_dataset` to the caller. Notable calls include `print`, `range`, `len`.
**CN:** `common_filter_chat` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `filtered_dataset`。其中较关键的调用包括 `print`, `range`, `len`。

### Lines 94-147: Function `sample_sharegpt_requests` / 函数 `sample_sharegpt_requests`
```python
def sample_sharegpt_requests(
    dataset_path: str,
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    disable_shuffle: bool = False,
    enable_multiturn: bool = True,
    fixed_output_len: Optional[int] = None,
) -> SampleOutput:
    if fixed_output_len is not None and fixed_output_len < 4:
        raise ValueError("output_len too small")

    # Download sharegpt if necessary
    if not os.path.isfile(dataset_path):
        dataset_path = download_and_cache_hf_file(
            repo_id=SHAREGPT_REPO_ID,
            filename=SHAREGPT_FILENAME,
        )

    # Load the dataset.
    with open(dataset_path) as f:
        dataset = json.load(f)
    # Filter out the conversations with less than 2 turns.
    dataset = [data for data in dataset if len(data["conversations"]) >= 2]

    # Keep one conversation in one list
    new_dataset = []
    for data in dataset:
        if len(data["conversations"]) % 2 != 0:
            continue
        if data["conversations"][0]["from"] != "human":
            continue
        chat = []
        total_len = 2
        if enable_multiturn:
            total_len = len(data["conversations"])
        for i in range(0, total_len, 2):
            # One user One Assistant
            chat.append(
                (
                    data["conversations"][i]["value"],
                    data["conversations"][i + 1]["value"],
                )
            )
        new_dataset.append(chat)

    if not disable_shuffle:
        # Shuffle the dataset.
        random.shuffle(new_dataset)

    # Filter out sequences that are too long or too short
    filtered_dataset: SampleOutput = common_filter_chat(
        num_requests, new_dataset, tokenizer, 4, 4, None, None, fixed_output_len
    )
    return filtered_dataset
```
**EN:** `sample_sharegpt_requests` is a function that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. It returns `filtered_dataset` to the caller. Notable calls include `common_filter_chat`, `ValueError`, `os.path.isfile`.
**CN:** `sample_sharegpt_requests` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `filtered_dataset`。其中较关键的调用包括 `common_filter_chat`, `ValueError`, `os.path.isfile`。

### Lines 150-195: Function `sample_ultrachat_requests` / 函数 `sample_ultrachat_requests`
```python
def sample_ultrachat_requests(
    dataset_path: str,
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    disable_shuffle: bool = False,
    enable_multiturn: bool = True,
    fixed_output_len: Optional[int] = None,
) -> SampleOutput:
    if fixed_output_len is not None and fixed_output_len < 4:
        raise ValueError("output_len too small")

    # Load the dataset
    dataset = []
    with open(dataset_path) as f:
        while True:
            line = f.readline()
            if not line:
                break
            dataset.append(json.loads(line))

    # Filter out the conversations with less than 2 turns.
    dataset = [data for data in dataset if len(data["data"]) >= 2]

    # Keep one conversation in one list
    new_dataset = []
    for data in dataset:
        if len(data["data"]) % 2 != 0:
            continue
        chat = []
        total_len = 2
        if enable_multiturn:
            total_len = len(data["data"])
        for i in range(0, total_len, 2):
            # One user One Assistant
            chat.append((data["data"][i], data["data"][i + 1]))
        new_dataset.append(chat)

    # Shuffle the dataset.
    if not disable_shuffle:
        random.shuffle(new_dataset)

    # Filter out sequences that are too long or too short
    filtered_dataset: SampleOutput = common_filter_chat(
        num_requests, new_dataset, tokenizer, 4, 4, None, None, fixed_output_len
    )
    return filtered_dataset
```
**EN:** `sample_ultrachat_requests` is a function that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. It returns `filtered_dataset` to the caller. Notable calls include `common_filter_chat`, `ValueError`, `open`.
**CN:** `sample_ultrachat_requests` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `filtered_dataset`。其中较关键的调用包括 `common_filter_chat`, `ValueError`, `open`。

### Lines 198-264: Function `sample_loogle_requests` / 函数 `sample_loogle_requests`
```python
def sample_loogle_requests(
    dataset_path: str,
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    disable_shuffle: bool = False,
    enable_multiturn: bool = True,
    enable_shared_prefix: bool = False,
    fixed_output_len: Optional[int] = None,
) -> SampleOutput:
    if fixed_output_len is not None and fixed_output_len < 4:
        raise ValueError("output_len too small")

    # Load the dataset
    dataset = []
    with open(dataset_path) as f:
        while True:
            line = f.readline()
            if not line:
                break
            dataset.append(json.loads(line))

    # Keep one conversation in one list
    new_dataset = []
    # TODO: Add shared prefix support for loogle
    # NOTE: Now we preprocess it only for chat
    for data in dataset:
        chat = []
        if (
            "qa_pairs" not in data
            or data["qa_pairs"] == "none"
            or len(data["qa_pairs"]) == 0
        ):
            # If Q is none (for summarization),
            # We add a question for summarization
            # And keep the summary up to 1024 words
            chat.append(
                (
                    "Input: "
                    + data["input"]
                    + " Question: "
                    + "Please summarize the input",
                    data["input"][:1024],
                )
            )
            new_dataset.append(chat)
        else:
            qa_pairs = eval(data["qa_pairs"])
            for i, qa in enumerate(qa_pairs):
                if i == 0 or enable_shared_prefix:
                    # Combine input with the first Q
                    chat.append(
                        ("Input: " + data["input"] + " Question: " + qa["Q"], qa["A"])
                    )
                elif enable_multiturn:
                    chat.append((qa["Q"], qa["A"]))

            new_dataset.append(chat)

    # Shuffle the dataset.
    if not disable_shuffle:
        random.shuffle(new_dataset)

    # Filter out sequences that are too long or too short
    filtered_dataset: SampleOutput = common_filter_chat(
        num_requests, new_dataset, tokenizer, 4, None, None, None, fixed_output_len
    )
    return filtered_dataset
```
**EN:** `sample_loogle_requests` is a function that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. It returns `filtered_dataset` to the caller. Notable calls include `common_filter_chat`, `ValueError`, `open`.
**CN:** `sample_loogle_requests` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `filtered_dataset`。其中较关键的调用包括 `common_filter_chat`, `ValueError`, `open`。

### Lines 267-344: Function `sample_nextqa_requests` / 函数 `sample_nextqa_requests`
```python
def sample_nextqa_requests(
    dataset_path: str,
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    max_frames: int,  # Specific for video
    model_path: str,
    disable_shuffle: bool = False,
    enable_multiturn: bool = True,  # No multiturn support for now
    backend: str = "sglang-oai",
    chat_template_name: Optional[str] = None,
    fixed_output_len: Optional[int] = None,
) -> SampleOutput:
    """
    Example of messages:
    message = {
        "role": "user",
        "content": [
            {"type": "image_url", "image_url": {"url": base64_data}},
            {"type": "text", "text": video.prompt},
        ],
    }
    """

    if fixed_output_len is None:
        fixed_output_len = 4096

    # TODO: Check for multiturn
    dataset = NExTQALoader(video_dir=dataset_path, max_frames=max_frames)
    new_dataset = []
    for v in dataset:
        new_dataset.append(v)

    if not disable_shuffle:
        random.shuffle(new_dataset)

    # TODO: prompt len can get from server side
    filtered_dataset = []
    l = 0
    while l < num_requests:
        for i in range(len(new_dataset)):
            if l == num_requests:
                break

            video = new_dataset[i]

            # text prompt
            prompt = video.prompt

            # NOTE: Chat Template is a must for video benchmark because we have to
            # add special image token for later expansion
            if backend == "sglang" or backend == "sglang-native":
                if "chat_template" in tokenizer.init_kwargs:
                    chat_template = get_chat_template(tokenizer.get_chat_template())
                elif chat_template_name is not None:
                    chat_template = get_chat_template(chat_template_name)
                else:
                    chat_template = get_chat_template_by_model_path(model_path)
                prompt = chat_template.image_token + prompt

            prompt_token_ids = tokenizer(prompt).input_ids
            prompt_len = len(prompt_token_ids)
            output_len = fixed_output_len  # max output len, not real output len

            # video input
            base64_data = encode_video_base64(video.path, video.num_frames)

            # NOTE: This will be replaced by the expanded length from the server
            prompt_len += video.num_frames

            # add to content
            content = [
                {"type": "image_url", "image_url": {"url": base64_data}},
                {"type": "text", "text": prompt},
            ]

            filtered_dataset.append([(content, prompt_len, output_len)])
            l += 1
    return filtered_dataset
```
**EN:** `sample_nextqa_requests` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. The docstring frames it as: Example of messages: message = { "role": "user", "content": [ {"type": "image_url", "image_url": {"url": base64_data}}, {"type": "text", "text": video.prompt}, ], } It returns `filtered_dataset` to the caller. Notable calls include `NExTQALoader`, `new_dataset.append`, `random.shuffle`.
**CN:** `sample_nextqa_requests` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `filtered_dataset`。其中较关键的调用包括 `NExTQALoader`, `new_dataset.append`, `random.shuffle`。

### Lines 347-431: Function `sample_random_requests` / 函数 `sample_random_requests`
```python
def sample_random_requests(
    input_len: int,
    output_len: int,
    num_prompts: int,
    range_ratio: float,
    tokenizer: PreTrainedTokenizerBase,
    dataset_path: str,
    disable_shuffle: bool = False,
) -> SampleOutput:

    input_lens = np.random.randint(
        max(int(input_len * range_ratio), 1),
        input_len + 1,
        size=num_prompts,
    )
    output_lens = np.random.randint(
        int(output_len * range_ratio),
        output_len + 1,
        size=num_prompts,
    )

    if True:
        # Sample token ids from ShareGPT and repeat/truncate them to satisfy the input_lens

        # Download sharegpt if necessary
        if not os.path.isfile(dataset_path):
            dataset_path = download_and_cache_hf_file(
                repo_id=SHAREGPT_REPO_ID,
                filename=SHAREGPT_FILENAME,
            )

        # Load the dataset.
        with open(dataset_path) as f:
            dataset = json.load(f)
        # Filter out the conversations with less than 2 turns.
        dataset = [data for data in dataset if len(data["conversations"]) >= 2]
        # Only keep the first two turns of each conversation.
        dataset = [
            (data["conversations"][0]["value"], data["conversations"][1]["value"])
            for data in dataset
        ]

        if not disable_shuffle:
            # Shuffle the dataset.
            random.shuffle(dataset)

        # Filter out sequences that are too long or too short
        input_requests: SampleOutput = []
        for data in dataset:
            i = len(input_requests)
            if i == num_prompts:
                break

            # Tokenize the prompts and completions.
            prompt = data[0]
            prompt_token_ids = tokenizer.encode(prompt)
            prompt_len = len(prompt_token_ids)

            # Skip empty prompt
            if prompt_len == 0:
                continue

            if prompt_len > input_lens[i]:
                input_ids = prompt_token_ids[: input_lens[i]]
            else:
                ratio = (input_lens[i] + prompt_len - 1) // prompt_len
                input_ids = (prompt_token_ids * ratio)[: input_lens[i]]
            prompt = tokenizer.decode(input_ids)
            input_requests.append([(prompt, int(input_lens[i]), int(output_lens[i]))])
    else:
        # Sample token ids from random integers. This can cause some NaN issues.
        offsets = np.random.randint(0, tokenizer.vocab_size, size=num_prompts)
        input_requests = []
        for i in range(num_prompts):
            prompt = tokenizer.decode(
                [
                    (offsets[i] + i + j) % tokenizer.vocab_size
                    for j in range(input_lens[i])
                ]
            )
            input_requests.append([(prompt, int(input_lens[i]), int(output_lens[i]))])

    print(f"#Input tokens: {np.sum(input_lens)}")
    print(f"#Output tokens: {np.sum(output_lens)}")
    return input_requests
```
**EN:** `sample_random_requests` is a function that sends requests to serving or OpenAI-compatible APIs, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. It returns `input_requests` to the caller. Notable calls include `np.random.randint`, `print`, `max`.
**CN:** `sample_random_requests` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。它会向调用方返回 `input_requests`。其中较关键的调用包括 `np.random.randint`, `print`, `max`。

### Lines 434-517: Function `sample_generated_shared_prefix_requests` / 函数 `sample_generated_shared_prefix_requests`
```python
def sample_generated_shared_prefix_requests(
    num_groups: int,
    prompts_per_group: int,
    system_prompt_len: int,
    question_len: int,
    output_len: int,
    tokenizer: PreTrainedTokenizerBase,
    args,
    disable_shuffle: bool = False,
) -> SampleOutput:
    """Generate benchmark requests with shared system prompts using random tokens and caching."""
    cache_path = get_gen_prefix_cache_path(
        args.seed,
        num_groups,
        prompts_per_group,
        system_prompt_len,
        question_len,
        output_len,
        tokenizer,
    )

    # Try to load from cache first
    if cache_path.exists():
        print(f"\nLoading cached generated input data from {cache_path}")
        with open(cache_path, "rb") as f:
            return pickle.load(f)

    print("\nGenerating new input data...")

    # Generate system prompts for each group
    system_prompts = []
    for _ in range(num_groups):
        system_prompt = gen_prompt(tokenizer, system_prompt_len)
        system_prompts.append(system_prompt)

    # Generate questions
    questions = []
    for _ in range(num_groups * prompts_per_group):
        question = gen_prompt(tokenizer, question_len)
        questions.append(question)

    # Combine system prompts with questions
    input_requests = []
    total_input_tokens = 0
    total_output_tokens = 0

    for group_idx in tqdm(range(num_groups), desc="Generating system prompt"):
        system_prompt = system_prompts[group_idx]
        input_requests.append([])
        for prompt_idx in tqdm(
            range(prompts_per_group), desc="Generating questions", leave=False
        ):
            question = questions[group_idx * prompts_per_group + prompt_idx]
            full_prompt = f"{system_prompt}\n\n{question}"
            prompt_len = len(tokenizer.encode(full_prompt))
            input_requests[-1].append((full_prompt, prompt_len, output_len))
            total_input_tokens += prompt_len
            total_output_tokens += output_len

    if not disable_shuffle:
        # Shuffle questions
        random.shuffle(input_requests)

    # Print statistics
    print(f"\nGenerated shared prefix dataset statistics:")
    print(f"Number of groups: {num_groups}")
    print(f"Prompts per group: {prompts_per_group}")
    print(f"Total prompts: {len(input_requests) * prompts_per_group}")
    print(f"Total input tokens: {total_input_tokens}")
    print(f"Total output tokens: {total_output_tokens}")
    print(
        f"Average system prompt length: {sum(len(tokenizer.encode(sp)) for sp in system_prompts) / len(system_prompts):.1f} tokens"
    )
    print(
        f"Average question length: {sum(len(tokenizer.encode(q)) for q in questions) / len(questions):.1f} tokens\n"
    )

    # Save to cache
    cache_path.parent.mkdir(parents=True, exist_ok=True)
    print(f"Caching generated input data to {cache_path}")
    with open(cache_path, "wb") as f:
        pickle.dump(input_requests, f)

    return input_requests
```
**EN:** `sample_generated_shared_prefix_requests` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. The docstring frames it as: Generate benchmark requests with shared system prompts using random tokens and caching. It returns `input_requests` to the caller. Notable calls include `get_gen_prefix_cache_path`, `cache_path.exists`, `print`.
**CN:** `sample_generated_shared_prefix_requests` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `input_requests`。其中较关键的调用包括 `get_gen_prefix_cache_path`, `cache_path.exists`, `print`。

### Lines 520-583: Function `get_dataset` / 函数 `get_dataset`
```python
def get_dataset(args, tokenizer):
    if args.dataset_name == "sharegpt":
        input_requests = sample_sharegpt_requests(
            dataset_path=args.dataset_path,
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            disable_shuffle=args.disable_shuffle,
            enable_multiturn=args.enable_multiturn,
            fixed_output_len=args.fixed_output_len,
        )
    elif args.dataset_name == "ultrachat":
        input_requests = sample_ultrachat_requests(
            dataset_path=args.dataset_path,
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            disable_shuffle=args.disable_shuffle,
            enable_multiturn=args.enable_multiturn,
            fixed_output_len=args.fixed_output_len,
        )
    elif args.dataset_name == "loogle":
        input_requests = sample_loogle_requests(
            dataset_path=args.dataset_path,
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            disable_shuffle=args.disable_shuffle,
            enable_multiturn=args.enable_multiturn,
            enable_shared_prefix=args.enable_shared_prefix,
            fixed_output_len=args.fixed_output_len,
        )
    elif args.dataset_name == "nextqa":
        input_requests = sample_nextqa_requests(
            dataset_path=args.dataset_path,
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            max_frames=args.max_frames,
            model_path=args.model,
            disable_shuffle=args.disable_shuffle,
            enable_multiturn=args.enable_multiturn,
            backend=args.backend,
            chat_template_name=args.chat_template,
            fixed_output_len=args.fixed_output_len,
        )
    elif args.dataset_name == "random":
        input_requests = sample_random_requests(
            input_len=args.random_input_len,
            output_len=args.random_output_len,
            num_prompts=args.num_prompts,
            range_ratio=args.random_range_ratio,
            tokenizer=tokenizer,
            dataset_path=args.dataset_path,
        )
    elif args.dataset_name == "generated-shared-prefix":
        input_requests = sample_generated_shared_prefix_requests(
            num_groups=args.gsp_num_groups,
            prompts_per_group=args.gsp_prompts_per_group,
            system_prompt_len=args.gsp_system_prompt_len,
            question_len=args.gsp_question_len,
            output_len=args.gsp_output_len,
            args=args,
            tokenizer=tokenizer,
        )
    else:
        raise ValueError(f"Unknown dataset: {args.dataset_name}")
    return input_requests
```
**EN:** `get_dataset` is a function that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. It returns `input_requests` to the caller. Notable calls include `sample_sharegpt_requests`, `sample_ultrachat_requests`, `sample_loogle_requests`.
**CN:** `get_dataset` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `input_requests`。其中较关键的调用包括 `sample_sharegpt_requests`, `sample_ultrachat_requests`, `sample_loogle_requests`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `pickle`, `random`, `typing`
- **Third-party / 第三方依赖**: `numpy`, `nextqa`, `tqdm.asyncio`, `transformers`
- **Internal / 项目内部依赖**: `sglang.benchmark.datasets.common`, `sglang.benchmark.datasets.generated_shared_prefix`, `sglang.benchmark.utils`, `sglang.lang.chat_template`, `sglang.srt.entrypoints.openai.protocol`, `sglang.utils`

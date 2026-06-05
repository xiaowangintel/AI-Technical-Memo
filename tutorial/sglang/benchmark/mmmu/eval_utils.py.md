# eval_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmmu/eval_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmmu eval. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mmmu eval 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and setup / 导入与初始化
```python
"""Response Parsing and Evaluation for various models"""

import argparse
import dataclasses
import json
import os
import pprint
import random
import re
from concurrent.futures import ThreadPoolExecutor, as_completed
from typing import Dict, Optional

import numpy as np
import torch
from data_utils import (
    CAT_SHORT2LONG,
    DOMAIN_CAT2SUB_CAT,
    construct_prompt,
    load_yaml,
    process_single_sample,
    save_json,
)
from datasets import concatenate_datasets, load_dataset
from tqdm import tqdm
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、准备张量并调用 GPU 内核。

### Lines 28-28: Class `EvalArgs` declaration / 类 `EvalArgs` 声明
```python
class EvalArgs:
```
**EN:** This block introduces class `EvalArgs`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `EvalArgs`，用于通过统一接口组织相关行为。

### Lines 29-43: Class-level state / 类级状态
```python
    seed: int = 42
    split: str = "validation"
    image_pixels_limit: int = -1
    result_filename: str = f"./val_sglang.json"
    prompt_format_file: str = "prompt_format.yaml"
    dataset_path: str = "MMMU/MMMU"
    extra_request_body: Optional[str] = None
    profile: bool = False
    profile_number: int = 5
    concurrency: int = 1
    max_new_tokens: Optional[int] = None
    temperature: Optional[float] = None
    response_answer_regex: str = "(?s)(.*)"
    lora_path: Optional[str] = None
    reasoning_effort: Optional[str] = None
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。

### Lines 46-130: Method `EvalArgs.add_cli_args` / 方法 `EvalArgs.add_cli_args`
```python
    def add_cli_args(parser: argparse.ArgumentParser):
        parser.add_argument(
            "--result-filename",
            type=str,
            default=EvalArgs.result_filename,
            help="The filename to save the evaluation results.",
        )
        parser.add_argument(
            "--image-pixels-limit",
            type=int,
            default=EvalArgs.image_pixels_limit,
            help="The maximum number of pixels allowed for an image. If an image exceeds this limit, it will be skipped during evaluation.",
        )
        parser.add_argument(
            "--dataset-path",
            type=str,
            default=EvalArgs.dataset_path,
            help="path to the dataset",
        )
        parser.add_argument("--seed", type=int, default=1, help="The random seed.")
        parser.add_argument(
            "--prompt-format-file",
            type=str,
            help="The path to the prompt format of mmmu. If not, a default format llava_config.yaml will be used",
        )
        parser.add_argument(
            "--split",
            type=str,
            default=EvalArgs.split,
            help='Split of the dataset to use for evaluation. Default is "validation".',
        )
        parser.add_argument(
            "--extra-request-body",
            metavar='{"key1": "value1", "key2": "value2"}',
            type=str,
            default=EvalArgs.extra_request_body,
            help="Append given JSON object to the request payload. You can use this to specify"
            "additional generate params like sampling params.",
        )
        parser.add_argument(
            "--profile", action="store_true", help="enable mmmu profile"
        )
        parser.add_argument(
            "--profile-number",
            type=int,
            default=EvalArgs.profile_number,
            help="Number of samples to profile. If not set, will profile all samples.",
        )
        parser.add_argument(
            "--concurrency",
            type=int,
            default=EvalArgs.concurrency,
            help="Number of concurrent requests to make during evaluation. Default is 1, which means no concurrency.",
        )
        parser.add_argument(
            "--max-new-tokens",
            type=int,
            default=EvalArgs.max_new_tokens,
            help="Maximum number of new tokens to generate per sample.",
        )
        parser.add_argument(
            "--temperature",
            type=float,
            default=EvalArgs.temperature,
            help="Sampling temperature for generation.",
        )
        parser.add_argument(
            "--response-answer-regex",
            type=str,
            default=EvalArgs.response_answer_regex,
            help="Specific regex to capture the answer from the response, string",
        )
        parser.add_argument(
            "--lora-path",
            type=str,
            default=EvalArgs.lora_path,
            help="Specify the LoRA path to use for evaluation. If specified, the value will be specified in the body of every request as `lora-path`.",
        )
        parser.add_argument(
            "--reasoning-effort",
            type=str,
            default=EvalArgs.reasoning_effort,
            choices=["none", "high"],
            help="Reasoning effort for the model (none or high).",
        )
```
**EN:** `EvalArgs.add_cli_args` is a method that builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and coordinates asynchronous or parallel execution. Notable calls include `parser.add_argument`.
**CN:** `EvalArgs.add_cli_args` 是一个方法，用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行。其中较关键的调用包括 `parser.add_argument`。

### Lines 133-135: Method `EvalArgs.from_cli_args` / 方法 `EvalArgs.from_cli_args`
```python
    def from_cli_args(cls, args: argparse.Namespace):
        attrs = [attr.name for attr in dataclasses.fields(cls)]
        return cls(**{attr: getattr(args, attr) for attr in attrs})
```
**EN:** `EvalArgs.from_cli_args` is a method that builds command-line arguments and runtime configuration. It returns `cls(**{attr: getattr(args, attr) for attr in attrs})` to the caller. Notable calls include `cls`, `dataclasses.fields`, `getattr`.
**CN:** `EvalArgs.from_cli_args` 是一个方法，用于构建命令行参数与运行时配置。它会向调用方返回 `cls(**{attr: getattr(args, attr) for attr in attrs})`。其中较关键的调用包括 `cls`, `dataclasses.fields`, `getattr`。

### Lines 138-151: Function `set_seed` / 函数 `set_seed`
```python
def set_seed(seed_value):
    """
    Set the seed for PyTorch (both CPU and CUDA), Python, and NumPy for reproducible results.

    :param seed_value: An integer value to be used as the seed.
    """
    torch.manual_seed(seed_value)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed_value)
        torch.cuda.manual_seed_all(seed_value)  # For multi-GPU setups
    random.seed(seed_value)
    np.random.seed(seed_value)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
```
**EN:** `set_seed` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. The docstring frames it as: Set the seed for PyTorch (both CPU and CUDA), Python, and NumPy for reproducible results. Notable calls include `torch.manual_seed`, `torch.cuda.is_available`, `random.seed`.
**CN:** `set_seed` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `torch.manual_seed`, `torch.cuda.is_available`, `random.seed`。

### Lines 154-255: Function `prepare_samples` / 函数 `prepare_samples`
```python
def prepare_samples(eval_args: EvalArgs):
    print("Preparing samples...")
    # Build prompts
    set_seed(eval_args.seed)

    prompt_format_file = (
        eval_args.prompt_format_file
        if eval_args.prompt_format_file is not None
        else os.path.join(os.path.dirname(__file__), "prompt_format.yaml")
    )
    # load config and process to one value
    eval_args.config = load_yaml(prompt_format_file)
    for key, value in eval_args.config.items():
        if key != "eval_params" and type(value) == list:
            assert len(value) == 1, "key {} has more than one value".format(key)
            eval_args.config[key] = value[0]

    # run for each subject in parallel
    sub_dataset_list = []
    subjects = list(CAT_SHORT2LONG.values())  # Get a fixed list of subjects

    print(f"Loading datasets for {len(subjects)} subjects...")
    with ThreadPoolExecutor() as executor:
        # Submit all load_dataset tasks
        future_to_subject = {
            executor.submit(
                load_dataset, eval_args.dataset_path, subject, split=eval_args.split
            ): subject
            for subject in subjects
        }

        # Collect results as they complete
        results = {}
        for future in tqdm(
            as_completed(future_to_subject),
            total=len(subjects),
            desc="Loading datasets",
        ):
            subject = future_to_subject[future]
            try:
                results[subject] = future.result()
            except Exception as exc:
                print(f"{subject} generated an exception: {exc}")

    # Ensure datasets are added in the original order for consistency
    for subject in subjects:
        if subject in results:
            sub_dataset_list.append(results[subject])
        else:
            # Handle cases where a dataset failed to load (optional, depends on desired behavior)
            print(f"Warning: Dataset for subject '{subject}' could not be loaded.")

    # merge all dataset
    dataset = concatenate_datasets(sub_dataset_list)

    # Prepare images in parallel
    images_path = os.path.expanduser("~/.cache/mmmu/images")
    os.makedirs(images_path, exist_ok=True)
    print(f"Saving images to: {images_path}")

    samples = []
    skip_count = 0

    def process_sample(i, sample):
        sample = process_single_sample(sample)
        sample = construct_prompt(sample, eval_args.config)
        image = sample["image"]
        width, height = image.size
        if 0 < eval_args.image_pixels_limit <= width * height:
            return None, True
        # Use a unique identifier for the image path to avoid potential collisions if indices reset
        image_path = f"{images_path}/image_{sample['id']}.png"
        if not os.path.exists(image_path):
            image.save(image_path)
        sample["image_path"] = image_path
        return sample, False

    print("Processing samples...")
    with ThreadPoolExecutor() as executor:
        # Pass the sample itself to process_sample, index is less reliable now
        futures = [
            executor.submit(
                process_sample, i, sample
            )  # Keep index i for tqdm maybe? Or remove it. Let's keep it for now.
            for i, sample in enumerate(dataset)
        ]
        for future in tqdm(
            as_completed(futures), total=len(dataset), desc="Processing samples"
        ):
            sample, skipped = future.result()
            if skipped:
                skip_count += 1
            elif sample:
                samples.append(sample)

    samples.sort(key=lambda x: x["final_input_prompt"])

    print(
        f"Skipping {skip_count} samples with large images, {round((float(skip_count) / len(dataset)) * 100, 2)}% of dataset"
    )
    print("Samples have been prepared")
    return samples
```
**EN:** `prepare_samples` is a function that coordinates asynchronous or parallel execution and loads, filters, or serializes benchmark datasets. It returns `samples` to the caller. Notable calls include `print`, `set_seed`, `load_yaml`.
**CN:** `prepare_samples` 是一个函数，用于协调异步或并行执行、加载、筛选或序列化基准测试数据集。它会向调用方返回 `samples`。其中较关键的调用包括 `print`, `set_seed`, `load_yaml`。

### Lines 258-272: Function `get_sampling_params` / 函数 `get_sampling_params`
```python
def get_sampling_params(eval_args):
    extra_request_body = {}
    if eval_args.extra_request_body:
        extra_request_body = json.loads(eval_args.extra_request_body)
    sampling_params = {
        **extra_request_body,
    }

    if eval_args.max_new_tokens is not None and eval_args.max_new_tokens > 0:
        sampling_params.update({"max_completion_tokens": eval_args.max_new_tokens})

    if eval_args.temperature is not None:
        sampling_params.update({"temperature": eval_args.temperature})

    return sampling_params
```
**EN:** `get_sampling_params` is a function that loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and reads or writes local files and intermediate artifacts. It returns `sampling_params` to the caller. Notable calls include `json.loads`, `sampling_params.update`.
**CN:** `get_sampling_params` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、读写本地文件及中间产物。它会向调用方返回 `sampling_params`。其中较关键的调用包括 `json.loads`, `sampling_params.update`。

### Lines 280-289: Top-level execution logic / 顶层执行逻辑
```python
_EXPLICIT_ANSWER_PATTERNS = (
    # "answer: X" / "Final answer: X" (with optional bold/parens)
    r"\banswer\s*:\s*\*{0,2}\s*\(?([A-Z])\)?\s*\*{0,2}(?![A-Za-z])",
    # bare "X" / "(X)" on its own line at the end of the response
    r"(?:^|\n)\s*\*{0,2}\s*\(?([A-Z])\)?\s*\*{0,2}\s*\.?\s*$",
    # "\boxed{X}" (LaTeX boxed answer, common in math/CoT outputs)
    r"\\boxed\{\s*\*{0,2}\s*\(?([A-Z])\)?\s*\*{0,2}\s*\}",
    # "(the) answer is X" / "(the) correct answer is X"
    r"\b(?:the\s+)?answer\s+is\s*\*{0,2}\s*\(?([A-Z])\)?\s*\*{0,2}(?![A-Za-z])",
)
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers. In practice it computes evaluation scores and aggregate statistics.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。 从实现上看，它会计算评测分数与聚合统计结果。

### Lines 292-300: Function `_parse_explicit_multi_choice_answer` / 函数 `_parse_explicit_multi_choice_answer`
```python
def _parse_explicit_multi_choice_answer(response, all_choices):
    choice_map = {choice.upper(): choice for choice in all_choices}
    matches = []
    for pattern in _EXPLICIT_ANSWER_PATTERNS:
        for match in re.finditer(pattern, response, flags=re.IGNORECASE):
            candidate = match.group(1).upper()
            if candidate in choice_map:
                matches.append((match.start(1), choice_map[candidate]))
    return max(matches)[1] if matches else None
```
**EN:** `_parse_explicit_multi_choice_answer` is a function that computes evaluation scores and aggregate statistics. It returns `max(matches)[1] if matches else None` to the caller. Notable calls include `choice.upper`, `re.finditer`, `match.group(1).upper`.
**CN:** `_parse_explicit_multi_choice_answer` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `max(matches)[1] if matches else None`。其中较关键的调用包括 `choice.upper`, `re.finditer`, `match.group(1).upper`。

### Lines 303-359: Function `parse_multi_choice_response` / 函数 `parse_multi_choice_response`
```python
def parse_multi_choice_response(response, all_choices, index2ans):
    """
    Parse the prediction from the generated response.
    Return the predicted index e.g., A, B, C, D.
    """
    explicit_answer = _parse_explicit_multi_choice_answer(response, all_choices)
    if explicit_answer is not None:
        return explicit_answer

    for char in [",", ".", "!", "?", ";", ":", "'"]:
        response = response.strip(char)
    response = " " + response + " "  # add space to avoid partial match

    index_ans = True
    ans_with_brack = False
    candidates = []
    for choice in all_choices:  # e.g., (A) (B) (C) (D)
        if f"({choice})" in response:
            candidates.append(choice)
            ans_with_brack = True

    if len(candidates) == 0:
        for choice in all_choices:  # e.g., A B C D
            if f" {choice} " in response:
                candidates.append(choice)

    # if all above doesn't get candidates, check if the content is larger than 5 tokens and try to parse the example
    if len(candidates) == 0 and len(response.split()) > 5:
        for index, ans in index2ans.items():
            if ans.lower() in response.lower():
                candidates.append(index)
                index_ans = False  # it's content ans.

    if len(candidates) == 0:  # still not get answer, randomly choose one.
        pred_index = random.choice(all_choices)
    elif len(candidates) > 1:
        start_indexes = []
        if index_ans:
            if ans_with_brack:
                for can in candidates:
                    index = response.rfind(f"({can})")
                    start_indexes.append(index)  # -1 will be ignored anyway
                # start_indexes = [generated_response.index(f'({can})') for can in candidates]
            else:
                for can in candidates:
                    index = response.rfind(f" {can} ")
                    start_indexes.append(index)
        else:
            for can in candidates:
                index = response.lower().rfind(index2ans[can].lower())
                start_indexes.append(index)
        # get the last one
        pred_index = candidates[np.argmax(start_indexes)]
    else:  # if only one candidate, use it.
        pred_index = candidates[0]

    return pred_index
```
**EN:** `parse_multi_choice_response` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. The docstring frames it as: Parse the prediction from the generated response. It returns `pred_index` to the caller. Notable calls include `_parse_explicit_multi_choice_answer`, `response.strip`, `len`.
**CN:** `parse_multi_choice_response` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `pred_index`。其中较关键的调用包括 `_parse_explicit_multi_choice_answer`, `response.strip`, `len`。

### Lines 363-372: Function `check_is_number` / 函数 `check_is_number`
```python
def check_is_number(string):
    """
    Check if the given string a number.
    """
    try:
        float(string.replace(",", ""))
        return True
    except ValueError:
        # check if there's comma inside
        return False
```
**EN:** `check_is_number` is a function that implements the core logic for this scope. The docstring frames it as: Check if the given string a number. It returns `True` to the caller. Notable calls include `float`, `string.replace`.
**CN:** `check_is_number` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `True`。其中较关键的调用包括 `float`, `string.replace`。

### Lines 375-397: Function `normalize_str` / 函数 `normalize_str`
```python
def normalize_str(string):
    """
    Normalize the str to lower case and make them float numbers if possible.
    """
    # check if characters in the string

    # if number, numerize it.
    string = string.strip()

    is_number = check_is_number(string)

    if is_number:
        string = string.replace(",", "")
        string = float(string)
        # leave 2 decimal
        string = round(string, 2)
        return [string]
    else:  # it's likely to be a string
        # lower it
        string = string.lower()
        if len(string) == 1:
            return [" " + string, string + " "]  # avoid trivial matches
        return [string]
```
**EN:** `normalize_str` is a function that computes evaluation scores and aggregate statistics. The docstring frames it as: Normalize the str to lower case and make them float numbers if possible. It returns `[string]` to the caller. Notable calls include `string.strip`, `check_is_number`, `string.replace`.
**CN:** `normalize_str` 是一个函数，用于计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `[string]`。其中较关键的调用包括 `string.strip`, `check_is_number`, `string.replace`。

### Lines 400-420: Function `extract_numbers` / 函数 `extract_numbers`
```python
def extract_numbers(string):
    """
    Exact all forms of numbers from a string with regex.
    """
    # Pattern for numbers with commas
    pattern_commas = r"-?\b\d{1,3}(?:,\d{3})+\b"
    # Pattern for scientific notation
    pattern_scientific = r"-?\d+(?:\.\d+)?[eE][+-]?\d+"
    # Pattern for simple numbers without commas
    pattern_simple = r"-?(?:\d+\.\d+|\.\d+|\d+\b)(?![eE][+-]?\d+)(?![,\d])"

    # Extract numbers with commas
    numbers_with_commas = re.findall(pattern_commas, string)
    # Extract numbers in scientific notation
    numbers_scientific = re.findall(pattern_scientific, string)
    # Extract simple numbers without commas
    numbers_simple = re.findall(pattern_simple, string)

    # Combine all extracted numbers
    all_numbers = numbers_with_commas + numbers_scientific + numbers_simple
    return all_numbers
```
**EN:** `extract_numbers` is a function that implements the core logic for this scope. The docstring frames it as: Exact all forms of numbers from a string with regex. It returns `all_numbers` to the caller. Notable calls include `re.findall`.
**CN:** `extract_numbers` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `all_numbers`。其中较关键的调用包括 `re.findall`。

### Lines 423-493: Function `parse_open_response` / 函数 `parse_open_response`
```python
def parse_open_response(response):
    """
    Parse the prediction from the generated response.
    Return a list of predicted strings or numbers.
    """

    # content = content.strip("\n").strip(".").strip(" ")
    def get_key_subresponses(response):
        key_responses = []
        response = response.strip().strip(".").lower()
        sub_responses = re.split(r"\.\s(?=[A-Z])|\n", response)
        indicators_of_keys = [
            "could be ",
            "so ",
            "is ",
            "thus ",
            "therefore ",
            "final ",
            "answer ",
            "result ",
        ]
        key_responses = []
        for index, resp in enumerate(sub_responses):
            # if last one, accept it's an equation (the entire response can be just one sentence with equation)
            if index == len(sub_responses) - 1:
                indicators_of_keys.extend(["="])
            shortest_key_response = None  # the shortest response that may contain the answer (tail part of the response)
            for indicator in indicators_of_keys:
                if indicator in resp:
                    if not shortest_key_response:
                        shortest_key_response = resp.split(indicator)[-1].strip()
                    else:
                        if len(resp.split(indicator)[-1].strip()) < len(
                            shortest_key_response
                        ):
                            shortest_key_response = resp.split(indicator)[-1].strip()
                    # key_responses.append(resp.split(indicator)[1].strip())

            if shortest_key_response:
                # and it's not trivial
                if shortest_key_response.strip() not in [
                    ":",
                    ",",
                    ".",
                    "!",
                    "?",
                    ";",
                    ":",
                    "'",
                ]:
                    key_responses.append(shortest_key_response)
        if len(key_responses) == 0:  # did not found any
            return [response]
        return key_responses

    # pdb.set_trace()
    key_responses = get_key_subresponses(response)

    pred_list = key_responses.copy()  # keep the original string response
    for resp in key_responses:
        pred_list.extend(extract_numbers(resp))

    tmp_pred_list = []
    for i in range(len(pred_list)):
        tmp_pred_list.extend(normalize_str(pred_list[i]))
    pred_list = tmp_pred_list

    # remove duplicates
    pred_list = list(set(pred_list))

    return pred_list
```
**EN:** `parse_open_response` is a function that computes evaluation scores and aggregate statistics. The docstring frames it as: Parse the prediction from the generated response. It returns `pred_list` to the caller. Notable calls include `get_key_subresponses`, `key_responses.copy`, `range`.
**CN:** `parse_open_response` 是一个函数，用于计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `pred_list`。其中较关键的调用包括 `get_key_subresponses`, `key_responses.copy`, `range`。

### Lines 499-525: Function `eval_multi_choice` / 函数 `eval_multi_choice`
```python
def eval_multi_choice(gold_i, pred_i):
    """
    Evaluate a multiple choice instance.
    """
    correct = False
    # for case like Answer: A, Answer is A, answer is A, answer: A
    for _exp in ["Answer:", "Answer is ", "answer is ", "answer: "]:
        if _exp in pred_i:
            pred_i = pred_i.split(_exp)[1].strip()
            break
    # for case like (A), (B), (C), (D) ......
    if "(" in pred_i and ")" in pred_i:
        try:
            pred_i = re.search(r"\(([A-Z])\)", pred_i).group(1)
        except:
            print(f"Error to extract answer from: {pred_i}")
            pass
    # only they are exactly the same, we consider it as correct
    if isinstance(gold_i, list):
        for answer in gold_i:
            if answer == pred_i:
                correct = True
                break
    else:  # gold_i is a string
        if gold_i == pred_i:
            correct = True
    return correct
```
**EN:** `eval_multi_choice` is a function that computes evaluation scores and aggregate statistics. The docstring frames it as: Evaluate a multiple choice instance. It returns `correct` to the caller. Notable calls include `isinstance`, `pred_i.split(_exp)[1].strip`, `re.search('\\(([A-Z])\\)', pred_i).group`.
**CN:** `eval_multi_choice` 是一个函数，用于计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `correct`。其中较关键的调用包括 `isinstance`, `pred_i.split(_exp)[1].strip`, `re.search('\\(([A-Z])\\)', pred_i).group`。

### Lines 528-553: Function `eval_open` / 函数 `eval_open`
```python
def eval_open(gold_i, pred_i):
    """
    Evaluate an open question instance
    """
    correct = False
    if isinstance(gold_i, list):
        # use float to avoid trivial matches
        norm_answers = []
        for answer in gold_i:
            norm_answers.extend(normalize_str(answer))
    else:
        norm_answers = normalize_str(gold_i)
    for pred in pred_i:  # pred is already normalized in parse response phase
        if isinstance(pred, str):  # if it's a string, then find if ans in the pred_i
            for norm_ans in norm_answers:
                # only see if the string answer in the string pred
                if isinstance(norm_ans, str) and norm_ans in pred:
                    if not correct:
                        correct = True
                    break
        else:  # it's a float number
            if pred in norm_answers:
                if not correct:
                    correct = True
                break
    return correct
```
**EN:** `eval_open` is a function that computes evaluation scores and aggregate statistics and reads or writes local files and intermediate artifacts. The docstring frames it as: Evaluate an open question instance It returns `correct` to the caller. Notable calls include `isinstance`, `normalize_str`, `norm_answers.extend`.
**CN:** `eval_open` 是一个函数，用于计算评测分数与聚合统计结果、读写本地文件及中间产物。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `correct`。其中较关键的调用包括 `isinstance`, `normalize_str`, `norm_answers.extend`。

### Lines 557-580: Function `evaluate` / 函数 `evaluate`
```python
def evaluate(samples):
    """
    Batch evaluation for multiple choice and open questions.
    """
    pred_correct = 0
    judge_dict = dict()
    for sample in samples:
        gold_i = sample["answer"]
        pred_i = sample["parsed_pred"]
        if sample["question_type"] == "multiple-choice":
            correct = eval_multi_choice(gold_i, pred_i)
        else:  # open question
            correct = eval_open(gold_i, pred_i)

        if correct:
            judge_dict[sample["id"]] = "Correct"
            pred_correct += 1
        else:
            # print(f"Wrong! expected {pred_i}, answered with {gold_i}")
            judge_dict[sample["id"]] = "Wrong"

    if len(samples) == 0:
        return {"acc": 0}
    return judge_dict, {"acc": pred_correct / len(samples)}
```
**EN:** `evaluate` is a function that loads, filters, or serializes benchmark datasets, computes evaluation scores and aggregate statistics, and reads or writes local files and intermediate artifacts. The docstring frames it as: Batch evaluation for multiple choice and open questions. It returns `(judge_dict, {'acc': pred_correct / len(samples)})` to the caller. Notable calls include `dict`, `len`, `eval_multi_choice`.
**CN:** `evaluate` 是一个函数，用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果、读写本地文件及中间产物。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(judge_dict, {'acc': pred_correct / len(samples)})`。其中较关键的调用包括 `dict`, `len`, `eval_multi_choice`。

### Lines 584-593: Function `calculate_ins_level_acc` / 函数 `calculate_ins_level_acc`
```python
def calculate_ins_level_acc(results: Dict):
    """Calculate the instruction level accuracy for given Subject results"""
    acc = 0
    ins_num = 0
    for cat_results in results.values():
        acc += cat_results["acc"] * cat_results["num_example"]
        ins_num += cat_results["num_example"]
    if ins_num == 0:
        return 0
    return acc / ins_num
```
**EN:** `calculate_ins_level_acc` is a function that computes evaluation scores and aggregate statistics. The docstring frames it as: Calculate the instruction level accuracy for given Subject results It returns `acc / ins_num` to the caller. Notable calls include `results.values`.
**CN:** `calculate_ins_level_acc` 是一个函数，用于计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `acc / ins_num`。其中较关键的调用包括 `results.values`。

### Lines 596-617: Function `process_result` / 函数 `process_result`
```python
def process_result(response, sample, answer_dict, out_samples):
    if response is None:
        return
    if sample["question_type"] == "multiple-choice":
        pred_ans = parse_multi_choice_response(
            response, sample["all_choices"], sample["index2ans"]
        )
    else:  # open question
        pred_ans = response

    out_samples[sample["id"]] = {
        "pred_ans": pred_ans,
        "original_response": sample["original_response"],
        "ground_truth": sample["answer"],
        "question_type": sample["question_type"],
    }

    # set ground truth answer
    answer_dict[sample["id"]] = {
        "question_type": sample["question_type"],
        "ground_truth": sample["answer"],
    }
```
**EN:** `process_result` is a function that loads, filters, or serializes benchmark datasets and computes evaluation scores and aggregate statistics. Notable calls include `parse_multi_choice_response`.
**CN:** `process_result` 是一个函数，用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。其中较关键的调用包括 `parse_multi_choice_response`。

### Lines 620-731: Function `eval_result` / 函数 `eval_result`
```python
def eval_result(model_answer_path, answer_dict, eval_output_path=None):
    if eval_output_path is None:
        eval_output_path = model_answer_path
    print("Evaluating...")
    output_dict = json.load(open(model_answer_path))
    # answer_dict = json.load(open(answer_path))

    # group by category
    output_dict_w_cat = {}
    for data_id, parsed_pred in output_dict.items():
        if isinstance(parsed_pred, str):
            parsed_pred = parsed_pred
        elif isinstance(parsed_pred, dict):
            parsed_pred = parsed_pred["pred_ans"]
        else:
            raise ValueError(f"Unknown type of parsed_pred: {type(parsed_pred)}")
        category = "_".join(data_id.split("_")[1:-1])
        if category not in output_dict_w_cat:
            output_dict_w_cat.update({category: {}})
        output_dict_w_cat[category].update({data_id: parsed_pred})

    # group by category
    answer_dict_w_cat = {}
    for data_id, parsed_pred in answer_dict.items():
        category = "_".join(data_id.split("_")[1:-1])
        if category not in answer_dict_w_cat:
            answer_dict_w_cat.update({category: {}})
        answer_dict_w_cat[category].update({data_id: parsed_pred})

    evaluation_result = {}

    for category in CAT_SHORT2LONG.values():
        # print("Evaluating: {}".format(category))
        # get cat_outputs and cat_answers
        try:
            cat_outputs = output_dict_w_cat[category]
            cat_answers = answer_dict_w_cat[category]
        except KeyError:
            # print("Skipping {} for not found".format(category))
            continue

        exampels_to_eval = []
        for data_id, parsed_pred in cat_outputs.items():
            question_type = cat_answers[data_id]["question_type"]
            if question_type != "multiple-choice":
                parsed_pred = parse_open_response(
                    parsed_pred
                )  # mainly for type consistency (make it number, etc.)
            else:
                parsed_pred = parsed_pred

            exampels_to_eval.append(
                {
                    "id": data_id,
                    "question_type": question_type,
                    "answer": cat_answers[data_id]["ground_truth"],
                    "parsed_pred": parsed_pred,
                }
            )

        judge_dict, metric_dict = evaluate(exampels_to_eval)
        metric_dict.update({"num_example": len(exampels_to_eval)})
        for key, value in judge_dict.items():
            output_dict[key]["judge"] = value

        evaluation_result[category] = metric_dict

    save_json(model_answer_path, output_dict)
    printable_results = {}
    # pdb.set_trace()
    # add domain Subject
    for domain, in_domain_cats in DOMAIN_CAT2SUB_CAT.items():
        in_domain_cat_results = {}
        for cat_name in in_domain_cats:  # use the order in DOMAIN_CAT2SUB_CAT
            if cat_name in evaluation_result.keys():
                in_domain_cat_results[cat_name] = evaluation_result[cat_name]
            else:
                pass
        in_domain_ins_acc = calculate_ins_level_acc(in_domain_cat_results)
        in_domain_data_num = sum(
            [
                cat_results["num_example"]
                for cat_results in in_domain_cat_results.values()
            ]
        )
        printable_results["Overall-" + domain] = {
            "num": int(in_domain_data_num),
            "acc": round(in_domain_ins_acc, 3),
        }
        # add sub category
        for cat_name, cat_results in in_domain_cat_results.items():
            printable_results[cat_name] = {
                "num": int(cat_results["num_example"]),
                "acc": round(cat_results["acc"], 3),
            }

    # table.append(["-----------------------------", "-----", "----"])
    all_ins_acc = calculate_ins_level_acc(evaluation_result)
    overall_acc = round(all_ins_acc, 3)
    printable_results["Overall"] = {
        "num": sum(
            [cat_results["num_example"] for cat_results in evaluation_result.values()]
        ),
        "acc": overall_acc,
    }
    pprint.pprint(printable_results)
    out = eval_output_path
    with open(out, "w", encoding="utf-8") as outfile:
        json.dump(printable_results, outfile)
        print(f"eval out saved to {out}")

    print(f"Overall accuracy: {overall_acc}")
```
**EN:** `eval_result` is a function that loads, filters, or serializes benchmark datasets, computes evaluation scores and aggregate statistics, and reads or writes local files and intermediate artifacts. Notable calls include `print`, `json.load`, `output_dict.items`.
**CN:** `eval_result` 是一个函数，用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果、读写本地文件及中间产物。其中较关键的调用包括 `print`, `json.load`, `output_dict.items`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `dataclasses`, `json`, `os`, `pprint`, `random`, `re`, `concurrent.futures`, `typing`
- **Third-party / 第三方依赖**: `numpy`, `torch`, `data_utils`, `datasets`, `tqdm`

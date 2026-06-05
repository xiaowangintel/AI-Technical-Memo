# bench_hf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmmu/bench_hf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmmu hf. It primarily builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. / 该 Python 模块聚焦于 mmmu hf 相关流程。它主要用于构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and setup / 导入与初始化
```python
import argparse

import PIL
import torch
from data_utils import save_json
from eval_utils import (
    EvalArgs,
    eval_result,
    get_sampling_params,
    prepare_samples,
    process_result,
)
from tqdm import tqdm
from transformers import AutoModel, AutoProcessor, GenerationConfig
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

### Lines 18-159: Function `eval_mmmu` / 函数 `eval_mmmu`
```python
def eval_mmmu(args):
    eval_args = EvalArgs.from_cli_args(args)

    sampling_params = get_sampling_params(eval_args)
    generation_config = GenerationConfig(
        max_new_tokens=sampling_params["max_new_tokens"],
        do_sample=False,
    )

    try:
        from transformers import AutoModelForImageTextToText

        model = AutoModelForImageTextToText.from_pretrained(
            args.model_path,
            torch_dtype="auto",
            trust_remote_code=True,
        )
    except Exception as first_exception:
        try:
            # check if the model is belongs to internvl
            if "InternVL" in args.model_path:
                from transformers import AutoTokenizer

                from sglang.srt.multimodal.internvl_utils import image_to_pixel_values

                tokenizer = AutoTokenizer.from_pretrained(args.model_path)
                model = AutoModel.from_pretrained(
                    args.model_path,
                    torch_dtype="auto",
                    trust_remote_code=True,
                )
                generation_config_internvl = dict(
                    max_new_tokens=sampling_params["max_new_tokens"], do_sample=False
                )

            else:
                model = AutoModel.from_pretrained(
                    args.model_path,
                    torch_dtype="auto",
                    trust_remote_code=True,
                    init_tts=False,
                )
        except Exception as second_exception:
            raise RuntimeError(
                f"Failed to load model: First attempt failed with {first_exception}, "
                f"second attempt failed with {second_exception}"
            ) from second_exception

    model = model.eval().cuda()

    processor = AutoProcessor.from_pretrained(
        args.model_path, torch_dtype="auto", device_map="auto", trust_remote_code=True
    )

    samples = prepare_samples(eval_args)
    if getattr(args, "limit", None):
        total = len(samples)
        samples = samples[: args.limit]
        print(f"--limit {args.limit}: keeping {len(samples)} of {total} samples")
    out_samples = dict()

    answer_dict = {}
    for sample in tqdm(samples):
        prompt = sample["final_input_prompt"]
        image = sample["image"]
        prefix = prompt.split("<")[0]
        suffix = prompt.split(">")[1]
        assert image is not None

        if "InternVL" in args.model_path:
            image = PIL.Image.open(sample["image_path"]).convert("RGB")
            pixel_values = image_to_pixel_values(
                image, input_size=448, max_num=12, use_thumbnail=True
            )
            pixel_values = pixel_values.to(device="cuda", dtype=torch.bfloat16)
            contents = ""
            if prefix:
                contents += prefix
            contents += "<image>\n"
            if suffix:
                contents += suffix
            response = model.chat(
                tokenizer, pixel_values, contents, generation_config_internvl
            )
            sample["original_response"] = response
            process_result(response, sample, answer_dict, out_samples)
            continue

        contents = []
        if prefix:
            contents += [{"type": "text", "text": prefix}]
        contents += [
            {
                "type": "image",
                "image": sample["image_path"],
            }
        ]
        if suffix:
            contents += [{"type": "text", "text": suffix}]
        messages = [{"role": "user", "content": contents}]
        try:
            model_inputs = processor.tokenizer.apply_chat_template(
                messages,
                tokenize=True,
                return_dict=True,
                add_generation_prompt=True,
                return_tensors="pt",
            ).to(model.device)
            input_len = model_inputs["input_ids"].shape[-1]
            generation = model.generate(
                **model_inputs, generation_config=generation_config
            )
            generation = generation[0][input_len:]
            response = processor.decode(generation, skip_special_tokens=True)
        except:
            contents = []
            if prefix:
                contents += [prefix]
            image = PIL.Image.open(sample["image_path"])
            contents += [image]
            if suffix:
                contents += [suffix]
            messages = [{"role": "user", "content": contents}]
            response = model.chat(
                msgs=messages,
                tokenizer=processor.tokenizer,
                sampling=False,
                max_new_tokens=sampling_params["max_new_tokens"],
                use_tts_template=False,
                generate_audio=False,
                temperature=0.0,
            )
        sample["original_response"] = response
        process_result(response, sample, answer_dict, out_samples)

    args.output_path = f"{args.model_path}_answer_hf.json"
    save_json(args.output_path, out_samples)
    eval_result(
        model_answer_path=args.output_path,
        answer_dict=answer_dict,
        eval_output_path=f"{args.model_path}_val_hf.json",
    )
```
**EN:** `eval_mmmu` is a function that prepares tensors and invokes GPU kernels, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. Notable calls include `torch.no_grad`, `EvalArgs.from_cli_args`, `get_sampling_params`.
**CN:** `eval_mmmu` 是一个函数，用于准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `torch.no_grad`, `EvalArgs.from_cli_args`, `get_sampling_params`。

### Lines 162-179: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--model-path",
        type=str,
        help="The path of the model weights. This can be a local folder or a Hugging Face repo ID.",
        required=True,
    )
    parser.add_argument(
        "--limit",
        type=int,
        default=None,
        help="If set, only evaluate this many samples (debug / smoke runs).",
    )
    EvalArgs.add_cli_args(parser)
    args = parser.parse_args()

    eval_mmmu(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Audio processing / 音频处理**: Prepares waveform data for speech or streaming benchmarks. / 为语音或流式基准测试准备波形数据。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`
- **Third-party / 第三方依赖**: `PIL`, `torch`, `data_utils`, `eval_utils`, `tqdm`, `transformers`
- **Internal / 项目内部依赖**: `sglang.srt.multimodal.internvl_utils`

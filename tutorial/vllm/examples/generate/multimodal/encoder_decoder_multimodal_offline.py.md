# encoder_decoder_multimodal_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/encoder_decoder_multimodal_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with the explicit/implicit prompt format on enc-dec LMMs for text generation / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference with
the explicit/implicit prompt format on enc-dec LMMs for text generation.
"""
```
**EN:** This example shows how to use vLLM for running offline inference with the explicit/implicit prompt format on enc-dec LMMs for text generation.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
import time
from collections.abc import Sequence
from typing import NamedTuple

from vllm import LLM, EngineArgs, PromptType, SamplingParams
from vllm.assets.audio import AudioAsset
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as os, time, collections.abc, and typing and pulls in vLLM APIs like vllm, vllm.assets.audio, and vllm.utils.argparse_utils.
**CN:** 这一部分加载 os、time、collections.abc，以及 typing 等辅助库，并引入 vllm、vllm.assets.audio，以及 vllm.utils.argparse_utils 等 vLLM API。

### Class: ModelRequestData
```python
class ModelRequestData(NamedTuple):
    engine_args: EngineArgs
    prompts: Sequence[PromptType]
```
**EN:** This class packages the ModelRequestData abstraction used by the example. It extends NamedTuple.
**CN:** 该类封装了示例中使用的 ModelRequestData 抽象。它继承自 NamedTuple。

### Function: run_whisper
```python
def run_whisper():
    os.environ["VLLM_WORKER_MULTIPROC_METHOD"] = "spawn"

    engine_args = EngineArgs(
        model="openai/whisper-large-v3-turbo",
        max_model_len=448,
        max_num_seqs=16,
        limit_mm_per_prompt={"audio": 1},
        dtype="half",
    )

    prompts = [
        {  # Test implicit prompt
            "prompt": "<|startoftranscript|>",
            "multi_modal_data": {
                "audio": AudioAsset("mary_had_lamb").audio_and_sample_rate,
            },
        },
        {  # Test explicit encoder/decoder prompt
            "encoder_prompt": {
                "prompt": "",
                "multi_modal_data": {
                    "audio": AudioAsset("winning_call").audio_and_sample_rate,
                },
            },
            "decoder_prompt": "<|startoftranscript|>",
        },
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include AudioAsset, EngineArgs, and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。关键操作包括 AudioAsset、EngineArgs，以及 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_fireredasr2
```python
def run_fireredasr2():
    """
    FireRedASR2 – Automatic Speech Recognition model.

    This model uses a Conformer encoder + Qwen2 LLM decoder architecture
    for speech-to-text transcription.  Audio is passed via the implicit
    prompt format with the ``<|AUDIO|>`` placeholder token.
    """
    engine_args = EngineArgs(
        model="allendou/FireRedASR2-LLM-vllm",
        max_model_len=448,
        max_num_seqs=16,
        limit_mm_per_prompt={"audio": 1},
    )

    prompt_str = (
        "<|im_start|>user\n<|AUDIO|>请转写音频为文字<|im_end|>\n<|im_start|>assistant\n"
    )

    prompts = [
        {  # Implicit prompt with audio
            "prompt": prompt_str,
            "multi_modal_data": {
                "audio": AudioAsset("mary_had_lamb").audio_and_sample_rate,
            },
        },
        {  # Another audio sample
            "prompt": prompt_str,
            "multi_modal_data": {
                "audio": AudioAsset("winning_call").audio_and_sample_rate,
            },
        },
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** FireRedASR2 – Automatic Speech Recognition model.. Key operations include AudioAsset, EngineArgs, and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。关键操作包括 AudioAsset、EngineArgs，以及 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Function: run_fireredlid
```python
def run_fireredlid():
    """
    FireRedLID – Language Identification model.

    This encoder-decoder model identifies the spoken language of an audio
    clip. It outputs at most 2 tokens representing the detected language
    (e.g. "en", "zh mandarin").
    """
    engine_args = EngineArgs(
        model="PatchyTisa/FireRedLID-vllm",
        max_model_len=8,
        max_num_seqs=16,
        limit_mm_per_prompt={"audio": 1},
    )

    prompts = [
        {  # Test explicit encoder/decoder prompt
            "encoder_prompt": {
                "prompt": "",
                "multi_modal_data": {
                    "audio": AudioAsset("mary_had_lamb").audio_and_sample_rate,
                },
            },
            "decoder_prompt": "<sos>",
        },
        {  # Another audio sample
            "encoder_prompt": {
                "prompt": "",
                "multi_modal_data": {
                    "audio": AudioAsset("winning_call").audio_and_sample_rate,
                },
            },
            "decoder_prompt": "<sos>",
        },
    ]

    return ModelRequestData(
        engine_args=engine_args,
        prompts=prompts,
    )
```
**EN:** FireRedLID – Language Identification model.. Key operations include AudioAsset, EngineArgs, and ModelRequestData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。关键操作包括 AudioAsset、EngineArgs，以及 ModelRequestData。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
model_example_map = {
    "fireredasr2": run_fireredasr2,
    "fireredlid": run_fireredlid,
    "whisper": run_whisper,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as model_example_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 model_example_map 等变量。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser(
        description="Demo on using vLLM for offline inference with "
        "vision language models for text generation"
    )
    parser.add_argument(
        "--model-type",
        "-m",
        type=str,
        default="whisper",
        choices=model_example_map.keys(),
        help='Huggingface "model_type".',
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=0,
        help="Set the seed when initializing `vllm.LLM`.",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, parser.parse_args, and model_example_map.keys. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、parser.parse_args，以及 model_example_map.keys。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    model = args.model_type
    if model not in model_example_map:
        raise ValueError(f"Model type {model} is not supported.")

    req_data = model_example_map[model]()

    # Disable other modalities to save memory
    engine_args = req_data.engine_args
    default_limits = {"image": 0, "video": 0, "audio": 0}
    limit_mm_per_prompt = default_limits | (engine_args.limit_mm_per_prompt or {})
    engine_args.limit_mm_per_prompt = limit_mm_per_prompt
    engine_args.seed = args.seed
    llm = LLM.from_engine_args(engine_args)

    prompts = req_data.prompts

    # Create a sampling params object.
    sampling_params = SamplingParams(
        temperature=0,
        top_p=1.0,
        max_tokens=64,
        skip_special_tokens=False,
    )

    start = time.time()

    # Generate output tokens from the prompts. The output is a list of
    # RequestOutput objects that contain the prompt, generated
    # text, and other information.
    outputs = llm.generate(prompts, sampling_params)

    # Print the outputs.
    for output in outputs:
        prompt = output.prompt
        generated_text = output.outputs[0].text
        print(f"Decoder prompt: {prompt!r}, Generated text: {generated_text!r}")

    duration = time.time() - start

    print("Duration:", duration)
    print("RPS:", len(prompts) / duration)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, time.time, LLM.from_engine_args, SamplingParams, and llm.generate.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、time.time、LLM.from_engine_args、SamplingParams，以及 llm.generate。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.assets.audio`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `time`, `collections.abc`, `typing` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `ModelRequestData`, `run_whisper`, `run_fireredasr2`, `run_fireredlid`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `AudioAsset`, `EngineArgs`, `ModelRequestData`, `parser.add_argument`, `FlexibleArgumentParser`, `parser.parse_args`, `model_example_map.keys`, `print` reveal the main execution path / 这些调用体现了主要执行链路。

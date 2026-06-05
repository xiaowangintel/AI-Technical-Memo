# only_thinker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/qwen2_5_omni/only_thinker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with the correct prompt format on Qwen2.5-Omni (thinker only) / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference
with the correct prompt format on Qwen2.5-Omni (thinker only).
"""
```
**EN:** This example shows how to use vLLM for running offline inference with the correct prompt format on Qwen2.5-Omni (thinker only).
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from typing import NamedTuple

from vllm import LLM, SamplingParams
from vllm.assets.audio import AudioAsset
from vllm.assets.image import ImageAsset
from vllm.assets.video import VideoAsset
from vllm.multimodal.image import convert_image_mode
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as typing and pulls in vLLM APIs like vllm, vllm.assets.audio, vllm.assets.image, vllm.assets.video, and vllm.multimodal.image.
**CN:** 这一部分加载 typing 等辅助库，并引入 vllm、vllm.assets.audio、vllm.assets.image、vllm.assets.video，以及 vllm.multimodal.image 等 vLLM API。

### Class: QueryResult
```python
class QueryResult(NamedTuple):
    inputs: dict
    limit_mm_per_prompt: dict[str, int]
```
**EN:** This class packages the QueryResult abstraction used by the example. It extends NamedTuple.
**CN:** 该类封装了示例中使用的 QueryResult 抽象。它继承自 NamedTuple。

### Top-level setup
```python
default_system = (
    "You are Qwen, a virtual human developed by the Qwen Team, Alibaba "
    "Group, capable of perceiving auditory and visual inputs, as well as "
    "generating text and speech."
)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as default_system.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 default_system 等变量。

### Function: get_mixed_modalities_query
```python
def get_mixed_modalities_query() -> QueryResult:
    question = (
        "What is recited in the audio? "
        "What is the content of this image? Why is this video funny?"
    )
    prompt = (
        f"<|im_start|>system\n{default_system}<|im_end|>\n"
        "<|im_start|>user\n<|audio_bos|><|AUDIO|><|audio_eos|>"
        "<|vision_bos|><|IMAGE|><|vision_eos|>"
        "<|vision_bos|><|VIDEO|><|vision_eos|>"
        f"{question}<|im_end|>\n"
        f"<|im_start|>assistant\n"
    )
    return QueryResult(
        inputs={
            "prompt": prompt,
            "multi_modal_data": {
                "audio": AudioAsset("mary_had_lamb").audio_and_sample_rate,
                "image": convert_image_mode(
                    ImageAsset("cherry_blossom").pil_image, "RGB"
                ),
                "video": VideoAsset(name="baby_reading", num_frames=16).np_ndarrays,
            },
        },
        limit_mm_per_prompt={"audio": 1, "image": 1, "video": 1},
    )
```
**EN:** This function sends a request and handles the reply. Key operations include QueryResult, convert_image_mode, AudioAsset, VideoAsset, and ImageAsset. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。关键操作包括 QueryResult、convert_image_mode、AudioAsset、VideoAsset，以及 ImageAsset。其返回值会继续传给示例管线的下一阶段。

### Function: get_use_audio_in_video_query
```python
def get_use_audio_in_video_query() -> QueryResult:
    question = (
        "Describe the content of the video, then convert what the baby say into text."
    )
    prompt = (
        f"<|im_start|>system\n{default_system}<|im_end|>\n"
        "<|im_start|>user\n<|vision_bos|><|VIDEO|><|vision_eos|>"
        f"{question}<|im_end|>\n"
        f"<|im_start|>assistant\n"
    )
    asset = VideoAsset(name="baby_reading", num_frames=16)
    audio = asset.get_audio(sampling_rate=16000)

    return QueryResult(
        inputs={
            "prompt": prompt,
            "multi_modal_data": {
                "video": asset.np_ndarrays,
                "audio": audio,
            },
            "mm_processor_kwargs": {
                "use_audio_in_video": True,
            },
        },
        limit_mm_per_prompt={"audio": 1, "video": 1},
    )
```
**EN:** This function sends a request and handles the reply. Key operations include VideoAsset, asset.get_audio, and QueryResult. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。关键操作包括 VideoAsset、asset.get_audio，以及 QueryResult。其返回值会继续传给示例管线的下一阶段。

### Function: get_multi_audios_query
```python
def get_multi_audios_query() -> QueryResult:
    question = "Are these two audio clips the same?"
    prompt = (
        f"<|im_start|>system\n{default_system}<|im_end|>\n"
        "<|im_start|>user\n<|audio_bos|><|AUDIO|><|audio_eos|>"
        "<|audio_bos|><|AUDIO|><|audio_eos|>"
        f"{question}<|im_end|>\n"
        f"<|im_start|>assistant\n"
    )
    return QueryResult(
        inputs={
            "prompt": prompt,
            "multi_modal_data": {
                "audio": [
                    AudioAsset("winning_call").audio_and_sample_rate,
                    AudioAsset("mary_had_lamb").audio_and_sample_rate,
                ],
            },
        },
        limit_mm_per_prompt={
            "audio": 2,
        },
    )
```
**EN:** This function sends a request and handles the reply. Key operations include AudioAsset and QueryResult. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。关键操作包括 AudioAsset 和 QueryResult。其返回值会继续传给示例管线的下一阶段。

### Function: get_multi_images_query
```python
def get_multi_images_query() -> QueryResult:
    question = "What are the differences between these two images?"
    prompt = (
        f"<|im_start|>system\n{default_system}<|im_end|>\n"
        "<|im_start|>user\n<|vision_bos|><|IMAGE|><|vision_eos|>"
        "<|vision_bos|><|IMAGE|><|vision_eos|>"
        f"{question}<|im_end|>\n"
        f"<|im_start|>assistant\n"
    )
    return QueryResult(
        inputs={
            "prompt": prompt,
            "multi_modal_data": {
                "image": [
                    convert_image_mode(ImageAsset("cherry_blossom").pil_image, "RGB"),
                    convert_image_mode(ImageAsset("stop_sign").pil_image, "RGB"),
                ],
            },
        },
        limit_mm_per_prompt={
            "image": 2,
        },
    )
```
**EN:** This function sends a request and handles the reply. Key operations include convert_image_mode, ImageAsset, and QueryResult. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。关键操作包括 convert_image_mode、ImageAsset，以及 QueryResult。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
query_map = {
    "mixed_modalities": get_mixed_modalities_query,
    "use_audio_in_video": get_use_audio_in_video_query,
    "multi_audios": get_multi_audios_query,
    "multi_images": get_multi_images_query,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as query_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 query_map 等变量。

### Function: main
```python
def main(args):
    model_name = "Qwen/Qwen2.5-Omni-7B"
    query_result = query_map[args.query_type]()

    llm = LLM(
        model=model_name,
        max_model_len=5632,
        max_num_seqs=5,
        limit_mm_per_prompt=query_result.limit_mm_per_prompt,
        seed=args.seed,
    )

    # We set temperature to 0.2 so that outputs can be different
    # even when all prompts are identical when running batch inference.
    sampling_params = SamplingParams(temperature=0.2, max_tokens=64)

    outputs = llm.generate(query_result.inputs, sampling_params=sampling_params)

    for o in outputs:
        generated_text = o.outputs[0].text
        print(generated_text)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include LLM, SamplingParams, llm.generate, and print.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 LLM、SamplingParams、llm.generate，以及 print。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser(
        description="Demo on using vLLM for offline inference with "
        "audio language models"
    )
    parser.add_argument(
        "--query-type",
        "-q",
        type=str,
        default="mixed_modalities",
        choices=query_map.keys(),
        help="Query type.",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=0,
        help="Set the seed when initializing `vllm.LLM`.",
    )

    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, parser.parse_args, and query_map.keys. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、parser.parse_args，以及 query_map.keys。其返回值会继续传给示例管线的下一阶段。

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
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.assets.audio`, `vllm.assets.image`, `vllm.assets.video`, `vllm.multimodal.image`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `typing` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `QueryResult`, `get_mixed_modalities_query`, `get_use_audio_in_video_query`, `get_multi_audios_query`, `get_multi_images_query`, `main`, `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `QueryResult`, `convert_image_mode`, `AudioAsset`, `VideoAsset`, `ImageAsset`, `asset.get_audio`, `LLM`, `SamplingParams` reveal the main execution path / 这些调用体现了主要执行链路。

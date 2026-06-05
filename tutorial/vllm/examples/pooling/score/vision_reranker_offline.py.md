# vision_reranker_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/vision_reranker_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use vLLM for running offline inference with vision language reranker models for multimodal scoring tasks / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use vLLM for running offline inference with
vision language reranker models for multimodal scoring tasks.

Vision language rerankers score the relevance between a text query and
multimodal documents (text + images/videos).
"""
```
**EN:** This example shows how to use vLLM for running offline inference with vision language reranker models for multimodal scoring tasks.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from argparse import Namespace
from collections.abc import Callable
from pathlib import Path
from typing import NamedTuple

from vllm import LLM, EngineArgs
from vllm.multimodal.utils import encode_image_url, fetch_image
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as argparse, collections.abc, pathlib, and typing and pulls in vLLM APIs like vllm, vllm.multimodal.utils, and vllm.utils.argparse_utils.
**CN:** 这一部分加载 argparse、collections.abc、pathlib，以及 typing 等辅助库，并引入 vllm、vllm.multimodal.utils，以及 vllm.utils.argparse_utils 等 vLLM API。

### Top-level setup
```python
TEMPLATE_HOME = Path(__file__).parent / "template"


query = "A woman playing with her dog on a beach at sunset."
document = (
    "A woman shares a joyful moment with her golden retriever on a sun-drenched "
    "beach at sunset, as the dog offers its paw in a heartwarming display of "
    "companionship and trust."
)
image_url = "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"
video_url = "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen3-Omni/demo/draw.mp4"
documents = [
# ... key logic omitted for brevity ...
        "type": "image_url",
        "image_url": {"url": encode_image_url(fetch_image(image_url))},
    },
    {
        "type": "video_url",
        "video_url": {"url": video_url},
    },
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as TEMPLATE_HOME, query, document, image_url, video_url, and documents. It also performs early helper calls such as Path, encode_image_url, and fetch_image.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 TEMPLATE_HOME、query、document、image_url、video_url，以及 documents 等变量。它还会提前执行 Path、encode_image_url，以及 fetch_image 等辅助调用。

### Class: RerankModelData
```python
class RerankModelData(NamedTuple):
    engine_args: EngineArgs
    chat_template: str | None = None
    modality: set[str] = {}
```
**EN:** This class packages the RerankModelData abstraction used by the example. It extends NamedTuple.
**CN:** 该类封装了示例中使用的 RerankModelData 抽象。它继承自 NamedTuple。

### Function: run_jinavl_reranker
```python
def run_jinavl_reranker() -> RerankModelData:
    engine_args = EngineArgs(
        model="jinaai/jina-reranker-m0",
        runner="pooling",
        max_model_len=32768,
        trust_remote_code=True,
        mm_processor_kwargs={
            "min_pixels": 3136,
            "max_pixels": 602112,
        },
    )
    return RerankModelData(engine_args=engine_args, modality={"image"})
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include EngineArgs and RerankModelData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。关键操作包括 EngineArgs 和 RerankModelData。其返回值会继续传给示例管线的下一阶段。

### Function: run_qwen3_vl_reranker
```python
def run_qwen3_vl_reranker() -> RerankModelData:
    engine_args = EngineArgs(
        model="Qwen/Qwen3-VL-Reranker-2B",
        runner="pooling",
        max_model_len=16384,
        # HuggingFace model configuration overrides required for compatibility
        hf_overrides={
            # Manually route to sequence classification architecture
            # This tells vLLM to use Qwen3VLForSequenceClassification instead of
            # the default Qwen3VLForConditionalGeneration
            "architectures": ["Qwen3VLForSequenceClassification"],
            # Specify which token logits to extract from the language model head
            # The original reranker uses "no" and "yes" token logits for scoring
            "classifier_from_token": ["no", "yes"],
            # Enable special handling for original Qwen3-Reranker models
            # This flag triggers conversion logic that transforms the two token
            # vectors into a single classification vector
            "is_original_qwen3_reranker": True,
        },
    )
    chat_template_path = "qwen3_vl_reranker.jinja"
    chat_template = (TEMPLATE_HOME / chat_template_path).read_text()
    return RerankModelData(
        engine_args=engine_args,
        chat_template=chat_template,
        modality={"image", "video"},
    )
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include EngineArgs, read_text, and RerankModelData. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。关键操作包括 EngineArgs、read_text，以及 RerankModelData。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
model_example_map: dict[str, Callable[[], RerankModelData]] = {
    "jinavl_reranker": run_jinavl_reranker,
    "qwen3_vl_reranker": run_qwen3_vl_reranker,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as model_example_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 model_example_map 等变量。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser(
        description="Demo on using vLLM for offline inference with "
        "vision language reranker models for multimodal scoring tasks."
    )
    parser.add_argument(
        "--model-name",
        "-m",
        type=str,
        default="jinavl_reranker",
        choices=model_example_map.keys(),
        help="The name of the reranker model.",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include FlexibleArgumentParser, parser.add_argument, parser.parse_args, and model_example_map.keys. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 FlexibleArgumentParser、parser.add_argument、parser.parse_args，以及 model_example_map.keys。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args: Namespace):
    # Run the selected reranker model
    model_request = model_example_map[args.model_name]()
    engine_args = model_request.engine_args

    llm = LLM.from_engine_args(engine_args)

    print("Query: string & Document: string")
    outputs = llm.score(query, document)
    print("Relevance scores:", [output.outputs.score for output in outputs])

    print("Query: string & Document: text")
    outputs = llm.score(
        query, {"content": [documents[0]]}, chat_template=model_request.chat_template
    )
    print("Relevance scores:", [output.outputs.score for output in outputs])

    print("Query: string & Document: image url")
    outputs = llm.score(
        query, {"content": [documents[1]]}, chat_template=model_request.chat_template
    # ... key logic omitted for brevity ...
    print("Relevance scores:", [output.outputs.score for output in outputs])

    print("Query: string & Document: list")
    outputs = llm.score(
        query,
        [
            document,
            {"content": [documents[0]]},
            {"content": [documents[1]]},
            {"content": [documents[0], documents[1]]},
        ],
        chat_template=model_request.chat_template,
    )
    print("Relevance scores:", [output.outputs.score for output in outputs])
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, llm.score, and LLM.from_engine_args.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、llm.score，以及 LLM.from_engine_args。

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
- **RAG orchestration / RAG 编排**: Documents are loaded, indexed, retrieved, and injected into prompts. / 文档会被加载、索引、检索，并注入到提示词中。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.multimodal.utils`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `collections.abc`, `pathlib`, `typing` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `RerankModelData`, `run_jinavl_reranker`, `run_qwen3_vl_reranker`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `Path`, `encode_image_url`, `fetch_image`, `EngineArgs`, `RerankModelData`, `read_text`, `FlexibleArgumentParser`, `parser.add_argument` reveal the main execution path / 这些调用体现了主要执行链路。

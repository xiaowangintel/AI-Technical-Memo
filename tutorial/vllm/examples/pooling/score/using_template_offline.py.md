# using_template_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/using_template_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates scoring and reranking workflows. / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
from argparse import Namespace
from pathlib import Path
from typing import Any

from vllm import LLM, EngineArgs
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as argparse, pathlib, and typing and pulls in vLLM APIs like vllm and vllm.utils.argparse_utils.
**CN:** 这一部分加载 argparse、pathlib，以及 typing 等辅助库，并引入 vllm 和 vllm.utils.argparse_utils 等 vLLM API。

### Function: parse_args
```python
def parse_args():
    """Parse command line arguments for the reranking example.

    This function sets up the argument parser with default values
    specific to reranking models, including the model name and
    runner type.
    """
    parser = FlexibleArgumentParser()
    # Add all EngineArgs command line arguments to the parser
    parser = EngineArgs.add_cli_args(parser)

    # Set default values specific to this reranking example
    # These defaults ensure the script works out-of-the-box for reranking tasks
    parser.set_defaults(
        model="nvidia/llama-nemotron-rerank-1b-v2",  # Default reranking model
        runner="pooling",  # Required for cross-encoder/reranking models
        trust_remote_code=True,  # Allow loading models with custom code
    )
    return parser.parse_args()
```
**EN:** Parse command line arguments for the reranking example.. Key operations include FlexibleArgumentParser, EngineArgs.add_cli_args, parser.set_defaults, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 FlexibleArgumentParser、EngineArgs.add_cli_args、parser.set_defaults，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: get_chat_template
```python
def get_chat_template(model: str) -> str:
    """Load the appropriate chat template for the specified model.

    Reranking models require specific prompt templates to format
    query-document pairs correctly. This function maps model names
    to their corresponding template files.
    """
    # Directory containing all chat template files
    template_home = Path(__file__).parent / "template"

    # Mapping from model names to their corresponding template files
    # Each reranking model has its own specific prompt format
    model_name_to_template_path_map = {
        "BAAI/bge-reranker-v2-gemma": "bge-reranker-v2-gemma.jinja",
        "Qwen/Qwen3-Reranker-0.6B": "qwen3_reranker.jinja",
        "Qwen/Qwen3-Reranker-4B": "qwen3_reranker.jinja",
        "Qwen/Qwen3-Reranker-8B": "qwen3_reranker.jinja",
        "tomaarsen/Qwen3-Reranker-0.6B-seq-cls": "qwen3_reranker.jinja",
        "tomaarsen/Qwen3-Reranker-4B-seq-cls": "qwen3_reranker.jinja",
        "tomaarsen/Qwen3-Reranker-8B-seq-cls": "qwen3_reranker.jinja",
        "mixedbread-ai/mxbai-rerank-base-v2": "mxbai_rerank_v2.jinja",
        "mixedbread-ai/mxbai-rerank-large-v2": "mxbai_rerank_v2.jinja",
        "nvidia/llama-nemotron-rerank-1b-v2": "nemotron-rerank.jinja",
    }

    # Get the template filename for the specified model
    template_path = model_name_to_template_path_map.get(model)

    if template_path is None:
        raise ValueError(f"This demo does not support model name: {model}.")

    # Read and return the template content
    return (template_home / template_path).read_text()
```
**EN:** Load the appropriate chat template for the specified model.. It works with parameters such as model. Key operations include model_name_to_template_path_map.get, read_text, ValueError, and Path. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 model 等参数。关键操作包括 model_name_to_template_path_map.get、read_text、ValueError，以及 Path。其返回值会继续传给示例管线的下一阶段。

### Function: get_hf_overrides
```python
def get_hf_overrides(model: str) -> dict[str, Any]:
    """Convert Large Language Models (LLMs) to Sequence Classification models.

    note:
        Some reranking models require special configuration overrides to work
        correctly with vLLM's score API.
        Reference: https://github.com/vllm-project/vllm/blob/main/examples/pooling/score/qwen3_reranker_offline.py
        Reference: https://github.com/vllm-project/vllm/blob/main/examples/pooling/score/convert_model_to_seq_cls.py
    """

    model_name_to_hf_overrides_map = {
        "BAAI/bge-reranker-v2-gemma": {
            "architectures": ["GemmaForSequenceClassification"],
            "classifier_from_token": ["Yes"],
            "method": "no_post_processing",
        },
        "Qwen/Qwen3-Reranker-0.6B": {
            "architectures": ["Qwen3ForSequenceClassification"],
            "classifier_from_token": ["no", "yes"],
            "is_original_qwen3_reranker": True,
    # ... key logic omitted for brevity ...
        "mixedbread-ai/mxbai-rerank-large-v2": {
            "architectures": ["Qwen2ForSequenceClassification"],
            "classifier_from_token": ["0", "1"],
            "method": "from_2_way_softmax",
        },
        "nvidia/llama-nemotron-rerank-1b-v2": {},
    }

    hf_overrides = model_name_to_hf_overrides_map.get(model)

    if hf_overrides is None:
        raise ValueError(f"This demo does not support model name: {model}.")

    return hf_overrides
```
**EN:** Convert Large Language Models (LLMs) to Sequence Classification models.. It works with parameters such as model. Key operations include model_name_to_hf_overrides_map.get and ValueError. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 model 等参数。关键操作包括 model_name_to_hf_overrides_map.get 和 ValueError。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args: Namespace):
    """Main execution function for the reranking example."""

    # Get the overrides for the specified model
    args.hf_overrides = get_hf_overrides(args.model)

    # Initialize the LLM with all provided arguments
    llm = LLM(**vars(args))

    # Example query for demonstration
    query = "how much protein should a female eat?"

    # Example documents to be reranked based on relevance to the query
    documents = [
        "As a general guideline, the CDC's average requirement of protein for women ages 19 to 70 is 46 grams per day. But, as you can see from this chart, you'll need to increase that if you're expecting or training for a marathon. Check out the chart below to see how much protein you should be eating each day.",
        "Definition of summit for English Language Learners. : 1  the highest point of a mountain : the top of a mountain. : 2  the highest level. : 3  a meeting or series of meetings between the leaders of two or more governments.",
        "Calorie intake should not fall below 1,200 a day in women or 1,500 a day in men, except under the supervision of a health professional.",
    ]

    # Load the appropriate chat template for the selected model
    # The template formats query-document pairs for the reranking model
    chat_template = get_chat_template(args.model)

    # Score documents based on relevance to the query
    # The score method returns relevance scores for each document
    outputs = llm.score(query, documents, chat_template=chat_template)

    # Display the relevance scores
    # Higher scores indicate more relevant documents
    print("-" * 30)
    print([output.outputs.score for output in outputs])
    print("-" * 30)
```
**EN:** Main execution function for the reranking example.. It works with parameters such as args. Key operations include print, get_hf_overrides, LLM, get_chat_template, and llm.score.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、get_hf_overrides、LLM、get_chat_template，以及 llm.score。

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
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。
- **Task-specific scoring / 任务特定打分**: Model outputs are converted into scores, labels, or reward values. / 模型输出会被转换为分数、标签或奖励值。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `pathlib`, `typing` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args`, `get_chat_template`, `get_hf_overrides`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `FlexibleArgumentParser`, `EngineArgs.add_cli_args`, `parser.set_defaults`, `parser.parse_args`, `model_name_to_template_path_map.get`, `read_text`, `ValueError`, `Path` reveal the main execution path / 这些调用体现了主要执行链路。

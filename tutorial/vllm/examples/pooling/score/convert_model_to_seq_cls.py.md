# convert_model_to_seq_cls.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/convert_model_to_seq_cls.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Script to convert Large Language Models (LLMs) to Sequence Classification models / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Script to convert Large Language Models (LLMs) to Sequence Classification models.
This is particularly useful for converting reranker models that use next-token
prediction to a sequence classification format for compatibility with standard
classification and rerank pipelines.

Usage examples:
- For BAAI/bge-reranker-v2-gemma:
  python convert_model_to_seq_cls.py --model_name BAAI/bge-reranker-v2-gemma \
    --classifier_from_tokens '["Yes"]' --method no_post_processing \
    --path ./bge-reranker-v2-gemma-seq-cls

- For mxbai-rerank-v2:
  python convert_model_to_seq_cls.py --model_name mixedbread-ai/mxbai-rerank-base-v2 \
    --classifier_from_tokens '["0", "1"]' --method from_2_way_softmax \
    --path ./mxbai-rerank-base-v2-seq-cls

- For Qwen3-Reranker:
  python convert_model_to_seq_cls.py --model_name Qwen/Qwen3-Reranker-0.6B \
    --classifier_from_tokens '["no", "yes"]' --method from_2_way_softmax \
    --path ./Qwen3-Reranker-0.6B-seq-cls

Note: For BAAI/bge-reranker-v2-gemma, "Yes" and "yes" are different tokens.
"""
```
**EN:** Script to convert Large Language Models (LLMs) to Sequence Classification models.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import json

import torch
import transformers
```
**EN:** This block loads helper libraries such as argparse, json, torch, and transformers.
**CN:** 这一部分加载 argparse、json、torch，以及 transformers 等辅助库。

### Function: from_2_way_softmax
```python
def from_2_way_softmax(causal_lm, seq_cls_model, tokenizer, tokens, device):
    """
    This method extracts the difference between weights for 'true' and 'false' tokens
    from the language model head to create a single classification weight vector.

    Args:
        causal_lm: The original causal language model
        seq_cls_model: The target sequence classification model
        tokenizer: Model tokenizer
        tokens: List of two tokens representing [false_token, true_token]
        device: Target device (cpu/cuda)

    Reference: https://huggingface.co/Qwen/Qwen3-Reranker-0.6B/discussions/3
    """
    assert len(tokens) == 2, (
        "Method requires exactly two tokens for binary classification"
    )

    # Get the language model head weights (vocabulary_size x hidden_size)
    lm_head_weights = causal_lm.lm_head.weight

    # Convert token strings to their corresponding token IDs
    false_id = tokenizer.convert_tokens_to_ids(tokens[0])
    true_id = tokenizer.convert_tokens_to_ids(tokens[1])

    # Compute the classification weight as the difference between true and false token weights
    # This follows the approach in: https://huggingface.co/Qwen/Qwen3-Reranker-0.6B/discussions/3
    score_weight = lm_head_weights[true_id].to(device).to(
        torch.float32
    ) - lm_head_weights[false_id].to(device).to(torch.float32)

    # Copy the computed weights to the sequence classification model
    with torch.no_grad():
        seq_cls_model.score.weight.copy_(score_weight.unsqueeze(0))
        if seq_cls_model.score.bias is not None:
            seq_cls_model.score.bias.zero_()
```
**EN:** This method extracts the difference between weights for 'true' and 'false' tokens from the language model head to create a single classification weight vector.. It works with parameters such as causal_lm, seq_cls_model, tokenizer, tokens, and device. Key operations include to, tokenizer.convert_tokens_to_ids, len, torch.no_grad, and seq_cls_model.score.weight.copy_.
**CN:** 该函数封装示例中的可复用步骤。它会处理 causal_lm、seq_cls_model、tokenizer、tokens，以及 device 等参数。关键操作包括 to、tokenizer.convert_tokens_to_ids、len、torch.no_grad，以及 seq_cls_model.score.weight.copy_。

### Function: no_post_processing
```python
def no_post_processing(causal_lm, seq_cls_model, tokenizer, tokens, device):
    """
    Directly use token weights from the language model head for classification.

    This method maps each classification label directly to a corresponding token
    in the vocabulary without additional transformation.

    Args:
        causal_lm: The original causal language model
        seq_cls_model: The target sequence classification model
        tokenizer: Model tokenizer
        tokens: List of tokens representing class labels
        device: Target device (cpu/cuda)
    """
    # Get the language model head weights (vocabulary_size x hidden_size)
    lm_head_weights = causal_lm.lm_head.weight

    # Convert all tokens to their corresponding token IDs
    token_ids = [tokenizer.convert_tokens_to_ids(t) for t in tokens]

    # Extract weights for the specific tokens (num_tokens x hidden_size)
    score_weight = lm_head_weights[token_ids].to(device)

    # Copy the weights to the sequence classification model
    with torch.no_grad():
        seq_cls_model.score.weight.copy_(score_weight)
        if seq_cls_model.score.bias is not None:
            seq_cls_model.score.bias.zero_()
```
**EN:** Directly use token weights from the language model head for classification.. It works with parameters such as causal_lm, seq_cls_model, tokenizer, tokens, and device. Key operations include to, tokenizer.convert_tokens_to_ids, torch.no_grad, seq_cls_model.score.weight.copy_, and seq_cls_model.score.bias.zero_.
**CN:** 该函数发送请求并处理返回结果。它会处理 causal_lm、seq_cls_model、tokenizer、tokens，以及 device 等参数。关键操作包括 to、tokenizer.convert_tokens_to_ids、torch.no_grad、seq_cls_model.score.weight.copy_，以及 seq_cls_model.score.bias.zero_。

### Top-level setup
```python
method_map = {
    function.__name__: function for function in [from_2_way_softmax, no_post_processing]
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as method_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 method_map 等变量。

### Function: converting
```python
def converting(
    model_name, classifier_from_tokens, path, method, use_sep_token=False, device="cpu"
):
    """
    Main conversion function to transform a CausalLM model to SequenceClassification.

    Args:
        model_name: Name or path of the pretrained model
        classifier_from_tokens: List of tokens used for classification
        path: Output path to save the converted model
        method: Conversion method ('from_2_way_softmax' or 'no_post_processing')
        use_sep_token: Whether to use separating token in the sequence classification model
        device: Device to load the model on ('cpu' or 'cuda')
    """
    assert method in method_map, f"Unknown method: {method}"

    # Determine number of labels based on conversion method
    if method == "from_2_way_softmax":
        assert len(classifier_from_tokens) == 2
        num_labels = 1
    # ... key logic omitted for brevity ...

    # Apply the selected conversion method to transfer weights
    method_map[method](
        causal_lm, seq_cls_model, tokenizer, classifier_from_tokens, device
    )

    # Configure separating token settings
    # Note: `llm as reranker` defaults to not using separating token.
    seq_cls_model.config.use_sep_token = use_sep_token
    seq_cls_model.config.sep_token_id = tokenizer.sep_token_id

    # Save the converted model and tokenizer
    seq_cls_model.save_pretrained(path)
    tokenizer.save_pretrained(path)
```
**EN:** Main conversion function to transform a CausalLM model to SequenceClassification.. It works with parameters such as model_name, classifier_from_tokens, path, method, use_sep_token, and device. Key operations include len, transformers.AutoTokenizer.from_pretrained, transformers.AutoModelForCausalLM.from_pretrained, transformers.AutoModelForSequenceClassification.from_pretrained, and seq_cls_model.save_pretrained.
**CN:** 该函数把中间结果转换成所需格式。它会处理 model_name、classifier_from_tokens、path、method、use_sep_token，以及 device 等参数。关键操作包括 len、transformers.AutoTokenizer.from_pretrained、transformers.AutoModelForCausalLM.from_pretrained、transformers.AutoModelForSequenceClassification.from_pretrained，以及 seq_cls_model.save_pretrained。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Converting *ForCausalLM models to "
        "*ForSequenceClassification models."
    )
    parser.add_argument(
        "--model_name",
        type=str,
        default="BAAI/bge-reranker-v2-gemma",
        help="HuggingFace model name or local path",
    )
    parser.add_argument(
        "--classifier_from_tokens",
        type=str,
        default='["Yes"]',
        help="JSON string of tokens used for classification labels",
    )
    parser.add_argument(
        "--method",
        type=str,
        default="no_post_processing",
        help="Conversion method to use",
    )
    parser.add_argument(
        "--use-sep-token",
        action="store_true",
        help="Enable separating token in the sequence classification model",
    )
    parser.add_argument(
        "--path",
        type=str,
        default="./bge-reranker-v2-gemma-seq-cls",
        help="Output directory to save the converted model",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()

    converting(
        model_name=args.model_name,
        classifier_from_tokens=json.loads(args.classifier_from_tokens),
        method=args.method,
        use_sep_token=args.use_sep_token,
        path=args.path,
    )
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args, converting, and json.loads.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args、converting，以及 json.loads。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。
- **Task-specific scoring / 任务特定打分**: Model outputs are converted into scores, labels, or reward values. / 模型输出会被转换为分数、标签或奖励值。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `json`, `torch`, `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `from_2_way_softmax`, `no_post_processing`, `converting`, `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `to`, `tokenizer.convert_tokens_to_ids`, `len`, `torch.no_grad`, `seq_cls_model.score.weight.copy_`, `score_weight.unsqueeze`, `seq_cls_model.score.bias.zero_`, `transformers.AutoTokenizer.from_pretrained` reveal the main execution path / 这些调用体现了主要执行链路。
